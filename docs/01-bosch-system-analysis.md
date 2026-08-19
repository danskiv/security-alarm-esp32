# 📘 Analisis Mendalam Sistem Bosch Solution 2000 & 3000

Dokumen ini membedah seluruh komponen, karakteristik elektrik, logika *state machine*, dan sistem konfigurasi dari panel alarm komersial **Bosch Solution 2000 (ICP-SOL2-P)** dan **Bosch Solution 3000 (ICP-SOL3-P)**.

---

## 1. Arsitektur Kelistrikan & Power Management

### 1.1 Input Daya Utama (AC Power)
* **Spesifikasi:** 16–18 VAC, 50/60 Hz disuplai melalui transformator plug-pack eksternal (**TF008-B / 1.5A**).
* **Deteksi Gangguan Listrik (AC Fail):**
  * Sirkuit *Zero-crossing / Rectifier Voltage Monitor* membaca ada/tidaknya tegangan AC.
  * Dilengkapi *programmable AC Fail Delay* (1–60 menit) untuk mencegah pengiriman sinyal kepanikan ke Central Monitoring Station saat listrik padam sesaat (*power flicker*).

### 1.2 Catu Daya Cadangan (Battery Backup)
* **Spesifikasi:** 12V 7.0 Ah / 1.2 Ah *Sealed Lead Acid* (SLA) Rechargeable Battery.
* **Charger Circuit:** *Constant Voltage / Trickle Charge* pada ~13.8 VDC.
* **Dynamic Battery Load Test:** Panel secara berkala menarik arus beban buatan selama beberapa milidetik untuk mengukur resistansi internal baterai, bukan sekadar membaca voltase mengambang (*float voltage*).
* **Supervisi Voltase:**
  * **Low Battery Alarm:** Memicu sinyal *Trouble* jika voltase < 11.5 VDC.
  * **Low Voltage Disconnect (Deep Discharge Protection):** Sirkuit *auto cut-off* memutus baterai saat voltase menyentuh ~10.2–10.5 VDC agar sel baterai tidak mengalami desulfasi permanen atau melembung.

### 1.3 Catu Daya Aksesori (AUX Power)
* **Spesifikasi:** 13.8 VDC regulated.
* **Proteksi:** Dilindungi sekring PTC (*Positive Temperature Coefficient* resettable fuse) dengan batas kontinu ~1.0A untuk menyuplai sensor PIR, detektor asap, codepad, dan modul ekspansi.

---

## 2. Sirkuit Pembacaan Zona (Zone Loops & Topologi EOL)

### 2.1 Kapasitas Fisik
* **Solution 2000:** 4 terminal fisik onboard (dapat dimaksimalkan menjadi 8 zona via *Zone Doubling*).
* **Solution 3000:** 8 terminal fisik onboard (dapat dimaksimalkan menjadi 16 zona via *Zone Doubling* atau ekspander B228).

### 2.2 Skema Topologi Resistor EOL (End-of-Line)

```
1. Single EOL (SEOL - 3.3kΩ):
   [Terminal Z+] ---- [NC Sensor Switch] ---- [3.3kΩ Resistor] ---- [Terminal COM]
   - Loop Resistance:
     * ~3.3 kΩ  : SEALED (Normal / Aman)
     * ~0 Ω / Open : UNSEALED (Alarm Terpicu)

2. Dual EOL (DEOL - Anti-Sabotage Line Supervision):
   [Terminal Z+] ---- [Tamper NC] ---- [Alarm NC + 3.3kΩ Parallel] ---- [3.3kΩ Series] ---- [Terminal COM]
   - Loop Resistance:
     * 0 Ω        : TAMPER SHORT (Kabel sengaja dikonsletkan)
     * ~3.3 kΩ    : SEALED (Normal / Aman)
     * ~6.6 kΩ    : ALARM (Sensor PIR/Door terpicu)
     * Open (∞)   : TAMPER OPEN / CUT (Kabel dipotong oleh penyusup)

3. Split EOL / Zone Doubling (ZD - 3.3kΩ & 6.8kΩ):
   Menggabungkan dua zona independen (misal Zone 1 dan Zone 9) pada 1 pasang terminal fisik:
   - Low Zone (Z1) memakai resistor R1 = 3.3 kΩ
   - High Zone (Z9) memakai resistor R2 = 6.8 kΩ
   - Kombinasi Nilai Resistansi:
     * Keduanya Sealed (Normal)        : ~2.22 kΩ (R1 || R2)
     * Low Zone Alarm, High Sealed     : 6.8 kΩ
     * High Zone Alarm, Low Sealed     : 3.3 kΩ
     * Keduanya Alarm                  : Open Circuit (Tak Hingga)
```

