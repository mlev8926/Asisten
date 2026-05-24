# 💊 Interaktif Medical - Asisten Jadwal Pengobatan 10 Hari

**Aplikasi Web Interaktif untuk Manajemen & Monitoring Jadwal Minum Obat dengan Keselamatan Medis Berlapis**

![Version](https://img.shields.io/badge/version-4.1.0-blue) ![Language](https://img.shields.io/badge/language-HTML-orange) ![License](https://img.shields.io/badge/license-CC0%201.0-green) ![Status](https://img.shields.io/badge/status-Production%20Ready-brightgreen)

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

**Interaktif Medical** adalah asisten digital yang dirancang khusus untuk membantu pasien dalam menjaga kepatuhan pengobatan (medication adherence) selama 10 hari pengobatan. Aplikasi ini menggunakan siklus medis 24 jam yang dimulai dari **pukul 06:35 WIB** untuk memberikan jadwal yang konsisten dan mudah dipahami.

### Masalah yang Dipecahkan
- ❌ **Lupa minum obat** pada waktu yang tepat
- ❌ **Double-dosing** (minum dua kali dalam waktu dekat)
- ❌ **Kebingungan jadwal** terutama saat dini hari
- ❌ **Tidak ada log** kapan obat benar-benar diminum
- ❌ **Risiko keselamatan medis** akibat klik salah

### Solusi yang Ditawarkan
✅ Jadwal otomatis yang tepat waktu  
✅ Status real-time setiap obat  
✅ Sistem keselamatan berlapis untuk mencegah kesalahan  
✅ Log terverifikasi dengan timestamp  
✅ Antarmuka intuitif berbahasa Indonesia  

---

## ✨ Fitur Utama

### 1. **Monitoring Hari Ini (Siklus Aktif)**
- Tampilan **kartu responsif** untuk semua obat yang harus diminum hari ini
- Status real-time: `BELUM_WAKTUNYA` → `AKTIF` → `DIMINUM` / `TERLEWAT`
- Tombol konfirmasi interaktif dengan validasi keselamatan
- Countdown waktu (sisa berapa menit sebelum auto-skip)
- Badge terverifikasi dengan timestamp konfirmasi

### 2. **Tabel Master 10 Hari**
- **Grid dinamis** menampilkan seluruh jadwal 10 hari pengobatan
- Kolom otomatis berdasarkan slot waktu aktif yang unik
- Baris untuk setiap hari dengan status warna-kode
- Hari aktif saat ini ditandai dengan border highlight
- Interaktif: klik sel untuk konfirmasi status

### 3. **Manajemen Obat (CRUD)**
- **Tambah obat baru** dengan nama, dosis, dan frekuensi minum
- **Pilih aturan minum**: 1x1 (1×sehari), 2x1 (2×sehari), 3x1, 4x1
- **Smart automatic spacing** - sistem otomatis menghitung jam distribusi:
  - `1x1` → 24 jam jeda
  - `2x1` → 12 jam jeda (pagi & malam)
  - `3x1` → 8 jam jeda (pagi, siang, malam)
  - `4x1` → 6 jam jeda (pagi, siang, sore, dini hari)
- **Hapus obat** dengan konfirmasi keamanan
- **Reset ke default dokter** kapan saja

### 4. **Siklus Medis 24 Jam Cerdas**
- **Anchor waktu:** 06:35 pagi (konsisten setiap hari)
- **Dini hari offset:** Slot 00:35 malam dihitung sebagai bagian dari hari sebelumnya (meski kalender menunjukkan keesokan hari)
- **Transisi otomatis:** Ketika pukul 02:00 pagi, sistem beralih ke siklus hari berikutnya
- **Contoh:**
  ```
  Hari 1: 23 Mei 06:35 WIB → 24 Mei 00:35 WIB
  Hari 2: 24 Mei 06:35 WIB → 25 Mei 00:35 WIB
  ... dst sampai Hari 10
  ```

### 5. **Keselamatan Berlapis**
- **Batas toleransi ketat:** 30 menit dari jam target untuk **semua obat**
- **Read-only protection:** Status `BELUM_WAKTUNYA` dan `TERLEWAT` tidak bisa diubah
- **Modal keselamatan:** Konfirmasi ganda saat membatalkan obat yang sudah terverifikasi
- **Lock otomatis:** Jika lewat waktu grace period, jadwal otomatis dikunci sebagai TERLEWAT
- **Audio feedback:** Beep suara untuk peringatan dan konfirmasi

### 6. **Panduan & Informasi**
- **Tab Aturan Medis:** Penjelasan siklus 24 jam dan standar frekuensi
- **Tab Developer:** Profil pembuat & misi aplikasi
- **Tab Changelog:** Riwayat perubahan versi dan fitur baru

### 7. **Penyimpanan Lokal (Offline-Ready)**
- Semua data tersimpan di **localStorage browser**
- Tidak memerlukan koneksi internet setelah loading pertama
- Data persisten: tanggal mulai, daftar obat, status minum
- Sinkronisasi real-time di antara tab browser

---

## 🎯 Panduan Penggunaan

### **Langkah 1: Buka Aplikasi**
Buka file `index.html` di browser modern (Chrome, Firefox, Safari, Edge).

```
https://github.com/akseldwiartanto-del/Interaktif-medical/blob/main/index.html
```

Atau jika sudah di-deploy, akses via URL aplikasi web Anda.

### **Langkah 2: Atur Tanggal Mulai Hari 1**
1. Lihat panel atas: "Tanggal Mulai Hari 1 (Jam 06.35)"
2. Pilih tanggal saat pengobatan dimulai
3. Klik simpan - sistem akan menghitung 10 hari ke depan otomatis
4. Tanggal tersimpan di localStorage, tidak perlu set ulang setiap kali buka

### **Langkah 3: Kelola Obat (Opsional)**
Jika ingin mengubah resep:
1. Klik tombol **"Kelola Obat (CRUD)"** di kanan atas
2. **Tambah Obat Baru:**
   - Masukkan nama & dosis (contoh: "Paracetamol 500 mg")
   - Pilih aturan minum (1x1, 2x1, 3x1, atau 4x1)
   - Tentukan jam minum pertama (default 06:35)
   - Klik "Simpan"
3. **Hapus Obat:**
   - Klik tombol merah di baris obat
   - Konfirmasi di modal
4. **Reset ke Default:**
   - Klik "Pulihkan ke Default Dokter"
   - Semua obat kembali ke resep awal

### **Langkah 4: Monitor & Konfirmasi Minum**

#### **Tab "Monitor Hari Ini (Siklus Aktif)"**
- Tampilkan semua jadwal obat untuk hari/siklus aktif saat ini
- Setiap kartu menunjukkan:
  - **Jam minum** (contoh: 06:35)
  - **Tanggal kalender** obat tersebut
  - **Daftar obat** yang harus diminum
  - **Status badge** (warna-kode)
  - **Tombol aksi** jika sedang aktif

**Jenis Status:**
| Status | Warna | Arti | Aksi |
|--------|-------|------|------|
| **Belum Waktunya** | Abu-abu | Jadwal belum tiba | ❌ Terkunci |
| **AKTIF** | Kuning | Saatnya minum! | ✅ Klik konfirmasi |
| **Sudah Diminum** | Hijau | Obat terverifikasi | 📋 Tampilkan badge |
| **Terlewat** | Merah | Lewat waktu grace | ❌ Terkunci auto-skip |

**Cara Konfirmasi:**
1. Tunggu hingga waktu minum tiba (status berubah menjadi AKTIF, kuning)
2. Ambil dan minum obat sesuai dosis
3. Klik tombol "Ketuk untuk Konfirmasi Minum" pada kartu tersebut
4. Sistem akan mencatat waktu konfirmasi secara real-time
5. Status berubah menjadi "Sudah Diminum" dengan badge biru terverifikasi

#### **Tab "Master Tabel 10 Hari"**
- Lihat jadwal lengkap dalam format tabel grid
- Setiap kolom = satu slot waktu (06:35, 12:35, 18:35, dll)
- Setiap baris = satu hari pengobatan (Hari 1–10)
- Hari aktif saat ini disorot dengan border biru
- Klik sel untuk mengubah status (sama seperti tab Monitor)

### **Langkah 5: Pantau Statistik**
- Lihat panel "Total Kepatuhan Minum Obat (10 Hari)" di atas
- Counter otomatis update:
  - **Belum:** Jadwal belum tiba
  - **Aktif:** Menunggu konfirmasi
  - **Diminum:** Terverifikasi
  - **Skip:** Lewat waktu

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

### **Data Persistence**
```javascript
localStorage {
  'med_definitions_spacing_v4': JSON,      // Daftar obat
  'med_taken_db_spacing': JSON,            // Status minum
  'med_start_date_spacing': String         // Tanggal mulai
}
```

### **Architecture Highlights**
- **Real-time state evaluation engine** (evaluateSchedule)
- **Dynamic schedule generation** berdasarkan frekuensi obat
- **Timezone-locked** ke Asia/Jakarta (WIB)
- **Strict state machine** untuk transisi status
- **Safety validation** di setiap aksi user

---

## 🔒 Keselamatan Medis

Aplikasi ini dirancang dengan **standar keselamatan farmasi** untuk mencegah kesalahan obat:

### **1. Batas Toleransi Ketat (30 Menit)**
- Semua obat harus diminum dalam **30 menit dari jam target**
- Contoh: Target 06:35 → deadline 07:05
- Setelah deadline, status otomatis **TERLEWAT** dan **TERKUNCI**
- **Tidak ada pengecualian** untuk obat kritis atau umum

### **2. Pencegahan Double-Dosing**
- Sekali obat ditandai "DIMINUM", tidak bisa diubah kembali ke aktif
- Jika user ingin membatalkan verifikasi:
  - Modal keselamatan muncul dengan peringatan
  - Harus klik tombol konfirmasi dua kali
  - Obat akan dikunci sebagai TERLEWAT (skip), bukan aktif kembali

### **3. Read-Only Protection**
- Status `BELUM_WAKTUNYA` (belum waktu): **❌ Tidak bisa diinteraksi**
- Status `TERLEWAT` (lewat waktu): **❌ Tidak bisa diinteraksi**
- Hanya status `AKTIF` yang bisa diklik untuk konfirmasi
- Lock visual: `cursor: not-allowed`, warna muted, ikon gembok

### **4. Siklus Medis Konsisten (06:35 Anchor)**
- Satu "Hari Pengobatan" = 24 jam dari **06:35 pagi hari ini** hingga **06:35 pagi besok**
- Slot dini hari (00:35) tetap bagian dari hari yang sama, meski kalender menunjukkan keesokan hari
- **Transisi otomatis pukul 02:00 pagi** ke siklus hari berikutnya
- Mencegah kebingungan jadwal saat malam/dini hari

### **5. Audio & Visual Alerts**
- **Beep ganda** saat obat baru menjadi AKTIF (notifikasi penting)
- **Beep rendah** saat user coba aksi terlarang
- **Toast notification** di sudut kanan bawah untuk feedback
- **Badge terverifikasi biru** untuk status terconfirmasi

### **Compliance dengan Standar Farmasi:**
- ✅ No unauthorized state transitions
- ✅ Strict grace period (30 min)
- ✅ Immutable taken records
- ✅ Medical day cycle consistency
- ✅ Prevents accidental double-dosing
- ✅ Audit trail (timestamp logs)

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
3. Akses aplikasi di: `https://username.github.io/Interaktif-medical/`

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

**Solusi:** Download file JSON hasil export sebelum hapus cache.

### **Q: Bisa diakses offline?**
**A:** Ya! Setelah membuka aplikasi sekali, Anda bisa akses offline tanpa internet. Semua data tersimpan lokal.

### **Q: Bagaimana jika ada kesalahan saat konfirmasi obat?**
**A:** Jika status sudah "Diminum" tapi sebenarnya salah:
1. Klik kartu/sel tersebut lagi
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
- Atau setup cloud sync (future feature)

### **Q: Berapa lama masa berlaku 10 hari?**
**A:** 10 hari = 10 × 24 jam = 240 jam pengobatan. Setelah Hari 10 selesai (jam 06:35), counter berakhir dan aplikasi menampilkan "Selesai 10 Hari". Jika perlu lanjut, atur tanggal mulai yang baru.

### **Q: Obat berapa banyak yang bisa ditambah?**
**A:** Tidak ada batasan teknis. Rekomendasi: maksimal 5–10 obat aktif simultan untuk kemudahan UI.

### **Q: Timezone harus WIB (Jakarta)?**
**A:** Ya, aplikasi ini **locked ke WIB (UTC+7)**. Waktu sistem selalu ditampilkan dalam WIB terlepas dari zona browser user.

---

## 👨‍💻 Pengembang

### **Profil Developer**
- **Nama:** mlev@2026
- **Email:** mlevian@protonmail.com
- **Misi:** "Kepatuhan dalam pengobatan bukan hanya mematuhi resep, melainkan komitmen harian yang menjaga harapan hidup."

### **Versi Terbaru: v4.1.0**
**Fitur Terbaru:**
- ✅ Perlindungan modal keselamatan untuk pembatalan status
- ✅ Transisi otomatis siklus hari setelah pukul 02:00 pagi
- ✅ Tab Profil Developer dengan kontak
- ✅ Changelog lengkap versi-per-versi

### **Riwayat Versi**
```
v4.1.0 (2026-05-23) - Revisi Terakhir
  • Integrasi modal keselamatan medis
  • Transisi otomatis siklus hari
  • Tab developer & profil

v4.0.0 - Revisi Batas Kedaluwarsa Ketat
  • Ekspiry gate 30 menit untuk semua obat

v3.0.0 - Revisi Lencana Premium
  • Ganti toggle button → Verified Badge
  • Real-time timestamp logging

v2.0.0 - Revisi CRUD & Smart Spacing
  • Fungsi CRUD dinamis
  • Auto-spacing berdasar frekuensi

v1.0.0 - Rilis Awal
  • Konsep jadwal obat 10 hari
```

### **Lisensi**
Aplikasi ini dilisensikan di bawah **Creative Commons Zero v1.0 Universal (CC0)**, artinya:
- ✅ Gratis untuk digunakan, modify, distribusikan
- ✅ Tidak ada atribusi wajib
- ✅ Public domain

---

## 🔗 Links & Resources

- **Repository:** [GitHub](https://github.com/akseldwiartanto-del/Interaktif-medical)
- **Issue Tracker:** [GitHub Issues](https://github.com/akseldwiartanto-del/Interaktif-medical/issues)
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

---

**Semoga Anda segera pulih! 💚**

*Dibuat dengan perhatian untuk keselamatan pasien.*

Last Updated: 2026-05-23 | Status: ✅ Production Ready