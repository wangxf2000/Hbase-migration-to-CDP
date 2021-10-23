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
##### deploy CDP7 base cluster using https://github.com/wangxf2000/OneNodeCDPCluster ，this cluster also with CDP license for using replication manager.
Cloudera Manager ：7.4.4 

Cloudera Runtime : 7.1.7 

Kerberos ：Yes 

Ranger : Yes

Main components:Yarn,HDFS,hive,tez,impala, Hive on Tez,kudu,hbase,ranger

![width=800](/images/CDP_components.jpg)


### Data preparation
prepare CDH5 cluster data for migraion with https://github.com/wangxf2000/buildTestdata

this github will initialize hive/kudu/hbase data.

`default.test` is a text table.

`default.test_orc` is an orc table.

`default.test_parquet` is a parquet table.

`default.test_kudu` is a kudu table.

`default.hive_hbase_table` is a hbase table.

![width=800](/images/cdh5_tables.jpg)

### CDH5 migrate to CDP Lab
With step by step to migrate CDH5 to CDP. we focus on Hive table in this lab.

#### Step 1: setup Replication Peers in CDP Cloudera Manager.
open CDP Cloudera Manager with 7180 port, Cloudera Manager->Replication->Peers

![width=800](/images/find_peers.jpg)

Click Add Peer in Peers Page

![width=800](/images/add_peer.jpg)

Input add peer information in Add Peer.

Peer Name:any name as your wish.

Peer URL: CDH5 cluster's Cloudera Manager's URL.

Peer Admin Username: CDH5 cluster's Cloudera Manager's username, need admin privedge.

Peer Admin Password: CDH5 cluster's Cloudera Manager's password.

![width=800](/images/add_peer_info.jpg)

Then click Add to test connectivity.

![width=800](/images/peer_test_connection.jpg)

if successful, then click Close. else need to modify the peer info then test it again. 

you can see the peer you just added it in Peer Page.

![width=800](/images/display_peer_info.jpg)

#### Step 2:


