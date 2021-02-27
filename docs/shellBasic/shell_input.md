# 处理 Shell 输入

到目前为止，你已经看到了如何编写脚本，处理数据、变量和 Linux 系统上的文件。有时，你编写的脚本还得能够与使用者进行交互。bash shell 提供了一些不同的方法来从用户处获得数据，包括命令行参数（添加在命令后的数据）、命令行选项（可修改命令行为的单个字母）以及直接从键盘读取输入的能力。本章将会讨论如何在你的 bash shell 脚本运用这些方法来从脚本用户处获得数据。

## 命令行参数

向 shell 脚本传递数据的最基本方法是使用命令行参数。命令行参数允许在运行脚本时向命令行添加数据。

```bash
$ ./addem 10 30
```

本例向脚本 addem 传递了两个命令行参数（10 和 30）。脚本会通过特殊的变量来处理命令行参数。后面几节将会介绍如何在 bash shell 脚本中使用命令行参数。

### 读取参数

bash shell 会将一些称为`位置参数`（positional parameter）的特殊变量分配给输入到命令行中的所有参数。这也包括 shell 所执行的脚本名称。位置参数变量是标准的数字：$0 是程序名，$1 是第一个参数，$2 是第二个参数，依次类推，直到第九个参数$9。

下面是在 shell 脚本中使用单个命令行参数的简单例子。

```bash
$ cat test1.sh
#!/bin/bash
# using one command line parameter
#
factorial=1
for (( number = 1; number <= $1 ; number++ ))
do
    factorial=$[ $factorial * $number ]
done
echo The factorial of $1 is $factorial
$
$ ./test1.sh 5
The factorial of 5 is 120
$
```

可以在 shell 脚本中像使用其他变量一样使用$1 变量。shell 脚本会自动将命令行参数的值分配给变量，不需要你作任何处理。

如果需要输入更多的命令行参数，则每个参数都必须用空格分开。

```bash
$ cat test2.sh
#!/bin/bash
# testing two command line parameters
#
total=$[ $1 * $2 ]
echo The first parameter is $1.
echo The second parameter is $2.
echo The total value is $total.
$
$ ./test2.sh 2 5
The first parameter is 2.
The second parameter is 5.
The total value is 10.
$
```

shell 会将每个参数分配给对应的变量。在前面的例子中，用到的命令行参数都是数值。也可以在命令行上用文本字符串。shell 将输入到命令行的字符串值传给脚本。但碰到含有空格的文本字符串时就会出现问题。记住，每个参数都是用空格分隔的，所以 shell 会将空格当成两个值的分隔符。要在参数值中包含空格，必须要用引号（单引号或双引号均可）。

> 将文本字符串作为参数传递时，引号并非数据的一部分。它们只是表明数据的起止位置。

如果脚本需要的命令行参数不止 9 个，你仍然可以处理，但是需要稍微修改一下变量名。在第 9 个变量之后，你必须在变量数字周围加上花括号，比如${10}。这种方法允许你根据需要向脚本添加任意多的命令行参数。加上花括号会显得比较直观，不过经过测试，目前版本的 bash 不加花括号也可以正常运行，只是不是那么直观罢了。

### 读取脚本名

可以用$0 参数获取 shell 在命令行启动的脚本名。这在编写多功能工具时很方便。

```bash
$ cat test5.sh
#!/bin/bash
# Testing the $0 parameter
#
echo The zero parameter is set to: $0
#
$
$ bash test5.sh
The zero parameter is set to: test5.sh
$
```

但是这里存在一个潜在的问题。如果使用另一种方式来运行 shell 脚本，命令会和脚本名混在一起，出现在$0 参数中。

```bash
$ ./test5.sh
The zero parameter is set to: ./test5.sh
$
```

这还不是唯一的问题。当传给$0 变量的实际字符串不仅仅是脚本名，而是完整的脚本路径时，变量$0 就会使用整个路径。

```bash
$ bash /home/Christine/test5.sh
The zero parameter is set to: /home/Christine/test5.sh
$
```

如果你要编写一个根据脚本名来执行不同功能的脚本，就得做点额外工作。你得把脚本的运行路径给剥离掉。另外，还要删除与脚本名混杂在一起的命令。
幸好有个方便的小命令可以帮到我们。basename 命令会返回不包含路径的脚本名。

