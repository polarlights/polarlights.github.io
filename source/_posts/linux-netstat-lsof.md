---
title:        Linux命令之netstat, lsof
date:         2015-08-30 20:00:00
categories:
  - Tech
  - Linux
tags:
  - Linux
  - netstat
  - lsof
---

# netstat

Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。

        -a (all)显示所有选项，默认不显示LISTEN相关
        -t (tcp)仅显示tcp相关选项
        -u (udp)仅显示udp相关选项
        -n 拒绝显示别名，能显示数字的全部转化成数字。
        -l 仅列出有在 Listen (监听) 的服務状态

        -p 显示建立相关链接的程序名
        -r 显示路由信息，路由表
        -e 显示扩展信息，例如uid等
        -s 按各个协议进行统计
        -c 每隔一个固定时间，执行该netstat命令。

## 举个例子

# lsof

lsof（list open files）是一个列出当前系统打开文件的工具。在linux环境下，任何事物都以文件的形式存在，
通过文件不仅仅可以访问常规数据，还可以访问网络连接和硬件。

不过 lsof 一般是以 Root 权限运行的。

<!-- more -->

## 举个例子

* 查看某个进程打开文件数量

    > ls -l /proc/[pid]/fd | wc -l [Linux]

    > lsof -p [pid] | wc -l [Mac]

* 查看谁在使用某个文件

    > lsof /var/demo.txt

* 恢复已删除文件

当Linux计算机受到入侵时，常见的情况是日志文件被删除，以掩盖攻击者的踪迹。管理错误也可能导致意外删除重要的文件，比如在清理旧日志时，意外地删除了数据库的活动事务日志。有时可以通过lsof来恢复这些文件。

当进程打开了某个文件时，只要该进程保持打开该文件，即使将其删除，它依然存在于磁盘中。这意味着，进程并不知道文件已经被删除，它仍然可以向打开该文件时提供给它的文件描述符进行读取和写入。除了该进程之外，这个文件是不可见的，因为已经删除了其相应的目录索引节点。

在/proc 目录下，其中包含了反映内核和进程树的各种文件。/proc目录挂载的是在内存中所映射的一块区域，所以这些文件和目录并不存在于磁盘中，因此当我们对这些文件进行读取和写入时，实际上是在从内存中获取相关信息。大多数与 lsof 相关的信息都存储于以进程的 PID 命名的目录中，即 /proc/1234 中包含的是 PID 为 1234 的进程的信息。每个进程目录中存在着各种文件，它们可以使得应用程序简单地了解进程的内存空间、文件描述符列表、指向磁盘上的文件的符号链接和其他系统信息。lsof 程序使用该信息和其他关于内核内部状态的信息来产生其输出。所以lsof 可以显示进程的文件描述符和相关的文件名等信息。也就是我们通过访问进程的文件描述符可以找到该文件的相关信息。

当系统中的某个文件被意外地删除了，只要这个时候系统中还有进程正在访问该文件，那么我们就可以通过lsof从/proc目录下恢复该文件的内容。 假如由于误操作将/var/log/messages文件删除掉了，那么这时要将/var/log/messages文件恢复的方法如下：

首先使用lsof来查看当前是否有进程打开/var/logmessages文件，如下： 

> lsof |grep /var/log/messages

syslogd 1283 root 2w REG 3,3 5381017 1773647 /var/log/messages (deleted)

从上面的信息可以看到 PID 1283（syslogd）打开文件的文件描述符为 2。同时还可以看到/var/log/messages已经标记被删除了。因此我们可以在 /proc/1283/fd/2 （fd下的每个以数字命名的文件表示进程对应的文件描述符）中查看相应的信息，如下：

> head -n 10 /proc/1283/fd/2

> cat /proc/1283/fd/2 > ~/restored_messages


* 显示开启文件abc.txt的进程

> lsof abc.txt


* 显示22端口现在被什么程序占用

> lsof -i 22


* 显示abc进程现在正在打开的文件

> lsof -c abc


* 显示归属gid的进程情况

> lsof -g gid

* 显示指定目录下被进程开启的文件，不会遍历该目录下的所有子目录

> lsof +d /usr/local/


* 显示指定目录下被进程开启的文件，会遍历该目录下得所有子目录

> lsof +D /usr/local/


* 显示使用fd为4的进程

> lsof -d 4


* 不进行域名解析，缺省会进行，比较慢

> lsof -n


* 查看进程号为12的进程打开了哪些文件

> lsof -p 12


* 让lsof重复执行，缺省15s刷新

> lsof +|-r [t]

-r, lsof会永远执行，直到被中断

+r, lsof会一直执行，直到没可显示的内容

Example：

查看目前ftp连接的情况：lsof -i tcp@test.com:ftp -r


* 列出打开文件的大小，如果大小为0，则空

> lsof -s


* 以UID，列出打开的文件

> lsof -u username
