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
$ # 开启sftp:
$ Subsystem sftp /usr/lib/openssh/sftp-server
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