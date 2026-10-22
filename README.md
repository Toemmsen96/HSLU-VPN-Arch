# HSLU-VPN-arch
PulseSecure / Ivanti Secure Access Client package build for HSLU VPN on Arch Linux

## What changed in v0.0.2

- **Source changed from `.rpm` to `.deb`** — the installer from HSLU's download portal now only ships a `.deb` (Ubuntu/Debian) and `.rpm`; this package uses the `.deb`.
- **No `.sha256` file needed** — checksum verification via a separate file was dropped.
- **Arch compatibility shims** — the following Ubuntu 22.04 libraries are bundled in `/opt/pulsesecure/lib` to work around missing or incompatible versions on Arch:
  - `libwebkit2gtk-4.0` / `libjavascriptcoregtk-4.0` — dropped from Arch; symlinking to `webkit2gtk-4.1` causes a fatal `libsoup2`/`libsoup3` conflict at runtime
  - `libicu70` — ICU symbol versions differ between 70 and 78; cannot redirect
  - `libxml2 2.9` — same issue with XML2 symbol versions vs Arch's 2.12

## Requirements

The following packages are required:
```sh
sudo pacman -S gcc-libs libgnome-keyring openssl curl dbus libbsd dmidecode gtkmm3 webkit2gtk-4.1 psmisc libsoup
```

If anything is missing, `makepkg` will tell you.

## Installation

1. Clone this repo
2. Download the Linux **`.deb`** package for HSLU VPN Ivanti Secure Access from https://softwaredownload.hslu.ch
3. Place the `.deb` file (e.g. `ps-pulse-linux-22.8r5-b41063-64bit-installer.deb`) into this repo folder
4. Run `makepkg -i`
5. Install any missing dependencies and repeat

If you are updating from an older version, you may need to run `makepkg -Ccf && makepkg -i`, to get a clean and updated build instead of getting the old version installed. 

## Post install

### Start the service

```sh
# one-time start
sudo systemctl start pulsesecure
# or enable at every boot
sudo systemctl enable pulsesecure
```

Verify it is running:
```sh
systemctl status pulsesecure.service
```

### Install the Chromium Embedded Browser (CEF v132)

Required for the SAML login portal. On first login the GUI will prompt you to install it — confirm with OK. If the GUI install fails, run manually:

```sh
sudo /opt/pulsesecure/bin/setup_cef.sh install
```

If the installer reports a missing `shasum` utility, replace the two occurrences of `$SHASUM -a 256` in `setup_cef.sh` with `sha256sum`.

Verify the CEF installation:
```sh
sudo /opt/pulsesecure/bin/setup_cef.sh check_installed
```

If the CEF lib still shows `not installed`, uninstall, reboot, and reinstall:
```sh
sudo /opt/pulsesecure/bin/setup_cef.sh uninstall
# reboot, then:
sudo /opt/pulsesecure/bin/setup_cef.sh install
```

### Test the UI

```sh
/opt/pulsesecure/bin/pulseUI.webkit41
```

> **Note:** `pulseUI` requires the `pulsesecure` service to be running, otherwise it will fail with DBus errors. Start the service first.
> naming for `pulseUI.webkit41` may change in future versions. If not found, navigate to `/opt/pulsesecure/bin/` and choose an applicable `pulseUI`.

## Connect

Open the **Pulse** application and add a new connection:

| Field | HSLU                 | PHLU                 |
|-------|----------------------|----------------------|
| Name  | HSLU                 | PHLU                 |
| URL   | https://vpn.hslu.ch/ | https://vpn.phlu.ch/ |

Click **Add**, then **Connect**. Enter your user ID, password, and confirm MFA.

To disconnect, click **Disconnect** on the connection entry.
