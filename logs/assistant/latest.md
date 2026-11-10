Audit selesai. Berikut laporan lengkapnya.

---

# AUDIT READ-ONLY — Optimasi LongCat-2.5 sebagai Agent Coding & Automation

## Ringkasan eksekutif

Sistem berfungsi, tapi ada **3 masalah struktural** yang menyebabkan lambat dan kualitas menurun:

1. **Context compression mati** — context membengkak sampai **~231k token** dan terus tumbuh; setiap request mengirim riwayat penuh.
2. **Semua provider auxiliary tidak tersedia** (nous & Google sama-sama kena 429) — compression dan vision gagal/fallback lambat.
3. **Hook `sync_logs.py` gagal di setiap panggilan terminal** — menambah latency dan noise.

---

## 1. Versi & model aktif

| Item | Nilai |
|---|---|
| Hermes | v0.23.0 (2026.9.24), stable, local `2bd378ed1b8d` |
| Python / SDK | 3.14.9 / OpenAI SDK 2.54.0 |
| Model | `meituan/longcat-2.5:free` |
| Provider | Nous Portal |
| fallback_active | **false** |

## 2. Endpoint & protokol

- Endpoint: `https://inference-api.nousresearch.com/v1`
- Protokol: **OpenAI chat_completions** (streaming)
- Prompt cache: **95–99% hit** (16,4 juta cache-read token kumulatif)
-tidak ada fallback provider yang dikonfigurasi.

## 3. reasoning_effort & thinking

- `agent.reasoning_effort: medium`
- Output token: **101.191 / 154 panggilan ≈ 657 token/panggilan** — model banyak "berpikir" (kadang berguna, kadang pemborosan).
- Tidak ada `thinking_budget` eksplisit.

## 4. Context compression & riwayat

| Setting | Nilai |
|---|---|
| compression.enabled | true |
| threshold | 0.50 (difloorkan 0.75 untuk window <512K) |
| target_ratio | 0.20 |
| tail_mode | lean |
| protect_last_n | 20 |
| proactive_prune_tokens | **0 (OFF)** |
| micro_compact | false |

**Masalah:** `No auxiliary LLM provider for compression — summaries will be unavailable.` Compression terpicu tapi **gagal membuat ringkasan**. Akibatnya context tumbuh: 142k → 206k → **231k token** (426 pesan: 202 assistant, 201 tool, 25 user).

## 5. Toolsets, MCP, skills, Graphify

- **Toolsets aktif:** web, browser, terminal, file, code_execution, vision, image_gen, tts, skills, todo, memory, session_search, connections, clarify, delegation, cronjob, computer_use.
- **MCP:** `buddy` (HTTP, timeout 120s), `craftworld-vision` (stdio).
- **Skills:** ~38 direktori.
- **Graphify:** `graphify-out/graph.json` (51 node) di `/root/craftworld-vision`.
- **tool_search:** 18 core tools, 62 deferred (~13.441 token) — sudah aktif, bagus.

## 6. Auxiliary models

| Tugas | Rute | Hasil |
|---|---|---|
| vision | auto → nous `stepfun/step-3.7-flash:free` → **429** (cooldown 164–691s) → fallback main model | **74–117 detik/panggilan** |
| compression | auto → nous (429) → OpenRouter (tanpa key) → custom (tidak ada) → Google (key **exhausted/429**) | **GAGAL** |
| fallback_providers | — | **kosong** |

## 7. Timeout, retry, tool berulang

- `agent.api_max_retries: 3`, `auto_recovery_cycles: 5`
- terminal timeout 180s; MCP 120s; hook 120s; auxiliary 120s
- **Tool berulang:** 154 panggilan API dalam satu sesi; vision dipanggil berulang kali (74–117s each).
- **Hook `post_tool_call` → `sync_logs.py`** jalan di **setiap** panggilan terminal: `git add/commit/push`, dan **`git push` gagal (exit 128)** — latency + noise di tiap terminal call.

## 8. Pengiriman riwayat penuh

**Ya.** Setiap request mengirim seluruh riwayat: `in=206k–231k token`. Contoh: `in=206078`, `in=231595`.

## 9. Log: waktu respons, token, tool wait

