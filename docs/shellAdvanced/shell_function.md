# 函数

在编写 shell 脚本时，你经常会发现在多个地方使用了同一段代码。如果只是一小段代码，一般也无关紧要。但要在 shell 脚本中多次重写大块代码段就太累人了。bash shell 提供的用户自定义函数功能可以解决这个问题。可以将 shell 脚本代码放进函数中封装起来，这样就能在脚本中的任何地方多次使用它了。本章将会带你逐步了解如何创建自己的 shell 脚本函数，并演示如何在 shell 脚本应用中使用它们。

## 函数基础

在开始编写较复杂的 shell 脚本时，你会发现自己重复使用了部分能够执行特定任务的代码。这些代码有时很简单，比如显示一条文本消息，或者从脚本用户那里获得一个答案；有时则会比较复杂，需要作为大型处理过程中的一部分被多次使用。在后一类情况下，在脚本中一遍又一遍地编写同样的代码会很烦人。如果能只写一次，随后在脚本中可多次引用这部分代码就好了。

bash shell 提供了这种功能。函数是一个脚本代码块，你可以为其命名并在代码中任何位置重用。要在脚本中使用该代码块时，只要使用所起的函数名就行了（这个过程称为调用函数）。本节将会介绍如何在 shell 脚本中创建和使用函数。

### 创建函数

有两种格式可以用来在 bash shell 脚本中创建函数。第一种格式采用关键字 function，后跟分配给该代码块的函数名。

```bash
function name {
    commands
}
```

name 属性定义了赋予函数的唯一名称。脚本中定义的每个函数都必须有一个唯一的名称。commands 是构成函数的一条或多条 bash shell 命令。在调用该函数时，bash shell 会按命令在函数中出现的顺序依次执行，就像在普通脚本中一样。

在 bash shell 脚本中定义函数的第二种格式更接近于其他编程语言中定义函数的方式。

```bash
name() {
    commands
}
```

函数名后的空括号表明正在定义的是一个函数。这种格式的命名规则和之前定义 shell 脚本函数的格式一样。

### 使用函数

要在脚本中使用函数，只需要像其他 shell 命令一样，在行中指定函数名就行了。

```bash
$ cat test1
#!/bin/bash
# using a function in a script
function func1 {
    echo "This is an example of a function"
}
count=1
while [ $count -le 5 ]
do
    func1
    count=$[ $count + 1 ]
done
echo "This is the end of the loop"
func1
echo "Now this is the end of the script"
$
$ ./test1
This is an example of a function
This is an example of a function
This is an example of a function
This is an example of a function
This is an example of a function
This is the end of the loop
This is an example of a function
Now this is the end of the script
$
```

每次引用函数名 func1 时，bash shell 会找到 func1 函数的定义并执行你在那里定义的命令。

函数定义不一定非得是 shell 脚本中首先要做的事，但一定要小心。如果在函数被定义前使用函数，你会收到一条错误消息。

```bash
$ cat test2
#!/bin/bash
# using a function located in the middle of a script
count=1
echo "This line comes before the function definition"
function func1 {
    echo "This is an example of a function"
}
while [ $count -le 5 ]
do
    func1
    count=$[ $count + 1 ]
done
echo "This is the end of the loop"
func2
echo "Now this is the end of the script"
function func2 {
    echo "This is an example of a function"
}
$
$ ./test2
This line comes before the function definition
This is an example of a function
This is an example of a function
This is an example of a function
This is an example of a function
This is an example of a function
This is the end of the loop
./test2: func2: command not found
Now this is the end of the script
$
```

第一个函数 func1 的定义出现在脚本中的几条语句之后，这当然没任何问题。当 func1 函数在脚本中被使用时，shell 知道去哪里找它。  
然而，脚本试图在 func2 函数被定义之前使用它。由于 func2 函数还没有定义，脚本运行函数调用处时，产生了一条错误消息。  
你也必须注意函数名。记住，函数名必须是唯一的，否则也会有问题。如果你重定义了函数，新定义会覆盖原来函数的定义，这一切不会产生任何错误消息。

```bash
$ cat test3
#!/bin/bash
# testing using a duplicate function name
function func1 {
    echo "This is the first definition of the function name"
}
func1
function func1 {
    echo "This is a repeat of the same function name"
}
func1
echo "This is the end of the script"
$
$ ./test3
This is the first definition of the function name
This is a repeat of the same function name
This is the end of the script
$
```

