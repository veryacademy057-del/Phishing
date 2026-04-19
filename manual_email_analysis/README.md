# 📧 Manual Email Analysis — Investigasi Email Phishing

> **Seri Lab:** Blue Team SOC Lab  
> **Platform:** [TryHackMe](https://tryhackme.com)  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/_TiD-1Q3RaM?si=nkzucMZQc4P0L22w)  
> **Kategori:** `Phishing Analysis, SOC Level 1`

---

## 📋 Deskripsi

Lab ini mensimulasikan peran **SOC Analyst Level 1** yang bertugas menganalisis email mencurigakan yang dilaporkan oleh pengguna. Setiap email diinvestigasi secara manual — mulai dari header, body, hingga link yang terkandung di dalamnya — untuk mengidentifikasi indikator phishing dan aktivitas berbahaya.

> ⚠️ **Peringatan:** Domain, URL, dan IP address yang disebutkan dalam lab ini pernah atau masih menjadi host konten berbahaya. Jangan mengakses atau berinteraksi dengan URL tersebut di luar lingkungan lab yang terkontrol.

---

## 🎬 Video Tutorial

[![Manual Email Analysis - Phishing Investigation](https://img.youtube.com/vi/_TiD-1Q3RaM/maxresdefault.jpg)](https://youtu.be/_TiD-1Q3RaM?si=nkzucMZQc4P0L22w)

> 📺 **[Tonton di YouTube → Manual Email Analysis: Investigasi Email Phishing](https://youtu.be/_TiD-1Q3RaM?si=nkzucMZQc4P0L22w)**

---

## 🎯 Skenario

Kamu adalah **SOC Analyst Level 1** — garis pertahanan pertama organisasi. Tugasmu adalah:

- Menganalisis email mencurigakan yang dilaporkan oleh pengguna
- Mengekstrak indikator berbahaya (*Indicators of Compromise / IoC*)
- Mengidentifikasi teknik phishing yang digunakan
- Membantu tim SOC mengembangkan detection rules untuk mencegah ancaman serupa

---

## 🛠️ Tools yang Digunakan

| Tool | Fungsi |
|------|--------|
| **Thunderbird** | Membuka dan menganalisis file `.eml` |
| **View → Message Source** | Melihat raw header email secara lengkap |
| **CyberChef** | Decode URL, analisis karakter mencurigakan |
| **URL Expander** | Membuka redirect URL yang diperpendek |

---

## 📂 File yang Dianalisis

```
Phish3Case1.eml
```

Buka file ini di **Thunderbird** yang tersedia di virtual machine TryHackMe.

---

## 🔍 Metodologi Analisis Email

### Langkah 1 — Buka File `.eml` di Thunderbird

```
Buka Thunderbird
→ File → Open Saved Message
→ Pilih Phish3Case1.eml
```

---

### Langkah 2 — Analisis Header Email

Header email menyimpan informasi teknis yang tidak terlihat oleh pengguna biasa. Buka Message Source:

```
View → Message Source
(atau tekan Ctrl + U)
```

**Field header yang wajib diperiksa:**

| Field | Fungsi dalam Investigasi |
|-------|--------------------------|
| `From:` | Alamat pengirim yang ditampilkan — bisa dipalsukan |
| `Reply-To:` | Ke mana balasan dikirim — sering berbeda dari `From` pada phishing |
| `Return-Path:` | Alamat bounce email — mengungkap domain asli pengirim |
| `Received: from` | IP server yang mengirim email — lacak asal sesungguhnya |
| `To:` | Penerima yang dituju |
| `Subject:` | Judul email |
| `X-Originating-IP:` | IP asal pengiriman email |

---

### Langkah 3 — Analisis Body Email

Periksa isi email untuk mengidentifikasi:

```
✅ Brand yang ditiru (logo, warna, nama layanan)
✅ Urgensi palsu ("Akun akan ditutup!", "Verifikasi sekarang!")
✅ Link / tombol yang mencurigakan
✅ Tata bahasa yang tidak natural
✅ Pengirim yang tidak konsisten dengan brand
```

---

### Langkah 4 — Investigasi Link / Tombol

Jangan klik langsung. Hover di atas tombol atau link untuk melihat URL tujuan, atau inspect source HTML:

```
Klik kanan pada email body → View Page Source
Cari tag <a href="..."> untuk melihat URL asli
```

---

## 🧪 Hasil Investigasi — Phish3Case1.eml

### Temuan

| Pertanyaan | Jawaban |
|------------|---------|
| **Brand yang ditiru** | *(temukan saat analisis di lab)* |
| **Penerima email** | *(lihat field `To:` di header)* |
| **IP `Received: from`** | *(lihat di Message Source → `Received: from`)* |
| **Domain mencurigakan di `Return-Path`** | *(lihat field `Return-Path:` di header)* |
| **URL di balik tombol `UPDATE ACCOUNT NOW`** | `https://t.co/yuxfZm8KPg?amp=1` |

> 💡 **Catatan:** Isi tabel di atas dengan temuanmu sendiri saat menganalisis email di lab. Setiap investigasi melatih kemampuan analisis yang dibutuhkan SOC Analyst.

---

### Analisis URL: `https://t.co/yuxfZm8KPg?amp=1`

URL di balik tombol `UPDATE ACCOUNT NOW` menggunakan **URL shortener** (`t.co` milik Twitter/X) — teknik umum yang dipakai attacker untuk menyembunyikan domain tujuan sebenarnya.

```
URL yang ditampilkan : (teks tombol atau link pendek)
URL shortener        : https://t.co/yuxfZm8KPg?amp=1
URL tujuan akhir     : (gunakan URL expander untuk unshorten)
```

**Cara investigasi URL pendek tanpa mengkliknya:**

```
1. Gunakan tool: https://urlex.org atau https://checkshorturl.com
2. Paste URL pendek → lihat tujuan akhirnya
3. Analisis domain tujuan — apakah legitimate atau mencurigakan?
```

---

## 🚩 Indikator Phishing yang Ditemukan

Kumpulkan semua IoC (*Indicators of Compromise*) dari investigasi:

| Tipe IoC | Nilai | Keterangan |
|----------|-------|------------|
| **IP Address** | *(dari `Received: from`)* | IP server pengirim asli |
| **Domain** | *(dari `Return-Path`)* | Domain mencurigakan pengirim |
| **URL** | `https://t.co/yuxfZm8KPg?amp=1` | URL di balik tombol phishing |
| **Brand ditiru** | *(dari analisis body)* | Brand yang digunakan sebagai kamuflase |

---

## 🧠 Teknik Phishing yang Digunakan

Berdasarkan investigasi email ini, teknik yang diidentifikasi:

| Teknik | Penjelasan |
|--------|------------|
| **Brand Impersonation** | Menyamar sebagai brand terpercaya untuk membangun kepercayaan korban |
| **Urgency / Panic Trigger** | Pesan mendesak seperti "akun ditangguhkan" untuk memaksa korban bertindak cepat |
| **URL Shortener** | Menyembunyikan domain berbahaya di balik link pendek yang terlihat tidak berbahaya |
| **Fake CTA Button** | Tombol yang tampak resmi (`UPDATE ACCOUNT NOW`) tapi mengarah ke halaman palsu |

---

## 📋 Template Laporan SOC

Gunakan template ini untuk mendokumentasikan hasil investigasi email phishing:

```
============================
PHISHING EMAIL ANALYSIS REPORT
============================
Tanggal Analisis  : 
Analis            : 
File              : Phish3Case1.eml

--- HEADER ANALYSIS ---
From              : 
Reply-To          : 
Return-Path       : 
Received From IP  : 
To (Penerima)     : 
Subject           : 

--- BODY ANALYSIS ---
Brand ditiru      : 
Teknik urgency    : 
Tombol/Link       : 

--- INDICATORS OF COMPROMISE ---
IP Address        : 
Domain            : 
URL               : https://t.co/yuxfZm8KPg?amp=1

--- KESIMPULAN ---
Kategori          : Phishing
Tingkat Risiko    : High
Rekomendasi       : Block domain, report URL, awareness ke user
============================
```

---

## 📌 Kesimpulan

| Aspek | Poin Penting |
|-------|-------------|
| **Header email** | Menyimpan informasi teknis yang mengungkap asal dan jalur email sebenarnya |
| **Return-Path** | Sering berbeda dari `From` — mengungkap domain asli pengirim |
| **URL Shortener** | Teknik menyembunyikan domain berbahaya — selalu di-expand sebelum investigasi |
| **IoC** | Setiap temuan (IP, domain, URL) harus didokumentasikan untuk detection rules |

---

## 📚 Referensi

- [TryHackMe — Phishing Room](https://tryhackme.com/room/phishingemails1tryoe)
- [MXToolbox — Email Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx)
- [URL Expander Tool](https://urlex.org/)
- [CyberChef](https://gchq.github.io/CyberChef/)
- [MITRE ATT&CK — Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/_TiD-1Q3RaM?si=nkzucMZQc4P0L22w) | ⭐ Star repo ini jika membantu!*
