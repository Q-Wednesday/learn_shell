# 编写第一个shell脚本
## shell脚本是啥
简单的解释就是包含一系列命令的文件，shell读取这个文件并执行这些命令。shell很独特，既是强大的命令行接口也是脚本解释器。

## 如何写
1. 编写脚本。需要一个文本编辑器，最好可以高亮。
2. 使脚本可以执行。系统不会把任何老式文本文件当作程序，需要给脚本设置权限为允许执行。
3. 把脚本放在shell可以发现的位置。

### 脚本的格式
创建hello world
```
#!bin/bash
# This is our first script

echo 'Hello World!'
```

### 可执行权限
需要修改权限，因为默认没有

### 脚本位置
执行 ./hello_world<br>
注意不能执行hello_world,必须要显示告知脚本位置。不加位置的话会在PATH里搜寻，通过echo $PATH就可以看到。把当前文件移动到PATH种或者把目录设置为PATH也可以不加./运行。在Ubuntu中，如果有~/bin,则当前用户会自动把~/bin添加到PATH中

### 脚本的理想位置
~/bin是存放个人脚本的理想位置。

## 更多的格式诀窍
认真编写脚本是给维护提供便利。

1. 长选命名<br>
很多命令有长选项，如ls -ad其实相当于 ls --all --directory<br>
在命令行敲的时候会选择短的，但是编写脚本可以选择长的。

2. 缩进和行连接<br>
把命令扩展为多行可以提高可读性，如
```
find playground \
    \( \
        -type f \
        -not -perm 0600 \
        -exec chmod 0600 '{}' ';' \

    \) \
    -or \
    \( \
        -type d \
        -not -perm 0700 \
        -exec chmod 0700 '{}' ';'
    \)
```
通过行连接符（反斜杠-回车符序列）和缩进可以清晰了解复杂命令的逻辑。其实在命令行里也能用，但是比较麻烦<br>

## 为了编写脚本配置vim
:syntax on可以打开语法高亮<br>
:set health 用来把搜索的结果高亮显示<br>
:set tabstop=4  设置tab的空格长度为4个<br>
:set autoindent 开启自动缩进<br>
把这些命令（不要前面的冒号）添加到-/.vimrc中，改变永久生效。


# 启动一个项目
编写一个报告生成器，显示系统的各种统计数据和状态并以HTML格式产生报告。这样就可以用Web浏览器查看。<br>
程序一般由一系列阶段组成，每一个阶段都会增加一些特性和功能，第一阶段就是创造不含任何系统信息的HTML页面

## 最小的文档
写一个脚本sys_info_page来输出一个foo.html。
由于我是在服务器上弄的，所以还会花一些时间用Nginx或Apache提供静态文件服务。

## 加入一点数据
## 变量和常量
使用变量替代常量更易于维护.
1. 创建变量和常量<br>
shell遇到一个变量后会自动创建变量而不需要声明。取得变量的值需要使用$。如果$一个不存在的变量，那么会被扩展为空，即`echo $foo`相当于`echo` 。

2. 变量和常量赋值<br>
shell不关心赋给变量的值的类型，它会把其都当作字符串。如`foo=z`,那么会把字符串"z"给foo。使用带-i选项的declare命令可以强制把变量转化为整数类型。但是，这如同把变量设置为只读一样，通常很少这样用。<br>
可以在一行给多个变量赋值，如
    ``` bash
    a=5 b="a string"
    ```
    在扩展期间可以用花括号括住扩展名称，如选择`${file}1`而不是`$file1`,如果我们只有file这个变量的话，可以得到$file+1这个值，这样很方便。

## here 文档
我们已经知道怎么用echo输出文本，此外还有成为hure文档或者here脚本的方法来输入文本。here文档是I/O重定向的另外一种形式，在脚本中嵌入正文文本，然后输入到一个命令的标准输入中，如
``` bash
command << token
text
token
```
token是用来指示结尾的字符串。比如
``` bash
cat << _EOF_
....
_EOF_
```
使用cat的here文档来替代echo。这会带来很多好处，比如""不再具有特殊意义，会被直接输出。而且使用<<- 可以忽略tab缩进，提供可读性

