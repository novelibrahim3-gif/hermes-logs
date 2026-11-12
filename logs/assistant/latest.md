HERMES — PEMERIKSAAN AWAL (tanpa tindakan Android)

1. .hermes.md Craft World
Status: DIMUAT. 579 baris / 15.473 byte, terbaca utuh sebagai project context.

Ringkasan aturan yang saya pahami:
- Hierarki info: screenshot aktual > analisis visual > Craftworld Vision > OCR > screenshot lama (historis). Kalau bukti bertentangan: screenshot baru, berhenti sebelum tap berisiko.
- Koordinat lama (basis 720x1600) hanya referensi. Wajib cek resolusi, cari ulang tombol per bentuk/warna/posisi relatif, konversi skala, pastikan tidak tertutup panel.
- Maks 4 tap per baris phone_do_steps; batch besar hanya setelah target dan urutan pasti.
- Setiap tindakan penting diverifikasi lewat kondisi layar SESUDAH tindakan.
- Larangan mutlak: Install/Get/Confirm/pembelian; mengganti restart pink dengan play hijau; menganggap iklan selesai tanpa bukti reward; menyentuh STASHED; menyentuh "ADD +4h"/panel pembelian; membatalkan jadwal bukan milik workflow; membuat jadwal tanpa timer valid; mengarang nama mesin, angka, timer, atau status.
- Tahapan siklus: 0 buka Craft World -> 1 baca kondisi + kumpul pile >20k -> 2 restart Earth Mine (pink x2, tunggu reward) -> 3 verifikasi Earth Mine (bandingkan pile; RUNNING/STOPPED/UNKNOWN) -> 4 Boost x2 -> 5 rantai mud->clay->sand->copper->silver -> 6 STASHED tidak disentuh -> 7 penjadwalan (list_scheduled -> min(timer Earth Mine, timer Boost) + 2 menit -> buat -> verifikasi list lagi) -> 8 simpan status -> 9 Home Screen (hanya bila jadwal terverifikasi).
- Mekanisme verifikasi: screenshot baru pasca-tindakan; pile berkurang / resource bertambah; teks "Reward granted" sebelum menutup iklan; list_scheduled setelah membuat jadwal; status UNKNOWN bila bukti kurang. Prinsip: screenshot aktual mengalahkan koordinat lama, bukti visual mengalahkan asumsi, verifikasi mengalahkan klaim.

2. Tool Buddy MCP
- Konfigurasi: ada dan enabled — url http://127.0.0.1:8765/mcp, header Bearer (BUDDY_TOKEN tersedia di env, panjang 48).
- Status server: TIDAK BERJALAN. Port 8765 tidak listening, tidak ada proses buddy, curl ditolak (exit 7, connection refused).
- Akibat: tool mcp__buddy__* (open_app, read_screen, screenshot, tap, swipe, press, wait, notify, ask, dll.) tidak bisa dipanggil; tool_search di sesi ini juga tidak menemukannya.
- buddy-run tersedia di /root/buddy-android/linux/buddy-run (usage: buddy-run <token> ask|stop|delete|prune|ping). Server harus di-start dulu sebelum siklus device bisa berjalan. Belum saya start, sesuai instruksi.

3. Tool scheduling (tersedia, schema terverifikasi via tool_describe)
- cronjob_manage — action: create/list/update/pause/resume/remove/run/cancel. Parameter: schedule (wajib saat create; bentuk "in 30m", "every 2h", cron "0 9 * * *", atau ISO one-shot), prompt (self-contained), skills, repeat, deliver, failure_deliver, script, monitor, no_agent, continuity, context_from, enabled_toolsets, workdir, timezone, pinned, attach_to_session, catch_up. job_id wajib untuk semua aksi selain create/list.
- todo_list — todos: [{id, content, status: pending/in_progress/completed/cancelled, parent}], merge: true/false.

4. Tool craftworld-vision (tersedia, 7 tool)
- analyze_screenshot(image_path) — resolusi, fitur ORB, OCR.
- compare_screenshots(image1, image2) — ORB/RANSAC + gambar hasil.
- find_objects(image_path, target) — deteksi green_play, pink_x2, tank_body, tank, earth_mine.
- get_prompt(name, arguments), list_prompts(), list_resources(), read_resource(uri) — utilitas MCP.

5. Gap penting
- Script grab.sh, ocr.sh, wake.sh yang direferensikan .hermes.md TIDAK DITEMUKAN di filesystem (find di /root, /usr/local, /opt). Tanpa itu Tahap 0/1 tidak punya alat capture lokal; capture harus lewat Buddy MCP selama server hidup.

6. Tindakan yang dilakukan
- Tidak ada tap, tidak ada pembatalan jadwal, tidak ada jadwal dibuat. Hanya baca file, probe HTTP read-only, dan tool_describe.

Kesimpulan: aturan Craft World sudah dimuat dan dipahami; tool scheduling dan craftworld-vision siap; Buddy MCP belum berjalan sehingga semua aksi device belum bisa dieksekusi. Sebutkan bila ingin saya start server Buddy atau membaca daftar jadwal yang ada (read-only).
