# Latihan 3 tier
## Gambarajah 3 tier architecture  
![3 Tier Architecture](/assets/3-tier-architecture.png)

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
### Pemasangan lxc
1. Pasangkan lxc  
`sudo apt update && sudo apt install lxc`

### Pemasangan mariadb galera cluster dalam container
1. Wujudkan satu lxc container  
`sudo lxc-create -n db1 -t download -- -d ubuntu -r noble -a amd64`

2. Jalankan container  
`sudo lxc-start -n db1`

3. Masuk ke container  
`sudo lxc-attach -n db1`

4. Pasangkan curl  
`apt install curl -y` 

5. Pasang repo mariadb  
`curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash`

6. Pasangkan mariadb dan galera cluster  
`apt install mariadb-server mariadb-client galera4 -y`

7. Pastikan servis mariadb sudah dihidupkan
`sudo systemctl start mariadb`

8. Tingkatkan keselamatan mariadb
`sudo mysql_secure_installation`

9. Matikan db1
`sudo lxc-stop db1`

10. Klon db1 menjadi db2 dan db3
`sudo lxc-copy -n db1 -N db2 && sudo lxc-copy -n db1 -N db3`

11. Hidupkan semua container, dan dapatkan alamat ip
`sudo lxc-start db1; sudo lxc-start db2; sudo lxc-start db3`
`sudo lxc-ls -f`

12. Masuk ke db1, dan wujudkan fail /etc/mysql/conf.d/galera.cnf
`sudo lxc-attach -n db1`
`echo "
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster Configuration
wsrep_cluster_name="mycluster"
wsrep_cluster_address="gcomm://10.0.3.218,10.0.3.249,10.0.3.223"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Node Configuration
wsrep_node_address="10.0.3.218"
wsrep_node_name="db1"" | sudo tee /etc/mysql/conf.d/galera.cnf`

13. Masuk ke db2, dan wujudkan fail /etc/mysql/conf.d/galera.cnf
`sudo lxc-attach -n db2`
`echo "
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster Configuration
wsrep_cluster_name="mycluster"
wsrep_cluster_address="gcomm://10.0.3.218,10.0.3.249,10.0.3.223"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Node Configuration
wsrep_node_address="10.0.3.249"
wsrep_node_name="db2"" | sudo tee /etc/mysql/conf.d/galera.cnf`

14. Masuk ke db3, dan wujudkan fail /etc/mysql/conf.d/galera.cnf  
`sudo lxc-attach -n db3`
`echo "
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Cluster Configuration
wsrep_cluster_name="mycluster"
wsrep_cluster_address="gcomm://10.0.3.218,10.0.3.249,10.0.3.223"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Node Configuration
wsrep_node_address="10.0.3.223"
wsrep_node_name="db3"" | sudo tee /etc/mysql/conf.d/galera.cnf`

15. Masuk ke db1 (node pertama dalam senarai gcomm), matikan mariadb dan mulakan galera cluster  
`sudo lxc-attach -n db1 -- systemctl stop mariadb`
`sudo lxc-attach -n db1 -- galera_new_cluster`
 
16. Semak bahawa galera cluster cluster sudah berjaya dibentuk
`sudo lxc-attach -n db1` 
`mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"`

17. Hidupkan mariadb di db2 dan db3
`sudo lxc-attach -n db2 -- systemctl start mariadb`
`sudo lxc-attach -n db3 -- systemctl start mariadb`

18. Semak saiz galera cluster, pastikan saiz bertambah selepas mariadb di db2 dan db3 dihidupkan 
`sudo lxc-attach -n db1` 
`mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"`
`mysql -u root -p -e "show status like 'wsrep_local_state_comment'`

### Pemasangan maxscale
https://severalnines.com/blog/how-install-and-configure-maxscale-mariadb/

1. Pasangkan keperluan  
`sudo apt update && sudo apt install curl ca-certificates apt-transport-https -y`

2. Tambahkan repo mariadb  
`curl -LsS https://r.mariadb.com/downloads/mariadb_repo_setup | sudo bash`

3. Pasangkan maxscale  
`sudo apt update && sudo apt install maxscale -y`

4. Wujudkan akaun pengguna untuk maxscale  
`mysql -u root -p -h db1`
`
CREATE USER 'maxscale'@'%' IDENTIFIED BY 'maxscale_password';
GRANT SELECT ON mysql.user TO 'maxscale'@'%';
GRANT SELECT ON mysql.db TO 'maxscale'@'%';
GRANT SELECT ON mysql.tables_priv TO 'maxscale'@'%';
GRANT SELECT ON mysql.columns_priv TO 'maxscale'@'%';
GRANT SELECT ON mysql.procs_priv TO 'maxscale'@'%';
GRANT SELECT ON mysql.proxies_priv TO 'maxscale'@'%';
GRANT SELECT ON mysql.roles_mapping TO 'maxscale'@'%';
GRANT SHOW DATABASES ON *.* TO 'maxscale'@'%';
`
5. Wujudkan akaun pengguna untuk client dan berikan "grants" yang sama seperti pengguna dalam database  
`mysql -u root -p -h db1`
`CREATE USER 'wpuser'@'maxscale-host' IDENTIFIED BY 'my_secret_password';
show grants for user wpuser@db1;
grant select, insert, update, delete on *.* wpuser@maxscale-host;`

6. Konfigurasi maxscale  
`echo "
[maxscale]          
threads=auto
log_augmentation = 1 
ms_timestamp = 1
syslog = 1   
[server1]                 
type=server         
address=10.0.3.218
port=3306                
protocol=MariaDBBackend
[server2]
type=server          
address=10.0.3.249
port=3306                 
protocol=MariaDBBackend
[server3]
type=server     
address=10.0.3.223
port=3306
protocol=MariaDBBackend
[MariaDB-Monitor]
type=monitor
module=mariadbmon
servers=server1,server2,server3
user=maxscale
password=maxscale_password
monitor_interval=2s
[Read-Only-Service]
type=service
router=readconnroute
servers=server2,server3
user=maxscale
password=maxscale_password
router_options=slave
[Read-Write-Service]
type=service
router=readwritesplit
servers=server1
user=maxscale
password=maxscale_password
[Read-Only-Listener]
type=listener
service=Read-Only-Service
protocol=MariaDBClient
port=4008
[Read-Write-Listener]
type=listener
service=Read-Write-Service
protocol=MariaDBClient
port=4006
" | sudo tee /etc/maxscale/maxscale.cnf 

7. Restart servis maxscale  
`sudo systemctl restart maxscale`

8. Periksa services dalam maxscale  
`sudo maxctl list services`

9. Periksa servers dalam maxscale  
`sudo maxctl list servers`


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
