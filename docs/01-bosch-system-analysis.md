# 📘 Analisis Mendalam Sistem Bosch Solution 2000 (ICP-SOL2-P)

Dokumen ini membedah seluruh aspek teknis, operasional, kelistrikan, diagnostik, dan antarmuka lapangan dari panel alarm komersial **Bosch Solution 2000 (ICP-SOL2-P)**. Dibuat berdasarkan standar teknisi instalasi (*field engineer manual*).

---

## 1. Arsitektur Kelistrikan, Power Supply & Battery Supervision

```
                                [ 16-18 VAC Mains Plug-Pack (TF008-B) ]
                                                   |
                                                   v
   +-----------------------+              +------------------+
   |  12V 7Ah SLA Battery  |<------------>|  Onboard Linear/ |
   |  - Dynamic Load Test  |              |  Switching PMIC  |
   |  - Low Batt (<11.5V)  |              +------------------+
   |  - Auto Cutoff (<10.2)|                       |
   +-----------------------+                       |
                                                   +---> +13.8VDC AUX (1.0A PTC Fuse)
                                                   +---> +12VDC Switched Strobe
                                                   +---> Supervised Horn/Siren Driver
```

### 1.1 Input Daya Utama (AC Power) & Proteksi
* **Spesifikasi:** 16–18 VAC (50/60 Hz), disuplai via transformator eksternal **TF008-B (1.5A)**.
* **AC Mains Fail Monitoring:** Sirkuit zero-crossing membaca siklus AC. Jika AC terputus:
  * Indikator `MAINS` pada codepad berkedip.
  * Codepad mengeluarkan bunyi *beep* setiap 1 menit (dapat di-acknowledge via tombol `[#]`).
  * **AC Fail Delay Timer:** 0–60 menit (default 1–5 menit) sebelum mengirim sinyal transmisi laporan (*report*) agar tidak membanjiri CMS saat tegangan PLN kedip (*brownout*).

### 1.2 Sirkuit Manajemen Baterai (SLA 12V 7Ah)
* **Float / Trickle Charging:** ~13.8 VDC regulated.
* **Dynamic Battery Load Test:**
  * Diuji secara otomatis setiap **4 jam** dan setiap kali sistem melakukan **Arming**.
  * Panel menarik arus beban buatan selama beberapa milidetik untuk mengukur *internal resistance* / *voltage sag* (bukan hanya membaca tegangan mengambang tanpa beban).
* **Threshold Supervisi:**
  * `Low Battery Warning`: Terpicu saat voltase < **11.5 VDC**.
  * `Deep Discharge Cut-Off`: Sirkuit proteksi internal otomatis memutus jalur baterai pada **~10.2–10.5 VDC** untuk mencegah kerusakan kimiawi sel timbal asam (*sulfation*).

### 1.3 Catu Daya Aksesori (AUX Power)
* **Spesifikasi:** 13.8 VDC, dilindungi sekring *Polyswitch / PTC Resettable Fuse* dengan batas beban kontinu **~1.0A** untuk piranti PIR, beam sensor, codepad, dan modul eksternal.

---

## 2. Sirkuit Pembacaan Zona (Zone Loops & Topologi Resistor)

### 2.1 Kapasitas Zona Sol 2000
* **Onboard:** 4 terminal fisik (Z1–Z4 dan COM).
* **Maksimum Zona:** 8 zona (menggunakan metode *Split EOL / Zone Doubling*).

### 2.2 Topologi Resistor EOL Lengkap

