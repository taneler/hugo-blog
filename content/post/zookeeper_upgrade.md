+++
title = 'Zookeeper 版本升级'
date = 2024-07-10T14:57:05+08:00
draft = false
slug = '1fc9fcbc-1454-4cef-bb14-151322ed496f'
+++

Zookeeper 集群如果是 3 个节点以上的集群，那么可以采用不停业务滚动升级的方式进行版本升级，先对 `follower` 节点进行升级，最后对 `leader` 节点进行升级，可以使用以下命令查询当前节点状态。

```bash
~/app/apache-zookeeper-3.6.3-bin/bin/zkServer.sh status
```

将新版本的安装包上传到服务器进行解压到安装目录中。

```bash
tar -xzvf apache-zookeeper-3.8.4-bin.tar.gz -C ~/app
```

复制配置文件和相关依赖到新版本的安装目录中。

```bash
cp ~/app/apache-zookeeper-3.6.3-bin/lib/{kafka-clients-3.2.0.jar,lz4-java-1.8.0.jar} ~/app/apache-zookeeper-3.8.4-bin/lib
cp ~/app/apache-zookeeper-3.6.3-bin/conf/{zk_server_jaas.conf,zoo.cfg,java.env} ~/app/apache-zookeeper-3.8.4-bin/conf
```

修改 `~/app/apache-zookeeper-3.8.4-bin/conf/zoo.cfg` 配置文件中的 Zookeeper 数据保存目录。

```bash
sed -i 's/3.6.3/3.8.4/g' ~/app/apache-zookeeper-3.8.4-bin/conf/zoo.cfg
```

根据 `zoo.cfg` 配置文件中的 `dataDir` 配置项决定数据目录路径，例如文档中的集群数据目录为安装目录下的 `data` 目录，修改完成后，创建数据目录，并将 `myid` 文件复制到数据目录下。

```bash
mkdir ~/app/apache-zookeeper-3.8.4-bin/data
cp ~/app/apache-zookeeper-3.6.3-bin/data/myid ~/app/apache-zookeeper-3.8.4-bin/data
```

停止旧版本实例的运行。

```bash
~/app/apache-zookeeper-3.6.3-bin/bin/zkServer.sh stop
```

启动新版本实例的运行。

```bash
~/app/apache-zookeeper-3.8.4-bin/bin/zkServer.sh start
```

检查实例运行状态。

```bash
~/app/apache-zookeeper-3.8.4-bin/bin/zkServer.sh status
```

按照上面步骤依次完成 `follower` 节点以及 `leader` 节点的升级后，检查 Kafka 之类依赖 Zookeeper 的组件是否正常。

```bash
kafka-topics.sh \\
	--command-config /data/aspire/kafka/kafka_2.13-3.2.0/config/sasl_command_config.properties \\
	--bootstrap-server 10.17.8.97:9092,10.17.8.98:9092,10.17.8.99:9092 \\
	--list
```

如果 Kafka 集群没有启用 SASL 验证，那么可以去除上面命令中的 `--command-config` 行。
