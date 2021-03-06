# Linux常用命令

`man`：获得帮助信息

* 示例：`man [命令或配置文件]`

`help`：查看内建命令信息

* 示例：`help [命令]`

***

**1. 文件目录相关**

* `ls`：显示目录内容
  * 示例：`ls [目录]`
  * `-a`：显示所有文件信息，包括隐藏文件；`-l`：显示详细信息；`-d`：显示目录信息；`-h`：人性化；`-i`：i节点

* `mkdir`：创建目录
  * `mkdir [目录名]`
  * `-p`：递归创建目录；`cd`：切换目录；`pwd`：显示当前目录；`rmdir`：删除空目录；`cp`：复制文件或目录；`-r`：复制目录；`-p`：保留文件属性
* `mv`：移动文件
  * `mv [文件src] [文件dest]`

* `rm`：删除文件
  * 示例：`rm [文件]`
  * `-r`：删除目录；`-f`：强制删除

* `touch`：创建空文件
  * 示例：`torch [文件]`

* `cat`：显示文件内容
  * 示例：`cat [文件]`

* `tac`：倒着显示文件内容
  * 示例：`cat [文件]`

* `more`：分页显示文件内容
  * 示例：`more [文件]`
  * 空格(或`f`)翻页，`Enter`换行，`q`或`Q`退出

* `less`：分页显示文件内容
  * 示例：`less [文件]`

* `head`：显示文件前几行
  * 示例：`head [文件]`
  * `-n`：指定行数，默认10行

* `tail`：显示文件后面几行
  * 示例：`tail [文件]`
  * `-n`：指定行数；`-f`：动态显示文件末尾内容

* `ln`：创建连接文件
  * 示例：`ln [src] [dest]`
  * `-s`：创建软连接

* `lsof`：展示系统上所有被打开的文件
  * 示例：`lsof -u user`
  * `-u`：指定具体用户打开的所有文件

***

**2. 文件搜索命令**

* `find`：查找文件
  * `find [文件]`
  * `-name/-iname`：区分/不分区大小写； `*/?`匹配多个/一个字符；`-size +-`：指定大小大于或小于具体字节；`-user -group`：指定文件属于哪个用户或组；`-type f d`：指定文件类型是文件还是目录；`-inum`：指定文件i值；`-a -o`：and或or条件；`-exec/-ok [命令] {} \;`：将搜寻结果再执行指定命令。

* `locate`：在文件资料库中查找文件
  * 示例：`locate [文件]`

  * `-i`不区分大小写，`updatedb`更新资料库，无法从临时文件夹`/tmp`下查找文件

* `which`：搜索命令所在目录及别名信息
* 示例：`which [命令]`

* `whereis`：搜索命令所在目录及帮助文档路径
  * 示例：`whereis [命令]`

* `grep`：在文件中搜索字串匹配的行并输出
  * `grep [指定字串] [文件]`
  * `-i`不区分大小写，`-v`排除指定字串

***

**3. 压缩文件命令**

* `gzip`：压缩文件，压缩后格式`.gz`，不能压缩目录，不保留原文件
  * 示例：`gzip [文件]`
* `gunzip`：解压缩`.gz`压缩文件，或者`gzip -d`
  * 示例：`gunzip [压缩文件]`

* `tar`：打包、解压目录，压缩后格式`.tar.gz`
  * 示例：`tar [选项] [压缩后文件名] [目录]`，`tar -zcf name.tar.gz name`，`tar -zxf name.tar.gz`
  * `-c`：打包；`-x`：解包；`-v`：显示详细信息；`-f`：指定文件名；`-z`打包同时压缩/解压缩

* `zip`：压缩文件或目录，压缩后格式`.zip`
  * 示例：`zip [选项] [压缩后文件名] [文件或目录]`
  * `-r`：压缩目录

* `unzip`：解压`.zip`的压缩文件
  * 示例：`unzip [压缩文件]`

* `bzip2`：压缩文件，压缩后格式为`.bz2`
  * 示例：`bzip2 [选项] [文件]`
  * `-k`：产生压缩文件后保留原文件