```
1. SINGLE EOL (SEOL - Default 3.3kΩ):
   [ Z1 ] ---- [ NC Alarm Contact ] ---- [ 3.3kΩ ] ---- [ COM ]
   - Resistansi ~3.3kΩ = SEALED (Normal)
   - Resistansi 0Ω atau Open = UNSEALED (Alarm)

2. DUAL EOL (DEOL - Anti-Sabotage 4-State):
   [ Z1 ] ---- [ NC Tamper ] ---- [ NC Alarm + 3.3kΩ Parallel ] ---- [ 3.3kΩ Series ] ---- [ COM ]
   - 0 Ω         : TAMPER SHORT (Kabel dikonsletkan)
   - 3.3 kΩ      : NORMAL / SEALED (Aman)
   - 6.6 kΩ      : ALARM / UNSEALED (Sensor terpicu)
   - Open Loop (∞): TAMPER OPEN / CUT (Kabel dipotong penyusup)

3. SPLIT EOL / ZONE DOUBLING (ZD - 3.3kΩ & 6.8kΩ):
   Menyatukan Zone 1 (Low Zone) & Zone 5 (High Zone) pada satu pasang terminal [Z1 - COM]:
   - Low Zone Resistor (R1)  : 3.3 kΩ
   - High Zone Resistor (R2) : 6.8 kΩ
   - Pembacaan Impedansi:
     * ~2.22 kΩ (Paralel)    : Z1 Normal & Z5 Normal
     * 6.8 kΩ                : Z1 Alarm & Z5 Normal
     * 3.3 kΩ                : Z5 Alarm & Z1 Normal
     * Open Circuit (∞)      : Z1 Alarm & Z5 Alarm
```

---

## 3. Matriks Tipe Zona (Zone Types)

Setiap zona memiliki alamat konfigurasi tipe zona (*Zone Type Location*):

| Kode | Tipe Zona | Karakteristik Perilaku Sistem |
| :--- | :--- | :--- |
| **0** | **Instant** | Memicu alarm dan sirene seketika jika sensor terbuka saat sistem dalam kondisi *Armed* (AWAY/STAY). Cocok untuk pintu samping, sensor kaca pecah, atau PIR ruang belakang. |
| **1** | **Handover (Follower)** | Jika zona *Delay-1* terpicu lebih dulu, zona ini ikut menghitung sisa waktu *Entry Delay* (memberikan waktu user berjalan menuju codepad). Jika zona ini terinjak *tanpa* ada delay sebelumnya, seketika menjadi *Instant Alarm*. |
| **2** | **Delay-1** | Pintu akses utama. Memberikan hitung mundur *Entry Delay 1* (default 20 detik) sebelum sirene berbunyi, dan *Exit Delay* saat proses arming. |
| **3** | **Delay-2** | Pintu sekunder atau garasi mobil dengan timer *Entry Delay 2* yang lebih panjang (misal 45–60 detik). |
| **9** | **24-Hour Tamper** | Aktif 24 jam terus menerus tanpa memandang status Arm/Disarm. Jika kabel dipotong/short atau switch cover terbuka, alarm sabotase berbunyi. |
| **11** | **Keyswitch** | Dihubungkan ke saklar kunci putar (*momentary* atau *latching*) untuk arm/disarm sistem dari luar ruangan. |
| **12** | **24-Hour Burglary / Panic** | Aktif 24 jam untuk tombol panik tersembunyi (*under-desk panic button* atau *money-clip sensor*). |
| **14** | **Chime Only** | Tidak memicu sirene luar saat armed; hanya membunyikan nada *ding-dong / chime* pada codepad saat sistem disarm (fungsi bel toko). |
| **15** | **Not Used** | Zona dinonaktifkan sepenuhnya. |

---

## 4. Mode Diagnostik & Analisis Gangguan (Fault Analysis Mode)

Teknisi lapangan memeriksa kesehatan sistem dengan menahan tombol `[5]` hingga berbunyi dua bip (*Two-Stage Fault Analysis*):

```
                                  [ HOLD KEY 5 (Fault Mode) ]
                                               |
                               +---------------+---------------+
                               |                               |
                     [ Zone 1 Terang ]               [ Zone 2 - 8 Terang ]
                   (Tekan [1] utk Sub-Fault)       (Tekan angka zona terkait)
                               |                               |
       +-----------------------+-------+       +---------------+---------------+
       | 1 = Low Battery               |       | 2 = RF Device Low Battery     |
       | 2 = Date & Time Reset         |       | 3 = Zone Tamper Alarm         |
       | 3 = RF Receiver Fail          |       | 4 = Sensor Watch Fault        |
       | 4 = Output 1-3 Fail (Siren)   |       | 5 = RF Sensor Missing         |
       | 5 = Telephone Line Fail       |       | 6 = Communication Fail        |
       | 7 = Power Supply / AUX Fail   |       | 7 = Output / Codepad Fail     |
       | 8 = Onboard Panel Tamper      |       | 8 = Keyfob Low Battery        |
       +-------------------------------+       +-------------------------------+
```

