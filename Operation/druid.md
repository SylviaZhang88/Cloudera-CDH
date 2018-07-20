# CDH Charts环境搭建

标签（空格分隔）： 大数据

---

[TOC]   
## cloudera CDH环境搭建 
```shell
略
```

## Druid安装 
参考链接
```shell
https://www.cnblogs.com/niejunlei/p/5977895.html
https://ask.hellobi.com/blog/ambition119/8280 
```

```shell
mkfs.xfs /dev/sdd
mkdir /druid

echo "/dev/sdd /druid xfs defaults,noatime,nodiratime 0 0" >> /etc/fstab
mount -a

wget http://static.druid.io/artifacts/releases/druid-0.12.1-bin.tar.gz

tar -zxvf druid-0.12.1-bin.tar.gz 

cd druid-0.12.1

mv conf-quickstart conf

cd conf/druid/_common/

cp common.runtime.properties bk.common.runtime.properties


vi common.runtime.properties
druid.extensions.loadList=["druid-hdfs-storage","mysql-metadata-storage"]
 
druid.extensions.hadoopDependenciesDir=/opt/druid/druid-0.12.1/hadoop-dependencies
druid.zk.service.host=zhy-cdh-1:2181,zhy-cdh-3:2181,zhy-cdh-2:2181
druid.zk.paths.base=/druid

# For MySQL:
druid.metadata.storage.type=mysql
druid.metadata.storage.connector.connectURI=jdbc:mysql://localhost:3306/druid?characterEncoding=UTF-8
druid.metadata.storage.connector.user=druid
druid.metadata.storage.connector.password=123456

# For HDFS:
druid.storage.type=hdfs
druid.storage.storageDirectory=/druid/segments

# For HDFS:
druid.indexer.logs.type=hdfs
druid.indexer.logs.directory=/druid/indexing-logs

 


 

```