* `bunzip2`：解压缩
  * 示例：`bunzip2 [选项] [压缩文件]`，`bunzip2 -k name.bz2`，`tar -xjf name.tar.bz2`
  * `-k`：解压缩后保留原文件

| 压缩格式 | 压缩命令 | 解压缩命令 |
| -------- | -------- | ---------- |
| .gz      | gzip     | gunzip     |
| .tar     | tar -cf  | tar -xf    |
| .tar.gz  | tar -zcf | tar -xcf   |
| .zip     | zip -r   | unzip      |
| .bz2     | bzip2    | bunzip2    |
| .tar.gz2 | tar -cjf | tar -xjf   |

***

**4. 网络命令**

* `write`：给另一个用户发送消息
  * 示例：`write [user]`
* `wall`：给所有用户发消息
  * 示例：`wall [message]`
* `ping`：测试网络连通性
  * 示例：`ping [地址]`
  * `-c`：指定发送次数
* `ifconfig`：查看和设置网卡信息
  * 示例：`ifconfig [网卡名称] [IP地址]`

* `whois`：查看域名信息
  * 示例：`whois domain`

* `dig`：展示域名的DNS信息
 * 示例：`dig domain`

* `host`：DNS查询
  * 示例：`host domain`

* `mail`：查看发送电子邮件
  * 示例：`mail [用户]`

* `last`：列出目前与过去登入系统的用户信息
  * 示例：`last`
* `traceroute`：显示数据包到主机间的路径
  * 示例：`traceroute [地址]`

* `netstat`：显示网络相关信息
  * 示例：`netstat [选项]`；`netstat -tlun`：查看本机监听端口；`netstat -an`：查看本机所有的网络连接；`netstat -rn`：查看本机路由表
  * `-t`：TCP协议；`-u`：UDP协议；`-l`：监听；`-r`：路由；`-n`：显示IP地址和端口号

* `setup`：配置网络
  * 示例：`setup`

* `mount`：挂载文件系统，`umount`卸载
  * 示例：`mount [-t 文件系统] 设备文件名 挂载点`

***

**5. 用户管理命令**

* `useradd`：添加新用户
  * 示例：`useradd [用户名]`
  * `password [用户名]`添加密码

* `who`：查看登录用户信息
* 示例：`who`

* `w`：看登录用户信息以及用户的操作信息
  * 示例：`w`

***

**6. 进程相关**

* `strace`：打印一个正在运行的程序和它的子进程调用的每个系统调用的轨迹

* `ps`：列出当前系统中的进程（包括僵死进程）

* `top`：打印出关于当前进程资源使用的信息

* `htop`：`top`替代品，交互式

* `pmap`：显示进程的内存映射

* `/proc`：一个虚拟文件系统，以ASCII文本格式输出大量内核数据结构内容
  * 示例：`cat /proc/loadavg`

***

**7. 硬件信息**

* `dmesg`：显示内核环缓冲区信息

* `cat /proc/cpuinfo`：显示CPU信息

* `cat /proc/meminfo`：显示内存信息

* `free`：显示空闲可用内存
  * `-h`：显示信息更符合人类阅读；`-m`：MB；`-g`：GB

* `df`：报告文件系统磁盘空间使用情况

* `lspci -tv`：显示PCI设备

* `lsusb -tv`：显示USB设备

* `dmidecode`：显示来自BIOS的DMI/SMBIOS硬件信息

* `hdparm -i /dev/sda`：显示sda磁盘信息

* `hdparm -tT /dev/sda`：测试磁盘sda的读取速度

* `badbloacks -s /dev/sda`：测试磁盘sda上的不可读块

***

**8. 其它**

* `shutdown`：关机重启
  * 示例：`shutdown [选项] 时间`，`shutdown -h now`
  * `-c`：取消前一个关机命令；`-h`：关机；`-r`：重启

* 其它重启命令：`reboot`、`init 6`、`halt`、`poweroff`、`init 0`
* 系统运行级别：0：关机；1：单用户；2：不完全多用户，不含NFS服务；3：完全多用户；4：未分配；5：图形界面；6：重启；

* `runlevel`：打印当前系统运行级别