```bash
$ cat test5b.sh
#!/bin/bash
# Using basename with the $0 parameter
#
name=$(basename $0)
echo
echo The script name is: $name
#
$
$ bash /home/Christine/test5b.sh
The script name is: test5b.sh
$
$ ./test5b.sh
The script name is: test5b.sh
$

```

现在好多了。可以用这种方法来编写基于脚本名执行不同功能的脚本。这里有个简单的例子。

```bash
$ cat test6.sh
#!/bin/bash
# Testing a Multi-function script
#
name=$(basename $0)
#
if [ $name = "addem" ]
then
    total=$[ $1 + $2 ]
#
elif [ $name = "multem" ]
then
    total=$[ $1 * $2 ]
fi
#
echo echo The calculated value is $total
#
$
$ cp test6.sh addem
$ chmod u+x addem
$
$ ln -s test6.sh multem
$
$ ls -l *em
-rwxrw-r--. 1 Christine Christine 224 Jun 30 23:50 addem
lrwxrwxrwx. 1 Christine Christine   8 Jun 30 23:50 multem -> test6.sh
$
$ ./addem 2 5
The calculated value is 7
$
$ ./multem 2 5
The calculated value is 10
$
```

本例从 test6.sh 脚本中创建了两个不同的文件名：一个通过复制文件创建（addem），另一个通过链接（参见第 3 章）创建（multem）。在两种情况下都会先获得脚本的基本名称，然后根据该值执行相应的功能。

### 测试参数

在 shell 脚本中使用命令行参数时要小心些。如果脚本不加参数运行，可能会出问题。

```bash
$ ./addem 2
./addem: line 8: 2 +  : syntax error: operand expected (error
token is " ")
The calculated value is
$
```

当脚本认为参数变量中会有数据而实际上并没有时，脚本很有可能会产生错误消息。这种写脚本的方法并不可取。在使用参数前一定要检查其中是否存在数据。

```bash
$ cat test7.sh
#!/bin/bash
# testing parameters before use
#
if [ -n "$1" ]
then
    echo Hello $1, glad to meet you.
else
    echo "Sorry, you did not identify yourself. "
fi
$
$ ./test7.sh Rich
Hello Rich, glad to meet you.
$
$ ./test7.sh
Sorry, you did not identify yourself.
$
```

在本例中，使用了-n 测试来检查命令行参数$1 中是否有数据。在下一节中，你会看到还有另一种检查命令行参数的方法。

## 特殊参数变量

在 bash shell 中有些特殊变量，它们会记录命令行参数。本节将会介绍这些变量及其用法。

### 参数统计

如在上一节中看到的，在脚本中使用命令行参数之前应该检查一下命令行参数。对于使用多个命令行参数的脚本来说，这有点麻烦。
你可以统计一下命令行中输入了多少个参数，无需测试每个参数。bash shell 为此提供了一个特殊变量。

特殊变量$#含有脚本运行时携带的命令行参数的个数。可以在脚本中任何地方使用这个特殊变量，就跟普通变量一样。

```bash
$ cat test8.sh
#!/bin/bash
# getting the number of parameters
#
echo There were $# parameters supplied.
$
$ ./test8.sh
There were 0 parameters supplied.
$
$ ./test8.sh 1 2 3 4 5
There were 5 parameters supplied.
```

现在你就能在使用参数前测试参数的总数了。

```bash
$ cat test9.sh
#!/bin/bash
# Testing parameters
#
if [ $# -ne 2 ]
then
    echo
    echo Usage: test9.sh a b
    echo
else
    total=$[ $1 + $2 ]
    echo
    echo The total is $total
    echo
fi
#
$
$ bash test9.sh
Usage: test9.sh a b
$ bash test9.sh 10 15
The total is 25
$
```

if-then 语句用-ne 测试命令行参数数量。如果参数数量不对，会显示一条错误消息告知脚本的正确用法。

这个变量还提供了一个简便方法来获取命令行中最后一个参数，完全不需要知道实际上到底用了多少个参数。不过要实现这一点，得稍微多花点工夫。如果你仔细考虑过，可能会觉得既然$#变量含有参数的总数，那么变量${$#}就代表了最后一个命令行参数变量。试试看会发生什么。

```bash
$ cat badtest1.sh
#!/bin/bash
# testing grabbing last parameter
#
echo The last parameter was ${$#}
$
$ ./badtest1.sh 10
The last parameter was 15354
$
```