func1 函数最初的定义工作正常，但重新定义该函数后，后续的函数调用都会使用第二个定义。

## 返回值

bash shell 会把函数当作一个小型脚本，运行结束时会返回一个退出状态码。有 3 种不同的方法来为函数生成退出状态码。

### 默认退出状态码

默认情况下，函数的退出状态码是函数中最后一条命令返回的退出状态码。在函数执行结束后，可以用标准变量$?来确定函数的退出状态码。

```bash
$ cat test4
#!/bin/bash
# testing the exit status of a function

func1() {
    echo "trying to display a non-existent file"
    ls -l badfile
}
echo "testing the function: "
func1 echo
"The exit status is: $?"
$
$ ./test4
testing the function:
trying to display a non-existent file
ls: badfile: No such file or directory
The exit status is: 1
$
```

函数的退出状态码是 1，这是因为函数中的最后一条命令没有成功运行。但你无法知道函数中其他命令中是否成功运行。看下面的例子。

```bash
$ cat test4b
#!/bin/bash
# testing the exit status of a function
func1() {
    ls -l badfile
    echo "This was a test of a bad command"
}
echo "testing the function:"
func1
echo "The exit status is: $?"
$
$ ./test4b
testing the function:
ls: badfile: No such file or directory
This was a test of a bad command
The exit status is: 0
$
```

这次，由于函数最后一条语句 echo 运行成功，该函数的退出状态码就是 0，尽管其中有一条命令并没有正常运行。使用函数的默认退出状态码是很危险的。幸运的是，有几种办法可以解决这个问题。

### 使用 return 命令

bash shell 使用 return 命令来退出函数并返回特定的退出状态码。return 命令允许指定一个整数值来定义函数的退出状态码，从而提供了一种简单的途径来编程设定函数退出状态码。

```bash
$ cat test5
#!/bin/bash
# using the return command in a function
function dbl {
    read -p "Enter a value: " value
    echo "doubling the value"
    return $[ $value * 2 ]
}
dbl
echo "The new value is $?"
$
```

dbl 函数会将$value变量中用户输入的值翻倍，然后用return命令返回结果。脚本用$?变量显示了该值。但当用这种方法从函数中返回值时，要小心了。记住下面两条技巧来避免问题：

- 记住，函数一结束就取返回值；
- 记住，退出状态码必须是 0~255。

如果在用$?变量提取函数返回值之前执行了其他命令，函数的返回值就会丢失。记住，$?变量会返回执行的最后一条命令的退出状态码。

第二个问题界定了返回值的取值范围。由于退出状态码必须小于 256，函数的结果必须生成一个小于 256 的整数值。任何大于 256 的值都会产生一个错误值,输出为计算结果对 256 取模。

```bash
$ ./test5
Enter a value: 200
doubling the value The new value is 144
$
```

要返回较大的整数值或者字符串值的话，你就不能用这种返回值的方法了。我们在下一节中将会介绍另一种方法。

### 使用函数输出

正如可以将命令的输出保存到 shell 变量中一样，你也可以对函数的输出采用同样的处理办法。可以用这种技术来获得任何类型的函数输出，并将其保存到变量中：

```bash
result=`dbl`
```

这个命令会将 dbl 函数的输出赋给$result 变量。下面是在脚本中使用这种方法的例子。

```bash
$ cat test5b
#!/bin/bash
# using the echo to return a value
function dbl {
    read -p "Enter a value: " value
    echo $[ $value * 2 ]
}
result=$(dbl)
echo "The new value is $result"
$
$ ./test5b
Enter a value: 200
The new value is 400
$
$ ./test5b
Enter a value: 1000
The new value is 2000
$
```

新函数会用 echo 语句来显示计算的结果。该脚本会获取 dbl 函数的输出，而不是查看退出状态码。

这个例子中演示了一个不易察觉的技巧。你会注意到 dbl 函数实际上输出了两条消息。read 命令输出了一条简短的消息来向用户询问输入值。bash shell 脚本非常聪明，并不将其作为 STDOUT 输出的一部分，并且忽略掉它。如果你用 echo 语句生成这条消息来向用户查询，那么它会与输出值一起被读进 shell 变量中。

> 通过这种技术，你还可以返回浮点值和字符串值。这使它成为一种获取函数返回值的强大方法。

