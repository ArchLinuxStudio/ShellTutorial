# 控制脚本

当开始构建高级脚本时，你大概会问如何在 Linux 系统上运行和控制它们。在本书中，到目前为止，我们运行脚本的唯一方式就是以实时模式在命令行界面上直接运行。这并不是 Linux 上运行脚本的唯一方式。有不少方法可以用来运行 shell 脚本。另外还有一些选项能够用于控制脚本。这些控制方法包括向脚本发送信号、修改脚本的优先级以及在脚本运行时切换到运行模式。本章将会对逐一介绍这些方法。

## 处理信号

Linux 利用信号与运行在系统中的进程进行通信。之前介绍了不同的 Linux 信号以及 Linux 如何用这些信号来停止、启动、终止进程。可以通过对脚本进行编程，使其在收到特定信号时执行某些命令，从而控制 shell 脚本的操作。

### 重温 Linux 信号

Linux 系统和应用程序可以生成超过 30 个信号。如下列出了在 Linux 编程时会遇到的最常见的 Linux 系统信号。

- 1 SIGHUP 挂起进程
- 2 SIGINT 终止进程
- 3 SIGQUIT 停止进程
- 9 SIGKILL 无条件终止进程
- 15 SIGTERM 尽可能终止进程
- 17 SIGSTOP 无条件停止进程，但不是终止进程
- 18 IGTSTP 停止或暂停进程，但不终止进程
- 19 SIGCONT 继续运行停止的进程

默认情况下，bash shell 会忽略收到的任何 SIGQUIT (3)和 SIGTERM (15)信号（正因为这样，交互式 shell 才不会被意外终止）。但是 bash shell 会处理收到的 SIGHUP (1)和 SIGINT (2)信号。

如果 bash shell 收到了 SIGHUP 信号，比如当你要离开一个交互式 shell，它就会退出。但在退出之前，它会将 SIGHUP 信号传给所有由该 shell 所启动的进程（包括正在运行的 shell 脚本）。

通过 SIGINT 信号，可以中断 shell。Linux 内核会停止为 shell 分配 CPU 处理时间。这种情况发生时，shell 会将 SIGINT 信号传给所有由它所启动的进程，以此告知出现的状况。

你可能也注意到了，shell 会将这些信号传给 shell 脚本程序来处理。而 shell 脚本的默认行为是忽略这些信号。它们可能会不利于脚本的运行。要避免这种情况，你可以脚本中加入识别信号的代码，并执行命令来处理信号。

### 生成信号

bash shell 允许用键盘上的组合键生成两种基本的 Linux 信号。这个特性在需要停止或暂停失控程序时非常方便。

<!-- 补充 ctrl + d -->

1. 中断进程

Ctrl+C 组合键会生成 SIGINT 信号，并将其发送给当前在 shell 中运行的所有进程。可以运行一条需要很长时间才能完成的命令，然后按下 Ctrl+C 组合键来测试它。

```bash
$ sleep 100
^C
$
```

Ctrl+C 组合键会发送 SIGINT 信号，停止 shell 中当前运行的进程。sleep 命令会使得 shell 暂停指定的秒数，命令提示符直到计时器超时才会返回。在超时前按下 Ctrl+C 组合键，就可以提前终止 sleep 命令。

2. 暂停进程

你可以在进程运行期间暂停进程，而无需终止它。尽管有时这可能会比较危险（比如，脚本打开了一个关键的系统文件的文件锁），但通常它可以在不终止进程的情况下使你能够深入脚本内部一窥究竟。

Ctrl+Z 组合键会生成一个 SIGTSTP 信号，停止 shell 中运行的任何进程。停止（stopping）进程跟终止（terminating）进程不同：停止进程会让程序继续保留在内存中，并能从上次停止的位置继续运行。随后你会了解如何重启一个已经停止的进程。

当用 Ctrl+Z 组合键时，shell 会通知你进程已经被停止了。

```bash
$ sleep 100
^Z
[1]+ Stopped sleep 100
$
```

方括号中的数字是 shell 分配的作业号（job number）。shell 将 shell 中运行的每个进程称为作业，并为每个作业分配唯一的作业号。它会给第一个作业分配作业号 1，第二个作业号 2，以此类推。

如果你的 shell 会话中有一个已停止的作业，在退出 shell 时，bash 会提醒你。

```bash
$ sleep 100
^Z
[1]+  Stopped                 sleep 100
$ exit
exit There are stopped jobs.
$
```

可以用 ps 命令来查看已停止的作业。

```bash
$ sleep 100
^Z [1]+  Stopped                 sleep 100
$
$ ps -lF
S UID   PID  PPID  C PRI NI ADDR SZ WCHAN  TTY       TIME CMD
0 S 501  2431  2430  0  80  0 - 27118 wait   pts/0 00:00:00 bash
0 T 501  2456  2431  0  80  0 - 25227 signal pts/0 00:00:00 sleep
0 R 501  2458  2431  0  80  0 - 27034 -      pts/0 00:00:00 ps
$
```

