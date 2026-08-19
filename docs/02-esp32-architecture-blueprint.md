# 📐 Cetak Biru Arsitektur ESP32 Security Alarm Controller

Dokumen ini merinci rancangan arsitektur hardware, skema analog front-end, manajemen catu daya, dan struktur firmware berbasis mikrokontroler **ESP32** (ESP32-WROOM-32E atau ESP32-S3).

---

## 1. Arsitektur Blok Diagram Sistem

```
                                    +-----------------------+
                                    |   220VAC Mains Grid   |
                                    +-----------------------+
                                                |
                                                v
   +--------------------+           +-----------------------+
   | 12V 7Ah SLA Batt   |<--------->| Intelligent UPS &     |
   | (Deep Discharge &  |           | Battery Charger PMIC  |
   | Dynamic Test Mosfet|           +-----------------------+
   +--------------------+                       |
                                                v (+13.8V DC Aux / +5V / +3.3V)
+-----------------------------------------------------------------------------------------+
|                                    ESP32 MAINBOARD                                      |
|                                                                                         |
|  +--------------------+    +--------------------+    +-------------------------------+  |
|  | 8/16 Zone Terminal |    | Precision Analog   |    | ESP32-S3 Core                 |  |
|  | - TVS Diode Clamps |--->| Front-End          |--->| - FreeRTOS Multitasking       |  |
|  | - RC Noise Filter  |    | - ADS1115 (16-bit) |    | - Deterministic Security FSM  |  |
|  | - Voltage Divider  |    | - 16-Ch MUX (4067) |    | - NVS Storage & Event Logger  |  |
|  +--------------------+    +--------------------+    +-------------------------------+  |
|                                                              |                          |
|  +--------------------+    +--------------------+            |                          |
|  | Supervised Outputs |    | Isolated PGM Out   |            |                          |
|  | - Siren Driver     |<---| - Solid-State /    |<-----------+                          |
|  | - Shunt Sense Loop |    |   Opto-Relay Out   |            |                          |
|  +--------------------+    +--------------------+            |                          |
|                                                              v                          |
|  +-----------------------------------------------------------------------------------+  |
|  |                            Communication Subsystems                               |  |
|  | - Native Ethernet (W5500 via SPI)                                                 |  |
|  | - Wi-Fi 802.11 b/g/n (2.4GHz)                                                     |  |
|  | - RS-485 Industrial Peripheral Bus (MAX485 / ISO3082 for Smart Keypad)          |  |
|  | - 4G LTE Cellular Module (UART: SIM7600E / A7670E)                                |  |
|  +-----------------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------------+
                               |              |              |
                               v              v              v
                       [Home Assistant]  [Telegram Bot]  [Local Web UI]
                         (MQTT Broker)    (Direct HTTPS)   (Responsive)
```

---

## 2. Rincian Desain Hardware & Sirkuit

### 2.1 Analog Front-End (Pembacaan Resistor Zona)
* **Masalah Internal ADC ESP32:** ADC internal ESP32 memiliki non-linearitas parah pada tegangan < 0.1V dan > 3.1V, memiliki *thermal noise*, dan ADC2 tidak dapat berfungsi jika Wi-Fi aktif.
* **Solusi Hardware:**
  * Menggunakan **ADS1115 (16-bit Precision I2C ADC)**.
  * Dipadukan dengan multiplexer analog **CD74HC4067 (16-Channel)** untuk memindai 8 hingga 16 zona secara presisi.
  * Rangkaian *Voltage Divider* presisi menggunakan resistor metal film toleransi 0.1% / 1%.
* **Proteksi Sirkuit Input:**
  * Setiap jalur terminal zona dilindungi oleh **TVS Diode SMAJ5.0CA** (menahan lonjakan listrik statis dan induksi petir).
  * Filter **RC Low-Pass (100Ω + 100nF)** untuk meredam *high-frequency electrical noise*.
  * Dioda Schottky pengaman rail (*clamping diodes*) ke 3.3V dan GND.

### 2.2 Supervised Output Driver (Sirene & Strobo)
* **Driver:** N-Channel Power MOSFET (misal IRLZ44N / AOD4184A) dengan *low Rds(on)*.
* **Sirkuit Supervisi Kabel:**
  * Melewatkan arus uji sangat kecil (beberapa mikroampere) melalui resistor pull-up tinggi saat sirene mati.
  * Pembacaan tegangan pada titik beban memverifikasi apakah kumparan sirene masih terhubung (*closed*), putus (*open*), atau korslet (*short* ke ground).

### 2.3 Power Management & Battery Charger
* Input 15–18 VDC / VAC diperbaiki (*rectified*) dan diturunkan dengan switching buck regulator berefisiensi tinggi (**LM2596 / XL4015** atau controller UPS otomatis).
* Deteksi kehilangan daya utama menggunakan optocoupler (**PC817**) yang memicu *hardware interrupt* pada ESP32.
* Rangkaian uji beban baterai menggunakan MOSFET dan resistor daya 10Ω 5W untuk *dynamic load testing*.
* Sirkuit pemutus tegangan rendah (*Low Voltage Cutoff*) menggunakan komparator atau relay latching untuk melindungi baterai dari *deep discharge* (< 10.5V).

### 2.4 Bus Komunikasi Periferal (RS-485)
* Menggantikan proprietary SDI2 bus dengan standar terbuka **RS-485 (Half-Duplex)** menggunakan IC transceiver terisolasi (**MAX485 / ADM2483**).
* Menghubungkan unit panel utama ke satu atau beberapa *Smart Keypad* pada jarak hingga 500 meter kabel STP (*Shielded Twisted Pair*).

---

## 3. Desain Arsitektur Firmware (C++ / ESP-IDF / FreeRTOS)

Firmware dirancang berbasis *deterministic tasks* untuk menjamin respon seketika terhadap insiden keamanan:

1. **`ZoneScannerTask` (Prioritas Tinggi - Core 1):**
   * Berjalan setiap 20 ms.
   * Melakukan oversampling pembacaan ADC pada 16 channel.
   * Menghitung status zona (Sealed, Alarm, Tamper Short, Tamper Open).
   * Menangani *Pulse Count* dan *Debounce Filter*.

2. **`SecurityFSMTask` (Prioritas Kritis - Core 1):**
   * Mengelola *Finite State Machine* (Disarm, Away, Stay 1, Stay 2, Entry Delay, Exit Delay, Alarm).
   * Memproses transisi status, sirene trigger, dan batas *Swinger Shutdown*.

3. **`CommsTask` (Prioritas Sedang - Core 0):**
   * Mengelola koneksi jaringan (Ethernet W5500 & Wi-Fi failover).
   * Mengirim payload status via **MQTT (Home Assistant MQTT Alarm Control Panel)**.
   * Mengirim notifikasi darurat via **Telegram Bot API (HTTPS)**.
   * Mengirim SMS / Panggilan darurat via modul GSM (SIM7600).

4. **`WebServerTask` (Prioritas Rendah - Core 0):**
   * Menyajikan antarmuka Web UI lokal untuk konfigurasi teknisi (tanpa kode alamat angka).
   * Menampilkan grafik resistansi *real-time* setiap zona untuk mempermudah diagnosa kabel di lapangan.

5. **`StorageManager`:**
   * Menyimpan status partisi, bypass flag, dan counter ke **NVS (Non-Volatile Storage)**.
   * Menyimpan histori kejadian (*Event Log*) berformat JSON / SQLite ringan di partisi SPIFFS/LittleFS atau MicroSD.
