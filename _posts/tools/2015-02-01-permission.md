---
toc: true
toc_label: "Linux 的使用 - 权限控制"
toc_icon: "book"
title: "Linux 的使用 - 权限控制"
tags: linux umask sudo su 权限
category: "tools"
classes: wide
excerpt: "传统的 Linux 权限控制，SELinux"
header:
  overlay_image: /assets/images/header/access-control-systems.jpg
  overlay_filter: rgba(0, 0, 0, 0.6)
published: false
---



## 权限控制的类别

* MAC：Mandatory Access Control，**强制** 访问控制。
* DAC：Discretionary Access Control，**自由** 访问控制。
* RBAC：Role Based Access Control，基于 **角色** 的访问控制。

传统的 Linux 权限控制其主体是 **UID** 及 **GID**。


### DAC

Discretionary Access Control，**自由** 访问控制。它偏重于 **可用性**，它的决策是基于 **权限**。

DAC 让用户管理自己的数据。如在线社区网络中，让用户自行选择谁可以访问他们的数据。通过 DAC ，用户可以轻松地立即废除或转发权限。


### RBAC

Role Based Access Control，基于 **角色** 的访问控制。它的决策是基于 **功能** 或 **角色**。

RBAC 适用于在一个系统中把各项责任拆开，分配给不同的角色。RBAC 既适用于企业，也适用于单用户操作系统，用来实施最小权限原则。RBAC 的设计初衷是：通过允许用户为特定任务选择自己角色，来 **拆分责任**。因此关键的问题是，你是否使用角色来完成特定的任务，并且需要经由统一授权来分配角色。或者你需要使用角色，来让用户在自己的对象上控制权限，导致每个对象可以使用不同的角色。


### MAC

Mandatory Access Control，**强制** 访问控制。

MAC 的概念本身有些模糊，它的实现有许多方式。它适用于对 **保密级别** 要求较高的情况，它的决策是先基于 **标签**，然后是 **权限**。

在生产中，经常使用不同权限控制的组合。例如，在 UNIX 系统中，最常用使用的是 DAC ，但 root 帐户却可以绕过其限制。在企业中，除了用 MAC / RBAC 来为各部门分配权限之外，还可以允许通过 CAC 在同事之间共享信息。






















## 文件权限


### `umask`

设置默认权限掩码

`umask [ -S ] [ Mask ]`

`-S`	用符号表示当前权限掩码

如果不带参数运行，`umask` 命令直接显示当前掩码

`umask a=rx,ug+w`	以符号方式设置

`umask 002`	以数字方式设置

`umask -S`	以符号方式显示当前设置

`umask g-w`		组去掉修改权限

`umask -- -w`	所有人去掉修改权限





### `chmod`

`chmod` 用于修改文件的权限位。

`chmod [参数] 文件名`

`chmod -R 777 /path/`	 **递归** 修改目录中所有文件

`chmod u=rwx,go=rx file`	同时修改 U/G/O 权限

`chmod a+w file`	单独修改 rwx 权限

`chmod 4755 file`	修改 SUID、SGID、SBIT






### `chown`

修改文件所有者、属组

`chown [参数] 用户 文件`

`-R`	递归处理，将指定目录下的所有文件及子目录一并处理

`-v`	显示指令执行过程

`--reference=`	把指定文件或目录的所属群组全部改为和参考文件或目录的所属群组相同

* 范例

`chown -R newuser file`	修改所有者

`chown -R newuser:newgroup file`  	同时修改所有者及属组

`chown -R :newgroup file`	修改文件的属组为另一个现有组





### `chgrp`

修改文件属组

`chgrp [参数] 组 文件`

`-R`	递归处理，将指定目录下的所有文件及子目录一并处理

`-v`	显示指令执行过程

`--reference=`	把指定文件或目录的所属群组全部改为和参考文件或目录的所属群组相同

* 范例

`chgrp -R anothergroup file`	修改文件的属组为另一个现有组















## 文件属性




### `chattr`  

`chattr` 命令用于修改文件属性，但只能在 EXT 中完整生效，XFS 系统中仅支持部份参数。

`chattr [ -RVf ] [ -v version ] [ -p project ] [ mode ] files...`

`A`	**锁定 atime**，可减少 I/O。

`a`	**只能追加** 内容，不能删除文件，不能修改，限 **超级用户** 使用

`d`	不被 dump 备份

`i`	禁止 **删除、改名、链接、修改**，限 **超级用户** 使用

`S`	将文件的修改 **同步** 写入磁盘。

