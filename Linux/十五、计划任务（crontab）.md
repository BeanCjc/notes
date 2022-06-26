# 十五、计划任务（crontab）

Linux计划任务分为两种：一次性计划任务（at）和周期性计划任务（crontab）。

- 一次性计划任务（at）：at是个可以处理仅执行一次性就结束的命令，不过要执行at命令必须要有**atd这个服务**。
- 周期性计划任务（crontab）：crontab这个命令所设置的任务将会循环地一直执行下去，可循环的时间为分钟、小时、天、月、年、周等。除了使用命令外还可以编辑配置文件`/etc/crontab`来支持。同样，crontab需要**crond这个服务**。

无论是at还是crontab，执行任务都是在整分钟时间，整分钟时间指的是：当时间的秒为0的时刻。



## 仅执行一次的计划任务（at）

使用at命令来产生所要运行的任务，并将该任务以文本文件的方式写入`/var/spool/at`目录内，该任务便能能带atd服务的使用与执行了。

出于安全考虑，并不是所有的用户都能执行at计划任务。规则如下：

- 寻找`/etc/at.allow`文件，写在这个文件中的用户才能使用at
- 如果`/etc/at.allow`文件不存在，则寻找`/etc/at.deny`，在该文件中的用户不能使用at，而没有在`/etc/at.deny`文件中的用户就可以使用at
- 若以上两个配置文件都不存在，那么只有root才能使用at命令。如果两个文件都存在，优先按照`/etc/at.allow`文件的配置来

at命令的使用很简单，只要at 加上一个时间即可。

```shell
[root@localhost ~]# at [-mldv] TIME
[root@localhost ~]# at -c 任务号码
选项与参数：
-m：当at的任务完成后，即使没有输出信息，亦发email通知使用者该任务已完成
-l：at -l相当于atq，列出目前系统上面的所有该使用者的at计划任务
-d：at -d相当于atrm，可以取消一个在at计划中的任务
-v：可以使用较明显的时间格式列出at计划中的任务表
-c：可以列出后面的该项任务的实际命令内容
TIME：时间格式，格式有：
	HH:MM	在今日的HH:MM时刻执行，若该时刻已超过，则明天的HH:MM时刻执行。例如04:00
	HH:MM YYYY-MM-DD	强制规定在具体的某一天的某一时刻执行
	HH:MM[am|pm] [Month] [Date]	也是一样强制在具体的某一天的某一时刻执行
	HH:MM[am|pm]+number [minutes|hours|days|weeks]	在某个时间点再加上几个时间后才执行，例如now+5 minutes

# 再过5分钟后将/root/.bashrc 发给root自己
[root@localhost ~]# at now + 5 minutes 
at> /bin/mail -s "testing at job" root < /root/.bashrc
at> <EOF>

# 将上述的任务内容列出来看
[root@localhost ~]# at -c 2    # 2是上一步创建返回的一个id
# 输出暂时不显示
```

当我们使用at时会进入一个at shell的环境来让用户执行任务命令。此时最好使用**绝对路径来执行命令**，避免出现问题。



```shell
# 查询
[root@localhost ~]# atq
# 输出暂时不显示

# 移除
[root@localhost ~]# atrm jobnumber
[root@localhost ~]# atrm 2
```



如果服务器繁忙，可使用**`batch`**命令让计划任务在闲时执行计划任务。batch可以在CPU的任务负载小于0.8的时候才执行任务。任务负载指的是：CPU在单一时间点所负责的任务数量，而不是CPU的使用率。举例来说，如果有一个程序需要一直使用CPU的运算能力，那么此时CPU的使用率可能达100%，但是CPU的任务负载则是趋近于1，因为CPU仅负责一个任务；若同时执行两个这样的任务呢？CPU的使用率还是100%，但任务负载则变成了2。大概就是这个意思。可以使用uptime查看cpu使用情况。**因为`batch`命令是在闲时才执行，所以没有办法设置指定时间**。

```shell
[root@localhost ~]# batch
at> /usr/bin/updatedb
at> <EOF>
```





##  循环执行的计划任务（crontab）

同样的出于安全考虑，并不是所有的用户都能执行crontab计划任务。规则如下：

- 寻找`/etc/cron.allow`文件，写在这个文件中的用户才能使用at
- 如果`/etc/cron.allow`文件不存在，则寻找`/etc/cron.deny`，在该文件中的用户不能使用crontab，而没有在`/etc/cron.deny`文件中的用户就可以使用crontab
- 若以上两个配置文件都不存在，那么只有root才能使用crontab命令。如果两个文件都存在，优先按照`/etc/cron.allow`文件的配置来

当用户使用crontab命令来建立计划任务后，该项任务会被记录到`/var/spool/cron/`中，而且是以账号来作为判断依据的。记录来说，如果是user1使用crontab后，它的任务会被记录到`/var/spool/cron/user1`中。

另外cron的执行记录都会被记录到`/var/log/cron`这个日志文件中。

```shell
[root@localhost ~]# crontab [-u username] [-l|-e|-r]
选项与参数：
-u：只有root才能执行这个任务，亦即帮其他用户建立/删除crontab计划任务
-l：查看crontab的任务内容
-e：编辑crontab的任务内容
-r：删除所有的crontab的任务内容，若仅要删除一项，请使用-e去编辑删除对应的行即可

# 用user1的身份在没有12：00发信给自己
[user1@localhost ~]$ crontab -e
# 此时会进入vi的编辑界面让您编辑任务，每一行都是一个任务
0  12 * * * mail -s "title" user1 < /home/user1/.bashrc
#分 时 天 月 周 命令串
#0~59 0~23 1~31 1~12 0~7（0或7都是星期天的意思）
```

