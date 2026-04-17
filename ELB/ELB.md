# ELB (Elastic Load Balancer) 이란

![elb-basic](images/elb-basic.png)

수신되는 여러 트레픽을 IP, 인스턴스, Lambda, 또다른 Application Load Balancer 로 분산하는 시스템입니다. 로드 밸런서는 클라이언트에서 오는 트래픽을 하나 이상의 가용영역에 등록된 대상으로 라우팅합니다.

- 헬스 체크를 통해 풀 내 비정상 인스턴스를 감지하고 비정상 인스턴스가 복원될 때 까지 정상 인스턴스로 라우팅
- 단일 AZ 또는 여러 AZ 영역에서 사용 가능(Cross-Zone Load Balancing)
- 동일한 클라이언트에게 동일한 대상으로 라우팅하는 고정 세션(Sticky Session) 지원
- 사용자와 인스턴스가 암호화 통신을 해야하는 경우 인스턴스의 붇담을 줄이기 위해 ELB 가 대신 암호화 통신을 수행(SSL Offload)

## 가용 영역 및 로드 밸런서 노드
로드 밸런서에서 가용 영역을 활성화 하면 ELB 가 해당 가용 영역에 로드 밸런서 노드를 생성합니다. 등록된 모든 가용 영역의 로드 밸런서가 트래픽을 수신하는 것이 아니라 활성화 된 가용 영역의 로드 밸런서만 라우팅됩니다.

### Cross-Zone Load Balancing - 교차 영역 로드 밸런싱
교차 영역 로드 밸런싱을 활성화 하면 각 로드 밸런서 노드가 활성화된 모든 가용 영역에 있는 대상에게 트래픽을 분산합니다. Application Load Balancer 는 기본적으로 이 기능이 항상 사용되지만 Network Load Balancer 는 비활성화 됩니다. 또한 이 기능은 생성 후 활성화 하거나 비활성화 할 수 있습니다.

- 교차 영역 로드 밸런싱이 활성화된 경우

    ![elb-cross-zone-enabled](images/cross_zone_load_balancing_enabled.png)

- 교차 영역 로드 밸런싱이 비활성화된 경우

    ![cross_zone_load_balancing_disabled](images/cross_zone_load_balancing_disabled.png)

### 영역 전환
영역 전환은 Amazon Application Recovery Controller(ARC) 내의 기능입니다. 이 기능으로 손상된 가용 영역에서 로드 밸런서 리소스를 다른 곳으로 이동할 수 있습니다. 이 기능을 통해 AWS 리전의 다른 정상 가용 영역에서 계속 운영하도록 구성 가능합니다.

## 라우팅 요청
클라이언트는 로드 밸런서에 요청을 보내기 전에 먼저 DNS 로 부터 로드 밸런서 노드의 IP 를 반환 받습니다. Network Load Balancer 를 생성할 때 필요에 따라 Elastic IP 주소 하나를 네트워크 인터페이스에 연결할 수 있습니다.

### 라우팅 알고리즘

#### Application Load Balancer
먼저 적용할 규칙을 결정하기 위해 우선순위에 따라 리스터 규칙을 평가합니다. 그리고 설정된 규칙에 따라 대상 그룹을 선택합니다. 기본 라우팅은 라운드 로빈입니다.

#### Network Load Balancer
흐름 해쉬 알고리즘을 사용하여 기본 규칙에 대한 대상 그룹을 선택합니다. 알고리즘은 다음을 기반으로 동작합니다.

- 프로토콜
- 소스 IP 주소 및 소스 포트
- 대상 IP 주소 및 대상 포트
- TCP 시퀀스 번호

각 개별 TCP 연결은 수명 동안 하나의 대상에 라우팅됩니다. 클라이언트로부터의 TCP 연결은 소스 포트와 시퀀스가 다르므로 다르게 인식됩니다.

### HTTP 연결
Application Load Balancer 는 연결 멀티플랙싱을 사용합니다. 여러 프런트 엔드 연결에 있는 여러 요청을 단일 백엔드 연결을 통해 라우팅될 수 있습니다. 이 기능을 사용하지 않으려면 HTTP 응답에 `HTTP Connection: close` 헤더를 설정하여 keep-alive 를 비활성화 할 수 있습니다.

Application Load Balancer 는 프런트 엔드 연결에서 파이프라인 HTTP 를 지원합니다. 단, 백엔드 연결에서는 지원하지 않습니다.

