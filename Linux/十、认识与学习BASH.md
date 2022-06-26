# 十、认识与学习BASH

## 认识BASH这个shell

我们必须要通过shell将我们输入的命令与内核沟通，好让内核可以控制硬件来正确无误地工作。为什么要学习shell呢？有一点很重要的就是：命令行模式的传输速度一定比较快，而且叫不容哦那个一出现掉线或者是信息外流的问题。

shell有很多版本。早期的sh（Bourne shell）、Bash（Bourne Again Shell）是增强版的sh，商业常用的K shell，TCSH等。可以通过查看`/etc/shells`这个文件，显示系统有多少可以使用的shell。

```shell
[root@localhost ~]# cat /etc/shells
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

为什么我么系统上合法的shell要写入`/etc/shells`文件中呢？这是因为系统某些服务在运行过程中，会去检查用户能够使用的shells，而这些shell的查询就是借由`/etc/shells`这个文件。

`/etc/passwd`文件记录了用户何时可以使用哪种shell，其中每行的最后一列表示你登录后可以取得的默认的shell。

bash shell的功能：

- 历史命令
- 命令与文件补全功能
- 命令别名设置功能
- 任务挂历、前台、后台控制
- 程序化脚本
- 通配符（Wildcard）

查询命令是否为bash shell的内置命令：type

```shell
[root@localhost ~]# type [-tpa] name
选项与参数：
不加任何参数时，type会显示出那么是外部命令还是bash内置命令
-t：当加入-t是，type将name以下面这些字眼显示处他的含义：
	file：表示外部命令
	alias：表示该命令为命令别名的命令功能
	builtin：表示该命令为bash内置命令
