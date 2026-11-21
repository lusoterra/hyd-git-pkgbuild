# Maintainer: Lev <lev@lusosterone.net>
pkgname=hyd-git
pkgver=0.2.0.r0.g0000000
pkgrel=1
pkgdesc="Read-only C++ mirror of the Hydrus Public Tag Repository with a Client API subset"
arch=('x86_64' 'aarch64')
url="https://github.com/lusoterra/hyd"
license=('AGPL-3.0-or-later')
depends=('curl' 'openssl' 'zlib' 'sqlite' 'nlohmann-json' 'simdjson')
makedepends=('git' 'cmake' 'ninja' 'python')
backup=('etc/hydd.conf')
install=hydd.install
source=("$pkgname::git+https://github.com/lusoterra/hyd.git")
sha256sums=('SKIP')

pkgver() {
    cd "$pkgname"
    local version
    version=$(sed -n 's/^project(hyd VERSION \([0-9.]*\).*/\1/p' CMakeLists.txt)
    printf "%s.r%s.%s" "$version" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
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

    # Source files use .sysusers/.tmpfiles, but systemd only reads *.conf.
    install -Dm644 hyd.sysusers "$pkgdir/usr/lib/sysusers.d/hyd.conf"
    install -Dm644 hyd.tmpfiles "$pkgdir/usr/lib/tmpfiles.d/hyd.conf"
}
