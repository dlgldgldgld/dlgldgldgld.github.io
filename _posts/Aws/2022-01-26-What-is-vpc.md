---
layout: single
title:  "VPC란?"
category: Aws
tag: Aws
---

# before read
> 해당 글에는 개인적인 생각이 들어가 있습니다. 틀린 내용이 있을수도 있으니 이에 대해서 양해 부탁드립니다.
<br>

----
<br>


# VPC란?
- VPC는 Virtual Private Cloud의 약자로 AWS 상에 존재하는 가상 네트워크를 뜻한다. 
- <u>"이것이 왜 필요한가?"</u> 에 대해서 생각해 볼 수 있다. AWS Instacne는 각각의 고유한 Private IP를 가지고 있다. VPC가 없던 시절 AWS 사용자들은 이 IP들을 하나로 묶어서 별도의 Network 망을 구축할수가 없었다고 한다. 
- "Network 망이 있든 없든 무슨 상관이냐?" 라고 궁금증을 가질 수 있다. 생각해보자, 내가 생성한 모든 Instance들에 대해 특정 트래픽은 차단하거나 외부 인터넷에서  접속이 불가능 하도록 만들고 싶다고 가정하자. 이를 위해서는 어떻게 해야 할까? 각각의 Instance 들마다 트래픽을 차단을 설정해야 될 것이고, 각 PC마다 설정된 Public IP를 모두 제거 해주는 등 관리하기 귀찮은 상황이 생기게 될 것이다. 
- 위 같은 상황을 효율적으로 관리하기 위해서 AWS에서 제공해주는 것이 바로 VPC이다. VPC는 AWS 사용자에게 실제로는 존재하지 않는 가상의 Network망을 구축해서 내부에 속한 Instance들의 관리를 편리하게 할 수 있도록 도와준다. <u>즉, AWS라는 넓은 Cloud의 세계에서 가상의 개인적인 Cloud 세계 ( Virtual Private Cloud ) 를 구축할 수 있도록 도와주는 것이 VPC</u> 이다. 


# VPC 구성하기
VPC는 크게 4가지 방식으로 구성된다. 
1. VPC 내부에서 사용될 IP 주소의 범위를 정하기.
2. 가용 영역(AZ)별 서브넷 설정하기.
3. 인터넷으로 향하는 라우트 경로 만들기.
4. 트래픽 Control을 위한 VPC 보안 설정 만들기.

각각의 항목에 대해서 자세하게 알아보자.

## 1. VPC의 IP 주소 범위 설정 
- AWS에서는 Private IP Address를 <sup>[[1]](#cidr)</sup> CIDR(Classless Inter-Domain Routing) 방식으로 구성한다.
- VPC IP 주소는 반드시 <sup>[[2]](#rfc1918) </sup> RFC1918 사설망 범위에 포함시키도록 한다.
  - 이는 AWS Public IP와 Private IP의 Overlapping을 방지하기 위해서라고 한다.
   
## 2. 가용 영역 ( Availability Zone ) 별 서브넷 설정
- VPC에서 특정 범위의 IP 주소를 사용하도록 설정을 했다면 또 이것을 그룹화 시키기 위해서 서브넷이라는 것을 사용한다.
- 이 서브넷은 Region내에 있는 <sup>[[3]](#az) </sup>가용영역 ( Availability Zone ) 내부에만  구성을 할 수 있다.
- 서브넷들은 VPC 내에서 Overlapping 되지 말아야 하기 떄문에 서로 다른 IP 범위를 설정해야 한다.
  - 만약 VPC IP 할당량이 172.56.0.0/16 이고 이곳에 2개의 서브넷을 구성한다면 하나는 172.56.`0`.0/24, 다른 하나는 172.56.`1`.0/24로 구성해야 한다는 의미이다.
- 서브넷은 /24 ( 251 address ) 만큼만 사용할 것을 권장한다.
  - /24 라서 8개의 bit를 통해 256개의 address를 사용 가능해야 하나 5개의 ip address는 대표주소, 브로드캐스트, aws 내부 사용 ip 3개가 이미 할당되어 있어 251개만 사용이 가능하다. 
    ![alt](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/nat-gateway-diagram.png)
    <br>
    출처 : https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/nat-gateway-diagram.png
    
## 3. 라우팅 테이블 만들기


<!--
## 4. VPC 트래픽 설정
## NAT 게이트 웨이 ( Network Address Translation )
-->

<br>
<br>
<br>
<br>

----

<a name="cidr">[1]</a>: CIDR 방식은 "192.168.0.0/16"와 같이 표현하며 의미는 11000000(192).10101000(168).00000000(0).00000000(0) IP에서 상위 16 비트는 고정해서 Network ID로 구성하고, 하위 16 비트는 HostID로 구성을 하겠다는 의미이다. 즉, "192.168.123.1" 이나 "192.168.111.31" 은 모두 "192.168.0.0" 으로 간주해서 네트워크를 group화 시키겠다는 의미이다.

<a name="rfc1918">[2]</a>: IPv4 범위 = 10.0.0.0~10.255.255.255 (10.0.0.0/8), 172.16.0.0~172.31.255.255 (172.16.0.0/12), 	192.168.0.0~192.168.255.255 (192.168.0.0/16)

<a name="az">[3]</a>: 가용 영역(AZ)이란 Availability Zone의 약자로 Region별로 2~3개 정도 구성된다. 이것의 역할을 고가용성을 위해 데이터 센터를 여러곳에 배치해두고 자연재해 등 서버가 물리적으로 파괴되었을때를 대비해서 Data가 백업될 수 있도록 운영하는 AWS의 방식이다.