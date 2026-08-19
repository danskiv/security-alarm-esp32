# 🗺️ Peta Fitur & Status Adopsi (Roadmap & Feature Mapping)

Dokumen ini memetakan seluruh fitur inti **Bosch Solution 2000 (ICP-SOL2-P)**, perbandingan dengan arsitektur **ESP32 Security Alarm Controller**, serta status tinjauan teknis.

---

## 📊 Matriks Pemetaan Fitur Komprehensif

| No | Modul / Kategori Fitur | Fitur Asli Bosch Solution 2000 | Rencana Adopsi pada ESP32 | Status Adopsi | Catatan & Analisis Teknisi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Zona Fisik & EOL** | 4 terminal fisik (maks 8 zona via Split EOL 3.3k/6.8k). Mendukung SEOL 3.3k dan DEOL Tamper 4-state. | 8–16 Onboard Zone via 16-bit ADC (ADS1115) + MUX (CD74HC4067). Support SEOL, DEOL Tamper, Split ZD. | 🟡 **Pending Review** | Resistor calibration via Web UI; toleransi resistansi dapat disesuaikan otomatis terhadap penurunan kualitas kabel. |
| **2** | **Tipe Zona (Zone Types)** | Instant (0), Handover (1), Delay-1 (2), Delay-2 (3), 24h Tamper (9), Keyswitch (11), 24h Panic (12), Chime (14), Not Used (15). | Mengadopsi ke-9 tipe zona standar ke dalam FreeRTOS FSM ESP32. | 🟡 **Pending Review** | Ditambah tipe zona lingkungan (*Gas, Water Leak, Temperature Sensor*). |
| **3** | **Anti-False Alarm & Swinger** | Zone Pulse Count (1–15 pulsa) & Swinger Shutdown Count (Siren & Reporting). | Moving-average digital filter + Swinger Auto-Mute software counter. | 🟡 **Pending Review** | Mencegah sirene tetangga terganggu akibat ranting pohon atau kucing yang melintas berulang. |
| **4** | **Mode Operasi Arming** | Away Arm, Stay 1, Stay 2, Instant Stay, Entry Delay (20-30s), Exit Delay (60s), Forced Arming. | Deterministic FSM state machine dengan timer non-blocking dan partisi ganda (Area 1 & Area 2). | 🟡 **Pending Review** | State dipertahankan dalam Non-Volatile Storage (NVS); tahan terhadap reboot atau restart mendadak. |
| **5** | **Supervisi Kelistrikan** | AC Fail Monitoring (Timer delay), Dynamic Battery Load Test (tiap 4 jam & arming), Low Batt (<11.5V), Auto Cut-Off (<10.2V). | Optocoupler AC-loss detector, MOSFET load resistor tester, ADC battery monitoring, relay auto-disconnect. | 🟡 **Pending Review** | Menjaga sel baterai SLA 12V dari kerusakan permanen akibat *deep discharge*. |
| **6** | **Supervisi Sirene (Output 1)** | Horn Speaker 8/16Ω atau DC Siren dengan *coil supervision loop* (deteksi kabel putus/short). | Shunt resistor + precision op-amp current sense untuk verifikasi loop beban sirene saat siaga (*standby*). | 🟡 **Pending Review** | Memberi peringatan sebelum pembobolan terjadi jika kabel sirene luar sengaja diputus pelaku. |
| **7** | **Fault Analysis Mode** | 2-Stage Fault Analysis (Hold `[5]`): Battery, Time, RF, Output 1-3, Line, AUX Power, Tamper, Sensor Watch, Comm Fail. | Diagnostic Engine terpadu dengan reporting via Web Dashboard & push notification Telegram. | 🟡 **Pending Review** | Menghilangkan kode angka biner/lampu kedip; semua status gangguan langsung muncul dalam bahasa manusia. |
| **8** | **Sensor Watch (Anti-Blind)** | Mengawasi jika sensor gerak tidak pernah terpicu selama durasi tertentu (1–30 hari) saat disarm. | Timer pemantauan aktivitas zona di firmware; mengirim alert pemeliharaan (*maintenance alert*). | 🟡 **Pending Review** | Mendeteksi jika sensor tertutup perabotan, terhalang debu, atau rusak sebelum sistem di-arm. |
| **9** | **Walk Test Mode** | `Master Code + [7] + [#]`: Bip pendek di codepad & horn speaker saat zona diinjak tanpa memicu alarm penuh. | Mode Walk Test khusus dengan umpan balik visual *real-time* di layar HP/Keypad dan nada audio pendek. | 🟡 **Pending Review** | Memudahkan teknisi menguji seluruh sensor di lokasi seorang diri (*single-man test*). |
| **10** | **Codepad Alarms & Duress** | Sandera PIN `[User Code]+9+#`, Dual-Key Emergency (`1+3` Panic, `4+6` Fire, `7+9` Medical), PIN Lockout. | Mendukung Duress Code, PIN lock out, dan software-triggered emergency button di UI/Keypad. | 🟡 **Pending Review** | Duress PIN tetap mendisarm sirene lokal namun mengirim sinyal SOS rahasia berprioritas tinggi. |
| **11** | **Day Alarm / Chime Mode** | Hold `[4]` untuk mengaktifkan bel zona toko/resepsionis saat siang hari tanpa membunyikan sirene utama. | Chime mode via buzzer onboard / smart speaker (Home Assistant) saat pintu masuk terbuka di siang hari. | 🟡 **Pending Review** | Sangat berguna untuk ruko, toko retail, atau kantor pelayanan. |
| **12** | **Metode Pemrograman** | Alamat Lokasi Hex/Decimal (`[Location#] [Value*]`) via Icon/Text Codepad atau kabel DLA / A-Link Plus. | **Embedded Web Server (Local Web Dashboard)** interaktif dengan konfigurasi visual instan. | 🟢 **Modernisasi Baru** | Menghilangkan kebutuhan menghafal manual buku tebal; konfigurasi via Wi-Fi/Ethernet langsung dari HP. |
| **13** | **Konektivitas Jaringan** | PSTN Onboard, Modul B426-M (Ethernet IP), Modul B450-M (4G LTE). | **Ethernet W5500 Onboard + Wi-Fi 2.4GHz + Modul LTE UART (SIM7600/A7670)**. | 🟢 **Modernisasi Baru** | Redundansi tiga lapis (*Triple-Bearer Failover*): Ethernet -> Wi-Fi -> 4G LTE. |
| **14** | **Smart Home & Integrasi IoT** | Proprietary Bosch Cloud / RSC+ App berbayar/terbatas. | **Native MQTT Protocol (Home Assistant Alarm Control Panel Discovery)**. | 🟢 **Modernisasi Baru** | Integrasi penuh ke automasi rumah pintar, lampu otomatis saat alarm, dan monitoring multi-kamera. |
| **15** | **Notifikasi Insiden** | Dial-up Contact ID / SIA DC-09 ke CMS Komersial. | **Telegram Bot Instant Push + Webhooks + SMS/Voice Call (via 4G) + SIA DC-09 IP**. | 🟢 **Modernisasi Baru** | Pemilik rumah menerima notifikasi instan di Telegram dalam hitungan detik saat ada insiden. |
| **16** | **Kapasitas Event Log** | 256 kejadian tersimpan di EEPROM. | **> 10,000 Event Log di Flash LittleFS / MicroSD** lengkap dengan NTP timestamp. | 🟢 **Modernisasi Baru** | Riwayat log dapat diekspor langsung ke format CSV / JSON untuk keperluan investigasi keamanan. |

---

## 📌 Penjelasan Kategori Status
* 🟡 **Pending Review:** Fitur inti Bosch Solution 2000 yang telah dibedah secara tuntas (menunggu Yang Mulia menelaah dan memutuskan prioritas adopsinya).
* 🟢 **Modernisasi Baru:** Peningkatan teknologi mutakhir berbasis ESP32 yang mengatasi batasan sistem konvensional.
* ⏳ **Fase Mendatang:** Fitur nirkabel (*wireless RF 433MHz / Zigbee*) yang dialokasikan setelah arsitektur *hardwired* utama rampung.
