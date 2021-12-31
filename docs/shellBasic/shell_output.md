# 处理输出

到目前为止，本书中出现的脚本都是通过将数据打印在屏幕上或将数据重定向到文件中来显示信息。之前演示了如何将命令的输出重定向到文件中。本章将会展开这个主题，演示如何将脚本的输出重定向到 Linux 系统的不同位置。

## 理解输入和输出

至此你已经知道了两种显示脚本输出的方法：

- 在显示器屏幕上显示输出
- 将输出重定向到文件中

这两种方法要么将数据输出全部显示，要么什么都不显示。但有时将一部分数据在显示器上显示，另一部分数据保存到文件中也是不错的。对此，了解 Linux 如何处理输入输出能够帮助你就能将脚本输出放到正确位置。

下面几节会介绍如何用标准的 Linux 输入和输出系统来将脚本输出导向特定位置。

### 标准文件描述符

Linux 系统将每个对象当作文件处理。这包括输入和输出进程。Linux 用`文件描述符`（filedescriptor）来标识每个文件对象。文件描述符是一个非负整数，可以唯一标识会话中打开的文件。每个进程一次最多可以有九个文件描述符。出于特殊目的，bash shell 保留了前三个文件描述符（0、1 和 2）。这三个特殊文件描述符会处理脚本的输入和输出。

- 0 缩写:STDIN 含义:标准输入
- 1 缩写:STDOUT 含义:标准输出
- 2 缩写:STDERR 含义:标准错误

shell 用它们将 shell 默认的输入和输出导向到相应的位置。下面几节将会进一步介绍这些标准文件描述符。

1. STDIN

STDIN 文件描述符代表 shell 的标准输入。对终端界面来说，标准输入是键盘。shell 从 STDIN 文件描述符对应的键盘获得输入，在用户输入时处理每个字符。

在使用输入重定向符号（<）时，Linux 会用重定向指定的文件来替换标准输入文件描述符。它会读取文件并提取数据，就如同它是键盘上键入的。

许多 bash 命令能接受 STDIN 的输入，尤其是没有在命令行上指定文件的话。下面是个用 cat 命令处理 STDIN 输入的数据的例子。

```bash
$ cat
this is a test
this is a test
this is a second test.
this is a second test.
```

当在命令行上只输入 cat 命令时，它会从 STDIN 接受输入。输入一行，cat 命令就会显示出一行。但你也可以通过 STDIN 重定向符号强制 cat 命令接受来自另一个非 STDIN 文件的输入。

```bash
$ cat < testfile
This is the first line.
This is the second line.
This is the third line.
$
```

现在 cat 命令会用 testfile 文件中的行作为输入。你可以使用这种技术将数据输入到任何能从 STDIN 接受数据的 shell 命令中。

2. STDOUT

STDOUT 文件描述符代表 shell 的标准输出。在终端界面上，标准输出就是终端显示器。shell 的所有输出（包括 shell 中运行的程序和脚本）会被定向到标准输出中，也就是显示器。

默认情况下，大多数 bash 命令会将输出导向 STDOUT 文件描述符。你可以用输出重定向来改变输出位置。

```bash
$ ls -l > test2
$ cat test2
total 20
-rw-rw-r-- 1 rich rich 53 2014-10-16 11:30 test
-rw-rw-r-- 1 rich rich  0 2014-10-16 11:32 test2
```

通过输出重定向符号，通常会显示到显示器的所有输出会被 shell 重定向到指定的重定向文件。你也可以将数据追加到某个文件。这可以用>>符号来完成

```bash
$ who >> test2
$ cat test2
total 20
-rw-rw-r-- 1 rich rich 53 2014-10-16 11:30 test
-rw-rw-r-- 1 rich rich  0 2014-10-16 11:32 test2
rich     pts/0        2014-10-17 15:34 (192.168.1.2)
$

```

who 命令生成的输出会被追加到 test2 文件中已有数据的后面。

但是，如果你对脚本使用了标准输出重定向，你会遇到一个问题。下面的例子说明了可能会出现什么情况。

```bash
$ ls -al badfile > test3
ls: cannot access badfile: No such file or directory
$ cat test3
$
```

当命令生成错误消息时，shell 并未将错误消息重定向到输出重定向文件。shell 创建了输出重定向文件，但错误消息却显示在了显示器屏幕上。注意，在显示 test3 文件的内容时并没有任何错误。test3 文件创建成功了，只是里面是空的。

shell 对于错误消息的处理是跟普通输出分开的。如果你创建了在后台模式下运行的 shell 脚本，通常你必须依赖发送到日志文件的输出消息。用这种方法的话，如果出现了错误信息，这些信息是不会出现在日志文件中的。你需要换种方法来处理。

3. STDERR

