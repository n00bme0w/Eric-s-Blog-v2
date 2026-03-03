---
title: "如何阅读adb dumpsys meminfo"
date: 2026-03-03
draft: false
description: "a description"
tags: ["example", "tag"]
---
{{< katex >}}
## 例子(抖音)
使用dumpsys meminfo com.corporation.package可以查看应用使用内存的情况


```
blazer:/ $ dumpsys meminfo com.ss.android.ugc.aweme        
Applications Memory Usage (in Kilobytes):
Uptime: 1954149922 Realtime: 2520666124

** MEMINFO in pid 24989 [com.ss.android.ugc.aweme] **
                   Pss  Private  Private     Swap      Rss     Heap     Heap     Heap
                 Total    Dirty    Clean    Dirty    Total     Size    Alloc     Free
                ------   ------   ------   ------   ------   ------   ------   ------
  Native Heap    96260    96244        0        0   100616   132796   104822    23789
  Dalvik Heap   142377   142324        0        0   150532   224107   125803    98304
 Dalvik Other    21484    21332        0        0    23204                           
        Stack     8864     8864        0        0     8872                           
       Ashmem     1172      160        0        0     3984                           
    Other dev      144        0      144        0      456                           
     .so mmap    85092     6820    72432        0   165076                           
    .jar mmap     1694        0        4        0    51284                           
    .apk mmap    47703      500    38372        0    69472                           
    .ttf mmap     1112        0      688        0     1808                           
    .dex mmap   560981      772   558064        0   564252                           
    .oat mmap      318        0        0        0    13212                           
    .art mmap     1846     1328      368        0    22832                           
   Other mmap    25399       52    22748        0    35620                           
   EGL mtrack    51460    51460        0        0    51460                           
    GL mtrack    22580    22580        0        0    22580                           
      Unknown    76309    74628     1680        0    77792                           
        TOTAL  1144795   427064   694500        0  1363052   356903   230625   122093
 
 App Summary
                       Pss(KB)                        Rss(KB)
                        ------                         ------
           Java Heap:   144020                         173364
         Native Heap:    96244                         100616
                Code:   677652                         865404
               Stack:     8864                           8872
            Graphics:    74040                          74040
       Private Other:   120744
              System:    23231
             Unknown:                                  140756
 
           TOTAL PSS:  1144795            TOTAL RSS:  1363052      TOTAL SWAP (KB):        0
 
 Objects
               Views:     2883         ViewRootImpl:        1
         AppContexts:        9           Activities:        1
              Assets:       32        AssetManagers:        0
       Local Binders:      196        Proxy Binders:      116
       Parcel memory:      132         Parcel count:      474
    Death Recipients:        5             WebViews:        1
 
 Native Allocations
                         Count                       Total(kB)
                        ------                         ------
   Bitmap (malloced):      131                           4575
    Other (malloced):     7725                            737
 Other (nonmalloced):      342                            200
 
 SQL
         MEMORY_USED:     6285
  PAGECACHE_OVERFLOW:     4686          MALLOC_SIZE:     3563
 
 DATABASES
      pgsz     dbsz   Lookaside(b) cache hits cache misses cache size  Dbname
PER CONNECTION STATS
         4       20             37     1   150     2  /data/user/0/com.ss.android.ugc.aweme/databases/Content.db
         4       32             43     2   245     4  /data/user/0/com.ss.android.ugc.aweme/databases/free_flow
         4        8                    0     0     0    (attached) temp
         4       32             27     1   146     2  /data/user/0/com.ss.android.ugc.aweme/databases/free_flow (2)
         4       48            115  8798   236    17  /data/user/0/com.ss.android.ugc.aweme/databases/ss_app_log.db
         4       48            110   181   167    10  /data/user/0/com.ss.android.ugc.aweme/databases/ss_app_log.db (2)
         4       28             23     1   150     2  /data/user/0/com.ss.android.ugc.aweme/databases/aweme.db
         4       20             23     1   160     2  /data/user/0/com.ss.android.ugc.aweme/databases/verifystorage.db
         4       20             29  2332   161     3  /data/user/0/com.ss.android.ugc.aweme/databases/applog_priority_db_P1@1128
         4       20             29     1   162     2  /data/user/0/com.ss.android.ugc.aweme/databases/TTVideoEngine_vod_strategy_database_v01
         4       44            123  1384   157     7  /data/user/0/com.ss.android.ugc.aweme/databases/downloader.db
         4       48             67   133   150     4  /data/user/0/com.ss.android.ugc.aweme/databases/bytebench_db
         4       20             29  2326   161     3  /data/user/0/com.ss.android.ugc.aweme/databases/applog_priority_db_P0@1128
         4       20             31     1   148     2  /data/user/0/com.ss.android.ugc.aweme/databases/OriginalSound
         4       64             23     0   261     2  /data/user/0/com.ss.android.ugc.aweme/databases/pz_database
         4        8                    0     0     0    (attached) temp
         4       64             25     4   152     2  /data/user/0/com.ss.android.ugc.aweme/databases/pz_database (3)
         4       64             45    30   162     5  /data/user/0/com.ss.android.ugc.aweme/databases/pz_database (2)
         4       44            116    58   155     9  /data/user/0/com.ss.android.ugc.aweme/databases/effect_platform_effect_db
         4       60             23     0   239     2  /data/user/0/com.ss.android.ugc.aweme/databases/bd_sync_sdk_v4.db
         4        8                    0     0     0    (attached) temp
         4       60             81    76   147     3  /data/user/0/com.ss.android.ugc.aweme/databases/bd_sync_sdk_v4.db (2)
         4       24             41     5   269     4  /data/user/0/com.ss.android.ugc.aweme/databases/deviceinfo_stats
         4        8                    0     0     0    (attached) temp
         4       24             29     3   158     2  /data/user/0/com.ss.android.ugc.aweme/databases/deviceinfo_stats (2)
         4     4168             62  1976   163     5  /data/user/0/com.ss.android.ugc.aweme/databases/applog_priority_db_P2@1128
         4       20             25     1   160     2  /data/user/0/com.ss.android.ugc.aweme/databases/DOUYIN_SCORE.db
         4       20             47    88   165     5  /data/user/0/com.ss.android.ugc.aweme/databases/video_record.db
POOL STATS
     cache hits  cache misses    cache size  Dbname
              1           151           152  /data/user/0/com.ss.android.ugc.aweme/databases/Content.db
              5           411           416  /data/user/0/com.ss.android.ugc.aweme/databases/free_flow
           8979           416          9395  /data/user/0/com.ss.android.ugc.aweme/databases/ss_app_log.db
              1           151           152  /data/user/0/com.ss.android.ugc.aweme/databases/aweme.db
              1           161           162  /data/user/0/com.ss.android.ugc.aweme/databases/verifystorage.db
           2332           162          2494  /data/user/0/com.ss.android.ugc.aweme/databases/applog_priority_db_P1@1128
              1           163           164  /data/user/0/com.ss.android.ugc.aweme/databases/TTVideoEngine_vod_strategy_database_v01
           1384           158          1542  /data/user/0/com.ss.android.ugc.aweme/databases/downloader.db
            133           151           284  /data/user/0/com.ss.android.ugc.aweme/databases/bytebench_db
           2326           162          2488  /data/user/0/com.ss.android.ugc.aweme/databases/applog_priority_db_P0@1128
              1           149           150  /data/user/0/com.ss.android.ugc.aweme/databases/OriginalSound
             25           591           616  /data/user/0/com.ss.android.ugc.aweme/databases/pz_database
             58           156           214  /data/user/0/com.ss.android.ugc.aweme/databases/effect_platform_effect_db
             78           406           484  /data/user/0/com.ss.android.ugc.aweme/databases/bd_sync_sdk_v4.db
             10           447           457  /data/user/0/com.ss.android.ugc.aweme/databases/deviceinfo_stats
           1976           164          2140  /data/user/0/com.ss.android.ugc.aweme/databases/applog_priority_db_P2@1128
              1           161           162  /data/user/0/com.ss.android.ugc.aweme/databases/DOUYIN_SCORE.db
             88           166           254  /data/user/0/com.ss.android.ugc.aweme/databases/video_record.db
```

