# 脚本结构化命令

上一章给出的那些 shell 脚本里，shell 按照命令在脚本中出现的顺序依次进行处理。对顺序操作来说，这已经足够了，因为在这种操作环境下，你想要的就是所有的命令按照正确的顺序执行。然而，并非所有程序都如此操作。
许多程序要求对 shell 脚本中的命令施加一些逻辑流程控制。有一类命令会根据条件使脚本跳过某些命令。这样的命令通常称为`结构化命令`（structured command）。
结构化命令允许你改变程序执行的顺序。在 bash shell 中有不少结构化命令，我们会逐个研究。

## if 语句

最基本的结构化命令就是 if-then 语句。if-then 语句有如下格式。

if _command_  
then  
 _commands_  
fi

如果你在用其他编程语言的 if-then 语句，这种形式可能会让你有点困惑。在其他编程语言中，if 语句之后的对象是一个等式，这个等式的求值结果为 TRUE 或 FALSE。但 bash shell 的 if 语句并不是这么做的。

bash shell 的 if 语句会运行 if 后面的那个命令。如果该命令的退出状态码是 0（该命令成功运行），位于 then 部分的命令就会被执行。如果该命令的退出状态码是其他值，then 部分的命令就不会被执行，bash shell 会继续执行脚本中的下一个命令。fi 语句用来表示 if-then 语句到此结束。

这里有个简单的例子可解释这个概念。

```bash
$ cat test1.sh
#!/bin/bash
# testing the if statement
if pwd
then
    echo "It worked"
fi
$
```

这个脚本在 if 行采用了 pwd 命令。如果命令成功结束，echo 语句就会显示该文本字符串。在命令行运行该脚本时，会得到如下结果。

```bash
$ ./test1.sh
/home/Christine
It worked
$
```

shell 执行了 if 行中的 pwd 命令。由于退出状态码是 0，它就又执行了 then 部分的 echo 语句。

你可能在有些脚本中看到过 if-then 语句的另一种形式：

```bash
if command; then
    commands
fi
```

通过把分号放在待求值的命令尾部，就可以将 then 语句放在同一行上了，这样看起来更像其他编程语言中的 if-then 语句。

在 then 部分，你可以使用不止一条命令。可以像在脚本中的其他地方一样在这里列出多条命令。bash shell 会将这些命令当成一个块，如果 if 语句行的命令的退出状态值为 0，所有的命令都会被执行；如果 if 语句行的命令的退出状态不为 0，所有的命令都会被跳过。

```bash
$ cat test3.sh
#!/bin/bash
# testing multiple commands in the then section
#
testuser=Christine
#
if grep $testuser /etc/passwd; then
    echo "This is my first command"
    echo "This is my second command"
    echo "I can even put in other commands besides echo:"
    ls -a /home/$testuser/.b*
fi
$
```

if 语句行使用 grep 命令在/etc/passwd 文件中查找某个用户名当前是否在系统上使用。如果有用户使用了那个登录名，脚本会显示一些文本信息并列出该用户 HOME 目录的 bash 文件。

```bash
$ ./test3.sh
Christine:x:501:501:Christine B:/home/Christine:/bin/bash
This is my first command
This is my second command
I can even put in other commands besides echo:
/home/Christine/.bash_history  /home/Christine/.bash_profile
/home/Christine/.bash_logout   /home/Christine/.bashrc
$
```

但是，如果将 testuser 变量设置成一个系统上不存在的用户，则什么都不会显示。看起来也没什么新鲜的。如果在这里显示的一些消息可说明这个用户名在系统中未找到，这样可能就会显得更友好。此时可以用 if-then-else 语句来做到这一点。当 if 语句中的命令返回非零退出状态码时，bash shell 会执行 else 部分中的命令。现在可以复制并修改测试脚本来加入 else 部分。

```bash
$ cp test3.sh test4.sh
$
$ vim test4.sh
$
$ cat test4.sh
#!/bin/bash
# testing the else section
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
    echo "The bash files for user $testuser are:"
    ls -a /home/$testuser/.b*
    echo
else
    echo "The user $testuser does not exist on this system."
    echo
fi
$
$ ./test4.sh
The user NoSuchUser does not exist on this system.
$
```

