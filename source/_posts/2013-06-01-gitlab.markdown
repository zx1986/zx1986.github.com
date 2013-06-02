---
layout: post
title: "安裝 Gitlab"
date: 2013-06-01T23:04:05+08:00
comments: true
external-url: 
categories: 
- Git
- Linux
---

[Gitlab](http://gitlab.org) 是一套 Open Source 的 Git 專案管理系統。
可以把它想象成是 Open Source 的 [Github](http://github.com) 網站。

Gitlab 是使用 Rails 搭配 Gitlab Shell 來實現 Git 專案管理，
其中 Rails 框架負責網頁與專案管理（account、project、issue、wiki 等）相關的東西，
在背後透過 [grit](https://github.com/mojombo/grit)： 
一個 Ruby 語言實作的操作 Git 的函式庫，
搭配 [Gitlab Shell](https://github.com/gitlabhq/gitlab-shell)：
一個類似 [Gitolite](https://github.com/sitaramc/gitolite) 的 Git Repositories 存取控制系統，
來控制實際在檔案系統（File System）中的 Git Repositories。
題外話，Gitlab 在 [5.0 以前][2]，底層的存取控制系統使用的還是 Gitolite。

把 Gitlab 架設起來之後，等於是有了一臺自己的 Git Projects Hosting Server。
其它類似 Gitlab 的 Open Source 軟體還有：
同樣用 Rails 框架開發的 [Gitorious](http://gitorious.org/)；
使用 PHP 的 Pluf 框架開發的 [Indefero](http://www.indefero.net/)。

本文記錄 RHEL 6.3 x86_64 環境中安裝 Gitlab 的過程。
如果是 Ubuntu 的環境要安裝 Gitlab 會更簡單，請直接參考[官方的安裝手冊][1]。

## 安裝相依性套件

1. cd /opt
1. wget http://mirror01.idc.hinet.net/EPEL/6/i386/epel-release-6-8.noarch.rpm
1. rpm -iUvh epel-release-6-8.noarch.rpm
1. yum clean all
1. yum list
1. yum remove gitosis
1. yum groupinstall "Development Tools" 
1. yum update && yum upgrade
1. yum install vim python wget curl git openssh-server
1. yum install sqlite sqlite-devel mysql mysql-libs mysql-devel
1. yum install ncurses-devel libcurl-devel libcurl curl patch sudo
1. yum install libxslt-devel libyaml-devel libxml2 libxml2-devel gdbm-devel libffi libffi-devel zlib zlib-devel openssl-devel readline readline-devel curl-devel openssl-devel pcre-devel memcached-devel valgrind-devel ImageMagick-devel ImageMagick libicu libicu-devel make bzip2 autoconf automake libtool bison redis
1. chkconfig redis on
1. service redis start
1. yum install postfix
1. chkconfig postfix on
1. service postfix start

如果EPEL 套件庫少東西，請到 http://pkgs.org/ 把套件手動補齊。
比較常有問題的是以下這幾個套件：

* gdbm-devel
* libffi-devel
* libicu-devel
* iconv-devel
* valgrind-devel
* ImageMagick-devel

## 設定 Gitlab 系統帳號

1. useradd git
1. passwd git
1. chmod 755 /home/git
1. su - git
1. mkdir -p /home/git/.ssh
1. chmod -R 700 /home/git/.ssh
1. touch /home/git/.ssh/authorized_keys
1. chmod -R 600 /home/git/.ssh/authorized_keys
1. ssh-keygen -t rsa
1. cat /home/git/.ssh/id_rsa.pub >> /home/git/.ssh/authorized_keys

## 安裝 Ruby 環境

1. su - git
1. mkdir -p /home/git/temp
1. mkdir -p /home/git/ruby
1. cd /home/git/temp
1. wget -c "http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p392.tar.gz"
1. tar zxvf ruby-1.9.3-p392.tar.gz && cd ruby-1.9.3-p392
1. ./configure &#45;&#45;prefix=/home/git/ruby
1. make
1. make install
1. vim ~/.bash_profile   `PATH=$HOME/ruby/bin:$PATH:$HOME/bin;`
1. vim ~/.bashrc   `PATH=$HOME/ruby/bin:$PATH:$HOME/bin;`
1. source ~/.bash_profile
1. which ruby
1. ruby &#45;&#45;version
1. gem install bundler
1. gem install charlock_holmes &#45;&#45;version '0.6.9'
1. cd ~
1. rm -rf ~/temp

## 設定資料庫

1. yum install mysql-server mysql myslq-devel
1. chkconfig mysqld on
1. service mysqld start
1. mysql -u root -p
1. mysql> CREATE DATABASE IF NOT EXISTS \`gitlab\` DEFAULT CHARACTER SET \`utf8\` COLLATE \`utf8_unicode_ci\`;
1. mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY '12345678';
1. mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON \`gitlab\`.* TO 'gitlab'@'localhost';

## 安裝 Gitlab Shell

1. su - git
1. cd /home/git
1. git clone http://github.com/gitlabhq/gitlab-shell.git
1. cd gitlab-shell
1. cp config.yml.example config.yml
1. vim config.yml

```
# GitLab user. git by default
user: git

# Url to gitlab instance. Used for api calls. Should be ends with slash.
gitlab_url: "http://test-git.mfc.cwb/"

http_settings:
#  user: someone
#  password: somepass
   self_signed_cert: false

# Repositories path
repos_path: "/home/git/repositories"

# File used as authorized_keys for gitlab user
auth_file: "/home/git/.ssh/authorized_keys"
```

1. /home/git/gitlab-shell/bin/install

## 安裝 Gitlab

1. su - git
1. cd /home/git
1. git config &#45;&#45;global user.name  "GitLab"
1. git config &#45;&#45;global user.email "gitlab@gitlab.YOUR.HOST"
1. mkdir gitlab-satellites
1. git clone http://github.com/gitlabhq/gitlabhq.git gitlab
1. cd gitlab
1. git checkout 5-0-stable
1. more VERSION
1. cp config/gitlab.yml.example config/gitlab.yml
1. vim config/gitlab.yml

```
## GitLab settings
  gitlab:

## Web server settings
  host: test-git.mfc.cwb
  port: 80
  https: false
  user: git

## Gravatar
  gravatar:
  enabled: false
```

1. mkdir tmp/pids/
1. chown -R git log/ tmp/
1. chmod -R u+rwX  log/ tmp/
1. cp config/unicorn.rb.example config/unicorn.rb
1. vim config/unicorn.rb   `listen "127.0.0.1:8080"`
1. cp config/database.yml.mysql config/database.yml
1. vim config/database.yml

```
production:
  adapter: mysql2
  encoding: utf8
  reconnect: false
  database: gitlab
  pool: 5
  username: gitlab
  password: "12345678"
  host: localhost
```

1. cd /home/git/gitlab
1. <del>編輯 Gemfile 並將所有 git:// 與 https:// 改成 http://</del>
1. bundle install &#45;&#45;no-deployment
1. bundle install &#45;&#45;deployment &#45;&#45;without development test postgres
1. bundle exec rake gitlab:setup RAILS_ENV=production
1. <del>bundle exec rake db:seed_fu RAILS_ENV=production</del>
1. bundle exec rake gitlab:env:info RAILS_ENV=production
1. bundle exec rake gitlab:check RAILS_ENV=production

* 碰到 invalid gem format for /path/to/cache/xxx.gem 問題，
刪掉該 cache 的 gem，再重跑 bundle install 即可

## 設定 Gitlab 啓動腳本

1. su - root
1. curl &#45;&#45;output /etc/init.d/gitlab https://raw.github.com/gitlabhq/gitlab-recipes/master/init.d/gitlab
1. chkconfig &#45;&#45;add gitlab
1. chkconfig gitlab on
1. chmod +x /etc/init.d/gitlab
1. /etc/init.d/gitlab start
1. /etc/init.d/gitlab status
1. Unicorn 與 Sidekiq 服務跑起來需要點時間，要等一下

## 設定 Apache

1. su - root
1. yum install httpd
1. vim /etc/httpd/conf/httpd.conf

```
NameVirtualHost *:80

<VirtualHost *:80>
    ServerName gitlab.YOUR.HOST
    DocumentRoot /home/git/gitlab/public
    CustomLog logs/gitlab-access.log combined
    ErrorLog logs/gitlab-error.log

    ProxyPass /  http://127.0.0.1:8080/
    ProxyPassReverse /  http://127.0.0.1:8080/
    ProxyPreserveHost On
</VirtualHost>
```

1. chmod 755 /home/git
1. chkconfig httpd on
1. service httpd restart

Gitlab 預設帳號：admin@local.host   
Gitlab 預設密碼：5iveL!fe   

[1]: https://github.com/gitlabhq/gitlabhq#installation
[2]: http://blog.gitlab.org/gitlab-5-dot-0-has-been-released/