# 自顶向下设计
把大的任务拆分为小的任务。
## shell 函数
shell函数有两种语法形式
``` bash
function name{
    commands
    return
}
```
*name* 是函数名称，*commands* 是这个函数的一系列命令。第二种是
``` bash
name () {
    commands
    return
}
```
shell读取脚本的时候会跳过函数部分，当执行遇到调用函数的时候再跳到函数里去执行。shell函数的命名规则和变量相同。一个函数必须至少要包含一条命令。return命令（可选的）可以满足该要求

## 局部变量
在编写的脚本里，所有变量都是全局变量。在shell函数中经常要用到局部变量，仅在函数里有效。需要声明为
```
local foo
```

## 保持脚本的运行
进行改动后要对脚本进行测试。可以通过添加空函数(stub)来在早期验证程序流程（就是函数只输出信息不执行改动）<br>
df -h可以检查磁盘空间；du -sh是检查指定目录空间。

# if流程控制
## 使用if
``` bash
x=5
if [ $x=5 ];then
    echo "x equazls 5"
else
    echo "x does not equal 5"
fi
```
直接在命令行里输入则写为
``` bash
if [ $x=5 ];then echo "x equazls 5";else echo "x does not equal 5";fi
```
if的语法格式如下
``` bash
if commands; then
    commands
[elif commands; then 
    commands...]
[else 
    commands]
fi
```
commands可以是一组命令

## 退出状态
命令执行完毕后会向操作系统发送一个值，叫退出状态。一般是0~255的整数。0表示执行成功，其他表示失败。如
``` bash
ls -d /usr/bin
echo $?
```
参数?表示上一个程序的退出状态。shell提供了两个简单的内置命令true和false,true用0作为退出状态（总是成功），false用1作为退出状态（总是失败）。而if真正做的就是评估命令的成功与失败。当if后面的命令执行成功，就会执行then后面的命令。

## 使用test命令
经常和if一起使用的是test命令。test命令执行各种检查比较，这个命令有两种等价的形式，一种是
``` bash
test expression
```
一种是更流行的
``` bash
expression
```
expression表示的是表达式，结果是true或者false.

1. 文件表达式
``` bash
file1 -ef file2#两个文件有相同的信息节点编号（通过硬链接指向同一个文件）
file1 -nt file2 #file1更新
file1 -ot file2 #file1更旧
-d file #file存在且是目录
-e file #file存在
-f file #file存在且是普通文件
-L file #file存在且是符号链接
```
还有很多就部列举了

2. 字符串表达式
``` bash
string #不为空
-n string #长度大于0
-z string #长度等于0
string1==string2 string1=string2 #一个等号和两个都可以。表示相等
string1 != string2
string1>string2 #排序的时候，string在string2的后面
string1 <string2
#警告，在是哟个test命令的时候要用引号或反斜杠对<>进行处理，不然会被shell扩展为重定向符号
```

3. 整数表达式
``` bash
int1 -eq int2
int1 -ne int2
int1 -le int2 #int1<=int2
int1 -lt int2 #int1<int2
int1 -ge int2 #int1>=int2
int1 -gt int2 #int1>int2
```
## 更现代的test命令
bash的最新版本增加的符合命令，相当于增强test命令，语法为`[[ expression ]]` 。[[]]命令和test命令类似（支持所有的表达式），不过增加了一个很重要的新字符串表达式
```
string1=~regex
```
如果string1能够和正则表达式regex匹配，就返回true。比如进行整数比较的时候先判断除了数字外是不是有别的字符出现：
``` bash
INT=-5
if [[ $INT =~ ^-?[0-9]+$ ]];then
...
```
另外一个特性是==操作符支持模式匹配，，就像路径扩展名一样
``` bash
if [[ $FILE==*.png ]];then
...
```