怎么了？显然，出了点问题。它表明你不能在花括号内使用美元符。必须将美元符换成感叹号。很奇怪，但的确管用。

```bash
$ cat test10.sh
#!/bin/bash
# Grabbing the last parameter
#
params=$#
echo
echo The last parameter is $params
echo The last parameter is ${!#}
echo
#
$
$ bash test10.sh 1 2 3 4 5
The last parameter is 5
The last parameter is 5
$
$ bash test10.sh
The last parameter is 0
The last parameter is test10.sh
$
```

这个测试将$#变量的值赋给了变量params，然后也按特殊命令行参数变量的格式使用了该变量。两种方法都没问题。重要的是要注意，当命令行上没有任何参数时，$#的值为 0，params 变量的值也一样，但${!#}变量会返回命令行用到的脚本名。

### 抓取所有的数据

有时候需要抓取命令行上提供的所有参数。这时候不需要先用$#变量来判断命令行上有多少参数，然后再进行遍历，你可以使用一组其他的特殊变量来解决这个问题。
$\*和$@变量可以用来轻松访问所有的参数。这两个变量都能够在单个变量中存储所有的命令行参数。
$\*变量会将命令行上提供的所有参数当作一个单词保存。这个单词包含了命令行中出现的每一个参数值。基本上$\*变量会将这些参数视为一个整体，而不是多个个体。
另一方面，$@变量会将命令行上提供的所有参数当作同一字符串中的多个独立的单词。这样你就能够遍历所有的参数值，得到每个参数。这通常通过 for 命令完成。
这两个变量的工作方式不太容易理解。看个例子，你就能理解二者之间的区别了。

```bash
$ cat test11.sh
#!/bin/bash
# testing $* and $@
#
echo
echo "Using the \$* method: $*"
echo
echo "Using the \$@ method: $@"
$
$ ./test11.sh rich barbara katie jessica
Using the $* method: rich barbara katie jessica
Using the $@ method: rich barbara katie jessica
$

```

注意，从表面上看，两个变量产生的是同样的输出，都显示出了所有命令行参数。下面的例子给出了二者的差异。

```bash
$ cat test12.sh
#!/bin/bash
# testing $* and $@
#
echo
count=1
#
for param in "$*"
do
    echo "\$* Parameter #$count = $param"
    count=$[ $count + 1 ]
done
#
echo
count=1
#
for param in "$@"
do
    echo "\$@ Parameter #$count = $param"
    count=$[ $count + 1 ]
done
$
$ ./test12.sh rich barbara katie jessica
$* Parameter #1 = rich barbara katie jessica
$@ Parameter #1 = rich
$@ Parameter #2 = barbara
$@ Parameter #3 = katie
$@ Parameter #4 = jessica
$
```

现在清楚多了。通过使用 for 命令遍历这两个特殊变量，你能看到它们是如何不同地处理命令行参数的。$*变量会将所有参数当成单个参数，而$@变量会单独处理每个参数。这是遍历命令行参数的一个绝妙方法。

## 移动变量

bash shell 工具箱中另一件工具是 shift 命令。bash shell 的 shift 命令能够用来操作命令行参数。跟字面上的意思一样，shift 命令会根据它们的相对位置来移动命令行参数。

在使用 shift 命令时，默认情况下它会将每个参数变量向左移动一个位置。所以，变量$3 的值会移到$2 中，变量$2 的值会移到$1 中，而变量$1 的值则会被删除（注意，变量$0 的值，也就是程序名，不会改变）。

这是遍历命令行参数的另一个好方法，尤其是在你不知道到底有多少参数时。你可以只操作第一个参数，移动参数，然后继续操作第一个参数。这里有个例子来解释它是如何工作的。

```bash
$ cat test13.sh
#!/bin/bash
# demonstrating the shift command
echo
count=1
while [ -n "$1" ]
do
    echo "Parameter #$count = $1"
    count=$[ $count + 1 ]
    shift
done
$
$ ./test13.sh rich barbara katie jessica
Parameter #1 = rich
Parameter #2 = barbara
Parameter #3 = katie
Parameter #4 = jessica
$
```

这个脚本通过测试第一个参数值的长度执行了一个 while 循环。当第一个参数的长度为零时，循环结束。测试完第一个参数后，shift 命令会将所有参数的位置移动一个位置。

