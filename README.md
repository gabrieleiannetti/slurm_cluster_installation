
Installation of a Slurm Cluster
===============================

A short installation guide for a Slurm cluster running each component on a different node.


Host Setup
----------

Listing of participating hosts and its function within the cluster.

Function                     |Host               
-----------------------------|-----------------------------
Slurm Controller             |lxcm01
Slurm Database Daemon        |lxcc01
MySQL Database               |lxdb01
Slurm Worker                 |lxdev01
Slurm Worker                 |lxdev02


Installation and Configuration
------------------------------

We will start to setup the MySQL database first and are continuing with the Slurm database daemon, controller and finally the worker nodes.

### MySQL Database

Install the MySQL server package
```
apt-get install mysql-server
```

Edit the MySQL server configuration file under: **/etc/mysql/my.cnf**  
Uncomment the following line to allow remote access from a different host rather then localhost:
```
bind-address           = 127.0.0.1
```


### Slurm Database Daemon




### Slurm Controller




### Slurm Worker
