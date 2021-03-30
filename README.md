# getmysql2clickhouse
##  getmysql2clickhouse概述

Clickhouse作为高效的面向列型数据库管理系统，近年来在国内受到了越来越多的科技公司的青睐。基于类似zookeeper分布式配置之后，clickhouse集群可以对数据表进行切分(shard)和备份(replica)，这极大地提升了数据管理的高效和安全。作为基本的应用，该程序模块的目的是基于python将MYSQL数据库表的数据导入到clickhouse数据库表中。

## 程序模块 getmysql2clickhouse 使用

在Linux终端里，通过````pip install getmysql2clickhouse````安装getmysql2clickhouse模块。该安装会自动安装````pymysql````和````clickhouse````两个python模块。安装完成进入python环境里，输入````import getmysql2clickhouse````即可使用该模块下集成的子程序。

主要子程序包括：
+ ````connectMySql(DBNAME, DBPASS, DBHOST="localhost", DBUSER="root")````
+ ````connectClickHouse(DBNAME, DBHOST="localhost", DBPORT="9000", DBUSER="default", DBPASS="")````
+ ````deliverData(mysqlConn, myTBNAME, ckClient, ckTBNAME)````

说明1：
````connectMySql````子程序目的是连接MYSQL数据库。
其输入包括：MYSQL数据库名称，数据库密码，数据库地址（默认为localhost）,数据库用户（默认为root）。
````connectMySql````输出记为````mysqlConn````是MYSQL的Connection object。 
````connectMySql````的屏幕输出：连接MYSQL的数据库成功时，返回“mysql数据库连接成功!”;否则返回“mysql数据库连接失败 + 原因”并终止程序。

说明2：
````connectClickHouse````子程序目的是连接clickhouse数据库。
其输入包括：clickhouse数据库名称，数据库地址（默认为localhost）,数据库端口(默认为9000)，数据库用户（默认为default），数据库密码（默认没有）。
````connectClickHouse````输出记为````ckClient````是clickhouse的Client object。
````connectClickHouse````的屏幕输出和````connectMySql````一样。

说明3：
````deliverData````子程序目的是将MYSQL表上的数据传递给clickhouse表。
其输入包括：````mysqlConn```` (MYSQL的Connection object), ````myTBNAME````(MYSQL的数据表名), ````ckClient```` (clickhouse的Client object)，````ckTBNAME````(clickhouse的数据表名)。
````deliverData````没有输出而其屏幕输出为“mySQL表的总条数是 XX”, “mySQL表的输出数据总大小(MB) XX”，“空余内存为(MB) XX”， “mySQL表的读取轮次为 XX”。这些屏幕输出主要目的是一方面通过估计表的总大小，一方面读取本地内存大小，使得每次都是根据内存的容量传递数据。

次要子程序包括（使用getmysql2clickhouse模块不需要，但是模块中出现的）：
+ ````getMemorySize()````：子程序目的是读取并返回本地内存
+ ````estimateBatchFetch(myCursor, TBNAME)````：子程序目的是计算并返回数据传递的轮次

## 程序模块 getmysql2clickhouse 实现发布的步骤

