#### we can use hue/zeppelin/beeline to check the hive3 new features and CDH to CDP hive changes. make sure you are using hive engine for testing hive3 new features.
#### I show these SQL in hue.
access Hue from Cloudera Manager->Hue->Web UI->Hue, if there is add account in Hue UI,then you can input the username/password for creating the admin user, or you can get the username/password from the admins.

Choose the Hive engine for check hive3 new features.

![width=800](/images/hue_select_hive_engine.jpg)

we can edit the sql in SQL editor,then choose what you want to run and click the run button to run it.

![width=800](/images/hue_Sql_editor.jpg)

we use the default database for this test.

#### Changes 1:Hive Configuration Property Changes
There are a lot of property changes from CDH5 to CDP, so you need to check the property. all change properties can be found from https://docs.cloudera.com/cdp-private-cloud-upgrade/latest/upgrade-cdh/topics/ug_hive_configuration_changes.html

#### Changes 2: LOCATION and MANAGEDLOCATION clauses
Hive assigns a default location in the warehouse to managed tables. In CDP, Hive does not allow the LOCATION clause in queries to create a managed table. Using this clause, you can specify a location only when creating external tables. 

##### External table limitation for creating table locations
```
CREATE EXTERNAL TABLE my_external_table (a string, b string)  
ROW FORMAT SERDE 'com.mytables.MySerDe' 
WITH SERDEPROPERTIES ( "input.regex" = "*.csv")
LOCATION '/warehouse/tablespace/external/hive/marketing';
show create table my_external_table;
```

![width=800](/images/hue_Sql_editor.jpg)

##### Table MANAGEDLOCATION clause

if you set LOCATION in create managed table, you'll get an error.
```
CREATE TABLE my_managed_table (a string, b string)  
ROW FORMAT SERDE 'com.mytables.MySerDe' 
WITH SERDEPROPERTIES ( "input.regex" = "*.csv")
LOCATION '/warehouse/tablespace/external/hive/marketing';
show create table my_managed_table;
```

![width=800](/images/hue_Sql_editor.jpg)

In the MANAGEDLOCATION clause, you specify a top level directory for managed tables when creating a Hive database. Do not set LOCATION and MANAGEDLOCATION to the same HDFS path.

#### Changes 3: Handling table reference syntax
For ANSI SQL compliance, Hive 3.x rejects `db.table` in SQL queries as described by the Hive-16907 bug fix. A dot (.) is not allowed in table names. As a Data Engineer, you need to ensure that Hive tables do not contain these references before migrating the tables to CDP, that scripts are changed to comply with the SQL standard references, and that users are aware of the requirement.

```
SELECT * FROM `DEFAULT.my_external_table`;
SELECT * FROM `DEFAULT`.`my_external_table`;
```
![width=800](/images/hue_Sql_editor.jpg)

#### Changes 4: Identifying semantic changes and workarounds

##### Casting timestamps
###### Before Upgrade to CDP

Casting a numeric type value into a timestamp could be used to produce a result that reflected the time zone of the cluster. For example, 1597217764557 is 2020-08-12 00:36:04 PDT. Running the following query casts the numeric to a timestamp in PDT:

```
SELECT CAST(1597217764557 AS TIMESTAMP); 
```
The result is: `2020-08-12 00:36:04`
###### After Upgrade to CDP
Casting a numeric type value into a timestamp produces a result that reflects the UTC instead of the time zone of the cluster. Running the following query casts the numeric to a timestamp in UTC.

```
SELECT CAST(1597217764557 AS TIMESTAMP); 
```
![width=800](/images/hue_Sql_editor.jpg)

###### Action Required

Change applications. Do not cast from a numeral to obtain a local time zone. Built-in functions from_utc_timestamp and to_utc_timestamp can be used to mimic behavior before the upgrade.


