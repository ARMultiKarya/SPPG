# SIDOK SPPG — Sistem Dokumen Operasional SPPG
### Kabupaten Magelang — 42 Satuan Pelayanan Pemenuhan Gizi (SPPG)

Aplikasi web **Multi-Tenant** modern berbasis **Next.js 14+ (App Router)** dan **Supabase PostgreSQL** untuk pengelolaan operasional terpadu, penggajian AK AG, kwitansi operasional, laporan pertanggungjawaban harian (LPDH), pemrosesan template Excel multi-sheet, dokumen legalitas KOP Yayasan, serta update harian dapur untuk 42 SPPG di Kabupaten Magelang.

---

## 🌟 Fitur Utama & Modul Sistem

1. **Arsitektur Multi-Tenant Terisolasi Ketat (RLS)**:
   - Data operasional dipisahkan per `sppg_id`.
   - **Role ADMIN**: Memantau dan mengelola seluruh 42 SPPG, dokumen yayasan, pemrosesan Excel master, dan persetujuan akses.
   - **Role SPPG**: Hanya memiliki hak akses ke data unit SPPG tempat pengguna terdaftar. Tidak dapat melihat atau mengunduh data SPPG lain.
2. **Dashboard Operasional**:
   - Tampilan agregasi menyeluruh bagi Admin (total unit, LPDH masuk, kwitansi, gaji, permohonan KOP).
   - Tampilan ringkasan khusus unit bagi akun SPPG.
3. **Modul Gaji AK AG**:
   - Formulir input honorarium/gaji per bulan: hari kerja, gaji pokok, tunjangan, potongan, dan total otomatis.
   - Tabel rekapitulasi gaji bulanan dan per SPPG.
   - Tombol **Ekspor Excel (.xlsx)** laporan gaji bulanan.
4. **Modul Kwitansi**:
   - 5 Kategori bawaan: BBM, Galon, Genset, Insentif, Kebersihan (+ kemampuan Admin menambah kategori baru).
   - Form pencatatan transaksi: tanggal, kategori, nominal otomatis Rupiah, keterangan, dan upload foto/PDF bukti kwitansi.
   - Rekap total per kategori dan ekspor data kwitansi ke Excel.
5. **Modul LPDH Harian**:
   - Komponen upload file Excel LPDH harian khusus format `.xlsx` dan `.xls`.
   - Penyimpanan berkas di Supabase Storage privat dan pencatatan metadata.
   - Pengunduhan berkas aman melalui temporary Signed URL.
6. **Modul Pengajian LPDH / Template Excel Multi-Sheet**:
   - Admin mengunggah template Excel dengan banyak sheet (contoh: *Oktober*, *November*, *Desember*).
   - Sistem otomatis memisahkan setiap sheet menjadi file `.xlsx` mandiri yang dinamai sesuai nama sheet aslinya (`Oktober.xlsx`, `November.xlsx`, `Desember.xlsx`).
   - Tampilan antarmuka berbasis tab interaktif dengan pratinjau data tabular dan unduhan per sheet.
7. **Modul KOP Yayasan & Alur Persetujuan Akses**:
   - Admin mengunggah dokumen yayasan (CV, SK, Profil Yayasan, dsb) dan menandai apakah *"Hanya Admin"* atau *"Dapat Diakses SPPG Setelah Persetujuan"*.
   - SPPG mengajukan permohonan akses dengan status: `MENUNGGU`, `DISETUJUI`, `DITOLAK`.
   - Admin meninjau permohonan, memberikan catatan resmi, dan menyetujui/menolak.
8. **Modul Update Dapur**:
   - Khusus Admin: mengunggah template Excel harian multi-sheet (Menu Pagi, Siang, Pengawasan Sanitasi).
   - Pratinjau data sebelum konfirmasi impor ke database.
   - Pencatatan riwayat impor batch lengkap dengan fitur **Rollback / Pembatalan Batch**.
9. **Log Audit Aktivitas Dokumen**:
   - Melacak seluruh aksi: `UPLOAD`, `DOWNLOAD`, `CREATE`, `UPDATE`, `DELETE`, `APPROVE`, `REJECT`, dan `ROLLBACK`.

