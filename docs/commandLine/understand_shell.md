# 理解 Shell

## shell 的父子关系

用于登录的某个虚拟控制器终端，或在 GUI 中运行终端仿真器时所启动的默认的交互 shell，是一个父 shell。本书到目前为止都是父 shell 提供 CLI 提示符，然后等待命令输入。

### 查看父子结构

在 CLI 提示符后输入/bin/bash 命令或其他等效的 bash 命令时，会创建一个新的 shell 程序。这个 shell 程序被称为子 shell（child shell）。子 shell 也拥有 CLI 提示符，同样会等待命令输入。当输入 bash、生成子 shell 的时候，你是看不到任何相关的信息的，因此需要另一条命令帮助我们理清这一切。在之前讲过的 ps 命令能够派上用场，在生成子 shell 的前后配合选项-f 来观察不同。

```bash
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
501       1841  1840  0 11:50 pts/0    00:00:00 -bash
501       2429  1841  4 13:44 pts/0    00:00:00 ps -f
$
$ bash
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
501       1841  1840  0 11:50 pts/0    00:00:00 -bash
501       2430  1841  0 13:44 pts/0    00:00:00 bash
501       2444  2430  1 13:44 pts/0    00:00:00 ps -f
$
```

第一次使用 ps -f 的时候，显示出了两个进程。其中一个进程的进程 ID 是 1841（第二列），运行的是 bash shell 程序（最后一列）。另一个进程（进程 ID 为 2429）对应的是命令 ps -f。

输入命令 bash 之后，一个子 shell 就出现了。第二个 ps -f 是在子 shell 中执行的。可以从显示结果中看到有两个 bash shell 程序在运行。第一个 bash shell 程序，也就是父 shell 进程，其原始进程 ID 是 1814。第二个 bash shell 程序，即子 shell 进程，其 PID 是 2430。注意，子 shell 的父进程 ID（PPID）是 1841，指明了这个父 shell 进程就是该子 shell 的父进程。

进程就是正在运行的程序。bash shell 是一个程序，当它运行的时候，就成为了一个进程。一个运行着的 shell 就是某种进程而已。因此，在说到运行一个 bash shell 的时候，你经常会看到“shell”和“进程”这两个词交换使用。

在生成子 shell 进程时，只有`部分`父进程的环境被复制到子 shell 环境中。这会对包括变量在内的一些东西造成影响，我们会在后续谈及相关的内容。

同样的，你也可以在子 shell 中不停的继续创建子 shell,它们最终会形成一个嵌套结构，可以用 ps --forest 命令展示了这些子 shell 间的嵌套结构。可以利用 exit 命令有条不紊地退出各个子 shell。

另一个创建子 shell 的方式是使用`进程列表`。命令列表要想成为进程列表，这些命令必须包含在括号里。

```bash
$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls)
```

括号的加入使一串命令变成了进程列表，生成了一个子 shell 来执行对应的命令。

> 进程列表是一种命令分组（command grouping）。另一种命令分组是将命令放入花括号中，并在命令列表尾部加上分号（;），前后的空格均不可省略。语法为{ command; }。使用花括号进行命令分组并不会像进程列表那样创建出子 shell。

要想知道是否生成了子 shell，得借助一个使用了环境变量的命令。（环境变量会在后续详述。）这个命令就是 echo \$BASH_SUBSHELL。如果该命令返回 0，就表明没有子 shell。如果返回 1 或者其他更大的数字，就表明存在一个或多个子 shell。

下面的例子中使用了一串命令列表，列表尾部是 echo \$BASH_SUBSHELL

```bash
$ pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL
...
0
```

在命令输出的最后，显示的是数字 0。这就表明这些命令不是在子 shell 中运行的。要是使用`进程列表`的话，结果就不一样了。在列表最后加入 echo \$BASH_SUBSHELL。

```bash
$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL)
...
1
```

这次在命令输入的最后显示出了数字 1。这表明的确创建了子 shell，并用于执行这些命令。所以说，`进程列表`就是使用括号包围起来的一组命令，它能够创建出子 shell 来执行这些命令。你甚至可以在`进程列表`中嵌套括号来创建子 shell 的子 shell。

```bash
$ ( pwd ; echo $BASH_SUBSHELL)
/home/Christine
1
```

```bash
$ ( pwd ; (echo $BASH_SUBSHELL))
/home/Christine
2
```

