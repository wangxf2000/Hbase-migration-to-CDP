# hive-migration-to-CDP

## CDH5 migrate to CDP Base

### Environment
#### Source Cluster
##### deploy CDH5 cluster using https://github.com/wangxf2000/OneNodeCDHCluster
be care to choose CDH5 run script. 
Cloudera Manager: 5.16.2 
CDH : 5.16.2 
Kerberos：No 
Sentry：No 
main component: Yarn,HDFS,Hive,Hue,Impala,Kudu,Hbase 
![width=800](/images/CDH5_components.jpg)

#### Target Cluster
##### deploy CDP7 base cluster using https://github.com/wangxf2000/OneNodeCDPCluster 
Cloudera Manager ：7.4.4 
Cloudera Runtime : 7.1.7 
Kerberos ：Yes 
Ranger : Yes 