## 在函数中使用变量

你可能已经注意到，在上面的 test5 例子中，我们在函数里用了一个叫作$result 的变量来保存处理后的值。在函数中使用变量时，你需要注意它们的定义方式以及处理方式。这是 shell 脚本中常见错误的根源。本节将会介绍一些处理 shell 脚本函数内外变量的方法。

### 向函数传递参数

我们在之前提到过，bash shell 会将函数当作小型脚本来对待。这意味着你可以像普通脚本那样向函数传递参数。函数可以使用标准的参数环境变量来表示命令行上传给函数的参数。例如，函数名会在$0变量中定义，函数命令行上的任何参数都会通过$1、$2等定义。也可以用特殊变量$#来判断传给函数的参数数目。在脚本中指定函数时，必须将参数和函数放在同一行，像这样：

```bash
func1 $value1 10
```

然后函数可以用参数环境变量来获得参数值。这里有个使用此方法向函数传值的例子。

```bash
$ cat test6
#!/bin/bash
# passing parameters to a function
function addem {
    if [ $# -eq 0 ] || [ $# -gt 2 ]
    then
        echo -1
    elif [ $# -eq 1 ]
    then
        echo $[ $1 + $1 ]
    else
        echo $[ $1 + $2 ]
    fi
}
echo -n "Adding 10 and 15: "
value=$(addem 10 15)
echo $value
echo -n "Let's try adding just one number: "
value=$(addem 10)
echo $value
echo -n "Now trying adding no numbers: "
value=$(addem)
echo $value
echo -n "Finally, try adding three numbers: "
value=$(addem 10 15 20)
echo $value
$
$ ./test6
Adding 10 and 15: 25
Let's try adding just one number: 20
Now trying adding no numbers: -1
Finally, try adding three numbers: -1
$
```

test6 脚本中的 addem 函数首先会检查脚本传给它的参数数目。如果没有任何参数，或者参数多于两个，addem 会返回值-1。如果只有一个参数，addem 会将参数与自身相加。如果有两个参数，addem 会将它们进行相加。

由于函数使用特殊参数环境变量作为自己的参数值，因此它无法直接获取脚本在命令行中的参数值。下面的例子将会运行失败。

```bash
$ cat badtest1
#!/bin/bash
# trying to access script parameters inside a function
function badfunc1 {
    echo $[ $1 * $2 ]
}
if [ $# -eq 2 ]
then
    value=$(badfunc1)
    echo "The result is $value"
else
    echo "Usage: badtest1 a b"
fi
$
$ ./badtest1
Usage: badtest1 a b
$ ./badtest1 10 15
./badtest1: *  : syntax error: operand expected (error token is "*  ")
The result is
$
```

尽管函数也使用了$1 和$2 变量，但它们和脚本主体中的$1 和$2 变量并不相同。要在函数中使用这些值，必须在调用函数时手动将它们传过去。

```bash
$ cat test7
#!/bin/bash
# trying to access script parameters inside a function
function func7 {
    echo $[ $1 * $2 ]
}
if [ $# -eq 2 ]
then
    value=$(func7 $1 $2)
    echo "The result is $value"
else
    echo "Usage: badtest1 a b"
fi
$
$ ./test7
Usage: badtest1 a b
$ ./test7 10 15
The result is 150
$
```

通过将$1 和$2 变量传给函数，它们就能跟其他变量一样供函数使用了。

### 在函数中处理变量

给 shell 脚本程序员带来麻烦的原因之一就是变量的`作用域`。作用域是变量可见的区域。函数中定义的变量与普通变量的作用域不同。也就是说，对脚本的其他部分而言，它们是隐藏的。函数使用两种类型的变量：

- 全局变量
- 局部变量

下面几节将会介绍这两种类型的变量在函数中的用法。

1. 全局变量

`全局变量`是在 shell 脚本中任何地方都有效的变量。如果你在脚本的主体部分定义了一个全局变量，那么可以在函数内读取它的值。类似地，如果你在函数内定义了一个全局变量，可以在脚本的主体部分读取它的值。

默认情况下，你在脚本中定义的任何变量都是全局变量。在函数外定义的变量可在函数内正常访问。

```bash
$ cat test8
#!/bin/bash
# using a global variable to pass a value
function dbl {
    value=$[ $value * 2 ]
}
read -p "Enter a value: " value
dbl
echo "The new value is: $value"
$
$ ./test8
Enter a value: 450
The new value is: 900
$
```

