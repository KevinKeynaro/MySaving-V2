# MySaving V2 — Backend

Panduan untuk AI CLI. Baca file ini sebelum menulis atau mengubah kode di folder `backend/`.

## Ringkasan proyek

MySaving adalah website pribadi untuk menghitung tabungan. Pengguna membuat **target tabungan** (misalnya handphone Rp 3.000.000) dan mencatat **setoran**. Aplikasi menghitung progres dan **estimasi kapan tabungan selesai**.

Aturan produk yang TIDAK boleh dilanggar:
- **Tidak ada deadline.** Jangan menambah kolom, field, atau logika deadline. Aplikasi hanya menghitung kapan tabungan akan selesai berdasarkan laju menabung, bukan kapan seharusnya selesai.
- **Single-user.** Tidak ada register, tidak ada role, tidak ada admin. User dibuat lewat seed.
- **Mata uang hanya IDR.** Tidak ada kolom `currency`, tidak ada konversi kurs.
- **Nominal selalu bilangan bulat** (`BIGINT UNSIGNED`), karena Rupiah tidak memakai sen. Jangan memakai float atau DECIMAL.

## Tech stack

- Bahasa: **JavaScript** (bukan TypeScript), ES Modules (`"type": "module"`, gunakan `import`/`export`).
- Runtime: Node.js + **Express.js**.
- Database: **MySQL** (dijalankan lewat Laragon, dilihat lewat MySQL Workbench).
- Query builder dan migrations: **Knex**.
- Frontend (folder terpisah): Next.js.

## Library yang sudah terpasang di backend

Dependencies:

| Library | Fungsi |
|---|---|
| `express` | framework server |
| `mysql2` | driver MySQL, dipakai oleh Knex |
| `knex` | query builder, migrations, seeds |
| `dotenv` | membaca `.env` |
| `cors` | mengizinkan frontend Next.js memanggil API |
| `zod` | validasi body, params, dan query request |
| `jsonwebtoken` | membuat dan memverifikasi JWT |
| `bcrypt` | hash password |
| `cookie-parser` | membaca cookie (JWT disimpan di cookie `httpOnly`) |

DevDependencies:

| Library | Fungsi |
|---|---|
| `nodemon` | restart otomatis saat development |

Library di luar daftar ini **belum terpasang**. Jangan mengimpor atau menambah library baru tanpa persetujuan pengguna. Khususnya, jangan memakai: Prisma, Sequelize, TypeORM, Mongoose, Joi, Yup, `express-validator`, `axios`, atau TypeScript.

## Script

```json
"scripts": {
  "dev": "nodemon src/app.js",
  "start": "node src/app.js"
}
```

Perintah Knex:

```bash
npx knex migrate:latest
npx knex migrate:rollback
npx knex migrate:status
npx knex seed:run
```

## Struktur folder

```
backend/
├── src/
│   ├── controllers/     auth, goals, deposits
│   ├── db/
│   │   ├── index.js     instance Knex (diekspor, dipakai semua controller)
│   │   ├── migrations/
│   │   └── seeds/
│   ├── middlewares/     authMiddleware, validate, errorHandler
│   ├── routes/          auth, goals, deposits
│   ├── schemas/         skema Zod
│   ├── services/        calculator.js (rumus kalkulasi)
│   └── app.js
├── .env
├── .env.example
├── knexfile.js
└── package.json
```

## Konvensi kode

- Akses database hanya lewat instance Knex dari `src/db/index.js`. Jangan membuat koneksi `mysql2` terpisah.
- Validasi input dengan Zod melalui middleware `validate`, bukan di dalam controller.
- Semua rumus kalkulasi (terkumpul, sisa, progres, estimasi selesai) ada di `src/services/calculator.js`, bukan di controller.
- Route selain login wajib melewati `authMiddleware` (verifikasi JWT).
- Error ditangani terpusat di `errorHandler`.
- Rahasia (kredensial DB, `JWT_SECRET`) hanya dari `.env`. Jangan menulisnya langsung di kode. `.env` tidak boleh di-commit.

## Skema database (ringkas)

- `users`: id, email, password_hash, locale (`id` | `en`), created_at
- `goals`: id, user_id, name, target_amount, plan_amount (nullable), plan_frequency (`daily` | `weekly` | `monthly`, nullable), status (`active` | `completed` | `archived`), timestamps
- `deposits`: id, goal_id, amount, deposited_at, note (nullable), created_at

Urutan migration karena foreign key: users, goals, deposits.

## Rumus

- terkumpul = jumlah semua setoran pada target
- sisa = target_amount − terkumpul
- progres (%) = terkumpul ÷ target_amount × 100
- sisa periode = sisa ÷ plan_amount, dibulatkan ke atas
- estimasi selesai = tanggal hari ini + sisa periode (dalam hari, minggu, atau bulan sesuai `plan_frequency`)

## Bahasa

Teks antarmuka mendukung bahasa Indonesia (`id`) dan Inggris (`en`). Pesan error dari API sebaiknya berupa kode atau key agar bisa diterjemahkan di frontend.