
- 가상머신을 이용해서 EC2 위에 MSSQL 설치

## RDS for MSSQL vs SQL Server on EC2
- 2008 이상 버전부터 지원
- 기존 라이센스를 AWS위에서 사용 가능
- RDS : SQL에만 집중 가능
- EC2 : 옵션 등 많은 설정이 필요함. DBA 롤

## RDS for MSSQL
- sysadmin 권한 사용 불가
- CloudWatch로 모니터링
- Native Backup / Restore 지원. S3에 백업
- 자동 백업 / 수동 백업(스냅샷) 가능
 - 스냅샷을 통한 복원 시 다른이름으로 생성해서 복원해야 된다고 함
- Multi AZ 배포 구성
 - MSSQL 미러링 기능 활용
 - Failover로 동일한 Endpoint 접속 가능
 - 사용자 및 권한 자동 복제

## SQL Server on EC2
- AMI Window 이미지 세팅 후 설치
- HA
 - 미러링, Read/Write 분리, Multi AZ
- Always On AG
 - EC2 VIP 설정 후 Cluster Core Resources에서 설정

## MSSQL to AWS Migration
1. 파일로 저장 후 S3복사 이후에 마이그레이션
- IDC 서버와 Async
- AWS Database Migration Service 사용
 - 데이터베이스, 데이터 웨어하우스 마이그레이션 서비스
 - IDC -> DMS 복제인스턴스 -> 새로운 MSSQL
