# 十四、磁盘配额（Quota）与高级文件系统管理

如果您的Linux服务器有多个用户经常存取数据，为了维护所有用户在使用硬盘时的公平，磁盘配额就是一款非常有用的工具。另外，如果你的用户经常抱怨磁盘容量不够用，那么就得要学习更高级的文件系统。下面会介绍磁盘阵列（RAID）及逻辑卷管理器（LVM），这些工具都可以帮助你管理与维护用户可用的磁盘容量。





## 磁盘配额

### 概念

磁盘配额就是限制使用磁盘容量。磁盘配额的限制情况分为以下三种：

- 限制某一用户组所能使用的最大磁盘配额
- 限制某一用户的最大磁盘配额
- 限制某一目录的磁盘配额

在旧版CentOS种，使用的默认文件系统时ext系列，这种文件系统的磁盘配额主要是针对整个文件系统来处理，所以大多针对挂载点进行设计。新的xfs可以使用project这种模式，就能够针对个别的目录（非文件系统）来设计磁盘配额。

磁盘配额的使用限制：

- ext文件系统仅能针对整个文件系统，无法针对个别目录
- 内核必须支持磁盘配额，CentOS7默认支持
- **只对一般身份用户有限**，对root无效，所有的都是root的所以多root进行限额没意义
- 若启用SELinux，非所有目录均可设置磁盘配额

新版CentOS默认都启用SELinux这个内核功能，该功能会加强某些特殊的权限控制。由于担心管理员不小心设置错误，因此默认的情况下，磁盘配额似乎仅能针对/home进行设置而已，因此，如果你要针对其他不同的目录进行设置，需要关闭SELinux。

- **用户组（group）与目录（project）的限制无法同时存在**

- 磁盘配额的规范设置选项。以下针对xfs文件系统的限制选项进行说明：

  - 分别针对用户、用户组或个别目录（user、group和project）
  - 容量限制或文件数量限制（block或inode）
  - 限制inode使用量：管理用户可以建立的文件数量
  - 限制block使用量：**管理用户磁盘容量的限制，常用方案**
  - 软限制与应限制（soft/hard）

  不管是inode还是block，限制值都有两个，分别是soft和hard，通常hard限制值要比soft高。举例来说，若选项为block，可以限制hard为500MB，soft为400MB。

  **hard**：表示用户的使用量绝对不会超过这个值，以上面这个例子，用户所使用的磁盘容量绝对不会超过500MB，若超过这个值则系统会锁定该用户的磁盘使用权。

  **soft**：表示用户在低于soft时（此例中为400MB），可以正常使用磁盘，但若超过了soft但低于hard（400MB~500MB之间），每次用户登录系统时，系统会主动发出磁盘容量即将耗尽的警告信息，且会给予一个宽限时间（grace time）。不过，若用户在宽限时间内将容量再次降到soft值之下，则宽限时间会停止。

  **宽限时间（grace time）**：当用户使用的磁盘容量达到soft与hard之间时（例子中的400MB~500MB之间），系统会给予警告，但也会给一段时间让用户自行管理磁盘。一般默认是7天，如果7天内没有进行磁盘管理，那么soft限制值会即刻替换hard值限制值来作为磁盘配额的配置，也就是说如果宽限时间内没有进行磁盘管理（扩容、降低磁盘使用量、提高配额等），那么你的磁盘使用权就会被锁定而无法新增文件了。





### 范例实践

这里使用一个范例来设计以下如何处理磁盘配额的设置流程

- 目的与账号：现在我想要让五个用户为一组，这五个人的账号分别是user1、user2、user3、user4、user5，这五个用户的密码都是password，且这五个用户所属的初始用户组都是myquotagroup，其他的账号属性使用默认值
- 账号的磁盘容量限制值：这五个用户都能够获取300MB的磁盘使用量（hard），文件数量不限制。此外，只要容量使用超过250MB，就予以警告（soft）
- 用户组的配额（option1）：由于我的系统里还有其他用户存在，因此我仅承认myquotagroup这个用户组最多使用1GB。也就是说如果user1、user2、user3都使用了280MB的容量，那么其他两个人最多只能使用（1000-280*3=160MB）的磁盘容量，这就是用户与用户组同时设置时会产生的效果
- 共享目录配额（option2）：另一种设置方式。每个用户还是具有自己独立的容量限制，但是这五个人的共享目录在`/homg/myquota`这里。该目录设置为其他人没有任何权限的共享目录空间，仅有myquotagroup用户组拥有全部权限。且无论如何该目录最多使用磁盘容量为500MB。**注意，用户组与目录的限制无法同时存在**
- 宽限时间：最后，我希望每个用户在超过soft限制值后，都还能够有14天的宽限时间

建立账号和环境

```shell
# 使用脚本来建立账号和所需环境
[root@localhost ~]# vim addaccount.sh
#!/bin/bash
groupadd myquotagroup
for username in user1 user2 user3 user4 user5
do
	useradd -g myquotegroup ${username}
	echo "password" | passwd --stdin passwd ${username}
done
mkdir /home/myquota
chmod 2770 /home/myquota
[root@localhost ~]# chmod a+x ./addaccount.sh
[root@localhost ~]# ./addaccount.sh
```





### 磁盘配额流程范例

#### 文件系统的支持与查看

磁盘配额需要内核与文件系统支持才行，CengOS7内核默认支持磁盘配额。此外不要再根目录下进行磁盘配额设置，这会让文件系统变得太复杂。下面以`/home`这个xfs文件系统为例。

