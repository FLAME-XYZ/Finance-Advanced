# Flame Finance Advanced (ProTrader Analytics)

Website analisis trading multi-aset (crypto, saham, forex, metals/komoditas) berbasis HTML/CSS/JS statis, dengan sistem akun & lisensi premium memakai Firebase (Auth + Realtime Database). Tidak ada backend server kustom — seluruh logika jalan di browser (client-side), dan keamanan akses data dijaga lewat **Firebase Realtime Database Rules**.

> Bahasa UI & komentar kode: Bahasa Indonesia.

---

## Daftar Isi

- [Ringkasan Fitur](#ringkasan-fitur)
- [Struktur Folder](#struktur-folder)
- [Arsitektur & Alur Data](#arsitektur--alur-data)
- [Sistem Akun, Role & Lisensi Premium](#sistem-akun-role--lisensi-premium)
- [Sumber Data / API Eksternal](#sumber-data--api-eksternal)
- [Library & Resource Eksternal](#library--resource-eksternal)
- [Setup & Menjalankan Project](#setup--menjalankan-project)
- [Konfigurasi Firebase](#konfigurasi-firebase)
- [Urutan Load Script (PENTING)](#urutan-load-script-penting)
- [Alur Bisnis: Beli & Aktivasi Premium](#alur-bisnis-beli--aktivasi-premium)
- [Admin Panel](#admin-panel)
- [Keamanan](#keamanan)
- [Keterbatasan & Catatan Teknis](#keterbatasan--catatan-teknis)
- [Hal yang Perlu Ditindaklanjuti](#hal-yang-perlu-ditindaklanjuti)

---

## Ringkasan Fitur

**Untuk semua user (gratis & premium):**
- Dashboard market overview, watchlist, heatmap, Fear & Greed Index
- Chart candlestick real-time (TradingView Widget) + indikator teknikal
- Analisis teknikal lanjutan: SMC, ICT, Fibonacci, Elliott Wave, Wyckoff, Harmonic Pattern, Price Action, Supply & Demand
- Confluence scoring & sinyal trading otomatis, analisis multi-timeframe (MTF)
- Screener multi-instrumen (crypto, saham, forex, metals)
- News feed (RSS FXStreet)
- Kalender ekonomi (link ke Trading Economics)
- Hub Developer & Komunitas (`devcom.html`) — grup WhatsApp + link dokumentasi API

**Khusus Premium:**
- **Screen Analysis** — universal instrument scanner (lebih dalam dari Screener biasa)
- **Market Replay** — simulator trading dengan replay candle historis
- Tanpa limit harian (lihat [Limit Pemakaian](#sistem-akun-role--lisensi-premium))
- Export chart hasil analisis (candlestick + marking SMC/ICT/Fibonacci/Entry-SL-TP) jadi 1 file PNG

---

## Struktur Folder

```
PREMIUM-F2/
├── index.html                # SPA shell + router halaman utama (#main-content)
├── login.html                # Halaman login
├── register.html             # Halaman registrasi
├── premium.html              # Halaman upgrade premium (QRIS + redeem kode)
├── adminpanel.html           # Panel admin (generate/hapus lisensi, kelola user)
├── database.rules.json       # Firebase Realtime Database security rules
├── logoku.png                # Logo aplikasi
├── qris.png                  # Gambar QRIS untuk pembayaran manual
│
├── css/
│   ├── style.css             # Styling global aplikasi
│   └── auth-style.css        # Styling khusus halaman auth (login/register/premium/admin)
│
├── js/
│   ├── firebase-init.js      # Init Firebase SDK + konstanta aplikasi (window.APP_CONST)
│   ├── device-fingerprint.js # Generate device ID persisten (info/audit, BUKAN gate akses)
│   ├── auth.js               # Register, login, logout, status premium efektif
│   ├── license.js            # Generate/hapus/redeem kode lisensi
│   ├── limits.js             # Pembatasan pemakaian harian fitur untuk user free
│   ├── admin.js              # List & hapus user (dipakai adminpanel.html)
│   ├── config.js             # Konstanta: endpoint API, daftar instrumen, timeframe, dll
│   ├── api.js                # Fetch data harga/candle dari semua sumber eksternal
│   ├── twelvedata.js         # Integrasi Twelve Data API (saham & forex real)
│   ├── yahoo-idx.js          # Sumber data gratis khusus saham IDX (fallback Twelve Data)
│   ├── gold-offset.js        # Koreksi manual selisih harga XAU (TradingView vs Binance)
│   ├── indicators.js         # Library kalkulasi indikator teknikal murni JS
│   ├── analysis.js           # SMC, ICT, Fibonacci, Elliott, Wyckoff, Harmonic, dll
│   ├── signals.js            # Confluence scoring, generate trade setup, MTF analysis
│   ├── chart.js              # Stub kompatibilitas (chart asli pakai TradingView Widget)
│   ├── chart-export.js       # Export chart + anotasi analisis ke PNG (fitur premium)
│   └── ui.js                 # Utilitas UI global: toast, dark mode, format, dll
│
└── pages/                    # Halaman yang di-load via SPA router ke index.html
    ├── dashboard.html         # Market overview, watchlist, heatmap (UNLIMITED_PAGES)
    ├── chart.html              # Chart & indikator (LIMITED_FEATURES)
    ├── analysis.html            # Analisis advanced (LIMITED_FEATURES)
    ├── screener.html            # Multi-instrument screener (LIMITED_FEATURES)
    ├── screenanalysis.html      # PREMIUM — Screen Analysis scanner
    ├── replay.html              # PREMIUM — Market Replay & Trading Simulator
    ├── news.html                # News feed (UNLIMITED_PAGES)
    └── devcom.html              # Developer & Komunitas hub
```

---

## Arsitektur & Alur Data

```
┌──────────────────────────────────────────────────────────────────┐
│                          BROWSER (Client)                        │
│                                                                    │
│  index.html (SPA shell)                                          │
│   └─ loadPage(name) → fetch pages/{name}.html → inject ke        │
│      #main-content, lalu jalankan init function halaman tsb       │
│                                                                    │
│  Modul JS global (window.*):                                     │
│   Auth · License · Limits · AdminUsers · DeviceFingerprint        │
│   CONFIG · Indicators · Analysis · Signals · UI                   │
└───────────────┬───────────────────────────┬───────────────────────┘
                │                           │
                ▼                           ▼
   ┌─────────────────────────┐   ┌───────────────────────────────┐
   │  Firebase (BaaS)        │   │  API Pihak Ketiga (publik)    │
   │  • Authentication       │   │  • Binance Global              │
   │  • Realtime Database    │   │  • Twelve Data                 │
   │    (users/, licenses/)  │   │  • ExchangeRate-API             │
   │  • Rules = satu-satunya │   │  • CoinGecko                   │
   │    lapisan keamanan     │   │  • Yahoo Finance (unofficial)  │
   │    server-side          │   │  • rss2json (FXStreet)         │
   └─────────────────────────┘   │  • alternative.me (F&G Index)  │
                                  │  • TradingView Widget (chart)  │
                                  └───────────────────────────────┘
```

Tidak ada server aplikasi (Node/PHP/dll). Semua "backend" adalah Firebase Realtime Database, dan semua "API key" yang terlihat di kode (`firebase-init.js`, `twelvedata.js`) **memang terlihat oleh siapa pun** yang membuka DevTools — ini bawaan dari arsitektur situs statis tanpa server, bukan kebocoran yang bisa "diperbaiki" tanpa menambah backend. Yang menjaga data tetap aman adalah `database.rules.json`.

---

## Sistem Akun, Role & Lisensi Premium

### Role

Disimpan di `users/{uid}/role`, nilai: `'free'` | `'premium'` | `'admin'`.

- **admin** — hardcode lewat `APP_CONST.ADMIN_EMAIL` (`admin@flamefinance.id`). Saat akun dengan email ini register/login, role otomatis di-set `'admin'`. Admin selalu dianggap unlimited, tidak kena gate apa pun.
- **premium** — didapat dengan redeem kode lisensi yang valid. Premium aktif selama `premiumUntil` belum lewat. **Tidak ada device-lock** — premium yang sah aktif di device/browser manapun selama akun yang sama login (lihat catatan riwayat perbaikan di bawah).
- **free** — default. Kena limit harian untuk fitur tertentu.

### Limit Pemakaian (Free Tier)

Diatur di `js/limits.js` + `APP_CONST` (`firebase-init.js`):

| Kategori | Halaman | Limit Free |
|---|---|---|
| `UNLIMITED_PAGES` | dashboard, news | Tidak pernah dicek, selalu bebas |
| `LIMITED_FEATURES` | chart, analysis, calculator, screener | 2x per fitur **per hari** (reset tiap pergantian hari zona WIB) |
| `PREMIUM_EXCLUSIVE_FEATURES` | compare *(lihat catatan)*, screenanalysis, replay | **0x** — user free tidak bisa akses sama sekali |

Counter pemakaian harian disimpan di `users/{uid}/usage/{YYYY-MM-DD}/{feature}` dan ditambah lewat Firebase `transaction()` supaya aman dari race condition (misal user buka 2 tab bersamaan).

### Lisensi

Skema di `licenses/{CODE}`:

```js
{
  code, duration,        // 7 | 30 | 365 (hari)
  status,                 // 'unused' | 'used'
  createdAt, createdBy,
  usedBy, usedByEmail, usedAt, deviceId
}
```

- **Generate** (admin only) — format kode `FLAME-XXXX-XXXX` (huruf besar A–Z tanpa O/I + angka 2–9 tanpa 0/1, supaya gampang dibacakan via WhatsApp dan tidak ambigu).
- **Redeem** (user) — pakai Firebase `transaction()` di node `licenses/{CODE}` supaya atomic; kalau 2 device coba redeem kode yang sama bersamaan, hanya satu yang berhasil. Setelah berhasil, `users/{uid}` diupdate: `role: 'premium'`, `premiumUntil: now + duration*hari`, `licenseUsed: CODE`.
- **Hapus** (admin only) — kalau kode yang dihapus statusnya `'used'`, user yang memakainya otomatis di-revoke balik ke `'free'` (asalkan `licenseUsed` user tsb masih menunjuk ke kode itu — supaya tidak salah revoke kalau user sudah redeem kode lain setelahnya).
- **"1 kode 1x pakai, lintas akun"** dijamin di dua lapis: transaksi atomic di client **dan** `database.rules.json` di server (lihat bagian [Keamanan](#keamanan)) — jadi tidak bisa dibobol lewat console browser sekalipun.

### Riwayat Perbaikan Bug Penting

Dua bug berikut pernah dilaporkan dan sudah diperbaiki di kode saat ini:

1. **Premium berubah jadi free saat login di device/browser lain** — sebelumnya `auth.js` mengunci status premium ke 1 `deviceId` (disimpan di `localStorage`). Sekarang device-lock sudah dihapus total dari `getEffectivePremiumStatus()`; `deviceId`/`deviceLabel` hanya tersimpan sebagai info riwayat di adminpanel, tidak lagi memblokir akses.
2. **Adminpanel masih menampilkan "Premium" setelah lisensi dihapus** — `license.js` sudah benar merevoke `role` user jadi `'free'` di database, tapi tabel User di adminpanel tidak ikut refresh otomatis. Sudah diperbaiki: tombol "Hapus kode" sekarang memanggil `loadUsers()` juga, bukan cuma `loadLicenses()`.
3. **Celah self-extend premium** — sebelumnya field `premiumUntil` tidak punya validasi server, sehingga user premium yang sah secara teknis bisa menulis `premiumUntil` ke nilai bebas (extend sendiri tanpa kode baru) lewat console browser. Sudah ditutup lewat `.validate` baru di `database.rules.json` yang membatasi `premiumUntil` sesuai `duration` kode lisensi yang sah.

---

## Sumber Data / API Eksternal

Semua endpoint berikut **gratis dan tanpa API key**, kecuali Twelve Data (perlu key, free tier terbatas):

| Sumber | Dipakai untuk | File | Catatan |
|---|---|---|---|
| [Binance Global](https://binance-docs.github.io/apidocs/spot/en/) | Harga & candle crypto, metals (XAUTUSDT/XAGUSDT), minyak (CLUSDT) | `api.js`, `config.js` | CORS OK, tanpa key |
| [Twelve Data](https://twelvedata.com/docs) | Saham (US & IDX) & forex — data real | `twelvedata.js` | **Perlu API key**, key terlihat di client (lihat [Keterbatasan](#keterbatasan--catatan-teknis)) |
| Yahoo Finance (endpoint internal, tidak resmi) | Fallback gratis khusus saham IDX | `yahoo-idx.js` | ⚠️ Tidak resmi/tidak didukung Yahoo — bisa berhenti berfungsi sewaktu-waktu tanpa pemberitahuan |
| [ExchangeRate-API](https://www.exchangerate-api.com/) (open.er-api.com) | Kurs forex | `api.js`, `config.js` | Gratis, update ±1 jam |
| [CoinGecko](https://www.coingecko.com/en/api) | Data market global crypto | `api.js`, `config.js` | Rate-limited di free tier |
| [Alternative.me](https://alternative.me/crypto/fear-and-greed-index/) | Fear & Greed Index | `api.js` | Gratis |
| [rss2json](https://rss2json.com/) | Konversi RSS FXStreet → JSON untuk halaman News | `api.js` | Gratis dengan rate limit |
| [TradingView Widget](https://www.tradingview.com/widget/) (`s3.tradingview.com/tv.js`) | Rendering chart candlestick | `chart.html` | Widget eksternal — chart **tidak bisa** di-screenshot via canvas (cross-origin iframe), makanya export PNG (`chart-export.js`) di-render ulang manual dari data OHLCV, bukan screenshot widget |

---

## Library & Resource Eksternal

| Library | Sumber | Fungsi |
|---|---|---|
| Firebase SDK (compat) v10.13.1 | `gstatic.com/firebasejs` | Auth + Realtime Database |
| Lightweight Charts | `unpkg.com/lightweight-charts` | Charting library (cadangan/komponen tertentu — chart utama pakai TradingView Widget) |
| Google Fonts — Inter & JetBrains Mono | `fonts.googleapis.com` | Tipografi |

Link dokumentasi dev/komunitas yang dicantumkan di `pages/devcom.html`:
- Pine Script Docs (TradingView)
- Financial Modeling Prep API Docs
- Binance API Docs
- Twelve Data API Docs
- Firebase Docs
- Trading Economics — Kalender Ekonomi
- Grup komunitas WhatsApp (link invite ada di `devcom.html`)

---

## Setup & Menjalankan Project

Ini situs statis murni — tidak perlu build step atau Node.js untuk menjalankannya.

1. **Clone/extract** folder `PREMIUM-F2/` ke web server statis mana pun (Firebase Hosting, Netlify, Vercel, Apache/Nginx, atau cukup `python3 -m http.server` untuk testing lokal).

   > ⚠️ Jangan dibuka langsung lewat `file://` — beberapa fitur (terutama Firebase Auth persistence dan fetch ke API eksternal) butuh konteks `http://` atau `https://`.

2. Pastikan project Firebase sudah dibuat (Authentication dengan **Email/Password** aktif, dan Realtime Database dibuat).
3. Update `firebaseConfig` di `js/firebase-init.js` sesuai project Firebase Anda (lihat bagian berikut).
4. **Deploy `database.rules.json` ke Firebase Console** (Realtime Database → tab Rules → paste → Publish). Ini **wajib** — rules di file lokal tidak otomatis aktif tanpa di-publish manual.
5. Kalau pakai Twelve Data, isi API key di `js/twelvedata.js`.
6. Ganti `qris.png` dengan QRIS pembayaran Anda sendiri, dan `WHATSAPP_NUMBER` di `firebase-init.js` dengan nomor WA admin.

---

## Konfigurasi Firebase

Konfigurasi ada di `js/firebase-init.js`:

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  databaseURL: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "...",
  measurementId: "..."
};
```

> `apiKey` Firebase untuk web app **memang publik secara desain** — Google sendiri menyebut ini bukan rahasia (bukan seperti API key server biasa). Keamanan data dijaga oleh **Realtime Database Rules**, bukan dengan menyembunyikan config ini.

Konstanta aplikasi lain (`window.APP_CONST`) juga ada di file ini:

```js
window.APP_CONST = {
  ADMIN_EMAIL: 'admin@flamefinance.id',
  WHATSAPP_NUMBER: '62XXXXXXXXXXX',
  LICENSE_PLANS: [ /* 7 hari, 30 hari, 1 tahun + harga */ ],
  LIMITED_FEATURES: ['chart', 'analysis', 'calculator', 'screener'],
  FREE_DAILY_LIMIT: 2,
  PREMIUM_EXCLUSIVE_FEATURES: ['compare', 'screenanalysis', 'replay'],
  UNLIMITED_PAGES: ['dashboard', 'news'],
  FEATURE_LABELS: { /* label tampilan tiap fitur */ },
};
```

---

## Urutan Load Script (PENTING)

Urutan `<script>` di `index.html` **tidak boleh diubah sembarangan** karena tiap modul saling depend ke modul sebelumnya:

```html
<!-- 1. SDK & Init Firebase -->
<script src=".../firebase-app-compat.js"></script>
<script src=".../firebase-auth-compat.js"></script>
<script src=".../firebase-database-compat.js"></script>
<script src="js/firebase-init.js"></script>

<!-- 2. Auth & Lisensi (depend ke firebase-init.js) -->
<script src="js/device-fingerprint.js"></script>
<script src="js/auth.js"></script>
<script src="js/license.js"></script>
<script src="js/limits.js"></script>

<!-- 3. Data & Analisis (depend ke config.js, satu sama lain berurutan) -->
<script src="js/config.js"></script>        <!-- Konstanta & AppState -->
<script src="js/gold-offset.js"></script>   <!-- Koreksi harga XAU -->
<script src="js/twelvedata.js"></script>    <!-- Data saham/forex real -->
<script src="js/yahoo-idx.js"></script>     <!-- Fallback saham IDX -->
<script src="js/api.js"></script>           <!-- Fetch API & AppInit -->
<script src="js/indicators.js"></script>    <!-- Kalkulasi indikator -->
<script src="js/signals.js"></script>       <!-- Confluence & signals -->
<script src="js/analysis.js"></script>      <!-- SMC, ICT, Fib, dll -->
<script src="js/chart-export.js"></script>  <!-- Export chart premium -->
<script src="js/chart.js"></script>         <!-- Setup chart TradingView -->
<script src="js/ui.js"></script>            <!-- Utilitas UI — HARUS TERAKHIR -->
```

Halaman lain yang juga memuat modul auth (`login.html`, `register.html`, `premium.html`, `adminpanel.html`) hanya butuh urutan grup 1 & 2 di atas (tidak perlu seluruh modul data/analisis).

---

## Alur Bisnis: Beli & Aktivasi Premium

```
User                          Admin                         Sistem
 │                              │                              │
 │ 1. Buka premium.html         │                              │
 │ 2. Scan QRIS, transfer       │                              │
 │ 3. Konfirmasi via WhatsApp ─▶│                              │
 │                              │ 4. Generate kode lisensi     │
 │                              │    di adminpanel.html        │
 │                              │    (License.generate)        │
 │                              │ 5. Kirim kode FLAME-XXXX-XXXX│
 │◀─────────────────────────────┤    via WhatsApp              │
 │ 6. Masukkan kode di           │                              │
 │    premium.html (redeem)  ───┼─────────────────────────────▶│
 │                              │                              │ 7. Transaction:
 │                              │                              │    licenses/{code}
 │                              │                              │    status→used
 │                              │                              │ 8. users/{uid}:
 │                              │                              │    role→premium
 │ 9. Akses premium aktif       │                              │    premiumUntil set
 │    (di device manapun)       │                              │
```

Pembayaran **manual** (tidak ada payment gateway otomatis) — admin generate kode setelah verifikasi transfer secara manual via WhatsApp.

---

## Admin Panel

`adminpanel.html` — hanya bisa diakses akun dengan `role === 'admin'`.

**Tab Kode Lisensi:**
- Generate kode baru (7 hari / 30 hari / 1 tahun) — tidak ada batas jumlah kode
- Lihat semua kode (status, siapa yang pakai, kapan dipakai)
- Hapus kode (otomatis revoke user terkait jadi `free` jika kode sudah terpakai)

**Tab Manajemen User:**
- Lihat semua user (email, username, role, premium hingga kapan, device terakhir, tanggal daftar)
- Hapus user — menghapus profil di Realtime Database (`users/{uid}`). **Tidak menghapus akun Firebase Auth-nya** (butuh Admin SDK di server, di luar cakupan situs statis ini) — tapi karena profil sudah tidak ada, user akan otomatis ke-logout paksa & tidak bisa login lagi (lihat `auth.js: loginUser()`).

---

## Keamanan

Karena tidak ada server aplikasi, **`database.rules.json` adalah satu-satunya lapisan pertahanan nyata** — semua validasi di kode JS (`license.js`, `auth.js`, dll) hanya kenyamanan UI, **bisa dilewati** siapa pun yang membuka console browser dan memanggil Firebase SDK langsung. Rules saat ini menjamin:

- User biasa hanya bisa membaca/menulis profilnya sendiri; admin bisa baca/tulis semua.
- `role` tidak bisa di-set sembarangan ke `'premium'` atau `'admin'` — harus melalui kode lisensi yang valid (tervalidasi balik ke `licenses/{code}/usedBy`) atau email admin yang dikenali.
- `premiumUntil` tidak bisa di-set ke nilai bebas — dibatasi sesuai `duration` kode lisensi yang sah yang sedang dipegang user (+ toleransi 5 menit untuk selisih jam client/server).
- Kode lisensi yang statusnya sudah `'used'