EXT 文件系统支持所有参数，XFS 仅支持 `AadiS`。
{: .notice--warning}

`c`	读之前自动解压，写之前先压缩

`s`	如果删除文件，会被完全移除出硬盘空间，无法恢复

`u`	删除文件后，数据仍存在磁盘，可以恢复

* 范例

`chattr +i`	锁定文件，禁止改名、修改、删除、链接

`chattr +a`	文件只允许追加内容，不能删除、修改

`chattr +d`	不会被 dump 备份

`chattr +A`	锁定 atime，减少 IO

`chattr +S`	有变化时 **同步** 写入磁盘






### `lsattr`  

查看文件 **隐藏属性**

`lsattr -a`	查看当前目录中所有文件，包括隐藏文件的隐藏属性

`lsattr -d`	仅查看目录本身的隐藏属性

`lsattr -R`	查看连同子目录的隐藏属性





### `setfattr`


使用 `setfattr` 来设置扩展属性

`setfattr [-h] -n name [-v value] pathname...`

`-n`	指定属性名，用户扩展属性必须用前缀 `user.` 来命名

`-v`	指定属性的值

`-x`	删除指定扩展属性，接属性名

* 范例

`setfattr -n user.cell -v "13099999999" file`
















## ACL





### 查看系统是否支持 ACL

`dmesg | grep -i acl`





### `getfacl`

`getfacl`  用于查看文件的 ACL 条目

`getfacl filename`





### `setfacl`

`setfacl` 用于设置、修改、删除普通文件和目录的访问控制列表。

`setfacl [-bkRd] [{-m|-x} acl参数] 目标文件名`

`setfacl -m <条目> <文件>`


`-m`  **修改** ACL 条目

`-x`  **删除** ACL 条目

`-b`  **删除所有** 的 ACL 设置

`-R`  **递归** 设置 ACL

`-d`  设置目录的 **默认 ACL**

`-k`  **删除默认 ACL**

`--set` 和 `--set-file` 用于设置文件或目录的 ACL 条目，先前的设定将被覆盖

`-M` 修改 ACL 条目，**从文件或 STDIN 读取** ACL 条目

`-X` 删除 ACL 条目，**从文件或 STDIN 读取** ACL 条目

当使用 `-M`，`-X` 选项从文件中读取规则时，`setfacl` 接受 `getfacl` 命令输出的格式。每行至少一条规则，以 `#` 开始的行将被视为注释。
{: .notice}  





#### 范例

* 为指定用户设定权限

`setfacl -m u:user1:rx file`

* 为文件所有者设定权限

`setfacl -m u::rwx file`

`u` 后面留空，表示针对文件 **所有者**。

* 为指定用户设定权限，空

`setfacl -m u:pro3:- /srv/projecta`

设置用户对目录 **无任何权限**，权限字段用 **`-`**，不能留空。

* 为指定组设定权限

`setfacl -m g:mygroup1:rx file`

* 设定有效权限掩码

`setfacl -m m:r file`

借助 mask 来设定最大权限，可以避免权限误分配。
{: .notice--success}

* 删除指定用户权限

`setfacl -x u:500 /project/somefile`

* 为目录设定默认 ACL

通过设置目录对于特定用户的默认权限，使这些用户在目录内新建的文件能够继承其 ACL。

只需在规则前加上 `d:`

`setfacl -m d:u:myuser1:rx /srv/projecta`
























## 切换用户



### `su`

`su` 在某个登陆会话中，变成其他用户。

`su [options] [name]`

如果不指定用户名，则会切换到 root。

`-c command` 临时使用指定用户的 shell 运行命令

`-`/`-l`  以 login shell 方式切换，提供该用户登陆后的环境。

`-m`  保留当前的环境变量（$PATH, $IFS 除外）

#### 范例


* 以 non-login shell 方式切换 root

`su`  环境变量仍为原用户的

* 用 login shell 切换 root

`su -` 环境变量更新为 root 的

* 临时执行命令

`su - -c "head -n 3 /etc/shadow"`

临时用 root 身份执行，命令完成之后仍是原身份

* 切换为其他用户

`su -l user1` 以 login shell 方式切换，获得新用户环境变量









### `sudo`

`sudo [-b] [-u 新用户帐号]`


`-b`   把命令置于后台运行

`-u`   指定切换的用户，省略则切换为 root



#### 配置文件

`sudo` 专用的配置文件为 `/etc/sudoers`，但不能直接修改，一定要使用 **`visudo`** 来编辑。

