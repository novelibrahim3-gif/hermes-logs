Selesai. Satu bug di script asli, sudah aku perbaiki.

BUG
`hermes -z --toolsets terminal ...` tidak valid: `-z` butuh nilai prompt, tapi diikuti `--toolsets`.
Bukti (parse test): `hermes: error: argument -z/--oneshot: expected one argument`, exit 2.
Urutan yang benar: `hermes -z "PROMPT" --toolsets terminal --usage-file "$USAGE"`.
Script asli akan gagal total (HERMES_EXIT=2, usage file tidak dibuat, nested hermes tidak jalan). Versi corrected ada di /tmp/hermes_e2e_test.sh.

HASIL TEST (setelah diperbaiki)
TEST_ID=HERMES_E2E_20261111_115742
- Nested hermes: HERMES_EXIT=0
- Terminal dipakai tepat 1x: `printf '%s\n' 'HERMES_E2E_20261111_115742'` -> output `HERMES_E2E_20261111_115742`, exit_code 0
- Jawaban assistant persis: `ASSISTANT_HERMES_E2E_20261111_115742`

USAGE REPORT (/tmp/HERMES_E2E_20261111_115742-usage.json)
- total_tokens 17.184 (in 16.987, out 197, cache_read 8.320, reasoning 126)
- api_calls 2 (+1 auxiliary title_generation = 235 tok)
- model meituan/longcat-2.5:free, provider nous
- completed=true, failed=false, turn_exit_reason=text_response(finish_reason=stop)

VERIFIKASI LOG END-TO-END
- logs/tool-results/calls-20261111.jsonl: tercatat pemanggilan terminal nested (status ok, 2636 ms, result berisi TEST_ID)
- logs/assistant/responses-20261111.jsonl: tercatat `ASSISTANT_HERMES_E2E_20261111_115742`
- latest.txt dan latest.md ter-update
- Push: commit 6a6dc4fa, local HEAD = remote HEAD = 6a6dc4fa098f8c2f7d445d6ebfa156ab759e77d4
- sync-errors.log: kosong (tidak ada kegagalan push)

File proyek tidak diubah. Status juga sudah dikirim ke HP via Buddy notify.
