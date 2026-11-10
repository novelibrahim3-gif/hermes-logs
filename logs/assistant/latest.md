Audit selesai. Berikut laporan lengkapnya.

---

# AUDIT READ-ONLY — Hook `post_tool_call` & `sync_logs.py`

## 1. Konfigurasi hook aktif

```yaml
hooks:
  post_tool_call:
    - matcher: "^terminal$"
      command: "/root/hermes-logs-sync/sync_logs.py"
      timeout: 120
```

- `fail_closed` tidak diset → default **false** (fail-open).
- Matcher `^terminal$` → hook ini jalan di **setiap** panggilan tool `terminal`.
- Log registrasi: `shell hook registered: post_tool_call -> .../sync_logs.py (matcher=^terminal$, timeout=120s, fail_closed=False)`.

## 2. Isi `sync_logs.py` (alur)

```
stdin JSON → parse → validasi event/tool
  → clean_data() (redaksi regex token/password)
  → tulis logs/tool-results/calls-YYYYMMDD.jsonl   (APPEND, lokal)
  → tulis logs/tool-results/latest.txt             (lokal)
  → copy *.txt dari /sdcard/Download/ChatGPT-Logs → logs/
  → git add -- logs
  → git diff --cached --quiet        (0 = tidak ada perubahan → return)
  → git commit -m "Sync Hermes terminal results"
  → git push origin main             ← gagal di sini
```

## 3. Alur pemanggilan (synchronous)

```
terminal tool selesai
  → model_tools._emit_post_tool_call_hook()
  → hermes_cli.lifecycle.invoke_hook("post_tool_call")
  → shell hook callback (agent/shell_hooks.py:_callback)
  → _spawn() → subprocess.Popen + communicate(timeout=120)   ← BLOKIR
  → sync_logs.py selesai → agent baru lanjut
```

Hook **sinkron** — agent menunggu sampai script selesai (termasuk git push).

## 4. Penyebab `git push` exit 128

**Terkonfirmasi via dry-run dengan credential helper dimatikan:**

```
fatal: could not read Username for 'https://github.com': terminal prompts disabled
exit: 128
```

Penyebab: **tidak ada credential persisten.**
- `credential.helper = cache --timeout=3600` → **memori-only**, ada di `~/.git-credential-cache/`.
- Direktori cache **tidak ada** saat diperiksa → cache kosong/daemon sudah mati.
- Tidak ada `~/.git-credentials`, tidak ada token di URL remote, tidak ada `GH_TOKEN`/`GITHUB_TOKEN`.
- Hook jalan non-interaktif (stdin = payload, bukan TTY) → git tidak bisa meminta username → **exit 128**.

Akibatnya: push **intermittent** — berhasil saat cache masih hangat, gagal saat cache kedaluwarsa (1 jam) atau daemon mati. 7 kegagalan hari ini (20:22, 22:01, 22:06, 22:07, 23:20).

## 5. Remote, branch, status, auth

| Item | Nilai |
|---|---|
| Remote | `https://github.com/novelibrahim3-gif/hermes-logs.git` |
| Branch | `main` → tracking `origin/main` |
| Status | bersih (hanya untracked `__pycache__/`, `log_llm.py`, `sync_logs.py` — di luar `logs/`) |
| Remote reachable | ya (`ls-remote` 0,79s) |
| Local vs remote | sinkron di `b4eeef7` |
| Kredensial | hanya cache memori (tidak persisten) |

## 6. Pekerjaan tidak perlu & latency per terminal call

| Komponen | Waktu |
|---|---|
| Python startup + parse (early-return) | **0,314s** |
| `git add -- logs` | 0,025s |
| `git diff --cached --quiet` | 0,015s |
| `git commit` | ~0,2s |
| `git push` (network) | ~0,8s |
| **Total per terminal call** | **~1,5 detik** |

