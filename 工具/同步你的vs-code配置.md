---
title: 同步你的vs-code配置
date: 2019/5/2 21:11:00
categories: 
- 开发环境
tags: 
- IDE
- vscode
- 开发环境
---

在不同的设备上使用 [vs code](https://code.visualstudio.com/)，配置可能会不一样，比如eslint工具，各种插件的配置。使用 [vs code](https://marketplace.visualstudio.com/items?itemName=Shan.code-settings-sync) 的 Settings Sync 插件就可以简单地同步配置。

这个插件的原理其实就是利用 [github gist](https://gist.github.com/)（代码片段功能）来保存和同步配置信息。

1. 在 vsc 中找到“插件(Extensions)”，搜索 Settings Sync （输入settings即可），找到插件点击安装。
2. 使用快捷键：shift+alt+u (macbook是option键) 上传配置，首次使用将会看到一个界面，选择 "login with github"，登录 github 账号后，点击下方按钮来自动创建 gist。
3. 在另一台电脑上，安装 Settings Sync 插件，同样登录到 github，这时会让你选择已有的 gist，从给出的列表中选出刚才创建好的 gist，即可完成绑定。
4. 使用快捷键：shift+alt+d 来下载配置。
