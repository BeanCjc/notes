# 十三、Linux账号管理与与ACL权限设置

## Linux的账号与用户组

用户表示UID与GID

Linux的账号信息存储在`/etc/passwd`文件中，密码存储相关信息存储在`/etc/shadow`文件中。用户组存储在`/etc/group`文件中，用户组密码相关信息存储在`/etc/gshadow`文件中。



### `/etc/passwd`文件结构

这个文件的构造是这样的：每一行代表一个账号，有几行就代表有几个账号在Linux系统中。不过需要注意的是，里面很多账号本来就是系统正常运行所必须的，可以称为系统账号，例如bin、daemon、adm、nobody等，这些账号不要随意删除。因为各程序需要读取`/etc/passwd`来了解不同账号的权限，因此**`/etc/passed`的权限需要设置为-rw-r--r--**。

下面查看下这个文件的具体内容：

```shell
[root@localhost ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
jw:x:1000:1000::/home/vsftpd/jw/file:/bin/bash
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
ntp:x:38:38::/etc/ntp:/sbin/nologin
```

每个Linux系统都有该文件中的第一行，就是root这个系统管理员。每一行使用【:】分隔开，总共七个字段。

**1. 账号名称**

**2. 密码**

早期密码是存放在这个字段上，但由于该文件的特性是所有的程序都能够读取，这样一来就很容易造成密码数据被窃取，因此后来就将这个字段改成放到`/etc/shadow`文件中，同时这里用一个【x】来替代显示。

**3. UID**

用户标识符。通常Linux对于UID有几个限制如下：

| ID范围                | 该ID用户特性                                                 |
| --------------------- | ------------------------------------------------------------ |
| 0：系统管理员         | 当UID是0时，代表该账号是系统管理员，所以当你要让其他账号名称也具有root的权限时，将该账号的UID改成0即可。也就是说，一台系统上面的系统管理员不见得就只有root，不过，不建议有多个账号的UID是0，容易让系统管理员混乱 |
| 1~999：系统账号       | 保留给系统使用的ID，除了0之外，其他的UID权限与特性并没有不一样。默认1000以下的数字留给系统作为保留账号只是一个习惯。由于系统上面启动的网络服务或后台服务希望使用较小的权限去运行，因此不希望使用root的身份去执行这些服务，所以我们就要提供这些运行中程序的拥有者账号才行。这些系统账号通常是不可登录的，这类账号又分为两种：1~200：有Linux发行版自行建立的系统账号；201~999：若用户有系统账号需求时，可以使用的账号UID |
| 1000~6000：可登录账号 | 给一般用户使用                                               |

**4. GID**

这个与`/etc/group`文件有关，`/etc/group`的概念与`/etc/passwd`差不多，只是用来规范组名与GID的对应而已。

**5. 用户信息说明栏**

这个字段基本没什么用途，只是用来解释这个账号的意义而已。不过，如果使用`finger`命令时，这个字段可以提供更多的信息。

**6. home目录**

**7. shell**



### `/etc/shadow`文件结构

查看下这个文件的具体内容：

```shell
[root@localhost ~]# cat /etc/shadow
root:$6$cVKD/WQ4x/OUIF1f$lQ7myLr6.wH/6NgmIKGJqV8HD3vMDugOqW/exL/Lj4c4/NBdKcBT7RWlKoUw0/RGfwsW1fupE.Y5bFyV/Dyn4/::0:99999:7:::
bin:*:17834:0:99999:7:::
daemon:*:17834:0:99999:7:::
adm:*:17834:0:99999:7:::
lp:*:17834:0:99999:7:::
sync:*:17834:0:99999:7:::
shutdown:*:17834:0:99999:7:::
halt:*:17834:0:99999:7:::
mail:*:17834:0:99999:7:::
operator:*:17834:0:99999:7:::
games:*:17834:0:99999:7:::
ftp:*:17834:0:99999:7:::
nobody:*:17834:0:99999:7:::
systemd-network:!!:18245::::::
dbus:!!:18245::::::
polkitd:!!:18245::::::
sshd:!!:18245::::::
postfix:!!:18245::::::
chrony:!!:18245::::::
jw:$6$wU27s4s5$VydDRxFD/riwkeCuKUzGjwrkSmHbJGiLlTDIXzz0QNJ674ZN.IoBtzFnBLybhFhQUxU7LxSw5yl2YhYElTG93.:18611:0:99999:7:::
tss:!!:18710::::::
ntp:!!:18711::::::
```

因为该文件涉及到密码，所以这个文件的默认权限为**【-rw-------】**或**【----------】**，即只有root才可以读写。该文件同样以【:】作为分隔符，总共有九个字段：

**1. 账号名称**

与账号对应，必须与`/etc/passwd`相同才行

**2. 密码**

密码是经过摘要算法算出来，一般默认使用sha512算法。当修改这个字段的内容后，改密码就会失效。因此很多软件通过这个功能，在此字段前面加上**!**或者*****修改密码字段，就会让密码暂时失效。可以通过命令`authconfig --test | hashing`查看使用的是哪种加密方式。

```shell
[root@localhost ~]# authconfig --test|grep hashing
 password hashing algorithm is sha512
```

**3. 最近修改密码的日期**

这个字段记录了最近一次修改密码的那一天的日期，该字段记录的方式为：以1970年1月1日作为基础日期，距离今天的天数，1970年1月1日为第一天。如果该字段为0，即表示用户首次登录系统后需要强制修改密码。

