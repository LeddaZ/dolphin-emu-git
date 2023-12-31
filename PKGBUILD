# Maintainer: Daniel Peukert <daniel@peukert.cc>
# Contributor: Maxime Gauduin <alucryd@archlinux.org>
# Contributor: Lightning <sgsdxzy@gmail.com>
_projectname='dolphin'
_mainpkgname="$_projectname-emu"
_noguipkgname="$_projectname-emu-nogui"
pkgbase="$_mainpkgname-git"
pkgname=("$pkgbase" "$_noguipkgname-git")
pkgver='5.0.r19742.g7a2352f90c'
pkgrel='1'
pkgdesc='A Gamecube / Wii emulator'
_pkgdescappend=' - git version'
arch=('x86_64' 'aarch64')
url="https://$_mainpkgname.org"
license=('GPL2')
depends=(
	'alsa-lib' 'bluez-libs' 'cubeb' 'enet' 'hidapi' 'libevdev' 'libgl' 'libpulse'
	'libspng' 'libx11' 'libxi' 'libxrandr' 'lzo' 'mbedtls2' 'minizip-ng'
	'pugixml' 'qt6-base' 'qt6-svg' 'sfml' 'zlib'
	'libavcodec.so' 'libavformat.so' 'libavutil.so' 'libcurl.so' 'libfmt.so'
	'libminiupnpc.so' 'libswscale.so' 'libudev.so' 'libusb-1.0.so'
)
makedepends=('cmake' 'git' 'ninja' 'python')
optdepends=('pulseaudio: PulseAudio backend')
source=(
	"$pkgname::git+https://github.com/LeddaZ/dolphin.git"
	"$pkgname-implot::git+https://github.com/epezent/implot.git"
	"$pkgname-mgba::git+https://github.com/mgba-emu/mgba.git"
	"$pkgname-rcheevos::git+https://github.com/RetroAchievements/rcheevos.git"
	"$pkgname-vma::git+https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator.git"
	"$pkgname-zlibng::git+https://github.com/zlib-ng/zlib-ng.git"
)
sha512sums=('SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP'
            'SKIP')

_sourcedirectory="$pkgname"

prepare() {
	cd "$srcdir/$_sourcedirectory/"
	if [ -d 'build/' ]; then rm -rf 'build/'; fi
	mkdir 'build/'

	# Provide submodules
	declare -A _submodules=(
		[implot]='implot/implot'
		[mgba]='mGBA/mgba' # The current version of mgba in the repos is not compatible with Dolphin
		[rcheevos]='rcheevos/rcheevos'
		[vma]='VulkanMemoryAllocator'
		[zlibng]='zlib-ng/zlib-ng'
	)

	for _submod in "${!_submodules[@]}"; do
		_path="Externals/${_submodules[$_submod]}"
		git submodule init "$_path"
		git config "submodule.$_path.url" "$srcdir/$pkgname-$_submod/"
		git -c protocol.file.allow=always submodule update "$_path"
	done
}

pkgver() {
	cd "$srcdir/$_sourcedirectory/"
	git describe --long --tags | sed -e 's/-\([^-]*-g[^-]*\)$/-r\1/' -e 's/-/./g'
}

build() {
	# The dolphin-emu package in the repos MAKE_BUILD_TYPE=None for some reason, so we use it as well
	cd "$srcdir/$_sourcedirectory/"
	cmake -S '.' -B 'build/' -G Ninja \
		-DCMAKE_BUILD_TYPE=None \
		-DCMAKE_INSTALL_PREFIX='/usr' \
		-DDISTRIBUTOR=archlinux.org \
		-DENABLE_TESTS=OFF \
		-DENABLE_AUTOUPDATE=OFF \
		-DUSE_SHARED_ENET=ON \
		-DCMAKE_DISABLE_FIND_PACKAGE_LIBMGBA=True \
		-Wno-dev
	cmake --build 'build/'
}

package_dolphin-emu-git() {
	pkgdesc="$pkgdesc$_pkgdescappend"
	provides=("$_mainpkgname")
	conflicts=("$_mainpkgname")

	cd "$srcdir/$_sourcedirectory/"
	DESTDIR="$pkgdir" cmake --install 'build/'
	install -Dm644 'Data/51-usb-device.rules' "$pkgdir/usr/lib/udev/rules.d/51-usb-device.rules"

	rm -rf "$pkgdir/usr/bin/$_noguipkgname"
	rm -rf "$pkgdir/usr/include"
	rm -rf "$pkgdir/usr/lib/libdiscord-rpc.a"
	rm -rf "$pkgdir/usr/share/man/man6/$_noguipkgname.6"
}

package_dolphin-emu-nogui-git() {
	pkgdesc="$pkgdesc - no GUI$_pkgdescappend"
	depends=("$pkgbase")
	optdepends=()
	provides=("$_noguipkgname" "$_mainpkgname-cli")
	conflicts=("$_noguipkgname" "$_mainpkgname-cli")

	cd "$srcdir/$_sourcedirectory/"
	install -Dm755 "$srcdir/$_sourcedirectory/build/Binaries/$_noguipkgname" "$pkgdir/usr/bin/$_noguipkgname"
	ln -sf "/usr/bin/$_noguipkgname" "$pkgdir/usr/bin/$_mainpkgname-cli"
	install -Dm644 "Data/$_noguipkgname.6" "$pkgdir/usr/share/man/man6/$_noguipkgname.6"
}
