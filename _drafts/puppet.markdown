---
layout: post
title: "使用 Puppet"
date: 2013-05-26T21:43:54+08:00
comments: true
external-url: 
categories: 
- Ruby
- Linux
---

[Puppet](http://puppetlabs.com/) 
是用 Ruby 語言開發的 Client/Server 式的配置、設定、服務管理工具。

簡單描述一下，
就是所有機器的配置集中在 Puppet Server 上管理，
Puppet Client 機器連線到 Puppet Server，
Puppet Server 根據 Puppet Client 提供的資訊，
動態產生正確的配置，對 Puppet Client 進行設定。

Puppet Client 使用的是一個叫 facter 的工具取得主機基本資訊，
Puppet Server 根據傳來的資訊（例如：IP，hostname，OS）產生對應的配置內容。

在 Ubuntu 系統上安裝 Puppet 非常簡單：

    sudo apt-get install ruby-full # 安裝 Ruby 環境
    sudo apt-get install facter # 安裝 Facter 後可以直接執行 facter 測試一下
    sudo apt-get install puppet # Puppet Client 套件 
    sudo apt-get install puppetmaster # Puppet Server 套件，Puppet Client 不需安裝

    # 在 Redhat 環境，Puppet Client 套件叫：puppet
    # 在 Redhat 環境，Puppet Server 套件叫：puppet-server

Puppet 看待裝置或服務的哲學跟 UNIX 很像：
每一個裝置、服務對 Puppet 來說，都是一個「資源（Resource）」。

### *.pp 檔

Puppet 解讀名爲 *.pp 的檔案，然後對不同的 Client 編譯產生不同的配置。

一個簡單的 test.pp 檔案內容如下：

    file
    {
        "/tmp/test.txt": content => "hello, world";
    }

執行：

    puppet test.pp

執行結果是在 /tmp 底下產生一個 test.txt 檔，且內容爲「hello, world」。

*.pp 有一套簡單的文法與規則，其中亦有類別跟繼承等，強烈建議閱讀文件！

### Puppet Client/Server 運作流程

1. Puppet Client 的 puppetd 程式呼叫 facter 程式，
facter 程式會偵測出 Puppet Client 主機的相關資訊，
例如 hostname、RAM、Hard Disk、IP 等等。
Puppet Client 的 puppetd 再透過 SSL 把這些訊息傳到 Puppet Server。

2. Puppet Server 的 puppetmaster 程式檢查 Puppet Client 送來的資訊，
會使用 Puppet Client 的 hostname 來找到 /etc/puppet/manifest 裡面對應的 node 配置，
然後分析以及解讀牽涉到的 *.pp 檔或 Puppet 程式碼，
Puppet Client 使用 facter 生成的訊息會被當成變數傳入這些 *.pp 檔或 Puppet 程式碼。

3. 當 Puppet Server 知道需要處理哪些 *.pp 檔後，就會將它們進行解析，
這個 *.pp 檔解析的動作可以看成是程式的編譯（直譯？）。
解析會分成幾個階段，首先是語法檢查，語法錯誤就會直接報錯了；
語法檢查通過，會產生解析的結果，
這個結果同樣會透過 SSL 傳送回 Puppet Client。

4. Puppet Client 收到 Puppet Server 的解析結果，執行，並把執行的結果回傳。

5. Puppet Server 把 Puppet Client 的執行結果寫到 log。

### Puppet Client/Server 配置

Puppet Client 與 Puppet Server 的安裝、設定、配置，最好都使用 root 帳號。

進行 Client/Server 設置前，請務必千萬確定每臺主機的 hostname！
Puppet 的 Client/Server 配置有綁定 hostname 進行驗證，
且要求符合 FQDN 的 hostname。
請先執行 hostname 並檢查 /etc/hostname 進行確定，
並確認 /etc/hosts 檔案有正確的 hostname 與 IP 配對。

連線與驗證都需要 root 身份來進行。
Puppet Client 與 Server 的系統時間要校正一致，否則認證會出問題！
務必使用同樣的 NTP Server 進行時間同步！

在 Client/Server 環境中，
Puppet Server 預設讀取 *.pp 的路徑是：

    /etc/puppet/manifests/

可以編寫一個 /etc/puppet/manifests/site.pp 檔測試：

    node default
    {
        file
        {
            "/tmp/puppet_server.message":
            content => "Hello, Puppet Client!";
        }
    }

第一次進行 Client/Server 的配置連線會需要驗證。

先在 Puppet Client 執行：

    puppetd --server 伺服器主機名稱 --test

再到 Puppet Server 執行：

    puppetca --sign 客戶端主機名稱

最後回到 Puppet Client 執行：

    puppetd --server 伺服器主機名稱 --test

此時，Puppet Client 應該會得到一個 /tmp/puppet_server.message 檔。

### Puppet CA 驗證

Puppet 的驗證文件預設在：

    /var/lib/puppet/ssl # Puppet Client
    /var/lib/puppet/ssl/ca/ # Puppet Server

若某個 Puppet Client 的驗證有問題，
可以在 Puppet Server 使用 `puppetca --clean 主機名稱` 刪除認證，
相關指令可以參考：

    puppetca --list --all # 列出目前 Server 所有的 Client
    puppetca --list
    puppetca --sign CLIENT_HOSTNAME
    puppetca --print CLIENT_HOSTNAME
    puppetca --clean CLIENT_HOSTNAME  # 刪除舊的認證

    man puppetca

如果還是不行，直接到 Puppet Server 的 `/var/lib/puppet/ssl/ca/` 資料夾，
找到相對應的 *.pem，刪除，再重新認證 Puppet client。

### [建議的 /etc/puppet/ 目錄架構][1]

    manifests/
            site.pp
            templates.pp
            nodes.pp

    modules/
            {module_name}

    modules/user/

    services/

    clients/

    notes/

    plugins/

    tools/


### [使用 Puppet Module][2]

### [使用 Puppet Template][3]

Puppet Template 可以用來動態產生不同的內容。
應用的情境比較像是同樣的設定檔，但是有些許的設定不盡相同。
例如都是 httpd 服務，不見得所有 Puppet node 的 httpd.conf 都完全相同。

Puppet Template 檔是使用 Ruby 的 [ERB Template][5]，並不難寫。 
Template 的使用也很簡單：

    $value = template("my_template.erb")

my_template.erb 可換成 erb 檔的絕對路徑，
或把 erb 檔放到 Puppet 預設會找的路徑。
Red Hat 環境預設是 `/var/lib/puppet/templates`；
Ubuntu 環境預設是 `/etc/puppet/templates`。

可以使用 `puppet --configprint templatedir` 指令確認路徑，
實作的建議是把 erb 檔放到各個 Module 資料夾中。

題外話，可以執行 `puppet --configprint all` 看看 :-)