**4. 密码不可被修改的天数（一般默认为0，不限制）**

这个字段记录了最后一次修改密码后最少要经过多少天才可以再次修改密码，同样以天数来记录。该字段值为0时表示不限制密码修改。

**5. 密码需要重新修改的天数（一般默认为9999，不限制）**

经常修改密码是个好习惯。为了强制用户修改密码，这个字段可以指定在最近一次修改密码后多少天内需要再次修改密码。若值为99999（273年）表示密码的修改没有强制性的意思。若在这个天数内没有修改密码这个账号的密码将会过期。

**6. 密码需要修改期限前的警告天数（一般默认为7）**

当需要修改密码的日期快到时，会提前几天提醒用户修改密码

**7. 密码过期后的账号宽限天数（也叫密码失效日）**

密码过期后还允许几天内可以登录系统。登录时系统会强制要求你必须重新设置密码才能登录继续使用。若超过了宽限天数，即到达了密码失效日后，该账号无法再使用该密码登录。

**8. 账号失效日（以天数记）**

账号失效日规定，在该日期后该账号再也无法使用。

**9. 保留**



忘记密码的解决方案有：

- 一般账号密码忘记：请管理员帮忙，以root身份使用`passwd`命令重置密码即可
- root账号密码忘记：通过各种手段进入系统并修改`/etc/shadow`文件，将密码字段置空，重启便可以空密码进入，再进行修改密码即可。一般通过重新启动后进入单人维护模式的方法进入系统。



### `/etc/group`文件结构

查看下这个文件的具体内容：

```shell
[root@localhost ~]# cat /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:
cdrom:x:11:
mail:x:12:postfix
man:x:15:
dialout:x:18:
floppy:x:19:
games:x:20:
tape:x:33:
video:x:39:
ftp:x:50:
lock:x:54:
audio:x:63:
nobody:x:99:
users:x:100:
utmp:x:22:
utempter:x:35:
input:x:999:
systemd-journal:x:190:
systemd-network:x:192:
dbus:x:81:
polkitd:x:998:
ssh_keys:x:997:
sshd:x:74:
postdrop:x:90:
postfix:x:89:
chrony:x:996:
jw:x:1000:
tss:x:59:
ntp:x:38:
```

该文件同样以【:】作为分隔符，总共有四个字段：

**1. 用户组名称**

**2. 用户组密码**

通常不需要设置，这个设置通常是给用户组管理员使用，目前基本不会使用到。同样密码已经移动到`/etc/gshadow`文件中，这里只显示一个【x】而已。

**3. GID**

**4. 此用户组支持的账号名称**

一个账号是可以支持多个用户组的，所以如果想要某个账号加入此用户组，将该账号填入这个字段即可。多个用逗号隔开。例如想让myuser1合myuser2也加入root这个组，那么在第一行的最后面加上`myuser1,myuser2`即可，注意不能有空格，使其成为：`root:x:0:myuser1,myuser2`。

至于`/etc/group`文件的第四栏，因为每个用户都可以拥有多个用户组，那么有个奇怪的现象就是，当创建文件时该新建的文件的所属用户组应该是哪一个呢？所以也就有有效用户组和初始用户组的概念。前面的这个问题的答案就是该新建的文件会使用用户的**有效用户组**。

- 初始用户组

`/etc/passwd`文件中第四栏的GID，这个GID表示的就是初始用户组（initial group）。也就是说，一个用户一登录系统，立刻就会有用这个用户组的相关权限。因为是初始用户组，所以在`/etc/group`文件的第四栏中无需体现该用户。一般情况下用户的初始用户组就是用户的有效用户组。

- 有效用户组

在账号的所有用户组中通过命令`groups`显示出来的第一个即为有效用户组。该命令会显示出账号的所有用户组。

```shell
[root@localhost ~]# groups
root mygroup1
```



有效用户组可以通过命令`newgrp`进行切换，你想要切换的用户组必须是该账号已经有支持的用户组。命令如下：

```shell
[root@localhost ~]# groups mygroup1
[root@localhost ~]# groups  # 此时查看有效用户组已切换为mygrouop1了
# 通过touch或者其他命令创建的文件或目录的所属组就是mygroup1了
mygroup1 root
[root@localhost ~]# exit
```

最后为什么要exit退出newgrp环境呢？因为`newgrp`这个命令是以另外一个shell来提供这个功能的。新的shell环境会重新计算用户组权限。若想回到原本环境只要输入exit即可。



### `/etc/gshadow`文件结构

查看下这个文件的具体内容：

```shell
[root@localhost ~]# cat /etc/gshadow
root:::
bin:::
daemon:::
sys:::
adm:::
tty:::
disk:::
lp:::
mem:::
kmem:::
wheel:::
cdrom:::
mail:::postfix
man:::
dialout:::
floppy:::
games:::
tape:::
video:::
ftp:::
lock:::
audio:::
nobody:::
users:::
utmp:!::
utempter:!::
input:!::
systemd-journal:!::
systemd-network:!::
dbus:!::
polkitd:!::
ssh_keys:!::
sshd:!::
postdrop:!::
postfix:!::
chrony:!::
jw:!::
tss:!::
ntp:!::
```

该文件同样以【:】作为分隔符，总共有四个字段：

**1. 组名**

**2. 密码栏**

**3. 用户组管理员的账号**

