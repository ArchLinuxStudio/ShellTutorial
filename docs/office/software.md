# 办公软件

本章记录日常办公需要用到的软件及配置。同时包括 QQ 等即时通讯软件的配置与使用。

#### 办公套件

主要两个选择是 [WPS](<https://wiki.archlinux.org/index.php/WPS_Office_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>) 与 [LibreOffice](https://wiki.archlinux.org/index.php/LibreOffice)。LibreOffice 目前的安装已经非常简单

```bash
sudo pacman -S libreoffice-still   #稳定版
sudo pacman -S libreoffice-fresh   #尝鲜版
```

WPS 请按官方文档安装

#### 截图

flameshot
[火焰截图](https://www.bilibili.com/video/BV1LK4y1s7wX/)

```
sudo pacman -S flameshot
```

#### 即时通讯

wine 版本应用均不建议安装。QQ 刚需可以尝试 [scrcpy](https://aur.archlinux.org/packages/scrcpy/)
安装 wine 版本前先确保[字体](https://wiki.archlinux.org/index.php/Microsoft_fonts)的安装，否则微软汉字均为方块。

```bash
sudo pacman -S telegram-desktop #全球流行的即时通信软件 俗称电报
yay -S linuxqq                  #腾讯官方出版的辣鸡linuxqq
yay -S slack-desktop            #优秀的团队合作交流软件
yay -S deepin-wine-qq           #基于deepin wine的qq
yay -S deepin-wine-tim          #基于deepin wine的tim
yay -S deepin-wine-wechat       #基于deepin-wine-wechat
```

#### 网盘存储

- [坚果云](https://aur.archlinux.org/packages/nutstore/)<sup>AUR</sup> 可直接使用 [web 版本](https://www.jianguoyun.com/d/home#/)
- [Mega](https://aur.archlinux.org/packages/megasync/)<sup>AUR</sup> 可直接使用 [web 版本](https://mega.nz/fm/dashboard)
- [超星网盘](http://i.mooc.chaoxing.com/space/index?t=1600061701200) 据说免费 100G 未验证
- [百度网盘](https://aur.archlinux.org/packages/baidunetdisk-bin/) 辣鸡

#### 图片浏览

快速看图

- [feh](https://www.archlinux.org/packages/extra/x86_64/feh/)
- [nomacs](https://www.archlinux.org/packages/community/x86_64/nomacs/)
