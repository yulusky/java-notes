# Table of Contents

* [简单操作以及概念](#简单操作以及概念)
  * [快捷键](#快捷键)
  * [求助](#求助)
  * [关机](#关机)
  * [PATH](#path)
  * [sudo](#sudo)
* [文件系统](#文件系统)
  * [Linux文件系统简介](#linux文件系统简介)
  * [文件类型与目录结构](#文件类型与目录结构)
  * [inode](#inode)
  * [文件与目录的基本操作](#文件与目录的基本操作)
  * [修改权限](#修改权限)
  * [默认权限](#默认权限)
  * [目录的权限](#目录的权限)
  * [链接](#链接)
    * [1. 实体链接](#1-实体链接)
    * [2. 符号链接](#2-符号链接)
  * [获取文件内容](#获取文件内容)
  * [指令与文件搜索](#指令与文件搜索)
* [进程管理](#进程管理)
  * [查看进程](#查看进程)
  * [进程状态](#进程状态)
  * [SIGCHLD](#sigchld)
  * [wait()](#wait)
  * [waitpid()](#waitpid)
  * [孤儿进程](#孤儿进程)
  * [僵尸进程](#僵尸进程)
* [常用重要命令](#常用重要命令)
  * [查看内存、磁盘的命令](#查看内存、磁盘的命令)
  * [vim](#vim)
    * [VIM 三个模式](#vim-三个模式)
    * [VIM查找和替换命令](#vim查找和替换命令)
  * [文本文件找特定字符串](#文本文件找特定字符串)
* [select、poll、epoll](#select、poll、epoll)
  * [select，poll](#select，poll)
  * [epoll](#epoll)
* [压缩与打包](#压缩与打包)
  * [压缩文件名](#压缩文件名)
  * [压缩指令](#压缩指令)
  * [打包](#打包)
* [参考资料](#参考资料)



# 简单操作以及概念

## 快捷键

- Tab：命令和文件名补全；
- Ctrl+C：中断正在运行的程序；
- Ctrl+D：结束键盘输入（End Of File，EOF）

## 求助

1. --help

指令的基本用法与选项介绍。

2. man

man 是 manual 的缩写，将指令的具体信息显示出来。

当执行 `man date` 时，有 DATE(1) 出现，其中的数字代表指令的类型，常用的数字及其类型如下：

| 代号 | 类型                                            |
| :--: | ----------------------------------------------- |
|  1   | 用户在 shell 环境中可以操作的指令或者可执行文件 |
|  5   | 配置文件                                        |
|  8   | 系统管理员可以使用的管理指令                    |

3. info

info 与 man 类似，但是 info 将文档分成一个个页面，每个页面可以进行跳转。

4. doc

/usr/share/doc 存放着软件的一整套说明文件。

## 关机

1. who

在关机前需要先使用 who 命令查看有没有其它用户在线。

2. sync

为了加快对磁盘文件的读写速度，位于内存中的文件数据不会立即同步到磁盘上，因此关机之前需要先进行 sync 同步操作。

3. shutdown

```html
shutdown [-krhc] 时间 [信息]
-k ： 不会关机，只是发送警告信息，通知所有在线的用户
-r ： 将系统的服务停掉后就重新启动
-h ： 将系统的服务停掉后就立即关机
-c ： 取消已经在进行的 shutdown 指令内容
```

## PATH

可以在环境变量 PATH 中声明可执行文件的路径，路径之间用 : 分隔。

```html
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/dmtsai/.local/bin:/home/dmtsai/bin
```

## sudo

sudo 允许一般用户使用 root 可执行的命令，不过只有在 /etc/sudoers 配置文件中添加的用户才能使用该指令。

# 文件系统

## Linux文件系统简介

**在Linux操作系统中，所有被操作系统管理的资源，例如网络接口卡、磁盘驱动器、打印机、输入输出设备、普通文件或是目录都被看作是一个文件。**

也就是说在LINUX系统中有一个重要的概念：**一切都是文件**。其实这是UNIX哲学的一个体现，而Linux是重写UNIX而来，所以这个概念也就传承了下来。在UNIX系统中，把一切资源都看作是文件，包括硬件设备。UNIX系统把每个硬件都看成是一个文件，通常称为设备文件，这样用户就可以用读写文件的方式实现对硬件的访问。

## 文件类型与目录结构

**Linux支持5种文件类型 ：** [![文件类型](https://camo.githubusercontent.com/3d2c05419cb1f93fae15869cfc541c2bb3d5674b/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f372f332f313634356631613764363464656631613f773d39303126683d35343726663d706e6726733d3732363932)](https://camo.githubusercontent.com/3d2c05419cb1f93fae15869cfc541c2bb3d5674b/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f372f332f313634356631613764363464656631613f773d39303126683d35343726663d706e6726733d3732363932)

**Linux的目录结构如下：**

Linux文件系统的结构层次鲜明，就像一棵倒立的树，最顶层是其根目录： [![Linux的目录结构](https://camo.githubusercontent.com/5f3d9061e41baf498b2b37c4d80fad87fcf422ff/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f372f332f313634356631633635363736636166363f773d38323326683d33313526663d706e6726733d3135323236)](https://camo.githubusercontent.com/5f3d9061e41baf498b2b37c4d80fad87fcf422ff/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f372f332f313634356631633635363736636166363f773d38323326683d33313526663d706e6726733d3135323236)

**常见目录说明：**

- **/bin：** 存放二进制可执行文件(ls、cat、mkdir等)，常用命令一般都在这里；
- **/etc：** 存放系统管理和配置文件；
- **/home：** 存放所有用户文件的根目录，是用户主目录的基点，比如用户user的主目录就是/home/user，可以用~user表示；
- **/usr ：** 用于存放系统应用程序；
- **/opt：** 额外安装的可选应用程序包所放置的位置。一般情况下，我们可以把tomcat等都安装到这里；
- **/proc：** 虚拟文件系统目录，是系统内存的映射。可直接访问这个目录来获取系统信息；
- **/root：** 超级用户（系统管理员）的主目录（特权阶级^o^）；
- **/sbin:** 存放二进制可执行文件，只有root才能访问。这里存放的是系统管理员使用的系统级别的管理命令和程序。如ifconfig等；
- **/dev：** 用于存放设备文件；
- **/mnt：** 系统管理员安装临时文件系统的安装点，系统提供这个目录是让用户临时挂载其他的文件系统；
- **/boot：** 存放用于系统引导时使用的各种文件；
- **/lib ：** 存放着和系统运行相关的库文件 ；
- **/tmp：** 用于存放各种临时文件，是公用的临时文件存储点；
- **/var：** 用于存放运行时需要改变数据的文件，也是某些大文件的溢出区，比方说各种服务的日志文件（系统启动日志等。）等；
- **/lost+found：** 这个目录平时是空的，系统非正常关机而留下“无家可归”的文件（windows下叫什么.chk）就在这里。

## inode

一个文件占用一个 inode，记录文件的属性

iNode就是用来存储这些数据的信息，这些信息包括文件大小、属主、归属的用户组、读写权限等。

## 文件与目录的基本操作

1. ls

列出文件或者目录的信息，目录的信息就是其中包含的文件。

```html
ls [-aAdfFhilnrRSt] file|dir
-a ：列出全部的文件
-d ：仅列出目录本身
-l ：以长数据串行列出，包含文件的属性与权限等等数据
```

2. cd

更换当前目录。

```
cd [相对路径或绝对路径]
```

3. mkdir

创建目录。

```
mkdir [-mp] 目录名称
-m ：配置目录权限
-p ：递归创建目录
```

4. rmdir

删除目录，目录必须为空。

```html
rmdir [-p] 目录名称
-p ：递归删除目录
```

5. touch

更新文件时间或者建立新文件。

```html
touch [-acdmt] filename
-a ： 更新 atime
-c ： 更新 ctime，若该文件不存在则不建立新文件
-m ： 更新 mtime
-d ： 后面可以接更新日期而不使用当前日期，也可以使用 --date="日期或时间"
-t ： 后面可以接更新时间而不使用当前时间，格式为[YYYYMMDDhhmm]
```

6. cp

复制文件。如果源文件有两个以上，则目的文件一定要是目录才行。

```html
cp [-adfilprsu] source destination
-a ：相当于 -dr --preserve=all
-d ：若来源文件为链接文件，则复制链接文件属性而非文件本身
-i ：若目标文件已经存在时，在覆盖前会先询问
-p ：连同文件的属性一起复制过去
-r ：递归复制
-u ：destination 比 source 旧才更新 destination，或 destination 不存在的情况下才复制
--preserve=all ：除了 -p 的权限相关参数外，还加入 SELinux 的属性, links, xattr 等也复制了
```

7. rm

删除文件。

```html
rm [-fir] 文件或目录
-r ：递归删除

```

8. mv

移动文件。

```html
mv [-fiu] source destination
mv [options] source1 source2 source3 .... directory
-f ： force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖

```

## 修改权限

可以将一组权限用数字来表示，此时一组权限的 3 个位当做二进制数字的位，从左到右每个位的权值为 4、2、1，即每个权限对应的数字权值为 r : 4、w : 2、x : 1。

```html
chmod [-R] xyz dirname/filename

```

示例：将 .bashrc 文件的权限修改为 -rwxr-xr--。

```html
chmod 754 .bashrc

```

也可以使用符号来设定权限。

```html
chmod [ugoa]  [+-=] [rwx] dirname/filename
- u：拥有者
- g：所属群组
- o：其他人
- a：所有人
- +：添加权限
- -：移除权限
- =：设定权限

```

示例：为 .bashrc 文件的所有用户添加写权限。

```html
chmod a+w .bashrc

```

## 默认权限

- 文件默认权限：文件默认没有可执行权限，因此为 666，也就是 -rw-rw-rw- 。
- 目录默认权限：目录必须要能够进入，也就是必须拥有可执行权限，因此为 777 ，也就是 drwxrwxrwx。

可以通过 umask 设置或者查看默认权限，通常以掩码的形式来表示，例如 002 表示其它用户的权限去除了一个 2 的权限，也就是写权限，因此建立新文件时默认的权限为 -rw-rw-r--。

## 目录的权限

文件名不是存储在一个文件的内容中，而是存储在一个文件所在的目录中。因此，拥有文件的 w 权限并不能对文件名进行修改。

目录存储文件列表，一个目录的权限也就是对其文件列表的权限。因此，目录的 r 权限表示可以读取文件列表；w 权限表示可以修改文件列表，具体来说，就是添加删除文件，对文件名进行修改；x 权限可以让该目录成为工作目录，x 权限是 r 和 w 权限的基础，如果不能使一个目录成为工作目录，也就没办法读取文件列表以及对文件列表进行修改了。

## 链接

<div align="center"> <img src="pics/1e46fd03-0cda-4d60-9b1c-0c256edaf6b2.png" width="450px"> </div><br>

```html
ln [-sf] source_filename dist_filename
-s ：默认是实体链接，加 -s 为符号链接
-f ：如果目标文件存在时，先删除目标文件

```

### 1. 实体链接

在目录下创建一个条目，记录着文件名与 inode 编号，这个 inode 就是源文件的 inode。

iNode相同的文件是硬链接文件

删除任意一个条目，文件还是存在，只要引用数量不为 0。

有以下限制：不能跨越文件系统、不能对目录进行链接。	

```html
ln /etc/crontab .
ll -i /etc/crontab crontab
34474855 -rw-r--r--. 2 root root 451 Jun 10 2014 crontab
34474855 -rw-r--r--. 2 root root 451 Jun 10 2014 /etc/crontab

```

### 2. 符号链接

符号链接文件保存着源文件所在的绝对路径，在读取时会定位到源文件上，可以理解为 Windows 的快捷方式。

软链接文件只是占用了iNode来存储软链接文件属性等信息，但文件存储是指向源文件的。

当源文件被删除了，链接文件就打不开了。

因为记录的是路径，所以可以为目录建立符号链接。

```html
ll -i /etc/crontab /root/crontab2
34474855 -rw-r--r--. 2 root root 451 Jun 10 2014 /etc/crontab
53745909 lrwxrwxrwx. 1 root root 12 Jun 23 22:31 /root/crontab2 -> /etc/crontab

```

## 获取文件内容

1. cat

取得文件内容。

```html
cat [-AbEnTv] filename
-n ：打印出行号，连同空白行也会有行号，-b 不会

```

2. tac

是 cat 的反向操作，从最后一行开始打印。

3. more

和 cat 不同的是它可以一页一页查看文件内容，比较适合大文件的查看。

4. less

和 more 类似，但是多了一个向前翻页的功能。

5. head

取得文件前几行。

```html
head [-n number] filename
-n ：后面接数字，代表显示几行的意思

```

6. tail

是 head 的反向操作，只是取得是后几行。

7. od

以字符或者十六进制的形式显示二进制文件。

## 指令与文件搜索

1. which

指令搜索。

```html
 which [-a] command
-a ：将所有指令列出，而不是只列第一个

```

2. whereis

文件搜索。速度比较快，因为它只搜索几个特定的目录。

```html
whereis [-bmsu] dirname/filename

```

3. locate

文件搜索。可以用关键字或者正则表达式进行搜索。

locate 使用 /var/lib/mlocate/ 这个数据库来进行搜索，它存储在内存中，并且每天更新一次，所以无法用 locate 搜索新建的文件。可以使用 updatedb 来立即更新数据库。

```html
locate [-ir] keyword
-r：正则表达式

```

4. find

文件搜索。可以使用文件的属性和权限进行搜索。

```html
find [basedir] [option]
example: find . -name "shadow*"

```

**① 与时间有关的选项** 

```html
-mtime  n ：列出在 n 天前的那一天修改过内容的文件
-mtime +n ：列出在 n 天之前 (不含 n 天本身) 修改过内容的文件
-mtime -n ：列出在 n 天之内 (含 n 天本身) 修改过内容的文件
-newer file ： 列出比 file 更新的文件

```

+4、4 和 -4 的指示的时间范围如下：

<div align="center"> <img src="pics/658fc5e7-79c0-4247-9445-d69bf194c539.png" width=""/> </div><br>

**② 与文件拥有者和所属群组有关的选项** 

```html
-uid n
-gid n
-user name
-group name
-nouser ：搜索拥有者不存在 /etc/passwd 的文件
-nogroup：搜索所属群组不存在于 /etc/group 的文件

```

**③ 与文件权限和名称有关的选项** 

```html
-name filename
-size [+-]SIZE：搜寻比 SIZE 还要大 (+) 或小 (-) 的文件。这个 SIZE 的规格有：c: 代表 byte，k: 代表 1024bytes。所以，要找比 50KB 还要大的文件，就是 -size +50k
-type TYPE
-perm mode  ：搜索权限等于 mode 的文件
-perm -mode ：搜索权限包含 mode 的文件
-perm /mode ：搜索权限包含任一 mode 的文件

```

# 进程管理

## 查看进程

1. ps

查看某个时间点的进程信息。

示例一：查看自己的进程

```sh
ps -l

```

示例二：查看系统所有进程

```sh
ps aux

```

示例三：查看特定的进程

```sh
ps aux | grep threadx

```

2. pstree

查看进程树。

示例：查看所有进程树

```sh
pstree -A

```

3. top

实时显示进程信息。

示例：两秒钟刷新一次

```sh
top -d 2

```

4. netstat

查看占用端口的进程

示例：查看特定端口的进程

```sh
netstat -anp | grep port

```

## 进程状态

| 状态 | 说明                                                         |
| :--: | ------------------------------------------------------------ |
|  R   | running or runnable (on run queue)<br>正在执行或者可执行，此时进程位于执行队列中。 |
|  D   | uninterruptible sleep (usually I/O)<br>不可中断阻塞，通常为 IO 阻塞。 |
|  S   | interruptible sleep (waiting for an event to complete) <br> 可中断阻塞，此时进程正在等待某个事件完成。 |
|  Z   | zombie (terminated but not reaped by its parent)<br>僵死，进程已经终止但是不可被其父进程获取信息。 |
|  T   | stopped (either by a job control signal or because it is being traced) <br> 结束，进程既可以被作业控制信号结束，也可能是正在被追踪。 |

<br>

<div align="center"> <img src="pics/2bab4127-3e7d-48cc-914e-436be859fb05.png" width="490px"/> </div><br>

## SIGCHLD

当一个子进程改变了它的状态时（停止运行，继续运行或者退出），有两件事会发生在父进程中：

- 得到 SIGCHLD 信号；
- waitpid() 或者 wait() 调用会返回。

其中子进程发送的 SIGCHLD 信号包含了子进程的信息，比如进程 ID、进程状态、进程使用 CPU 的时间等。

在子进程退出时，它的进程描述符不会立即释放，这是为了让父进程得到子进程信息，父进程通过 wait() 和 waitpid() 来获得一个已经退出的子进程的信息。

<div align="center"> <!-- <img src="pics/flow.png" width=""/> --> </div><br>

## wait()

```c
pid_t wait(int *status)

```

父进程调用 wait() 会一直阻塞，直到收到一个子进程退出的 SIGCHLD 信号，之后 wait() 函数会销毁子进程并返回。

如果成功，返回被收集的子进程的进程 ID；如果调用进程没有子进程，调用就会失败，此时返回 -1，同时 errno 被置为 ECHILD。

参数 status 用来保存被收集的子进程退出时的一些状态，如果对这个子进程是如何死掉的毫不在意，只想把这个子进程消灭掉，可以设置这个参数为 NULL。

## waitpid()

```c
pid_t waitpid(pid_t pid, int *status, int options)

```

作用和 wait() 完全相同，但是多了两个可由用户控制的参数 pid 和 options。

pid 参数指示一个子进程的 ID，表示只关心这个子进程退出的 SIGCHLD 信号。如果 pid=-1 时，那么和 wait() 作用相同，都是关心所有子进程退出的 SIGCHLD 信号。

options 参数主要有 WNOHANG 和 WUNTRACED 两个选项，WNOHANG 可以使 waitpid() 调用变成非阻塞的，也就是说它会立即返回，父进程可以继续执行其它任务。

## 孤儿进程

一个父进程退出，而它的一个或多个子进程还在运行，那么这些子进程将成为孤儿进程。

孤儿进程将被 init 进程（进程号为 1）所收养，并由 init 进程对它们完成状态收集工作。

由于孤儿进程会被 init 进程收养，所以孤儿进程不会对系统造成危害。

## 僵尸进程

一个子进程的进程描述符在子进程退出时不会释放，只有当父进程通过 wait() 或 waitpid() 获取了子进程信息后才会释放。如果子进程退出，而父进程并没有调用 wait() 或 waitpid()，那么子进程的进程描述符仍然保存在系统中，这种进程称之为僵尸进程。（**可以理解为尸位素餐**）

僵尸进程通过 ps 命令显示出来的状态为 Z（zombie）。

系统所能使用的进程号是有限的，如果产生大量僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程。

要消灭系统中大量的僵尸进程，只需要将其父进程杀死，此时僵尸进程就会变成孤儿进程，从而被 init 进程所收养，这样 init 进程就会释放所有的僵尸进程所占有的资源，从而结束僵尸进程。

# 常用重要命令

## 查看内存、磁盘的命令

​	top 
​		任务管理器
​	free 
​		整体内存使用
​	cat /proc/meminfo 
​		查看RAM使用情况
​	ps aux –sort -rss 
​		列出目前所有的正在内存当中的程序。 
​	vmstat -s
​		vmstat命令显示实时的和平均的统计，覆盖CPU、内存、I/O等内容。例如内存情况，不仅显示物理内存，也统计虚拟内存。
​	netstat  -anp  |grep   端口号
​		LINUX中如何查看某个端口是否被占用

## vim

[vim常用命令总结 （转)](https://www.cnblogs.com/yangjig/p/6014198.html)

### VIM 三个模式

- 一般指令模式（Command mode）：VIM 的默认模式，可以用于移动游标查看内容；
- 编辑模式（Insert mode）：按下 "i" 等按键之后进入，可以对文本进行编辑；
- 指令列模式（Bottom-line mode）：按下 ":" 按键之后进入，用于保存退出等操作。

<div align="center"> <img src="pics/b5e9fa4d-78d3-4176-8273-756d970742c7.png" width="500"/> </div><br>

在指令列模式下，有以下命令用于离开或者保存文件。

| 命令 |                             作用                             |
| :--: | :----------------------------------------------------------: |
|  :w  |                           写入磁盘                           |
| :w!  | 当文件为只读时，强制写入磁盘。到底能不能写入，与用户对该文件的权限有关 |
|  :q  |                             离开                             |
| :q!  |                        强制离开不保存                        |
| :wq  |                        写入磁盘后离开                        |
| :wq! |                      强制写入磁盘后离开                      |

### VIM查找和替换命令

​	查找
​		/oracle [/表示向下查找]
​		?orace [?表示向上查找]
​	单行替换
​		 s/oracle/hello
​			将20行的第一个oracle替换为hello
​		 s/oracle/hello/g
​			将20行的所有oracle替换为hello
​	多行替换
​		A,Bs/oracle/hello
​			参数A表明开始行，B表示结束行，如果B为$，则表示为最后一行。此命令表示从A行开始到B行结束的每行的第一个oracle要替换为hello。
​		A,Bs/oracle/hello/g
​			此命令表示从A行开始到B行结束的每行的每一个oracle要替换为hello。
​	全文替换
​		%s/oracle/hello/g
​			此命令表示将当前文件的所有oracle替换为hello。

## 文本文件找特定字符串

[Linux查找含有某字符串的所有文件](https://www.cnblogs.com/wangkongming/p/4476933.html)

[Linux下xargs命令详解](https://www.cnblogs.com/perfy/archive/2012/07/24/2606101.html)

如果你想在当前目录下 查找"hello,world!"字符串,可以这样:

grep -rn "hello,world!" *

\* : 表示当前目录所有文件，也可以是某个文件名

-r 是递归查找

-n 是显示行号

-R 查找所有文件包含子目录

-i 忽略大小写

下面是一些有意思的命令行参数：

 

grep -i pattern files ：不区分大小写地搜索。默认情况区分大小写， 

grep -l pattern files ：只列出匹配的文件名， 

grep -L pattern files ：列出不匹配的文件名， 

grep -w pattern files ：只匹配整个单词，而不是字符串的一部分（如匹配‘magic’，而不是‘magical’）， 

grep -C number pattern files ：匹配的上下文分别显示[number]行， 

grep pattern1 | pattern2 files ：显示匹配 pattern1 或 pattern2 的行， 

grep pattern1 files | grep pattern2 ：显示既匹配 pattern1 又匹配 pattern2 的行。 

这里还有些用于搜索的特殊符号：

 

\< 和 \> 分别标注单词的开始与结尾。

例如： 

grep man * 会匹配 ‘Batman’、‘manic’、‘man’等， 

grep '\<man' * 匹配‘manic’和‘man’，但不是‘Batman’， 

grep '\<man\>' 只匹配‘man’，而不是‘Batman’或‘manic’等其他的字符串。 

'^'：指匹配的字符串在行首， 

'$'：指匹配的字符串在行尾，  

 

2，xargs配合grep查找

find -type f -name '*.php'|xargs grep 'GroupRecord'

# select、poll、epoll

[select、poll、epoll之间的区别总结](https://www.cnblogs.com/jokezl/p/10140802.html)
[mmap](https://blog.csdn.net/qq_33611327/article/details/81738195)
[select、poll、epoll之间的区别总结](https://www.cnblogs.com/Anker/p/3265058.html)
[Linux下的I/O复用与epoll详解](https://www.cnblogs.com/lojunren/p/3856290.html)

## select，poll

1、poll本质上和select没有区别，它将用户传入的数组**拷贝**到内核空间，然后查询每个fd对应的设备状态，如果设备没有就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前进程，直到设备就绪或者主动超时，被唤醒后它又要再次**遍历fd**。这个过程经历了多次无谓的遍历。

2、select文件描述符fd使用**数组**存储，数量有限，poll使用**链表**存储，数量没有限制

3、需要**拷贝**和**遍历**

4、水平触发：如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd。

## epoll

1、边缘触发：它只告诉进程哪些fd刚刚变为就需态，并且只会通知一次。

2、**mmap**：内核和用户态共享数据，减少复制开销

3、**红黑树**：红黑树将存储epoll所监听的套接字。上面mmap出来的内存如何保存epoll所监听的套接字，必然也得有一套数据结构，epoll在实现上采用红黑树去存储所有套接字，当添加或者删除一个套接字时（epoll_ctl），都在红黑树上去处理，红黑树本身插入和删除性能比较好，时间复杂度O(logN)。

4、**双向链表（就绪队列）**：epoll使用“事件”的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，就会将事件加入就绪链表，epoll_wait只用查询就绪链表便可以收到通知。

**epoll的重要函数**
			新建epoll描述符==epoll_create()
			epoll_ctrl(epoll描述符，添加或者删除所有待监控的连接)
			返回的活跃连接 ==epoll_wait（ epoll描述符 ）
	链接
		[select、poll、epoll之间的区别总结](https://www.cnblogs.com/jokezl/p/10140802.html)
		[mmap](https://blog.csdn.net/qq_33611327/article/details/81738195)
		[select、poll、epoll之间的区别总结](https://www.cnblogs.com/Anker/p/3265058.html)
		[Linux下的I/O复用与epoll详解](https://www.cnblogs.com/lojunren/p/3856290.html)

# 压缩与打包

## 压缩文件名

Linux 底下有很多压缩文件名，常见的如下：

| 扩展名     | 压缩程序                              |
| ---------- | ------------------------------------- |
| \*.Z       | compress                              |
| \*.zip     | zip                                   |
| \*.gz      | gzip                                  |
| \*.bz2     | bzip2                                 |
| \*.xz      | xz                                    |
| \*.tar     | tar 程序打包的数据，没有经过压缩      |
| \*.tar.gz  | tar 程序打包的文件，经过 gzip 的压缩  |
| \*.tar.bz2 | tar 程序打包的文件，经过 bzip2 的压缩 |
| \*.tar.xz  | tar 程序打包的文件，经过 xz 的压缩    |

## 压缩指令

1. gzip

gzip 是 Linux 使用最广的压缩指令，可以解开 compress、zip 与 gzip 所压缩的文件。

经过 gzip 压缩过，源文件就不存在了。

有 9 个不同的压缩等级可以使用。

可以使用 zcat、zmore、zless 来读取压缩文件的内容。

```html
$ gzip [-cdtv#] filename
-c ：将压缩的数据输出到屏幕上
-d ：解压缩
-t ：检验压缩文件是否出错
-v ：显示压缩比等信息
-# ： # 为数字的意思，代表压缩等级，数字越大压缩比越高，默认为 6

```

2. bzip2

提供比 gzip 更高的压缩比。

查看命令：bzcat、bzmore、bzless、bzgrep。

```html
$ bzip2 [-cdkzv#] filename
-k ：保留源文件

```

3. xz

提供比 bzip2 更佳的压缩比。

可以看到，gzip、bzip2、xz 的压缩比不断优化。不过要注意的是，压缩比越高，压缩的时间也越长。

查看命令：xzcat、xzmore、xzless、xzgrep。

```html
$ xz [-dtlkc#] filename

```

## 打包

压缩指令只能对一个文件进行压缩，而打包能够将多个文件打包成一个大文件。tar 不仅可以用于打包，也可以使用 gzip、bzip2、xz 将打包文件进行压缩。

```html
$ tar [-z|-j|-J] [cv] [-f 新建的 tar 文件] filename...  ==打包压缩
$ tar [-z|-j|-J] [tv] [-f 已有的 tar 文件]              ==查看
$ tar [-z|-j|-J] [xv] [-f 已有的 tar 文件] [-C 目录]    ==解压缩
-z ：使用 zip；
-j ：使用 bzip2；
-J ：使用 xz；
-c ：新建打包文件；
-t ：查看打包文件里面有哪些文件；
-x ：解打包或解压缩的功能；
-v ：在压缩/解压缩的过程中，显示正在处理的文件名；
-f : filename：要处理的文件；
-C 目录 ： 在特定目录解压缩。

```

| 使用方式 | 命令                                                  |
| :------: | ----------------------------------------------------- |
| 打包压缩 | tar -jcv -f filename.tar.bz2 要被压缩的文件或目录名称 |
|  查 看   | tar -jtv -f filename.tar.bz2                          |
|  解压缩  | tar -jxv -f filename.tar.bz2 -C 要解压缩的目录        |

# 参考资料

[**CS-Notes**]([https://github.com/CyC2018/CS-Notes/blob/master/notes/Linux.md#%E5%88%86%E5%8C%BA%E4%B8%8E%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F](https://github.com/CyC2018/CS-Notes/blob/master/notes/Linux.md#分区与文件系统))

[**JavaGuide**]([https://github.com/Snailclimb/JavaGuide/blob/master/docs/operating-system/%E5%90%8E%E7%AB%AF%E7%A8%8B%E5%BA%8F%E5%91%98%E5%BF%85%E5%A4%87%E7%9A%84Linux%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86.md#31-linux%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%AE%80%E4%BB%8B](https://github.com/Snailclimb/JavaGuide/blob/master/docs/operating-system/后端程序员必备的Linux基础知识.md#31-linux文件系统简介))
