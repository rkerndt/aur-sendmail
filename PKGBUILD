# Maintainer: Thomas Berryhill <tb01110100@gmail.com>
# Contributor: Vlad M. <vlad@arhclinux.net>
# Contributor: chrisl echo archlinux@c2h0r1i2s4t5o6p7h8e9r-l3u4n1a.com|sed 's/[0-9]//g'
# Contributor: mazieres
# Contributor: doze_worm <shuimao@gmail.com> the original port.

pkgname="sendmail"
pkgver=8.15.2
pkgrel=3
pkgdesc="The sendmail MTA"
url="http://www.sendmail.org"
arch=('i686' 'x86_64')
license=('Sendmail License')
provides=('sendmail=8.15')
conflicts=('msmtp-mta'
           'postfix'
           'exim'
           'opensmtpd')
backup=('etc/conf.d/sendmail'
        'etc/mail/aliases'
        'etc/mail/sendmail.cf'
        'etc/pam.d/smtp')
source=("ftp://ftp.sendmail.org/pub/${pkgname}/${pkgname}.${pkgver}.tar.gz"
        'sendmail.service'
        'sendmail.conf'
        'sm-client.service'
        'sendmail-compile-against-openssl-1.1.0.patch'
        'smtp')
depends=('db'
         'cyrus-sasl')
sha256sums=('24f94b5fd76705f15897a78932a5f2439a32b1a2fdc35769bb1a5f5d9b4db439'
            '380edeb289dfdfc5b0d4ea38df3a0fd35e6f83eeee76254ec7b3506eadfb674f'
            '39730f2be66bb1f1e6bc7fff61911db632ecf4b891d348df525abe2020274580'
            'ecbd0a27e868d73d87fcfec292c19ea9479d0a8e9783788596d9add5e012218f'
            '92aa2e6d81c3b27baf78df2c00775279e93303dedec2b9989c9c528413908ad4'
            'c19d98aebf1002f6e2bf2603735e4c65a30374baa1e9e6f06364af13278f0655')
install="${pkgname}.install"

prepare(){
  # patch to fix tls.c compilation errors when built with openssl-1.1, taken
  # from Desbian here:
  #https://bugs.debian.org/cgi-bin/bugreport.cgi?att=1;bug=828540;filename=sendmail-compile-against-openssl-1.1.0.patch;msg=41
  cd ${srcdir}/${pkgname}-${pkgver}
  patch -p1 -i "${srcdir}/sendmail-compile-against-openssl-1.1.0.patch"

  # set build features in site.config.m4 for STARTTLS, SASL, and inet6
  siteconfig="$srcdir/${pkgname}-${pkgver}/devtools/Site/site.config.m4"
  touch ${siteconfig}
  echo "APPENDDEF(\`conf_sendmail_ENVDEF', \`-DSTARTTLS')" >> ${siteconfig}
  echo "APPENDDEF(\`confLIBS', \`-lssl -lcrypto')" >> ${siteconfig}
  echo "APPENDDEF(\`confENVDEF', \`-DNETINET6')" >> ${siteconfig}
  echo "APPENDDEF(\`conf_sendmail_ENVDEF', \`-DSASL=2')" >> ${siteconfig}
  echo "APPENDDEF(\`conf_sendmail_LIBS', \`-lsasl2')" >> ${siteconfig}  
}

build(){
  cd ${srcdir}/${pkgname}-${pkgver}
  ./Build || return 1
}

package(){
  mkdir -p $pkgdir/etc/conf.d/
  mkdir -p $pkgdir/usr/{bin,sbin,share/man,share/doc/sendmail,/lib/systemd/system} \
    $pkgdir/usr/man/man{1,5,8} $pkgdir/var/spool/mqueue \
    || return 1
  cp sendmail.service sm-client.service $pkgdir/usr/lib/systemd/system
  cp sendmail.conf $pkgdir/etc/conf.d/sendmail
  cp smtp $pkgdir/etc/pam.d
  cd "$srcdir/${pkgname}-${pkgver}" || return 1
  make install DESTDIR="$pkgdir" || return 1
  make -C mail.local force-install DESTDIR="$pkgdir" || return 1
  make -C rmail force-install DESTDIR="$pkgdir" || return 1
  mv $pkgdir/usr/man/* $pkgdir/usr/share/man/
  rmdir $pkgdir/usr/man
  cp -r cf $pkgdir/usr/share/sendmail-cf
  cp sendmail/aliases $pkgdir/etc/mail/aliases
  cp cf/cf/generic-linux.cf $pkgdir/etc/mail/sendmail.cf
#  cp doc/op/op.{ps,txt} $pkgdir/usr/share/doc/sendmail/
  install -D -m644 LICENSE "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
  find $pkgdir -user bin -print | xargs chown root
  find $pkgdir -group bin -print | xargs chgrp root
  mv $pkgdir/usr/sbin/* $pkgdir/usr/bin/
  rmdir $pkgdir/usr/sbin
}
