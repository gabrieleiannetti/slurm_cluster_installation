
Installation of a Slurm Cluster
===============================

A short installation guide for a Slurm cluster running each component on a different node.


Host Setup
----------

Listing of participating hosts and its function within the cluster.

Function                     |Host                         |FQDN
-----------------------------|-----------------------------|-----------------------------
Slurm Controller             |lxcm01                       |lxcm01.devops.test
Slurm Database Daemon        |lxcc01                       |lxcc01.devops.test
MySQL Database               |lxdb01                       |lxdb01.devops.test
Slurm Worker                 |lxdev01                      |lxdev01.devops.test
Slurm Worker                 |lxdev02                      |lxdev02.devops.test


Installation and Configuration
------------------------------

We will start to setup the MySQL database first and are continuing with the Slurm database daemon, controller and finally the worker nodes.

### MySQL Database

##### Installation and Configuration

Install the MySQL server package
```
apt-get install mysql-server
```

Edit the MySQL server configuration file under: **/etc/mysql/my.cnf**  
Uncomment the following line to allow remote access from a different host rather than localhost:
```
bind-address           = 127.0.0.1
```

Reboot the MySQL database
```
systemctl restart mysql
```

##### Creating a Slurm database user

Create the Slurm database user:
```
CREATE USER 'slurm'@'lxcc01.devops.test' IDENTIFIED BY '12345678'; 
```

Grant access to the Slurm database user on the slurm accounting database:
```
GRANT ALL ON slurm_acct_db.* TO 'slurm'@'lxcc01.devops.test';
```

Finally flush the privileges:
```
FLUSH PRIVILEGES;
```

### Slurm Database Daemon




### Slurm Controller




### Slurm Worker
