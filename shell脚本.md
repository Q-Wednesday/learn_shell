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
