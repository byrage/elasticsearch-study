# GitHub to Lambda: Developing, testing and deploying serverless apps

## Agenda
- Code Star?
- 클라우드 베이스 소프트웨ㅓ 생성, 관리 배포
- develop - build - deploy

- Github (ex Lambda function) -> [Notification Message] -> Codestar (CloudFormation, CloudWatch, CodePipleine, IAM, Lambda, S3)

## Getting Started
- create IAM
- create Codestar project
- Explore the project Coddestar and Github
- 코드 커밋 시 자동으로 배포

### Create IAM
- IAM Console -> Add user
- AWS Management Console access 체크
- reset password 언체크
- S3FullAccess / CodeStarAccess / LambdaFullAcces
- 생성된 URL 복사

### Signin Identity and Access Management Admin user
- 복사한 IAM 로그인 URL 접속

### Create CodeStar project
- 프로젝트 생성 - 프로젝트 타입 지정 (Node or Spring 등등)
- github에 소스 저장
- 15~20분 걸린다고 함

### Explore the project
- 대시보드에서 어플리케이션 엔드포인트를 보여줌
- 대시보드에서 브랜치나 커밋로그, github issue, PR을 한 페이지에서 볼 수 있다.
- 빌드에서 빌드 현황 확인 가능
- Deploy에서 Code Deploy를 통해 히스토리 확인 가능
- Pipeline - Code Pipeline

### Code Change
- 코드 변경 시 대시보드에서 커밋 히스토리 확인 가능
- 코드 변경시 자동으로 배포가 되는듯?

- 브랜치 빌드 및 배포 가능 Lambda 테스트 환경
- 브랜치 보호
- PR
- 어플리케이션을 오픈소스로 퍼블리시 가능 - IAM 아이디를 통해 협업이 가능하다.
