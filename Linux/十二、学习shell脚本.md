# 十二、学习shell脚本

shell脚本和Windows的批处理文件类似。通过shell脚本来简化日常的任务管理，而且Linux环境中，一些服务的启动也都是通过shell脚本完成的。

shell脚本是利用shell的功能所写的一个程序。这个程序是使用纯文本文件，将一些shell的语法与命令（含外部命令）写在里面，搭配正则表达式、管道命令与数据流重定向等功能，以达到我们所想要的处理目的。而且shell脚本更提供数组、循环、条件与逻辑判断等重要功能。

shell脚本用在系统管理上面是很好的一项工具，但是用在处理大量数值运算上，就不够好了，原因在于shell脚本的速度较慢，且使用的CPU资源较多，会造成主机资源的分配不良。

学习shell的原因大概有如下这些：

- 自动化管理的重要依据
- 跟踪与管理系统的重要工作
- 简单入侵检测功能
- 连续命令单一化
- 简易的数据处理
- 跨平台支持与学习历程较短

## shell脚本的规则与约束

编写shell脚本时应当注意一下事项：

- shell脚本内命令是从上而下、从左至右地分析与执行。特别是使用到函数时，需将函数定义在脚本的上部分，在这之后才能使用函数
- shell脚本内命令的执行如同普通命令一般，命令、选项与参数间的多个空格都会被忽略掉
- 空白行也将被忽略掉，并且[Tab]键产生的空白同样视为空格键
- 如果读取到一个Enter符号（CR），就尝试执行该行（该串）命令
- 至于如果一行的内容太多，则可以使用【\[Enter]】来扩展至下一行
- 【#】可作为注释，任何加在 # 后面的数据将全部被视为注释文字而被忽略
- 脚本第一行需要以【#!/bin/bash】来声明这个问卷内使用的是bash的语法

shell脚本文件的执行，此处举例以文件/home/bean/bin/hello.sh，执行有如下几种方法：

- 直接命令执行：hello.sh文件必须具有可读可执行（rx）的权限，然后：
  - 绝对路径执行：使用/home/bean/bin/hello.sh来执行命令
  - 相对路径执行：假设工作目录在/home/bean/，则使用./bin/hello.sh来执行
- 环境变量【PATH】功能：将hello.sh放在PATH指定的目录下，例如环境变量PATH中有路径~/bin，那么此时执行hello..sh即可
- 以bash程序来执行：通过【bash ./bin/hello.sh】或【sh ./bin/hello.sh】来执行
- 以source或【.】来执行

**以source或者【.】执行的是在父进程（当前进程）中执行，shell脚本里的变量什么的都会在当前bash留存下来；而其他方式执行的脚本是在子进程中执行的，执行完毕后在子进程的相关变量不会传递回父进程。**



## 第一个shell脚本，hello world

```shell
[root@localhost ~]# cd /home; mkdir shell
[root@localhost ~]# vim helloworld.sh
#!/bin/bash
echo -e "hello world! \a \n"
exit 0
[root@localhost ~]# chmod a+x helloworld.sh   # 赋予可执行权限
[root@localhost ~]# ./helloworld.sh
hello world!
```



## shell脚本的良好编写习惯

- 脚本的功能说明
- 脚本的版本信息
- 脚本的作者与联络方式
- 脚本的版权声明方式
- 脚本的历史记录
- 脚本内较特殊的命令，使用【绝对路径】的方式来执行
- 脚本运行时需要的环境变量预先声明与设置
- 脚本最好以[Tab]键缩进

## 简单的shell脚本练习

### 示例一

编写一个脚本让它可以读入用户输入：firstname和lastname，最后在屏幕上显示your full name is：的内容。

```shell
[root@localhost ~]# cd /home/shell; vim showname.sh
#!/bin/bash
PATH=${PATH}:/home/shell
export PATH
read -p "please input your first name:" firstname    # 提示使用者输入
read -p "please input your last name:" lastname
echo -e "your full name is: ${firstname} ${lastname}."
exit 0
[root@localhost ~]# chmod a+x showname.sh
[root@localhost ~]# showname.sh
please input your first name:bean   		#用户输入bean
please input your last name:cai			#用户输入cai
your full name is: bean cai.	# 程序输出结果

```



### 示例二

假设我想建立三个空文件（通过touch），文件名最开头由用户输入决定，文件名后缀为前天、昨天、今天的日期，比如文件前缀是filename，今天是2021-09-24，那么生成的三个文件名是：filename_20210922、filename_20210923、filename_20210924.

```shell
[root@localhost ~]# cd /home/shell; vim create_three_file.sh
#!/bin/bash
# create three file which maned by user's input and date command.
PATH=${PATH}:/home/shell
export PATH
echo -e "I will use 'touch' command to create 3 files."
read -p "Please input your filename:" fileuser
# 为了避免使用者随意按Enter，利用变量功能分析文件名是否有设置？
filename=${fileuser:-"filename"}
date1=$(date --date='2 days ago' +%Y%m%d)   # 前两天的日期
date2=$(date --date='1 days ago' +%Y%m%d)   # 前一天的日期
date3=$(date +%Y%m%d)   # 今天日期
touch "${filename}_${date1}"
touch "${filename}_${date2}"
touch "${filename}_${date3}"
exit 0
[root@localhost ~]# chmod a+x create_three_file.sh
[root@localhost ~]# ./create_three_file.sh
I will use 'touch' command to create 3 files.
Please input your filename:tempfile
[root@localhost ~]# ls
create_three_file.sh  hello-world.sh  showname.sh  tempfile_20210922  tempfile_20210923  tempfile_20210924
```