在 S 列中（进程状态），ps 命令将已停止作业的状态为显示为 T。这说明命令已经被停止了。

如果在有已停止作业存在的情况下，你仍旧想退出 shell，只要再输入一遍 exit 命令就行了。shell 会退出，终止已停止作业。或者，既然你已经知道了已停止作业的 PID，就可以用 kill 命令来发送一个 SIGKILL 信号来终止它。

```bash
$ kill -9 2456
$ [1]+  Killed                  sleep 100
$
```

在终止作业时，最开始你不会得到任何回应。但下次如果你做了能够产生 shell 提示符的操作（比如按回车键），你就会看到一条消息，显示作业已经被终止了。每当 shell 产生一个提示符时，它就会显示 shell 中状态发生改变的作业的状态。在你终止一个作业后，下次强制 shell 生成一个提示符时，shell 会显示一条消息，说明作业在运行时被终止了。

### 捕获信号

也可以不忽略信号，在信号出现时捕获它们并执行其他命令。trap 命令允许你来指定 shell 脚本要监看并从 shell 中拦截的 Linux 信号。如果脚本收到了 trap 命令中列出的信号，该信号不再由 shell 处理，而是交由本地处理。

```bash
trap commands signals
```

非常简单！在 trap 命令行上，你只要列出想要 shell 执行的命令，以及一组用空格分开的待捕获的信号。你可以用数值或 Linux 信号名来指定信号。

这里有个简单例子，展示了如何使用 trap 命令来忽略 SIGINT 信号，并控制脚本的行为。

```bash
$ cat test1.sh
#!/bin/bash
# Testing signal trapping
#
trap "echo ' Sorry! I have trapped Ctrl-C'" SIGINT
#
echo This is a test script
#
count=1
while [ $count -le 10 ]
do
    echo "Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
echo "This is the end of the test script" #
```

本例中用到的 trap 命令会在每次检测到 SIGINT 信号时显示一行简单的文本消息。捕获这些信号会阻止用户用 bash shell 组合键 Ctrl+C 来停止程序。

```bash
$ ./test1.sh
This is a test script
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
^C Sorry! I have trapped Ctrl-C
Loop #6
Loop #7
Loop #8
^C Sorry! I have trapped Ctrl-C
Loop #9
Loop #10
This is the end of the test script
$
```

每次使用 Ctrl+C 组合键，脚本都会执行 trap 命令中指定的 echo 语句，而不是处理该信号并允许 shell 停止该脚本。

### 捕获脚本退出

除了在 shell 脚本中捕获信号，你也可以在 shell 脚本退出时进行捕获。这是在 shell 完成任务时执行命令的一种简便方法。

要捕获 shell 脚本的退出，只要在 trap 命令后加上 EXIT 信号就行。

```bash
$ cat test2.sh
#!/bin/bash
# Trapping the script exit
#
trap "echo Goodbye..." EXIT
#
count=1
while [ $count -le 5 ]
do
    echo "Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
$
$ ./test2.sh
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
Goodbye...
$
```

当脚本运行到正常的退出位置时，捕获就被触发了，shell 会执行在 trap 命令行指定的命令。如果提前退出脚本，同样能够捕获到 EXIT。

```bash
$ ./test2.sh
Loop #1
Loop #2
Loop #3
^CGoodbye...
$
```

因为 SIGINT 信号并没有出现在 trap 命令的捕获列表中，当按下 Ctrl+C 组合键发送 SIGINT 信号时，脚本就退出了。但在脚本退出前捕获到了 EXIT，于是 shell 执行了 trap 命令

### 修改或移除捕获

要想在脚本中的不同位置进行不同的捕获处理，只需重新使用带有新选项的 trap 命令。

```bash
$ cat test3.sh
#!/bin/bash
# Modifying a set trap
#
trap "echo ' Sorry... Ctrl-C is trapped.'" SIGINT
#
count=1
while [ $count -le 5 ]
do
    echo "Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
trap "echo ' I modified the trap!'" SIGINT
#
count=1
while [ $count -le 5 ]
do
    echo "Second Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
$
```

修改了信号捕获之后，脚本处理信号的方式就会发生变化。但如果一个信号是在捕获被修改前接收到的，那么脚本仍然会根据最初的 trap 命令进行处理。

```bash
$ ./test3.sh
Loop #1
Loop #2
Loop #3
^C Sorry... Ctrl-C is trapped.
Loop #4
Loop #5
Second Loop #1
Second Loop #2
^C I modified the trap!
Second Loop #3
Second Loop #4
Second Loop #5
$
```

也可以删除已设置好的捕获。只需要在 trap 命令与希望恢复默认行为的信号列表之间加上两个破折号就行了。