**Bukti pemborosan:**
- **1114 terminal call hari ini** → ~28 menit terbuang hanya untuk hook.
- **170 commit** dibuat (hampir 1 commit per terminal call; cadence 3–40 detik).
- `logs/tool-results/calls-20261111.jsonl` baru 165 baris, tapi 170 commit.
- Setiap commit = `git add` + `diff` + `commit` + `push` diulang.

## 7. Keamanan log lokal

**Log lokal TIDAK hilang jika push gagal.** Penulisan lokal (`calls-YYYYMMDD.jsonl` + `latest.txt`) terjadi di **baris 97–119**, sebelum operasi git (baris 129–141). Jadi kegagalan push hanya memengaruhi sinkronisasi GitHub, bukan penyimpanan lokal. ✓

Masalahnya: commit lokal tetap dibuat, dan saat push gagal, commit menumpuk di lokal (branch mendahului remote) — berisiko non-fast-forward bila remote berubah.

---

## Rekomendasi perubahan minimal

### A. Pisahkan push dari hook per-call (paling penting)

Hapus `git push origin main` dari `sync_logs.py`. Hook cukup:
- tulis JSONL + latest.txt (lokal) ✓
- `git add` + `git commit` (lokal, murah ~0,04s) — opsional
- **jangan push**

Push dipindah ke jalur terpisah:
- **Cron berkala** (mis. tiap 15–30 menit) yang menjalankan `git push`, atau
- **Saat session berakhir** (`on_session_end`), atau
- **Every N call** (mis. push hanya jika JSONL bertambah ≥ 50 baris sejak push terakhir).

Ini menghilangkan ~1,5s × 1114 = **~28 menit/hari** latency.

### B. Credential persisten (perbaiki exit 128)

Ganti cache memori dengan credential persisten:
- `git config --global credential.helper store` + simpan token di `~/.git-credentials` (chmod 600), **atau**
- `gh auth setup-git` (GitHub CLI), **atau**
- remote SSH (`git@github.com:...`).

Dengan ini push tidak lagi gagal saat cache kedaluwarsa.

### C. Fail-open tanpa noise

Bungkus `git push` dalam `try/except`: kegagalan dicatat ke `logs/sync-errors.log`, **jangan `exit 1`**. Agent tidak perlu tahupush gagal; log lokal sudah aman.

### D. Batch commit (opsional)

Alih-alih commit per panggilan, akumulasi dan commit berkala (mis. tiap 10 menit) — mengurangi 170 commit menjadi ~10–20.

---

## Rancangan perbaikan konkret

**`sync_logs.py` (minimal):**
```python
    # ... tulis JSONL + latest.txt (tetap di awal, lokal) ...
    run("git", "add", "--", "logs")
    changed = subprocess.run(["git", "diff", "--cached", "--quiet"], cwd=REPO).returncode
    if changed == 0:
        print(json.dumps({"status": "ok", "action": "no_changes"})); return
    run("git", "commit", "-m", "Sync Hermes terminal results")
    # PUSH DIPINDAH — tidak di sini
    print(json.dumps({"status": "ok", "action": "committed"}))
```

**`sync_push.py` (baru, dijalankan cron/session-end):**
```python
    try:
        run("git", "push", "origin", "main")
    except subprocess.CalledProcessError as e:
        (OUT / "sync-errors.log").open("a").write_text(f"{datetime.now()} push failed: {e}\n")
        # jangan exit 1 — fail open
```

**Auth:** `git config --global credential.helper store` + token di `~/.git-credentials`.

**Hook config:** tidak perlu diubah (tetap mencatat lokal). Bila push sudah dipindah, hook menjadi ringan (~0,4s) dan tidak bergantung pada network.

---

**Ringkasan penyebab:** hook menjalankan `git push` (network, ~0,8s) + commit (**sinkron, memblokir agent**) di **setiap** terminal call, dengan credential memori yang kedaluwarsa → exit 128 intermittent. Log lokal sudah aman; yang perlu diperbaiki adalah **memindahkan push keluar dari hook** dan **membuat credential persisten**.

Tidak ada file diubah, tidak ada commit/push, tidak ada rahasia ditampilkan.
