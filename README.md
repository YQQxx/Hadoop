# 分布式系统

------



## 常见模式：

1. 去中心化模式：没有明确中心，协调工作
2. 中心化模式：有明确中心，基于中心节点分配工作

注意：去中心模式又名主从模式（Master-Slaves）Hadoop正是这种模式

## Hadoop基础建构

### HDFS

​	HDFS 是Hadoop三大组件之一

​	全称：Hadoop Distributed File System（Hadoop分布式文件系统）

​	作用：是Hadoop技术栈内提供的分布式数据存储解决方案可以在多台服务器上构建集群，存储海量的数据。

​	HDFS的基础架构

### 三个角色：

1. 主角色(NameNode)：是一个独立的进程，负责管理HDFS整个文件系统，负责管理DataNode。

2. 从角色(DataNode)：主要负责数据的存储，即存入数据和取出数据。

3. 主角色辅助角色(SecondaryNameNode)：主要帮助NameNode完成数据整理工作。

   注意：典型的HDFS集群就是由一个主角色加若干从角色组成。

## HDFS集群环境部署

#### Hadoop安装包目录结构

1. bin：存放Hadoop的各类程序（命令）
2. etc：存放Hadoop的配置文件
3. include：c语言的一些头文件
4. lib：存放Linux系统的动态链接库（.so文件）
5. libexec：存放配置Hadoop系统的脚本文件（.sh和.cmd）
6. lincenses-vinary：存放许可文件
7. sbin：管理员程序（super bin）
8. share：存放二进制源码（Java jar包）

#### 配置HDFS集群

1. 上传Hadoop安装包到node1 节点

```
	tar-zxvf Hadoop-3.3.4.tar.gz -c /export/server
	cd /export/server
	ln-s /export/server/hadoop-3.3.4 hadoop
```

2. 配置workers文件

   ```
   cd /export/server/hadoop/etc/hadoop
   vim workers
   node1
   node2
   node3
   ```

   

3. 配置Hadoop-env.sh文件

```
export JAVA_HOME=/export/server/jdk——指明JDK坏境位置

export HADOOP_HOME=/export/server/hadoop——指明Hadoop安装位置

export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop——指明Hadoop配置文件位置

export  HADOOP_LOG_DIR=$HADOOP_HOME/logs——指明Hadoop运行日志目录位置
```

4. 配置core-site.xml文件

```
<configuration>

	<property>

		<name>fs.defaultFS</name>

		<value>hdfs://node1:8020</value>
	</property>

	<property>
		<name>io.file.buffer.size</name>
		<value>131072</value>
	</property>

</configuration>
```

​		key：fs.defaultFS

​		含义：HDFS文件系统的网络通讯路径

​		值：hdfs://node1:8020

​				协议为hdfs://

​				namenode为node1

​				namenode通讯端口为8020

​		key：io.file.bufer.size

​		含义：io操作文件缓冲区大小

​		值：131072 b

5. 配置hdfs-site.xml文件

```
<configuration>

	<property>
		<name>dfs.datanode.data.dir.perm</name>
		<value>700</value>
	</property>
		
	<property>
		<name>dfs.namenode.name.dir</name>
		<value>/data/nn</value>
	</property>
	
	<property>
		<name>dfs.namenode.hosts</name>
		<value>node1,node2,node3</value>
	</property>
	
	<property>
		<name>dfs.blocksize</name>
		<value>268435456</value>
	</property>
	
	<property>
		<name>dfs.namenode.handler.count</name>
		<value>100</value>
	</property>
	
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/data/dn</value>
	</property>
	
</configuration>
```

​		key：dfs.datanode.data.dir.perm

​		含义：hdfs文件系统，默认创建的文件权限设置

​		值：700，即：rwx------

​		key：dfs.namenode.name.dir

​		含义：NameNode元数据的存储位置值：/data/nn，在node1节点的/data/nn目录下key：dfs.namenode.hosts

​		含义：NameNode允许哪几个节点的DataNode连接（即允许加入集群）

​		值：node1、node2、node3，这三台服务器被授权

​		key：dfs.blocksize

​		含义：hdfs默认块大小

​		值：268435456（256MB）

​		key：dfs.namenode.handler.count

​		含义：namenode处理的并发线程数

