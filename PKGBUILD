# Maintainer:  Tk-Glitch <ti3nou@gmail.com>

_pkgbase=gamescope
pkgname=${_pkgbase}-git
pkgver=3.6.1.r0.gde16a27
pkgrel=1
_where=$PWD # track basedir as different Arch based distros are moving srcdir around
source "$_where"/customization.cfg

# Load external configuration file if present. Available variable values will overwrite customization.cfg ones.
if [ -e "$_EXT_CONFIG_PATH" ]; then
  source "$_EXT_CONFIG_PATH" && msg2 "External configuration file $_EXT_CONFIG_PATH will be used to override customization.cfg values.\n"
fi

arch=('x86_64')
url="https://github.com/Plagman/gamescope"
license=('BSD 2-Clause "Simplified" License')
pkgdesc="gamescope: the micro-compositor formerly known as steamcompmgr"

makedepends=('git' 'meson' 'ninja' 'cmake' 'pixman' 'pkgconf' 'vulkan-headers' 'wayland-protocols>=1.17')
depends=(wayland opengl-driver xorg-server-xwayland libdrm libinput libxkbcommon libxcomposite libcap libxcb libpng glslang libxrender libxtst vulkan-icd-loader sdl2)
conflicts=('gamescope')

# custom commit to pass to git
if [ -n "$_gamescope_commit" ]; then
  _gamescope_commit="#commit=${_gamescope_commit}"
fi

source=("git+https://github.com/Plagman/gamescope.git${_gamescope_commit}")
md5sums=('SKIP')
sha512sums=('SKIP')
options=('staticlibs')

pkgver() {
    cd ${_pkgbase}
    git describe --long --tags --always | sed 's/\([^-]*-g\)/r\1/;s/-/./g;s/^v//'
}

prepare() {
    if [  -d _build ]; then
        rm -rf _build
    fi
    mkdir _build
}

build() {
    cd ${_pkgbase}

    meson \
      --buildtype release \
      --prefix /usr \
      ${srcdir}/_build
}

package() {
    DESTDIR="$pkgdir" ninja -C _build install
     
    provides=(gamescope=$pkgver)
     
    msg "Removing unnecessary wlroots files"
    rm -rfv "${pkgdir}"/usr/include
    rm -rfv "${pkgdir}"/usr/lib/libwlroots*
    rm -fv  "${pkgdir}"/usr/lib/pkgconfig/wlroots.pc

    install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 "${srcdir}/${_pkgbase}/LICENSE"
}
