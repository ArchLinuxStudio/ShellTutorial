# 初识 sed 和 gawk

到目前为止， shell 脚本最常见的一个用途就是处理文本文件。检查日志文件、读取配置文件、处理数据元素，shell 脚本可以帮助我们将文本文件中各种数据的日常处理任务自动化。但仅靠 shell 脚本命令来处理文本文件的内容有点勉为其难。如果想在 shell 脚本中处理任何类型的数据，你得熟悉 Linux 中的 sed 和 gawk 工具。这两个工具能够极大简化需要进行的数据处理任务。

大多数情况，我们使用 vim 这类编辑器处理文本，但有时候，你会发现需要自动处理文本文件，可你又不想动用全副武装的交互式文本编辑器。在这种情况下，有个能够轻松实现自动格式化、插入、修改或删除文本元素的简单命令行编辑器就方便多了。

Linux 系统提供了两个常见的具备上述功能的工具。本节将会介绍 Linux 世界中最广泛使用的两个命令行编辑器：sed 和 gawk。

## sed 编辑器

sed 编辑器被称作`流编辑器`（stream editor），和普通的交互式文本编辑器恰好相反。在交互式文本编辑器中（比如 vim），你可以用键盘命令来交互式地插入、删除或替换数据中的文本。流编辑器则会在编辑器处理数据之前基于预先提供的一组规则来编辑数据流。

sed 编辑器可以根据命令来处理数据流中的数据，这些命令要么从命令行中输入，要么存储在一个命令文本文件中。sed 编辑器会执行下列操作。

- 一次从输入中读取一行数据。
- 根据所提供的编辑器命令匹配数据。
- 按照命令修改流中的数据。
- 将新的数据输出到 STDOUT。

在流编辑器将所有命令与一行数据匹配完毕后，它会读取下一行数据并重复这个过程。在流编辑器处理完流中的所有数据行后，它就会终止。

由于命令是按顺序逐行给出的，sed 编辑器只需对数据流进行一遍处理就可以完成编辑操作。这使得 sed 编辑器要比交互式编辑器快得多，你可以快速完成对数据的自动修改。

sed 命令的格式如下。

```bash
sed options script file
```

选项允许你修改 sed 命令的行为，可以使用的选项已在下面列出

- -e script 在处理输入时，将 script 中指定的命令添加到已有的命令中
- -f file 在处理输入时，将 file 中指定的命令添加到已有的命令中
- -n 不产生命令输出，使用 print 命令来完成输出

script 参数指定了应用于流数据上的单个命令。如果需要用多个命令，要么使用-e 选项在命令行中指定，要么使用-f 选项在单独的文件中指定。有大量的命令可用来处理数据。我们将会在本章后面介绍一些 sed 编辑器的基本命令，然后在后续章节中会看到另外一些高级命令。

### 在命令行定义编辑器命令

默认情况下，sed 编辑器会将指定的命令应用到 STDIN 输入流上。这样你可以直接将数据通过管道输入 sed 编辑器处理。这里有个简单的示例。

```bash
$ echo "This is a test" | sed 's/test/big test/'
This is a big test
$
```

这个例子在 sed 编辑器中使用了 s 命令。s 命令会用斜线间指定的第二个文本字符串来替换第一个文本字符串模式。在本例中是 big test 替换了 test。
在运行这个例子时，结果应该立即就会显示出来。这就是使用 sed 编辑器的强大之处。你可以同时对数据做出多处修改，而所消耗的时间却只够一些交互式编辑器启动而已。
当然，这个简单的测试只是修改了一行数据。不过就算编辑整个文件，处理速度也相差无几。

```bash
$ cat data1.txt
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
$
$ sed 's/dog/cat/' data1.txt
The quick brown fox jumps over the lazy cat.
The quick brown fox jumps over the lazy cat.
The quick brown fox jumps over the lazy cat.
The quick brown fox jumps over the lazy cat.
$
```

sed 命令几乎瞬间就执行完并返回数据。在处理每行数据的同时，结果也显示出来了。可以在 sed 编辑器处理完整个文件之前就开始观察结果。
重要的是，要记住，sed 编辑器并不会修改文本文件的数据。它只会将修改后的数据发送到 STDOUT。如果你查看原来的文本文件，它仍然保留着原始数据。

### 在命令行使用多个编辑器命令

要在 sed 命令行上执行多个命令时，只要用-e 选项就可以了。

```bash
$ sed -e 's/brown/green/; s/dog/cat/' data1.txt
The quick green fox jumps over the lazy cat.
The quick green fox jumps over the lazy cat.
The quick green fox jumps over the lazy cat.
The quick green fox jumps over the lazy cat.
```

两个命令都作用到文件中的每行数据上。命令之间必须用分号隔开，并且在命令末尾和分号之间不能有空格。

如果不想用分号，也可以用 bash shell 中的次提示符来分隔命令。只要输入第一个单引号标示出 sed 程序脚本的起始（sed 编辑器命令列表），bash 会继续提示你输入更多命令，直到输入了标示结束的单引号。

```bash
$ sed -e '
> s/brown/green/
> s/fox/elephant/
> s/dog/cat/' data1.txt
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
$
```

必须记住，要在封尾单引号所在行结束命令。bash shell 一旦发现了封尾的单引号，就会执行命令。开始后，sed 命令就会将你指定的每条命令应用到文本文件中的每一行上。

### 从文件中读取编辑器命令

最后，如果有大量要处理的 sed 命令，那么将它们放进一个单独的文件中通常会更方便一些。可以在 sed 命令中用-f 选项来指定文件。

```bash
$ cat script1.sed
s/brown/green/
s/fox/elephant/
s/dog/cat/
$
$ sed -f script1.sed data1.txt
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
The quick green elephant jumps over the lazy cat.
$
```

在这种情况下，不用在每条命令后面放一个分号。sed 编辑器知道每行都是一条单独的命令。跟在命令行输入命令一样，sed 编辑器会从指定文件中读取命令，并将它们应用到数据文件中的每一行上。

> 我们很容易就会把 sed 编辑器脚本文件与 bash shell 脚本文件搞混。为了避免这种情况，可以使用.sed 作为 sed 脚本文件的扩展名。

### 更多的替换选项

你已经懂得了如何用 s 命令（substitute）来在行中替换文本。这个命令还有另外一些选项能让事情变得更为简单。关于替换命令如何替换字符串中所匹配的模式需要注意一点。看看下面这个例子中会出现什么情况。

```bash
$ cat data4.txt
This is a test of the test script.
This is the second test of the test script.
$
$ sed 's/test/trial/' data4.txt
This is a trial of the test script.
This is the second trial of the test script.
$
```

替换命令在替换多行中的文本时能正常工作，但默认情况下它只替换每行中出现的第一处。要让替换命令能够替换一行中不同地方出现的文本必须使用`替换标记`（substitution flag）。替换标记会在替换命令字符串之后设置。

```bash
s/pattern/replacement/flags
```

有 4 种可用的替换标记：

- 数字，表明新文本将替换第几处模式匹配的地方；
- g，表明新文本将会替换所有匹配的文本；
- p，表明原先行的内容要打印出来；
- w file，将替换的结果写到文件中。

在第一类替换中，可以指定 sed 编辑器用新文本替换第几处模式匹配的地方。

```bash
$ sed 's/test/trial/2' data4.txt
This is a test of the trial script.
This is the second test of the trial script.
$
```

将替换标记指定为 2 的结果就是：sed 编辑器只替换每行中第二次出现的匹配模式。而 g 替换标记使你能替换文本中匹配模式所匹配的每处地方。p 替换标记会打印与替换命令中指定的模式匹配的行。这通常会和 sed 的-n 选项一起使用。

```bash
$ cat data5.txt
This is a test line.
This is a different line.
$
$ sed -n 's/test/trial/p' data5.txt
This is a trial line.
$
```

-n 选项将禁止 sed 编辑器输出。但 p 替换标记会输出修改过的行。将二者配合使用的效果就是只输出被替换命令修改过的行。

w 替换标记会产生同样的输出，不过会将输出保存到指定文件中。

```bash
$ sed 's/test/trial/w test.txt' data5.txt
This is a trial line.
This is a different line.
$
$ cat test.txt
This is a trial line.
$
```

sed 编辑器的正常输出是在 STDOUT 中，而只有那些包含匹配模式的行才会保存在指定的输出文件中。

---

有时你会在文本字符串中遇到一些不太方便在替换模式中使用的字符。Linux 中一个常见的例子就是正斜线（/）。替换文件中的路径名会比较麻烦。比如，如果想用 C shell 替换/etc/passwd 文件中的 bash shell，必须这么做：

```bash
$ sed 's/\/bin\/bash/\/bin\/csh/' /etc/passwd
```

由于正斜线通常用作字符串分隔符，因而如果它出现在了模式文本中的话，必须用反斜线来转义。这通常会带来一些困惑和错误。要解决这个问题，sed 编辑器允许选择其他字符来作为替换命令中的字符串分隔符：

```bash
$ sed 's!/bin/bash!/bin/csh!' /etc/passwd
```

在这个例子中，感叹号被用作字符串分隔符，这样路径名就更容易阅读和理解了

### 使用地址

默认情况下，在 sed 编辑器中使用的命令会作用于文本数据的所有行。如果只想将命令作用于特定行或某些行，则必须用`行寻址`（line addressing）。在 sed 编辑器中有两种形式的行寻址：

- 以数字形式表示行区间
- 用文本模式来过滤出行

两种形式都使用相同的格式来指定地址：

```bash
[address]command
```

也可以将特定地址的多个命令分组：

```bash
address {
    command1
    command2
    command3
}
```

sed 编辑器会将指定的每条命令作用到匹配指定地址的行上。下面将会演示如何在 sed 编辑器脚本中使用两种寻址方法。

当使用数字方式的行寻址时，可以用行在文本流中的行位置来引用。sed 编辑器会将文本流中的第一行编号为 1，然后继续按顺序为接下来的行分配行号。在命令中指定的地址可以是单个行号，或是用起始行号、逗号以及结尾行号指定的一定区间范围内的行。这里有个 sed 命令作用到指定行号的例子。

```bash
$ sed '2s/dog/cat/' data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy dog
$
```

sed 编辑器只修改地址指定的第二行的文本。这里有另一个例子，这次使用了行地址区间。

```bash
$ sed '2,3s/dog/cat/' data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy dog
$
```

如果想将命令作用到文本中从某行开始的所有行，可以用特殊地址——美元符。可能你并不知道文本中到底有多少行数据，因此美元符用起来通常很方便。

```bash
$ sed '2,$s/dog/cat/' data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy cat
The quick brown fox jumps over the lazy cat
$
```

---

另一种限制命令作用到哪些行上的方法会稍稍复杂一些。sed 编辑器允许指定文本模式来过滤出命令要作用的行。格式如下：

```bash
/pattern/command
```

必须用正斜线将要指定的 pattern 封起来。sed 编辑器会将该命令作用到包含指定文本模式的行上。举个例子，如果你想只修改用户 Samantha 的默认 shell，可以使用 sed 命令。

```bash
$ grep Samantha /etc/passwd
Samantha:x:502:502::/home/Samantha:/bin/bash
$
$ sed '/Samantha/s/bash/csh/' /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
[...]
Christine:x:501:501:Christine B:/home/Christine:/bin/bash
Samantha:x:502:502::/home/Samantha:/bin/csh
Timothy:x:503:503::/home/Timothy:/bin/bash
$
```

该命令只作用到匹配文本模式的行上。虽然使用固定文本模式能帮你过滤出特定的值，就跟上面这个用户名的例子一样，但其作用难免有限。sed 编辑器在文本模式中采用了一种称为`正则表达式`（regular expression）的特性来帮助你创建匹配效果更好的模式。  
正则表达式允许创建高级文本模式匹配表达式来匹配各种数据。这些表达式结合了一系列通配符、特殊字符以及固定文本字符来生成能够匹配几乎任何形式文本的简练模式。正则表达式是 shell 脚本编程中令人心生退意的部分之一，下一章将会详细介绍相关内容。

---

如果需要在单行上执行多条命令，可以用花括号将多条命令组合在一起。sed 编辑器会处理地址行处列出的每条命令。

```bash
$ sed '2{
> s/fox/elephant/
> s/dog/cat/
> }' data1.txt
The quick brown fox jumps over the lazy dog.
The quick brown elephant jumps over the lazy cat.
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
$
```

两条命令都会作用到该地址上。当然，也可以在一组命令前指定一个地址区间。sed 编辑器会将所有命令作用到该地址区间内的所有行上。

```bash
$ sed '3,${
> s/brown/green/
> s/lazy/active/
> }' data1.txt
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
The quick green fox jumps over the active dog.
The quick green fox jumps over the active dog.
$
```

### 删除行

文本替换命令不是 sed 编辑器唯一的命令。如果需要删除文本流中的特定行，可以用删除命令。删除命令 d 名副其实，它会删除匹配指定寻址模式的所有行。使用该命令时要特别小心，如果你忘记加入寻址模式的话，流中的所有文本行都会被删除。

```bash
$ cat data1.txt
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy dog
The quick brown fox jumps over the lazy dog
$
$ sed 'd' data1.txt
$
```

当和指定地址一起使用时，删除命令显然能发挥出最大的功用。可以从数据流中删除特定的文本行，通过行号指定：

```bash
$ cat data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
$
$ sed '3d' data6.txt
This is line number 1.
This is line number 2.
This is line number 4.
$
```

或者通过特定行区间指定：

```bash
$ sed '2,3d' data6.txt
This is line number 1.
This is line number 4.
$
```

或者通过特殊的文件结尾字符：

```bash
$ sed '3,$d' data6.txt
This is line number 1.
This is line number 2.
$
```

sed 编辑器的模式匹配特性也适用于删除命令。sed 编辑器会删掉包含匹配指定模式的行。

```bash
$ sed '/number 1/d' data6.txt
This is line number 2.
This is line number 3.
This is line number 4.
$
```

> 记住，sed 编辑器不会修改原始文件。你删除的行只是从 sed 编辑器的输出中消失了。原始文件仍然包含那些“删掉的”行。

也可以使用两个文本模式来删除某个区间内的行，但这么做时要小心。你指定的第一个模式会“打开”行删除功能，第二个模式会“关闭”行删除功能。sed 编辑器会删除两个指定行之间的所有行（包括指定的行）。

```bash
$ sed '/1/,/3/d' data6.txt
This is line number 4.
$
```

除此之外，你要特别小心，因为只要 sed 编辑器在数据流中匹配到了开始模式，删除功能就会打开。这可能会导致意外的结果。

```bash
$ cat data7.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
This is line number 1 again.
This is text you want to keep.
This is the last line in the file.
$
$ sed '/1/,/3/d' data7.txt
This is line number 4.
$
```

第二个出现数字“1”的行再次触发了删除命令，因为没有找到停止模式，所以就将数据流中的剩余行全部删除了。当然，如果你指定了一个从未在文本中出现的停止模式，显然会出现另外一个问题。

```bash
$ sed '/1/,/5/d' data7.txt
$
```

因为删除功能在匹配到第一个模式的时候打开了，但一直没匹配到结束模式，所以整个数据流都被删掉了。

### 插入和附加文本

如你所期望的，跟其他编辑器类似，sed 编辑器允许向数据流插入和附加文本行。两个操作的区别可能比较让人费解：

- 插入（insert）命令（i）会在指定行前增加一个新行；
- 附加（append）命令（a）会在指定行后增加一个新行。

这两条命令的费解之处在于它们的格式。它们不能在单个命令行上使用。你必须指定是要将行插入还是附加到另一行。格式如下：

```bash
sed '[address]command\
new line'
```

new line 中的文本将会出现在 sed 编辑器输出中你指定的位置。记住，当使用插入命令时，文本会出现在数据流文本的前面。

```bash
$ echo "Test Line 2" | sed 'i\Test Line 1'
Test Line 1
Test Line 2
$
```

当使用附加命令时，文本会出现在数据流文本的后面。

```bash
$ echo "Test Line 2" | sed 'a\Test Line 1'
Test Line 2
Test Line 1
$
```

在命令行界面提示符上使用 sed 编辑器时，你会看到次提示符来提醒输入新的行数据。你必须在该行完成 sed 编辑器命令。一旦你输入了结尾的单引号，bash shell 就会执行该命令。

```bash
$ echo "Test Line 2" | sed 'i\
> Test Line 1'
Test Line 1
Test Line 2
$
```

这样能够给数据流中的文本前面或后面添加文本，但如果要向数据流内部添加文本呢？要向数据流行内部插入或附加数据，你必须用寻址来告诉 sed 编辑器你想让数据出现在什么位置。可以在用这些命令时只指定一个行地址。可以匹配一个数字行号或文本模式，但不能用地址区间。这合乎逻辑，因为你只能将文本插入或附加到单个行的前面或后面，而不是行区间的前面或后面。下面的例子是将一个新行插入到数据流第三行前。

```bash
$ sed '3i\
> This is an inserted line.' data6.txt
This is line number 1.
This is line number 2.
This is an inserted line.
This is line number 3.
This is line number 4.
$
```

下面的例子是将一个新行附加到数据流中第三行后。

```bash
$ sed '3a\
> This is an inserted line.' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is an inserted line.
This is line number 4.
$
```

它使用与插入命令相同的过程，只是将新文本行放到了指定的行号后面。如果你有一个多行数据流，想要将新行附加到数据流的末尾，只要用代表数据最后一行的美元符就可以了。

```bash
$ sed '$a\
> This is an inserted line.' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
This is an inserted line.
$
```

同样的方法也适用于要在数据流起始位置增加一个新行。只要在第一行之前插入新行即可。  
要插入或附加多行文本，就必须对要插入或附加的新文本中的每一行使用反斜线，直到最后一行。

```bash
$ sed '1i\
> This is one line of new text.\
> This is another line of new text.' data6.txt
This is one line of new text.
This is another line of new text.
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
$
```

指定的两行都会被添加到数据流中。

### 修改行

修改（change）命令允许修改数据流中整行文本的内容。它跟插入和附加命令的工作机制一样，你必须在 sed 命令中单独指定新行。

```bash
$ sed '3c\
> This is a changed line of text.' data6.txt
This is line number 1.
This is line number 2.
This is a changed line of text.
This is line number 4.
$
```

在这个例子中，sed 编辑器会修改第三行中的文本。也可以用文本模式来寻址。

```bash
$ sed '/number 3/c\
> This is a changed line of text.' data6.txt
This is line number 1.
This is line number 2.
This is a changed line of text.
This is line number 4.
$
```

文本模式修改命令会修改它匹配的数据流中的任意文本行。

```bash
$ cat data8.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
This is line number 1 again.
This is yet another line.
This is the last line in the file.
$
$ sed '/number 1/c\
> This is a changed line of text.' data8.txt
This is a changed line of text.
This is line number 2.
This is line number 3.
This is line number 4.
This is a changed line of text.
This is yet another line.
This is the last line in the file.
$
```

你可以在修改命令中使用地址区间，但结果未必如愿。

```bash
$ sed '2,3c\
> This is a new line of text.' data6.txt
This is line number 1.
This is a new line of text.
This is line number 4.
$
```

sed 编辑器会用这一行文本来替换数据流中的两行文本，而不是逐一修改这两行文本。

### 转换命令

转换（transform）命令（y）是唯一可以处理单个字符的 sed 编辑器命令。转换命令格式如下。

```bash
[address]y/inchars/outchars/
```

转换命令会对 inchars 和 outchars 值进行一对一的映射。inchars 中的第一个字符会被转换为 outchars 中的第一个字符，第二个字符会被转换成 outchars 中的第二个字符。这个映射过程会一直持续到处理完指定字符。如果 inchars 和 outchars 的长度不同，则 sed 编辑器会产生一条错误消息。这里有个使用转换命令的简单例子。

```bash
$ sed 'y/123/789/' data8.txt
This is line number 7.
This is line number 8.
This is line number 9.
This is line number 4.
This is line number 7 again.
This is yet another line.
This is the last line in the file.
$
```

如你在输出中看到的，inchars 模式中指定字符的每个实例都会被替换成 outchars 模式中相同位置的那个字符。转换命令是一个全局命令，也就是说，它会文本行中找到的所有指定字符自动进行转换，而不会考虑它们出现的位置。

```bash
$ echo "This 1 is a test of 1 try." | sed 'y/123/456/'
This 4 is a test of 4 try.
$
```

sed 编辑器转换了在文本行中匹配到的字符 1 的两个实例。你无法限定只转换在特定地方出现的字符。

### 回顾打印

之前介绍了如何使用 p `标记`和替换命令显示 sed 编辑器修改过的行。另外有 3 个命令也能用来打印数据流中的信息：

- p 命令用来打印文本行；
- 等号（=）命令用来打印行号；
- l（小写的 L）命令用来列出行。

---

跟替换命令中的 p `标记`类似，p `命令`可以打印 sed 编辑器输出中的一行。如果只用这个命令，也没什么特别的。

```bash
$ echo "this is a test" | sed 'p'
this is a test
this is a test
$
```

它所做的就是打印已有的数据文本。打印命令最常见的用法是打印包含匹配文本模式的行。

```bash
$ cat data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
$
$ sed -n '/number 3/p' data6.txt
This is line number 3.
$
```

在命令行上用-n 选项，你可以禁止输出其他行，只打印包含匹配文本模式的行。也可以用它来快速打印数据流中的某些行。

```bash
$ sed -n '2,3p' data6.txt
This is line number 2.
This is line number 3.
$
```

如果需要在修改之前查看行，也可以使用打印命令，比如与替换或修改命令一起使用。可以创建一个脚本在修改行之前显示该行。

```bash
$ sed -n '/3/{
> p
> s/line/test/p
> }' data6.txt
This is line number 3.
This is test number 3.
$
```

sed 编辑器命令会查找包含数字 3 的行，然后执行两条命令。首先，脚本用 p 命令来打印出原始行；然后它用 s 命令替换文本，并用 p 标记打印出替换结果。输出同时显示了原来的行文本和新的行文本。

---

等号命令会打印行在数据流中的当前行号。行号由数据流中的换行符决定。每次数据流中出现一个换行符，sed 编辑器会认为一行文本结束了。

```bash
$ cat data1.txt
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
The quick brown fox jumps over the lazy dog.
$
$ sed '=' data1.txt
1
The quick brown fox jumps over the lazy dog.
2
The quick brown fox jumps over the lazy dog.
3
The quick brown fox jumps over the lazy dog.
4
The quick brown fox jumps over the lazy dog.
$
```

sed 编辑器在实际的文本行出现前打印了行号。如果你要在数据流中查找特定文本模式的话，等号命令用起来非常方便。

```bash
$ sed -n '/number 4/{
> =
> p
> }' data6.txt
4
This is line number 4.
$
```

利用-n 选项，你就能让 sed 编辑器只显示包含匹配文本模式的行的行号和文本。

---

列出（list）命令（l）可以打印数据流中的文本和不可打印的 ASCII 字符。任何不可打印字符要么在其八进制值前加一个反斜线，要么使用标准 C 风格的命名法（用于常见的不可打印字符），比如\t，来代表制表符。

```bash
$ cat data9.txt
This    line    contains        tabs.
$
$ sed -n 'l' data9.txt
This\tline\tcontains\ttabs.$
$
```

制表符的位置使用\t 来显示。行尾的美元符表示换行符。如果数据流包含了转义字符，列出命令会在必要时候用八进制码来显示。

```bash
$ cat data10.txt
This line contains an escape character.
$
$ sed -n 'l' data10.txt
This line contains an escape character. \a$
$
```

data10.txt 文本文件包含了一个转义控制码来产生铃声。当用 cat 命令来显示文本文件时，你看不到转义控制码，只能听到声音（如果你的音箱打开的话）。但是，利用列出命令，你就能显示出所使用的转义控制码。

### 使用 sed 处理文件

替换命令包含一些可以用于文件的标记。还有一些 sed 编辑器命令也可以实现同样的目标，不需要非得替换文本。

w 命令用来向文件写入行。该命令的格式如下：

```bash
[address]w filename0
```

filename 可以使用相对路径或绝对路径，但不管是哪种，运行 sed 编辑器的用户都必须有文件的写权限。地址可以是 sed 中支持的任意类型的寻址方式，例如单个行号、文本模式、行区间或文本模式。

下面的例子是将数据流中的前两行打印到一个文本文件中。

```bash
$ sed '1,2w test.txt' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
$
$ cat test.txt
This is line number 1.
This is line number 2.
$
```

当然，如果你不想让行显示到 STDOUT 上，你可以用 sed 命令的-n 选项。  
如果要根据一些公用的文本值从主文件中创建一份数据文件，比如下面的邮件列表中的，那么 w 命令会非常好用。

```bash
$ cat data11.txt
Blum, R       Browncoat
McGuiness, A  Alliance
Bresnahan, C  Browncoat
Harken, C     Alliance
$
$ sed -n '/Browncoat/w Browncoats.txt' data11.txt
$
$ cat Browncoats.txt
Blum, R       Browncoat
Bresnahan, C  Browncoat
$
```

sed 编辑器会只将包含文本模式的数据行写入目标文件。

---

你已经了解了如何在 sed 命令行上向数据流中插入或附加文本。读取（read）命令（r）允许你将一个独立文件中的数据插入到数据流中。读取命令的格式如下：

```bash
[address]r filename
```

filename 参数指定了数据文件的绝对路径或相对路径。你在读取命令中使用地址区间，只能指定单独一个行号或文本模式地址。sed 编辑器会将文件中的文本插入到指定地址后。

```bash
$ cat data12.txt
This is an added line.
This is the second added line.
$
$ sed '3r data12.txt' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is an added line.
This is the second added line.
This is line number 4.
$
```

sed 编辑器会将数据文件中的所有文本行都插入到数据流中。同样的方法在使用文本模式地址时也适用。

```bash
$ sed '/number 2/r data12.txt' data6.txt
This is line number 1.
This is line number 2.
This is an added line.
This is the second added line.
This is line number 3.
This is line number 4.
$
```

如果你要在数据流的末尾添加文本，只需用美元符地址符就行了。

```bash
$ sed '$r data12.txt' data6.txt
This is line number 1.
This is line number 2.
This is line number 3.
This is line number 4.
This is an added line.
This is the second added line.
$
```

读取命令的另一个很酷的用法是和删除命令配合使用：利用另一个文件中的数据来替换文件中的占位文本。举例来说，假定你有一份套用信件保存在文本文件中：

```bash
$ cat notice.std
Would the following people:
LIST
please report to the ship's captain.
$
```

套用信件将通用占位文本 LIST 放在人物名单的位置。要在占位文本后插入名单，只需读取命令就行了。但这样的话，占位文本仍然会留在输出中。要删除占位文本的话，你可以用删除命令。结果如下：

```bash
$ sed '/LIST/{
> r data11.txt
> d
> }' notice.std
Would the following people:
Blum, R       Browncoat
McGuiness, A  Alliance
Bresnahan, C  Browncoat
Harken, C     Alliance
please report to the ship's captain.
$
```

现在占位文本已经被替换成了数据文件中的名单。

## gawk 程序

虽然 sed 编辑器是非常方便自动修改文本文件的工具，但其也有自身的限制。通常你需要一个用来处理文件中的数据的更高级工具，它能提供一个类编程环境来修改和重新组织文件中的数据。这正是 gawk 能够做到的。

> 很多发行版中都没有默认安装 gawk 程序。如果你所用的 Linux 发行版中没有包含 gawk，请自行安装。arch linux 可直接安装包组 base-devel 来使用 gawk

gawk 程序是 Unix 中的原始 awk 程序的 GNU 版本。gawk 程序让流编辑迈上了一个新的台阶，它提供了一种编程语言而不只是编辑器命令。在 gawk 编程语言中，你可以做下面的事情：

- 定义变量来保存数据；
- 使用算术和字符串操作符来处理数据；
- 使用结构化编程概念（比如 if-then 语句和循环）来为数据处理增加处理逻辑；
- 通过提取数据文件中的数据元素，将其重新排列或格式化，生成格式化报告。

gawk 程序的报告生成能力通常用来从大文本文件中提取数据元素，并将它们格式化成可读的报告。其中最完美的例子是格式化日志文件。在日志文件中找出错误行会很难，gawk 程序可以让你从日志文件中过滤出需要的数据元素，然后你可以将其格式化，使得重要的数据更易于阅读。gawk 的强大之处在于程序脚本。可以写脚本来读取文本行的数据，然后处理并显示数据，创建任何类型的输出报告。

### 基础用法

gawk 程序的基本格式如下：

```bash
gawk options program file
```

如下显示了 gawk 程序的可用选项。命令行选项提供了一个简单的途径来定制 gawk 程序中的功能。我们会在探索 gawk 时进一步了解这些选项。

- -F fs 指定行中划分数据字段的字段分隔符
- -f file 从指定的文件中读取程序
- -v var=value 定义 gawk 程序中的一个变量及其默认值
- -mf N 指定要处理的数据文件中的最大字段数
- -mr N 指定数据文件中的最大数据行数
- -W keyword 指定 gawk 的兼容模式或警告等级

gawk 程序脚本用一对花括号来定义。你必须将脚本命令放到两个花括号（{}）中。如果你错误地使用了圆括号来包含 gawk 脚本，就会得到一条类似于下面的错误提示。

```bash
$ gawk '(print "Hello World!"}'
gawk: (print "Hello World!"}
gawk:  ^ syntax error
```

由于 gawk 命令行假定脚本是单个文本字符串，你还必须将脚本放到单引号中。下面的例子在命令行上指定了一个简单的 gawk 程序脚本：

```bash
$ gawk '{print "Hello World!"}'
```

这个程序脚本定义了一个命令：print 命令。这个命令名副其实：它会将文本打印到 STDOUT。如果尝试运行这个命令，你可能会有些失望，因为什么都不会发生。原因在于没有在命令行上指定文件名，所以 gawk 程序会从 STDIN 接收数据。在运行这个程序时，它会一直等待从 STDIN 输入的文本。  
如果你输入一行文本并按下回车键，gawk 会对这行文本运行一遍程序脚本。跟 sed 编辑器一样，gawk 程序会针对数据流中的每行文本执行程序脚本。由于程序脚本被设为显示一行固定的文本字符串，因此不管你在数据流中输入什么文本，都会得到同样的文本输出。  
要终止这个 gawk 程序，你必须表明数据流已经结束了。bash shell 提供了一个组合键来生成 EOF（End-of-File）字符。Ctrl+D 组合键会在 bash 中产生一个 EOF 字符。这个组合键能够终止该 gawk 程序并返回到命令行界面提示符下。

### 使用数据字段变量

gawk 的主要特性之一是其处理文本文件中数据的能力。它会自动给一行中的每个数据元素分配一个变量。默认情况下，gawk 会将如下变量分配给它在文本行中发现的数据字段：

- $0 代表整个文本行；
- $1 代表文本行中的第 1 个数据字段；
- $2 代表文本行中的第 2 个数据字段；
- $n 代表文本行中的第 n 个数据字段。

在文本行中，每个数据字段都是通过`字段分隔符`划分的。gawk 在读取一行文本时，会用预定义的字段分隔符划分每个数据字段。gawk 中默认的字段分隔符是任意的空白字符（例如空格或制表符）。

在下面的例子中，gawk 程序读取文本文件，只显示第 1 个数据字段的值。

```bash
$ cat data2.txt
One line of test text.
Two lines of test text.
Three lines of test text.
$
$ gawk '{print $1}' data2.txt
One
Two
Three
$
```

该程序用$1 字段变量来仅显示每行文本的第 1 个数据字段。

如果你要读取采用了其他字段分隔符的文件，可以用-F 选项指定。

```bash
$ gawk -F: '{print $1}' /etc/passwd
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
[...]
```

这个简短的程序显示了系统中密码文件的第 1 个数据字段。由于/etc/passwd 文件用冒号来分隔数字字段，因而如果要划分开每个数据元素，则必须在 gawk 选项中将冒号指定为字段分隔符。

### 在程序脚本中使用多个命令

如果一种编程语言只能执行一条命令，那么它不会有太大用处。gawk 编程语言允许你将多条命令组合成一个正常的程序。要在命令行上的程序脚本中使用多条命令，只要在命令之间放个分号即可。

```bash
$ echo "My name is Rich" | gawk '{$4="Christine"; print $0}'
My name is Christine
$
```

第一条命令会给字段变量$4 赋值。第二条命令会打印整个数据字段。注意， gawk 程序在输出中已经将原文本中的第四个数据字段替换成了新值。  
也可以用次提示符一次一行地输入程序脚本命令。

```bash
$ gawk '{
> $4="Christine"
> print $0}'
My name is Rich
My name is Christine
$
```

在你用了表示起始的单引号后，bash shell 会使用次提示符来提示你输入更多数据。你可以每次在每行加一条命令，直到输入了结尾的单引号。因为没有在命令行中指定文件名，gawk 程序会从 STDIN 中获得数据。当运行这个程序的时候，它会等着读取来自 STDIN 的文本。要退出程序，只需按下 Ctrl+D 组合键来表明数据结束。

### 从文件中读取程序

跟 sed 编辑器一样，gawk 编辑器允许将程序存储到文件中，然后再在命令行中引用。

```bash
$ cat script2.gawk
{print $1 "'s home directory is " $6}
$
$ gawk -F: -f script2.gawk /etc/passwd
root's home directory is /root
bin's home directory is /bin
daemon's home directory is /sbin
adm's home directory is /var/adm
lp's home directory is /var/spool/lpd
[...]
Christine's home directory is /home/Christine
Samantha's home directory is /home/Samantha
$
```

script2.gawk 程序脚本会再次使用 print 命令打印/etc/passwd 文件的主目录数据字段（字段变量$6），以及 userid 数据字段（字段变量$1）  
可以在程序文件中指定多条命令。要这么做的话，只要一条命令放一行即可，不需要用分号。

```bash
$ cat script3.gawk
{
text = "'s home directory is "
print $1 text $6
}
$
$ gawk -F: -f script3.gawk /etc/passwd
root's home directory is /root
bin's home directory is /bin
daemon's home directory is /sbin
adm's home directory is /var/adm
lp's home directory is /var/spool/lpd
[...]
Christine's home directory is /home/Christine S
amantha's home directory is /home/Samantha
$
```

script3.gawk 程序脚本定义了一个变量来保存 print 命令中用到的文本字符串。注意，gawk 程序在引用变量值时并未像 shell 脚本一样使用美元符。

### 在处理数据前运行脚本

gawk 还允许指定程序脚本何时运行。默认情况下，gawk 会从输入中读取一行文本，然后针对该行的数据执行程序脚本。有时可能需要在处理数据前运行脚本，比如为报告创建标题。BEGIN 关键字就是用来做这个的。它会强制 gawk 在读取数据前执行 BEGIN 关键字后指定的程序脚本。

```bash
$ gawk 'BEGIN {print "Hello World!"}'
Hello World!
$
```

这次 print 命令会在读取数据前显示文本。但在它显示了文本后，它会快速退出，不等待任何数据。如果想使用正常的程序脚本中处理数据，必须用另一个脚本区域来定义程序。

```bash
$ cat data3.txt
Line 1
Line 2
Line 3
$
$ gawk 'BEGIN {print "The data3 File Contents:"}
> {print $0}' data3.txt
The data3 File Contents:
Line 1
Line 2
Line 3
$
```

在 gawk 执行了 BEGIN 脚本后，它会用第二段脚本来处理文件数据。这么做时要小心，两段脚本仍然被认为是 gawk 命令行中的一个文本字符串。你需要相应地加上单引号。

### 在处理数据后运行脚本

与 BEGIN 关键字类似，END 关键字允许你指定一个程序脚本，gawk 会在读完数据后执行它。

```bash
$ gawk 'BEGIN {print "The data3 File Contents:"}
> {print $0}
> END {print "End of File"}' data3.txt
The data3 File Contents:
Line 1
Line 2
Line 3
End of File
$
```

当 gawk 程序打印完文件内容后，它会执行 END 脚本中的命令。这是在处理完所有正常数据后给报告添加页脚的最佳方法。

可以将所有这些内容放到一起组成一个漂亮的小程序脚本文件，用它从一个简单的数据文件中创建一份完整的报告。

```bash
$ cat script4.gawk BEGIN {
print "The latest list of users and shells"
print " UserID \t Shell"
print "-------- \t -------"
FS=":"
}

{ print $1 "     \t "  $7
}

END {
print "This concludes the listing"
}
$
```

这个脚本用 BEGIN 脚本来为报告创建标题。它还定义了一个叫作 FS 的特殊变量。这是定义字段分隔符的另一种方法。这样你就不用依靠脚本用户在命令行选项中定义字段分隔符了。

下面是这个 gawk 程序脚本的输出（有部分删节）。

```bash

$ gawk -f script4.gawk /etc/passwd
The latest list of users and shells
UserID          Shell
--------         -------
root             /bin/bash
bin              /sbin/nologin
daemon           /sbin/nologin
[...]
Christine        /bin/bash
mysql            /bin/bash
This concludes the listing
$
```

与预想的一样，BEGIN 脚本创建了标题，程序脚本处理特定数据文件（/etc/passwd）中的信息，END 脚本生成页脚。

这个简单的脚本让你小试了一把 gawk 的强大威力。后续将继续介绍另外一些编写 gawk 脚本时的简单原则，以及一些可用于 gawk 程序脚本中的高级编程概念。学会了它们之后，就算是面对最晦涩的数据文件，你也能够创建出专业范儿的报告。

---

使用 sed 和 gawk 程序的关键在于了解如何使用正则表达式。正则表达式是为提取和处理文本文件中数据创建定制过滤器的关键。下一章将会深入经常被人们误解的正则表达式世界，并演示如何构建正则表达式来操作各种类型的数据。
