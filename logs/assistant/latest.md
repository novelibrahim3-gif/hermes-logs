Hasil pengujian pemetaan visual Craft World menggunakan MCP Buddy dan engine `craftworld-vision`:

### 1. Alur Graf (Graphify)
Buddy MCP -> screenshot (`/root/.hermes/cache/images/`) -> salin ke scratch (`step-00.jpg`, `step-01.jpg`) -> craftworld-vision MCP / `targets.py` (`find_objects`) -> `detection.py` (color mask, connected components, filtering, relative pairing, confidence scoring) -> hasil deteksi JSON.

### 2. Tabel Perbandingan Deteksi (Awal vs Setelah Swipe)

| Target / Resource / Icon | Jml Awal (Step-00) | Jml Setelah Swipe (Step-01) | Koordinat Awal (Center X, Y) | Koordinat Setelah Swipe (Center X, Y) | Confidence | Status |
|---|---:|---:|---|---|---:|---|
| **green_play** | 3 | 1 | (303, 120), (303, 440), (742, 223) | (605, 223) | 0.85 - 0.87 | Terdeteksi (sebagian keluar layar / bergeser) |
| **pink_x2** | 1 | 1 | (741, 320) | (604, 320) | 0.67 | Terdeteksi & Bergeser |
| **tank_body** | 9 | 7 | Beragam (357~1540) | Beragam (291~1540) | 1.00 | Terdeteksi (beberapa keluar/masuk viewport) |
| **tank** | 3 | 1 | (367, 113), (364, 434.5), (804, 261) | (668, 266) | 0.92 - 0.94 | Terdeteksi (kamera bergeser) |
| **earth_mine** | 1 | 1 | (804, 261) | (668, 266) | 0.92 - 0.94 | Terdeteksi & Bergeser |

### 3. Analisis & Kesimpulan
- **Pergeseran Posisi:** Koordinat objek berubah secara konsisten mengikuti pergeseran kamera (swipe ke atas/bawah menggeser elemen vertikal/horizontal).
- **Visibilitas:** Jumlah deteksi objek seperti `green_play` dan `tank` berkurang (dari 3 menjadi 1) karena sebagian objek keluar dari area *viewport* layar setelah digeser.
- **Konsistensi Engine:** Vision engine lokal (`targets.py` + `detection.py`) berhasil mengenali dan melacak target yang masih berada dalam jangkauan layar tanpa kehilangan akurasi confidence (tetap tinggi di kisaran 0.67 - 1.00).
