# MySaving V2 — Backend

Panduan untuk AI CLI. Baca file ini sebelum menulis atau mengubah kode di folder `backend/`.

## Ringkasan proyek

MySaving adalah website pribadi untuk menghitung tabungan. Pengguna membuat **target tabungan** (misalnya handphone Rp 3.000.000), menetapkan **komitmen harian** (misalnya Rp 10.000 per hari), lalu mencatat **setoran** dan **pengeluaran** dari tabungan. Aplikasi menghitung saldo dan **estimasi kapan tabungan selesai**, dan menghitung ulang setiap kali ada transaksi baru.

## Aturan produk yang TIDAK boleh dilanggar

- **Tanpa deadline.** Jangan menambah kolom, field, atau logika deadline. Aplikasi hanya menghitung kapan tabungan akan selesai berdasarkan komitmen harian, bukan kapan seharusnya selesai.
- **Estimasi memakai komitmen harian** (`daily_amount`) yang diisi pengguna. Jangan menghitung dari rata-rata riwayat setoran.
- **`daily_amount` wajib diisi** pada setiap target dan harus lebih dari 0.
- **Hanya komitmen harian.** Tidak ada frekuensi mingguan atau bulanan.
- **Single-user.** Tidak ada register, role, atau admin. User dibuat lewat seed.
- **Mata uang hanya IDR.** Tidak ada kolom `currency`, tidak ada konversi kurs.
- **Nominal selalu bilangan bulat** (`BIGINT UNSIGNED`), karena Rupiah tidak memakai sen. Jangan memakai float atau DECIMAL.
- **Pengeluaran tidak boleh melebihi saldo.** Saldo target tidak boleh negatif. Tolak transaksi yang membuat saldo di bawah 0.
- **Tanggal selesai tidak disimpan di database.** Selalu dihitung saat data dibaca, dari tanggal hari ini.
- **Status tercapai dihitung dari saldo**, bukan diisi manual (lihat bagian Status).

## Tech stack

- Bahasa: **JavaScript** (bukan TypeScript), ES Modules (`"type": "module"`, gunakan `import`/`export`).
- Runtime: Node.js + **Express.js**. Port: 5000.
- Database: **MySQL** (dijalankan lewat Laragon, dilihat lewat MySQL Workbench).
- Query builder dan migrations: **Knex**. Knex BUKAN ORM. Jangan menambah ORM.
- Frontend (folder terpisah): Next.js, port 3000.

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

Library di luar daftar ini **belum terpasang**. Jangan mengimpor atau menambah library baru tanpa persetujuan pengguna. Khususnya, jangan memakai: Prisma, Sequelize, TypeORM, Mongoose, Joi, Yup, `express-validator`, `axios`, `date-fns`, `dayjs`, `moment`, atau TypeScript. Perhitungan tanggal cukup dengan objek `Date` bawaan JavaScript.

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
│   ├── controllers/     auth, goals, transactions
│   ├── db/
│   │   ├── index.js     instance Knex (diekspor, dipakai semua controller)
│   │   ├── migrations/
│   │   └── seeds/
│   ├── middlewares/     authMiddleware, validate, errorHandler
│   ├── routes/          auth, goals, transactions
│   ├── schemas/         skema Zod
│   ├── services/        calculator.js (rumus kalkulasi)
│   └── app.js
├── .env
├── .env.example
├── knexfile.js
└── package.json
```

`knexfile.js` harus menunjuk ke `./src/db/migrations` dan `./src/db/seeds`.

## Skema database

```
users        : id, email, password_hash, locale ('id' | 'en'), created_at
goals        : id, user_id, name, target_amount, daily_amount,
               is_archived (boolean, default false), created_at, updated_at
transactions : id, goal_id, type ('deposit' | 'withdrawal'),
               amount, transacted_at (date), note (nullable), created_at
