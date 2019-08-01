---
title: "RSA 공개 키 만들기"
date: 2019-05-15
tags: Linux
---

### 사용방법
- A 서버에서 키를 생성하여 B 서버로 카피
- A 서버에서 B로 접근할때 PASSWORD 입력 없이 접근


### A 서버에서 공개 키 생성방법
```bash
$ ssh-keygen -t rsa 

Generating public/private rsa key pair. 
Enter file in which to save the key (/root/.ssh/id_rsa): <ENTER> 
Enter passphrase (empty for no passphrase): <ENTER> 
Enter same passphrase again: <ENTER> 
Your identification has been saved in /root/.ssh/id_rsa. 
Your public key has been saved in /root/.ssh/id_rsa.pub. 
The key fingerprint is: 
b7:95:f7:a0:e1:52:01:d5:ec:48:e3:73:f7:45:40:46 root@machineA 


$ cd ~/.ssh
$ ls -l
total 32 
-rw------- 1 root root 883 Nov 07 11:41 id_rsa 
-rw-r--r-- 1 root root 222 Nov 07 11:41 id_rsa.pub 
-rw-r--r-- 1 root root 915 Nov 06 12:30 known_hosts 
```

생성된 id_rsa.pub를 B서버의 .ssh 경로로 카피

### B 서버 설정 방법
```bash
$ cd ~/.ssh
$ mv id_rsa.pub authorized_keys
$ ls -l
total 32 
-rw------- 1 root root 883 Nov 07 11:41 authorized_keys 
-rw-r--r-- 1 root root 915 Nov 06 12:30 known_hosts 
```