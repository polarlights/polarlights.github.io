---
title: idea productive tips
date: 2018/09/19 23:20:49
categories:
  - editor
tags:
  - idea
  - editor
---

This article is wroten according to a youtube video: https://www.youtube.com/watch?v=eq3KiAH4IBI.

We use Idea programming everyday. How can we use it effectively, now let's reading it below:

1. Close the tab list of IDE

    Preferences → Editor → Editor Tabs; Change Tab Placement from default **Top** to **None**.
    
    Tab is annoying when many files are opened. It's hard to find file when opening to many files, which it's inevitable. Use recent files window or search everywhere (Double Shift). We can use **⌘+E** to explore recent files instead.

1. Auto scroll to source

    When we select a file, we want to explore the file content, other than just the file name. If we want to see the source, we have to double click the file in project window, by default. While Idea provides us a feature that when we select a file, Idea will open the source automatically:
    In the project window, right top of the window, click the config icon, check the "scroll to source" in the dropdown list.

1. Create directory or file effectivelly

   If I want to create a nested directory or package, I will select the parent directory and press `⌘+N`, then input the directory name, before. There's more effective way: Just input the relative directory path, Idea will create the sub directory for us, if it doesn't exist.
   
   We can create file or directory through the navigation bar.

1. Search file or class by search

    Double shift(press shift double times) is our friend, it pops up a search window which we can search file, class. It will list the more recent used files at the file lists. 

1. Scratch file feature

    It is a very common scenario that we want test a piece of code outside of the project context. We can press **⌘+⇧+N** to create a scratch file which will be removed after we finish test. See more details, please visit https://www.jetbrains.com/help/idea/scratches.html

1. Shrink Selection:

    Increase selection: **⌥ + ↑**
    Shrink selection: **⌥ + ↓**

1. Move code block

    move code block up: **⌥ + ⇧ + ↑**
    move code block down: **⌥ + ⇧ + ↓**
    
1. Rename (Refectory)

    Shift + F6

3. Auto comple

    Please follow both steps:
    
    1. - Enable Automake from the compiler
    Press: ctrl + shift + A (For Mac ⌘ + shift + A)
    Type: make project automatically
    Hit: Enter
    Enable Make Project automatically feature
    2. - Enable Automake when the application is running
    Press: ctrl + shift + A (For Mac ⌘ + shift + A)
    Type: Registry
    Find the key compiler.automake.allow.when.app.running and enable it or click the checkbox next to it

1. Auto wrap code

    * When editing code:

    File → Editor → Code Style → Java → Wrappings And Brackets
    
    Change: Wrap On Typing → true
    
    * When viewing code:
    
     https://stackoverflow.com/questions/23004520/code-wrap-intellij 

1. Insert Language Template

    https://www.jetbrains.com/help/idea/using-language-injections.html

1. Navigation Bar
    
    Command + ↑
