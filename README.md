# ESP32 Nepel Stock Monitoring System

Sistem monitoring persediaan nepel berbasis ESP32, multiplexer (MUX), dan web server lokal, dengan fitur pengiriman data ke Google Sheets dan tampilan real-time status melalui halaman web. Sistem ini digunakan untuk mendeteksi apakah stok nepel tersedia (`ready`) atau habis (`runout`), dan mencatat log perubahan status serta uptime WiFi ESP32.

---

## Fitur Utama

- Monitoring hingga 4 channel sensor digital (bisa diperluas)
- Deteksi perubahan status sensor secara otomatis
- Pengiriman data ke Google Spreadsheet via WebScript
- Webserver lokal dengan tampilan status dan log
- Fitur uptime monitoring (timer berjalan jika WiFi aktif)
- Auto-refresh dashboard jika ada perubahan
- Log maksimal 10 perubahan terakhir ditampilkan di halaman web
- Auto-reconnect jika WiFi terputus

---

## Struktur Proyek

File utama:
- `main.ino`= file kode utama ESP32
- `README.md`= dokumentasi ini
- 'Appscript for Spreadsheet.json' = Appscript penghubung ke spreadsheet
- 'Dashboard monitoring stock nepel.html' = Auto redirect ke halaman webserver
---

## Tampilan Webserver

Dashboard lokal menampilkan:
- Status semua channel (warna hijau = ready, merah = runout)
- Timestamp update terakhir setiap channel
- Log perubahan terakhir
- Uptime dari koneksi WiFi

---

##  Cara Upload dan Jalankan

1. Buka file `.ino` dengan Arduino IDE
2. Pastikan library berikut sudah tersedia:
   - `WiFi.h`
   - `WebServer.h`
   - `HTTPClient.h`
   - `ArduinoJson.h`
   - `time.h`
3. Atur:
   - `ssid` dan `password` sesuai WiFi lokal
   - `scriptURL` sesuai link Google Apps Script Anda
4. Upload ke ESP32
5. Buka file 'Dashboard monitoring stock nepel.html' di browser

---

## Menambah Channel Sensor

Secara default, sistem membaca 4 channel (`totalChannels = 4`). Untuk menambah jumlah channel:

1. Ubah variabel `totalChannels`:
   ```cpp
   const int totalChannels = 8;
2. Update array namaNepel[] agar mencakup semua channel baru:

const char* namaNepel[totalChannels] = {
"Nepel 12-12", "Nepel 12-10", "Nepel 10-10", "Nepel Single 10",
"Nepel Extra 1", "Nepel Extra 2", "Nepel Extra 3", "Nepel Extra 4"
	};
3. (Opsional) Aktifkan multiplexer kedua/ketiga jika total channel melebihi 8 atau 16:

digitalWrite(muxEN1, LOW);  // Channel 0–7
digitalWrite(muxEN2, LOW);  // Channel 8–15 (MUX 2)
digitalWrite(muxEN3, LOW);  // Channel 16–23 (MUX 3)

## Mengganti Nama Channel
1. Edit isi array namaNepel[] sesuai urutan channel:

const char* namaNepel[totalChannels] = {
  "Nepel A", "Nepel B", "Nepel C", "Nepel D"
};

2. Nama ini akan digunakan untuk:

- Label log di spreadsheet

- Tampilan web dashboard

- JSON yang dikirim ke Google Sheets

## Membuka Webserver Menggunakan redirect `.html`

Agar lebih mudah mengakses IP lokal ESP32 tanpa mengetik alamat IP manual, gunakan file Redirect `Dashboard monitoring stock nepel.html`.


