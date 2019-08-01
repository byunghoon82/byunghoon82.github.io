---
layout: post
comments: true
title: "Splunk 설치 방법"
date: 2019-05-17
tags: SIEM
---

- 목적: SIEM (Security Information Event Management) 중 널리 사용되고 있는 도구 확인

1. Splunk를  
https://api.slack.com/slack-apps


2. 생성된 앱을 workspace에 인스톨한다.  
https://api.slack.com/apps/AGYR6MPMK/install-on-team?
![Alt text](/assets/post_images/slack/slack_1_1.png)

3. 앱을 생성하면 생성된 App의 auth token을 확인할 수 있다.
- OAuth Access Token : xoxp- 으로 시작한다.
- Bot User OAuth Access Token: xoxb- 으로 시작한다.  
![Alt text](/assets/post_images/slack/slack_1_2.png)


4. 워크스페이스에 추가하면 다음과 같이 채널에 앱이 추가된 것을 확인할 수 있다.  
![Alt text](/assets/post_images/slack/slack_1_3.png)
 

5. 앱의 bot은 하기 앱의 bot 메뉴에서 제어할 수 있다.  
![Alt text](/assets/post_images/slack/slack_1_4.png)


6. curl을 이용한 워크스페이스/채널에 파일 업로드 방법  
```bash
# curl -F file=@/file-name -F channels=#channel-name -H "Authorization: Bearer xoxb-xxxx..." https://slack.com/api/files.upload
```