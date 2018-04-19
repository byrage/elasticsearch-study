# 글로벌 네트워크 인프라
- 초기 5년 : 4개 Region
- 다음 5년 : 7개 확장
- 2016~2017 : 7개
- 2018~2019(예정) : 4개

## Amazon Global Network Backbone
- 100 GbE 네트워크 이중화
- 모든 AWS 리전 간 트래픽은 Global Backbone을 이용하도록 설계
- Edge Location
 - CloudFront, Route53, Shield, WAF 서비스
- Direct Connection
 - 고객사 전용망을 연결하고 뒷단에서 사용하는 커넥션
 - 한국 : KINX 서울 가산, LG U+ 경기 평촌

# VPC와 네트워크 보안
## VPC
- Virtual Private Cloud
- 독립적인 네트워크망을 제공
- IP 할당, 서브넷 구성, 라우팅
- 보안 툴 : Security Groups & ACLs, NAT Gateway, Flow Logs
 - NAT Gateway : 프라이빗 존에서 소프트웨어 업데이트 등을 이용하는 용도
 - Flow Logs : 모든 네트워크 로그를 저장. enable 옵션 필요
### 최근 추가된 기능
- VPC 사이즈 확장 : 기존 VPC에 IP대역 (CIDRs) 추가
- 시큐리티 그룹에 설명(주석) 지원
- Default VPC : API를 통해 기본 VPC 생성
- EIP 복구 : 같은 EIP가 다시 필요하면 재제공
### Amazon Time Sync 서비스
- 외부 인터넷 연결 없이 VPC 내에서 위성, 원자시계 기반의 NTP 서비스 제공 (169.254.169.123)

## 네트워크
### 리전 내 VPC 통신
- VPC Peering
 - 요청/수락을 통해 양쪽의 VPC에 게이트웨이 채널을 제공함
 - 없다면 인터넷이나 VPN을 통해 직접 연결해야 함
 - AWS 계정이 달라도 Peering을 지원함

### 글로벌 리전 간 VPC 통신 ex) 서울 <-> 뉴욕
## Before : IPSec 기반의 VPN
- EC2에 IPSec을 구매해서 설치 및 운영 필요
- Global Backbone 활용
## After : Integer-Region Peering
- AWS 관리 서비스.
- AWS에서 이중화, 트래픽 암호화 제공
- 단점 : IP대역이 중복되면 안됨
- Transitive Routing 미지원 (1:1:1 구조가 아닌 매쉬구조로)

## VPC와 AWS Public 서비스(S3/DynamoDB)와 연동?
- Before : Private Zone - NAT Gateway를 통해 연결
- After : VPC EndPoint를 통해서 연결

### VPC EndPoint
- VPC당 1개의 엔드포인트를 가지는 듯
- VPC Route에 추가
- NAT Gateway 사용하지 않아도됨.

### Private Link
- 전용으로 통신할 수 있는 네트워크 카드(ENI) 생성
- VPC 엔드포인트 DNS로 요청해서 통신 가능
- 앞단에 NLB가 있어야 하고 엔드포인트를 서비스로 등록해야 함.
- Direct Connect 1개를 연결하면 앞단의 Endpoint ENI를 통해서 뒷단까지 통신 가능

- 뒷단 : SaaS 솔루션 사용자들
- AWS 서비스를 위한 PrivateLink : EC2 API, SNS, Kinesis Data Stream, CloudWatch Logs
 - EC2 API : 사용자가 aws-cli를 입력하면 (퍼블릭 존)에서 EC2 명령을 받아서 private zone 내의 실제 ec2 인스턴스에게 명령을 전달

## 보안
- WAF (Website Application Firewall) : SQL Injection, XSS 방어
 - 람다 같은것을 연결해서 스크립팅 가능. 어렵다면 WAF Management Ruleset을 마켓플레이스에서 구입해서 적용 가능
- AWS Shield : DDoS 방어
 - Layer 3,4에 대해서는 기본적인 AWS 서비스 모두에 방어해주고 있음. Standard
 - Advanced - EC2와 같은 레이어에서 어플리케이션 레벨까지 일어나는 공격까지 방어해줌. 전담팀의 Support 가능
- Certificate Manager : SSL / TLS 인증서 배포 및 관리. Public Service를 위한 무료 버전
 - 인증서를 쉽게 받고 갱신 가능
 - ELB나 CloudFront에 자동 배치/갱신 가능
- 유료 버전 Certificate Manager : IoT에서 적용 가능

## 로드 밸런서
- ALB, NLB, CLB

### CLB
- 원래는 그냥 Elastic Load Balancer 였는데, 다른 애들이 추가되면서 이름이 Classic이 됨 ㅜ
- ALB, NLB로 분리 되고 있다.
- TCP/HTTP/HTTPS 트래픽 처리

### ALB
- HTTP/HTTPS (Layer 7) : Path 기반의 라우팅 지원
- 컨테이너, 최신 웹 프로토콜 지원
- Multiple Certificate, Blue/Green 배포 가능
- 고급 라우팅 기능 활용 및 Multiple SNI 적용으로 로드 밸런서 통합 및 비용 절감 가능
 - SNI?

### NLB
- TCP (Layer 4)
- Availibity Zone당 1개의 각각의 로드밸런서에 고정 IP 지원
- Source IP 유지 : X-Forwarded-For 설정 x
- 오래 지속되는 커넥션
- Pre-warming이 필요 없음. 애초에 초당 1M+를 지원하기 위해 나왔기 때문에 따로 필요 없음
- 빠른 속도와 낮은 레이턴시 제공.

## 온-프리미스 네트워크와 연동 방법
### 온-프리미스 네트워크 안의 자원으로 로드 밸런싱 가능
 - ex) 기존 IDC 서버 / 마이그레이션 중인 AWS 서버

### 고객사와 VPN 연동
- Virtual Private Gateway를 직접 구현 가능

### Direct Connection
- 안전한 네트워크 제공
- 일관적인 성능
- AWS in/out 네트워크 비용을 절감
 - 기본 비용보다 전용선을 사용하는 것이 저렴함
- AWS <-> [$Kinx$] <-> 고객사
