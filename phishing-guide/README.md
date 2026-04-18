# 🎣 Phishing — Teknik Penipuan untuk Mencuri Data Digital

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/srCu30ENnJU?si=kepQHsodnYk5IZPO)  
> **Kategori:** `Security Awareness, Social Engineering`

---

## 📋 Deskripsi

Materi ini membahas **phishing** secara mendalam — mulai dari cara kerja, teknik yang digunakan attacker, analisis ciri-ciri phishing, hingga teknik lanjutan seperti **Homograph Attack**. Pemahaman ini penting bagi Blue Team untuk mengenali, menganalisis, dan mencegah serangan phishing.

---

## 🎬 Video Tutorial

[![Phishing - Teknik Penipuan untuk Mencuri Data Digital](https://img.youtube.com/vi/srCu30ENnJU/maxresdefault.jpg)](https://youtu.be/srCu30ENnJU?si=kepQHsodnYk5IZPO)

> 📺 **[Tonton di YouTube → Phishing: Cara Kerja, Dampak, dan Pencegahan](https://youtu.be/srCu30ENnJU?si=kepQHsodnYk5IZPO)**

---

## 🎯 Apa itu Phishing?

**Phishing** adalah upaya penyerang untuk menipu korban agar memberikan data sensitif dengan **menyamar sebagai pihak terpercaya** — seperti bank, platform media sosial, atau institusi resmi.

> 💡 **Contoh nyata:** Email bertuliskan *"Bank Anda bermasalah, klik di sini untuk verifikasi akun"* yang mengarahkan ke website palsu yang tampak identik dengan website bank asli.

---

## 🔄 Alur Serangan Phishing

```
[1] Penyerang membuat website/email palsu
    (tampilan dibuat semirip mungkin dengan aslinya)
            │
            ▼
[2] Korban menerima link
    (via email, SMS, DM media sosial, dll)
            │
            ▼
[3] Korban klik link dan membuka website palsu
            │
            ▼
[4] Korban login → data dikirim ke server penyerang
            │
            ▼
[5] Akun korban diambil alih
    (email, banking, media sosial, dll)
```

---

## ⚠️ Teknik yang Digunakan Attacker

| Teknik | Penjelasan |
|--------|------------|
| **Spoofing** | Menyamar sebagai pengirim email resmi — alamat pengirim dibuat semirip mungkin |
| **Social Engineering** | Memanfaatkan emosi korban — rasa panik, takut, atau terburu-buru untuk bertindak cepat |
| **Fake Website (Clone)** | Membuat salinan website asli dengan tampilan yang 100% identik |
| **URL Redirect** | Link yang terlihat aman di permukaan, tapi sebenarnya mengarahkan ke domain berbeda |

---

## 🗂️ Jenis-Jenis Phishing

| Jenis | Target | Metode |
|-------|--------|--------|
| **Email Phishing** | Massal — siapa saja | Email dengan link/attachment berbahaya |
| **Spear Phishing** | Target spesifik (misal karyawan perusahaan tertentu) | Email yang dipersonalisasi dengan info korban |
| **Whaling** | Eksekutif / pejabat perusahaan | Email yang sangat meyakinkan, topik bisnis |
| **Smishing** | Pengguna smartphone | Link berbahaya via SMS |
| **Vishing** | Siapa saja | Penipuan via telepon, penyerang berpura-pura jadi petugas resmi |

---

## 🔍 Ciri-Ciri Teknis Phishing

**Kenapa phishing sering berhasil?** Karena korban tidak teliti dan terburu-buru saat menerima pesan yang terkesan mendesak.

| Ciri | Contoh / Penjelasan |
|------|---------------------|
| **Domain mencurigakan** | `bank-login123.com` bukan `bank.com` |
| **HTTPS bukan jaminan aman** | Sertifikat SSL hanya membuktikan koneksi terenkripsi — bukan membuktikan website tersebut legitimate |
| **Link redirect** | URL yang ditampilkan berbeda dari URL tujuan sebenarnya |
| **Header email mencurigakan** | `From: support@bank.com` tapi `Reply-To: hacker@domain.xyz` |
| **Urgensi berlebihan** | *"Akun Anda akan ditutup dalam 24 jam!"* |

> ⚠️ **Insight penting:** Phishing modern bisa sangat meyakinkan — hampir tidak bisa dibedakan secara visual. Analisis teknis diperlukan untuk memastikan keaslian sebuah domain atau email.

---

## 💥 Dampak Serangan Phishing

| Dampak | Keterangan |
|--------|------------|
| **Akun diretas** | Email, media sosial, akun kerja diambil alih |
| **Kerugian finansial** | Rekening bank dikuras, transaksi tidak sah |
| **Data perusahaan bocor** | Credential karyawan digunakan untuk akses internal |
| **Pencurian identitas** | Data pribadi disalahgunakan untuk penipuan lanjutan |

---

## 🛡️ Pencegahan Phishing

| Langkah | Penjelasan |
|---------|------------|
| **Cek URL sebelum login** | Pastikan domain benar-benar milik layanan yang dimaksud |
| **Jangan klik link sembarangan** | Terutama dari email, SMS, atau DM yang tidak dikenal |
| **Aktifkan 2FA** | Multi-Factor Authentication mempersulit attacker meskipun password sudah dicuri |
| **Gunakan password manager** | Tidak akan otofill di domain palsu — pertahanan tersembunyi yang efektif |
| **Edukasi dan awareness** | **Ini yang paling penting** — manusia adalah target utama phishing, bukan sistem |

> 💡 **Kesimpulan:** Phishing bukan soal teknologi yang canggih — tapi tentang **manipulasi psikologis manusia**. Pertahanan terbaik adalah kesadaran dan ketelitian pengguna.

---

## 🔬 Praktik: Homograph Attack — Domain Asli vs Palsu

### Apa itu Homograph Attack?

Teknik phishing lanjutan di mana attacker **mengganti huruf dalam domain dengan karakter Unicode yang tampak identik** dari alfabet lain — sehingga domain palsu tidak bisa dibedakan secara visual.

### Contoh Kasus

| | Domain |
|-|--------|
| **Asli** | `google.com` |
| **Palsu** | `gοοgle.com` |

> 👁️ Secara visual keduanya terlihat **sama persis** — tapi sebenarnya menggunakan karakter yang berbeda.

---

### Analisis dengan CyberChef

Gunakan **CyberChef** ([gchq.github.io/CyberChef](https://gchq.github.io/CyberChef/)) untuk menganalisis perbedaan karakter:

**Konversi ke Hex:**

| Karakter | Tampilan | Hex | Keterangan |
|----------|----------|-----|------------|
| `o` (asli) | o | `6f` | Huruf Latin biasa |
| `ο` (palsu) | ο | `ce bf` | Huruf Greek Omicron |

**Unicode Code Point:**

| Karakter | Unicode | Alfabet |
|----------|---------|---------|
| `o` | `U+006F` | Latin (normal) |
| `ο` | `U+03BF` | Greek (Omicron) |

---

### Cara Mendeteksi Homograph Attack

```
1. Copy URL yang mencurigakan
2. Buka CyberChef → pilih operasi "To Hex" atau "Encode Text"
3. Jika karakter menghasilkan lebih dari 1 byte (contoh: ce bf)
   → Domain tersebut menggunakan karakter non-Latin
   → KEMUNGKINAN BESAR domain phishing!
```

**Indikasi lain:**

```
Jika URL di-copy ke text editor dan muncul karakter aneh
seperti: gÎ¿Î¿gle.com
→ Konfirmasi bahwa domain menggunakan karakter Unicode palsu
```

---

### Kesimpulan Homograph Attack

| Aspek | Penjelasan |
|-------|------------|
| **Teknik** | Mengganti huruf dengan karakter mirip dari alfabet lain (Greek, Cyrillic, dll) |
| **Tujuan** | Domain palsu terlihat identik dengan domain asli secara visual |
| **Deteksi** | Harus dianalisis menggunakan tools — tidak bisa dilihat dengan mata biasa |
| **Tool** | CyberChef, browser address bar (IDN encoding) |

---

## 📌 Ringkasan

| Konsep | Poin Penting |
|--------|-------------|
| **Phishing** | Manipulasi psikologis — bukan hanya serangan teknis |
| **Spear Phishing** | Paling berbahaya karena dipersonalisasi untuk target spesifik |
| **HTTPS** | Bukan jaminan website aman — hanya membuktikan koneksi terenkripsi |
| **Homograph Attack** | Domain palsu yang tidak bisa dibedakan secara visual |
| **Pertahanan terbaik** | Kesadaran pengguna + 2FA + verifikasi URL |

---

## 📚 Referensi

- [CyberChef — GCHQ](https://gchq.github.io/CyberChef/)
- [MITRE ATT&CK — Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)
- [PhishTank — Database Phishing](https://www.phishtank.com/)
- [Google Safe Browsing](https://safebrowsing.google.com/)
- [IDN Homograph Attack — Wikipedia](https://en.wikipedia.org/wiki/IDN_homograph_attack)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/srCu30ENnJU?si=kepQHsodnYk5IZPO) | ⭐ Star repo ini jika membantu!*
