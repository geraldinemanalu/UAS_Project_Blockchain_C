# 🎓 CertiChain

**Platform penerbitan & verifikasi sertifikat digital berbasis smart contract Ethereum.**
Transparan, immutable, dan dapat diverifikasi siapa saja — tanpa perantara.

> Dibuat oleh **Kelompok 1 — Blockchain C, Institut Teknologi PLN** (

---

## 📋 Deskripsi

CertiChain adalah aplikasi terdesentralisasi (dApp) untuk menerbitkan, mencari, memverifikasi, dan mencabut sertifikat pelatihan/kursus. Seluruh data sertifikat dicatat langsung di blockchain Ethereum (Sepolia Testnet) melalui sebuah smart contract, sehingga:

- **Tidak bisa dipalsukan** — data tersimpan on-chain dan diikat dengan hash unik (keccak256).
- **Transparan** — siapa pun dapat memverifikasi keaslian sertifikat tanpa menghubungi pihak penerbit.
- **Tanpa server/database terpusat** — front-end murni HTML/CSS/JavaScript yang langsung berbicara ke smart contract via MetaMask + Ethers.js.

---

## ✨ Fitur

| Fitur | Deskripsi |
|---|---|
| 🪪 Terbitkan Sertifikat | Admin/issuer menerbitkan sertifikat baru (nama peserta + nama kursus) langsung ke blockchain |
| 🔍 Lihat Sertifikat | Cari sertifikat berdasarkan **ID** maupun **hash** (0x + 64 karakter heksadesimal) |
| ✅ Verifikasi | Cek keaslian & status aktif/tercabut suatu sertifikat secara *trustless* |
| 🚫 Cabut Sertifikat | Admin/issuer dapat mencabut (revoke) sertifikat yang sudah terbit |
| 🗂️ Explorer | Jelajahi seluruh sertifikat yang pernah tercatat on-chain, tanpa perlu login |
| 👥 Multi-Role | Dashboard **Admin** (akses penuh) dan **Pengguna** (khusus lihat & verifikasi, read-only) |
| 🦊 Integrasi MetaMask | Auto-deteksi jaringan, auto-reconnect wallet, notifikasi ganti akun/jaringan |
| 🖼️ Sertifikat Digital | Modal sertifikat visual dengan efek konfeti + unduh sebagai gambar PNG |
| 🌐 CDN Fallback | Pemuatan `ethers.js` dengan fallback otomatis ke beberapa CDN bila salah satu gagal |

---

## 🛠️ Teknologi yang Digunakan

- **Smart Contract:** Solidity `^0.8.0`, di-deploy via Remix IDE ke **Ethereum Sepolia Testnet**
- **Front-end:** HTML5, CSS3, JavaScript murni (vanilla — tanpa framework/bundler)
- **Web3 Library:** [Ethers.js v6](https://docs.ethers.org/v6/) (dimuat via CDN dengan fallback)
- **Wallet:** MetaMask (`window.ethereum` / EIP-1193)
- **Utilitas front-end:** html2canvas (unduh sertifikat sebagai PNG)
- **Autentikasi:** Berbasis `localStorage`/`sessionStorage` (demo, sisi klien saja — lihat [Catatan Keamanan](#-catatan-keamanan))

---

## 📁 Struktur Proyek

```
cc_final/
├── CertificateSystem.sol      # Smart contract utama (Solidity)
├── index.html                 # Dashboard ADMIN (SPA: home, app, explorer, about)
├── user.html                  # Dashboard PENGGUNA (read-only: lihat & verifikasi)
├── login.html                 # Halaman login (role selector: admin/pengguna)
├── register.html              # Halaman registrasi akun baru
├── css/
│   ├── base.css                # Variabel warna, reset, tipografi dasar
│   ├── header.css               # Navbar, nav-pill, wallet chip
│   ├── pages.css                 # Layout semua halaman & komponen
│   ├── certificate.css            # Modal sertifikat digital
│   └── tilt3d.css                  # Efek 3D tilt pada kartu fitur
└── js/
    ├── ethers-loader.js         # Loader ethers.js v6 + fallback multi-CDN
    ├── auth.js                  # Sistem login/register/session (localStorage)
    ├── blockchain.js            # Koneksi MetaMask + semua fungsi smart contract
    ├── certificate.js           # Modal sertifikat, unduh PNG, konfeti
    ├── explorer.js               # Daftar seluruh sertifikat on-chain
    ├── router.js                 # Navigasi SPA (single-page application)
    ├── ui.js                      # Nav indicator, mobile menu, carousel workflow
    └── tilt3d.js                   # Efek hover 3D pada feature card
```

---

## ⚙️ Smart Contract — `CertificateSystem.sol`

### Jaringan & Alamat

| | |
|---|---|
| **Jaringan** | Ethereum Sepolia Testnet (chainId `11155111`) |
| **Alamat Kontrak** | `0x3A47f2eAD6228bEd643bF669060E9a73A8369B43` *(sesuai konfigurasi di `js/blockchain.js` — verifikasi ulang dengan hasil deploy Anda sendiri di Etherscan)* |
| **Lisensi Kontrak** | MIT (SPDX-License-Identifier: MIT) |

### Struktur Data

```solidity
struct Certificate {
    uint256 id;            // ID unik sertifikat
    string  recipientName; // Nama peserta pelatihan
    string  courseName;    // Nama kursus/pelatihan
    uint256 issueDate;     // Tanggal terbit (UNIX timestamp)
    bool    isValid;       // Status: true = aktif, false = dicabut
    address issuedBy;      // Alamat wallet yang menerbitkan
    bytes32 certHash;      // Keccak256 hash unik dari data sertifikat
}
```

### Fungsi Utama

| Fungsi | Akses | Jenis | Keterangan |
|---|---|---|---|
| `createCertificate(name, course)` | `onlyAuthorized` | write | Menerbitkan sertifikat baru, mengembalikan ID |
| `getCertificate(id)` | publik | view | Ambil data sertifikat berdasarkan ID |
| `getCertificateByHash(hash)` | publik | view | Ambil data sertifikat berdasarkan hash |
| `verifyCertificate(id)` | publik | view | Cek keaslian & status by ID |
| `verifyCertificateByHash(hash)` | publik | view | Cek keaslian & status by hash |
| `revokeCertificate(id)` | `onlyAuthorized` | write | Mencabut sertifikat by ID |
| `revokeCertificateByHash(hash)` | `onlyAuthorized` | write | Mencabut sertifikat by hash |
| `authorizeIssuer(address)` | `onlyOwner` | write | Menambahkan issuer baru |
| `revokeIssuer(address)` | `onlyOwner` | write | Mencabut wewenang issuer |
| `getHashNonce()` | publik | view | Melihat nilai nonce anti-tabrakan hash saat ini |
| `owner()` / `certificateCount()` / `authorizedIssuers(addr)` | publik | view | Getter otomatis dari state variable `public` |

### Events

```solidity
event CertificateCreated(uint256 indexed id, string recipientName, string courseName, address issuedBy, bytes32 certHash);
event CertificateRevoked(uint256 indexed id, address revokedBy);
event IssuerAuthorized(address indexed issuer);
event IssuerRevoked(address indexed issuer);
```

Front-end mengekstrak `id` dan `certHash` dari event `CertificateCreated` pada transaction receipt setiap kali sertifikat baru diterbitkan.

---

## 🚀 Cara Menjalankan

### Prasyarat

1. **Browser** dengan ekstensi [MetaMask](https://metamask.io/) terpasang.
2. Wallet MetaMask sudah diarahkan ke jaringan **Sepolia Testnet**.
3. Sejumlah **Sepolia ETH** (gas fee) — bisa didapat gratis dari faucet, misalnya [sepoliafaucet.com](https://sepoliafaucet.com) atau faucet Sepolia lainnya.
4. Koneksi internet aktif (untuk memuat `ethers.js` dari CDN).

### Langkah-langkah

1. **Clone / unduh** folder `cc_final/` ini.
2. Karena aplikasi ini murni HTML/CSS/JS statis, jalankan lewat **local web server** (disarankan, agar tidak ada isu CORS/`file://`), misalnya salah satu dari:
   ```bash
   # Opsi 1 — Python
   cd cc_final
   python3 -m http.server 8080

   # Opsi 2 — Node.js (http-server)
   npx http-server cc_final -p 8080

   # Opsi 3 — VS Code
   # klik kanan index.html → "Open with Live Server"
   ```
3. Buka `http://localhost:8080` di browser.
4. Klik **Connect Wallet**, setujui koneksi di popup MetaMask, pastikan jaringan aktif adalah **Sepolia**.
5. (Opsional) Bila ingin memakai kontrak Anda sendiri: deploy ulang `CertificateSystem.sol` via [Remix IDE](https://remix.ethereum.org/) ke Sepolia, lalu ganti nilai `CONTRACT_ADDRESS` di `js/blockchain.js` dengan alamat kontrak baru Anda.

---

## 🔑 Akun Demo (Login)

Akun berikut otomatis ter-seed ke `localStorage` saat aplikasi pertama kali dibuka (lihat `DEFAULT_ACCOUNTS` di `js/auth.js`):

| Peran | Email | Password |
|---|---|---|
| 🛡️ Admin | `admin@certichain.id` | `Admin@123` |
| 👤 Pengguna | `user@certichain.id` | `User@123` |

> Akun baru juga dapat dibuat lewat halaman **register.html** — namun ingat, ini hanya disimpan di `localStorage` browser (lihat catatan keamanan di bawah).

---

## 👥 Peran Pengguna

| Halaman | Peran | Tab yang Tersedia |
|---|---|---|
| `index.html` | Admin | Terbitkan · Lihat · Verifikasi · Cabut |
| `user.html` | Pengguna biasa | Lihat · Verifikasi *(read-only, tanpa gas fee)* |

Routing dan pembatasan akses ditangani oleh `authGuard()` di `js/auth.js`, yang otomatis mengarahkan pengguna ke dashboard sesuai perannya.

---

## 🧭 Alur Penggunaan Singkat

1. **Login** → pilih peran (Admin/Pengguna) → masuk ke dashboard.
2. **Connect Wallet** → hubungkan MetaMask (wajib jaringan Sepolia).
3. *(Admin)* Buka tab **Terbitkan** → isi nama peserta & nama kursus → konfirmasi transaksi di MetaMask → sertifikat digital muncul lengkap dengan ID & hash.
4. Siapa pun bisa membuka tab **Lihat**/**Verifikasi** dan memasukkan ID atau hash sertifikat untuk mengecek keasliannya — gratis, tanpa gas fee.
5. *(Admin)* Tab **Cabut** digunakan bila sertifikat perlu dinonaktifkan (misalnya diterbitkan keliru).
6. Halaman **Explorer** menampilkan seluruh riwayat sertifikat yang pernah tercatat on-chain.

---

## 🔒 Catatan Keamanan

Proyek ini dibuat untuk tujuan **pembelajaran/tugas kuliah** dan memiliki beberapa keterbatasan yang perlu disadari sebelum dipakai di lingkungan produksi:

- **Autentikasi bersifat demo** — akun & password disimpan plaintext di `localStorage` browser, tanpa backend maupun hashing. Jangan gunakan password asli/sensitif.
- **Sentralisasi kontrol** — `owner` kontrak memegang kendali penuh atas `authorizeIssuer`/`revokeIssuer` tanpa mekanisme multi-signature atau pemulihan kunci.
- **Data on-chain bersifat publik & permanen** — nama peserta dan nama kursus tersimpan selamanya di blockchain dan tidak dapat dihapus.

Silakan sesuaikan/tambahkan mitigasi bila proyek ini dikembangkan lebih lanjut menuju penggunaan nyata.

---

## 📄 Lisensi

Smart contract dirilis dengan lisensi **MIT** (lihat header `CertificateSystem.sol`). Sesuaikan bagian ini bila kode front-end ingin dilisensikan secara terpisah.

---

## 🙏 Kredit

Dikembangkan sebagai proyek tugas mata kuliah **Teknologi Blockchain** — Program Studi Sistem Informasi.