### 2.3 Filter Sinyal & Anti-False Alarm
* **Loop Response Time:** Waktu stabilisasi sinyal dapat diatur antara 20 ms hingga 500 ms (mencegah *noise* induksi kabel panjang).
* **Zone Pulse Count:** Fitur perhitungan pulsa (1–15 kali trigger dalam jendela waktu tertentu) sebelum alarm dinyatakan valid—sangat berguna untuk sensor seismik/getar dan PIR luar ruangan.

---

## 3. Output Sirkuit & Driver Sirene

### 3.1 Output 1 — Supervised Siren / Bell Output
* **Tipe:** Direct Horn Speaker Driver (8Ω / 16Ω) atau 12VDC Electronic Siren.
* **Supervision:** Sirkuit memonitor keberadaan resistansi kumparan speaker secara kontinu. Jika kabel sirene luar dipotong atau dikonsletkan, sistem seketika memicu status *System Trouble*.

### 3.2 Output 2 — Strobe Output
* **Tipe:** 12VDC Switched Open-Collector / Relay output.
* **Karakteristik:** *Latching output* yang menyalakan lampu strobo visual hingga sistem di-disarm secara sah dengan PIN.

### 3.3 Output 3 & Output 4 — Programmable Outputs (PGM)
* **Tipe:** Open-Collector Pulldown (100–500mA rating).
* **Fungsi:** Trigger relay eksternal, reset daya 4-wire Smoke Detector, status arming LED, atau integrasi ke gerbang otomatis.

---

## 4. Bus Peripheral & Ekosistem Modul (SDI2 Bus)

* **Protokol:** Proprietary 4-Wire Bus (`+12V`, `GND`, `CLK`, `DATA`).
* **Dukungan Perangkat:**
  * **IUI-SOL-ICON:** Keypad LCD Icon monokrom.
  * **IUI-SOL-TEXT:** Keypad LCD Alfanumerik teks 2-baris.
  * **IUI-SOL-TS5 / TS7:** Layar sentuh grafis 5" & 7".
  * **B308 Octo-output Module:** Modul ekspansi 8 relay output.
  * **B810 RADION Receiver:** Modul penerima wireless 433.42 MHz (mendukung remote keyfob HCT-4, PIR nirkabel, sensor magnetik nirkabel).
  * **B426-M / B450-M:** Modul komunikasi jaringan Ethernet IP dan 4G LTE.

---

## 5. Logika State Machine, Tipe Zona, & Partisi

### 5.1 Mode Operasi Panel
1. **DISARMED:** Sistem tidak aktif kecuali untuk zona 24-Jam (*Tamper*, *Fire*, *Panic*).
2. **AWAY ARMED:** Seluruh perimeter dan sensor gerak interior aktif.
3. **STAY MODE 1 & 2:** Hanya sensor perimeter (pintu/jendela) yang aktif, sensor interior di-*bypass* otomatis.
4. **ENTRY DELAY:** Waktu hitung mundur (default 20–30 detik) saat pintu masuk terbuka sebelum sirene meledak.
5. **EXIT DELAY:** Waktu hitung mundur (default 60 detik) bagi pengguna untuk meninggalkan lokasi setelah menekan tombol arm.

### 5.2 Matriks Tipe Zona Standar Bosch
* `0 = Instant`: Seketika memicu alarm saat armed (kaca pecah, pintu samping).
* `1 = Handover (Follower)`: Mengikuti *Entry Delay* jika pintu delay terbuka lebih dulu; menjadi *Instant* jika sensor ini terinjak pertama kali.
* `2 = Delay-1`: Pintu akses utama dengan Entry Delay 1.
* `3 = Delay-2`: Pintu akses sekunder/garasi dengan Entry Delay 2 yang lebih panjang.
* `9 = 24-Hour Tamper`: Aktif 24 jam untuk mendeteksi sabotase fisik housing / kabel.
* `11 = Keyswitch`: Saklar kunci putar untuk arm/disarm.
* `12 = 24-Hour Burglary`: Tombol panik / darurat kasir.
* `14 = Chime Only`: Hanya membunyikan nada lonceng saat disarmed (doorbell mode).
* `15 = Not Used`: Zona nonaktif.

### 5.3 Partisi (Areas)
Mendukung **2 Area Mandiri** (Area 1 dan Area 2), memungkinkan dua lantai atau dua unit ruko menggunakan satu panel sentral yang sama dengan kode otorisasi terpisah.
