
---
title:        Mac mail app tricks
date: 2018/11/09 10:05:12
categories:
  - Mac
tags:
  - Mac
  - Mail
  - tricks
---

## 常用设置

1.修改邮件的一些默认设置
```bash
# Disable send and reply animations in Mail.app
defaults write com.apple.mail DisableReplyAnimations -bool true
defaults write com.apple.mail DisableSendAnimations -bool true

# Copy email addresses as `foo@example.com` instead of `Foo Bar <foo@example.com>` in Mail.app
defaults write com.apple.mail AddressesIncludeNameOnPasteboard -bool false

# Add the keyboard shortcut ⌘ + Enter to send an email in Mail.app
defaults write com.apple.mail NSUserKeyEquivalents -dict-add "Send" "@\U21a9"

# Display emails in threaded mode, sorted by date (oldest at the top)
defaults write com.apple.mail DraftsViewerAttributes -dict-add "DisplayInThreadedMode" -string "yes"
defaults write com.apple.mail DraftsViewerAttributes -dict-add "SortedDescending" -string "yes"
defaults write com.apple.mail DraftsViewerAttributes -dict-add "SortOrder" -string "received-date"

# Disable inline attachments (just show the icons)
defaults write com.apple.mail DisableInlineAttachmentViewing -bool true

# Disable automatic spell checking
defaults write com.apple.mail SpellCheckingBehavior -string "NoSpellCheckingEnabled"
```
## 邮件分类放在不同目录下

邮件全部放在收件箱里面，不做分类的话，随着邮件越积越多，邮件非常杂乱。我们可以在邮件中添加目录(mail.app 中叫 mailbox)，然后在“Preferences" -> "Rules"，添加新的邮件规则。

过滤规则中，还可以设置发送通知，以便于我们不错过重要的邮件。

## Smart Mailbox

邮件放在不同目录下会比较有序，但是它只是比较粗的分类，有时我们需要更加定制化的从另外一个层面去透视邮件。比如筛选某些特定发件人、邮件主题、旗标、某个发件日期等邮件。

在邮箱左侧“Smart Mailboxes"，选择添加，

{% asset_img 15417270316426.jpg %}

## VIP

对于某些 Very Import Person 发送的邮件，可以很方便地将其加入到 VIP 列表：在发件人邮箱上点击右键，选择添加到 VIP 即可。

## 旗标

默认旗标只有一些颜色名称，而没有其它语义。给颜色添加一种语义，以方便我们理解。

先给邮件添加某个旗标，然后在左侧可以看到对应旗标出现；右键选择重命名(或者单击某个旗标，再次点击)，即可对其重命名。如果没有对应旗标的邮件，改旗标不会出现在左侧，也无法对其更名。

{% asset_img 15417288936047.jpg %}



