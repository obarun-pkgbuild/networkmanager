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
pkgver=1.10.6
pkgrel=6
pkgdesc="Network connection manager and user applications"
arch=(x86_64)
license=(GPL2 LGPL2.1)
url="https://wiki.gnome.org/Projects/NetworkManager"
_pppver=2.4.7
makedepends=(intltool dhclient iptables gobject-introspection gtk-doc "ppp=$_pppver" modemmanager
             dbus-glib iproute2 nss polkit wpa_supplicant libgudev libmm-glib
             libnewt libndp libteam vala perl-yaml python-gobject git vala jansson bluez-libs
             glib2-docs gettext dhcpcd)
checkdepends=(libx11 python-dbus)
_commit=dd8cf21cea13fa1bbee11fd3e0e7519e4b4ba712 # tags/1.10.6^0
source=("git+https://anongit.freedesktop.org/git/NetworkManager/NetworkManager#commit=$_commit"
		20-connectivity.conf
        NetworkManager.conf)
sha256sums=('SKIP'
            '477d609aefd991c48aca93dc7ea5a77ebebf46e0481184530cceda4c0d8d72c6'
            '35db584a3324c4583006e56efa6a64c236b339019474d9b307c98c4bc9860aca')
validpgpkeys=('6DD4217456569BA711566AC7F06E8FDE7B45DAAC') # Eric Vidal

prepare() {
  mkdir -p libnm{,-glib}/usr/{include,lib/{girepository-1.0,pkgconfig},share/{gir-1.0,gtk-doc/html,vala/vapi}}

  cd NetworkManager
  git cherry-pick -n 4d1f090aedf05c0e2955d431638e311d1e18a52f
  NOCONFIGURE=1 ./autogen.sh
}

#pkgver() {
#  cd NetworkManager
#  git describe | sed 's/-dev/dev/;s/-rc/rc/;s/-/+/g'
#}

build() {
	export PYTHONPATH="/usr/share/glib-2.0"
  cd NetworkManager
  ./configure --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    runstatedir=/run \
    --sbindir=/usr/bin \
    --libexecdir=/usr/lib \
    --disable-ifcfg-rh \
    --disable-ifcfg-suse \
    --disable-ifnet \
    --disable-ifupdown \
    --disable-lto \
    --disable-more-logging \
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
    --with-config-dhcp-default=internal \
    --with-config-dns-rc-manager-default=symlink \
    --with-config-plugins-default=keyfile,ibft \
    --with-crypto=nss \
    --with-dbus-sys-dir=/usr/share/dbus-1/system.d \
    --with-dhclient=/usr/bin/dhclient \
    --with-dhcpcd-supports-ipv6 \
    --with-dhcpcd=/usr/bin/dhcpcd \
    --with-dist-version="$pkgver-$pkgrel, Arch Linux" \
    --with-dnsmasq=/usr/bin/dnsmasq \
    --with-dnssec-trigger=/usr/lib/dnssec-trigger/dnssec-trigger-script \
    --with-hostname-persist=default \
    --with-iptables=/usr/bin/iptables \
    --with-kernel-firmware-dir=/usr/lib/firmware \
    --with-libnm-glib \
    --with-modem-manager-1 \
    --with-nmcli \
    --with-nmtui \
    --with-pppd-plugin-dir=/usr/lib/pppd/$_pppver \
    --with-pppd=/usr/bin/pppd \
    --with-resolvconf=/usr/bin/resolvconf \
    --with-session-tracking=consolekit \
    --with-suspend-resume=upower \
    --with-system-ca-path=/etc/ssl/certs \
    --with-udev-dir=/usr/lib/udev \
    --with-wext \
    --without-dhcpcd \
    --without-libaudit \
    --without-more-asserts \
    --without-netconfig \
    --without-ofono \
    --without-selinux \
    --with-systemdsystemunitdir=no \
	--with-systemd-journal=no \
    --with-systemd-logind=no
    
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool

  make
}

check() {
  cd NetworkManager
  # netns tests fail in our containers
  make -k check || :
}

package_networkmanager() {
  depends=(libnm-glib iproute2 polkit wpa_supplicant libmm-glib libnewt libndp libteam
           curl bluez-libs upower dhclient)
  optdepends=('dnsmasq: connection sharing'
              'bluez: Bluetooth support'
              'ppp: dialup connection support'
              'modemmanager: cellular network support')
  backup=('etc/NetworkManager/NetworkManager.conf')
  groups=('gnome')
  
  cd NetworkManager
  make DESTDIR="$pkgdir" install

  # packaged configuration
  install -Dm644 "${srcdir}/20-connectivity.conf" "$pkgdir/usr/lib/NetworkManager/conf.d/20-connectivity.conf" 

  # /etc/NetworkManager
  install -d "$pkgdir"/etc/NetworkManager/{conf,dnsmasq}.d
  install -dm700 "$pkgdir/etc/NetworkManager/system-connections"
  install -Dm644 "${srcdir}/NetworkManager.conf" "$pkgdir/etc/NetworkManager/NetworkManager.conf" 

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
