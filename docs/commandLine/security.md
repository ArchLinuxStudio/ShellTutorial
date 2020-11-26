# 理解 Linux 文件权限

缺乏安全性的系统不是完整的系统。系统中必须有一套能够保护文件免遭非授权用户浏览或修改的机制。Linux 沿用了 Unix 文件权限的办法，即允许用户和组根据每个文件和目录的安全性设置来访问文件。本章将介绍如何在必要时利用 Linux 文件安全系统保护和共享数据。

## Linux 的安全性

Linux 安全系统的核心是用户账户。每个能进入 Linux 系统的用户都会被分配唯一的用户账户。用户对系统中各种对象的访问权限取决于他们登录系统时用的账户。

用户权限是通过创建用户时分配的用户 ID（User ID，通常缩写为 UID）来跟踪的。UID 是数值，每个用户都有唯一的 UID，但在登录系统时用的不是 UID，而是登录名。登录名是用户用来登录系统的最长八字符的字符串（字符可以是数字或字母），同时会关联一个对应的密码。

Linux 系统使用特定的文件和工具来跟踪和管理系统上的用户账户。在我们讨论文件权限之前，先来看一下 Linux 是怎样处理用户账户的。本节会介绍管理用户账户需要的文件和工具，这样在处理文件权限问题时，你就知道如何使用它们了。

### /etc/passwd 文件

Linux 系统使用一个专门的文件来将用户的登录名匹配到对应的 UID 值。这个文件就是/etc/passwd 文件，它包含了一些与用户有关的信息。下面是 Linux 系统上典型的/etc/passwd 文件的一个例子。

```bash
$ cat /etc/passwd
root:x:0:0::/root:/bin/bash
bin:x:1:1::/:/usr/bin/nologin
daemon:x:2:2::/:/usr/bin/nologin
mail:x:8:12::/var/spool/mail:/usr/bin/nologin
ftp:x:14:11::/srv/ftp:/usr/bin/nologin
http:x:33:33::/srv/http:/usr/bin/nologin
nobody:x:65534:65534:Nobody:/:/usr/bin/nologin
dbus:x:81:81:System Message Bus:/:/usr/bin/nologin
systemd-journal-remote:x:982:982:systemd Journal Remote:/:/usr/bin/nologin
systemd-network:x:981:981:systemd Network Management:/:/usr/bin/nologin
systemd-resolve:x:980:980:systemd Resolver:/:/usr/bin/nologin
systemd-timesync:x:979:979:systemd Time Synchronization:/:/usr/bin/nologin
systemd-coredump:x:978:978:systemd Core Dumper:/:/usr/bin/nologin
uuidd:x:68:68::/:/usr/bin/nologin
avahi:x:973:973:Avahi mDNS/DNS-SD daemon:/:/usr/bin/nologin
colord:x:972:972:Color management daemon:/var/lib/colord:/usr/bin/nologin
deepin-anything-server:x:977:977:Deepin Anthing Server:/:/usr/bin/nologin
deepin_anything_server:x:976:976:Deepin Anything Server:/:/usr/bin/nologin
deepin-sound-player:x:975:975:Deepin Sound Player:/:/usr/bin/nologin
deepin-daemon:x:974:974:Deepin Daemon:/:/usr/bin/nologin
lightdm:x:971:971:Light Display Manager:/var/lib/lightdm:/usr/bin/nologin
polkitd:x:102:102:PolicyKit daemon:/:/usr/bin/nologin
rtkit:x:133:133:RealtimeKit:/proc:/usr/bin/nologin
usbmux:x:140:140:usbmux user:/:/usr/bin/nologin
wallen:x:1000:985::/home/wallen:/bin/bash
git:x:970:970:git daemon user:/:/usr/bin/git-shell
nvidia-persistenced:x:143:143:NVIDIA Persistence Daemon:/:/usr/bin/nologin
cups:x:209:209:cups helper user:/:/usr/bin/nologin
dhcpcd:x:969:969:dhcpcd privilege separation:/var/lib/dhcpcd:/usr/bin/nologin
sddm:x:968:968:Simple Desktop Display Manager:/var/lib/sddm:/usr/bin/nologin
geoclue:x:967:967:Geoinformation service:/var/lib/geoclue:/usr/bin/nologin
gdm:x:120:120:Gnome Display Manager:/var/lib/gdm:/usr/bin/nologin
deluge:x:966:966:Deluge BitTorrent daemon:/srv/deluge:/usr/bin/nologin
redsocks:x:965:965:redsocks nologin sysuser:/:/usr/bin/nologin
mysql:x:964:964:MariaDB:/var/lib/mysql:/usr/bin/nologin
```

