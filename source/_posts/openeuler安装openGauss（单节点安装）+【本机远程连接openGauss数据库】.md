# 准备条件

首先我们需要准备一台带有openeuler操作系统的服务器，然后我们需要一个能够访问服务器的一个工具，这里我是用的是windows自带的openSSH服务，其实你用什么都可以，看你个人的喜好

# 安装前的准备工作

首先我们要知道openGauss数据库是不能够在root用户下安装的（别问我为什么，这个我试过！，其实官网有写，只不过没有强调），所以我们需要一个普通用户，这个你自己随便起名字，还要创建一个用户组（虽然我目前不知道有什么具体作用，但是官网说要，那就先创一个），但是创建的用户基本没有任何权限，所以我们还要赋予他可以为所欲为的权限！
	&emsp;&emsp;1. 创建普通用户
	&emsp;&emsp;2. 创建用户组
	&emsp;&emsp;3. 创建普通用户密码
	&emsp;&emsp;4. 赋予普通用户权限

OK，我们已经要知道要干什么了，接下来就将他们一一实现吧！

## 创建普通用户

我这里用omm
```
sudo adduser omm
```

## 创建用户组

```
groupadd mygroup
```

## 创建普通用户密码

```
 passwd omm

# 我密码这里直接设置了12345678
```

## 赋予普通用户权限
  
我们先创建我们需要openGauss安装的文件夹
```
mkdir /opt/software && mkdir /opt/software/openGauss
```
  
然后我们将文件夹的权限给omm
```
sudo chown omm /opt/software/openGauss 

# 这里需要输入我们一开始设置的omm用户的密码
```

## 准备openGauss安装包
  