```

Catatan:

- Semua kolom nominal (`target_amount`, `daily_amount`, `amount`) bertipe `BIGINT UNSIGNED`.
- `transactions.amount` **selalu positif**. Arah transaksi ditentukan oleh kolom `type`.
- `goals.daily_amount` bersifat `NOT NULL`.
- Foreign key memakai `ON DELETE CASCADE`.
- Urutan migration karena foreign key: `users`, `goals`, `transactions`.

## Rumus (semua ada di `src/services/calculator.js`)

Variabel: T = `target_amount`, R = `daily_amount`, S = saldo.

```
S         = jumlah setoran − jumlah pengeluaran
sisa      = max(T − S, 0)
progres % = min(S ÷ T × 100, 100)
H         = ceil(sisa ÷ R)            hari yang dibutuhkan
tanggal selesai = hari ini + H hari   dihitung saat dibaca
```

Saldo di SQL:

```sql
SELECT COALESCE(SUM(CASE WHEN type = 'deposit' THEN amount ELSE -amount END), 0) AS balance
FROM transactions WHERE goal_id = ?;
```

Contoh: T = 100.000 dan R = 10.000.
- Saldo 0: H = 10 hari.
- Saldo 50.000: H = 5 hari.
- Setelah pengeluaran 10.000, saldo 40.000: H = 6 hari.
- Saldo ≥ 100.000: H = 0, target tercapai.

Pembagian selalu dibulatkan ke atas, sehingga sisa sekecil apa pun tetap dihitung sebagai satu hari.

Jika pengguna tidak mencatat transaksi, H tidak berubah, tetapi tanggal selesai mundur satu hari setiap hari karena dihitung dari hari ini.

## Status target

Status **tidak disimpan**, melainkan dihitung dan dikembalikan API:

- `archived` jika `is_archived = true` (diatur manual oleh pengguna)
- `completed` jika saldo ≥ target
- `active` selain itu

Karena status mengikuti saldo, target yang sudah `completed` otomatis kembali `active` jika pengguna mencatat pengeluaran sehingga saldo turun di bawah target.

## Aturan transaksi

- Setoran menambah saldo. Pengeluaran mengurangi saldo.
- Saat mencatat pengeluaran, hitung saldo terlebih dahulu. Jika `amount` lebih besar dari saldo, tolak dengan error `WITHDRAWAL_EXCEEDS_BALANCE` (HTTP 400).
- Saat mengubah atau menghapus transaksi, pastikan saldo sesudahnya tidak menjadi negatif. Jika negatif, tolak dengan error yang sama.
- Pengecekan saldo dan penyimpanan transaksi dilakukan dalam satu transaksi database Knex (`knex.transaction`).
- Respons detail target memuat: `balance`, `remaining`, `progress`, `days_remaining`, `estimated_finish_date`, `status`.

## Endpoint

```
POST   /api/auth/login
POST   /api/auth/logout
GET    /api/auth/me

GET    /api/goals
POST   /api/goals
GET    /api/goals/:id                  detail + hasil kalkulasi
PUT    /api/goals/:id
DELETE /api/goals/:id

GET    /api/goals/:id/transactions
POST   /api/goals/:id/transactions
PUT    /api/transactions/:id
DELETE /api/transactions/:id
```

Semua route selain login wajib melewati `authMiddleware`.

## Konvensi kode

- Akses database hanya lewat instance Knex dari `src/db/index.js`. Jangan membuat koneksi `mysql2` terpisah.
- Validasi input dengan Zod melalui middleware `validate`, bukan di dalam controller.
- Semua rumus kalkulasi ada di `src/services/calculator.js`, bukan di controller. Buat sebagai fungsi murni agar mudah dites.
- Error ditangani terpusat di `errorHandler`. Respons error berupa kode atau key (misalnya `WITHDRAWAL_EXCEEDS_BALANCE`) agar bisa diterjemahkan di frontend.
- Rahasia (kredensial DB, `JWT_SECRET`) hanya dari `.env`. Jangan menulisnya langsung di kode. `.env` tidak boleh di-commit; sediakan `.env.example` tanpa nilai rahasia.

## Autentikasi, CORS, dan cookie

Frontend (port 3000) dan backend (port 5000) berbeda origin, sedangkan JWT disimpan di cookie `httpOnly`. Supaya login tersimpan:

- Backend: `cors({ origin: process.env.FRONTEND_URL, credentials: true })`. Isi `FRONTEND_URL` dengan `http://localhost:3000` saat development.
- Frontend: setiap `fetch` ke API memakai `credentials: 'include'`.
- Cookie JWT: `httpOnly: true`, `sameSite: 'lax'`, dan `secure: true` hanya di production.

## Bahasa

Teks antarmuka mendukung bahasa Indonesia (`id`) dan Inggris (`en`). API mengembalikan kode atau key, bukan kalimat, sehingga penerjemahan dilakukan di frontend.