$value 变量在函数外定义并被赋值。当 dbl 函数被调用时，该变量及其值在函数中都依然有效。如果变量在函数内被赋予了新值，那么在脚本中引用该变量时，新值也依然有效。

但这其实很危险，尤其是如果你想在不同的 shell 脚本中使用函数的话。它要求你清清楚楚地知道函数中具体使用了哪些变量，包括那些用来计算非返回值的变量。这里有个例子可说明事情是如何搞砸的。

```bash
$ cat badtest2
#!/bin/bash
# demonstrating a bad use of variables
function func1 {
    temp=$[ $value + 5 ]
    result=$[ $temp * 2 ]
}
temp=4
value=6
func1
echo "The result is $result"
if [ $temp -gt $value ]
then
    echo "temp is larger"
else
    echo "temp is smaller"
fi
$
$ ./badtest2
The result is 22
temp is larger
$
```

由于函数中用到了$temp 变量，它的值在脚本中使用时受到了影响，产生了意想不到的后果。有个简单的办法可以在函数中解决这个问题，下面将会介绍。

2. 局部变量

无需在函数中使用全局变量，函数内部使用的任何变量都可以被声明成局部变量。要实现这一点，只要在变量声明的前面加上 local 关键字就可以了。

```bash
local temp
```

也可以在变量赋值语句中使用 local 关键字：

```bash
local temp=$[ $value + 5 ]
```

local 关键字保证了变量只局限在该函数中。如果脚本中在该函数之外有同样名字的变量，那么 shell 将会保持这两个变量的值是分离的。现在你就能很轻松地将函数变量和脚本变量隔离开了，只共享需要共享的变量。

```bash
$ cat test9
#!/bin/bash
# demonstrating the local keyword
function func1 {
    local temp=$[ $value + 5 ]
    result=$[ $temp * 2 ]
}
temp=4
value=6
func1
echo "The result is $result"
if [ $temp -gt $value ]
then
    echo "temp is larger"
else
    echo "temp is smaller"
fi
$
$ ./test9 The result is 22
temp is smaller
$
```

现在，在 func1 函数中使用$temp变量时，并不会影响在脚本主体中赋给$temp 变量的值。

## 数组变量和函数

在之前我们讨论了使用数组来在单个变量中保存多个值的高级用法。在函数中使用数组变量值有点麻烦，而且还需要一些特殊考虑。本节将会介绍一种方法来解决这个问题。

### 向函数传数组参数

向脚本函数传递数组变量的方法会有点不好理解。将数组变量当作单个参数传递的话，它不会起作用。

```bash
$ cat badtest3
#!/bin/bash
# trying to pass an array variable
function testit {
    echo "The parameters are: $@"
    thisarray=$1
    echo "The received array is ${thisarray[*]}"
}
myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
testit $myarray
$
$ ./badtest3
The original array is: 1 2 3 4 5
The parameters are: 1
The received array is 1
$
```

如果你试图将该数组变量作为函数参数，函数只会取数组变量的第一个值。

要解决这个问题，你必须将该数组变量的值分解成单个的值，然后将这些值作为函数参数使用。在函数内部，可以将所有的参数重新组合成一个新的变量。下面是个具体的例子。

```bash
$ cat test10
#!/bin/bash
# array variable to function test
function testit {
    local newarray
    newarray=`echo "$@"`
    echo "The new array value is: ${newarray[*]}"
}
myarray=(1 2 3 4 5)
echo "The original array is ${myarray[*]}"
testit ${myarray[*]}
$
$ ./test10
The original array is 1 2 3 4 5
The new array value is: 1 2 3 4 5
$
```

该脚本用$myarray 变量来保存所有的数组元素，然后将它们都放在函数的命令行上。该函数随后从命令行参数中重建数组变量。在函数内部，数组仍然可以像其他数组一样使用。

```bash
$ cat test11
#!/bin/bash
# adding values in an array
function addarray {
    local sum=0
    local newarray
    newarray=($(echo "$@"))
    for value in ${newarray[*]}
    do
        sum=$[ $sum + $value ]
    done
    echo $sum
}
myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
arg1=$(echo ${myarray[*]})
result=$(addarray $arg1)
echo "The result is $result"
$
$ ./test11
The original array is: 1 2 3 4 5
The result is 15
$
```

