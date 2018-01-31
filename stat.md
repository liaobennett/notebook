# stat

# 

```
cpu  2141528 0 556623 548389457 406368 115 10325 116432 0 0
cpu0 2141528 0 556623 548389457 406368 115 10325 116432 0 0
intr 551323037 19 9 0 0 2 0 2 0 2 0 0 0 144 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 474757771 0 0 0 0 510 0 4150701 72394413 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 ....(好多〇)
ctxt 1870344087
btime 1447738559
processes 2930163
procs_running 1
procs_blocked 0
softirq 224417040 0 77979395 36732867 41760790 0 0 133963 0 81548 67728477
```

* cpu: 单位是jiffies[^1]。挨个是：
** user: normal processes executing in user mode。这个值应该越高越好。
** nice: niced processes executing in user mode。通常我们不会有。
** system: processes executing in kernel mode。一般的不希望这个值太高。
** idle: twiddling thumbs
** iowait: waiting for I/O to complete
** irq: servicing interrupts
** softirq: servicing softirqs
** steal: involuntary wait. 被强制等待虚拟CPU的时间,此时hypervisor在为另一个虚拟处理器服务。这个值高了表示物理机超卖了。
** guest: running a normal guest, 运行guest os 虚拟 cpu 所用。
** guest_nice: running a niced guest
intr 是总的中断的统计。
* ctxt 所有 cpu 上的 context switches 总数。
* btime system boot time, 单位 seconds, 从 unix epoch 开始算。
* processes 表示创建过的processes and threads， 包括但不限于 fork() clone()创建的。

所以在 cpu 需要关注的指标有 ``` cpu.user ``` , cpu.sys, cpu.idel, cpu.iowait, cpu.irq, cpu.softirq, cpu.util, cpu.switches ```.

可以这样算 cpu.util: 1 - idle - iowait - steal

[^1]: jiffies 的大小在  <linux/jiffies.h> 里定义。x86平台上是 1/100s。

## procfs

```
/proc/stat
```
