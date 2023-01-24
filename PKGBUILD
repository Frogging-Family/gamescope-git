# Maintainer:  Tk-Glitch <ti3nou@gmail.com>

_pkgbase=gamescope
pkgname=${_pkgbase}-git
pkgver=3.11.51.r120.gd6c1df4
pkgrel=1
_where="$PWD" # track basedir as different Arch based distros are moving srcdir around
if [ -e "$_where"/customization.cfg ]; then
  source "$_where"/customization.cfg
fi

# Load external configuration file if present. Available variable values will overwrite customization.cfg ones.
if [ -e "$_EXT_CONFIG_PATH" ]; then
  source "$_EXT_CONFIG_PATH" && msg2 "External configuration file $_EXT_CONFIG_PATH will be used to override customization.cfg values.\n"
fi

arch=('x86_64')
url="https://github.com/Plagman/gamescope"
license=('BSD 2-Clause "Simplified" License')
pkgdesc="gamescope: the micro-compositor formerly known as steamcompmgr"

exit_cleanup() {
  # Prevent subproject conflicts
  rm -rf "$_where/src/$_pkgbase"
  #rm -rf "$_where"/*.mygamescopepatch

  remove_deps

  msg2 "Cleanup done"
}

makedepends=('git' 'meson' 'ninja' 'cmake' 'pixman' 'pkgconf' 'vulkan-headers' 'wayland-protocols>=1.17')
depends=(wayland opengl-driver xorg-server-xwayland pipewire libdrm libinput libxkbcommon libxcomposite libxmu libcap libxcb libpng glslang libxrender libxtst libxres vulkan-icd-loader sdl2 xcb-util-renderutil xcb-util-wm seatd)
conflicts=('gamescope')

# custom commit to pass to git
if [ -n "$_gamescope_commit" ]; then
  _gamescope_commit="#commit=${_gamescope_commit}"
fi

source=("git+https://github.com/Plagman/gamescope.git${_gamescope_commit}")
md5sums=('SKIP')
sha512sums=('SKIP')
options=('staticlibs')

user_patcher() {
	# To patch the user because all your base are belong to us
	local _patches=("$_where"/*."${_userpatch_ext}revert")
	if [ ${#_patches[@]} -ge 2 ] || [ -e "${_patches}" ]; then
	  if [ "$_user_patches_no_confirm" != "true" ]; then
	    msg2 "Found ${#_patches[@]} 'to revert' userpatches for ${_userpatch_target}:"
	    printf '%s\n' "${_patches[@]}"
	    read -rp "Do you want to install it/them? - Be careful with that ;)"$'\n> N/y : ' _CONDITION;
	  fi
	  if [ "$_CONDITION" == "y" ] || [ "$_user_patches_no_confirm" == "true" ]; then
	    for _f in "${_patches[@]}"; do
	      if [ -e "${_f}" ]; then
	        msg2 "######################################################"
	        msg2 ""
	        msg2 "Reverting your own ${_userpatch_target} patch ${_f}"
	        msg2 ""
	        msg2 "######################################################"
	        patch -Np1 -R < "${_f}"
	        echo "Reverted your own patch ${_f}" >> "$_where"/last_build_config.log
	      fi
	    done
	  fi
	fi

	_patches=("$_where"/*."${_userpatch_ext}patch")
	if [ ${#_patches[@]} -ge 2 ] || [ -e "${_patches}" ]; then
	  if [ "$_user_patches_no_confirm" != "true" ]; then
	    msg2 "Found ${#_patches[@]} userpatches for ${_userpatch_target}:"
	    printf '%s\n' "${_patches[@]}"
	    read -rp "Do you want to install it/them? - Be careful with that ;)"$'\n> N/y : ' _CONDITION;
	  fi
	  if [ "$_CONDITION" == "y" ] || [ "$_user_patches_no_confirm" == "true" ]; then
	    for _f in "${_patches[@]}"; do
	      if [ -e "${_f}" ]; then
	        msg2 "######################################################"
	        msg2 ""
	        msg2 "Applying your own ${_userpatch_target} patch ${_f}"
	        msg2 ""
	        msg2 "######################################################"
	        patch -Np1 < "${_f}"
	        echo "Applied your own patch ${_f}" >> "$_where"/last_build_config.log
	      fi
	    done
	  fi
	fi
}

pkgver() {
    cd ${_pkgbase}
    git describe --long --tags --always | sed 's/\([^-]*-g\)/r\1/;s/-/./g;s/^v//'
}

prepare() {
    if [  -d _build ]; then
        rm -rf _build
    fi
    mkdir _build

    ( cd "${_pkgbase}" && git reset --hard HEAD && git clean -xdf )
}

build() {
    cd ${_pkgbase}
    git submodule update --init --recursive

    # user patches
    #cd ${_pkgbase}
    _userpatch_target="gamescope"
    _userpatch_ext="mygamescope"
    user_patcher
    #cd "$_where"

    meson \
      --buildtype release \
      --force-fallback-for=vkroots,wlroots,libliftoff,stb,libdisplay-info \
      -Dpipewire=enabled \
      -Dwlroots:backends=drm,libinput,x11 \
      -Dwlroots:renderers=gles2,vulkan \
      --prefix /usr \
      ${srcdir}/_build
}

package() {
    DESTDIR="$pkgdir" ninja -C _build install
     
    provides=(gamescope=$pkgver)
     
    msg2 "Removing unnecessary wlroots files"
    rm -rfv "${pkgdir}"/usr/include
    rm -rfv "${pkgdir}"/usr/lib/libwlroots*
    rm -fv  "${pkgdir}"/usr/lib/pkgconfig/wlroots.pc

    install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 "${srcdir}/${_pkgbase}/LICENSE"
}

trap exit_cleanup EXIT
