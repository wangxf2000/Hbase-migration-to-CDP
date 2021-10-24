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


![width=800](/images/drop_tables&databases.jpg)

### step 2:Preparing tables for migration
login source cluster in shell.

Obtain the Hive Upgrade Check tool.Download the Hive Upgrade Check tool from the Community-based github location.

```
wget https://github.com/dstreev/cloudera_upgrade_utils/releases/download/2.3.6.6-SNAPSHOT/hive-sre-dist.tar.gz
tar zxvf hive-sre-dist.tar.gz
```

![width=800](/images/get_hive_check_source.jpg)

distribute hive_sre tools.Switch to root user then run the command:
```
hive-sre-install/setup.sh
```
![width=800](/images/setup_hive_sre_tools.jpg)

Then check the hive-sre-cli and hive-sre command whether work.
```
export HADOOP_USER_NAME=hdfs
hdfs dfs -mkdir /user/root
hdfs dfs -chown root:hadoop /user/root
hive-sre-cli
hive-sre
```

![width=800](/images/check_hive_src_cli_work.jpg)
![width=800](/images/check_hive_src_cli_work2.jpg)
![width=800](/images/check_hive_src_work.jpg)

create db_cfg.yaml file for hive table check.
```
# Required to connect to Metastore RDBMS.  RDBMS driver needs to be included in the classpath
metastore_direct:
  uri: "jdbc:mysql://ip-10-0-0-154.us-west-1.compute.internal:3306/metastore" 
  type: MYSQL
  # Needed for Oracle Connections to pick the right schema for hive.
  # initSql: "ALTER SESSION SET CURRENT_SCHEMA=metastore"
  connectionProperties:
    user: "hive"
    password: "cloudera"
  connectionPool:
    min: 3
    max: 5
# Control the number of threads to run scans with.  Should not exceed host core count.
# Increase parallelism will increase HDFS namenode pressure.  Advise monitoring namenode
# RPC latency while running this process.
parallelism: 4
queries:
  db_tbl_count:
    parameters:
      dbs:
        override: "%"
```
you can modify the information as your env. reference:https://github.com/dstreev/cloudera_upgrade_utils/blob/master/hive-sre/config.md
![width=800](/images/create_db_cfg_yaml.jpg)

Create the directory ${HOME}/.hive-sre/aux_libs and copy the jdbc driver to this directory

```
mkdir -p ${HOME}/.hive-sre/aux_libs
cp /usr/share/java/mysql-connector-java.jar ${HOME}/.hive-sre/aux_libs/
ls ${HOME}/.hive-sre/aux_libs
```
![width=800](/create_and_copy_driver.jpg)


Then run hive table check command:

```
hive-sre sre  -cfg db_cfg.yaml -o ./sre-out
```
![width=800](/images/run_hive_sre_command1.jpg)
![width=800](/images/run_hive_sre_command2.jpg)

Then check every out file and Fix the problem.

![width=800](/images/run_hive_sre_command_result.jpg)

#### Reference:https://docs.cloudera.com/cdp-private-cloud-upgrade/latest/upgrade-hdp/topics/hive-expedited-migration-tasks.html


### Step 3: Creating a list of tables to migrate
Build the migrate table list in Yaml, named as test.yaml

```
---
databaseIncludeLists:
testdata:
- "test"
- "test_orc"
- "test_parquet
```

upload test.yaml to hdfs /tmp/yaml

![width=800](/images/upload_yaml_to_hdfs.jpg)

### Step 4: Migrating tables to CDP
In CDP Cloudera Manager, go to Clusters > Hive-on-Tez. Then Stop the Hive-on-Tez service.

![width=800](/images/stop_hive_on_tez_services.jpg)

In Configuration, search for table migration control file URL.

Set the value of the Table migration control file URL property to the absolute path and file name of your YAML include list.Save configuration changes.

![width=800](/images/set_migration_file.jpg)

Click Clusters > Hive-on-Tez, and in Actions, click Migrate Hive tables for CDP upgrade.

![width=800](/images/start_migration_hive_table.jpg)
![width=800](/images/confirm_migrate_hive_table.jpg)

HSMM migrates the Hive tables listed in the YAML.

![width=800](/images/start_migration_hive_table.jpg)


To prevent problems with any subsequent HSMM run, remove the value you set for Table migration control file URL, leaving the value blank.

![width=800](/images/start_migration_hive_table.jpg)

Save configuration changes.
Start the Hive-on-Tez service.
The YAML-specified tables and databases are migrated.

