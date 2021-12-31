# 更多的结构化命令

上一章里，你看到了如何通过检查命令的输出和变量的值来改变 shell 脚本程序的流程。本章会继续介绍能够控制 shell 脚本流程的结构化命令。你会了解如何重复一些过程和命令，也就是循环执行一组命令直至达到了某个特定条件。本章将会讨论和演示 bash shell 的循环命令 for、while 和 until 等。

## for 命令

重复执行一系列命令在编程中很常见。通常你需要重复一组命令直至达到某个特定条件，比如处理某个目录下的所有文件、系统上的所有用户或是某个文本文件中的所有行。for 命令有几种不同的方式来读取列表中的值，下面几节将会介绍各种方式。

### 读取列表中的值

for 命令最基本的用法就是遍历 for 命令自身所定义的一系列值。

```bash
$ cat test1
 #!/bin/bash
# basic for command
for test in Alabama Alaska Arizona Arkansas California Colorado
do
    echo The next state is $test
done
echo "The last state we visited was $test"
test=Connecticut
echo "Wait, now we're visiting $test"
$ ./test1
The next state is Alabama
The next state is Alaska
The next state is Arizona
The next state is Arkansas
The next state is California
The next state is Colorado
The last state we visited was Colorado
Wait, now we're visiting Connecticut
$
```

每次 for 命令遍历值列表，它都会将列表中的下个值赋给$test变量。$test 变量可以像 for 命令语句中的其他脚本变量一样使用。在最后一次迭代后，$test 变量的值会在 shell 脚本的剩余部分一直保持有效。它会一直保持最后一次迭代的值（除非你修改了它）。$test 变量保持了其值，也允许我们修改它的值，并在 for 命令循环之外跟其他变量一样使用

事情并不会总像你在 for 循环中看到的那么简单。有时会遇到难处理的数据。有时 for 循环的值列表中可能存在中间有空格的值，此时使用单引号或者双引号将中间存在空格的值括起来即可。有时候，有的值自身中存在单引号或双引号，这时需要用另外一种相反的引号将其括起来，或者使用反斜杠转义即可正常使用。

### 从变量读取列表

通常 shell 脚本遇到的情况是，你将一系列值都集中存储在了一个变量中，然后需要遍历变量中的整个列表。也可以通过 for 命令完成这个任务。

```bash
$ cat test4
#!/bin/bash
# using a variable to hold the list
list="Alabama Alaska Arizona Arkansas Colorado"
list=$list" Connecticut"
for state in $list
do
echo "Have you ever visited $state?"
done
$ ./test4
Have you ever visited Alabama?
Have you ever visited Alaska?
Have you ever visited Arizona?
Have you ever visited Arkansas?
Have you ever visited Colorado?
Have you ever visited Connecticut?

```

$list变量包含了用于迭代的标准文本值列表。注意，代码还是用了另一个赋值语句向$list 变量包含的已有列表中添加（或者说是拼接）了一个值。这是向变量中存储的已有文本字符串尾部添加文本的一个常用方法。

### 从命令读取值

生成列表中所需值的另外一个途径就是使用命令的输出。可以用命令替换来执行任何能产生输出的命令，然后在 for 命令中使用该命令的输出。

```bash
$ cat test5
#!/bin/bash
# reading values from a file
file="states"
for state in $(cat $file)
do
echo "Visit beautiful $state"
done
$ cat states
Alabama
Alaska
Arizona
Arkansas
Colorado
Connecticut
Delaware
Florida
Georgia
$ ./test5
Visit beautiful Alabama
Visit beautiful Alaska
Visit beautiful Arizona
Visit beautiful Arkansas
Visit beautiful Colorado
Visit beautiful Connecticut
Visit beautiful Delaware
Visit beautiful Florida
Visit beautiful Georgia
$
```

这个例子在命令替换中使用了 cat 命令来输出文件 states 的内容。你会注意到 states 文件中每一行有一个州，而不是通过空格分隔的。for 命令仍然以每次一行的方式遍历了 cat 命令的输出，假定每个州都是在单独的一行上。但这并没有解决数据中有空格的问题。如果你列出了一个名字中有空格的州，for 命令仍然会将每个单词当作单独的值。这是有原因的，下一节我们将会了解。

> test5 的代码范例将文件名赋给变量，文件名中没有加入路径。这要求文件和脚本位于同一个目录中。如果不是的话，你需要使用全路径名（不管是绝对路径还是相对路径）来引用文件位置。

### 更改字段分隔符