这样就更友好了。跟 then 部分一样，else 部分也可以包含多条命令。

## 嵌套 if

有时你需要检查脚本代码中的多种条件。对此，可以使用嵌套的 if-then 语句。

要检查/etc/passwd 文件中是否存在某个用户名以及该用户的目录是否尚在，可以使用嵌套的 if-then 语句。嵌套的 if-then 语句位于主 if-then-else 语句的 else 代码块中。

```bash
$ ls -d /home/NoSuchUser/
/home/NoSuchUser/
$
$ cat test5.sh
#!/bin/bash
# Testing nested ifs
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
echo "The user $testuser exists on this system."
else
    echo "The user $testuser does not exist on this system."
    if ls -d /home/$testuser/
    then
        echo "However, $testuser has a directory."
    fi
fi
$
$ ./test5.sh
The user NoSuchUser does not exist on this system.
/home/NoSuchUser/
However, NoSuchUser has a directory.
$
```

这个脚本准确无误地发现，尽管登录名已经从/etc/passwd 中删除了，但是该用户的目录仍然存在。在脚本中使用这种嵌套 if-then 语句的问题在于代码不易阅读，很难理清逻辑流程。
可以使用 else 部分的另一种形式：elif。这样就不用再书写多个 if-then 语句了。elif 使用另一个 if-then 语句延续 else 部分。elif 语句行提供了另一个要测试的命令，这类似于原始的 if 语句行。如果 elif 后命令的退出状态码是 0，则 bash 会执行第二个 then 语句部分的命令。使用这种嵌套方法，代码更清晰，逻辑更易懂。甚至可以更进一步，让脚本检查拥有目录的不存在用户以及没有拥有目录的不存在用户。这可以通过在嵌套 elif 中加入一个 else 语句来实现。

```bash
$ cat test5.sh
#!/bin/bash
# Testing nested ifs - use elif & else
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
    echo "The user $testuser exists on this system."
#
elif ls -d /home/$testuser
then
    echo "The user $testuser does not exist on this system."
    echo "However, $testuser has a directory."
#
else
    echo "The user $testuser does not exist on this system."
    echo "And, $testuser does not have a directory."
fi
$
$ ./test5.sh
/home/NoSuchUser
The user NoSuchUser does not exist on this system.
However, NoSuchUser has a directory.
$
$ sudo rmdir /home/NoSuchUser
[sudo] password for Christine:
$
$ ./test5.sh
ls: cannot access /home/NoSuchUser: No such file or directory
The user NoSuchUser does not exist on this system.
And, NoSuchUser does not have a directory.
$
```

在/home/NoSuchUser 目录被删除之前，这个测试脚本执行的是 elif 语句，返回零值的退出状态。因此 elif 的 then 代码块中的语句得以执行。删除了/home/NoSuchUser 目录之后，elif 语句返回的是非零值的退出状态。这使得 elif 块中的 else 代码块得以执行。

> 记住，在 elif 语句中，紧跟其后的 else 语句属于 elif 代码块。它们并不属于之前的 if-then 代码块。

可以继续将多个 elif 语句串起来，形成一个大的 if-then-elif 嵌套组合。每块命令都会根据命令是否会返回退出状态码 0 来执行。记住，bash shell 会依次执行 if 语句，只有第一个返回退出状态码 0 的语句中的 then 部分会被执行。

```bash
if command1
then
    command set 1
elif command2
then
    command set 2
elif command3
then
    command set 3
elif command4
then
    command set 4
fi
```

尽管使用了 elif 语句的代码看起来更清晰，但是脚本的逻辑仍然会让人犯晕。在本章稍后你会看到如何使用 case 命令代替 if-then 语句的大量嵌套。

## test 命令

到目前为止，在 if 语句中看到的都是普通 shell 命令。你可能想问，if-then 语句是否能直接测试命令退出状态码之外的条件。
答案是不能。但在 bash shell 中有个好用的工具可以帮你通过 if-then 语句测试其他条件。