root 用户账户是 Linux 系统的管理员，固定分配给它的 UID 是 0。就像上例中显示的，Linux 系统会为各种各样的功能创建不同的用户账户，而这些账户并不是真的用户。这些账户叫作`系统账户`，是系统上运行的各种服务进程访问资源用的特殊账户。所有运行在后台的服务都需要用一个系统用户账户登录到 Linux 系统上。

在安全成为一个大问题之前，这些服务经常会用 root 账户登录。遗憾的是，如果有非授权的用户攻陷了这些服务中的一个，他立刻就能作为 root 用户进入系统。为了防止发生这种情况，现在运行在 Linux 服务器后台的几乎所有的服务都是用自己的账户登录。这样的话，即使有人攻入了某个服务，也无法访问整个系统。

Linux 为系统账户预留了 1000 以下的 UID 值。有些服务甚至要用特定的 UID 才能正常工作。为普通用户创建账户时，大多数 Linux 系统会从 1000 开始，将第一个可用 UID 分配给这个账户（并非所有的 Linux 发行版都是这样）。
你可能已经注意到/etc/passwd 文件中还有很多用户登录名和 UID 之外的信息。/etc/passwd 文件的字段包含了如下信息：

- 登录用户名
- 用户密码
- 用户账户的 UID（数字形式）
- 用户账户的组 ID（GID）（数字形式）
- 用户账户的文本描述（称为备注字段）
- 用户 HOME 目录的位置
- 用户的默认 shell

/etc/passwd 文件中的密码字段都被设置成了 x，这并不是说所有的用户账户都用相同的密码。在早期的 Linux 上，/etc/passwd 文件里有加密后的用户密码。但鉴于很多程序都需要访问/etc/passwd 文件获取用户信息，这就成了一个安全隐患。随着用来破解加密密码的工具的不断演进，用心不良的人开始忙于破解存储在/etc/passwd 文件中的密码。Linux 开发人员需要重新考虑这个策略。

现在，绝大多数 Linux 系统都将用户密码保存在另一个单独的文件中（叫作 shadow 文件，位置在/etc/shadow）。只有特定的程序（比如登录程序）才能访问这个文件。

/etc/passwd 是一个标准的文本文件。你可以用任何文本编辑器在/etc/password 文件里直接手动进行用户管理（比如添加、修改或删除用户账户）。但这样做极其危险。如果/etc/passwd 文件出现损坏，系统就无法读取它的内容了，这样会导致用户无法正常登录（即便是 root 用户）。用标准的 Linux 用户管理工具去执行这些用户管理功能就会安全许多。

### /etc/shadow 文件

/etc/shadow 文件对 Linux 系统密码管理提供了更多的控制。只有 root 用户才能访问/etc/shadow 文件，这让它比起/etc/passwd 安全许多。

/etc/shadow 文件为系统上的每个用户账户都保存了一条记录。记录就像下面这样：

```bash
wallen:$6$inJRhswsgTqYbpOp$TjcILRMDZqa6noSe87RMJQpkqS9zGdk/lLovn4M1xYjMpKLY0mlEGJl15IossZF/5ZlJlqnjyDia1tS5RuTTs.:18361:0:99999:7:::
```

在/etc/shadow 文件的每条记录中都有 9 个字段

- 与/etc/passwd 文件中的登录名字段对应的登录名
- 加密后的密码
- 自上次修改密码后过去的天数（自 1970 年 1 月 1 日开始计算）
- 多少天后才能更改密码
- 多少天后必须更改密码
- 密码过期前提前多少天提醒用户更改密码
- 密码过期后多少天禁用用户账户
- 用户账户被禁用的日期（用自 1970 年 1 月 1 日到当天的天数表示）
- 预留字段给将来使用

使用 shadow 密码系统后，Linux 系统可以更好地控制用户密码。它可以控制用户多久更改一次密码，以及什么时候禁用该用户账户，如果密码未更新的话。

### 添加新用户

用来向 Linux 系统添加新用户的主要工具是 useradd。这个命令简单快捷，可以一次性创建新用户账户及设置用户 HOME 目录结构。useradd 命令使用系统的默认值以及命令行参数来设置用户账户。系统默认值被设置在/etc/default/useradd 文件中。

```bash
$ sudo cat /etc/default/useradd
# useradd defaults file for ArchLinux
# original changes by TomK
GROUP=users
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```

在创建新用户时，如果你不在命令行中指定具体的值，useradd 命令就会使用那些默认值。这个例子列出的默认值如下：

