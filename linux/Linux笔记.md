# Linux笔记

## 一、linux目录

linux一切皆文件:

* ==/bin==

  > 是Binary的缩写，这个目录存放着最经常使用的命令，ls，cat......

* /sbin

  > 存放着系统管理员使用的系统管理程序

* ==/home==

  > 存放着普通用户的主目录，在Linux中每个用户都有一个自己的目录，一般该目录名是以用户的账号命名

* ==/root==

  > 该目录为系统管理员的用户主目录

* /lib

  > 系统开机所需要最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库

* ==/etc==

  > 所有的系统管理所需要的配置文件和子目录 my.conf

* ==/usr==

  > 非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似windows下的program files目录

* /boot

  > 存放的是启动linux时使用的一些核心文件，包括一些连接文件以及镜像文件

* ==/media==

  > linux系统会自动识别一些设备，例如U盘、光驱等，识别后，linux会把识别的设备挂载到这个目录下

* /dev（device->设备）

  > 类似于Windows的设备管理器，把所有的硬件用文件的形式存储（一切皆文件）

* ==/mnt==

  > 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将外部的存储挂载在/mnt/ 上，然后进入该目录就可以查看里面的内容了。 d:/myshare

* /opt

  > 这是主机额外安装软件所摆放的目录，如安装Oracle数据库就可放到该目录下。默认为空，一般来讲，要把系统额外安装的安装软件先放到这个目录下

* ==/usr/local==

  > 这是另一个给主机额外安装软件所安装的目录。一般是通过编译源码方式安装的程序

* /var

  > 这个目录种存放着在不断 扩充着的东西，习惯性将经常被修改的目录放到这个目录下

## 二、vim基本使用

​		在linux下使用==vim xxx.xxx==表示使用vim编辑一个文件

### 1.一般模式

​		打开一个文件后，进入一般模式，此模式下可以进行文件替换，整行插入、删除等。

### 2.插入模式

​		在一般模式下按i或a等进入插入模式，此模式下可以对文件进行编辑。

>  插入模式下按ESC回到一般模式

### 3.命令模式

​		一般模式下，使用   “:“   表示进入命令模式，==”:“ 后跟命令，====有wq(写入文件并退出)，q(退出)，q!(强制退出，并且不保存)。==

### 常用快捷键

* 复制/删除

> 一般模式下，==yy==复制当前行，复制当前行的下面n行，输入==”n“yy==
>
> 输入==dd==删除当前行，输入==”n“dd==删除当前行后的n行

* 粘贴

> 一般模式下，输入==p==将复制的内容粘贴到当前行

* 定位

> 一般模式下，输入==G==定位到文件末行
>
> 输入==gg==定位到文件首行
>
> 输入==n==后再输入==shift+g==定位到文件第n行

* 查找

> 命令模式下，/后跟关键词（大小写区分），就能在文件中搜索关键词，输入n寻找下一个

* 显示/不显示行号

> 命令模式下，==:set nu显示文件行号==
>
> ==:set nonu不显示文件行号==

* 撤销

> 一般模式下，==按u撤销动作==

### vim键盘图

![查看源图像](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/1-900-png_6_0_0_0_0_2160_1350_2160_1350-1440-0-0-1440.jpg)

## 三、linux开关机

* shutdown -h now（h代表halt，即停止的意思）

> 代表现在立刻关机

* shutdonw -h n

> 代表n分钟后关机

* shutdown -r now（r代表reboot，重新启动的意思）

> 代表现在立刻重启

* sync

> 代表将内存中的数据同步到磁盘

## 四、用户操作

### 1. 添加/删除用户

> 使用==useradd 用户名 -p密码==，给系统添加用户及设定密码
>
> 使用==userdel 用户名==，删除系统用户，但是保留家目录（/home下的用户目录）
>
> 使用==userdel -r 用户名==，删除系统用户以及家目录

### 2. 登入管理员及注销

> 使用==su 管理员用户名==，切换管理员权限
>
> 使用==exit退出管理员权限==，使用logout注销登录账户

### 3. 修改密码

> 使用==passwd==修改当前用户的密码
>
> 使用==passwd 用户名==，修改指定用户的密码

### 4.查询用户信息

> 使用==id 用户名==，查询用户信息
>
> 使用==who am i==，查询当前用户信息

### 5. 组管理

​	在创建用户没有指定组时，会默认创建一个与用户名相同的组，将要创建的用户加入进去。

> 创建用户时加上==-g 组名==，将创建的用户加入指定组中，
>
> 如：==useradd -g mygroup 用户名==

### 6. 用户修改

> 使用==usemod==修改用户。
>
> 如：==usermod -g 组名 用户名==，修改用户的组

## 五、 用户和组的相关文件

### 1. /etc/passwd文件

用户的配置文件，记录用户的各种信息，

每行的含义： ==用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell==

### 2. /etc/shadow文件

口令的配置文件

每行的含义：登录名:==加密口令（加密过的用户密码）:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志==

### 3. /etc/group文件

组的配置文件，记录linux包含的组的信息

每行含义：==组名:口令:组标识号:组内用户列表==

## 六、linux运行级别

### 1.运行级别说明

* 0：关机
* 1：单用户（找回丢失密码）
* 2：多用户状态没有网络服务
* ==3==：多用户状态有网络服务
* 4：系统未使用保留给用户
* ==5==：图形界面
* 6：系统重启

> 使用init [0123456]切换系统运行级别

## 七、linux常用指令

### (1)  基础指令

#### 1.   cd

> ==可以到绝对路径，也可以写相对路径==

#### 2.   ls

> 参数 -l：列出文件详细情况
>
> 参数 -a：列出所有文件，包括隐藏文件（以“.”开头的文件）
>
> 如：ls -al /home （表示列出/home下的所有文件的详细信息）

#### 3.   man/help

> ==查看一个指令的帮助==
>
> 如：man ls，help cd

### (2)  文件目录指令

#### 1.   pwd

> ==显示当前工作目录的绝对路径==

#### 2.   mkdir

> ==创建目录（文件夹）==
>
> 如在home下创建hoshino：mkdir /home/hoshino
>
> 如果是创建多级目录可添加选项-p，如在/下创建home下的hoshino：mkdir -p /home/hoshino

#### 3.   touch

> ==创建文件==
>
> 如：touch hello.txt

#### 4.   rmdir/rm

> ==删除目录==
>
> 可删除单级目录，如在home下删除hoshino（此时hoshino下没有文件）：rmdir /home/hoshino
>
> 如果删除的目录下有文件则无法删除
>
> 可以添加选项-rf（慎重!!!!）：rm -rf /home（-r是递归 -f是强制）

#### 5.   cp

> ==拷贝==
>
> 基本语法：==cp [-option]  resource  desc==
>
> 如，cp /hello.txt /home/bbb。将/下的hello.txt文件拷贝到/home/bbb下
>
> cp -r /home/bbb /opt。将/home/bbb拷贝到/opt下，-r表示递归
>
> \cp -r /home/bbb /opt。\cp表示强制覆盖，这样不用每次覆盖文件都确认一遍

#### 6.   mv

