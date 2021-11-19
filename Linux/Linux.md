#  :pencil:VMtools

2021年11月14日10:16:52

1. 切换到opt目录：`cd /opt`
2. 查看该目录下的文件：`ls`
3. 解压命令：`tar -zxvf VM....tar.gz`  (可以按tab键补齐)
4. `cd vmware-tools-distrib/`
5. 安装：`./vmware-install.pl`  （./vm……tab补齐）

- VMware：**拍摄快照一定要在虚拟机关闭的情况下拍摄快照。**

##  共享文件夹

- VMware-》虚拟机设置-》选项-》共享文件夹-》总是启用-》添加
- 主文件夹-》其他位置-》计算机-》-》mnt目录-》hgfs目录-》可以看到Windows共享的文件夹

#  :pencil:目录结构详解

- 在根目录下：添加用户：`useradd jack`
  - 删除用户：`userdel -r jack`

```markdown
bin   dev  home  lib64    media  opt   root  sbin  sys  usr
boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
```

1. 配置环境变量的配置文件：`vim /etc/profile`
   1. `export JAVA_HOME=/usr/local/java/jdk1.8.0_161`
   2. `export PATH =$ JAVA_HOME/bin:$PATH`

#  :pencil:Xshell7&Xfpt7远程登录

##  ifconfig

- 用Xshell7登录Linux公网：需要知道Linux的IP地址：`ifconfig`

```shell
[root@hrjCentos7 ~]# ifconfig
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.168.128  netmask 255.255.255.0  broadcast 192.168.168.255
        inet6 fe80::362d:9f23:d8e0:e02b  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:f3:0f:4b  txqueuelen 1000  (Ethernet)
        RX packets 307  bytes 39680 (38.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 215  bytes 25025 (24.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 32  bytes 2592 (2.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 32  bytes 2592 (2.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:19:d3:19  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```shell