如果只是要增加 sudoers，建议在 `/etc/sudoers.d/` 目录中新建自定义配置文件，用 `sudo visudo -f /etc/sudoers.d/neo` 来编辑该文件来实现，既安全，又灵活。
{: .notice--success}



##### 为特定用户增加 SUDO 权限

```conf
neo ALL=(ALL) ALL
```

##### 为特定组增加 SUDO 权限

与配置用户的区别就是在组名前面加上百分号 `%`。

```conf
%wheel ALL=(ALL) ALL
```

##### 允许免输密码使用 SUDO

```conf
neo ALL=NOPASSWD: ALL
%wheel ALL=NOPASSWD: ALL
```

这样配置以后，用户使用 sudo 命令时无需再输入密码。



#### 范例

* 以其他身份创建文件

`sudo -u sshd touch /tmp/mysshd`


* 以其他身份执行脚本

`sudo sh -c "cd /home ; du -s * | sort -rn > USAGE"`






















## SELinux






### 配置文件

`/etc/selinux/config` 是 SELinux 的主要配置文件，它控制着 SELinux 启动还是禁用，以及当前使用哪个 SELinux 模式、哪个策略。

通过编辑 `/etc/selinux/config` 配置文件，来切换 SELinux 状态或模式。

```
SELINUX=enforcing/permissive/disabled
```









### 状态与模式

`getenforce`  查看 SELinux 当前工作模式

`setenforce 0`  切换到宽容模式

`setenforce 1`  切换到强制模式



#### 系统启动时指定参数

`enforcing=0`  让系统以宽容模式启动

`selinux=0`  禁用 SELinux

`autorelabel=1`  创建 `/.autorelabel`，然后重启









### 上下文



#### 查看上下文

`ls -Z` 查看文件和目录的上下文

`id -Z` 查看当前 L 用户的上下文

`ps -eZ | grep httpd`  查看特定进程的上下文



#### 上下文操作


##### 修改上下文

`chcon -t samba_share_t /var/www/html/testfile`  修改文件类型

`chcon -v --reference=/etc/shadow /etc/cron.d/checktime`  参考文件自动修改上下文

`restorecon -v /usr/sbin/httpd`  修复文件上下文

`restorecon -FR -v /home/neo`  修改目录上下文

免于 RELABEL 的修改方式：

```bash
~]# semanage fcontext -a options file-name|directory-name
~]# restorecon -v file-name|directory-name
```


##### 标签维护

`cp --preserve=context file /tmp/` 复制时保留原上下文

`cp --context=system_u:object_r:samba_share_t:s0` 复制时指定目标文件的上下文

`matchpathcon -V /var/www/html/*`  检查该目录中所有文件的上下文是否正确

`tar --selinux`  打包或解包时保留上下文，不用参数解包时会继承目标目录的上下文




#### 挂载文件系统

`mount -o context=SELinux_user:role:type:level`  挂载文件系统时使用上下文参数

当文件系统 **挂载时使用了 `context` 参数** 时，系统会 **禁止用户和进程修改上下文**，`chcon` 会返回 `Operation not supported` 错误。
{: .notice--warning}


##### 用挂载参数控制服务对 NFS 的访问


希望 **禁止** 其它服务访问挂载的 NFS，挂载时 **不加额外参数**

```
~]# mount server:/export /local/mount/point
```

希望 **允许** 网页服务访问挂载的 NFS，可以为其 **指定** 网页服务专用的 **类型**：

```
~]# mount server:/export /local/mount/point -o \
context="system_u:object_r:httpd_sys_content_t:s0"
```


##### 修改文件系统的默认上下文

```
~]# mount /dev/sda2 /test/ -o defcontext="system_u:object_r:samba_share_t:s0"
```


##### NFS 同一导出目录不同子目录，用不同类型分别挂载

```
~]# mount server:/export/web /local/web \
-o nosharecache,context="system_u:object_r:httpd_sys_content_t:s0"
#                                          ^^^^^^^^^^^^^^^^^^^

~]# mount server:/export/database /local/database \
-o nosharecache,context="system_u:object_r:mysqld_db_t:s0"
#                                          ^^^^^^^^^^^
```


##### `/etc/fstab` 中指定挂载参数

```
server:/export /local/mount/ nfs context="system_u:object_r:httpd_sys_content_t:s0" 0 0
```









### 布尔值

`semanage boolean -l`  查看布尔值名称、描述、开关状态

`getsebool -a`  查看布尔值名称、开关状态