Application Load Balancer 는 `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `OPTIONS`, `PATCH` 요청 메서드를 지원합니다.

Application Load Balancer 는 프런트 엔드 연결에서 HTTP/0.9, HTTP/1.0, HTTP/1.1, HTTP/2 등의 프로토콜을 지원합니다. HTTPS 리스너에서만 HTTP/2 를 사용할수 있고 하나의 HTTP/2 연결을 통해 최대 128개의 요청을 동시에 전송할 수있습니다. 또한 HTTP에서 WebSocket 으로 연결을 업그레이드 할 수도 있습니다. 단, 이 경우 리스너 및 AWS WAF 통합은 더 이상 적용되지 않습니다. 백엔드 연결에서는 기본적으로 HTTP/1.1 을 사용하며, HTTP/2 또는 gRPC 를 사용해 요청을 보낼 수도 있습니다.

## 로드 밸런서 체계
로드 밸런서를 생성할 때 밸런서를 내부 또는 인터넷 경계 로드 밸런서로 생성할지 여부를 선택할 수 있습니다.

![lb-scheme](images/lb-scheme.png)

인터넷 경계 로드 밸런서의 노드는 Public IP 주소를 가지며 DNS 이름으로 퍼블릭 IP 로 접근 가능합니다. 이 구성으로 인터넷을 통한 클라이언트 요청을 라우팅할 수 있습니다. 내부 로드 밸런서는 오직 Private IP 만 가지며 오직 로드 밸런서를 위한 VPC 에 엑세스하여 라우팅할 수 있습니다. 두 체계 모두 Private IP 주소를 사용하여 라우팅합니다. 

### 온프레미스 vs AWS 아키첵쳐 매칭
일반적인 3-Tier 아키텍쳐(Web-WAS-DB)를 AWS 환경으로 구현한 모델로 매칭해보면 다음과 같습니다.

|     온프레미스     |      AWS      |
|:-------------:|:-------------:|
|   외부망(DMZ)    | Public Subnet |
|   외부 로드 밸런서   | 인터넷 경계 로드 밸런서 |
| 내부망(Internal) | Priave Subnet |
|   내부 로드 밸런서   |   내부 로드 밸런서   |
|  방화벽(L3/L4)   | 보안 그룹 / NACL  |

이 구성으로 별도의 Web 서버 없이도 ALB 를 통해 직접 WAS 로 라우팅할 수 있는 구성이 가능해 집니다. ALB 자체가 L7 레이어의 기능을 수행하기 때문에, 온프레미스에서 Apache 나 Nginx 가 하던 Proxy 역할을 ALB 가 대신할 수 있습니다.

## 로드 밸런서에 대한 네트워크 MTU
최대 전송 단위(MTU) 는 네트워크를 통해 전송할 수 있는 최대 패킷의 크기(바이트)를 결정합니다. 인터넷 게이트웨이를 통해 전송되는 트래픽에 대한 MTU 가 1500 MTU 이기 때문에 로드 밸런서 또한 인터넷 게이트웨이로 들어오는 요청에 대해서는 이 MTU 에 종속됩니다.

로드 밸런서의 MTU 는 설정할 수 없으며, 각 로드 밸런서는 Jumbo frames (9001 MTU) 가 표준입니다.

## 제품 비교

![elb-choice](images/elb-choice.png)

> [!WARNING]
> Classic Load Balancer 는 초기에 나온 서비스로 현재 deprecated 된 상태 입니다.

애플리케이션 요구사항에 따라 적절한 로드 밸런서를 선택할 수 있습니다. 애플리케이션 레벨에서 유연한 구성이 필요한 경우 Application Load Balancer를 사용하는 것이 좋고, 성능 및 고정 IP 가 필요한 경우 Network Load Balancer를 사용하는 것이 좋습니다.

| 기능           | Application Load Balancer |        Network Load Balancer        |  Gateway Load Balancer   |
|--------------|:-------------------------:|:-----------------------------------:|:------------------------:|
| 로드 밸런서 유형    |           계층 7            |                계층 7                 | 계층 3 게이트웨이 + 계층 4 로드 밸런싱 |
| 대상 유형        |     IP, 인스턴스, Lambda      | IP, 인스턴스, Application Load Balancer |         IP, 인스턴스         |
| 흐름/프록시 동작 종료 |             예             |                  예                  |           아니요            |
| 프로토콜 리스너     |     HTTP, HTTPS, gRPC     |            TCP, UDP, TLS            |            IP            |
| 다음을 통해 연결 가능 |            VIP            |                 VIP                 |        라우팅 테이블 항목        |

## ELB 구성 요소
ELB 는 VPC 에 탑재되며, 사용자 요청을 받고 이를 VPC 내의 인스턴스로 부하 분산합니다. ELB 는 외부의 요청을 받아들이는 리스너(Listener)와 요청을 분산/전달할 리소스 집합인 대상 그룹(Target Group)을 구성됩니다. ELB 는 1개 이상의 리스너와 대상그룹을 가질 수 있습니다. 일반적인 ALB 를 통한 서비스 구성은 아래와 같습니다.

![elb-internal](images/elb-internal.png)

### 리스너(Listener)
리스너는 프로토콜과 포트 기반으로 클라이언트의 요청을 설정된 대상으로 라우팅하는 기능을 수행합니다. 로드 밸런서에는 최소 1개 이상의 리스너가 필요하며, 최대 10개 까지 설정 가능합니다.

### 규칙(Rule)

### 대상 그룹 (Target Group)

### 유휴 제한 시간(Connection Time Out)


---
참고
1. https://inpa.tistory.com/entry/AWS-%F0%9F%93%9A-ELB-Elastic-Load-Balancer-%EA%B0%9C%EB%85%90-%EC%9B%90%EB%A6%AC-%EA%B5%AC%EC%B6%95-%EC%84%B8%ED%8C%85-CLB-ALB-NLB-GLB
2. https://docs.aws.amazon.com/ko_kr/elasticloadbalancing/latest/userguide/what-is-load-balancing.html