造成这个问题的原因是特殊的环境变量 IFS，叫作内部字段分隔符（internal field separator）。IFS 环境变量定义了 bash shell 用作字段分隔符的一系列字符。默认情况下，bash shell 会将下列字符当作字段分隔符：

- 空格
- 制表符
- 换行符

如果 bash shell 在数据中看到了这些字符中的任意一个，它就会假定这表明了列表中一个新数据字段的开始。在处理可能含有空格的数据（比如文件名）时，这会非常麻烦，就像你在上一个脚本示例中看到的。

要解决这个问题，可以在 shell 脚本中临时更改 IFS 环境变量的值来限制被 bash shell 当作字段分隔符的字符。例如，如果你想修改 IFS 的值，使其只能识别换行符，那就必须这么做：

```bash
IFS=$'\n'
```

将这个语句加入到脚本中，告诉 bash shell 在数据值中忽略空格和制表符。对前一个脚本使用这种方法，shell 脚本就能够使用列表中含有空格的值了。

> 在处理代码量较大的脚本时，可能在一个地方需要修改 IFS 的值，然后忽略这次修改，在脚本的其他地方继续沿用 IFS 的默认值。一个可参考的安全实践是在改变 IFS 之前保存原来的 IFS 值，之后再恢复它。这种技术可以这样实现： IFS.OLD=$IFS   IFS=$'\n' <在代码中使用新的 IFS 值> IFS=$IFS.OLD 这就保证了在脚本的后续操作中使用的是 IFS 的默认值。

还有其他一些 IFS 环境变量的绝妙用法。假定你要遍历一个文件中用冒号分隔的值（比如在/etc/passwd 文件中）。你要做的就是将 IFS 的值设为冒号

```bash
IFS=:
```

如果要指定多个 IFS 字符，只要将它们在赋值行串起来就行。

```bash
IFS=$'\n':;"
```

这个赋值会将换行符、冒号、分号和双引号作为字段分隔符。如何使用 IFS 字符解析数据没有任何限制。

### 用通配符读取目录

最后，可以用 for 命令来自动遍历目录中的文件。进行此操作时，必须在文件名或路径名中使用通配符。它会强制 shell 使用文件扩展匹配。文件扩展匹配是生成匹配指定通配符的文件名或路径名的过程。

如果不知道所有的文件名，这个特性在处理目录中的文件时就非常好用。

```bash
$ cat test6
#!/bin/bash
# iterate through all the files in a directory
for file in /home/rich/test/*
do
    if [ -d "$file" ]
    then
        echo "$file is a directory"
    elif [ -f "$file" ]
    then
    echo "$file is a file"
    fi
done
$ ./test6
/home/rich/test/dir1 is a directory
/home/rich/test/myprog.c is a file
/home/rich/test/myprog is a file
/home/rich/test/myscript is a file
/home/rich/test/newdir is a directory
/home/rich/test/newfile is a file
/home/rich/test/newfile2 is a file
/home/rich/test/testdir is a directory
/home/rich/test/testing is a file
/home/rich/test/testprog is a file
/home/rich/test/testprog.c is a file
$
```

for 命令会遍历/home/rich/test/\*输出的结果。该代码用 test 命令测试了每个条目（使用方括号方法），以查看它是目录（通过-d 参数）还是文件（通过-f 参数）

注意，我们在这个例子的 if 语句中做了一些不同的处理

```bash
if [ -d "$file" ]
```

在 Linux 中，目录名和文件名中包含空格当然是合法的。要适应这种情况，应该将$file 变量用双引号圈起来。如果不这么做，遇到含有空格的目录名或文件名时就会有错误产生。

```bash
./test6: line 6: [: too many arguments
./test6: line 9: [: too many arguments
```

在 test 命令中，bash shell 会将额外的单词当作参数，进而造成错误。

也可以在 for 命令中列出多个目录通配符，将目录查找和列表合并进同一个 for 语句。

```bash
$ cat test7
#!/bin/bash
# iterating through multiple directories
for file in /home/rich/.b* /home/rich/badtest
do
    if [ -d "$file" ]
    then
        echo "$file is a directory"
    elif [ -f "$file" ]
    then
        echo "$file is a file"
    else
        echo "$file doesn't exist"
    fi
done

$ ./test7
/home/rich/.backup.timestamp is a file
/home/rich/.bash_history is a file
/home/rich/.bash_logout is a file
/home/rich/.bash_profile is a file
/home/rich/.bashrc is a file
/home/rich/badtest doesn't exist
$
```

for 语句首先使用了文件扩展匹配来遍历通配符生成的文件列表，然后它会遍历列表中的下一个文件。可以将任意多的通配符放进列表中。

