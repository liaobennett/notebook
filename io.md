# 
### I/O
数据都在 /proc/diskstats里面。我们平时都看 iostat -x 1 的输出。先看下它的输出的含义(翻译自 manpage)：

* rrqm/s: 每秒加到读队列的�读操作数量。
* wrqm/s: 每秒加到写队列的�读操作数量。
* r/s: 每秒完成读操作的次数。
* w/s: 每秒完成写操作的次数。
上面说的操作都是 merge 之后的，相邻的读/写可能会合并。

rrqm是系统合并后的值， r 是真正落到磁盘的上请求数量。

* rsec/s: 每秒读的扇区数。
* wsec/s: �每秒写的扇区数。
* rkB/s:  每秒读K字节数.是 rsect/s 的一半,因为每扇区大小为512字节。
* wkB/s:  每秒读K字节数.是 rsect/s 的一半,因为每扇区大小为512字节。
* avgrq-sz：平均每次I/O操作的数据大小 (扇区)。
* avgqu-sz：平均I/O队列长度。.
* await: 平均每次操作的等待时间。 r_await 和 w_await 解释和这个一模一样。
* svctm: manpage 说这个指标要废弃了。
* util: 理解成采样时间内有多少时间队列是非空的。这个值太高说明磁盘存在瓶颈。
这些指标都可以采集回来。

平时我们重点关注 r/s w/s 和 util 就够了。 前两个反映读写的 iops，最后一个反映磁盘的整体负载情况。

参考：

* http://linux.die.net/man/1/iostat
* https://github.com/alibaba/tsar/blob/master/info.md#io
