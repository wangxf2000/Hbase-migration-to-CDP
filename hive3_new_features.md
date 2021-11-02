#### we can use hue/zeppelin/beeline to check the hive3 new features. make sure you are using hive engine for testing hive3 new features.
#### I show these SQL in hue.
access Hue from Cloudera Manager->Hue->Web UI->Hue, if there is add account in Hue UI,then you can input the username/password for creating the admin user, or you can get the username/password from the admins.

Choose the Hive engine for check hive3 new features.

![width=800](/images/hue_select_hive_engine.jpg)

we can edit the sql in SQL editor,then choose what you want to run and click the run button to run it.

![width=800](/images/hue_Sql_editor.jpg)

we use the default database for this test.

#### Feature 1: Hive3 ACID
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


