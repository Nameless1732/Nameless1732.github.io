title: Hadoop高可用集群搭建
author: Jie
tags:
  - Linux
date: 2022-11-05 22:03:53
---
之前的博客写了搭建hadoop集群环境，今天写一写搭建高可用（HA）环境。
Hadoop-HA模式大致分为两个(个人在学习中的理解)：

- namenode 高可用
- yarn 高可用

![Hadoop HA](/images/pasted-8.png)
<!-- more -->
## 一、NameNode HA
- 首先启动zookeeper集群

```shell
#先启动三台zookeeper集群：
[root@hadoop01 bin]# pwd
/opt/module/zookeeper-3.4.10/bin
[root@hadoop01 bin]# ./zkServer.sh start

[root@hadoop02 bin]# pwd
/opt/module/zookeeper-3.4.10/bin
[root@hadoop02 bin]# ./zkServer.sh start

[root@hadoop03 bin]# pwd
/opt/module/zookeeper-3.4.10/bin
[root@hadoop03 bin]# ./zkServer.sh start

#分别查看三台状态
[root@hadoop01 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower

[root@hadoop02 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: leader

[root@hadoop03 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /opt/module/zookeeper-3.4.10/bin/../conf/zoo.cfg
Mode: follower

# 启动成功
```
- 修改配置

修改core-site.xml

修改hdfs.site.xml

- 启动journalnode

```shell
[root@hadoop01 hadoop]# sbin/hadoop-daemon.sh start journalnode
starting journalnode, logging to /opt/module/hadoop/logs/hadoop-root-journalnode-hadoop01.out
[root@hadoop01 hadoop]# jps
5873 JournalNode
5511 QuorumPeerMain
5911 Jps

[root@hadoop03 hadoop]# sbin/hadoop-daemon.sh start journalnode
starting journalnode, logging to /opt/module/hadoop/logs/hadoop-root-journalnode-hadoop03.out
[root@hadoop03 hadoop]# jps
5344 JournalNode
5382 Jps
5021 QuorumPeerMain
```
- 格式化NameNode并启动，也是只需要启动hadoop01和hadoop03

```shell
# 第一个namenode格式化
[root@hadoop01 hadoop]# bin/hdfs namenode -format

# 格式化之后启动namenode
[root@hadoop01 hadoop]# sbin/hadoop-daemon.sh start namenode
starting namenode, logging to /opt/module/hadoop/logs/hadoop-root-namenode-hadoop01.out
[root@hadoop01 hadoop]# jps
5511 QuorumPeerMain
7000 Jps
6591 JournalNode
6895 NameNode

# 第二个namenode同步第一个
[root@hadoop03 hadoop]# bin/hdfs namenode -bootstrapStandby

# 启动第二个namenode
[root@hadoop03 hadoop]# sbin/hadoop-daemon.sh start namenode
```
- 查看namenode(如下图，都启动成功，只是都在standby状态)

![hadoop01](/images/pasted-6.png)
![hadoop03](/images/pasted-7.png)
- 手动切换nn1为激活状态

```shell
[root@hadoop01 hadoop]# bin/hdfs haadmin -transitionToActive nn1
Automatic failover is enabled for NameNode at hadoop03/192.168.170.133:8020
Refusing to manually manage HA state, since it may cause
a split-brain scenario or other incorrect state.
If you are very sure you know what you are doing, please 
specify the forcemanual flag.
# 这里需要强制切换

[root@hadoop01 hadoop]# bin/hdfs haadmin -transitionToActive --forcemanual nn1
You have specified the forcemanual flag. This flag is dangerous, as it can induce a split-brain scenario that WILL CORRUPT your HDFS namespace, possibly irrecoverably.

It is recommended not to use this flag, but instead to shut down the cluster and disable automatic failover if you prefer to manually manage your HA state.

You may abort safely by answering 'n' or hitting ^C now.

Are you sure you want to continue? (Y or N) y
18/08/21 16:12:59 WARN ha.HAAdmin: Proceeding with manual HA state management even though
automatic failover is enabled for NameNode at hadoop03/192.168.170.133:8020
18/08/21 16:13:00 WARN ha.HAAdmin: Proceeding with manual HA state management even though
automatic failover is enabled for NameNode at hadoop01/192.168.170.131:8020

# 可以看到nn2已经激活，nn1在standby状态
[root@hadoop01 hadoop]# bin/hdfs haadmin -getServiceState nn1
standby
[root@hadoop01 hadoop]# bin/hdfs haadmin -getServiceState nn2
active
```
- 在zookeeper上配置故障自动转移节点

