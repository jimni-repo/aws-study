# ELB (Elastic Load Balancer) 이란

![elb-basic](images/elb-basic.png)

수신되는 여러 트레픽을 IP, 인스턴스, Lambda, 또다른 Application Load Balancer 로 분산하는 시스템입니다. 로드 밸런서는 클라이언트에서 오는 트래픽을 하나 이상의 가용영역에 등록된 대상으로 라우팅합니다.

- 헬스 체크를 통해 풀 내 비정상 인스턴스를 감지하고 비정상 인스턴스가 복원될 때 까지 정상 인스턴스로 라우팅
- 단일 AZ 또는 여러 AZ 영역에서 사용 가능(Cross-Zone Load Balancing)
- 동일한 클라이언트에게 동일한 대상으로 라우팅하는 고정 세션(Sticky Session) 지원
- 사용자와 인스턴스가 암호화 통신을 해야하는 경우 인스턴스의 붇담을 줄이기 위해 ELB 가 대신 암호화 통신을 수행(SSL Offload)

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

### 규칙(Rule)

### 대상 그룹 (Target Group)

### 유휴 제한 시간(Connection Time Out)