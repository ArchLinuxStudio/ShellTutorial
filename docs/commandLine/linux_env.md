# Linux 环境变量

Linux 环境变量能帮你提升 Linux shell 体验。很多程序和脚本都通过环境变量来获取系统信息、存储临时数据和配置信息。在 Linux 系统上有很多地方可以设置环境变量，了解去哪里设置相应的环境变量很重要。

## 认识环境变量

bash shell 用`环境变量`（environment variable）的特性来存储有关 shell 会话和工作环境的信息（这也是它们被称作环境变量的原因）。这项特性允许你在内存中存储数据，以便程序或 shell 中运行的脚本能够轻松访问到它们。这也是存储持久数据的一种简便方法。
在 bash shell 中，环境变量分为两类：

- 全局变量
- 局部变量

全局环境变量对于 shell 会话和所有生成的子 shell 都是可见的。局部变量则只对创建它们的 shell 可见。这让全局环境变量对那些所创建的子 shell 需要获取父 shell 信息的程序来说非常有用。Linux 系统在你开始 bash 会话时就设置了一些全局环境变量。系统环境变量基本上都是使用全大写字母，以区别于普通用户的环境变量。要查看全局变量，可以使用 env 或 printenv 命令。

系统为 bash shell 设置的全局环境变量数目众多，我们不得不在展示的时候进行删减。其中有很多是在登录过程中设置的，另外，你的登录方式也会影响到所设置的环境变量。

要显示个别环境变量的值，可以使用 printenv 命令，但是不要用 env 命令。

```bash
$ printenv HOME
/home/Christine
$
$ env HOME
env: HOME: No such file or directory
$
```

也可以使用 echo 显示变量的值。在这种情况下引用某个环境变量的时候，必须在变量前面加上一个美元符（\$）。

```bash
$ echo $HOME
/home/Christine
$
```

在变量名前加上\$也可在其他命令中作为参数使用

```bash
$ ls $HOME # 等价与ls ~
```

正如前面提到的，全局环境变量可用于进程的所有子 shell。

```bash
$ bash
$
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
501       2017  2016  0 16:00 pts/0    00:00:00 -bash
501       2082  2017  0 16:08 pts/0    00:00:00 bash
501       2095  2082  0 16:08 pts/0    00:00:00 ps -f
$
$ echo $HOME
/home/Christine
$
$ exit
$
```

在这个例子中，用 bash 命令生成一个子 shell 后，显示了 HOME 环境变量的当前值，这个值和父 shell 中的一模一样，都是/home/Chrisine。

---

局部环境变量,顾名思义只能在定义它们的进程中可见。尽管它们是局部的，但是和全局环境变量一样重要。事实上，Linux 系统也默认定义了标准的局部环境变量。不过你也可以定义自己的局部变量，如你所想，这些变量被称为用户定义局部变量。

查看局部环境变量的列表有点复杂。遗憾的是，在 Linux 系统并没有一个只显示局部环境变量的命令。set 命令会显示为某个特定进程设置的所有环境变量，包括局部变量、全局变量以及用户定义变量。

所有通过 printenv 命令能看到的全局环境变量都出现在了 set 命令的输出中。但在 set 命令的输出中还有其他一些环境变量，即局部环境变量和用户定义变量。

> 命令 env、printenv 和 set 之间的差异很细微。set 命令会显示出全局变量、局部变量以及用户定义变量。它还会按照字母顺序对结果进行排序。env 和 printenv 命令同 set 命令的区别在于前两个命令不会对变量排序，也不会输出局部变量和用户定义变量。在这种情况下，env 和 printenv 的输出是重复的。不过 env 命令除了查看环境变量外还有一些其他功能。

## 操作环境变量

一旦启动了 bash shell（或者执行一个 shell 脚本），就能创建在这个 shell 进程内可见的`局部变量`了。可以通过等号给环境变量赋值，值可以是数值或字符串

