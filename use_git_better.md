# Git 使用备忘录

### 初始化git账号
git config --global user.name "自定义用户名"

git config --global user.email "邮箱"

### 生成git ssh public key
ssh-keygen -t rsa -C "yourmail@xxx.com"

cat .ssh/id_rsa.pub 