`getsebool boolean-name1 boolean-name2`  查看指定名称的布尔值状态

`setsebool -P guest_exec_content off`  关掉布尔值

`-P` 参数做出的改变会持续生效，因此如果不希望在重启后还有效，就不要使用 `-P` 参数。
{: .notice--warning}










### 信息分析



#### `seinfo` 查询 SELinux 策略中的各要素

`seinfo` 用于查询 SELinux 策略中的各要素，身份、角色、类型、规则。它使用 `policy.conf`、二进制策略文件、策略包的模块化列表或一个策略列表文件做为输入。其输出也会根据不同的输入而有所区别。

`seinfo [-trub]`

`--all`   查看所有要素：SELinux 状态、规则、身份、角色、类型

`-u`		查看所有身份

`-r`		查看所有角色

`-t`		查看所有类型

`-b`		查看所有规则

`seinfo -u`  查看用户列表

`seinfo -r` 查看所有角色




#### `avcstat` 本次开机以来 AVC 的简要统计

`avcstat`  查看本次开机以来 AVC 的简要统计




#### `sestatus` 查询 SELinux 当前状态信息


* SELinux 当前状态，启动或关闭
* SELinux 关键目录的位置，/etc/selinux
* 当前已载入的策略名称
* SELinux 的挂载点
* 当前工作模式
* 配置文件中指定的工作模式
* MLS 状态
* 对于未定义操作，是否默认采取拒绝

`sestatus -v`   查看 SELinux 当前状态信息

`sestatus -b`   查看所有布尔值列表，左侧为名称，右侧为布尔值





#### `semanage` 配置策略中的各要素

`semanage {login|user|port|interface|fcontext|translation} -l `

`semanage` 用于配置策略中的 **特定要素**，而无需重新编译策略源文件。

* **L 用户到 SL 用户的映射**。该映射控制着 L 用户登陆后获得授权角色时，系统为其分配的初始上下文。
* **为各种不同类别的客体分配的上下文映射**。主要指网络端口、网络接口、网络主机等类别。
* **文件上下文的映射**。

`semanage login` 用于由 Linux login 用户映射到 SL 用户。

`semanage user` 用于由 SL 用户映射到角色组（一用户可以有多角色）。

多数时间里，管理员只需要使用 `semanage login` 来设置，后者原则上是由基础策略模块定义的，通常不需要修改。


##### 修改目录默认上下文

默认使用 `targeted` `策略时，semanage` 查询到的关于上下文的信息都来自于 `/etc/selinux/targeted/contexts/files/` 目录中的各个文件。

`semanage fcontext -{a|d|m} [-frst] file_spec`

`fcontext`	设置安全上下文

`-l`			查询

`-a`			增加

`-m`			修改

`-d`			删除


##### 用户操作

* 查看 L 用户和 SL 用户的映射

	`semanage login -l`

* 创建新 SL 用户，并指定其默认角色及附加的受限管理员角色

	`semanage user -a -r s0-s0:c0.c1023 -R "staff_r webadm_r" neo_u`

* 把新建 SL 用户映射给现有 L 用户 `neo`

	`semanage login -a -s neo_u -rs0:c0.c1023 neo`

* 新建 L 用户时，指定其 XL 的映射对象

	`useradd -Z user_u user`

* 修改现有用户的映射

	`semange login -a -s neo_u neo`  原 SL 用户可能为 `unconfined_u`，修改为 `neo_u`

* 修改默认映射

	`semanage login -m -S targeted -s "user_u" -r s0 __default__`


#####  范例：查看 `/etc` `/etc/cron.d` 两个目录的默认上下文

> 思路：先用 `semanage fcontext -l` 列出系统中所有文件上下文的定义，有几千条，然后使用 grep 过滤。

```
~]#semanage fcontext -l | grep -E '^/etc |^/etc/cron'
SELinux fcontext type Context
/etc all files system_u:object_r: etc_t :s0
/etc/cron\.d(/.*)? all files system_u:object_r: system_cron_spool_t :s0
```

`semanage fcontext -l` 返回的结果格式：

`文件的绝对路径（正则表达式)  文件种类  上下文`

由查询结果可知，`/etc` 目录的默认文件类型为 etc_t；`/etc/cron.d` 目录的默认文件类型为 `system_cron_spool_t`。


##### 范例：设置新建目录的默认上下文类型

😈 新建的目录是不存在的上下文默认值的，可以手动创建，以便之后在该目录可以随时使用 `restorecon` 命令。本例只创建其默认的 **文件类型**。

