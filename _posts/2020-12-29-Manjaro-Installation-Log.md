---
title: Manjaro Installation Log
date: 2020-12-29 12:19
---

# Manjaro Installation Log

Ubuntu 在上周更新系统的时候崩掉了，索性直接把硬盘里的 Ubuntu 清掉了。之前在系统里面盲目配置了一堆乱七八糟的东西导致自己有些难以掌控。

我想要一个**足够简洁**、配置**相对简单**、**管理软件方便**的一个操作系统，之前没试过 Arch 系的系统，但是 Arch 的安装实在魔鬼，那就 [Manjaro](manjaro.org) 好了。

现在已经安装好了 Manjaro，这篇 Blog 就是在新系统下使用 Vim 写下的，记录一下安装和配置的过程。

## 安装和配置过程的原则

- 力求最简，绝不配置乱七八糟的美化插件；
- 搞清楚每一个步骤，时刻提醒自己到底在做什么。

## Why Manjro?

- 易于安装和配置（相比于 Arch Linux）；
- pacman
- ...

选择安装的Manjaro 版本是 Manjaro 20.2 Nibia | XFCE - minimal

### Why XFCE？

> 1. XFCE embodies the **tradition UNIX philosophy** of modularity and reusability.
> 2. Supported by the manjaro team.
> 3. No advanced customizations.

除了 XFCE 桌面环境外，Manjaro 官方也支持 KDE 和 Gnome。之所以不选择更加流行的 KDE 或者 Gnome，是因为之前在配置 Gnome Ubuntu 时在配置主题、各种插件小组件这些乱七八糟的事情上面花费了太多时间，反而丢了来使用 Linux 的初衷。所以 XFCE 的 *No advanced customizations* 对我而言反而是一个优势。

或许以后会尝试 [i3wm](https://i3wm.org/) ;)

## Install Manjaro

因为还需要打游戏但是又没有第二台电脑（QAQ，所以还是得留着Windows，在笔记本上安装双系统。

### 在Windows下对磁盘分区

使用 Windows 下的磁盘管理工具，在待装系统的SSD上通过**压缩卷**分出一个256GB的空间来安装 Manjaro。

1. 右键开始菜单徽标->选择*磁盘管理*
2. 右键待分区的磁盘->压缩卷->256\* 1025 = 262144MB

使用 rufus 制作好启动盘之后即可重启电脑开始安装了。

插上启动盘，开机时进入 BIOS 选择从 USB 启动即可进入 Manjaro 的安装界面。需要注意的是，我的笔记本因为有 Nvidia 的独立显卡（真是毫无用处，徒增耗电），需要选择 Boot with proprietary drivers，而不能选择开源驱动。

### Manjaro 系统分区

接下来安装的过程都很简单，需要注意的就是系统分区的分配，一共需要设置 EFI、Swap、Root 和 Home 这四个分区：

- EFI
    - 需要找到Windows系统盘下的EFI分区挂在到这里，一定**不要**格式化。
    - Mountpoint -> /boot/efi
    - Flags -> boot -> OK
- SWAP
    - Size -> 20480MB
    - Filesystem -> linuxswap
    - Flags -> swap
- ROOT
    - Size -> 80\*1024 = 81920MB
    - Filesystem -> ext4
    - Mountpoint -> /
    - Flags -> root -> OK
- HOME 
    - Size -> remained
    - Filesystem -> ext4
    - Mountpoint -> /home

之后按照指示安装即可。因为国内的网络环境会在安装到93%的时候卡住，网络断掉就可以了，等到成功进入系统以后更新就行了。

安装以后先不要重启，打开 terminal，输入 `efibootmgr` 来查看一下 efi 启动项，确认 Manjaro 的序号在启动顺序里面的第一个。

Reboot...

Bingo! 顺利进入到Manjaro的开机引导界面，Windows 和 Manjaro 都能正常引导开机。

## After Installation

### 修改镜像源

生成可用的中国镜像源列表

``` shell
sudo pacman-mirrors -i -c China -m rank
```

选择速度较快的镜像源。之后刷新缓存：

``` shell
sudo pacman -Syy
```

添加 [archlinuxcn](https://www.archlinuxcn.org/) 源，在 `etc/pacman.conf` 文件末尾添加：

``` 
[archlinuxcn]

Include = /etc/pacman.d/archlinuxcn
```

`archlinuxcn`这个文件从 [archlinuxcn/mirrorlist-repo](https://github.com/archlinuxcn/mirrorlist-repo) 这个 GitHub 仓库中 clone，并把较快的镜像源放在靠前的位置。

之后安装 `archlinuxcn-keyring` 包以导入 GPG key：
``` shell
sudo pacman -Sy archlinuxcn-keyring (using '-Sy' to download database file for 'archlinuxcn'
```

### Clash

从 [GitHub](https://github.com/Dreamacro/clash) 下载适用于系统的 clash（分普通版本和 premium 版本，有些机场的配置文件需要使用 premium 版本），为了方便启动将其重命名为 clash，并赋予权限：
``` shell
sudo chmod +x clash
```
将可执行文件移到 `/usr/local/bin` 即可直接在 Terminal 中执行。

第一次启动后会默认在 `~/.config/clash/` 目录下生成 `config.yaml` 和 `Country.mmdb`。

如果机场提供订阅链接的话可以使用 wget 下载配置文件：

``` shell
wget --no-check-certificate -O ~/.config/clash/config.yaml 订阅地址
```

将 clash 配置文件的目录初始化为一个 git 仓库，就能在每次更新完配置文件后使用 `git diff` 查看更新的内容（增加\移除了哪些节点）了。

`Country.mmdb` 是代理工具用以判断IP地址是否属于中国大陆，可以使用 [Hackl0us/GeoIP2-CN](https://github.com/Hackl0us/GeoIP2-CN) 来替换，更准确、更轻量。

#### 设置代理

使用 `echo $http_proxy` 即可查看当前的终端是否设置代理。

在终端中设置代理可以直接通过`export`命令实现：
``` shell
export http_proxy="http://SERVER:PORT
```
但是这样只能对当前的窗口设置代理，但是想要浏览器也走代理需要设置系统全局代理。

没想到 Manjaro XFCE 精简到了连设置系统全局代理的 GUI 都没有，使用 Gnome 桌面环境时可以直接到设置中设置手动代理，输入 clash 运行出显示的端口设置即可。如果设置中没有提供，可以通过环境变量 `/etc/environment` 来设置全局代理。在 `/etc/environment` 中添加：

``` shell
no_proxy="127.0.0.1, localhost"
all_proxy="socks://SERVER:PORT"
http_proxy="http://SERVER:PORT"
https_proxy="http"//SERVER:PORT"
```

如果想要在当前的窗口暂时取消代理：
``` shell
unset {http,https,all}_proxy
```

#### 输入法

使用的是 RIME，居然安装后自带了微软双拼了，就完全不用折腾。

``` shell
sudo pacman -S fcitx fcitx-im fcitx-configtool fcitx-rime
```
安装后编辑配置文件 `~/.xprofile`
``` shell
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
重启即可。