> 注意，你可以在数据列表中放入任何东西。即使文件或目录不存在，for 语句也会尝试处理列表中的内容。在处理文件或目录时，这可能会是个问题。你无法知道你正在尝试遍历的目录是否存在：在处理之前测试一下文件或目录总是好的。

## C 语言风格的 for 命令

如果你从事过 C 语言编程，可能会对 bash shell 中 for 命令的工作方式有点惊奇。在 C 语言中，for 循环通常定义一个变量，然后这个变量会在每次迭代时自动改变。通常程序员会将这个变量用作计数器，并在每次迭代中让计数器增一或减一。bash 的 for 命令也提供了这个功能。本节将会告诉你如何在 bash shell 脚本中使用 C 语言风格的 for 命令。

C 语言的 for 命令有一个用来指明变量的特定方法，一个必须保持成立才能继续迭代的条件，以及另一个在每个迭代中改变变量的方法。当指定的条件不成立时，for 循环就会停止。条件等式通过标准的数学符号定义。比如，考虑下面的 C 语言代码：

```c
for (i = 0; i < 10; i++) {
    printf("The next number is %d\n", i);
}
```

这段代码产生了一个简单的迭代循环，其中变量 i 作为计数器。第一部分将一个默认值赋给该变量。中间的部分定义了循环重复的条件。当定义的条件不成立时，for 循环就停止迭代。最后一部分定义了迭代的过程。在每次迭代之后，最后一部分中定义的表达式会被执行。在本例中，i 变量会在每次迭代后增一。

bash shell 也支持一种 for 循环，它看起来跟 C 语言风格的 for 循环类似，但有一些细微的不同，其中包括一些让 shell 脚本程序员困惑的东西。以下是 bash 中 C 语言风格的 for 循环的基本格式。

```bash
for (( variableassignment ; condition ; iterationprocess ))
```

C 语言风格的 for 循环的格式会让 bash shell 脚本程序员摸不着头脑，因为它使用了 C 语言风格的变量引用方式而不是 shell 风格的变量引用方式。C 语言风格的 for 命令看起来如下。

```c
for (( a = 1; a < 10; a++ ))
```

注意，有些部分并没有遵循 bash shell 标准的 for 命令：

- 变量赋值可以有空格；
- 条件中的变量不以美元符开头；
- 迭代过程的算式未用 expr 命令格式。

shell 开发人员创建了这种格式以更贴切地模仿 C 语言风格的 for 命令。这虽然对 C 语言程序员来说很好，但也会把专家级的 shell 程序员弄得一头雾水。在脚本中使用 C 语言风格的 for 循环时要小心。

以下例子是在 bash shell 程序中使用 C 语言风格的 for 命令

```bash
$ cat test8
#!/bin/bash
# testing the C-style for loop
for (( i=1; i <= 3; i++ ))
do
    echo "The next number is $i"
done
$ ./test8
The next number is 1
The next number is 2
The next number is 3
$
```

for 循环通过定义好的变量（本例中是变量 i）来迭代执行这些命令。在每次迭代中，$i 变量包含了 for 循环中赋予的值。在每次迭代后，循环的迭代过程会作用在变量上，在本例中，变量增一。

C 语言风格的 for 命令也允许为迭代使用多个变量。循环会单独处理每个变量，你可以为每个变量定义不同的迭代过程。尽管可以使用多个变量，但你只能在 for 循环中定义一种条件。

```bash
$ cat test9
#!/bin/bash
# multiple variables
for (( a=1, b=10; a <= 3; a++, b-- ))
do
    echo "$a - $b"
done
$ ./test9
1 - 10
2 - 9
3 - 8
$
```

变量 a 和 b 分别用不同的值来初始化并且定义了不同的迭代过程。循环的每次迭代在增加变量 a 的同时减小了变量 b。

## while 命令

while 命令某种意义上是 if-then 语句和 for 循环的混杂体。while 命令允许定义一个要测试的命令，然后循环执行一组命令，只要定义的测试命令返回的是退出状态码 0。它会在每次迭代的一开始测试 test 命令。在 test 命令返回非零退出状态码时，while 命令会停止执行那组命令。

while 命令的格式是：

```bash
while test command
do
    other commands
done
```

while 命令中定义的 test command 和 if-then 语句中的格式一模一样。可以使用任何普通的 bash shell 命令，或者用 test 命令进行条件测试，比如测试变量值。
while 命令的关键在于所指定的 test command 的退出状态码必须随着循环中运行的命令而改变。如果退出状态码不发生变化，while 循环就将一直不停地进行下去。
最常见的 test command 的用法是用方括号来检查循环命令中用到的 shell 变量的值。