```shell
[root@localhost ~]# df -dT /home
Filesystem	Type	Size	Used	Avail	Use%	Mounted on
/dev/mapper/centos-home  xfs	5.0G	67M		5.0G	2%   /home
# 可以看出/home目录确实是独立的文件系统，且是xfs文件系统。
```

xfs文件系统启动磁盘配额功能一般在挂载之初就声明了，因此无法使用remount来重新启动磁盘配额的功能，一定要写入`/etc/fstab`文件中，或是在初始挂载过程中加入这个选项，否则不会生效。下面来看看如何修改fatab吧！

```shell
[root@localhost ~]# vim /etc/fstab
/dev/mapper/centos-home  /home	xfs	defaults,usrquota,grpquota	0 0
# 其他挂载信息不列出来了
[root@localhost ~]# umount /home
[root@localhost ~]# mount -a
[root@localhost ~]# mount | grep home
/dev/mapper/cntos-home on /home type xfs
(rw, relatime, seclabel,attr2,inode64,usrquota,grpquota)
```

基本上，针对磁盘配额限制的选项主要有三项：

- uquota/usrquota/quota：针对用户账号的设置
- gquota/grpquota：针对用户组的设置
- pquota/prjquota：针对单一目录的设置，但不可与gquota同时存在



#### 查看磁盘配额报告数据

使用`xfs_quota`命令操作磁盘配额。

```shell
[root@localhost ~]# xfs_quota -x -c "命令" 挂载点
选项与参数：
-x：专家模式，后续才能够加入-c的命令参数
-c：后面加的就是命令，这个小节线来谈谈数据报告的命令
命令：
	print：单纯地列出目前主机内的文件系统参数等数据
	df：与原本的df一样的功能，可以加上-b（block）、-i（inode）、-h（加上单位）等
	report：列出目前的磁盘配额选项，有ugp（user/group/project）及-bi等
	state：说明目前支持磁盘配额的文件系统信息，有没有使用相关选项等
	
# 列出目前系统各个文件系统，以及文件系统的磁盘配额挂载参数支持
[root@localhost ~]# xfs_quota -x -c "print"
# 输出暂时不显示

# 列出目前/home这个支持磁盘配额的挂载点文件系统使用情况
[root@localhost ~]# xfs_quota -x -c "df -h" /home
# 输出暂时不显示

# 列出目前/home的所有用户的磁盘配额限制值
[root@localhost ~]# xfs_quota -x -c "report -ubih" /home
# 输出暂时不显示
# soft/hard若为0，代表没限制

# 列出目前支持的磁盘配额文件系统是否有启动磁盘配额功能
[root@localhost ~]# xfs_quota -x -c "state" /home
# 输出暂时不显示
```



#### 限制值设置方式（针对用户和用户组）

目标：

- 用户和用户组限制
- 每个用户250MB/300MB的容量限制
- 用户组950MB/1GB的容量限制
- 设置grace time为14天

```shell
[root@localhost ~]# xfs-quota -x -c "limit [-ug] b[soft|hard]=N i[soft|hard]=N username|groupname"
[root@localhost ~]# xfs_quota -x -c "timer [-ug] [-bi] Ndays"
选项与参数：
limit：实际限制的选项，可以针对user/group来限制，限制的选项有以下内容：
	bsoft/bhard：block的soft/hard限制值，可以加单位
	isoft/ihard：inode的soft/hard限制值，可以加单位
	username|groupname：就是用户或者用户组
timer：用来设置grace time的选项，也是可以针对user/group以及block/inode设置
# 设置用户（user1、user2、user3、user4、user5）的block限制值（本示例没有限制inode）
[root@localhost ~]# xfs_quota -x -c "limit -u bsoft=250M bhard=300M user1" /home
[root@localhost ~]# xfs_quota -x -c "limit -u bsoft=250M bhard=300M user2" /home
[root@localhost ~]# xfs_quota -x -c "limit -u bsoft=250M bhard=300M user3" /home
[root@localhost ~]# xfs_quota -x -c "limit -u bsoft=250M bhard=300M user4" /home
[root@localhost ~]# xfs_quota -x -c "limit -u bsoft=250M bhard=300M user5" /home
[root@localhost ~]# xfs_quota -x -c "report -ubih"
# 输出暂时不显示

# 设置myquotagroup的限制值
[root@localhost ~]# xfs_quota -x -c "limit -g bsoft=950MB bhard=1G myquotagroup" /home

# 设置grace time为14天
[root@localhost ~]# xfs_quota -x -c "timer -ug -b 14" /home

[root@localhost ~]# xfs_quota -x -c "state" /home
# 输出暂时不显示
```



#### 限制值设置方式（针对用户和目录）

因为group和project不可同时存在，故先取消group设置，并加入project设置。

```shell
[root@localhost ~]# vim /etc/fstab
/dev/mapper/centos-home  /home	xfs	defaults,usrquota,prjquota	0 0
[root@localhost ~]# umount /home
[root@localhost ~]# mount -a

# 查看已启动磁盘配额功能及选项信息
[root@localhost ~]# xfs_quota -x -c "state" /home
# 输出暂时不显示
```

目录的设置有一定的约定：它必须指定一个选项名称（name）和选项标识符（ID）来规范才行，两者都可自定义；还需要将这两个信息写入到指定的两个配置文件中`/etc/projects`、`/etc/projid`。我们现在要规范的目录是`/home/myquota`，选项名称定为myquotaproject，选项标识符定为112233，如下配置：

```shell
# 1.指定选项标识符（ID）与目录的对应在配置文件/etc/projects
[root@localhost ~]# echo "112233:/home/myquota" >> /etc/projects
# 2.规范方案名称（name）与标识符（ID）在配置文件/etc/projid
[root@localhost ~]# echo "myqoutaproject:112233" >> /etc/projid
# 3.初始化方案名称
[root@localhost ~]# xfs_quota -x -c "project -s myquotaproject"
# 输出暂时不显示

[root@localhost ~]# xfs_quota -x -c "print"
# 输出暂时不显示
[root@localhost ~]# xfs_quota -x -c "report -pbih" /home
# 输出暂时不显示

# 方案已经有了，下面就开始配置方案的限制值
[root@localhost ~]# xfs_quota -x -c "limit -p bsoft=950M bhard=1G myquotaproject" /home
[root@localhost ~]# xfs_quota -x -c "report -pbih" /home
# 输出暂时不显示
```



### 磁盘配额的管理与额外命令对照表

如果需要暂停磁盘配额限制，或者需要重新启动磁盘配额的限制时，该怎么做？还是使用xfs_quota`命令即可。

- disable：暂时取消磁盘配额的限制，但其实系统还是在计算磁盘配额中，只是没有管制而已，应该算是最有用的功能
- enable：就是恢复到正常管制的状态中，与disable可与相互取消、启用
- off：完全关闭磁盘配额的限制，使用这个状态之后，你**只有卸载再重新挂载才能够再次启动磁盘配额**。也就是说，用了off状态后，你无法使用enable再次恢复磁盘配额的管制。注意，这个状态不要乱用，一般建议用disable即可，除非你需要执行remove的操作
- remove：必须再off的状态下才能够执行的命令，这个remove可以删除磁盘配额的限制设置。例如要取消project的设置，无需重新设置为0，只要remove -p即可

```shell
# 暂时关闭xfs文件系统的磁盘配额限制功能
[root@localhost ~]# xfs_quota -x -c "disable -up" /home

[root@localhost ~]# xfs_quota -x -c "state" /home

# 重新启动磁盘配额限制功能
[root@localhost ~]# xfs_quota -x -c "enable -up"

# 完全关闭磁盘配额的限制功能
[root@localhost ~]# xfs_quota -x -c "off -up" /home
[root@localhost ~]# xfs_quota -x -c "enable -up" /home
XFS_QUOTAON: Function not inplemented
# 这个时候是无法用enable再次启动磁盘配额限制功能

# 需要重新卸载并挂载才可恢复
[root@localhost ~]# umount /home
[root@localhost ~]# mount -a

