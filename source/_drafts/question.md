语速慢，留有思考时间

1. 对称加密算法 非对称加密算法

2. startservice bindservice

3. android o 特性

4. 启动过程

bootloader - linux kernel - init process 两个责任，一是挂载目录，比如/sys、/dev、/proc，二是运行init.rc脚本 -  zygote/vm - system server - service manager 服务实例，如：电源、网络、Wifi、蓝牙，USB等 - home launcher

核心服务：

启动电源管理器；
创建Activity管理器；
启动电话注册；
启动包管理器；
设置Activity管理服务为系统进程；
启动上下文管理器；
启动系统Context Providers；
启动电池服务；
启动定时管理器；
启动传感服务；
启动窗口管理器；
启动蓝牙服务；
启动挂载服务。


5. 如何降低奔溃率
code review
自动化测试
线上日志搜集

6. https自签名
1)客户端向服务器传输客户端的SSL协议版本号，支持的加密算法的种类，产生的随机数Key1及其他信息

2)服务器在客户端发送过来的加密算法列表中选取一种，产生随机数Key2，然后发送给客户端

3)服务器将自己的证书发送给客户端

4)客户端验证服务器的合法性，服务器的合法性包括：证书是否过期，发行服务器证书的CA是否可靠，发行者的公钥能否正确解开服务器证书的”发行者的数字签名”，服务器证书上的域名是否和服务器的实际域名相匹配，如果合法性验证没有通过，通信将断开，如果合法性验证通过，将继续向下进行；

5)客户端随机产生一个Pre-Master-Key，然后用服务器的公钥(从证书中获得)对其加密，然后将该Pre-Master-Key发送给服务器

6)服务器接收到Pre-Master-Key，则使用协商好的算法(H)计算出真正的用户通信过程中使用的对称加密密钥Master-Key=H(C1+S1+PreMaster);

7)至此为止，服务器和客户端之间都得到Master-Key，之后的通信过程就使用Master-Key作为对称加密的密钥进行安全通信；

针对SSL的中间人攻击方式主要有两类，分别是SSL劫持攻击和SSL剥离攻击

3.1 SSL劫持攻击

SSL劫持攻击即SSL证书欺骗攻击，攻击者为了获得HTTPS传输的明文数据，需要先将自己接入到客户端和目标网站之间；在传输过程中伪造服务器的证书，将服务器的公钥替换成自己的公钥，这样，中间人就可以得到明文传输带Key1、Key2和Pre-Master-Key，从而窃取客户端和服务端的通信数据；

但是对于客户端来说，如果中间人伪造了证书，在校验证书过程中会提示证书错误，由用户选择继续操作还是返回，由于大多数用户的安全意识不强，会选择继续操作，此时，中间人就可以获取浏览器和服务器之间的通信数据

3.2  SSL剥离攻击

这种攻击方式也需要将攻击者设置为中间人，之后见HTTPS范文替换为HTTP返回给浏览器，而中间人和服务器之间仍然保持HTTPS服务器。由于HTTP是明文传输的，所以中间人可以获取客户端和服务器传输数据
