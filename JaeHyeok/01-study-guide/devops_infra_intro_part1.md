# DevOps 인프라 자동화 기초 입문 (Part 1)  
*CI/CD, IaC, GitOps: 개념 및 기초 정리*

---

## 1. DevOps란?

**DevOps**란 개발(Development)과 운영(Operations)이 협업과 자동화를 통해 소프트웨어를 빠르고 안정적으로 제공하기 위한 문화와 실천 방법론입니다.

- **목적**: 빠른 배포, 신뢰성 향상, 자동화, 협업 강화  
- **특징**: 자동화 도구 활용, 지속적인 개선, 모니터링  

---

## 2. CI/CD(지속적 통합/지속적 배포)

- **CI(Continuous Integration)**  
    - 여러 개발자가 작성한 코드를 자주 통합하고, 자동으로 테스트하여 문제를 빠르게 발견하는 방식
- **CD(Continuous Delivery / Continuous Deployment)**  
    - 통합된 코드를 프로덕션에 가까운 환경(혹은 실제 운영)에 자동으로 배포하는 방식  

**장점 요약**
- 코드 충돌을 빠르게 해결  
- 테스트 자동화로 품질 향상  
- 빠른 배포 주기 가능  
- 팀 단위 협업 문화 마련  

---

## 3. IaC (Infrastructure as Code)

**IaC란?**  
> 인프라 리소스를 코드 형태로 선언하고 배포/관리를 자동화하는 기법

### 예시 도구
- Terraform (HashiCorp)
- AWS CloudFormation
- Pulumi
- Ansible (Declarative + Procedural 하이브리드)

### 주요 장점
- **재현 가능성**: 환경을 동일하게 재생성 가능  
- **버전 관리 가능**: Git으로 인프라 이력 추적  
- **협업 강화**: 코드 리뷰 기반 운영  
- **변경 추적**: 로그로 인프라 변경 내용 파악

---

## 4. GitOps란?

**GitOps**는 Git 저장소를 인프라/애플리케이션 배포의 단일 사실의 근원(Single Source of Truth)으로 삼아, 자동으로 적용·동기화하는 현대적 배포 전략입니다.

### 핵심 개념
- Git = 배포 명세서  
- Git Commit = 구성을 선언하는 명령  
- CI/CD와 달리, **CD를 Git 상태에 맞춰 자동 동기화**  

### GitOps 주요 툴
- **ArgoCD**
- **Flux**
- GitHub Actions (CI와 연결)

### GitOps 장점
- 변경 이력 추적 가능 (PR 기반 CD)
- 자동/수동 롤백 용이
- 보안성과 접근 제어 향상 (Git 권한만으로 관리)
