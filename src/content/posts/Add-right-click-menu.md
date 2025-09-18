---
title: 在文件夹右键菜单添加cmd管理员、终端(powershell)管理员
published: 2025-08-19
description: '在文件夹右键菜单添加cmd管理员、终端(powershell)管理员'
image: ''
tags: [cmd, powershell, windows]
category: 'Share'
draft: true 
lang: ''
---



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