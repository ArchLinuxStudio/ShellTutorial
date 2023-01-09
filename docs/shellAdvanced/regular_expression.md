# 正则表达式

在 shell 脚本中成功运用 sed 编辑器和 gawk 程序的关键在于熟练使用正则表达式。这可不是件简单的事，从大量数据中过滤出特定数据可能会（而且经常会）很复杂。本章将介绍如何在 sed 编辑器和 gawk 程序中创建正则表达式来过滤出需要的数据。

## 什么是正则表达式

理解正则表达式的第一步在于弄清它们到底是什么。本节将会解释什么是正则表达式并介绍 Linux 如何使用正则表达式。

### 定义

正则表达式是你所定义的`模式模板`（pattern template），Linux 工具可以用它来过滤文本。Linux 工具（比如 sed 编辑器或 gawk 程序）能够在处理数据时使用正则表达式对数据进行模式匹配。如果数据匹配模式，它就会被接受并进一步处理；如果数据不匹配模式，它就会被滤掉。

正则表达式模式利用通配符来描述数据流中的一个或多个字符。Linux 中有很多场景都可以使用通配符来描述不确定的数据。在本书之前你已经看到过在 Linux 的 ls 命令中使用通配符列出文件和目录的例子。

星号通配符允许你只列出满足特定条件的文件，例如：

```bash
$ ls -al da*
-rw-r--r--    1 rich     rich           45 Nov 26 12:42 data
-rw-r--r--    1 rich     rich           25 Dec  4 12:40 data.tst
-rw-r--r--    1 rich     rich          180 Nov 26 12:42 data1
-rw-r--r--    1 rich     rich           45 Nov 26 12:44 data2
-rw-r--r--    1 rich     rich           73 Nov 27 12:31 data3
-rw-r--r--    1 rich     rich           79 Nov 28 14:01 data4
-rw-r--r--    1 rich     rich          187 Dec  4 09:45 datatest
$
```

da\*参数会让 ls 命令只列出名字以 da 开头的文件。文件名中 da 之后可以有任意多个字符（包括什么也没有）。ls 命令会读取目录中所有文件的信息，但只显示跟通配符匹配的文件的信息。

正则表达式通配符模式的工作原理与之类似。正则表达式模式含有文本或特殊字符，为 sed 编辑器和 gawk 程序定义了一个匹配数据时采用的模板。可以在正则表达式中使用不同的特殊字符来定义特定的数据过滤模式。

### 正则表达式的类型

使用正则表达式最大的问题在于有不止一种类型的正则表达式。Linux 中的不同应用程序可能会用不同类型的正则表达式。这其中包括编程语言（Java、Perl 和 Python）、Linux 实用工具（比如 sed 编辑器、gawk 程序和 grep 工具）以及主流应用（比如 MySQL 和 PostgreSQL 数据库服务器）。

正则表达式是通过`正则表达式引擎`（regular expression engine）实现的。正则表达式引擎是一套底层软件，负责解释正则表达式模式并使用这些模式进行文本匹配。在 Linux 中，有两种流行的正则表达式引擎：

- POSIX 基础正则表达式（basic regular expression，BRE）引擎
- POSIX 扩展正则表达式（extended regular expression，ERE）引擎

大多数 Linux 工具都至少符合 POSIX BRE 引擎规范，能够识别该规范定义的所有模式符号。遗憾的是，有些工具（比如 sed 编辑器）只符合了 BRE 引擎规范的子集。这是出于速度方面的考虑导致的，因为 sed 编辑器希望能尽可能快地处理数据流中的文本。

POSIX BRE 引擎通常出现在依赖正则表达式进行文本过滤的编程语言中。它为常见模式提供了高级模式符号和特殊符号，比如匹配数字、单词以及按字母排序的字符。gawk 程序用 ERE 引擎来处理它的正则表达式模式。

由于实现正则表达式的方法太多，很难用一个简洁的描述来涵盖所有可能的正则表达式。后续几节将会讨论最常见的正则表达式，并演示如何在 sed 编辑器和 gawk 程序中使用它们。

## 定义 BRE 模式

最基本的 BRE 模式是匹配数据流中的文本字符。本节将会演示如何在正则表达式中定义文本以及会得到什么样的结果。

### 纯文本

前面演示了如何在 sed 编辑器和 gawk 程序中用标准文本字符串来过滤数据。通过下面的例子来复习一下。

```bash
$ echo "This is a test" | sed -n '/test/p'
This is a test
$ echo "This is a test" | sed -n '/trial/p'
$
$ echo "This is a test" | gawk '/test/{print $0}'
This is a test
$ echo "This is a test" | gawk '/trial/{print $0}'
$
```

第一个模式定义了一个单词 test。sed 编辑器和 gawk 程序脚本用它们各自的 print 命令打印出匹配该正则表达式模式的所有行。由于 echo 语句在文本字符串中包含了单词 test，数据流文本能够匹配所定义的正则表达式模式，因此 sed 编辑器显示了该行。

第二个模式也定义了一个单词，这次是 trial。因为 echo 语句文本字符串没包含该单词，所以正则表达式模式没有匹配，因此 sed 编辑器和 gawk 程序都没打印该行。

你可能注意到了，正则表达式并不关心模式在数据流中的位置。它也不关心模式出现了多少次。一旦正则表达式匹配了文本字符串中任意位置上的模式，它就会将该字符串传回 Linux 工具。

关键在于将正则表达式模式匹配到数据流文本上。重要的是记住正则表达式对匹配的模式非常挑剔。第一条原则就是：正则表达式模式都区分大小写。这意味着它们只会匹配大小写也相符的模式。

```bash
$ echo "This is a test" | sed -n '/this/p'
$
$ echo "This is a test" | sed -n '/This/p'
This is a test
$
```

第一次尝试没能匹配成功，因为 this 在字符串中并不都是小写，而第二次尝试在模式中使用大写字母，所以能正常工作。

在正则表达式中，你不用写出整个单词。只要定义的文本出现在数据流中，正则表达式就能够匹配。

```bash
$ echo "The books are expensive" | sed -n '/book/p'
The books are expensive
$
```

尽管数据流中的文本是 books，但数据中含有正则表达式 book，因此正则表达式模式跟数据匹配。当然，反之正则表达式就不成立了。

```bash
$ echo "The book is expensive" | sed -n '/books/p'
$
```

完整的正则表达式文本并未在数据流中出现，因此匹配失败，sed 编辑器不会显示任何文本。

你也不用局限于在正则表达式中只用单个文本单词，可以在正则表达式中使用空格和数字。

```bash
$ echo "This is line number 1" | sed -n '/ber 1/p'
This is line number 1
$
```

在正则表达式中，空格和其他的字符并没有什么区别。

```bash
$ echo "This is line number1" | sed -n '/ber 1/p'
$
```

如果你在正则表达式中定义了空格，那么它必须出现在数据流中。甚至可以创建匹配多个连续空格的正则表达式模式。

```bash
$ cat data1
This is a normal line of text.
This is  a line with too many spaces.
$ sed -n '/  /p' data1
This is  a line with too many spaces.
$
```

单词间有两个空格的行匹配正则表达式模式。这是用来查看文本文件中空格问题的好办法。

### 特殊字符

在正则表达式模式中使用文本字符时，有些事情值得注意。在正则表达式中定义文本字符时有一些特例。有些字符在正则表达式中有特别的含义。如果要在文本模式中使用这些字符，结果会超出你的意料。

正则表达式识别的特殊字符包括：

```bash
.*[]^${}\+?|()
```

随着本章内容的继续，你会了解到这些特殊字符在正则表达式中有何用处。不过现在只要记住不能在文本模式中单独使用这些字符就行了。果要用某个特殊字符作为文本字符，就必须`转义`。在转义特殊字符时，你需要在它前面加一个特殊字符来告诉正则表达式引擎应该将接下来的字符当作普通的文本字符。这个特殊字符就是反斜线（\）。举个例子，如果要查找文本中的美元符，只要在它前面加个反斜线。

```bash
$ cat data2
The cost is $4.00
$ sed -n '/\$/p' data2
The cost is $4.00
$
```

由于反斜线是特殊字符，如果要在正则表达式模式中使用它，你必须对其转义，这样就产生了两个反斜线。

```bash
$ echo "\ is a special character" | sed -n '/\\/p'
\ is a special character
$
```

最终，尽管正斜线不是正则表达式的特殊字符，但如果它出现在 sed 编辑器或 gawk 程序的正则表达式中，你就会得到一个错误。

```bash
$ echo "3 / 2" | sed -n '///p'
sed: -e expression #1, char 2: No previous regular expression
$
```

要使用正斜线，也需要进行转义。

```bash
$ echo "3 / 2" | sed -n '/\//p'
3 / 2
$
```

现在 sed 编辑器能正确解释正则表达式模式了，一切都很顺利。

### 锚字符

默认情况下，当指定一个正则表达式模式时，只要模式出现在数据流中的任何地方，它就能匹配。有两个特殊字符可以用来将模式锁定在数据流中的行首或行尾。

脱字符（^）定义从数据流中文本行的行首开始的模式。如果模式出现在行首之外的位置，正则表达式模式则无法匹配。要用脱字符，就必须将它放在正则表达式中指定的模式前面。

```bash
$ echo "The book store" | sed -n '/^book/p'
$
$ echo "Books are great" | sed -n '/^Book/p'
Books are great
$
```

脱字符会在每个由换行符决定的新数据行的行首检查模式。

```bash
$ cat data3
This is a test line.
this is another test line.
A line that tests this feature.
Yet more testing of this
$ sed -n '/^this/p' data3
this is another test line.
$
```

只要模式出现在新行的行首，脱字符就能够发现它。  
如果你将脱字符放到模式开头之外的其他位置，那么它就跟普通字符一样，不再是特殊字符了：

```bash
$ echo "This ^ is a test" | sed -n '/s ^/p'
This ^ is a test
$
```

由于脱字符出现在正则表达式模式的尾部，sed 编辑器会将它当作普通字符来匹配。

> 如果指定正则表达式模式时只用了脱字符，就不需要用反斜线来转义。但如果你在模式中先指定了脱字符，随后还有其他一些文本，那么你必须在脱字符前用转义字符。

---

跟在行首查找模式相反的就是在行尾查找。特殊字符美元符（$）定义了行尾锚点。将这个特殊字符放在文本模式之后来指明数据行必须以该文本模式结尾。

```bash
$ echo "This is a good book" | sed -n '/book$/p'
This is a good book
$ echo "This book is good" | sed -n '/book$/p'
$
```

使用结尾文本模式的问题在于你必须要留意到底要查找什么。

```bash
$ echo "There are a lot of good books" | sed -n '/book$/p'
$
```

将行尾的单词 book 改成复数形式，就意味着它不再匹配正则表达式模式了，尽管 book 仍然在数据流中。要想匹配，文本模式必须是行的最后一部分。

---

在一些常见情况下，可以在同一行中将行首锚点和行尾锚点组合在一起使用。在第一种情况中，假定你要查找只含有特定文本模式的数据行。

```bash
$ cat data4
this is a test of using both anchors
I said this is a test
this is a test
I'm sure this is a test.
$ sed -n '/^this is a test$/p' data4
this is a test
$
```

sed 编辑器忽略了那些不单单包含指定的文本的行。

第二种情况乍一看可能有些怪异，但极其有用。将两个锚点直接组合在一起，之间不加任何文本，这样过滤出数据流中的空白行。考虑下面这个例子。

```bash
$ cat data5
This is one test line.

This is another test line.
$ sed '/^$/d' data5
This is one test line.
This is another test line.
$
```

定义的正则表达式模式会查找行首和行尾之间什么都没有的那些行。由于空白行在两个换行符之间没有文本，刚好匹配了正则表达式模式。sed 编辑器用删除命令 d 来删除匹配该正则表达式模式的行，因此删除了文本中的所有空白行。这是从文档中删除空白行的有效方法。

### 点号字符

特殊字符点号用来匹配除换行符之外的任意单个字符。它必须匹配一个字符，如果在点号字符的位置没有字符，那么模式就不成立。来看一些在正则表达式模式中使用点号字符的例子。

```bash
$ cat data6
This is a test of a line.
The cat is sleeping.
That is a very nice hat.
This test is at line four.
at ten o'clock we'll go home.
$ sed -n '/.at/p'
data6
The cat is sleeping.
That is a very nice hat.
This test is at line four.
$
```

你应该能够明白为什么第一行无法匹配，而第二行和第三行就可以。第四行有点复杂。注意，我们匹配了 at，但在 at 前面并没有任何字符来匹配点号字符。其实是有的！在正则表达式中，空格也是字符，因此 at 前面的空格刚好匹配了该模式。第五行证明了这点，将 at 放在行首就不会匹配该模式了。

### 字符组

点号特殊字符在匹配某个字符位置上的任意字符时很有用。但如果你想要限定待匹配的具体字符呢？在正则表达式中，这称为`字符组`（character class）。可以定义用来匹配文本模式中某个位置的一组字符。如果字符组中的某个字符出现在了数据流中，那它就匹配了该模式。

使用方括号来定义一个字符组。方括号中包含所有你希望出现在该字符组中的字符。然后你可以在模式中使用整个组，就跟使用其他通配符一样。这需要一点时间来适应，但一旦你适应了，效果可是令人惊叹的。下面是个创建字符组的例子。

```bash
$ sed -n '/[ch]at/p' data6
The cat is sleeping.
That is a very nice hat.
$
```

这里用到的数据文件和点号特殊字符例子中的一样，但得到的结果却不一样。这次我们成功滤掉了只包含单词 at 的行。匹配这个模式的单词只有 cat 和 hat。还要注意以 at 开头的行也没有匹配。字符组中必须有个字符来匹配相应的位置。

在不太确定某个字符的大小写时，字符组会非常有用。

```bash
$ echo "Yes" | sed -n '/[Yy]es/p'
Yes
$ echo "yes" | sed -n '/[Yy]es/p'
yes
$
```

可以在单个表达式中用多个字符组。

```bash
$ echo "Yes" | sed -n '/[Yy][Ee][Ss]/p'
Yes
$ echo "yEs" | sed -n '/[Yy][Ee][Ss]/p'
yEs
$ echo "yeS" | sed -n '/[Yy][Ee][Ss]/p'
yeS
$
```

正则表达式使用了 3 个字符组来涵盖了 3 个字符位置含有大小写的情况。  
字符组不必只含有字母，也可以在其中使用数字。

```bash
$ cat data7
This line doesn't contain a number.
This line has 1 number on it.
This line a number 2 on it.
This line has a number 4 on it.
$ sed -n '/[0123]/p' data7
This line has 1 number on it.
This line a number 2 on it.
$
```

这个正则表达式模式匹配了任意含有数字 0、1、2 或 3 的行。含有其他数字以及不含有数字的行都会被忽略掉。

可以将字符组组合在一起，以检查数字是否具备正确的格式，比如电话号码和邮编。但当你尝试匹配某种特定格式时，必须小心。这里有个匹配邮编出错的例子。

```bash
$ cat data8
60633
46201
223001
4353
22203
$ sed -n '
>/[0123456789][0123456789][0123456789][0123456789][0123456789]/p
>' data8
60633
46201
223001
22203
$
```

这个结果出乎意料。它成功过滤掉了不可能是邮编的那些过短的数字，因为最后一个字符组没有字符可匹配。但它也通过了那个六位数，尽管我们只定义了 5 个字符组。

记住，正则表达式模式可见于数据流中文本的任何位置。经常有匹配模式的字符之外的其他字符。如果要确保只匹配五位数，就必须将匹配的字符和其他字符分开，要么用空格，要么像这个例子中这样，指明它们就在行首和行尾。

```bash
$ sed -n '

> /^[0123456789][0123456789][0123456789][0123456789][0123456789]$/p
> ' data8
60633
46201
22203
$

```

现在好多了！本章随后会看到如何进一步进行简化。

字符组的一个极其常见的用法是解析拼错的单词，比如用户表单输入的数据。你可以创建正则表达式来接受数据中常见的拼写错误。

```bash
$ cat data9
I need to have some maintenence done on my car.
I'll pay that in a seperate invoice.
After I pay for the maintenance my car will be as good as new.
$ sed -n '
/maint[ea]n[ae]nce/p
/sep[ea]r[ea]te/p
' data9
I need to have some maintenence done on my car.
I'll pay that in a seperate invoice.
After I pay for the maintenance my car will be as good as new.
$
```

本例中的两个 sed 打印命令利用正则表达式字符组来帮助找到文本中拼错的单词 maintenance 和 separate。同样的正则表达式模式也能匹配正确拼写的 maintenance。

### 排除型字符组

在正则表达式模式中，也可以反转字符组的作用。可以寻找组中没有的字符，而不是去寻找组中含有的字符。要这么做的话，只要在字符组的开头加个脱字符。

```bash
$ sed -n '/[^ch]at/p' data6
This test is at line four.
$
```

通过排除型字符组，正则表达式模式会匹配 c 或 h 之外的任何字符以及文本模式。由于空格字符属于这个范围，它通过了模式匹配。但即使是排除，字符组仍然必须匹配一个字符，所以以 at 开头的行仍然未能匹配模式。

### 区间

你可能注意到了，我之前演示邮编的例子的时候，必须在每个字符组中列出所有可能的数字，这实在有点麻烦。好在有一种便捷的方法可以让人免受这番劳苦。可以用单破折线符号在字符组中表示字符区间。只需要指定区间的第一个字符、单破折线以及区间的最后一个字符就行了。根据 Linux 系统采用的字符集，正则表达式会包括此区间内的任意字符。现在你可以通过指定数字区间来简化邮编的例子。

```bash
$ sed -n '/^[0-9][0-9][0-9][0-9][0-9]$/p' data8
60633
46201
45902
$
```

这样可是节省了不少的键盘输入！每个字符组都会匹配 0~9 的任意数字。如果字母出现在数据中的任何位置，这个模式都将不成立。

同样的方法也适用于字母。

```bash
$ sed -n '/[c-h]at/p' data6
The cat is sleeping.
That is a very nice hat.
$
```

新的模式[c-h]at 匹配了首字母在字母 c 和字母 h 之间的单词。这种情况下，只含有单词 at 的行将无法匹配该模式。

还可以在单个字符组指定多个不连续的区间。

```bash
$ sed -n '/[a-ch-m]at/p' data6
The cat is sleeping.
That is a very nice hat.
$
```

该字符组允许区间 a~c、h~m 中的字母出现在 at 文本前，但不允许出现 d~g 的字母。

```bash
$ echo "I'm getting too fat." | sed -n '/[a-ch-m]at/p'
$
```

该模式不匹配 fat 文本，因为它没在指定的区间。

### 特殊的字符组

除了定义自己的字符组外，BRE 还包含了一些特殊的字符组，可用来匹配特定类型的字符。下面介绍了可用的 BRE 特殊的字符组。

- [[:alpha:]] 匹配任意字母字符，不管是大写还是小写
- [[:alnum:]] 匹配任意字母数字字符 0~9、A~Z 或 a~z
- [[:blank:]] 匹配空格或制表符
- [[:digit:]] 匹配 0~9 之间的数字
- [[:lower:]] 匹配小写字母字符 a~z
- [[:upper:]] 匹配任意大写字母字符 A~Z
- [[:print:]] 匹配任意可打印字符
- [[:punct:]] 匹配标点符号
- [[:space:]] 匹配任意空白字符：空格、制表符、NL、FF、VT 和 CR

可以在正则表达式模式中将特殊字符组像普通字符组一样使用。

```bash
$ echo "abc" | sed -n '/[[:digit:]]/p'
$
$ echo "abc" | sed -n '/[[:alpha:]]/p'
abc
$ echo "abc123" | sed -n '/[[:digit:]]/p'
abc123
$ echo "This is, a test" | sed -n '/[[:punct:]]/p'
This is, a test
$ echo "This is a test" | sed -n '/[[:punct:]]/p'
$
```

使用特殊字符组可以很方便地定义区间。如可以用[[:digit:]]来代替区间[0-9]。

### 星号

在字符后面放置星号表明该字符必须在匹配模式的文本中出现 0 次或多次。

```bash
$ echo "ik" | sed -n '/ie*k/p'
ik
$ echo "iek" | sed -n '/ie*k/p'
iek
$ echo "ieek" | sed -n '/ie*k/p'
ieek
$ echo "ieeek" | sed -n '/ie*k/p'
ieeek
$ echo "ieeeek" | sed -n '/ie*k/p'
ieeeek
$
```

这个模式符号广泛用于处理有常见拼写错误或在不同语言中有拼写变化的单词。举个例子，如果需要写个可能用在美式或英式英语中的脚本，可以这么写：

```bash
$ echo "I'm getting a color TV" | sed -n '/colou*r/p'
I'm getting a color TV
$ echo "I'm getting a colour TV" | sed -n '/colou*r/p'
I'm getting a colour TV
$
```

模式中的 u\*表明字母 u 可能出现或不出现在匹配模式的文本中。类似地，如果你知道一个单词经常被拼错，你可以用星号来允许这种错误。

```bash
$ echo "I ate a potatoe with my lunch." | sed -n '/potatoe*/p'
I ate a potatoe with my lunch.
$ echo "I ate a potato with my lunch." | sed -n '/potatoe*/p'
I ate a potato with my lunch.
$
```

在可能出现的额外字母后面放个星号将允许接受拼错的单词。

另一个方便的特性是将点号特殊字符和星号特殊字符组合起来。这个组合能够匹配任意数量的任意字符。它通常用在数据流中两个可能相邻或不相邻的文本字符串之间。

```bash
$ echo "this is a regular pattern expression" | sed -n '
> /regular.*expression/p'
this is a regular pattern expression
$
```

可以使用这个模式轻松查找可能出现在数据流中文本行内任意位置的多个单词。

星号还能用在字符组上。它允许指定可能在文本中出现多次的字符组或字符区间。

```bash
$ echo "bt" | sed -n '/b[ae]*t/p'
bt
$ echo "bat" | sed -n '/b[ae]*t/p'
bat
$ echo "bet" | sed -n '/b[ae]*t/p'
bet
$ echo "btt" | sed -n '/b[ae]*t/p'
btt
$
$ echo "baat" | sed -n '/b[ae]*t/p'
baat
$ echo "baaeeet" | sed -n '/b[ae]*t/p'
baaeeet
$ echo "baeeaeeat" | sed -n '/b[ae]*t/p'
baeeaeeat
$ echo "baakeeet" | sed -n '/b[ae]*t/p'
$
```

只要 a 和 e 字符以任何组合形式出现在 b 和 t 字符之间（就算完全不出现也行），模式就能够匹配。如果出现了字符组之外的字符，该模式匹配就会不成立。

## 扩展正则表达式

POSIX ERE 模式包括了一些可供 Linux 应用和工具使用的额外符号。gawk 程序能够识别 ERE 模式，但 sed 编辑器不能。

> 记住，sed 编辑器和 gawk 程序的正则表达式引擎之间是有区别的。gawk 程序可以使用大多数扩展正则表达式模式符号，并且能提供一些额外过滤功能，而这些功能都是 sed 编辑器所不具备的。但正因为如此，gawk 程序在处理数据流时通常才比较慢。

本节将介绍可用在 gawk 程序脚本中的较常见的 ERE 模式符号。

### 问号

问号类似于星号，不过有点细微的不同。问号表明前面的字符可以出现 0 次或 1 次，但只限于此。它不会匹配多次出现的字符。

```bash
$ echo "bt" | gawk '/be?t/{print $0}'
bt
$ echo "bet" | gawk '/be?t/{print $0}'
bet
$ echo "beet" | gawk '/be?t/{print $0}'
$
$ echo "beeet" | gawk '/be?t/{print $0}'
$
```

如果字符 e 并未在文本中出现，或者它只在文本中出现了 1 次，那么模式会匹配。

与星号一样，你可以将问号和字符组一起使用。

```bash
$ echo "bt" | gawk '/b[ae]?t/{print $0}'
bt
$ echo "bat" | gawk '/b[ae]?t/{print $0}'
bat
$ echo "bot" | gawk '/b[ae]?t/{print $0}'
$
$ echo "bet" | gawk '/b[ae]?t/{print $0}'
bet
$ echo "baet" | gawk '/b[ae]?t/{print $0}'
$
$ echo "beat" | gawk '/b[ae]?t/{print $0}'
$
$ echo "beet" | gawk '/b[ae]?t/{print $0}'
$
```

如果字符组中的字符出现了 0 次或 1 次，模式匹配就成立。但如果两个字符都出现了，或者其中一个字符出现了 2 次，模式匹配就不成立。

### 加号

加号是类似于星号的另一个模式符号，但跟问号也有不同。加号表明前面的字符可以出现 1 次或多次，但必须至少出现 1 次。如果该字符没有出现，那么模式就不会匹配。

```bash
$ echo "beeet" | gawk '/be+t/{print $0}'
beeet
$ echo "beet" | gawk '/be+t/{print $0}'
beet
$ echo "bet" | gawk '/be+t/{print $0}'
bet
$ echo "bt" | gawk '/be+t/{print $0}'
$
```

如果字符 e 没有出现，模式匹配就不成立。加号同样适用于字符组，与星号和问号的使用方式相同。

```bash
$ echo "bt" | gawk '/b[ae]+t/{print $0}'
$
$ echo "bat" | gawk '/b[ae]+t/{print $0}'
bat
$ echo "bet" | gawk '/b[ae]+t/{print $0}'
bet
$ echo "beat" | gawk '/b[ae]+t/{print $0}'
beat
$ echo "beet" | gawk '/b[ae]+t/{print $0}'
beet
$ echo "beeat" | gawk '/b[ae]+t/{print $0}'
beeat
$
```

这次如果字符组中定义的任一字符出现了，文本就会匹配指定的模式。

### 使用花括号

ERE 中的花括号允许你为可重复的正则表达式指定一个上限。这通常称为`间隔`（interval）。可以用两种格式来指定区间。

- 正则表达式准确出现 m 次。
- m, n：正则表达式至少出现 m 次，至多 n 次。

这个特性可以精确调整字符或字符集在模式中具体出现的次数。

> 如果你的 gawk 版本过老，gawk 程序不会识别正则表达式间隔。必须额外指定 gawk 程序的--re- interval 命令行选项才能识别正则表达式间隔。

这里有个使用简单的单值间隔的例子。

```bash
$ echo "bt" | gawk --re-interval '/be{1}t/{print $0}'
$
$ echo "bet" | gawk --re-interval '/be{1}t/{print $0}'
bet
$ echo "beet" | gawk --re-interval '/be{1}t/{print $0}'
$
```

通过指定间隔为 1，限定了该字符在匹配模式的字符串中出现的次数。如果该字符出现多次，模式匹配就不成立。

很多时候，同时指定下限和上限也很方便。

```bash
$ echo "bt" | gawk --re-interval '/be{1,2}t/{print $0}'
$
$ echo "bet" | gawk --re-interval '/be{1,2}t/{print $0}'
bet
$ echo "beet" | gawk --re-interval '/be{1,2}t/{print $0}'
beet
$ echo "beeet" | gawk --re-interval '/be{1,2}t/{print $0}'
$
```

在这个例子中，字符 e 可以出现 1 次或 2 次，这样模式就能匹配；否则，模式无法匹配

间隔模式匹配同样适用于字符组。

```bash
$ echo "bt" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
$
$ echo "bat" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
bat
$ echo "bet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
bet
$ echo "beat" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
beat
$ echo "beet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
beet
$ echo "beeat" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
$
$ echo "baeet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
$
$ echo "baeaet" | gawk --re-interval '/b[ae]{1,2}t/{print $0}'
$
```

如果字母 a 或 e 在文本模式中只出现了 1~2 次，则正则表达式模式匹配；否则，模式匹配失败。

### 管道符号

管道符号允许你在检查数据流时，用逻辑 OR 方式指定正则表达式引擎要用的两个或多个模式。如果任何一个模式匹配了数据流文本，文本就通过测试。如果没有模式匹配，则数据流文本匹配失败。

使用管道符号的格式如下：

```bash
expr1|expr2|...
```

这里有个例子。

```bash
$ echo "The cat is asleep" | gawk '/cat|dog/{print $0}'
The cat is asleep
$ echo "The dog is asleep" | gawk '/cat|dog/{print $0}'
The dog is asleep
$ echo "The sheep is asleep" | gawk '/cat|dog/{print $0}'
$
```

