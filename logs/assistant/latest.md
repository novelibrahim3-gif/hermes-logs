Selesai. Kedua skill diperbarui dan terverifikasi dimuat.

File yang berubah (7):
1. buddy-mobile-control/SKILL.md — v1.2.0 → v1.3.0
2. buddy-mobile-control/references/rewarded-ad-flow.md — ditulis ulang
3. buddy-mobile-control/references/scheduled-automation.md — opsi jadwal Velnor
4. buddy-mobile-control/references/mcp-http-api.md — read_screen = gambar, chat id
5. craftworld-visual-inspector/SKILL.md — v1.5.0 → v1.6.0
6. craftworld-visual-inspector/references/rewarded-ad-handling.md — aturan monitor iklan
7. craftworld-visual-inspector/references/vision-routing.md — tool Velnor

Ringkasan perubahan:
A. Kontrol Android (buddy): foreground check dulu, screenshot terbaru sebagai dasar, target dari kondisi aktual bukan koordinat lama, satu tindakan lalu verifikasi, tool sukses ≠ tujuan tercapai, pemulihan bertahap, dan kegagalan root (su/cmd sandbox).
B. Rewarded ads (keduanya): jangan asumsikan jumlah/durasi/situs/posisi tombol; tunggu countdown; pahami tombol ">>" sebelum menekan; jangan tekan X sebelum terlihat; pastikan game kembali; bila redirect Play Store → Home + buka ulang + verifikasi; pastikan reward benar-benar masuk.
C. Earth Mine: pemicu = tombol x2 terlihat ATAU timer habis; verifikasi peti aktif + badge x2 + glow pink + timer tepat di bawah peti; bukti tambahan laju produksi 2x.
D. Boost: ikuti jumlah aktivasi yang diminta, cek sisa waktu dulu, berhenti setelah tercapai, jangan menambah bila waktu sudah banyak.
E. Penjadwalan: baca kedua timer, pilih yang habis duluan + margin 2 menit, buat jadwal Velnor (?chat=<id>) atau cronjob_manage, verifikasi tersimpan & aktif.
Plus perbaikan kontradiksi lama: aturan ×1.5 (Buddy) vs 1:1 (Velnor) ditandai eksplisit, referensi mcp__buddy__* → mcp__velnor__*, dan read_screen sebagai gambar (bukan daftar elemen).

Lokasi backup: /root/.hermes/cache/scratch/skill-backup-2026-11-14/ (buddy-mobile-control + craftworld-visual-inspector, sebelum edit).

Bukti verifikasi:
- skill_view buddy-mobile-control → success, version 1.3.0, bagian "Verified control flow (14 Nov 2026)" dan "Penjadwalan" termuat.
- skill_view craftworld-visual-inspector → success, version 1.6.0, bagian "Kondisi pemicu", "Bukti tambahan", "Ikuti jumlah aktivasi", dan "Penjadwalan (Earth Mine + Boost)" termuat.
- diff -rq backup vs live → tepat 7 file berbeda, sesuai daftar di atas.
- Tidak ada instruksi bertentangan; referensi Buddy tetap berlaku untuk lingkungan Buddy, Velnor untuk sesi ini.

Catatan: sempat ada kejadian path discovery skill_manage memilih salinan skill di dalam skills root, jadi hasil edit saya pindahkan ke lokasi live dan backup asli disimpan di luar skills root. Tidak ada tindakan pada HP dan tidak ada perubahan kondisi game selama proses ini. Log + commit 66e7ec5 di repo craftworld-vision.