shell 通过特殊的 STDERR 文件描述符来处理错误消息。STDERR 文件描述符代表 shell 的标准错误输出。shell 或 shell 中运行的程序和脚本出错时生成的错误消息都会发送到这个位置。  
默认情况下，STDERR 文件描述符会和 STDOUT 文件描述符指向同样的地方（尽管分配给它们的文件描述符值不同）。也就是说，默认情况下，错误消息也会输出到显示器输出中。  
但从上面的例子可以看出，STDERR 并不会随着 STDOUT 的重定向而发生改变。使用脚本时，你常常会想改变这种行为，尤其是当你希望将错误消息保存到日志文件中的时候。

### 重定向错误

你已经知道如何用重定向符号来重定向 STDOUT 数据。重定向 STDERR 数据也没太大差别，只要在使用重定向符号时定义 STDERR 文件描述符就可以了。有几种办法实现方法。

1. 只重定向错误

你已经知道，STDERR 文件描述符被设成 2。可以选择只重定向错误消息，将该文件描述符值放在重定向符号前。该值必须紧紧地放在重定向符号前，否则不会工作。

```bash
$ ls -al badfile 2> test4
$ cat test4
ls: cannot access badfile: No such file or directory
$
```

现在运行该命令，错误消息不会出现在屏幕上了。该命令生成的任何错误消息都会保存在输出文件中。用这种方法，shell 会只重定向错误消息，而非普通数据。这里是另一个将 STDOUT 和 STDERR 消息混杂在同一输出中的例子。

```bash
$ ls -al test badtest test2 2> test5
-rw-rw-r-- 1 rich rich 158 2014-10-16 11:32 test2
$ cat test5
ls: cannot access test: No such file or directory
ls: cannot access badtest: No such file or directory
$
```

ls 命令的正常 STDOUT 输出仍然会发送到默认的 STDOUT 文件描述符，也就是显示器。由于该命令将文件描述符 2 的输出（STDERR）重定向到了一个输出文件，shell 会将生成的所有错误消息直接发送到指定的重定向文件中。

2. 重定向错误和正常输出

如果想重定向错误和正常输出，必须用两个重定向符号。需要在符号前面放上待重定向数据所对应的文件描述符，然后指向用于保存数据的输出文件。

```bash
$ ls -al test test2 test3 badtest 2> test6 1> test7
$ cat test6
ls: cannot access test: No such file or directory
ls: cannot access badtest: No such file or directory
$ cat test7
-rw-rw-r-- 1 rich rich 158 2014-10-16 11:32 test2
-rw-rw-r-- 1 rich rich   0 2014-10-16 11:33 test3
$
```

shell 利用 1>符号将 ls 命令的正常输出重定向到了 test7 文件，而这些输出本该是进入 STDOUT 的。所有本该输出到 STDERR 的错误消息通过 2>符号被重定向到了 test6 文件。  
可以用这种方法将脚本的正常输出和脚本生成的错误消息分离开来。这样就可以轻松地识别出错误信息，再不用在成千上万行正常输出数据中翻腾了。  
另外，如果愿意，也可以将 STDERR 和 STDOUT 的输出重定向到同一个输出文件。为此 bash shell 提供了特殊的重定向符号&>。

```bash
$ ls -al test test2 test3 badtest &> test7
$ cat test7
ls: cannot access test: No such file or directory
ls: cannot access badtest: No such file or directory
-rw-rw-r-- 1 rich rich 158 2014-10-16 11:32 test2
-rw-rw-r-- 1 rich rich   0 2014-10-16 11:33 test3 $
```

当使用&>符时，命令生成的所有输出都会发送到同一位置，包括数据和错误。你会注意到其中一条错误消息出现的位置和预想中的不一样。badtest 文件（列出的最后一个文件）的这条错误消息出现在输出文件中的第二行。为了避免错误信息散落在输出文件中，相较于标准输出，bash shell 自动赋予了错误消息更高的优先级。这样你能够集中浏览错误信息了。

## 在脚本中重定向输出

可以在脚本中用 STDOUT 和 STDERR 文件描述符以在多个位置生成输出，只要简单地重定向相应的文件描述符就行了。有两种方法来在脚本中重定向输出：

- 临时重定向行输出
- 永久重定向脚本中的所有命令

### 临时重定向

如果有意在脚本中生成错误消息，可以将单独的一行输出重定向到 STDERR。你所需要做的是使用输出重定向符来将输出信息重定向到 STDERR 文件描述符。在重定向到文件描述符时，你必须在文件描述符数字之前加一个&：

```bash
echo "This is an error message" >&2
```

这行会在脚本的 STDERR 文件描述符所指向的位置显示文本，而不是通常的 STDOUT。下面这个例子就利用了这项功能。

