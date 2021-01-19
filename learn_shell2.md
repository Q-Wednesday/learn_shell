这里对应Linux命令行大全的第二部分！配置与环境
# 环境
尽管大多数程序使用配置文件来储存程序的设置，但是也有一些程序会查找环境中的变量来调整自己的行为。用户可以用环境来自定义shell

## 环境中储存的是什么
shell环境中储存了两种类型的数据，分别为环境变量和shell变量，不过它们在bash中没有什么区别。shell变量是bash存放的少量数据，环境变量就是除此之外的其他变量。除了变量之外shell也储存了一些编程数据(programmatic data)，也就是别名和shell函数。

1. 检查环境<br>
需要使用set或者printenv命令。set会显示shell变量和环境变量，printenv只显示环境变量。而且set的输出结果是按照字母顺序排列的。使用printenv 变量名 可以列出特定的值。如
    ```
    printenv USER
    ```
    想查看单个变量的值，也可以直接用<br>
    ```
    echo $HOME
    ```
    查看别名，使用不带参数的alias
    ```
    alias
    ```
2. 一些有趣的变量<nr>
环境中包含了许多变量比如SHELL就是本机shell名称，LANG是语言字符集，TZ为时区等等。需要的时候可以去查询

## 环境是如何建立的

用户登陆后bash启动并读取一系列启动文件。

1. login和non-login shell<br>
login shell即提示用户输入用户名和密码的绘画，如虚拟控制台会话。而在GUI中启动的终端就是non-login shell会话。
login shell读取一个或多个启动文件<br>
/etc/profile 适用于所有用户<br>
~/.bash_profile 用户个人启动文件，可以扩展或重写全局配置脚本<br>
~/bash_login 如果上面的文件缺失就读取这个<br>
~/.profile 上面两个文件都缺失才读取<br>

而non-login shell读取<br>
/etc/bash.bashrc 适用于所有用户<br>
~/.bashrc 用户个人启动文件<br>
除了读取上述文件，non-login shell会继承父进程的环境，父进程通常是一个login shell

2. 启动文件中有什么<br>
#开头的是注释。里面有一些脚本语言写的语句比如条件判断，赋值<br>
shell如何找到命令？在这些启动文件中会设置PATH的值

## 修改环境
知道了上面的知识，用户就可以自己修改启动文件来定义环境了

1. 修改哪些文件？

2. 文本编辑器<br>
为了编辑shell的启动文件，以及其他配置文件，我们都需要文本编辑器。为什么一个系统中会有那么多编辑器？因为程序员热衷于编写文本编辑器来拥有符合自己工作方式的工具。<br>
图形界面的编辑器有gedit(GNOME配备)，kedit,kwrite,kate(KDE配备)<br>
还有基于文本的编辑器，nano,vi和emacs。nano是很简易的，vi(现在一般都用vim了)是传统的文本编辑器。emacs是一个庞大的、万能的、可以做任何事情的编辑环境，尽管emacs仍然可以用，但是Linux系统很少默认安装emacs。

3. 使用nano<br>
Ctrl-X是退出程序，Ctrl-O是保存。

4. 激活更改<br>
只有在启动shell的时候才会读取配置文件，如果对配置文件进行了修改可以强制读取
    ```
    source .bashrc
    ```
    

    