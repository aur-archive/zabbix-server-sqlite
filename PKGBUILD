# Maintainer: Oleg Rakhmanov <orakhmanov [at] gmail [dot] com>
# Contributor: Idares <idares@seznam.cz>
# Contributor: Phillip Smith <fukawi2@NO-SPAM.gmail.com>
# Contributor: Enrico Morelli <morelli@cerm.unifi.it>


pkgname=zabbix-server-sqlite
pkgver=2.2.3
pkgrel=1
pkgdesc="Software for monitoring of your applications, network and servers."
arch=('i686' 'x86_64' 'arm' 'armv6h' 'armv7h')
url="http://www.zabbix.com"
license=('GPL')
depends=('sqlite3' 'php' 'php-gd' 'fping' 'net-snmp' 'curl' 'iksemel' 'libxml2')
optdepends=('apache: HTTP server'
            'lighttpd: HTTP server'
            'nginx: HTTP server')
backup=('etc/zabbix/zabbix_server.conf' 'etc/zabbix/zabbix.db')
install='zabbix-server.install'
options=('emptydirs')
conflicts=('zabbix-server' 'zabbix-server-mysql')
source=("http://downloads.sourceforge.net/sourceforge/zabbix/zabbix-$pkgver.tar.gz"
        'zabbix-server.service'
        'zabbix-server.tmpfiles'
        'zabbix.db')

md5sums=('cb1bda41a742175a445e8d32d103a315'
         '7200c01662be3a1d364c280ff2a818ac'
         '9ce692356b4ac0a71595ce55fe3b44c1'
         'd41d8cd98f00b204e9800998ecf8427e')

_HTMLPATH='srv/http/zabbix'

build() {
  cd "$srcdir/zabbix-$pkgver"

  ./configure \
    --prefix=/usr \
    --sbindir=/usr/bin \
    --enable-server \
    --with-net-snmp \
    --with-jabber \
    --with-libcurl \
    --with-sqlite3 \
    --with-libxml2 \
    --sysconfdir=/etc/zabbix

  make
}

package() {
  cd "$srcdir/zabbix-$pkgver"

  make DESTDIR="$pkgdir" install

  # Create data dirs required
  install -d -m0750 $pkgdir/var/log/zabbix

  # database schema
  _DBSCHEMADIR="$pkgdir/etc/zabbix/database/sqlite3"
  mkdir -p $_DBSCHEMADIR
  for _SQLFILE in {schema,data,images}.sql; do
    install -D -m 0444 database/sqlite3/$_SQLFILE $_DBSCHEMADIR/$_SQLFILE
  done

  # frontends
  mkdir -p $pkgdir/$_HTMLPATH/
  cp -r frontends/php/* $pkgdir/$_HTMLPATH/
  chown -R http:http $pkgdir/$_HTMLPATH/
  chmod -R u=rwX,g=rX,o= $pkgdir/$_HTMLPATH/

  # default configuration files
  install -D -m 0640 conf/zabbix_server.conf $pkgdir/etc/zabbix/zabbix_server.conf

  # change pid file location
  sed -i 's:# PidFile=.*:PidFile=/run/zabbix/zabbix_server.pid:' $pkgdir/etc/zabbix/zabbix_server.conf

  # change database location
  sed -i 's:DBName=.*:DBName=/etc/zabbix/zabbix.db:' $pkgdir/etc/zabbix/zabbix_server.conf

  # service file
  install -D -m 0644 $srcdir/zabbix-server.service $pkgdir/usr/lib/systemd/system/zabbix-server.service

  # tmpfile
  install -D -m 0644 $srcdir/zabbix-server.tmpfiles $pkgdir/usr/lib/tmpfiles.d/zabbix-server.conf

  # empty db file
  install -D -m 0664 $srcdir/zabbix.db $pkgdir/etc/zabbix/zabbix.db
}

