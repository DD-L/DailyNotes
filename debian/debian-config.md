debian 7.8:(netinst)
=====================

download

一个小型安装映像：可以被很快的下载到您的计算机，并须要把它复制到可删除的媒介，如光盘、U 盘等。安装过程中，您的计算机需要互联网连线。

http://cdimage.debian.org/debian-cd/7.8.0/amd64/iso-cd/debian-7.8.0-amd64-netinst.iso

via https://www.debian.org/distrib/

硬件配置(VM)
=============

<pre>
Memory			512MB
Processors		4
Hard Disk(SCSI)		20G
Network Adapter		Bridged(Automatic)
</pre>


Network
=======

<pre>
http://www.cnblogs.com/lizunicon/p/3798122.html

file: /etc/network/interfaces :
追加：
iface eth0 inet dhcp

/etc/init.d/network restart
</pre>

软件源:
=======

<pre><code>
/etc/apt/sources.list

# 考虑到7.x的libc库版本较低，于是就干脆装 8.x的包吧
#debian 7.x (wheezy)
#deb http://mirrors.aliyun.com/debian/ wheezy main non-free contrib
#deb http://mirrors.aliyun.com/debian/ wheezy-proposed-updates main non-free contrib
#deb-src http://mirrors.aliyun.com/debian/ wheezy main non-free contrib
#deb-src http://mirrors.aliyun.com/debian/ wheezy-proposed-updates main non-free contrib

#debian 8.x (jessie)
deb http://mirrors.aliyun.com/debian/ jessie main non-free contrib
deb http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ jessie main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ jessie-proposed-updates main non-free contrib

apt-get update
</code></pre>

语言配置
==========

<pre>
dpkg-reconfigure locales

or （
/etc/default/locale
LANG="zh_CN.UTF-8"
LANGUAGE="zh_CN:zh"
）

</pre>

ssh server:
============

<pre><code>
apt-get install openssh-server
#apt-get update
/etc/init.d/ssh restart
vi /etc/ssh/sshd_config # 参考http://www.jb51.net/os/Ubuntu/161384.html
开启sftp:
Subsystem sftp /usr/lib/openssh/sftp-server
</code></pre>

更新libc库，以便小米提供的toolchain能跑起来
==============================================

<pre><code>
apt-get update
apt-get install libc6
</code></pre>

/etc/profile
==============

<pre><code>
# added by D_L
alias 'll=ls -al --color'
alias '..=cd ..'
alias '...=cd ../..'

MIWIFI_TC_HOME=/home/deel/miwifi-dev/sdk_package/toolchain
alias 'arm-gcc=$MIWIFI_TC_HOME/bin/mipsel-openwrt-linux-uclibc-gcc'
alias 'arm-g++=$MIWIFI_TC_HOME/bin/mipsel-openwrt-linux-uclibc-g++'
</code></pre>

sudo
=====

<pre>
apt-get install sudo
chmod +w /etc/sudoers
vi /etc/sudoers
添加一行:
username     ALL=(ALL) ALL
username 为要设置的用户名
chmod 0440 /etc/sudoers
</pre>

dos2unix
==========

<pre>
apt-get install dos2unix
</pre>

git
====

<pre>
apt-get install git
</pre>


.vimrc
=======

<pre>
apt-get install vim
https://github.com/amix/vimrc
install Awesome version:
apt-get install ctags
git clone git://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
</pre>


arm-test
==========

<pre>
$ arm-g++ main.cpp -o main_test
mipsel-openwrt-linux-uclibc-g++: warning: environment variable 'STAGING_DIR' not defined

解决方法：
小米提供的sdk中没有叫“staging_dir”的。
写一个脚本
export STAGING_DIR=/home/deel/miwifi-dev/sdk_package
#export TOOLCHAIN_DIR=$STAGING_DIR/toolchain
#export PATH=$STAGING_DIR/bin:$PATH
#export LDCFLAGS=$TOOLCHAIN_DIR/usr/lib/
#export LD_LIBRARY_PATH=$TOOLCHAIN_DIR/usr/lib/
再执行arm-g++ main.cpp -o main_test可无告警通过

传到 板子上测试程序
可以执行，也可以显示中文
</pre>

xz
===

<pre>
sudo apt-get install xz-utils
</pre>

tree
=====

<pre>
sudo apt-get install tree
</pre>

debootstrap
============

<pre>
sudo apt-get install debootstrap

