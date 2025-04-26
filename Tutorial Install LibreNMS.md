# Instalasi LibreNMS on Ubuntu Server 24.04 LTS

## Lets Get Start

1. **Perbarui sistem:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

2. **Pasang paket-paket yang dibutuhkan:**
   ```bash
   sudo apt install acl curl fping git graphviz imagemagick mariadb-client mariadb-server mtr-tiny nginx-full nmap php-cli php-curl php-fpm php-gd php-gmp php-json php-mbstring php-mysql php-snmp php-xml php-zip rrdtool snmp snmpd unzip python3-command-runner python3-pymysql python3-dotenv python3-redis python3-setuptools python3-psutil python3-systemd python3-pip whois traceroute -y
   ```

## Menambahkan Pengguna LibreNMS

```bash
sudo useradd librenms -d /opt/librenms -M -r -s "$(which bash)"
```

## Mengunduh dan Mengatur LibreNMS

1. **Klon repositori LibreNMS:**
   ```bash
   cd /opt
   sudo git clone https://github.com/librenms/librenms.git
   ```

2. **Atur kepemilikan dan izin:**
   ```bash
   sudo chown -R librenms:librenms /opt/librenms
   sudo chmod 771 /opt/librenms
   sudo setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
   sudo setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
   ```

## Instalasi Dependensi PHP

```bash
sudo su - librenms
./scripts/composer_wrapper.php install --no-dev
exit
```

## Mengatur Zona Waktu PHP

1. **Edit konfigurasi PHP:**
   ```bash
   sudo nano /etc/php/8.2/fpm/php.ini
   ```

2. **Ubah baris:**
   ```ini
   date.timezone = Asia/Jakarta
   ```

3. **Restart PHP-FPM:**
   ```bash
   sudo systemctl restart php8.2-fpm
   ```

## Konfigurasi MariaDB

1. **Amankan MariaDB:**
   ```bash
   sudo mysql_secure_installation
   ```

2. **Buat database dan user:**
   ```bash
   sudo mysql -u root -p

   CREATE DATABASE librenms CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'password_kuat';
   GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

## Konfigurasi Nginx

1. **Buat konfigurasi Nginx:**
   ```bash
   sudo nano /etc/nginx/sites-available/librenms
   ```

2. **Isi file konfigurasi:**
   ```nginx
   server {
       listen 80;
       server_name librenms.example.com;
       root /opt/librenms/html;
       index index.php;

       location / {
           try_files $uri $uri/ /index.php?$query_string;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/run/php/php8.2-fpm.sock;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

3. **Aktifkan dan restart Nginx:**
   ```bash
   sudo ln -s /etc/nginx/sites-available/librenms /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## Konfigurasi SNMP

1. **Salin file konfigurasi:**
   ```bash
   sudo cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
   ```

2. **Restart SNMPD:**
   ```bash
   sudo systemctl restart snmpd
   ```

## Menambahkan Cron Job dan Logrotate

```bash
sudo cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms
sudo cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms
```

## 💻 Instalasi Web

1. **Buka browser dan navigasi ke:**
   ```
   http://librenms.example.com/install
   ```

2. **Ikuti panduan instalasi web.**

## Validate Instalasi

Pastikan semua item pada halaman validasi ditandai dengan "OK". Jika ada masalah, periksa kembali konfigurasi Anda.

---

**Catatan:** Gantilah `librenms.example.com` dengan nama domain atau IP server Anda.

Untuk info lebih lanjut, kunjungi: [https://docs.librenms.org](https://docs.librenms.org)

Jika Installasi Server LibreNMS sudah selesai dan Validate All is OK, kalian bisa add Device yang ingin di monitor dengan cara mengaktifkan dan membuat Profil SNMP dan arah kan IP ke IP server LibreNMS anda.
Anda jg bisa membuat Poller menjadi 1 menit dengan merubah config di Cron Tab nya.