### Puppet 設定檔範例

https://github.com/example42/puppet-modules
https://github.com/ghoneycutt/puppet-generic   
https://github.com/jenkinsci/puppet-jenkins   
https://github.com/jfryman/puppet-nginx

### [Puppet 名詞解釋][4]

catalog : A catalog is the totality of resources, files, properties, etc, for a given system.

manifest : A configuration file written in the Puppet language. These files should have the .pp extension.

module : A collection of classes, resource types, files, and templates, organized around a particular purpose. 

node (general noun) : An individual server; for the purposes of discussing Puppet, this generally refers to an agent node.

node (Puppet language keyword) : A collection of classes and/or resources to be applied to the agent node whose unique identifier (“certname”) matches the specified node name. Nodes defined in manifests allow inheritance, although this should be used with care due to the behavior of dynamic variable scoping.

provider : A simple implementation of a type; examples of package providers are dpkg and rpm, and examples of user providers are useradd and netinfo. Most often, providers are just Ruby wrappers around shell commands, and they are usually very short and thus easy to create.

templates : templates are ERB files used to generate configuration files for systems and are used in cases where the configuration file is not static but only requires minor changes based on variables that Puppet can provide (such as hostname). See also distributable file.

type : abstract description of a type of resource. Can be implemented as a native type, plug-in type, or defined type.

agent or agent node : An operating system instance managed by Puppet. This can be an operating system running on its own hardware or a virtual image.

推薦文件：   
http://puppet.wikidot.com

Reference：   
http://puppet-manifest-share.googlecode.com/files/puppet-1.0.pdf    
http://www.comeonsa.com/category/puppet/    
http://bitcube.co.uk/content/puppet-errors-explained    
http://www.example42.com    
http://docs.puppetlabs.com/guides/troubleshooting.html    
http://docs.puppetlabs.com/guides/troubleshooting.html    
http://blog.akquinet.de/2011/11/23/managing-an-apache-server-with-puppet/    

[1]: http://projects.puppetlabs.com/projects/puppet/wiki/Puppet_Best_Practice2
[2]: http://docs.puppetlabs.com/guides/modules.html
[3]: http://docs.puppetlabs.com/learning/templates.html
[4]: http://docs.puppetlabs.com/references/glossary.html
[5]: http://www.ruby-doc.org/stdlib/libdoc/erb/rdoc/classes/ERB.html