## (()) 为整数设计
除了[[]] 外bash提供了(())来操作整数，支持完成的算数运算。当结果是非零的时候代表true

## 组合表达式
用逻辑把表达式组合起来，逻辑操作符如下
| 操作符 | test | [[]] and (()) |
| --- | --- | --- |
|AND | -a | && |
| OR | -o | \|\| |
| NOT | ! | ! |
比如检测整数是否在某个范围内
``` bash
if [[ INT -le MAX &&  INT -ge MIN]];then
...
```
可以用圆括号对表达式进行分组。如果用在test表达式即单个的括号里面的话需要用转义字符

## 控制运算符：另一种方式的分支
bash提供的&&和||可以用作分支。很好理解，a||b,那么只有a失败的时候才会执行b。a&&b，只有a成功的时候才能执行b。比如
``` bash
cd dir1 || mkdir dir1
```
如果没有dir1就会创建。也可以在脚本中写
``` bash
[ -d temp ] || exit 1
```
如果没有目录temp,就退出且退出状态为1


# 读取键盘输入
是时候给程序添加可以和用户交互的功能了。

## read 从标准输入读取输入值
内嵌命令read的作用就是读取一行标准输入。可以用于读取键盘输入值或者用重定向读取文件的一行。<br>
read [ -*options* ][ *variable* ...]<br>
options有多个可用选项，而variable是一道多个用来存放输入的变量。如果没有提供任何此类变量，则由shell变量REPLY来存储数据行。
| 选项 | 描述 |
| --- | --- |
| -a array | 把输入值从索引0的位置开始赋值给array |
| -d delimiter | 用字符串delimiter的第一个字符标志输入的结束，而不是新一行的开始 |
| -e | 使用Readline处理输入。此命令使用户能使用命令行模式的相同方式编辑输入 |
| -n num | 从输入中读取num个字符  |
| -p prompt | 使用prompt字符串提示用户进行输入 |
| -r | 原始模式。不能将后斜线字符翻译为转义码 |
| -s | 保密模式。 |
| -t seconds | 超时，在seconds秒后结束输入 |
| -u fd | 从文件说明符fd 读取输入而不是标准输入 |

基本上read命令把标准输入的字段值分别给指定变量，比如提示输入一个整数：
``` bash
echo -n "Please enter an integer -> "
read int
```
使用带-n选项的echo命令可以不换行，作为提示符。可以接受多个变量
``` bash
read var1 var2 var3 var4 var5
```
如果输入少于五个，后面的值全都是''。如果多了，那么最后一个变量会包括后面所有多出来的，如输入'1 2 3 4 5 6 7',var5='5 6 7'。如果read后没有变量即
``` bash
read 
```
那么输入的值就给REPLY了。
1. 选项 -p设置提示符，-t限制时间，-s秘密输入
2. 使用IFS间隔输入字段，shell的间隔有空格、制表符和换行符。可以通过改变IFS值来控制。比如csv用逗号间隔等等。下面尝试读取以:为间隔的输入
    ``` bash
    OLD_IFS="$IFS"
    IFS=":"
    read user ped uid gid name home shell <<< "$file_info"
    IFS="$OLD_IFS"
    ```
    先储存旧的值，给新的值，读取后恢复为原来的值。shell也允许命令执行之前对一到多个变量赋值，这些操作改变的是暂时的，也就是说上面的可以写为
    ``` bash
    IFS=":"     read user ped uid gid name home shell <<< "$file_info"
    ```
    这样更简单。操作符`<<<` 象征一条嵌入字符串，实际上就是把file_info中的内容读出来传输给read。为什么不能写成管道的形式？<br>
    echo "$file_info" | IFS=":" read...<br>
    因为read不可以重定向。这是因为shell处理管道的方式是，管道创建一个子shell,子shell复制了原来的shell环境，执行完命令后销毁复制出的环境。所以子shell永远不改变父类进程的shell环境。而read进行赋值，变量会成为环境的一部分。

