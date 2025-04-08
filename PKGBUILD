# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>
# Contributor: Dan Johansen <strit@manjaro.org>
# Contributor: Halhadus <eneshakans45@proton.me>

# ALARM: Kevin Mihelich <kevin@archlinuxarm.org>
#  - Removed Gallium3D drivers/packages for chipsets that don't exist in our ARM devices (intel, VMware svga).
#  - added broadcom and panfrost vulkan packages
#  - enable lto for aarch64
#  ALARM: John Audia <therealgraysky@proton.me>
#  - added freedreno driver, PR#1973 and PR#1996
# Turnip: Halhadus <eneshakans45@proton.me>
# - Removed every other not needed Gallium3D drivers/packages
# - Added KGSL support for freedreno vulkan driver and Termux-X11 support

highmem=1

pkgbase=mesa
pkgname=(
  mesa
  opencl-clover-mesa
  opencl-rusticl-mesa
  vulkan-mesa-layers
  vulkan-swrast
  vulkan-virtio
  vulkan-freedreno
)
pkgver=24.2.7
pkgrel=1
epoch=1
pkgdesc="Open-source OpenGL drivers"
url="https://www.mesa3d.org/"
arch=(aarch64)
license=("MIT AND BSD-3-Clause AND SGI-B-2.0")
makedepends=(
  clang
  expat
  gcc-libs
  glibc
  libdrm
  libelf
  libglvnd
  libva
  libvdpau
  libx11
  libxcb
  libxext
  libxfixes
  libxml2
  libxrandr
  libxshmfence
  libxxf86vm
  llvm
  llvm-libs
  lm_sensors
  rust
  spirv-llvm-translator
  spirv-tools
  systemd-libs
  vulkan-icd-loader
  wayland
  xcb-util-keysyms
  zlib
  zstd

  # shared between mesa and lib32-mesa
  cbindgen
  clang
  cmake
  elfutils
  glslang
  libclc
  meson
  python-mako
  python-packaging
  python-ply
  python-yaml
  rust-bindgen
  wayland-protocols
  xorgproto

  # valgrind deps
  valgrind

  # d3d12 deps
  directx-headers
)
options=(
  # GCC 14 LTO causes segfault in LLVM under si_llvm_optimize_module
  # https://gitlab.freedesktop.org/mesa/mesa/-/issues/11140
  #
  # In general, upstream considers LTO to be broken until explicit notice.
  !lto
)
source=(
  "https://mesa.freedesktop.org/archive/mesa-$pkgver.tar.xz"{,.sig}
  'https://raw.githubusercontent.com/Halhadus/alarm-mesa-turnip/refs/heads/main/turnip.patch'
)

b2sums=('eb1b0285e14e77c3140275b322ff084fca74a1048e6df38f4b14cb03ed7fc436897f7b33d107d1e262d9d4944229fb1e85d02e731c645ead5a7b269dec9334b7'
        'SKIP'
	'a33072c7a24174d87283e47ee38ab10adaec8d128c68b426794cec10c234bd50662275782bed5f1ab42f04788658d3f3b54f0f04ac588d82ec4f5215a77f1a26')

# https://docs.mesa3d.org/relnotes.html
sha256sums=('a0ce37228679647268a83b3652d859dcf23d6f6430d751489d4464f6de6459fd'
            'SKIP'
            'fd8d9a1fd11ff987b0b4e7f9800a1a8d9e37e9bcc4052804ef6732f75b744ef8')

prepare() {
  cd mesa-$pkgver
  # Apply patches
  patch -p1 < "$srcdir/turnip.patch"

  # Include package release in version string so Chromium invalidates
  # its GPU cache; otherwise it can cause pages to render incorrectly.
  # https://bugs.launchpad.net/ubuntu/+source/chromium-browser/+bug/2020604
  echo "$pkgver-arch$epoch.$pkgrel" >VERSION
}

build() {

  local meson_options=(
    -D android-libbacktrace=disabled
    #-D b_lto=$([[ $CARCH == aarch64 ]] && echo true || echo false)
    -D b_ndebug=true
    -D gallium-drivers=freedreno,llvmpipe,softpipe,virgl,zink,d3d12${GALLIUM}
    -D gallium-extra-hud=true
    -D gallium-nine=true
    -D gallium-opencl=icd
    -D gallium-rusticl=true
    -D gallium-xa=disabled
    -D gallium-omx=disabled
    -D freedreno-kmds=kgsl
    -D gles1=disabled
    -D glx=dri
    -D intel-rt=disabled
    -D libunwind=disabled
    -D microsoft-clc=disabled
    -D osmesa=true
    -D platforms=x11,wayland
    -D rust_std=2021
    -D valgrind=enabled
    -D video-codecs=all
    -D vulkan-drivers=swrast,virtio,freedreno
    -D vulkan-layers=device-select,overlay
  )

  # Build only minimal debug info to reduce size
  CFLAGS+=" -g1"
  CXXFLAGS+=" -g1"

  # Inject subproject packages
  export MESON_PACKAGE_CACHE_DIR="$srcdir"

  arch-meson mesa-$pkgver build "${meson_options[@]}"
  meson compile -C build
}

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv -v "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