> 使用 shift 命令的时候要小心。如果某个参数被移出，它的值就被丢弃了，无法再恢复。

另外，你也可以一次性移动多个位置，只需要给 shift 命令提供一个参数，指明要移动的位置数就行了。如`shift 2`一次性移动两个位置。

## 处理选项

如果你认真读过本书前面的所有内容，应该就见过了一些同时提供了参数和选项的 bash 命令。选项是跟在单破折线后面的单个字母，它能改变命令的行为。本节将会介绍 3 种在脚本中处理选项的方法。

### 查找选项

表面上看，命令行选项也没什么特殊的。在命令行上，它们紧跟在脚本名之后，就跟命令行参数一样。实际上，如果愿意，你可以像处理命令行参数一样处理命令行选项。

1. 处理简单选项

在前面的 test13.sh 脚本中，你看到了如何使用 shift 命令来依次处理脚本程序携带的命令行参数。你也可以用同样的方法来处理命令行选项。在提取每个单独参数时，用 case 语句来判断某个参数是否为选项。

```bash
$ cat test15.sh
#!/bin/bash
# extracting command line options as parameters
#
echo
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option" ;;
        -b) echo "Found the -b option" ;;
        -c) echo "Found the -c option" ;;
        *) echo "$1 is not an option" ;;
    esac
    shift
done
$
$ ./test15.sh -a -b -c -d
Found the -a option
Found the -b option
Found the -c option
-d is not an option
$
```

case 语句会检查每个参数是不是有效选项。如果是的话，就运行对应 case 语句中的命令。不管选项按什么顺序出现在命令行上，这种方法都适用。case 语句在命令行参数中找到一个选项，就处理一个选项。如果命令行上还提供了其他参数，你可以在 case 语句的通用情况处理部分中处理。

2. 分离参数和选项

你会经常遇到在 shell 脚本中同时使用选项和参数的情况。Linux 中处理这个问题的标准方式是用特殊字符来将二者分开，该字符会告诉脚本何时选项结束以及普通参数何时开始。
对 Linux 来说，这个特殊字符是双破折线（--）。shell 会用双破折线来表明选项列表结束。在双破折线之后，脚本就可以放心地将剩下的命令行参数当作参数，而不是选项来处理了。

要检查双破折线，只要在 case 语句中加一项就行了。

```bash
$ cat test16.sh
#!/bin/bash
# extracting options and parameters
#
echo
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option" ;;
        -b) echo "Found the -b option" ;;
        -c) echo "Found the -c option" ;;
        --) shift
            break ;;
        *) echo "$1 is not an option" ;;
    esac
    shift
done

count=1
for param in $@
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done

```

在遇到双破折线时，脚本用 break 命令来跳出 while 循环。由于过早地跳出了循环，我们需要再加一条 shift 命令来将双破折线移出参数变量。
对于第一个测试，试试用一组普通的选项和参数来运行这个脚本。

```bash
$
$ ./test16.sh -c -a -b test1 test2 test3
Found the -c option
Found the -a option
Found the -b option
test1 is not an option
test2 is not an option
test3 is not an option
$
```

结果说明在处理时脚本认为所有的命令行参数都是选项。接下来，进行同样的测试，只是这次会用双破折线来将命令行上的选项和参数划分开来。

```bash
$ ./test16.sh -c -a -b -- test1 test2 test3
Found the -c option
Found the -a option
Found the -b option
Parameter #1: test1
Parameter #2: test2
Parameter #3: test3
$
```

当脚本遇到双破折线时，它会停止处理选项，并将剩下的参数都当作命令行参数。

3. 处理带值的选项

有些选项会带上一个额外的参数值。在这种情况下，命令行看起来像下面这样。

```bash
$ ./testing.sh -a test1 -b -c -d test2

```

当命令行选项要求额外的参数时，脚本必须能检测到并正确处理。下面是如何处理的例子。

```bash
$ cat test17.sh
#!/bin/bash
# extracting command line options and values
echo
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option";;
        -b) param="$2"
            echo "Found the -b option, with parameter value $param"
            shift ;;
        -c) echo "Found the -c option";;
        --) shift
            break ;;
        *) echo "$1 is not an option";;
    esac
    shift
done
#
count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done
$
$ ./test17.sh -a -b test1 -d
Found the -a option
Found the -b option, with parameter value test1
-d is not an option
$
```