同样命令执行建议使用绝对路径来执行



### 系统的配置文件：/etc/crontab、/etc/cron.d/*

crond服务每分钟会去读取一次配置文件`/etc/crontab`、`/etc/cron.d/*`和`/var/spool/cron/*`。因为这个的原因这三个是最重要的配置文件，其他配置文件都是从这三个配置文件的内容衍生出来的。



#### /etc/crontab

```shell
[root@localhost ~]# cat /etc/crontab 
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

`/etc/crontab`该文件默认仅声明一些重要的环境变量。这里面可以设置系统计划任务，格式为【分 时 日 月 周 用户 命令】。



#### /etc/cron.d/*

```shell
# 查看下/etc/cron.d/目录下有什么文件
[root@localhost ~]# ll /etc/cron.d/
total 8
-rw-r--r--. 1 root root 128 Aug  9  2019 0hourly
-rw-------. 1 root root 235 Apr  1  2020 sysstat

# 查看/etc/cron.d/0hourly文件内容
[root@localhost ~]# cat /etc/cron.d/0hourly 
# Run the hourly jobs
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
01 * * * * root run-parts /etc/cron.hourly

# 查看/etc/cron.hourly/目录下有什么可执行文件
[root@localhost ~]# ll /etc/cron.hourly/
total 4
-rwxr-xr-x. 1 root root 392 Aug  9  2019 0anacron

[root@localhost ~]# cat /etc/cron.hourly/0anacron 
#!/bin/sh
# Check whether 0anacron was run today already
if test -r /var/spool/anacron/cron.daily; then
    day=`cat /var/spool/anacron/cron.daily`
fi
if [ `date +%Y%m%d` = "$day" ]; then
    exit 0;
fi

# Do not run jobs when on battery power
if test -x /usr/bin/on_ac_power; then
    /usr/bin/on_ac_power >/dev/null 2>&1
    if test $? -eq 1; then
    exit 0
    fi
fi
/usr/sbin/anacron -s

# 发现该程序最终会调用anacron，anacron脚本会使用配置文件/etc/anacrontab
[root@localhost ~]# cat /etc/anacrontab 
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# the maximal random delay added to the base delay of the jobs
RANDOM_DELAY=45
# the jobs will be started during the following hours only
START_HOURS_RANGE=3-22

#period in days   delay in minutes   job-identifier   command
1	5	cron.daily		nice run-parts /etc/cron.daily
7	25	cron.weekly		nice run-parts /etc/cron.weekly
@monthly 45	cron.monthly		nice run-parts /etc/cron.monthly

# /etc/anacrontab配置文件定义了每天每周每月的计划任务
```

发现`/etc/cron.d/0hourly `该配置文件中定义了一个计划任务，该计划任务会每个小时执行一次，同时会去/etc/cron.hourly/目录下查找所有的可执行文件。其中run-parts命令会在大约5分钟内随机选一个时间来执行/etc/cron.hourly目录内的所有可执行文件。因此，放在/etc/cron.hourly目录下的文件都是可执行文件，而不是分、时、日、月、周的设置值。

`/etc/cron.hourly/0anacron`程序每一个小时执行一次，**也就是anacron每个小时执行一次**，执行时回去读取配置文件`/etc/anacrontab`，该文件声明了每天、每周、每月的计划任务。

anacron时通过当前时间与配置文件`/var/spool/anacron/cron.daily`、`/var/spool/anacron/cron.weekly`、`/var/spool/anacron/cron.monthly`当中记录的时间做对比来判断是否执行过任务。



**综上**，cron的调用链文件是这样的：**/etc/crontab（配置文件）**、**/etc/cron.d/*（该目录下的所有配置文件）**、**/var/spool/cron/*（该目录下的所有配置文件）****、**/etc/cron.d/0hourly**、**/etc/cron.hourly/0anacron**、**/etc/anacrontab**、**/etc/cron.daily/***、**/etc/cron.weekly/***、**/etc/cron.monthly/***以及记录时间戳的配置**/var/spool/anacron/cron.daily**、**/var/spool/anacron/cron.weekly**、**/var/spool/anacron/cron.monthly**



所以，关于计划任务的可执行脚本放置目录的建议方式为：

**/etc/crontab**：配置需要自定义执行计划的自定义系统计划任务

**/etc/cron.hourly/***：需要每小时执行的自定义系统计划任务

**/etc/cron.daily/***：需要每天执行的自定义系统计划任务

**/etc/cron.weekly/***：需要每周执行的自定义系统计划任务

**/etc/cron.monthly/***：需要每月执行的自定义系统计划任务

**/var/spool/cron/***：用户自定义的执行计划任务声明，可执行脚本可放置到各个用户的home目录下



- 配置文件：`/etc/crontab`、`/etc/cron.d/*`、`/etc/anacrontab`、`/var/spool/cron/*`、`/etc/cron.allow`、`/etc/cron.deny`
- 可执行目录任务目录：`/etc/cron.hourly/`、`/etc/cron.daily/`、`/etc/cron.weekly/`、`/etc/cron.monthly/`



anacron与crontab的区别在于，anacron如果在计划的时间内由于主机异常（例如停掉导致不在线）等未执行计划任务，待之后主机正常之后依然可以执行前面没有执行的任务。而crantab不行，一旦执行时间点过了，就不再执行该任务。