数值运算：简单的加减乘除可以利用【$((计算式))】来实现

### 示例三

让用户输入两个变量，然后将两个数值相乘并输出结果。

```shell
[root@localhost ~]# cd /home/shell; vim multiplying.sh
#!/bin/bash
# user input 2 integer numbers. program will cross these two numbers.
PATH=${PATH}:/home/shell
export PATH
echo -e "You SHOULD input 2 numbers, I will multiplying them!"
read -p "first number:" firstnumber
read -p "second number:" secondnumber
total=$((${firstnumber} * ${secondnumber}))
#declare -i total=$((${firstnumber} * ${secondnumber}))
echo "total:${total}"
exit 0
[root@localhost ~]# chmod a+x multiplying.sh
[root@localhost ~]# ./multipyling.sh
first number:5
second number:6
total:30
```



### 示例四

用户输入小数点位数，程序输出Pi

```shell
[root@localhost ~]# cd /home/shell; vim cal_pi.sh
#!/bin/bash
PATH=${PATH}:/home/shell
export PATH
echo -e "this program will calculate pi value"
echo -e "you will input a float number to calculate pi value"
read -p "the scale number(10~10000): " checking
num=${checking:-"10"}   # 开始判断是否有输入数值
echo -e "starting calculate pi value. be patient..."
time echo "scale=${num}; 4*a(1)" | bc -l
[root@localhost ~]# chmod a+x cal_pi.sh
[root@localhost ~]# ./cal_pi.sh
this program will calculate pi value
you will input a float number to calculate pi value
the scale number(10~10000): 5
starting calculate pi value. be patient...
3.14156

real	0m0.003s
user	0m0.000s
sys	0m0.004s
```



## 判断式

### test

```shell
[root@localhost ~]# test -e ./test  # 检查。.test是否存在
[root@localhost ~]# test -e ./test && echo "exist" || echo "not exist"
```

test还有很多参数如下表格所示：

1.关于某个文件的【文件类型】判断，如`test -e filename`表示是否存在

| 测试的参数 | 代表意义                                          |
| ---------- | ------------------------------------------------- |
| -e         | 该【文件名】是否存在（常用）                      |
| -f         | 该【文件名】是否为存在且为文件（file）（常用）    |
| -d         | 该【文件名】是否存在且为目录（directory）（常用） |
| -b         | 该【文件名】是否存在且为一个block device设备      |
| -c         | 该【文件名】是否存在且为一个character device设备  |
| -S         | 该【文件名】是否存在且为一个socket文件            |
| -p         | 该【文件名】是否存在且为一个FIFO（pipe）文件      |
| -L         | 该【文件名】是否存在且为一个链接文件              |

2.关于文件的权限检测，如`test -r filename`表示是否可读

| 测试的参数 | 代表意义                                       |
| ---------- | ---------------------------------------------- |
| -r         | 检测该文件名是否存在且具有【可读】的权限       |
| -w         | 检测该文件名是否存在且具有【可写】的权限       |
| -x         | 检测该文件名是否存在且具有【可执行】的权限     |
| -u         | 检测该文件名是否存在且具有【SUID】的属性       |
| -g         | 检测该文件名是否存在且具有【SGUI】的属性       |
| -k         | 检测该文件名是否存在且具有【Sticky bit】的属性 |
| -s         | 检测该文件名是否存在且为【非空文件】           |

3. 两个文件之间的比较，如`test file1-nt file2 `

| 测试的参数 | 代表意义                                                     |
| ---------- | ------------------------------------------------------------ |
| -nt        | （newer then）判断file1是否比file2新                         |
| -ot        | （older then）判断file1是否比file2旧                         |
| -ef        | 判断file1与file2是否为同一文件，可用在判断hard link的判定上。主要意义在判定，两个文件是否均指向同一个inode |

4.关于两个整数之间的比较，如`test n1 -eq n2`

| 测试的参数 | 代表意义                              |
| ---------- | ------------------------------------- |
| -eq        | 两数值相等（equal）                   |
| -ne        | 两数值不等（not equal）               |
| -gt        | n1大于n2（greater then）              |
| -lt        | n1小于n2（less then）                 |
| -ge        | n1大于等于n2（greater then or equal） |
| -le        | n1小于等于n2（less then or equal）    |

5.判定字符串的数据

| 测试的参数        | 代表意义                                           |
| ----------------- | -------------------------------------------------- |
| test -z string    | 判定字符串是否为0？若string为空字符串，则为true    |
| test -n string    | 判定字符串是否非为0？若string为空字符串，则为false |
| test str1 == str2 | 判定str1是否等于str2，若相等，则返回true           |
| test str1 != str2 | 判定str1是否不等于str2，若相等，则返回false        |

