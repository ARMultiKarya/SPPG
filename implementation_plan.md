# Rencana Implementasi: Aplikasi Web Multi-Tenant "Dokumen Operasional SPPG" (SIDOK SPPG)
**Kabupaten Magelang — 42 Satuan Pelayanan Pemenuhan Gizi (SPPG)**

---

## 1. Ringkasan Eksekutif & Arsitektur Sistem

Aplikasi **SIDOK SPPG** dirancang sebagai sistem operasi dokumen terpadu berbasis **multi-tenant** untuk 42 SPPG di wilayah Kabupaten Magelang. Sistem ini menjamin isolasi data yang ketat antar-SPPG melalui kombinasi **Row Level Security (RLS)** pada basis data PostgreSQL Supabase dan validasi sesi di layer Next.js 14+ App Router.

### Tech Stack Utama
- **Framework Frontend & Server**: Next.js 14+ (App Router), TypeScript, Tailwind CSS, Lucide React
- **Komponen Antarmuka**: Shadcn/UI (Dialog, Dropdown, Table, Tabs, Sheet, Card, Badge, Toaster, Form)
- **Backend & Database**: Supabase (PostgreSQL, Supabase Auth, Storage Buckets privat, RLS)
- **Pemrosesan Excel**: `exceljs` dan `xlsx` (SheetJS) untuk pembacaan, validasi, pemisahan multi-sheet, dan ekspor
- **Bahasa Antarmuka**: Bahasa Indonesia formal dan ramah pengguna

```
                 +-------------------------------------------------------+
                 |            Next.js 14+ App Router Client              |
                 +---------------------------+---------------------------+
                                             |
                                  Middleware & Session Check
                                             |
                         +-------------------+-------------------+
                         |                                       |
                 [Role: ADMIN]                           [Role: SPPG]
             (Akses Semua 42 SPPG)                 (Akses Hanya 1 SPPG Terkait)
                         |                                       |
                         +-------------------+-------------------+
                                             |
                              Supabase Client / Server Actions
                                             |
                      +----------------------+----------------------+
                      |                      |                      |
             PostgreSQL + RLS         Supabase Storage       Audit Trail Engine
          (user_sppg multi-tenant)   (Signed URLs Private)     (audit_logs)
```

---

## 2. Struktur Folder Proyek

Struktur folder dibuat modular, terorganisir dengan rapi sesuai konvensi Next.js App Router:

```
sidok-sppg/
├── .env.example
├── .env.local
├── .gitignore
├── README.md
├── implementation_plan.md
├── next.config.mjs
├── package.json
├── postcss.config.mjs
├── tailwind.config.ts
├── tsconfig.json
│
├── supabase/
│   ├── migrations/
│   │   ├── 20261014000001_initial_schema.sql       # Skema 13 tabel, enums, triggers
│   │   ├── 20261014000002_rls_policies.sql         # RLS untuk multi-tenancy & storage
│   │   └── 20261014000003_seed_42_sppg.sql         # Data 42 SPPG Kabupaten Magelang
│   └── seed.sql
│
├── public/
│   ├── icons/
│   ├── logo-sppg.png
│   └── templates/
│
├── src/
│   ├── app/
│   │   ├── layout.tsx                              # Root Layout + Font & Toast Provider
│   │   ├── page.tsx                                # Landing page / redirect ke login
│   │   ├── globals.css                             # Theme styling & Shadcn variables
│   │   │
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   │   └── page.tsx                        # Halaman Login Multi-Role
│   │   │   └── auth/callback/
│   │   │       └── route.ts                        # Supabase Auth code exchange
│   │   │
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx                          # App Shell (Sidebar responsif, Header, SPPG Switcher)
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx                        # Modul 1: Dashboard Ringkasan (Admin vs SPPG)
│   │   │   │
│   │   │   ├── gaji/
│   │   │   │   └── page.tsx                        # Modul 2: Gaji AK AG (Input, Rekap, Export Excel)
│   │   │   │
│   │   │   ├── kwitansi/
│   │   │   │   └── page.tsx                        # Modul 3: Kwitansi Operasional (Kategori, Upload, Rekap)
│   │   │   │
│   │   │   ├── lpdh/
│   │   │   │   └── page.tsx                        # Modul 4: LPDH Harian (Upload Excel, Filter, Unduh)
│   │   │   │
│   │   │   ├── pengajian-lpdh/
│   │   │   │   └── page.tsx                        # Modul 5: Template Excel Multi-Sheet (Split & Download)
│   │   │   │
│   │   │   ├── kop-yayasan/
│   │   │   │   └── page.tsx                        # Modul 6: KOP Yayasan (Pengajuan Akses, Status, Unduh)
│   │   │   │
│   │   │   ├── update-dapur/
│   │   │   │   └── page.tsx                        # Modul 7: Update Dapur (Admin Multi-Sheet Import & Rollback)
│   │   │   │
│   │   │   ├── audit-log/
│   │   │   │   └── page.tsx                        # Modul 8: Log Audit Aktivitas Dokumen (Admin Only)
│   │   │   │
│   │   │   └── admin/
│   │   │       └── sppg/
│   │   │           └── page.tsx                    # Manajemen 42 SPPG & Asosiasi Akun Pengguna
│   │   │
│   │   └── api/
│   │       ├── storage/signed-url/route.ts         # Endpoint pembuatan Signed URL aman
│   │       ├── excel/split-sheets/route.ts         # Endpoint pemisahan sheet Excel
│   │       └── excel/export/gaji/route.ts          # Endpoint generate Excel Gaji AK AG
│   │
│   ├── components/
│   │   ├── ui/                                     # Shadcn UI primitives (button, dialog, select, etc.)
│   │   ├── layout/
│   │   │   ├── sidebar.tsx                         # Sidebar navigasi (Desktop & Mobile Drawer)
│   │   │   ├── header.tsx                          # Header profil, notifikasi, dan active tenant badge
│   │   │   └── sppg-switcher.tsx                   # Dropdown pemilih SPPG untuk peran ADMIN
│   │   ├── shared/
│   │   │   ├── file-upload-dropzone.tsx            # Komponen drag-and-drop upload file dengan validasi
│   │   │   ├── data-table.tsx                      # Tabel data reusable dengan sorting, pagination, search
│   │   │   ├── confirmation-modal.tsx              # Modal konfirmasi (hapus/rollback)
│   │   │   └── export-excel-button.tsx             # Tombol trigger ekspor Excel instan
│   │   └── modules/
│   │       ├── dashboard/                          # Widget & metrik ringkasan
│   │       ├── gaji/                               # Form gaji, slip preview, modal rekap
│   │       ├── kwitansi/                           # Form kwitansi, preview berkas, rekap kategori
│   │       ├── lpdh/                               # File validator & upload history
│   │       ├── pengajian-lpdh/                     # Sheet viewer tabs & batch downloader
│   │       ├── kop-yayasan/                        # Access request table & approval badges
│   │       └── update-dapur/                       # Sheet mapper & import confirmation
│   │
│   ├── hooks/
│   │   ├── use-current-user.ts                     # Hook data profil & role aktif
│   │   ├── use-sppg-context.ts                     # Hook tenant aktif (tersinkron di cookies/state)
│   │   └── use-signed-url.ts                       # Hook download file privat
│   │
│   ├── lib/
│   │   ├── supabase/
│   │   │   ├── client.ts                           # Supabase browser client
│   │   │   ├── server.ts                           # Supabase server client (cookies)
│   │   │   ├── admin.ts                            # Supabase service role client (bypassing RLS for admin ops)
│   │   │   └── middleware.ts                       # Session refresher
│   │   ├── excel/
│   │   │   ├── reader.ts                           # Pembacaan multi-sheet dengan SheetJS/ExcelJS
│   │   │   ├── splitter.ts                         # Ekstraksi & generate file terpisah per sheet
│   │   │   ├── exporter-gaji.ts                    # Builder file Excel Gaji AK AG
│   │   │   └── exporter-kwitansi.ts                # Builder file Excel Kwitansi Rekap
│   │   ├── audit.ts                                # Helper pencatatan audit log
│   │   ├── storage.ts                              # Helper upload & signed URL Supabase Storage
│   │   └── utils.ts                                # Format Rupiah, format tanggal Indonesia, cn utility
│   │
│   ├── middleware.ts                               # Route guard: cek auth & validasi hak akses SPPG
│   └── types/
│       ├── database.ts                             # Tipe TypeScript hasil query Supabase
│       └── index.ts                                # Tipe entitas bisnis & enum
```

---

## 3. Daftar Halaman & Hak Akses

| URL Route | Nama Modul | Hak Akses | Deskripsi Fitur Utama |
|---|---|---|---|
| `/login` | Masuk Akun | Publik | Otentikasi Email & Kata Sandi, proteksi redirect jika sudah login |
| `/dashboard` | Dashboard Operasional | ADMIN & SPPG | **Admin**: Agregasi 42 SPPG (total user, LPDH masuk, kwitansi, gaji, permohonan KOP).<br>**SPPG**: Metrik khusus SPPG terkait. |
| `/gaji` | Gaji AK AG | ADMIN & SPPG | Input gaji bulanan (pokok, tunjangan, potongan, total otomatis), rekapitulasi, ekspor file Excel bulanan. |
| `/kwitansi` | Kwitansi Operasional | ADMIN & SPPG | Input kwitansi (BBM, Galon, Genset, Insentif, Kebersihan, + custom), upload berkas PDF/foto, filter periode & SPPG, ekspor Excel. |
| `/lpdh` | LPDH Harian | ADMIN & SPPG | Upload berkas LPDH harian (.xlsx/.xls), riwayat upload, admin dapat memfilter & mengunduh semua SPPG. |
| `/pengajian-lpdh` | Template Excel Multi-Sheet | ADMIN | Upload file Excel multi-sheet, parsing otomatis, pratinjau tiap sheet dalam tab, unduh file satuan (contoh: `Oktober.xlsx`, `November.xlsx`). |
| `/kop-yayasan` | KOP Yayasan | ADMIN & SPPG | **Admin**: Unggah CV/Profil/SK, set visibilitas, setujui/tolak permohonan SPPG.<br>**SPPG**: Mengajukan akses, pantau status (MENUNGGU/DISETUJUI/DITOLAK), unduh jika disetujui. |
| `/update-dapur` | Update Dapur | ADMIN | Upload template Excel harian, pratinjau data tabular, impor batch ke database, histori impor, dan fitur rollback/hapus batch impor. |
| `/audit-log` | Log Audit Dokumen | ADMIN | Rekam jejak seluruh aksi upload, unduh, ubah, hapus, dan persetujuan berkas dengan timestamp & identitas user. |
| `/admin/sppg` | Manajemen 42 SPPG | ADMIN | Daftar 42 SPPG Magelang, status aktif/non-aktif, pemetaan user ke SPPG (`user_sppg`). |

---

## 4. Daftar Tabel Database PostgreSQL

Skema database dirancang lengkap dengan integritas referensial, indeks performa, dan trigger update timestamp:

### 1. `sppg`
Menyimpan master data 42 SPPG di Kabupaten Magelang.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `nama_sppg` (VARCHAR(150), NOT NULL, UNIQUE)
- `kecamatan` (VARCHAR(100), NOT NULL)
- `alamat` (TEXT, NULL)
- `status_aktif` (BOOLEAN, default `true`)
- `created_at` (TIMESTAMPTZ, default `now()`)
- `updated_at` (TIMESTAMPTZ, default `now()`)

### 2. `profiles`
Profil pengguna yang terhubung dengan `auth.users` Supabase.
- `id` (UUID, Primary Key, references `auth.users(id)` ON DELETE CASCADE)
- `nama` (VARCHAR(150), NOT NULL)
- `email` (VARCHAR(255), NOT NULL, UNIQUE)
- `role_global` (VARCHAR(20), NOT NULL, CHECK in (`'ADMIN'`, `'SPPG'`))
- `nomor_kontak` (VARCHAR(30), NULL)
- `avatar_url` (TEXT, NULL)
- `created_at` (TIMESTAMPTZ, default `now()`)
- `updated_at` (TIMESTAMPTZ, default `now()`)

### 3. `user_sppg`
Tabel relasi multi-tenant yang menentukan SPPG mana yang dapat diakses oleh user.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `user_id` (UUID, NOT NULL, references `profiles(id)` ON DELETE CASCADE)
- `sppg_id` (UUID, NOT NULL, references `sppg(id)` ON DELETE CASCADE)
- `role` (VARCHAR(20), NOT NULL, default `'SPPG'`, CHECK in (`'ADMIN'`, `'SPPG'`))
- `created_at` (TIMESTAMPTZ, default `now()`)
- *Constraint*: `UNIQUE(user_id, sppg_id)`

