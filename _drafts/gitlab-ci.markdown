---
layout: post
title: "安裝 Gitlab CI"
date: 2013-06-01T19:17:35+08:00
comments: true
external-url: 
categories: 
- Git
- Linux
- Test
---

## 關於 Continous Integration

......

在 RHEL（Red Hat/CentOS/Fedora）環境安裝 Gitlab CI；   
以下安裝過程爲 RHEL 6.3 x86_64 中的執行記錄。

## Install Required Packages

1. cd /opt
1. wget http://mirror01.idc.hinet.net/EPEL/6/i386/epel-release-6-8.noarch.rpm
1. rpm -iUvh epel-release-6-8.noarch.rpm
1. yum clean all
1. yum groupinstall "Development Tools" 
1. yum update && yum upgrade
1. yum install vim python wget curl git openssh-server
1. yum install sqlite sqlite-devel mysql mysql-libs mysql-devel
1. yum install ncurses-devel libcurl-devel libcurl curl patch sudo
1. yum install libxslt-devel libyaml-devel libxml2 libxml2-devel gdbm-devel libffi libffi-devel zlib zlib-devel openssl-devel readline readline-devel curl-devel openssl-devel pcre-devel memcached-devel valgrind-devel ImageMagick-devel ImageMagick libicu libicu-devel make bzip2 autoconf automake libtool bison redis libpq-devel libicu-devel postgresql-libs postgresql-devel
1. chkconfig redis on
1. service redis start
1. yum install postfix
1. chkconfig postfix on
1. service postfix start

## Setup Database (MySQL)

1. su - root
1. yum install mysql mysql-devel
1. chkconfig mysqld on
1. service mysqld start
1. mysql -u root -p
1. mysql> CREATE DATABASE IF NOT EXISTS \`gitlab_ci\` DEFAULT CHARACTER SET \`utf8\` COLLATE \`utf8_unicode_ci\`;
1. mysql> CREATE USER 'gitlab_ci'@'localhost' IDENTIFIED BY '12345678';
1. mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON \`gitlab_ci\`.* TO 'gitlab_ci'@'localhost';

* 預設資料庫帳號爲 gitlab_ci
* 預設資料庫密碼爲 12345678

## Setup Account (gitlab_ci)

1. useradd gitlab_ci
1. passwd gitlab_ci
1. chmod 755 /home/gitlab_ci

## Setup Ruby (using RVM)

1. su - gitlab_ci
1. \curl -L https://get.rvm.io | bash -s stable &#45;&#45;ruby
1. echo "source /home/gitlab_ci/.rvm/scripts/rvm" >> ~/.bashrc

## Setup Gitlab CI

1. su - gitlab_ci
1. cd /home/gitlab_ci/
1. git clone https://github.com/gitlabhq/gitlab-ci.git
1. cd gitlab-ci
1. git checkout 2-2-stable
1. mkdir -p tmp/pids
1. mkdir -p tmp/sockets
1. gem install bundler
1. bundle &#45;&#45;without development test
1. cp config/database.yml.mysql config/database.yml
1. vim config/database.yml

```
    production:
      adapter: mysql2
      encoding: utf8
      reconnect: false
      database: gitlab_ci
      pool: 5
      username: gitlab_ci
      password: "12345678"
      host: localhost
      # socket: /tmp/mysql.sock

    ......
```

1. bundle exec rake db:setup RAILS_ENV=production
1. bundle exec whenever -w RAILS_ENV=production
1. crontab -l

## Setup gitlab_ci Service Script

1. su - root
1. wget https://raw.github.com/gitlabhq/gitlab-ci/2-2-stable/lib/support/init.d/gitlab_ci -P /etc/init.d/ &#45;&#45;no-check-certificate
1. chmod +x /etc/init.d/gitlab_ci
1. /etc/init.d/gitlab_ci start
1. /etc/init.d/gitlab_ci status

## Setup Nginx

1. yum install nginx
1. wget https://raw.github.com/gitlabhq/gitlab-ci/2-2-stable/lib/support/nginx/gitlab_ci -P /etc/nginx/conf.d/ &#45;&#45;no-check-certificate
1. mv /etc/nginx/gitlab_ci /etc/nginx/gitlab_ci.conf
1. vim /etc/nginx/gitlab_ci.conf

```
    ......

    server {
      listen YOUR_SERVER_IP:80;         
      server_name YOUR_SERVER_NAME;          
      root /home/gitlab_ci/gitlab-ci/public;

    ......
```

1. chkconfig nginx on
1. service nginx start
1. netstat -ltn

Gitlab Default Account: admin@local.host      
Gitlab Default Password: 5iveL!fe      