---

## 🏛️ Daftar 42 SPPG Kabupaten Magelang yang Didukung

| No | Nama Unit SPPG | Kecamatan | No | Nama Unit SPPG | Kecamatan |
|---|---|---|---|---|---|
| 1 | SPPG ADIKARTO | MUNTILAN | 22 | SPPG KETEP | SAWANGAN |
| 2 | SPPG ARGOMULYO | SALATIGA | 23 | SPPG KRINCING | SECANG |
| 3 | SPPG BALEKERTO | KALIANGKRIK | 24 | SPPG MRANGGEN 2 | SRUMBUNG |
| 4 | SPPG BANYUSARI | TEGALREJO | 25 | SPPG NGADIHARJO | BOROBUDUR |
| 5 | SPPG BATURONO | SALAM | 26 | SPPG NGASINAN | GRABAG |
| 6 | SPPG BAWANG | PAKIS | 27 | SPPG NGAWEN | MUNTILAN |
| 7 | SPPG BESERAN | KALIANGKRIK | 28 | SPPG NOBOREJO | ARGOMULYO |
| 8 | SPPG BLEMBENG | TEGALREJO | 29 | SPPG PASURUHAN | MERTOYUDAN |
| 9 | SPPG BOJONG | MUNGKID | 30 | SPPG PLOSOGEDE | NGLUWAR |
| 10 | SPPG CANDISARI | SECANG | 31 | SPPG PODOSOKO | CANDIMULYO |
| 11 | SPPG DULMAS | TEGALREJO | 32 | SPPG SIDOGAWUNG | TEMPURAN |
| 12 | SPPG GANDUSARI | BANDONGAN | 33 | SPPG SIDOGEDE | GRABAG |
| 13 | SPPG GIRIKULON | SECANG | 34 | SPPG SIDOMULYO | SALAMAN |
| 14 | SPPG GIYANTI | CANDIMULYO | 35 | SPPG SIDOREJO | KIDUL |
| 15 | SPPG GONDOSULI | MUNTILAN | 36 | SPPG SRUMBUNG | MRANGGEN 1 |
| 16 | SPPG GONDOWANGI | SAWANGAN | 37 | SPPG SRUMBUNG | SRUMBUNG |
| 17 | SPPG JOGOMULYO | TEMPURAN | 38 | SPPG SUGIHMAS | GRABAG |
| 18 | SPPG JOGOYASAN | NGABLAK | 39 | SPPG SUMURARUM | GRABAG |
| 19 | SPPG KAJORAN | KAJORAN | 40 | SPPG SUROJOYO | CANDIMULYO |
| 20 | SPPG KALISALAK | SALAMAN | 41 | SPPG TRASAN | BANDONGAN |
| 21 | SPPG KAPONAN | PAKIS | 42 | SPPG WINDUSARI | WINDUSARI |

---

## 🚀 Panduan Setup & Instalasi

### 1. Prasyarat Sistem
- **Node.js**: v18.17+ atau v20+ LTS
- **npm** atau **pnpm**
- Akun **Supabase** (Cloud atau Local Self-Hosted)

### 2. Instalasi Dependensi
Jalankan perintah berikut di direktori proyek:
```bash
npm install
```

### 3. Konfigurasi Database Supabase
Buka **SQL Editor** pada dashboard Supabase proyek Anda, lalu jalankan ketiga file migrasi SQL secara berurutan:

1. **Migrasi 01 (Skema Tabel & Functions)**:
   Buka dan jalankan isi file `supabase/migrations/20261014000001_initial_schema.sql`.
   *Membuat 13 tabel, indeks performa, trigger update timestamp, trigger pendaftaran user, dan fungsi security definer `is_admin()` serta `user_has_sppg_access()`.*

2. **Migrasi 02 (Kebijakan Row Level Security & Storage)**:
   Buka dan jalankan isi file `supabase/migrations/20261014000002_rls_policies.sql`.
   *Mengaktifkan RLS pada seluruh tabel, mencegah cross-tenant leak, dan mengonfigurasi private storage buckets.*

