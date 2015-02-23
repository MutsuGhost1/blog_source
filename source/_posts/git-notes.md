title: Git Notes
date: 2015-02-05 21:09:56
tags: [Software, Git]
---

本篇紀錄一些 Git 的用法. <!--more-->

## Origin ##
* 假設你們團隊有個地址為 git.ourcompany.com 的 Git 伺服器.
  如果你從這裡克隆, Git 會自動為你將此遠端倉庫命名為 origin, 並下載其中所有的資料, 建立一個指向它的 master 分支的指標,
  在本地命名為 origin/master，但你無法在本地更改其資料。
* 為了演示擁有多個遠端分支（在不同的遠端伺服器上）的專案是如何工作的,
  我們假設你還有另一個僅供你的敏捷開發小組使用的內部伺服器 git.team1.ourcompany.com.
  可以用 git remote add 命令把它加為當前專案的遠端分支之一. 
  我們把它命名為 teamone, 以便代替完整的 Git URL 以方便使用.

## Remote Tracking Branch ##
* 為何 git 可以知道 master branch 是對應到 origin/master branch 呢 ?
  雖然我們可以從名稱中, 大致猜到有如此的對應關係.
    * 可以從下列兩種情況中, 確定這樣的對應關係.
        1. 使用 pull 的時候, 下載 commit 到 origin/master, 並且 merge 這些 commit 到 master branch,
           這就表示這個 merge 的目標是決定於這個對應關係.
        2. 使用 push 的時候, 在 master brach 上面的 commit 被 push 到 remote 上面的 master branch.
           (它在 local 被表示成 origin/master), 這就表示 push 的目標是決定於 master 以及 origin/master 之間的對應關係.
    * 事實上, local branch 與 remote branch 之間的關係, 是用 "remote tracking" 特性來表示的.
      這將表示 master 與 origin/master 的對應關係, master branch 被設定用來 track origin/master,
      這就表示對於 master branch 來說的話, 有一個 merge 的目標與 push 的目標.
    * 一般來講, 這樣的關係在做完 git clone 的時候, git 會針對每一個在 remote 上面的 branch 建立一個 branch (ex: origin/master),
      之後他會建立一個 local branch 來追蹤目前在 remote 上面的 active branch, 而大部分的狀況下, 幾乎都是設定 master branch.
      這也解釋為何當你 clone 的時候可能會看到以下的指令被輸出:
      local branch "master" set to track remote branch "origin/master"
* 如何自己設定 local branch 與 remote branch 的 remote tracking 關係呢 ?
    1. git checkout -b new_branch origin/remote_branch (建立 branch 的時候就綁定 remote tracking 關係)
    2. git branch -u origin/remote_branch branch_name  (將指定的 branch 與 remote branch 綁定 remote tracking 關係)
       git branch -u origin/remote_branch              (將 current branch 與 remote branch 綁定 remote tracking 關係)
* 如何看目前 branch 與 remote branch 之間的 remote tracking 關係 ?
* 如何解除或更改目前 branch 與 remote branch 之間的 remote tracking 關係嗎 ?

## Push 相關參數 ##
* 如果目前 checkout 的 branch 有對應的 remote tracking branch, 這時候可以直接執行 git push
* git push 的完整參數應為: 
    * git push <remote> <source>:<destination>
    * <source> 表示任意可以被 git 識別的位置
* 有時候也可以簡略成:
    * git push <remote> <source>
    * 這時候就是將 <source> 的變動, 推送到其對應的 remote tracking branch 

## Reference ##
1. [pull with conflict](http://backlogtool.com/git-guide/tw/stepup/stepup3_1.html "Pull with Conflict")
2. [rebase -i 合併提交](http://backlogtool.com/git-guide/tw/stepup/stepup7_5.html "rebase -i 合併提交")
3. [rebase -i 修改提交](http://backlogtool.com/git-guide/tw/stepup/stepup7_6.html "rebase -i 修改提交")
4. [merge --squash 將提交合併成一筆 Merge](http://backlogtool.com/git-guide/tw/stepup/stepup7_7.html "Merge --squash 將提交合併成一筆 Merge")
5. [git learning -- 透過遊戲學習 git](http://pcottle.github.io/learnGitBranching/)
6. [origin -- 說明 origin](http://git-scm.com/book/zh-tw/v1/Git-%E5%88%86%E6%94%AF-%E9%81%A0%E7%AB%AF%E5%88%86%E6%94%AF)