检查异常文件
sudo -u hdfs hdfs fsck /
sudo -u hdfs hdfs fsck / | egrep -v '^\.+$' | grep -v eplica

找到异常文件位置
hdfs fsck /path/to/corrupt/file -locations -blocks -files
> hdfs fsck /hbase/archive/data/default/mtms_scd/f8cc813954beeed516c20abf837e95bf/f/04a14e1ee9de472f803fbb86ad11ff0f  -locations -blocks -files 
> Connecting to namenode via http://IC-CDH-R1-1:50070 
> FSCK started by root (auth:SIMPLE) from /192.168.20.104 for path /hbase/archive/data/default/mtms_scd/f8cc813954beeed516c20abf837e95bf/f/04a14e1ee9de472f803fbb86ad11ff0f at Tue Jul 03 10:23:25 CST 2018
/hbase/archive/data/default/mtms_scd/f8cc813954beeed516c20abf837e95bf/f/04a14e1ee9de472f803fbb86ad11ff0f 46344876 bytes, 1 block(s):  OK 
0. BP-1711507351-192.168.10.5-1515394480336:blk_1074008410_267712 len=46344876 Live_repl=3 [DatanodeInfoWithStorage[192.168.20.104:50010,DS-fb8e8244-2e4d-44df-bd41-f176b9df05fc,DISK], DatanodeInfoWithStorage[192.168.20.105:50010,DS-b8cd56d3-0966-4515-a07e-97b235e18800,DISK], DatanodeInfoWithStorage[192.168.20.106:50010,DS-a05820ee-af9d-4e61-8c29-b53d9fc4d2db,DISK]] 
>  
> Status: HEALTHY 
> Total size:	46344876 B 
>  Total dirs:	0 
> Total files:	1 
 Total symlinks:		0 
 Total blocks (validated):	1 (avg. block size 46344876 B) 
 Minimally replicated blocks:	1 (100.0 %) 
 Over-replicated blocks:	0 (0.0 %)
 Under-replicated blocks:	0 (0.0 %)
 Mis-replicated blocks:		0 (0.0 %)
 Default replication factor:	3
 Average block replication:	3.0
 Corrupt blocks:		0
 Missing replicas:		0 (0.0 %)
 Number of data-nodes:		6
 Number of racks:		1
FSCK ended at Tue Jul 03 10:23:25 CST 2018 in 1 milliseconds 
> 
> The filesystem under path '/hbase/archive/data/default/mtms_scd/f8cc813954beeed516c20abf837e95bf/f/04a14e1ee9de472f803fbb86ad11ff0f' is HEALTHY


删除异常块
hdfs fsck / -delete