### 4. `gaji_ak_ag`
Data penggajian AK AG bulanan.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `sppg_id` (UUID, NOT NULL, references `sppg(id)` ON DELETE CASCADE)
- `periode_bulan` (SMALLINT, NOT NULL, CHECK `1 <= periode_bulan <= 12`)
- `periode_tahun` (INT, NOT NULL, CHECK `periode_tahun >= 2024`)
- `nama_pegawai` (VARCHAR(150), NOT NULL)
- `jabatan` (VARCHAR(100), NOT NULL)
- `hari_kerja` (INT, NOT NULL, default 0)
- `gaji_pokok` (NUMERIC(14,2), NOT NULL, default 0)
- `tunjangan` (NUMERIC(14,2), NOT NULL, default 0)
- `potongan` (NUMERIC(14,2), NOT NULL, default 0)
- `total` (NUMERIC(14,2), NOT NULL) -- dihitung `gaji_pokok + tunjangan - potongan`
- `created_by` (UUID, references `profiles(id)`)
- `created_at` (TIMESTAMPTZ, default `now()`)
- `updated_at` (TIMESTAMPTZ, default `now()`)

### 5. `kategori_kwitansi`
Master kategori pengeluaran operasional.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `nama_kategori` (VARCHAR(100), NOT NULL, UNIQUE)
- `aktif` (BOOLEAN, default `true`)
- `created_at` (TIMESTAMPTZ, default `now()`)
- *Data Bawaan*: BBM, Galon, Genset, Insentif, Kebersihan.

### 6. `kwitansi`
Transaksi pengeluaran dan upload bukti kwitansi.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `sppg_id` (UUID, NOT NULL, references `sppg(id)` ON DELETE CASCADE)
- `kategori_id` (UUID, NOT NULL, references `kategori_kwitansi(id)`)
- `tanggal` (DATE, NOT NULL)
- `nominal` (NUMERIC(14,2), NOT NULL, CHECK `nominal >= 0`)
- `keterangan` (TEXT, NULL)
- `file_url` (TEXT, NOT NULL)
- `storage_path` (TEXT, NOT NULL)
- `created_by` (UUID, references `profiles(id)`)
- `created_at` (TIMESTAMPTZ, default `now()`)
- `updated_at` (TIMESTAMPTZ, default `now()`)

### 7. `lpdh`
Unggahan berkas LPDH harian dalam format Excel.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `sppg_id` (UUID, NOT NULL, references `sppg(id)` ON DELETE CASCADE)
- `tanggal` (DATE, NOT NULL)
- `nama_file` (VARCHAR(255), NOT NULL)
- `file_url` (TEXT, NOT NULL)
- `storage_path` (TEXT, NOT NULL)
- `file_size` (BIGINT, NOT NULL)
- `uploaded_by` (UUID, references `profiles(id)`)
- `created_at` (TIMESTAMPTZ, default `now()`)

### 8. `excel_template`
Riwayat unggahan template Excel multi-sheet oleh Admin.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `jenis` (VARCHAR(100), NOT NULL, default `'PENGAJIAN_LPDH'`)
- `nama_file` (VARCHAR(255), NOT NULL)
- `file_url` (TEXT, NOT NULL)
- `storage_path` (TEXT, NOT NULL)
- `total_sheets` (INT, default 1)
- `uploaded_by` (UUID, references `profiles(id)`)
- `created_at` (TIMESTAMPTZ, default `now()`)

### 9. `excel_sheet`
Rincian masing-masing sheet yang berhasil diekstraksi dari template Excel.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `excel_template_id` (UUID, NOT NULL, references `excel_template(id)` ON DELETE CASCADE)
- `nama_sheet` (VARCHAR(150), NOT NULL)
- `file_url` (TEXT, NOT NULL)
- `storage_path` (TEXT, NOT NULL)
- `total_baris` (INT, default 0)
- `total_kolom` (INT, default 0)
- `sheet_data_json` (JSONB, NULL)
- `created_at` (TIMESTAMPTZ, default `now()`)

### 10. `dokumen_yayasan`
Dokumen legalitas/profil yayasan (CV, SK, Profil, dll).
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `judul` (VARCHAR(200), NOT NULL)
- `jenis` (VARCHAR(100), NOT NULL)
- `deskripsi` (TEXT, NULL)
- `file_url` (TEXT, NOT NULL)
- `storage_path` (TEXT, NOT NULL)
- `hanya_admin` (BOOLEAN, default `false`)
- `uploaded_by` (UUID, references `profiles(id)`)
- `created_at` (TIMESTAMPTZ, default `now()`)
- `updated_at` (TIMESTAMPTZ, default `now()`)

