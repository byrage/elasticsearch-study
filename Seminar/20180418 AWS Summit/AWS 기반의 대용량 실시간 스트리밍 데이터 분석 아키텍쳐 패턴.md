# Data Firhorse
- 전체를 입력받고 S3, REDSHIFT, ELASTIC으로 전송하는 서비스
- 목적지로 데이터 전송

# Data Anlytics
- Streams, Firhorse에 연결해서 SQL 검색
- 다시 보내서 데이터 확인

# Data Anlytics 어플리케이션
- 스트리밍 소스에 연결
- 데이터 처리를 위해 SQL 작성
- SQL 결과 전달
```
CREATE STREAM calls_per_ip_strem (
  ...
)
```
- 인 어플리케이션 스트림 : 인 메모리 객체 (스트림에 적합)
- 스트리밍 SQL : 펌프 - 연속 쿼리

# 스트리밍 어플리케이션 패턴
1. 스트리밍 수집-변환-적재
 - 50%정도의 유즈케이스
- 지속적인 지표 생성
 - 30%
 - 리더보드 같은 데이터가 생성될 때 분석 - 사용자의 스코어를 보여줌
- 실용적인 통찰력
 - 불법행위를 실시간으로 확인

## 스트리밍 수집-변환-적재
- 적용 예시
 - S3에 로그 데이터 수집 및 적재
 - Clickstream 데이터 수집 및 정리, 데이터 웨어하우스에 유지 - 클릭에 해당하는 모든 정보를 수집
- 핵심 요구사항
 - 이벤트를 수집
 - 간단한 변환 수행
 - 데이터 보존

### 적용예시 : 네트워크 로그 분석
- 로그 전달 : VPC Subnet -> CloudWatch Logs -> Lambda
- 집계 및 변환 : Data Firehose -> S3
- 시각화 : Athena -> QuickSight(BI tool)

- 여러 단계로 데이터를 버퍼링 - 독자적으로 데이터를 버퍼링해서 Latency 문제 처리
- 작업 유형에 알맞은 데이터 형식 선택 - JSON vs GZIP vs Parquet
 - Athena : Parquet 등

### 지속적인 지표 생성
- 적용예시
 - 로그 사용해서 지표 생성
 - 리더보드 구축 (Top 웹 페이지 또는 앱 화면)
 - Clickstream 분석
- 핵심 요구사항
 - 늦거나 순서가 바뀔 수 있는 데이터라는 전제를 해야 함 - 10분 뒤 데이터
 - 최종 사용자에게 데이터를 신속하게 제공

- 수집 : IoT Sensor -> AWS IoT -> Data Streams
- 10초마다 평균 기온 계산 : Data Analytics
- 저장 : Streams -> Lambda -> RDB

- 타임스탬프를 필수로 저장해야 함, 스트림으로 온 값이 아닌 생성될 때의 타임스탬프
- S3 / Redshift로 정적 또는 과거의 데이터와 결합해서 분석 가능해야 함

### 반응형 분석
- 적용 예시
 - 추천 엔진
 - 지능적 장치 운영 및 경보
 - 사용자의 행동 추세와 이상행동 감지
- 핵심 요구사항
 - 어뷰징에 대해 즉각 통지
 - 스트림을 통한 지속적 실행. Stateful

- 수집 : users -> API gateway -> Kinesis Streams
- 이상 징후 및 감지 조치 : Analytics -> Streams -> Lambda
- 알림 : SNS -> email / SMS

- ex) 티켓팅 어뷰징. 수집 및 처리가 모두 실시간이어야 하며 데이터 확장성이 필요하다.
- 데이터 과학자만큼의 데이터에 대해 높은 이해가 필요
- 로그를 파악하기 위해 충분한 문맥정보 제공

## 대전게임 예시
- 캐릭터 정한 뒤 대전해서 스코어를 많이 획득하는게 목표
- 한 게임 후 수십 여개의 데이터 생성
- 집계 항목 : Top 20 캐릭터/클래스, 캐릭터/클랙스 지표 (KDA, 승률 등등), 캐릭터나 클래스로 필터링 가능
- 데이터는 S3와 Elastic에 저장

- 아키텍처
 - 수집 : Game Data -> Server -> Data Streams ->
 - 분석 : Data Analytics
 - 저장 : Firehose -> ElasticSearch/S3

- Game Data에서는 많은 데이터가 있음. 필요한 데이터만 Stream으로 전송
- Analytics : Stream, Pump 생성
- Reference Data : 캐릭터/클래스를 매핑하는 데이터
 - S3에 저장해 두고 cli에 접속해서 참조 데이터를 통해 STREAM SQL 생성 가능

- 스트리밍 데이터 집계 방법
 - 일반적인 요구사항 : 지정된 시간 내에 수집된 이벤트 들을 대상으로 set 명령 (count, average, 등)
- 윈도우란? 시간 또는 행을 기반으로 정의하는 고정된 길이의 창 t1~t3 (sum) -> 17, t3~53 -> 14
- 텀블링 윈도우 : 간격 별 집계 (0~5 / 5~10 / ...) - 데이터가 없어도 데이터 집계
- 슬라이딩 윈도우 : 지속적으로 재평가되는 윈도우 - 이벤트가 생성될 때의 기준으로 정해놓은 타임만큼 데이터 집계
