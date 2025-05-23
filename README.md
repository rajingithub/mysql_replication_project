**Setup to Create Master and Read replicas for MySQL locally as docker containers.**

Note : Taken help from LLMs(GEMINI) to create the setup.


1. up the docker-compose file to start the services

`docker-compose up -d`

2. go inside the master mysql container and run the following command

`
CREATE USER 'replica_user'@'%' IDENTIFIED WITH mysql_native_password BY 'replication_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'%';
FLUSH PRIVILEGES;
`

Note down the following details of the master

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


whatever write operations we do on master will be written on the log file `mysql-bin.000003` in the location `/var/lib/mysql` in the master container.
read replicas will connect to the master database using the above created replica_user and read the write operations written to the logfiles and replicas 
will syncsame data their database.    

ran follwoing commands on master and the folllowing binlog file content.     

`CREATE TABLE authors (id INT, name VARCHAR(20), email VARCHAR(20));`     
`INSERT INTO authors (id,name,email) VALUES(1,"Vivek","xuz@abc.com");`       
`INSERT INTO authors (id,name,email) VALUES(1,"bmr","bmr@abc.com");`    


<img width="1512" alt="Screenshot 2025-05-23 at 8 33 01 PM" src="https://github.com/user-attachments/assets/9b7e9e2a-4b96-4098-853b-c44a8d61a29f" />


