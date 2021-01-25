# 常见任务与主要工具

# 软件包管理
在Linux早期，安装软件需要先下载源代码然后进行编译，现在很多人都是安装Linux经销商发布的软件包来满足需求了。

## 软件包系统
不同Linux发行版使用不同的软件包系统，原则上适用于一种发行版的软件包与其他的版本不兼容。多数Linux发行版就两种软件包技术阵营：Debian的.deb和Red Hat的.rpm。

## 软件包系统工作方式
Linux所有软件都可以在网上找到，多数以软件包文件形式由发行商提供。其余的可以手动安装源代码。

1. 软件包文件<br>
一个包可能包含了大量程序以及支持这些程序的数据文件，包文件既包含了安装文件，又包含了文档。

2. 库<br>
中心库一般包含了成千上万个软件包，每一个都专门为该发行版建立和维护。

3. 依赖关系<br>
几乎没有任何一个程序是独立的，与之相反，程序之间相互依赖彼此完成既定工作。现代软件包管理系统都提供依赖性解决策略，从而确保用户安装了软件包同时也安装了依赖。

4. 高级和低级软件包工具<br>

执行安装、删除等事务的是低级工具(如dpkg,rpm)，进行元数据搜索以及提供依赖性解决的高级工具。（如apt-get,aptitude,yum）

## 常见软件包管理任务
由于目前用的是ubuntu，所以全以Debian的举例
1. 在库里查找软件包<br>
apt-get update或apt-cache search string

2. 安装库中的软件包<br>
apt-get install package_name

3. 安装软件包文件中的软件包<br>
dpkg --install package_file

4. 删除软件包<br>
apt-get remove package_name

5. 更新库中的软件包<br>
apt-get update;apt-get upgrade

6. 更新软件包文件中的软件包<br>
直接安装新的取代原来的即可 dpkg --install package_file

7. 列出已安装的软件包列表<br>
dpkg --list

8. 判断软件包是否安装<br>
dpkg --status package_name

9. 显示已安装软件包的相关信息<br>
apt-cache show package_name

10. 查看某个文件由哪个安装包得到<br>
dpkg --search file_name

# 文件搜索
## locate 比较简单的方式查找文件
比如想找到用zip字符串开头的程序，由于想找到的是程序，可以搜索`bin/zip` ，尝试
```
locate bin/zip
```
locate程序已经用了很长时间，出现了多种衍生体如slocate何mlocate。<br>
locate搜索的数据从何而来？系统刚刚装好的时候locate命令不能工作，第二天就可以了。因为Locate搜索的数据库由一个叫updatedb的程序创建，通常该程序作为一个cron定时任务执行。所以locate搜索数据并不是连续更新的，locate命令查找不到非常新的文件。解决方法是`sudo updatedb `

## find 比较复杂的方式查找文件
locate只能根据文件名，find可以根据各种属性在既定目录查找。<br>
最简单的用法就是给定一个或多个目录名作为搜索范围，如列出系统主目录的
```
find ~
```
对于活跃用户，主目录文件会跟多，可以用wc程序计算文件总量
```
find ~ | wc -l #数出输出行数就可以了
```
find可以综合多个选项来实现高级的文件搜索，下面一一介绍

1. test选项<br>
-type指定搜索范围，d表示目录，f表示普通文件。c为字符设备文件，b为块设备文件，l是符号链接。<br>
此外还可以通过其他test参数实现文件大小何文件名的检索，-name可以搜索文件名如`-name "*.JPG"` 注意要用双引号来避免shell路径名扩展。<br>
-size选项表示大小，+1M表示大于1M。还有一些别的计量单位：b 512字节，c字节，w两个字节的字，k KB,G GB。<br>
还有多种test参数，可以去查看更多。<br>
    ### 操作符
    test参数之间的逻辑关系是可以用操作符描述的，比如要招权限不是0600的文件和权限不是0700的目录，可以使用 -or 操作，同样的还有 -or,-not,(),()是用来确定运算优先级的，为了避免被识别为shell中的特殊含义，要用转义字符，写法如下
    ``` bash
    find ~ \( -type f -not -perm 0600\) -or \( -type d -not -perm 0700\)
    ```

2. action 选项<br>
找到需要的文件后可以使用用户自定义动作，先看预定义动作：-delete 删除匹配文件。-ls 对匹配文件进行ls。-print 把匹配到的文件全路径以标准输出形式输出。 -quit 匹配成功就退出。<br>
与test参数比，actions的参数更多，可以参看man手册来获取更全面的信息。一个示例
    ``` bash
    find ~ -print
    ```

    ### 用户自定义操作
    除了已经预定义的操作，也可以任意条用用户想执行的命令，传统方法是-exec操作
    ``` bash
    -exec command () ;
    ```
    command是要执行的操作名，{}代表当前路径。分号是必要的表示命令结束的分隔符。比如使用-exec来完成删除，可以写
    ``` bash
    -exec rm '{}' ';'
    ```
    由于括号和分号有特殊含义，所以要用引号引起来。当然也可以交互式执行，使用-ok选项，每次操作都会询问用户
    ``` bash
    find ~ -type f -name 'foo*' -ok ls -l '{}' ';'
    ```

    ### 提高效率
    使用-exec的时候，每次匹配到文件都会执行一次，有的时候用户可能更希望调用一次命令完成对所有文件的操作。<br>
    可以直接在命令后加上+，就会找到匹配文件后执行一次。
    ``` bash
    find ~ -type f -name 'foo*' -exec ls -l '{}' ';' +
    ```
    同样可以用xargs得到标准信息然后作为某条指令的输入，如
    ``` bash
    find ~ -type f -name 'foo*' -print | xargs ls -l
    ```
    但是这不代表可以无限输入，可能会由于参数太多无法承受。可以在xargs --show--limits查看最大能承受的参数数量。
3. opyion选项<br>
用于控制搜索范围，比较常用的有-depth,引导find先处理目录内文件;-maxdepth levels 陷入目录的最大级数;-mindepth levels;-mount ；-noleaf