[root@localhost ~]# xfs_quota -x -c "off -up" /home
# 仅移除目录的限制
[root@localhost ~]# xfs_quota -x -c "remove -p"
[root@localhost ~]# umount /home;mount -a
[root@localhost ~]# xfs_quota -x -c "report -pbh" /home
# 输出暂时不显示
```



| 设置流程选项           | xfs文件系统                                      | ext系列文件系统   |
| ---------------------- | ------------------------------------------------ | ----------------- |
| /etc/fstab参数设置     | usrquota/grpquota/prjquota                       | usrquota/grpquota |
| 磁盘配额配置文件       | 不需要                                           | quotacheck        |
| 设置用户/用户组限制值  | xfs_quota -x -c "limit -[u\|g] ..."              | edquota或setquota |
| 设置grace time         | xfs_quota -x -c "timer -[u\|g] -[bi] Ndays"      | edquota           |
| 设置目录限制值         | xfs_quota -x -c "limit -p..."                    | 无                |
| 查看报告               | xfs_quota -x -c "report ..."                     | repquota或quota   |
| 启动与关闭磁盘配额限制 | xfs_quota -x -c "[disable\|enable\|off\|remove]" | quotaoff或quotaon |
| 发送警告信息给用户     | 目前版本尚未支持                                 | warnquota         |





## 软件磁盘阵列（Software RAID）

磁盘阵列全名是【Reduandant Arrays Of Inexpensive Disks，RAID】，中文意思是独立冗余磁盘阵列。RAID通过技术（硬件或者软件）将多个较小的磁盘整合成为一个较大的磁盘设备，而这个大的磁盘功能可不止是存储而已，它还具有数据保护功能。整个RAID由于选择的级别不同，而使得整合后的磁盘具有不同的功能，基本常见的level有：RAID 0、RAID 1、RAID 1+0、RAID 0+1、RAID 5以及RAID 6。



### 磁盘阵列级别：



#### RAID 0（等量模式，stripe）：性能最佳

这种模式如果使用相同型号与容量的磁盘来组成时，效果较佳。这种模式的RAID会将磁盘先且分出等量的数据块（名为chunk，一般可设置为4KB~1MB），然后当一个文件要写入RAID时，该文件会根据chunk的大小切割好，之后再依序放到各个磁盘里面去。由于每个磁盘会交错地存放数据，因此**当你的数据要写入RAID时，数据会被等量地放置在各个磁盘上面**。举例来说，你有两块磁盘组成的RAID 0，当你有100MB的数据要写入，每个磁盘会被个分配到50MB的存储量。所以按照这样的分配方式，越多块的磁盘组成的RAID 0性能会越好，因为每块负责的数据量就耕低了。此外磁盘总量也变大了，**磁盘总量等于所有磁盘的总和**。很明显，这种模式如果某一块磁盘损坏了，那么文件数据将缺一块，此时文件就损坏了。由于每个文件都是这样存放的，因此**RAID 0只要有任何一块磁盘损坏，在RAID上面的所有数据都会遗失**而无法读取。



RAID 1（镜像模式，mirror）：完整备份

这种模式也需要相同的磁盘容量，最好时一摸一样的磁盘。**如果是不同容量的磁盘组成的RAID 1时，那么总容量将以最小的那一块磁盘为主。这种模式主要是让一份数据，完整地保存在两块磁盘上面**。举例来说，如果有一个100MB的文件，且我仅有两块磁盘组成RAID 1时，那么这两块磁盘将会写入100MB到它们各自的存储空间中。因此，整体RAID的容量几乎少了50%。由于两块磁盘内容一模一样，好像镜子映照出来的一样，所以也叫它镜像模式。RAID 1的最大优点大概就在于数据的备份。不过由于磁盘容量有一半用在备份，因此总容量会是全部磁盘容量的一半。



#### RAID 1+0，RAID 0+1

RAID 0的性能佳，但是数据不安全，RAID 1的数据安全但是性能不佳，那么能不能将这两者整合起来设置RAID呢？可以，那就是RAID 1+0或RAID 0+1。所谓RAID 1+0就是：①先让两块磁盘组成RAID 1，并且这样的设置共两组；②将两组RAID 1再组成一组RAID 0，这就是RAID 1+0.反过来说，RAID 0+1就是先组成RAID 0再组成RAID 1的意思。RAID 1+0是目前存储设备厂商最推荐的方法。

为何会推荐RAID 1+0呢？假设有20块磁盘组成的系统。每两块组成一个RAID 1，因此就有总共10组可以自己恢复的系统了。然后这10组再组成一个新的RAID 0，速度上立刻提升了10倍。同时要注意，因为RAID 1是独立存在的，所以任何一块磁盘损坏，数据都是从另一块磁盘直接复制过来重建，并不像RAID 5/RAID 6必须要整组RAID的磁盘共同重建一块独立的磁盘系统，性能上差得非常多。而且RAID 0与RAID 1是不需要经过计算得（striping），读写性能也比其他得RAID级别好太多了。



#### RAID 5：性能与数据备份得均衡考虑

RAID需要三块以上得磁盘才能够组成这种类型得磁盘阵列。这种磁盘阵列的数据写入有点类似RAID 0，不过每个循环的写入过程中（striping），在每块磁盘还会加入一个奇偶校验数据（Parity），这个数据会记录其他磁盘的备份数据，用于当有磁盘损坏时的恢复。RAID 5读写的情况有点像下面这样：

每个循环写入时，都会有部分的奇偶校验值（parity）被记录下来，并且每次都记录在不同的磁盘，因此，任何一个磁盘损坏时都能够借由其他磁盘的检查码来重建原本磁盘内的数据。不过，需要注意的是，由于有奇偶校验值，因此RAID 5的总量会是整体磁盘数量减一块。相当于原本三块磁盘只会剩下两块磁盘的容量。而且当损坏的磁盘数量大于等于两块时，这整租RAID 5的数据就损坏了，因为RAID 5默认仅能支持一块磁盘损坏的情况。

在读写性能上，RAID 5读取的性能还不赖，与RAID 0有的比，不过写的性能就不见得能够增加很多，因为要写入RAID 5的数据还要经过计算奇偶检验值得关系。由于加上这个计算得操作，所以写入得性能与系统得硬件关系较大。尤其当使用软件磁盘阵列时，奇偶校验值是通过CPU去计算而非专职得磁盘阵列卡，因此性能方面还需评估。

另外，由于RAID 5 仅能支持一块磁盘得损坏，因此近年来又发展出另一种级别，RAID 6。这个RAID 6使用两块磁盘得容量存储奇偶校验值，因此整体得磁盘容量会减少两块，但是允许出错得磁盘数量就可以达到两块了。也就是在RAID 6得情况下，两块磁盘同时损坏时，数据还是可以救回来 的。



#### Spare Disk：热备份磁盘

当磁盘阵列的磁盘损坏时，就需要将坏掉的磁盘拔除，然后换一块新的磁盘。换成新的磁盘并且顺利启动磁盘阵列后，磁盘阵列就会开始主动重建（rebuild）原本坏掉的那块磁盘数据到新的磁盘上，然后你的磁盘阵列上面的数据就恢复了，这就是磁盘阵列的优点。不过，我们还是得手动拔插硬盘，除非你的系统支持热拔插，否则通常得要关机才能这么做。

为了让系统可以实时地在坏掉硬盘时主动地重建，就需要热备份磁盘（spare disk）的辅助。所谓热备份磁盘就是一块或多块没有包含在原本磁盘阵列级别中的磁盘，这块磁盘平时并不会被磁盘阵列所使用，当磁盘阵列有任何磁盘损坏时，这块热备份磁盘就会被主动拉进磁盘阵列中，并将坏掉的那块硬盘移出磁盘阵列，然后立即重建数据系统，如此你的系统就可以永保安康。若你的磁盘阵列支持热拔插就更完美了，直接将坏掉的那块磁盘拔除并换一块新的，再将那块新的的设置成热备份磁盘，就完成了。



### 磁盘阵列的优点

- 数据安全与可靠性：指的是并非网络信息安全，而是当硬件（指磁盘）损坏时，数据还是能够安全地恢复或使用之意
- 读写性能：例如RAID 0可以加强读写性能，让你的系统I/O部分得以改善
- 容量：可以让多块磁盘组合起来，故单一文件系统可以有相当大的容量



### 各种RAID级别的优缺点汇总表

假设有n块磁盘

| 项目           | RAID 0                      | RAID 1     | RAID 1+0           | RAID 5     | RAID 6     |
| -------------- | --------------------------- | ---------- | :----------------- | ---------- | ---------- |
| 最少磁盘数     | 2                           | 2          | 4                  | 3          | 4          |
| 最大容错磁盘数 | 无                          | n-1        | n/2                | 1          | 2          |
| 数据安全性     | 完全没有                    | 最佳       | 最佳               | 较好       | 比RAID 5好 |
| 理论写入性能   | n                           | 1          | n/2                | <n-1       | <n-2       |
| 理论读出性能   | n                           | n          | n                  | <n-1       | <n-2       |
| 可用容量       | n                           | 1          | n/2                | n-1        | n-2        |
| 一般应用       | q强调性能但数据不重要的环境 | 数据与备份 | 服务器、云系统常用 | 数据与备份 | 数据与备份 |

因为RAID 5、RAID 6读写都需要经过计算奇偶校验值，所以读写性能都不会刚好满足于使用的磁盘数量。

另外，根据使用的情况不同，一般推荐的磁盘阵列级别也不一样。在云环境下由于RAID 5和RAID 6的性能较弱方面，一般采用RAID 1+0的方案。在某些特别的环境搭配SSD那才更具有性能上的优势。



### 硬件RAID

磁盘阵列又分为硬件磁盘阵列与软件磁盘阵列。所谓的硬件磁盘阵列（hardware RAID）是通过**磁盘阵列卡**来完成磁盘阵列的功能。磁盘阵列卡上面有一块专门的芯片用于处理RAID的任务，因此在性能方面会比较好。在很多任务（例如RAID 5的奇偶校验值计算）中，磁盘阵列并不会重复消耗原本系统的I/O总线，理论上性能会较佳。此外目前一般的中高级磁盘阵列卡都**支持热拔插**，即在不关机的情况下抽换损坏的磁盘，在系统的恢复与数据的可靠性方面非常好用。

不过一块好的磁盘阵列卡动不动就上千块，便宜的在主板上【附赠】的磁盘阵列卡功能又不支持某些高级功能。此外操作系统也必须拥有磁盘阵列卡的驱动程序，才能够正确地识别到磁盘阵列所产生的磁盘驱动器。



### 软件磁盘阵列

由于硬件磁盘阵列昂贵，因此有发展出了利用软件来模拟磁盘阵列的功能，这就是所谓的软件磁盘阵列（Software RAID）。软件磁盘阵列主要是通过软件来模拟磁盘阵列的任务，因此会损耗较多的系统资源，比如说CPU的运算与I/O总线的资源等。不过目前的计算机已经非常快了，这种损耗可以忽略。

CentOS提供的软件磁盘阵列为mdadm这个软件，这个软件会以**分区**或者**disk**为单位，也就是说，你不需要两块以上的磁盘，只有有两个以上的硬盘分区（partition）就能够设计你的磁盘阵列了。此外，mdadm支持前面提到的RAID 0、RAID 1、RAID 5、热备份磁盘等。而且提供的管理机制还可以达到类似热拔插的功能，可以在线（文件系统正常使用）进行分区的抽换，使用上也非常方便。

另外，必须知道的是，**硬件磁盘阵列**在Linux下面看起来就是一块实际的大磁盘，因此硬件磁盘阵列的设备文件名为**`/dev/sd[a-p]`**，因为使用到SCSI的模块之故。至于**软件磁盘阵列**则是系统模拟的，因此使用的设备文件名是系统的设备文件，文件名为**`/dev/md0`**、**`/dev/md1`**等，两者的设备文件名并不相同，不要搞混了。



### 软件磁盘阵列的设置

软件磁盘阵列使用`mdadm`命令

```shell
[root@localhost ~]# mdadm --detail /dev/md0
[root@localhost ~]# mdadm --craete -dev/md[0-9] --auto --level=[015] --chunk=NK --raid-devices=N --spare-devices=N /dev/sdx /dev/hdx...
选项与参数：
--create：为建立RAID的选项
--auto：决定建立后面接的软件磁盘阵列设备，亦即/dev/md0、/dev/md1等
--chunk：决定这个设备的chunk大小，也可以当成stripe大小，一般是64K或512K
--raid-devices：使用几个磁盘分区（partition）作为磁盘阵列的设备
--spare-devices：使用几个磁盘作为备用（spare）的设备
--level=[015]：设置这组磁盘阵列的级别，支持很多，不过建议只要用0、1、5即可
--detail：后面所接的那个磁盘阵列设备的详细信息
# 最后面会接许多的设备文件名，这些设备文件名可以是正块磁盘，也可以是分区，不过这些设备名的总数必须要等于--raid-devices与--spare-devices的个数总和才行。
```



举例操作磁盘阵列

目标：

- 利用4个分区组成的RAID 5
- 每个分区约为1GB大小，需确定每个分区一样大较佳
- 将1个分区设为热备份磁盘
- chunk设置为256KB这么大即可
- 这个热备份磁盘的大小与其他RAID所需分区一样大
- 将此RAID 5设备挂载到`/srv/raid`目录下



创建磁盘阵列：

```shell
[root@localhost ~]# gdisk /dev/vds
# 输出暂时不显示

# 以mdadm来创建RAID
[root@localhost ~]# mdadm --create /dev/md0 --auto=yes --level=5 --chunk=256K --raid-devices=5 --spare-device=1 /dev/vda{5,6,7,8,9}
# 输出暂时不显示

# 查看磁盘阵列信息
[root@localhost ~]# mdadm --detail /dev/md0
# 输出暂时不显示
# 创建需要一些时间，最好等几分钟再用mdadm --detail /dev/md0 ，否则可能会看到某些磁盘正在spare rebuilding之类的创建字样

# 可以查看如下的文件来看看软件磁盘阵列的情况
[root@localhost ~]# cat /proc/mdstat
# 输出暂时不显示

# 格式化RAID，格式化时需要注意参数，stripe（chunk）的容量为256KB，所以su=256K
# 共有4块组成的RAID 5，因此容量少一块，所以sw=3
# 由上面两项计算出数据宽度为：256K*3=768K
# 所以格式化就变成这样：
[root@localhost ~]# mkfs.xfs -f -d su=256k,sw=3 -r extsize=768k /dev/md0
[root@localhost ~]# mkdir /srv/raid

# 挂载
[root@localhost ~]# mount /dev/md0 /dev/raid
[root@localhost ~]# df -hT /srv/raid
# 输出暂时不显示
```



模拟RAID错误的恢复模式：

```shell
[root@localhost ~]# mdadm --manage /dev/md[0-9] [--add 设备] [--remove 设备] [--fail 设备]
选项与参数：
--add：会将后面的设备加入到这个md[0-9]中
--remove：会将后面的设备从这个md[0-9]中删除
--fail：会将后面的设备设置成出错的状态

# 先复制一些东西到/srv/raid中去，假设这个RAID已经使用
[root@localhost ~]# cp -a /etc /var/log /srv/raid
[root@localhost ~]# df -hT /srv/raid;du -sm /srv/raid
# 输出暂时不显示

# 1.假设/dev/vda7这个设备出错了，实际模拟的方式
[root@localhost ~]# mdadm --manage /dev/md0 --fail /dev/vda7
# 输出暂时不显示

# 2.快速键入下面命令，可看到重建的输出
[root@localhost ~]# mdadm --detail /dev/md0
# 输出暂时不显示

# 3.移除旧的/dev/vda7磁盘
[root@localhost ~]# mdadm --manage /dev/md0 --remove /dev/vda7

# 4.安装新的/dev/vda7
[root@localhost ~]# mdadm --manage /dev/md0 --add /dev/vda7
[root@localhost ~]# mdadm --detail /dev/md0
# 输出暂时不显示


# 以上第3、4步骤可以不关机也能操作的步骤，实现热拔插的效果
```



开机自动启动RAID并自动挂载

```shell
# RAID也是有自己的配置文件的，这个配置文是/etcmdamd.conf，这个配置文件内容很简单，只要知道/dev/md0的UUID就能够设置这个文件了
[root@localhost ~]# mdadm --detail|grep -i uuid
# 输出暂时不显示，假设输出的uuid为cbb3c67a:67005dae:901bfaf9:99e22557

# 设置配置文件/etc/mdadm.conf
[root@localhost ~]# echo "ARRAY /dev/md0 UUID=cbb3c67a:67005dae:901bfaf9:99e22557" >> /etc/mdadm.conf

# 设置开启自动挂载并测试
[root@localhost ~]# blkid /dev/md0
/dev/md0: UUID="a0325dfb-e4cf-4a8a-9316-ace1e1ce153e" TYPE="xfs"
[root@localhost ~]# echo "UUID=a0325dfb-e4cf-4a8a-9316-ace1e1ce153e /srv/raid xfs defaults 0 0" >> /etc/fstab
[root@localhost ~]# umount /dev/md0; mount -a
[root@localhost ~]# df -Th /srv/md0
# 输出暂时不显示
```



软件关闭RAID

如果不需要了的话，只是将`/dev/md0`卸载，然后忘记将RAID关闭，结果就是未来你在重新划分/dev/vdax时可能会出现一些莫名的错误，所以才需要关闭软件RAID。

```shell
# 1.先卸载且删除配置文件内月这个/dev/md0相关的配置
[root@localhost ~]# umount /srv/raid
[root@localhost ~]# vim /etc/fstab
#将前面添加的下面这一行自动挂载的配置删除
UUID=a0325dfb-e4cf-4a8a-9316-ace1e1ce153e /srv/raid xfs defaults 0 0~~

# 2.先覆盖掉RAID的metadata以及xfs的superblock，才关闭/dev/md0的方法
[root@localhost ~]# dd if=/dev/zero of=/dev/md0 bs=1M count=50
[root@localhost ~]# mdadm --stop /dev/md0   # 这样就关闭了
[root@localhost ~]# dd if=/dev/zero of=/dev/vda5 bs=1M count=10
[root@localhost ~]# dd if=/dev/zero of=/dev/vda6 bs=1M count=10
[root@localhost ~]# dd if=/dev/zero of=/dev/vda7 bs=1M count=10
[root@localhost ~]# dd if=/dev/zero of=/dev/vda8 bs=1M count=10
[root@localhost ~]# dd if=/dev/zero of=/dev/vda9 bs=1M count=10
[root@localhost ~]# cat /proc/mdstat
# 输出暂时不显示，可以看到确实不存在任何磁盘阵列设备

[root@localhost ~]# vim /etc/mdadm.conf
#将前面添加的下面这一行配置删除
ARRAY /dev/md0 UUID=cbb3c67a:67005dae:901bfaf9:99e22557
```

这里需要注意的是，为什么上面会有多个dd的命令？这是因为RAID的相关数据其实也会存一份在磁盘当中，所以如果你只是将配置文件删除，同时关闭了RAID，但是分区并没有重新规划过，那么重新启动过后，系统还是会将这块磁盘阵列建立起来，只是名称可能会变成/dev/md127。因此，删除掉软件RAID时，上述的dd命令不要忘记了，但是千万不要dd到错误的磁盘，那可是会欲哭无泪的。





## 逻辑卷管理器（Logical Volume Manager）

### 概念（PV、PE、VG、LV等）

想象一个情况，你在当初规划主机的时候，只给了/home 500G，等到用户众多之后，这个文件系统不够大，此时怎么做？多数人都是这样：再加一块新硬盘，然后重新分区并格式化，将/home的数据完整地复制过来，然后将原本的分区卸载重新挂载新的分区，好麻烦！

LVM的重点在于**可以弹性地调整文件系统的容量**，而并不在于性能与数据安全。**LVM可以整合多个物理分区，让这些分区看起来像是一个磁盘一样。并且，未来还可以在这个LVM管理的磁盘当中新增或删除其他的物理分区。**

LVM的全名是Logical Volume Manager，中文翻译为逻辑卷管理器。LVM的做法是将几个物理的分区或磁盘通过软件组成一块看起来是独立的大磁盘（VG），然后将这块大磁盘再经过划分成可使用的分区（LV），最终就能够挂载使用了。以下介绍几个重要的概念。

- 物理卷（Physical Volume，PV）

我们实际的分区（或Disk）需要调整成系统标识符（system id）成为8e（LVM的标识符），然后再经过pvcreate的命令将它转成LVM最底层的物理卷（PV），之后才能够将这些PV加以利用，调整system id的方式就是通过gdisk

- 卷组（Volume Group，VG）

所谓的LVM大磁盘就是将许多PV整合成这个VG，所以VG就是LVM组合起来的大磁盘，这个开袋就行了。

- 物理扩展块（Physical Extent，PE）

LVM默认使用4MB的PE数据块，它是整个LVM最小的存储数据单元，也就是说，其实我们的文件数据都是借由写入PE来完成的。简单的说PE有点像文件系统里的block大小。

- 逻辑卷（Logical Volume，LV）

最终的VG还是会被切成LV，这个LV就是最后可以被格式化使用的类似分区的东西了。LV的大小必须是PE的整数倍。为了方便用户利用LVM管理其系统，LV的设备文件名通常为`/dev/vgname/vnmane`的样式。

**LVM可以弹性地修改文件系统的容量是通过交换PE来进行数据转换**，将原本LV内的PE移动到其他设备中以降低LV容量，或将其他设备的PE移到此LV中以加大容量。

那么数据写入这个LV时，到底它是怎么写入硬盘当中的呢？有两种方式：

1. 线性模式：假如我将/dev/vda1、/dev/vdb1这两个分区加入到VG中，并且整个VG只有一个LV时，那么所谓的线性模式就是当/dev/vda1的容量用完之后，/dev/vdb1的硬盘才会被使用到，这也是我们所建议的模式
2. 交错模式：将一条数据拆分成两部分，分别写入/dev/vda1、/dev/vdb1的意思，有点像RAID 0，如此一来，一份数据用两块硬盘来写入，理论上读写的性能会比较好。



**综上：LVM做主要的用途就是用在可以弹性调整容量的文件系统上，而不是一个建立性能为主的磁盘上。**





### 创建范例实践

大致流程如下：

- 使用4个硬盘分区，每个分区的容量均为1GB，且system id为8e
- 全部的分区整合成为一个VG，VG名称设置为testvg，且PE的大小为16MB
- 建立一个名为testlv的LV，容量为2GB
- 最终这个LV格式化为xfs的文件系统，且挂载在/srv/lvm中



1. disk阶段（实际的磁盘）

```shell
[root@localhost ~]# gdisk -l /dev/vda
# 输出暂时不显示

# 其实system id不修改也没关系，只是为了让管理员清楚知道该分区的内容
# 建议还是自定义成正确的磁盘内容较佳
```

2. PV阶段

要建立PV其实很简单，只要使用pvcreate即可。pv相关的命令

- pvcreate：将物理粪污建立成PV
- pvscan：查找目前系统里面任何具有PV的磁盘
- pvdisplay：显示出目前系统上面的PV状态
- pvs：pvdisplay的简版信息
- pvremove：将PV属性删除，让该分区不具有PV属性

```shell
# 检查有无PV在系统上，然后将/dev/vda{5-8}建立成为PV格式
[root@localhost ~]# pvscan
# 输出暂时不显示

[root@localhost ~]# pvcreate /dev/vda{5,6,7,8}

[root@localhost ~]# pvscan
# 输出暂时不显示

# 更详细地列出系统上每个PV的个别信息
[root@localhost ~]# pvdisplay /dev/vda5
# 输出暂时不显示
```



3. VG阶段

VG阶段的相关命令有：

- vgcreate：建立VG的命令，参数比较多，等下介绍

- vgscan：查找系统上面是否有VG存在

- vgdisplay：查找目前系统上面的VG状态

- vgs：vgdisplay的简版

- vgextend：在VG内增加额外的PV
- vgreduce：在VG内减少PV
- vgchange：设置vg是否启动（active）
- vgremove：删除一个vg

与PV不同的是，VG的名称是自定义的。PV的名称其实就是分区的设备文件名，但VG的名称可以随便取。

```shell
# 创建VG
[root@localhost ~]# vgcreate [-s N[mgt]] VG名称 PV名称 PV名称
选项与参数：
-s：后面接PE的大小，可带单位

# 将/dev/vda5-7建立成一个VG，且指定PE为16MB
[root@localhost ~]# vgcreate -s 16m testvg /dev/vda{5,6,7}
[root@localhost ~]# vgscan
# 输出暂时不显示