- 新用户会被添加到 GID 为 100 的公共组；
- 新用户的 HOME 目录将会位于/home/loginname；
- 新用户账户密码在过期后不会被禁用；
- 新用户账户未被设置过期日期；
- 新用户账户将 bash shell 作为默认 shell；
- 系统会将/etc/skel 目录下的内容复制到用户的 HOME 目录下；
- 系统不会为该用户账户在 mail 目录下创建一个用于接收邮件的文件。

倒数第二个值很有意思。useradd 命令允许管理员创建一份默认的 HOME 目录配置，然后把它作为创建新用户 HOME 目录的模板。这样就能自动在每个新用户的 HOME 目录里放置默认的系统文件。在 Arch Linux 系统上，/etc/skel 目录有下列文件：

```bash
$ ls -al /etc/skel
总用量 20
drwxr-xr-x  2 root root 4096 Aug 20 13:32 .
drwxr-xr-x 99 root root 4096 Nov 26 13:03 ..
-rw-r--r--  1 root root   21 Aug 10 00:27 .bash_logout
-rw-r--r--  1 root root   57 Aug 10 00:27 .bash_profile
-rw-r--r--  1 root root  141 Aug 10 00:27 .bashrc

```

它们是 bash shell 环境的标准启动文件。系统会自动将这些默认文件复制到你创建的每个用户的 HOME 目录。

可以用默认系统参数创建一个新用户账户，然后检查一下新用户的 HOME 目录。

```bash
$ sudo useradd -m test
```

默认情况下，useradd 命令不会创建 HOME 目录，但是-m 命令行选项会使其创建 HOME 目录。你能在此例中看到，useradd 命令创建了新 HOME 目录，并将/etc/skel 目录中的文件复制了过来。

> 运行本章中提到的用户账户管理命令，需要以 root 用户账户登录或者通过 sudo 命令以 root 用户账户身份运行这些命令。

要想在创建用户时改变默认值或默认行为，可以使用 useradd 的额外命令行参数，具体可参见 man。

你会发现，在创建新用户账户时使用命令行参数可以更改系统指定的默认值。但如果总需要修改某个值的话，最好还是修改一下系统的默认值。可以在-D 选项后跟上一个指定的值来修改系统默认的新用户设置。如下示例

```bash
# useradd -D -s /bin/tsch

```

现在，useradd 命令会将 tsch shell 作为所有新建用户的默认登录 shell。

### 删除用户

如果你想从系统中删除用户，userdel 可以满足这个需求。默认情况下，userdel 命令会只删除/etc/passwd 文件中的用户信息，而不会删除系统中属于该账户的任何文件。

如果加上-r 参数，userdel 会删除用户的 HOME 目录以及邮件目录。然而，系统上仍可能存有已删除用户的其他文件。这在有些环境中会造成问题。

下面是用 userdel 命令删除已有用户账户的一个例子。

```bash
$sudo userdel -r test
```

> 在有大量用户的环境中使用-r 参数时要特别小心。你永远不知道用户是否在其 HOME 目录下存放了其他用户或其他程序要使用的重要文件。记住，在删除用户的 HOME 目录之前一定要检查清楚！

### 修改用户

Linux 提供了一些不同的工具来修改已有用户账户的信息。如下列出了这些工具。

- usermod: 修改用户账户的字段，还可以指定主要组以及附加组的所属关系
- passwd: 修改已有用户的密码
- chpasswd: 从文件中读取登录名密码对，并更新密码
- chage: 修改密码的过期日期
- chfn: 修改用户账户的备注信息
- chsh: 修改用户账户的默认登录 shell

每种工具都提供了特定的功能来修改用户账户信息。下面将具体介绍这些工具

#### usermod

usermod 命令是用户账户修改工具中最强大的一个。它能用来修改/etc/passwd 文件中的大部分字段，只需用与想修改的字段对应的命令行参数就可以了。参数大部分跟 useradd 命令的参数一样（比如，-c 修改备注字段，-e 修改过期日期，-g 修改默认的登录组）。除此之外，还有另外一些可能派上用场的选项。

- -l 修改用户账户的登录名。
- -L 锁定账户，使用户无法登录。
- -p 修改账户的密码。
- -U 解除锁定，使用户能够登录。

-L 选项尤其实用。它可以将账户锁定，使用户无法登录，同时无需删除账户和用户的数据。要让账户恢复正常，只要用-U 选项就行了。

#### passwd 和 chpasswd

改变用户密码的一个简便方法就是用 passwd 命令

