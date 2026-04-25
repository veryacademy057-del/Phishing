# 🎣 Phishing Detection & Analysis — The Greenholt Phish

> **Seri Lab:** Blue Team SOC Lab  
> **Platform:** [TryHackMe](https://tryhackme.com)  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/KwWMVzdtqxQ?si=xOWOUp2QGMAt4IRk)  
> **Kategori:** `Phishing Analysis, SOC Level 1`

---

## 📋 Deskripsi

Lab ini adalah investigasi nyata email phishing yang diterima oleh seorang sales executive di **Greenholt PLC**. Email yang tampak berasal dari customer dikenal ini mengandung berbagai red flag — mulai dari SPF/DKIM/DMARC yang gagal, Reply-To yang berbeda, hingga lampiran berbahaya yang disamarkan sebagai PDF. Kita akan menginvestigasi secara menyeluruh dan menghasilkan laporan eskalasi ke L2 Analyst.

---

## 🎬 Video Tutorial

[![Phishing Detection & Analysis - The Greenholt Phish](https://img.youtube.com/vi/KwWMVzdtqxQ/maxresdefault.jpg)](https://youtu.be/KwWMVzdtqxQ?si=xOWOUp2QGMAt4IRk)

> 📺 **[Tonton di YouTube → Phishing Detection & Analysis: The Greenholt Phish](https://youtu.be/KwWMVzdtqxQ?si=xOWOUp2QGMAt4IRk)**

---

## 🏢 Latar Belakang Kasus

Seorang **sales executive di Greenholt PLC** menerima email mencurigakan dari seseorang yang mengaku sebagai customer yang dikenal. Email tersebut langsung menunjukkan beberapa red flag:

- Greeting yang generic (bukan nama penerima)
- Permintaan yang melibatkan transfer uang besar
- Lampiran yang tidak pernah diminta
- Perilaku tidak sesuai dengan komunikasi customer biasanya

Email dieskalasi ke **SOC** untuk investigasi lebih lanjut.

---

## 📧 Isi Email Asli

```
From    : Mr. James Jackson <info@mutawamarine.com>
To      : webmaster@redacted.org
Subject : webmaster@redacted.org your: Transfer Reference Number:(09674321)

Good day webmaster@redacted.org,

As instructed, funds has been transferred to your account
this morning via SWIFT.

Details are as below and a receipt of payment is attached.

Interbank Transfer Reference Number : 09674321
Transaction Status                  : Successful
Transaction Date / Time             : 10-06-2020 09:18:55
Transaction Description             : Balance / Final Payment
From Account                        : 3105234819
Amount                              : 149,650
Currency                            : USD
Bank Charges                        : $ 146.05

Best regards,
Mr. James Jackson
Accounts Payable
SEC MARINE SERVICES PTE LTD
```

---

## 🔍 Langkah 1 — Analisis Header Email

### 1.1 Informasi Dasar

| Field | Nilai |
|-------|-------|
| **Transfer Reference** | `09674321` |
| **Display Name** | Mr. James Jackson |
| **From Email** | `info@mutawamarine.com` |
| **Reply-To** | `info.mutawamarine@mail.com` |
| **To** | `webmaster@redacted.org` |
| **Date** | 09 Jun 2020 22:58:27 -0700 |

> 🚨 **Red Flag #1 — Reply-To berbeda dari From!**  
> Jika korban membalas email, balasan akan masuk ke `info.mutawamarine@mail.com` yang dikontrol attacker — bukan ke `info@mutawamarine.com`. Ini teknik klasik untuk menangkap balasan korban.

---

### 1.2 Routing Email

```
Origin   : hwsrv-737338.hostwindsdns.com (192.119.71.157)
               │  helo=mutawamarine.com  ← SPOOFED!
               ▼
Relay 1  : sub.redacted.com
               ▼
Relay 2  : mta4212.mail.bf1.yahoo.com
               ▼
Delivery : atlas125.free.mail.bf1.yahoo.com
               ▼
Final    : redacted@yahoo.com
```

> 🚨 **Red Flag #2 — HELO Spoofing!**  
> Email dikirim dari server **HostWinds** (`192.119.71.157`) yang berpura-pura menjadi `mutawamarine.com` menggunakan HELO spoofing. Server asli mutawamarine.com tidak pernah mengirim email ini.

---

### 1.3 Originating IP Address

| Field | Nilai |
|-------|-------|
| **IP Address** | `192.119.71.157` |
| **Hostname** | `hwsrv-737338.hostwindsdns.com` |
| **Owner** | HostWinds LLC (layanan web hosting murah) |
| **Port** | `51810` |

> 🚨 **Red Flag #3 — Server Hosting Murah!**  
> IP ini adalah server hosting murah milik HostWinds, bukan server resmi mutawamarine.com. Attacker menyewa hosting untuk mengirim email phishing ini.

---

## 🔍 Langkah 2 — Analisis SPF, DKIM & DMARC

### 2.1 SPF Check

```
Received-SPF: fail
(domain of mutawamarine.com does not designate
192.119.71.157 as permitted sender)

spf=fail smtp.mailfrom=mutawamarine.com
```

**SPF Record mutawamarine.com:**
```dns
v=spf1 mx a ptr ?all
```

| Field | Nilai |
|-------|-------|
| **Status** | ❌ FAIL |
| **Penjelasan** | Server `192.119.71.157` tidak ada di SPF record domain |
| **SPF Record** | `v=spf1 mx a ptr ?all` |
| **Kelemahan** | Menggunakan `?all` (neutral) — bukan `~all` atau `-all` |

> 💡 `?all` artinya semua server lain diperlakukan **neutral** — bukan fail/reject. Ini konfigurasi SPF yang sangat lemah dan tidak memberikan proteksi yang cukup.

---

### 2.2 DKIM Check

```
Authentication-Results: atlas125.free.mail.bf1.yahoo.com;
dkim=tidak ada
```

| Field | Nilai |
|-------|-------|
| **Status** | ❌ None |
| **Penjelasan** | Tidak ada DKIM signature sama sekali di email ini |

> 🚨 **Red Flag #4 — Tidak ada DKIM!**  
> Tidak ada digital signature yang membuktikan email ini benar-benar dikirim dari `mutawamarine.com`. Email legitimate dari perusahaan resmi hampir selalu memiliki DKIM signature.

---

### 2.3 DMARC Check

```
dmarc=unknown
```

**DMARC Record mutawamarine.com:**
```dns
v=DMARC1; p=none; rua=mailto:report@mutawamarine.com
```

| Field | Nilai |
|-------|-------|
| **Status** | ⚠️ Unknown |
| **Policy** | `p=none` — hanya monitor, tidak reject/quarantine |
| **Dampak** | Meskipun SPF dan DKIM gagal, email tetap masuk ke inbox |

> 🚨 **Red Flag #5 — DMARC Policy Lemah!**  
> Dengan `p=none`, domain `mutawamarine.com` tidak memproteksi dirinya dari spoofing. Attacker leluasa memalsukan domain ini.

---

### 2.4 Ringkasan Autentikasi

| Mekanisme | Hasil | Penjelasan |
|-----------|-------|------------|
| **SPF** | ❌ FAIL | Server pengirim tidak authorized oleh domain |
| **DKIM** | ❌ None | Tidak ada digital signature |
| **DMARC** | ⚠️ Unknown | Policy `p=none` — email tetap masuk meskipun SPF/DKIM gagal |

---

## 🔍 Langkah 3 — Analisis Lampiran

### 3.1 Informasi Lampiran

```
Content-Type: application/octet-stream
filename="SWT_#09674321____PDF__.CAB"
Content-Transfer-Encoding: base64
```

| Field | Nilai |
|-------|-------|
| **Nama File** | `SWT_#09674321____PDF__.CAB` |
| **Ekstensi Terlihat** | `.CAB` (disamarkan seolah PDF) |
| **Tipe File Asli** | RAR Archive |

> 🚨 **Red Flag #6 — Double Extension Trick!**  
> Nama file `____PDF__.CAB` dirancang untuk membuat korban mengira ini file PDF bukti transfer. Padahal ekstensi aslinya adalah `.CAB` yang merupakan RAR Archive — kemungkinan besar berisi malware.

---

### 3.2 Cek SHA256 Hash

Jalankan perintah berikut di terminal virtual machine TryHackMe:

```bash
sha256sum "SWT_#09674321____PDF__.CAB"
```

---

### 3.3 Investigasi di VirusTotal

```
1. Buka https://www.virustotal.com
2. Pilih tab "Search"
3. Paste hasil SHA256 hash dari langkah sebelumnya
4. Analisis hasilnya
```

**Yang perlu diperhatikan di VirusTotal:**

| Field | Yang Dicari |
|-------|-------------|
| **File Type** | Konfirmasi tipe file asli (RAR Archive) |
| **Detection** | Jumlah AV vendor yang mendeteksi sebagai malicious |
| **Behavior** | Aktivitas yang dilakukan file saat dieksekusi |
| **Relations** | Domain/IP yang dihubungi file tersebut |

---

## 🔍 Langkah 4 — Analisis Konten & Social Engineering

### Red Flag di Konten Email

| # | Red Flag | Detail |
|---|----------|--------|
| 1 | **Generic Greeting** | `"Good day webmaster@..."` — bukan nama asli penerima |
| 2 | **Transfer Tidak Diminta** | `"As instructed"` — padahal korban tidak pernah minta transfer |
| 3 | **Jumlah Besar** | $149,650 — membuat korban panik |
| 4 | **Urgency Palsu** | `"Transaction Status: Successful"` — seolah sudah terjadi dan tidak bisa dibatalkan |
| 5 | **Lampiran Berbahaya** | `.CAB` disamarkan sebagai PDF bukti transfer |
| 6 | **Reply-To Berbeda** | Balasan masuk ke email yang dikontrol attacker |

### Teknik Social Engineering yang Digunakan

| Teknik | Penjelasan |
|--------|------------|
| **Fear & Urgency** | Transfer $149,650 sudah berhasil → korban panik dan terburu-buru mengklik lampiran |
| **Authority** | Mengaku dari "Accounts Payable" perusahaan — terkesan resmi |
| **Familiarity** | Menggunakan domain `mutawamarine.com` yang legitimate — terlihat terpercaya |
| **Deception** | File `.CAB` disamarkan sebagai PDF dengan nama yang menipu |

---

## 🚨 Langkah 5 — Verdict & Kesimpulan

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         HASIL INVESTIGASI
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SPF              → ❌ FAIL
DKIM             → ❌ None
DMARC            → ⚠️ Unknown (p=none)
Reply-To         → 🚨 Berbeda dari From address
Originating IP   → 🚨 HostWinds LLC (bukan server resmi)
                      192.119.71.157
Attachment       → 🚨 RAR Archive disamarkan sebagai .CAB/PDF
Content          → 🚨 Social engineering transfer palsu
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
VERDICT          → 🚨 TRUE POSITIVE — PHISHING!
SEVERITY         → 🔴 HIGH
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📋 Langkah 6 — Indicators of Compromise (IoC)

| Tipe | Nilai | Keterangan |
|------|-------|------------|
| **IP Address** | `192.119.71.157` | Server pengirim asli (HostWinds LLC) |
| **Domain (Spoofed)** | `mutawamarine.com` | Domain yang dipalsukan |
| **Email From** | `info@mutawamarine.com` | Alamat pengirim yang dipalsukan |
| **Email Reply-To** | `info.mutawamarine@mail.com` | Email attacker untuk tangkap balasan |
| **Filename** | `SWT_#09674321____PDF__.CAB` | Lampiran berbahaya |
| **File Type Asli** | RAR Archive | Bukan PDF seperti yang diklaim |
| **SHA256** | *(hasil sha256sum)* | Hash untuk identifikasi malware |

---

## 📤 Langkah 7 — Template Eskalasi ke L2

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
SUBJECT: [ESCALATION] Phishing Email - True Positive - HIGH Severity
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Kepada L2 Analyst,

Ditemukan email phishing dengan detail sebagai berikut:

SEVERITY     : HIGH
STATUS       : True Positive
DATE         : 10 Jun 2020 05:58:55 UTC
CASE         : Greenholt PLC - Suspicious Transfer Email

IOC:
├── Source IP   : 192.119.71.157 (HostWinds LLC)
├── From        : info@mutawamarine.com (SPOOFED)
├── Reply-To    : info.mutawamarine@mail.com (Attacker)
└── Target      : webmaster@redacted.org

AUTHENTICATION:
├── SPF    : FAIL
├── DKIM   : None
└── DMARC  : Unknown (p=none)

ATTACHMENT:
├── Filename  : SWT_#09674321____PDF__.CAB
├── Real Type : RAR Archive
├── SHA256    : (hasil sha256sum)
└── Status    : Belum dibuka — JANGAN DIBUKA

ACTION REQUIRED L2:
1. Analisa lampiran di sandbox environment
2. Block IP 192.119.71.157 di firewall / email gateway
3. Block domain mutawamarine.com di email filter
4. Block email info.mutawamarine@mail.com
5. Scan endpoint target — pastikan tidak terinfeksi
6. Threat hunting — cari target lain yang terima email serupa
7. Cek VirusTotal hash untuk identifikasi jenis malware

Terlampir: Email header lengkap + raw email (.eml)

Regards,
L1 SOC Analyst
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 📚 Pelajaran dari Kasus Ini

| Pelajaran | Penjelasan |
|-----------|------------|
| **Selalu cek Reply-To** | Reply-To berbeda dari From adalah red flag terbesar — dan sering terlewat |
| **Verifikasi SPF/DKIM/DMARC** | Ketiga gagal sekaligus = email pasti tidak authentic |
| **Waspadai ekstensi ganda** | `____PDF__.CAB` adalah teknik klasik penyamaran malware |
| **Jangan terburu-buru** | Social engineering selalu menciptakan rasa urgency — itu justru sinyal waspada |
| **Verifikasi out-of-band** | Konfirmasi transfer via telepon/chat langsung ke pengirim, jangan balas email |

---

## 📚 Referensi

- [TryHackMe — Phishing Analysis Room](https://tryhackme.com)
- [VirusTotal](https://www.virustotal.com)
- [MXToolbox Email Header Analyzer](https://mxtoolbox.com/EmailHeaders.aspx)
- [MITRE ATT&CK — Phishing (T1566)](https://attack.mitre.org/techniques/T1566/)
- [HostWinds IP Lookup — IPInfo](https://ipinfo.io/192.119.71.157)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/KwWMVzdtqxQ?si=xOWOUp2QGMAt4IRk) | ⭐ Star repo ini jika membantu!*
