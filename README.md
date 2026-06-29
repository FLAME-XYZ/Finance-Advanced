<div align="center">

<img src="logoku.png" alt="Flame Finance Advanced Logo" width="220"/>

# 🔥 FLAME FINANCE ADVANCED

**Platform analisis multi-aset — Crypto, Saham (US/IDX), Forex, Emas/Perak/Minyak**

![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat-square&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat-square&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black)
![Firebase](https://img.shields.io/badge/Firebase-FFCA28?style=flat-square&logo=firebase&logoColor=black)
![Status](https://img.shields.io/badge/status-active-success?style=flat-square)
![License](https://img.shields.io/badge/license-proprietary-red?style=flat-square)

</div>

---

## 📖 Tentang Project

**Flame Finance Advanced** adalah web app analisis pasar finansial yang berjalan **100% di sisi client** (static site, tanpa backend custom). Semua data realtime diambil langsung dari API premium, sementara autentikasi, lisensi premium, dan limit penggunaan dikelola lewat **Firebase**.

## 🗂️ Bahasa & Teknologi

Project ini dibangun pure web stack, langsung jalan di browser:

| Layer | Teknologi |
|---|---|
| Markup | **HTML5** |
| Styling | **CSS3** (custom, tanpa framework CSS) |
| Logic | **JavaScript (Vanilla ES6+)**, modular via `window.*` namespace |
| Auth & Database | **Firebase Authentication** + **Firebase Realtime Database** (SDK v10, compat mode) |
| Rendering Chart | **Canvas2D API** (native, untuk export chart beranotasi) |

> Tidak ada Node.js/npm build step — cukup buka `index.html` lewat web server statis apapun.

---

## 🔌 Resource API

Semua data pasar diambil langsung dari browser ke API (client-side fetch):

| Sumber Data | Untuk | Auth | Catatan |
|---|---|---|---|
| **Binance Global API** (`api.binance.com`) | Crypto OHLCV, ticker, order book + harga Emas/Perak/Minyak sintetik (XAUTUSDT dkk.) | Tidak perlu API key | Gratis, CORS terbuka |
| **Twelve Data API** | Saham US & IDX, Forex (time series & quote real) | **Perlu API key** | Free tier: ±800 request/hari, 8 request/menit. Hasil di-cache 45 detik di client. |
| **Yahoo Finance (endpoint internal)** | Fallback data saham Indonesia (IDX) | Tidak resmi | Bukan API publik resmi Yahoo — bisa berubah/berhenti tanpa pemberitahuan, dipakai sebagai fallback gratis. |
| **ExchangeRate-API** (`open.er-api.com`) | Kurs forex | Tidak perlu API key | Update ±1 jam sekali |
| **Alternative.me** | Crypto Fear & Greed Index | Tidak perlu API key | — |
| **CoinGecko API** | Data market crypto tambahan (market cap, global stats) | Tidak perlu API key | Ada rate limit |
| **RSS2JSON** (`api.rss2json.com`) | Konversi RSS feed berita finansial ke JSON | Tidak perlu API key | Dipakai oleh halaman News |

---

## 🗄️ Database

Menggunakan **Firebase Realtime Database** (JSON tree) + **Firebase Authentication** (Email/Password).

<div align="center">

Made with 🔥 — **Flame Finance Advanced**

</div>