[root@hrjCentos7 ~]# who am i
root     pts/0        2021-11-15 12:40 (192.168.168.1)
```

- windows ->cmd ：`ping 输入查看到的IP地址`（192.168.168.128），如果没有问题就说名两台机器（不同的操作系统）之间是可以通信的。
- 用Xshell7登录：新建-》名称：192.168.168.128[hrjCentos]-》协议-》SSH-》主机：192.168.168.128 =》端口22 =》连通之后：**接收并保存**
  - 使用Xshell7与Linux操作系统的主机连接通信后，在Xshell7界面输入：`reboot` 指令可以重启Linux。
- Xftp7：协议选择 SFTP，端口号22，主机选择：查到的Linux地址。`ifconfig`

#  :pencil:vi&vim文本编辑器

##  使用xshell在Linux下编写一个程序

1. 输入 `vim hello.java`（表示我们编写的文件的名字为hello.java）
2. 输入i。进入编辑操作
3. 进行运行。按下`esc`键，再输入冒号：`wq`表示写入并退出

##  快捷键

1. 进入插入模式：`i`
2. 如果错按了 `Ctrl+s` 想要保存，屏幕锁住了，解锁：`Ctrl+q`
3. 在命令行下：保存退出：`:wq` 或者`shift+zz` 、退出不保存：`:q`、 强制退出不保存：`:q!`
4. 命令行模式：先`esc`，再输入`:`
5. 拷贝当前行 `yy`, 拷贝当前行向下的5行 `5yy`,并粘贴（`输入p`）
6. 删除当前行 `dd`，删除当前行向下的5行 `5dd`
7. 在文件中查找某单词【`/关键字`，回车 查找，输入`n就是查找下一个`，`N上一个`】
8. 显示行号：在正常模式下，输入 `:set nu` ;如果去掉行号，在正常模式下：`:set nonu`
9. 编辑/etc/profile 文件，在一般模式下，使用快捷键 `G` 到该文档的最末行，使用 `gg` 到最首行。
10. 编辑一个文件时，如果想要 **撤销** 刚刚的输入操作，在一般模式下，输入`u`（u=undo）
    1. Ctrl + r 返回到undo之前（r=）
11. 编辑/etc/profile文件，在一般模式下，定位到20行，`输入20,再shift+g`  或者直接 `20gg`
    1. 或者 `:`20，快速定位到20行
    2. 无人值守vim：`yum install -y vim` 


#  :pencil:Linux 实操

##  关机重启

>不管是重启系统还是关闭系统，首先要运行 ==`sync`==命令，把内存中的数据写到磁盘中。

1. **立刻进行关机**：`shutdown -h now`  (h代表halt)  马上关闭计算机，但是可以给其他用户发送消息 【或者：`init 0`】 
2. 一分钟后（默认）关机（会发送信息到每个用户终端）：`shutdown -h 1`
3. 现在重新启动计算机：`shutdown -r now `（r代表reboot）【或者：`init 6`】 
4. 关机（与上述类似）：`halt` 表示直接拔掉电源  或者 `poweroff` 表示直接关闭机器，但是有可能当前虚拟机其他人在使用
5. 立刻重新启动计算机：`reboot`
6. **把内存的数据同步到磁盘**：`sync`

##  用户登录和注销

1. 登录时尽量少用root账号登录，可以利用普通用户登录，登录后再用 `su - root` 命令来切换成系统管理员身份。
   1. 如：用tom登录后执行，`cd /root `发现权限不够，可以切换成系统管理员：su - root
2. `logout` 即可退出用户，在 Centos 图形化界面无效，退出终端只能用`exit`。
   1. 用Xshell 界面操作，如果是tom，输入logout后，就会注销用户，退出系统；如果是tom切换成的root，输入logout，就会注销root，来到tom用户权限，再次输入logout才是退出系统。

##  用户管理

>Linux 系统是一个多用户多任务的操作系统，任何一个要使用系统资源的用户，都必须首先向系统管理员申请一个账号，然后以这个账号的身份进入系统

1. 查看有哪些用户：在Xshell 界面,输入`cd /home`  然后 `ls` 
2. 添加用户：`useradd 用户名`  

   1. 如：`useradd Jack` 默认该用户的家目录在 `/home/Jack` 下
3. 如果想要给新创建的用户指定家目录：`useradd -d 指定目录 新的用户名`

   1. `useradd -d /home/test Jack`   指定用户Jack的家目录是test
4. 指定密码：`passwd 用户名`

   1. 如：给root设置密码：`passwd root `
5. 显示当前用户在哪个目录下面：`pwd`
6. 删除用户：`userdel 用户名`  【一般情况下保留家目录】

   1. 如：删除用户Jack，但是要保留家目录：`userdel Jack`
   2. 如：删除用户Jack以及Jack的主目录：`userdel -r Jack`  【**这个操作一定要慎重**】
7. 查询用户信息指令：`id 用户名`  id root
8. **切换用户**：`su - 用户名`  如：`su - root`  【**su =  super user**】
   1. 从权限高的用户切换到权限低的用户，不需要输入密码，反之需要
   2. 当需要返回到原来的用户时，使用 `exit` 或 `logout` 指令
9. 查看当前用户/登录用户：`whoami` 或者 `who`
   1. -m或am I 只显示运行who命令的用户名，登录终端和登录时间
   2. -q或–count 只显示用户的登录账号和登录用户的数量 ,`who --count`


>用户组：
>
>- 类似于角色，系统可以对有共性/权限的多个用户进行统一的管理

1. 新增组：`groupadd 组名`
2. 删除组：`groupdel 组名`
3. 增加用户时并加上组：`useradd -g 用户组 用户名`
   1. 如：先创建一个huawei组：`groupadd huawei`，再 `useradd -g huawei Rose`
4. 修改组：`usermod -g 用户组 用户名`
   1. 如：`useradd -g tom Rose`  【tom组是用户tom默认设置的分组】

>用户和组相关文件

1. `vim /etc/passwd`：用户（user）的配置文件，记录用户的各种信息
   1. 每行的含义：用户名：口令：用户标识号：组标识号：注释性描述：主目录：登录shell
2. `vim /etc/shadow`：口令的配置文件
   1. 每行的含义：登录名：加密口令：最后一次修改时间：最小时间间隔：最大时间间隔：警告时间：不活动时间：失效时间：标志
3. `vim /etc/group`：组（group）的配置文件，记录Linux包含的组的信息
   1. 每行含义：组名：口令：组标识号：组内用户列表 

>指定运行级别
>
>>0：关机 1：单用户（找回丢失密码） 2：多用户状态没有网络服务 **3：多用户状态有网络服务** 4：系统未使用保留给用户 **5：图形界面** 6：系统重启
>>
>>- 常用的运行级别是 **3和5**，也可以指定默认运行级别。

1. 例如：从级别5切换到运行级别3：`init 3`
2. 查看当前的运行级别：`systemctl get-default`   【**ctl = control**】
   1. 如果是图形化界面显示：**graphical.target**
3. 从5切换到3：`systemcl set-default multi-user.target`
   1. 多用户状态有网络服务：**multi-user.target**

##  找回root密码

[找回root密码](https://blog.csdn.net/Mr_GYF/article/details/112852340)

1. 重启系统，进入开机界面按"`e`"进入编辑界面
2. 进入编辑界面，使用键盘上的上下键把光标往下移动，找到以“linux16”开头内容所在的行数，在行的最后输入：`init=/bin/sh`
3. 输入完成后，直接按快捷键：`Ctrl+x`进入单用户模式
4. 在光标闪烁的位置中输入：`mount -o remount,rw /`（各个单词之间有空格）
5. 在新的一行最后面输入：`passwd`，完成后按键盘的回车键。输入密码，然后再次确认密码即可
6. 在光标闪烁的位置输入：`touch /.autorelabel`(touch 与斜杠之间有空格)，完成后按回车
7. 在光标闪烁位置输入：`exec /sbin/init`（exec与后面斜杠有一个空格）耐心等待系统完成即可，新密码即可生效

##  帮助指令(man/help)

>在linux下，隐藏文件是以`"."`开头的

1. man获得帮助信息：`man 命令或配置文件`
   1. 比如查看ls命令的帮助信息：`man ls`
2. help获得shell内置命令的帮助信息：`help 命令`
   1. 如查看cd命令的帮助信息：`help cd`

##  文件目录类(cd/ls/pwd/mkdir/rmdir/touch/cp/

##  rm/mv/cat/more/less/echo/head/tail/>/>>/ln)

<img src="imgs\tree.png" align="left" />

1. cd指令：切换到指定目录

   1. `cd ~`  表示回到自己的家目录

   2. `cd..`表示回到当前目录的上一级目录

   3. >使用相对路径切换到root目录，比如在/home/tom

      [root@hrjCentos7 /]# cd /home/tom/
      [root@hrjCentos7 tom]# pwd
      /home/tom
      [root@hrjCentos7 tom]# ==cd ../../root/==
      [root@hrjCentos7 ~]# pwd
      /root

2. 列举当前目录下的内容（不包含隐藏文件）`ls` `ll`

   1. 列举所有文件：`ls -a`
   2. 单列输出所有文件(不包含隐藏文件)：`ls -l`   一般情况下文件的大小都是显示的字节，用人类看得懂的方式显示`ls -lh`
   3. 单列输出所有文件(包含隐藏文件)【选项可以组合使用】：`ls -la`   或者 `ls -al` 或者 `ls -al /root` `ls -al /home`

3. pwd指令：显示当前工作目录的绝对路径 【绝对路径从`/` 目录开始，相对路径从当前路径开始】

4. mkdir指令：`mkdir[选项]要创建的目录`

   1. 创建单级目录：`mkdir /home/dog`
   2. 创建多级目录：【必须打开选项】：`mkdir -p /home/animal/tiger`

5. rmdir指令：`rmdir[选项] 要删除的空目录`

   1. 功能：删除空目录，比如：`rmdir /home/dog` 【删除的是空目录，如果目录下有内容是无法删除的】
   2. 想要**删除非空目录**，需要使用指令:`rm -rf 要输出的目录`   **(该行为较危险)**
      1. 比如：使用rmdir删除home/animal不成功：`rmdir /home/animal/`
      2. 强制删除非空目录：`rm -rf /home/animal/`   ==【强制递归：**Forced recursion**】==

6. touch指令创建空文件：`touch 文件名称`  如：`touch hello.txt`

   1. 如果创建的文件已经存在，且内容一样，则会更新创建时间。

7. cp指令拷贝文件到指定目录：`cp[选项]source dest`   常用选项：`-r` **递归复制整个文件夹**

   1. 例一：将/home/hello.txt 拷贝到 /home/bbb 目录下 `cp hello.txt bbb/`
   2. 例二：**递归复制整个文件夹**，比如将/home/bbb 整个目录（包括里面的文件）,拷贝到/opt目录下 `cp -r /home/bbb /opt`
      1. 问题：如果出现一个文件夹复制到另外一个目录中后，文件存在覆盖问题。但是Liunx会提示用户是否需要进行覆盖。
      2. 强制覆盖不提示的方法：`\cp[选项]source dest`

8. rm指令：`rm[选项] 要删除的文件或目录` ;如果不想要提示，则使用`rm -f`

   1. `-r`:递归删除整个文件夹【谨慎使用】；`-f`：强制删除且不提示 ：`rm -r /home/Rose`  （执行userdel Rose后）
   2. 比如：删除home目录下的空文件hello.txt：①`cd /home` ②`ls` ③ `rm hello.txt` 或者 `rm -f hello.txt `
   3. **不建议 `-rf`  一起使用**

9. ==mv指令==移动文件与目录或重命名：注意细节：【重命名表示移动到相同目录下，移动文件表示移动到别的目录下】

   1. 重命名：`mv oldNameFile newNameFile`

   2. 移动文件：`mv /temp/movefile /targetFolder` 

   3. ```shell
      [root@hrjCentos7 home]# touch cat.txt
      [root@hrjCentos7 home]# ls
      cat.txt  Rose  temp1  temp2  tom
      [root@hrjCentos7 home]# mv cat.txt pig.txt
      [root@hrjCentos7 home]# ls
      pig.txt  Rose  temp1  temp2  tom
      移动文件的同时重命名：
      [root@hrjCentos7 temp1]# ls
      hello1.txt  pig.txt
      [root@hrjCentos7 temp1]# mv pig.txt /home/temp2/inner2/cat.txt
      [root@hrjCentos7 temp1]# cd /home/temp2/inner2
      [root@hrjCentos7 inner2]# ls
      cat.txt
      ```

   4. 移动整个目录，比如将 /opt/bbb 移动到 /home下：`mv /opt/bbb /home`

10. cat指令查看文件内容：`cat [选项][要查看的文件]`   ==`-n`表示显示行号==

    1. `vim` 同样也可以查看文件，但是vim可以修改，cat不能进行修改，【相对一些重要文件配置只进行cat查看即可】
    2. 为了浏览方便一般会带上`管道指令|more`  如：==`cat -n /etc/profile | more`==

11. more指令用户交互：基于VI编辑器的文本过滤器。它以全屏幕的方式按页显示文本文件的内容。more指令中内置了若干快捷键

    1. `空格键`	代表向下翻一页
    2. `Enter`	代表向下翻一行
    3. `q	`代表立刻离开more不再显示该文件内容
    4. `Ctrl+F`	向下滚动一屏
    5. `Ctrl+B`	返回上一屏			
    6. `=`输出当前行的行号
    7. `:f`	输出文件名和当前行的行号
    8. `more /etc/profile`  

12. less指令【查看大文件】：`less  /home/temp1/杂文.txt`  `less /etc/profile`

    1. /输入要查找的内容， n下一个查找到的单词，N上一个查找到的单词

13. echo指令输出内容到控制台：`echo[选项] [输出内容]`

    1. **使用echo指令输出环境变量**，比如输出 $PATH $HOSTNAME `echo $PATH` `echo $HOSTNAME`
    2. 使用echo指令输出语句：`echo hello,world!` 

14. head指令显示文件的开头部分内容

    1. `head /etc/profile ` 默认情况下head指令显示文件的前10行内容
    2. `head -n 5 /etc/profile`  查看文件头5行的内容

15. tail指令用于输出文件尾部内容

    1. `tail /etc/profile`  ，默认情况下tail指令显示文件的尾10行内容
    2. `tail -n 5 /etc/profile` , 查看文件尾5行内容
    3. 实时追踪该文档的所有更新：`tail -f /home/hello.txt`  (在Xshell7界面输入这段指令，在Centos7的Linux界面输入：`echo "helloWorld!" > /home/hello.txt` )
    4. 退出跟踪：`Ctrl+c`

16. ==`>`指令和 `>>` 指令==：`>`表示  **重定向（覆盖）**,`>>`表示  **追加**；

    1. 列表的内容写入到某文件中（覆盖写）：`ls -l > 文件`  

       1. `ls -l /home > /home/info.txt`
    2. 列表的内容追加到某文件的末尾：`ls -al >> 文件` 

       1. `cal >> /home/temp3`  （如果temp3没有则会创建）（`cal` 显示当前日历信息）
    3. 将文件1的内容覆盖到文件2：`cat 文件1 > 文件2` 

       1. `cat /etc/profile > /home/myprofile`
    4. 控制台内容追加到文件中:`echo “内容” >> 文件` 

       1.  `echo "HELLOWORLD" >> /home/info.txt`

```shell
案例一：将/home目录下的文件列表写入到/home/info.txt中，覆盖写入（如果info.txt没有则会创建）
[root@hrjCentos7 home]# ls -l /home
总用量 16
drwx------.  3 Rose huawei 4096 11月 14 21:22 Rose
drwxr-xr-x.  2 root root   4096 11月 15 15:43 temp1
drwxr-xr-x.  3 root root   4096 11月 15 15:14 temp2
drwx------. 15 tom  tom    4096 11月 14 23:20 tom
[root@hrjCentos7 home]# ls -l /home > /home/info.txt
[root@hrjCentos7 home]# ls
info.txt  Rose  temp1  temp2  tom
[root@hrjCentos7 home]# cat info.txt
总用量 16
-rw-r--r--.  1 root root      0 11月 15 17:20 info.txt
drwx------.  3 Rose huawei 4096 11月 14 21:22 Rose
drwxr-xr-x.  2 root root   4096 11月 15 15:43 temp1
drwxr-xr-x.  3 root root   4096 11月 15 15:14 temp2
drwx------. 15 tom  tom    4096 11月 14 23:20 tom
[root@hrjCentos7 home]# echo "HELLOWORLD" >> /home/info.txt
[root@hrjCentos7 home]# cat info.txt
总用量 16
-rw-r--r--.  1 root root      0 11月 15 17:20 info.txt
drwx------.  3 Rose huawei 4096 11月 14 21:22 Rose
drwxr-xr-x.  2 root root   4096 11月 15 15:43 temp1
drwxr-xr-x.  3 root root   4096 11月 15 15:14 temp2
drwx------. 15 tom  tom    4096 11月 14 23:20 tom
HELLOWORLD
[root@hrjCentos7 home]# echo "OK" > /home/info.txt
[root@hrjCentos7 home]# cat info.txt
OK
```

```shell
案例二：将当前日历信息追加>>到 /home/temp1文件夹中
[root@hrjCentos7 home]# ls
info.txt  Rose  temp1  temp2  tom
[root@hrjCentos7 home]# cal >> /home/temp3  （如果temp3没有则会创建）
[root@hrjCentos7 home]# cat /home/temp3
        十一月 2021    
   日 一 二 三 四 五 六
       1  2  3  4  5  6
    7  8  9 10 11 12 13
   14 15 16 17 18 19 20
   21 22 23 24 25 26 27
   28 29 30
