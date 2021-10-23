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

Main components: Yarn,HDFS,Hive,Hue,Impala,Kudu,Hbase 

![width=800](/images/CDH5_components.jpg)

#### Target Cluster
##### deploy CDP7 base cluster using https://github.com/wangxf2000/OneNodeCDPCluster 
Cloudera Manager ：7.4.4 

Cloudera Runtime : 7.1.7 

Kerberos ：Yes 

Ranger : Yes 

Main components:Yarn,HDFS,hive,tez,impala, Hive on Tez,kudu,hbase,ranger


#### Data preparation
prepare CDH5 cluster data for migraion with https://github.com/wangxf2000/buildTestdata

this github will initialize hive/kudu/hbase data.

`default.test` is a text table.

`default.test_orc` is an orc table.

`default.test_parquet` is a parquet table.

`default.test_kudu` is a kudu table.

`default.hive_hbase_table` is a hbase table.

![width=800](/images/cdh5_tables.jpg)
