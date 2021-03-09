---
title: Manjaro 更新后丢失 grub 菜单
data: 2021-03-09 15:30
---

今天更新了 Manjaro 后发现，开机时直接进入了 Manjaro 而没有显示用来启动 Windows 和 Manjaro 双系统的 grub 菜单。遍查了网上关于 grub 更新后丢失启动项的帖子后，最后居然在 [Manjaro Linux Forum](https://forum.manjaro.org/) 找到了问题所在（多看更新日志和官方社区！）。

原帖链接：[After update, grub doesn’t find windows install](https://forum.manjaro.org/t/after-update-grub-doesnt-find-windows-install/56899)

根本原因在昨天 Manjaro Linux 的[更新日志](https://forum.manjaro.org/t/stable-update-2021-03-08-kernels-plasma-5-21-2-haskell-kodi-grub-kde-dev/56877/14)中就能找到:

> -   **Grub** got some needed [security updates 37](https://wiki.ubuntu.com/SecurityTeam/KnowledgeBase/GRUB2SecureBootBypass2021). Note that **os-prober** is now disabled by default for security reasons: [broken patch 8](https://git.savannah.gnu.org/cgit/grub.git/commit/?id=e346414725a70e5c74ee87ca14e580c66f517666); [fixed patch 15](https://lists.gnu.org/archive/html/grub-devel/2021-03/msg00193.html). More infos about it [here 44](https://forum.manjaro.org/t/grub-disable-os-prober-flag-appears-to-be-ignored-in-etc-default-grub/56382).

Grub 在更新因为一些安全原因默认禁用了 [os-prober](https://github.com/mator/os-prober)，原因在上面引用中的 fixed patch 中也有详谈。

os-prober 可以为 grub 检测硬盘中安装的其他操作系统，并在 grub 中添加引导。在 Arch Linux Wiki 中关于 [Grub-Detecting other operating systems](https://wiki.archlinux.org/index.php/GRUB#Detecting_other_operating_systems)有提到 os-prober 的使用：

```shell
sudo os-prober
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo update-grub
```
即可检测到其他操作系统并将其添加到 grub。

回到 Manjaro 的更新，为了重新在 grub 中启用 os-prober，需要编辑 `/etc/default/grub.cfg` 在其中添加：

```
GRUB_DISABLE_OS_PROBER=false
```

然后执行 `sudo update-grub` 即可将 Windows 的 Grub menu entry 添加回去。