# Laomera — Aplikasi Web Deteksi Kesehatan Kulit Wajah Berbasis AI

## Identitas

| | |
|---|---|
| **Nama** | Robi Wahyudi |
| **NIM** | `41257156` |
| **Kelompok** | `Cangkol Perangkat Lunak` |
| **Tema Topik Kelompok** | `aplikasi kesehatan wajah berbasis web` |
| **Mata Kuliah** | Rekayasa Perangkat Lunak (TIKB2503021), Semester 3 |
| **Dosen Pengampu** | Yudhistira Arie Wijaya, M.Kom., CDS. |
| **Perguruan Tinggi** | STMIK IKMI Cirebon |

## Deskripsi Aplikasi

**Laomera** (sebelumnya bernama SkinCheck AI) adalah aplikasi web yang
membantu pengguna memahami kondisi kesehatan kulit wajahnya sendiri secara
personal dan non-invasif — sebagai alat bantu edukasi dan indikasi visual
awal, **bukan alat medis dan bukan pengganti konsultasi dokter kulit**.

Pengguna cukup memakai kamera HP atau laptop untuk scan wajah. Sistem
menganalisis foto wajah dan menampilkan skor untuk lima kondisi kulit,
menyimpan riwayat scan untuk dibandingkan dari waktu ke waktu, menjawab
pertanyaan perawatan kulit lewat chatbot AI, dan merekomendasikan produk
perawatan lewat marketplace.

## Fitur Utama

- **Face Scan multi-pose (12 frame)** — kamera memandu 6 pose (depan,
  kanan, kiri, atas, bawah, close-up); wajah dideteksi dengan YuNet,
  hanya frame yang lolos deteksi yang dianalisis.
- **Analisis 5 kondisi kulit (AI)** — model VGG16 (`finetuned_best.h5`)
  menghasilkan skor severity 0–100 untuk jerawat, kulit kering, kulit
  berminyak, dark spots (flek hitam), dan kerutan; skor kesehatan
  keseluruhan = 100 − rata-rata kelima skor. Inferensi berjalan via Modal.
- **Laomera Asisten (chatbot AI)** — provider Gemini
  (`gemini-3.5-flash-lite`), dengan Groq sebagai provider
  alternatif. Chatbot membaca database langsung (tanpa angka hardcoded),
  dilengkapi guardrail anti-karang dan toleran terhadap typo.
- **Verifikasi akun OTP WhatsApp** — alur register → pending → OTP
  WhatsApp 6 digit (Fonnte) → isi biodata → active. Login customer yang
  belum terverifikasi dibatasi sampai verifikasi selesai.
- **Notifikasi WhatsApp otomatis** — OTP verifikasi dan "Hasil scan siap"
  saat analisis scan selesai.
- **Dashboard & Riwayat** — dashboard dan chatbot memakai **rata-rata
  semua scan** pengguna; riwayat scan dapat dibandingkan dengan scan
  sebelumnya (membaik/memburuk per kondisi).
- **Marketplace** — katalog produk perawatan kulit dengan rekomendasi
  produk aktif per kondisi dan tautan marketplace.
- **Panel Admin** — kelola pengguna (termasuk status verifikasi),
  produk, audit scan, tren scan, pengaturan AI, dan monitoring percakapan
  chatbot.
- **CI/CD** — GitHub Actions berjalan otomatis di repo backend dan
  frontend pada setiap push / pull request ke `main`.

## Arsitektur & Teknologi

| Lapisan | Teknologi |
|---|---|
| Frontend | Next.js 14.2 + React 18 (deploy Vercel) |
| Backend | FastAPI (Python, deploy Vercel) |
| Database | TiDB (MySQL-compatible, TLS) |
| Inferensi AI | Modal (VGG16), YuNet (deteksi wajah) |
| Chatbot AI | Gemini / Groq |
| OTP & notifikasi WA | Fonnte |
| CI/CD | GitHub Actions |

## Aplikasi Live

- **Frontend (produksi):** https://skincheck-frontend-robimhmmd-uxs-projects.vercel.app
- **Backend (produksi):** https://skincheck-backend-robimhmmd-uxs-projects.vercel.app
- **Backend repo:** https://github.com/robimhmmd-ux/proyek-rpl-laomera-backend
- **Frontend repo:** https://github.com/robimhmmd-ux/proyek-rpl-laomera-frontend

> Pakai **URL produksi di atas**. Jangan pakai URL deployment lama
> ber-hash (preview) — URL itu bisa kedaluwarsa atau menunjuk build lama.

## Video Presentasi

| Video | Tautan |
|---|---|
| Video Demonstrasi — gabungan Progres 1 (PRD & Konsep, 3–5 mnt) + Progres 2 (UML & Arsitektur, 5–7 mnt) + Final (Live Demo, 8–12 mnt), total maks. 24 menit | https://youtu.be/G1qA7cOC3JM |

Video diunggah ke YouTube (Unlisted/Public) atau Google Drive
(Anyone with the link), dengan webcam aktif dan demo live real-time.
Perekaman boleh berjeda antar segmen, hasil akhirnya satu video.

## Petunjuk Instalasi (Setup Instruction)

### Prasyarat

- Node.js 18+ dan npm
- Python 3.11+ dan pip (CI memakai Python 3.12)
- Akun TiDB Cloud (database `skincheck_db`) — atau MySQL lokal
- Kunci API Gemini (untuk chatbot) dan token Fonnte (untuk OTP WhatsApp,
  bisa diatur lewat `.env` atau lewat pengaturan admin — bukan lewat chat)

### 1. Clone kedua repo

```bash
git clone https://github.com/robimhmmd-ux/proyek-rpl-laomera-backend.git
git clone https://github.com/robimhmmd-ux/proyek-rpl-laomera-frontend.git
```

### 2. Backend (FastAPI)

```bash
cd proyek-rpl-laomera-backend
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env            # lalu isi .env (lihat tabel di bawah)
python setup_db.py              # membuat tabel (tanpa akun demo — daftar lewat register)
python run.py                   # http://127.0.0.1:8000
# atau: uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Isi `.env` (jangan pernah commit file ini):

| Variabel | Keterangan |
|---|---|
| `SECRET_KEY` | String acak panjang (wajib; jangan pakai default) |
| `DB_CONNECTION` | `mysql` |
| `DB_HOST` | Host TiDB Cloud, atau `127.0.0.1` untuk lokal |
| `DB_PORT` | `4000` (TiDB Cloud) / `3306` (MySQL lokal) |
| `DB_DATABASE` | `skincheck_db` |
| `DB_USERNAME` / `DB_PASSWORD` | Kredensial database |
| `DB_SSL` | `true` untuk TiDB Cloud, `false` untuk lokal |
| `FRONTEND_URL` | Origin frontend, mis. `http://localhost:3000` |
| `SESSION_SAME_SITE` | `lax` untuk lokal, `none` untuk produksi lintas situs |
| `GEMINI_API_KEY` | Kunci API Gemini (untuk chatbot) |
| `FONNTE_TOKEN` | Token Fonnte untuk OTP WhatsApp (alternatif: isi lewat pengaturan admin) |

Cek backend hidup: buka `http://127.0.0.1:8000/api/v2/health`
(harus mengembalikan status sehat).

Catatan:

- File model `.h5` (174 MB) **tidak ada di repo** dan tidak dibutuhkan
  untuk menjalankan API — inferensi AI berjalan via Modal.
- Untuk seeding katalog produk saja: `python seed_products.py`.

### 3. Frontend (Next.js)

```bash
cd proyek-rpl-laomera-frontend
npm install
cp .env.example .env.local
# isi NEXT_PUBLIC_API_URL di .env.local dengan URL backend
# (lokal: http://127.0.0.1:8000 — tanpa trailing slash)
npm run dev                     # http://localhost:3000
```

Build produksi:

```bash
npm run build
npm run start
```

Deploy Vercel: import repo frontend sebagai project Vercel baru, isi
environment variable `NEXT_PUBLIC_API_URL` dengan URL backend produksi,
lalu deploy. Jangan commit `.env.local`.

### 4. Verifikasi instalasi

1. Buka `http://localhost:3000` — landing page Laomera tampil.
2. Register akun baru → cek OTP WhatsApp masuk → isi biodata → login.
3. Lakukan satu scan wajah → hasil 5 skor + skor keseluruhan tampil.
4. Tanya Laomera Asisten, mis. kondisi kulit berdasarkan hasil scan.
5. Login sebagai admin → cek dashboard admin, daftar pengguna, dan
   pengaturan AI.

## Pengujian & CI/CD

- Unit test backend: 37 test (`pytest`, lihat folder `tests/`).
  Jalankan lokal: `PYTHONPATH=$PWD python -m pytest tests/ -q`
- Unit test frontend: 17 test (`vitest`, lihat `lib/scores.test.js`).
  Jalankan lokal: `npm test`
- Pipeline GitHub Actions (`.github/workflows/ci.yml`) berjalan di kedua
  repo pada setiap push / pull request ke `main`: backend menjalankan
  pytest, frontend menjalankan unit test + build Next.js.
- Pengujian manual end-to-end (register → OTP → scan → chatbot →
  marketplace → admin) dilakukan terhadap URL produksi sebelum video
  demo direkam.

## Dokumen Terkait

- Panduan proyek: *Panduan Tugas Proyek Akhir Mandiri & Rubrik
  Penilaian OBE, RPL (TIKB2503021)* v4 — STMIK IKMI Cirebon.
- Dokumen PRD/SRS, SDD (UML + UI/UX), dan Laporan Pengujian/CI-CD
  digabung dalam satu berkas PDF dan diunggah ke portal LMS (tenggat:
  Minggu ke-16 / UAS).
- Dokumentasi teknis backend (pemisahan dari monolit, API v2):
  [`docs/BACKEND-TECHNICAL.md`](docs/BACKEND-TECHNICAL.md)

## Lisensi

Proyek akhir mata kuliah — STMIK IKMI Cirebon, 2026.