```bash
$ cat test10
#!/bin/bash
# while command test

var1=10
while [ $var1 -gt 0 ]
do
    echo $var1
    var1=$[ $var1 - 1 ]
done
$ ./test10
10
9
8
7
6
5
4
3
2
1
$
```

while 命令定义了每次迭代时检查的测试条件：`while [ $var1 -gt 0 ]` 。只要测试条件成立，while 命令就会不停地循环执行定义好的命令。在这些命令中，测试条件中用到的变量必须被修改，否则就会陷入无限循环。在本例中，我们用 shell 算术来将变量值减一：`var1=$[ $var1 - 1 ]` 。while 循环会在测试条件不再成立时停止。

while 命令允许你在 while 语句行定义多个测试命令。只有最后一个测试命令的退出状态码会被用来决定什么时候结束循环。如果你不够小心，可能会导致一些有意思的结果。下面的例子将说明这一点。

```bash
$ cat test11
#!/bin/bash
# testing a multicommand while loop
var1=10
while echo $var1
    [ $var1 -ge 0 ]
do
    echo "This is inside the loop"
    var1=$[ $var1 - 1 ]
done
$ ./test11
10
This is inside the loop
9
This is inside the loop
8
This is inside the loop
7
This is inside the loop
6
This is inside the loop
5
This is inside the loop
4
This is inside the loop
3
This is inside the loop
2
This is inside the loop
1
This is inside the loop
0
This is inside the loop
-1
```

while 语句中定义了两个测试命令。第一个测试简单地显示了 var1 变量的当前值。第二个测试用方括号来判断 var1 变量的值。在循环内部，echo 语句会显示一条简单的消息，说明循环被执行了。注意当你运行本例时输出最后还有一个-1。

while 循环会在 var1 变量等于 0 时执行 echo 语句，然后将 var1 变量的值减一。接下来再次执行测试命令，用于下一次迭代。echo 测试命令被执行并显示了 var 变量的值（现在小于 0 了）。直到 shell 执行 test 测试命令，whle 循环才会停止。

## until 命令

until 命令和 while 命令工作的方式完全相反。until 命令要求你指定一个通常返回非零退出状态码的测试命令。只有测试命令的退出状态码不为 0，bash shell 才会执行循环中列出的命令。一旦测试命令返回了退出状态码 0，循环就结束了。和 while 命令类似，你可以在 until 命令语句中放入多个测试命令。只有最后一个命令的退出状态码决定了 bash shell 是否执行已定义的 other commands。下面是使用 until 命令的一个例子。

```bash
$ cat test12
#!/bin/bash
# using the until command
var1=100
until [ $var1 -eq 0 ]
do
    echo $var1
    var1=$[ $var1 - 25 ]
done
$ ./test12
100
75
50
25
$
```

本例中会测试 var1 变量来决定 until 循环何时停止。只要该变量的值等于 0，until 命令就会停止循环。同 while 命令一样，在 until 命令中使用多个测试命令时要注意。

```bash
$ cat test13
#!/bin/bash
# using the until command
var1=100
until echo $var1
    [ $var1 -eq 0 ]
do
    echo Inside the loop: $var1
    var1=$[ $var1 - 25 ]
done
$ ./test13
100
Inside the loop: 100
75
Inside the loop: 75
50
Inside the loop: 50
25
Inside the loop: 25
0
$

```

## 嵌套循环

循环语句可以在循环内使用任意类型的命令，包括其他循环命令。这种循环叫作`嵌套循环`（nested loop）。注意，在使用嵌套循环时，你是在迭代中使用迭代，与命令运行的次数是乘积关系。不注意这点的话，有可能会在脚本中造成问题。

```bash
$ cat test14
#!/bin/bash
# nesting for loops
for (( a = 1; a <= 3; a++ ))
do
    echo "Starting loop $a:"
    for (( b = 1; b <= 3; b++ ))
        do
            echo "   Inside loop: $b"
        done
done
$ ./test14
Starting loop 1:
    Inside loop: 1
    Inside loop: 2
    Inside loop: 3
Starting loop 2:
    Inside loop: 1
    Inside loop: 2
    Inside loop: 3
Starting loop 3:
    Inside loop: 1
    Inside loop: 2
    Inside loop: 3
$
```

这个被嵌套的循环（也称为内部循环，inner loop）会在外部循环的每次迭代中遍历一次它所有的值。注意，两个循环的 do 和 done 命令没有任何差别。bash shell 知道当第一个 done 命令执行时是指内部循环而非外部循环。

在混用循环命令时也一样，比如在 while 循环内部放置一个 for 循环。