test 命令提供了在 if-then 语句中测试不同条件的途径。如果 test 命令中列出的条件成立，test 命令就会退出并返回退出状态码 0。这样 if-then 语句就与其他编程语言中的 if-then 语句以类似的方式工作了。如果条件不成立，test 命令就会退出并返回非零的退出状态码，这使得 if-then 语句不会再被执行。

如果只执行 test 命令本身，不写 test 命令的条件部分，它会以非零的退出状态码退出，并执行 else 语句块。当你加入一个条件时，test 命令会测试该条件。例如，可以使用 test 命令确定变量中是否有内容。这只需要一个简单的条件表达式。

```bash
$ cat test6.sh
#!/bin/bash
# Testing the test command
#
my_variable="Full"
#
if test $my_variable
then
    echo "The $my_variable expression returns a True"
#
else
    echo "The $my_variable expression returns a False"
fi
$
$ ./test6.sh
The Full expression returns a True
$
```

变量 my_variable 中包含有内容（Full），因此当 test 命令测试条件时，返回的退出状态为 0。这使得 then 语句块中的语句得以执行。如你所料，如果该变量中没有包含内容，就会出现相反的情况。

bash shell 提供了另一种条件测试方法，无需在 if-then 语句中声明 test 命令。

```bash
if [ condition ]
then
    commands
fi
```

方括号定义了测试条件，是与 test 命令同义的特殊 bash 命令。注意，第一个方括号之后和第二个方括号之前必须加上一个空格，否则就会报错。
test 命令可以判断三类条件：

- 数值比较
- 字符串比较
- 文件比较

接下来将会介绍如何在 if-then 语句中使用这些条件测试。

### 数值比较

使用 test 命令最常见的情形是对两个数值进行比较。如下列出了测试两个值时可用的条件参数。

- n1 -eq n2 检查 n1 是否与 n2 相等
- n1 -ge n2 检查 n1 是否大于或等于 n2
- n1 -gt n2 检查 n1 是否大于 n2
- n1 -le n2 检查 n1 是否小于或等于 n2
- n1 -lt n2 检查 n1 是否小于 n2
- n1 -ne n2 检查 n1 是否不等于 n2

数值条件测试可以用在数字和变量上。这里有个例子。

```bash
$ cat numeric_test.sh
#!/bin/bash
# Using numeric test evaluations
#
value1=10
value2=11
#
if [ $value1 -gt 5 ]
then
    echo "The test value $value1 is greater than 5"
fi
#
if [ $value1 -eq $value2 ]
then
    echo "The values are equal"
else
    echo "The values are different"
fi
#
$
```

第一个条件测试测试变量 value1 的值是否大于 5。第二个条件测试测试变量 value1 的值是否和变量 value2 的值相等。两个数值条件测试的结果和预想一致。

但是涉及浮点值时，数值条件测试会有一个限制。bash shell 只能处理整数。如果你只是要通过 echo 语句来显示这个结果，那没问题。但是，在基于数字的函数中就不行了，比如数值测试条件，不能在 test 命令中使用浮点值。

### 字符串比较

条件测试还允许比较字符串值。比较字符串比较烦琐。

- str1 = str2 检查 str1 是否和 str2 相同
- str1 != str2 检查 str1 是否和 str2 不同
- str1 < str2 检查 str1 是否比 str2 小
- str1 > str2 检查 str1 是否比 str2 大
- -n str1 检查 str1 的长度是否非 0
- -z str1 检查 str1 的长度是否为 0

记住，在比较字符串的相等性时，比较测试会将所有的标点和大小写情况都考虑在内。

要测试一个字符串是否比另一个字符串大就是麻烦的开始。当要开始使用测试条件的大于或小于功能时，就会出现两个经常困扰 shell 程序员的问题：

- 大于号和小于号必须转义，否则 shell 会把它们当作重定向符号，把字符串值当作文件名；
- 大于和小于顺序和 sort 命令所采用的不同。

在编写脚本时，第一条可能会导致一个不易察觉的严重问题。下面的例子展示了 shell 脚本编程初学者时常碰到的问题。

