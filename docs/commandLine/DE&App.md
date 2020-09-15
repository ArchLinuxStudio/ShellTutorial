# 桌面环境与常用应用

官方文档: [安装后的工作](<https://wiki.archlinux.org/index.php/General_recommendations_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>)  
本文只介绍最基本的，能使系统真正意义上可用所需的组件  
相关视频链接： 2020ArchLinux 安装桌面环境和常用软件<sup>TODO</sup> 视频文字结合效果更好  
注: 文档中带有 <sup>AUR</sup> 角标的软件代表是用户自行打包的第三方软件[AUR](https://aur.archlinux.org/)，不在 Arch 官方支持范围内，可能会出现各种问题。如果不是实在没有官方支持的同类软件，则不建议使用。

#### 1.确保系统为最新

```bash
pacman -Syyu    #升级系统中全部包
```

#### 2.准备非 root 用户

添加用户，比如新增加的用户叫 wallen

```bash
useradd -m -g users -G wheel -s /bin/bash wallen  #wheel附加组可sudo进行提权
```

设置新用户 wallen 的密码

```bash
passwd wallen
```

编辑 sudo 文件

```bash
visudo
```

找到这样的一行,把前面的注释符号#去掉，`:wq`保存并退出即可。

```bash
#%wheel ALL=(ALL) ALL
```

这里稍微解释一下
%wheel 代表是 wheel 组，百分号是前缀  
ALL= 代表在所有主机上都生效(如果把同样的`sudoers`文件下发到了多个主机上)  
(ALL) 代表可以成为任意目标用户  
ALL 代表可以执行任意命令  
一个更详细的例子:

```bash
%mailadmin   snow,rain=(root) /usr/sbin/postfix, /usr/sbin/postsuper, /usr/bin/doveadm
nobody       ALL=(root) NOPASSWD: /usr/sbin/rndc reload
```

组 mailadmin 可以作为 root 用户，执行一些邮件服务器控制命令。可以在 "snow" 和 "rain"这两台主机上执行  
用户 nobody 可以以 root 用户执行`rndc reload`命令。可以在所有主机上执行。同时可以不输入密码。(正常来说 sudo 都是要求输入调用方的密码的)

#### 3.安装 KDE Plasma 桌面环境

```bash
pacman -S plasma-meta #安装plasma-meta元软件包
```

#### 4.安装 greeter sddm

```
pacman -S sddm
```

重启电脑，即可看到欢迎界面，输入新用户的密码即可登录桌面

#### 5.开启 32 位支持库与 ArchLinuxCN 支持库

进入桌面后，搜索 terminal，可以找到 konsole。它是 KDE 桌面环境默认的命令行终端。

```bash
sudo vim /etc/pacman.conf
```

去掉[multilib]一节中两行的注释，来开启 32 位库支持。  
在文档结尾处加入下面的文字，来开启 ArchLinuxCN 源。

```bash
[archlinuxcn]
SigLevel = Optional TrustAll
Server = http://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

上面服务器的地址是清华的，也可用下面中科大的

```bash
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```

最后:wq 保存退出，刷新 pacman 数据库

```bash
sudo pacman -Syyu
```

#### 6.安装基础功能包

接下来我们安装一些基础功能包

```bash
sudo pacman -S ntfs-3g                                                      #识别NTFS格式的硬盘
sudo pacman -S bash-completion                                              #命令行补全工具
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei                   #安装几个开源中文字体
sudo pacman -S ttf-liberation                                               #安装Red Hats字体
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra  #安装谷歌开源字体
sudo pacman -S firefox chromium                                             #安装常用的火狐、谷歌浏览器
sudo pacman -S yay                                                          #yay命令可以让用户安装AUR中的软件 格式：yay -S xxx
```

#### 7.设置系统为中文

如果想要系统换为中文，需要重新设置 locale

编辑 /etc/locale.gen，去掉 zh_CN.UTF-8 的注释符号（#）。

```bash
locale-gen  #重新生成locale
```

编辑 /etc/locale.conf

```bash
echo 'LANG=zh_CN.UTF-8'  >> /etc/locale.conf
```

#### 8.安装输入法

使用 [Fcitx5](<https://wiki.archlinux.org/index.php/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)>) 的默认输入法。
中文及日文输入法均体验良好。这部分的官方中文文档质量良好，建议尝试跟随安装。

#### 9.显卡驱动

现在是 2020 年，显卡驱动的安装在 Arch Linux 上已经变得非常容易。

- 英特尔核心显卡 [官网文档](https://wiki.archlinux.org/index.php/Intel_graphics)

  ```bash
  sudo pacman -S xf86-video-intel mesa lib32-mesa vulkan-intel
  ```

- Nvidia 显卡

  - 若为台式机，拥有独立的显卡，直接安装如下两个包即可。[台式机显卡官方文档](https://wiki.archlinux.org/index.php/NVIDIA)

  ```bash
  sudo pacman -S nvidia nvidia-settings nvidia-utils lib32-nvidia-utils
  ```

  如果是 GeForce 630 以下到 GeForce 400 系列的老卡，尝试安装 [nvidia-390xx-dkms](https://aur.archlinux.org/packages/nvidia-390xx-dkms/)<sup>AUR</sup>

  再老的显卡直接使用[开源驱动](https://wiki.archlinux.org/index.php/Nouveau)即可。

  如有需要，安装 32 位支持的相关包。

  - 若为笔记本，除上述的包，推荐安装 optimus-manager。可以在核心显卡和独立显卡间轻松切换。[笔记本双显卡官方文档](https://wiki.archlinux.org/index.php/NVIDIA_Optimus)

  ```
  sudo yay -S optimus-manager
  ```

- AMD 显卡  
  群主目前没有 AMD 显卡或者笔记本，无法提供经验证的最佳实践

  <!-- ```bash
  sudo pacman -S xf86-video-amdgpu    #amd显卡
  ``` -->

- OpenCL  
  如果你需要 OpenCL 的安装，请参照[官方文档](https://wiki.archlinux.org/index.php/GPGPU)针对自己的显卡进行配置。

#### 后续

如果作为一个普通使用者，到这里你的系统已经配置完毕了。不会命令行也没太大关系，你可以慢慢探索 KDE 这个桌面环境，记住每天用如下命令更新系统即可。

```bash
sudo pacman -Syyu
```

接下来你可以查阅娱乐、办公、多媒体等章节了解更多使用软件的安装与使用。如果你需要成为一名较为专业的人员，那么请阅读进阶、高级、以及编程章节。
