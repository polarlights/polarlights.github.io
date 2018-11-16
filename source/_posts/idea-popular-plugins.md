---
title: idea popular plugins
date: 2018/09/19 23:20:49
categories:
  - editor
tags:
  - idea
  - plugins
---

## 基本

1. [lombok plugin](https://plugins.jetbrains.com/plugin/6317-lombok-plugin)

2. [GenerateSerialVersionUID](https://plugins.jetbrains.com/plugin/185-generateserialversionuid)

    Adds a new action 'SerialVersionUID' in the generate menu (alt + ins). The action adds an serialVersionUID field in the current class or updates it if it already exists, and assigns it the same value the standard 'serialver' JDK tool would return. The action is only visible when IDEA is not rebuilding its indexes, the class is serializable and either no serialVersionUID field exists or its value is different from the one the 'serialver' tool would return.

## 效率篇

1. [Jrebel for IDEA](https://plugins.jetbrains.com/plugin/4441-jrebel-for-intellij)

    JRebel is a productivity tool that allows developers to reload code changes instantly.

1. [save actions](https://plugins.jetbrains.com/plugin/7642-save-actions)

    Supports configurable, Eclipse like, save actions, including "optimize imports", "reformat code", "rearrange code", "compile file" and some quick fixes for Java like "add / remove 'this' qualifier", etc. The plugin executes the configured actions when the file is synchronised (or saved) on disk.
    
1. [ace-jump](https://plugins.jetbrains.com/plugin/7086-acejump)

    AceJump allows you to quickly navigate the caret to any position visible in the editor.
    
    {% asset_img 15373699756710.jpg %}

    
## 代码质量

1. [checkstyle](https://plugins.jetbrains.com/plugin/1065-checkstyle-idea)

1. [Find bugs](https://plugins.jetbrains.com/plugin/3847-findbugs-idea)

    The FindBugs plugin for IntelliJ IDEA.
    
1. [SonarLint](https://plugins.jetbrains.com/plugin/7973-sonarlint)

    SonarLint is an IDE extension that helps you detect and fix quality issues as you write code.
    
2. [alibaba-coding-guideline](https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines)

    Alibaba Java Coding Guidelines plugin support.

    {% asset_img alibaba alibaba.gif %}


## 语法高亮

1. [ignore](https://plugins.jetbrains.com/plugin/7495--ignore)

    **.ignore** is a plugin for .gitignore (Git), .hgignore (Mercurial), .npmignore (NPM), .dockerignore (Docker), .chefignore (Chef), .cvsignore (CVS), .bzrignore (Bazaar), .boringignore (Darcs), .mtn-ignore (Monotone), ignore-glob (Fossil), .jshintignore (JSHint), .tfignore (Team Foundation), .p4ignore (Perforce), .prettierignore (Prettier), .flooignore (Floobits), .eslintignore (ESLint), .cfignore (Cloud Foundry), .jpmignore (Jetpack), .stylelintignore (StyleLint), .stylintignore (Stylint), .swagger-codegen-ignore (Swagger Codegen), .helmignore (Kubernetes Helm), .upignore (Up), .prettierignore (Prettier), .ebignore (ElasticBeanstalk) files in your project. 
    
1. [markdown-support](https://plugins.jetbrains.com/plugin/7793-markdown-support)

    Provides the capability to edit markdown files within the IDE and see the rendered HTML in a live preview. 
    
1. [asciidoc](https://plugins.jetbrains.com/plugin/7391-asciidoc)

    AsciiDoc language support for IntelliJ platform.

## 其它

3. [ideavim](https://plugins.jetbrains.com/plugin/164-ideavim)

    IdeaVim supports many Vim features including normal/insert/visual modes, motion keys, deletion/changing, marks, registers, some Ex commands, Vim regexps, configuration via ~/.ideavimrc, macros, window commands, etc.
    
1. [grep-console](https://plugins.jetbrains.com/plugin/7125-grep-console)

    1. Change colors of matching text.
    2. Grep output into a new console tab.
    3. Change output or execute any action using custom groovy scripts or plugins.
    4. Filter out unwanted lines.
    5. Fold output.
    6. Play sounds on matches.
    7. Clear Console on matches.
    8. Tail files*.

    {% asset_img 15373698266180.jpg %}
{% asset_img 15373698363530.jpg %}

1. [key-promoter-x](https://plugins.jetbrains.com/plugin/9792-key-promoter-x)

    Shows the user the keyboard short-cuts when a button is pressed with the mouse. This provides an easy way to learn how to replace tedious mouse work with keyboard keys and helps to transition to a faster, mouse free development. Currently, it supports toolbar buttons, menu buttons, and tool windows and the actions therein. 
