# Network Card

## /proc/net/dev

在Linux系統中，系統調用是作業系統提供給應用程式使用作業系統服務的重要介面，但同時也正是通過系統調用機制，作業系統遮罩了使用者直接訪問系統內核的可能性。

幸運的是Linux提供了LKM機制可以使我們在內核空間工作，在LKM機制中一個重要的組成部分就是proc偽檔案系統，它為使用者提供了動態操作Linux內核資訊的介面，是除系統調用之外另一個重要的Linux內核空間與使用者空間交換資料的途徑。

```
cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
eno16777736: 1150470   14704    0    0    0     0          0         0   720003    7105    0    0    0     0       0          0
         lo: 1720952079 1301029    0    0    0     0          0         0 1720952079 1301029    0    0    0     0       0          0
eno33554960:  300255    3030    0    0    0     0          0         0     8142      44    0    0    0     0       0          0
```

根據proc/net/dev中每一項的含義是：

Receive :

* bytes: The total number of bytes of data transmitted or received by the interface.（介面發送或接收的資料的總位元組數）
* packets: The total number of packets of data transmitted or received by the interface.（介面發送或接收的資料包總數）
* errs: The total number of transmit or receive errors detected by the device driver.（由設備驅動程式檢測到的發送或接收錯誤的總數）
* drop: The total number of packets dropped by the device driver.（設備驅動程式丟棄的資料包總數）
* fifo: The number of FIFO buffer errors.（FIFO緩衝區錯誤的數量）
* frame: The number of packet framing errors.（分組幀錯誤的數量）
* colls: The number of collisions detected on the interface.（介面上檢測到的衝突數）
* compressed: The number of compressed packets transmitted or received by the device driver. (This appears to be unused in the 2.2.15 kernel.)（設備驅動程式發送或接收的壓縮資料包數）
* carrier: The number of carrier losses detected by the device driver.（由設備驅動程式檢測到的載波損耗的數量）
* multicast: The number of multicast frames transmitted or received by the device driver.（設備驅動程式發送或接收的多播幀數）

Transmit:
* bytes:
* packets: 
* errs: 
* drop: 
* fifo: 
* colls: 
* carrier: 
* compressed: 

http://www.onlamp.com/pub/a/linux/2000/11/16/LinuxAdmin.html

## /sys/class/net/*amp;/statistics/*amp;

這些指標是網卡的統計資料，可以用 ethtool -S ethx 看到。
/sys/class/net/*/statistics/*。

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
