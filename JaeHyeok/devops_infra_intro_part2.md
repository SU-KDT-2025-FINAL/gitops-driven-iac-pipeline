# DevOps 인프라 자동화 기초 입문 (Part 2)  
*GitOps 기반 IaC 관리 & CI/CD 파이프라인 예제*

---

## 5. GitOps 기반 IaC 관리 CI/CD 파이프라인 예시

flowchart LR
Dev[Developer]
Git[Git Repository]
CI[CI 서버(GitHub Actions 등)]
CD[CD(ArgoCD/Flux 등)]
K8s[Kubernetes/서버 인프라]

text
Dev-->|코드 커밋|Git
Git-->|빌드+테스트|CI
CI-->|IaC 코드+앱 배포 yaml 반영|Git
CD-->|변경 감지/자동 배포|K8s
text

### 설명

1. 개발자는 앱 코드 또는 IaC 코드(Terraform, Helm 등)를 변경하여 Git에 푸시
2. GitHub Actions 등 CI 서버가 자동으로 빌드 및 테스트 수행
3. 변경 사항이 Git 저장소에 병합되면, CD 도구(ArgoCD 등)가 Git 상태 ↔ 클러스터 상태를 비교
4. 변경이 발생한 경우 자동으로 배포 수행  

---

## 6. 서비스 개발 방향 및 적용 예시

### 기본 전략
- 인프라, 앱, 설정 파일 등 모든 리소스를 Git에서 관리
- PR 기반 코드리뷰로 검토 및 병합
- Git 상태 = 실제 인프라 상태

### CI 구성 (GitHub Actions 예)
- 빌드, 테스팅, 린트 검사
- Terraform plan 검증 포함

### CD 구성 (ArgoCD 예)
- Git의 새로운 커밋 감지
- 쿠버네티스 클러스터에 자동 동기화
- Rollback 기능 포함