- **2470 panggilan API hari ini**, **748 kejadian 429/rate_limit**.
- Latensi API: tipikal **4–25s**, spike **45s, 94.9s, 111.3s**.
- Tool: terminal 0.4–145s, vision 74–117s, search_files 110s.
- Biaya: **$0.00** (model free) — tapi cache-read 16,4 juta token.

---

## Diagnosis

### A. Penyebab lambat
1. **Context 231k token dikirim penuh** tiap request (compression mati) → latency 4–25s, spike 111s.
2. **Compression gagal** karena tidak ada auxiliary provider (nous & Google dua-duanya 429).
3. **Vision auxiliary lambat**: 429 → cooldown 365–691s → fallback ke model utama.
4. **Hook `sync_logs.py`** menjalankan git push yang gagal di setiap terminal call.
5. **154 panggilan API** dalam satu sesi (loop tool-calling + retry).
6. Model free `longcat-2.5` intrinsik lambat.

### B. Penyebab kualitas jawaban kurang baik
1. **Model LongCat-2.5:free** — kelas model gratis, lebih lemah untuk coding kompleks.
2. **reasoning_effort medium** — kadang kurang dalam untuk tugas multi-langkah.
3. **Context membengkak tanpa ringkasan** → konteks awal terdorong keluar, noise menumpuk.
4. **429 berulang** → retry/fallback → jawaban tidak konsisten.
5. **62 tool deferred + system prompt 10–40k char** → beban kognitif tinggi.

### C. Penyebab agent gagal menyelesaikan tugas
1. **Compression mati** → risiko context overflow.
2. **Tanpa fallback provider** → auxiliary gagal total saat nous rate-limited.
3. **Hook sync_logs gagal** → noise berulang.
4. **Vision gagal/lambat** → analisis gambar tertunda.
5. **426 pesan dalam satu sesi** → konteks ekstrem.

---

## Rekomendasi (perubahan minimal, urut dampak)

### A. Perubahan konfigurasi

1. **Aktifkan prune deterministik (paling penting, tanpa LLM):**
   ```yaml
   compression:
     proactive_prune_tokens: 48000
   ```
   Ini memangkas tool-result lama secara deterministik — **tidak butuh auxiliary LLM**, langsung mengurangi context.

2. **Perbaiki hook `sync_logs.py`:** hapus/perbaiki `git push` yang gagal, atau matikan hook `post_tool_call` bila tidak kritis.

3. **Tambah fallback provider** untuk auxiliary:
   ```yaml
   fallback_providers:
     - "google/gemini-3.5-flash-lite"
   ```
   (Google key ada tapi sedang exhausted — alternatif: tambah OpenRouter key.)

4. **Set `model.context_length: 256000`** eksplisit agar tidak ada probing.

5. **Naikkan `reasoning_effort` ke `high`** untuk tugas coding kompleks (atau `low` untuk tugas sederhana).

### B. Perubahan prompt

1. Instruksi agent **meminimalkan screenshot berulang** dan tool call yang redundan.
2. Kurangi auto-load skill yang panjang; andalkan `tool_search` (sudah aktif).
3. Pisahkan instruksi "jangan bertele-tele" agar output lebih padat.

### C. Perubahan arsitektur agent

1. **Mulai sesi baru** untuk tugas baru — jangan tumpuk 400+ pesan dalam satu sesi.
2. **Gunakan `delegate_task`** untuk sub-tugas independen (parallelsasi, isolasi konteks).
3. **Pertimbangkan model yang lebih cepat/andal** untuk coding (mis. `gemini-3.5-flash` via Google key, atau model berbayar) — LongCat-2.5 free adalah bottleneck utama.

---

**Kesimpulan:** perubahan paling efektif dengan effort terkecil = **aktifkan `proactive_prune_tokens`** (langsung mengecilkan context) + **perbaiki hook sync_logs** (hilangkan latency per terminal call) + **tambah fallback provider** (agar compression/vision tidak mati). Ketiganya konfigurasi murni, tanpa mengubah kode.

Tidak ada file yang diubah, tidak ada paket dipasang, dan tidak ada rahasia yang ditampilkan.