### 11. `akses_kop_yayasan`
Alur persetujuan akses dokumen yayasan untuk masing-masing SPPG.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `sppg_id` (UUID, NOT NULL, references `sppg(id)` ON DELETE CASCADE)
- `dokumen_id` (UUID, NULL, references `dokumen_yayasan(id)` ON DELETE CASCADE)
- `status` (VARCHAR(20), NOT NULL, default `'MENUNGGU'`, CHECK in (`'MENUNGGU'`, `'DISETUJUI'`, `'DITOLAK'`))
- `catatan_admin` (TEXT, NULL)
- `diajukan_oleh` (UUID, references `profiles(id)`)
- `disetujui_oleh` (UUID, references `profiles(id)`)
- `tanggal_persetujuan` (TIMESTAMPTZ, NULL)
- `created_at` (TIMESTAMPTZ, default `now()`)

### 12. `update_dapur_batch`
Riwayat batch upload Excel dapur untuk pelacakan dan pembatalan (rollback).
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `nama_file` (VARCHAR(255), NOT NULL)
- `tanggal_impor` (TIMESTAMPTZ, default `now()`)
- `jumlah_baris` (INT, NOT NULL, default 0)
- `imported_by` (UUID, references `profiles(id)`)
- `status` (VARCHAR(30), default `'ACTIVE'`, CHECK in (`'ACTIVE'`, `'ROLLED_BACK'`))
- `created_at` (TIMESTAMPTZ, default `now()`)

### 13. `update_dapur`
Data operasional harian dapur hasil parsing Excel.
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `batch_id` (UUID, NOT NULL, references `update_dapur_batch(id)` ON DELETE CASCADE)
- `sppg_id` (UUID, NOT NULL, references `sppg(id)` ON DELETE CASCADE)
- `tanggal` (DATE, NOT NULL)
- `data_json` (JSONB, NOT NULL)
- `file_sumber_id` (UUID, NULL, references `excel_template(id)`)
- `imported_by` (UUID, references `profiles(id)`)
- `created_at` (TIMESTAMPTZ, default `now()`)

### 14. `audit_logs`
Pencatatan seluruh aktivitas penting (Upload, Download, Edit, Hapus, Persetujuan).
- `id` (UUID, Primary Key, default `gen_random_uuid()`)
- `user_id` (UUID, references `profiles(id)`)
- `sppg_id` (UUID, NULL, references `sppg(id)`)
- `aksi` (VARCHAR(50), NOT NULL) -- `'UPLOAD'`, `'DOWNLOAD'`, `'CREATE'`, `'UPDATE'`, `'DELETE'`, `'APPROVE'`, `'REJECT'`, `'ROLLBACK'`
- `tabel` (VARCHAR(50), NOT NULL)
- `record_id` (UUID, NULL)
- `detail` (JSONB, NULL)
- `created_at` (TIMESTAMPTZ, default `now()`)

---

## 5. Kebijakan Row Level Security (RLS) & Isolasi Multi-Tenant

Untuk mencegah kebocoran data antar-tenant (*cross-tenant leakage*), RLS diaktifkan secara wajib (`ALTER TABLE ... ENABLE ROW LEVEL SECURITY`) pada seluruh tabel operasional.

### Helper Functions Bebas Rekursi
Dibuat fungsi PostgreSQL `SECURITY DEFINER` dengan `search_path = public`:
1. `is_admin(user_uuid UUID)`: Memeriksa apakah pengguna memiliki `role_global = 'ADMIN'`.
2. `user_sppg_ids(user_uuid UUID)`: Mengembalikan daftar `sppg_id` yang terdaftar untuk user tersebut di `user_sppg`.
3. `has_sppg_access(user_uuid UUID, target_sppg_id UUID)`: Evaluasi langsung apakah user adalah Admin ATAU target_sppg_id ada dalam relasi user.