```
~]# mkdir /srv/mycron
~]# semanage fcontext -a -t system_cron_spool_t "/srv/mycron（/.*）?"
```





#### `sesearch` 在策略中查找特定的规则

可以在策略源文件中查找，也可以在二进制文件中查找。

`sesearch [-A] [-s 主体类型] [-t 客体类型] [-b 规则名称]`

`-A`  仅查看放行的规则

`-C`  显示条件表达式及状态

>`sesearch -C` 会在返回结果的开头加上两个字母，第一个字母表示 **本条规则当前状态**，第二个字母表示 **布尔值的哪个状态会启用本条规则**。
> E D T F 分别为 Enabled, Disabled, True, False 的首字母。
> * ET：本规则当前启用，布尔值打开时会启用本规则
> * EF：本规则当前启用，布尔值关闭时会启用本规则
> * DT：本规则当前禁用，布尔值打开时会启用本规则
> * DF：本规则当前禁用，布尔值关闭时会启用本规则

`-s`  指定主体类型，域

`-t`  指定客体类型，文件类型

`-b`  指定规则名称

`sesearch` 返回的格式是这样的：

```
  allow crond_t setfiles_exec_t : file { read getattr execute open } ;
# 控制 |   域  |     类型       | 客体  |           操作
```


##### 范例：指定的域有权访问哪些类型的文件

运行于 `crond_t` 域中的进程是否有权访问与 `spool` 相关的文件。

先查询该域所允许的所有规则，再以关键字过滤：

```
~]# sesearch -A -s crond_t | grep spool
   allow crond_t system_cron_spool_t : file { ioctl read write create } ;
#   放行   域        文件类型
```

查询结果表明：`crond_t` 域中的进程允许访问 `system_cron_spool_t` 类型的文件。


##### 范例：指定的域是否有权访问特定文件

运行于 `crond_t` 域中的进程是否有权访问 `/etc/cron.d/checktime` 文件。

先查看该文件的 SL 类型，再查询该域是否有权访问该类型。

```
~]# ll -Z /etc/cron.d/checktime  
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 /etc/cron.d/checktime
#                                            ^^^^^类型
```

查询 `crond_t` 域的所有规则，从中把 `admin_home_t` 筛选出来：

```
~]# sesearch -A -s crond_t | grep admin_home_t
#                                              ^^^^^^^^^^
   allow domain admin_home_t : dir { getattr search open } ;
   allow domain admin_home_t : lnk_file { read getattr } ;
   allow crond_t admin_home_t : dir { ioctl read getattr lock search open } ;
   allow crond_t admin_home_t : lnk_file { read getattr } ;
```

虽然有 `crond_t` `admin_home_t` 的规则，但不是针对文件的，因此 `crond_t` 域中的进程无权访问 `admin_home_t` 类型的文件。


##### 范例：查看某个布尔值所影响的所有规则

要知道布尔值的完整名称，通过该命令可以了解该布尔值开或关时，会影响到哪些具体的规则的状态。布尔值开的时候，有哪些规则会启用，哪些会禁用，以及布尔值关的时候的情况。

仅查看布尔值影响的规则：

```
~]# sesearch -A -b httpd_enable_homedirs
#                       布尔值名称
Found 72 semantic av rules:
   allow httpd_suexec_t user_home_dir_t : dir { getattr search open } ;
   allow httpd_t nfs_t : lnk_file { read getattr } ;
```

查看布尔值影响规则的同时，显示条件表达式及状态：

```
root #sesearch -b user_dmesg -AC
Found 4 semantic av rules:
ET allow user_t kernel_t : system syslog_read ; [ user_dmesg ]
ET allow user_t user_t : capability2 syslog ; [ user_dmesg ]
ET allow staff_t kernel_t : system syslog_read ; [ user_dmesg ]
ET allow staff_t staff_t : capability2 syslog ; [ user_dmesg ]
```


##### 范例：哪些角色可以访问指定类型的文件

用 `--role_allow` 参数来查找，哪些角色可以访问 `httpd_sys_content_t` 类型的文件：

```
~]$ sesearch --role_allow -t httpd_sys_content_t
Found 20 role allow rules:
   allow system_r sysadm_r;
   allow sysadm_r system_r;
   allow sysadm_r staff_r;
......
```


























## 案例







### SELinux 排错





#### `vsftpd` 服务常遇到的问题


##### vsftpd 服务要求

匿名用户：

