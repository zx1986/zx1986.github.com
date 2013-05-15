---
layout: post
title: "安裝 Percona XtraDB Cluster"
date: 2013-03-04 17:15
comments: true
categories:
- Database
---

## 背景知識
 
 
#### Codership

[Codership][1] 是一家成立於 2007 年的公司，公司的 Founder 都是 Database 的專家。
Codership 致力於研究及實做高擴展性且快速的資料庫同步（Replication）機制，
並帶頭制定了一套叫 WSREP 的 API 標準，自己也根據這套 API 實做了 Galera 同步器（Replicator）。
 
#### WSREP（Write Set REPlication）

[WSREP][2] 是一個爲 DBMS 類型應用設計的通用的、插件式（pluggable）的 API 標準，
它爲 DBMS 類型的應用（即常見的 Database 軟體）建立了一個 Replication 介面，
這個介面（Interface）介於 Database 軟體與 Replication Servcie Provider（即 Replicator）之間。

[WSREP Group][3] 則是討論與建立這個標準的開放性羣組。

> wsrep API defines a set of application callbacks and replication library calls necessary to implement synchronous writeset replication of transactional databases and similar applications. It aims to abstract and isolate replication implementation from application details.

#### Galera Replicator

Galera 是一套根據 WSREP 標準實做出來的 Replication 函式庫。
Galera 的運作架構可以參考[它們的說明][4]。

其原則就是：
當對 Cluster 中其中一個 Node 做寫入（Write）時，
Galera 會自動將寫入動作 Replication 到 Cluster 其他的 Node 上。

> Galera implements wsrep pluggable interface, and can provide several replication modes and topologies, including the ultimate Synchronous Multi-Master replication.

https://launchpad.net/galera

#### MySQL Galera Cluster

傳統的 MySQL Server 只要打上 WSREP 的 Patch，支援了 WSREP，
再搭配服用 Galera 函式庫，調整好設定檔，就可以多臺組成一個 Cluster。

> MySQL/Galera cluster uses Galera library for the replication implementation. To interface with Galera replication, we have enhanced MySQL server to support replication API definition in the wsrep API project.

http://www.codership.com/products/mysql_galera

#### MariaDB Galera Cluster

相較於 MySQL 要打 Patch，MariaDB 直接推出包好的 MariaDB Galera Cluster：
https://downloads.mariadb.org/mariadb-galera/

MariaDB 還針對不同的 Linux 發佈版提供了套件庫：
https://downloads.mariadb.org/mariadb/repositories/

MariaDB Galera Cluster 是 Percona XtraDB Cluster 之外的另一個選擇。

#### Percona XtraDB Cluster（PXC）

Percona 是一家專業的 MySQL 顧問與技術公司。
Percona 有一個知名的 MySQL Blog：<i class="icon-external-link icon-sx"> </i><http://www.mysqlperformanceblog.com/> 
Percona 開發了一堆的資料庫工具與軟體：<i class="icon-external-link icon-sx"> </i><http://www.percona.com/software> 

XtraDB：Percona 基於 InnoDB 開發的一個資料庫引擎。
Percona Server：基於 XtraDB 引擎且改良過的 MySQL Server。
Percona XtraDB Cluster：Percona Server + Galera Library

Percona XtraDB Cluster 是把 Percona Server 與 Galera 整合在一起，包成一個產品。
Percona XtraDB Cluster 的資料庫同步機制是靠 Galera 來完成的（即 Write Replication）。 


## 安裝 Percona XtraDB Cluster
 
最新的 XtraDB 安裝檔：
<i class="icon-external-link icon-sx"> </i><http://www.percona.com/downloads/Percona-XtraDB-Cluster/LATEST/>
<i class="icon-external-link icon-sx"> </i><http://www.percona.com/downloads/XtraBackup/LATEST/>

以 Red Hat 環境（RHEL，Cent OS）爲例。

rpm -Uhv http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm
yum install Percona-XtraDB-Cluster-server Percona-XtraDB-Cluster-client percona-xtrabackup
vim /etc/my.cnf

[mysqld]
wsrep_provider=/usr/lib64/libgalera_smm.so
wsrep_cluster_name=叢集的名稱
wsrep_cluster_address=gcomm://節點一的位址,節點二的位址,節點三的位址
wsrep_slave_threads=4
wsrep_sst_method=rsync
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1

service mysql start --wsrep-cluster-address="gcomm://"
mysql -e "CREATE FUNCTION fnv1a_64 RETURNS INTEGER SONAME 'libfnv1a_udf.so'"
mysql -e "CREATE FUNCTION fnv_64 RETURNS INTEGER SONAME 'libfnv_udf.so'"
mysql -e "CREATE FUNCTION murmur_hash RETURNS INTEGER SONAME 'libmurmur_udf.so'"
mysqladmin -u root password '12345678'
service mysql stop
service mysql start
mysql -u root -p
mysql> show status like 'wsrep_%';
 
其他節點啓動 mysqld 請執行：service mysql start
service mysql start --wsrep-cluster-address="gcomm://" 代表啓動一個全新的叢集！

#### SST

> State Snapshot Transfer is the full copy of data from one node to another.

SST 是 State Snapshot Transfer 的縮寫，指的是 PXC 間 Node 資料同步的方式。
在 /etc/my.cnf 中可以透過 wsrep_sst_method 參數來設定。

在 PXC 中有三種方式，分別是：

* wsrep_sst_method=mysqldump

> If you use mysqldump SST it should be the same as this mysql client connection address plus you need to set wsrep_sst_auth variable to hold user:password pair. The user should be privileged enough to read system tables from donor and create system tables on this node. For simplicity that could be just the root user. Note that it also means that you need to properly set up the privileges on the new node before attempting to join the cluster.

* wsrep_sst_method=rsync

> If you use rsync SST, wsrep_sst_auth is not necessary unless your SST script makes use of it.

* wsrep_sst_method=xtrabackup

> If you use xtrabackup as SST method, it will use /usr/bin/wsrep_sst_xtrabackup provided in Percona-XtraDB-Cluster-server package. And this script also needs user password if you have a password for root@localhost.

要使用 xtrabackup 當作 SST method 時，需要設定 Database 的 root password 到 /etc/my.cnf

 wsrep_sst_auth=root:12345678

<i class="icon-external-link icon-sx"> </i><http://serverfault.com/questions/389190/xtrabackup-for-sst-with-xtradb-cluster>



官方手冊：
<i class="icon-external-link icon-sx"> </i><http://www.percona.com/doc/percona-xtradb-cluster/installation.html>
<i class="icon-external-link icon-sx"> </i><http://www.percona.com/doc/percona-xtradb-cluster/manual/bootstrap.html>
<i class="icon-external-link icon-sx"> </i><http://www.percona.com/files/presentations/WEBINAR-percona-xtradb-cluster-installation-and-setup.pdf>

專有名詞：
<i class="icon-external-link icon-sx"> </i><http://www.percona.com/doc/percona-xtradb-cluster/glossary.html>

Reference：
http://www.mysqlperformanceblog.com/2013/01/29/how-to-start-a-percona-xtradb-cluster/

[1]: http://www.codership.com/company/
[2]: https://launchpad.net/wsrep/
[3]: https://launchpad.net/wsrep-group/
[4]: http://www.codership.com/products/galera_replication/
