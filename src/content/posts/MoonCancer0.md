---
title: MoonCancer BB亲小助手
published: 2025-08-17
description: 'MoonCancer基础说明，快速学会如何使用MoonCaner'
image: 'https://tsundora.com/image/2018/02/fate_stay_night_2816.jpg'
tags: [fgo, 脚本, MoonCancer]
category: MoonCancer
draft: false 
---
<font size=5>**配置不够？不能满羁绊？刷本太慢？<br>真拿你没办法，前辈 ❤，让 BB 亲来帮你吧**</font>

## 功能简介

- `日常任务`：打开模拟器，打开游戏，抽每日友情池，打白纸化或戴冠战结束，也可以在主页开始
- `开始战斗`：无烦恼刷各种副本，复杂的机制关可能不能正常通关，建议手动修改根据机制来完成战斗
- `过主线`：自动过主线，可以选择朗读剧情，或者跳过剧情，部分情况可以要手动调整重新启动任务
- `清理free本`：自动清理free本(We Are Free!)
- `Cancer`：局内数值修改器，只能在战斗中使用，用来手动修改数值
- `Alter`：提供更灵活的方式修改局内数值，主要用来打圣杯战线
- `Eisyou`：随机分配角色朗读剧情，可以挂着当相声听
- `Kidou`：打开模拟，再打开游戏进入到主界面

**只支持国服和日服安卓模拟器使用（能在模拟器上登录即可用，包括渠道服、IOS也可也在模拟器上登录）**

**有些问题懂的都懂不要到处传播，打枪滴不要，也不要上纲上线，我只是为了轻松的玩 FGO，怕别用，用别怕。**