```

17. ln软链接也成为符号链接，类似于windows中的快捷方式，主要存放了链接其他文件的路径  【**ln = link**】

    1. 给原文件创建一个软链接：`ln -s[原文件或目录][软连接名]`  
    
2. 案例：在/home 目录下创建一个软连接 myroot，连接到 /root目录：`ln -s /root /home/myroot` 删除：`rm /home/myroot`

```shell
[root@hrjCentos7 home]# ln -s /root /home/myroot
[root@hrjCentos7 home]# ls
info.txt  myroot  Rose  temp1  temp2  temp3  tom
[root@hrjCentos7 home]# ls -l
总用量 24
-rw-r--r--.  1 root root      3 11月 15 17:31 info.txt
lrwxrwxrwx.  1 root root      5 11月 15 20:15 myroot -> /root
drwx------.  3 Rose huawei 4096 11月 14 21:22 Rose
drwxr-xr-x.  2 root root   4096 11月 15 15:43 temp1
drwxr-xr-x.  3 root root   4096 11月 15 15:14 temp2
-rw-r--r--.  1 root root    146 11月 15 17:36 temp3
drwx------. 15 tom  tom    4096 11月 14 23:20 tom
[root@hrjCentos7 home]# rm /home/myroot
rm：是否删除符号链接 "/home/myroot"？y
```

18. history查看已经执行过历史命令，也可以执行历史命令
    1. 查询所有历史命令：`history`
    2. 显示最近十条历史命令：`history 10`
    3. 执行编号为5的指令：`!5`

##  时间日期类指令(date/date -s)

2021年11月15日22:54:55

>1.date指令——显示当前日期

- `date`：显示当前时间
- `date+%Y`：显示当前年份,`date+%m`：显示当前月份,`date+%d`：显示当前哪一天,`date"+%Y-%m-%d %H:%M:%S"`：显示年月日时分秒
- 设置系统当前时间为，`date -s "2021-1-1 12:22:22"`
- 不加选项，显示本月日历：`cal` ，显示全年日历：`cal -y`

##  搜索查找类(find/locate/which/grep)

1. find指令：从指定目录向下递归的，遍历其各个子目录，将满足条件的文件或目录显示在终端。`find 搜索范围 选项`
   1. 选项：按照指定的文件名查找模式查找文件  `-name<查询方式>  `
      1. 案例：按文件名，根据名称查找 /home目录下的hello.txt文件：`find /home -name hello.txt`
      1. `find / -name java`  
      1. `whereis java`
   2. 选项：查找属于指定用户名所有文件 `-user<用户名>`
      1. 案例：按拥有者，查找/opt目录下，用户名称为 nobody的文件： `find /opt -user nobody`
   3. 选项：按照指定的文件大小查找文件 `-size<文件大小>` (+200M：大于200M的文件，-200M：小于200M的文件，200M：等于200M文件)  , 一般情况下文件的大小都是显示的字节，用人类看得懂的方式显示  `ls -lh`
2. locate指令：①可以快速定位文件路径、②利用事先建立的系统中所有文件名称及路径的locate数据库实现快速定位给定的文件、③无需遍历整个文件系统，查询速度较快。为了保证查询结果的准确度，管理员必须定期更新locate时刻
   1. 由于locate指令基于数据库进行查询，所以第一次运行前，必须使用 `updatedb` 指令创建locate数据库，如：`locate info.txt`

3. which指令：查看某个指令在哪个目录下，比如查看ls指令，可以使用   `which ls`
4. grep指令：过滤查找，管道符，“|”，表示将前一个命令的处理结果输出传递给后面的命令处理 `grep[选项]查找内容 源文件`
   1. 选项`-n` 显示匹配行及行号，选项`-i` 忽略字母大小写
   2. 例如在calender.txt文件中，查找 “1"所在行，并且显示行号
      1. `cat calender.txt | grep -n 1`
      2. `grep -n 1 /home/calender.txt`


## 压缩和解压类(gzip/gunzip/zip/unzip/tar)

1. `gzip 文件`：压缩文件，只能将文件压缩为 *.gz文件; `gunzip 文件.gz`：解压缩文件命令
   1. 案例一：将/home下的hello.txt文件进行压缩：`gzip /home/info.txt`
   2. 案例二：将/home下的hello.txt.gz文件进行解压缩：`gunzip /home/info.txt.gz`
   
2. 压缩文件和目录的命令：`zip [选项]XXX.zip 将要压缩的内容`  ；解压缩文件：`unzip[选项]XXX.zip`
   1. zip常见选项：`-r`：递归压缩，即压缩目录
      1. 案例一：将/home下的 所有文件（包括文件夹）进行压缩成 myhome.zip (将home目录及其包含的文件和子文件夹都压缩)
      2.   `zip -r myhome.zip /home/`
   2. unzip常见选项：`-d<目录>`： 指定解压后文件的存放目录
      1. 案例二：将myhome.zip解压到 /opt/tmp目录下  
      2. `mkdir /opt/tmp`   `unzip -d /opt/tmp /home/myhome.zip`
   
3. tar打包指令，最后打包后的文件是.tar.gz的文件：`tar[选项] XXX.tar.gz 打包的内容`
   1. 选项：产生.tar打包文件`-c`，显示详细信息`-v`，指定压缩后的我呢见名`-f`，打包同时压缩 `-z`，解包.tar文件`-x`	
      1. `-f` 后面要跟参数（file），所以要放到最后一个，别的顺序随意
   2. 案例一：压缩多个文件，将/home/pig.txt 和 /home/cat.txt 压缩成 pc.tar.gz 
      1.  `tar -zcvf pc.tar.gz /home/pig.txt /home/cat.txt`
   3. 案例二：将/home的文件夹压缩成myhome.tar.gz
      1. `tar -zcvf myhome.tar.gz /home/`
   4. 案例三：将pc.tar.gz 解压到当前目录
      1. `tar -zcvf pc.tar.gz`
   5. 案例四：将myhome.tar.gz 解压到/opt/tmp2目录下  【==-C 是参数，指定解压目录==】
      1. `mkdir /opt/tmp2`      `tar -zxvf /home/myhome.tar.gz -C /opt/tmp2` 
   
   ```shell
   [root@junhrCentos7 ~]# cd /opt/jdk
   [root@junhrCentos7 jdk]# ls
   jdk-8u161-linux-x64.tar.gz
   [root@junhrCentos7 jdk]# tar zxvf jdk-8u161-linux-x64.tar.gz 
   ```

## 文件/目录（所有者/所有组/其他组）

>a.所有者：一般为文件的创建者，谁创建了该文件，就自然的成为了该文件的所有者。

### 查看文件的所有者：`ls -ahl` 

### 修改文件的所有者：`chown 用户名 文件名`

1. 案例一：使用root创建一个文件apple.txt，然后将其所有者修改成tom

```shell
[root@hrjCentos7 home]# touch apple.txt
[root@hrjCentos7 home]# ls -ahl
-rw-r--r--.  1 root  root    0 11月 16 11:01 apple.txt
[root@hrjCentos7 home]# chown tom apple.txt
[root@hrjCentos7 home]# ls -ahl
-rw-r--r--.  1 tom  root    0 11月 16 11:01 apple.txt
```
>b.所有组：当某个用户创建了一个文件后，这个文件的所在组就是该用户所在的组
- 查看文件/目录所在组：实例：创建用户fox，并创建一个文件，看看该文件属于哪个组

```shell
[root@hrjCentos7 home]# useradd -g tom fox
[root@hrjCentos7 home]# passwd fox
passwd：所有的身份验证令牌已经成功更新。
[root@hrjCentos7 home]# su fox
[fox@hrjCentos7 ~]$ touch fox.txt
[fox@hrjCentos7 ~]$ ll
总用量 0
-rw-r--r--. 1 fox tom 0 11月 16 11:31 fox.txt
```

###  修改文件所在的组：`chgrp 组名 文件名` 

```shell
[root@hrjCentos7 ~]# ll /home
总用量 32
-rw-r--r--.  1 tom  root    0 11月 16 11:01 apple.txt
[root@hrjCentos7 ~]# chgrp tom /home/apple.txt
[root@hrjCentos7 ~]# ll /home
总用量 32
-rw-r--r--.  1 tom  tom     0 11月 16 11:01 apple.txt
```

>c.其他组：除文件的所有者和所在组的用户外，系统的其他用户都是文件的其他组

###  改变用户所在组：`usermod -g 新组名 用户名`  

- 在添加用户时，可以指定该用户添加到哪个组中，同样的用root的管理权限可以改变某个用户所在的组

1.   ==修改之前查看是否存在某个组：`cat /etc/group | grep tom`==

```shell
[root@hrjCentos7 ~]# cat /etc/group | grep tom
tom:x:1000:tom
[root@hrjCentos7 ~]# id fox
uid=1001(fox) gid=1000(tom) 组=1000(tom)
[root@hrjCentos7 ~]# cat /etc/group | grep huawei
huawei:x:1001:
[root@hrjCentos7 ~]# usermod -g huawei fox
[root@hrjCentos7 ~]# id fox
uid=1001(fox) gid=1001(huawei) 组=1001(huawei)
```
###  改变该用户登录的初始目录：`usermod -d 目录名 用户名`

- 【用户需要有进入到新目录的权限】

##  权限基本介绍
- ls -l显示的内容

```shell
[root@hrjCentos7 ~]# ls -l /home
总用量 32
-rw-r--r--.  1 tom  tom       0 11月 16 11:01 apple.txt
lrwxrwxrwx.  1 root root      8 11月 15 22:09 temp3 -> info.txt
drwxr-xr-x.  3 root root   4096 11月 16 10:37 temp4
drwx------. 15 tom  tom    4096 11月 15 21:26 tom
[root@hrjCentos7 /]# ls -l /dev
总用量 0
crw-rw----. 1 root video    10, 175 11月 15 21:23 agpgart
brw-rw----+ 1 root cdrom    11,   0 11月 15 21:23 sr0
lrwxrwxrwx. 1 root root          15 11月 15 21:23 stderr -> /proc/self/fd/2
```
1. 第0位：确定文件类型（d,-,l,c,b）

   - `l`：表示链接，相当于window中的快捷方式；

   - `-`：表示普通文件，如txt文件等：

   - `d`：表示目录，相当于windows中的文件夹；

   - `c`：表示字符设备文件，鼠标，键盘等；

   - `b`：表示是块设备，如硬盘等

2. 第1-3位：确定所有者（该文件的所有者） 拥有该文件的权限——User 
   1. lrwxrwxrwx.  1 root root  8 11月 15 22:09 temp3 -> info.txt ：比如里面的`rwx`代表可读read，可写write，可执行execute

3. 第4-6位：确定所属组（同用户组的）拥有该文件的权限——Group ：比如`rwx` ，代表同组的所有用户都对该文件有：可读，可写，可执行操作。

4. 第7-9位：确定其他用户（其他组的用户）拥有该文件的权限——Other

###  rwx权限详解

1. rwx作用到**文件**：
   - `【r】`代表可读（read）：可以读取，查看
   - `【w】`代表可写（write）：可以修改，但是不代表可以删除文件，删除一个文件的前提条件是对该文件所在目录有写权限，才能删除该文件
   - `【x】`代表可执行（execute）：可以被执行

2. rwx作用到**目录**：
   - `【r】`代表可读（read）：可以读取，ls查看目录内容
   - `【w】`代表可读（write）：可以修改，对目录内创建+删除+重命名目录
   - `【x】`代表可读（execute）：可以进入该目录

3. 其他说明如：-rw-r--r--.  1 tom  tom  0 11月 16 11:01 apple.txt
   - rwx可以用数字表示，其中 `r=4;w=2;x=1`
   - `1`  表示文件：硬连接数或目录：子目录
   - `tom` 表示用户 `tom`  表示组 `0` 表示文件大小（字节）`时间` 表示最后修改日期  `最后一栏表示文件名`

###  chmod修改权限

>通过chmod指令，可以修改**文件或目录**的权限

1. 第一种方式：`+，-，=` 变更权限，u：所有者 g：所在组 o：其他人 a：所有人（ugo的总和）
   - `chmod u=rwx,g=rx,o=x 文件/目录名`（表示直接给权限）
   - `chmod o+w 文件/目录名`  (表示单独添加w权限)
   - `chmod a-x 文件/目录名`（表示去掉x权限），`chmod u-x,g-x,o-x  aa.txt`

- 案例一：给abc文件的所有者读写执行的权限，给所在组读执行权限，给其他组读执行权限：`chmod u=rwx,g=rx,o=rx abc.txt`
- 案例二：给abc文件的所有者除去执行的权限，增加组写的权限：`chmod u-x,o+w abc.txt`
- 案例三：给abc文件的所有用户添加读的权限：`chmod a+r abc.txt`

2. 第二种方式：通过数字变更权限
   - `r =4 w=2 x=1`     `rwx=r+w+x=7`    
   - `chmod u=rwx,g=rx,o=x 文件目录名`   相当于   `chmod 751 文件目录名`

- 案例一：将/home/abc.txt文件的权限修改程rwxr-xr-x，使用给数字的方式实现：`chmod 755 /home/abc.txt`

```shell
[root@hrjCentos7 /]# ls -l /home
总用量 32
-rw-r--r--.  1 tom  tom       0 11月 16 11:01 apple.txt
[root@hrjCentos7 /]# chmod 765 /home/apple.txt 
[root@hrjCentos7 /]# ls -l /home
总用量 32
-rwxrw-r-x.  1 tom  tom       0 11月 16 11:01 apple.txt
[root@hrjCentos7 /]# chmod u=rw,g=r,o=r /home/apple.txt
[root@hrjCentos7 /]# ls -l /home
总用量 32
-rw-r--r--.  1 tom  tom       0 11月 16 11:01 apple.txt
```

### chown修改文件/目录所有者

1. 查看文件的所有者：`ls -ahl`
2. 改变所有者：**`chown newowner 文件/目录名`**：`chown root /home/apple.txt`
3. 改变所有者和所在组：**`chown newowner:newgroup 文件/目录`**： 
   - **`-R`**：如果是目录 则使其下所有子文件或目录递归生效

###  chgrp修改文件/目录所在组

>改变用户所在的组：`usermod -g 新组名 用户名`

1. 改变所在组：`chgrp newgroup 文件/目录`：如果想要修改修改一整个文件，需要加 `-R` 。与上面修改所有者一样
   - `chgrp -R shaolin /home/test`

###  理解文件夹rwx的读写权限

1. 用户组g：对一个目录的读 `r` 权限：表示能否用  `ls` 查看该目录下的文件列表
2. 用户组g：对一个目录的写 `w` 权限：表示能否在该文件下 执行`touch xx.txt` 或者`rm monkey.java`
3. `x` ：表示可以进入该目录，比如：`cd`

##  crond任务调度：`crontab [选项]`

- `-e`	编辑crontab定时任务
- `-l`	查询crontab任务
- `-r`	删除当前用户所有的crontab任务

##  at定时任务

#  :pencil:Linux分区

##  查看所有设备挂载情况

- `lsblk` 或者 `lsblk -f`

##  磁盘情况查询

1. 查询系统整个磁盘使用情况：`df -h`
2. 查询指定目录的磁盘占用情况：du -hac --max-depth=1 /opt

##  工作实用指令

###   统计/opt文件夹下的文件的个数

- ` ls -l /opt | grep "^-"| wc -l`  ：①第一条显示所有的，②第二条进行过滤表示以-（文件）开头的 ，③第三条表示显示数目

###  统计/opt文件夹下的目录的个数

- `ls -l /opt | grep "^d"| wc -l` ：②第二条进行过滤表示以d（目录）开头的 

###  统计/opt文件夹下的文件的个数，包括子文件夹里的

- `ls -lR /opt | grey "^-" |wc -l` ：①加上R,R表示递归，将子目录下的东西全部递归

###  统计/opt文件夹下目录的个数，包括子文件夹里的

- `ls -lR /opt | grey "^d" |wc -l`  

###  以树状显示目录结构

- 注意一开始使用是没有tree指令的，需要先下载，其指令为：**`yum install tree`**，随后不断输入`y`表示确定即可
- 指令： **`tree 文件夹名`**

#  :pencil:网络配置

- Linux：`ifconfig`  inet：192.168.168.128
- windows：`ipconfig`  VMnet8：IPv4地址：192.168.168.1
  - 两者属于同一个 网段

##  Linux网络环境配置指定ip

- 直接修改配置文件来指定IP，并可以连接到外网：==**`vim /etc/sysconfig/network-scripts/ifcfg-ens33`**==

```shell
把BOOTPROTO改成静态：BOOTPROTO=static
#IP地址
IPADDR=192.168.200.130
#网关
GATEWAY=192.168.200.2
#域名解析器
DNS1=192.168.200.2
```

- 重启网络服务：`service network restart` 或者重启系统 `reboot`  生效

###  ifcfg-ens33文件说明

1. `DEVICE =eth0`	#接口名（设备，网卡）
2. `HWADDR=00:0C:2X:6X:0X:XX`	#MAC地址
3. `TYPE = Ethernet`	#网络类型(通常是 Ethemet）
4. UUID：随机id
5. `ONBOOT= yes`	#系统启动的时候网络接口是否有效(yes/no)
6. `BOOTPROTO = static`	IP 的配置方法( none ，static，bootp，dhcp ）(引导时不使用协议，静态分配 IP，BOOTP 协议，DHCP 协议)
7. `IPADDR=192.168.200.130`   #IP地址
8. `GATEWAY=192.168.200.2`  #网关
9. `DNS1=192.168.200.2`  #域名解析器

>方法二：自动获取

- 说明：登陆后，通过界面的来设置自动获取ip
- 特点：linux启动后会自动获取IP，缺点是每次自动获取的ip地址可能不一样
- 工作环境中，必须将IP地址固定

##  设置主机名和hosts映射

1. 查看主机名：`hostname`
   - 也可以修改主机名：`vim /etc/hostname`  修改后，重启生效

2. 设置hosts映射

- 思考：如何通过主机名能够找到（比如ping）某个Linux系统？
- **Windows**： 在`C:\Windows\System32\drivers\etc\hosts`  文件指定即可
  - 192.168.200.130 junhrCentos7
- **Linux**：在 `vim /etc/hosts`文件 指定：

##  主机名和解析过程

###  Host是什么

- 一个文本文件，用来记录IP和Hostname（主机名）的映射关系

###  DNS

- 域名系统（Domain Name System），是互联网上作为域名和IP地址相互映射的一个**分布式数据库**
- 如：网络和Internet设置=》更改适配器选项=》使用下面的DNS服务器地址：
  - 首选DNS服务器：202.100.222.34，当我们访问www.baidu.com的时候，如果我们在本地查不到百度的ip地址的时候，就会去这个DNS地址去查找。

###  当用户在浏览器中输入www.baidu.com时

1. 浏览器先检查浏览器中有没有该域名解析IP地址，有就先调用这个IP完成解析；如果没有，就检查DNS解析器缓存，如果有直接返回IP完成解析。这两个缓存，可以理解为本地解析器缓存
2. 一般电脑第一次访问网站后，在一定时间内，浏览器或操作系统会缓存它的IP地址（DNS解析记录），如在cmd中输入 `ipconfig /displaydns`（DNS域名解析缓存）`ipconfig /flushdns`（手动清理dns缓存）
3. 如果本地解析器缓存没有找到对应映射，检查系统中hosts文件中有没有配置对应的域名IP映射，如果有，完成解析并换回
4. 如果本地DNS解析器缓存和hosts文件中均没有找到对应的IP，则到域名服务DNS进行解析域

- 本地=》根=》顶级=》权限域名服务器

![dns](D:\typoraDoc\4-JavaEE\\imgs\dns.png)

<img src="imgs\dns.png" align='left' />





#  :pencil:进程管理

[进程管理，显示执行的进程，终止进程，查看进程树](https://blog.csdn.net/Mr_GYF/article/details/113704875)

- `top` 查看进程，`q` 退出

##  显示系统执行的进程 `ps`

- `top` 命令动态显示系统进程。
- 进程识别号：`PID`	终端机号：`TTY` 此进程所消耗CPU时间：`TIME`  正在执行的命令或进程名：`CMD`

1. 【daemon 守护进程】
1. 查询当前控制台上运行的进程：**ps**
1. 显示当前终端的所有进程信息：**`ps -a`**
2. 以用户的格式显示进程信息：**`ps -u`**
3. 显示后台进程运行参数：**`ps -x`**

>`ps` 详解：(process status)

- 分页显示进程信息：`ps -aux | more`  
  - ==`ps -aux`== 查看系统中所有运行的进程，包括后台进程【参数a 是所有进程，参数 x 包括不占用控制台的进程，参数u 显示用户】
  - `ps  -ef`  查看系统中所有运行的进程，包括后台进程。【-ef 查询所有进程，并显示**父进程**的进程号】

- 查看是否有sshd服务： `ps -aux | grep sshd`

1. System V 展示风格
2. %MEN：进程占用物理内存的百分比
3. VSZ：进程占用虚拟内存大小（单位：kb）
4. RSS：进程占用的物理内存大小（单位：kb）
5. TT：终端名称，缩写
6. STAT：进程状态，S-睡眠。s-表示进程是会话先导进程，N-表示进程拥有比普通优先级更低的优先级，R-正在运行，D-短期等待，Z-僵死进程，T-被跟踪或被停止
7. STARTED：进程的启动时间
8. TIME：CPU时间，即进程使用CPU的总时间
9. COMMAND：启动进程所用的命令和参数，如果时间过长会被截断显示

<img src="D:\typoraDoc\4-JavaEE\imgs\psaux.png" align="left"/>

##  以全格式显示当前所有进程，查看进程的父进程

- 以全格式显示当前所有的进程：**`ps -ef`**
  - 显示所有进程：选项：`e`
  - 全格式：`f`
- **`ps-ef|grep xxx`**  如： `ps -ef | grep sshd`  该指令是BSD风格
  - PPID：父进程IPD
  - C：CPU用于计算执行优先级的因子，数值越大，表明进程是CPU密集型运算，执行优先级会降低；数值越小，表明进程是I/O密集型运算，执行优先级会提高
  - STIME：进程启动时间
  - TIME：CPU时间
  - CMD：启动进程所用的命令和参数

<img src="imgs\psef.png" align="left" />

##  强制终止一个终端

<img src="imgs\killbash.png" align="left"/>

##  查看进程树 `pstree`

1. 以树状形式显示进程的pid ：`pstree -p`
2. 以树状的形式进程的用户id：`pstree -u`

#  :pencil:Service服务管理

[服务管理，监控网络状态，防火墙开启端口与关闭端口，service指令，服务的自启动和关闭，systemctl指令，查看服务](https://blog.csdn.net/Mr_GYF/article/details/113730210)

1. service指令管理的服务在 **`/etc/init.d`** 查看：`ls -l /etc/init.d`
   1. **方式二**：使用 **`setup`**  -> 系统服务就可以查看到全部 【带有 * 的服务，Linux启动时自行启动】
2. service 服务名 [ start | stop | restart | reload | status ]：如：`service network status`
   1. 在Linux图形化界面：输入`service network stop`后，XShell7就连接不上了。
3. 在centos7.0之后，很多服务不再使用service。而是使用 **`systemctl`**

#  :pencil:systemctl指令

##  Centos7后运行级别说明

1. 查看当前运行级别：`systemctl get-default`
2. 设置当前运行级别：`systemctl set-default multi-user.target` ：设置级别3
   1. **`systemctl set-default graphical.target`** ：设置级别5

#  :pencil:firewall指令

1. 打开端口：firewall-cmd--permanent--add-port = 端口号/协议
2. 关闭端口：firewall-cmd--pernament--remove-port=端口号/协议
3. 重新载入，才能生效：firewall-cmd--reload
4. 查询端口是否开放：firewall-cmd--query-port=端口/协议

# :pencil:监控网络状态

<img src="imgs\netstatanp.png" align="left" />

#  :pencil:RPM(RedHat Package Manager)

[RPM和YUM的介绍和使用，两种方式下载软件](https://blog.csdn.net/Mr_GYF/article/details/113755026)

1. 常用参数：
   - `i`：安装应用程序 (install)  `rpm  -ivh  xxx.rpm`
   - `e` 卸载应用程序 (erase)  `rpm  -evh  xxx.rpm`
   - `vh`  显示安装进度 （verbose hash）
   - `U`  升级软件包（update）`rpm -Uvh  xxx.rpm`
   - `qa` 显示所有已安装的软件包 （query all） `rpm  -qa`

- 查询已安装的rpm列表 ：**`rpm -qa|grep xx`**   [e=erase]
  - 例如：查询系统是否安装火狐浏览器 `rpm -qa|grep firefox`

#  :pencil:安装jdk/tomcat/mysql/IDEA

[提供资源、在Linux完成jdk的安装，tomcat安装，mysql安装，安装IDEA](https://blog.csdn.net/Mr_GYF/article/details/113766245)

##  JDK

1. `mkdir /opt/jdk`
2. 通过xfp7上传到/opt/jdk下
3. `cd/opt/jdk`
4. 解压：`tar -zxvf jdk-8u`
5. `mkdir /usr/local/java`
6. `mv /opt/jdk/jdk1.8.0 /usr/local/java/`
   1. echo指令输出环境变量，比如输出 $PATH $HOSTNAME `echo $PATH` `echo $HOSTNAME`
7. 配置环境变量的配置文件：`vim /etc/profile`
8. `export JAVA_HOME=/usr/local/java/jdk1.8.0_161`
9. `export PATH =$ JAVA_HOME/bin:$PATH`
10. `source /etc/profile` [让文件生效]

```shell
[root@junhrCentos7 ~]# cd /opt/jdk
[root@junhrCentos7 jdk]# ls
jdk-8u161-linux-x64.tar.gz
[root@junhrCentos7 jdk]# tar zxvf jdk-8u161-linux-x64.tar.gz 
[root@junhrCentos7 jdk]# ll
总用量 185316
drwxr-xr-x. 8   10  143      4096 12月 20 2017 jdk1.8.0_161
-rw-r--r--. 1 root root 189756259 11月 17 16:04 jdk-8u161-linux-x64.tar.gz
[root@junhrCentos7 jdk]# mkdir /usr/local/java
[root@junhrCentos7 jdk]# mv jdk1.8.0_161/ /usr/local/java
[root@junhrCentos7 jdk]# ll
总用量 185312
-rw-r--r--. 1 root root 189756259 11月 17 16:04 jdk-8u161-linux-x64.tar.gz