## 验证输入
用条件判断验证用户的输入是否符合要求

## 菜单
菜单驱动是一种常见的交互方式，呈现一系列选项请求用户选择。

# WHILE与UNTIL循环

## while
格式 while *commands*;do *commands*;done<br>
实例
``` bash
count=1

while [ $count -le 5 ]; do
    echo $count
    count=$((count+1))
done
echo "Finished"
```

## 跳出循环
continue和break
## until
while命令在退出状态不为0的时候终止循环，而until正好相反。除此之外二者很相似。实例
``` bash
count=1

until [ $count -gt 5 ]; do
    echo $count
    count=$((count+1))
done
echo "Finished"
```

## 使用循环读取文件
while和until可以处理标准输入，这让while和until处理文件成为可能，如下
``` bash
#!/bin/bash

#while-read:read lines from a file

while read distro version release; do
    printf "Distro :%\tVersion: %s\tReleased: %s\n" \
        $distro \
        $version \
        $release
done < distros.txt
```
在done后面接重定向就可以循环使用read命令读取重定向文件中的字段

# 故障诊断
## 语法错误
下面列举一些常见错误
1. 引号缺失<br>
缺了引号的一般，报错的并不是缺少了引号的行，而是之后的代码。

2. 符号缺失冗余<br>
if或者while的结构不完整。比如if后面的test部分结尾没写引号，那么单词then也会被添加到参数中，这没有违背语法。会在遇到else的时候报错。

3. 非预期展开<br>
比如
    ``` bash
    number=
    if [$number=1];then
    ...
    ```
    这时展开效果为[=1],语法是错误的。可以在test命令中用双引号来避免这个错误 ["$number"=1],结果就是[""=1]

## 逻辑错误
逻辑错误不会阻碍脚本运行，可能发生很多逻辑错误影响结果：
* 条件表达错误。写了不正确的if,then,else
* 从1开始错误。计数应该从0开始
* 非预期情形。比如非预期的展开

1. 防御编程<br>
对脚本所用的命令进行仔细的评估，对各种情况进行条件判断
2. 输入值验证

## 测试
1. 桩 stub<br>
2. 测试用例

## 调试
1. 找到问题域<br>
把问题相关的脚本隔离出来，比如用注释
2. 追踪<br>
比如脚本有的部分根本没有执行，可以在某些特定的地方把错误信息发送到错误流
    ``` bash
    echo  'error' > &2
    ```
    bash也提供了一种追踪方法，直接使用-x选项激活对整个脚本的追踪，激活之后可以查看变量展开的执行情况。要对脚本的某个部分进行追踪，可以直接写一行`set -x`

3. 运行过程中变量的检验<br>
通常通过echo输出来实现。


# 流控制:case
## case
语法如下:
```
case word in
[ pattern [| pattern]...) commands ;;]...
esac
```
可以使用case来简化多个if语句，一个例子：
``` bash
case $REPLY in
    0)  echo ...
        ;;
    1) ...
        ;;
    2)  ...
        ;;
    *)  ...
        ;;
esac
```

1. 模式<br>
和路径名展开一样，case使用)字符结尾模式，有一些有效模式如下：
    ```
    a) 关键字为a则吻合
    [[:alpha:]]) 关键字为单个字母则吻合
    ???) 关键字为三个字符则吻合
    *.txt) 关键字以.txt结尾吻合
    *) 与任何关键字吻合，一般放在最后作为defaultr
    ```

2. 多个模式组合<br>
可以使用竖线作为分隔符来组合多个模式，模式之间是或的关系。这对一些类似同时处理大写和小写字母的情况很有帮助，如
    ```
    case $REPLY in
        q|Q)    ,,,
                ;;
        ...
    ```

# 位置参数
之前一致没有设计程序接收和处理命令行选项及实参的能力，这里将会介绍

