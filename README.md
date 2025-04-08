# Mesa 24.2.7 with Turnip+Termux-X11 Patches &amp; more PKGBUILD for ArchLinuxArm

For some reason after Mesa 24.3, with Turnip+zink+Termux-X11 combination can't use EGL/GLES. I created a PKGBUILD file (based ALARM's PKGBUILD file) which patching Mesa 24.2.7 source with dri3 patches (thanks MastaG) and build only some needed libraries.

I'm too lazy to add sign things, please build with --skippgpcheck flag.

Also I don't know how can I use other files which included in same repo with PKGBUILD, so it's getting patches from Github(before it was getting file from my website heheh).

Also idk which licence should I add. Create an issue if you know it.

Don't forget add this packages to hold list after build & install packages:

/etc/pacman.conf
```sh
HoldPkg = ... mesa mesa-amber mesa-docs opencl-clover-mesa opencl-rusticl-mesa vulkan-mesa-layers vulkan-broadcom vulkan-freedreno vulkan-gfxstream vulkan-nouveau vulkan-panfrost vulkan-radeon vulkan-swrast vulkan-virtio vullan-dzn ...
```

PKGBUILD source:
https://github.com/archlinuxarm/PKGBUILDs/commit/c1c2776b762fed5173bf3d900ee9237062400f16

Patches(I mixed files):
https://github.com/MastaG/mesa-turnip-ppa/tree/main/turnip-patches