-p：如果后面接的name是外部命令时，才会显示出完整的文件名
-a：会由PATH变量定义的路径中，将所有含name的命令都列出来，包含alias
[root@localhost ~]# type -t cd
builtin
```



命令的执行与快速编辑按钮

如果命令太长可利用【\Enter】来换行书写，也就是在未完成的命令末尾加 \ ,并紧接按回车换到下一行。

命令有一些常见的快速组合键：

| 组合键   | 功能与示范                     |
| -------- | ------------------------------ |
| [Ctrl]+u | 从光标处向前删除命令串         |
| [Ctrl]+k | 从光标处向后删除命令串         |
| [Ctrl]+a | 让光标移动到整个命令串的最前面 |
| [Ctrl]+e | 让光标移动到整个命令串的最后面 |



## shell的变量功能

### 变量的使用echo

使用变量只需要在变量名称前加$或`${变量}`即可访问变量，推荐使用`${变量}`。

```shell
[root@localhost ~]# echo $variable
[root@localhost ~]# echo ${variable} # 推荐使用这种
[root@localhost ~]# ehco ${PATH}  # 打印PATH环境变量
/usr/local/lib/nodejs/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/apache-maven-3.6.3/bin:/usr/local/jdk1.8.0_221/bin:/root/bin
```

### 变量的设置规则

- 变量与变量内容以一个等号【=】来连接
- 等号两边不能直接接空格
- 变量名称只能是英文字母与数字，且开头不能是数字
- 变量内容若有空格可使用双引号【"】或者单引号【'】将变量内容结合起来
- 双引号内的特殊字符如$等，可以保有原本的特性
- 单引号内的特殊字符仅为一般字符，即单引号内的内容是什么就是什么
- 可用转义符【\】将特殊字符（如[Enter]、$、\、空格、'等）变成一般字符，例如：`myname=bean\ c`
- 在一串命令的执行中，还需借由其他额外的命令所提供的信息时，可使用反引号\``命令`\`或$(命令)，例如：`version=$(uname -r)`
- 如哦该变量为扩增变量内容时，可使用`$变量名称`或`${变量名称}`累加内容，例如：`PATH=${PATH}:/home/bin`
- 若该变量需要在其他子程序执行，则需要以export来使变量变成环境变量，例如：`export PATH`
- 通常大写字符为系统默认变量，自行设置变量可以使用小写字符，方便判断
- 取消变量的方法为使用unset，`unset 变量名`



### 环境变量的功能

查看所有环境变量使用命令`env`

```shell
[root@localhost ~]# env
```

常用环境变量说明：

- HOME：代表用户的根目录。`cd ~`就可以直接回到用户的根目录
- SHELL：告知我们，目前这个环境使用的是哪个shell程序？默认使用/bin/bash
- HISTSIZE：与历史命令有关，允许被记录的条数
- MAIL：这个使用者的mailbox的位置
- PATH：就是执行文件查找的路径，目录与目录中间以冒号【:】分隔，由于文件的查找是依序由PATH变量内的目录来查询，所以目录的顺序也是很重要的
- LANG：语系。
- RANDOM：随机数的变量。大多数Linux发行版都会有随机数生成器，那就是/dev/random这个文件。我们可以通过该变量获取随机数，例如：`echo ${RANDOM}`，默认随机数范围为0~32767。万一想使用0~9的数值可利用declare声明数值类型，例如：`declare -i number=${RANDOM}*10/32768; echo ${number}`

set可以查看所有变量（含环境变量和自定义变量）。set除了环境变量之外，还会将其他在bash内的变量通通显示出来。其中重要的几个参数：

- PS1：命令提示符的设置，默认值为`[\u@\h \W]\$ `，其中有一些特殊符号的意义：
  - \d：可显示出【星期 月 日】的日期格式，如【Mon Feb 2】
  - \H：完整的主机名。举例来说，练习机器的为【localhost.localdomain】
  - \h：仅取主机名在第一个小数点之前的名字，如练习机器的为【localhost】，后面省略
  - \t：显示时间，为24小时格式的【HH:MM:SS】
  - \T：显示时间，为12小时格式的【HH:MM:SS】
  - \A：显示时间，为24小时格式的【HH:MM】
  - \@：显示时间，为12小时格式的【am/pm】样式
  - \u：目前用户的账号名称，如【root】
  - \v：bash的版本信息，如练习机器的版本为4.2.26（1）-release，进取【4.2】显示。bash命令行下利用组合键[Ctrl]+x -> [Ctrl]+v即可显示bash版本信息
  - \w：完整的工作目录名称，由根目录写起的目录名称，但根目录一般会以~替换
  - \W：利用basename函数取得工作目录名称，所以仅列出最后一个目录名
  - \#：执行的第几个命令
  - \$：提示符，如果是root时，提示符为#，否则就是$
- $：（关于本shell的PID），若我们想知道当前shall的进程号可以用命令`echo $$`来获取
- ?：（关于上一个命令的返回值代码），当上条命令成功返回0，否则返回错误代码，例如127

影响显示结果的语系变量（locale）

```shell
[root@localhost ~]# locale -a  # 查看系统支持的所有语系
[root@localhost ~]# locale  # 查看与语系相关的环境变量
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

其中重要的两个变量时LANG和LC_ALL，一般只要设置了这两个变量即可。



### 变量的有效范围

环境变量=全局变量；自定义变量=局部变量



### 变量键盘读取

read命令可用来获取从键盘输入的变量值

```shell
[root@localhost ~]# read [-pt] variable
选项与参数：
-p：后面可以接提示字符
-t：后面可以接等待的秒数，到达该秒数后即退出，不会一直等待用户输入
[root@localhost ~]# read atest
this is input  # 此时光标会等待你的输入
[root@localhost ~]# echo ${atest}
this is input
[root@localhost ~]# read -p "please keyin your name:" -t 30 named
please keyin your name:bean    # 此时会有提示字符【please keyin your name:】
[root@localhost ~]# echo ${named}
bean
```



### declare，typeset声明变量类型