> ==移动文件与目录，或重命名==
>
> 重命名： ==mv oldNameFile newNameFile==（两文件目录相同）
>
> ==如：mv /home/cat.txt /home/pig.txt。==将home下的cat.txt改名为pig.txt
>
> 移动文件：==mv   /temp/movefile   /targetFolder==
>
> mv    /home/pig.txt    /root/cat.txt。移动并且重命名。

#### 7.   cat

> ==查看文件内容==
>
> 基本语法：==cat   [-option]    要查看的文件==
>
> 常用选项：-n，显示行号
>
> 为了浏览方便，通常带上管道命令：|more

#### 8.   more/less

> 都是查看一个文件用的
>
> 基本语法：==more   文件名/less   文件名==

#### 9.   echo

> 输出信息到命令行
>
> 如输出环境变量：==echo   $PATH==

#### 10.   head/tail

> head输出文件前n行的信息，tail输出尾信息
>
> head  -n   5   /etc/profile。将/etc/profile文件的前5行输出
>
> tail    -n    5    /etc/profile。

#### 11.   >与>>

> \> ：输出重定向
>
> \>>：追加
>
> 基本语法：==ls   -l   >文件。（列表的内容写入文件中（覆盖写））==
>
> ​				   ==ls    -al   >> 文件。（列表的内容追加写到文件末尾）==
>
> ​				   ==cat   文件1> 文件2。（将文件1的内容覆盖到文件2）==
>
> ​				   ==echo   "内容">>文件。（将内容追加写入文件）==

#### 12.    ln

> 软连接
>
> 基本语法：==ln  -s   [原文件或目录]   [软链接名]==。（给原文件创建一个软链接（相当于快捷方式））
>
> 如：ln   -s   /root    /home/myroot。在/home/下建立一个叫myroot的软链接，指向/root。
>
> 删除软链接跟删除文件一样

#### 13.  history及!

> ==history查看已经执行过的指令==
>
> 基本语法：history。查看所有已经执行过的指令
>
> 显示最近用过的10条指令：history 10。
>
> ==!执行以前执行过的指令==，如有一条指令编号143，为ls  /root
>
> 则可以!143，表示执行ls  /root

### (3)  时间日期指令

#### 1.    date

> date 查看当前时间，可以格式化
>
> 如：date "+%Y-%m-%d   %H-%M-%S"进行格式化输出当前时间

#### 2.    cal

> 输出日历
>
> cal  年份：输出整年的日历

### (4)  搜索查找指令

#### 1.    find

> 从指定目录向下递归查找文件或目录，显示在终端
>
> 基本语法：==find   [搜索范围]   [选项]==
>
> 按文件名查：find  /home  -name  *.txt。查在/home下的所有.txt结尾的文件
>
> 按拥有者查： find   /opt  -user   nobody。查找/opt下的用户名为nobody的文件
>
> 按大小查：find   /   -size   +200M。查找 / 下的大于200M的文件
>
> ​					+大于，-小于，不写就等于，单位有k，M，G。
>
> 按时间查：find  /home  -atime +10  -name  "*.tar.gz" 。查找十天以前的名字后缀为.tar.gz的文件

#### 2.    locate

locate指令可以利用事先建立的系统中所有文件名称及路径的locate数据库快速定位文件路径，使查找文件无需遍历整个文件系统，为了保证查询结构准确度，管理员必须定期更新locate时刻。

==由于locate指令基于数据库进行查询，所以第一次运行前，必须使用updatedb指令创建locate数据库==

> locate 搜索文件

#### 3.    which

> 查看某个指令在哪个目录下
>
> 查看ls指令在哪个目录：which  ls

#### 4.   grep与管道符号 “|”

grep可以查找文件里的内容

> 基本语法：grep  [-option]  "查找内容"  [文件]

grep也可以和管道搭配使用。

如，在/home/hello.txt中查找hello，并表示行号，可以：

> 1.  grep  -n  "hello"  /home/hello.txt
> 2. cat  /home/hello.txt  |  grep  -n  "hello"

选项：  -n，显示匹配行及行号

​			  -i，忽略字母大小写	

### (5)   压缩与解压

#### 1.   gzip与gunzip

gzip是将文件打包为tar.gz格式的压缩文件

gunzip从tar.gz包中解压出某个文件

gzip是一种文件压缩工具（或该压缩工具产生的压缩文件格式），它的设计目标是处理==单个==的文件。

> 压缩文件：gzip   Hello.java
>
> 解压文件：gunzip   Hello.java

#### 2.   zip与unzip

与gzip不同，zip只是一种数据结构，跟rar同类型。zip是适用于压缩==多个==文件的格式（相应的工具有PkZip和WinZip等）。

> 基本语法：zip   [选项]   XXX.zip   要压缩的内容
>
> ​				    unzip   [选项]   XXX.zip

> 如：
>
> 将/home下的所有文件/文件夹进行压缩成myzip.zip：
>
> ==zip   -r   myzip.zip  /home==
>
> 将myzip.zip解压到/opt/tmp目录下
>
> unzip  -d /opt/tmp  /home/myzip.zip

gzip与zip压缩算法不一样。

#### 3.   tar

tar指令是打包指令，最后打包后的文件是.tar.gz的文件

> 基本语法：tar  [选项]  XXX.tar.gz  打包的内容
>
> 选项： -c   产生 .tar打包文件
>
> ​			 -v  显示详细信息
>
> ​			 -f  指定压缩后的文件名
>
> ​			 -z  打包同时压缩
>
> ​			 -x  解包 .tar文件
>
> 一般来说：
>
> 1. ==如果是.tar.gz压缩包，压缩选项直接-zcvf，解压选项-zxvf就行==。
> 2. ==如果是.tar压缩包，则选项去掉z即可==。

> 将/home/pig.txt与/home/cat.txt压缩成pc.tar.gz：
>
> tar  -zcvf  pc.tar.gz  /home/pig.txt  /home/cat.txt
>
> 将home文件夹压缩成 myhome.tar.gz：
>
> tar  -zcvf  myhome.tar.gz  /home
>
> 将pc.tar.gz解压到当前目录：
>
> tar  -zxvf  pc.tar.gz
>
> 将myhome.tar.gz解压到/opt/tmp目录下
>
> tar  -zxvf  myhome.tar.gz  -C  /opt/tmp（-C表示choose，即选择要解压到的目录）

## 八、权限

### 1.   概述

linux中，文件或目录对不同的用户、不同的组，所给予的权限不同。默认创建文件的用户为所有者，所有者所在的组为文件所属组。

![image-20220707155816350](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707155816350.png)

上图中，以bbb目录为例，所有者为hoshino，所属组为mygroup。

> 第一列中，表示了该文件的权限。
>
> 0 - 9位说明：
>
> 1. 第0位确定文件类型：
>
>    ​	l 是链接，相当于windows快捷方式
>
>    ​	d 是目录
>
>    ​	c 是字符设备文件，如鼠标，键盘
>
>    ​	b 是块设备，如硬盘
>
> 2. 第1 - 3位确定所有者拥有该文件的权限
>
> 3. 第4 - 6位确定所属组拥有该文件的权限（在这个组内的用户拥有的权限）
>
> 4. 第7 - 9位确定其他用户拥有该文件的权限
>
> > 权限分为：
> >
> > r   ： 可读
> >
> > w  ：可写
> >
> > x   ：可执行

