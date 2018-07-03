# CDH环境搭建

标签（空格分隔）： 大数据

---

[TOC]   


## 一、规划概述
### 1.1 网络及域名  (示例IP及域名可修改，修改时注意修改执行步骤相关内容） 

| 主机名 | 域名 | eth0 |  备注 |  
|:----:|:----:|:----:|:----:|
| CDH-01 | CDH-01 | 192.168.0.210 |  `cloudera manager+CDH节点` |   
| CDH-02 | CDH-02 | 192.168.0.211 |  `CDH节点` | 
| CDH-03 | CDH-03 | 192.168.0.212 |  `CDH节点` | 
| local_repo | local_repo | 192.168.0.200 |  `本地源节点，使用公网安装时可不使用` | 

### 1.2 数据库服务及密码  

| 服务名 | 数据库用户 | 密码 | 备注 |  
|:----:|:----:|:----:|:----:| 
| hive  | hive | hive_password | - |
| amon | amon | amon_password | - |
| oozie | oozie | oozie_password | - |
| hue | hue | hue_password | - |
| scm | scm | scm_password | - |
			

## 二、物理环境配置
### 2.1 BIOX & Raid
- 配置Raid(略)

### 2.2 安装配置操作系统
- 安装方式：局域网PXE
- 操作系统：CentOS 7.2 64位
    - 用户名：root
    - 密码：root1234
- 配置：静态IP&主机名
- 配置IP&主机名过程（略）


## 三、系统环境搭建
### 3.1 安装CDH节点
#### 3.1.1 基础环境准备
- 配置集群主机域名
```shell
echo -e "192.168.0.210\tCDH-01" >> /etc/hosts
echo -e "192.168.0.211\tCDH-02" >> /etc/hosts
echo -e "192.168.0.212\tCDH-03" >> /etc/hosts

#使用本地源时，添加
echo -e "192.168.0.200\tlocal_repo" >> /etc/hosts
```

- 配置集群时间同步服务器  
```shell
# 示例同步时间服务器为公网服务器，可按需修改
yum install -y chrony
sed -i -e "s/^server .*/#&/" /etc/chrony.conf
sed -i -e "s/^allow .*/#&/" /etc/chrony.conf
cat >> /etc/chrony.conf << EOF
server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
stratumweight 0
driftfile /var/lib/chrony/drift
rtcsync
makestep 10 3
bindcmdaddress 127.0.0.1
bindcmdaddress ::1
keyfile /etc/chrony.keys
commandkey 1
generatecommandkey
noclientlog
logchange 0.5
logdir /var/log/chrony
allow 192.168.0.0/16
EOF

systemctl restart chronyd
sleep 10
chronyc sources -v


```  

#### 3.1.2 安装配置 
- 配置内核参数 
```shell
chmod +x /etc/rc.d/rc.local  

echo "echo 0 > /proc/sys/vm/swappiness" >> /etc/rc.d/rc.local 

echo "echo 0 > /proc/sys/vm/zone_reclaim_mode" >> /etc/rc.d/rc.local  

echo "echo 1048576 > /proc/sys/vm/min_free_kbytes"  >> /etc/rc.d/rc.local  

echo "echo never > /sys/kernel/mm/transparent_hugepage/defrag" >> /etc/rc.d/rc.local  

echo 0 > /proc/sys/vm/swappiness  

echo 0 > /proc/sys/vm/zone_reclaim_mode  

echo 1048576 > /proc/sys/vm/min_free_kbytes  

echo never > /sys/kernel/mm/transparent_hugepage/defrag  

ulimit -n 8192 
echo 'ulimit -SHn 8192' >> /etc/profile
echo "DefaultLimitCORE=infinity" >> /etc/systemd/system.conf
echo "DefaultLimitNOFILE=65536" >> /etc/systemd/system.conf
echo "Default LimitNPROC=65536" >> /etc/systemd/system.conf
echo "DefaultLimitCORE=infinity" >> /etc/systemd/user.conf
echo "DefaultLimitNOFILE=65536" >> /etc/systemd/user.conf
echo "Default LimitNPROC=65536" >> /etc/systemd/user.conf

systemctl daemon-reload
reboot

ulimit -a

setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config

getenforce

```

- 安装java
```shell
yum install java -y
rpm -qa | grep java-1 | xargs rpm -e --nodeps
################
###1、公网安装###
################
# 下载jdk-8u102-linux-x64.rpm（略）
rpm -ivh jdk-8u102-linux-x64.rpm  


##################
###2、本地源安装###
##################
rpm -ivh http://local_repo/repos/jdk/jdk-8u102-linux-x64.rpm

java -version
```



### 3.2 安装Cloudera Manager节点
#### 3.2.1 安装配置数据库(如使用已有配置的数据库，此步略)
- 安装数据库
```shell
yum install mariadb-server mysql-connector-java -y
```

