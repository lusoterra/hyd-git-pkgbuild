# Maintainer: Lev <lev@lusosterone.net>
pkgname=hyd-git
pkgver=0.7.1.r43.a6137b6
pkgrel=1
pkgdesc="Read-only C++ mirror of the Hydrus Public Tag Repository with a Client API subset"
arch=('x86_64' 'aarch64')
url="https://github.com/lusoterra/hyd"
license=('AGPL-3.0-or-later')
depends=('curl' 'openssl' 'zlib' 'nlohmann-json' 'simdjson' 'bash')
makedepends=('git' 'cmake' 'ninja' 'python')
conflicts=("hyd" "hyd-git")
backup=('etc/hydd.conf')
install=hydd.install
source=("$pkgname::git+https://github.com/lusoterra/hyd.git"
        'hyd.tmpfiles'
        'hydd-perms.conf')
sha256sums=('SKIP'
            '4be8c96584f6d662d4109e56079a5f688844faf8909f2dfb76c98fd01eb52786'
            '0a6978b457b14a104981634b421d718d1e2e23d464303ea63cc696014124ff56')


pkgver() {
    cd "$pkgname"
    local version
    version=$(sed -n 's/^project(hyd VERSION \([0-9.]*\).*/\1/p' CMakeLists.txt)
    printf "%s.r%s.%s" "$version" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

prepare() {
    cd "$pkgname"
    # Logs contain no credentials (repository key is logged as "(set)", access
    # key is never printed), but upstream hardcodes 0640 and re-applies it on
    # every open/rotation. Relax to 0644 until log.cpp is fixed upstream.
    sed -i 's/\b0640\b/0644/g' src/common/log.cpp
}

build() {
    cmake -S "$pkgname" -B build -G Ninja \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DCMAKE_INSTALL_PREFIX=/usr
    cmake --build build -j "$(nproc)"
}

check() {
    ctest --test-dir build --output-on-failure
}

package() {
    DESTDIR="$pkgdir" cmake --install build --prefix /usr
    install -Dm644 "$pkgname/LICENSE" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

    # Upstream CMake already installs sysusers.d/hyd.conf and a 0750
    # tmpfiles.d/hyd.conf; override the latter with 0755 dirs.
    install -Dm644 hyd.tmpfiles "$pkgdir/usr/lib/tmpfiles.d/hyd.conf"
    # Upstream unit uses StateDirectoryMode/LogsDirectoryMode=0750 and UMask=0077.
    install -Dm644 hydd-perms.conf \
        "$pkgdir/usr/lib/systemd/system/hydd.service.d/perms.conf"
}
