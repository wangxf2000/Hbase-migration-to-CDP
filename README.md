# HBase-migration-to-CDP

## Using Hbase Snapshot & replication to migrate hbase data from CDH5 to CDP Base

### Environment
#### Source Cluster
##### deploy CDH5 cluster using https://github.com/wangxf2000/OneNodeCDHCluster
be care to choose CDH5 run script. 

Cloudera Manager: 5.16.2 

CDH : 5.16.2 

Kerberos：No 

Sentry：No 

Main components: Yarn,HDFS,Hbase,Hue,Impala,Kudu,Hbase 

![width=800](/images/CDH5_components.jpg)

#### Target Cluster
##### deploy CDP7 base cluster using https://github.com/wangxf2000/OneNodeCDPCluster.
Cloudera Manager ：7.4.4 

Cloudera Runtime : 7.1.7 

Kerberos ：Yes 

Ranger : Yes

Main components:Yarn,HDFS,hive,tez,impala, Hive on Tez,kudu,hbase,ranger

![width=800](/images/CDP_components.jpg)


### Data preparation
Use HBase's PerformanceEvaluation to generate a table in SNAPPY format
hbase org.apache.hadoop.hbase.PerformanceEvaluation --compress=SNAPPY --size=1 sequentialWrite 10
check the HBase table info and size
```hbase(main):003:0> desc 'TestTable'
Table TestTable is ENABLED
TestTable
COLUMN FAMILIES DESCRIPTION
{NAME => 'info', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE', DATA_BLOCK_ENCODING => 'NONE', TTL => '
FOREVER', COMPRESSION => 'SNAPPY', MIN_VERSIONS => '0', BLOCKCACHE => 'true', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.0530 seconds ```

![width=800](/images/source_hbase_table_desc.png)


check the Hbase table size 
```hadoop fs -du -h /hbase/data/default/
257.5 M  257.5 M  /hbase/data/default/TestTable
```

![width=800](/images/source_hbase_table_size.png)

### CDH5 migrate to CDP Lab
With step by step to migrate CDH5 to CDP. we focus on Hbase table in this lab.




