<center><h1><a href = "https://man.linuxde.net/whereis">Linux</a></h1></center>

[TOC]

## 文件处理

### 解压文件

```shell
tar -zxvf source_file -C target_file 
```



### 搜索文件位置

> 1. locate : 从文件索引数据库（系统自动维护，每天更新一次）中查找文件，查找速度快
> 2. which / whereis : 前者查找已经安装好的命令，后者查找文件索引数据库中的内容。
> 3. find : 用于查找指定目录的文件，可以设置时间等参数。
>    例如：
>    `find /etc -name "ifg*"` 在/etc目录下查找以ifg开头的文件

### 配置环境变量
> 1. `vim ~/.bashrc`
> 2. `source ~/.bashrc`










### 文件软链接

`ln -s [源地址] [目的地址]`



### ext文件系统

> ext4文件系统会把分区主要分为两大部分（暂时不提超级块）：一小部分用于保存文件的inode（i节点）信息；剩余的大部分用于保存block信息。
>
> 1. inode的默认大小为128 Byte，用来记录文件的权限（r、w、x）、文件的所有者和属组、文件的大小、文件的状态改变时间（ctime）、文件的最近一次读取时间（atime）、文件的最近一次修改时间（mtime）、文件的数据真正保存的block编号。每个文件需要占用一个inode。大家如果仔细查看，就会发现inode中是不记录文件名的，那是因为文件名记录在文件所在目录的block中。
> 2. block的大小可以是1KB、2KB、4KB，默认为4KB。block用于实际的数据存储，如果一个block放不下数据，则可以占用多个block。例如，有一个10KB的文件需要存储，则会占用3个block，虽然最后一个block不能占满，但也不能再放入其他文件的数据。这3个block有可能是连续的，也有可能是分散的。

![image](0D8CCFFB15154DEC81A35E6B8A03A50B)

### 重定向
`2>/dev/null`  
意思就是把错误输出到“黑洞”

`>/dev/null 2>&1`  
默认情况是1，也就是等同于1>/dev/null 2>&1。意思就是把标准输出重定向到“黑洞”，还把错误输出2重定向到标准输出1，也就是标准输出和错误输出都进了“黑洞”

`2>&1 >/dev/null`  
意思就是把错误输出2重定向到标准出书1，也就是屏幕，标准输出进了“黑洞”，也就是标准输出进了黑洞，错误输出打印到屏幕

### 设备文件

1. 硬盘设备
   `sda` 表示第一块硬盘

   `sdb` 表示第二块硬盘

   主分区从1开始，逻辑分区从5开始
   
2. 



### 文本处理

#### awk

- 格式：
  `awk 动作 文件名`
  `awk '{print $0}' demo.txt`

- 特殊变量：
  `$0`表示当前行
  `$1`表示第一行(空格为默认分隔符)
  `$NF`表示最后一行
  `$NR`表示当前行

- 特殊用法
	```shell
  #输出以空格为分隔符且第一个字符为 root 或者 bin 的行
	awk '$1 == "root" || $1 == "bin" {print $0} demo.txt
  
  #输出以空格为分隔符的奇数行的第一列的数据
  awk -F ':' 'NR % 2 == 1 {print $1}' demo.txt
  
  #以 : 为分隔符，当第一列的字典序大于等于 m 时输出该列内容，否则输出 --- 
  awk -F ':' '{if ($1 > "m") print $0; else print "---"}' demo.txt
  ```
  
#### sed
- 格式
  `sed [-hnV][-e<script>][-f<script文件>][文本文件]`

- 参数
  - a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
  - c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
  - d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
  - i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
  - p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
  - s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！

- 例子
  ```shell
  # 在testfile文件的第四行后添加一行，并将结果输出到标准输出
  sed -e '4a\newLine' testfile
  ```


#### cut

- 格式
  `cut OPTION... [FILE]...`

- 参数
  -f : 提取指定的字段，cut 命令使用 Tab 作为默认的分隔符。
  -d : Tab 是默认的分隔符，使用这一选项可以指定自己的分隔符。
  -b : 提取指定的字节，也可以指定一个范围。
  -c : 提取指定的字符，可以是以逗号分隔的数字的列表，也可以是以连字符分隔的数字的范围。
  –complement : 补充选中的部分，即反选。
  –output-delimiter : 修改输出时使用的分隔符。
  --only-delimited : 不输出不包含分隔符的列。

- 例子
  ```shell
  #输出以 : 为分隔符，第 1,2,3,4,6,7行
  grep "/bin/bash" /etc/passwd | cut -d':' -f1-4,6,7
  ```


#### vim

**文本处理**

1. 替换
   `i,j s /word1/word2/gc` 在 i行 和 j行 之间用word2替换word1，s替换命令，g对范围内所有匹配字符串起作用(否则只)，c表示每次替换前询问