```bash
$ echo $my_variable
$ my_variable=Hello
$
$ echo $my_variable
Hello
```

非常简单！现在每次引用 my_variable 环境变量的值，只要通过\$my_variable 引用即可。如果要给变量赋一个含有空格的字符串值，必须用引号来界定字符串的首和尾。没有单引号的话，bash shell 会以为下一个词是另一个要执行的命令。注意，你定义的局部环境变量用的是小写字母，而到目前为止你所看到的系统环境变量都是大写字母。

> 所有的环境变量名均使用大写字母，这是 bash shell 的标准惯例。如果是你自己创建的局部变量或是 shell 脚本，请使用小写字母。变量名区分大小写。在涉及用户定义的局部变量时坚持使用小写字母，这能够避免重新定义系统环境变量可能带来的灾难。

设置了局部环境变量后，就能在 shell 进程的任何地方使用它了。但是，如果生成了另外一个 子 shell，它在子 shell 中就不可用。当你退出子 shell 并回到原来的 shell 时，这个局部环境变量依然可用。

类似地，如果你在子进程中设置了一个局部变量，那么一旦你退出了子进程，那个局部环境变量就不可用。

这种时候可以设置全局环境变量。在设定全局环境变量的进程所创建的子进程中，该变量都是可见的。创建全局环境变量的方法是先创建一个局部环境变量，然后再把它导出到全局环境中。这个过程通过 export 命令来完成。

```bash
$ my_variable="I am Global now"
$ export my_variable
```

在定义并导出局部环境变量 my_variable 后，可通过 bash 命令启动一个子 shell。在这个子 shell 中能够正确的显示出变量 my_variable 的值。该变量能够保留住它的值是因为 export 命令使其变成了全局环境变量。

**修改子 shell 中全局环境变量并不会影响到父 shell 中该变量的值。除此之外，子 shell 甚至无法使用 export 命令改变父 shell 中全局环境变量的值。**

当然，既然可以创建新的环境变量，自然也能删除已经存在的环境变量。可以用 unset 命令完成这个操作。在 unset 命令中引用环境变量时，记住不要使用\$。

```bash
$ echo $my_variable
I am Global now
$
$ unset my_variable
$
$ echo $my_variable
$
```

> 在涉及环境变量名时，什么时候该使用$，什么时候不该使用$，实在让人摸不着头脑。记住一点就行了：如果要用到变量，使用$；如果要操作变量，不使用$。这条规则的一个例外就是使用 printenv 显示某个变量的值

在处理全局环境变量时，事情就有点棘手了。如果你是在子进程中删除了一个全局环境变量，这只对子进程有效。该全局环境变量在父进程中依然可用。和修改变量一样，在子 shell 中删除全局变量后，你无法将效果反映到父 shell 中。

## 默认的环境变量

默认情况下，bash shell 会用一些特定的环境变量来定义系统环境。这些变量在你的 Linux 系统上都已经设置好了，只管放心使用。bash shell 源自当初的 Unix Bourne shell，因此也保留了 Unix Bourne shell 里定义的那些环境变量。除了默认的 Bourne 的环境变量，bash shell 还提供一些自有的变量。你可能已经注意到，不是所有的默认环境变量都会在运行 set 命令时列出。尽管这些都是默认环境变量，但并不是每一个都必须有一个值。

其中最重要的一个环境变量为 PATH。当你在 shell 命令行界面中输入一个外部命令时，shell 必须搜索系统来找到对应的程序。PATH 环境变量定义了用于进行命令和程序查找的目录。在本书所用的 Arch Linux 系统中，PATH 环境变量的内容是这样的：

```bash
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/bin/site_perl:/usr/bin/vendor_perl:/usr/bin/core_perl
```

输出中显示了多个可供 shell 用来查找命令和程序的目录。PATH 中的目录使用冒号分隔。如果命令或者程序的位置没有包括在 PATH 变量中，那么如果不使用绝对路径的话，shell 是没法找到的。如果 shell 找不到指定的命令或程序，它会产生一个经典的错误信息：`command not found`。

