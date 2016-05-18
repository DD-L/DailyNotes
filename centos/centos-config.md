CentOS-7-x86_64-Minimal-1511
============================

## Download

```shell
# 从国内镜像站点下载
wget http://mirrors.aliyun.com/centos/7.2.1511/isos/x86_64/CentOS-7-x86_64-Minimal-1511.iso
```

## hostname

```shell
$ # Centos 7中修改hostname的方法与老版本不同
$ hostnamectl set-hostname 你要修改到的hostname
```

## Language

```shell
$ localectl set-locale LANG=要修改的语言
```

## user

```shell
$ adduser $username
$ passwd $username
```

## Set Password Quality Requirements

```shell
$ cp /etc/pam.d/system-auth /root/backup/system-auth
$ vi /etc/pam.d/system-auth
```
Find the line:

`password    requisite     pam_pwqulity.so try_first_pass local_users_only retry=3 authtok_type=`

and replace it with the following line:

`password    requisite    pam_cracklib.so min=disabled,disabled,12,8,7 local_users_only retry=3 authtok_type=`

Where,

* min=N0,N1,N2,N3,N4 – **min=disabled,disabled,12,8,7** is the password policy. Each filed (N0,N1..N4) is used for different purpose. The keyword disabled can be used to disallow passwords of a given kind regardless of their length. Each subsequent number is required to be no larger than the preceding one. N0 is used for passwords consisting of characters from one character class only. The character classes are – digits, lower-case letters, upper-case letters, and other characters.
	1. N1 is used for passwords consisting of characters from two character classes which do not meet the requirements for a passphrase.
	2. N2 is used for passphrases. A passphrase must consist of sufficient words (see the passphrase option below).
	3. N3 and N4 are used for passwords consisting of characters from three and four character classes, respectively.
	4. When calculating the number of character classes, upper-case letters used as the first character and digits used as the last character of a password are not counted.
	5. In addition to being sufficiently long, passwords are required to contain enough different characters for the character classes and the minimum length they have been checked against.
	6. **retry=3** – The number of times the module will ask for a new password if the user fails to provide a sufficiently strong password and enter it twice the first time.

See the help file /usr/share/doc/pam_passwdqc-1.0.2/README and the man page pam_passwdqc for detailed configuration options.

## ssh server

```shell
$ yum install openssh-server
$ yum update
$ vi /etc/ssh/sshd_config
# 开启sftp:
Subsystem sftp /usr/lib/openssh/sftp-server
# 好像默认即是开启的，不用更改
$ service sshd restart
```
## gcc

``` shell
$ yum install gcc
```
## git

```shell
$ yum install git
```

## Docker

更完整的方法：

* https://docs.docker.com/engine/installation/linux/centos/ [推荐]
* https://docs.docker.com/engine/installation/

```shell
$ # Docker（重新编译自 RHEL 7）已收录在 CentOS-Extras 软件库内。只须执行
$ yum install docker
$ # 安装 docker 后，你必须引导该服务才能应用它
$ systemctl start docker
$ # 若要开机时引导 docker 服务
$ systemctl enable docker
```

## vim

```shell
$ yum install vim
$ mkdir ~/.vim
$ vi ~/.vim/vimrc
set nu
set showmode
set ruler
set autoindent
syntax on

" 自动缩进
set autoindent
set cindent 
" Tab键的宽度
set tabstop=4
" 统一缩进为4 
set softtabstop=4
set shiftwidth=4
" 不要用空格代替制表符
set noexpandtab

" 侦测文件类型
filetype on
" 载入文件类型插件
filetype plugin on
" 为特定文件类型载入相关缩进文件
filetype indent on
```
## wget

```shell
$ yum install wget
```
## net_tools (ifconfig/netstat)

```
$ yum install net-tools
```

## http_load

```shell
$ # http_load 是一款体积很小的 web 压力测试工具
$ # 最新版本在这里(更新很快)： http://www.acme.com/software/http_load/
$ wget http://www.acme.com/software/http_load/http_load-09Mar2016.tar.gz
$ tar xvf http_load-09Mar2016.tar.gz
$ cd http_load-09Mar2016
$ # 如果要使其支持 https, 修改 Makefile 并编译 OpenSSL.
$ make
$ make -p /usr/local/man/man1 
$ make install
$ cd ..
$ rm -rf http_load-09Mar2016.tar.gz http_load-09Mar2016
$ man http_load
```

```shell
$ # 示例
$ cat urls.txt
http://news.baidu.com/
http://internet.baidu.com/
http://yule.baidu.com/
http://fashion.baidu.com

$ cat command 
http_load -rate 1000 -fetches 100000 -timeout 150 -proxy 192.168.33.124:8118 urls.txt


详细说明一下使用格式：

./http_load [-checksum] [-throttle] [-proxy host:port] [-verbose] [timeout secs] [-sip sip_file]

         -parallel N | -rate N [-jitter]
		 -fetches N | -seconds N
		 url_file

选项与参数：

-fetches：总计要访问url的次数，无论成功失败都记为一次，到达数量后程序退出。
-rate：每秒访问的次数（即访问频率），控制性能测试的速度。最大支持1000并发（每秒）
-seconds：工具运行的时间，到达seconds设置的时间后程序退出。
-parallel：最大并发访问的数目，控制性能测试的速度。
-verbose：使用该选项后，每60秒会在屏幕上打印一次当前测试的进度信息。
-jitter：该选项必须与-rate同时使用，表示实际的访问频率会在rate设置的值上下随机波动10%的幅度。
-checksum：由于要访问某个url很多次，为了保证每次访问时收到的服务器回包内容都一样，可以采用checksum检查，不一致会在屏幕上输出错误信息。
-cipher：使用SSL层的时候会用到此参数（url是https开头），使用特定的密码集。
-timeout：设置超时时间，以秒为单位，默认为60秒。每超过一次则记为一次超时的连接
-proxy：设置web代理(HTTP)，格式为-proxy host:port
-throttle：限流模式，限制每秒收到的数据量，单位bytes/sec。该模式下默认限制为3360bytes/sec。
-sip：指定一个source ip文件，该文件每一行都是ip+port的形式。

需要特别说明的是，-parallel参数和-rate参数中必须有一个，用于指定发请求包的方式；-fetches和-seconds两个参数必须有一个，用于指定程序的终止条件。
```