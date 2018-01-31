# 網路
## netstat -st輸出的兩個重要資訊來源分別是/proc/net/snmp和/proc/net/netstat

## 從 /proc/net/snmp 可以拿到這些指標：

### tcp

* ActiveOpens:主動打開的tcp連接數量。
* PassiveOpens:被動打開的tcp連接數量。
* InSegs: 收到的tcp報文數量。
* OutSegs:發出的tcp報文數量。
* EstabResets: established 中發生的 reset。
* AttemptFails: 連接失敗的數量。
* CurrEstab:當前狀態為ESTABLISHED的tcp連接數。
* RetransSegs: 重傳的報文數量。
* retran:系統的重傳率 (RetransSegs－last RetransSegs) ／ (OutSegs－last OutSegs) * 100%

### udp

* InDatagrams
* OutDatagrams
* NoPorts: 目的地址或者埠不存在。
* InErrors： 無效數據包。
* RcvbufErrors：內核的 buffer 滿了導致的接收失敗。
* SndbufErrors：同上。
* InCsumErrors：checksum 錯誤的 udp 包數量。

還可以用 ss 把這些 tcp 連結的資訊採集起來：

* ss.orphaned
* ss.closed
* ss.timewait
* ss.slabinfo.timewait
* ss.synrecv
* ss.estab

collectd 預設會把每個 TCP 狀態的連接數都採集下來。

這些指標可能不需要報警。但是收集起來用來排查問題、查看網路負載很有用。

