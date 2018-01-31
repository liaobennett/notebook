
指標參考下面幾個內容整理。

* basic-perfomance
* tsar https://github.com/alibaba/tsar/blob/master/info.md#io
* netflixs http://techblog.netflix.com/2015/04/introducing-vector-netflixs-on-host.html
* openfalcon http://book.open-falcon.com/zh/faq/linux-metrics.html
含義參考對應的 manpage, kernel doc。

### I/O
資料都在 /proc/diskstats裡面。我們平時都看 iostat -x 1 的輸出。先看下它的輸出的含義

* rrqm/s: 每秒加到讀佇列的讀運算元量。
* wrqm/s: 每秒加到寫佇列的讀運算元量。
* r/s: 每秒完成讀操作的次數。
* w/s: 每秒完成寫操作的次數。
上面說的操作都是 merge 之後的，相鄰的讀/寫可能會合並。

rrqm是系統合併後的值， r 是真正落到磁片的上請求數量。

* rsec/s: 每秒讀的磁區數。
* wsec/s: 每秒寫的磁區數。
* rkB/s:  每秒讀K位元組數.是 rsect/s 的一半,因為每磁區大小為512位元組。
* wkB/s:  每秒讀K位元組數.是 rsect/s 的一半,因為每磁區大小為512位元組。
* avgrq-sz：平均每次I/O操作的資料大小 (磁區)。
* avgqu-sz：平均I/O佇列長度。.
* await: 平均每次操作的等待時間。 r_await 和 w_await 解釋和這個一模一樣。
* svctm: manpage 說這個指標要廢棄了。
* util: 理解成採樣時間內有多少時間佇列是非空的。這個值太高說明磁片存在瓶頸。
這些指標都可以採集回來。

平時我們重點關注 r/s w/s 和 util 就夠了。 前兩個反映讀寫的 iops，最後一個反映磁片的整體負載情況。

參考：
* http://linux.die.net/man/1/iostat
* https://github.com/alibaba/tsar/blob/master/info.md#io
