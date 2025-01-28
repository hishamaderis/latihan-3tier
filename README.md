# Latihan-3tier

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