**命令模式**

1. 水平分割的shell
   `:vert term` 使用 `etit` 加 `:q`退出
2. 创建标签页
   `:tabedit [filename]` 或者  `:e filename` 给定文件名，使用 `:tabn` 和 `:tabp` 或者`number + gt`切换标签
3. 将文件恢复到最开始的状态
   `:e!`
4. 将文件另存为
   `:w newfilename`
5. 将其他文件插入到现在操作的文件的末尾
   `:r filename`
6. 将进程挂起
   `ctrl + v` 或者 `ctrl + c`，`jobs` 查看挂起进程，`fg 1`把一号进程放到前台执行，`bg 1` 把一号进程放到后台执行
7. 录制操作
   - `qa` 启动录制
   - 输入操作
   - `q` 结束录制
   - `10@a` 将操作执行10次
8. 

## 重要文件

|                 文件作用                 |                 保存路径                  | 备注 |
| :--------------------------------------: | :---------------------------------------: | :--: |
|             网络服务配置文件             | /etc/sysconfig/network-scripts/ifcfg-eth0 |      |
|           主机名和IP的映射文件           |                /etc/hosts                 |      |
|             主机名称配置文件             |          /etc/sysconfig/network           |      |
| 配置系统变量 或者 环境变量 或者 别名信息 |               /etc/profile                |      |
|                                          |                                           |      |



## 挂起进程的处理

`jobs` 查看所有挂起的进程

`kill %1` 杀死挂起的一号进程

`jobs -l` 可以查看到进程号，然后用 `kill 进程号` 的方式杀死进程

`fg 1`把一号进程放到前台执行，`bg 1` 把一号进程放到后台执行

## 防火墙

1. 查看防火墙状态 `systemctl status firewalld.service`
2. 关闭防火墙 `systemctl stop firewalld.service`
3. 永久关闭防火墙 `systemctl disable firewalld.service`

## 好用的命令

### 查看历史命令

`history` 显示最近1000条命令

1. 可以使用上、下箭头调用以前的历史命令
2. 使用“!n”重复执行第n条历史命令
3. 使用“!!”重复执行上一条命令
4. 使用“!字串”重复执行最后一条以该字串开头的命令



## 权限管理

### 修改用户权限

> 用户身份
>
> - u 表示“用户（user）”，即文件或目录的所有者。
>
> - g 表示“同组（group）用户”，即与文件属主有相同组ID的所有用户。
> - o 表示“其他（others）用户”。
> - a 表示“所有（all）用户”。它是系统默认值。
>
>
> 赋予权限
>
> - ‘+’ 赋予权限
> - ‘-’ 剥夺权限
> - '=' 设置权限
>
> 权限
>
> - r 读权限(read)
> - w 写权限(write)
> - x 执行权限(execute)



> 1. chmod : 修改文件的权限模式
> 2. chgrp : 修改文件和目录的所属组

```
sudo chown -R [用户名] [路径]
chmod ug+w，o-x text
```

### 创建用户

1. `useradd -m <UserName> -s /bin/bash` 创建用户并设置用户目录和默认shell
2. `passwd <UserName>` 设置用户密码
3. `su <UserName>` 切换用户





## 机器管理

### 关闭错误的下载进程

```shell
snap changes  
sudo snap abort 5
```

### 关闭和重启

```shell
shutdown
shutdown -h 10 'notice'	#10分钟后关机,ctrl + c取消
shutdown -h 1:00		#1:00关机
shutdown -h now			#关机
shutdown -r				#重启
shutdown -c				#取消关机

reboot = shutdown -r now
half=shut -h now
poweroff=halt+关电
```



## 网络

### 监听端口

> 监听9870端口是否被占用
```shell
netstat -antup | grep 9870
```



## 软件管理

### 更改镜像

1. `vim /etc/apt/sources.list` 进入配置文件

2. ```markdown
   # 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse
   deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
   # deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse
   ```

3. `apt-get update` 更新配置



## 大数据

### HDFS

**首次进入时初始化工作目录**

`hdfs dfs -mkdir -p /user/hadoop`



**操作文件系统**

```shell
hdfs dfs -[命令]

hadoop fs -[命令]
```



**创建文件和文件夹****

```shell
//在默认路径（家目录）创建一个input文件
hdfs dfs -mkdir input

//在默认路径（家目录）创建一个文件夹
hdfs dfs -mkdir /input

//在根目录下创建一个文件夹
hdfs dfs -mkdir 
```



编写Jar包并运行

```shell
 hadoop jar  ./HadoopTest02_1-1.0-SNAPSHOT.jar
 
 com.MaxTemperature  /user/hadoop/input/sample.txt
 
 /user/hadoop/output
```





