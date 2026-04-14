# ELB (Elastic Load Balancer) 이란

- 수신되는 여러 트레픽을 IP, 인스턴스, Lambda, 또다른 Application Load Balancer 로 분산하는 시스템
- 헬스 체크를 통해 풀 내 비정상 인스턴스를 감지하고 비정상 인스턴스가 복원될 때 까지 정상 인스턴스로 라우팅
- 단일 AZ 또는 여러 AZ 영역에서 사용 가능(Cross-Zone Load Balancing)
- 동일한 클라이언트에게 동일한 대상으로 라우팅하는 고정 세션(Sticky Session) 지원
- 사용자와 인스턴스가 암호화 통신을 해야하는 경우 인스턴스의 붇담을 줄이기 위해 ELB 가 대신 암호화 통신을 수행(SSL Offload)

## 제품 비교
애플리케이션 요구사항에 따라 적절한 로드 밸런서를 선택할 수 있습니다. 애플리케이션 레벨에서 유연한 구성이 필요한 경우 Application Load Balancer를 사용하는 것이 좋고, 성능 및 고정 IP 가 필요한 경우 Network Loac Balancer를 사용하는 것이 좋습니다.

| 기능           | Application Load Balancer | Network Load Balancer               |
|--------------|---------------------------|-------------------------------------|
| 로드 밸런서 유형    | 계층 7                      | 계층 7                                |
| 대상 유형        | IP, 인스턴스, Lambda          | IP, 인스턴스, Application Load Balancer |
| 프로토콜 리스너     | HTTP, HTTPS, gRPC         | TCP, UDP, TLS                       |
| 다음을 통해 연결 가능 | VIP                       | VIP                                 |

## ELB 기능
![1](images/elb-basic.png)

부하분산은 말 그대로 부하를 분산하여 트래픽이 과도하게 하나의 인스턴스로 집중되는 현상을 막기위한 기술입니다. ELB 에서 트래픽을 하나의 경로로 받아서 설정된 다수의 인스턴스로 분산합니다. 인스턴스가 비정상적인 상태에 빠지거나 서비스 불가능한 상태가 될 경우 Health Check 를 통해 자동으로 해당 인스턴스로 라우딩하지 않게 됩니다.

또한 오토 스케일링과 조합하여 서버 시이즈를 조절해 서비스를 원할히 유지할 수 도 있습니다. 이 설정을 위해 ELB 에서는 스케일 아웃에 대한 하나의 엔드포인트를 제공합니다.

## ELB 구성 요소
ELB 는 VPC 에 탑재되며, 사용자 요청을 받고 이를 VPC 내의 인스턴스로 부하 분산합니다. ELB 는 외부의 요청을 받아들이는 리스너(Listener)와 요청을 분산/전달할 리소스 집합인 대상 그룹(Target Group)을 구성됩니다. ELB 는 1개 이상의 리스너와 대상그룹을 가질 수 있습니다.

![2](images/elb-internal.png)

