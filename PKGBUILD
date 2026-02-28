# Maintainer:  Tk-Glitch <ti3nou@gmail.com>

_pkgbase=gamescope
pkgname=${_pkgbase}-git
pkgver=3.16.20.r4.g2f9d9e3e
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
depends=(wayland opengl-driver xorg-server-xwayland pipewire libdrm libinput libavif libxkbcommon libxcomposite libxmu libcap libxcb libpng glslang libxrender libxtst libxres vulkan-icd-loader sdl2 xcb-util-renderutil xcb-util-wm seatd benchmark)
conflicts=('gamescope')

# custom commit to pass to git
if [ -n "$_gamescope_commit" ]; then
  _gamescope_commit="#commit=${_gamescope_commit}"
fi

source=("git+https://github.com/ValveSoftware/gamescope.git${_gamescope_commit}"
        'git+https://github.com/Joshua-Ashton/wlroots.git'
        'git+https://gitlab.freedesktop.org/emersion/libliftoff.git'
        'git+https://github.com/Joshua-Ashton/vkroots.git'
        'git+https://gitlab.freedesktop.org/emersion/libdisplay-info.git'
        'git+https://github.com/ValveSoftware/openvr.git'
        'git+https://github.com/Joshua-Ashton/reshade.git'
        'git+https://github.com/Joshua-Ashton/GamescopeShaders.git#tag=v0.1'
        'git+https://github.com/KhronosGroup/SPIRV-Headers.git')
md5sums=('SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP'
         'SKIP')
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
    export CMAKE_POLICY_VERSION_MINIMUM=3.5

    cd ${_pkgbase}

    meson subprojects download

    git submodule init subprojects/wlroots
    git config submodule.subprojects/wlroots.url ../wlroots

    git submodule init subprojects/libliftoff
    git config submodule.subprojects/libliftoff.url ../libliftoff

    git submodule init subprojects/vkroots
    git config submodule.subprojects/vkroots.url ../vkroots

    git submodule init subprojects/libdisplay-info
    git config submodule.subprojects/libdisplay-info.url ../libdisplay-info

    git submodule init subprojects/openvr
    git config submodule.subprojects/openvr.url ../openvr

    git submodule init src/reshade
    git config submodule.src/reshade.url ../reshade

    git submodule init thirdparty/SPIRV-Headers
    git config submodule.thirdparty/SPIRV-Headers.url ../SPIRV-Headers

    git -c protocol.file.allow=always submodule update

    # Workaround for erroneous libdisplay-info submodule in the tree
    #( cd subprojects/libdisplay-info && git checkout 92b031749c0fe84ef5cdf895067b84a829920e25  )

    # Use Arch's libdisplay-info
    #rm -rf subprojects/libdisplay-info

    # user patches
    #cd ${_pkgbase}
    _userpatch_target="gamescope"
    _userpatch_ext="mygamescope"
    user_patcher
    #cd "$_where"

    meson \
      --buildtype release \
      -Dforce_fallback_for=stb,wlroots,vkroots,libliftoff,glm,libdisplay-info \
      -Dpipewire=enabled \
      ${srcdir}/_build
}

package() {
    install -d "$pkgdir"/usr/share/gamescope/reshade
    cp -r "$srcdir"/GamescopeShaders/* "$pkgdir"/usr/share/gamescope/reshade/
    chmod -R 755 "$pkgdir"/usr/share/gamescope

    meson install -C _build --skip-subprojects --destdir="${pkgdir}"

    install -Dm 644 "${srcdir}/$_pkgbase/LICENSE" -t "${pkgdir}/usr/share/licenses/$_pkgbase/"
}

trap exit_cleanup EXIT