##### Casting invalid dates
Casting of an invalid date differs from Hive 1 in CDH 5 to Hive 3 in CDP. Hive 3 uses a different parser formatter from the one used in Hive 1, which affects semantics. Hive 1 considers 00 invalid for date fields. Hive 3 considers 00 valid for date fields. Neither Hive 1 nor Hive 3 correctly handles invalid dates, and Hive-25056 addresses this issue.

###### Before Upgrade to CDP

Casting of invalid date (zero value in one or more of the 3 fields of date, month, year) returns a NULL value:

```
SELECT CAST ('0000-00-00' as date) , CAST ('000-00-00 00:00:00' AS TIMESTAMP) ;
```

The result is `NULL NULL`

###### After Upgrade to CDP

Casting of an invalid date returns a result.

```
SELECT CAST ('0000-00-00' as date) , CAST ('000-00-00 00:00:00' AS TIMESTAMP) ;
```

![width=800](/images/hue_Sql_editor.jpg)

###### Action Required

Do not cast invalid dates in Hive 3.


#### Changing incompatible column types
A default configuration change can cause applications that change column types to fail.

###### Before Upgrade to CDP

In HDP 2.x and CDH 5.x and CDH 6 `hive.metastore.disallow.incompatible.col.type.changes` is false by default to allow changes to incompatible column types. For example, you can change a STRING column to a column of an incompatible type, such as MAP<STRING, STRING>. No error occurs.

###### After Upgrade to CDP

In CDP, `hive.metastore.disallow.incompatible.col.type.changes` is true by default. Hive prevents changes to incompatible column types. Compatible column type changes, such as INT, STRING, BIGINT, are not blocked.

###### Action Required

Change applications to disallow incompatible column type changes to prevent possible data corruption. Check ALTER TABLE statements and change those that would fail due to incompatible column types.



#### Creating tables
To improve useability and functionality, Hive 3 significantly changed table creation.

Hive has changed table creation in the following ways:

Creates ACID-compliant table, which is the default in CDP

Supports simple writes and inserts

Writes to multiple partitions

Inserts multiple data updates in a single SELECT statement

Eliminates the need for bucketing.

If you have an ETL pipeline that creates tables in Hive, the tables will be created as ACID. Hive now tightly controls access and performs compaction periodically on the tables. The way you access managed Hive tables from Spark and other clients changes. In CDP, access to external tables requires you to set up security access permissions.

###### Before Upgrade to CDP

In CDH and HDP 2.6.5, by default CREATE TABLE created a non-ACID table.

###### After Upgrade to CDP

In CDP, by default CREATE TABLE creates a full, ACID transactional table in ORC format.

###### Action Required

Perform one or more of the following actions:

Configure legacy CREATE TABLE behavior (see the next section) to create external tables by default.

To read Hive ACID tables from Spark, you connect to Hive using the Hive Warehouse Connector (HWC) or the HWC Spark Direct Reader. To write ACID tables to Hive from Spark, you use the HWC and HWC API. Spark creates an external table with the purge property when you do not use the HWC API. For more information, see HWC Spark Direct Reader and Hive Warehouse Connector.

Set up Ranger policies and HDFS ACLs for tables. For more information, see HDFS ACLs and HDFS ACL Permissions.



#### Changes 1: Hive3 ACID
create a Hive3 acid table, then do the insert/select/update/delete, also show table information.
```
CREATE TABLE acidtbl (a INT, b STRING);
SHOW CREATE TABLE acidtbl;
--insert
INSERT INTO acidtbl (a,b) VALUES (100, "oranges"), (200, "apples"), (300, "bananas");
select * from acidtbl;
--delete
DELETE FROM acidTbl where a = 200;
--update
UPDATE acidTbl SET b = "pears" where a = 300;
select * from acidtbl;
```
![width=800](/images/hue_create_acid_table.jpg)
![width=800](/images/show_acid_table_info.jpg)
![width=800](/images/show_insert_select_acid.jpg)
![width=800](/images/show_delete_update_acid.jpg)

