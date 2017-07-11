

# 过滤

## 表达式

1. 协议过滤, 比如TCP，只显示TCP协议。(tcp, http, dns)
2. IP 过滤

    比如 ip.src ==192.168.1.102 显示源地址为192.168.1.102，
    ip.dst==192.168.1.102, 目标地址为192.168.1.102

3. 端口过滤

    tcp.port ==80,  端口为80的
    tcp.srcport == 80,  只显示TCP协议的源端口为80的。

4. Http模式过滤

    http.request.method=="GET",   只显示HTTP GET方法的。

5. 逻辑运算符为 AND 或 OR


# 封包详细信息

* Frame:   物理层的数据帧概况
* Ethernet II: 数据链路层以太网帧头部信息
* Internet Protocol Version 4: 互联网层IP包头部信息
* Transmission Control Protocol:  传输层T的数据段头部信息，此处是TCP
* Hypertext Transfer Protocol:  应用层的信息，此处是HTTP协议

![](pic/wireshark01.png)

TCP 包的内容:

![](pic/wireshark02.png)




# 参考

* [Wireshark基本介绍和学习TCP三次握手](http://www.cnblogs.com/TankXiao/archive/2012/10/10/2711777.html)