```shell
[root@localhost ~]# declare [-aixr] variable
选项与参数：
-a：后面名为variable的变量定义为数组类型
-i：后面名为variable的变量定义为整数类型
-x：用法与export一样，将后面的variable变成环境变量
-r：将变量设置称readonly类型，该变量内容不可更改，也不能unset
[root@localhost ~]# sum=100+200+300 # 变量默认类型是字符串，所以直接当成字符串了
[root@localhost ~]# echo ${sum}
100+200+300
[root@localhost ~]# declare -i sum=100+200+300 # 定义整数变量
[root@localhost ~]# echo ${sum}
600
[root@localhost ~]# declare -x sum # 或者export sum 将sum变成环境变量
[root@localhost ~]# declare +x sum # 将sum变成非环境变量
[root@localhost ~]# declare -p sum #可以单独列出变量的类型
declare -i sum="600"

[root@localhost ~]# var[index]=content  # 数组的设置方式
[root@localhost ~]# myarr[1]="xiaoming"   
[root@localhost ~]# echo ${myarr[1]}
xiaoming
```



## 命令别名与历史命令

### 命令别名设置：alias、unalias

别名的设置很简单，类似设置变量，仅需在前面增加alias即可，例如`alias lm='ls -al | more'`，该命令为lm，当输入lm时即执行`ls -al | more`。取消别名用命令`unalias 别名`。例如`unalias lm`。



### 历史命令

```shell
[root@localhost ~]# history [-n]
[root@localhost ~]# history [-c]
[root@localhost ~]# history [-raw] [histfiles]
选项与参数：
不加参数时，显示所有近期执行的命令
-n：数据，列出最近的n条命令
-c：将目前的shell中的所有history内容全部清除
-a：将目前新增的history命令新增入histfiles中，若没有加histfiles，则默认写入~/.bash_history
-r：将histfiles的内容导入到目前这个shell的history记录中
-w：将目前history记录内容写入histfiles中
[root@localhost ~]# histroy   # 列出内存内所有的history记录
1017 man bash
1018 ll
1019 history
[root@localhost ~]# !number  # 执行第几条命令的意思
[root@localhost ~]# !command  # 由最近的命令向前查找【命令串开头为command】的哪个命令，并执行
[root@localhost ~]# !!执行上一条命令
```



## Bash shell的操作环境

### 命令运行的顺序

1. 以相对/绝对路径执行命令，例如【/bin/ls】或【./ls】
2. 由alias找到该命令来执行
3. 由bash内置的（builtin）命令来执行
4. 通过$PATH这个变量的顺序查找到的第一个命令来执行



### bash的登录与欢迎信息：/etc/issue、/etc/motd

登录信息存储在配置文件`/etc/issue`中，该文件默认内容为:

```shell
\S
Kernel \r on \m
```

该文件有一些特殊的符号，其含义如下：

| issue内的个代码意义                         |
| ------------------------------------------- |
| \d 本地端时间的日期                         |
| \l 显示第几个终端页面                       |
| \m 显示硬件的等级（i386/i486/i586/i686...） |
| \n 显示主机的网络名称                       |
| \O 显示domain name                          |
| \r 操作系统的版本（相当于uname -r）         |
| \t 显示本地端时间的时间                     |
| \S 操作系统的名称                           |
| \v 操作系统的版本                           |



### bash的环境配置文件

你是否会觉得奇怪，怎么我们什么操作都没有进行，但是进入bash就取得一堆有用的变量了呢？这是因为系统有一些环境配置文件的存在，让bash在启动的时候直接取读取这些配置文件，以规划好bash

的操作环境。而这些配置文件由可以划分为全局系统配置文件和用户个人偏好配置文件。前面谈到的命令别名、自定义变量，在你注销bash后就会失效，若要保留设置，就要将这些设置写入配置文件才行。

在介绍配置文件前先介绍一下login与non-login shell，因为这两种shell读取的配置文件并不一样。

