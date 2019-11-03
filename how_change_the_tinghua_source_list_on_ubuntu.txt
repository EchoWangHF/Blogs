1 进入到/etc/apt文件夹当中，找到sources.list，将其备份。命令：cp -p sources.list sources.list.old

2 采用管理员方式打开sources.list: sudo vim sources.list

3 在清华源网站上https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/ 找到对应版本的清华源，将sources.list当中的内容进行替换。

4 更新软件包和缓存：sudo apt-get update