```bash
$ cat test8
#!/bin/bash
# testing STDERR messages
echo "This is an error" >&2
echo "This is normal output"
$
```

如果像平常一样运行这个脚本，你可能看不出什么区别。

```bash
$ ./test8
This is an error
This is normal output
$
```

记住，默认情况下，Linux 会将 STDERR 导向 STDOUT。但是，如果你在运行脚本时重定向了 STDERR，脚本中所有导向 STDERR 的文本都会被重定向。

```bash
$ ./test8 2> test9
This is normal output
$ cat test9
This is an error
$
```

太好了！通过 STDOUT 显示的文本显示在了屏幕上，而发送给 STDERR 的 echo 语句的文本则被重定向到了输出文件。这个方法非常适合在脚本中生成错误消息。如果有人用了你的脚本，他们可以像上面的例子中那样轻松地通过 STDERR 文件描述符重定向错误消息。

### 永久重定向

如果脚本中有大量数据需要重定向，那重定向每个 echo 语句就会很烦琐。取而代之，你可以用 exec 命令告诉 shell 在脚本执行期间重定向某个特定文件描述符。

```bash
$ cat test10
#!/bin/bash
# redirecting all output to a file
exec 1>testout
echo "This is a test of redirecting all output"
echo "from a script to another file."
echo "without having to redirect every individual line"
$ ./test10
$ cat testout
This is a test of redirecting all output
from a script to another file.
without having to redirect every individual line
$
```

exec 命令会启动一个新 shell 并将 STDOUT 文件描述符重定向到文件。脚本中发给 STDOUT 的所有输出会被重定向到文件。

可以在脚本执行过程中重定向 STDOUT。

```bash
$ cat test11
#!/bin/bash
# redirecting output to different locations
exec 2>testerror
echo "This is the start of the script"
echo "now redirecting all output to another location"
exec 1>testout
echo "This output should go to the testout file"
echo "but this should go to the testerror file" >&2
$
$ ./test11
This is the start of the script
now redirecting all output to another location
$ cat testout
This output should go to the testout file
$ cat testerror
but this should go to the testerror file
$
```

这个脚本用 exec 命令来将发给 STDERR 的输出重定向到文件 testerror。接下来，脚本用 echo 语句向 STDOUT 显示了几行文本。随后再次使用 exec 命令来将 STDOUT 重定向到 testout 文件。注意，尽管 STDOUT 被重定向了，但你仍然可以将 echo 语句的输出发给 STDERR，在本例中还是重定向到 testerror 文件。

当你只想将脚本的部分输出重定向到其他位置时（如错误日志），这个特性用起来非常方便。不过这样做的话，会碰到一个问题。

一旦重定向了 STDOUT 或 STDERR，就很难再将它们重定向回原来的位置。如果你需要在重定向中来回切换的话，有个办法可以用，后文将会讨论该方法以及如何在脚本中使用。

## 在脚本中重定向输入

你可以使用与脚本中重定向 STDOUT 和 STDERR 相同的方法来将 STDIN 从键盘重定向到其他位置。exec 命令允许你将 STDIN 重定向到 Linux 系统上的文件中：

```bash
exec 0< testfile
```

这个命令会告诉 shell 它应该从文件 testfile 中获得输入，而不是 STDIN。这个重定向只要在脚本需要输入时就会作用。下面是该用法的实例。

```bash
$ cat test12
#!/bin/bash
# redirecting file input
exec 0< testfile
count=1
while read line
do
    echo "Line #$count: $line"
    count=$[ $count + 1 ]
done
$ ./test12
Line #1: This is the first line.
Line #2: This is the second line.
Line #3: This is the third line.
$
```

上一章介绍了如何使用 read 命令读取用户在键盘上输入的数据。将 STDIN 重定向到文件后，当 read 命令试图从 STDIN 读入数据时，它会到文件去取数据，而不是键盘。这是在脚本中从待处理的文件中读取数据的绝妙办法。Linux 系统管理员的一项日常任务就是从日志文件中读取数据并处理。这是完成该任务最简单的办法。

## 创建自己的重定向

在脚本中重定向输入和输出时，并不局限于这 3 个默认的文件描述符。我曾提到过，在 shell 中最多可以有 9 个打开的文件描述符。其他 6 个从 3~8 的文件描述符均可用作输入或输出重定向。你可以将这些文件描述符中的任意一个分配给文件，然后在脚本中使用它们。本节将介绍如何在脚本中使用其他文件描述符。

### 创建输出文件描述符

可以用 exec 命令来给输出分配文件描述符。和标准的文件描述符一样，一旦将另一个文件描述符分配给一个文件，这个重定向就会一直有效，直到你重新分配。这里有个在脚本中使用其他文件描述符的简单例子。