```bash
$ cat test15
#!/bin/bash
# placing a for loop inside a while loop
var1=5
while [ $var1 -ge 0 ]
do
    echo "Outer loop: $var1"
    for (( var2 = 1; $var2 < 3; var2++ ))
    do
        var3=$[ $var1 * $var2 ]
        echo "  Inner loop: $var1 * $var2 = $var3"
    done
var1=$[ $var1 - 1 ]
done
$ ./test15
Outer loop: 5
    Inner loop: 5 * 1 = 5
    Inner loop: 5 * 2 = 10
Outer loop: 4
    Inner loop: 4 * 1 = 4
    Inner loop: 4 * 2 = 8
Outer loop: 3
    Inner loop: 3 * 1 = 3
    Inner loop: 3 * 2 = 6
Outer loop: 2
    Inner loop: 2 * 1 = 2
    Inner loop: 2 * 2 = 4
Outer loop: 1
    Inner loop: 1 * 1 = 1
    Inner loop: 1 * 2 = 2
Outer loop: 0
    Inner loop: 0 * 1 = 0
    Inner loop: 0 * 2 = 0
$
```

同样，shell 能够区分开内部 for 循环和外部 while 循环各自的 do 和 done 命令。如果真的想挑战脑力，可以混用 until 和 while 循环。

```bash
$ cat test16
#!/bin/bash
# using until and while loops
var1=3
until [ $var1 -eq 0 ]
do
    echo "Outer loop: $var1"
    var2=1
    while [ $var2 -lt 5 ]
    do
        var3=$(echo "scale=4; $var1 / $var2" | bc)
        echo "   Inner loop: $var1 / $var2 = $var3"
        var2=$[ $var2 + 1 ]
    done
    var1=$[ $var1 - 1 ]
done
$ ./test16
Outer loop: 3
    Inner loop: 3 / 1 = 3.0000
    Inner loop: 3 / 2 = 1.5000
    Inner loop: 3 / 3 = 1.0000
    Inner loop: 3 / 4 = .7500
Outer loop: 2
    Inner loop: 2 / 1 = 2.0000
    Inner loop: 2 / 2 = 1.0000
    Inner loop: 2 / 3 = .6666
    Inner loop: 2 / 4 = .5000
Outer loop: 1
    Inner loop: 1 / 1 = 1.0000
    Inner loop: 1 / 2 = .5000
    Inner loop: 1 / 3 = .3333
    Inner loop: 1 / 4 = .2500
$
```

外部的 until 循环以值 3 开始，并继续执行到值等于 0。内部 while 循环以值 1 开始并一直执行，只要值小于 5。每个循环都必须改变在测试条件中用到的值，否则循环就会无止尽进行下去。

## 循环处理文件数据

如果需要遍历存储在文件中的数据，则需要结合已经讲过的两种技术：

- 使用嵌套循环
- 修改 IFS 环境变量

通过修改 IFS 环境变量，就能强制 for 命令将文件中的每行都当成单独的一个条目来处理，即便数据中有空格也是如此。一旦从文件中提取出了单独的行，可能需要再次利用循环来提取行中的数据。

典型的例子是处理/etc/passwd 文件中的数据。这要求你逐行遍历/etc/passwd 文件，然后将 IFS 变量的值改成冒号，这样就能分隔开每行中的各个数据段了。

```bash
#!/bin/bash
# changing the IFS value
IFS.OLD=$IFS
IFS=$'\n'
for entry in $(cat /etc/passwd)
do
    echo "Values in $entry –"
    IFS=:
    for value in $entry
    do
        echo "   $value"
    done
done
$
```

这个脚本使用了两个不同的 IFS 值来解析数据。第一个 IFS 值解析出/etc/passwd 文件中的单独的行。内部 for 循环接着将 IFS 的值修改为冒号，允许你从/etc/passwd 的行中解析出单独的值。内部循环会解析出/etc/passwd 每行中的各个值。这种方法在处理外部导入电子表格所采用的逗号分隔的数据时也很方便。

## 控制循环

你可能会想，一旦启动了循环，就必须苦等到循环完成所有的迭代。并不是这样的。有两个命令能帮我们控制循环内部的情况：

- break 命令
- continue 命令

每个命令在如何控制循环的执行方面有不同的用法。下面几节将介绍如何使用这些命令来控制循环。

### break 命令

break 命令是退出循环的一个简单方法。可以用 break 命令来退出任意类型的循环，包括 while 和 until 循环。有几种情况可以使用 break 命令，本节将介绍这些方法。

1. 跳出单个循环

在 shell 执行 break 命令时，它会尝试跳出当前正在执行的循环。