一般来说，默认环境变量有很多，在需要用到时查阅用法即可，不必全部记忆。

可以把新的搜索目录添加到现有的 PATH 环境变量中，无需从头定义。PATH 中各个目录之间是用冒号分隔的。你只需引用原来的 PATH 值，然后再给这个字符串添加新目录就行了。

```bash
$ PATH=$PATH:/home/christine/Scripts
```

一些程序员习惯将单点符也加入 PATH 环境变量。该单点符代表当前目录

```bash
$ PATH=$PATH:.
```

如此对 PATH 变量的修改只能持续到终端退出或重启系统。这种效果并不能一直持续。在下一节中，你会学到如何永久保持环境变量的修改效果。

## 定位系统环境变量

在你登入 Linux 系统启动一个 bash shell 时，默认情况下 bash 会在几个文件中查找命令。这些文件叫作启动文件或环境文件。bash 检查的启动文件取决于你启动 bash shell 的方式。启动 bash shell 有 3 种方式：

- 登录时作为默认登录 shell
- 作为非登录 shell 的交互式 shell
- 作为运行脚本的非交互 shell

下面几节介绍了 bash shell 在不同的方式下启动文件。

### 登录 shell

当你登录 Linux 系统时，bash shell 会作为登录 shell 启动。登录 shell 会从 5 个不同的启动文件里读取命令：

- /etc/profile
- \$HOME/.bash_profile
- \$HOME/.bashrc
- \$HOME/.bash_login
- \$HOME/.profile

/etc/profile 文件是系统上默认的 bash shell 的主启动文件。系统上的每个用户登录时都会执行这个启动文件

> 要留意的是有些 Linux 发行版使用了可拆卸式认证模块（Pluggable Authentication Modules ，PAM）。在这种情况下，PAM 文件会在 bash shell 启动之前处理，这些文件中可能会包含环境变量。PAM 文件包括/etc/environment 文件和\$HOME/.pam_environment 文件。PAM 更多的相关信息可以在 http://linux-pam.org 中找到。

另外 4 个启动文件是针对用户的，可根据个人需求定制。注意，这四个文件都以点号开头，这说明它们是隐藏文件（不会在通常的 ls 命令输出列表中出现）。它们位于用户的 HOME 目录下，所以每个用户都可以编辑这些文件并添加自己的环境变量，这些环境变量会在每次启动 bash shell 会话时生效。

> Linux 发行版在环境文件方面存在的差异非常大。本节中所列出的$HOME下的那些文件并非每个用户都有。例如有些用户可能只有一个$HOME/.bash_profile 文件。这很正常。

1. /etc/profile 文件

/etc/profile 文件是 bash shell 默认的的主启动文件。只要你登录了 Linux 系统，bash 就会执行/etc/profile 启动文件中的命令。不同的 Linux 发行版在这个文件里放了不同的命令。每个发行版的/etc/profile 文件都有不同的设置和命令。例如，在 Arch Linux 的/etc/profile 文件中，涉及了一个叫作/etc/bash.bashrc 的文件。这个文件包含了系统环境变量等其他内容。但是，/etc/profile 各个发行版有所不同，在 CentOS 发行版的/etc/profile 文件中，并没有出现这个文件。

许多发行版的/etc/profile 文件都用到了同一个特性：for 语句。它用来迭代/etc/profile.d 目录下的所有文件（该语句会在后续章节详述）。这为 Linux 系统提供了一个放置特定应用程序启动文件的地方，当用户登录时，shell 会执行这些文件。在本书所用的 Arch Linux 系统中，/etc/profile.d 目录下包含以下文件：

