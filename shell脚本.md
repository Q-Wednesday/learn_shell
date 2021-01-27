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