​		值：100，以100个并行度处理文件系统的管理任务key：dfs.datanode.data.dir

​		含义：从节点DataNode的数据存储目录

​		值：/data/dn，即数据存放在node1、node2、node3，三台机器的/data/dn内

#### 准备数据目录

1. 在node1节点：

```
	mkdir -p /data/nn

	mkdir /data/dn
```

2. 在node2和node3节点：

```
	mkdir -p /data/dn
```



#### 分发Hadoop文件夹

```
	cd /export/server

	scp -r hadoop-3.3.4 node2:`pwd`/

	scp -r hadoop-3.3.4 node3:`pwd` /
```

#### 在node2和node3执行，为hadoop配置软链接

```
	ln -s /export/server/hadoop-3.3.4 /export/server/hadoop
```

#### 配置环境变量

```
	vim /etc/profile

	export HADOOP_HOME=/export/server/hadoop

	export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

	surce /etc/profile
```

#### 授权为hadoop用户

```
	chown -R hadoop:hadoop /data

	chown -R hadoop:hadoop /export
```

#### 格式化整个文件系统

```
	#确保以hadoop用户执行
	su - hadoop
	#格式化namenode 
	hadoop namenode -format
	#一键启动hdfs集群
	start-dfs.sh
    #一键关闭hdfs集群
	stop-dfs.sh

```

#### 查看HDFS WEBUI

http://node1:9870，即可查看到hdfs文件系统的管理网页。

### HDFS的Shell操作

#### 进程启停管理

```
$HADOOP_HOME/sbin/start-dfs.sh   一键启动HDFS集群
$HADOOP_HOME/sbin/stop-dfs.sh    一键关闭HDFS集群
```

执行原理：在执行此脚本的机器上，启动SecondaryNameNode

​				读取core-site.xml内容（fs.defaultFS项）

​				确认NameNode所在机器，启动NameNode

​				读取workers内容，确认DataNode所在机器，启动全部DataNode

```
此脚本可以单独控制所在机器的进程启停
hadoop-daemon.sh (start|status|stop) (namenode|secondarynamenode|datanode)
此程序也可以用以单独控制所在机器的进程的启停
hdfs --daemon (start|status|stop) (namenode|secondarynamenode|datanode)
```

#### 文件操作命令

​	HDFS同Linux系统一样，均是以/作为根目录的组织形式

​	如何区分呢？

```
Linux：file:///
HDFS：hdfs://namenode:port/
```

注意：协议头file:///或hdfs://node1:8020/可以省略

​			需要提供Linux路径的参数，会自动识别为file://			

​			需要提供HDFS路径的参数，会自动识别为hdfs://除非你明确需要写或不写会有BUG，否则一般不用写协议头

##### 创建文件夹

```
hadoop fs -mkdir [-p] <path> ...
hdfs dfs -mkdir [-p] <path> ...
	path为待创建的目录
	-p沿着路径创建父目录
	注意：不能直接创建文件
```

##### 查看指定目录下内容

```
hadoop fs -ls [-h] [-R] [<path> ...]
hdfs dfs -ls [-h] [-R] [<path> ...]
	path指定目录路径
	-h人性化显示文件size
	-R递归查看指定目录及其子目录
```

##### 上传文件到HDFS指定目录下

```
hadoop fs -put [-f] [-p] <localsrc> ...<dst>
hdfs dfs -put [-f] [-p] <localsrc> ...<dst>
	-f覆盖目标文件（已存在下）
	-p保留访问和修改时间，所有权和权限。
	localsrc本地文件系统（客户端所在机器）
	dst目标文件系统（HDFS）
```

##### 查看HDFS文件内容

```
hadoop fs -cat <src> ...
hdfs dfs -cat <src> ...
读取指定文件，显示在标准输出控制台。
hadoop fs -cat <src> | more
hdfs dfs -cat <src> | more
读取大文件可以使用管道符配合more
```

##### 下载HDFS文件

```
adoop fs -get [-f] [-p] <src> ...<localdst>
hdfs dfs -get [-f] [-p] <src> ...<localdst>
下载文件到本地文件系统指定目录，
	localdst必须是目录
    -f覆盖目标文件（已存在下）
	-p保留访问和修改时间，所有权和权限。
```
