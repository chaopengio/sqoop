### New Feature
Fix use mysql as metastore transaction bug.

### How to use
##### 1. Add following configuration in sqoop-site.xml

```xml
<property>
    <name>sqoop.metastore.client.enable.autoconnect</name>
    <value>true</value>
    <description>If true, Sqoop will connect to a local metastore
      for job management when no other metastore arguments are
      provided.
    </description>
  </property>
  <property>
    <name>sqoop.metastore.client.autoconnect.url</name>
    <value>jdbc:mysql://127.0.0.1:3306/sqoop?createDatabaseIfNotExist=true</value>
  </property>
  <property>
    <name>sqoop.metastore.client.autoconnect.username</name>
    <value>sqoop</value>
  </property>
  <property>
    <name>sqoop.metastore.client.autoconnect.password</name>
    <value>test</value>
  </property>
</configuration>
```

##### 2. Create the following object in the database.
```sql
CREATE TABLE SQOOP_ROOT (
    version INT,
    propname VARCHAR(128) NOT NULL,
    propval VARCHAR(256),
    CONSTRAINT SQOOP_ROOT_unq UNIQUE (version, propname)
);
```

##### 3. Insert the following row
```sql
INSERT INTO
    SQOOP_ROOT
VALUES(
    NULL,
    'sqoop.mysqldb.job.storage.version',
    '0'
);
```

##### 4. Test
Create test job

```shell
sqoop job -D sqoop.hbase.add.row.key=true  --meta-connect "jdbc:mysql://127.0.0.1:3306/sqoop?user=sqoop&password=test" \
     --create sqoop_incr_test -- import --verbose -m 2 \
     --connect "jdbc:mysql://127.0.0.1:3306/testtable" --username testuser --password "pswd" \
     --table "sqooptest" \
     --hbase-table sqooptest --column-family data --hbase-row-key "id" --columns "id, name, update_time" \
     --split-by id --incremental append --check-column id --hbase-create-table
```

Run test job

```shell
sqoop job -D sqoop.hbase.add.row.key=true \
     --meta-connect "jdbc:mysql://127.0.0.1:3306/sqoop?user=sqoop&password=test" --exec sqoop_incr_test
```

### Note
Sqoop metastore password should not contain special chars, such as "/". Those chars will failed when connect to mysql.

### Reference
The following patch fix the transaction bug. However, the autoconnect is not working. My friend [Song Hou](http://housong.github.io/) and I fixed the autoconnect problem.
- https://issues.apache.org/jira/secure/attachment/12732641/sqoop-patch.txt
- http://stackoverflow.com/questions/24078668/how-to-change-sqoop-metastore