```bash
$ ls -l /etc/profile.d
总用量 36
-rw-r--r-- 1 root root  545 Oct 20 17:10 freetype2.sh
-rw-r--r-- 1 root root 1107 Apr 15  2020 gawk.csh
-rw-r--r-- 1 root root  757 Apr 15  2020 gawk.sh
-rwxr-xr-x 1 root root  105 Sep 25 21:52 gpm.sh
-rw-r--r-- 1 root root  766 Sep  3 06:30 locale.sh
-rw-r--r-- 1 root root  468 Sep 18 05:12 perlbin.csh
-rw-r--r-- 1 root root  464 Sep 18 05:12 perlbin.sh
```

不难发现，有些文件与系统中的特定应用有关。大部分应用都会创建两个启动文件：一个供 bash shell 使用（使用.sh 扩展名），一个供 c shell 使用（使用.csh 扩展名）。locale.sh 文件会尝试去判定系统上所采用的默认语言字符集，然后设置对应的 LANG 环境变量。

2. \$HOME 目录下的启动文件

这些启动文件都起着同一个作用：提供一个用户专属的启动文件来定义该用户所用到的环境变量。大多数 Linux 发行版只用这四个启动文件中的一到两个：

- \$HOME/.bash_profile
- \$HOME/.bashrc
- \$HOME/.bash_login
- \$HOME/.profile

shell 会按照按照下列顺序，运行第一个被找到的文件，余下的则被忽略：

- \$HOME/.bash_profile
- \$HOME/.bash_login
- \$HOME/.profile

注意，这个列表中并没有\$HOME/.bashrc 文件。这是因为该文件通常通过其他文件运行的。

> 记住，\$HOME 表示的是某个用户的主目录。它和波浪号（~）的作用一样。

Arch Linux 系统中的.bash_profile 文件的内容如下

```bash
$ cat $HOME/.bash_profile
#
# ~/.bash_profile
#

[[ -f ~/.bashrc ]] && . ~/.bashrc
```

.bash_profile 启动文件会先去检查 HOME 目录中是不是还有一个叫.bashrc 的启动文件。如果有的话，会先执行启动文件里面的命令。

### 交互式 shell 进程

如果你的 bash shell 不是登录系统时启动的（比如是在命令行提示符下敲入 bash 时启动），那么你启动的 shell 叫作`交互式` shell。交互式 shell 不会像登录 shell 一样运行，但它依然提供了命令行提示符来输入命令。

如果 bash 是作为交互式 shell 启动的，它就不会访问/etc/profile 文件，只会检查用户 HOME 目录中的.bashrc 文件。

在本书所用的 Arch Linux 系统上，这个文件看起来如下：

```bash
$ cat .bashrc
#
# ~/.bashrc
#

# If not running interactively, don't do anything
[[ $- != *i* ]] && return

alias ls='ls --color=auto'
PS1='[\u@\h \W]\$'
```

.bashrc 文件有两个作用：一是查看并执行/etc 目录下通用的 bashrc 文件(/etc/bashrc)，在 Arch Linux 上无此表现，但是在 Centos 上是存在的。二是为用户提供一个定制自己的命令别名和私有脚本函数（将在后文讲到）的地方。

> 上面的 PS1 值就是终端下提示符的格式，如`[testuser@archlinux ~]$`

### 非交互式 shell

最后一种 shell 是非交互式 shell。系统执行 shell 脚本时用的就是这种 shell。不同的地方在于它没有命令行提示符。但是当你在系统上运行脚本时，也许希望能够运行一些特定启动的命令。

> 脚本能以不同的方式执行。只有其中的某一些方式能够启动子 shell。你会在后续学习到 shell 不同的执行方式。

为了处理这种情况，bash shell 提供了 BASH_ENV 环境变量。当 shell 启动一个非交互式 shell 进程时，它会检查这个环境变量来查看要执行的启动文件。如果有指定的文件，shell 会执行该文件里的命令，这通常包括 shell 脚本变量设置。

在本书所用的 Arch Linux 发行版中，变量 BASH_ENV 没有被设置。记住，如果变量未设置，echo 命令会显示一个空行，然后返回 CLI 提示符：

```bash
$ printenv BASH_ENV

$
```

