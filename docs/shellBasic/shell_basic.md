# 初识脚本编程

到目前为止我们已经知道了 Linux 系统和命令行的基础知识，是时候开始编程了。本章讨论编写 shell 脚本的基础知识。在开始编写自己的 shell 脚本大作前，你必须了解这些基本概念。

## 使用多个命令

到目前为止，你已经了解了如何使用 shell 的命令行界面提示符来输入命令和查看命令的结果。shell 脚本的关键在于输入多个命令并处理每个命令的结果，甚至需要将一个命令的结果传给另一个命令。shell 可以让你将多个命令串起来，一次执行完成。如果要两个命令一起运行，可以把它们放在同一行中，彼此间用分号隔开。

```bash
$ date ; who
Sun Dec 20 08:59:29 AM CST 2020
testuser   tty1         2020-12-20 08:11 (:0)
testuser   pts/0        2020-12-20 08:11 (:0)
testuser   pts/1        2020-12-20 08:59 (:0)
$
```

恭喜，你刚刚已经写好了一个脚本。这个简单的脚本只用到了两个 bash shell 命令。date 命令先运行，显示了当前日期和时间，后面紧跟着 who 命令的输出，显示当前是谁登录到了系统上。使用这种办法就能将任意多个命令串连在一起使用了，只要不超过最大命令行字符数 255 就行。

这种技术对于小型脚本尚可，但它有一个很大的缺陷：每次运行之前，你都必须在命令提示符下输入整个命令。可以将这些命令组合成一个简单的文本文件，这样就不需要在命令行中手动输入了。在需要运行这些命令时，只用运行这个文本文件就行了。

## 创建 shell 脚本文件

要将 shell 命令放到文本文件中，首先需要用文本编辑器来创建一个文件，然后将命令输入到文件中。在创建 shell 脚本文件时，必须在文件的第一行指定要使用的 shell。其格式为：

```bash
#!/bin/bash
```

在通常的 shell 脚本中，井号（#）用作注释行。shell 并不会处理 shell 脚本中的注释行。然而，shell 脚本文件的第一行是个例外，#后面的惊叹号会告诉 shell 用哪个 shell 来运行脚本（是的，你可以使用 bash shell，同时还可以使用另一个 shell 来运行你的脚本）。

在指定了 shell 之后，就可以在文件的每一行中输入命令，然后加一个回车符。之前提到过，注释可用#添加。例如：

```bash
# This script displays the date and who's logged on
date
who
```

这就是脚本的所有内容了。可以根据需要，使用分号将两个命令放在一行上，但在 shell 脚本中，你可以在独立的行中书写命令。shell 会按根据命令在文件中出现的顺序进行处理。

还有，要注意另有一行也以#开头，并添加了一个注释。shell 不会解释以#开头的行（除了以#!开头的第一行）。留下注释来说明脚本做了什么，这种方法非常好。当两年后回过来再看这个脚本时，你还可以很容易回忆起做过什么。

将这个脚本保存在名为 test1 的文件中，基本就好了。在运行新脚本前，还要做其他一些事。现在运行脚本，结果可能会叫你有点失望。

```bash
$ test1
bash: test1: command not found
```

你要跨过的第一个障碍是让 bash shell 能找到你的脚本文件。如之前所述，shell 会通过 PATH 环境变量来查找命令。快速查看一下 PATH 环境变量就可以弄清问题所在。

```bash
$ echo $PATH
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/bin:/usr/bin :/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/user/bin
```

PATH 环境变量被设置成只在一组目录中查找命令。要让 shell 找到 test1 脚本，只需采取以下两种作法之一：

- 将 shell 脚本文件所处的目录添加到 PATH 环境变量中；
- 在提示符中用绝对或相对文件路径来引用 shell 脚本文件。

> 有些 Linux 发行版将$HOME/bin 目录添加进了 PATH 环境变量。它在每个用户的 HOME 目录下提供了一个存放文件的地方，shell 可以在那里查找要执行的命令。

在这个例子中，我们将用第二种方式将脚本文件的确切位置告诉 shell。记住，为了引用当前目录下的文件，可以在 shell 中使用单点操作符。

```bash
$ ./test1
bash: ./test1: Permission denied
```