### 網卡
這些指標是網卡的統計資料，可以用 ethtool -S ethx 看到。資料來源來自：/proc/net/dev和 /sys/class/net/*/statistics/*。

* net.if.in.bytes : 收到的資料的總 bytes.

* net.if.in.dropped: 進入了 ring buffer， 在拷貝到記憶體的時候被丟了。

* net.if.in.errors: 收到的錯誤包的總數。錯誤包包括：crc 校驗錯誤、幀同步錯誤、ring buffer溢出等。

* net.if.in.fifo.errs: 這個是 ifconfig 裡看到的 overruns.表示資料包沒到 ring buffer 就被丟了。也就是 cpu 來不及處理 ringbuffer 裡的資料，通常在網卡壓力大、沒有做affinity的時候會發生。

* net.if.in.frame.errs: misaligned frames. frame 的長度（bit）不能被 8 整除。

* net.if.in.multicast: 組播。

* net.if.in.packets: 收到的 packets 數量統計。

* net.if.out.bytes

* net.if.out.carrier.errs: 這個意味著實體層出問題了。比如網卡的工作模式不對。

* net.if.out.collisions： 因為 CSMA/CD 造成的傳輸錯誤。

* net.if.out.dropped

* net.if.out.errors

* net.if.out.fifo.errs

* net.if.out.packets

* net.if.total.bytes

* net.if.total.dropped

網卡的工作模式。全雙工千兆/萬兆。

網路壓力大的服務，是有必要做中斷的 affinity 和提高 ring buffer 值的。

## netstat參數和使用

常用參數-anplt

-a 顯示所有活動的連接以及本機偵聽的TCP、UDP埠

-l 顯示監聽的server port

-n 直接使用IP位址，不通過功能變數名稱伺服器

-p 正在使用Socket的程式PID和程式名稱

-r 顯示路由表

-t 顯示TCP傳輸協定的連線狀況

-u 顯示UDP傳輸協定的連線狀況

-w 顯示RAW傳輸協定的連線狀況

在Linux下，raw格式的資料通常可以通過/proc/net/dev獲得。在Windows平臺，netstat資訊可以通過IP Helper API的GetTcpTable和GetUdpTable函數獲得。

## ss（socket statistics）參數和使用

常用參數和netstat類似，如-anp

-a顯示所有的sockets

-l顯示正在監聽的

-n顯示數位IP和埠，不通過功能變數名稱伺服器

-p顯示使用socket的對應的程式

-t只顯示TCP sockets

-u只顯示UDP sockets

-4 -6 只顯示v4或v6V版本的sockets

-s列印出統計資訊。這個選項不解析從各種源獲得的socket。對於解析/proc/net/top大量的sockets計數時很有效

-0 顯示PACKET sockets

-w 只顯示RAW sockets

-x只顯示UNIX域sockets

-r嘗試進行功能變數名稱解析，位址/埠

ss比netstat快的主要原因是，netstat是遍歷/proc下面每個PID目錄，ss直接讀/proc/net下面的統計資訊。所以ss執行的時候消耗資源以及消耗的時間都比netstat少很多。

當伺服器的socket連接數量非常大時（如上萬個），無論是使用netstat命令還是直接cat /proc/net/tcp執行速度都會很慢，相比之下ss可以節省很多時間。

ss快的秘訣在於，它利用了TCP協議棧中tcp_diag，這是一個用於分析統計的模組，可以獲得Linux內核中的第一手資訊。如果系統中沒有tcp_diag，ss也可以正常運行，只是效率會變得稍微慢但仍然比netstat要快。

### netstat屬於net-tools工具集，ss屬於ipoute工具集。替換方案如下：

| 用途  | net-tool  | iproute2 |
| :------------ |:---------------:| -----:|
| 位址與鏈路配置 | ifconfig | ip addr,ip link |
| 路由表 | route | ip route  |
| 鄰居 | arp | ip neigh |
| VLAN | vconfig | ip link |
| 隧道 | iptunnel | ip tunnel |
| 組播 | ipmaddr | ip maddr |
| 統計 | netstat | ss |

## Linux 網路常見監控項以及報錯
### 查看丟包
網路丟包會有多種可能，例如，交換機，上連和下連埠的流量跑滿或鏈路有問題，那麼資料包就有可能會被交換機丟掉；負載均衡設備，包括了硬體設備以及軟體的負載均衡。在此，我們僅查看本機可能導致的掉包。

### 作業系統處理不過來，丟棄資料
有兩種情況，一是網卡發現作業系統處理不過來，丟資料包，可以讀取下面的檔：

```
$ cat /proc/net/dev
```

每個網路介面一行統計資料，第 4 列（errs）是接收出錯的資料包數量，第 5 列（drop）是接收不過來丟棄的數量。

第二部分是傳統非 NAPI 介面實現的網卡驅動，每個 CPU 有一個佇列，當在佇列中緩存的資料包數量超過 net.core.netdev_max_backlog 時，網路卡驅動程式會丟掉資料包，這個見下面的檔：

```
$ cat /proc/net/softnet_stat
```

每個 CPU 有一行統計資料，第二列是對應 CPU 丟棄的資料包數量。

### 應用程式處理不過來，作業系統丟棄

內核中記錄了兩個計數器：

* ListenOverflows：當 socket 的 listen queue 已滿，當新增一個連接請求時，應用程式來不及處理；

* ListenDrops：包含上面的情況，除此之外，當記憶體不夠無法為新的連接分配 socket 相關的資料結構時，也會加 1，當然還有別的異常情況下會增加 1。

分別對應下面文件中的第 21 列（ListenOverflows）和第 22 列（ListenDrops），可以通過如下命令查看：

```
$ cat /proc/net/netstat | awk '/TcpExt/ { print $21,$22 }'
```

如果使用 netstat 命令，有丟包時會看到 “times the listen queue of a socket overflowed” 以及 “SYNs to LISTEN sockets ignored” 對應行前面的數位；如果值為 0 則不會輸出對應的行。

### Out of memory

如上所示，出現記憶體不足可能會有兩種情況：

* 有太多的 orphan sockets，通常對於一些前端的伺服器經常會出現這種情況。

* 分配給 TCP 的記憶體確實較少，從而導致記憶體不足。

#### 記憶體不足

這個比較好排查，只需要查看一下實際分配給 TCP 多少記憶體，現在時用了多少記憶體即可。需要注意的是，通常的配置項使用的單位是 Bytes，在此用的是 Pages，通常為 4K 。

先查看下給 TCP 分配了多少記憶體。

```
$ cat /proc/net/sockstat
sockets: used 855
TCP: inuse 17 orphan 1 tw 0 alloc 19 mem 3
UDP: inuse 16 mem 10
UDPLITE: inuse 0
RAW: inuse 1
FRAG: inuse 0 memory 0
```

其中的 mem 表示使用了多少 Pages，如果相比 tcp_mem 的配置來說還很小，那麼就有可能是由於 orphan sockets 導致的。

#### orphan sockets

首先介紹一下什麼是 orphan sockets，簡單來說就是該 socket 不與任何一個檔描述符相關聯。例如，當應用調用 close() 關閉一個連結時，此時該 socket 就成為了 orphan，但是該 sock 仍然會保留一段時間，直到最後根據 TCP 協定結束。

實際上 orphan socket 對於應用來說是無用的，因此內核希望盡可能減小 orphan 的數量。對於像 http 這樣的短請求來說，出現 orphan 的概率會比較大。

對於系統允許的最大 orphan 數量，以及當前的 orphan 數量可以通過如下方式查看：

```
$ cat /proc/sys/net/ipv4/tcp_max_orphans
32768
$ cat /proc/net/sockstat
... ...
TCP: inuse 37 orphan 14 tw 8 alloc 39 mem 9
... ...
```

你可能會發現，sockstat 中的 orphan 數量要遠小於 tcp_max_orphans 的數目。

實際上，可以從代碼中看到，實際會有個偏移量 shift，該值範圍為 [0, 2] 。

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

也就是說，在某些場景下會對 orphan 做些懲罰，將 orphan 的數量 2x 甚至 4x，這也就解釋了上述的問題。

如果是這樣，那麼就可以根據具體的情況，將 tcp_max_orphans 值適當調大。

### sysctl
有些 sysctl 的配置項的值也需要關心下：

* net.netfilter.nf_conntrack_max
* net.netfilter.nf_conntrack_count
* fs.file-max 、fs.file-nr 這兩個參數可能更需要關注單個進程內的值。
還有一些我們自己調整過的、對業務會有影響的項。比如 ip_forward.

### Procfs 檔案系統

* /proc/net/tcp：記錄 TCP 的狀態資訊。

# TCP 狀態排查

可以通過如下命令查看 TCP 狀態。

----- 查看連結狀態，並對其進行統計，如下的兩種方法相同

```
$ netstat -atn | awk '/^tcp/ {++s[$NF]} END {for(key in s) print s[key], "\t", key}' | sort -nr
$ ss -ant | awk ' {++s[$1]} END {for(key in s) print s[key], "\t", key}' | sort -nr
```

----- 查找較多time_wait連接

```
$ netstat -n|grep TIME_WAIT|awk '{print $5}'|sort|uniq -c|sort -rn|head -n20
```

----- 對接的IP進行排序

```
$ netstat -ntu | awk '/^tcp/ {print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
```

----- 查看80埠連接數最多的20個IP

```
$ netstat -ant |awk '/:80/{split($5,ip,":");++A[ip[1]]}END{for(i in A) print A[i],i}' |sort -rn|head -n20
```

----- 80埠的各個TCP連結狀態

```
$ netstat -n | grep `hostname -i`:80 |awk '/^tcp/{++S[$NF]}END{for (key in S) print key,S[key]}'
```

# 硬體層面
* 電源健康狀態。是不是兩路電都在。

* 硬碟的健康狀態。 分區是否可寫。

* raid 卡的健康狀態。

* 遠控卡是不是可用。

* 遠控卡中的異常日誌。

* 網卡的工作狀態是否正常。