**4. 有加入该用户组支持的所属账号（与`/etc/group`内容相同）**



## 账号管理

### 用户新增、修改、删除

#### `useradd`

命令`useradd`用来创建账号，密码则使用`passwd`来设置。

```shell
[root@localhost ~]# useradd [-u UID] [-g 初始用户组GID] [-G 次要用户组GID] [-mM] [-c 说明栏] [-d home目录的绝对路径] [-s shell] 使用者账号
参数与选项：
-u：后面接UID，是一个数字，直接指定一个特定的UID给这个账号
-g：后面接用户组（可以是GID也可以是用户组名），这个用户组就是该账号的初始用户组，该GID会被放到/etc/passwd的第四个栏位
-G：后面接用户组（可以是GID也可以是用户组名），该账号可加入的其他用户组，该用户组必须提前存在，该参数会修改/etc/group文件
-M：强制，不要建立使用者的home目录（系统账号默认值）
-m：强制，要建立使用者的home目录（一般账号默认值）
-c：就是/etc/passwd文件的第五栏
-d：指定某个目录为home目录，使用绝对路径
-r：建立一个系统的账号
-s：后面接一个shell，若没有指定默认是/bin/bash
-e：后面接一个日期，格式为【YYYY-MM-DD】此选项可写入/etc/shadow的第八栏位，账号失效日期
-f：后面接/etc/shadow的第七栏，指定密码是否会失效，0为立刻失效，-1为永远不失效

# 使用默认值建立一个使用者 myuser
[root@localhost ~]# useradd myuser
# 建立一个系统账号 myuser1
[root@localhost ~]# useradd -r myuser1  # 系统账号默认不会建立home目录
```