- login shell：取得bash时需要完整的登录流程，就称为login shell
- non-login shell：取得bash的方法不需要重复登录的操作。举例来说，（1）你以X Wimdow登录Linux后，再以X的图形化接口启动终端，此时这个终端接口并没有需要再次的输入账号与密码，该bash的环境就称为non-login shell。（2）你在原本的bash环境下再次执行bash这个命令，同样的也没有输入账号和密码，那第二个bash（子进程）也是non-login shell。



#### login shell读取的配置文件

**login shell**会读取的配置文件有两个：
1.`/etc/profile`：这是系统整体的设置，最好不要修改这个文件。
2.`~/.bash_profile`或`~/.bash_login`或`~/.profile`：属于用户个人设置，你要添加自己的数据，就写在这里。

- `/etc/profile`

用户登陆后依次读取配置文件`/etc/profile`，该文件有一些重要的配置变量，同时该配置文件会去读取`/etc/profile.d/*.sh`目录下的配置文件。`/etc/profile`该文件设置的变量主要有：

- PATH：会根据UID决定PATH变量要不要含有sbin的系统命令目录
- MAIL：根据账号设置好用户的mailbox到`/var/spool/mail`
- USER：根据用户的账号设置此变量内容
- HOSTNAME：根据主机的hostname命令决定此变量内容
- HISTSIZE：历史命令历史条数，一般为1000
- umask：root默认为022，其他用户默认为002

读取的配置文件可能还会涉及`/etc/locale.conf`，该文件是被`/etc/profile.d/lang.sh`所调，该配置文件中重要的变量就是LANG和LC_ALL。

- ~/.bash_profile

bash在读取了整体环境设置的`/etc/profile`并借此调用其他配置后，接下来则会读取用户的个人配置文件。在bash环境中，所读取的个人偏好配置文件有三个，依序分别是`~/.bash_profile`、`~/.bash_login`、`~/.profile`。这三个文件时按顺序读的，若读到第一个则后面两个就不会读取，若第一个不存在，才会顺延读取第二个甚至第三个配置文件。

```shell
[root@localhost ~]# cat ~/.bash_profile 
# .bash_profile

# Get the aliases and functions
#下面这个if在判断是否存在~/.bashrc这个配置文件，若存在即将配置内容加载到shell环境中
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
```

通过读取配置文件`~/.bash_profile`可以知道该配置文件会调用配置文件`~/.bashrc`。所以实际上用户个人偏好的设置其实是设置在`~/.bashrc`这个文件中的。

由于配置文件`/etc/profile`和`~/.bash_profile`都是在取得login shell后才会读取的配置，若将个人偏好设置写入相关配置文件，此时是不会生效的。若想生效要么注销后再登录，要么就用source或小数点（.）命令：将配置文件的内容读到目前的shell环境中。

```shell
[root@localhost ~]# source 配置文件名
[root@localhost ~]# source ~/.bashrc  # 将配置文件~/.bashrc读到当前shell环境中
[root@localhost ~]# . ~/.bashrc  # 将配置文件~/.bashrc读到当前shell环境中
```

总结一下，login shell最终调用的配置文件依次为：`/etc/profile`、`/etc/profile.d/*.sh`、`/etc/locale.conf`、`~/.bash_profile`或`~/.bash_login`或`~/.profile`、`~/.bashrc`、`/etc/bashrc`。



#### non-login shell读取的配置文件

non-login这种非登录的情况取得bash后仅会读取配置文件`~/.bashrc`。看下该文件的默认内容：

```shell
[root@localhost ~]# cat ~/.bashrc
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
```

可见该文件除了文件自身一些变量之外，还会去读取系统全局的配置文件`/etc/bashrc`。为什么会去调`/etc/bashrc`呢？因为`/etc/bashrc`帮我们的bash定义了如下内容：