```bash
$ cat test3b.sh
#!/bin/bash
# Removing a set trap
#
trap "echo ' Sorry... Ctrl-C is trapped.'" SIGINT
#
count=1
while [ $count -le 5 ]
do
    echo "Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
# Remove the trap
trap -- SIGINT
echo "I just removed the trap"
#
count=1
while [ $count -le 5 ]
do
    echo "Second Loop #$count"
    sleep 1
    count=$[ $count + 1 ]
done
#
$ ./test3b.sh
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
I just removed the trap
Second Loop #1
Second Loop #2
Second Loop #3 ^C
$
```

> 也可以在 trap 命令后使用单破折号来恢复信号的默认行为。单破折号和双破折号都可以正常发挥作用。

移除信号捕获后，脚本按照默认行为来处理 SIGINT 信号，也就是终止脚本运行。但如果信号是在捕获被移除前接收到的，那么脚本会按照原先 trap 命令中的设置进行处理。

```bash
$ ./test3b.sh
Loop #1
Loop #2
Loop #3
^C Sorry... Ctrl-C is trapped.
Loop #4
Loop #5
I just removed the trap
Second Loop #1
Second Loop #2
^C
$
```

在本例中，第一个 Ctrl+C 组合键用于提前终止脚本。因为信号在捕获被移除前已经接收到了，脚本会照旧执行 trap 中指定的命令。捕获随后被移除，再按 Ctrl+C 就能够提前终止脚本了。

## 以后台模式运行脚本

直接在命令行界面运行 shell 脚本有时不怎么方便。一些脚本可能要执行很长一段时间，而你可能不想在命令行界面一直干等着。当脚本在运行时，你没法在终端会话里做别的事情。幸好有个简单的方法可以解决。

在用 ps 命令时，会看到运行在 Linux 系统上的一系列不同进程。显然，所有这些进程都不是运行在你的终端显示器上的。这样的现象被称为在后台（background）运行进程。在后台模式中，进程运行时不会和终端会话上的 STDIN、STDOUT 以及 STDERR 关联。

也可以在 shell 脚本中试试这个特性，允许它们在后台运行而不用占用终端会话。之前简单讲述过后台模式，下面几节将会继续介绍如何在 Linux 系统上以后台模式运行脚本。

### 后台运行脚本

以后台模式运行 shell 脚本非常简单。只要在命令后加个&符就行了。

```bash

$ cat test4.sh
#!/bin/bash
# Test running in the background
#
count=1
while [ $count -le 10 ]
do
    sleep 1
    count=$[ $count + 1 ]
done
#
$
$ ./test4.sh &
[1] 3231
$
```

当&符放到命令后时，它会将命令和 bash shell 分离开来，将命令作为系统中的一个独立的后台进程运行。显示的第一行是：

```bash
[1] 3231
```

方括号中的数字是 shell 分配给后台进程的作业号。下一个数是 Linux 系统分配给进程的进程 ID（PID）。Linux 系统上运行的每个进程都必须有一个唯一的 PID。
一旦系统显示了这些内容，新的命令行界面提示符就出现了。你可以回到 shell，而你所执行的命令正在以后台模式安全的运行。这时，你可以在提示符输入新的命令。

当后台进程结束时，它会在终端上显示出一条消息：

```bash
[1]   Done                    ./test4.sh
```

这表明了作业的作业号以及作业状态（Done），还有用于启动作业的命令。注意，当后台进程运行时，它仍然会使用终端显示器来显示 STDOUT 和 STDERR 消息。

```bash
$ cat test5.sh
#!/bin/bash
# Test running in the background with output
#
echo "Start the test script"
count=1
while [ $count -le 5 ]
do
    echo "Loop #$count"
    sleep 5
    count=$[ $count + 1 ]
done
#
echo "Test script is complete"
#
$
$ ./test5.sh &
[1] 3275
$ Start the test script
Loop #1
Loop #2
Loop #3
Loop #4
Loop #5
Test script is complete
[1]   Done                    ./test5.sh
$
```

你会注意到在上面的例子中，脚本 test5.sh 的输出与 shell 提示符混杂在了一起，这也是为什么 Start the test script 会出现在提示符旁边的原因。  
在显示输出的同时，你仍然可以运行命令。

```bash
$ ./test5.sh &
[1] 3319
$ Start the test script
Loop #1
Loop #2
Loop #3
ls myprog* myprog  myprog.c
$ Loop #4
Loop #5
st script is complete
[1]+  Done                    ./test5.sh
$$
```

当脚本 test5.sh 运行在后台模式时，我们输入了命令 ls myprog\*。脚本输出、输入的命令以及命令输出全都混在了一起。真是让人头昏脑胀！最好是将后台运行的脚本的 STDOUT 和 STDERR 进行重定向，避免这种杂乱的输出。

### 运行多个后台作业

可以在命令行提示符下同时启动多个后台作业。

```bash
$ ./test6.sh &
[1] 3568
$ This is Test Script #1
$ ./test7.sh &
[2] 3570
$ This is Test Script #2
$ ./test8.sh &
[3] 3573
$ And...another Test script
$ ./test9.sh &[4] 3576
$ Then...there was one more test script
$
```

