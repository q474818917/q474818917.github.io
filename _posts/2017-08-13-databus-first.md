---
layout: post
title: Databus 首章 - mysql主从实例
categories: Blog
description: github example实战
keywords: Databus、mysql、binlog、elasticsearch
---

## [Databus](https://github.com/linkedin/databus)
+ 来源独立：Databus支持多种数据来源的变更抓取，包括Oracle和MySQL。
+ 可扩展、高度可用：Databus能扩展到支持数千消费者和事务数据来源，同时保持高度可用性。
+ 事务按序提交：Databus能保持来源数据库中的事务完整性，并按照事务分组和来源的提交顺寻交付变更事件。
+ 低延迟、支持多种订阅机制：数据源变更完成后，Databus能在毫秒级内将事务提交给消费者。同时，消费者使用Databus中的服务器端过滤功能，可以只获取自己需要的特定数据。
+ 无限回溯：对消费者支持无限回溯能力，例如当消费者需要产生数据的完整拷贝时，它不会对数据库产生任何额外负担。当消费者的数据大大落后于来源数据库时，也可以使用该功能。

## 数据一致性方案：
+ 应用驱动双向写：同时向数据库及MQ写入，一致性不可靠，较灵活
+ 数据库日志挖掘：基于数据库binlog，一致性可靠，不灵活

## Databus架构图
	类似于Search Index及Read Replica这些作为Databus的Consumer（类节点）使用的将是Client Library（客户端库）。当对一个主OLTP数据库做写操作时，连接了这个数据库的Relay们将会把改变存入Relay中；Databus这些被嵌入内存或者索引的Consumer将会把它从Relay或Bootstrap（引导程序）中取出，并且根据情况修改索引或者缓存，这就做到了根据源数据库的状态实时的更新索引。
![](http://cms.csdnimg.cn/article/201302/27/512da48083705.jpg)

## Databus Demo

### 数据库配置
+ 数据库版本5.6，支持checknum特性，需要关闭

```
log-bin=mysqlbin-log	# 开启binlog，前缀是mysqlbin-log
server_id=1001			# master服务器ID
binlog_format=row       # binlog的format是row，当然还有Statement,MiXED
binlog-row-image=full	
binlog_checksum=none	# 关闭checksum
```

+ 创建mysql数据库及表

```
CREATE DATABASE or_test;

DROP TABLE IF EXISTS `person`;

CREATE TABLE `person` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `first_name` varchar(120) DEFAULT NULL,
  `last_name` varchar(120) DEFAULT NULL,
  `birth_date` date DEFAULT NULL,
  `deleted` varchar(5) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

### 编译Databus
+ git clone https://github.com/linkedin/databus

```
cd ~/databus/databus2-example
```

+ 编辑 databus2-example-relay-pkg

需要修改databus2-example-relay-pkg/conf下的sources-or-person.json, 对应relay_or_person.properties.
其中sources-person.json是oracle下使用的

```
{
    "name" : "person",
    "id"  : 1,
    "uri" : "mysql://username%2Fpassword@localhost:3306/1/mysqlbin-log",
    "slowSourceQueryThreshold" : 2000,
    "sources" :
    [
        {
        "id" : 40,
        "name" : "com.linkedin.events.example.or_test.Person",
        "uri": "or_test.person",
        "partitionFunction" : "constant:1"
         }
    ]
}

```
databus2-example-relay-pkg/schemas_registry下的
com.linkedin.events.example.or_test.Person.1.avsc：mysql使用的avsc
com.linkedin.events.example.person.Person.1.avsc：oracle使用的avsc

+ 编辑 databus2-example-client-pkg

databus2-example-client-pkg/conf/client_person.properties默认不需要修改
修改databus2-example-client下的PersonClientMain.java

```
public class PersonClientMain
{
  static final String PERSON_SOURCE = "com.linkedin.events.example.or_test.Person"; //修改记录

  public static void main(String[] args) throws Exception
  {
    DatabusHttpClientImpl.Config configBuilder = new DatabusHttpClientImpl.Config();

    //Try to connect to a relay on localhost
    configBuilder.getRuntime().getRelay("1").setHost("localhost");
    configBuilder.getRuntime().getRelay("1").setPort(11115);
    configBuilder.getRuntime().getRelay("1").setSources(PERSON_SOURCE);

    //Instantiate a client using command-line parameters if any
    DatabusHttpClientImpl client = DatabusHttpClientImpl.createFromCli(args, configBuilder);

    //register callbacks
    PersonConsumer personConsumer = new PersonConsumer();
    client.registerDatabusStreamListener(personConsumer, null, PERSON_SOURCE);
    client.registerDatabusBootstrapListener(personConsumer, null, PERSON_SOURCE);

    //fire off the Databus client
    client.startAndBlock();
  }

}

```

+ 编译源代码
download ojdbc6.jar with version at 11.2.0.2.0 [here](http://www.oracle.com/technetwork/database/enterprise-edition/jdbc-112010-090769.html)
把jar包放入到databus/sandbox-repo/com/oracle/ojdbc6/11.2.0.2.0/ojdbc6-11.2.0.2.0.jar

```
gradle -Dopen_source=true assemble
```

### 启动relay、client
编译后databus根目录产生build文件夹

+ 启动relay

```
cd build/databus2-example-relay-pkg/distributions
tar -xvf databus2-example-relay-pkg.tar.gz
./bin/start-example-relay.sh or_person -Y ./conf/sources-or-person.json
```

+ 启动client

```
cd build/databus2-example-client-pkg/distributions
tar -xvf databus2-example-client-pkg.tar.gz
./bin/start-example-client.sh person
```

### 验证结果

+ dml 数据库
+ 查看event

```
curl -s http://localhost:11115/containerStats/inbound/events/total?pretty | grep -m1 numDataEvents
```
+ 发现错误可以查看日志：
	+ client.log、databus2-client-person.out
	+ relay.log、databus2-relay-or_person.out


