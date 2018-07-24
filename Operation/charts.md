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
http://kylin.apache.org/docs21/install/index.html

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
# Username [admin]: root
# User first name [admin]: root
# User last name [user]: root
# Email [admin@fab.org]: root@leadingsoft.com
# Password: root1234
# Repeat for confirmation: root1234
# Recognized Database Authentications.
# Admin User root created.

#初始化Superset 
superset db upgrade 

#装载初始化样例数据 
superset load_examples 

#创建默认角色和权限 
superset init 

#启动Superset 
nohup superset runserver &

# 访问
http://192.168.0.53:8088

```

## Kylin
```shell
wget http://mirrors.tuna.tsinghua.edu.cn/apache/kylin/apache-kylin-2.4.0/apache-kylin-2.4.0-bin-cdh57.tar.gz

tar -xzvf apache-kylin-2.4.0-bin-cdh57.tar.gz
export KYLIN_HOME=/opt/apache-kylin-2.4.0-bin-cdh57

cd apache-kylin-2.4.0-bin-cdh57
# 检查环境
./bin/check-env.sh

# 启动server
chown -R hdfs:hadoop apache-kylin-2.4.0-bin-cdh57
nohup bin/kylin.sh start &

http://192.168.0.53:7070/kylin
# username/password  ADMIN/KYLIN
## kylin.sh stop
```