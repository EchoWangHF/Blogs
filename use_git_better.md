# Git 使用备忘录

### 1 初始化git账号
git config --global user.name "自定义用户名"

git config --global user.email "邮箱"

### 2 生成git ssh public key
ssh-keygen -o -f ~/.ssh/id_rsa

cat ~/.ssh/id_rsa.pub 

### 3 将多个提交合并成一个提交
比如将前3个提交合并成1个提交： `git rebase -i HEAD~3`

将除第一个以外的提交的`pick`改为`fixup`

然后 `:wq` 退出保存即可。

### 4 如何将fork过来的仓库同步到原仓库
1 `cd` 到本地仓库的目录

2 `git remote -v` 查看你的仓库有没有上游分支 `upstream`, 如果没有 `git remote add upstream xxxx upstream ssh/http地址xxxx.git `

3 `git fetch upstream` 抓取  `upstream` 的更新

4 `git checkout master` 切到你自己的仓库的master当中，`git merge upstream/master` 合并你的分支和上游分支。

5 `git push` 到远程即可

### 5 如何将某一个文件从缓存池当中删除(git add 后的文件)
1 `git rm --cached + file_path`，不删除物理文件，仅将该文件从缓存中删除.

2 `git rm --f + file_path`，不仅将该文件从缓存中删除，还会将物理文件删除（不会回收到垃圾桶）.

### 6 如何在仓库当中关联一个子仓
1 `git submodule add xxxxx子仓链接xxxx.git` 添加子仓的git链接

2 `git submodule update --init --recursive` 更新子仓

3 删除已经关联的子仓：(1) 删除 `.gitsubmodule`里相关部分 （2）删除 `.git/config` 文件里相关字段 (3) 删除子仓库目录。

### 7 修改git默认启动的编辑器
1 在`.git/config` 文件的`core`下面添加`editor=vim`.