- 根据不同的UID设置umask值
- 根据不同的UID设置提示字符（就是PS1变量）
- 调用/etc/profile.d/*.sh的设置

总结一下,non-login shell最终调用的配置文件依次为：`~/.bashrc`、`/etc/bashrc`、`/etc/profile.d/*.sh`。



#### 其他相关配置

事实上还有一些其他配置文件可能会影响到你的bash操作的。

- `/etc/man_db.conf`

这个文件乍看之下好像跟bash没关系，但对于系统管理员来说，却也很重要。这个文件的内容规范了使用man的时候，man page的路径到哪里去寻找，所有简单点说就是这个文件规定了执行man的时候去哪里查看数据的问题。

那么什么时候需要来修改这个文件呢？如果你是以tarball的方式来安装你的软件，那么你的man page可能会放置在/usr/local/softpackage/man里面，这个softpackage是你的软件名称，这个时候你就得以手动的方式将该路径追加到/etc/man_db.conf里面，否则使用man的时候就会找不到相关的说明文件。

- `~/.bash_history`

默认情况下我们的历史命令就记录在这里。而这个文件能够记录几条数据，则与HISTSIZE这个变量有关，每次登录bash后，bash会先读取这个文件，将所有的历史命令读入内存，因此，当我们登录bash后就可以查知上次使用过哪些命令。

- `~/.bash_logout`

这个文件记录了**当注销bash后，系统再帮我做完什么操作后才离开**的意思。你可以去读取一下这个文件的内容，默认情况下，注销时，bash只是帮我们清掉屏幕信息而已。不过，你也可以将一些备份或是其他你认为重要的任务写入这个文件中（例如清空缓存），那么当你离开Linux时，就可以解决一些烦人的事情。

### bash默认组合键

| 组合按键 | 执行结果                                               |
| -------- | ------------------------------------------------------ |
| [Ctrl]+c | 终止当前名                                             |
| [Ctrl]+d | 输入结束（EOF）                                        |
| [Ctrl]+m | 就是回车                                               |
| [Ctrl]+s | 暂停屏幕的输出，若无意按到该组合键，可用[Ctrl]+q来恢复 |
| [Ctrl]+q | 恢复屏幕的输出                                         |
| [Ctrl]+u | 从光标处向前删除                                       |
| [Ctrl]+z | 暂停目前的命令                                         |



### 通配符与特殊符号

| 符号 | 意义                                                         |
| :--: | ------------------------------------------------------------ |
|  *   | 代表0到无穷个任意字符                                        |
|  ?   | 代表一个字符的占位符                                         |
|  []  | 代表括号内任意一个字符。例如[abcd]代表一定有a、b、c、d其中一个字符 |
| [-]  | 代表在编码顺序内的所有字符。例如[0-9]代表0和9之间的任意一个字符 |
| [^]  | 代表反向选择的意思。例如\[^abc]代表只要不是a、b、c三个字母的其他任意字符 |

| 符号  | 内容                                                         |
| :---: | ------------------------------------------------------------ |
|   #   | 注释符号                                                     |
|   \   | 转义符，将特殊字符或通配符还原成一般字符                     |
|  \|   | 管道符                                                       |
|   ;   | 连续命令执行分隔符，与管道符不一样                           |
|   ~   | 用户的home目录                                               |
|   $   | 使用变量的前导符                                             |
|   &   | 任务管理，将命令变成后台任务                                 |
|   !   | 逻辑运算意义上的非得意思                                     |
|   /   | 目录符号，路径分隔符                                         |
| >、>> | 数据流重定向：输出定向。前者替换，后者追加                   |
| <、<< | 数据流重定向：输入定向。前者输入定向；后者可接结束字符，表示输入中碰到定义得结束字符就结束输入 |
|  ' '  | 单引号，不具有变量替换功能                                   |
|  " "  | 双引号，具有变量替换功能                                     |
|  ``   | 反引号，中间可以先执行得命令，同$(命令)                      |
|  ( )  | 在中间的子shell块                                            |
|  { }  | 在中间为命令区块的组合                                       |



## 数据流重定向

- 标准输入（stdin）：代码为0，使用 < 或者 <<
- 标准输出（stdout）：代码为1，使用 > 或者 >>
- 标准错误输出（stderr）：代码2，使用 2> 或者 2>>

```shell
[root@localhost ~]# ll ~   # 此时屏幕会显示出文件名信息
[root@localhost ~]# ll ~ > ./temp.txt   # 屏幕并无任何信息，输出的信息都写入了文件temp.txt中
[root@localhost ~]# find /home -name .bashrc > list 2>&1  # 将标准输出和标准错误输出到同一个文件
[root@localhost ~]# find /home -name .bashrc &> list  # 将标准输出和标准错误输出到同一个文件
```

/dev/null垃圾桶黑洞设备与特殊写法

```shell
[root@localhost ~]# find /home -name .bashrc 2> /dev/null   # 将错误信息丢弃，屏幕上只显示正确的数据
```



命令执行的判断依据：；、&&、||

逻辑符号是通过命令的结果来判断的， 结果取值为$?，成功的返回值为0，其他值为失败。例如cmd1 && cmd2，若cmd1执行正确($?=0)，则执行cmd2；若cmd1执行正确($?不等于0)，则不执行cmd2。



## 管道命令（pipe）

管道命令仅能处理经由前面一个命令传来的正确信息，也就是标准输出的信息，对于标准错误并没有直接处理的能力。

- 管道命令仅会处理标准输出，对于标准错误会给予忽略
- 管道命令必须能够接受来自前一个命令的数据成为标准输入继续处理才行。

一般来说以下的命令都是针对**一行一行**来分析的。

### 选取命令：cut、grep

- cut：这个命令可以将一段信息的某一段给他切出来

```shell
[root@localhost ~]# cut -d 分隔字符 -f fields   # 用于有特定分隔符，且将其分割后选取第几个
[root@localhost ~]# cut -c 字符区间   # 截取区间内的字符串，例如截取第1-20个字符
选项与参数：
-d：后面接分隔符，与-f一起使用
-f：根据-d分隔符将一段信息划分成数段，用-f选取其中几段的意思
-c：以字符的单位取出固定字符区间
[root@localhost ~]# echo ${PATH}
/usr/local/lib/nodejs/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
#         第1段           |      第2段     |      第3段   |   第4段  |  第5段  |  第6段

# 将PATH变量取出，找出第五个路径
[root@localhost ~]# echo ${PATH} | cut -d ':' -f 5
/usr/bin

# 将PATH变量取出，找出第三、第五个路径
[root@localhost ~]# echo ${PATH} | cut -d ':' -f 3,5
/usr/local/bin:/usr/bin

[root@localhost ~]# export
declare -x DISPLAY="localhost:10.0"
declare -x HISTCONTROL="ignoredups"
declare -x HISTSIZE="1000"
declare -x HOME="/root"
declare -x HOSTNAME="localhost.localdomain"
declare -x JAVA_HOME="/usr/local/jdk1.8.0_221"

#不显示export输出的declare -x 
[root@localhost ~]# export | cut -c 12-         # 12-指的是从第12个字符到行尾，也可指定区间，例如12-40
DISPLAY="localhost:10.0"
HISTCONTROL="ignoredups"
HISTSIZE="1000"
HISTHOME="/root"
HOSTNAME="localhost.localdomain"
JAVA_HOME="/usr/local/jdk1.8.0_221"
LANG="en_US.UTF-8"

```

- grep：查找字符串，该命令可单独使用，也可配合管道符使用

```shell
[root@localhost ~]# grep [-acinvAB] [--color=auto] '查找字符串' [filename]
选项与参数：
-a：将二进制文件以文本形式查找数据
-c：计算找到'查找字符串'的次数
-i：忽略大小写
-n：输出行号
-v：反向选择，即显示出没有'查找字符串'内容的那一行
-A：后面可接数字，after的意思，除了列出改行外，后续的n行也列出来
-B：后面可接数字，before的意思，除了列出改行外，前面的n行也列出来
--color=auto：可以将找到的关键字部分加上颜色显示出来
[root@localhost ~]# last | grep 'root'   # 查找有出现root的那一行数据
[root@localhost ~]# last | grep -v 'root'   # 查找没有出现root的那一行数据
[root@localhost ~]# last | grep 'root' | cut -d ' ' -f 1   # 查找有出现root的那一行，并将第一列取出来
```



### 排序命令：sort、wc、uniq

- sort：排序与语系的编码有关，默认以字符升序

```shell
[root@localhost ~]# sort [-fbMnrtuk] [file or stdin]
选项与参数：
-f：忽略大小写
-b：忽略最前面的空格
-M：以月份的名字来排序，例如JAN、DEC等的排序方法
-n：使用纯数字进行排序，默认是以文字来排序
-r：反向排序
-u：就是uniq，相同的数据，仅输出一行
-t：分隔符号，默认使用[tab]，配合-k使用
-k：以哪个区间（field）来排序，配合-t使用

# 个人账号都记录在/etc/passwd中，请将账号排序
[root@localhost ~]# cat /etc/passwd | sort
adm:x:3:4:adm:/var/adm:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
halt:x:7:0:halt:/sbin:/sbin/halt
jw:x:1000:1000::/home/vsftpd/jw/file:/bin/bash
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

# /etc/passwd内容是以:分隔的，我想以第三栏来排序
[root@localhost ~]# cat /etc/passwd | sort -t ':' -k 3  # 发现输出结果仅是以第三栏内容的字符串排序
root:x:0:0:root:/root:/bin/bash
jw:x:1000:1000::/home/vsftpd/jw/file:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin

# /etc/passwd内容是以:分隔的，我想以第三栏按数字来排序
[root@localhost ~]# cat /etc/passwd | sort -n -t ':' -k 3
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt

```

- uniq：去重

```shell
[root@localhost ~]# uniq [-ic]
选项与参数：
-i：忽略大小写
-c：进行计数

# 使用last将账号列出，仅列出账号栏，进行排序后去重
[root@localhost ~]# last | cut -d ' ' -f 1 | sort |uniq
reboot
root
wtmp

# 使用last将账号列出，仅列出账号栏，进行排序后去重并计数
[root@localhost ~]# last | cut -d ' ' -f 1 | sort |uniq
      1 
     15 reboot
    511 root
      1 wtmp
```

- wc：统计行、字符数

```shell
[root@localhost ~]# wc [-lwm]
选项与参数：
-l：仅列出行
-w：仅列出多少字（英文字母）
-m：多少字符

[root@localhost ~]# last | wc
    528    5279   40540
[root@localhost ~]# last | wc -l
528
[root@localhost ~]# last | wc -w
5279
[root@localhost ~]# last | wc -m
40540
```



### 双向重定向：tee

tee同时将数据流送到文件与屏幕

```shell
[root@localhost ~]# tee [-a] file
选项与参数：
-a：以累加的方式写入file
[root@localhost ~]# last | tee tast.list
```



### 划分命令： split

如果有一个文件太大，导致携带不太方便的话，可以用split来切分。它可以按文件大小或者行数来切分称小文件。

```shell
[root@localhost ~]# split [-bl] file prefix
选项与参数：
-b：以文件大小来切分，后接文件大小，可加单位b、k、m等
-l：以行数来进行切分，后接行数
prefix：代表前缀字符
# 切分后文件按照英文字母a、b、c的顺序生成
# /etc/services有六百多k，若想分成300k一个文件时
[root@localhost ~]# split -b 300k /etc/services services
[root@localhost ~]# ls
servicesa servicesb servicesc
```



### 减号【-】的用途

管道命令在bash的连续的处理程序中是相当重要的。另外，在日志文件的分析当中也是相当重要的一环。在管道命令中，常常会使用到前一个命令的stdout作为这次的stdin，某些命令需要用到文件名来进行处理时，该stdin和sdtout可以利用减号【-】来替代。

