---   
layout: post   
title: "安裝 Percona XtraDB Cluster"   
date: 2013-03-04 17:15   
comments: true   
categories: ["Database"]   
---   
   
最新的 XtraDB 安裝檔：   
<http://www.percona.com/downloads/Percona-XtraDB-Cluster/LATEST/>   
<http://www.percona.com/downloads/XtraBackup/LATEST/>   
   
官方手冊：   
<http://www.percona.com/doc/percona-xtradb-cluster/installation.html>   
<http://www.percona.com/doc/percona-xtradb-cluster/manual/bootstrap.html>   
<http://www.percona.com/files/presentations/WEBINAR-percona-xtradb-cluster-installation-and-setup.pdf>   
   
專有名詞：   
<http://www.percona.com/doc/percona-xtradb-cluster/glossary.html>   
   
Reference：   
<http://www.mysqlperformanceblog.com/2013/01/29/how-to-start-a-percona-xtradb-cluster/>   
   
1. yum install Percona-XtraDB-Cluster-server Percona-XtraDB-Cluster-client percona-xtrabackup   
1. vim /etc/my.cnf   

        [mysqld]   
        wsrep_provider=/usr/lib64/libgalera_smm.so   
        wsrep_cluster_name=叢集的名稱   
        wsrep_cluster_address=gcomm://172.16.6.221,172.16.6.222,172.16.6.223   
        wsrep_slave_threads=4   
        wsrep_sst_method=rsync   
        binlog_format=ROW   
        default_storage_engine=InnoDB   
        innodb_autoinc_lock_mode=2   
        innodb_locks_unsafe_for_binlog=1   

1. 在第一個節點執行：service mysql start --wsrep-cluster-address="gcomm://"   
1. mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"   
1. mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"   
1. mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"   
1. mysqladmin -u root password '12345678'   
1. service mysql stop   
1. service mysql start   
1. mysql -u root -p   
1. mysql> show status like 'wsrep_%';   
   
* 其他節點啓動 mysqld 請執行：service mysql start   
* service mysql start --wsrep-cluster-address="gcomm://" 代表啓動一個全新的叢集！   
   
### WSREP   
   
MySQL Write Set Replication (MySQL-wsrep) 是一個 open source project，   
目標在制定了一個 API，將資料庫的同步寫入抽象出來，原文如下：   
   
> wsrep API defines a set of application callbacks and replication library calls necessary to implement synchronous writeset replication of transactional databases and similar applications. It aims to abstract and isolate replication implementation from application details.   
   
http://www.codership.com/products/mysql-write-set-replication-project   
https://launchpad.net/wsrep   
   
### Galera 函式庫   
   
Galera 是一套根據 WSREP 標準實做出來的函式庫：   
   
> MySQL/Galera cluster uses Galera library for the replication implementation.   
> To interface with Galera replication, we have enhanced MySQL server to support replication API definition in the wsrep API project.   
   
經過特殊強化後的 MySQL（MariaDB，Percona Server，新版的 MySQL）都支援 WSREP API   
   
### XtraDB   
   
XtraDB 是一套資料庫引擎，其實就是強化後的 InnoDB，它是 Percona 公司主導開發的。   
   
> Percona XtraDB is an enhanced version of the InnoDB storage engine for MySQL and MariaDB.   
> It has much faster performance than InnoDB and better scalability on modern hardware.   
   
> The Percona XtraDB engine does not have separate binary releases.   
> It is available as part of Percona Server and MariaDB .   
   
XtraDB 引擎沒有獨立（額外）發佈執行檔，它直接包在 Percona Server 或 MariaDB 中。   
   
### XtraDB Cluster = XtraDB + Galera   
   
> XtraDB Cluster integrates Percona Server with the Galera library of high availability solutions in a product package.   
   
XtraDB Cluster 是把 Percona Server 與 Galera 整合在一起，包成一個產品。   
Percona Server 其實就是 Open Source 的 MySQL Server   
   
XtraDB Cluster 的資料庫同步機制就是靠 Galera 的函式庫來完成：   
http://www.codership.com/products/galera_replication   
   
### SST   
   
State Snapshot Transfer is the full copy of data from one node to another.   
   
If you use mysqldump SST it should be the same as this mysql client connection address plus you need to set wsrep_sst_auth variable to hold user:password pair.    
The user should be privileged enough to read system tables from donor and create system tables on this node.   
For simplicity that could be just the root user.    
Note that it also means that you need to properly set up the privileges on the new node before attempting to join the cluster.    
   
If you use xtrabackup as SST method, it will use /usr/bin/wsrep_sst_xtrabackup provided in Percona-XtraDB-Cluster-server package.   
And this script also needs user password if you have a password for root@localhost.   
   
If you use rsync SST, wsrep_sst_auth is not necessary unless your SST script makes use of it.    
   
### wsrep_sst_receive_address   
   
This is the address at which the node will be listening for and receiving the state.    
Note that in galera cluster *joining nodes* are waiting for connections from *donors*.    
It goes contrary to tradition and seems to confuse people time and again,    
but there are good reasons it was made like that.   
   
要使用 xtrabackup 當作 SST method 時，需要設定 database 的 root password 到 /etc/my.cnf   
   
    wsrep_sst_auth=root:12345678   
    wsrep_sst_receive_address=172.16.6.221   
   
http://serverfault.com/questions/389190/xtrabackup-for-sst-with-xtradb-cluster   
   
donor node：The node elected to provide a state transfer (SST or IST).   
joiner node：The node joining the cluster, usually a state transfer target.   
   
### wsrep_start_position   
   
> This variable contains the UUID:seqno value.   
> By setting all the nodes to have the same value for this option cluster can be set up without the state transfer.   
   
傳說中，只要所有的 node 設定這個 UUID 相同，就不用進行 state transfer 了？！   
   
https://www.percona.com/doc/percona-xtradb-cluster/wsrep-system-index.html#wsrep_start_position   