* 用浏览器连接到 FTP 服务器时，默认用匿名用户登陆系统。
* 匿名用户的主文件夹默认为 `/var/ftp`
* 匿名用户只有权访问主文件夹，只能下载不能上传

普通帐号：

* 默认 UID 大于 1000 的帐号都可以访问 FTP
* 所有帐号只允许访问自己的主文件夹
* 所有帐号可以上传、下载


##### 准备测试环境

新建测试帐号

```
~]# useradd -s /sbin/nologin ftptest  
~]# echo "myftp123" | passwd --stdin ftptest
```

匿名用户通常的主目录为 `/var/ftp/pub` 。

把 `/etc/securetty` 和 `/etc/sysctl.conf` 复制到 `/var/ftp/pub` 提供下载：

```
~]# cp -a /etc/securetty /etc/sysctl.conf /var/ftp/pub
~]# ll /var/ftp/pub
```

在 ftptest 用户的主文件夹创建文件

```
~]# echo "testing" > ~/test.txt
~]# cp -a /etc/hosts /etc/sysctl.conf .
```

安装 vsftpd 并设置开机启动

```
~]# yum install /mnt/Packages/vsftpd-3*
~]# systemctl start vsftpd
~]# systemctl enable vsftpd
```


##### 匿名用户无法下载某些文件

查看文件 rwx 权限是否正常。


##### 普通用户从家目录之外的目录上传/下载文件

假设想提供 `/srv/go` 这个目录给 ftptest 用户使用：

```
~]# mkdir /srv/go
~]# chgrp ftptest /srv/go
~]# echo "test" > /srv/go/test.txt

~]# curl ftp://ftptest:myftp123@localhost/srv/go/test.txt
curl: （78） RETR response: 550
```

拒绝访问，查一下日志，看有没有 SELinux 的警告消息：

```
~]# grep sealert /var/log/messages | tail
Aug  9 04:23:12 station3-39 setroubleshoot: SELinux is preventing /usr/sbin/vsftpd from
 read access on the file test.txt. For complete SELinux messages. run sealert -l
 08d3c0a2-5160-49ab-b199-47a51a5fc8dd
~]# sealert -l 08d3c0a2-5160-49ab-b199-47a51a5fc8dd
SELinux is preventing /usr/sbin/vsftpd from read access on the file test.txt.
```

在 sealert 返回的消息中有一条有用的建议：

```
# semanage fcontext -a -t FILE_TYPE 'test.txt'
```

它建议你增加一条规则，把这个文件的文件类型改为 ftp 有权读取的，最快捷的方法就是直接查看 `/var/ftp` 目录的默认文件类型：

```
~]# ll -Zd /var/ftp
drwxr-xr-x. root root system_u:object_r:public_content_t:s0 /var/ftp
#                                       ^^^^^^^^^^^^^^^^
```

借用 setroubleshoot 的建议，为了一劳永逸，我们应该在规则中指定 `/srv/go` 目录中所有文件，以便日后随时可以用 `restorecon` 来纠正：

```
~]# semanage fcontext -a -t public_content_t "/srv/go（/.*）?"
~]# restorecon -Rv /srv/go
~]# curl ftp://ftptest:myftp123@localhost//srv/gogogo/test.txt
test
```


##### 修改 vsftpd 端口号

```
~]# vim /etc/vsftpd/vsftpd.conf
# 新增一行：
listen_port=555
```

重启 vsftpd 服务后出错：

```
~]# systemctl restart vsftpd
Job for vsftpd.service failed because the control process exited with error code. See "systemctl status vsftpd.service" and "journalctl -xe" for details.
```

检查日志：

```
~]# grep sealert /var/log/messages | tail
Oct 27 16:16:18 zion setroubleshoot: SELinux is preventing /usr/sbin/vsftpd from name_bind access on the tcp_socket port 555. For complete SELinux messages run: sealert -l 3588594e-a1dd-431f-913d-0b94f576f31e
```

执行完 sealert 之后，setroubleshoot 给的建议：

```
# semanage port -a -t PORT_TYPE -p tcp 555
```

因为现有规则中没有放行 555 端口的规则，我们要新建：

```
~]# semanage port -a -t ftp_port_t -p tcp 555
~]# systemctl restart vsftpd

~]# netstat -tlnp
tcp6       0      0 :::555           :::*              LISTEN   8436/vsftpd

~]# curl ftp://localhost:555/pub/
-rw-r--r--    1 0        0             221 Oct 29  2014 securetty
-rw-r--r--    1 0        0             225 Mar 06 03:05 sysctl.conf
```
