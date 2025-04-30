Step-by-Step Setup Nginx di Rocky Linux
1. Update Sistem
```bash
sudo dnf update -y
```
2. Install Nginx
```bash
sudo dnf install nginx -y
```
3. Start dan Enable Nginx
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```
Cek status:
```bash
sudo systemctl status nginx
```

4. Buka Port HTTP & HTTPS di Firewall
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
Cek:

```bash
sudo firewall-cmd --list-all
```
5. Akses Web Server
Coba akses di browser:
```
http://[IP_ADDRESS_VM]
```
Harusnya muncul halaman default Nginx: "Welcome to Nginx on Rocky Linux".

6. Ubah Root Web Directory
Default root: /usr/share/nginx/html

Ganti file index.html:

```bash
echo "<h1>Halo dari Rocky Linux</h1>" | sudo tee /usr/share/nginx/html/index.html
```
7. Log File Lokasi
Access log: /var/log/nginx/access.log

Error log: /var/log/nginx/error.log

Cek log:

```bash
sudo tail -f /var/log/nginx/access.log
```
8. Autostart & Recovery
Nginx biasanya sudah auto-start setelah reboot karena systemctl enable.

Kalau kamu mau pastikan auto-restart jika crash, buat override:

```bash
sudo systemctl edit nginx
```
Isi dengan:

```bash
[Service]
Restart=always
RestartSec=5
```
Simpan, lalu:

```bash
sudo systemctl daemon-reexec
```
9. Optional Security: Hide Nginx Version
Edit /etc/nginx/nginx.conf:

```bash
sudo nano /etc/nginx/nginx.conf
```
Di dalam http {} tambahkan:

```
server_tokens off;
```
Simpan, lalu restart:

```bash
sudo systemctl restart nginx
```
10. Test Konfigurasi
Setiap habis edit config, test dulu:

```bash
sudo nginx -t
```
Kalau OK:

```bash
sudo systemctl reload nginx
```