```

##  Tomcat

1. 上传安装文件，并解压缩到`/opt/tomcat`
2.  进入解压目录/bin,启动tomcat，`./startup.sh`
3. 如果不开放8080，在Windows浏览器地址栏输入：192.168.200.130:8080 发现访问不到
   1. 开放端口8080：`firewall-cmd --permanent --add-port=8080/tcp`
   2. ` firewall-cmd --reload`
   3. ` firewall-cmd --permanent --query-port=8080/tcp`
4. 修改完后，再在浏览器中访问192.168.200.130:8080 可以访问得到。

###  jps

shutdown.bat shutdown.sh（Linux环境下的关闭程序）查看tomcat是否启动：ps aux|grep tomcat，②jps 如果tomcat启动，会有Bootstrap显示③切换到tomcat安装目录的logs目录里面，cd logs/输入ls：会有catalina.out日志记录 。实时监控tomcat：**tail -f catalina.out**

```shell
[root@192 bin]# sh startup.sh
Using CATALINA_BASE:   /usr/tomcat
Using CATALINA_HOME:   /usr/tomcat
Using CATALINA_TMPDIR: /usr/tomcat/temp
Using JRE_HOME:        /usr/java/jdk/jdk1.8.0_161
Using CLASSPATH:       /usr/tomcat/bin/bootstrap.jar:/usr/tomcat/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
[root@192 bin]# jps
1332 Jps
1304 Bootstrap
[root@192 bin]# kill 1304
[root@192 bin]# jps
1346 Jps
```

##  IDEA

1. `mkdir /opt/idea`
2. 使用xftp7上传idea到/opt/idea
3. `tar zxvf ideaIU-2021.2.3.tar.gz `
4. 运行该软件脚本：`./idea.sh`

<img src="imgs\idea.png" align="left" />

##  MySQL

1. `mkdir /opt/mysql`
2. 通过xftp上传（也可以使用wget指令）：`wget http://dev.mysql.com/get/mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar`
3. 解压：`tar -xvf mysql-5.7.26-1.el7.x86_64.rpm-bundle.tar\`
4. 注意：删除mariadb
   1. 因centos7自带类的mysql数据库是mariadb，会和mysql冲突，需要先删除
   2. 运行 **`rpm -qa|grep mariadb`**,查询相关的mariadb相关安装包
   3. 使用指令： **`rpm -e --nodeps mariadb-libs`** 进行卸载
5. 依次按照顺序安装mysql
   1. rpm -ivh mysql-community-common-5.7.26-1.el7.x86_64.rpm
   2. rpm -ivh mysql-community-libs-5.7.26-1.el7.x86_64.rpm
   3. rpm -ivh mysql-community-client-5.7.26-1.el7.x86_64.rpm
   4. rpm -ivh mysql-community-server-5.7.26-1.el7.x86_64.rpm
6. 运行mysql服务：`systemctl start mysqld.service`
7. 给root用户设置密码：
   1. 运行 **`grep "temporary password" /var/log/mysqld.log`** 可以查看当前密码：3Pg4odzZKu-D
   2. 登录mysql
   3. 设置密码策略：`set global validate_password_policy=0;`
      1. 给root用户设置新密码：`set password for 'root'@'localhost' =password('新密码');`
      2. 随后刷新：` flush privileges;`
   4. 重新登录

<img src="imgs\yum-mysql.png" align="left" />

- 不需要提醒：sudo yum install -y mysql-community-server
  - sudo表示超级管理员，普通用户需要加。root用户不需要加sudo

###   MySQL主从复制

<img src="imgs\master-slave.png" align="left" />

###   读写分离

<img src="imgs\cluster.png" align="left" />

#  :pencil:Shell编程

##  快速入门

- 脚本格式要求：

  - 脚本以  `#!/bin/bash` 开头：`vim hello.sh`  `#!/bin/bash`  `echo "hello,world!"` 
  - 脚本需要有可执行权限

