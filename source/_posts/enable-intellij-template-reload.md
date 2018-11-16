---
title: Enable reload chagned java and template files In IntelliJ
date: 2018/01/06 17:12:09
categories:
  - Intellij
  - Java
tags:
  - Java
---

最近在学习 Spring Boot 的时候发现真心不如 Ruby on Rails 爽，其中一点是：RoR 在 开发模式下默认可以自动 reload 相关路径下的文件：

```bash
$ bin/rails r 'puts ActiveSupport::Dependencies.autoload_paths'
.../app/assets
.../app/controllers
.../app/helpers
.../app/mailers
.../app/models
.../app/controllers/concerns
.../app/models/concerns
.../test/mailers/previews
```

在 `app/views`下的文件在修改后也会立即产生效果。

但是在 Java 世界里，就没有全家桶了。开始的时候很不习惯，需要各种重启，每次重启8-10秒，几乎时时刻刻在和重启打交道。

还好在 Java 世界里是有解决方法的。

#### *.java

java 文件可以使用 `spring-boot-devtools`，但是，但是，任何在`classpath`中的文件的变动，都会引起整个程序重启。是的，没看错，是重启。233。好消息是：静态文件和模板文件的变动不会引起程序重启。

它不会提高太多效率，因为我们需要的是重新加载，而不是重启。

还好我们有`JRebel`，在 IntelliJ 的 "Perferences" → "Plugins" → “Browse Repositories"，搜索`JRebel`，安装插件后重启。安装成功后，在`Run`菜单下，会多出两个子菜单：a. 使用 JRebel 启动程序。 b. 使用 JRebel 调试程序。

使用 JRebel 启动程序后，修改源文件，保存后，可以在标准输出看到`JRebel: Reloading class 'io.polarlights.web.HomeController'.`这样的字样，说明它在正常工作了。

#### templates

1. 启用自动构建项目

File –> Setting –> Build, Execution, Deployment –> Compiler –> check **Build project automatically**

2. 启用执行时自动编译

打开 Action 窗口:

    Linux : CTRL+SHIFT+A
    Mac OSX : SHIFT+COMMAND+A
    Windows : CTRL+ALT+SHIFT+/

 输入"Registry..."，启用`compiler.automake.allow.when.app.running`

#### 附录

##### 激活 JRebel

1. 打开 `https://my.jrebel.com/`，使用 twitter/facebook 账号，填写相关的信息。注意：邮箱请一定不要乱填。就可以收到官方发来的激活码。

2. {% asset_img 15152289327942.jpg %}

输入邮箱收到的相关数据就 OK。


##### References: 

1. https://www.mkyong.com/spring-boot/intellij-idea-spring-boot-template-reload-is-not-working/

2. http://www.jhipster.tech/configuring-ide-idea/

2. https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html

3. https://www.jetbrains.com/help/idea/navigating-to-action.html

3. http://guides.rubyonrails.org/autoloading_and_reloading_constants.html

