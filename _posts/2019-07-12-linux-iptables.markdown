---
layout: post
comments: true
title: "IPTABLES 설정 방법"
date: 2019-07-12
tags: Linux
---

### 설정 방법
  

기본 IPTALBE 초기화 방법
```bash
iptables -F
iptables -X
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
```
  
  

루프백 허용
```bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT
```
  
  

외부 > 서버로 22번 포트 허용
```bash
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT
```
  
  

서버 > 외부로 22포트 허용
```bash
iptables -A INPUT -p tcp --sport 22 -j ACCEPT
iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT
```
  
  

외부 > 서버로 80번 포트 허용
```bash
iptables -A INPUT -p tcp  --dport 80 -j ACCEPT
iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
```
  
  

DNS와의 통신이 허용 되어야만 외부망 접속이 가능
```bash
iptables -A INPUT -p udp --sport 53 -j ACCEPT
iptables -A OUTPUT -p udp  --dport 53 -j ACCEPT
```
  
  

외부 > 서버로 5901번 포트 허용
```bash
iptables -A INPUT  -p tcp -m tcp --dport 5901 -j ACCEPT
iptables -A OUTPUT -p tcp -m tcp --sport 5901 -j ACCEPT
```
  
  

scp를 이용하여 포트를 오픈할때 로그를 받는 서버에서 실행
```bash
iptables -A INPUT  -p tcp -m tcp --sport 2203 -j ACCEPT
iptables -A OUTPUT - -p tcp -m tcp --dport 2203 -j ACCEPT
```
  
  

변경 규칙 저장
```bash
/etc/init.d/iptables save
```
  
  

규칙삭제
```bash
iptables -D INPUT 1 (몇번째 규칙 삭제)
```
  
  

규칙확인
```bash
iptables -L -v
```
  
  

### PREROUTING / POSTROUTING 설정 방법
  

PREROUTING은 패킷에서 목적지 IP를 변경하여 패킷을 특정 IP로 포워딩 할때 사용하며 내 자신의 PC에서 OUTPUT되는 패킷을 특정 PC 로 포워딩 하고 싶을때 사용한다.  
POSTROUTING은 패킷에서 보낸이 IP를 특정(사설/공인) IP로 변경하고 재전송한다.
  
  

먼저 FORWARDING 설정
```bash
sysctl -w net.ipv4.ip_forward=1
sysctl -p /etc/sysctl.conf
```
  
  

아래 예제는 A서버에 APPLICATION이 운영되고 B서버에서 IPTABLES를 통해 포워딩이 이루어지고 불특정 다수의 사용자는 B서버를 통해 A서버로 접근할때 필요로 하는 설정으로
PREROUTING에서는 불특정 사용자가 B라는 서버에 접근할때 목적지 IP를 A서버로 변경하였고 POSTROUTING을 통해 나가는 패킷을 MASQUERAD (불특정 다수)하였다.
```bash
iptables -A PREROUTING -t nat -p tcp  -d 호스트서버의아이피  --dport 포트번호 -j DNAT --to 포워딩내부아이피:포트번호
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
  
  

정책 설정 FORWARD는 모드 ACCEPT
```bash
iptables -A FORWARD -i ens1 -j ACCEPT
iptables -A FORWARD -o ens1 -j ACCEPT
```
  
  

-i en3로 인/아웃되고 목적지가 -d 192.168.0.93인 패킷을 192.168.0.57로 포워딩
```bash
iptables -t nat -A PREROUTING -p tcp -i ens3 -d 192.168.0.93 -j DNAT --to 192.168.0.57
```
  
  
출발지가 -s destination.com인 패킷을 192.168.0.57로 포워딩
```bash
iptables -t nat -s destination.com -A PREROUTING -p tcp -j DNAT --to 192.168.0.57
```
  
  

-o eth0로 나가는 패킷이 내부망에서 인터넷 외부로 전송될때 MASQUERADE 옵션 설정, MASQURADE는 사설 IP로 공인 IP로 변경하는 옵션
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
  
  

### PREROUTING / POSTROUTING 관련 명령어
  

nat 테이블 등록
```bash
iptables -t nat -A POSTROUTING -s 192.168.2.0/24 -o eth0 -j MASQUERADE
```
  
  

nat 리스트보기
```bash
iptables -t nat -L -n
```
  
  

nat 테이블 모두 삭제
```bash
iptables -t nat -F
```
  
  

nat 테이블 2번째 POSTROUTING 정책 삭제
```bash
iptables -t nat -D POSTROUTING 2
```
  
  

iptables 저장
```bash
service iptables save 
```