```bash
$ sudo passwd test
新的密码：
重新输入新的密码：
passwd：已成功更新密码
```

如果只用 passwd 命令，它会改你自己的密码。系统上的任何用户都能改自己的密码，但只有 root 用户才有权限改别人的密码。

-e 选项能强制用户下次登录时修改密码。你可以先给用户设置一个简单的密码，之后再强制在下次登录时改成他们能记住的更复杂的密码。

如果需要为系统中的大量用户修改密码，chpasswd 命令可以事半功倍。chpasswd 命令能从标准输入自动读取登录名和密码对（由冒号分割）列表，给密码加密，然后为用户账户设置。你也可以用重定向命令来将含有 userid:passwd 对的文件重定向给该命令。

```bash
$ sudo chpasswd < users.txt
```

#### chsh、chfn 和 chage

chsh、chfn 和 chage 工具专门用来修改特定的账户信息。chsh 命令用来快速修改默认的用户登录 shell。使用时必须用 shell 的全路径名作为参数，不能只用 shell 名。

```bash
$ sudo chsh -s /bin/csh test
Changing shell for test.
Shell changed.
```

chfn 命令提供了在/etc/passwd 文件的备注字段中存储信息的标准方法。chfn 命令会将用于 Unix 的 finger 命令的信息存进备注字段，而不是简单地存入一些随机文本（比如名字或昵称之类的），或是将备注字段留空。finger 命令可以非常方便地查看 Linux 系统上的用户信息。

```bash
$ sudo finger wallen
Login: wallen                           Name: (null)
Directory: /home/wallen                 Shell: /bin/bash
On since Thu Nov 26 13:02 (CST) on tty1 from :0
    8 hours 54 minutes idle
On since Thu Nov 26 13:02 (CST) on pts/0 from :0
   8 hours 53 minutes idle
On since Thu Nov 26 21:17 (CST) on pts/1 from :0
   5 seconds idle
     (messages off)
No mail.
No Plan.

```

> 出于安全性考虑，很多 Linux 系统管理员会在系统上禁用 finger 命令，不少 Linux 发行版甚至都没有默认安装该命令。在 Arch Linux 上，需要通过 AUR 安装`netkit-bsd-finger`包来使用 finger 命令

如果在使用 chfn 命令时没有参数，它会向你询问要将哪些适合的内容加进备注字段。

```bash
$ sudo chfn test
Changing finger information for test.
Name []: Ima Test
Office []: Director of Technology Office
Phone []: (123)555-1234
Home Phone []: (123)555-9876
Finger information changed.
```

查看/etc/passwd 文件中的记录，你会看到下面这样的结果。

```bash
$ grep test /etc/passwd
test:x:504:504:Ima Test,Director of Technology,(123)555- 1234,(123)555-9876:/home/test:/bin/csh
```

所有的指纹信息现在都存在/etc/passwd 文件中了。

最后，chage 命令用来帮助管理用户账户的有效期。

- -d: 设置上次修改密码到现在的天数
- -E: 设置密码过期的日期
- -I: 设置密码过期到锁定账户的天数
- -m: 设置修改密码之间最少要多少天
- -W: 设置密码过期前多久开始出现提醒信息

chage 命令的日期值可以用下面两种方式中的任意一种：

- YYYY-MM-DD 格式的日期
- 代表从 1970 年 1 月 1 日起到该日期天数的数值

chage 命令中有个好用的功能是设置账户的过期日期。有了它，你就能创建在特定日期自动过期的临时用户，再也不需要记住删除用户了！过期的账户跟锁定的账户很相似：账户仍然存在，但用户无法用它登录。

## 使用 Linux 组

用户账户在控制单个用户安全性方面很好用，但涉及在共享资源的一组用户时就捉襟见肘了。为了解决这个问题，Linux 系统采用了另外一个安全概念——`组`（group）

组权限允许多个用户对系统中的对象（比如文件、目录或设备等）共享一组共用的权限。（更多内容会在后文细述。）

Linux 发行版在处理默认组的成员关系时略有差异。有些 Linux 发行版会创建一个组，把所有用户都当作这个组的成员。遇到这种情况要特别小心，因为一个用户的文件很有可能对其他用户也是可读的。有些发行版会为每个用户创建单独的一个组，这样可以更安全一些。例如，在 KDE 中创建用户， 就会为每个用户创建一个单独的与用户账户同名的组。在添加用户前后可用 grep 命令或 tail 命令查看/etc/group 文件的内容比较。

