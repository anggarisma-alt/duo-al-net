# PANDUAN INSTALASI — DUO AL NET (Backend GAS + Frontend GitHub Pages)

Arsitektur baru:

```
HP / Komputer  ──►  GitHub Pages (tampilan: index.html, css/, js/)
                         │  fetch (JSON)
                         ▼
                Google Apps Script (/exec) = API  ──►  Google Sheets (data)
```

Data pelanggan, tagihan, dan pembayaran Anda **tetap di Spreadsheet yang sama**. Yang berubah hanya cara aplikasi ditampilkan dan dipanggil.

**Keuntungan:** tampilan jauh lebih cepat (data diambil dari cache perangkat dulu), dan Web Bluetooth untuk printer thermal bekerja lebih baik karena halaman tidak lagi berada di dalam iframe Google.

---

## BAGIAN A — Backend: pasang di Google Apps Script

### Jika Anda SUDAH memakai DUO AL NET versi lama (migrasi)

1. Buka proyek Apps Script lama Anda (script.google.com → proyek DUO AL NET).
2. **Hapus** 3 file lama di sisi kiri: `Index`, `Stylesheet`, `JavaScript` (klik titik tiga di samping nama file → Delete). Backend baru tidak memakainya.
3. Buka `Kode.gs`, **hapus semua isinya**, lalu **tempel isi `Kode.gs` yang baru**. Klik 💾 Simpan.
4. (Disarankan) Klik ⚙️ **Project Settings** → centang **Show "appsscript.json" manifest file in editor**. Buka `appsscript.json`, ganti isinya dengan file `appsscript.json` dari paket ini, simpan.
5. Di bagian atas, pilih fungsi **`setupAppEnvironment`** → klik **▶ Run**. Aman diulang: data lama **tidak dihapus**. Fungsi ini menambahkan sheet `AppConfig`, membuat kunci token, dan memasang ulang trigger tagihan.
6. **Deploy tanpa mengganti URL:** **Deploy → Manage deployments → ✏️ (ikon pensil) → Version: _New version_ → Deploy.**
   > ⚠️ Jangan pilih "New deployment" — itu membuat URL baru. Pakai Edit deployment agar URL `/exec` tetap sama.
7. Pastikan pengaturan deployment: **Execute as: Me** dan **Who has access: Anyone**. (Frontend di GitHub Pages memanggil API tanpa login Google; keamanannya dijaga oleh login aplikasi dan token.)
8. Salin **Web app URL** (berakhiran `/exec`). Anda butuh ini di Bagian B.

> Setelah langkah ini, semua pengguna harus **login ulang** satu kali (sesi lama tidak berlaku lagi). Password lama tetap berlaku. URL `/exec` tidak lagi menampilkan aplikasi — ia hanya menjawab data JSON. Jika sebelumnya aplikasi di-embed di Blogger, embed itu tidak diperlukan lagi.

### Jika ini instalasi BARU

1. Buka https://script.google.com → **New project**. Beri nama `DUO AL NET API`.
2. Ganti nama `Code.gs` menjadi `Kode.gs`, tempel isi `Kode.gs` dari paket ini.
3. Aktifkan manifest (⚙️ Project Settings → Show appsscript.json) dan tempel isi `appsscript.json`.
4. Jalankan **`setupAppEnvironment`** sekali, izinkan otorisasi (klik *Advanced → Go to ... (unsafe)* bila muncul peringatan, itu normal untuk skrip pribadi).
5. **Deploy → New deployment → ⚙️ Web app** → Execute as **Me**, Who has access **Anyone** → Deploy → salin URL `/exec`.

### Cek backend hidup

Tempel URL `/exec` di browser. Harus tampil teks JSON seperti:
`{"success":true,"data":{"app":"DUO AL NET","versi":"2.0",...}}`
Jika yang tampil halaman login Google atau error, periksa lagi "Who has access: Anyone".

Akun awal (hanya untuk instalasi baru) — **segera ganti password**: `admin / admin123`, `owner / owner123`, `kasir / kasir123`.

---

## BAGIAN B — Isi alamat API di frontend

1. Ekstrak `duo-al-net-frontend.zip` ke folder, misalnya `C:\Users\NAMA\Documents\duo-al-net-frontend`.
2. Buka file `js\config.js` dengan Notepad. Ganti tulisannya:

```js
const GAS_URL = 'https://script.google.com/macros/s/AKfycb....../exec';
```
Simpan. **Lakukan ini sebelum push pertama.**

---

## BAGIAN C — Pasang frontend di GitHub Pages (lewat terminal)

> **Jangan** memakai "Add file → Upload files" di web GitHub: struktur folder `css/` dan `js/` akan rusak dan situs jadi 404. Selalu lewat terminal.

### C1. Install Git
- **Windows:** unduh dari https://git-scm.com/download/win, install dengan pengaturan bawaan.
- **Mac:** buka Terminal, ketik `git --version` (ikuti tawaran instalasi bila muncul).