### 2.   rwx权限详解

* rwx作用到文件

​			r：可以读取，查看

​			w：可写，可以修改，但不代表可以删除，可删除的前提是对文件所在的目录有写权限，才能删除该文件

​			x：代表可执行

* rwx作用到目录

​			r：可以读取，ls查看目录内容

​			w：可以修改，对目录内创建+删除+重命名目录

​			x：可以进入该目录

### 3.   权限修改

u：所有者

g：所属组

o：其他人

a：所有人（u、g、o的总和）

#### （1）+、-、=变更权限

> chmod  u=rwx,g=rx,o=x  文件/目录名

#### （2）额外添加权限

> chmod  o+w  文件/目录名（给其他人添加执行权限）

#### （3）删除权限

> chmod  a-x  文件/目录名 （给所有人去除执行权限）

示例：给abc文件的所有者除去执行的权限，增加所属组写的权限：

> chmod  u-x,g+w  abc

#### （4）使用十进制数修改权限

r：4（100）

w：2（010）

x：1（001）

示例：将abc文件改为这样的权限，rwx-wxrw-

> chmod  736
>
> > 第一组：rwx相加为7
> >
> > 第二组：-wx相加为3
> >
> > 第三组：rw-相加为6

选项 -R  递归修改权限。

### 4.   修改文件/目录所有者与所属组 ——chown 

基本介绍：

> 改变所有者chown  newowner  文件/目录

> 改变所有者与所在组：chown  newowner:newgroup  文件/目录

-R选项： 如果是目录，则使其下所有子文件或目录递归生效

### 5.   修改文件/目录所属组 —— chgrp

基本介绍：

> chgrp  newgroup  文件/目录

-R选项： 如果是目录，则使其下所有子文件或目录递归生效

## 九、任务调度

### （1）crond任务调度

#### 1.    快速入门

设置个人任务调度：==crontab  -e==

终止任务调度：==crontab -r==

列出当前有哪些任务调度：==crontab  -l==

> 设置任务调度文件：/etc/crontab
>
> 接着输入任务到调度文件
>
> 如：==*/1 * * * *  ls  -l  /etc/ > /tmp/to.txt==
>
> 意为：每小时每分钟执行 ls  -l  /etc/ > /tmp/to.txt

#### 2.   参数细节说明

| 项目    | 含义                 | 范围                    |
| ------- | -------------------- | ----------------------- |
| 第一个* | 一小时当中的第几分钟 | 0-59                    |
| 第二个* | 一天当中的第几小时   | 0-23                    |
| 第三个* | 一个月当中的第几天   | 1-31                    |
| 第四个* | 一年当中的第几月     | 1-12                    |
| 第五个* | 一周当中的星期几     | 0-7（0与7都代表星期日） |

#### 3.   时间规则

| 特殊符号 | 含义                                                         |
| -------- | ------------------------------------------------------------ |
| *        | 代表任何时间，比如第一个*，就代表一小时中的每分钟都执行一次  |
| ，       | 代表不连续的时间，比如 ”0  8,12,16  *  *  *“命令，就代表在每天的8点0分，12点0分，16点0分都执行一次命令 |
| -        | 代表连续的时间，比如 ”0  5  *  *  1-6“，就代表在每周的星期一到星期六的早上5点0分都执行一次命令 |
| */n      | 代表每隔多久执行一次，比如 ”*/10  *  *  *  *“，代表每隔十分钟执行一次 |

#### 4.   定时执行脚本

> * 写脚本
> * 给执行权限
> * crontab  -e设定任务

如：需要把当前日期与日历每5分钟追加写入到/home/mycal中：

> 1. 写脚本：如在/home下写一个mycal.sh脚本：
>
>    ​	date >> /home/mycal
>
>    ​	cal >> /home/mycal
>
> 2. 赋予用户执行权限：chmod  u+x  mycal.sh
>
> 3. crontab  -e：
>
>    ​	*/5  *  *  *  *  /home/mycal.sh

### （2）at任务调度

#### 1.   基本介绍

* ==at命令是一次性定时计划任务，at的守护进程atd会以后台模式运行==
* 默认情况下，atd守护进程每60秒检查作业队列，有作业时，会检查作业运行时间，如果时间与当前时间匹配，则执行此作业。
* at命令是一次性定时计划任务，执行完一个任务后，不再执行此任务了。
* 在使用at命令的时候，一定要保证atd进程的启动，可以使用相关指令来查看。

​			（ps  -ef查看所有在运行的进程）

​		所以可以 ps  -ef | grep  atd来查看atd是否在运行

> 命令格式： at  [选项]  [时间]
>
> ​					Ctrl+D结束at命令的输入

#### 2.   at命令选项

| 选项           | 含义                                                       |
| -------------- | ---------------------------------------------------------- |
| -m             | 当前指定的任务被完成后，将给用户发送邮件，即使没有标准输出 |
| -I             | atq的别名                                                  |
| -d             | atrm的别名                                                 |
| -v             | 显示任务将被执行的时间                                     |
| -c             | 打印任务的内容到标准输出                                   |
| -V             | 显示版本信息                                               |
| -q（队列）     | 使用指定的队列                                             |
| -f（文件）     | 从指定文件读入任务而不是从标准输入读入                     |
| -t（时间参数） | 以时间参数的形式提交要运行的任务                           |

#### 3.   应用实例

1. 两天后下午5点执行/bin/ls  /home

> 先：at  5pm + 2days
>
> 然后输入要执行的指令：/bin/ls  /home
>
> 输入完成后，按两次Ctrl+D保存退出 

2. atq命令查看系统中没有执行的工作任务

> atq

3. 明天17点，输出时间到指定文件内，比如/root/date100.log

> 先：at  5pm  tomorrow
>
> 然后输入要执行的指令：date > /root/date100.log
>
> 输入完成后，按两次Ctrl+D保存退出 

4. 2分钟后，输出时间到指定文件内，比如/root/date200.log

> 先：at  now +  2minutes
>
> 然后输入要执行的指令：date > /root/date200.log
>
> 输入完成后，按两次Ctrl+D保存退出 

at执行的任务也可以是一个脚本

## 十、磁盘

#### （1）Linux磁盘分区

* linux无论有几个分区，分给哪一目录使用，它归根结底就只有一个根目录，一个独立且唯一的文件结构。linux中每个分区都是用来组成整个文件系统的一部分。
* linux采用了一种叫 ”载入“ 的处理方法，它的整个文件系统中包含了一整套的文件和目录，且将一个分区和一个目录联系起来。这时要载入的一个分区将使它的存储空间在一个目录下获得。
* 示意图：

![image-20220708153556310](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708153556310.png)

#### （2）   查看磁盘情况

> 使用lsblk指令，能查看具体情况

![image-20220708153842859](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708153842859.png)

可以看到，这里磁盘vda只有一个分区，vda1，挂载了根目录 /，所有目录都在这个磁盘下

#### （3）    查看磁盘占用情况

> df  -h

![image-20220709155911861](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220709155911861.png)

#### （4）   查看目录下的占用情况