每次启动新作业时，Linux 系统都会为其分配一个新的作业号和 PID。通过 ps 命令，可以看到所有脚本处于运行状态。

```bash
$ ps
PID TTY          TIME CMD
2431 pts/0    00:00:00 bash
3568 pts/0    00:00:00 test6.sh
3570 pts/0    00:00:00 test7.sh
3573 pts/0    00:00:00 test8.sh
3574 pts/0    00:00:00 sleep
3575 pts/0    00:00:00 sleep
3576 pts/0    00:00:00 test9.sh
3577 pts/0    00:00:00 sleep
3578 pts/0    00:00:00 sleep
3579 pts/0    00:00:00 ps
$
```

在终端会话中使用后台进程时一定要小心。注意，在 ps 命令的输出中，每一个后台进程都和终端会话（pts/0）终端联系在一起。如果终端会话退出，那么后台进程也会随之退出。

> 本章之前曾经提到过当你要退出终端会话时，要是存在被停止的进程，会出现警告信息。但如果使用了后台进程，只有某些终端仿真器会在你退出终端会话前提醒你还有后台作业在运行。

如果希望运行在后台模式的脚本在登出控制台后能够继续运行，需要借助于别的手段。下一节中我们会讨论怎么来实现。

## 在非控制台下运行脚本

有时你会想在终端会话中启动 shell 脚本，然后让脚本一直以后台模式运行到结束，即使你退出了终端会话。这可以用 nohup 命令来实现。nohup 命令运行了另外一个命令来阻断所有发送给该进程的 SIGHUP 信号。这会在退出终端会话时阻止进程退出。

nohup 命令的格式如下：

```bash
$ nohup ./test1.sh &
[1] 3856
$ nohup: ignoring input and appending output to 'nohup.out'
$
```

和普通后台进程一样，shell 会给命令分配一个作业号，Linux 系统会为其分配一个 PID 号。区别在于，当你使用 nohup 命令时，如果关闭该会话，脚本会忽略终端会话发过来的 SIGHUP 信号。

由于 nohup 命令会解除终端与进程的关联，进程也就不再同 STDOUT 和 STDERR 联系在一起。为了保存该命令产生的输出，nohup 命令会自动将 STDOUT 和 STDERR 的消息重定向到一个名为 nohup.out 的文件中。

> 如果使用 nohup 运行了另一个命令，该命令的输出会被追加到已有的 nohup.out 文件中。当运行位于同一个目录中的多个命令时一定要当心，因为所有的输出都会被发送到同一个 nohup.out 文件中，结果会让人摸不清头脑。

nohup.out 文件包含了通常会发送到终端显示器上的所有输出。在进程完成运行后，你可以查看 nohup.out 文件中的输出结果。输出会出现在 nohup.out 文件中，就跟进程在命令行下运行时一样。

<!-- 补充nohup自定义输出部分 -->

## 作业控制

在本章的前面部分，你已经知道了如何用组合键停止 shell 中正在运行的作业。在作业停止后，可以选择是终止还是重启。你可以用 kill 命令终止该进程。要重启停止的进程需要向其发送一个 SIGCONT 信号。

启动、停止、终止以及恢复作业的这些功能统称为作业控制。通过作业控制，就能完全控制 shell 环境中所有进程的运行方式了。本节将介绍用于查看和控制在 shell 中运行的作业的命令。

### 查看作业

作业控制中的关键命令是 jobs 命令。jobs 命令允许查看 shell 当前正在处理的作业。

```bash
$ cat test10.sh
#!/bin/bash
# Test job control
#
echo "Script Process ID: $$"
#
count=1
while [ $count -le 10 ]
do
    echo "Loop #$count"
    sleep 10
    count=$[ $count + 1 ]
done
#
echo "End of script..."
#
$

```

脚本用$$变量来显示 Linux 系统分配给该脚本的 PID，然后进入循环，每次迭代都休眠 10 秒。可以从命令行中启动脚本，然后使用 Ctrl+Z 组合键来停止脚本。

```bash
$ ./test10.sh
Script Process ID: 1897
Loop #1
Loop #2
^Z
[1]+  Stopped                 ./test10.sh
$
```

还是使用同样的脚本，利用&将另外一个作业作为后台进程启动。出于简化的目的，脚本的输出被重定向到文件中，避免出现在屏幕上。

```bash
$ ./test10.sh > test10.out &
[2] 1917
$
```

jobs 命令可以查看分配给 shell 的作业。jobs 命令会显示这两个已停止/运行中的作业，以及它们的作业号和作业中使用的命令。

```bash
$ jobs
[1]+  Stopped                 ./test10.sh
[2]-  Running                 ./test10.sh > test10.out &
$
```

要想查看作业的 PID，可以在 jobs 命令中加入-l 选项（小写的 L）。

```bash
$ jobs -l
[1]+  1897 Stopped                 ./test10.sh
[2]-  1917 Running                 ./test10.sh > test10.out &
$
```

jobs 命令使用一些不同的命令行参数如下

