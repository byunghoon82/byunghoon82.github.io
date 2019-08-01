---
layout: post
comments: true
title: "iostat Usage"
date: 2019-05-16
tags: Linux
---
  
### iostat 
디스크와 파티션을 위한 cpu 상태와 i/o 상태를 보고하는 툴이다.

| %user | 유저레벨에서 실행되고 있는 프로세스의 퍼센티지 |
| %nice | 우선순위 실행여부를 퍼센티지 |
| %sys  | 시스템 차원에서 실행되는 프로세스의 퍼센티지 |
| %idle | 휴지시간(idle time) |
| Device | Device 컬럼은 디바이스명 |
| tps | 디바이스를 초점에 맞추어 초당 전송속도의 수치 |
| Blk_read/s | 드라이브로부터 데이터를 읽어오는 양 / 초당 블록(block)의 수치 |
| Blk_write/s | 드라이브에 데이터를 쓰는 양 / 초당 블록(blocks per second)의 수치 |
| Blk_read | 읽혀진 블록의 총 수치 |
| Blk_wrtn | 쓰여진 블록의 총 수치 |


```bash
$ iostat -k -d /dev/sda -t 2 6
Linux 2.6.18-8.el5 (protexip.co.kr)     03/12/2010      _x86_64_        (2 CPU)

03/12/2010 02:02:10 PM
Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda             527.31     51297.50       126.13  798899074    1964259

03/12/2010 02:02:13 PM
Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda             612.00      4386.00     30520.00       8772      61040
```

-k: 키로바이트당  
-d: 모니터링 할 파일시스템  
-t: 초당 디스플레이  
-6: 여섯번만 출력