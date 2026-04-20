# Ecoxyztem Digital Business Card

Kartu nama digital untuk tim Ecoxyztem — setiap anggota dapat URL unik, QR code, dan tombol "Simpan ke Kontak" langsung.

**Stack:** Next.js 14 (App Router) · TypeScript · Tailwind · Supabase (Postgres) · Vercel

---

## Fitur

- Form input dengan validasi ketat (Zod di frontend & backend)
- Halaman kartu publik di `/card/[slug]` dengan SSR (<1s load)
- QR code auto-generated, bisa di-download PNG/SVG
- Tombol Copy email/telepon, Save to Contacts (vCard .vcf), Share (native/WA/LinkedIn)
- Dark mode
- Halaman admin: search, bulk upload CSV, edit, hapus
- View counter atomic (RPC Postgres)
- Slug collision handling otomatis (`arya-rahmanto` → `arya-rahmanto-2` → …)
- Row Level Security di Supabase: public read only, semua write lewat service role

---

## Prasyarat

- Node.js 18.17+ ([nodejs.org](https://nodejs.org))
- Akun GitHub gratis
- Akun Supabase gratis ([supabase.com](https://supabase.com))
- Akun Vercel gratis ([vercel.com](https://vercel.com))

Kartu kredit tidak diperlukan untuk semua layanan di atas.

---

## Panduan dari Nol sampai Deploy

### Langkah 1 — Setup lokal

```bash
# Dari folder zip yang kamu download:
cd ecoxyztem-card
npm install
cp .env.example .env.local
```

### Langkah 2 — Buat project Supabase

1. Login ke [supabase.com](https://supabase.com) → **New project**
2. Isi: Name = `ecoxyztem-card`, Database password = (simpan baik-baik), Region = `Southeast Asia (Singapore)`, Plan = **Free**
3. Tunggu ~2 menit sampai project selesai dibuat

### Langkah 3 — Jalankan schema database

1. Di dashboard Supabase, masuk ke **SQL Editor** (ikon `</>` di sidebar)
2. Buka file `supabase/schema.sql` dari project ini
3. Copy seluruh isinya, paste ke SQL Editor, klik **Run**
4. Pastikan muncul "Success. No rows returned"

Schema ini membuat tabel `companies` + `cards`, index, RLS policies, dan seed company Ecoxyztem.

### Langkah 4 — Dapatkan API keys Supabase

Di dashboard Supabase: **Project Settings → API**. Copy tiga nilai ini:

- **Project URL** → `NEXT_PUBLIC_SUPABASE_URL`
- **anon public** key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- **service_role** key (klik "Reveal") → `SUPABASE_SERVICE_ROLE_KEY` ⚠️ rahasia, jangan pernah di-commit

Paste semuanya ke `.env.local`. Contoh:

```env
NEXT_PUBLIC_SUPABASE_URL=https://abcxyz123.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGc...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGc...
NEXT_PUBLIC_APP_URL=http://localhost:3000
ADMIN_SECRET=ganti-dengan-string-panjang-acak-minimal-32-karakter
```

**Tips generate ADMIN_SECRET:** jalankan `openssl rand -base64 32` di terminal.

### Langkah 5 — Jalankan lokal

```bash
npm run dev
```

Buka [http://localhost:3000](http://localhost:3000). Isi form → kartu muncul di `/card/[namamu]`.

Admin page ada di `/admin` — masukkan `ADMIN_SECRET` yang kamu set di `.env.local`.

### Langkah 6 — Push ke GitHub

```bash
git init
git add .
git commit -m "Initial commit"
```

Buat repo baru kosong di GitHub (jangan centang "Add README"), lalu:

```bash
git remote add origin https://github.com/USERNAME/ecoxyztem-card.git
git branch -M main
git push -u origin main
```

### Langkah 7 — Deploy ke Vercel

1. Buka [vercel.com](https://vercel.com), login dengan GitHub
2. **Add New → Project**, pilih repo `ecoxyztem-card`
3. Di layar konfigurasi, **jangan klik Deploy dulu** — scroll ke **Environment Variables** dan tambahkan semua key dari `.env.local` **kecuali** `NEXT_PUBLIC_APP_URL` (biarkan kosong dulu, atau isi dummy)
4. Klik **Deploy**. Tunggu 2–3 menit.
5. Setelah live, copy URL production Vercel (mis. `https://ecoxyztem-card.vercel.app`)
6. Kembali ke **Project Settings → Environment Variables**, set `NEXT_PUBLIC_APP_URL` ke URL production itu → **Redeploy** (menu Deployments → klik "⋯" → Redeploy)

**Kenapa redeploy?** Karena `NEXT_PUBLIC_APP_URL` dipakai untuk generate link QR code. Kalau tidak diset, QR akan menunjuk ke `localhost`.

### Langkah 8 — (Opsional) Custom domain

Di Vercel: **Settings → Domains** → tambahkan domain kamu (mis. `card.ecoxyztem.com`). Update DNS sesuai instruksi Vercel. Jangan lupa update `NEXT_PUBLIC_APP_URL` ke domain baru dan redeploy.

---

## Cara Pakai

### Untuk end-user
Buka halaman utama → isi form → kartu langsung jadi di URL unik.

### Untuk admin (tim Ecoxyztem)
1. Buka `/admin`, masukkan `ADMIN_SECRET`
2. **Search** nama/email/jabatan real-time
3. **Bulk upload** 50 user sekaligus via CSV

**Format CSV** (header wajib ada di baris pertama):
```csv
full_name,job_title,email,phone_number,linkedin_url
Arya Rahmanto,Investment Associate,arya@ecoxyztem.com,+6281234567890,https://linkedin.com/in/aryarahmanto
Dewi Lestari,Venture Partner,dewi@ecoxyztem.com,+6287654321098,https://linkedin.com/in/dewilestari
```

### Edit kartu
Buka `/edit/[slug]`. Akses langsung via link — tidak ada UI tombol "Edit" di halaman publik (by design, supaya tidak ada orang asing bisa edit). Untuk akses terbatas ke tim tertentu, pertimbangkan upgrade auth di bawah.

---

## Struktur Folder

```
ecoxyztem-card/
├── supabase/
│   └── schema.sql              # Jalankan sekali di Supabase SQL Editor
├── src/
│   ├── app/
│   │   ├── layout.tsx          # Root layout + fonts
│   │   ├── page.tsx            # Landing (form)
│   │   ├── globals.css
│   │   ├── not-found.tsx
│   │   ├── card/[slug]/page.tsx      # Public card page (SSR)
│   │   ├── edit/[slug]/page.tsx      # Edit form
│   │   ├── admin/page.tsx            # Admin dashboard (client)
│   │   └── api/
│   │       ├── cards/route.ts                # POST create, GET list (admin)
│   │       ├── cards/[slug]/route.ts         # PATCH edit, DELETE admin
│   │       ├── cards/[slug]/vcard/route.ts   # Download .vcf
│   │       └── admin/bulk-upload/route.ts    # CSV import
│   ├── components/             # UI components
│   ├── lib/
│   │   ├── supabase.ts         # Public + admin clients
│   │   ├── validators.ts       # Zod schemas
│   │   ├── slug.ts             # Slug generator + collision handling
│   │   ├── vcard.ts            # RFC 6350 vCard builder
│   │   └── utils.ts
│   └── types/index.ts
├── package.json
├── tailwind.config.ts
└── README.md
```

---

## Arsitektur singkat

**Database (Postgres di Supabase):**
- `companies` — satu baris untuk Ecoxyztem (seeded). Skalabel ke multi-company tanpa refactor.
- `cards` — FK ke `companies`, unique constraint di `email` dan `slug`.
- Index di `company_id`, `created_at`, dan trigram index di `full_name` untuk fuzzy search.
- RPC `increment_card_view` — atomic counter, aman dari race condition.

**Security:**
- RLS aktif: anon key hanya bisa `SELECT`. Semua `INSERT/UPDATE/DELETE` lewat API route yang pakai `service_role` key (server-side saja).
- Admin endpoint diproteksi dengan `x-admin-secret` header. Untuk production scale, ganti dengan Supabase Auth (lihat "Upgrade path").

**Performance:**
- Card page pakai ISR (`revalidate = 60`), jadi QR scan berkali-kali nge-hit cache Vercel, bukan database.
- View counter fire-and-forget (tidak blocking render).
- Untuk 50–100 user & traffic normal, free tier Supabase (500MB DB, 5GB bandwidth) dan Vercel (100GB bandwidth) sangat cukup.

---

## Upgrade Path (jika nanti dibutuhkan)

| Kebutuhan | Solusi |
|---|---|
| Multiple companies | Tabel `companies` sudah siap — tinggal tambah kolom `company_id` ke form admin |
| Real auth (SSO, magic link) | Pakai Supabase Auth (`@supabase/ssr`), ganti `ADMIN_SECRET` dengan role check |
| Custom domain per user | Tambah kolom `custom_domain` + middleware rewrite |
| Analytics detail (bukan hanya count) | Tabel `card_views` dengan timestamp + user-agent |
| Email notification saat kartu dilihat | Supabase Edge Function trigger on insert ke `card_views` |

---

## Troubleshooting

**`Missing Supabase env vars` saat `npm run dev`**
→ Pastikan `.env.local` ada dan sudah diisi. Restart dev server setelah edit.

**QR code muter menunjuk ke `localhost:3000` di production**
→ `NEXT_PUBLIC_APP_URL` belum diset di Vercel. Set + redeploy.

**`permission denied for table cards` saat insert**
→ RLS aktif tapi route pakai `supabasePublic` (anon key). Harus pakai `supabaseAdmin()` untuk write. Cek bahwa route kamu sudah import yang benar.

**Bulk upload sukses tapi error "Email already exists"**
→ Email di CSV ada duplikat di database. Normal — error count akan muncul, kartu lainnya tetap dibuat.

**Halaman `/admin` tidak bisa masuk**
→ `ADMIN_SECRET` di browser (yang kamu ketik) harus persis sama dengan di env var Vercel. Re-check, lalu redeploy kalau baru ganti.

---

## Lisensi

Internal Ecoxyztem. Bebas dimodifikasi untuk kebutuhan tim.
