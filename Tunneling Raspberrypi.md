**Tunneling-Raspi**

**Step 1** Install cloudflaredcloudflared adalah aplikasi client dari Cloudflare yang akan membuat tunnel dari Raspi kamu ke jaringan Cloudflare. Karena Raspi kamu 64-bit, kita pakai versi arm64.
```bash
trk@trk:~ $ curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb -o cloudflared.deb
```
**Menginstall package yang sudah didownload ke sistem.**
```bash
sudo dpkg -i cloudflared.deb
```
**Step 2 — Login ke akun Cloudflare**
Langkah ini menghubungkan cloudflared di Raspi kamu dengan akun Cloudflare. Credentials login akan disimpan sebagai file ```cert.pem.```

```bash
cloudflared tunnel login
```

**Step 3 — Buat Tunnel**
Tunnel adalah koneksi terenkripsi antara Raspi kamu dan jaringan Cloudflare. Setiap tunnel punya ID unik yang akan dipakai di konfigurasi.
```bash
cloudflared tunnel create robot-tunnel
```
Verifikasi tunnel sudah terbuat:
```bash
cloudflared tunnel list
```
**Step 4 — Buat file konfigurasi**
File config memberi tahu cloudflared tunnel mana yang dipakai, dan traffic dari hostname mana yang diarahkan ke port berapa di Raspi.

```bash
sudo mkdir -p /etc/cloudflared
sudo nano /etc/cloudflared/config.yml
```
Isi file dengan konfigurasi berikut:
```bash
tunnel: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
credentials-file: /etc/cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json

ingress:
  - hostname: robot.domainkamu.com
    service: http://localhost:5000
  - service: http_status:404
```
**Step 5 — Copy credentials ke /etc/cloudflared**
File JSON credentials tunnel tadi tersimpan di folder home user. Kita perlu copy ke ```/etc/cloudflared``` supaya bisa dibaca oleh service yang berjalan sebagai root.
```bash
sudo cp ~/.cloudflared/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json /etc/cloudflared/
```
Verifikasi file sudah ada:
```bash
sudo ls /etc/cloudflared/
```
Output yang diharapkan:
```bash
config.yml    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx.json
```

**Step 6 — Buat DNS Record otomatis**
Langkah ini membuat CNAME record di Cloudflare Dashboard secara otomatis, yang mengarahkan subdomain kamu ke tunnel.

```bash
cloudflared tunnel route dns robot-tunnel robot.domainkamu.com
```
Output sukses:
```bash
Added CNAME robot.domainkamu.com which will route to this tunnel tunnelID=xxxxxxxx-...
```

**Step 7 — Install & jalankan sebagai service**
Agar cloudflared otomatis berjalan setiap kali Raspi dinyalakan, kita install sebagai systemd service.
```bash
sudo cloudflared service install
```
Perintah ```enable``` memastikan service otomatis start saat boot.
```bash
sudo systemctl enable cloudflared
```
Menjalankan service sekarang tanpa perlu reboot.
```bash
sudo systemctl start cloudflared
```

**Step 8 — Verifikasi tunnel berjalan**
```bash
sudo systemctl status cloudflared
```
Output yang menandakan berhasil:
```bash
● cloudflared.service - cloudflared
   Active: active (running)
   ...
   INF Registered tunnel connection connIndex=0
   INF Registered tunnel connection connIndex=1
   INF Registered tunnel connection connIndex=2
   INF Registered tunnel connection connIndex=3
```

**Step 9 — Test akses dari luar**
```bash
curl -I https://robot.domainkamu.com
```
akses langsung menggunakan ```robot.domainkamu.com``` menggunakan browser

**SELESAI**

