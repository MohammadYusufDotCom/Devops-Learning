## How to reset password of mysql

#### Step 1: step my sql server
```
systemctl stop mysql
```

#### Step 2: Start MySQL in safe mode (without password checks)
```
sudo mysqld_safe --skip-grant-tables &
```

#### Step 3: if gives error like below 
Directory '/var/run/mysqld' for UNIX socket file doesn't exist."
run below 
```
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld
sudo mysqld_safe --skip-grant-tables &
```

#### Step 4: Log in to db 
```
mysql -u root
```

#### Step 5: Add a new user using below queries
```
INSERT INTO mysql.user (Host, User, ssl_type, ssl_cipher, x509_issuer, x509_subject, max_questions, max_updates, max_connections, max_user_connections, plugin, password_expired, account_locked)
VALUES ('localhost', 'newadmin', '', '', '', '', 0,0,0,0,'mysql_native_password','N','N');
```

#### Step 6: Insert required field
```
UPDATE mysql.user
SET 
 authentication_string = PASSWORD('newadmin123'),
 Select_priv = 'Y', Insert_priv = 'Y', Update_priv = 'Y', Delete_priv = 'Y',
 Create_priv = 'Y', Drop_priv = 'Y', Reload_priv = 'Y', Shutdown_priv = 'Y',
 Process_priv = 'Y', File_priv = 'Y', Grant_priv = 'Y', References_priv = 'Y',
 Index_priv = 'Y', Alter_priv = 'Y', Show_db_priv = 'Y', Super_priv = 'Y',
 Create_tmp_table_priv = 'Y', Lock_tables_priv = 'Y', Execute_priv = 'Y',
 Repl_slave_priv = 'Y', Repl_client_priv = 'Y', Create_view_priv = 'Y', Show_view_priv = 'Y',
 Create_routine_priv = 'Y', Alter_routine_priv = 'Y', Create_user_priv = 'Y', Event_priv = 'Y',
 Trigger_priv = 'Y', Create_tablespace_priv = 'Y'
WHERE User = 'newadmin' AND Host = 'localhost';
```

#### Step 7: Exit from databse and kill the process you run in step 2 find pid and kill 
```
ps -ef |grep mysql 
```

#### Step 8: Start mysql with systemd and log in with new credential 
```
myswl -u newadmin -p 
```

#### Step 9: Take backup of data only that our applicaiton has not mysql user, sys, information_schema, performance_schema and sys
```
mysqldump -u newadmin -p --routines --databases db1 db2 > app_backup.sql
```
if want to compress 
```
mysqldump -u newadmin -p --routines --databases db1 db2 | gzip > app_backup.sql.gz
```
if taking backup of running db then use --single-transaction, this will not lock the row hence not block update delete or insert queries 
```
mysqldump -u newadmin -p --single-transaction --routines --databases db1 db2 | gzip > app_backup.sql.gz
```

#### Step 10: restore data to new db using below commands 
```
mysql -u root -p < app_backup.sql
```
Or if compressed:
```
gunzip < app_backup.sql.gz | mysql -u root -p
```


#### to find which db if you have any triggers or routines
```
SELECT TRIGGER_NAME, EVENT_MANIPULATION, EVENT_OBJECT_TABLE, ACTION_STATEMENT, TRIGGER_SCHEMA
FROM information_schema.TRIGGERS
WHERE TRIGGER_SCHEMA = 'your_database';

SELECT ROUTINE_NAME, ROUTINE_TYPE
FROM information_schema.ROUTINES
WHERE ROUTINE_SCHEMA = 'your_database';

SELECT EVENT_NAME, EVENT_DEFINITION, EVENT_SCHEDULE, EVENT_SCHEMA
FROM information_schema.EVENTS
WHERE EVENT_SCHEMA = 'your_database';
```
