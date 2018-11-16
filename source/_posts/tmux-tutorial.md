---
title: 那些好用的工具之 Tmux 篇
date: 2018/09/18 20:03:32
categories:
  - great tools
tags:
  - tools
---

工欲善其事，必先利其器。突然来了兴致，想利用一下睡觉前的时间，整理一下自己用过的，感觉很棒的命令行工具，作为一个系列。其中有的工具会比较复杂，有的会比较简单。复杂的，多写一些。简单的，拼盘拼成一篇。

今天的主题是 Tmux。

{% asset_img 15152533520178.jpg %}


## 一. Tmux 是什么？

[Tmux](https://github.com/tmux/tmux) 是一个终端复用工具。它可以允许在一个窗口创建、访问以及控制的多个终端。

## 二. Tmux 解决了什么问题？

* Alice SSH 登录到远程机器，对机器进行维护。她要跑一个任务耗时很久。程序已经开始跑了。但是……中间发生了一个小插曲：Alice 因为自己电脑的故障，断开了远程机器的连接。结果因为执行任务的父进程被杀掉，导致程序跑了一半，被杀掉了。Alice 又需要从头开始执行。Poor Alice...
* Bob 平时使用 Vim 开发自己的项目。他一边要编辑代码，一边要做 Debug，一边需要一个交互式 Shell 查询数据库的数据情况。于是 Bob 开了3个 Tab 窗口来回切换。他发现，这样操作太麻烦了，需要来回手动切换 Tab 才可以看到自己想看的内容。

  有小伙伴给他推荐了 iTerm2，它可以支持分割窗口。Bob 使用了一阵子后，依然觉得不爽：a. 不够灵活 b. 不小心关掉后窗口或 Tab 后，在执行的命令同样会中断。c. 切出来的窗口，停留在 Home 目录，需要手动执行`cd`命令，才能跳转到工作目录。
  {% asset_img 15152542845524.jpg %}



有了 Tmux 上面的问题，都迎刃而解。它可以：

1. 在目标机器上运行，可以后台运行，即使网络连接断开也不受影响。
2. 很方便的切换窗口，而且还可以自动切换到工作目录。
3. 支持配置，通过配置文件管理初始化会话时的窗口及执行的命令。
4. 支持 Pair Programming。


## 三. 一些概念

{% asset_img 15154155956817.jpg %}


### 1. Server

Tmux 能够让用户断连后，重新登录回来，可以保留工作现场的原因是，它本身是 C/S 架构的。Server 和 各个客户端之间通过在`/tmp/`的 socket 来进行通信。

Tmux 启动的时候，默认会创建一个Session（会话）。

### 2. Session

Session 是在 tmux 管理下的虚拟终端的集合。每个 Session 下面会有很多窗口(Window)

### 3. Window

Window 是单个可见的窗口，它有自己的编号，默认从0开始。Window 可以被分割成很多的 Pane（窗格)。

### 4. Pane

 就像一个大窗户会有很多小窗格一样，tmux 可以很方便的将窗口分割成一个个的小格子，每个格子可以称之为：Pane。
 
 他们之间的关系可以很形象地用一张图来标明：
 
 {% asset_img 15154180277103.jpg %}

### 5. Prefix Key
 
 {% asset_img 15154156612162.jpg %}


##  四. 安装

* Ubuntu

```bash
sudo apt-get install tmux
```

* macOSX

```bash
brew install tmux
```

## 五. 常用操作及配置

#### 1. 操作会话

tmux 创建会话很简单，只需要在终端输入 `tmux` 就可以，个人不推荐在日常使用中这样做。因为人比较擅长记忆有语义性的东西，建议在启动的时候指定 session 的名字: `tmux new -s sessionA`。界面启动后是这样的：

如何查看现在正在运行的 session 列表呢？`tmux ls`，它就会列出所有的 session 列表。

如果我们想 attatch 到特定的 session，我们可以执行`tmux at -t [sessionName]`。

有些 session 在创建了之后，我们不想再看到它。处女座的盆友们不能忍。执行`tmux kill -t [sessionName]`。

那如果想干掉整个 tmux 服务呢？ `tmux kill-server`。
    
    # 切换session
    bind -r ( switch-client -p 
    bind -r  ) switch-client -n


普遍用户对之不太习惯。我们可以对它做一下设置，使之符合我们的日常使用习惯。

#### 2. 通用配置

我们先看一张键盘图：

{% asset_img "ADM-3A keyboard" 15154208245016.jpg %}

是不是觉得很怪异？上图是 ADM-3A 的键盘图，实际上也是 vi 编辑器的默认的按键绑定的图。tmux 的按键同样也是用了 vi 的键盘布局。
tmux 的 Prefix 按键是`C-b`，所以为了方便使用，建议将按键改为 `C-a`，当然，建议小伙伴们将 Capslock 按键映射成 Ctrl。

    # 取消默认绑定的按键，改为 Ctrl-a
    unbind C-b
    set -g prefix C-a
    
    # 设置 prefix,防止 tmux 按键和程序按键冲突。比如c-a 在 vim 配置为全选,在运行 tmux 时，需要先按 c-a,然后再按 vim 中的 c-a
    bind C-a send-prefix
    
    # 修改默认延迟时间
    set -g escape-time 0
    
    # 设置终端颜色，有时 vim 的 colortheme 有问题，设置它可以解决
    set -g default-terminal "screen-256color"
    
    # 重新加载配置文件
    bind r source ~/.tmux.conf \; display "Configuration reloaded!"

    # 设置历史大小
    set -g history-limit 10000


#### 3. 切换窗口

    #设置 window 窗口 index 默认开始值
    set -g base-index 1
    #设置 pane 的 index 默认起始值
    set -g pane-base-index 1
    #分割窗口
    unbind '"'
    bind | split-window -h
    bind-key v split-window -h -p 50 -c "#{pane_current_path}"
    unbind %
    bind - split-window -v
    bind-key s split-window -p 50 -c "#{pane_current_path}"
    #Disable rename window name of shell command
    set-option -g allow-rename off
    # Renumber the windows of current session
    set -g renumber-windows on
    #Unbind Space. Not use layout change
    unbind Space


#### 4. 操作 Pane

    #在 pane 中移动
    bind h select-pane -L 
    bind j select-pane -D 
    bind k select-pane -U 
    bind l select-pane -R
    #调整 pane大小
    bind H resize-pane -L 5 
    bind J resize-pane -D 5 
    bind K resize-pane -U 5 
    bind L resize-pane -R 5
    #pane 转为 window
    unbind Up
    bind Up new-window -d -n tmp \; swap-pane -s tmp.1 \; select-window -t tmp
    unbind Down
    bind Down last-window \; swap-pane -s tmp.1 \; kill-window -t tmp


#### 5. 复制粘贴

#### 6. 鼠标模式

#### 7. 定制样式

#### 8. 管理会话

#### 9. 结对编程

## 六. 推荐资料

1. [Productive Mouse-Free Development By Brian P.Hogan](https://pragprog.com/book/bhtmux2/tmux-2)
2. tmux man page.
3. [ADM-3A](https://en.wikipedia.org/wiki/ADM-3A)
4. [Tmux 插件](https://github.com/tmux-plugins)
5. [Tmuxinator 管理复杂的 tmux 会话](https://github.com/tmuxinator/tmuxinator)
3. Google :)

