# Maintainer: Andrés Merlo <a.merlo.truji10@gmail.com>

pkgname=ttf-atkinson-hyperlegible-next-nerd-git
pkgdesc="Patched font Atkinson Hyperlegible Next (2024) from nerd fonts library. Packaged by Andrés Merlo Trujillo only for local use!"
pkgver=r17.7925f50
pkgrel=1
arch=('any')
license=('OFL-1.1')
# options=('!debug' '!strip')
source=("${pkgname}::git+https://github.com/googlefonts/atkinson-hyperlegible-next.git")
sha256sums=('SKIP')
# noextract=("${source[@]%%::*}")
makedepends=('font-patcher')

pkgver() {
  cd ${pkgname}

  printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
}

build() {
  cd ${pkgname}

  mkdir output

  find fonts/ttf -iname "*.ttf" -print0 | xargs -I {} -0 -P $(nproc) fontforge -script /usr/share/font-patcher/font-patcher -q --outputdir "output/" --complete --careful --makegroups 5 --metrics TYPO "{}"
}

package() {
  cd ${pkgname}

  install -Dm644 output/*.ttf -t "${pkgdir}/usr/share/fonts/TTF"
  install -Dm644 OFL.txt -t "${pkgdir}/usr/share/licenses/${pkgname}/"
}