Cek: `git --version` → harus muncul nomor versi.

### C2. Akun GitHub
Daftar di https://github.com. Username Anda akan menjadi bagian alamat situs: `USERNAME.github.io`.

### C3. Identitas Git (sekali saja)
```bash
git config --global user.name "Nama Anda"
git config --global user.email "email-akun-github@contoh.com"
```
(`user.name` bebas, hanya label di riwayat perubahan.)

### C4. Buat repository
Di github.com: tombol **+** → **New repository** → nama: `duo-al-net` → pilih **Public** → **JANGAN** centang README/.gitignore/license → **Create repository**. Biarkan halamannya terbuka.

> Public aman: file frontend tidak memuat password. `GAS_URL` memang alamat publik; yang dilindungi adalah login aplikasi.

### C5. Masuk ke folder yang BENAR
Folder kerja Anda adalah **folder hasil ekstraksi ZIP itu sendiri**, yaitu `C:\Users\NAMA\Documents\duo-al-net-frontend` — **bukan** folder di atasnya, **bukan** folder di dalamnya.

Cara cepat Windows: buka folder itu di File Explorer → klik address bar → ketik `powershell` → Enter.

Lalu cek isinya:
```powershell
dir
```
**Gerbang wajib — sebelum lanjut, pastikan terlihat:**
- ✅ `index.html` langsung di daftar teratas
- ✅ folder `css` dan folder `js`
- ❌ **Tidak ada** file `.gs` (itu backend, bukan untuk GitHub)

Jika yang terlihat justru satu folder lain (mis. `duo-al-net-frontend` lagi), masuk dulu: `cd duo-al-net-frontend`, lalu `dir` ulang. **Jangan lanjut sebelum `index.html` terlihat.**

### C6. Push pertama (satu per satu)
```bash
git init
git add .
git commit -m "Upload pertama"
git branch -M main
git remote add origin https://github.com/USERNAME/duo-al-net.git
git push -u origin main
```
(Ganti `USERNAME`. Ada **titik** di akhir `git add .`.)

Saat diminta **Password**, tempel **Personal Access Token** (bukan password GitHub). Layar **terlihat kosong saat paste — itu normal**. Di PowerShell: klik kanan untuk paste, lalu Enter.

Berhasil jika muncul `Writing objects: 100%` dan `[new branch] main -> main`.

### C7. Buat Personal Access Token (bila diminta)
1. Buka https://github.com/settings/tokens → **Generate new token (classic)**.
2. Note: `git-push-token`, Expiration: 90 days, centang **repo**.
3. **Generate token**, salin `ghp_...` (hanya tampil sekali, simpan di Notepad).
4. Jalankan `git push -u origin main` lagi, tempel token sebagai password.