- 配置数据库character
```shell
echo "character-set-server=utf8" >> /etc/my.cnf
```

- 启动服务
```
systemctl start mariadb
systemctl enable mariadb
```

- 数据通用安全配置
```shell
mysql_secure_installation
# 输入顺序： 
#       enter n y y y y
```

#### 3.2.2 安装Cloudera manager
- 安装cloudera manager
```shell
##############################################################
###1、公网安装（如网络条件不好，可将rpm包先下载到本地，再进行安装）###
##############################################################
rpm -ivh http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.8.1/RPMS/x86_64/cloudera-manager-daemons-5.8.1-1.cm581.p0.7.el7.x86_64.rpm
rpm -ivh http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.8.1/RPMS/x86_64/cloudera-manager-server-5.8.1-1.cm581.p0.7.el7.x86_64.rpm

#另外，使用公网安装时，如网络条件不好，建议所有CDH节点的cloudera-manager-agent包，也先进行下载安装，避免网络安装超时出错(默认是cloudera-manager管理页面操作安装)
wget http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.8.1/RPMS/x86_64/cloudera-manager-daemons-5.8.1-1.cm581.p0.7.el7.x86_64.rpm
wget http://archive.cloudera.com/cm5/redhat/7/x86_64/cm/5.8.1/RPMS/x86_64/cloudera-manager-agent-5.8.1-1.cm581.p0.7.el7.x86_64.rpm
yum install -y perl
rpm -ivh cloudera-manager-daemons-5.8.1-1.cm581.p0.7.el7.x86_64.rpm
yum install -y MySQL-python python-psycopg2 openssl-devel mod_ssl httpd /lib/lsb/init-functions fuse-libs fuse cyrus-sasl-gssapi cyrus-sasl-plain psmisc bind-utils 
rpm -ivh cloudera-manager-agent-5.8.1-1.cm581.p0.7.el7.x86_64.rpm

###################
###2、本地源安装###
###################
rpm -ivh http://local_repo/centos/7/cloudera/cm/5/RPMS/x86_64/cloudera-manager-daemons-5.8.1-1.cm581.p0.7.el7.x86_64.rpm
rpm -ivh http://local_repo/centos/7/cloudera/cm/5/RPMS/x86_64/cloudera-manager-server-5.8.1-1.cm581.p0.7.el7.x86_64.rpm

```


- 获取CDH parcel包
```shell

###################
###1、通过公网获取###
###################
wget http://archive.cloudera.com/cdh5/parcels/5.8.0/CDH-5.8.0-1.cdh5.8.0.p0.42-el7.parcel -P /opt/cloudera/parcel-repo/
wget http://archive.cloudera.com/cdh5/parcels/5.8.0/CDH-5.8.0-1.cdh5.8.0.p0.42-el7.parcel.sha1 -O /opt/cloudera/parcel-repo/CDH-5.8.0-1.cdh5.8.0.p0.42-el7.parcel.sha

wget https://archive.cloudera.com/gplextras5/parcels/5.8.0/GPLEXTRAS-5.8.0-1.cdh5.8.0.p0.39-el7.parcel -P /opt/cloudera/parcel-repo/
wget https://archive.cloudera.com/gplextras5/parcels/5.8.0/GPLEXTRAS-5.8.0-1.cdh5.8.0.p0.39-el7.parcel.sha1 -O /opt/cloudera/parcel-repo/GPLEXTRAS-5.8.0-1.cdh5.8.0.p0.39-el7.parcel.sha

#####################
###2、通过本地源获取###
#####################
wget http://local_repo/repos/cdh/CDH-5.8.0-1.cdh5.8.0.p0.42-el7.parcel -P /opt/cloudera/parcel-repo/
wget http://local_repo/repos/cdh/CDH-5.8.0-1.cdh5.8.0.p0.42-el7.parcel.sha1 -O /opt/cloudera/parcel-repo/CDH-5.8.0-1.cdh5.8.0.p0.42-el7.parcel.sha
wget http://local_repo/repos/gplextras/GPLEXTRAS-5.8.0-1.cdh5.8.0.p0.39-el7.parcel -P /opt/cloudera/parcel-repo/
wget http://local_repo/repos/gplextras/GPLEXTRAS-5.8.0-1.cdh5.8.0.p0.39-el7.parcel.sha1 -O /opt/cloudera/parcel-repo/GPLEXTRAS-5.8.0-1.cdh5.8.0.p0.39-el7.parcel.sha
```

