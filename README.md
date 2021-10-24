# hive-migration-to-CDP

## Using CDP Replication Manager to migrate hive data from CDH5 to CDP Base

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


If you can't see HDFS file Browser manu, you can use hdfs snapshot command to do it.

#### step 3: Replicating from unsecure to secure clusters
To enable replication from an unsecure cluster to a secure cluster, you need a user that exists on all the hosts on both the source cluster and destination cluster. Specify this user in the Run As Username field when you create a replication schedule.

On a host in the source or destination cluster, the following command creates a user named milton:
```sudo -u hdfs hdfs dfs -mkdir -p /user/etl_user```

Set the permissions for the user directory ,the following command makes milton the owner of the milton directory:

```sudo -u hdfs hdfs dfs -chown etl_user:hadoop /user/etl_user```

Create the local user 'etl_user` in source cluster:

```useradd -p `openssl passwd -1 -salt 'cloudera' cloudera` etl_user```

Repeat this process for all hosts in the source and destination clusters so that the user and group exists on all of them.

After you complete this process, specify the user you created in the Run As Username field when you create a replication policy.

we do this step in `Data preparation` part.

#### Step 4: Hive replication in dynamic environments
To use Replication Manager for Hive replication in environments where the Hive Metastore changes, such as when a database or table gets created or deleted, additional configuration is needed.

Open the Cloudera Manager Admin Console.

Search for the HDFS Client Advanced Configuration Snippet (Safety Valve) for hdfs-site.xml property on the source cluster.

Add the following properties:

Name: replication.hive.ignoreDatabaseNotFound

Value: true

Name: replication.hive.ignoreTableNotFound

value: true

![width=800](/images/hive_replication_env.jpg)
Save the changes.

Restart the HDFS service.

#### step 4: Provide hdfs user permission to "all-database, table, column" in hdfs under the Hadoop_SQL section in Ranger in CDP cluster
Log in to Ranger Admin UI. the username and password maybe admin/BadPass#1

![width=800](/images/ranger_admin_ui.jpg)

Provide hdfs user permission to "all-database, table, column" in hdfs under the Hadoop_SQL section.

![width=800](/images/ranger_hdfs_policy1.jpg)

![width=800](/images/ranger_hdfs_policy.jpg)

#### Step 5: Create Replication Policy for replicate CDH5 tables to CDP.

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

Click OK button. 

we can see several manu in replication policies job, like dry run,run now and so on.

![width=800](/images/job_run_manu.jpg)

you can dry run first to check whether it works.

![width=800](/images/dry_run_replication_job.jpg)

Then you can click run now button to run it now. The cluster will run the replication job.

![width=800](/images/run_replication_job.jpg)

![width=800](/images/run_replication_job_completely.jpg)

Now we can check the replication result.

![width=800](/images/check_replication_table_list.jpg)

![width=800](/images/check_replication_table_records.jpg)

Now we finish Hive migrate from CDH5 to CDP lab.

### Reference: https://docs.cloudera.com/cdp-private-cloud-upgrade/latest/data-migration/topics/cdp-data-migration-replication-manager-to-cdp-data-center.html

## Using CDP Hive on Tez tools to migrate hive data to CDP Base, this works on CDH5/CDH6/HDP3/Apache Hive.
The Environment is the same as last lab.

### Step 1: drop the target database and tables in CDP.
drop the database and tables in hue.
``` drop table testdata.test;
drop table testdata.test_parquet;
drop table testdata.test_orc;
drop database testdata;
```

you need to drop table first, then drop the database.

### step 2:Preparing tables for migration
login source cluster in shell.

Obtain the Hive Upgrade Check tool.Download the Hive Upgrade Check tool from the Community-based github location.

```git clone https://github.com/dstreev/cloudera_upgrade_utils.git```


Follow instructions in the github readme to run the tool.
The Hive Upgrade Check (v.2.3.5.6+) will create a yaml file (hsmm_<name>.yaml) identifying databases and tables that require attention.
Follow instructions in prompts from the Hive Upgrade Check tool to resolve problems with the tables.
At a minimum, you must run the following processes described in the github readme:
process ID 1 Table / Partition Location Scan - Missing Directories
process id 3 Hive 3 Upgrade Checks - Managed Non-ACID to ACID Table Migrations