> du  -hac  --max-depth=1   目录名（设置最大深度为1，只看第一层的占用情况）

![image-20220709160130288](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220709160130288.png)

## 十一、网络配置（虚拟机环境的局域网下，与公网环境不同）

### 1.   修改ip地址

#### （1）概论

在实际开发中，一般不采取DHCP（ **动态主机配置协议 ，自动分配ip**），而采取静态ip（要保证用户能通过同一个ip访问到，不能这一次这个ip行得通，下一次换ip就行不通）。

> ifconfig查看网络配置

#### （2）操作

对网卡文件：/etc/sysconfig/network-scripts/ifcfg-ens33进行编辑配置。

> * 将BOOTPROTO改为static
> * 添加IPADDR为要固定的ip地址
> * 添加网关GATEWAT为固定的网关地址
> * 添加域名解析器DNS1为与网关相同

此处虚拟机IPADDR网段应与主机上虚拟机的以太网适配器网段相同，如IPADDR为192.168.200.x，则适配器也应为192.168.200.x。

具体操作方法：

改完上述文件后，在虚拟机的虚拟网络编辑器中，将子网ip改为相应网段，上述例子中应改为192.168.200.0。

在NAT设置中，将网关地址改为相应地址。

### 2.   主机名和hosts映射

#### （1）修改主机名

编辑/etc/hostname

> vim  /etc/hostname
>
> 在其中输入想要的主机名即可

#### （2）主机名（域名）映射

与Windows的hosts文件相同用法，在linux下是/etc/hosts。

域名解析优先级：浏览器缓存 -> hosts文件 -> DNS解析

#### （3）修改dns配置

1. 临时修改：

​		打开/etc/resolv.conf文件，第一行修改主要DNS，第二行修改备用DNS

2. 永久修改：

​		打开/etc/resolvconf/resolv.conf.d/base，第一行修改主要DNS，第二行修改备用DNS

## 十二、进程

### 1.   基本指令

* ps命令是用来查看目前系统中，有哪些正在执行，以及它们执行的状态。可以不加任何参数。

> ps  -a：显示当前终端的所有进程信息
>
> ps  -u：以用户的格式显示进程信息
>
> ps  -x：显示后台进程运行的参数

一般三条指令可以联合使用：ps  -aux  | more

想查看特定的进程： ps  -aux  |  grep  xxx

但是一般查看进程信息我们会用：==ps  -ef==

### 2.   说明

> * USER：用户名称
> * PID：进程号
> * %CPU：进程占用CPU的百分比
> * %MEM：进程占用物理内存的百分比
> * VSZ：进程占用的虚拟内存大小（单位：KB）
> * RSS：进程占用的物理内存大小（单位：KB）
> * TT：终端名称，缩写
> * STAT：进程状态，==其中S表示睡眠==，s表示该进程是会话的先导进程，N表示进程拥有比普通优先级更低的优先级，==R表示正在运行==，D表示短期等待，==Z表示僵死进程==，T表示被跟踪或者被停止等。
> * STARTED：进程的启动时间
> * TIME：CPU时间，即进程使用CPU的总时间
> * COMMAND：启动进程所用的命令和参数，如果过长会被截断显示

### 3.   父子进程

一个进程可以创建多个其他进程，这个进程就叫其他进程的父进程，相对其他进程叫这个进程的子进程。

在linux中，可以使用==ps  -ef | grep  xxx==查看指定的进程，在CMD描述中，如果前缀为xxx：开头的，即为xxx的子进程。

### 4.   终止进程 kill 与 killall

> 基本语法：
>
> 1. kill  [选项]  进程号（通过进程号杀死进程）
> 2. killall  进程名称（通过进程名称杀死进程，也支持通配符，在系统负载过大时很有用）

> 常用选项：
>
> * -9：表示强迫进程立即停止

### 5.   实例

如果我们想踢掉某个非法登录用户，我们可以先杀掉远程登录服务sshd：

>  kill   sshd的进程号

恢复服务：/bin/systemctl   start   sshd.service



有些时候，在运行的进程通过普通的kill无法退出，因为系统认为该进程还在执行，你的kill可能是一个误操作，所以系统会置之不理，但如果你就是想强制关闭它，则：

> kill   -9   进程号

### 6.   进程树

基本语法：

> pstree：会将所有进程以树状表现出来

常用选项：

> -p：在进程后面显示进程号
>
> -u：显示用户

## 十二、服务管理

### 1.   开机流程说明

开机 -> BIOS -> /boot -> systemd进程1 -> 运行级别 -> 运行级对应的服务

### 2.   查看服务在各个运行级别下的自启动状态

chkconfig  --list（--list可省略），只能列出SysV服务（一些网络服务）：

![image-20220710142525040](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220710142525040.png)

chkconfig基本语法：

> 查看服务：chkconfig  --list | grep xxx
>
> ​					chkconfig   服务名   --list
>
> 修改某个运行级别（此处为5级别）下的服务：chkconfig  --level  5  服务名  on/off

chkconfig重新设置服务后需要重启reboot

### 3.   systemctl管理指令

基本语法：

> 1. systemctl  [start|stop|restart|status]  服务名（==临时==地打开、关闭....某个服务，当系统重启还是原来的）
> 2. systemctl指令管理的服务在==/usr/lib/systemd/system==查看，这里包含了所有服务
> 3. 查看服务开机启动状态，grep可以过滤：==systemctl  list-unit-files [|grep  服务名]==
> 4. 设置服务开机启动：systemctl  enable  服务名（==永久==打开某个服务）
> 5. 关闭服务开机启动：systemctl  disable  服务名（==永久==关闭某个服务）
> 6. 查询某个服务是否是自启动的：systemctl  is-enabled  服务名

查看特定服务：

> ls  -l  /usr/lib/systemd/system |  grep  xxx

### 4.   防火墙

在实际应用开发中，我们需要打开防火墙保证系统安全性，但如果把防火墙打开，外部请求数据包就不能跟服务器监听端口通讯。这时，需要打开指定的端口，比如80，22，3306，8080等。

firewall指令：

> 打开端口：firewall-cmd  --permanent  --add-port=端口号/协议
>
> 关闭端口：firewall-cmd  --permanent  --remove-port=端口号/协议
>
> 查询端口是否开放：firewall-cmd  --query-port=端口/协议

注意：

==修改防火墙配置后，需要重新载入才能生效：firewall-cmd  --reload==

另外，对端口的协议可以用：netstat  -anp | more查看

示例：要打开8080端口：

> 1. firewall-cmd  --permanent  -add-port=8080/tcp
> 2. firewall-cmd  --reload

## 十三、动态监控系统

### 1.   基本操作

此前的ps  -ef可以显示一个时间点上的系统进程情况，而无法动态显示更新。

而top指令，可以动态显示系统进程情况。q或ctrl+c都可以退出top。

选项：

> -d  秒数：指定更新频率。（top更新频率默认为3秒一更新）
>
> -i：使top不显示任何闲置或僵死进程
>
> -p：通过指定监控进程ID来仅仅监控某个进程的状态

### 2.   top交互操作