- 创建服务数据库
```shell
####################################
###使用已有配置的数据库，执行以下命令###
####################################
mysql

create database scm DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON *.* TO 'scm'@'localhost' IDENTIFIED BY 'scm_password' WITH GRANT OPTION;
 GRANT ALL PRIVILEGES ON *.* TO 'scm'@'%' IDENTIFIED BY 'scm_password' WITH GRANT OPTION;

create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on hive.* TO 'hive'@'%' IDENTIFIED BY 'hive_password';
grant all on hive.* TO 'hive'@'localhost' IDENTIFIED BY 'hive_password';

create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'amon_password';
grant all on amon.* TO 'amon'@'localhost' IDENTIFIED BY 'amon_password';

create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie_password';
grant all on oozie.* TO 'oozie'@'localhost' IDENTIFIED BY 'oozie_password';

create database hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on hue.* TO 'hue'@'%' IDENTIFIED BY 'hue_password';
grant all on hue.* TO 'hue'@'localhost' IDENTIFIED BY 'hue_password';

flush privileges;

exit


################################
###使用本地数据库时，执行以下命令###
################################
mysql

create database hive DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on hive.* TO 'hive'@'%' IDENTIFIED BY 'hive_password';
grant all on hive.* TO 'hive'@'localhost' IDENTIFIED BY 'hive_password';

create database amon DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'amon_password';
grant all on amon.* TO 'amon'@'localhost' IDENTIFIED BY 'amon_password';

create database oozie DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie_password';
grant all on oozie.* TO 'oozie'@'localhost' IDENTIFIED BY 'oozie_password';

create database hue DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
grant all on hue.* TO 'hue'@'%' IDENTIFIED BY 'hue_password';
grant all on hue.* TO 'hue'@'localhost' IDENTIFIED BY 'hue_password';

flush privileges;

exit

```

- 创建cloudera manager数据库
```shell
####################################
###使用已有配置的数据库，执行以下命令###
####################################
/usr/share/cmf/schema/scm_prepare_database.sh mysql -h controller-vip --scm-host CDH-01 scm scm scm_password
#注：如使用已有配置的数据库，在cloudera manager页面上Database Setup时，需要修改为database所在主机的hostname，默认为cloudera manager主机名CDH-01

################################
###使用本地数据库时，执行以下命令###
################################
/usr/share/cmf/schema/scm_prepare_database.sh mysql -uroot scm scm scm_password
```

- 启动cloudera manager server服务
```shell
systemctl start cloudera-scm-server
```


### 3.3 安装配置CDH服务
#### 3.3.1 安装CDH服务
- Cloudera manager管理界面地址
```
http://192.168.0.210:7180
```
- 用户名密码
```
admin / admin
```

- 自定义存储库
```
http://local_repo/centos/7/cloudera/cm/5/
```

- CDH 安装（页面操作）
```shell
注意事项： 

# 其中在选择CDH包位置的时候需要制定使用自定义的本地库（使用公网安装时不需填写）
http://local_repo/centos/7/cloudera/cm/5/

# 配置hdfs dn路径时，需要提前在系统中进行路径的创建和绑定（磁盘使用可按需修改）
mkfs.xfs /dev/vdb
mkfs.xfs /dev/vdc
mkfs.xfs /dev/sdd
mkdir -p /dfs/nn/1
mkdir -p /dfs/dn/2
mkdir -p /dfs/dn/3

echo "/dev/vdb /dfs/nn/1 xfs defaults,noatime,nodiratime 0 0" >> /etc/fstab
echo "/dev/vdb /dfs/dn/1 xfs defaults,noatime,nodiratime 0 0" >> /etc/fstab
echo "/dev/sdd /dfs/dn/3 xfs defaults,noatime,nodiratime 0 0" >> /etc/fstab

mount -a

chown hdfs:hadoop /dfs/dn/1
chown hdfs:hadoop /dfs/dn/2
chown hdfs:hadoop /dfs/dn/3
```

- 远程 Parcel 存储库 URL(安装时不必使用)
```
https://archive.cloudera.com/cdh5/parcels/5/
https://archive.cloudera.com/gplextras5/parcels/5/
```

#### 3.3.2 配置CDH服务
- 配置Hue Service-Wide为hbase thrift server

- 配置HDFS NN HA 
```
配置时指定active namenode位于CDH-01；

standby namenode位于CDH-02；

journal node位于CDH-01、CDH-02、CDH-03；

创建journal node目录为/dfs/jn，可以不用提前在主机中创建目录，会自动创建。
```

- 配置HDFS DN topology
```
配置服务器所属rack
CDH-01 rack1
CDH-02 rack2
CDH-03 rack3
```


#### 3.3.3 调优
```
略 
```


#### 4 常见错误参考
```
http://microdevsys.com/wp/cloudera-manager-installation-issues/

chmod 755 /var/lib/zookeeper
chown zookeeper.zookeeper /var/lib/zookeeper
ls -altrid /var/lib/zookeeper
setfacl -m "u:zookeeper:rwx,g:zookeeper:rwx" /var/lib/zookeeper/
getfacl /var/lib/zookeeper/
mkdir /var/lib/zookeeper/version-2
chown zookeeper.zookeeper /var/lib/zookeeper/version-2
```






