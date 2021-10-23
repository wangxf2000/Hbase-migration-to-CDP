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

```default.test``` is a text table.

```default.test_orc``` is an orc table.

```default.test_parquet``` is a parquet table.

```default.test_kudu``` is a kudu table.

```default.hive_hbase_table``` is a hbase table.

![width=800](/images/cdh5_tables.jpg)

### CDH5 migrate to CDP Lab
With step by step to migrate CDH5 to CDP. we focus on Hive table in this lab.

#### Step 1: setup Replication Peers in CDP Cloudera Manager.
open CDP Cloudera Manager with 7180 port, Cloudera Manager->Replication->Peers

![width=800](/images/find_peers.jpg)

Click `Add Peer` in Peers Page

![width=800](/images/add_peer.jpg)

Input add peer information in Add Peer.

`Peer Name`:any name as your wish.

`Peer URL`: CDH5 cluster's Cloudera Manager's URL.

`Peer Admin Username`: CDH5 cluster's Cloudera Manager's username, need admin privedge.

`Peer Admin Password`: CDH5 cluster's Cloudera Manager's password.

![width=800](/images/add_peer_info.jpg)

Then click Add to test connectivity.

![width=800](/images/peer_test_connection.jpg)

if successful, then click Close. else need to modify the peer info then test it again. 

you can see the peer you just added it in Peer Page.

![width=800](/images/display_peer_info.jpg)

#### Step 2: Enable HDFS snapshots on Hive Warehouse directory on source cluster.
open CDH5 Cloudera Manager with 7180 port, Cloudera Manager-> Clusters-> Hive->Configuration, then find `hive.metastore.warehouse.dir` porperty. we'll enable `hive.metastore.warehouse.dir` value snapshots.

![width=800](/images/cdh5_warehouse_dir.jpg)

Cloudera Manager-> Clusters-> HDFS->File Browser to find `/user/hive/warehosue` directory

![width=800](/images/hdfs_file_browser_hive.jpg)

Click the `Enable Snapshots` button

![width=800](/images/enable_hive_snapshots.jpg)

Cloudera Manager will run Enable Snapshot Command and we can see Hive directory is enabled snapshots. Then we can Take Snapshot in File Browser.

![width=800](/images/enable_snapshot_and_take_snapshot.jpg)

Give Snapshot a Name then Click Ok, the cluster will build a snapshot for /user/hive/warehouse directory.

![width=800](/images/give_snapshot_name.jpg)

![width=800](/images/run_snapshot_command.jpg)

So you can see the snapshot in File Browser page.

![width=800](/images/hive_snapshot.jpg)

If you can't see HDFS file Browser manu, you can use hdfs snapshot command to do it.

#### Step 3: Create Replication Policy for replicate CDH5 tables to CDP.

open CDP Cloudera Manager with 7180 port, Cloudera Manager->Replication->Replication Policy

![width=800](/images/cdp_open_replication_policy.jpg)

Click `Create Replication Policy` Button in Replication Policies page. then choose Hive Replication Policy

![width=800](/images/open_hive_replication_policy_manu.jpg)

Begin to Create Hive Replication Policy now. 
##### Select the General tab to configure the following:

Use the `Name` field to provide a unique name for the replication policy.

Use the `Source` drop-down list to select the cluster with the Hive service you want to replicate. default is the source you add in Peer.

Use the `Destination` drop-down list to select the destination for the replication. default is this CDP cluster.

Based on the type of destination cluster you plan to use, select Use HDFS Destination. we use blank in this lab.

Permission: Do not import Sentry Permissions (Default)

Database:uncheck ALL, then input default test[\w]+

Run As Username: etl_user

Run on Peer as Username:using default


![width=800](/images/hive_replication_policy_general.jpg)

##### Select the Resources tab to configure the following: we use default in this lab.

![width=800](/images/hive_replication_policy_resource.jpg)

Scheduler Pool – (Optional) Enter the name of a resource pool in the field. The value you enter is used by the MapReduce Service you specified when Cloudera Manager executes the MapReduce job for the replication. The job specifies the value using one of these properties:
MapReduce – Fair scheduler: mapred.fairscheduler.pool
MapReduce – Capacity scheduler: queue.name
YARN – mapreduce.job.queuename
Maximum Map Slots and Maximum Bandwidth – Limits for the number of map slots and for bandwidth per mapper. The default is 100 MB.
Replication Strategy – Whether file replication should be static (the default) or dynamic. Static replication distributes file replication tasks among the mappers up front to achieve a uniform distribution based on file sizes. Dynamic replication distributes file replication tasks in small sets to the mappers, and as each mapper processes its tasks, it dynamically acquires and processes the next unallocated set of tasks.

##### Select the Advanced tab to specify an export location, modify the parameters of the MapReduce job that will perform the replication, and set other options. We use default in this lab. 

![width=800](/images/hive_replication_policy_advance.jpg)

Then Click `Save Policy` button to save policy.

The Replication Manager check the source cluster hdfs snapshots whether enabled. and display a confirm dialog.

![width=800](/images/create_replication_policy_confirm.jpg)

Click OK button. The cluster will run the replication job.

![width=800](/images/cdp_replication_command_running.jpg)