# debootstrap --arch [平台] [发行版本代号] [目录]
# debootstrap --arch i386 trusty /mnt
debootstrap --foreign --arch mipsel wheezy ./debian
</pre>

BOA web server
=================

<pre><code>
$ # 资料： 
$ # http://blog.sina.com.cn/s/blog_9e7fb3070101gumj.html
$ # http://blog.csdn.net/handyhuang/article/details/17127583
$ 
$ apt-get install byacc bison flex
$ apt-get install logrotate mime-support
$ wget http://www.boa.org/boa-0.94.13.tar.gz
$ tar zxvf boa-0.94.13.tar.gz
$ cd boa-0.94.13

$ mkdir /opt/boa
$ #sudo chown deel /opt/boa

$ # 修改 src/defines.h SERVER_ROOT 
$ # #define SERVER_ROOT /opt/boa
$ cd src
$ ./configure
$ make
util.c: In function ‘get_commonlog_time’:
util.c:100:1: error: pasting "t" and "->" does not give a valid preprocessing token
make: *** [util.o] Error 1
$ make clean
$ # 修改 
$ vi src/compat.h
$ # #define TIMEZONE_OFFSET(foo) foo##->tm_gmtoff
$ # ->
$ # #define TIMEZONE_OFFSET(foo) (foo)->tm_gmtoff
$ make 

$ # 配置 BOA 
$ cp boa.conf /opt/boa/
$ mkdir /opt/www
$ sudo chown deel /opt/www 
$ 
$ mkdir /opt/boa/lib
$ cp src/boa_indexer /opt/boa/lib/
$ cp src/boa /opt/boa/
$ 
$ sudo mkdir /var/log/boa/
$ # sudo chown deel /var/log/boa/
$
$ vi /opt/boa/boa.conf
$ ##########################
$ Port 8080
$ DocumentRoot /opt/www
$ DirectoryMaker /opt/boa/lib/boa_indexer
$ ##########################
</code></pre>

##curl

<pre><code>
$ apt-get install curl
</code></pre>

##bzip2

<pre><code>
$ apt-get install bzip2
</code></pre>

##nmap

<pre><code>
$ # http://nmap.org/download.html
$ wget https://nmap.org/dist/nmap-6.49BETA2.tar.bz2
$ bzip2 -cd nmap-6.49BETA2.tar.bz2 | tar xvf -
$ cd nmap-6.49BETA2
$ ./configure
$ sudo make clean
$ find . -name "*" | xargs -I '{}' touch '{}'
$ sudo make
$ sudo make install
</code></pre>

##python3

<pre><code>
apt-get install python3-pip
apt-get install python3-dev
</code></pre>

##netstat -ano

<pre></pre>


##Apache

<pre><code>
$ apt-get install apache2 apache2-utils
$ /etc/init.d/apache2 start
$ # /etc/init.d/apache2 
$ #Usage: apache2 {start|stop|graceful-stop|restart|reload|force-reload|start-htcacheclean|stop-htcacheclean}
$ # 配置
$ # ...
$ # http://www.laozuo.org/3423.html
$ # url重写（伪静态）默认APACHE是没有安装的,需要执行脚本（as root）启动rewrite： 
$ # a2enmod rewrite 
$
$
$ /usr/sbin/apache2 -v
Server version: Apache/2.4.10 (Debian)
Server built:   Mar 15 2015 09:51:43
</code></pre>

##mod_wsgi

<pre><code>
$ # https://pypi.python.org/pypi/mod_wsgi
$ #apt-get install apache2-mpm-prefork apache2-prefork-dev
$ #wget https://pypi.python.org/packages/source/m/mod_wsgi/mod_wsgi-4.4.13.tar.gz
$ apt-get update
$ apt-get upgrade
$ apt-get install libapache2-mod-wsgi
$ mkdir /var/run/apache2/wsgi
$ vi /etc/apache2/mods-available/wsgi.conf
WSGISocketPrefix /var/run/apache2/wsgi
WSGIScriptAlias /wsgi "/var/www/html/application.py"

# '''application.py'''
# def application(environ, start_response):
#    import sys
#    status = '200 OK'
#    output = 'Hello World!\n' + sys.version + '\n'
#    response_headers = [('Content-type', 'text/plain'),
#                        ('Content-Length', str(len(output)))]
#    start_response(status, response_headers)
#
#    return [output]

