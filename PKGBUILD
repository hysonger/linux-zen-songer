# Maintainer: Jan Alexander Steffens (heftig) <heftig@archlinux.org>
# Modified by Songer
# 根据Archlinux官方仓库版本修改，默认使用clang，合并了cjktty补丁和来自xanmod的更多目标平台补丁
# 我瞎改的，请自行承担使用责任 PERSONAL MODIFICATION PLEASE TAKE RESPONSIBILITY ON YOUR OWN

pkgbase=linux-zen-songer # 需要自己修改以区分开

_kernel_ver=6.3.5 # 更新版本要改
_srcname=linux-${_kernel_ver} # 更新版本要改
pkgver=${_kernel_ver}.zen1 # 更新版本要改

pkgrel=1
pkgdesc='Linux ZEN (modified by Songer)'
_srctag=v${pkgver%.*}-${pkgver##*.}
url="https://github.com/zen-kernel/zen-kernel/commits/$_srctag"
arch=(x86_64)
license=(GPL2)
makedepends=(
  bc
  cpio
  gettext
  git
  libelf
  pahole
  perl
  tar
  xz
  clang # 使用clang

  # I DO NOT NEED THEM
  # htmldocs
  #graphviz
  #imagemagick
  #python-sphinx
  #texlive-latexextra
)
options=('!strip')

source=(
  "https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-${_kernel_ver}.tar."{xz,sign}
  "https://github.com/zen-kernel/zen-kernel/releases/download/${_srctag}/${_srctag}.patch."{xz,xz.sig}
  "https://github.com/zhmars/cjktty-patches/raw/master/v6.x/cjktty-6.3.patch"
  "https://github.com/zhmars/cjktty-patches/raw/master/cjktty-add-cjk32x32-font-data.patch"
  "https://github.com/xanmod/linux-patches/raw/master/linux-6.3.y-xanmod/x86/0001-x86-kconfig-more-uarches-for-kernel-5.17.patch"
  config  # I DO NOT NEED THE ORIGINAL ONE
)
# 记得导入这几个b的公钥，官方仓库有，自己想办法
validpgpkeys=(
  ABAF11C65A2970B130ABE3C479BE3E4300411886  # Linus Torvalds
  647F28654894E3BD457199BE38DBBDC86092693E  # Greg Kroah-Hartman
  A2FF3A36AAA56654109064AB19802F8B0D70FC30  # Jan Alexander Steffens (heftig)
  C5ADB4F3FEBBCE27A3E54D7D9AE4078033F8024D  # Steven Barrett <steven@liquorix.net>
)

b2sums=('646a94591eae93db9301a11e5300579c8cce7d2a544727cb88efed86d05ba070a247498d9c83d7b7cdbead4e7d46537134c877813aa7f188dd36b403c58d0c11'
        'SKIP'
        '3e84827891292f71307c67d4fd764c0c06437dccb90998d828a9778279e5181f75a12ed3224c3c476b1747ab96ea0f21c8e1895d07f3b020fe3dd82c4a76c00f'
        'SKIP'
        '8176dce642d239ed6747ec77697ae4025e2584ab33d3f77c4a6cdd81eec187bece44de8c0592eb61e308fcdf03a914610d6c0f5db4d5273bb547ecfe66ebb643'
        'd8d9e27996cf5c015c3bd04768e7ed48bf7206d7cc49e6fa2ccdb8b88bc121615fb9429b0b99d69a5ec6a5b2b5a8efe10de68ace4a34a242d056b4de5da83d72'
        'd748197ff69c14f53d39044e30109eebccb44e1a655f4578fdf74821328af1d96dfde88f068399ec24d15251cf64cc8a592a35aa8dcc42daba55c65097eb1348'
        'SKIP')

_compiler_flags="CC=clang HOSTCC=clang LLVM=1 LLVM_IAS=1" # 使用clang

export KBUILD_BUILD_HOST=archlinux
export KBUILD_BUILD_USER=songer
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

_make() {
  test -s version
  make ${_compiler_flags} KERNELRELEASE="$(<version)" "$@"
}

prepare() {
  cd $_srcname

  echo ">>> 设置版本信息..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  _make defconfig
  _make -s kernelrelease > version
  _make mrproper

  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    [[ $src = *.patch ]] || continue
    echo ">>> 应用补丁 $src..."
    patch -Np1 < "../$src"
  done

  echo ">>> 配置 config 文件..."

  cp -f ../config .config # 把config复制过去

  _make olddefconfig

  diff ../config .config
  echo ">>> config 和留存版本相比，出现以上修改"
  read

  _make LSMOD=$HOME/.config/modprobed.db localmodconfig # 可选 探测记录的modules 配合modprobed-db使用

  diff ../config .config
  echo ">>> config 和留存版本相比，出现以上修改"
  read

  _make menuconfig # 编译前再检查一次设置

  echo ">>> 已准备好 $pkgbase 版本 $(<version)"

  cat .config > "${SRCDEST}/config.last" # 可选 备份最后生成的config文件 将其改名为config下次编译就可以此配置为起点
}

build() {
  echo ">>> 开始编译 (make all)..."
  cd $_srcname
  _make all
}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=(
    coreutils
    initramfs
    kmod
  )
  optdepends=(
    'wireless-regdb: to set the correct wireless channels of your country'
    'linux-firmware: firmware images needed for some devices'
  )
  provides=(
    KSMBD-MODULE
    UKSMD-BUILTIN
    VHBA-MODULE
    VIRTUALBOX-GUEST-MODULES
    WIREGUARD-MODULE
  )
  replaces=(
  )

  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(_make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  _make ${_compiler_flags} INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install  # Suppress depmod

  # remove build and source links
  rm "$modulesdir"/{source,build}
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=(pahole)

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts

  # required when STACK_VALIDATION is enabled
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  #install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids # 未开启，会引发错误，故注释 noexist error

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */x86/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -Sib "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Stripping vmlinux..."
  strip -v $STRIP_STATIC "$builddir/vmlinux"

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=(
  "$pkgbase"
  "$pkgbase-headers"
#  "$pkgbase-docs" # no need
)
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
