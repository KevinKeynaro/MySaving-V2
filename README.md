# MySaving

**Tahu kapan tabunganmu selesai, bukan kapan seharusnya selesai.**

MySaving adalah website pribadi untuk menghitung tabungan. Kamu menentukan target (misalnya sebuah handphone), mencatat setiap setoran, dan MySaving menghitung berapa yang sudah terkumpul, berapa yang tersisa, serta **kapan tabunganmu akan selesai** berdasarkan kecepatan menabungmu.

> Status: **dalam pengembangan** (MySaving V2, dibuat ulang dari nol).

## Apa bedanya?

Banyak aplikasi tabungan meminta kamu menentukan tenggat waktu, lalu memberi tahu berapa yang harus ditabung agar tepat waktu. MySaving sengaja tidak memakai deadline. Kamu menabung sebesar yang kamu mampu, dan MySaving menjawab satu pertanyaan sederhana: *kapan tabunganku selesai?*

**Contoh:** kamu menabung Rp 10.000 setiap hari untuk handphone seharga Rp 3.000.000. Sisa yang harus dikumpulkan dibagi setoran harian menghasilkan 300 hari, sehingga MySaving menampilkan perkiraan tanggal selesai. Setiap kali kamu menambah setoran, estimasi tanggalnya ikut diperbarui.

## Fitur

Fitur inti:

- Login untuk satu pengguna (tanpa halaman register)
- Kelola target tabungan (nama dan nominal target)
- Catat setoran (nominal, tanggal, catatan)
- Kalkulasi otomatis: terkumpul, sisa, persentase progres, dan estimasi tanggal selesai
- Dashboard ringkasan semua target dengan progress bar
- Dua bahasa: Indonesia dan Inggris

Direncanakan:

- Riwayat setoran per target
- Simulasi ("kalau setoranku dinaikkan, selesai kapan?")
- Grafik pertumbuhan tabungan
- Streak menabung

## Prinsip desain

- **Tanpa deadline.** Aplikasi hanya memperkirakan kapan target tercapai.
- **Hanya untuk dipakai sendiri.** Satu pengguna, tidak ada role atau admin.
- **Hanya Rupiah (IDR).** Semua nominal berupa bilangan bulat, tanpa konversi kurs.

## Tech stack

| Bagian | Teknologi |
|---|---|
| Frontend | Next.js, Tailwind CSS, lucide-react, next-intl |
| Backend | Node.js, Express.js |
| Database | MySQL, dikelola dengan migrations Knex |
| Autentikasi | JWT (cookie `httpOnly`), bcrypt |
| Validasi | Zod |

Seluruh kode ditulis dengan JavaScript (bukan TypeScript).

## Struktur proyek

```
MySaving/
├── backend/     API Express.js + migrations Knex
└── frontend/    aplikasi Next.js
```

## Menjalankan di komputer sendiri

### Prasyarat

- Node.js
- MySQL (misalnya lewat Laragon), dan opsional MySQL Workbench untuk melihat isi tabel

### 1. Clone repositori

```bash
git clone <url-repositori>
cd MySaving
```

### 2. Siapkan database

Buat database kosong bernama `mysaving` di MySQL.

### 3. Jalankan backend

```bash
cd backend
npm install
```

Buat file `.env` (lihat `.env.example`):

```
PORT=5000
DB_HOST=127.0.0.1
DB_USER=root
DB_PASSWORD=
DB_NAME=mysaving
JWT_SECRET=isi-dengan-string-acak-panjang
```

Jalankan migrations dan seed user, lalu nyalakan server:

```bash
npx knex migrate:latest
npx knex seed:run
npm run dev
```

### 4. Jalankan frontend

```bash
cd frontend
npm install
npm run dev
```

Buka `http://localhost:3000`.

## Cara kerja estimasi

| Hasil | Rumus |
|---|---|
| Terkumpul | jumlah semua setoran pada target |
| Sisa | target − terkumpul |
| Progres | terkumpul ÷ target × 100% |
| Estimasi selesai | tanggal hari ini + (sisa ÷ laju menabung) |

## Catatan

Repositori ini adalah **MySaving V2**. Versi pertama (HTML, Tailwind, JavaScript, dan localStorage tanpa akun dan database) dibuat sebagai proyek pribadi dan kini menjadi arsip.

## Lisensi

Proyek pribadi. Belum menggunakan lisensi tertentu.