### Aturan RLS per Tabel
| Tabel | Kebijakan SELECT | Kebijakan INSERT / UPDATE / DELETE |
|---|---|---|
| `sppg` | `is_admin(auth.uid()) OR id IN (SELECT user_sppg_ids(auth.uid()))` | Hanya `is_admin(auth.uid())` |
| `profiles` | User membaca profil diri sendiri ATAU `is_admin(auth.uid())` | User mengubah profil diri sendiri ATAU Admin |
| `user_sppg` | User melihat pemetaan miliknya ATAU `is_admin(auth.uid())` | Hanya `is_admin(auth.uid())` |
| `gaji_ak_ag` | `has_sppg_access(auth.uid(), sppg_id)` | `has_sppg_access(auth.uid(), sppg_id)` |
| `kategori_kwitansi`| Semua user yang terotentikasi | Hanya `is_admin(auth.uid())` |
| `kwitansi` | `has_sppg_access(auth.uid(), sppg_id)` | `has_sppg_access(auth.uid(), sppg_id)` |
| `lpdh` | `has_sppg_access(auth.uid(), sppg_id)` | `has_sppg_access(auth.uid(), sppg_id)` |
| `excel_template` & `excel_sheet` | Hanya `is_admin(auth.uid())` | Hanya `is_admin(auth.uid())` |
| `dokumen_yayasan` | `is_admin(auth.uid())` ATAU (`NOT hanya_admin` DAN SPPG user memiliki izin di `akses_kop_yayasan` status `'DISETUJUI'`) | Hanya `is_admin(auth.uid())` |
| `akses_kop_yayasan` | `is_admin(auth.uid()) OR has_sppg_access(auth.uid(), sppg_id)` | SPPG mengajukan INSERT; Hanya Admin yang boleh UPDATE (menyetujui/menolak) |
| `update_dapur` & batch | Hanya `is_admin(auth.uid())` | Hanya `is_admin(auth.uid())` |
| `audit_logs` | Hanya `is_admin(auth.uid())` | INSERT diizinkan untuk semua user via server action / trigger |

### Supabase Storage Security (Private Buckets)
1. Bucket: `dokumen-operasional`, `kwitansi-berkas`, `lpdh-berkas`, `dokumen-yayasan`, `excel-processed` disetel `public = false`.
2. Semua tautan unduhan dibuat melalui **Signed URL** dengan masa berlaku terbatas (contoh: 60 menit) setelah melalui otentikasi hak akses di layer backend Next.js.
3. Path file dipisahkan per tenant: `{sppg_id}/{tahun}/{bulan}/{nama_file}` untuk menjamin kerapian struktur penyimpanan.

---

## 6. Data Bawaan (Seed Data): 42 SPPG Kabupaten Magelang

Seluruh 42 SPPG telah dipetakan secara akurat ke kecamatannya masing-masing di Kabupaten Magelang dan sekitarnya:

```sql
INSERT INTO sppg (nama_sppg, kecamatan, status_aktif) VALUES
  ('SPPG ADIKARTO', 'MUNTILAN', true),
  ('SPPG ARGOMULYO', 'SALATIGA', true),
  ('SPPG BALEKERTO', 'KALIANGKRIK', true),
  ('SPPG BANYUSARI', 'TEGALREJO', true),
  ('SPPG BATURONO', 'SALAM', true),
  ('SPPG BAWANG', 'PAKIS', true),
  ('SPPG BESERAN', 'KALIANGKRIK', true),
  ('SPPG BLEMBENG', 'TEGALREJO', true),
  ('SPPG BOJONG', 'MUNGKID', true),
  ('SPPG CANDISARI', 'SECANG', true),
  ('SPPG DULMAS', 'TEGALREJO', true),
  ('SPPG GANDUSARI', 'BANDONGAN', true),
  ('SPPG GIRIKULON', 'SECANG', true),
  ('SPPG GIYANTI', 'CANDIMULYO', true),
  ('SPPG GONDOSULI', 'MUNTILAN', true),
  ('SPPG GONDOWANGI', 'SAWANGAN', true),
  ('SPPG JOGOMULYO', 'TEMPURAN', true),
  ('SPPG JOGOYASAN', 'NGABLAK', true),
  ('SPPG KAJORAN', 'KAJORAN', true),
  ('SPPG KALISALAK', 'SALAMAN', true),
  ('SPPG KAPONAN', 'PAKIS', true),
  ('SPPG KETEP', 'SAWANGAN', true),
  ('SPPG KRINCING', 'SECANG', true),
  ('SPPG MRANGGEN 2', 'SRUMBUNG', true),
  ('SPPG NGADIHARJO', 'BOROBUDUR', true),
  ('SPPG NGASINAN', 'GRABAG', true),
  ('SPPG NGAWEN', 'MUNTILAN', true),
  ('SPPG NOBOREJO', 'ARGOMULYO', true),
  ('SPPG PASURUHAN', 'MERTOYUDAN', true),
  ('SPPG PLOSOGEDE', 'NGLUWAR', true),
  ('SPPG PODOSOKO', 'CANDIMULYO', true),
  ('SPPG SIDOGAWUNG', 'TEMPURAN', true),
  ('SPPG SIDOGEDE', 'GRABAG', true),
  ('SPPG SIDOMULYO', 'SALAMAN', true),
  ('SPPG SIDOREJO', 'KIDUL', true),
  ('SPPG SRUMBUNG', 'MRANGGEN 1', true),
  ('SPPG SRUMBUNG', 'SRUMBUNG', true),
  ('SPPG SUGIHMAS', 'GRABAG', true),
  ('SPPG SUMURARUM', 'GRABAG', true),
  ('SPPG SUROJOYO', 'CANDIMULYO', true),
  ('SPPG TRASAN', 'BANDONGAN', true),
  ('SPPG WINDUSARI', 'WINDUSARI', true)
ON CONFLICT (nama_sppg) DO NOTHING;
```

