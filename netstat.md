# netstat
## netstat -st输出的两个重要信息来源分别是/proc/net/snmp和/proc/net/netstat


## Linux 网络常见监控项以及报错
### 查看丢包
网络丢包会有多种可能，例如，交换机，上连和下连端口的流量跑满或链路有问题，那么数据包就有可能会被交换机丢掉；负载均衡设备，包括了硬件设备以及软件的负载均衡。在此，我们仅查看本机可能导致的掉包。

### 操作系统处理不过来，丢弃数据
有两种情况，一是网卡发现操作系统处理不过来，丢数据包，可以读取下面的文件：

``` $ cat /proc/net/dev ```

每个网络接口一行统计数据，第 4 列（errs）是接收出错的数据包数量，第 5 列（drop）是接收不过来丢弃的数量。

第二部分是传统非 NAPI 接口实现的网卡驱动，每个 CPU 有一个队列，当在队列中缓存的数据包数量超过 net.core.netdev_max_backlog 时，网卡驱动程序会丢掉数据包，这个见下面的文件：

``` $ cat /proc/net/softnet_stat ```

每个 CPU 有一行统计数据，第二列是对应 CPU 丢弃的数据包数量。

### 应用程序处理不过来，操作系统丢弃

内核中记录了两个计数器：

* ListenOverflows：当 socket 的 listen queue 已满，当新增一个连接请求时，应用程序来不及处理；

* ListenDrops：包含上面的情况，除此之外，当内存不够无法为新的连接分配 socket 相关的数据结构时，也会加 1，当然还有别的异常情况下会增加 1。

分别对应下面文件中的第 21 列（ListenOverflows）和第 22 列（ListenDrops），可以通过如下命令查看：

``` $ cat /proc/net/netstat | awk '/TcpExt/ { print $21,$22 }' ```

如果使用 netstat 命令，有丢包时会看到 “times the listen queue of a socket overflowed” 以及 “SYNs to LISTEN sockets ignored” 对应行前面的数字；如果值为 0 则不会输出对应的行。

### Out of memory

如上所示，出现内存不足可能会有两种情况：

* 有太多的 orphan sockets，通常对于一些前端的服务器经常会出现这种情况。

* 分配给 TCP 的内存确实较少，从而导致内存不足。

#### 内存不足

这个比较好排查，只需要查看一下实际分配给 TCP 多少内存，现在时用了多少内存即可。需要注意的是，通常的配置项使用的单位是 Bytes，在此用的是 Pages，通常为 4K 。

先查看下给 TCP 分配了多少内存。

``` $ cat /proc/net/sockstat
sockets: used 855
TCP: inuse 17 orphan 1 tw 0 alloc 19 mem 3
UDP: inuse 16 mem 10
UDPLITE: inuse 0
RAW: inuse 1
FRAG: inuse 0 memory 0 ```

其中的 mem 表示使用了多少 Pages，如果相比 tcp_mem 的配置来说还很小，那么就有可能是由于 orphan sockets 导致的。

### Procfs 文件系统

*/proc/net/tcp：记录 TCP 的状态信息。
