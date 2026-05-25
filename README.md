# 💊 Interaktif Medical - Asisten Jadwal Pengobatan 10 Hari

**Aplikasi Web Interaktif untuk Manajemen & Monitoring Jadwal Minum Obat dengan Keselamatan Medis Berlapis**

![Version](https://img.shields.io/badge/version-5.1.0-blue) ![Language](https://img.shields.io/badge/language-HTML-orange) ![License](https://img.shields.io/badge/license-CC0%201.0-green) ![Status](https://img.shields.io/badge/status-Production%20Ready-brightgreen)

---

## 📋 Daftar Isi

1. [Tentang Aplikasi](#tentang-aplikasi)
2. [Fitur Utama](#fitur-utama)
3. [Panduan Penggunaan](#panduan-penggunaan)
4. [Teknologi & Arsitektur](#teknologi--arsitektur)
5. [Keselamatan Medis](#keselamatan-medis)
6. [Persyaratan Sistem](#persyaratan-sistem)
7. [Instalasi & Pendeployan](#instalasi--pendeployan)
8. [FAQ](#faq)
9. [Pengembang](#pengembang)

---

## 🏥 Tentang Aplikasi

**Interaktif Medical** adalah asisten digital yang dirancang khusus untuk membantu pasien dalam menjaga kepatuhan pengobatan (medication adherence) selama 10 hari pengobatan. Aplikasi ini menggunakan teknologi terkini dengan fokus pada **keselamatan medis berlapis** dan **manajemen multi-pasien**.

### Masalah yang Dipecahkan
- ❌ **Lupa minum obat** pada waktu yang tepat
- ❌ **Double-dosing** (minum dua kali dalam waktu dekat)
- ❌ **Kebingungan jadwal** terutama saat dini hari
- ❌ **Tidak ada log** kapan obat benar-benar diminum
- ❌ **Risiko keselamatan medis** akibat klik salah
- ❌ **Distribusi obat tidak terverifikasi** per medicine (NEW - v5.1)

### Solusi yang Ditawarkan
✅ Jadwal otomatis yang tepat waktu  
✅ Status real-time setiap obat  
✅ Sistem keselamatan berlapis untuk mencegah kesalahan  
✅ Log terverifikasi dengan timestamp  
✅ Antarmuka intuitif berbahasa Indonesia  
✅ **Konfirmasi obat terstruktur per medicine** (NEW - v5.1)  
✅ **Multi-pasien dengan isolasi data** (v5.0)

---

## ✨ Fitur Utama

### 1. **Monitoring Hari Ini (Siklus Aktif)** - UPDATED v5.1
- Tampilan **kartu responsif** untuk semua obat yang harus diminum hari ini
- Status real-time: `BELUM_WAKTUNYA` → `AKTIF` → `DIMINUM` / `TERLEWAT`
- **Tombol "Konfirmasi Minum"** bukan direct toggle (NEW v5.1)
- Countdown waktu (sisa berapa menit sebelum auto-skip)
- Badge terverifikasi dengan timestamp konfirmasi

**Workflow Konfirmasi Obat (NEW - v5.1):**
1. Saat jadwal obat masuk status AKTIF (kuning), tombol "Konfirmasi Minum" muncul
2. Klik tombol → Modal checklist terbuka
3. Untuk setiap obat, pilih status: **✓ Diberikan / ✗ Tidak Diberikan / ⊘ Distop**
4. Klik "Verifikasi & Simpan" → Status tersimpan, slot berubah ke DIMINUM (hijau)

### 2. **Tabel Master 10 Hari**
- **Grid dinamis** menampilkan seluruh jadwal 10 hari pengobatan
- Kolom otomatis berdasarkan slot waktu aktif yang unik
- Baris untuk setiap hari dengan status warna-kode
- Hari aktif saat ini ditandai dengan border highlight
- Interaktif: lihat status ringkas setiap slot

### 3. **Manajemen Obat (CRUD)**
- **Tambah obat baru** dengan nama, dosis, dan frekuensi minum
- **Pilih aturan minum**: 1x1 (1×sehari), 2x1 (2×sehari), 3x1, 4x1
- **Smart automatic spacing** - sistem otomatis menghitung jam distribusi:
  - `1x1` → 24 jam jeda (contoh: 20:00)
  - `2x1` → 12 jam jeda (contoh: 06:35, 18:35)
  - `3x1` → 8 jam jeda (contoh: 06:35, 14:35, 22:35)
  - `4x1` → 6 jam jeda (contoh: 06:35, 12:35, 18:35, 00:35)
- **Hapus obat** dengan konfirmasi keamanan
- **Reset ke default dokter** kapan saja

### 4. **Siklus Medis 24 Jam Cerdas** - FIXED v5.1
- **Anchor waktu:** 06:35 pagi (konsisten setiap hari)
- **Dini hari offset:** Slot 00:35 malam dihitung sebagai bagian dari hari sebelumnya
- **Rotasi Hari DIPERBAIKI:** Transisi pada pukul **00:00:01** (bukan 02:00)
- **Slot 00:35 tetap di periode medis sebelumnya** meskipun kalender berubah
- **Contoh:**
  ```
  Hari 1: 23 Mei 06:35 WIB → 24 Mei 06:35 WIB
    - Slot 06:35 pada 23 Mei = Hari 1
    - Slot 00:35 pada 24 Mei = TETAP Hari 1 (bukan Hari 2)
    - Rotasi ke Hari 2 saat clock hit 00:00:01 pada 24 Mei
  
  Hari 2: 24 Mei 06:35 WIB → 25 Mei 06:35 WIB
  ```

### 5. **Keselamatan Berlapis** - ENHANCED v5.1
- **Batas toleransi ketat:** 30 menit dari jam target untuk **semua obat**
- **Read-only protection:** Status `BELUM_WAKTUNYA` dan `TERLEWAT` tidak bisa diubah
- **Modal keselamatan:** Konfirmasi ganda saat membatalkan obat
- **Lock otomatis:** Jika lewat waktu grace period, jadwal otomatis dikunci sebagai TERLEWAT
- **Konfirmasi terstruktur:** Checklist per medicine dengan status distribusi (NEW v5.1)
- **Audio feedback:** Beep suara untuk peringatan dan konfirmasi

### 6. **Manajemen Multi-Pasien** (v5.0)
- **Patient Selector:** Dropdown cepat di header untuk switch pasien
- **Patient Management Modal:** Daftar semua pasien, cari, tambah baru
- **Auto-Generated RM:** Format `RM-YYYY[10-digit]` (contoh: RM-2026000000001)
- **Search:** Cari nama atau 3 digit RM terakhir
- **Isolasi Data:** Setiap pasien punya database terpisah di localStorage

### 7. **Pemantauan TTV (Tanda-Tanda Vital)**
- **Form entry:** Tensi, Nadi, Nafas, SpO2, Suhu, Catatan klinis
- **Riwayat klinis:** Tabel dengan edit/delete (max 1x edit per entry)
- **Timestamp WIB:** Semua entry tercatat dengan waktu otomatis

### 8. **Panduan & Informasi**
- **Tab Aturan Medis:** Penjelasan siklus 24 jam dan standar frekuensi (UPDATED v5.1)
- **Tab Developer:** Profil pembuat & misi aplikasi
- **Tab Changelog:** Riwayat perubahan versi (dengan v5.1 entries)

### 9. **Penyimpanan Lokal (Offline-Ready)**
- Semua data tersimpan di **localStorage browser**
- Tidak memerlukan koneksi internet setelah loading pertama
- Data persisten: tanggal mulai, daftar obat, status minum, status distribusi obat (NEW v5.1)
- Sinkronisasi real-time di antara tab browser

---

## 🎯 Panduan Penggunaan

### **Langkah 1: Buka Aplikasi**
Buka file `index.html` di browser modern (Chrome, Firefox, Safari, Edge).

### **Langkah 2: Buat atau Pilih Pasien**
1. Klik tombol **"Pasien"** di kanan atas
2. **Tab "Daftar Pasien"**: Lihat daftar, cari, atau pilih pasien
3. **Tab "Tambah Pasien Baru"**: Masukkan nama → RM auto-generate → Simpan
4. Pasien sekarang aktif (terlihat di "Pilih Pasien" dropdown)

### **Langkah 3: Atur Tanggal Mulai Hari 1**
1. Lihat panel atas: "Tanggal Mulai Hari 1"
2. Pilih tanggal saat pengobatan dimulai (contoh: 23 Mei 2026)
3. Sistem otomatis menghitung 10 hari ke depan
4. Data tersimpan per pasien (tidak perlu set ulang)

### **Langkah 4: Kelola Obat (Opsional)**
Jika ingin mengubah resep:
1. Klik tombol **"Obat"** di kanan atas
2. **Tambah Obat Baru:**
   - Masukkan nama & dosis (contoh: "Paracetamol 500 mg")
   - Pilih aturan minum (1x1, 2x1, 3x1, atau 4x1)
   - Tentukan jam minum pertama (default 06:35)
   - Klik "Simpan"
3. **Hapus Obat:** Klik tombol merah di baris obat → Konfirmasi
4. **Reset ke Default:** Klik "Pulihkan ke Default Dokter"

### **Langkah 5: Monitor & Konfirmasi Minum (NEW - v5.1)**

#### **Tab "Monitor Hari Ini (Siklus Aktif)"**
- Tampilkan semua jadwal obat untuk hari aktif saat ini
- Setiap kartu menunjukkan:
  - **Jam minum** (contoh: 06:35)
  - **Tanggal kalender** obat tersebut
  - **Daftar obat** yang harus diminum
  - **Status badge** (warna-kode)
  - **Tombol aksi** jika sedang AKTIF

**Jenis Status:**
| Status | Warna | Arti | Aksi |
|--------|-------|------|------|
| **Belum Waktunya** | Abu-abu | Jadwal belum tiba | ❌ Terkunci |
| **AKTIF** | Kuning | Saatnya minum! | ✅ Klik "Konfirmasi Minum" |
| **Sudah Diminum** | Hijau | Obat terverifikasi | 📋 Tampilkan badge |
| **Terlewat** | Merah | Lewat waktu grace | ❌ Terkunci auto-skip |

**Cara Konfirmasi (NEW v5.1):**
1. Tunggu hingga waktu minum tiba (status berubah menjadi AKTIF, kuning)
2. Ambil dan minum obat sesuai dosis
3. Klik tombol **"Konfirmasi Minum"** pada kartu tersebut
4. **Modal Checklist** terbuka:
   - Untuk setiap obat di slot itu, pilih status:
     - ✓ **Diberikan** (obat diberikan ke pasien)
     - ✗ **Tidak Diberikan** (obat tidak diberikan)
     - ⊘ **Distop** (obat dihentikan/tidak diberikan)
5. Klik **"Verifikasi & Simpan"**
6. Sistem mencatat waktu konfirmasi dan status setiap medicine
7. Status kartu berubah menjadi **"Sudah Diminum"** (hijau) dengan badge

#### **Tab "Master Tabel 10 Hari"**
- Lihat jadwal lengkap dalam format tabel grid
- Setiap kolom = satu slot waktu (06:35, 12:35, 18:35, dll)
- Setiap baris = satu hari pengobatan (Hari 1–10)
- Hari aktif saat ini disorot dengan border biru
- Ringkas status setiap slot (BLM/AKT/DIM/TER)

### **Langkah 6: Pantau Statistik**
- Lihat panel **"Total Kepatuhan Minum Obat (10 Hari)"** di atas
- Counter otomatis update:
  - **Belum:** Jadwal belum tiba
  - **Aktif:** Menunggu konfirmasi
  - **Diminum:** Terverifikasi
  - **Skip:** Lewat waktu

### **Langkah 7: Monitor Tanda-Tanda Vital (TTV)**
1. Klik tab **"Pemantauan TTV"**
2. **Form di sisi kiri:**
   - Tensi (mmHg): Sistolik/Diastolik
   - Nadi (bpm)
   - Nafas (x/mnt)
   - SpO2 (%)
   - Suhu (°C, opsional)
   - Catatan kondisi
3. Klik **"Simpan TTV"** → Tersimpan di tabel riwayat
4. Edit (max 1x): Klik tombol edit pada entri
5. Delete: Klik tombol delete

---

## 🛠️ Teknologi & Arsitektur

### **Single-File Architecture**
Seluruh aplikasi dalam **satu file `index.html`** tanpa dependensi eksternal:
- ✅ Zero setup required
- ✅ Deploy dimana saja (local, web server, GitHub Pages)
- ✅ Portable dan mudah dishare

### **Frontend Stack**
- **HTML5:** Semantic markup, form elements
- **CSS3:** Inline styling, Bootstrap 5.3.2 framework, responsive design
- **Vanilla JavaScript:** Pure ES6+, no jQuery or frameworks
- **Bootstrap 5.3.2 & Icons:** CDN import, UI components

### **Data Persistence** (Updated v5.1)
```javascript
localStorage {
  'patients_registry': JSON,              // Daftar pasien global
  'active_patient_id': String,            // Pasien aktif
  
  // Per-patient (patient_{id}_{dataType})
  'patient_P-2026-0000001_meds': JSON,           // Daftar obat
  'patient_P-2026-0000001_taken': JSON,          // Status minum
  'patient_P-2026-0000001_vitals': JSON,         // Data TTV
  'patient_P-2026-0000001_medicineStatus': JSON, // Status distribusi per medicine (NEW v5.1)
  'patient_P-2026-0000001_startDate': String     // Tanggal mulai
}
```

### **Architecture Highlights**
- **Real-time state evaluation engine** (evaluateSchedule)
- **Dynamic schedule generation** berdasarkan frekuensi obat
- **Timezone-locked** ke Asia/Jakarta (WIB)
- **Strict state machine** untuk transisi status
- **Safety validation** di setiap aksi user
- **Multi-patient data isolation** (v5.0)
- **Structured medicine confirmation** (NEW v5.1)

---

## 🔒 Keselamatan Medis

Aplikasi ini dirancang dengan **standar keselamatan farmasi** untuk mencegah kesalahan obat:

### **1. Batas Toleransi Ketat (30 Menit)**
- Semua obat harus diminum dalam **30 menit dari jam target**
- Contoh: Target 06:35 → deadline 07:05
- Setelah deadline, status otomatis **TERLEWAT** dan **TERKUNCI**
- **Tidak ada pengecualian** untuk obat kritis atau umum

### **2. Pencegahan Double-Dosing**
- Sekali obat ditandai "DIMINUM", tidak bisa diubah kembali
- **Konfirmasi terstruktur** (v5.1): Setiap medicine harus dikonfirmasi status-nya
- Jika user ingin membatalkan verifikasi:
  - Modal keselamatan muncul dengan peringatan
  - Harus klik tombol konfirmasi dua kali
  - Obat akan dikunci sebagai TERLEWAT (skip), bukan aktif kembali

### **3. Konfirmasi Obat Terstruktur (NEW - v5.1)**
- Sebelum marking slot sebagai "DIMINUM", user harus:
  - Membuka modal checklist
  - Memilih status **untuk setiap medicine individual**:
    - ✓ **Diberikan** (obat diberikan)
    - ✗ **Tidak Diberikan** (obat tidak diberikan/tidak ada)
    - ⊘ **Distop** (obat dihentikan atas instruksi medis)
  - Status tersimpan per medicine per slot
  - Audit trail lengkap: siapa, kapan, status apa

### **4. Read-Only Protection**
- Status `BELUM_WAKTUNYA` (belum waktu): **❌ Tidak bisa diinteraksi**
- Status `TERLEWAT` (lewat waktu): **❌ Tidak bisa diinteraksi**
- Hanya status `AKTIF` yang bisa diklik untuk konfirmasi
- Lock visual: `cursor: not-allowed`, warna muted, ikon gembok

### **5. Siklus Medis Konsisten (06:35 Anchor)**
- Satu "Hari Pengobatan" = 24 jam dari **06:35 pagi hari ini** hingga **06:35 pagi besok**
- Slot dini hari (00:35) **tetap bagian dari hari yang sama**, meski kalender menunjukkan keesokan hari
- **Rotasi otomatis pukul 00:00:01** (v5.1) ke siklus hari berikutnya
- Mencegah kebingungan jadwal saat malam/dini hari

### **6. Rotasi Hari Diperbaiki (NEW - v5.1)**
- **OLD (v5.0):** Rotasi pada pukul 02:00 → Slot 00:35-06:35 masih di hari lama
- **NEW (v5.1):** Rotasi pada pukul 00:00:01 → Lebih intuitif dengan perubahan kalender
- **Slot 00:35 TETAP di periode sebelumnya** (tidak otomatis naik hari)

### **7. Audio & Visual Alerts**
- **Beep ganda** saat obat baru menjadi AKTIF (notifikasi penting)
- **Beep rendah** saat user coba aksi terlarang
- **Toast notification** di sudut kanan bawah untuk feedback
- **Badge terverifikasi hijau** untuk status terconfirmasi dengan timestamp

### **Compliance dengan Standar Farmasi:**
- ✅ No unauthorized state transitions
- ✅ Strict grace period (30 min)
- ✅ Immutable taken records
- ✅ Medical day cycle consistency
- ✅ Prevents accidental double-dosing
- ✅ Audit trail (timestamp + distribution status per medicine)
- ✅ **Structured medicine confirmation** (NEW v5.1)

---

## 💻 Persyaratan Sistem

### **Browser Kompatibel**
- ✅ Google Chrome 90+
- ✅ Mozilla Firefox 88+
- ✅ Apple Safari 14+
- ✅ Microsoft Edge 90+

### **Perangkat**
- ✅ Desktop (Windows, Mac, Linux)
- ✅ Tablet (iPad, Android tablets)
- ✅ Mobile (iPhone, Android phones)
- ✅ Responsive design untuk semua ukuran layar

### **Persyaratan Software**
- ✅ Tidak perlu instalasi
- ✅ Tidak perlu database
- ✅ Browser modern dengan JavaScript enabled
- ✅ localStorage support (semua browser modern)

### **Koneksi Internet**
- ✅ Diperlukan untuk loading pertama
- ✅ Setelahnya: offline-ready (tidak perlu internet untuk operasi)

---

## 🚀 Instalasi & Pendeployan

### **Opsi 1: Buka Lokal (Paling Mudah)**
1. Download file `index.html`
2. Buka di browser:
   ```
   File → Open → pilih index.html
   ```
   atau double-click file tersebut

### **Opsi 2: Deploy ke GitHub Pages**
1. Fork repository ini
2. Settings → Pages → Source = `main branch`
3. Akses aplikasi di: `https://username.github.io/Asisten/`

### **Opsi 3: Deploy ke Web Server**
1. Pindahkan `index.html` ke web server (Apache, Nginx, etc)
2. Akses via URL aplikasi Anda
3. Contoh:
   ```
   https://yourserver.com/index.html
   ```

### **Opsi 4: Embed di Aplikasi Lain**
- Copy paste HTML ke dalam aplikasi web Anda
- Atau gunakan `<iframe>` jika ingin isolasi data:
   ```html
   <iframe src="path/to/index.html"></iframe>
   ```

---

## ❓ FAQ

### **Q: Data saya hilang, bagaimana cara restore?**
**A:** Data tersimpan di localStorage browser. Jika:
- **Hapus browser history:** Data hilang permanen (backup localStorage secara manual)
- **Ganti browser/device:** Data tidak tersinkronisasi (satu perangkat = satu database)
- **Clear cache:** Jika juga clear localStorage, data hilang

**Solusi:** Download file JSON hasil export sebelum hapus cache (fitur export: TBD di v5.2).

### **Q: Bisa diakses offline?**
**A:** Ya! Setelah membuka aplikasi sekali, Anda bisa akses offline tanpa internet. Semua data tersimpan lokal.

### **Q: Bagaimana jika ada kesalahan saat konfirmasi obat? (NEW v5.1)**
**A:** Jika status sudah "Diminum" tapi sebenarnya salah:
1. Klik kartu/sel tersebut lagi (jika masih dalam tolerance window)
2. Modal keselamatan akan muncul
3. Klik "Ya, Batalkan & Kunci"
4. Status berubah ke "Terlewat" dan **terkunci** (tidak bisa diubah lagi)
5. Hubungi dokter untuk instruksi lebih lanjut

### **Q: Apakah aman menyimpan data medis di browser?**
**A:** localStorage adalah **client-side storage**, artinya:
- Data **tidak dikirim ke server** manapun
- Hanya tersimpan di device Anda sendiri
- Untuk keamanan lebih, gunakan private browsing
- **Jangan** buka di komputer publik tanpa clear cache setelahnya

### **Q: Bisa sinkronisasi antar device?**
**A:** Saat ini belum. Setiap device memiliki database terpisah. Untuk sinkronisasi:
- Buka aplikasi di device yang sama
- Atau setup cloud sync (direncanakan v5.2 dengan Firebase)

### **Q: Berapa lama masa berlaku 10 hari?**
**A:** 10 hari = 10 × 24 jam = 240 jam pengobatan. Setelah Hari 10 selesai (jam 06:35), counter berakhir dan aplikasi menampilkan "Selesai 10 Hari". Jika perlu lanjut, atur tanggal mulai yang baru.

### **Q: Obat berapa banyak yang bisa ditambah?**
**A:** Tidak ada batasan teknis. Rekomendasi: maksimal 5–10 obat aktif simultan untuk kemudahan UI.

### **Q: Timezone harus WIB (Jakarta)?**
**A:** Ya, aplikasi ini **locked ke WIB (UTC+7)**. Waktu sistem selalu ditampilkan dalam WIB terlepas dari zona browser user.

### **Q: Bagaimana cara mengubah status obat setelah disimpan? (NEW v5.1)**
**A:** Status distribusi obat per medicine tersimpan dalam modal konfirmasi. Jika perlu ubah:
1. Slot harus masih dalam status AKTIF (dalam 30 menit dari target time)
2. Klik tombol "Konfirmasi Minum" lagi
3. Modal checklist terbuka dengan status sebelumnya
4. Ubah dropdown → Klik "Verifikasi & Simpan"
5. Jika sudah TERLEWAT/DIMINUM, **tidak bisa diubah** (proteksi keselamatan)

---

## 👨‍💻 Pengembang

### **Profil Developer**
- **Nama:** mlev@2026
- **Email:** mlevian@protonmail.com
- **Misi:** "Kepatuhan dalam pengobatan bukan hanya mematuhi resep, melainkan komitmen harian yang menjaga harapan hidup."

### **Versi Terbaru: v5.1.0** (2026-05-25)
**Fitur Terbaru:**
- ✅ **Konfirmasi obat terstruktur** dengan checklist per medicine
- ✅ **Status distribusi:** Diberikan / Tidak Diberikan / Distop
- ✅ **Rotasi hari diperbaiki:** 00:00:01 (bukan 02:00)
- ✅ **Slot 00:35 tetap di periode sebelumnya**
- ✅ Audit trail per medicine per slot
- ✅ Multi-patient data isolation maintained

### **Riwayat Versi**
```
v5.1.0 (2026-05-25) - Medicine Confirmation & Day Rotation Fix
  • Konfirmasi obat dengan checklist (Diberikan/Tidak Diberikan/Distop)
  • Rotasi hari pada 00:00:01 (bukan 02:00)
  • Slot 00:35 tetap di periode medis sebelumnya
  • Medicine status database per patient
  • Audit trail terstruktur per medicine

v5.0.0 (2026-05-24) - Multi-Patient Architecture
  • Patient Registry dengan RM auto-generation
  • Patient-scoped data isolation (localStorage namespace)
  • Patient selector & management modal
  • Search by name / 3 digit RM
  • Full patient data portability

v4.2.0 - Monitor TTV
  • Tab Pemeriksaan Tanda-Tanda Vital (TTV) Mandiri
  • Batasan ketat: Maksimal 1x Edit per rekaman

v4.1.0 - Safety Modal & Developer Info
  • Integrasi modal keselamatan medis
  • Tab Developer & profil pembuat
  • Changelog lengkap versi-per-versi

v4.0.0 - Revisi Batas Kedaluwarsa Ketat
  • Ekspiry gate 30 menit untuk semua obat
  • Read-only enforcement untuk BELUM_WAKTUNYA & TERLEWAT

v3.0.0 - Revisi Lencana Premium
  • Ganti toggle button → Verified Badge
  • Real-time timestamp logging

v2.0.0 - Revisi CRUD & Smart Spacing
  • Fungsi CRUD dinamis
  • Auto-spacing berdasar frekuensi

v1.0.0 - Rilis Awal
  • Konsep jadwal obat 10 hari
  • State machine dasar
```

### **Lisensi**
Aplikasi ini dilisensikan di bawah **Creative Commons Zero v1.0 Universal (CC0)**, artinya:
- ✅ Gratis untuk digunakan, modify, distribusikan
- ✅ Tidak ada atribusi wajib
- ✅ Public domain

---

## 🔗 Links & Resources

- **Repository:** [GitHub](https://github.com/mlev8926/Asisten)
- **Issue Tracker:** [GitHub Issues](https://github.com/mlev8926/Asisten/issues)
- **Live Demo:** [GitHub Pages](https://mlev8926.github.io/Asisten/)
- **Skill Specification:** Lihat file `skill.md` untuk detail teknis lengkap
- **Developer Contact:** mlevian@protonmail.com

---

## 📝 Catatan Penting

### **Disclaimer Medis**
⚠️ Aplikasi ini adalah **asisten pengingat**, bukan pengganti resep dokter atau saran medis profesional. Selalu:
- Konsultasikan dengan dokter atau apoteker
- Ikuti petunjuk dosis resmi
- Jangan ubah jadwal tanpa persetujuan dokter
- Laporkan efek samping ke penyedia layanan kesehatan

### **Tanggung Jawab User**
- Aplikasi hanya **mengingat** jadwal, bukan diagnosis medis
- Keputusan minum obat ada di tangan user
- Hubungi dokter untuk perubahan dosis atau reaksi alergi
- **Konfirmasi obat via checklist (v5.1) adalah tanggung jawab pengguna**, bukan sistem

### **Data Privacy**
- Data tersimpan **lokal di browser saja** (tidak ada cloud/server)
- Untuk keamanan maksimal, gunakan private browsing atau perangkat pribadi
- Clear cache & cookies jika membuka di komputer publik

---

**Semoga Anda segera pulih! 💚**

*Dibuat dengan perhatian untuk keselamatan pasien.*

---

**Last Updated:** 2026-05-25 | **Status:** ✅ Production Ready v5.1.0  
**Compliance:** ✅ Medical Safety Standards | ✅ Multi-Patient | ✅ Structured Confirmation
