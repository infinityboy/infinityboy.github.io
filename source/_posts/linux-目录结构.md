---
title: linux 目录结构
date: 2019-06-03 20:43:54
tags: Linux
categories: 
- 后端
---


>本篇博客仅以个人学习记录使用

1. /  根目录 
每一个文件和目录都是从根目录开始，但是只有root用户具有该目录下的写权限，这里需要注意的是/root是root用户的主目录，这与/.不一样，cd /. 或者 cd / 进入根目录 使用ll命令即可看到/root目录。 cd ~ 及为进入root目录
2. ~ root目录
3. /bin中 - 用户的二进制文件，包含了二进制的可执行文件，在单用户模式下，需要使用的常见linux命令都位于此目录下。系统的所有用户使用的命令都在这里，例如：ps, cp
4. /sbin - 系统的二进制文件，就行/bin，/sbin同样也包含二进制可执行文件。但是，在这个目录下的linux命令通常由管理员使用，对系统进行维护，例如：iptables、reboot、fdisk、ifconfig、swapon命令
5. /etc - 配置文件 包含了所有程序所需的配置文件，也包含了用于启动/停止单个程序的启动和关闭shell脚本。例如：/etc/resolv.conf、/etc/logrotate.conf。重要的配置文件有 /etc/inittab、/etc/fstab、/etc/init.d、/etc/X11、/etc/sysconfig、/etc/xinetd.d修改配置文件之前记得备份。注：/etc/X11 存放与 x windows 有关的设置。
6. /dev - 设备文件，包括设备文件。这些包括终端设备，usb或连接到系统的任何设备。例如：/dev/tty1、/dev/usbmon0。访问该目录下某个文件，相当于访问某个设备，常用的是挂载光驱 mount /dev/cdrom /mnt
7. /pro - 进程信息，包含系统进程的相关信息。这是一个虚拟的文件系统，包含有关正在运行的进程的信息。例如：/proc/{pid}目录中包含的与特定pid相关的信息。系统资源都是以文本形式存在的。例如：/proc/uptime
8. /var - 变量文件，这个目录可以找到内容可能增长的文件，这包括:系统日志文件(/var/log);包和数据库文件(/var/lib);电子邮件(/var/mail);打印队列(/var/spool);锁文件(/var/lock);多次重新启动需要的临时文件(/var/tmp)。某些大文件的溢出区，比方说各种服务的日志文件
9. /tmp - 临时文件,包含系统和用户创建的临时文件，当系统重新启动时，这个目录下的文件都将被删除
10. /usr - 用户程序，包含二进制文件、库文件、文档和二级程序的源代码。

    /usr/bin中包含用户程序的二进制文件。如果你在/bin中找不到用户二进制文件，到usr/bin目录看看。例如：at、awk、cc、less、scp。
    
    /usr/sbin中包含系统管理员的二进制文件。如果你在/sbin中找不到系统二进制文件，到/usr/sbin目录看看。例如：atd、cron、sshd、useradd、userdel。
    
    /usr/lib中包含了/usr/bin和/usr/sbin用到的库。
    
    /usr/local中包含了从源安装的用户程序。例如，当你从源安装Apache，它会在/usr/local/apache2中
    