```bash
cat test17
#!/bin/bash
# breaking out of a for loop
for var1 in 1 2 3 4 5 6 7 8 9 10
do
    if [ $var1 -eq 5 ]
    then
        break
    fi
    echo "Iteration number: $var1"
done
echo "The for loop is completed"


$ ./test17
Iteration number: 1
Iteration number: 2
Iteration number: 3
Iteration number: 4
The for loop is completed
$
```

for 循环通常都会遍历列表中指定的所有值。但当满足 if-then 的条件时，shell 会执行 break 命令，停止 for 循环。这种方法同样适用于 while 和 until 循环。

```bash
$ cat test18
#!/bin/bash
# breaking out of a while loop
var1=1
while [ $var1 -lt 10 ]
do
    if [ $var1 -eq 5 ]
    then
        break
    fi
    echo "Iteration: $var1"
    var1=$[ $var1 + 1 ]
done
echo "The while loop is completed"
$ ./test18
Iteration: 1
Iteration: 2
Iteration: 3
Iteration: 4
The while loop is completed
$
```

while 循环会在 if-then 的条件满足时执行 break 命令，终止。

2. 跳出内部循环

在处理多个循环时，break 命令会自动终止你所在的最内层的循环。

```bash
$ cat test19
#!/bin/bash
# breaking out of an inner loop
for (( a = 1; a < 4; a++ ))
do
    echo "Outer loop: $a"
    for (( b = 1; b < 100; b++ ))
    do
        if [ $b -eq 5 ]
        then
            break
        fi
        echo "   Inner loop: $b"
    done
done
$ ./test19
Outer loop: 1
    Inner loop: 1
    Inner loop: 2
    Inner loop: 3
    Inner loop: 4
Outer loop: 2
    Inner loop: 1
    Inner loop: 2
    Inner loop: 3
    Inner loop: 4
Outer loop: 3
    Inner loop: 1
    Inner loop: 2
    Inner loop: 3
    Inner loop: 4
$
```

内部循环里的 for 语句指明当变量 b 等于 100 时停止迭代。但内部循环的 if-then 语句指明当变量 b 的值等于 5 时执行 break 命令。注意，即使内部循环通过 break 命令终止了，外部循环依然继续执行。

3. 跳出外部循环

有时你在内部循环，但需要停止外部循环。break 命令接受单个命令行参数值：

```bash
break n
```

其中 n 指定了要跳出的循环层级。默认情况下，n 为 1，表明跳出的是当前的循环。如果你将 n 设为 2，break 命令就会停止下一级的外部循环。

```bash
$ cat test20
#!/bin/bash
# breaking out of an outer loop
for (( a = 1; a < 4; a++ ))
do
    echo "Outer loop: $a"
    for (( b = 1; b < 100; b++ ))
    do
        if [ $b -gt 4 ]
        then
            break 2
        fi
        echo "   Inner loop: $b"
    done
done
$ ./test20
Outer loop: 1
    Inner loop: 1
    Inner loop: 2
    Inner loop: 3
    Inner loop: 4
$
```

注意，当 shell 执行了 break 命令后，外部循环就停止了。

### continue 命令

continue 命令可以提前中止某次循环中的命令，但并不会完全终止整个循环。可以在循环内部设置 shell 不执行命令的条件。这里有个在 for 循环中使用 continue 命令的简单例子。

```bash
$ cat test21
#!/bin/bash
# using the continue command
for (( var1 = 1; var1 < 15; var1++ ))
do
    if [ $var1 -gt 5 ] && [ $var1 -lt 10 ]
    then
        continue
    fi
    echo "Iteration number: $var1"
done
$ ./test21
Iteration number: 1
Iteration number: 2
Iteration number: 3
Iteration number: 4
Iteration number: 5
Iteration number: 10
Iteration number: 11
Iteration number: 12
Iteration number: 13
Iteration number: 14
$
```

当 if-then 语句的条件被满足时（值大于 5 且小于 10），shell 会执行 continue 命令，跳过此次循环中剩余的命令，但整个循环还会继续。当 if-then 的条件不再被满足时，一切又回到正轨。

也可以在 while 和 until 循环中使用 continue 命令，但要特别小心。记住，当 shell 执行 continue 命令时，它会跳过剩余的命令。如果你在其中某个条件里对测试条件变量进行增值，问题就会出现。

