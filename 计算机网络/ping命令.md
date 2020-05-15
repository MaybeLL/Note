# ping命令
![ping命令](https://s1.ax1x.com/2020/05/15/YrZPk8.png)



## TTL
>存活时间（英语：Time To Live，简写TTL）是电脑网络技术的一个术语，指一个数据包在经过一个路由器时，可传递的最长距离（跃点数）。每当数据包经过一个路由器时，其存活次数就会被减一。当其存活次数为0时，路由器便会取消该数据包转发，IP网络的话，会向原数据包的发出者发送一个ICMP TTL数据包以告知跃点数超限。其设计目的是防止数据包因不正确的路由表等原因造成的无限循环而无法送达及耗尽网络资源。


### 通过TTL来判断服务器操作系统类型
TTL默认初始值：默认情况下，Linux系统的TTL值为64或255，Windows NT/2000/XP系统的TTL值为128，Windows 98系统的TTL值为32，UNIX主机的TTL值为255。

```
ping google.cn
```
来自 203.208.40.127 的回复: 字节=32 时间=154ms TTL=116

TTL接近128大致可以猜测是windows

验证猜测：tracert google.cn
```
tracert google.cn
```

![tracert](https://s1.ax1x.com/2020/05/15/YruBVI.png)
路径12个路由，刚好128-116 =12

## ping 127.0.0.1
