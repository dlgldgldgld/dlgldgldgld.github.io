---
layout: single
title:  "WSL2 웹서버를 Local IP에 매칭해서 사용해보기"
category: Window
tag: [Window, WSL]
---

# 서론
## 글을 쓰게된 이유
> 최근 AWS, k8s 공부를 하면서 네트워크 쪽에 너무 무지하다는 생각이 들었다. 물론 대학생때도 한번 공부를 했으나 내 기억력은 그렇게 좋은 편이 아니라 모두 reset 되버린 상태다. 그래서 이를 보완하고자 네트워크 기초를 다시 공부하게 되었고 어느덧 OSI Layer 4 까지 공부하게 되었다. 
> 강의를 들으면서 문득 이런 생각이 들었다. 
> "WSL2를 Local Airflow 웹서버로 사용하고 있는데, 내 IP를 WSL2와 연결하고 포트포워딩을 하면 외부에서도 Airflow를 사용할 수 있지 않을까?" 
> 다행히 이에 대해서도 해법이 존재하였고, 오늘은 이 주제에 대해서 글을 써보고자 한다.
> 추가로 다른 사람들이 올린 글에서는 PowerShell Script에 대한 설명은 나와있지 않았는데 이에 대해서도 간략하게 정리해보고자 한다.


## 이 글이 필요한 분들
1. WSL2를 Local 웹서버로 사용 중이나 이를 내부 IP에 Mapping 하고자 하시는 분.
2. PowerShell Script를 통해 방화벽 Inbound, Outbound를 추가 제거하고자 하시는 분.


# Mapping WSL2 Webserver to Local IP
## 배경 설명
나는 WSL2를 Airflow Webserver용으로 사용하고 있었다. 

WSL 웹서버는 `localhost:port` 주소를 통해 접속이 가능했다. 여기에 대해서 다음과 같이 생각을 해보았다. 

"`localhost`는 나 자신을 가리키는건데, WSL2도 결국에는 Window OS가 깔린 내 PC와 동일한 IP를 가지게 되는가보다. 그래서 접속이 되나보다. :) "

그래서 "`ipconfig`에서 내 ip를 찾고, WSL에서 port 8000을 사용했으니깐 `my_ip:8000`으로 접속 하는게 가능하겠다" 라는 결론에 다다르게 되었다.

허나 이 전제는 보기좋게 빗나갔는데 왜 그런지 살펴보자.