## 访问命令行
shell提供了一组名为位置参数的变量用于储存命令行中的关键字，这些变量分别命名为0~9，用以下方式展示：
``` bash
#!/bin/bash

echo "
\$0=$0
\$1=$1
...
"
```
没有任何命令行实参的情形下，$0也是有值的，其值就是命令行的第一项数据及程序路径名。
* 注意，使用参数扩展技术用户实际可以获取多于9个参数，为了标明一个大于9的数字，将数字用大括号括起来，如${10}

1. 确定参数数目<br>
shell提供了变量$#给出命令行参数的数目
2. shift 处理大量的实参<br>
如何处理几十上百个实参呢？有一种比较笨的办法，每次执行shift后所有参数的值就下移一位，这样只要处理一个参数（除了$0以外，因为$0值恒定）就可以完成全部任务。示例如下
    ``` bash
    #!/bin/bash
    count=1

    while [[ $# -gt 0]]; do
        echo "Arguments $count = $1"
        count=$((count+1))
        shift
    done
    ```
    每次执行shift,$2的值就给了$1,$3给了$2,依次类推

3. 简单的应用程序<br>
不适用shift也可以写出使用任意位置参数的应用程序。<br>
basename $0。该命令可以去掉路径名的起始部分，只留下基本的文件名。

4. shell函数中使用位置参数<br>
位置参数同样适用于shell函数的实参传递。参数FUNCNAME表示当前函数的函数名。变量$0始终表示的是命令行第一项路径全名，函数接受的参数从$1开始
``` bash
file_info(){
        if [[ -e $1 ]]; then
            echo -e "\nFile Type:"
            file $1
            echo -e "\nFile Status:"
            stat $1
        else
            echo "$FUNCNAME: usage: $FUNCNAME file" >&2
            return 1
        fi
}
```
注意echo -e表示激活转义字符！

## 处理多个位置参数
有时候把所有位置参数作为一个整体处理会比较方便。shell为这项功能提供了两种特殊的参数。<br>
$* 可以扩展为从1开始的参数序列，当包括双引号的时候，扩展为双引号引用的全部位置参数构成的字符串。<br>
$@ 扩展为从1开始的位置参数列，包括双引号在内的时候每个位置参数扩展为双引号引用的单独单词<br>
演示脚本查看p382。把"word" "word with space"传入，$\*和$@将其全部拆开（4个参数）;"$*"把参数合并起来（1个参数）;"$@"依旧保留连个参数。这么看来"$@"在多数情况可以令人满意。

## 更完整的程序
重新回到sys_info_page的编辑工作，添加如下命令行选项
* 输出文件。-f file或--file file
* 交互模式 提示用户输出文件的命令并检验文件是否已经存在。形式为-i或--interactive
* 帮助，形式为-h或--help

首先是程序名字参数，用PROGNAME,
```
PROGNAME=$(basename $0)
```
编写usage函数，使用help选项或者未知选项的时候输出相关信息。使用循环读取位置参数，用case检验是否匹配，发现匹配的参数就做出相应行为

# 流控制：for
## for 传统shell形式
for *variable* [in *words*]; do<br>
    *commands*<br>
done<br>
其中variable是在循环时会增值的变量名，words是一列将值按顺序赋给变量variable的可选项。commands是每次都会执行的指令。如
``` bash
for i in A B C D;do
    echo $i
done
```
就会分别输出A b c d。for真正强大的功能在于创建字符列表的方式有多种。比如使用花括号扩展
``` bash
for i in {A...D};do
    echo $i
done
```
或者使用路径名扩展
``` bash
for i in *.txt;do
    echo $i
done
```

## for C语言形式
很多bash版本加入了第二种for语法，类似C语言。<br>
for (( *expression1;expression2;expression3* ));do<br>
    *command*<br>
done<br>
如
``` bash
for ((i=0; i<5; i=i+1)); do
    echo $i
done
```
每当需要数值序列的时候，C语言形式就可以发挥作用了。