```bash
$ cat badtest.sh
#!/bin/bash
# mis-using string comparisons
#
val1=baseball
val2=hockey
#
if [ $val1 > $val2 ]
then
    echo "$val1 is greater than $val2"
else
    echo "$val1 is less than $val2"
fi
$
$ ./badtest.sh
baseball is greater than hockey
$ ls -l hockey
-rw-r--r-- 1 rich rich 0 Sep 30 19:08 hockey
$
```

这个脚本中只用了大于号，没有出现错误，但结果是错的。脚本把大于号解释成了输出重定向。因此，它创建了一个名为 hockey 的文件。由于重定向的顺利完成，test 命令返回了退出状态码 0，if 语句便以为所有命令都成功结束了。要解决这个问题，就需要使用反斜杠`\>`正确转义大于号。

第二个问题更细微，除非你经常处理大小写字母，否则几乎遇不到。sort 命令处理大写字母的方法刚好跟 test 命令相反。比如两个变量`val1=Testing val2=testing`，在 test 命令中，大写字母被认为是小于小写字母的。但 sort 命令恰好相反。当你将同样的字符串放进文件中并用 sort 命令排序时，小写字母会先出现。这是由各个命令使用的排序技术不同造成的。

test 命令中使用的是标准的 ASCII 顺序，根据每个字符的 ASCII 数值来决定排序结果。sort 命令使用的是系统的本地化语言设置中定义的排序顺序。对于英语，本地化设置指定了在排序顺序中小写字母出现在大写字母前。

> test 命令测试表达式使用标准的数学比较符号来表示字符串比较，而用文本代码来表示数值比较。这个细微的特性被很多程序员理解反了。如果你对数值使用了数学运算符号，shell 会将它们当成字符串值，可能无法得到正确的结果。

最后，-n 和-z 可以检查一个变量是否含有数据。如果一个变量为空字符串，或其从未被定义，那么均会被认为它的字符串长度为 0。

> 空的和未初始化的变量会对 shell 脚本测试造成灾难性的影响。如果不是很确定一个变量的内容，最好在将其用于数值或字符串比较之前先通过-n 或-z 来测试一下变量是否含有值。

### 文件比较

最后一类比较测试很有可能是 shell 编程中最为强大、也是用得最多的比较形式。它允许你测试 Linux 文件系统上文件和目录的状态。

- -d file 检查 file 是否存在并是一个目录
- -e file 检查 file 是否存在（文件或目录）
- -f file 检查 file 是否存在并是一个文件
- -r file 检查 file 是否存在并可读
- -s file 检查 file 是否存在并非空
- -w file 检查 file 是否存在并可写
- -x file 检查 file 是否存在并可执行
- -O file 检查 file 是否存在并属当前用户所有
- -G file 检查 file 是否存在并且默认组与当前用户相同
- file1 -nt file2 检查 file1 是否比 file2 新
- file1 -ot file2 检查 file1 是否比 file2 旧

这些测试条件使你能够在 shell 脚本中检查文件系统中的文件。它们经常出现在需要进行文件访问的脚本中。鉴于其使用广泛，建议熟练掌握。用于比较文件路径是相对你运行该脚本的目录而言的。

需要注意的是，-G 比较会检查文件的默认组，如果它匹配了用户的默认组，则测试成功。由于-G 比较只会检查默认组而非用户所属的所有组，这会叫人有点困惑。如果文件的组被改成了某个组，用户也是其中的一员，但用户并不以其为默认组，此时-G 比较会失败，因为它只比较默认组，不会去比较其他的组。

此外，在比较两个文件的新旧时，这些比较都不会先检查文件是否存在，如果你要检查的文件已经移走，就会出现问题。在你尝试使用-nt 或-ot 比较文件之前，必须先确认文件是存在的。

## 复合条件测试

if-then 语句允许你使用布尔逻辑来组合测试。有两种布尔运算符可用：

- [ condition1 ] && [ condition2 ]
- [ condition1 ] || [ condition2 ]

结合方括号测试方式和布尔逻辑组合，可以测试更多条件。

## if 语句的高级特性

bash shell 提供了两项可在 if-then 语句中使用的高级特性：

