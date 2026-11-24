# Praktek 9 — REST API Node-RED untuk Sistem IoT

**Mata kuliah:** Pemrograman IoT (SIF1502) · **Dosen:** Ahmadi, M.Kom
**Nama:** Ahmad Febryan · **NPM:** 2410020081 · **Kelas:** 5A SI BJB

Alur: `simulasi_esp32.py` → MQTT (Mosquitto) → Node-RED (REST API + WebSocket + SQLite) → `dashboard.html`

## Isi repo
| File | Fungsi |
|---|---|
| `flows.json` | Flow Node-RED (import) |
| `dashboard.html` | Dashboard custom (HTML/CSS/JS + Chart.js) |
| `simulasi_esp32.py` | Simulator ESP32 (paho-mqtt v2) |

## Endpoint
| Method | URL | Fungsi | Auth |
|---|---|---|---|
| GET | `/api/suhu` | Data sensor terakhir + batas suhu | X-API-Key |
| GET | `/api/history` | 50 data terakhir dari SQLite | X-API-Key |
| POST | `/api/kontrol` | Kirim perintah ke ESP32 (MQTT `iot/kontrol`) | X-API-Key |
| PUT | `/api/threshold` | Set batas suhu `{"batas": 30}` | X-API-Key |
| WS | `/ws/sensor` | Push data real-time & alert | - |

## Fitur nilai tambah
1. **PUT /api/threshold** + notifikasi: saat suhu melewati batas, flow publish ke MQTT `iot/notifikasi`, kirim alert via WebSocket, dan dashboard menampilkan banner + Browser Notification.
2. **Chart.js real-time** (suhu, kelembaban, garis batas).
3. **API key** via header `X-API-Key` (default `iot-kampus-2026`, bisa diganti lewat env `API_KEY`). Tanpa key → `401`.
4. **WebSocket** (`websocket-listener` + `websocket out`): dashboard tidak polling; polling hanya fallback bila WS putus.
5. **SQLite** (`node-red-node-sqlite`): history disimpan di `iot.db`, bukan flow context.

## Cara menjalankan
```bash
# 1. Install dependensi
pip install paho-mqtt
cd ~/.node-red && npm install node-red-node-sqlite

# 2. Terminal 1: broker
mosquitto -v
# 3. Terminal 2: Node-RED  (opsional ganti key:  set API_KEY=rahasia  / export API_KEY=rahasia)
node-red
```
4. Node-RED → ☰ → **Import** → pilih `flows.json` → **Deploy**.
   (Opsional: ubah path DB di node config `sqlitedb` jadi path absolut.)
5. Salin `dashboard.html` ke folder static, mis. `C:\node-red-static\`, lalu di `settings.js` aktifkan `httpStatic: 'C:/node-red-static'`, restart Node-RED. Buka **http://localhost:1880/dashboard.html**.
6. Terminal 3: `python simulasi_esp32.py`

## Pengujian (curl)
```bash
curl -H "X-API-Key: iot-kampus-2026" http://localhost:1880/api/suhu
curl -H "X-API-Key: iot-kampus-2026" http://localhost:1880/api/history
curl http://localhost:1880/api/suhu                      # -> 401
curl -X POST http://localhost:1880/api/kontrol -H "X-API-Key: iot-kampus-2026" \
  -H "Content-Type: application/json" -d '{"device":"lamp","action":"on"}'
curl -X PUT http://localhost:1880/api/threshold -H "X-API-Key: iot-kampus-2026" \
  -H "Content-Type: application/json" -d '{"batas":28}'
```
Verifikasi: simulator mencetak `← iot/kontrol` saat tombol lampu ditekan dan `← iot/notifikasi` saat suhu melewati batas.

## Screenshot
(tambahkan: flow Node-RED, dashboard, hasil curl 200 & 401, isi tabel SQLite)