3. **Migrasi 03 (Seed 42 SPPG & Data Awal)**:
   Buka dan jalankan isi file `supabase/migrations/20261014000003_seed_42_sppg.sql`.
   *Memasukkan seluruh data 42 SPPG Kab. Magelang dan kategori kwitansi default.*

### 4. Konfigurasi Environment Variables
Salin file `.env.example` menjadi `.env.local`:
```bash
cp .env.example .env.local
```
Sesuaikan nilainya dengan kredensial Supabase Anda:
```env
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxxxxxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 5. Membuat Akun Admin Pertama
1. Daftarkan akun baru melalui halaman aplikasi atau melalui Supabase Auth Dashboard (contoh email: `admin@sppg.magelangkab.go.id`).
2. Di **SQL Editor** Supabase, jalankan fungsi helper yang telah disiapkan:
   ```sql
   SELECT public.set_user_as_admin('admin@sppg.magelangkab.go.id');
   ```
3. Akun tersebut sekarang memiliki wewenang `ADMIN` penuh dan dapat mengakses seluruh 42 SPPG.

### 6. Menjalankan Aplikasi
Jalankan development server:
```bash
npm run dev
```
Akses aplikasi melalui peramban web di: `http://localhost:3000`.

---

## 🛡️ Pengujian Hak Akses Multi-Tenant (Demo & Verifikasi)

Aplikasi telah dilengkapi dengan **Uji Coba Hak Akses (Role Switcher)** di bagian kiri bawah sidebar:
1. **Pilih Akun Admin**:
   - Dropdown di header akan menampilkan **🌟 Semua SPPG (42 Unit)**. Admin dapat memilih unit mana saja untuk mengawasi atau menginput data.
   - Seluruh modul khusus Admin (Pengajian LPDH, Update Dapur, Audit Log, Kelola 42 SPPG) dapat diakses.
2. **Pilih Akun SPPG (Contoh: SPPG Adikarto)**:
   - Konteks terkunci secara otomatis ke unit SPPG miliknya.
   - Seluruh tabel gaji, kwitansi, dan LPDH hanya memuat data SPPG Adikarto.
   - Modul khusus Admin disembunyikan secara otomatis.
   - Mengajukan akses dokumen KOP Yayasan dan memantau status persetujuan dari Admin.

---

## 📦 Struktur Folder

```
src/
├── app/
│   ├── (auth)/login/page.tsx           # Halaman Login Multi-Role
│   ├── (dashboard)/layout.tsx          # App Shell (Sidebar & Header Responsif)
│   ├── (dashboard)/dashboard/page.tsx  # Modul 1: Dashboard Agregasi vs Unit
│   ├── (dashboard)/gaji/page.tsx       # Modul 2: Gaji AK AG & Ekspor Excel
│   ├── (dashboard)/kwitansi/page.tsx   # Modul 3: Kwitansi & Kategori
│   ├── (dashboard)/lpdh/page.tsx       # Modul 4: LPDH Harian (.xlsx/.xls)
│   ├── (dashboard)/pengajian-lpdh/     # Modul 5: Template Excel Multi-Sheet
│   ├── (dashboard)/kop-yayasan/        # Modul 6: KOP Yayasan & Persetujuan
│   ├── (dashboard)/update-dapur/       # Modul 7: Update Dapur & Batch Rollback
│   ├── (dashboard)/audit-log/          # Modul 8: Log Audit Aktivitas
│   └── (dashboard)/admin/sppg/         # Modul 9: Daftar 42 SPPG Magelang
├── components/
│   ├── layout/                         # Sidebar, Header, SPPG Switcher
│   └── providers/                      # AppContext multi-tenant state
├── lib/
│   ├── excel/                          # Splitter & Exporter SheetJS
│   ├── supabase/                       # Client, Server, Admin, Middleware
│   ├── audit.ts                        # Audit logger helper
│   └── mock-data.ts                    # Seed 42 SPPG & data contoh
└── types/                              # Definisi tipe TypeScript
```

---

## 📄 Lisensi
Hak Cipta © 2026 Pemerintah Kabupaten Magelang — Dinas Kesehatan / Tim Operasional SPPG.