- 编写一个Shell脚本

- 脚本的常用执行方式：

  - 方式1：（输入脚本的绝对路径或者相对路径）
    - 赋予hello.sh 脚本的`x` 权限，再执行脚本

  - 方式2：`sh + 脚本`
    - 不用赋予可执行权限，直接执行

```shell
[root@192 ~]# mkdir /root/shcode
[root@192 ~]# cd /root/shcode
[root@192 shcode]# vim hello.sh
[root@192 shcode]# ll
总用量 4
-rw-r--r--. 1 root root 32 11月 19 13:18 hello.sh
[root@192 shcode]# ./hello.sh
-bash: ./hello.sh: 权限不够
[root@192 shcode]# chmod u+x hello.sh
[root@192 shcode]# ll
总用量 4
-rwxr--r--. 1 root root 32 11月 19 13:18 hello.sh
[root@192 shcode]# ./hello.sh
Hello World!
方式2：sh + 脚本
[root@192 shcode]# chmod u-x hello.sh
[root@192 shcode]# sh hello.sh
Hello World!
```

使用绝对路径（方式1，有执行权限）：

```shell
[root@192 shcode]# hello.sh
-bash: hello.sh: 未找到命令
[root@192 shcode]# /root/shcode/hello.sh
Hello World!
```

