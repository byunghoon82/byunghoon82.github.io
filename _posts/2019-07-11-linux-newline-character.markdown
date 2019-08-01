---
layout: post
comments: true
title: "vi 개행문자 삭제 방법"
date: 2019-07-11
tags: Linux
---

목적: 윈도우에서 Unix 계열로 텍스트 파일 전송 시 엔터가 있던 자리에 ^M이 표시된다. 윈도우는 \n\r이고, Unix는 \n 이기때문에 \r이 남아서 발생하는 문제이다.

### vi 편집기를 이용한 제거 방법
- vi를 binary 편집 모드로 실행
```bash 
# vi -b 파일명 
```
- vi 명령 줄에 다음을 입력 후 엔터
```
%s/^M//g
```
  - ^ : ctrl + v
  - M : ctrl + M


- 저장 후 vi 종료



### 명령어를 이용한 제거 방법
```bash
# perl -pi -e 's/\r//g' [FileName] 
```