## Shell脚本

### 变量

#### 定义变量

```shell
x=100

y=x

declare - z=100

read -p "Please input an integer number: " number


A=`ls` 反引号,执行里面的命令
A=$(ls) 等价于反引号

```



#### 变量的引用

```shell
$x

${x}

$1 $2 #引用自己传入的参数

$((2*x+1))	#结果为2*x+1

${!y}


```



#### 变量替换

```shell
${var:-value}

${var:=value}

${var:+value}

${var:?value}
```



#### 算术运算

1. let命令

```shell
x=100

let x++

echo x		//101
```



2. pxpr

```shell
x=100
y=100

`pxpr $x + $y`	#用空格隔开
`pxpr x++`

echo x		//101
```

3. $(())

`$((12 + 32))`

4. $[]

`$[ 1 + 1 ]`



#### 特殊参数

```shell
$n # $1 代表命令本身、$1-$9 代表第1到9个参数，10以上参数用花括号，如 ${10}。
$* # 命令行中所有参数，且把所有参数看成一个整体。
$@ # 命令行中所有参数，且把每个参数区分对待。
$# # 所有参数个数。
$$ #当前进程的 PID 进程号。
$! #后台运行的最后一个进程的 PID 进程号。
$? #上一次命令的退出状态值
```



### 分支

#### 判断条件



#### 判断语句

```shell
test-e /tmp/file.sh   #判断文件是否存在
$? #取得判断的值


num=1
num1=1

if [ $num -eq $num1 ]; then
	echo "equals"
else
	echo "not equals"
fi


read -p "input an integer number: " num
read -p "input an integer number: " num1
if [ $num -gt $num1 ]; then
	echo "great than"
elif [ $num -lt $num1 ]; then
	echo "less than"
else
	echo "equals"
fi


read -p "input an integer number: " num
case $num in
1) echo "equals 1";;
2) echo "equals 2";;
3) echo "equals 3";;
*) echo "unknown";;
esac
```



### 循环

```shell
#循环 (contunue 2 表示跳出的嵌套数)
for((i=1;i<10;i++))
do
	echo "Hello world"
done

for num in {1..10}
do
	echo "Hello world"
done


for i in `seq 1 2 100` #输出1~100之间所有奇数
do
	echo $i
done


strings="s1 s2 s3 s4"
for str in $strings
do
	echo $str
done

read -p "input a integer: " num
while [[ $num -gt 0 ]]
do
	echo "num great than 0"
	let "num--"
done

cat student.txt | while read line	#按行输出文件
do
	echo line
done

for((i=1;i<10;i++))	#九九乘法表
do
	for((j=1;j<=$i;j++))
	do
		let "mult=$i*$j"
		echo -n "$i*$j=$mult " //在一行输出
	done
	echo
done
```



#### 函数

```shell
#函数
function func(){
	echo "Hello world"
}
func	#调用函数

function add(){
	echo `expr $1 + $2`
}
add $1 $2	#传入参数
./script.sh 1 2
```



## 备注

### 命令行快捷键

1. 最常用快捷键：
   `tab`  补全命令，每补全一个字符至少两次。

2. 移动光标快捷键：
   `Ctrl+A`  光标回到命令行首。
   `Ctrl+E`  光标回到命令行末。
   `Ctrl+F`  光标向右移动一个字符。
   `Ctrl+B`  光标向左移动一个字符。

3. 复制快捷键：
   `Ctrl+Insert`  复制命令（选中字符进行复制）

4. 粘贴快捷键：
   `Ctrl+Insert`  粘贴命令 

5. 剪切命令：
   `Ctrl+K` 剪切光标处到行尾处的字符。（有删除的作用）
   `Ctrl+U` 剪切光标处到行首处的字符。（有删除的作用）
   `Ctrl+W` 剪切光标前的一个单词（有删除的作用）
   `Ctrl+Y`  粘贴之前剪切/删除的文本

6. 中断命令：
   `Ctrl+C` 中断正在执行的任务命令或者删除整行。

7. 暂停命令：
   `Ctrl+Z`  暂停正在运行行中的任务。
   
8. 清除命令：
   `Ctrl+H`  删除光标前一个字符，等同于Backspace
   `Del`    删除光标后的一个字符。
   `Ctrl+L`  清除屏幕上所有内容，并开始新的一行，

9. 锁定、 解锁命令：
   `Ctrl+S`  锁定界面，使之无法输入内容。
   `Ctrl+Q`  解开`Ctrl+S`的锁定界面，进行输入

10. 重复使用命令：
    `Ctrl+D`  退出当前shell命令行，也可以直接关闭shell运行。
    `Ctrl+R`  搜索命令行中使用过的命令记录。
    `Ctrl+G`  从正在执行 `Ctrl+R` 的搜索中退出。

