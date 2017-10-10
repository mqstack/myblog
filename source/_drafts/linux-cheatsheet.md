

## 获取linux版本
Kernel Version
$ uname -a

Distribution Information
$ lsb_release -a


## 创建一个拥有超级权限的新用户：

root@localhost:~# useradd -m -s /bin/bash yangxg

root@localhost:~# usermod -a -G sudo yangxg

root@localhost:~# passwd yangxg

root@localhost:~# su - yangxg

yangxg@localhost:~$



