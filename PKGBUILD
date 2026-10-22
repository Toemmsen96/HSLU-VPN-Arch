# Maintainer: Toemmsen96 <147745097+Toemmsen96@users.noreply.github.com>

_latest_deb=$(ls -t ps-pulse-linux-*-64bit-installer.deb 2>/dev/null | head -n1)

# webkit2gtk-4.0 and javascriptcoregtk-4.0 were dropped from Arch; pulseUI links them directly.
# Symlinking to 4.1 causes a libsoup2/libsoup3 conflict at runtime, so we bundle the Ubuntu 22.04
# versions along with their exact dependency versions (ICU 70, libxml2 2.9) to avoid symbol
# version mismatches against the newer libraries Arch ships.
_webkit_deb="libwebkit2gtk-4.0-37_2.50.4-0ubuntu0.22.04.1_amd64.deb"
_jsc_deb="libjavascriptcoregtk-4.0-18_2.50.4-0ubuntu0.22.04.1_amd64.deb"
_icu_deb="libicu70_70.1-2_amd64.deb"
_libxml2_deb="libxml2_2.9.13+dfsg-1build1_amd64.deb"
_ubuntu_webkit="http://security.ubuntu.com/ubuntu/pool/main/w/webkit2gtk"
_ubuntu_icu="http://archive.ubuntu.com/ubuntu/pool/main/i/icu"
_ubuntu_xml2="http://archive.ubuntu.com/ubuntu/pool/main/libx/libxml2"

pkgname=hslu-pulsesecure
pkgver=0.0.2
pkgrel=1
pkgdesc='Ivanti Secure Access Client for HSLU'
arch=(x86_64)
license=(custom)
url='https://www.pulsesecure.net/'
depends=(gcc-libs gnome-keyring openssl curl dbus libbsd dmidecode gtkmm3 webkit2gtk-4.1 psmisc libsoup3)
install=${pkgname}.install
source=(EULA.txt
        "$_ubuntu_webkit/$_webkit_deb"
        "$_ubuntu_webkit/$_jsc_deb"
        "$_ubuntu_icu/$_icu_deb"
        "$_ubuntu_xml2/$_libxml2_deb")
source_x86_64=("$_latest_deb")
sha256sums=('823b298db462d9963148888287ebde2eb013c9b7d0635f3893d18a35776fa8c3'
            '121a795af0b4851eaa13b4e0cd55ba6261d7cf1578801bf39ce7c295af7d1174'
            '68aafd3f6d0a560276875e6275713dc5f55e8cda570f9e33d38d94ba4906ce5e'
            '58a154f6307289813da2276f900498ef536ae7c0522d2cf31a3c3c5cf62dfd9a'
            '4831826d0e320f6715722716d9737c7e57a052d20e55dd03ff7444c6e502a3ff')
sha256sums_x86_64=('SKIP')
options=(!strip !debug)
conflicts=(pulse-connect-secure pulse-secure)

prepare() {
    cd "$srcdir"
    # Extract pulse installer
    ar x "$_latest_deb"
    mv data.tar.zst pulse-data.tar.zst
    bsdtar -xf pulse-data.tar.zst

    # Extract webkit2gtk-4.0 and javascriptcoregtk-4.0 .so files from Ubuntu debs
    for deb in "$_webkit_deb" "$_jsc_deb"; do
        ar x "$deb"
        mv data.tar.zst "${deb%.deb}-data.tar.zst"
        bsdtar -xf "${deb%.deb}-data.tar.zst" --strip-components=4 \
            "./usr/lib/x86_64-linux-gnu/libwebkit2gtk-4.0.so.37.74.7" \
            "./usr/lib/x86_64-linux-gnu/libjavascriptcoregtk-4.0.so.18.28.9" 2>/dev/null || true
    done

    # Extract ICU 70 — symbol versions differ between ICU 70 and 78, can't redirect
    ar x "$_icu_deb"
    mv data.tar.zst "${_icu_deb%.deb}-data.tar.zst"
    bsdtar -xf "${_icu_deb%.deb}-data.tar.zst" --strip-components=4 \
        "./usr/lib/x86_64-linux-gnu/libicui18n.so.70.1" \
        "./usr/lib/x86_64-linux-gnu/libicuuc.so.70.1" \
        "./usr/lib/x86_64-linux-gnu/libicudata.so.70.1" 2>/dev/null || true
    ln -sf libicui18n.so.70.1 libicui18n.so.70
    ln -sf libicuuc.so.70.1   libicuuc.so.70
    ln -sf libicudata.so.70.1 libicudata.so.70

    # Extract libxml2 2.9 — symbol versions also differ vs Arch's libxml2 2.12
    ar x "$_libxml2_deb"
    mv data.tar.zst "${_libxml2_deb%.deb}-data.tar.zst"
    bsdtar -xf "${_libxml2_deb%.deb}-data.tar.zst" --strip-components=4 \
        "./usr/lib/x86_64-linux-gnu/libxml2.so.2.9.13" 2>/dev/null || true
    ln -sf libxml2.so.2.9.13 libxml2.so.2
}

