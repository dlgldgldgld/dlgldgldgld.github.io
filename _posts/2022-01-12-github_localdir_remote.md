---
layout: single
title:  "현재 파일 Directory를 Github Project에 올려보기"
category: Github
tag: Github
---

1. Commit 하고싶은 Local 경로로 들어가서 `git init` 으로 원격 저장소 설정.
2. `gh repo create` 실행 후, **Push an existing local repository to Github** 선택.
    1. remote는 Yes Check시 git remote add origin “주소” 작업은 하지 않아도 됨.
        1. git remote add origin “레포짓 주소” ⇒ origin 이라는 이름으로 원격 저장소가 등록되었다는 뜻. origin 말고 다른것을 해도 됨.
3. `git add . && git commit -m “initial_commit”`  명령어로 commit 작업 진행
4. `git push origin master`  을 하면 Git에 upload 완료.