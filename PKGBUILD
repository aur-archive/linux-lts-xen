# Maintainer: Phillip Smith <fukawi2@NO-SPAM.gmail.com>
# http://github.com/fukawi2/aur-packages
# Maintainer: Si Feng <sci.feng at gmail.com>
# Maintainer: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

pkgbase=linux-lts-xen
pkgname=('linux-lts-xen') # Build stock -ARCH kernel with Xen guest support
# pkgname=linux-custom       # Build kernel with a different name
_kernelname=${pkgname#linux}
_basekernel=3.0
pkgver=${_basekernel}.74
pkgrel=1
arch=('i686' 'x86_64')
url='http://www.kernel.org'
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl')
options=('!strip')
source=(http://www.kernel.org/pub/linux/kernel/v3.x/linux-${_basekernel}.tar.xz
        http://www.kernel.org/pub/linux/kernel/v3.x/patch-${pkgver}.xz
        # the main kernel config files
        config.i686 config.x86_64
        # standard config files for mkinitcpio ramdisk
        ${pkgname}.preset
        change-default-console-loglevel.patch
        i915-fix-ghost-tv-output.patch
        ext4-options.patch)
md5sums=('ecf932280e2441bdd992423ef3d55f8f'
         '867605b6b3e0495464d3093556a3409d'
         'ee4f6157e3dfe886c1a9d2f5ce35d184'
         'b75c95ca17f3e7252a72140cfd5e8007'
         '00606e4eaf72f66a078f8545522017c9'
         '9d3c56a4b999c8bfbd4018089a62f662'
         '263725f20c0b9eb9c353040792d644e5'
         'c8299cf750a84e12d60b372c8ca7e1e8')

build() {
  cd "${srcdir}/linux-${_basekernel}"

  # add upstream patch
  msg "Applying kernel release patch..."
  patch -p1 -i "${srcdir}/patch-${pkgver}"

  # add latest fixes from stable queue, if needed
  # http://git.kernel.org/?p=linux/kernel/git/stable/stable-queue.git

  # Some chips detect a ghost TV output
  # mailing list discussion: http://lists.freedesktop.org/archives/intel-gfx/2011-April/010371.html
  # Arch Linux bug report: FS#19234
  #
  # It is unclear why this patch wasn't merged upstream, it was accepted,
  # then dropped because the reasoning was unclear. However, it is clearly
  # needed.
  msg "Applying i915 patch"
  patch -Np1 -i "${srcdir}/i915-fix-ghost-tv-output.patch"

  # set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
  # remove this when a Kconfig knob is made available by upstream
  # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
  msg "Applying default console loglevel patch"
  patch -Np1 -i "${srcdir}/change-default-console-loglevel.patch"

  # fix ext4 module to mount ext3/2 correct
  # https://bugs.archlinux.org/task/28653
  msg "Applying ext4 options patch"
  patch -Np1 -i "${srcdir}/ext4-options.patch"

  cat "${srcdir}"/config.$CARCH > ./.config

  if [ "${_kernelname}" != "" ]; then
    sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
  fi

  # set extraversion to pkgrel
  sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  yes "" | make config

  # build!
  make ${MAKEFLAGS} bzImage modules
}

package_linux-lts-xen() {
  pkgdesc="The Linux Kernel and modules with Xen guest support - stable longtime supported kernel package suitable for servers"
  depends=('coreutils' 'linux-firmware' 'module-init-tools>=3.16' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('linux-lts' 'kernel26-lts' 'kernel26-lts-xen')
  conflicts=('kernel26-lts' 'kernel26-lts-xen')
  replaces=('kernel26-lts' 'nouveau-drm-lts' 'kernel26-lts-xen')
  backup=("etc/mkinitcpio.d/${pkgname}.preset")
  install=${pkgname}.install

  cd "${srcdir}/linux-${_basekernel}"

  KARCH=x86

  # get kernel version
  _kernver="$(make kernelrelease)"

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
  make INSTALL_MOD_PATH="${pkgdir}" modules_install
  cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-${pkgname}"

  # add vmlinux
  install -D -m644 vmlinux "${pkgdir}/usr/src/linux-${_kernver}/vmlinux"

  # install fallback mkinitcpio.conf file and preset file for kernel
  install -D -m644 "${srcdir}/${pkgname}.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"

  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/g" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i "${startdir}/${pkgname}.install"
  sed \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-${pkgname}\"|g" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-${pkgname}.img\"|g" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-${pkgname}-fallback.img\"|g" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgname}.preset"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # gzip -9 all modules to save 100MB of space
  find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"
}

# vim:set ts=2 sw=2 et:
