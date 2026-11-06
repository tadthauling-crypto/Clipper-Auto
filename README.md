# Clipper Otomatis (Android)

Aplikasi Android yang mengubah video panjang menjadi klip vertikal 9:16, sesuai PRD:

- **Hook otomatis (F2)** – mencari momen dengan energi audio tertinggi (suara keras, tawa, teriakan), lalu membuka klip tepat di 3 detik terkuatnya. Mode *Cold open* memutar momen itu dulu, baru konteksnya.
- **Tempo cepat (F3)** – jeda diam dipotong otomatis (preset Natural / Standar / Cepat) dan kata pengisi seperti "eee", "hmm" dibuang bila transkripsi aktif.
- **Caption per kata (F4)** – caption besar 1–3 kata, kata yang sedang diucapkan disorot (gaya Bold Pop, Box, Minimal).
- **Format platform (F5)** – output MP4 H.264 1080×1920, tersimpan di Galeri › Movies/ClipperOtomatis.

Semua pemrosesan video berjalan di HP. Hanya caption yang membutuhkan internet, karena audio klip dikirim ke API transkripsi Whisper.

## Build APK lewat GitHub (tanpa Android Studio)

1. Buat repository baru di GitHub (boleh private).
2. Unggah seluruh isi folder ini ke repository, termasuk folder `.github`.
   - Cara termudah: **GitHub Desktop** atau perintah git:
     ```bash
     git init
     git add .
     git commit -m "Clipper Otomatis v0.1"
     git branch -M main
     git remote add origin https://github.com/USERNAME/NAMA-REPO.git
     git push -u origin main
     ```
   - Kalau unggah lewat web (Add file › Upload files), folder `.github` sering tidak ikut karena tersembunyi. Buat manual lewat **Add file › Create new file**, beri nama `.github/workflows/build-apk.yml`, lalu tempel isi file tersebut.
3. Buka tab **Actions**. Workflow **Build APK** berjalan otomatis setiap ada push (sekitar 5–8 menit). Bisa juga dijalankan manual lewat tombol **Run workflow**.
4. Setelah selesai (centang hijau), buka run tersebut, lalu unduh artifact **ClipperOtomatis-debug-apk**. Isinya file ZIP berisi `app-debug.apk`.
5. Pindahkan APK ke HP, buka, dan izinkan "Instal dari sumber tidak dikenal".

Ingin APK muncul di halaman **Releases**? Buat tag versi:
```bash
git tag v0.1.0
git push origin v0.1.0
```

Kalau build gagal, buka langkah yang merah di tab Actions, salin pesan errornya, dan perbaiki (atau tempel ke Claude).

## Mengaktifkan caption

Buka **Pengaturan transkripsi** di aplikasi dan isi:

| Penyedia | Base URL | Model |
| --- | --- | --- |
| OpenAI | `https://api.openai.com/v1` | `whisper-1` |
| Groq | `https://api.groq.com/openai/v1` | `whisper-large-v3-turbo` |

API key disimpan hanya di HP. Tanpa API key, klip tetap dibuat (hook, potong jeda, 9:16) tetapi tanpa caption.

## Struktur kode

| File | Isi |
| --- | --- |
| `AudioAnalyzer.kt` | Decode audio ke PCM, hitung loudness per 50 ms |
| `ClipPlanner.kt` | Skor momen, pilih hook, potong jeda dan kata pengisi |
| `Transcriber.kt` | Panggilan API transkripsi dengan timestamp per kata |
| `CaptionOverlay.kt` | Caption beranimasi per kata (template caption) |
| `ClipRenderer.kt` | Potong, crop 9:16, burn-in caption dengan Media3 Transformer |
| `MediaSaver.kt` | Simpan ke galeri, putar, bagikan |
| `MainViewModel.kt` / `MainActivity.kt` | Alur proses dan antarmuka (Jetpack Compose) |

## Batasan versi 0.1

- Crop 9:16 masih di tengah frame; pelacakan wajah (F5.2) belum ada.
- Pemilihan hook memakai sinyal audio; skor berbasis LLM dan visual (F2.3) belum ada.
- Biarkan aplikasi tetap terbuka saat memproses; layar dijaga tetap menyala.
- Butuh Android 10 (API 29) atau lebih baru.