注意，在第一个进程列表中，数字 1 表明了一个子 shell，这个结果和预期的一样。但是在第二个进程列表中，在命令 echo \$BASH_SUBSHELL 外面又多出了一对括号。这对括号在子 shell 中产生了另一个子 shell 来执行命令。因此数字 2 表明的就是这个子 shell。

### 后台模式

在 shell 脚本中，经常使用子 shell 进行多进程处理。但是采用子 shell 的成本不菲，会明显拖慢处理速度。在交互式的 CLI shell 会话中，子 shell 同样存在问题。它并非真正的多进程处理，因为终端控制着子 shell 的 I/O。

在交互式的 shell CLI 中，还有很多更富有成效的子 shell 用法。进程列表、协程和管道（后续会讲到）都利用了子 shell。它们都可以有效地在交互式 shell 中使用。在交互式 shell 中，一个高效的子 shell 用法就是使用后台模式。在讨论如何将后台模式与子 shell 搭配使用之前，你得先搞明白什么是后台模式。

在后台模式中运行命令可以在处理命令的同时让出 CLI，以供他用。演示后台模式的一个经典命令就是 sleep。sleep 命令接受一个参数，该参数是你希望进程等待（睡眠）的秒数。这个命令在脚本中常用于引入一段时间的暂停。命令 sleep 10 会将会话暂停 10 秒钟，然后返回 shell CLI 提示符。

```bash
$ sleep 10
```

要想将命令置入后台模式，可以在命令末尾加上字符&。把 sleep 命令置入后台模式可以让我们利用 ps 命令来小窥一番。

```bash
$ sleep 3000 &
[1] 2396
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
christi+  2338  2337  0 10:13 pts/9    00:00:00 -bash
christi+  2396  2338  0 10:17 pts/9    00:00:00 sleep 3000
christi+  2397  2338  0 10:17 pts/9    00:00:00 ps -f
$
```

sleep 命令会在后台（&）睡眠 3000 秒（50 分钟）。当它被置入后台，在 shell CLI 提示符返回之前，会出现两条信息。第一条信息是显示在方括号中的后台作业（background job）号（1）。第二条是后台作业的进程 ID（2396）。

ps 命令用来显示各种进程。我们可以注意到命令 sleep 3000 已经被列出来了。在第二列显示的进程 ID（PID）和命令进入后台时所显示的 PID 是一样的，都是 2396。

除了 ps 命令，你也可以使用 jobs 命令来显示后台作业信息。jobs 命令可以显示出当前运行在后台模式中的所有用户的进程（作业）。

```bash
$ jobs
[1]+  Running                 sleep 3000 &
$
```

jobs 命令在方括号中显示出作业号（1）。它还显示了作业的当前状态（running）以及对应的命令（sleep 3000 &）

利用 jobs 命令的-l 选项，你还能够看到更多的相关信息。除了默认信息之外，-l 选项还能够显示出命令的 PID。
一旦后台作业完成，就会显示出结束状态。

```bash
$ jobs -l
[1]+ 28331 Done                 sleep 3000 &
$
```

> 需要提醒的是：后台作业的结束状态可未必会一直等待到合适的时候才现身。当作业结束状态突然出现在屏幕上的时候，你可别吃惊啊。

之前说过，进程列表是运行在子 shell 中的一条或多条命令。使用包含了 sleep 命令的进程列表，并显示出变量 BASH_SUBSHELL，结果和期望的一样。

```bash
$ (sleep 2 ; echo $BASH_SUBSHELL ; sleep 2)
1
```

在上面的例子中，有一个 2 秒钟的暂停，接着显示出的数字 1 表明只有一个子 shell，在返回提示符之前又经历了另一个 2 秒钟的暂停，没什么特别的。

将相同的进程列表置入后台模式会在命令输出上表现出些许不同。

```bash
$ (sleep 2 ; echo $BASH_SUBSHELL ; sleep 2)&
[2] 2401
$ 1
[2]+  Done                  ( sleep 2; echo $BASH_SUBSHELL; sleep 2 )
```

把进程列表置入后台会产生一个作业号和进程 ID，然后返回到提示符。不过奇怪的是表明单一级子 shell 的数字 1 显示在了提示符的旁边！不要不知所措，只需要按一下回车键，就会得到另一个提示符。