### Rincian Sub-Fault Kritis:
* **Fault 1.1 (Low Battery):** Kapasitas baterai drop di bawah 11.5V atau gagal uji beban dinamis.
* **Fault 1.2 (Date & Time):** Jam sistem reset akibat listrik padam total dan baterai habis.
* **Fault 1.4 (Output 1–3 Fail):** Kabel horn speaker 8/16Ω atau sirene putus/short.
* **Fault 1.5 (Telephone Line Fail):** Tegangan PSTN line hilang (< 8VDC).
* **Fault 1.7 (Power Supply Fail):** Tegangan AUX 13.8V atau jalur data bus short/overload.
* **Fault 4 (Sensor Watch Fault):** Sensor PIR tertentu tidak pernah mendeteksi gerakan selama durasi *Sensor Watch Time* (1–30 hari) saat kondisi disarm—mengindikasikan sensor terhalang lemari, tertutup cat, atau mati total.

---

## 5. Fitur Operasional & Perintah Codepad Lapangan

### 5.1 Perintah Tahan Tombol (Hold-Down Functions)
* **Hold `[4]` (Day Alarm):** Mengaktifkan/menonaktifkan mode pengawasan zona toko saat disarm.
* **Hold `[5]` (Fault Analysis):** Masuk ke menu diagnosa kerusakan.
* **Hold `[6]` (Modem Call / Remote Connect):** Memerintahkan panel menghubungi software A-Link Plus.
* **Hold `[7]` (Codepad Buzzer Tone):** Mengubah frekuensi nada buzzer codepad (50 tingkat frekuensi).
* **Hold `[8]` (Set Date & Time):** Memasukkan format tanggal dan jam `[HHMMDDMMYY]`.
* **Hold `[9]` (Quick Arm):** Arming instan mode AWAY tanpa memasukkan PIN.
* **Hold `[0]` (Horn/Siren Test):** Menguji suara sirene, bell, dan strobo selama 2 detik.

### 5.2 Alarm Darurat Dua Tombol (Codepad Panic / Dual-Key Alarms)
* **`[1]` + `[3]`:** Panic Alarm (Dapat diprogram Audible atau Silent Panic).
* **`[4]` + `[6]`:** Fire Alarm (Menghasilkan modulasi suara sirene khusus kebakaran).
* **`[7]` + `[9]`:** Medical Alarm.

### 5.3 Fitur Keamanan Khusus
1. **Codepad Duress Alarm (Sandera):**
   * Pengguna dipaksa oleh perampok untuk mematikan alarm. Pengguna menekan `[User PIN] + [9] + [#]` (misal `25809#`).
   * Panel tampak normal dan ter-disarm secara visual di layar codepad, tetapi secara diam-diam (*silent report*) mengirimkan kode pembajakan/sandera (*Duress Code*) ke pihak keamanan/CMS.
2. **Access Denied Lockout:**
   * Jika PIN salah dimasukkan berturut-turut sebanyak batas percobaan (default 3–6 kali), codepad terkunci (*lockout*) selama waktu yang ditentukan (default 2 menit) dan memicu event log *Access Denied*.
3. **Walk Test Mode (`Master Code + 7 + #`):**
   * Mode pengujian jalan bagi teknisi. Setiap sensor yang terpicu akan membunyikan 1 bip panjang di codepad dan 1 pip di horn speaker luar, memudahkan teknisi menguji sensor sendirian tanpa memekakkan telinga.
4. **Swinger Shutdown:**
   * Membatasi jumlah bunyi sirene jika sensor mengalami kerusakan berulang (maks 1–15 kali dalam 1 sesi arming).