| 操作 | 功能                          |
| ---- | ----------------------------- |
| P    | 以CPU使用率排序，默认就是此项 |
| M    | 以内存使用率排序              |
| N    | 以PID排序                     |
| q    | 退出top                       |

## 十四、监控网络状态

### 1.   基本语法

> netstat  [选项]

### 2.   选项说明

* -an：按一定顺序排列输出
* -p：显示哪个进程在调用

一般用netstat  -anp | more就可以

检测网络连接ping就可以

## 十五、包管理工具yum与rpm

### 1.   rpm

#### （1）简介

​			rpm（redhat package manager），红帽子开发的包管理工具

#### （2）简单查询指令

​			查询已安装的rpm列表 ：

> rpm  -qa

​			查询指定软件是否安装：

> rpm  -q  xxx

​			查询软件详细信息：

> rpm  -qi  xxx

​			查询某个文件从属哪个包：

> rpm  -qf  /xxx/xxx

​			查询软件的安装路径

> rpm  -ql   xxxxxx

#### （3）rpm包名基本格式

一个rpm包名：firefox-60.2.2-1.el7.centos.x86_64

说明：

* 名称：firefox
* 版本号：60.2.2-1
* 适用操作系统：el7.centos.x86_64

这里表示适用64位操作系统，如果是32位，则为i686或i386。

noarch表示通用。

#### （4）rpm包的管理

##### 1.   卸载rpm包

基本语法：rpm  -e  rpm包名（-e代表erase）

例，删除firefox包：rpm  -e  firefox

如果一个包是其他包的依赖，删除这个包可能会导致其他包用不了，则系统在删除时会给出警告并忽视删除命令，也可以带上参数--nodeps强制删除：

例：rpm  -e  --nodeps  firefox

#### （5）安装rpm包

基本语法：rpm  -ivh  rpm包全路径名称

参数说明：

* i：install安装
* v：verbose提示
* h：hash进度条

### 2.   yum

#### （1）介绍

yum是一个Shell前端软件包管理器，基于rpm包管理，能够从指定的服务器自动下载rpm包并且安装，无需将rpm包搞到本地再安装，==可以自动处理依赖性关系，并且一次安装所有依赖的软件包==。

#### （2）基本使用

* 查询yum仓库中拥有的软件包，如果有多个版本会一起列出来

> yum list | grep  xxx

* 安装软件

> yum  install  xxx

它会根据系统自动选择合适的版本

## 十六、Java定制篇

### 1.   安装jdk

1. 建立目录存放安装包：mkdir  /opt/jdk
2. 上传压缩包到/opt/jdk下
3. 解压tar  -zxvf  jdk-xxxx.tar.gz
4. 建java目录，mkdir  /usr/local/java
5. mv  /opt/jdk/jdkxxx   /usr/local/java
6. 配置环境变量的配置文件：vim  /etc/profile
7. export  JAVA_HOME=/usr/local/java/jdkxxx
8. export  PATH=$JAVA_HOME/bin:\$PATH
9. 让新的环境变量生效：source  /etc/profile

### 2.   tomcat

安装步骤差不多。

使用：tomcat的bin目录下可通过startup.sh脚本启动，shutdown.sh脚本关闭，默认监听8080端口，所以我们需要把防火墙8080端口打开，==记得重启防火墙生效==。

### 3.  MySQL

1. 新建存放压缩包的目录：/opt/mysql
2. 运行 wget  http://dev.mysql.com/get/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar，直接从网络下载，或者手动下也行。
3. 运行tar  -xvf  mysql......tar
4. 运行rpm -qa | grep  maria，查询mariadb的相关安装包。

> ps：mariadb是centos的自带的类mysql数据库，会跟mysql冲突，要先删除

5. 执行rpm  -e  --nodeps  mariadb-libs卸载
6. 真正安装mysql：

> * rpm  -ivh  mysql-community-common-5.7.26-1.el7.x86_64.rpm
> * rpm  -ivh  mysql-community-libs-5.7.26-1.el7.x86_64.rpm
> * rpm  -ivh  mysql-community-client-5.7.26-1.el7.x86_64.rpm
> * rpm  -ivh  mysql-community-server-5.7.26-1.el7.x86_64.rpm

7. 运行systemctl  start  mysqld.service，启动mysql
8. 开始设置root用户密码：

mysql自动给root用户设置随机密码，执行grep  "password"  /var/log/mysqld.log，可以看到当前密码

9. 运行mysql  -u root  -p，使用root用户登录
10. 设置root密码，可设置密码复杂度等级，0最低：

> set  global  validate_password_policy=0

11. 修改密码：

> set  password  for  'root'@'localhost'=password('修改的密码');

12. 令密码设置生效：

> flush  privileges;

## 十七、Shell脚本编程

### 1.   shell概述

我们的命令无法直接给内核让其执行工作，比如mkdir  /xxx这条命令，内核是无法理解的，所以需要一个中间  “解释器”，由这个“解释器“进行翻译交给内核，再由内核控制硬件执行命令。

shell就是这样一个“解释器”，常用的shell有bash、sh、csh等。

### 2.   shell脚本

以bash为例：

1. 脚本以：#!/bin/bash开头
2. 脚本需要有可执行权限

脚本常用执行方式：

1. 输入脚本绝对路径或相对路径：先赋予脚本执行权限，再执行脚本
2. sh  脚本：不用赋予脚本x权限，直接执行即可。

脚本注释：

​	如果有这样一段脚本：

​	C=\`date\`

​	echo "C=$C"

​	则注释掉它：

​	:<<!

​	C=\`date\`

​	echo "C=$C"

​	!

### 3.   shell变量

linux shell中的变量分为：系统变量和用户自定义变量。

#### （1）系统变量

有：

> $HOME   \$PWD   \$SHELL   \$USER等等

显示当前shell中所有变量：==set==

#### （2）自定义变量

shell变量的定义：

> 基本语法：
>
> * 定义变量：变量=值
> * 撤销变量：unset  变量
> * 声明静态变量： readonly  变量，注意：不能unset

定义变量的规则：

* 变量名称跟各语言变量规则基本相同
* 等号两侧不能有空格
* 变量名一般习惯为大写，这是一个规范

将命令的返回值赋给变量：

* A=\`date\`，命令两侧加反引号，运行里面的命令，并把结果返回给A
* A=$(date)，等价于反引号

### 4.   设置环境变量

基本语法：

> 1. export  变量名=变量值（将shell变量输出为环境变量）
> 2. source  配置文件（让修改后的配置信息立即生效）
> 3. echo  $变量名（查询环境变量的值）

在/etc/profile里定义的变量就为环境变量/全局变量，可以在各个地方使用

### 5.   位置参数变量

如果有一个脚本执行：

./myshell   100   200

这里的100、200就是位置参数变量

> 基本语法：
>
> * \$n：n为数字，\$0代表命令本身，\$1-\$9代表第一到第九个参数，十以上的参数需要用大括号包含，如\${10}
> * $\*：这个变量代表命令行中的所有的参数，\$*把所有的参数看成一个整体
> * $@：这个变量也代表命令行中所有的参数，不过\$@把每个参数区分对待
> * $#：这个变量代表命令行中所有参数的个数

### 6.   预定义变量

​		就是shell设计者事先已经定义好的变量，可以直接在shell脚本中使用。

基本语法：

> \$\$：当前进程的进程号
>
> \$!：后台运行的最后一个进程的进程号
>
> \$?：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量值为非0（具体哪个数，由命令自己来决定），则证明上一个命令执行不正确。

### 7.   运算式与运算符

基本语法：

> 1. \$((运算式))  或  \$[运算式]  或  expr  m + n（expr的运算符两边必须要有空格）
> 2. \\*，/，%分别代表乘除取余，+、-就是加减（在一些linux系统中 * 代表乘）

![image-20220712144559146](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220712144559146.png)

### 8.   条件判断

#### （1）基本使用

基本语法：

> if后的判断条件：[  condition  ]，注意condition前后要有空格
>
> [  condition  ]   &&   doxxxxx：如果前面的条件成立，就做doxxxxx

[ asdsad ]  ：返回true

[ ]：返回false

比较运算符：

> 1. =：字符串比较
>
> 2. 整数比较：
>    * -lt：小于
>    * -le：小于等于
>    * -eq：等于
>    * -gt：大于
>    * -ge：大于等于
>    * -ne：不等于
> 3. 按文件权限进行判断：
>    * -r：有读的权限
>    * -w：有写的权限
>    * -x：有执行的权限
> 4. 按文件类型进行判断：
>    * -f：文件存在并且是一个常规的文件
>    * -e：文件存在
>    * -d：文件存在并是一个目录

![image-20220712150215704](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220712150215704.png)

### 9.   流程控制

#### （1）单分支与多分支结构

```bash
if  [ condition1 ]

then 

​		xxxxxxx

elif  [ condition2 ]

then

​		xxxxxxx

fi
```

![image-20220712150815440](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220712150815440.png)

#### （2） case语句

```bash
# 基本语法：
case 变量 in
"值1")
doxxxxx
;;
"值2")
doxxxx
;;
*)
do_other
esac
```

![image-20220712151508463](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220712151508463.png)

#### （3）for循环

基本语法1（需要循环的变量在一个集合中）：

> for  变量  in  值1  值2  值3......
>
> do
>
> 程序
>
> done

使用\$@，输出命令行参数（注意：使用\$*会将命令行参数作为一个整体输出）：

![image-20220713141745762](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713141745762.png)

基本语法2（需要循环的变量在一个范围）：

> 和c风格循环差不多：
>
> for   ((初始值; 循环控制条件; 变量变化))
>
> do
>
> 程序
>
> done

![image-20220713141720319](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713141720319.png)

#### （4）while循环

基本语法：

> while  [ 条件判断式 ]
>
> do
>
> 程序
>
> done
>
> 注意：==条件判断式左右要有空格==

![image-20220713142356577](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713142356577.png)

#### （5）read读取控制台输入

基本语法：

> read  （选项） （参数）
>
> 选项：
>
> * -p：指定读取值时的提示符
> * -t：指定读取值时等待的时间（秒），如果没有在指定的时间内输入，就不再等待
>
> 参数：
>
> * 变量：指定读取值得变量名

![image-20220713143325088](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713143325088.png)

#### （6）函数

##### ~1.   系统函数

系统函数有很多，这里介绍两个：

1. basename

功能：返回完整路径最后 / 的部分，常用于获取文件名

basename  [pathname]  [suffix]

![image-20220713143656945](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713143656945.png)

在后面加上后缀，输出删除后缀后的文件名：

![image-20220713143734492](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713143734492.png)

2. dirname

功能：与basename相反，dirname输出路径名

如上述案例用dirname则输出：/home

##### ~2.   自定义函数

定义函数基本语法与js相同

调用函数：函数名   参数1   参数2......

![image-20220713144634122](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713144634122.png)

#### （7）综合实例——备份数据库

前置：需要使用mysqldump备份数据库，其会生成一个.sql文件（当然你也可以用.txt保存，反正是一些sql语句，用于恢复为当前时间的数据库）。

要求：

1. 每天凌晨两点半，自动备份一份数据库
2. 备份后的文件要求以备份时间为文件名，并打包成.tar.gz的形式，如：2022-7-13_160413.tar.gz
3. 在备份的同时，检查备份目录下是否有十天前备份的数据库文件，如果有就将其删除

编写：

1. shell脚本：

![image-20220713160610296](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713160610296.png)

2. 使用crond设定定时任务：

crontab  -e，编写定时任务：

![image-20220713160855473](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713160855473.png)

crontab   -l，查看任务列表：

![image-20220713160926478](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220713160926478.png)

### 十九、Python定制篇

#### 1.   apt软件管理工具

##### （1）概述

​		apt可以从固定服务器拿到软件源，然后通过网络down到本地，默认服务器在国外，可以通过修改/etc/apt/sources.list文件更换镜像源。

![image-20220714150727974](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220714150727974.png)

​		在更改之前先备份：

> sudo  cp  /etc/apt/sources.list   /etc/apt/sources.list.bak

​		清空原有文件内容：

> sudo  echo  ""  >  /etc/apt/sources.list

​		将内容贴到文件内：

```txt
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

##### （2）ssh

ubuntu不同于centos，默认没有安装ssh服务，需要自己安装

> sudo  apt-get  install  openssh-server

执行完上述命令后，这台linux上就安装了SSH==服务端与客户端==

启动sshd服务：

> service  sshd  start

有时候，服务端是一个linux集群，需要用一台linux登录另一台linux，就可以用ssh客户端登录，只要两台机子都安装了openssh-server就能互相登录，比如我要登录192.168.200.114下的hoshino用户：

> ssh   hoshino@192.168.200.114

> 退出登录：logout

### 二十、日志

#### （1）概述

日志系统是linux下非常重要的系统，记录了每天发生的事，对系统安全的意义十分重大。

日志文件保存在：`/var/log`下。

#### （2）日志作用

| 日志文件              | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| ==/var/log/boot.log== | 系统启动日志                                                 |
| ==/var/log/cron==     | 记录与系统定时任务相关的日志                                 |
| /var/log/cups/        | 记录打印信息的日志                                           |
| /var/log/dmesg        | 记录了系统在开机时内核自检的信息。也可以使用dmesg命令直接查看内核自检信息 |
| /var/log/btmp         | 记录错误登录的日志。这个文件是二进制文件，不能直接用vi查看，而要使用lastb命令查看。命令如下：`lastb` |
| ==/var/log/lasllog==  | 记录系统中所有用户最后一次的登录时间的日志。这个文件也是二进制文件，要用lastlog命令查看 |
| ==/var/log/mailog==   | 记录邮件信息的日志                                           |
| ==/var/log/message==  | 记录系统重要消息的日志，这个日志文件中会记录linux系统的绝大多数重要信息。如果系统出现问题，首先要检查的应该就是这个日志文件 |
| ==/var/log/secure==   | 记录验证和授权方面的信息，只要涉及账户和密码的程序都会记录，比如系统的登录、ssh的登录、su切换用户、sudo授权，甚至添加用户和修改用户密码都会记录在这个日志文件中 |
| /var/log/wtmp         | 永久记录所有用户的登录、注销信息，同时记录系统的启动、重启、关机事件。是二进制文件，要使用last命令查看 |
| ==/var/tun/ulmp==     | 记录当前已经登录的用户的信息。这个文件会随着用户的登录和注销而不断变化，只记录当前登录用户的信息。这个文件不能用vi查看，而要用w、who、users等命令查看 |

