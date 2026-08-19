# 🛡️ Security Alarm System ESP32 (Bosch Solution 2000/3000 Modernized Adaptation)

Dokumentasi komprehensif analisis sistem security alarm **Bosch Solution 2000 & 3000 (ICP-SOL2-P / ICP-SOL3-P)** dan cetak biru (*blueprint*) adaptasi serta modernisasinya menggunakan mikrokontroler **ESP32**.

---

## 📌 Ringkasan Proyek
Proyek ini bertujuan mengadopsi ketangguhan logika industri, *hardware supervision*, dan *finite state machine* (FSM) dari panel alarm konvensional Bosch Solution 2000/3000, lalu mengombinasikannya dengan fleksibilitas IoT modern:
* Konfigurasi visual via Web UI responsif (menghilangkan sistem kode alamat lokasi `[Location#]`).
* Multi-bearer connectivity (Ethernet W5500 + Wi-Fi + Modul Seluler 4G LTE opsional).
* Integrasi langsung dengan ekosistem Smart Home (MQTT / Home Assistant Native Discovery) dan notifikasi instan (Telegram Bot / Webhook).
* Sirkuit analog presisi tinggi untuk pembacaan resistor EOL/DEOL/Split Zone Doubling dengan proteksi industri.

---

## 📑 Struktur Dokumentasi
1. [`docs/01-bosch-system-analysis.md`](docs/01-bosch-system-analysis.md) — Bedah tuntas arsitektur hardware, kelistrikan, tipe zona, timer, dan alur kerja Bosch Solution 2000/3000.
2. [`docs/02-esp32-architecture-blueprint.md`](docs/02-esp32-architecture-blueprint.md) — Rencana arsitektur hardware, skema analog front-end, power management, dan firmware ESP32.
3. [`docs/03-feature-roadmap-and-status.md`](docs/03-feature-roadmap-and-status.md) — Pemetaan fitur: Fitur Bosch Asli vs Rencana Adopsi ESP32 vs Fitur Modernisasi Baru (Status Pending / In-Review).

---

## 🗂️ Matriks Perbandingan Ringkas

| Fitur / Parameter | Bosch Solution 2000 / 3000 | Rencana ESP32 Alarm Controller |
| :--- | :--- | :--- |
| **Kapasitas Zona** | 4-8 onboard (hingga 8-16 dg Zone Doubling) | 8–16 onboard via High-Precision ADC (ADS1115 + MUX) |
| **Topologi Resistor** | Single EOL (3.3k), DEOL Tamper, Split (3.3k/6.8k) | Single EOL, DEOL, Split EOL (Configurable via Web UI) |
| **User Interface** | Icon / Text Codepad (SDI2 Bus 4-kabel) | Smart Keypad (RS-485 / I2C) + Web UI + Mobile App |
| **Metode Konfigurasi** | Alamat Lokasi Hex/Decimal via Keypad / A-Link Plus | Embedded Web Dashboard (Zero-install, visual, live graph) |
| **Konektivitas** | PSTN Onboard, Modul IP (B426-M), Modul 4G (B450-M) | Onboard Ethernet (W5500) + Wi-Fi + Modul LTE (A7670/SIM7600) |
| **Smart Home Integration** | Terbatas / Proprietary RSC+ App | Native MQTT (Home Assistant Auto-Discovery), Webhooks |
| **Notifikasi Alarm** | Dial-up Contact ID / SIA DC-09 / Push RSC+ | Telegram Bot Instan, SMS/Call (Modem 4G), SIA DC-09 IP |
| **Penyimpanan Log** | ~256 - 512 Event Log di EEPROM | > 10,000 Event Log di NVS / LittleFS Flash / MicroSD |

---

*Status Proyek: **Brainstorming & Technical Review Phase**.*