在这个例子中，case 语句定义了三个它要处理的选项。-b 选项还需要一个额外的参数值。由于要处理的参数是$1，额外的参数值就应该位于$2（因为所有的参数在处理完之后都会被移出）。只要将参数值从$2 变量中提取出来就可以了。当然，因为这个选项占用了两个参数位，所以你还需要使用 shift 命令多移动一个位置。

只用这些基本的特性，整个过程就能正常工作，不管按什么顺序放置选项（但要记住包含每个选项相应的选项参数）。现在 shell 脚本中已经有了处理命令行选项的基本能力，但还有一些限制。比如，如果你想将多个选项放进一个参数中时，它就不能工作了。

```bash
$ ./test17.sh -ac
-ac is not an option
$
```

在 Linux 中，合并选项是一个很常见的用法，而且如果脚本想要对用户更友好一些，也要给用户提供这种特性。幸好，有另外一种处理选项的方法能够帮忙。

### 使用 getopt 命令

getopt 命令是一个在处理命令行选项和参数时非常方便的工具。它能够识别命令行参数，从而在脚本中解析它们时更方便。

1. 命令的格式

getopt 命令可以接受一系列任意形式的命令行选项和参数，并自动将它们转换成适当的格式。它的命令格式如下：

```bash
getopt optstring parameters
```

optstring 是这个过程的关键所在。它定义了命令行有效的选项字母，还定义了哪些选项字母需要参数值。

首先，在 optstring 中列出你要在脚本中用到的每个命令行选项字母。然后，在每个需要参数值的选项字母后加一个冒号。getopt 命令会基于你定义的 optstring 解析提供的参数。

> getopt 命令有一个更高级的版本叫作 getopts（注意这是复数形式）。getopts 命令会在本章随后部分讲到。因为这两个命令的拼写几乎一模一样，所以很容易搞混。一定要小心

下面是个 getopt 如何工作的简单例子。

```bash
$ getopt ab:cd -a -b test1 -cd test2 test3
-a -b test1 -c -d -- test2 test3
$
```

optstring 定义了四个有效选项字母：a、b、c 和 d。冒号（:）被放在了字母 b 后面，因为 b 选项需要一个参数值。当 getopt 命令运行时，它会检查提供的参数列表（-a -b test1 -cd test2 test3），并基于提供的 optstring 进行解析。注意，它会自动将-cd 选项分成两个单独的选项，并插入双破折线来分隔行中的额外参数。

如果指定了一个不在 optstring 中的选项，默认情况下，getopt 命令会产生一条错误消息。

```bash
$ getopt ab:cd -a -b test1 -cde test2 test3
getopt: invalid option -- e
-a -b test1 -c -d -- test2 test3
$
```

如果想忽略这条错误消息，可以在命令后加-q 选项。

```bash
$ getopt -q ab:cd -a -b test1 -cde test2 test3
-a -b test1 -c -d -- test2 test3
$
```

注意，getopt 命令选项必须出现在 optstring 之前。现在应该可以在脚本中使用此命令处理命令行选项了

2. 在脚本中使用 getopt

可以在脚本中使用 getopt 来格式化脚本所携带的任何命令行选项或参数，但用起来略微复杂。方法是用 getopt 命令生成的格式化后的版本来替换已有的命令行选项和参数。用 set 命令能够做到。在环境变量的章节中，你就已经见过 set 命令了。set 命令能够处理 shell 中的各种变量。

set 命令的选项之一是双破折线（--），它会将命令行参数替换成 set 命令的命令行值。然后，该方法会将原始脚本的命令行参数传给 getopt 命令，之后再将 getopt 命令的输出传给 set 命令，用 getopt 格式化后的命令行参数来替换原始的命令行参数，看起来如下所示。

```bash
set -- $(getopt -q ab:cd "$@")
```

现在原始的命令行参数变量的值会被 getopt 命令的输出替换，而 getopt 已经为我们格式化好了命令行参数。利用该方法，现在就可以写出能帮我们处理命令行参数的脚本。