#### （3）日志管理

在centos7.6，使用rsyslogd服务进行日志管理，该服务对日志的管理规则在==/etc/rsyslog.conf==下，是一个后台服务。

![image-20220714155935640](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220714155935640.png)

可以看出，规则对什么样的操作记录在哪个日志做了详细的规定。

#### （4）自定义日志规则

根据上述规则的格式，我们也可以自定义规则，在/etc/rsyslog.conf内可写入自己的规则，比如：

> \*.\*                          /var/log/hoshino.log

表示将所有服务的所有级别的日志写入/var/log/hoshino.log，系统写日志时会默认创建该文件，我们无需创建。

#### （5）日志轮替

当我们希望对日志文件进行一定规则的删除轮替时，可以在：

/etc/logrotate.conf（全局的日志轮替规则，当然可以单独给某个日志文件指定策略）内制定轮替规则。

##### 1.   基本介绍

​	日志轮替就是把旧的日志文件移动并改名，同时建立新的空日志文件，当旧日志文件超出保存的范围就会进行删除。

##### 2.   日志轮替文件命名

* centos7使用logrotate进行日志轮替管理，想要改变日志轮替文件名字，通过/etc/logrotate.conf配置文件中 “dateext”参数。
* 如果配置文件中有 “dateext”参数，那么日志会用日期来作为日志文件的后缀，例如 “secure-20220716”.这样日志文件名不会重叠，也就不需要日志文件的改名，只需要指定保存日志个数，删除多余的日志文件（最旧的）即可。
* 如果配置文件中没有 “dateext”参数，日志文件就需要进行改名了。当第一次进行日志轮替时，当前的 “secure”日志会自动改名为 “secure.1“，然后新建 “secure”日志，用来保存新的日志。当第二次进行日志轮替时，”secure.1“会改名为 ”secure.2“，而新的日志为 ”secure.1“， 依次类推。

/etc/logrotate.conf默认配置：

```bash
# see "man logrotate" for details
# rotate log files weekly
# 一星期轮替一次
weekly

# keep 4 weeks worth of backlogs
# 总共保存4份日志文件，多余的会被删除
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
# 按日期轮替
dateext

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
# 单独的日志轮替规则也可以写到/etc/logrotate.d（注意，这是一个目录）目录的文件中
include /etc/logrotate.d

# no packages own wtmp and btmp -- we'll rotate them here
# 单独的日志轮替规则，上面的为全局的
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}

/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}
```

##### 3.   自定义日志轮替规则

logrotate配置文件

参数说明：

1. daily：日志轮替周期是每天
2. weekly：日志轮替周期是每周
3. monthly：日志轮替周期是每月
4. rotate   数字：保留的日志文件的个数。0指没有备份
5. compress：日志轮替时，旧的日志进行压缩
6. create mode owner group：建立新日新日志时，同时指定新日志的权限与所有者和所属组。
7. mail address：当日志轮替时，输出内容通过邮件发生到指定的邮件地址。
8. missingok：如果日志不存在，则忽略该日志的警告信息
9. notifempty：如果日志为空文件，则不进行日志轮替
10. minsize  大小：日志轮替的最小值，当日志大小小过这个值时，即使时间到了也不进行轮替。
11. size  大小：日志只有大于指定大小才进行日志轮替，而不是按照时间轮替。
12. dateext：使用日期作为日志轮替文件的后缀
13. sharedscripts：在此关键字之后的脚本只执行一次
14. prerotate / endscript：在日志轮替之前执行脚本命令
15. postrotate / endscript：在日志轮替之后执行脚本命令。

例如，我们可以在/etc/logrotate.d下新建一个轮替规则文件，用于规定/var/log/下的hoshino.log文件：

> cd  /etc/logrotate.d
>
> vim  hoshino：
>
> /var/log/hoshino.log
>
> {
>
> ​		missingok
>
> ​		daily
>
> ​		rotate  7
>
> ​		notifempty
>
> }

##### 4.   说明

日志轮替之所以可以在指定的时间备份日志，是依赖系统定时任务。在/etc/cron.daily/目录，会发现目录下有个logrotate文件（可执行），logrotate通过这个文件依赖定时任务执行的。

#### （6）内存日志

系统重启后会消失，保存系统运行的一些只在内存中的日志称为内存日志。

基本命令：journalctl

参数：

* -n：查看最新3条
* --since  19:00  --until  19:10:10：查看从起始时间到结束时间的日志，可加日期
* -p err：报错日志
* _PID=XXXX  _COMM=sshd：查看包含这些参数的日志
* -o  verbose：日志详细内容

### 二十一、备份与恢复

#### 1.  备份

##### （1）   基本介绍

​	dump支持分卷和增量备份（增量备份就是备份上次备份后修改/增加过的文件，对没有变化的文件不进行再次备份）。

##### （2）  dump语法说明

dump  [-cu] [-123456789] [-f <备份后文件名>] [-T <日期>] [目录或文件系统]

* -c：创建新的归档文件，并将由一个或多个文件参数所指定的内容写入归档文件的开头
* -0123456789：备份的层级。0为最完整备份，会备份所有文件。若指定0以上的层级，则备份至上一次备份以来修改或新增的文件，到9后，可以再次轮替（最大层级为9）
* -f  <备份后文件名>：指定备份后文件名
* -j：调用==bzlib库压缩备份文件==，将备份后文件压缩成bz2格式，让文件更小
* -T <日期>：指定开始备份的时间与日期
* -u：备份完毕后，在/etc/dumpdares中记录备份的文件系统，层级，日期与时间等。
* -t：指定文件名，若该文件已存在备份文件中，则列出名称
* -W：显示需要备份的文件及其最后一次备份的层级，时间，日期。
* -w：与-W类似，但仅显示需要备份的文件。

##### （3）  使用

如果我们要将boot==磁盘分区==，备份为/opt/boot.bak0.bz2，可以：

```shell
dump -0uj -f /opt/boot.bak0.bz2 /boot/
```

注意：如果是备份==文件或目录则不支持增量备份==，也就是备份层级只能为0。

#### 2.  恢复

##### （1）基本介绍

​	restore命令用来恢复已备份的文件，可以从dump生成的备份文件中恢复原文件

##### （2）基本语法

​	restore  [模式选项]  [选项]

​	以下四个模式，不能混用，一次命令中只能指定一种：

1. -C：使用对比模式，将备份的文件与已存在的文件相互对比
2. -i：使用交互模式，在进行还原操作时，restore指令将依序询问用户
3. -r：进行还原模式
4. -t：查看模式，看备份文件有哪些文件

​	选项：

* -f <备份设备>：从指定的文件中读取备份数据，进行还原操作

##### （3）示例

如果要恢复上述“备份”中备份的文件：

```shell
restore -r -f /opt/boot.bak0.bz2
```

### 二十二、面试题

#### 1.  统计访问量与连接数

##### （1）统计访问量