该程序模块的程序见 [getmysql2clickhouse的package](https://github.com/zhihaogong25/getmysql2clickhouse/blob/main/getmysql2clickhouse/__init__.py)。该程序模块通过[setup.py程序](https://github.com/zhihaogong25/getmysql2clickhouse/blob/main/setup.py)上传到<https://pypi.org>。通过那个网站进行发布。
上传的bash代码为： 1 ````python setup.py sdist build```` 2 ````python setup.py bdist_wheel --universal ```` 3 ````twine upload dist/*````

## 程序模块 getmysql2clickhouse 测试说明

1. 设置了MYSQL数据库和clickhouse单机（无切片无备份），getmysql2clickhouse模块可以传递数据。
2. 设置了MYSQL数据库和clickhouse伪集群（单机，2切片无备份），getmysql2clickhouse模块可以传递数据。

## 附录A: 通过docker搭建clickhouse集群(基于zookeeper分布式)
**1. 搭建zookeeper分布式**： 在root用户下，在目录````/app/````下通过````wget ````方式下载的zookeeper压缩包解包为目录````zookeeper````。在zookeeper目录下，创建目录````data````。拷贝目录````conf/zoo_sample.cfg````为````conf/zoo.cfg````。并修改````zoo.cfg````文件中的变量 ````dataDir````(用来存放数据的snapshot) 和 ````dataLogDir````(用来存放操作记录log)。这里我设为
````
dataDir=/apps/zookeeper/data
dataLogDir=/apps/zookeeper/log
````
在````zoo.cfg````中还需要添加局域网中的其他服务器节点（包括自己的这台服务器节点）的两个通信端口号。我这里为：
````
server.1=host133:2888:3888
server.2=host134:2888:3888
server.3=host136:2888:3888
````
其中"host133","host134", "host136" 与ip地址的对应关系说明会提前写在````/etc/hosts````文件内。
进一步，在目录````data````下，创建文件````myid````来标记这台服务器节点，文件中写这台节点的阿拉伯数字编号。例如对于"server.1"，使用命令 echo "1" > myid。
最后，在zookeeper目录下，运行
````
bin/zkServer.sh start
````
等到屏幕返回"STARTED"之后， 再使用命令 ````bin/zkServer.sh status````检查运行情况。对于单节点运行时，屏幕返回"mode: standalone"。 对于我这里的三节点情况即为"follower - leader - follower" 模式。
 
**2. 基于docker搭建clickhouse集群**：在不理解其底层原理的前提下，在虚拟机中利用docker来搭建clickhouse，可以部分避免一些未察觉的与环境不匹配。docker的安装可以在网上直接搜索，一般都能安装成功。搭建clickhouse集群基本步参考CSDN文章 [使用docker搭建clickhouse](https://blog.csdn.net/weixin_46918845/article/details/115133887?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-1&spm=1001.2101.3001.4242)。其大概的思路是先在一台节点上拉取clickhouse-server的镜像运行作为临时容器之后，将临时容器中的````/etc/clickhouse-server ````拷贝到节点根目录下来。并进行配置的修改，将该文件夹````scp````到所有的节点，然后删去原来临时容器。接着在每个节点上运行clickhouse-server的镜像，运行命令中会将容器中的配置目录定向到节点根目录中````/etc/clickhouse-server ````，同时暴露端口号和添加各个节点的host，这样就搭建完成。
但是这里我能搭建正常相互通信的clickhouse集群有四个地方需要强调：

1. 一直保持root用户，所有的命令都在root用户下完成。

2. 在修改confg.xml，我的修改如下：
 ````
<include_from>/etc/clickhouse-server/metrika.xml</include_from>
<remote_servers incl="clickhouse_remote_servers" />
<zookeeper incl="zookeeper-servers" optional="true" />
<macros incl="macros" />
 ````
3. 我这里没有修改user.xml。如果按照CSDN上的文章修改，clickhouse-server无法启动（通过命令````docker ps -a````中发现clickhouse-erver的状态"status"为"exited"，而非"up"）

4. 查询运行中的问题log使用命令 ````docker logs -f clickhouse-server```` 。

运行了参考网页上的步骤之后，集群下任意一个节点里clickhouse的system.clusters中可以检查集群是否有构建成功，如下：(这里是2个切片，2个副本)
![Capture](https://user-images.githubusercontent.com/17373280/112940925-67d78e80-9160-11eb-87ea-e9a821138376.JPG)

## 附录B: 在clickhouse中构建分布表
在mysql中我们建了一个数据表，其类型如下：
````
create table stock(
日期 date not null,
股票代码 varchar(10) not null,
名称 varchar(20) not null,
收盘价 float not null,
最高价 float not null,
最低价 float not null,
开盘价 float not null,
涨跌额 float not null,
涨跌幅 float not null,
换手率 float not null,
成交量 double not null,
成交金额 double not null,
总市值 double not null,
流通市值  double not null
)engine=csv;
````
在clickhouse中，对于每一个节点，我们都构建一个该节点的表。表的类型如下
````
CREATE TABLE `stock_replica` (
  `日期` Date,
  `股票代码` String,
  `名称` String,
  `收盘价` Float32,
  `最高价` Float32,
  `最低价` Float32,
  `开盘价` Float32,
  `涨跌额` Float32,
  `涨跌幅` Float32,
  `换手率` Float32,
  `成交量` Float64,
  `成交金额` Float64,
   `总市值` Float64,
   `流通市值` Float64
  ) ENGINE = MergeTree
ORDER BY (`日期`)
SETTINGS index_granularity = 8192;
````
