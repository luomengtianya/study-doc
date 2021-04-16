#### 简介
```text
Solr是Apache Lucene专案的开源企业搜寻平台。
其主要功能包括全文检索、命中标示、分面搜寻、动态聚类、资料库整合，以及富文字的处理。
Solr是高度可延伸的，并提供了分散式搜寻和索引复制。
Solr是最流行的企业级搜寻引擎，Solr 4还增加了NoSQL支援。
```

#### 资源
* [官网](https://solr.apache.org)  
* [v8.8文档](https://solr.apache.org/guide/8_8/)  


#### 依赖环境
* [java8及以上版本](https://www.oracle.com/java/technologies/oracle-java-archive-downloads.html)

#### 目录结构
```text
bin：solr的运行脚本

contrib：solr的一些软件/插件，用于增强solr的功能。

dist：该目录包含jar文件，以及相关的依赖文件。

docs：solr的API文档

example：solr工程示例：

licenses：solr相关引用的一些许可信息

Server:solr的核心，可以看成是一个数据库里面有多个实例
```

#### 按照示例启动solr
执行启动命令
```ssh
./bin/solr start -e cloud
```

提示选择节点数量
```text
*** [WARN] ***  Your Max Processes Limit is currently 63068. 
 It should be set to 65000 to avoid operational disruption. 
 If you no longer wish to see this warning, set SOLR_ULIMIT_CHECKS to false in your profile or solr.in.sh

Welcome to the SolrCloud example!

This interactive session will help you launch a SolrCloud cluster on your local workstation.
To begin, how many Solr nodes would you like to run in your local cluster? (specify 1-4 nodes) [2]:
```

这个提示是询问要运行多少个节点。注意最后一行末尾的[2]；这是默认的节点数。若是不需要修改默认数据，按 `enter` 键即可

```text
3
Ok, let's start up 3 Solr nodes for your example SolrCloud cluster.
Please enter the port for node1 [8983]: 

Please enter the port for node2 [7574]: 

Please enter the port for node3 [8984]:
```
若是想要3各节点则输入3即可，后续会提示输入每个阶段的端口，后面的数字是默认端口，不需要修改则按 `enter` 键即可。  
但是启动会报错
```text
*** [WARN] ***  Your Max Processes Limit is currently 63068. 
 It should be set to 65000 to avoid operational disruption. 
 If you no longer wish to see this warning, set SOLR_ULIMIT_CHECKS to false in your profile or solr.in.sh
WARNING: Starting Solr as the root user is a security risk and not considered best practice. Exiting.
         Please consult the Reference Guide. To override this check, start with argument '-force'
```
这是建议我们不直接使用root用户启动，要是确定用root用户启动则在后面添加参数 `-force` 即可

* 建立新用户
添加用户
```ssh
useradd -m solr -p solr
```
把`solr-8.8.2`相关文件夹赋权给`solr`用户
```ssh
chown -R solr /var/local/solr-8.8.2/
```
切换用户
```ssh
su solr
```
`root`用户切换时不需要输入密码，切换的`solr`用户有部分权限  

再次启动`solr`
```text
Started Solr server on port 8984 (pid=23742). Happy searching!

INFO  - 2021-04-13 19:45:19.968; org.apache.solr.common.cloud.ConnectionManager; Waiting for client to connect to ZooKeeper
INFO  - 2021-04-13 19:45:19.987; org.apache.solr.common.cloud.ConnectionManager; zkClient has connected
INFO  - 2021-04-13 19:45:19.988; org.apache.solr.common.cloud.ConnectionManager; Client is connected to ZooKeeper
INFO  - 2021-04-13 19:45:20.006; org.apache.solr.common.cloud.ZkStateReader; Updated live nodes from ZooKeeper... (0) -> (3)
INFO  - 2021-04-13 19:45:20.023; org.apache.solr.client.solrj.impl.ZkClientClusterStateProvider; Cluster at localhost:9983 ready

Now let's create a new collection for indexing documents in your 3-node cluster.
Please provide a name for your new collection: [gettingstarted] 
```
启动完成后，系统将提示您创建一个用于索引数据的集合
```ssh
How many shards would you like to split solrtest into? [2]

How many replicas per shard would you like to create? [2] 

Please choose a configuration for the solrtest collection, available options are:
_default or sample_techproducts_configs [_default] 

Created collection 'solrtest' with 2 shard(s), 2 replica(s) with config-set 'solrtest'

Enabling auto soft-commits with maxTime 3 secs using the Config API

POSTing request to Config API: http://localhost:8983/solr/solrtest/config
{"set-property":{"updateHandler.autoSoftCommit.maxTime":"3000"}}
Successfully set-property updateHandler.autoSoftCommit.maxTime to 3000
```
此时我们创建了一个`3`实例,`2`分片(平均分配索引数据),`2`副本(用于故障转移)的solr集群

若是不想要集群，直接单机启动即可
```ssh
./bin/solr start
```
查看启动的进程，发现启动参数如下
```text
java -server -Xms512m -Xmx512m -XX:+UseG1GC -XX:+PerfDisableSharedMem -XX:+ParallelRefProcEnabled 
-XX:MaxGCPauseMillis=250 -XX:+UseLargePages -XX:+AlwaysPreTouch -XX:+ExplicitGCInvokesConcurrent 
-verbose:gc -XX:+PrintHeapAtGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps 
-XX:+PrintGCTimeStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime 
-Xloggc:/var/local/solr-8.8.2/example/cloud/node1/solr/../logs/solr_gc.log 
-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=9 -XX:GCLogFileSize=20M 
-Dsolr.jetty.inetaccess.includes= -Dsolr.jetty.inetaccess.excludes= 
-DzkClientTimeout=30000 -DzkRun -Dsolr.log.dir=/var/local/solr-8.8.2/example/cloud/node1/solr/../logs 
-Djetty.port=8983 -DSTOP.PORT=7983 -DSTOP.KEY=solrrocks -Duser.timezone=UTC 
-XX:-OmitStackTraceInFastThrow -Djetty.home=/var/local/solr-8.8.2/server 
-Dsolr.solr.home=/var/local/solr-8.8.2/example/cloud/node1/solr 
-Dsolr.data.home= -Dsolr.install.dir=/var/local/solr-8.8.2 
-Dsolr.default.confdir=/var/local/solr-8.8.2/server/solr/configsets/_default/conf 
-Dlog4j.configurationFile=/var/local/solr-8.8.2/server/resources/log4j2.xml -Xss256k 
-Dsolr.log.muteconsole -XX:OnOutOfMemoryError=/var/local/solr-8.8.2/bin/oom_solr.sh 8983 
/var/local/solr-8.8.2/example/cloud/node1/solr/../logs -jar start.jar --module=http
```
此数据可以在solr的控制台中查看 `http://localhost:8983/solr/#/`  
关闭solr
```text
bin/solr stop -p 8983
or
bin/solr stop -all
```
自定义启动
```text
Creating Solr home directory /var/local/solr-8.8.2/example/cloud/node1/solr
Cloning /var/local/solr-8.8.2/example/cloud/node1 into
   /var/local/solr-8.8.2/example/cloud/node2
Cloning /var/local/solr-8.8.2/example/cloud/node1 into
   /var/local/solr-8.8.2/example/cloud/node3

Starting up Solr on port 8888 using command:
"bin/solr" start -cloud -p 8888 -s "example/cloud/node1/solr"
  
Starting up Solr on port 7574 using command:
"bin/solr" start -cloud -p 7574 -s "example/cloud/node2/solr" -z localhost:9888

Starting up Solr on port 8984 using command:
"bin/solr" start -cloud -p 8984 -s "example/cloud/node3/solr" -z localhost:9888
```




#### 关于ZooKeeper
solr内部集了ZooKeeper，当solr启动的时候ZooKeeper也会启动，solr关闭的时候ZooKeeper也会关闭，它不提供任何故障转移，依赖于它的任何分片或Solr实例也无法彼此通信，因此生产中最好使用外部的ZooKeeper

[zookeeper官网](https://zookeeper.apache.org/)

若是单机测试时不想自己启动ZooKeeper，可以修改配置文件下的`zoo.cfg`文件，放开指定端口的配置，这样所有的solr启动实例都会注册到这个zookeeper中(启动第二次会报错，然后注册已启动的zk实例中)


#### 同步数据库数据



#### 添加插件
* 添加拼音分词器






















