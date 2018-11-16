---
title:        Linux命令之bg、fg、jobs 和 Ctrl-Z, Ctrl-C
date:         2015-08-30 20:00:00
categories:
  - Tech
  - Linux
tags:
  - Linux
  - bg
  - fg
  - jobs
---
# Ctrl-C, Ctrl-D, Ctrl-Z 的区别

在 Linux 的日常使用中，`Ctrl+C` 应该是用的最多的，他的用途是终止当前进程。那么`Ctrl+Z` 和`Ctrl+D` 又有什么用途么？

`Ctrl+Z` 表示暂停一个进程，`Ctrl+D` 表示文件结束符(EOF)。

假如我们有一个会长期执行的程序，如果它原来就是在前台运行的话(`bundle exec sidekiq`)，
如果使用`Ctrl+Z`，会在终端输出`susppended bundle exec sidekiq`。当然 Ctrl+D 是不起作用的，因为它的
应用场景不是这样的。被暂停的进程可以使用`ps -ef | grep [进程名]`看到。Ctrl+D 更多地
在文件操作上，每个文件都有对应的标志(EOF)表示文件的结束。

# fg、bg 和 jobs

`fg %[job num]`把一个任务从后台拿到前台来处理.foreground
`bg %[job num]` 把一个任务从前台拿到后台去执行。background

那么 `job num` 怎么看呢？ 使用`jobs -l`, 最左侧的数字即是。比如有一个后台进程job id 为2，那么执行`fg %2`即可。

如何将暂停的进程在后台继续执行呢？执行`bg %[job num]` 就可以了。如果有多个的话，使用`bg`(没有 job num)。