[root@localhost ~]# pvscan
# 输出暂时不显示

[root@localhost ~]# vgdisplay testvg
# 输出暂时不显示

# 将剩余的PV（/dev/vda8）增加给testvg
[root@localhost ~]# vgextend testvg /dev/vda8

[root@localhost ~]# vgdisplay
# 输出暂时不显示
```



4. LV阶段

创建出VG这个大磁盘后，再来就是要建立分区，这个分区就是LV。涉及的命令有：

- lvcreate：建立LV
- lvscan：查询系统上面的LV
- lvdisplay：显示系统上的LV状态
- lvs：简版lvdisplay
- lvextend：在LV里面增加容量
- lvreduce：在LV里面减少容量
- lvremove：删除一个LV
- lvresize：对LV进行容量大小调整

```shell
# 创建lv
[root@localhost ~]# lvcreate [-L N[mgt]] [-n LV名称] vg名称
[root@localhost ~]# lvcrate [-l N] [-n LV名称] vg名称
选项与参数：
-L：后面接容量，可带单位，M、G、T等，最小单位是PE，所以该值必须是PE的整数倍
-l：后面接PE的个数
-n：后面接lv名称
[root@localhost ~]# lvcreate -L 2G -n testlv testvg
[root@localhost ~]# lvscan
# 输出暂时不显示

# 此处lv必须使用全称
[root@localhost ~]# lvdisplay /dev/testvg/testlv
# 输出暂时不显示
```



5. 文件系统阶段

```shell
# 格式化与挂载LV
[root@localhost ~]# mkfs.xfs /dev/testvg/testlv 
[root@localhost ~]# mkdir /srv/lvm
[root@localhost ~]# mount /dev/testvg/testlv /srv/lvm
[root@localhost ~]# df -Th /srv/lvm
# 输出暂时不显示
```





### 扩容LV容量范例实践

大致流程是：

- VG阶段需要有剩余的容量，若容量不足可以加硬盘等其他操作
- LV阶段产生更多的可用容量，可使用`lvresize`或者`lvextend`命令进行增加
- 文件系统阶段的放大，仅放大LV是不会影响文件系统的，**xfs文件系统不支持减小**，xfs文件系统使用`xfs_growfs`命令对文件系统进行放大。

```shell
# 显示当前/dev/testvg/testlv的容量
[root@localhost ~]# lvdisplay /dec/testvg/testlv
# 输出暂时不显示

# 放大LV，增加500MB
# 也可用
[root@localhost ~]# lvresize -L +500m /dev/testvg/testlv
# +500m表示增加500MB，如果是 lvresize 500m /dev/testvg/testlv 表示增加到500MB
# -500m表示减少500MB
# 也可用lvextend或者lvreduce来减少
# 可增加-r参数，来一并增加文件系统

[root@localhost ~]# lvscan
# 输出暂时不显示

[root@localhost ~]# df -Th /srv/lvm
# 输出暂时不显示，此时该文件系统容量还未增加

# 先看下原本的文件系统内的superblock记录情况
[root@localhost ~]# xfs_info /srv/lvm
# 输出暂时不显示

# 增加文件系统容量
[root@localhost ~]# xfs_growfs /srv/lvm
[root@localhost ~]# df -Th /srv/lvm
```



### 使用LVM thin Volume让LVm自动调整磁盘使用率



### LVM的磁盘快照



### LVM相关命令集合与LVM关闭

| 任务                  | PV阶段    | VG阶段    | LV阶段    | 文件系统阶段（xfs） | 文件系统阶段（ext4） |
| --------------------- | --------- | --------- | --------- | ------------------- | -------------------- |
| 查找（scan）          | pvscan    | vgscan    | lvscan    | lsblk、blkid        | lsblk、blkid         |
| 建立（create）        | pvcreate  | vgcreate  | lvcreate  | mkfs.xfs            | mkfs.ext4            |
| 列出（display）       | pvdisplay | vgdisplya | lvdisplay | df、mount           | df、mount            |
| 增加（extend）        |           | vgextend  | lvextend  | xfs_growfs          | resize2fs            |
| 减少（reduce）        |           | vgreduce  | lvreduce  | 不支持              | resize2fs            |
| 删除（remove）        | pvremove  | vgremove  | lvremove  | umount，重新格式化  | umount，重新格式化   |
| 修改容量（resize）    |           |           | lvresize  | xfs_growfs          | resize2fs            |
| 修改属性（attribute） | pvchange  | vgchange  | lvchange  | /etc/fstab、remount | /etc/fstab、remount  |



删除LVM的流程：

- 先卸载系统上面的LVM文件系统（包括快照与LV）
- 使用lvremove删除LV
- 使用vgchange -a n vgname让这个vg不具有active标志
- 使用vgremove删除VG
- 使用pvremove删除PV
- 最后使用fdisk/gdisk修改ID回来

```shell
[root@localhost ~]# umount /srv/lvm /srv/thin /srv/snapshot1
[root@localhost ~]# lvs testvg
# 输出不显示

# 要注意，先删除testthin-->testpool-->testlv比较好
[root@localhost ~]# lvremove /dev/testvg/testthin /dev/testvg/testpool
[root@localhost ~]# lvremove /dev/testlv
[root@localhost ~]# vgchange -a n testvg
[root@localhost ~]# vgremove testvg
[root@localhost ~]# pvremove /dev/vda{5,6,7,8}

# 再用gdisk将磁盘的ID改回83
# 整个流程就这样完成了
```