使用useradd建立账号时，其实会修改不少文件：`/etc/passwd`、`/etc/shadow`、`/etc/group`、`/etc/gshadow`、`/home/账号名称`。那么使用useradd建立用户时的参考文件有什么呢？主要包含：`/etc/default/useradd`、`/etc/login.defs`、/etc/skel/*`。

- `/etc/default/useradd`

查看useradd的默认参数可使用命令`useradd -D`或者查看文件`/etc/default/useradd`

```shell
[root@localhost ~]# useradd -D
GROUP=100
HOME=/home     # 默认的home目录所在基准目录
INACTIVE=-1    # 密码失效日，shadow的第七栏
EXPIRE=        # 账号失效日，shadow的第八栏
SHELL=/bin/bash   # 默认的shell
SKEL=/etc/skel    # home目录下的内容数据参考目录
CREATE_MAIL_SPOOL=yes   # 是否主动帮使用者创建邮箱

[root@localhost ~]# cat /etc/default/useradd 
# useradd defaults file
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

- `/etc/skel`默认包含三个文件：`.bashrc`、`.bash_logout`、`.bash_profile`

- `/etc/login.defs`文件提供了UID和GID的密码参数设置参考。该文件内容如下：

```shell
[root@localhost ~]# cat /etc/login.defs 
MAIL_DIR	/var/spool/mail          # 使用者默认邮箱放置目录
PASS_MAX_DAYS	99999                # /etc/shadow文件的第五栏，多久需要修改密码天数
PASS_MIN_DAYS	0					 # /etc/shadow文件的第四栏，多久不可修改密码天数
PASS_MIN_LEN	5					 # 密码最短的字符长度，已被PAM模块替换，已经启用该参数
PASS_WARN_AGE	7					 # /etc/shadow文件的第六栏，过期前警告天数
UID_MIN                  1000        # 使用者最小的ID，即小于1000为系统保留
UID_MAX                 60000        # 使用者能够使用的最大ID
SYS_UID_MIN               201		 # 保留给使用者自行设置的系统账号最小UID
SYS_UID_MAX               999		 # 保留给使用者自行设置的系统账号最大UID
GID_MIN                  1000		 # 使用者自定义用户组的最小GID，小于1000为系统保留
GID_MAX                 60000		 # 使用者自定义用户组的最大GID
SYS_GID_MIN               201		 # 保留给使用者自行设置的系统账号最小GID
SYS_GID_MAX               999		 # 保留给使用者自行设置的系统账号最小GID
CREATE_HOME	yes						 # 在不加-m即-M时，是否主动建立使用者home目录
UMASK           077					 # 使用者home目录的权限，700
USERGROUPS_ENAB yes					 # 使用userdel时，是否会删除初始用户组
ENCRYPT_METHOD SHA512				 # 密码加密的机制使用的是sha-512
```



#### `passwd`

默认情况下，新建立的账号是暂时锁定的，也就是说该账号是无法登录的。需要使用`pwsswd`命令进行设置。

```shel
[root@localhost ~]# passwd [--stdin] [账号名称] # 所有人均可使用来改自己的密码
[root@localhost ~]# pwsswd [-l] [-u] [--stdin] [-S] [-n 天数] [-x 天数] [-w 天数] [-i 天数] [-e 天数] 账号    # root功能
选项与参数：
--stdin：可以通过来自前一个管道的数据，作为密码输入
-l：是lock的意思，会将/etc/shadow的第二栏最前面加上!是密码失效
-u：unlock的意思
-S：status的意思，列出密码相关参数，即/etc/shadow文件内大部分信息
-n：minimum的意思，后面接天数，/etc/shadow的第四栏，多少天不可修改密码
-x：maximum的意思，后面接天数，/etc/shadow的第五栏，多少天内必须修改密码
-w：warning的意思，后面接天数，/etc/shadow的第六栏，密码过期前多少天提醒
-i：inactive的意思，后面接天数，/etc/shadow的七栏，密码失效日
-e：expire的意思，后面接天数，/etc/shadow的第八栏，账号过期日

# 请root设置myuser的密码
[root@localhost ~]# passwd myuser
# 修改自己密码需要输入自己的就密码，root除外
# 输入密码并确认密码

# myuser登录后，修改自己密码
[myuser@localhost ~]$ passwd   # 后面没有加账号就是改自己密码
# 输入密码并确认密码

# 使用标准输入建立用户密码，这样的操作直接更新用户的密码，无需二次输入确认
[root@localhost ~]# echo "mypassword@123" | passwd --stdin muyser

# 锁定用户myuser，其实只是在/etc/shadow的密码字段最前面增加!!
[root@localhost ~]# passwd -l myuser

# 解锁
[root@localhost ~]# passwd -u myuser

# 设置myuser60天内要修改密码
[root@localhost ~]# passwd -x 60 myuser

# 查看root的密码相关信息
[root@localhost ~]# passwd -S root
root PS 1969-12-31 0 99999 7 -1 (Password set, SHA512 crypt.)
# 密码建立时间，0 最小天数， 99999 最大天数， 7 密码过期提醒天数， -1 密码不会失效，密码加密方式
```

要帮一般账号建立或修改密码使用`passwd 账号`，使用`passwd`表示修改自己的密码。

与root账号不同的是，一般账号在修改自己密码时需要先输入自己的旧密码，然后在输入新密码。

密码规则最好符合以下几点：

- 密码不与账号相同
- 密码尽量不要选用字典里会出现的字符串
- 密码需要超过8个字符
- 密码不要使用个人信息，如身份证、手机号码等
- 密码不要使用简单的关系式，比如1+1=2等
- 密码尽量使用大小写字符、数字、特殊字符的组合



#### `chage`

除了使用`passwd -S`之外，`chage`命令也可用来设置密码的相关参数，改命令无法修改密码。用法如下：

```shell
[root@localhost ~]# chage [-ldEInMW] 账号名
选项与参数：
-l：列出该账号的详细密码参数
-d：后面接日期，修改/etc/shadow的第三栏，最近修改密码的日期，格式为【YYYY-MM-DD】
-m：后面接天数，修改/etc/shadow的第四栏，多少天内不可修改密码
-M：后面接天数，修改/etc/shadow的第五栏，多少天内必须修改密码
-W：后面接天数，修改/etc/shadow的第六栏，密码过期前多少天提醒
-I：后面接天数，修改/etc/shadow的第七栏，密码失效日
-E：后面接天数，修改/etc/shadow的第八栏，账号失效日

# 列出root的详细密码数据
[root@localhost ~]# chage -l root
Last password change					: never
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7

# 用户第一次登录时，强制它们修改密码后才能够使用系统
[root@localhost ~]# chage -d 0 myuser
# 此时账号的密码建立时间会被改为1970/1/1，所以会有问题，必须要用户登陆后修改密码
```



#### `usermod`

修改用户信息，当然也可以直接修改`/etc/passwd`、`/etc/shadow`文件。用法如下：

```shell
[root@localhost ~]# usermod [-cdegGlsuLU] username
选项与参数：
-c：后面接账号说明，即/etc/passwd的第五栏
-d：后面接用户的home目录，即/etc/passwd的第六栏
-e：后面接日期，格式为YYYY-MM-DD，账号失效日，即/etc/shadow的第八栏
-f：后面接天数，密码失效日，即/etc/shadow的第七栏
-g：后面接初始用户组，即/etc/passwd的第四栏
-G：后面接次要用户组，会修改/etc/group
-a：与-G合用，可增加次要用户组，而非设置
-l：后面接账号名称，即修改账号名称，即/etc/passwd的第一栏
-s：后面接shell的实际文件，例如/bin/sh或/bin/bash等
-u：后面接UID数字，即/etc/passwd的第三栏
-L：锁定账号，其实是修改/etc/shadow的密码栏，增加!
-U：解锁账号
其他参数可使用usermod --help或者man usermod进行查看
```



#### `userdel`

删除用户会涉及到的数据有：`/etc/passwd`、`/etc/shadow`、`/etc/group`、`/etc/gshadow`、`/home/username`、`/var/spool/mail/username`。用法如下：

```shell
[root@localhost ~]# userdel [-r] username
选项与参数：
-r：连同使用者的home目录和邮箱目录（/var/spool/mail/username）一起删除
[root@localhost ~]# userdel -r myuser
```



### 用户功能

上述`useradd`、`usermod`、`userdel`都是**系统管理员**所使用的命令，如果我是一般用户，那么我除了修改密码之外，就无法修改其他数据了吗？以下这些命令可以给**一般用户**使用。

#### `id`

`id`这个命令可以查询某人或自己相关的UID/GID等信息。

```shell
[root@localhost ~]# id [username]
[root@localhost ~]# id   # root查看自己的相关ID信息，后面那个context=...则是SELinux的内容，先不管
uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[root@localhost ~]# id root
uid=0(root) gid=0(root) groups=0(root)
```

#### `finger`

该命令可以查看的信息大部分都在`/etc/shadow`文件中，默认不安装该命令。

```shell
[root@localhost ~]# finger [-s] username
选项与参数：
-s：仅列出使用者的账号、全名、终端代号与登录时间
-m： 列出与后面接的账号相同者，而不是利用部分比对

# 查看root的使用者相关账号属性
[root@localhost ~]# finger -s root
Login     Name       Tty      Idle  Login Time   Office     Office Phone   Host
root      root       pts/0      4d  Sep 23 15:30                           (10.10.71.30)
root      root       pts/1      1d  Sep 27 16:08                           (10.10.71.42)
root      root       pts/2          Sep 28 19:26                           (10.10.40.40)
root      root       pts/3      2d  Sep 23 15:55                           (10.10.71.102)
```



#### `chsh`

change shell的意思。

```shell
[root@localhost ~]# chsl [-ls]
选项与参数：
-l：列出目前系统上可用的shell，其实就是/etc/shells的内容
-s：设置修改自己的shell

# 以myuser的身份列出系统上所有合法的shell
[myuser@localhost ~]$ chsh -l
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash

# 修改shell为sh
[myuser@localhost ~]$ chsh -s /bin/sh

# 通过查看/etc/passwd文件看shell是什么
[myuser@localhost ~]$ grep myuser /etc/passwd
myuser:x:1001:1001:myuser:/myuser:/bin/sh
```



### 用户组新增、修改、删除

#### `groupadd`

```shell
[root@localhost ~]# groupadd [-g gid] [-r] groupname
选项与参数：
-g：后面接某个特定的GID，用来直接设置某个GID
-r：建立系统用户组，与/etc/login.defs内的GID_MIN有关

# 建立一个用户组，名称为group1
[root@localhost ~]# groupadd group1
[root@localhost ~]# grep group1 /etc/group /etc/gshadow
/etc/group:group1:x:1001:
/etc/gshadow:group1:!::
```



#### `groupmod`

```shell
[root@localhost ~]# groupmod [-g gid] [-n newgroupname] groupname
选项与参数：
-g：修改既有的GID数字
-n：修改既有的用户组名称

# 将刚刚建立的group1名称改为mygroup，GID改为201
[root@localhost ~]# groupmod -g 201 -n mygroup group1
[root@localhost ~]# grep group1 /etc/group /etc/gshadow
/etc/group:mygroup:x:201:
/etc/gshadow:mygroup:!::
```



#### `groupdel`

```shell
[root@localhost ~]# groupdel groupname

# 删除上面建立的用户组mygroup
[root@localhost ~]# groupdel mygroup

# 若某个账号的初始用户组使用该用户组，也就是/etc/passwd的第四栏中引用了该用户组是不能删除的
[root@localhost ~]# groupdel grouptest
grouptest:cannot remove the primary group of user 'username'
```



#### `gpasswd`

用户组管理员功能，该功能已经不怎么使用了。





## 主机的详细权限规划：ACL的使用

ACL是access control list的缩写，中文译为访问控制列表，主要目的是提供传统的属主、所属用户组、其他人的读、写、执行之外的详细权限设置。ACL可以针对单一用户、单一文件或目录进行r、w、x的权限设置，对于需要特殊权限的使用状况非常有帮助。目前ACL几乎已经默认加入了所有常见的Linux文件系统的挂载参数中（ext2、ext3、ext3、xfs等）。

ACL主要可以针对哪些方面来控制权限，主要有如下三个：

- 用户（user）：可以针对用户设置权限
- 用户组（group）：可以针对用户组为对象来设置权限
- 默认属性（mask）：还可以针对在该目录下建立新文件/目录时，规范新数据的默认权限



### ACL设置技巧：`getfacl`、`setfacl`

- `gefacl`：获取某个文件/目录的ACL设置选项
- `setfacl`：设置某个目录/文件的ACL规范

`setfacl`命令用法介绍及最简单的【u:账号:权限】设置

- 针对单一用户设置

```shell
[root@localhost ~]# setfacl [-bkRd] [{-m|-x} acl参数] 目标文件名
选项与参数：
-m：设置后续的ACL参数给文件用，不可与-x合用
-x：删除后续的ACL参数，不可与-m合用
-b：删除所有的ACL设置参数
-k：删除默认的ACL设置参数
-R：递归设置ACL，即包括子目录都会被设置起来
-d：设置默认ACL参数，只对目录有效，在该目录新建的数据会引用此默认值

# 针对单一用户的设置方式
# 设置规范【u:[username]:[rwx]】，例如针对myuser的权限规范rx
[root@localhost ~]# touch acl_test
[root@localhost ~]# ll acl_test
-rw-r--r--. 1 root root 0 Sep 28 23:18 acl_test
[root@localhost ~]# setfacl -m u:myuser:rx acl_test
[root@localhost ~]# ll acl_test
-rw-r-xr--+ 1 root root 0 Sep 28 23:19 acl_test
# 权限部分多了个【+】，并且权限由原本的644变成654

[root@localhost ~]# setfacl -m u::rwx acl_test
# 设置值中的u后面无使用者列表，表示设置该文件的拥有者，所以root的权限变成了rwx
[root@localhost ~]# ll acl_test
-rwxr-xr--+ 1 root root 0 Sep 28 23:19 acl_test
```



`getfacl`命令用法：

```shell
[root@localhost ~]# getfacl filename
选项与参数：
getfacl的选项几乎与setfacl相同

# 列出刚才设置的acl_test的权限内容
[root@localhost ~]# getfacl acl_test
# file: acl_test     # 说明文件名而已
# owner: root		 # 此文件的拥有者
# group: root		 # 此文件的所属用户组
user::rwx			 # 使用者列表栏是空，代表文件所有者，此处为root
user:myuser:r-x		 # 针对myuser的权限设置为rx，与拥有者并不同
group::r--			 # 针对文件用户组的权限设置仅有r
mask::r-x			 # 此文件默认的有效权限（mask）
other::r--			 # 其他人拥有的权限
```



- 针对单一用户组的权限设置：【g:[groupname]:[rwx]】

```shell
# 设置规范【g:[grouprname]:[rwx]】，例如针对mygroup的权限规范rx
[root@localhost ~]# setfacl -m g:mygroup:rx acl_test
[root@localhost ~]# getfacl acl_test
# file: acl_test
# owner: root
# group: root
user::rwx
user:myuser:r-x
group::r--
group:mygroup:r-x	 # 这里是新增部分，多了这个用户组的权限设置
mask::r-x
other::r--
```



- 针对有效权限设置：【m:权限】

```shell
# 设置规范【m:[rwx]】，例如针对刚刚的文件规范为仅有r
[root@localhost ~]# setfacl -m m:r acl_test
[root@localhost ~]# getfacl acl_test
# file: acl_test
# owner: root
# group: root
user::rwx
user:myuser:r-x		 # 有效的仅有r，x不会生效
group::r--
group:mygroup:r-x	 # 有效的仅有r，x不会生效
mask::r-x
other::r--
```

通过设置mask来规范最大允许的权限。如上例中，mask仅为r，尽管myuser由r-x的权限，由于mask仅为r，故myuser的权限也只有r。



- 使用默认权限设置目录未来文件的ACL权限继承：【d:[u|g]:[username|groupname]:[rwx]】

```shell
# 设置规范【d:[u|g]:[username|groupname]:[rwx]】
# 例如让myuser在/src/project下面一直具有rx的默认权限
[root@localhost ~]# setfacl -m d:u:myuser:rx /src/project
[root@localhost ~]# getfacl /src/project
# file: /src/project
# owner: root
# group: root
user::rwx
user:myuser:r-x
group::r--
mask::r-x
other::r--
default:user:rwx
default:myuser:r-x
default:group::r-x
default:mask::r-x
default:other::r--
# 之后在目录/src/project下新建的文件或者目录myuser账号都会有rx的权限
```



## 用户切换身份su、sudo

### su(switch user)

`su`命令用来切换账号。

```shell
[root@localhost ~]# su [-lm] [-c 命令] [username]
选项与参数：
- :单纯使用su - 代表使用login-shell的变量文件读取方式来登陆系统。若没有username，则代表切换为root
-l：等同于-
-m：-m与-p一样，表示使用目前的环境设置，不读取新使用者的配置文件
-c：仅进行一次命令，所以-c后面可以加命令
```

单纯以su切换的用户，例如`su`（切换到root），都是以non-login shell的方式切换账号，此时环境变量都还是切换之前账号的变量。所以一般切换账号都会使用`su - [username]`。切换用户是会新进入一个shell进程，所以若想回到上一个账号的shell环境，可以使用`exit`命令退出当前新账号的环境。

使用root切换到任何用户时，无需输入密码。



### sudo

相对于su需要了解新切换的用户密码（常常是需要root的密码），sodu的执行则仅需要自己的密码即可。甚至可以设置不需要密码即可执行sudo。由于sudo可以让你以其他用户的身份执行命令（通常使用root的身份来执行命令），**因此并非所有人都能够执行sudo，而是仅有规范到`/etc/sudoers`文件内的用户才能执行`sudo`这个命令**。事实上一般用户要具有sodu的权限，需要管理员先审核通过后才能开放。系统最初默认规定仅有root可以执行sudo。

```shell
[root@localhost ~]# sudo [-b] [-u username] 命令
选项与参数：
-b：将后续的命令放到后台中让系统执行，而不与目前的shell产生英雄
-u：后面接欲切换的账号，若无此项则代表切换身份为root

# 若想以sshd的身份在/tmp下面建立一个名为mysshd的文件
[root@localhost ~]# sudo -u sshd touch /tmp/mysshd

# 若想以root的身份查看/etc/shadow文件
[myuser@localhost ~]$ sudo cat /etc/shadow
# 此时需要输入myuser自己的密码
```

sudo默认仅有root能使用，为什么呢？因为sudo的执行流程是这样的：

1. 当用户执行sudo时，系统于`/etc/sudoers`文件中查找该用户是否具有执行`sudo`的权限
2. 若用户具有可执行`sudo`的权限后，便让用户输入自己的密码来确认
3. 若密码输入成功，便开始在执行后续接的命令
4. 若欲切换的身份与执行者身份相同，那也不需要输入密码；root执行sudo时不需要输入密码

所以sudo执行的重点是：能否使用sudo必须看`/etc/sudoers`的设置值，而可以使用sudo者是通过输入自己的密码来执行后续的命令。

除了root之外的其他账号，若想要使用sudo执行属于root权限命令，则root需要先使用visudo去修改/etc/sudoers，让该账号能够使用全部或部分的root命令功能。为什么要使用visudo呢？因为visudo是设置过语法的，如果设置错就会造成无法使用sudo命令的后果。因此才会使用visudo去修改，并且在结束退出修改界面时，操作系统会去检查/etc/sudoers的语法是否正确。

一般来说visudo的设置有以下6种简单的方法：

**1. 单一用户可使用root所有命令，与sudoers文件语法**

```shell
[root@localhost ~]# visudo
......前面省略......
root	ALL=(ALL)		ALL		# 这一行是默认值
myuser	All=(ALL)		ALL		# 新增这一行
# 使用者账号   登录者的来源主机名称=（可切换的身份）     可执行的命令
......下面省略......
```

上述四栏的意义：

- 用户账号：操作系统的哪个账号可以使用sudo这个命令
- 登录者的来源主机名：当这个账号由哪台主机连接到本Linux系统，意思是这个账号可能是由哪一台网络主机连接过来的，这个设置值可以指定客户端计算机（信任的来源的意思），默认值root可来自任何一台网络主机
- 可切换的身份：这个账号可以切换成什么身份来执行后续的命令，默认root可以切换成任何人
- 可执行的命令：可用该身份执行什么命令？**这个命令请务必使用绝对路径**，默认root可以切换任何身份来执行任何命令。多个命令用【,】隔开。

那个ALL是特殊的关键词，代表任何身份、任何主机、任何命令的意思。



**2. 利用wheel用户组以及免密码的功能处理visudo**

假设由三个用户，usre1、user2、user3，能否通过用户组的功能让这三个人可以管理系统？

```shell
[root@localhost ~]# visudo
......前面省略......
%wheel	ALL=(ALL)		ALL		# 这一行是默认值
# 在最左边加上%代表后面接的是一个用户组

%wheel	ALL=(ALL)		NOPASSWD: ALL
					    # 该NOPASSWD关键词表示免除密码输入的意思
......下面省略......

# 所以只需将这三个人加入到wheel这个用户组即可成为管理员，之后这三个人就可以使用sudo
[root@localhost ~]# usermod -a -G wheel user1
[root@localhost ~]# usermod -a -G wheel user2
[root@localhost ~]# usermod -a -G wheel user3
```



**3. 有限的命令操作**

假设想让用户user1仅能使用passwd这个命令，该如何配置？

```shell
[root@localhost ~]# visudo
......前面省略......
user1	ALL=(root)		/usr/bin/passwd    # 命令必须以绝对路径
......下面省略......

# 修改user2的密码
[user1@localhost ~]$ sudo passwd user2
[sudo] password for user1:        # 输入user1的密码
Changing password for user user2.   # 下面修改的是user2的密码
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

# 修改root密码
[user1@localhost ~]$ sudo passwd
Changing password for user root.   # 可见修改的是root的密码
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

# 该配置正常是可以修改一般用户的密码，但同时也能修改root的密码，所以需要做以下调整
[root@localhost ~]# visudo
......前面省略......
user1	ALL=(root)		!/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd root
# 在设置命令的前面增加!表示不可执行的意思，因此这样设置user1就无法修改root的密码了
......下面省略......
```



**4. 通过别名创建visudo**

假设有15个用户需要加入管理员行列，那么除了前面将这15个都加入whell用户组之外有没有别的方法设置呢？答案是有的。v可以通过设置别名来进行配置。visudo的别名可以是账户别名、主机别名、命令别名。

```shell
[root@localhost ~]# visudo
......前面省略......
# 这个别名名称一定要使用大写字符来处理
User_Alias ADPWD = user1, user2, user3, ..., user15
Cmnd_Alias ADMPWCOM = !/usr/bin/passwd, /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd root
# Host_Alias     FILESERVERS = fs1, fs2
ADMPW All=(root) ADMPWCOM
......下面省略......
```



**5. sudo的时间间隔问题**

如果我使用同一个账号在短时间内重复操作sudo来运行命令的话，在第二次执行sudo时，并不需要输入密码也能正确执行。只要两次执行sudo的时间间隔在5分钟内，就无需再次输入密码。如果两次执行sudo的间隔时间超过5分钟就要输入密码。



**6. sudo搭配su的使用方式**

很多时候需要大量执行很多root的工作，所以一直用sudo觉得很烦。那有没有办法使用sudo搭配su，一口气将身份转为root呢？而且还是用自己的密码。有，这个方法很简单：

```shell
[root@localhost ~]# visudo
User_Alias ADMINS = user1, user2, user3
AMINDS ALL=(ALL)	/bin/su -
# 这样设置后，这三个用户只需执行sudo su -   这个命令就可以通过输入自己的密码使用root身份进行执行命令了
```





## Linux主机上的用户信息传递

### 查询用户w、who、last、lastlog

如何知道目前已登录在系统上面的用户？可以通过`w`或`who`命令来查询。

```shell
[root@localhost ~]# w
 21:06:13 up 6 days,  5:39,  5 users,  load average: 1.56, 1.66, 1.69
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    10.10.71.30      23Sep21  5days 16:29m  1:16  java -Djava.security.egd=file:/dev/./urandom -
root     pts/1    10.10.71.42      Mon16    2days  0.03s  0.03s -bash
root     pts/2    10.10.40.74      19:47    5.00s  0.12s  0.04s w
root     pts/3    10.10.71.102     23Sep21  3days  0.04s  0.04s -bash
root     pts/4    10.10.71.1       21:05   44.00s  0.02s  0.02s -bash
# 第一行显示目前的时间、启动（up）多久，几个使用者在系统上平均负载
# 第二行只是各个选项说明
# 第三行以后，每行代表一个使用者

[root@localhost ~]# who
root     pts/0        2021-09-23 15:30 (10.10.71.30)
root     pts/1        2021-09-27 16:08 (10.10.71.42)
root     pts/2        2021-09-29 19:47 (10.10.40.74)
root     pts/3        2021-09-23 15:55 (10.10.71.102)
root     pts/4        2021-09-29 21:05 (10.10.71.1)
```



若想知道每个账号最近登录的时间，可以使用`lastlog`命令，这个命令会去读取`/var/log/lastlog`文件，并将数据输出：

```shell
[root@localhost ~]# lastlog -u root   # 仅显示root最近一次的登录信息

[root@localhost ~]# lastlog   # 显示所有
Username         Port     From             Latest
root             pts/4    10.10.71.1       Wed Sep 29 21:05:29 +0800 2021
bin                                        **Never logged in**
daemon                                     **Never logged in**
adm                                        **Never logged in**
lp                                         **Never logged in**
sync                                       **Never logged in**
shutdown                                   **Never logged in**
halt                                       **Never logged in**
mail                                       **Never logged in**
operator                                   **Never logged in**
games                                      **Never logged in**
ftp                                        **Never logged in**
nobody                                     **Never logged in**
systemd-network                            **Never logged in**
dbus                                       **Never logged in**
polkitd                                    **Never logged in**
sshd                                       **Never logged in**
postfix                                    **Never logged in**
chrony                                     **Never logged in**
jw               pts/2                     Fri Aug 20 17:12:25 +0800 2021
tss                                        **Never logged in**
ntp
```



`last`命令可以查询用户到底啥时候登录？

```shell
[root@localhost ~]# last
root     pts/4        10.10.71.1       Wed Sep 29 21:05   still logged in   
root     pts/2        10.10.40.74      Wed Sep 29 19:47   still logged in   
root     pts/2        10.10.40.74      Wed Sep 29 19:38 - 19:47  (00:08)    
root     pts/2        10.10.40.40      Tue Sep 28 19:26 - 23:57  (04:30)    
root     pts/1        10.10.71.42      Mon Sep 27 16:08   still logged in   
root     pts/5        10.10.71.72      Mon Sep 27 10:02 - 10:02  (00:00)    
root     pts/2        10.10.71.72      Mon Sep 27 10:02 - 10:02  (00:00)    
root     pts/4        10.10.71.50      Mon Sep 27 09:23 - 17:08 (1+07:45)   
root     pts/2        10.10.71.50      Sun Sep 26 11:56 - 09:27  (21:30)    
root     pts/1        10.10.71.1       Sun Sep 26 11:56 - 12:19 (1+00:23)   
root     pts/2        10.10.40.149     Fri Sep 24 14:41 - 18:20  (03:38)    
root     pts/2        10.10.40.149     Fri Sep 24 09:14 - 14:19  (05:04)    
root     pts/3        10.10.71.102     Thu Sep 23 15:55   still logged in   
root     pts/2        10.10.40.224     Thu Sep 23 15:37 - 18:00  (02:23)    
root     pts/1        10.10.71.50      Thu Sep 23 15:31 - 11:31 (2+19:59)
```



### 用户交谈：write、mesg、wall

`write`仅针对对单用户发送消息。假设root要跟user1讲话，可以这样做：

```shell
[root@localhost ~]# write 使用者账号 [使用者所在终端界面]
[root@localhost ~]# who
root     pts/0        2021-09-23 15:30 (10.10.71.30)
root     pts/1        2021-09-27 16:08 (10.10.71.42)
root     pts/2        2021-09-29 19:47 (10.10.40.74)
root     pts/3        2021-09-23 15:55 (10.10.71.102)
user1     pts/4        2021-09-29 21:05 (10.10.71.1)

# 可以给在终端pts/4的root发消息
[root@localhost ~]# write user1 pts/4	# 下一行开始输入消息内容，遇到[Ctrl]+d结束输入
hey guys.
I'm a little dog.
# 按[Ctrl]+d 结束，此时user1的画面会不断出现发送者的输入内容，知道发送者取消发送或者user1主动关闭
Message from root@localhost.localdomain on pts/2 at 21:23 ...
hey guys.
I'm a little dog.
EOF

# 如果root pts/4当时在查数据的话，这些信息会中断当前的任务


# 若user1不想接收任何人的信息可以执行命令mesg n，不过mesg的功能对root发来的消息无效
[root@localhost ~]# mesg n

# 开启消息接收
[root@localhost ~]# mesg y

# 查看mesg的状态
[root@localhost ~]# mesg
is y
```



`wall`命令可以广播消息

```shell
# 所有人都会收到该条广播消息
[root@localhost ~]# wall "System Broadcast"
```





## CentOS7环境下大量创建账号的方法

`chpasswd`命令可以用来修改用户密码，类似`passwd --stdin username`。命令格式为【username:password】。
例如将myuser的密码设置为password@123，则命令为`[root@localhost ~]# echo "myuser:password@123" | chpasswd`

大批量创建账号可采用如下两个思路进行书写shell脚本创建。

- 思路1：通过usradd usrename；echo "username:passwordstr" | chpasswd；书写shell脚本。

- 思路2：通过usradd usrename；echo passwordstr | passwd --stdin username；书写shell脚本。