```bash
$ cat test18.sh
#!/bin/bash
# Extract command line options & values with getopt
#
set -- $(getopt -q ab:cd "$@")
#
echo
while [ -n "$1" ]
do
    case "$1" in
    -a) echo "Found the -a option" ;;
    -b) param="$2"
        echo "Found the -b option, with parameter value $param"
        shift ;;
    -c) echo "Found the -c option" ;;
    --) shift
        break ;;
    *) echo "$1 is not an option";;
    esac
    shift
done
#
count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$[ $count + 1 ]
done
#
$
```

你会注意到它跟脚本 test17.sh 一样，唯一不同的是加入了 getopt 命令来帮助格式化命令行参数。现在如果运行带有复杂选项的脚本，就可以看出效果更好了。

```bash
$ ./test18.sh -ac
Found the -a option
Found the -c option
$
```

当然，之前的功能照样没有问题。

```bash
$ ./test18.sh -a -b test1 -cd test2 test3 test4
Found the -a option
Found the -b option, with parameter value 'test1'
Found the -c option
-d is not an option
Parameter #1: 'test2'
Parameter #2: 'test3'
Parameter #3: 'test4'
$
```

现在看起来相当不错了。但是，在 getopt 命令中仍然隐藏着一个小问题。看看这个例子。

```bash
$ ./test18.sh -a -b test1 -cd "test2 test3" test4
Found the -a option
Found the -b option, with parameter value 'test1'
Found the -c option
-d is not an option
Parameter #1: 'test2
Parameter #2: test3'
Parameter #3: 'test4'
$
```

getopt 命令并不擅长处理带空格和引号的参数值。它会将空格当作参数分隔符，而不是根据双引号将二者当作一个参数。幸而还有另外一个办法能解决这个问题。

### 使用更高级的 getopts

getopts 命令（注意是复数）内建于 bash shell。它跟近亲 getopt 看起来很像，但多了一些扩展功能。

与 getopt 不同，前者将命令行上选项和参数处理后只生成一个输出，而 getopts 命令能够和已有的 shell 参数变量配合默契。

每次调用它时，它一次只处理命令行上检测到的一个参数。处理完所有的参数后，它会退出并返回一个大于 0 的退出状态码。这让它非常适合用于解析命令行所有参数的循环中。
getopts 命令的格式如下：

```bash
getopts optstring variable
```

optstring 值类似于 getopt 命令中的那个。有效的选项字母都会列在 optstring 中，如果选项字母要求有个参数值，就加一个冒号。要去掉错误消息的话，可以在 optstring 之前加一个冒号。getopts 命令将当前参数保存在命令行中定义的 variable 中。

getopts 命令会用到两个环境变量。如果选项需要跟一个参数值，OPTARG 环境变量就会保存这个值。OPTIND 环境变量保存了参数列表中 getopts 正在处理的参数位置。这样你就能在处理完选项之后继续处理其他命令行参数了。让我们看个使用 getopts 命令的简单例子。

```bash
$ cat test19.sh
#!/bin/bash
# simple demonstration of the getopts command
#
echo
while getopts :ab:c opt
do
    case "$opt" in
        a) echo "Found the -a option" ;;
        b) echo "Found the -b option, with value $OPTARG";;
        c) echo "Found the -c option" ;;
        *) echo "Unknown option: $opt";;
    esac
done
$
$ ./test19.sh -ab test1 -c
Found the -a option
Found the -b option, with value test1
Found the -c option
$
```

while 语句定义了 getopts 命令，指明了要查找哪些命令行选项，以及每次迭代中存储它们的变量名（opt）。

你会注意到在本例中 case 语句的用法有些不同。getopts 命令解析命令行选项时会移除开头的单破折线，所以在 case 定义中不用单破折线。

getopts 命令有几个好用的功能。对新手来说，可以在参数值中包含空格。

```bash
$ ./test19.sh -b "test1 test2" -a
Found the -b option, with value test1 test2
Found the -a option
$
```

另一个好用的功能是将选项字母和参数值放在一起使用，而不用加空格。

```bash
$ ./test19.sh -abtest1
Found the -a option
Found the -b option, with value test1
$
```

getopts 命令能够从-b 选项中正确解析出 test1 值。除此之外，getopts 还能够将命令行上找到的所有未定义的选项统一输出成问号。

```bash
$ ./test19.sh -d
Unknown option: ?
$
$ ./test19.sh -acde
Found the -a option
Found the -c option
Unknown option: ?
Unknown option: ?
$
```