#### Feature 2: Managed table and Externale table
create students.csv file with the following info,then upload to hdfs /tmp/student/student.csv

```
1,jane,doe,senior,mathematics
2,john,smith,junior,engineering 
```

![width=800](/images/hdfs_put_student.jpg)

create external table names_text and specify the data file

```
CREATE EXTERNAL TABLE IF NOT EXISTS names_text(
  student_ID INT, FirstName STRING, LastName STRING,    
  year STRING, Major STRING)
  COMMENT 'Student Names'
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/tmp/student';

  DESCRIBE FORMATTED names_text;
  SELECT * FROM names_text;
 ```

 ![width=800](/images/desc_external_table.jpg)
 ![width=800](/images/select_external_table.jpg)
 
  Create managed table Names
  
  ```
  CREATE TABLE IF NOT EXISTS Names(
  student_ID INT, FirstName STRING, LastName STRING,    
  year STRING, Major STRING)
  COMMENT 'Student Names';

  DESCRIBE FORMATTED Names;
  
  --insert external table into managed table
  INSERT OVERWRITE TABLE Names SELECT * FROM names_text;
  
  --validate data
  SELECT * from Names; 
DROP TABLE names_text;
SELECT * from Names; 

--check external table data
SELECT * from names_text;
```
![width=800](/images/desc_managed_table.jpg)
![width=800](/images/insert_managed_table.jpg)
![width=800](/images/select_managed_table.jpg)
![width=800](/images/select_droped_external_table.jpg)

check hdfs source file is droped
```
hdfs dfs -ls /tmp/student/student.csv
hdfs dfs -cat /tmp/student/student.csv
```

![width=800](/images/check_drop_external_table_hdfs.jpg)

Create an external table that can delete data

```
CREATE EXTERNAL TABLE IF NOT EXISTS names_text(
  a INT, b STRING)
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/tmp/student'
  TBLPROPERTIES ('external.table.purge'='true');
show create table names_text;
--drop external table 
DROP TABLE names_text;
```
![width=800](/images/create_purge_external_table.jpg)
![width=800](/images/drop_purge_external_table.jpg)

Check hdfs source file is droped
```
hdfs dfs -ls /tmp/student/student.csv
hdfs dfs -cat /tmp/student/student.csv
```

![width=800](/images/check_drop_external_table_hdfs.jpg)

--Convert managed non-transactional tables to external

CREATE TABLE T2(a int, b int) 
  STORED AS ORC
  TBLPROPERTIES ('transactional'='true',
'transactional_properties'='insert_only');
INSERT INTO TABLE T2 VALUES ( 35, 1), ( 32, 2);

DESCRIBE FORMATTED T2;
ALTER TABLE T2 SET TBLPROPERTIES('EXTERNAL'='TRUE','external.table.purge'='true');
DESCRIBE FORMATTED T2;


--Example of constraints
CREATE TABLE t(a TINYINT, b SMALLINT NOT NULL ENABLE, c INT);
--works
INSERT INTO t values(2,45,5667);
--constraints error
INSERT INTO t values(2,NULL,5667); 

--foreign constraints
CREATE TABLE Persons (   
     ID INT NOT NULL,   
     Name STRING NOT NULL,   
     Age INT,
     Creator STRING DEFAULT CURRENT_USER(),    
     CreateDate DATE DEFAULT CURRENT_DATE(),
     PRIMARY KEY (ID) DISABLE NOVALIDATE);
     
     CREATE TABLE BusinessUnit (
     ID INT NOT NULL,    
     Head INT NOT NULL,
     Creator STRING DEFAULT CURRENT_USER(),    
     CreateDate DATE DEFAULT CURRENT_DATE(),
     PRIMARY KEY (ID) DISABLE NOVALIDATE,
     CONSTRAINT fk FOREIGN KEY (Head) REFERENCES Persons(ID) DISABLE NOVALIDATE
     );


