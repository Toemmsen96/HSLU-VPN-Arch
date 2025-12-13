# HSLU-VPN-arch
pulsesecure package build for HSLU VPN for Arch linux
## Requirements
The following packages are required:
```sh
sudo pacman -S gcc-libs libgnome-keyring openssl curl dbus libbsd dmidecode gtkmm3 webkit2gtk psmisc
```
if anything is missing the building of the package will tell you

## Installation
- clone this repo
- Download the Linux package for HSLU VPN Ivanti Secure Access from https://softwaredownload.hslu.ch
- Extract .rpm file and its .sha256 into this repo folder
- Install by using makepkg -i
- Install any dependencies if they are missing and repeat

## Post install
- After installation run ```sudo systemctl start pulsesecure``` or ```sudo systemctl enable pulsesecure```

- run setup_cef.sh for the chromium embedded browser
```sh
sudo /opt/pulsesecure/bin/setup_cef.sh install
```
if that fails with the error
```sh
./setup_cef.sh: line 353: -a: command not found
```
edit the script and add the location of your binary for shasum where ```SHASUM=``` is defined by getting it using ```where shasum``` and entering your path 

- Start pulsesecure

```sh
# enter this for onetime start
sudo systemctl start pulsesecure
# or this to enable it for every startup:
sudo systemctl enable pulsesecure 
```
n PulseUI# Connect
Connect by adding https://vpn.hslu.ch if it isn't already
