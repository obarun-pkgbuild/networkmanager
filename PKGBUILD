# Maintainer: Eric Vidal <eric@obarun.org>
# based on the original https://projects.archlinux.org/svntogit/packages.git/tree/trunk?h=packages/networkmanager
# 						Maintainer: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# 						Maintainer: Jan de Groot <jgc@archlinxu.org>
# 						Contributor: Wael Nasreddine <gandalf@siemens-mobiles.org>
# 						Contributor: Tor Krill <tor@krill.nu>
# 						Contributor: Will Rea <sillywilly@gmail.com>
# 						Contributor: Valentine Sinitsyn <e_val@inbox.ru>

pkgbase=networkmanager
pkgname=(networkmanager libnm libnm-glib)
pkgver=1.8.4
pkgrel=2
pkgdesc="Network connection manager and user applications"
arch=(x86_64)
license=(GPL2 LGPL2.1)
url="https://wiki.gnome.org/Projects/NetworkManager"
_pppver=2.4.7
makedepends=(intltool dhclient iptables gobject-introspection gtk-doc "ppp=$_pppver" modemmanager
             dbus-glib iproute2 nss polkit wpa_supplicant libsoup libgudev libmm-glib
             libnewt libndp libteam vala perl-yaml python-gobject git vala jansson bluez-libs
             glib2-docs gettext)
checkdepends=(libx11 python-dbus)
_commit=51fdc50ab179a0582012d1b50e587761765ad15e # tags/1.8.4^0
source=("git+https://anongit.freedesktop.org/git/NetworkManager/NetworkManager#commit=$_commit"
		20-connectivity.conf
        NetworkManager.conf)
sha256sums=('SKIP'
            '477d609aefd991c48aca93dc7ea5a77ebebf46e0481184530cceda4c0d8d72c6'
            '1afb0e849054ba68c3a3565746ed532a10a156fb3b686ac1280cf4afa985b89d')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  mkdir -p libnm{,-glib}/usr/{include,lib/{girepository-1.0,pkgconfig},share/{gir-1.0,gtk-doc/html,vala/vapi}}

  cd NetworkManager
  NOCONFIGURE=1 ./autogen.sh
}

pkgver() {
  cd NetworkManager
  git describe | sed 's/-dev/dev/;s/-rc/rc/;s/-/+/g'
}

build() {
  cd NetworkManager
  ./configure --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    runstatedir=/run \
    --sbindir=/usr/bin \
    --libexecdir=/usr/lib/NetworkManager \
    --disable-ifcfg-rh \
    --disable-ifcfg-suse \
    --disable-ifnet \
    --disable-ifupdown \
    --disable-lto \
    --disable-more-warnings \
    --disable-static \
    --enable-bluez5-dun \
    --enable-concheck \
    --enable-config-plugin-ibft \
    --enable-gtk-doc \
    --enable-introspection=yes \
    --enable-json-validation \
    --enable-ld-gc \
    --enable-modify-system \
    --enable-polkit=yes \
    --enable-polkit-agent \
    --enable-teamdctl \
    --enable-wifi \
    --with-config-dhcp-default=dhclient \
    --with-config-dns-rc-manager-default=resolvconf \
    --with-config-plugins-default=keyfile,ibft \
    --with-crypto=nss \
    --with-dbus-sys-dir=/usr/share/dbus-1/system.d \
    --with-dhclient=/usr/bin/dhclient \
    --with-dist-version="$pkgver-$pkgrel, Arch Linux" \
    --with-dnsmasq=/usr/bin/dnsmasq \
    --with-dnssec-trigger=/usr/lib/dnssec-trigger \
    --with-hostname-persist=default \
    --with-iptables=/usr/bin/iptables \
    --with-kernel-firmware-dir=/usr/lib/firmware \
    --with-libnm-glib \
    --with-libsoup \
    --with-modem-manager-1 \
    --with-nmcli \
    --with-nmtui \
    --with-pppd-plugin-dir=/usr/lib/pppd/$_pppver \
    --with-pppd=/usr/bin/pppd \
    --with-resolvconf=/usr/bin/resolvconf \
    --with-session-tracking=consolekit \
    --with-suspend-resume=upower \
    --with-system-ca-path=/etc/ssl/certs \
    --with-systemd-journal=no \
    --with-systemd-logind=no \
    --with-udev-dir=/usr/lib/udev \
    --with-wext \
    --without-dhcpcd \
    --without-libaudit \
    --without-netconfig \
    --without-ofono \
    --without-selinux

  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool

  make
}

check() {
  cd NetworkManager
  make -k check
}

package_networkmanager() {
  depends=(libnm-glib iproute2 polkit wpa_supplicant libsoup libmm-glib libnewt libndp libteam
           curl bluez-libs upower)
  optdepends=('dnsmasq: connection sharing'
              'bluez: Bluetooth support'
              'openresolv: resolvconf support'
              'ppp: dialup connection support'
              'dhclient: External DHCP client'
              'modemmanager: cellular network support')
  backup=('etc/NetworkManager/NetworkManager.conf')
  groups=('gnome')
  
  cd NetworkManager
  make DESTDIR="$pkgdir" install

  install -dm700 "$pkgdir/etc/NetworkManager/system-connections"
  install -d "$pkgdir"/etc/NetworkManager/{conf,dnsmasq}.d
  install -m644 ../NetworkManager.conf "$pkgdir/etc/NetworkManager/"
  install -Dm644 ../20-connectivity.conf \
	"$pkgdir/usr/lib/NetworkManager/conf.d/20-connectivity.conf"

### Split libnm

  cd ../libnm
  mv "$pkgdir"/usr/include/libnm usr/include
  mv "$pkgdir"/usr/lib/girepository-1.0/NM-* usr/lib/girepository-1.0
  mv "$pkgdir"/usr/lib/libnm.* usr/lib
  mv "$pkgdir"/usr/lib/pkgconfig/libnm.pc usr/lib/pkgconfig
  mv "$pkgdir"/usr/share/gir-1.0/NM-* usr/share/gir-1.0
  mv "$pkgdir"/usr/share/gtk-doc/html/libnm usr/share/gtk-doc/html
  mv "$pkgdir"/usr/share/vala/vapi/libnm.* usr/share/vala/vapi

### Split libnm-glib

  cd ../libnm-glib
  mv "$pkgdir"/usr/include/* usr/include
  mv "$pkgdir"/usr/lib/girepository-1.0/* usr/lib/girepository-1.0
  mv "$pkgdir"/usr/lib/libnm-* usr/lib
  mv "$pkgdir"/usr/lib/pkgconfig/* usr/lib/pkgconfig
  mv "$pkgdir"/usr/share/gir-1.0/* usr/share/gir-1.0
  mv "$pkgdir"/usr/share/gtk-doc/html/libnm-* usr/share/gtk-doc/html
  mv "$pkgdir"/usr/share/vala/vapi/* usr/share/vala/vapi

  rmdir -p --ignore-fail-on-non-empty \
    "$pkgdir"/usr/include \
    "$pkgdir"/usr/lib/{girepository-1.0,pkgconfig} \
    "$pkgdir"/usr/share/{gir-1.0,vala/vapi}
}

package_libnm() {
  pkgdesc="NetworkManager client library"
  depends=(glib2 libgudev nss libutil-linux jansson)
  mv libnm/* "$pkgdir"
}

package_libnm-glib() {
  pkgdesc="NetworkManager client library (legacy)"
  depends=(libnm dbus-glib)
  mv libnm-glib/* "$pkgdir"
}