11. Esc相关命令：
    `Esc+.`  获取上一条命令的最后部分（空格分隔）
    `Esc+B`  移动到当前单词的开头。
    `Esc+F`  移动到当前单词的结尾。

### 命令对应的单词

**文件管理**：

> ls -- LiSt
>
> 
>
> cd -- Change Directory
>
> 
>
> pwd -- Print Working Directory
>
> 
>
> cp -- CoPy
>
> 
>
> mv -- MoVe
>
> 
>
> rm -- ReMove
>
> 
>
> pushd -- PUSH to Directory
>
> 
>
> popd -- POP from Directory
>
> 
>
> mkdir -- MaKe DIRectory
>
> 
>
> rmdir -- ReMove DIRectory
>
> 
>
> cat -- CATenate
>
> 
>
> sed -- Stream EDitor
>
> 
>
> diff -- DIFFerence
>
> 
>
> wc -- Word Count
>
> 
>
> chmod -- CHange MODe
>
> 
>
> chown -- CHange OWNer
>
> 
>
> chgrp -- CHange GRouP
>
> 
>
> awk -- Aho Weinberger andKernighan
>
> 
>
> gawk -- Gnu Aho Weinberger andKernighan
>
> 
>
> grep -- General RegularExpression Print
>
> 
>
> ln -- LiNk
>
> 
>
> tar -- TARball

 

**硬件管理**：

> df -- Disk Free
>
> 
>
> du -- Disk Usage
>
> 
>
> dd -- Data Description（一说是Convertand Copy，但是cc被用掉了，就用dd了）
>
> 
>
> parted -- PARTition EDitor
>
> 
>
> lspci -- LiSt PeripheralComponent Interconnect
>
> 
>
> lscpu -- LiSt Central ProcessUnit
>
> 
>
> lsusb -- LiSt Universal SerialBus
>
> 
>
> lsblk -- LiSt BLock
>
> 
>
> fdisk -- format DISK
>
> 
>
> mdadm -- Multiple Disk AndDevice M anager

 

 

**系统管理**：

> depmod -- DEPend MODule
>
> 
>
> lsmod -- LiSt MODule
>
> 
>
> modprobe -- MODule PROBE
>
> 
>
> modinfo -- MODule INFOrmation
>
> 
>
> insmod -- INSert MODule
>
> 
>
> rmmod -- ReMove MODule
>
> 
>
> mkfs -- Make FileSyatem
>
> 
>
> fsck -- File SystemConsistency Check
>
> 
>
> ps -- Processes Status
>
> 
>
> su -- Substitute User
>
> 
>
> bash -- Bourne Again SHell
>
> 
>
> init -- INITialization
>
> 
>
> ssh -- Secure SHell
>
> 
>
> wine -- Wine Is Not anEmulator
>
> 
>
> exec -- EXECute
>
> 
>
> fstab -- FileSystem TABle
>
> 
>
> passwd -- PASSWorD
>
> 
>
> chpasswd -- Change PASSWORD
>
> 
>
> pwconv -- PassWord CONVERT
>
> 
>
> pwunconv -- PassWord UNCONVERT
>
> 
>
> tty -- TeleTYpe
>
> 
>
> sudo -- SuperUser DO
>
> 
>
> grub -- GRand UnifiedBootloader
>
> 
>
> tzselect -- Time Zone SELECT
>
> 
>
> sync -- SYNChronize
>
> 
>
> systemd -- SYSTEM Daemon

 

**lvm**：

> lvm -- Logical Volume Manager
>
> 
>
> pvcreate -- Physical VolumeCREATE
>
> 
>
> vgcreate -- Volume GroupCREATE
>
> 
>
> lvcreate -- Logical VolumeCREATE
>
> 
>
> pvdisplay -- Physical VolumeDISPLAY
>
> 
>
> vgdisplay -- Volume Grou[DISPLAY
>
> 
>
> lvdisplay -- Logical VolumeDISPLAY
>
> 
>
> pvresize -- Physical VolumeRESIZE
>
> 
>
> vgresize -- Volume Group RESIZE
>
> 
>
> lvresize -- Logical VolumeRESIZE
>
> 
>
> pvextend -- Physical VolumeEXTEND
>
> 
>
> vgextend -- Volume GroupEXTEND
>
> 
>
> lvextend -- Logical VolumeEXTEND
>
> 
>
> pvremove -- Physical VolumeREMOVE
>
> 
>
> vgremove -- Volume GroupREMOVE
>
> 
>
> lvremove -- Logical VolumeREMOVE
>
> 
>
> pvs -- Physical Volume Status
>
> 
>
> vgs -- Volume Group Status
>
> 
>
> lvs -- Logical Volume Status