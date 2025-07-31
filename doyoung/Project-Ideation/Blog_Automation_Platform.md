# 개발자 블로그 자동화 플랫폼

## 1. 핵심 서비스 아이디어

**개발자 블로그 및 포트폴리오 자동 생성/배포 플랫폼**

사용자가 자신의 GitHub 저장소에 마크다운 형식으로 글을 작성하여 Push하면, 자동으로 웹사이트가 빌드되고 전용 인프라에 배포되는 서비스입니다. 각 사용자의 인프라(S3, CloudFront 등) 생성부터 콘텐츠 업데이트까지 모든 과정을 IaC와 GitOps로 자동화합니다.

## 2. 서비스 구성

### Git-Webhook-Service
- 사용자의 Git Push 이벤트를 수신하는 서비스
- GitHub/GitLab 웹훅을 통해 변경사항 감지

### Build-Service
- Git 저장소를 복제하여 정적 사이트(Hugo, Jekyll 등)를 빌드
- 빌드 결과물을 S3에 업로드
- 빌드 상태 및 로그 관리

### Provisioning-Service
- 신규 사용자 등록 시 블로그에 필요한 AWS 인프라 자동 생성
- Terraform 코드를 실행하여 S3 버킷, CloudFront 등 관리
- 인프라 상태 모니터링 및 업데이트

### User-Service
- 사용자 계정 및 Git 저장소 정보 관리
- 인증/인가 처리
- 사용자별 설정 관리

## 3. 시스템 유형

**서버리스 이벤트 주도 아키텍처 (Serverless EDA)**

Git Push라는 이벤트가 발생하면 Git-Webhook-Service(Lambda)가 실행되고, 이 서비스가 SQS 같은 메시지 큐에 빌드 작업을 요청합니다. Build-Service(Lambda)가 이 메시지를 받아 빌드를 수행하는 등 전체 흐름이 이벤트 중심으로 비동기적으로 동작합니다.

### 주요 특징
- 이벤트 기반 비동기 처리
- 서버리스 아키텍처로 비용 효율성
- 자동 스케일링
- GitOps 기반 인프라 관리