# NLB
joonhyun@amazon.com - Cloud Support Engineer

- CLB : 2010년 출시. 새로운 feature나 버그에 대한 지원하지 않음. 없어지지는 않을 예정. security 관련 업데이트만 ex) HTTP2 지원 안할 예정
- ALB : 2016년 출시. HTTP LB. URI 기반이고 HTTP 분석 가능
- NLB : 2017년 출시. L4에 최적화. 상위 레이어에 대한 Response 분석이 안됨

## CLB/ALB vs NLB
- 최초의 사이즈가 크지 않음.
- NLB는 IP개수에 대한 변동이 없다. 하지만 실제로 생성하자마자 수백만 Request까지는 받을 수 있다.(RequestPerSecond)
- Internal-NLB : private vpc 내의 IP 중에 하나
- NLB : 퍼블릭 IP 또는 EIP 중에 하나
- nginx, HAProxy와 다르게 upstream/downtream이 없다. latency를 마이크로세컨드 기준으로 측정하기 떄문에 좀 더 빨라야 하는 서비스에 적합

## feature
- LB시 IP를 고정해서 사용하는경우 타겟으로 사용할 수 있다.
- Client IP를 기준으로 stickiness를 할 수 있다. 올해안에 개발 예정
- CLB/ALB에서 static IP
- access log가 따로는 없다. 위의 레이어에 로그를 알 수는 없다.
- public/private nlb 생성 시 ip 변경이 없다.
- CLB에서 HTTP Listener의 밸런싱은 부하가 적은쪽으로 보내고, ALB는 라운드로빈 방식
- NLB는 Flow Hash방식을 사용한다. ALB에도 지원 예정
- metrics : Traffic, Error, Target health
- ELB 과금 : 이용시간 + LCU(로드밸런서 Capacity Unit - 커넥션 기준 또는 트래픽 기준)
- 사용할만한 서비스 : 트래픽의 급격한 변화, 이벤트
- 도중에 커넥션을 맺거나 끊는 작업이 없다. 레이턴시를 좀 더 빠르게
- NLB vs CLB 인스턴스 타입이 같다라고 했을 때 비교? 일단 두개 자체는 인스턴스 타입이 다르다.
  - ELB 자체도 EC2를 사용하고
  - 1노드 1IP
  - CLB는 리스너 타입에 따라서도 커넥션 맺는 타입이 (HTTP/TCP)다름. NLB는 아예 동작 방식이 다르다.
- 동작방식
 - NLB to Target : Destination, Port, 체크섬 헤더를 변경 - Source IP가 실제 클라이언트 IP
 - 커넥션을 맺지 않고 NAT처럼 Source, Destination만 변경해서 던져줌
- HTTP/HTTPS 통신을 NLB에서 사용 : 장점 - 고정 IP 사용, pre-warm 없이 사용 가능. 단점 - access log, cloudwatch 로그 볼 수 없음. 각 서버에서 로그를 수집해서 봐야 함. 몇개를 포기하면 쓰는데는 상관없다.
- Migration from CLB to NLB해서 생성 후에 트래픽 10%정도 넣어서 테스트 하는 방식으로 하면 될듯
- HealthCheck는 초당 10번정도..

## Pre-warm
- 최고와 최저 리퀘스트, 트래픽 발생 시간(빈도)에 따라 다름
- scale out은 급격하게 sclae in은 천천히
- scale out 시 노드(IP) 추가됨
- DNS에서 일단 제거되고 처리하던 커넥션을 처리하고 노드를 터미네이션 시킴. 강제종료를 하지는 않음
- 리퀘스트 카운트보다 전체 트래픽 사이즈를 봄.
- 미리 ELB를 만들어서 일부러 스트레스를 줘서 scale out을 한 후 사용하는 고객들도 많다.
- 5분 사이에 50%정도 트래픽으로 단계적으로 줘도 될 것 같다.
- Pre-warm 신청기간이 끝나도 노드를 강제적으로 줄이진 않는다.
- 노드 수가 늘어났는지 확인하는 방법 : (비공식적으로) dig로 IP 숫자가 늘어났는지 확인
- CLB Max Response가 튀는 경우 : Backend-time 이면 우리 문제다.

## Other
- EC2 메모리/디스크 메트릭이 기본 메트릭이 아닌 이유?
 - 하이퍼바이저 레이어에서는 얼마나 사용하는지 알 수 없음. 할당이 얼마인지는 알 수 있지만 Used는 알 수 없다.
- AWS 내부적으로도 CloudFormation을 많이 쓴다.
- ELB 오토스케일링 관리가 어떻게 되는지? 별도의 어떤것을 쓰진 않고 AWS 내부의 서비스를 사용한다.
 - ex) CloudWatch 메트릭을 가지고 AutoScailing을 쓴다던지..
 - 그 팀 보다 해당 서비스를 더 잘 만들 수는 없기 때문에 그대로 가져다 씀
- AWS에서 LB 트래픽을 가장 많이 쓰는 곳은 S3. 모든 회사를 합쳐도
- 회사에서는 기존에 빈스톡환경이라 CLB를 대부분으로 사용하고 있다. 최근에 빈스톡에서 ALB 기능이 추가됨
- SYN Packet, SYN ACK
- HTTP 통신을 사용하지 않으면 NLB 사용을 권장한다.
- ssl 접속이 안되는 케이스에서 Cloud Watch 지표에서 아무것도 확인할 수 없어서 재시작 했음.
 - 1/2 -> 인터페이스에 대한 핑
 - 2/2 -> 커널이라던지 내부적인 문제