optstring 中未定义的选项字母会以问号形式发送给代码。

getopts 命令知道何时停止处理选项，并将参数留给你处理。在 getopts 处理每个选项时，它会将 OPTIND 环境变量值增一。在 getopts 完成处理时，你可以使用 shift 命令和 OPTIND 值来移动参数。

```bash
$ cat test20.sh
#!/bin/bash
# Processing options & parameters with getopts
#
echo
while getopts :ab:cd opt
do
    case "$opt" in
        a) echo "Found the -a option"  ;;
        b) echo "Found the -b option, with value $OPTARG" ;;
        c) echo "Found the -c option"  ;;
        d) echo "Found the -d option"  ;;
        *) echo "Unknown option: $opt" ;;
    esac
done
#
shift $[ $OPTIND - 1 ]
#
echo
count=1
for param in "$@"
do
    echo "Parameter $count: $param"
    count=$[ $count + 1 ]
done
#
$
$ ./test20.sh -a -b test1 -d test2 test3 test4
Found the -a option
Found the -b option, with value test1
Found the -d option
Parameter 1: test2
Parameter 2: test3
Parameter 3: test4
$
```

现在你就拥有了一个能在所有 shell 脚本中使用的全功能命令行选项和参数处理工具。

## 将选项标准化

在创建 shell 脚本时，显然可以控制具体怎么做。你完全可以决定用哪些字母选项以及它们的用法。但有些字母选项在 Linux 世界里已经拥有了某种程度的标准含义。如果你能在 shell 脚本中支持这些选项，脚本看起来能更友好一些。下面展示了 Linux 中用到的一些命令行选项的常用含义。

- -a 显示所有对象
- -c 生成一个计数
- -d 指定一个目录
- -e 扩展一个对象
- -f 指定读入数据的文件
- -h 显示命令的帮助信息
- -i 忽略文本大小写
- -l 产生输出的长格式版本
- -n 使用非交互模式（批处理）
- -o 将所有输出重定向到的指定的输出文件
- -q 以安静模式运行
- -r 递归地处理目录和文件
- -s 以安静模式运行
- -v 生成详细输出
- -x 排除某个对象
- -y 对所有问题回答 yes

通过学习本书时遇到的各种 bash 命令，你大概已经知道这些选项中大部分的含义了。如果你的选项也采用同样的含义，这样用户在使用你的脚本时就不用去查手册了。

## 获得用户输入

尽管命令行选项和参数是从脚本用户处获得输入的一种重要方式，但有时脚本的交互性还需要更强一些。比如你想要在脚本运行时问个问题，并等待运行脚本的人来回答。bash shell 为此提供了 read 命令。

### 基本的读取

read 命令从标准输入（键盘）或另一个文件描述符中接受输入。在收到输入后，read 命令会将数据放进一个变量。下面是 read 命令的最简单用法。

```bash
$ cat test21.sh
#!/bin/bash
# testing the read command
#
echo -n "Enter your name: "
read name
echo "Hello $name, welcome to my program. "
#
$
$ ./test21.sh
Enter your name: Rich Blum
Hello Rich Blum, welcome to my program.
$
```

相当简单。注意，生成提示的 echo 命令使用了-n 选项。该选项不会在字符串末尾输出换行符，允许脚本用户紧跟其后输入数据，而不是下一行。这让脚本看起来更像表单。

实际上，read 命令包含了-p 选项，允许你直接在 read 命令行指定提示符。

```bash
$ cat test22.sh
#!/bin/bash
# testing the read -p option
#
read -p "Please enter your age: " age
days=$[ $age * 365 ]
echo
"That makes you over $days days old! "
#
$
$ ./test22.sh
Please enter your age: 10
That makes you over 3650 days old!
$
```

你会注意到，在第一个例子中当有名字输入时，read 命令会将姓和名保存在同一个变量中。read 命令会将提示符后输入的所有数据分配给单个变量，要么你就指定多个变量。输入的每个数据值都会分配给变量列表中的下一个变量。如果变量数量不够，剩下的数据就全部分配给最后一个变量。

```bash
$ cat test23.sh
#!/bin/bash
# entering multiple variables
#
read -p "Enter your name: " first last
echo "Checking data for $last, $first..."
$
$ ./test23.sh
Enter your name: Rich Blum
Checking data for Blum, Rich...
$
```

