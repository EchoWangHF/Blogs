一般情况下，将 `/etc/apt/sources.list` 文件中 `Ubuntu` 默认的源地址 `http://archive.ubuntu.com/` 替换为 `http://mirrors.ustc.edu.cn/` 即可
```
sudo sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
```
其他操作可以参考：https://mirrors.ustc.edu.cn/help/ubuntu.html

-----------------------------------------------update-----------------------------------------------
1 进入到/etc/apt文件夹当中，找到sources.list，将其备份。命令：cp -p sources.list sources.list.old

2 采用管理员方式打开sources.list: sudo vim sources.list

3 在清华源网站上https://mirror.tuna.tsinghua.edu.cn/help/ubuntu/ 找到对应版本的清华源，将sources.list当中的内容进行替换。

4 更新软件包和缓存：sudo apt-get update
