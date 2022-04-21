# Maintainer: xaque <xaque at duck dot com>

pkgname=o3de-nightly-bin
# <Stable release>.<Build date>
_stablever=2111.2
pkgver=${_stablever}_20220217
_engver=0.0.0.0
pkgrel=1
pkgdesc='Open 3D Engine - An open-source, real-time 3D development engine (Nightly build)'
arch=('x86_64')
license=('APACHE' "MIT")
url='https://o3de.org/'
depends=('clang' 'cmake' 'curl' 'fontconfig' 'gcc-libs' 'glibc' 'glu' 'libffi7' 'libglvnd' 'libxau' 'libx11' 'libxcb' 'libxcrypt-compat' 'libxkbcommon' 'libxkbcommon-x11' 'mesa' 'openssl' 'sdl2' 'zlib')
optdepends=('ninja: Support for multiple build configurations per project')
makedepends=('icoutils')
options=('!strip')
provides=('o3de-nightly')
install="${pkgname}.install"
source=("open-3d-engine-nightly.desktop"
        "https://o3debinaries.org/development/Latest/Linux/O3DE_latest.deb"
        "https://o3debinaries.org/development/Latest/Linux/O3DE_latest.deb.sha256"
        "https://o3debinaries.org/main/Latest/Linux/o3de-releases.gpg"
        "LICENSE.txt::https://raw.githubusercontent.com/o3de/o3de/development/LICENSE.txt"
        "LICENSE_MIT.txt::https://raw.githubusercontent.com/o3de/o3de/development/LICENSE_MIT.TXT"
        "LICENSE_APACHE2.txt::https://raw.githubusercontent.com/o3de/o3de/development/LICENSE_APACHE2.TXT")
sha256sums=('SKIP'
            'SKIP'
            'SKIP'
            'f27d4324d7fe38ed228e4e0218d5e988ecaf73e550210df4b897f99146def037'
            'SKIP'
            'SKIP'
            'SKIP')

pkgver() {
    # Look at modified date of gpg signature to determine build date
    echo ${_stablever}_$(date -r ${srcdir}/_gpgbuilder "+%Y%m%d")
}

prepare() {
    echo -n "    Verifying checksum for O3DE_latest.deb ..."
    sha256sum -c O3DE_latest.deb.sha256 >/dev/null
    echo " Passed"

    echo -n "    Verifying PGP for O3DE_latest.deb ..."
    gpgv --keyring "./o3de-releases.gpg" "O3DE_latest.deb" >/dev/null 2>&1
    echo " Passed"
}

package() {
    echo -n "    Extracting data to /opt/O3DE ."
    tar -xzf data.tar.gz -C "${pkgdir}" --checkpoint=.50000
    echo " Done"

    if [ ! -d "${pkgdir}/opt/O3DE/${_engver}" ]; then
        echo "Expected O3DE ${_engver}. PKGBUILD may need to be updated for modified paths with a new major engine version. Aborting." 1>&2
        exit 1
    fi

    # Trying to create new project fails if launcher doesn't find clang-12
    # Force use of system clang with local symlink in PATH
    mkdir -p "${pkgdir}"/opt/O3DE/${_engver}/symbin
    ln -s $(which clang) "${pkgdir}"/opt/O3DE/${_engver}/symbin/clang-12
    ln -s $(which clang++) "${pkgdir}"/opt/O3DE/${_engver}/symbin/clang++-12
    ln -s $(which clang++) "${pkgdir}"/opt/O3DE/${_engver}/symbin/clang++-13

    # Script in /usr/bin to run o3de with modified env
    mkdir -p "${pkgdir}/usr/bin"
    echo '#!/bin/sh' >"${pkgdir}/usr/bin/o3de-nightly"
    echo "PATH=\""'$PATH'":/opt/O3DE/${_engver}/symbin\" CC=/opt/O3DE/${_engver}/symbin/clang-12 CXX=/opt/O3DE/${_engver}/symbin/clang++-12 /opt/O3DE/${_engver}/bin/Linux/profile/Default/o3de" >>"${pkgdir}/usr/bin/o3de-nightly"
    chmod +x "${pkgdir}/usr/bin/o3de-nightly"

    # Extract .ico and install icons
    icotool -x "${pkgdir}"/opt/O3DE/${_engver}/cmake/Platform/Windows/Packaging/product_icon.ico -o .
    iter=1
    for size in 256 128 64 48 32 16; do
        install -Dm644 "product_icon_${iter}_${size}x${size}x32.png" \
            "${pkgdir}/usr/share/icons/hicolor/${size}x${size}/apps/o3de-nightly.png"
        ((iter++))
    done

    # Install desktop file
    install -Dm644 open-3d-engine-nightly.desktop "${pkgdir}"/usr/share/applications/open-3d-engine-nightly.desktop

    # Install license files
    install -Dm644 LICENSE.txt "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE.txt"
    install -Dm644 LICENSE_MIT.txt "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE_MIT.txt"
    install -Dm644 LICENSE_APACHE2.txt "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE_APACHE2.txt"

    # Fix warning for mismatched /opt permissions
    chmod --reference /opt "${pkgdir}"/opt
}
