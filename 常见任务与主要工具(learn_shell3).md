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

# 归档与备份
## 文件压缩
数据压缩，是移除数据冗余信息的过程。比如一张100x100的纯黑图像，包含了完全冗余的数据，可以压缩成数字30000（3个通道）和一个0表示，这种方式成为游程编码，是最基本的压缩技术。如今的压缩技术更复杂，但是基本目标一直是消除冗余信息。<br>
压缩算法一般分为有损和无损两种。JPEG和MP3就是有损压缩的实例。下面讨论仅讨论无损压缩，因为很多数据无法容忍损失

1. gzip 文件压缩与解压缩<br>
gzip命令用于压缩一个或多个文件，执行命令后，原文件会被其压缩文件取代。gunzip用于把压缩文件还原成原来的文件。压缩和解压缩后权限和时间戳是保持一致的。gzip有以下选项<br>
-c 把输出内容写到标准输出端口并保持源文件，可用--stdout或--to-stdout代替。<br>
-d 解压缩，加上后相当于gunzip<br>
-f 强制压缩，源文件的压缩版本已经存在也可以压缩<br>
-l 列出压缩文件的统计<br>
-r 递归<br>
-t 检验压缩文件完整性<br>
-v 压缩时显示信息<br>
-number 设置压缩级别，1表示速度快比例小，9表示速度慢压缩比例大。默认比例6<br>
zcat和cat功能相同，只是zcat对压缩文件进行操作。同样也有zless。

2. bzip2 牺牲速度换取高质量的压缩<br>
功能和gzip相仿，压缩算法不同。压缩后文件后缀是.bz2。除了-r选项不可用外其他与gzip相同。解压工具为bunzip2,bzcat。还有专门的bzip2revover,可以恢复损坏的文件。

## 文件归档
归档是和压缩操作配合使用的文件管理任务。归档是把众多文件合并成一个大文件的过程，通常作为系统备份的一部分。

1. tar 磁带归档工具<br>
tape archieve的缩写。tar归档文件可以由许多独立的文件、一个或多个目录层次或者两者混合组成，用法为：<br>
tar *mode* [ *options* ] *pathname* ...<br>
部分模式：c是创建文件，x是从归档文件中提取文件，r是在归档文件末尾追加指定路径名，t是列出归档文件内容。<br>
指定路径名的时候通常不支持通配符，GNU版本的tar命令通过--wildcards选项支持通配符。<br>
tar命令创建归档文件常用find命令辅助，先用find查找然后用tar压缩，如下：
    ``` bash
    find dir -name 'file-A' -exec tar rf dir.tar '{}' '+'
    ```
    实现了把文件file-A压缩进dir.tar中。是增量备份。tar命令可以利用标准输入、输出
    ``` bash
    find dir -name 'file-A' | tar cf - --files-from=- | gzip > dir.tgz
    ```
    先用find找到文件，把文件交给tar处理，文件名处指定-，表示标准输入输出文件，--files-from表示tar命令从问卷文件中读取文件路径名列表（简写为-T）归档后的文件由gzip压缩。现代GNU版本的tar命令提供gzip+z选项和bzip2+j选项直接实现上述功能
    ``` bash
    find dir -name 'file-A' | tar czf dir.tgz -T -#用gzip压缩
    find dir -name 'file-A' | tar cjf dir.tgz -T -#用bzip2压缩
    ```
    可以利用tar在远程主机和本地间传输文件：
    ```
    ssh remote 'tar cf - Documants' |tar xf -
    ```

2. zip 打包压缩文件<br>
zip既是压缩工具也是归档工具，windows用户很熟悉这个。Linux用户主要用zip程序与windows系统交换文件，而不是将其用于压缩与归档。<br>
zip *options* *zipfile* *file*...
<br>选项-r表示递归压缩。用unzip提取文件中内容。unzip -l只列出归档文件内容而不提取，可以增加-v选项获得更详细的列表。zip命令也可以利用标准输入输出，尽管作用不大。使用-@选项来输送文件。


## 同步文件和目录
1. rsync 远程文件、目录同步<br>
该命令通过运用rsync远程更新协议，同步本地系统和远程系统上的目录。该协议允许rsync命令快速检测本地文件与远程系统上两个目录之间的不同，以最少数量的赋值动作完成同步。因为比起其他复制命令更快更经济。调用方式：
```
rsync options source destination
```
source和destination中必须至少有一个是本地的。如把本地的dir文件夹备份到foo
```
rsync -av dir foo
```
-a选项用于归档，-v用于详细输出。此时在foo中会生成dir的镜像备份。把系统备份到外部硬盘
```
sudo rsync -av --delete /etc /home /usr/local /media/BigDisk/backup
```
其中delete选项会删除残留在备份设备中而源设备不存在的文件。


2， 在网络上使用rsync命令
使用--rsh=ssh选项，并在destination路径名前指定远程主机名，如`remote-sys:/destination`