```bash
$ cat test13
#!/bin/bash
# using an alternative file descriptor
exec 3>test13out
echo "This should display on the monitor"
echo "and this should be stored in the file" >&3
echo "Then this should be back on the monitor"
$ ./test13
This should display on the monitor
Then this should be back on the monitor
$ cat test13out
and this should be stored in the file
$
```

这个脚本用 exec 命令将文件描述符 3 重定向到另一个文件。当脚本执行 echo 语句时，输出内容会像预想中那样显示在 STDOUT 上。但你重定向到文件描述符 3 的那行 echo 语句的输出却进入了另一个文件。这样你就可以在显示器上保持正常的输出，而将特定信息重定向到文件中（比如日志文件）。

也可以不用创建新文件，而是使用 exec 命令来将输出追加到现有文件中。

```bash
exec 3>>test13out
```

现在输出会被追加到 test13out 文件，而不是创建一个新文件。

### 重定向文件描述符

现在介绍怎么恢复已重定向的文件描述符。你可以分配另外一个文件描述符给标准文件描述符，反之亦然。这意味着你可以将 STDOUT 的原来位置重定向到另一个文件描述符，然后再利用该文件描述符重定向回 STDOUT。听起来可能有点复杂，但实际上相当直接。这个简单的例子能帮你理清楚。

```bash
$ cat test14
#!/bin/bash
# storing STDOUT, then coming back to it
exec 3>&1
exec 1>test14out
echo "This should store in the output file"
echo "along with this line."
exec 1>&3
echo "Now things should be back to normal"
$
$ ./test14
Now things should be back to normal
$ cat test14out
This should store in the output file
along with this line.
$
```

这个例子有点叫人抓狂，来一段一段地看。首先，脚本将文件描述符 3 重定向到文件描述符 1 的当前位置，也就是 STDOUT。这意味着任何发送给文件描述符 3 的输出都将出现在显示器上。  
第二个 exec 命令将 STDOUT 重定向到文件，shell 现在会将发送给 STDOUT 的输出直接重定向到输出文件中。但是，文件描述符 3 仍然指向 STDOUT 原来的位置，也就是显示器。如果此时将输出数据发送给文件描述符 3，它仍然会出现在显示器上，尽管 STDOUT 已经被重定向了。  
在向 STDOUT（现在指向一个文件）发送一些输出之后，脚本将 STDOUT 重定向到文件描述符 3 的当前位置（现在仍然是显示器）。这意味着现在 STDOUT 又指向了它原来的位置：显示器。  
这个方法可能有点叫人困惑，但这是一种在脚本中临时重定向输出，然后恢复默认输出设置的常用方法。

### 创建输入文件描述符

可以用和重定向输出文件描述符同样的办法重定向输入文件描述符。在重定向到文件之前，先将 STDIN 文件描述符保存到另外一个文件描述符，然后在读取完文件之后再将 STDIN 恢复到它原来的位置。

```bash
$ cat test15
#!/bin/bash
# redirecting input file descriptors
exec 6<&0
exec 0< testfile
count=1
while read line
do
    echo "Line #$count: $line"
    count=$[ $count + 1 ]
done
exec 0<&6
read -p "Are you done now? " answer
case $answer in
    Y|y) echo "Goodbye";;
    N|n) echo "Sorry, this is the end.";;
esac
$ ./test15
Line #1: This is the first line.
Line #2: This is the second line.
Line #3: This is the third line.
Are you done now? y
Goodbye
$
```

在这个例子中，文件描述符 6 用来保存 STDIN 的位置。然后脚本将 STDIN 重定向到一个文件。read 命令的所有输入都来自重定向后的 STDIN（也就是输入文件）。在读取了所有行之后，脚本会将 STDIN 重定向到文件描述符 6，从而将 STDIN 恢复到原先的位置。该脚本用了另外一个 read 命令来测试 STDIN 是否恢复正常了。这次它会等待键盘的输入。

### 创建读写文件描述符

尽管看起来可能会很奇怪，但是你也可以打开单个文件描述符来作为输入和输出。可以用同一个文件描述符对同一个文件进行读写。不过用这种方法时，你要特别小心。由于你是对同一个文件进行数据读写，shell 会维护一个内部指针，指明在文件中的当前位置。任何读或写都会从文件指针上次的位置开始。如果不够小心，它会产生一些令人瞠目的结果。看看下面这个例子。

```bash
$ cat test16
#!/bin/bash
# testing input/output file descriptor
exec 3<> testfile
read line <&3
echo "Read: $line"
echo "This is a test line" >&3
$ cat testfile
This is the first line.
This is the second line.
This is the third line.
$ ./test16
Read: This is the first line.
$ cat testfile
This is the first line.
This is a test line
ine.
This is the third line.
$
```

