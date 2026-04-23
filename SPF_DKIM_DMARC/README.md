# 🔐 SPF, DKIM & DMARC — Autentikasi Email untuk Deteksi Phishing

> **Seri Lab:** Blue Team SOC Lab  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/O0KCcq79rYw?si=XSp5ldZISs4vFBnS)  
> **Kategori:** `Email Security, Phishing Analysis`

---

## 📋 Deskripsi

Materi ini membahas tiga mekanisme autentikasi email yang wajib dipahami oleh setiap SOC Analyst — **SPF**, **DKIM**, dan **DMARC**. Ketiga mekanisme ini bekerja bersama untuk memverifikasi keaslian email dan menjadi dasar analisis saat menginvestigasi phishing.

---

## 🎬 Video Tutorial

[![SPF, DKIM & DMARC - Autentikasi Email](https://img.youtube.com/vi/O0KCcq79rYw/maxresdefault.jpg)](https://youtu.be/O0KCcq79rYw?si=XSp5ldZISs4vFBnS)

> 📺 **[Tonton di YouTube → SPF, DKIM & DMARC: Keamanan Email yang Wajib Diketahui](https://youtu.be/O0KCcq79rYw?si=XSp5ldZISs4vFBnS)**

---

## 📨 SPF — Sender Policy Framework

### Apa itu SPF?

SPF adalah mekanisme yang memverifikasi apakah **server pengirim berhak mengirim email atas nama domain tertentu**.

> 💡 **Analogi:** Seperti daftar tamu di gedung. Hanya server yang namanya ada di daftar DNS yang boleh mengirim email mewakili domain tersebut.

### Cara Kerja SPF

```
[1] Domain owner buat SPF record di DNS
    Contoh: v=spf1 ip4:192.168.1.1 include:gmail.com ~all
                    │
                    ▼
[2] Email dikirim dari server
                    │
                    ▼
[3] Server penerima cek SPF record di DNS domain pengirim
                    │
                    ▼
[4] Apakah IP server pengirim ada di SPF record?
         │                        │
         ▼                        ▼
      PASS ✅                  FAIL ❌
  Email legitimate          Email mencurigakan
```

### Hasil SPF

| Status | Arti |
|--------|------|
| `Pass` | Server pengirim diizinkan oleh domain |
| `Fail` | Server pengirim tidak diizinkan |
| `SoftFail` | Server tidak diizinkan tapi tidak langsung ditolak (`~all`) |
| `Neutral` | Domain tidak punya kebijakan jelas |
| `None` | Tidak ada SPF record di DNS |

---

## ✍️ DKIM — DomainKeys Identified Mail

### Apa itu DKIM?

DKIM memverifikasi bahwa **email tidak dimodifikasi selama pengiriman** menggunakan digital signature (tanda tangan digital).

> 💡 **Analogi:** Seperti segel pada amplop surat. Jika segelnya rusak saat tiba, berarti isi surat sudah diubah oleh pihak lain di tengah perjalanan.

### Cara Kerja DKIM

```
SISI PENGIRIM:
[1] Server pengirim membuat digital signature
    dari isi email menggunakan Private Key
                    │
                    ▼
[2] Signature ditambahkan ke header email:
    DKIM-Signature: v=1; a=rsa-sha256; d=domain.com...

─────────────────────────────────────────
Email melakukan perjalanan ke server tujuan
─────────────────────────────────────────

SISI PENERIMA:
[3] Server penerima mengambil Public Key dari DNS
                    │
                    ▼
[4] Verifikasi signature menggunakan Public Key
                    │
         ┌──────────┴──────────┐
         ▼                     ▼
      PASS ✅               FAIL ❌
   Email asli &          Email dimodifikasi
   tidak diubah          di tengah jalan
```

### Hasil DKIM

| Status | Arti |
|--------|------|
| `Pass` | Email tidak dimodifikasi, signature valid |
| `Fail` | Email dimodifikasi atau signature tidak valid |
| `None` | Tidak ada DKIM signature di email |
| `Neutral` | DKIM ada tapi tidak bisa diverifikasi |

---

## 🏛️ DMARC — Domain-based Message Authentication Reporting and Conformance

### Apa itu DMARC?

DMARC adalah **kebijakan** yang menentukan apa yang harus dilakukan server penerima jika email gagal verifikasi SPF atau DKIM. DMARC juga memberikan laporan ke domain owner tentang email yang gagal verifikasi.

> 💡 **Analogi:** Seperti SOP satpam gedung. Jika ada orang yang tidak ada di daftar tamu (SPF fail) atau segelnya rusak (DKIM fail) — SOP menentukan apakah orang itu harus diusir (*reject*), ditahan dulu (*quarantine*), atau dibiarkan masuk tapi dicatat (*none*).

### Cara Kerja DMARC

```
Email masuk
      │
      ▼
Cek SPF + DKIM
      │
      ├── Keduanya PASS → Email OK, masuk normal ✅
      │
      └── Salah satu atau keduanya FAIL
                    │
                    ▼
            Cek DMARC Policy
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
      none      quarantine    reject
   (log saja)  (folder spam)  (ditolak)
```

### DMARC Policy

| Policy | Arti | Aksi |
|--------|------|------|
| `none` | Monitor saja | Email tetap masuk, hanya dicatat di laporan |
| `quarantine` | Karantina | Email masuk ke folder spam |
| `reject` | Tolak | Email langsung ditolak — tidak sampai ke penerima |

### Contoh DMARC Record di DNS

```dns
v=DMARC1; p=reject; rua=mailto:report@domain.com; pct=100
```

| Parameter | Arti |
|-----------|------|
| `p=reject` | Policy: tolak semua email yang gagal verifikasi |
| `rua=mailto:` | Kirim laporan agregat ke alamat email ini |
| `pct=100` | Policy berlaku untuk 100% email |

---

## 🔗 Bagaimana Ketiga Mekanisme Bekerja Bersama

```
Email Masuk
      │
      ▼
┌─────────────────────────────────────┐
│            SPF Check                │
│  "Apakah server ini diizinkan       │
│   mengirim email dari domain X?"    │
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│            DKIM Check               │
│  "Apakah email ini asli dan         │
│   tidak dimodifikasi di perjalanan?"│
└─────────────────────────────────────┘
      │
      ▼
┌─────────────────────────────────────┐
│           DMARC Check               │
│  "Apa yang harus dilakukan jika     │
│   SPF atau DKIM gagal?"             │
└─────────────────────────────────────┘
      │
      ▼
Keputusan: PASS / QUARANTINE / REJECT
```

---

## 🧪 Contoh Kasus di Dunia Nyata

| Kondisi | SPF | DKIM | DMARC | Hasil |
|---------|-----|------|-------|-------|
| Email legitimate | ✅ Pass | ✅ Pass | ✅ Pass | Email masuk normal |
| Email phishing (domain palsu) | ❌ Fail | ❌ Fail | ❌ Reject | Email langsung ditolak |
| Email spoofing (pura-pura dari domain asli) | ❌ Fail | ❌ Fail | ⚠️ Quarantine | Masuk folder spam |
| Email dimodifikasi di tengah jalan | ✅ Pass | ❌ Fail | ⚠️ Quarantine | Masuk folder spam |

---

## 🔍 Cara Membaca Hasil di Header Email

Saat menganalisis email phishing, cari field berikut di **Message Source**:

```
Authentication-Results: mx.google.com;
   spf=pass (google.com: domain of sender@domain.com designates 1.2.3.4 as permitted sender)
   dkim=fail (signature did not verify)
   dmarc=fail (p=REJECT sp=REJECT dis=REJECT) header.from=domain.com
```

**Cara membacanya:**

| Field | Nilai | Interpretasi |
|-------|-------|--------------|
| `spf=pass` | Pass | Server pengirim legitimate |
| `dkim=fail` | Fail | Email kemungkinan dimodifikasi |
| `dmarc=fail` | Fail + p=REJECT | Seharusnya ditolak oleh server |

> 💡 Kombinasi `spf=pass` tapi `dkim=fail` adalah pola yang sering ditemukan pada email yang **dimodifikasi setelah dikirim** — indikator kuat bahwa email tersebut telah dimanipulasi.

---

## 🛠️ Tools untuk Verifikasi SPF/DKIM/DMARC

| Tool | Fungsi | Link |
|------|--------|------|
| **MXToolbox** | Cek SPF, DKIM, DMARC record sebuah domain | [mxtoolbox.com](https://mxtoolbox.com) |
| **Google Admin Toolbox** | Analisis header email lengkap | [toolbox.googleapps.com](https://toolbox.googleapps.com/apps/messageheader/) |
| **Mail Header Analyzer** | Visualisasi perjalanan email | [mxtoolbox.com/EmailHeaders.aspx](https://mxtoolbox.com/EmailHeaders.aspx) |
| **DMARC Analyzer** | Cek DMARC policy domain | [dmarcanalyzer.com](https://www.dmarcanalyzer.com) |

---

## 📌 Kesimpulan

| Mekanisme | Fungsi | Yang Diverifikasi |
|-----------|--------|-------------------|
| **SPF** | Verifikasi pengirim | Apakah server diizinkan kirim email dari domain ini? |
| **DKIM** | Verifikasi integritas | Apakah isi email tidak diubah selama pengiriman? |
| **DMARC** | Kebijakan penegakan | Apa yang dilakukan jika SPF/DKIM gagal? |

> 💡 Ketiga mekanisme ini adalah yang pertama kali diperiksa saat menganalisis email phishing. Memahami hasilnya (`pass`, `fail`, `quarantine`, `reject`) adalah kemampuan dasar wajib bagi setiap SOC Analyst.

---

## 📚 Referensi

- [SPF Record Syntax — OpenSPF](http://www.open-spf.org/SPF_Record_Syntax/)
- [DKIM.org Documentation](https://dkim.org/)
- [DMARC.org Official Guide](https://dmarc.org/overview/)
- [MXToolbox SPF/DKIM/DMARC Checker](https://mxtoolbox.com/)
- [MITRE ATT&CK — Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/O0KCcq79rYw?si=XSp5ldZISs4vFBnS) | ⭐ Star repo ini jika membantu!*
