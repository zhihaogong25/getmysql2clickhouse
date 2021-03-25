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

## 程序模块 getmysql2clickhouse 实现原理
该程序模块见
