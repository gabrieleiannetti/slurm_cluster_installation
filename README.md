
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

We are starting to setup the MySQL database and are continuing with the database daemon, cluster controller and finally the worker nodes.


Some basic Git commands are:
```
git status
git add
git commit
```