这个例子用了 exec 命令将文件描述符 3 分配给文件 testfile 以进行文件读写。接下来，它通过分配好的文件描述符，使用 read 命令读取文件中的第一行，然后将这一行显示在 STDOUT 上。最后，它用 echo 语句将一行数据写入由同一个文件描述符打开的文件中。

在运行脚本时，一开始还算正常。输出内容表明脚本读取了 testfile 文件中的第一行。但如果你在脚本运行完毕后，查看 testfile 文件内容的话，你会发现写入文件中的数据覆盖了已有的数据。

当脚本向文件中写入数据时，它会从文件指针所处的位置开始。read 命令读取了第一行数据，所以它使得文件指针指向了第二行数据的第一个字符。在 echo 语句将数据输出到文件时，它会将数据放在文件指针的当前位置，覆盖了该位置的已有数据。

### 关闭文件描述符

如果你创建了新的输入或输出文件描述符，shell 会在脚本退出时自动关闭它们。然而在有些情况下，你需要在脚本结束前手动关闭文件描述符。

要关闭文件描述符，将它重定向到特殊符号&-。脚本中看起来如下：

```bash
exec 3>&-
```

该语句会关闭文件描述符 3，不再在脚本中使用它。这里有个例子来说明当你尝试使用已关闭的文件描述符时会怎样。

```bash
$ cat badtest
#!/bin/bash
# testing closing file descriptors
exec 3> test17file
echo "This is a test line of data" >&3
exec 3>&-
echo "This won't work" >&3
$ ./badtest
./badtest: 3: Bad file descriptor
$
```

一旦关闭了文件描述符，就不能在脚本中向它写入任何数据，否则 shell 会生成错误消息。在关闭文件描述符时还要注意另一件事。如果随后你在脚本中打开了同一个输出文件，shell 会用一个新文件来替换已有文件。这意味着如果你输出数据，它就会覆盖已有文件。考虑下面这个问题的例子。

```bash
$ cat test17
#!/bin/bash
# testing closing file descriptors
exec 3> test17file
echo "This is a test line of data" >&3
exec 3>&-
cat test17file
exec 3> test17file
echo "This'll be bad" >&3
$ ./test17
This is a test line of data
$ cat test17file
This'll be bad
$
```

在向 test17file 文件发送一个数据字符串并关闭该文件描述符之后，脚本用了 cat 命令来显示文件的内容。到目前为止，一切都还好。下一步，脚本重新打开了该输出文件并向它发送了另一个数据字符串。当显示该输出文件的内容时，你所能看到的只有第二个数据字符串。shell 覆盖了原来的输出文件。

## 列出打开的文件描述符

你能用的文件描述符只有 9 个，你可能会觉得这没什么复杂的。但有时要记住哪个文件描述符被重定向到了哪里很难。为了帮助你理清条理，bash shell 提供了 lsof 命令。lsof 命令会列出整个 Linux 系统打开的所有文件描述符。这是个有争议的功能，因为它会向非系统管理员用户提供 Linux 系统的信息。鉴于此，许多 Linux 系统隐藏了该命令，这样用户就不会一不小心就发现了。

有大量的命令行选项和参数可以用来帮助过滤 lsof 的输出。最常用的有-p 和-d，前者允许指定进程 ID（PID），后者允许指定要显示的文件描述符编号。

要想知道进程的当前 PID，可以用特殊环境变量$$（shell 会将它设为当前 PID）。-a 选项用来对其他两个选项的结果执行布尔 AND 运算，这会产生如下输出。

```bash
$ lsof -a -p $$ -d 0,1,2
lsof: WARNING: can't stat() tracefs file system /sys/kernel/debug/tracing
      Output information may be incomplete.
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    162540 testuser    0u   CHR  136,1      0t0    4 /dev/pts/1
bash    162540 testuser    1u   CHR  136,1      0t0    4 /dev/pts/1
bash    162540 testuser    2u   CHR  136,1      0t0    4 /dev/pts/1
```

上例显示了当前进程（bash shell）的默认文件描述符（0、1 和 2）。同时可以注意到，因为是以非 root 用户执行，没有权限查看全部文件描述符信息，lsof 提示了你信息可能不完整。lsof 的默认输出中有 7 列信息，详情如下。

- COMMAND 正在运行的命令名的前 9 个字符
- PID 进程的 PID
- USER 进程属主的登录名
- FD 文件描述符号以及访问类型（r 代表读，w 代表写，u 代表读写）
- TYPE 文件的类型（CHR 代表字符型，BLK 代表块型，DIR 代表目录，REG 代表常规文件）
- DEVICE 设备的设备号（主设备号和从设备号）
- SIZE 如果有的话，表示文件的大小
- NODE 本地文件的节点号
- NAME 文件名

