Note : Taken help from LLMs(GEMINI) to create the setup.


1. up the docker-compose file to start the services

`docker-compose up -d`

2. go inside the master mysql container and run the following command

`
CREATE USER 'replica_user'@'%' IDENTIFIED WITH mysql_native_password BY 'replication_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
`

Note down the follwoing details of the master

`SHOW MASTER STATUS;`

eg :

mysql> SHOW MASTER STATUS;     

`
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      843 | my_database  |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
`

3. go inside the slave mysql replica and run the following command

`
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='mysql_master',
  SOURCE_USER='replica_user',
  SOURCE_PASSWORD='replication_password',
  SOURCE_LOG_FILE='<YOUR_MASTER_FILE_FROM_ABOVE>',  -- e.g., 'mysql-bin.000003'
  SOURCE_LOG_POS=<YOUR_MASTER_POS_FROM_ABOVE>;      -- e.g., 843
`

`START REPLICA;`

see the replica status : 

`SHOW REPLICA STATUS\G`
