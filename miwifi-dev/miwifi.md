1. 相关文章的链接:
==================
<pre>
http://www.openwrt.org.cn/bbs/forum.php?mod=viewthread&tid=2801
(用debootstrap在openwrt上安装debian squeeze，构建编译环境)

http://jingyan.baidu.com/article/2d5afd69f2c1bc85a3e28e7c.html
http://www.miui.com/thread-2093928-1-1.html

http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages
(opkg相关)
</pre>

2. 说明
=======
因为那个交叉编译环境，对在板子上安装部署应用很不方便，所以最好在开发板上也构建一个编译环境（开发版那个小心脏啊，可能会受不了）

3.操作记录：
============

#####3.1 修改opkg源
<pre><code>
$ cp -a /etc/opkg.conf /etc/opkg.conf.old
$ cat /etc/opkg.conf.old
src/gz attitude_adjustment http://downloads.openwrt.org/attitude_adjustment/12.09/ramips/mt7620a/packages
dest root /data
dest ram /tmp
lists_dir ext /data/var/opkg-lists
option overlay_root /data
$ # 修改 /etc/opkg.conf 如下(因为板子空间小，所以就用装在usb扩展上)
$ cat /etc/opkg.conf
src/gz barrier_breaker_base http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/base
src/gz barrier_breaker_luci http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/luci
src/gz barrier_breaker_management http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/management
src/gz barrier_breaker_oldpackages http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/oldpackages
src/gz barrier_breaker_packages http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/packages
src/gz barrier_breaker_routing http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/routing 
src/gz barrier_breaker_telephony http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/telephony
dest root /extdisks/sda1/miwifi/opkg
dest ram /tmp
lists_dir ext /extdisks/sda1/miwifi/opkg/var/opkg-lists
option overlay_root /extdisks/sda1/miwifi/opkg
dest usb /extdisks/sda1/miwifi/opkg
$ mkdir -p /extdisks/sda1/miwifi/opkg/var /extdisks/sda1/miwifi/download
$ cd /extdisks/sda1/miwifi/opkg/download
$ opkg update
$ opkg download libgcc libcurl libevent2 libopenssl libpthread librt libpolarssl zlib
$ # wget http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/base/libc_0.9.33.2-1_ramips_24kec.ipk
$ opkg -d usb install *.ipk # or like: opkg -d usb install libc
$ opkg list-installed
</code></pre>

#####3.2 配置debian debootstrap
相关软件： debootstrap binutils objdump libbfd
<pre><code>
$ # 因为debootstrap安装debian时，需要用的dpkg或者ar来解包deb软件包，考虑到dpkg安装需要的空间较大，转用ar,而ar在binutils软件包中，ar的运行需要libbfd库支持。因此先安装objdump，连支持库libbfd也安装了
$ opkg update
$ opkg -d usb install objdump
$ # 解包binutils，获得ar可执行文件
$ # opkg安装binutils需要6M多的空间，而我们只需要其中一个ar,所以我只解包取出其中的ar文件
$ mkdir /extdisks/sda1/miwifi/download/tmp
$ cd /extdisks/sda1/miwifi/download/tmp
$ wget http://downloads.openwrt.org/barrier_breaker/14.07/ramips/mt7620a/packages/base/binutils_2.24-2_ramips_24kec.ipk
$ tar zxvf binutils_2.24-2_ramips_24kec.ipk
$ tar zxvf data.tar.gz
$ cp -a ./usr/bin/ar /usr/bin
cp: can't create '/usr/bin/ar': Read-only file system
$ # 没办法，只能全部安装在 usb 上了
$ opkg -d usb install binutils_2.24-2_ramips_24kec.ipk
$ cd /extdisks/sda1/miwifi/opkg/usr/bin
$ ./ar
./ar: can't load library 'libbfd-2.24.so'
$ 配置$PATH 如下
$ cat /etc/profile
#!/bin/sh
[ -f /etc/banner ] && cat /etc/banner

export PATH=/bin:/sbin:/usr/bin:/usr/sbin
export HOME=$(grep -e "^${USER:-root}:" /etc/passwd | cut -d ":" -f 6)
export HOME=${HOME:-/root}
export PS1='\u@\h:\w\$ '

[ -x /bin/more ] || alias more=less
[ -x /usr/bin/vim ] && alias vi=vim || alias vim=vi

[ -z "$KSH_VERSION" -o \! -s /etc/mkshrc ] || . /etc/mkshrc

[ -x /usr/bin/arp ] || arp() { cat /proc/net/arp; }
[ -x /usr/bin/ldd ] || ldd() { LD_TRACE_LOADED_OBJECTS=1 $*; }

# added by D_L
alias 'll=ls -al --color'
alias '..=cd ..'
alias '...=cd ../..'

export MY_OPKG_PATH=/extdisks/sda1/miwifi/opkg
export PATH=$PATH:$MY_OPKG_PATH/bin:$MY_OPKG_PATH/usr/bin:$MY_OPKG_PATH/usr/sbin:$MY_OPKG_PATH/sbin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MY_OPKG_PATH/lib:$MY_OPKG_PATH/usr/lib

