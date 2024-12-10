---
title: java gc 横向对比
date: 2024-12-05 10:12:47
tags: ['java']
category: java
article: java gc 横向对比
---

# java

### GC

分代假设：大部分新生对象很快无用，存活较长时间的对象，可能存活更长时间。经历了15次GC还存在的就放在老年代.新生代80%，S0,S1各10%新生代到存活区是复制，到老年代是移动。

-XX: +MaxTenuringThreshold=15 表示15次后放到老年代。

可以做为GC ROOT的对象
1. 当前正在执行的方法里的局部变量和输入参数
2. 活动线程
3. 所有类的静态字段
4. JNI引用

#### Serial GC/ParNewGC

-XX: +UseSerialGC开启

对年轻代使用mark-copy算法，对老年代使用mark-sweep-compact算法

串行GC不能并行处理，所以触发全部暂停(STW)

ParNewGC可以配合CMSGC使用

适用场景：
- 单线程应用或资源有限的环境（如嵌入式系统）。
- 小型应用，不需要频繁的垃圾回收。

java版本19，测试GC效率

命令
```
java -XX:+UseSerialGC -Xms128m -Xmx128m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

效果。可以看到当分配128m内存的时候，频繁触发GC,最终还是OOM，说明堆大小设置的太小了。

```
[0.003s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.009s][info   ][gc,init] CardTable entry size: 512
[0.009s][info   ][gc     ] Using Serial
[0.009s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.009s][info   ][gc,init] CPUs: 8 total, 8 available
[0.009s][info   ][gc,init] Memory: 16384M
[0.009s][info   ][gc,init] Large Page Support: Disabled
[0.009s][info   ][gc,init] NUMA Support: Disabled
[0.009s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.009s][info   ][gc,init] Heap Min Capacity: 128M
[0.009s][info   ][gc,init] Heap Initial Capacity: 128M
[0.009s][info   ][gc,init] Heap Max Capacity: 128M
[0.009s][info   ][gc,init] Pre-touch: Disabled
[0.009s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.009s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.009s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.042s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.044s][info   ][gc,heap     ] GC(0) DefNew: 34915K(39296K)->4352K(39296K) Eden: 34915K(34944K)->0K(34944K) From: 0K(4352K)->4352K(4352K)
[0.045s][info   ][gc,heap     ] GC(0) Tenured: 992K(87424K)->11701K(87424K)
[0.045s][info   ][gc,metaspace] GC(0) Metaspace: 141K(384K)->141K(384K) NonClass: 135K(256K)->135K(256K) Class: 5K(128K)->5K(128K)
[0.045s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 35M->15M(123M) 2.838ms
[0.045s][info   ][gc,cpu      ] GC(0) User=0.00s Sys=0.00s Real=0.00s
[0.048s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.052s][info   ][gc,heap     ] GC(1) DefNew: 39296K(39296K)->4347K(39296K) Eden: 34944K(34944K)->0K(34944K) From: 4352K(4352K)->4347K(4352K)
[0.052s][info   ][gc,heap     ] GC(1) Tenured: 11701K(87424K)->25898K(87424K)
[0.052s][info   ][gc,metaspace] GC(1) Metaspace: 145K(384K)->145K(384K) NonClass: 139K(256K)->139K(256K) Class: 5K(128K)->5K(128K)
[0.052s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 49M->29M(123M) 3.616ms
[0.052s][info   ][gc,cpu      ] GC(1) User=0.00s Sys=0.00s Real=0.01s
[0.056s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.060s][info   ][gc,heap     ] GC(2) DefNew: 39098K(39296K)->4349K(39296K) Eden: 34751K(34944K)->0K(34944K) From: 4347K(4352K)->4349K(4352K)
[0.060s][info   ][gc,heap     ] GC(2) Tenured: 25898K(87424K)->42379K(87424K)
[0.060s][info   ][gc,metaspace] GC(2) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.061s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 63M->45M(123M) 4.161ms
[0.061s][info   ][gc,cpu      ] GC(2) User=0.01s Sys=0.00s Real=0.01s
[0.067s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.069s][info   ][gc,heap     ] GC(3) DefNew: 39069K(39296K)->4349K(39296K) Eden: 34720K(34944K)->0K(34944K) From: 4349K(4352K)->4349K(4352K)
[0.069s][info   ][gc,heap     ] GC(3) Tenured: 42379K(87424K)->53353K(87424K)
[0.069s][info   ][gc,metaspace] GC(3) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.069s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 79M->56M(123M) 2.338ms
[0.069s][info   ][gc,cpu      ] GC(3) User=0.00s Sys=0.00s Real=0.00s
[0.074s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.076s][info   ][gc,heap     ] GC(4) DefNew: 39137K(39296K)->4349K(39296K) Eden: 34787K(34944K)->0K(34944K) From: 4349K(4352K)->4349K(4352K)
[0.076s][info   ][gc,heap     ] GC(4) Tenured: 53353K(87424K)->65261K(87424K)
[0.076s][info   ][gc,metaspace] GC(4) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.076s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 90M->67M(123M) 2.158ms
[0.076s][info   ][gc,cpu      ] GC(4) User=0.00s Sys=0.01s Real=0.00s
[0.079s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.081s][info   ][gc,heap     ] GC(5) DefNew: 39247K(39296K)->4349K(39296K) Eden: 34898K(34944K)->0K(34944K) From: 4349K(4352K)->4349K(4352K)
[0.081s][info   ][gc,heap     ] GC(5) Tenured: 65261K(87424K)->81029K(87424K)
[0.081s][info   ][gc,metaspace] GC(5) Metaspace: 155K(384K)->155K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.081s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 102M->83M(123M) 2.255ms
[0.081s][info   ][gc,cpu      ] GC(5) User=0.00s Sys=0.00s Real=0.01s
[0.085s][info   ][gc,start    ] GC(6) Pause Young (Allocation Failure)
[0.085s][info   ][gc          ] GC(6) Pause Young (Allocation Failure) 117M->117M(123M) 0.040ms
[0.085s][info   ][gc,cpu      ] GC(6) User=0.00s Sys=0.00s Real=0.00s
[0.085s][info   ][gc,start    ] GC(7) Pause Full (Allocation Failure)
[0.085s][info   ][gc,phases,start] GC(7) Phase 1: Mark live objects
[0.086s][info   ][gc,phases      ] GC(7) Phase 1: Mark live objects 1.153ms
[0.086s][info   ][gc,phases,start] GC(7) Phase 2: Compute new object addresses
[0.086s][info   ][gc,phases      ] GC(7) Phase 2: Compute new object addresses 0.337ms
[0.086s][info   ][gc,phases,start] GC(7) Phase 3: Adjust pointers
[0.087s][info   ][gc,phases      ] GC(7) Phase 3: Adjust pointers 0.472ms
[0.087s][info   ][gc,phases,start] GC(7) Phase 4: Move objects
[0.091s][info   ][gc,phases      ] GC(7) Phase 4: Move objects 4.672ms
[0.092s][info   ][gc,heap        ] GC(7) DefNew: 39110K(39296K)->8773K(39296K) Eden: 34760K(34944K)->8773K(34944K) From: 4349K(4352K)->0K(4352K)
[0.092s][info   ][gc,heap        ] GC(7) Tenured: 81029K(87424K)->87392K(87424K)
[0.092s][info   ][gc,metaspace   ] GC(7) Metaspace: 155K(384K)->155K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.092s][info   ][gc             ] GC(7) Pause Full (Allocation Failure) 117M->93M(123M) 6.850ms
[0.092s][info   ][gc,cpu         ] GC(7) User=0.01s Sys=0.00s Real=0.01s
[0.095s][info   ][gc,start       ] GC(8) Pause Full (Allocation Failure)
[0.095s][info   ][gc,phases,start] GC(8) Phase 1: Mark live objects
[0.095s][info   ][gc,phases      ] GC(8) Phase 1: Mark live objects 0.749ms
[0.095s][info   ][gc,phases,start] GC(8) Phase 2: Compute new object addresses
[0.096s][info   ][gc,phases      ] GC(8) Phase 2: Compute new object addresses 0.296ms
[0.096s][info   ][gc,phases,start] GC(8) Phase 3: Adjust pointers
[0.096s][info   ][gc,phases      ] GC(8) Phase 3: Adjust pointers 0.471ms
[0.096s][info   ][gc,phases,start] GC(8) Phase 4: Move objects
[0.099s][info   ][gc,phases      ] GC(8) Phase 4: Move objects 2.909ms
[0.099s][info   ][gc,heap        ] GC(8) DefNew: 39262K(39296K)->18370K(39296K) Eden: 34944K(34944K)->18370K(34944K) From: 4318K(4352K)->0K(4352K)
[0.099s][info   ][gc,heap        ] GC(8) Tenured: 87392K(87424K)->86941K(87424K)
[0.099s][info   ][gc,metaspace   ] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.099s][info   ][gc             ] GC(8) Pause Full (Allocation Failure) 123M->102M(123M) 4.681ms
[0.099s][info   ][gc,cpu         ] GC(8) User=0.01s Sys=0.00s Real=0.00s
[0.102s][info   ][gc,start       ] GC(9) Pause Full (Allocation Failure)
[0.102s][info   ][gc,phases,start] GC(9) Phase 1: Mark live objects
[0.103s][info   ][gc,phases      ] GC(9) Phase 1: Mark live objects 0.799ms
[0.103s][info   ][gc,phases,start] GC(9) Phase 2: Compute new object addresses
[0.103s][info   ][gc,phases      ] GC(9) Phase 2: Compute new object addresses 0.277ms
[0.103s][info   ][gc,phases,start] GC(9) Phase 3: Adjust pointers
[0.103s][info   ][gc,phases      ] GC(9) Phase 3: Adjust pointers 0.586ms
[0.103s][info   ][gc,phases,start] GC(9) Phase 4: Move objects
[0.107s][info   ][gc,phases      ] GC(9) Phase 4: Move objects 3.101ms
[0.107s][info   ][gc,heap        ] GC(9) DefNew: 39236K(39296K)->22308K(39296K) Eden: 34944K(34944K)->22308K(34944K) From: 4292K(4352K)->0K(4352K)
[0.107s][info   ][gc,heap        ] GC(9) Tenured: 87271K(87424K)->87127K(87424K)
[0.107s][info   ][gc,metaspace   ] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.107s][info   ][gc             ] GC(9) Pause Full (Allocation Failure) 123M->106M(123M) 5.070ms
[0.107s][info   ][gc,cpu         ] GC(9) User=0.01s Sys=0.00s Real=0.00s
[0.108s][info   ][gc,start       ] GC(10) Pause Full (Allocation Failure)
[0.108s][info   ][gc,phases,start] GC(10) Phase 1: Mark live objects
[0.109s][info   ][gc,phases      ] GC(10) Phase 1: Mark live objects 0.717ms
[0.109s][info   ][gc,phases,start] GC(10) Phase 2: Compute new object addresses
[0.109s][info   ][gc,phases      ] GC(10) Phase 2: Compute new object addresses 0.264ms
[0.109s][info   ][gc,phases,start] GC(10) Phase 3: Adjust pointers
[0.109s][info   ][gc,phases      ] GC(10) Phase 3: Adjust pointers 0.514ms
[0.109s][info   ][gc,phases,start] GC(10) Phase 4: Move objects
[0.115s][info   ][gc,phases      ] GC(10) Phase 4: Move objects 5.167ms
[0.115s][info   ][gc,heap        ] GC(10) DefNew: 39000K(39296K)->20667K(39296K) Eden: 34944K(34944K)->20667K(34944K) From: 4056K(4352K)->0K(4352K)
[0.115s][info   ][gc,heap        ] GC(10) Tenured: 87127K(87424K)->87132K(87424K)
[0.115s][info   ][gc,metaspace   ] GC(10) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.115s][info   ][gc             ] GC(10) Pause Full (Allocation Failure) 123M->105M(123M) 6.894ms
[0.115s][info   ][gc,cpu         ] GC(10) User=0.00s Sys=0.00s Real=0.01s
[0.117s][info   ][gc,start       ] GC(11) Pause Full (Allocation Failure)
[0.117s][info   ][gc,phases,start] GC(11) Phase 1: Mark live objects
[0.118s][info   ][gc,phases      ] GC(11) Phase 1: Mark live objects 0.705ms
[0.118s][info   ][gc,phases,start] GC(11) Phase 2: Compute new object addresses
[0.118s][info   ][gc,phases      ] GC(11) Phase 2: Compute new object addresses 0.447ms
[0.118s][info   ][gc,phases,start] GC(11) Phase 3: Adjust pointers
[0.119s][info   ][gc,phases      ] GC(11) Phase 3: Adjust pointers 0.495ms
[0.119s][info   ][gc,phases,start] GC(11) Phase 4: Move objects
[0.119s][info   ][gc,phases      ] GC(11) Phase 4: Move objects 0.717ms
[0.119s][info   ][gc,heap        ] GC(11) DefNew: 39253K(39296K)->26315K(39296K) Eden: 34944K(34944K)->26315K(34944K) From: 4309K(4352K)->0K(4352K)
[0.119s][info   ][gc,heap        ] GC(11) Tenured: 87132K(87424K)->87132K(87424K)
[0.119s][info   ][gc,metaspace   ] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.119s][info   ][gc             ] GC(11) Pause Full (Allocation Failure) 123M->110M(123M) 2.592ms
[0.119s][info   ][gc,cpu         ] GC(11) User=0.00s Sys=0.00s Real=0.01s
[0.120s][info   ][gc,start       ] GC(12) Pause Full (Allocation Failure)
[0.120s][info   ][gc,phases,start] GC(12) Phase 1: Mark live objects
[0.121s][info   ][gc,phases      ] GC(12) Phase 1: Mark live objects 0.716ms
[0.121s][info   ][gc,phases,start] GC(12) Phase 2: Compute new object addresses
[0.121s][info   ][gc,phases      ] GC(12) Phase 2: Compute new object addresses 0.262ms
[0.121s][info   ][gc,phases,start] GC(12) Phase 3: Adjust pointers
[0.121s][info   ][gc,phases      ] GC(12) Phase 3: Adjust pointers 0.468ms
[0.121s][info   ][gc,phases,start] GC(12) Phase 4: Move objects
[0.122s][info   ][gc,phases      ] GC(12) Phase 4: Move objects 0.754ms
[0.122s][info   ][gc,heap        ] GC(12) DefNew: 39165K(39296K)->31275K(39296K) Eden: 34944K(34944K)->31275K(34944K) From: 4221K(4352K)->0K(4352K)
[0.122s][info   ][gc,heap        ] GC(12) Tenured: 87132K(87424K)->87132K(87424K)
[0.122s][info   ][gc,metaspace   ] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.122s][info   ][gc             ] GC(12) Pause Full (Allocation Failure) 123M->115M(123M) 2.393ms
[0.122s][info   ][gc,cpu         ] GC(12) User=0.00s Sys=0.00s Real=0.00s
[0.123s][info   ][gc,start       ] GC(13) Pause Full (Allocation Failure)
[0.123s][info   ][gc,phases,start] GC(13) Phase 1: Mark live objects
[0.123s][info   ][gc,phases      ] GC(13) Phase 1: Mark live objects 0.703ms
[0.123s][info   ][gc,phases,start] GC(13) Phase 2: Compute new object addresses
[0.124s][info   ][gc,phases      ] GC(13) Phase 2: Compute new object addresses 0.257ms
[0.124s][info   ][gc,phases,start] GC(13) Phase 3: Adjust pointers
[0.124s][info   ][gc,phases      ] GC(13) Phase 3: Adjust pointers 0.455ms
[0.124s][info   ][gc,phases,start] GC(13) Phase 4: Move objects
[0.124s][info   ][gc,phases      ] GC(13) Phase 4: Move objects 0.272ms
[0.124s][info   ][gc,heap        ] GC(13) DefNew: 38965K(39296K)->34581K(39296K) Eden: 34944K(34944K)->34581K(34944K) From: 4021K(4352K)->0K(4352K)
[0.124s][info   ][gc,heap        ] GC(13) Tenured: 87132K(87424K)->87090K(87424K)
[0.124s][info   ][gc,metaspace   ] GC(13) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.125s][info   ][gc             ] GC(13) Pause Full (Allocation Failure) 123M->118M(123M) 1.844ms
[0.125s][info   ][gc,cpu         ] GC(13) User=0.00s Sys=0.00s Real=0.00s
[0.125s][info   ][gc,start       ] GC(14) Pause Full (Allocation Failure)
[0.125s][info   ][gc,phases,start] GC(14) Phase 1: Mark live objects
[0.126s][info   ][gc,phases      ] GC(14) Phase 1: Mark live objects 0.624ms
[0.126s][info   ][gc,phases,start] GC(14) Phase 2: Compute new object addresses
[0.126s][info   ][gc,phases      ] GC(14) Phase 2: Compute new object addresses 0.256ms
[0.126s][info   ][gc,phases,start] GC(14) Phase 3: Adjust pointers
[0.126s][info   ][gc,phases      ] GC(14) Phase 3: Adjust pointers 0.467ms
[0.126s][info   ][gc,phases,start] GC(14) Phase 4: Move objects
[0.130s][info   ][gc,phases      ] GC(14) Phase 4: Move objects 3.626ms
[0.130s][info   ][gc,heap        ] GC(14) DefNew: 39291K(39296K)->31220K(39296K) Eden: 34944K(34944K)->31220K(34944K) From: 4347K(4352K)->0K(4352K)
[0.130s][info   ][gc,heap        ] GC(14) Tenured: 87386K(87424K)->87412K(87424K)
[0.130s][info   ][gc,metaspace   ] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.130s][info   ][gc             ] GC(14) Pause Full (Allocation Failure) 123M->115M(123M) 5.206ms
[0.130s][info   ][gc,cpu         ] GC(14) User=0.01s Sys=0.00s Real=0.01s
[0.130s][info   ][gc,start       ] GC(15) Pause Full (Allocation Failure)
[0.130s][info   ][gc,phases,start] GC(15) Phase 1: Mark live objects
[0.131s][info   ][gc,phases      ] GC(15) Phase 1: Mark live objects 0.681ms
[0.131s][info   ][gc,phases,start] GC(15) Phase 2: Compute new object addresses
[0.131s][info   ][gc,phases      ] GC(15) Phase 2: Compute new object addresses 0.254ms
[0.131s][info   ][gc,phases,start] GC(15) Phase 3: Adjust pointers
[0.132s][info   ][gc,phases      ] GC(15) Phase 3: Adjust pointers 0.470ms
[0.132s][info   ][gc,phases,start] GC(15) Phase 4: Move objects
[0.133s][info   ][gc,phases      ] GC(15) Phase 4: Move objects 0.595ms
[0.133s][info   ][gc,heap        ] GC(15) DefNew: 38637K(39296K)->34717K(39296K) Eden: 34944K(34944K)->34717K(34944K) From: 3693K(4352K)->0K(4352K)
[0.133s][info   ][gc,heap        ] GC(15) Tenured: 87412K(87424K)->87412K(87424K)
[0.133s][info   ][gc,metaspace   ] GC(15) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.133s][info   ][gc             ] GC(15) Pause Full (Allocation Failure) 123M->119M(123M) 2.189ms
[0.133s][info   ][gc,cpu         ] GC(15) User=0.00s Sys=0.00s Real=0.00s
[0.133s][info   ][gc,start       ] GC(16) Pause Full (Allocation Failure)
[0.133s][info   ][gc,phases,start] GC(16) Phase 1: Mark live objects
[0.134s][info   ][gc,phases      ] GC(16) Phase 1: Mark live objects 0.642ms
[0.134s][info   ][gc,phases,start] GC(16) Phase 2: Compute new object addresses
[0.134s][info   ][gc,phases      ] GC(16) Phase 2: Compute new object addresses 0.259ms
[0.134s][info   ][gc,phases,start] GC(16) Phase 3: Adjust pointers
[0.135s][info   ][gc,phases      ] GC(16) Phase 3: Adjust pointers 0.468ms
[0.135s][info   ][gc,phases,start] GC(16) Phase 4: Move objects
[0.135s][info   ][gc,phases      ] GC(16) Phase 4: Move objects 0.522ms
[0.135s][info   ][gc,heap        ] GC(16) DefNew: 39167K(39296K)->35607K(39296K) Eden: 34944K(34944K)->34667K(34944K) From: 4223K(4352K)->939K(4352K)
[0.135s][info   ][gc,heap        ] GC(16) Tenured: 87412K(87424K)->87412K(87424K)
[0.135s][info   ][gc,metaspace   ] GC(16) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.135s][info   ][gc             ] GC(16) Pause Full (Allocation Failure) 123M->120M(123M) 2.098ms
[0.135s][info   ][gc,cpu         ] GC(16) User=0.00s Sys=0.00s Real=0.00s
[0.136s][info   ][gc,start       ] GC(17) Pause Full (Allocation Failure)
[0.136s][info   ][gc,phases,start] GC(17) Phase 1: Mark live objects
[0.136s][info   ][gc,phases      ] GC(17) Phase 1: Mark live objects 0.638ms
[0.136s][info   ][gc,phases,start] GC(17) Phase 2: Compute new object addresses
[0.137s][info   ][gc,phases      ] GC(17) Phase 2: Compute new object addresses 0.257ms
[0.137s][info   ][gc,phases,start] GC(17) Phase 3: Adjust pointers
[0.137s][info   ][gc,phases      ] GC(17) Phase 3: Adjust pointers 0.453ms
[0.137s][info   ][gc,phases,start] GC(17) Phase 4: Move objects
[0.138s][info   ][gc,phases      ] GC(17) Phase 4: Move objects 0.920ms
[0.138s][info   ][gc,heap        ] GC(17) DefNew: 38716K(39296K)->36469K(39296K) Eden: 34944K(34944K)->34616K(34944K) From: 3772K(4352K)->1853K(4352K)
[0.138s][info   ][gc,heap        ] GC(17) Tenured: 87412K(87424K)->87412K(87424K)
[0.138s][info   ][gc,metaspace   ] GC(17) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.138s][info   ][gc             ] GC(17) Pause Full (Allocation Failure) 123M->120M(123M) 2.471ms
[0.138s][info   ][gc,cpu         ] GC(17) User=0.01s Sys=0.00s Real=0.00s
[0.138s][info   ][gc,start       ] GC(18) Pause Full (Allocation Failure)
[0.138s][info   ][gc,phases,start] GC(18) Phase 1: Mark live objects
[0.139s][info   ][gc,phases      ] GC(18) Phase 1: Mark live objects 0.701ms
[0.139s][info   ][gc,phases,start] GC(18) Phase 2: Compute new object addresses
[0.139s][info   ][gc,phases      ] GC(18) Phase 2: Compute new object addresses 0.264ms
[0.139s][info   ][gc,phases,start] GC(18) Phase 3: Adjust pointers
[0.140s][info   ][gc,phases      ] GC(18) Phase 3: Adjust pointers 0.461ms
[0.140s][info   ][gc,phases,start] GC(18) Phase 4: Move objects
[0.143s][info   ][gc,phases      ] GC(18) Phase 4: Move objects 3.442ms
[0.143s][info   ][gc,heap        ] GC(18) DefNew: 39244K(39296K)->36771K(39296K) Eden: 34943K(34944K)->34930K(34944K) From: 4300K(4352K)->1840K(4352K)
[0.144s][info   ][gc,heap        ] GC(18) Tenured: 87412K(87424K)->87365K(87424K)
[0.144s][info   ][gc,metaspace   ] GC(18) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.144s][info   ][gc             ] GC(18) Pause Full (Allocation Failure) 123M->121M(123M) 5.098ms
[0.144s][info   ][gc,cpu         ] GC(18) User=0.00s Sys=0.00s Real=0.01s
[0.144s][info   ][gc,start       ] GC(19) Pause Full (Allocation Failure)
[0.144s][info   ][gc,phases,start] GC(19) Phase 1: Mark live objects
[0.145s][info   ][gc,phases      ] GC(19) Phase 1: Mark live objects 0.789ms
[0.145s][info   ][gc,phases,start] GC(19) Phase 2: Compute new object addresses
[0.145s][info   ][gc,phases      ] GC(19) Phase 2: Compute new object addresses 0.258ms
[0.145s][info   ][gc,phases,start] GC(19) Phase 3: Adjust pointers
[0.145s][info   ][gc,phases      ] GC(19) Phase 3: Adjust pointers 0.477ms
[0.145s][info   ][gc,phases,start] GC(19) Phase 4: Move objects
[0.146s][info   ][gc,phases      ] GC(19) Phase 4: Move objects 0.258ms
[0.146s][info   ][gc,heap        ] GC(19) DefNew: 39219K(39296K)->38523K(39296K) Eden: 34944K(34944K)->34930K(34944K) From: 4275K(4352K)->3592K(4352K)
[0.146s][info   ][gc,heap        ] GC(19) Tenured: 87365K(87424K)->87365K(87424K)
[0.146s][info   ][gc,metaspace   ] GC(19) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.146s][info   ][gc             ] GC(19) Pause Full (Allocation Failure) 123M->122M(123M) 1.957ms
[0.146s][info   ][gc,cpu         ] GC(19) User=0.00s Sys=0.00s Real=0.00s
[0.146s][info   ][gc,start       ] GC(20) Pause Full (Allocation Failure)
[0.146s][info   ][gc,phases,start] GC(20) Phase 1: Mark live objects
[0.147s][info   ][gc,phases      ] GC(20) Phase 1: Mark live objects 0.717ms
[0.147s][info   ][gc,phases,start] GC(20) Phase 2: Compute new object addresses
[0.147s][info   ][gc,phases      ] GC(20) Phase 2: Compute new object addresses 0.253ms
[0.147s][info   ][gc,phases,start] GC(20) Phase 3: Adjust pointers
[0.147s][info   ][gc,phases      ] GC(20) Phase 3: Adjust pointers 0.448ms
[0.147s][info   ][gc,phases,start] GC(20) Phase 4: Move objects
[0.147s][info   ][gc,phases      ] GC(20) Phase 4: Move objects 0.008ms
[0.147s][info   ][gc,heap        ] GC(20) DefNew: 39281K(39296K)->38838K(39296K) Eden: 34944K(34944K)->34931K(34944K) From: 4337K(4352K)->3907K(4352K)
[0.147s][info   ][gc,heap        ] GC(20) Tenured: 87365K(87424K)->87365K(87424K)
[0.147s][info   ][gc,metaspace   ] GC(20) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.147s][info   ][gc             ] GC(20) Pause Full (Allocation Failure) 123M->123M(123M) 1.594ms
[0.147s][info   ][gc,cpu         ] GC(20) User=0.00s Sys=0.00s Real=0.00s
[0.148s][info   ][gc,start       ] GC(21) Pause Full (Allocation Failure)
[0.148s][info   ][gc,phases,start] GC(21) Phase 1: Mark live objects
[0.148s][info   ][gc,phases      ] GC(21) Phase 1: Mark live objects 0.551ms
[0.148s][info   ][gc,phases,start] GC(21) Phase 2: Compute new object addresses
[0.148s][info   ][gc,phases      ] GC(21) Phase 2: Compute new object addresses 0.248ms
[0.148s][info   ][gc,phases,start] GC(21) Phase 3: Adjust pointers
[0.149s][info   ][gc,phases      ] GC(21) Phase 3: Adjust pointers 0.418ms
[0.149s][info   ][gc,phases,start] GC(21) Phase 4: Move objects
[0.149s][info   ][gc,phases      ] GC(21) Phase 4: Move objects 0.012ms
[0.149s][info   ][gc,heap        ] GC(21) DefNew: 39249K(39296K)->39101K(39296K) Eden: 34944K(34944K)->34931K(34944K) From: 4305K(4352K)->4170K(4352K)
[0.149s][info   ][gc,heap        ] GC(21) Tenured: 87365K(87424K)->87365K(87424K)
[0.149s][info   ][gc,metaspace   ] GC(21) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.149s][info   ][gc             ] GC(21) Pause Full (Allocation Failure) 123M->123M(123M) 1.355ms
[0.149s][info   ][gc,cpu         ] GC(21) User=0.00s Sys=0.00s Real=0.00s
[0.149s][info   ][gc,start       ] GC(22) Pause Full (Allocation Failure)
[0.149s][info   ][gc,phases,start] GC(22) Phase 1: Mark live objects
[0.150s][info   ][gc,phases      ] GC(22) Phase 1: Mark live objects 0.558ms
[0.150s][info   ][gc,phases,start] GC(22) Phase 2: Compute new object addresses
[0.150s][info   ][gc,phases      ] GC(22) Phase 2: Compute new object addresses 0.253ms
[0.150s][info   ][gc,phases,start] GC(22) Phase 3: Adjust pointers
[0.150s][info   ][gc,phases      ] GC(22) Phase 3: Adjust pointers 0.426ms
[0.150s][info   ][gc,phases,start] GC(22) Phase 4: Move objects
[0.154s][info   ][gc,phases      ] GC(22) Phase 4: Move objects 3.805ms
[0.154s][info   ][gc,heap        ] GC(22) DefNew: 39280K(39296K)->39047K(39296K) Eden: 34931K(34944K)->34871K(34944K) From: 4348K(4352K)->4175K(4352K)
[0.154s][info   ][gc,heap        ] GC(22) Tenured: 87387K(87424K)->87368K(87424K)
[0.154s][info   ][gc,metaspace   ] GC(22) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.154s][info   ][gc             ] GC(22) Pause Full (Allocation Failure) 123M->123M(123M) 5.380ms
[0.154s][info   ][gc,cpu         ] GC(22) User=0.00s Sys=0.00s Real=0.01s
[0.154s][info   ][gc,start       ] GC(23) Pause Full (Allocation Failure)
[0.154s][info   ][gc,phases,start] GC(23) Phase 1: Mark live objects
[0.155s][info   ][gc,phases      ] GC(23) Phase 1: Mark live objects 0.698ms
[0.155s][info   ][gc,phases,start] GC(23) Phase 2: Compute new object addresses
[0.155s][info   ][gc,phases      ] GC(23) Phase 2: Compute new object addresses 0.277ms
[0.155s][info   ][gc,phases,start] GC(23) Phase 3: Adjust pointers
[0.156s][info   ][gc,phases      ] GC(23) Phase 3: Adjust pointers 0.468ms
[0.156s][info   ][gc,phases,start] GC(23) Phase 4: Move objects
[0.156s][info   ][gc,phases      ] GC(23) Phase 4: Move objects 0.008ms
[0.156s][info   ][gc,heap        ] GC(23) DefNew: 39131K(39296K)->39047K(39296K) Eden: 34871K(34944K)->34871K(34944K) From: 4259K(4352K)->4175K(4352K)
[0.156s][info   ][gc,heap        ] GC(23) Tenured: 87368K(87424K)->87368K(87424K)
[0.156s][info   ][gc,metaspace   ] GC(23) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.156s][info   ][gc             ] GC(23) Pause Full (Allocation Failure) 123M->123M(123M) 1.622ms
[0.156s][info   ][gc,cpu         ] GC(23) User=0.00s Sys=0.00s Real=0.00s
[0.156s][info   ][gc,start       ] GC(24) Pause Full (Allocation Failure)
[0.156s][info   ][gc,phases,start] GC(24) Phase 1: Mark live objects
[0.157s][info   ][gc,phases      ] GC(24) Phase 1: Mark live objects 0.616ms
[0.157s][info   ][gc,phases,start] GC(24) Phase 2: Compute new object addresses
[0.157s][info   ][gc,phases      ] GC(24) Phase 2: Compute new object addresses 0.254ms
[0.157s][info   ][gc,phases,start] GC(24) Phase 3: Adjust pointers
[0.157s][info   ][gc,phases      ] GC(24) Phase 3: Adjust pointers 0.450ms
[0.157s][info   ][gc,phases,start] GC(24) Phase 4: Move objects
[0.157s][info   ][gc,phases      ] GC(24) Phase 4: Move objects 0.011ms
[0.158s][info   ][gc,heap        ] GC(24) DefNew: 39219K(39296K)->39152K(39296K) Eden: 34938K(34944K)->34871K(34944K) From: 4281K(4352K)->4281K(4352K)
[0.158s][info   ][gc,heap        ] GC(24) Tenured: 87368K(87424K)->87368K(87424K)
[0.158s][info   ][gc,metaspace   ] GC(24) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.158s][info   ][gc             ] GC(24) Pause Full (Allocation Failure) 123M->123M(123M) 1.490ms
[0.158s][info   ][gc,cpu         ] GC(24) User=0.00s Sys=0.00s Real=0.00s
[0.158s][info   ][gc,start       ] GC(25) Pause Full (Allocation Failure)
[0.158s][info   ][gc,phases,start] GC(25) Phase 1: Mark live objects
[0.158s][info   ][gc,phases      ] GC(25) Phase 1: Mark live objects 0.751ms
[0.158s][info   ][gc,phases,start] GC(25) Phase 2: Compute new object addresses
[0.159s][info   ][gc,phases      ] GC(25) Phase 2: Compute new object addresses 0.253ms
[0.159s][info   ][gc,phases,start] GC(25) Phase 3: Adjust pointers
[0.159s][info   ][gc,phases      ] GC(25) Phase 3: Adjust pointers 0.463ms
[0.159s][info   ][gc,phases,start] GC(25) Phase 4: Move objects
[0.162s][info   ][gc,phases      ] GC(25) Phase 4: Move objects 2.588ms
[0.162s][info   ][gc,heap        ] GC(25) DefNew: 39152K(39296K)->39152K(39296K) Eden: 34871K(34944K)->34871K(34944K) From: 4281K(4352K)->4281K(4352K)
[0.162s][info   ][gc,heap        ] GC(25) Tenured: 87368K(87424K)->87366K(87424K)
[0.162s][info   ][gc,metaspace   ] GC(25) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.162s][info   ][gc             ] GC(25) Pause Full (Allocation Failure) 123M->123M(123M) 4.292ms
[0.162s][info   ][gc,cpu         ] GC(25) User=0.01s Sys=0.00s Real=0.01s
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at GCLogAnalysis.generateGarbage(GCLogAnalysis.java:42)
	at GCLogAnalysis.main(GCLogAnalysis.java:20)
[0.163s][info   ][gc,heap,exit   ] Heap
[0.163s][info   ][gc,heap,exit   ]  def new generation   total 39296K, used 39225K [0x00000007f8000000, 0x00000007faaa0000, 0x00000007faaa0000)
[0.163s][info   ][gc,heap,exit   ]   eden space 34944K, 100% used [0x00000007f8000000, 0x00000007fa220000, 0x00000007fa220000)
[0.163s][info   ][gc,heap,exit   ]   from space 4352K,  98% used [0x00000007fa220000, 0x00000007fa64e5c0, 0x00000007fa660000)
[0.163s][info   ][gc,heap,exit   ]   to   space 4352K,   0% used [0x00000007fa660000, 0x00000007fa660000, 0x00000007faaa0000)
[0.163s][info   ][gc,heap,exit   ]  tenured generation   total 87424K, used 87366K [0x00000007faaa0000, 0x0000000800000000, 0x0000000800000000)
[0.163s][info   ][gc,heap,exit   ]    the space 87424K,  99% used [0x00000007faaa0000, 0x00000007ffff1a90, 0x00000007ffff1c00, 0x0000000800000000)
[0.163s][info   ][gc,heap,exit   ]  Metaspace       used 163K, committed 384K, reserved 1114112K
[0.163s][info   ][gc,heap,exit   ]   class space    used 7K, committed 128K, reserved 1048576K
```

接下来试试512m堆大小

```
java -XX:+UseSerialGC -Xms512m -Xmx512m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以看到效果，前面几次yong GC后，触发了很多次full GC，最终成功运行，但是时间很长
```
[0.002s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.006s][info   ][gc,init] CardTable entry size: 512
[0.006s][info   ][gc     ] Using Serial
[0.006s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.006s][info   ][gc,init] CPUs: 8 total, 8 available
[0.006s][info   ][gc,init] Memory: 16384M
[0.006s][info   ][gc,init] Large Page Support: Disabled
[0.006s][info   ][gc,init] NUMA Support: Disabled
[0.006s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.006s][info   ][gc,init] Heap Min Capacity: 512M
[0.006s][info   ][gc,init] Heap Initial Capacity: 512M
[0.006s][info   ][gc,init] Heap Max Capacity: 512M
[0.006s][info   ][gc,init] Pre-touch: Disabled
[0.006s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.006s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.006s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.061s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.079s][info   ][gc,heap     ] GC(0) DefNew: 139776K(157248K)->17472K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->17472K(17472K)
[0.079s][info   ][gc,heap     ] GC(0) Tenured: 992K(349568K)->36431K(349568K)
[0.079s][info   ][gc,metaspace] GC(0) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.079s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 137M->52M(494M) 18.248ms
[0.079s][info   ][gc,cpu      ] GC(0) User=0.01s Sys=0.01s Real=0.02s
[0.098s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.112s][info   ][gc,heap     ] GC(1) DefNew: 157248K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17472K(17472K)->17471K(17472K)
[0.112s][info   ][gc,heap     ] GC(1) Tenured: 36431K(349568K)->90386K(349568K)
[0.112s][info   ][gc,metaspace] GC(1) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.112s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 189M->105M(494M) 14.222ms
[0.112s][info   ][gc,cpu      ] GC(1) User=0.00s Sys=0.01s Real=0.01s
[0.125s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.133s][info   ][gc,heap     ] GC(2) DefNew: 156732K(157248K)->17471K(157248K) Eden: 139260K(139776K)->0K(139776K) From: 17471K(17472K)->17471K(17472K)
[0.133s][info   ][gc,heap     ] GC(2) Tenured: 90386K(349568K)->136437K(349568K)
[0.133s][info   ][gc,metaspace] GC(2) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.133s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 241M->150M(494M) 7.935ms
[0.133s][info   ][gc,cpu      ] GC(2) User=0.01s Sys=0.00s Real=0.00s
[0.142s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.151s][info   ][gc,heap     ] GC(3) DefNew: 157247K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17471K(17472K)->17471K(17472K)
[0.152s][info   ][gc,heap     ] GC(3) Tenured: 136437K(349568K)->192764K(349568K)
[0.152s][info   ][gc,metaspace] GC(3) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.152s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 286M->205M(494M) 9.671ms
[0.152s][info   ][gc,cpu      ] GC(3) User=0.00s Sys=0.01s Real=0.01s
[0.161s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.168s][info   ][gc,heap     ] GC(4) DefNew: 157247K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17471K(17472K)->17471K(17472K)
[0.168s][info   ][gc,heap     ] GC(4) Tenured: 192764K(349568K)->239146K(349568K)
[0.168s][info   ][gc,metaspace] GC(4) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.168s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 341M->250M(494M) 6.916ms
[0.168s][info   ][gc,cpu      ] GC(4) User=0.00s Sys=0.00s Real=0.01s
[0.176s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.183s][info   ][gc,heap     ] GC(5) DefNew: 157247K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17471K(17472K)->17471K(17472K)
[0.183s][info   ][gc,heap     ] GC(5) Tenured: 239146K(349568K)->289739K(349568K)
[0.183s][info   ][gc,metaspace] GC(5) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.183s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 387M->300M(494M) 7.126ms
[0.183s][info   ][gc,cpu      ] GC(5) User=0.01s Sys=0.01s Real=0.00s
[0.193s][info   ][gc,start    ] GC(6) Pause Young (Allocation Failure)
[0.200s][info   ][gc,heap     ] GC(6) DefNew: 157247K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17471K(17472K)->17471K(17472K)
[0.200s][info   ][gc,heap     ] GC(6) Tenured: 289739K(349568K)->340047K(349568K)
[0.200s][info   ][gc,metaspace] GC(6) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.200s][info   ][gc          ] GC(6) Pause Young (Allocation Failure) 436M->349M(494M) 7.159ms
[0.200s][info   ][gc,cpu      ] GC(6) User=0.00s Sys=0.00s Real=0.01s
[0.213s][info   ][gc,start    ] GC(7) Pause Young (Allocation Failure)
[0.213s][info   ][gc          ] GC(7) Pause Young (Allocation Failure) 485M->485M(494M) 0.085ms
[0.213s][info   ][gc,cpu      ] GC(7) User=0.00s Sys=0.00s Real=0.00s
[0.213s][info   ][gc,start    ] GC(8) Pause Full (Allocation Failure)
[0.213s][info   ][gc,phases,start] GC(8) Phase 1: Mark live objects
[0.214s][info   ][gc,phases      ] GC(8) Phase 1: Mark live objects 1.714ms
[0.214s][info   ][gc,phases,start] GC(8) Phase 2: Compute new object addresses
[0.216s][info   ][gc,phases      ] GC(8) Phase 2: Compute new object addresses 1.139ms
[0.216s][info   ][gc,phases,start] GC(8) Phase 3: Adjust pointers
[0.216s][info   ][gc,phases      ] GC(8) Phase 3: Adjust pointers 0.541ms
[0.216s][info   ][gc,phases,start] GC(8) Phase 4: Move objects
[0.245s][info   ][gc,phases      ] GC(8) Phase 4: Move objects 28.964ms
[0.245s][info   ][gc,heap        ] GC(8) DefNew: 157247K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17471K(17472K)->0K(17472K)
[0.245s][info   ][gc,heap        ] GC(8) Tenured: 340047K(349568K)->265000K(349568K)
[0.245s][info   ][gc,metaspace   ] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.245s][info   ][gc             ] GC(8) Pause Full (Allocation Failure) 485M->258M(494M) 32.666ms
[0.245s][info   ][gc,cpu         ] GC(8) User=0.02s Sys=0.02s Real=0.04s
[0.256s][info   ][gc,start       ] GC(9) Pause Young (Allocation Failure)
[0.259s][info   ][gc,heap        ] GC(9) DefNew: 139776K(157248K)->17470K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->17470K(17472K)
[0.259s][info   ][gc,heap        ] GC(9) Tenured: 265000K(349568K)->297074K(349568K)
[0.259s][info   ][gc,metaspace   ] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.259s][info   ][gc             ] GC(9) Pause Young (Allocation Failure) 395M->307M(494M) 3.024ms
[0.259s][info   ][gc,cpu         ] GC(9) User=0.00s Sys=0.00s Real=0.00s
[0.268s][info   ][gc,start       ] GC(10) Pause Young (Allocation Failure)
[0.268s][info   ][gc             ] GC(10) Pause Young (Allocation Failure) 443M->443M(494M) 0.064ms
[0.268s][info   ][gc,cpu         ] GC(10) User=0.00s Sys=0.00s Real=0.00s
[0.268s][info   ][gc,start       ] GC(11) Pause Full (Allocation Failure)
[0.268s][info   ][gc,phases,start] GC(11) Phase 1: Mark live objects
[0.269s][info   ][gc,phases      ] GC(11) Phase 1: Mark live objects 0.848ms
[0.269s][info   ][gc,phases,start] GC(11) Phase 2: Compute new object addresses
[0.269s][info   ][gc,phases      ] GC(11) Phase 2: Compute new object addresses 0.585ms
[0.269s][info   ][gc,phases,start] GC(11) Phase 3: Adjust pointers
[0.270s][info   ][gc,phases      ] GC(11) Phase 3: Adjust pointers 0.548ms
[0.270s][info   ][gc,phases,start] GC(11) Phase 4: Move objects
[0.279s][info   ][gc,phases      ] GC(11) Phase 4: Move objects 9.684ms
[0.280s][info   ][gc,heap        ] GC(11) DefNew: 157246K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17470K(17472K)->0K(17472K)
[0.280s][info   ][gc,heap        ] GC(11) Tenured: 297074K(349568K)->286655K(349568K)
[0.280s][info   ][gc,metaspace   ] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.280s][info   ][gc             ] GC(11) Pause Full (Allocation Failure) 443M->279M(494M) 11.897ms
[0.280s][info   ][gc,cpu         ] GC(11) User=0.01s Sys=0.00s Real=0.01s
[0.287s][info   ][gc,start       ] GC(12) Pause Young (Allocation Failure)
[0.290s][info   ][gc,heap        ] GC(12) DefNew: 139776K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->17471K(17472K)
[0.290s][info   ][gc,heap        ] GC(12) Tenured: 286655K(349568K)->322214K(349568K)
[0.290s][info   ][gc,metaspace   ] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.290s][info   ][gc             ] GC(12) Pause Young (Allocation Failure) 416M->331M(494M) 2.629ms
[0.290s][info   ][gc,cpu         ] GC(12) User=0.00s Sys=0.00s Real=0.00s
[0.297s][info   ][gc,start       ] GC(13) Pause Young (Allocation Failure)
[0.297s][info   ][gc             ] GC(13) Pause Young (Allocation Failure) 467M->467M(494M) 0.094ms
[0.297s][info   ][gc,cpu         ] GC(13) User=0.00s Sys=0.00s Real=0.00s
[0.297s][info   ][gc,start       ] GC(14) Pause Full (Allocation Failure)
[0.297s][info   ][gc,phases,start] GC(14) Phase 1: Mark live objects
[0.298s][info   ][gc,phases      ] GC(14) Phase 1: Mark live objects 0.916ms
[0.298s][info   ][gc,phases,start] GC(14) Phase 2: Compute new object addresses
[0.299s][info   ][gc,phases      ] GC(14) Phase 2: Compute new object addresses 0.565ms
[0.299s][info   ][gc,phases,start] GC(14) Phase 3: Adjust pointers
[0.299s][info   ][gc,phases      ] GC(14) Phase 3: Adjust pointers 0.548ms
[0.299s][info   ][gc,phases,start] GC(14) Phase 4: Move objects
[0.310s][info   ][gc,phases      ] GC(14) Phase 4: Move objects 10.832ms
[0.310s][info   ][gc,heap        ] GC(14) DefNew: 156857K(157248K)->0K(157248K) Eden: 139386K(139776K)->0K(139776K) From: 17471K(17472K)->0K(17472K)
[0.310s][info   ][gc,heap        ] GC(14) Tenured: 322214K(349568K)->307104K(349568K)
[0.310s][info   ][gc,metaspace   ] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.310s][info   ][gc             ] GC(14) Pause Full (Allocation Failure) 467M->299M(494M) 13.115ms
[0.310s][info   ][gc,cpu         ] GC(14) User=0.01s Sys=0.00s Real=0.01s
[0.317s][info   ][gc,start       ] GC(15) Pause Young (Allocation Failure)
[0.317s][info   ][gc             ] GC(15) Pause Young (Allocation Failure) 436M->436M(494M) 0.088ms
[0.317s][info   ][gc,cpu         ] GC(15) User=0.00s Sys=0.00s Real=0.00s
[0.317s][info   ][gc,start       ] GC(16) Pause Full (Allocation Failure)
[0.318s][info   ][gc,phases,start] GC(16) Phase 1: Mark live objects
[0.318s][info   ][gc,phases      ] GC(16) Phase 1: Mark live objects 0.847ms
[0.318s][info   ][gc,phases,start] GC(16) Phase 2: Compute new object addresses
[0.319s][info   ][gc,phases      ] GC(16) Phase 2: Compute new object addresses 0.515ms
[0.319s][info   ][gc,phases,start] GC(16) Phase 3: Adjust pointers
[0.319s][info   ][gc,phases      ] GC(16) Phase 3: Adjust pointers 0.585ms
[0.319s][info   ][gc,phases,start] GC(16) Phase 4: Move objects
[0.332s][info   ][gc,phases      ] GC(16) Phase 4: Move objects 12.746ms
[0.332s][info   ][gc,heap        ] GC(16) DefNew: 139776K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->0K(17472K)
[0.332s][info   ][gc,heap        ] GC(16) Tenured: 307104K(349568K)->302981K(349568K)
[0.332s][info   ][gc,metaspace   ] GC(16) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.332s][info   ][gc             ] GC(16) Pause Full (Allocation Failure) 436M->295M(494M) 14.919ms
[0.332s][info   ][gc,cpu         ] GC(16) User=0.01s Sys=0.00s Real=0.01s
[0.340s][info   ][gc,start       ] GC(17) Pause Young (Allocation Failure)
[0.343s][info   ][gc,heap        ] GC(17) DefNew: 139776K(157248K)->17471K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->17471K(17472K)
[0.343s][info   ][gc,heap        ] GC(17) Tenured: 302981K(349568K)->344094K(349568K)
[0.343s][info   ][gc,metaspace   ] GC(17) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.343s][info   ][gc             ] GC(17) Pause Young (Allocation Failure) 432M->353M(494M) 3.047ms
[0.344s][info   ][gc,cpu         ] GC(17) User=0.00s Sys=0.00s Real=0.00s
[0.352s][info   ][gc,start       ] GC(18) Pause Young (Allocation Failure)
[0.352s][info   ][gc             ] GC(18) Pause Young (Allocation Failure) 489M->489M(494M) 0.039ms
[0.352s][info   ][gc,cpu         ] GC(18) User=0.00s Sys=0.00s Real=0.00s
[0.352s][info   ][gc,start       ] GC(19) Pause Full (Allocation Failure)
[0.352s][info   ][gc,phases,start] GC(19) Phase 1: Mark live objects
[0.353s][info   ][gc,phases      ] GC(19) Phase 1: Mark live objects 0.851ms
[0.353s][info   ][gc,phases,start] GC(19) Phase 2: Compute new object addresses
[0.353s][info   ][gc,phases      ] GC(19) Phase 2: Compute new object addresses 0.641ms
[0.353s][info   ][gc,phases,start] GC(19) Phase 3: Adjust pointers
[0.354s][info   ][gc,phases      ] GC(19) Phase 3: Adjust pointers 0.556ms
[0.354s][info   ][gc,phases,start] GC(19) Phase 4: Move objects
[0.364s][info   ][gc,phases      ] GC(19) Phase 4: Move objects 9.830ms
[0.364s][info   ][gc,heap        ] GC(19) DefNew: 157247K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17471K(17472K)->0K(17472K)
[0.364s][info   ][gc,heap        ] GC(19) Tenured: 344094K(349568K)->335517K(349568K)
[0.364s][info   ][gc,metaspace   ] GC(19) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.364s][info   ][gc             ] GC(19) Pause Full (Allocation Failure) 489M->327M(494M) 12.093ms
[0.364s][info   ][gc,cpu         ] GC(19) User=0.01s Sys=0.00s Real=0.02s
[0.372s][info   ][gc,start       ] GC(20) Pause Young (Allocation Failure)
[0.372s][info   ][gc             ] GC(20) Pause Young (Allocation Failure) 464M->464M(494M) 0.060ms
[0.372s][info   ][gc,cpu         ] GC(20) User=0.00s Sys=0.00s Real=0.00s
[0.372s][info   ][gc,start       ] GC(21) Pause Full (Allocation Failure)
[0.372s][info   ][gc,phases,start] GC(21) Phase 1: Mark live objects
[0.373s][info   ][gc,phases      ] GC(21) Phase 1: Mark live objects 0.795ms
[0.373s][info   ][gc,phases,start] GC(21) Phase 2: Compute new object addresses
[0.373s][info   ][gc,phases      ] GC(21) Phase 2: Compute new object addresses 0.629ms
[0.373s][info   ][gc,phases,start] GC(21) Phase 3: Adjust pointers
[0.374s][info   ][gc,phases      ] GC(21) Phase 3: Adjust pointers 0.549ms
[0.374s][info   ][gc,phases,start] GC(21) Phase 4: Move objects
[0.384s][info   ][gc,phases      ] GC(21) Phase 4: Move objects 9.949ms
[0.384s][info   ][gc,heap        ] GC(21) DefNew: 139622K(157248K)->0K(157248K) Eden: 139622K(139776K)->0K(139776K) From: 0K(17472K)->0K(17472K)
[0.384s][info   ][gc,heap        ] GC(21) Tenured: 335517K(349568K)->333618K(349568K)
[0.384s][info   ][gc,metaspace   ] GC(21) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.384s][info   ][gc             ] GC(21) Pause Full (Allocation Failure) 464M->325M(494M) 12.146ms
[0.384s][info   ][gc,cpu         ] GC(21) User=0.01s Sys=0.00s Real=0.02s
[0.395s][info   ][gc,start       ] GC(22) Pause Young (Allocation Failure)
[0.395s][info   ][gc             ] GC(22) Pause Young (Allocation Failure) 462M->462M(494M) 0.044ms
[0.395s][info   ][gc,cpu         ] GC(22) User=0.00s Sys=0.00s Real=0.00s
[0.395s][info   ][gc,start       ] GC(23) Pause Full (Allocation Failure)
[0.395s][info   ][gc,phases,start] GC(23) Phase 1: Mark live objects
[0.396s][info   ][gc,phases      ] GC(23) Phase 1: Mark live objects 0.919ms
[0.396s][info   ][gc,phases,start] GC(23) Phase 2: Compute new object addresses
[0.397s][info   ][gc,phases      ] GC(23) Phase 2: Compute new object addresses 0.660ms
[0.397s][info   ][gc,phases,start] GC(23) Phase 3: Adjust pointers
[0.397s][info   ][gc,phases      ] GC(23) Phase 3: Adjust pointers 0.552ms
[0.397s][info   ][gc,phases,start] GC(23) Phase 4: Move objects
[0.408s][info   ][gc,phases      ] GC(23) Phase 4: Move objects 10.520ms
[0.408s][info   ][gc,heap        ] GC(23) DefNew: 139776K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->0K(17472K)
[0.408s][info   ][gc,heap        ] GC(23) Tenured: 333618K(349568K)->337034K(349568K)
[0.408s][info   ][gc,metaspace   ] GC(23) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.408s][info   ][gc             ] GC(23) Pause Full (Allocation Failure) 462M->329M(494M) 12.928ms
[0.408s][info   ][gc,cpu         ] GC(23) User=0.02s Sys=0.00s Real=0.01s
[0.416s][info   ][gc,start       ] GC(24) Pause Young (Allocation Failure)
[0.416s][info   ][gc             ] GC(24) Pause Young (Allocation Failure) 465M->465M(494M) 0.049ms
[0.416s][info   ][gc,cpu         ] GC(24) User=0.00s Sys=0.00s Real=0.00s
[0.416s][info   ][gc,start       ] GC(25) Pause Full (Allocation Failure)
[0.416s][info   ][gc,phases,start] GC(25) Phase 1: Mark live objects
[0.417s][info   ][gc,phases      ] GC(25) Phase 1: Mark live objects 0.831ms
[0.417s][info   ][gc,phases,start] GC(25) Phase 2: Compute new object addresses
[0.418s][info   ][gc,phases      ] GC(25) Phase 2: Compute new object addresses 0.650ms
[0.418s][info   ][gc,phases,start] GC(25) Phase 3: Adjust pointers
[0.418s][info   ][gc,phases      ] GC(25) Phase 3: Adjust pointers 0.546ms
[0.418s][info   ][gc,phases,start] GC(25) Phase 4: Move objects
[0.430s][info   ][gc,phases      ] GC(25) Phase 4: Move objects 11.446ms
[0.430s][info   ][gc,heap        ] GC(25) DefNew: 139776K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 0K(17472K)->0K(17472K)
[0.430s][info   ][gc,heap        ] GC(25) Tenured: 337034K(349568K)->323827K(349568K)
[0.430s][info   ][gc,metaspace   ] GC(25) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.430s][info   ][gc             ] GC(25) Pause Full (Allocation Failure) 465M->316M(494M) 13.698ms
[0.430s][info   ][gc,cpu         ] GC(25) User=0.02s Sys=0.00s Real=0.01s
[0.438s][info   ][gc,start       ] GC(26) Pause Young (Allocation Failure)
[0.438s][info   ][gc             ] GC(26) Pause Young (Allocation Failure) 452M->452M(494M) 0.045ms
[0.438s][info   ][gc,cpu         ] GC(26) User=0.00s Sys=0.00s Real=0.00s
[0.438s][info   ][gc,start       ] GC(27) Pause Full (Allocation Failure)
[0.438s][info   ][gc,phases,start] GC(27) Phase 1: Mark live objects
[0.438s][info   ][gc,phases      ] GC(27) Phase 1: Mark live objects 0.780ms
[0.438s][info   ][gc,phases,start] GC(27) Phase 2: Compute new object addresses
[0.439s][info   ][gc,phases      ] GC(27) Phase 2: Compute new object addresses 0.578ms
[0.439s][info   ][gc,phases,start] GC(27) Phase 3: Adjust pointers
[0.440s][info   ][gc,phases      ] GC(27) Phase 3: Adjust pointers 0.543ms
[0.440s][info   ][gc,phases,start] GC(27) Phase 4: Move objects
[0.448s][info   ][gc,phases      ] GC(27) Phase 4: Move objects 8.568ms
[0.448s][info   ][gc,heap        ] GC(27) DefNew: 139324K(157248K)->0K(157248K) Eden: 139324K(139776K)->0K(139776K) From: 0K(17472K)->0K(17472K)
[0.448s][info   ][gc,heap        ] GC(27) Tenured: 323827K(349568K)->341582K(349568K)
[0.448s][info   ][gc,metaspace   ] GC(27) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.448s][info   ][gc             ] GC(27) Pause Full (Allocation Failure) 452M->333M(494M) 10.722ms
[0.448s][info   ][gc,cpu         ] GC(27) User=0.01s Sys=0.00s Real=0.01s
[0.455s][info   ][gc,start       ] GC(28) Pause Young (Allocation Failure)
[0.455s][info   ][gc             ] GC(28) Pause Young (Allocation Failure) 470M->470M(494M) 0.078ms
[0.455s][info   ][gc,cpu         ] GC(28) User=0.00s Sys=0.00s Real=0.00s
[0.455s][info   ][gc,start       ] GC(29) Pause Full (Allocation Failure)
[0.455s][info   ][gc,phases,start] GC(29) Phase 1: Mark live objects
[0.456s][info   ][gc,phases      ] GC(29) Phase 1: Mark live objects 0.869ms
[0.456s][info   ][gc,phases,start] GC(29) Phase 2: Compute new object addresses
[0.457s][info   ][gc,phases      ] GC(29) Phase 2: Compute new object addresses 0.548ms
[0.457s][info   ][gc,phases,start] GC(29) Phase 3: Adjust pointers
[0.457s][info   ][gc,phases      ] GC(29) Phase 3: Adjust pointers 0.552ms
[0.457s][info   ][gc,phases,start] GC(29) Phase 4: Move objects
[0.468s][info   ][gc,phases      ] GC(29) Phase 4: Move objects 10.686ms
[0.468s][info   ][gc,heap        ] GC(29) DefNew: 139776K(157248K)->2162K(157248K) Eden: 139776K(139776K)->2162K(139776K) From: 0K(17472K)->0K(17472K)
[0.468s][info   ][gc,heap        ] GC(29) Tenured: 341582K(349568K)->349021K(349568K)
[0.468s][info   ][gc,metaspace   ] GC(29) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.468s][info   ][gc             ] GC(29) Pause Full (Allocation Failure) 470M->342M(494M) 12.888ms
[0.468s][info   ][gc,cpu         ] GC(29) User=0.01s Sys=0.00s Real=0.01s
[0.476s][info   ][gc,start       ] GC(30) Pause Full (Allocation Failure)
[0.476s][info   ][gc,phases,start] GC(30) Phase 1: Mark live objects
[0.477s][info   ][gc,phases      ] GC(30) Phase 1: Mark live objects 0.774ms
[0.477s][info   ][gc,phases,start] GC(30) Phase 2: Compute new object addresses
[0.478s][info   ][gc,phases      ] GC(30) Phase 2: Compute new object addresses 0.627ms
[0.478s][info   ][gc,phases,start] GC(30) Phase 3: Adjust pointers
[0.478s][info   ][gc,phases      ] GC(30) Phase 3: Adjust pointers 0.549ms
[0.478s][info   ][gc,phases,start] GC(30) Phase 4: Move objects
[0.490s][info   ][gc,phases      ] GC(30) Phase 4: Move objects 11.357ms
[0.490s][info   ][gc,heap        ] GC(30) DefNew: 157236K(157248K)->6660K(157248K) Eden: 139776K(139776K)->6660K(139776K) From: 17460K(17472K)->0K(17472K)
[0.490s][info   ][gc,heap        ] GC(30) Tenured: 349348K(349568K)->349431K(349568K)
[0.490s][info   ][gc,metaspace   ] GC(30) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.490s][info   ][gc             ] GC(30) Pause Full (Allocation Failure) 494M->347M(494M) 13.570ms
[0.490s][info   ][gc,cpu         ] GC(30) User=0.02s Sys=0.00s Real=0.01s
[0.499s][info   ][gc,start       ] GC(31) Pause Full (Allocation Failure)
[0.499s][info   ][gc,phases,start] GC(31) Phase 1: Mark live objects
[0.500s][info   ][gc,phases      ] GC(31) Phase 1: Mark live objects 0.808ms
[0.500s][info   ][gc,phases,start] GC(31) Phase 2: Compute new object addresses
[0.500s][info   ][gc,phases      ] GC(31) Phase 2: Compute new object addresses 0.630ms
[0.500s][info   ][gc,phases,start] GC(31) Phase 3: Adjust pointers
[0.501s][info   ][gc,phases      ] GC(31) Phase 3: Adjust pointers 0.551ms
[0.501s][info   ][gc,phases,start] GC(31) Phase 4: Move objects
[0.513s][info   ][gc,phases      ] GC(31) Phase 4: Move objects 12.077ms
[0.513s][info   ][gc,heap        ] GC(31) DefNew: 157242K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17466K(17472K)->0K(17472K)
[0.513s][info   ][gc,heap        ] GC(31) Tenured: 349539K(349568K)->336807K(349568K)
[0.513s][info   ][gc,metaspace   ] GC(31) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.513s][info   ][gc             ] GC(31) Pause Full (Allocation Failure) 494M->328M(494M) 14.318ms
[0.513s][info   ][gc,cpu         ] GC(31) User=0.02s Sys=0.00s Real=0.01s
[0.521s][info   ][gc,start       ] GC(32) Pause Young (Allocation Failure)
[0.521s][info   ][gc             ] GC(32) Pause Young (Allocation Failure) 465M->465M(494M) 0.063ms
[0.521s][info   ][gc,cpu         ] GC(32) User=0.00s Sys=0.00s Real=0.00s
[0.521s][info   ][gc,start       ] GC(33) Pause Full (Allocation Failure)
[0.521s][info   ][gc,phases,start] GC(33) Phase 1: Mark live objects
[0.522s][info   ][gc,phases      ] GC(33) Phase 1: Mark live objects 0.819ms
[0.522s][info   ][gc,phases,start] GC(33) Phase 2: Compute new object addresses
[0.523s][info   ][gc,phases      ] GC(33) Phase 2: Compute new object addresses 0.560ms
[0.523s][info   ][gc,phases,start] GC(33) Phase 3: Adjust pointers
[0.523s][info   ][gc,phases      ] GC(33) Phase 3: Adjust pointers 0.558ms
[0.523s][info   ][gc,phases,start] GC(33) Phase 4: Move objects
[0.530s][info   ][gc,phases      ] GC(33) Phase 4: Move objects 6.884ms
[0.530s][info   ][gc,heap        ] GC(33) DefNew: 139703K(157248K)->2909K(157248K) Eden: 139703K(139776K)->2909K(139776K) From: 0K(17472K)->0K(17472K)
[0.530s][info   ][gc,heap        ] GC(33) Tenured: 336807K(349568K)->349437K(349568K)
[0.530s][info   ][gc,metaspace   ] GC(33) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.530s][info   ][gc             ] GC(33) Pause Full (Allocation Failure) 465M->344M(494M) 9.019ms
[0.530s][info   ][gc,cpu         ] GC(33) User=0.00s Sys=0.00s Real=0.01s
[0.538s][info   ][gc,start       ] GC(34) Pause Full (Allocation Failure)
[0.539s][info   ][gc,phases,start] GC(34) Phase 1: Mark live objects
[0.539s][info   ][gc,phases      ] GC(34) Phase 1: Mark live objects 0.845ms
[0.539s][info   ][gc,phases,start] GC(34) Phase 2: Compute new object addresses
[0.540s][info   ][gc,phases      ] GC(34) Phase 2: Compute new object addresses 0.634ms
[0.540s][info   ][gc,phases,start] GC(34) Phase 3: Adjust pointers
[0.541s][info   ][gc,phases      ] GC(34) Phase 3: Adjust pointers 0.548ms
[0.541s][info   ][gc,phases,start] GC(34) Phase 4: Move objects
[0.551s][info   ][gc,phases      ] GC(34) Phase 4: Move objects 10.689ms
[0.551s][info   ][gc,heap        ] GC(34) DefNew: 156992K(157248K)->14461K(157248K) Eden: 139776K(139776K)->14461K(139776K) From: 17216K(17472K)->0K(17472K)
[0.551s][info   ][gc,heap        ] GC(34) Tenured: 349437K(349568K)->348945K(349568K)
[0.551s][info   ][gc,metaspace   ] GC(34) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.551s][info   ][gc             ] GC(34) Pause Full (Allocation Failure) 494M->354M(494M) 13.010ms
[0.551s][info   ][gc,cpu         ] GC(34) User=0.02s Sys=0.00s Real=0.01s
[0.560s][info   ][gc,start       ] GC(35) Pause Full (Allocation Failure)
[0.560s][info   ][gc,phases,start] GC(35) Phase 1: Mark live objects
[0.561s][info   ][gc,phases      ] GC(35) Phase 1: Mark live objects 0.747ms
[0.561s][info   ][gc,phases,start] GC(35) Phase 2: Compute new object addresses
[0.562s][info   ][gc,phases      ] GC(35) Phase 2: Compute new object addresses 0.623ms
[0.562s][info   ][gc,phases,start] GC(35) Phase 3: Adjust pointers
[0.562s][info   ][gc,phases      ] GC(35) Phase 3: Adjust pointers 0.540ms
[0.562s][info   ][gc,phases,start] GC(35) Phase 4: Move objects
[0.573s][info   ][gc,phases      ] GC(35) Phase 4: Move objects 10.715ms
[0.573s][info   ][gc,heap        ] GC(35) DefNew: 157247K(157248K)->13031K(157248K) Eden: 139776K(139776K)->13031K(139776K) From: 17471K(17472K)->0K(17472K)
[0.573s][info   ][gc,heap        ] GC(35) Tenured: 349562K(349568K)->349380K(349568K)
[0.573s][info   ][gc,metaspace   ] GC(35) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.573s][info   ][gc             ] GC(35) Pause Full (Allocation Failure) 494M->353M(494M) 12.855ms
[0.573s][info   ][gc,cpu         ] GC(35) User=0.02s Sys=0.00s Real=0.01s
[0.582s][info   ][gc,start       ] GC(36) Pause Full (Allocation Failure)
[0.582s][info   ][gc,phases,start] GC(36) Phase 1: Mark live objects
[0.583s][info   ][gc,phases      ] GC(36) Phase 1: Mark live objects 0.793ms
[0.583s][info   ][gc,phases,start] GC(36) Phase 2: Compute new object addresses
[0.583s][info   ][gc,phases      ] GC(36) Phase 2: Compute new object addresses 0.617ms
[0.583s][info   ][gc,phases,start] GC(36) Phase 3: Adjust pointers
[0.584s][info   ][gc,phases      ] GC(36) Phase 3: Adjust pointers 0.548ms
[0.584s][info   ][gc,phases,start] GC(36) Phase 4: Move objects
[0.597s][info   ][gc,phases      ] GC(36) Phase 4: Move objects 13.087ms
[0.597s][info   ][gc,heap        ] GC(36) DefNew: 156867K(157248K)->2089K(157248K) Eden: 139776K(139776K)->2089K(139776K) From: 17091K(17472K)->0K(17472K)
[0.597s][info   ][gc,heap        ] GC(36) Tenured: 349380K(349568K)->349559K(349568K)
[0.597s][info   ][gc,metaspace   ] GC(36) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.597s][info   ][gc             ] GC(36) Pause Full (Allocation Failure) 494M->343M(494M) 15.326ms
[0.597s][info   ][gc,cpu         ] GC(36) User=0.01s Sys=0.00s Real=0.02s
[0.609s][info   ][gc,start       ] GC(37) Pause Full (Allocation Failure)
[0.609s][info   ][gc,phases,start] GC(37) Phase 1: Mark live objects
[0.610s][info   ][gc,phases      ] GC(37) Phase 1: Mark live objects 0.811ms
[0.610s][info   ][gc,phases,start] GC(37) Phase 2: Compute new object addresses
[0.610s][info   ][gc,phases      ] GC(37) Phase 2: Compute new object addresses 0.620ms
[0.610s][info   ][gc,phases,start] GC(37) Phase 3: Adjust pointers
[0.611s][info   ][gc,phases      ] GC(37) Phase 3: Adjust pointers 0.556ms
[0.611s][info   ][gc,phases,start] GC(37) Phase 4: Move objects
[0.620s][info   ][gc,phases      ] GC(37) Phase 4: Move objects 9.410ms
[0.621s][info   ][gc,heap        ] GC(37) DefNew: 157135K(157248K)->18892K(157248K) Eden: 139776K(139776K)->18892K(139776K) From: 17359K(17472K)->0K(17472K)
[0.621s][info   ][gc,heap        ] GC(37) Tenured: 349559K(349568K)->349187K(349568K)
[0.621s][info   ][gc,metaspace   ] GC(37) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.621s][info   ][gc             ] GC(37) Pause Full (Allocation Failure) 494M->359M(494M) 11.644ms
[0.621s][info   ][gc,cpu         ] GC(37) User=0.01s Sys=0.00s Real=0.01s
[0.629s][info   ][gc,start       ] GC(38) Pause Full (Allocation Failure)
[0.629s][info   ][gc,phases,start] GC(38) Phase 1: Mark live objects
[0.630s][info   ][gc,phases      ] GC(38) Phase 1: Mark live objects 0.840ms
[0.630s][info   ][gc,phases,start] GC(38) Phase 2: Compute new object addresses
[0.631s][info   ][gc,phases      ] GC(38) Phase 2: Compute new object addresses 0.642ms
[0.631s][info   ][gc,phases,start] GC(38) Phase 3: Adjust pointers
[0.631s][info   ][gc,phases      ] GC(38) Phase 3: Adjust pointers 0.594ms
[0.631s][info   ][gc,phases,start] GC(38) Phase 4: Move objects
[0.642s][info   ][gc,phases      ] GC(38) Phase 4: Move objects 10.918ms
[0.642s][info   ][gc,heap        ] GC(38) DefNew: 157187K(157248K)->25556K(157248K) Eden: 139776K(139776K)->25556K(139776K) From: 17411K(17472K)->0K(17472K)
[0.642s][info   ][gc,heap        ] GC(38) Tenured: 349483K(349568K)->349555K(349568K)
[0.642s][info   ][gc,metaspace   ] GC(38) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.642s][info   ][gc             ] GC(38) Pause Full (Allocation Failure) 494M->366M(494M) 13.287ms
[0.642s][info   ][gc,cpu         ] GC(38) User=0.01s Sys=0.00s Real=0.01s
[0.650s][info   ][gc,start       ] GC(39) Pause Full (Allocation Failure)
[0.650s][info   ][gc,phases,start] GC(39) Phase 1: Mark live objects
[0.650s][info   ][gc,phases      ] GC(39) Phase 1: Mark live objects 0.815ms
[0.650s][info   ][gc,phases,start] GC(39) Phase 2: Compute new object addresses
[0.651s][info   ][gc,phases      ] GC(39) Phase 2: Compute new object addresses 0.603ms
[0.651s][info   ][gc,phases,start] GC(39) Phase 3: Adjust pointers
[0.652s][info   ][gc,phases      ] GC(39) Phase 3: Adjust pointers 0.557ms
[0.652s][info   ][gc,phases,start] GC(39) Phase 4: Move objects
[0.663s][info   ][gc,phases      ] GC(39) Phase 4: Move objects 11.124ms
[0.663s][info   ][gc,heap        ] GC(39) DefNew: 157159K(157248K)->18623K(157248K) Eden: 139776K(139776K)->18623K(139776K) From: 17383K(17472K)->0K(17472K)
[0.663s][info   ][gc,heap        ] GC(39) Tenured: 349555K(349568K)->349322K(349568K)
[0.663s][info   ][gc,metaspace   ] GC(39) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.663s][info   ][gc             ] GC(39) Pause Full (Allocation Failure) 494M->359M(494M) 13.374ms
[0.663s][info   ][gc,cpu         ] GC(39) User=0.01s Sys=0.00s Real=0.01s
[0.671s][info   ][gc,start       ] GC(40) Pause Full (Allocation Failure)
[0.671s][info   ][gc,phases,start] GC(40) Phase 1: Mark live objects
[0.672s][info   ][gc,phases      ] GC(40) Phase 1: Mark live objects 0.886ms
[0.672s][info   ][gc,phases,start] GC(40) Phase 2: Compute new object addresses
[0.672s][info   ][gc,phases      ] GC(40) Phase 2: Compute new object addresses 0.600ms
[0.672s][info   ][gc,phases,start] GC(40) Phase 3: Adjust pointers
[0.673s][info   ][gc,phases      ] GC(40) Phase 3: Adjust pointers 0.686ms
[0.673s][info   ][gc,phases,start] GC(40) Phase 4: Move objects
[0.685s][info   ][gc,phases      ] GC(40) Phase 4: Move objects 12.441ms
[0.685s][info   ][gc,heap        ] GC(40) DefNew: 156701K(157248K)->2071K(157248K) Eden: 139776K(139776K)->2071K(139776K) From: 16925K(17472K)->0K(17472K)
[0.685s][info   ][gc,heap        ] GC(40) Tenured: 349322K(349568K)->349271K(349568K)
[0.686s][info   ][gc,metaspace   ] GC(40) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.686s][info   ][gc             ] GC(40) Pause Full (Allocation Failure) 494M->343M(494M) 14.957ms
[0.686s][info   ][gc,cpu         ] GC(40) User=0.01s Sys=0.00s Real=0.02s
[0.694s][info   ][gc,start       ] GC(41) Pause Full (Allocation Failure)
[0.694s][info   ][gc,phases,start] GC(41) Phase 1: Mark live objects
[0.695s][info   ][gc,phases      ] GC(41) Phase 1: Mark live objects 0.787ms
[0.695s][info   ][gc,phases,start] GC(41) Phase 2: Compute new object addresses
[0.696s][info   ][gc,phases      ] GC(41) Phase 2: Compute new object addresses 0.637ms
[0.696s][info   ][gc,phases,start] GC(41) Phase 3: Adjust pointers
[0.696s][info   ][gc,phases      ] GC(41) Phase 3: Adjust pointers 0.546ms
[0.697s][info   ][gc,phases,start] GC(41) Phase 4: Move objects
[0.706s][info   ][gc,phases      ] GC(41) Phase 4: Move objects 9.183ms
[0.706s][info   ][gc,heap        ] GC(41) DefNew: 157171K(157248K)->15970K(157248K) Eden: 139776K(139776K)->15970K(139776K) From: 17395K(17472K)->0K(17472K)
[0.706s][info   ][gc,heap        ] GC(41) Tenured: 349271K(349568K)->349083K(349568K)
[0.706s][info   ][gc,metaspace   ] GC(41) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.706s][info   ][gc             ] GC(41) Pause Full (Allocation Failure) 494M->356M(494M) 11.410ms
[0.706s][info   ][gc,cpu         ] GC(41) User=0.01s Sys=0.00s Real=0.01s
[0.715s][info   ][gc,start       ] GC(42) Pause Full (Allocation Failure)
[0.715s][info   ][gc,phases,start] GC(42) Phase 1: Mark live objects
[0.716s][info   ][gc,phases      ] GC(42) Phase 1: Mark live objects 0.868ms
[0.716s][info   ][gc,phases,start] GC(42) Phase 2: Compute new object addresses
[0.717s][info   ][gc,phases      ] GC(42) Phase 2: Compute new object addresses 0.638ms
[0.717s][info   ][gc,phases,start] GC(42) Phase 3: Adjust pointers
[0.717s][info   ][gc,phases      ] GC(42) Phase 3: Adjust pointers 0.553ms
[0.717s][info   ][gc,phases,start] GC(42) Phase 4: Move objects
[0.728s][info   ][gc,phases      ] GC(42) Phase 4: Move objects 10.819ms
[0.728s][info   ][gc,heap        ] GC(42) DefNew: 157093K(157248K)->14052K(157248K) Eden: 139776K(139776K)->14052K(139776K) From: 17317K(17472K)->0K(17472K)
[0.728s][info   ][gc,heap        ] GC(42) Tenured: 349083K(349568K)->349199K(349568K)
[0.728s][info   ][gc,metaspace   ] GC(42) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.728s][info   ][gc             ] GC(42) Pause Full (Allocation Failure) 494M->354M(494M) 13.173ms
[0.728s][info   ][gc,cpu         ] GC(42) User=0.02s Sys=0.00s Real=0.01s
[0.736s][info   ][gc,start       ] GC(43) Pause Full (Allocation Failure)
[0.736s][info   ][gc,phases,start] GC(43) Phase 1: Mark live objects
[0.737s][info   ][gc,phases      ] GC(43) Phase 1: Mark live objects 0.867ms
[0.737s][info   ][gc,phases,start] GC(43) Phase 2: Compute new object addresses
[0.738s][info   ][gc,phases      ] GC(43) Phase 2: Compute new object addresses 0.631ms
[0.738s][info   ][gc,phases,start] GC(43) Phase 3: Adjust pointers
[0.738s][info   ][gc,phases      ] GC(43) Phase 3: Adjust pointers 0.552ms
[0.738s][info   ][gc,phases,start] GC(43) Phase 4: Move objects
[0.749s][info   ][gc,phases      ] GC(43) Phase 4: Move objects 11.168ms
[0.749s][info   ][gc,heap        ] GC(43) DefNew: 157182K(157248K)->7478K(157248K) Eden: 139776K(139776K)->7478K(139776K) From: 17406K(17472K)->0K(17472K)
[0.749s][info   ][gc,heap        ] GC(43) Tenured: 349199K(349568K)->349196K(349568K)
[0.749s][info   ][gc,metaspace   ] GC(43) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.750s][info   ][gc             ] GC(43) Pause Full (Allocation Failure) 494M->348M(494M) 13.534ms
[0.750s][info   ][gc,cpu         ] GC(43) User=0.02s Sys=0.00s Real=0.01s
[0.757s][info   ][gc,start       ] GC(44) Pause Full (Allocation Failure)
[0.757s][info   ][gc,phases,start] GC(44) Phase 1: Mark live objects
[0.758s][info   ][gc,phases      ] GC(44) Phase 1: Mark live objects 0.791ms
[0.758s][info   ][gc,phases,start] GC(44) Phase 2: Compute new object addresses
[0.759s][info   ][gc,phases      ] GC(44) Phase 2: Compute new object addresses 0.618ms
[0.759s][info   ][gc,phases,start] GC(44) Phase 3: Adjust pointers
[0.759s][info   ][gc,phases      ] GC(44) Phase 3: Adjust pointers 0.538ms
[0.759s][info   ][gc,phases,start] GC(44) Phase 4: Move objects
[0.772s][info   ][gc,phases      ] GC(44) Phase 4: Move objects 12.159ms
[0.772s][info   ][gc,heap        ] GC(44) DefNew: 157054K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17278K(17472K)->0K(17472K)
[0.772s][info   ][gc,heap        ] GC(44) Tenured: 349196K(349568K)->340337K(349568K)
[0.772s][info   ][gc,metaspace   ] GC(44) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.772s][info   ][gc             ] GC(44) Pause Full (Allocation Failure) 494M->332M(494M) 14.407ms
[0.772s][info   ][gc,cpu         ] GC(44) User=0.01s Sys=0.00s Real=0.01s
[0.780s][info   ][gc,start       ] GC(45) Pause Young (Allocation Failure)
[0.780s][info   ][gc             ] GC(45) Pause Young (Allocation Failure) 468M->468M(494M) 0.069ms
[0.780s][info   ][gc,cpu         ] GC(45) User=0.00s Sys=0.00s Real=0.00s
[0.780s][info   ][gc,start       ] GC(46) Pause Full (Allocation Failure)
[0.780s][info   ][gc,phases,start] GC(46) Phase 1: Mark live objects
[0.781s][info   ][gc,phases      ] GC(46) Phase 1: Mark live objects 0.898ms
[0.781s][info   ][gc,phases,start] GC(46) Phase 2: Compute new object addresses
[0.781s][info   ][gc,phases      ] GC(46) Phase 2: Compute new object addresses 0.646ms
[0.781s][info   ][gc,phases,start] GC(46) Phase 3: Adjust pointers
[0.782s][info   ][gc,phases      ] GC(46) Phase 3: Adjust pointers 0.566ms
[0.782s][info   ][gc,phases,start] GC(46) Phase 4: Move objects
[0.791s][info   ][gc,phases      ] GC(46) Phase 4: Move objects 8.666ms
[0.791s][info   ][gc,heap        ] GC(46) DefNew: 139776K(157248K)->7769K(157248K) Eden: 139776K(139776K)->7769K(139776K) From: 0K(17472K)->0K(17472K)
[0.791s][info   ][gc,heap        ] GC(46) Tenured: 340337K(349568K)->349312K(349568K)
[0.791s][info   ][gc,metaspace   ] GC(46) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.791s][info   ][gc             ] GC(46) Pause Full (Allocation Failure) 468M->348M(494M) 11.036ms
[0.791s][info   ][gc,cpu         ] GC(46) User=0.01s Sys=0.00s Real=0.01s
[0.799s][info   ][gc,start       ] GC(47) Pause Full (Allocation Failure)
[0.799s][info   ][gc,phases,start] GC(47) Phase 1: Mark live objects
[0.800s][info   ][gc,phases      ] GC(47) Phase 1: Mark live objects 0.828ms
[0.800s][info   ][gc,phases,start] GC(47) Phase 2: Compute new object addresses
[0.800s][info   ][gc,phases      ] GC(47) Phase 2: Compute new object addresses 0.634ms
[0.801s][info   ][gc,phases,start] GC(47) Phase 3: Adjust pointers
[0.801s][info   ][gc,phases      ] GC(47) Phase 3: Adjust pointers 0.550ms
[0.801s][info   ][gc,phases,start] GC(47) Phase 4: Move objects
[0.812s][info   ][gc,phases      ] GC(47) Phase 4: Move objects 10.607ms
[0.812s][info   ][gc,heap        ] GC(47) DefNew: 157222K(157248K)->10018K(157248K) Eden: 139776K(139776K)->10018K(139776K) From: 17446K(17472K)->0K(17472K)
[0.812s][info   ][gc,heap        ] GC(47) Tenured: 349514K(349568K)->349365K(349568K)
[0.812s][info   ][gc,metaspace   ] GC(47) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.812s][info   ][gc             ] GC(47) Pause Full (Allocation Failure) 494M->350M(494M) 12.986ms
[0.812s][info   ][gc,cpu         ] GC(47) User=0.01s Sys=0.00s Real=0.01s
[0.820s][info   ][gc,start       ] GC(48) Pause Full (Allocation Failure)
[0.820s][info   ][gc,phases,start] GC(48) Phase 1: Mark live objects
[0.821s][info   ][gc,phases      ] GC(48) Phase 1: Mark live objects 0.826ms
[0.821s][info   ][gc,phases,start] GC(48) Phase 2: Compute new object addresses
[0.821s][info   ][gc,phases      ] GC(48) Phase 2: Compute new object addresses 0.621ms
[0.821s][info   ][gc,phases,start] GC(48) Phase 3: Adjust pointers
[0.822s][info   ][gc,phases      ] GC(48) Phase 3: Adjust pointers 0.557ms
[0.822s][info   ][gc,phases,start] GC(48) Phase 4: Move objects
[0.833s][info   ][gc,phases      ] GC(48) Phase 4: Move objects 11.102ms
[0.833s][info   ][gc,heap        ] GC(48) DefNew: 156898K(157248K)->16090K(157248K) Eden: 139776K(139776K)->16090K(139776K) From: 17122K(17472K)->0K(17472K)
[0.833s][info   ][gc,heap        ] GC(48) Tenured: 349365K(349568K)->349424K(349568K)
[0.833s][info   ][gc,metaspace   ] GC(48) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.833s][info   ][gc             ] GC(48) Pause Full (Allocation Failure) 494M->356M(494M) 13.404ms
[0.833s][info   ][gc,cpu         ] GC(48) User=0.01s Sys=0.00s Real=0.01s
[0.841s][info   ][gc,start       ] GC(49) Pause Full (Allocation Failure)
[0.841s][info   ][gc,phases,start] GC(49) Phase 1: Mark live objects
[0.842s][info   ][gc,phases      ] GC(49) Phase 1: Mark live objects 0.820ms
[0.842s][info   ][gc,phases,start] GC(49) Phase 2: Compute new object addresses
[0.843s][info   ][gc,phases      ] GC(49) Phase 2: Compute new object addresses 0.598ms
[0.843s][info   ][gc,phases,start] GC(49) Phase 3: Adjust pointers
[0.843s][info   ][gc,phases      ] GC(49) Phase 3: Adjust pointers 0.565ms
[0.843s][info   ][gc,phases,start] GC(49) Phase 4: Move objects
[0.856s][info   ][gc,phases      ] GC(49) Phase 4: Move objects 13.026ms
[0.856s][info   ][gc,heap        ] GC(49) DefNew: 157019K(157248K)->0K(157248K) Eden: 139776K(139776K)->0K(139776K) From: 17243K(17472K)->0K(17472K)
[0.856s][info   ][gc,heap        ] GC(49) Tenured: 349424K(349568K)->348902K(349568K)
[0.856s][info   ][gc,metaspace   ] GC(49) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.856s][info   ][gc             ] GC(49) Pause Full (Allocation Failure) 494M->340M(494M) 15.290ms
[0.856s][info   ][gc,cpu         ] GC(49) User=0.01s Sys=0.00s Real=0.02s
[0.863s][info   ][gc,start       ] GC(50) Pause Young (Allocation Failure)
[0.863s][info   ][gc             ] GC(50) Pause Young (Allocation Failure) 477M->477M(494M) 0.077ms
[0.863s][info   ][gc,cpu         ] GC(50) User=0.00s Sys=0.00s Real=0.00s
[0.864s][info   ][gc,start       ] GC(51) Pause Full (Allocation Failure)
[0.864s][info   ][gc,phases,start] GC(51) Phase 1: Mark live objects
[0.864s][info   ][gc,phases      ] GC(51) Phase 1: Mark live objects 0.903ms
[0.864s][info   ][gc,phases,start] GC(51) Phase 2: Compute new object addresses
[0.865s][info   ][gc,phases      ] GC(51) Phase 2: Compute new object addresses 0.601ms
[0.865s][info   ][gc,phases,start] GC(51) Phase 3: Adjust pointers
[0.866s][info   ][gc,phases      ] GC(51) Phase 3: Adjust pointers 0.547ms
[0.866s][info   ][gc,phases,start] GC(51) Phase 4: Move objects
[0.875s][info   ][gc,phases      ] GC(51) Phase 4: Move objects 8.977ms
[0.875s][info   ][gc,heap        ] GC(51) DefNew: 139776K(157248K)->23562K(157248K) Eden: 139776K(139776K)->23562K(139776K) From: 0K(17472K)->0K(17472K)
[0.875s][info   ][gc,heap        ] GC(51) Tenured: 348902K(349568K)->349464K(349568K)
[0.875s][info   ][gc,metaspace   ] GC(51) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.875s][info   ][gc             ] GC(51) Pause Full (Allocation Failure) 477M->364M(494M) 11.281ms
[0.875s][info   ][gc,cpu         ] GC(51) User=0.01s Sys=0.00s Real=0.02s
[0.882s][info   ][gc,start       ] GC(52) Pause Full (Allocation Failure)
[0.882s][info   ][gc,phases,start] GC(52) Phase 1: Mark live objects
[0.883s][info   ][gc,phases      ] GC(52) Phase 1: Mark live objects 0.917ms
[0.883s][info   ][gc,phases,start] GC(52) Phase 2: Compute new object addresses
[0.883s][info   ][gc,phases      ] GC(52) Phase 2: Compute new object addresses 0.661ms
[0.883s][info   ][gc,phases,start] GC(52) Phase 3: Adjust pointers
[0.884s][info   ][gc,phases      ] GC(52) Phase 3: Adjust pointers 0.584ms
[0.884s][info   ][gc,phases,start] GC(52) Phase 4: Move objects
[0.898s][info   ][gc,phases      ] GC(52) Phase 4: Move objects 13.409ms
[0.898s][info   ][gc,heap        ] GC(52) DefNew: 157156K(157248K)->25183K(157248K) Eden: 139776K(139776K)->25183K(139776K) From: 17380K(17472K)->0K(17472K)
[0.898s][info   ][gc,heap        ] GC(52) Tenured: 349464K(349568K)->349024K(349568K)
[0.898s][info   ][gc,metaspace   ] GC(52) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.898s][info   ][gc             ] GC(52) Pause Full (Allocation Failure) 494M->365M(494M) 15.926ms
[0.898s][info   ][gc,cpu         ] GC(52) User=0.01s Sys=0.00s Real=0.02s
[0.906s][info   ][gc,start       ] GC(53) Pause Full (Allocation Failure)
[0.906s][info   ][gc,phases,start] GC(53) Phase 1: Mark live objects
[0.907s][info   ][gc,phases      ] GC(53) Phase 1: Mark live objects 0.937ms
[0.907s][info   ][gc,phases,start] GC(53) Phase 2: Compute new object addresses
[0.907s][info   ][gc,phases      ] GC(53) Phase 2: Compute new object addresses 0.613ms
[0.907s][info   ][gc,phases,start] GC(53) Phase 3: Adjust pointers
[0.908s][info   ][gc,phases      ] GC(53) Phase 3: Adjust pointers 0.632ms
[0.908s][info   ][gc,phases,start] GC(53) Phase 4: Move objects
[0.922s][info   ][gc,phases      ] GC(53) Phase 4: Move objects 13.518ms
[0.922s][info   ][gc,heap        ] GC(53) DefNew: 157246K(157248K)->30369K(157248K) Eden: 139776K(139776K)->30369K(139776K) From: 17470K(17472K)->0K(17472K)
[0.922s][info   ][gc,heap        ] GC(53) Tenured: 349525K(349568K)->349346K(349568K)
[0.922s][info   ][gc,metaspace   ] GC(53) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.922s][info   ][gc             ] GC(53) Pause Full (Allocation Failure) 494M->370M(494M) 16.112ms
[0.922s][info   ][gc,cpu         ] GC(53) User=0.01s Sys=0.00s Real=0.01s
[0.929s][info   ][gc,start       ] GC(54) Pause Full (Allocation Failure)
[0.929s][info   ][gc,phases,start] GC(54) Phase 1: Mark live objects
[0.930s][info   ][gc,phases      ] GC(54) Phase 1: Mark live objects 0.899ms
[0.930s][info   ][gc,phases,start] GC(54) Phase 2: Compute new object addresses
[0.931s][info   ][gc,phases      ] GC(54) Phase 2: Compute new object addresses 0.544ms
[0.931s][info   ][gc,phases,start] GC(54) Phase 3: Adjust pointers
[0.931s][info   ][gc,phases      ] GC(54) Phase 3: Adjust pointers 0.554ms
[0.931s][info   ][gc,phases,start] GC(54) Phase 4: Move objects
[0.945s][info   ][gc,phases      ] GC(54) Phase 4: Move objects 13.288ms
[0.945s][info   ][gc,heap        ] GC(54) DefNew: 157162K(157248K)->15434K(157248K) Eden: 139776K(139776K)->15434K(139776K) From: 17386K(17472K)->0K(17472K)
[0.945s][info   ][gc,heap        ] GC(54) Tenured: 349441K(349568K)->349541K(349568K)
[0.945s][info   ][gc,metaspace   ] GC(54) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.945s][info   ][gc             ] GC(54) Pause Full (Allocation Failure) 494M->356M(494M) 16.098ms
[0.945s][info   ][gc,cpu         ] GC(54) User=0.02s Sys=0.00s Real=0.02s
[0.954s][info   ][gc,start       ] GC(55) Pause Full (Allocation Failure)
[0.954s][info   ][gc,phases,start] GC(55) Phase 1: Mark live objects
[0.954s][info   ][gc,phases      ] GC(55) Phase 1: Mark live objects 0.854ms
[0.954s][info   ][gc,phases,start] GC(55) Phase 2: Compute new object addresses
[0.955s][info   ][gc,phases      ] GC(55) Phase 2: Compute new object addresses 0.604ms
[0.955s][info   ][gc,phases,start] GC(55) Phase 3: Adjust pointers
[0.956s][info   ][gc,phases      ] GC(55) Phase 3: Adjust pointers 0.554ms
[0.956s][info   ][gc,phases,start] GC(55) Phase 4: Move objects
[0.965s][info   ][gc,phases      ] GC(55) Phase 4: Move objects 9.673ms
[0.965s][info   ][gc,heap        ] GC(55) DefNew: 156722K(157248K)->26861K(157248K) Eden: 139776K(139776K)->26861K(139776K) From: 16946K(17472K)->0K(17472K)
[0.965s][info   ][gc,heap        ] GC(55) Tenured: 349541K(349568K)->349125K(349568K)
[0.965s][info   ][gc,metaspace   ] GC(55) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.965s][info   ][gc             ] GC(55) Pause Full (Allocation Failure) 494M->367M(494M) 11.989ms
[0.966s][info   ][gc,cpu         ] GC(55) User=0.02s Sys=0.00s Real=0.02s
[0.972s][info   ][gc,start       ] GC(56) Pause Full (Allocation Failure)
[0.972s][info   ][gc,phases,start] GC(56) Phase 1: Mark live objects
[0.973s][info   ][gc,phases      ] GC(56) Phase 1: Mark live objects 0.788ms
[0.973s][info   ][gc,phases,start] GC(56) Phase 2: Compute new object addresses
[0.974s][info   ][gc,phases      ] GC(56) Phase 2: Compute new object addresses 0.557ms
[0.974s][info   ][gc,phases,start] GC(56) Phase 3: Adjust pointers
[0.974s][info   ][gc,phases      ] GC(56) Phase 3: Adjust pointers 0.557ms
[0.974s][info   ][gc,phases,start] GC(56) Phase 4: Move objects
[0.986s][info   ][gc,phases      ] GC(56) Phase 4: Move objects 11.644ms
[0.986s][info   ][gc,heap        ] GC(56) DefNew: 156826K(157248K)->26775K(157248K) Eden: 139776K(139776K)->26775K(139776K) From: 17050K(17472K)->0K(17472K)
[0.986s][info   ][gc,heap        ] GC(56) Tenured: 349125K(349568K)->349506K(349568K)
[0.986s][info   ][gc,metaspace   ] GC(56) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.986s][info   ][gc             ] GC(56) Pause Full (Allocation Failure) 494M->367M(494M) 13.802ms
[0.986s][info   ][gc,cpu         ] GC(56) User=0.02s Sys=0.00s Real=0.02s
[0.993s][info   ][gc,start       ] GC(57) Pause Full (Allocation Failure)
[0.994s][info   ][gc,phases,start] GC(57) Phase 1: Mark live objects
[0.994s][info   ][gc,phases      ] GC(57) Phase 1: Mark live objects 0.843ms
[0.994s][info   ][gc,phases,start] GC(57) Phase 2: Compute new object addresses
[0.995s][info   ][gc,phases      ] GC(57) Phase 2: Compute new object addresses 0.592ms
[0.995s][info   ][gc,phases,start] GC(57) Phase 3: Adjust pointers
[0.996s][info   ][gc,phases      ] GC(57) Phase 3: Adjust pointers 0.566ms
[0.996s][info   ][gc,phases,start] GC(57) Phase 4: Move objects
[1.008s][info   ][gc,phases      ] GC(57) Phase 4: Move objects 12.205ms
[1.008s][info   ][gc,heap        ] GC(57) DefNew: 157232K(157248K)->28181K(157248K) Eden: 139776K(139776K)->28181K(139776K) From: 17456K(17472K)->0K(17472K)
[1.008s][info   ][gc,heap        ] GC(57) Tenured: 349506K(349568K)->349071K(349568K)
[1.008s][info   ][gc,metaspace   ] GC(57) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.008s][info   ][gc             ] GC(57) Pause Full (Allocation Failure) 494M->368M(494M) 14.498ms
[1.008s][info   ][gc,cpu         ] GC(57) User=0.02s Sys=0.00s Real=0.02s
[1.015s][info   ][gc,start       ] GC(58) Pause Full (Allocation Failure)
[1.016s][info   ][gc,phases,start] GC(58) Phase 1: Mark live objects
[1.016s][info   ][gc,phases      ] GC(58) Phase 1: Mark live objects 0.831ms
[1.016s][info   ][gc,phases,start] GC(58) Phase 2: Compute new object addresses
[1.017s][info   ][gc,phases      ] GC(58) Phase 2: Compute new object addresses 0.606ms
[1.017s][info   ][gc,phases,start] GC(58) Phase 3: Adjust pointers
[1.018s][info   ][gc,phases      ] GC(58) Phase 3: Adjust pointers 0.553ms
[1.018s][info   ][gc,phases,start] GC(58) Phase 4: Move objects
[1.030s][info   ][gc,phases      ] GC(58) Phase 4: Move objects 12.647ms
[1.030s][info   ][gc,heap        ] GC(58) DefNew: 157242K(157248K)->8800K(157248K) Eden: 139776K(139776K)->8800K(139776K) From: 17466K(17472K)->0K(17472K)
[1.030s][info   ][gc,heap        ] GC(58) Tenured: 349534K(349568K)->349202K(349568K)
[1.030s][info   ][gc,metaspace   ] GC(58) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.030s][info   ][gc             ] GC(58) Pause Full (Allocation Failure) 494M->349M(494M) 14.913ms
[1.030s][info   ][gc,cpu         ] GC(58) User=0.02s Sys=0.00s Real=0.01s
counter:30528
[1.037s][info   ][gc,heap,exit   ] Heap
[1.037s][info   ][gc,heap,exit   ]  def new generation   total 157248K, used 14321K [0x00000007e0000000, 0x00000007eaaa0000, 0x00000007eaaa0000)
[1.037s][info   ][gc,heap,exit   ]   eden space 139776K,  10% used [0x00000007e0000000, 0x00000007e0dfc490, 0x00000007e8880000)
[1.037s][info   ][gc,heap,exit   ]   from space 17472K,   0% used [0x00000007e8880000, 0x00000007e8880000, 0x00000007e9990000)
[1.037s][info   ][gc,heap,exit   ]   to   space 17472K,   0% used [0x00000007e9990000, 0x00000007e9990000, 0x00000007eaaa0000)
[1.037s][info   ][gc,heap,exit   ]  tenured generation   total 349568K, used 349202K [0x00000007eaaa0000, 0x0000000800000000, 0x0000000800000000)
[1.037s][info   ][gc,heap,exit   ]    the space 349568K,  99% used [0x00000007eaaa0000, 0x00000007fffa4a88, 0x00000007fffa4c00, 0x0000000800000000)
[1.037s][info   ][gc,heap,exit   ]  Metaspace       used 233K, committed 448K, reserved 1114112K
[1.037s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```

再试试1G内存呢
```
java -XX:+UseSerialGC -Xms1g -Xmx1g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以看到GC次数大大减少，效率提升，512m的时候生成了30000多个对象，1g的时候生成了33000多个对象
```
[0.008s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.013s][info   ][gc,init] CardTable entry size: 512
[0.013s][info   ][gc     ] Using Serial
[0.013s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.013s][info   ][gc,init] CPUs: 8 total, 8 available
[0.013s][info   ][gc,init] Memory: 16384M
[0.013s][info   ][gc,init] Large Page Support: Disabled
[0.013s][info   ][gc,init] NUMA Support: Disabled
[0.014s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.014s][info   ][gc,init] Heap Min Capacity: 1G
[0.014s][info   ][gc,init] Heap Initial Capacity: 1G
[0.014s][info   ][gc,init] Heap Max Capacity: 1G
[0.014s][info   ][gc,init] Pre-touch: Disabled
[0.014s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.014s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.014s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.135s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.188s][info   ][gc,heap     ] GC(0) DefNew: 279616K(314560K)->34944K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 0K(34944K)->34944K(34944K)
[0.188s][info   ][gc,heap     ] GC(0) Tenured: 992K(699072K)->51147K(699072K)
[0.189s][info   ][gc,metaspace] GC(0) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.189s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 274M->84M(989M) 53.599ms
[0.189s][info   ][gc,cpu      ] GC(0) User=0.01s Sys=0.02s Real=0.06s
[0.219s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.240s][info   ][gc,heap     ] GC(1) DefNew: 314560K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34944K(34944K)->34943K(34944K)
[0.240s][info   ][gc,heap     ] GC(1) Tenured: 51147K(699072K)->141144K(699072K)
[0.240s][info   ][gc,metaspace] GC(1) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.240s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 357M->171M(989M) 21.714ms
[0.240s][info   ][gc,cpu      ] GC(1) User=0.00s Sys=0.01s Real=0.02s
[0.255s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.307s][info   ][gc,heap     ] GC(2) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.307s][info   ][gc,heap     ] GC(2) Tenured: 141144K(699072K)->242755K(699072K)
[0.307s][info   ][gc,metaspace] GC(2) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.307s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 445M->271M(989M) 51.781ms
[0.307s][info   ][gc,cpu      ] GC(2) User=0.01s Sys=0.01s Real=0.06s
[0.324s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.340s][info   ][gc,heap     ] GC(3) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.340s][info   ][gc,heap     ] GC(3) Tenured: 242755K(699072K)->320838K(699072K)
[0.340s][info   ][gc,metaspace] GC(3) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.340s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 544M->347M(989M) 15.604ms
[0.340s][info   ][gc,cpu      ] GC(3) User=0.01s Sys=0.01s Real=0.02s
[0.356s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.373s][info   ][gc,heap     ] GC(4) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.373s][info   ][gc,heap     ] GC(4) Tenured: 320838K(699072K)->407917K(699072K)
[0.373s][info   ][gc,metaspace] GC(4) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.373s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 620M->432M(989M) 17.074ms
[0.373s][info   ][gc,cpu      ] GC(4) User=0.00s Sys=0.01s Real=0.01s
[0.401s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.440s][info   ][gc,heap     ] GC(5) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.441s][info   ][gc,heap     ] GC(5) Tenured: 407917K(699072K)->492748K(699072K)
[0.441s][info   ][gc,metaspace] GC(5) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.441s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 705M->515M(989M) 39.247ms
[0.441s][info   ][gc,cpu      ] GC(5) User=0.00s Sys=0.01s Real=0.04s
[0.460s][info   ][gc,start    ] GC(6) Pause Young (Allocation Failure)
[0.480s][info   ][gc,heap     ] GC(6) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.480s][info   ][gc,heap     ] GC(6) Tenured: 492748K(699072K)->586536K(699072K)
[0.480s][info   ][gc,metaspace] GC(6) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.480s][info   ][gc          ] GC(6) Pause Young (Allocation Failure) 788M->606M(989M) 20.249ms
[0.480s][info   ][gc,cpu      ] GC(6) User=0.01s Sys=0.01s Real=0.02s
[0.500s][info   ][gc,start    ] GC(7) Pause Young (Allocation Failure)
[0.512s][info   ][gc,heap     ] GC(7) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.512s][info   ][gc,heap     ] GC(7) Tenured: 586536K(699072K)->665345K(699072K)
[0.512s][info   ][gc,metaspace] GC(7) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.512s][info   ][gc          ] GC(7) Pause Young (Allocation Failure) 879M->683M(989M) 12.727ms
[0.512s][info   ][gc,cpu      ] GC(7) User=0.01s Sys=0.01s Real=0.01s
[0.566s][info   ][gc,start    ] GC(8) Pause Young (Allocation Failure)
[0.566s][info   ][gc          ] GC(8) Pause Young (Allocation Failure) 956M->956M(989M) 0.081ms
[0.566s][info   ][gc,cpu      ] GC(8) User=0.00s Sys=0.00s Real=0.00s
[0.566s][info   ][gc,start    ] GC(9) Pause Full (Allocation Failure)
[0.566s][info   ][gc,phases,start] GC(9) Phase 1: Mark live objects
[0.568s][info   ][gc,phases      ] GC(9) Phase 1: Mark live objects 1.876ms
[0.568s][info   ][gc,phases,start] GC(9) Phase 2: Compute new object addresses
[0.572s][info   ][gc,phases      ] GC(9) Phase 2: Compute new object addresses 4.283ms
[0.572s][info   ][gc,phases,start] GC(9) Phase 3: Adjust pointers
[0.573s][info   ][gc,phases      ] GC(9) Phase 3: Adjust pointers 0.796ms
[0.573s][info   ][gc,phases,start] GC(9) Phase 4: Move objects
[0.620s][info   ][gc,phases      ] GC(9) Phase 4: Move objects 46.776ms
[0.620s][info   ][gc,heap        ] GC(9) DefNew: 314559K(314560K)->0K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->0K(34944K)
[0.620s][info   ][gc,heap        ] GC(9) Tenured: 665345K(699072K)->365197K(699072K)
[0.620s][info   ][gc,metaspace   ] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.620s][info   ][gc             ] GC(9) Pause Full (Allocation Failure) 956M->356M(989M) 54.196ms
[0.620s][info   ][gc,cpu         ] GC(9) User=0.02s Sys=0.03s Real=0.05s
[0.640s][info   ][gc,start       ] GC(10) Pause Young (Allocation Failure)
[0.647s][info   ][gc,heap        ] GC(10) DefNew: 279616K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 0K(34944K)->34943K(34944K)
[0.648s][info   ][gc,heap        ] GC(10) Tenured: 365197K(699072K)->432245K(699072K)
[0.648s][info   ][gc,metaspace   ] GC(10) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.648s][info   ][gc             ] GC(10) Pause Young (Allocation Failure) 629M->456M(989M) 7.406ms
[0.648s][info   ][gc,cpu         ] GC(10) User=0.01s Sys=0.00s Real=0.01s
[0.664s][info   ][gc,start       ] GC(11) Pause Young (Allocation Failure)
[0.672s][info   ][gc,heap        ] GC(11) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.672s][info   ][gc,heap        ] GC(11) Tenured: 432245K(699072K)->519962K(699072K)
[0.672s][info   ][gc,metaspace   ] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.672s][info   ][gc             ] GC(11) Pause Young (Allocation Failure) 729M->541M(989M) 8.206ms
[0.672s][info   ][gc,cpu         ] GC(11) User=0.00s Sys=0.01s Real=0.01s
[0.688s][info   ][gc,start       ] GC(12) Pause Young (Allocation Failure)
[0.695s][info   ][gc,heap        ] GC(12) DefNew: 314559K(314560K)->34942K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34942K(34944K)
[0.695s][info   ][gc,heap        ] GC(12) Tenured: 519962K(699072K)->607219K(699072K)
[0.695s][info   ][gc,metaspace   ] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.695s][info   ][gc             ] GC(12) Pause Young (Allocation Failure) 814M->627M(989M) 7.465ms
[0.695s][info   ][gc,cpu         ] GC(12) User=0.00s Sys=0.00s Real=0.00s
[0.710s][info   ][gc,start       ] GC(13) Pause Young (Allocation Failure)
[0.710s][info   ][gc             ] GC(13) Pause Young (Allocation Failure) 900M->900M(989M) 0.080ms
[0.710s][info   ][gc,cpu         ] GC(13) User=0.00s Sys=0.00s Real=0.00s
[0.710s][info   ][gc,start       ] GC(14) Pause Full (Allocation Failure)
[0.710s][info   ][gc,phases,start] GC(14) Phase 1: Mark live objects
[0.711s][info   ][gc,phases      ] GC(14) Phase 1: Mark live objects 0.829ms
[0.711s][info   ][gc,phases,start] GC(14) Phase 2: Compute new object addresses
[0.712s][info   ][gc,phases      ] GC(14) Phase 2: Compute new object addresses 1.122ms
[0.712s][info   ][gc,phases,start] GC(14) Phase 3: Adjust pointers
[0.713s][info   ][gc,phases      ] GC(14) Phase 3: Adjust pointers 0.564ms
[0.713s][info   ][gc,phases,start] GC(14) Phase 4: Move objects
[0.727s][info   ][gc,phases      ] GC(14) Phase 4: Move objects 14.587ms
[0.727s][info   ][gc,heap        ] GC(14) DefNew: 314558K(314560K)->0K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34942K(34944K)->0K(34944K)
[0.727s][info   ][gc,heap        ] GC(14) Tenured: 607219K(699072K)->377979K(699072K)
[0.727s][info   ][gc,metaspace   ] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.727s][info   ][gc             ] GC(14) Pause Full (Allocation Failure) 900M->369M(989M) 17.364ms
[0.727s][info   ][gc,cpu         ] GC(14) User=0.01s Sys=0.00s Real=0.02s
[0.742s][info   ][gc,start       ] GC(15) Pause Young (Allocation Failure)
[0.747s][info   ][gc,heap        ] GC(15) DefNew: 279616K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 0K(34944K)->34943K(34944K)
[0.748s][info   ][gc,heap        ] GC(15) Tenured: 377979K(699072K)->443461K(699072K)
[0.748s][info   ][gc,metaspace   ] GC(15) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.748s][info   ][gc             ] GC(15) Pause Young (Allocation Failure) 642M->467M(989M) 5.446ms
[0.748s][info   ][gc,cpu         ] GC(15) User=0.00s Sys=0.00s Real=0.01s
[0.764s][info   ][gc,start       ] GC(16) Pause Young (Allocation Failure)
[0.769s][info   ][gc,heap        ] GC(16) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.770s][info   ][gc,heap        ] GC(16) Tenured: 443461K(699072K)->538072K(699072K)
[0.770s][info   ][gc,metaspace   ] GC(16) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.770s][info   ][gc             ] GC(16) Pause Young (Allocation Failure) 740M->559M(989M) 5.349ms
[0.770s][info   ][gc,cpu         ] GC(16) User=0.00s Sys=0.00s Real=0.01s
[0.787s][info   ][gc,start       ] GC(17) Pause Young (Allocation Failure)
[0.793s][info   ][gc,heap        ] GC(17) DefNew: 314559K(314560K)->34942K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34942K(34944K)
[0.793s][info   ][gc,heap        ] GC(17) Tenured: 538072K(699072K)->623620K(699072K)
[0.793s][info   ][gc,metaspace   ] GC(17) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.795s][info   ][gc             ] GC(17) Pause Young (Allocation Failure) 832M->643M(989M) 7.911ms
[0.795s][info   ][gc,cpu         ] GC(17) User=0.01s Sys=0.00s Real=0.00s
[0.812s][info   ][gc,start       ] GC(18) Pause Young (Allocation Failure)
[0.812s][info   ][gc             ] GC(18) Pause Young (Allocation Failure) 916M->916M(989M) 0.057ms
[0.812s][info   ][gc,cpu         ] GC(18) User=0.00s Sys=0.00s Real=0.00s
[0.812s][info   ][gc,start       ] GC(19) Pause Full (Allocation Failure)
[0.812s][info   ][gc,phases,start] GC(19) Phase 1: Mark live objects
[0.813s][info   ][gc,phases      ] GC(19) Phase 1: Mark live objects 0.855ms
[0.813s][info   ][gc,phases,start] GC(19) Phase 2: Compute new object addresses
[0.814s][info   ][gc,phases      ] GC(19) Phase 2: Compute new object addresses 1.188ms
[0.814s][info   ][gc,phases,start] GC(19) Phase 3: Adjust pointers
[0.815s][info   ][gc,phases      ] GC(19) Phase 3: Adjust pointers 0.566ms
[0.815s][info   ][gc,phases,start] GC(19) Phase 4: Move objects
[0.827s][info   ][gc,phases      ] GC(19) Phase 4: Move objects 12.398ms
[0.828s][info   ][gc,heap        ] GC(19) DefNew: 314558K(314560K)->0K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34942K(34944K)->0K(34944K)
[0.828s][info   ][gc,heap        ] GC(19) Tenured: 623620K(699072K)->383627K(699072K)
[0.828s][info   ][gc,metaspace   ] GC(19) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.828s][info   ][gc             ] GC(19) Pause Full (Allocation Failure) 916M->374M(989M) 15.248ms
[0.828s][info   ][gc,cpu         ] GC(19) User=0.02s Sys=0.00s Real=0.02s
[0.843s][info   ][gc,start       ] GC(20) Pause Young (Allocation Failure)
[0.847s][info   ][gc,heap        ] GC(20) DefNew: 279616K(314560K)->34944K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 0K(34944K)->34944K(34944K)
[0.847s][info   ][gc,heap        ] GC(20) Tenured: 383627K(699072K)->441402K(699072K)
[0.847s][info   ][gc,metaspace   ] GC(20) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.847s][info   ][gc             ] GC(20) Pause Young (Allocation Failure) 647M->465M(989M) 4.018ms
[0.847s][info   ][gc,cpu         ] GC(20) User=0.01s Sys=0.00s Real=0.01s
[0.862s][info   ][gc,start       ] GC(21) Pause Young (Allocation Failure)
[0.867s][info   ][gc,heap        ] GC(21) DefNew: 314077K(314560K)->34943K(314560K) Eden: 279133K(279616K)->0K(279616K) From: 34944K(34944K)->34943K(34944K)
[0.867s][info   ][gc,heap        ] GC(21) Tenured: 441402K(699072K)->529351K(699072K)
[0.867s][info   ][gc,metaspace   ] GC(21) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.867s][info   ][gc             ] GC(21) Pause Young (Allocation Failure) 737M->551M(989M) 5.370ms
[0.867s][info   ][gc,cpu         ] GC(21) User=0.01s Sys=0.00s Real=0.01s
[0.881s][info   ][gc,start       ] GC(22) Pause Young (Allocation Failure)
[0.887s][info   ][gc,heap        ] GC(22) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.887s][info   ][gc,heap        ] GC(22) Tenured: 529351K(699072K)->627351K(699072K)
[0.887s][info   ][gc,metaspace   ] GC(22) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.887s][info   ][gc             ] GC(22) Pause Young (Allocation Failure) 824M->646M(989M) 5.647ms
[0.887s][info   ][gc,cpu         ] GC(22) User=0.01s Sys=0.00s Real=0.01s
[0.902s][info   ][gc,start       ] GC(23) Pause Young (Allocation Failure)
[0.902s][info   ][gc             ] GC(23) Pause Young (Allocation Failure) 919M->919M(989M) 0.079ms
[0.902s][info   ][gc,cpu         ] GC(23) User=0.00s Sys=0.00s Real=0.00s
[0.902s][info   ][gc,start       ] GC(24) Pause Full (Allocation Failure)
[0.902s][info   ][gc,phases,start] GC(24) Phase 1: Mark live objects
[0.903s][info   ][gc,phases      ] GC(24) Phase 1: Mark live objects 0.983ms
[0.903s][info   ][gc,phases,start] GC(24) Phase 2: Compute new object addresses
[0.904s][info   ][gc,phases      ] GC(24) Phase 2: Compute new object addresses 1.191ms
[0.904s][info   ][gc,phases,start] GC(24) Phase 3: Adjust pointers
[0.905s][info   ][gc,phases      ] GC(24) Phase 3: Adjust pointers 0.629ms
[0.905s][info   ][gc,phases,start] GC(24) Phase 4: Move objects
[0.921s][info   ][gc,phases      ] GC(24) Phase 4: Move objects 16.387ms
[0.921s][info   ][gc,heap        ] GC(24) DefNew: 314559K(314560K)->0K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->0K(34944K)
[0.921s][info   ][gc,heap        ] GC(24) Tenured: 627351K(699072K)->366208K(699072K)
[0.921s][info   ][gc,metaspace   ] GC(24) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.921s][info   ][gc             ] GC(24) Pause Full (Allocation Failure) 919M->357M(989M) 19.686ms
[0.921s][info   ][gc,cpu         ] GC(24) User=0.02s Sys=0.00s Real=0.02s
[0.937s][info   ][gc,start       ] GC(25) Pause Young (Allocation Failure)
[0.942s][info   ][gc,heap        ] GC(25) DefNew: 279616K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 0K(34944K)->34943K(34944K)
[0.942s][info   ][gc,heap        ] GC(25) Tenured: 366208K(699072K)->427626K(699072K)
[0.942s][info   ][gc,metaspace   ] GC(25) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.942s][info   ][gc             ] GC(25) Pause Young (Allocation Failure) 630M->451M(989M) 5.413ms
[0.942s][info   ][gc,cpu         ] GC(25) User=0.01s Sys=0.00s Real=0.00s
[0.957s][info   ][gc,start       ] GC(26) Pause Young (Allocation Failure)
[0.962s][info   ][gc,heap        ] GC(26) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.962s][info   ][gc,heap        ] GC(26) Tenured: 427626K(699072K)->508828K(699072K)
[0.962s][info   ][gc,metaspace   ] GC(26) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.962s][info   ][gc             ] GC(26) Pause Young (Allocation Failure) 724M->531M(989M) 5.551ms
[0.962s][info   ][gc,cpu         ] GC(26) User=0.01s Sys=0.00s Real=0.00s
[0.978s][info   ][gc,start       ] GC(27) Pause Young (Allocation Failure)
[0.985s][info   ][gc,heap        ] GC(27) DefNew: 314559K(314560K)->34943K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->34943K(34944K)
[0.985s][info   ][gc,heap        ] GC(27) Tenured: 508828K(699072K)->598561K(699072K)
[0.985s][info   ][gc,metaspace   ] GC(27) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.985s][info   ][gc             ] GC(27) Pause Young (Allocation Failure) 804M->618M(989M) 6.823ms
[0.985s][info   ][gc,cpu         ] GC(27) User=0.01s Sys=0.00s Real=0.00s
[1.000s][info   ][gc,start       ] GC(28) Pause Young (Allocation Failure)
[1.000s][info   ][gc             ] GC(28) Pause Young (Allocation Failure) 891M->891M(989M) 0.078ms
[1.000s][info   ][gc,cpu         ] GC(28) User=0.00s Sys=0.00s Real=0.00s
[1.000s][info   ][gc,start       ] GC(29) Pause Full (Allocation Failure)
[1.000s][info   ][gc,phases,start] GC(29) Phase 1: Mark live objects
[1.001s][info   ][gc,phases      ] GC(29) Phase 1: Mark live objects 0.884ms
[1.001s][info   ][gc,phases,start] GC(29) Phase 2: Compute new object addresses
[1.002s][info   ][gc,phases      ] GC(29) Phase 2: Compute new object addresses 1.359ms
[1.002s][info   ][gc,phases,start] GC(29) Phase 3: Adjust pointers
[1.003s][info   ][gc,phases      ] GC(29) Phase 3: Adjust pointers 0.875ms
[1.003s][info   ][gc,phases,start] GC(29) Phase 4: Move objects
[1.016s][info   ][gc,phases      ] GC(29) Phase 4: Move objects 13.026ms
[1.016s][info   ][gc,heap        ] GC(29) DefNew: 314559K(314560K)->0K(314560K) Eden: 279616K(279616K)->0K(279616K) From: 34943K(34944K)->0K(34944K)
[1.016s][info   ][gc,heap        ] GC(29) Tenured: 598561K(699072K)->385239K(699072K)
[1.016s][info   ][gc,metaspace   ] GC(29) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.016s][info   ][gc             ] GC(29) Pause Full (Allocation Failure) 891M->376M(989M) 16.473ms
[1.016s][info   ][gc,cpu         ] GC(29) User=0.02s Sys=0.00s Real=0.02s
counter:33659
[1.040s][info   ][gc,heap,exit   ] Heap
[1.040s][info   ][gc,heap,exit   ]  def new generation   total 314560K, used 279616K [0x00000007c0000000, 0x00000007d5550000, 0x00000007d5550000)
[1.040s][info   ][gc,heap,exit   ]   eden space 279616K, 100% used [0x00000007c0000000, 0x00000007d1110000, 0x00000007d1110000)
[1.040s][info   ][gc,heap,exit   ]   from space 34944K,   0% used [0x00000007d1110000, 0x00000007d1110000, 0x00000007d3330000)
[1.040s][info   ][gc,heap,exit   ]   to   space 34944K,   0% used [0x00000007d3330000, 0x00000007d3330000, 0x00000007d5550000)
[1.040s][info   ][gc,heap,exit   ]  tenured generation   total 699072K, used 385239K [0x00000007d5550000, 0x0000000800000000, 0x0000000800000000)
[1.040s][info   ][gc,heap,exit   ]    the space 699072K,  55% used [0x00000007d5550000, 0x00000007ecd85fa8, 0x00000007ecd86000, 0x0000000800000000)
[1.040s][info   ][gc,heap,exit   ]  Metaspace       used 236K, committed 448K, reserved 1114112K
[1.040s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```

接下来试试2g内存
```
java -XX:+UseSerialGC -Xms2g -Xmx2g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以看到GC次数减少了一半，生成了36000多个对象。性能再次提升
```
[0.001s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.005s][info   ][gc,init] CardTable entry size: 512
[0.005s][info   ][gc     ] Using Serial
[0.006s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.006s][info   ][gc,init] CPUs: 8 total, 8 available
[0.006s][info   ][gc,init] Memory: 16384M
[0.006s][info   ][gc,init] Large Page Support: Disabled
[0.006s][info   ][gc,init] NUMA Support: Disabled
[0.006s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.006s][info   ][gc,init] Heap Min Capacity: 2G
[0.006s][info   ][gc,init] Heap Initial Capacity: 2G
[0.006s][info   ][gc,init] Heap Max Capacity: 2G
[0.006s][info   ][gc,init] Pre-touch: Disabled
[0.006s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.006s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.006s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.101s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.122s][info   ][gc,heap     ] GC(0) DefNew: 559232K(629120K)->69888K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 0K(69888K)->69888K(69888K)
[0.122s][info   ][gc,heap     ] GC(0) Tenured: 992K(1398144K)->99229K(1398144K)
[0.122s][info   ][gc,metaspace] GC(0) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.122s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 547M->165M(1979M) 21.143ms
[0.122s][info   ][gc,cpu      ] GC(0) User=0.01s Sys=0.01s Real=0.02s
[0.157s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.181s][info   ][gc,heap     ] GC(1) DefNew: 629120K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69888K(69888K)->69887K(69888K)
[0.181s][info   ][gc,heap     ] GC(1) Tenured: 99229K(1398144K)->226904K(1398144K)
[0.181s][info   ][gc,metaspace] GC(1) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.181s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 711M->289M(1979M) 24.433ms
[0.181s][info   ][gc,cpu      ] GC(1) User=0.01s Sys=0.01s Real=0.03s
[0.216s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.236s][info   ][gc,heap     ] GC(2) DefNew: 629119K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->69887K(69888K)
[0.236s][info   ][gc,heap     ] GC(2) Tenured: 226904K(1398144K)->371886K(1398144K)
[0.236s][info   ][gc,metaspace] GC(2) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.236s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 835M->431M(1979M) 20.184ms
[0.236s][info   ][gc,cpu      ] GC(2) User=0.01s Sys=0.01s Real=0.02s
[0.268s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.300s][info   ][gc,heap     ] GC(3) DefNew: 629119K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->69887K(69888K)
[0.300s][info   ][gc,heap     ] GC(3) Tenured: 371886K(1398144K)->506536K(1398144K)
[0.300s][info   ][gc,metaspace] GC(3) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.300s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 977M->562M(1979M) 32.214ms
[0.301s][info   ][gc,cpu      ] GC(3) User=0.01s Sys=0.02s Real=0.04s
[0.330s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.361s][info   ][gc,heap     ] GC(4) DefNew: 629119K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->69887K(69888K)
[0.361s][info   ][gc,heap     ] GC(4) Tenured: 506536K(1398144K)->650167K(1398144K)
[0.361s][info   ][gc,metaspace] GC(4) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.361s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 1109M->703M(1979M) 30.771ms
[0.361s][info   ][gc,cpu      ] GC(4) User=0.01s Sys=0.01s Real=0.03s
[0.393s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.428s][info   ][gc,heap     ] GC(5) DefNew: 629119K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->69887K(69888K)
[0.428s][info   ][gc,heap     ] GC(5) Tenured: 650167K(1398144K)->778619K(1398144K)
[0.428s][info   ][gc,metaspace] GC(5) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.429s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 1249M->828M(1979M) 36.079ms
[0.429s][info   ][gc,cpu      ] GC(5) User=0.01s Sys=0.01s Real=0.04s
[0.465s][info   ][gc,start    ] GC(6) Pause Young (Allocation Failure)
[0.499s][info   ][gc,heap     ] GC(6) DefNew: 629119K(629120K)->69888K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->69888K(69888K)
[0.499s][info   ][gc,heap     ] GC(6) Tenured: 778619K(1398144K)->921168K(1398144K)
[0.500s][info   ][gc,metaspace] GC(6) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.500s][info   ][gc          ] GC(6) Pause Young (Allocation Failure) 1374M->967M(1979M) 34.268ms
[0.500s][info   ][gc,cpu      ] GC(6) User=0.02s Sys=0.02s Real=0.04s
[0.531s][info   ][gc,start    ] GC(7) Pause Young (Allocation Failure)
[0.557s][info   ][gc,heap     ] GC(7) DefNew: 629120K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69888K(69888K)->69887K(69888K)
[0.557s][info   ][gc,heap     ] GC(7) Tenured: 921168K(1398144K)->1054491K(1398144K)
[0.557s][info   ][gc,metaspace] GC(7) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.558s][info   ][gc          ] GC(7) Pause Young (Allocation Failure) 1513M->1098M(1979M) 26.353ms
[0.558s][info   ][gc,cpu      ] GC(7) User=0.01s Sys=0.02s Real=0.02s
[0.590s][info   ][gc,start    ] GC(8) Pause Young (Allocation Failure)
[0.617s][info   ][gc,heap     ] GC(8) DefNew: 629119K(629120K)->69888K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->69888K(69888K)
[0.617s][info   ][gc,heap     ] GC(8) Tenured: 1054491K(1398144K)->1190510K(1398144K)
[0.617s][info   ][gc,metaspace] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.617s][info   ][gc          ] GC(8) Pause Young (Allocation Failure) 1644M->1230M(1979M) 26.769ms
[0.617s][info   ][gc,cpu      ] GC(8) User=0.01s Sys=0.01s Real=0.02s
[0.647s][info   ][gc,start    ] GC(9) Pause Young (Allocation Failure)
[0.674s][info   ][gc,heap     ] GC(9) DefNew: 629120K(629120K)->69887K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69888K(69888K)->69887K(69888K)
[0.674s][info   ][gc,heap     ] GC(9) Tenured: 1190510K(1398144K)->1324896K(1398144K)
[0.674s][info   ][gc,metaspace] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.674s][info   ][gc          ] GC(9) Pause Young (Allocation Failure) 1776M->1362M(1979M) 26.741ms
[0.674s][info   ][gc,cpu      ] GC(9) User=0.01s Sys=0.01s Real=0.03s
[0.710s][info   ][gc,start    ] GC(10) Pause Young (Allocation Failure)
[0.710s][info   ][gc          ] GC(10) Pause Young (Allocation Failure) 1908M->1908M(1979M) 0.093ms
[0.710s][info   ][gc,cpu      ] GC(10) User=0.00s Sys=0.00s Real=0.00s
[0.710s][info   ][gc,start    ] GC(11) Pause Full (Allocation Failure)
[0.710s][info   ][gc,phases,start] GC(11) Phase 1: Mark live objects
[0.711s][info   ][gc,phases      ] GC(11) Phase 1: Mark live objects 1.296ms
[0.711s][info   ][gc,phases,start] GC(11) Phase 2: Compute new object addresses
[0.719s][info   ][gc,phases      ] GC(11) Phase 2: Compute new object addresses 8.079ms
[0.719s][info   ][gc,phases,start] GC(11) Phase 3: Adjust pointers
[0.720s][info   ][gc,phases      ] GC(11) Phase 3: Adjust pointers 0.684ms
[0.720s][info   ][gc,phases,start] GC(11) Phase 4: Move objects
[0.808s][info   ][gc,phases      ] GC(11) Phase 4: Move objects 87.442ms
[0.808s][info   ][gc,heap        ] GC(11) DefNew: 629119K(629120K)->0K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69887K(69888K)->0K(69888K)
[0.808s][info   ][gc,heap        ] GC(11) Tenured: 1324896K(1398144K)->345972K(1398144K)
[0.808s][info   ][gc,metaspace   ] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.808s][info   ][gc             ] GC(11) Pause Full (Allocation Failure) 1908M->337M(1979M) 97.966ms
[0.808s][info   ][gc,cpu         ] GC(11) User=0.03s Sys=0.06s Real=0.09s
[0.862s][info   ][gc,start       ] GC(12) Pause Young (Allocation Failure)
[0.887s][info   ][gc,heap        ] GC(12) DefNew: 559232K(629120K)->69888K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 0K(69888K)->69888K(69888K)
[0.888s][info   ][gc,heap        ] GC(12) Tenured: 345972K(1398144K)->447899K(1398144K)
[0.888s][info   ][gc,metaspace   ] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.888s][info   ][gc             ] GC(12) Pause Young (Allocation Failure) 883M->505M(1979M) 25.431ms
[0.888s][info   ][gc,cpu         ] GC(12) User=0.01s Sys=0.01s Real=0.02s
[0.921s][info   ][gc,start       ] GC(13) Pause Young (Allocation Failure)
[0.950s][info   ][gc,heap        ] GC(13) DefNew: 629120K(629120K)->69888K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69888K(69888K)->69888K(69888K)
[0.950s][info   ][gc,heap        ] GC(13) Tenured: 447899K(1398144K)->587988K(1398144K)
[0.950s][info   ][gc,metaspace   ] GC(13) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.950s][info   ][gc             ] GC(13) Pause Young (Allocation Failure) 1051M->642M(1979M) 28.591ms
[0.950s][info   ][gc,cpu         ] GC(13) User=0.01s Sys=0.02s Real=0.03s
[0.980s][info   ][gc,start       ] GC(14) Pause Young (Allocation Failure)
[1.031s][info   ][gc,heap        ] GC(14) DefNew: 629120K(629120K)->69888K(629120K) Eden: 559232K(559232K)->0K(559232K) From: 69888K(69888K)->69888K(69888K)
[1.031s][info   ][gc,heap        ] GC(14) Tenured: 587988K(1398144K)->731410K(1398144K)
[1.031s][info   ][gc,metaspace   ] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.031s][info   ][gc             ] GC(14) Pause Young (Allocation Failure) 1188M->782M(1979M) 51.353ms
[1.031s][info   ][gc,cpu         ] GC(14) User=0.01s Sys=0.01s Real=0.05s
counter:36077
[1.038s][info   ][gc,heap,exit   ] Heap
[1.038s][info   ][gc,heap,exit   ]  def new generation   total 629120K, used 92263K [0x0000000780000000, 0x00000007aaaa0000, 0x00000007aaaa0000)
[1.038s][info   ][gc,heap,exit   ]   eden space 559232K,   4% used [0x0000000780000000, 0x00000007815d9dc0, 0x00000007a2220000)
[1.038s][info   ][gc,heap,exit   ]   from space 69888K, 100% used [0x00000007a6660000, 0x00000007aaaa0000, 0x00000007aaaa0000)
[1.038s][info   ][gc,heap,exit   ]   to   space 69888K,   0% used [0x00000007a2220000, 0x00000007a2220000, 0x00000007a6660000)
[1.038s][info   ][gc,heap,exit   ]  tenured generation   total 1398144K, used 731410K [0x00000007aaaa0000, 0x0000000800000000, 0x0000000800000000)
[1.038s][info   ][gc,heap,exit   ]    the space 1398144K,  52% used [0x00000007aaaa0000, 0x00000007d74e4a20, 0x00000007d74e4c00, 0x0000000800000000)
[1.038s][info   ][gc,heap,exit   ]  Metaspace       used 232K, committed 448K, reserved 1114112K
[1.038s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```

#### Parallel GC

目标是最大化应用程序运行时间（吞吐量），最小化 GC 时间。还是会短暂的暂停业务。需要业务能接受短暂的暂停。

-XX: +UseParallelGC
-XX: +UseParallelOldGC 这个在java9以上已经被移除

年轻代和老年代GC都会触发STW事件。

对年轻代使用mark-copy算法，对老年代使用mark-sweep-compact算法

-XX: ParallelGCThreads=N 来指定GC线程数，默认值为CPU核心数。

优点：
- 多线程回收显著提高了回收效率，适合多核环境。
- 停顿时间较 Serial GC 短。
- 两次GC之间不消耗系统资源。

缺点：
- GC 停顿仍然是全暂停（STW）。
- 不适合对延迟要求苛刻的场景。

适用场景：
- 需要高吞吐量的大型后台任务（如批处理、数据分析）。
- 多核 CPU 环境。

接下来试试并行GC 128m内存
```
java -XX:+UseParallelGC -Xms128m -Xmx128m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

结果同样OOM，但是仅仅GC了16次,串行GC了25次，花了0.162s才OOM,而并行GC花了0.108s
```
[0.002s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.005s][info   ][gc,init] CardTable entry size: 512
[0.005s][info   ][gc     ] Using Parallel
[0.005s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.005s][info   ][gc,init] CPUs: 8 total, 8 available
[0.005s][info   ][gc,init] Memory: 16384M
[0.005s][info   ][gc,init] Large Page Support: Disabled
[0.005s][info   ][gc,init] NUMA Support: Disabled
[0.005s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.005s][info   ][gc,init] Alignments: Space 512K, Generation 512K, Heap 8M
[0.005s][info   ][gc,init] Heap Min Capacity: 128M
[0.005s][info   ][gc,init] Heap Initial Capacity: 128M
[0.005s][info   ][gc,init] Heap Max Capacity: 128M
[0.005s][info   ][gc,init] Pre-touch: Disabled
[0.005s][info   ][gc,init] Parallel Workers: 8
[0.006s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.006s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.006s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.028s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.031s][info   ][gc,heap     ] GC(0) PSYoungGen: 32583K(38400K)->5118K(38400K) Eden: 32583K(33280K)->0K(33280K) From: 0K(5120K)->5118K(5120K)
[0.031s][info   ][gc,heap     ] GC(0) ParOldGen: 992K(87552K)->10238K(87552K)
[0.031s][info   ][gc,metaspace] GC(0) Metaspace: 141K(384K)->141K(384K) NonClass: 135K(256K)->135K(256K) Class: 5K(128K)->5K(128K)
[0.031s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 32M->14M(123M) 2.555ms
[0.031s][info   ][gc,cpu      ] GC(0) User=0.00s Sys=0.00s Real=0.00s
[0.034s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.036s][info   ][gc,heap     ] GC(1) PSYoungGen: 38054K(38400K)->5094K(38400K) Eden: 32935K(33280K)->0K(33280K) From: 5118K(5120K)->5094K(5120K)
[0.036s][info   ][gc,heap     ] GC(1) ParOldGen: 10238K(87552K)->22029K(87552K)
[0.036s][info   ][gc,metaspace] GC(1) Metaspace: 147K(384K)->147K(384K) NonClass: 141K(256K)->141K(256K) Class: 5K(128K)->5K(128K)
[0.036s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 47M->26M(123M) 1.805ms
[0.036s][info   ][gc,cpu      ] GC(1) User=0.00s Sys=0.01s Real=0.00s
[0.038s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.040s][info   ][gc,heap     ] GC(2) PSYoungGen: 37809K(38400K)->5111K(38400K) Eden: 32715K(33280K)->0K(33280K) From: 5094K(5120K)->5111K(5120K)
[0.040s][info   ][gc,heap     ] GC(2) ParOldGen: 22029K(87552K)->37342K(87552K)
[0.040s][info   ][gc,metaspace] GC(2) Metaspace: 154K(384K)->154K(384K) NonClass: 148K(256K)->148K(256K) Class: 5K(128K)->5K(128K)
[0.040s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 58M->41M(123M) 2.612ms
[0.040s][info   ][gc,cpu      ] GC(2) User=0.00s Sys=0.00s Real=0.00s
[0.043s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.046s][info   ][gc,heap     ] GC(3) PSYoungGen: 38003K(38400K)->5115K(38400K) Eden: 32892K(33280K)->0K(33280K) From: 5111K(5120K)->5115K(5120K)
[0.046s][info   ][gc,heap     ] GC(3) ParOldGen: 37342K(87552K)->53010K(87552K)
[0.046s][info   ][gc,metaspace] GC(3) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.046s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 73M->56M(123M) 2.422ms
[0.046s][info   ][gc,cpu      ] GC(3) User=0.01s Sys=0.00s Real=0.00s
[0.048s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.051s][info   ][gc,heap     ] GC(4) PSYoungGen: 38395K(38400K)->5112K(38400K) Eden: 33280K(33280K)->0K(33280K) From: 5115K(5120K)->5112K(5120K)
[0.051s][info   ][gc,heap     ] GC(4) ParOldGen: 53010K(87552K)->65895K(87552K)
[0.051s][info   ][gc,metaspace] GC(4) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.051s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 89M->69M(123M) 2.624ms
[0.051s][info   ][gc,cpu      ] GC(4) User=0.01s Sys=0.01s Real=0.00s
[0.053s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.055s][info   ][gc,heap     ] GC(5) PSYoungGen: 38105K(38400K)->5100K(19968K) Eden: 32992K(33280K)->0K(14848K) From: 5112K(5120K)->5100K(5120K)
[0.056s][info   ][gc,heap     ] GC(5) ParOldGen: 65895K(87552K)->79828K(87552K)
[0.056s][info   ][gc,metaspace] GC(5) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.056s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 101M->82M(105M) 2.261ms
[0.056s][info   ][gc,cpu      ] GC(5) User=0.00s Sys=0.00s Real=0.00s
[0.056s][info   ][gc,start    ] GC(6) Pause Full (Ergonomics)
[0.056s][info   ][gc,phases,start] GC(6) Marking Phase
[0.057s][info   ][gc,phases      ] GC(6) Marking Phase 1.307ms
[0.057s][info   ][gc,phases,start] GC(6) Summary Phase
[0.057s][info   ][gc,phases      ] GC(6) Summary Phase 0.011ms
[0.057s][info   ][gc,phases,start] GC(6) Adjust Roots
[0.057s][info   ][gc,phases      ] GC(6) Adjust Roots 0.145ms
[0.057s][info   ][gc,phases,start] GC(6) Compaction Phase
[0.061s][info   ][gc,phases      ] GC(6) Compaction Phase 3.587ms
[0.061s][info   ][gc,phases,start] GC(6) Post Compact
[0.061s][info   ][gc,phases      ] GC(6) Post Compact 0.048ms
[0.061s][info   ][gc,heap        ] GC(6) PSYoungGen: 5100K(19968K)->0K(29184K) Eden: 0K(14848K)->0K(14848K) From: 5100K(5120K)->0K(14336K)
[0.061s][info   ][gc,heap        ] GC(6) ParOldGen: 79828K(87552K)->78802K(87552K)
[0.061s][info   ][gc,metaspace   ] GC(6) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.061s][info   ][gc             ] GC(6) Pause Full (Ergonomics) 82M->76M(114M) 5.271ms
[0.061s][info   ][gc,cpu         ] GC(6) User=0.01s Sys=0.00s Real=0.01s
[0.062s][info   ][gc,start       ] GC(7) Pause Full (Ergonomics)
[0.062s][info   ][gc,phases,start] GC(7) Marking Phase
[0.064s][info   ][gc,phases      ] GC(7) Marking Phase 1.826ms
[0.064s][info   ][gc,phases,start] GC(7) Summary Phase
[0.064s][info   ][gc,phases      ] GC(7) Summary Phase 0.014ms
[0.064s][info   ][gc,phases,start] GC(7) Adjust Roots
[0.064s][info   ][gc,phases      ] GC(7) Adjust Roots 0.144ms
[0.064s][info   ][gc,phases,start] GC(7) Compaction Phase
[0.066s][info   ][gc,phases      ] GC(7) Compaction Phase 1.702ms
[0.066s][info   ][gc,phases,start] GC(7) Post Compact
[0.066s][info   ][gc,phases      ] GC(7) Post Compact 0.054ms
[0.066s][info   ][gc,heap        ] GC(7) PSYoungGen: 14848K(29184K)->0K(29184K) Eden: 14848K(14848K)->0K(14848K) From: 0K(14336K)->0K(14336K)
[0.066s][info   ][gc,heap        ] GC(7) ParOldGen: 78802K(87552K)->83255K(87552K)
[0.066s][info   ][gc,metaspace   ] GC(7) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.066s][info   ][gc             ] GC(7) Pause Full (Ergonomics) 91M->81M(114M) 3.928ms
[0.066s][info   ][gc,cpu         ] GC(7) User=0.01s Sys=0.00s Real=0.00s
[0.067s][info   ][gc,start       ] GC(8) Pause Full (Ergonomics)
[0.067s][info   ][gc,phases,start] GC(8) Marking Phase
[0.069s][info   ][gc,phases      ] GC(8) Marking Phase 2.060ms
[0.069s][info   ][gc,phases,start] GC(8) Summary Phase
[0.069s][info   ][gc,phases      ] GC(8) Summary Phase 0.010ms
[0.069s][info   ][gc,phases,start] GC(8) Adjust Roots
[0.070s][info   ][gc,phases      ] GC(8) Adjust Roots 0.616ms
[0.070s][info   ][gc,phases,start] GC(8) Compaction Phase
[0.075s][info   ][gc,phases      ] GC(8) Compaction Phase 5.364ms
[0.076s][info   ][gc,phases,start] GC(8) Post Compact
[0.076s][info   ][gc,phases      ] GC(8) Post Compact 0.056ms
[0.076s][info   ][gc,heap        ] GC(8) PSYoungGen: 14643K(29184K)->0K(29184K) Eden: 14643K(14848K)->0K(14848K) From: 0K(14336K)->0K(14336K)
[0.076s][info   ][gc,heap        ] GC(8) ParOldGen: 83255K(87552K)->87322K(87552K)
[0.076s][info   ][gc,metaspace   ] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.076s][info   ][gc             ] GC(8) Pause Full (Ergonomics) 95M->85M(114M) 8.277ms
[0.076s][info   ][gc,cpu         ] GC(8) User=0.01s Sys=0.01s Real=0.00s
[0.078s][info   ][gc,start       ] GC(9) Pause Full (Ergonomics)
[0.078s][info   ][gc,phases,start] GC(9) Marking Phase
[0.079s][info   ][gc,phases      ] GC(9) Marking Phase 1.251ms
[0.079s][info   ][gc,phases,start] GC(9) Summary Phase
[0.079s][info   ][gc,phases      ] GC(9) Summary Phase 0.011ms
[0.079s][info   ][gc,phases,start] GC(9) Adjust Roots
[0.079s][info   ][gc,phases      ] GC(9) Adjust Roots 0.162ms
[0.079s][info   ][gc,phases,start] GC(9) Compaction Phase
[0.083s][info   ][gc,phases      ] GC(9) Compaction Phase 3.974ms
[0.084s][info   ][gc,phases,start] GC(9) Post Compact
[0.084s][info   ][gc,phases      ] GC(9) Post Compact 0.088ms
[0.084s][info   ][gc,heap        ] GC(9) PSYoungGen: 14556K(29184K)->4765K(29184K) Eden: 14556K(14848K)->4765K(14848K) From: 0K(14336K)->0K(14336K)
[0.084s][info   ][gc,heap        ] GC(9) ParOldGen: 87322K(87552K)->87356K(87552K)
[0.084s][info   ][gc,metaspace   ] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.084s][info   ][gc             ] GC(9) Pause Full (Ergonomics) 99M->89M(114M) 5.831ms
[0.084s][info   ][gc,cpu         ] GC(9) User=0.00s Sys=0.00s Real=0.00s
[0.085s][info   ][gc,start       ] GC(10) Pause Full (Ergonomics)
[0.086s][info   ][gc,phases,start] GC(10) Marking Phase
[0.088s][info   ][gc,phases      ] GC(10) Marking Phase 2.046ms
[0.088s][info   ][gc,phases,start] GC(10) Summary Phase
[0.088s][info   ][gc,phases      ] GC(10) Summary Phase 0.011ms
[0.088s][info   ][gc,phases,start] GC(10) Adjust Roots
[0.088s][info   ][gc,phases      ] GC(10) Adjust Roots 0.129ms
[0.088s][info   ][gc,phases,start] GC(10) Compaction Phase
[0.090s][info   ][gc,phases      ] GC(10) Compaction Phase 1.986ms
[0.090s][info   ][gc,phases,start] GC(10) Post Compact
[0.090s][info   ][gc,phases      ] GC(10) Post Compact 0.060ms
[0.090s][info   ][gc,heap        ] GC(10) PSYoungGen: 14439K(29184K)->10132K(29184K) Eden: 14439K(14848K)->10132K(14848K) From: 0K(14336K)->0K(14336K)
[0.090s][info   ][gc,heap        ] GC(10) ParOldGen: 87356K(87552K)->86976K(87552K)
[0.090s][info   ][gc,metaspace   ] GC(10) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.090s][info   ][gc             ] GC(10) Pause Full (Ergonomics) 99M->94M(114M) 4.406ms
[0.090s][info   ][gc,cpu         ] GC(10) User=0.01s Sys=0.00s Real=0.01s
[0.091s][info   ][gc,start       ] GC(11) Pause Full (Ergonomics)
[0.091s][info   ][gc,phases,start] GC(11) Marking Phase
[0.093s][info   ][gc,phases      ] GC(11) Marking Phase 2.085ms
[0.093s][info   ][gc,phases,start] GC(11) Summary Phase
[0.093s][info   ][gc,phases      ] GC(11) Summary Phase 0.011ms
[0.093s][info   ][gc,phases,start] GC(11) Adjust Roots
[0.093s][info   ][gc,phases      ] GC(11) Adjust Roots 0.125ms
[0.093s][info   ][gc,phases,start] GC(11) Compaction Phase
[0.094s][info   ][gc,phases      ] GC(11) Compaction Phase 1.132ms
[0.094s][info   ][gc,phases,start] GC(11) Post Compact
[0.094s][info   ][gc,phases      ] GC(11) Post Compact 0.054ms
[0.094s][info   ][gc,heap        ] GC(11) PSYoungGen: 14839K(29184K)->11329K(29184K) Eden: 14839K(14848K)->11329K(14848K) From: 0K(14336K)->0K(14336K)
[0.094s][info   ][gc,heap        ] GC(11) ParOldGen: 86976K(87552K)->87429K(87552K)
[0.094s][info   ][gc,metaspace   ] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.094s][info   ][gc             ] GC(11) Pause Full (Ergonomics) 99M->96M(114M) 3.584ms
[0.094s][info   ][gc,cpu         ] GC(11) User=0.00s Sys=0.00s Real=0.00s
[0.095s][info   ][gc,start       ] GC(12) Pause Full (Ergonomics)
[0.095s][info   ][gc,phases,start] GC(12) Marking Phase
[0.097s][info   ][gc,phases      ] GC(12) Marking Phase 2.000ms
[0.097s][info   ][gc,phases,start] GC(12) Summary Phase
[0.097s][info   ][gc,phases      ] GC(12) Summary Phase 0.008ms
[0.097s][info   ][gc,phases,start] GC(12) Adjust Roots
[0.097s][info   ][gc,phases      ] GC(12) Adjust Roots 0.136ms
[0.097s][info   ][gc,phases,start] GC(12) Compaction Phase
[0.097s][info   ][gc,phases      ] GC(12) Compaction Phase 0.325ms
[0.097s][info   ][gc,phases,start] GC(12) Post Compact
[0.097s][info   ][gc,phases      ] GC(12) Post Compact 0.041ms
[0.097s][info   ][gc,heap        ] GC(12) PSYoungGen: 14804K(29184K)->12796K(29184K) Eden: 14804K(14848K)->12796K(14848K) From: 0K(14336K)->0K(14336K)
[0.097s][info   ][gc,heap        ] GC(12) ParOldGen: 87429K(87552K)->87429K(87552K)
[0.097s][info   ][gc,metaspace   ] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.097s][info   ][gc             ] GC(12) Pause Full (Ergonomics) 99M->97M(114M) 2.665ms
[0.097s][info   ][gc,cpu         ] GC(12) User=0.01s Sys=0.00s Real=0.01s
[0.098s][info   ][gc,start       ] GC(13) Pause Full (Ergonomics)
[0.098s][info   ][gc,phases,start] GC(13) Marking Phase
[0.099s][info   ][gc,phases      ] GC(13) Marking Phase 0.880ms
[0.099s][info   ][gc,phases,start] GC(13) Summary Phase
[0.099s][info   ][gc,phases      ] GC(13) Summary Phase 0.008ms
[0.099s][info   ][gc,phases,start] GC(13) Adjust Roots
[0.099s][info   ][gc,phases      ] GC(13) Adjust Roots 0.103ms
[0.099s][info   ][gc,phases,start] GC(13) Compaction Phase
[0.101s][info   ][gc,phases      ] GC(13) Compaction Phase 1.874ms
[0.101s][info   ][gc,phases,start] GC(13) Post Compact
[0.101s][info   ][gc,phases      ] GC(13) Post Compact 0.047ms
[0.101s][info   ][gc,heap        ] GC(13) PSYoungGen: 14544K(29184K)->13373K(29184K) Eden: 14544K(14848K)->13373K(14848K) From: 0K(14336K)->0K(14336K)
[0.101s][info   ][gc,heap        ] GC(13) ParOldGen: 87429K(87552K)->87084K(87552K)
[0.101s][info   ][gc,metaspace   ] GC(13) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.101s][info   ][gc             ] GC(13) Pause Full (Ergonomics) 99M->98M(114M) 3.072ms
[0.101s][info   ][gc,cpu         ] GC(13) User=0.01s Sys=0.00s Real=0.00s
[0.101s][info   ][gc,start       ] GC(14) Pause Full (Ergonomics)
[0.101s][info   ][gc,phases,start] GC(14) Marking Phase
[0.102s][info   ][gc,phases      ] GC(14) Marking Phase 0.963ms
[0.102s][info   ][gc,phases,start] GC(14) Summary Phase
[0.102s][info   ][gc,phases      ] GC(14) Summary Phase 0.009ms
[0.102s][info   ][gc,phases,start] GC(14) Adjust Roots
[0.102s][info   ][gc,phases      ] GC(14) Adjust Roots 0.175ms
[0.102s][info   ][gc,phases,start] GC(14) Compaction Phase
[0.103s][info   ][gc,phases      ] GC(14) Compaction Phase 0.350ms
[0.103s][info   ][gc,phases,start] GC(14) Post Compact
[0.103s][info   ][gc,phases      ] GC(14) Post Compact 0.048ms
[0.103s][info   ][gc,heap        ] GC(14) PSYoungGen: 14724K(29184K)->13944K(29184K) Eden: 14724K(14848K)->13944K(14848K) From: 0K(14336K)->0K(14336K)
[0.103s][info   ][gc,heap        ] GC(14) ParOldGen: 87084K(87552K)->87084K(87552K)
[0.103s][info   ][gc,metaspace   ] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.103s][info   ][gc             ] GC(14) Pause Full (Ergonomics) 99M->98M(114M) 1.712ms
[0.103s][info   ][gc,cpu         ] GC(14) User=0.00s Sys=0.00s Real=0.00s
[0.103s][info   ][gc,start       ] GC(15) Pause Full (Ergonomics)
[0.103s][info   ][gc,phases,start] GC(15) Marking Phase
[0.105s][info   ][gc,phases      ] GC(15) Marking Phase 1.857ms
[0.105s][info   ][gc,phases,start] GC(15) Summary Phase
[0.105s][info   ][gc,phases      ] GC(15) Summary Phase 0.009ms
[0.105s][info   ][gc,phases,start] GC(15) Adjust Roots
[0.105s][info   ][gc,phases      ] GC(15) Adjust Roots 0.104ms
[0.105s][info   ][gc,phases,start] GC(15) Compaction Phase
[0.105s][info   ][gc,phases      ] GC(15) Compaction Phase 0.326ms
[0.105s][info   ][gc,phases,start] GC(15) Post Compact
[0.105s][info   ][gc,phases      ] GC(15) Post Compact 0.039ms
[0.105s][info   ][gc,heap        ] GC(15) PSYoungGen: 14610K(29184K)->14454K(29184K) Eden: 14610K(14848K)->14454K(14848K) From: 0K(14336K)->0K(14336K)
[0.105s][info   ][gc,heap        ] GC(15) ParOldGen: 87084K(87552K)->87084K(87552K)
[0.105s][info   ][gc,metaspace   ] GC(15) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.105s][info   ][gc             ] GC(15) Pause Full (Ergonomics) 99M->99M(114M) 2.439ms
[0.105s][info   ][gc,cpu         ] GC(15) User=0.00s Sys=0.00s Real=0.00s
[0.105s][info   ][gc,start       ] GC(16) Pause Full (Allocation Failure)
[0.105s][info   ][gc,phases,start] GC(16) Marking Phase
[0.106s][info   ][gc,phases      ] GC(16) Marking Phase 1.070ms
[0.107s][info   ][gc,phases,start] GC(16) Summary Phase
[0.107s][info   ][gc,phases      ] GC(16) Summary Phase 0.009ms
[0.107s][info   ][gc,phases,start] GC(16) Adjust Roots
[0.107s][info   ][gc,phases      ] GC(16) Adjust Roots 0.090ms
[0.107s][info   ][gc,phases,start] GC(16) Compaction Phase
[0.107s][info   ][gc,phases      ] GC(16) Compaction Phase 0.678ms
[0.107s][info   ][gc,phases,start] GC(16) Post Compact
[0.107s][info   ][gc,phases      ] GC(16) Post Compact 0.056ms
[0.107s][info   ][gc,heap        ] GC(16) PSYoungGen: 14454K(29184K)->14454K(29184K) Eden: 14454K(14848K)->14454K(14848K) From: 0K(14336K)->0K(14336K)
[0.107s][info   ][gc,heap        ] GC(16) ParOldGen: 87084K(87552K)->87082K(87552K)
[0.107s][info   ][gc,metaspace   ] GC(16) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.107s][info   ][gc             ] GC(16) Pause Full (Allocation Failure) 99M->99M(114M) 2.041ms
[0.107s][info   ][gc,cpu         ] GC(16) User=0.01s Sys=0.00s Real=0.01s
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at GCLogAnalysis.generateGarbage(GCLogAnalysis.java:42)
	at GCLogAnalysis.main(GCLogAnalysis.java:20)
[0.108s][info   ][gc,heap,exit   ] Heap
[0.108s][info   ][gc,heap,exit   ]  PSYoungGen      total 29184K, used 14811K [0x00000007fd580000, 0x0000000800000000, 0x0000000800000000)
[0.108s][info   ][gc,heap,exit   ]   eden space 14848K, 99% used [0x00000007fd580000,0x00000007fe3f6d70,0x00000007fe400000)
[0.108s][info   ][gc,heap,exit   ]   from space 14336K, 0% used [0x00000007fe400000,0x00000007fe400000,0x00000007ff200000)
[0.108s][info   ][gc,heap,exit   ]   to   space 14336K, 0% used [0x00000007ff200000,0x00000007ff200000,0x0000000800000000)
[0.108s][info   ][gc,heap,exit   ]  ParOldGen       total 87552K, used 87082K [0x00000007f8000000, 0x00000007fd580000, 0x00000007fd580000)
[0.108s][info   ][gc,heap,exit   ]   object space 87552K, 99% used [0x00000007f8000000,0x00000007fd50a8f0,0x00000007fd580000)
[0.108s][info   ][gc,heap,exit   ]  Metaspace       used 163K, committed 384K, reserved 1114112K
[0.108s][info   ][gc,heap,exit   ]   class space    used 7K, committed 128K, reserved 1048576K
```

接下来试试512m
```
java -XX:+UseParallelGC -Xms512m -Xmx512m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以发现GC次数大大增加了，吞吐量也降低了。27000多个对象，99次GC。
```
[0.293s][info   ][gc,phases      ] GC(35) Adjust Roots 0.149ms
[0.293s][info   ][gc,phases,start] GC(35) Compaction Phase
[0.298s][info   ][gc,phases      ] GC(35) Compaction Phase 5.147ms
[0.298s][info   ][gc,phases,start] GC(35) Post Compact
[0.298s][info   ][gc,phases      ] GC(35) Post Compact 0.154ms
[0.298s][info   ][gc,heap        ] GC(35) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.298s][info   ][gc,heap        ] GC(35) ParOldGen: 319789K(349696K)->323740K(349696K)
[0.298s][info   ][gc,metaspace   ] GC(35) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.298s][info   ][gc             ] GC(35) Pause Full (Ergonomics) 369M->316M(455M) 7.260ms
[0.298s][info   ][gc,cpu         ] GC(35) User=0.03s Sys=0.00s Real=0.01s
[0.301s][info   ][gc,start       ] GC(36) Pause Full (Ergonomics)
[0.301s][info   ][gc,phases,start] GC(36) Marking Phase
[0.303s][info   ][gc,phases      ] GC(36) Marking Phase 2.059ms
[0.303s][info   ][gc,phases,start] GC(36) Summary Phase
[0.303s][info   ][gc,phases      ] GC(36) Summary Phase 0.017ms
[0.303s][info   ][gc,phases,start] GC(36) Adjust Roots
[0.303s][info   ][gc,phases      ] GC(36) Adjust Roots 0.151ms
[0.303s][info   ][gc,phases,start] GC(36) Compaction Phase
[0.309s][info   ][gc,phases      ] GC(36) Compaction Phase 5.908ms
[0.309s][info   ][gc,phases,start] GC(36) Post Compact
[0.309s][info   ][gc,phases      ] GC(36) Post Compact 0.157ms
[0.310s][info   ][gc,heap        ] GC(36) PSYoungGen: 58212K(116736K)->0K(116736K) Eden: 58212K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.310s][info   ][gc,heap        ] GC(36) ParOldGen: 323740K(349696K)->324378K(349696K)
[0.310s][info   ][gc,metaspace   ] GC(36) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.310s][info   ][gc             ] GC(36) Pause Full (Ergonomics) 373M->316M(455M) 8.553ms
[0.310s][info   ][gc,cpu         ] GC(36) User=0.04s Sys=0.01s Real=0.01s
[0.313s][info   ][gc,start       ] GC(37) Pause Full (Ergonomics)
[0.313s][info   ][gc,phases,start] GC(37) Marking Phase
[0.315s][info   ][gc,phases      ] GC(37) Marking Phase 2.002ms
[0.315s][info   ][gc,phases,start] GC(37) Summary Phase
[0.315s][info   ][gc,phases      ] GC(37) Summary Phase 0.017ms
[0.315s][info   ][gc,phases,start] GC(37) Adjust Roots
[0.315s][info   ][gc,phases      ] GC(37) Adjust Roots 0.145ms
[0.315s][info   ][gc,phases,start] GC(37) Compaction Phase
[0.320s][info   ][gc,phases      ] GC(37) Compaction Phase 5.274ms
[0.320s][info   ][gc,phases,start] GC(37) Post Compact
[0.320s][info   ][gc,phases      ] GC(37) Post Compact 0.156ms
[0.320s][info   ][gc,heap        ] GC(37) PSYoungGen: 58330K(116736K)->0K(116736K) Eden: 58330K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.320s][info   ][gc,heap        ] GC(37) ParOldGen: 324378K(349696K)->330545K(349696K)
[0.321s][info   ][gc,metaspace   ] GC(37) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.321s][info   ][gc             ] GC(37) Pause Full (Ergonomics) 373M->322M(455M) 7.833ms
[0.321s][info   ][gc,cpu         ] GC(37) User=0.03s Sys=0.00s Real=0.00s
[0.324s][info   ][gc,start       ] GC(38) Pause Full (Ergonomics)
[0.324s][info   ][gc,phases,start] GC(38) Marking Phase
[0.326s][info   ][gc,phases      ] GC(38) Marking Phase 1.991ms
[0.326s][info   ][gc,phases,start] GC(38) Summary Phase
[0.326s][info   ][gc,phases      ] GC(38) Summary Phase 0.016ms
[0.326s][info   ][gc,phases,start] GC(38) Adjust Roots
[0.326s][info   ][gc,phases      ] GC(38) Adjust Roots 0.132ms
[0.326s][info   ][gc,phases,start] GC(38) Compaction Phase
[0.331s][info   ][gc,phases      ] GC(38) Compaction Phase 4.801ms
[0.331s][info   ][gc,phases,start] GC(38) Post Compact
[0.331s][info   ][gc,phases      ] GC(38) Post Compact 0.196ms
[0.331s][info   ][gc,heap        ] GC(38) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.331s][info   ][gc,heap        ] GC(38) ParOldGen: 330545K(349696K)->327226K(349696K)
[0.331s][info   ][gc,metaspace   ] GC(38) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.331s][info   ][gc             ] GC(38) Pause Full (Ergonomics) 380M->319M(455M) 7.330ms
[0.331s][info   ][gc,cpu         ] GC(38) User=0.03s Sys=0.00s Real=0.00s
[0.334s][info   ][gc,start       ] GC(39) Pause Full (Ergonomics)
[0.334s][info   ][gc,phases,start] GC(39) Marking Phase
[0.336s][info   ][gc,phases      ] GC(39) Marking Phase 1.339ms
[0.336s][info   ][gc,phases,start] GC(39) Summary Phase
[0.336s][info   ][gc,phases      ] GC(39) Summary Phase 0.017ms
[0.336s][info   ][gc,phases,start] GC(39) Adjust Roots
[0.336s][info   ][gc,phases      ] GC(39) Adjust Roots 0.147ms
[0.336s][info   ][gc,phases,start] GC(39) Compaction Phase
[0.342s][info   ][gc,phases      ] GC(39) Compaction Phase 5.525ms
[0.342s][info   ][gc,phases,start] GC(39) Post Compact
[0.342s][info   ][gc,phases      ] GC(39) Post Compact 0.159ms
[0.342s][info   ][gc,heap        ] GC(39) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.342s][info   ][gc,heap        ] GC(39) ParOldGen: 327226K(349696K)->328568K(349696K)
[0.342s][info   ][gc,metaspace   ] GC(39) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.342s][info   ][gc             ] GC(39) Pause Full (Ergonomics) 377M->320M(455M) 7.366ms
[0.342s][info   ][gc,cpu         ] GC(39) User=0.03s Sys=0.00s Real=0.00s
[0.345s][info   ][gc,start       ] GC(40) Pause Full (Ergonomics)
[0.345s][info   ][gc,phases,start] GC(40) Marking Phase
[0.347s][info   ][gc,phases      ] GC(40) Marking Phase 1.575ms
[0.347s][info   ][gc,phases,start] GC(40) Summary Phase
[0.347s][info   ][gc,phases      ] GC(40) Summary Phase 0.024ms
[0.347s][info   ][gc,phases,start] GC(40) Adjust Roots
[0.347s][info   ][gc,phases      ] GC(40) Adjust Roots 0.156ms
[0.347s][info   ][gc,phases,start] GC(40) Compaction Phase
[0.352s][info   ][gc,phases      ] GC(40) Compaction Phase 5.421ms
[0.352s][info   ][gc,phases,start] GC(40) Post Compact
[0.353s][info   ][gc,phases      ] GC(40) Post Compact 0.152ms
[0.353s][info   ][gc,heap        ] GC(40) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.353s][info   ][gc,heap        ] GC(40) ParOldGen: 328568K(349696K)->327872K(349696K)
[0.353s][info   ][gc,metaspace   ] GC(40) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.353s][info   ][gc             ] GC(40) Pause Full (Ergonomics) 378M->320M(455M) 7.517ms
[0.353s][info   ][gc,cpu         ] GC(40) User=0.03s Sys=0.00s Real=0.01s
[0.356s][info   ][gc,start       ] GC(41) Pause Full (Ergonomics)
[0.356s][info   ][gc,phases,start] GC(41) Marking Phase
[0.358s][info   ][gc,phases      ] GC(41) Marking Phase 2.070ms
[0.358s][info   ][gc,phases,start] GC(41) Summary Phase
[0.358s][info   ][gc,phases      ] GC(41) Summary Phase 0.015ms
[0.358s][info   ][gc,phases,start] GC(41) Adjust Roots
[0.358s][info   ][gc,phases      ] GC(41) Adjust Roots 0.139ms
[0.358s][info   ][gc,phases,start] GC(41) Compaction Phase
[0.363s][info   ][gc,phases      ] GC(41) Compaction Phase 5.207ms
[0.363s][info   ][gc,phases,start] GC(41) Post Compact
[0.363s][info   ][gc,phases      ] GC(41) Post Compact 0.153ms
[0.363s][info   ][gc,heap        ] GC(41) PSYoungGen: 58603K(116736K)->0K(116736K) Eden: 58603K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.363s][info   ][gc,heap        ] GC(41) ParOldGen: 327872K(349696K)->325933K(349696K)
[0.363s][info   ][gc,metaspace   ] GC(41) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.363s][info   ][gc             ] GC(41) Pause Full (Ergonomics) 377M->318M(455M) 7.759ms
[0.364s][info   ][gc,cpu         ] GC(41) User=0.04s Sys=0.00s Real=0.01s
[0.366s][info   ][gc,start       ] GC(42) Pause Full (Ergonomics)
[0.366s][info   ][gc,phases,start] GC(42) Marking Phase
[0.368s][info   ][gc,phases      ] GC(42) Marking Phase 1.962ms
[0.368s][info   ][gc,phases,start] GC(42) Summary Phase
[0.368s][info   ][gc,phases      ] GC(42) Summary Phase 0.013ms
[0.368s][info   ][gc,phases,start] GC(42) Adjust Roots
[0.369s][info   ][gc,phases      ] GC(42) Adjust Roots 0.159ms
[0.369s][info   ][gc,phases,start] GC(42) Compaction Phase
[0.375s][info   ][gc,phases      ] GC(42) Compaction Phase 6.241ms
[0.375s][info   ][gc,phases,start] GC(42) Post Compact
[0.375s][info   ][gc,phases      ] GC(42) Post Compact 0.169ms
[0.375s][info   ][gc,heap        ] GC(42) PSYoungGen: 58777K(116736K)->0K(116736K) Eden: 58777K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.375s][info   ][gc,heap        ] GC(42) ParOldGen: 325933K(349696K)->329364K(349696K)
[0.375s][info   ][gc,metaspace   ] GC(42) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.375s][info   ][gc             ] GC(42) Pause Full (Ergonomics) 375M->321M(455M) 8.773ms
[0.375s][info   ][gc,cpu         ] GC(42) User=0.03s Sys=0.00s Real=0.01s
[0.378s][info   ][gc,start       ] GC(43) Pause Full (Ergonomics)
[0.378s][info   ][gc,phases,start] GC(43) Marking Phase
[0.380s][info   ][gc,phases      ] GC(43) Marking Phase 2.313ms
[0.380s][info   ][gc,phases,start] GC(43) Summary Phase
[0.380s][info   ][gc,phases      ] GC(43) Summary Phase 0.017ms
[0.380s][info   ][gc,phases,start] GC(43) Adjust Roots
[0.380s][info   ][gc,phases      ] GC(43) Adjust Roots 0.177ms
[0.380s][info   ][gc,phases,start] GC(43) Compaction Phase
[0.385s][info   ][gc,phases      ] GC(43) Compaction Phase 5.140ms
[0.386s][info   ][gc,phases,start] GC(43) Post Compact
[0.386s][info   ][gc,phases      ] GC(43) Post Compact 0.163ms
[0.386s][info   ][gc,heap        ] GC(43) PSYoungGen: 58784K(116736K)->0K(116736K) Eden: 58784K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.386s][info   ][gc,heap        ] GC(43) ParOldGen: 329364K(349696K)->326673K(349696K)
[0.386s][info   ][gc,metaspace   ] GC(43) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.386s][info   ][gc             ] GC(43) Pause Full (Ergonomics) 379M->319M(455M) 8.001ms
[0.386s][info   ][gc,cpu         ] GC(43) User=0.04s Sys=0.00s Real=0.01s
[0.389s][info   ][gc,start       ] GC(44) Pause Full (Ergonomics)
[0.389s][info   ][gc,phases,start] GC(44) Marking Phase
[0.390s][info   ][gc,phases      ] GC(44) Marking Phase 1.584ms
[0.390s][info   ][gc,phases,start] GC(44) Summary Phase
[0.390s][info   ][gc,phases      ] GC(44) Summary Phase 0.018ms
[0.390s][info   ][gc,phases,start] GC(44) Adjust Roots
[0.391s][info   ][gc,phases      ] GC(44) Adjust Roots 0.324ms
[0.391s][info   ][gc,phases,start] GC(44) Compaction Phase
[0.396s][info   ][gc,phases      ] GC(44) Compaction Phase 5.119ms
[0.396s][info   ][gc,phases,start] GC(44) Post Compact
[0.396s][info   ][gc,phases      ] GC(44) Post Compact 0.165ms
[0.396s][info   ][gc,heap        ] GC(44) PSYoungGen: 58849K(116736K)->0K(116736K) Eden: 58849K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.396s][info   ][gc,heap        ] GC(44) ParOldGen: 326673K(349696K)->323222K(349696K)
[0.396s][info   ][gc,metaspace   ] GC(44) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.396s][info   ][gc             ] GC(44) Pause Full (Ergonomics) 376M->315M(455M) 7.410ms
[0.396s][info   ][gc,cpu         ] GC(44) User=0.04s Sys=0.01s Real=0.01s
[0.398s][info   ][gc,start       ] GC(45) Pause Full (Ergonomics)
[0.398s][info   ][gc,phases,start] GC(45) Marking Phase
[0.400s][info   ][gc,phases      ] GC(45) Marking Phase 1.678ms
[0.400s][info   ][gc,phases,start] GC(45) Summary Phase
[0.400s][info   ][gc,phases      ] GC(45) Summary Phase 0.016ms
[0.400s][info   ][gc,phases,start] GC(45) Adjust Roots
[0.400s][info   ][gc,phases      ] GC(45) Adjust Roots 0.132ms
[0.400s][info   ][gc,phases,start] GC(45) Compaction Phase
[0.406s][info   ][gc,phases      ] GC(45) Compaction Phase 5.856ms
[0.406s][info   ][gc,phases,start] GC(45) Post Compact
[0.406s][info   ][gc,phases      ] GC(45) Post Compact 0.160ms
[0.406s][info   ][gc,heap        ] GC(45) PSYoungGen: 58674K(116736K)->0K(116736K) Eden: 58674K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.406s][info   ][gc,heap        ] GC(45) ParOldGen: 323222K(349696K)->330037K(349696K)
[0.406s][info   ][gc,metaspace   ] GC(45) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.406s][info   ][gc             ] GC(45) Pause Full (Ergonomics) 372M->322M(455M) 8.008ms
[0.406s][info   ][gc,cpu         ] GC(45) User=0.03s Sys=0.00s Real=0.01s
[0.409s][info   ][gc,start       ] GC(46) Pause Full (Ergonomics)
[0.409s][info   ][gc,phases,start] GC(46) Marking Phase
[0.413s][info   ][gc,phases      ] GC(46) Marking Phase 3.590ms
[0.413s][info   ][gc,phases,start] GC(46) Summary Phase
[0.413s][info   ][gc,phases      ] GC(46) Summary Phase 0.015ms
[0.413s][info   ][gc,phases,start] GC(46) Adjust Roots
[0.413s][info   ][gc,phases      ] GC(46) Adjust Roots 0.170ms
[0.413s][info   ][gc,phases,start] GC(46) Compaction Phase
[0.418s][info   ][gc,phases      ] GC(46) Compaction Phase 5.598ms
[0.418s][info   ][gc,phases,start] GC(46) Post Compact
[0.419s][info   ][gc,phases      ] GC(46) Post Compact 0.166ms
[0.419s][info   ][gc,heap        ] GC(46) PSYoungGen: 58618K(116736K)->0K(116736K) Eden: 58618K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.419s][info   ][gc,heap        ] GC(46) ParOldGen: 330037K(349696K)->330388K(349696K)
[0.419s][info   ][gc,metaspace   ] GC(46) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.419s][info   ][gc             ] GC(46) Pause Full (Ergonomics) 379M->322M(455M) 9.759ms
[0.419s][info   ][gc,cpu         ] GC(46) User=0.03s Sys=0.00s Real=0.01s
[0.422s][info   ][gc,start       ] GC(47) Pause Full (Ergonomics)
[0.422s][info   ][gc,phases,start] GC(47) Marking Phase
[0.424s][info   ][gc,phases      ] GC(47) Marking Phase 1.691ms
[0.424s][info   ][gc,phases,start] GC(47) Summary Phase
[0.424s][info   ][gc,phases      ] GC(47) Summary Phase 0.026ms
[0.424s][info   ][gc,phases,start] GC(47) Adjust Roots
[0.424s][info   ][gc,phases      ] GC(47) Adjust Roots 0.139ms
[0.424s][info   ][gc,phases,start] GC(47) Compaction Phase
[0.429s][info   ][gc,phases      ] GC(47) Compaction Phase 4.712ms
[0.429s][info   ][gc,phases,start] GC(47) Post Compact
[0.429s][info   ][gc,phases      ] GC(47) Post Compact 0.166ms
[0.429s][info   ][gc,heap        ] GC(47) PSYoungGen: 58864K(116736K)->0K(116736K) Eden: 58864K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.429s][info   ][gc,heap        ] GC(47) ParOldGen: 330388K(349696K)->328098K(349696K)
[0.429s][info   ][gc,metaspace   ] GC(47) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.429s][info   ][gc             ] GC(47) Pause Full (Ergonomics) 380M->320M(455M) 6.936ms
[0.429s][info   ][gc,cpu         ] GC(47) User=0.04s Sys=0.00s Real=0.01s
[0.432s][info   ][gc,start       ] GC(48) Pause Full (Ergonomics)
[0.432s][info   ][gc,phases,start] GC(48) Marking Phase
[0.434s][info   ][gc,phases      ] GC(48) Marking Phase 1.833ms
[0.434s][info   ][gc,phases,start] GC(48) Summary Phase
[0.434s][info   ][gc,phases      ] GC(48) Summary Phase 0.018ms
[0.434s][info   ][gc,phases,start] GC(48) Adjust Roots
[0.434s][info   ][gc,phases      ] GC(48) Adjust Roots 0.134ms
[0.434s][info   ][gc,phases,start] GC(48) Compaction Phase
[0.439s][info   ][gc,phases      ] GC(48) Compaction Phase 5.470ms
[0.439s][info   ][gc,phases,start] GC(48) Post Compact
[0.440s][info   ][gc,phases      ] GC(48) Post Compact 0.162ms
[0.440s][info   ][gc,heap        ] GC(48) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.440s][info   ][gc,heap        ] GC(48) ParOldGen: 328098K(349696K)->329754K(349696K)
[0.440s][info   ][gc,metaspace   ] GC(48) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.440s][info   ][gc             ] GC(48) Pause Full (Ergonomics) 377M->322M(455M) 7.822ms
[0.440s][info   ][gc,cpu         ] GC(48) User=0.03s Sys=0.00s Real=0.01s
[0.443s][info   ][gc,start       ] GC(49) Pause Full (Ergonomics)
[0.443s][info   ][gc,phases,start] GC(49) Marking Phase
[0.445s][info   ][gc,phases      ] GC(49) Marking Phase 2.241ms
[0.445s][info   ][gc,phases,start] GC(49) Summary Phase
[0.445s][info   ][gc,phases      ] GC(49) Summary Phase 0.014ms
[0.445s][info   ][gc,phases,start] GC(49) Adjust Roots
[0.445s][info   ][gc,phases      ] GC(49) Adjust Roots 0.148ms
[0.445s][info   ][gc,phases,start] GC(49) Compaction Phase
[0.451s][info   ][gc,phases      ] GC(49) Compaction Phase 5.532ms
[0.451s][info   ][gc,phases,start] GC(49) Post Compact
[0.451s][info   ][gc,phases      ] GC(49) Post Compact 0.171ms
[0.451s][info   ][gc,heap        ] GC(49) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.451s][info   ][gc,heap        ] GC(49) ParOldGen: 329754K(349696K)->329787K(349696K)
[0.451s][info   ][gc,metaspace   ] GC(49) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.451s][info   ][gc             ] GC(49) Pause Full (Ergonomics) 379M->322M(455M) 8.304ms
[0.451s][info   ][gc,cpu         ] GC(49) User=0.03s Sys=0.01s Real=0.00s
[0.454s][info   ][gc,start       ] GC(50) Pause Full (Ergonomics)
[0.454s][info   ][gc,phases,start] GC(50) Marking Phase
[0.455s][info   ][gc,phases      ] GC(50) Marking Phase 1.630ms
[0.455s][info   ][gc,phases,start] GC(50) Summary Phase
[0.455s][info   ][gc,phases      ] GC(50) Summary Phase 0.014ms
[0.455s][info   ][gc,phases,start] GC(50) Adjust Roots
[0.456s][info   ][gc,phases      ] GC(50) Adjust Roots 0.144ms
[0.456s][info   ][gc,phases,start] GC(50) Compaction Phase
[0.461s][info   ][gc,phases      ] GC(50) Compaction Phase 5.108ms
[0.461s][info   ][gc,phases,start] GC(50) Post Compact
[0.461s][info   ][gc,phases      ] GC(50) Post Compact 0.147ms
[0.461s][info   ][gc,heap        ] GC(50) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.461s][info   ][gc,heap        ] GC(50) ParOldGen: 329787K(349696K)->333951K(349696K)
[0.461s][info   ][gc,metaspace   ] GC(50) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.461s][info   ][gc             ] GC(50) Pause Full (Ergonomics) 379M->326M(455M) 7.241ms
[0.461s][info   ][gc,cpu         ] GC(50) User=0.04s Sys=0.00s Real=0.00s
[0.464s][info   ][gc,start       ] GC(51) Pause Full (Ergonomics)
[0.464s][info   ][gc,phases,start] GC(51) Marking Phase
[0.465s][info   ][gc,phases      ] GC(51) Marking Phase 1.691ms
[0.465s][info   ][gc,phases,start] GC(51) Summary Phase
[0.465s][info   ][gc,phases      ] GC(51) Summary Phase 0.014ms
[0.465s][info   ][gc,phases,start] GC(51) Adjust Roots
[0.466s][info   ][gc,phases      ] GC(51) Adjust Roots 0.133ms
[0.466s][info   ][gc,phases,start] GC(51) Compaction Phase
[0.471s][info   ][gc,phases      ] GC(51) Compaction Phase 5.033ms
[0.471s][info   ][gc,phases,start] GC(51) Post Compact
[0.471s][info   ][gc,phases      ] GC(51) Post Compact 0.160ms
[0.471s][info   ][gc,heap        ] GC(51) PSYoungGen: 58617K(116736K)->0K(116736K) Eden: 58617K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.471s][info   ][gc,heap        ] GC(51) ParOldGen: 333951K(349696K)->333136K(349696K)
[0.471s][info   ][gc,metaspace   ] GC(51) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.471s][info   ][gc             ] GC(51) Pause Full (Ergonomics) 383M->325M(455M) 7.240ms
[0.471s][info   ][gc,cpu         ] GC(51) User=0.03s Sys=0.00s Real=0.00s
[0.475s][info   ][gc,start       ] GC(52) Pause Full (Ergonomics)
[0.475s][info   ][gc,phases,start] GC(52) Marking Phase
[0.476s][info   ][gc,phases      ] GC(52) Marking Phase 1.368ms
[0.476s][info   ][gc,phases,start] GC(52) Summary Phase
[0.476s][info   ][gc,phases      ] GC(52) Summary Phase 0.015ms
[0.476s][info   ][gc,phases,start] GC(52) Adjust Roots
[0.476s][info   ][gc,phases      ] GC(52) Adjust Roots 0.167ms
[0.476s][info   ][gc,phases,start] GC(52) Compaction Phase
[0.483s][info   ][gc,phases      ] GC(52) Compaction Phase 6.434ms
[0.483s][info   ][gc,phases,start] GC(52) Post Compact
[0.483s][info   ][gc,phases      ] GC(52) Post Compact 0.160ms
[0.483s][info   ][gc,heap        ] GC(52) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.483s][info   ][gc,heap        ] GC(52) ParOldGen: 333136K(349696K)->335627K(349696K)
[0.483s][info   ][gc,metaspace   ] GC(52) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.483s][info   ][gc             ] GC(52) Pause Full (Ergonomics) 382M->327M(455M) 8.342ms
[0.483s][info   ][gc,cpu         ] GC(52) User=0.04s Sys=0.00s Real=0.01s
[0.486s][info   ][gc,start       ] GC(53) Pause Full (Ergonomics)
[0.486s][info   ][gc,phases,start] GC(53) Marking Phase
[0.487s][info   ][gc,phases      ] GC(53) Marking Phase 1.415ms
[0.487s][info   ][gc,phases,start] GC(53) Summary Phase
[0.487s][info   ][gc,phases      ] GC(53) Summary Phase 0.013ms
[0.487s][info   ][gc,phases,start] GC(53) Adjust Roots
[0.487s][info   ][gc,phases      ] GC(53) Adjust Roots 0.125ms
[0.487s][info   ][gc,phases,start] GC(53) Compaction Phase
[0.493s][info   ][gc,phases      ] GC(53) Compaction Phase 5.581ms
[0.493s][info   ][gc,phases,start] GC(53) Post Compact
[0.493s][info   ][gc,phases      ] GC(53) Post Compact 0.169ms
[0.493s][info   ][gc,heap        ] GC(53) PSYoungGen: 58857K(116736K)->0K(116736K) Eden: 58857K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.493s][info   ][gc,heap        ] GC(53) ParOldGen: 335627K(349696K)->336226K(349696K)
[0.493s][info   ][gc,metaspace   ] GC(53) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.493s][info   ][gc             ] GC(53) Pause Full (Ergonomics) 385M->328M(455M) 7.470ms
[0.493s][info   ][gc,cpu         ] GC(53) User=0.03s Sys=0.00s Real=0.01s
[0.496s][info   ][gc,start       ] GC(54) Pause Full (Ergonomics)
[0.496s][info   ][gc,phases,start] GC(54) Marking Phase
[0.498s][info   ][gc,phases      ] GC(54) Marking Phase 2.113ms
[0.498s][info   ][gc,phases,start] GC(54) Summary Phase
[0.498s][info   ][gc,phases      ] GC(54) Summary Phase 0.017ms
[0.498s][info   ][gc,phases,start] GC(54) Adjust Roots
[0.499s][info   ][gc,phases      ] GC(54) Adjust Roots 0.131ms
[0.499s][info   ][gc,phases,start] GC(54) Compaction Phase
[0.504s][info   ][gc,phases      ] GC(54) Compaction Phase 5.501ms
[0.504s][info   ][gc,phases,start] GC(54) Post Compact
[0.504s][info   ][gc,phases      ] GC(54) Post Compact 0.164ms
[0.504s][info   ][gc,heap        ] GC(54) PSYoungGen: 58838K(116736K)->0K(116736K) Eden: 58838K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.504s][info   ][gc,heap        ] GC(54) ParOldGen: 336226K(349696K)->333074K(349696K)
[0.504s][info   ][gc,metaspace   ] GC(54) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.504s][info   ][gc             ] GC(54) Pause Full (Ergonomics) 385M->325M(455M) 8.120ms
[0.504s][info   ][gc,cpu         ] GC(54) User=0.03s Sys=0.01s Real=0.01s
[0.507s][info   ][gc,start       ] GC(55) Pause Full (Ergonomics)
[0.507s][info   ][gc,phases,start] GC(55) Marking Phase
[0.509s][info   ][gc,phases      ] GC(55) Marking Phase 1.806ms
[0.509s][info   ][gc,phases,start] GC(55) Summary Phase
[0.509s][info   ][gc,phases      ] GC(55) Summary Phase 0.017ms
[0.509s][info   ][gc,phases,start] GC(55) Adjust Roots
[0.509s][info   ][gc,phases      ] GC(55) Adjust Roots 0.145ms
[0.509s][info   ][gc,phases,start] GC(55) Compaction Phase
[0.516s][info   ][gc,phases      ] GC(55) Compaction Phase 6.341ms
[0.516s][info   ][gc,phases,start] GC(55) Post Compact
[0.516s][info   ][gc,phases      ] GC(55) Post Compact 0.176ms
[0.516s][info   ][gc,heap        ] GC(55) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.516s][info   ][gc,heap        ] GC(55) ParOldGen: 333074K(349696K)->336203K(349696K)
[0.516s][info   ][gc,metaspace   ] GC(55) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.516s][info   ][gc             ] GC(55) Pause Full (Ergonomics) 382M->328M(455M) 8.700ms
[0.516s][info   ][gc,cpu         ] GC(55) User=0.04s Sys=0.00s Real=0.01s
[0.520s][info   ][gc,start       ] GC(56) Pause Full (Ergonomics)
[0.520s][info   ][gc,phases,start] GC(56) Marking Phase
[0.522s][info   ][gc,phases      ] GC(56) Marking Phase 2.003ms
[0.522s][info   ][gc,phases,start] GC(56) Summary Phase
[0.522s][info   ][gc,phases      ] GC(56) Summary Phase 0.013ms
[0.522s][info   ][gc,phases,start] GC(56) Adjust Roots
[0.523s][info   ][gc,phases      ] GC(56) Adjust Roots 0.145ms
[0.523s][info   ][gc,phases,start] GC(56) Compaction Phase
[0.527s][info   ][gc,phases      ] GC(56) Compaction Phase 4.909ms
[0.527s][info   ][gc,phases,start] GC(56) Post Compact
[0.528s][info   ][gc,phases      ] GC(56) Post Compact 0.170ms
[0.528s][info   ][gc,heap        ] GC(56) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.528s][info   ][gc,heap        ] GC(56) ParOldGen: 336203K(349696K)->333268K(349696K)
[0.528s][info   ][gc,metaspace   ] GC(56) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.528s][info   ][gc             ] GC(56) Pause Full (Ergonomics) 385M->325M(455M) 7.453ms
[0.528s][info   ][gc,cpu         ] GC(56) User=0.03s Sys=0.00s Real=0.01s
[0.531s][info   ][gc,start       ] GC(57) Pause Full (Ergonomics)
[0.531s][info   ][gc,phases,start] GC(57) Marking Phase
[0.533s][info   ][gc,phases      ] GC(57) Marking Phase 2.016ms
[0.533s][info   ][gc,phases,start] GC(57) Summary Phase
[0.533s][info   ][gc,phases      ] GC(57) Summary Phase 0.014ms
[0.533s][info   ][gc,phases,start] GC(57) Adjust Roots
[0.533s][info   ][gc,phases      ] GC(57) Adjust Roots 0.382ms
[0.534s][info   ][gc,phases,start] GC(57) Compaction Phase
[0.540s][info   ][gc,phases      ] GC(57) Compaction Phase 6.359ms
[0.540s][info   ][gc,phases,start] GC(57) Post Compact
[0.540s][info   ][gc,phases      ] GC(57) Post Compact 0.189ms
[0.540s][info   ][gc,heap        ] GC(57) PSYoungGen: 58677K(116736K)->0K(116736K) Eden: 58677K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.540s][info   ][gc,heap        ] GC(57) ParOldGen: 333268K(349696K)->330756K(349696K)
[0.540s][info   ][gc,metaspace   ] GC(57) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.540s][info   ][gc             ] GC(57) Pause Full (Ergonomics) 382M->323M(455M) 9.153ms
[0.540s][info   ][gc,cpu         ] GC(57) User=0.03s Sys=0.00s Real=0.01s
[0.543s][info   ][gc,start       ] GC(58) Pause Full (Ergonomics)
[0.543s][info   ][gc,phases,start] GC(58) Marking Phase
[0.545s][info   ][gc,phases      ] GC(58) Marking Phase 1.607ms
[0.545s][info   ][gc,phases,start] GC(58) Summary Phase
[0.545s][info   ][gc,phases      ] GC(58) Summary Phase 0.014ms
[0.545s][info   ][gc,phases,start] GC(58) Adjust Roots
[0.545s][info   ][gc,phases      ] GC(58) Adjust Roots 0.199ms
[0.545s][info   ][gc,phases,start] GC(58) Compaction Phase
[0.550s][info   ][gc,phases      ] GC(58) Compaction Phase 5.526ms
[0.550s][info   ][gc,phases,start] GC(58) Post Compact
[0.551s][info   ][gc,phases      ] GC(58) Post Compact 0.183ms
[0.551s][info   ][gc,heap        ] GC(58) PSYoungGen: 58547K(116736K)->0K(116736K) Eden: 58547K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.551s][info   ][gc,heap        ] GC(58) ParOldGen: 330756K(349696K)->331991K(349696K)
[0.551s][info   ][gc,metaspace   ] GC(58) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.551s][info   ][gc             ] GC(58) Pause Full (Ergonomics) 380M->324M(455M) 7.739ms
[0.551s][info   ][gc,cpu         ] GC(58) User=0.03s Sys=0.01s Real=0.00s
[0.553s][info   ][gc,start       ] GC(59) Pause Full (Ergonomics)
[0.553s][info   ][gc,phases,start] GC(59) Marking Phase
[0.555s][info   ][gc,phases      ] GC(59) Marking Phase 1.991ms
[0.555s][info   ][gc,phases,start] GC(59) Summary Phase
[0.555s][info   ][gc,phases      ] GC(59) Summary Phase 0.016ms
[0.555s][info   ][gc,phases,start] GC(59) Adjust Roots
[0.556s][info   ][gc,phases      ] GC(59) Adjust Roots 0.163ms
[0.556s][info   ][gc,phases,start] GC(59) Compaction Phase
[0.561s][info   ][gc,phases      ] GC(59) Compaction Phase 5.049ms
[0.561s][info   ][gc,phases,start] GC(59) Post Compact
[0.561s][info   ][gc,phases      ] GC(59) Post Compact 0.173ms
[0.561s][info   ][gc,heap        ] GC(59) PSYoungGen: 58488K(116736K)->0K(116736K) Eden: 58488K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.561s][info   ][gc,heap        ] GC(59) ParOldGen: 331991K(349696K)->330268K(349696K)
[0.561s][info   ][gc,metaspace   ] GC(59) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.561s][info   ][gc             ] GC(59) Pause Full (Ergonomics) 381M->322M(455M) 7.630ms
[0.561s][info   ][gc,cpu         ] GC(59) User=0.03s Sys=0.00s Real=0.00s
[0.564s][info   ][gc,start       ] GC(60) Pause Full (Ergonomics)
[0.564s][info   ][gc,phases,start] GC(60) Marking Phase
[0.566s][info   ][gc,phases      ] GC(60) Marking Phase 1.843ms
[0.566s][info   ][gc,phases,start] GC(60) Summary Phase
[0.566s][info   ][gc,phases      ] GC(60) Summary Phase 0.013ms
[0.566s][info   ][gc,phases,start] GC(60) Adjust Roots
[0.566s][info   ][gc,phases      ] GC(60) Adjust Roots 0.191ms
[0.566s][info   ][gc,phases,start] GC(60) Compaction Phase
[0.573s][info   ][gc,phases      ] GC(60) Compaction Phase 6.442ms
[0.573s][info   ][gc,phases,start] GC(60) Post Compact
[0.573s][info   ][gc,phases      ] GC(60) Post Compact 0.171ms
[0.573s][info   ][gc,heap        ] GC(60) PSYoungGen: 58733K(116736K)->0K(116736K) Eden: 58733K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.573s][info   ][gc,heap        ] GC(60) ParOldGen: 330268K(349696K)->332936K(349696K)
[0.573s][info   ][gc,metaspace   ] GC(60) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.573s][info   ][gc             ] GC(60) Pause Full (Ergonomics) 379M->325M(455M) 8.837ms
[0.573s][info   ][gc,cpu         ] GC(60) User=0.03s Sys=0.00s Real=0.01s
[0.576s][info   ][gc,start       ] GC(61) Pause Full (Ergonomics)
[0.576s][info   ][gc,phases,start] GC(61) Marking Phase
[0.578s][info   ][gc,phases      ] GC(61) Marking Phase 2.161ms
[0.578s][info   ][gc,phases,start] GC(61) Summary Phase
[0.578s][info   ][gc,phases      ] GC(61) Summary Phase 0.014ms
[0.578s][info   ][gc,phases,start] GC(61) Adjust Roots
[0.578s][info   ][gc,phases      ] GC(61) Adjust Roots 0.146ms
[0.578s][info   ][gc,phases,start] GC(61) Compaction Phase
[0.584s][info   ][gc,phases      ] GC(61) Compaction Phase 5.470ms
[0.584s][info   ][gc,phases,start] GC(61) Post Compact
[0.584s][info   ][gc,phases      ] GC(61) Post Compact 0.166ms
[0.584s][info   ][gc,heap        ] GC(61) PSYoungGen: 58865K(116736K)->0K(116736K) Eden: 58865K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.584s][info   ][gc,heap        ] GC(61) ParOldGen: 332936K(349696K)->330348K(349696K)
[0.584s][info   ][gc,metaspace   ] GC(61) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.584s][info   ][gc             ] GC(61) Pause Full (Ergonomics) 382M->322M(455M) 8.180ms
[0.584s][info   ][gc,cpu         ] GC(61) User=0.04s Sys=0.00s Real=0.01s
[0.586s][info   ][gc,start       ] GC(62) Pause Full (Ergonomics)
[0.587s][info   ][gc,phases,start] GC(62) Marking Phase
[0.589s][info   ][gc,phases      ] GC(62) Marking Phase 2.143ms
[0.589s][info   ][gc,phases,start] GC(62) Summary Phase
[0.589s][info   ][gc,phases      ] GC(62) Summary Phase 0.024ms
[0.589s][info   ][gc,phases,start] GC(62) Adjust Roots
[0.589s][info   ][gc,phases      ] GC(62) Adjust Roots 0.143ms
[0.589s][info   ][gc,phases,start] GC(62) Compaction Phase
[0.594s][info   ][gc,phases      ] GC(62) Compaction Phase 4.937ms
[0.594s][info   ][gc,phases,start] GC(62) Post Compact
[0.594s][info   ][gc,phases      ] GC(62) Post Compact 0.176ms
[0.594s][info   ][gc,heap        ] GC(62) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.594s][info   ][gc,heap        ] GC(62) ParOldGen: 330348K(349696K)->327859K(349696K)
[0.594s][info   ][gc,metaspace   ] GC(62) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.594s][info   ][gc             ] GC(62) Pause Full (Ergonomics) 380M->320M(455M) 7.643ms
[0.594s][info   ][gc,cpu         ] GC(62) User=0.03s Sys=0.00s Real=0.01s
[0.597s][info   ][gc,start       ] GC(63) Pause Full (Ergonomics)
[0.597s][info   ][gc,phases,start] GC(63) Marking Phase
[0.599s][info   ][gc,phases      ] GC(63) Marking Phase 1.793ms
[0.599s][info   ][gc,phases,start] GC(63) Summary Phase
[0.599s][info   ][gc,phases      ] GC(63) Summary Phase 0.024ms
[0.599s][info   ][gc,phases,start] GC(63) Adjust Roots
[0.599s][info   ][gc,phases      ] GC(63) Adjust Roots 0.176ms
[0.599s][info   ][gc,phases,start] GC(63) Compaction Phase
[0.605s][info   ][gc,phases      ] GC(63) Compaction Phase 6.124ms
[0.605s][info   ][gc,phases,start] GC(63) Post Compact
[0.605s][info   ][gc,phases      ] GC(63) Post Compact 0.158ms
[0.605s][info   ][gc,heap        ] GC(63) PSYoungGen: 58766K(116736K)->0K(116736K) Eden: 58766K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.605s][info   ][gc,heap        ] GC(63) ParOldGen: 327859K(349696K)->330218K(349696K)
[0.605s][info   ][gc,metaspace   ] GC(63) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.605s][info   ][gc             ] GC(63) Pause Full (Ergonomics) 377M->322M(455M) 8.541ms
[0.605s][info   ][gc,cpu         ] GC(63) User=0.03s Sys=0.01s Real=0.01s
[0.608s][info   ][gc,start       ] GC(64) Pause Full (Ergonomics)
[0.608s][info   ][gc,phases,start] GC(64) Marking Phase
[0.610s][info   ][gc,phases      ] GC(64) Marking Phase 1.500ms
[0.610s][info   ][gc,phases,start] GC(64) Summary Phase
[0.610s][info   ][gc,phases      ] GC(64) Summary Phase 0.013ms
[0.610s][info   ][gc,phases,start] GC(64) Adjust Roots
[0.610s][info   ][gc,phases      ] GC(64) Adjust Roots 0.141ms
[0.610s][info   ][gc,phases,start] GC(64) Compaction Phase
[0.616s][info   ][gc,phases      ] GC(64) Compaction Phase 6.293ms
[0.616s][info   ][gc,phases,start] GC(64) Post Compact
[0.616s][info   ][gc,phases      ] GC(64) Post Compact 0.204ms
[0.617s][info   ][gc,heap        ] GC(64) PSYoungGen: 58592K(116736K)->0K(116736K) Eden: 58592K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.617s][info   ][gc,heap        ] GC(64) ParOldGen: 330218K(349696K)->330538K(349696K)
[0.617s][info   ][gc,metaspace   ] GC(64) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.617s][info   ][gc             ] GC(64) Pause Full (Ergonomics) 379M->322M(455M) 8.385ms
[0.617s][info   ][gc,cpu         ] GC(64) User=0.03s Sys=0.00s Real=0.01s
[0.620s][info   ][gc,start       ] GC(65) Pause Full (Ergonomics)
[0.620s][info   ][gc,phases,start] GC(65) Marking Phase
[0.621s][info   ][gc,phases      ] GC(65) Marking Phase 1.650ms
[0.621s][info   ][gc,phases,start] GC(65) Summary Phase
[0.621s][info   ][gc,phases      ] GC(65) Summary Phase 0.015ms
[0.621s][info   ][gc,phases,start] GC(65) Adjust Roots
[0.621s][info   ][gc,phases      ] GC(65) Adjust Roots 0.133ms
[0.621s][info   ][gc,phases,start] GC(65) Compaction Phase
[0.627s][info   ][gc,phases      ] GC(65) Compaction Phase 5.194ms
[0.627s][info   ][gc,phases,start] GC(65) Post Compact
[0.627s][info   ][gc,phases      ] GC(65) Post Compact 0.157ms
[0.627s][info   ][gc,heap        ] GC(65) PSYoungGen: 58406K(116736K)->0K(116736K) Eden: 58406K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.627s][info   ][gc,heap        ] GC(65) ParOldGen: 330538K(349696K)->332417K(349696K)
[0.627s][info   ][gc,metaspace   ] GC(65) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.627s][info   ][gc             ] GC(65) Pause Full (Ergonomics) 379M->324M(455M) 7.320ms
[0.627s][info   ][gc,cpu         ] GC(65) User=0.04s Sys=0.00s Real=0.01s
[0.630s][info   ][gc,start       ] GC(66) Pause Full (Ergonomics)
[0.630s][info   ][gc,phases,start] GC(66) Marking Phase
[0.632s][info   ][gc,phases      ] GC(66) Marking Phase 1.899ms
[0.632s][info   ][gc,phases,start] GC(66) Summary Phase
[0.632s][info   ][gc,phases      ] GC(66) Summary Phase 0.014ms
[0.632s][info   ][gc,phases,start] GC(66) Adjust Roots
[0.632s][info   ][gc,phases      ] GC(66) Adjust Roots 0.144ms
[0.632s][info   ][gc,phases,start] GC(66) Compaction Phase
[0.638s][info   ][gc,phases      ] GC(66) Compaction Phase 5.538ms
[0.638s][info   ][gc,phases,start] GC(66) Post Compact
[0.638s][info   ][gc,phases      ] GC(66) Post Compact 0.152ms
[0.638s][info   ][gc,heap        ] GC(66) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.638s][info   ][gc,heap        ] GC(66) ParOldGen: 332417K(349696K)->333424K(349696K)
[0.638s][info   ][gc,metaspace   ] GC(66) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.638s][info   ][gc             ] GC(66) Pause Full (Ergonomics) 382M->325M(455M) 7.917ms
[0.638s][info   ][gc,cpu         ] GC(66) User=0.03s Sys=0.00s Real=0.01s
[0.641s][info   ][gc,start       ] GC(67) Pause Full (Ergonomics)
[0.641s][info   ][gc,phases,start] GC(67) Marking Phase
[0.642s][info   ][gc,phases      ] GC(67) Marking Phase 1.727ms
[0.642s][info   ][gc,phases,start] GC(67) Summary Phase
[0.642s][info   ][gc,phases      ] GC(67) Summary Phase 0.017ms
[0.642s][info   ][gc,phases,start] GC(67) Adjust Roots
[0.643s][info   ][gc,phases      ] GC(67) Adjust Roots 0.148ms
[0.643s][info   ][gc,phases,start] GC(67) Compaction Phase
[0.648s][info   ][gc,phases      ] GC(67) Compaction Phase 5.819ms
[0.648s][info   ][gc,phases,start] GC(67) Post Compact
[0.649s][info   ][gc,phases      ] GC(67) Post Compact 0.203ms
[0.649s][info   ][gc,heap        ] GC(67) PSYoungGen: 58770K(116736K)->0K(116736K) Eden: 58770K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.649s][info   ][gc,heap        ] GC(67) ParOldGen: 333424K(349696K)->337292K(349696K)
[0.649s][info   ][gc,metaspace   ] GC(67) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.649s][info   ][gc             ] GC(67) Pause Full (Ergonomics) 383M->329M(455M) 8.141ms
[0.649s][info   ][gc,cpu         ] GC(67) User=0.03s Sys=0.00s Real=0.01s
[0.652s][info   ][gc,start       ] GC(68) Pause Full (Ergonomics)
[0.652s][info   ][gc,phases,start] GC(68) Marking Phase
[0.654s][info   ][gc,phases      ] GC(68) Marking Phase 2.332ms
[0.654s][info   ][gc,phases,start] GC(68) Summary Phase
[0.654s][info   ][gc,phases      ] GC(68) Summary Phase 0.017ms
[0.654s][info   ][gc,phases,start] GC(68) Adjust Roots
[0.654s][info   ][gc,phases      ] GC(68) Adjust Roots 0.159ms
[0.654s][info   ][gc,phases,start] GC(68) Compaction Phase
[0.659s][info   ][gc,phases      ] GC(68) Compaction Phase 5.186ms
[0.660s][info   ][gc,phases,start] GC(68) Post Compact
[0.660s][info   ][gc,phases      ] GC(68) Post Compact 0.163ms
[0.660s][info   ][gc,heap        ] GC(68) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.660s][info   ][gc,heap        ] GC(68) ParOldGen: 337292K(349696K)->339280K(349696K)
[0.660s][info   ][gc,metaspace   ] GC(68) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.660s][info   ][gc             ] GC(68) Pause Full (Ergonomics) 386M->331M(455M) 8.047ms
[0.660s][info   ][gc,cpu         ] GC(68) User=0.03s Sys=0.01s Real=0.01s
[0.664s][info   ][gc,start       ] GC(69) Pause Full (Ergonomics)
[0.664s][info   ][gc,phases,start] GC(69) Marking Phase
[0.666s][info   ][gc,phases      ] GC(69) Marking Phase 2.091ms
[0.666s][info   ][gc,phases,start] GC(69) Summary Phase
[0.666s][info   ][gc,phases      ] GC(69) Summary Phase 0.019ms
[0.666s][info   ][gc,phases,start] GC(69) Adjust Roots
[0.666s][info   ][gc,phases      ] GC(69) Adjust Roots 0.141ms
[0.666s][info   ][gc,phases,start] GC(69) Compaction Phase
[0.672s][info   ][gc,phases      ] GC(69) Compaction Phase 5.529ms
[0.672s][info   ][gc,phases,start] GC(69) Post Compact
[0.672s][info   ][gc,phases      ] GC(69) Post Compact 0.168ms
[0.672s][info   ][gc,heap        ] GC(69) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.672s][info   ][gc,heap        ] GC(69) ParOldGen: 339280K(349696K)->341907K(349696K)
[0.672s][info   ][gc,metaspace   ] GC(69) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.672s][info   ][gc             ] GC(69) Pause Full (Ergonomics) 388M->333M(455M) 8.258ms
[0.672s][info   ][gc,cpu         ] GC(69) User=0.03s Sys=0.00s Real=0.00s
[0.675s][info   ][gc,start       ] GC(70) Pause Full (Ergonomics)
[0.675s][info   ][gc,phases,start] GC(70) Marking Phase
[0.677s][info   ][gc,phases      ] GC(70) Marking Phase 2.046ms
[0.677s][info   ][gc,phases,start] GC(70) Summary Phase
[0.677s][info   ][gc,phases      ] GC(70) Summary Phase 0.016ms
[0.677s][info   ][gc,phases,start] GC(70) Adjust Roots
[0.677s][info   ][gc,phases      ] GC(70) Adjust Roots 0.145ms
[0.677s][info   ][gc,phases,start] GC(70) Compaction Phase
[0.684s][info   ][gc,phases      ] GC(70) Compaction Phase 6.715ms
[0.684s][info   ][gc,phases,start] GC(70) Post Compact
[0.684s][info   ][gc,phases      ] GC(70) Post Compact 0.190ms
[0.684s][info   ][gc,heap        ] GC(70) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.684s][info   ][gc,heap        ] GC(70) ParOldGen: 341907K(349696K)->340487K(349696K)
[0.684s][info   ][gc,metaspace   ] GC(70) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.684s][info   ][gc             ] GC(70) Pause Full (Ergonomics) 391M->332M(455M) 9.337ms
[0.684s][info   ][gc,cpu         ] GC(70) User=0.04s Sys=0.00s Real=0.01s
[0.687s][info   ][gc,start       ] GC(71) Pause Full (Ergonomics)
[0.687s][info   ][gc,phases,start] GC(71) Marking Phase
[0.689s][info   ][gc,phases      ] GC(71) Marking Phase 2.016ms
[0.689s][info   ][gc,phases,start] GC(71) Summary Phase
[0.689s][info   ][gc,phases      ] GC(71) Summary Phase 0.016ms
[0.689s][info   ][gc,phases,start] GC(71) Adjust Roots
[0.689s][info   ][gc,phases      ] GC(71) Adjust Roots 0.132ms
[0.689s][info   ][gc,phases,start] GC(71) Compaction Phase
[0.695s][info   ][gc,phases      ] GC(71) Compaction Phase 5.310ms
[0.695s][info   ][gc,phases,start] GC(71) Post Compact
[0.695s][info   ][gc,phases      ] GC(71) Post Compact 0.160ms
[0.695s][info   ][gc,heap        ] GC(71) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.695s][info   ][gc,heap        ] GC(71) ParOldGen: 340487K(349696K)->338549K(349696K)
[0.695s][info   ][gc,metaspace   ] GC(71) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.695s][info   ][gc             ] GC(71) Pause Full (Ergonomics) 390M->330M(455M) 7.848ms
[0.695s][info   ][gc,cpu         ] GC(71) User=0.04s Sys=0.00s Real=0.01s
[0.699s][info   ][gc,start       ] GC(72) Pause Full (Ergonomics)
[0.699s][info   ][gc,phases,start] GC(72) Marking Phase
[0.702s][info   ][gc,phases      ] GC(72) Marking Phase 2.214ms
[0.702s][info   ][gc,phases,start] GC(72) Summary Phase
[0.702s][info   ][gc,phases      ] GC(72) Summary Phase 0.016ms
[0.702s][info   ][gc,phases,start] GC(72) Adjust Roots
[0.702s][info   ][gc,phases      ] GC(72) Adjust Roots 0.152ms
[0.702s][info   ][gc,phases,start] GC(72) Compaction Phase
[0.708s][info   ][gc,phases      ] GC(72) Compaction Phase 6.382ms
[0.708s][info   ][gc,phases,start] GC(72) Post Compact
[0.708s][info   ][gc,phases      ] GC(72) Post Compact 0.166ms
[0.708s][info   ][gc,heap        ] GC(72) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.708s][info   ][gc,heap        ] GC(72) ParOldGen: 338549K(349696K)->338045K(349696K)
[0.709s][info   ][gc,metaspace   ] GC(72) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.709s][info   ][gc             ] GC(72) Pause Full (Ergonomics) 388M->330M(455M) 9.121ms
[0.709s][info   ][gc,cpu         ] GC(72) User=0.03s Sys=0.00s Real=0.01s
[0.711s][info   ][gc,start       ] GC(73) Pause Full (Ergonomics)
[0.711s][info   ][gc,phases,start] GC(73) Marking Phase
[0.713s][info   ][gc,phases      ] GC(73) Marking Phase 1.658ms
[0.713s][info   ][gc,phases,start] GC(73) Summary Phase
[0.713s][info   ][gc,phases      ] GC(73) Summary Phase 0.015ms
[0.713s][info   ][gc,phases,start] GC(73) Adjust Roots
[0.713s][info   ][gc,phases      ] GC(73) Adjust Roots 0.154ms
[0.713s][info   ][gc,phases,start] GC(73) Compaction Phase
[0.719s][info   ][gc,phases      ] GC(73) Compaction Phase 5.657ms
[0.719s][info   ][gc,phases,start] GC(73) Post Compact
[0.719s][info   ][gc,phases      ] GC(73) Post Compact 0.159ms
[0.719s][info   ][gc,heap        ] GC(73) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.719s][info   ][gc,heap        ] GC(73) ParOldGen: 338045K(349696K)->339245K(349696K)
[0.719s][info   ][gc,metaspace   ] GC(73) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.719s][info   ][gc             ] GC(73) Pause Full (Ergonomics) 387M->331M(455M) 7.902ms
[0.719s][info   ][gc,cpu         ] GC(73) User=0.03s Sys=0.00s Real=0.01s
[0.722s][info   ][gc,start       ] GC(74) Pause Full (Ergonomics)
[0.722s][info   ][gc,phases,start] GC(74) Marking Phase
[0.724s][info   ][gc,phases      ] GC(74) Marking Phase 1.732ms
[0.724s][info   ][gc,phases,start] GC(74) Summary Phase
[0.724s][info   ][gc,phases      ] GC(74) Summary Phase 0.014ms
[0.724s][info   ][gc,phases,start] GC(74) Adjust Roots
[0.724s][info   ][gc,phases      ] GC(74) Adjust Roots 0.166ms
[0.724s][info   ][gc,phases,start] GC(74) Compaction Phase
[0.730s][info   ][gc,phases      ] GC(74) Compaction Phase 5.952ms
[0.730s][info   ][gc,phases,start] GC(74) Post Compact
[0.730s][info   ][gc,phases      ] GC(74) Post Compact 0.218ms
[0.730s][info   ][gc,heap        ] GC(74) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.730s][info   ][gc,heap        ] GC(74) ParOldGen: 339245K(349696K)->339435K(349696K)
[0.730s][info   ][gc,metaspace   ] GC(74) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.730s][info   ][gc             ] GC(74) Pause Full (Ergonomics) 388M->331M(455M) 8.335ms
[0.730s][info   ][gc,cpu         ] GC(74) User=0.03s Sys=0.00s Real=0.01s
[0.733s][info   ][gc,start       ] GC(75) Pause Full (Ergonomics)
[0.733s][info   ][gc,phases,start] GC(75) Marking Phase
[0.736s][info   ][gc,phases      ] GC(75) Marking Phase 2.042ms
[0.736s][info   ][gc,phases,start] GC(75) Summary Phase
[0.736s][info   ][gc,phases      ] GC(75) Summary Phase 0.015ms
[0.736s][info   ][gc,phases,start] GC(75) Adjust Roots
[0.736s][info   ][gc,phases      ] GC(75) Adjust Roots 0.155ms
[0.736s][info   ][gc,phases,start] GC(75) Compaction Phase
[0.742s][info   ][gc,phases      ] GC(75) Compaction Phase 6.069ms
[0.742s][info   ][gc,phases,start] GC(75) Post Compact
[0.742s][info   ][gc,phases      ] GC(75) Post Compact 0.172ms
[0.742s][info   ][gc,heap        ] GC(75) PSYoungGen: 58731K(116736K)->0K(116736K) Eden: 58731K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.742s][info   ][gc,heap        ] GC(75) ParOldGen: 339435K(349696K)->341139K(349696K)
[0.742s][info   ][gc,metaspace   ] GC(75) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.742s][info   ][gc             ] GC(75) Pause Full (Ergonomics) 388M->333M(455M) 8.642ms
[0.742s][info   ][gc,cpu         ] GC(75) User=0.04s Sys=0.00s Real=0.00s
[0.745s][info   ][gc,start       ] GC(76) Pause Full (Ergonomics)
[0.745s][info   ][gc,phases,start] GC(76) Marking Phase
[0.747s][info   ][gc,phases      ] GC(76) Marking Phase 1.948ms
[0.747s][info   ][gc,phases,start] GC(76) Summary Phase
[0.747s][info   ][gc,phases      ] GC(76) Summary Phase 0.015ms
[0.747s][info   ][gc,phases,start] GC(76) Adjust Roots
[0.748s][info   ][gc,phases      ] GC(76) Adjust Roots 0.147ms
[0.748s][info   ][gc,phases,start] GC(76) Compaction Phase
[0.753s][info   ][gc,phases      ] GC(76) Compaction Phase 5.275ms
[0.753s][info   ][gc,phases,start] GC(76) Post Compact
[0.753s][info   ][gc,phases      ] GC(76) Post Compact 0.160ms
[0.753s][info   ][gc,heap        ] GC(76) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.753s][info   ][gc,heap        ] GC(76) ParOldGen: 341139K(349696K)->342065K(349696K)
[0.753s][info   ][gc,metaspace   ] GC(76) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.753s][info   ][gc             ] GC(76) Pause Full (Ergonomics) 390M->334M(455M) 7.788ms
[0.753s][info   ][gc,cpu         ] GC(76) User=0.04s Sys=0.00s Real=0.01s
[0.757s][info   ][gc,start       ] GC(77) Pause Full (Ergonomics)
[0.757s][info   ][gc,phases,start] GC(77) Marking Phase
[0.759s][info   ][gc,phases      ] GC(77) Marking Phase 2.001ms
[0.759s][info   ][gc,phases,start] GC(77) Summary Phase
[0.759s][info   ][gc,phases      ] GC(77) Summary Phase 0.014ms
[0.759s][info   ][gc,phases,start] GC(77) Adjust Roots
[0.759s][info   ][gc,phases      ] GC(77) Adjust Roots 0.184ms
[0.759s][info   ][gc,phases,start] GC(77) Compaction Phase
[0.765s][info   ][gc,phases      ] GC(77) Compaction Phase 6.052ms
[0.765s][info   ][gc,phases,start] GC(77) Post Compact
[0.765s][info   ][gc,phases      ] GC(77) Post Compact 0.248ms
[0.765s][info   ][gc,heap        ] GC(77) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.765s][info   ][gc,heap        ] GC(77) ParOldGen: 342065K(349696K)->340685K(349696K)
[0.765s][info   ][gc,metaspace   ] GC(77) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.765s][info   ][gc             ] GC(77) Pause Full (Ergonomics) 391M->332M(455M) 8.753ms
[0.765s][info   ][gc,cpu         ] GC(77) User=0.03s Sys=0.01s Real=0.01s
[0.768s][info   ][gc,start       ] GC(78) Pause Full (Ergonomics)
[0.768s][info   ][gc,phases,start] GC(78) Marking Phase
[0.770s][info   ][gc,phases      ] GC(78) Marking Phase 1.999ms
[0.770s][info   ][gc,phases,start] GC(78) Summary Phase
[0.770s][info   ][gc,phases      ] GC(78) Summary Phase 0.015ms
[0.770s][info   ][gc,phases,start] GC(78) Adjust Roots
[0.770s][info   ][gc,phases      ] GC(78) Adjust Roots 0.142ms
[0.770s][info   ][gc,phases,start] GC(78) Compaction Phase
[0.775s][info   ][gc,phases      ] GC(78) Compaction Phase 4.969ms
[0.775s][info   ][gc,phases,start] GC(78) Post Compact
[0.776s][info   ][gc,phases      ] GC(78) Post Compact 0.164ms
[0.776s][info   ][gc,heap        ] GC(78) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.776s][info   ][gc,heap        ] GC(78) ParOldGen: 340685K(349696K)->345247K(349696K)
[0.776s][info   ][gc,metaspace   ] GC(78) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.776s][info   ][gc             ] GC(78) Pause Full (Ergonomics) 390M->337M(455M) 7.506ms
[0.776s][info   ][gc,cpu         ] GC(78) User=0.04s Sys=0.00s Real=0.01s
[0.779s][info   ][gc,start       ] GC(79) Pause Full (Ergonomics)
[0.779s][info   ][gc,phases,start] GC(79) Marking Phase
[0.780s][info   ][gc,phases      ] GC(79) Marking Phase 1.796ms
[0.780s][info   ][gc,phases,start] GC(79) Summary Phase
[0.780s][info   ][gc,phases      ] GC(79) Summary Phase 0.021ms
[0.780s][info   ][gc,phases,start] GC(79) Adjust Roots
[0.781s][info   ][gc,phases      ] GC(79) Adjust Roots 0.136ms
[0.781s][info   ][gc,phases,start] GC(79) Compaction Phase
[0.786s][info   ][gc,phases      ] GC(79) Compaction Phase 4.974ms
[0.786s][info   ][gc,phases,start] GC(79) Post Compact
[0.786s][info   ][gc,phases      ] GC(79) Post Compact 0.169ms
[0.786s][info   ][gc,heap        ] GC(79) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.786s][info   ][gc,heap        ] GC(79) ParOldGen: 345247K(349696K)->341591K(349696K)
[0.786s][info   ][gc,metaspace   ] GC(79) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.786s][info   ][gc             ] GC(79) Pause Full (Ergonomics) 394M->333M(455M) 7.317ms
[0.786s][info   ][gc,cpu         ] GC(79) User=0.04s Sys=0.00s Real=0.01s
[0.788s][info   ][gc,start       ] GC(80) Pause Full (Ergonomics)
[0.788s][info   ][gc,phases,start] GC(80) Marking Phase
[0.790s][info   ][gc,phases      ] GC(80) Marking Phase 1.717ms
[0.790s][info   ][gc,phases,start] GC(80) Summary Phase
[0.790s][info   ][gc,phases      ] GC(80) Summary Phase 0.014ms
[0.790s][info   ][gc,phases,start] GC(80) Adjust Roots
[0.790s][info   ][gc,phases      ] GC(80) Adjust Roots 0.139ms
[0.790s][info   ][gc,phases,start] GC(80) Compaction Phase
[0.800s][info   ][gc,phases      ] GC(80) Compaction Phase 9.428ms
[0.800s][info   ][gc,phases,start] GC(80) Post Compact
[0.800s][info   ][gc,phases      ] GC(80) Post Compact 0.151ms
[0.800s][info   ][gc,heap        ] GC(80) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.800s][info   ][gc,heap        ] GC(80) ParOldGen: 341591K(349696K)->339177K(349696K)
[0.800s][info   ][gc,metaspace   ] GC(80) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.800s][info   ][gc             ] GC(80) Pause Full (Ergonomics) 391M->331M(455M) 11.639ms
[0.800s][info   ][gc,cpu         ] GC(80) User=0.03s Sys=0.00s Real=0.01s
[0.803s][info   ][gc,start       ] GC(81) Pause Full (Ergonomics)
[0.803s][info   ][gc,phases,start] GC(81) Marking Phase
[0.805s][info   ][gc,phases      ] GC(81) Marking Phase 2.156ms
[0.805s][info   ][gc,phases,start] GC(81) Summary Phase
[0.806s][info   ][gc,phases      ] GC(81) Summary Phase 0.014ms
[0.806s][info   ][gc,phases,start] GC(81) Adjust Roots
[0.806s][info   ][gc,phases      ] GC(81) Adjust Roots 0.153ms
[0.806s][info   ][gc,phases,start] GC(81) Compaction Phase
[0.811s][info   ][gc,phases      ] GC(81) Compaction Phase 5.151ms
[0.811s][info   ][gc,phases,start] GC(81) Post Compact
[0.811s][info   ][gc,phases      ] GC(81) Post Compact 0.159ms
[0.811s][info   ][gc,heap        ] GC(81) PSYoungGen: 58707K(116736K)->0K(116736K) Eden: 58707K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.811s][info   ][gc,heap        ] GC(81) ParOldGen: 339177K(349696K)->337198K(349696K)
[0.811s][info   ][gc,metaspace   ] GC(81) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.811s][info   ][gc             ] GC(81) Pause Full (Ergonomics) 388M->329M(455M) 7.834ms
[0.811s][info   ][gc,cpu         ] GC(81) User=0.04s Sys=0.00s Real=0.00s
[0.814s][info   ][gc,start       ] GC(82) Pause Full (Ergonomics)
[0.814s][info   ][gc,phases,start] GC(82) Marking Phase
[0.816s][info   ][gc,phases      ] GC(82) Marking Phase 1.875ms
[0.816s][info   ][gc,phases,start] GC(82) Summary Phase
[0.816s][info   ][gc,phases      ] GC(82) Summary Phase 0.017ms
[0.816s][info   ][gc,phases,start] GC(82) Adjust Roots
[0.816s][info   ][gc,phases      ] GC(82) Adjust Roots 0.342ms
[0.816s][info   ][gc,phases,start] GC(82) Compaction Phase
[0.822s][info   ][gc,phases      ] GC(82) Compaction Phase 5.214ms
[0.822s][info   ][gc,phases,start] GC(82) Post Compact
[0.822s][info   ][gc,phases      ] GC(82) Post Compact 0.187ms
[0.822s][info   ][gc,heap        ] GC(82) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.822s][info   ][gc,heap        ] GC(82) ParOldGen: 337198K(349696K)->333216K(349696K)
[0.822s][info   ][gc,metaspace   ] GC(82) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.822s][info   ][gc             ] GC(82) Pause Full (Ergonomics) 386M->325M(455M) 7.852ms
[0.822s][info   ][gc,cpu         ] GC(82) User=0.04s Sys=0.01s Real=0.00s
[0.825s][info   ][gc,start       ] GC(83) Pause Full (Ergonomics)
[0.825s][info   ][gc,phases,start] GC(83) Marking Phase
[0.827s][info   ][gc,phases      ] GC(83) Marking Phase 2.247ms
[0.827s][info   ][gc,phases,start] GC(83) Summary Phase
[0.827s][info   ][gc,phases      ] GC(83) Summary Phase 0.017ms
[0.827s][info   ][gc,phases,start] GC(83) Adjust Roots
[0.829s][info   ][gc,phases      ] GC(83) Adjust Roots 1.196ms
[0.829s][info   ][gc,phases,start] GC(83) Compaction Phase
[0.834s][info   ][gc,phases      ] GC(83) Compaction Phase 5.418ms
[0.834s][info   ][gc,phases,start] GC(83) Post Compact
[0.834s][info   ][gc,phases      ] GC(83) Post Compact 0.163ms
[0.834s][info   ][gc,heap        ] GC(83) PSYoungGen: 58736K(116736K)->0K(116736K) Eden: 58736K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.834s][info   ][gc,heap        ] GC(83) ParOldGen: 333216K(349696K)->333547K(349696K)
[0.834s][info   ][gc,metaspace   ] GC(83) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.834s][info   ][gc             ] GC(83) Pause Full (Ergonomics) 382M->325M(455M) 9.264ms
[0.834s][info   ][gc,cpu         ] GC(83) User=0.03s Sys=0.00s Real=0.01s
[0.837s][info   ][gc,start       ] GC(84) Pause Full (Ergonomics)
[0.837s][info   ][gc,phases,start] GC(84) Marking Phase
[0.839s][info   ][gc,phases      ] GC(84) Marking Phase 2.042ms
[0.839s][info   ][gc,phases,start] GC(84) Summary Phase
[0.839s][info   ][gc,phases      ] GC(84) Summary Phase 0.015ms
[0.839s][info   ][gc,phases,start] GC(84) Adjust Roots
[0.840s][info   ][gc,phases      ] GC(84) Adjust Roots 0.145ms
[0.840s][info   ][gc,phases,start] GC(84) Compaction Phase
[0.845s][info   ][gc,phases      ] GC(84) Compaction Phase 5.179ms
[0.845s][info   ][gc,phases,start] GC(84) Post Compact
[0.845s][info   ][gc,phases      ] GC(84) Post Compact 0.172ms
[0.845s][info   ][gc,heap        ] GC(84) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.845s][info   ][gc,heap        ] GC(84) ParOldGen: 333547K(349696K)->332593K(349696K)
[0.845s][info   ][gc,metaspace   ] GC(84) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.845s][info   ][gc             ] GC(84) Pause Full (Ergonomics) 383M->324M(455M) 7.745ms
[0.845s][info   ][gc,cpu         ] GC(84) User=0.03s Sys=0.00s Real=0.01s
[0.848s][info   ][gc,start       ] GC(85) Pause Full (Ergonomics)
[0.848s][info   ][gc,phases,start] GC(85) Marking Phase
[0.850s][info   ][gc,phases      ] GC(85) Marking Phase 1.964ms
[0.850s][info   ][gc,phases,start] GC(85) Summary Phase
[0.850s][info   ][gc,phases      ] GC(85) Summary Phase 0.017ms
[0.850s][info   ][gc,phases,start] GC(85) Adjust Roots
[0.850s][info   ][gc,phases      ] GC(85) Adjust Roots 0.143ms
[0.850s][info   ][gc,phases,start] GC(85) Compaction Phase
[0.855s][info   ][gc,phases      ] GC(85) Compaction Phase 5.475ms
[0.855s][info   ][gc,phases,start] GC(85) Post Compact
[0.856s][info   ][gc,phases      ] GC(85) Post Compact 0.219ms
[0.856s][info   ][gc,heap        ] GC(85) PSYoungGen: 58805K(116736K)->0K(116736K) Eden: 58805K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.856s][info   ][gc,heap        ] GC(85) ParOldGen: 332593K(349696K)->337090K(349696K)
[0.856s][info   ][gc,metaspace   ] GC(85) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.856s][info   ][gc             ] GC(85) Pause Full (Ergonomics) 382M->329M(455M) 8.015ms
[0.856s][info   ][gc,cpu         ] GC(85) User=0.04s Sys=0.00s Real=0.01s
[0.859s][info   ][gc,start       ] GC(86) Pause Full (Ergonomics)
[0.859s][info   ][gc,phases,start] GC(86) Marking Phase
[0.862s][info   ][gc,phases      ] GC(86) Marking Phase 2.598ms
[0.862s][info   ][gc,phases,start] GC(86) Summary Phase
[0.862s][info   ][gc,phases      ] GC(86) Summary Phase 0.026ms
[0.862s][info   ][gc,phases,start] GC(86) Adjust Roots
[0.862s][info   ][gc,phases      ] GC(86) Adjust Roots 0.160ms
[0.862s][info   ][gc,phases,start] GC(86) Compaction Phase
[0.869s][info   ][gc,phases      ] GC(86) Compaction Phase 7.084ms
[0.869s][info   ][gc,phases,start] GC(86) Post Compact
[0.869s][info   ][gc,phases      ] GC(86) Post Compact 0.157ms
[0.869s][info   ][gc,heap        ] GC(86) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.869s][info   ][gc,heap        ] GC(86) ParOldGen: 337090K(349696K)->340061K(349696K)
[0.869s][info   ][gc,metaspace   ] GC(86) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.869s][info   ][gc             ] GC(86) Pause Full (Ergonomics) 386M->332M(455M) 10.208ms
[0.869s][info   ][gc,cpu         ] GC(86) User=0.04s Sys=0.00s Real=0.01s
[0.872s][info   ][gc,start       ] GC(87) Pause Full (Ergonomics)
[0.872s][info   ][gc,phases,start] GC(87) Marking Phase
[0.874s][info   ][gc,phases      ] GC(87) Marking Phase 1.803ms
[0.874s][info   ][gc,phases,start] GC(87) Summary Phase
[0.874s][info   ][gc,phases      ] GC(87) Summary Phase 0.016ms
[0.874s][info   ][gc,phases,start] GC(87) Adjust Roots
[0.874s][info   ][gc,phases      ] GC(87) Adjust Roots 0.135ms
[0.874s][info   ][gc,phases,start] GC(87) Compaction Phase
[0.880s][info   ][gc,phases      ] GC(87) Compaction Phase 5.511ms
[0.880s][info   ][gc,phases,start] GC(87) Post Compact
[0.880s][info   ][gc,phases      ] GC(87) Post Compact 0.155ms
[0.880s][info   ][gc,heap        ] GC(87) PSYoungGen: 58622K(116736K)->0K(116736K) Eden: 58622K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.880s][info   ][gc,heap        ] GC(87) ParOldGen: 340061K(349696K)->339338K(349696K)
[0.880s][info   ][gc,metaspace   ] GC(87) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.880s][info   ][gc             ] GC(87) Pause Full (Ergonomics) 389M->331M(455M) 7.801ms
[0.880s][info   ][gc,cpu         ] GC(87) User=0.04s Sys=0.01s Real=0.01s
[0.883s][info   ][gc,start       ] GC(88) Pause Full (Ergonomics)
[0.883s][info   ][gc,phases,start] GC(88) Marking Phase
[0.885s][info   ][gc,phases      ] GC(88) Marking Phase 2.168ms
[0.885s][info   ][gc,phases,start] GC(88) Summary Phase
[0.885s][info   ][gc,phases      ] GC(88) Summary Phase 0.017ms
[0.885s][info   ][gc,phases,start] GC(88) Adjust Roots
[0.885s][info   ][gc,phases      ] GC(88) Adjust Roots 0.144ms
[0.885s][info   ][gc,phases,start] GC(88) Compaction Phase
[0.891s][info   ][gc,phases      ] GC(88) Compaction Phase 5.649ms
[0.891s][info   ][gc,phases,start] GC(88) Post Compact
[0.891s][info   ][gc,phases      ] GC(88) Post Compact 0.159ms
[0.891s][info   ][gc,heap        ] GC(88) PSYoungGen: 58825K(116736K)->0K(116736K) Eden: 58825K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.891s][info   ][gc,heap        ] GC(88) ParOldGen: 339338K(349696K)->338980K(349696K)
[0.891s][info   ][gc,metaspace   ] GC(88) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.891s][info   ][gc             ] GC(88) Pause Full (Ergonomics) 388M->331M(455M) 8.363ms
[0.891s][info   ][gc,cpu         ] GC(88) User=0.04s Sys=0.00s Real=0.00s
[0.894s][info   ][gc,start       ] GC(89) Pause Full (Ergonomics)
[0.894s][info   ][gc,phases,start] GC(89) Marking Phase
[0.896s][info   ][gc,phases      ] GC(89) Marking Phase 2.470ms
[0.896s][info   ][gc,phases,start] GC(89) Summary Phase
[0.896s][info   ][gc,phases      ] GC(89) Summary Phase 0.020ms
[0.896s][info   ][gc,phases,start] GC(89) Adjust Roots
[0.896s][info   ][gc,phases      ] GC(89) Adjust Roots 0.148ms
[0.896s][info   ][gc,phases,start] GC(89) Compaction Phase
[0.903s][info   ][gc,phases      ] GC(89) Compaction Phase 6.296ms
[0.903s][info   ][gc,phases,start] GC(89) Post Compact
[0.903s][info   ][gc,phases      ] GC(89) Post Compact 0.186ms
[0.903s][info   ][gc,heap        ] GC(89) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.903s][info   ][gc,heap        ] GC(89) ParOldGen: 338980K(349696K)->337584K(349696K)
[0.903s][info   ][gc,metaspace   ] GC(89) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.903s][info   ][gc             ] GC(89) Pause Full (Ergonomics) 388M->329M(455M) 9.335ms
[0.903s][info   ][gc,cpu         ] GC(89) User=0.04s Sys=0.00s Real=0.01s
[0.906s][info   ][gc,start       ] GC(90) Pause Full (Ergonomics)
[0.906s][info   ][gc,phases,start] GC(90) Marking Phase
[0.908s][info   ][gc,phases      ] GC(90) Marking Phase 2.283ms
[0.909s][info   ][gc,phases,start] GC(90) Summary Phase
[0.909s][info   ][gc,phases      ] GC(90) Summary Phase 0.016ms
[0.909s][info   ][gc,phases,start] GC(90) Adjust Roots
[0.909s][info   ][gc,phases      ] GC(90) Adjust Roots 0.150ms
[0.909s][info   ][gc,phases,start] GC(90) Compaction Phase
[0.914s][info   ][gc,phases      ] GC(90) Compaction Phase 5.759ms
[0.915s][info   ][gc,phases,start] GC(90) Post Compact
[0.915s][info   ][gc,phases      ] GC(90) Post Compact 0.154ms
[0.915s][info   ][gc,heap        ] GC(90) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.915s][info   ][gc,heap        ] GC(90) ParOldGen: 337584K(349696K)->337049K(349696K)
[0.915s][info   ][gc,metaspace   ] GC(90) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.915s][info   ][gc             ] GC(90) Pause Full (Ergonomics) 387M->329M(455M) 8.577ms
[0.915s][info   ][gc,cpu         ] GC(90) User=0.04s Sys=0.00s Real=0.01s
[0.918s][info   ][gc,start       ] GC(91) Pause Full (Ergonomics)
[0.918s][info   ][gc,phases,start] GC(91) Marking Phase
[0.920s][info   ][gc,phases      ] GC(91) Marking Phase 2.154ms
[0.920s][info   ][gc,phases,start] GC(91) Summary Phase
[0.920s][info   ][gc,phases      ] GC(91) Summary Phase 0.013ms
[0.920s][info   ][gc,phases,start] GC(91) Adjust Roots
[0.920s][info   ][gc,phases      ] GC(91) Adjust Roots 0.176ms
[0.920s][info   ][gc,phases,start] GC(91) Compaction Phase
[0.926s][info   ][gc,phases      ] GC(91) Compaction Phase 5.547ms
[0.926s][info   ][gc,phases,start] GC(91) Post Compact
[0.926s][info   ][gc,phases      ] GC(91) Post Compact 0.158ms
[0.926s][info   ][gc,heap        ] GC(91) PSYoungGen: 58798K(116736K)->0K(116736K) Eden: 58798K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.926s][info   ][gc,heap        ] GC(91) ParOldGen: 337049K(349696K)->332320K(349696K)
[0.926s][info   ][gc,metaspace   ] GC(91) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.926s][info   ][gc             ] GC(91) Pause Full (Ergonomics) 386M->324M(455M) 8.247ms
[0.926s][info   ][gc,cpu         ] GC(91) User=0.04s Sys=0.00s Real=0.01s
[0.930s][info   ][gc,start       ] GC(92) Pause Full (Ergonomics)
[0.930s][info   ][gc,phases,start] GC(92) Marking Phase
[0.932s][info   ][gc,phases      ] GC(92) Marking Phase 2.350ms
[0.932s][info   ][gc,phases,start] GC(92) Summary Phase
[0.932s][info   ][gc,phases      ] GC(92) Summary Phase 0.015ms
[0.932s][info   ][gc,phases,start] GC(92) Adjust Roots
[0.932s][info   ][gc,phases      ] GC(92) Adjust Roots 0.153ms
[0.932s][info   ][gc,phases,start] GC(92) Compaction Phase
[0.938s][info   ][gc,phases      ] GC(92) Compaction Phase 5.653ms
[0.938s][info   ][gc,phases,start] GC(92) Post Compact
[0.938s][info   ][gc,phases      ] GC(92) Post Compact 0.160ms
[0.938s][info   ][gc,heap        ] GC(92) PSYoungGen: 58620K(116736K)->0K(116736K) Eden: 58620K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.938s][info   ][gc,heap        ] GC(92) ParOldGen: 332320K(349696K)->332929K(349696K)
[0.938s][info   ][gc,metaspace   ] GC(92) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.938s][info   ][gc             ] GC(92) Pause Full (Ergonomics) 381M->325M(455M) 8.542ms
[0.938s][info   ][gc,cpu         ] GC(92) User=0.03s Sys=0.01s Real=0.01s
[0.941s][info   ][gc,start       ] GC(93) Pause Full (Ergonomics)
[0.941s][info   ][gc,phases,start] GC(93) Marking Phase
[0.943s][info   ][gc,phases      ] GC(93) Marking Phase 2.195ms
[0.943s][info   ][gc,phases,start] GC(93) Summary Phase
[0.943s][info   ][gc,phases      ] GC(93) Summary Phase 0.014ms
[0.943s][info   ][gc,phases,start] GC(93) Adjust Roots
[0.944s][info   ][gc,phases      ] GC(93) Adjust Roots 0.155ms
[0.944s][info   ][gc,phases,start] GC(93) Compaction Phase
[0.949s][info   ][gc,phases      ] GC(93) Compaction Phase 5.683ms
[0.949s][info   ][gc,phases,start] GC(93) Post Compact
[0.950s][info   ][gc,phases      ] GC(93) Post Compact 0.159ms
[0.950s][info   ][gc,heap        ] GC(93) PSYoungGen: 58576K(116736K)->0K(116736K) Eden: 58576K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.950s][info   ][gc,heap        ] GC(93) ParOldGen: 332929K(349696K)->332316K(349696K)
[0.950s][info   ][gc,metaspace   ] GC(93) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.950s][info   ][gc             ] GC(93) Pause Full (Ergonomics) 382M->324M(455M) 8.425ms
[0.950s][info   ][gc,cpu         ] GC(93) User=0.03s Sys=0.00s Real=0.01s
[0.954s][info   ][gc,start       ] GC(94) Pause Full (Ergonomics)
[0.954s][info   ][gc,phases,start] GC(94) Marking Phase
[0.956s][info   ][gc,phases      ] GC(94) Marking Phase 1.754ms
[0.956s][info   ][gc,phases,start] GC(94) Summary Phase
[0.956s][info   ][gc,phases      ] GC(94) Summary Phase 0.016ms
[0.956s][info   ][gc,phases,start] GC(94) Adjust Roots
[0.956s][info   ][gc,phases      ] GC(94) Adjust Roots 0.130ms
[0.956s][info   ][gc,phases,start] GC(94) Compaction Phase
[0.961s][info   ][gc,phases      ] GC(94) Compaction Phase 5.308ms
[0.962s][info   ][gc,phases,start] GC(94) Post Compact
[0.962s][info   ][gc,phases      ] GC(94) Post Compact 0.159ms
[0.962s][info   ][gc,heap        ] GC(94) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.962s][info   ][gc,heap        ] GC(94) ParOldGen: 332316K(349696K)->333329K(349696K)
[0.962s][info   ][gc,metaspace   ] GC(94) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.962s][info   ][gc             ] GC(94) Pause Full (Ergonomics) 382M->325M(455M) 7.525ms
[0.962s][info   ][gc,cpu         ] GC(94) User=0.03s Sys=0.00s Real=0.00s
[0.965s][info   ][gc,start       ] GC(95) Pause Full (Ergonomics)
[0.965s][info   ][gc,phases,start] GC(95) Marking Phase
[0.967s][info   ][gc,phases      ] GC(95) Marking Phase 1.880ms
[0.967s][info   ][gc,phases,start] GC(95) Summary Phase
[0.967s][info   ][gc,phases      ] GC(95) Summary Phase 0.014ms
[0.967s][info   ][gc,phases,start] GC(95) Adjust Roots
[0.967s][info   ][gc,phases      ] GC(95) Adjust Roots 0.148ms
[0.967s][info   ][gc,phases,start] GC(95) Compaction Phase
[0.973s][info   ][gc,phases      ] GC(95) Compaction Phase 5.887ms
[0.973s][info   ][gc,phases,start] GC(95) Post Compact
[0.973s][info   ][gc,phases      ] GC(95) Post Compact 0.170ms
[0.973s][info   ][gc,heap        ] GC(95) PSYoungGen: 58673K(116736K)->0K(116736K) Eden: 58673K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.973s][info   ][gc,heap        ] GC(95) ParOldGen: 333329K(349696K)->329838K(349696K)
[0.973s][info   ][gc,metaspace   ] GC(95) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.973s][info   ][gc             ] GC(95) Pause Full (Ergonomics) 382M->322M(455M) 8.300ms
[0.973s][info   ][gc,cpu         ] GC(95) User=0.04s Sys=0.00s Real=0.01s
[0.976s][info   ][gc,start       ] GC(96) Pause Full (Ergonomics)
[0.976s][info   ][gc,phases,start] GC(96) Marking Phase
[0.979s][info   ][gc,phases      ] GC(96) Marking Phase 2.129ms
[0.979s][info   ][gc,phases,start] GC(96) Summary Phase
[0.979s][info   ][gc,phases      ] GC(96) Summary Phase 0.016ms
[0.979s][info   ][gc,phases,start] GC(96) Adjust Roots
[0.979s][info   ][gc,phases      ] GC(96) Adjust Roots 0.142ms
[0.979s][info   ][gc,phases,start] GC(96) Compaction Phase
[0.984s][info   ][gc,phases      ] GC(96) Compaction Phase 5.409ms
[0.984s][info   ][gc,phases,start] GC(96) Post Compact
[0.984s][info   ][gc,phases      ] GC(96) Post Compact 0.161ms
[0.984s][info   ][gc,heap        ] GC(96) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[0.984s][info   ][gc,heap        ] GC(96) ParOldGen: 329838K(349696K)->332469K(349696K)
[0.984s][info   ][gc,metaspace   ] GC(96) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.984s][info   ][gc             ] GC(96) Pause Full (Ergonomics) 379M->324M(455M) 8.052ms
[0.984s][info   ][gc,cpu         ] GC(96) User=0.04s Sys=0.00s Real=0.01s
[0.991s][info   ][gc,start       ] GC(97) Pause Full (Ergonomics)
[0.991s][info   ][gc,phases,start] GC(97) Marking Phase
[0.993s][info   ][gc,phases      ] GC(97) Marking Phase 2.074ms
[0.993s][info   ][gc,phases,start] GC(97) Summary Phase
[0.993s][info   ][gc,phases      ] GC(97) Summary Phase 0.017ms
[0.993s][info   ][gc,phases,start] GC(97) Adjust Roots
[0.993s][info   ][gc,phases      ] GC(97) Adjust Roots 0.146ms
[0.993s][info   ][gc,phases,start] GC(97) Compaction Phase
[1.000s][info   ][gc,phases      ] GC(97) Compaction Phase 6.613ms
[1.000s][info   ][gc,phases,start] GC(97) Post Compact
[1.000s][info   ][gc,phases      ] GC(97) Post Compact 0.187ms
[1.000s][info   ][gc,heap        ] GC(97) PSYoungGen: 58749K(116736K)->0K(116736K) Eden: 58749K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[1.000s][info   ][gc,heap        ] GC(97) ParOldGen: 332469K(349696K)->338455K(349696K)
[1.000s][info   ][gc,metaspace   ] GC(97) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.000s][info   ][gc             ] GC(97) Pause Full (Ergonomics) 382M->330M(455M) 9.249ms
[1.000s][info   ][gc,cpu         ] GC(97) User=0.03s Sys=0.01s Real=0.01s
[1.003s][info   ][gc,start       ] GC(98) Pause Full (Ergonomics)
[1.003s][info   ][gc,phases,start] GC(98) Marking Phase
[1.005s][info   ][gc,phases      ] GC(98) Marking Phase 1.781ms
[1.005s][info   ][gc,phases,start] GC(98) Summary Phase
[1.005s][info   ][gc,phases      ] GC(98) Summary Phase 0.029ms
[1.005s][info   ][gc,phases,start] GC(98) Adjust Roots
[1.005s][info   ][gc,phases      ] GC(98) Adjust Roots 0.137ms
[1.005s][info   ][gc,phases,start] GC(98) Compaction Phase
[1.010s][info   ][gc,phases      ] GC(98) Compaction Phase 5.105ms
[1.010s][info   ][gc,phases,start] GC(98) Post Compact
[1.011s][info   ][gc,phases      ] GC(98) Post Compact 0.156ms
[1.011s][info   ][gc,heap        ] GC(98) PSYoungGen: 58880K(116736K)->0K(116736K) Eden: 58880K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[1.011s][info   ][gc,heap        ] GC(98) ParOldGen: 338455K(349696K)->344861K(349696K)
[1.011s][info   ][gc,metaspace   ] GC(98) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.011s][info   ][gc             ] GC(98) Pause Full (Ergonomics) 388M->336M(455M) 7.448ms
[1.011s][info   ][gc,cpu         ] GC(98) User=0.03s Sys=0.00s Real=0.00s
[1.014s][info   ][gc,start       ] GC(99) Pause Full (Ergonomics)
[1.014s][info   ][gc,phases,start] GC(99) Marking Phase
[1.016s][info   ][gc,phases      ] GC(99) Marking Phase 1.975ms
[1.016s][info   ][gc,phases,start] GC(99) Summary Phase
[1.016s][info   ][gc,phases      ] GC(99) Summary Phase 0.015ms
[1.016s][info   ][gc,phases,start] GC(99) Adjust Roots
[1.016s][info   ][gc,phases      ] GC(99) Adjust Roots 0.143ms
[1.016s][info   ][gc,phases,start] GC(99) Compaction Phase
[1.021s][info   ][gc,phases      ] GC(99) Compaction Phase 5.385ms
[1.021s][info   ][gc,phases,start] GC(99) Post Compact
[1.021s][info   ][gc,phases      ] GC(99) Post Compact 0.176ms
[1.021s][info   ][gc,heap        ] GC(99) PSYoungGen: 58862K(116736K)->0K(116736K) Eden: 58862K(58880K)->0K(58880K) From: 0K(57856K)->0K(57856K)
[1.021s][info   ][gc,heap        ] GC(99) ParOldGen: 344861K(349696K)->340647K(349696K)
[1.021s][info   ][gc,metaspace   ] GC(99) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.021s][info   ][gc             ] GC(99) Pause Full (Ergonomics) 394M->332M(455M) 7.849ms
[1.021s][info   ][gc,cpu         ] GC(99) User=0.03s Sys=0.00s Real=0.00s
counter:27740
[1.027s][info   ][gc,heap,exit   ] Heap
[1.027s][info   ][gc,heap,exit   ]  PSYoungGen      total 116736K, used 2641K [0x00000007f5580000, 0x0000000800000000, 0x0000000800000000)
[1.027s][info   ][gc,heap,exit   ]   eden space 58880K, 4% used [0x00000007f5580000,0x00000007f5814680,0x00000007f8f00000)
[1.027s][info   ][gc,heap,exit   ]   from space 57856K, 0% used [0x00000007fc780000,0x00000007fc780000,0x0000000800000000)
[1.027s][info   ][gc,heap,exit   ]   to   space 57856K, 0% used [0x00000007f8f00000,0x00000007f8f00000,0x00000007fc780000)
[1.027s][info   ][gc,heap,exit   ]  ParOldGen       total 349696K, used 340647K [0x00000007e0000000, 0x00000007f5580000, 0x00000007f5580000)
[1.027s][info   ][gc,heap,exit   ]   object space 349696K, 97% used [0x00000007e0000000,0x00000007f4ca9c40,0x00000007f5580000)
[1.027s][info   ][gc,heap,exit   ]  Metaspace       used 232K, committed 448K, reserved 1114112K
[1.027s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```

再来试试1g内存
```
java -XX:+UseParallelGC -Xms1g -Xmx1g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

跟串行GC相比，GC次数增加，但是吞吐量也增加了很多，串行GC生成了33000多个对象，而并行GC生成了50000多个对象。有明显提升。
```
[0.002s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.005s][info   ][gc,init] CardTable entry size: 512
[0.005s][info   ][gc     ] Using Parallel
[0.005s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.005s][info   ][gc,init] CPUs: 8 total, 8 available
[0.005s][info   ][gc,init] Memory: 16384M
[0.005s][info   ][gc,init] Large Page Support: Disabled
[0.005s][info   ][gc,init] NUMA Support: Disabled
[0.005s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.005s][info   ][gc,init] Alignments: Space 512K, Generation 512K, Heap 8M
[0.005s][info   ][gc,init] Heap Min Capacity: 1G
[0.005s][info   ][gc,init] Heap Initial Capacity: 1G
[0.005s][info   ][gc,init] Heap Max Capacity: 1G
[0.005s][info   ][gc,init] Pre-touch: Disabled
[0.005s][info   ][gc,init] Parallel Workers: 8
[0.006s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.006s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.006s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.070s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.085s][info   ][gc,heap     ] GC(0) PSYoungGen: 262144K(305664K)->43515K(305664K) Eden: 262144K(262144K)->0K(262144K) From: 0K(43520K)->43515K(43520K)
[0.085s][info   ][gc,heap     ] GC(0) ParOldGen: 992K(699392K)->38587K(699392K)
[0.085s][info   ][gc,metaspace] GC(0) Metaspace: 155K(384K)->155K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.085s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 256M->80M(981M) 14.914ms
[0.085s][info   ][gc,cpu      ] GC(0) User=0.01s Sys=0.04s Real=0.02s
[0.112s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.144s][info   ][gc,heap     ] GC(1) PSYoungGen: 305659K(305664K)->43519K(305664K) Eden: 262144K(262144K)->0K(262144K) From: 43515K(43520K)->43519K(43520K)
[0.144s][info   ][gc,heap     ] GC(1) ParOldGen: 38587K(699392K)->123605K(699392K)
[0.144s][info   ][gc,metaspace] GC(1) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.144s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 336M->163M(981M) 32.202ms
[0.144s][info   ][gc,cpu      ] GC(1) User=0.02s Sys=0.08s Real=0.04s
[0.158s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.172s][info   ][gc,heap     ] GC(2) PSYoungGen: 305663K(305664K)->43517K(305664K) Eden: 262144K(262144K)->0K(262144K) From: 43519K(43520K)->43517K(43520K)
[0.172s][info   ][gc,heap     ] GC(2) ParOldGen: 123605K(699392K)->208282K(699392K)
[0.172s][info   ][gc,metaspace] GC(2) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.172s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 419M->245M(981M) 13.297ms
[0.172s][info   ][gc,cpu      ] GC(2) User=0.01s Sys=0.04s Real=0.01s
[0.185s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.197s][info   ][gc,heap     ] GC(3) PSYoungGen: 305661K(305664K)->43515K(305664K) Eden: 262144K(262144K)->0K(262144K) From: 43517K(43520K)->43515K(43520K)
[0.197s][info   ][gc,heap     ] GC(3) ParOldGen: 208282K(699392K)->278596K(699392K)
[0.197s][info   ][gc,metaspace] GC(3) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.197s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 501M->314M(981M) 12.068ms
[0.197s][info   ][gc,cpu      ] GC(3) User=0.01s Sys=0.04s Real=0.01s
[0.211s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.225s][info   ][gc,heap     ] GC(4) PSYoungGen: 305659K(305664K)->43503K(305664K) Eden: 262144K(262144K)->0K(262144K) From: 43515K(43520K)->43503K(43520K)
[0.225s][info   ][gc,heap     ] GC(4) ParOldGen: 278596K(699392K)->357837K(699392K)
[0.225s][info   ][gc,metaspace] GC(4) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.225s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 570M->391M(981M) 14.287ms
[0.225s][info   ][gc,cpu      ] GC(4) User=0.01s Sys=0.06s Real=0.02s
[0.239s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.252s][info   ][gc,heap     ] GC(5) PSYoungGen: 305647K(305664K)->43514K(160256K) Eden: 262144K(262144K)->0K(116736K) From: 43503K(43520K)->43514K(43520K)
[0.252s][info   ][gc,heap     ] GC(5) ParOldGen: 357837K(699392K)->436169K(699392K)
[0.252s][info   ][gc,metaspace] GC(5) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.252s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 647M->468M(839M) 13.030ms
[0.252s][info   ][gc,cpu      ] GC(5) User=0.01s Sys=0.04s Real=0.01s
[0.258s][info   ][gc,start    ] GC(6) Pause Young (Allocation Failure)
[0.261s][info   ][gc,heap     ] GC(6) PSYoungGen: 160085K(160256K)->74365K(232960K) Eden: 116571K(116736K)->0K(116736K) From: 43514K(43520K)->74365K(116224K)
[0.261s][info   ][gc,heap     ] GC(6) ParOldGen: 436169K(699392K)->440682K(699392K)
[0.261s][info   ][gc,metaspace] GC(6) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.261s][info   ][gc          ] GC(6) Pause Young (Allocation Failure) 582M->502M(910M) 2.826ms
[0.261s][info   ][gc,cpu      ] GC(6) User=0.01s Sys=0.00s Real=0.00s
[0.266s][info   ][gc,start    ] GC(7) Pause Young (Allocation Failure)
[0.269s][info   ][gc,heap     ] GC(7) PSYoungGen: 191101K(232960K)->103114K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 74365K(116224K)->103114K(116224K)
[0.269s][info   ][gc,heap     ] GC(7) ParOldGen: 440682K(699392K)->449904K(699392K)
[0.269s][info   ][gc,metaspace] GC(7) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.269s][info   ][gc          ] GC(7) Pause Young (Allocation Failure) 616M->540M(910M) 2.714ms
[0.269s][info   ][gc,cpu      ] GC(7) User=0.01s Sys=0.00s Real=0.00s
[0.274s][info   ][gc,start    ] GC(8) Pause Young (Allocation Failure)
[0.278s][info   ][gc,heap     ] GC(8) PSYoungGen: 219586K(232960K)->115563K(232960K) Eden: 116471K(116736K)->0K(116736K) From: 103114K(116224K)->115563K(116224K)
[0.278s][info   ][gc,heap     ] GC(8) ParOldGen: 449904K(699392K)->466909K(699392K)
[0.278s][info   ][gc,metaspace] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.278s][info   ][gc          ] GC(8) Pause Young (Allocation Failure) 653M->568M(910M) 3.646ms
[0.278s][info   ][gc,cpu      ] GC(8) User=0.01s Sys=0.01s Real=0.00s
[0.283s][info   ][gc,start    ] GC(9) Pause Young (Allocation Failure)
[0.293s][info   ][gc,heap     ] GC(9) PSYoungGen: 232299K(232960K)->82692K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 115563K(116224K)->82692K(116224K)
[0.293s][info   ][gc,heap     ] GC(9) ParOldGen: 466909K(699392K)->528082K(699392K)
[0.293s][info   ][gc,metaspace] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.293s][info   ][gc          ] GC(9) Pause Young (Allocation Failure) 682M->596M(910M) 9.914ms
[0.293s][info   ][gc,cpu      ] GC(9) User=0.01s Sys=0.02s Real=0.01s
[0.299s][info   ][gc,start    ] GC(10) Pause Young (Allocation Failure)
[0.313s][info   ][gc,heap     ] GC(10) PSYoungGen: 199428K(232960K)->45380K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 82692K(116224K)->45380K(116224K)
[0.313s][info   ][gc,heap     ] GC(10) ParOldGen: 528082K(699392K)->601186K(699392K)
[0.313s][info   ][gc,metaspace] GC(10) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.313s][info   ][gc          ] GC(10) Pause Young (Allocation Failure) 710M->631M(910M) 13.654ms
[0.313s][info   ][gc,cpu      ] GC(10) User=0.01s Sys=0.03s Real=0.01s
[0.313s][info   ][gc,start    ] GC(11) Pause Full (Ergonomics)
[0.313s][info   ][gc,phases,start] GC(11) Marking Phase
[0.320s][info   ][gc,phases      ] GC(11) Marking Phase 6.583ms
[0.320s][info   ][gc,phases,start] GC(11) Summary Phase
[0.321s][info   ][gc,phases      ] GC(11) Summary Phase 0.869ms
[0.321s][info   ][gc,phases,start] GC(11) Adjust Roots
[0.321s][info   ][gc,phases      ] GC(11) Adjust Roots 0.197ms
[0.321s][info   ][gc,phases,start] GC(11) Compaction Phase
[0.360s][info   ][gc,phases      ] GC(11) Compaction Phase 39.470ms
[0.360s][info   ][gc,phases,start] GC(11) Post Compact
[0.361s][info   ][gc,phases      ] GC(11) Post Compact 0.593ms
[0.361s][info   ][gc,heap        ] GC(11) PSYoungGen: 45380K(232960K)->0K(232960K) Eden: 0K(116736K)->0K(116736K) From: 45380K(116224K)->0K(116224K)
[0.361s][info   ][gc,heap        ] GC(11) ParOldGen: 601186K(699392K)->305448K(699392K)
[0.361s][info   ][gc,metaspace   ] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.361s][info   ][gc             ] GC(11) Pause Full (Ergonomics) 631M->298M(910M) 47.948ms
[0.361s][info   ][gc,cpu         ] GC(11) User=0.04s Sys=0.10s Real=0.05s
[0.367s][info   ][gc,start       ] GC(12) Pause Young (Allocation Failure)
[0.369s][info   ][gc,heap        ] GC(12) PSYoungGen: 116736K(232960K)->46339K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 0K(116224K)->46339K(116224K)
[0.369s][info   ][gc,heap        ] GC(12) ParOldGen: 305448K(699392K)->305448K(699392K)
[0.369s][info   ][gc,metaspace   ] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.369s][info   ][gc             ] GC(12) Pause Young (Allocation Failure) 412M->343M(910M) 1.555ms
[0.369s][info   ][gc,cpu         ] GC(12) User=0.00s Sys=0.00s Real=0.00s
[0.375s][info   ][gc,start       ] GC(13) Pause Young (Allocation Failure)
[0.377s][info   ][gc,heap        ] GC(13) PSYoungGen: 162419K(232960K)->49019K(232960K) Eden: 116079K(116736K)->0K(116736K) From: 46339K(116224K)->49019K(116224K)
[0.377s][info   ][gc,heap        ] GC(13) ParOldGen: 305448K(699392K)->346492K(699392K)
[0.377s][info   ][gc,metaspace   ] GC(13) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.377s][info   ][gc             ] GC(13) Pause Young (Allocation Failure) 456M->386M(910M) 2.714ms
[0.377s][info   ][gc,cpu         ] GC(13) User=0.01s Sys=0.00s Real=0.00s
[0.383s][info   ][gc,start       ] GC(14) Pause Young (Allocation Failure)
[0.385s][info   ][gc,heap        ] GC(14) PSYoungGen: 165755K(232960K)->41809K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 49019K(116224K)->41809K(116224K)
[0.385s][info   ][gc,heap        ] GC(14) ParOldGen: 346492K(699392K)->387467K(699392K)
[0.385s][info   ][gc,metaspace   ] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.385s][info   ][gc             ] GC(14) Pause Young (Allocation Failure) 500M->419M(910M) 2.585ms
[0.385s][info   ][gc,cpu         ] GC(14) User=0.02s Sys=0.00s Real=0.01s
[0.391s][info   ][gc,start       ] GC(15) Pause Young (Allocation Failure)
[0.393s][info   ][gc,heap        ] GC(15) PSYoungGen: 158545K(232960K)->46584K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 41809K(116224K)->46584K(116224K)
[0.393s][info   ][gc,heap        ] GC(15) ParOldGen: 387467K(699392K)->423884K(699392K)
[0.393s][info   ][gc,metaspace   ] GC(15) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.393s][info   ][gc             ] GC(15) Pause Young (Allocation Failure) 533M->459M(910M) 2.366ms
[0.393s][info   ][gc,cpu         ] GC(15) User=0.01s Sys=0.00s Real=0.00s
[0.399s][info   ][gc,start       ] GC(16) Pause Young (Allocation Failure)
[0.401s][info   ][gc,heap        ] GC(16) PSYoungGen: 163320K(232960K)->40142K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 46584K(116224K)->40142K(116224K)
[0.401s][info   ][gc,heap        ] GC(16) ParOldGen: 423884K(699392K)->462716K(699392K)
[0.401s][info   ][gc,metaspace   ] GC(16) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.401s][info   ][gc             ] GC(16) Pause Young (Allocation Failure) 573M->491M(910M) 2.467ms
[0.401s][info   ][gc,cpu         ] GC(16) User=0.01s Sys=0.01s Real=0.00s
[0.407s][info   ][gc,start       ] GC(17) Pause Young (Allocation Failure)
[0.409s][info   ][gc,heap        ] GC(17) PSYoungGen: 156878K(232960K)->43590K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 40142K(116224K)->43590K(116224K)
[0.409s][info   ][gc,heap        ] GC(17) ParOldGen: 462716K(699392K)->496672K(699392K)
[0.409s][info   ][gc,metaspace   ] GC(17) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.410s][info   ][gc             ] GC(17) Pause Young (Allocation Failure) 605M->527M(910M) 2.280ms
[0.410s][info   ][gc,cpu         ] GC(17) User=0.01s Sys=0.00s Real=0.00s
[0.416s][info   ][gc,start       ] GC(18) Pause Young (Allocation Failure)
[0.418s][info   ][gc,heap        ] GC(18) PSYoungGen: 160326K(232960K)->39611K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 43590K(116224K)->39611K(116224K)
[0.419s][info   ][gc,heap        ] GC(18) ParOldGen: 496672K(699392K)->536456K(699392K)
[0.419s][info   ][gc,metaspace   ] GC(18) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.419s][info   ][gc             ] GC(18) Pause Young (Allocation Failure) 641M->562M(910M) 2.493ms
[0.419s][info   ][gc,cpu         ] GC(18) User=0.01s Sys=0.00s Real=0.00s
[0.425s][info   ][gc,start       ] GC(19) Pause Young (Allocation Failure)
[0.427s][info   ][gc,heap        ] GC(19) PSYoungGen: 156347K(232960K)->46572K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 39611K(116224K)->46572K(116224K)
[0.427s][info   ][gc,heap        ] GC(19) ParOldGen: 536456K(699392K)->570411K(699392K)
[0.427s][info   ][gc,metaspace   ] GC(19) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.427s][info   ][gc             ] GC(19) Pause Young (Allocation Failure) 676M->602M(910M) 2.223ms
[0.427s][info   ][gc,cpu         ] GC(19) User=0.02s Sys=0.00s Real=0.00s
[0.433s][info   ][gc,start       ] GC(20) Pause Young (Allocation Failure)
[0.436s][info   ][gc,heap        ] GC(20) PSYoungGen: 163019K(232960K)->38093K(232960K) Eden: 116447K(116736K)->0K(116736K) From: 46572K(116224K)->38093K(116224K)
[0.436s][info   ][gc,heap        ] GC(20) ParOldGen: 570411K(699392K)->610440K(699392K)
[0.436s][info   ][gc,metaspace   ] GC(20) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.436s][info   ][gc             ] GC(20) Pause Young (Allocation Failure) 716M->633M(910M) 2.895ms
[0.436s][info   ][gc,cpu         ] GC(20) User=0.01s Sys=0.00s Real=0.00s
[0.442s][info   ][gc,start       ] GC(21) Pause Young (Allocation Failure)
[0.447s][info   ][gc,heap        ] GC(21) PSYoungGen: 154491K(232960K)->49732K(232960K) Eden: 116398K(116736K)->0K(116736K) From: 38093K(116224K)->49732K(116224K)
[0.447s][info   ][gc,heap        ] GC(21) ParOldGen: 610440K(699392K)->642556K(699392K)
[0.447s][info   ][gc,metaspace   ] GC(21) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.447s][info   ][gc             ] GC(21) Pause Young (Allocation Failure) 747M->676M(910M) 4.859ms
[0.447s][info   ][gc,cpu         ] GC(21) User=0.01s Sys=0.02s Real=0.01s
[0.447s][info   ][gc,start       ] GC(22) Pause Full (Ergonomics)
[0.447s][info   ][gc,phases,start] GC(22) Marking Phase
[0.450s][info   ][gc,phases      ] GC(22) Marking Phase 2.570ms
[0.450s][info   ][gc,phases,start] GC(22) Summary Phase
[0.450s][info   ][gc,phases      ] GC(22) Summary Phase 0.031ms
[0.450s][info   ][gc,phases,start] GC(22) Adjust Roots
[0.450s][info   ][gc,phases      ] GC(22) Adjust Roots 0.169ms
[0.450s][info   ][gc,phases,start] GC(22) Compaction Phase
[0.458s][info   ][gc,phases      ] GC(22) Compaction Phase 8.449ms
[0.458s][info   ][gc,phases,start] GC(22) Post Compact
[0.459s][info   ][gc,phases      ] GC(22) Post Compact 0.365ms
[0.459s][info   ][gc,heap        ] GC(22) PSYoungGen: 49732K(232960K)->0K(232960K) Eden: 0K(116736K)->0K(116736K) From: 49732K(116224K)->0K(116224K)
[0.459s][info   ][gc,heap        ] GC(22) ParOldGen: 642556K(699392K)->329548K(699392K)
[0.459s][info   ][gc,metaspace   ] GC(22) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.459s][info   ][gc             ] GC(22) Pause Full (Ergonomics) 676M->321M(910M) 11.804ms
[0.459s][info   ][gc,cpu         ] GC(22) User=0.03s Sys=0.00s Real=0.01s
[0.466s][info   ][gc,start       ] GC(23) Pause Young (Allocation Failure)
[0.468s][info   ][gc,heap        ] GC(23) PSYoungGen: 116736K(232960K)->51238K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 0K(116224K)->51238K(116224K)
[0.468s][info   ][gc,heap        ] GC(23) ParOldGen: 329548K(699392K)->329548K(699392K)
[0.468s][info   ][gc,metaspace   ] GC(23) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.468s][info   ][gc             ] GC(23) Pause Young (Allocation Failure) 435M->371M(910M) 2.526ms
[0.468s][info   ][gc,cpu         ] GC(23) User=0.00s Sys=0.00s Real=0.00s
[0.474s][info   ][gc,start       ] GC(24) Pause Young (Allocation Failure)
[0.476s][info   ][gc,heap        ] GC(24) PSYoungGen: 167896K(232960K)->39986K(232960K) Eden: 116658K(116736K)->0K(116736K) From: 51238K(116224K)->39986K(116224K)
[0.476s][info   ][gc,heap        ] GC(24) ParOldGen: 329548K(699392K)->372500K(699392K)
[0.476s][info   ][gc,metaspace   ] GC(24) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.476s][info   ][gc             ] GC(24) Pause Young (Allocation Failure) 485M->402M(910M) 2.028ms
[0.476s][info   ][gc,cpu         ] GC(24) User=0.01s Sys=0.00s Real=0.00s
[0.482s][info   ][gc,start       ] GC(25) Pause Young (Allocation Failure)
[0.485s][info   ][gc,heap        ] GC(25) PSYoungGen: 156653K(232960K)->48228K(232960K) Eden: 116666K(116736K)->0K(116736K) From: 39986K(116224K)->48228K(116224K)
[0.485s][info   ][gc,heap        ] GC(25) ParOldGen: 372500K(699392K)->407933K(699392K)
[0.485s][info   ][gc,metaspace   ] GC(25) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.485s][info   ][gc             ] GC(25) Pause Young (Allocation Failure) 516M->445M(910M) 2.561ms
[0.485s][info   ][gc,cpu         ] GC(25) User=0.01s Sys=0.00s Real=0.01s
[0.490s][info   ][gc,start       ] GC(26) Pause Young (Allocation Failure)
[0.492s][info   ][gc,heap        ] GC(26) PSYoungGen: 164964K(232960K)->46375K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 48228K(116224K)->46375K(116224K)
[0.492s][info   ][gc,heap        ] GC(26) ParOldGen: 407933K(699392K)->452835K(699392K)
[0.492s][info   ][gc,metaspace   ] GC(26) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.492s][info   ][gc             ] GC(26) Pause Young (Allocation Failure) 559M->487M(910M) 2.187ms
[0.492s][info   ][gc,cpu         ] GC(26) User=0.01s Sys=0.00s Real=0.00s
[0.499s][info   ][gc,start       ] GC(27) Pause Young (Allocation Failure)
[0.501s][info   ][gc,heap        ] GC(27) PSYoungGen: 162793K(232960K)->45280K(232960K) Eden: 116417K(116736K)->0K(116736K) From: 46375K(116224K)->45280K(116224K)
[0.501s][info   ][gc,heap        ] GC(27) ParOldGen: 452835K(699392K)->492807K(699392K)
[0.501s][info   ][gc,metaspace   ] GC(27) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.501s][info   ][gc             ] GC(27) Pause Young (Allocation Failure) 601M->525M(910M) 2.327ms
[0.501s][info   ][gc,cpu         ] GC(27) User=0.01s Sys=0.00s Real=0.00s
[0.508s][info   ][gc,start       ] GC(28) Pause Young (Allocation Failure)
[0.510s][info   ][gc,heap        ] GC(28) PSYoungGen: 162016K(232960K)->50138K(232960K) Eden: 116736K(116736K)->0K(116736K) From: 45280K(116224K)->50138K(116224K)
[0.510s][info   ][gc,heap        ] GC(28) ParOldGen: 492807K(699392K)->531392K(699392K)
[0.510s][info   ][gc,metaspace   ] GC(28) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.510s][info   ][gc             ] GC(28) Pause Young (Allocation Failure) 639M->567M(910M) 2.295ms
[0.510s][info   ][gc,cpu         ] GC(28) User=0.01s Sys=0.00s Real=0.00s
[0.515s][info   ][gc,start       ] GC(29) Pause Young (Allocation Failure)
[0.518s][info   ][gc,heap        ] GC(29) PSYoungGen: 166874K(232960K)->41125K(239104K) Eden: 116736K(116736K)->0K(125440K) From: 50138K(116224K)->41125K(113664K)
[0.518s][info   ][gc,heap        ] GC(29) ParOldGen: 531392K(699392K)->575382K(699392K)
[0.518s][info   ][gc,metaspace   ] GC(29) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.518s][info   ][gc             ] GC(29) Pause Young (Allocation Failure) 681M->602M(916M) 2.468ms
[0.518s][info   ][gc,cpu         ] GC(29) User=0.01s Sys=0.00s Real=0.00s
[0.524s][info   ][gc,start       ] GC(30) Pause Young (Allocation Failure)
[0.526s][info   ][gc,heap        ] GC(30) PSYoungGen: 166565K(239104K)->52109K(235520K) Eden: 125440K(125440K)->0K(125440K) From: 41125K(113664K)->52109K(110080K)
[0.526s][info   ][gc,heap        ] GC(30) ParOldGen: 575382K(699392K)->610145K(699392K)
[0.526s][info   ][gc,metaspace   ] GC(30) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.526s][info   ][gc             ] GC(30) Pause Young (Allocation Failure) 724M->646M(913M) 2.349ms
[0.526s][info   ][gc,cpu         ] GC(30) User=0.01s Sys=0.00s Real=0.00s
[0.532s][info   ][gc,start       ] GC(31) Pause Young (Allocation Failure)
[0.536s][info   ][gc,heap        ] GC(31) PSYoungGen: 177549K(235520K)->48950K(248320K) Eden: 125440K(125440K)->0K(143360K) From: 52109K(110080K)->48950K(104960K)
[0.536s][info   ][gc,heap        ] GC(31) ParOldGen: 610145K(699392K)->653512K(699392K)
[0.536s][info   ][gc,metaspace   ] GC(31) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.536s][info   ][gc             ] GC(31) Pause Young (Allocation Failure) 769M->685M(925M) 3.575ms
[0.536s][info   ][gc,cpu         ] GC(31) User=0.01s Sys=0.00s Real=0.01s
[0.536s][info   ][gc,start       ] GC(32) Pause Full (Ergonomics)
[0.536s][info   ][gc,phases,start] GC(32) Marking Phase
[0.538s][info   ][gc,phases      ] GC(32) Marking Phase 1.833ms
[0.538s][info   ][gc,phases,start] GC(32) Summary Phase
[0.538s][info   ][gc,phases      ] GC(32) Summary Phase 0.018ms
[0.538s][info   ][gc,phases,start] GC(32) Adjust Roots
[0.538s][info   ][gc,phases      ] GC(32) Adjust Roots 0.463ms
[0.538s][info   ][gc,phases,start] GC(32) Compaction Phase
[0.545s][info   ][gc,phases      ] GC(32) Compaction Phase 7.015ms
[0.545s][info   ][gc,phases,start] GC(32) Post Compact
[0.546s][info   ][gc,phases      ] GC(32) Post Compact 0.327ms
[0.546s][info   ][gc,heap        ] GC(32) PSYoungGen: 48950K(248320K)->0K(244224K) Eden: 0K(143360K)->0K(143360K) From: 48950K(104960K)->0K(100864K)
[0.546s][info   ][gc,heap        ] GC(32) ParOldGen: 653512K(699392K)->360191K(699392K)
[0.546s][info   ][gc,metaspace   ] GC(32) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.546s][info   ][gc             ] GC(32) Pause Full (Ergonomics) 685M->351M(921M) 9.868ms
[0.546s][info   ][gc,cpu         ] GC(32) User=0.03s Sys=0.01s Real=0.01s
[0.556s][info   ][gc,start       ] GC(33) Pause Young (Allocation Failure)
[0.558s][info   ][gc,heap        ] GC(33) PSYoungGen: 143272K(244224K)->54984K(252416K) Eden: 143272K(143360K)->0K(151552K) From: 0K(100864K)->54984K(100864K)
[0.558s][info   ][gc,heap        ] GC(33) ParOldGen: 360191K(699392K)->360191K(699392K)
[0.558s][info   ][gc,metaspace   ] GC(33) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.558s][info   ][gc             ] GC(33) Pause Young (Allocation Failure) 491M->405M(929M) 1.840ms
[0.558s][info   ][gc,cpu         ] GC(33) User=0.00s Sys=0.00s Real=0.00s
[0.568s][info   ][gc,start       ] GC(34) Pause Young (Allocation Failure)
[0.571s][info   ][gc,heap        ] GC(34) PSYoungGen: 206536K(252416K)->49881K(248320K) Eden: 151552K(151552K)->0K(151552K) From: 54984K(100864K)->49881K(96768K)
[0.571s][info   ][gc,heap        ] GC(34) ParOldGen: 360191K(699392K)->407788K(699392K)
[0.571s][info   ][gc,metaspace   ] GC(34) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.571s][info   ][gc             ] GC(34) Pause Young (Allocation Failure) 553M->446M(925M) 3.013ms
[0.571s][info   ][gc,cpu         ] GC(34) User=0.01s Sys=0.00s Real=0.00s
[0.579s][info   ][gc,start       ] GC(35) Pause Young (Allocation Failure)
[0.581s][info   ][gc,heap        ] GC(35) PSYoungGen: 201433K(248320K)->56576K(259072K) Eden: 151552K(151552K)->0K(165888K) From: 49881K(96768K)->56576K(93184K)
[0.581s][info   ][gc,heap        ] GC(35) ParOldGen: 407788K(699392K)->449784K(699392K)
[0.581s][info   ][gc,metaspace   ] GC(35) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.581s][info   ][gc             ] GC(35) Pause Young (Allocation Failure) 594M->494M(936M) 2.424ms
[0.581s][info   ][gc,cpu         ] GC(35) User=0.01s Sys=0.00s Real=0.00s
[0.589s][info   ][gc,start       ] GC(36) Pause Young (Allocation Failure)
[0.591s][info   ][gc,heap        ] GC(36) PSYoungGen: 222464K(259072K)->61935K(256000K) Eden: 165888K(165888K)->0K(165888K) From: 56576K(93184K)->61935K(90112K)
[0.592s][info   ][gc,heap        ] GC(36) ParOldGen: 449784K(699392K)->496669K(699392K)
[0.592s][info   ][gc,metaspace   ] GC(36) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.592s][info   ][gc             ] GC(36) Pause Young (Allocation Failure) 656M->545M(933M) 2.855ms
[0.592s][info   ][gc,cpu         ] GC(36) User=0.01s Sys=0.00s Real=0.00s
[0.599s][info   ][gc,start       ] GC(37) Pause Young (Allocation Failure)
[0.602s][info   ][gc,heap        ] GC(37) PSYoungGen: 227823K(256000K)->57373K(262144K) Eden: 165888K(165888K)->0K(172544K) From: 61935K(90112K)->57373K(89600K)
[0.602s][info   ][gc,heap        ] GC(37) ParOldGen: 496669K(699392K)->547969K(699392K)
[0.602s][info   ][gc,metaspace   ] GC(37) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.602s][info   ][gc             ] GC(37) Pause Young (Allocation Failure) 707M->591M(939M) 2.692ms
[0.602s][info   ][gc,cpu         ] GC(37) User=0.01s Sys=0.00s Real=0.00s
[0.610s][info   ][gc,start       ] GC(38) Pause Young (Allocation Failure)
[0.612s][info   ][gc,heap        ] GC(38) PSYoungGen: 229917K(262144K)->60373K(259584K) Eden: 172544K(172544K)->0K(172544K) From: 57373K(89600K)->60373K(87040K)
[0.612s][info   ][gc,heap        ] GC(38) ParOldGen: 547969K(699392K)->596538K(699392K)
[0.612s][info   ][gc,metaspace   ] GC(38) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.612s][info   ][gc             ] GC(38) Pause Young (Allocation Failure) 759M->641M(936M) 2.529ms
[0.612s][info   ][gc,cpu         ] GC(38) User=0.01s Sys=0.00s Real=0.00s
[0.620s][info   ][gc,start       ] GC(39) Pause Young (Allocation Failure)
[0.623s][info   ][gc,heap        ] GC(39) PSYoungGen: 232917K(259584K)->61240K(264192K) Eden: 172544K(172544K)->0K(178176K) From: 60373K(87040K)->61240K(86016K)
[0.623s][info   ][gc,heap        ] GC(39) ParOldGen: 596538K(699392K)->646693K(699392K)
[0.623s][info   ][gc,metaspace   ] GC(39) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.623s][info   ][gc             ] GC(39) Pause Young (Allocation Failure) 810M->691M(941M) 3.024ms
[0.623s][info   ][gc,cpu         ] GC(39) User=0.01s Sys=0.00s Real=0.00s
[0.623s][info   ][gc,start       ] GC(40) Pause Full (Ergonomics)
[0.623s][info   ][gc,phases,start] GC(40) Marking Phase
[0.625s][info   ][gc,phases      ] GC(40) Marking Phase 1.908ms
[0.625s][info   ][gc,phases,start] GC(40) Summary Phase
[0.625s][info   ][gc,phases      ] GC(40) Summary Phase 0.027ms
[0.625s][info   ][gc,phases,start] GC(40) Adjust Roots
[0.625s][info   ][gc,phases      ] GC(40) Adjust Roots 0.168ms
[0.625s][info   ][gc,phases,start] GC(40) Compaction Phase
[0.632s][info   ][gc,phases      ] GC(40) Compaction Phase 6.803ms
[0.632s][info   ][gc,phases,start] GC(40) Post Compact
[0.632s][info   ][gc,phases      ] GC(40) Post Compact 0.261ms
[0.632s][info   ][gc,heap        ] GC(40) PSYoungGen: 61240K(264192K)->0K(263168K) Eden: 0K(178176K)->0K(178176K) From: 61240K(86016K)->0K(84992K)
[0.632s][info   ][gc,heap        ] GC(40) ParOldGen: 646693K(699392K)->353610K(699392K)
[0.632s][info   ][gc,metaspace   ] GC(40) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.632s][info   ][gc             ] GC(40) Pause Full (Ergonomics) 691M->345M(940M) 9.320ms
[0.632s][info   ][gc,cpu         ] GC(40) User=0.03s Sys=0.00s Real=0.01s
[0.641s][info   ][gc,start       ] GC(41) Pause Young (Allocation Failure)
[0.643s][info   ][gc,heap        ] GC(41) PSYoungGen: 177868K(263168K)->60186K(265216K) Eden: 177868K(178176K)->0K(180224K) From: 0K(84992K)->60186K(84992K)
[0.643s][info   ][gc,heap        ] GC(41) ParOldGen: 353610K(699392K)->353610K(699392K)
[0.643s][info   ][gc,metaspace   ] GC(41) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.643s][info   ][gc             ] GC(41) Pause Young (Allocation Failure) 519M->404M(942M) 2.137ms
[0.643s][info   ][gc,cpu         ] GC(41) User=0.01s Sys=0.01s Real=0.00s
[0.653s][info   ][gc,start       ] GC(42) Pause Young (Allocation Failure)
[0.657s][info   ][gc,heap        ] GC(42) PSYoungGen: 240410K(265216K)->64911K(264192K) Eden: 180224K(180224K)->0K(180224K) From: 60186K(84992K)->64911K(83968K)
[0.657s][info   ][gc,heap        ] GC(42) ParOldGen: 353610K(699392K)->400995K(699392K)
[0.657s][info   ][gc,metaspace   ] GC(42) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.657s][info   ][gc             ] GC(42) Pause Young (Allocation Failure) 580M->454M(941M) 3.931ms
[0.657s][info   ][gc,cpu         ] GC(42) User=0.01s Sys=0.00s Real=0.01s
[0.665s][info   ][gc,start       ] GC(43) Pause Young (Allocation Failure)
[0.668s][info   ][gc,heap        ] GC(43) PSYoungGen: 245135K(264192K)->65593K(264704K) Eden: 180224K(180224K)->0K(180224K) From: 64911K(83968K)->65593K(84480K)
[0.668s][info   ][gc,heap        ] GC(43) ParOldGen: 400995K(699392K)->452628K(699392K)
[0.668s][info   ][gc,metaspace   ] GC(43) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.668s][info   ][gc             ] GC(43) Pause Young (Allocation Failure) 630M->506M(941M) 3.067ms
[0.668s][info   ][gc,cpu         ] GC(43) User=0.02s Sys=0.00s Real=0.00s
[0.676s][info   ][gc,start       ] GC(44) Pause Young (Allocation Failure)
[0.679s][info   ][gc,heap        ] GC(44) PSYoungGen: 245817K(264704K)->66009K(246272K) Eden: 180224K(180224K)->0K(180224K) From: 65593K(84480K)->66009K(66048K)
[0.679s][info   ][gc,heap        ] GC(44) ParOldGen: 452628K(699392K)->502125K(699392K)
[0.679s][info   ][gc,metaspace   ] GC(44) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.679s][info   ][gc             ] GC(44) Pause Young (Allocation Failure) 682M->554M(923M) 2.990ms
[0.679s][info   ][gc,cpu         ] GC(44) User=0.02s Sys=0.00s Real=0.00s
[0.689s][info   ][gc,start       ] GC(45) Pause Young (Allocation Failure)
[0.696s][info   ][gc,heap        ] GC(45) PSYoungGen: 246184K(246272K)->65652K(264192K) Eden: 180174K(180224K)->0K(179200K) From: 66009K(66048K)->65652K(84992K)
[0.696s][info   ][gc,heap        ] GC(45) ParOldGen: 502125K(699392K)->555658K(699392K)
[0.696s][info   ][gc,metaspace   ] GC(45) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.697s][info   ][gc             ] GC(45) Pause Young (Allocation Failure) 730M->606M(941M) 7.331ms
[0.697s][info   ][gc,cpu         ] GC(45) User=0.02s Sys=0.00s Real=0.01s
[0.705s][info   ][gc,start       ] GC(46) Pause Young (Allocation Failure)
[0.708s][info   ][gc,heap        ] GC(46) PSYoungGen: 244852K(264192K)->63211K(264192K) Eden: 179200K(179200K)->0K(179200K) From: 65652K(84992K)->63211K(84992K)
[0.708s][info   ][gc,heap        ] GC(46) ParOldGen: 555658K(699392K)->610839K(699392K)
[0.708s][info   ][gc,metaspace   ] GC(46) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.708s][info   ][gc             ] GC(46) Pause Young (Allocation Failure) 781M->658M(941M) 3.544ms
[0.708s][info   ][gc,cpu         ] GC(46) User=0.02s Sys=0.00s Real=0.00s
[0.717s][info   ][gc,start       ] GC(47) Pause Young (Allocation Failure)
[0.720s][info   ][gc,heap        ] GC(47) PSYoungGen: 242378K(264192K)->64949K(265728K) Eden: 179166K(179200K)->0K(181760K) From: 63211K(84992K)->64949K(83968K)
[0.720s][info   ][gc,heap        ] GC(47) ParOldGen: 610839K(699392K)->661578K(699392K)
[0.720s][info   ][gc,metaspace   ] GC(47) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.720s][info   ][gc             ] GC(47) Pause Young (Allocation Failure) 833M->709M(942M) 3.314ms
[0.720s][info   ][gc,cpu         ] GC(47) User=0.02s Sys=0.00s Real=0.00s
[0.720s][info   ][gc,start       ] GC(48) Pause Full (Ergonomics)
[0.720s][info   ][gc,phases,start] GC(48) Marking Phase
[0.722s][info   ][gc,phases      ] GC(48) Marking Phase 2.200ms
[0.722s][info   ][gc,phases,start] GC(48) Summary Phase
[0.722s][info   ][gc,phases      ] GC(48) Summary Phase 0.027ms
[0.722s][info   ][gc,phases,start] GC(48) Adjust Roots
[0.723s][info   ][gc,phases      ] GC(48) Adjust Roots 0.218ms
[0.723s][info   ][gc,phases,start] GC(48) Compaction Phase
[0.729s][info   ][gc,phases      ] GC(48) Compaction Phase 6.742ms
[0.729s][info   ][gc,phases,start] GC(48) Post Compact
[0.730s][info   ][gc,phases      ] GC(48) Post Compact 0.278ms
[0.730s][info   ][gc,heap        ] GC(48) PSYoungGen: 64949K(265728K)->0K(265216K) Eden: 0K(181760K)->0K(181760K) From: 64949K(83968K)->0K(83456K)
[0.730s][info   ][gc,heap        ] GC(48) ParOldGen: 661578K(699392K)->353965K(699392K)
[0.730s][info   ][gc,metaspace   ] GC(48) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.730s][info   ][gc             ] GC(48) Pause Full (Ergonomics) 709M->345M(942M) 9.648ms
[0.730s][info   ][gc,cpu         ] GC(48) User=0.03s Sys=0.00s Real=0.01s
[0.739s][info   ][gc,start       ] GC(49) Pause Young (Allocation Failure)
[0.740s][info   ][gc,heap        ] GC(49) PSYoungGen: 181290K(265216K)->66108K(266240K) Eden: 181290K(181760K)->0K(182784K) From: 0K(83456K)->66108K(83456K)
[0.741s][info   ][gc,heap        ] GC(49) ParOldGen: 353965K(699392K)->353965K(699392K)
[0.741s][info   ][gc,metaspace   ] GC(49) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.741s][info   ][gc             ] GC(49) Pause Young (Allocation Failure) 522M->410M(943M) 1.834ms
[0.741s][info   ][gc,cpu         ] GC(49) User=0.01s Sys=0.00s Real=0.00s
[0.749s][info   ][gc,start       ] GC(50) Pause Young (Allocation Failure)
[0.751s][info   ][gc,heap        ] GC(50) PSYoungGen: 248892K(266240K)->72180K(254976K) Eden: 182784K(182784K)->0K(182784K) From: 66108K(83456K)->72180K(72192K)
[0.751s][info   ][gc,heap        ] GC(50) ParOldGen: 353965K(699392K)->403126K(699392K)
[0.751s][info   ][gc,metaspace   ] GC(50) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.751s][info   ][gc             ] GC(50) Pause Young (Allocation Failure) 588M->464M(932M) 2.622ms
[0.751s][info   ][gc,cpu         ] GC(50) User=0.01s Sys=0.00s Real=0.00s
[0.760s][info   ][gc,start       ] GC(51) Pause Young (Allocation Failure)
[0.764s][info   ][gc,heap        ] GC(51) PSYoungGen: 254964K(254976K)->66763K(264704K) Eden: 182784K(182784K)->0K(179712K) From: 72180K(72192K)->66763K(84992K)
[0.764s][info   ][gc,heap        ] GC(51) ParOldGen: 403126K(699392K)->461598K(699392K)
[0.764s][info   ][gc,metaspace   ] GC(51) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.764s][info   ][gc             ] GC(51) Pause Young (Allocation Failure) 642M->515M(941M) 3.769ms
[0.764s][info   ][gc,cpu         ] GC(51) User=0.01s Sys=0.00s Real=0.01s
[0.778s][info   ][gc,start       ] GC(52) Pause Young (Allocation Failure)
[0.781s][info   ][gc,heap        ] GC(52) PSYoungGen: 246475K(264704K)->64251K(264192K) Eden: 179712K(179712K)->0K(179712K) From: 66763K(84992K)->64251K(84480K)
[0.781s][info   ][gc,heap        ] GC(52) ParOldGen: 461598K(699392K)->514402K(699392K)
[0.781s][info   ][gc,metaspace   ] GC(52) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.781s][info   ][gc             ] GC(52) Pause Young (Allocation Failure) 691M->565M(941M) 2.976ms
[0.781s][info   ][gc,cpu         ] GC(52) User=0.01s Sys=0.00s Real=0.00s
[0.789s][info   ][gc,start       ] GC(53) Pause Young (Allocation Failure)
[0.793s][info   ][gc,heap        ] GC(53) PSYoungGen: 243963K(264192K)->66862K(266752K) Eden: 179712K(179712K)->0K(183808K) From: 64251K(84480K)->66862K(82944K)
[0.793s][info   ][gc,heap        ] GC(53) ParOldGen: 514402K(699392K)->562241K(699392K)
[0.793s][info   ][gc,metaspace   ] GC(53) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.793s][info   ][gc             ] GC(53) Pause Young (Allocation Failure) 740M->614M(943M) 3.696ms
[0.793s][info   ][gc,cpu         ] GC(53) User=0.01s Sys=0.00s Real=0.00s
[0.801s][info   ][gc,start       ] GC(54) Pause Young (Allocation Failure)
[0.804s][info   ][gc,heap        ] GC(54) PSYoungGen: 250670K(266752K)->69836K(266240K) Eden: 183808K(183808K)->0K(183808K) From: 66862K(82944K)->69836K(82432K)
[0.804s][info   ][gc,heap        ] GC(54) ParOldGen: 562241K(699392K)->617792K(699392K)
[0.804s][info   ][gc,metaspace   ] GC(54) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.804s][info   ][gc             ] GC(54) Pause Young (Allocation Failure) 793M->671M(943M) 3.001ms
[0.804s][info   ][gc,cpu         ] GC(54) User=0.01s Sys=0.00s Real=0.01s
[0.813s][info   ][gc,start       ] GC(55) Pause Young (Allocation Failure)
[0.817s][info   ][gc,heap        ] GC(55) PSYoungGen: 253644K(266240K)->71613K(264192K) Eden: 183808K(183808K)->0K(181248K) From: 69836K(82432K)->71613K(82944K)
[0.817s][info   ][gc,heap        ] GC(55) ParOldGen: 617792K(699392K)->672132K(699392K)
[0.817s][info   ][gc,metaspace   ] GC(55) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.817s][info   ][gc             ] GC(55) Pause Young (Allocation Failure) 851M->726M(941M) 4.353ms
[0.817s][info   ][gc,cpu         ] GC(55) User=0.01s Sys=0.01s Real=0.01s
[0.817s][info   ][gc,start       ] GC(56) Pause Full (Ergonomics)
[0.817s][info   ][gc,phases,start] GC(56) Marking Phase
[0.819s][info   ][gc,phases      ] GC(56) Marking Phase 1.712ms
[0.819s][info   ][gc,phases,start] GC(56) Summary Phase
[0.819s][info   ][gc,phases      ] GC(56) Summary Phase 0.027ms
[0.819s][info   ][gc,phases,start] GC(56) Adjust Roots
[0.819s][info   ][gc,phases      ] GC(56) Adjust Roots 0.148ms
[0.819s][info   ][gc,phases,start] GC(56) Compaction Phase
[0.825s][info   ][gc,phases      ] GC(56) Compaction Phase 6.432ms
[0.826s][info   ][gc,phases,start] GC(56) Post Compact
[0.826s][info   ][gc,phases      ] GC(56) Post Compact 0.284ms
[0.826s][info   ][gc,heap        ] GC(56) PSYoungGen: 71613K(264192K)->0K(265216K) Eden: 0K(181248K)->0K(181248K) From: 71613K(82944K)->0K(83968K)
[0.826s][info   ][gc,heap        ] GC(56) ParOldGen: 672132K(699392K)->354796K(699392K)
[0.826s][info   ][gc,metaspace   ] GC(56) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.826s][info   ][gc             ] GC(56) Pause Full (Ergonomics) 726M->346M(942M) 8.802ms
[0.826s][info   ][gc,cpu         ] GC(56) User=0.04s Sys=0.00s Real=0.01s
[0.835s][info   ][gc,start       ] GC(57) Pause Young (Allocation Failure)
[0.836s][info   ][gc,heap        ] GC(57) PSYoungGen: 181248K(265216K)->73337K(262144K) Eden: 181248K(181248K)->0K(178176K) From: 0K(83968K)->73337K(83968K)
[0.836s][info   ][gc,heap        ] GC(57) ParOldGen: 354796K(699392K)->354796K(699392K)
[0.836s][info   ][gc,metaspace   ] GC(57) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.836s][info   ][gc             ] GC(57) Pause Young (Allocation Failure) 523M->418M(939M) 1.907ms
[0.836s][info   ][gc,cpu         ] GC(57) User=0.00s Sys=0.00s Real=0.00s
[0.845s][info   ][gc,start       ] GC(58) Pause Young (Allocation Failure)
[0.847s][info   ][gc,heap        ] GC(58) PSYoungGen: 251513K(262144K)->64626K(263680K) Eden: 178176K(178176K)->0K(178176K) From: 73337K(83968K)->64626K(85504K)
[0.847s][info   ][gc,heap        ] GC(58) ParOldGen: 354796K(699392K)->413600K(699392K)
[0.847s][info   ][gc,metaspace   ] GC(58) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.847s][info   ][gc             ] GC(58) Pause Young (Allocation Failure) 592M->467M(940M) 2.891ms
[0.847s][info   ][gc,cpu         ] GC(58) User=0.01s Sys=0.00s Real=0.00s
[0.857s][info   ][gc,start       ] GC(59) Pause Young (Allocation Failure)
[0.859s][info   ][gc,heap        ] GC(59) PSYoungGen: 242798K(263680K)->59709K(266240K) Eden: 178171K(178176K)->0K(182784K) From: 64626K(85504K)->59709K(83456K)
[0.859s][info   ][gc,heap        ] GC(59) ParOldGen: 413600K(699392K)->465864K(699392K)
[0.859s][info   ][gc,metaspace   ] GC(59) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.859s][info   ][gc             ] GC(59) Pause Young (Allocation Failure) 641M->513M(943M) 2.607ms
[0.859s][info   ][gc,cpu         ] GC(59) User=0.01s Sys=0.00s Real=0.00s
[0.868s][info   ][gc,start       ] GC(60) Pause Young (Allocation Failure)
[0.870s][info   ][gc,heap        ] GC(60) PSYoungGen: 241907K(266240K)->64716K(265728K) Eden: 182198K(182784K)->0K(182784K) From: 59709K(83456K)->64716K(82944K)
[0.870s][info   ][gc,heap        ] GC(60) ParOldGen: 465864K(699392K)->518404K(699392K)
[0.870s][info   ][gc,metaspace   ] GC(60) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.870s][info   ][gc             ] GC(60) Pause Young (Allocation Failure) 691M->569M(942M) 2.742ms
[0.870s][info   ][gc,cpu         ] GC(60) User=0.02s Sys=0.00s Real=0.00s
[0.878s][info   ][gc,start       ] GC(61) Pause Young (Allocation Failure)
[0.882s][info   ][gc,heap        ] GC(61) PSYoungGen: 247263K(265728K)->70965K(267264K) Eden: 182546K(182784K)->0K(186368K) From: 64716K(82944K)->70965K(80896K)
[0.882s][info   ][gc,heap        ] GC(61) ParOldGen: 518404K(699392K)->572494K(699392K)
[0.882s][info   ][gc,metaspace   ] GC(61) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.882s][info   ][gc             ] GC(61) Pause Young (Allocation Failure) 747M->628M(944M) 3.305ms
[0.882s][info   ][gc,cpu         ] GC(61) User=0.02s Sys=0.00s Real=0.00s
[0.891s][info   ][gc,start       ] GC(62) Pause Young (Allocation Failure)
[0.894s][info   ][gc,heap        ] GC(62) PSYoungGen: 257266K(267264K)->69320K(267776K) Eden: 186301K(186368K)->0K(186368K) From: 70965K(80896K)->69320K(81408K)
[0.894s][info   ][gc,heap        ] GC(62) ParOldGen: 572494K(699392K)->626234K(699392K)
[0.894s][info   ][gc,metaspace   ] GC(62) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.894s][info   ][gc             ] GC(62) Pause Young (Allocation Failure) 810M->679M(944M) 2.927ms
[0.894s][info   ][gc,cpu         ] GC(62) User=0.01s Sys=0.00s Real=0.01s
[0.894s][info   ][gc,start       ] GC(63) Pause Full (Ergonomics)
[0.894s][info   ][gc,phases,start] GC(63) Marking Phase
[0.896s][info   ][gc,phases      ] GC(63) Marking Phase 1.986ms
[0.896s][info   ][gc,phases,start] GC(63) Summary Phase
[0.896s][info   ][gc,phases      ] GC(63) Summary Phase 0.027ms
[0.896s][info   ][gc,phases,start] GC(63) Adjust Roots
[0.896s][info   ][gc,phases      ] GC(63) Adjust Roots 0.169ms
[0.896s][info   ][gc,phases,start] GC(63) Compaction Phase
[0.903s][info   ][gc,phases      ] GC(63) Compaction Phase 6.331ms
[0.903s][info   ][gc,phases,start] GC(63) Post Compact
[0.903s][info   ][gc,phases      ] GC(63) Post Compact 0.282ms
[0.903s][info   ][gc,heap        ] GC(63) PSYoungGen: 69320K(267776K)->0K(267776K) Eden: 0K(186368K)->0K(186368K) From: 69320K(81408K)->0K(81408K)
[0.903s][info   ][gc,heap        ] GC(63) ParOldGen: 626234K(699392K)->348289K(699392K)
[0.903s][info   ][gc,metaspace   ] GC(63) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.903s][info   ][gc             ] GC(63) Pause Full (Ergonomics) 679M->340M(944M) 8.978ms
[0.903s][info   ][gc,cpu         ] GC(63) User=0.03s Sys=0.00s Real=0.00s
[0.912s][info   ][gc,start       ] GC(64) Pause Young (Allocation Failure)
[0.914s][info   ][gc,heap        ] GC(64) PSYoungGen: 185978K(267776K)->63872K(267776K) Eden: 185978K(186368K)->0K(186368K) From: 0K(81408K)->63872K(81408K)
[0.914s][info   ][gc,heap        ] GC(64) ParOldGen: 348289K(699392K)->348289K(699392K)
[0.914s][info   ][gc,metaspace   ] GC(64) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.914s][info   ][gc             ] GC(64) Pause Young (Allocation Failure) 521M->402M(944M) 1.973ms
[0.914s][info   ][gc,cpu         ] GC(64) User=0.01s Sys=0.00s Real=0.01s
[0.923s][info   ][gc,start       ] GC(65) Pause Young (Allocation Failure)
[0.925s][info   ][gc,heap        ] GC(65) PSYoungGen: 250191K(267776K)->63711K(269824K) Eden: 186319K(186368K)->0K(189440K) From: 63872K(81408K)->63711K(80384K)
[0.925s][info   ][gc,heap        ] GC(65) ParOldGen: 348289K(699392K)->400820K(699392K)
[0.925s][info   ][gc,metaspace   ] GC(65) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.925s][info   ][gc             ] GC(65) Pause Young (Allocation Failure) 584M->453M(946M) 2.612ms
[0.925s][info   ][gc,cpu         ] GC(65) User=0.01s Sys=0.00s Real=0.01s
[0.935s][info   ][gc,start       ] GC(66) Pause Young (Allocation Failure)
[0.937s][info   ][gc,heap        ] GC(66) PSYoungGen: 253151K(269824K)->66375K(268800K) Eden: 189440K(189440K)->0K(189440K) From: 63711K(80384K)->66375K(79360K)
[0.937s][info   ][gc,heap        ] GC(66) ParOldGen: 400820K(699392K)->454260K(699392K)
[0.937s][info   ][gc,metaspace   ] GC(66) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.937s][info   ][gc             ] GC(66) Pause Young (Allocation Failure) 638M->508M(945M) 2.595ms
[0.937s][info   ][gc,cpu         ] GC(66) User=0.01s Sys=0.00s Real=0.00s
[0.949s][info   ][gc,start       ] GC(67) Pause Young (Allocation Failure)
[0.954s][info   ][gc,heap        ] GC(67) PSYoungGen: 255320K(268800K)->69962K(270848K) Eden: 188944K(189440K)->0K(192512K) From: 66375K(79360K)->69962K(78336K)
[0.954s][info   ][gc,heap        ] GC(67) ParOldGen: 454260K(699392K)->509352K(699392K)
[0.954s][info   ][gc,metaspace   ] GC(67) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.954s][info   ][gc             ] GC(67) Pause Young (Allocation Failure) 692M->565M(947M) 4.234ms
[0.954s][info   ][gc,cpu         ] GC(67) User=0.01s Sys=0.00s Real=0.01s
[0.964s][info   ][gc,start       ] GC(68) Pause Young (Allocation Failure)
[0.967s][info   ][gc,heap        ] GC(68) PSYoungGen: 262474K(270848K)->66503K(270848K) Eden: 192512K(192512K)->0K(192512K) From: 69962K(78336K)->66503K(78336K)
[0.967s][info   ][gc,heap        ] GC(68) ParOldGen: 509352K(699392K)->563032K(699392K)
[0.967s][info   ][gc,metaspace   ] GC(68) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.967s][info   ][gc             ] GC(68) Pause Young (Allocation Failure) 753M->614M(947M) 3.037ms
[0.967s][info   ][gc,cpu         ] GC(68) User=0.01s Sys=0.00s Real=0.00s
[0.977s][info   ][gc,start       ] GC(69) Pause Young (Allocation Failure)
[0.981s][info   ][gc,heap        ] GC(69) PSYoungGen: 259001K(270848K)->70775K(269824K) Eden: 192497K(192512K)->0K(192512K) From: 66503K(78336K)->70775K(77312K)
[0.981s][info   ][gc,heap        ] GC(69) ParOldGen: 563032K(699392K)->617318K(699392K)
[0.981s][info   ][gc,metaspace   ] GC(69) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.981s][info   ][gc             ] GC(69) Pause Young (Allocation Failure) 802M->671M(946M) 4.155ms
[0.981s][info   ][gc,cpu         ] GC(69) User=0.02s Sys=0.00s Real=0.00s
[0.991s][info   ][gc,start       ] GC(70) Pause Young (Allocation Failure)
[0.994s][info   ][gc,heap        ] GC(70) PSYoungGen: 263287K(269824K)->72525K(265216K) Eden: 192512K(192512K)->0K(192512K) From: 70775K(77312K)->72525K(72704K)
[0.994s][info   ][gc,heap        ] GC(70) ParOldGen: 617318K(699392K)->675529K(699392K)
[0.994s][info   ][gc,metaspace   ] GC(70) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.994s][info   ][gc             ] GC(70) Pause Young (Allocation Failure) 859M->730M(942M) 3.413ms
[0.994s][info   ][gc,cpu         ] GC(70) User=0.01s Sys=0.00s Real=0.01s
[0.994s][info   ][gc,start       ] GC(71) Pause Full (Ergonomics)
[0.994s][info   ][gc,phases,start] GC(71) Marking Phase
[0.996s][info   ][gc,phases      ] GC(71) Marking Phase 1.916ms
[0.996s][info   ][gc,phases,start] GC(71) Summary Phase
[0.996s][info   ][gc,phases      ] GC(71) Summary Phase 0.027ms
[0.996s][info   ][gc,phases,start] GC(71) Adjust Roots
[0.996s][info   ][gc,phases      ] GC(71) Adjust Roots 0.148ms
[0.996s][info   ][gc,phases,start] GC(71) Compaction Phase
[1.002s][info   ][gc,phases      ] GC(71) Compaction Phase 6.091ms
[1.002s][info   ][gc,phases,start] GC(71) Post Compact
[1.003s][info   ][gc,phases      ] GC(71) Post Compact 0.287ms
[1.003s][info   ][gc,heap        ] GC(71) PSYoungGen: 72525K(265216K)->0K(269824K) Eden: 0K(192512K)->0K(190464K) From: 72525K(72704K)->0K(79360K)
[1.003s][info   ][gc,heap        ] GC(71) ParOldGen: 675529K(699392K)->349547K(699392K)
[1.003s][info   ][gc,metaspace   ] GC(71) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.003s][info   ][gc             ] GC(71) Pause Full (Ergonomics) 730M->341M(946M) 8.659ms
[1.003s][info   ][gc,cpu         ] GC(71) User=0.04s Sys=0.00s Real=0.00s
[1.013s][info   ][gc,start       ] GC(72) Pause Young (Allocation Failure)
[1.015s][info   ][gc,heap        ] GC(72) PSYoungGen: 190464K(269824K)->64261K(269824K) Eden: 190464K(190464K)->0K(190464K) From: 0K(79360K)->64261K(79360K)
[1.015s][info   ][gc,heap        ] GC(72) ParOldGen: 349547K(699392K)->349547K(699392K)
[1.015s][info   ][gc,metaspace   ] GC(72) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.015s][info   ][gc             ] GC(72) Pause Young (Allocation Failure) 527M->404M(946M) 2.106ms
[1.015s][info   ][gc,cpu         ] GC(72) User=0.00s Sys=0.00s Real=0.01s
counter:50304
[1.025s][info   ][gc,heap,exit   ] Heap
[1.025s][info   ][gc,heap,exit   ]  PSYoungGen      total 269824K, used 174317K [0x00000007eab00000, 0x0000000800000000, 0x0000000800000000)
[1.025s][info   ][gc,heap,exit   ]   eden space 190464K, 57% used [0x00000007eab00000,0x00000007f167a1d0,0x00000007f6500000)
[1.025s][info   ][gc,heap,exit   ]   from space 79360K, 80% used [0x00000007f6500000,0x00000007fa3c1450,0x00000007fb280000)
[1.025s][info   ][gc,heap,exit   ]   to   space 78336K, 0% used [0x00000007fb380000,0x00000007fb380000,0x0000000800000000)
[1.025s][info   ][gc,heap,exit   ]  ParOldGen       total 699392K, used 349547K [0x00000007c0000000, 0x00000007eab00000, 0x00000007eab00000)
[1.025s][info   ][gc,heap,exit   ]   object space 699392K, 49% used [0x00000007c0000000,0x00000007d555ad30,0x00000007eab00000)
[1.025s][info   ][gc,heap,exit   ]  Metaspace       used 233K, committed 448K, reserved 1114112K
[1.025s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```

在试试2g内存
```
java -XX:+UseParallelGC -Xms2g -Xmx2g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以发现跟1g内存相比，GC次数大大减少，但是吞吐量并没有明显增加。仅仅生成了55000多个对象。但是性能依然比串行GC强大
```
[0.001s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.008s][info   ][gc,init] CardTable entry size: 512
[0.008s][info   ][gc     ] Using Parallel
[0.009s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.009s][info   ][gc,init] CPUs: 8 total, 8 available
[0.009s][info   ][gc,init] Memory: 16384M
[0.009s][info   ][gc,init] Large Page Support: Disabled
[0.009s][info   ][gc,init] NUMA Support: Disabled
[0.009s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.009s][info   ][gc,init] Alignments: Space 512K, Generation 512K, Heap 8M
[0.009s][info   ][gc,init] Heap Min Capacity: 2G
[0.009s][info   ][gc,init] Heap Initial Capacity: 2G
[0.009s][info   ][gc,init] Heap Max Capacity: 2G
[0.009s][info   ][gc,init] Pre-touch: Disabled
[0.009s][info   ][gc,init] Parallel Workers: 8
[0.009s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.009s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.010s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.090s][info   ][gc,start    ] GC(0) Pause Young (Allocation Failure)
[0.112s][info   ][gc,heap     ] GC(0) PSYoungGen: 524800K(611840K)->87031K(611840K) Eden: 524800K(524800K)->0K(524800K) From: 0K(87040K)->87031K(87040K)
[0.112s][info   ][gc,heap     ] GC(0) ParOldGen: 992K(1398272K)->61286K(1398272K)
[0.112s][info   ][gc,metaspace] GC(0) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.112s][info   ][gc          ] GC(0) Pause Young (Allocation Failure) 513M->144M(1963M) 22.679ms
[0.112s][info   ][gc,cpu      ] GC(0) User=0.01s Sys=0.10s Real=0.03s
[0.140s][info   ][gc,start    ] GC(1) Pause Young (Allocation Failure)
[0.173s][info   ][gc,heap     ] GC(1) PSYoungGen: 611831K(611840K)->87036K(611840K) Eden: 524800K(524800K)->0K(524800K) From: 87031K(87040K)->87036K(87040K)
[0.173s][info   ][gc,heap     ] GC(1) ParOldGen: 61286K(1398272K)->191982K(1398272K)
[0.173s][info   ][gc,metaspace] GC(1) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.173s][info   ][gc          ] GC(1) Pause Young (Allocation Failure) 657M->272M(1963M) 32.979ms
[0.173s][info   ][gc,cpu      ] GC(1) User=0.01s Sys=0.15s Real=0.04s
[0.199s][info   ][gc,start    ] GC(2) Pause Young (Allocation Failure)
[0.221s][info   ][gc,heap     ] GC(2) PSYoungGen: 611836K(611840K)->87029K(611840K) Eden: 524800K(524800K)->0K(524800K) From: 87036K(87040K)->87029K(87040K)
[0.221s][info   ][gc,heap     ] GC(2) ParOldGen: 191982K(1398272K)->317206K(1398272K)
[0.221s][info   ][gc,metaspace] GC(2) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.221s][info   ][gc          ] GC(2) Pause Young (Allocation Failure) 784M->394M(1963M) 21.600ms
[0.221s][info   ][gc,cpu      ] GC(2) User=0.02s Sys=0.09s Real=0.02s
[0.246s][info   ][gc,start    ] GC(3) Pause Young (Allocation Failure)
[0.267s][info   ][gc,heap     ] GC(3) PSYoungGen: 611829K(611840K)->87039K(611840K) Eden: 524800K(524800K)->0K(524800K) From: 87029K(87040K)->87039K(87040K)
[0.267s][info   ][gc,heap     ] GC(3) ParOldGen: 317206K(1398272K)->438522K(1398272K)
[0.267s][info   ][gc,metaspace] GC(3) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.267s][info   ][gc          ] GC(3) Pause Young (Allocation Failure) 907M->513M(1963M) 20.535ms
[0.267s][info   ][gc,cpu      ] GC(3) User=0.02s Sys=0.09s Real=0.02s
[0.292s][info   ][gc,start    ] GC(4) Pause Young (Allocation Failure)
[0.311s][info   ][gc,heap     ] GC(4) PSYoungGen: 611839K(611840K)->87030K(611840K) Eden: 524800K(524800K)->0K(524800K) From: 87039K(87040K)->87030K(87040K)
[0.311s][info   ][gc,heap     ] GC(4) ParOldGen: 438522K(1398272K)->552672K(1398272K)
[0.311s][info   ][gc,metaspace] GC(4) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.311s][info   ][gc          ] GC(4) Pause Young (Allocation Failure) 1025M->624M(1963M) 18.904ms
[0.311s][info   ][gc,cpu      ] GC(4) User=0.02s Sys=0.07s Real=0.01s
[0.337s][info   ][gc,start    ] GC(5) Pause Young (Allocation Failure)
[0.357s][info   ][gc,heap     ] GC(5) PSYoungGen: 611830K(611840K)->87039K(320000K) Eden: 524800K(524800K)->0K(232960K) From: 87030K(87040K)->87039K(87040K)
[0.357s][info   ][gc,heap     ] GC(5) ParOldGen: 552672K(1398272K)->674964K(1398272K)
[0.357s][info   ][gc,metaspace] GC(5) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.357s][info   ][gc          ] GC(5) Pause Young (Allocation Failure) 1137M->744M(1678M) 20.775ms
[0.357s][info   ][gc,cpu      ] GC(5) User=0.02s Sys=0.08s Real=0.02s
[0.368s][info   ][gc,start    ] GC(6) Pause Young (Allocation Failure)
[0.372s][info   ][gc,heap     ] GC(6) PSYoungGen: 319999K(320000K)->146545K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 87039K(87040K)->146545K(232960K)
[0.372s][info   ][gc,heap     ] GC(6) ParOldGen: 674964K(1398272K)->677669K(1398272K)
[0.372s][info   ][gc,metaspace] GC(6) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.372s][info   ][gc          ] GC(6) Pause Young (Allocation Failure) 971M->804M(1820M) 4.059ms
[0.372s][info   ][gc,cpu      ] GC(6) User=0.01s Sys=0.01s Real=0.01s
[0.382s][info   ][gc,start    ] GC(7) Pause Young (Allocation Failure)
[0.387s][info   ][gc,heap     ] GC(7) PSYoungGen: 379505K(465920K)->184215K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 146545K(232960K)->184215K(232960K)
[0.387s][info   ][gc,heap     ] GC(7) ParOldGen: 677669K(1398272K)->684801K(1398272K)
[0.387s][info   ][gc,metaspace] GC(7) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.387s][info   ][gc          ] GC(7) Pause Young (Allocation Failure) 1032M->848M(1820M) 4.360ms
[0.387s][info   ][gc,cpu      ] GC(7) User=0.02s Sys=0.00s Real=0.00s
[0.398s][info   ][gc,start    ] GC(8) Pause Young (Allocation Failure)
[0.405s][info   ][gc,heap     ] GC(8) PSYoungGen: 417175K(465920K)->189133K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 184215K(232960K)->189133K(232960K)
[0.405s][info   ][gc,heap     ] GC(8) ParOldGen: 684801K(1398272K)->712032K(1398272K)
[0.406s][info   ][gc,metaspace] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.406s][info   ][gc          ] GC(8) Pause Young (Allocation Failure) 1076M->880M(1820M) 7.531ms
[0.406s][info   ][gc,cpu      ] GC(8) User=0.03s Sys=0.01s Real=0.01s
[0.416s][info   ][gc,start    ] GC(9) Pause Young (Allocation Failure)
[0.429s][info   ][gc,heap     ] GC(9) PSYoungGen: 422093K(465920K)->141865K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 189133K(232960K)->141865K(232960K)
[0.429s][info   ][gc,heap     ] GC(9) ParOldGen: 712032K(1398272K)->794384K(1398272K)
[0.429s][info   ][gc,metaspace] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.429s][info   ][gc          ] GC(9) Pause Young (Allocation Failure) 1107M->914M(1820M) 12.835ms
[0.429s][info   ][gc,cpu      ] GC(9) User=0.01s Sys=0.05s Real=0.01s
[0.441s][info   ][gc,start    ] GC(10) Pause Young (Allocation Failure)
[0.458s][info   ][gc,heap     ] GC(10) PSYoungGen: 374825K(465920K)->76198K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 141865K(232960K)->76198K(232960K)
[0.458s][info   ][gc,heap     ] GC(10) ParOldGen: 794384K(1398272K)->904007K(1398272K)
[0.458s][info   ][gc,metaspace] GC(10) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.458s][info   ][gc          ] GC(10) Pause Young (Allocation Failure) 1141M->957M(1820M) 17.027ms
[0.458s][info   ][gc,cpu      ] GC(10) User=0.02s Sys=0.05s Real=0.02s
[0.469s][info   ][gc,start    ] GC(11) Pause Young (Allocation Failure)
[0.478s][info   ][gc,heap     ] GC(11) PSYoungGen: 309158K(465920K)->84341K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 76198K(232960K)->84341K(232960K)
[0.478s][info   ][gc,heap     ] GC(11) ParOldGen: 904007K(1398272K)->961218K(1398272K)
[0.478s][info   ][gc,metaspace] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.478s][info   ][gc          ] GC(11) Pause Young (Allocation Failure) 1184M->1021M(1820M) 8.948ms
[0.478s][info   ][gc,cpu      ] GC(11) User=0.01s Sys=0.03s Real=0.01s
[0.489s][info   ][gc,start    ] GC(12) Pause Young (Allocation Failure)
[0.499s][info   ][gc,heap     ] GC(12) PSYoungGen: 317301K(465920K)->91409K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 84341K(232960K)->91409K(232960K)
[0.499s][info   ][gc,heap     ] GC(12) ParOldGen: 961218K(1398272K)->1024078K(1398272K)
[0.499s][info   ][gc,metaspace] GC(12) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.499s][info   ][gc          ] GC(12) Pause Young (Allocation Failure) 1248M->1089M(1820M) 10.357ms
[0.499s][info   ][gc,cpu      ] GC(12) User=0.01s Sys=0.03s Real=0.01s
[0.511s][info   ][gc,start    ] GC(13) Pause Young (Allocation Failure)
[0.522s][info   ][gc,heap     ] GC(13) PSYoungGen: 324369K(465920K)->86160K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 91409K(232960K)->86160K(232960K)
[0.522s][info   ][gc,heap     ] GC(13) ParOldGen: 1024078K(1398272K)->1090071K(1398272K)
[0.522s][info   ][gc,metaspace] GC(13) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.522s][info   ][gc          ] GC(13) Pause Young (Allocation Failure) 1316M->1148M(1820M) 10.590ms
[0.522s][info   ][gc,cpu      ] GC(13) User=0.02s Sys=0.04s Real=0.02s
[0.534s][info   ][gc,start    ] GC(14) Pause Young (Allocation Failure)
[0.545s][info   ][gc,heap     ] GC(14) PSYoungGen: 319120K(465920K)->84397K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 86160K(232960K)->84397K(232960K)
[0.545s][info   ][gc,heap     ] GC(14) ParOldGen: 1090071K(1398272K)->1154089K(1398272K)
[0.545s][info   ][gc,metaspace] GC(14) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.545s][info   ][gc          ] GC(14) Pause Young (Allocation Failure) 1376M->1209M(1820M) 11.072ms
[0.545s][info   ][gc,cpu      ] GC(14) User=0.01s Sys=0.03s Real=0.01s
[0.556s][info   ][gc,start    ] GC(15) Pause Young (Allocation Failure)
[0.571s][info   ][gc,heap     ] GC(15) PSYoungGen: 317357K(465920K)->82614K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 84397K(232960K)->82614K(232960K)
[0.571s][info   ][gc,heap     ] GC(15) ParOldGen: 1154089K(1398272K)->1218247K(1398272K)
[0.571s][info   ][gc,metaspace] GC(15) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.571s][info   ][gc          ] GC(15) Pause Young (Allocation Failure) 1436M->1270M(1820M) 15.289ms
[0.571s][info   ][gc,cpu      ] GC(15) User=0.02s Sys=0.02s Real=0.02s
[0.582s][info   ][gc,start    ] GC(16) Pause Young (Allocation Failure)
[0.593s][info   ][gc,heap     ] GC(16) PSYoungGen: 315458K(465920K)->86317K(465920K) Eden: 232843K(232960K)->0K(232960K) From: 82614K(232960K)->86317K(232960K)
[0.593s][info   ][gc,heap     ] GC(16) ParOldGen: 1218247K(1398272K)->1282310K(1398272K)
[0.593s][info   ][gc,metaspace] GC(16) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.593s][info   ][gc          ] GC(16) Pause Young (Allocation Failure) 1497M->1336M(1820M) 10.896ms
[0.593s][info   ][gc,cpu      ] GC(16) User=0.02s Sys=0.04s Real=0.01s
[0.593s][info   ][gc,start    ] GC(17) Pause Full (Ergonomics)
[0.593s][info   ][gc,phases,start] GC(17) Marking Phase
[0.598s][info   ][gc,phases      ] GC(17) Marking Phase 4.460ms
[0.598s][info   ][gc,phases,start] GC(17) Summary Phase
[0.598s][info   ][gc,phases      ] GC(17) Summary Phase 0.032ms
[0.598s][info   ][gc,phases,start] GC(17) Adjust Roots
[0.598s][info   ][gc,phases      ] GC(17) Adjust Roots 0.291ms
[0.598s][info   ][gc,phases,start] GC(17) Compaction Phase
[0.613s][info   ][gc,phases      ] GC(17) Compaction Phase 14.951ms
[0.613s][info   ][gc,phases,start] GC(17) Post Compact
[0.615s][info   ][gc,phases      ] GC(17) Post Compact 1.997ms
[0.617s][info   ][gc,heap        ] GC(17) PSYoungGen: 86317K(465920K)->0K(465920K) Eden: 0K(232960K)->0K(232960K) From: 86317K(232960K)->0K(232960K)
[0.617s][info   ][gc,heap        ] GC(17) ParOldGen: 1282310K(1398272K)->354109K(1055744K)
[0.617s][info   ][gc,metaspace   ] GC(17) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.617s][info   ][gc             ] GC(17) Pause Full (Ergonomics) 1336M->345M(1486M) 24.063ms
[0.617s][info   ][gc,cpu         ] GC(17) User=0.04s Sys=0.04s Real=0.02s
[0.629s][info   ][gc,start       ] GC(18) Pause Young (Allocation Failure)
[0.631s][info   ][gc,heap        ] GC(18) PSYoungGen: 232960K(465920K)->81182K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 0K(232960K)->81182K(232960K)
[0.631s][info   ][gc,heap        ] GC(18) ParOldGen: 354109K(1055744K)->354109K(1055744K)
[0.631s][info   ][gc,metaspace   ] GC(18) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.631s][info   ][gc             ] GC(18) Pause Young (Allocation Failure) 573M->425M(1486M) 2.269ms
[0.631s][info   ][gc,cpu         ] GC(18) User=0.01s Sys=0.00s Real=0.00s
[0.643s][info   ][gc,start       ] GC(19) Pause Young (Allocation Failure)
[0.646s][info   ][gc,heap        ] GC(19) PSYoungGen: 314142K(465920K)->88267K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 81182K(232960K)->88267K(232960K)
[0.646s][info   ][gc,heap        ] GC(19) ParOldGen: 354109K(1055744K)->416374K(1055744K)
[0.646s][info   ][gc,metaspace   ] GC(19) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.646s][info   ][gc             ] GC(19) Pause Young (Allocation Failure) 652M->492M(1486M) 3.236ms
[0.646s][info   ][gc,cpu         ] GC(19) User=0.01s Sys=0.01s Real=0.00s
[0.657s][info   ][gc,start       ] GC(20) Pause Young (Allocation Failure)
[0.660s][info   ][gc,heap        ] GC(20) PSYoungGen: 321227K(465920K)->79763K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 88267K(232960K)->79763K(232960K)
[0.660s][info   ][gc,heap        ] GC(20) ParOldGen: 416374K(1055744K)->480706K(1055744K)
[0.660s][info   ][gc,metaspace   ] GC(20) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.660s][info   ][gc             ] GC(20) Pause Young (Allocation Failure) 720M->547M(1486M) 3.221ms
[0.660s][info   ][gc,cpu         ] GC(20) User=0.02s Sys=0.00s Real=0.00s
[0.671s][info   ][gc,start       ] GC(21) Pause Young (Allocation Failure)
[0.674s][info   ][gc,heap        ] GC(21) PSYoungGen: 312723K(465920K)->86716K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 79763K(232960K)->86716K(232960K)
[0.675s][info   ][gc,heap        ] GC(21) ParOldGen: 480706K(1055744K)->539008K(1055744K)
[0.675s][info   ][gc,metaspace   ] GC(21) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.675s][info   ][gc             ] GC(21) Pause Young (Allocation Failure) 774M->611M(1486M) 3.790ms
[0.675s][info   ][gc,cpu         ] GC(21) User=0.01s Sys=0.00s Real=0.01s
[0.686s][info   ][gc,start       ] GC(22) Pause Young (Allocation Failure)
[0.690s][info   ][gc,heap        ] GC(22) PSYoungGen: 319676K(465920K)->86769K(465920K) Eden: 232960K(232960K)->0K(232960K) From: 86716K(232960K)->86769K(232960K)
[0.690s][info   ][gc,heap        ] GC(22) ParOldGen: 539008K(1055744K)->606035K(1055744K)
[0.690s][info   ][gc,metaspace   ] GC(22) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.690s][info   ][gc             ] GC(22) Pause Young (Allocation Failure) 838M->676M(1486M) 3.756ms
[0.690s][info   ][gc,cpu         ] GC(22) User=0.02s Sys=0.00s Real=0.00s
[0.701s][info   ][gc,start       ] GC(23) Pause Young (Allocation Failure)
[0.706s][info   ][gc,heap        ] GC(23) PSYoungGen: 319729K(465920K)->84737K(474624K) Eden: 232960K(232960K)->0K(243712K) From: 86769K(232960K)->84737K(230912K)
[0.706s][info   ][gc,heap        ] GC(23) ParOldGen: 606035K(1055744K)->673967K(1055744K)
[0.706s][info   ][gc,metaspace   ] GC(23) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.706s][info   ][gc             ] GC(23) Pause Young (Allocation Failure) 904M->740M(1494M) 4.809ms
[0.706s][info   ][gc,cpu         ] GC(23) User=0.02s Sys=0.01s Real=0.00s
[0.718s][info   ][gc,start       ] GC(24) Pause Young (Allocation Failure)
[0.722s][info   ][gc,heap        ] GC(24) PSYoungGen: 328339K(474624K)->89229K(467968K) Eden: 243602K(243712K)->0K(243712K) From: 84737K(230912K)->89229K(224256K)
[0.722s][info   ][gc,heap        ] GC(24) ParOldGen: 673967K(1055744K)->736386K(1055744K)
[0.722s][info   ][gc,metaspace   ] GC(24) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.722s][info   ][gc             ] GC(24) Pause Young (Allocation Failure) 978M->806M(1488M) 4.306ms
[0.722s][info   ][gc,cpu         ] GC(24) User=0.02s Sys=0.00s Real=0.01s
[0.734s][info   ][gc,start       ] GC(25) Pause Young (Allocation Failure)
[0.739s][info   ][gc,heap        ] GC(25) PSYoungGen: 332941K(467968K)->90960K(490496K) Eden: 243712K(243712K)->0K(273920K) From: 89229K(224256K)->90960K(216576K)
[0.739s][info   ][gc,heap        ] GC(25) ParOldGen: 736386K(1055744K)->802595K(1055744K)
[0.739s][info   ][gc,metaspace   ] GC(25) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.739s][info   ][gc             ] GC(25) Pause Young (Allocation Failure) 1044M->872M(1510M) 5.029ms
[0.739s][info   ][gc,cpu         ] GC(25) User=0.02s Sys=0.01s Real=0.00s
[0.752s][info   ][gc,start       ] GC(26) Pause Young (Allocation Failure)
[0.757s][info   ][gc,heap        ] GC(26) PSYoungGen: 364880K(490496K)->100075K(482304K) Eden: 273920K(273920K)->0K(273920K) From: 90960K(216576K)->100075K(208384K)
[0.757s][info   ][gc,heap        ] GC(26) ParOldGen: 802595K(1055744K)->864645K(1055744K)
[0.757s][info   ][gc,metaspace   ] GC(26) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.757s][info   ][gc             ] GC(26) Pause Young (Allocation Failure) 1140M->942M(1502M) 4.670ms
[0.757s][info   ][gc,cpu         ] GC(26) User=0.02s Sys=0.00s Real=0.00s
[0.771s][info   ][gc,start       ] GC(27) Pause Young (Allocation Failure)
[0.777s][info   ][gc,heap        ] GC(27) PSYoungGen: 373995K(482304K)->92740K(506368K) Eden: 273920K(273920K)->0K(306688K) From: 100075K(208384K)->92740K(199680K)
[0.777s][info   ][gc,heap        ] GC(27) ParOldGen: 864645K(1055744K)->931036K(1055744K)
[0.777s][info   ][gc,metaspace   ] GC(27) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.777s][info   ][gc             ] GC(27) Pause Young (Allocation Failure) 1209M->999M(1525M) 5.561ms
[0.777s][info   ][gc,cpu         ] GC(27) User=0.01s Sys=0.00s Real=0.00s
[0.791s][info   ][gc,start       ] GC(28) Pause Young (Allocation Failure)
[0.796s][info   ][gc,heap        ] GC(28) PSYoungGen: 399198K(506368K)->109395K(499200K) Eden: 306457K(306688K)->0K(306688K) From: 92740K(199680K)->109395K(192512K)
[0.796s][info   ][gc,heap        ] GC(28) ParOldGen: 931036K(1055744K)->988954K(1055744K)
[0.796s][info   ][gc,metaspace   ] GC(28) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.796s][info   ][gc             ] GC(28) Pause Young (Allocation Failure) 1299M->1072M(1518M) 5.066ms
[0.796s][info   ][gc,cpu         ] GC(28) User=0.02s Sys=0.01s Real=0.00s
[0.796s][info   ][gc,start       ] GC(29) Pause Full (Ergonomics)
[0.796s][info   ][gc,phases,start] GC(29) Marking Phase
[0.799s][info   ][gc,phases      ] GC(29) Marking Phase 2.435ms
[0.799s][info   ][gc,phases,start] GC(29) Summary Phase
[0.799s][info   ][gc,phases      ] GC(29) Summary Phase 0.026ms
[0.799s][info   ][gc,phases,start] GC(29) Adjust Roots
[0.799s][info   ][gc,phases      ] GC(29) Adjust Roots 0.177ms
[0.799s][info   ][gc,phases,start] GC(29) Compaction Phase
[0.806s][info   ][gc,phases      ] GC(29) Compaction Phase 6.812ms
[0.806s][info   ][gc,phases,start] GC(29) Post Compact
[0.807s][info   ][gc,phases      ] GC(29) Post Compact 1.197ms
[0.807s][info   ][gc,heap        ] GC(29) PSYoungGen: 109395K(499200K)->0K(513536K) Eden: 0K(306688K)->0K(328192K) From: 109395K(192512K)->0K(185344K)
[0.807s][info   ][gc,heap        ] GC(29) ParOldGen: 988954K(1055744K)->356192K(1094144K)
[0.807s][info   ][gc,metaspace   ] GC(29) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.807s][info   ][gc             ] GC(29) Pause Full (Ergonomics) 1072M->347M(1570M) 10.854ms
[0.807s][info   ][gc,cpu         ] GC(29) User=0.04s Sys=0.00s Real=0.01s
[0.822s][info   ][gc,start       ] GC(30) Pause Young (Allocation Failure)
[0.825s][info   ][gc,heap        ] GC(30) PSYoungGen: 327511K(513536K)->118904K(513536K) Eden: 327511K(328192K)->0K(328192K) From: 0K(185344K)->118904K(185344K)
[0.825s][info   ][gc,heap        ] GC(30) ParOldGen: 356192K(1094144K)->356192K(1094144K)
[0.825s][info   ][gc,metaspace   ] GC(30) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.825s][info   ][gc             ] GC(30) Pause Young (Allocation Failure) 667M->463M(1570M) 2.847ms
[0.825s][info   ][gc,cpu         ] GC(30) User=0.01s Sys=0.00s Real=0.00s
[0.841s][info   ][gc,start       ] GC(31) Pause Young (Allocation Failure)
[0.845s][info   ][gc,heap        ] GC(31) PSYoungGen: 447096K(513536K)->109238K(522752K) Eden: 328192K(328192K)->0K(339968K) From: 118904K(185344K)->109238K(182784K)
[0.845s][info   ][gc,heap        ] GC(31) ParOldGen: 356192K(1094144K)->436555K(1094144K)
[0.846s][info   ][gc,metaspace   ] GC(31) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.846s][info   ][gc             ] GC(31) Pause Young (Allocation Failure) 784M->533M(1579M) 4.302ms
[0.846s][info   ][gc,cpu         ] GC(31) User=0.02s Sys=0.00s Real=0.00s
[0.862s][info   ][gc,start       ] GC(32) Pause Young (Allocation Failure)
[0.867s][info   ][gc,heap        ] GC(32) PSYoungGen: 449206K(522752K)->122327K(516096K) Eden: 339968K(339968K)->0K(339968K) From: 109238K(182784K)->122327K(176128K)
[0.867s][info   ][gc,heap        ] GC(32) ParOldGen: 436555K(1094144K)->507874K(1094144K)
[0.867s][info   ][gc,metaspace   ] GC(32) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.867s][info   ][gc             ] GC(32) Pause Young (Allocation Failure) 865M->615M(1572M) 4.370ms
[0.867s][info   ][gc,cpu         ] GC(32) User=0.02s Sys=0.01s Real=0.00s
[0.884s][info   ][gc,start       ] GC(33) Pause Young (Allocation Failure)
[0.887s][info   ][gc,heap        ] GC(33) PSYoungGen: 462295K(516096K)->114844K(528384K) Eden: 339968K(339968K)->0K(353280K) From: 122327K(176128K)->114844K(175104K)
[0.888s][info   ][gc,heap        ] GC(33) ParOldGen: 507874K(1094144K)->589433K(1094144K)
[0.888s][info   ][gc,metaspace   ] GC(33) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.888s][info   ][gc             ] GC(33) Pause Young (Allocation Failure) 947M->687M(1584M) 3.693ms
[0.888s][info   ][gc,cpu         ] GC(33) User=0.03s Sys=0.00s Real=0.00s
[0.905s][info   ][gc,start       ] GC(34) Pause Young (Allocation Failure)
[0.909s][info   ][gc,heap        ] GC(34) PSYoungGen: 468124K(528384K)->116811K(523776K) Eden: 353280K(353280K)->0K(353280K) From: 114844K(175104K)->116811K(170496K)
[0.909s][info   ][gc,heap        ] GC(34) ParOldGen: 589433K(1094144K)->668986K(1094144K)
[0.909s][info   ][gc,metaspace   ] GC(34) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.909s][info   ][gc             ] GC(34) Pause Young (Allocation Failure) 1032M->767M(1580M) 3.912ms
[0.909s][info   ][gc,cpu         ] GC(34) User=0.02s Sys=0.00s Real=0.00s
[0.927s][info   ][gc,start       ] GC(35) Pause Young (Allocation Failure)
[0.931s][info   ][gc,heap        ] GC(35) PSYoungGen: 470091K(523776K)->105742K(536576K) Eden: 353280K(353280K)->0K(369152K) From: 116811K(170496K)->105742K(167424K)
[0.931s][info   ][gc,heap        ] GC(35) ParOldGen: 668986K(1094144K)->749228K(1094144K)
[0.931s][info   ][gc,metaspace   ] GC(35) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.931s][info   ][gc             ] GC(35) Pause Young (Allocation Failure) 1112M->834M(1592M) 4.281ms
[0.931s][info   ][gc,cpu         ] GC(35) User=0.02s Sys=0.00s Real=0.01s
[0.949s][info   ][gc,start       ] GC(36) Pause Young (Allocation Failure)
[0.953s][info   ][gc,heap        ] GC(36) PSYoungGen: 474894K(536576K)->111211K(531456K) Eden: 369152K(369152K)->0K(369152K) From: 105742K(167424K)->111211K(162304K)
[0.953s][info   ][gc,heap        ] GC(36) ParOldGen: 749228K(1094144K)->816076K(1094144K)
[0.953s][info   ][gc,metaspace   ] GC(36) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.953s][info   ][gc             ] GC(36) Pause Young (Allocation Failure) 1195M->905M(1587M) 4.001ms
[0.953s][info   ][gc,cpu         ] GC(36) User=0.02s Sys=0.00s Real=0.01s
[0.972s][info   ][gc,start       ] GC(37) Pause Young (Allocation Failure)
[0.976s][info   ][gc,heap        ] GC(37) PSYoungGen: 480363K(531456K)->115482K(543744K) Eden: 369152K(369152K)->0K(386048K) From: 111211K(162304K)->115482K(157696K)
[0.976s][info   ][gc,heap        ] GC(37) ParOldGen: 816076K(1094144K)->889667K(1094144K)
[0.976s][info   ][gc,metaspace   ] GC(37) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.976s][info   ][gc             ] GC(37) Pause Young (Allocation Failure) 1266M->981M(1599M) 4.116ms
[0.976s][info   ][gc,cpu         ] GC(37) User=0.02s Sys=0.00s Real=0.00s
[0.997s][info   ][gc,start       ] GC(38) Pause Young (Allocation Failure)
[1.002s][info   ][gc,heap        ] GC(38) PSYoungGen: 501526K(543744K)->129272K(541184K) Eden: 386044K(386048K)->0K(386048K) From: 115482K(157696K)->129272K(155136K)
[1.002s][info   ][gc,heap        ] GC(38) ParOldGen: 889667K(1094144K)->958781K(1094144K)
[1.002s][info   ][gc,metaspace   ] GC(38) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.002s][info   ][gc             ] GC(38) Pause Young (Allocation Failure) 1358M->1062M(1597M) 4.907ms
[1.002s][info   ][gc,cpu         ] GC(38) User=0.03s Sys=0.00s Real=0.01s
[1.020s][info   ][gc,start       ] GC(39) Pause Young (Allocation Failure)
[1.027s][info   ][gc,heap        ] GC(39) PSYoungGen: 515320K(541184K)->131431K(536064K) Eden: 386048K(386048K)->0K(378368K) From: 129272K(155136K)->131431K(157696K)
[1.027s][info   ][gc,heap        ] GC(39) ParOldGen: 958781K(1094144K)->1039401K(1094144K)
[1.027s][info   ][gc,metaspace   ] GC(39) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.027s][info   ][gc             ] GC(39) Pause Young (Allocation Failure) 1439M->1143M(1592M) 6.384ms
[1.027s][info   ][gc,cpu         ] GC(39) User=0.03s Sys=0.00s Real=0.01s
[1.027s][info   ][gc,start       ] GC(40) Pause Full (Ergonomics)
[1.027s][info   ][gc,phases,start] GC(40) Marking Phase
[1.029s][info   ][gc,phases      ] GC(40) Marking Phase 2.024ms
[1.029s][info   ][gc,phases,start] GC(40) Summary Phase
[1.029s][info   ][gc,phases      ] GC(40) Summary Phase 0.020ms
[1.029s][info   ][gc,phases,start] GC(40) Adjust Roots
[1.029s][info   ][gc,phases      ] GC(40) Adjust Roots 0.393ms
[1.029s][info   ][gc,phases,start] GC(40) Compaction Phase
[1.036s][info   ][gc,phases      ] GC(40) Compaction Phase 6.420ms
[1.036s][info   ][gc,phases,start] GC(40) Post Compact
[1.036s][info   ][gc,phases      ] GC(40) Post Compact 0.476ms
[1.036s][info   ][gc,heap        ] GC(40) PSYoungGen: 131431K(536064K)->0K(538624K) Eden: 0K(378368K)->0K(378368K) From: 131431K(157696K)->0K(160256K)
[1.036s][info   ][gc,heap        ] GC(40) ParOldGen: 1039401K(1094144K)->358721K(1170944K)
[1.036s][info   ][gc,metaspace   ] GC(40) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.036s][info   ][gc             ] GC(40) Pause Full (Ergonomics) 1143M->350M(1669M) 9.603ms
[1.036s][info   ][gc,cpu         ] GC(40) User=0.03s Sys=0.01s Real=0.01s
counter:55949
[1.042s][info   ][gc,heap,exit   ] Heap
[1.042s][info   ][gc,heap,exit   ]  PSYoungGen      total 538624K, used 15448K [0x00000007d5580000, 0x0000000800000000, 0x0000000800000000)
[1.042s][info   ][gc,heap,exit   ]   eden space 378368K, 4% used [0x00000007d5580000,0x00000007d64962c0,0x00000007ec700000)
[1.042s][info   ][gc,heap,exit   ]   from space 160256K, 0% used [0x00000007ec700000,0x00000007ec700000,0x00000007f6380000)
[1.042s][info   ][gc,heap,exit   ]   to   space 160256K, 0% used [0x00000007f6380000,0x00000007f6380000,0x0000000800000000)
[1.042s][info   ][gc,heap,exit   ]  ParOldGen       total 1170944K, used 358721K [0x0000000780000000, 0x00000007c7780000, 0x00000007d5580000)
[1.042s][info   ][gc,heap,exit   ]   object space 1170944K, 30% used [0x0000000780000000,0x0000000795e50400,0x00000007c7780000)
[1.042s][info   ][gc,heap,exit   ]  Metaspace       used 231K, committed 448K, reserved 1114112K
[1.042s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```


#### CMS GC

-XX: +UseConcMarkSweepGC

对年轻代使用STW的mark-copy算法，对老年代主要使用并发的mark-sweep算法。这个GC已经在java14版本被移除了。

设计目标:专为老年代设计，目标是最小化 GC 停顿时间。
1. 不对老年代进行整理，而是使用空闲列表来管理内存空间的回收
2. 在mark-and-sweep的工作和业务线程并发执行。

默认并发线程数等于CPU核心数的1/4

6个阶段
1. 初始标记
2. 并发标记
3. 并发预清理
4. 最终标记
5. 并发清楚
6. 并发重制

MaxHeapSize是系统的1/4内存
MaxNewSize是MaxHeapSize的1/3
NewSize是系统的1/64

优点：
- 只有yongGC暂停业务。GC 停顿时间短，适合延迟敏感的应用。
- 并发回收利用多核资源减少 STW 时间。

缺点：
- 内存碎片化：CMS 不会整理内存，可能导致分配大对象失败（触发 Full GC）。
- CPU 开销较高：并发阶段可能与用户线程争抢资源。
- 容易产生 "Concurrent Mode Failure"：若老年代空间不足，回退到 Serial GC。

适用场景：
- 延迟敏感的应用（如 Web 服务、在线交易系统）。
- 多核环境下的中大型应用。

#### G1 GC
分区堆内存：将堆划分为若干独立的固定大小的区域（Region），每个 Region 可充当年轻代、老年代或其他用途。混合回收：通过优先回收包含最多垃圾的 Region（Garbage-First）。并行和并发回收：减少 STW 时间。内置碎片整理机制，避免了 CMS 的碎片化问题。一般一个Region是1M。一部分 Region 保留为 Humongous（H） 区，用于存储超过单个 Region 大小一半的巨型对象。


-XX: +UseG1GC -XX: MaxGCPauseMillis=50

将STW的时间和分布变成可预期和可配置的，可设置某项特定的性能指标，为了达成可预期的指标，有独特的实现。增量方式，每次处理一部分，称为回收集合，每次处理所有的年轻代和部分老年代。能看到哪个块的垃圾多，优先回收他们

处理步骤
1. 年轻代模式转移暂停
2. 并发标记
3. 转移暂停：混合模式

G1GC可能退化成串行GC
1. 并发模式失败：增加堆大小
2. 晋升失败：
3. 巨型对象分配失败：增加内存或增大Region大小

优点：
- 减少内存碎片化。
- 更好地控制 GC 停顿时间，可通过 -XX:MaxGCPauseMillis 调整。
- 自动调节年轻代和老年代的大小。

缺点：
- 实现复杂，配置选项多。
- 内存占用较高，CPU 开销大。

适用场景：
- 需要低延迟的中大型应用。
- 堆内存较大的环境（如 >4GB）。
- 替代 CMS 的推荐选择。

Mixed GC
- 老年代内存使用率达到一定阈值（默认 45%，可通过 -XX:InitiatingHeapOccupancyPercent 调整）。
- 同时回收年轻代和部分老年代。
- 执行
  - 标记 GC Roots 直接引用的对象。
  - 并发扫描老年代，标记存活对象。
  - 处理标记期间新产生的引用变化。
- 根据垃圾优先（Garbage-First）的原则，优先选择包含最多垃圾的 Region。

接下来试试G1 GC 128m内存
```
java -XX:+UseG1GC -Xms128m -Xmx128m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

同样GC了103次后发生了OOM，可以看到GC次数很多。时间也很长，花了0.136s
```
[0.094s][info   ][gc             ] GC(36) Pause Young (Normal) (G1 Preventive Collection) 115M->115M(128M) 0.243ms
[0.094s][info   ][gc,cpu         ] GC(36) User=0.00s Sys=0.00s Real=0.00s
[0.094s][info   ][gc,start       ] GC(37) Pause Young (Normal) (G1 Humongous Allocation)
[0.094s][info   ][gc,task        ] GC(37) Using 3 workers of 8 for evacuation
[0.094s][info   ][gc,phases      ] GC(37)   Pre Evacuate Collection Set: 0.0ms
[0.094s][info   ][gc,phases      ] GC(37)   Merge Heap Roots: 0.0ms
[0.094s][info   ][gc,phases      ] GC(37)   Evacuate Collection Set: 0.0ms
[0.094s][info   ][gc,phases      ] GC(37)   Post Evacuate Collection Set: 0.0ms
[0.094s][info   ][gc,phases      ] GC(37)   Other: 0.0ms
[0.094s][info   ][gc,heap        ] GC(37) Eden regions: 1->0(6)
[0.094s][info   ][gc,heap        ] GC(37) Survivor regions: 1->0(0)
[0.094s][info   ][gc,heap        ] GC(37) Old regions: 77->79
[0.094s][info   ][gc,heap        ] GC(37) Archive regions: 2->2
[0.094s][info   ][gc,heap        ] GC(37) Humongous regions: 47->47
[0.094s][info   ][gc,metaspace   ] GC(37) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.094s][info   ][gc             ] GC(37) Pause Young (Normal) (G1 Humongous Allocation) (Evacuation Failure) 116M->117M(128M) 0.219ms
[0.094s][info   ][gc,cpu         ] GC(37) User=0.00s Sys=0.00s Real=0.00s
[0.094s][info   ][gc,ergo        ] Attempting full compaction
[0.094s][info   ][gc,start       ] GC(38) Pause Full (G1 Compaction Pause)
[0.094s][info   ][gc,task        ] GC(38) Using 3 workers of 8 for full compaction
[0.094s][info   ][gc,phases,start] GC(38) Phase 1: Mark live objects
[0.095s][info   ][gc,phases      ] GC(38) Phase 1: Mark live objects 0.306ms
[0.095s][info   ][gc,phases,start] GC(38) Phase 2: Prepare compaction
[0.095s][info   ][gc,phases      ] GC(38) Phase 2: Prepare compaction 0.075ms
[0.095s][info   ][gc,phases,start] GC(38) Phase 3: Adjust pointers
[0.095s][info   ][gc,phases      ] GC(38) Phase 3: Adjust pointers 0.192ms
[0.095s][info   ][gc,phases,start] GC(38) Phase 4: Compact heap
[0.096s][info   ][gc,phases      ] GC(38) Phase 4: Compact heap 0.523ms
[0.096s][info   ][gc,heap        ] GC(38) Eden regions: 0->0(6)
[0.096s][info   ][gc,heap        ] GC(38) Survivor regions: 0->0(0)
[0.096s][info   ][gc,heap        ] GC(38) Old regions: 79->75
[0.096s][info   ][gc,heap        ] GC(38) Archive regions: 2->2
[0.096s][info   ][gc,heap        ] GC(38) Humongous regions: 47->47
[0.096s][info   ][gc,metaspace   ] GC(38) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.096s][info   ][gc             ] GC(38) Pause Full (G1 Compaction Pause) 117M->115M(128M) 1.529ms
[0.096s][info   ][gc,cpu         ] GC(38) User=0.00s Sys=0.01s Real=0.00s
[0.096s][info   ][gc,marking     ] GC(32) Concurrent Mark From Roots 3.526ms
[0.096s][info   ][gc,marking     ] GC(32) Concurrent Mark Abort
[0.096s][info   ][gc             ] GC(32) Concurrent Mark Cycle 3.602ms
[0.096s][info   ][gc,start       ] GC(39) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.096s][info   ][gc,task        ] GC(39) Using 3 workers of 8 for evacuation
[0.096s][info   ][gc,phases      ] GC(39)   Pre Evacuate Collection Set: 0.0ms
[0.096s][info   ][gc,phases      ] GC(39)   Merge Heap Roots: 0.0ms
[0.096s][info   ][gc,phases      ] GC(39)   Evacuate Collection Set: 0.1ms
[0.096s][info   ][gc,phases      ] GC(39)   Post Evacuate Collection Set: 0.1ms
[0.096s][info   ][gc,phases      ] GC(39)   Other: 0.0ms
[0.096s][info   ][gc,heap        ] GC(39) Eden regions: 1->0(5)
[0.096s][info   ][gc,heap        ] GC(39) Survivor regions: 0->1(1)
[0.096s][info   ][gc,heap        ] GC(39) Old regions: 75->75
[0.096s][info   ][gc,heap        ] GC(39) Archive regions: 2->2
[0.096s][info   ][gc,heap        ] GC(39) Humongous regions: 48->47
[0.096s][info   ][gc,metaspace   ] GC(39) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.096s][info   ][gc             ] GC(39) Pause Young (Concurrent Start) (G1 Humongous Allocation) 117M->116M(128M) 0.322ms
[0.096s][info   ][gc,cpu         ] GC(39) User=0.00s Sys=0.00s Real=0.00s
[0.096s][info   ][gc             ] GC(40) Concurrent Mark Cycle
[0.096s][info   ][gc,marking     ] GC(40) Concurrent Clear Claimed Marks
[0.096s][info   ][gc,marking     ] GC(40) Concurrent Clear Claimed Marks 0.008ms
[0.096s][info   ][gc,marking     ] GC(40) Concurrent Scan Root Regions
[0.096s][info   ][gc,marking     ] GC(40) Concurrent Scan Root Regions 0.016ms
[0.096s][info   ][gc,marking     ] GC(40) Concurrent Mark
[0.096s][info   ][gc,marking     ] GC(40) Concurrent Mark From Roots
[0.096s][info   ][gc,start       ] GC(41) Pause Young (Normal) (G1 Preventive Collection)
[0.096s][info   ][gc,task        ] GC(40) Using 2 workers of 2 for marking
[0.096s][info   ][gc,task        ] GC(41) Using 3 workers of 8 for evacuation
[0.097s][info   ][gc,phases      ] GC(41)   Pre Evacuate Collection Set: 0.0ms
[0.097s][info   ][gc,phases      ] GC(41)   Merge Heap Roots: 0.0ms
[0.097s][info   ][gc,phases      ] GC(41)   Evacuate Collection Set: 0.0ms
[0.097s][info   ][gc,phases      ] GC(41)   Post Evacuate Collection Set: 0.1ms
[0.097s][info   ][gc,phases      ] GC(41)   Other: 0.0ms
[0.097s][info   ][gc,heap        ] GC(41) Eden regions: 1->0(5)
[0.097s][info   ][gc,heap        ] GC(41) Survivor regions: 1->1(1)
[0.097s][info   ][gc,heap        ] GC(41) Old regions: 75->76
[0.097s][info   ][gc,heap        ] GC(41) Archive regions: 2->2
[0.097s][info   ][gc,heap        ] GC(41) Humongous regions: 48->48
[0.097s][info   ][gc,metaspace   ] GC(41) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.097s][info   ][gc             ] GC(41) Pause Young (Normal) (G1 Preventive Collection) (Evacuation Failure) 118M->118M(128M) 0.308ms
[0.097s][info   ][gc,cpu         ] GC(41) User=0.00s Sys=0.00s Real=0.00s
[0.097s][info   ][gc,start       ] GC(42) Pause Young (Normal) (G1 Evacuation Pause)
[0.097s][info   ][gc,task        ] GC(42) Using 3 workers of 8 for evacuation
[0.097s][info   ][gc,phases      ] GC(42)   Pre Evacuate Collection Set: 0.0ms
[0.097s][info   ][gc,phases      ] GC(42)   Merge Heap Roots: 0.0ms
[0.097s][info   ][gc,phases      ] GC(42)   Evacuate Collection Set: 0.0ms
[0.097s][info   ][gc,phases      ] GC(42)   Post Evacuate Collection Set: 0.1ms
[0.097s][info   ][gc,phases      ] GC(42)   Other: 0.0ms
[0.097s][info   ][gc,heap        ] GC(42) Eden regions: 1->0(6)
[0.097s][info   ][gc,heap        ] GC(42) Survivor regions: 1->0(0)
[0.097s][info   ][gc,heap        ] GC(42) Old regions: 76->78
[0.097s][info   ][gc,heap        ] GC(42) Archive regions: 2->2
[0.097s][info   ][gc,heap        ] GC(42) Humongous regions: 48->48
[0.097s][info   ][gc,metaspace   ] GC(42) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.097s][info   ][gc             ] GC(42) Pause Young (Normal) (G1 Evacuation Pause) (Evacuation Failure) 119M->119M(128M) 0.272ms
[0.097s][info   ][gc,cpu         ] GC(42) User=0.00s Sys=0.00s Real=0.00s
[0.097s][info   ][gc,ergo        ] Attempting full compaction
[0.097s][info   ][gc,start       ] GC(43) Pause Full (G1 Compaction Pause)
[0.097s][info   ][gc,task        ] GC(43) Using 3 workers of 8 for full compaction
[0.097s][info   ][gc,phases,start] GC(43) Phase 1: Mark live objects
[0.098s][info   ][gc,phases      ] GC(43) Phase 1: Mark live objects 0.289ms
[0.098s][info   ][gc,phases,start] GC(43) Phase 2: Prepare compaction
[0.098s][info   ][gc,phases      ] GC(43) Phase 2: Prepare compaction 0.080ms
[0.098s][info   ][gc,phases,start] GC(43) Phase 3: Adjust pointers
[0.098s][info   ][gc,phases      ] GC(43) Phase 3: Adjust pointers 0.208ms
[0.098s][info   ][gc,phases,start] GC(43) Phase 4: Compact heap
[0.098s][info   ][gc,phases      ] GC(43) Phase 4: Compact heap 0.200ms
[0.098s][info   ][gc,heap        ] GC(43) Eden regions: 0->0(6)
[0.098s][info   ][gc,heap        ] GC(43) Survivor regions: 0->0(0)
[0.098s][info   ][gc,heap        ] GC(43) Old regions: 78->76
[0.098s][info   ][gc,heap        ] GC(43) Archive regions: 2->2
[0.098s][info   ][gc,heap        ] GC(43) Humongous regions: 48->48
[0.098s][info   ][gc,metaspace   ] GC(43) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.098s][info   ][gc             ] GC(43) Pause Full (G1 Compaction Pause) 119M->117M(128M) 1.131ms
[0.098s][info   ][gc,cpu         ] GC(43) User=0.01s Sys=0.00s Real=0.00s
[0.098s][info   ][gc,marking     ] GC(40) Concurrent Mark From Roots 2.025ms
[0.098s][info   ][gc,marking     ] GC(40) Concurrent Mark Abort
[0.098s][info   ][gc             ] GC(40) Concurrent Mark Cycle 2.138ms
[0.098s][info   ][gc,start       ] GC(44) Pause Young (Normal) (G1 Preventive Collection)
[0.098s][info   ][gc,task        ] GC(44) Using 3 workers of 8 for evacuation
[0.099s][info   ][gc,phases      ] GC(44)   Pre Evacuate Collection Set: 0.0ms
[0.099s][info   ][gc,phases      ] GC(44)   Merge Heap Roots: 0.0ms
[0.099s][info   ][gc,phases      ] GC(44)   Evacuate Collection Set: 0.1ms
[0.099s][info   ][gc,phases      ] GC(44)   Post Evacuate Collection Set: 0.0ms
[0.099s][info   ][gc,phases      ] GC(44)   Other: 0.0ms
[0.099s][info   ][gc,heap        ] GC(44) Eden regions: 1->0(5)
[0.099s][info   ][gc,heap        ] GC(44) Survivor regions: 0->1(1)
[0.099s][info   ][gc,heap        ] GC(44) Old regions: 76->76
[0.099s][info   ][gc,heap        ] GC(44) Archive regions: 2->2
[0.099s][info   ][gc,heap        ] GC(44) Humongous regions: 48->48
[0.099s][info   ][gc,metaspace   ] GC(44) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.099s][info   ][gc             ] GC(44) Pause Young (Normal) (G1 Preventive Collection) 118M->118M(128M) 0.248ms
[0.099s][info   ][gc,cpu         ] GC(44) User=0.00s Sys=0.00s Real=0.00s
[0.099s][info   ][gc,start       ] GC(45) Pause Young (Concurrent Start) (G1 Evacuation Pause)
[0.099s][info   ][gc,task        ] GC(45) Using 3 workers of 8 for evacuation
[0.099s][info   ][gc,phases      ] GC(45)   Pre Evacuate Collection Set: 0.0ms
[0.099s][info   ][gc,phases      ] GC(45)   Merge Heap Roots: 0.0ms
[0.099s][info   ][gc,phases      ] GC(45)   Evacuate Collection Set: 0.1ms
[0.099s][info   ][gc,phases      ] GC(45)   Post Evacuate Collection Set: 0.0ms
[0.099s][info   ][gc,phases      ] GC(45)   Other: 0.0ms
[0.099s][info   ][gc,heap        ] GC(45) Eden regions: 1->0(6)
[0.099s][info   ][gc,heap        ] GC(45) Survivor regions: 1->0(0)
[0.099s][info   ][gc,heap        ] GC(45) Old regions: 76->78
[0.099s][info   ][gc,heap        ] GC(45) Archive regions: 2->2
[0.099s][info   ][gc,heap        ] GC(45) Humongous regions: 48->48
[0.099s][info   ][gc,metaspace   ] GC(45) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.099s][info   ][gc             ] GC(45) Pause Young (Concurrent Start) (G1 Evacuation Pause) (Evacuation Failure) 118M->118M(128M) 0.237ms
[0.099s][info   ][gc,cpu         ] GC(45) User=0.00s Sys=0.00s Real=0.00s
[0.099s][info   ][gc,ergo        ] Attempting full compaction
[0.099s][info   ][gc,start       ] GC(46) Pause Full (G1 Compaction Pause)
[0.099s][info   ][gc,task        ] GC(46) Using 3 workers of 8 for full compaction
[0.099s][info   ][gc             ] GC(47) Concurrent Mark Cycle
[0.099s][info   ][gc,marking     ] GC(47) Concurrent Clear Claimed Marks
[0.099s][info   ][gc,marking     ] GC(47) Concurrent Clear Claimed Marks 0.006ms
[0.099s][info   ][gc,marking     ] GC(47) Concurrent Scan Root Regions
[0.099s][info   ][gc,marking     ] GC(47) Concurrent Scan Root Regions 0.006ms
[0.099s][info   ][gc,marking     ] GC(47) Concurrent Mark
[0.099s][info   ][gc,marking     ] GC(47) Concurrent Mark From Roots
[0.099s][info   ][gc,task        ] GC(47) Using 2 workers of 2 for marking
[0.099s][info   ][gc,phases,start] GC(46) Phase 1: Mark live objects
[0.099s][info   ][gc,phases      ] GC(46) Phase 1: Mark live objects 0.304ms
[0.100s][info   ][gc,phases,start] GC(46) Phase 2: Prepare compaction
[0.100s][info   ][gc,phases      ] GC(46) Phase 2: Prepare compaction 0.058ms
[0.100s][info   ][gc,phases,start] GC(46) Phase 3: Adjust pointers
[0.100s][info   ][gc,phases      ] GC(46) Phase 3: Adjust pointers 0.221ms
[0.100s][info   ][gc,phases,start] GC(46) Phase 4: Compact heap
[0.100s][info   ][gc,phases      ] GC(46) Phase 4: Compact heap 0.236ms
[0.100s][info   ][gc,heap        ] GC(46) Eden regions: 0->0(6)
[0.100s][info   ][gc,heap        ] GC(46) Survivor regions: 0->0(0)
[0.100s][info   ][gc,heap        ] GC(46) Old regions: 78->77
[0.100s][info   ][gc,heap        ] GC(46) Archive regions: 2->2
[0.100s][info   ][gc,heap        ] GC(46) Humongous regions: 48->47
[0.100s][info   ][gc,metaspace   ] GC(46) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.100s][info   ][gc             ] GC(46) Pause Full (G1 Compaction Pause) 118M->117M(128M) 1.197ms
[0.100s][info   ][gc,cpu         ] GC(46) User=0.00s Sys=0.00s Real=0.00s
[0.100s][info   ][gc,marking     ] GC(47) Concurrent Mark From Roots 1.169ms
[0.100s][info   ][gc,marking     ] GC(47) Concurrent Mark Abort
[0.100s][info   ][gc             ] GC(47) Concurrent Mark Cycle 1.230ms
[0.100s][info   ][gc,start       ] GC(48) Pause Young (Normal) (G1 Preventive Collection)
[0.100s][info   ][gc,task        ] GC(48) Using 3 workers of 8 for evacuation
[0.101s][info   ][gc,phases      ] GC(48)   Pre Evacuate Collection Set: 0.0ms
[0.101s][info   ][gc,phases      ] GC(48)   Merge Heap Roots: 0.0ms
[0.101s][info   ][gc,phases      ] GC(48)   Evacuate Collection Set: 0.1ms
[0.101s][info   ][gc,phases      ] GC(48)   Post Evacuate Collection Set: 0.0ms
[0.101s][info   ][gc,phases      ] GC(48)   Other: 0.0ms
[0.101s][info   ][gc,heap        ] GC(48) Eden regions: 1->0(5)
[0.101s][info   ][gc,heap        ] GC(48) Survivor regions: 0->1(1)
[0.101s][info   ][gc,heap        ] GC(48) Old regions: 77->77
[0.101s][info   ][gc,heap        ] GC(48) Archive regions: 2->2
[0.101s][info   ][gc,heap        ] GC(48) Humongous regions: 47->47
[0.101s][info   ][gc,metaspace   ] GC(48) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.101s][info   ][gc             ] GC(48) Pause Young (Normal) (G1 Preventive Collection) 118M->117M(128M) 0.264ms
[0.101s][info   ][gc,cpu         ] GC(48) User=0.00s Sys=0.00s Real=0.00s
[0.101s][info   ][gc,start       ] GC(49) Pause Young (Concurrent Start) (G1 Evacuation Pause)
[0.101s][info   ][gc,task        ] GC(49) Using 3 workers of 8 for evacuation
[0.101s][info   ][gc,phases      ] GC(49)   Pre Evacuate Collection Set: 0.0ms
[0.101s][info   ][gc,phases      ] GC(49)   Merge Heap Roots: 0.0ms
[0.101s][info   ][gc,phases      ] GC(49)   Evacuate Collection Set: 0.1ms
[0.101s][info   ][gc,phases      ] GC(49)   Post Evacuate Collection Set: 0.1ms
[0.101s][info   ][gc,phases      ] GC(49)   Other: 0.0ms
[0.101s][info   ][gc,heap        ] GC(49) Eden regions: 1->0(6)
[0.101s][info   ][gc,heap        ] GC(49) Survivor regions: 1->0(0)
[0.101s][info   ][gc,heap        ] GC(49) Old regions: 77->79
[0.101s][info   ][gc,heap        ] GC(49) Archive regions: 2->2
[0.101s][info   ][gc,heap        ] GC(49) Humongous regions: 47->47
[0.101s][info   ][gc,metaspace   ] GC(49) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.101s][info   ][gc             ] GC(49) Pause Young (Concurrent Start) (G1 Evacuation Pause) (Evacuation Failure) 118M->118M(128M) 0.281ms
[0.101s][info   ][gc,cpu         ] GC(49) User=0.00s Sys=0.00s Real=0.00s
[0.101s][info   ][gc,ergo        ] Attempting full compaction
[0.101s][info   ][gc,start       ] GC(50) Pause Full (G1 Compaction Pause)
[0.101s][info   ][gc,task        ] GC(50) Using 3 workers of 8 for full compaction
[0.101s][info   ][gc             ] GC(51) Concurrent Mark Cycle
[0.101s][info   ][gc,marking     ] GC(51) Concurrent Clear Claimed Marks
[0.101s][info   ][gc,marking     ] GC(51) Concurrent Clear Claimed Marks 0.006ms
[0.101s][info   ][gc,marking     ] GC(51) Concurrent Scan Root Regions
[0.101s][info   ][gc,marking     ] GC(51) Concurrent Scan Root Regions 0.006ms
[0.101s][info   ][gc,marking     ] GC(51) Concurrent Mark
[0.101s][info   ][gc,marking     ] GC(51) Concurrent Mark From Roots
[0.101s][info   ][gc,task        ] GC(51) Using 2 workers of 2 for marking
[0.101s][info   ][gc,phases,start] GC(50) Phase 1: Mark live objects
[0.101s][info   ][gc,phases      ] GC(50) Phase 1: Mark live objects 0.309ms
[0.102s][info   ][gc,phases,start] GC(50) Phase 2: Prepare compaction
[0.102s][info   ][gc,phases      ] GC(50) Phase 2: Prepare compaction 0.070ms
[0.102s][info   ][gc,phases,start] GC(50) Phase 3: Adjust pointers
[0.102s][info   ][gc,phases      ] GC(50) Phase 3: Adjust pointers 0.203ms
[0.102s][info   ][gc,phases,start] GC(50) Phase 4: Compact heap
[0.102s][info   ][gc,phases      ] GC(50) Phase 4: Compact heap 0.416ms
[0.102s][info   ][gc,heap        ] GC(50) Eden regions: 0->0(6)
[0.102s][info   ][gc,heap        ] GC(50) Survivor regions: 0->0(0)
[0.102s][info   ][gc,heap        ] GC(50) Old regions: 79->76
[0.102s][info   ][gc,heap        ] GC(50) Archive regions: 2->2
[0.102s][info   ][gc,heap        ] GC(50) Humongous regions: 47->47
[0.102s][info   ][gc,metaspace   ] GC(50) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.102s][info   ][gc             ] GC(50) Pause Full (G1 Compaction Pause) 118M->117M(128M) 1.406ms
[0.102s][info   ][gc,cpu         ] GC(50) User=0.00s Sys=0.00s Real=0.00s
[0.102s][info   ][gc,marking     ] GC(51) Concurrent Mark From Roots 1.345ms
[0.102s][info   ][gc,marking     ] GC(51) Concurrent Mark Abort
[0.103s][info   ][gc             ] GC(51) Concurrent Mark Cycle 1.406ms
[0.103s][info   ][gc,start       ] GC(52) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.103s][info   ][gc,task        ] GC(52) Using 3 workers of 8 for evacuation
[0.103s][info   ][gc,phases      ] GC(52)   Pre Evacuate Collection Set: 0.0ms
[0.103s][info   ][gc,phases      ] GC(52)   Merge Heap Roots: 0.0ms
[0.103s][info   ][gc,phases      ] GC(52)   Evacuate Collection Set: 0.1ms
[0.103s][info   ][gc,phases      ] GC(52)   Post Evacuate Collection Set: 0.1ms
[0.103s][info   ][gc,phases      ] GC(52)   Other: 0.0ms
[0.103s][info   ][gc,heap        ] GC(52) Eden regions: 1->0(6)
[0.103s][info   ][gc,heap        ] GC(52) Survivor regions: 0->0(1)
[0.103s][info   ][gc,heap        ] GC(52) Old regions: 76->76
[0.103s][info   ][gc,heap        ] GC(52) Archive regions: 2->2
[0.103s][info   ][gc,heap        ] GC(52) Humongous regions: 47->47
[0.103s][info   ][gc,metaspace   ] GC(52) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.103s][info   ][gc             ] GC(52) Pause Young (Concurrent Start) (G1 Humongous Allocation) 118M->117M(128M) 0.342ms
[0.103s][info   ][gc,cpu         ] GC(52) User=0.00s Sys=0.00s Real=0.00s
[0.103s][info   ][gc             ] GC(53) Concurrent Mark Cycle
[0.103s][info   ][gc,marking     ] GC(53) Concurrent Clear Claimed Marks
[0.103s][info   ][gc,marking     ] GC(53) Concurrent Clear Claimed Marks 0.009ms
[0.103s][info   ][gc,marking     ] GC(53) Concurrent Scan Root Regions
[0.103s][info   ][gc,marking     ] GC(53) Concurrent Scan Root Regions 0.007ms
[0.103s][info   ][gc,marking     ] GC(53) Concurrent Mark
[0.103s][info   ][gc,marking     ] GC(53) Concurrent Mark From Roots
[0.103s][info   ][gc,task        ] GC(53) Using 2 workers of 2 for marking
[0.103s][info   ][gc,start       ] GC(54) Pause Young (Normal) (G1 Preventive Collection)
[0.103s][info   ][gc,task        ] GC(54) Using 3 workers of 8 for evacuation
[0.103s][info   ][gc,phases      ] GC(54)   Pre Evacuate Collection Set: 0.0ms
[0.103s][info   ][gc,phases      ] GC(54)   Merge Heap Roots: 0.0ms
[0.103s][info   ][gc,phases      ] GC(54)   Evacuate Collection Set: 0.1ms
[0.103s][info   ][gc,phases      ] GC(54)   Post Evacuate Collection Set: 0.1ms
[0.103s][info   ][gc,phases      ] GC(54)   Other: 0.0ms
[0.103s][info   ][gc,heap        ] GC(54) Eden regions: 1->0(5)
[0.103s][info   ][gc,heap        ] GC(54) Survivor regions: 0->1(1)
[0.103s][info   ][gc,heap        ] GC(54) Old regions: 76->76
[0.103s][info   ][gc,heap        ] GC(54) Archive regions: 2->2
[0.103s][info   ][gc,heap        ] GC(54) Humongous regions: 48->47
[0.103s][info   ][gc,metaspace   ] GC(54) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.103s][info   ][gc             ] GC(54) Pause Young (Normal) (G1 Preventive Collection) 119M->117M(128M) 0.299ms
[0.103s][info   ][gc,cpu         ] GC(54) User=0.00s Sys=0.00s Real=0.01s
[0.104s][info   ][gc,start       ] GC(55) Pause Young (Normal) (G1 Preventive Collection)
[0.104s][info   ][gc,task        ] GC(55) Using 3 workers of 8 for evacuation
[0.104s][info   ][gc,phases      ] GC(55)   Pre Evacuate Collection Set: 0.0ms
[0.104s][info   ][gc,phases      ] GC(55)   Merge Heap Roots: 0.0ms
[0.104s][info   ][gc,phases      ] GC(55)   Evacuate Collection Set: 0.1ms
[0.104s][info   ][gc,phases      ] GC(55)   Post Evacuate Collection Set: 0.2ms
[0.104s][info   ][gc,phases      ] GC(55)   Other: 0.0ms
[0.104s][info   ][gc,heap        ] GC(55) Eden regions: 1->0(5)
[0.104s][info   ][gc,heap        ] GC(55) Survivor regions: 1->1(1)
[0.104s][info   ][gc,heap        ] GC(55) Old regions: 76->76
[0.104s][info   ][gc,heap        ] GC(55) Archive regions: 2->2
[0.104s][info   ][gc,heap        ] GC(55) Humongous regions: 47->47
[0.104s][info   ][gc,metaspace   ] GC(55) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.104s][info   ][gc             ] GC(55) Pause Young (Normal) (G1 Preventive Collection) 118M->117M(128M) 0.390ms
[0.104s][info   ][gc,cpu         ] GC(55) User=0.00s Sys=0.00s Real=0.00s
[0.104s][info   ][gc,start       ] GC(56) Pause Young (Normal) (G1 Preventive Collection)
[0.104s][info   ][gc,task        ] GC(56) Using 3 workers of 8 for evacuation
[0.104s][info   ][gc,phases      ] GC(56)   Pre Evacuate Collection Set: 0.0ms
[0.104s][info   ][gc,phases      ] GC(56)   Merge Heap Roots: 0.0ms
[0.104s][info   ][gc,phases      ] GC(56)   Evacuate Collection Set: 0.1ms
[0.104s][info   ][gc,phases      ] GC(56)   Post Evacuate Collection Set: 0.1ms
[0.104s][info   ][gc,phases      ] GC(56)   Other: 0.0ms
[0.104s][info   ][gc,heap        ] GC(56) Eden regions: 0->0(5)
[0.104s][info   ][gc,heap        ] GC(56) Survivor regions: 1->1(1)
[0.104s][info   ][gc,heap        ] GC(56) Old regions: 76->76
[0.104s][info   ][gc,heap        ] GC(56) Archive regions: 2->2
[0.104s][info   ][gc,heap        ] GC(56) Humongous regions: 48->47
[0.104s][info   ][gc,metaspace   ] GC(56) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.105s][info   ][gc             ] GC(56) Pause Young (Normal) (G1 Preventive Collection) 118M->117M(128M) 0.383ms
[0.105s][info   ][gc,cpu         ] GC(56) User=0.00s Sys=0.00s Real=0.00s
[0.105s][info   ][gc,start       ] GC(57) Pause Young (Normal) (G1 Preventive Collection)
[0.105s][info   ][gc,task        ] GC(57) Using 3 workers of 8 for evacuation
[0.105s][info   ][gc,phases      ] GC(57)   Pre Evacuate Collection Set: 0.0ms
[0.105s][info   ][gc,phases      ] GC(57)   Merge Heap Roots: 0.0ms
[0.105s][info   ][gc,phases      ] GC(57)   Evacuate Collection Set: 0.1ms
[0.105s][info   ][gc,phases      ] GC(57)   Post Evacuate Collection Set: 0.1ms
[0.105s][info   ][gc,phases      ] GC(57)   Other: 0.0ms
[0.105s][info   ][gc,heap        ] GC(57) Eden regions: 1->0(5)
[0.105s][info   ][gc,heap        ] GC(57) Survivor regions: 1->1(1)
[0.105s][info   ][gc,heap        ] GC(57) Old regions: 76->76
[0.105s][info   ][gc,heap        ] GC(57) Archive regions: 2->2
[0.105s][info   ][gc,heap        ] GC(57) Humongous regions: 47->47
[0.105s][info   ][gc,metaspace   ] GC(57) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.105s][info   ][gc             ] GC(57) Pause Young (Normal) (G1 Preventive Collection) 118M->118M(128M) 0.410ms
[0.105s][info   ][gc,cpu         ] GC(57) User=0.00s Sys=0.00s Real=0.00s
[0.105s][info   ][gc,start       ] GC(58) Pause Young (Normal) (G1 Preventive Collection)
[0.105s][info   ][gc,task        ] GC(58) Using 3 workers of 8 for evacuation
[0.106s][info   ][gc,phases      ] GC(58)   Pre Evacuate Collection Set: 0.0ms
[0.106s][info   ][gc,phases      ] GC(58)   Merge Heap Roots: 0.0ms
[0.106s][info   ][gc,phases      ] GC(58)   Evacuate Collection Set: 0.1ms
[0.106s][info   ][gc,phases      ] GC(58)   Post Evacuate Collection Set: 0.0ms
[0.106s][info   ][gc,phases      ] GC(58)   Other: 0.0ms
[0.106s][info   ][gc,heap        ] GC(58) Eden regions: 1->0(5)
[0.106s][info   ][gc,heap        ] GC(58) Survivor regions: 1->1(1)
[0.106s][info   ][gc,heap        ] GC(58) Old regions: 76->77
[0.106s][info   ][gc,heap        ] GC(58) Archive regions: 2->2
[0.106s][info   ][gc,heap        ] GC(58) Humongous regions: 47->47
[0.106s][info   ][gc,metaspace   ] GC(58) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.106s][info   ][gc             ] GC(58) Pause Young (Normal) (G1 Preventive Collection) (Evacuation Failure) 119M->118M(128M) 0.268ms
[0.106s][info   ][gc,cpu         ] GC(58) User=0.00s Sys=0.00s Real=0.00s
[0.106s][info   ][gc,start       ] GC(59) Pause Young (Normal) (G1 Evacuation Pause)
[0.106s][info   ][gc,task        ] GC(59) Using 3 workers of 8 for evacuation
[0.106s][info   ][gc,phases      ] GC(59)   Pre Evacuate Collection Set: 0.0ms
[0.106s][info   ][gc,phases      ] GC(59)   Merge Heap Roots: 0.0ms
[0.106s][info   ][gc,phases      ] GC(59)   Evacuate Collection Set: 0.0ms
[0.106s][info   ][gc,phases      ] GC(59)   Post Evacuate Collection Set: 0.1ms
[0.106s][info   ][gc,phases      ] GC(59)   Other: 0.0ms
[0.106s][info   ][gc,heap        ] GC(59) Eden regions: 1->0(6)
[0.106s][info   ][gc,heap        ] GC(59) Survivor regions: 1->0(0)
[0.106s][info   ][gc,heap        ] GC(59) Old regions: 77->79
[0.106s][info   ][gc,heap        ] GC(59) Archive regions: 2->2
[0.106s][info   ][gc,heap        ] GC(59) Humongous regions: 47->47
[0.106s][info   ][gc,metaspace   ] GC(59) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.106s][info   ][gc             ] GC(59) Pause Young (Normal) (G1 Evacuation Pause) (Evacuation Failure) 119M->119M(128M) 0.234ms
[0.106s][info   ][gc,cpu         ] GC(59) User=0.00s Sys=0.00s Real=0.00s
[0.106s][info   ][gc,ergo        ] Attempting full compaction
[0.106s][info   ][gc,start       ] GC(60) Pause Full (G1 Compaction Pause)
[0.106s][info   ][gc,task        ] GC(60) Using 3 workers of 8 for full compaction
[0.106s][info   ][gc,phases,start] GC(60) Phase 1: Mark live objects
[0.106s][info   ][gc,phases      ] GC(60) Phase 1: Mark live objects 0.265ms
[0.106s][info   ][gc,phases,start] GC(60) Phase 2: Prepare compaction
[0.106s][info   ][gc,phases      ] GC(60) Phase 2: Prepare compaction 0.058ms
[0.106s][info   ][gc,phases,start] GC(60) Phase 3: Adjust pointers
[0.107s][info   ][gc,phases      ] GC(60) Phase 3: Adjust pointers 0.191ms
[0.107s][info   ][gc,phases,start] GC(60) Phase 4: Compact heap
[0.107s][info   ][gc,phases      ] GC(60) Phase 4: Compact heap 0.402ms
[0.107s][info   ][gc,heap        ] GC(60) Eden regions: 0->0(6)
[0.107s][info   ][gc,heap        ] GC(60) Survivor regions: 0->0(0)
[0.107s][info   ][gc,heap        ] GC(60) Old regions: 79->75
[0.107s][info   ][gc,heap        ] GC(60) Archive regions: 2->2
[0.107s][info   ][gc,heap        ] GC(60) Humongous regions: 47->46
[0.107s][info   ][gc,metaspace   ] GC(60) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.107s][info   ][gc             ] GC(60) Pause Full (G1 Compaction Pause) 119M->117M(128M) 1.422ms
[0.107s][info   ][gc,cpu         ] GC(60) User=0.00s Sys=0.00s Real=0.00s
[0.107s][info   ][gc,marking     ] GC(53) Concurrent Mark From Roots 4.478ms
[0.107s][info   ][gc,marking     ] GC(53) Concurrent Mark Abort
[0.107s][info   ][gc             ] GC(53) Concurrent Mark Cycle 4.589ms
[0.108s][info   ][gc,start       ] GC(61) Pause Young (Normal) (G1 Preventive Collection)
[0.108s][info   ][gc,task        ] GC(61) Using 3 workers of 8 for evacuation
[0.108s][info   ][gc,phases      ] GC(61)   Pre Evacuate Collection Set: 0.0ms
[0.108s][info   ][gc,phases      ] GC(61)   Merge Heap Roots: 0.1ms
[0.108s][info   ][gc,phases      ] GC(61)   Evacuate Collection Set: 0.1ms
[0.108s][info   ][gc,phases      ] GC(61)   Post Evacuate Collection Set: 0.1ms
[0.108s][info   ][gc,phases      ] GC(61)   Other: 0.0ms
[0.108s][info   ][gc,heap        ] GC(61) Eden regions: 1->0(5)
[0.108s][info   ][gc,heap        ] GC(61) Survivor regions: 0->1(1)
[0.108s][info   ][gc,heap        ] GC(61) Old regions: 75->75
[0.108s][info   ][gc,heap        ] GC(61) Archive regions: 2->2
[0.108s][info   ][gc,heap        ] GC(61) Humongous regions: 49->48
[0.108s][info   ][gc,metaspace   ] GC(61) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.108s][info   ][gc             ] GC(61) Pause Young (Normal) (G1 Preventive Collection) 121M->120M(128M) 0.379ms
[0.108s][info   ][gc,cpu         ] GC(61) User=0.00s Sys=0.00s Real=0.00s
[0.108s][info   ][gc,start       ] GC(62) Pause Young (Concurrent Start) (G1 Preventive Collection)
[0.108s][info   ][gc,task        ] GC(62) Using 3 workers of 8 for evacuation
[0.108s][info   ][gc,phases      ] GC(62)   Pre Evacuate Collection Set: 0.0ms
[0.108s][info   ][gc,phases      ] GC(62)   Merge Heap Roots: 0.0ms
[0.108s][info   ][gc,phases      ] GC(62)   Evacuate Collection Set: 0.1ms
[0.108s][info   ][gc,phases      ] GC(62)   Post Evacuate Collection Set: 0.1ms
[0.108s][info   ][gc,phases      ] GC(62)   Other: 0.1ms
[0.108s][info   ][gc,heap        ] GC(62) Eden regions: 1->0(6)
[0.108s][info   ][gc,heap        ] GC(62) Survivor regions: 1->0(1)
[0.108s][info   ][gc,heap        ] GC(62) Old regions: 75->76
[0.108s][info   ][gc,heap        ] GC(62) Archive regions: 2->2
[0.108s][info   ][gc,heap        ] GC(62) Humongous regions: 48->48
[0.108s][info   ][gc,metaspace   ] GC(62) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.108s][info   ][gc             ] GC(62) Pause Young (Concurrent Start) (G1 Preventive Collection) 121M->120M(128M) 0.371ms
[0.108s][info   ][gc,cpu         ] GC(62) User=0.00s Sys=0.00s Real=0.00s
[0.108s][info   ][gc             ] GC(63) Concurrent Mark Cycle
[0.109s][info   ][gc,marking     ] GC(63) Concurrent Clear Claimed Marks
[0.109s][info   ][gc,marking     ] GC(63) Concurrent Clear Claimed Marks 0.009ms
[0.109s][info   ][gc,marking     ] GC(63) Concurrent Scan Root Regions
[0.109s][info   ][gc,start       ] GC(64) Pause Young (Normal) (G1 Preventive Collection)
[0.109s][info   ][gc,marking     ] GC(63) Concurrent Scan Root Regions 0.048ms
[0.109s][info   ][gc,task        ] GC(64) Using 3 workers of 8 for evacuation
[0.109s][info   ][gc,marking     ] GC(63) Concurrent Mark
[0.109s][info   ][gc,marking     ] GC(63) Concurrent Mark From Roots
[0.109s][info   ][gc,task        ] GC(63) Using 2 workers of 2 for marking
[0.109s][info   ][gc,phases      ] GC(64)   Pre Evacuate Collection Set: 0.0ms
[0.109s][info   ][gc,phases      ] GC(64)   Merge Heap Roots: 0.1ms
[0.109s][info   ][gc,phases      ] GC(64)   Evacuate Collection Set: 0.1ms
[0.109s][info   ][gc,phases      ] GC(64)   Post Evacuate Collection Set: 0.1ms
[0.109s][info   ][gc,phases      ] GC(64)   Other: 0.0ms
[0.109s][info   ][gc,heap        ] GC(64) Eden regions: 1->0(5)
[0.109s][info   ][gc,heap        ] GC(64) Survivor regions: 0->1(1)
[0.109s][info   ][gc,heap        ] GC(64) Old regions: 76->76
[0.109s][info   ][gc,heap        ] GC(64) Archive regions: 2->2
[0.109s][info   ][gc,heap        ] GC(64) Humongous regions: 48->48
[0.109s][info   ][gc,metaspace   ] GC(64) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.109s][info   ][gc             ] GC(64) Pause Young (Normal) (G1 Preventive Collection) 121M->120M(128M) 0.433ms
[0.109s][info   ][gc,cpu         ] GC(64) User=0.01s Sys=0.00s Real=0.00s
[0.109s][info   ][gc,start       ] GC(65) Pause Young (Normal) (G1 Humongous Allocation)
[0.109s][info   ][gc,task        ] GC(65) Using 3 workers of 8 for evacuation
[0.109s][info   ][gc,phases      ] GC(65)   Pre Evacuate Collection Set: 0.0ms
[0.109s][info   ][gc,phases      ] GC(65)   Merge Heap Roots: 0.0ms
[0.109s][info   ][gc,phases      ] GC(65)   Evacuate Collection Set: 0.0ms
[0.109s][info   ][gc,phases      ] GC(65)   Post Evacuate Collection Set: 0.1ms
[0.109s][info   ][gc,phases      ] GC(65)   Other: 0.0ms
[0.109s][info   ][gc,heap        ] GC(65) Eden regions: 1->0(6)
[0.109s][info   ][gc,heap        ] GC(65) Survivor regions: 1->0(0)
[0.109s][info   ][gc,heap        ] GC(65) Old regions: 76->78
[0.109s][info   ][gc,heap        ] GC(65) Archive regions: 2->2
[0.109s][info   ][gc,heap        ] GC(65) Humongous regions: 48->48
[0.109s][info   ][gc,metaspace   ] GC(65) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.109s][info   ][gc             ] GC(65) Pause Young (Normal) (G1 Humongous Allocation) (Evacuation Failure) 121M->121M(128M) 0.306ms
[0.109s][info   ][gc,cpu         ] GC(65) User=0.00s Sys=0.00s Real=0.00s
[0.109s][info   ][gc,ergo        ] Attempting full compaction
[0.109s][info   ][gc,start       ] GC(66) Pause Full (G1 Compaction Pause)
[0.110s][info   ][gc,task        ] GC(66) Using 3 workers of 8 for full compaction
[0.110s][info   ][gc,phases,start] GC(66) Phase 1: Mark live objects
[0.110s][info   ][gc,phases      ] GC(66) Phase 1: Mark live objects 0.459ms
[0.110s][info   ][gc,phases,start] GC(66) Phase 2: Prepare compaction
[0.110s][info   ][gc,phases      ] GC(66) Phase 2: Prepare compaction 0.057ms
[0.110s][info   ][gc,phases,start] GC(66) Phase 3: Adjust pointers
[0.110s][info   ][gc,phases      ] GC(66) Phase 3: Adjust pointers 0.223ms
[0.110s][info   ][gc,phases,start] GC(66) Phase 4: Compact heap
[0.111s][info   ][gc,phases      ] GC(66) Phase 4: Compact heap 0.178ms
[0.111s][info   ][gc,heap        ] GC(66) Eden regions: 0->0(6)
[0.111s][info   ][gc,heap        ] GC(66) Survivor regions: 0->0(0)
[0.111s][info   ][gc,heap        ] GC(66) Old regions: 78->76
[0.111s][info   ][gc,heap        ] GC(66) Archive regions: 2->2
[0.111s][info   ][gc,heap        ] GC(66) Humongous regions: 48->48
[0.111s][info   ][gc,metaspace   ] GC(66) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.111s][info   ][gc             ] GC(66) Pause Full (G1 Compaction Pause) 121M->120M(128M) 1.353ms
[0.111s][info   ][gc,cpu         ] GC(66) User=0.00s Sys=0.00s Real=0.00s
[0.111s][info   ][gc,marking     ] GC(63) Concurrent Mark From Roots 2.212ms
[0.111s][info   ][gc,marking     ] GC(63) Concurrent Mark Abort
[0.111s][info   ][gc             ] GC(63) Concurrent Mark Cycle 2.447ms
[0.111s][info   ][gc,start       ] GC(67) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.111s][info   ][gc,task        ] GC(67) Using 3 workers of 8 for evacuation
[0.111s][info   ][gc,phases      ] GC(67)   Pre Evacuate Collection Set: 0.0ms
[0.111s][info   ][gc,phases      ] GC(67)   Merge Heap Roots: 0.0ms
[0.111s][info   ][gc,phases      ] GC(67)   Evacuate Collection Set: 0.1ms
[0.111s][info   ][gc,phases      ] GC(67)   Post Evacuate Collection Set: 0.1ms
[0.111s][info   ][gc,phases      ] GC(67)   Other: 0.0ms
[0.111s][info   ][gc,heap        ] GC(67) Eden regions: 1->0(6)
[0.111s][info   ][gc,heap        ] GC(67) Survivor regions: 0->0(0)
[0.111s][info   ][gc,heap        ] GC(67) Old regions: 76->77
[0.111s][info   ][gc,heap        ] GC(67) Archive regions: 2->2
[0.111s][info   ][gc,heap        ] GC(67) Humongous regions: 49->48
[0.111s][info   ][gc,metaspace   ] GC(67) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.111s][info   ][gc             ] GC(67) Pause Young (Concurrent Start) (G1 Humongous Allocation) (Evacuation Failure) 122M->121M(128M) 0.341ms
[0.111s][info   ][gc,cpu         ] GC(67) User=0.00s Sys=0.00s Real=0.00s
[0.111s][info   ][gc             ] GC(68) Concurrent Mark Cycle
[0.111s][info   ][gc,marking     ] GC(68) Concurrent Clear Claimed Marks
[0.111s][info   ][gc,marking     ] GC(68) Concurrent Clear Claimed Marks 0.007ms
[0.111s][info   ][gc,marking     ] GC(68) Concurrent Scan Root Regions
[0.111s][info   ][gc,marking     ] GC(68) Concurrent Scan Root Regions 0.008ms
[0.111s][info   ][gc,marking     ] GC(68) Concurrent Mark
[0.111s][info   ][gc,marking     ] GC(68) Concurrent Mark From Roots
[0.111s][info   ][gc,task        ] GC(68) Using 2 workers of 2 for marking
[0.111s][info   ][gc,start       ] GC(69) Pause Young (Normal) (G1 Evacuation Pause)
[0.111s][info   ][gc,task        ] GC(69) Using 3 workers of 8 for evacuation
[0.112s][info   ][gc,phases      ] GC(69)   Pre Evacuate Collection Set: 0.0ms
[0.112s][info   ][gc,phases      ] GC(69)   Merge Heap Roots: 0.0ms
[0.112s][info   ][gc,phases      ] GC(69)   Evacuate Collection Set: 0.0ms
[0.112s][info   ][gc,phases      ] GC(69)   Post Evacuate Collection Set: 0.1ms
[0.112s][info   ][gc,phases      ] GC(69)   Other: 0.0ms
[0.112s][info   ][gc,heap        ] GC(69) Eden regions: 0->0(6)
[0.112s][info   ][gc,heap        ] GC(69) Survivor regions: 0->0(0)
[0.112s][info   ][gc,heap        ] GC(69) Old regions: 77->77
[0.112s][info   ][gc,heap        ] GC(69) Archive regions: 2->2
[0.112s][info   ][gc,heap        ] GC(69) Humongous regions: 49->48
[0.112s][info   ][gc,metaspace   ] GC(69) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.112s][info   ][gc             ] GC(69) Pause Young (Normal) (G1 Evacuation Pause) 122M->121M(128M) 0.268ms
[0.112s][info   ][gc,cpu         ] GC(69) User=0.00s Sys=0.00s Real=0.00s
[0.112s][info   ][gc,marking     ] GC(68) Concurrent Mark From Roots 0.698ms
[0.112s][info   ][gc,marking     ] GC(68) Concurrent Preclean
[0.112s][info   ][gc,start       ] GC(70) Pause Young (Normal) (G1 Humongous Allocation)
[0.112s][info   ][gc,task        ] GC(70) Using 3 workers of 8 for evacuation
[0.112s][info   ][gc,phases      ] GC(70)   Pre Evacuate Collection Set: 0.0ms
[0.112s][info   ][gc,phases      ] GC(70)   Merge Heap Roots: 0.0ms
[0.112s][info   ][gc,phases      ] GC(70)   Evacuate Collection Set: 0.0ms
[0.112s][info   ][gc,phases      ] GC(70)   Post Evacuate Collection Set: 0.1ms
[0.112s][info   ][gc,phases      ] GC(70)   Other: 0.0ms
[0.112s][info   ][gc,heap        ] GC(70) Eden regions: 1->0(6)
[0.112s][info   ][gc,heap        ] GC(70) Survivor regions: 0->0(0)
[0.112s][info   ][gc,heap        ] GC(70) Old regions: 77->77
[0.112s][info   ][gc,heap        ] GC(70) Archive regions: 2->2
[0.112s][info   ][gc,heap        ] GC(70) Humongous regions: 48->48
[0.112s][info   ][gc,metaspace   ] GC(70) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.113s][info   ][gc             ] GC(70) Pause Young (Normal) (G1 Humongous Allocation) 122M->121M(128M) 0.799ms
[0.113s][info   ][gc,cpu         ] GC(70) User=0.00s Sys=0.00s Real=0.00s
[0.113s][info   ][gc,marking     ] GC(68) Concurrent Preclean 0.870ms
[0.113s][info   ][gc,start       ] GC(71) Pause Young (Normal) (G1 Evacuation Pause)
[0.113s][info   ][gc,task        ] GC(71) Using 3 workers of 8 for evacuation
[0.113s][info   ][gc,phases      ] GC(71)   Pre Evacuate Collection Set: 0.0ms
[0.113s][info   ][gc,phases      ] GC(71)   Merge Heap Roots: 0.0ms
[0.113s][info   ][gc,phases      ] GC(71)   Evacuate Collection Set: 0.1ms
[0.113s][info   ][gc,phases      ] GC(71)   Post Evacuate Collection Set: 0.1ms
[0.113s][info   ][gc,phases      ] GC(71)   Other: 0.0ms
[0.113s][info   ][gc,heap        ] GC(71) Eden regions: 0->0(6)
[0.113s][info   ][gc,heap        ] GC(71) Survivor regions: 0->0(0)
[0.113s][info   ][gc,heap        ] GC(71) Old regions: 77->77
[0.113s][info   ][gc,heap        ] GC(71) Archive regions: 2->2
[0.113s][info   ][gc,heap        ] GC(71) Humongous regions: 49->49
[0.113s][info   ][gc,metaspace   ] GC(71) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.114s][info   ][gc             ] GC(71) Pause Young (Normal) (G1 Evacuation Pause) 122M->122M(128M) 0.466ms
[0.114s][info   ][gc,cpu         ] GC(71) User=0.00s Sys=0.00s Real=0.01s
[0.114s][info   ][gc,ergo        ] Attempting full compaction
[0.114s][info   ][gc,start       ] GC(72) Pause Full (G1 Compaction Pause)
[0.114s][info   ][gc,task        ] GC(72) Using 3 workers of 8 for full compaction
[0.114s][info   ][gc,phases,start] GC(72) Phase 1: Mark live objects
[0.114s][info   ][gc,phases      ] GC(72) Phase 1: Mark live objects 0.493ms
[0.118s][info   ][gc,phases,start] GC(72) Phase 2: Prepare compaction
[0.118s][info   ][gc,phases      ] GC(72) Phase 2: Prepare compaction 0.124ms
[0.118s][info   ][gc,phases,start] GC(72) Phase 3: Adjust pointers
[0.118s][info   ][gc,phases      ] GC(72) Phase 3: Adjust pointers 0.228ms
[0.118s][info   ][gc,phases,start] GC(72) Phase 4: Compact heap
[0.118s][info   ][gc,phases      ] GC(72) Phase 4: Compact heap 0.159ms
[0.119s][info   ][gc,heap        ] GC(72) Eden regions: 0->0(6)
[0.119s][info   ][gc,heap        ] GC(72) Survivor regions: 0->0(0)
[0.119s][info   ][gc,heap        ] GC(72) Old regions: 77->75
[0.119s][info   ][gc,heap        ] GC(72) Archive regions: 2->2
[0.119s][info   ][gc,heap        ] GC(72) Humongous regions: 49->49
[0.119s][info   ][gc,metaspace   ] GC(72) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.119s][info   ][gc             ] GC(72) Pause Full (G1 Compaction Pause) 122M->122M(128M) 5.242ms
[0.119s][info   ][gc,cpu         ] GC(72) User=0.00s Sys=0.00s Real=0.00s
[0.119s][info   ][gc,marking     ] GC(68) Concurrent Mark Abort
[0.119s][info   ][gc             ] GC(68) Concurrent Mark Cycle 7.491ms
[0.119s][info   ][gc,start       ] GC(73) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.119s][info   ][gc,task        ] GC(73) Using 3 workers of 8 for evacuation
[0.119s][info   ][gc,phases      ] GC(73)   Pre Evacuate Collection Set: 0.0ms
[0.119s][info   ][gc,phases      ] GC(73)   Merge Heap Roots: 0.0ms
[0.119s][info   ][gc,phases      ] GC(73)   Evacuate Collection Set: 0.1ms
[0.119s][info   ][gc,phases      ] GC(73)   Post Evacuate Collection Set: 0.1ms
[0.119s][info   ][gc,phases      ] GC(73)   Other: 0.0ms
[0.119s][info   ][gc,heap        ] GC(73) Eden regions: 1->0(5)
[0.119s][info   ][gc,heap        ] GC(73) Survivor regions: 0->1(1)
[0.119s][info   ][gc,heap        ] GC(73) Old regions: 75->75
[0.119s][info   ][gc,heap        ] GC(73) Archive regions: 2->2
[0.119s][info   ][gc,heap        ] GC(73) Humongous regions: 49->49
[0.119s][info   ][gc,metaspace   ] GC(73) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.119s][info   ][gc             ] GC(73) Pause Young (Concurrent Start) (G1 Humongous Allocation) 122M->122M(128M) 0.367ms
[0.119s][info   ][gc,cpu         ] GC(73) User=0.00s Sys=0.00s Real=0.00s
[0.119s][info   ][gc             ] GC(74) Concurrent Mark Cycle
[0.119s][info   ][gc,marking     ] GC(74) Concurrent Clear Claimed Marks
[0.119s][info   ][gc,marking     ] GC(74) Concurrent Clear Claimed Marks 0.007ms
[0.119s][info   ][gc,marking     ] GC(74) Concurrent Scan Root Regions
[0.119s][info   ][gc,start       ] GC(75) Pause Young (Normal) (G1 Preventive Collection)
[0.119s][info   ][gc,task        ] GC(75) Using 3 workers of 8 for evacuation
[0.119s][info   ][gc,marking     ] GC(74) Concurrent Scan Root Regions 0.035ms
[0.119s][info   ][gc,marking     ] GC(74) Concurrent Mark
[0.119s][info   ][gc,marking     ] GC(74) Concurrent Mark From Roots
[0.119s][info   ][gc,task        ] GC(74) Using 2 workers of 2 for marking
[0.120s][info   ][gc,phases      ] GC(75)   Pre Evacuate Collection Set: 0.0ms
[0.120s][info   ][gc,phases      ] GC(75)   Merge Heap Roots: 0.0ms
[0.120s][info   ][gc,phases      ] GC(75)   Evacuate Collection Set: 0.1ms
[0.120s][info   ][gc,phases      ] GC(75)   Post Evacuate Collection Set: 0.1ms
[0.120s][info   ][gc,phases      ] GC(75)   Other: 0.0ms
[0.120s][info   ][gc,heap        ] GC(75) Eden regions: 0->0(5)
[0.120s][info   ][gc,heap        ] GC(75) Survivor regions: 1->1(1)
[0.120s][info   ][gc,heap        ] GC(75) Old regions: 75->75
[0.120s][info   ][gc,heap        ] GC(75) Archive regions: 2->2
[0.120s][info   ][gc,heap        ] GC(75) Humongous regions: 49->49
[0.120s][info   ][gc,metaspace   ] GC(75) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.120s][info   ][gc             ] GC(75) Pause Young (Normal) (G1 Preventive Collection) 122M->122M(128M) 0.262ms
[0.120s][info   ][gc,cpu         ] GC(75) User=0.00s Sys=0.01s Real=0.00s
[0.120s][info   ][gc,start       ] GC(76) Pause Young (Normal) (G1 Evacuation Pause)
[0.120s][info   ][gc,task        ] GC(76) Using 3 workers of 8 for evacuation
[0.120s][info   ][gc,phases      ] GC(76)   Pre Evacuate Collection Set: 0.0ms
[0.120s][info   ][gc,phases      ] GC(76)   Merge Heap Roots: 0.0ms
[0.120s][info   ][gc,phases      ] GC(76)   Evacuate Collection Set: 0.0ms
[0.120s][info   ][gc,phases      ] GC(76)   Post Evacuate Collection Set: 0.1ms
[0.120s][info   ][gc,phases      ] GC(76)   Other: 0.0ms
[0.120s][info   ][gc,heap        ] GC(76) Eden regions: 0->0(6)
[0.120s][info   ][gc,heap        ] GC(76) Survivor regions: 1->0(0)
[0.120s][info   ][gc,heap        ] GC(76) Old regions: 75->76
[0.120s][info   ][gc,heap        ] GC(76) Archive regions: 2->2
[0.120s][info   ][gc,heap        ] GC(76) Humongous regions: 50->49
[0.120s][info   ][gc,metaspace   ] GC(76) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.120s][info   ][gc             ] GC(76) Pause Young (Normal) (G1 Evacuation Pause) (Evacuation Failure) 123M->122M(128M) 0.255ms
[0.120s][info   ][gc,cpu         ] GC(76) User=0.00s Sys=0.00s Real=0.00s
[0.120s][info   ][gc,start       ] GC(77) Pause Young (Normal) (G1 Evacuation Pause)
[0.120s][info   ][gc,task        ] GC(77) Using 3 workers of 8 for evacuation
[0.120s][info   ][gc,phases      ] GC(77)   Pre Evacuate Collection Set: 0.0ms
[0.120s][info   ][gc,phases      ] GC(77)   Merge Heap Roots: 0.0ms
[0.120s][info   ][gc,phases      ] GC(77)   Evacuate Collection Set: 0.0ms
[0.120s][info   ][gc,phases      ] GC(77)   Post Evacuate Collection Set: 0.1ms
[0.120s][info   ][gc,phases      ] GC(77)   Other: 0.0ms
[0.120s][info   ][gc,heap        ] GC(77) Eden regions: 1->0(6)
[0.120s][info   ][gc,heap        ] GC(77) Survivor regions: 0->0(0)
[0.120s][info   ][gc,heap        ] GC(77) Old regions: 76->77
[0.120s][info   ][gc,heap        ] GC(77) Archive regions: 2->2
[0.120s][info   ][gc,heap        ] GC(77) Humongous regions: 49->49
[0.120s][info   ][gc,metaspace   ] GC(77) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.120s][info   ][gc             ] GC(77) Pause Young (Normal) (G1 Evacuation Pause) (Evacuation Failure) 123M->123M(128M) 0.258ms
[0.120s][info   ][gc,cpu         ] GC(77) User=0.00s Sys=0.00s Real=0.00s
[0.120s][info   ][gc,ergo        ] Attempting full compaction
[0.120s][info   ][gc,start       ] GC(78) Pause Full (G1 Compaction Pause)
[0.120s][info   ][gc,task        ] GC(78) Using 3 workers of 8 for full compaction
[0.120s][info   ][gc,phases,start] GC(78) Phase 1: Mark live objects
[0.121s][info   ][gc,phases      ] GC(78) Phase 1: Mark live objects 0.272ms
[0.121s][info   ][gc,phases,start] GC(78) Phase 2: Prepare compaction
[0.121s][info   ][gc,phases      ] GC(78) Phase 2: Prepare compaction 0.079ms
[0.121s][info   ][gc,phases,start] GC(78) Phase 3: Adjust pointers
[0.121s][info   ][gc,phases      ] GC(78) Phase 3: Adjust pointers 0.203ms
[0.121s][info   ][gc,phases,start] GC(78) Phase 4: Compact heap
[0.121s][info   ][gc,phases      ] GC(78) Phase 4: Compact heap 0.067ms
[0.121s][info   ][gc,heap        ] GC(78) Eden regions: 0->0(6)
[0.121s][info   ][gc,heap        ] GC(78) Survivor regions: 0->0(0)
[0.121s][info   ][gc,heap        ] GC(78) Old regions: 77->76
[0.121s][info   ][gc,heap        ] GC(78) Archive regions: 2->2
[0.121s][info   ][gc,heap        ] GC(78) Humongous regions: 49->49
[0.121s][info   ][gc,metaspace   ] GC(78) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.121s][info   ][gc             ] GC(78) Pause Full (G1 Compaction Pause) 123M->122M(128M) 0.956ms
[0.121s][info   ][gc,cpu         ] GC(78) User=0.00s Sys=0.00s Real=0.00s
[0.121s][info   ][gc,marking     ] GC(74) Concurrent Mark From Roots 1.986ms
[0.121s][info   ][gc,marking     ] GC(74) Concurrent Mark Abort
[0.121s][info   ][gc             ] GC(74) Concurrent Mark Cycle 2.084ms
[0.121s][info   ][gc,start       ] GC(79) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.121s][info   ][gc,task        ] GC(79) Using 3 workers of 8 for evacuation
[0.122s][info   ][gc,phases      ] GC(79)   Pre Evacuate Collection Set: 0.0ms
[0.122s][info   ][gc,phases      ] GC(79)   Merge Heap Roots: 0.0ms
[0.122s][info   ][gc,phases      ] GC(79)   Evacuate Collection Set: 0.1ms
[0.122s][info   ][gc,phases      ] GC(79)   Post Evacuate Collection Set: 0.0ms
[0.122s][info   ][gc,phases      ] GC(79)   Other: 0.0ms
[0.122s][info   ][gc,heap        ] GC(79) Eden regions: 1->0(6)
[0.122s][info   ][gc,heap        ] GC(79) Survivor regions: 0->0(0)
[0.122s][info   ][gc,heap        ] GC(79) Old regions: 76->77
[0.122s][info   ][gc,heap        ] GC(79) Archive regions: 2->2
[0.122s][info   ][gc,heap        ] GC(79) Humongous regions: 49->49
[0.122s][info   ][gc,metaspace   ] GC(79) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.122s][info   ][gc             ] GC(79) Pause Young (Concurrent Start) (G1 Humongous Allocation) (Evacuation Failure) 123M->123M(128M) 0.298ms
[0.122s][info   ][gc,cpu         ] GC(79) User=0.00s Sys=0.00s Real=0.00s
[0.122s][info   ][gc             ] GC(80) Concurrent Mark Cycle
[0.122s][info   ][gc,ergo        ] Attempting full compaction clearing soft references
[0.122s][info   ][gc,marking     ] GC(80) Concurrent Clear Claimed Marks
[0.122s][info   ][gc,start       ] GC(81) Pause Full (G1 Compaction Pause)
[0.122s][info   ][gc,marking     ] GC(80) Concurrent Clear Claimed Marks 0.035ms
[0.122s][info   ][gc,task        ] GC(81) Using 3 workers of 8 for full compaction
[0.122s][info   ][gc,marking     ] GC(80) Concurrent Scan Root Regions
[0.122s][info   ][gc,marking     ] GC(80) Concurrent Scan Root Regions 0.049ms
[0.122s][info   ][gc,marking     ] GC(80) Concurrent Mark
[0.122s][info   ][gc,marking     ] GC(80) Concurrent Mark From Roots
[0.122s][info   ][gc,task        ] GC(80) Using 2 workers of 2 for marking
[0.122s][info   ][gc,phases,start] GC(81) Phase 1: Mark live objects
[0.122s][info   ][gc,phases      ] GC(81) Phase 1: Mark live objects 0.317ms
[0.122s][info   ][gc,phases,start] GC(81) Phase 2: Prepare compaction
[0.122s][info   ][gc,phases      ] GC(81) Phase 2: Prepare compaction 0.062ms
[0.122s][info   ][gc,phases,start] GC(81) Phase 3: Adjust pointers
[0.123s][info   ][gc,phases      ] GC(81) Phase 3: Adjust pointers 0.217ms
[0.123s][info   ][gc,phases,start] GC(81) Phase 4: Compact heap
[0.123s][info   ][gc,phases      ] GC(81) Phase 4: Compact heap 0.056ms
[0.123s][info   ][gc,heap        ] GC(81) Eden regions: 0->0(6)
[0.123s][info   ][gc,heap        ] GC(81) Survivor regions: 0->0(0)
[0.123s][info   ][gc,heap        ] GC(81) Old regions: 77->76
[0.123s][info   ][gc,heap        ] GC(81) Archive regions: 2->2
[0.123s][info   ][gc,heap        ] GC(81) Humongous regions: 49->49
[0.123s][info   ][gc,metaspace   ] GC(81) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.123s][info   ][gc             ] GC(81) Pause Full (G1 Compaction Pause) 123M->122M(128M) 1.094ms
[0.123s][info   ][gc,cpu         ] GC(81) User=0.01s Sys=0.00s Real=0.00s
[0.123s][info   ][gc,marking     ] GC(80) Concurrent Mark From Roots 1.100ms
[0.123s][info   ][gc,marking     ] GC(80) Concurrent Mark Abort
[0.123s][info   ][gc             ] GC(80) Concurrent Mark Cycle 1.273ms
[0.123s][info   ][gc,start       ] GC(82) Pause Young (Normal) (G1 Evacuation Pause)
[0.123s][info   ][gc,task        ] GC(82) Using 3 workers of 8 for evacuation
[0.123s][info   ][gc,phases      ] GC(82)   Pre Evacuate Collection Set: 0.0ms
[0.123s][info   ][gc,phases      ] GC(82)   Merge Heap Roots: 0.0ms
[0.123s][info   ][gc,phases      ] GC(82)   Evacuate Collection Set: 0.0ms
[0.123s][info   ][gc,phases      ] GC(82)   Post Evacuate Collection Set: 0.1ms
[0.123s][info   ][gc,phases      ] GC(82)   Other: 0.1ms
[0.123s][info   ][gc,heap        ] GC(82) Eden regions: 0->0(6)
[0.123s][info   ][gc,heap        ] GC(82) Survivor regions: 0->0(0)
[0.123s][info   ][gc,heap        ] GC(82) Old regions: 76->76
[0.123s][info   ][gc,heap        ] GC(82) Archive regions: 2->2
[0.123s][info   ][gc,heap        ] GC(82) Humongous regions: 50->49
[0.123s][info   ][gc,metaspace   ] GC(82) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.123s][info   ][gc             ] GC(82) Pause Young (Normal) (G1 Evacuation Pause) 123M->122M(128M) 0.400ms
[0.123s][info   ][gc,cpu         ] GC(82) User=0.00s Sys=0.00s Real=0.01s
[0.124s][info   ][gc,start       ] GC(83) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.124s][info   ][gc,task        ] GC(83) Using 3 workers of 8 for evacuation
[0.124s][info   ][gc,phases      ] GC(83)   Pre Evacuate Collection Set: 0.0ms
[0.124s][info   ][gc,phases      ] GC(83)   Merge Heap Roots: 0.0ms
[0.124s][info   ][gc,phases      ] GC(83)   Evacuate Collection Set: 0.1ms
[0.124s][info   ][gc,phases      ] GC(83)   Post Evacuate Collection Set: 0.1ms
[0.124s][info   ][gc,phases      ] GC(83)   Other: 0.0ms
[0.124s][info   ][gc,heap        ] GC(83) Eden regions: 1->0(6)
[0.124s][info   ][gc,heap        ] GC(83) Survivor regions: 0->0(0)
[0.124s][info   ][gc,heap        ] GC(83) Old regions: 76->76
[0.124s][info   ][gc,heap        ] GC(83) Archive regions: 2->2
[0.124s][info   ][gc,heap        ] GC(83) Humongous regions: 49->49
[0.124s][info   ][gc,metaspace   ] GC(83) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.124s][info   ][gc             ] GC(83) Pause Young (Concurrent Start) (G1 Humongous Allocation) 122M->122M(128M) 0.410ms
[0.124s][info   ][gc,cpu         ] GC(83) User=0.00s Sys=0.00s Real=0.00s
[0.124s][info   ][gc             ] GC(84) Concurrent Mark Cycle
[0.124s][info   ][gc,marking     ] GC(84) Concurrent Clear Claimed Marks
[0.124s][info   ][gc,marking     ] GC(84) Concurrent Clear Claimed Marks 0.007ms
[0.124s][info   ][gc,marking     ] GC(84) Concurrent Scan Root Regions
[0.124s][info   ][gc,marking     ] GC(84) Concurrent Scan Root Regions 0.007ms
[0.124s][info   ][gc,marking     ] GC(84) Concurrent Mark
[0.124s][info   ][gc,marking     ] GC(84) Concurrent Mark From Roots
[0.124s][info   ][gc,task        ] GC(84) Using 2 workers of 2 for marking
[0.124s][info   ][gc,start       ] GC(85) Pause Young (Normal) (G1 Evacuation Pause)
[0.124s][info   ][gc,task        ] GC(85) Using 3 workers of 8 for evacuation
[0.124s][info   ][gc,phases      ] GC(85)   Pre Evacuate Collection Set: 0.0ms
[0.124s][info   ][gc,phases      ] GC(85)   Merge Heap Roots: 0.0ms
[0.124s][info   ][gc,phases      ] GC(85)   Evacuate Collection Set: 0.1ms
[0.124s][info   ][gc,phases      ] GC(85)   Post Evacuate Collection Set: 0.1ms
[0.124s][info   ][gc,phases      ] GC(85)   Other: 0.0ms
[0.124s][info   ][gc,heap        ] GC(85) Eden regions: 0->0(6)
[0.124s][info   ][gc,heap        ] GC(85) Survivor regions: 0->0(0)
[0.124s][info   ][gc,heap        ] GC(85) Old regions: 76->76
[0.124s][info   ][gc,heap        ] GC(85) Archive regions: 2->2
[0.124s][info   ][gc,heap        ] GC(85) Humongous regions: 50->49
[0.124s][info   ][gc,metaspace   ] GC(85) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.124s][info   ][gc             ] GC(85) Pause Young (Normal) (G1 Evacuation Pause) 123M->122M(128M) 0.290ms
[0.124s][info   ][gc,cpu         ] GC(85) User=0.00s Sys=0.00s Real=0.00s
[0.124s][info   ][gc,start       ] GC(86) Pause Young (Normal) (G1 Humongous Allocation)
[0.125s][info   ][gc,task        ] GC(86) Using 3 workers of 8 for evacuation
[0.125s][info   ][gc,phases      ] GC(86)   Pre Evacuate Collection Set: 0.0ms
[0.125s][info   ][gc,phases      ] GC(86)   Merge Heap Roots: 0.0ms
[0.125s][info   ][gc,phases      ] GC(86)   Evacuate Collection Set: 0.0ms
[0.125s][info   ][gc,phases      ] GC(86)   Post Evacuate Collection Set: 0.1ms
[0.125s][info   ][gc,phases      ] GC(86)   Other: 0.0ms
[0.125s][info   ][gc,heap        ] GC(86) Eden regions: 1->0(6)
[0.125s][info   ][gc,heap        ] GC(86) Survivor regions: 0->0(0)
[0.125s][info   ][gc,heap        ] GC(86) Old regions: 76->76
[0.125s][info   ][gc,heap        ] GC(86) Archive regions: 2->2
[0.125s][info   ][gc,heap        ] GC(86) Humongous regions: 49->49
[0.125s][info   ][gc,metaspace   ] GC(86) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.125s][info   ][gc             ] GC(86) Pause Young (Normal) (G1 Humongous Allocation) 122M->122M(128M) 0.351ms
[0.125s][info   ][gc,cpu         ] GC(86) User=0.00s Sys=0.00s Real=0.00s
[0.125s][info   ][gc,start       ] GC(87) Pause Young (Normal) (G1 Evacuation Pause)
[0.125s][info   ][gc,task        ] GC(87) Using 3 workers of 8 for evacuation
[0.125s][info   ][gc,phases      ] GC(87)   Pre Evacuate Collection Set: 0.0ms
[0.125s][info   ][gc,phases      ] GC(87)   Merge Heap Roots: 0.0ms
[0.125s][info   ][gc,phases      ] GC(87)   Evacuate Collection Set: 0.0ms
[0.125s][info   ][gc,phases      ] GC(87)   Post Evacuate Collection Set: 0.0ms
[0.125s][info   ][gc,phases      ] GC(87)   Other: 0.0ms
[0.125s][info   ][gc,heap        ] GC(87) Eden regions: 0->0(6)
[0.125s][info   ][gc,heap        ] GC(87) Survivor regions: 0->0(0)
[0.125s][info   ][gc,heap        ] GC(87) Old regions: 76->76
[0.125s][info   ][gc,heap        ] GC(87) Archive regions: 2->2
[0.125s][info   ][gc,heap        ] GC(87) Humongous regions: 50->50
[0.125s][info   ][gc,metaspace   ] GC(87) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.125s][info   ][gc             ] GC(87) Pause Young (Normal) (G1 Evacuation Pause) 123M->123M(128M) 0.208ms
[0.125s][info   ][gc,cpu         ] GC(87) User=0.00s Sys=0.00s Real=0.00s
[0.125s][info   ][gc,ergo        ] Attempting full compaction
[0.125s][info   ][gc,start       ] GC(88) Pause Full (G1 Compaction Pause)
[0.125s][info   ][gc,task        ] GC(88) Using 3 workers of 8 for full compaction
[0.125s][info   ][gc,phases,start] GC(88) Phase 1: Mark live objects
[0.126s][info   ][gc,phases      ] GC(88) Phase 1: Mark live objects 0.361ms
[0.126s][info   ][gc,phases,start] GC(88) Phase 2: Prepare compaction
[0.126s][info   ][gc,phases      ] GC(88) Phase 2: Prepare compaction 0.086ms
[0.126s][info   ][gc,phases,start] GC(88) Phase 3: Adjust pointers
[0.126s][info   ][gc,phases      ] GC(88) Phase 3: Adjust pointers 0.191ms
[0.126s][info   ][gc,phases,start] GC(88) Phase 4: Compact heap
[0.126s][info   ][gc,phases      ] GC(88) Phase 4: Compact heap 0.058ms
[0.126s][info   ][gc,heap        ] GC(88) Eden regions: 0->0(6)
[0.126s][info   ][gc,heap        ] GC(88) Survivor regions: 0->0(0)
[0.126s][info   ][gc,heap        ] GC(88) Old regions: 76->75
[0.126s][info   ][gc,heap        ] GC(88) Archive regions: 2->2
[0.126s][info   ][gc,heap        ] GC(88) Humongous regions: 50->50
[0.126s][info   ][gc,metaspace   ] GC(88) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.126s][info   ][gc             ] GC(88) Pause Full (G1 Compaction Pause) 123M->123M(128M) 1.086ms
[0.126s][info   ][gc,cpu         ] GC(88) User=0.00s Sys=0.00s Real=0.00s
[0.126s][info   ][gc,marking     ] GC(84) Concurrent Mark From Roots 2.316ms
[0.126s][info   ][gc,start       ] GC(89) Pause Young (Normal) (G1 Humongous Allocation)
[0.126s][info   ][gc,task        ] GC(89) Using 3 workers of 8 for evacuation
[0.127s][info   ][gc,phases      ] GC(89)   Pre Evacuate Collection Set: 0.0ms
[0.127s][info   ][gc,phases      ] GC(89)   Merge Heap Roots: 0.0ms
[0.127s][info   ][gc,phases      ] GC(89)   Evacuate Collection Set: 0.1ms
[0.127s][info   ][gc,phases      ] GC(89)   Post Evacuate Collection Set: 0.1ms
[0.127s][info   ][gc,phases      ] GC(89)   Other: 0.0ms
[0.127s][info   ][gc,heap        ] GC(89) Eden regions: 1->0(6)
[0.127s][info   ][gc,heap        ] GC(89) Survivor regions: 0->0(0)
[0.127s][info   ][gc,heap        ] GC(89) Old regions: 75->76
[0.127s][info   ][gc,heap        ] GC(89) Archive regions: 2->2
[0.127s][info   ][gc,heap        ] GC(89) Humongous regions: 50->50
[0.127s][info   ][gc,metaspace   ] GC(89) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.127s][info   ][gc             ] GC(89) Pause Young (Normal) (G1 Humongous Allocation) (Evacuation Failure) 123M->123M(128M) 0.300ms
[0.127s][info   ][gc,cpu         ] GC(89) User=0.00s Sys=0.00s Real=0.00s
[0.127s][info   ][gc,ergo        ] Attempting full compaction
[0.127s][info   ][gc,start       ] GC(90) Pause Full (G1 Compaction Pause)
[0.127s][info   ][gc,task        ] GC(90) Using 3 workers of 8 for full compaction
[0.127s][info   ][gc,phases,start] GC(90) Phase 1: Mark live objects
[0.127s][info   ][gc,phases      ] GC(90) Phase 1: Mark live objects 0.310ms
[0.127s][info   ][gc,phases,start] GC(90) Phase 2: Prepare compaction
[0.127s][info   ][gc,phases      ] GC(90) Phase 2: Prepare compaction 0.100ms
[0.127s][info   ][gc,phases,start] GC(90) Phase 3: Adjust pointers
[0.127s][info   ][gc,phases      ] GC(90) Phase 3: Adjust pointers 0.195ms
[0.127s][info   ][gc,phases,start] GC(90) Phase 4: Compact heap
[0.127s][info   ][gc,phases      ] GC(90) Phase 4: Compact heap 0.047ms
[0.128s][info   ][gc,heap        ] GC(90) Eden regions: 0->0(6)
[0.128s][info   ][gc,heap        ] GC(90) Survivor regions: 0->0(0)
[0.128s][info   ][gc,heap        ] GC(90) Old regions: 76->75
[0.128s][info   ][gc,heap        ] GC(90) Archive regions: 2->2
[0.128s][info   ][gc,heap        ] GC(90) Humongous regions: 50->50
[0.128s][info   ][gc,metaspace   ] GC(90) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.128s][info   ][gc             ] GC(90) Pause Full (G1 Compaction Pause) 123M->123M(128M) 0.996ms
[0.128s][info   ][gc,cpu         ] GC(90) User=0.00s Sys=0.00s Real=0.00s
[0.128s][info   ][gc,marking     ] GC(84) Concurrent Mark Abort
[0.128s][info   ][gc             ] GC(84) Concurrent Mark Cycle 3.785ms
[0.128s][info   ][gc,start       ] GC(91) Pause Young (Normal) (G1 Evacuation Pause)
[0.128s][info   ][gc,task        ] GC(91) Using 3 workers of 8 for evacuation
[0.128s][info   ][gc,phases      ] GC(91)   Pre Evacuate Collection Set: 0.0ms
[0.128s][info   ][gc,phases      ] GC(91)   Merge Heap Roots: 0.0ms
[0.128s][info   ][gc,phases      ] GC(91)   Evacuate Collection Set: 0.0ms
[0.128s][info   ][gc,phases      ] GC(91)   Post Evacuate Collection Set: 0.1ms
[0.128s][info   ][gc,phases      ] GC(91)   Other: 0.0ms
[0.128s][info   ][gc,heap        ] GC(91) Eden regions: 0->0(6)
[0.128s][info   ][gc,heap        ] GC(91) Survivor regions: 0->0(0)
[0.128s][info   ][gc,heap        ] GC(91) Old regions: 75->75
[0.128s][info   ][gc,heap        ] GC(91) Archive regions: 2->2
[0.128s][info   ][gc,heap        ] GC(91) Humongous regions: 51->50
[0.128s][info   ][gc,metaspace   ] GC(91) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.128s][info   ][gc             ] GC(91) Pause Young (Normal) (G1 Evacuation Pause) 124M->123M(128M) 0.285ms
[0.128s][info   ][gc,cpu         ] GC(91) User=0.00s Sys=0.00s Real=0.00s
[0.128s][info   ][gc,start       ] GC(92) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.128s][info   ][gc,task        ] GC(92) Using 3 workers of 8 for evacuation
[0.128s][info   ][gc,phases      ] GC(92)   Pre Evacuate Collection Set: 0.0ms
[0.128s][info   ][gc,phases      ] GC(92)   Merge Heap Roots: 0.0ms
[0.128s][info   ][gc,phases      ] GC(92)   Evacuate Collection Set: 0.1ms
[0.128s][info   ][gc,phases      ] GC(92)   Post Evacuate Collection Set: 0.1ms
[0.128s][info   ][gc,phases      ] GC(92)   Other: 0.0ms
[0.128s][info   ][gc,heap        ] GC(92) Eden regions: 1->0(6)
[0.128s][info   ][gc,heap        ] GC(92) Survivor regions: 0->0(0)
[0.129s][info   ][gc,heap        ] GC(92) Old regions: 75->76
[0.129s][info   ][gc,heap        ] GC(92) Archive regions: 2->2
[0.129s][info   ][gc,heap        ] GC(92) Humongous regions: 50->50
[0.129s][info   ][gc,metaspace   ] GC(92) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.129s][info   ][gc             ] GC(92) Pause Young (Concurrent Start) (G1 Humongous Allocation) (Evacuation Failure) 123M->123M(128M) 0.325ms
[0.129s][info   ][gc,cpu         ] GC(92) User=0.00s Sys=0.00s Real=0.00s
[0.129s][info   ][gc,ergo        ] Attempting full compaction clearing soft references
[0.129s][info   ][gc,start       ] GC(93) Pause Full (G1 Compaction Pause)
[0.129s][info   ][gc,task        ] GC(93) Using 3 workers of 8 for full compaction
[0.129s][info   ][gc             ] GC(94) Concurrent Mark Cycle
[0.129s][info   ][gc,marking     ] GC(94) Concurrent Clear Claimed Marks
[0.129s][info   ][gc,marking     ] GC(94) Concurrent Clear Claimed Marks 0.007ms
[0.129s][info   ][gc,marking     ] GC(94) Concurrent Scan Root Regions
[0.129s][info   ][gc,marking     ] GC(94) Concurrent Scan Root Regions 0.006ms
[0.129s][info   ][gc,marking     ] GC(94) Concurrent Mark
[0.129s][info   ][gc,marking     ] GC(94) Concurrent Mark From Roots
[0.129s][info   ][gc,task        ] GC(94) Using 2 workers of 2 for marking
[0.129s][info   ][gc,phases,start] GC(93) Phase 1: Mark live objects
[0.129s][info   ][gc,phases      ] GC(93) Phase 1: Mark live objects 0.309ms
[0.129s][info   ][gc,phases,start] GC(93) Phase 2: Prepare compaction
[0.129s][info   ][gc,phases      ] GC(93) Phase 2: Prepare compaction 0.065ms
[0.129s][info   ][gc,phases,start] GC(93) Phase 3: Adjust pointers
[0.129s][info   ][gc,phases      ] GC(93) Phase 3: Adjust pointers 0.217ms
[0.129s][info   ][gc,phases,start] GC(93) Phase 4: Compact heap
[0.129s][info   ][gc,phases      ] GC(93) Phase 4: Compact heap 0.032ms
[0.130s][info   ][gc,heap        ] GC(93) Eden regions: 0->0(6)
[0.130s][info   ][gc,heap        ] GC(93) Survivor regions: 0->0(0)
[0.130s][info   ][gc,heap        ] GC(93) Old regions: 76->75
[0.130s][info   ][gc,heap        ] GC(93) Archive regions: 2->2
[0.130s][info   ][gc,heap        ] GC(93) Humongous regions: 50->50
[0.130s][info   ][gc,metaspace   ] GC(93) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.130s][info   ][gc             ] GC(93) Pause Full (G1 Compaction Pause) 123M->123M(128M) 1.008ms
[0.130s][info   ][gc,cpu         ] GC(93) User=0.00s Sys=0.00s Real=0.00s
[0.130s][info   ][gc,marking     ] GC(94) Concurrent Mark From Roots 0.978ms
[0.130s][info   ][gc,start       ] GC(95) Pause Young (Normal) (G1 Evacuation Pause)
[0.130s][info   ][gc,task        ] GC(95) Using 3 workers of 8 for evacuation
[0.130s][info   ][gc,phases      ] GC(95)   Pre Evacuate Collection Set: 0.0ms
[0.130s][info   ][gc,phases      ] GC(95)   Merge Heap Roots: 0.0ms
[0.130s][info   ][gc,phases      ] GC(95)   Evacuate Collection Set: 0.0ms
[0.130s][info   ][gc,phases      ] GC(95)   Post Evacuate Collection Set: 0.1ms
[0.130s][info   ][gc,phases      ] GC(95)   Other: 0.0ms
[0.130s][info   ][gc,heap        ] GC(95) Eden regions: 0->0(6)
[0.130s][info   ][gc,heap        ] GC(95) Survivor regions: 0->0(0)
[0.130s][info   ][gc,heap        ] GC(95) Old regions: 75->75
[0.130s][info   ][gc,heap        ] GC(95) Archive regions: 2->2
[0.130s][info   ][gc,heap        ] GC(95) Humongous regions: 51->51
[0.130s][info   ][gc,metaspace   ] GC(95) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.130s][info   ][gc             ] GC(95) Pause Young (Normal) (G1 Evacuation Pause) 124M->124M(128M) 0.266ms
[0.130s][info   ][gc,cpu         ] GC(95) User=0.00s Sys=0.00s Real=0.00s
[0.130s][info   ][gc,ergo        ] Attempting full compaction
[0.130s][info   ][gc,start       ] GC(96) Pause Full (G1 Compaction Pause)
[0.130s][info   ][gc,task        ] GC(96) Using 3 workers of 8 for full compaction
[0.130s][info   ][gc,phases,start] GC(96) Phase 1: Mark live objects
[0.130s][info   ][gc,phases      ] GC(96) Phase 1: Mark live objects 0.279ms
[0.130s][info   ][gc,phases,start] GC(96) Phase 2: Prepare compaction
[0.130s][info   ][gc,phases      ] GC(96) Phase 2: Prepare compaction 0.081ms
[0.130s][info   ][gc,phases,start] GC(96) Phase 3: Adjust pointers
[0.131s][info   ][gc,phases      ] GC(96) Phase 3: Adjust pointers 0.189ms
[0.131s][info   ][gc,phases,start] GC(96) Phase 4: Compact heap
[0.131s][info   ][gc,phases      ] GC(96) Phase 4: Compact heap 0.061ms
[0.131s][info   ][gc,heap        ] GC(96) Eden regions: 0->0(6)
[0.131s][info   ][gc,heap        ] GC(96) Survivor regions: 0->0(0)
[0.131s][info   ][gc,heap        ] GC(96) Old regions: 75->75
[0.131s][info   ][gc,heap        ] GC(96) Archive regions: 2->2
[0.131s][info   ][gc,heap        ] GC(96) Humongous regions: 51->51
[0.131s][info   ][gc,metaspace   ] GC(96) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.131s][info   ][gc             ] GC(96) Pause Full (G1 Compaction Pause) 124M->124M(128M) 0.982ms
[0.131s][info   ][gc,cpu         ] GC(96) User=0.00s Sys=0.00s Real=0.00s
[0.131s][info   ][gc,ergo        ] Attempting maximal full compaction clearing soft references
[0.131s][info   ][gc,start       ] GC(97) Pause Full (G1 Compaction Pause)
[0.131s][info   ][gc,task        ] GC(97) Using 3 workers of 8 for full compaction
[0.131s][info   ][gc,phases,start] GC(97) Phase 1: Mark live objects
[0.131s][info   ][gc,phases      ] GC(97) Phase 1: Mark live objects 0.362ms
[0.131s][info   ][gc,phases,start] GC(97) Phase 2: Prepare compaction
[0.131s][info   ][gc,phases      ] GC(97) Phase 2: Prepare compaction 0.080ms
[0.132s][info   ][gc,phases,start] GC(97) Phase 3: Adjust pointers
[0.132s][info   ][gc,phases      ] GC(97) Phase 3: Adjust pointers 0.247ms
[0.132s][info   ][gc,phases,start] GC(97) Phase 4: Compact heap
[0.132s][info   ][gc,phases      ] GC(97) Phase 4: Compact heap 0.171ms
[0.132s][info   ][gc,heap        ] GC(97) Eden regions: 0->0(6)
[0.132s][info   ][gc,heap        ] GC(97) Survivor regions: 0->0(0)
[0.132s][info   ][gc,heap        ] GC(97) Old regions: 75->75
[0.132s][info   ][gc,heap        ] GC(97) Archive regions: 2->2
[0.132s][info   ][gc,heap        ] GC(97) Humongous regions: 51->51
[0.132s][info   ][gc,metaspace   ] GC(97) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.132s][info   ][gc             ] GC(97) Pause Full (G1 Compaction Pause) 124M->123M(128M) 1.263ms
[0.132s][info   ][gc,cpu         ] GC(97) User=0.00s Sys=0.00s Real=0.00s
[0.132s][info   ][gc,marking     ] GC(94) Concurrent Mark Abort
[0.132s][info   ][gc             ] GC(94) Concurrent Mark Cycle 3.715ms
[0.132s][info   ][gc,start       ] GC(98) Pause Young (Normal) (G1 Evacuation Pause)
[0.132s][info   ][gc,task        ] GC(98) Using 3 workers of 8 for evacuation
[0.133s][info   ][gc,phases      ] GC(98)   Pre Evacuate Collection Set: 0.0ms
[0.133s][info   ][gc,phases      ] GC(98)   Merge Heap Roots: 0.0ms
[0.133s][info   ][gc,phases      ] GC(98)   Evacuate Collection Set: 0.0ms
[0.133s][info   ][gc,phases      ] GC(98)   Post Evacuate Collection Set: 0.1ms
[0.133s][info   ][gc,phases      ] GC(98)   Other: 0.0ms
[0.133s][info   ][gc,heap        ] GC(98) Eden regions: 0->0(6)
[0.133s][info   ][gc,heap        ] GC(98) Survivor regions: 0->0(0)
[0.133s][info   ][gc,heap        ] GC(98) Old regions: 75->75
[0.133s][info   ][gc,heap        ] GC(98) Archive regions: 2->2
[0.133s][info   ][gc,heap        ] GC(98) Humongous regions: 51->51
[0.133s][info   ][gc,metaspace   ] GC(98) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.133s][info   ][gc             ] GC(98) Pause Young (Normal) (G1 Evacuation Pause) 123M->123M(128M) 0.305ms
[0.133s][info   ][gc,cpu         ] GC(98) User=0.00s Sys=0.00s Real=0.00s
[0.133s][info   ][gc,ergo        ] Attempting full compaction
[0.133s][info   ][gc,start       ] GC(99) Pause Full (G1 Compaction Pause)
[0.133s][info   ][gc,task        ] GC(99) Using 3 workers of 8 for full compaction
[0.133s][info   ][gc,phases,start] GC(99) Phase 1: Mark live objects
[0.133s][info   ][gc,phases      ] GC(99) Phase 1: Mark live objects 0.334ms
[0.133s][info   ][gc,phases,start] GC(99) Phase 2: Prepare compaction
[0.133s][info   ][gc,phases      ] GC(99) Phase 2: Prepare compaction 0.051ms
[0.133s][info   ][gc,phases,start] GC(99) Phase 3: Adjust pointers
[0.133s][info   ][gc,phases      ] GC(99) Phase 3: Adjust pointers 0.205ms
[0.133s][info   ][gc,phases,start] GC(99) Phase 4: Compact heap
[0.133s][info   ][gc,phases      ] GC(99) Phase 4: Compact heap 0.035ms
[0.134s][info   ][gc,heap        ] GC(99) Eden regions: 0->0(6)
[0.134s][info   ][gc,heap        ] GC(99) Survivor regions: 0->0(0)
[0.134s][info   ][gc,heap        ] GC(99) Old regions: 75->75
[0.134s][info   ][gc,heap        ] GC(99) Archive regions: 2->2
[0.134s][info   ][gc,heap        ] GC(99) Humongous regions: 51->51
[0.134s][info   ][gc,metaspace   ] GC(99) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.134s][info   ][gc             ] GC(99) Pause Full (G1 Compaction Pause) 123M->123M(128M) 0.932ms
[0.134s][info   ][gc,cpu         ] GC(99) User=0.01s Sys=0.00s Real=0.01s
[0.134s][info   ][gc,ergo        ] Attempting maximal full compaction clearing soft references
[0.134s][info   ][gc,start       ] GC(100) Pause Full (G1 Compaction Pause)
[0.134s][info   ][gc,task        ] GC(100) Using 3 workers of 8 for full compaction
[0.134s][info   ][gc,phases,start] GC(100) Phase 1: Mark live objects
[0.134s][info   ][gc,phases      ] GC(100) Phase 1: Mark live objects 0.287ms
[0.134s][info   ][gc,phases,start] GC(100) Phase 2: Prepare compaction
[0.134s][info   ][gc,phases      ] GC(100) Phase 2: Prepare compaction 0.121ms
[0.134s][info   ][gc,phases,start] GC(100) Phase 3: Adjust pointers
[0.134s][info   ][gc,phases      ] GC(100) Phase 3: Adjust pointers 0.203ms
[0.134s][info   ][gc,phases,start] GC(100) Phase 4: Compact heap
[0.134s][info   ][gc,phases      ] GC(100) Phase 4: Compact heap 0.045ms
[0.134s][info   ][gc,heap        ] GC(100) Eden regions: 0->0(6)
[0.135s][info   ][gc,heap        ] GC(100) Survivor regions: 0->0(0)
[0.135s][info   ][gc,heap        ] GC(100) Old regions: 75->75
[0.135s][info   ][gc,heap        ] GC(100) Archive regions: 2->2
[0.135s][info   ][gc,heap        ] GC(100) Humongous regions: 51->51
[0.135s][info   ][gc,metaspace   ] GC(100) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.135s][info   ][gc             ] GC(100) Pause Full (G1 Compaction Pause) 123M->123M(128M) 0.932ms
[0.135s][info   ][gc,cpu         ] GC(100) User=0.00s Sys=0.00s Real=0.00s
[0.135s][info   ][gc,start       ] GC(101) Pause Young (Concurrent Start) (G1 Evacuation Pause)
[0.135s][info   ][gc,task        ] GC(101) Using 3 workers of 8 for evacuation
[0.135s][info   ][gc,phases      ] GC(101)   Pre Evacuate Collection Set: 0.0ms
[0.135s][info   ][gc,phases      ] GC(101)   Merge Heap Roots: 0.0ms
[0.135s][info   ][gc,phases      ] GC(101)   Evacuate Collection Set: 0.1ms
[0.135s][info   ][gc,phases      ] GC(101)   Post Evacuate Collection Set: 0.1ms
[0.135s][info   ][gc,phases      ] GC(101)   Other: 0.0ms
[0.135s][info   ][gc,heap        ] GC(101) Eden regions: 0->0(6)
[0.135s][info   ][gc,heap        ] GC(101) Survivor regions: 0->0(0)
[0.135s][info   ][gc,heap        ] GC(101) Old regions: 75->75
[0.135s][info   ][gc,heap        ] GC(101) Archive regions: 2->2
[0.135s][info   ][gc,heap        ] GC(101) Humongous regions: 51->51
[0.135s][info   ][gc,metaspace   ] GC(101) Metaspace: 156K(384K)->156K(384K) NonClass: 151K(256K)->151K(256K) Class: 5K(128K)->5K(128K)
[0.135s][info   ][gc             ] GC(101) Pause Young (Concurrent Start) (G1 Evacuation Pause) 123M->123M(128M) 0.309ms
[0.135s][info   ][gc,cpu         ] GC(101) User=0.00s Sys=0.00s Real=0.00s
[0.135s][info   ][gc,ergo        ] Attempting full compaction
[0.135s][info   ][gc             ] GC(102) Concurrent Mark Cycle
[0.135s][info   ][gc,start       ] GC(103) Pause Full (G1 Compaction Pause)
[0.135s][info   ][gc,marking     ] GC(102) Concurrent Clear Claimed Marks
[0.135s][info   ][gc,task        ] GC(103) Using 3 workers of 8 for full compaction
[0.135s][info   ][gc,marking     ] GC(102) Concurrent Clear Claimed Marks 0.039ms
[0.135s][info   ][gc,marking     ] GC(102) Concurrent Scan Root Regions
[0.135s][info   ][gc,marking     ] GC(102) Concurrent Scan Root Regions 0.015ms
[0.135s][info   ][gc,marking     ] GC(102) Concurrent Mark
[0.135s][info   ][gc,marking     ] GC(102) Concurrent Mark From Roots
[0.135s][info   ][gc,task        ] GC(102) Using 2 workers of 2 for marking
[0.135s][info   ][gc,phases,start] GC(103) Phase 1: Mark live objects
[0.135s][info   ][gc,phases      ] GC(103) Phase 1: Mark live objects 0.282ms
[0.135s][info   ][gc,phases,start] GC(103) Phase 2: Prepare compaction
[0.136s][info   ][gc,phases      ] GC(103) Phase 2: Prepare compaction 0.083ms
[0.136s][info   ][gc,phases,start] GC(103) Phase 3: Adjust pointers
[0.136s][info   ][gc,phases      ] GC(103) Phase 3: Adjust pointers 0.190ms
[0.136s][info   ][gc,phases,start] GC(103) Phase 4: Compact heap
[0.136s][info   ][gc,phases      ] GC(103) Phase 4: Compact heap 0.067ms
[0.136s][info   ][gc,heap        ] GC(103) Eden regions: 0->0(76)
[0.136s][info   ][gc,heap        ] GC(103) Survivor regions: 0->0(0)
[0.136s][info   ][gc,heap        ] GC(103) Old regions: 75->3
[0.136s][info   ][gc,heap        ] GC(103) Archive regions: 2->2
[0.136s][info   ][gc,heap        ] GC(103) Humongous regions: 51->0
[0.136s][info   ][gc,metaspace   ] GC(103) Metaspace: 156K(384K)->156K(384K) NonClass: 151K(256K)->151K(256K) Class: 5K(128K)->5K(128K)
[0.136s][info   ][gc             ] GC(103) Pause Full (G1 Compaction Pause) 123M->1M(128M) 1.088ms
[0.136s][info   ][gc,cpu         ] GC(103) User=0.00s Sys=0.00s Real=0.00s
[0.136s][info   ][gc,marking     ] GC(102) Concurrent Mark From Roots 0.993ms
[0.136s][info   ][gc,marking     ] GC(102) Concurrent Mark Abort
[0.136s][info   ][gc             ] GC(102) Concurrent Mark Cycle 1.161ms
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at GCLogAnalysis.generateGarbage(GCLogAnalysis.java:42)
	at GCLogAnalysis.main(GCLogAnalysis.java:20)
[0.137s][info   ][gc,heap,exit   ] Heap
[0.137s][info   ][gc,heap,exit   ]  garbage-first heap   total 131072K, used 1717K [0x00000007f8000000, 0x0000000800000000)
[0.137s][info   ][gc,heap,exit   ]   region size 1024K, 1 young (1024K), 0 survivors (0K)
[0.137s][info   ][gc,heap,exit   ]  Metaspace       used 163K, committed 384K, reserved 1114112K
[0.137s][info   ][gc,heap,exit   ]   class space    used 7K, committed 128K, reserved 1048576K
```

接下来试试512m内存
```
java -XX:+UseG1GC -Xms512m -Xmx512m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以看到GC了600多次，才生成了29000多个对象。效率很低。但是比并行GC还是要好一些。
```
[0.944s][info   ][gc,task        ] GC(590) Using 8 workers of 8 for evacuation
[0.945s][info   ][gc,phases      ] GC(590)   Pre Evacuate Collection Set: 0.1ms
[0.945s][info   ][gc,phases      ] GC(590)   Merge Heap Roots: 0.1ms
[0.945s][info   ][gc,phases      ] GC(590)   Evacuate Collection Set: 0.2ms
[0.945s][info   ][gc,phases      ] GC(590)   Post Evacuate Collection Set: 0.1ms
[0.945s][info   ][gc,phases      ] GC(590)   Other: 0.0ms
[0.945s][info   ][gc,heap        ] GC(590) Eden regions: 24->0(21)
[0.945s][info   ][gc,heap        ] GC(590) Survivor regions: 1->4(4)
[0.945s][info   ][gc,heap        ] GC(590) Old regions: 275->280
[0.945s][info   ][gc,heap        ] GC(590) Archive regions: 2->2
[0.945s][info   ][gc,heap        ] GC(590) Humongous regions: 178->170
[0.945s][info   ][gc,metaspace   ] GC(590) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.945s][info   ][gc             ] GC(590) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 453M->429M(512M) 0.711ms
[0.945s][info   ][gc,cpu         ] GC(590) User=0.00s Sys=0.00s Real=0.00s
[0.947s][info   ][gc,start       ] GC(591) Pause Young (Mixed) (G1 Evacuation Pause)
[0.947s][info   ][gc,task        ] GC(591) Using 8 workers of 8 for evacuation
[0.947s][info   ][gc,phases      ] GC(591)   Pre Evacuate Collection Set: 0.1ms
[0.947s][info   ][gc,phases      ] GC(591)   Merge Heap Roots: 0.1ms
[0.947s][info   ][gc,phases      ] GC(591)   Evacuate Collection Set: 0.5ms
[0.947s][info   ][gc,phases      ] GC(591)   Post Evacuate Collection Set: 0.1ms
[0.947s][info   ][gc,phases      ] GC(591)   Other: 0.0ms
[0.947s][info   ][gc,heap        ] GC(591) Eden regions: 21->0(21)
[0.947s][info   ][gc,heap        ] GC(591) Survivor regions: 4->4(4)
[0.947s][info   ][gc,heap        ] GC(591) Old regions: 280->281
[0.947s][info   ][gc,heap        ] GC(591) Archive regions: 2->2
[0.947s][info   ][gc,heap        ] GC(591) Humongous regions: 178->167
[0.947s][info   ][gc,metaspace   ] GC(591) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.947s][info   ][gc             ] GC(591) Pause Young (Mixed) (G1 Evacuation Pause) 458M->430M(512M) 0.889ms
[0.947s][info   ][gc,cpu         ] GC(591) User=0.00s Sys=0.00s Real=0.00s
[0.948s][info   ][gc,start       ] GC(592) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.948s][info   ][gc,task        ] GC(592) Using 8 workers of 8 for evacuation
[0.948s][info   ][gc,phases      ] GC(592)   Pre Evacuate Collection Set: 0.1ms
[0.948s][info   ][gc,phases      ] GC(592)   Merge Heap Roots: 0.0ms
[0.948s][info   ][gc,phases      ] GC(592)   Evacuate Collection Set: 0.2ms
[0.948s][info   ][gc,phases      ] GC(592)   Post Evacuate Collection Set: 0.1ms
[0.948s][info   ][gc,phases      ] GC(592)   Other: 0.0ms
[0.948s][info   ][gc,heap        ] GC(592) Eden regions: 4->0(23)
[0.948s][info   ][gc,heap        ] GC(592) Survivor regions: 4->2(4)
[0.948s][info   ][gc,heap        ] GC(592) Old regions: 281->284
[0.948s][info   ][gc,heap        ] GC(592) Archive regions: 2->2
[0.948s][info   ][gc,heap        ] GC(592) Humongous regions: 167->167
[0.949s][info   ][gc,metaspace   ] GC(592) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.949s][info   ][gc             ] GC(592) Pause Young (Concurrent Start) (G1 Humongous Allocation) 433M->431M(512M) 0.703ms
[0.949s][info   ][gc,cpu         ] GC(592) User=0.01s Sys=0.00s Real=0.00s
[0.949s][info   ][gc             ] GC(593) Concurrent Mark Cycle
[0.949s][info   ][gc,marking     ] GC(593) Concurrent Clear Claimed Marks
[0.949s][info   ][gc,marking     ] GC(593) Concurrent Clear Claimed Marks 0.009ms
[0.949s][info   ][gc,marking     ] GC(593) Concurrent Scan Root Regions
[0.949s][info   ][gc,marking     ] GC(593) Concurrent Scan Root Regions 0.032ms
[0.949s][info   ][gc,marking     ] GC(593) Concurrent Mark
[0.949s][info   ][gc,marking     ] GC(593) Concurrent Mark From Roots
[0.949s][info   ][gc,task        ] GC(593) Using 2 workers of 2 for marking
[0.950s][info   ][gc,marking     ] GC(593) Concurrent Mark From Roots 0.869ms
[0.950s][info   ][gc,marking     ] GC(593) Concurrent Preclean
[0.950s][info   ][gc,marking     ] GC(593) Concurrent Preclean 0.008ms
[0.950s][info   ][gc,start       ] GC(593) Pause Remark
[0.950s][info   ][gc             ] GC(593) Pause Remark 458M->458M(512M) 0.282ms
[0.950s][info   ][gc,cpu         ] GC(593) User=0.00s Sys=0.00s Real=0.00s
[0.950s][info   ][gc,marking     ] GC(593) Concurrent Mark 1.361ms
[0.950s][info   ][gc,marking     ] GC(593) Concurrent Rebuild Remembered Sets
[0.950s][info   ][gc,marking     ] GC(593) Concurrent Rebuild Remembered Sets 0.487ms
[0.951s][info   ][gc,start       ] GC(593) Pause Cleanup
[0.951s][info   ][gc             ] GC(593) Pause Cleanup 467M->467M(512M) 0.215ms
[0.951s][info   ][gc,cpu         ] GC(593) User=0.00s Sys=0.00s Real=0.00s
[0.951s][info   ][gc,marking     ] GC(593) Concurrent Cleanup for Next Mark
[0.951s][info   ][gc,marking     ] GC(593) Concurrent Cleanup for Next Mark 0.277ms
[0.951s][info   ][gc             ] GC(593) Concurrent Mark Cycle 2.642ms
[0.951s][info   ][gc,start       ] GC(594) Pause Young (Prepare Mixed) (G1 Preventive Collection)
[0.951s][info   ][gc,task        ] GC(594) Using 8 workers of 8 for evacuation
[0.952s][info   ][gc,phases      ] GC(594)   Pre Evacuate Collection Set: 0.1ms
[0.952s][info   ][gc,phases      ] GC(594)   Merge Heap Roots: 0.1ms
[0.952s][info   ][gc,phases      ] GC(594)   Evacuate Collection Set: 0.3ms
[0.952s][info   ][gc,phases      ] GC(594)   Post Evacuate Collection Set: 0.2ms
[0.952s][info   ][gc,phases      ] GC(594)   Other: 0.0ms
[0.952s][info   ][gc,heap        ] GC(594) Eden regions: 22->0(21)
[0.952s][info   ][gc,heap        ] GC(594) Survivor regions: 2->4(4)
[0.952s][info   ][gc,heap        ] GC(594) Old regions: 284->290
[0.952s][info   ][gc,heap        ] GC(594) Archive regions: 2->2
[0.952s][info   ][gc,heap        ] GC(594) Humongous regions: 189->174
[0.952s][info   ][gc,metaspace   ] GC(594) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.952s][info   ][gc             ] GC(594) Pause Young (Prepare Mixed) (G1 Preventive Collection) 475M->446M(512M) 0.786ms
[0.952s][info   ][gc,cpu         ] GC(594) User=0.00s Sys=0.00s Real=0.00s
[0.954s][info   ][gc,start       ] GC(595) Pause Young (Mixed) (G1 Preventive Collection)
[0.954s][info   ][gc,task        ] GC(595) Using 8 workers of 8 for evacuation
[0.955s][info   ][gc,phases      ] GC(595)   Pre Evacuate Collection Set: 0.1ms
[0.955s][info   ][gc,phases      ] GC(595)   Merge Heap Roots: 0.0ms
[0.955s][info   ][gc,phases      ] GC(595)   Evacuate Collection Set: 0.5ms
[0.955s][info   ][gc,phases      ] GC(595)   Post Evacuate Collection Set: 0.2ms
[0.955s][info   ][gc,phases      ] GC(595)   Other: 0.0ms
[0.955s][info   ][gc,heap        ] GC(595) Eden regions: 19->0(21)
[0.955s][info   ][gc,heap        ] GC(595) Survivor regions: 4->4(4)
[0.955s][info   ][gc,heap        ] GC(595) Old regions: 290->314
[0.955s][info   ][gc,heap        ] GC(595) Archive regions: 2->2
[0.955s][info   ][gc,heap        ] GC(595) Humongous regions: 181->173
[0.955s][info   ][gc,metaspace   ] GC(595) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.955s][info   ][gc             ] GC(595) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 471M->469M(512M) 1.102ms
[0.955s][info   ][gc,cpu         ] GC(595) User=0.01s Sys=0.00s Real=0.00s
[0.955s][info   ][gc,start       ] GC(596) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.955s][info   ][gc,task        ] GC(596) Using 8 workers of 8 for evacuation
[0.955s][info   ][gc,phases      ] GC(596)   Pre Evacuate Collection Set: 0.2ms
[0.955s][info   ][gc,phases      ] GC(596)   Merge Heap Roots: 0.0ms
[0.955s][info   ][gc,phases      ] GC(596)   Evacuate Collection Set: 0.2ms
[0.955s][info   ][gc,phases      ] GC(596)   Post Evacuate Collection Set: 0.1ms
[0.955s][info   ][gc,phases      ] GC(596)   Other: 0.0ms
[0.955s][info   ][gc,heap        ] GC(596) Eden regions: 1->0(24)
[0.955s][info   ][gc,heap        ] GC(596) Survivor regions: 4->1(4)
[0.955s][info   ][gc,heap        ] GC(596) Old regions: 314->318
[0.955s][info   ][gc,heap        ] GC(596) Archive regions: 2->2
[0.955s][info   ][gc,heap        ] GC(596) Humongous regions: 174->174
[0.955s][info   ][gc,metaspace   ] GC(596) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.955s][info   ][gc             ] GC(596) Pause Young (Concurrent Start) (G1 Humongous Allocation) 471M->470M(512M) 0.684ms
[0.955s][info   ][gc,cpu         ] GC(596) User=0.00s Sys=0.01s Real=0.00s
[0.956s][info   ][gc             ] GC(597) Concurrent Mark Cycle
[0.956s][info   ][gc,marking     ] GC(597) Concurrent Clear Claimed Marks
[0.956s][info   ][gc,marking     ] GC(597) Concurrent Clear Claimed Marks 0.007ms
[0.956s][info   ][gc,marking     ] GC(597) Concurrent Scan Root Regions
[0.956s][info   ][gc,marking     ] GC(597) Concurrent Scan Root Regions 0.068ms
[0.956s][info   ][gc,marking     ] GC(597) Concurrent Mark
[0.956s][info   ][gc,marking     ] GC(597) Concurrent Mark From Roots
[0.956s][info   ][gc,task        ] GC(597) Using 2 workers of 2 for marking
[0.956s][info   ][gc,start       ] GC(598) Pause Young (Normal) (G1 Preventive Collection)
[0.956s][info   ][gc,task        ] GC(598) Using 8 workers of 8 for evacuation
[0.957s][info   ][gc,phases      ] GC(598)   Pre Evacuate Collection Set: 0.1ms
[0.957s][info   ][gc,phases      ] GC(598)   Merge Heap Roots: 0.0ms
[0.957s][info   ][gc,phases      ] GC(598)   Evacuate Collection Set: 0.1ms
[0.957s][info   ][gc,phases      ] GC(598)   Post Evacuate Collection Set: 0.1ms
[0.957s][info   ][gc,phases      ] GC(598)   Other: 0.0ms
[0.957s][info   ][gc,heap        ] GC(598) Eden regions: 9->0(22)
[0.957s][info   ][gc,heap        ] GC(598) Survivor regions: 1->3(4)
[0.957s][info   ][gc,heap        ] GC(598) Old regions: 318->318
[0.957s][info   ][gc,heap        ] GC(598) Archive regions: 2->2
[0.957s][info   ][gc,heap        ] GC(598) Humongous regions: 178->174
[0.957s][info   ][gc,metaspace   ] GC(598) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.957s][info   ][gc             ] GC(598) Pause Young (Normal) (G1 Preventive Collection) 483M->473M(512M) 0.579ms
[0.957s][info   ][gc,cpu         ] GC(598) User=0.00s Sys=0.00s Real=0.00s
[0.957s][info   ][gc,marking     ] GC(597) Concurrent Mark From Roots 1.518ms
[0.957s][info   ][gc,marking     ] GC(597) Concurrent Preclean
[0.957s][info   ][gc,marking     ] GC(597) Concurrent Preclean 0.008ms
[0.957s][info   ][gc,start       ] GC(597) Pause Remark
[0.958s][info   ][gc             ] GC(597) Pause Remark 479M->479M(512M) 0.423ms
[0.958s][info   ][gc,cpu         ] GC(597) User=0.00s Sys=0.00s Real=0.00s
[0.958s][info   ][gc,marking     ] GC(597) Concurrent Mark 2.117ms
[0.958s][info   ][gc,marking     ] GC(597) Concurrent Rebuild Remembered Sets
[0.958s][info   ][gc,start       ] GC(599) Pause Young (Normal) (G1 Preventive Collection)
[0.958s][info   ][gc,task        ] GC(599) Using 8 workers of 8 for evacuation
[0.959s][info   ][gc,phases      ] GC(599)   Pre Evacuate Collection Set: 0.1ms
[0.959s][info   ][gc,phases      ] GC(599)   Merge Heap Roots: 0.1ms
[0.959s][info   ][gc,phases      ] GC(599)   Evacuate Collection Set: 0.3ms
[0.959s][info   ][gc,phases      ] GC(599)   Post Evacuate Collection Set: 0.3ms
[0.959s][info   ][gc,phases      ] GC(599)   Other: 0.0ms
[0.959s][info   ][gc,heap        ] GC(599) Eden regions: 5->0(23)
[0.959s][info   ][gc,heap        ] GC(599) Survivor regions: 3->2(4)
[0.959s][info   ][gc,heap        ] GC(599) Old regions: 318->321
[0.959s][info   ][gc,heap        ] GC(599) Archive regions: 2->2
[0.959s][info   ][gc,heap        ] GC(599) Humongous regions: 179->175
[0.959s][info   ][gc,metaspace   ] GC(599) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.959s][info   ][gc             ] GC(599) Pause Young (Normal) (G1 Preventive Collection) 482M->476M(512M) 0.965ms
[0.959s][info   ][gc,cpu         ] GC(599) User=0.00s Sys=0.00s Real=0.00s
[0.959s][info   ][gc,marking     ] GC(597) Concurrent Rebuild Remembered Sets 1.741ms
[0.960s][info   ][gc,start       ] GC(597) Pause Cleanup
[0.960s][info   ][gc             ] GC(597) Pause Cleanup 477M->477M(512M) 0.332ms
[0.960s][info   ][gc,cpu         ] GC(597) User=0.01s Sys=0.00s Real=0.00s
[0.960s][info   ][gc,marking     ] GC(597) Concurrent Cleanup for Next Mark
[0.960s][info   ][gc,marking     ] GC(597) Concurrent Cleanup for Next Mark 0.304ms
[0.960s][info   ][gc             ] GC(597) Concurrent Mark Cycle 4.944ms
[0.961s][info   ][gc,start       ] GC(600) Pause Young (Prepare Mixed) (G1 Preventive Collection)
[0.961s][info   ][gc,task        ] GC(600) Using 8 workers of 8 for evacuation
[0.961s][info   ][gc,phases      ] GC(600)   Pre Evacuate Collection Set: 0.1ms
[0.961s][info   ][gc,phases      ] GC(600)   Merge Heap Roots: 0.0ms
[0.961s][info   ][gc,phases      ] GC(600)   Evacuate Collection Set: 0.2ms
[0.961s][info   ][gc,phases      ] GC(600)   Post Evacuate Collection Set: 0.1ms
[0.961s][info   ][gc,phases      ] GC(600)   Other: 0.0ms
[0.961s][info   ][gc,heap        ] GC(600) Eden regions: 5->0(21)
[0.961s][info   ][gc,heap        ] GC(600) Survivor regions: 2->4(4)
[0.961s][info   ][gc,heap        ] GC(600) Old regions: 321->321
[0.961s][info   ][gc,heap        ] GC(600) Archive regions: 2->2
[0.961s][info   ][gc,heap        ] GC(600) Humongous regions: 177->174
[0.961s][info   ][gc,metaspace   ] GC(600) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.961s][info   ][gc             ] GC(600) Pause Young (Prepare Mixed) (G1 Preventive Collection) 483M->477M(512M) 0.630ms
[0.961s][info   ][gc,cpu         ] GC(600) User=0.00s Sys=0.00s Real=0.01s
[0.962s][info   ][gc,start       ] GC(601) Pause Young (Mixed) (G1 Preventive Collection)
[0.962s][info   ][gc,task        ] GC(601) Using 8 workers of 8 for evacuation
[0.962s][info   ][gc,phases      ] GC(601)   Pre Evacuate Collection Set: 0.1ms
[0.962s][info   ][gc,phases      ] GC(601)   Merge Heap Roots: 0.1ms
[0.962s][info   ][gc,phases      ] GC(601)   Evacuate Collection Set: 0.3ms
[0.962s][info   ][gc,phases      ] GC(601)   Post Evacuate Collection Set: 0.1ms
[0.962s][info   ][gc,phases      ] GC(601)   Other: 0.0ms
[0.962s][info   ][gc,heap        ] GC(601) Eden regions: 3->0(23)
[0.962s][info   ][gc,heap        ] GC(601) Survivor regions: 4->2(4)
[0.962s][info   ][gc,heap        ] GC(601) Old regions: 321->322
[0.962s][info   ][gc,heap        ] GC(601) Archive regions: 2->2
[0.962s][info   ][gc,heap        ] GC(601) Humongous regions: 175->175
[0.962s][info   ][gc,metaspace   ] GC(601) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.962s][info   ][gc             ] GC(601) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 480M->477M(512M) 0.719ms
[0.963s][info   ][gc,cpu         ] GC(601) User=0.00s Sys=0.00s Real=0.00s
[0.963s][info   ][gc,start       ] GC(602) Pause Young (Mixed) (G1 Preventive Collection)
[0.963s][info   ][gc,task        ] GC(602) Using 8 workers of 8 for evacuation
[0.964s][info   ][gc,phases      ] GC(602)   Pre Evacuate Collection Set: 0.1ms
[0.964s][info   ][gc,phases      ] GC(602)   Merge Heap Roots: 0.1ms
[0.964s][info   ][gc,phases      ] GC(602)   Evacuate Collection Set: 0.5ms
[0.964s][info   ][gc,phases      ] GC(602)   Post Evacuate Collection Set: 0.2ms
[0.964s][info   ][gc,phases      ] GC(602)   Other: 0.0ms
[0.964s][info   ][gc,heap        ] GC(602) Eden regions: 1->0(24)
[0.964s][info   ][gc,heap        ] GC(602) Survivor regions: 2->1(4)
[0.964s][info   ][gc,heap        ] GC(602) Old regions: 322->328
[0.964s][info   ][gc,heap        ] GC(602) Archive regions: 2->2
[0.964s][info   ][gc,heap        ] GC(602) Humongous regions: 175->175
[0.964s][info   ][gc,metaspace   ] GC(602) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.964s][info   ][gc             ] GC(602) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 478M->483M(512M) 0.998ms
[0.964s][info   ][gc,cpu         ] GC(602) User=0.00s Sys=0.00s Real=0.00s
[0.964s][info   ][gc,start       ] GC(603) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.964s][info   ][gc,task        ] GC(603) Using 8 workers of 8 for evacuation
[0.964s][info   ][gc,phases      ] GC(603)   Pre Evacuate Collection Set: 0.1ms
[0.964s][info   ][gc,phases      ] GC(603)   Merge Heap Roots: 0.0ms
[0.964s][info   ][gc,phases      ] GC(603)   Evacuate Collection Set: 0.2ms
[0.964s][info   ][gc,phases      ] GC(603)   Post Evacuate Collection Set: 0.1ms
[0.964s][info   ][gc,phases      ] GC(603)   Other: 0.1ms
[0.964s][info   ][gc,heap        ] GC(603) Eden regions: 1->0(23)
[0.964s][info   ][gc,heap        ] GC(603) Survivor regions: 1->2(4)
[0.964s][info   ][gc,heap        ] GC(603) Old regions: 328->328
[0.964s][info   ][gc,heap        ] GC(603) Archive regions: 2->2
[0.964s][info   ][gc,heap        ] GC(603) Humongous regions: 175->175
[0.964s][info   ][gc,metaspace   ] GC(603) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.964s][info   ][gc             ] GC(603) Pause Young (Concurrent Start) (G1 Humongous Allocation) 484M->483M(512M) 0.629ms
[0.964s][info   ][gc,cpu         ] GC(603) User=0.00s Sys=0.00s Real=0.00s
[0.964s][info   ][gc             ] GC(604) Concurrent Mark Cycle
[0.964s][info   ][gc,marking     ] GC(604) Concurrent Clear Claimed Marks
[0.964s][info   ][gc,marking     ] GC(604) Concurrent Clear Claimed Marks 0.008ms
[0.964s][info   ][gc,marking     ] GC(604) Concurrent Scan Root Regions
[0.965s][info   ][gc,marking     ] GC(604) Concurrent Scan Root Regions 0.044ms
[0.965s][info   ][gc,marking     ] GC(604) Concurrent Mark
[0.965s][info   ][gc,marking     ] GC(604) Concurrent Mark From Roots
[0.965s][info   ][gc,task        ] GC(604) Using 2 workers of 2 for marking
[0.965s][info   ][gc,start       ] GC(605) Pause Young (Normal) (G1 Preventive Collection)
[0.965s][info   ][gc,task        ] GC(605) Using 8 workers of 8 for evacuation
[0.965s][info   ][gc,phases      ] GC(605)   Pre Evacuate Collection Set: 0.1ms
[0.965s][info   ][gc,phases      ] GC(605)   Merge Heap Roots: 0.0ms
[0.965s][info   ][gc,phases      ] GC(605)   Evacuate Collection Set: 0.1ms
[0.965s][info   ][gc,phases      ] GC(605)   Post Evacuate Collection Set: 0.1ms
[0.965s][info   ][gc,phases      ] GC(605)   Other: 0.0ms
[0.965s][info   ][gc,heap        ] GC(605) Eden regions: 1->0(22)
[0.965s][info   ][gc,heap        ] GC(605) Survivor regions: 2->3(3)
[0.965s][info   ][gc,heap        ] GC(605) Old regions: 328->328
[0.965s][info   ][gc,heap        ] GC(605) Archive regions: 2->2
[0.965s][info   ][gc,heap        ] GC(605) Humongous regions: 176->174
[0.965s][info   ][gc,metaspace   ] GC(605) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.965s][info   ][gc             ] GC(605) Pause Young (Normal) (G1 Preventive Collection) 485M->483M(512M) 0.488ms
[0.965s][info   ][gc,cpu         ] GC(605) User=0.00s Sys=0.00s Real=0.00s
[0.966s][info   ][gc,start       ] GC(606) Pause Young (Normal) (G1 Preventive Collection)
[0.966s][info   ][gc,task        ] GC(606) Using 8 workers of 8 for evacuation
[0.966s][info   ][gc,phases      ] GC(606)   Pre Evacuate Collection Set: 0.1ms
[0.966s][info   ][gc,phases      ] GC(606)   Merge Heap Roots: 0.0ms
[0.966s][info   ][gc,phases      ] GC(606)   Evacuate Collection Set: 0.2ms
[0.966s][info   ][gc,phases      ] GC(606)   Post Evacuate Collection Set: 0.1ms
[0.966s][info   ][gc,phases      ] GC(606)   Other: 0.0ms
[0.966s][info   ][gc,heap        ] GC(606) Eden regions: 1->0(22)
[0.966s][info   ][gc,heap        ] GC(606) Survivor regions: 3->3(4)
[0.966s][info   ][gc,heap        ] GC(606) Old regions: 328->328
[0.966s][info   ][gc,heap        ] GC(606) Archive regions: 2->2
[0.966s][info   ][gc,heap        ] GC(606) Humongous regions: 174->174
[0.966s][info   ][gc,metaspace   ] GC(606) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.966s][info   ][gc             ] GC(606) Pause Young (Normal) (G1 Preventive Collection) 484M->483M(512M) 0.554ms
[0.966s][info   ][gc,cpu         ] GC(606) User=0.00s Sys=0.00s Real=0.00s
[0.966s][info   ][gc,start       ] GC(607) Pause Young (Normal) (G1 Preventive Collection)
[0.966s][info   ][gc,task        ] GC(607) Using 8 workers of 8 for evacuation
[0.967s][info   ][gc,phases      ] GC(607)   Pre Evacuate Collection Set: 0.0ms
[0.967s][info   ][gc,phases      ] GC(607)   Merge Heap Roots: 0.0ms
[0.967s][info   ][gc,phases      ] GC(607)   Evacuate Collection Set: 0.2ms
[0.967s][info   ][gc,phases      ] GC(607)   Post Evacuate Collection Set: 0.1ms
[0.967s][info   ][gc,phases      ] GC(607)   Other: 0.0ms
[0.967s][info   ][gc,heap        ] GC(607) Eden regions: 1->0(22)
[0.967s][info   ][gc,heap        ] GC(607) Survivor regions: 3->3(4)
[0.967s][info   ][gc,heap        ] GC(607) Old regions: 328->329
[0.967s][info   ][gc,heap        ] GC(607) Archive regions: 2->2
[0.967s][info   ][gc,heap        ] GC(607) Humongous regions: 174->174
[0.967s][info   ][gc,metaspace   ] GC(607) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.967s][info   ][gc             ] GC(607) Pause Young (Normal) (G1 Preventive Collection) 484M->484M(512M) 0.457ms
[0.967s][info   ][gc,cpu         ] GC(607) User=0.00s Sys=0.00s Real=0.00s
[0.967s][info   ][gc,start       ] GC(608) Pause Young (Normal) (G1 Preventive Collection)
[0.967s][info   ][gc,task        ] GC(608) Using 8 workers of 8 for evacuation
[0.967s][info   ][gc,phases      ] GC(608)   Pre Evacuate Collection Set: 0.1ms
[0.967s][info   ][gc,phases      ] GC(608)   Merge Heap Roots: 0.0ms
[0.967s][info   ][gc,phases      ] GC(608)   Evacuate Collection Set: 0.1ms
[0.967s][info   ][gc,phases      ] GC(608)   Post Evacuate Collection Set: 0.0ms
[0.967s][info   ][gc,phases      ] GC(608)   Other: 0.0ms
[0.967s][info   ][gc,heap        ] GC(608) Eden regions: 1->0(22)
[0.967s][info   ][gc,heap        ] GC(608) Survivor regions: 3->3(3)
[0.967s][info   ][gc,heap        ] GC(608) Old regions: 329->329
[0.967s][info   ][gc,heap        ] GC(608) Archive regions: 2->2
[0.967s][info   ][gc,heap        ] GC(608) Humongous regions: 174->174
[0.967s][info   ][gc,metaspace   ] GC(608) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.968s][info   ][gc             ] GC(608) Pause Young (Normal) (G1 Preventive Collection) 484M->484M(512M) 0.361ms
[0.968s][info   ][gc,cpu         ] GC(608) User=0.00s Sys=0.00s Real=0.00s
[0.968s][info   ][gc,start       ] GC(609) Pause Young (Normal) (G1 Preventive Collection)
[0.968s][info   ][gc,task        ] GC(609) Using 8 workers of 8 for evacuation
[0.968s][info   ][gc,phases      ] GC(609)   Pre Evacuate Collection Set: 0.0ms
[0.968s][info   ][gc,phases      ] GC(609)   Merge Heap Roots: 0.0ms
[0.968s][info   ][gc,phases      ] GC(609)   Evacuate Collection Set: 0.1ms
[0.968s][info   ][gc,phases      ] GC(609)   Post Evacuate Collection Set: 0.1ms
[0.968s][info   ][gc,phases      ] GC(609)   Other: 0.0ms
[0.968s][info   ][gc,heap        ] GC(609) Eden regions: 0->0(23)
[0.968s][info   ][gc,heap        ] GC(609) Survivor regions: 3->2(3)
[0.968s][info   ][gc,heap        ] GC(609) Old regions: 329->329
[0.968s][info   ][gc,heap        ] GC(609) Archive regions: 2->2
[0.968s][info   ][gc,heap        ] GC(609) Humongous regions: 175->174
[0.968s][info   ][gc,metaspace   ] GC(609) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.968s][info   ][gc             ] GC(609) Pause Young (Normal) (G1 Preventive Collection) 485M->483M(512M) 0.351ms
[0.968s][info   ][gc,cpu         ] GC(609) User=0.00s Sys=0.00s Real=0.00s
[0.968s][info   ][gc,marking     ] GC(604) Concurrent Mark From Roots 3.586ms
[0.968s][info   ][gc,marking     ] GC(604) Concurrent Preclean
[0.968s][info   ][gc,marking     ] GC(604) Concurrent Preclean 0.009ms
[0.968s][info   ][gc,start       ] GC(604) Pause Remark
[0.968s][info   ][gc             ] GC(604) Pause Remark 484M->483M(512M) 0.174ms
[0.968s][info   ][gc,cpu         ] GC(604) User=0.00s Sys=0.00s Real=0.00s
[0.968s][info   ][gc,marking     ] GC(604) Concurrent Mark 3.928ms
[0.968s][info   ][gc,marking     ] GC(604) Concurrent Rebuild Remembered Sets
[0.969s][info   ][gc,start       ] GC(610) Pause Young (Normal) (G1 Preventive Collection)
[0.969s][info   ][gc,task        ] GC(610) Using 8 workers of 8 for evacuation
[0.969s][info   ][gc,phases      ] GC(610)   Pre Evacuate Collection Set: 0.0ms
[0.969s][info   ][gc,phases      ] GC(610)   Merge Heap Roots: 0.1ms
[0.969s][info   ][gc,phases      ] GC(610)   Evacuate Collection Set: 0.1ms
[0.969s][info   ][gc,phases      ] GC(610)   Post Evacuate Collection Set: 0.0ms
[0.969s][info   ][gc,phases      ] GC(610)   Other: 0.0ms
[0.969s][info   ][gc,heap        ] GC(610) Eden regions: 2->0(22)
[0.969s][info   ][gc,heap        ] GC(610) Survivor regions: 2->3(3)
[0.969s][info   ][gc,heap        ] GC(610) Old regions: 328->328
[0.969s][info   ][gc,heap        ] GC(610) Archive regions: 2->2
[0.969s][info   ][gc,heap        ] GC(610) Humongous regions: 175->173
[0.969s][info   ][gc,metaspace   ] GC(610) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.969s][info   ][gc             ] GC(610) Pause Young (Normal) (G1 Preventive Collection) 485M->482M(512M) 0.428ms
[0.969s][info   ][gc,cpu         ] GC(610) User=0.00s Sys=0.00s Real=0.00s
[0.970s][info   ][gc,start       ] GC(611) Pause Young (Normal) (G1 Preventive Collection)
[0.970s][info   ][gc,task        ] GC(611) Using 8 workers of 8 for evacuation
[0.970s][info   ][gc,phases      ] GC(611)   Pre Evacuate Collection Set: 0.1ms
[0.970s][info   ][gc,phases      ] GC(611)   Merge Heap Roots: 0.0ms
[0.970s][info   ][gc,phases      ] GC(611)   Evacuate Collection Set: 0.2ms
[0.970s][info   ][gc,phases      ] GC(611)   Post Evacuate Collection Set: 0.1ms
[0.970s][info   ][gc,phases      ] GC(611)   Other: 0.0ms
[0.970s][info   ][gc,heap        ] GC(611) Eden regions: 1->0(22)
[0.970s][info   ][gc,heap        ] GC(611) Survivor regions: 3->3(4)
[0.970s][info   ][gc,heap        ] GC(611) Old regions: 328->329
[0.970s][info   ][gc,heap        ] GC(611) Archive regions: 2->2
[0.970s][info   ][gc,heap        ] GC(611) Humongous regions: 174->172
[0.970s][info   ][gc,metaspace   ] GC(611) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.970s][info   ][gc             ] GC(611) Pause Young (Normal) (G1 Preventive Collection) 484M->482M(512M) 0.494ms
[0.970s][info   ][gc,cpu         ] GC(611) User=0.00s Sys=0.00s Real=0.00s
[0.970s][info   ][gc,marking     ] GC(604) Concurrent Rebuild Remembered Sets 1.919ms
[0.970s][info   ][gc,start       ] GC(604) Pause Cleanup
[0.971s][info   ][gc             ] GC(604) Pause Cleanup 483M->483M(512M) 0.179ms
[0.971s][info   ][gc,cpu         ] GC(604) User=0.00s Sys=0.00s Real=0.00s
[0.971s][info   ][gc,marking     ] GC(604) Concurrent Cleanup for Next Mark
[0.971s][info   ][gc,start       ] GC(612) Pause Young (Prepare Mixed) (G1 Preventive Collection)
[0.971s][info   ][gc,task        ] GC(612) Using 8 workers of 8 for evacuation
[0.971s][info   ][gc,phases      ] GC(612)   Pre Evacuate Collection Set: 0.1ms
[0.971s][info   ][gc,phases      ] GC(612)   Merge Heap Roots: 0.0ms
[0.971s][info   ][gc,phases      ] GC(612)   Evacuate Collection Set: 0.1ms
[0.971s][info   ][gc,phases      ] GC(612)   Post Evacuate Collection Set: 0.1ms
[0.971s][info   ][gc,phases      ] GC(612)   Other: 0.0ms
[0.971s][info   ][gc,heap        ] GC(612) Eden regions: 2->0(22)
[0.971s][info   ][gc,heap        ] GC(612) Survivor regions: 3->3(4)
[0.971s][info   ][gc,heap        ] GC(612) Old regions: 329->330
[0.971s][info   ][gc,heap        ] GC(612) Archive regions: 2->2
[0.971s][info   ][gc,heap        ] GC(612) Humongous regions: 172->172
[0.971s][info   ][gc,metaspace   ] GC(612) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.971s][info   ][gc             ] GC(612) Pause Young (Prepare Mixed) (G1 Preventive Collection) 483M->483M(512M) 0.414ms
[0.971s][info   ][gc,cpu         ] GC(612) User=0.00s Sys=0.00s Real=0.01s
[0.971s][info   ][gc,start       ] GC(613) Pause Young (Mixed) (G1 Preventive Collection)
[0.971s][info   ][gc,task        ] GC(613) Using 8 workers of 8 for evacuation
[0.972s][info   ][gc,phases      ] GC(613)   Pre Evacuate Collection Set: 0.1ms
[0.972s][info   ][gc,phases      ] GC(613)   Merge Heap Roots: 0.1ms
[0.972s][info   ][gc,phases      ] GC(613)   Evacuate Collection Set: 0.2ms
[0.972s][info   ][gc,phases      ] GC(613)   Post Evacuate Collection Set: 0.2ms
[0.972s][info   ][gc,phases      ] GC(613)   Other: 0.0ms
[0.972s][info   ][gc,heap        ] GC(613) Eden regions: 0->0(24)
[0.972s][info   ][gc,heap        ] GC(613) Survivor regions: 3->1(4)
[0.972s][info   ][gc,heap        ] GC(613) Old regions: 330->319
[0.972s][info   ][gc,heap        ] GC(613) Archive regions: 2->2
[0.972s][info   ][gc,heap        ] GC(613) Humongous regions: 173->173
[0.972s][info   ][gc,metaspace   ] GC(613) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.972s][info   ][gc             ] GC(613) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 484M->473M(512M) 0.594ms
[0.972s][info   ][gc,cpu         ] GC(613) User=0.00s Sys=0.00s Real=0.00s
[0.972s][info   ][gc,marking     ] GC(604) Concurrent Cleanup for Next Mark 1.462ms
[0.972s][info   ][gc             ] GC(604) Concurrent Mark Cycle 7.720ms
[0.972s][info   ][gc,start       ] GC(614) Pause Young (Mixed) (G1 Preventive Collection)
[0.972s][info   ][gc,task        ] GC(614) Using 8 workers of 8 for evacuation
[0.973s][info   ][gc,phases      ] GC(614)   Pre Evacuate Collection Set: 0.1ms
[0.973s][info   ][gc,phases      ] GC(614)   Merge Heap Roots: 0.1ms
[0.973s][info   ][gc,phases      ] GC(614)   Evacuate Collection Set: 0.3ms
[0.973s][info   ][gc,phases      ] GC(614)   Post Evacuate Collection Set: 0.1ms
[0.973s][info   ][gc,phases      ] GC(614)   Other: 0.0ms
[0.973s][info   ][gc,heap        ] GC(614) Eden regions: 5->0(23)
[0.973s][info   ][gc,heap        ] GC(614) Survivor regions: 1->2(4)
[0.973s][info   ][gc,heap        ] GC(614) Old regions: 319->330
[0.973s][info   ][gc,heap        ] GC(614) Archive regions: 2->2
[0.973s][info   ][gc,heap        ] GC(614) Humongous regions: 175->174
[0.973s][info   ][gc,metaspace   ] GC(614) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.973s][info   ][gc             ] GC(614) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 480M->486M(512M) 0.763ms
[0.973s][info   ][gc,cpu         ] GC(614) User=0.00s Sys=0.01s Real=0.00s
[0.973s][info   ][gc,start       ] GC(615) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.973s][info   ][gc,task        ] GC(615) Using 8 workers of 8 for evacuation
[0.974s][info   ][gc,phases      ] GC(615)   Pre Evacuate Collection Set: 0.2ms
[0.974s][info   ][gc,phases      ] GC(615)   Merge Heap Roots: 0.0ms
[0.974s][info   ][gc,phases      ] GC(615)   Evacuate Collection Set: 0.1ms
[0.974s][info   ][gc,phases      ] GC(615)   Post Evacuate Collection Set: 0.1ms
[0.974s][info   ][gc,phases      ] GC(615)   Other: 0.0ms
[0.974s][info   ][gc,heap        ] GC(615) Eden regions: 1->0(23)
[0.974s][info   ][gc,heap        ] GC(615) Survivor regions: 2->2(3)
[0.974s][info   ][gc,heap        ] GC(615) Old regions: 330->330
[0.974s][info   ][gc,heap        ] GC(615) Archive regions: 2->2
[0.974s][info   ][gc,heap        ] GC(615) Humongous regions: 174->174
[0.974s][info   ][gc,metaspace   ] GC(615) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.974s][info   ][gc             ] GC(615) Pause Young (Concurrent Start) (G1 Humongous Allocation) 486M->486M(512M) 0.575ms
[0.974s][info   ][gc,cpu         ] GC(615) User=0.00s Sys=0.00s Real=0.00s
[0.974s][info   ][gc             ] GC(616) Concurrent Mark Cycle
[0.974s][info   ][gc,marking     ] GC(616) Concurrent Clear Claimed Marks
[0.974s][info   ][gc,marking     ] GC(616) Concurrent Clear Claimed Marks 0.006ms
[0.974s][info   ][gc,marking     ] GC(616) Concurrent Scan Root Regions
[0.974s][info   ][gc,marking     ] GC(616) Concurrent Scan Root Regions 0.054ms
[0.974s][info   ][gc,marking     ] GC(616) Concurrent Mark
[0.974s][info   ][gc,marking     ] GC(616) Concurrent Mark From Roots
[0.974s][info   ][gc,task        ] GC(616) Using 2 workers of 2 for marking
[0.974s][info   ][gc,start       ] GC(617) Pause Young (Normal) (G1 Preventive Collection)
[0.974s][info   ][gc,task        ] GC(617) Using 8 workers of 8 for evacuation
[0.974s][info   ][gc,phases      ] GC(617)   Pre Evacuate Collection Set: 0.1ms
[0.974s][info   ][gc,phases      ] GC(617)   Merge Heap Roots: 0.0ms
[0.974s][info   ][gc,phases      ] GC(617)   Evacuate Collection Set: 0.1ms
[0.974s][info   ][gc,phases      ] GC(617)   Post Evacuate Collection Set: 0.1ms
[0.974s][info   ][gc,phases      ] GC(617)   Other: 0.0ms
[0.974s][info   ][gc,heap        ] GC(617) Eden regions: 0->0(22)
[0.974s][info   ][gc,heap        ] GC(617) Survivor regions: 2->3(3)
[0.974s][info   ][gc,heap        ] GC(617) Old regions: 330->330
[0.974s][info   ][gc,heap        ] GC(617) Archive regions: 2->2
[0.974s][info   ][gc,heap        ] GC(617) Humongous regions: 175->174
[0.974s][info   ][gc,metaspace   ] GC(617) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.974s][info   ][gc             ] GC(617) Pause Young (Normal) (G1 Preventive Collection) 487M->486M(512M) 0.391ms
[0.974s][info   ][gc,cpu         ] GC(617) User=0.00s Sys=0.00s Real=0.00s
[0.974s][info   ][gc,start       ] GC(618) Pause Young (Normal) (G1 Preventive Collection)
[0.974s][info   ][gc,task        ] GC(618) Using 8 workers of 8 for evacuation
[0.975s][info   ][gc,phases      ] GC(618)   Pre Evacuate Collection Set: 0.1ms
[0.975s][info   ][gc,phases      ] GC(618)   Merge Heap Roots: 0.0ms
[0.975s][info   ][gc,phases      ] GC(618)   Evacuate Collection Set: 0.1ms
[0.975s][info   ][gc,phases      ] GC(618)   Post Evacuate Collection Set: 0.1ms
[0.975s][info   ][gc,phases      ] GC(618)   Other: 0.0ms
[0.975s][info   ][gc,heap        ] GC(618) Eden regions: 1->0(23)
[0.975s][info   ][gc,heap        ] GC(618) Survivor regions: 3->2(2)
[0.975s][info   ][gc,heap        ] GC(618) Old regions: 330->332
[0.975s][info   ][gc,heap        ] GC(618) Archive regions: 2->2
[0.975s][info   ][gc,heap        ] GC(618) Humongous regions: 174->174
[0.975s][info   ][gc,metaspace   ] GC(618) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.975s][info   ][gc             ] GC(618) Pause Young (Normal) (G1 Preventive Collection) (Evacuation Failure) 487M->488M(512M) 0.448ms
[0.975s][info   ][gc,cpu         ] GC(618) User=0.01s Sys=0.00s Real=0.00s
[0.975s][info   ][gc,start       ] GC(619) Pause Young (Normal) (G1 Preventive Collection)
[0.975s][info   ][gc,task        ] GC(619) Using 8 workers of 8 for evacuation
[0.975s][info   ][gc,phases      ] GC(619)   Pre Evacuate Collection Set: 0.0ms
[0.975s][info   ][gc,phases      ] GC(619)   Merge Heap Roots: 0.0ms
[0.975s][info   ][gc,phases      ] GC(619)   Evacuate Collection Set: 0.1ms
[0.975s][info   ][gc,phases      ] GC(619)   Post Evacuate Collection Set: 0.1ms
[0.975s][info   ][gc,phases      ] GC(619)   Other: 0.0ms
[0.975s][info   ][gc,heap        ] GC(619) Eden regions: 1->0(24)
[0.975s][info   ][gc,heap        ] GC(619) Survivor regions: 2->1(1)
[0.975s][info   ][gc,heap        ] GC(619) Old regions: 332->335
[0.975s][info   ][gc,heap        ] GC(619) Archive regions: 2->2
[0.975s][info   ][gc,heap        ] GC(619) Humongous regions: 174->174
[0.975s][info   ][gc,metaspace   ] GC(619) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.975s][info   ][gc             ] GC(619) Pause Young (Normal) (G1 Preventive Collection) (Evacuation Failure) 488M->489M(512M) 0.457ms
[0.976s][info   ][gc,cpu         ] GC(619) User=0.00s Sys=0.00s Real=0.00s
[0.976s][info   ][gc,ergo        ] Attempting full compaction
[0.976s][info   ][gc,start       ] GC(620) Pause Full (G1 Compaction Pause)
[0.976s][info   ][gc,task        ] GC(620) Using 8 workers of 8 for full compaction
[0.976s][info   ][gc,phases,start] GC(620) Phase 1: Mark live objects
[0.976s][info   ][gc,phases      ] GC(620) Phase 1: Mark live objects 0.537ms
[0.976s][info   ][gc,phases,start] GC(620) Phase 2: Prepare compaction
[0.976s][info   ][gc,phases      ] GC(620) Phase 2: Prepare compaction 0.255ms
[0.977s][info   ][gc,phases,start] GC(620) Phase 3: Adjust pointers
[0.977s][info   ][gc,phases      ] GC(620) Phase 3: Adjust pointers 0.276ms
[0.977s][info   ][gc,phases,start] GC(620) Phase 4: Compact heap
[0.980s][info   ][gc,phases      ] GC(620) Phase 4: Compact heap 2.877ms
[0.980s][info   ][gc,heap        ] GC(620) Eden regions: 0->0(25)
[0.980s][info   ][gc,heap        ] GC(620) Survivor regions: 1->0(1)
[0.980s][info   ][gc,heap        ] GC(620) Old regions: 335->260
[0.980s][info   ][gc,heap        ] GC(620) Archive regions: 2->2
[0.980s][info   ][gc,heap        ] GC(620) Humongous regions: 174->174
[0.980s][info   ][gc,metaspace   ] GC(620) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.980s][info   ][gc             ] GC(620) Pause Full (G1 Compaction Pause) 489M->404M(512M) 4.692ms
[0.980s][info   ][gc,cpu         ] GC(620) User=0.01s Sys=0.00s Real=0.00s
[0.980s][info   ][gc,marking     ] GC(616) Concurrent Mark From Roots 6.517ms
[0.980s][info   ][gc,marking     ] GC(616) Concurrent Mark Abort
[0.980s][info   ][gc             ] GC(616) Concurrent Mark Cycle 6.632ms
[0.981s][info   ][gc,start       ] GC(621) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.981s][info   ][gc,task        ] GC(621) Using 8 workers of 8 for evacuation
[0.981s][info   ][gc,phases      ] GC(621)   Pre Evacuate Collection Set: 0.2ms
[0.981s][info   ][gc,phases      ] GC(621)   Merge Heap Roots: 0.0ms
[0.981s][info   ][gc,phases      ] GC(621)   Evacuate Collection Set: 0.1ms
[0.981s][info   ][gc,phases      ] GC(621)   Post Evacuate Collection Set: 0.1ms
[0.981s][info   ][gc,phases      ] GC(621)   Other: 0.0ms
[0.981s][info   ][gc,heap        ] GC(621) Eden regions: 2->0(24)
[0.981s][info   ][gc,heap        ] GC(621) Survivor regions: 0->1(4)
[0.981s][info   ][gc,heap        ] GC(621) Old regions: 260->260
[0.981s][info   ][gc,heap        ] GC(621) Archive regions: 2->2
[0.981s][info   ][gc,heap        ] GC(621) Humongous regions: 175->175
[0.981s][info   ][gc,metaspace   ] GC(621) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.981s][info   ][gc             ] GC(621) Pause Young (Concurrent Start) (G1 Humongous Allocation) 407M->406M(512M) 0.660ms
[0.981s][info   ][gc,cpu         ] GC(621) User=0.01s Sys=0.00s Real=0.01s
[0.981s][info   ][gc             ] GC(622) Concurrent Mark Cycle
[0.981s][info   ][gc,marking     ] GC(622) Concurrent Clear Claimed Marks
[0.981s][info   ][gc,marking     ] GC(622) Concurrent Clear Claimed Marks 0.007ms
[0.981s][info   ][gc,marking     ] GC(622) Concurrent Scan Root Regions
[0.981s][info   ][gc,marking     ] GC(622) Concurrent Scan Root Regions 0.040ms
[0.981s][info   ][gc,marking     ] GC(622) Concurrent Mark
[0.981s][info   ][gc,marking     ] GC(622) Concurrent Mark From Roots
[0.981s][info   ][gc,task        ] GC(622) Using 2 workers of 2 for marking
[0.982s][info   ][gc,marking     ] GC(622) Concurrent Mark From Roots 0.719ms
[0.982s][info   ][gc,marking     ] GC(622) Concurrent Preclean
[0.982s][info   ][gc,marking     ] GC(622) Concurrent Preclean 0.008ms
[0.982s][info   ][gc,start       ] GC(622) Pause Remark
[0.982s][info   ][gc             ] GC(622) Pause Remark 422M->421M(512M) 0.273ms
[0.982s][info   ][gc,cpu         ] GC(622) User=0.00s Sys=0.00s Real=0.00s
[0.983s][info   ][gc,marking     ] GC(622) Concurrent Mark 1.169ms
[0.983s][info   ][gc,marking     ] GC(622) Concurrent Rebuild Remembered Sets
[0.983s][info   ][gc,marking     ] GC(622) Concurrent Rebuild Remembered Sets 0.448ms
[0.983s][info   ][gc,start       ] GC(622) Pause Cleanup
[0.983s][info   ][gc             ] GC(622) Pause Cleanup 440M->440M(512M) 0.159ms
[0.983s][info   ][gc,cpu         ] GC(622) User=0.00s Sys=0.00s Real=0.00s
[0.983s][info   ][gc,marking     ] GC(622) Concurrent Cleanup for Next Mark
[0.984s][info   ][gc,marking     ] GC(622) Concurrent Cleanup for Next Mark 0.283ms
[0.984s][info   ][gc             ] GC(622) Concurrent Mark Cycle 2.361ms
[0.984s][info   ][gc,start       ] GC(623) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.984s][info   ][gc,task        ] GC(623) Using 8 workers of 8 for evacuation
[0.984s][info   ][gc,phases      ] GC(623)   Pre Evacuate Collection Set: 0.1ms
[0.984s][info   ][gc,phases      ] GC(623)   Merge Heap Roots: 0.1ms
[0.985s][info   ][gc,phases      ] GC(623)   Evacuate Collection Set: 0.3ms
[0.985s][info   ][gc,phases      ] GC(623)   Post Evacuate Collection Set: 0.1ms
[0.985s][info   ][gc,phases      ] GC(623)   Other: 0.0ms
[0.985s][info   ][gc,heap        ] GC(623) Eden regions: 24->0(21)
[0.985s][info   ][gc,heap        ] GC(623) Survivor regions: 1->4(4)
[0.985s][info   ][gc,heap        ] GC(623) Old regions: 260->271
[0.985s][info   ][gc,heap        ] GC(623) Archive regions: 2->2
[0.985s][info   ][gc,heap        ] GC(623) Humongous regions: 194->177
[0.985s][info   ][gc,metaspace   ] GC(623) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.985s][info   ][gc             ] GC(623) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 449M->421M(512M) 0.845ms
[0.985s][info   ][gc,cpu         ] GC(623) User=0.00s Sys=0.00s Real=0.00s
[0.986s][info   ][gc,start       ] GC(624) Pause Young (Mixed) (G1 Evacuation Pause)
[0.986s][info   ][gc,task        ] GC(624) Using 8 workers of 8 for evacuation
[0.987s][info   ][gc,phases      ] GC(624)   Pre Evacuate Collection Set: 0.1ms
[0.987s][info   ][gc,phases      ] GC(624)   Merge Heap Roots: 0.1ms
[0.987s][info   ][gc,phases      ] GC(624)   Evacuate Collection Set: 0.5ms
[0.987s][info   ][gc,phases      ] GC(624)   Post Evacuate Collection Set: 0.1ms
[0.987s][info   ][gc,phases      ] GC(624)   Other: 0.0ms
[0.987s][info   ][gc,heap        ] GC(624) Eden regions: 21->0(21)
[0.987s][info   ][gc,heap        ] GC(624) Survivor regions: 4->4(4)
[0.987s][info   ][gc,heap        ] GC(624) Old regions: 271->278
[0.987s][info   ][gc,heap        ] GC(624) Archive regions: 2->2
[0.987s][info   ][gc,heap        ] GC(624) Humongous regions: 185->172
[0.987s][info   ][gc,metaspace   ] GC(624) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.987s][info   ][gc             ] GC(624) Pause Young (Mixed) (G1 Evacuation Pause) 450M->429M(512M) 0.957ms
[0.987s][info   ][gc,cpu         ] GC(624) User=0.01s Sys=0.00s Real=0.00s
[0.987s][info   ][gc,start       ] GC(625) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.987s][info   ][gc,task        ] GC(625) Using 8 workers of 8 for evacuation
[0.988s][info   ][gc,phases      ] GC(625)   Pre Evacuate Collection Set: 0.1ms
[0.988s][info   ][gc,phases      ] GC(625)   Merge Heap Roots: 0.1ms
[0.988s][info   ][gc,phases      ] GC(625)   Evacuate Collection Set: 0.2ms
[0.988s][info   ][gc,phases      ] GC(625)   Post Evacuate Collection Set: 0.2ms
[0.988s][info   ][gc,phases      ] GC(625)   Other: 0.0ms
[0.988s][info   ][gc,heap        ] GC(625) Eden regions: 4->0(22)
[0.988s][info   ][gc,heap        ] GC(625) Survivor regions: 4->3(4)
[0.988s][info   ][gc,heap        ] GC(625) Old regions: 278->282
[0.988s][info   ][gc,heap        ] GC(625) Archive regions: 2->2
[0.988s][info   ][gc,heap        ] GC(625) Humongous regions: 172->172
[0.988s][info   ][gc,metaspace   ] GC(625) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.988s][info   ][gc             ] GC(625) Pause Young (Concurrent Start) (G1 Humongous Allocation) 433M->431M(512M) 0.757ms
[0.988s][info   ][gc,cpu         ] GC(625) User=0.00s Sys=0.00s Real=0.00s
[0.988s][info   ][gc             ] GC(626) Concurrent Mark Cycle
[0.988s][info   ][gc,marking     ] GC(626) Concurrent Clear Claimed Marks
[0.988s][info   ][gc,marking     ] GC(626) Concurrent Clear Claimed Marks 0.007ms
[0.988s][info   ][gc,marking     ] GC(626) Concurrent Scan Root Regions
[0.988s][info   ][gc,marking     ] GC(626) Concurrent Scan Root Regions 0.062ms
[0.988s][info   ][gc,marking     ] GC(626) Concurrent Mark
[0.988s][info   ][gc,marking     ] GC(626) Concurrent Mark From Roots
[0.988s][info   ][gc,task        ] GC(626) Using 2 workers of 2 for marking
[0.989s][info   ][gc,marking     ] GC(626) Concurrent Mark From Roots 0.805ms
[0.989s][info   ][gc,marking     ] GC(626) Concurrent Preclean
[0.989s][info   ][gc,marking     ] GC(626) Concurrent Preclean 0.008ms
[0.989s][info   ][gc,start       ] GC(626) Pause Remark
[0.990s][info   ][gc             ] GC(626) Pause Remark 450M->450M(512M) 0.283ms
[0.990s][info   ][gc,cpu         ] GC(626) User=0.00s Sys=0.00s Real=0.00s
[0.990s][info   ][gc,marking     ] GC(626) Concurrent Mark 1.321ms
[0.990s][info   ][gc,marking     ] GC(626) Concurrent Rebuild Remembered Sets
[0.990s][info   ][gc,marking     ] GC(626) Concurrent Rebuild Remembered Sets 0.453ms
[0.990s][info   ][gc,start       ] GC(626) Pause Cleanup
[0.990s][info   ][gc             ] GC(626) Pause Cleanup 457M->457M(512M) 0.140ms
[0.990s][info   ][gc,cpu         ] GC(626) User=0.00s Sys=0.00s Real=0.00s
[0.990s][info   ][gc,marking     ] GC(626) Concurrent Cleanup for Next Mark
[0.991s][info   ][gc,marking     ] GC(626) Concurrent Cleanup for Next Mark 0.270ms
[0.991s][info   ][gc             ] GC(626) Concurrent Mark Cycle 2.515ms
[0.991s][info   ][gc,start       ] GC(627) Pause Young (Prepare Mixed) (G1 Preventive Collection)
[0.991s][info   ][gc,task        ] GC(627) Using 8 workers of 8 for evacuation
[0.992s][info   ][gc,phases      ] GC(627)   Pre Evacuate Collection Set: 0.1ms
[0.992s][info   ][gc,phases      ] GC(627)   Merge Heap Roots: 0.1ms
[0.992s][info   ][gc,phases      ] GC(627)   Evacuate Collection Set: 0.4ms
[0.992s][info   ][gc,phases      ] GC(627)   Post Evacuate Collection Set: 0.2ms
[0.992s][info   ][gc,phases      ] GC(627)   Other: 0.0ms
[0.992s][info   ][gc,heap        ] GC(627) Eden regions: 21->0(21)
[0.992s][info   ][gc,heap        ] GC(627) Survivor regions: 3->4(4)
[0.992s][info   ][gc,heap        ] GC(627) Old regions: 282->290
[0.992s][info   ][gc,heap        ] GC(627) Archive regions: 2->2
[0.992s][info   ][gc,heap        ] GC(627) Humongous regions: 187->173
[0.992s][info   ][gc,metaspace   ] GC(627) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.992s][info   ][gc             ] GC(627) Pause Young (Prepare Mixed) (G1 Preventive Collection) 467M->442M(512M) 0.887ms
[0.992s][info   ][gc,cpu         ] GC(627) User=0.00s Sys=0.00s Real=0.01s
[0.993s][info   ][gc,start       ] GC(628) Pause Young (Mixed) (G1 Preventive Collection)
[0.993s][info   ][gc,task        ] GC(628) Using 8 workers of 8 for evacuation
[0.994s][info   ][gc,phases      ] GC(628)   Pre Evacuate Collection Set: 0.1ms
[0.994s][info   ][gc,phases      ] GC(628)   Merge Heap Roots: 0.1ms
[0.994s][info   ][gc,phases      ] GC(628)   Evacuate Collection Set: 0.6ms
[0.994s][info   ][gc,phases      ] GC(628)   Post Evacuate Collection Set: 0.3ms
[0.994s][info   ][gc,phases      ] GC(628)   Other: 0.0ms
[0.994s][info   ][gc,heap        ] GC(628) Eden regions: 16->0(21)
[0.994s][info   ][gc,heap        ] GC(628) Survivor regions: 4->4(4)
[0.994s][info   ][gc,heap        ] GC(628) Old regions: 290->299
[0.994s][info   ][gc,heap        ] GC(628) Archive regions: 2->2
[0.994s][info   ][gc,heap        ] GC(628) Humongous regions: 182->169
[0.994s][info   ][gc,metaspace   ] GC(628) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.994s][info   ][gc             ] GC(628) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 466M->449M(512M) 1.372ms
[0.994s][info   ][gc,cpu         ] GC(628) User=0.00s Sys=0.00s Real=0.00s
[0.995s][info   ][gc,start       ] GC(629) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.995s][info   ][gc,task        ] GC(629) Using 8 workers of 8 for evacuation
[0.995s][info   ][gc,phases      ] GC(629)   Pre Evacuate Collection Set: 0.1ms
[0.995s][info   ][gc,phases      ] GC(629)   Merge Heap Roots: 0.0ms
[0.995s][info   ][gc,phases      ] GC(629)   Evacuate Collection Set: 0.2ms
[0.995s][info   ][gc,phases      ] GC(629)   Post Evacuate Collection Set: 0.1ms
[0.995s][info   ][gc,phases      ] GC(629)   Other: 0.1ms
[0.995s][info   ][gc,heap        ] GC(629) Eden regions: 2->0(24)
[0.995s][info   ][gc,heap        ] GC(629) Survivor regions: 4->1(4)
[0.995s][info   ][gc,heap        ] GC(629) Old regions: 299->304
[0.995s][info   ][gc,heap        ] GC(629) Archive regions: 2->2
[0.995s][info   ][gc,heap        ] GC(629) Humongous regions: 170->169
[0.995s][info   ][gc,metaspace   ] GC(629) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.995s][info   ][gc             ] GC(629) Pause Young (Concurrent Start) (G1 Humongous Allocation) 451M->450M(512M) 0.767ms
[0.995s][info   ][gc,cpu         ] GC(629) User=0.00s Sys=0.00s Real=0.00s
[0.995s][info   ][gc             ] GC(630) Concurrent Mark Cycle
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Clear Claimed Marks
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Clear Claimed Marks 0.020ms
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Scan Root Regions
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Scan Root Regions 0.046ms
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Mark
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Mark From Roots
[0.996s][info   ][gc,task        ] GC(630) Using 2 workers of 2 for marking
[0.996s][info   ][gc,marking     ] GC(630) Concurrent Mark From Roots 0.878ms
[0.997s][info   ][gc,marking     ] GC(630) Concurrent Preclean
[0.997s][info   ][gc,marking     ] GC(630) Concurrent Preclean 0.011ms
[0.997s][info   ][gc,start       ] GC(630) Pause Remark
[0.997s][info   ][gc             ] GC(630) Pause Remark 471M->471M(512M) 0.266ms
[0.997s][info   ][gc,cpu         ] GC(630) User=0.00s Sys=0.00s Real=0.00s
[0.997s][info   ][gc,marking     ] GC(630) Concurrent Mark 1.368ms
[0.997s][info   ][gc,marking     ] GC(630) Concurrent Rebuild Remembered Sets
[0.997s][info   ][gc,start       ] GC(631) Pause Young (Normal) (G1 Preventive Collection)
[0.997s][info   ][gc,task        ] GC(631) Using 8 workers of 8 for evacuation
[0.998s][info   ][gc,phases      ] GC(631)   Pre Evacuate Collection Set: 0.1ms
[0.998s][info   ][gc,phases      ] GC(631)   Merge Heap Roots: 0.1ms
[0.998s][info   ][gc,phases      ] GC(631)   Evacuate Collection Set: 0.2ms
[0.998s][info   ][gc,phases      ] GC(631)   Post Evacuate Collection Set: 0.1ms
[0.998s][info   ][gc,phases      ] GC(631)   Other: 0.0ms
[0.998s][info   ][gc,heap        ] GC(631) Eden regions: 15->0(21)
[0.998s][info   ][gc,heap        ] GC(631) Survivor regions: 1->4(4)
[0.998s][info   ][gc,heap        ] GC(631) Old regions: 304->308
[0.998s][info   ][gc,heap        ] GC(631) Archive regions: 2->2
[0.998s][info   ][gc,heap        ] GC(631) Humongous regions: 177->171
[0.998s][info   ][gc,metaspace   ] GC(631) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.998s][info   ][gc             ] GC(631) Pause Young (Normal) (G1 Preventive Collection) 473M->459M(512M) 0.590ms
[0.998s][info   ][gc,cpu         ] GC(631) User=0.00s Sys=0.00s Real=0.00s
[0.998s][info   ][gc,marking     ] GC(630) Concurrent Rebuild Remembered Sets 1.352ms
[0.998s][info   ][gc,start       ] GC(630) Pause Cleanup
[0.999s][info   ][gc             ] GC(630) Pause Cleanup 469M->469M(512M) 0.192ms
[0.999s][info   ][gc,cpu         ] GC(630) User=0.00s Sys=0.00s Real=0.00s
[0.999s][info   ][gc,marking     ] GC(630) Concurrent Cleanup for Next Mark
[0.999s][info   ][gc,marking     ] GC(630) Concurrent Cleanup for Next Mark 0.278ms
[0.999s][info   ][gc             ] GC(630) Concurrent Mark Cycle 3.542ms
[0.999s][info   ][gc,start       ] GC(632) Pause Young (Prepare Mixed) (G1 Preventive Collection)
[0.999s][info   ][gc,task        ] GC(632) Using 8 workers of 8 for evacuation
[1.000s][info   ][gc,phases      ] GC(632)   Pre Evacuate Collection Set: 0.1ms
[1.000s][info   ][gc,phases      ] GC(632)   Merge Heap Roots: 0.0ms
[1.000s][info   ][gc,phases      ] GC(632)   Evacuate Collection Set: 0.2ms
[1.000s][info   ][gc,phases      ] GC(632)   Post Evacuate Collection Set: 0.1ms
[1.000s][info   ][gc,phases      ] GC(632)   Other: 0.0ms
[1.000s][info   ][gc,heap        ] GC(632) Eden regions: 11->0(21)
[1.000s][info   ][gc,heap        ] GC(632) Survivor regions: 4->4(4)
[1.000s][info   ][gc,heap        ] GC(632) Old regions: 308->312
[1.000s][info   ][gc,heap        ] GC(632) Archive regions: 2->2
[1.000s][info   ][gc,heap        ] GC(632) Humongous regions: 175->168
[1.000s][info   ][gc,metaspace   ] GC(632) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.000s][info   ][gc             ] GC(632) Pause Young (Prepare Mixed) (G1 Preventive Collection) 474M->461M(512M) 0.683ms
[1.000s][info   ][gc,cpu         ] GC(632) User=0.00s Sys=0.00s Real=0.00s
[1.001s][info   ][gc,start       ] GC(633) Pause Young (Mixed) (G1 Preventive Collection)
[1.001s][info   ][gc,task        ] GC(633) Using 8 workers of 8 for evacuation
[1.001s][info   ][gc,phases      ] GC(633)   Pre Evacuate Collection Set: 0.1ms
[1.001s][info   ][gc,phases      ] GC(633)   Merge Heap Roots: 0.1ms
[1.001s][info   ][gc,phases      ] GC(633)   Evacuate Collection Set: 0.3ms
[1.001s][info   ][gc,phases      ] GC(633)   Post Evacuate Collection Set: 0.1ms
[1.001s][info   ][gc,phases      ] GC(633)   Other: 0.0ms
[1.001s][info   ][gc,heap        ] GC(633) Eden regions: 10->0(23)
[1.001s][info   ][gc,heap        ] GC(633) Survivor regions: 4->2(4)
[1.001s][info   ][gc,heap        ] GC(633) Old regions: 312->319
[1.001s][info   ][gc,heap        ] GC(633) Archive regions: 2->2
[1.001s][info   ][gc,heap        ] GC(633) Humongous regions: 172->170
[1.001s][info   ][gc,metaspace   ] GC(633) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.001s][info   ][gc             ] GC(633) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 475M->469M(512M) 0.874ms
[1.001s][info   ][gc,cpu         ] GC(633) User=0.00s Sys=0.00s Real=0.01s
[1.002s][info   ][gc,start       ] GC(634) Pause Young (Mixed) (G1 Preventive Collection)
[1.002s][info   ][gc,task        ] GC(634) Using 8 workers of 8 for evacuation
[1.003s][info   ][gc,phases      ] GC(634)   Pre Evacuate Collection Set: 0.1ms
[1.003s][info   ][gc,phases      ] GC(634)   Merge Heap Roots: 0.1ms
[1.003s][info   ][gc,phases      ] GC(634)   Evacuate Collection Set: 0.3ms
[1.003s][info   ][gc,phases      ] GC(634)   Post Evacuate Collection Set: 0.1ms
[1.003s][info   ][gc,phases      ] GC(634)   Other: 0.0ms
[1.003s][info   ][gc,heap        ] GC(634) Eden regions: 4->0(21)
[1.003s][info   ][gc,heap        ] GC(634) Survivor regions: 2->4(4)
[1.003s][info   ][gc,heap        ] GC(634) Old regions: 319->319
[1.003s][info   ][gc,heap        ] GC(634) Archive regions: 2->2
[1.003s][info   ][gc,heap        ] GC(634) Humongous regions: 173->168
[1.003s][info   ][gc,metaspace   ] GC(634) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.003s][info   ][gc             ] GC(634) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 476M->469M(512M) 0.781ms
[1.003s][info   ][gc,cpu         ] GC(634) User=0.00s Sys=0.00s Real=0.00s
[1.003s][info   ][gc,start       ] GC(635) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[1.003s][info   ][gc,task        ] GC(635) Using 8 workers of 8 for evacuation
[1.003s][info   ][gc,phases      ] GC(635)   Pre Evacuate Collection Set: 0.1ms
[1.003s][info   ][gc,phases      ] GC(635)   Merge Heap Roots: 0.0ms
[1.003s][info   ][gc,phases      ] GC(635)   Evacuate Collection Set: 0.2ms
[1.003s][info   ][gc,phases      ] GC(635)   Post Evacuate Collection Set: 0.1ms
[1.003s][info   ][gc,phases      ] GC(635)   Other: 0.0ms
[1.003s][info   ][gc,heap        ] GC(635) Eden regions: 1->0(23)
[1.003s][info   ][gc,heap        ] GC(635) Survivor regions: 4->2(4)
[1.003s][info   ][gc,heap        ] GC(635) Old regions: 319->322
[1.003s][info   ][gc,heap        ] GC(635) Archive regions: 2->2
[1.003s][info   ][gc,heap        ] GC(635) Humongous regions: 168->168
[1.003s][info   ][gc,metaspace   ] GC(635) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.003s][info   ][gc             ] GC(635) Pause Young (Concurrent Start) (G1 Humongous Allocation) 469M->469M(512M) 0.544ms
[1.003s][info   ][gc             ] GC(636) Concurrent Mark Cycle
[1.003s][info   ][gc,cpu         ] GC(635) User=0.00s Sys=0.00s Real=0.00s
[1.003s][info   ][gc,marking     ] GC(636) Concurrent Clear Claimed Marks
[1.003s][info   ][gc,marking     ] GC(636) Concurrent Clear Claimed Marks 0.024ms
[1.003s][info   ][gc,marking     ] GC(636) Concurrent Scan Root Regions
[1.003s][info   ][gc,marking     ] GC(636) Concurrent Scan Root Regions 0.041ms
[1.003s][info   ][gc,marking     ] GC(636) Concurrent Mark
[1.003s][info   ][gc,marking     ] GC(636) Concurrent Mark From Roots
[1.003s][info   ][gc,task        ] GC(636) Using 2 workers of 2 for marking
[1.004s][info   ][gc,start       ] GC(637) Pause Young (Normal) (G1 Preventive Collection)
[1.004s][info   ][gc,task        ] GC(637) Using 8 workers of 8 for evacuation
[1.004s][info   ][gc,phases      ] GC(637)   Pre Evacuate Collection Set: 0.1ms
[1.004s][info   ][gc,phases      ] GC(637)   Merge Heap Roots: 0.0ms
[1.004s][info   ][gc,phases      ] GC(637)   Evacuate Collection Set: 0.1ms
[1.004s][info   ][gc,phases      ] GC(637)   Post Evacuate Collection Set: 0.1ms
[1.004s][info   ][gc,phases      ] GC(637)   Other: 0.0ms
[1.004s][info   ][gc,heap        ] GC(637) Eden regions: 6->0(21)
[1.004s][info   ][gc,heap        ] GC(637) Survivor regions: 2->4(4)
[1.004s][info   ][gc,heap        ] GC(637) Old regions: 322->322
[1.004s][info   ][gc,heap        ] GC(637) Archive regions: 2->2
[1.004s][info   ][gc,heap        ] GC(637) Humongous regions: 175->170
[1.004s][info   ][gc,metaspace   ] GC(637) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.004s][info   ][gc             ] GC(637) Pause Young (Normal) (G1 Preventive Collection) 481M->474M(512M) 0.489ms
[1.004s][info   ][gc,cpu         ] GC(637) User=0.00s Sys=0.00s Real=0.00s
[1.005s][info   ][gc,start       ] GC(638) Pause Young (Normal) (G1 Preventive Collection)
[1.005s][info   ][gc,task        ] GC(638) Using 8 workers of 8 for evacuation
[1.005s][info   ][gc,phases      ] GC(638)   Pre Evacuate Collection Set: 0.1ms
[1.005s][info   ][gc,phases      ] GC(638)   Merge Heap Roots: 0.0ms
[1.005s][info   ][gc,phases      ] GC(638)   Evacuate Collection Set: 0.2ms
[1.005s][info   ][gc,phases      ] GC(638)   Post Evacuate Collection Set: 0.1ms
[1.005s][info   ][gc,phases      ] GC(638)   Other: 0.0ms
[1.005s][info   ][gc,heap        ] GC(638) Eden regions: 4->0(24)
[1.005s][info   ][gc,heap        ] GC(638) Survivor regions: 4->1(4)
[1.005s][info   ][gc,heap        ] GC(638) Old regions: 322->326
[1.005s][info   ][gc,heap        ] GC(638) Archive regions: 2->2
[1.005s][info   ][gc,heap        ] GC(638) Humongous regions: 173->170
[1.005s][info   ][gc,metaspace   ] GC(638) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.005s][info   ][gc             ] GC(638) Pause Young (Normal) (G1 Preventive Collection) 480M->475M(512M) 0.494ms
[1.005s][info   ][gc,cpu         ] GC(638) User=0.00s Sys=0.00s Real=0.00s
[1.005s][info   ][gc,marking     ] GC(636) Concurrent Mark From Roots 1.901ms
[1.005s][info   ][gc,marking     ] GC(636) Concurrent Preclean
[1.005s][info   ][gc,marking     ] GC(636) Concurrent Preclean 0.008ms
[1.005s][info   ][gc,start       ] GC(636) Pause Remark
[1.006s][info   ][gc             ] GC(636) Pause Remark 479M->479M(512M) 0.224ms
[1.006s][info   ][gc,cpu         ] GC(636) User=0.00s Sys=0.00s Real=0.00s
[1.006s][info   ][gc,marking     ] GC(636) Concurrent Mark 2.274ms
[1.006s][info   ][gc,marking     ] GC(636) Concurrent Rebuild Remembered Sets
[1.006s][info   ][gc,start       ] GC(639) Pause Young (Normal) (G1 Preventive Collection)
[1.006s][info   ][gc,task        ] GC(639) Using 8 workers of 8 for evacuation
[1.007s][info   ][gc,phases      ] GC(639)   Pre Evacuate Collection Set: 0.1ms
[1.007s][info   ][gc,phases      ] GC(639)   Merge Heap Roots: 0.1ms
[1.007s][info   ][gc,phases      ] GC(639)   Evacuate Collection Set: 0.1ms
[1.007s][info   ][gc,phases      ] GC(639)   Post Evacuate Collection Set: 0.1ms
[1.007s][info   ][gc,phases      ] GC(639)   Other: 0.0ms
[1.007s][info   ][gc,heap        ] GC(639) Eden regions: 5->0(22)
[1.007s][info   ][gc,heap        ] GC(639) Survivor regions: 1->3(4)
[1.007s][info   ][gc,heap        ] GC(639) Old regions: 326->326
[1.007s][info   ][gc,heap        ] GC(639) Archive regions: 2->2
[1.007s][info   ][gc,heap        ] GC(639) Humongous regions: 174->169
[1.007s][info   ][gc,metaspace   ] GC(639) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.007s][info   ][gc             ] GC(639) Pause Young (Normal) (G1 Preventive Collection) 483M->475M(512M) 0.597ms
[1.007s][info   ][gc,cpu         ] GC(639) User=0.00s Sys=0.00s Real=0.00s
[1.007s][info   ][gc,start       ] GC(640) Pause Young (Normal) (G1 Preventive Collection)
[1.007s][info   ][gc,task        ] GC(640) Using 8 workers of 8 for evacuation
[1.007s][info   ][gc,phases      ] GC(640)   Pre Evacuate Collection Set: 0.1ms
[1.007s][info   ][gc,phases      ] GC(640)   Merge Heap Roots: 0.0ms
[1.007s][info   ][gc,phases      ] GC(640)   Evacuate Collection Set: 0.1ms
[1.007s][info   ][gc,phases      ] GC(640)   Post Evacuate Collection Set: 0.1ms
[1.007s][info   ][gc,phases      ] GC(640)   Other: 0.0ms
[1.007s][info   ][gc,heap        ] GC(640) Eden regions: 4->0(22)
[1.007s][info   ][gc,heap        ] GC(640) Survivor regions: 3->3(4)
[1.007s][info   ][gc,heap        ] GC(640) Old regions: 326->327
[1.007s][info   ][gc,heap        ] GC(640) Archive regions: 2->2
[1.007s][info   ][gc,heap        ] GC(640) Humongous regions: 172->168
[1.007s][info   ][gc,metaspace   ] GC(640) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.007s][info   ][gc             ] GC(640) Pause Young (Normal) (G1 Preventive Collection) 481M->475M(512M) 0.497ms
[1.007s][info   ][gc,cpu         ] GC(640) User=0.00s Sys=0.00s Real=0.00s
[1.008s][info   ][gc,marking     ] GC(636) Concurrent Rebuild Remembered Sets 2.104ms
[1.008s][info   ][gc,start       ] GC(636) Pause Cleanup
[1.008s][info   ][gc             ] GC(636) Pause Cleanup 481M->481M(512M) 0.139ms
[1.008s][info   ][gc,cpu         ] GC(636) User=0.00s Sys=0.00s Real=0.00s
[1.008s][info   ][gc,marking     ] GC(636) Concurrent Cleanup for Next Mark
[1.008s][info   ][gc,start       ] GC(641) Pause Young (Prepare Mixed) (G1 Preventive Collection)
[1.008s][info   ][gc,task        ] GC(641) Using 8 workers of 8 for evacuation
[1.009s][info   ][gc,phases      ] GC(641)   Pre Evacuate Collection Set: 0.1ms
[1.009s][info   ][gc,phases      ] GC(641)   Merge Heap Roots: 0.0ms
[1.009s][info   ][gc,phases      ] GC(641)   Evacuate Collection Set: 0.2ms
[1.009s][info   ][gc,phases      ] GC(641)   Post Evacuate Collection Set: 0.1ms
[1.009s][info   ][gc,phases      ] GC(641)   Other: 0.0ms
[1.009s][info   ][gc,heap        ] GC(641) Eden regions: 3->0(22)
[1.009s][info   ][gc,heap        ] GC(641) Survivor regions: 3->3(4)
[1.009s][info   ][gc,heap        ] GC(641) Old regions: 327->328
[1.009s][info   ][gc,heap        ] GC(641) Archive regions: 2->2
[1.009s][info   ][gc,heap        ] GC(641) Humongous regions: 172->169
[1.009s][info   ][gc,metaspace   ] GC(641) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.009s][info   ][gc             ] GC(641) Pause Young (Prepare Mixed) (G1 Preventive Collection) 482M->478M(512M) 0.489ms
[1.009s][info   ][gc,cpu         ] GC(641) User=0.00s Sys=0.00s Real=0.00s
[1.009s][info   ][gc,start       ] GC(642) Pause Young (Mixed) (G1 Preventive Collection)
[1.009s][info   ][gc,task        ] GC(642) Using 8 workers of 8 for evacuation
[1.009s][info   ][gc,phases      ] GC(642)   Pre Evacuate Collection Set: 0.1ms
[1.009s][info   ][gc,phases      ] GC(642)   Merge Heap Roots: 0.0ms
[1.009s][info   ][gc,phases      ] GC(642)   Evacuate Collection Set: 0.2ms
[1.009s][info   ][gc,phases      ] GC(642)   Post Evacuate Collection Set: 0.2ms
[1.009s][info   ][gc,phases      ] GC(642)   Other: 0.0ms
[1.009s][info   ][gc,heap        ] GC(642) Eden regions: 1->0(24)
[1.009s][info   ][gc,heap        ] GC(642) Survivor regions: 3->1(4)
[1.009s][info   ][gc,heap        ] GC(642) Old regions: 328->322
[1.009s][info   ][gc,heap        ] GC(642) Archive regions: 2->2
[1.009s][info   ][gc,heap        ] GC(642) Humongous regions: 173->170
[1.009s][info   ][gc,metaspace   ] GC(642) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.009s][info   ][gc             ] GC(642) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 483M->472M(512M) 0.697ms
[1.009s][info   ][gc,cpu         ] GC(642) User=0.00s Sys=0.00s Real=0.00s
[1.010s][info   ][gc,marking     ] GC(636) Concurrent Cleanup for Next Mark 1.582ms
[1.010s][info   ][gc             ] GC(636) Concurrent Mark Cycle 6.365ms
[1.010s][info   ][gc,start       ] GC(643) Pause Young (Mixed) (G1 Preventive Collection)
[1.010s][info   ][gc,task        ] GC(643) Using 8 workers of 8 for evacuation
[1.011s][info   ][gc,phases      ] GC(643)   Pre Evacuate Collection Set: 0.1ms
[1.011s][info   ][gc,phases      ] GC(643)   Merge Heap Roots: 0.0ms
[1.011s][info   ][gc,phases      ] GC(643)   Evacuate Collection Set: 0.3ms
[1.011s][info   ][gc,phases      ] GC(643)   Post Evacuate Collection Set: 0.2ms
[1.011s][info   ][gc,phases      ] GC(643)   Other: 0.0ms
[1.011s][info   ][gc,heap        ] GC(643) Eden regions: 4->0(23)
[1.011s][info   ][gc,heap        ] GC(643) Survivor regions: 1->2(4)
[1.011s][info   ][gc,heap        ] GC(643) Old regions: 322->330
[1.011s][info   ][gc,heap        ] GC(643) Archive regions: 2->2
[1.011s][info   ][gc,heap        ] GC(643) Humongous regions: 171->169
[1.011s][info   ][gc,metaspace   ] GC(643) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.011s][info   ][gc             ] GC(643) Pause Young (Mixed) (G1 Preventive Collection) (Evacuation Failure) 476M->481M(512M) 0.768ms
[1.011s][info   ][gc,cpu         ] GC(643) User=0.01s Sys=0.01s Real=0.00s
[1.011s][info   ][gc,start       ] GC(644) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[1.011s][info   ][gc,task        ] GC(644) Using 8 workers of 8 for evacuation
[1.011s][info   ][gc,phases      ] GC(644)   Pre Evacuate Collection Set: 0.2ms
[1.011s][info   ][gc,phases      ] GC(644)   Merge Heap Roots: 0.0ms
[1.011s][info   ][gc,phases      ] GC(644)   Evacuate Collection Set: 0.1ms
[1.011s][info   ][gc,phases      ] GC(644)   Post Evacuate Collection Set: 0.1ms
[1.011s][info   ][gc,phases      ] GC(644)   Other: 0.1ms
[1.011s][info   ][gc,heap        ] GC(644) Eden regions: 1->0(23)
[1.011s][info   ][gc,heap        ] GC(644) Survivor regions: 2->2(4)
[1.011s][info   ][gc,heap        ] GC(644) Old regions: 330->330
[1.011s][info   ][gc,heap        ] GC(644) Archive regions: 2->2
[1.011s][info   ][gc,heap        ] GC(644) Humongous regions: 170->169
[1.011s][info   ][gc,metaspace   ] GC(644) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.011s][info   ][gc             ] GC(644) Pause Young (Concurrent Start) (G1 Humongous Allocation) 482M->480M(512M) 0.598ms
[1.011s][info   ][gc,cpu         ] GC(644) User=0.00s Sys=0.00s Real=0.01s
[1.011s][info   ][gc             ] GC(645) Concurrent Mark Cycle
[1.011s][info   ][gc,marking     ] GC(645) Concurrent Clear Claimed Marks
[1.011s][info   ][gc,marking     ] GC(645) Concurrent Clear Claimed Marks 0.006ms
[1.011s][info   ][gc,marking     ] GC(645) Concurrent Scan Root Regions
[1.011s][info   ][gc,marking     ] GC(645) Concurrent Scan Root Regions 0.021ms
[1.011s][info   ][gc,marking     ] GC(645) Concurrent Mark
[1.011s][info   ][gc,marking     ] GC(645) Concurrent Mark From Roots
[1.011s][info   ][gc,task        ] GC(645) Using 2 workers of 2 for marking
[1.012s][info   ][gc,start       ] GC(646) Pause Young (Normal) (G1 Preventive Collection)
[1.012s][info   ][gc,task        ] GC(646) Using 8 workers of 8 for evacuation
[1.012s][info   ][gc,phases      ] GC(646)   Pre Evacuate Collection Set: 0.1ms
[1.012s][info   ][gc,phases      ] GC(646)   Merge Heap Roots: 0.0ms
[1.012s][info   ][gc,phases      ] GC(646)   Evacuate Collection Set: 0.1ms
[1.012s][info   ][gc,phases      ] GC(646)   Post Evacuate Collection Set: 0.1ms
[1.012s][info   ][gc,phases      ] GC(646)   Other: 0.0ms
[1.012s][info   ][gc,heap        ] GC(646) Eden regions: 3->0(21)
[1.012s][info   ][gc,heap        ] GC(646) Survivor regions: 2->4(4)
[1.012s][info   ][gc,heap        ] GC(646) Old regions: 330->330
[1.012s][info   ][gc,heap        ] GC(646) Archive regions: 2->2
[1.012s][info   ][gc,heap        ] GC(646) Humongous regions: 171->169
[1.012s][info   ][gc,metaspace   ] GC(646) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.012s][info   ][gc             ] GC(646) Pause Young (Normal) (G1 Preventive Collection) 485M->482M(512M) 0.549ms
[1.012s][info   ][gc,cpu         ] GC(646) User=0.00s Sys=0.00s Real=0.00s
[1.012s][info   ][gc,start       ] GC(647) Pause Young (Normal) (G1 Preventive Collection)
[1.012s][info   ][gc,task        ] GC(647) Using 8 workers of 8 for evacuation
[1.013s][info   ][gc,phases      ] GC(647)   Pre Evacuate Collection Set: 0.1ms
[1.013s][info   ][gc,phases      ] GC(647)   Merge Heap Roots: 0.0ms
[1.013s][info   ][gc,phases      ] GC(647)   Evacuate Collection Set: 0.1ms
[1.013s][info   ][gc,phases      ] GC(647)   Post Evacuate Collection Set: 0.1ms
[1.013s][info   ][gc,phases      ] GC(647)   Other: 0.0ms
[1.013s][info   ][gc,heap        ] GC(647) Eden regions: 1->0(23)
[1.013s][info   ][gc,heap        ] GC(647) Survivor regions: 4->2(4)
[1.013s][info   ][gc,heap        ] GC(647) Old regions: 330->332
[1.013s][info   ][gc,heap        ] GC(647) Archive regions: 2->2
[1.013s][info   ][gc,heap        ] GC(647) Humongous regions: 170->170
[1.013s][info   ][gc,metaspace   ] GC(647) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.013s][info   ][gc             ] GC(647) Pause Young (Normal) (G1 Preventive Collection) 484M->483M(512M) 0.494ms
[1.013s][info   ][gc,cpu         ] GC(647) User=0.00s Sys=0.00s Real=0.00s
[1.013s][info   ][gc,start       ] GC(648) Pause Young (Normal) (G1 Preventive Collection)
[1.013s][info   ][gc,task        ] GC(648) Using 8 workers of 8 for evacuation
[1.014s][info   ][gc,phases      ] GC(648)   Pre Evacuate Collection Set: 0.1ms
[1.014s][info   ][gc,phases      ] GC(648)   Merge Heap Roots: 0.0ms
[1.014s][info   ][gc,phases      ] GC(648)   Evacuate Collection Set: 0.2ms
[1.014s][info   ][gc,phases      ] GC(648)   Post Evacuate Collection Set: 0.1ms
[1.014s][info   ][gc,phases      ] GC(648)   Other: 0.0ms
[1.014s][info   ][gc,heap        ] GC(648) Eden regions: 3->0(22)
[1.014s][info   ][gc,heap        ] GC(648) Survivor regions: 2->3(3)
[1.014s][info   ][gc,heap        ] GC(648) Old regions: 332->332
[1.014s][info   ][gc,heap        ] GC(648) Archive regions: 2->2
[1.014s][info   ][gc,heap        ] GC(648) Humongous regions: 170->170
[1.014s][info   ][gc,metaspace   ] GC(648) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.014s][info   ][gc             ] GC(648) Pause Young (Normal) (G1 Preventive Collection) 486M->484M(512M) 0.535ms
[1.014s][info   ][gc,cpu         ] GC(648) User=0.00s Sys=0.00s Real=0.00s
[1.014s][info   ][gc,start       ] GC(649) Pause Young (Normal) (G1 Preventive Collection)
[1.014s][info   ][gc,task        ] GC(649) Using 8 workers of 8 for evacuation
[1.014s][info   ][gc,phases      ] GC(649)   Pre Evacuate Collection Set: 0.1ms
[1.014s][info   ][gc,phases      ] GC(649)   Merge Heap Roots: 0.0ms
[1.014s][info   ][gc,phases      ] GC(649)   Evacuate Collection Set: 0.1ms
[1.014s][info   ][gc,phases      ] GC(649)   Post Evacuate Collection Set: 0.1ms
[1.014s][info   ][gc,phases      ] GC(649)   Other: 0.0ms
[1.014s][info   ][gc,heap        ] GC(649) Eden regions: 1->0(23)
[1.014s][info   ][gc,heap        ] GC(649) Survivor regions: 3->2(4)
[1.014s][info   ][gc,heap        ] GC(649) Old regions: 332->334
[1.014s][info   ][gc,heap        ] GC(649) Archive regions: 2->2
[1.014s][info   ][gc,heap        ] GC(649) Humongous regions: 170->170
[1.014s][info   ][gc,metaspace   ] GC(649) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.014s][info   ][gc             ] GC(649) Pause Young (Normal) (G1 Preventive Collection) 485M->484M(512M) 0.500ms
[1.014s][info   ][gc,cpu         ] GC(649) User=0.00s Sys=0.00s Real=0.00s
[1.015s][info   ][gc,start       ] GC(650) Pause Young (Normal) (G1 Preventive Collection)
[1.015s][info   ][gc,task        ] GC(650) Using 8 workers of 8 for evacuation
[1.015s][info   ][gc,phases      ] GC(650)   Pre Evacuate Collection Set: 0.1ms
[1.015s][info   ][gc,phases      ] GC(650)   Merge Heap Roots: 0.1ms
[1.015s][info   ][gc,phases      ] GC(650)   Evacuate Collection Set: 0.1ms
[1.015s][info   ][gc,phases      ] GC(650)   Post Evacuate Collection Set: 0.1ms
[1.015s][info   ][gc,phases      ] GC(650)   Other: 0.0ms
[1.015s][info   ][gc,heap        ] GC(650) Eden regions: 2->0(23)
[1.015s][info   ][gc,heap        ] GC(650) Survivor regions: 2->2(2)
[1.015s][info   ][gc,heap        ] GC(650) Old regions: 334->334
[1.015s][info   ][gc,heap        ] GC(650) Archive regions: 2->2
[1.015s][info   ][gc,heap        ] GC(650) Humongous regions: 170->170
[1.015s][info   ][gc,metaspace   ] GC(650) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.015s][info   ][gc             ] GC(650) Pause Young (Normal) (G1 Preventive Collection) 486M->485M(512M) 0.448ms
[1.015s][info   ][gc,cpu         ] GC(650) User=0.00s Sys=0.00s Real=0.00s
[1.015s][info   ][gc,marking     ] GC(645) Concurrent Mark From Roots 3.787ms
[1.015s][info   ][gc,marking     ] GC(645) Concurrent Preclean
[1.015s][info   ][gc,marking     ] GC(645) Concurrent Preclean 0.009ms
[1.015s][info   ][gc,start       ] GC(645) Pause Remark
[1.016s][info   ][gc             ] GC(645) Pause Remark 486M->485M(512M) 0.236ms
[1.016s][info   ][gc,cpu         ] GC(645) User=0.00s Sys=0.00s Real=0.00s
[1.016s][info   ][gc,marking     ] GC(645) Concurrent Mark 4.231ms
[1.016s][info   ][gc,marking     ] GC(645) Concurrent Rebuild Remembered Sets
[1.016s][info   ][gc,start       ] GC(651) Pause Young (Normal) (G1 Preventive Collection)
[1.016s][info   ][gc,task        ] GC(651) Using 8 workers of 8 for evacuation
[1.016s][info   ][gc,phases      ] GC(651)   Pre Evacuate Collection Set: 0.1ms
[1.016s][info   ][gc,phases      ] GC(651)   Merge Heap Roots: 0.0ms
[1.016s][info   ][gc,phases      ] GC(651)   Evacuate Collection Set: 0.1ms
[1.016s][info   ][gc,phases      ] GC(651)   Post Evacuate Collection Set: 0.1ms
[1.016s][info   ][gc,phases      ] GC(651)   Other: 0.0ms
[1.016s][info   ][gc,heap        ] GC(651) Eden regions: 2->0(22)
[1.016s][info   ][gc,heap        ] GC(651) Survivor regions: 2->3(3)
[1.016s][info   ][gc,heap        ] GC(651) Old regions: 333->333
[1.016s][info   ][gc,heap        ] GC(651) Archive regions: 2->2
[1.016s][info   ][gc,heap        ] GC(651) Humongous regions: 170->170
[1.016s][info   ][gc,metaspace   ] GC(651) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.016s][info   ][gc             ] GC(651) Pause Young (Normal) (G1 Preventive Collection) 486M->485M(512M) 0.538ms
[1.016s][info   ][gc,cpu         ] GC(651) User=0.01s Sys=0.00s Real=0.00s
[1.016s][info   ][gc,start       ] GC(652) Pause Young (Normal) (G1 Preventive Collection)
[1.016s][info   ][gc,task        ] GC(652) Using 8 workers of 8 for evacuation
[1.017s][info   ][gc,phases      ] GC(652)   Pre Evacuate Collection Set: 0.1ms
[1.017s][info   ][gc,phases      ] GC(652)   Merge Heap Roots: 0.0ms
[1.017s][info   ][gc,phases      ] GC(652)   Evacuate Collection Set: 0.1ms
[1.017s][info   ][gc,phases      ] GC(652)   Post Evacuate Collection Set: 0.1ms
[1.017s][info   ][gc,phases      ] GC(652)   Other: 0.0ms
[1.017s][info   ][gc,heap        ] GC(652) Eden regions: 0->0(23)
[1.017s][info   ][gc,heap        ] GC(652) Survivor regions: 3->2(3)
[1.017s][info   ][gc,heap        ] GC(652) Old regions: 333->333
[1.017s][info   ][gc,heap        ] GC(652) Archive regions: 2->2
[1.017s][info   ][gc,heap        ] GC(652) Humongous regions: 171->171
[1.017s][info   ][gc,metaspace   ] GC(652) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.017s][info   ][gc             ] GC(652) Pause Young (Normal) (G1 Preventive Collection) 486M->485M(512M) 0.497ms
[1.017s][info   ][gc,cpu         ] GC(652) User=0.00s Sys=0.00s Real=0.00s
[1.017s][info   ][gc,start       ] GC(653) Pause Young (Normal) (G1 Preventive Collection)
[1.017s][info   ][gc,task        ] GC(653) Using 8 workers of 8 for evacuation
[1.018s][info   ][gc,phases      ] GC(653)   Pre Evacuate Collection Set: 0.1ms
[1.018s][info   ][gc,phases      ] GC(653)   Merge Heap Roots: 0.1ms
[1.018s][info   ][gc,phases      ] GC(653)   Evacuate Collection Set: 0.1ms
[1.018s][info   ][gc,phases      ] GC(653)   Post Evacuate Collection Set: 0.1ms
[1.018s][info   ][gc,phases      ] GC(653)   Other: 0.0ms
[1.018s][info   ][gc,heap        ] GC(653) Eden regions: 1->0(22)
[1.018s][info   ][gc,heap        ] GC(653) Survivor regions: 2->3(3)
[1.018s][info   ][gc,heap        ] GC(653) Old regions: 333->333
[1.018s][info   ][gc,heap        ] GC(653) Archive regions: 2->2
[1.018s][info   ][gc,heap        ] GC(653) Humongous regions: 171->170
[1.018s][info   ][gc,metaspace   ] GC(653) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.018s][info   ][gc             ] GC(653) Pause Young (Normal) (G1 Preventive Collection) 486M->485M(512M) 0.465ms
[1.018s][info   ][gc,cpu         ] GC(653) User=0.00s Sys=0.00s Real=0.00s
[1.018s][info   ][gc,start       ] GC(654) Pause Young (Normal) (G1 Preventive Collection)
[1.018s][info   ][gc,task        ] GC(654) Using 8 workers of 8 for evacuation
[1.018s][info   ][gc,phases      ] GC(654)   Pre Evacuate Collection Set: 0.1ms
[1.018s][info   ][gc,phases      ] GC(654)   Merge Heap Roots: 0.0ms
[1.018s][info   ][gc,phases      ] GC(654)   Evacuate Collection Set: 0.1ms
[1.018s][info   ][gc,phases      ] GC(654)   Post Evacuate Collection Set: 0.1ms
[1.018s][info   ][gc,phases      ] GC(654)   Other: 0.0ms
[1.018s][info   ][gc,heap        ] GC(654) Eden regions: 1->0(23)
[1.018s][info   ][gc,heap        ] GC(654) Survivor regions: 3->2(3)
[1.018s][info   ][gc,heap        ] GC(654) Old regions: 333->334
[1.018s][info   ][gc,heap        ] GC(654) Archive regions: 2->2
[1.018s][info   ][gc,heap        ] GC(654) Humongous regions: 170->170
[1.018s][info   ][gc,metaspace   ] GC(654) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.018s][info   ][gc             ] GC(654) Pause Young (Normal) (G1 Preventive Collection) 486M->485M(512M) 0.411ms
[1.018s][info   ][gc,cpu         ] GC(654) User=0.00s Sys=0.00s Real=0.00s
[1.019s][info   ][gc,marking     ] GC(645) Concurrent Rebuild Remembered Sets 3.122ms
[1.019s][info   ][gc,start       ] GC(645) Pause Cleanup
[1.019s][info   ][gc             ] GC(645) Pause Cleanup 487M->487M(512M) 0.243ms
[1.019s][info   ][gc,cpu         ] GC(645) User=0.00s Sys=0.00s Real=0.00s
[1.019s][info   ][gc,marking     ] GC(645) Concurrent Cleanup for Next Mark
[1.019s][info   ][gc,marking     ] GC(645) Concurrent Cleanup for Next Mark 0.282ms
[1.020s][info   ][gc             ] GC(645) Concurrent Mark Cycle 8.118ms
counter:29048
[1.025s][info   ][gc,heap,exit   ] Heap
[1.025s][info   ][gc,heap,exit   ]  garbage-first heap   total 524288K, used 499181K [0x00000007e0000000, 0x0000000800000000)
[1.025s][info   ][gc,heap,exit   ]   region size 1024K, 3 young (3072K), 2 survivors (2048K)
[1.025s][info   ][gc,heap,exit   ]  Metaspace       used 233K, committed 448K, reserved 1114112K
[1.025s][info   ][gc,heap,exit   ]   class space    used 9K, committed 128K, reserved 1048576K
```

接下来试试1g内存
```
java -XX:+UseG1GC -Xms1g -Xmx1g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以发现效率大大提升，生成了56000多个对象。GC了174次。其吞吐量相比并行GC稍微提升，GC次数却很多，说明GC期间很少影响业务。
```
[0.628s][info   ][gc,phases   ] GC(92)   Post Evacuate Collection Set: 0.2ms
[0.628s][info   ][gc,phases   ] GC(92)   Other: 0.1ms
[0.628s][info   ][gc,heap     ] GC(92) Eden regions: 74->0(67)
[0.628s][info   ][gc,heap     ] GC(92) Survivor regions: 15->12(12)
[0.628s][info   ][gc,heap     ] GC(92) Old regions: 545->579
[0.628s][info   ][gc,heap     ] GC(92) Archive regions: 2->2
[0.628s][info   ][gc,heap     ] GC(92) Humongous regions: 209->180
[0.628s][info   ][gc,metaspace] GC(92) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.628s][info   ][gc          ] GC(92) Pause Young (Concurrent Start) (G1 Humongous Allocation) 843M->771M(1024M) 1.601ms
[0.628s][info   ][gc,cpu      ] GC(92) User=0.01s Sys=0.00s Real=0.00s
[0.628s][info   ][gc          ] GC(93) Concurrent Mark Cycle
[0.628s][info   ][gc,marking  ] GC(93) Concurrent Clear Claimed Marks
[0.628s][info   ][gc,marking  ] GC(93) Concurrent Clear Claimed Marks 0.021ms
[0.628s][info   ][gc,marking  ] GC(93) Concurrent Scan Root Regions
[0.628s][info   ][gc,marking  ] GC(93) Concurrent Scan Root Regions 0.063ms
[0.628s][info   ][gc,marking  ] GC(93) Concurrent Mark
[0.628s][info   ][gc,marking  ] GC(93) Concurrent Mark From Roots
[0.628s][info   ][gc,task     ] GC(93) Using 2 workers of 2 for marking
[0.629s][info   ][gc,marking  ] GC(93) Concurrent Mark From Roots 1.243ms
[0.629s][info   ][gc,marking  ] GC(93) Concurrent Preclean
[0.629s][info   ][gc,marking  ] GC(93) Concurrent Preclean 0.012ms
[0.629s][info   ][gc,start    ] GC(93) Pause Remark
[0.630s][info   ][gc          ] GC(93) Pause Remark 798M->729M(1024M) 0.345ms
[0.630s][info   ][gc,cpu      ] GC(93) User=0.00s Sys=0.00s Real=0.00s
[0.630s][info   ][gc,marking  ] GC(93) Concurrent Mark 1.879ms
[0.630s][info   ][gc,marking  ] GC(93) Concurrent Rebuild Remembered Sets
[0.631s][info   ][gc,marking  ] GC(93) Concurrent Rebuild Remembered Sets 0.920ms
[0.631s][info   ][gc,start    ] GC(93) Pause Cleanup
[0.631s][info   ][gc          ] GC(93) Pause Cleanup 744M->744M(1024M) 0.167ms
[0.631s][info   ][gc,cpu      ] GC(93) User=0.00s Sys=0.00s Real=0.00s
[0.631s][info   ][gc,marking  ] GC(93) Concurrent Cleanup for Next Mark
[0.632s][info   ][gc,marking  ] GC(93) Concurrent Cleanup for Next Mark 0.507ms
[0.632s][info   ][gc          ] GC(93) Concurrent Mark Cycle 3.847ms
[0.634s][info   ][gc,start    ] GC(94) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.634s][info   ][gc,task     ] GC(94) Using 8 workers of 8 for evacuation
[0.635s][info   ][gc,phases   ] GC(94)   Pre Evacuate Collection Set: 0.1ms
[0.635s][info   ][gc,phases   ] GC(94)   Merge Heap Roots: 0.1ms
[0.635s][info   ][gc,phases   ] GC(94)   Evacuate Collection Set: 1.0ms
[0.635s][info   ][gc,phases   ] GC(94)   Post Evacuate Collection Set: 0.2ms
[0.635s][info   ][gc,phases   ] GC(94)   Other: 0.0ms
[0.635s][info   ][gc,heap     ] GC(94) Eden regions: 67->0(41)
[0.635s][info   ][gc,heap     ] GC(94) Survivor regions: 12->10(10)
[0.635s][info   ][gc,heap     ] GC(94) Old regions: 510->542
[0.635s][info   ][gc,heap     ] GC(94) Archive regions: 2->2
[0.635s][info   ][gc,heap     ] GC(94) Humongous regions: 206->173
[0.635s][info   ][gc,metaspace] GC(94) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.635s][info   ][gc          ] GC(94) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 795M->725M(1024M) 1.636ms
[0.635s][info   ][gc,cpu      ] GC(94) User=0.00s Sys=0.00s Real=0.00s
[0.639s][info   ][gc,start    ] GC(95) Pause Young (Mixed) (G1 Evacuation Pause)
[0.639s][info   ][gc,task     ] GC(95) Using 8 workers of 8 for evacuation
[0.640s][info   ][gc,phases   ] GC(95)   Pre Evacuate Collection Set: 0.1ms
[0.640s][info   ][gc,phases   ] GC(95)   Merge Heap Roots: 0.1ms
[0.640s][info   ][gc,phases   ] GC(95)   Evacuate Collection Set: 0.8ms
[0.640s][info   ][gc,phases   ] GC(95)   Post Evacuate Collection Set: 0.2ms
[0.640s][info   ][gc,phases   ] GC(95)   Other: 0.0ms
[0.640s][info   ][gc,heap     ] GC(95) Eden regions: 41->0(44)
[0.640s][info   ][gc,heap     ] GC(95) Survivor regions: 10->7(7)
[0.640s][info   ][gc,heap     ] GC(95) Old regions: 542->463
[0.640s][info   ][gc,heap     ] GC(95) Archive regions: 2->2
[0.640s][info   ][gc,heap     ] GC(95) Humongous regions: 191->174
[0.640s][info   ][gc,metaspace] GC(95) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.640s][info   ][gc          ] GC(95) Pause Young (Mixed) (G1 Evacuation Pause) 784M->644M(1024M) 1.392ms
[0.640s][info   ][gc,cpu      ] GC(95) User=0.01s Sys=0.00s Real=0.00s
[0.644s][info   ][gc,start    ] GC(96) Pause Young (Mixed) (G1 Evacuation Pause)
[0.644s][info   ][gc,task     ] GC(96) Using 8 workers of 8 for evacuation
[0.645s][info   ][gc,phases   ] GC(96)   Pre Evacuate Collection Set: 0.3ms
[0.645s][info   ][gc,phases   ] GC(96)   Merge Heap Roots: 0.1ms
[0.645s][info   ][gc,phases   ] GC(96)   Evacuate Collection Set: 1.1ms
[0.645s][info   ][gc,phases   ] GC(96)   Post Evacuate Collection Set: 0.2ms
[0.645s][info   ][gc,phases   ] GC(96)   Other: 0.0ms
[0.645s][info   ][gc,heap     ] GC(96) Eden regions: 44->0(44)
[0.645s][info   ][gc,heap     ] GC(96) Survivor regions: 7->7(7)
[0.645s][info   ][gc,heap     ] GC(96) Old regions: 463->402
[0.645s][info   ][gc,heap     ] GC(96) Archive regions: 2->2
[0.645s][info   ][gc,heap     ] GC(96) Humongous regions: 195->176
[0.645s][info   ][gc,metaspace] GC(96) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.645s][info   ][gc          ] GC(96) Pause Young (Mixed) (G1 Evacuation Pause) 709M->585M(1024M) 1.900ms
[0.645s][info   ][gc,cpu      ] GC(96) User=0.01s Sys=0.01s Real=0.00s
[0.649s][info   ][gc,start    ] GC(97) Pause Young (Mixed) (G1 Evacuation Pause)
[0.649s][info   ][gc,task     ] GC(97) Using 8 workers of 8 for evacuation
[0.650s][info   ][gc,phases   ] GC(97)   Pre Evacuate Collection Set: 0.1ms
[0.651s][info   ][gc,phases   ] GC(97)   Merge Heap Roots: 0.1ms
[0.651s][info   ][gc,phases   ] GC(97)   Evacuate Collection Set: 1.2ms
[0.651s][info   ][gc,phases   ] GC(97)   Post Evacuate Collection Set: 0.2ms
[0.651s][info   ][gc,phases   ] GC(97)   Other: 0.1ms
[0.651s][info   ][gc,heap     ] GC(97) Eden regions: 44->0(44)
[0.651s][info   ][gc,heap     ] GC(97) Survivor regions: 7->7(7)
[0.651s][info   ][gc,heap     ] GC(97) Old regions: 402->349
[0.651s][info   ][gc,heap     ] GC(97) Archive regions: 2->2
[0.651s][info   ][gc,heap     ] GC(97) Humongous regions: 202->170
[0.651s][info   ][gc,metaspace] GC(97) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.651s][info   ][gc          ] GC(97) Pause Young (Mixed) (G1 Evacuation Pause) 655M->526M(1024M) 1.993ms
[0.651s][info   ][gc,cpu      ] GC(97) User=0.00s Sys=0.00s Real=0.00s
[0.654s][info   ][gc,start    ] GC(98) Pause Young (Mixed) (G1 Evacuation Pause)
[0.654s][info   ][gc,task     ] GC(98) Using 8 workers of 8 for evacuation
[0.655s][info   ][gc,phases   ] GC(98)   Pre Evacuate Collection Set: 0.1ms
[0.655s][info   ][gc,phases   ] GC(98)   Merge Heap Roots: 0.1ms
[0.655s][info   ][gc,phases   ] GC(98)   Evacuate Collection Set: 0.5ms
[0.655s][info   ][gc,phases   ] GC(98)   Post Evacuate Collection Set: 0.1ms
[0.655s][info   ][gc,phases   ] GC(98)   Other: 0.0ms
[0.655s][info   ][gc,heap     ] GC(98) Eden regions: 44->0(182)
[0.655s][info   ][gc,heap     ] GC(98) Survivor regions: 7->7(7)
[0.655s][info   ][gc,heap     ] GC(98) Old regions: 349->367
[0.655s][info   ][gc,heap     ] GC(98) Archive regions: 2->2
[0.655s][info   ][gc,heap     ] GC(98) Humongous regions: 199->175
[0.655s][info   ][gc,metaspace] GC(98) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.655s][info   ][gc          ] GC(98) Pause Young (Mixed) (G1 Evacuation Pause) 599M->549M(1024M) 0.986ms
[0.655s][info   ][gc,cpu      ] GC(98) User=0.00s Sys=0.00s Real=0.00s
[0.669s][info   ][gc,start    ] GC(99) Pause Young (Normal) (G1 Evacuation Pause)
[0.669s][info   ][gc,task     ] GC(99) Using 8 workers of 8 for evacuation
[0.672s][info   ][gc,phases   ] GC(99)   Pre Evacuate Collection Set: 0.2ms
[0.672s][info   ][gc,phases   ] GC(99)   Merge Heap Roots: 0.1ms
[0.672s][info   ][gc,phases   ] GC(99)   Evacuate Collection Set: 1.8ms
[0.672s][info   ][gc,phases   ] GC(99)   Post Evacuate Collection Set: 0.2ms
[0.672s][info   ][gc,phases   ] GC(99)   Other: 0.0ms
[0.672s][info   ][gc,heap     ] GC(99) Eden regions: 182->0(156)
[0.672s][info   ][gc,heap     ] GC(99) Survivor regions: 7->24(24)
[0.672s][info   ][gc,heap     ] GC(99) Old regions: 367->413
[0.672s][info   ][gc,heap     ] GC(99) Archive regions: 2->2
[0.672s][info   ][gc,heap     ] GC(99) Humongous regions: 298->169
[0.672s][info   ][gc,metaspace] GC(99) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.672s][info   ][gc          ] GC(99) Pause Young (Normal) (G1 Evacuation Pause) 854M->606M(1024M) 2.417ms
[0.672s][info   ][gc,cpu      ] GC(99) User=0.01s Sys=0.00s Real=0.01s
[0.681s][info   ][gc,start    ] GC(100) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.681s][info   ][gc,task     ] GC(100) Using 8 workers of 8 for evacuation
[0.683s][info   ][gc,phases   ] GC(100)   Pre Evacuate Collection Set: 0.2ms
[0.683s][info   ][gc,phases   ] GC(100)   Merge Heap Roots: 0.0ms
[0.683s][info   ][gc,phases   ] GC(100)   Evacuate Collection Set: 1.3ms
[0.683s][info   ][gc,phases   ] GC(100)   Post Evacuate Collection Set: 0.2ms
[0.683s][info   ][gc,phases   ] GC(100)   Other: 0.1ms
[0.683s][info   ][gc,heap     ] GC(100) Eden regions: 128->0(134)
[0.683s][info   ][gc,heap     ] GC(100) Survivor regions: 24->23(23)
[0.683s][info   ][gc,heap     ] GC(100) Old regions: 413->455
[0.683s][info   ][gc,heap     ] GC(100) Archive regions: 2->2
[0.683s][info   ][gc,heap     ] GC(100) Humongous regions: 258->172
[0.683s][info   ][gc,metaspace] GC(100) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.683s][info   ][gc          ] GC(100) Pause Young (Concurrent Start) (G1 Humongous Allocation) 822M->650M(1024M) 2.140ms
[0.683s][info   ][gc,cpu      ] GC(100) User=0.01s Sys=0.00s Real=0.00s
[0.683s][info   ][gc          ] GC(101) Concurrent Undo Cycle
[0.683s][info   ][gc,marking  ] GC(101) Concurrent Cleanup for Next Mark
[0.683s][info   ][gc,marking  ] GC(101) Concurrent Cleanup for Next Mark 0.047ms
[0.684s][info   ][gc          ] GC(101) Concurrent Undo Cycle 0.114ms
[0.692s][info   ][gc,start    ] GC(102) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.692s][info   ][gc,task     ] GC(102) Using 8 workers of 8 for evacuation
[0.694s][info   ][gc,phases   ] GC(102)   Pre Evacuate Collection Set: 0.2ms
[0.694s][info   ][gc,phases   ] GC(102)   Merge Heap Roots: 0.1ms
[0.694s][info   ][gc,phases   ] GC(102)   Evacuate Collection Set: 1.6ms
[0.694s][info   ][gc,phases   ] GC(102)   Post Evacuate Collection Set: 0.2ms
[0.694s][info   ][gc,phases   ] GC(102)   Other: 0.1ms
[0.694s][info   ][gc,heap     ] GC(102) Eden regions: 111->0(116)
[0.694s][info   ][gc,heap     ] GC(102) Survivor regions: 23->20(20)
[0.694s][info   ][gc,heap     ] GC(102) Old regions: 455->495
[0.694s][info   ][gc,heap     ] GC(102) Archive regions: 2->2
[0.694s][info   ][gc,heap     ] GC(102) Humongous regions: 240->172
[0.694s][info   ][gc,metaspace] GC(102) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.694s][info   ][gc          ] GC(102) Pause Young (Concurrent Start) (G1 Humongous Allocation) 828M->687M(1024M) 2.197ms
[0.694s][info   ][gc,cpu      ] GC(102) User=0.01s Sys=0.01s Real=0.00s
[0.694s][info   ][gc          ] GC(103) Concurrent Undo Cycle
[0.694s][info   ][gc,marking  ] GC(103) Concurrent Cleanup for Next Mark
[0.694s][info   ][gc,marking  ] GC(103) Concurrent Cleanup for Next Mark 0.054ms
[0.694s][info   ][gc          ] GC(103) Concurrent Undo Cycle 0.134ms
[0.702s][info   ][gc,start    ] GC(104) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.702s][info   ][gc,task     ] GC(104) Using 8 workers of 8 for evacuation
[0.705s][info   ][gc,phases   ] GC(104)   Pre Evacuate Collection Set: 0.2ms
[0.705s][info   ][gc,phases   ] GC(104)   Merge Heap Roots: 0.1ms
[0.705s][info   ][gc,phases   ] GC(104)   Evacuate Collection Set: 1.9ms
[0.705s][info   ][gc,phases   ] GC(104)   Post Evacuate Collection Set: 0.3ms
[0.705s][info   ][gc,phases   ] GC(104)   Other: 0.1ms
[0.705s][info   ][gc,heap     ] GC(104) Eden regions: 107->0(99)
[0.705s][info   ][gc,heap     ] GC(104) Survivor regions: 20->17(17)
[0.705s][info   ][gc,heap     ] GC(104) Old regions: 495->535
[0.705s][info   ][gc,heap     ] GC(104) Archive regions: 2->2
[0.705s][info   ][gc,heap     ] GC(104) Humongous regions: 221->168
[0.705s][info   ][gc,metaspace] GC(104) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.705s][info   ][gc          ] GC(104) Pause Young (Concurrent Start) (G1 Humongous Allocation) 842M->720M(1024M) 2.767ms
[0.705s][info   ][gc,cpu      ] GC(104) User=0.00s Sys=0.00s Real=0.00s
[0.705s][info   ][gc          ] GC(105) Concurrent Undo Cycle
[0.705s][info   ][gc,marking  ] GC(105) Concurrent Cleanup for Next Mark
[0.705s][info   ][gc,marking  ] GC(105) Concurrent Cleanup for Next Mark 0.047ms
[0.705s][info   ][gc          ] GC(105) Concurrent Undo Cycle 0.123ms
[0.709s][info   ][gc,start    ] GC(106) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.709s][info   ][gc,task     ] GC(106) Using 8 workers of 8 for evacuation
[0.711s][info   ][gc,phases   ] GC(106)   Pre Evacuate Collection Set: 0.2ms
[0.711s][info   ][gc,phases   ] GC(106)   Merge Heap Roots: 0.1ms
[0.711s][info   ][gc,phases   ] GC(106)   Evacuate Collection Set: 0.9ms
[0.711s][info   ][gc,phases   ] GC(106)   Post Evacuate Collection Set: 0.4ms
[0.711s][info   ][gc,phases   ] GC(106)   Other: 0.1ms
[0.711s][info   ][gc,heap     ] GC(106) Eden regions: 45->0(90)
[0.711s][info   ][gc,heap     ] GC(106) Survivor regions: 17->15(15)
[0.711s][info   ][gc,heap     ] GC(106) Old regions: 535->552
[0.711s][info   ][gc,heap     ] GC(106) Archive regions: 2->2
[0.711s][info   ][gc,heap     ] GC(106) Humongous regions: 202->178
[0.711s][info   ][gc,metaspace] GC(106) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.711s][info   ][gc          ] GC(106) Pause Young (Concurrent Start) (G1 Humongous Allocation) 799M->745M(1024M) 1.878ms
[0.711s][info   ][gc,cpu      ] GC(106) User=0.00s Sys=0.00s Real=0.01s
[0.711s][info   ][gc          ] GC(107) Concurrent Undo Cycle
[0.711s][info   ][gc,marking  ] GC(107) Concurrent Cleanup for Next Mark
[0.711s][info   ][gc,marking  ] GC(107) Concurrent Cleanup for Next Mark 0.062ms
[0.711s][info   ][gc          ] GC(107) Concurrent Undo Cycle 0.104ms
[0.714s][info   ][gc,start    ] GC(108) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.714s][info   ][gc,task     ] GC(108) Using 8 workers of 8 for evacuation
[0.716s][info   ][gc,phases   ] GC(108)   Pre Evacuate Collection Set: 0.2ms
[0.716s][info   ][gc,phases   ] GC(108)   Merge Heap Roots: 0.1ms
[0.716s][info   ][gc,phases   ] GC(108)   Evacuate Collection Set: 1.0ms
[0.716s][info   ][gc,phases   ] GC(108)   Post Evacuate Collection Set: 0.3ms
[0.716s][info   ][gc,phases   ] GC(108)   Other: 0.1ms
[0.716s][info   ][gc,heap     ] GC(108) Eden regions: 39->0(85)
[0.716s][info   ][gc,heap     ] GC(108) Survivor regions: 15->14(14)
[0.716s][info   ][gc,heap     ] GC(108) Old regions: 552->569
[0.716s][info   ][gc,heap     ] GC(108) Archive regions: 2->2
[0.716s][info   ][gc,heap     ] GC(108) Humongous regions: 192->171
[0.716s][info   ][gc,metaspace] GC(108) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.716s][info   ][gc          ] GC(108) Pause Young (Concurrent Start) (G1 Humongous Allocation) 798M->754M(1024M) 1.897ms
[0.716s][info   ][gc,cpu      ] GC(108) User=0.01s Sys=0.00s Real=0.00s
[0.716s][info   ][gc          ] GC(109) Concurrent Undo Cycle
[0.716s][info   ][gc,marking  ] GC(109) Concurrent Cleanup for Next Mark
[0.716s][info   ][gc,marking  ] GC(109) Concurrent Cleanup for Next Mark 0.080ms
[0.716s][info   ][gc          ] GC(109) Concurrent Undo Cycle 0.328ms
[0.718s][info   ][gc,start    ] GC(110) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.718s][info   ][gc,task     ] GC(110) Using 8 workers of 8 for evacuation
[0.719s][info   ][gc,phases   ] GC(110)   Pre Evacuate Collection Set: 0.3ms
[0.719s][info   ][gc,phases   ] GC(110)   Merge Heap Roots: 0.1ms
[0.719s][info   ][gc,phases   ] GC(110)   Evacuate Collection Set: 0.6ms
[0.719s][info   ][gc,phases   ] GC(110)   Post Evacuate Collection Set: 0.2ms
[0.719s][info   ][gc,phases   ] GC(110)   Other: 0.1ms
[0.719s][info   ][gc,heap     ] GC(110) Eden regions: 20->0(81)
[0.719s][info   ][gc,heap     ] GC(110) Survivor regions: 14->8(13)
[0.719s][info   ][gc,heap     ] GC(110) Old regions: 569->583
[0.719s][info   ][gc,heap     ] GC(110) Archive regions: 2->2
[0.719s][info   ][gc,heap     ] GC(110) Humongous regions: 182->173
[0.719s][info   ][gc,metaspace] GC(110) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.719s][info   ][gc          ] GC(110) Pause Young (Concurrent Start) (G1 Humongous Allocation) 785M->764M(1024M) 1.404ms
[0.719s][info   ][gc,cpu      ] GC(110) User=0.00s Sys=0.00s Real=0.00s
[0.719s][info   ][gc          ] GC(111) Concurrent Mark Cycle
[0.719s][info   ][gc,marking  ] GC(111) Concurrent Clear Claimed Marks
[0.719s][info   ][gc,marking  ] GC(111) Concurrent Clear Claimed Marks 0.013ms
[0.719s][info   ][gc,marking  ] GC(111) Concurrent Scan Root Regions
[0.719s][info   ][gc,marking  ] GC(111) Concurrent Scan Root Regions 0.061ms
[0.719s][info   ][gc,marking  ] GC(111) Concurrent Mark
[0.720s][info   ][gc,marking  ] GC(111) Concurrent Mark From Roots
[0.720s][info   ][gc,task     ] GC(111) Using 2 workers of 2 for marking
[0.721s][info   ][gc,marking  ] GC(111) Concurrent Mark From Roots 1.663ms
[0.721s][info   ][gc,marking  ] GC(111) Concurrent Preclean
[0.721s][info   ][gc,marking  ] GC(111) Concurrent Preclean 0.010ms
[0.721s][info   ][gc,start    ] GC(111) Pause Remark
[0.722s][info   ][gc          ] GC(111) Pause Remark 796M->733M(1024M) 0.479ms
[0.722s][info   ][gc,cpu      ] GC(111) User=0.00s Sys=0.00s Real=0.00s
[0.722s][info   ][gc,marking  ] GC(111) Concurrent Mark 2.338ms
[0.722s][info   ][gc,marking  ] GC(111) Concurrent Rebuild Remembered Sets
[0.723s][info   ][gc,marking  ] GC(111) Concurrent Rebuild Remembered Sets 1.199ms
[0.723s][info   ][gc,start    ] GC(111) Pause Cleanup
[0.724s][info   ][gc          ] GC(111) Pause Cleanup 768M->768M(1024M) 0.359ms
[0.724s][info   ][gc,cpu      ] GC(111) User=0.00s Sys=0.00s Real=0.00s
[0.724s][info   ][gc,marking  ] GC(111) Concurrent Cleanup for Next Mark
[0.724s][info   ][gc,marking  ] GC(111) Concurrent Cleanup for Next Mark 0.760ms
[0.724s][info   ][gc          ] GC(111) Concurrent Mark Cycle 5.188ms
[0.728s][info   ][gc,start    ] GC(112) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.728s][info   ][gc,task     ] GC(112) Using 8 workers of 8 for evacuation
[0.730s][info   ][gc,phases   ] GC(112)   Pre Evacuate Collection Set: 0.2ms
[0.730s][info   ][gc,phases   ] GC(112)   Merge Heap Roots: 0.1ms
[0.730s][info   ][gc,phases   ] GC(112)   Evacuate Collection Set: 1.2ms
[0.730s][info   ][gc,phases   ] GC(112)   Post Evacuate Collection Set: 0.2ms
[0.730s][info   ][gc,phases   ] GC(112)   Other: 0.0ms
[0.730s][info   ][gc,heap     ] GC(112) Eden regions: 81->0(39)
[0.730s][info   ][gc,heap     ] GC(112) Survivor regions: 8->12(12)
[0.730s][info   ][gc,heap     ] GC(112) Old regions: 520->550
[0.730s][info   ][gc,heap     ] GC(112) Archive regions: 2->2
[0.730s][info   ][gc,heap     ] GC(112) Humongous regions: 230->171
[0.730s][info   ][gc,metaspace] GC(112) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.730s][info   ][gc          ] GC(112) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 839M->733M(1024M) 2.000ms
[0.730s][info   ][gc,cpu      ] GC(112) User=0.01s Sys=0.00s Real=0.00s
[0.733s][info   ][gc,start    ] GC(113) Pause Young (Mixed) (G1 Evacuation Pause)
[0.733s][info   ][gc,task     ] GC(113) Using 8 workers of 8 for evacuation
[0.734s][info   ][gc,phases   ] GC(113)   Pre Evacuate Collection Set: 0.2ms
[0.735s][info   ][gc,phases   ] GC(113)   Merge Heap Roots: 0.1ms
[0.735s][info   ][gc,phases   ] GC(113)   Evacuate Collection Set: 1.1ms
[0.735s][info   ][gc,phases   ] GC(113)   Post Evacuate Collection Set: 0.3ms
[0.735s][info   ][gc,phases   ] GC(113)   Other: 0.0ms
[0.735s][info   ][gc,heap     ] GC(113) Eden regions: 39->0(44)
[0.735s][info   ][gc,heap     ] GC(113) Survivor regions: 12->7(7)
[0.735s][info   ][gc,heap     ] GC(113) Old regions: 550->476
[0.735s][info   ][gc,heap     ] GC(113) Archive regions: 2->2
[0.735s][info   ][gc,heap     ] GC(113) Humongous regions: 191->171
[0.735s][info   ][gc,metaspace] GC(113) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.735s][info   ][gc          ] GC(113) Pause Young (Mixed) (G1 Evacuation Pause) 792M->654M(1024M) 1.961ms
[0.735s][info   ][gc,cpu      ] GC(113) User=0.00s Sys=0.00s Real=0.00s
[0.738s][info   ][gc,start    ] GC(114) Pause Young (Mixed) (G1 Evacuation Pause)
[0.738s][info   ][gc,task     ] GC(114) Using 8 workers of 8 for evacuation
[0.740s][info   ][gc,phases   ] GC(114)   Pre Evacuate Collection Set: 0.1ms
[0.740s][info   ][gc,phases   ] GC(114)   Merge Heap Roots: 0.1ms
[0.740s][info   ][gc,phases   ] GC(114)   Evacuate Collection Set: 1.1ms
[0.740s][info   ][gc,phases   ] GC(114)   Post Evacuate Collection Set: 0.2ms
[0.740s][info   ][gc,phases   ] GC(114)   Other: 0.0ms
[0.740s][info   ][gc,heap     ] GC(114) Eden regions: 44->0(44)
[0.740s][info   ][gc,heap     ] GC(114) Survivor regions: 7->7(7)
[0.740s][info   ][gc,heap     ] GC(114) Old regions: 476->408
[0.740s][info   ][gc,heap     ] GC(114) Archive regions: 2->2
[0.740s][info   ][gc,heap     ] GC(114) Humongous regions: 202->170
[0.740s][info   ][gc,metaspace] GC(114) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.740s][info   ][gc          ] GC(114) Pause Young (Mixed) (G1 Evacuation Pause) 729M->585M(1024M) 1.744ms
[0.740s][info   ][gc,cpu      ] GC(114) User=0.00s Sys=0.00s Real=0.00s
[0.743s][info   ][gc,start    ] GC(115) Pause Young (Mixed) (G1 Evacuation Pause)
[0.743s][info   ][gc,task     ] GC(115) Using 8 workers of 8 for evacuation
[0.745s][info   ][gc,phases   ] GC(115)   Pre Evacuate Collection Set: 0.1ms
[0.745s][info   ][gc,phases   ] GC(115)   Merge Heap Roots: 0.1ms
[0.745s][info   ][gc,phases   ] GC(115)   Evacuate Collection Set: 1.5ms
[0.745s][info   ][gc,phases   ] GC(115)   Post Evacuate Collection Set: 0.2ms
[0.745s][info   ][gc,phases   ] GC(115)   Other: 0.0ms
[0.745s][info   ][gc,heap     ] GC(115) Eden regions: 44->0(44)
[0.745s][info   ][gc,heap     ] GC(115) Survivor regions: 7->7(7)
[0.745s][info   ][gc,heap     ] GC(115) Old regions: 408->354
[0.745s][info   ][gc,heap     ] GC(115) Archive regions: 2->2
[0.745s][info   ][gc,heap     ] GC(115) Humongous regions: 191->164
[0.745s][info   ][gc,metaspace] GC(115) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.745s][info   ][gc          ] GC(115) Pause Young (Mixed) (G1 Evacuation Pause) 650M->525M(1024M) 2.118ms
[0.745s][info   ][gc,cpu      ] GC(115) User=0.00s Sys=0.01s Real=0.00s
[0.748s][info   ][gc,start    ] GC(116) Pause Young (Mixed) (G1 Evacuation Pause)
[0.749s][info   ][gc,task     ] GC(116) Using 8 workers of 8 for evacuation
[0.750s][info   ][gc,phases   ] GC(116)   Pre Evacuate Collection Set: 0.2ms
[0.750s][info   ][gc,phases   ] GC(116)   Merge Heap Roots: 0.0ms
[0.750s][info   ][gc,phases   ] GC(116)   Evacuate Collection Set: 0.6ms
[0.750s][info   ][gc,phases   ] GC(116)   Post Evacuate Collection Set: 0.2ms
[0.750s][info   ][gc,phases   ] GC(116)   Other: 0.0ms
[0.750s][info   ][gc,heap     ] GC(116) Eden regions: 44->0(188)
[0.750s][info   ][gc,heap     ] GC(116) Survivor regions: 7->7(7)
[0.750s][info   ][gc,heap     ] GC(116) Old regions: 354->367
[0.750s][info   ][gc,heap     ] GC(116) Archive regions: 2->2
[0.750s][info   ][gc,heap     ] GC(116) Humongous regions: 194->175
[0.750s][info   ][gc,metaspace] GC(116) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.750s][info   ][gc          ] GC(116) Pause Young (Mixed) (G1 Evacuation Pause) 599M->549M(1024M) 1.240ms
[0.750s][info   ][gc,cpu      ] GC(116) User=0.00s Sys=0.00s Real=0.00s
[0.767s][info   ][gc,start    ] GC(117) Pause Young (Normal) (G1 Evacuation Pause)
[0.767s][info   ][gc,task     ] GC(117) Using 8 workers of 8 for evacuation
[0.769s][info   ][gc,phases   ] GC(117)   Pre Evacuate Collection Set: 0.2ms
[0.769s][info   ][gc,phases   ] GC(117)   Merge Heap Roots: 0.1ms
[0.769s][info   ][gc,phases   ] GC(117)   Evacuate Collection Set: 1.9ms
[0.769s][info   ][gc,phases   ] GC(117)   Post Evacuate Collection Set: 0.2ms
[0.769s][info   ][gc,phases   ] GC(117)   Other: 0.1ms
[0.769s][info   ][gc,heap     ] GC(117) Eden regions: 188->0(157)
[0.769s][info   ][gc,heap     ] GC(117) Survivor regions: 7->25(25)
[0.769s][info   ][gc,heap     ] GC(117) Old regions: 367->417
[0.769s][info   ][gc,heap     ] GC(117) Archive regions: 2->2
[0.769s][info   ][gc,heap     ] GC(117) Humongous regions: 280->170
[0.769s][info   ][gc,metaspace] GC(117) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.770s][info   ][gc          ] GC(117) Pause Young (Normal) (G1 Evacuation Pause) 842M->612M(1024M) 2.656ms
[0.770s][info   ][gc,cpu      ] GC(117) User=0.00s Sys=0.00s Real=0.00s
[0.780s][info   ][gc,start    ] GC(118) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.780s][info   ][gc,task     ] GC(118) Using 8 workers of 8 for evacuation
[0.782s][info   ][gc,phases   ] GC(118)   Pre Evacuate Collection Set: 0.1ms
[0.782s][info   ][gc,phases   ] GC(118)   Merge Heap Roots: 0.1ms
[0.782s][info   ][gc,phases   ] GC(118)   Evacuate Collection Set: 1.4ms
[0.782s][info   ][gc,phases   ] GC(118)   Post Evacuate Collection Set: 0.2ms
[0.782s][info   ][gc,phases   ] GC(118)   Other: 0.1ms
[0.782s][info   ][gc,heap     ] GC(118) Eden regions: 127->0(139)
[0.782s][info   ][gc,heap     ] GC(118) Survivor regions: 25->23(23)
[0.782s][info   ][gc,heap     ] GC(118) Old regions: 417->458
[0.782s][info   ][gc,heap     ] GC(118) Archive regions: 2->2
[0.782s][info   ][gc,heap     ] GC(118) Humongous regions: 252->168
[0.782s][info   ][gc,metaspace] GC(118) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.782s][info   ][gc          ] GC(118) Pause Young (Concurrent Start) (G1 Humongous Allocation) 821M->649M(1024M) 1.987ms
[0.782s][info   ][gc,cpu      ] GC(118) User=0.01s Sys=0.01s Real=0.01s
[0.782s][info   ][gc          ] GC(119) Concurrent Undo Cycle
[0.782s][info   ][gc,marking  ] GC(119) Concurrent Cleanup for Next Mark
[0.782s][info   ][gc,marking  ] GC(119) Concurrent Cleanup for Next Mark 0.032ms
[0.782s][info   ][gc          ] GC(119) Concurrent Undo Cycle 0.085ms
[0.790s][info   ][gc,start    ] GC(120) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.790s][info   ][gc,task     ] GC(120) Using 8 workers of 8 for evacuation
[0.792s][info   ][gc,phases   ] GC(120)   Pre Evacuate Collection Set: 0.3ms
[0.792s][info   ][gc,phases   ] GC(120)   Merge Heap Roots: 0.0ms
[0.792s][info   ][gc,phases   ] GC(120)   Evacuate Collection Set: 1.1ms
[0.792s][info   ][gc,phases   ] GC(120)   Post Evacuate Collection Set: 0.3ms
[0.792s][info   ][gc,phases   ] GC(120)   Other: 0.1ms
[0.792s][info   ][gc,heap     ] GC(120) Eden regions: 91->0(125)
[0.792s][info   ][gc,heap     ] GC(120) Survivor regions: 23->21(21)
[0.792s][info   ][gc,heap     ] GC(120) Old regions: 458->489
[0.792s][info   ][gc,heap     ] GC(120) Archive regions: 2->2
[0.792s][info   ][gc,heap     ] GC(120) Humongous regions: 232->172
[0.792s][info   ][gc,metaspace] GC(120) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.792s][info   ][gc          ] GC(120) Pause Young (Concurrent Start) (G1 Humongous Allocation) 803M->682M(1024M) 1.953ms
[0.792s][info   ][gc,cpu      ] GC(120) User=0.00s Sys=0.00s Real=0.01s
[0.792s][info   ][gc          ] GC(121) Concurrent Undo Cycle
[0.792s][info   ][gc,marking  ] GC(121) Concurrent Cleanup for Next Mark
[0.792s][info   ][gc,marking  ] GC(121) Concurrent Cleanup for Next Mark 0.040ms
[0.792s][info   ][gc          ] GC(121) Concurrent Undo Cycle 0.101ms
[0.798s][info   ][gc,start    ] GC(122) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.798s][info   ][gc,task     ] GC(122) Using 8 workers of 8 for evacuation
[0.799s][info   ][gc,phases   ] GC(122)   Pre Evacuate Collection Set: 0.1ms
[0.799s][info   ][gc,phases   ] GC(122)   Merge Heap Roots: 0.0ms
[0.799s][info   ][gc,phases   ] GC(122)   Evacuate Collection Set: 1.0ms
[0.799s][info   ][gc,phases   ] GC(122)   Post Evacuate Collection Set: 0.2ms
[0.799s][info   ][gc,phases   ] GC(122)   Other: 0.1ms
[0.799s][info   ][gc,heap     ] GC(122) Eden regions: 72->0(105)
[0.799s][info   ][gc,heap     ] GC(122) Survivor regions: 21->19(19)
[0.799s][info   ][gc,heap     ] GC(122) Old regions: 489->522
[0.799s][info   ][gc,heap     ] GC(122) Archive regions: 2->2
[0.799s][info   ][gc,heap     ] GC(122) Humongous regions: 217->169
[0.799s][info   ][gc,metaspace] GC(122) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.799s][info   ][gc          ] GC(122) Pause Young (Concurrent Start) (G1 Humongous Allocation) 799M->710M(1024M) 1.549ms
[0.799s][info   ][gc,cpu      ] GC(122) User=0.00s Sys=0.00s Real=0.00s
[0.799s][info   ][gc          ] GC(123) Concurrent Undo Cycle
[0.799s][info   ][gc,marking  ] GC(123) Concurrent Cleanup for Next Mark
[0.799s][info   ][gc,marking  ] GC(123) Concurrent Cleanup for Next Mark 0.085ms
[0.799s][info   ][gc          ] GC(123) Concurrent Undo Cycle 0.121ms
[0.804s][info   ][gc,start    ] GC(124) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.804s][info   ][gc,task     ] GC(124) Using 8 workers of 8 for evacuation
[0.805s][info   ][gc,phases   ] GC(124)   Pre Evacuate Collection Set: 0.1ms
[0.805s][info   ][gc,phases   ] GC(124)   Merge Heap Roots: 0.1ms
[0.805s][info   ][gc,phases   ] GC(124)   Evacuate Collection Set: 0.8ms
[0.805s][info   ][gc,phases   ] GC(124)   Post Evacuate Collection Set: 0.2ms
[0.805s][info   ][gc,phases   ] GC(124)   Other: 0.1ms
[0.805s][info   ][gc,heap     ] GC(124) Eden regions: 57->0(96)
[0.805s][info   ][gc,heap     ] GC(124) Survivor regions: 19->16(16)
[0.805s][info   ][gc,heap     ] GC(124) Old regions: 522->545
[0.805s][info   ][gc,heap     ] GC(124) Archive regions: 2->2
[0.805s][info   ][gc,heap     ] GC(124) Humongous regions: 206->173
[0.805s][info   ][gc,metaspace] GC(124) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.805s][info   ][gc          ] GC(124) Pause Young (Concurrent Start) (G1 Humongous Allocation) 804M->734M(1024M) 1.445ms
[0.805s][info   ][gc,cpu      ] GC(124) User=0.00s Sys=0.00s Real=0.00s
[0.805s][info   ][gc          ] GC(125) Concurrent Undo Cycle
[0.805s][info   ][gc,marking  ] GC(125) Concurrent Cleanup for Next Mark
[0.805s][info   ][gc,marking  ] GC(125) Concurrent Cleanup for Next Mark 0.053ms
[0.805s][info   ][gc          ] GC(125) Concurrent Undo Cycle 0.139ms
[0.809s][info   ][gc,start    ] GC(126) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.809s][info   ][gc,task     ] GC(126) Using 8 workers of 8 for evacuation
[0.810s][info   ][gc,phases   ] GC(126)   Pre Evacuate Collection Set: 0.1ms
[0.810s][info   ][gc,phases   ] GC(126)   Merge Heap Roots: 0.1ms
[0.810s][info   ][gc,phases   ] GC(126)   Evacuate Collection Set: 0.8ms
[0.810s][info   ][gc,phases   ] GC(126)   Post Evacuate Collection Set: 0.1ms
[0.810s][info   ][gc,phases   ] GC(126)   Other: 0.1ms
[0.810s][info   ][gc,heap     ] GC(126) Eden regions: 55->0(84)
[0.810s][info   ][gc,heap     ] GC(126) Survivor regions: 16->14(14)
[0.810s][info   ][gc,heap     ] GC(126) Old regions: 545->569
[0.810s][info   ][gc,heap     ] GC(126) Archive regions: 2->2
[0.810s][info   ][gc,heap     ] GC(126) Humongous regions: 195->166
[0.810s][info   ][gc,metaspace] GC(126) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.810s][info   ][gc          ] GC(126) Pause Young (Concurrent Start) (G1 Humongous Allocation) 811M->749M(1024M) 1.363ms
[0.810s][info   ][gc,cpu      ] GC(126) User=0.01s Sys=0.00s Real=0.00s
[0.810s][info   ][gc          ] GC(127) Concurrent Undo Cycle
[0.810s][info   ][gc,marking  ] GC(127) Concurrent Cleanup for Next Mark
[0.811s][info   ][gc,marking  ] GC(127) Concurrent Cleanup for Next Mark 0.049ms
[0.811s][info   ][gc          ] GC(127) Concurrent Undo Cycle 0.098ms
[0.813s][info   ][gc,start    ] GC(128) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.813s][info   ][gc,task     ] GC(128) Using 8 workers of 8 for evacuation
[0.814s][info   ][gc,phases   ] GC(128)   Pre Evacuate Collection Set: 0.1ms
[0.814s][info   ][gc,phases   ] GC(128)   Merge Heap Roots: 0.1ms
[0.814s][info   ][gc,phases   ] GC(128)   Evacuate Collection Set: 0.6ms
[0.814s][info   ][gc,phases   ] GC(128)   Post Evacuate Collection Set: 0.2ms
[0.814s][info   ][gc,phases   ] GC(128)   Other: 0.1ms
[0.814s][info   ][gc,heap     ] GC(128) Eden regions: 29->0(75)
[0.814s][info   ][gc,heap     ] GC(128) Survivor regions: 14->13(13)
[0.814s][info   ][gc,heap     ] GC(128) Old regions: 569->583
[0.814s][info   ][gc,heap     ] GC(128) Archive regions: 2->2
[0.814s][info   ][gc,heap     ] GC(128) Humongous regions: 185->165
[0.814s][info   ][gc,metaspace] GC(128) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.814s][info   ][gc          ] GC(128) Pause Young (Concurrent Start) (G1 Humongous Allocation) 796M->761M(1024M) 1.205ms
[0.814s][info   ][gc,cpu      ] GC(128) User=0.00s Sys=0.00s Real=0.00s
[0.814s][info   ][gc          ] GC(129) Concurrent Undo Cycle
[0.814s][info   ][gc,marking  ] GC(129) Concurrent Cleanup for Next Mark
[0.814s][info   ][gc,marking  ] GC(129) Concurrent Cleanup for Next Mark 0.046ms
[0.814s][info   ][gc          ] GC(129) Concurrent Undo Cycle 0.078ms
[0.817s][info   ][gc,start    ] GC(130) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.817s][info   ][gc,task     ] GC(130) Using 8 workers of 8 for evacuation
[0.818s][info   ][gc,phases   ] GC(130)   Pre Evacuate Collection Set: 0.1ms
[0.818s][info   ][gc,phases   ] GC(130)   Merge Heap Roots: 0.0ms
[0.818s][info   ][gc,phases   ] GC(130)   Evacuate Collection Set: 0.6ms
[0.818s][info   ][gc,phases   ] GC(130)   Post Evacuate Collection Set: 0.2ms
[0.818s][info   ][gc,phases   ] GC(130)   Other: 0.1ms
[0.818s][info   ][gc,heap     ] GC(130) Eden regions: 32->0(74)
[0.818s][info   ][gc,heap     ] GC(130) Survivor regions: 13->11(11)
[0.818s][info   ][gc,heap     ] GC(130) Old regions: 583->596
[0.818s][info   ][gc,heap     ] GC(130) Archive regions: 2->2
[0.818s][info   ][gc,heap     ] GC(130) Humongous regions: 182->163
[0.818s][info   ][gc,metaspace] GC(130) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.818s][info   ][gc          ] GC(130) Pause Young (Concurrent Start) (G1 Humongous Allocation) 809M->770M(1024M) 1.156ms
[0.818s][info   ][gc,cpu      ] GC(130) User=0.00s Sys=0.01s Real=0.00s
[0.818s][info   ][gc          ] GC(131) Concurrent Undo Cycle
[0.818s][info   ][gc,marking  ] GC(131) Concurrent Cleanup for Next Mark
[0.818s][info   ][gc,marking  ] GC(131) Concurrent Cleanup for Next Mark 0.047ms
[0.818s][info   ][gc          ] GC(131) Concurrent Undo Cycle 0.095ms
[0.819s][info   ][gc,start    ] GC(132) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.819s][info   ][gc,task     ] GC(132) Using 8 workers of 8 for evacuation
[0.820s][info   ][gc,phases   ] GC(132)   Pre Evacuate Collection Set: 0.1ms
[0.820s][info   ][gc,phases   ] GC(132)   Merge Heap Roots: 0.0ms
[0.820s][info   ][gc,phases   ] GC(132)   Evacuate Collection Set: 0.5ms
[0.820s][info   ][gc,phases   ] GC(132)   Post Evacuate Collection Set: 0.2ms
[0.820s][info   ][gc,phases   ] GC(132)   Other: 0.0ms
[0.820s][info   ][gc,heap     ] GC(132) Eden regions: 13->0(71)
[0.820s][info   ][gc,heap     ] GC(132) Survivor regions: 11->6(11)
[0.820s][info   ][gc,heap     ] GC(132) Old regions: 596->606
[0.820s][info   ][gc,heap     ] GC(132) Archive regions: 2->2
[0.820s][info   ][gc,heap     ] GC(132) Humongous regions: 173->171
[0.820s][info   ][gc,metaspace] GC(132) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.820s][info   ][gc          ] GC(132) Pause Young (Concurrent Start) (G1 Humongous Allocation) 793M->782M(1024M) 0.960ms
[0.820s][info   ][gc,cpu      ] GC(132) User=0.00s Sys=0.00s Real=0.00s
[0.820s][info   ][gc          ] GC(133) Concurrent Mark Cycle
[0.820s][info   ][gc,marking  ] GC(133) Concurrent Clear Claimed Marks
[0.820s][info   ][gc,marking  ] GC(133) Concurrent Clear Claimed Marks 0.008ms
[0.820s][info   ][gc,marking  ] GC(133) Concurrent Scan Root Regions
[0.820s][info   ][gc,marking  ] GC(133) Concurrent Scan Root Regions 0.049ms
[0.820s][info   ][gc,marking  ] GC(133) Concurrent Mark
[0.820s][info   ][gc,marking  ] GC(133) Concurrent Mark From Roots
[0.820s][info   ][gc,task     ] GC(133) Using 2 workers of 2 for marking
[0.821s][info   ][gc,marking  ] GC(133) Concurrent Mark From Roots 0.993ms
[0.821s][info   ][gc,marking  ] GC(133) Concurrent Preclean
[0.821s][info   ][gc,marking  ] GC(133) Concurrent Preclean 0.009ms
[0.821s][info   ][gc,start    ] GC(133) Pause Remark
[0.822s][info   ][gc          ] GC(133) Pause Remark 802M->725M(1024M) 0.449ms
[0.822s][info   ][gc,cpu      ] GC(133) User=0.00s Sys=0.00s Real=0.00s
[0.822s][info   ][gc,marking  ] GC(133) Concurrent Mark 1.735ms
[0.822s][info   ][gc,marking  ] GC(133) Concurrent Rebuild Remembered Sets
[0.823s][info   ][gc,marking  ] GC(133) Concurrent Rebuild Remembered Sets 1.137ms
[0.823s][info   ][gc,start    ] GC(133) Pause Cleanup
[0.823s][info   ][gc          ] GC(133) Pause Cleanup 752M->752M(1024M) 0.279ms
[0.823s][info   ][gc,cpu      ] GC(133) User=0.00s Sys=0.00s Real=0.00s
[0.823s][info   ][gc,marking  ] GC(133) Concurrent Cleanup for Next Mark
[0.824s][info   ][gc,marking  ] GC(133) Concurrent Cleanup for Next Mark 0.742ms
[0.824s][info   ][gc          ] GC(133) Concurrent Mark Cycle 4.280ms
[0.826s][info   ][gc,start    ] GC(134) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.826s][info   ][gc,task     ] GC(134) Using 8 workers of 8 for evacuation
[0.828s][info   ][gc,phases   ] GC(134)   Pre Evacuate Collection Set: 0.1ms
[0.828s][info   ][gc,phases   ] GC(134)   Merge Heap Roots: 0.1ms
[0.828s][info   ][gc,phases   ] GC(134)   Evacuate Collection Set: 1.1ms
[0.828s][info   ][gc,phases   ] GC(134)   Post Evacuate Collection Set: 0.3ms
[0.828s][info   ][gc,phases   ] GC(134)   Other: 0.0ms
[0.828s][info   ][gc,heap     ] GC(134) Eden regions: 71->0(41)
[0.828s][info   ][gc,heap     ] GC(134) Survivor regions: 6->10(10)
[0.828s][info   ][gc,heap     ] GC(134) Old regions: 529->552
[0.828s][info   ][gc,heap     ] GC(134) Archive regions: 2->2
[0.828s][info   ][gc,heap     ] GC(134) Humongous regions: 218->167
[0.828s][info   ][gc,metaspace] GC(134) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.828s][info   ][gc          ] GC(134) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 823M->729M(1024M) 1.812ms
[0.828s][info   ][gc,cpu      ] GC(134) User=0.01s Sys=0.00s Real=0.00s
[0.832s][info   ][gc,start    ] GC(135) Pause Young (Mixed) (G1 Evacuation Pause)
[0.832s][info   ][gc,task     ] GC(135) Using 8 workers of 8 for evacuation
[0.833s][info   ][gc,phases   ] GC(135)   Pre Evacuate Collection Set: 0.2ms
[0.833s][info   ][gc,phases   ] GC(135)   Merge Heap Roots: 0.1ms
[0.833s][info   ][gc,phases   ] GC(135)   Evacuate Collection Set: 0.8ms
[0.833s][info   ][gc,phases   ] GC(135)   Post Evacuate Collection Set: 0.2ms
[0.833s][info   ][gc,phases   ] GC(135)   Other: 0.0ms
[0.833s][info   ][gc,heap     ] GC(135) Eden regions: 41->0(44)
[0.833s][info   ][gc,heap     ] GC(135) Survivor regions: 10->7(7)
[0.833s][info   ][gc,heap     ] GC(135) Old regions: 552->474
[0.833s][info   ][gc,heap     ] GC(135) Archive regions: 2->2
[0.833s][info   ][gc,heap     ] GC(135) Humongous regions: 194->168
[0.833s][info   ][gc,metaspace] GC(135) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.833s][info   ][gc          ] GC(135) Pause Young (Mixed) (G1 Evacuation Pause) 797M->649M(1024M) 1.592ms
[0.833s][info   ][gc,cpu      ] GC(135) User=0.00s Sys=0.00s Real=0.00s
[0.836s][info   ][gc,start    ] GC(136) Pause Young (Mixed) (G1 Evacuation Pause)
[0.837s][info   ][gc,task     ] GC(136) Using 8 workers of 8 for evacuation
[0.838s][info   ][gc,phases   ] GC(136)   Pre Evacuate Collection Set: 0.1ms
[0.838s][info   ][gc,phases   ] GC(136)   Merge Heap Roots: 0.1ms
[0.838s][info   ][gc,phases   ] GC(136)   Evacuate Collection Set: 1.1ms
[0.838s][info   ][gc,phases   ] GC(136)   Post Evacuate Collection Set: 0.2ms
[0.838s][info   ][gc,phases   ] GC(136)   Other: 0.1ms
[0.838s][info   ][gc,heap     ] GC(136) Eden regions: 44->0(44)
[0.838s][info   ][gc,heap     ] GC(136) Survivor regions: 7->7(7)
[0.838s][info   ][gc,heap     ] GC(136) Old regions: 474->404
[0.838s][info   ][gc,heap     ] GC(136) Archive regions: 2->2
[0.838s][info   ][gc,heap     ] GC(136) Humongous regions: 190->166
[0.838s][info   ][gc,metaspace] GC(136) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.838s][info   ][gc          ] GC(136) Pause Young (Mixed) (G1 Evacuation Pause) 715M->577M(1024M) 1.787ms
[0.838s][info   ][gc,cpu      ] GC(136) User=0.00s Sys=0.00s Real=0.00s
[0.841s][info   ][gc,start    ] GC(137) Pause Young (Mixed) (G1 Evacuation Pause)
[0.841s][info   ][gc,task     ] GC(137) Using 8 workers of 8 for evacuation
[0.843s][info   ][gc,phases   ] GC(137)   Pre Evacuate Collection Set: 0.1ms
[0.843s][info   ][gc,phases   ] GC(137)   Merge Heap Roots: 0.1ms
[0.843s][info   ][gc,phases   ] GC(137)   Evacuate Collection Set: 1.1ms
[0.843s][info   ][gc,phases   ] GC(137)   Post Evacuate Collection Set: 0.2ms
[0.843s][info   ][gc,phases   ] GC(137)   Other: 0.0ms
[0.843s][info   ][gc,heap     ] GC(137) Eden regions: 44->0(44)
[0.843s][info   ][gc,heap     ] GC(137) Survivor regions: 7->7(7)
[0.843s][info   ][gc,heap     ] GC(137) Old regions: 404->347
[0.843s][info   ][gc,heap     ] GC(137) Archive regions: 2->2
[0.843s][info   ][gc,heap     ] GC(137) Humongous regions: 186->167
[0.843s][info   ][gc,metaspace] GC(137) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.843s][info   ][gc          ] GC(137) Pause Young (Mixed) (G1 Evacuation Pause) 641M->521M(1024M) 1.755ms
[0.843s][info   ][gc,cpu      ] GC(137) User=0.00s Sys=0.01s Real=0.00s
[0.847s][info   ][gc,start    ] GC(138) Pause Young (Mixed) (G1 Evacuation Pause)
[0.847s][info   ][gc,task     ] GC(138) Using 8 workers of 8 for evacuation
[0.848s][info   ][gc,phases   ] GC(138)   Pre Evacuate Collection Set: 0.1ms
[0.848s][info   ][gc,phases   ] GC(138)   Merge Heap Roots: 0.1ms
[0.848s][info   ][gc,phases   ] GC(138)   Evacuate Collection Set: 0.7ms
[0.848s][info   ][gc,phases   ] GC(138)   Post Evacuate Collection Set: 0.2ms
[0.848s][info   ][gc,phases   ] GC(138)   Other: 0.0ms
[0.848s][info   ][gc,heap     ] GC(138) Eden regions: 44->0(176)
[0.848s][info   ][gc,heap     ] GC(138) Survivor regions: 7->7(7)
[0.848s][info   ][gc,heap     ] GC(138) Old regions: 347->360
[0.848s][info   ][gc,heap     ] GC(138) Archive regions: 2->2
[0.848s][info   ][gc,heap     ] GC(138) Humongous regions: 200->166
[0.848s][info   ][gc,metaspace] GC(138) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.848s][info   ][gc          ] GC(138) Pause Young (Mixed) (G1 Evacuation Pause) 598M->533M(1024M) 1.189ms
[0.848s][info   ][gc,cpu      ] GC(138) User=0.01s Sys=0.00s Real=0.00s
[0.862s][info   ][gc,start    ] GC(139) Pause Young (Normal) (G1 Evacuation Pause)
[0.862s][info   ][gc,task     ] GC(139) Using 8 workers of 8 for evacuation
[0.864s][info   ][gc,phases   ] GC(139)   Pre Evacuate Collection Set: 0.2ms
[0.864s][info   ][gc,phases   ] GC(139)   Merge Heap Roots: 0.1ms
[0.864s][info   ][gc,phases   ] GC(139)   Evacuate Collection Set: 1.6ms
[0.864s][info   ][gc,phases   ] GC(139)   Post Evacuate Collection Set: 0.2ms
[0.864s][info   ][gc,phases   ] GC(139)   Other: 0.0ms
[0.864s][info   ][gc,heap     ] GC(139) Eden regions: 176->0(145)
[0.864s][info   ][gc,heap     ] GC(139) Survivor regions: 7->23(23)
[0.864s][info   ][gc,heap     ] GC(139) Old regions: 360->407
[0.864s][info   ][gc,heap     ] GC(139) Archive regions: 2->2
[0.864s][info   ][gc,heap     ] GC(139) Humongous regions: 265->182
[0.864s][info   ][gc,metaspace] GC(139) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.864s][info   ][gc          ] GC(139) Pause Young (Normal) (G1 Evacuation Pause) 808M->612M(1024M) 2.272ms
[0.864s][info   ][gc,cpu      ] GC(139) User=0.01s Sys=0.00s Real=0.00s
[0.873s][info   ][gc,start    ] GC(140) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.873s][info   ][gc,task     ] GC(140) Using 8 workers of 8 for evacuation
[0.875s][info   ][gc,phases   ] GC(140)   Pre Evacuate Collection Set: 0.1ms
[0.875s][info   ][gc,phases   ] GC(140)   Merge Heap Roots: 0.0ms
[0.875s][info   ][gc,phases   ] GC(140)   Evacuate Collection Set: 1.1ms
[0.875s][info   ][gc,phases   ] GC(140)   Post Evacuate Collection Set: 0.2ms
[0.875s][info   ][gc,phases   ] GC(140)   Other: 0.1ms
[0.875s][info   ][gc,heap     ] GC(140) Eden regions: 119->0(123)
[0.875s][info   ][gc,heap     ] GC(140) Survivor regions: 23->21(21)
[0.875s][info   ][gc,heap     ] GC(140) Old regions: 407->445
[0.875s][info   ][gc,heap     ] GC(140) Archive regions: 2->2
[0.875s][info   ][gc,heap     ] GC(140) Humongous regions: 262->198
[0.875s][info   ][gc,metaspace] GC(140) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.875s][info   ][gc          ] GC(140) Pause Young (Concurrent Start) (G1 Humongous Allocation) 811M->664M(1024M) 1.672ms
[0.875s][info   ][gc,cpu      ] GC(140) User=0.00s Sys=0.00s Real=0.00s
[0.875s][info   ][gc          ] GC(141) Concurrent Undo Cycle
[0.875s][info   ][gc,marking  ] GC(141) Concurrent Cleanup for Next Mark
[0.875s][info   ][gc,marking  ] GC(141) Concurrent Cleanup for Next Mark 0.040ms
[0.875s][info   ][gc          ] GC(141) Concurrent Undo Cycle 0.073ms
[0.884s][info   ][gc,start    ] GC(142) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.884s][info   ][gc,task     ] GC(142) Using 8 workers of 8 for evacuation
[0.886s][info   ][gc,phases   ] GC(142)   Pre Evacuate Collection Set: 0.2ms
[0.886s][info   ][gc,phases   ] GC(142)   Merge Heap Roots: 0.1ms
[0.886s][info   ][gc,phases   ] GC(142)   Evacuate Collection Set: 1.3ms
[0.886s][info   ][gc,phases   ] GC(142)   Post Evacuate Collection Set: 0.2ms
[0.886s][info   ][gc,phases   ] GC(142)   Other: 0.1ms
[0.886s][info   ][gc,heap     ] GC(142) Eden regions: 123->0(111)
[0.886s][info   ][gc,heap     ] GC(142) Survivor regions: 21->18(18)
[0.886s][info   ][gc,heap     ] GC(142) Old regions: 445->491
[0.886s][info   ][gc,heap     ] GC(142) Archive regions: 2->2
[0.886s][info   ][gc,heap     ] GC(142) Humongous regions: 247->185
[0.886s][info   ][gc,metaspace] GC(142) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.886s][info   ][gc          ] GC(142) Pause Young (Concurrent Start) (G1 Humongous Allocation) 835M->694M(1024M) 2.034ms
[0.886s][info   ][gc,cpu      ] GC(142) User=0.01s Sys=0.00s Real=0.00s
[0.886s][info   ][gc          ] GC(143) Concurrent Undo Cycle
[0.886s][info   ][gc,marking  ] GC(143) Concurrent Cleanup for Next Mark
[0.886s][info   ][gc,marking  ] GC(143) Concurrent Cleanup for Next Mark 0.045ms
[0.886s][info   ][gc          ] GC(143) Concurrent Undo Cycle 0.075ms
[0.889s][info   ][gc,start    ] GC(144) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.889s][info   ][gc,task     ] GC(144) Using 8 workers of 8 for evacuation
[0.890s][info   ][gc,phases   ] GC(144)   Pre Evacuate Collection Set: 0.2ms
[0.890s][info   ][gc,phases   ] GC(144)   Merge Heap Roots: 0.1ms
[0.890s][info   ][gc,phases   ] GC(144)   Evacuate Collection Set: 0.8ms
[0.890s][info   ][gc,phases   ] GC(144)   Post Evacuate Collection Set: 0.1ms
[0.890s][info   ][gc,phases   ] GC(144)   Other: 0.1ms
[0.890s][info   ][gc,heap     ] GC(144) Eden regions: 47->0(105)
[0.890s][info   ][gc,heap     ] GC(144) Survivor regions: 18->17(17)
[0.890s][info   ][gc,heap     ] GC(144) Old regions: 491->510
[0.890s][info   ][gc,heap     ] GC(144) Archive regions: 2->2
[0.890s][info   ][gc,heap     ] GC(144) Humongous regions: 217->189
[0.890s][info   ][gc,metaspace] GC(144) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.891s][info   ][gc          ] GC(144) Pause Young (Concurrent Start) (G1 Humongous Allocation) 773M->716M(1024M) 1.367ms
[0.891s][info   ][gc,cpu      ] GC(144) User=0.00s Sys=0.00s Real=0.00s
[0.891s][info   ][gc          ] GC(145) Concurrent Undo Cycle
[0.891s][info   ][gc,marking  ] GC(145) Concurrent Cleanup for Next Mark
[0.891s][info   ][gc,marking  ] GC(145) Concurrent Cleanup for Next Mark 0.093ms
[0.891s][info   ][gc          ] GC(145) Concurrent Undo Cycle 0.126ms
[0.893s][info   ][gc,start    ] GC(146) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.893s][info   ][gc,task     ] GC(146) Using 8 workers of 8 for evacuation
[0.894s][info   ][gc,phases   ] GC(146)   Pre Evacuate Collection Set: 0.2ms
[0.894s][info   ][gc,phases   ] GC(146)   Merge Heap Roots: 0.1ms
[0.894s][info   ][gc,phases   ] GC(146)   Evacuate Collection Set: 0.8ms
[0.894s][info   ][gc,phases   ] GC(146)   Post Evacuate Collection Set: 0.2ms
[0.894s][info   ][gc,phases   ] GC(146)   Other: 0.1ms
[0.894s][info   ][gc,heap     ] GC(146) Eden regions: 26->0(95)
[0.894s][info   ][gc,heap     ] GC(146) Survivor regions: 17->9(16)
[0.894s][info   ][gc,heap     ] GC(146) Old regions: 510->526
[0.894s][info   ][gc,heap     ] GC(146) Archive regions: 2->2
[0.894s][info   ][gc,heap     ] GC(146) Humongous regions: 204->191
[0.894s][info   ][gc,metaspace] GC(146) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.894s][info   ][gc          ] GC(146) Pause Young (Concurrent Start) (G1 Humongous Allocation) 756M->726M(1024M) 1.508ms
[0.894s][info   ][gc,cpu      ] GC(146) User=0.00s Sys=0.01s Real=0.00s
[0.894s][info   ][gc          ] GC(147) Concurrent Mark Cycle
[0.894s][info   ][gc,marking  ] GC(147) Concurrent Clear Claimed Marks
[0.894s][info   ][gc,marking  ] GC(147) Concurrent Clear Claimed Marks 0.007ms
[0.894s][info   ][gc,marking  ] GC(147) Concurrent Scan Root Regions
[0.894s][info   ][gc,marking  ] GC(147) Concurrent Scan Root Regions 0.058ms
[0.894s][info   ][gc,marking  ] GC(147) Concurrent Mark
[0.894s][info   ][gc,marking  ] GC(147) Concurrent Mark From Roots
[0.894s][info   ][gc,task     ] GC(147) Using 2 workers of 2 for marking
[0.896s][info   ][gc,marking  ] GC(147) Concurrent Mark From Roots 1.293ms
[0.896s][info   ][gc,marking  ] GC(147) Concurrent Preclean
[0.896s][info   ][gc,marking  ] GC(147) Concurrent Preclean 0.010ms
[0.896s][info   ][gc,start    ] GC(147) Pause Remark
[0.896s][info   ][gc          ] GC(147) Pause Remark 750M->708M(1024M) 0.582ms
[0.896s][info   ][gc,cpu      ] GC(147) User=0.00s Sys=0.00s Real=0.00s
[0.896s][info   ][gc,marking  ] GC(147) Concurrent Mark 2.116ms
[0.896s][info   ][gc,marking  ] GC(147) Concurrent Rebuild Remembered Sets
[0.898s][info   ][gc,marking  ] GC(147) Concurrent Rebuild Remembered Sets 1.063ms
[0.898s][info   ][gc,start    ] GC(147) Pause Cleanup
[0.898s][info   ][gc          ] GC(147) Pause Cleanup 736M->736M(1024M) 0.321ms
[0.898s][info   ][gc,cpu      ] GC(147) User=0.00s Sys=0.00s Real=0.00s
[0.898s][info   ][gc,marking  ] GC(147) Concurrent Cleanup for Next Mark
[0.899s][info   ][gc,marking  ] GC(147) Concurrent Cleanup for Next Mark 0.515ms
[0.899s][info   ][gc          ] GC(147) Concurrent Mark Cycle 4.431ms
[0.903s][info   ][gc,start    ] GC(148) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.903s][info   ][gc,task     ] GC(148) Using 8 workers of 8 for evacuation
[0.905s][info   ][gc,phases   ] GC(148)   Pre Evacuate Collection Set: 0.1ms
[0.905s][info   ][gc,phases   ] GC(148)   Merge Heap Roots: 0.1ms
[0.905s][info   ][gc,phases   ] GC(148)   Evacuate Collection Set: 1.0ms
[0.905s][info   ][gc,phases   ] GC(148)   Post Evacuate Collection Set: 0.2ms
[0.905s][info   ][gc,phases   ] GC(148)   Other: 0.0ms
[0.905s][info   ][gc,heap     ] GC(148) Eden regions: 95->0(38)
[0.905s][info   ][gc,heap     ] GC(148) Survivor regions: 9->13(13)
[0.905s][info   ][gc,heap     ] GC(148) Old regions: 484->516
[0.905s][info   ][gc,heap     ] GC(148) Archive regions: 2->2
[0.905s][info   ][gc,heap     ] GC(148) Humongous regions: 256->194
[0.905s][info   ][gc,metaspace] GC(148) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.905s][info   ][gc          ] GC(148) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 844M->723M(1024M) 1.620ms
[0.905s][info   ][gc,cpu      ] GC(148) User=0.01s Sys=0.00s Real=0.00s
[0.908s][info   ][gc,start    ] GC(149) Pause Young (Mixed) (G1 Evacuation Pause)
[0.908s][info   ][gc,task     ] GC(149) Using 8 workers of 8 for evacuation
[0.909s][info   ][gc,phases   ] GC(149)   Pre Evacuate Collection Set: 0.2ms
[0.909s][info   ][gc,phases   ] GC(149)   Merge Heap Roots: 0.1ms
[0.909s][info   ][gc,phases   ] GC(149)   Evacuate Collection Set: 0.8ms
[0.909s][info   ][gc,phases   ] GC(149)   Post Evacuate Collection Set: 0.2ms
[0.909s][info   ][gc,phases   ] GC(149)   Other: 0.0ms
[0.909s][info   ][gc,heap     ] GC(149) Eden regions: 38->0(44)
[0.909s][info   ][gc,heap     ] GC(149) Survivor regions: 13->7(7)
[0.909s][info   ][gc,heap     ] GC(149) Old regions: 516->437
[0.909s][info   ][gc,heap     ] GC(149) Archive regions: 2->2
[0.909s][info   ][gc,heap     ] GC(149) Humongous regions: 218->190
[0.909s][info   ][gc,metaspace] GC(149) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.909s][info   ][gc          ] GC(149) Pause Young (Mixed) (G1 Evacuation Pause) 785M->634M(1024M) 1.521ms
[0.909s][info   ][gc,cpu      ] GC(149) User=0.01s Sys=0.00s Real=0.00s
[0.913s][info   ][gc,start    ] GC(150) Pause Young (Mixed) (G1 Evacuation Pause)
[0.913s][info   ][gc,task     ] GC(150) Using 8 workers of 8 for evacuation
[0.914s][info   ][gc,phases   ] GC(150)   Pre Evacuate Collection Set: 0.1ms
[0.914s][info   ][gc,phases   ] GC(150)   Merge Heap Roots: 0.1ms
[0.914s][info   ][gc,phases   ] GC(150)   Evacuate Collection Set: 1.0ms
[0.914s][info   ][gc,phases   ] GC(150)   Post Evacuate Collection Set: 0.2ms
[0.914s][info   ][gc,phases   ] GC(150)   Other: 0.1ms
[0.914s][info   ][gc,heap     ] GC(150) Eden regions: 44->0(44)
[0.914s][info   ][gc,heap     ] GC(150) Survivor regions: 7->7(7)
[0.914s][info   ][gc,heap     ] GC(150) Old regions: 437->372
[0.914s][info   ][gc,heap     ] GC(150) Archive regions: 2->2
[0.914s][info   ][gc,heap     ] GC(150) Humongous regions: 211->189
[0.914s][info   ][gc,metaspace] GC(150) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.914s][info   ][gc          ] GC(150) Pause Young (Mixed) (G1 Evacuation Pause) 699M->568M(1024M) 1.610ms
[0.914s][info   ][gc,cpu      ] GC(150) User=0.00s Sys=0.00s Real=0.00s
[0.918s][info   ][gc,start    ] GC(151) Pause Young (Mixed) (G1 Evacuation Pause)
[0.918s][info   ][gc,task     ] GC(151) Using 8 workers of 8 for evacuation
[0.919s][info   ][gc,phases   ] GC(151)   Pre Evacuate Collection Set: 0.1ms
[0.919s][info   ][gc,phases   ] GC(151)   Merge Heap Roots: 0.1ms
[0.919s][info   ][gc,phases   ] GC(151)   Evacuate Collection Set: 1.0ms
[0.919s][info   ][gc,phases   ] GC(151)   Post Evacuate Collection Set: 0.2ms
[0.919s][info   ][gc,phases   ] GC(151)   Other: 0.0ms
[0.919s][info   ][gc,heap     ] GC(151) Eden regions: 44->0(204)
[0.919s][info   ][gc,heap     ] GC(151) Survivor regions: 7->7(7)
[0.919s][info   ][gc,heap     ] GC(151) Old regions: 372->333
[0.919s][info   ][gc,heap     ] GC(151) Archive regions: 2->2
[0.919s][info   ][gc,heap     ] GC(151) Humongous regions: 212->189
[0.919s][info   ][gc,metaspace] GC(151) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.919s][info   ][gc          ] GC(151) Pause Young (Mixed) (G1 Evacuation Pause) 635M->529M(1024M) 1.554ms
[0.919s][info   ][gc,cpu      ] GC(151) User=0.00s Sys=0.00s Real=0.00s
[0.934s][info   ][gc,start    ] GC(152) Pause Young (Normal) (G1 Evacuation Pause)
[0.934s][info   ][gc,task     ] GC(152) Using 8 workers of 8 for evacuation
[0.936s][info   ][gc,phases   ] GC(152)   Pre Evacuate Collection Set: 0.1ms
[0.936s][info   ][gc,phases   ] GC(152)   Merge Heap Roots: 0.1ms
[0.936s][info   ][gc,phases   ] GC(152)   Evacuate Collection Set: 1.9ms
[0.936s][info   ][gc,phases   ] GC(152)   Post Evacuate Collection Set: 0.2ms
[0.936s][info   ][gc,phases   ] GC(152)   Other: 0.0ms
[0.937s][info   ][gc,heap     ] GC(152) Eden regions: 204->0(163)
[0.937s][info   ][gc,heap     ] GC(152) Survivor regions: 7->27(27)
[0.937s][info   ][gc,heap     ] GC(152) Old regions: 333->390
[0.937s][info   ][gc,heap     ] GC(152) Archive regions: 2->2
[0.937s][info   ][gc,heap     ] GC(152) Humongous regions: 297->186
[0.937s][info   ][gc,metaspace] GC(152) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.937s][info   ][gc          ] GC(152) Pause Young (Normal) (G1 Evacuation Pause) 841M->603M(1024M) 2.583ms
[0.937s][info   ][gc,cpu      ] GC(152) User=0.00s Sys=0.01s Real=0.00s
[0.946s][info   ][gc,start    ] GC(153) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.946s][info   ][gc,task     ] GC(153) Using 8 workers of 8 for evacuation
[0.947s][info   ][gc,phases   ] GC(153)   Pre Evacuate Collection Set: 0.1ms
[0.947s][info   ][gc,phases   ] GC(153)   Merge Heap Roots: 0.1ms
[0.947s][info   ][gc,phases   ] GC(153)   Evacuate Collection Set: 1.4ms
[0.947s][info   ][gc,phases   ] GC(153)   Post Evacuate Collection Set: 0.2ms
[0.947s][info   ][gc,phases   ] GC(153)   Other: 0.1ms
[0.948s][info   ][gc,heap     ] GC(153) Eden regions: 118->0(136)
[0.948s][info   ][gc,heap     ] GC(153) Survivor regions: 27->24(24)
[0.948s][info   ][gc,heap     ] GC(153) Old regions: 390->436
[0.948s][info   ][gc,heap     ] GC(153) Archive regions: 2->2
[0.948s][info   ][gc,heap     ] GC(153) Humongous regions: 259->183
[0.948s][info   ][gc,metaspace] GC(153) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.948s][info   ][gc          ] GC(153) Pause Young (Concurrent Start) (G1 Humongous Allocation) 793M->643M(1024M) 1.973ms
[0.948s][info   ][gc,cpu      ] GC(153) User=0.01s Sys=0.00s Real=0.00s
[0.948s][info   ][gc          ] GC(154) Concurrent Undo Cycle
[0.948s][info   ][gc,marking  ] GC(154) Concurrent Cleanup for Next Mark
[0.948s][info   ][gc,marking  ] GC(154) Concurrent Cleanup for Next Mark 0.041ms
[0.948s][info   ][gc          ] GC(154) Concurrent Undo Cycle 0.109ms
[0.954s][info   ][gc,start    ] GC(155) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.954s][info   ][gc,task     ] GC(155) Using 8 workers of 8 for evacuation
[0.956s][info   ][gc,phases   ] GC(155)   Pre Evacuate Collection Set: 0.2ms
[0.956s][info   ][gc,phases   ] GC(155)   Merge Heap Roots: 0.1ms
[0.956s][info   ][gc,phases   ] GC(155)   Evacuate Collection Set: 1.1ms
[0.956s][info   ][gc,phases   ] GC(155)   Post Evacuate Collection Set: 0.2ms
[0.956s][info   ][gc,phases   ] GC(155)   Other: 0.1ms
[0.956s][info   ][gc,heap     ] GC(155) Eden regions: 88->0(113)
[0.956s][info   ][gc,heap     ] GC(155) Survivor regions: 24->20(20)
[0.956s][info   ][gc,heap     ] GC(155) Old regions: 436->476
[0.956s][info   ][gc,heap     ] GC(155) Archive regions: 2->2
[0.956s][info   ][gc,heap     ] GC(155) Humongous regions: 244->185
[0.956s][info   ][gc,metaspace] GC(155) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.956s][info   ][gc          ] GC(155) Pause Young (Concurrent Start) (G1 Humongous Allocation) 791M->681M(1024M) 1.816ms
[0.956s][info   ][gc,cpu      ] GC(155) User=0.00s Sys=0.00s Real=0.00s
[0.956s][info   ][gc          ] GC(156) Concurrent Undo Cycle
[0.956s][info   ][gc,marking  ] GC(156) Concurrent Cleanup for Next Mark
[0.956s][info   ][gc,marking  ] GC(156) Concurrent Cleanup for Next Mark 0.060ms
[0.956s][info   ][gc          ] GC(156) Concurrent Undo Cycle 0.201ms
[0.961s][info   ][gc,start    ] GC(157) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.961s][info   ][gc,task     ] GC(157) Using 8 workers of 8 for evacuation
[0.963s][info   ][gc,phases   ] GC(157)   Pre Evacuate Collection Set: 0.2ms
[0.963s][info   ][gc,phases   ] GC(157)   Merge Heap Roots: 0.1ms
[0.963s][info   ][gc,phases   ] GC(157)   Evacuate Collection Set: 1.2ms
[0.963s][info   ][gc,phases   ] GC(157)   Post Evacuate Collection Set: 0.1ms
[0.963s][info   ][gc,phases   ] GC(157)   Other: 0.1ms
[0.963s][info   ][gc,heap     ] GC(157) Eden regions: 66->0(102)
[0.963s][info   ][gc,heap     ] GC(157) Survivor regions: 20->17(17)
[0.963s][info   ][gc,heap     ] GC(157) Old regions: 476->507
[0.963s][info   ][gc,heap     ] GC(157) Archive regions: 2->2
[0.963s][info   ][gc,heap     ] GC(157) Humongous regions: 232->180
[0.963s][info   ][gc,metaspace] GC(157) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.963s][info   ][gc          ] GC(157) Pause Young (Concurrent Start) (G1 Humongous Allocation) 793M->704M(1024M) 2.021ms
[0.963s][info   ][gc,cpu      ] GC(157) User=0.00s Sys=0.00s Real=0.00s
[0.963s][info   ][gc          ] GC(158) Concurrent Undo Cycle
[0.963s][info   ][gc,marking  ] GC(158) Concurrent Cleanup for Next Mark
[0.963s][info   ][gc,marking  ] GC(158) Concurrent Cleanup for Next Mark 0.053ms
[0.963s][info   ][gc          ] GC(158) Concurrent Undo Cycle 0.146ms
[0.967s][info   ][gc,start    ] GC(159) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.967s][info   ][gc,task     ] GC(159) Using 8 workers of 8 for evacuation
[0.968s][info   ][gc,phases   ] GC(159)   Pre Evacuate Collection Set: 0.1ms
[0.969s][info   ][gc,phases   ] GC(159)   Merge Heap Roots: 0.1ms
[0.969s][info   ][gc,phases   ] GC(159)   Evacuate Collection Set: 0.8ms
[0.969s][info   ][gc,phases   ] GC(159)   Post Evacuate Collection Set: 0.2ms
[0.969s][info   ][gc,phases   ] GC(159)   Other: 0.1ms
[0.969s][info   ][gc,heap     ] GC(159) Eden regions: 57->0(93)
[0.969s][info   ][gc,heap     ] GC(159) Survivor regions: 17->15(15)
[0.969s][info   ][gc,heap     ] GC(159) Old regions: 507->531
[0.969s][info   ][gc,heap     ] GC(159) Archive regions: 2->2
[0.969s][info   ][gc,heap     ] GC(159) Humongous regions: 216->184
[0.969s][info   ][gc,metaspace] GC(159) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.969s][info   ][gc          ] GC(159) Pause Young (Concurrent Start) (G1 Humongous Allocation) 796M->730M(1024M) 1.457ms
[0.969s][info   ][gc,cpu      ] GC(159) User=0.01s Sys=0.00s Real=0.00s
[0.969s][info   ][gc          ] GC(160) Concurrent Undo Cycle
[0.969s][info   ][gc,marking  ] GC(160) Concurrent Cleanup for Next Mark
[0.969s][info   ][gc,marking  ] GC(160) Concurrent Cleanup for Next Mark 0.037ms
[0.969s][info   ][gc          ] GC(160) Concurrent Undo Cycle 0.115ms
[0.972s][info   ][gc,start    ] GC(161) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.972s][info   ][gc,task     ] GC(161) Using 8 workers of 8 for evacuation
[0.973s][info   ][gc,phases   ] GC(161)   Pre Evacuate Collection Set: 0.2ms
[0.973s][info   ][gc,phases   ] GC(161)   Merge Heap Roots: 0.0ms
[0.973s][info   ][gc,phases   ] GC(161)   Evacuate Collection Set: 0.6ms
[0.973s][info   ][gc,phases   ] GC(161)   Post Evacuate Collection Set: 0.2ms
[0.973s][info   ][gc,phases   ] GC(161)   Other: 0.1ms
[0.973s][info   ][gc,heap     ] GC(161) Eden regions: 37->0(81)
[0.973s][info   ][gc,heap     ] GC(161) Survivor regions: 15->14(14)
[0.973s][info   ][gc,heap     ] GC(161) Old regions: 531->547
[0.973s][info   ][gc,heap     ] GC(161) Archive regions: 2->2
[0.973s][info   ][gc,heap     ] GC(161) Humongous regions: 203->183
[0.973s][info   ][gc,metaspace] GC(161) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.973s][info   ][gc          ] GC(161) Pause Young (Concurrent Start) (G1 Humongous Allocation) 785M->744M(1024M) 1.255ms
[0.973s][info   ][gc,cpu      ] GC(161) User=0.01s Sys=0.00s Real=0.00s
[0.973s][info   ][gc          ] GC(162) Concurrent Undo Cycle
[0.973s][info   ][gc,marking  ] GC(162) Concurrent Cleanup for Next Mark
[0.973s][info   ][gc,marking  ] GC(162) Concurrent Cleanup for Next Mark 0.038ms
[0.973s][info   ][gc          ] GC(162) Concurrent Undo Cycle 0.068ms
[0.975s][info   ][gc,start    ] GC(163) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.975s][info   ][gc,task     ] GC(163) Using 8 workers of 8 for evacuation
[0.976s][info   ][gc,phases   ] GC(163)   Pre Evacuate Collection Set: 0.1ms
[0.976s][info   ][gc,phases   ] GC(163)   Merge Heap Roots: 0.1ms
[0.976s][info   ][gc,phases   ] GC(163)   Evacuate Collection Set: 0.5ms
[0.976s][info   ][gc,phases   ] GC(163)   Post Evacuate Collection Set: 0.1ms
[0.976s][info   ][gc,phases   ] GC(163)   Other: 0.1ms
[0.976s][info   ][gc,heap     ] GC(163) Eden regions: 26->0(81)
[0.976s][info   ][gc,heap     ] GC(163) Survivor regions: 14->10(12)
[0.976s][info   ][gc,heap     ] GC(163) Old regions: 547->561
[0.976s][info   ][gc,heap     ] GC(163) Archive regions: 2->2
[0.976s][info   ][gc,heap     ] GC(163) Humongous regions: 201->182
[0.976s][info   ][gc,metaspace] GC(163) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.976s][info   ][gc          ] GC(163) Pause Young (Concurrent Start) (G1 Humongous Allocation) 788M->752M(1024M) 1.022ms
[0.976s][info   ][gc,cpu      ] GC(163) User=0.00s Sys=0.00s Real=0.00s
[0.976s][info   ][gc          ] GC(164) Concurrent Undo Cycle
[0.976s][info   ][gc,marking  ] GC(164) Concurrent Cleanup for Next Mark
[0.976s][info   ][gc,marking  ] GC(164) Concurrent Cleanup for Next Mark 0.064ms
[0.976s][info   ][gc          ] GC(164) Concurrent Undo Cycle 0.119ms
[0.978s][info   ][gc,start    ] GC(165) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.978s][info   ][gc,task     ] GC(165) Using 8 workers of 8 for evacuation
[0.979s][info   ][gc,phases   ] GC(165)   Pre Evacuate Collection Set: 0.2ms
[0.979s][info   ][gc,phases   ] GC(165)   Merge Heap Roots: 0.1ms
[0.979s][info   ][gc,phases   ] GC(165)   Evacuate Collection Set: 0.4ms
[0.979s][info   ][gc,phases   ] GC(165)   Post Evacuate Collection Set: 0.1ms
[0.979s][info   ][gc,phases   ] GC(165)   Other: 0.1ms
[0.979s][info   ][gc,heap     ] GC(165) Eden regions: 26->0(86)
[0.979s][info   ][gc,heap     ] GC(165) Survivor regions: 10->9(12)
[0.979s][info   ][gc,heap     ] GC(165) Old regions: 561->570
[0.979s][info   ][gc,heap     ] GC(165) Archive regions: 2->2
[0.979s][info   ][gc,heap     ] GC(165) Humongous regions: 191->177
[0.979s][info   ][gc,metaspace] GC(165) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.979s][info   ][gc          ] GC(165) Pause Young (Concurrent Start) (G1 Humongous Allocation) 786M->756M(1024M) 0.976ms
[0.979s][info   ][gc,cpu      ] GC(165) User=0.00s Sys=0.00s Real=0.00s
[0.979s][info   ][gc          ] GC(166) Concurrent Undo Cycle
[0.979s][info   ][gc,marking  ] GC(166) Concurrent Cleanup for Next Mark
[0.979s][info   ][gc,marking  ] GC(166) Concurrent Cleanup for Next Mark 0.037ms
[0.979s][info   ][gc          ] GC(166) Concurrent Undo Cycle 0.086ms
[0.979s][info   ][gc,start    ] GC(167) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.979s][info   ][gc,task     ] GC(167) Using 8 workers of 8 for evacuation
[0.980s][info   ][gc,phases   ] GC(167)   Pre Evacuate Collection Set: 0.1ms
[0.980s][info   ][gc,phases   ] GC(167)   Merge Heap Roots: 0.0ms
[0.980s][info   ][gc,phases   ] GC(167)   Evacuate Collection Set: 0.3ms
[0.980s][info   ][gc,phases   ] GC(167)   Post Evacuate Collection Set: 0.1ms
[0.980s][info   ][gc,phases   ] GC(167)   Other: 0.0ms
[0.980s][info   ][gc,heap     ] GC(167) Eden regions: 2->0(80)
[0.980s][info   ][gc,heap     ] GC(167) Survivor regions: 9->1(12)
[0.980s][info   ][gc,heap     ] GC(167) Old regions: 570->580
[0.980s][info   ][gc,heap     ] GC(167) Archive regions: 2->2
[0.980s][info   ][gc,heap     ] GC(167) Humongous regions: 179->177
[0.980s][info   ][gc,metaspace] GC(167) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.980s][info   ][gc          ] GC(167) Pause Young (Concurrent Start) (G1 Humongous Allocation) 759M->757M(1024M) 0.656ms
[0.980s][info   ][gc,cpu      ] GC(167) User=0.01s Sys=0.00s Real=0.00s
[0.980s][info   ][gc          ] GC(168) Concurrent Mark Cycle
[0.980s][info   ][gc,marking  ] GC(168) Concurrent Clear Claimed Marks
[0.980s][info   ][gc,marking  ] GC(168) Concurrent Clear Claimed Marks 0.008ms
[0.980s][info   ][gc,marking  ] GC(168) Concurrent Scan Root Regions
[0.980s][info   ][gc,marking  ] GC(168) Concurrent Scan Root Regions 0.047ms
[0.980s][info   ][gc,marking  ] GC(168) Concurrent Mark
[0.980s][info   ][gc,marking  ] GC(168) Concurrent Mark From Roots
[0.980s][info   ][gc,task     ] GC(168) Using 2 workers of 2 for marking
[0.981s][info   ][gc,marking  ] GC(168) Concurrent Mark From Roots 1.036ms
[0.981s][info   ][gc,marking  ] GC(168) Concurrent Preclean
[0.981s][info   ][gc,marking  ] GC(168) Concurrent Preclean 0.009ms
[0.981s][info   ][gc,start    ] GC(168) Pause Remark
[0.982s][info   ][gc          ] GC(168) Pause Remark 783M->737M(1024M) 0.352ms
[0.982s][info   ][gc,cpu      ] GC(168) User=0.00s Sys=0.00s Real=0.00s
[0.982s][info   ][gc,marking  ] GC(168) Concurrent Mark 1.575ms
[0.982s][info   ][gc,marking  ] GC(168) Concurrent Rebuild Remembered Sets
[0.983s][info   ][gc,marking  ] GC(168) Concurrent Rebuild Remembered Sets 0.925ms
[0.983s][info   ][gc,start    ] GC(168) Pause Cleanup
[0.983s][info   ][gc          ] GC(168) Pause Cleanup 759M->759M(1024M) 0.247ms
[0.983s][info   ][gc,cpu      ] GC(168) User=0.00s Sys=0.00s Real=0.00s
[0.983s][info   ][gc,marking  ] GC(168) Concurrent Cleanup for Next Mark
[0.984s][info   ][gc,marking  ] GC(168) Concurrent Cleanup for Next Mark 0.534ms
[0.984s][info   ][gc          ] GC(168) Concurrent Mark Cycle 3.744ms
[0.987s][info   ][gc,start    ] GC(169) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.987s][info   ][gc,task     ] GC(169) Using 8 workers of 8 for evacuation
[0.989s][info   ][gc,phases   ] GC(169)   Pre Evacuate Collection Set: 0.1ms
[0.989s][info   ][gc,phases   ] GC(169)   Merge Heap Roots: 0.1ms
[0.989s][info   ][gc,phases   ] GC(169)   Evacuate Collection Set: 0.9ms
[0.989s][info   ][gc,phases   ] GC(169)   Post Evacuate Collection Set: 0.3ms
[0.989s][info   ][gc,phases   ] GC(169)   Other: 0.0ms
[0.989s][info   ][gc,heap     ] GC(169) Eden regions: 80->0(40)
[0.989s][info   ][gc,heap     ] GC(169) Survivor regions: 1->11(11)
[0.989s][info   ][gc,heap     ] GC(169) Old regions: 534->559
[0.989s][info   ][gc,heap     ] GC(169) Archive regions: 2->2
[0.989s][info   ][gc,heap     ] GC(169) Humongous regions: 229->184
[0.989s][info   ][gc,metaspace] GC(169) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.989s][info   ][gc          ] GC(169) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 843M->754M(1024M) 1.649ms
[0.989s][info   ][gc,cpu      ] GC(169) User=0.00s Sys=0.00s Real=0.00s
[0.992s][info   ][gc,start    ] GC(170) Pause Young (Mixed) (G1 Evacuation Pause)
[0.992s][info   ][gc,task     ] GC(170) Using 8 workers of 8 for evacuation
[0.993s][info   ][gc,phases   ] GC(170)   Pre Evacuate Collection Set: 0.2ms
[0.993s][info   ][gc,phases   ] GC(170)   Merge Heap Roots: 0.1ms
[0.993s][info   ][gc,phases   ] GC(170)   Evacuate Collection Set: 0.8ms
[0.993s][info   ][gc,phases   ] GC(170)   Post Evacuate Collection Set: 0.2ms
[0.993s][info   ][gc,phases   ] GC(170)   Other: 0.0ms
[0.993s][info   ][gc,heap     ] GC(170) Eden regions: 40->0(44)
[0.993s][info   ][gc,heap     ] GC(170) Survivor regions: 11->7(7)
[0.993s][info   ][gc,heap     ] GC(170) Old regions: 559->481
[0.993s][info   ][gc,heap     ] GC(170) Archive regions: 2->2
[0.993s][info   ][gc,heap     ] GC(170) Humongous regions: 208->186
[0.993s][info   ][gc,metaspace] GC(170) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.993s][info   ][gc          ] GC(170) Pause Young (Mixed) (G1 Evacuation Pause) 818M->674M(1024M) 1.536ms
[0.993s][info   ][gc,cpu      ] GC(170) User=0.01s Sys=0.00s Real=0.00s
[0.997s][info   ][gc,start    ] GC(171) Pause Young (Mixed) (G1 Evacuation Pause)
[0.997s][info   ][gc,task     ] GC(171) Using 8 workers of 8 for evacuation
[0.999s][info   ][gc,phases   ] GC(171)   Pre Evacuate Collection Set: 0.2ms
[0.999s][info   ][gc,phases   ] GC(171)   Merge Heap Roots: 0.1ms
[0.999s][info   ][gc,phases   ] GC(171)   Evacuate Collection Set: 1.1ms
[0.999s][info   ][gc,phases   ] GC(171)   Post Evacuate Collection Set: 0.2ms
[0.999s][info   ][gc,phases   ] GC(171)   Other: 0.1ms
[0.999s][info   ][gc,heap     ] GC(171) Eden regions: 44->0(44)
[0.999s][info   ][gc,heap     ] GC(171) Survivor regions: 7->7(7)
[0.999s][info   ][gc,heap     ] GC(171) Old regions: 481->415
[0.999s][info   ][gc,heap     ] GC(171) Archive regions: 2->2
[0.999s][info   ][gc,heap     ] GC(171) Humongous regions: 205->182
[0.999s][info   ][gc,metaspace] GC(171) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.999s][info   ][gc          ] GC(171) Pause Young (Mixed) (G1 Evacuation Pause) 737M->604M(1024M) 1.811ms
[0.999s][info   ][gc,cpu      ] GC(171) User=0.01s Sys=0.00s Real=0.00s
[1.002s][info   ][gc,start    ] GC(172) Pause Young (Mixed) (G1 Evacuation Pause)
[1.002s][info   ][gc,task     ] GC(172) Using 8 workers of 8 for evacuation
[1.004s][info   ][gc,phases   ] GC(172)   Pre Evacuate Collection Set: 0.2ms
[1.004s][info   ][gc,phases   ] GC(172)   Merge Heap Roots: 0.1ms
[1.004s][info   ][gc,phases   ] GC(172)   Evacuate Collection Set: 1.1ms
[1.004s][info   ][gc,phases   ] GC(172)   Post Evacuate Collection Set: 0.3ms
[1.004s][info   ][gc,phases   ] GC(172)   Other: 0.0ms
[1.004s][info   ][gc,heap     ] GC(172) Eden regions: 44->0(44)
[1.004s][info   ][gc,heap     ] GC(172) Survivor regions: 7->7(7)
[1.004s][info   ][gc,heap     ] GC(172) Old regions: 415->361
[1.004s][info   ][gc,heap     ] GC(172) Archive regions: 2->2
[1.004s][info   ][gc,heap     ] GC(172) Humongous regions: 211->182
[1.004s][info   ][gc,metaspace] GC(172) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.004s][info   ][gc          ] GC(172) Pause Young (Mixed) (G1 Evacuation Pause) 676M->550M(1024M) 1.933ms
[1.004s][info   ][gc,cpu      ] GC(172) User=0.01s Sys=0.00s Real=0.00s
[1.007s][info   ][gc,start    ] GC(173) Pause Young (Mixed) (G1 Evacuation Pause)
[1.007s][info   ][gc,task     ] GC(173) Using 8 workers of 8 for evacuation
[1.008s][info   ][gc,phases   ] GC(173)   Pre Evacuate Collection Set: 0.2ms
[1.008s][info   ][gc,phases   ] GC(173)   Merge Heap Roots: 0.0ms
[1.008s][info   ][gc,phases   ] GC(173)   Evacuate Collection Set: 0.7ms
[1.008s][info   ][gc,phases   ] GC(173)   Post Evacuate Collection Set: 0.1ms
[1.008s][info   ][gc,phases   ] GC(173)   Other: 0.0ms
[1.008s][info   ][gc,heap     ] GC(173) Eden regions: 44->0(165)
[1.008s][info   ][gc,heap     ] GC(173) Survivor regions: 7->7(7)
[1.008s][info   ][gc,heap     ] GC(173) Old regions: 361->373
[1.008s][info   ][gc,heap     ] GC(173) Archive regions: 2->2
[1.008s][info   ][gc,heap     ] GC(173) Humongous regions: 203->187
[1.008s][info   ][gc,metaspace] GC(173) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.008s][info   ][gc          ] GC(173) Pause Young (Mixed) (G1 Evacuation Pause) 615M->567M(1024M) 1.325ms
[1.008s][info   ][gc,cpu      ] GC(173) User=0.00s Sys=0.00s Real=0.00s
[1.020s][info   ][gc,start    ] GC(174) Pause Young (Normal) (G1 Evacuation Pause)
[1.021s][info   ][gc,task     ] GC(174) Using 8 workers of 8 for evacuation
[1.022s][info   ][gc,phases   ] GC(174)   Pre Evacuate Collection Set: 0.2ms
[1.022s][info   ][gc,phases   ] GC(174)   Merge Heap Roots: 0.1ms
[1.022s][info   ][gc,phases   ] GC(174)   Evacuate Collection Set: 1.4ms
[1.022s][info   ][gc,phases   ] GC(174)   Post Evacuate Collection Set: 0.2ms
[1.022s][info   ][gc,phases   ] GC(174)   Other: 0.0ms
[1.022s][info   ][gc,heap     ] GC(174) Eden regions: 165->0(139)
[1.022s][info   ][gc,heap     ] GC(174) Survivor regions: 7->22(22)
[1.022s][info   ][gc,heap     ] GC(174) Old regions: 373->422
[1.022s][info   ][gc,heap     ] GC(174) Archive regions: 2->2
[1.022s][info   ][gc,heap     ] GC(174) Humongous regions: 288->183
[1.022s][info   ][gc,metaspace] GC(174) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.023s][info   ][gc          ] GC(174) Pause Young (Normal) (G1 Evacuation Pause) 833M->627M(1024M) 2.066ms
[1.023s][info   ][gc,cpu      ] GC(174) User=0.01s Sys=0.00s Real=0.01s
counter:56917
[1.029s][info   ][gc,heap,exit] Heap
[1.029s][info   ][gc,heap,exit]  garbage-first heap   total 1048576K, used 643781K [0x00000007c0000000, 0x0000000800000000)
[1.029s][info   ][gc,heap,exit]   region size 1024K, 23 young (23552K), 22 survivors (22528K)
[1.029s][info   ][gc,heap,exit]  Metaspace       used 233K, committed 448K, reserved 1114112K
[1.029s][info   ][gc,heap,exit]   class space    used 9K, committed 128K, reserved 1048576K
```

再试试2g内存
```
java -XX:+UseG1GC -Xms2g -Xmx2g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以看到GC次数减少，但是性能并没有什么提升，这也说明了GC对于性能的影响开始减少
```
[0.002s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.005s][info   ][gc,init] CardTable entry size: 512
[0.005s][info   ][gc     ] Using G1
[0.007s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.008s][info   ][gc,init] CPUs: 8 total, 8 available
[0.008s][info   ][gc,init] Memory: 16384M
[0.008s][info   ][gc,init] Large Page Support: Disabled
[0.008s][info   ][gc,init] NUMA Support: Disabled
[0.008s][info   ][gc,init] Compressed Oops: Enabled (Zero based)
[0.008s][info   ][gc,init] Heap Region Size: 1M
[0.008s][info   ][gc,init] Heap Min Capacity: 2G
[0.008s][info   ][gc,init] Heap Initial Capacity: 2G
[0.008s][info   ][gc,init] Heap Max Capacity: 2G
[0.008s][info   ][gc,init] Pre-touch: Disabled
[0.008s][info   ][gc,init] Parallel Workers: 8
[0.008s][info   ][gc,init] Concurrent Workers: 2
[0.008s][info   ][gc,init] Concurrent Refinement Workers: 8
[0.008s][info   ][gc,init] Periodic GC: Disabled
[0.008s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.008s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.008s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.047s][info   ][gc,start    ] GC(0) Pause Young (Normal) (G1 Evacuation Pause)
[0.047s][info   ][gc,task     ] GC(0) Using 8 workers of 8 for evacuation
[0.054s][info   ][gc,phases   ] GC(0)   Pre Evacuate Collection Set: 0.1ms
[0.054s][info   ][gc,phases   ] GC(0)   Merge Heap Roots: 0.0ms
[0.054s][info   ][gc,phases   ] GC(0)   Evacuate Collection Set: 5.8ms
[0.054s][info   ][gc,phases   ] GC(0)   Post Evacuate Collection Set: 0.3ms
[0.054s][info   ][gc,phases   ] GC(0)   Other: 0.3ms
[0.054s][info   ][gc,heap     ] GC(0) Eden regions: 102->0(89)
[0.054s][info   ][gc,heap     ] GC(0) Survivor regions: 0->13(13)
[0.054s][info   ][gc,heap     ] GC(0) Old regions: 0->30
[0.054s][info   ][gc,heap     ] GC(0) Archive regions: 2->2
[0.054s][info   ][gc,heap     ] GC(0) Humongous regions: 57->22
[0.054s][info   ][gc,metaspace] GC(0) Metaspace: 155K(384K)->155K(384K) NonClass: 149K(256K)->149K(256K) Class: 5K(128K)->5K(128K)
[0.054s][info   ][gc          ] GC(0) Pause Young (Normal) (G1 Evacuation Pause) 159M->65M(2048M) 6.974ms
[0.054s][info   ][gc,cpu      ] GC(0) User=0.01s Sys=0.02s Real=0.01s
[0.066s][info   ][gc,start    ] GC(1) Pause Young (Normal) (G1 Evacuation Pause)
[0.066s][info   ][gc,task     ] GC(1) Using 8 workers of 8 for evacuation
[0.071s][info   ][gc,phases   ] GC(1)   Pre Evacuate Collection Set: 0.1ms
[0.071s][info   ][gc,phases   ] GC(1)   Merge Heap Roots: 0.0ms
[0.071s][info   ][gc,phases   ] GC(1)   Evacuate Collection Set: 4.8ms
[0.071s][info   ][gc,phases   ] GC(1)   Post Evacuate Collection Set: 0.2ms
[0.071s][info   ][gc,phases   ] GC(1)   Other: 0.0ms
[0.071s][info   ][gc,heap     ] GC(1) Eden regions: 89->0(89)
[0.072s][info   ][gc,heap     ] GC(1) Survivor regions: 13->13(13)
[0.072s][info   ][gc,heap     ] GC(1) Old regions: 30->63
[0.072s][info   ][gc,heap     ] GC(1) Archive regions: 2->2
[0.072s][info   ][gc,heap     ] GC(1) Humongous regions: 72->47
[0.072s][info   ][gc,metaspace] GC(1) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.072s][info   ][gc          ] GC(1) Pause Young (Normal) (G1 Evacuation Pause) 204M->123M(2048M) 5.455ms
[0.072s][info   ][gc,cpu      ] GC(1) User=0.01s Sys=0.02s Real=0.01s
[0.083s][info   ][gc,start    ] GC(2) Pause Young (Normal) (G1 Evacuation Pause)
[0.083s][info   ][gc,task     ] GC(2) Using 8 workers of 8 for evacuation
[0.089s][info   ][gc,phases   ] GC(2)   Pre Evacuate Collection Set: 0.4ms
[0.089s][info   ][gc,phases   ] GC(2)   Merge Heap Roots: 0.1ms
[0.089s][info   ][gc,phases   ] GC(2)   Evacuate Collection Set: 4.7ms
[0.089s][info   ][gc,phases   ] GC(2)   Post Evacuate Collection Set: 0.2ms
[0.089s][info   ][gc,phases   ] GC(2)   Other: 0.0ms
[0.089s][info   ][gc,heap     ] GC(2) Eden regions: 89->0(89)
[0.089s][info   ][gc,heap     ] GC(2) Survivor regions: 13->13(13)
[0.089s][info   ][gc,heap     ] GC(2) Old regions: 63->95
[0.089s][info   ][gc,heap     ] GC(2) Archive regions: 2->2
[0.089s][info   ][gc,heap     ] GC(2) Humongous regions: 100->62
[0.089s][info   ][gc,metaspace] GC(2) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.089s][info   ][gc          ] GC(2) Pause Young (Normal) (G1 Evacuation Pause) 265M->170M(2048M) 5.613ms
[0.089s][info   ][gc,cpu      ] GC(2) User=0.00s Sys=0.02s Real=0.01s
[0.096s][info   ][gc,start    ] GC(3) Pause Young (Normal) (G1 Evacuation Pause)
[0.096s][info   ][gc,task     ] GC(3) Using 8 workers of 8 for evacuation
[0.101s][info   ][gc,phases   ] GC(3)   Pre Evacuate Collection Set: 0.2ms
[0.101s][info   ][gc,phases   ] GC(3)   Merge Heap Roots: 0.1ms
[0.102s][info   ][gc,phases   ] GC(3)   Evacuate Collection Set: 5.0ms
[0.102s][info   ][gc,phases   ] GC(3)   Post Evacuate Collection Set: 0.2ms
[0.102s][info   ][gc,phases   ] GC(3)   Other: 0.1ms
[0.102s][info   ][gc,heap     ] GC(3) Eden regions: 89->0(89)
[0.102s][info   ][gc,heap     ] GC(3) Survivor regions: 13->13(13)
[0.102s][info   ][gc,heap     ] GC(3) Old regions: 95->131
[0.102s][info   ][gc,heap     ] GC(3) Archive regions: 2->2
[0.102s][info   ][gc,heap     ] GC(3) Humongous regions: 103->72
[0.102s][info   ][gc,metaspace] GC(3) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.102s][info   ][gc          ] GC(3) Pause Young (Normal) (G1 Evacuation Pause) 300M->216M(2048M) 5.680ms
[0.102s][info   ][gc,cpu      ] GC(3) User=0.01s Sys=0.02s Real=0.01s
[0.110s][info   ][gc,start    ] GC(4) Pause Young (Normal) (G1 Evacuation Pause)
[0.110s][info   ][gc,task     ] GC(4) Using 8 workers of 8 for evacuation
[0.115s][info   ][gc,phases   ] GC(4)   Pre Evacuate Collection Set: 0.3ms
[0.116s][info   ][gc,phases   ] GC(4)   Merge Heap Roots: 0.1ms
[0.116s][info   ][gc,phases   ] GC(4)   Evacuate Collection Set: 5.1ms
[0.116s][info   ][gc,phases   ] GC(4)   Post Evacuate Collection Set: 0.2ms
[0.116s][info   ][gc,phases   ] GC(4)   Other: 0.1ms
[0.116s][info   ][gc,heap     ] GC(4) Eden regions: 89->0(94)
[0.116s][info   ][gc,heap     ] GC(4) Survivor regions: 13->13(13)
[0.116s][info   ][gc,heap     ] GC(4) Old regions: 131->168
[0.116s][info   ][gc,heap     ] GC(4) Archive regions: 2->2
[0.116s][info   ][gc,heap     ] GC(4) Humongous regions: 133->86
[0.116s][info   ][gc,metaspace] GC(4) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.116s][info   ][gc          ] GC(4) Pause Young (Normal) (G1 Evacuation Pause) 366M->267M(2048M) 5.949ms
[0.116s][info   ][gc,cpu      ] GC(4) User=0.01s Sys=0.02s Real=0.00s
[0.123s][info   ][gc,start    ] GC(5) Pause Young (Normal) (G1 Evacuation Pause)
[0.123s][info   ][gc,task     ] GC(5) Using 8 workers of 8 for evacuation
[0.129s][info   ][gc,phases   ] GC(5)   Pre Evacuate Collection Set: 0.1ms
[0.129s][info   ][gc,phases   ] GC(5)   Merge Heap Roots: 0.1ms
[0.129s][info   ][gc,phases   ] GC(5)   Evacuate Collection Set: 5.4ms
[0.129s][info   ][gc,phases   ] GC(5)   Post Evacuate Collection Set: 0.3ms
[0.129s][info   ][gc,phases   ] GC(5)   Other: 0.0ms
[0.129s][info   ][gc,heap     ] GC(5) Eden regions: 94->0(132)
[0.129s][info   ][gc,heap     ] GC(5) Survivor regions: 13->14(14)
[0.129s][info   ][gc,heap     ] GC(5) Old regions: 168->200
[0.129s][info   ][gc,heap     ] GC(5) Archive regions: 2->2
[0.129s][info   ][gc,heap     ] GC(5) Humongous regions: 139->107
[0.129s][info   ][gc,metaspace] GC(5) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.129s][info   ][gc          ] GC(5) Pause Young (Normal) (G1 Evacuation Pause) 414M->321M(2048M) 6.092ms
[0.129s][info   ][gc,cpu      ] GC(5) User=0.00s Sys=0.03s Real=0.01s
[0.143s][info   ][gc,start    ] GC(6) Pause Young (Normal) (G1 Evacuation Pause)
[0.143s][info   ][gc,task     ] GC(6) Using 8 workers of 8 for evacuation
[0.151s][info   ][gc,phases   ] GC(6)   Pre Evacuate Collection Set: 0.1ms
[0.151s][info   ][gc,phases   ] GC(6)   Merge Heap Roots: 0.1ms
[0.151s][info   ][gc,phases   ] GC(6)   Evacuate Collection Set: 7.3ms
[0.151s][info   ][gc,phases   ] GC(6)   Post Evacuate Collection Set: 0.3ms
[0.151s][info   ][gc,phases   ] GC(6)   Other: 0.1ms
[0.151s][info   ][gc,heap     ] GC(6) Eden regions: 132->0(161)
[0.151s][info   ][gc,heap     ] GC(6) Survivor regions: 14->19(19)
[0.151s][info   ][gc,heap     ] GC(6) Old regions: 200->245
[0.151s][info   ][gc,heap     ] GC(6) Archive regions: 2->2
[0.151s][info   ][gc,heap     ] GC(6) Humongous regions: 191->124
[0.151s][info   ][gc,metaspace] GC(6) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.151s][info   ][gc          ] GC(6) Pause Young (Normal) (G1 Evacuation Pause) 537M->388M(2048M) 8.018ms
[0.151s][info   ][gc,cpu      ] GC(6) User=0.01s Sys=0.03s Real=0.01s
[0.165s][info   ][gc,start    ] GC(7) Pause Young (Normal) (G1 Evacuation Pause)
[0.165s][info   ][gc,task     ] GC(7) Using 8 workers of 8 for evacuation
[0.174s][info   ][gc,phases   ] GC(7)   Pre Evacuate Collection Set: 0.1ms
[0.174s][info   ][gc,phases   ] GC(7)   Merge Heap Roots: 0.1ms
[0.174s][info   ][gc,phases   ] GC(7)   Evacuate Collection Set: 8.3ms
[0.174s][info   ][gc,phases   ] GC(7)   Post Evacuate Collection Set: 0.2ms
[0.174s][info   ][gc,phases   ] GC(7)   Other: 0.0ms
[0.174s][info   ][gc,heap     ] GC(7) Eden regions: 161->0(211)
[0.174s][info   ][gc,heap     ] GC(7) Survivor regions: 19->23(23)
[0.174s][info   ][gc,heap     ] GC(7) Old regions: 245->296
[0.174s][info   ][gc,heap     ] GC(7) Archive regions: 2->2
[0.174s][info   ][gc,heap     ] GC(7) Humongous regions: 212->133
[0.174s][info   ][gc,metaspace] GC(7) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.174s][info   ][gc          ] GC(7) Pause Young (Normal) (G1 Evacuation Pause) 637M->452M(2048M) 9.020ms
[0.174s][info   ][gc,cpu      ] GC(7) User=0.00s Sys=0.04s Real=0.01s
[0.194s][info   ][gc,start    ] GC(8) Pause Young (Normal) (G1 Evacuation Pause)
[0.194s][info   ][gc,task     ] GC(8) Using 8 workers of 8 for evacuation
[0.205s][info   ][gc,phases   ] GC(8)   Pre Evacuate Collection Set: 0.1ms
[0.205s][info   ][gc,phases   ] GC(8)   Merge Heap Roots: 0.1ms
[0.205s][info   ][gc,phases   ] GC(8)   Evacuate Collection Set: 10.3ms
[0.205s][info   ][gc,phases   ] GC(8)   Post Evacuate Collection Set: 0.3ms
[0.205s][info   ][gc,phases   ] GC(8)   Other: 0.1ms
[0.205s][info   ][gc,heap     ] GC(8) Eden regions: 211->0(323)
[0.205s][info   ][gc,heap     ] GC(8) Survivor regions: 23->30(30)
[0.205s][info   ][gc,heap     ] GC(8) Old regions: 296->356
[0.205s][info   ][gc,heap     ] GC(8) Archive regions: 2->2
[0.205s][info   ][gc,heap     ] GC(8) Humongous regions: 236->131
[0.205s][info   ][gc,metaspace] GC(8) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.205s][info   ][gc          ] GC(8) Pause Young (Normal) (G1 Evacuation Pause) 765M->517M(2048M) 11.131ms
[0.205s][info   ][gc,cpu      ] GC(8) User=0.01s Sys=0.04s Real=0.01s
[0.239s][info   ][gc,start    ] GC(9) Pause Young (Normal) (G1 Evacuation Pause)
[0.239s][info   ][gc,task     ] GC(9) Using 8 workers of 8 for evacuation
[0.253s][info   ][gc,phases   ] GC(9)   Pre Evacuate Collection Set: 0.1ms
[0.253s][info   ][gc,phases   ] GC(9)   Merge Heap Roots: 0.1ms
[0.253s][info   ][gc,phases   ] GC(9)   Evacuate Collection Set: 12.9ms
[0.253s][info   ][gc,phases   ] GC(9)   Post Evacuate Collection Set: 0.4ms
[0.253s][info   ][gc,phases   ] GC(9)   Other: 0.1ms
[0.253s][info   ][gc,heap     ] GC(9) Eden regions: 323->0(333)
[0.253s][info   ][gc,heap     ] GC(9) Survivor regions: 30->45(45)
[0.253s][info   ][gc,heap     ] GC(9) Old regions: 356->432
[0.253s][info   ][gc,heap     ] GC(9) Archive regions: 2->2
[0.253s][info   ][gc,heap     ] GC(9) Humongous regions: 299->157
[0.253s][info   ][gc,metaspace] GC(9) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.253s][info   ][gc          ] GC(9) Pause Young (Normal) (G1 Evacuation Pause) 1008M->634M(2048M) 13.849ms
[0.253s][info   ][gc,cpu      ] GC(9) User=0.01s Sys=0.05s Real=0.01s
[0.282s][info   ][gc,start    ] GC(10) Pause Young (Normal) (G1 Evacuation Pause)
[0.282s][info   ][gc,task     ] GC(10) Using 8 workers of 8 for evacuation
[0.295s][info   ][gc,phases   ] GC(10)   Pre Evacuate Collection Set: 0.2ms
[0.295s][info   ][gc,phases   ] GC(10)   Merge Heap Roots: 0.1ms
[0.295s][info   ][gc,phases   ] GC(10)   Evacuate Collection Set: 12.3ms
[0.295s][info   ][gc,phases   ] GC(10)   Post Evacuate Collection Set: 0.3ms
[0.295s][info   ][gc,phases   ] GC(10)   Other: 0.1ms
[0.295s][info   ][gc,heap     ] GC(10) Eden regions: 333->0(639)
[0.295s][info   ][gc,heap     ] GC(10) Survivor regions: 45->48(48)
[0.295s][info   ][gc,heap     ] GC(10) Old regions: 432->512
[0.295s][info   ][gc,heap     ] GC(10) Archive regions: 2->2
[0.295s][info   ][gc,heap     ] GC(10) Humongous regions: 323->156
[0.295s][info   ][gc,metaspace] GC(10) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.295s][info   ][gc          ] GC(10) Pause Young (Normal) (G1 Evacuation Pause) 1133M->716M(2048M) 13.062ms
[0.295s][info   ][gc,cpu      ] GC(10) User=0.01s Sys=0.05s Real=0.01s
[0.335s][info   ][gc,start    ] GC(11) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.335s][info   ][gc,task     ] GC(11) Using 8 workers of 8 for evacuation
[0.357s][info   ][gc,phases   ] GC(11)   Pre Evacuate Collection Set: 0.3ms
[0.357s][info   ][gc,phases   ] GC(11)   Merge Heap Roots: 0.1ms
[0.357s][info   ][gc,phases   ] GC(11)   Evacuate Collection Set: 20.3ms
[0.357s][info   ][gc,phases   ] GC(11)   Post Evacuate Collection Set: 0.5ms
[0.357s][info   ][gc,phases   ] GC(11)   Other: 0.1ms
[0.357s][info   ][gc,heap     ] GC(11) Eden regions: 438->0(554)
[0.357s][info   ][gc,heap     ] GC(11) Survivor regions: 48->86(86)
[0.357s][info   ][gc,heap     ] GC(11) Old regions: 512->582
[0.357s][info   ][gc,heap     ] GC(11) Archive regions: 2->2
[0.357s][info   ][gc,heap     ] GC(11) Humongous regions: 407->166
[0.357s][info   ][gc,metaspace] GC(11) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.357s][info   ][gc          ] GC(11) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1404M->834M(2048M) 21.423ms
[0.357s][info   ][gc,cpu      ] GC(11) User=0.01s Sys=0.09s Real=0.02s
[0.357s][info   ][gc          ] GC(12) Concurrent Undo Cycle
[0.357s][info   ][gc,marking  ] GC(12) Concurrent Cleanup for Next Mark
[0.357s][info   ][gc,marking  ] GC(12) Concurrent Cleanup for Next Mark 0.051ms
[0.357s][info   ][gc          ] GC(12) Concurrent Undo Cycle 0.074ms
[0.380s][info   ][gc,start    ] GC(13) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.380s][info   ][gc,task     ] GC(13) Using 8 workers of 8 for evacuation
[0.386s][info   ][gc,phases   ] GC(13)   Pre Evacuate Collection Set: 0.5ms
[0.386s][info   ][gc,phases   ] GC(13)   Merge Heap Roots: 0.1ms
[0.386s][info   ][gc,phases   ] GC(13)   Evacuate Collection Set: 4.8ms
[0.386s][info   ][gc,phases   ] GC(13)   Post Evacuate Collection Set: 0.3ms
[0.386s][info   ][gc,phases   ] GC(13)   Other: 0.1ms
[0.386s][info   ][gc,heap     ] GC(13) Eden regions: 298->0(559)
[0.386s][info   ][gc,heap     ] GC(13) Survivor regions: 86->80(80)
[0.386s][info   ][gc,heap     ] GC(13) Old regions: 582->657
[0.386s][info   ][gc,heap     ] GC(13) Archive regions: 2->2
[0.386s][info   ][gc,heap     ] GC(13) Humongous regions: 337->164
[0.386s][info   ][gc,metaspace] GC(13) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.386s][info   ][gc          ] GC(13) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1302M->901M(2048M) 6.106ms
[0.386s][info   ][gc,cpu      ] GC(13) User=0.02s Sys=0.01s Real=0.00s
[0.386s][info   ][gc          ] GC(14) Concurrent Undo Cycle
[0.386s][info   ][gc,marking  ] GC(14) Concurrent Cleanup for Next Mark
[0.386s][info   ][gc,marking  ] GC(14) Concurrent Cleanup for Next Mark 0.036ms
[0.386s][info   ][gc          ] GC(14) Concurrent Undo Cycle 0.088ms
[0.399s][info   ][gc,start    ] GC(15) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.399s][info   ][gc,task     ] GC(15) Using 8 workers of 8 for evacuation
[0.403s][info   ][gc,phases   ] GC(15)   Pre Evacuate Collection Set: 0.4ms
[0.403s][info   ][gc,phases   ] GC(15)   Merge Heap Roots: 0.1ms
[0.403s][info   ][gc,phases   ] GC(15)   Evacuate Collection Set: 3.2ms
[0.403s][info   ][gc,phases   ] GC(15)   Post Evacuate Collection Set: 0.2ms
[0.403s][info   ][gc,phases   ] GC(15)   Other: 0.1ms
[0.403s][info   ][gc,heap     ] GC(15) Eden regions: 168->0(532)
[0.403s][info   ][gc,heap     ] GC(15) Survivor regions: 80->65(80)
[0.403s][info   ][gc,heap     ] GC(15) Old regions: 657->719
[0.403s][info   ][gc,heap     ] GC(15) Archive regions: 2->2
[0.403s][info   ][gc,heap     ] GC(15) Humongous regions: 262->172
[0.403s][info   ][gc,metaspace] GC(15) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.403s][info   ][gc          ] GC(15) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1166M->955M(2048M) 4.199ms
[0.403s][info   ][gc,cpu      ] GC(15) User=0.01s Sys=0.01s Real=0.00s
[0.403s][info   ][gc          ] GC(16) Concurrent Undo Cycle
[0.403s][info   ][gc,marking  ] GC(16) Concurrent Cleanup for Next Mark
[0.403s][info   ][gc,marking  ] GC(16) Concurrent Cleanup for Next Mark 0.040ms
[0.403s][info   ][gc          ] GC(16) Concurrent Undo Cycle 0.103ms
[0.407s][info   ][gc,start    ] GC(17) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.407s][info   ][gc,task     ] GC(17) Using 8 workers of 8 for evacuation
[0.410s][info   ][gc,phases   ] GC(17)   Pre Evacuate Collection Set: 0.4ms
[0.410s][info   ][gc,phases   ] GC(17)   Merge Heap Roots: 0.0ms
[0.410s][info   ][gc,phases   ] GC(17)   Evacuate Collection Set: 2.3ms
[0.410s][info   ][gc,phases   ] GC(17)   Post Evacuate Collection Set: 0.2ms
[0.410s][info   ][gc,phases   ] GC(17)   Other: 0.1ms
[0.410s][info   ][gc,heap     ] GC(17) Eden regions: 50->0(480)
[0.410s][info   ][gc,heap     ] GC(17) Survivor regions: 65->21(75)
[0.410s][info   ][gc,heap     ] GC(17) Old regions: 719->781
[0.410s][info   ][gc,heap     ] GC(17) Archive regions: 2->2
[0.410s][info   ][gc,heap     ] GC(17) Humongous regions: 201->168
[0.410s][info   ][gc,metaspace] GC(17) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.410s][info   ][gc          ] GC(17) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1034M->969M(2048M) 3.196ms
[0.410s][info   ][gc,cpu      ] GC(17) User=0.00s Sys=0.00s Real=0.01s
[0.411s][info   ][gc          ] GC(18) Concurrent Mark Cycle
[0.411s][info   ][gc,marking  ] GC(18) Concurrent Clear Claimed Marks
[0.411s][info   ][gc,marking  ] GC(18) Concurrent Clear Claimed Marks 0.010ms
[0.411s][info   ][gc,marking  ] GC(18) Concurrent Scan Root Regions
[0.411s][info   ][gc,marking  ] GC(18) Concurrent Scan Root Regions 0.201ms
[0.411s][info   ][gc,marking  ] GC(18) Concurrent Mark
[0.411s][info   ][gc,marking  ] GC(18) Concurrent Mark From Roots
[0.411s][info   ][gc,task     ] GC(18) Using 2 workers of 2 for marking
[0.413s][info   ][gc,marking  ] GC(18) Concurrent Mark From Roots 2.107ms
[0.413s][info   ][gc,marking  ] GC(18) Concurrent Preclean
[0.413s][info   ][gc,marking  ] GC(18) Concurrent Preclean 0.012ms
[0.413s][info   ][gc,start    ] GC(18) Pause Remark
[0.414s][info   ][gc          ] GC(18) Pause Remark 1023M->718M(2048M) 0.519ms
[0.414s][info   ][gc,cpu      ] GC(18) User=0.00s Sys=0.00s Real=0.00s
[0.414s][info   ][gc,marking  ] GC(18) Concurrent Mark 2.960ms
[0.414s][info   ][gc,marking  ] GC(18) Concurrent Rebuild Remembered Sets
[0.415s][info   ][gc,marking  ] GC(18) Concurrent Rebuild Remembered Sets 1.016ms
[0.415s][info   ][gc,start    ] GC(18) Pause Cleanup
[0.415s][info   ][gc          ] GC(18) Pause Cleanup 734M->734M(2048M) 0.311ms
[0.415s][info   ][gc,cpu      ] GC(18) User=0.00s Sys=0.00s Real=0.00s
[0.415s][info   ][gc,marking  ] GC(18) Concurrent Cleanup for Next Mark
[0.420s][info   ][gc,marking  ] GC(18) Concurrent Cleanup for Next Mark 4.259ms
[0.420s][info   ][gc          ] GC(18) Concurrent Mark Cycle 9.130ms
[0.450s][info   ][gc,start    ] GC(19) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.450s][info   ][gc,task     ] GC(19) Using 8 workers of 8 for evacuation
[0.459s][info   ][gc,phases   ] GC(19)   Pre Evacuate Collection Set: 0.3ms
[0.459s][info   ][gc,phases   ] GC(19)   Merge Heap Roots: 0.1ms
[0.459s][info   ][gc,phases   ] GC(19)   Evacuate Collection Set: 7.8ms
[0.459s][info   ][gc,phases   ] GC(19)   Post Evacuate Collection Set: 0.8ms
[0.459s][info   ][gc,phases   ] GC(19)   Other: 0.1ms
[0.459s][info   ][gc,heap     ] GC(19) Eden regions: 480->0(39)
[0.459s][info   ][gc,heap     ] GC(19) Survivor regions: 21->63(63)
[0.459s][info   ][gc,heap     ] GC(19) Old regions: 476->561
[0.459s][info   ][gc,heap     ] GC(19) Archive regions: 2->2
[0.459s][info   ][gc,heap     ] GC(19) Humongous regions: 466->178
[0.459s][info   ][gc,metaspace] GC(19) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.459s][info   ][gc          ] GC(19) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 1442M->802M(2048M) 9.293ms
[0.459s][info   ][gc,cpu      ] GC(19) User=0.01s Sys=0.03s Real=0.01s
[0.462s][info   ][gc,start    ] GC(20) Pause Young (Mixed) (G1 Evacuation Pause)
[0.463s][info   ][gc,task     ] GC(20) Using 8 workers of 8 for evacuation
[0.466s][info   ][gc,phases   ] GC(20)   Pre Evacuate Collection Set: 0.3ms
[0.466s][info   ][gc,phases   ] GC(20)   Merge Heap Roots: 0.3ms
[0.466s][info   ][gc,phases   ] GC(20)   Evacuate Collection Set: 2.4ms
[0.466s][info   ][gc,phases   ] GC(20)   Post Evacuate Collection Set: 0.2ms
[0.466s][info   ][gc,phases   ] GC(20)   Other: 0.0ms
[0.466s][info   ][gc,heap     ] GC(20) Eden regions: 39->0(781)
[0.466s][info   ][gc,heap     ] GC(20) Survivor regions: 63->13(13)
[0.466s][info   ][gc,heap     ] GC(20) Old regions: 561->475
[0.466s][info   ][gc,heap     ] GC(20) Archive regions: 2->2
[0.466s][info   ][gc,heap     ] GC(20) Humongous regions: 206->186
[0.466s][info   ][gc,metaspace] GC(20) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.466s][info   ][gc          ] GC(20) Pause Young (Mixed) (G1 Evacuation Pause) 869M->674M(2048M) 3.500ms
[0.466s][info   ][gc,cpu      ] GC(20) User=0.01s Sys=0.00s Real=0.00s
[0.499s][info   ][gc,start    ] GC(21) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.500s][info   ][gc,task     ] GC(21) Using 8 workers of 8 for evacuation
[0.504s][info   ][gc,phases   ] GC(21)   Pre Evacuate Collection Set: 0.2ms
[0.504s][info   ][gc,phases   ] GC(21)   Merge Heap Roots: 0.1ms
[0.504s][info   ][gc,phases   ] GC(21)   Evacuate Collection Set: 3.2ms
[0.504s][info   ][gc,phases   ] GC(21)   Post Evacuate Collection Set: 0.5ms
[0.504s][info   ][gc,phases   ] GC(21)   Other: 0.1ms
[0.504s][info   ][gc,heap     ] GC(21) Eden regions: 443->0(595)
[0.504s][info   ][gc,heap     ] GC(21) Survivor regions: 13->100(100)
[0.504s][info   ][gc,heap     ] GC(21) Old regions: 475->511
[0.504s][info   ][gc,heap     ] GC(21) Archive regions: 2->2
[0.504s][info   ][gc,heap     ] GC(21) Humongous regions: 444->198
[0.504s][info   ][gc,metaspace] GC(21) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.504s][info   ][gc          ] GC(21) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1374M->809M(2048M) 4.400ms
[0.504s][info   ][gc,cpu      ] GC(21) User=0.01s Sys=0.00s Real=0.00s
[0.504s][info   ][gc          ] GC(22) Concurrent Undo Cycle
[0.504s][info   ][gc,marking  ] GC(22) Concurrent Cleanup for Next Mark
[0.504s][info   ][gc,marking  ] GC(22) Concurrent Cleanup for Next Mark 0.055ms
[0.504s][info   ][gc          ] GC(22) Concurrent Undo Cycle 0.096ms
[0.529s][info   ][gc,start    ] GC(23) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.529s][info   ][gc,task     ] GC(23) Using 8 workers of 8 for evacuation
[0.533s][info   ][gc,phases   ] GC(23)   Pre Evacuate Collection Set: 0.4ms
[0.533s][info   ][gc,phases   ] GC(23)   Merge Heap Roots: 0.1ms
[0.533s][info   ][gc,phases   ] GC(23)   Evacuate Collection Set: 3.4ms
[0.533s][info   ][gc,phases   ] GC(23)   Post Evacuate Collection Set: 0.4ms
[0.533s][info   ][gc,phases   ] GC(23)   Other: 0.1ms
[0.533s][info   ][gc,heap     ] GC(23) Eden regions: 318->0(575)
[0.533s][info   ][gc,heap     ] GC(23) Survivor regions: 100->87(87)
[0.534s][info   ][gc,heap     ] GC(23) Old regions: 511->582
[0.534s][info   ][gc,heap     ] GC(23) Archive regions: 2->2
[0.534s][info   ][gc,heap     ] GC(23) Humongous regions: 408->210
[0.534s][info   ][gc,metaspace] GC(23) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.534s][info   ][gc          ] GC(23) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1336M->879M(2048M) 4.439ms
[0.534s][info   ][gc,cpu      ] GC(23) User=0.02s Sys=0.01s Real=0.00s
[0.534s][info   ][gc          ] GC(24) Concurrent Undo Cycle
[0.534s][info   ][gc,marking  ] GC(24) Concurrent Cleanup for Next Mark
[0.534s][info   ][gc,marking  ] GC(24) Concurrent Cleanup for Next Mark 0.045ms
[0.534s][info   ][gc          ] GC(24) Concurrent Undo Cycle 0.111ms
[0.550s][info   ][gc,start    ] GC(25) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.550s][info   ][gc,task     ] GC(25) Using 8 workers of 8 for evacuation
[0.553s][info   ][gc,phases   ] GC(25)   Pre Evacuate Collection Set: 0.3ms
[0.553s][info   ][gc,phases   ] GC(25)   Merge Heap Roots: 0.1ms
[0.553s][info   ][gc,phases   ] GC(25)   Evacuate Collection Set: 2.6ms
[0.553s][info   ][gc,phases   ] GC(25)   Post Evacuate Collection Set: 0.3ms
[0.553s][info   ][gc,phases   ] GC(25)   Other: 0.1ms
[0.553s][info   ][gc,heap     ] GC(25) Eden regions: 208->0(481)
[0.553s][info   ][gc,heap     ] GC(25) Survivor regions: 87->74(83)
[0.553s][info   ][gc,heap     ] GC(25) Old regions: 582->642
[0.553s][info   ][gc,heap     ] GC(25) Archive regions: 2->2
[0.553s][info   ][gc,heap     ] GC(25) Humongous regions: 337->194
[0.553s][info   ][gc,metaspace] GC(25) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.553s][info   ][gc          ] GC(25) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1214M->909M(2048M) 3.585ms
[0.553s][info   ][gc,cpu      ] GC(25) User=0.02s Sys=0.00s Real=0.00s
[0.553s][info   ][gc          ] GC(26) Concurrent Undo Cycle
[0.553s][info   ][gc,marking  ] GC(26) Concurrent Cleanup for Next Mark
[0.553s][info   ][gc,marking  ] GC(26) Concurrent Cleanup for Next Mark 0.045ms
[0.553s][info   ][gc          ] GC(26) Concurrent Undo Cycle 0.128ms
[0.565s][info   ][gc,start    ] GC(27) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.565s][info   ][gc,task     ] GC(27) Using 8 workers of 8 for evacuation
[0.568s][info   ][gc,phases   ] GC(27)   Pre Evacuate Collection Set: 0.4ms
[0.568s][info   ][gc,phases   ] GC(27)   Merge Heap Roots: 0.1ms
[0.568s][info   ][gc,phases   ] GC(27)   Evacuate Collection Set: 2.2ms
[0.568s][info   ][gc,phases   ] GC(27)   Post Evacuate Collection Set: 0.3ms
[0.568s][info   ][gc,phases   ] GC(27)   Other: 0.1ms
[0.568s][info   ][gc,heap     ] GC(27) Eden regions: 156->0(503)
[0.568s][info   ][gc,heap     ] GC(27) Survivor regions: 74->59(70)
[0.568s][info   ][gc,heap     ] GC(27) Old regions: 642->699
[0.568s][info   ][gc,heap     ] GC(27) Archive regions: 2->2
[0.568s][info   ][gc,heap     ] GC(27) Humongous regions: 277->176
[0.568s][info   ][gc,metaspace] GC(27) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.568s][info   ][gc          ] GC(27) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1148M->934M(2048M) 3.315ms
[0.568s][info   ][gc,cpu      ] GC(27) User=0.01s Sys=0.01s Real=0.01s
[0.568s][info   ][gc          ] GC(28) Concurrent Undo Cycle
[0.568s][info   ][gc,marking  ] GC(28) Concurrent Cleanup for Next Mark
[0.568s][info   ][gc,marking  ] GC(28) Concurrent Cleanup for Next Mark 0.045ms
[0.568s][info   ][gc          ] GC(28) Concurrent Undo Cycle 0.075ms
[0.573s][info   ][gc,start    ] GC(29) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.573s][info   ][gc,task     ] GC(29) Using 8 workers of 8 for evacuation
[0.576s][info   ][gc,phases   ] GC(29)   Pre Evacuate Collection Set: 0.3ms
[0.576s][info   ][gc,phases   ] GC(29)   Merge Heap Roots: 0.1ms
[0.576s][info   ][gc,phases   ] GC(29)   Evacuate Collection Set: 1.8ms
[0.576s][info   ][gc,phases   ] GC(29)   Post Evacuate Collection Set: 0.2ms
[0.576s][info   ][gc,phases   ] GC(29)   Other: 0.1ms
[0.576s][info   ][gc,heap     ] GC(29) Eden regions: 76->0(497)
[0.576s][info   ][gc,heap     ] GC(29) Survivor regions: 59->31(71)
[0.576s][info   ][gc,heap     ] GC(29) Old regions: 699->752
[0.576s][info   ][gc,heap     ] GC(29) Archive regions: 2->2
[0.576s][info   ][gc,heap     ] GC(29) Humongous regions: 221->180
[0.576s][info   ][gc,metaspace] GC(29) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.576s][info   ][gc          ] GC(29) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1055M->963M(2048M) 2.640ms
[0.576s][info   ][gc,cpu      ] GC(29) User=0.01s Sys=0.00s Real=0.00s
[0.576s][info   ][gc          ] GC(30) Concurrent Mark Cycle
[0.576s][info   ][gc,marking  ] GC(30) Concurrent Clear Claimed Marks
[0.576s][info   ][gc,marking  ] GC(30) Concurrent Clear Claimed Marks 0.016ms
[0.576s][info   ][gc,marking  ] GC(30) Concurrent Scan Root Regions
[0.576s][info   ][gc,marking  ] GC(30) Concurrent Scan Root Regions 0.091ms
[0.576s][info   ][gc,marking  ] GC(30) Concurrent Mark
[0.576s][info   ][gc,marking  ] GC(30) Concurrent Mark From Roots
[0.576s][info   ][gc,task     ] GC(30) Using 2 workers of 2 for marking
[0.577s][info   ][gc,marking  ] GC(30) Concurrent Mark From Roots 1.154ms
[0.577s][info   ][gc,marking  ] GC(30) Concurrent Preclean
[0.577s][info   ][gc,marking  ] GC(30) Concurrent Preclean 0.010ms
[0.578s][info   ][gc,start    ] GC(30) Pause Remark
[0.578s][info   ][gc          ] GC(30) Pause Remark 991M->706M(2048M) 0.534ms
[0.578s][info   ][gc,cpu      ] GC(30) User=0.00s Sys=0.00s Real=0.01s
[0.578s][info   ][gc,marking  ] GC(30) Concurrent Mark 1.971ms
[0.578s][info   ][gc,marking  ] GC(30) Concurrent Rebuild Remembered Sets
[0.579s][info   ][gc,marking  ] GC(30) Concurrent Rebuild Remembered Sets 1.040ms
[0.579s][info   ][gc,start    ] GC(30) Pause Cleanup
[0.580s][info   ][gc          ] GC(30) Pause Cleanup 721M->721M(2048M) 0.344ms
[0.580s][info   ][gc,cpu      ] GC(30) User=0.00s Sys=0.00s Real=0.00s
[0.580s][info   ][gc,marking  ] GC(30) Concurrent Cleanup for Next Mark
[0.582s][info   ][gc,marking  ] GC(30) Concurrent Cleanup for Next Mark 2.087ms
[0.582s][info   ][gc          ] GC(30) Concurrent Mark Cycle 5.913ms
[0.617s][info   ][gc,start    ] GC(31) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.617s][info   ][gc,task     ] GC(31) Using 8 workers of 8 for evacuation
[0.625s][info   ][gc,phases   ] GC(31)   Pre Evacuate Collection Set: 0.2ms
[0.625s][info   ][gc,phases   ] GC(31)   Merge Heap Roots: 0.1ms
[0.625s][info   ][gc,phases   ] GC(31)   Evacuate Collection Set: 7.3ms
[0.625s][info   ][gc,phases   ] GC(31)   Post Evacuate Collection Set: 0.3ms
[0.625s][info   ][gc,phases   ] GC(31)   Other: 0.1ms
[0.625s][info   ][gc,heap     ] GC(31) Eden regions: 497->0(36)
[0.625s][info   ][gc,heap     ] GC(31) Survivor regions: 31->66(66)
[0.625s][info   ][gc,heap     ] GC(31) Old regions: 467->556
[0.625s][info   ][gc,heap     ] GC(31) Archive regions: 2->2
[0.625s][info   ][gc,heap     ] GC(31) Humongous regions: 467->177
[0.625s][info   ][gc,metaspace] GC(31) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.625s][info   ][gc          ] GC(31) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 1462M->799M(2048M) 8.309ms
[0.625s][info   ][gc,cpu      ] GC(31) User=0.02s Sys=0.03s Real=0.01s
[0.628s][info   ][gc,start    ] GC(32) Pause Young (Mixed) (G1 Evacuation Pause)
[0.628s][info   ][gc,task     ] GC(32) Using 8 workers of 8 for evacuation
[0.631s][info   ][gc,phases   ] GC(32)   Pre Evacuate Collection Set: 0.1ms
[0.631s][info   ][gc,phases   ] GC(32)   Merge Heap Roots: 0.2ms
[0.631s][info   ][gc,phases   ] GC(32)   Evacuate Collection Set: 2.5ms
[0.631s][info   ][gc,phases   ] GC(32)   Post Evacuate Collection Set: 0.4ms
[0.631s][info   ][gc,phases   ] GC(32)   Other: 0.1ms
[0.631s][info   ][gc,heap     ] GC(32) Eden regions: 36->0(820)
[0.631s][info   ][gc,heap     ] GC(32) Survivor regions: 66->13(13)
[0.631s][info   ][gc,heap     ] GC(32) Old regions: 556->459
[0.631s][info   ][gc,heap     ] GC(32) Archive regions: 2->2
[0.631s][info   ][gc,heap     ] GC(32) Humongous regions: 193->172
[0.631s][info   ][gc,metaspace] GC(32) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.632s][info   ][gc          ] GC(32) Pause Young (Mixed) (G1 Evacuation Pause) 851M->644M(2048M) 3.482ms
[0.632s][info   ][gc,cpu      ] GC(32) User=0.01s Sys=0.00s Real=0.00s
[0.666s][info   ][gc,start    ] GC(33) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.666s][info   ][gc,task     ] GC(33) Using 8 workers of 8 for evacuation
[0.670s][info   ][gc,phases   ] GC(33)   Pre Evacuate Collection Set: 0.4ms
[0.670s][info   ][gc,phases   ] GC(33)   Merge Heap Roots: 0.1ms
[0.670s][info   ][gc,phases   ] GC(33)   Evacuate Collection Set: 2.9ms
[0.670s][info   ][gc,phases   ] GC(33)   Post Evacuate Collection Set: 0.3ms
[0.670s][info   ][gc,phases   ] GC(33)   Other: 0.1ms
[0.670s][info   ][gc,heap     ] GC(33) Eden regions: 462->0(690)
[0.670s][info   ][gc,heap     ] GC(33) Survivor regions: 13->105(105)
[0.670s][info   ][gc,heap     ] GC(33) Old regions: 459->500
[0.670s][info   ][gc,heap     ] GC(33) Archive regions: 2->2
[0.670s][info   ][gc,heap     ] GC(33) Humongous regions: 461->174
[0.670s][info   ][gc,metaspace] GC(33) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.670s][info   ][gc          ] GC(33) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1395M->779M(2048M) 3.997ms
[0.670s][info   ][gc,cpu      ] GC(33) User=0.02s Sys=0.01s Real=0.01s
[0.670s][info   ][gc          ] GC(34) Concurrent Undo Cycle
[0.670s][info   ][gc,marking  ] GC(34) Concurrent Cleanup for Next Mark
[0.670s][info   ][gc,marking  ] GC(34) Concurrent Cleanup for Next Mark 0.061ms
[0.670s][info   ][gc          ] GC(34) Concurrent Undo Cycle 0.166ms
[0.705s][info   ][gc,start    ] GC(35) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.705s][info   ][gc,task     ] GC(35) Using 8 workers of 8 for evacuation
[0.716s][info   ][gc,phases   ] GC(35)   Pre Evacuate Collection Set: 0.2ms
[0.716s][info   ][gc,phases   ] GC(35)   Merge Heap Roots: 0.1ms
[0.716s][info   ][gc,phases   ] GC(35)   Evacuate Collection Set: 10.9ms
[0.716s][info   ][gc,phases   ] GC(35)   Post Evacuate Collection Set: 0.3ms
[0.716s][info   ][gc,phases   ] GC(35)   Other: 0.1ms
[0.716s][info   ][gc,heap     ] GC(35) Eden regions: 445->0(576)
[0.717s][info   ][gc,heap     ] GC(35) Survivor regions: 105->100(100)
[0.717s][info   ][gc,heap     ] GC(35) Old regions: 500->573
[0.717s][info   ][gc,heap     ] GC(35) Archive regions: 2->2
[0.717s][info   ][gc,heap     ] GC(35) Humongous regions: 419->187
[0.717s][info   ][gc,metaspace] GC(35) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.717s][info   ][gc          ] GC(35) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1469M->860M(2048M) 11.856ms
[0.717s][info   ][gc,cpu      ] GC(35) User=0.02s Sys=0.03s Real=0.01s
[0.717s][info   ][gc          ] GC(36) Concurrent Undo Cycle
[0.717s][info   ][gc,marking  ] GC(36) Concurrent Cleanup for Next Mark
[0.717s][info   ][gc,marking  ] GC(36) Concurrent Cleanup for Next Mark 0.094ms
[0.717s][info   ][gc          ] GC(36) Concurrent Undo Cycle 0.193ms
[0.742s][info   ][gc,start    ] GC(37) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.742s][info   ][gc,task     ] GC(37) Using 8 workers of 8 for evacuation
[0.746s][info   ][gc,phases   ] GC(37)   Pre Evacuate Collection Set: 0.3ms
[0.746s][info   ][gc,phases   ] GC(37)   Merge Heap Roots: 0.1ms
[0.746s][info   ][gc,phases   ] GC(37)   Evacuate Collection Set: 3.7ms
[0.746s][info   ][gc,phases   ] GC(37)   Post Evacuate Collection Set: 0.2ms
[0.746s][info   ][gc,phases   ] GC(37)   Other: 0.1ms
[0.746s][info   ][gc,heap     ] GC(37) Eden regions: 261->0(567)
[0.746s][info   ][gc,heap     ] GC(37) Survivor regions: 100->85(85)
[0.746s][info   ][gc,heap     ] GC(37) Old regions: 573->640
[0.746s][info   ][gc,heap     ] GC(37) Archive regions: 2->2
[0.746s][info   ][gc,heap     ] GC(37) Humongous regions: 346->184
[0.746s][info   ][gc,metaspace] GC(37) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.746s][info   ][gc          ] GC(37) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1280M->909M(2048M) 4.706ms
[0.746s][info   ][gc,cpu      ] GC(37) User=0.02s Sys=0.01s Real=0.00s
[0.746s][info   ][gc          ] GC(38) Concurrent Undo Cycle
[0.747s][info   ][gc,marking  ] GC(38) Concurrent Cleanup for Next Mark
[0.747s][info   ][gc,marking  ] GC(38) Concurrent Cleanup for Next Mark 0.063ms
[0.747s][info   ][gc          ] GC(38) Concurrent Undo Cycle 0.109ms
[0.759s][info   ][gc,start    ] GC(39) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.760s][info   ][gc,task     ] GC(39) Using 8 workers of 8 for evacuation
[0.763s][info   ][gc,phases   ] GC(39)   Pre Evacuate Collection Set: 0.2ms
[0.763s][info   ][gc,phases   ] GC(39)   Merge Heap Roots: 0.1ms
[0.763s][info   ][gc,phases   ] GC(39)   Evacuate Collection Set: 2.5ms
[0.763s][info   ][gc,phases   ] GC(39)   Post Evacuate Collection Set: 0.3ms
[0.763s][info   ][gc,phases   ] GC(39)   Other: 0.1ms
[0.763s][info   ][gc,heap     ] GC(39) Eden regions: 180->0(564)
[0.763s][info   ][gc,heap     ] GC(39) Survivor regions: 85->66(82)
[0.763s][info   ][gc,heap     ] GC(39) Old regions: 640->698
[0.763s][info   ][gc,heap     ] GC(39) Archive regions: 2->2
[0.763s][info   ][gc,heap     ] GC(39) Humongous regions: 279->173
[0.763s][info   ][gc,metaspace] GC(39) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.763s][info   ][gc          ] GC(39) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1183M->937M(2048M) 3.421ms
[0.763s][info   ][gc,cpu      ] GC(39) User=0.01s Sys=0.00s Real=0.00s
[0.763s][info   ][gc          ] GC(40) Concurrent Undo Cycle
[0.763s][info   ][gc,marking  ] GC(40) Concurrent Cleanup for Next Mark
[0.763s][info   ][gc,marking  ] GC(40) Concurrent Cleanup for Next Mark 0.043ms
[0.763s][info   ][gc          ] GC(40) Concurrent Undo Cycle 0.118ms
[0.770s][info   ][gc,start    ] GC(41) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.770s][info   ][gc,task     ] GC(41) Using 8 workers of 8 for evacuation
[0.773s][info   ][gc,phases   ] GC(41)   Pre Evacuate Collection Set: 0.4ms
[0.773s][info   ][gc,phases   ] GC(41)   Merge Heap Roots: 0.1ms
[0.773s][info   ][gc,phases   ] GC(41)   Evacuate Collection Set: 2.3ms
[0.773s][info   ][gc,phases   ] GC(41)   Post Evacuate Collection Set: 0.2ms
[0.773s][info   ][gc,phases   ] GC(41)   Other: 0.1ms
[0.773s][info   ][gc,heap     ] GC(41) Eden regions: 96->0(449)
[0.773s][info   ][gc,heap     ] GC(41) Survivor regions: 66->38(79)
[0.773s][info   ][gc,heap     ] GC(41) Old regions: 698->757
[0.773s][info   ][gc,heap     ] GC(41) Archive regions: 2->2
[0.773s][info   ][gc,heap     ] GC(41) Humongous regions: 221->164
[0.773s][info   ][gc,metaspace] GC(41) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.773s][info   ][gc          ] GC(41) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1080M->958M(2048M) 3.213ms
[0.773s][info   ][gc,cpu      ] GC(41) User=0.00s Sys=0.01s Real=0.00s
[0.773s][info   ][gc          ] GC(42) Concurrent Mark Cycle
[0.773s][info   ][gc,marking  ] GC(42) Concurrent Clear Claimed Marks
[0.773s][info   ][gc,marking  ] GC(42) Concurrent Clear Claimed Marks 0.022ms
[0.774s][info   ][gc,marking  ] GC(42) Concurrent Scan Root Regions
[0.774s][info   ][gc,marking  ] GC(42) Concurrent Scan Root Regions 0.137ms
[0.774s][info   ][gc,marking  ] GC(42) Concurrent Mark
[0.774s][info   ][gc,marking  ] GC(42) Concurrent Mark From Roots
[0.774s][info   ][gc,task     ] GC(42) Using 2 workers of 2 for marking
[0.775s][info   ][gc,marking  ] GC(42) Concurrent Mark From Roots 1.130ms
[0.775s][info   ][gc,marking  ] GC(42) Concurrent Preclean
[0.775s][info   ][gc,marking  ] GC(42) Concurrent Preclean 0.008ms
[0.775s][info   ][gc,start    ] GC(42) Pause Remark
[0.775s][info   ][gc          ] GC(42) Pause Remark 991M->661M(2048M) 0.501ms
[0.775s][info   ][gc,cpu      ] GC(42) User=0.00s Sys=0.00s Real=0.00s
[0.775s][info   ][gc,marking  ] GC(42) Concurrent Mark 1.819ms
[0.775s][info   ][gc,marking  ] GC(42) Concurrent Rebuild Remembered Sets
[0.776s][info   ][gc,marking  ] GC(42) Concurrent Rebuild Remembered Sets 0.885ms
[0.776s][info   ][gc,start    ] GC(42) Pause Cleanup
[0.777s][info   ][gc          ] GC(42) Pause Cleanup 683M->683M(2048M) 0.310ms
[0.777s][info   ][gc,cpu      ] GC(42) User=0.00s Sys=0.00s Real=0.00s
[0.777s][info   ][gc,marking  ] GC(42) Concurrent Cleanup for Next Mark
[0.778s][info   ][gc,marking  ] GC(42) Concurrent Cleanup for Next Mark 1.485ms
[0.778s][info   ][gc          ] GC(42) Concurrent Mark Cycle 4.979ms
[0.807s][info   ][gc,start    ] GC(43) Pause Young (Prepare Mixed) (G1 Evacuation Pause)
[0.807s][info   ][gc,task     ] GC(43) Using 8 workers of 8 for evacuation
[0.811s][info   ][gc,phases   ] GC(43)   Pre Evacuate Collection Set: 0.2ms
[0.811s][info   ][gc,phases   ] GC(43)   Merge Heap Roots: 0.2ms
[0.811s][info   ][gc,phases   ] GC(43)   Evacuate Collection Set: 3.1ms
[0.811s][info   ][gc,phases   ] GC(43)   Post Evacuate Collection Set: 0.3ms
[0.811s][info   ][gc,phases   ] GC(43)   Other: 0.1ms
[0.811s][info   ][gc,heap     ] GC(43) Eden regions: 449->0(41)
[0.811s][info   ][gc,heap     ] GC(43) Survivor regions: 38->61(61)
[0.811s][info   ][gc,heap     ] GC(43) Old regions: 427->520
[0.811s][info   ][gc,heap     ] GC(43) Archive regions: 2->2
[0.811s][info   ][gc,heap     ] GC(43) Humongous regions: 398->163
[0.811s][info   ][gc,metaspace] GC(43) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.811s][info   ][gc          ] GC(43) Pause Young (Prepare Mixed) (G1 Evacuation Pause) 1311M->744M(2048M) 4.128ms
[0.811s][info   ][gc,cpu      ] GC(43) User=0.02s Sys=0.00s Real=0.01s
[0.814s][info   ][gc,start    ] GC(44) Pause Young (Mixed) (G1 Evacuation Pause)
[0.814s][info   ][gc,task     ] GC(44) Using 8 workers of 8 for evacuation
[0.817s][info   ][gc,phases   ] GC(44)   Pre Evacuate Collection Set: 0.1ms
[0.817s][info   ][gc,phases   ] GC(44)   Merge Heap Roots: 0.1ms
[0.817s][info   ][gc,phases   ] GC(44)   Evacuate Collection Set: 1.9ms
[0.817s][info   ][gc,phases   ] GC(44)   Post Evacuate Collection Set: 0.3ms
[0.817s][info   ][gc,phases   ] GC(44)   Other: 0.0ms
[0.817s][info   ][gc,heap     ] GC(44) Eden regions: 41->0(622)
[0.817s][info   ][gc,heap     ] GC(44) Survivor regions: 61->13(13)
[0.817s][info   ][gc,heap     ] GC(44) Old regions: 520->456
[0.817s][info   ][gc,heap     ] GC(44) Archive regions: 2->2
[0.817s][info   ][gc,heap     ] GC(44) Humongous regions: 181->162
[0.817s][info   ][gc,metaspace] GC(44) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.817s][info   ][gc          ] GC(44) Pause Young (Mixed) (G1 Evacuation Pause) 803M->631M(2048M) 2.566ms
[0.817s][info   ][gc,cpu      ] GC(44) User=0.01s Sys=0.01s Real=0.00s
[0.865s][info   ][gc,start    ] GC(45) Pause Young (Normal) (G1 Evacuation Pause)
[0.865s][info   ][gc,task     ] GC(45) Using 8 workers of 8 for evacuation
[0.888s][info   ][gc,phases   ] GC(45)   Pre Evacuate Collection Set: 0.2ms
[0.888s][info   ][gc,phases   ] GC(45)   Merge Heap Roots: 0.2ms
[0.888s][info   ][gc,phases   ] GC(45)   Evacuate Collection Set: 21.7ms
[0.888s][info   ][gc,phases   ] GC(45)   Post Evacuate Collection Set: 0.5ms
[0.888s][info   ][gc,phases   ] GC(45)   Other: 0.1ms
[0.888s][info   ][gc,heap     ] GC(45) Eden regions: 622->0(566)
[0.888s][info   ][gc,heap     ] GC(45) Survivor regions: 13->80(80)
[0.888s][info   ][gc,heap     ] GC(45) Old regions: 456->547
[0.888s][info   ][gc,heap     ] GC(45) Archive regions: 2->2
[0.888s][info   ][gc,heap     ] GC(45) Humongous regions: 525->168
[0.888s][info   ][gc,metaspace] GC(45) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.888s][info   ][gc          ] GC(45) Pause Young (Normal) (G1 Evacuation Pause) 1616M->795M(2048M) 22.840ms
[0.888s][info   ][gc,cpu      ] GC(45) User=0.02s Sys=0.05s Real=0.02s
[0.936s][info   ][gc,start    ] GC(46) Pause Young (Normal) (G1 Evacuation Pause)
[0.936s][info   ][gc,task     ] GC(46) Using 8 workers of 8 for evacuation
[0.953s][info   ][gc,phases   ] GC(46)   Pre Evacuate Collection Set: 0.2ms
[0.953s][info   ][gc,phases   ] GC(46)   Merge Heap Roots: 0.1ms
[0.953s][info   ][gc,phases   ] GC(46)   Evacuate Collection Set: 16.1ms
[0.953s][info   ][gc,phases   ] GC(46)   Post Evacuate Collection Set: 0.6ms
[0.953s][info   ][gc,phases   ] GC(46)   Other: 0.1ms
[0.953s][info   ][gc,heap     ] GC(46) Eden regions: 566->0(506)
[0.953s][info   ][gc,heap     ] GC(46) Survivor regions: 80->81(81)
[0.953s][info   ][gc,heap     ] GC(46) Old regions: 547->653
[0.953s][info   ][gc,heap     ] GC(46) Archive regions: 2->2
[0.953s][info   ][gc,heap     ] GC(46) Humongous regions: 487->173
[0.953s][info   ][gc,metaspace] GC(46) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[0.953s][info   ][gc          ] GC(46) Pause Young (Normal) (G1 Evacuation Pause) 1680M->907M(2048M) 17.409ms
[0.953s][info   ][gc,cpu      ] GC(46) User=0.02s Sys=0.05s Real=0.02s
[0.993s][info   ][gc,start    ] GC(47) Pause Young (Concurrent Start) (G1 Humongous Allocation)
[0.993s][info   ][gc,task     ] GC(47) Using 8 workers of 8 for evacuation
[1.008s][info   ][gc,phases   ] GC(47)   Pre Evacuate Collection Set: 0.2ms
[1.008s][info   ][gc,phases   ] GC(47)   Merge Heap Roots: 0.1ms
[1.008s][info   ][gc,phases   ] GC(47)   Evacuate Collection Set: 13.5ms
[1.008s][info   ][gc,phases   ] GC(47)   Post Evacuate Collection Set: 0.4ms
[1.008s][info   ][gc,phases   ] GC(47)   Other: 0.1ms
[1.008s][info   ][gc,heap     ] GC(47) Eden regions: 489->0(441)
[1.008s][info   ][gc,heap     ] GC(47) Survivor regions: 81->74(74)
[1.008s][info   ][gc,heap     ] GC(47) Old regions: 653->755
[1.008s][info   ][gc,heap     ] GC(47) Archive regions: 2->2
[1.008s][info   ][gc,heap     ] GC(47) Humongous regions: 460->205
[1.008s][info   ][gc,metaspace] GC(47) Metaspace: 156K(384K)->156K(384K) NonClass: 150K(256K)->150K(256K) Class: 5K(128K)->5K(128K)
[1.008s][info   ][gc          ] GC(47) Pause Young (Concurrent Start) (G1 Humongous Allocation) 1683M->1034M(2048M) 14.430ms
[1.008s][info   ][gc,cpu      ] GC(47) User=0.01s Sys=0.04s Real=0.01s
[1.008s][info   ][gc          ] GC(48) Concurrent Undo Cycle
[1.008s][info   ][gc,marking  ] GC(48) Concurrent Cleanup for Next Mark
[1.008s][info   ][gc,marking  ] GC(48) Concurrent Cleanup for Next Mark 0.182ms
[1.008s][info   ][gc          ] GC(48) Concurrent Undo Cycle 0.325ms
counter:56583
[1.025s][info   ][gc,heap,exit] Heap
[1.025s][info   ][gc,heap,exit]  garbage-first heap   total 2097152K, used 1275412K [0x0000000780000000, 0x0000000800000000)
[1.025s][info   ][gc,heap,exit]   region size 1024K, 211 young (216064K), 74 survivors (75776K)
[1.025s][info   ][gc,heap,exit]  Metaspace       used 236K, committed 448K, reserved 1114112K
[1.025s][info   ][gc,heap,exit]   class space    used 9K, committed 128K, reserved 1048576K
```


#### ZGC

专注于低延迟，目标是将 GC 停顿时间控制在 10ms 以下。支持非常大的堆内存（TB 级）。使用多线程并发回收，避免长时间的 STW。基于标记-整理算法，避免内存碎片化。使用指针染色（Pointer Coloring）来实现并发标记和引用更新。

-XX: +UnlockExpermentalVMOptions
-XX: +UseZGC
-XX: -Xmx16g

优点：
- 极低的 GC 停顿时间。
- 支持超大堆，扩展性好。
- 减少内存碎片。
- 与G1相比，应用吞吐量下降不超过15%

缺点：
- 内存占用较高：由于需要染色指针和写屏障。
- 不适合资源紧张的环境。

-XX: +UseShenandoahGC

立项比ZGC早，暂停时间与堆大小无关

适用场景：
- 延迟敏感的大型应用（如金融交易、高并发系统）。
- 超大堆应用（TB 级别内存）。

接下来试试 ZGC 128m内存
```
java -XX:+UseZGC -Xms128m -Xmx128m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

可以看到GC12次就出现了OOM，花了0.157s。和并行GC接近，比G1GC的GC次数少
```
[0.002s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.005s][info   ][gc,init] Initializing The Z Garbage Collector
[0.006s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.006s][info   ][gc,init] NUMA Support: Disabled
[0.006s][info   ][gc,init] CPUs: 8 total, 8 available
[0.006s][info   ][gc,init] Memory: 16384M
[0.006s][info   ][gc,init] Large Page Support: Disabled
[0.006s][info   ][gc,init] GC Workers: 1 (dynamic)
[0.006s][info   ][gc,init] Address Space Type: Contiguous/Unrestricted/Complete
[0.006s][info   ][gc,init] Address Space Size: 2048M x 3 = 6144M
[0.006s][info   ][gc,init] Min Capacity: 128M
[0.006s][info   ][gc,init] Initial Capacity: 128M
[0.006s][info   ][gc,init] Max Capacity: 128M
[0.006s][info   ][gc,init] Medium Page Size: 4M
[0.006s][info   ][gc,init] Pre-touch: Disabled
[0.006s][info   ][gc,init] Uncommit: Implicitly Disabled (-Xms equals -Xmx)
[0.006s][info   ][gc,init] Runtime Workers: 1
[0.006s][info   ][gc     ] Using The Z Garbage Collector
[0.006s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.006s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.006s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.083s][info   ][gc,start    ] GC(0) Garbage Collection (Allocation Stall)
[0.083s][info   ][gc,ref      ] GC(0) Clearing All SoftReferences
[0.083s][info   ][gc,task     ] GC(0) Using 1 workers
[0.084s][info   ][gc,phases   ] GC(0) Pause Mark Start 0.012ms
[0.087s][info   ][gc,phases   ] GC(0) Concurrent Mark 3.476ms
[0.088s][info   ][gc,phases   ] GC(0) Pause Mark End 0.012ms
[0.088s][info   ][gc,phases   ] GC(0) Concurrent Mark Free 0.001ms
[0.088s][info   ][gc,phases   ] GC(0) Concurrent Process Non-Strong References 0.411ms
[0.088s][info   ][gc,phases   ] GC(0) Concurrent Reset Relocation Set 0.000ms
[0.095s][info   ][gc          ] Allocation Stall (main) 11.982ms
[0.095s][info   ][gc,phases   ] GC(0) Concurrent Select Relocation Set 7.184ms
[0.096s][info   ][gc,phases   ] GC(0) Pause Relocate Start 0.006ms
[0.103s][info   ][gc,phases   ] GC(0) Concurrent Relocate 6.950ms
[0.103s][info   ][gc,load     ] GC(0) Load: 4.33/4.72/4.91
[0.103s][info   ][gc,mmu      ] GC(0) MMU: 2ms/99.4%, 5ms/99.5%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.103s][info   ][gc,marking  ] GC(0) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.103s][info   ][gc,marking  ] GC(0) Mark Stack Usage: 32M
[0.103s][info   ][gc,nmethod  ] GC(0) NMethods: 112 registered, 0 unregistered
[0.103s][info   ][gc,metaspace] GC(0) Metaspace: 0M used, 0M committed, 1088M reserved
[0.103s][info   ][gc,ref      ] GC(0) Soft: 5 encountered, 3 discovered, 2 enqueued
[0.103s][info   ][gc,ref      ] GC(0) Weak: 26 encountered, 19 discovered, 3 enqueued
[0.103s][info   ][gc,ref      ] GC(0) Final: 0 encountered, 0 discovered, 0 enqueued
[0.103s][info   ][gc,ref      ] GC(0) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.103s][info   ][gc,reloc    ] GC(0) Small Pages: 20 / 40M, Empty: 0M, Relocated: 12M, In-Place: 0
[0.103s][info   ][gc,reloc    ] GC(0) Medium Pages: 5 / 20M, Empty: 0M, Relocated: 5M, In-Place: 0
[0.103s][info   ][gc,reloc    ] GC(0) Large Pages: 34 / 68M, Empty: 38M, Relocated: 0M, In-Place: 0
[0.103s][info   ][gc,reloc    ] GC(0) Forwarding Usage: 0M
[0.103s][info   ][gc,heap     ] GC(0) Min Capacity: 128M(100%)
[0.103s][info   ][gc,heap     ] GC(0) Max Capacity: 128M(100%)
[0.103s][info   ][gc,heap     ] GC(0) Soft Max Capacity: 128M(100%)
[0.103s][info   ][gc,heap     ] GC(0)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.103s][info   ][gc,heap     ] GC(0)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.103s][info   ][gc,heap     ] GC(0)      Free:        0M (0%)            0M (0%)           36M (28%)          18M (14%)          42M (33%)           0M (0%)
[0.103s][info   ][gc,heap     ] GC(0)      Used:      128M (100%)        128M (100%)         92M (72%)         110M (86%)         128M (100%)         86M (67%)
[0.103s][info   ][gc,heap     ] GC(0)      Live:         -                49M (39%)          49M (39%)          49M (39%)            -                  -
[0.103s][info   ][gc,heap     ] GC(0) Allocated:         -                 0M (0%)            2M (2%)           53M (42%)            -                  -
[0.103s][info   ][gc,heap     ] GC(0)   Garbage:         -                78M (61%)          40M (31%)           6M (5%)             -                  -
[0.103s][info   ][gc,heap     ] GC(0) Reclaimed:         -                  -                38M (30%)          71M (56%)            -                  -
[0.103s][info   ][gc          ] GC(0) Garbage Collection (Allocation Stall) 128M(100%)->110M(86%)
[0.105s][info   ][gc,start    ] GC(1) Garbage Collection (Allocation Stall)
[0.105s][info   ][gc,ref      ] GC(1) Clearing All SoftReferences
[0.105s][info   ][gc,task     ] GC(1) Using 1 workers
[0.105s][info   ][gc,phases   ] GC(1) Pause Mark Start 0.003ms
[0.107s][info   ][gc,phases   ] GC(1) Concurrent Mark 2.000ms
[0.107s][info   ][gc,phases   ] GC(1) Pause Mark End 0.005ms
[0.107s][info   ][gc,phases   ] GC(1) Concurrent Mark Free 0.000ms
[0.108s][info   ][gc,phases   ] GC(1) Concurrent Process Non-Strong References 0.234ms
[0.108s][info   ][gc,phases   ] GC(1) Concurrent Reset Relocation Set 0.001ms
[0.111s][info   ][gc,phases   ] GC(1) Concurrent Select Relocation Set 3.769ms
[0.111s][info   ][gc          ] Allocation Stall (main) 6.277ms
[0.112s][info   ][gc,phases   ] GC(1) Pause Relocate Start 0.006ms
[0.113s][info   ][gc,phases   ] GC(1) Concurrent Relocate 1.459ms
[0.113s][info   ][gc,load     ] GC(1) Load: 4.33/4.72/4.91
[0.113s][info   ][gc,mmu      ] GC(1) MMU: 2ms/99.4%, 5ms/99.5%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.113s][info   ][gc,marking  ] GC(1) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.113s][info   ][gc,marking  ] GC(1) Mark Stack Usage: 32M
[0.113s][info   ][gc,nmethod  ] GC(1) NMethods: 112 registered, 0 unregistered
[0.113s][info   ][gc,metaspace] GC(1) Metaspace: 0M used, 0M committed, 1088M reserved
[0.113s][info   ][gc,ref      ] GC(1) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.113s][info   ][gc,ref      ] GC(1) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.113s][info   ][gc,ref      ] GC(1) Final: 0 encountered, 0 discovered, 0 enqueued
[0.113s][info   ][gc,ref      ] GC(1) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.113s][info   ][gc,reloc    ] GC(1) Small Pages: 16 / 32M, Empty: 0M, Relocated: 6M, In-Place: 0
[0.113s][info   ][gc,reloc    ] GC(1) Medium Pages: 6 / 24M, Empty: 4M, Relocated: 5M, In-Place: 0
[0.113s][info   ][gc,reloc    ] GC(1) Large Pages: 36 / 72M, Empty: 26M, Relocated: 0M, In-Place: 0
[0.113s][info   ][gc,reloc    ] GC(1) Forwarding Usage: 0M
[0.113s][info   ][gc,heap     ] GC(1) Min Capacity: 128M(100%)
[0.113s][info   ][gc,heap     ] GC(1) Max Capacity: 128M(100%)
[0.113s][info   ][gc,heap     ] GC(1) Soft Max Capacity: 128M(100%)
[0.113s][info   ][gc,heap     ] GC(1)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.113s][info   ][gc,heap     ] GC(1)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.113s][info   ][gc,heap     ] GC(1)      Free:        0M (0%)            0M (0%)           28M (22%)          24M (19%)          30M (23%)           0M (0%)
[0.113s][info   ][gc,heap     ] GC(1)      Used:      128M (100%)        128M (100%)        100M (78%)         104M (81%)         128M (100%)         98M (77%)
[0.114s][info   ][gc,heap     ] GC(1)      Live:         -                75M (59%)          75M (59%)          75M (59%)            -                  -
[0.114s][info   ][gc,heap     ] GC(1) Allocated:         -                 0M (0%)            2M (2%)           21M (17%)            -                  -
[0.114s][info   ][gc,heap     ] GC(1)   Garbage:         -                52M (41%)          22M (17%)           6M (5%)             -                  -
[0.114s][info   ][gc,heap     ] GC(1) Reclaimed:         -                  -                30M (23%)          45M (36%)            -                  -
[0.114s][info   ][gc          ] GC(1) Garbage Collection (Allocation Stall) 128M(100%)->104M(81%)
[0.115s][info   ][gc,start    ] GC(2) Garbage Collection (Allocation Stall)
[0.115s][info   ][gc,ref      ] GC(2) Clearing All SoftReferences
[0.115s][info   ][gc,task     ] GC(2) Using 1 workers
[0.115s][info   ][gc,phases   ] GC(2) Pause Mark Start 0.003ms
[0.116s][info   ][gc,phases   ] GC(2) Concurrent Mark 1.023ms
[0.116s][info   ][gc,phases   ] GC(2) Pause Mark End 0.007ms
[0.116s][info   ][gc,phases   ] GC(2) Concurrent Mark Free 0.000ms
[0.116s][info   ][gc,phases   ] GC(2) Concurrent Process Non-Strong References 0.207ms
[0.116s][info   ][gc,phases   ] GC(2) Concurrent Reset Relocation Set 0.001ms
[0.118s][info   ][gc          ] Allocation Stall (main) 2.970ms
[0.118s][info   ][gc,phases   ] GC(2) Concurrent Select Relocation Set 1.470ms
[0.118s][info   ][gc,phases   ] GC(2) Pause Relocate Start 0.005ms
[0.119s][info   ][gc,phases   ] GC(2) Concurrent Relocate 1.146ms
[0.119s][info   ][gc,load     ] GC(2) Load: 4.33/4.72/4.91
[0.119s][info   ][gc,mmu      ] GC(2) MMU: 2ms/99.4%, 5ms/99.5%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.119s][info   ][gc,marking  ] GC(2) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.119s][info   ][gc,marking  ] GC(2) Mark Stack Usage: 32M
[0.119s][info   ][gc,nmethod  ] GC(2) NMethods: 115 registered, 0 unregistered
[0.119s][info   ][gc,metaspace] GC(2) Metaspace: 0M used, 0M committed, 1088M reserved
[0.119s][info   ][gc,ref      ] GC(2) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.119s][info   ][gc,ref      ] GC(2) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.119s][info   ][gc,ref      ] GC(2) Final: 0 encountered, 0 discovered, 0 enqueued
[0.120s][info   ][gc,ref      ] GC(2) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.120s][info   ][gc,reloc    ] GC(2) Small Pages: 15 / 30M, Empty: 0M, Relocated: 2M, In-Place: 0
[0.120s][info   ][gc,reloc    ] GC(2) Medium Pages: 5 / 20M, Empty: 0M, Relocated: 6M, In-Place: 0
[0.120s][info   ][gc,reloc    ] GC(2) Large Pages: 39 / 78M, Empty: 20M, Relocated: 0M, In-Place: 0
[0.120s][info   ][gc,reloc    ] GC(2) Forwarding Usage: 0M
[0.120s][info   ][gc,heap     ] GC(2) Min Capacity: 128M(100%)
[0.120s][info   ][gc,heap     ] GC(2) Max Capacity: 128M(100%)
[0.120s][info   ][gc,heap     ] GC(2) Soft Max Capacity: 128M(100%)
[0.120s][info   ][gc,heap     ] GC(2)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.120s][info   ][gc,heap     ] GC(2)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.120s][info   ][gc,heap     ] GC(2)      Free:        0M (0%)            0M (0%)           14M (11%)          22M (17%)          22M (17%)           0M (0%)
[0.120s][info   ][gc,heap     ] GC(2)      Used:      128M (100%)        128M (100%)        114M (89%)         106M (83%)         128M (100%)        106M (83%)
[0.120s][info   ][gc,heap     ] GC(2)      Live:         -                92M (72%)          92M (72%)          92M (72%)            -                  -
[0.120s][info   ][gc,heap     ] GC(2) Allocated:         -                 0M (0%)            6M (5%)            7M (6%)             -                  -
[0.120s][info   ][gc,heap     ] GC(2)   Garbage:         -                35M (28%)          15M (12%)           5M (4%)             -                  -
[0.120s][info   ][gc,heap     ] GC(2) Reclaimed:         -                  -                20M (16%)          29M (23%)            -                  -
[0.120s][info   ][gc          ] GC(2) Garbage Collection (Allocation Stall) 128M(100%)->106M(83%)
[0.121s][info   ][gc,start    ] GC(3) Garbage Collection (Allocation Stall)
[0.121s][info   ][gc,ref      ] GC(3) Clearing All SoftReferences
[0.121s][info   ][gc,task     ] GC(3) Using 1 workers
[0.121s][info   ][gc,phases   ] GC(3) Pause Mark Start 0.002ms
[0.122s][info   ][gc,phases   ] GC(3) Concurrent Mark 1.059ms
[0.122s][info   ][gc,phases   ] GC(3) Pause Mark End 0.008ms
[0.122s][info   ][gc,phases   ] GC(3) Concurrent Mark Free 0.000ms
[0.122s][info   ][gc,phases   ] GC(3) Concurrent Process Non-Strong References 0.210ms
[0.122s][info   ][gc,phases   ] GC(3) Concurrent Reset Relocation Set 0.001ms
[0.124s][info   ][gc          ] Allocation Stall (main) 2.902ms
[0.124s][info   ][gc,phases   ] GC(3) Concurrent Select Relocation Set 1.414ms
[0.124s][info   ][gc,phases   ] GC(3) Pause Relocate Start 0.007ms
[0.124s][info   ][gc          ] Allocation Stall (main) 0.077ms
[0.124s][info   ][gc,phases   ] GC(3) Concurrent Relocate 0.774ms
[0.125s][info   ][gc,load     ] GC(3) Load: 4.33/4.72/4.91
[0.125s][info   ][gc,mmu      ] GC(3) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.8%, 50ms/99.8%, 100ms/99.9%
[0.125s][info   ][gc,marking  ] GC(3) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.125s][info   ][gc,marking  ] GC(3) Mark Stack Usage: 32M
[0.125s][info   ][gc,nmethod  ] GC(3) NMethods: 117 registered, 0 unregistered
[0.125s][info   ][gc,metaspace] GC(3) Metaspace: 0M used, 0M committed, 1088M reserved
[0.125s][info   ][gc,ref      ] GC(3) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.125s][info   ][gc,ref      ] GC(3) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.125s][info   ][gc,ref      ] GC(3) Final: 0 encountered, 0 discovered, 0 enqueued
[0.125s][info   ][gc,ref      ] GC(3) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.125s][info   ][gc,reloc    ] GC(3) Small Pages: 18 / 36M, Empty: 0M, Relocated: 3M, In-Place: 0
[0.125s][info   ][gc,reloc    ] GC(3) Medium Pages: 6 / 24M, Empty: 0M, Relocated: 2M, In-Place: 0
[0.125s][info   ][gc,reloc    ] GC(3) Large Pages: 34 / 68M, Empty: 8M, Relocated: 0M, In-Place: 0
[0.125s][info   ][gc,reloc    ] GC(3) Forwarding Usage: 0M
[0.125s][info   ][gc,heap     ] GC(3) Min Capacity: 128M(100%)
[0.125s][info   ][gc,heap     ] GC(3) Max Capacity: 128M(100%)
[0.125s][info   ][gc,heap     ] GC(3) Soft Max Capacity: 128M(100%)
[0.125s][info   ][gc,heap     ] GC(3)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.125s][info   ][gc,heap     ] GC(3)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.125s][info   ][gc,heap     ] GC(3)      Free:        0M (0%)            0M (0%)            6M (5%)            6M (5%)            8M (6%)            0M (0%)
[0.125s][info   ][gc,heap     ] GC(3)      Used:      128M (100%)        128M (100%)        122M (95%)         122M (95%)         128M (100%)        120M (94%)
[0.125s][info   ][gc,heap     ] GC(3)      Live:         -                99M (78%)          99M (78%)          99M (78%)            -                  -
[0.125s][info   ][gc,heap     ] GC(3) Allocated:         -                 0M (0%)            2M (2%)           15M (12%)            -                  -
[0.125s][info   ][gc,heap     ] GC(3)   Garbage:         -                28M (22%)          20M (16%)           6M (5%)             -                  -
[0.125s][info   ][gc,heap     ] GC(3) Reclaimed:         -                  -                 8M (6%)           21M (17%)            -                  -
[0.125s][info   ][gc          ] GC(3) Garbage Collection (Allocation Stall) 128M(100%)->122M(95%)
[0.125s][info   ][gc,start    ] GC(4) Garbage Collection (Allocation Stall)
[0.125s][info   ][gc,ref      ] GC(4) Clearing All SoftReferences
[0.125s][info   ][gc,task     ] GC(4) Using 1 workers
[0.125s][info   ][gc,phases   ] GC(4) Pause Mark Start 0.002ms
[0.126s][info   ][gc,phases   ] GC(4) Concurrent Mark 0.862ms
[0.126s][info   ][gc,phases   ] GC(4) Pause Mark End 0.003ms
[0.126s][info   ][gc,phases   ] GC(4) Concurrent Mark Free 0.000ms
[0.126s][info   ][gc,phases   ] GC(4) Concurrent Process Non-Strong References 0.253ms
[0.126s][info   ][gc,phases   ] GC(4) Concurrent Reset Relocation Set 0.001ms
[0.128s][info   ][gc          ] Allocation Stall (main) 2.716ms
[0.128s][info   ][gc,phases   ] GC(4) Concurrent Select Relocation Set 1.445ms
[0.128s][info   ][gc,phases   ] GC(4) Pause Relocate Start 0.005ms
[0.128s][info   ][gc,phases   ] GC(4) Concurrent Relocate 0.679ms
[0.128s][info   ][gc,load     ] GC(4) Load: 4.33/4.72/4.91
[0.128s][info   ][gc,mmu      ] GC(4) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.8%, 50ms/99.8%, 100ms/99.9%
[0.128s][info   ][gc,marking  ] GC(4) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.128s][info   ][gc,marking  ] GC(4) Mark Stack Usage: 32M
[0.128s][info   ][gc,nmethod  ] GC(4) NMethods: 117 registered, 0 unregistered
[0.129s][info   ][gc,metaspace] GC(4) Metaspace: 0M used, 0M committed, 1088M reserved
[0.129s][info   ][gc,ref      ] GC(4) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.129s][info   ][gc,ref      ] GC(4) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.129s][info   ][gc,ref      ] GC(4) Final: 0 encountered, 0 discovered, 0 enqueued
[0.129s][info   ][gc,ref      ] GC(4) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.129s][info   ][gc,reloc    ] GC(4) Small Pages: 15 / 30M, Empty: 0M, Relocated: 1M, In-Place: 0
[0.129s][info   ][gc,reloc    ] GC(4) Medium Pages: 6 / 24M, Empty: 0M, Relocated: 3M, In-Place: 0
[0.129s][info   ][gc,reloc    ] GC(4) Large Pages: 37 / 74M, Empty: 12M, Relocated: 0M, In-Place: 0
[0.129s][info   ][gc,reloc    ] GC(4) Forwarding Usage: 0M
[0.129s][info   ][gc,heap     ] GC(4) Min Capacity: 128M(100%)
[0.129s][info   ][gc,heap     ] GC(4) Max Capacity: 128M(100%)
[0.129s][info   ][gc,heap     ] GC(4) Soft Max Capacity: 128M(100%)
[0.129s][info   ][gc,heap     ] GC(4)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.129s][info   ][gc,heap     ] GC(4)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.129s][info   ][gc,heap     ] GC(4)      Free:        0M (0%)            0M (0%)           10M (8%)           10M (8%)           12M (9%)            0M (0%)
[0.129s][info   ][gc,heap     ] GC(4)      Used:      128M (100%)        128M (100%)        118M (92%)         118M (92%)         128M (100%)        116M (91%)
[0.129s][info   ][gc,heap     ] GC(4)      Live:         -               102M (80%)         102M (80%)         102M (80%)            -                  -
[0.129s][info   ][gc,heap     ] GC(4) Allocated:         -                 0M (0%)            2M (2%)            7M (6%)             -                  -
[0.129s][info   ][gc,heap     ] GC(4)   Garbage:         -                25M (20%)          13M (10%)           7M (6%)             -                  -
[0.129s][info   ][gc,heap     ] GC(4) Reclaimed:         -                  -                12M (9%)           17M (14%)            -                  -
[0.129s][info   ][gc          ] GC(4) Garbage Collection (Allocation Stall) 128M(100%)->118M(92%)
[0.129s][info   ][gc,start    ] GC(5) Garbage Collection (Allocation Stall)
[0.129s][info   ][gc,ref      ] GC(5) Clearing All SoftReferences
[0.129s][info   ][gc,task     ] GC(5) Using 1 workers
[0.129s][info   ][gc,phases   ] GC(5) Pause Mark Start 0.002ms
[0.130s][info   ][gc,phases   ] GC(5) Concurrent Mark 0.898ms
[0.130s][info   ][gc,phases   ] GC(5) Pause Mark End 0.003ms
[0.130s][info   ][gc,phases   ] GC(5) Concurrent Mark Free 0.000ms
[0.130s][info   ][gc,phases   ] GC(5) Concurrent Process Non-Strong References 0.200ms
[0.130s][info   ][gc,phases   ] GC(5) Concurrent Reset Relocation Set 0.001ms
[0.132s][info   ][gc          ] Allocation Stall (main) 2.735ms
[0.132s][info   ][gc,phases   ] GC(5) Concurrent Select Relocation Set 1.510ms
[0.132s][info   ][gc,phases   ] GC(5) Pause Relocate Start 0.004ms
[0.132s][info   ][gc,phases   ] GC(5) Concurrent Relocate 0.195ms
[0.132s][info   ][gc,load     ] GC(5) Load: 4.33/4.72/4.91
[0.132s][info   ][gc,mmu      ] GC(5) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.9%
[0.132s][info   ][gc,marking  ] GC(5) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.132s][info   ][gc,marking  ] GC(5) Mark Stack Usage: 32M
[0.132s][info   ][gc,nmethod  ] GC(5) NMethods: 117 registered, 0 unregistered
[0.132s][info   ][gc,metaspace] GC(5) Metaspace: 0M used, 0M committed, 1088M reserved
[0.132s][info   ][gc,ref      ] GC(5) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.132s][info   ][gc,ref      ] GC(5) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.132s][info   ][gc,ref      ] GC(5) Final: 0 encountered, 0 discovered, 0 enqueued
[0.132s][info   ][gc,ref      ] GC(5) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.132s][info   ][gc,reloc    ] GC(5) Small Pages: 17 / 34M, Empty: 0M, Relocated: 1M, In-Place: 0
[0.132s][info   ][gc,reloc    ] GC(5) Medium Pages: 6 / 24M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.132s][info   ][gc,reloc    ] GC(5) Large Pages: 35 / 70M, Empty: 10M, Relocated: 0M, In-Place: 0
[0.132s][info   ][gc,reloc    ] GC(5) Forwarding Usage: 0M
[0.133s][info   ][gc,heap     ] GC(5) Min Capacity: 128M(100%)
[0.133s][info   ][gc,heap     ] GC(5) Max Capacity: 128M(100%)
[0.133s][info   ][gc,heap     ] GC(5) Soft Max Capacity: 128M(100%)
[0.133s][info   ][gc,heap     ] GC(5)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.133s][info   ][gc,heap     ] GC(5)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.133s][info   ][gc,heap     ] GC(5)      Free:        0M (0%)            0M (0%)            8M (6%)            6M (5%)           10M (8%)            0M (0%)
[0.133s][info   ][gc,heap     ] GC(5)      Used:      128M (100%)        128M (100%)        120M (94%)         122M (95%)         128M (100%)        118M (92%)
[0.133s][info   ][gc,heap     ] GC(5)      Live:         -               104M (82%)         104M (82%)         104M (82%)            -                  -
[0.133s][info   ][gc,heap     ] GC(5) Allocated:         -                 0M (0%)            2M (2%)            7M (6%)             -                  -
[0.133s][info   ][gc,heap     ] GC(5)   Garbage:         -                23M (18%)          13M (10%)           9M (7%)             -                  -
[0.133s][info   ][gc,heap     ] GC(5) Reclaimed:         -                  -                10M (8%)           13M (11%)            -                  -
[0.133s][info   ][gc          ] GC(5) Garbage Collection (Allocation Stall) 128M(100%)->122M(95%)
[0.133s][info   ][gc,start    ] GC(6) Garbage Collection (Allocation Stall)
[0.133s][info   ][gc,ref      ] GC(6) Clearing All SoftReferences
[0.133s][info   ][gc,task     ] GC(6) Using 1 workers
[0.133s][info   ][gc,phases   ] GC(6) Pause Mark Start 0.002ms
[0.134s][info   ][gc,phases   ] GC(6) Concurrent Mark 0.775ms
[0.134s][info   ][gc,phases   ] GC(6) Pause Mark End 0.003ms
[0.134s][info   ][gc,phases   ] GC(6) Concurrent Mark Free 0.001ms
[0.134s][info   ][gc,phases   ] GC(6) Concurrent Process Non-Strong References 0.216ms
[0.134s][info   ][gc,phases   ] GC(6) Concurrent Reset Relocation Set 0.001ms
[0.135s][info   ][gc          ] Allocation Stall (main) 2.571ms
[0.135s][info   ][gc,phases   ] GC(6) Concurrent Select Relocation Set 1.416ms
[0.135s][info   ][gc,phases   ] GC(6) Pause Relocate Start 0.005ms
[0.136s][info   ][gc          ] Relocation Stall (main) 0.345ms
[0.136s][info   ][gc,phases   ] GC(6) Concurrent Relocate 0.479ms
[0.136s][info   ][gc,load     ] GC(6) Load: 4.33/4.72/4.91
[0.136s][info   ][gc,mmu      ] GC(6) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.9%
[0.136s][info   ][gc,marking  ] GC(6) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.136s][info   ][gc,marking  ] GC(6) Mark Stack Usage: 32M
[0.136s][info   ][gc,nmethod  ] GC(6) NMethods: 117 registered, 0 unregistered
[0.136s][info   ][gc,metaspace] GC(6) Metaspace: 0M used, 0M committed, 1088M reserved
[0.136s][info   ][gc,ref      ] GC(6) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.136s][info   ][gc,ref      ] GC(6) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.136s][info   ][gc,ref      ] GC(6) Final: 0 encountered, 0 discovered, 0 enqueued
[0.136s][info   ][gc,ref      ] GC(6) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.136s][info   ][gc,reloc    ] GC(6) Small Pages: 16 / 32M, Empty: 0M, Relocated: 1M, In-Place: 0
[0.136s][info   ][gc,reloc    ] GC(6) Medium Pages: 7 / 28M, Empty: 0M, Relocated: 3M, In-Place: 1
[0.136s][info   ][gc,reloc    ] GC(6) Large Pages: 34 / 68M, Empty: 2M, Relocated: 0M, In-Place: 0
[0.136s][info   ][gc,reloc    ] GC(6) Forwarding Usage: 0M
[0.136s][info   ][gc,heap     ] GC(6) Min Capacity: 128M(100%)
[0.136s][info   ][gc,heap     ] GC(6) Max Capacity: 128M(100%)
[0.136s][info   ][gc,heap     ] GC(6) Soft Max Capacity: 128M(100%)
[0.136s][info   ][gc,heap     ] GC(6)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.136s][info   ][gc,heap     ] GC(6)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.136s][info   ][gc,heap     ] GC(6)      Free:        0M (0%)            0M (0%)            0M (0%)            6M (5%)            6M (5%)            0M (0%)
[0.136s][info   ][gc,heap     ] GC(6)      Used:      128M (100%)        128M (100%)        128M (100%)        122M (95%)         128M (100%)        122M (95%)
[0.136s][info   ][gc,heap     ] GC(6)      Live:         -               111M (87%)         111M (87%)         111M (87%)            -                  -
[0.136s][info   ][gc,heap     ] GC(6) Allocated:         -                 0M (0%)            2M (2%)            2M (2%)             -                  -
[0.136s][info   ][gc,heap     ] GC(6)   Garbage:         -                16M (13%)          14M (11%)           8M (7%)             -                  -
[0.136s][info   ][gc,heap     ] GC(6) Reclaimed:         -                  -                 2M (2%)            8M (6%)             -                  -
[0.136s][info   ][gc          ] GC(6) Garbage Collection (Allocation Stall) 128M(100%)->122M(95%)
[0.136s][info   ][gc,start    ] GC(7) Garbage Collection (Allocation Stall)
[0.136s][info   ][gc,ref      ] GC(7) Clearing All SoftReferences
[0.136s][info   ][gc,task     ] GC(7) Using 1 workers
[0.136s][info   ][gc,phases   ] GC(7) Pause Mark Start 0.002ms
[0.137s][info   ][gc,phases   ] GC(7) Concurrent Mark 0.866ms
[0.137s][info   ][gc,phases   ] GC(7) Pause Mark End 0.005ms
[0.137s][info   ][gc,phases   ] GC(7) Concurrent Mark Free 0.000ms
[0.138s][info   ][gc,phases   ] GC(7) Concurrent Process Non-Strong References 0.238ms
[0.138s][info   ][gc,phases   ] GC(7) Concurrent Reset Relocation Set 0.001ms
[0.139s][info   ][gc          ] Allocation Stall (main) 2.866ms
[0.139s][info   ][gc,phases   ] GC(7) Concurrent Select Relocation Set 1.510ms
[0.139s][info   ][gc,phases   ] GC(7) Pause Relocate Start 0.003ms
[0.140s][info   ][gc          ] Allocation Stall (main) 0.201ms
[0.140s][info   ][gc,phases   ] GC(7) Concurrent Relocate 0.292ms
[0.140s][info   ][gc,load     ] GC(7) Load: 4.33/4.72/4.91
[0.140s][info   ][gc,mmu      ] GC(7) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.9%
[0.140s][info   ][gc,marking  ] GC(7) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.140s][info   ][gc,marking  ] GC(7) Mark Stack Usage: 32M
[0.140s][info   ][gc,nmethod  ] GC(7) NMethods: 117 registered, 0 unregistered
[0.140s][info   ][gc,metaspace] GC(7) Metaspace: 0M used, 0M committed, 1088M reserved
[0.140s][info   ][gc,ref      ] GC(7) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.140s][info   ][gc,ref      ] GC(7) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.140s][info   ][gc,ref      ] GC(7) Final: 0 encountered, 0 discovered, 0 enqueued
[0.140s][info   ][gc,ref      ] GC(7) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.140s][info   ][gc,reloc    ] GC(7) Small Pages: 16 / 32M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.140s][info   ][gc,reloc    ] GC(7) Medium Pages: 7 / 28M, Empty: 0M, Relocated: 3M, In-Place: 1
[0.140s][info   ][gc,reloc    ] GC(7) Large Pages: 34 / 68M, Empty: 2M, Relocated: 0M, In-Place: 0
[0.140s][info   ][gc,reloc    ] GC(7) Forwarding Usage: 0M
[0.140s][info   ][gc,heap     ] GC(7) Min Capacity: 128M(100%)
[0.140s][info   ][gc,heap     ] GC(7) Max Capacity: 128M(100%)
[0.140s][info   ][gc,heap     ] GC(7) Soft Max Capacity: 128M(100%)
[0.140s][info   ][gc,heap     ] GC(7)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.140s][info   ][gc,heap     ] GC(7)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.140s][info   ][gc,heap     ] GC(7)      Free:        0M (0%)            0M (0%)            0M (0%)            2M (2%)            4M (3%)            0M (0%)
[0.140s][info   ][gc,heap     ] GC(7)      Used:      128M (100%)        128M (100%)        128M (100%)        126M (98%)         128M (100%)        124M (97%)
[0.140s][info   ][gc,heap     ] GC(7)      Live:         -               112M (88%)         112M (88%)         112M (88%)            -                  -
[0.140s][info   ][gc,heap     ] GC(7) Allocated:         -                 0M (0%)            2M (2%)            4M (3%)             -                  -
[0.140s][info   ][gc,heap     ] GC(7)   Garbage:         -                15M (12%)          13M (11%)           9M (8%)             -                  -
[0.140s][info   ][gc,heap     ] GC(7) Reclaimed:         -                  -                 2M (2%)            6M (5%)             -                  -
[0.140s][info   ][gc          ] GC(7) Garbage Collection (Allocation Stall) 128M(100%)->126M(98%)
[0.140s][info   ][gc,start    ] GC(8) Garbage Collection (Allocation Stall)
[0.140s][info   ][gc,ref      ] GC(8) Clearing All SoftReferences
[0.140s][info   ][gc,task     ] GC(8) Using 1 workers
[0.140s][info   ][gc,phases   ] GC(8) Pause Mark Start 0.002ms
[0.141s][info   ][gc,phases   ] GC(8) Concurrent Mark 0.721ms
[0.141s][info   ][gc,phases   ] GC(8) Pause Mark End 0.002ms
[0.141s][info   ][gc,phases   ] GC(8) Concurrent Mark Free 0.000ms
[0.141s][info   ][gc,phases   ] GC(8) Concurrent Process Non-Strong References 0.211ms
[0.141s][info   ][gc,phases   ] GC(8) Concurrent Reset Relocation Set 0.000ms
[0.142s][info   ][gc          ] Allocation Stall (main) 2.726ms
[0.142s][info   ][gc,phases   ] GC(8) Concurrent Select Relocation Set 1.367ms
[0.142s][info   ][gc,phases   ] GC(8) Pause Relocate Start 0.003ms
[0.143s][info   ][gc,phases   ] GC(8) Concurrent Relocate 0.074ms
[0.143s][info   ][gc,load     ] GC(8) Load: 4.33/4.72/4.91
[0.143s][info   ][gc,mmu      ] GC(8) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.9%
[0.143s][info   ][gc,marking  ] GC(8) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.143s][info   ][gc,marking  ] GC(8) Mark Stack Usage: 32M
[0.143s][info   ][gc,nmethod  ] GC(8) NMethods: 117 registered, 0 unregistered
[0.143s][info   ][gc,metaspace] GC(8) Metaspace: 0M used, 0M committed, 1088M reserved
[0.143s][info   ][gc,ref      ] GC(8) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.143s][info   ][gc,ref      ] GC(8) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.143s][info   ][gc,ref      ] GC(8) Final: 0 encountered, 0 discovered, 0 enqueued
[0.143s][info   ][gc,ref      ] GC(8) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.143s][info   ][gc,reloc    ] GC(8) Small Pages: 17 / 34M, Empty: 0M, Relocated: 0M, In-Place: 1
[0.143s][info   ][gc,reloc    ] GC(8) Medium Pages: 6 / 24M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.143s][info   ][gc,reloc    ] GC(8) Large Pages: 34 / 68M, Empty: 2M, Relocated: 0M, In-Place: 0
[0.143s][info   ][gc,reloc    ] GC(8) Forwarding Usage: 0M
[0.143s][info   ][gc,heap     ] GC(8) Min Capacity: 128M(100%)
[0.143s][info   ][gc,heap     ] GC(8) Max Capacity: 128M(100%)
[0.143s][info   ][gc,heap     ] GC(8) Soft Max Capacity: 128M(100%)
[0.143s][info   ][gc,heap     ] GC(8)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.143s][info   ][gc,heap     ] GC(8)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.143s][info   ][gc,heap     ] GC(8)      Free:        2M (2%)            2M (2%)            0M (0%)            2M (2%)            4M (3%)            0M (0%)
[0.143s][info   ][gc,heap     ] GC(8)      Used:      126M (98%)         126M (98%)         128M (100%)        126M (98%)         128M (100%)        124M (97%)
[0.143s][info   ][gc,heap     ] GC(8)      Live:         -               112M (88%)         112M (88%)         112M (88%)            -                  -
[0.143s][info   ][gc,heap     ] GC(8) Allocated:         -                 0M (0%)            4M (3%)            4M (3%)             -                  -
[0.143s][info   ][gc,heap     ] GC(8)   Garbage:         -                13M (11%)          11M (9%)            9M (8%)             -                  -
[0.143s][info   ][gc,heap     ] GC(8) Reclaimed:         -                  -                 2M (2%)            4M (3%)             -                  -
[0.143s][info   ][gc          ] GC(8) Garbage Collection (Allocation Stall) 126M(98%)->126M(98%)
[0.143s][info   ][gc,start    ] GC(9) Garbage Collection (Allocation Stall)
[0.143s][info   ][gc,ref      ] GC(9) Clearing All SoftReferences
[0.143s][info   ][gc,task     ] GC(9) Using 1 workers
[0.143s][info   ][gc,phases   ] GC(9) Pause Mark Start 0.001ms
[0.144s][info   ][gc,phases   ] GC(9) Concurrent Mark 0.790ms
[0.144s][info   ][gc,phases   ] GC(9) Pause Mark End 0.003ms
[0.144s][info   ][gc,phases   ] GC(9) Concurrent Mark Free 0.000ms
[0.144s][info   ][gc,phases   ] GC(9) Concurrent Process Non-Strong References 0.246ms
[0.144s][info   ][gc,phases   ] GC(9) Concurrent Reset Relocation Set 0.000ms
[0.146s][info   ][gc,phases   ] GC(9) Concurrent Select Relocation Set 1.427ms
[0.146s][info   ][gc,phases   ] GC(9) Pause Relocate Start 0.006ms
[0.146s][info   ][gc,phases   ] GC(9) Concurrent Relocate 0.099ms
[0.146s][info   ][gc,load     ] GC(9) Load: 4.33/4.72/4.91
[0.146s][info   ][gc,mmu      ] GC(9) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.9%
[0.146s][info   ][gc,marking  ] GC(9) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.146s][info   ][gc,marking  ] GC(9) Mark Stack Usage: 32M
[0.146s][info   ][gc          ] Allocation Stall (main) 3.072ms
[0.146s][info   ][gc,nmethod  ] GC(9) NMethods: 117 registered, 0 unregistered
[0.146s][info   ][gc,metaspace] GC(9) Metaspace: 0M used, 0M committed, 1088M reserved
[0.146s][info   ][gc,ref      ] GC(9) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.146s][info   ][gc,ref      ] GC(9) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.146s][info   ][gc,ref      ] GC(9) Final: 0 encountered, 0 discovered, 0 enqueued
[0.146s][info   ][gc,ref      ] GC(9) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.146s][info   ][gc,reloc    ] GC(9) Small Pages: 17 / 34M, Empty: 0M, Relocated: 1M, In-Place: 1
[0.146s][info   ][gc,reloc    ] GC(9) Medium Pages: 7 / 28M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.146s][info   ][gc,reloc    ] GC(9) Large Pages: 33 / 66M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.146s][info   ][gc,reloc    ] GC(9) Forwarding Usage: 0M
[0.146s][info   ][gc,heap     ] GC(9) Min Capacity: 128M(100%)
[0.146s][info   ][gc,heap     ] GC(9) Max Capacity: 128M(100%)
[0.146s][info   ][gc,heap     ] GC(9) Soft Max Capacity: 128M(100%)
[0.146s][info   ][gc,heap     ] GC(9)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.146s][info   ][gc,heap     ] GC(9)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.146s][info   ][gc,heap     ] GC(9)      Free:        0M (0%)            0M (0%)            0M (0%)            0M (0%)            2M (2%)            0M (0%)
[0.146s][info   ][gc,heap     ] GC(9)      Used:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        126M (98%)
[0.146s][info   ][gc,heap     ] GC(9)      Live:         -               114M (89%)         114M (89%)         114M (89%)            -                  -
[0.146s][info   ][gc,heap     ] GC(9) Allocated:         -                 0M (0%)            0M (0%)            2M (2%)             -                  -
[0.146s][info   ][gc,heap     ] GC(9)   Garbage:         -                13M (11%)          13M (11%)          11M (9%)             -                  -
[0.146s][info   ][gc,heap     ] GC(9) Reclaimed:         -                  -                 0M (0%)            2M (2%)             -                  -
[0.146s][info   ][gc          ] GC(9) Garbage Collection (Allocation Stall) 128M(100%)->128M(100%)
[0.146s][info   ][gc,start    ] GC(10) Garbage Collection (Allocation Stall)
[0.146s][info   ][gc,ref      ] GC(10) Clearing All SoftReferences
[0.146s][info   ][gc,task     ] GC(10) Using 1 workers
[0.146s][info   ][gc,phases   ] GC(10) Pause Mark Start 0.002ms
[0.147s][info   ][gc,phases   ] GC(10) Concurrent Mark 0.773ms
[0.147s][info   ][gc,phases   ] GC(10) Pause Mark End 0.003ms
[0.147s][info   ][gc,phases   ] GC(10) Concurrent Mark Free 0.000ms
[0.147s][info   ][gc,phases   ] GC(10) Concurrent Process Non-Strong References 0.245ms
[0.147s][info   ][gc,phases   ] GC(10) Concurrent Reset Relocation Set 0.000ms
[0.149s][info   ][gc,phases   ] GC(10) Concurrent Select Relocation Set 1.374ms
[0.149s][info   ][gc,phases   ] GC(10) Pause Relocate Start 0.004ms
[0.149s][info   ][gc,phases   ] GC(10) Concurrent Relocate 0.005ms
[0.149s][info   ][gc,load     ] GC(10) Load: 4.33/4.72/4.91
[0.149s][info   ][gc,mmu      ] GC(10) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.9%
[0.149s][info   ][gc,marking  ] GC(10) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.149s][info   ][gc,marking  ] GC(10) Mark Stack Usage: 32M
[0.149s][info   ][gc,nmethod  ] GC(10) NMethods: 117 registered, 0 unregistered
[0.149s][info   ][gc,metaspace] GC(10) Metaspace: 0M used, 0M committed, 1088M reserved
[0.149s][info   ][gc,ref      ] GC(10) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.149s][info   ][gc,ref      ] GC(10) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.149s][info   ][gc,ref      ] GC(10) Final: 0 encountered, 0 discovered, 0 enqueued
[0.149s][info   ][gc,ref      ] GC(10) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.149s][info   ][gc,reloc    ] GC(10) Small Pages: 16 / 32M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.149s][info   ][gc,reloc    ] GC(10) Medium Pages: 7 / 28M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.149s][info   ][gc,reloc    ] GC(10) Large Pages: 34 / 68M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.149s][info   ][gc,reloc    ] GC(10) Forwarding Usage: 0M
[0.149s][info   ][gc,heap     ] GC(10) Min Capacity: 128M(100%)
[0.149s][info   ][gc,heap     ] GC(10) Max Capacity: 128M(100%)
[0.149s][info   ][gc,heap     ] GC(10) Soft Max Capacity: 128M(100%)
[0.149s][info   ][gc,heap     ] GC(10)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.149s][info   ][gc,heap     ] GC(10)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.149s][info   ][gc,heap     ] GC(10)      Free:        0M (0%)            0M (0%)            0M (0%)            0M (0%)            0M (0%)            0M (0%)
[0.149s][info   ][gc,heap     ] GC(10)      Used:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.149s][info   ][gc,heap     ] GC(10)      Live:         -               116M (91%)         116M (91%)         116M (91%)            -                  -
[0.149s][info   ][gc,heap     ] GC(10) Allocated:         -                 0M (0%)            0M (0%)            0M (0%)             -                  -
[0.149s][info   ][gc,heap     ] GC(10)   Garbage:         -                11M (9%)           11M (9%)           11M (9%)             -                  -
[0.149s][info   ][gc,heap     ] GC(10) Reclaimed:         -                  -                 0M (0%)            0M (0%)             -                  -
[0.149s][info   ][gc          ] GC(10) Garbage Collection (Allocation Stall) 128M(100%)->128M(100%)
[0.149s][info   ][gc          ] Allocation Stall (main) 3.065ms
[0.149s][info   ][gc,start    ] GC(11) Garbage Collection (Allocation Stall)
[0.149s][info   ][gc,ref      ] GC(11) Clearing All SoftReferences
[0.149s][info   ][gc,task     ] GC(11) Using 1 workers
[0.149s][info   ][gc,phases   ] GC(11) Pause Mark Start 0.001ms
[0.150s][info   ][gc,phases   ] GC(11) Concurrent Mark 0.766ms
[0.150s][info   ][gc,phases   ] GC(11) Pause Mark End 0.005ms
[0.150s][info   ][gc,phases   ] GC(11) Concurrent Mark Free 0.000ms
[0.150s][info   ][gc,phases   ] GC(11) Concurrent Process Non-Strong References 0.219ms
[0.150s][info   ][gc,phases   ] GC(11) Concurrent Reset Relocation Set 0.000ms
[0.152s][info   ][gc,phases   ] GC(11) Concurrent Select Relocation Set 1.443ms
[0.152s][info   ][gc,phases   ] GC(11) Pause Relocate Start 0.004ms
[0.152s][info   ][gc,phases   ] GC(11) Concurrent Relocate 0.021ms
[0.152s][info   ][gc,load     ] GC(11) Load: 4.33/4.72/4.91
[0.152s][info   ][gc,mmu      ] GC(11) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.7%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.152s][info   ][gc,marking  ] GC(11) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.152s][info   ][gc,marking  ] GC(11) Mark Stack Usage: 32M
[0.152s][info   ][gc,nmethod  ] GC(11) NMethods: 117 registered, 0 unregistered
[0.152s][info   ][gc,metaspace] GC(11) Metaspace: 0M used, 0M committed, 1088M reserved
[0.152s][info   ][gc,ref      ] GC(11) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.152s][info   ][gc,ref      ] GC(11) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.152s][info   ][gc,ref      ] GC(11) Final: 0 encountered, 0 discovered, 0 enqueued
[0.152s][info   ][gc,ref      ] GC(11) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.152s][info   ][gc,reloc    ] GC(11) Small Pages: 16 / 32M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.152s][info   ][gc,reloc    ] GC(11) Medium Pages: 7 / 28M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.152s][info   ][gc,reloc    ] GC(11) Large Pages: 34 / 68M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.152s][info   ][gc,reloc    ] GC(11) Forwarding Usage: 0M
[0.152s][info   ][gc,heap     ] GC(11) Min Capacity: 128M(100%)
[0.152s][info   ][gc,heap     ] GC(11) Max Capacity: 128M(100%)
[0.152s][info   ][gc,heap     ] GC(11) Soft Max Capacity: 128M(100%)
[0.152s][info   ][gc,heap     ] GC(11)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.152s][info   ][gc,heap     ] GC(11)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.152s][info   ][gc,heap     ] GC(11)      Free:        0M (0%)            0M (0%)            0M (0%)            0M (0%)            0M (0%)            0M (0%)
[0.152s][info   ][gc,heap     ] GC(11)      Used:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.152s][info   ][gc,heap     ] GC(11)      Live:         -               116M (91%)         116M (91%)         116M (91%)            -                  -
[0.152s][info   ][gc,heap     ] GC(11) Allocated:         -                 0M (0%)            0M (0%)            0M (0%)             -                  -
[0.152s][info   ][gc,heap     ] GC(11)   Garbage:         -                11M (9%)           11M (9%)           11M (9%)             -                  -
[0.152s][info   ][gc,heap     ] GC(11) Reclaimed:         -                  -                 0M (0%)            0M (0%)             -                  -
[0.152s][info   ][gc          ] GC(11) Garbage Collection (Allocation Stall) 128M(100%)->128M(100%)
[0.152s][info   ][gc          ] Allocation Stall (main) 3.102ms
[0.152s][info   ][gc          ] Out Of Memory (main)
[0.152s][info   ][gc,start    ] GC(12) Garbage Collection (Allocation Stall)
[0.152s][info   ][gc,ref      ] GC(12) Clearing All SoftReferences
[0.152s][info   ][gc,task     ] GC(12) Using 1 workers
[0.152s][info   ][gc,phases   ] GC(12) Pause Mark Start 0.003ms
[0.153s][info   ][gc,phases   ] GC(12) Concurrent Mark 0.594ms
[0.153s][info   ][gc,phases   ] GC(12) Pause Mark End 0.004ms
[0.153s][info   ][gc,phases   ] GC(12) Concurrent Mark Free 0.001ms
[0.153s][info   ][gc,phases   ] GC(12) Concurrent Process Non-Strong References 0.233ms
[0.153s][info   ][gc,phases   ] GC(12) Concurrent Reset Relocation Set 0.000ms
[0.155s][info   ][gc          ] Allocation Stall (main) 2.616ms
[0.155s][info   ][gc,phases   ] GC(12) Concurrent Select Relocation Set 1.484ms
[0.155s][info   ][gc,phases   ] GC(12) Pause Relocate Start 0.004ms
Exception in thread "main" [0.155s][info   ][gc,phases   ] GC(12) Concurrent Relocate 0.334ms
[0.155s][info   ][gc,load     ] GC(12) Load: 4.33/4.72/4.91
[0.155s][info   ][gc,mmu      ] GC(12) MMU: 2ms/99.3%, 5ms/99.5%, 10ms/99.6%, 20ms/99.7%, 50ms/99.7%, 100ms/99.8%
[0.155s][info   ][gc,marking  ] GC(12) Mark: 1 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.155s][info   ][gc,marking  ] GC(12) Mark Stack Usage: 32M
[0.156s][info   ][gc,nmethod  ] GC(12) NMethods: 117 registered, 0 unregistered
[0.156s][info   ][gc,metaspace] GC(12) Metaspace: 0M used, 0M committed, 1088M reserved
[0.156s][info   ][gc,ref      ] GC(12) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.156s][info   ][gc,ref      ] GC(12) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.156s][info   ][gc,ref      ] GC(12) Final: 0 encountered, 0 discovered, 0 enqueued
[0.156s][info   ][gc,ref      ] GC(12) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.156s][info   ][gc,reloc    ] GC(12) Small Pages: 16 / 32M, Empty: 16M, Relocated: 0M, In-Place: 0
[0.156s][info   ][gc,reloc    ] GC(12) Medium Pages: 7 / 28M, Empty: 28M, Relocated: 0M, In-Place: 0
[0.156s][info   ][gc,reloc    ] GC(12) Large Pages: 34 / 68M, Empty: 68M, Relocated: 0M, In-Place: 0
[0.156s][info   ][gc,reloc    ] GC(12) Forwarding Usage: 0M
[0.156s][info   ][gc,heap     ] GC(12) Min Capacity: 128M(100%)
[0.156s][info   ][gc,heap     ] GC(12) Max Capacity: 128M(100%)
[0.156s][info   ][gc,heap     ] GC(12) Soft Max Capacity: 128M(100%)
[0.156s][info   ][gc,heap     ] GC(12)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.156s][info   ][gc,heap     ] GC(12)  Capacity:      128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)        128M (100%)
[0.156s][info   ][gc,heap     ] GC(12)      Free:        0M (0%)            0M (0%)          110M (86%)         124M (97%)         124M (97%)           0M (0%)
[0.156s][info   ][gc,heap     ] GC(12)      Used:      128M (100%)        128M (100%)         18M (14%)           4M (3%)          128M (100%)          4M (3%)
[0.156s][info   ][gc,heap     ] GC(12)      Live:         -                 0M (0%)            0M (0%)            0M (0%)             -                  -
[0.156s][info   ][gc,heap     ] GC(12) Allocated:         -                 0M (0%)            2M (2%)            1M (2%)             -                  -
[0.156s][info   ][gc,heap     ] GC(12)   Garbage:         -               127M (100%)         15M (12%)           1M (1%)             -                  -
[0.156s][info   ][gc,heap     ] GC(12) Reclaimed:         -                  -               112M (88%)         125M (98%)            -                  -
[0.156s][info   ][gc          ] GC(12) Garbage Collection (Allocation Stall) 128M(100%)->4M(3%)
java.lang.OutOfMemoryError: Java heap space
	at GCLogAnalysis.generateGarbage(GCLogAnalysis.java:36)
	at GCLogAnalysis.main(GCLogAnalysis.java:20)
[0.157s][info   ][gc,heap,exit] Heap
[0.157s][info   ][gc,heap,exit]  ZHeap           used 4M, capacity 128M, max capacity 128M
[0.157s][info   ][gc,heap,exit]  Metaspace       used 215K, committed 448K, reserved 1114112K
[0.157s][info   ][gc,heap,exit]   class space    used 7K, committed 128K, reserved 1048576K
```

试试512m内存
```
java -XX:+UseZGC -Xms512m -Xmx512m -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

79次GC,生成了35000多个对象。可以看到效果比G1GC要好一些。
```
[0.757s][info   ][gc,ref      ] GC(53) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.757s][info   ][gc,reloc    ] GC(53) Small Pages: 83 / 166M, Empty: 0M, Relocated: 22M, In-Place: 0
[0.757s][info   ][gc,reloc    ] GC(53) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 40M, In-Place: 3
[0.757s][info   ][gc,reloc    ] GC(53) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.757s][info   ][gc,reloc    ] GC(53) Forwarding Usage: 0M
[0.757s][info   ][gc,heap     ] GC(53) Min Capacity: 512M(100%)
[0.757s][info   ][gc,heap     ] GC(53) Max Capacity: 512M(100%)
[0.757s][info   ][gc,heap     ] GC(53) Soft Max Capacity: 512M(100%)
[0.757s][info   ][gc,heap     ] GC(53)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.758s][info   ][gc,heap     ] GC(53)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.758s][info   ][gc,heap     ] GC(53)      Free:       10M (2%)           10M (2%)           10M (2%)           46M (9%)           46M (9%)            6M (1%)
[0.758s][info   ][gc,heap     ] GC(53)      Used:      502M (98%)         502M (98%)         502M (98%)         466M (91%)         506M (99%)         466M (91%)
[0.758s][info   ][gc,heap     ] GC(53)      Live:         -               348M (68%)         348M (68%)         348M (68%)            -                  -
[0.758s][info   ][gc,heap     ] GC(53) Allocated:         -                 0M (0%)            0M (0%)           25M (5%)             -                  -
[0.758s][info   ][gc,heap     ] GC(53)   Garbage:         -               153M (30%)         153M (30%)          91M (18%)            -                  -
[0.758s][info   ][gc,heap     ] GC(53) Reclaimed:         -                  -                 0M (0%)           61M (12%)            -                  -
[0.758s][info   ][gc          ] GC(53) Garbage Collection (Allocation Stall) 502M(98%)->466M(91%)
[0.760s][info   ][gc,start    ] GC(54) Garbage Collection (Allocation Stall)
[0.760s][info   ][gc,ref      ] GC(54) Clearing All SoftReferences
[0.760s][info   ][gc,task     ] GC(54) Using 2 workers
[0.760s][info   ][gc,phases   ] GC(54) Pause Mark Start 0.003ms
[0.762s][info   ][gc,phases   ] GC(54) Concurrent Mark 1.819ms
[0.762s][info   ][gc,phases   ] GC(54) Pause Mark End 0.007ms
[0.762s][info   ][gc,phases   ] GC(54) Concurrent Mark Free 0.000ms
[0.762s][info   ][gc,phases   ] GC(54) Concurrent Process Non-Strong References 0.287ms
[0.762s][info   ][gc,phases   ] GC(54) Concurrent Reset Relocation Set 0.001ms
[0.764s][info   ][gc,phases   ] GC(54) Concurrent Select Relocation Set 1.488ms
[0.764s][info   ][gc,phases   ] GC(54) Pause Relocate Start 0.006ms
[0.765s][info   ][gc          ] Allocation Stall (main) 4.888ms
[0.767s][info   ][gc,phases   ] GC(54) Concurrent Relocate 2.941ms
[0.767s][info   ][gc,load     ] GC(54) Load: 4.02/4.33/4.71
[0.767s][info   ][gc,mmu      ] GC(54) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.767s][info   ][gc,marking  ] GC(54) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.767s][info   ][gc,marking  ] GC(54) Mark Stack Usage: 32M
[0.767s][info   ][gc,nmethod  ] GC(54) NMethods: 121 registered, 0 unregistered
[0.767s][info   ][gc,metaspace] GC(54) Metaspace: 0M used, 0M committed, 1088M reserved
[0.767s][info   ][gc,ref      ] GC(54) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.767s][info   ][gc,ref      ] GC(54) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.767s][info   ][gc,ref      ] GC(54) Final: 0 encountered, 0 discovered, 0 enqueued
[0.767s][info   ][gc,ref      ] GC(54) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.767s][info   ][gc,reloc    ] GC(54) Small Pages: 86 / 172M, Empty: 0M, Relocated: 19M, In-Place: 0
[0.767s][info   ][gc,reloc    ] GC(54) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 69M, In-Place: 3
[0.767s][info   ][gc,reloc    ] GC(54) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.767s][info   ][gc,reloc    ] GC(54) Forwarding Usage: 0M
[0.767s][info   ][gc,heap     ] GC(54) Min Capacity: 512M(100%)
[0.767s][info   ][gc,heap     ] GC(54) Max Capacity: 512M(100%)
[0.767s][info   ][gc,heap     ] GC(54) Soft Max Capacity: 512M(100%)
[0.767s][info   ][gc,heap     ] GC(54)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.767s][info   ][gc,heap     ] GC(54)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.767s][info   ][gc,heap     ] GC(54)      Free:        4M (1%)            4M (1%)            4M (1%)           58M (11%)          58M (11%)           0M (0%)
[0.767s][info   ][gc,heap     ] GC(54)      Used:      508M (99%)         508M (99%)         508M (99%)         454M (89%)         512M (100%)        454M (89%)
[0.767s][info   ][gc,heap     ] GC(54)      Live:         -               346M (68%)         346M (68%)         346M (68%)            -                  -
[0.767s][info   ][gc,heap     ] GC(54) Allocated:         -                 0M (0%)            0M (0%)           27M (5%)             -                  -
[0.767s][info   ][gc,heap     ] GC(54)   Garbage:         -               161M (32%)         161M (32%)          79M (16%)            -                  -
[0.767s][info   ][gc,heap     ] GC(54) Reclaimed:         -                  -                 0M (0%)           81M (16%)            -                  -
[0.767s][info   ][gc          ] GC(54) Garbage Collection (Allocation Stall) 508M(99%)->454M(89%)
[0.771s][info   ][gc,start    ] GC(55) Garbage Collection (Allocation Stall)
[0.771s][info   ][gc,ref      ] GC(55) Clearing All SoftReferences
[0.771s][info   ][gc,task     ] GC(55) Using 2 workers
[0.771s][info   ][gc,phases   ] GC(55) Pause Mark Start 0.004ms
[0.774s][info   ][gc,phases   ] GC(55) Concurrent Mark 2.284ms
[0.774s][info   ][gc,phases   ] GC(55) Pause Mark End 0.005ms
[0.774s][info   ][gc,phases   ] GC(55) Concurrent Mark Free 0.000ms
[0.774s][info   ][gc,phases   ] GC(55) Concurrent Process Non-Strong References 0.211ms
[0.774s][info   ][gc,phases   ] GC(55) Concurrent Reset Relocation Set 0.006ms
[0.776s][info   ][gc,phases   ] GC(55) Concurrent Select Relocation Set 1.482ms
[0.776s][info   ][gc,phases   ] GC(55) Pause Relocate Start 0.006ms
[0.777s][info   ][gc          ] Allocation Stall (main) 5.454ms
[0.777s][info   ][gc          ] Allocation Stall (main) 0.170ms
[0.778s][info   ][gc,phases   ] GC(55) Concurrent Relocate 2.125ms
[0.778s][info   ][gc,load     ] GC(55) Load: 4.02/4.33/4.71
[0.778s][info   ][gc,mmu      ] GC(55) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.778s][info   ][gc,marking  ] GC(55) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.778s][info   ][gc,marking  ] GC(55) Mark Stack Usage: 32M
[0.778s][info   ][gc,nmethod  ] GC(55) NMethods: 121 registered, 0 unregistered
[0.778s][info   ][gc,metaspace] GC(55) Metaspace: 0M used, 0M committed, 1088M reserved
[0.778s][info   ][gc,ref      ] GC(55) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.778s][info   ][gc,ref      ] GC(55) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.778s][info   ][gc,ref      ] GC(55) Final: 0 encountered, 0 discovered, 0 enqueued
[0.778s][info   ][gc,ref      ] GC(55) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.778s][info   ][gc,reloc    ] GC(55) Small Pages: 88 / 176M, Empty: 0M, Relocated: 27M, In-Place: 0
[0.778s][info   ][gc,reloc    ] GC(55) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 38M, In-Place: 3
[0.778s][info   ][gc,reloc    ] GC(55) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.778s][info   ][gc,reloc    ] GC(55) Forwarding Usage: 0M
[0.778s][info   ][gc,heap     ] GC(55) Min Capacity: 512M(100%)
[0.778s][info   ][gc,heap     ] GC(55) Max Capacity: 512M(100%)
[0.778s][info   ][gc,heap     ] GC(55) Soft Max Capacity: 512M(100%)
[0.778s][info   ][gc,heap     ] GC(55)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.778s][info   ][gc,heap     ] GC(55)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.778s][info   ][gc,heap     ] GC(55)      Free:        0M (0%)            0M (0%)            0M (0%)           46M (9%)           46M (9%)            0M (0%)
[0.778s][info   ][gc,heap     ] GC(55)      Used:      512M (100%)        512M (100%)        512M (100%)        466M (91%)         512M (100%)        466M (91%)
[0.778s][info   ][gc,heap     ] GC(55)      Live:         -               348M (68%)         348M (68%)         348M (68%)            -                  -
[0.778s][info   ][gc,heap     ] GC(55) Allocated:         -                 0M (0%)            0M (0%)           21M (4%)             -                  -
[0.778s][info   ][gc,heap     ] GC(55)   Garbage:         -               163M (32%)         163M (32%)          95M (19%)            -                  -
[0.778s][info   ][gc,heap     ] GC(55) Reclaimed:         -                  -                 0M (0%)           67M (13%)            -                  -
[0.778s][info   ][gc          ] GC(55) Garbage Collection (Allocation Stall) 512M(100%)->466M(91%)
[0.782s][info   ][gc,start    ] GC(56) Garbage Collection (Allocation Stall)
[0.782s][info   ][gc,ref      ] GC(56) Clearing All SoftReferences
[0.782s][info   ][gc,task     ] GC(56) Using 2 workers
[0.782s][info   ][gc,phases   ] GC(56) Pause Mark Start 0.004ms
[0.784s][info   ][gc,phases   ] GC(56) Concurrent Mark 1.837ms
[0.784s][info   ][gc,phases   ] GC(56) Pause Mark End 0.005ms
[0.784s][info   ][gc,phases   ] GC(56) Concurrent Mark Free 0.001ms
[0.784s][info   ][gc,phases   ] GC(56) Concurrent Process Non-Strong References 0.185ms
[0.784s][info   ][gc,phases   ] GC(56) Concurrent Reset Relocation Set 0.002ms
[0.786s][info   ][gc,phases   ] GC(56) Concurrent Select Relocation Set 1.407ms
[0.786s][info   ][gc,phases   ] GC(56) Pause Relocate Start 0.006ms
[0.787s][info   ][gc          ] Allocation Stall (main) 4.687ms
[0.788s][info   ][gc,phases   ] GC(56) Concurrent Relocate 1.760ms
[0.788s][info   ][gc,load     ] GC(56) Load: 4.02/4.33/4.71
[0.788s][info   ][gc,mmu      ] GC(56) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.788s][info   ][gc,marking  ] GC(56) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.788s][info   ][gc,marking  ] GC(56) Mark Stack Usage: 32M
[0.788s][info   ][gc,nmethod  ] GC(56) NMethods: 121 registered, 0 unregistered
[0.788s][info   ][gc,metaspace] GC(56) Metaspace: 0M used, 0M committed, 1088M reserved
[0.788s][info   ][gc,ref      ] GC(56) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.788s][info   ][gc,ref      ] GC(56) Weak: 23 encountered, 2 discovered, 0 enqueued
[0.788s][info   ][gc,ref      ] GC(56) Final: 0 encountered, 0 discovered, 0 enqueued
[0.788s][info   ][gc,ref      ] GC(56) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.788s][info   ][gc,reloc    ] GC(56) Small Pages: 85 / 170M, Empty: 0M, Relocated: 26M, In-Place: 0
[0.788s][info   ][gc,reloc    ] GC(56) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 41M, In-Place: 3
[0.788s][info   ][gc,reloc    ] GC(56) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.788s][info   ][gc,reloc    ] GC(56) Forwarding Usage: 0M
[0.788s][info   ][gc,heap     ] GC(56) Min Capacity: 512M(100%)
[0.788s][info   ][gc,heap     ] GC(56) Max Capacity: 512M(100%)
[0.788s][info   ][gc,heap     ] GC(56) Soft Max Capacity: 512M(100%)
[0.788s][info   ][gc,heap     ] GC(56)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.788s][info   ][gc,heap     ] GC(56)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.788s][info   ][gc,heap     ] GC(56)      Free:        6M (1%)            6M (1%)            6M (1%)           48M (9%)           48M (9%)            2M (0%)
[0.788s][info   ][gc,heap     ] GC(56)      Used:      506M (99%)         506M (99%)         506M (99%)         464M (91%)         510M (100%)        464M (91%)
[0.788s][info   ][gc,heap     ] GC(56)      Live:         -               350M (68%)         350M (68%)         350M (68%)            -                  -
[0.788s][info   ][gc,heap     ] GC(56) Allocated:         -                 0M (0%)            0M (0%)           23M (5%)             -                  -
[0.788s][info   ][gc,heap     ] GC(56)   Garbage:         -               155M (30%)         155M (30%)          89M (18%)            -                  -
[0.788s][info   ][gc,heap     ] GC(56) Reclaimed:         -                  -                 0M (0%)           65M (13%)            -                  -
[0.788s][info   ][gc          ] GC(56) Garbage Collection (Allocation Stall) 506M(99%)->464M(91%)
[0.790s][info   ][gc,start    ] GC(57) Garbage Collection (Allocation Stall)
[0.790s][info   ][gc,ref      ] GC(57) Clearing All SoftReferences
[0.790s][info   ][gc,task     ] GC(57) Using 2 workers
[0.790s][info   ][gc,phases   ] GC(57) Pause Mark Start 0.003ms
[0.793s][info   ][gc,phases   ] GC(57) Concurrent Mark 2.162ms
[0.793s][info   ][gc,phases   ] GC(57) Pause Mark End 0.006ms
[0.793s][info   ][gc,phases   ] GC(57) Concurrent Mark Free 0.000ms
[0.793s][info   ][gc,phases   ] GC(57) Concurrent Process Non-Strong References 0.280ms
[0.793s][info   ][gc,phases   ] GC(57) Concurrent Reset Relocation Set 0.002ms
[0.794s][info   ][gc,phases   ] GC(57) Concurrent Select Relocation Set 1.411ms
[0.794s][info   ][gc,phases   ] GC(57) Pause Relocate Start 0.004ms
[0.796s][info   ][gc          ] Allocation Stall (main) 5.518ms
[0.796s][info   ][gc          ] Allocation Stall (main) 0.207ms
[0.798s][info   ][gc,phases   ] GC(57) Concurrent Relocate 3.403ms
[0.798s][info   ][gc,load     ] GC(57) Load: 4.02/4.33/4.71
[0.798s][info   ][gc,mmu      ] GC(57) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.798s][info   ][gc,marking  ] GC(57) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.798s][info   ][gc,marking  ] GC(57) Mark Stack Usage: 32M
[0.798s][info   ][gc,nmethod  ] GC(57) NMethods: 121 registered, 0 unregistered
[0.798s][info   ][gc,metaspace] GC(57) Metaspace: 0M used, 0M committed, 1088M reserved
[0.798s][info   ][gc,ref      ] GC(57) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.798s][info   ][gc,ref      ] GC(57) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.798s][info   ][gc,ref      ] GC(57) Final: 0 encountered, 0 discovered, 0 enqueued
[0.798s][info   ][gc,ref      ] GC(57) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.798s][info   ][gc,reloc    ] GC(57) Small Pages: 80 / 160M, Empty: 0M, Relocated: 22M, In-Place: 0
[0.798s][info   ][gc,reloc    ] GC(57) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 67M, In-Place: 5
[0.798s][info   ][gc,reloc    ] GC(57) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.798s][info   ][gc,reloc    ] GC(57) Forwarding Usage: 0M
[0.798s][info   ][gc,heap     ] GC(57) Min Capacity: 512M(100%)
[0.798s][info   ][gc,heap     ] GC(57) Max Capacity: 512M(100%)
[0.798s][info   ][gc,heap     ] GC(57) Soft Max Capacity: 512M(100%)
[0.798s][info   ][gc,heap     ] GC(57)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.798s][info   ][gc,heap     ] GC(57)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.798s][info   ][gc,heap     ] GC(57)      Free:        0M (0%)            0M (0%)            0M (0%)           30M (6%)           46M (9%)            0M (0%)
[0.798s][info   ][gc,heap     ] GC(57)      Used:      512M (100%)        512M (100%)        512M (100%)        482M (94%)         512M (100%)        466M (91%)
[0.798s][info   ][gc,heap     ] GC(57)      Live:         -               357M (70%)         357M (70%)         357M (70%)            -                  -
[0.798s][info   ][gc,heap     ] GC(57) Allocated:         -                 0M (0%)            0M (0%)           41M (8%)             -                  -
[0.798s][info   ][gc,heap     ] GC(57)   Garbage:         -               154M (30%)         154M (30%)          82M (16%)            -                  -
[0.798s][info   ][gc,heap     ] GC(57) Reclaimed:         -                  -                 0M (0%)           71M (14%)            -                  -
[0.798s][info   ][gc          ] GC(57) Garbage Collection (Allocation Stall) 512M(100%)->482M(94%)
[0.802s][info   ][gc,start    ] GC(58) Garbage Collection (Allocation Stall)
[0.802s][info   ][gc,ref      ] GC(58) Clearing All SoftReferences
[0.802s][info   ][gc,task     ] GC(58) Using 2 workers
[0.802s][info   ][gc,phases   ] GC(58) Pause Mark Start 0.006ms
[0.804s][info   ][gc,phases   ] GC(58) Concurrent Mark 1.870ms
[0.804s][info   ][gc,phases   ] GC(58) Pause Mark End 0.004ms
[0.804s][info   ][gc,phases   ] GC(58) Concurrent Mark Free 0.000ms
[0.804s][info   ][gc,phases   ] GC(58) Concurrent Process Non-Strong References 0.230ms
[0.804s][info   ][gc,phases   ] GC(58) Concurrent Reset Relocation Set 0.002ms
[0.806s][info   ][gc,phases   ] GC(58) Concurrent Select Relocation Set 1.448ms
[0.806s][info   ][gc,phases   ] GC(58) Pause Relocate Start 0.005ms
[0.807s][info   ][gc          ] Allocation Stall (main) 5.039ms
[0.808s][info   ][gc,phases   ] GC(58) Concurrent Relocate 2.126ms
[0.808s][info   ][gc,load     ] GC(58) Load: 4.02/4.33/4.71
[0.808s][info   ][gc,mmu      ] GC(58) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.808s][info   ][gc,marking  ] GC(58) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.808s][info   ][gc,marking  ] GC(58) Mark Stack Usage: 32M
[0.808s][info   ][gc,nmethod  ] GC(58) NMethods: 121 registered, 0 unregistered
[0.808s][info   ][gc,metaspace] GC(58) Metaspace: 0M used, 0M committed, 1088M reserved
[0.808s][info   ][gc,ref      ] GC(58) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.808s][info   ][gc,ref      ] GC(58) Weak: 23 encountered, 2 discovered, 0 enqueued
[0.808s][info   ][gc,ref      ] GC(58) Final: 0 encountered, 0 discovered, 0 enqueued
[0.808s][info   ][gc,ref      ] GC(58) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.808s][info   ][gc,reloc    ] GC(58) Small Pages: 80 / 160M, Empty: 0M, Relocated: 22M, In-Place: 0
[0.808s][info   ][gc,reloc    ] GC(58) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 13M, In-Place: 1
[0.808s][info   ][gc,reloc    ] GC(58) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.808s][info   ][gc,reloc    ] GC(58) Forwarding Usage: 0M
[0.808s][info   ][gc,heap     ] GC(58) Min Capacity: 512M(100%)
[0.808s][info   ][gc,heap     ] GC(58) Max Capacity: 512M(100%)
[0.808s][info   ][gc,heap     ] GC(58) Soft Max Capacity: 512M(100%)
[0.808s][info   ][gc,heap     ] GC(58)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.808s][info   ][gc,heap     ] GC(58)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.808s][info   ][gc,heap     ] GC(58)      Free:        0M (0%)            0M (0%)            0M (0%)           40M (8%)           40M (8%)            0M (0%)
[0.808s][info   ][gc,heap     ] GC(58)      Used:      512M (100%)        512M (100%)        512M (100%)        472M (92%)         512M (100%)        472M (92%)
[0.808s][info   ][gc,heap     ] GC(58)      Live:         -               360M (70%)         360M (70%)         360M (70%)            -                  -
[0.808s][info   ][gc,heap     ] GC(58) Allocated:         -                 0M (0%)            0M (0%)           19M (4%)             -                  -
[0.808s][info   ][gc,heap     ] GC(58)   Garbage:         -               151M (30%)         151M (30%)          91M (18%)            -                  -
[0.808s][info   ][gc,heap     ] GC(58) Reclaimed:         -                  -                 0M (0%)           59M (12%)            -                  -
[0.808s][info   ][gc          ] GC(58) Garbage Collection (Allocation Stall) 512M(100%)->472M(92%)
[0.812s][info   ][gc,start    ] GC(59) Garbage Collection (Allocation Stall)
[0.812s][info   ][gc,ref      ] GC(59) Clearing All SoftReferences
[0.812s][info   ][gc,task     ] GC(59) Using 2 workers
[0.812s][info   ][gc,phases   ] GC(59) Pause Mark Start 0.004ms
[0.814s][info   ][gc,phases   ] GC(59) Concurrent Mark 1.838ms
[0.814s][info   ][gc,phases   ] GC(59) Pause Mark End 0.005ms
[0.814s][info   ][gc,phases   ] GC(59) Concurrent Mark Free 0.000ms
[0.814s][info   ][gc,phases   ] GC(59) Concurrent Process Non-Strong References 0.191ms
[0.814s][info   ][gc,phases   ] GC(59) Concurrent Reset Relocation Set 0.002ms
[0.816s][info   ][gc,phases   ] GC(59) Concurrent Select Relocation Set 1.544ms
[0.816s][info   ][gc,phases   ] GC(59) Pause Relocate Start 0.005ms
[0.817s][info   ][gc          ] Allocation Stall (main) 5.187ms
[0.818s][info   ][gc,phases   ] GC(59) Concurrent Relocate 2.015ms
[0.818s][info   ][gc,load     ] GC(59) Load: 4.02/4.33/4.71
[0.818s][info   ][gc,mmu      ] GC(59) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.818s][info   ][gc,marking  ] GC(59) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.818s][info   ][gc,marking  ] GC(59) Mark Stack Usage: 32M
[0.818s][info   ][gc,nmethod  ] GC(59) NMethods: 121 registered, 0 unregistered
[0.818s][info   ][gc,metaspace] GC(59) Metaspace: 0M used, 0M committed, 1088M reserved
[0.818s][info   ][gc,ref      ] GC(59) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.818s][info   ][gc,ref      ] GC(59) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.818s][info   ][gc,ref      ] GC(59) Final: 0 encountered, 0 discovered, 0 enqueued
[0.818s][info   ][gc,ref      ] GC(59) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.818s][info   ][gc,reloc    ] GC(59) Small Pages: 80 / 160M, Empty: 0M, Relocated: 17M, In-Place: 0
[0.818s][info   ][gc,reloc    ] GC(59) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 41M, In-Place: 3
[0.818s][info   ][gc,reloc    ] GC(59) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.818s][info   ][gc,reloc    ] GC(59) Forwarding Usage: 0M
[0.818s][info   ][gc,heap     ] GC(59) Min Capacity: 512M(100%)
[0.818s][info   ][gc,heap     ] GC(59) Max Capacity: 512M(100%)
[0.818s][info   ][gc,heap     ] GC(59) Soft Max Capacity: 512M(100%)
[0.818s][info   ][gc,heap     ] GC(59)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.818s][info   ][gc,heap     ] GC(59)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.818s][info   ][gc,heap     ] GC(59)      Free:        0M (0%)            0M (0%)            0M (0%)           34M (7%)           36M (7%)            0M (0%)
[0.818s][info   ][gc,heap     ] GC(59)      Used:      512M (100%)        512M (100%)        512M (100%)        478M (93%)         512M (100%)        476M (93%)
[0.818s][info   ][gc,heap     ] GC(59)      Live:         -               354M (69%)         354M (69%)         354M (69%)            -                  -
[0.818s][info   ][gc,heap     ] GC(59) Allocated:         -                 0M (0%)            0M (0%)           21M (4%)             -                  -
[0.818s][info   ][gc,heap     ] GC(59)   Garbage:         -               157M (31%)         157M (31%)         101M (20%)            -                  -
[0.818s][info   ][gc,heap     ] GC(59) Reclaimed:         -                  -                 0M (0%)           55M (11%)            -                  -
[0.818s][info   ][gc          ] GC(59) Garbage Collection (Allocation Stall) 512M(100%)->478M(93%)
[0.820s][info   ][gc,start    ] GC(60) Garbage Collection (Warmup)
[0.820s][info   ][gc,task     ] GC(60) Using 2 workers
[0.820s][info   ][gc,phases   ] GC(60) Pause Mark Start 0.003ms
[0.822s][info   ][gc,phases   ] GC(60) Concurrent Mark 1.774ms
[0.822s][info   ][gc,phases   ] GC(60) Pause Mark End 0.005ms
[0.822s][info   ][gc,phases   ] GC(60) Concurrent Mark Free 0.000ms
[0.822s][info   ][gc,phases   ] GC(60) Concurrent Process Non-Strong References 0.228ms
[0.822s][info   ][gc,phases   ] GC(60) Concurrent Reset Relocation Set 0.002ms
[0.824s][info   ][gc,phases   ] GC(60) Concurrent Select Relocation Set 1.419ms
[0.824s][info   ][gc,phases   ] GC(60) Pause Relocate Start 0.002ms
[0.825s][info   ][gc          ] Allocation Stall (main) 4.701ms
[0.826s][info   ][gc,phases   ] GC(60) Concurrent Relocate 2.090ms
[0.826s][info   ][gc,load     ] GC(60) Load: 4.02/4.33/4.71
[0.826s][info   ][gc,mmu      ] GC(60) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.826s][info   ][gc,marking  ] GC(60) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.826s][info   ][gc,marking  ] GC(60) Mark Stack Usage: 32M
[0.826s][info   ][gc,nmethod  ] GC(60) NMethods: 121 registered, 0 unregistered
[0.826s][info   ][gc,metaspace] GC(60) Metaspace: 0M used, 0M committed, 1088M reserved
[0.826s][info   ][gc,ref      ] GC(60) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.826s][info   ][gc,ref      ] GC(60) Weak: 23 encountered, 2 discovered, 0 enqueued
[0.826s][info   ][gc,ref      ] GC(60) Final: 0 encountered, 0 discovered, 0 enqueued
[0.826s][info   ][gc,ref      ] GC(60) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.826s][info   ][gc,reloc    ] GC(60) Small Pages: 78 / 156M, Empty: 0M, Relocated: 12M, In-Place: 1
[0.826s][info   ][gc,reloc    ] GC(60) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 39M, In-Place: 3
[0.826s][info   ][gc,reloc    ] GC(60) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.826s][info   ][gc,reloc    ] GC(60) Forwarding Usage: 0M
[0.826s][info   ][gc,heap     ] GC(60) Min Capacity: 512M(100%)
[0.826s][info   ][gc,heap     ] GC(60) Max Capacity: 512M(100%)
[0.826s][info   ][gc,heap     ] GC(60) Soft Max Capacity: 512M(100%)
[0.826s][info   ][gc,heap     ] GC(60)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.826s][info   ][gc,heap     ] GC(60)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.826s][info   ][gc,heap     ] GC(60)      Free:        4M (1%)            2M (0%)            2M (0%)           32M (6%)           32M (6%)            0M (0%)
[0.826s][info   ][gc,heap     ] GC(60)      Used:      508M (99%)         510M (100%)        510M (100%)        480M (94%)         512M (100%)        480M (94%)
[0.826s][info   ][gc,heap     ] GC(60)      Live:         -               355M (69%)         355M (69%)         355M (69%)            -                  -
[0.826s][info   ][gc,heap     ] GC(60) Allocated:         -                 2M (0%)            2M (0%)           23M (5%)             -                  -
[0.826s][info   ][gc,heap     ] GC(60)   Garbage:         -               152M (30%)         152M (30%)         100M (20%)            -                  -
[0.826s][info   ][gc,heap     ] GC(60) Reclaimed:         -                  -                 0M (0%)           51M (10%)            -                  -
[0.826s][info   ][gc          ] GC(60) Garbage Collection (Warmup) 508M(99%)->480M(94%)
[0.827s][info   ][gc,start    ] GC(61) Garbage Collection (Allocation Stall)
[0.827s][info   ][gc,ref      ] GC(61) Clearing All SoftReferences
[0.827s][info   ][gc,task     ] GC(61) Using 2 workers
[0.827s][info   ][gc,phases   ] GC(61) Pause Mark Start 0.004ms
[0.829s][info   ][gc,phases   ] GC(61) Concurrent Mark 1.789ms
[0.829s][info   ][gc,phases   ] GC(61) Pause Mark End 0.003ms
[0.829s][info   ][gc,phases   ] GC(61) Concurrent Mark Free 0.000ms
[0.829s][info   ][gc,phases   ] GC(61) Concurrent Process Non-Strong References 0.189ms
[0.829s][info   ][gc,phases   ] GC(61) Concurrent Reset Relocation Set 0.004ms
[0.831s][info   ][gc,phases   ] GC(61) Concurrent Select Relocation Set 1.448ms
[0.831s][info   ][gc,phases   ] GC(61) Pause Relocate Start 0.006ms
[0.832s][info   ][gc          ] Allocation Stall (main) 4.986ms
[0.833s][info   ][gc,phases   ] GC(61) Concurrent Relocate 2.043ms
[0.833s][info   ][gc,load     ] GC(61) Load: 4.02/4.33/4.71
[0.833s][info   ][gc,mmu      ] GC(61) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.833s][info   ][gc,marking  ] GC(61) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.833s][info   ][gc,marking  ] GC(61) Mark Stack Usage: 32M
[0.833s][info   ][gc,nmethod  ] GC(61) NMethods: 121 registered, 0 unregistered
[0.833s][info   ][gc,metaspace] GC(61) Metaspace: 0M used, 0M committed, 1088M reserved
[0.833s][info   ][gc,ref      ] GC(61) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.833s][info   ][gc,ref      ] GC(61) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.833s][info   ][gc,ref      ] GC(61) Final: 0 encountered, 0 discovered, 0 enqueued
[0.833s][info   ][gc,ref      ] GC(61) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.833s][info   ][gc,reloc    ] GC(61) Small Pages: 77 / 154M, Empty: 0M, Relocated: 12M, In-Place: 0
[0.833s][info   ][gc,reloc    ] GC(61) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 41M, In-Place: 3
[0.833s][info   ][gc,reloc    ] GC(61) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.833s][info   ][gc,reloc    ] GC(61) Forwarding Usage: 0M
[0.833s][info   ][gc,heap     ] GC(61) Min Capacity: 512M(100%)
[0.834s][info   ][gc,heap     ] GC(61) Max Capacity: 512M(100%)
[0.834s][info   ][gc,heap     ] GC(61) Soft Max Capacity: 512M(100%)
[0.834s][info   ][gc,heap     ] GC(61)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.834s][info   ][gc,heap     ] GC(61)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.834s][info   ][gc,heap     ] GC(61)      Free:        6M (1%)            6M (1%)            6M (1%)           38M (7%)           38M (7%)            2M (0%)
[0.834s][info   ][gc,heap     ] GC(61)      Used:      506M (99%)         506M (99%)         506M (99%)         474M (93%)         510M (100%)        474M (93%)
[0.834s][info   ][gc,heap     ] GC(61)      Live:         -               358M (70%)         358M (70%)         358M (70%)            -                  -
[0.834s][info   ][gc,heap     ] GC(61) Allocated:         -                 0M (0%)            0M (0%)           19M (4%)             -                  -
[0.834s][info   ][gc,heap     ] GC(61)   Garbage:         -               147M (29%)         147M (29%)          95M (19%)            -                  -
[0.834s][info   ][gc,heap     ] GC(61) Reclaimed:         -                  -                 0M (0%)           51M (10%)            -                  -
[0.834s][info   ][gc          ] GC(61) Garbage Collection (Allocation Stall) 506M(99%)->474M(93%)
[0.835s][info   ][gc,start    ] GC(62) Garbage Collection (Allocation Stall)
[0.835s][info   ][gc,ref      ] GC(62) Clearing All SoftReferences
[0.835s][info   ][gc,task     ] GC(62) Using 2 workers
[0.835s][info   ][gc,phases   ] GC(62) Pause Mark Start 0.004ms
[0.837s][info   ][gc,phases   ] GC(62) Concurrent Mark 1.927ms
[0.837s][info   ][gc,phases   ] GC(62) Pause Mark End 0.008ms
[0.837s][info   ][gc,phases   ] GC(62) Concurrent Mark Free 0.000ms
[0.838s][info   ][gc,phases   ] GC(62) Concurrent Process Non-Strong References 0.228ms
[0.838s][info   ][gc,phases   ] GC(62) Concurrent Reset Relocation Set 0.001ms
[0.839s][info   ][gc,phases   ] GC(62) Concurrent Select Relocation Set 1.485ms
[0.839s][info   ][gc,phases   ] GC(62) Pause Relocate Start 0.002ms
[0.840s][info   ][gc          ] Allocation Stall (main) 4.752ms
[0.841s][info   ][gc,phases   ] GC(62) Concurrent Relocate 1.532ms
[0.841s][info   ][gc,load     ] GC(62) Load: 4.02/4.33/4.71
[0.841s][info   ][gc,mmu      ] GC(62) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.841s][info   ][gc,marking  ] GC(62) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.841s][info   ][gc,marking  ] GC(62) Mark Stack Usage: 32M
[0.841s][info   ][gc,nmethod  ] GC(62) NMethods: 121 registered, 0 unregistered
[0.841s][info   ][gc,metaspace] GC(62) Metaspace: 0M used, 0M committed, 1088M reserved
[0.841s][info   ][gc,ref      ] GC(62) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.841s][info   ][gc,ref      ] GC(62) Weak: 23 encountered, 1 discovered, 0 enqueued
[0.841s][info   ][gc,ref      ] GC(62) Final: 0 encountered, 0 discovered, 0 enqueued
[0.841s][info   ][gc,ref      ] GC(62) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.841s][info   ][gc,reloc    ] GC(62) Small Pages: 78 / 156M, Empty: 0M, Relocated: 20M, In-Place: 0
[0.841s][info   ][gc,reloc    ] GC(62) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 41M, In-Place: 3
[0.841s][info   ][gc,reloc    ] GC(62) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.841s][info   ][gc,reloc    ] GC(62) Forwarding Usage: 0M
[0.841s][info   ][gc,heap     ] GC(62) Min Capacity: 512M(100%)
[0.841s][info   ][gc,heap     ] GC(62) Max Capacity: 512M(100%)
[0.841s][info   ][gc,heap     ] GC(62) Soft Max Capacity: 512M(100%)
[0.841s][info   ][gc,heap     ] GC(62)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.841s][info   ][gc,heap     ] GC(62)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.841s][info   ][gc,heap     ] GC(62)      Free:        4M (1%)            4M (1%)            4M (1%)           40M (8%)           40M (8%)            0M (0%)
[0.841s][info   ][gc,heap     ] GC(62)      Used:      508M (99%)         508M (99%)         508M (99%)         472M (92%)         512M (100%)        472M (92%)
[0.841s][info   ][gc,heap     ] GC(62)      Live:         -               360M (70%)         360M (70%)         360M (70%)            -                  -
[0.841s][info   ][gc,heap     ] GC(62) Allocated:         -                 0M (0%)            0M (0%)           21M (4%)             -                  -
[0.841s][info   ][gc,heap     ] GC(62)   Garbage:         -               147M (29%)         147M (29%)          89M (17%)            -                  -
[0.841s][info   ][gc,heap     ] GC(62) Reclaimed:         -                  -                 0M (0%)           57M (11%)            -                  -
[0.841s][info   ][gc          ] GC(62) Garbage Collection (Allocation Stall) 508M(99%)->472M(92%)
[0.843s][info   ][gc,start    ] GC(63) Garbage Collection (Allocation Stall)
[0.843s][info   ][gc,ref      ] GC(63) Clearing All SoftReferences
[0.843s][info   ][gc,task     ] GC(63) Using 2 workers
[0.843s][info   ][gc,phases   ] GC(63) Pause Mark Start 0.004ms
[0.845s][info   ][gc,phases   ] GC(63) Concurrent Mark 1.763ms
[0.845s][info   ][gc,phases   ] GC(63) Pause Mark End 0.004ms
[0.845s][info   ][gc,phases   ] GC(63) Concurrent Mark Free 0.000ms
[0.845s][info   ][gc,phases   ] GC(63) Concurrent Process Non-Strong References 0.198ms
[0.845s][info   ][gc,phases   ] GC(63) Concurrent Reset Relocation Set 0.002ms
[0.847s][info   ][gc,phases   ] GC(63) Concurrent Select Relocation Set 1.399ms
[0.847s][info   ][gc,phases   ] GC(63) Pause Relocate Start 0.002ms
[0.848s][info   ][gc          ] Allocation Stall (main) 4.651ms
[0.849s][info   ][gc          ] Allocation Stall (main) 0.808ms
[0.849s][info   ][gc,phases   ] GC(63) Concurrent Relocate 2.375ms
[0.849s][info   ][gc,load     ] GC(63) Load: 4.02/4.33/4.71
[0.849s][info   ][gc,mmu      ] GC(63) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.849s][info   ][gc,marking  ] GC(63) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.849s][info   ][gc,marking  ] GC(63) Mark Stack Usage: 32M
[0.849s][info   ][gc,nmethod  ] GC(63) NMethods: 121 registered, 0 unregistered
[0.849s][info   ][gc,metaspace] GC(63) Metaspace: 0M used, 0M committed, 1088M reserved
[0.849s][info   ][gc,ref      ] GC(63) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.849s][info   ][gc,ref      ] GC(63) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.849s][info   ][gc,ref      ] GC(63) Final: 0 encountered, 0 discovered, 0 enqueued
[0.849s][info   ][gc,ref      ] GC(63) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.849s][info   ][gc,reloc    ] GC(63) Small Pages: 78 / 156M, Empty: 0M, Relocated: 19M, In-Place: 1
[0.850s][info   ][gc,reloc    ] GC(63) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 53M, In-Place: 4
[0.850s][info   ][gc,reloc    ] GC(63) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.850s][info   ][gc,reloc    ] GC(63) Forwarding Usage: 0M
[0.850s][info   ][gc,heap     ] GC(63) Min Capacity: 512M(100%)
[0.850s][info   ][gc,heap     ] GC(63) Max Capacity: 512M(100%)
[0.850s][info   ][gc,heap     ] GC(63) Soft Max Capacity: 512M(100%)
[0.850s][info   ][gc,heap     ] GC(63)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.850s][info   ][gc,heap     ] GC(63)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.850s][info   ][gc,heap     ] GC(63)      Free:        4M (1%)            4M (1%)            4M (1%)           38M (7%)           38M (7%)            0M (0%)
[0.850s][info   ][gc,heap     ] GC(63)      Used:      508M (99%)         508M (99%)         508M (99%)         474M (93%)         512M (100%)        474M (93%)
[0.850s][info   ][gc,heap     ] GC(63)      Live:         -               361M (71%)         361M (71%)         361M (71%)            -                  -
[0.850s][info   ][gc,heap     ] GC(63) Allocated:         -                 0M (0%)            0M (0%)           21M (4%)             -                  -
[0.850s][info   ][gc,heap     ] GC(63)   Garbage:         -               146M (29%)         146M (29%)          90M (18%)            -                  -
[0.850s][info   ][gc,heap     ] GC(63) Reclaimed:         -                  -                 0M (0%)           55M (11%)            -                  -
[0.850s][info   ][gc          ] GC(63) Garbage Collection (Allocation Stall) 508M(99%)->474M(93%)
[0.852s][info   ][gc,start    ] GC(64) Garbage Collection (Allocation Stall)
[0.852s][info   ][gc,ref      ] GC(64) Clearing All SoftReferences
[0.852s][info   ][gc,task     ] GC(64) Using 2 workers
[0.852s][info   ][gc,phases   ] GC(64) Pause Mark Start 0.004ms
[0.854s][info   ][gc,phases   ] GC(64) Concurrent Mark 2.055ms
[0.854s][info   ][gc,phases   ] GC(64) Pause Mark End 0.003ms
[0.854s][info   ][gc,phases   ] GC(64) Concurrent Mark Free 0.000ms
[0.855s][info   ][gc,phases   ] GC(64) Concurrent Process Non-Strong References 0.175ms
[0.855s][info   ][gc,phases   ] GC(64) Concurrent Reset Relocation Set 0.002ms
[0.856s][info   ][gc,phases   ] GC(64) Concurrent Select Relocation Set 1.444ms
[0.856s][info   ][gc,phases   ] GC(64) Pause Relocate Start 0.006ms
[0.859s][info   ][gc          ] Relocation Stall (main) 1.560ms
[0.859s][info   ][gc          ] Allocation Stall (main) 6.522ms
[0.859s][info   ][gc,phases   ] GC(64) Concurrent Relocate 3.196ms
[0.859s][info   ][gc,load     ] GC(64) Load: 4.02/4.33/4.71
[0.859s][info   ][gc,mmu      ] GC(64) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.859s][info   ][gc,marking  ] GC(64) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.859s][info   ][gc,marking  ] GC(64) Mark Stack Usage: 32M
[0.859s][info   ][gc,nmethod  ] GC(64) NMethods: 121 registered, 0 unregistered
[0.859s][info   ][gc,metaspace] GC(64) Metaspace: 0M used, 0M committed, 1088M reserved
[0.859s][info   ][gc,ref      ] GC(64) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.859s][info   ][gc,ref      ] GC(64) Weak: 23 encountered, 12 discovered, 0 enqueued
[0.859s][info   ][gc,ref      ] GC(64) Final: 0 encountered, 0 discovered, 0 enqueued
[0.859s][info   ][gc,ref      ] GC(64) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.859s][info   ][gc,reloc    ] GC(64) Small Pages: 80 / 160M, Empty: 0M, Relocated: 15M, In-Place: 0
[0.860s][info   ][gc,reloc    ] GC(64) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 69M, In-Place: 3
[0.860s][info   ][gc,reloc    ] GC(64) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.860s][info   ][gc,reloc    ] GC(64) Forwarding Usage: 0M
[0.860s][info   ][gc,heap     ] GC(64) Min Capacity: 512M(100%)
[0.860s][info   ][gc,heap     ] GC(64) Max Capacity: 512M(100%)
[0.860s][info   ][gc,heap     ] GC(64) Soft Max Capacity: 512M(100%)
[0.860s][info   ][gc,heap     ] GC(64)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.860s][info   ][gc,heap     ] GC(64)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.860s][info   ][gc,heap     ] GC(64)      Free:        0M (0%)            0M (0%)            0M (0%)           54M (11%)          54M (11%)           0M (0%)
[0.860s][info   ][gc,heap     ] GC(64)      Used:      512M (100%)        512M (100%)        512M (100%)        458M (89%)         512M (100%)        458M (89%)
[0.860s][info   ][gc,heap     ] GC(64)      Live:         -               359M (70%)         359M (70%)         359M (70%)            -                  -
[0.860s][info   ][gc,heap     ] GC(64) Allocated:         -                 0M (0%)            0M (0%)           20M (4%)             -                  -
[0.860s][info   ][gc,heap     ] GC(64)   Garbage:         -               152M (30%)         152M (30%)          78M (15%)            -                  -
[0.860s][info   ][gc,heap     ] GC(64) Reclaimed:         -                  -                 0M (0%)           74M (14%)            -                  -
[0.860s][info   ][gc          ] GC(64) Garbage Collection (Allocation Stall) 512M(100%)->458M(89%)
[0.863s][info   ][gc,start    ] GC(65) Garbage Collection (Allocation Stall)
[0.863s][info   ][gc,ref      ] GC(65) Clearing All SoftReferences
[0.863s][info   ][gc,task     ] GC(65) Using 2 workers
[0.863s][info   ][gc,phases   ] GC(65) Pause Mark Start 0.006ms
[0.866s][info   ][gc,phases   ] GC(65) Concurrent Mark 2.956ms
[0.866s][info   ][gc,phases   ] GC(65) Pause Mark End 0.004ms
[0.866s][info   ][gc,phases   ] GC(65) Concurrent Mark Free 0.000ms
[0.866s][info   ][gc,phases   ] GC(65) Concurrent Process Non-Strong References 0.228ms
[0.866s][info   ][gc,phases   ] GC(65) Concurrent Reset Relocation Set 0.002ms
[0.868s][info   ][gc,phases   ] GC(65) Concurrent Select Relocation Set 1.485ms
[0.868s][info   ][gc,phases   ] GC(65) Pause Relocate Start 0.005ms
[0.869s][info   ][gc          ] Allocation Stall (main) 5.988ms
[0.869s][info   ][gc          ] Allocation Stall (main) 0.179ms
[0.870s][info   ][gc,phases   ] GC(65) Concurrent Relocate 2.007ms
[0.870s][info   ][gc,load     ] GC(65) Load: 4.02/4.33/4.71
[0.870s][info   ][gc,mmu      ] GC(65) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.870s][info   ][gc,marking  ] GC(65) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.870s][info   ][gc,marking  ] GC(65) Mark Stack Usage: 32M
[0.870s][info   ][gc,nmethod  ] GC(65) NMethods: 121 registered, 0 unregistered
[0.870s][info   ][gc,metaspace] GC(65) Metaspace: 0M used, 0M committed, 1088M reserved
[0.870s][info   ][gc,ref      ] GC(65) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.870s][info   ][gc,ref      ] GC(65) Weak: 23 encountered, 7 discovered, 0 enqueued
[0.870s][info   ][gc,ref      ] GC(65) Final: 0 encountered, 0 discovered, 0 enqueued
[0.870s][info   ][gc,ref      ] GC(65) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.870s][info   ][gc,reloc    ] GC(65) Small Pages: 80 / 160M, Empty: 0M, Relocated: 12M, In-Place: 0
[0.870s][info   ][gc,reloc    ] GC(65) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 27M, In-Place: 2
[0.870s][info   ][gc,reloc    ] GC(65) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.870s][info   ][gc,reloc    ] GC(65) Forwarding Usage: 0M
[0.870s][info   ][gc,heap     ] GC(65) Min Capacity: 512M(100%)
[0.870s][info   ][gc,heap     ] GC(65) Max Capacity: 512M(100%)
[0.870s][info   ][gc,heap     ] GC(65) Soft Max Capacity: 512M(100%)
[0.870s][info   ][gc,heap     ] GC(65)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.870s][info   ][gc,heap     ] GC(65)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.870s][info   ][gc,heap     ] GC(65)      Free:        0M (0%)            0M (0%)            0M (0%)           50M (10%)          50M (10%)           0M (0%)
[0.870s][info   ][gc,heap     ] GC(65)      Used:      512M (100%)        512M (100%)        512M (100%)        462M (90%)         512M (100%)        462M (90%)
[0.870s][info   ][gc,heap     ] GC(65)      Live:         -               355M (69%)         355M (69%)         355M (69%)            -                  -
[0.870s][info   ][gc,heap     ] GC(65) Allocated:         -                 0M (0%)            0M (0%)           22M (4%)             -                  -
[0.870s][info   ][gc,heap     ] GC(65)   Garbage:         -               156M (31%)         156M (31%)          84M (17%)            -                  -
[0.870s][info   ][gc,heap     ] GC(65) Reclaimed:         -                  -                 0M (0%)           72M (14%)            -                  -
[0.870s][info   ][gc          ] GC(65) Garbage Collection (Allocation Stall) 512M(100%)->462M(90%)
[0.873s][info   ][gc,start    ] GC(66) Garbage Collection (Allocation Stall)
[0.873s][info   ][gc,ref      ] GC(66) Clearing All SoftReferences
[0.873s][info   ][gc,task     ] GC(66) Using 2 workers
[0.873s][info   ][gc,phases   ] GC(66) Pause Mark Start 0.003ms
[0.875s][info   ][gc,phases   ] GC(66) Concurrent Mark 2.275ms
[0.875s][info   ][gc,phases   ] GC(66) Pause Mark End 0.004ms
[0.875s][info   ][gc,phases   ] GC(66) Concurrent Mark Free 0.000ms
[0.876s][info   ][gc,phases   ] GC(66) Concurrent Process Non-Strong References 0.204ms
[0.876s][info   ][gc,phases   ] GC(66) Concurrent Reset Relocation Set 0.002ms
[0.877s][info   ][gc,phases   ] GC(66) Concurrent Select Relocation Set 1.384ms
[0.877s][info   ][gc,phases   ] GC(66) Pause Relocate Start 0.003ms
[0.878s][info   ][gc          ] Allocation Stall (main) 5.086ms
[0.879s][info   ][gc,phases   ] GC(66) Concurrent Relocate 2.002ms
[0.879s][info   ][gc,load     ] GC(66) Load: 4.02/4.33/4.71
[0.879s][info   ][gc,mmu      ] GC(66) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.879s][info   ][gc,marking  ] GC(66) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.879s][info   ][gc,marking  ] GC(66) Mark Stack Usage: 32M
[0.879s][info   ][gc,nmethod  ] GC(66) NMethods: 121 registered, 0 unregistered
[0.879s][info   ][gc,metaspace] GC(66) Metaspace: 0M used, 0M committed, 1088M reserved
[0.879s][info   ][gc,ref      ] GC(66) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.879s][info   ][gc,ref      ] GC(66) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.879s][info   ][gc,ref      ] GC(66) Final: 0 encountered, 0 discovered, 0 enqueued
[0.879s][info   ][gc,ref      ] GC(66) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.879s][info   ][gc,reloc    ] GC(66) Small Pages: 83 / 166M, Empty: 0M, Relocated: 20M, In-Place: 0
[0.879s][info   ][gc,reloc    ] GC(66) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 41M, In-Place: 3
[0.879s][info   ][gc,reloc    ] GC(66) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.879s][info   ][gc,reloc    ] GC(66) Forwarding Usage: 0M
[0.879s][info   ][gc,heap     ] GC(66) Min Capacity: 512M(100%)
[0.879s][info   ][gc,heap     ] GC(66) Max Capacity: 512M(100%)
[0.879s][info   ][gc,heap     ] GC(66) Soft Max Capacity: 512M(100%)
[0.879s][info   ][gc,heap     ] GC(66)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.879s][info   ][gc,heap     ] GC(66)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.879s][info   ][gc,heap     ] GC(66)      Free:       10M (2%)           10M (2%)           10M (2%)           46M (9%)           46M (9%)            6M (1%)
[0.879s][info   ][gc,heap     ] GC(66)      Used:      502M (98%)         502M (98%)         502M (98%)         466M (91%)         506M (99%)         466M (91%)
[0.879s][info   ][gc,heap     ] GC(66)      Live:         -               354M (69%)         354M (69%)         354M (69%)            -                  -
[0.879s][info   ][gc,heap     ] GC(66) Allocated:         -                 0M (0%)            0M (0%)           22M (4%)             -                  -
[0.879s][info   ][gc,heap     ] GC(66)   Garbage:         -               147M (29%)         147M (29%)          89M (17%)            -                  -
[0.879s][info   ][gc,heap     ] GC(66) Reclaimed:         -                  -                 0M (0%)           58M (11%)            -                  -
[0.880s][info   ][gc          ] GC(66) Garbage Collection (Allocation Stall) 502M(98%)->466M(91%)
[0.882s][info   ][gc,start    ] GC(67) Garbage Collection (Allocation Stall)
[0.882s][info   ][gc,ref      ] GC(67) Clearing All SoftReferences
[0.882s][info   ][gc,task     ] GC(67) Using 2 workers
[0.882s][info   ][gc,phases   ] GC(67) Pause Mark Start 0.003ms
[0.884s][info   ][gc,phases   ] GC(67) Concurrent Mark 2.133ms
[0.884s][info   ][gc,phases   ] GC(67) Pause Mark End 0.005ms
[0.884s][info   ][gc,phases   ] GC(67) Concurrent Mark Free 0.000ms
[0.884s][info   ][gc,phases   ] GC(67) Concurrent Process Non-Strong References 0.203ms
[0.884s][info   ][gc,phases   ] GC(67) Concurrent Reset Relocation Set 0.002ms
[0.886s][info   ][gc,phases   ] GC(67) Concurrent Select Relocation Set 1.421ms
[0.886s][info   ][gc,phases   ] GC(67) Pause Relocate Start 0.005ms
[0.887s][info   ][gc          ] Allocation Stall (main) 5.415ms
[0.888s][info   ][gc,phases   ] GC(67) Concurrent Relocate 2.096ms
[0.888s][info   ][gc,load     ] GC(67) Load: 4.02/4.33/4.71
[0.888s][info   ][gc,mmu      ] GC(67) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.888s][info   ][gc,marking  ] GC(67) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.888s][info   ][gc,marking  ] GC(67) Mark Stack Usage: 32M
[0.888s][info   ][gc,nmethod  ] GC(67) NMethods: 121 registered, 0 unregistered
[0.888s][info   ][gc,metaspace] GC(67) Metaspace: 0M used, 0M committed, 1088M reserved
[0.888s][info   ][gc,ref      ] GC(67) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.888s][info   ][gc,ref      ] GC(67) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.888s][info   ][gc,ref      ] GC(67) Final: 0 encountered, 0 discovered, 0 enqueued
[0.888s][info   ][gc,ref      ] GC(67) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.888s][info   ][gc,reloc    ] GC(67) Small Pages: 84 / 168M, Empty: 0M, Relocated: 20M, In-Place: 0
[0.888s][info   ][gc,reloc    ] GC(67) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 54M, In-Place: 4
[0.888s][info   ][gc,reloc    ] GC(67) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.888s][info   ][gc,reloc    ] GC(67) Forwarding Usage: 0M
[0.888s][info   ][gc,heap     ] GC(67) Min Capacity: 512M(100%)
[0.888s][info   ][gc,heap     ] GC(67) Max Capacity: 512M(100%)
[0.888s][info   ][gc,heap     ] GC(67) Soft Max Capacity: 512M(100%)
[0.888s][info   ][gc,heap     ] GC(67)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.888s][info   ][gc,heap     ] GC(67)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.888s][info   ][gc,heap     ] GC(67)      Free:        8M (2%)            8M (2%)            8M (2%)           46M (9%)           46M (9%)            4M (1%)
[0.888s][info   ][gc,heap     ] GC(67)      Used:      504M (98%)         504M (98%)         504M (98%)         466M (91%)         508M (99%)         466M (91%)
[0.888s][info   ][gc,heap     ] GC(67)      Live:         -               346M (68%)         346M (68%)         346M (68%)            -                  -
[0.888s][info   ][gc,heap     ] GC(67) Allocated:         -                 0M (0%)            0M (0%)           24M (5%)             -                  -
[0.888s][info   ][gc,heap     ] GC(67)   Garbage:         -               157M (31%)         157M (31%)          95M (19%)            -                  -
[0.888s][info   ][gc,heap     ] GC(67) Reclaimed:         -                  -                 0M (0%)           62M (12%)            -                  -
[0.888s][info   ][gc          ] GC(67) Garbage Collection (Allocation Stall) 504M(98%)->466M(91%)
[0.891s][info   ][gc,start    ] GC(68) Garbage Collection (Allocation Stall)
[0.891s][info   ][gc,ref      ] GC(68) Clearing All SoftReferences
[0.891s][info   ][gc,task     ] GC(68) Using 2 workers
[0.891s][info   ][gc,phases   ] GC(68) Pause Mark Start 0.003ms
[0.893s][info   ][gc,phases   ] GC(68) Concurrent Mark 1.826ms
[0.893s][info   ][gc,phases   ] GC(68) Pause Mark End 0.006ms
[0.893s][info   ][gc,phases   ] GC(68) Concurrent Mark Free 0.000ms
[0.893s][info   ][gc,phases   ] GC(68) Concurrent Process Non-Strong References 0.193ms
[0.893s][info   ][gc,phases   ] GC(68) Concurrent Reset Relocation Set 0.002ms
[0.895s][info   ][gc,phases   ] GC(68) Concurrent Select Relocation Set 1.413ms
[0.895s][info   ][gc,phases   ] GC(68) Pause Relocate Start 0.004ms
[0.897s][info   ][gc,phases   ] GC(68) Concurrent Relocate 1.760ms
[0.897s][info   ][gc          ] Allocation Stall (main) 5.686ms
[0.897s][info   ][gc,load     ] GC(68) Load: 4.02/4.33/4.71
[0.897s][info   ][gc,mmu      ] GC(68) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.897s][info   ][gc,marking  ] GC(68) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.897s][info   ][gc,marking  ] GC(68) Mark Stack Usage: 32M
[0.897s][info   ][gc,nmethod  ] GC(68) NMethods: 121 registered, 0 unregistered
[0.897s][info   ][gc,metaspace] GC(68) Metaspace: 0M used, 0M committed, 1088M reserved
[0.897s][info   ][gc,ref      ] GC(68) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.897s][info   ][gc,ref      ] GC(68) Weak: 23 encountered, 11 discovered, 0 enqueued
[0.897s][info   ][gc,ref      ] GC(68) Final: 0 encountered, 0 discovered, 0 enqueued
[0.897s][info   ][gc,ref      ] GC(68) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.897s][info   ][gc,reloc    ] GC(68) Small Pages: 85 / 170M, Empty: 0M, Relocated: 36M, In-Place: 0
[0.897s][info   ][gc,reloc    ] GC(68) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 40M, In-Place: 3
[0.897s][info   ][gc,reloc    ] GC(68) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.897s][info   ][gc,reloc    ] GC(68) Forwarding Usage: 0M
[0.897s][info   ][gc,heap     ] GC(68) Min Capacity: 512M(100%)
[0.897s][info   ][gc,heap     ] GC(68) Max Capacity: 512M(100%)
[0.897s][info   ][gc,heap     ] GC(68) Soft Max Capacity: 512M(100%)
[0.897s][info   ][gc,heap     ] GC(68)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.897s][info   ][gc,heap     ] GC(68)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.897s][info   ][gc,heap     ] GC(68)      Free:        6M (1%)            6M (1%)            6M (1%)           62M (12%)          62M (12%)           4M (1%)
[0.897s][info   ][gc,heap     ] GC(68)      Used:      506M (99%)         506M (99%)         506M (99%)         450M (88%)         508M (99%)         450M (88%)
[0.897s][info   ][gc,heap     ] GC(68)      Live:         -               341M (67%)         341M (67%)         341M (67%)            -                  -
[0.897s][info   ][gc,heap     ] GC(68) Allocated:         -                 0M (0%)            0M (0%)           16M (3%)             -                  -
[0.897s][info   ][gc,heap     ] GC(68)   Garbage:         -               164M (32%)         164M (32%)          92M (18%)            -                  -
[0.897s][info   ][gc,heap     ] GC(68) Reclaimed:         -                  -                 0M (0%)           72M (14%)            -                  -
[0.897s][info   ][gc          ] GC(68) Garbage Collection (Allocation Stall) 506M(99%)->450M(88%)
[0.901s][info   ][gc,start    ] GC(69) Garbage Collection (Allocation Stall)
[0.901s][info   ][gc,ref      ] GC(69) Clearing All SoftReferences
[0.901s][info   ][gc,task     ] GC(69) Using 2 workers
[0.901s][info   ][gc,phases   ] GC(69) Pause Mark Start 0.004ms
[0.903s][info   ][gc,phases   ] GC(69) Concurrent Mark 1.786ms
[0.903s][info   ][gc,phases   ] GC(69) Pause Mark End 0.003ms
[0.903s][info   ][gc,phases   ] GC(69) Concurrent Mark Free 0.001ms
[0.903s][info   ][gc,phases   ] GC(69) Concurrent Process Non-Strong References 0.233ms
[0.903s][info   ][gc,phases   ] GC(69) Concurrent Reset Relocation Set 0.003ms
[0.905s][info   ][gc,phases   ] GC(69) Concurrent Select Relocation Set 1.398ms
[0.905s][info   ][gc,phases   ] GC(69) Pause Relocate Start 0.005ms
[0.906s][info   ][gc          ] Allocation Stall (main) 5.233ms
[0.907s][info   ][gc          ] Allocation Stall (main) 0.244ms
[0.909s][info   ][gc,phases   ] GC(69) Concurrent Relocate 3.941ms
[0.909s][info   ][gc,load     ] GC(69) Load: 4.02/4.33/4.71
[0.909s][info   ][gc,mmu      ] GC(69) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.909s][info   ][gc,marking  ] GC(69) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.909s][info   ][gc,marking  ] GC(69) Mark Stack Usage: 32M
[0.909s][info   ][gc,nmethod  ] GC(69) NMethods: 121 registered, 0 unregistered
[0.909s][info   ][gc,metaspace] GC(69) Metaspace: 0M used, 0M committed, 1088M reserved
[0.909s][info   ][gc,ref      ] GC(69) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.909s][info   ][gc,ref      ] GC(69) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.909s][info   ][gc,ref      ] GC(69) Final: 0 encountered, 0 discovered, 0 enqueued
[0.909s][info   ][gc,ref      ] GC(69) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.909s][info   ][gc,reloc    ] GC(69) Small Pages: 80 / 160M, Empty: 0M, Relocated: 26M, In-Place: 0
[0.909s][info   ][gc,reloc    ] GC(69) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 82M, In-Place: 4
[0.909s][info   ][gc,reloc    ] GC(69) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.909s][info   ][gc,reloc    ] GC(69) Forwarding Usage: 0M
[0.909s][info   ][gc,heap     ] GC(69) Min Capacity: 512M(100%)
[0.909s][info   ][gc,heap     ] GC(69) Max Capacity: 512M(100%)
[0.909s][info   ][gc,heap     ] GC(69) Soft Max Capacity: 512M(100%)
[0.909s][info   ][gc,heap     ] GC(69)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.909s][info   ][gc,heap     ] GC(69)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.909s][info   ][gc,heap     ] GC(69)      Free:        0M (0%)            0M (0%)            0M (0%)           70M (14%)          70M (14%)           0M (0%)
[0.909s][info   ][gc,heap     ] GC(69)      Used:      512M (100%)        512M (100%)        512M (100%)        442M (86%)         512M (100%)        442M (86%)
[0.909s][info   ][gc,heap     ] GC(69)      Live:         -               336M (66%)         336M (66%)         336M (66%)            -                  -
[0.909s][info   ][gc,heap     ] GC(69) Allocated:         -                 0M (0%)            0M (0%)           27M (5%)             -                  -
[0.909s][info   ][gc,heap     ] GC(69)   Garbage:         -               175M (34%)         175M (34%)          77M (15%)            -                  -
[0.909s][info   ][gc,heap     ] GC(69) Reclaimed:         -                  -                 0M (0%)           97M (19%)            -                  -
[0.909s][info   ][gc          ] GC(69) Garbage Collection (Allocation Stall) 512M(100%)->442M(86%)
[0.911s][info   ][gc,start    ] GC(70) Garbage Collection (Allocation Stall)
[0.911s][info   ][gc,ref      ] GC(70) Clearing All SoftReferences
[0.911s][info   ][gc,task     ] GC(70) Using 2 workers
[0.911s][info   ][gc,phases   ] GC(70) Pause Mark Start 0.006ms
[0.913s][info   ][gc,phases   ] GC(70) Concurrent Mark 1.793ms
[0.913s][info   ][gc,phases   ] GC(70) Pause Mark End 0.004ms
[0.913s][info   ][gc,phases   ] GC(70) Concurrent Mark Free 0.000ms
[0.914s][info   ][gc,phases   ] GC(70) Concurrent Process Non-Strong References 0.233ms
[0.914s][info   ][gc,phases   ] GC(70) Concurrent Reset Relocation Set 0.003ms
[0.915s][info   ][gc,phases   ] GC(70) Concurrent Select Relocation Set 1.419ms
[0.915s][info   ][gc,phases   ] GC(70) Pause Relocate Start 0.004ms
[0.916s][info   ][gc          ] Allocation Stall (main) 4.290ms
[0.918s][info   ][gc,phases   ] GC(70) Concurrent Relocate 2.218ms
[0.918s][info   ][gc,load     ] GC(70) Load: 4.02/4.33/4.71
[0.918s][info   ][gc,mmu      ] GC(70) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.918s][info   ][gc,marking  ] GC(70) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.918s][info   ][gc,marking  ] GC(70) Mark Stack Usage: 32M
[0.918s][info   ][gc,nmethod  ] GC(70) NMethods: 121 registered, 0 unregistered
[0.918s][info   ][gc,metaspace] GC(70) Metaspace: 0M used, 0M committed, 1088M reserved
[0.918s][info   ][gc,ref      ] GC(70) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.918s][info   ][gc,ref      ] GC(70) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.918s][info   ][gc,ref      ] GC(70) Final: 0 encountered, 0 discovered, 0 enqueued
[0.918s][info   ][gc,ref      ] GC(70) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.918s][info   ][gc,reloc    ] GC(70) Small Pages: 81 / 162M, Empty: 0M, Relocated: 17M, In-Place: 0
[0.918s][info   ][gc,reloc    ] GC(70) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 69M, In-Place: 2
[0.918s][info   ][gc,reloc    ] GC(70) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.918s][info   ][gc,reloc    ] GC(70) Forwarding Usage: 0M
[0.918s][info   ][gc,heap     ] GC(70) Min Capacity: 512M(100%)
[0.918s][info   ][gc,heap     ] GC(70) Max Capacity: 512M(100%)
[0.918s][info   ][gc,heap     ] GC(70) Soft Max Capacity: 512M(100%)
[0.918s][info   ][gc,heap     ] GC(70)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.918s][info   ][gc,heap     ] GC(70)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.918s][info   ][gc,heap     ] GC(70)      Free:       14M (3%)           14M (3%)           14M (3%)           82M (16%)          82M (16%)          10M (2%)
[0.918s][info   ][gc,heap     ] GC(70)      Used:      498M (97%)         498M (97%)         498M (97%)         430M (84%)         502M (98%)         430M (84%)
[0.918s][info   ][gc,heap     ] GC(70)      Live:         -               337M (66%)         337M (66%)         337M (66%)            -                  -
[0.918s][info   ][gc,heap     ] GC(70) Allocated:         -                 0M (0%)            0M (0%)           25M (5%)             -                  -
[0.918s][info   ][gc,heap     ] GC(70)   Garbage:         -               160M (31%)         160M (31%)          66M (13%)            -                  -
[0.918s][info   ][gc,heap     ] GC(70) Reclaimed:         -                  -                 0M (0%)           93M (18%)            -                  -
[0.918s][info   ][gc          ] GC(70) Garbage Collection (Allocation Stall) 498M(97%)->430M(84%)
[0.922s][info   ][gc,start    ] GC(71) Garbage Collection (Allocation Stall)
[0.922s][info   ][gc,ref      ] GC(71) Clearing All SoftReferences
[0.922s][info   ][gc,task     ] GC(71) Using 2 workers
[0.922s][info   ][gc,phases   ] GC(71) Pause Mark Start 0.012ms
[0.926s][info   ][gc,phases   ] GC(71) Concurrent Mark 3.452ms
[0.926s][info   ][gc,phases   ] GC(71) Pause Mark End 0.012ms
[0.926s][info   ][gc,phases   ] GC(71) Concurrent Mark Free 0.000ms
[0.927s][info   ][gc,phases   ] GC(71) Concurrent Process Non-Strong References 0.274ms
[0.927s][info   ][gc,phases   ] GC(71) Concurrent Reset Relocation Set 0.002ms
[0.928s][info   ][gc,phases   ] GC(71) Concurrent Select Relocation Set 1.449ms
[0.928s][info   ][gc,phases   ] GC(71) Pause Relocate Start 0.007ms
[0.930s][info   ][gc          ] Allocation Stall (main) 7.966ms
[0.932s][info   ][gc,phases   ] GC(71) Concurrent Relocate 4.215ms
[0.933s][info   ][gc,load     ] GC(71) Load: 4.02/4.33/4.71
[0.933s][info   ][gc,mmu      ] GC(71) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.933s][info   ][gc,marking  ] GC(71) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.933s][info   ][gc,marking  ] GC(71) Mark Stack Usage: 32M
[0.933s][info   ][gc,nmethod  ] GC(71) NMethods: 121 registered, 0 unregistered
[0.933s][info   ][gc,metaspace] GC(71) Metaspace: 0M used, 0M committed, 1088M reserved
[0.933s][info   ][gc,ref      ] GC(71) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.933s][info   ][gc,ref      ] GC(71) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.933s][info   ][gc,ref      ] GC(71) Final: 0 encountered, 0 discovered, 0 enqueued
[0.933s][info   ][gc,ref      ] GC(71) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.933s][info   ][gc,reloc    ] GC(71) Small Pages: 88 / 176M, Empty: 0M, Relocated: 31M, In-Place: 0
[0.933s][info   ][gc,reloc    ] GC(71) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 54M, In-Place: 2
[0.933s][info   ][gc,reloc    ] GC(71) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.933s][info   ][gc,reloc    ] GC(71) Forwarding Usage: 0M
[0.933s][info   ][gc,heap     ] GC(71) Min Capacity: 512M(100%)
[0.933s][info   ][gc,heap     ] GC(71) Max Capacity: 512M(100%)
[0.933s][info   ][gc,heap     ] GC(71) Soft Max Capacity: 512M(100%)
[0.933s][info   ][gc,heap     ] GC(71)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.933s][info   ][gc,heap     ] GC(71)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.933s][info   ][gc,heap     ] GC(71)      Free:        0M (0%)            0M (0%)            0M (0%)           60M (12%)          60M (12%)           0M (0%)
[0.933s][info   ][gc,heap     ] GC(71)      Used:      512M (100%)        512M (100%)        512M (100%)        452M (88%)         512M (100%)        452M (88%)
[0.933s][info   ][gc,heap     ] GC(71)      Live:         -               341M (67%)         341M (67%)         341M (67%)            -                  -
[0.933s][info   ][gc,heap     ] GC(71) Allocated:         -                 0M (0%)            0M (0%)           31M (6%)             -                  -
[0.933s][info   ][gc,heap     ] GC(71)   Garbage:         -               170M (33%)         170M (33%)          78M (15%)            -                  -
[0.933s][info   ][gc,heap     ] GC(71) Reclaimed:         -                  -                 0M (0%)           91M (18%)            -                  -
[0.933s][info   ][gc          ] GC(71) Garbage Collection (Allocation Stall) 512M(100%)->452M(88%)
[0.937s][info   ][gc,start    ] GC(72) Garbage Collection (Allocation Stall)
[0.937s][info   ][gc,ref      ] GC(72) Clearing All SoftReferences
[0.937s][info   ][gc,task     ] GC(72) Using 2 workers
[0.937s][info   ][gc,phases   ] GC(72) Pause Mark Start 0.007ms
[0.940s][info   ][gc,phases   ] GC(72) Concurrent Mark 2.289ms
[0.941s][info   ][gc,phases   ] GC(72) Pause Mark End 0.007ms
[0.941s][info   ][gc,phases   ] GC(72) Concurrent Mark Free 0.000ms
[0.941s][info   ][gc,phases   ] GC(72) Concurrent Process Non-Strong References 0.264ms
[0.941s][info   ][gc,phases   ] GC(72) Concurrent Reset Relocation Set 0.003ms
[0.943s][info   ][gc,phases   ] GC(72) Concurrent Select Relocation Set 1.734ms
[0.943s][info   ][gc,phases   ] GC(72) Pause Relocate Start 0.005ms
[0.945s][info   ][gc,phases   ] GC(72) Concurrent Relocate 2.002ms
[0.945s][info   ][gc,load     ] GC(72) Load: 4.02/4.33/4.71
[0.945s][info   ][gc,mmu      ] GC(72) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.945s][info   ][gc,marking  ] GC(72) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.945s][info   ][gc,marking  ] GC(72) Mark Stack Usage: 32M
[0.945s][info   ][gc,nmethod  ] GC(72) NMethods: 121 registered, 0 unregistered
[0.945s][info   ][gc,metaspace] GC(72) Metaspace: 0M used, 0M committed, 1088M reserved
[0.945s][info   ][gc,ref      ] GC(72) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.945s][info   ][gc          ] Allocation Stall (main) 8.139ms
[0.945s][info   ][gc,ref      ] GC(72) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.946s][info   ][gc,ref      ] GC(72) Final: 0 encountered, 0 discovered, 0 enqueued
[0.946s][info   ][gc,ref      ] GC(72) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.946s][info   ][gc,reloc    ] GC(72) Small Pages: 88 / 176M, Empty: 0M, Relocated: 31M, In-Place: 1
[0.946s][info   ][gc,reloc    ] GC(72) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 39M, In-Place: 3
[0.946s][info   ][gc,reloc    ] GC(72) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.946s][info   ][gc,reloc    ] GC(72) Forwarding Usage: 0M
[0.946s][info   ][gc,heap     ] GC(72) Min Capacity: 512M(100%)
[0.946s][info   ][gc,heap     ] GC(72) Max Capacity: 512M(100%)
[0.946s][info   ][gc,heap     ] GC(72) Soft Max Capacity: 512M(100%)
[0.946s][info   ][gc,heap     ] GC(72)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.946s][info   ][gc,heap     ] GC(72)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.946s][info   ][gc,heap     ] GC(72)      Free:        0M (0%)            0M (0%)            0M (0%)           60M (12%)          60M (12%)           0M (0%)
[0.946s][info   ][gc,heap     ] GC(72)      Used:      512M (100%)        512M (100%)        512M (100%)        452M (88%)         512M (100%)        452M (88%)
[0.946s][info   ][gc,heap     ] GC(72)      Live:         -               347M (68%)         347M (68%)         347M (68%)            -                  -
[0.946s][info   ][gc,heap     ] GC(72) Allocated:         -                 0M (0%)            0M (0%)           16M (3%)             -                  -
[0.946s][info   ][gc,heap     ] GC(72)   Garbage:         -               164M (32%)         164M (32%)          88M (17%)            -                  -
[0.947s][info   ][gc,heap     ] GC(72) Reclaimed:         -                  -                 0M (0%)           76M (15%)            -                  -
[0.947s][info   ][gc          ] GC(72) Garbage Collection (Allocation Stall) 512M(100%)->452M(88%)
[0.949s][info   ][gc,start    ] GC(73) Garbage Collection (Allocation Stall)
[0.950s][info   ][gc,ref      ] GC(73) Clearing All SoftReferences
[0.950s][info   ][gc,task     ] GC(73) Using 2 workers
[0.950s][info   ][gc,phases   ] GC(73) Pause Mark Start 0.004ms
[0.952s][info   ][gc,phases   ] GC(73) Concurrent Mark 2.526ms
[0.952s][info   ][gc,phases   ] GC(73) Pause Mark End 0.024ms
[0.952s][info   ][gc,phases   ] GC(73) Concurrent Mark Free 0.000ms
[0.953s][info   ][gc,phases   ] GC(73) Concurrent Process Non-Strong References 0.739ms
[0.953s][info   ][gc,phases   ] GC(73) Concurrent Reset Relocation Set 0.003ms
[0.955s][info   ][gc,phases   ] GC(73) Concurrent Select Relocation Set 1.491ms
[0.955s][info   ][gc,phases   ] GC(73) Pause Relocate Start 0.008ms
[0.956s][info   ][gc          ] Allocation Stall (main) 6.990ms
[0.958s][info   ][gc,phases   ] GC(73) Concurrent Relocate 2.941ms
[0.958s][info   ][gc,load     ] GC(73) Load: 4.02/4.33/4.71
[0.958s][info   ][gc,mmu      ] GC(73) MMU: 2ms/98.7%, 5ms/99.4%, 10ms/99.6%, 20ms/99.7%, 50ms/99.8%, 100ms/99.8%
[0.958s][info   ][gc,marking  ] GC(73) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.958s][info   ][gc,marking  ] GC(73) Mark Stack Usage: 32M
[0.958s][info   ][gc,nmethod  ] GC(73) NMethods: 121 registered, 0 unregistered
[0.958s][info   ][gc,metaspace] GC(73) Metaspace: 0M used, 0M committed, 1088M reserved
[0.958s][info   ][gc,ref      ] GC(73) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.958s][info   ][gc,ref      ] GC(73) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.958s][info   ][gc,ref      ] GC(73) Final: 0 encountered, 0 discovered, 0 enqueued
[0.958s][info   ][gc,ref      ] GC(73) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.958s][info   ][gc,reloc    ] GC(73) Small Pages: 80 / 160M, Empty: 0M, Relocated: 17M, In-Place: 0
[0.958s][info   ][gc,reloc    ] GC(73) Medium Pages: 22 / 352M, Empty: 0M, Relocated: 41M, In-Place: 3
[0.958s][info   ][gc,reloc    ] GC(73) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.958s][info   ][gc,reloc    ] GC(73) Forwarding Usage: 0M
[0.958s][info   ][gc,heap     ] GC(73) Min Capacity: 512M(100%)
[0.958s][info   ][gc,heap     ] GC(73) Max Capacity: 512M(100%)
[0.958s][info   ][gc,heap     ] GC(73) Soft Max Capacity: 512M(100%)
[0.958s][info   ][gc,heap     ] GC(73)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.958s][info   ][gc,heap     ] GC(73)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.958s][info   ][gc,heap     ] GC(73)      Free:        0M (0%)            0M (0%)            0M (0%)           48M (9%)           48M (9%)            0M (0%)
[0.959s][info   ][gc,heap     ] GC(73)      Used:      512M (100%)        512M (100%)        512M (100%)        464M (91%)         512M (100%)        464M (91%)
[0.959s][info   ][gc,heap     ] GC(73)      Live:         -               345M (67%)         345M (67%)         345M (67%)            -                  -
[0.959s][info   ][gc,heap     ] GC(73) Allocated:         -                 0M (0%)            0M (0%)           24M (5%)             -                  -
[0.959s][info   ][gc,heap     ] GC(73)   Garbage:         -               166M (33%)         166M (33%)          94M (19%)            -                  -
[0.959s][info   ][gc,heap     ] GC(73) Reclaimed:         -                  -                 0M (0%)           72M (14%)            -                  -
[0.959s][info   ][gc          ] GC(73) Garbage Collection (Allocation Stall) 512M(100%)->464M(91%)
[0.962s][info   ][gc,start    ] GC(74) Garbage Collection (Allocation Stall)
[0.962s][info   ][gc,ref      ] GC(74) Clearing All SoftReferences
[0.962s][info   ][gc,task     ] GC(74) Using 2 workers
[0.962s][info   ][gc,phases   ] GC(74) Pause Mark Start 0.006ms
[0.965s][info   ][gc,phases   ] GC(74) Concurrent Mark 3.067ms
[0.965s][info   ][gc,phases   ] GC(74) Pause Mark End 0.058ms
[0.965s][info   ][gc,phases   ] GC(74) Concurrent Mark Free 0.000ms
[0.966s][info   ][gc,phases   ] GC(74) Concurrent Process Non-Strong References 0.555ms
[0.966s][info   ][gc,phases   ] GC(74) Concurrent Reset Relocation Set 0.002ms
[0.968s][info   ][gc,phases   ] GC(74) Concurrent Select Relocation Set 1.962ms
[0.968s][info   ][gc,phases   ] GC(74) Pause Relocate Start 0.007ms
[0.969s][info   ][gc          ] Allocation Stall (main) 7.516ms
[0.971s][info   ][gc,phases   ] GC(74) Concurrent Relocate 3.156ms
[0.971s][info   ][gc,load     ] GC(74) Load: 4.02/4.33/4.71
[0.971s][info   ][gc,mmu      ] GC(74) MMU: 2ms/97.1%, 5ms/98.7%, 10ms/99.3%, 20ms/99.5%, 50ms/99.7%, 100ms/99.8%
[0.972s][info   ][gc,marking  ] GC(74) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.972s][info   ][gc,marking  ] GC(74) Mark Stack Usage: 32M
[0.972s][info   ][gc,nmethod  ] GC(74) NMethods: 121 registered, 0 unregistered
[0.972s][info   ][gc,metaspace] GC(74) Metaspace: 0M used, 0M committed, 1088M reserved
[0.972s][info   ][gc,ref      ] GC(74) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.972s][info   ][gc,ref      ] GC(74) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.972s][info   ][gc,ref      ] GC(74) Final: 0 encountered, 0 discovered, 0 enqueued
[0.972s][info   ][gc,ref      ] GC(74) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.972s][info   ][gc,reloc    ] GC(74) Small Pages: 86 / 172M, Empty: 0M, Relocated: 29M, In-Place: 0
[0.972s][info   ][gc,reloc    ] GC(74) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 69M, In-Place: 4
[0.972s][info   ][gc,reloc    ] GC(74) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.972s][info   ][gc,reloc    ] GC(74) Forwarding Usage: 0M
[0.972s][info   ][gc,heap     ] GC(74) Min Capacity: 512M(100%)
[0.972s][info   ][gc,heap     ] GC(74) Max Capacity: 512M(100%)
[0.972s][info   ][gc,heap     ] GC(74) Soft Max Capacity: 512M(100%)
[0.972s][info   ][gc,heap     ] GC(74)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.972s][info   ][gc,heap     ] GC(74)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.972s][info   ][gc,heap     ] GC(74)      Free:        4M (1%)            4M (1%)            4M (1%)           46M (9%)           58M (11%)           0M (0%)
[0.972s][info   ][gc,heap     ] GC(74)      Used:      508M (99%)         508M (99%)         508M (99%)         466M (91%)         512M (100%)        454M (89%)
[0.972s][info   ][gc,heap     ] GC(74)      Live:         -               343M (67%)         343M (67%)         343M (67%)            -                  -
[0.972s][info   ][gc,heap     ] GC(74) Allocated:         -                 0M (0%)            0M (0%)           46M (9%)             -                  -
[0.972s][info   ][gc,heap     ] GC(74)   Garbage:         -               164M (32%)         164M (32%)          76M (15%)            -                  -
[0.972s][info   ][gc,heap     ] GC(74) Reclaimed:         -                  -                 0M (0%)           88M (17%)            -                  -
[0.972s][info   ][gc          ] GC(74) Garbage Collection (Allocation Stall) 508M(99%)->466M(91%)
[0.976s][info   ][gc,start    ] GC(75) Garbage Collection (Allocation Stall)
[0.976s][info   ][gc,ref      ] GC(75) Clearing All SoftReferences
[0.976s][info   ][gc,task     ] GC(75) Using 2 workers
[0.976s][info   ][gc,phases   ] GC(75) Pause Mark Start 0.006ms
[0.978s][info   ][gc,phases   ] GC(75) Concurrent Mark 2.190ms
[0.978s][info   ][gc,phases   ] GC(75) Pause Mark End 0.004ms
[0.978s][info   ][gc,phases   ] GC(75) Concurrent Mark Free 0.000ms
[0.979s][info   ][gc,phases   ] GC(75) Concurrent Process Non-Strong References 0.189ms
[0.979s][info   ][gc,phases   ] GC(75) Concurrent Reset Relocation Set 0.003ms
[0.980s][info   ][gc,phases   ] GC(75) Concurrent Select Relocation Set 1.407ms
[0.980s][info   ][gc,phases   ] GC(75) Pause Relocate Start 0.002ms
[0.981s][info   ][gc          ] Allocation Stall (main) 4.963ms
[0.981s][info   ][gc          ] Allocation Stall (main) 0.518ms
[0.983s][info   ][gc,phases   ] GC(75) Concurrent Relocate 3.130ms
[0.983s][info   ][gc,load     ] GC(75) Load: 4.02/4.33/4.71
[0.983s][info   ][gc,mmu      ] GC(75) MMU: 2ms/97.1%, 5ms/98.7%, 10ms/99.3%, 20ms/99.5%, 50ms/99.7%, 100ms/99.8%
[0.983s][info   ][gc,marking  ] GC(75) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.983s][info   ][gc,marking  ] GC(75) Mark Stack Usage: 32M
[0.983s][info   ][gc,nmethod  ] GC(75) NMethods: 121 registered, 0 unregistered
[0.983s][info   ][gc,metaspace] GC(75) Metaspace: 0M used, 0M committed, 1088M reserved
[0.983s][info   ][gc,ref      ] GC(75) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.983s][info   ][gc,ref      ] GC(75) Weak: 23 encountered, 6 discovered, 0 enqueued
[0.983s][info   ][gc,ref      ] GC(75) Final: 0 encountered, 0 discovered, 0 enqueued
[0.983s][info   ][gc,ref      ] GC(75) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.983s][info   ][gc,reloc    ] GC(75) Small Pages: 88 / 176M, Empty: 0M, Relocated: 26M, In-Place: 0
[0.983s][info   ][gc,reloc    ] GC(75) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 69M, In-Place: 5
[0.983s][info   ][gc,reloc    ] GC(75) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.983s][info   ][gc,reloc    ] GC(75) Forwarding Usage: 0M
[0.983s][info   ][gc,heap     ] GC(75) Min Capacity: 512M(100%)
[0.983s][info   ][gc,heap     ] GC(75) Max Capacity: 512M(100%)
[0.983s][info   ][gc,heap     ] GC(75) Soft Max Capacity: 512M(100%)
[0.983s][info   ][gc,heap     ] GC(75)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.983s][info   ][gc,heap     ] GC(75)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.983s][info   ][gc,heap     ] GC(75)      Free:        0M (0%)            0M (0%)            0M (0%)           62M (12%)          62M (12%)           0M (0%)
[0.983s][info   ][gc,heap     ] GC(75)      Used:      512M (100%)        512M (100%)        512M (100%)        450M (88%)         512M (100%)        450M (88%)
[0.983s][info   ][gc,heap     ] GC(75)      Live:         -               343M (67%)         343M (67%)         343M (67%)            -                  -
[0.984s][info   ][gc,heap     ] GC(75) Allocated:         -                 0M (0%)            0M (0%)           26M (5%)             -                  -
[0.984s][info   ][gc,heap     ] GC(75)   Garbage:         -               168M (33%)         168M (33%)          80M (16%)            -                  -
[0.984s][info   ][gc,heap     ] GC(75) Reclaimed:         -                  -                 0M (0%)           88M (17%)            -                  -
[0.984s][info   ][gc          ] GC(75) Garbage Collection (Allocation Stall) 512M(100%)->450M(88%)
[0.988s][info   ][gc,start    ] GC(76) Garbage Collection (Allocation Stall)
[0.988s][info   ][gc,ref      ] GC(76) Clearing All SoftReferences
[0.988s][info   ][gc,task     ] GC(76) Using 2 workers
[0.988s][info   ][gc,phases   ] GC(76) Pause Mark Start 0.007ms
[0.990s][info   ][gc,phases   ] GC(76) Concurrent Mark 2.313ms
[0.990s][info   ][gc,phases   ] GC(76) Pause Mark End 0.004ms
[0.990s][info   ][gc,phases   ] GC(76) Concurrent Mark Free 0.000ms
[0.991s][info   ][gc,phases   ] GC(76) Concurrent Process Non-Strong References 0.274ms
[0.991s][info   ][gc,phases   ] GC(76) Concurrent Reset Relocation Set 0.002ms
[0.992s][info   ][gc,phases   ] GC(76) Concurrent Select Relocation Set 1.449ms
[0.992s][info   ][gc,phases   ] GC(76) Pause Relocate Start 0.006ms
[0.993s][info   ][gc          ] Allocation Stall (main) 5.269ms
[0.993s][info   ][gc          ] Allocation Stall (main) 0.297ms
[0.994s][info   ][gc,phases   ] GC(76) Concurrent Relocate 2.334ms
[0.995s][info   ][gc,load     ] GC(76) Load: 4.02/4.33/4.71
[0.995s][info   ][gc,mmu      ] GC(76) MMU: 2ms/97.1%, 5ms/98.7%, 10ms/99.3%, 20ms/99.5%, 50ms/99.7%, 100ms/99.8%
[0.995s][info   ][gc,marking  ] GC(76) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.995s][info   ][gc,marking  ] GC(76) Mark Stack Usage: 32M
[0.995s][info   ][gc,nmethod  ] GC(76) NMethods: 121 registered, 0 unregistered
[0.995s][info   ][gc,metaspace] GC(76) Metaspace: 0M used, 0M committed, 1088M reserved
[0.995s][info   ][gc,ref      ] GC(76) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.995s][info   ][gc,ref      ] GC(76) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.995s][info   ][gc,ref      ] GC(76) Final: 0 encountered, 0 discovered, 0 enqueued
[0.995s][info   ][gc,ref      ] GC(76) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.995s][info   ][gc,reloc    ] GC(76) Small Pages: 88 / 176M, Empty: 0M, Relocated: 22M, In-Place: 0
[0.995s][info   ][gc,reloc    ] GC(76) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 53M, In-Place: 3
[0.995s][info   ][gc,reloc    ] GC(76) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.995s][info   ][gc,reloc    ] GC(76) Forwarding Usage: 0M
[0.995s][info   ][gc,heap     ] GC(76) Min Capacity: 512M(100%)
[0.995s][info   ][gc,heap     ] GC(76) Max Capacity: 512M(100%)
[0.995s][info   ][gc,heap     ] GC(76) Soft Max Capacity: 512M(100%)
[0.995s][info   ][gc,heap     ] GC(76)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.995s][info   ][gc,heap     ] GC(76)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[0.995s][info   ][gc,heap     ] GC(76)      Free:        0M (0%)            0M (0%)            0M (0%)           62M (12%)          62M (12%)           0M (0%)
[0.995s][info   ][gc,heap     ] GC(76)      Used:      512M (100%)        512M (100%)        512M (100%)        450M (88%)         512M (100%)        450M (88%)
[0.995s][info   ][gc,heap     ] GC(76)      Live:         -               341M (67%)         341M (67%)         341M (67%)            -                  -
[0.995s][info   ][gc,heap     ] GC(76) Allocated:         -                 0M (0%)            0M (0%)           24M (5%)             -                  -
[0.995s][info   ][gc,heap     ] GC(76)   Garbage:         -               170M (33%)         170M (33%)          84M (16%)            -                  -
[0.995s][info   ][gc,heap     ] GC(76) Reclaimed:         -                  -                 0M (0%)           86M (17%)            -                  -
[0.995s][info   ][gc          ] GC(76) Garbage Collection (Allocation Stall) 512M(100%)->450M(88%)
[0.999s][info   ][gc,start    ] GC(77) Garbage Collection (Allocation Stall)
[0.999s][info   ][gc,ref      ] GC(77) Clearing All SoftReferences
[0.999s][info   ][gc,task     ] GC(77) Using 2 workers
[0.999s][info   ][gc,phases   ] GC(77) Pause Mark Start 0.005ms
[1.002s][info   ][gc,phases   ] GC(77) Concurrent Mark 2.347ms
[1.002s][info   ][gc,phases   ] GC(77) Pause Mark End 0.006ms
[1.002s][info   ][gc,phases   ] GC(77) Concurrent Mark Free 0.000ms
[1.002s][info   ][gc,phases   ] GC(77) Concurrent Process Non-Strong References 0.229ms
[1.002s][info   ][gc,phases   ] GC(77) Concurrent Reset Relocation Set 0.003ms
[1.003s][info   ][gc,phases   ] GC(77) Concurrent Select Relocation Set 1.404ms
[1.003s][info   ][gc,phases   ] GC(77) Pause Relocate Start 0.002ms
[1.004s][info   ][gc          ] Allocation Stall (main) 4.928ms
[1.004s][info   ][gc          ] Allocation Stall (main) 0.043ms
[1.006s][info   ][gc,phases   ] GC(77) Concurrent Relocate 2.666ms
[1.006s][info   ][gc,load     ] GC(77) Load: 4.02/4.33/4.71
[1.006s][info   ][gc,mmu      ] GC(77) MMU: 2ms/97.1%, 5ms/98.7%, 10ms/99.3%, 20ms/99.5%, 50ms/99.7%, 100ms/99.8%
[1.006s][info   ][gc,marking  ] GC(77) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[1.006s][info   ][gc,marking  ] GC(77) Mark Stack Usage: 32M
[1.006s][info   ][gc,nmethod  ] GC(77) NMethods: 121 registered, 0 unregistered
[1.006s][info   ][gc,metaspace] GC(77) Metaspace: 0M used, 0M committed, 1088M reserved
[1.006s][info   ][gc,ref      ] GC(77) Soft: 3 encountered, 0 discovered, 0 enqueued
[1.006s][info   ][gc,ref      ] GC(77) Weak: 23 encountered, 17 discovered, 0 enqueued
[1.006s][info   ][gc,ref      ] GC(77) Final: 0 encountered, 0 discovered, 0 enqueued
[1.006s][info   ][gc,ref      ] GC(77) Phantom: 1 encountered, 1 discovered, 0 enqueued
[1.006s][info   ][gc,reloc    ] GC(77) Small Pages: 87 / 174M, Empty: 0M, Relocated: 22M, In-Place: 0
[1.006s][info   ][gc,reloc    ] GC(77) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 69M, In-Place: 5
[1.006s][info   ][gc,reloc    ] GC(77) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[1.006s][info   ][gc,reloc    ] GC(77) Forwarding Usage: 0M
[1.006s][info   ][gc,heap     ] GC(77) Min Capacity: 512M(100%)
[1.006s][info   ][gc,heap     ] GC(77) Max Capacity: 512M(100%)
[1.006s][info   ][gc,heap     ] GC(77) Soft Max Capacity: 512M(100%)
[1.006s][info   ][gc,heap     ] GC(77)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[1.006s][info   ][gc,heap     ] GC(77)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[1.006s][info   ][gc,heap     ] GC(77)      Free:        2M (0%)            2M (0%)            2M (0%)           56M (11%)          58M (11%)           0M (0%)
[1.006s][info   ][gc,heap     ] GC(77)      Used:      510M (100%)        510M (100%)        510M (100%)        456M (89%)         512M (100%)        454M (89%)
[1.006s][info   ][gc,heap     ] GC(77)      Live:         -               350M (69%)         350M (69%)         350M (69%)            -                  -
[1.007s][info   ][gc,heap     ] GC(77) Allocated:         -                 0M (0%)            0M (0%)           28M (5%)             -                  -
[1.007s][info   ][gc,heap     ] GC(77)   Garbage:         -               159M (31%)         159M (31%)          77M (15%)            -                  -
[1.007s][info   ][gc,heap     ] GC(77) Reclaimed:         -                  -                 0M (0%)           82M (16%)            -                  -
[1.007s][info   ][gc          ] GC(77) Garbage Collection (Allocation Stall) 510M(100%)->456M(89%)
[1.010s][info   ][gc,start    ] GC(78) Garbage Collection (Allocation Stall)
[1.010s][info   ][gc,ref      ] GC(78) Clearing All SoftReferences
[1.010s][info   ][gc,task     ] GC(78) Using 2 workers
[1.010s][info   ][gc,phases   ] GC(78) Pause Mark Start 0.003ms
[1.012s][info   ][gc,phases   ] GC(78) Concurrent Mark 2.156ms
[1.012s][info   ][gc,phases   ] GC(78) Pause Mark End 0.003ms
[1.012s][info   ][gc,phases   ] GC(78) Concurrent Mark Free 0.000ms
[1.012s][info   ][gc,phases   ] GC(78) Concurrent Process Non-Strong References 0.220ms
[1.012s][info   ][gc,phases   ] GC(78) Concurrent Reset Relocation Set 0.003ms
[1.014s][info   ][gc,phases   ] GC(78) Concurrent Select Relocation Set 1.407ms
[1.014s][info   ][gc,phases   ] GC(78) Pause Relocate Start 0.006ms
[1.015s][info   ][gc          ] Allocation Stall (main) 5.070ms
[1.016s][info   ][gc,phases   ] GC(78) Concurrent Relocate 1.897ms
[1.016s][info   ][gc,load     ] GC(78) Load: 4.02/4.33/4.71
[1.016s][info   ][gc,mmu      ] GC(78) MMU: 2ms/97.1%, 5ms/98.7%, 10ms/99.3%, 20ms/99.5%, 50ms/99.7%, 100ms/99.8%
[1.016s][info   ][gc,marking  ] GC(78) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[1.016s][info   ][gc,marking  ] GC(78) Mark Stack Usage: 32M
[1.016s][info   ][gc,nmethod  ] GC(78) NMethods: 121 registered, 0 unregistered
[1.016s][info   ][gc,metaspace] GC(78) Metaspace: 0M used, 0M committed, 1088M reserved
[1.016s][info   ][gc,ref      ] GC(78) Soft: 3 encountered, 0 discovered, 0 enqueued
[1.016s][info   ][gc,ref      ] GC(78) Weak: 23 encountered, 3 discovered, 0 enqueued
[1.016s][info   ][gc,ref      ] GC(78) Final: 0 encountered, 0 discovered, 0 enqueued
[1.016s][info   ][gc,ref      ] GC(78) Phantom: 1 encountered, 1 discovered, 0 enqueued
[1.016s][info   ][gc,reloc    ] GC(78) Small Pages: 88 / 176M, Empty: 0M, Relocated: 26M, In-Place: 0
[1.016s][info   ][gc,reloc    ] GC(78) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 13M, In-Place: 1
[1.016s][info   ][gc,reloc    ] GC(78) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[1.016s][info   ][gc,reloc    ] GC(78) Forwarding Usage: 0M
[1.016s][info   ][gc,heap     ] GC(78) Min Capacity: 512M(100%)
[1.016s][info   ][gc,heap     ] GC(78) Max Capacity: 512M(100%)
[1.016s][info   ][gc,heap     ] GC(78) Soft Max Capacity: 512M(100%)
[1.016s][info   ][gc,heap     ] GC(78)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[1.016s][info   ][gc,heap     ] GC(78)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[1.016s][info   ][gc,heap     ] GC(78)      Free:        0M (0%)            0M (0%)            0M (0%)           50M (10%)          50M (10%)           0M (0%)
[1.016s][info   ][gc,heap     ] GC(78)      Used:      512M (100%)        512M (100%)        512M (100%)        462M (90%)         512M (100%)        462M (90%)
[1.016s][info   ][gc,heap     ] GC(78)      Live:         -               348M (68%)         348M (68%)         348M (68%)            -                  -
[1.016s][info   ][gc,heap     ] GC(78) Allocated:         -                 0M (0%)            0M (0%)           21M (4%)             -                  -
[1.016s][info   ][gc,heap     ] GC(78)   Garbage:         -               163M (32%)         163M (32%)          91M (18%)            -                  -
[1.016s][info   ][gc,heap     ] GC(78) Reclaimed:         -                  -                 0M (0%)           71M (14%)            -                  -
[1.016s][info   ][gc          ] GC(78) Garbage Collection (Allocation Stall) 512M(100%)->462M(90%)
[1.019s][info   ][gc,start    ] GC(79) Garbage Collection (Warmup)
[1.019s][info   ][gc,task     ] GC(79) Using 2 workers
[1.019s][info   ][gc,phases   ] GC(79) Pause Mark Start 0.003ms
[1.021s][info   ][gc,phases   ] GC(79) Concurrent Mark 1.778ms
[1.021s][info   ][gc,phases   ] GC(79) Pause Mark End 0.004ms
[1.021s][info   ][gc,phases   ] GC(79) Concurrent Mark Free 0.001ms
[1.021s][info   ][gc,phases   ] GC(79) Concurrent Process Non-Strong References 0.283ms
[1.021s][info   ][gc,phases   ] GC(79) Concurrent Reset Relocation Set 0.002ms
[1.023s][info   ][gc          ] Allocation Stall (main) 2.728ms
[1.024s][info   ][gc,phases   ] GC(79) Concurrent Select Relocation Set 2.969ms
[1.024s][info   ][gc,phases   ] GC(79) Pause Relocate Start 0.003ms
[1.028s][info   ][gc          ] Relocation Stall (main) 2.880ms
[1.028s][info   ][gc,phases   ] GC(79) Concurrent Relocate 3.598ms
[1.028s][info   ][gc,load     ] GC(79) Load: 4.02/4.33/4.71
[1.028s][info   ][gc,mmu      ] GC(79) MMU: 2ms/97.1%, 5ms/98.7%, 10ms/99.3%, 20ms/99.5%, 50ms/99.7%, 100ms/99.8%
[1.028s][info   ][gc,marking  ] GC(79) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[1.028s][info   ][gc,marking  ] GC(79) Mark Stack Usage: 32M
[1.028s][info   ][gc,nmethod  ] GC(79) NMethods: 124 registered, 0 unregistered
[1.028s][info   ][gc,metaspace] GC(79) Metaspace: 0M used, 0M committed, 1088M reserved
[1.028s][info   ][gc,ref      ] GC(79) Soft: 3 encountered, 0 discovered, 0 enqueued
[1.028s][info   ][gc,ref      ] GC(79) Weak: 23 encountered, 13 discovered, 0 enqueued
[1.028s][info   ][gc,ref      ] GC(79) Final: 0 encountered, 0 discovered, 0 enqueued
[1.028s][info   ][gc,ref      ] GC(79) Phantom: 1 encountered, 1 discovered, 0 enqueued
[1.028s][info   ][gc,reloc    ] GC(79) Small Pages: 79 / 158M, Empty: 2M, Relocated: 13M, In-Place: 0
[1.028s][info   ][gc,reloc    ] GC(79) Medium Pages: 21 / 336M, Empty: 0M, Relocated: 82M, In-Place: 2
[1.028s][info   ][gc,reloc    ] GC(79) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[1.028s][info   ][gc,reloc    ] GC(79) Forwarding Usage: 0M
[1.028s][info   ][gc,heap     ] GC(79) Min Capacity: 512M(100%)
[1.028s][info   ][gc,heap     ] GC(79) Max Capacity: 512M(100%)
[1.028s][info   ][gc,heap     ] GC(79) Soft Max Capacity: 512M(100%)
[1.028s][info   ][gc,heap     ] GC(79)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[1.028s][info   ][gc,heap     ] GC(79)  Capacity:      512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)        512M (100%)
[1.028s][info   ][gc,heap     ] GC(79)      Free:       18M (4%)            0M (0%)            0M (0%)           66M (13%)          66M (13%)           0M (0%)
[1.028s][info   ][gc,heap     ] GC(79)      Used:      494M (96%)         512M (100%)        512M (100%)        446M (87%)         512M (100%)        446M (87%)
[1.028s][info   ][gc,heap     ] GC(79)      Live:         -               348M (68%)         348M (68%)         348M (68%)            -                  -
[1.028s][info   ][gc,heap     ] GC(79) Allocated:         -                18M (4%)           20M (4%)           19M (4%)             -                  -
[1.028s][info   ][gc,heap     ] GC(79)   Garbage:         -               145M (28%)         143M (28%)          77M (15%)            -                  -
[1.028s][info   ][gc,heap     ] GC(79) Reclaimed:         -                  -                 2M (0%)           67M (13%)            -                  -
[1.028s][info   ][gc          ] GC(79) Garbage Collection (Warmup) 494M(96%)->446M(87%)
counter:35431
[1.033s][info   ][gc,heap,exit] Heap
[1.033s][info   ][gc,heap,exit]  ZHeap           used 448M, capacity 512M, max capacity 512M
[1.033s][info   ][gc,heap,exit]  Metaspace       used 274K, committed 448K, reserved 1114112K
[1.033s][info   ][gc,heap,exit]   class space    used 9K, committed 128K, reserved 1048576K
```

再看看1g内存
```
java -XX:+UseZGC -Xms1g -Xmx1g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

结果22次GC,生成了57000多个对象,GC次数相比G1减少了很多，性能也差不多。
```
[0.003s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.006s][info   ][gc,init] Initializing The Z Garbage Collector
[0.006s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.006s][info   ][gc,init] NUMA Support: Disabled
[0.006s][info   ][gc,init] CPUs: 8 total, 8 available
[0.006s][info   ][gc,init] Memory: 16384M
[0.006s][info   ][gc,init] Large Page Support: Disabled
[0.006s][info   ][gc,init] GC Workers: 2 (dynamic)
[0.006s][info   ][gc,init] Address Space Type: Contiguous/Unrestricted/Complete
[0.006s][info   ][gc,init] Address Space Size: 16384M x 3 = 49152M
[0.006s][info   ][gc,init] Min Capacity: 1024M
[0.006s][info   ][gc,init] Initial Capacity: 1024M
[0.006s][info   ][gc,init] Max Capacity: 1024M
[0.006s][info   ][gc,init] Medium Page Size: 32M
[0.006s][info   ][gc,init] Pre-touch: Disabled
[0.006s][info   ][gc,init] Uncommit: Implicitly Disabled (-Xms equals -Xmx)
[0.006s][info   ][gc,init] Runtime Workers: 5
[0.007s][info   ][gc     ] Using The Z Garbage Collector
[0.007s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.007s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.007s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.119s][info   ][gc,start    ] GC(0) Garbage Collection (Warmup)
[0.119s][info   ][gc,task     ] GC(0) Using 2 workers
[0.119s][info   ][gc,phases   ] GC(0) Pause Mark Start 0.007ms
[0.123s][info   ][gc,phases   ] GC(0) Concurrent Mark 3.747ms
[0.123s][info   ][gc,phases   ] GC(0) Pause Mark End 0.004ms
[0.123s][info   ][gc,phases   ] GC(0) Concurrent Mark Free 0.000ms
[0.124s][info   ][gc,phases   ] GC(0) Concurrent Process Non-Strong References 0.224ms
[0.124s][info   ][gc,phases   ] GC(0) Concurrent Reset Relocation Set 0.000ms
[0.128s][info   ][gc,phases   ] GC(0) Concurrent Select Relocation Set 3.728ms
[0.128s][info   ][gc,phases   ] GC(0) Pause Relocate Start 0.002ms
[0.137s][info   ][gc,phases   ] GC(0) Concurrent Relocate 9.685ms
[0.138s][info   ][gc,load     ] GC(0) Load: 3.49/3.80/4.40
[0.138s][info   ][gc,mmu      ] GC(0) MMU: 2ms/99.6%, 5ms/99.8%, 10ms/99.9%, 20ms/99.9%, 50ms/100.0%, 100ms/100.0%
[0.138s][info   ][gc,marking  ] GC(0) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.138s][info   ][gc,marking  ] GC(0) Mark Stack Usage: 32M
[0.138s][info   ][gc,nmethod  ] GC(0) NMethods: 118 registered, 0 unregistered
[0.138s][info   ][gc,metaspace] GC(0) Metaspace: 0M used, 0M committed, 1088M reserved
[0.138s][info   ][gc,ref      ] GC(0) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.138s][info   ][gc,ref      ] GC(0) Weak: 26 encountered, 19 discovered, 3 enqueued
[0.138s][info   ][gc,ref      ] GC(0) Final: 0 encountered, 0 discovered, 0 enqueued
[0.138s][info   ][gc,ref      ] GC(0) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.138s][info   ][gc,reloc    ] GC(0) Small Pages: 158 / 316M, Empty: 0M, Relocated: 57M, In-Place: 0
[0.138s][info   ][gc,reloc    ] GC(0) Medium Pages: 13 / 416M, Empty: 0M, Relocated: 133M, In-Place: 0
[0.138s][info   ][gc,reloc    ] GC(0) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.138s][info   ][gc,reloc    ] GC(0) Forwarding Usage: 0M
[0.138s][info   ][gc,heap     ] GC(0) Min Capacity: 1024M(100%)
[0.138s][info   ][gc,heap     ] GC(0) Max Capacity: 1024M(100%)
[0.138s][info   ][gc,heap     ] GC(0) Soft Max Capacity: 1024M(100%)
[0.138s][info   ][gc,heap     ] GC(0)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.138s][info   ][gc,heap     ] GC(0)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.138s][info   ][gc,heap     ] GC(0)      Free:      292M (29%)         248M (24%)         202M (20%)         690M (67%)         690M (67%)         168M (16%)
[0.138s][info   ][gc,heap     ] GC(0)      Used:      732M (71%)         776M (76%)         822M (80%)         334M (33%)         856M (84%)         334M (33%)
[0.138s][info   ][gc,heap     ] GC(0)      Live:         -               191M (19%)         191M (19%)         191M (19%)            -                  -
[0.138s][info   ][gc,heap     ] GC(0) Allocated:         -                44M (4%)           90M (9%)          111M (11%)            -                  -
[0.138s][info   ][gc,heap     ] GC(0)   Garbage:         -               540M (53%)         540M (53%)          30M (3%)             -                  -
[0.138s][info   ][gc,heap     ] GC(0) Reclaimed:         -                  -                 0M (0%)          509M (50%)            -                  -
[0.138s][info   ][gc          ] GC(0) Garbage Collection (Warmup) 732M(71%)->334M(33%)
[0.195s][info   ][gc,start    ] GC(1) Garbage Collection (Allocation Stall)
[0.195s][info   ][gc,ref      ] GC(1) Clearing All SoftReferences
[0.195s][info   ][gc,task     ] GC(1) Using 2 workers
[0.195s][info   ][gc,phases   ] GC(1) Pause Mark Start 0.003ms
[0.199s][info   ][gc,phases   ] GC(1) Concurrent Mark 4.635ms
[0.200s][info   ][gc,phases   ] GC(1) Pause Mark End 0.006ms
[0.200s][info   ][gc,phases   ] GC(1) Concurrent Mark Free 0.000ms
[0.200s][info   ][gc,phases   ] GC(1) Concurrent Process Non-Strong References 0.235ms
[0.200s][info   ][gc,phases   ] GC(1) Concurrent Reset Relocation Set 0.005ms
[0.202s][info   ][gc          ] Allocation Stall (main) 7.946ms
[0.203s][info   ][gc,phases   ] GC(1) Concurrent Select Relocation Set 2.682ms
[0.203s][info   ][gc,phases   ] GC(1) Pause Relocate Start 0.015ms
[0.204s][info   ][gc          ] Allocation Stall (main) 1.001ms
[0.204s][info   ][gc          ] Allocation Stall (main) 0.366ms
[0.221s][info   ][gc,phases   ] GC(1) Concurrent Relocate 18.012ms
[0.221s][info   ][gc,load     ] GC(1) Load: 3.49/3.80/4.40
[0.221s][info   ][gc,mmu      ] GC(1) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/100.0%, 100ms/100.0%
[0.221s][info   ][gc,marking  ] GC(1) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.221s][info   ][gc,marking  ] GC(1) Mark Stack Usage: 32M
[0.221s][info   ][gc,nmethod  ] GC(1) NMethods: 120 registered, 0 unregistered
[0.221s][info   ][gc,metaspace] GC(1) Metaspace: 0M used, 0M committed, 1088M reserved
[0.221s][info   ][gc,ref      ] GC(1) Soft: 5 encountered, 2 discovered, 2 enqueued
[0.221s][info   ][gc,ref      ] GC(1) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.221s][info   ][gc,ref      ] GC(1) Final: 0 encountered, 0 discovered, 0 enqueued
[0.221s][info   ][gc,ref      ] GC(1) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.221s][info   ][gc,reloc    ] GC(1) Small Pages: 208 / 416M, Empty: 2M, Relocated: 88M, In-Place: 0
[0.221s][info   ][gc,reloc    ] GC(1) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 191M, In-Place: 1
[0.221s][info   ][gc,reloc    ] GC(1) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.221s][info   ][gc,reloc    ] GC(1) Forwarding Usage: 0M
[0.221s][info   ][gc,heap     ] GC(1) Min Capacity: 1024M(100%)
[0.221s][info   ][gc,heap     ] GC(1) Max Capacity: 1024M(100%)
[0.221s][info   ][gc,heap     ] GC(1) Soft Max Capacity: 1024M(100%)
[0.221s][info   ][gc,heap     ] GC(1)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.221s][info   ][gc,heap     ] GC(1)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.221s][info   ][gc,heap     ] GC(1)      Free:        0M (0%)            0M (0%)            0M (0%)          426M (42%)         426M (42%)           0M (0%)
[0.221s][info   ][gc,heap     ] GC(1)      Used:     1024M (100%)       1024M (100%)       1024M (100%)        598M (58%)        1024M (100%)        598M (58%)
[0.221s][info   ][gc,heap     ] GC(1)      Live:         -               282M (28%)         282M (28%)         282M (28%)            -                  -
[0.221s][info   ][gc,heap     ] GC(1) Allocated:         -                 0M (0%)            2M (0%)          277M (27%)            -                  -
[0.221s][info   ][gc,heap     ] GC(1)   Garbage:         -               741M (72%)         739M (72%)          37M (4%)             -                  -
[0.221s][info   ][gc,heap     ] GC(1) Reclaimed:         -                  -                 2M (0%)          703M (69%)            -                  -
[0.221s][info   ][gc          ] GC(1) Garbage Collection (Allocation Stall) 1024M(100%)->598M(58%)
[0.254s][info   ][gc,start    ] GC(2) Garbage Collection (Allocation Stall)
[0.254s][info   ][gc,ref      ] GC(2) Clearing All SoftReferences
[0.254s][info   ][gc,task     ] GC(2) Using 2 workers
[0.254s][info   ][gc,phases   ] GC(2) Pause Mark Start 0.004ms
[0.258s][info   ][gc,phases   ] GC(2) Concurrent Mark 3.659ms
[0.258s][info   ][gc,phases   ] GC(2) Pause Mark End 0.006ms
[0.258s][info   ][gc,phases   ] GC(2) Concurrent Mark Free 0.000ms
[0.258s][info   ][gc,phases   ] GC(2) Concurrent Process Non-Strong References 0.229ms
[0.258s][info   ][gc,phases   ] GC(2) Concurrent Reset Relocation Set 0.007ms
[0.260s][info   ][gc,phases   ] GC(2) Concurrent Select Relocation Set 1.598ms
[0.260s][info   ][gc,phases   ] GC(2) Pause Relocate Start 0.002ms
[0.262s][info   ][gc          ] Allocation Stall (main) 7.645ms
[0.267s][info   ][gc,phases   ] GC(2) Concurrent Relocate 6.951ms
[0.267s][info   ][gc,load     ] GC(2) Load: 3.49/3.80/4.40
[0.267s][info   ][gc,mmu      ] GC(2) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/100.0%, 100ms/100.0%
[0.267s][info   ][gc,marking  ] GC(2) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.267s][info   ][gc,marking  ] GC(2) Mark Stack Usage: 32M
[0.267s][info   ][gc,nmethod  ] GC(2) NMethods: 120 registered, 0 unregistered
[0.267s][info   ][gc,metaspace] GC(2) Metaspace: 0M used, 0M committed, 1088M reserved
[0.267s][info   ][gc,ref      ] GC(2) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.267s][info   ][gc,ref      ] GC(2) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.267s][info   ][gc,ref      ] GC(2) Final: 0 encountered, 0 discovered, 0 enqueued
[0.267s][info   ][gc,ref      ] GC(2) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.267s][info   ][gc,reloc    ] GC(2) Small Pages: 200 / 400M, Empty: 2M, Relocated: 95M, In-Place: 0
[0.267s][info   ][gc,reloc    ] GC(2) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 214M, In-Place: 1
[0.267s][info   ][gc,reloc    ] GC(2) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.267s][info   ][gc,reloc    ] GC(2) Forwarding Usage: 0M
[0.267s][info   ][gc,heap     ] GC(2) Min Capacity: 1024M(100%)
[0.267s][info   ][gc,heap     ] GC(2) Max Capacity: 1024M(100%)
[0.267s][info   ][gc,heap     ] GC(2) Soft Max Capacity: 1024M(100%)
[0.267s][info   ][gc,heap     ] GC(2)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.267s][info   ][gc,heap     ] GC(2)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.267s][info   ][gc,heap     ] GC(2)      Free:       16M (2%)           16M (2%)           18M (2%)          596M (58%)         596M (58%)          16M (2%)
[0.267s][info   ][gc,heap     ] GC(2)      Used:     1008M (98%)        1008M (98%)        1006M (98%)         428M (42%)        1008M (98%)         428M (42%)
[0.267s][info   ][gc,heap     ] GC(2)      Live:         -               314M (31%)         314M (31%)         314M (31%)            -                  -
[0.267s][info   ][gc,heap     ] GC(2) Allocated:         -                 0M (0%)            0M (0%)           97M (10%)            -                  -
[0.267s][info   ][gc,heap     ] GC(2)   Garbage:         -               693M (68%)         691M (68%)          15M (2%)             -                  -
[0.267s][info   ][gc,heap     ] GC(2) Reclaimed:         -                  -                 2M (0%)          677M (66%)            -                  -
[0.267s][info   ][gc          ] GC(2) Garbage Collection (Allocation Stall) 1008M(98%)->428M(42%)
[0.305s][info   ][gc,start    ] GC(3) Garbage Collection (Allocation Stall)
[0.305s][info   ][gc,ref      ] GC(3) Clearing All SoftReferences
[0.305s][info   ][gc,task     ] GC(3) Using 2 workers
[0.305s][info   ][gc,phases   ] GC(3) Pause Mark Start 0.004ms
[0.308s][info   ][gc,phases   ] GC(3) Concurrent Mark 3.224ms
[0.309s][info   ][gc,phases   ] GC(3) Pause Mark End 0.005ms
[0.309s][info   ][gc,phases   ] GC(3) Concurrent Mark Free 0.000ms
[0.309s][info   ][gc,phases   ] GC(3) Concurrent Process Non-Strong References 0.223ms
[0.309s][info   ][gc,phases   ] GC(3) Concurrent Reset Relocation Set 0.006ms
[0.310s][info   ][gc,phases   ] GC(3) Concurrent Select Relocation Set 1.487ms
[0.310s][info   ][gc,phases   ] GC(3) Pause Relocate Start 0.002ms
[0.311s][info   ][gc          ] Allocation Stall (main) 5.924ms
[0.318s][info   ][gc,phases   ] GC(3) Concurrent Relocate 7.081ms
[0.318s][info   ][gc,load     ] GC(3) Load: 3.49/3.80/4.40
[0.318s][info   ][gc,mmu      ] GC(3) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/100.0%, 100ms/100.0%
[0.318s][info   ][gc,marking  ] GC(3) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.318s][info   ][gc,marking  ] GC(3) Mark Stack Usage: 32M
[0.318s][info   ][gc,nmethod  ] GC(3) NMethods: 120 registered, 0 unregistered
[0.318s][info   ][gc,metaspace] GC(3) Metaspace: 0M used, 0M committed, 1088M reserved
[0.318s][info   ][gc,ref      ] GC(3) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.318s][info   ][gc,ref      ] GC(3) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.318s][info   ][gc,ref      ] GC(3) Final: 0 encountered, 0 discovered, 0 enqueued
[0.318s][info   ][gc,ref      ] GC(3) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.318s][info   ][gc,reloc    ] GC(3) Small Pages: 202 / 404M, Empty: 0M, Relocated: 92M, In-Place: 0
[0.318s][info   ][gc,reloc    ] GC(3) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 215M, In-Place: 1
[0.318s][info   ][gc,reloc    ] GC(3) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.318s][info   ][gc,reloc    ] GC(3) Forwarding Usage: 0M
[0.318s][info   ][gc,heap     ] GC(3) Min Capacity: 1024M(100%)
[0.318s][info   ][gc,heap     ] GC(3) Max Capacity: 1024M(100%)
[0.318s][info   ][gc,heap     ] GC(3) Soft Max Capacity: 1024M(100%)
[0.318s][info   ][gc,heap     ] GC(3)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.318s][info   ][gc,heap     ] GC(3)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.318s][info   ][gc,heap     ] GC(3)      Free:       12M (1%)           12M (1%)           12M (1%)          550M (54%)         550M (54%)          10M (1%)
[0.318s][info   ][gc,heap     ] GC(3)      Used:     1012M (99%)        1012M (99%)        1012M (99%)         474M (46%)        1014M (99%)         474M (46%)
[0.318s][info   ][gc,heap     ] GC(3)      Live:         -               338M (33%)         338M (33%)         338M (33%)            -                  -
[0.318s][info   ][gc,heap     ] GC(3) Allocated:         -                 0M (0%)            0M (0%)          103M (10%)            -                  -
[0.318s][info   ][gc,heap     ] GC(3)   Garbage:         -               673M (66%)         673M (66%)          31M (3%)             -                  -
[0.318s][info   ][gc,heap     ] GC(3) Reclaimed:         -                  -                 0M (0%)          641M (63%)            -                  -
[0.318s][info   ][gc          ] GC(3) Garbage Collection (Allocation Stall) 1012M(99%)->474M(46%)
[0.319s][info   ][gc,start    ] GC(4) Garbage Collection (Warmup)
[0.319s][info   ][gc,task     ] GC(4) Using 2 workers
[0.319s][info   ][gc,phases   ] GC(4) Pause Mark Start 0.003ms
[0.324s][info   ][gc,phases   ] GC(4) Concurrent Mark 4.527ms
[0.324s][info   ][gc,phases   ] GC(4) Pause Mark End 0.009ms
[0.324s][info   ][gc,phases   ] GC(4) Concurrent Mark Free 0.000ms
[0.324s][info   ][gc,phases   ] GC(4) Concurrent Process Non-Strong References 0.270ms
[0.324s][info   ][gc,phases   ] GC(4) Concurrent Reset Relocation Set 0.006ms
[0.326s][info   ][gc,phases   ] GC(4) Concurrent Select Relocation Set 1.490ms
[0.326s][info   ][gc,phases   ] GC(4) Pause Relocate Start 0.003ms
[0.328s][info   ][gc,phases   ] GC(4) Concurrent Relocate 1.424ms
[0.328s][info   ][gc,load     ] GC(4) Load: 3.49/3.80/4.40
[0.328s][info   ][gc,mmu      ] GC(4) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.328s][info   ][gc,marking  ] GC(4) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.328s][info   ][gc,marking  ] GC(4) Mark Stack Usage: 32M
[0.328s][info   ][gc,nmethod  ] GC(4) NMethods: 120 registered, 0 unregistered
[0.328s][info   ][gc,metaspace] GC(4) Metaspace: 0M used, 0M committed, 1088M reserved
[0.328s][info   ][gc,ref      ] GC(4) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.328s][info   ][gc,ref      ] GC(4) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.328s][info   ][gc,ref      ] GC(4) Final: 0 encountered, 0 discovered, 0 enqueued
[0.328s][info   ][gc,ref      ] GC(4) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.328s][info   ][gc,reloc    ] GC(4) Small Pages: 85 / 170M, Empty: 0M, Relocated: 20M, In-Place: 0
[0.328s][info   ][gc,reloc    ] GC(4) Medium Pages: 11 / 352M, Empty: 0M, Relocated: 48M, In-Place: 0
[0.328s][info   ][gc,reloc    ] GC(4) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.328s][info   ][gc,reloc    ] GC(4) Forwarding Usage: 0M
[0.328s][info   ][gc,heap     ] GC(4) Min Capacity: 1024M(100%)
[0.328s][info   ][gc,heap     ] GC(4) Max Capacity: 1024M(100%)
[0.328s][info   ][gc,heap     ] GC(4) Soft Max Capacity: 1024M(100%)
[0.328s][info   ][gc,heap     ] GC(4)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.328s][info   ][gc,heap     ] GC(4)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.328s][info   ][gc,heap     ] GC(4)      Free:      502M (49%)         456M (45%)         446M (44%)         514M (50%)         540M (53%)         414M (40%)
[0.328s][info   ][gc,heap     ] GC(4)      Used:      522M (51%)         568M (55%)         578M (56%)         510M (50%)         610M (60%)         484M (47%)
[0.328s][info   ][gc,heap     ] GC(4)      Live:         -               329M (32%)         329M (32%)         329M (32%)            -                  -
[0.328s][info   ][gc,heap     ] GC(4) Allocated:         -                46M (4%)           56M (5%)           97M (10%)            -                  -
[0.328s][info   ][gc,heap     ] GC(4)   Garbage:         -               192M (19%)         192M (19%)          82M (8%)             -                  -
[0.328s][info   ][gc,heap     ] GC(4) Reclaimed:         -                  -                 0M (0%)          109M (11%)            -                  -
[0.328s][info   ][gc          ] GC(4) Garbage Collection (Warmup) 522M(51%)->510M(50%)
[0.366s][info   ][gc,start    ] GC(5) Garbage Collection (Allocation Stall)
[0.366s][info   ][gc,ref      ] GC(5) Clearing All SoftReferences
[0.366s][info   ][gc,task     ] GC(5) Using 2 workers
[0.366s][info   ][gc,phases   ] GC(5) Pause Mark Start 0.011ms
[0.369s][info   ][gc,phases   ] GC(5) Concurrent Mark 2.923ms
[0.369s][info   ][gc,phases   ] GC(5) Pause Mark End 0.006ms
[0.369s][info   ][gc,phases   ] GC(5) Concurrent Mark Free 0.000ms
[0.369s][info   ][gc,phases   ] GC(5) Concurrent Process Non-Strong References 0.372ms
[0.370s][info   ][gc,phases   ] GC(5) Concurrent Reset Relocation Set 0.002ms
[0.372s][info   ][gc,phases   ] GC(5) Concurrent Select Relocation Set 2.464ms
[0.372s][info   ][gc,phases   ] GC(5) Pause Relocate Start 0.006ms
[0.373s][info   ][gc          ] Allocation Stall (main) 7.172ms
[0.380s][info   ][gc,phases   ] GC(5) Concurrent Relocate 7.684ms
[0.380s][info   ][gc,load     ] GC(5) Load: 3.49/3.80/4.40
[0.380s][info   ][gc,mmu      ] GC(5) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.380s][info   ][gc,marking  ] GC(5) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.380s][info   ][gc,marking  ] GC(5) Mark Stack Usage: 32M
[0.380s][info   ][gc,nmethod  ] GC(5) NMethods: 120 registered, 0 unregistered
[0.380s][info   ][gc,metaspace] GC(5) Metaspace: 0M used, 0M committed, 1088M reserved
[0.380s][info   ][gc,ref      ] GC(5) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.380s][info   ][gc,ref      ] GC(5) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.380s][info   ][gc,ref      ] GC(5) Final: 0 encountered, 0 discovered, 0 enqueued
[0.380s][info   ][gc,ref      ] GC(5) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.380s][info   ][gc,reloc    ] GC(5) Small Pages: 199 / 398M, Empty: 2M, Relocated: 104M, In-Place: 0
[0.380s][info   ][gc,reloc    ] GC(5) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 220M, In-Place: 1
[0.380s][info   ][gc,reloc    ] GC(5) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.380s][info   ][gc,reloc    ] GC(5) Forwarding Usage: 0M
[0.380s][info   ][gc,heap     ] GC(5) Min Capacity: 1024M(100%)
[0.380s][info   ][gc,heap     ] GC(5) Max Capacity: 1024M(100%)
[0.380s][info   ][gc,heap     ] GC(5) Soft Max Capacity: 1024M(100%)
[0.380s][info   ][gc,heap     ] GC(5)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.380s][info   ][gc,heap     ] GC(5)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.380s][info   ][gc,heap     ] GC(5)      Free:       18M (2%)           18M (2%)           20M (2%)          510M (50%)         542M (53%)          18M (2%)
[0.380s][info   ][gc,heap     ] GC(5)      Used:     1006M (98%)        1006M (98%)        1004M (98%)         514M (50%)        1006M (98%)         482M (47%)
[0.380s][info   ][gc,heap     ] GC(5)      Live:         -               344M (34%)         344M (34%)         344M (34%)            -                  -
[0.380s][info   ][gc,heap     ] GC(5) Allocated:         -                 0M (0%)            0M (0%)          143M (14%)            -                  -
[0.380s][info   ][gc,heap     ] GC(5)   Garbage:         -               661M (65%)         659M (64%)          25M (2%)             -                  -
[0.380s][info   ][gc,heap     ] GC(5) Reclaimed:         -                  -                 2M (0%)          635M (62%)            -                  -
[0.380s][info   ][gc          ] GC(5) Garbage Collection (Allocation Stall) 1006M(98%)->514M(50%)
[0.411s][info   ][gc,start    ] GC(6) Garbage Collection (Allocation Stall)
[0.411s][info   ][gc,ref      ] GC(6) Clearing All SoftReferences
[0.411s][info   ][gc,task     ] GC(6) Using 2 workers
[0.411s][info   ][gc,phases   ] GC(6) Pause Mark Start 0.004ms
[0.415s][info   ][gc,phases   ] GC(6) Concurrent Mark 3.951ms
[0.415s][info   ][gc,phases   ] GC(6) Pause Mark End 0.008ms
[0.415s][info   ][gc,phases   ] GC(6) Concurrent Mark Free 0.000ms
[0.416s][info   ][gc,phases   ] GC(6) Concurrent Process Non-Strong References 0.294ms
[0.416s][info   ][gc,phases   ] GC(6) Concurrent Reset Relocation Set 0.008ms
[0.417s][info   ][gc,phases   ] GC(6) Concurrent Select Relocation Set 1.586ms
[0.418s][info   ][gc,phases   ] GC(6) Pause Relocate Start 0.005ms
[0.418s][info   ][gc          ] Allocation Stall (main) 6.804ms
[0.426s][info   ][gc,phases   ] GC(6) Concurrent Relocate 8.405ms
[0.426s][info   ][gc,load     ] GC(6) Load: 3.49/3.80/4.40
[0.426s][info   ][gc,mmu      ] GC(6) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.426s][info   ][gc,marking  ] GC(6) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.426s][info   ][gc,marking  ] GC(6) Mark Stack Usage: 32M
[0.426s][info   ][gc,nmethod  ] GC(6) NMethods: 120 registered, 0 unregistered
[0.426s][info   ][gc,metaspace] GC(6) Metaspace: 0M used, 0M committed, 1088M reserved
[0.426s][info   ][gc,ref      ] GC(6) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.426s][info   ][gc,ref      ] GC(6) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.426s][info   ][gc,ref      ] GC(6) Final: 0 encountered, 0 discovered, 0 enqueued
[0.426s][info   ][gc,ref      ] GC(6) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.426s][info   ][gc,reloc    ] GC(6) Small Pages: 195 / 390M, Empty: 2M, Relocated: 103M, In-Place: 0
[0.426s][info   ][gc,reloc    ] GC(6) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 190M, In-Place: 1
[0.426s][info   ][gc,reloc    ] GC(6) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.426s][info   ][gc,reloc    ] GC(6) Forwarding Usage: 0M
[0.426s][info   ][gc,heap     ] GC(6) Min Capacity: 1024M(100%)
[0.426s][info   ][gc,heap     ] GC(6) Max Capacity: 1024M(100%)
[0.426s][info   ][gc,heap     ] GC(6) Soft Max Capacity: 1024M(100%)
[0.426s][info   ][gc,heap     ] GC(6)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.426s][info   ][gc,heap     ] GC(6)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.426s][info   ][gc,heap     ] GC(6)      Free:       26M (3%)           26M (3%)           28M (3%)          502M (49%)         512M (50%)          26M (3%)
[0.426s][info   ][gc,heap     ] GC(6)      Used:      998M (97%)         998M (97%)         996M (97%)         522M (51%)         998M (97%)         512M (50%)
[0.426s][info   ][gc,heap     ] GC(6)      Live:         -               332M (32%)         332M (32%)         332M (32%)            -                  -
[0.426s][info   ][gc,heap     ] GC(6) Allocated:         -                 0M (0%)            0M (0%)          151M (15%)            -                  -
[0.426s][info   ][gc,heap     ] GC(6)   Garbage:         -               665M (65%)         663M (65%)          37M (4%)             -                  -
[0.426s][info   ][gc,heap     ] GC(6) Reclaimed:         -                  -                 2M (0%)          627M (61%)            -                  -
[0.426s][info   ][gc          ] GC(6) Garbage Collection (Allocation Stall) 998M(97%)->522M(51%)
[0.460s][info   ][gc,start    ] GC(7) Garbage Collection (Allocation Stall)
[0.460s][info   ][gc,ref      ] GC(7) Clearing All SoftReferences
[0.460s][info   ][gc,task     ] GC(7) Using 2 workers
[0.460s][info   ][gc,phases   ] GC(7) Pause Mark Start 0.004ms
[0.464s][info   ][gc,phases   ] GC(7) Concurrent Mark 4.091ms
[0.464s][info   ][gc,phases   ] GC(7) Pause Mark End 0.005ms
[0.464s][info   ][gc,phases   ] GC(7) Concurrent Mark Free 0.000ms
[0.465s][info   ][gc,phases   ] GC(7) Concurrent Process Non-Strong References 0.269ms
[0.465s][info   ][gc,phases   ] GC(7) Concurrent Reset Relocation Set 0.006ms
[0.466s][info   ][gc          ] Allocation Stall (main) 6.220ms
[0.466s][info   ][gc,phases   ] GC(7) Concurrent Select Relocation Set 1.452ms
[0.466s][info   ][gc,phases   ] GC(7) Pause Relocate Start 0.002ms
[0.467s][info   ][gc          ] Allocation Stall (main) 0.467ms
[0.475s][info   ][gc,phases   ] GC(7) Concurrent Relocate 7.966ms
[0.475s][info   ][gc,load     ] GC(7) Load: 3.49/3.80/4.40
[0.475s][info   ][gc,mmu      ] GC(7) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.475s][info   ][gc,marking  ] GC(7) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.475s][info   ][gc,marking  ] GC(7) Mark Stack Usage: 32M
[0.475s][info   ][gc,nmethod  ] GC(7) NMethods: 121 registered, 0 unregistered
[0.475s][info   ][gc,metaspace] GC(7) Metaspace: 0M used, 0M committed, 1088M reserved
[0.475s][info   ][gc,ref      ] GC(7) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.475s][info   ][gc,ref      ] GC(7) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.475s][info   ][gc,ref      ] GC(7) Final: 0 encountered, 0 discovered, 0 enqueued
[0.475s][info   ][gc,ref      ] GC(7) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.475s][info   ][gc,reloc    ] GC(7) Small Pages: 208 / 416M, Empty: 2M, Relocated: 103M, In-Place: 0
[0.475s][info   ][gc,reloc    ] GC(7) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 216M, In-Place: 1
[0.475s][info   ][gc,reloc    ] GC(7) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.475s][info   ][gc,reloc    ] GC(7) Forwarding Usage: 0M
[0.475s][info   ][gc,heap     ] GC(7) Min Capacity: 1024M(100%)
[0.475s][info   ][gc,heap     ] GC(7) Max Capacity: 1024M(100%)
[0.475s][info   ][gc,heap     ] GC(7) Soft Max Capacity: 1024M(100%)
[0.475s][info   ][gc,heap     ] GC(7)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.475s][info   ][gc,heap     ] GC(7)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.475s][info   ][gc,heap     ] GC(7)      Free:        0M (0%)            0M (0%)            0M (0%)          588M (57%)         588M (57%)           0M (0%)
[0.475s][info   ][gc,heap     ] GC(7)      Used:     1024M (100%)       1024M (100%)       1024M (100%)        436M (43%)        1024M (100%)        436M (43%)
[0.475s][info   ][gc,heap     ] GC(7)      Live:         -               343M (34%)         343M (34%)         343M (34%)            -                  -
[0.475s][info   ][gc,heap     ] GC(7) Allocated:         -                 0M (0%)            2M (0%)           63M (6%)             -                  -
[0.475s][info   ][gc,heap     ] GC(7)   Garbage:         -               680M (66%)         678M (66%)          28M (3%)             -                  -
[0.475s][info   ][gc,heap     ] GC(7) Reclaimed:         -                  -                 2M (0%)          651M (64%)            -                  -
[0.475s][info   ][gc          ] GC(7) Garbage Collection (Allocation Stall) 1024M(100%)->436M(43%)
[0.512s][info   ][gc,start    ] GC(8) Garbage Collection (Allocation Stall)
[0.512s][info   ][gc,ref      ] GC(8) Clearing All SoftReferences
[0.512s][info   ][gc,task     ] GC(8) Using 2 workers
[0.512s][info   ][gc,phases   ] GC(8) Pause Mark Start 0.006ms
[0.515s][info   ][gc,phases   ] GC(8) Concurrent Mark 3.516ms
[0.515s][info   ][gc,phases   ] GC(8) Pause Mark End 0.005ms
[0.515s][info   ][gc,phases   ] GC(8) Concurrent Mark Free 0.000ms
[0.516s][info   ][gc,phases   ] GC(8) Concurrent Process Non-Strong References 0.217ms
[0.516s][info   ][gc,phases   ] GC(8) Concurrent Reset Relocation Set 0.008ms
[0.517s][info   ][gc,phases   ] GC(8) Concurrent Select Relocation Set 1.398ms
[0.517s][info   ][gc,phases   ] GC(8) Pause Relocate Start 0.005ms
[0.519s][info   ][gc          ] Allocation Stall (main) 7.862ms
[0.525s][info   ][gc,phases   ] GC(8) Concurrent Relocate 7.640ms
[0.525s][info   ][gc,load     ] GC(8) Load: 3.49/3.80/4.40
[0.525s][info   ][gc,mmu      ] GC(8) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.525s][info   ][gc,marking  ] GC(8) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.525s][info   ][gc,marking  ] GC(8) Mark Stack Usage: 32M
[0.525s][info   ][gc,nmethod  ] GC(8) NMethods: 121 registered, 0 unregistered
[0.525s][info   ][gc,metaspace] GC(8) Metaspace: 0M used, 0M committed, 1088M reserved
[0.525s][info   ][gc,ref      ] GC(8) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.525s][info   ][gc,ref      ] GC(8) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.525s][info   ][gc,ref      ] GC(8) Final: 0 encountered, 0 discovered, 0 enqueued
[0.525s][info   ][gc,ref      ] GC(8) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.525s][info   ][gc,reloc    ] GC(8) Small Pages: 207 / 414M, Empty: 0M, Relocated: 96M, In-Place: 0
[0.525s][info   ][gc,reloc    ] GC(8) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 245M, In-Place: 1
[0.525s][info   ][gc,reloc    ] GC(8) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.525s][info   ][gc,reloc    ] GC(8) Forwarding Usage: 0M
[0.525s][info   ][gc,heap     ] GC(8) Min Capacity: 1024M(100%)
[0.525s][info   ][gc,heap     ] GC(8) Max Capacity: 1024M(100%)
[0.525s][info   ][gc,heap     ] GC(8) Soft Max Capacity: 1024M(100%)
[0.525s][info   ][gc,heap     ] GC(8)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.525s][info   ][gc,heap     ] GC(8)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.525s][info   ][gc,heap     ] GC(8)      Free:        2M (0%)            2M (0%)            2M (0%)          548M (54%)         550M (54%)           2M (0%)
[0.525s][info   ][gc,heap     ] GC(8)      Used:     1022M (100%)       1022M (100%)       1022M (100%)        476M (46%)        1022M (100%)        474M (46%)
[0.525s][info   ][gc,heap     ] GC(8)      Live:         -               352M (34%)         352M (34%)         352M (34%)            -                  -
[0.525s][info   ][gc,heap     ] GC(8) Allocated:         -                 0M (0%)            0M (0%)          101M (10%)            -                  -
[0.525s][info   ][gc,heap     ] GC(8)   Garbage:         -               669M (65%)         669M (65%)          21M (2%)             -                  -
[0.525s][info   ][gc,heap     ] GC(8) Reclaimed:         -                  -                 0M (0%)          647M (63%)            -                  -
[0.525s][info   ][gc          ] GC(8) Garbage Collection (Allocation Stall) 1022M(100%)->476M(46%)
[0.558s][info   ][gc,start    ] GC(9) Garbage Collection (Allocation Stall)
[0.559s][info   ][gc,ref      ] GC(9) Clearing All SoftReferences
[0.559s][info   ][gc,task     ] GC(9) Using 2 workers
[0.559s][info   ][gc,phases   ] GC(9) Pause Mark Start 0.003ms
[0.562s][info   ][gc,phases   ] GC(9) Concurrent Mark 3.252ms
[0.562s][info   ][gc,phases   ] GC(9) Pause Mark End 0.006ms
[0.562s][info   ][gc,phases   ] GC(9) Concurrent Mark Free 0.000ms
[0.562s][info   ][gc,phases   ] GC(9) Concurrent Process Non-Strong References 0.204ms
[0.562s][info   ][gc,phases   ] GC(9) Concurrent Reset Relocation Set 0.008ms
[0.564s][info   ][gc,phases   ] GC(9) Concurrent Select Relocation Set 1.407ms
[0.564s][info   ][gc,phases   ] GC(9) Pause Relocate Start 0.002ms
[0.564s][info   ][gc          ] Allocation Stall (main) 5.988ms
[0.573s][info   ][gc,phases   ] GC(9) Concurrent Relocate 9.026ms
[0.573s][info   ][gc,load     ] GC(9) Load: 3.49/3.80/4.40
[0.573s][info   ][gc,mmu      ] GC(9) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.573s][info   ][gc,marking  ] GC(9) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.573s][info   ][gc,marking  ] GC(9) Mark Stack Usage: 32M
[0.573s][info   ][gc,nmethod  ] GC(9) NMethods: 121 registered, 0 unregistered
[0.573s][info   ][gc,metaspace] GC(9) Metaspace: 0M used, 0M committed, 1088M reserved
[0.573s][info   ][gc,ref      ] GC(9) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.573s][info   ][gc,ref      ] GC(9) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.573s][info   ][gc,ref      ] GC(9) Final: 0 encountered, 0 discovered, 0 enqueued
[0.573s][info   ][gc,ref      ] GC(9) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.573s][info   ][gc,reloc    ] GC(9) Small Pages: 198 / 396M, Empty: 4M, Relocated: 103M, In-Place: 0
[0.573s][info   ][gc,reloc    ] GC(9) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 250M, In-Place: 1
[0.573s][info   ][gc,reloc    ] GC(9) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.573s][info   ][gc,reloc    ] GC(9) Forwarding Usage: 0M
[0.573s][info   ][gc,heap     ] GC(9) Min Capacity: 1024M(100%)
[0.573s][info   ][gc,heap     ] GC(9) Max Capacity: 1024M(100%)
[0.573s][info   ][gc,heap     ] GC(9) Soft Max Capacity: 1024M(100%)
[0.573s][info   ][gc,heap     ] GC(9)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.573s][info   ][gc,heap     ] GC(9)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.573s][info   ][gc,heap     ] GC(9)      Free:       20M (2%)           20M (2%)           24M (2%)          540M (53%)         540M (53%)          20M (2%)
[0.573s][info   ][gc,heap     ] GC(9)      Used:     1004M (98%)        1004M (98%)        1000M (98%)         484M (47%)        1004M (98%)         484M (47%)
[0.573s][info   ][gc,heap     ] GC(9)      Live:         -               357M (35%)         357M (35%)         357M (35%)            -                  -
[0.573s][info   ][gc,heap     ] GC(9) Allocated:         -                 0M (0%)            0M (0%)          115M (11%)            -                  -
[0.573s][info   ][gc,heap     ] GC(9)   Garbage:         -               646M (63%)         642M (63%)          10M (1%)             -                  -
[0.573s][info   ][gc,heap     ] GC(9) Reclaimed:         -                  -                 4M (0%)          635M (62%)            -                  -
[0.573s][info   ][gc          ] GC(9) Garbage Collection (Allocation Stall) 1004M(98%)->484M(47%)
[0.606s][info   ][gc,start    ] GC(10) Garbage Collection (Allocation Stall)
[0.606s][info   ][gc,ref      ] GC(10) Clearing All SoftReferences
[0.606s][info   ][gc,task     ] GC(10) Using 2 workers
[0.606s][info   ][gc,phases   ] GC(10) Pause Mark Start 0.004ms
[0.609s][info   ][gc,phases   ] GC(10) Concurrent Mark 3.487ms
[0.610s][info   ][gc,phases   ] GC(10) Pause Mark End 0.004ms
[0.610s][info   ][gc,phases   ] GC(10) Concurrent Mark Free 0.000ms
[0.610s][info   ][gc,phases   ] GC(10) Concurrent Process Non-Strong References 0.206ms
[0.610s][info   ][gc,phases   ] GC(10) Concurrent Reset Relocation Set 0.007ms
[0.611s][info   ][gc,phases   ] GC(10) Concurrent Select Relocation Set 1.478ms
[0.611s][info   ][gc,phases   ] GC(10) Pause Relocate Start 0.004ms
[0.613s][info   ][gc          ] Allocation Stall (main) 6.895ms
[0.619s][info   ][gc,phases   ] GC(10) Concurrent Relocate 7.578ms
[0.619s][info   ][gc,load     ] GC(10) Load: 3.49/3.80/4.40
[0.619s][info   ][gc,mmu      ] GC(10) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.619s][info   ][gc,marking  ] GC(10) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.619s][info   ][gc,marking  ] GC(10) Mark Stack Usage: 32M
[0.619s][info   ][gc,nmethod  ] GC(10) NMethods: 121 registered, 0 unregistered
[0.619s][info   ][gc,metaspace] GC(10) Metaspace: 0M used, 0M committed, 1088M reserved
[0.619s][info   ][gc,ref      ] GC(10) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.619s][info   ][gc,ref      ] GC(10) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.619s][info   ][gc,ref      ] GC(10) Final: 0 encountered, 0 discovered, 0 enqueued
[0.619s][info   ][gc,ref      ] GC(10) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.619s][info   ][gc,reloc    ] GC(10) Small Pages: 205 / 410M, Empty: 6M, Relocated: 108M, In-Place: 0
[0.619s][info   ][gc,reloc    ] GC(10) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 215M, In-Place: 1
[0.619s][info   ][gc,reloc    ] GC(10) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.619s][info   ][gc,reloc    ] GC(10) Forwarding Usage: 0M
[0.619s][info   ][gc,heap     ] GC(10) Min Capacity: 1024M(100%)
[0.619s][info   ][gc,heap     ] GC(10) Max Capacity: 1024M(100%)
[0.619s][info   ][gc,heap     ] GC(10) Soft Max Capacity: 1024M(100%)
[0.619s][info   ][gc,heap     ] GC(10)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.619s][info   ][gc,heap     ] GC(10)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.619s][info   ][gc,heap     ] GC(10)      Free:        6M (1%)            6M (1%)           12M (1%)          548M (54%)         548M (54%)           6M (1%)
[0.619s][info   ][gc,heap     ] GC(10)      Used:     1018M (99%)        1018M (99%)        1012M (99%)         476M (46%)        1018M (99%)         476M (46%)
[0.619s][info   ][gc,heap     ] GC(10)      Live:         -               341M (33%)         341M (33%)         341M (33%)            -                  -
[0.619s][info   ][gc,heap     ] GC(10) Allocated:         -                 0M (0%)            0M (0%)          103M (10%)            -                  -
[0.619s][info   ][gc,heap     ] GC(10)   Garbage:         -               676M (66%)         670M (65%)          30M (3%)             -                  -
[0.619s][info   ][gc,heap     ] GC(10) Reclaimed:         -                  -                 6M (1%)          645M (63%)            -                  -
[0.619s][info   ][gc          ] GC(10) Garbage Collection (Allocation Stall) 1018M(99%)->476M(46%)
[0.620s][info   ][gc,start    ] GC(11) Garbage Collection (Warmup)
[0.620s][info   ][gc,task     ] GC(11) Using 2 workers
[0.620s][info   ][gc,phases   ] GC(11) Pause Mark Start 0.003ms
[0.624s][info   ][gc,phases   ] GC(11) Concurrent Mark 4.084ms
[0.624s][info   ][gc,phases   ] GC(11) Pause Mark End 0.005ms
[0.624s][info   ][gc,phases   ] GC(11) Concurrent Mark Free 0.000ms
[0.629s][info   ][gc,phases   ] GC(11) Concurrent Process Non-Strong References 4.673ms
[0.629s][info   ][gc,phases   ] GC(11) Concurrent Reset Relocation Set 0.008ms
[0.631s][info   ][gc,phases   ] GC(11) Concurrent Select Relocation Set 2.151ms
[0.631s][info   ][gc,phases   ] GC(11) Pause Relocate Start 0.007ms
[0.634s][info   ][gc,phases   ] GC(11) Concurrent Relocate 2.506ms
[0.634s][info   ][gc,load     ] GC(11) Load: 3.49/3.80/4.40
[0.634s][info   ][gc,mmu      ] GC(11) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.634s][info   ][gc,marking  ] GC(11) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.634s][info   ][gc,marking  ] GC(11) Mark Stack Usage: 32M
[0.634s][info   ][gc,nmethod  ] GC(11) NMethods: 121 registered, 0 unregistered
[0.634s][info   ][gc,metaspace] GC(11) Metaspace: 0M used, 0M committed, 1088M reserved
[0.634s][info   ][gc,ref      ] GC(11) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.634s][info   ][gc,ref      ] GC(11) Weak: 23 encountered, 2 discovered, 0 enqueued
[0.634s][info   ][gc,ref      ] GC(11) Final: 0 encountered, 0 discovered, 0 enqueued
[0.634s][info   ][gc,ref      ] GC(11) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.634s][info   ][gc,reloc    ] GC(11) Small Pages: 81 / 162M, Empty: 0M, Relocated: 15M, In-Place: 0
[0.634s][info   ][gc,reloc    ] GC(11) Medium Pages: 11 / 352M, Empty: 0M, Relocated: 46M, In-Place: 0
[0.634s][info   ][gc,reloc    ] GC(11) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.634s][info   ][gc,reloc    ] GC(11) Forwarding Usage: 0M
[0.634s][info   ][gc,heap     ] GC(11) Min Capacity: 1024M(100%)
[0.634s][info   ][gc,heap     ] GC(11) Max Capacity: 1024M(100%)
[0.634s][info   ][gc,heap     ] GC(11) Soft Max Capacity: 1024M(100%)
[0.634s][info   ][gc,heap     ] GC(11)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.634s][info   ][gc,heap     ] GC(11)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.634s][info   ][gc,heap     ] GC(11)      Free:      510M (50%)         462M (45%)         402M (39%)         454M (44%)         510M (50%)         360M (35%)
[0.634s][info   ][gc,heap     ] GC(11)      Used:      514M (50%)         562M (55%)         622M (61%)         570M (56%)         664M (65%)         514M (50%)
[0.634s][info   ][gc,heap     ] GC(11)      Live:         -               343M (34%)         343M (34%)         343M (34%)            -                  -
[0.634s][info   ][gc,heap     ] GC(11) Allocated:         -                48M (5%)          108M (11%)         155M (15%)            -                  -
[0.634s][info   ][gc,heap     ] GC(11)   Garbage:         -               170M (17%)         170M (17%)          70M (7%)             -                  -
[0.634s][info   ][gc,heap     ] GC(11) Reclaimed:         -                  -                 0M (0%)           99M (10%)            -                  -
[0.634s][info   ][gc          ] GC(11) Garbage Collection (Warmup) 514M(50%)->570M(56%)
[0.661s][info   ][gc,start    ] GC(12) Garbage Collection (Allocation Stall)
[0.661s][info   ][gc,ref      ] GC(12) Clearing All SoftReferences
[0.661s][info   ][gc,task     ] GC(12) Using 2 workers
[0.661s][info   ][gc,phases   ] GC(12) Pause Mark Start 0.006ms
[0.665s][info   ][gc,phases   ] GC(12) Concurrent Mark 3.750ms
[0.665s][info   ][gc,phases   ] GC(12) Pause Mark End 0.008ms
[0.665s][info   ][gc,phases   ] GC(12) Concurrent Mark Free 0.000ms
[0.665s][info   ][gc,phases   ] GC(12) Concurrent Process Non-Strong References 0.466ms
[0.665s][info   ][gc,phases   ] GC(12) Concurrent Reset Relocation Set 0.002ms
[0.667s][info   ][gc,phases   ] GC(12) Concurrent Select Relocation Set 1.557ms
[0.667s][info   ][gc,phases   ] GC(12) Pause Relocate Start 0.004ms
[0.667s][info   ][gc          ] Allocation Stall (main) 6.807ms
[0.674s][info   ][gc,phases   ] GC(12) Concurrent Relocate 7.056ms
[0.674s][info   ][gc,load     ] GC(12) Load: 3.49/3.80/4.40
[0.674s][info   ][gc,mmu      ] GC(12) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.674s][info   ][gc,marking  ] GC(12) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.674s][info   ][gc,marking  ] GC(12) Mark Stack Usage: 32M
[0.674s][info   ][gc,nmethod  ] GC(12) NMethods: 121 registered, 0 unregistered
[0.674s][info   ][gc,metaspace] GC(12) Metaspace: 0M used, 0M committed, 1088M reserved
[0.674s][info   ][gc,ref      ] GC(12) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.674s][info   ][gc,ref      ] GC(12) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.674s][info   ][gc,ref      ] GC(12) Final: 0 encountered, 0 discovered, 0 enqueued
[0.674s][info   ][gc,ref      ] GC(12) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.674s][info   ][gc,reloc    ] GC(12) Small Pages: 194 / 388M, Empty: 0M, Relocated: 102M, In-Place: 0
[0.674s][info   ][gc,reloc    ] GC(12) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 194M, In-Place: 1
[0.674s][info   ][gc,reloc    ] GC(12) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.674s][info   ][gc,reloc    ] GC(12) Forwarding Usage: 0M
[0.674s][info   ][gc,heap     ] GC(12) Min Capacity: 1024M(100%)
[0.674s][info   ][gc,heap     ] GC(12) Max Capacity: 1024M(100%)
[0.674s][info   ][gc,heap     ] GC(12) Soft Max Capacity: 1024M(100%)
[0.674s][info   ][gc,heap     ] GC(12)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.674s][info   ][gc,heap     ] GC(12)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.674s][info   ][gc,heap     ] GC(12)      Free:       28M (3%)           28M (3%)           28M (3%)          518M (51%)         520M (51%)          26M (3%)
[0.674s][info   ][gc,heap     ] GC(12)      Used:      996M (97%)         996M (97%)         996M (97%)         506M (49%)         998M (97%)         504M (49%)
[0.674s][info   ][gc,heap     ] GC(12)      Live:         -               335M (33%)         335M (33%)         335M (33%)            -                  -
[0.674s][info   ][gc,heap     ] GC(12) Allocated:         -                 0M (0%)            0M (0%)          103M (10%)            -                  -
[0.674s][info   ][gc,heap     ] GC(12)   Garbage:         -               660M (64%)         660M (64%)          66M (6%)             -                  -
[0.674s][info   ][gc,heap     ] GC(12) Reclaimed:         -                  -                 0M (0%)          593M (58%)            -                  -
[0.674s][info   ][gc          ] GC(12) Garbage Collection (Allocation Stall) 996M(97%)->506M(49%)
[0.704s][info   ][gc,start    ] GC(13) Garbage Collection (Allocation Stall)
[0.704s][info   ][gc,ref      ] GC(13) Clearing All SoftReferences
[0.704s][info   ][gc,task     ] GC(13) Using 2 workers
[0.704s][info   ][gc,phases   ] GC(13) Pause Mark Start 0.004ms
[0.708s][info   ][gc,phases   ] GC(13) Concurrent Mark 3.456ms
[0.708s][info   ][gc,phases   ] GC(13) Pause Mark End 0.007ms
[0.708s][info   ][gc,phases   ] GC(13) Concurrent Mark Free 0.000ms
[0.708s][info   ][gc,phases   ] GC(13) Concurrent Process Non-Strong References 0.226ms
[0.708s][info   ][gc,phases   ] GC(13) Concurrent Reset Relocation Set 0.007ms
[0.710s][info   ][gc,phases   ] GC(13) Concurrent Select Relocation Set 1.469ms
[0.710s][info   ][gc,phases   ] GC(13) Pause Relocate Start 0.002ms
[0.713s][info   ][gc          ] Allocation Stall (main) 8.956ms
[0.716s][info   ][gc,phases   ] GC(13) Concurrent Relocate 6.298ms
[0.716s][info   ][gc,load     ] GC(13) Load: 3.45/3.79/4.40
[0.716s][info   ][gc,mmu      ] GC(13) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.716s][info   ][gc,marking  ] GC(13) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.716s][info   ][gc,marking  ] GC(13) Mark Stack Usage: 32M
[0.716s][info   ][gc,nmethod  ] GC(13) NMethods: 121 registered, 0 unregistered
[0.716s][info   ][gc,metaspace] GC(13) Metaspace: 0M used, 0M committed, 1088M reserved
[0.716s][info   ][gc,ref      ] GC(13) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.716s][info   ][gc,ref      ] GC(13) Weak: 23 encountered, 2 discovered, 0 enqueued
[0.716s][info   ][gc,ref      ] GC(13) Final: 0 encountered, 0 discovered, 0 enqueued
[0.716s][info   ][gc,ref      ] GC(13) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.716s][info   ][gc,reloc    ] GC(13) Small Pages: 192 / 384M, Empty: 0M, Relocated: 101M, In-Place: 0
[0.716s][info   ][gc,reloc    ] GC(13) Medium Pages: 20 / 640M, Empty: 0M, Relocated: 193M, In-Place: 1
[0.716s][info   ][gc,reloc    ] GC(13) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.716s][info   ][gc,reloc    ] GC(13) Forwarding Usage: 0M
[0.716s][info   ][gc,heap     ] GC(13) Min Capacity: 1024M(100%)
[0.716s][info   ][gc,heap     ] GC(13) Max Capacity: 1024M(100%)
[0.716s][info   ][gc,heap     ] GC(13) Soft Max Capacity: 1024M(100%)
[0.716s][info   ][gc,heap     ] GC(13)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.716s][info   ][gc,heap     ] GC(13)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.716s][info   ][gc,heap     ] GC(13)      Free:        0M (0%)            0M (0%)            0M (0%)          568M (55%)         570M (56%)           0M (0%)
[0.716s][info   ][gc,heap     ] GC(13)      Used:     1024M (100%)       1024M (100%)       1024M (100%)        456M (45%)        1024M (100%)        454M (44%)
[0.716s][info   ][gc,heap     ] GC(13)      Live:         -               337M (33%)         337M (33%)         337M (33%)            -                  -
[0.716s][info   ][gc,heap     ] GC(13) Allocated:         -                 0M (0%)            0M (0%)           51M (5%)             -                  -
[0.716s][info   ][gc,heap     ] GC(13)   Garbage:         -               686M (67%)         686M (67%)          66M (7%)             -                  -
[0.716s][info   ][gc,heap     ] GC(13) Reclaimed:         -                  -                 0M (0%)          619M (61%)            -                  -
[0.716s][info   ][gc          ] GC(13) Garbage Collection (Allocation Stall) 1024M(100%)->456M(45%)
[0.720s][info   ][gc,start    ] GC(14) Garbage Collection (Allocation Rate)
[0.720s][info   ][gc,task     ] GC(14) Using 2 workers
[0.720s][info   ][gc,phases   ] GC(14) Pause Mark Start 0.004ms
[0.724s][info   ][gc,phases   ] GC(14) Concurrent Mark 4.310ms
[0.724s][info   ][gc,phases   ] GC(14) Pause Mark End 0.007ms
[0.724s][info   ][gc,phases   ] GC(14) Concurrent Mark Free 0.000ms
[0.725s][info   ][gc,phases   ] GC(14) Concurrent Process Non-Strong References 0.277ms
[0.725s][info   ][gc,phases   ] GC(14) Concurrent Reset Relocation Set 0.006ms
[0.726s][info   ][gc,phases   ] GC(14) Concurrent Select Relocation Set 1.429ms
[0.726s][info   ][gc,phases   ] GC(14) Pause Relocate Start 0.005ms
[0.727s][info   ][gc,phases   ] GC(14) Concurrent Relocate 0.958ms
[0.727s][info   ][gc,load     ] GC(14) Load: 3.45/3.79/4.40
[0.727s][info   ][gc,mmu      ] GC(14) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.727s][info   ][gc,marking  ] GC(14) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.727s][info   ][gc,marking  ] GC(14) Mark Stack Usage: 32M
[0.727s][info   ][gc,nmethod  ] GC(14) NMethods: 121 registered, 0 unregistered
[0.727s][info   ][gc,metaspace] GC(14) Metaspace: 0M used, 0M committed, 1088M reserved
[0.727s][info   ][gc,ref      ] GC(14) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.728s][info   ][gc,ref      ] GC(14) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.728s][info   ][gc,ref      ] GC(14) Final: 0 encountered, 0 discovered, 0 enqueued
[0.728s][info   ][gc,ref      ] GC(14) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.728s][info   ][gc,reloc    ] GC(14) Small Pages: 82 / 164M, Empty: 0M, Relocated: 17M, In-Place: 0
[0.728s][info   ][gc,reloc    ] GC(14) Medium Pages: 11 / 352M, Empty: 0M, Relocated: 27M, In-Place: 0
[0.728s][info   ][gc,reloc    ] GC(14) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.728s][info   ][gc,reloc    ] GC(14) Forwarding Usage: 0M
[0.728s][info   ][gc,heap     ] GC(14) Min Capacity: 1024M(100%)
[0.728s][info   ][gc,heap     ] GC(14) Max Capacity: 1024M(100%)
[0.728s][info   ][gc,heap     ] GC(14) Soft Max Capacity: 1024M(100%)
[0.728s][info   ][gc,heap     ] GC(14)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.728s][info   ][gc,heap     ] GC(14)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.728s][info   ][gc,heap     ] GC(14)      Free:      508M (50%)         458M (45%)         450M (44%)         512M (50%)         512M (50%)         416M (41%)
[0.728s][info   ][gc,heap     ] GC(14)      Used:      516M (50%)         566M (55%)         574M (56%)         512M (50%)         608M (59%)         512M (50%)
[0.728s][info   ][gc,heap     ] GC(14)      Live:         -               331M (32%)         331M (32%)         331M (32%)            -                  -
[0.728s][info   ][gc,heap     ] GC(14) Allocated:         -                50M (5%)           58M (6%)           97M (10%)            -                  -
[0.728s][info   ][gc,heap     ] GC(14)   Garbage:         -               184M (18%)         184M (18%)          82M (8%)             -                  -
[0.728s][info   ][gc,heap     ] GC(14) Reclaimed:         -                  -                 0M (0%)          101M (10%)            -                  -
[0.728s][info   ][gc          ] GC(14) Garbage Collection (Allocation Rate) 516M(50%)->512M(50%)
[0.757s][info   ][gc,start    ] GC(15) Garbage Collection (Allocation Stall)
[0.757s][info   ][gc,ref      ] GC(15) Clearing All SoftReferences
[0.757s][info   ][gc,task     ] GC(15) Using 2 workers
[0.757s][info   ][gc,phases   ] GC(15) Pause Mark Start 0.007ms
[0.760s][info   ][gc,phases   ] GC(15) Concurrent Mark 3.584ms
[0.761s][info   ][gc,phases   ] GC(15) Pause Mark End 0.005ms
[0.761s][info   ][gc,phases   ] GC(15) Concurrent Mark Free 0.000ms
[0.761s][info   ][gc,phases   ] GC(15) Concurrent Process Non-Strong References 0.224ms
[0.761s][info   ][gc,phases   ] GC(15) Concurrent Reset Relocation Set 0.002ms
[0.762s][info   ][gc,phases   ] GC(15) Concurrent Select Relocation Set 1.453ms
[0.762s][info   ][gc,phases   ] GC(15) Pause Relocate Start 0.004ms
[0.764s][info   ][gc          ] Allocation Stall (main) 7.591ms
[0.769s][info   ][gc,phases   ] GC(15) Concurrent Relocate 6.677ms
[0.769s][info   ][gc,load     ] GC(15) Load: 3.45/3.79/4.40
[0.769s][info   ][gc,mmu      ] GC(15) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.769s][info   ][gc,marking  ] GC(15) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.769s][info   ][gc,marking  ] GC(15) Mark Stack Usage: 32M
[0.769s][info   ][gc,nmethod  ] GC(15) NMethods: 121 registered, 0 unregistered
[0.769s][info   ][gc,metaspace] GC(15) Metaspace: 0M used, 0M committed, 1088M reserved
[0.769s][info   ][gc,ref      ] GC(15) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.769s][info   ][gc,ref      ] GC(15) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.769s][info   ][gc,ref      ] GC(15) Final: 0 encountered, 0 discovered, 0 enqueued
[0.769s][info   ][gc,ref      ] GC(15) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.769s][info   ][gc,reloc    ] GC(15) Small Pages: 195 / 390M, Empty: 2M, Relocated: 99M, In-Place: 0
[0.769s][info   ][gc,reloc    ] GC(15) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 211M, In-Place: 1
[0.769s][info   ][gc,reloc    ] GC(15) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.769s][info   ][gc,reloc    ] GC(15) Forwarding Usage: 0M
[0.769s][info   ][gc,heap     ] GC(15) Min Capacity: 1024M(100%)
[0.769s][info   ][gc,heap     ] GC(15) Max Capacity: 1024M(100%)
[0.769s][info   ][gc,heap     ] GC(15) Soft Max Capacity: 1024M(100%)
[0.769s][info   ][gc,heap     ] GC(15)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.769s][info   ][gc,heap     ] GC(15)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.769s][info   ][gc,heap     ] GC(15)      Free:       26M (3%)           26M (3%)           28M (3%)          554M (54%)         560M (55%)          26M (3%)
[0.769s][info   ][gc,heap     ] GC(15)      Used:      998M (97%)         998M (97%)         996M (97%)         470M (46%)         998M (97%)         464M (45%)
[0.769s][info   ][gc,heap     ] GC(15)      Live:         -               339M (33%)         339M (33%)         339M (33%)            -                  -
[0.769s][info   ][gc,heap     ] GC(15) Allocated:         -                 0M (0%)            0M (0%)           95M (9%)             -                  -
[0.769s][info   ][gc,heap     ] GC(15)   Garbage:         -               658M (64%)         656M (64%)          34M (3%)             -                  -
[0.769s][info   ][gc,heap     ] GC(15) Reclaimed:         -                  -                 2M (0%)          623M (61%)            -                  -
[0.769s][info   ][gc          ] GC(15) Garbage Collection (Allocation Stall) 998M(97%)->470M(46%)
[0.800s][info   ][gc,start    ] GC(16) Garbage Collection (Allocation Stall)
[0.800s][info   ][gc,ref      ] GC(16) Clearing All SoftReferences
[0.800s][info   ][gc,task     ] GC(16) Using 2 workers
[0.800s][info   ][gc,phases   ] GC(16) Pause Mark Start 0.003ms
[0.804s][info   ][gc,phases   ] GC(16) Concurrent Mark 3.066ms
[0.804s][info   ][gc,phases   ] GC(16) Pause Mark End 0.005ms
[0.804s][info   ][gc,phases   ] GC(16) Concurrent Mark Free 0.000ms
[0.804s][info   ][gc,phases   ] GC(16) Concurrent Process Non-Strong References 0.245ms
[0.804s][info   ][gc,phases   ] GC(16) Concurrent Reset Relocation Set 0.006ms
[0.805s][info   ][gc,phases   ] GC(16) Concurrent Select Relocation Set 1.477ms
[0.806s][info   ][gc,phases   ] GC(16) Pause Relocate Start 0.005ms
[0.806s][info   ][gc          ] Allocation Stall (main) 5.740ms
[0.812s][info   ][gc,phases   ] GC(16) Concurrent Relocate 6.476ms
[0.812s][info   ][gc,load     ] GC(16) Load: 3.45/3.79/4.40
[0.812s][info   ][gc,mmu      ] GC(16) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.812s][info   ][gc,marking  ] GC(16) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.812s][info   ][gc,marking  ] GC(16) Mark Stack Usage: 32M
[0.812s][info   ][gc,nmethod  ] GC(16) NMethods: 121 registered, 0 unregistered
[0.812s][info   ][gc,metaspace] GC(16) Metaspace: 0M used, 0M committed, 1088M reserved
[0.812s][info   ][gc,ref      ] GC(16) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.812s][info   ][gc,ref      ] GC(16) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.812s][info   ][gc,ref      ] GC(16) Final: 0 encountered, 0 discovered, 0 enqueued
[0.812s][info   ][gc,ref      ] GC(16) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.812s][info   ][gc,reloc    ] GC(16) Small Pages: 199 / 398M, Empty: 0M, Relocated: 103M, In-Place: 0
[0.812s][info   ][gc,reloc    ] GC(16) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 195M, In-Place: 1
[0.812s][info   ][gc,reloc    ] GC(16) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.812s][info   ][gc,reloc    ] GC(16) Forwarding Usage: 0M
[0.812s][info   ][gc,heap     ] GC(16) Min Capacity: 1024M(100%)
[0.812s][info   ][gc,heap     ] GC(16) Max Capacity: 1024M(100%)
[0.812s][info   ][gc,heap     ] GC(16) Soft Max Capacity: 1024M(100%)
[0.812s][info   ][gc,heap     ] GC(16)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.812s][info   ][gc,heap     ] GC(16)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.812s][info   ][gc,heap     ] GC(16)      Free:       18M (2%)           18M (2%)           18M (2%)          522M (51%)         540M (53%)          16M (2%)
[0.812s][info   ][gc,heap     ] GC(16)      Used:     1006M (98%)        1006M (98%)        1006M (98%)         502M (49%)        1008M (98%)         484M (47%)
[0.812s][info   ][gc,heap     ] GC(16)      Live:         -               336M (33%)         336M (33%)         336M (33%)            -                  -
[0.812s][info   ][gc,heap     ] GC(16) Allocated:         -                 0M (0%)            0M (0%)           97M (10%)            -                  -
[0.812s][info   ][gc,heap     ] GC(16)   Garbage:         -               669M (65%)         669M (65%)          67M (7%)             -                  -
[0.812s][info   ][gc,heap     ] GC(16) Reclaimed:         -                  -                 0M (0%)          601M (59%)            -                  -
[0.812s][info   ][gc          ] GC(16) Garbage Collection (Allocation Stall) 1006M(98%)->502M(49%)
[0.820s][info   ][gc,start    ] GC(17) Garbage Collection (Allocation Rate)
[0.820s][info   ][gc,task     ] GC(17) Using 2 workers
[0.820s][info   ][gc,phases   ] GC(17) Pause Mark Start 0.003ms
[0.826s][info   ][gc,phases   ] GC(17) Concurrent Mark 6.329ms
[0.826s][info   ][gc,phases   ] GC(17) Pause Mark End 0.008ms
[0.827s][info   ][gc,phases   ] GC(17) Concurrent Mark Free 0.000ms
[0.827s][info   ][gc,phases   ] GC(17) Concurrent Process Non-Strong References 0.247ms
[0.827s][info   ][gc,phases   ] GC(17) Concurrent Reset Relocation Set 0.008ms
[0.828s][info   ][gc,phases   ] GC(17) Concurrent Select Relocation Set 1.442ms
[0.828s][info   ][gc,phases   ] GC(17) Pause Relocate Start 0.005ms
[0.831s][info   ][gc,phases   ] GC(17) Concurrent Relocate 2.171ms
[0.831s][info   ][gc,load     ] GC(17) Load: 3.45/3.79/4.40
[0.831s][info   ][gc,mmu      ] GC(17) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.831s][info   ][gc,marking  ] GC(17) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.831s][info   ][gc,marking  ] GC(17) Mark Stack Usage: 32M
[0.831s][info   ][gc,nmethod  ] GC(17) NMethods: 121 registered, 0 unregistered
[0.831s][info   ][gc,metaspace] GC(17) Metaspace: 0M used, 0M committed, 1088M reserved
[0.831s][info   ][gc,ref      ] GC(17) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.831s][info   ][gc,ref      ] GC(17) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.831s][info   ][gc,ref      ] GC(17) Final: 0 encountered, 0 discovered, 0 enqueued
[0.831s][info   ][gc,ref      ] GC(17) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.831s][info   ][gc,reloc    ] GC(17) Small Pages: 103 / 206M, Empty: 0M, Relocated: 24M, In-Place: 0
[0.831s][info   ][gc,reloc    ] GC(17) Medium Pages: 13 / 416M, Empty: 0M, Relocated: 76M, In-Place: 0
[0.831s][info   ][gc,reloc    ] GC(17) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.831s][info   ][gc,reloc    ] GC(17) Forwarding Usage: 0M
[0.831s][info   ][gc,heap     ] GC(17) Min Capacity: 1024M(100%)
[0.831s][info   ][gc,heap     ] GC(17) Max Capacity: 1024M(100%)
[0.831s][info   ][gc,heap     ] GC(17) Soft Max Capacity: 1024M(100%)
[0.831s][info   ][gc,heap     ] GC(17)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.831s][info   ][gc,heap     ] GC(17)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.831s][info   ][gc,heap     ] GC(17)      Free:      402M (39%)         342M (33%)         302M (29%)         488M (48%)         490M (48%)         268M (26%)
[0.831s][info   ][gc,heap     ] GC(17)      Used:      622M (61%)         682M (67%)         722M (71%)         536M (52%)         756M (74%)         534M (52%)
[0.831s][info   ][gc,heap     ] GC(17)      Live:         -               328M (32%)         328M (32%)         328M (32%)            -                  -
[0.831s][info   ][gc,heap     ] GC(17) Allocated:         -                60M (6%)          100M (10%)         117M (12%)            -                  -
[0.831s][info   ][gc,heap     ] GC(17)   Garbage:         -               293M (29%)         293M (29%)          89M (9%)             -                  -
[0.831s][info   ][gc,heap     ] GC(17) Reclaimed:         -                  -                 0M (0%)          203M (20%)            -                  -
[0.831s][info   ][gc          ] GC(17) Garbage Collection (Allocation Rate) 622M(61%)->536M(52%)
[0.858s][info   ][gc,start    ] GC(18) Garbage Collection (Allocation Stall)
[0.858s][info   ][gc,ref      ] GC(18) Clearing All SoftReferences
[0.858s][info   ][gc,task     ] GC(18) Using 2 workers
[0.858s][info   ][gc,phases   ] GC(18) Pause Mark Start 0.007ms
[0.861s][info   ][gc,phases   ] GC(18) Concurrent Mark 2.957ms
[0.861s][info   ][gc,phases   ] GC(18) Pause Mark End 0.004ms
[0.861s][info   ][gc,phases   ] GC(18) Concurrent Mark Free 0.000ms
[0.862s][info   ][gc,phases   ] GC(18) Concurrent Process Non-Strong References 0.225ms
[0.862s][info   ][gc,phases   ] GC(18) Concurrent Reset Relocation Set 0.003ms
[0.863s][info   ][gc,phases   ] GC(18) Concurrent Select Relocation Set 1.466ms
[0.863s][info   ][gc,phases   ] GC(18) Pause Relocate Start 0.005ms
[0.865s][info   ][gc          ] Allocation Stall (main) 7.012ms
[0.869s][info   ][gc,phases   ] GC(18) Concurrent Relocate 6.186ms
[0.869s][info   ][gc,load     ] GC(18) Load: 3.45/3.79/4.40
[0.869s][info   ][gc,mmu      ] GC(18) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.870s][info   ][gc,marking  ] GC(18) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.870s][info   ][gc,marking  ] GC(18) Mark Stack Usage: 32M
[0.870s][info   ][gc,nmethod  ] GC(18) NMethods: 121 registered, 0 unregistered
[0.870s][info   ][gc,metaspace] GC(18) Metaspace: 0M used, 0M committed, 1088M reserved
[0.870s][info   ][gc,ref      ] GC(18) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.870s][info   ][gc,ref      ] GC(18) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.870s][info   ][gc,ref      ] GC(18) Final: 0 encountered, 0 discovered, 0 enqueued
[0.870s][info   ][gc,ref      ] GC(18) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.870s][info   ][gc,reloc    ] GC(18) Small Pages: 200 / 400M, Empty: 0M, Relocated: 99M, In-Place: 0
[0.870s][info   ][gc,reloc    ] GC(18) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 188M, In-Place: 1
[0.870s][info   ][gc,reloc    ] GC(18) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.870s][info   ][gc,reloc    ] GC(18) Forwarding Usage: 0M
[0.870s][info   ][gc,heap     ] GC(18) Min Capacity: 1024M(100%)
[0.870s][info   ][gc,heap     ] GC(18) Max Capacity: 1024M(100%)
[0.870s][info   ][gc,heap     ] GC(18) Soft Max Capacity: 1024M(100%)
[0.870s][info   ][gc,heap     ] GC(18)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.870s][info   ][gc,heap     ] GC(18)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.870s][info   ][gc,heap     ] GC(18)      Free:       16M (2%)           16M (2%)           16M (2%)          558M (54%)         586M (57%)          16M (2%)
[0.870s][info   ][gc,heap     ] GC(18)      Used:     1008M (98%)        1008M (98%)        1008M (98%)         466M (46%)        1008M (98%)         438M (43%)
[0.870s][info   ][gc,heap     ] GC(18)      Live:         -               333M (33%)         333M (33%)         333M (33%)            -                  -
[0.870s][info   ][gc,heap     ] GC(18) Allocated:         -                 0M (0%)            0M (0%)           91M (9%)             -                  -
[0.870s][info   ][gc,heap     ] GC(18)   Garbage:         -               674M (66%)         674M (66%)          40M (4%)             -                  -
[0.870s][info   ][gc,heap     ] GC(18) Reclaimed:         -                  -                 0M (0%)          633M (62%)            -                  -
[0.870s][info   ][gc          ] GC(18) Garbage Collection (Allocation Stall) 1008M(98%)->466M(46%)
[0.901s][info   ][gc,start    ] GC(19) Garbage Collection (Allocation Stall)
[0.902s][info   ][gc,ref      ] GC(19) Clearing All SoftReferences
[0.902s][info   ][gc,task     ] GC(19) Using 2 workers
[0.902s][info   ][gc,phases   ] GC(19) Pause Mark Start 0.004ms
[0.905s][info   ][gc,phases   ] GC(19) Concurrent Mark 2.959ms
[0.905s][info   ][gc,phases   ] GC(19) Pause Mark End 0.006ms
[0.905s][info   ][gc,phases   ] GC(19) Concurrent Mark Free 0.000ms
[0.905s][info   ][gc,phases   ] GC(19) Concurrent Process Non-Strong References 0.261ms
[0.905s][info   ][gc,phases   ] GC(19) Concurrent Reset Relocation Set 0.007ms
[0.907s][info   ][gc,phases   ] GC(19) Concurrent Select Relocation Set 1.605ms
[0.907s][info   ][gc,phases   ] GC(19) Pause Relocate Start 0.005ms
[0.907s][info   ][gc          ] Allocation Stall (main) 5.839ms
[0.914s][info   ][gc,phases   ] GC(19) Concurrent Relocate 7.134ms
[0.914s][info   ][gc,load     ] GC(19) Load: 3.45/3.79/4.40
[0.914s][info   ][gc,mmu      ] GC(19) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.914s][info   ][gc,marking  ] GC(19) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.914s][info   ][gc,marking  ] GC(19) Mark Stack Usage: 32M
[0.914s][info   ][gc,nmethod  ] GC(19) NMethods: 121 registered, 0 unregistered
[0.914s][info   ][gc,metaspace] GC(19) Metaspace: 0M used, 0M committed, 1088M reserved
[0.914s][info   ][gc,ref      ] GC(19) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.914s][info   ][gc,ref      ] GC(19) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.914s][info   ][gc,ref      ] GC(19) Final: 0 encountered, 0 discovered, 0 enqueued
[0.914s][info   ][gc,ref      ] GC(19) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.914s][info   ][gc,reloc    ] GC(19) Small Pages: 204 / 408M, Empty: 0M, Relocated: 101M, In-Place: 0
[0.914s][info   ][gc,reloc    ] GC(19) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 193M, In-Place: 1
[0.914s][info   ][gc,reloc    ] GC(19) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.914s][info   ][gc,reloc    ] GC(19) Forwarding Usage: 0M
[0.914s][info   ][gc,heap     ] GC(19) Min Capacity: 1024M(100%)
[0.914s][info   ][gc,heap     ] GC(19) Max Capacity: 1024M(100%)
[0.914s][info   ][gc,heap     ] GC(19) Soft Max Capacity: 1024M(100%)
[0.914s][info   ][gc,heap     ] GC(19)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.914s][info   ][gc,heap     ] GC(19)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.914s][info   ][gc,heap     ] GC(19)      Free:        8M (1%)            8M (1%)            8M (1%)          512M (50%)         512M (50%)           6M (1%)
[0.914s][info   ][gc,heap     ] GC(19)      Used:     1016M (99%)        1016M (99%)        1016M (99%)         512M (50%)        1018M (99%)         512M (50%)
[0.914s][info   ][gc,heap     ] GC(19)      Live:         -               341M (33%)         341M (33%)         341M (33%)            -                  -
[0.914s][info   ][gc,heap     ] GC(19) Allocated:         -                 0M (0%)            0M (0%)          103M (10%)            -                  -
[0.914s][info   ][gc,heap     ] GC(19)   Garbage:         -               674M (66%)         674M (66%)          66M (7%)             -                  -
[0.914s][info   ][gc,heap     ] GC(19) Reclaimed:         -                  -                 0M (0%)          607M (59%)            -                  -
[0.914s][info   ][gc          ] GC(19) Garbage Collection (Allocation Stall) 1016M(99%)->512M(50%)
[0.920s][info   ][gc,start    ] GC(20) Garbage Collection (Allocation Rate)
[0.920s][info   ][gc,task     ] GC(20) Using 2 workers
[0.920s][info   ][gc,phases   ] GC(20) Pause Mark Start 0.003ms
[0.924s][info   ][gc,phases   ] GC(20) Concurrent Mark 4.449ms
[0.925s][info   ][gc,phases   ] GC(20) Pause Mark End 0.008ms
[0.925s][info   ][gc,phases   ] GC(20) Concurrent Mark Free 0.000ms
[0.925s][info   ][gc,phases   ] GC(20) Concurrent Process Non-Strong References 0.308ms
[0.925s][info   ][gc,phases   ] GC(20) Concurrent Reset Relocation Set 0.007ms
[0.926s][info   ][gc,phases   ] GC(20) Concurrent Select Relocation Set 1.432ms
[0.927s][info   ][gc,phases   ] GC(20) Pause Relocate Start 0.005ms
[0.929s][info   ][gc,phases   ] GC(20) Concurrent Relocate 2.234ms
[0.929s][info   ][gc,load     ] GC(20) Load: 3.45/3.79/4.40
[0.929s][info   ][gc,mmu      ] GC(20) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.929s][info   ][gc,marking  ] GC(20) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.929s][info   ][gc,marking  ] GC(20) Mark Stack Usage: 32M
[0.929s][info   ][gc,nmethod  ] GC(20) NMethods: 121 registered, 0 unregistered
[0.929s][info   ][gc,metaspace] GC(20) Metaspace: 0M used, 0M committed, 1088M reserved
[0.929s][info   ][gc,ref      ] GC(20) Soft: 3 encountered, 0 discovered, 0 enqueued
[0.929s][info   ][gc,ref      ] GC(20) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.929s][info   ][gc,ref      ] GC(20) Final: 0 encountered, 0 discovered, 0 enqueued
[0.929s][info   ][gc,ref      ] GC(20) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.929s][info   ][gc,reloc    ] GC(20) Small Pages: 104 / 208M, Empty: 0M, Relocated: 34M, In-Place: 0
[0.929s][info   ][gc,reloc    ] GC(20) Medium Pages: 13 / 416M, Empty: 0M, Relocated: 77M, In-Place: 0
[0.929s][info   ][gc,reloc    ] GC(20) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.929s][info   ][gc,reloc    ] GC(20) Forwarding Usage: 0M
[0.929s][info   ][gc,heap     ] GC(20) Min Capacity: 1024M(100%)
[0.929s][info   ][gc,heap     ] GC(20) Max Capacity: 1024M(100%)
[0.929s][info   ][gc,heap     ] GC(20) Soft Max Capacity: 1024M(100%)
[0.929s][info   ][gc,heap     ] GC(20)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.929s][info   ][gc,heap     ] GC(20)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.929s][info   ][gc,heap     ] GC(20)      Free:      400M (39%)         350M (34%)         312M (30%)         502M (49%)         502M (49%)         280M (27%)
[0.929s][info   ][gc,heap     ] GC(20)      Used:      624M (61%)         674M (66%)         712M (70%)         522M (51%)         744M (73%)         522M (51%)
[0.929s][info   ][gc,heap     ] GC(20)      Live:         -               322M (31%)         322M (31%)         322M (31%)            -                  -
[0.929s][info   ][gc,heap     ] GC(20) Allocated:         -                50M (5%)           88M (9%)           99M (10%)            -                  -
[0.929s][info   ][gc,heap     ] GC(20)   Garbage:         -               301M (29%)         301M (29%)          99M (10%)            -                  -
[0.929s][info   ][gc,heap     ] GC(20) Reclaimed:         -                  -                 0M (0%)          201M (20%)            -                  -
[0.929s][info   ][gc          ] GC(20) Garbage Collection (Allocation Rate) 624M(61%)->522M(51%)
[0.959s][info   ][gc,start    ] GC(21) Garbage Collection (Allocation Stall)
[0.959s][info   ][gc,ref      ] GC(21) Clearing All SoftReferences
[0.959s][info   ][gc,task     ] GC(21) Using 2 workers
[0.959s][info   ][gc,phases   ] GC(21) Pause Mark Start 0.007ms
[0.962s][info   ][gc,phases   ] GC(21) Concurrent Mark 2.826ms
[0.962s][info   ][gc,phases   ] GC(21) Pause Mark End 0.005ms
[0.962s][info   ][gc,phases   ] GC(21) Concurrent Mark Free 0.000ms
[0.963s][info   ][gc,phases   ] GC(21) Concurrent Process Non-Strong References 0.229ms
[0.963s][info   ][gc,phases   ] GC(21) Concurrent Reset Relocation Set 0.003ms
[0.964s][info   ][gc,phases   ] GC(21) Concurrent Select Relocation Set 1.519ms
[0.964s][info   ][gc,phases   ] GC(21) Pause Relocate Start 0.002ms
[0.965s][info   ][gc          ] Allocation Stall (main) 5.619ms
[0.971s][info   ][gc,phases   ] GC(21) Concurrent Relocate 7.294ms
[0.972s][info   ][gc,load     ] GC(21) Load: 3.45/3.79/4.40
[0.972s][info   ][gc,mmu      ] GC(21) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[0.972s][info   ][gc,marking  ] GC(21) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.972s][info   ][gc,marking  ] GC(21) Mark Stack Usage: 32M
[0.972s][info   ][gc,nmethod  ] GC(21) NMethods: 121 registered, 0 unregistered
[0.972s][info   ][gc,metaspace] GC(21) Metaspace: 0M used, 0M committed, 1088M reserved
[0.972s][info   ][gc,ref      ] GC(21) Soft: 3 encountered, 1 discovered, 0 enqueued
[0.972s][info   ][gc,ref      ] GC(21) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.972s][info   ][gc,ref      ] GC(21) Final: 0 encountered, 0 discovered, 0 enqueued
[0.972s][info   ][gc,ref      ] GC(21) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.972s][info   ][gc,reloc    ] GC(21) Small Pages: 203 / 406M, Empty: 4M, Relocated: 104M, In-Place: 0
[0.972s][info   ][gc,reloc    ] GC(21) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 195M, In-Place: 1
[0.972s][info   ][gc,reloc    ] GC(21) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.972s][info   ][gc,reloc    ] GC(21) Forwarding Usage: 0M
[0.972s][info   ][gc,heap     ] GC(21) Min Capacity: 1024M(100%)
[0.972s][info   ][gc,heap     ] GC(21) Max Capacity: 1024M(100%)
[0.972s][info   ][gc,heap     ] GC(21) Soft Max Capacity: 1024M(100%)
[0.972s][info   ][gc,heap     ] GC(21)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.972s][info   ][gc,heap     ] GC(21)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[0.972s][info   ][gc,heap     ] GC(21)      Free:       10M (1%)           10M (1%)           14M (1%)          546M (53%)         546M (53%)          10M (1%)
[0.972s][info   ][gc,heap     ] GC(21)      Used:     1014M (99%)        1014M (99%)        1010M (99%)         478M (47%)        1014M (99%)         478M (47%)
[0.972s][info   ][gc,heap     ] GC(21)      Live:         -               327M (32%)         327M (32%)         327M (32%)            -                  -
[0.972s][info   ][gc,heap     ] GC(21) Allocated:         -                 0M (0%)            0M (0%)           99M (10%)            -                  -
[0.972s][info   ][gc,heap     ] GC(21)   Garbage:         -               686M (67%)         682M (67%)          50M (5%)             -                  -
[0.972s][info   ][gc,heap     ] GC(21) Reclaimed:         -                  -                 4M (0%)          635M (62%)            -                  -
[0.972s][info   ][gc          ] GC(21) Garbage Collection (Allocation Stall) 1014M(99%)->478M(47%)
[1.007s][info   ][gc,start    ] GC(22) Garbage Collection (Allocation Stall)
[1.007s][info   ][gc,ref      ] GC(22) Clearing All SoftReferences
[1.007s][info   ][gc,task     ] GC(22) Using 2 workers
[1.007s][info   ][gc,phases   ] GC(22) Pause Mark Start 0.004ms
[1.010s][info   ][gc,phases   ] GC(22) Concurrent Mark 3.220ms
[1.011s][info   ][gc,phases   ] GC(22) Pause Mark End 0.005ms
[1.011s][info   ][gc,phases   ] GC(22) Concurrent Mark Free 0.000ms
[1.011s][info   ][gc,phases   ] GC(22) Concurrent Process Non-Strong References 0.176ms
[1.011s][info   ][gc,phases   ] GC(22) Concurrent Reset Relocation Set 0.007ms
[1.012s][info   ][gc,phases   ] GC(22) Concurrent Select Relocation Set 1.408ms
[1.012s][info   ][gc,phases   ] GC(22) Pause Relocate Start 0.005ms
[1.014s][info   ][gc          ] Allocation Stall (main) 7.377ms
[1.021s][info   ][gc,phases   ] GC(22) Concurrent Relocate 8.435ms
[1.021s][info   ][gc,load     ] GC(22) Load: 3.45/3.79/4.40
[1.021s][info   ][gc,mmu      ] GC(22) MMU: 2ms/99.3%, 5ms/99.6%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/99.9%
[1.021s][info   ][gc,marking  ] GC(22) Mark: 2 stripe(s), 1 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[1.021s][info   ][gc,marking  ] GC(22) Mark Stack Usage: 32M
[1.021s][info   ][gc,nmethod  ] GC(22) NMethods: 121 registered, 0 unregistered
[1.021s][info   ][gc,metaspace] GC(22) Metaspace: 0M used, 0M committed, 1088M reserved
[1.021s][info   ][gc,ref      ] GC(22) Soft: 3 encountered, 1 discovered, 0 enqueued
[1.021s][info   ][gc,ref      ] GC(22) Weak: 23 encountered, 16 discovered, 0 enqueued
[1.021s][info   ][gc,ref      ] GC(22) Final: 0 encountered, 0 discovered, 0 enqueued
[1.021s][info   ][gc,ref      ] GC(22) Phantom: 1 encountered, 0 discovered, 0 enqueued
[1.021s][info   ][gc,reloc    ] GC(22) Small Pages: 207 / 414M, Empty: 2M, Relocated: 103M, In-Place: 0
[1.021s][info   ][gc,reloc    ] GC(22) Medium Pages: 19 / 608M, Empty: 0M, Relocated: 192M, In-Place: 1
[1.021s][info   ][gc,reloc    ] GC(22) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[1.021s][info   ][gc,reloc    ] GC(22) Forwarding Usage: 0M
[1.021s][info   ][gc,heap     ] GC(22) Min Capacity: 1024M(100%)
[1.021s][info   ][gc,heap     ] GC(22) Max Capacity: 1024M(100%)
[1.021s][info   ][gc,heap     ] GC(22) Soft Max Capacity: 1024M(100%)
[1.021s][info   ][gc,heap     ] GC(22)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[1.021s][info   ][gc,heap     ] GC(22)  Capacity:     1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)       1024M (100%)
[1.021s][info   ][gc,heap     ] GC(22)      Free:        2M (0%)            2M (0%)            4M (0%)          516M (50%)         518M (51%)           2M (0%)
[1.021s][info   ][gc,heap     ] GC(22)      Used:     1022M (100%)       1022M (100%)       1020M (100%)        508M (50%)        1022M (100%)        506M (49%)
[1.021s][info   ][gc,heap     ] GC(22)      Live:         -               334M (33%)         334M (33%)         334M (33%)            -                  -
[1.021s][info   ][gc,heap     ] GC(22) Allocated:         -                 0M (0%)            0M (0%)          101M (10%)            -                  -
[1.021s][info   ][gc,heap     ] GC(22)   Garbage:         -               687M (67%)         685M (67%)          71M (7%)             -                  -
[1.021s][info   ][gc,heap     ] GC(22) Reclaimed:         -                  -                 2M (0%)          615M (60%)            -                  -
[1.021s][info   ][gc          ] GC(22) Garbage Collection (Allocation Stall) 1022M(100%)->508M(50%)
counter:57527
[1.029s][info   ][gc,heap,exit] Heap
[1.029s][info   ][gc,heap,exit]  ZHeap           used 526M, capacity 1024M, max capacity 1024M
[1.029s][info   ][gc,heap,exit]  Metaspace       used 278K, committed 512K, reserved 1114112K
[1.029s][info   ][gc,heap,exit]   class space    used 9K, committed 128K, reserved 1048576K
```

再试试2g内存
```
java -XX:+UseZGC -Xms2g -Xmx2g -XX:+PrintGCDetails -Xlog:gc\*:file=gc.log:time,uptime,level,tags GCLogAnalysis
```

结果是9次GC，生成了54000多个对象
```
[0.002s][warning][gc] -XX:+PrintGCDetails is deprecated. Will use -Xlog:gc* instead.
[0.005s][info   ][gc,init] Initializing The Z Garbage Collector
[0.005s][info   ][gc,init] Version: 19.0.2+7-44 (release)
[0.005s][info   ][gc,init] NUMA Support: Disabled
[0.005s][info   ][gc,init] CPUs: 8 total, 8 available
[0.005s][info   ][gc,init] Memory: 16384M
[0.005s][info   ][gc,init] Large Page Support: Disabled
[0.005s][info   ][gc,init] GC Workers: 2 (dynamic)
[0.005s][info   ][gc,init] Address Space Type: Contiguous/Unrestricted/Complete
[0.005s][info   ][gc,init] Address Space Size: 32768M x 3 = 98304M
[0.005s][info   ][gc,init] Min Capacity: 2048M
[0.006s][info   ][gc,init] Initial Capacity: 2048M
[0.006s][info   ][gc,init] Max Capacity: 2048M
[0.006s][info   ][gc,init] Medium Page Size: 32M
[0.006s][info   ][gc,init] Pre-touch: Disabled
[0.006s][info   ][gc,init] Uncommit: Implicitly Disabled (-Xms equals -Xmx)
[0.006s][info   ][gc,init] Runtime Workers: 5
[0.006s][info   ][gc     ] Using The Z Garbage Collector
[0.006s][info   ][gc,metaspace] CDS archive(s) mapped at: [0x0000000800000000-0x0000000800c44000-0x0000000800c44000), size 12861440, SharedBaseAddress: 0x0000000800000000, ArchiveRelocationMode: 0.
[0.006s][info   ][gc,metaspace] Compressed class space mapped at: 0x0000000801000000-0x0000000841000000, reserved size: 1073741824
[0.006s][info   ][gc,metaspace] Narrow klass base: 0x0000000800000000, Narrow klass shift: 0, Narrow klass range: 0x100000000
gc...
[0.115s][info   ][gc,start    ] GC(0) Garbage Collection (Warmup)
[0.115s][info   ][gc,task     ] GC(0) Using 2 workers
[0.115s][info   ][gc,phases   ] GC(0) Pause Mark Start 0.005ms
[0.120s][info   ][gc,phases   ] GC(0) Concurrent Mark 5.021ms
[0.121s][info   ][gc,phases   ] GC(0) Pause Mark End 0.008ms
[0.121s][info   ][gc,phases   ] GC(0) Concurrent Mark Free 0.000ms
[0.121s][info   ][gc,phases   ] GC(0) Concurrent Process Non-Strong References 0.291ms
[0.121s][info   ][gc,phases   ] GC(0) Concurrent Reset Relocation Set 0.000ms
[0.127s][info   ][gc,phases   ] GC(0) Concurrent Select Relocation Set 5.404ms
[0.127s][info   ][gc,phases   ] GC(0) Pause Relocate Start 0.012ms
[0.138s][info   ][gc,phases   ] GC(0) Concurrent Relocate 11.631ms
[0.139s][info   ][gc,load     ] GC(0) Load: 5.25/4.13/4.45
[0.139s][info   ][gc,mmu      ] GC(0) MMU: 2ms/99.4%, 5ms/99.8%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.139s][info   ][gc,marking  ] GC(0) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.139s][info   ][gc,marking  ] GC(0) Mark Stack Usage: 32M
[0.139s][info   ][gc,nmethod  ] GC(0) NMethods: 118 registered, 0 unregistered
[0.139s][info   ][gc,metaspace] GC(0) Metaspace: 0M used, 0M committed, 1088M reserved
[0.139s][info   ][gc,ref      ] GC(0) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.139s][info   ][gc,ref      ] GC(0) Weak: 26 encountered, 19 discovered, 3 enqueued
[0.139s][info   ][gc,ref      ] GC(0) Final: 0 encountered, 0 discovered, 0 enqueued
[0.139s][info   ][gc,ref      ] GC(0) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.139s][info   ][gc,reloc    ] GC(0) Small Pages: 155 / 310M, Empty: 2M, Relocated: 60M, In-Place: 0
[0.139s][info   ][gc,reloc    ] GC(0) Medium Pages: 12 / 384M, Empty: 0M, Relocated: 133M, In-Place: 0
[0.139s][info   ][gc,reloc    ] GC(0) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.139s][info   ][gc,reloc    ] GC(0) Forwarding Usage: 0M
[0.139s][info   ][gc,heap     ] GC(0) Min Capacity: 2048M(100%)
[0.139s][info   ][gc,heap     ] GC(0) Max Capacity: 2048M(100%)
[0.139s][info   ][gc,heap     ] GC(0) Soft Max Capacity: 2048M(100%)
[0.139s][info   ][gc,heap     ] GC(0)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.139s][info   ][gc,heap     ] GC(0)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.139s][info   ][gc,heap     ] GC(0)      Free:     1354M (66%)        1308M (64%)        1262M (62%)        1664M (81%)        1664M (81%)        1226M (60%)
[0.139s][info   ][gc,heap     ] GC(0)      Used:      694M (34%)         740M (36%)         786M (38%)         384M (19%)         822M (40%)         384M (19%)
[0.139s][info   ][gc,heap     ] GC(0)      Live:         -               195M (10%)         195M (10%)         195M (10%)            -                  -
[0.139s][info   ][gc,heap     ] GC(0) Allocated:         -                46M (2%)           94M (5%)          155M (8%)             -                  -
[0.139s][info   ][gc,heap     ] GC(0)   Garbage:         -               498M (24%)         496M (24%)          32M (2%)             -                  -
[0.139s][info   ][gc,heap     ] GC(0) Reclaimed:         -                  -                 2M (0%)          465M (23%)            -                  -
[0.139s][info   ][gc          ] GC(0) Garbage Collection (Warmup) 694M(34%)->384M(19%)
[0.218s][info   ][gc,start    ] GC(1) Garbage Collection (Warmup)
[0.218s][info   ][gc,task     ] GC(1) Using 2 workers
[0.218s][info   ][gc,phases   ] GC(1) Pause Mark Start 0.006ms
[0.222s][info   ][gc,phases   ] GC(1) Concurrent Mark 3.648ms
[0.222s][info   ][gc,phases   ] GC(1) Pause Mark End 0.007ms
[0.222s][info   ][gc,phases   ] GC(1) Concurrent Mark Free 0.000ms
[0.223s][info   ][gc,phases   ] GC(1) Concurrent Process Non-Strong References 0.250ms
[0.223s][info   ][gc,phases   ] GC(1) Concurrent Reset Relocation Set 0.005ms
[0.225s][info   ][gc,phases   ] GC(1) Concurrent Select Relocation Set 2.101ms
[0.225s][info   ][gc,phases   ] GC(1) Pause Relocate Start 0.007ms
[0.242s][info   ][gc,phases   ] GC(1) Concurrent Relocate 16.905ms
[0.242s][info   ][gc,load     ] GC(1) Load: 5.25/4.13/4.45
[0.242s][info   ][gc,mmu      ] GC(1) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.242s][info   ][gc,marking  ] GC(1) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.242s][info   ][gc,marking  ] GC(1) Mark Stack Usage: 32M
[0.242s][info   ][gc,nmethod  ] GC(1) NMethods: 122 registered, 0 unregistered
[0.242s][info   ][gc,metaspace] GC(1) Metaspace: 0M used, 0M committed, 1088M reserved
[0.242s][info   ][gc,ref      ] GC(1) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.242s][info   ][gc,ref      ] GC(1) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.242s][info   ][gc,ref      ] GC(1) Final: 0 encountered, 0 discovered, 0 enqueued
[0.242s][info   ][gc,ref      ] GC(1) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.242s][info   ][gc,reloc    ] GC(1) Small Pages: 267 / 534M, Empty: 10M, Relocated: 90M, In-Place: 0
[0.242s][info   ][gc,reloc    ] GC(1) Medium Pages: 24 / 768M, Empty: 0M, Relocated: 195M, In-Place: 0
[0.242s][info   ][gc,reloc    ] GC(1) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.242s][info   ][gc,reloc    ] GC(1) Forwarding Usage: 0M
[0.242s][info   ][gc,heap     ] GC(1) Min Capacity: 2048M(100%)
[0.242s][info   ][gc,heap     ] GC(1) Max Capacity: 2048M(100%)
[0.242s][info   ][gc,heap     ] GC(1) Soft Max Capacity: 2048M(100%)
[0.242s][info   ][gc,heap     ] GC(1)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.242s][info   ][gc,heap     ] GC(1)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.242s][info   ][gc,heap     ] GC(1)      Free:      746M (36%)         698M (34%)         698M (34%)        1474M (72%)        1474M (72%)         666M (33%)
[0.242s][info   ][gc,heap     ] GC(1)      Used:     1302M (64%)        1350M (66%)        1350M (66%)         574M (28%)        1382M (67%)         574M (28%)
[0.242s][info   ][gc,heap     ] GC(1)      Live:         -               302M (15%)         302M (15%)         302M (15%)            -                  -
[0.242s][info   ][gc,heap     ] GC(1) Allocated:         -                48M (2%)           58M (3%)          219M (11%)            -                  -
[0.242s][info   ][gc,heap     ] GC(1)   Garbage:         -               999M (49%)         989M (48%)          51M (3%)             -                  -
[0.242s][info   ][gc,heap     ] GC(1) Reclaimed:         -                  -                10M (0%)          947M (46%)            -                  -
[0.242s][info   ][gc          ] GC(1) Garbage Collection (Warmup) 1302M(64%)->574M(28%)
[0.318s][info   ][gc,start    ] GC(2) Garbage Collection (Warmup)
[0.318s][info   ][gc,task     ] GC(2) Using 2 workers
[0.318s][info   ][gc,phases   ] GC(2) Pause Mark Start 0.006ms
[0.324s][info   ][gc,phases   ] GC(2) Concurrent Mark 5.506ms
[0.324s][info   ][gc,phases   ] GC(2) Pause Mark End 0.006ms
[0.324s][info   ][gc,phases   ] GC(2) Concurrent Mark Free 0.000ms
[0.328s][info   ][gc,phases   ] GC(2) Concurrent Process Non-Strong References 4.195ms
[0.328s][info   ][gc,phases   ] GC(2) Concurrent Reset Relocation Set 0.010ms
[0.330s][info   ][gc,phases   ] GC(2) Concurrent Select Relocation Set 1.556ms
[0.330s][info   ][gc,phases   ] GC(2) Pause Relocate Start 0.005ms
[0.343s][info   ][gc,phases   ] GC(2) Concurrent Relocate 12.763ms
[0.343s][info   ][gc,load     ] GC(2) Load: 5.25/4.13/4.45
[0.343s][info   ][gc,mmu      ] GC(2) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.343s][info   ][gc,marking  ] GC(2) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.343s][info   ][gc,marking  ] GC(2) Mark Stack Usage: 32M
[0.343s][info   ][gc,nmethod  ] GC(2) NMethods: 127 registered, 0 unregistered
[0.343s][info   ][gc,metaspace] GC(2) Metaspace: 0M used, 0M committed, 1088M reserved
[0.343s][info   ][gc,ref      ] GC(2) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.343s][info   ][gc,ref      ] GC(2) Weak: 23 encountered, 17 discovered, 0 enqueued
[0.343s][info   ][gc,ref      ] GC(2) Final: 0 encountered, 0 discovered, 0 enqueued
[0.343s][info   ][gc,ref      ] GC(2) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.343s][info   ][gc,reloc    ] GC(2) Small Pages: 318 / 636M, Empty: 20M, Relocated: 102M, In-Place: 0
[0.343s][info   ][gc,reloc    ] GC(2) Medium Pages: 30 / 960M, Empty: 32M, Relocated: 217M, In-Place: 0
[0.343s][info   ][gc,reloc    ] GC(2) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.343s][info   ][gc,reloc    ] GC(2) Forwarding Usage: 0M
[0.343s][info   ][gc,heap     ] GC(2) Min Capacity: 2048M(100%)
[0.343s][info   ][gc,heap     ] GC(2) Max Capacity: 2048M(100%)
[0.343s][info   ][gc,heap     ] GC(2) Soft Max Capacity: 2048M(100%)
[0.343s][info   ][gc,heap     ] GC(2)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.343s][info   ][gc,heap     ] GC(2)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.343s][info   ][gc,heap     ] GC(2)      Free:      452M (22%)         404M (20%)         402M (20%)        1480M (72%)        1482M (72%)         352M (17%)
[0.343s][info   ][gc,heap     ] GC(2)      Used:     1596M (78%)        1644M (80%)        1646M (80%)         568M (28%)        1696M (83%)         566M (28%)
[0.343s][info   ][gc,heap     ] GC(2)      Live:         -               321M (16%)         321M (16%)         321M (16%)            -                  -
[0.343s][info   ][gc,heap     ] GC(2) Allocated:         -                48M (2%)          102M (5%)          233M (11%)            -                  -
[0.343s][info   ][gc,heap     ] GC(2)   Garbage:         -              1274M (62%)        1222M (60%)          12M (1%)             -                  -
[0.344s][info   ][gc,heap     ] GC(2) Reclaimed:         -                  -                52M (3%)         1261M (62%)            -                  -
[0.344s][info   ][gc          ] GC(2) Garbage Collection (Warmup) 1596M(78%)->568M(28%)
[0.418s][info   ][gc,start    ] GC(3) Garbage Collection (Allocation Rate)
[0.418s][info   ][gc,task     ] GC(3) Using 2 workers
[0.418s][info   ][gc,phases   ] GC(3) Pause Mark Start 0.004ms
[0.424s][info   ][gc,phases   ] GC(3) Concurrent Mark 5.888ms
[0.424s][info   ][gc,phases   ] GC(3) Pause Mark End 0.006ms
[0.424s][info   ][gc,phases   ] GC(3) Concurrent Mark Free 0.000ms
[0.426s][info   ][gc,phases   ] GC(3) Concurrent Process Non-Strong References 1.397ms
[0.426s][info   ][gc,phases   ] GC(3) Concurrent Reset Relocation Set 0.010ms
[0.427s][info   ][gc,phases   ] GC(3) Concurrent Select Relocation Set 1.562ms
[0.428s][info   ][gc,phases   ] GC(3) Pause Relocate Start 0.003ms
[0.457s][info   ][gc,phases   ] GC(3) Concurrent Relocate 29.631ms
[0.458s][info   ][gc,load     ] GC(3) Load: 5.25/4.13/4.45
[0.458s][info   ][gc,mmu      ] GC(3) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.458s][info   ][gc,marking  ] GC(3) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.458s][info   ][gc,marking  ] GC(3) Mark Stack Usage: 32M
[0.458s][info   ][gc,nmethod  ] GC(3) NMethods: 127 registered, 0 unregistered
[0.458s][info   ][gc,metaspace] GC(3) Metaspace: 0M used, 0M committed, 1088M reserved
[0.458s][info   ][gc,ref      ] GC(3) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.458s][info   ][gc,ref      ] GC(3) Weak: 23 encountered, 3 discovered, 0 enqueued
[0.458s][info   ][gc,ref      ] GC(3) Final: 0 encountered, 0 discovered, 0 enqueued
[0.458s][info   ][gc,ref      ] GC(3) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.458s][info   ][gc,reloc    ] GC(3) Small Pages: 379 / 758M, Empty: 44M, Relocated: 104M, In-Place: 0
[0.458s][info   ][gc,reloc    ] GC(3) Medium Pages: 33 / 1056M, Empty: 0M, Relocated: 217M, In-Place: 0
[0.458s][info   ][gc,reloc    ] GC(3) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.458s][info   ][gc,reloc    ] GC(3) Forwarding Usage: 0M
[0.458s][info   ][gc,heap     ] GC(3) Min Capacity: 2048M(100%)
[0.458s][info   ][gc,heap     ] GC(3) Max Capacity: 2048M(100%)
[0.458s][info   ][gc,heap     ] GC(3) Soft Max Capacity: 2048M(100%)
[0.458s][info   ][gc,heap     ] GC(3)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.458s][info   ][gc,heap     ] GC(3)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.458s][info   ][gc,heap     ] GC(3)      Free:      234M (11%)         186M (9%)          190M (9%)         1540M (75%)        1570M (77%)         146M (7%)
[0.458s][info   ][gc,heap     ] GC(3)      Used:     1814M (89%)        1862M (91%)        1858M (91%)         508M (25%)        1902M (93%)         478M (23%)
[0.458s][info   ][gc,heap     ] GC(3)      Live:         -               337M (16%)         337M (16%)         337M (16%)            -                  -
[0.458s][info   ][gc,heap     ] GC(3) Allocated:         -                48M (2%)           88M (4%)          141M (7%)             -                  -
[0.458s][info   ][gc,heap     ] GC(3)   Garbage:         -              1476M (72%)        1432M (70%)          28M (1%)             -                  -
[0.459s][info   ][gc,heap     ] GC(3) Reclaimed:         -                  -                44M (2%)         1447M (71%)            -                  -
[0.459s][info   ][gc          ] GC(3) Garbage Collection (Allocation Rate) 1814M(89%)->508M(25%)
[0.518s][info   ][gc,start    ] GC(4) Garbage Collection (Allocation Rate)
[0.518s][info   ][gc,task     ] GC(4) Using 2 workers
[0.518s][info   ][gc,phases   ] GC(4) Pause Mark Start 0.004ms
[0.523s][info   ][gc,phases   ] GC(4) Concurrent Mark 4.595ms
[0.523s][info   ][gc,phases   ] GC(4) Pause Mark End 0.008ms
[0.523s][info   ][gc,phases   ] GC(4) Concurrent Mark Free 0.000ms
[0.529s][info   ][gc,phases   ] GC(4) Concurrent Process Non-Strong References 5.904ms
[0.529s][info   ][gc,phases   ] GC(4) Concurrent Reset Relocation Set 0.015ms
[0.531s][info   ][gc,phases   ] GC(4) Concurrent Select Relocation Set 1.670ms
[0.531s][info   ][gc,phases   ] GC(4) Pause Relocate Start 0.010ms
[0.540s][info   ][gc,phases   ] GC(4) Concurrent Relocate 9.221ms
[0.540s][info   ][gc,load     ] GC(4) Load: 5.25/4.13/4.45
[0.540s][info   ][gc,mmu      ] GC(4) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.540s][info   ][gc,marking  ] GC(4) Mark: 2 stripe(s), 3 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.540s][info   ][gc,marking  ] GC(4) Mark Stack Usage: 32M
[0.540s][info   ][gc,nmethod  ] GC(4) NMethods: 128 registered, 0 unregistered
[0.540s][info   ][gc,metaspace] GC(4) Metaspace: 0M used, 0M committed, 1088M reserved
[0.540s][info   ][gc,ref      ] GC(4) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.540s][info   ][gc,ref      ] GC(4) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.540s][info   ][gc,ref      ] GC(4) Final: 0 encountered, 0 discovered, 0 enqueued
[0.540s][info   ][gc,ref      ] GC(4) Phantom: 1 encountered, 1 discovered, 0 enqueued
[0.540s][info   ][gc,reloc    ] GC(4) Small Pages: 292 / 584M, Empty: 12M, Relocated: 108M, In-Place: 0
[0.540s][info   ][gc,reloc    ] GC(4) Medium Pages: 26 / 832M, Empty: 0M, Relocated: 221M, In-Place: 0
[0.541s][info   ][gc,reloc    ] GC(4) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.541s][info   ][gc,reloc    ] GC(4) Forwarding Usage: 0M
[0.541s][info   ][gc,heap     ] GC(4) Min Capacity: 2048M(100%)
[0.541s][info   ][gc,heap     ] GC(4) Max Capacity: 2048M(100%)
[0.541s][info   ][gc,heap     ] GC(4) Soft Max Capacity: 2048M(100%)
[0.541s][info   ][gc,heap     ] GC(4)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.541s][info   ][gc,heap     ] GC(4)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.541s][info   ][gc,heap     ] GC(4)      Free:      632M (31%)         584M (29%)         532M (26%)        1448M (71%)        1448M (71%)         498M (24%)
[0.541s][info   ][gc,heap     ] GC(4)      Used:     1416M (69%)        1464M (71%)        1516M (74%)         600M (29%)        1550M (76%)         600M (29%)
[0.541s][info   ][gc,heap     ] GC(4)      Live:         -               346M (17%)         346M (17%)         346M (17%)            -                  -
[0.541s][info   ][gc,heap     ] GC(4) Allocated:         -                48M (2%)          112M (5%)          229M (11%)            -                  -
[0.541s][info   ][gc,heap     ] GC(4)   Garbage:         -              1069M (52%)        1057M (52%)          23M (1%)             -                  -
[0.541s][info   ][gc,heap     ] GC(4) Reclaimed:         -                  -                12M (1%)         1045M (51%)            -                  -
[0.541s][info   ][gc          ] GC(4) Garbage Collection (Allocation Rate) 1416M(69%)->600M(29%)
[0.618s][info   ][gc,start    ] GC(5) Garbage Collection (Allocation Rate)
[0.618s][info   ][gc,task     ] GC(5) Using 2 workers
[0.618s][info   ][gc,phases   ] GC(5) Pause Mark Start 0.004ms
[0.623s][info   ][gc,phases   ] GC(5) Concurrent Mark 4.536ms
[0.623s][info   ][gc,phases   ] GC(5) Pause Mark End 0.007ms
[0.623s][info   ][gc,phases   ] GC(5) Concurrent Mark Free 0.000ms
[0.624s][info   ][gc,phases   ] GC(5) Concurrent Process Non-Strong References 0.288ms
[0.624s][info   ][gc,phases   ] GC(5) Concurrent Reset Relocation Set 0.010ms
[0.625s][info   ][gc,phases   ] GC(5) Concurrent Select Relocation Set 1.430ms
[0.625s][info   ][gc,phases   ] GC(5) Pause Relocate Start 0.004ms
[0.633s][info   ][gc,phases   ] GC(5) Concurrent Relocate 7.538ms
[0.633s][info   ][gc,load     ] GC(5) Load: 5.25/4.13/4.45
[0.633s][info   ][gc,mmu      ] GC(5) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.633s][info   ][gc,marking  ] GC(5) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.633s][info   ][gc,marking  ] GC(5) Mark Stack Usage: 32M
[0.633s][info   ][gc,nmethod  ] GC(5) NMethods: 128 registered, 0 unregistered
[0.633s][info   ][gc,metaspace] GC(5) Metaspace: 0M used, 0M committed, 1088M reserved
[0.633s][info   ][gc,ref      ] GC(5) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.633s][info   ][gc,ref      ] GC(5) Weak: 23 encountered, 16 discovered, 0 enqueued
[0.633s][info   ][gc,ref      ] GC(5) Final: 0 encountered, 0 discovered, 0 enqueued
[0.633s][info   ][gc,ref      ] GC(5) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.633s][info   ][gc,reloc    ] GC(5) Small Pages: 379 / 758M, Empty: 38M, Relocated: 106M, In-Place: 0
[0.633s][info   ][gc,reloc    ] GC(5) Medium Pages: 34 / 1088M, Empty: 0M, Relocated: 219M, In-Place: 0
[0.633s][info   ][gc,reloc    ] GC(5) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.633s][info   ][gc,reloc    ] GC(5) Forwarding Usage: 0M
[0.633s][info   ][gc,heap     ] GC(5) Min Capacity: 2048M(100%)
[0.633s][info   ][gc,heap     ] GC(5) Max Capacity: 2048M(100%)
[0.633s][info   ][gc,heap     ] GC(5) Soft Max Capacity: 2048M(100%)
[0.633s][info   ][gc,heap     ] GC(5)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.633s][info   ][gc,heap     ] GC(5)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.633s][info   ][gc,heap     ] GC(5)      Free:      202M (10%)         152M (7%)          182M (9%)         1470M (72%)        1496M (73%)         146M (7%)
[0.633s][info   ][gc,heap     ] GC(5)      Used:     1846M (90%)        1896M (93%)        1866M (91%)         578M (28%)        1902M (93%)         552M (27%)
[0.633s][info   ][gc,heap     ] GC(5)      Live:         -               340M (17%)         340M (17%)         340M (17%)            -                  -
[0.633s][info   ][gc,heap     ] GC(5) Allocated:         -                50M (2%)           58M (3%)          207M (10%)            -                  -
[0.633s][info   ][gc,heap     ] GC(5)   Garbage:         -              1505M (74%)        1467M (72%)          29M (1%)             -                  -
[0.633s][info   ][gc,heap     ] GC(5) Reclaimed:         -                  -                38M (2%)         1475M (72%)            -                  -
[0.633s][info   ][gc          ] GC(5) Garbage Collection (Allocation Rate) 1846M(90%)->578M(28%)
[0.716s][info   ][gc,start    ] GC(6) Garbage Collection (Allocation Rate)
[0.716s][info   ][gc,task     ] GC(6) Using 2 workers
[0.716s][info   ][gc,phases   ] GC(6) Pause Mark Start 0.004ms
[0.731s][info   ][gc,phases   ] GC(6) Concurrent Mark 14.523ms
[0.731s][info   ][gc,phases   ] GC(6) Pause Mark End 0.007ms
[0.731s][info   ][gc,phases   ] GC(6) Concurrent Mark Free 0.000ms
[0.733s][info   ][gc,phases   ] GC(6) Concurrent Process Non-Strong References 1.525ms
[0.733s][info   ][gc,phases   ] GC(6) Concurrent Reset Relocation Set 0.014ms
[0.736s][info   ][gc,phases   ] GC(6) Concurrent Select Relocation Set 2.500ms
[0.736s][info   ][gc,phases   ] GC(6) Pause Relocate Start 0.006ms
[0.780s][info   ][gc,phases   ] GC(6) Concurrent Relocate 44.301ms
[0.780s][info   ][gc,load     ] GC(6) Load: 5.25/4.13/4.45
[0.780s][info   ][gc,mmu      ] GC(6) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.780s][info   ][gc,marking  ] GC(6) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.780s][info   ][gc,marking  ] GC(6) Mark Stack Usage: 32M
[0.780s][info   ][gc,nmethod  ] GC(6) NMethods: 128 registered, 0 unregistered
[0.780s][info   ][gc,metaspace] GC(6) Metaspace: 0M used, 0M committed, 1088M reserved
[0.780s][info   ][gc,ref      ] GC(6) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.780s][info   ][gc,ref      ] GC(6) Weak: 23 encountered, 17 discovered, 0 enqueued
[0.780s][info   ][gc,ref      ] GC(6) Final: 0 encountered, 0 discovered, 0 enqueued
[0.780s][info   ][gc,ref      ] GC(6) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.780s][info   ][gc,reloc    ] GC(6) Small Pages: 405 / 810M, Empty: 40M, Relocated: 110M, In-Place: 0
[0.780s][info   ][gc,reloc    ] GC(6) Medium Pages: 35 / 1120M, Empty: 0M, Relocated: 222M, In-Place: 0
[0.780s][info   ][gc,reloc    ] GC(6) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.780s][info   ][gc,reloc    ] GC(6) Forwarding Usage: 0M
[0.780s][info   ][gc,heap     ] GC(6) Min Capacity: 2048M(100%)
[0.780s][info   ][gc,heap     ] GC(6) Max Capacity: 2048M(100%)
[0.780s][info   ][gc,heap     ] GC(6) Soft Max Capacity: 2048M(100%)
[0.780s][info   ][gc,heap     ] GC(6)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.780s][info   ][gc,heap     ] GC(6)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.780s][info   ][gc,heap     ] GC(6)      Free:      118M (6%)           78M (4%)          112M (5%)         1320M (64%)        1322M (65%)          74M (4%)
[0.780s][info   ][gc,heap     ] GC(6)      Used:     1930M (94%)        1970M (96%)        1936M (95%)         728M (36%)        1974M (96%)         726M (35%)
[0.780s][info   ][gc,heap     ] GC(6)      Live:         -               347M (17%)         347M (17%)         347M (17%)            -                  -
[0.780s][info   ][gc,heap     ] GC(6) Allocated:         -                40M (2%)           46M (2%)          325M (16%)            -                  -
[0.780s][info   ][gc,heap     ] GC(6)   Garbage:         -              1582M (77%)        1542M (75%)          54M (3%)             -                  -
[0.780s][info   ][gc,heap     ] GC(6) Reclaimed:         -                  -                40M (2%)         1527M (75%)            -                  -
[0.781s][info   ][gc          ] GC(6) Garbage Collection (Allocation Rate) 1930M(94%)->728M(36%)
[0.818s][info   ][gc,start    ] GC(7) Garbage Collection (Allocation Rate)
[0.818s][info   ][gc,task     ] GC(7) Using 2 workers
[0.818s][info   ][gc,phases   ] GC(7) Pause Mark Start 0.003ms
[0.823s][info   ][gc,phases   ] GC(7) Concurrent Mark 4.747ms
[0.823s][info   ][gc,phases   ] GC(7) Pause Mark End 0.008ms
[0.823s][info   ][gc,phases   ] GC(7) Concurrent Mark Free 0.000ms
[0.824s][info   ][gc,phases   ] GC(7) Concurrent Process Non-Strong References 0.327ms
[0.824s][info   ][gc,phases   ] GC(7) Concurrent Reset Relocation Set 0.013ms
[0.827s][info   ][gc,phases   ] GC(7) Concurrent Select Relocation Set 3.474ms
[0.827s][info   ][gc,phases   ] GC(7) Pause Relocate Start 0.004ms
[0.852s][info   ][gc,phases   ] GC(7) Concurrent Relocate 24.640ms
[0.852s][info   ][gc,load     ] GC(7) Load: 5.25/4.13/4.45
[0.852s][info   ][gc,mmu      ] GC(7) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.852s][info   ][gc,marking  ] GC(7) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.852s][info   ][gc,marking  ] GC(7) Mark Stack Usage: 32M
[0.852s][info   ][gc,nmethod  ] GC(7) NMethods: 128 registered, 0 unregistered
[0.852s][info   ][gc,metaspace] GC(7) Metaspace: 0M used, 0M committed, 1088M reserved
[0.852s][info   ][gc,ref      ] GC(7) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.852s][info   ][gc,ref      ] GC(7) Weak: 23 encountered, 10 discovered, 0 enqueued
[0.852s][info   ][gc,ref      ] GC(7) Final: 0 encountered, 0 discovered, 0 enqueued
[0.852s][info   ][gc,ref      ] GC(7) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.852s][info   ][gc,reloc    ] GC(7) Small Pages: 229 / 458M, Empty: 4M, Relocated: 106M, In-Place: 0
[0.852s][info   ][gc,reloc    ] GC(7) Medium Pages: 23 / 736M, Empty: 32M, Relocated: 211M, In-Place: 0
[0.852s][info   ][gc,reloc    ] GC(7) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.852s][info   ][gc,reloc    ] GC(7) Forwarding Usage: 0M
[0.852s][info   ][gc,heap     ] GC(7) Min Capacity: 2048M(100%)
[0.852s][info   ][gc,heap     ] GC(7) Max Capacity: 2048M(100%)
[0.852s][info   ][gc,heap     ] GC(7) Soft Max Capacity: 2048M(100%)
[0.852s][info   ][gc,heap     ] GC(7)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.853s][info   ][gc,heap     ] GC(7)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.853s][info   ][gc,heap     ] GC(7)      Free:      854M (42%)         808M (39%)         794M (39%)        1406M (69%)        1432M (70%)         760M (37%)
[0.853s][info   ][gc,heap     ] GC(7)      Used:     1194M (58%)        1240M (61%)        1254M (61%)         642M (31%)        1288M (63%)         616M (30%)
[0.853s][info   ][gc,heap     ] GC(7)      Live:         -               336M (16%)         336M (16%)         336M (16%)            -                  -
[0.853s][info   ][gc,heap     ] GC(7) Allocated:         -                46M (2%)           96M (5%)          269M (13%)            -                  -
[0.853s][info   ][gc,heap     ] GC(7)   Garbage:         -               857M (42%)         821M (40%)          35M (2%)             -                  -
[0.853s][info   ][gc,heap     ] GC(7) Reclaimed:         -                  -                36M (2%)          821M (40%)            -                  -
[0.853s][info   ][gc          ] GC(7) Garbage Collection (Allocation Rate) 1194M(58%)->642M(31%)
[0.918s][info   ][gc,start    ] GC(8) Garbage Collection (Allocation Rate)
[0.918s][info   ][gc,task     ] GC(8) Using 2 workers
[0.918s][info   ][gc,phases   ] GC(8) Pause Mark Start 0.011ms
[0.923s][info   ][gc,phases   ] GC(8) Concurrent Mark 4.585ms
[0.923s][info   ][gc,phases   ] GC(8) Pause Mark End 0.005ms
[0.923s][info   ][gc,phases   ] GC(8) Concurrent Mark Free 0.000ms
[0.924s][info   ][gc,phases   ] GC(8) Concurrent Process Non-Strong References 0.352ms
[0.924s][info   ][gc,phases   ] GC(8) Concurrent Reset Relocation Set 0.008ms
[0.925s][info   ][gc,phases   ] GC(8) Concurrent Select Relocation Set 1.484ms
[0.925s][info   ][gc,phases   ] GC(8) Pause Relocate Start 0.005ms
[0.936s][info   ][gc,phases   ] GC(8) Concurrent Relocate 10.284ms
[0.936s][info   ][gc,load     ] GC(8) Load: 5.25/4.13/4.45
[0.936s][info   ][gc,mmu      ] GC(8) MMU: 2ms/99.4%, 5ms/99.7%, 10ms/99.8%, 20ms/99.9%, 50ms/99.9%, 100ms/100.0%
[0.936s][info   ][gc,marking  ] GC(8) Mark: 2 stripe(s), 2 proactive flush(es), 1 terminate flush(es), 0 completion(s), 0 continuation(s)
[0.936s][info   ][gc,marking  ] GC(8) Mark Stack Usage: 32M
[0.936s][info   ][gc,nmethod  ] GC(8) NMethods: 128 registered, 0 unregistered
[0.936s][info   ][gc,metaspace] GC(8) Metaspace: 0M used, 0M committed, 1088M reserved
[0.936s][info   ][gc,ref      ] GC(8) Soft: 5 encountered, 0 discovered, 0 enqueued
[0.936s][info   ][gc,ref      ] GC(8) Weak: 23 encountered, 0 discovered, 0 enqueued
[0.936s][info   ][gc,ref      ] GC(8) Final: 0 encountered, 0 discovered, 0 enqueued
[0.936s][info   ][gc,ref      ] GC(8) Phantom: 1 encountered, 0 discovered, 0 enqueued
[0.936s][info   ][gc,reloc    ] GC(8) Small Pages: 311 / 622M, Empty: 18M, Relocated: 114M, In-Place: 0
[0.936s][info   ][gc,reloc    ] GC(8) Medium Pages: 27 / 864M, Empty: 0M, Relocated: 220M, In-Place: 0
[0.936s][info   ][gc,reloc    ] GC(8) Large Pages: 0 / 0M, Empty: 0M, Relocated: 0M, In-Place: 0
[0.936s][info   ][gc,reloc    ] GC(8) Forwarding Usage: 0M
[0.936s][info   ][gc,heap     ] GC(8) Min Capacity: 2048M(100%)
[0.936s][info   ][gc,heap     ] GC(8) Max Capacity: 2048M(100%)
[0.936s][info   ][gc,heap     ] GC(8) Soft Max Capacity: 2048M(100%)
[0.936s][info   ][gc,heap     ] GC(8)                Mark Start          Mark End        Relocate Start      Relocate End           High               Low
[0.936s][info   ][gc,heap     ] GC(8)  Capacity:     2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)       2048M (100%)
[0.936s][info   ][gc,heap     ] GC(8)      Free:      562M (27%)         516M (25%)         526M (26%)        1490M (73%)        1490M (73%)         488M (24%)
[0.936s][info   ][gc,heap     ] GC(8)      Used:     1486M (73%)        1532M (75%)        1522M (74%)         558M (27%)        1560M (76%)         558M (27%)
[0.936s][info   ][gc,heap     ] GC(8)      Live:         -               335M (16%)         335M (16%)         335M (16%)            -                  -
[0.936s][info   ][gc,heap     ] GC(8) Allocated:         -                46M (2%)           54M (3%)          213M (10%)            -                  -
[0.936s][info   ][gc,heap     ] GC(8)   Garbage:         -              1150M (56%)        1132M (55%)           8M (0%)             -                  -
[0.936s][info   ][gc,heap     ] GC(8) Reclaimed:         -                  -                18M (1%)         1141M (56%)            -                  -
[0.936s][info   ][gc          ] GC(8) Garbage Collection (Allocation Rate) 1486M(73%)->558M(27%)
[1.018s][info   ][gc,start    ] GC(9) Garbage Collection (Allocation Rate)
[1.018s][info   ][gc,task     ] GC(9) Using 2 workers
[1.019s][info   ][gc,phases   ] GC(9) Pause Mark Start 0.005ms
[1.023s][info   ][gc,phases   ] GC(9) Concurrent Mark 4.596ms
[1.023s][info   ][gc,phases   ] GC(9) Pause Mark End 0.005ms
[1.023s][info   ][gc,phases   ] GC(9) Concurrent Mark Free 0.000ms
[1.024s][info   ][gc,phases   ] GC(9) Concurrent Process Non-Strong References 0.288ms
[1.024s][info   ][gc,phases   ] GC(9) Concurrent Reset Relocation Set 0.010ms
[1.025s][info   ][gc,phases   ] GC(9) Concurrent Select Relocation Set 1.497ms
[1.025s][info   ][gc,phases   ] GC(9) Pause Relocate Start 0.002ms
counter:54449
[1.029s][info   ][gc          ] GC(9) Garbage Collection (Allocation Rate) Aborted
[1.029s][info   ][gc,heap,exit] Heap
[1.029s][info   ][gc,heap,exit]  ZHeap           used 1228M, capacity 2048M, max capacity 2048M
[1.029s][info   ][gc,heap,exit]  Metaspace       used 280K, committed 512K, reserved 1114112K
[1.029s][info   ][gc,heap,exit]   class space    used 9K, committed 128K, reserved 1048576K
```

#### Shenandoah GC

目标是极低延迟，尽可能减少 STW 时间。采用并发压缩技术（Concurrent Compaction），避免碎片化问题。大部分回收工作与应用线程并发执行。

优点：
- GC 停顿时间低。
- 内存整理避免碎片化。
- 对堆内存的需求比 ZGC 更低。

缺点：
- 较新的 GC，生态和调优支持可能不如 G1 和 ZGC。
- CPU 开销高。

适用场景：
- 延迟敏感的应用。
- 中大型堆内存的场景。

#### GC总结

从GC的发展历史来看，目标是两个
1. 尽可能减少停顿时间，不影响业务。
2. 尽可能减少GC时间，让GC更快。

通过以上的对比可以发现，堆的大小不能设置的太小导致OOM，同样当堆设置的过大对于性能也并没有什么提升，比如2g对比1g内存，就没什么提升。从512m和1g内存对比上来看，ZGC最优，不仅减少了GC次数，性能也很好。而G1GC的优化最好，因为它的GC次数最多，但是性能同样强大。

从结果来看最好的是ZGC，其次可以选择G1GC或者并行GC。
