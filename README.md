# 🎯 FocusGuard

> **Help you stay focused.** AI yang menegur kamu pakai video meme setiap kali ketahuan main HP.

FocusGuard memakai kamera laptop dan model deteksi objek **YOLO11** untuk mengenali HP di tanganmu secara realtime. Model AI-nya jalan **langsung di browser**, jadi video kamu tidak pernah dikirim ke server mana pun.

## ✨ Fitur

- 📷 **Deteksi realtime**: YOLO11n berjalan di browser lewat WebAssembly (onnxruntime-web)
- 🚨 **Teguran meme**: video meme langsung muncul saat kamu ketahuan main HP
- ⏱️ **Timer fokus**: menghitung berapa lama kamu bertahan tanpa HP
- 📊 **Riwayat & statistik**: durasi dan jumlah sesi main HP per hari
- 🔒 **Privasi**: video diproses di perangkatmu sendiri, dan riwayat dipisah per browser lewat cookie HttpOnly
- 🎚️ **Sensitivitas bisa diatur**: sesuaikan confidence threshold deteksi

## 🧠 Cara Kerja

```
Webcam ─▶ letterbox 640×640 ─▶ YOLO11n (ONNX, di browser) ─▶ NMS ─▶ ada "cell phone"?
                                                                        │
                                       ┌────────────────────────────────┤
                                       ▼                                ▼
                             overlay video meme            POST /api/sessions (Node.js)
                                                           disimpan per device token
```

1. **Preprocess**: frame webcam di-resize tanpa distorsi (letterbox) lalu diubah jadi tensor `[1, 3, 640, 640]`.
2. **Inference**: model mengeluarkan `[1, 84, 8400]`, yaitu 8400 kandidat kotak × (4 koordinat + 80 skor kelas COCO).
3. **Postprocess**: filter berdasarkan skor, lalu Non-Max Suppression. Kelas ke-67 (`cell phone`) memicu peringatan.
4. **Smoothing**: kamu dianggap main HP kalau HP terdeteksi di ≥60% dari 10 frame terakhir, supaya status tidak berkedip.

## 🛠️ Tech Stack

| Bagian | Teknologi |
|---|---|
| Frontend | Next.js 16, React 19, Tailwind CSS 4 |
| AI | YOLO11n (Ultralytics) → ONNX, onnxruntime-web |
| Backend | Next.js Route Handlers (Node.js) |
| Keamanan | Cookie `HttpOnly` + `SameSite`, token di-hash SHA-256, validasi input |

## 🚀 Menjalankan

**Butuh:** Node.js 20+ dan Python 3.9+ (Python hanya untuk export model, sekali saja)

```bash
git clone https://github.com/bubibu12212/NoPhoneZone.git
cd NoPhoneZone
npm install
```

Export model YOLO ke ONNX:

```bash
pip install ultralytics
python scripts/export_model.py
```

Tambahkan video meme favoritmu di `public/memes/warning.mp4`, lalu jalankan:

```bash
npm run dev
```

Buka [http://localhost:3000](http://localhost:3000) dan klik **Mulai Fokus**.

## 📁 Struktur Project

```
app/
├── page.tsx                  # halaman utama
├── components/PhoneDetector.tsx  # webcam, loop deteksi, UI
└── api/sessions/route.ts     # API riwayat (GET / POST)
lib/
├── yolo.ts                   # preprocess, inference, postprocess YOLO
└── device.ts                 # device token via cookie HttpOnly
scripts/
├── export_model.py           # export YOLO11n .pt → .onnx
└── copy-ort-wasm.mjs         # copy file WebAssembly onnxruntime ke public/
public/memes/warning.mp4      # video meme peringatan
```

## ⚙️ Kustomisasi

- **Ganti video meme**: taruh file di `public/memes/`, lalu ubah `MEME_VIDEO` di `app/components/PhoneDetector.tsx`.
- **Ubah kepekaan**: atur `WINDOW` dan `ON_RATIO` di file yang sama.

## 📄 Lisensi

Model YOLO11 dari [Ultralytics](https://github.com/ultralytics/ultralytics) memakai lisensi **AGPL-3.0**.