```shell
[root@hadoop01 hadoop]# bin/hdfs zkfc -formatZK
[zk: localhost:2181(CONNECTED) 5] ls /
[zookeeper, hadoop-ha]
# 可以看到zookeeper上已经有了hadoop-ha节点了
```
- 启动集群

```shell
[root@hadoop01 hadoop]# sbin/start-dfs.sh
Starting namenodes on [hadoop01 hadoop03]
hadoop01: namenode running as process 6895. Stop it first.
hadoop03: namenode running as process 6103. Stop it first.
hadoop03: starting datanode, logging to /opt/module/hadoop/logs/hadoop-root-datanode-hadoop03.out
hadoop01: starting datanode, logging to /opt/module/hadoop/logs/hadoop-root-datanode-hadoop01.out
hadoop02: starting datanode, logging to /opt/module/hadoop/logs/hadoop-root-datanode-hadoop02.out
Starting journal nodes [hadoop01 hadoop02 hadoop03]
hadoop01: journalnode running as process 6591. Stop it first.
hadoop03: journalnode running as process 5814. Stop it first.
hadoop02: journalnode running as process 5450. Stop it first.
Starting ZK Failover Controllers on NN hosts [hadoop01 hadoop03]
hadoop01: starting zkfc, logging to /opt/module/hadoop/logs/hadoop-root-zkfc-hadoop01.out
hadoop03: starting zkfc, logging to /opt/module/hadoop/logs/hadoop-root-zkfc-hadoop03.out
[root@hadoop01 hadoop]# jps
8114 DFSZKFailoverController
7478 ZooKeeperMain
5511 QuorumPeerMain
8169 Jps
7803 DataNode
6591 JournalNode
6895 NameNode
# 在三台设备上分别jps一下，都启动了
```
- 接下来kill掉一个nn1，看看zookeeper是否会自动切换nn2

```shell
[root@hadoop03 hadoop]# jps
6708 DFSZKFailoverController
6549 DataNode
5814 JournalNode
6103 NameNode
6825 Jps
5021 QuorumPeerMain

# 杀掉namenode进程
[root@hadoop03 hadoop]# kill -9 6103

#查看nn2状态，连接不上了
[root@hadoop03 hadoop]# bin/hdfs haadmin -getServiceState nn2
18/08/21 16:28:52 INFO ipc.Client: Retrying connect to server: hadoop03/192.168.170.133:8020. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=1, sleepTime=1000 MILLISECONDS)
Operation failed: Call From hadoop03/192.168.170.133 to hadoop03:8020 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused

# 查看nn1状态，已经激活了
[root@hadoop03 hadoop]# bin/hdfs haadmin -getServiceState nn1
active

#重新启动同步nn1，并启动nn2
[root@hadoop03 hadoop]# bin/hdfs namenode -bootstrapStandby
[root@hadoop03 hadoop]# sbin/hadoop-daemon.sh start namenode
starting namenode, logging to /opt/module/hadoop/logs/hadoop-root-namenode-hadoop03.out
[root@hadoop03 hadoop]# jps
7169 Jps
6708 DFSZKFailoverController
6549 DataNode
5814 JournalNode
7084 NameNode
5021 QuorumPeerMain

# 接下来杀掉nn1，看看会不会自动激活nn2
[root@hadoop01 hadoop]# jps
8114 DFSZKFailoverController
8418 Jps
7478 ZooKeeperMain
5511 QuorumPeerMain
7803 DataNode
6591 JournalNode
6895 NameNode
[root@hadoop01 hadoop]# kill -9 6895
[root@hadoop01 hadoop]# bin/hdfs haadmin -getServiceState nn1
18/08/21 16:32:11 INFO ipc.Client: Retrying connect to server: hadoop01/192.168.170.131:8020. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=1, sleepTime=1000 MILLISECONDS)
Operation failed: Call From hadoop01/192.168.170.131 to hadoop01:8020 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused
[root@hadoop01 hadoop]# bin/hdfs haadmin -getServiceState nn2
active
```
**NameNode-HA搭建完成！**
## 二、Yarn HA
- 修改配置

修改yarn-site.xml

- 启动yarn，由于hadoop02是resourceManager，所以在hadoop02上启动

