bash 脚本编程
======

[这里是本文的参考链接](https://github.com/wangdoc/bash-tutorial)，算是对原文的一个精简，因方便自己日后学习查找而生不熟悉的朋友建议直接浏览原文。
shell 是一个命令解释器同时也支持变量、条件判断、循环等语法，所以也亦成为脚本。bash 全称为 Bourne Again Shell。

## PART 1

### echo

使用 echo 命令进行输出的时候，可以使用双引号以输出多行内容。

-n，用以取消换行，对比一下两条命令的差别：

```bash
echo hello; echo world
echo -n hello; echo -n world
```

-e，用以解释引号当中的特殊字符，例如`\n`换行符；

### 组合符

除了有`;`分割两条命令外，还有`&&`与`||`来控制语句间的逻辑关系。

## 扩展

### 模式扩展

特殊字符的扩展，成为模式扩展（globbing）。其中用到通配符又称为 wildcard expansion。

?，匹配文件路径里的任意单个字符；

*，匹配文件路径里的任意数量的任意字符；

[...]，匹配括号中的任意一个字符，其中 [^] 表示匹配不在范围内的字符；

[start-end]，匹配范围；

{}，大括号中的直用都好分隔，扩展成其中的所有直；

```bash
cp main.c{,.bakup}
```

{start..end}，扩展为一个连续的系列；

{start..end..step}，扩展时制定步长；

### 变量扩展

`$`符号开头的将被 bash 是为变量，并将其扩展为变量直，也可以使用`${VAR}`的形式。

```bash
echo ${!S*}
# SECONDS SHELL SHELLOPTS SHELL_SESSION_ID SHLVL
```

将现实所有以大写 S 开头的变量名。

### 子命令扩展

`$(COMMAND)` 将被扩展为该命令的运行结果，当然也可以嵌套使用。

### 算数扩展

`$((EXPRESSION))`将被扩展为整数运算的结果：

```bash
echo $((1 + 1))
```

### 字符类

[[:alnum:]]

[[:alpha:]]

[[:blank:]]

[[:digit:]]

[[:lower:]]

[[:upper:]]

或者匹配其之外的字符，如`[![:alnum:]]`

## 引号和转义

Bash 只有一种数据类型，字符串。用户的输入都被视为字符串。

```bash
echo $date # 并不会们有任何结果，无此变量
echo \$date
# \ 本身也是特殊字符
echo \\
```

反斜线`\`除了用于转义，还可用于表示一些不可打印的字符：

- `\a`：响铃
- `\b`：退格
- `\n`：换行
- `\r`：回车
- `\t`：指标符

使用时，需要加上`-e`选项并放于双引号当中：

```bash
echo -e "hello\nworld"
```

### 单引号

单引号用于保留字符的字面含义，特殊字符放于但引号中都会变为普通字符。

### 双引号

双引号的限制稍微宽松，仅剩美元符号、反引号、反斜线依然有效：

```bash
echo "$SHELL"
```

### Here documnet

这是一种输入多行字符串的方法：

```bash
<< EOF
hello
world
EOF
```

其格式为开始标记（<< EOF）和结束表示（EOF），可加上重定向用于写入文件：

```bash
cat << EOF > main.c
#include <stdio.h>
EOF
```

### Here string

用于将字符串通过标准输入传递给命令，

```bash
cat <<< hello
md5sum <<< 'hello'
```

## Bash 变量

Bash 变量分为环境变量和自定义变量两类，环境变量通常已由系统定义好可直接使用，如下查看所有环境变量：

```bash
env # 或者
printenv
```

注意：Bash 变量区分大小写。

`set`命令显示所有变量，以及所有 Bash 函数。

### 创建变量

用户创建的变量必须符合，由字符数字下划线组成，并且第一个字母必须是字母或下划线。

声明语法为（注两边不能有空格）：

```bash
variable=value
```

**Bash 没有数据类型的概念，所有变量直都是字符串。**

### 读取变量

读取时直接在变量名前加上`$`即可，每当 shell 看到一美元符开头的单词时都会尝试读取这个变量名对应的直，如果该变量不存在不会引发报错，仅会输出空白字符。

当遇见变量名遇见与别的字符相连时，可用花括号进行分离：

```bash
hell=999
echo ${hell}oworld
```

### 删除变量

使用`unset`进行删除，或者直接对该变量赋值空字符串。

```bash
unset NAME
foo=''
```

### 输出变量 export

用户创建的变量仅可用于当前的 Shell，子进程默认读取不到父 Shell 中定义的变量。使用 export 即可将变量传递给子 Shell。

```bash
NAME=foo
export NAME
# export NAME=foo
```

后在子 Shell 中修改继承的变量将不会影响父 Shell。

### 特殊变量

这些特殊变量由 Shell 提供用户不能进行赋值：

- $?：上一条命令的退出码，0 表示成功执行，非零表示执行失败；

- $$：当前 Shell  的进程号；
- $_：上一命令的最后一个参数；
- $! ：最后一个后台执行的异步命令的进程号；
- $0：当前 Shell  的名称；
- $-：当前 Shell 的启动参数；
- $@, $#：表示脚本的参数数量。

### 变量的默认值

Bash 的四种语法设置变量的默认值，以保证变量不为空。

```bash
${variable:-word}
```

如果`variable`存在且不为空，则返回它本身的直，否则返回`word`。其目的是返回一个默认直 ，如`${cout:-word}`

```bash
${variable:=word}
```

如果`variable`存在且不为空，则返回它本身的直，否则将它设置为`word`。其目的是为变量设置默认直。

```bash
${variable:+word}
```

如果变量名存在且不为空，则返回`word`，否则返回空直。它的目的是检测变量是否存在。

```bash
${variable:?message}
```

如果变量存在且不为空，则返回它的直，否则打印出`variable: message`并中断脚本的执行。目的是防止变量未定义。

### declare 命令

用以声明一些特殊类型的变量，为变量设置一些限制，如声明成为只读或整数类型，语法如下：

```bash
declare OPTION VARIABLE=value
# help declare
```

其中主要参数（OPTION）有下：

- -a：声明数组变量；
- -i：声明整数变量；
- -r：声明只读变量，等同于 readonly；
- -x：该变量输出为环境变量，等同于 export；

```bash
readonly foo=1
declare -p foo
```

- -l：声明变量为小写字母；
- -u：声明变量为大写字母；
- -p：查看变量信息；
- -f：输出所有函数定义；

### let 命令

用 `let`命令声明变量时，可直接执行算数表达式：

```bash
let val1=1+2
echo $val1
```



## 字符串操作

获取字符串的长度：

```bash
${#variable}
echo ${#PATH}
```

其中花括号是必需的，否则 Shell 会首先解释出`$#`当前脚本参数之意。

### 子字符串

```bash
${variable:offset:length}
```

此为返回变量`$variable`的从`offset`开始的长度为`length`的子串。此操作不能直接操作字符串，而只能通过变量来读取字符串，并且不会改变原始字符串。

### 头尾模式匹配、以及任意位置匹配

[Links](https://wangdoc.com/bash/string.html)，用到了再来补充。



## Bash 的算数运算

进行算数运算的语法为：

```bash
~]# ((foo = 5 + 4))
~]# echo $foo
```

双括号中可以添加任意数量的空格。当算数运算的结果不为`0`时，命令就算执行成功`echo $?`，支持的算数运算符有余数、指数、前缀或后缀的自增自减。

算数运算中的字符串会被 Bash 当作不存在的变量处理，即为空直。

### 数值的进制

Bash 中默认以十进制表示，也有如下的方式：

- `0number`：八进制；
- `0xnumber`：十六进制；
- `base#number`：base 进制。

```bash
~]$ echo $((2#111010101010))
3754
# 注：美元符号不属于运算表达式部分
```

### 位运算

`((...))`同样支持以下的二进制位运算：左移右移、按位与或否、异或。

### 赋值运算

算数表达式`((...))`可以执行赋值运算：

```bash
~]$ echo $((foo=3))
3
```

### 逗号运算符

逗号`,`此处的作用与 C 语言中的类似，用以返回后一个表达式的直。

```bash
~]$ echo $((4324,234,234,32,2))
2
```

### expr 命令

expr 命令可以直接计算算数表达式：

```bash
~]$ expr 3  + 3
6
```

expr 同样不支持非整数的运算。

### 提到的为变量赋运算表达式直的方式

```bash
let foo=1+1
((foo=2+2))
foo=$(expr 3 + 3)
```



## 目录堆栈

Bash 默认情况下只记录用户前一次所在的目录，`cd -`便可返回前一次的目录。

### pushd，popd，dirs

当你希望 Bash 为你记录多重目录时，便可使用`pushd`命令，进入目录并将其加入目录堆栈。

```bash
pushd dirname
```

第一次使用`pushed`命令会先将当前目录放入堆栈，再将所要进入的目录也放入堆栈；再以后的使用中都会将所要进入的目录压在堆栈顶部。

`popd`命令不带参数时，会弹出堆栈顶部的记录，并且进入新堆栈顶部的目录。

`dirs`命令用于显示目录堆栈的内容。



# PART2 脚本

脚本 script 就是包含一系列命令的一个文本文件，Shell 读取文件依次执行里面的所有命令，就好像是直接输入到命令行一样。

## 入门操作

### Shebang

脚本的第一行用于指定解释器，必须要以`#!`字符开头，后跟上解释器的位置。

如无 Shebang 行，则需要你手动将脚本传递给解释器。

指定 Shebang 行过后，脚本需要有可执行权限：

```bash
chmod +x script.sh
```

### env 命令

env 命令总是指向`/usr/bin/env`文件，也就是说这个二进制文件总是在目录`/usr/bin`中。

`/usr/bin/env NAME`的意思是，让 Shell 查找`$PATH`中的第一个匹配；当你希望兼容其他用户机器时，这样的写法就很有用：

```bash
#!/usr/bin/env bash
```

当使用`-i,--ignore-environment`参数时表示启动时不带环境变量。

### 注释

Bash 脚本中`#`表示注释，可以放在行首或者行尾。在脚本开头注释说明脚本作用，以便日后维护。

### 脚本参数

调用脚本时，脚本文件名后可以带有参数：

```bash
./script.sh p1 p2 p3d
```

在脚本内部可以使用特殊变量，进行引用：

- `$0`：脚本文件名；
- `$1`~`$9`：对应脚本的第一到九个参数；
- `$#`：参数的总数；
- `$@`：全部参数，参数间使用空格分割；
- `$*`：全部参数，参数间使用变量`$IFS`直的第一个字符分割，默认为空格，也可自定义。

当脚本参数大于 9 个时，第十个参数使用`${10}`的格式，后续参数依次类推。

### shift 命令

shift 命令用以改变脚本参数，每次执行都会移除当前脚本的第一个参数（`$1`）使得后面的参数向前一位。也可以整数参数来移除相应个数个参数。

### getopts 命令

在脚本内部用以解析复杂的脚本命令行参数，通常与`while`循环一起使用，取出脚本所有的带有前置连词线（`-`）的参数。

```bash
getopts optstring name
```

第一个参数`optstring`是字符串，给出脚本所有的连词线参数。

如某脚本有三个配置项参数`-l`、`-h`、`-a`，而其中只有`-a`可以带有参数值，剩余的皆为开关参数；那么第一个参数 就应该些为`lha:`，顺序不重要。**注意**`a`后面有一个冒号，表示该参数带有参数值，有规定带有参数值的配置项参数后面必须要有一个冒号。

第二个参数`name`是一个变量名，用来保存当前取到的配置项参数，即`l`、`h`还有`a`。

```bash
while getopts 'lha:' OPTION; do
  case "$OPTION" in
    l)
      echo "linuxconfig"
      ;;

    h)
      echo "h stands for h"
      ;;

    a)
      avalue="$OPTARG"
      echo "The value provided is $OPTARG"
      ;;
    ?)
      echo "script usage: $(basename $0) [-l] [-h] [-a somevalue]" >&2
      exit 1
      ;;
  esac
done
shift "$(($OPTIND - 1))"
```

运行示例如下：

```bash
$ ./script.sh -lha name_a
linuxconfig
h stand for humanreadable
The value is name_a
```

while 循环不断执行`getopts 'lha:' OPTION`命令，每一次的执行都会读取一个连词线参数（以及其对应的值），然后进入循环体。

变量`OPTION`保存的是当前正在处理的那一个连词线参数（即`l`、`h`或者`a`）；如果用户输入了不存在的参数，那么`OPTION`的值将为`?`。由此循环体内部的 case 才能处理这四种不同的情况。

当处理至某个带有参数值的连词线参数时，如`-a name_a`，那么处理`a`参数时环境变量`$OPTARG`保存的就是该参数值。

变量`$OPTIND`在`getopts`开始执行前是`1`，接着每次执行都会增加`1`。等到退出 while 循环也就是连词参数全部处理完毕时，`$OPTIND - 1`就是已经处理了的连词线参数的个数，再使用`shift`命令将这些参数移除一保证后续代码可正常处理`$1`、`$2`等主参数。

### exit 命令

用于终止当前脚本的执行，并向 Shell 返回一个退出值。

不跟参数时就将最后一条命令的推出状态作为整个脚本的推出状态；若后面跟有参数，该参数就是推出状态。

`0`表示正常，`1`表示发生错误，当脚本被信号`N`终止，则退出值为`128+N`。

### 利用命令的执行结果

命令执行结束后的返回值`0`表示执行成功，环境变量`$?`可以读取前一命令的返回值。当然也可以这样：

```bash
if cd /tmp; then
	pwd
else
	exit 1
fi
```

或者用更简介的写法：

```bash
cd /tmp && pwd || exit 1
```

### source 命令

通过`source`命令执行一个脚本时，通常用于重新加载一个配置文件。

```bash
source ~/.bashrc
```

其最大的特点是`source`会在当前 Shell 执行脚本，而非新建一个子 Shell。

source 还有一种简写形式`.`:

```bash
. ~/.bashrc
```

### read 命令

将用户的输入存入一个变量，回车表示输入结束。

```bash
read [-options] [variable...]
```

variable 是用来保存输入数值的一个或多个变量名，若未提供变量名，则由环境变量`REPLY`包含用户输入的一整行数据。

当用户的输入少于给出的变量名时，额外的变量值将为空；若输入多于变量数，多余的输入将会被包含到最后一个变量当中。

read 还可用来读取文件的输入，交互时使用示例如下：

```bash
read < script.sh
```

参数 `-t`可设置超时秒数，超时将继续执行下条命令。

参数`-p`后可跟上提供给用户的帮助信息。

参数`-n`可指定只读取若干字符作为变量值而非整行读取。

参数`-e`可允许用户输入时使用`Tab`快捷键的自动补全功能。



## 条件判断

### if 命令

语法结构如下：

```bash
if commands; then
	commands
[elif commands; then
	commands]
fi
```

`ture`与`false`是两个关键字，当然 Shell 脚本也可以单行写：

```bash
if true; then echo 'hello world'; fi
```

当然也可以将命令的返回值（成功返回0）来作为判断条件：

```bash
if cd; then pwd; fi
```

### test 命令

if 结构中的判断条件一般使用`test`命令，其有三种格式：

```bash
test expression

[ expression ]

[[ expression ]]
```

三种写法等价，但是仅第三种还支持正则判断。

上面的`expression`表达式为真时，`test`命令成功执行返回值为`0`；**注意**第二三种写法中，`[`与`]`和内部的表达式中必须要有空格。

```bash
~]$ test -f /etc/passwd
~]$ echo $?
0
```

实际`[`字符是`test`命令的简写，这就解释了他的后面为什么要有空格。

### 文件判断

一下为判断文件状态的表达式：

- `[ -a file ]`：如果 file 存在，则为`true`。
- `[ -b file ]`：如果 file 存在并且是一个块（设备）文件，则为`true`。
- `[ -c file ]`：如果 file 存在并且是一个字符（设备）文件，则为`true`。
- `[ -d file ]`：如果 file 存在并且是一个目录，则为`true`。
- `[ -e file ]`：如果 file 存在，则为`true`。
- `[ -f file ]`：如果 file 存在并且是一个普通文件，则为`true`。
- `[ -p file ]`：如果 file 存在并且是一个命名管道，则为`true`。
- `[ -r file ]`：如果 file 存在并且可读（当前用户有可读权限），则为`true`。
- `[ -s file ]`：如果 file 存在且其长度大于零，则为`true`。
- `[ -S file ]`：如果 file 存在且是一个网络 socket，则为`true`。
- `[ -t fd ]`：如果 fd 是一个文件描述符，并且重定向到终端，则为`true`。 这可以用来判断是否重定向了标准输入／输出错误。
- `[ -u file ]`：如果 file 存在并且设置了 setuid 位，则为`true`。
- `[ -w file ]`：如果 file 存在并且可写（当前用户拥有可写权限），则为`true`。
- `[ -x file ]`：如果 file 存在并且可执行（有效用户有执行／搜索权限），则为`true`。
- `[ file1 -nt file2 ]`：如果 FILE1 比 FILE2 的更新时间最近，或者 FILE1 存在而 FILE2 不存在，则为`true`。
- `[ file1 -ot file2 ]`：如果 FILE1 比 FILE2 的更新时间更旧，或者 FILE2 存在而 FILE1 不存在，则为`true`。
- `[ FILE1 -ef FILE2 ]`：如果 FILE1 和 FILE2 引用相同的设备和 inode 编号，则为`true`。

**注意：**在使用自定义变量如`$FILENAME`时，要将它用双引号引起放入`[]`中，使之变量不存在时总是返回一个空字符串：

```bash
~]$ [ -f ]
~]$ echo $?
0
~]$ [ -f "" ]
~]$ echo $?
1
```

### 字符串判断

以下表达式用于字符串的判断：

- `[ string ]`：如果`string`不为空（长度大于0），则判断为真。
- `[ -n string ]`：如果字符串`string`的长度大于零，则判断为真。
- `[ -z string ]`：如果字符串`string`的长度为零，则判断为真。
- `[ string1 = string2 ]`：如果`string1`和`string2`相同，则判断为真。
- `[ string1 == string2 ]` 等同于`[ string1 = string2 ]`。
- `[ string1 != string2 ]`：如果`string1`和`string2`不相同，则判断为真。
- `[ string1 '>' string2 ]`：如果按照字典顺序`string1`排列在`string2`之后，则判断为真。
- `[ string1 '<' string2 ]`：如果按照字典顺序`string1`排列在`string2`之前，则判断为真。

**注意：**命令内部的`<`必须用引号引起来，或者是用反斜杠转移，否则会被 Shell 解释为重定向符。

### 整数判断

下面的表达式用作整数判断：

- `[ integer1 -eq integer2 ]`：如果`integer1`等于`integer2`，则为`true`。
- `[ integer1 -ne integer2 ]`：如果`integer1`不等于`integer2`，则为`true`。
- `[ integer1 -le integer2 ]`：如果`integer1`小于或等于`integer2`，则为`true`。
- `[ integer1 -lt integer2 ]`：如果`integer1`小于`integer2`，则为`true`。
- `[ integer1 -ge integer2 ]`：如果`integer1`大于或等于`integer2`，则为`true`。
- `[ integer1 -gt integer2 ]`：如果`integer1`大于`integer2`，则为`true`。

### 正则判断

`[[ expression ]]`的判断形式支持正则表达式。

### test 判断的逻辑运算

- `AND`运算：符号`&&`，也可使用参数`-a`。
- `OR`运算：符号`||`，也可使用参数`-o`。
- `NOT`运算：符号`!`。

### 算数判断

`((...))`用作算数条件进行算数运算的判断：

```bash
if (( 2 > 1 )); then pwd; fi
```

**注意：**算数运算的结果的非零直都将导致判断成立！

```bash
if (( foo = 5 )); then pwd; fi
```

上面的算数运算中完成了两件事情，首先为变量赋值并且根据返回值完成了判断。

### case 结构

case 结构用于多直判断，可以为每个直指定对应执行的命令，等价于多个`if-else`结构：

```bash
case expression in
	pattern )
		commands ;;
	pattern )
		commands ;;
	...
esac
```

其中`expression`是一个表达式，`pattern`是表达式的直或者一个模式，每条以两个分号结尾。

若最后一条匹配语句的模式为`*`，则这个通配符可以匹配其他字符和没有输入字符的情况。



## 循环

### while

while 循环有一个判断条件，语法如下：

```bash
while condition; do
	commands
done
```

`condition`的用法可以与`if`相似。

### until

```bash
until condition; do
	commands
done
```

### for...in

```bash
for variable in list; do
	commands
done
```

示例：

```bash
for i in $( ls ); do echo $i; done
```

### for

for 循环支持 C 语言循环语法：

```bash
for (( expression1; expression2; expression3 )); do
	command;
done
```

### 同样支持使用 break，continue

### select 结构

用于生成简单的菜单，其语法与`for...in`一致。

```bash
select name
[in list]
do
	commands
done
```

bash 会进行一下处理：

1. 生成一个`list`的选择菜单，并且附带编号；
2. Bash 提示用户输入；
3. Bash 将用户的输入存入变量`name`当中，选项的编号将存入`REPLY`当中；
4. 执行`command`命令；
5. 执行结束后回到第一步。

