也可以在 read 命令行中不指定变量。如果是这样，read 命令会将它收到的任何数据都放进特殊环境变量 REPLY 中。

```bash
$ cat test24.sh
#!/bin/bash
# Testing the REPLY Environment variable
#
read -p "Enter your name: "
echo
echo Hello $REPLY, welcome to my program.
#
$
$ ./test24.sh
Enter your name: Christine
Hello Christine, welcome to my program.
$
```

REPLY 环境变量会保存输入的所有数据，可以在 shell 脚本中像其他变量一样使用。

### 超时

使用 read 命令时要当心。脚本很可能会一直苦等着脚本用户的输入。如果不管是否有数据输入，脚本都必须继续执行，你可以用-t 选项来指定一个计时器。-t 选项指定了 read 命令等待输入的秒数。当计时器过期后，read 命令会返回一个非零退出状态码。

```bash
$ cat test25.sh
#!/bin/bash
# timing the data entry
#
if read -t 5 -p "Please enter your name: " name
then
    echo "Hello $name, welcome to my script"
else
    echo
    echo "Sorry, too slow! "
fi
$
$ ./test25.sh
Please enter your name: Rich
Hello Rich, welcome to my script
$
```

如果计时器过期，read 命令会以非零退出状态码退出，可以使用如 if-then 语句或 while 循环这种标准的结构化语句来理清所发生的具体情况。在本例中，计时器过期时，if 语句不成立，shell 会执行 else 部分的命令。

也可以不对输入过程计时，而是让 read 命令来统计输入的字符数。当输入的字符达到预设的字符数时，就自动退出，将输入的数据赋给变量。

```bash
$ cat test26.sh
#!/bin/bash
# getting just one character of input
#
read -n1 -p "Do you want to continue [Y/N]? " answer
case $answer in
Y | y)  echo
        echo "fine, continue on...";;
N | n)  echo
        echo OK, goodbye
        exit;;
esac
echo "This is the end of the script"
$
$ ./test26.sh
Do you want to continue [Y/N]? Y
fine, continue on...
This is the end of the script
$
$ ./test26.sh
Do you want to continue [Y/N]? n
OK, goodbye
$
```

本例中将-n 选项和值 1 一起使用，告诉 read 命令在接受单个字符后退出。只要按下单个字符回答后，read 命令就会接受输入并将它传给变量，无需按回车键。

### 隐藏方式读取

有时你需要从脚本用户处得到输入，但又不在屏幕上显示输入信息。其中典型的例子就是输入的密码，但除此之外还有很多其他需要隐藏的数据类型。

-s 选项可以避免在 read 命令中输入的数据出现在显示器上（实际上，数据会被显示，只是 read 命令会将文本颜色设成跟背景色一样）。这里有个在脚本中使用-s 选项的例子。

```bash
$ cat test27.sh
#!/bin/bash
# hiding input data from the monitor
#
read -s -p "Enter your password: " pass
echo
echo "Is your password really $pass? "
$
$ ./test27.sh
Enter your password:
Is your password really T3st1ng?
$
```

输入提示符输入的数据不会出现在屏幕上，但会赋给变量，以便在脚本中使用。

### 从文件中读取

最后，也可以用 read 命令来读取 Linux 系统上文件里保存的数据。每次调用 read 命令，它都会从文件中读取一行文本。当文件中再没有内容时，read 命令会退出并返回非零退出状态码。

其中最难的部分是将文件中的数据传给 read 命令。最常见的方法是对文件使用 cat 命令，将结果通过管道直接传给含有 read 命令的 while 命令。下面的例子说明怎么处理。

```bash
$ cat test28.sh
#!/bin/bash
# reading data from a file
#
count=1
cat test | while read line
do
    echo "Line $count: $line"
    count=$[ $count + 1]
done
echo "Finished processing the file"
$
$ cat test
The quick brown dog jumps over the lazy fox.
This is a test, this is only a test.
O Romeo, Romeo! Wherefore art thou Romeo?
$
$ ./test28.sh
Line 1: The quick brown dog jumps over the lazy fox.
Line 2: This is a test, this is only a test.
Line 3: O Romeo, Romeo! Wherefore art thou Romeo?
Finished processing the file
$
```

while 循环会持续通过 read 命令处理文件中的行，直到 read 命令以非零退出状态码退出。