我这里是用的scp命令将下好到我桌面的openGauss压缩包传到openeuler服务器里面
首先我们先进入到官网里面：[软件包 | openGauss](https://opengauss.org/zh/download/)
  
我这里选择的是openGauss 5.0.0 (LTS)，下载的是openGauss Server
	&emsp;&emsp;1. 架构：x86_64
	&emsp;&emsp;2. 操作系统：openEuler 20.03 LTS
	&emsp;&emsp;3. 软件包类型：openGauss_5.0.0 极简版
  
点击下载，他将下载到我们本地
  
然后我们用scp命令
```
scp【本地文件的路径】【服务器用户名】@【服务器地址】：【服务器上存放文件的路径】
# 示例
scp /Users/mac_pc/Desktop/test.png root@192.168.1.1:/root
```
  
我们将文件传到`/opt/software/openGauss` 下

## 开始安装openGauss数据库

进入omm用户
```
su omm
```

在安装前安装所需软件包，不然刚发车就熄火了。。。
```
yum install libaio*
# 没有权限使用yum，就在前面加sudo
```
  
然后进入openGauss安装包所在的文件夹
```
cd /opt/software/openGauss
```
  
执行解压命令
```
tar -jxf openGauss-5.0.0-openEuler-64bit.tar.bz2
```
  
如果没有权限，则用下面的命令之后再运行上一条命令
```
sudo chown omm /opt/software/openGauss/openGauss-5.0.0-openEuler-64bit.tar.bz2
```
  
解压完成以后，我们需要执行安装命令，openGauss数据库的安装是用shell脚本进行安装的
```
cd /opt/software/openGauss/simpleInstall
sh install.sh  -w "xxx" &&source ~/.bashrc

# xxx 代表数据库密码（8位，3种不同字符）
```
  
安装执行完成后，使用ps和gs_ctl查看进程是否正常
```
ps ux | grep gaussdb
gs_ctl query -D /opt/software/openGauss/data/single_node
```
  
执行ps命令，显示类似如下信息：
```
omm      24209 11.9  1.0 1852000 355816 pts/0  Sl   01:54   0:33 /opt/software/openGauss/bin/gaussdb -D /opt/software/openGauss/single_node
omm      20377  0.0  0.0 119880  1216 pts/0    S+   15:37   0:00 grep --color=auto gaussdb
```
  
执行gs_ctl命令，显示类似如下信息：
```
gs_ctl query ,datadir is /opt/software/openGauss/data/single_node
HA state:
    local_role                     : Normal
    static_connections             : 0
    db_state                       : Normal
    detail_information             : Normal

Senders info:
    No information
    
 Receiver info:
No information 
```

## 使用openGauss数据库

我想重新启动一下数据库
```
gs_ctl restart -D /opt/software/openGauss/data/single_node -Z single_node
```

但是他给我报错！
```
waiting for server to shut down... done
server stopped
[2023-11-22 01:04:05.651][1074251][][gs_ctl]: waiting for server to start...
.0 LOG:  [Alarm Module]can not read GAUSS_WARNING_TYPE env.

0 LOG:  [Alarm Module]Host Name: maker-iot-2022-1

0 LOG:  [Alarm Module]Host IP: maker-iot-2022-1. Copy hostname directly in case of taking 10s to use 'gethostbyname' when /etc/hosts does not contain <HOST IP>

0 LOG:  [Alarm Module]Cluster Name: dbCluster

0 LOG:  [Alarm Module]Invalid data in AlarmItem file! Read alarm English name failed! line: 57

0 WARNING:  failed to open feature control file, please check whether it exists: FileName=gaussdb.version, Errno=2, Errmessage=No such file or directory.
0 WARNING:  failed to parse feature control file: gaussdb.version.
0 WARNING:  Failed to load the product control file, so gaussdb cannot distinguish product version.
The core dump path is an invalid directory
2023-11-22 01:04:05.694 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  when starting as multi_standby mode, we couldn't support data replicaton.
2023-11-22 01:04:05.699 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  [Alarm Module]can not read GAUSS_WARNING_TYPE env.

2023-11-22 01:04:05.699 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  [Alarm Module]Host Name: maker-iot-2022-1

2023-11-22 01:04:05.699 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  [Alarm Module]Host IP: maker-iot-2022-1. Copy hostname directly in case of taking 10s to use 'gethostbyname' when /etc/hosts does not contain <HOST IP>

2023-11-22 01:04:05.699 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  [Alarm Module]Cluster Name: dbCluster

2023-11-22 01:04:05.699 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  [Alarm Module]Invalid data in AlarmItem file! Read alarm English name failed! line: 57

2023-11-22 01:04:05.705 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  loaded library "security_plugin"
2023-11-22 01:04:05.705 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] WARNING:  could not create any HA TCP/IP sockets
2023-11-22 01:04:05.705 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] WARNING:  could not create any HA TCP/IP sockets
2023-11-22 01:04:05.707 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  InitNuma numaNodeNum: 1 numa_distribute_mode: none inheritThreadPool: 0.
2023-11-22 01:04:05.707 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  reserved memory for backend threads is: 220 MB
2023-11-22 01:04:05.707 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  reserved memory for WAL buffers is: 128 MB
2023-11-22 01:04:05.707 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  Set max backend reserve memory is: 348 MB, max dynamic memory is: 8143 MB
2023-11-22 01:04:05.707 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  shared memory 3284 Mbytes, memory context 8491 Mbytes, max process memory 12288 Mbytes
2023-11-22 01:04:05.755 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [CACHE] LOG:  set data cache  size(402653184)
2023-11-22 01:04:05.792 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [SEGMENT_PAGE] LOG:  Segment-page constants: DF_MAP_SIZE: 8156, DF_MAP_BIT_CNT: 65248, DF_MAP_GROUP_EXTENTS: 4175872, IPBLOCK_SIZE: 8168, EXTENTS_PER_IPBLOCK: 1021, IPBLOCK_GROUP_SIZE: 4090, BMT_HEADER_LEVEL0_TOTAL_PAGES: 8323072, BktMapEntryNumberPerBlock: 2038, BktMapBlockNumber: 25, BktBitMaxMapCnt: 512
2023-11-22 01:04:05.812 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  gaussdb: fsync file "/opt/software/openGauss/data/single_node/gaussdb.state.temp" success
2023-11-22 01:04:05.812 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  create gaussdb state file success: db state(STARTING_STATE), server mode(Normal), connection index(1)
2023-11-22 01:04:05.842 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  max_safe_fds = 976, usable_fds = 1000, already_open = 14
The core dump path is an invalid directory
2023-11-22 01:04:05.844 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  the configure file /opt/software/openGauss/etc/gscgroup_omm.cfg doesn't exist or the size of configure file has changed. Please create it by root user!
2023-11-22 01:04:05.844 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [BACKEND] LOG:  Failed to parse cgroup config file.
2023-11-22 01:04:05.857 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] WARNING:  Failed to obtain environment value $GAUSSLOG!
2023-11-22 01:04:05.857 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] DETAIL:  N/A
2023-11-22 01:04:05.857 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] CAUSE:  Incorrect environment value.
2023-11-22 01:04:05.857 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] ACTION:  Please refer to backend log for more details.
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] WARNING:  Failed to obtain environment value $GAUSSLOG!
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] DETAIL:  N/A
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] CAUSE:  Incorrect environment value.
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] ACTION:  Please refer to backend log for more details.
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] WARNING:  Failed to obtain environment value $GAUSSLOG!
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] DETAIL:  N/A
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] CAUSE:  Incorrect environment value.
2023-11-22 01:04:05.858 [unknown] [unknown] localhost 140492023295936 0[0:0#0]  0 [EXECUTOR] ACTION:  Please refer to backend log for more details.

[2023-11-22 01:04:06.656][1074251][][gs_ctl]:  done
[2023-11-22 01:04:06.656][1074251][][gs_ctl]: server started (/opt/software/openGauss/data/single_node)
```

我可以用以下命令进入openGauss数据库
```
gsql postgres
gsql -U omm -d postgres
```

但是我不能用以下命令
```
gs_om -t start
# bash: gs_om: command not found

gs_clt -t start
# bash: gs_clt: command not found

gs_ctl start -D /opt/software/openGauss/data/single_node -Z single_node
# [2023-11-22 01:03:49.536][1074208][][gs_ctl]: gs_ctl started,datadir is /opt/software/openGauss/data/single_node
# [2023-11-22 01:03:49.539][1074208][][gs_ctl]:  another server might be running; Please use the restart command
```

但是我要用端口的话，他还是会报错
```
gsql -d postgres -p 8000

# failed to connect Unknown:8000.
```

好，我知道了，官方文档的是8000，但是openGauss数据库的默认端口是5432，可以使用下面的命令进入数据库
```
gsql -d postgres -p 5432
```

暂时不纠结了，我先实现一下远程连接数据库

## 远程连接数据库

### 完成远程连接配置

在omm用户下，进入数据库节点目录内
```
cd /opt/software/openGauss/data/single_node
```

我们需要将放行IP添加到pg_hba.conf
```
# 使用vim打开
vim pg_hba.conf

# 需要复制的内容
host all all 本机ip/32 md5 
host all all 0.0.0.0/0 md5

# 这里的本机IP地址是目前你正在使用的电脑的IPv4地址
# 在cmd中输入 ipconfig
# 在无线局域网适配器 WLAN:
```

接下来我们开始修改postgresql.conf文件
```
vim postgresql.conf

# 按下ESC 输入:/listen_address(这是vim的查找命令)
# 将listen_address的值改为*
# 按下ESC 输入:/password_encryption_type
# 将password_encryption_type的值改为0，使用md5进行加密（记得删前面的#）
```

别问我为什么使用md5进行加密，因为我一开始没有改加密算法，在进行数据库连接时会报错：
```
SASL: Only mechanism SCRAM-SHA-256 is currently supported
```

修改完成后，我们执行以下命令重启openGauss数据库
```
gs_ctl stop -D /opt/software/openGauss/data/single_node 
gs_ctl start -D /opt/software/openGauss/data/single_node
```

如果停止openGauss失败（上面的第一条命令停止不了），我们采取杀死进程的方式停止openGauss然后重启
```
 lsof -i:5432
 #输出内容：
 #COMMAND     PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
 #gaussdb 1086801  omm    8u  IPv4 45624200      0t0  TCP *:postgres (LISTEN)
 #gaussdb 1086801  omm    9u  IPv6 45624201      0t0  TCP *:postgres (LISTEN)
 #gaussdb 1086801  omm  111u  IPv4 45628788      0t0  TCP localhost:postgres->localhost:42404 (ESTABLISHED)

 kill -9 1086801
 gs_ctl start -D /opt/software/openGauss/data/single_node
```

因为官方说明禁止使用omm用户进行远程连接数据库，那我们先创建一个其他的openGauss数据库用户
```
# 先进数据库，这里使用的是omm管理员账户
gsql -d postgres

# 创建ryanzhang用户
CREATE USER ryanzhang PASSWORD 'Jack2022';

# 授予新建用户权限
GRANT ALL PRIVILEGES TO ryanzhang;
```

操作openGauss数据库的一些简单常用命令
```
\l --查看所有数据库 
\c --进入某个数据库 
\dt --查看数据库里面的表 
\q --退出Gauss
```

### 开始远程连接

我在网上查了一下，一般用两个东西进行远程连接（**Navicat或Data Studio**）
但是我不想下，我一直用的是vscode上各种扩展，我用的是Mysql![[Pasted image 20231122031849.png]]

navicat连接到openGauss的数据库类型选择：PostgreSQL
也就是说，我的数据库类型选择为PostgreSQL，就可以用我的vscode扩展进行连接
![[Pasted image 20231122032152.png]]

下面是我填的一些配置，连接成功！
![[Pasted image 20231122032416.png]]
