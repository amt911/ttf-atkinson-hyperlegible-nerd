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

  # Put the latin i back (Atkynson -> Atkinson): upstream's OFL copyright line
  # declares NO Reserved Font Name, so a modified version may keep the name —
  # the patcher's blanket rename (FontnameTools.py) is caution this font does
  # not require.
  fontforge -lang=py -c "
import sys, fontforge
for path in sys.argv[1:]:
    f = fontforge.open(path)
    fix = lambda s: s.replace('Atkynson', 'Atkinson') if s else s
    f.fontname, f.familyname, f.fullname = fix(f.fontname), fix(f.familyname), fix(f.fullname)
    f.sfnt_names = tuple((l, k, fix(v)) for l, k, v in f.sfnt_names)
    f.generate(path)
    f.close()
" output/*.ttf
  for f in output/*Atkynson*; do [ -e "$f" ] || continue; mv "$f" "${f//Atkynson/Atkinson}"; done
}

package() {
  cd ${pkgname}

  install -Dm644 output/*.ttf -t "${pkgdir}/usr/share/fonts/TTF"
  install -Dm644 OFL.txt -t "${pkgdir}/usr/share/licenses/${pkgname}/"
}