addarray 函数会遍历所有的数组元素，将它们累加在一起。你可以在 myarray 数组变量中放置任意多的值，addarry 函数会将它们都加起来。

### 从函数返回数组

从函数里向 shell 脚本传回数组变量也用类似的方法。函数用 echo 语句来按正确顺序输出单个数组值，然后脚本再将它们重新放进一个新的数组变量中。

```bash
$ cat test12
#!/bin/bash
# returning an array value
function arraydblr {
    local origarray
    local newarray
    local elements
    local i
    origarray=($(echo "$@"))
    newarray=($(echo "$@"))
    elements=$[ $# - 1 ]
    for (( i = 0; i <= $elements; i++ ))
    {
        newarray[$i]=$[ ${origarray[$i]} * 2 ]
    }
    echo ${newarray[*]}
}
myarray=(1 2 3 4 5)
echo "The original array is: ${myarray[*]}"
arg1=$(echo ${myarray[*]})
result=($(arraydblr $arg1))
echo "The new array is: ${result[*]}"
$
$ ./test12
The original array is: 1 2 3 4 5
The new array is: 2 4 6 8 10
```

该脚本用$arg1 变量将数组值传给 arraydblr 函数。arraydblr 函数将该数组重组到新的数组变量中，生成该输出数组变量的一个副本。然后对数据元素进行遍历，将每个元素值翻倍，并将结果存入函数中该数组变量的副本。

arraydblr 函数使用 echo 语句来输出每个数组元素的值。脚本用 arraydblr 函数的输出来重新生成一个新的数组变量。

## 函数递归

局部函数变量的一个特性是自成体系。除了从脚本命令行处获得的变量，自成体系的函数不需要使用任何外部资源。

这个特性使得函数可以`递归地`调用，也就是说，函数可以调用自己来得到结果。通常递归函数都有一个最终可以迭代到的基准值。许多高级数学算法用递归对复杂的方程进行逐级规约，直到基准值定义的那级。

递归算法的经典例子是计算阶乘。一个数的阶乘是该数之前的所有数乘以该数的值。因此，要计算 5 的阶乘，可以执行如下方程：

```bash
5! = 1 * 2 * 3 * 4 * 5 = 120
```

使用递归，方程可以简化成以下形式：

```bash
x! = x * (x-1)!
```

也就是说，x 的阶乘等于 x 乘以 x1 的阶乘。这可以用简单的递归脚本表达为：

```bash
function factorial {
    if [ $1 -eq 1 ]
    then
        echo 1
    else
        local temp=$[ $1 - 1 ]
        local result=`factorial $temp`
        echo $[ $result * $1 ]
    fi
}
```

阶乘函数用它自己来计算阶乘的值：

```bash
$ cat test13
#!/bin/bash
# using recursion
function factorial {
    if [ $1 -eq 1 ]
    then
        echo 1
    else
        local temp=$[ $1 - 1 ]
        local result=`factorial $temp`
        echo $[ $result * $1 ]
    fi
}
read -p "Enter value: " value
result=$(factorial $value)
echo "The factorial of $value is: $result"
$
$ ./test13
Enter value: 5
The factorial of 5 is: 120
$
```

使用阶乘函数很容易。创建了这样的函数后，你可能想把它用在其他脚本中。接下来，我们来看看如何有效地利用函数。

## 创建库

使用函数可以在脚本中省去一些输入工作，这一点是显而易见的。但如果你碰巧要在多个脚本中使用同一段代码呢？显然，为了使用一次而在每个脚本中都定义同样的函数太过麻烦。

有个方法能解决这个问题！bash shell 允许创建函数`库文件`，然后在多个脚本中引用该库文件。

这个过程的第一步是创建一个包含脚本中所需函数的公用库文件。这里有个叫作 myfuncs 的库文件，它定义了 3 个简单的函数。

```bash
$ cat myfuncs
# my script functions
function addem {
    echo $[ $1 + $2 ]
}
function multem {
    echo $[ $1 * $2 ]
}
function divem {
    if [ $2 -ne 0 ]
    then
        echo $[ $1 / $2 ]
    else
        echo -1
    fi
}
$
```

下一步是在用到这些函数的脚本文件中包含 myfuncs 库文件。从这里开始，事情就变复杂了。

