# Latihan 3 tier

## Pemasangan virtualbox
1. Muat turun virtualbox installer dari [sini](https://download.virtualbox.org/virtualbox/7.1.6/VirtualBox-7.1.6-167084-Win.exe)
2. Pasangkan virtualbox dengan mengikuti wizard 
3. Reboot mesin

## Pemasangan linux dalam virtualbox

### Ubuntu
1. Muat turun iso ubuntu server dari [sini](https://releases.ubuntu.com/24.04.1/ubuntu-24.04.1-live-server-amd64.iso)
2. Klik "New"  
![Klik "New"](/assets/vbox-ubuntu-1.png)
3. Masukkan butiran VM seperti nama, iso dan lokasi penyimpanan fail disk, dan klik "Next" ![Butiran VM](/assets/vbox-ubuntu-2.png)
4. Masukkan spesifikasi cpu dan memory VM dan klik "Next" ![CPU & Memory VM](/assets/vbox-ubuntu-3.png)
5. Masukkan spesifikasi disk VM dan klik "Next" ![Disk VM](/assets/vbox-ubuntu-4.png)
6. Semak konfigurasi VM dan klik "Finish" ![Confirmation](/assets/vbox-ubuntu-5.png)
7. Klik nama VM dan klik "Start" ![Hidupkan VM](/assets/vbox-ubuntu-6.png)
8. Ikuti installation wizard untuk memasang ubuntu
9. Reboot selepas siap pemasangan
10. Update pakej aplikasi dalam ubuntu   
`sudo apt update && sudo apt upgrade -y`
11. Reboot vm ubuntu  
`sudo reboot`

### Almalinux
1. Muat turun iso almalinux minimal dari [sini](https://repo.almalinux.org/almalinux/9.5/isos/x86_64/AlmaLinux-9.5-x86_64-minimal.iso)
2. Pasangkan almalinux dalam virtualbox
3. Update almalinux dalam virtualbox  
`sudo dnf update -y`
4. Reboot vm almalinux  
`sudo reboot`

## Pemasangan database cluster
### Pemasangan maxscale
1. Pasangkan keperluan  
`sudo apt update && sudo apt install curl ca-certificates apt-transport-https -y`

2. Tambahkan repo mariadb  
`curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash`

3. Pasangkan maxscale  
`sudo apt update && sudo apt install maxscale -y`

### Pemasangan lxc


### Pemasangan mariadb
1. Pastikan repo mariadb sudah dipasang

2. Pasangkan mariadb  
`sudo apt install mariadb-server mariadb-client mariadb-backup -y`


## Pemasangan php

1. Pasangkan php
`sudo apt update && sudo apt install php -y`

2. Untuk menguji php, wujudkan fail /var/www/html/index.php dengan kod di bawah
`echo "
<?php
echo "this is my php page";
?>" | sudo tee /var/www/html/index.php`

3. Hidupkan pelayan web php terbina dalam
`cd /var/www/html/
sudo php -S 0.0.0.0:8000`

4. Akses localhost:8000 menggunakan curl atau web browser
`curl localhost:8000`

## Pemasangan nginx & bunkerweb
1. Pasangkan repo nginx
`sudo apt install -y curl gnupg2 ca-certificates lsb-release ubuntu-keyring && \
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
| sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null && \
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
| sudo tee /etc/apt/sources.list.d/nginx.list`

2. Pasangkan nginx 1.26.2  
`sudo apt update && \
sudo apt install -y nginx=1.26.2-1~$(lsb_release -cs)`

Pilihan: eksport untuk aktifkan setup wizard  
`export UI_WIZARD=1`

3. Pasangkan bunkerweb  
`curl -s https://repo.bunkerweb.io/install/script.deb.sh | sudo bash && \
sudo apt update && \
sudo -E apt install -y bunkerweb=1.5.12`

4. Kunci versi nginx dan bunkerweb, untuk mengelakkan kemaskini perisian tanpa sengaja  
`sudo apt-mark hold nginx bunkerweb`

5. Masukkan tetapan bunkerweb  
`echo "
DNS_RESOLVERS=9.9.9.9 8.8.8.8 8.8.4.4
HTTP_PORT=80
HTTPS_PORT=443
API_LISTEN_IP=127.0.0.1
UI_HOST=http://127.0.0.1:7000
" | sudo tee /etc/bunkerweb/variables.env`

6. Wujudkan fail database untuk bunkerweb-ui
`sudo touch /var/lib/bunkerweb/db.sqlite3`

7. Matikan apache2, hidupkan bunkerweb & bunkerweb-ui, dan aktifkan ketika boot  
`sudo systemctl disable --now apache2 &&
sudo systemctl enable --now bunkerweb && 
sudo systemctl enable --now bunkerweb-ui`

8. Untuk akses bunkerweb-ui, memandangkan bunkerweb-ui hanya "listen" di localhost, gunakan ssh tunnel
- ssh kepada vm, dan wujudkan tunnel: ssh vm -L 7000:localhost:7000
- buka browser, dan layari http://localhost:7000
- masukkan username dan password yang dikehendaki, password memerlukan 8 aksara, dengan kombinasi nombor, huruf dan simbol ![Masukkan username dan password](/assets/bunkerweb-1.png)
- masukkan UI Host, UI URL dan Server name ![Masukkan UI Host, UI URL dan Server name](/assets/bunkerweb-2.png)
- tandakan "I've read and agree to the privacy policy" dan tekan "SETUP" ![Tekan "SETUP" selepas setuju dengan privacy policy](/assets/bunkerweb-3.png)