WSL2는 일반 Ehternet이 아닌 vEthernet이라는 **Hypver-V의 Virtual Switch 구동**이 된다. ( [WSL2 변경 사항](https://docs.microsoft.com/ko-kr/windows/wsl/compare-versions) 참조 )

잘 생각해보면 WSL도 또 하나의 다른 컴퓨터라서 이를 사용하기 위한 가상의 네트워크가 필요할 수 밖에 없다. 그래서 내 IP는 사실 WSL과 연결이 되어 있는 것이 전혀 아니였다.

그럼 Window Web Application에서 localhost를 통해 바로 접속이 되는 것은 왜 그런 것이였던 걸까? 
이에 대한 답은
[WSL-네트워킹 고려사항](https://docs.microsoft.com/ko-kr/windows/wsl/networking) 에서 찾아 볼 수 있었는데, 기본적으로 localhost에 대해서는 wsl 웹 어플리케이션으로 자동 연결을 해준다고 한다.

그러면 도대체 어떻게 해야 내 IP를 WSL 웹서버에 연결을 할 수 있을까..? 이리저리 구글링을 해본결과 아래의 방법으로 적용이 가능하다는 것을 알 수 있었다.



## 적용 방법
결론적으로 아래 Powershell 스크립트를 수행하면 된다. 각 과정에서 어떤 작업이 진행되는지 살펴보자. Linux 내부 ip는 매번 새로 업데이트되기 때문에 해당 script를 시작프로그램으로 등록하는 것이 좋다.

```powershell
$remoteport = bash.exe -c "ifconfig eth0 | grep 'inet '"
$found = $remoteport -match '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}';

if( $found ){
$remoteport = $matches[0];
} else{
echo "The Script Exited, the ip address of WSL 2 cannot be found";
exit;
}

#[Ports]
#All the ports you want to forward separated by coma
$ports=@(22,80,3306,10000,3000,5000);

#[Static ip]
#You can change the addr to your ip config to listen to a specific address
$addr='0.0.0.0';
$ports_a = $ports -join ",";

#Remove Firewall Exception Rules
iex "Remove-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' ";

#adding Exception Rules for inbound and outbound Rules
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Outbound -LocalPort $ports_a -Action Allow -Protocol TCP";
iex "New-NetFireWallRule -DisplayName 'WSL 2 Firewall Unlock' -Direction Inbound -LocalPort $ports_a -Action Allow -Protocol TCP";

for( $i = 0; $i -lt $ports.length; $i++ ){
$port = $ports[$i];
iex "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$addr";
iex "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$addr connectport=$port connectaddress=$remoteport";
}
```
[출처] : https://github.com/microsoft/WSL/issues/4150#issuecomment-504209723

### Line별 설명

1. LINE 1 : bash.exe는 wsl의 bash를 의미한다. wsl 내부 ip를 가져오기 위해 `ifconfig eth0 | grep 'inet '`을 수행한다.
   - 이 내부 ip를 $remoteport에 저장한다. 
   - 결과값은 `inet 'linux-ip' netmask 'subnet-mask' broadcast 'linux broadcast ip'` 이다.
2. LINE 2 ~ LINE 9: $remoteport에 저장된 값 중 inet에 있는 linux-ip 값을 가져오기 위해 수행한다. `-match` 명령어는 정규식에 매칭되는 값을 `matches` 변수에 저장한다. 성공적으로 찾았다면 `-match` 명령어는 True를 반환한다.
3. LINE 13 : foward 하고싶은 포트들을 명시하는 곳이다.
4. LINE 17 : 공유기로 부터 할당받은 PC의 Private ip를 입력한다. 0.0.0.0으로 해도 동작한다.
5. LINE 18 : $ports에 있는 배열을 "," seperator를 사용해서 나열한다.
   - string "22,80,3306,10000,3000,5000"이 된다.
6. LINE 21 : 'WSL 2 Firewall Unlock' 이라는 명칭의 방화벽을 제거하는 명령어다.
   - 해당 명령어를 수행한 후에 방화벽을 확인해보면 실제로 제거되어 있다.
7. LINE 24, 25 : 방화벽을 추가하는 명령어다. `-Driection`은 Inbound, Outbound를 뜻하는 것이고 프로토콜 및 Port 들을 설정할 수 있다.
8. LINE 29, 30 : `netsh interface portproxy` 명령어는 내 IP와 Linux 가상 IP를 포워드 해줄 수 있게한다. Linux 가상 IP는 프로그램이 다시 시작 될 때마다 변경되기 때문에 이전에 정의해둔 것은 삭제하기 위해 delete 명령어를 수행한다. 정상수행 되었다면 `netsh interface portproxy show all`에 연결 내용이 기록되어 있다.
   - 이 방법은 여러 컴퓨터 중 한 컴퓨터만 외부에서 접속할 수 있는 상황에서 하나의 PC를 통해 여러 컴퓨터에 접속을 시키기 위해 사용하는 방식이라고 한다.



### 참조 사이트
1. Cheonghyun.com : [https://www.cheonghyun.com/blog/ko/107](https://www.cheonghyun.com/blog/ko/107)
2. github : [https://github.com/microsoft/WSL/issues/4150](https://github.com/microsoft/WSL/issues/4150)
3. 성태의 닷넷 이야기 : [https://www.sysnet.pe.kr/2/0/12347](https://www.sysnet.pe.kr/2/0/12347)
4. gmyankee's blog: [https://gmyankee.tistory.com/308](https://gmyankee.tistory.com/308)