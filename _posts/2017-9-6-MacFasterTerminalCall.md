---
layout:     post
title:      Mac Faster Terminal Call-out
subtitle:   Design Shortcuts for Mac
date:       2017-09-06
author:     Lyle
header-img: img/post-bg-mac.jpg
catalog: true
tags:
    - Mac
    - Efficient
    - DevTech
---

>在Mac下快速调出终端的方法：为终端添加一个快捷键打开方式

## 为终端添加一个快捷键打开方式

打开Mac下自带的软件 **Automator** -> New Document -> Quick Action -> 找到**Run AppleScript**

Workflow receives 选择 **no input**

修改框内的脚本

```
on run {input, parameters}
	tell application "Terminal"
		reopen
		activate
	end tell
end run

```

运行：`command + R`，如果没有问题，则会打开终端

保存：`Command + S`，将其命名为`OpenTerminal`或你想要的名字

设置快捷键

在 **System Preferences** -> **Keyboard** -> **Shortcuts** -> **Services**

选择我们创建好的`OpenTerminal`，设置你想要的快捷键，比如我设置了`⌥+⌘+T`

到此，设置完成。

聪明的你也许会发现，这个技巧能为所有的程序设置快捷启动。

将脚本中的 `Terminal` 替换成其他程序就可以

```
on run {input, parameters}
    tell application "App Name"
        reopen
        activate
    end tell
end run

```