$ /etc/init.d/apache2 restart
$ curl http://hostname:port/wsgi
$ # 此时显示wsgi 的python版本为2.x
$ # 若想改为python3.x, 升级 wsgi版本即可, 或用pip3直接装:
$ pip3 install mod_wsgi
$ /usr/local/bin/mod_wsgi-express install-module
LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi-py34.cpython-34m.so
WSGIPythonHome /usr
$ vi /etc/apache2/mods-available/wsgi.load
# python 2.x
#LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi.so
# python 3.x
LoadModule wsgi_module /usr/lib/apache2/modules/mod_wsgi-py34.cpython-34m.so
$ /etc/init.d/apache2 restart
$ #/var/www/html/application.py 还要增加一句：
+ output = output.encode("UTF-8")

$ # 下面给出更全的例子：
$ cat application.py
<pre>
#! /usr/bin/env python3
# -*- coding: utf-8 -*-
import sys
def application(environ, start_response):
    status = '200 OK'
    response_body = ['%s: %s' % (key, value)
            for key, value in sorted(environ.items())]
    response_body = '\n'.join(response_body)

    py_version = sys.version
    response_body = response_body + '\n\npython version: ' + py_version

    response_body  = response_body.encode("UTF-8")
    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(response_body)))]
    start_response(status, response_headers)

    return [response_body]
