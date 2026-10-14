# MySaving — Frontend

Aplikasi antarmuka MySaving, dibangun dengan **Next.js** (App Router) dan **JavaScript**. Folder ini hanya berisi tampilan. Semua data diambil dari API di folder `backend/`.

## Library yang digunakan

### Inti (terpasang otomatis oleh `create-next-app`)

| Library | Fungsi |
|---|---|
| `next` | framework utama: routing berbasis folder, rendering, dan server |
| `react` | library UI untuk membuat komponen |
| `react-dom` | menampilkan komponen React di browser |
| `tailwindcss` | styling dengan utility class langsung di JSX |
| `@tailwindcss/postcss` | plugin PostCSS agar Tailwind berjalan di Next.js |
| `eslint` dan `eslint-config-next` | pengecekan kualitas kode (ikut terpasang kalau ESLint dipilih saat setup) |

### Ditambahkan manual

| Library | Perintah install | Fungsi |
|---|---|---|
| `lucide-react` | `npm install lucide-react` | kumpulan ikon sebagai komponen React |
| `next-intl` | `npm install next-intl` | dukungan dua bahasa (Indonesia dan Inggris) |

### Contoh pemakaian singkat

**lucide-react** — impor ikon yang dibutuhkan saja:

```jsx
import { PiggyBank, Plus } from 'lucide-react';

<PiggyBank size={20} />
<Plus size={16} />
```

**next-intl** — teks dipindahkan ke file terjemahan, bukan ditulis langsung di komponen:

```jsx
import { useTranslations } from 'next-intl';

const t = useTranslations('Dashboard');
<h1>{t('title')}</h1>
```

File terjemahannya ada di `messages/id.json` dan `messages/en.json`.

## Yang tidak memakai library tambahan

- **Format Rupiah:** cukup `Intl.NumberFormat` bawaan JavaScript.

  ```js
  new Intl.NumberFormat('id-ID', {
    style: 'currency',
    currency: 'IDR',
    maximumFractionDigits: 0,
  }).format(3000000); // "Rp 3.000.000"
  ```

- **Memanggil API:** memakai `fetch` bawaan, tanpa `axios`.
- **Format tanggal:** memakai `Intl.DateTimeFormat` bawaan.

## Menjalankan

```bash
npm install
npm run dev
```

Buka `http://localhost:3000`. Pastikan backend sudah berjalan di port 5000.

| Script | Fungsi |
|---|---|
| `npm run dev` | menjalankan server development |
| `npm run build` | membuat build production |
| `npm run start` | menjalankan hasil build production |
| `npm run lint` | mengecek kode dengan ESLint |

## Struktur folder

```
frontend/
├── app/
│   └── [locale]/        halaman dibungkus bahasa (id atau en)
│       ├── login/
│       ├── dashboard/
│       └── goals/[id]/
├── components/          komponen yang dipakai ulang
├── messages/            terjemahan: id.json, en.json
└── lib/                 helper: pemanggil API, format Rupiah
```

## Catatan

- Proyek ini memakai **JavaScript**, bukan TypeScript.
- Jangan menambah library baru tanpa kebutuhan yang jelas. Cek dulu apakah fitur bawaan JavaScript atau Next.js sudah cukup.