与 STDIN、STDOUT 和 STDERR 关联的文件类型是字符型。因为 STDIN、STDOUT 和 STDERR 文件描述符都指向终端，所以输出文件的名称就是终端的设备名。所有 3 种标准文件都支持读和写（尽管向 STDIN 写数据以及从 STDOUT 读数据看起来有点奇怪）。

现在看一下在打开了多个替代性文件描述符的脚本中使用 lsof 命令的结果。

```bash
$ cat test18
#!/bin/bash
# testing lsof with file descriptors
exec 3> test18file1
exec 6> test18file2
exec 7< testfile
lsof -a -p $$ -d0,1,2,3,6,7
$ ./test18
lsof: WARNING: can't stat() tracefs file system /sys/kernel/debug/tracing
      Output information may be incomplete.
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF     NODE NAME
tt      177709 testuser    0u   CHR  136,1      0t0        4 /dev/pts/1
tt      177709 testuser    1u   CHR  136,1      0t0        4 /dev/pts/1
tt      177709 testuser    2u   CHR  136,1      0t0        4 /dev/pts/1
tt      177709 testuser    3w   REG  259,4        0 33822498 /home/testuser/test18file1
tt      177709 testuser    6w   REG  259,4        0 33822697 /home/testuser/test18file2
tt      177709 testuser    7r   REG  259,4       73 33823059 /home/testuser/testfile
```

该脚本创建了 3 个替代性文件描述符，两个作为输出（3 和 6），一个作为输入（7）。在脚本运行 lsof 命令时，可以在输出中看到新的文件描述符。我们去掉了输出中的第一部分，这样你就能看到文件名的结果了。文件名显示了文件描述符所使用的文件的完整路径名。它将每个文件都显示成 REG 类型的，这说明它们是文件系统中的常规文件。

## 阻止命令输出

有时候，你可能不想显示脚本的输出。这在将脚本作为后台进程运行时很常见，尤其是当运行会生成很多烦琐的小错误的脚本时。要解决这个问题，可以将 STDERR 重定向到一个叫作 null 文件的特殊文件。null 文件跟它的名字很像，文件里什么都没有。shell 输出到 null 文件的任何数据都不会保存，全部都被丢掉了。

在 Linux 系统上 null 文件的标准位置是/dev/null。你重定向到该位置的任何数据都会被丢掉，不会显示。

```bash
$ ls -al > /dev/null
$ cat /dev/null
$
```

这是避免出现错误消息，也无需保存它们的一个常用方法。

```bash
$ ls -al badfile test16 2> /dev/null
-rwxr--r--    1 rich     rich          135 Oct 29 19:57 test16*
$
```

也可以在输入重定向中将/dev/null 作为输入文件。由于/dev/null 文件不含有任何内容，程序员通常用它来快速清除现有文件中的数据，而不用先删除文件再重新创建。

```bash
$ cat testfile
This is the first line.
This is the second line.
This is the third line.
$ cat /dev/null > testfile
$ cat testfile
$
```

文件 testfile 仍然存在系统上，但现在它是空文件。这是清除日志文件的一个常用方法，因为日志文件必须时刻准备等待应用程序操作。

## 创建临时文件

Linux 系统有特殊的目录，专供临时文件使用。Linux 使用/tmp 目录来存放不需要永久保留的文件。大多数 Linux 发行版配置了系统在启动时自动删除/tmp 目录的所有文件。系统上的任何用户账户都有权限在读写/tmp 目录中的文件。这个特性为你提供了一种创建临时文件的简单方法，而且还不用操心清理工作。

有个特殊命令可以用来创建临时文件。mktemp 命令可以在/tmp 目录中创建一个唯一的临时文件。shell 会创建这个文件，但不用默认的 umask 值 。它会将文件的读和写权限分配给文件的属主，并将你设成文件的属主。一旦创建了文件，你就在脚本中有了完整的读写权限，但其他人没法访问它（当然，root 用户除外）。

### 创建本地临时文件

默认情况下，mktemp 会在本地目录中创建一个文件。要用 mktemp 命令在本地目录中创建一个临时文件，你只要指定一个文件名模板就行了。模板可以包含任意文本文件名，在文件名末尾加上 2 个以上 X 就行了。

```bash
$ mktemp testing.XXXXXX
testing.4OnP2E
$ ls -al testing*
-rw-------   1 rich     rich      0 Oct 17 21:30 testing.UfIi13
$
```

mktemp 命令会用 6 个字符码替换这 6 个 X，从而保证文件名在目录中是唯一的。你可以创建多个临时文件，它可以保证每个文件都是唯一的。mktemp 命令的输出正是它所创建的文件的名字。在脚本中使用 mktemp 命令时，可能要将文件名保存到变量中，这样就能在后面的脚本中引用了。