---

## 7. Urutan Pengerjaan Bertahap (Tahap 1 – Tahap 8)

Sesuai instruksi pengguna, pengerjaan akan dijalankan secara bertahap setelah rencana ini disetujui:

### **Tahap 1: Setup Proyek, Supabase, Auth, Role, dan Seed 42 SPPG**
- Inisialisasi Next.js 14+ App Router, Tailwind CSS, TypeScript, dan Shadcn/UI.
- Konfigurasi Supabase Client (`client.ts`, `server.ts`, `middleware.ts`, `admin.ts`).
- Pembuatan file migrasi SQL lengkap: skema 13 tabel, triggers, helper security functions, dan seed data 42 SPPG.
- Implementasi sistem autentikasi: login, logout, sesi pengguna, serta pemetaan role (ADMIN vs SPPG).
- Verifikasi proteksi middleware untuk rute publik vs terproteksi.

### **Tahap 2: Dashboard & Modul Gaji AK AG**
- Pembuatan shell aplikasi responsif: Sidebar (desktop & mobile drawer), Header dengan profil, dan Tenant Switcher (khusus Admin).
- Halaman **Dashboard**:
  - Admin: Ringkasan metrik 42 SPPG (total LPDH masuk, total kwitansi, rekap gaji, permohonan KOP).
  - Akun SPPG: Ringkasan metrik eksklusif untuk SPPG miliknya.
- Halaman **Gaji AK AG**:
  - Formulir input data gaji bulanan (nama, jabatan, hari kerja, gaji pokok, tunjangan, potongan, total otomatis).
  - Tabel rekapitulasi gaji per bulan dan per SPPG.
  - Fitur ekspor Excel laporan gaji bulanan menggunakan SheetJS/ExcelJS.

### **Tahap 3: Modul Kwitansi**
- Seed 5 kategori bawaan: BBM, Galon, Genset, Insentif, Kebersihan.
- Fitur penambahan kategori kwitansi baru oleh Admin.
- Formulir pencatatan kwitansi: tanggal, kategori, nominal (format Rupiah dinamis), keterangan, dan upload bukti (foto/PDF).
- Tabel daftar kwitansi dengan filter interaktif: SPPG, Kategori, Bulan, dan Tahun.
- Rekapitulasi total pengeluaran per kategori & ekspor data kwitansi ke file Excel.

### **Tahap 4: Modul LPDH Harian**
- Komponen drag-and-drop upload file Excel LPDH harian khusus file `.xlsx` dan `.xls`.
- Penyimpanan berkas di Supabase Storage bucket privat dan pencatatan metadata (SPPG, tanggal, ukuran file, user pengunggah).
- Tabel daftar riwayat LPDH:
  - SPPG hanya melihat riwayat SPPG-nya.
  - Admin melihat seluruh 42 SPPG dengan filter SPPG dan rentang tanggal.
- Pengunduhan berkas LPDH melalui Signed URL sementara.

### **Tahap 5: Pemrosesan Excel Multi-Sheet (Pengajian LPDH)**
- Halaman upload template Excel multi-sheet khusus Admin.
- Algoritma pemrosesan server-side:
  - Membaca semua sheet yang ada di dalam workbook Excel yang diunggah.
  - Memisahkan masing-masing sheet menjadi workbook `.xlsx` independen yang dinamai sesuai nama sheet aslinya (contoh: sheet `"Oktober"` menjadi `Oktober.xlsx`).
  - Menyimpan setiap file sheet ke Supabase Storage.
- Tampilan antarmuka berbasis Tabs untuk setiap sheet, lengkap dengan pratinjau data dan tombol unduh per sheet.
- Penyimpanan riwayat unggahan dan log pemisahan sheet.

### **Tahap 6: KOP Yayasan & Alur Persetujuan Akses**
- Halaman pengunggahan dokumen yayasan oleh Admin (CV, Profil Yayasan, SK, dsb).
- Pengaturan visibilitas dokumen: `"Hanya Admin"` vs `"Dapat Diakses SPPG Setelah Persetujuan"`.
- Alur kerja permohonan SPPG:
  - Akun SPPG mengajukan akses dokumen KOP Yayasan.
  - Status pengajuan: `MENUNGGU`, `DISETUJUI`, `DITOLAK`.
  - Admin meninjau permohonan, memberikan catatan, dan mengubah status.