##  Shell的变量

###  将命令的返回值赋给变量

- A=`date`反引号，运行里面的命令，并把结果返回给变量A
- A=$(date) 等价于 反引号

```shell
#!/bin/bash
#案例1：定义变量A
A=100
#输出变量需要加上$
echo A=$A
echo "A=$A"
#案例2：撤销变量B
B=200
unset B
#案例3：声明静态的变量C=2，不能unset
readonly C=2
echo "C=$C"
#静态变量C，unset会报错
unset C
#将指令返回的结果赋值给变量
D=`date`
E=$(date)
echo "D=$D"
echo "E=$E"
#使用环境变量TOMCAT_HOME,需要先配置环境变量
echo "tomcat_home=$TOMCAT_HOME"
```

###  设置环境变量

1. 在`/etc/profile`  文件中定义 TOMCAT_HOME  环境变量
2. 查看环境变量 TOMCAT_HOME 的值（在输出环境变量前，需要让其生效 `source/etc/profile`）
3. 在另外一个 shell 程序中使用 TOMCAT_HOME 

```shell
[root@192 /]# vim /etc/profile
#定义一个环境变量
export TOMCAT_HOME=/usr/tomcat
[root@192 usr]# source /etc/profile
[root@192 usr]# echo $TOMCAT_HOME
```

>shell编程多行注释：

```shell
:<<!
内容
!
```

###  位置参数变量

>为了获取到 命令行的参数信息

1. `$n` ：n为数字，`$0` 代表命令本身，10以上的参数需要用大括号包含，如：`${11}`  代表第11个参数
2. `$*` ：这个变量代表命令行中所有的参数，`$*`  把所有的参数看成是一个整体
   1. `$@` ：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待
3. `$#`  这个变量代表命令行中所有参数的个数
4. 案例：编写一个shell脚本，在脚本中获取到命令行的各个参数信息

```shell
[root@192 shcode]# vim position.sh
#!/bin/bash
echo "$0 第一个参数信息： $1 第二个参数信息： $2"
echo "所有的参数信息：$*"
echo "$@"
echo "参数的个数：$#"

[root@192 shcode]# sh position.sh 100 200
position.sh 第一个参数信息： 100 第二个参数信息： 200
所有的参数信息：100 200
100 200
参数的个数：2
```

###  预定义变量

>shell设计者事先定义好的变量，可以直接在shell脚本中使用

1. 