- l 列出进程的 PID 以及作业号
- n 只列出上次 shell 发出的通知后改变了状态的作业
- p 只列出作业的 PID
- r 只列出运行中的作业
- s 只列出已停止的作业

你可能注意到了 jobs 命令输出中的加号和减号。带加号的作业会被当做默认作业。在使用作业控制命令时，如果未在命令行指定任何作业号，该作业会被当成作业控制命令的操作对象。当前的默认作业完成处理后，带减号的作业成为下一个默认作业。任何时候都只有一个带加号的作业和一个带减号的作业，不管 shell 中有多少个正在运行的作业。

下面例子说明了队列中的下一个作业在默认作业移除时是如何成为默认作业的。有 3 个独立的进程在后台被启动。jobs 命令显示出了这些进程、进程的 PID 及其状态。注意，默认进程（带有加号的那个）是最后启动的那个进程，也就是 3 号作业。

```bash
$ ./test10.sh > test10a.out &
[1] 1950
$ ./test10.sh > test10b.out &
[2] 1952
$ ./test10.sh > test10c.out &
[3] 1955
$
$ jobs -l
[1]   1950 Running                 ./test10.sh > test10a.out &
[2]-  1952 Running                 ./test10.sh > test10b.out &
[3]+  1955 Running                 ./test10.sh > test10c.out &
$
```

我们调用了 kill 命令向默认进程发送一个 SIGHUP 信号，终止了该作业。在接下来的 jobs 命令输出中，先前带有减号的作业成了现在的默认作业，减号也变成了加号。

```bash
$ kill 1955
$ [3]+  Terminated              ./test10.sh > test10c.out
$
$ jobs -l
[1]-  1950 Running                 ./test10.sh > test10a.out &
[2]+  1952 Running                 ./test10.sh > test10b.out &
$
$ kill 1952
$
[2]+  Terminated              ./test10.sh > test10b.out
$
$ jobs -l
[1]+  1950 Running                 ./test10.sh > test10a.out &
$
```

尽管将一个后台作业更改为默认进程很有趣，但这并不意味着有用。下一节，你将学习在不用 PID 或作业号的情况下，使用命令和默认进程交互。

### 重启停止的作业

在 bash 作业控制中，可以将已停止的作业作为后台进程或前台进程重启。前台进程会接管你当前工作的终端，所以在使用该功能时要小心了。要以后台模式重启一个作业，可用 bg 命令加上作业号。

```bash
$ ./test11.sh
^Z
[1]+  Stopped                 ./test11.sh
$
$ bg
[1]+ ./test11.sh &
$
$ jobs
[1]+  Running                 ./test11.sh &
$
```

因为该作业是默认作业（从加号可以看出），只需要使用 bg 命令就可以将其以后台模式重启。注意，当作业被转入后台模式时，并不会列出其 PID。如果有多个作业，你得在 bg 命令后加上作业号。

```bash
$ ./test11.sh
^Z
[1]+  Stopped                 ./test11.sh
$
$ ./test12.sh
^Z
[2]+  Stopped                 ./test12.sh
$
$ bg 2
[2]+ ./test12.sh &
$
$ jobs
[1]+  Stopped                 ./test11.sh
[2]-  Running                 ./test12.sh &
$
```

命令 bg 2 用于将第二个作业置于后台模式。注意，当使用 jobs 命令时，它列出了作业及其状态，即便是默认作业当前并未处于后台模式。

要以前台模式重启作业，可用带有作业号的 fg 命令。

```bash
$ fg 2
./test12.sh
This is the script's end...
$
```

由于作业是以前台模式运行的，直到该作业完成后，命令行界面的提示符才会出现。

## 调整谦让度

在多任务操作系统中（Linux 就是），内核负责将 CPU 时间分配给系统上运行的每个进程。`调度优先级`（scheduling priority）是内核分配给进程的 CPU 时间（相对于其他进程）。在 Linux 系统中，由 shell 启动的所有进程的调度优先级默认都是相同的。

调度优先级是个整数值，从 -20（最高优先级）到+19（最低优先级）。默认情况下，bash shell 以优先级 0 来启动所有进程。

> 最低值 -20 是最高优先级，而最高值 19 是最低优先级，这太容易记混了。只要记住那句俗语“好人难做”就行了。越是“好”或高的值，获得 CPU 时间的机会越低。

有时你想要改变一个 shell 脚本的优先级。不管是降低它的优先级（这样它就不会从占用其他进程过多的处理能力），还是给予它更高的优先级（这样它就能获得更多的处理时间），你都可以通过 nice 命令做到。

### nice 命令

nice 命令允许你设置命令启动时的调度优先级。要让命令以更低的优先级运行，只要用 nice 的-n 命令行来指定新的优先级级别。

```bash
$ nice -n 10 ./test4.sh > test4.out &
[1] 4973
$
$ ps -p 4973 -o pid,ppid,ni,cmd
PID  PPID  NI CMD
4973  4721  10 /bin/bash ./test4.sh
$
```