</pre>
$ curl http://localhost:80/wsgi
<pre><code>
CONTEXT_DOCUMENT_ROOT: /var/www/html
CONTEXT_PREFIX: 
DOCUMENT_ROOT: /var/www/html
GATEWAY_INTERFACE: CGI/1.1
HTTP_ACCEPT: */*
HTTP_HOST: localhost
HTTP_USER_AGENT: curl/7.38.0
PATH_INFO: 
QUERY_STRING: 
REMOTE_ADDR: ::1
REMOTE_PORT: 41347
REQUEST_METHOD: GET
REQUEST_SCHEME: http
REQUEST_URI: /wsgi
SCRIPT_FILENAME: /var/www/html/application.py
SCRIPT_NAME: /wsgi
SERVER_ADDR: ::1
SERVER_ADMIN: webmaster@localhost
SERVER_NAME: localhost
SERVER_PORT: 80
SERVER_PROTOCOL: HTTP/1.1
SERVER_SIGNATURE: <address>Apache/2.4.10 (Debian) Server at localhost Port 80</address>

SERVER_SOFTWARE: Apache/2.4.10 (Debian)
apache.version: (2, 4, 10)
mod_wsgi.application_group: 127.0.1.1|/wsgi
mod_wsgi.callable_object: application
mod_wsgi.enable_sendfile: 0
mod_wsgi.handler_script: 
mod_wsgi.listener_host: 
mod_wsgi.listener_port: 80
mod_wsgi.path_info: 
mod_wsgi.process_group: 
mod_wsgi.request_handler: wsgi-script
mod_wsgi.request_start: 1434638287726480
mod_wsgi.script_name: /wsgi
mod_wsgi.script_reloading: 1
mod_wsgi.script_start: 1434638287727008
mod_wsgi.version: (4, 4, 13)
wsgi.errors: <_io.TextIOWrapper encoding='utf-8'>
wsgi.file_wrapper: <class 'mod_wsgi.FileWrapper'>
wsgi.input: <mod_wsgi.Input object at 0x7fd62839e6c0>
wsgi.multiprocess: True
wsgi.multithread: True
wsgi.run_once: False
wsgi.url_scheme: http
wsgi.version: (1, 0)

python version: 3.4.2 (default, Oct  8 2014, 10:47:48) 
[GCC 4.9.1]
</code></pre>
</code></pre>

##gevent/gevent-websocket

<pre><code>
$ # as root
$ #pip3 install gevent greenlet gevent-websocket
$ # python3下不能直接用pip(3)安装gevent
$ # 这样安装出来的还是python2的版本(痛苦)
$ pip3 freeze
chardet==2.3.0
colorama==0.3.2
gevent==1.0.2
gevent-websocket==0.9.5
greenlet==0.4.7
html5lib==0.999
mod-wsgi==4.4.13
requests==2.4.3
six==1.8.0
urllib3==1.9.1
wheel==0.24.0
$ pip3 uninstall gevent gevent-websocket greenlet
$ # 先安装 cython
$ apt-get install cython
$ # 到这里下载源码https://github.com/fantix/gevent
$ 然后 python3 setup.py install （as root）
$ git clone https://github.com/fantix/gevent.git
$ # or 
$ # wget https://github.com/fantix/gevent/archive/master.zip
$ python3 setup.py install
Finished processing dependencies for gevent==1.1
$ # run as root
$ pip3 install gevent-websocket
$ # 如果 from geventwebsocket.handler import WebSocketHandler 不报错即安装成功
$ # https://pypi.python.org/pypi/gevent-websocket/

$ #####################
$ # 因为mod_wsgi和gevent互不兼容，故舍弃
$ #####################
</code></pre>

## apache https

<pre><code>
$ # http://www.onlamp.com/2008/03/04/step-by-step-configuring-ssl-under-apache.html
$ # http://www.cnblogs.com/best-jobs/p/3298258.html
$ # http://www.oschina.net/question/234345_42365
$ find /usr/lib/apache2/modules/* -name "*ssl.so"
/usr/lib/apache2/modules/mod_ssl.so
$ openssl genrsa 1024 > server.key
$ openssl req -new -key server.key > server.csr
$ openssl req -x509 -days 366 -key server.key -in server.csr > server.crt
$ chmod 400 server.crt  server.csr  server.key # as root
$ # 'Listen 443' in ports.conf
$ 
$ mkdir /etc/apache2/ssl
$ cp -a server.c* /etc/apache2/ssl/
$ cp -a server.key /etc/apache2/ssl/
$ mkdir  /var/www/ssl_html
$ # chown deel /var/www/ssl_html
$ cp /var/www/html/index.html /var/www/ssl_html/
$ cp -a /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-enabled/site1-ssl.conf
$ #cp -a  /etc/apache2/ssl/server.crt /var/www/ssl_html
$ vi /etc/apache2/sites-enabled/site1-ssl.conf
NameVirtualHost *:443
< VirtualHost *:443 >
DocumentRoot /var/www/ssl_html
ErrorLog ${APACHE_LOG_DIR}/error_ssl.log
CustomLog ${APACHE_LOG_DIR}/access_ssl.log combined
SSLCertificateFile /etc/apache2/ssl/server.crt
SSLCertificateKeyFile /etc/apache2/ssl/server.key

$ vi /etc/apache2/mods-available/ssl.load
$ # check "LoadModule ssl_module /usr/lib/apache2/modules/mod_ssl.so"
$ cd /etc/apache2
$ ln mods-available/ssl.load mods-enabled/ssl.load
$ # 重启apache， 浏览器安装server.crt证书即可
</code></pre>

##apache VirtualHost

<pre><code>
$ # http://jingyan.baidu.com/article/363872ec870f6e6e4ba16feb.html
$ # 基于多端口的虚拟主机配置
$ # 80 端口 mod_wsgi (python3)
$ # 443 端口 见 ‘apache https’
$ # 8080 端口 (留给 php)

$ # 80 和 443 主机已经配置好，只需新增8080即可
$ cd /etc/apache2
$ vi ports.conf
Listen 8080
$ cp sites-available/000-default.conf sites-available/site_8080.conf
$ mkdir /var/www/8080_html
$ vi sites-available/site_8080.conf
<VirtualHost *:8080>
DocumentRoot /var/www/8080_html
ErrorLog ${APACHE_LOG_DIR}/error_8080.log
CustomLog ${APACHE_LOG_DIR}/access_8080.log combined
ln sites-available/site_8080.conf sites-enabled/site_8080.conf
$
$
$
$ # port 80:
$ # 在 wsgi.conf 中删除之前配置的一行：
$ # 因为在这里配置等于是全局配置，会影响所有虚拟主机站点
$ vi /etc/apache2/mods-available/wsgi.conf
WSGIScriptAlias /wsgi "/var/www/html/application.py"
$ # 改为在80端口的个虚拟主机上配置
$ cd /etc/apache2
$ vi sites-enabled/000-default.conf
WSGIScriptAlias / "/var/www/html/application.py"
$ # 其它端口同理可配
</code></pre>

##安装 php

<pre><code>
$ # http://www.laozuo.org/3423.html
$ ...apt-get install php5 php-pear
$ #/etc/php5/apache2/php.ini
$ # mkdir /var/log/php
$ #apt-get install php5-mysql
$ # 启用mysqli支持
$ # ..
$ #service apache2 restart
</code></pre>

##安装 mysql

<pre><code>
$ # mysql 暂不安装，可以直接使用宿主机的mysql服务
$ # apt-get install mysql-server
$ #file: /etc/mysql/my.cnf
$ # mysql_secure_installation
</code></pre>