```bash
$ cat badtest3
#!/bin/bash
# improperly using the continue command in a while loop
var1=0
while echo "while iteration: $var1"
    [ $var1 -lt 15 ]
do
    if [ $var1 -gt 5 ] && [ $var1 -lt 10 ]
    then
        continue
    fi
    echo "   Inside iteration number: $var1"
    var1=$[ $var1 + 1 ]
done
$ ./badtest3 | more
while iteration: 0
    Inside iteration number: 0
while iteration: 1
    Inside iteration number: 1
while iteration: 2
    Inside iteration number: 2
while iteration: 3
    Inside iteration number: 3
while iteration: 4
    Inside iteration number: 4
while iteration: 5
    Inside iteration number: 5
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
while iteration: 6
```

你得确保将脚本的输出重定向到了 more 命令，这样才能停止输出。在 if-then 的条件成立之前，所有一切看起来都很正常，然后 shell 执行了 continue 命令。当 shell 执行 continue 命令时，它跳过了 while 循环中余下的命令。不幸的是，被跳过的部分正是$var1 计数变量增值的地方，而这个变量又被用于 while 测试命令中。这意味着这个变量的值不会再变化了，从前面连续的输出显示中你也可以看出来。

和 break 命令一样，continue 命令也允许通过命令行参数指定要继续执行哪一级循环：

```bash
continue n
```

其中 n 定义了要继续的循环层级。下面是继续外部 for 循环的一个例子。

```bash
$ cat test22
#!/bin/bash
# continuing an outer loop
for (( a = 1; a <= 5; a++ ))
do
    echo "Iteration $a:"
    for (( b = 1; b < 3; b++ ))
    do
        if [ $a -gt 2 ] && [ $a -lt 4 ]
        then
            continue 2
        fi
        var3=$[ $a * $b ]
        echo "   The result of $a * $b is $var3"
    done
done

$ ./test22
Iteration 1:
    The result of 1 * 1 is 1
    The result of 1 * 2 is 2
Iteration 2:
    The result of 2 * 1 is 2
    The result of 2 * 2 is 4
Iteration 3:
Iteration 4:
    The result of 4 * 1 is 4
    The result of 4 * 2 is 8
Iteration 5:
    The result of 5 * 1 is 5
    The result of 5 * 2 is 10
$
```

此处用 continue 命令来停止处理循环内的命令，但会继续处理外部循环。注意，值为 3 的那次迭代并没有处理任何内部循环语句。尽管 continue 命令停止了内部处理过程，但外部循环依然会继续。

## 处理循环的输出

最后，在 shell 脚本中，你可以对循环的输出使用管道或进行重定向。这可以通过在 done 命令之后添加一个处理命令来实现。

```bash
for file in /home/rich/*
do
    if [ -d "$file" ]
    then
        echo "$file is a directory"
    elif
        echo "$file is a file"
    fi
done > output.txt
```

shell 会将 for 命令的结果重定向到文件 output.txt 中，而不是显示在屏幕上。考虑下面将 for 命令的输出重定向到文件的例子。

```bash
$ cat test23
#!/bin/bash
# redirecting the for output to a file
for (( a = 1; a < 10; a++ ))
do
    echo "The number is $a"
done > test23.txt
echo "The command is finished."
$ ./test23
The command is finished.
$ cat test23.txt
The number is 1
The number is 2
The number is 3
The number is 4
The number is 5
The number is 6
The number is 7
The number is 8
The number is 9
$
```

shell 创建了文件 test23.txt 并将 for 命令的输出重定向到这个文件。shell 在 for 命令之后正常显示了 echo 语句。

这种方法同样适用于将循环的结果管接给另一个命令。

```bash
$ cat test24
#!/bin/bash
# piping a loop to another command
for state in "North Dakota" Connecticut Illinois Alabama Tennessee
do
    echo "$state is the next place to go"
done | sort
echo "This completes our travels"
$ ./test24
Alabama is the next place to go
Connecticut is the next place to go
Illinois is the next place to go
North Dakota is the next place to go
Tennessee is the next place to go
This completes our travels
$
```

state 值并没有在 for 命令列表中以特定次序列出。for 命令的输出传给了 sort 命令，该命令会改变 for 命令输出结果的顺序。运行这个脚本实际上说明了结果已经在脚本内部排好序了。

## 实战例子

现在你已经看到了 shell 脚本中各种循环的使用方法，来看一些实际应用的例子吧。循环是对系统数据进行迭代的常用方法，无论是目录中的文件还是文件中的数据。下面的一些例子演示了如何使用简单的循环来处理数据。

### 查找可执行文件

当你从命令行中运行一个程序的时候，Linux 系统会搜索一系列目录来查找对应的文件。这些目录被定义在环境变量 PATH 中。如果你想找出系统中有哪些可执行文件可供使用，只需要扫描 PATH 环境变量中所有的目录就行了。如果要徒手查找的话，就得花点时间了。不过我们可以编写一个小小的脚本，轻而易举地搞定这件事。

