Selesai. Ringkasan:

## 1. File konfigurasi aktif
`/root/.hermes/config.yaml` (HERMES_HOME=`/root/.hermes`)

## 2. Verifikasi dukungan (v0.23.0)
`compression.proactive_prune_tokens` **didukung dan terverifikasi di source**:
- `hermes_cli/config_defaults.py:602` → `"proactive_prune_tokens": 0`
- `agent/agent_init.py:1463` → dibaca dari config `compression`
- `agent/context_compressor_prune.py:50-51` → trigger saat `current_tokens >= proactive_prune_tokens`
- `gateway/run.py:4115` → terdaftar sebagai config key resmi

**Arti 48000:** prune deterministik (tanpa LLM) diaktifkan; saat context mencapai 48.000 token, tool-result lama dipangkas (dedup, ringkas tool result besar, buang gambar lama), dengan tail 20 pesan dilindungi. Commit hanya jika reclaim ≥ 4096 token.

## 3. Backup
`/root/.hermes/config.yaml.bak-20261111-0305` (1254 byte, identik dengan file sebelum perubahan)

## 4. Perubahan (hanya 1 key)
```
compression:
  proactive_prune_tokens: 48000
```
Ditambah via `hermes config set compression.proactive_prune_tokens 48000` (jalur resmi; patch tool menolak config sensitif).

## 5. Nilai sebelum & sesudah

| Key | Sebelum | Sesudah |
|---|---|---|
| `compression.proactive_prune_tokens` | tidak ada (default **0**) | **48000** |

Diff vs backup hanya menambahkan 2 baris `compression:` + `proactive_prune_tokens: 48000`. Tidak ada key lain berubah.

## 6. Validasi
- `hermes config get compression.proactive_prune_tokens` → **48000** ✓
- `hermes config check` → Config version 52 ✓, tidak ada error
- YAML parse (node yaml) → **valid** ✓
- `agent`, `model`, `hooks`, `mcp_servers`, `providers` → tidak berubah ✓

## Catatan
Perubahan berlaku pada **sesi baru** (`/reset` atau session baru) — agent sesi ini sudah ter-init sebelumnya, jadi prune akan aktif mulai sesi berikutnya.