问题出在 shell 函数的作用域上。和环境变量一样，shell 函数仅在定义它的 shell 会话内有效。如果你在 shell 命令行界面的提示符下运行 myfuncs shell 脚本，shell 会创建一个新的 shell 并在其中运行这个脚本。它会为那个新 shell 定义这三个函数，但当你运行另外一个要用到这些函数的脚本时，它们是无法使用的。

这同样适用于脚本。如果你尝试像普通脚本文件那样运行库文件，函数并不会出现在脚本中。

```bash
$ cat badtest4
#!/bin/bash
# using a library file the wrong way
./myfuncs
result=$(addem 10 15)
echo "The result is $result"
$
$ ./badtest4
./badtest4: addem: command not found
The result is
$

```

使用函数库的关键在于 source 命令。source 命令会在当前 shell 上下文中执行命令，而不是创建一个新 shell。可以用 source 命令来在 shell 脚本中运行库文件脚本。这样脚本就可以使用库中的函数了。

source 命令有个快捷的别名，称作`点操作符`（dot operator）。要在 shell 脚本中运行 myfuncs 库文件，只需添加下面这行：

```bash
. ./myfuncs
```

这个例子假定 myfuncs 库文件和 shell 脚本位于同一目录。如果不是，你需要使用相应路径访问该文件。这里有个用 myfuncs 库文件创建脚本的例子。

```bash
$ cat test14
#!/bin/bash
# using functions defined in a library file
. ./myfuncs
value1=10
value2=5
result1=$(addem $value1 $value2)
result2=$(multem $value1 $value2)
result3=$(divem $value1 $value2)
echo "The result of adding them is: $result1"
echo "The result of multiplying them is: $result2"
echo "The result of dividing them is: $result3"
$
$ ./test14
The result of adding them is: 15
The result of multiplying them is: 50
The result of dividing them is: 2
$
```

该脚本成功地使用了 myfuncs 库文件中定义的函数。

## 在命令行上使用函数

可以用脚本函数来执行一些十分复杂的操作。有时也很有必要在命令行界面的提示符下直接使用这些函数。和在 shell 脚本中将脚本函数当命令使用一样，在命令行界面中你也可以这样做。这个功能很不错，因为一旦在 shell 中定义了函数，你就可以在整个系统中使用它了，无需担心脚本是不是在 PATH 环境变量里。重点在于让 shell 能够识别这些函数。有几种方法可以实现。

### 在命令行上创建函数

因为 shell 会解释用户输入的命令，所以可以在命令行上直接定义一个函数。有两种方法。一种方法是采用单行方式定义函数。

```bash
$ function divem { echo $[ $1 / $2 ];  }
$ divem 100 5
20
$
```

当在命令行上定义函数时，你必须记得在每个命令后面加个分号，这样 shell 就能知道在哪里是命令的起止了。

```bash
$ function doubleit { read -p "Enter value: " value; echo $[
    $value * 2 ]; }
$
$ doubleit Enter value: 20
40
$
```

另一种方法是采用多行方式来定义函数。在定义时，bash shell 会使用次提示符来提示输入更多命令。用这种方法，你不用在每条命令的末尾放一个分号，只要按下回车键就行。

```bash
$ function multem {
> echo $[ $1 * $2 ]
> }
$ multem 2 5
10
$
```

在函数的尾部使用花括号，shell 就会知道你已经完成了函数的定义。

> 在命令行上创建函数时要特别小心。如果你给函数起了个跟内建命令或另一个命令相同的名字，函数将会覆盖原来的命令。

### 在.bashrc 文件中定义函数

在命令行上直接定义 shell 函数的明显缺点是退出 shell 时，函数就消失了。对于复杂的函数来说，这可是个麻烦事。一个非常简单的方法是将函数定义在一个特定的位置，这个位置在每次启动一个新 shell 的时候，都会由 shell 重新载入。最佳地点就是.bashrc 文件。bash shell 在每次启动时都会在主目录下查找这个文件，不管是交互式 shell 还是从现有 shell 中启动的新 shell。

1.  直接定义函数

可以直接在主目录下的.bashrc 文件中定义函数。许多 Linux 发行版已经在.bashrc 文件中定义了一些东西，所以注意不要误删了。把你写的函数放在文件末尾就行了。这里有个例子。

