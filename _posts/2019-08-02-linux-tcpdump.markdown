---
title: "TCPDUMP 사용방법"
date: 2019-08-02
tags: Linux
---

tcpdump는 서버에서 송수신되는 패킷을 캡쳐하는 프로그램이다. 많은 네트워크 모니터링 도구가 있지만 가볍게 서버간 패킷이 원활하게 주고받는지 확인하기 위해 tcpdump를 많이 사용한다.
  

eth0 인터페이스 TCP 80포트 덤프
```bash
tcpdump -i eth0 tcp port 80
```
  

Source IP를 기준으로 패킷 덤프
```bash
tcpdump -i eth0 src 192.x.x.x
```
  
  
Destination IP를 기준으로 패킷 덤프
```bash
tcpdump -i eth0 dst 192.x.x.x
```
  

and 옵션을 사용하여 여러가지 조건 조합
```bash
tcpdump -i eth0 dst 192.x.x.x and tcp and src 192.x.x.x
```
  

*-w*,*-r* 옵션 사용으로 덤프내용을 파일로 저장하고 읽기 
```bash
tcpdump -w dumplog.log -i eth0 dst 192.x.x.x
tcpdump -r dumplog.log
```

그외 옵션  
[https://www.tcpdump.org/manpages/tcpdump.1.html](https://www.tcpdump.org/manpages/tcpdump.1.html)