注意，必须将 nice 命令和要启动的命令放在同一行中。ps 命令的输出验证了谦让度值（NI 列）已经被调整到了 10。

nice 命令会让脚本以更低的优先级运行。但如果想提高某个命令的优先级，你可能会吃惊。

```bash
$ nice -n -10 ./test4.sh > test4.out &
[1] 4985
$ nice: cannot set niceness: Permission denied
[1]+  Done                    nice -n -10 ./test4.sh > test4.out
$
```

nice 命令阻止普通系统用户来提高命令的优先级。注意，指定的作业的确运行了，但是试图使用 nice 命令提高其优先级的操作却失败了。

nice 命令的-n 选项并不是必须的，只需要在破折号后面跟上优先级就行了。

```bash
$ nice -10 ./test4.sh > test4.out &
[1] 4993
$
$ ps -p 4993 -o pid,ppid,ni,cmd
PID  PPID  NI CMD
4993  4721  10 /bin/bash ./test4.sh
$
```

### renice 命令

有时你想改变系统上已运行命令的优先级。这正是 renice 命令可以做到的。它允许你指定运行进程的 PID 来改变它的优先级。

renice 命令会自动更新当前运行进程的调度优先级。和 nice 命令一样，renice 命令也有一些限制：

```bash
$ ./test11.sh &
[1] 5055
$
$ ps -p 5055 -o pid,ppid,ni,cmd
PID  PPID  NI CMD
5055  4721   0 /bin/bash ./test11.sh
$
$ renice -n 10 -p 5055
5055: old priority 0, new priority 10
$
$ ps -p 5055 -o pid,ppid,ni,cmd
PID  PPID  NI CMD
5055  4721  10 /bin/bash ./test11.sh
$
```

- 只能对属于你的进程执行 renice
- 只能通过 renice 降低进程的优先级
- root 用户可以通过 renice 来任意调整进程的优先级。

如果想完全控制运行进程，必须以 root 账户身份登录或使用 sudo 命令。

## 定时运行作业

当你开始使用脚本时，可能会想要在某个预设时间运行脚本，这通常是在你不在场的时候。Linux 系统提供了多个在预选时间运行脚本的方法：at 命令和 cron 表。每个方法都使用不同的技术来安排脚本的运行时间和频率。接下来会依次介绍这些方法。

### 用 at 命令来计划执行作业

at 命令允许指定 Linux 系统何时运行脚本。at 命令会将作业提交到队列中，指定 shell 何时运行该作业。at 的守护进程 atd 会以后台模式运行，检查作业队列来运行作业。大多数 Linux 发行版会在启动时运行此守护进程(archlinux 需要安装`at`包)。

atd 守护进程会检查系统上的一个特殊目录（通常位于/var/spool/atd）来获取用 at 命令提交的作业。默认情况下，atd 守护进程会每 60 秒检查一下这个目录。有作业时，atd 守护进程会检查作业设置运行的时间。如果时间跟当前时间匹配，atd 守护进程就会运行此作业。

后面几节会介绍如何用 at 命令提交要运行的作业以及如何管理这些作业。

1. at 命令的格式

at 命令的基本格式非常简单：

```bash
at [-f filename] time
```

默认情况下，at 命令会将 STDIN 的输入放到队列中。你可以用-f 参数来指定用于读取命令（脚本文件）的文件名。time 参数指定了 Linux 系统何时运行该作业。如果你指定的时间已经错过，at 命令会在第二天的那个时间运行指定的作业。在如何指定时间这个问题上，你可以非常灵活。at 命令能识别多种不同的时间格式。

- 标准的小时和分钟格式，比如 10:15。
- AM/PM 指示符，比如 10:15 PM。
- 特定可命名时间，比如 now、noon、midnight 或者 teatime（4 PM）。

除了指定运行作业的时间，也可以通过不同的日期格式指定特定的日期。

- 标准日期格式，比如 MMDDYY、MM/DD/YY 或 DD.MM.YY。
- 文本日期，比如 Jul 4 或 Dec 25，加不加年份均可。
- 你也可以指定时间增量。
  - 当前时间+25 min
  - 明天 10:15 PM
  - 10:15+7 天

在你使用 at 命令时，该作业会被提交到作业队列（job queue）。作业队列会保存通过 at 命令提交的待处理的作业。针对不同优先级，存在 26 种不同的作业队列。作业队列通常用小写字母 a~z 和大写字母 A~Z 来指代。

> 在几年前，也可以使用 batch 命令在指定时间执行某个脚本。batch 命令很特别，你可以安排脚本在系统处于低负载时运行。但现在 batch 命令只不过是一个脚本而已（/usr/bin/batch），它会调用 at 命令并将作业提交到 b 队列中。

作业队列的字母排序越高，作业运行的优先级就越低（更高的 nice 值）。默认情况下，at 的作业会被提交到 a 作业队列。如果想以更高优先级运行作业，可以用-q 参数指定不同的队列字母。

2. 获取作业的输出

