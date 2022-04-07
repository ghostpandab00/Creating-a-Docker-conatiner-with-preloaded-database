## Creating-a-Docker-conatiner-with-preloaded-database

<p align="center">
  <img 
    width="700"
    height="300"
    src= "https://user-images.githubusercontent.com/65948438/162279419-6edb3fff-a332-4a04-b89e-09acdd429d10.jpg"
  >
</p>     

----
## Scenario
Consider the following scenario: we have a database dump that needs to be restored while the container is being built. After the container has been constructed, we may use 'source' to restore a database. But what if it's needed in the middle of the game?

## Prerequisites
 - Knowledge of Docker
 - Docker installed

## Solution
The usage of mounting makes it simple to restore a database dump. Here we have a dump of the database named "wpdb.sql". It's a dump of one wordpress database.

```sh
[ec2-user@ip-172-31-37-71 newsite]$ pwd
/home/ec2-user/newsite
[ec2-user@ip-172-31-37-71 newsite]$ du -sch wpdb.sql 
1.4M    wpdb.sql
1.4M    total
```

We have to mount the dump file's path to /docker-entrypoint-initdb.d/, So, what exactly is "docker-entrypoint-initdb.d?"

> Initialization scripts If you would like to do additional initialization in an image derived from this one, add one or more *.sql, *.sql.gz, or *.sh scripts under /docker-entrypoint-initdb.d (creating the directory if necessary). After the entrypoint calls initdb to create the default postgres user and database, it will run any *.sql files, run any executable *.sh scripts, and source any non-executable *.sh scripts found in that directory to do further initialization before starting the service.

Also follow thoguh the [offical mysql image doc from docker hub](https://hub.docker.com/_/mysql) and [volumes doc on docker docs](https://docs.docker.com/storage/volumes/) to get a better understanding. 

Now that we got an idea about "docker-entrypoint-initdb.d," let's create a docker container.

```sh
[ec2-user@ip-172-31-37-71 newsite]$ docker container run --name database -p 3306:3306 -d -v /home/ec2-user/newsite/wpdb.sql:/docker-entrypoint-initdb.d/wpdb.sql -e MYSQL_ROOT_PASSWORD="<mysql root password>"  mysql:5.7
```

We can see from the above that we've mounted the .sql file in /docker-entrypoint-initdb.d/, and Docker will use this file when building the container.

```sh
[ec2-user@ip-172-31-37-71 newsite]$ docker container ls
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
56d9859af1ac   mysql:5.7   "docker-entrypoint.s…"   9 seconds ago   Up 8 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp   database
```

Now that we have a working container, we can use the mysql console to see if the database has been restored. We must first use exec to enter the docker container in interactive mode.

```sh
[ec2-user@ip-172-31-37-71 newsite]$ docker container exec -it database bash
root@56d9859af1ac:/# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.37 MySQL Community Server (GPL)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wp                 |
+--------------------+
5 rows in set (0.00 sec)

mysql> use wp;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------------+
| Tables_in_wp          |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_term_relationships |
| wp_term_taxonomy      |
| wp_termmeta           |
| wp_terms              |
| wp_usermeta           |
| wp_users              |
+-----------------------+
12 rows in set (0.00 sec)

mysql> 
```

Take a look at what we've got! Hooray!! The wpdb.sql database dump was restored to the 'wp' database. If you learned something from my blog, please remember to share it  🌟 and if you REALLY enjoyed it, please follow me!