```bash
$ cat test19
#!/bin/bash
# creating and using a temp file
tempfile=$(mktemp test19.XXXXXX)
exec 3>$tempfile
echo "This script writes to temp file $tempfile"
echo "This is the first line" >&3
echo "This is the second line." >&3
echo "This is the last line." >&3
exec 3>&-
echo "Done creating temp file. The contents are:"
cat $tempfile
rm -f $tempfile 2> /dev/null
$ ./test19
This script writes to temp file test19.vCHoya
Done creating temp file.
The contents are:
This is the first line
This is the second line.
This is the last line.
$ ls -al test19*
-rwxr--r--    1 rich     rich          356 Oct 29 22:03 test19*
$
```

这个脚本用 mktemp 命令来创建临时文件并将文件名赋给$tempfile 变量。接着将这个临时文件作为文件描述符 3 的输出重定向文件。在将临时文件名显示在 STDOUT 之后，向临时文件中写入了几行文本，然后关闭了文件描述符。最后，显示出临时文件的内容，并用 rm 命令将其删除。

### 在/tmp 目录创建临时文件

-t 选项会强制 mktemp 命令来在系统的临时目录来创建该文件。在用这个特性时，mktemp 命令会返回用来创建临时文件的全路径，而不是只有文件名。

```bash
$ mktemp -t test.XXXXXX
/tmp/test.xG3374
$ ls -al /tmp/test*
-rw------- 1 rich rich 0 2014-10-29 18:41 /tmp/test.xG3374
$
```

由于 mktemp 命令返回了全路径名，你可以在 Linux 系统上的任何目录下引用该临时文件，不管临时目录在哪里。

```bash
$ cat test20
#!/bin/bash
# creating a temp file in /tmp
tempfile=$(mktemp -t tmp.XXXXXX)
echo "This is a test file." > $tempfile
echo "This is the second line of the test." >> $tempfile
echo "The temp file is located at: $tempfile"
cat $tempfile
rm -f $tempfile
$ ./test20
The temp file is located at: /tmp/tmp.Ma3390
This is a test file.
This is the second line of the test.
$
```

在 mktemp 创建临时文件时，它会将全路径名返回给变量。这样你就能在任何命令中使用该值来引用临时文件了。

### 创建临时目录

-d 选项告诉 mktemp 命令来创建一个临时目录而不是临时文件。这样你就能用该目录进行任何需要的操作了，比如创建其他的临时文件。

```bash
$ cat test21
#!/bin/bash
# using a temporary directory
tempdir=$(mktemp -d dir.XXXXXX)
cd $tempdir
tempfile1=$(mktemp temp.XXXXXX)
tempfile2=$(mktemp temp.XXXXXX)
exec 7> $tempfile1
exec 8> $tempfile2
echo "Sending data to directory $tempdir"
echo "This is a test line of data for $tempfile1" >&7
echo "This is a test line of data for $tempfile2" >&8
$ ./test21
Sending data to directory dir.ouT8S8
$ ls -al
total 72
drwxr-xr-x    3 rich     rich         4096 Oct 17 22:20 ./
drwxr-xr-x    9 rich     rich         4096 Oct 17 09:44 ../
drwx------    2 rich     rich         4096 Oct 17 22:20 dir.ouT8S8/
-rwxr--r--    1 rich     rich          338 Oct 17 22:20 test21*
$ cd dir.ouT8S8
[dir.ouT8S8]$ ls -al
total 16
drwx------    2 rich     rich         4096 Oct 17 22:20 ./
drwxr-xr-x    3 rich     rich         4096 Oct 17 22:20 ../
-rw-------    1 rich     rich           44 Oct 17 22:20 temp.N5F3O6
-rw-------    1 rich     rich           44 Oct 17 22:20 temp.SQslb7
[dir.ouT8S8]$ cat temp.N5F3O6
This is a test line of data for temp.N5F3O6
[dir.ouT8S8]$ cat temp.SQslb7
This is a test line of data for temp.SQslb7
[dir.ouT8S8]$
```

这段脚本在当前目录创建了一个临时目录，然后它用 cd 命令进入该目录，并创建了两个临时文件。之后这两个临时文件被分配给文件描述符，用来存储脚本的输出。

## 记录消息

将输出同时发送到显示器和日志文件，这种做法有时候能够派上用场。你不用将输出重定向两次，只要用特殊的 tee 命令就行。tee 命令相当于管道的一个 T 型接头。它将从 STDIN 过来的数据同时发往两处。一处是 STDOUT，另一处是 tee 命令行所指定的文件名：`tee filename`

由于 tee 会重定向来自 STDIN 的数据，你可以用它配合管道命令来重定向命令输出。

