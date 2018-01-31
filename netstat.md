# netstat
## netstat -st输出的两个重要信息来源分别是/proc/net/snmp和/proc/net/netstat


## Linux 网络常见监控项以及报错
### 查看丢包
网络丢包会有多种可能，例如，交换机，上连和下连端口的流量跑满或链路有问题，那么数据包就有可能会被交换机丢掉；负载均衡设备，包括了硬件设备以及软件的负载均衡。在此，我们仅查看本机可能导致的掉包。

### 操作系统处理不过来，丢弃数据
有两种情况，一是网卡发现操作系统处理不过来，丢数据包，可以读取下面的文件：

```
$ cat /proc/net/dev
```

每个网络接口一行统计数据，第 4 列（errs）是接收出错的数据包数量，第 5 列（drop）是接收不过来丢弃的数量。

第二部分是传统非 NAPI 接口实现的网卡驱动，每个 CPU 有一个队列，当在队列中缓存的数据包数量超过 net.core.netdev_max_backlog 时，网卡驱动程序会丢掉数据包，这个见下面的文件：

```
$ cat /proc/net/softnet_stat
```

每个 CPU 有一行统计数据，第二列是对应 CPU 丢弃的数据包数量。

### 应用程序处理不过来，操作系统丢弃

内核中记录了两个计数器：

* ListenOverflows：当 socket 的 listen queue 已满，当新增一个连接请求时，应用程序来不及处理；

* ListenDrops：包含上面的情况，除此之外，当内存不够无法为新的连接分配 socket 相关的数据结构时，也会加 1，当然还有别的异常情况下会增加 1。

分别对应下面文件中的第 21 列（ListenOverflows）和第 22 列（ListenDrops），可以通过如下命令查看：

```
$ cat /proc/net/netstat | awk '/TcpExt/ { print $21,$22 }'
```

如果使用 netstat 命令，有丢包时会看到 “times the listen queue of a socket overflowed” 以及 “SYNs to LISTEN sockets ignored” 对应行前面的数字；如果值为 0 则不会输出对应的行。

### Out of memory

如上所示，出现内存不足可能会有两种情况：

* 有太多的 orphan sockets，通常对于一些前端的服务器经常会出现这种情况。

* 分配给 TCP 的内存确实较少，从而导致内存不足。

#### 内存不足

这个比较好排查，只需要查看一下实际分配给 TCP 多少内存，现在时用了多少内存即可。需要注意的是，通常的配置项使用的单位是 Bytes，在此用的是 Pages，通常为 4K 。

先查看下给 TCP 分配了多少内存。

```
$ cat /proc/net/sockstat
sockets: used 855
TCP: inuse 17 orphan 1 tw 0 alloc 19 mem 3
UDP: inuse 16 mem 10
UDPLITE: inuse 0
RAW: inuse 1
FRAG: inuse 0 memory 0
```

其中的 mem 表示使用了多少 Pages，如果相比 tcp_mem 的配置来说还很小，那么就有可能是由于 orphan sockets 导致的。

#### orphan sockets

首先介绍一下什么是 orphan sockets，简单来说就是该 socket 不与任何一个文件描述符相关联。例如，当应用调用 close() 关闭一个链接时，此时该 socket 就成为了 orphan，但是该 sock 仍然会保留一段时间，直到最后根据 TCP 协议结束。

实际上 orphan socket 对于应用来说是无用的，因此内核希望尽可能减小 orphan 的数量。对于像 http 这样的短请求来说，出现 orphan 的概率会比较大。

对于系统允许的最大 orphan 数量，以及当前的 orphan 数量可以通过如下方式查看：

```
$ cat /proc/sys/net/ipv4/tcp_max_orphans
32768
$ cat /proc/net/sockstat
... ...
TCP: inuse 37 orphan 14 tw 8 alloc 39 mem 9
... ...
```

你可能会发现，sockstat 中的 orphan 数量要远小于 tcp_max_orphans 的数目。

实际上，可以从代码中看到，实际会有个偏移量 shift，该值范围为 [0, 2] 。

```
static inline bool tcp_too_many_orphans(struct sock *sk, int shift)
{
    struct percpu_counter *ocp = sk->sk_prot->orphan_count;
    int orphans = percpu_counter_read_positive(ocp);

    if (orphans << shift > sysctl_tcp_max_orphans) {
        orphans = percpu_counter_sum_positive(ocp);
        if (orphans << shift > sysctl_tcp_max_orphans)
            return true;
    }
    return false;
}
```

也就是说，在某些场景下会对 orphan 做些惩罚，将 orphan 的数量 2x 甚至 4x，这也就解释了上述的问题。

如果是这样，那么就可以根据具体的情况，将 tcp_max_orphans 值适当调大。

### Procfs 文件系统

* /proc/net/tcp：记录 TCP 的状态信息。

# TCP 状态排查

可以通过如下命令查看 TCP 状态。

----- 查看链接状态，并对其进行统计，如下的两种方法相同

```
$ netstat -atn | awk '/^tcp/ {++s[$NF]} END {for(key in s) print s[key], "\t", key}' | sort -nr
$ ss -ant | awk ' {++s[$1]} END {for(key in s) print s[key], "\t", key}' | sort -nr
```

----- 查找较多time_wait连接

```
$ netstat -n|grep TIME_WAIT|awk '{print $5}'|sort|uniq -c|sort -rn|head -n20
```

----- 对接的IP进行排序

```
$ netstat -ntu | awk '/^tcp/ {print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
```

----- 查看80端口连接数最多的20个IP

```
$ netstat -ant |awk '/:80/{split($5,ip,":");++A[ip[1]]}END{for(i in A) print A[i],i}' |sort -rn|head -n20
```

----- 80端口的各个TCP链接状态

```
$ netstat -n | grep `hostname -i`:80 |awk '/^tcp/{++S[$NF]}END{for (key in S) print key,S[key]}'
```
