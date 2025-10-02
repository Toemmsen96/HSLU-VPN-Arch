# Maintainer: Toemmsen96 <147745097+Toemmsen96@users.noreply.github.com>
DLAGENTS=("https::/usr/bin/curl -k -o %o %u")

# Function to get the latest rpm file (excluding hslu-ps.rpm)
_get_latest_rpm() {
    ls -t ps-pulse-linux-*.rpm 2>/dev/null | head -n1
}

# Function to get corresponding sha256 file
_get_sha256_file() {
    local rpm_file="$1"
    echo "${rpm_file%.rpm}.sha256"
}

# Get the latest rpm file
_latest_rpm=$(_get_latest_rpm)
_sha256_file=$(_get_sha256_file "$_latest_rpm")

pkgname=hslu-pulsesecure
pkgver=0.0.1
pkgrel=1
pkgdesc='Ivanti Secure Access Client for HSLU'
arch=(x86_64)
license=(custom)
url='https://www.pulsesecure.net/'
depends=(gcc-libs libgnome-keyring openssl curl dbus libbsd dmidecode gtkmm3 webkit2gtk psmisc)
install=${pkgname}.install
source=(EULA.txt)
source_x86_64=("$_latest_rpm")
sha256sums=('823b298db462d9963148888287ebde2eb013c9b7d0635f3893d18a35776fa8c3')
sha256sums_x86_64=('SKIP')  # Will be verified manually in prepare()
conflicts=(pulse-connect-secure pulse-secure)

prepare() {
    # Verify SHA256 checksum
    if [[ -f "$_sha256_file" ]]; then
        msg "Verifying SHA256 checksum for $_latest_rpm"
        local expected_hash=$(cut -d' ' -f1 "$_sha256_file")
        local actual_hash=$(sha256sum "$_latest_rpm" | cut -d' ' -f1)
        
        if [[ "$expected_hash" != "$actual_hash" ]]; then
            error "SHA256 checksum verification failed!"
            error "Expected: $expected_hash"
            error "Actual:   $actual_hash"
            return 1
        fi
        msg "SHA256 checksum verification passed"
    else
        warning "SHA256 file not found: $_sha256_file"
    fi
    
    # Extract the rpm file
    cd "$srcdir"
    if command -v bsdtar >/dev/null 2>&1; then
        bsdtar -xf "$_latest_rpm"
    elif command -v rpm2cpio >/dev/null 2>&1; then
        rpm2cpio "$_latest_rpm" | cpio -idmv
    else
        error "Neither bsdtar nor rpm2cpio found. Cannot extract RPM file."
        return 1
    fi
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
}
