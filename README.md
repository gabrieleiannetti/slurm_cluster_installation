
Installation of a Slurm Cluster
===============================

A short installation guide for a Slurm cluster running each component on a different node.


Host Setup
----------

Listing of participating hosts and its function within the cluster.

Function                     |Host                         |FQDN
-----------------------------|-----------------------------|-----------------------------
Slurm Primary Controller     |lxcm01                       |lxcm01.devops.test
Slurm Backup Controller      |lxcm02                       |lxcm02.devops.test
Slurm Database Daemon        |lxcc01                       |lxcc01.devops.test
MySQL Database               |lxdb01                       |lxdb01.devops.test
Slurm Worker                 |lxdev01                      |lxdev01.devops.test
Slurm Worker                 |lxdev02                      |lxdev02.devops.test


Common Installation and Configuration Tasks
-------------------------------------------
Describes common installation and configuration tasks that have to be done multiple times.

### Munge Authentification Service<a name="munge"></a>

Install the Munge package:
```
apt-get install munge
```

Provide a shared key under: **/etc/munge/munge.key**  
After setting the shared key do a reboot of the Munge service to active the key:
```
systemctl restart munge
```

### Slurm Controller and Worker Configuration File<a name="slurm_conf"></a>

Create the Slurm Controller/Worker configuration file under: **/etc/slurm-llnl/slurm.conf**
```
# MANAGEMENT POLICIES                                                              
ControlMachine=lxcm01                                                              
BackupController=lxcm02
AuthType=auth/munge                                                                
CryptoType=crypto/munge                                                            
SlurmUser=slurm                                                                    

# NODE CONFIGURATIONS                                                              
NodeName=lxdev0[1-4]                                                               

# PARTITION CONFIGURATIONS                                                         
PartitionName=debug Nodes=lxdev0[1-4] Default=YES                                  

# ACCOUNTING                                                                       
AccountingStorageType=accounting_storage/slurmdbd                                  
AccountingStorageHost=lxcc01                                                       
JobAcctGatherType=jobacct_gather/linux                                             
ClusterName=snowflake                                                              

# CONNECTION                                                                       
SlurmctldPort=6817                                                                 
SlurmdPort=6818                                                                    

# DIRECTORIES                                                                      
JobCheckpointDir=/var/lib/slurm-llnl/job_checkpoint                                    
SlurmdSpoolDir=/var/lib/slurm-llnl/slurmd                                          
StateSaveLocation=/var/lib/slurm-llnl/state_checkpoint                                    

# LOGGING                                                                          
SlurmctldDebug=debug                                                               
SlurmctldLogFile=/var/log/slurm-llnl/slurmctld.log                                 
SlurmdDebug=debug                                                                  
SlurmdLogFile=/var/log/slurm-llnl/slurmd.log                                       

# STATE INFO                                                                       
SlurmctldPidFile=/var/run/slurm-llnl/slurmctld.pid                                 
SlurmdPidFile=/var/run/slurm-llnl/slurmd.pid                                       

# SCHEDULING                                                                       
FastSchedule=2                                                                     

# ERROR RECOVERY                                                                   
ReturnToService=1                                    
```

Set configuration file owner- and group-ship for Slurm:
```
chown slurm:slurm /etc/slurm-llnl/slurm.conf
```

### Setting up the checkpoint directories between the Slurm Controller<a name="setup_checkdirs"></a>

Both directory paths which are specified by the parameter **JobCheckpointDir** and **StateSaveLocation** in the configuration file **/etc/slurm-llnl/slurm.conf** are set up here via a NFS mount, so both the primary and backup controller can share its directory contents.

##### Creating the checkpoint directories

Create the checkpoint directories on _both_ controller:
```
cd /var/lib/slurm-llnl/

mkdir job_checkpoint
mkdir state_checkpoint

chown slurm:slurm job_checkpoint
chown slurm:slurm state_checkpoint
```

##### Setting up the primary controller as NFS server
The primary controller is used as NFS server to share the checkpoint directories with the backup controller.

Install the required package:
```
apt-get install nfs-kernel-server
```

Edit the proper config file:
```
vi /etc/exports
/var/lib/slurm-llnl/job_checkpoint      lxcm02.devops.test(rw,async)
/var/lib/slurm-llnl/state_checkpoint    lxcm02.devops.test(rw,async)
```

Restart the NFS server:
```
systemctl restart nfs-kernel-server
```

##### Setting up the backup controller

Install the required package:
```
apt-get install nfs-common
```

Mount the checkpoint directories:
```
sudo mount -t nfs -o rw,nosuid lxcm01.devops.test:/var/lib/slurm-llnl/job_checkpoint /var/lib/slurm-llnl/job_checkpoint
sudo mount -t nfs -o rw,nosuid lxcm01.devops.test:/var/lib/slurm-llnl/state_checkpoint /var/lib/slurm-llnl/state_checkpoint
```


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

Check the log file if the MySQL database started successfully:
```
less /var/log/mysql/error.log
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
[Setup the Munge authentification service](#munge)

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
less /var/log/slurm-llnl/slurmdbd.log
```

### Slurm Primary Controller

##### Installing the Munge Authentification Service
[Setup the Munge authentification service](#munge)

##### Installing the Slurm Primary Controller

Install the Slurm controller package:
```
apt-get install slurmctld
```

##### Setup the Slurm Controller/Worker configuration file
[Setup the Slurm configuration file](#slurm_conf)

##### Setup the checkpoint directories for the primary controller
[Setup the checkpoint directories](#setup_checkdirs)

##### Starting the Slurm Primary Controller
Start the Slurm controller daemon:
```
systemctl start slurmctld
```

Check the log file if the Slurm controller started successfully:
```
less /var/log/slurm-llnl/slurmctld.log
```

##### Setting up the logical cluster
```
sacctmgr add cluster snowflake -i
sacctmgr -i add account slurm Cluster=snowflake Description='none' Organization='none'
```

### Slurm Backup Controller

##### Installing the Munge Authentification Service
[Setup the Munge authentification service](#munge)

##### Installing the Slurm Backup Controller

Install the Slurm controller package:
```
apt-get install slurmctld
```

##### Setup the Slurm Controller/Worker configuration file
[Setup the Slurm configuration file](#slurm_conf)

##### Setup the checkpoint directories for the backup controller
[Setup the checkpoint directories](#setup_checkdirs)

##### Starting the Slurm Backup Controller
Start the Slurm controller daemon:
```
systemctl start slurmctld
```

Check the log file if the Slurm controller started successfully:
```
less /var/log/slurm-llnl/slurmctld.log
```

### Slurm Worker

##### Installing the Munge Authentification Service
[Setup the Munge authentification service](#munge)

##### Installing the Slurm Worker

Install the Slurm worker package:
```
apt-get install slurmd
```

##### Setup the Slurm Controller/Worker configuration file
[Setup the Slurm configuration file](#slurm_conf)

Start the Slurm controller daemon:
```
systemctl start slurmd
```

Check the log file if the Slurm controller started successfully:
```
less /var/log/slurm-llnl/slurmd.log
```
