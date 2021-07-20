# Git 使用备忘录

### 1 初始化git账号
git config --global user.name "自定义用户名"

git config --global user.email "邮箱"

### 2 生成git ssh public key
ssh-keygen -t rsa -C "yourmail@xxx.com"

cat .ssh/id_rsa.pub 

### 3 将多个提交合并成一个提交
比如将前3个提交合并成1个提交： `git rebase -i HEAD~3`

将除第一个以外的提交的`pick`改为`fixup`

然后 `:wq` 退出保存即可。