6.多重条件判定，如`test -r filename -a -x filename`

| 测试的参数 | 代表意义                                            |
| ---------- | --------------------------------------------------- |
| -a         | （and）两条件同时成立                               |
| -o         | （or）两条件任何一个成立                            |
| !          | 反向状态，如test ! -x file，当file不具有x时返回true |

### 利用判断符号[ ]

除了用test之外，还可以使用判断符号【[ ]】（就是中括号）来进行数据的判断，用法同test。需要注意的是使用中括号作为shell的判断式时，必须要注意**中括号的两端需要有空格符来分隔**，例如`[ "${HOME} == "${MAIL}" ]`需要有空格的地方有`[空格"${HOME}空格==空格"${MAIL}"空格]`。其他需要注意的是中括号内的变量都以双引号括号起来，假如name="bean cai"，然后判定条件：`[ ${name} == "bean cai" ]`会被解析成`[ bean cai == "bean cai"]`，此时bash就会报错：bash: [: too man arguments

 ### shell脚本的默认变量（$0、$1、$2...）

/path/shellscript.sh opt1 opt2 opt3
             $0                    $1     $2     $3

还有一些特殊的变量：

- $#：代表后接的参数个数，例如上面这个这里显示为3
- $@：代表【"$1" "$2" "$3"】的意思，每个变量是独立的（用双引号括起来）
- $*：代表【"$1c$2c$3"】，其中c为分隔符，默认为空格

shift：造成变量数量号码偏移（直接偏移没掉）。shift或者shift n，n为数字，代表偏移几个。



## 条件判断式

### if...then

语法格式为：

```shell
if [ 条件判断式 ]; then
	条件成立时的程序块
fi
```

```shell
if [ 条件判断式 ]; then
	程序块
else
	程序块
fi
```

```shell
if [ 条件判断式 ]; then
	程序块
elif [ 条件判断式 ]; then
	程序块
elif [ 条件判断式 ]; then
	程序块
else
	程序块
fi
```



### case...esac

语法格式为：

```shell
case $变量名或者${变量名} in
	"情况1内容")
		程序块
		;;
	"情况2内容")
		程序块
		;;
	*)
	;;
esac
```



## 函数功能

因为shell脚本的执行方式是由上而下、由左至右，因此在shell脚本单中的function的设置一定要在程序的最前面。

其语法格式如下：

```shell
function funcname() {
	程序段
	若函数需要接收参数，以$0（$0是函数名）、$1、$2这样的形式接收，声明方式的小括号中无需写任何东西；
	调用者应当funcname param1 param2
}
```



## 循环

### while do done、until do done（不定循环）

语法格式为：

```shell
while [ 条件判定式 ]            # 条件成立时，进入循环体，直到条件不成立才退出
do
	循环体程序块
done
```

```shell
until [ 条件判定式 ]            # 条件不成立时，进入循环体，直到条件成立才退出循环
do
	循环体程序块
done
```

 

### for...do...done（固定循环）

已知知道要进行几次循环

语法格式为：

```shell
for variable in value1 value2 value3 ...
do
	程序块
done
```

以上面的例子来说，总共执行三次循环
第一次循环时，$variable的值为value1
第二次循环时，$variable的值为value2
第三次循环时，$variable的值为value3
...

```shell
for var in $(seq 1 100)  # seq为sequence的意思，会产生1 2 3 4...100个数字
do
	程序块
done
```

```shell
for var in 1..100  # 会产生1 2 3 4...100个数字
do
	程序块
done
```

```shell
for var in a..g  # 会产生a b c d e f g
do
	程序块
done
```

```shell
[root@localhost ~]# echo {a..d}
a b c d
[root@localhost ~]# echo {1..10}
1 2 3 4 5 6 7 8 9 10
```



### for...do...done

语法格式为：

```shell
for (( 初始值; 限制值; 赋值运算(循环因子迭代); ))
do
	程序块
done
```

```shell
num=10;
for (( i=1; i<=${num}; i=i+1))
do
	echo ${i}
done
```



## shell脚本的跟踪与调试

```shell
[root@localhost ~]# sh [-nvx] script.sh
选项与参数：
-n：不要执行脚本，仅查询语法的问题
-v：在执行脚本前，先将脚本文件的内容输出到屏幕上
-x：将脚本执行过程的命令全部打印到屏幕上，这个对于排错很有用
[root@localhost shell]# cat hello-world.sh 
#!/bin/bash
echo -e "hello world! \a \n"
exit 0
[root@localhost shell]# sh -n hello-world.sh  # 没有语法错误时没有输出内容
[root@localhost shell]# sh -v hello-world.sh 
#!/bin/bash
echo -e "hello world! \a \n"
hello world!  

exit 0
[root@localhost shell]# sh -x hello-world.sh 
+ echo -e 'hello world! \a \n'       # 以【+】号来显示是执行了这一行命令
hello world!  

+ exit 0
[root@localhost shell]# 
```

