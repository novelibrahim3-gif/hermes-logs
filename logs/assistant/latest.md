Masih gagal — root belum aktif.

- root_status: KernelSU: NOT GRANTED (sama seperti sebelumnya)
- screenshot: "KernelSU/su could not be started"
- swipe: UNAVAILABLE

Craft World tetap bisa dibuka, tapi tanpa root semua gesture/screenshot Buddy mati.

Kemungkinan penyebab:
1. Grant diberikan ke aplikasi yang salah. Di KernelSU, buka tab Superuser — pastikan yang diizinkan adalah Buddy/bantuan root-nya (bukan cuma app Buddy biasa).
2. Ada dialog superuser yang belum kamu setujui (cek notifikasi KernelSU).
3. Service root Buddy perlu di-restart setelah grant.

Tolong cek KernelSU > Superuser, izinkan Buddy/root helper, lalu bilang "ulangi". Saya cek ulang root_status dan lanjut scroll kalau sudah NOT GRANTED hilang.

Sudah saya push: commit b5e4250.