那如果 BASH_ENV 变量没有设置，shell 脚本到哪里去获得它们的环境变量呢？别忘了有些 shell 脚本是通过启动一个子 shell 来执行的。子 shell 可以继承父 shell 导出过的变量。

举例来说，如果父 shell 是登录 shell，在/etc/profile、/etc/profile.d/\*.sh 和\$HOME/.bashrc 文件中设置并导出了变量，用于执行脚本的子 shell 就能够继承这些变量。

要记住，由父 shell 设置但并未导出的变量都是局部变量。子 shell 无法继承局部变量。

对于那些不启动子 shell 的脚本，变量已经存在于当前 shell 中了。所以就算没有设置 BASH_ENV，也可以使用当前 shell 的局部变量和全局变量。

### 环境变量持久化

现在你已经了解了各种 shell 进程以及对应的环境文件，找出永久性环境变量就容易多了。也可以利用这些文件创建自己的永久性全局变量或局部变量。

对全局环境变量来说（Linux 系统中所有用户都需要使用的变量），可能更倾向于将新的或修改过的变量设置放在/etc/profile 文件中，但这可不是什么好主意。如果你升级了所用的发行版，这个文件也会跟着更新，那你所有定制过的变量设置可就都没有了。

最好是在/etc/profile.d 目录中创建一个以.sh 结尾的文件。把所有新的或修改过的全局环境变量设置放在这个文件中。

在大多数发行版中，存储个人用户永久性 bash shell 变量的地方是$HOME/.bashrc文件。这一点适用于所有类型的shell进程。但如果设置了BASH_ENV变量，那么记住，除非它指向的是$HOME/.bashrc，否则你应该将非交互式 shell 的用户变量放在别的地方。

> 图形化界面组成部分（如 GUI 客户端）的环境变量可能需要在另外一些配置文件中设置，这和设置 bash shell 环境变量的地方可能不一样。

想想之前讲过的 alias 命令设置就是不能持久的。你可以把自己的 alias 设置放在\$HOME/.bashrc 启动文件中，使其效果永久化。

## 数组变量

环境变量有一个很酷的特性就是，它们可作为`数组`使用。数组是能够存储多个值的变量。这些值可以单独引用，也可以作为整个数组来引用。
要给某个环境变量设置多个值，可以把值放在括号里，值与值之间用空格分隔。

```bash
mytest=(one two three four five)
```

没什么特别的地方。如果你想把数组像普通的环境变量那样显示，你会失望的。

```bash
$ echo $mytest
one
```

只有数组的第一个值显示出来了。要引用一个单独的数组元素，就必须用代表它在数组中位置的数值索引值。索引值要用方括号括起来。

```bash
$ echo ${mytest[2]}
three
```

> 环境变量数组的索引值都是从零开始。这通常会带来一些困惑。

要显示整个数组变量，可用星号作为通配符放在索引值的位置。

```bash
$ echo ${mytest[*]}
one two three four five
```

甚至能用 unset 命令删除数组中的某个值，但是要小心，这可能会有点复杂。看下面的例子。

```bash
$ unset mytest[2]
$
$ echo ${mytest[*]}
one two four five
$
$ echo ${mytest[2]}

$ echo ${mytest[3]}
four
$
```

这个例子用 unset 命令删除在索引值为 2 的位置上的值。显示整个数组时，看起来像是索引里面已经没这个索引了。但当专门显示索引值为 2 的位置上的值时，就能看到这个位置是空的。

最后，可以在 unset 命令后跟上数组名来删除整个数组。

```bash
$ unset mytest
$
$ echo ${mytest[*]}

$
```

有时数组变量会让事情很麻烦，所以在 shell 脚本编程时并不常用。对其他 shell 而言，数组变量的可移植性并不好，如果需要在不同的 shell 环境下从事大量的脚本编写工作，这会带来很多不便。有些 bash 系统环境变量使用了数组（比如 BASH_VERSINFO），但总体上不会太频繁用到。