11. /home - HOME目录。 所有用户用home目录来存储他们的个人档案。 例如：/home/john、/home/nikita。系统默认的用户家目录，新增用户账号时，用户的家目录都存放在此目录下，~表示当前用户的家目录，~edu 表示用户 edu 的家目录。建议单独分区，并设置较大的磁盘空间，方便用户存放数据
12. /boot - 引导加载程序文件。包含引导加载程序相关的文件（也就是linux系统启动时的一些文件）。内核的initrd、vmlinux、grub文件位于/boot下。例如：initrd.img-2.6.32-24-generic、vmlinuz-2.6.32-24-generic
13. /lib - 系统库。包含支持位于/bin和/sbin下的二进制文件的库文件.库文件名为 ld*或lib*.so.*例如：ld-2.11.1.so，libncurses.so.5.7
14. /opt - 可选的附加应用程序。opt代表可选的，包含从个别厂商的附加应用程序。附加应用程序应该安装在/opt/或者/opt/的子目录下。给主机额外安装软件所摆放的目录。如：FC4使用的Fedora 社群开发软件，如果想要自行安装新的 KDE 桌面软件，可以将该软件安装在该目录下。以前的 Linux 系统中，习惯放置在 /usr/local 目录下。
15. /mnt - 挂载目录。临时安装目录，系统管理员可以挂载文件系统。系统提供这个目录是让用户临时挂载其他的文件系统
16. /media - 可移动媒体设备。用于挂载可移动设备的临时目录。举例来说，挂载CD-ROM的/media/cdrom，挂载软盘驱动器的/media/floppy;
17. /srv - 服务数据。srv代表服务。包含服务器特定服务相关的数据。例如，/srv/cvs包含cvs相关的数据。服务启动之后需要访问的数据目录，如 www 服务需要访问的网页数据存放在 /srv/www 内
18. /lost+fount：系统异常产生错误时，会将一些遗失的片段放置于此目录下，通常这个目录会自动出现在装置目录下。如加载硬盘于 /disk 中，此目录下就会自动产生目录 /disk/lost+found。这个目录平时是空的，系统非正常关机而留下“无家可归”的文件（windows下叫什么.chk）就在这里
19. /mnt: /media：光盘默认挂载点，通常光盘挂载于 /mnt/cdrom 下，也不一定，可以选择任意位置进行挂载。
20. /proc：此目录的数据都在内存中，如系统核心，外部设备，网络状态，由于数据都存放于内存中，所以不占用磁盘空间，比较重要的目录有 /proc/cpuinfo、/proc/interrupts、/proc/dma、/proc/ioports、/proc/net/* 等。虚拟的目录，是系统内存的映射。可直接访问这个目录来获取系统信息

##### 一切皆文件
Linux 对数据文件(.mp3、.bmp)，程序文件(.c、.h、*.o)，设备文件（LCD、触摸屏、鼠标），网络文件( socket ) 等的管理都抽象为文件，使用统一的方式方法管理。
###### 文件分类：
1）普通文件( 数据文件 )：普通文件是用于存放数据、程序等信息的文件，一般都长期地存放在外存储器(磁盘)中。普通文件又分为文本文件和二进制文件。
2）目录文件：目录文件是文件系统中一个目录所包含的目录项所组成的文件。
3）设备文件：设备文件是用于为操作系统与设备提供连接的一种文件。在linux系统中将设备作为文件来处理，操作设备就像操作普通文件一样。每一个设备对应一个设备文件。存放在/dev目录中。
4）连接文件：似于windows下的快捷方式，链接又可以分为软连接(符号链接)和硬链接
5）管道文件：管道文件主要用于在进程间传递数据的一种特殊文件
6）套接口文件：主要用于不同计算机网络通信的一种特殊文件

##### linux软件安装
Linux 的软件安装目录是也是有讲究的，理解这一点，在对系统管理是有益的

/usr：系统级的目录，可以理解为C:/Windows/，/usr/lib理解为C:/Windows/System32。
/usr/local：用户级的程序目录，可以理解为C:/Progrem Files/。用户自己编译的软件默认会安装到这个目录下。
/opt：用户级的程序目录，可以理解为D:/Software，opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

源码放哪里？
/usr/src：系统级的源码目录。
/usr/local/src：用户级的源码目录

```
/opt

Here’s where optional stuff is put. Trying out the latest Firefox beta? Install it to /opt where you can delete it without affecting other settings. Programs in here usually live inside a single folder whick contains all of their data, libraries, etc.
这里主要存放那些可选的程序。你想尝试最新的firefox测试版吗?那就装到/opt目录下吧，这样，当你尝试完，想删掉firefox的时候，你就可 以直接删除它，而不影响系统其他任何设置。安装到/opt目录下的程序，它所有的数据、库文件等等都是放在同个目录下面。
举个例子：刚才装的测试版firefox，就可以装到/opt/firefox_beta目录下，/opt/firefox_beta目录下面就包含了运 行firefox所需要的所有文件、库、数据等等。要删除firefox的时候，你只需删除/opt/firefox_beta目录即可，非常简单。
```

```
/usr/local

This is where most manually installed(ie. outside of your package manager) software goes. It has the same structure as /usr. It is a good idea to leave /usr to your package manager and put any custom scripts and things into /usr/local, since nothing important normally lives in /usr/local.

这里主要存放那些手动安装的软件，即不是通过“新立得”或apt-get安装的软件。它和/usr目录具有相类似的目录结构。让软件包管理器来管理/usr目录，而把自定义的脚本(scripts)放到/usr/local目录下面，我想这应该是个不错的主意。
```
