# Latihan 3 tier

## Pemasangan virtualbox
1. Muat turun virtualbox installer dari [sini](https://download.virtualbox.org/virtualbox/7.1.6/VirtualBox-7.1.6-167084-Win.exe)
2. Pasangkan virtualbox dengan mengikuti wizard 
3. Reboot mesin

## Pemasangan linux dalam virtualbox

### Ubuntu
1. Muat turun iso ubuntu server dari [sini](https://releases.ubuntu.com/24.04.1/ubuntu-24.04.1-live-server-amd64.iso)
2. Pasangkan ubuntu dalam virtualbox
3. Update pakej aplikasi dalam ubuntu 
`sudo apt update && sudo apt upgrade -y`
4. Reboot vm ubuntu
`sudo reboot`

### Almalinux
1. Muat turun iso almalinux minimal dari [sini](https://repo.almalinux.org/almalinux/9.5/isos/x86_64/AlmaLinux-9.5-x86_64-minimal.iso)
2. Pasangkan almalinux dalam virtualbox
3. Update almalinux dalam virtualbox 
`sudo dnf update -y`
4. Reboot vm almalinux
`sudo reboot`

## Pemasangan php-fpm


## Pemasangan bunkerweb

### Ubuntu
1. Pasangkan repo 

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

4. Masukkan konfigurasi bunkerweb

echo "
HTTP_PORT=80
HTTPS_PORT=443
DNS_RESOLVERS=8.8.8.8 8.8.4.4
API_LISTEN_IP=127.0.0.1
DISABLE_DEFAULT_SERVER=yes
AUTO_LETS_ENCRYPT=yes
USE_CLIENT_CACHE=yes
USE_GZIP=yes
LOCAL_PHP=/run/php/php-fpm.sock
LOCAL_PHP_PATH=/var/www/html
MAX_CLIENT_SIZE=50m" | sudo tee /etc/bunkerweb/variables.env

5. Hidupkan bunkerweb, dan aktifkan ketika boot

sudo systemctl enable --now bunkerweb

6. Benarkan port http dan https di ufw

sudo ufw allow https
sudo ufw allow http

