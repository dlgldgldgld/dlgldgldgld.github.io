---
layout: single
title:  "자주쓰는 Linux 명령어 정리"
category: Linux
tag: Linux
---

# 자주쓰는 Linux 명령어

- `man` : manual의 약자. 명령어들이 어떤게 있는지 조회하는 명령어. 명령어의 상세 정보에 대해서 알려줌
	- `man ls` : ls 명령어의 사용방법들을 알려줌.\
	- `"tool" --help` : 명령어도 사용법을 알려주나, man 보다는 자세하지 않음.
- `pwd` : 현재 directory의 위치
- `cd` : directory로 이동
	- `cd ~` : home directory로 이동함 ( home/사용자 )
		- 그냥 `cd` 를 쳐도 동일하게 home directory로 감
	- `cd -` : 바로 이전의 directory로 이동
- `ls` : 현재 디렉토리에 있는 파일 목록 표시.
	- `ls -al` : 상세 내용에 대해서 표시.
- `cat "filename"` : file에 들어있는 내용을 출력.
- `head "filename"` : file의 시작부터 일부만 bash에 출력.
- `tail "filename"` : file의 끝부터 위쪽으로 일부의 내용만 bash에 출력.
- `more or less "filename"` : file의 위쪽부터 일부분만 표시. enter로 다음줄로 넘김.
- `grep` : 각 파일에서 패턴에 일치하는 행들을 찾음.
	- `grep -i "hello world" menu.c main.c` : menu.c, main.c에서 'hello world'를 찾음.
	- `ls -al | grep "kern.log"` : ls -al의 결과물에서 "kern.log"라는 text를 찾는 명령어, 이런식으로 찾아보는 경우가 많다고함.
	- `grep "startup packages confugre" dkpg.log | tail` : grep의 결과물에서 tail tool을 사용해 console에 출력.
	- `cat dpkg.log | grep "2022-01-07 15:39:32"` : cat 명령어의 결과물에서 "2022-01-07 15:39:32" text를 찾음.