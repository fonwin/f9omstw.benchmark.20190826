f9omstw benchmark
=================

[新版測試結果](https://github.com/fonwin/f9omstw.benchmark)

20190828 補充說明
* 這次的測試犯了一個錯誤:
  * CPU list: "10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29"
  * 應改成: "10,11,12,13,14,15,16,17,18,19,30,31,32,33,34,35,36,37,38,39"
  * 因為 0..9, 20..29 的 "physical id : 0"
  * 因為 10..19, 30..39 的 "physical id : 1"
  * 請參考 [cpuinfo](cpuinfo.txt)

## 基本說明
* sourc code
  * [libfon9](https://github.com/fonwin/libfon9) (版本: 3408e02)
  * [f9omstw](https://github.com/fonwin/f9omstw) (版本: c64f2d8)

* 測量的時間點:
  * [T0 Client before send](https://github.com/fonwin/f9omstw/blob/c64f2d8/f9omsrc/OmsRcClient_UT.c#L285)
  * [T1 Server before parse(封包收到時間)](https://github.com/fonwin/f9omstw/blob/c64f2d8/f9omsrc/OmsRcServerFunc.cpp#L194)
  * [T2 Server after first updated(Sending, Queuing, Reject...)](https://github.com/fonwin/f9omstw/blob/c64f2d8/f9omstw/OmsBackend.cpp#L202)
  * `T0-` = T0(n) - T0(n-1) 表示 Client 連續送單之間的延遲.
  * `T1-` = T1(n) - T1(n-1) 表示 RcServer 處理連續下單之間的延遲.
  * `T2-` = T2(n) - T2(n-1) 表示 OmsCore 處理連續下單要求之間的延遲 (包含送出 FIX.4.4 給證交所).
  * 有測試交易所成功回覆, 但沒有計算交易所回覆的時間

* OmsCore 使用 OmsCoreByThread
  * 收單 thread 與 OmsCore 在不同 thread.
  * 目前 WaitPolicy 固定為 Block(使用 condition variable).
  * 尚未針對 latency 最佳化, 例如: 綁定 CPU, 設定 WaitPolicy.
  * 甚至不使用 OmsCoreByThread 改用 mutex?

---------------------------------------
## [初次啟動](Startup.md)

## 測試環境
* HP ProLiant DL380p Gen8 / E5-2680 v2 @ 2.80GHz
* Ubuntu 16.04.2 LTS
* gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9)

## 測試指令
* Client 下單程式 & 下單指令:
```
~/devel/output/f9omstw/release/f9omsrc/OmsRcClient_UT -a "dn=localhost:6601|ClosedReopen=5" -u fonwin
> set TwsNew BrkId=8610|Market=T|Symbol=2317|OType=0|Pri=200|Qty=8000|Side=B|IvacNo=10
> send TwsNew 1 g1
> send TwsNew 10 g2
> send TwsNew 100 g3
> send TwsNew 1000 g4
> send TwsNew 10000 g5
> send TwsNew 100000 g6
```
* 分析程式
`~/devel/output/f9omstw/release/f9omstw/OmsLogAnalyser ~/f9utws/logs/20190826/omstws.log `

---------------------------------------
## [測試結果](Result.txt)
* 時間單位: us(microsecond)
* [詳細 log](logs/omstws.log.zip)

* [g1: 1 筆](logs/omstws.log.TwsNew.g1.txt)
  * T1-T0|49|
  * T2-T1|48|
* [g2: 10 筆](logs/omstws.log.TwsNew.g2.txt)
  * T1-T0|50%=79|75%=85|90%=91|99%=91|99.9%=91|99.99%=91|Worst=85|86|91|
  * T2-T1|50%=101|75%=119|90%=123|99%=123|99.9%=123|99.99%=123|Worst=119|122|123|
  * T0-|50%=4|75%=6|90%=18|99%=18|99.9%=18|99.99%=18|Worst=6|8|18|
  * T1-|50%=1|75%=1|90%=24|99%=24|99.9%=24|99.99%=24|Worst=1|1|24|
  * T2-|50%=2|75%=3|90%=19|99%=19|99.9%=19|99.99%=19|Worst=3|4|19|
* [g3: 100 筆](logs/omstws.log.TwsNew.g3.txt)
  * T1-T0|50%=21|75%=38|90%=80|99%=118|99.9%=118|99.99%=118|Worst=101|105|118|
  * T2-T1|50%=27|75%=110|90%=127|99%=139|99.9%=139|99.99%=139|Worst=132|134|139|
  * T0-|50%=6|75%=9|90%=13|99%=31|99.9%=31|99.99%=31|Worst=21|30|31|
  * T1-|50%=3|75%=12|90%=18|99%=37|99.9%=37|99.99%=37|Worst=30|30|37|
  * T2-|50%=2|75%=9|90%=18|99%=29|99.9%=29|99.99%=29|Worst=24|24|29|
* [g4: 1000 筆](logs/omstws.log.TwsNew.g4.txt)
  * T1-T0|50%=16|75%=20|90%=24|99%=56|99.9%=100|99.99%=100|Worst=82|87|100|
  * T2-T1|50%=9|75%=9|90%=14|99%=124|99.9%=130|99.99%=130|Worst=128|128|130|
  * T0-|50%=8|75%=11|90%=13|99%=26|99.9%=41|99.99%=41|Worst=30|32|41|
  * T1-|50%=8|75%=15|90%=18|99%=29|99.9%=45|99.99%=45|Worst=37|39|45|
  * T2-|50%=7|75%=15|90%=19|99%=30|99.9%=46|99.99%=46|Worst=37|40|46|
* [g5: 1萬筆](logs/omstws.log.TwsNew.g5.txt)
  * T1-T0|50%=16|75%=20|90%=23|99%=34|99.9%=81|99.99%=325|Worst=113|323|325|
  * T2-T1|50%=8|75%=9|90%=10|99%=19|99.9%=100|99.99%=108|Worst=105|106|108|
  * T0-|50%=9|75%=11|90%=13|99%=25|99.9%=30|99.99%=315|Worst=33|36|315|
  * T1-|50%=9|75%=15|90%=17|99%=28|99.9%=34|99.99%=320|Worst=41|41|320|
  * T2-|50%=9|75%=15|90%=18|99%=28|99.9%=34|99.99%=322|Worst=39|43|322|
* [g6: 10萬筆](logs/omstws.log.TwsNew.g6.txt)
  * T1-T0|50%=16|75%=19|90%=23|99%=32|99.9%=42|99.99%=335|Worst=392|399|407|
  * T2-T1|50%=8|75%=10|90%=12|99%=802|99.9%=1977|99.99%=2651|Worst=2729|2730|2740|
  * T0-|50%=9|75%=11|90%=13|99%=24|99.9%=29|99.99%=37|Worst=319|321|324|
  * T1-|50%=8|75%=15|90%=17|99%=27|99.9%=34|99.99%=48|Worst=329|330|401|
  * T2-|50%=7|75%=15|90%=18|99%=27|99.9%=36|99.99%=454|Worst=1424|1623|2744|
* 結果 log 裡面的群組: x1..x6 為使用 sudo 執行, 讓 $MemLock=Y 生效.
  * 但從結果來看, 為何沒有比較快?
  * 20190828: CPU list 設定有錯造成的?

---------------------------------------
## 測試結果探討
* 詳細的探討, 等有空時再說吧!
* 10萬筆, 可在 1 秒內完成: 收單解析、下單打包(FIX.4.4)、部分 send() 成功
  * 但因網路頻寬、SNDBUF 配置... 等因素, 下單打包的資料, 可能仍保留在程式的緩衝區裡面.
  * 以 g6:100000 最後這筆來看
    * T0 = 20190826031422.828570
    * T1 = 20190826031422.828587 (T1 - T0 = 17 us)
    * T2 = 20190826031422.828605 (T2 - T1 = 18 us)
    * OMS 打包完畢的時間 = T2
    * OMS 收到回報的時間 = 20190826031427.127363 ( - T2 = 花費 4.298758 秒)
    * 測試環境 OMS 到「模擬交易所」之間的網路頻寬 = 100M
* OmsCoreByThread(Context switch): `T2-T1`
* OmsCore 的效率: `T2-`
  * 包含 send() 呼叫
* Rc協定收單 的效率: `T1-`
* CPU cache 的影響?
* 記憶體用量?

### Linux 的 低延遲
* 這裡只是基礎, 核心的調教, 還要更多的研究
* Linux 啟動時使用 isolate 參數, 將 cpu core 保留給 OMS 使用.
  * sudo vi /etc/default/grub
    * GRUB_CMDLINE_LINUX_DEFAULT="isolcpus=10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29"
  * sudo update-grub
* Linux 設定 irqbalance
* 將 CPU 設定為高效能(關閉省電模式), 但若 IO 設定有使用 `Wait=Busy` 參數, 也可以考慮不調整 CPU 時脈.
```
#!/bin/bash

service irqbalance stop

# 將 CPU 時脈調到最高.
max_freq=$(for i in $(cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies); do echo $i; done | sort -nr | head -1)
for cpu_max_freq in /sys/devices/system/cpu/cpu*/cpufreq/scaling_max_freq; do echo $max_freq > $cpu_max_freq; done
for governor in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do echo performance > $governor; done

# echo -1 > /proc/sys/kernel/sched_rt_runtime_us
```
* 啟動時的優先權及綁定 cpu
  * 使用 chrt 設定優先權
  * 使用 taskset 設定使用的 cpu cores
    * `chrt -f 90 taskset -c 10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29 ./run.sh`
    * 若有錯誤訊息: `chrt: failed to set pid 0's policy: Operation not permitted`   
      則要先執行:   `sudo setcap  cap_sys_nice=eip  /usr/bin/chrt`

* 停用 irqbalance, 未設定 CPU 時脈.
```
processor   : 13
vendor_id   : GenuineIntel
cpu family  : 6
model       : 62
model name  : Intel(R) Xeon(R) CPU E5-2680 v2 @ 2.80GHz
stepping    : 4
microcode   : 0x428
cpu MHz     : 3568.031        因使用 Wait=Busy 所以此 core 會進入 turbo mode.
cache size  : 25600 KB
```
