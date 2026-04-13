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

## 로드 밸런서 기능

### 부하 분산 처리
