---
title: 在文件夹右键菜单添加cmd管理员、终端(powershell)管理员
published: 2025-08-19
description: '在文件夹右键菜单添加cmd管理员、终端(powershell)管理员'
image: ''
tags: [cmd, powershell, windows]
category: 'Share'
draft: false 
lang: ''
---

## 右键菜单创建用管理员打开cmd

将代码复制到txt，并修改后缀为.reg，然后双击执行
```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\shell\AdminOpenCmd]
"Icon"="cmd.exe"
@="管理员打开命令提示符"

[HKEY_CLASSES_ROOT\Directory\shell\AdminOpenCmd\command]
@="PowerShell -windowstyle hidden -Command \"Start-Process cmd.exe -ArgumentList '/s,/k, pushd,%V' -Verb RunAs\""

[HKEY_CLASSES_ROOT\Directory\Background\shell\AdminOpenCmd]
"Icon"="cmd.exe"
@="管理员打开命令提示符"

[HKEY_CLASSES_ROOT\Directory\Background\shell\AdminOpenCmd\command]
@="PowerShell -windowstyle hidden -Command \"Start-Process cmd.exe -ArgumentList '/s,/k, pushd,%V' -Verb RunAs\""
```
## 右键菜单创建打开终端管理员

将代码复制到txt，并修改后缀为.reg，然后双击执行，想要图标正常显示就修改icon路径为正确的地址
```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\Directory\shell\PowerShellAdmin]
@="终端管理员"
"Icon"="C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.22.12111.0_x64__8wekyb3d8bbwe\WindowsTerminal.exe"

[HKEY_CLASSES_ROOT\Directory\shell\PowerShellAdmin\command]
@="powershell.exe -Command \"Start-Process wt.exe -ArgumentList '--window 0 --startingDirectory \\\"%V\\\"' -Verb RunAs\""

[HKEY_CLASSES_ROOT\Directory\Background\shell\PowerShellAdmin]
@="终端管理员"
"Icon"="C:\Program Files\WindowsApps\Microsoft.WindowsTerminal_1.22.12111.0_x64__8wekyb3d8bbwe\WindowsTerminal.exe"

[HKEY_CLASSES_ROOT\Directory\Background\shell\PowerShellAdmin\command]
@="powershell.exe -Command \"Start-Process wt.exe -ArgumentList '--window 0 --startingDirectory \\\"%V\\\"' -Verb RunAs\""
```