Alternatif tanpa token manual: **GitHub Desktop** (https://desktop.github.com) → Add Local Repository → pilih folder → Publish repository.

### C8. Aktifkan Pages
Repo di github.com → **Settings → Pages**:

| Kolom | Nilai |
|---|---|
| Source | Deploy from a branch |
| Branch | `main` · `/ (root)` |
| Enforce HTTPS | ✅ dicentang |

**Save**, tunggu 1–2 menit, refresh. Situs: `https://USERNAME.github.io/duo-al-net/`

Cek di halaman repo: `index.html` ada di root, ada folder `css/` dan `js/`, tidak ada file `.gs`.

### C9. Cara update (redeploy)
Setiap ada perubahan file frontend, dari folder yang sama:
```bash
git add .
git commit -m "Deskripsi perubahan"
git push
```
Tunggu 1–2 menit. Jika masih versi lama: **Ctrl+Shift+R**.

Untuk mengubah **backend**: edit di Apps Script lalu **Deploy → Manage deployments → ✏️ → New version → Deploy** (URL tetap).

---

## BAGIAN D — Uji setelah online

1. Buka situs → login `admin`. Dashboard harus muncul dalam beberapa detik (pertama kali bisa 2–4 detik karena server Google "bangun"; berikutnya jauh lebih cepat).
2. Buka menu Bayar → cari pelanggan → simpan pembayaran → cek sheet `Pembayaran` di Spreadsheet.
3. Login sebagai `kasir`: menu Pelanggan/Dashboard tidak boleh muncul.
4. Setelan → hubungkan printer Bluetooth → Uji Cetak (butuh Chrome/Edge; tidak didukung iPhone → pakai RawBT atau Cetak Browser).
5. Cek kecepatan: tekan F12 → Console → ketik `Perf.table()`. Kolom `server` = waktu kerja Apps Script, `net` = jaringan + cold start.

## Jika ada masalah

| Gejala | Penyebab & solusi |
|---|---|
| Login: "GAS_URL belum diisi" | Isi `js/config.js`, lalu `git add .` → `commit` → `push` |
| "Server membalas HTTP ... (bukan JSON)" | Deployment belum **Anyone**, atau URL salah/bukan `/exec`, atau lupa **New version** setelah ubah kode |
| Situs 404 | `index.html` tidak di root repo. Ulangi C5: `git init` harus dari folder yang berisi `index.html` |
| Tampilan polos tanpa gaya | Folder `css/`/`js/` tidak ikut ter-push (upload via web GitHub). Push ulang lewat terminal |
| Tampil versi lama | Ctrl+Shift+R atau jendela Incognito |
| "Sesi berakhir" terus | Admin menonaktifkan akun/mengganti peran/reset password → login ulang |
| Printer BT tidak muncul | Gunakan Chrome/Edge, Bluetooth aktif, situs HTTPS. iPhone: pakai RawBT/Browser |
| Lambat setelah lama tidak dipakai | Cold start Google (1–4 dtk). Aplikasi sudah mengirim ping pemanasan saat dibuka |

## Pasang ikon di layar utama HP (opsional)
Buka situs di Chrome Android → menu ⋮ → **Tambahkan ke layar utama** (atau **Instal aplikasi**). Aplikasi akan terbuka layar penuh dengan ikon logo DUO AL NET.

## Mengganti logo
Ganti file di folder `img/` dengan nama yang sama: `logo.webp` (logo penuh, latar transparan, lebar ±640 px), `mark.webp` (lambang saja), `icon-192.png`, `icon-512.png`, `apple-touch-icon.png`, `favicon-32.png`, `favicon-64.png`. Lalu `git add .` → `commit` → `push`.

## BAGIAN E — Super Admin & Kontrol Sistem

**Super Admin** = semua kemampuan Admin + menu **Kontrol Sistem** dengan 3 saklar:

| Saklar | Saat dimatikan |
|---|---|
| Tagihan otomatis | Jadwal harian jam 01.00 tidak menerbitkan tagihan, dan pelanggan baru setelah tanggal 20 tidak memicu tagihan. Jadwal tetap terpasang, hanya tidak melakukan apa pun. |
| Tombol "Terbitkan tagihan" | Tombolnya hilang dari Pengaturan untuk Admin, dan server menolak permintaannya. |
| Pembayaran massal | Tab "Massal per ODP" dan tombol "Tagih massal" hilang, dan server menolak pembayaran lebih dari satu tagihan sekaligus. Pembayaran satuan tetap jalan. |

Super Admin sendiri **tetap bisa memakai** ketiga fitur itu (di layarnya diberi keterangan "disembunyikan"). Perubahan tersimpan di sheet `Pengaturan` dan berlaku langsung untuk semua pengguna. Saat tagihan otomatis dihentikan, Admin melihat peringatan kuning di Dashboard.

### E1. Perbarui backend
1. Buka proyek Apps Script → `Kode.gs` → hapus semua isi → tempel `Kode.gs` yang baru → 💾 Simpan.
2. **Deploy → Manage deployments → ✏️ → Version: New version → Deploy** (URL tetap).

### E2. Buat akun Super Admin
Admin **tidak bisa** membuat Super Admin lewat aplikasi (agar tidak bisa menaikkan hak sendiri). Akun pertama dibuat dari editor:
1. Di `Kode.gs` cari fungsi `buatSuperAdmin`. Ubah `NAMA` dan `PASSWORD` (minimal 8 karakter, gunakan yang kuat).
2. Pilih fungsi `buatSuperAdmin` di bagian atas → **▶ Run**. Jika `NAMA` sudah ada, akun itu dinaikkan menjadi Super Admin dan passwordnya diganti.
3. **Setelah selesai, kembalikan `PASSWORD` ke tulisan semula** (`GANTI-DENGAN-PASSWORD-KUAT`) lalu Simpan, supaya password tidak tersimpan di kode.
4. Login di aplikasi dengan akun itu → **Lainnya → Kontrol Sistem**.

Aturan keamanan akun:
- Hanya Super Admin yang bisa membuat, mengubah, atau mereset password akun Super Admin.
- Super Admin terakhir tidak bisa diturunkan atau dinonaktifkan (agar saklar tidak terkunci).
- Mengubah peran atau menonaktifkan akun langsung mengeluarkan sesinya.

### E3. Perbarui frontend
Salin isi ZIP terbaru ke folder frontend (**jangan menimpa `js\config.js`**), lalu:
```bash
git add .
git commit -m "Tambah Super Admin dan Kontrol Sistem"
git push
```

Catatan: saklar pembayaran massal hanya untuk **pembayaran**. Fitur Cetak Ulang massal tidak ikut disembunyikan.

## Catatan keamanan
- Ganti semua password bawaan lewat Lainnya → Ubah Password Saya.
- Hak akses dicek di **server** pada setiap permintaan; menyembunyikan menu hanya kosmetik.
- Token login berlaku 14 hari, bertanda tangan HMAC, dan langsung dicabut bila akun dinonaktifkan / peran diubah / password direset.
- Jangan membagikan URL `/exec` + akun ke orang yang tidak berwenang.
