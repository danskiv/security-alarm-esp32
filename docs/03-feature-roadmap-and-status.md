# 🗺️ Peta Fitur & Status Adopsi (Roadmap & Feature Mapping)

Dokumen ini memetakan fitur asli **Bosch Solution 2000/3000**, rencana adopsi ke **ESP32 Security Alarm Controller**, serta status implementasi saat ini.

---

## 📊 Matriks Pemetaan Fitur & Status

| No | Modul / Kategori Fitur | Fitur Asli Bosch Solution 2000/3000 | Rencana Adopsi pada ESP32 | Status Adopsi | Catatan Teknis / Rekomendasi |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Zona & Resistor EOL** | 4–8 Onboard Zone (Maks 16 via Split EOL / Zone Doubling). Nilai default 3.3kΩ. | 8–16 Onboard Zone via 16-bit ADC (ADS1115) + MUX CD74HC4067. Support SEOL, DEOL Tamper, Split ZD. | 🟡 **Pending Review** | Nilai resistor fleksibel (bisa dikalibrasi via Web UI tanpa ganti resistor fisik jika retrofit kabel lama). |
| **2** | **Tipe Zona (Zone Types)** | Instant (0), Handover (1), Delay-1 (2), Delay-2 (3), 24h Tamper (9), Keyswitch (11), 24h Panic (12), Chime (14), Not Used (15). | Mengadopsi seluruh 9 tipe zona inti Bosch ke dalam FSM C++ ESP32. | 🟡 **Pending Review** | Ditambah tipe zona modern: *Environmental Zone* (Gas/Banjir/Suhu) dengan threshold analog. |
| **3** | **Anti-False Alarm** | Zone Pulse Count (1–15 pulsa) & Swinger Shutdown Count (Siren & Report). | Algoritma *Moving Average Filter*, *Pulse Count Window*, & *Swinger Auto-Mute*. | 🟡 **Pending Review** | Memastikan sensor PIR murah di luar ruangan tidak memicu alarm palsu berulang. |
| **4** | **Mode Arming & Delay** | Away Arm, Stay Arm 1, Stay Arm 2, Entry Delay Timer (20-30s), Exit Delay Timer (60s). | Full deterministic FSM di FreeRTOS, timer non-blocking, multi-partisi (2 Partisi). | 🟡 **Pending Review** | State tersimpan di NVS; jika listrik padam total lalu hidup, sistem kembali ke state terakhir (Armed/Disarmed). |
| **5** | **Supervisi Kelistrikan** | AC Fail Monitoring (Timer delay), Dynamic Battery Load Test, Low Battery Alarm (<11.5V), Auto Cut-Off (<10.2V). | Sirkuit Optocoupler AC-Sense, MOSFET Load Resistor Test, ADC Voltage Divider, Relay Cutoff. | 🟡 **Pending Review** | Melindungi baterai SLA dari *deep discharge* dan memberi alert sebelum baterai benar-benar soak. |
| **6** | **Supervisi Sirene** | Output 1 Supervised (Mendeteksi kabel putus / short ke ground). | Sense resistor shunt + Op-amp / ADC monitor pada jalur ground sirene. | 🟡 **Pending Review** | Memberi peringatan jika kabel sirine luar diputus oleh maling sebelum membobol rumah. |
| **7** | **Peripheral Bus (Keypad)** | SDI2 Proprietary 4-Wire Bus (Icon Keypad, Text Keypad, Touchscreen TS5/TS7). | Menggunakan bus industri standar **RS-485 (Half-Duplex)**. | 🟡 **Pending Review** | Bisa dipasangkan dengan Smart Keypad ESP32 (TFT Touchscreen / Matrix Keypad) atau LCD 20x4 alfanumerik. |
| **8** | **Metode Pemrograman** | Alamat Lokasi Hexadecimal/Decimal via Keypad (`[Location#] [Value*]`) atau software A-Link Plus. | **Embedded Web Server (Local Web UI)** dengan dashboard interaktif dan wizard setup. | 🟢 **Modernisasi Baru** | Menghilangkan kerumitan buku manual tipis di lapangan. Teknisi cukup membuka IP panel via browser HP/Laptop. |
| **9** | **Konektivitas Jaringan** | PSTN Onboard, Modul B426-M (Ethernet IP), Modul B450-M (Cellular 2G/4G). | **Ethernet W5500 Onboard + Wi-Fi 2.4GHz + Modul 4G LTE UART (A7670E / SIM7600E)**. | 🟢 **Modernisasi Baru** | Konektivitas redundan: Prioritas Ethernet -> Wi-Fi -> 4G LTE Seluler. |
| **10** | **Integrasi Smart Home** | Sangat terbatas / Proprietary Bosch Cloud RSC+. | **MQTT Native Integration (Home Assistant Alarm Control Panel Discovery)**. | 🟢 **Modernisasi Baru** | Sensor zona diekspos sebagai entitas `binary_sensor` di Home Assistant secara otomatis. |
| **11** | **Notifikasi Insiden** | Dial-up Contact ID / SIA DC-09 IP format ke Central Monitoring Station. | **Telegram Bot Instant Push + Webhooks + SMS/Call Alert (via 4G) + SIA DC-09 IP**. | 🟢 **Modernisasi Baru** | Pemilik rumah menerima notifikasi instan di Telegram lengkap dengan nama zona yang terpicu. |
| **12** | **Event Logging** | 256–512 riwayat kejadian di EEPROM. | **> 10,000 Event Log di Flash LittleFS / MicroSD card** lengkap dengan timestamp NTP/RTC. | 🟢 **Modernisasi Baru** | Log dapat diunduh dalam format CSV / JSON langsung dari Web UI. |
| **13** | **Wireless Sensors** | Modul RADION B810 (433.42 MHz) untuk Keyfob HCT-4 dan Wireless PIR. | Modul RF Receiver 433MHz / CC1101 atau Zigbee SoC eksternal. | ⏳ **Fase Mendatang** | Diusulkan setelah arsitektur *hardwired* utama stabil. |

---

## 📌 Penjelasan Status:
* 🟢 **Modernisasi Baru:** Fitur peningkatan modern yang melompati batasan Bosch konvensional.
* 🟡 **Pending Review:** Fitur inti Bosch yang telah dianalisis detail dan siap diadopsi ke dalam rancangan ESP32, menunggu konfirmasi/evaluasi teknis dari Yang Mulia Danas.
* ⏳ **Fase Mendatang:** Fitur sekunder yang dialokasikan untuk tahap pengembangan lanjutan.
