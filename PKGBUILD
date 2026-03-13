# Maintainer: Gunther Schulz <https://github.com/Gunther-Schulz>
pkgname=display-mode-switcher
pkgver=1.0.0
pkgrel=1
pkgdesc="Switch dual-mode monitor display modes (5K↔2K) for gaming — toggle or Steam wrapper"
arch=('any')
url="https://github.com/Gunther-Schulz/display-mode-switcher"
license=('MIT')
depends=('ddcutil' 'mutter')
source=("https://github.com/Gunther-Schulz/display-mode-switcher/archive/refs/tags/v${pkgver}.tar.gz")
sha256sums=('SKIP')

package() {
  cd "$srcdir/$pkgname-${pkgver}"
  install -Dm755 display-mode-switcher "$pkgdir/usr/bin/display-mode-switcher"
  install -Dm644 README.md -t "$pkgdir/usr/share/doc/$pkgname/"
  install -Dm644 LICENSE -t "$pkgdir/usr/share/licenses/$pkgname/"
}