- 用于数学表达式的双括号
- 用于高级字符串处理功能的双方括号

### 使用双括号

双括号命令允许你在比较过程中使用高级数学表达式。test 命令只能在比较中使用简单的算术操作。双括号命令提供了更多的数学符号，这些符号对于用过其他编程语言的程序员而言并不陌生。除了 test 命令使用的标准数学运算符，如下列出了双括号命令中还可以使用的其他运算符。

- val++ 后增
- val-- 后减
- ++val 先增
- --val 先减
- ! 逻辑求反
- ~ 位求反
- \*\* 幂运算
- << 左位移
- \>\> 右位移
- & 位布尔和
- | 位布尔或
- && 逻辑和
- || 逻辑或

可以在 if 语句中用双括号命令，也可以在脚本中的普通命令里使用来赋值。

```bash
$ cat test23.sh
#!/bin/bash
# using double parenthesis
#
val1=10
#
if (( $val1 ** 2 > 90 ))
then
    (( val2 = $val1 ** 2 ))
    echo "The square of $val1 is $val2"
fi
$
$ ./test23.sh
The square of 10 is 100
$
```

注意，不需要将双括号中表达式里的大于号转义。这是双括号命令提供的另一个高级特性。

### 使用双方括号

双方括号命令提供了针对字符串比较的高级特性。双方括号使用了 test 命令中采用的标准字符串比较。但它提供了 test 命令未提供的另一个特性——`模式匹配`（pattern matching）。

> 双方括号在 bash shell 中工作良好。不过要小心，不是所有的 shell 都支持双方括号。

在模式匹配中，可以定义一个正则表达式（后续将详细讨论）来匹配字符串值。

```bash
$ cat test24.sh
#!/bin/bash
# using pattern matching
#
if [[ $USER == r* ]]
then
    echo "Hello $USER"
else
    echo "Sorry, I do not know you"
fi
$
$ ./test24.sh
Hello rich
$

```

在上面的脚本中，我们使用了双等号（==）。双等号将右边的字符串（r\*）视为一个模式，并应用模式匹配规则。双方括号命令$USER 环境变量进行匹配，看它是否以字母 r 开头。如果是的话，比较通过，shell 会执行 then 部分的命令。

## case 命令

你会经常发现自己在尝试计算一个变量的值，在一组可能的值中寻找特定值。在这种情形下，你不得不写出很长的 if-then-else 语句，就像下面这样。

```bash
$ cat test25.sh
#!/bin/bash
# looking for a possible value
#
if [ $USER = "rich" ]
then
    echo "Welcome $USER"
    echo "Please enjoy your visit"
elif [ $USER = "barbara" ]
then
    echo "Welcome $USER"
    echo "Please enjoy your visit"
elif [ $USER = "testing" ]
then
    echo "Special testing account"
elif [ $USER = "jessica" ]
then
    echo "Do not forget to logout when you're done"
else
    echo "Sorry, you are not allowed here"
fi
$
$ ./test25.sh
Welcome rich
Please enjoy your visit
$
```

elif 语句继续 if-then 检查，为比较变量寻找特定的值。有了 case 命令，就不需要再写出所有的 elif 语句来不停地检查同一个变量的值了。case 命令会采用列表格式来检查单个变量的多个值。

case 命令会将指定的变量与不同模式进行比较。如果变量和模式是匹配的，那么 shell 会执行为该模式指定的命令。可以通过竖线操作符在一行中分隔出多个模式模式。星号会捕获所有与已知模式不匹配的值。这里有个将 if-then-else 程序转换成用 case 命令的例子。

```bash
$ cat test26.sh
#!/bin/bash
# using the case command
#
case $USER in
rich | barbara)
    echo "Welcome, $USER"
    echo "Please enjoy your visit";;
testing)
    echo "Special testing account";;
jessica)
    echo "Do not forget to log off when you're done";;
*)
    echo "Sorry, you are not allowed here";;
esac
$
$ ./test26.sh
Welcome, rich
Please enjoy your visit
$
```

case 命令提供了一个更清晰的方法来为变量每个可能的值指定不同的选项。