---
:::warning
[&lt;&lt;哔哩哔哩游戏平台用户协议&gt;&gt;](https://yhxy.biligame.com/)第 11 条第 2 款规定
用户不得通过不正当的手段或其他不公平的手段使用本平台游戏、本平台服务或参与本平台活动。
用户不得干扰本平台正常地提供游戏和服务，包括但不限于：攻击、侵入本平台的网站服务器，或集中时间段内以超出正常用户登录需求的高频率登录或尝试登录服务器从而使网站服务器过载；破解、修改本平台游戏程序；攻击、侵入本平台游戏的服务器/游戏服务器端程序或使游戏服务器过载；制作、发布、传播、使用任何形式的妨碍游戏公平性的辅助工具或程序（包括但不限于“外挂”, “外挂”是指独立于游戏软件之外的，能够在游戏运行的同时影响游戏操作的所有程序，包括但不限于模拟用户操作、改变操作环境、修改数据等一切类型）。用户不得：利用程序的漏洞和错误(Bug)破坏游戏的正常进行或传播该漏洞或错误(Bug)；不合理地干扰或阻碍他人使用本平台游戏和本平台服务；通过异常或者非法的方式使用本平台游戏(包括但不限于利用本平台游戏登录游戏私服)；使用异常的方法登录游戏（包括但不限于使用非本平台开发、授权或认可的第三方软件、系统登录游戏）、使用网络加速器等外挂软件或机器人程序等恶意破坏服务设施、扰乱正常服务秩序的行为；修改、翻译、注释、整理、汇编、演绎本平台游戏；利用本平台游戏或者本平台服务可能存在的技术缺陷或漏洞而以各种形式为自己及他人牟利（包括但不限于复制游戏虚拟物品等）或者从事其他不正当行为等。
:::

## 界面展示
![main.png](../../assets/images/mooncaner/main.png)
![maindark.png](../../assets/images/mooncaner/maindark.png)
## 界面说明
右上角的帮助图标可以打开`帮助`弹窗，有问题随时可以查看，如果觉得有帮助可以赞助我一点。<br>
在启动任意任务后会出现`暂停`按钮，可以暂停执行中的任务。点击下面链接可查看每个界面具体说明：<br>
[Moon界面说明](../mooncancer1)<br>
[Cancer界面说明](../mooncancer2)<br>
[Alter界面说明](../mooncancer3)<br>
[Eisyou界面说明](../mooncancer4)<br>
## 下载安装
群文件下载`Updater.exe`，新建一个文件夹，将文件放到里面，例如`MoonCaner/Updater.exe`，然后运行`Updater.exe`，就会自动下载安装，同时也是更新程序
## 更新方式
- 自动检测更新，在设置中开启自动检测更新
- 手动检测更新，在设置->关于->检查更新
![update](../../assets/images/mooncaner/update.png)
- 运行程序根目录的`Updater.exe`
## 模拟器设置
- 使用其他模拟器理论上可行，但推荐MuMu或雷电，这两个适配了高速连接，本身玩fgo也比较稳定，MuMu现在官网已经只能下载到新版了，想用旧版这里有历史版本[MuMu模拟器历史版本](https://www.cnblogs.com/wutou/p/18165628)
- 模拟器用16：9的分辨率，1280x720起步，分辨率过低可能无法识别，最重要的mumu模拟器关闭后台保活，其他模拟器可能也有一些导致无法正常截图的设置
![mooncanermumusetting](../../assets/images/mooncaner/mumusetting.png)

## 脚本设置
:::important
请在设置之前打开要使用的模拟器，多开的情况要保证目标模拟器先启动，不然修改数值会失败
:::
![settings1](../../assets/images/mooncaner/settings1.png)
- `配置`支持添加移除配置，右键可以重命名配置，支持多配置切换
- `模拟器设备`列表里有的都支持自动搜索，没有就选other，这里选MuMu和雷电的话默认使用高速连接，切换到other时只要参数正确也能正常使用adb连接；如果高速连接出现问题可以切换到other使用默认连接方式，`如果是新版MuMu只能选mumu才能正常使用`。<br>
可以自动搜索已经在运行的模拟器，自动搜索设备并设置好所有参数理论支持MuMu、雷电、蓝叠、逍遥、夜神，但只测试了MuMu和雷电，搜不到就自己输入对应的参数，端口上网搜XX模拟器端口，连接问题可以参考[BBChannel](https://www.bilibili.com/opus/605452889655035035?spm_id_from=333.999.0.0)的相关说明。**注意，如果自动搜索成功就不要动下面的模拟器位置和虚拟机进程名称**
- `服务器和包名`可以选择国服和日服，更改后会自动触发包名设置。包名默认是B服，其他客户端要修改，设置好连接参数后，打开模拟器后能自动搜索，搜索失败就自己填了
:::warning
自动搜索并成功设置的情况下不要动`模拟器位置`和`虚拟机进程名称`这两项，除非你知道自己在干什么
:::
- `模拟器位置`是用来启动模拟器的命令，一般选择模拟主器程序的位置就行了，也可也自己加参数，自动搜索的是带参数的
- `虚拟机进程名称`是读取内存的目标，是修改的基础，**没填或没填对脚本没法正常工作**，默认就是mumu模拟器的
- `连接等待时间`如果自动启动模拟器时，模拟器启动时间过长导致没连上，可以增加等待时间
- 剩下的设置没啥好讲的了，自己看着选
:::tip
这里简单说一下怎么找到自己模拟器的虚拟机名称，打开模拟器，再打开任务管理，在进程里找到占用最高的模拟器相关进程(这里以mumu为例)，右键打开文件所在的位置，就可以直到程序，复制名称就行了，要包括.exe。
或者直接问AI一般也能得到一个比较准确的答案，可以参考着找一下，就问 **XX模拟器的虚拟机进程名称是什么**
![taskmanager](../../assets/images/mooncaner/taskmanager.png)
:::

## 游戏内设置
- 记得打开**筛选**（我该不会是最后一个知道的吧），**加快敌人消失速度**，**战斗二倍速**。筛选可以减少找到目标礼装的时间，不过现在羁绊礼装太多，午茶反而排到羁绊礼装的最后面去了
![filter](../../assets/images/mooncaner/filter.png)
## 其他说明
- 要自动白纸化每日，请尽量保持界面干净，保证要打的关卡已经开启，部分关卡依赖奏章Ⅱ定位（例如90⭐），所以至少要打完奏章Ⅰ，才能使用所有选项
![ordercall](../../assets/images/mooncaner/ordercall.png)

- 强化任务只要设置足够的次数理论上可以一次性打完，但有些有多个强化本的，开放时会有弹窗，可能会导致卡住，这个情况不好触发，暂时没适配，需要手动处理一下
![enhancequest](../../assets/images/mooncaner/enhancequest.png)
## 其他问题
- 如果开始后连接出现如下错误，而且执行的非常慢，请先卸载模拟器内的`yosemite`应用，然后重试

```log
[17:16:03][ERROR]<airtest.core.android.cap_methods.screen_proxy> OSError('socket connection broken')
[17:16:03][ERROR]<airtest.core.android.cap_methods.screen_proxy> Javacap setup up failed!
```

- 如果同时使用BBChannel，反复出现上诉问题，将`MoonCancer\assets\static`文件夹复制到BBchannel里面的`dist\BBchannel64\airtest\core\android\static`覆盖，这个方式同样能解决BBChannel的连接问题
- 遇到错误请在群里反馈，尽量在出现问题的地方截屏或录屏，最好用模拟器自带的方式截屏和录屏

## 碎碎念
有这个想法已经几年了终于是实现了，第一次写这么正式的项目，只能说有编程基础，摸着石头过河，写的很烂，但由于某些原因，不能公开分享。
开始这个项目快一年了，也更新到了2.0全新版本，还搭建博客，过的挺充实的

---
## 相关项目
使用和参考相关项目：
- [Blue Archive Auto Script碧蓝航线脚本](https://github.com/pur1fying/blue_archive_auto_script)
- [FGO-py脚本](https://github.com/hgjazhgj/FGO-py)
- [Airtest自动化测试库](https://github.com/AirtestProject/Airtest)
- [RapidOCR文字识别](https://github.com/RapidAI/RapidOCR)
- [PyQt-Fluent-Widgets图形界面组件库](https://github.com/zhiyiYo/PyQt-Fluent-Widgets)
- [Pymem内存读取修改](https://github.com/srounet/Pymem)

