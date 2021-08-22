# 搭建WordPress个人博客

## 1 安装apache

```sh
yum -y install httpd httpd-manual mod_ssl mod_perl mod_auth_mysql
systemctl start httpd.service
```

如果是阿里云等，需打开相应安全组下的对外端口，然后到浏览器上输入服务器的公网地址出现如下图示则启动成功

<img src="C:\Users\x3125\AppData\Roaming\Typora\typora-user-images\image-20210822113706832.png" alt="image-20210822113706832" style="zoom:30%;" />

## 2 安装mysql

因为我事先已经安装好MySQL了，如果有不会安装的可以看我这篇博客：[上链接]()

安装好MySQL后还需要创建一个用于搭建博客的库：wordpress

## 3 安装php

```sh
yum -y install php php-php-mysql php-php-gd php-php-xml php-php-common php-php-mbstring php-php-ldap php-php-pear php-php-xmlrpc php-php-imap
systemctl enable php-php-fpm
systemctl start php-php-fpm



# 如果有报错则在末尾添加参数 --skip-broken
echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
systemctl restart httpd
# 打开浏览器访问：http://<ECS公网IP>/phpinfo.php 显示如下则表示安装成功
```

<img src="C:\Users\x3125\AppData\Roaming\Typora\typora-user-images\image-20210822114429465.png" alt="image-20210822114429465" style="zoom:33%;" />









```sh
./configure --prefix=/opt/php/php/ --with-config-file-path=/opt/php/etc  --enable-fpm --enable-mysqlnd --enable-opcache --enable-pcntl --enable-mbstring --enable-soap --enable-zip --enable-calendar  --enable-bcmath --enable-exif --enable-ftp --enable-intl --with-mysqli  --with-pdo-mysql --with-openssl --with-curl --with-gd --with-gettext  --with-mhash --with-openssl --with-mcrypt --with-tidy --enable-wddx  --with-xmlrpc --with-zlib
```



```sh
./configure \
--prefix=/opt/php8/ \
--with-config-file-path=/opt/php8/etc \
--with-curl \
--with-freetype \
--enable-gd \
--with-jpeg \
--with-gettext \
--with-kerberos \
--with-libdir=lib64 \
--with-libxml \
--with-mysqli \
--with-openssl \
--with-pdo-mysql \
--with-pdo-sqlite \
--with-pear \
--enable-sockets \
--with-mhash \
--with-ldap-sasl \
--with-xsl \
--with-zlib \
--with-zip \
--with-bz2 \
--with-iconv \
--enable-fpm \
--enable-pdo \
--enable-bcmath \
--enable-mbregex \
--enable-mbstring \
--enable-opcache \
--enable-pcntl \
--enable-shmop \
--enable-soap \
--enable-sockets \
--enable-sysvsem \
--enable-xml \
--enable-sysvsem \
--enable-cli \
--enable-opcache \
--enable-intl \
--enable-calendar \
--enable-static \
--enable-mysqlnd
```



./configure --prefix=/opt/php8 --with-config-file-path=/opt/php8/etc \
--enable-fpm --enable-mysqlnd --enable-opcache --enable-pcntl \
--enable-mbstring --enable-soap --enable-zip --enable-calendar \
--enable-bcmath --enable-exif --enable-ftp --enable-intl --with-mysqli \
--with-pdo-mysql --with-openssl --with-curl --with-gd --with-gettext \
--with-mhash --with-openssl --with-mcrypt --with-tidy --enable-wddx \
--with-xmlrpc --with-zlib





vim /lib/systemd/system/php-fpm.service 





```sh
yum install php70w-common php70w-fpm php70w-opcache php70w-gd php70w-mysqlnd php70w-mbstring php70w-pecl-redis php70w-pecl-memcached php70w-devel
```





```sh
/opt/remi/php/root/usr/lib64/php
/opt/remi/php/root/usr/bin/php
/opt/remi/php/root/usr/share/php
/var/opt/remi/php/lib/php
/usr/bin/php
/usr/include/php-zts/php
```

