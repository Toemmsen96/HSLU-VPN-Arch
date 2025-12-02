# HSLU-VPN-arch
pulsesecure package for HSLU VPN for Arch linux
## Installation
- clone this repo
- Download the Linux package for HSLU VPN Ivanti Secure Access from https://softwaredownload.hslu.ch
- Extract .rpm file and its .sha256 into this repo folder
- Install by using makepkg -i
- Install any dependencies if they are missing
- After installation run sudo systemctl start pulsesecure or sudo systemctl enable pulsesecure
- It might be required to run /opt/pulsesecure/bin/setup_cef.sh install for everything to work properly
- Open PulseUI
## Connect
Connect by adding https://vpn.hslu.ch if it isn't already