package() {
    install -Dm644 EULA.txt "$pkgdir"/usr/share/licenses/$pkgname/EULA.txt

    for d in $(find opt/pulsesecure -type d); do
        install -dm755 "$d" "$pkgdir"/"$d";
    done
    for f in $(find opt/pulsesecure/bin -type f); do
        install -Dm755 "$f" "$pkgdir"/"$f";
    done
    for f in $(find opt/pulsesecure/lib -type f); do
        install -Dm755 "$f" "$pkgdir"/"$f";
    done
    for f in $(find opt/pulsesecure/resource -type f); do
        install -Dm644 "$f" "$pkgdir"/"$f";
    done
    install -Dm644 usr/share/man/man1/pulse.1.gz "$pkgdir"/usr/share/man/man1/pulse.1.gz

    # we move service unit file to /usr/lib/systemd/system due to pacman limitation
    install -Dm644 lib/systemd/system/pulsesecure.service "$pkgdir"/usr/lib/systemd/system/pulsesecure.service

    mkdir -p "$pkgdir"/usr/share/applications/ "$pkgdir"/usr/share/dbus-1/system.d/ "$pkgdir"/opt/pulsesecure/lib/JUNS/interfaces
    mkdir -p "$pkgdir"/var/lib/pulsesecure/pulse
    ln -s /opt/pulsesecure/resource/pulse.desktop "$pkgdir"/usr/share/applications/pulse.desktop
    ln -s /opt/pulsesecure/lib/JUNS/net.psecure.pulse.conf "$pkgdir"/usr/share/dbus-1/system.d/net.psecure.pulse.conf
    mkdir -p "$pkgdir"/etc/pki/ca-trust/extracted/openssl
    ln -sf /etc/ca-certificates/extracted/ca-bundle.trust.crt "$pkgdir"/etc/pki/ca-trust/extracted/openssl/ca-bundle.trust.crt
    for f in $(find opt/pulsesecure/lib/JUNS/interfaces -type l); do
        ln -s $(readlink $f) "$pkgdir"/"$f" ;
    done

    # Bundle webkit2gtk-4.0, javascriptcoregtk-4.0, and ICU 70 from Ubuntu 22.04
    local libdir="$pkgdir"/opt/pulsesecure/lib
    install -Dm755 "$srcdir"/libwebkit2gtk-4.0.so.37.74.7      "$libdir"/libwebkit2gtk-4.0.so.37
    install -Dm755 "$srcdir"/libjavascriptcoregtk-4.0.so.18.28.9 "$libdir"/libjavascriptcoregtk-4.0.so.18
    install -Dm755 "$srcdir"/libicui18n.so.70.1  "$libdir"/libicui18n.so.70.1
    install -Dm755 "$srcdir"/libicuuc.so.70.1    "$libdir"/libicuuc.so.70.1
    install -Dm755 "$srcdir"/libicudata.so.70.1  "$libdir"/libicudata.so.70.1
    ln -s libicui18n.so.70.1  "$libdir"/libicui18n.so.70
    ln -s libicuuc.so.70.1    "$libdir"/libicuuc.so.70
    ln -s libicudata.so.70.1  "$libdir"/libicudata.so.70
    install -Dm755 "$srcdir"/libxml2.so.2.9.13   "$libdir"/libxml2.so.2.9.13
    ln -s libxml2.so.2.9.13   "$libdir"/libxml2.so.2

    # pulseUI's RUNPATH omits /opt/pulsesecure/lib itself; register it with the system linker
    install -Dm644 /dev/stdin "$pkgdir"/etc/ld.so.conf.d/pulsesecure.conf <<< "/opt/pulsesecure/lib"
}
