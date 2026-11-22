# REST API Node-RED untuk Sistem IoT

Sistem monitoring dan kontrol IoT: simulator ESP32 mengirim data suhu dan kelembaban lewat MQTT, Node-RED bertindak sebagai backend REST API dan WebSocket, riwayat data disimpan di SQLite, dan dashboard web (HTML/CSS/JS) menampilkan data secara real-time.

## Arsitektur

```
simulasi_esp32.py ──MQTT (iot/sensor)──► Node-RED ──► SQLite (sensor_history)
                                            │
                                            ├──► REST API  (/api/suhu, /api/history, /api/kontrol, /api/threshold)
                                            ├──► WebSocket (/ws/sensor) ──► dashboard.html (Chart.js)
                                            ├──► MQTT iot/alert   (notifikasi suhu melewati batas)
                                            └──► MQTT iot/kontrol (perintah ke perangkat)
```

## Tugas Pengembangan (Nilai Tambah)

| No | Tugas | Implementasi |
|----|-------|--------------|
| 1 | Endpoint `PUT /api/threshold` + notifikasi suhu melewati batas | Flow: endpoint, peringatan di debug, publish MQTT `iot/alert`. Dashboard: kolom batas suhu dan banner peringatan. |
| 2 | Grafik real-time Chart.js | `dashboard.html`: grafik suhu, kelembaban, dan garis batas suhu. |
| 3 | Autentikasi API key (`X-API-Key`) | Dicek pada `PUT /api/threshold` dan `POST /api/kontrol`. |
| 4 | WebSocket untuk update tanpa polling | `ws://localhost:1880/ws/sensor`, dashboard tidak memakai polling. |
| 5 | History ke database, bukan flow context | SQLite, tabel `sensor_history`. `GET /api/history` membaca dari database. |

## Struktur Repositori

```
├── README.md
├── flows.json            # Flow Node-RED (export)
├── dashboard.html        # Dashboard web
├── simulasi_esp32.py     # Simulator ESP32 (publisher MQTT)
└── screenshots/          # Bukti pengujian
```

## Prasyarat

- Node-RED (`localhost:1880`) dengan node tambahan `node-red-node-sqlite`
- Mosquitto MQTT broker (`localhost:1883`)
- Python 3 dan `pip install paho-mqtt` (versi 2.x)
- Browser modern (dashboard memuat Chart.js dari CDN, perlu koneksi internet)

## Cara Menjalankan

1. Jalankan broker: `mosquitto -v`
2. Jalankan Node-RED: `node-red`
3. Import `flows.json` lewat menu ☰ → Import, lalu **Deploy**.
4. Buka konfigurasi node SQLite dan sesuaikan **path database** (`sensor.db`) dengan komputer Anda.
5. Jalankan simulator: `python simulasi_esp32.py`
6. Buka `dashboard.html` di browser.

Tabel `sensor_history` dibuat otomatis saat Node-RED start. Inject **Bersihkan data NULL** dan **Uji SELECT Database** tersedia untuk pengecekan database.

## Endpoint

| Method | Endpoint | API Key | Fungsi |
|--------|----------|---------|--------|
| GET | `/api/suhu` | Tidak | Data sensor terakhir (termasuk `alert` dan `threshold`) |
| GET | `/api/history` | Tidak | 50 data sensor terakhir dari SQLite |
| POST | `/api/kontrol` | Ya | Kirim perintah ke perangkat (MQTT `iot/kontrol`) |
| PUT | `/api/threshold` | Ya | Atur batas suhu |
| OPTIONS | `/api/*` | Tidak | CORS preflight |

API key dikirim lewat header `X-API-Key`. Key contoh pada proyek ini: `iot-rahasia-2026` (hanya untuk keperluan tugas).

### Contoh Request

```bash
curl http://localhost:1880/api/suhu

curl -X POST http://localhost:1880/api/kontrol \
  -H "Content-Type: application/json" -H "X-API-Key: iot-rahasia-2026" \
  -d '{"device":"lamp","action":"on"}'

curl -X PUT http://localhost:1880/api/threshold \
  -H "Content-Type: application/json" -H "X-API-Key: iot-rahasia-2026" \
  -d '{"threshold": 35}'
```

Tanpa header key, kedua request di atas mengembalikan status `401`. Nilai threshold yang tidak valid mengembalikan `400`.

## Screenshot

**Flow Node-RED**

![Flow Node-RED](screenshots/01-flow-nodered.png)

**Dashboard**

![Dashboard](screenshots/02-dashboard.png)

**Dashboard saat suhu melewati batas**

![Dashboard peringatan](screenshots/03-dashboard-alert.png)

**GET /api/suhu dan /api/history**

![GET suhu](screenshots/04-get-suhu.png)
![GET history](screenshots/05-get-history.png)

**Autentikasi API key (POST /api/kontrol)**

![Kontrol 401](screenshots/06-kontrol-401.png)
![Kontrol 200](screenshots/07-kontrol-200.png)

**PUT /api/threshold**

![Threshold 200](screenshots/08-threshold-200.png)
![Threshold 400](screenshots/09-threshold-400.png)

**Database SQLite**

![Debug SQLite](screenshots/10-debug-sqlite.png)

**Notifikasi dan perintah kontrol via MQTT**

![MQTT alert](screenshots/11-mqtt-alert.png)
![MQTT kontrol](screenshots/12-mqtt-kontrol.png)
