# $Id$
# Maintainer: Angel Velasquez <angvp@archlinux.org>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Stéphane Gaudreault <stephane@archlinux.org>
# Contributor: Allan McRae <allan@archlinux.org>
# Contributor: Jason Chu <jason@archlinux.org>

pkgname=python-debug
pkgver=3.4.3
pkgrel=2
_pybasever=3.4
_pybuildver=3.4d
pkgdesc="Debug-friendly build of Python"
arch=('i686' 'x86_64')
license=('custom')
url="http://www.python.org/"
depends=('expat' 'bzip2' 'gdbm' 'openssl' 'libffi' 'zlib')
makedepends=('tk' 'sqlite' 'valgrind' 'bluez-libs' 'mpdecimal' 'hardening-wrapper')
checkdepends=('gdb')
optdepends=('python-setuptools'
            'python-pip'
            'sqlite'
            'mpdecimal: for decimal'
            'xz: for lzma'
            'tk: for tkinter')
options=('!makeflags')
provides=('python3', 'python')
replaces=('python3', 'python')
conflicts=('python')
source=(http://www.python.org/ftp/python/${pkgver%rc*}/Python-${pkgver}.tar.xz)
sha1sums=('7ca5cd664598bea96eec105aa6453223bb6b4456')

prepare() {
  cd Python-${pkgver}

  # FS#23997
  sed -i -e "s|^#.* /usr/local/bin/python|#!/usr/bin/python|" Lib/cgi.py

  # Ensure that we are using the system copy of various libraries (expat, zlib, libffi, and libmpdec),
  # rather than copies shipped in the tarball
  rm -r Modules/expat
  rm -r Modules/zlib
  rm -r Modules/_ctypes/{darwin,libffi}*
  rm -r Modules/_decimal/libmpdec
}

build() {
  cd Python-${pkgver}

  # Disable bundled pip & setuptools
  ./configure --prefix=/usr \
              --enable-shared \
              --with-threads \
              --with-computed-gotos \
              --enable-ipv6 \
              --with-system-expat \
              --with-dbmliborder=gdbm:ndbm \
              --with-system-ffi \
              --with-system-libmpdec \
              --enable-loadable-sqlite-extensions \
              --without-ensurepip \
              --with-pydebug

  make
}

check() {
  # test_site: http://bugs.python.org/issue21535

  cd Python-${pkgver}
  LD_LIBRARY_PATH="${srcdir}/Python-${pkgver}":${LD_LIBRARY_PATH} \
  TERM=screen \
    "${srcdir}/Python-${pkgver}/python" -m test.regrtest -uall -x test_site test_posixpath test_urllib2_localnet test_tk
}

package() {
  cd Python-${pkgver}
  make DESTDIR="${pkgdir}" install maninstall

  # Why are these not done by default...
  ln -s python3               "${pkgdir}"/usr/bin/python
  ln -s python3-config        "${pkgdir}"/usr/bin/python-config
  ln -s idle3                 "${pkgdir}"/usr/bin/idle
  ln -s pydoc3                "${pkgdir}"/usr/bin/pydoc
  ln -s python${_pybasever}.1 "${pkgdir}"/usr/share/man/man1/python.1

  # Fix FS#22552
  ln -sf ../../libpython${_pybuildver}m.so \
    "${pkgdir}/usr/lib/python${_pybasever}/config-${_pybuildver}m/libpython${_pybuildver}m.so"

  # Clean-up reference to build directory
  sed -i "s|$srcdir/Python-${pkgver}:||" "$pkgdir/usr/lib/python${_pybasever}/config-${_pybuildver}m/Makefile"

  # License
  install -Dm644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