每个组都有唯一的 GID——跟 UID 类似，在系统上这是个唯一的数值。除了 GID，每个组还有唯一的组名。Linux 系统上有一些组工具可以创建和管理你自己的组。本节将细述组信息是如何保存的，以及如何用组工具创建新组和修改已有的组。

### /etc/group 文件

与用户账户类似，组信息也保存在系统的一个文件中。/etc/group 文件包含系统上用到的每个组的信息。下面是一些来自 Linux 系统上/etc/group 文件中的典型例子。

```bash
root:x:0:root
sys:x:3:bin
mem:x:8:
ftp:x:11:
mail:x:12:
log:x:19:
smmsp:x:25:
proc:x:26:polkitd
games:x:50:
lock:x:54:
network:x:90:
floppy:x:94:
scanner:x:96:
power:x:98:
adm:x:999:daemon
wheel:x:998:wallen,test,test2
kmem:x:997:
tty:x:5:
utmp:x:996:
audio:x:995:
disk:x:994:
input:x:993:
kvm:x:992:
lp:x:991:cups
optical:x:990:
render:x:989:
storage:x:988:
uucp:x:987:
video:x:986:sddm
users:x:985:
systemd-journal:x:984:
rfkill:x:983:
bin:x:1:daemon
daemon:x:2:bin
http:x:33:
nobody:x:65534:
dbus:x:81:
systemd-journal-remote:x:982:
systemd-network:x:981:
systemd-resolve:x:980:
systemd-timesync:x:979:
systemd-coredump:x:978:
uuidd:x:68:
avahi:x:973:
colord:x:972:
lightdm:x:971:
polkitd:x:102:
rtkit:x:133:
usbmux:x:140:
git:x:970:
nvidia-persistenced:x:143:
cups:x:209:
dhcpcd:x:969:
sddm:x:968:
geoclue:x:967:
gdm:x:120:
deluge:x:966:
redsocks:x:965:
mysql:x:964:
vboxusers:x:108:
vboxsf:x:109:
wallen:x:1000:
```

和 UID 一样，GID 在分配时也采用了特定的格式。系统账户用的组通常会分配低于 1000 的 GID 值，而用户组的 GID 则会从 1000 开始分配。/etc/group 文件有 4 个字段：

- 组名
- 组密码
- GID
- 属于该组的用户列表

组密码允许非组内成员通过它临时成为该组成员。这个功能并不很普遍，但确实存在。

千万不能通过直接修改/etc/group 文件来添加用户到一个组，要用 usermod 命令。在添加用户到不同的组之前，首先得创建组(创建用户时，默认生成的与用户名同名的组除外)。

> 用户账户列表某种意义上有些误导人。你会发现，在列表中，有些组并没有列出用户。这并不是说这些组没有成员。当一个用户在/etc/passwd 文件中指定某个组作为默认组时，用户账户不会作为该组成员再出现在/etc/group 文件中。多年以来，被这个问题难倒的系统管理员可不是一两个呢。如果想要严谨的查看一个组所有的全部用户，可以先取/etc/passwd 下以该组为默认组的用户，再加上/etc/group 下该组的用户，合并这两部分，就能得到该组全部的用户列表了。除此之外，以用户的维度，可以使用`id`命令查看一个用户所属的组的情况。

### 创建新组

groupadd 命令可在系统上创建新组。

```bash
$ sudo groupadd shared
```

在创建新组时，默认没有用户被分配到该组。groupadd 命令没有提供将用户添加到组中的选项，但可以用 usermod 命令来弥补这一点。

```bash
$ sudo usermod -G shared test
```

usermod 命令的-G 选项会把这个新组添加到该用户账户的组列表里。

> 如果更改了已登录系统账户所属的用户组，该用户必须登出系统后再登录，组关系的更改才能生效。

> 为用户账户分配组时要格外小心。如果加了-g 选项，指定的组名会替换掉该账户的默认组。-G 选项则将该组添加到用户的属组的列表里，不会影响默认组。

### 修改组

在/etc/group 文件中可以看到，需要修改的组信息并不多。groupmod 命令可以修改已有组的 GID（加-g 选项）或组名（加-n 选项）。

```bash
$ groupmod -n sharing shared
```

原 shared 组被更名为 sharing。

修改组名时，GID 和组成员不会变，只有组名改变。由于所有的安全权限都是基于 GID 的，你可以随意改变组名而不会影响文件的安全性。

## 理解文件权限

现在你已经了解了用户和组，是时候解读 ls 命令输出时所出现的谜一般的文件权限了。本节将会介绍如何对权限进行分析以及它们的来历。

### 使用文件权限符

使用 ls -l 可以查看到文件的权限情况

```bash
$ ls –l

```