当作业在 Linux 系统上运行时，显示器并不会关联到该作业。取而代之的是，Linux 系统会将提交该作业的用户的电子邮件地址作为 STDOUT 和 STDERR。任何发到 STDOUT 或 STDERR 的输出都会通过邮件系统发送给该用户。

这里有个在 CentOS 发行版中使用 at 命令安排作业执行的例子。

```bash
$ cat test13.sh
#!/bin/bash
# Test using at command
#
echo "This script ran at $(date +%B%d,%T)"
echo sleep 5
echo "This is the script's end..."
#
$ at -f test13.sh now
job 7 at 2015-07-14 12:38
$
```

at 命令会显示分配给作业的作业号以及为作业安排的运行时间。-f 选项指明使用哪个脚本文件，now 指示 at 命令立刻执行该脚本。

使用 e-mail 作为 at 命令的输出极其不便。at 命令利用 sendmail 应用程序来发送邮件。如果你的系统中没有安装 sendmail，那就无法获得任何输出！因此在使用 at 命令时，最好在脚本中对 STDOUT 和 STDERR 进行重定向，如下例所示。

```bash
$ cat test13b.sh
#!/bin/bash
# Test using at command
#
echo "This script ran at $(date +%B%d,%T)" > test13b.out
echo >> test13b.out
sleep 5
echo "This is the script's end..." >> test13b.out
#
$
$ at -M -f test13b.sh now
job 8 at 2015-07-14 12:48
$
$ cat test13b.out
This script ran at July14,12:48:18
This is the script's end...
$

```

如果不想在 at 命令中使用邮件或重定向，最好加上-M 选项来屏蔽作业产生的输出信息。

3. 列出等待的作业

atq 命令可以查看系统中有哪些作业在等待。

```bash
$ at -M -f test13b.sh teatime
job 17 at 2015-07-14 16:00
$
$ at -M -f test13b.sh tomorrow
job 18 at 2015-07-15 13:03
$
$ at -M -f test13b.sh 13:30
job 19 at 2015-07-14 13:30
$
$ at -M -f test13b.sh now
job 20 at 2015-07-14 13:03
$
$ atq
20      2015-07-14 13:03 = Christine
18      2015-07-15 13:03 a Christine
17      2015-07-14 16:00 a Christine
19      2015-07-14 13:30 a Christine
$
```

作业列表中显示了作业号、系统运行该作业的日期和时间及其所在的作业队列。

4. 删除作业

一旦知道了哪些作业在作业队列中等待，就能用 atrm 命令来删除等待中的作业。

```bash
$ atq
18      2015-07-15 13:03 a Christine
17      2015-07-14 16:00 a Christine
19      2015-07-14 13:30 a Christine
$
$ atrm 18
$
$ atq
17      2015-07-14 16:00 a Christine
19      2015-07-14 13:30 a Christine
$
```

只要指定想要删除的作业号就行了。只能删除你提交的作业，不能删除其他人的

### 安排需要定期执行的脚本

用 at 命令在预设时间安排脚本执行非常好用，但如果你需要脚本在每天的同一时间运行或是每周一次、每月一次呢？用不着再使用 at 不断提交作业了，你可以利用 Linux 系统的另一个功能。

Linux 系统使用 cron 程序来安排要定期执行的作业。cron 程序会在后台运行并检查一个特殊的表（被称作 cron 时间表），以获知已安排执行的作业。

1. cron 时间表

cron 时间表采用一种特别的格式来指定作业何时运行。其格式如下：

```bash
min hour dayofmonth month dayofweek command
```

cron 时间表允许你用特定值、取值范围（比如 1~5）或者是通配符（星号）来指定条目。例如，如果想在每天的 10:15 运行一个命令，可以用 cron 时间表条目：

```bash
15 10 * * * command
```

在 dayofmonth、month 以及 dayofweek 字段中使用了通配符，表明 cron 会在每个月每天的 10:15 执行该命令。要指定在每周一 4:15 PM 运行的命令，可以用下面的条目：

```bash
15 16 * * 1 command
```

可以用三字符的文本值（mon、tue、wed、thu、fri、sat、sun）或数值（0 为周日，6 为周六）来指定 dayofweek 表项。
这里还有另外一个例子：在每个月的第一天中午 12 点执行命令。可以用下面的格式：

```bash
00 12 1 * * command
```

dayofmonth 表项指定月份中的日期值（1~31）。

> 聪明的读者可能会问如何设置一个在每个月的最后一天执行的命令，因为你无法设置 dayofmonth 的值来涵盖所有的月份。这个问题困扰着 Linux 和 Unix 程序员，也激发了不少解决办法。常用的方法是加一条使用 date 命令的 if-then 语句来检查明天的日期是不是 01：`00 12 * * * if [`date +%d -d tomorrow` = 01 ] ; then ; command` 它会在每天中午 12 点来检查是不是当月的最后一天，如果是，cron 将会运行该命令。

命令列表必须指定要运行的命令或脚本的全路径名。你可以像在普通的命令行中那样，添加任何想要的命令行参数和重定向符号。

