---
layout: post
title: "使用 Git"
date: 2013-05-21T15:44:28+08:00
comments: true
external-url: 
categories: 
- Git
---

[Git][1] 是一套版本控制系統（Version Control System），
常用在對程式碼或文件進行版本管理及協作。
從前比較常用的版本控制系統是 SVN（Subversion）， 
但 SVN 是中央式的，而 Git 是分散式的，
且 Git 比 SVN 強大、先進許多。

我簡單解釋版本控制的概念，請想像一下：

你電腦內的資料夾類似一個書櫃，書櫃裡有許多書（檔案），
偶而有新書進來（新增檔案），有舊書捐出去（刪除檔案），
有時候會在某些書上作筆記、寫心得、畫畫（檔案內容變動）。

你在書櫃的側邊貼上一張「大大的白紙」，對書櫃裡的所有變動作紀錄。

紀錄新書擺入的時間，擺放的位置，甚至後面加個註記：『購於網路拍賣』；
紀錄舊書清除的時間，原本的位置，加個註記：『捐給讀書會』；
紀錄某書寫入筆記的時間，筆記的內容，心得或隨手塗鴉；
紀錄 ......

你可以把這張書櫃側邊貼著的「大大白紙」想像成版本控制系統（Git），
而且版本控制系統（Git）做的記錄會更鉅細靡遺。

當你對一個資料夾啟用 Git 進行追蹤管理與控制時（Git 初始化時），
Git 程式會在該資料夾底下新增一個名為「.git」的隱藏資料夾，
「.git」類似於前面提到那張「大大的白紙」，裡面紀錄了檔案的變化史。
Git 會對該資料夾內所有的檔案與其底層的所有資料夾進行紀錄追蹤，
而追蹤、記錄的結果都會儲存到「.git」這個資料夾內。

不過，Git 並不會主動記錄，必須是由使用者操作它去執行記錄的動作。
使用者類似史官的角色，而 Git 則是書寫史冊的工具。

Git 背後的運作方式是非常聰明而複雜的，
它的功能也不僅僅在於記錄（還有恢復、合併、差異處理等等）。

團隊合作時，同樣一個文件，在你手上跟在他人手上，可能有不一樣的變化史。
當你的檔案要與他人的合併時，內容有出入的地方，Git 會協助進行處理。
（例如開發同一個程式，你寫的 code 可能被他人改動，或反之。）

### 安裝與設定 Git

Ubuntu 底下安裝 Git 非常簡單，只要在終端機執行：

    sudo apt-get install git-core git-doc

接着，建議執行以下指令，將系統預設編輯器設定為 Vim：

    sudo update-alternatives --config editor

每個使用者帳號都會有自己的 Git 設定檔，通常是：

    ~/.gitconfig

例如我的設定檔內容是：

    [user]
    name = 張旭
    email = zx1986@gmail.com

    [color]
    diff = auto

初始化 Git
（資料夾內會多出一個名為 .git 的隱藏資料夾）：

    git init

之後只要每次修改或新增檔案後，
執行以下兩個指令，Git 就會做一次紀錄：

    git add 修改或新增的檔案名
    git commit -m '關於此次修改的描述訊息'

可以開一個新的資料夾進行練習：

    mkdir test
    cd test
    touch hello
    git init
    git add hello
    git commit -m 'hello, Git!'

好了，您已經開始在使用 Git 啦！

### Git 輔助說明

需要注意，Git 不會把空的資料夾加入控管，
例如 log、cache 這類資料夾，我們通常不會想要追蹤裏面的檔案，
但還是需要這個資料夾存在，可以在底下建立一個隱藏檔，例如 .gitkeep：

    cd log
    tocuh .gitkeep