首先是创建一个 for 循环，对环境变量 PATH 中的目录进行迭代。处理的时候别忘了设置 IFS 分隔符。

```bash
IFS=:
for folder in $PATH
do
```

现在你已经将各个目录存放在了变量$folder 中，可以使用另一个 for 循环来迭代特定目录中的所有文件。

```bash
for file in $folder/*
do
```

最后一步是检查各个文件是否具有可执行权限，你可以使用 if-then 测试功能来实现。

```bash
if [ -x $file ]
then
    echo "   $file"
fi
```

好了，搞定了！将这些代码片段组合成脚本就行了。

```bash
$ cat test25
#!/bin/bash
# finding files in the PATH
IFS=:
for folder in $PATH
do
    echo "$folder:"
    for file in $folder/*
    do
        if [ -x $file ]
        then
            echo "   $file"
        fi
    done
done
$
```

运行这段代码时，你会得到一个可以在命令行中使用的可执行文件的列表。输出显示了在环境变量 PATH 所包含的所有目录中找到的全部可执行文件。

### 创建多个用户账户

shell 脚本的目标是让系统管理员过得更轻松。如果你碰巧工作在一个拥有大量用户的环境中，最烦人的工作之一就是创建新用户账户。好在可以使用 while 循环来降低工作的难度。

你不用为每个需要创建的新用户账户手动输入 useradd 命令，而是可以将需要添加的新用户账户放在一个文本文件中，然后创建一个简单的脚本进行处理。这个文本文件的格式如下：

```bash
userid,user name
```

第一个条目是你为新用户账户所选用的用户 ID。第二个条目是用户的全名。两个值之间使用逗号分隔，这样就形成了一种名为逗号分隔值的文件格式（或者是.csv）。这种文件格式在电子表格中极其常见，所以你可以轻松地在电子表格程序中创建用户账户列表，然后将其保存成.csv 格式，以备 shell 脚本读取及处理。

要读取文件中的数据，得用上一点 shell 脚本编程技巧。我们将 IFS 分隔符设置成逗号，并将其放入 while 语句的条件测试部分。然后使用 read 命令读取文件中的各行。实现代码如下：

```bash
while IFS=’,’ read –r userid name
```

read 命令会自动读取.csv 文本文件的下一行内容，所以不需要专门再写一个循环来处理。当 read 命令返回 FALSE 时（也就是读取完整个文件时），while 命令就会退出。妙极了！
要想把数据从文件中送入 while 命令，只需在 while 命令尾部使用一个重定向符就可以了。将各部分处理过程写成脚本如下。

```bash
$ cat test26
#!/bin/bash
# process new user accounts
input="users.csv"
while IFS=',' read -r userid name
do
    echo "adding $userid"
    useradd -c "$name" -m $userid
done < "$input"
$
```

$input 变量指向数据文件，并且该变量被作为 while 命令的重定向数据。users.csv 文件内容如下。

```bash
$ cat users.csv
rich,Richard Blum
christine,Christine Bresnahan
barbara,Barbara Blum
tim,Timothy Bresnahan
$
```

必须作为 root 用户才能运行这个脚本，因为 useradd 命令需要 root 权限。执行此脚本后，看一眼/etc/passwd 文件，你会发现账户已经创建好了。

---

循环是编程的一部分。bash shell 提供了三种可用于脚本中的循环命令。for 命令允许你遍历一系列的值，不管是在命令行里提供好的、包含在变量中的还是通过文件扩展匹配获得的文件名和目录名。while 命令使用普通命令或测试命令提供了基于命令条件的循环。只有在命令（或条件）产生退出状态码 0 时，while 循环才会继续迭代指定的一组命令。until 命令也提供了迭代命令的一种方法，但它的迭代是建立在命令（或条件）产生非零退出状态码的基础上。这个特性允许你设置一个迭代结束前都必须满足的条件。可以在 shell 脚本中对循环进行组合，生成多层循环。bash shell 提供了 continue 和 break 命令，允许你根据循环内的不同值改变循环的正常流程。bash shell 还允许使用标准的命令重定向和管道来改变循环的输出。你可以使用重定向来将循环的输出重定向到一个文件或是另一个命令。这就为控制 shell 脚本执行提供了丰富的功能。下一章将会讨论如何和 shell 脚本用户交互。shell 脚本通常并不完全是自成一体的。它们需要在运行时被提供某些外部数据。下一章将讨论各种可用来向 shell 脚本提供实时数据的方法。
