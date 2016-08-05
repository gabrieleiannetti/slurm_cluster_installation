
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

Common Tasks
------------

### Munge Authentification Service<a name="munge"></a>




Installation and Configuration
------------------------------

We will start to setup the MySQL database first and are continuing with the Slurm database daemon, controller and finally the worker nodes.

### MySQL Database

##### Installation and Configuration

Install the MySQL server package:
```
apt-get install mysql-server
```

Edit the MySQL server configuration file under: **/etc/mysql/my.cnf**  
Uncomment the following line to allow remote access from a different host rather than localhost:
```
bind-address           = 127.0.0.1
```

Reboot the MySQL database:
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

##### Installing the Munge Authentification Service
[This is the link text](#munge)
Install the Munge package:
```
apt-get install munge
```

Provide a shared key for authentification by Munge under: **/etc/munge/munge.key**  
After setting the shared key do a reboot of the Munge service to active the key:
```
systemctl restart munge
```

##### Installing the Slurm Database Daemon

Install the Slurm database daemon package:
```
apt-get install slurmdbd
```

Create the Slurm database daemon configuration file under: **/etc/slurm-llnl/slurmdbd.conf**
```
AuthType=auth/munge
AuthInfo=/var/run/munge/munge.socket.2
DbdHost=lxcc01
StorageHost=lxdb01
StorageLoc=slurm_acct_db
StoragePass=12345678
StorageType=accounting_storage/mysql
StorageUser=slurm
LogFile=/var/log/slurm-llnl/slurmdbd.log
PidFile=/var/run/slurm-llnl/slurmdbd.pid
SlurmUser=slurm
```

Set configuration file owner- and group-ship to slurm:
```
chown slurm:slurm /etc/slurm-llnl/slurmdbd.conf
```

Start the Slurm database daemon:
```
systemctl start slurmdbd
```

Check the log file if the Slurm database daemon started successfully and everything rolled up:
```
view /var/log/slurm-llnl/slurmdbd.log
```

### Slurm Controller

##### Installing the Munge Authentification Service

Install the Munge package:
```
apt-get install munge
```

Provide a shared key for authentification by Munge under: **/etc/munge/munge.key**  
After setting the shared key do a reboot of the Munge service to active the key:
```
systemctl restart munge
```

##### Installing the Slurm Controller

Install the Slurm controller package:
```
apt-get install slurmctld
```

Create the Slurm configuration file under: **/etc/slurm-llnl/slurm.conf**
```

```

Set configuration file owner- and group-ship to slurm:
```
chown slurm:slurm /etc/slurm-llnl/slurmdbd.conf
```

Start the Slurm database daemon:
```
systemctl start slurmdbd
```

Check the log file if the Slurm database daemon started successfully and everything rolled up:
```
view /var/log/slurm-llnl/slurmdbd.log
```


### Slurm Worker
