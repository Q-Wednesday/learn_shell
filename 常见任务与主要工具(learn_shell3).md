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