```bash
15 10 * * * /home/rich/test4.sh > test4out
```

cron 程序会用提交作业的用户账户运行该脚本。因此，你必须有访问该命令和命令中指定的输出文件的权限。

2. 构建 cron 时间表

每个系统用户（包括 root 用户）都可以用自己的 cron 时间表来运行安排好的任务。Linux 提供了 crontab 命令来处理 cron 时间表。要列出已有的 cron 时间表，可以用-l 选项。archlinux 下需要安装 cronie 包

```bash
$ crontab -l
no crontab for rich
$
```

默认情况下，用户的 cron 时间表文件并不存在。要为 cron 时间表添加条目，可以用-e 选项。在添加条目时，crontab 命令会启用一个文本编辑器，使用已有的 cron 时间表作为文件内容（或者是一个空文件，如果时间表不存在的话）。如果想要以指定编辑打开，可以加上 EDITOR 前缀，如`EDITOR=vim crontab -e`

3. 浏览 cron 目录

如果你创建的脚本对精确的执行时间要求不高，用预配置的 cron 脚本目录会更方便。有 4 个基本目录：hourly、daily、monthly 和 weekly。

```bash
$ ls /etc/cron.*ly
/etc/cron.daily:

/etc/cron.hourly:
0anacron

/etc/cron.monthly:

/etc/cron.weekly:
```

因此，例如如果脚本需要每天运行一次，只要将脚本复制到 daily 目录，cron 就会每天执行它。

4. anacron 程序

cron 程序的唯一问题是它假定 Linux 系统是 7×24 小时运行的。除非将 Linux 当成服务器环境来运行，否则此假设未必成立。

如果某个作业在 cron 时间表中安排运行的时间已到，但这时候 Linux 系统处于关机状态，那么这个作业就不会被运行。当系统开机时，cron 程序不会再去运行那些错过的作业。要解决这个问题，许多 Linux 发行版还包含了 anacron 程序。

如果 anacron 知道某个作业错过了执行时间，它会尽快运行该作业。这意味着如果 Linux 系统关机了几天，当它再次开机时，原定在关机期间运行的作业会自动运行。

这个功能常用于进行常规日志维护的脚本。如果系统在脚本应该运行的时间刚好关机，日志文件就不会被整理，可能会变很大。通过 anacron，至少可以保证系统每次启动时整理日志文件

anacron 程序只会处理位于 cron 目录的程序，比如/etc/cron.monthly。它用时间戳来决定作业是否在正确的计划间隔内运行了。每个 cron 目录都有个时间戳文件，该文件位于/var/spool/anacron。

```bash
$ sudo cat /var/spool/anacron/cron.monthly
20150626
$
```

anacron 程序使用自己的时间表（通常位于/etc/anacrontab）来检查作业目录。

```bash
$ sudo cat /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly

```

anacron 时间表的基本格式和 cron 时间表略有不同：

```bash
period delay identifier command
```

period 条目定义了作业多久运行一次，以天为单位。anacron 程序用此条目来检查作业的时间戳文件。delay 条目会指定系统启动后 anacron 程序需要等待多少分钟再开始运行错过的脚本。command 条目包含了 run-parts 程序和一个 cron 脚本目录名。run-parts 程序负责运行目录中传给它的任何脚本。

注意，anacron 不会运行位于/etc/cron.hourly 的脚本。这是因为 anacron 程序不会处理执行时间需求小于一天的脚本。identifier 条目是一种特别的非空字符串，如 cron-weekly。它用于唯一标识日志消息和错误邮件中的作业。

### 使用新 shell 启动脚本

如果每次运行脚本的时候都能够启动一个新的 bash shell（即便只是某个用户启动了一个 bash shell），将会非常的方便。有时候，你希望为 shell 会话设置某些 shell 功能，或者只是为了确保已经设置了某个文件。

回想一下当用户登入 bash shell 时需要运行的启动文件（参见第 6 章）。另外别忘了，不是所有的发行版中都包含这些启动文件。基本上，依照下列顺序所找到的第一个文件会被运行，其余的文件会被忽略

- $HOME/.bash_profile
- $HOME/.bash_login
- $HOME/.profile

因此，应该将需要在登录时运行的脚本放在上面第一个文件中。每次启动一个新 shell 时，bash shell 都会运行.bashrc 文件。可以这样来验证：在主目录下的.bashrc 文件中加入一条简单的 echo 语句，然后启动一个新 shell。

```bash
$ cat .bashrc
# .bashrc
# Source global definitions
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi
# User specific aliases and functions
echo "I'm in a new shell!"
$
$ bash
I'm in a new shell!
$
$ exit
exit
$
```

.bashrc 文件通常也是通过某个 bash 启动文件来运行的。因为.bashrc 文件会运行两次：一次是当你登入 bash shell 时，另一次是当你启动一个 bash shell 时。如果你需要一个脚本在两个时刻都得以运行，可以把这个脚本放进该文件中。