在跟「.git」同一層的目錄中，可以建立一個 .gitignore 檔案。
.gitignore 用來設定「不希望被 Git 控管的檔案與資料夾」。
一個簡單的 .gitignore 內容可以是這樣：

    cache/
    log/*.log
    tmp/
    *.tmp
    *.swp
    *.o
    *.so
    *.a
    *.exe
    *~

執行 `git help` 會顯示常用的 Git 指令與簡單說明：

    add        Add file contents to the index
    bisect     Find by binary search the change that introduced a bug
    branch     List, create, or delete branches
    checkout   Checkout a branch or paths to the working tree
    clone      Clone a repository into a new directory
    commit     Record changes to the repository
    diff       Show changes between commits, commit and working tree, etc
    fetch      Download objects and refs from another repository
    grep       Print lines matching a pattern
    init       Create an empty git repository or reinitialize an existing one
    log        Show commit logs
    merge      Join two or more development histories together
    mv         Move or rename a file, a directory, or a symlink
    pull       Fetch from and merge with another repository or a local branch
    push       Update remote refs along with associated objects
    rebase     Forward-port local commits to the updated upstream head
    reset      Reset current HEAD to the specified state
    rm         Remove files from the working tree and from the index
    show       Show various types of objects
    status     Show the working tree status
    tag        Create, list, delete or verify a tag object signed with GPG

在 Git 指令後加上 -h 參數，能夠查詢該指令詳細的用法，例如：

    git add -h
    git commit -h
    git pull -h
    git push -h

要查詢更完整的指令手冊，則執行：`git help 指令名稱`

### 練習 Git

我推薦一個非常棒的線上練習：[Try Git](http://try.github.io)

這是由線上學習網站 [Code School](http://codeschool.com) 開發的，
他們還有一個 Git Real 的系列課程，我也非常推薦，
但 Git Real 系列課程是需要收費的。

Try Git 會帶着你練習基礎的 Git 指令，
畫面上半部是簡單明瞭的指令與情境說明，
畫面中間是一個模擬的 Console 端介面，
畫面下半部是當前情境下資料夾內的狀況。
雖然內容都是英文，但用字遣詞並不難，
就當成是玩電動闖關，邊玩邊學。

當進行到 Remote Repositories 這個關卡時，
如果對 Repository、Local、Remote 不大瞭解，
可以再回來這，繼續閱讀後面的內容。

### 關於 Git Repository 與 Branch

被 Git 所管理的專案，就是 Git Repository（倉儲）。
簡單點說，一個含有「.git」的資料夾，就是一個 Git Repository。
而一個 Git Repository 內，可以建立很多 Branch（分支），
不同的分支代表不同的變化史。

分支是可以任意建立、刪除、合併的。

Git Repository 裡預設的 Trunk Branch（主幹）稱為「master」，
其他的 Branch（分支）則由使用者自行命名，
master 這個 branch 也是可以被改成其他名字的。
（其實 Git 的世界並沒有 Trunk 這種講法，那是 SVN 的習慣，
在 Git 的世界，應該說那是“一個被叫做 master 的 branch”。）

Git 是一個分散式的版本控制系統，不同於 SVN 的 Server/Client 架構，
Git 不需要像 SVN 必須有一個 Repository Server 作為主要的倉儲伺服器。
Git 預設就可以使用 ssh 互相進行 Repository 傳輸了。

當使用 git clone 指令從遠端複製一個 Git Repository 到本地端電腦上時，
遠端的 Git Repository 通常稱為「origin」，「origin」可能有若干的 branch。
本地端 Git Repository 沒有特別的名稱，本地端也有自己的 branch。

`git remote -v` 指令可以查詢當前 Git Repository 的遠端來源。

假設一個簡單的應用情境：

假設遠端的電腦叫做 Remote；本地端的電腦叫做 Local。
Remote 上面有一個 Git Repository 資料夾叫做 remote_repository。

要將 remote_repository 複製到 Local 並命名為 local_repository，
在 Local 執行：

    Local$ git clone 「Remote 使用者帳號@Remote 位址」:「remote_repository 在 Remote 上的路徑」   local_repository

如果您執行過 scp 指令，相信對這個 git clone 格式會覺得很熟悉。

複製完成（git clone）後，Local 與 Remote 已經可以分開獨立工作了。
Git 不必拘泥於一定要把修改過的檔案存回當初取得檔案的地方。
Remote 可以在 remote_repository 裡發展它的檔案；
Local 可以在 local_repository 裡發展它的檔案。

等到哪天 Remote 突然想取得並合併 Local 發展的檔案，
可以在 Remote 上執行：

    Remote$ git pull 「Local 使用者帳號@Local 位址」:「local_repository 在 Local 上的路徑」

當然，如果 Local 想取得與合併其他人發展的檔案，
可以在 Local 上執行：

    Local$ git pull 「使用者帳號@位址」:「路徑」

* 補充說明

git fetch - Download objects and refs from another repository.   
git merge - Join two or more development histories together.   
git pull - Fetch from and merge with another repository or a local branch.   

`git pull` 等於先執行了 `git fetch`，然後再自動執行 `git merge` 。
[有前輩建議][4]少用 `git pull`，改用 `git fetch` 搭配 `git merge` 。
[也有前輩建議][5]應當多使用 `git rebase` 或 `git pull --rebase` 。

### SVN 式的往日時光

之前，我在不同的電腦上修改程式：研究室的電腦、宿舍的電腦、筆記型電腦。
因此我在研究室的一台主機上架了 SVN 伺服器，程式主要版本儲存在 SVN 伺服器上。
每當在不同的電腦進行程式編輯時，會先從 SVN 伺服器上抓最新版本的程式下來。
編輯告一段落後，再把修改過的程式上傳回 SVN 伺服器。
程式集中在一台 SVN 伺服器上，要編輯時從上面更新下來，編輯完再更新回去。

從 SVN 轉換到 Git 時，會很習慣於從前 SVN 那種模式：

1. 使用 svn checkout 從 SVN 伺服器將整個 Repository 複製到本機端。
2. 本機端對 Repository 的內容進行編輯、修改、新增、刪除等等。
3. 使用 svn update 檢查 SVN 伺服器有沒有其他更新與自己修改的內容有衝突。
4. 解決內容衝突的情況。
5. 使用 svn commit 將自己本機端的所有修改上傳到 SVN 伺服器。

怎麼用 Git 做到類似 SVN 那樣的情形？
有個簡單的方法。

首先，選定一台要當 Repository Server 的機器，假設叫 Server。
在 Server 開一個空的資料夾，假設叫 origin，並切換到該資料夾下。

    Server$ mkdir origin
    Server$ cd origin

在空資料夾底下執行：

    Server$ git init --bare

這個動作會產生一個 [Bare Git Repository][2]，
該資料夾下會產生：

    branches/
    config
    description
    HEAD
    hooks/
    info/
    objects/
    refs/

本地端的電腦，假設叫 Local。
Local 上一個叫 local_project 的資料夾要上傳到 Server 進行統一管理。

切換到該資料夾底下，執行：

    Local$ git add .
    Local$ git commit -a -m 'initialization'
    Local$ git remote add origin 「Server 使用者帳號@Server 位址」:「Server 上 origin 資料夾的路徑」
    Local$ git push origin master

完成以後，其他的電腦就可以使用以下指令，
從 Server 上的 origin 複製 local_project 的內容：

    Other$ git clone 「Server 使用者帳號@Server 位址」:「Server 上 origin 資料夾的路徑」 「自訂的資料夾名稱」

其他電腦要將其修改的內容傳回 Server，可以執行：

    Other$ git push origin master

### Git Repository Hosting

Git Repository 除了使用 git 協定或 ssh 協定，
還可以使用 http、https 等方式傳輸、瀏覽、管理。
如果不想要用簡單的 Bare Git Repository 架設 Repository Server，
有些 Open Source 的工具可以選擇：

- [Gitosis](https://github.com/res0nat0r/gitosis)
- [Gitorious](http://gitorious.org/)
- [Gitlab](http://gitlab.org/)

全世界最知名的 Git Repository Hosting 網站：

- [Github](https://github.com/)

中國大陸版的 Github：

- [GitCafe](https://gitcafe.com/)

Github 與 GitCafe 都非常棒，
它們的使用教學也有很多 Git 技巧可以參考：

https://help.github.com/   
https://gitcafe.com/GitCafe/Help   

GitCafe 的 Git [作弊表][3]。

_Git Cheating Sheet_

    git init         # 將當前資料夾進行 Git 初始化

    git add .        # 將當前資料夾內所有檔案加入 Git 追蹤（tracking 或 staging）
    git add 檔案名稱 # 把當前資料夾內某個檔案加入 Git 追蹤（tracking 或 staging）

    git status       # 查詢從上一次 commit 到現在，資料夾裡有哪些變化，各個檔案處於什麼狀況

    git commit -a         # 將目前的變動送繳 Git 進行紀錄，會進入編寫修改訊息的畫面
    git commit -a -m "*"  # commit 時直接寫入修改訊息，不進入編寫修改訊息的畫面

    git tag v1.0          # 將當前 commit 過後的檔案版本命名為 v1.0

    git diff                             # 比較所有檔案的內容與上一次 commit 時有何差異
    git diff v1.0 v2.0                   # 比較 v1.0 與 v2.0 兩個版本間所有檔案的內容
    git diff v1.0:檔案名稱 v2.0:檔案名稱 # 比較 v1.0 與 v2.0 兩個版本間某個檔案的內容

    git log                         # 查詢所有版本的修改狀況，顯示各版本的 hash 編號
    git log -p                      # 查詢哪幾行被修改
    git log --stat --summary        # 查詢每個版本間變動的檔案跟行數

    git show v1.0                   # 查詢 v1.0 版裡的修改內容
    git show v1.0:檔案名稱          # 查詢某個檔案在 v1.0 時的內容

    git show HEAD          # 看此版本修改的資料
    git show HEAD^         # 看此版本前一版的修改的資料
    git show HEAD^^        # 看此版本前前一版的修改的資料

    git grep "*" v1.0      # 查詢 0.01 版裡頭有沒有某些內容
    git grep "*"           # 查詢現在的版本裡有沒有某些內容

    git branch                # 查看現有的分支
    git branch 分支名稱       # 建立新的分支
    git branch 分支名稱 v1.0  # 依照 v1.0 版本裡的內容來建立一個分支
    git branch -d 分支名稱    # 刪除某個分支

    git merge 某個分支名稱    # 將當前所在的分支與某個分支合併，如果出現衝突，會紀錄在有衝突的檔案中

    git checkout master       # 切換到主幹上
    git checkout 分支名稱     # 切換到某個分支上

    git checkout HEAD         # 將所有檔案恢復到上次 commit 的狀態
    git checkout -- 檔案名稱  # 將某個檔案恢復到上次 commit 的狀態

    git reset --hard 某個版本的 hash 編號   # 整個 Repository 恢復到某個版本的狀態

    git count-objects     # 分析 Git 資料庫狀況，計算鬆散的物件
    git gc                # 維護 Git 資料庫，重組物件
    git fsck --full       # 應該是類似 Git 磁碟重組之類的東西

Reference：   
http://github.com/schacon/whygitisbetter   
http://git-scm.com/documentation   
http://gitcafe.com/riku/GitTips   
http://gitimmersion.com/    
http://gitref.org    
http://rypress.com/tutorials/git/    

[1]: http://git-scm.com/
[2]: http://www.saintsjd.com/2011/01/what-is-a-bare-git-repository/
[3]: https://gitcafe.com/GitCafe/Help/blob/master/Git/Git_Cheat_Sheet.md
[4]: http://longair.net/blog/2009/04/16/git-fetch-and-merge   
[5]: http://ihower.tw/blog/archives/3843   
