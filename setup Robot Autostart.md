# Setup Autostart Robot Mobil di Raspberry Pi

Panduan ini menjelaskan cara membuat script `main.py` berjalan otomatis setiap kali Raspberry Pi dinyalakan menggunakan **systemd service**.

---

## Prasyarat

- Raspberry Pi 64-bit menyala dan terhubung ke internet
- File `main.py` sudah ada di `/home/trk/robot/auto_remote/` **(sesuaikan direktori kalian masing masing)**
- Python sistem sudah terinstall (`python3`)

---

## Step 1 — Buat file service

Buat file service baru di direktori systemd:

```bash
sudo nano /etc/systemd/system/robot-mobil.service
```

Isi dengan konfigurasi berikut:

```ini
[Unit]
Description=Robot Mobil - Obstacle Avoidance + Remote Control
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/home/trk/robot/auto_remote
ExecStart=/usr/bin/python3 /home/trk/robot/auto_remote/main.py
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Simpan file: **Ctrl+X → Y → Enter**

### Penjelasan konfigurasi

| Baris | Keterangan |
|---|---|
| `After=network-online.target` | Service baru jalan setelah network siap |
| `User=root` | Dijalankan sebagai root karena akses GPIO butuh root |
| `WorkingDirectory` | Folder kerja script, penting agar path relatif (folder UI) terbaca |
| `ExecStart` | Perintah untuk menjalankan script Python |
| `Restart=on-failure` | Otomatis restart jika script crash |
| `RestartSec=5s` | Tunggu 5 detik sebelum restart |

---

## Step 2 — Aktifkan service

Reload systemd agar mengenali service baru:

```bash
sudo systemctl daemon-reload
```

Enable service supaya otomatis jalan saat boot:

```bash
sudo systemctl enable robot-mobil
```

Jalankan service sekarang tanpa harus reboot:

```bash
sudo systemctl start robot-mobil
```

---

## Step 3 — Verifikasi service berjalan

```bash
sudo systemctl status robot-mobil
```

Output yang menandakan **berhasil**:

```
● robot-mobil.service - Robot Mobil - Obstacle Avoidance + Remote Control
   Active: active (running)
   Main PID: xxxx (python3)
   ...
```

---

## Step 4 — Test akses HTTP server

Setelah service berjalan, robot langsung membuka HTTP server di port 5000. Akses dari browser di HP atau laptop yang satu jaringan:

```
http://<IP_RASPBERRY>:5000 (atau langsung akses dengan menggunakan domain kalian yang sudah dibuatkan tunnel sebelumnya)
```

---

## Perintah Berguna

| Perintah | Fungsi |
|---|---|
| `sudo systemctl start robot-mobil` | Jalankan robot |
| `sudo systemctl stop robot-mobil` | Hentikan robot |
| `sudo systemctl restart robot-mobil` | Restart robot |
| `sudo systemctl status robot-mobil` | Cek status service |
| `sudo journalctl -fu robot-mobil` | Lihat log realtime |
| `sudo systemctl disable robot-mobil` | Nonaktifkan autostart |

---

## Troubleshooting

| Masalah | Penyebab | Solusi |
|---|---|---|
| Service gagal start | Path script salah | Cek `ExecStart` dan `WorkingDirectory` di file service |
| UI tidak terbuka | Folder `UI/index.html` tidak ditemukan | Pastikan folder `UI` ada di `/home/trk/robot/auto_remote/UI/` |
| GPIO error | Script tidak dijalankan sebagai root | Pastikan `User=root` ada di file service |
| Port 5000 tidak bisa diakses | Firewall atau service belum jalan | Cek `sudo systemctl status robot-mobil` |

---

## Catatan

> Script ini dijalankan sebagai `root` karena `RPi.GPIO` membutuhkan akses langsung ke hardware GPIO. Jika ingin lebih aman, tambahkan user `trk` ke grup `gpio` dan ubah `User=trk` di file service.

```bash
# Alternatif lebih aman (opsional)
sudo usermod -aG gpio trk
```

Lalu ubah `User=root` menjadi `User=trk` di file service, kemudian:

```bash
sudo systemctl daemon-reload
sudo systemctl restart robot-mobil
```
