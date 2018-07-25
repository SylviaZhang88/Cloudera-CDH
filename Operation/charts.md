# CDH Charts环境搭建

标签（空格分隔）： 大数据

---

[TOC]   
## cloudera CDH环境搭建 
```shell
# 略
```

## Druid安装 
```shell
# 参考链接
https://www.cnblogs.com/niejunlei/p/5977895.html
https://ask.hellobi.com/blog/ambition119/8280 

https://blog.csdn.net/fenghuibian/article/details/53216141
https://www.jianshu.com/p/852bb8cfed6b

# 安装
## zhy-cdh-1 
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




## Superset安装
```shell
# 参考链接
https://blog.csdn.net/qq273681448/article/details/75050513
https://blog.csdn.net/TG229dvt5I93mxaQ5A6U/article/details/79089975


# 安装
## zhy-cdh-3
yum makecache
sudo easy_install -i http://pypi.douban.com/simple/ pip
sudo easy_install pip
sudo yum install gcc libffi-devel python-devel python-pip python-wheel openssl-devel libsasl2-devel openldap-devel

#安装Superset 
pip install superset 
## 安装是可能会报错
## Command "python setup.py egg_info" failed with error code 1 in /tmp/pip-install-UDMRV_/pydruid/
## 安装最新版的setuptools
sudo python -m pip install --upgrade --force pip 
sudo pip install setuptools==40.0.0
pip install superset 

## gcc: error trying to exec 'cc1plus': execvp: 没有那个文件或目录
##    error: command 'gcc' failed with exit status 1
yum install gcc gcc-c++
pip install superset 


#创建管理员用户名和密码 
# fabmanager create-admin --app superset 
fabmanager create-admin --app superset --username admin --password admin --firstname admin --lastname admin --email admin@leadingsoft.com

#初始化Superset 
superset db upgrade 

#装载初始化样例数据 
superset load_examples 

#创建默认角色和权限 
superset init 

# 
pip install kylinpy

#启动Superset 
nohup superset runserver &

# 访问
http://192.168.0.53:8088

```

## Kylin
```shell
# 参考链接
http://kylin.apache.org/docs21/install/index.html

wget http://mirrors.tuna.tsinghua.edu.cn/apache/kylin/apache-kylin-2.4.0/apache-kylin-2.4.0-bin-cdh57.tar.gz

tar -xzvf apache-kylin-2.4.0-bin-cdh57.tar.gz
export KYLIN_HOME=/opt/apache-kylin-2.4.0-bin-cdh57

vi ~/.bashrc
# 添加export KYLIN_HOME=/opt/apache-kylin-2.4.0-bin-cdh57

cd apache-kylin-2.4.0-bin-cdh57
# 检查环境
./bin/check-env.sh

# 启动server
chown -R hdfs:hadoop apache-kylin-2.4.0-bin-cdh57
nohup bin/kylin.sh start &

http://192.168.0.53:7070/kylin
# username/password  ADMIN/KYLIN
## kylin.sh stop

./bin/sample.sh
# Sample cube is created successfully in project 'learn_kylin'.
# Restart Kylin Server or click Web UI => System Tab => Reload Metadata to take effect
```

## superset配置kylin
```shell
# 参考链接
https://github.com/Kyligence/kylinpy
http://lxw1234.com/archives/2018/03/904.htm
https://yq.aliyun.com/articles/181110

# Sources=>Databases 右上角加号添加

# SQLAlchemy URI
kylin://ADMIN:KYLIN@192.168.0.53:7070/learn_kylin?version=v1
# 点击Test Connection, 链接成功提示 Seems OK
# 可以在该页下方看到learn_kylin所有的表

# 勾选Expose in SQL Lab和 Allow Run Sync项


```

## Kylin配置CDH
```shell
# 参考链接 
http://kylin.apache.org/docs20/tutorial/cube_spark.html

# 拷贝cdh配置文件到指定路径
cd /opt/apache-kylin-2.4.0-bin-cdh57
mkdir hadoop_conf
cp /etc/hadoop/conf/hdfs-site.xml ./hadoop_conf/
cp /etc/hadoop/conf/yarn-site.xml ./hadoop_conf/
cp /etc/hadoop/conf/core-site.xml ./hadoop_conf/
cp /etc/hbase/conf/hdfs-site.xml ./hadoop_conf/
cp /etc/hbase/conf/hbase-site.xml ./hadoop_conf/
cp /etc/hive/conf/hive-site.xml ./hadoop_conf/

# 修改hadoop配置文件路径
vi conf/kylin.properties
kylin.env.hadoop-conf-dir=/opt/apache-kylin-2.4.0-bin-cdh57/hadoop_conf

# 重启   
bin/kylin.sh  stop
bin/kylin.sh  start
```