package_mesa() {
  depends=(
    expat
    gcc-libs
    glibc
    libdrm
    libelf
    libglvnd
    libx11
    libxcb
    libxext
    libxfixes
    libxshmfence
    libxxf86vm
    llvm-libs
    lm_sensors
    wayland
    zlib
    zstd
  )
  optdepends=("opengl-man-pages: for the OpenGL API man pages")
  provides=(
    "libva-mesa-driver=$epoch:$pkgver-$pkgrel"
    "mesa-libgl=$epoch:$pkgver-$pkgrel"
    "mesa-vdpau=$epoch:$pkgver-$pkgrel"
    libva-driver
    opengl-driver
    vdpau-driver
    vulkan-intel
    vulkan-radeon
    vulkan-nouveau
    vulkan-panfrost
    vulkan-broadcom
    vulkan-gfxstream
    vulkan-dzn
    mesa-docs
  )
  conflicts=(
    'libva-mesa-driver<1:24.2.7-1'
    'mesa-libgl<17.0.1-2'
    'mesa-vdpau<1:24.2.7-1'
    'vulkan-intel'
    'vulkan-radeon'
    'vulkan-nouveau'
    'vulkan-panfrost'
    'vulkan-broadcom'
    'vulkan-gfxstream'
    'vulkan-dzn'
    'mesa-docs'
  )
  replaces=(
    'libva-mesa-driver<1:24.2.7-1'
    'mesa-libgl<17.0.1-2'
    'mesa-vdpau<1:24.2.7-1'
    'vulkan-intel'
    'vulkan-radeon'
    'vulkan-nouveau'
    'vulkan-panfrost'
    'vulkan-broadcom'
    'vulkan-gfxstream'
    'vulkan-dzn'
    'mesa-docs'
  )

  meson install -C build --destdir "$pkgdir"

  (
    local libdir=usr/lib icddir=usr/share/vulkan/icd.d

    cd "$pkgdir"

    _pick clover $libdir/gallium-pipe
    _pick clover $libdir/libMesaOpenCL*
    _pick clover etc/OpenCL/vendors/mesa.icd

    _pick clrust $libdir/libRusticlOpenCL*
    _pick clrust etc/OpenCL/vendors/rusticl.icd

    _pick vklayer $libdir/libVkLayer_*.so
    _pick vklayer usr/bin/mesa-overlay-control.py
    _pick vklayer usr/share/vulkan/{ex,im}plicit_layer.d

    _pick vkswrast $icddir/lvp_icd*.json
    _pick vkswrast $libdir/libvulkan_lvp.so

    _pick vkvirtio $icddir/virtio_icd*.json
    _pick vkvirtio $libdir/libvulkan_virtio.so

    _pick vkfreedreno $icddir/freedreno_icd*.json
    _pick vkfreedreno $libdir/libvulkan_freedreno.so

    # indirect rendering
    ln -sr $libdir/libGLX_{mesa,indirect}.so.0
  )

}

package_opencl-clover-mesa() {
  pkgdesc="Open-source OpenCL drivers - Clover variant"
  depends=(
    clang
    expat
    gcc-libs
    glibc
    libdrm
    libelf
    llvm-libs
    spirv-llvm-translator
    spirv-tools
    zlib
    zstd

    libclc
  )
  optdepends=("opencl-headers: headers necessary for OpenCL development")
  provides=(opencl-driver)
  replaces=("opencl-mesa<=23.1.4-1")
  conflicts=(opencl-mesa)

  mv clover/* "$pkgdir"

}

package_opencl-rusticl-mesa() {
  pkgdesc="Open-source OpenCL drivers - RustICL variant"
  depends=(
    clang
    expat
    gcc-libs
    glibc
    libdrm
    libelf
    llvm-libs
    spirv-llvm-translator
    spirv-tools
    zlib
    zstd

    libclc
  )
  optdepends=("opencl-headers: headers necessary for OpenCL development")
  provides=(opencl-driver)
  replaces=("opencl-mesa<=23.1.4-1")
  conflicts=(opencl-mesa)

  mv clrust/* "$pkgdir"

}

package_vulkan-mesa-layers() {
  pkgdesc="Mesa's Vulkan layers"
  depends=(
    gcc-libs
    glibc
    libdrm
    libxcb
    wayland

    python
  )
  conflicts=(vulkan-mesa-layer)
  replaces=(vulkan-mesa-layer)

  mv vklayer/* "$pkgdir"

}

package_vulkan-swrast() {
  pkgdesc="Open-source Vulkan driver for CPUs (Software Rasterizer)"
  depends=(
    expat
    gcc-libs
    glibc
    libdrm
    libx11
    libxcb
    libxshmfence
    llvm-libs
    systemd-libs
    vulkan-icd-loader
    wayland
    xcb-util-keysyms
    zlib
    zstd
  )
  optdepends=("vulkan-mesa-layers: additional vulkan layers")
  conflicts=(vulkan-mesa)
  replaces=(vulkan-mesa)
  provides=(vulkan-driver)

  mv vkswrast/* "$pkgdir"

}

package_vulkan-virtio() {
  pkgdesc="Open-source Vulkan driver for Virtio-GPU (Venus)"
  depends=(
    expat
    gcc-libs
    glibc
    libdrm
    libx11
    libxcb
    libxshmfence
    systemd-libs
    vulkan-icd-loader
    wayland
    xcb-util-keysyms
    zlib
    zstd
  )
  optdepends=("vulkan-mesa-layers: additional vulkan layers")
  provides=(vulkan-driver)

  mv vkvirtio/* "$pkgdir"

}

package_vulkan-freedreno() {
  pkgdesc="Freedreno Vulkan mesa driver"
  depends=(
    wayland
    libx11
    libxshmfence
    libdrm
  )
  optdepends=("vulkan-mesa-layers: additional vulkan layers")
  provides=(vulkan-driver)

  mv vkfreedreno/* "$pkgdir"

}

# vim:set sw=2 sts=-1 et:
