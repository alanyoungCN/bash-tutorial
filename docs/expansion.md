# Bash 的扩展模式

## 简介

Shell 接收到用户输入的命令以后，会根据空格将用户的输入，拆分成一个个词元（token）。然后，Shell 会扩展词元里面的特殊字符，扩展完成后才会调用相应的命令。

这种特殊字符的扩展，称为通配符扩展（wildcard expansion）或者模式扩展（globbing）。Bash 一共提供八种扩展，先进行扩展，然后再执行命令。

早期的 Unix 系统有一个`/etc/glob`文件，保存扩展的模板。后来 Bash 内置了这个功能，但是这个名字保留了下来。特殊字符的扩展早于正则表达式出现，可以看作是原始的正则表达式。它的功能没有正则那么强大灵活，但是优点是简单和方便。

Bash 允许用户关闭通配符扩展。

```bash
$ set -o noglob
# 或者
$ set -f
```

下面的命令可以重新打开通配符扩展。

```bash
$ set +o noglob
# 或者
$ set +f
```

## 波浪线扩展

波浪线`~`会自动扩展成当前用户的主目录。

```bash
$ echo ~
/home/me
```

如果要进入主目录的某个子目录，通常会把子目录放在波浪号后面。

```bash
# 进入 /home/me/foo 目录
$ cd ~/foo
```

如果`~`后面是已经存在的用户名，则会返回该用户的主目录。

```bash
$ echo ~foo
/home/foo

$ echo ~root
/root
```

上面例子中，Bash 会根据波浪号后面的用户名，返回该用户的主目录。

如果是不存在的用户名，则波浪号扩展不起作用。

```bash
$ echo ~nonExistedUser
~nonExistedUser
```

此外，`~+`会返回当前所在的目录，等同于`pwd`命令。

```bash
$ cd ~/foo
$ echo ~+
/home/me/foo
```

## `?` 字符扩展

`?`字符代表文件路径里面的任意单个字符，不包括空字符。比如，`Data???`匹配所有`Data`开头后面跟着三个字符的文件名。

```bash
# 存在文件 a.txt 和 b.txt
$ ls ?.txt
a.txt b.txt
```

上面命令中，`?`表示单个字符，所以会同时匹配`a.txt`和`b.txt`。

如果匹配多个字符，就需要多个`?`连用。

```bash
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls ??.txt
ab.txt
```

上面命令中，`??`匹配了两个字符。

注意，Bash 是先进行扩展，再执行命令。因此，扩展的结果决定了命令执行的结果，各种扩展只是 Bash 的功能，命令本身并不存在参数扩展，收到什么参数就原样执行。

```bash
# 当前目录有 a.txt 文件
$ echo ?.txt
a.txt

# 当前目录为空目录
$ echo ?.txt
?.txt
```

上面例子中，如果`?.txt`可以扩展成文件名，`echo`命令会输出扩展后的结果；如果不能扩展成文件名，`echo`就会原样输出`?.txt`。

## `*` 字符扩展

`*`字符代表文件路径里面的任意数量字符，包括零个字符。

```bash
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls *.txt
a.txt b.txt ab.txt

# 输出所有文件
$ ls *
```

下面是`*`匹配空字符的例子。

```bash
# 存在文件 a.txt、b.txt 和 ab.txt
$ ls a*.txt
a.txt ab.txt

$ ls *b*
b.txt ab.txt
```

注意，`*`不会匹配隐藏文件（以`.`开头的文件）。

```bash
# 显示所有隐藏文件
$ echo .*

# 只显示正常的隐藏文件，不显示 . 和 .. 这两个特殊文件
$ echo .[!.]*
```

## 方括号模式

方括号模式是`[...]`，可以匹配方括号之中的任意一个字符，比如`[aeiou]`可以匹配五个元音字母中的任意一个。该模式属于文件名匹配，即匹配后的结果必须符合现有的文件路径，如果不存在匹配，就会保持原样，不进行扩展。

```bash
# 存在文件 a.txt 和 b.txt
$ ls [ab].txt
a.txt b.txt

# 只存在文件 a.txt
$ ls [ab].txt
a.txt

# 不存在文件 a.txt 和 b.txt
$ ls [ab].txt
ls: 无法访问'[ab].txt': 没有那个文件或目录
```

上面命令中，`[ab]`表示可以扩展成`a`或`b`。具体的执行结果，取决于当前目录是否包含指定的文件。

方括号模式还有两种变体：`[^...]`和`[!...]`。它们表示匹配不在方括号里面的字符，这两种写法是等价的。比如，`[^abc]`或`[!abc]`表示匹配除了`a`、`b`、`c`以外的字符。

```bash
$ ls ?[!a]?
aba bbb
```

上面命令中，`[!a]`表示文件名第二个字符不是`a`的文件名。

注意，如果需要匹配`[`字符，可以放在方括号内，比如`[[aeiou]`。如果需要匹配连字号`-`，只能放在方括号内部的开头或结尾，比如`[-aeiou]`或`[aeiou-]`。

## [start-end] 模式

方括号模式`[start-end]`可以表示一个连续的范围。

```bash
# 存在文件 a.txt、b.txt 和 c.txt
$ ls [a-c].txt
a.txt
b.txt
c.txt

# 存在文件 report1.txt、report2.txt 和 report3.txt
$ ls report[0-9].txt
report1.txt
report2.txt
report3.txt
...
```

下面是更多的例子。

- `[a-z]`：所有小写字母。
- `[a-zA-Z]`：所有小写字母与大写字母。
- `[a-zA-Z0-9]`：所有小写字母、大写字母与数字。
- `[abc]*`：所有以`a`、`b`、`c`字符之一开头的文件名。
- `program.[co]`：文件`program.c`与文件`program.o`。
- `BACKUP.[0-9][0-9][0-9]`：所有以`BACKUP.`开头，后面是三个数字的文件名。

方括号模式的否定形式，也可以使用连续范围的写法`[!start-end]`，表示匹配不属于这个范围的字符。比如，`[!a-zA-Z]`表示匹配非英文字母的字符。

```bash
$ echo report[!1–3].txt
report4.txt report5.txt
```

上面代码中，`[!1-3]`表示排除1、2和3。

## 大括号扩展

大括号扩展`{...}`表示分别输出大括号里面的所有值，各个值之间使用逗号分隔。这种扩展跟上面的扩展都不一样，不是文件名扩展，即不会扩展成文件名，而是扩展成所有给定的值。

```bash
$ echo {1,2,3}
1 2 3

$ echo d{a,e,i,u,o}g
dag deg dig dug dog

$ echo Front-{A,B,C}-Back
Front-A-Back Front-B-Back Front-C-Back
```

注意，大括号内部的逗号前后不能有空格。否则，大括号模式会失效。

```bash
$ echo {1 , 2}
{1 , 2}
```

如果逗号前面没有值，就表示扩展的第一项为空。

```bash
$ cp a.log{,.bak}
# 等同于
# cp a.log a.log.bak
```

大括号可以嵌套。

```bash
$ echo {j{p,pe}g,png}
jpg jpeg png

$ echo a{A{1,2},B{3,4}}b
aA1b aA2b aB3b aB4b
```

大括号也可以与其他模式联用，并且总是先于其他模式进行扩展。

```bash
$ echo {cat,d*}
cat dawg dg dig dog doug dug
```

上面代码中，会先进行大括号扩展，然后进行`*`扩展。

大括号可以用于多字符的模式，方括号不行。

```bash
$ echo {cat,dog}
cat dog
```

大括号模式`{...}`与方括号模式`[...]`有一个很重要的区别。如果匹配的文件不存在，`[...]`会失去模式的功能，变成一个单纯的字符串，而`{...}`依然可以展开，因为大括号不是文件名扩展。

```bash
# 不存在 a.txt 和 b.txt
$ echo [ab].txt
[ab].txt

$ echo {a,b}.txt
a.txt b.txt
```

上面代码中，如果不存在`a.txt`和`b.txt`，那么`[ab].txt`就会变成一个普通的文件名，而`{a,b}.txt`可以照样展开。

## {start..end} 扩展

大括号里面两个点的`{start..end}`模式，表示扩展成一个连续序列，然后分别输出。比如`{a..z}`可以扩展成26个小写英文字母，然后输出。

```bash
$ echo {a..c}
a b c

$ echo {c..a}
c b a

$ echo d{a..d}g
dag dbg dcg ddg

$ echo {1..4}
1 2 3 4

$ echo Number_{1..5}
Number_1 Number_2 Number_3 Number_4 Number_5
```

大括号的双点号支持逆序。

```bash
$ echo {5..1}
5 4 3 2 1

$ echo {E..A}
E D C B A
```

如果遇到无法解释的扩展，模式会原样输出。

```bash
$ echo {a1..3c}
{a1..3c}
```

这种模式与逗号联用，可以写出复杂的模式。

```bash
$ echo .{mp{3..4},m4{a,b,p,v}}
.mp3 .mp4 .m4a .m4b .m4p .m4v
```

大括号扩展的常见用途为新建一系列目录。

```bash
$ mkdir {2007..2009}-{01..12}
```

上面命令会新建36个子目录，每个子目录的名字都是”年份-月份“。

整数前面可以使用前导`0`，然后输出的每一项都有前导`0`。

```bash
$ echo {01..5}
01 02 03 04 05

$ echo {001..5}
001 002 003 004 005
```

大括号里面还可以使用第二个双点号（`start..end..step`），用来指定扩展的步长。

```bash
$ echo {0..8..2}
0 2 4 6 8
```

上面代码将`0`扩展到`8`，每次递增的长度为`2`，所以一共输出5个数字。

多个大括号连用，会有循环处理的效果。

```bash
$ echo {a..c}{1..3}
a1 a2 a3 b1 b2 b3 c1 c2 c3
```

这个写法可以直接用于`for`循环。

```bash
for i in {1..4}
do
  echo $i
done
```

## 变量扩展

Bash 将美元符号`$`开头的词元视为变量，将其扩展成变量值，详见《Bash 变量》一章。

```bash
$ echo $SHELL
/bin/bash
```

变量名除了放在美元符号后面，也可以放在`${}`里面。

```bash
$ echo ${SHELL}
/bin/bash
```

`${!...*}`或`${!...@}`可以返回所有匹配的变量名。

```bash
$ echo ${!S*}
SECONDS SHELL SHELLOPTS SHLVL SSH_AGENT_PID SSH_AUTH_SOCK
```

上面例子中，`${!S*}`扩展成所有以`S`开头的变量名。

## 子命令扩展

`$(...)`可以扩展成另一个命令的运行结果，内部命令的所有输出都会作为返回值。

```bash
$ echo $(date)
Tue Jan 28 00:01:13 CST 2020
```

上面例子中，`$(date)`返回`date`命令的运行结果。

还有另一种较老的语法，子命令放在反引号之中，也可以扩展成命令的运行结果。

```bash
$ echo `date`
Tue Jan 28 00:01:13 CST 2020
```

`$(...)`还可以嵌套，比如`$(ls $(pwd))`。

## 算术扩展

`$((...))`可以扩展成整数运算的结果，详见《Bash 的算术运算》一章。

```bash
$ echo $((2 + 2))
4
```

## 字符类

`[[:class:]]`表示一个字符类，匹配某一类特定字符之中的一个。常用的字符类如下。

- `[[:alnum:]]`：匹配任意英文字母与数字
- `[[:alpha:]]`：匹配任意英文字母
- `[[:blank:]]`：空格和 Tab 键。
- `[[:cntrl:]]`：ASCII 码 0-31 的不可打印字符。
- `[[:digit:]]`：匹配任意数字 0-9。
- `[[:graph:]]`：A-Z、a-z、0-9 和标点符号。
- `[[:lower:]]`：匹配任意小写字母 a-z。
- `[[:print:]]`：ASCII 码 32-127 的可打印字符。
- `[[:punct:]]`：标点符号（除了 A-Z、a-z、0-9 的可打印字符）。
- `[[:space:]]`：空格、Tab、LF（10）、VT（11）、FF（12）、CR（13）。
- `[[:upper:]]`：匹配任意大写字母 A-Z。
- `[[:xdigit:]]`：16进制字符（A-F、a-f、0-9）。

请看下面的例子。

```bash
$ echo [[:upper:]]*
```

上面命令输出所有大写字母开头的文件名。

字符类的第一个方括号后面，可以加上感叹号`!`，表示否定。比如，`[![:digit:]]`匹配所有非数字。

```bash
$ echo [![:digit:]]*
```

上面命令输出所有不以数字开头的文件名。

## 使用注意点

通配符有一些使用注意点，不可不知。

**（1）通配符是先解释，再执行。**

Bash 接收到命令以后，发现里面有通配符，会进行通配符扩展，然后再执行命令。

```bash
$ ls a*.txt
ab.txt
```

上面命令的执行过程是，Bash 先将`a*.txt`扩展成`ab.txt`，然后再执行`ls ab.txt`。

**（2）通配符不匹配，会原样输出。**

Bash 扩展通配符的时候，发现不存在匹配的文件，会将通配符原样输出。

```bash
# 不存在 r 开头的文件名
$ echo r*
r*
```

上面代码中，由于不存在`r`开头的文件名，`r*`会原样输出。

下面是另一个例子。

```bash
$ ls *.csv
ls: *.csv: No such file or directory
```

另外，前面已经说过，这条规则对大括号模式`{...}`不适用。

**（3）只适用于单层路径。**

上面所有通配符只匹配单层路径，不能跨目录匹配，即无法匹配子目录里面的文件。或者说，`?`或`*`这样的通配符，不能匹配路径分隔符（`/`）。

如果要匹配子目录里面的文件，可以写成下面这样。

```bash
$ ls */*.txt
```

**（4）可用于文件名。**

Bash 允许文件名使用通配符，即文件名包括通配符字符。这时，引用文件名的时候，需要把文件名放在单引号里面。

```bash
$ touch 'fo*'
$ ls
fo*
```

上面代码创建了一个`fo*`文件，这时`*`就是文件名的一部分。

## 量词语法

量词语法用来控制模式匹配的次数。它需要在 Bash 的`extglob`参数打开的情况下，才能使用。不过，一般是默认打开的，可以用下面的命令查询。

```bash
$ shopt extglob
extglob        	on
```

量词语法有下面几个。

- `?(pattern-list)`：匹配零个或一个模式。
- `*(pattern-list)`：匹配零个或多个模式。
- `+(pattern-list)`：匹配一个或多个模式。
- `@(pattern-list)`：只匹配一个模式。
- `!(pattern-list)`：匹配零个或一个以上的模式，但不匹配单独一个的模式。

```bash
$ ls abc?(.)txt
abctxt abc.txt
```

上面例子中，`?(.)`匹配零个或一个点。

```bash
$ ls abc?(def)
abc abcdef
```

上面例子中，`?(def)`匹配零个或一个`def`。

```bash
$ ls abc+(.txt|.php)
abc.php abc.txt
```

上面例子中，`+(.txt|.php)`匹配文件有一个`.txt`或`.php`后缀名。

```bash
$ ls abc+(.txt)
abc.txt abc.txt.txt
```

上面例子中，`+(.txt)`匹配文件有一个或多个`.txt`后缀名。

## shopt 命令

`shopt`命令可以调整 Bash 的行为。它有好几个参数跟通配符扩展有关。

它的命令格式如下。

```bash
# 打开某个参数
$ shopt -s [optionname]

# 关闭某个参数
$ shopt -u [optionname]

# 查询某个参数关闭还是打开
$ shopt [optionname]
```

**（1）dotglob 参数**

`dotglob`参数可以让文件通配符包括隐藏文件（即点开头的文件）。

正常情况下，通配符扩展不包括隐藏文件。

```bash
$ ls *
abc.txt
```

打开`dotglob`，就会包括隐藏文件。

```bash
$ shopt -s dotglob
$ ls *
abc.txt .config
```

**（2）nullglob 参数**

`nullglob`参数可以让通配符不匹配任何文件名时，返回空字符。

默认情况下，通配符不匹配任何文件名时，会保持不变。

```bash
$ rm b*
rm: 无法删除'b*': 没有那个文件或目录
```

上面例子中，由于当前目录不包括`b`开头的文件名，导致`b*`不会发生文件名扩展，保持原样不变，所以`rm`命令报错没有`b*`这个文件。

打开`nullglob`参数，就可以让不匹配的通配符返回空字符串。

```bash
$ shopt -s nullglob
$ rm b*
rm: 缺少操作数
```

上面例子中，由于没有`b*`匹配的文件名，所以`rm b*`扩展成了`rm`，导致报错变成了”缺少操作数“。

**（3）failglob 参数**

`failglob`参数使得通配符不匹配任何文件名时，Bash 会直接报错，而不是让各个命令去处理。

```bash
$ shopt -s failglob
$ rm b*
bash: 无匹配: b*
```

上面例子中，打开`failglob`以后，由于`b*`不匹配任何文件名，Bash 直接报错了，不再让`rm`命令去处理。

**（4）extglob 参数**

`extglob`参数使得 Bash 支持 ksh 的一些扩展语法。它默认情况下面，应该是打开的。

```bash
$ shopt extglob
extglob        	on
```

它的主要应用是支持量词语法。如果不希望支持，可以用下面的命令关闭。

```bash
$ shopt -u extglob
```

**（5）nocaseglob 参数**

`nocaseglob`参数可以让通配符扩展不区分大小写。

```bash
$ shopt -s nocaseglob
$ ls /windows/program*
/windows/ProgramData
/windows/Program Files
/windows/Program Files (x86)
```

上面例子中，打开`nocaseglob`以后，`program*`就不区分大小写了，可以匹配`ProgramData`等。

## 参考链接

- [Think You Understand Wildcards? Think Again](https://medium.com/@leedowthwaite/why-most-people-only-think-they-understand-wildcards-63bb9c2024ab)
- [Advanced Wildcard Patterns Most People Don’t Know](https://appcodelabs.com/advanced-wildcard-patterns-most-people-dont-know)
