Hasil tes aktual (tanpa asumsi):

1. root_status
   Root MCP: responding.
   Accessibility: OFF.
   KernelSU: NOT GRANTED; open KernelSU and authorize Buddy.
   Display: ON; keyguard: UNLOCKED.
   Approval notifications: NOT READY.
   Display over other apps: not allowed.
   Install from Buddy: not allowed.
   Background helper paired: no; Phone commands setting: off; Linux tools: unknown.

2. Screenshot
   Root tidak tersedia, jadi saya tetap mengambil screenshot sekali untuk membuktikan kondisi nyata: GAGAL.
   Output: "Error: KernelSU/su could not be started. Grant Buddy root access in KernelSU, then retry."

3. Status screenshot
   Tidak berhasil. Tidak ada gambar yang dihasilkan, jadi tidak ada ukuran gambar.

4. Tidak ada swipe, tap, atau aksi lain — dipatuhi.

5. Tidak ada konfigurasi atau file yang diubah — dipatuhi. Karena ini melarang perubahan file, push otomatis ke repo saya lewati untuk jawaban ini saja.

Kesimpulan: izin root belum efektif. KernelSU masih melaporkan NOT GRANTED untuk Buddy. Perlu dicek di aplikasi KernelSU (tab Superuser) apakah Buddy/root helper benar-benar terdaftar dan diizinkan, lalu restart service Buddy.