```shell
[root@hadoop02 hadoop]# sbin/start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /opt/module/hadoop/logs/yarn-root-resourcemanager-hadoop02.out
hadoop02: starting nodemanager, logging to /opt/module/hadoop/logs/yarn-root-nodemanager-hadoop02.out
hadoop01: starting nodemanager, logging to /opt/module/hadoop/logs/yarn-root-nodemanager-hadoop01.out
hadoop03: starting nodemanager, logging to /opt/module/hadoop/logs/yarn-root-nodemanager-hadoop03.out
[root@hadoop02 hadoop]# jps
4898 QuorumPeerMain
7352 ResourceManager
5801 DataNode
7449 NodeManager
5450 JournalNode
7482 Jps

[root@hadoop01 hadoop]# jps
8114 DFSZKFailoverController
9508 Jps
7478 ZooKeeperMain
5511 QuorumPeerMain
7803 DataNode
9389 NodeManager
6591 JournalNode

[root@hadoop03 hadoop]# jps
6708 DFSZKFailoverController
6549 DataNode
5814 JournalNode
7961 Jps
7084 NameNode
7932 NodeManager
5021 QuorumPeerMain
```
- 在hadoop03上启动resourceManager

```shell
[root@hadoop03 hadoop]# sbin/yarn-daemon.sh start resourcemanager
starting resourcemanager, logging to /opt/module/hadoop/logs/yarn-root-resourcemanager-hadoop03.out
[root@hadoop03 hadoop]# jps
6708 DFSZKFailoverController
6549 DataNode
5814 JournalNode
8105 ResourceManager
7084 NameNode
7932 NodeManager
8140 Jps
5021 QuorumPeerMain
```
- 查看两个resourceManager的状态

```shell
[root@hadoop03 hadoop]# bin/yarn rmadmin -getServiceState rm1
active
[root@hadoop03 hadoop]# bin/yarn rmadmin -getServiceState rm2
standby
```
- 杀掉rm1，看看是否会自动切换rm2

```shell
[root@hadoop02 hadoop]# jps
4898 QuorumPeerMain
7352 ResourceManager
5801 DataNode
7449 NodeManager
5450 JournalNode
7854 Jps
# kill掉rm1
[root@hadoop02 hadoop]# kill -9 7352

# 查看rm1状态，已经离线了
[root@hadoop02 hadoop]# bin/yarn rmadmin -getServiceState rm1
18/08/21 17:22:31 INFO ipc.Client: Retrying connect to server: hadoop02/192.168.170.132:8033. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=1, sleepTime=1000 MILLISECONDS)
Operation failed: Call From hadoop02/192.168.170.132 to hadoop02:8033 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused

# 查看rm2状态，激活了
[root@hadoop02 hadoop]# bin/yarn rmadmin -getServiceState rm2
active

# 接下来重启rm1，kill掉rm2，看看是否会切换

# 启动hadoop02的rm1
[root@hadoop02 hadoop]# sbin/yarn-daemon.sh start resourcemanager
starting resourcemanager, logging to /opt/module/hadoop/logs/yarn-root-resourcemanager-hadoop02.out
[root@hadoop02 hadoop]# jps
4898 QuorumPeerMain
5801 DataNode
7449 NodeManager
5450 JournalNode
8091 Jps
8046 ResourceManager

# kill hadoop03的rm2
[root@hadoop03 hadoop]# jps
6708 DFSZKFailoverController
6549 DataNode
5814 JournalNode
8503 Jps
8105 ResourceManager
7084 NameNode
7932 NodeManager
5021 QuorumPeerMain
[root@hadoop03 hadoop]# kill 8105

# 查看rm1和rm2的状态
[root@hadoop03 hadoop]# bin/yarn rmadmin -getServiceState rm1
active
[root@hadoop03 hadoop]# bin/yarn rmadmin -getServiceState rm2
18/08/21 17:26:23 INFO ipc.Client: Retrying connect to server: hadoop03/192.168.170.133:8033. Already tried 0 time(s); retry policy is RetryUpToMaximumCountWithFixedSleep(maxRetries=1, sleepTime=1000 MILLISECONDS)
Operation failed: Call From hadoop03/192.168.170.133 to hadoop03:8033 failed on connection exception: java.net.ConnectException: Connection refused; For more details see:  http://wiki.apache.org/hadoop/ConnectionRefused

# 可以看到，已经切换成功了
```
**Yarn-HA搭建成功！**