```bash
$ date | tee testfile
Sun Oct 19 18:56:21 EDT 2014
$ cat testfile
Sun Oct 19 18:56:21 EDT 2014
$
```

输出出现在了 STDOUT 中，同时也写入了指定的文件中。注意，默认情况下，tee 命令会在每次使用时覆盖输出文件内容。

```bash
$ who | tee testfile
rich     pts/0        2014-10-17 18:41 (192.168.1.2)
$ cat testfile
rich     pts/0        2014-10-17 18:41 (192.168.1.2)
$
```

如果你想将数据追加到文件中，必须用-a 选项。

利用 tee，既能将数据保存在文件中，也能将数据显示在屏幕上。现在你就可以在为用户显示输出的同时再永久保存一份输出内容了。

```bash
$ cat test22
#!/bin/bash
# using the tee command for logging
tempfile=test22file
echo "This is the start of the test" | tee $tempfile
echo "This is the second line of the test" | tee -a $tempfile
echo "This is the end of the test" | tee -a $tempfile
$ ./test22
This is the start of the test
This is the second line of the test
This is the end of the test
$ cat test22file
This is the start of the test
This is the second line of the test
This is the end of the test
$
```

## 实例

文件重定向常见于脚本需要读入文件和输出文件时。下面的样例脚本两件事都做了。它读取.csv 格式的数据文件，输出 SQL INSERT 语句来将数据插入数据库。shell 脚本使用命令行参数指定待读取的.csv 文件。.csv 格式用于从电子表格中导出数据，所以你可以把数据库数据放入电子表格中，把电子表格保存成.csv 格式，读取文件，然后创建 INSERT 语句将数据插入 MySQL 数据库。

```bash
$cat test23
#!/bin/bash
# read file and create INSERT statements for MySQL
outfile='members.sql'
IFS=','
while read lname fname address city state zip
do
    cat >> $outfile << EOF
        INSERT INTO members (lname,fname,address,city,state,zip) VALUES
        ('$lname', '$fname', '$address', '$city', '$state', '$zip');
EOF
done < ${1}
$
```

这个脚本很短小，这都要感谢有了文件重定向！脚本中出现了三处重定向操作。while 循环使用 read 语句从数据文件中读取文本。注意在 done 语句中出现的重定向符号：

```bash
done < ${1}
```

当运行程序 test23 时，$1 代表第一个命令行参数。它指明了待读取数据的文件。read 语句会使用 IFS 字符解析读入的文本，我们在这里将 IFS 指定为逗号。

脚本中另外两处重定向操作出现在同一条语句中：

```bash
cat >> $outfile << EOF
```

这条语句中包含一个输出追加重定向（双大于号）和一个输入追加重定向（双小于号）。输出重定向将 cat 命令的输出追加到由$outfile 变量指定的文件中。cat 命令的输入不再取自标准输入，而是被重定向到脚本中存储的数据。EOF 符号标记了追加到文件中的数据的起止。

```bash
INSERT INTO members (lname,fname,address,city,state,zip) VALUES
('$lname', '$fname', '$address', '$city', '$state', '$zip');
```

上面的文本生成了一个标准的 SQL INSERT 语句。注意，其中的数据会由变量来替换，变量中内容则是由 read 语句存入的。所以 while 循环一次读取一行数据，将这些值放入 INSERT 语句模板中，然后将结果输出到输出文件中。

在这个例子中，使用以下输入数据文件。

```bash
$ cat members.csv
Blum,Richard,123 Main St.,Chicago,IL,60601
Blum,Barbara,123 Main St.,Chicago,IL,60601
Bresnahan,Christine,456 Oak Ave.,Columbus,OH,43201
Bresnahan,Timothy,456 Oak Ave.,Columbus,OH,43201
$
```

运行脚本时，显示器上不会出现任何输出：

```bash
$ ./test23 members.csv
$
```

但是在 members.sql 输出文件中，你会看到如下输出内容。

```bash
$ cat members.sql
INSERT INTO members (lname,fname,address,city,state,zip) VALUES ('Blum',  'Richard', '123 Main St.', 'Chicago', 'IL', '60601');
INSERT INTO members (lname,fname,address,city,state,zip) VALUES ('Blum',  'Barbara', '123 Main St.', 'Chicago', 'IL', '60601');
INSERT INTO members (lname,fname,address,city,state,zip) VALUES ('Bresnahan',  'Christine', '456 Oak Ave.', 'Columbus', 'OH', '43201');
INSERT INTO members (lname,fname,address,city,state,zip) VALUES ('Bresnahan',  'Timothy', '456 Oak Ave.', 'Columbus', 'OH', '43201');
$
```

结果和我们预想的一样！现在可以将 members.sql 文件导入 MySQL 数据表中了。