## 各个列指标解释

### Pss

Proportional Set Size (按比例计算的实际使用的物理内存)

$$ Pss = \Sigma(每个页 * 1/共享该页的进程数)*页面大小 
    = (所有独占的页+与其他进程共享的页*对应的占比)*页面大小
$$

实际上是用分摊的想法考虑共享的页的情况下计算出的物理页帧占用量，核心是共享的物理页帧按照占比计入

* 意义 :
最接近物理内存的实际使用量
* 用于评估整体内存压力

### Rss 
Resident Set Size (按所有驻留页计算的使用的物理内存)

$$Rss = \Sigma(所有映射到物理页帧的页)*页面大小$$

相比于Pss共享页按照100%计入

* 意义 :
可以表示进程占用的物理内存的上限。


### Private Dirty

$$Private Dirty = \Sigma (仅被本进程映射的脏页)*页面大小$$

* 意义 :
所有仅被当前进程映射的内容被修改过，必须写回swap才能回收的内存。
* 是GC的主要回收对象
### Private Clean

$$Private Clean = \Sigma(仅被本进程映射且未被修改过的页)*页面大小$$
* 意义 :
进程独占但可以直接丢弃无需IO的内存

$$ PrivateDirty <= Private Total <= Pss <= Rss$$


### Swap Dirty

$$ Swap Dirty = \Sigma(已被换出到zram/swap且内容被修改过的页)*页面大小$$

* 意义 :
已释放的物理内存，重新访问需要从swap读回

### HeapSize
堆的最大可分配容量

### HeapAlloc
堆的已分配给对象的内存

### HeapFree

堆内空闲块的总和

$$HeapSize = HeapAlloc + HeapFree$$

Alloc/Size过高时将触发GC


## 各个行指标解释

### Dalvik Heap

ART管理的Java对象堆

```
Dalvik Heap Pss = 
  Σ(存活 Java 对象的内存) +
  Σ(GC 元数据: mark bits, card table) +
  Σ(分配器碎片) +
  Σ(Thread Local Allocation Buffers)
```

```
Dalvik Heap Private Dirty = Java对象实际占用
Dalvik Heap Heap Alloc = ART分配器记录的已分配字节数
Dalvik Heap Heap Free = ART空闲链表中的可分配块
```

这里Private Dirty是内核物理值，Heap Alloc是ART的逻辑值。

### Dalvik Other

这部分内存由ART直接mmap或malloc分配，不经过GC堆分配器，GC不回收它们，其生命周期与进程一致。

### .art mmap/.dex mmap/.oat mmap

[.xxx map] 文件映射的内存区域

* 映射类型: 内容来自磁盘文件
* 共享性: 写时复制，初始可共享
* 初始状态: 与文件保持一致
* 写后状态: Dirty+Private COW创建副本

```
.dex mmap Private Clean = 未被执行的 DEX 代码页（可共享）
.dex mmap Private Dirty = 被 JIT 修改过的代码页（COW 后私有）
.oat mmap Private Clean = AOT 编译的机器码（只读，可共享）
```


### Native Heap

$$Native Heap = JNI/Bitmap/图形等非GC管理的native内存$$


### Ashmem/EGL mtrack/GL mtrack

主要是Android 图形/共享内存子系统