$ # 因为没有source命令，只能重新登录shell, 验证ar 命令执行成功
$ 下一步 安装 debootstrap
$ 
$ cd /extdisks/sda1/miwifi/download
$ opkg update
$ opkg -d usb install debootstrap

$ #/usr 只读，debootstrap 无法执行, 需要修改 mount
$ # 文章： http://wiki.openwrt.org/zh-cn/doc/uci/fstab
$ #需要将 / type SquashFS --> JFFS2 （放弃）
$ # 或者将直接修改 /dev/root -> /dev/mtdblock5 (放弃) 

$ # 还有一个方法：http://www.right.com.cn/forum/thread-101737-1-1.html
$ # 在电脑上安装debootstrap, 然后下载对应版本的debian,放到openwrt上去
$ # 回到电脑主机上：
$ mkdir /tmp/debian_mips
$ cd /tmp/debian_mips
$ sudo apt-get install debootstrap
$ # 提示说 debootstrap  安装失败，找不到http://cn.archive.ubuntu.com/ubuntu/pool/main/d/debootstrap/debootstrap_1.0.53ubuntu0.2_all.deb
$ # 于是手动下载版本接近的debootstrap
$ wget http://mirrors.aliyun.com/ubuntu/pool/main/d/debootstrap/debootstrap_1.0.59ubuntu0.3_all.deb
$ sudo dpkg -i debootstrap_1.0.59ubuntu0.3_all.deb
$ #debootstrap 安装成功 debootstrap --help
$ sudo debootstrap --arch mips --foreign squeeze ./debian http://ftp.tw.debian.org/debian/
$# sudo debootstrap --arch=mips ---variant=minbase --foreign squeeze ./debian http://ftp.tw.debian.org/debian/
$ # 其中：arch后面指定目标设备的体系结构，我是ar7241因此是mips，其它的小端的是mipsel。 foreign代表我们所下载的debian与现在所用的机器是不同的结构（在PC上下载mips版的，如果是在openwrt上直接执行debootstrap则不需要这个参数）.squeeze是现在debian的发行版，以前的lenny和etch都不能下了，请用这个squeeze，这是最新的。./是下载地址，后面是服务器地址，以前有教程给出的http://mirrors.geekbone.org/debian/ 这个地址已经不行了。经测试台湾的这个地址速度还不错，也可以选择其它镜像地址，这里是debian的镜像列表:http://www.debian.org/mirror/list 请注意选择architectures后面有mips/mipsel的地址。下载好后打个包：tar -czf debian.tar.gz ./debian
$ # 等待N久
$ sudo tar -czvf debian.tar.gz ./debian

$ #然后将 debian.tar.gz 传到板子上:
$ #在板子上执行:
$ cd /extdisks/sda1/miwifi
$ tar zxvf debian.tar.gz

$ #下面几行没有执行，因为debian-chroot计划失败,为了资料的完整性，列出后面的步骤：
···
$ mount -o bind /proc /extdisks/sda1/miwifi/debian/proc 
$ mount -o bind /dev /extdisks/sda1/miwifi/debian/dev
$ chroot /extdisks/sda1/miwifi/debian /bin/bash

$ # 第一次还是要先执行下debootstrap完成安装：
$ /debootstrap/debootstrap --second-stage
···

$ ############
$ 在板子上安装debootstrap的计划失败，原因是没有找到对应编译器编译的debootstrap, 所以chroot也失败了
$ 于是，还是老老实实的用交叉编译吧，由于小米提供的32位编译器，编译出来的二进制文件没能在板
$ 子上跑起来， 所以就安装个64-bit的debian系统，配置一下，可用。
$ -----
$ 自此，gcc开发环境搭建告一段落
$ ############


$ ##########
$ # （接下来这几行，为了丰富知识库而存在, 并没有什么卵用。
$ # 实际当中并没有用到这些，是这些因为，随后debootstrap安装成功了）
$ # 
$ # 下一步， 安装 python2.x 和 python 3.x
$ 
$ # Python 2.7.10 src
$ # https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tar.xz
$ # Python 3.4.3 src
$ # https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tar.xz
$ # 下载到debian上， 方法见 http://www.cnblogs.com/hit-python/p/4079506.html & http://www.cnblogs.com/hit-python/articles/4081673.html
$ 
$ # python 移植先缓一缓，如果类似的这些软件都在交叉编译环境下部署的话，
$ # 将是一个很累人的过程，如果能搭建好板子上的debootstrap环境，
$ # 会省好多力气， 虽然我的板子只有1核CPU(580MHz), 128MB内存。
$ ########### 

$ ##########
$ # 重启 debootstrap 计划 #
$ ##########

$ # 资料：
$ # https://wiki.debian.org/DebianBootstrap
$ # 板子上的系统为小端。（mips 大，mipsel小）
$ #大端系统链接：（没有用，因为板子原有系统是小端）
$ # (wheezy/bootstrap-base) https://packages.debian.org/zh-cn/wheezy/bootstrap-base -> https://packages.debian.org/zh-cn/wheezy/mips/bootstrap-base/download
$ # 小端（略）  
$ # 源码：
$ # (source/wheezy/debootstrap) https://packages.debian.org/zh-cn/source/wheezy/debootstrap
$ # (交叉编译文档) https://wiki.debian.org/DebianBootstrap#Cross-building
$ # 以上这些资料都没有鸟用

