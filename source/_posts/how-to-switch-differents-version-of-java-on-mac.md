---
title: Switch java versions on Mac OSX
date: 2017/12/27 21:39:52
categories:
  - Mac
  - Java
tags:
  - Java
---

最近公司部分产品开始从 Ruby 转到 Java，这应该是很多创业公司走的路吧，在业务变化快速阶段，希望可以使用高效率的开发套件；而到了业务相对稳定以及用户量上升之后，希望可以保证它的稳定性以及高性能。当然最主要的原因是招 Ruby 的小伙伴还是比较有难度. 233.

比较欣喜的是在 Java 9 中终于有了 Jshell，可以像 Ruby 那样动态调试代码；不过我还是最爱 Ruby On Rails，可以更快地去关注自己的业务，不用啰嗦地去写很多冗余分层的代码。但是我们的核心代码还是使用 Java 8 进行开发，偶尔还需要使用 JShell。

搞定 java 都需要设置环境变量 `JAVA_HOME` 的，如果要切换 Java 版本就需要修改 `JAVA_HOME` 的值。有没有比较好的方式来操作呢？

当然首先想到的是使用 alias，动态去搞定 Java 的环境变量，比如像这样:

```bash
alias j9="export JAVA_HOME='/Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home'"
alias j8="export JAVA_HOME='/Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home'"
```

这样可以解决问题，但是呢，我们要追求优雅，避免写 HARD CODE。

有没有更好的方法呢？答案必然是：有！

在 Mac OS X Leopard[^java_home] 及以后版本，在 `/usr/libexec/` 开始有了一个命令叫 `java_home`[^apple_java_home]。 该命令默认返回当前系统中安装的最高版本的 Java 版本。

我们先看它支持的参数及含义：

```bash
> /usr/libexec/java_home -h
Usage: java_home [options...]
    Returns the path to a Java home directory from the current user's settings.

Options:
    [-v/--version   <version>]       Filter Java versions in the "JVMVersion" form 1.X(+ or *).
    [-a/--arch      <architecture>]  Filter JVMs matching architecture (i386, x86_64, etc).
    [-d/--datamodel <datamodel>]     Filter JVMs capable of -d32 or -d64
    [-t/--task      <task>]          Use the JVM list for a specific task (Applets, WebStart, BundledApp, JNI, or CommandLine)
    [-F/--failfast]                  Fail when filters return no JVMs, do not continue with default.
    [   --exec      <command> ...]   Execute the $JAVA_HOME/bin/<command> with the remaining arguments.
    [-R/--request]                   Request installation of a Java Runtime if not installed.
    [-X/--xml]                       Print full JVM list and additional data as XML plist.
    [-V/--verbose]                   Print full JVM list with architectures.
    [-h/--help]                      This usage information.
```

对于我们比较有意义的是`-v`和`-V`两个参数。`-V`返回当前系统中安装的 Java 版本，`-v` 返回指定版本的 Java 的所在目录。

```bash
╰ ➤ /usr/libexec/java_home -V
Matching Java Virtual Machines (2):
    9.0.1, x86_64:      "Java SE 9.0.1" /Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home
    1.8.0_152, x86_64:  "Java SE 8"     /Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home

# java_home -v
╰ ➤ /usr/libexec/java_home -v 9
/Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home

╰ ➤ /usr/libexec/java_home -v 1.8
/Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home
```

现在已经知道了动态获取指定版本 Java 所在目录的方法，那么对之前的 `alias` 做一下简单重构：

```bash
alias j9="export JAVA_HOME=`/usr/libexec/java_home -v 9`"
alias j8="export JAVA_HOME=`/usr/libexec/java_home -v 1.8`"
```

现在，我们可以愉快地和多个 Java 版本玩耍了。

[^java_home]: https://iengchen.github.io/2016/07/16/best-practice-to-set-java-home-environment-variable-on-mac-osx/
[^apple_java_home]: https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/java_home.1.html



