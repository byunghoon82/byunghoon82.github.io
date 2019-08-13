---
title: "업데이트된 Original Repository와 Sync 실행하기"
date: 2019-07-11
tags: Git
---

Fork된 Repository에 업데이트된 Original Repository의 데이터를 반영하기 위한 방법이다.


- 먼저 Original Repository의 Branch를 확인하고 Fork된 Repository에도 동일한게 Branch를 생성한다.
  - Orignal Repository가 mater와 dev를 가지고 있다면
  - Fork Repository도 marster와 dev를 가지고 있어어야 한다.
  - 동일한 branch에 데이터가 업데이트가 된다.

- 다음을 실행한다.
```bash 
# git remote -v
# git remote add upstream https://github.com/original_owner/original_repository.git
# git remote -v
# git checkout master
# git merge upstream/master
```