---
title:        Linux命令之du
date:         2015-08-30 20:00:00
categories:
  - Tech
  - Linux
tags:
  - Linux
  - Bash
  - du
---

# du 的基本含义和参数

`du` 用来显示文件的磁盘使用情况。

    -a 根据目录层级显示所有的文件。
    -c 显示总的大小
    -d [depth] 特定深度的所有文件大小
    -h 以 Human 可读的格式输出，自动带 B/KB/MB/GB/TB/PB
    -I mask 根据执行的权限忽略(Ignore) 文件和目录。
    -gkm 分别按 GB、KB 和 MB 为单位显示文件大小。
    -s 只显示每个特定文件的总和`-d 0`等效。

<!-- more -->

# 举个栗子

*  显示总的文件大小
> du -s

* 显示2层的文件大小
> du -h -d 2

* 显示/var/demo 文件的统计信息
> du -ah /var/demo

# du 和 df 的区别

`du` 和`df` 只有一个字母之差。du是面向文件的命令，只计算被文件占用的空间。
不计算文件系统metadata 占用的空间。df则是基于文件系统总体来计算，通过文件系统中未分配空间来确定系统中已经分配空间的大小。
df命令可以获取硬盘占用了多少空间，还剩下多少空间，它也可以显示所有文件系统对节点和磁盘块的使用情况。

df 的参数列表：

    -a 全部文件系统列表
    -h 以 Human 友好的方式显示
    -H 和-h 相似，但是大小进制为1000，而不是1024（很多 ISP、磁盘供应商都这么来）
    -l 只显示 local 的文件系统
    -t 列出文件系统类型(文件、目录，块等), Linux上为 T。