在 CLI 中运用子 shell 的创造性方法之一就是将进程列表置入后台模式。你既可以在子 shell 中进行繁重的处理工作，同时也不会让子 shell 的 I/O 受制于终端。

当然了，sleep 和 echo 命令的进程列表只是作为一个示例而已。使用 tar 创建备份文件是有效利用后台进程列表的一个更实用的例子。

```bash
$ (tar -cf Rich.tar /home/rich ; tar -cf My.tar /home/christine)&
[3] 2423
```

### 协程的使用

将进程列表置入后台模式并不是子 shell 在 CLI 中仅有的创造性用法。协程就是另一种方法。协程可以同时做两件事。它在后台生成一个子 shell，并在这个子 shell 中执行命令。要进行协程处理，得使用 coproc 命令，还有要在子 shell 中执行的命令。

```bash
$ coproc sleep 10
[1] 2544
$
```

除了会创建子 shell 之外，协程基本上就是将命令置入后台模式。当输入 coproc 命令及其参数之后，你会发现启用了一个后台作业。屏幕上会显示出后台作业号（1）以及进程 ID（2544）。jobs 命令能够显示出协程的处理状态

```bash
$ jobs
[1]+  Running                 coproc COPROC sleep 10 &
```

在上面的例子中可以看到在子 shell 中执行的后台命令是 coproc COPROC sleep 10。COPROC 是 coproc 命令给进程起的名字。你可以使用命令的扩展语法自己设置这个名字。

```bash
$ coproc My_Job { sleep 10; }
[1] 2570
$
$ jobs [1]+  Running                 coproc My_Job { sleep 10; } &
$
```

通过使用扩展语法，协程的名字被设置成 My_Job。这里要注意的是，扩展语法写起来有点麻烦。必须确保在第一个花括号（{）和命令名之间有一个空格。还必须保证命令以分号（;）结尾。另外，分号和闭花括号（}）之间也得有一个空格。

> 协程能够让你尽情发挥想象力，发送或接收来自子 shell 中进程的信息。只有在拥有多个协程的时候才需要对协程进行命名，因为你得和它们进行通信。否则的话，让 coproc 命令将其设置成默认的名字 COPROC 就行了。

你可以发挥才智，将协程与进程列表结合起来产生嵌套的子 shell。只需要输入进程列表，然后把命令 coproc 放在前面就行了。

```bash
$ coproc ( sleep 10; sleep 2 )
[1] 143311
$ jobs
[1]+  Running     coproc COPROC ( sleep 10; sleep 2 ) &
$ ps -f --forest
wallen    142848  142839  0 18:00 pts/1    00:00:00 /bin/bash
wallen    143311  142848  0 18:01 pts/1    00:00:00  \_ /bin/bash
wallen    143312  143311  0 18:01 pts/1    00:00:00  |   \_ sleep 10
wallen    143776  142848  0 18:02 pts/1    00:00:00  \_ ps -f --forest
```

## 理解 shell 的内建命令

在学习 GNU bash shell 期间，你可能听到过“内建命令”这个术语。搞明白 shell 的内建命令和非内建（外部）命令非常重要。内建命令和非内建命令的操作方式大不相同。

外部命令，有时候也被称为文件系统命令，是存在于 bash shell 之外的程序。它们并不是 shell 程序的一部分。外部命令程序通常位于/bin、/usr/bin、/sbin 或/usr/sbin 中。

ps 就是一个外部命令。你可以使用 which 和 type 命令找到它。

```bash
$ which ps
/bin/ps
$
$ type -a ps
ps is /bin/ps
$
$ ls -l /bin/ps
-rwxr-xr-x 1 root root 93232 Jan  6 18:32 /bin/ps
$
```

当外部命令执行时，会创建出一个子进程。这种操作被称为`衍生`（forking）。外部命令 ps 很方便显示出它的父进程以及自己所对应的衍生子进程。

```bash
$ ps -f
UID        PID  PPID  C STIME TTY          TIME CMD
christi+  2743  2742  0 17:09 pts/9    00:00:00 -bash
christi+  2801  2743  0 17:16 pts/9    00:00:00 ps -f
$
```

作为外部命令，ps 命令执行时会创建出一个子进程。在这里，ps 命令的 PID 是 2801，父 PID 是 2743。作为父进程的 bash shell 的 PID 是 2743。

当进程必须执行衍生操作时，它需要花费时间和精力来设置新子进程的环境。所以说，外部命令多少还是有代价的。