这个例子会在数据流中查找正则表达式 cat 或 dog。正则表达式和管道符号之间不能有空格，否则它们也会被认为是正则表达式模式的一部分。  
管道符号两侧的正则表达式可以采用任何正则表达式模式（包括字符组）来定义文本。

```bash
$ echo "He has a hat." | gawk '/[ch]at|dog/{print $0}'
He has a hat.
$
```

这个例子会匹配数据流文本中的 cat、hat 或 dog。

### 圆括号

正则表达式模式也可以用圆括号进行分组。当你将正则表达式模式分组时，该组会被视为一个标准字符。可以像对普通字符一样给该组使用特殊字符。举个例子：

```bash
$ echo "Sat" | gawk '/Sat(urday)?/{print $0}'
Sat
$ echo "Saturday" | gawk '/Sat(urday)?/{print $0}'
Saturday
$
```

结尾的 urday 分组以及问号，使得模式能够匹配完整的 Saturday 或缩写 Sat。  
将分组和管道符号一起使用来创建可能的模式匹配组是很常见的做法。

```bash
$ echo "cat" | gawk '/(c|b)a(b|t)/{print $0}'
cat
$ echo "cab" | gawk '/(c|b)a(b|t)/{print $0}'
cab
$ echo "bat" | gawk '/(c|b)a(b|t)/{print $0}'
bat
$ echo "bab" | gawk '/(c|b)a(b|t)/{print $0}'
bab
$ echo "tab" | gawk '/(c|b)a(b|t)/{print $0}'
$
$ echo "tac" | gawk '/(c|b)a(b|t)/{print $0}'
$
```

模式(c|b)a(b|t)会匹配第一组中字母的任意组合以及第二组中字母的任意组合

## 正则表达式实战

现在你已经了解了使用正则表达式模式的规则和一些简单的例子，该把理论用于实践了。随后几节将会演示 shell 脚本中常见的一些正则表达式例子。

### 目录文件计数

让我们先看一个 shell 脚本，它会对 PATH 环境变量中定义的目录里的可执行文件进行计数。要这么做的话，首先你得将 PATH 变量解析成单独的目录名。前面介绍过如何显示 PATH 环境变量。

```bash
$ echo $PATH /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/ local/games
$
```

根据 Linux 系统上应用程序所处的位置，PATH 环境变量会有所不同。关键是要意识到 PATH 中的每个路径由冒号分隔。要获取可在脚本中使用的目录列表，就必须用空格来替换冒号。现在你会发现 sed 编辑器用一条简单表达式就能完成替换工作。

```bash
$ echo $PATH | sed 's/:/ /g'
/usr/local/sbin /usr/local/bin /usr/sbin /usr/bin /sbin /bin /usr/games /usr/local/games
$
```

分离出目录之后，你就可以使用标准 for 语句中来遍历每个目录。

```bash
mypath=$(echo $PATH | sed 's/:/ /g')
for directory in $mypath
do
    ...
done
```

一旦获得了单个目录，就可以用 ls 命令来列出每个目录中的文件，并用另一个 for 语句来遍历每个文件，为文件计数器增值。  
这个脚本的最终版本如下。

```bash
$ cat countfiles
#!/bin/bash
# count number of files in your PATH
mypath=$(echo $PATH | sed 's/:/ /g')
count=0
for directory in $mypath
do
    check=$(ls $directory)
    for item in $check
        do
            count=$[ $count + 1 ]
        done
    echo "$directory - $count"
    count=0
done
```

```
$ ./countfiles
/usr/local/sbin - 0
/usr/local/bin - 2
/usr/sbin - 213
/usr/bin - 1427
/sbin - 186
/bin - 152
/usr/games - 5
/usr/local/games – 0
$
```

现在我们开始体会到正则表达式背后的强大之处了！

### 验证电话号码

前面的例子演示了在处理数据时，如何将简单的正则表达式和 sed 配合使用来替换数据流中的字符。正则表达式通常用于验证数据，确保脚本中数据格式的正确性。  
一个常见的数据验证应用就是检查电话号码。数据输入表单通常会要求填入电话号码，而用户输入格式错误的电话号码是常有的事。在美国，电话号码有几种常见的形式：

```
(123)456-7890
(123) 456-7890
123-456-7890
123.456.7890
```

这样用户在表单中输入的电话号码就有 4 种可能。正则表达式必须足够强大，才能处理每一种情况。

在构建正则表达式时，最好从左手边开始，然后构建用来匹配可能遇到的字符的模式。在这个例子中，电话号码中可能有也可能没有左圆括号。这可以用如下模式来匹配：

```bash
^\(?
```

脱字符用来表明数据的开始。由于左圆括号是个特殊字符，因此必须将它转义成普通字符。问号表明左圆括号可能出现，也可能不出现。  
紧接着就是 3 位区号。在美国，区号以数字 2 开始（没有以数字 0 或 1 开始的区号），最大可到 9。要匹配区号，可以用如下模式。

```bash
[2-9][0-9]{2}
```

这要求第一个字符是 2~9 的数字，后跟任意两位数字。在区号后面，收尾的右圆括号可能存在，也可能不存在。

```bash
\)?
```

在区号后，存在如下可能：有一个空格，没有空格，有一条单破折线或一个点。你可以对它们使用管道符号，并用圆括号进行分组。

```bash
(| |-|\.)
```

第一个管道符号紧跟在左圆括号后，用来匹配没有空格的情形。你必须将点字符转义，否则它会被解释成可匹配任意字符。  
紧接着是 3 位电话交换机号码。这里没什么需要特别注意的。

```bash
[0-9]{3}
```

在电话交换机号码之后，你必须匹配一个空格、一条单破折线或一个点。

```bash
( |-|\.)
```

最后，必须在字符串尾部匹配 4 位本地电话分机号。

```bash
[0-9]{4}$
```

完整的模式如下。

```bash
^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}$
```

你可以在 gawk 程序中用这个正则表达式模式来过滤掉不符合格式的电话号码。现在你只需要在 gawk 程序中创建一个使用该正则表达式的简单脚本，然后用这个脚本来过滤你的电话薄。脚本如下,可以将电话号码重定向到脚本来处理。

```bash
$ cat isphone
#!/bin/bash
# script to filter out bad phone numbers
gawk --re-interval '/^\(?[2-9][0-9]{2}\)?(| |-|\.)[0-9]{3}( |-|\.)[0-9]{4}$/{print $0}'
$
```

```bash
$ echo "317-555-1234" | ./isphone
317-555-1234
$ echo "000-555-1234" | ./isphone
$ echo "312 555-1234" | ./isphone
312 555-1234
$
```

或者也可以将含有电话号码的整个文件重定向到脚本来过滤掉无效的号码。

```bash
$ cat phonelist
000-000-0000
123-456-7890
212-555-1234
(317)555-1234
(202) 555-9876
33523
1234567890
234.123.4567
$ cat phonelist | ./isphone
212-555-1234
(317)555-1234
(202) 555-9876
234.123.4567
$
```

只有匹配该正则表达式模式的有效电话号码才会出现。

### 解析邮件地址

如今这个时代，电子邮件地址已经成为一种重要的通信方式。验证邮件地址成为脚本程序员的一个不小的挑战，因为邮件地址的形式实在是千奇百怪。邮件地址的基本格式为：

```bash
username@hostname
```

username 值可用字母数字字符以及以下特殊字符：

- 点号
- 单破折线
- 加号
- 下划线

在有效的邮件用户名中，这些字符可能以任意组合形式出现。邮件地址的 hostname 部分由一个或多个域名和一个服务器名组成。服务器名和域名也必须遵照严格的命名规则，只允许字母数字字符以及以下特殊字符：

- 点号
- 下划线

服务器名和域名都用点分隔，先指定服务器名，紧接着指定子域名，最后是后面不带点号的顶级域名。  
顶级域名的数量在过去十分有限，正则表达式模式编写者会尝试将它们都加到验证模式中。然而遗憾的是，随着互联网的发展，可用的顶级域名也增多了。这种方法已经不再可行。  
从左侧开始构建这个正则表达式模式。我们知道，用户名中可以有多个有效字符。这个相当容易。

```bash
^([a-zA-Z0-9_\-\.\+]+)@
```

这个分组指定了用户名中允许的字符，加号表明必须有至少一个字符。下一个字符很明显是@，没什么意外的。

hostname 模式使用同样的方法来匹配服务器名和子域名。

```bash
([a-zA-Z0-9_\-\.]+)
```

这个模式可以匹配文本:

```
server
server.subdomain
server.subdomain.subdomain
```

对于顶级域名，有一些特殊的规则。顶级域名只能是字母字符，必须不少于二个字符（国家或地区代码中使用），并且长度上不得超过五个字符。下面就是顶级域名用的正则表达式模式。

```bash
\.([a-zA-Z]{2,5})$
```

将整个模式放在一起会生成如下模式。

```bash
^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$
```

这个模式会从数据列表中过滤掉那些格式不正确的邮件地址。现在可以创建脚本来实现这个正则表达式了。

```bash
$ echo "rich@here.now" | ./isemail
rich@here.now
$ echo "rich@here.now." | ./isemail
$
$
echo "rich@here.n" | ./isemail
$
$ echo "rich@here-now" | ./isemail
$
$ echo "rich.blum@here.now" | ./isemail
rich.blum@here.now
$ echo "rich_blum@here.now" | ./isemail
rich_blum@here.now
$ echo "rich/blum@here.now" | ./isemail
$
$ echo "rich#blum@here.now" | ./isemail
$
$ echo "rich*blum@here.now" | ./isemail
$
```
