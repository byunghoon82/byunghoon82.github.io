---
title: "Splunk 설치 방법"
date: 2019-08-13
tags: SIEM
---

Splunk는 여러 장비, 애플리케이션, 서버 등에서 제공하는 이벤트 및 데이터를 하나의 인터페이스에서 확인할 수 있는 플랫폼이다. 서버와 연동하여 서버 리소스 및 상태 모니터링, 애플리케이션과 연동하여 애플리케이션에서 제공하는 정형화된 데이터를 사용자에게 시각적으로 제공한다.

설치는 다음과 같은 방법으로 가능하다.

1. Splunk 설치  

- Downlod: Splunk Enterprise 7.2.3 Download(https://docs.splunk.com/Documentation/Splunk/7.2.3/Installation/InstallonLinux)
- Test Platform: Centos 7.x
- Command:
```bash
rpm -i --replacepkgs --prefix=/splunkdirectory/ splunk_package_name.rpm
cd /splunkdirctory/bin
./splunk start --accept-license
./splunk enable boot-start (optional)
```
- 설치완료 후 8000포트로 접속
```html
http<s>://<splunk_address>:8000
```


2. Splunk 데이터 입력 설정  
다른 애플리케이션에서 데이터를 받기위한 설정이다.

a. 설정 > 데이터 > TCP > 새 로컬 TCP
![Alt text](/assets/post_images/splunk/splunk_1_1.png)
![Alt text](/assets/post_images/splunk/splunk_1_2.png)

b. 위와 같이 생성하면 splunk 서버에 514번 포트가 오픈된다.
```bash
netstat -nlpt
tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      19513/splunkd
```

3. 애플리케이션에서 514번 포트로 이벤트 및 데이터가 전송되도록 설정한다.

4. splunk에서 결과 확인
![Alt text](/assets/post_images/splunk/splunk_1_3.png)