> 就算衍生出子进程或是创建了子 shell，你仍然可以通过发送信号与其沟通，这一点无论是在命令行还是在脚本编写中都是极其有用的。发送`信号`（signaling）使得进程间可以通过信号进行通信。信号及其发送会在后续章节中讲到。

---

`内建命令`和外部命令的区别在于前者不需要使用子进程来执行。它们已经和 shell 编译成了一体，作为 shell 工具的组成部分存在。不需要借助外部程序文件来运行

cd 和 exit 命令都内建于 bash shell。可以利用 type 命令来了解某个命令是否是内建的。

```bash
$ type cd
cd is a shell builtin
$
$ type exit
exit is a shell builtin
$
```

因为既不需要通过衍生出子进程来执行，也不需要打开程序文件，内建命令的执行速度要更快，效率也更高。

要注意，有些命令有多种实现。例如 echo 和 pwd 既有内建命令也有外部命令。两种实现略有不同。要查看命令的不同实现，使用 type 命令的-a 选项。

```bash
$ type -a echo
echo is a shell builtin
echo is /bin/echo
$
$ which echo
/bin/echo
$
$ type -a pwd
pwd is a shell builtin
pwd is /bin/pwd
$
$ which pwd
/bin/pwd
$
```

命令 type -a 显示出了每个命令的两种实现。注意，which 命令只显示出了外部命令文件。

> 对于有多种实现的命令，如果想要使用其外部命令实现，直接指明对应的文件就可以了。例如，要使用外部命令 pwd，可以输入/bin/pwd。

### history 历史记录

一个有用的内建命令是 history 命令。bash shell 会跟踪你用过的命令。你可以唤回这些命令并重新使用。要查看最近用过的命令列表，可以输入不带选项的 history 命令。通常历史记录中会保存最近的 500/1000 条命令。这个数量可是不少的！你可以设置保存在 bash 历史记录中的命令数。要想实现这一点，你需要修改名为 HISTSIZE 的环境变量。

你可以唤回并重用历史列表中最近的命令。这样能够节省时间和击键量。输入!!，然后按回车键就能够唤出刚刚用过的那条命令来使用。或者点击方向键中的向上键，也能使用刚刚用过的那条命令，连续点击还能向上翻阅执行历史命令。

你可以唤回历史列表中任意一条命令。只需输入惊叹号和命令在历史列表中的编号即可。如执行`!20`

命令历史记录被保存在隐藏文件.bash_history 中，它位于用户的主目录中。这里要注意的是，bash 命令的历史记录是先存放在内存中，当 shell 退出时才被写入到历史文件中。

可以在退出 shell 会话之前强制将命令历史记录写入.bash_history 文件。要实现强制写入，需要使用 history 命令的-a 选项。

> 如果你打开了多个终端会话，仍然可以使用 history -a 命令在打开的会话中向.bash_history 文件中添加记录。但是对于其他打开的终端会话，历史记录并不会自动更新。这是因为.bash_history 文件只有在打开首个终端会话时才会被读取。要想强制重新读取.bash_history 文件，更新终端会话的历史记录，可以使用 history -n 命令。

使用 bash shell 命令历史记录能够大大地节省时间。利用内建的 history 命令能够做到的事情远不止这里所描述的。可以通过输入 man history 来查看 history 命令的 bash 手册页面。

### alias 命令别名

alias 命令是另一个 shell 的内建命令。命令别名允许你为常用的命令（及其参数）创建另一个名称，从而将输入量减少到最低。你所使用的 Linux 发行版很有可能已经为你设置好了一些常用命令的别名。要查看当前可用的别名，使用 alias 命令以及选项-p。

```bash
$ alias -p
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto' #表明终端支持彩色模式的列表
```

可以使用 alias 命令创建属于自己的别名。

```bash
alias li='ls -li'
```

在定义好别名之后，你随时都可以在 shell 中使用它，就算在 shell 脚本中也没问题。要注意，因为命令别名属于内部命令，一个别名仅在它所被定义的 shell 进程中才有效。

```bash
$ alias li='ls -li'
$ bash
$ li
bash: li: command not found
```

不过好在有办法能够让别名在不同的子 shell 中都奏效。下一章中就会讲到具体的做法。shell、子 shell、进程和衍生进程都会受到环境变量的影响。下一章，我们会探究环境变量的影响方式以及如何在不同的上下文中使用环境变量。