- Jika disetujui, SPPG dapat melihat pratinjau dan mengunduh berkas melalui signed URL.

### **Tahap 7: Update Dapur & Audit Log**
- Modul **Update Dapur** (Admin Only):
  - Unggah template Excel harian multi-sheet.
  - Pratinjau data tabular sebelum konfirmasi impor.
  - Impor data ke tabel `update_dapur` dan pencatatan batch di `update_dapur_batch`.
  - Riwayat impor dengan fitur **Rollback / Hapus Batch** jika terjadi kekeliruan data.
- Modul **Audit Log**:
  - Tabel riwayat audit terpusat bagi Admin: melacak aksi `UPLOAD`, `DOWNLOAD`, `EDIT`, `HAPUS`, dan `PERSETUJUAN`.
  - Pencatatan otomatis waktu, pelaku, tenant SPPG, jenis tabel, dan detail perubahan.

### **Tahap 8: Pengujian Keamanan Hak Akses Multi-Tenant & Dokumentasi README**
- Pengujian isolasi data: memastikan akun SPPG A tidak dapat membaca/mengubah/mengunduh data SPPG B.
- Pengujian RLS Supabase langsung pada level API/SQL.
- Pengujian performa pemrosesan Excel berukuran besar.
- Penyusunan `README.md` komprehensif: panduan setup Supabase, konfigurasi variabel lingkungan `.env.local`, langkah eksekusi migrasi, pembuatan akun Admin pertama, dan instruksi menjalankan server lokal.

---

## 8. Analisis Risiko Teknis & Strategi Mitigasi

| Potensi Risiko Teknis | Dampak | Strategi Mitigasi |
|---|---|---|
| **Kebocoran Data Antar-SPPG (*Cross-Tenant Leak*)** | Data sensitif (gaji, kwitansi, LPDH) terlihat oleh SPPG lain. | **Pertahanan Berlapis (*Defense in Depth*)**: (1) RLS PostgreSQL di level database, (2) Helper function `SECURITY DEFINER` bebas rekursi, (3) Validasi sesi & tenant di Server Actions Next.js, (4) Filter eksplisit `WHERE sppg_id = ...` pada setiap query. |
| **Beban Memori (*Memory Spike*) saat Parse Excel Multi-Sheet** | Server crash atau timeout saat memproses file Excel besar. | Gunakan SheetJS/ExcelJS dengan parsing streaming atau optimasi buffer, membatasi ukuran file maksimum (maks. 25MB), serta validasi struktur sheet sebelum commit. |
| **Akses Langsung ke File di Supabase Storage** | Pengguna menebak URL file sensitif (kwitansi, LPDH, SK). | Seluruh bucket storage disetel `public = false`. File hanya dapat diakses melalui **Signed URL** berumur pendek (15-60 menit) yang di-generate setelah verifikasi hak akses di server. |
| **Rekursi Tak Hingga pada RLS PostgreSQL** | Query Supabase lambat (*hanging*) atau error 500 karena policy RLS memanggil tabel `profiles` atau `user_sppg` yang juga ber-RLS. | Pisahkan fungsi pengecekan role ke dalam fungsi khusus `is_admin()` dan `has_sppg_access()` dengan `SECURITY DEFINER` dan `SET search_path = public`, sehingga evaluasi policy tidak memicu siklus evaluasi berulang. |
| **Sinkronisasi Sesi Auth di Next.js Server & Client** | Pengguna terlempar ke login atau data tenant terlambat berubah. | Implementasikan `@supabase/ssr` dengan penanganan cookie terstandarisasi di `middleware.ts`, `createClient` server-side, dan client-side context provider. |
| **Kegagalan Batch Import pada Update Dapur** | Sebagian data tersimpan sebagian gagal, menyebabkan data inkonsisten. | Terapkan transaksi database (`BEGIN ... COMMIT`) atau pencatatan `batch_id` sehingga jika terjadi error atau Admin memilih "Rollback", seluruh baris dalam batch tersebut terhapus bersih. |

---

## 9. Langkah Selanjutnya

Silakan berikan persetujuan untuk memulai eksekusi **Tahap 1: Setup Proyek, Supabase, Auth, Role, dan Seed 42 SPPG**.