现在 shell 找到了脚本文件，但还有一个问题。shell 指明了你还没有执行文件的权限。快速查看一下文件权限就能找到问题所在。

```bash
$ ls -l test1
-rw-rw-r--    1 user     user           73 Sep 24 19:56 test1
```

在创建 test1 文件时，umask 的值决定了新文件的默认权限设置。由于 umask 变量在 ArchLinux 中被设成了 022，所以系统创建的文件的文件属主只有读/写权限。

下一步是通过 chmod 命令赋予文件属主执行文件的权限。

```bash
$ chmod u+x test1
$ ./test1
Sun Dec 20 08:59:29 AM CST 2020
testuser   tty1         2020-12-20 08:11 (:0)
testuser   pts/0        2020-12-20 08:11 (:0)
testuser   pts/1        2020-12-20 08:59 (:0)
```

成功了！

## 显示消息

大多数 shell 命令都会产生自己的输出，这些输出会显示在脚本所运行的控制台显示器上。很多时候，你可能想要添加自己的文本消息来告诉脚本用户脚本正在做什么。可以通过 echo 命令来实现这一点。如果在 echo 命令后面加上了一个字符串，该命令就能显示出这个文本字符串。

```bash
$ echo This is a test
This is a test
```

注意，默认情况下，不需要使用引号将要显示的文本字符串划定出来。但有时在字符串中出现引号的话就比较麻烦了。

```
$ echo Let's see if this'll work
Lets see if thisll work
```

echo 命令可用单引号或双引号来划定文本字符串。如果在字符串中用到了它们，你需要在文本中使用其中一种引号，而用另外一种来将字符串划定起来。

```
$ echo "This is a test to see if you're paying attention"
This is a test to see if you're paying attention
$ echo 'Rich says "scripting is easy".'
Rich says "scripting is easy".
```

所有的引号都可以正常输出了。

如果想把文本字符串和命令输出显示在同一行中，该怎么办呢？可以用 echo 语句的-n 参数。只要将第一个 echo 语句改成这样就行：

```bash
echo -n "The time and date are: "
date
#输出: The time and date are: Mon Feb 21 15:42:23 EST 2014
```

完美！echo 命令是 shell 脚本中与用户交互的重要工具。你会发现在很多地方都能用到它，尤其是需要显示脚本中变量的值的时候。我们下面继续了解这个。

## 使用变量

运行 shell 脚本中的单个命令自然有用，但这有其自身的限制。通常你会需要在 shell 命令使用其他数据来处理信息。这可以通过变量来实现。变量允许你临时性地将信息存储在 shell 脚本中，以便和脚本中的其他命令一起使用。本节将介绍如何在 shell 脚本中使用变量。

### 环境变量

你已经看到过 Linux 的一种变量在实际中的应用。前面介绍了 Linux 系统的环境变量。也可以在脚本中访问这些值。shell 维护着一组环境变量，用来记录特定的系统信息。比如系统的名称、登录到系统上的用户名、用户的系统 ID（也称为 UID）、用户的默认主目录以及 shell 查找程序的搜索路径。可以用 set 命令来显示一份完整的当前环境变量列表。

```bash
$ set
BASH=/bin/bash
[...]
HOME=/home/Samantha
HOSTNAME=localhost.localdomain
HOSTTYPE=i386
IFS=$' \t\n'
LANG=en_US.utf8
LESSOPEN='|/usr/bin/lesspipe.sh %s'
LINES=24
LOGNAME=Samantha
[...]
```

在脚本中，你可以在环境变量名称之前加上美元符（$）来使用这些环境变量。下面的脚本演示了这种用法。

```bash
$ cat test2
#!/bin/bash
# display user information from the system.
echo "User info for userid: $USER" #若为单引号包裹则不会变更$USER的值
echo UID: $UID
echo HOME: $HOME
$
```

$USER、$UID 和$HOME 环境变量用来显示已登录用户的有关信息。脚本输出如下：

```bash
$ chmod u+x test2
$ ./test2
User info for userid: Samantha
UID: 1001
HOME: /home/Samantha
```

> 你可能还见过通过${variable}形式引用的变量。变量名两侧额外的花括号通常用来帮助识别美元符后的变量名。

