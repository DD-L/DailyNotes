debian 7.8:(netinst)
===========

###download

一个小型安装映像：可以被很快的下载到您的计算机，并须要把它复制到可删除的媒介，如光盘、U 盘等。安装过程中，您的计算机需要互联网连线。

http://cdimage.debian.org/debian-cd/7.8.0/amd64/iso-cd/debian-7.8.0-amd64-netinst.iso

via https://www.debian.org/distrib/

硬件配置(VM)
========
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
=======

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
=======================================

<pre><code>
apt-get update
apt-get install libc6
</code></pre>

/etc/profile
============

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
====

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
========

<pre>
apt-get install dos2unix
</pre>

git
===

<pre>
apt-get install git
</pre>


.vimrc
======

<pre>
apt-get install vim
https://github.com/amix/vimrc
install Awesome version:
apt-get install ctags
git clone git://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
</pre>




arm-test
========

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
====

<pre>
sudo apt-get install tree
</pre>

debootstrap
===========

<pre>
sudo apt-get install debootstrap

# debootstrap --arch [平台] [发行版本代号] [目录]
# debootstrap --arch i386 trusty /mnt
debootstrap --foreign --arch mipsel wheezy ./debian
</pre>