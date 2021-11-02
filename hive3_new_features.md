#### we can use hue/zeppelin/beeline to check the hive3 new features.
##### I show these SQL in hue.
1.Create Hive ACID table and test
```
SHOW DATABASES;
CREATE TABLE acidtbl (a INT, b STRING);
SHOW CREATE TABLE acidtbl;
--insert
INSERT INTO acidtbl (a,b) VALUES (100, "oranges"), (200, "apples"), (300, "bananas");
--delete
DELETE FROM acidTbl where a = 200;
--update
UPDATE acidTbl SET b = "pears" where a = 300;

```

create students.csv file with the following info,then upload to hdfs /tmp/student/student.csv
1,jane,doe,senior,mathematics
2,john,smith,junior,engineering 

--create external table names_text and specify the data file
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
  
  --create managed table Names
  CREATE TABLE IF NOT EXISTS Names(
  student_ID INT, FirstName STRING, LastName STRING,    
  year STRING, Major STRING)
  COMMENT 'Student Names';

  --insert external table into managed table
  INSERT OVERWRITE TABLE Names SELECT * FROM names_text;
  
  --validate data
  SELECT * from Names; 
DROP TABLE names_text;
SELECT * from Names; 

--check external table data
SELECT * from names_text;

--check hdfs source file is droped
hdfs dfs -ls /tmp/student/student.csv
hdfs dfs -cat /tmp/student/student.csv

--Create an external table that can delete data
CREATE EXTERNAL TABLE IF NOT EXISTS names_text(
  a INT, b STRING)
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/tmp/student'
  TBLPROPERTIES ('external.table.purge'='true');

--drop external table 
DROP TABLE names_text;

--check hdfs source file is droped
hdfs dfs -ls /tmp/student/student.csv
hdfs dfs -cat /tmp/student/student.csv

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