注意，echo 命令中的环境变量会在脚本运行时替换成当前值。另外，在第一个字符串中可以将$USER 系统变量放置到双引号中，而 shell 依然能够知道我们的意图。但采用这种方法也有一个问题。看看下面这个例子会怎么样。

```bash
$ echo "The cost of the item is $15"
The cost of the item is 5
```

显然这不是我们想要的。只要脚本在引号中出现美元符，它就会以为你在引用一个变量。在这个例子中，脚本会尝试显示变量$1（但并未定义），再显示数字 5。要显示美元符，你必须在它前面放置一个反斜线。

```bash
$ echo "The cost of the item is \$15"
The cost of the item is $15

```

看起来好多了。反斜线允许 shell 脚本将美元符解读为实际的美元符，而不是变量。下一节将会介绍如何在脚本中创建自己的变量。

### 用户变量

除了环境变量，shell 脚本还允许在脚本中定义和使用自己的变量。定义变量允许临时存储数据并在整个脚本中使用，从而使 shell 脚本看起来更像一个真正的计算机程序。用户变量可以是任何由字母、数字或下划线组成的文本字符串，长度不超过 20 个。用户变量区分大小写，所以变量 Var1 和变量 var1 是不同的。这个小规矩经常让脚本编程初学者感到头疼。
使用等号将值赋给用户变量。在变量、等号和值之间不能出现空格（另一个困扰初学者的用法）。这里有一些给用户变量赋值的例子。

```bash
var1=10
var2=-57
var3=testing
var4="still more testing"
```

shell 脚本会自动决定变量值的数据类型。在脚本的整个生命周期里，shell 脚本中定义的变量会一直保持着它们的值，但在 shell 脚本结束时会被删除掉。

与系统变量类似，用户变量可通过美元符引用。变量每次被引用时，都会输出当前赋给它的值。重要的是要记住，引用一个变量值时需要使用美元符，而引用变量来对其进行赋值时则不要使用美元符。通过一个例子你就能明白我的意思。

```bash
$ cat test4
#!/bin/bash
# assigning a variable value to another variable
value1=10
value2=$value1
echo The resulting value is $value2
```

在赋值语句中使用 value1 变量的值时，仍然必须用美元符。这段代码产生如下输出。

```bash
$ chmod u+x test4
$ ./test4
The resulting value is 10

```

要是忘了用美元符，使得 value2 的赋值行变成了这样：

```bash
value2=value1
```

那你会得到如下输出：

```bash
$ ./test4
The resulting value is value1
```

没有美元符，shell 会将变量名解释成普通的文本字符串，通常这并不是你想要的结果。

### 命令替换

shell 脚本中最有用的特性之一就是可以从命令输出中提取信息，并将其赋给变量。把输出赋给变量之后，就可以随意在脚本中使用了。这个特性在处理脚本数据时尤为方便。

有两种方法可以将命令输出赋给变量：

- 反引号字符(\`) testing=\`date\`
- $()格式 testing=$(date)

shell 会运行命令替换符号中的命令，并将其输出赋给变量 testing。注意，赋值等号和命令替换字符之间没有空格。  
下面这个例子很常见，它在脚本中通过命令替换获得当前日期并用它来生成唯一文件名。

```bash
#!/bin/bash
# copy the /usr/bin directory listing to a log file
today=$(date +%y%m%d)
ls /usr/bin -al > log.$today
```

today 变量被赋予格式化后的 date 命令的输出。这是提取日期信息来生成日志文件名常用的一种技术。+%y%m%d 格式告诉 date 命令将日期显示为两位数的年月日的组合。

```bash
$ date +%y%m%d
201220
```

这个脚本将日期值赋给一个变量，之后再将其作为文件名的一部分。文件自身含有/usr/bin 目录列表的重定向输出（将在后续详细讨论）。运行该脚本之后，应该能在目录中看到一个新文件。

```bash
-rw-r--r--    1 user     user          769 Jan 31 10:15 log.140131
```

目录中出现的日志文件采用$today 变量的值作为文件名的一部分。日志文件的内容是/usr/bin 目录内容的列表输出。如果脚本在明天运行，日志文件名会是 log.201221，就这样为新的一天创建一个新文件。

> 命令替换会创建一个子 shell 来运行对应的命令。子 shell（subshell）是由运行该脚本的 shell 所创建出来的一个独立的子 shell（child shell）。正因如此，由该子 shell 所执行命令是无法使用脚本中所创建的变量的。

## 重定向输入和输出

有些时候你想要保存某个命令的输出而不仅仅只是让它显示在显示器上。bash shell 提供了几个操作符，可以将命令的输出重定向到另一个位置（比如文件）。重定向可以用于输入，也可以用于输出，可以将文件重定向到命令输入。

### 输出重定向

最基本的重定向将命令的输出发送到一个文件中。bash shell 用大于号（>）来完成这项功能，之前显示器上出现的命令输出会被保存到指定的输出文件中。

```bash
$ date > test6
$ ls -l test6
-rw-r--r--    1 user     user           29 Feb 10 17:56 test6
$ cat test6
Thu Feb 10 17:56:58 EDT 2020
```

重定向操作符创建了一个文件 test6（通过默认的 umask 设置），并将 date 命令的输出重定向到该文件中。如果输出文件已经存在了，重定向操作符会用新的文件数据覆盖已有文件。

有时，你可能并不想覆盖文件原有内容，而是想要将新命令的输出追加到已有文件内容的后面，比如你正在创建一个记录系统上某个操作的日志文件。在这种情况下，可以用双大于号（>>）来追加数据。

```bash
$ who > test6
$ date >> test6
$ cat test6
user     pts/0    Feb 10 17:55
Thu Feb 10 18:02:14 EDT 2020

```

test6 文件仍然包含早些时候 who 命令的数据，现在又加上了来自 date 命令的输出。

### 输入重定向

输入重定向和输出重定向正好相反。输入重定向将文件的内容重定向到命令，而非将命令的输出重定向到文件。输入重定向符号是小于号（<）。

一个简单的记忆方法就是：在命令行上，命令总是在左侧，而重定向符号“指向”数据流动的方向。小于号说明数据正在从输入文件流向命令。这里有个和 wc 命令一起使用输入重定向的例子。

```bash
$ wc < test6
 2      11      60
$
```

wc 命令可以对对数据中的文本进行计数。默认情况下，它会输出 3 个值：

- 文本的行数
- 文本的词数
- 文本的字节数

通过将文本文件重定向到 wc 命令，你立刻就可以得到文件中的行、词和字节的计数。这个例子说明 test6 文件有 2 行、11 个单词以及 60 字节。

还有另外一种输入重定向的方法，称为`内联输入重定向`（inline input redirection）。这种方法无需使用文件进行重定向，只需要在命令行中指定用于输入重定向的数据就可以了。乍看一眼，这可能有点奇怪，但有些应用会用到这种方式。

内联输入重定向符号是远小于号（<<）。除了这个符号，你必须指定一个文本标记来划分输入数据的开始和结尾。任何字符串都可作为文本标记，但在数据的开始和结尾文本标记必须一致。

在命令行上使用内联输入重定向时，shell 会用 PS2 环境变量中定义的次提示符（即'>'符号）来提示输入数据。下面是它的使用情况。

```bash
$ wc << EOF
> test string 1
> test string 2
> test string 3
> EOF
 3       9      42
$
```

次提示符会持续提示，以获取更多的输入数据，直到你输入了作为文本标记的那个字符串。wc 命令会对内联输入重定向提供的数据进行行、词和字节计数。

## 管道

有时需要将一个命令的输出作为另一个命令的输入。这可以用重定向来实现，只是有些笨拙。

```bash
$ rpm -qa > rpm.list
$ sort < rpm.list
abrt-1.1.14-1.fc14.i686
abrt-addon-ccpp-1.1.14-1.fc14.i686
abrt-addon-kerneloops-1.1.14-1.fc14.i686
abrt-addon-python-1.1.14-1.fc14.i686
...
```

rpm 命令通过 Red Hat 包管理系统（RPM）对系统（比如上例中的 Fedora 系统）上安装的软件包进行管理。配合-qa 选项使用时，它会生成已安装包的列表，但这个列表并不会遵循某种特定的顺序。如果你在查找某个或某组特定的包，想在 rpm 命令的输出中找到就比较困难了。
过标准输出重定向，rpm 命令的输出被重定向到了文件 rpm.list。命令完成后，rpm.list 保存着系统中所有已安装的软件包列表。接下来，输入重定向将 rpm.list 文件的内容发送给 sort 命令，该命令按字母顺序对软件包名称进行排序。

这种方法的确管用，但仍然是一种比较繁琐的信息生成方式。我们用不着将命令输出重定向到文件中，可以将其直接重定向到另一个命令。这个过程叫作`管道连接`（piping）。管道被放在命令之间，将一个命令的输出重定向到另一个命令中：

```bash
command1 | command2
```

不要以为由管道串起的两个命令会依次执行。Linux 系统实际上会同时运行这两个命令，在系统内部将它们连接起来。在第一个命令产生输出的同时，输出会被立即送给第二个命令。数据传输不会用到任何中间文件或缓冲区。

现在，可以利用管道将 rpm 命令的输出送入 sort 命令来产生结果。

```bash
rpm -qa | sort
```

除非你的眼神特别好，否则可能根本来不及看清楚命令的输出。由于管道操作是实时运行的，所以只要 rpm 命令一输出数据，sort 命令就会立即对其进行排序。等到 rpm 命令输出完数据，sort 命令就已经将数据排好序并显示了在显示器上。

可以在一条命令中使用任意多条管道。可以持续地将命令的输出通过管道传给其他命令来细化操作。

在这个例子中，sort 命令的输出会一闪而过，所以可以用一条文本分页命令（例如 less 或 more）来强行将输出按屏显示。

```bash
$ rpm -qa | sort | more
```

这行命令序列会先执行 rpm 命令，将它的输出通过管道传给 sort 命令，然后再将 sort 的输出通过管道传给 more 命令来显示，在显示完一屏信息后停下来。这样你就可以在继续处理前停下来阅读显示器上显示的信息。

如果想要更别致点，也可以搭配使用重定向和管道来将输出保存到文件中。

```bash
$ rpm -qa | sort > rpm.list
```

不出所料，rpm.list 文件中的数据现在已经排好序了。

到目前为止，管道最流行的用法之一是将命令产生的大量输出通过管道传送给 more 命令。这对 ls 命令来说尤为常见，ls -l 命令产生了目录中所有文件的长列表。对包含大量文件的目录来说，这个列表会相当长。通过将输出管道连接到 more 命令，可以强制输出在一屏数据显示后停下来。

## 执行数学运算

另一个对任何编程语言都很重要的特性是操作数字的能力。遗憾的是，对 shell 脚本来说，这个处理过程会比较麻烦。在 shell 脚本中有两种途径来进行数学运算。

### expr 命令

最开始，Bourne shell 提供了一个特别的命令用来处理数学表达式。expr 命令允许在命令行上处理数学表达式，但是特别笨拙。

```bash
$ expr 1 + 5
6
```

expr 命令能够识别少数的数学和字符串操作符。尽管标准操作符在 expr 命令中工作得很好，但在脚本或命令行上使用它们时仍有问题出现。许多 expr 命令操作符在 shell 中另有含义（比如星号）。当它们出现在在 expr 命令中时，会得到一些诡异的结果。

```bash
$ expr 5 * 2
expr: syntax error
```

要解决这个问题，对于那些容易被 shell 错误解释的字符，在它们传入 expr 命令之前，需要使用 shell 的转义字符（反斜线）将其标出来。

```bash
$ expr 5 \* 2
10
```

现在，麻烦才刚刚开始！在 shell 脚本中使用 expr 命令也同样复杂：

```bash
$ cat test6
#!/bin/bash
# An example of using the expr command
var1=10
var2=20
var3=$(expr $var2 / $var1) #命令替换的方式
echo The result is $var3
```

要将一个数学算式的结果赋给一个变量，需要使用命令替换来获取 expr 命令的输出：

```bash
$ chmod u+x test6
$ ./test6
The result is 2
```

幸好 bash shell 有一个针对处理数学运算符的改进，那就是方括号。

### 使用方括号

bash shell 为了保持跟 Bourne shell 的兼容而包含了 expr 命令，但它同样也提供了一种更简单的方法来执行数学表达式。在 bash 中，在将一个数学运算结果赋给某个变量时，可以用美元符和方括号（$[ operation ]）将数学表达式围起来。

```bash
$ var1=$[1 + 5]
$ echo $var1
6
$ var2=$[$var1 * 2]
$ echo $var2
12
$
```

用方括号执行 shell 数学运算比用 expr 命令方便很多。这种技术也适用于 shell 脚本。

```bash
$ cat test7
#!/bin/bash
var1=100
var2=50
var3=45
var4=$[$var1 * ($var2 - $var3)]
echo The final result is $var4  #The final result is 500
```

同样，注意在使用方括号来计算公式时，不用担心 shell 会误解乘号或其他符号。shell 知道它不是通配符，因为它在方括号内。

在 bash shell 脚本中进行算术运算会有一个主要的限制。请看下例：

```bash
$ cat test8
#!/bin/bash
var1=100
var2=45
var3=$[$var1 / $var2]
echo The final result is $var3 #The final result is 2
$
```

bash shell 数学运算符只支持整数运算。若要进行任何实际的数学计算，这是一个巨大的限制。

> z shell（zsh）提供了完整的浮点数算术操作。如果需要在 shell 脚本中进行浮点数运算，可以考虑看看 z shell。

### 浮点解决方案

有几种解决方案能够克服 bash 中数学运算的整数限制。最常见的方案是用内建的 bash 计算器，叫作 bc。

bash 计算器实际上是一种编程语言，它允许在命令行中输入浮点表达式，然后解释并计算该表达式，最后返回结果。bash 计算器能够识别：

- 数字（整数和浮点数）
- 变量（简单变量和数组）
- 注释（以#或 C 语言中的/\* \*/开始的行）
- 表达式
- 编程语句（例如 if-then 语句）
- 函数

可以在 shell 提示符下通过 bc 命令访问 bash 计算器：

```bash
$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.

12 * 5.4
64.8

3.156 * (3 + 5)
25.248
quit

$

```

浮点运算是由内建变量 scale 控制的。必须将这个值设置为你希望在计算结果中保留的小数位数，否则无法得到期望的结果。

```bash
$ bc -q
3.44 / 5
0
scale=4
3.44 / 5
.6880
quit
$

```

scale 变量的默认值是 0。在 scale 值被设置前，bash 计算器的计算结果不包含小数位。在将其值设置成 4 后，bash 计算器显示的结果包含四位小数。-q 命令行选项可以不显示 bash 计算器冗长的欢迎信息。

除了普通数字，bash 计算器还能支持变量。

```bash
$ bc -q
var1=10
var1 * 4
40
var2 = var1 / 5
print var2
2
quit
$
```

变量一旦被定义，你就可以在整个 bash 计算器会话中使用该变量了。print 语句允许你打印变量和数字。

---

现在你可能想问 bash 计算器是如何在 shell 脚本中帮助处理浮点运算的。还记得命令替换吗？是的，可以用命令替换运行 bc 命令，并将输出赋给一个变量。

```bash
$ cat test9
#!/bin/bash
var1=$(echo "scale=4; 3.44 / 5" | bc)
echo The answer is $var1  #The answer is .6880
```

也可以用 shell 脚本中定义好的变量进行运算：

```bash
$ cat test10
#!/bin/bash
var1=100
var2=45
var3=$(echo "scale=4; $var1 / $var2" | bc)
echo The answer for this is $var3 #The answer for this is 2.2222
```

当然，一旦变量被赋值，那个变量也可以用于其他运算。

```bash
$ cat test11
#!/bin/bash
var1=20
var2=3.14159
var3=$(echo "scale=4; $var1 * $var1" | bc)
var4=$(echo "scale=4; $var3 * $var2" | bc)
echo The final result is $var4
$
```

这个方法适用于较短的运算，但有时你会涉及更多的数字。如果需要进行大量运算，在一个命令行中列出多个表达式就会有点麻烦。

有一个方法可以解决这个问题。bc 命令能识别输入重定向，允许你将一个文件重定向到 bc 命令来处理。但这同样会叫人头疼，因为你还得将表达式存放到文件中。

最好的办法是使用内联输入重定向，它允许你直接在命令行中重定向数据，可以将所有 bash 计算器涉及的部分都放到同一个脚本文件的不同行。下面是在脚本中使用这种技术的例子。

```bash
$ cat test12
#!/bin/bash
var1=10.46
var2=43.67
var3=33.2
var4=71
var5=$(bc << EOF
scale = 4
a1 = ($var1 * $var2)
b1 = ($var3 * $var4)
a1 + b1
EOF
)
echo The final answer for this mess is $var5
```

将选项和表达式放在脚本的不同行中可以让处理过程变得更清晰，提高易读性。EOF 字符串标识了重定向给 bc 命令的数据的起止。当然，必须用命令替换符号标识出用来给变量赋值的命令。

你还会注意到，在这个例子中，你可以在 bash 计算器中赋值给变量。有一点很重要：在 bash 计算器中创建的变量只在 bash 计算器中有效，不能在 shell 脚本中使用。

## 退出脚本

迄今为止所有的示例脚本中，我们都是直接停止的。运行完最后一条命令时，脚本就结束了。其实还有另外一种更优雅的方法可以为脚本划上一个句号。

shell 中运行的每个命令都使用`退出状态码`（exit status）告诉 shell 它已经运行完毕。退出状态码是一个 0 ～ 255 的整数值，在命令结束运行时由命令传给 shell。可以捕获这个值并在脚本中使用。

### 查看退出状态码

Linux 提供了一个专门的变量`$?`来保存上个已执行命令的退出状态码。对于需要进行检查的命令，必须在其运行完毕后立刻查看或使用$?变量。它的值会变成由 shell 所执行的最后一条命令的退出状态码。

```bash
$ date
Sun Dec 20 01:35:39 PM CST 2020
$ echo $?
0
$
```

按照惯例，一个成功结束的命令的退出状态码是 0。如果一个命令结束时有错误，退出状态码就是一个正数值。

```bash
$ asdfg
-bash: asdfg: command not found
$ echo $?
127
```

无效命令会返回一个退出状态码 127。Linux 错误退出状态码没有什么标准可循，但有一些可用的参考。

- 0 命令成功结束
- 1 一般性未知错误
- 2 不适合的 shell 命令
- 126 命令不可执行
- 127 没找到命令
- 128 无效的退出参数
- 128+x 与 Linux 信号 x 相关的严重错误
- 130 通过 Ctrl+C 终止的命令
- 225 正常范围之外的退出状态码

退出状态码 126 表明用户没有执行命令的正确权限。

```bash
$ ./myprog.c
-bash: ./myprog.c: Permission denied
$ echo $?
126
$
```

另一个会碰到的常见错误是给某个命令提供了无效参数。

```bash
$ date %t
date: invalid date '%t'
$ echo $?
1
$
```

这会产生一般性的退出状态码 1，表明在命令中发生了未知错误。

### exit 命令

默认情况下，shell 脚本会以脚本中的最后一个命令的退出状态码退出。你可以改变这种默认行为，返回自己的退出状态码。exit 命令允许你在脚本结束时指定一个退出状态码。

```bash
$ cat test13
#!/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 + $var2]
echo The answer is $var3
exit 5
$
```

当查看脚本的退出码时，你会得到作为参数传给 exit 命令的值。

```bash
$ chmod u+x test13
$ ./test13
The answer is 40
$ echo $?
5
$
```

也可以在 exit 命令的参数中使用变量。

```bash
$ cat test14
#!/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 + $var2]
exit $var3
$
```

当你运行这个命令时，它会产生如下退出状态。

```bash
$ chmod u+x test14
$ ./test14
$ echo $?
40
$
```

在以往，exit 退出状态码最大只能是 255，如果超过了 255，最终的结果是指定的数值除以 256 后得到的余数。比如，指定的值是 300（返回值），余数是 44，因此这个余数就成了最后的状态退出码。但是在现在，此限制已经不存在，你可以使用 exit 指令指定更大的数值。

到目前为止，脚本中的命令都是按照有序的方式一个接着一个处理的。在下章中，你将学习如何用一些逻辑流程控制来更改命令的执行次序，也会了解到如何用 if-then 语句来检查某个命令返回的错误状态，以便知道命令是否成功。