```bash
$ cat .bashrc
# .bashrc
# Source global definitions
if [ -r /etc/bashrc ]; then
    . /etc/bashrc
fi
function addem {
    echo $[ $1 + $2 ]
}
$
```

该函数会在下次启动新 bash shell 时生效。随后你就能在系统上任意地方使用这个函数了。

2. 读取函数文件

只要是在 shell 脚本中，都可以用 source 命令（或者它的别名点操作符）将库文件中的函数添加到你的.bashrc 脚本中。

```bash
$ cat .bashrc
# .bashrc
# Source global definitions
if [ -r /etc/bashrc ]; then
    . /etc/bashrc
fi
    . /home/rich/libraries/myfuncs
$
```

要确保库文件的路径名正确，以便 bash shell 能够找到该文件。下次启动 shell 时，库中的所有函数都可在命令行界面下使用了。

```bash
$ addem 10 5
15
$ multem 10 5
50
$ divem 10 5
2
$
```

更好的是，shell 还会将定义好的函数传给子 shell 进程，这样一来，这些函数就自动能够用于该 shell 会话中的任何 shell 脚本了。你可以写个脚本，试试在不定义或使用 source 的情况下，直接使用这些函数。

```bash
$ cat test15
#!/bin/bash
# using a function defined in the .bashrc file
value1=10
value2=5
result1=$(addem $value1 $value2)
result2=$(multem $value1 $value2)
result3=$(divem $value1 $value2)
echo "The result of adding them is: $result1"
echo "The result of multiplying them is: $result2"
echo "The result of dividing them is: $result3"
$
$ ./test15
The result of adding them is: 15
The result of multiplying them is: 50
The result of dividing them is: 2
$
```

甚至都不用对库文件使用 source，这些函数就可以完美地运行在 shell 脚本中。

## 实例

函数的应用绝不仅限于创建自己的函数自娱自乐。在开源世界中，共享代码才是关键，而这一点同样适用于脚本函数。你可以下载大量各式各样的函数，并将其用于自己的应用程序中。本节介绍了如何使用 GNU shtool shell 脚本函数库。shtool 库提供了一些简单的 shell 脚本函数，可以用来完成日常的 shell 功能，例如处理临时文件和目录或者格式化输出显示。在 arch linux 下，可通过如下命令安装：

```bash
yay -S shtool
```

shtool 库提供了大量方便的、可用于 shell 脚本的函数。下面列出了库中可用的函数。

- arx 创建归档文件（包含一些扩展功能）
- echo 显示字符串，并提供了一些扩展构件
- fixperm 改变目录树中的文件权限
- install 安装脚本或文件
- mdate 显示文件或目录的修改时间
- mkdir 创建一个或更多目录
- mkln 使用相对路径创建链接
- mkshadow 创建一棵阴影树
- move 带有替换功能的文件移动
- ath 处理程序路径
- platform 显示平台标识
- prop 显示一个带有动画效果的进度条
- rotate 转置日志文件
- scpp 共享的 C 预处理器
- slo 根据库的类别，分离链接器选项
- subst 使用 sed 的替换操作
- table 以表格的形式显示由字段分隔（field-separated）的数据
- tarball 从文件和目录中创建 tar 文件
- version 创建版本信息文件

每个 shtool 函数都包含大量的选项和参数，你可以利用它们改变函数的工作方式。更多内容可使用 man 查看。可以在命令行或自己的 shell 脚本中直接使用 shtool 函数。下面是一个在 shell 脚本中使用 platform 函数的例子。

```bash
$ cat test16
#!/bin/bash
shtool platform
$ ./test16
Arch rolling (AMD64)
$
```

platform 函数会返回 Linux 发行版以及系统所使用的 CPU 硬件的相关信息。另一个函数是 prop 函数。它可以使用\、|、/和-字符创建一个旋转的进度条。这是一个非常漂亮的工具，可以告诉 shell 脚本用户目前正在进行一些后台处理工作。要使用 prop 函数，只需要将希望监看的输出管接到 shtool 脚本就行了。

```bash
$ ls –al /usr/bin | shtool prop –p "waiting..."
wating......\
$
```

prop 函数会在处理过程中不停地变换进度条字符。在本例中，输出信息来自于 ls 命令。你能看到多少进度条取决于 CPU 能以多快的速度列出/usr/bin 中的文件！-p 选项允许你定制输出文本，这段文本会出现在进度条字符之前。好了，尽情享受吧！