​		如果有一份文件，里面记载了服务器资源的每次访问的url，如下：

```
http://192.168.200.1/index.html
http://192.168.200.1/index.html
http://192.168.200.1/index.html
http://192.168.190.3/index.html
http://192.168.280.3/index.html
http://192.168.140.2/login.html
http://192.168.140.2/login.html
http://192.168.130.3/login.html
```

​		如何统计访问量ip呢？

```shell
# 如果文件名为ask，先输出ask内容
cat ask
# 我们需要的是ip，想办法取出ip，使用cut以“/”为分隔符，然后取出分割后的第三个元素即为ip
cat ask | cut -d "/" -f 3 
# 获得ip后，使用sort排序，将相同的ip排到一起（非必要）
cat ask | cut -d "/" -f 3 | sort
# 将ip归一化处理并计数
cat ask | cut -d "/" -f 3 | sort | uniq -c
# 将处理结果降序排序，（-nr:-n是将字符串按ascii数的大小排序，与一般编程语言的排序方式相同，-r是reverse，降序排序）
cat ask | cut -d "/" -f 3 | sort | uniq -c | sort -nr
```

拓展：如果只需要访问量前三位的ip，只需加一条head -n的指令就好（n代表取到第几位）：

```shell
cat ask | cut -d "/" -f 3 | sort | uniq -c | sort -nr | head -3
```

##### （2）统计连接数

​		当外部访问服务器时，服务器网络服务可以监听到外部访问，如何统计外部访问的ip并选出访问最多的ip呢？

![image-20220724223245098](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220724223245098.png)

```shell
# 首先使用netstat -an查看网络连接状况
netstat -an
# 选取访问的连接
netstat -an | grep ESTABLISHED
# 可以看到，第五项就是我们想要的外部ip，那么如何获得呢？因为每一项是由一长串空格分割的，用cut不太行得通，我们可以用awk指令，这里-F指定分隔符，awk的分隔符不是指定一个，而是一段相同的，在其后加'{print $5}'即可打印第五项了
netstat -an | grep ESTABLISHED | awk -F " " '{print $5}'
# 我们要的是ip，再以“:”为分隔符，将ip与端口号分离，取第1项就是ip
netstat -an | grep ESTABLISHED | awk -F " " '{print $5}' | cut -d ":" -f 1
# 接下来进行排序（非必要）并归一计数
netstat -an | grep ESTABLISHED | awk -F " " '{print $5}' | cut -d ":" -f 1 | sort | uniq -c
# 降序排序
netstat -an | grep ESTABLISHED | awk -F " " '{print $5}' | cut -d ":" -f 1 | sort | uniq -c | sort -nr
```

##### （3）tcpdump监听本机

​		使用tcpdump监听本机，将来自ip 192.168.200.1，tcp端口为22的数据，保存输出到/home/tcpdump.log下。

​		说明：

**tcpdump** 是Linux系统下的一个强大的命令，可以将网络中传送的数据包完全截获下来提供分析。它支持==针对网络层、协议、主机、网络或端口的过滤==，并提供and、or、not等逻辑语句来帮助你去掉无用的信息。

常用选项：

1. -i  网络接口：只查看指定网络接口的数据包（可以用ifconfig查看网络接口）
2. -w  文件：直接将输出存到文件中而不是打印出来
3. host  [ip]：抓取的是指定ip下的包
4. port  [port]：抓取指定端口下的包

```shell
# 首先抓取特定条件下的包（我的服务器上外部网卡为eth0）
tcpdump -i eth0
# 主机为192.168.200.1
tcpdump -i eth0 host 192.168.200.1
# 端口为22
tcpdump -i eth0 host 192.168.200.1 and port 22
# 写入/home/tcpdump.log内（可以用-ｗ，也可以直接>>）
tcpdump -i eth0 host 192.168.200.1 and port 22 >> /home/tcpdump.log
```

#### （4）权限控制

​	如果你是系统管理员，你会如何划分系统的权限，另外，如何控制使得任何人都不能使用useradd添加用户。

划分权限：

1. 可以从实际工作经验去谈，阐述一下文件与目录的rwx权限
2. 最小权限划分原则：在能够使用的情况下，尽量使用户获取最小的权限。

使得任何人都不能使用useradd添加用户：

* chattr：change attribute，修改文件属性，就是给文件加个锁，简单语法：

```shell
# 给文件加锁
chattr +i [文件路径]
# 给文件解锁
chattr -i [文件路径]
```

1. useradd命令在/etc/passwd中，所以应该对这个文件加锁。
2. 另外，为了安全性，可以将chattr命令移走，需要时再移回来
3. 但是移到其他地方也能通过全盘扫描找到，所以可以对chattr改个名（补充：chattr命令在/usr/bin下）

```shell
# 加锁
chattr +i /etc/passwd
# 移动chattr命令，这里移到/opt下
mv /usr/bin/chattr /opt/
# 改名
mv /opt/chattr /opt/h
```

#### （5）写出6个linux高级指令

* netstat：网络状态监控
* top：系统运行状态
* lsblk：查看硬盘分区
* find：查找文件
* ps -aux：查看运行进程
* chkconfig：查看服务启动状态（运行级别）
* systemctl：管理系统服务

#### （6）linux查看内存、io读写、磁盘存储、端口占用、进程查看的命令是什么？

1. 查看内存：可以直接使用top。
2. io读写监控：使用iotop，初始没有这个命令，需要yum安装
3. 磁盘存储情况：df命令，可加参数（-l，local，本地）,（-h，human-readable，人可读的，实际上就是将内存空间表示转成人容易理解的k，M，G）
4. 端口占用情况：netstat -tunlp
5. 进程查看：ps -aux、ps -ef

#### （7）求分数和

如果有一份文件：

```
张三 50
李四 60
王五 70
```

求三人分数总和并打印：

```shell
# 如果文件名为/home/grade.txt，先打印输出
cat /home/grade.txt
# 分割取值
cat /home/grade.txt | awk -F " "
# 求和
cat /home/grade.txt | awk -F " " '{sum+=$2}'
# 打印
cat /home/grade.txt | awk -F " " '{sum+=$2} END {print sum}'
```

#### （8）shell脚本检查文件是否存在

```shell
if [ -f 文件名 ]
then 
	echo "存在"
else 
	echo "不存在"
fi
```

#### （9）将文本的无序数字有序排列并输出，且输出它们的和

```shell
# 若有文本num.txt
9
7
4
11
8
16
5
# 排序输出
sort -nr num.txt | awk '{sum+=$1;print $1} END {print "和="sum}'
```

#### （10）用指令写出查找当前文件夹（/home）下所有的文件内容包含字符“cat”的文件名称

```shell
# 用grep找出包含字符串的文件 -r表示递归
grep -r "cat" /home
# 取出文件名
grep -r "cat" /home | cut -d ":" -f 1
```

#### （11）统计某个目录的文件个数（包含文件夹）

```shell
# find指令通过name找出所有文件
find /home -name "*.*"
# 统计个数
find /home -name "*.*" | wc -l

#拓展：列出一个目录下的所有文件，并统计每个文件的行数
find /home -name "*.*" | xargs wc -l
```