$ # 之前把板子上的平台给搞错了，所以才一直失败：
$ # 在 debian 上重新执行
$ sudo apt-get install debootstrap
$ debootstrap --foreign --arch mipsel wheezy ./debian
$ # 同样等待N久， 
$ # 关于debootstrap download的资料： http://www.latelee.org/using-gnu-linux/ubuntu-debootstrap.html
$ # # debootstrap --arch [平台] [发行版本代号] [目录]
$ 下载完成后，打包传到板子上。 解压后： /extdisks/sda1/miwifi/debian/
$ mount -o bind /proc /extdisks/sda1/miwifi/debian/proc 
$ mount -o bind /dev /extdisks/sda1/miwifi/debian/dev
$ chroot /extdisks/sda1/miwifi/debian /bin/bash
$ I have no name!@XiaoQiang:/
$ # 成功 ！！！！
$ # 但第一次还是要执行下安装程序：
$ /debootstrap/debootstrap --second-stage
</code></pre>
#####3.3 配置chroot环境
<pre><code>
$ apt-get install gcc
$ apt-get install g++
$ # 安装python2
$ apt-get install python-pip
$ apt-get install python-dev
$ ll /usr/bin/pip /usr/bin/python
lrwxrwxrwx 1 root root 7 Jun 23  2012 /usr/bin/pip -> pip-2.7
lrwxrwxrwx 1 root root 9 Sep 28  2013 /usr/bin/python -> python2.7 
$ # 安装python3
$ apt-get install python3-pip
$ apt-get install python3-dev
$ ll /usr/bin/pip* /usr/bin/python*
lrwxrwxrwx 1 root root       7 Jun 23  2012 /usr/bin/pip -> pip-2.7
-rwxrwxrwx 1 root root     286 Jun 23  2012 /usr/bin/pip-2.6
-rwxrwxrwx 1 root root     283 Jun 23  2012 /usr/bin/pip-2.7
-rwxrwxrwx 1 root root     286 Jun 23  2012 /usr/bin/pip-3.2
lrwxrwxrwx 1 root root       9 Sep 28  2013 /usr/bin/python -> python2.7
lrwxrwxrwx 1 root root      16 Sep 28  2013 /usr/bin/python-config -> python2.7-config
lrwxrwxrwx 1 root root       9 Sep 28  2013 /usr/bin/python2 -> python2.7
-rwxrwxrwx 1 root root 2562324 Jan 28  2013 /usr/bin/python2.6
-rwxrwxrwx 1 root root 2808532 Mar 14  2014 /usr/bin/python2.7
-rwxrwxrwx 1 root root    1652 Mar 14  2014 /usr/bin/python2.7-config
lrwxrwxrwx 1 root root       9 Oct 21  2012 /usr/bin/python3 -> python3.2
lrwxrwxrwx 1 root root      16 Oct 21  2012 /usr/bin/python3-config -> python3.2-config
lrwxrwxrwx 1 root root      11 Feb 20  2013 /usr/bin/python3.2 -> python3.2mu
lrwxrwxrwx 1 root root      18 Feb 20  2013 /usr/bin/python3.2-config -> python3.2mu-config
-rwxrwxrwx 1 root root 3018712 Feb 20  2013 /usr/bin/python3.2mu
-rwxrwxrwx 1 root root    1870 Feb 20  2013 /usr/bin/python3.2mu-config
lrwxrwxrwx 1 root root      11 Oct 21  2012 /usr/bin/python3mu -> python3.2mu
lrwxrwxrwx 1 root root      18 Oct 21  2012 /usr/bin/python3mu-config -> python3.2mu-config

</code></pre>
#####3.4 chroot
<pre><code>
mount -o bind /proc /extdisks/sda1/miwifi/debian/proc
mount -o bind /dev /extdisks/sda1/miwifi/debian/dev
chroot /extdisks/sda1/miwifi/debian /bin/bash
# chroot /extdisks/sda1/miwifi/debian touch /tmp/123
</code></pre>
#####3.5 开机脚本 /etc/rc.local
<pre><code>
*** chroot_script.sh ***
$ cat chroot_script.sh 
#! /bin/ash 
# chroot-script (with root)

# mount
mount | grep "/extdisks/sda1/miwifi/debian/proc" >/dev/null 2>&1
if [ $? -ne 0 ]
then
	mount -o bind /proc /extdisks/sda1/miwifi/debian/proc
fi
mount | grep "/extdisks/sda1/miwifi/debian/dev" >/dev/null 2>&1
if [ $? -ne 0 ]
then
	mount -o bind /dev /extdisks/sda1/miwifi/debian/dev
fi

# chroot
chroot /extdisks/sda1/miwifi/debian /bin/bash
</code></pre>