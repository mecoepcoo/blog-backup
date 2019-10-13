---
title: 让mocha支持ES6模块
date: 2019/10/5 21:08:00
updated: 2019/10/5 21:08:00
categories: 
- 开发环境
tags: 
- 工具
- 开发环境
---

Win 系统的控制台默认编码不是utf-8，使用vscode，git bash等工具时经常在控制台看到乱码。

调整这些配置，就能让控制台正确显示中文。

1. 使用 git bash 等工具时，先打开工具的设置，找找设置里有没有文本编码（character set）相关的选项，设置为"UTF-8"即可
2. 快捷键 `win+R` 打开运行，输入 `regedit` 回车，打开注册表编辑器，找到`HKEY_CURRENT_USER\Console\`在里面的每一项中，新建名为`CodePage`（如果已有则修改）的DWORD值，数值设为十进制65001即可
