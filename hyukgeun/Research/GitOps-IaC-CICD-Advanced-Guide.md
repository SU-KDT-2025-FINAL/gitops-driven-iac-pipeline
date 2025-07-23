# GitOps 기반 IaC 관리 CI/CD 파이프라인 고도화 가이드

## 1. GitOps 개요 및 원리

### 1-1. GitOps란 무엇인가?
- Git을 단일 진실 소스로 삼아 인프라/앱을 선언적으로 관리
- 모든 변경은 Git Commit/PR로 추적
- 인프라와 애플리케이션의 상태를 코드로 정의하고, Git 저장소에 저장함으로써 변경 이력, 감사, 롤백이 용이함
- 실무 예시: AWS EKS 클러스터, 네트워크, IAM 등 모든 리소스를 Terraform 코드로 정의하고 Git에 저장

### 1-2. GitOps의 4대 원칙
- **선언적(Declarative)**: 시스템의 원하는 상태를 선언적으로 기술 (예: K8s 매니페스트, Terraform 코드)
- **버전 관리(Versioned & Immutable)**: 모든 변경 이력은 Git에 저장, 롤백 및 감사가 가능
- **자동화된 동기화(Automated Sync)**: Git 상태와 실제 상태를 자동 동기화 (ArgoCD, Flux 등)
- **지속적 검증(Continuous Reconciliation)**: Drift(상태 불일치) 발생 시 자동 복구

### 1-3. GitOps의 장점과 한계
- **장점**: 투명성, 감사, 롤백, 협업 강화, 자동화된 운영, 감사 추적성, 인프라/앱의 일관성 유지
- **한계**:
  - 복잡한 상태 관리(대규모 인프라, 멀티클러스터 등)
  - 대규모 환경에서의 성능 이슈(동기화 지연, 대량 리소스 관리)
  - 실시간 동기화 한계, 외부 시스템 연동의 복잡성
- **고도화 방안**: 멀티클러스터 관리, 분산 GitOps, 이벤트 기반 동기화, 캐싱 전략, GitOps Operator의 확장성 확보

---

## 2. IaC(Terraform)를 활용한 인프라 정의 및 버전 관리

### 2-1. Terraform 기본 구조와 모듈화
- main.tf, variables.tf, outputs.tf, modules/ 등으로 구성
- 모듈화: 공통 인프라(네트워크, IAM 등)는 모듈로 분리, 재사용성/표준화 강화
- 예시:
```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr_block = var.vpc_cidr
}
```
- 실무에서는 네트워크, 보안, 클러스터, 데이터베이스 등 각 리소스를 모듈로 분리하여 관리

### 2-2. 상태 파일 관리
- 원격 Backend(S3, GCS, Azure Blob 등) 사용, 팀 협업/충돌 방지
- Locking: DynamoDB 등으로 동시 작업 방지
- State Drift 관리: terraform plan/apply, driftctl 등으로 상태 불일치 감지
- 보안: State 파일 내 민감정보 암호화, Backend 접근 권한 최소화, Vault 등 외부 시크릿 관리 연동, State 파일 자동 백업/감사
- 실무 예시:
```hcl
terraform {
  backend "s3" {
    bucket = "my-tf-state"
    key    = "dev/terraform.tfstate"
    region = "ap-northeast-2"
    dynamodb_table = "my-tf-lock"
    encrypt = true
  }
}
```

### 2-3. 실무적 모범 사례
- PR 기반 코드 리뷰, Merge 전 자동 plan 결과 공유
- 환경별 워크스페이스 분리(`dev`, `staging`, `prod`)
- 정책 적용: Sentinel, OPA(Policy as Code)로 인프라 변경 정책 자동 검증
- PR마다 자동 plan-comment, Slack/Teams 알림, 정책 위반시 자동 reject
- 예시: Terraform Cloud/Enterprise의 Policy Check, GitHub Actions에서 tfsec, checkov 등 취약점 검사

---

## 3. Git 저장소 구조 설계

### 3-1. 단일 vs 다중 저장소 전략
- Mono-repo: 모든 인프라/앱을 하나의 저장소에서 관리, 일관성↑, 충돌↑
- Multi-repo: 서비스/팀별 분리, 독립성↑, 관리 복잡도↑
- 선택 기준: 조직 규모, 서비스 독립성, 배포 빈도, 보안 요구사항 등
- 고도화 방안: Git submodule, monorepo 내 디렉토리별 권한 관리, trunk-based 개발, 환경별 브랜치 전략

### 3-2. 예시 디렉토리 구조
```
repo-root/
├── infra/
│   ├── dev/
│   ├── staging/
│   └── prod/
│       ├── main.tf
│       └── modules/
├── helm/
│   └── myapp/
├── apps/
│   └── myapp/
└── .github/
    └── workflows/
```

### 3-3. 환경별 분리 전략
- 환경별 디렉토리/브랜치/워크스페이스 분리
- 환경별 변수/시크릿은 별도 관리(예: SOPS, Sealed Secrets, HashiCorp Vault)
- 환경간 promote 자동화(예: dev→staging→prod), 환경별 정책 적용
- 실무 예시: 환경별로 별도 Git 브랜치, ArgoCD ApplicationSet으로 환경별 배포 자동화

---

## 4. GitHub Actions 또는 다른 CI도구를 활용한 terraform plan/apply 자동화

### 4-1. GitHub Actions 워크플로우 예시
- PR 시 terraform plan 결과를 PR comment로 자동 등록
- main merge 시 terraform apply 자동 실행
- 보안 인증: OIDC, AWS AssumeRole, GCP Workload Identity Federation 등
- 취약점 스캔(tfsec, checkov 등) 자동화, Slack/Teams 알림
- 예시:
```yaml
name: 'Terraform CI'
on:
  pull_request:
    paths: [ 'infra/**' ]
  push:
    branches: [ main ]
    paths: [ 'infra/**' ]
jobs:
  terraform:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: infra
    steps:
      - uses: actions/checkout@v3
      - uses: hashicorp/setup-terraform@v2
      - run: terraform init
      - run: terraform plan
        if: github.event_name == 'pull_request'
      - run: terraform apply -auto-approve
        if: github.ref == 'refs/heads/main'
      - uses: aquasecurity/tfsec-pr-commenter-action@v1.2.0
        if: github.event_name == 'pull_request'
```

### 4-2. 기타 CI 도구와의 비교
- Jenkins: 온프레미스, 플러그인 다양, 복잡한 파이프라인, Job별 RBAC
- GitLab CI: GitLab과 통합, 멀티 프로젝트 파이프라인, CI/CD as Code
- CircleCI: 빠른 빌드, 설정 간편, SaaS 기반
- Self-hosted runner: 보안, 네트워크 제약 환경에서 유리, 사내망 연동
- Runner 보안 강화, Job별 RBAC, 감사로그 연동, 취약점 스캔 자동화

---

## 5. ArgoCD를 이용한 애플리케이션 배포 자동화 흐름

### 5-1. ArgoCD 기본 동작 원리
- Git 저장소 모니터링 → K8s 동기화
- 수동/자동 동기화, Rollback, Health Check, Diff View 제공
- 실무 예시: Git에 Helm values.yaml 변경 → ArgoCD가 자동 감지 후 배포

### 5-2. 고도화된 운영 전략
- 멀티클러스터 배포: 여러 K8s 클러스터 동시 관리, ApplicationSet 활용
- App of Apps 패턴: 대규모 환경에서 앱/인프라 계층적 관리
- RBAC, SSO, 감사로그: 조직 정책 준수, 접근 제어, 변경 이력 추적
- ApplicationSet으로 대량 앱 자동 생성, Webhook 기반 실시간 동기화, ArgoCD Image Updater로 자동 이미지 태그 업데이트
- 실무 예시: staging/prod 클러스터 동시 배포, SSO 연동, 배포 승인 프로세스

---

## 6. Helm을 활용한 선언적 배포 방식

### 6-1. Helm 차트 구조와 템플릿화
- values.yaml, templates/로 파라미터화
- 환경별 values 파일 분리(values-dev.yaml, values-prod.yaml)
- Helm test, pre/post hook, chart lint 자동화
- 실무 예시: Helm chart lint, chart test, chart versioning 자동화

### 6-2. Helm + ArgoCD 통합
- ArgoCD에서 Helm 차트 직접 배포, values override 지원
- 시크릿 관리: Sealed Secrets, SOPS, External Secrets 연동
- Helm chart repository 관리, chart versioning, chart promotion 자동화
- 실무 예시: ArgoCD에서 Helm chart values를 환경별로 override, SOPS로 시크릿 암호화

---

## 7. Git commit → 인프라 변경 → 애플리케이션 반영까지의 전체 흐름

### 7-1. 전체 파이프라인 시나리오
1. 개발자 코드/인프라 변경 후 PR
2. CI에서 terraform plan, 코드 리뷰
3. main merge 시 terraform apply
4. Helm 차트/매니페스트 변경 → Git 반영
5. ArgoCD가 Git 변경 감지 후 자동 배포
6. 배포 결과/오류 실시간 알림

### 7-2. 텍스트 기반 아키텍처 다이어그램
```
[개발자]
   |
   v
[Git Commit/PR]
   |
   v
[CI/CD (GitHub Actions)]
   |
   v
[클라우드 인프라 (Terraform)]
   |
   v
[ArgoCD]
   |
   v
[Kubernetes 클러스터]
```

### 7-3. 실시간 모니터링 및 알림 연동
- Slack, Teams, Email, PagerDuty 등과 연동
- 배포 성공/실패, Drift 감지, 정책 위반 등 이벤트별 알림
- 알림 메시지 커스터마이즈, 대시보드 연동, 자동 이슈 생성
- 실무 예시: GitHub Actions에서 배포 성공/실패시 Slack 알림, ArgoCD Webhook으로 배포 상태 연동

---

## 8. 실습용 프로젝트 예시

### 8-1. 예시 디렉토리 구조
```
repo-root/
├── infra/
│   ├── dev/
│   ├── staging/
│   └── prod/
│       ├── main.tf
│       └── modules/
├── helm/
│   └── myapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-prod.yaml
│       └── templates/
│           └── deployment.yaml
├── apps/
│   └── myapp/
│       └── src/
└── .github/
    └── workflows/
        └── terraform.yml
```

### 8-2. 예시 코드 및 실습 시나리오
- Terraform으로 EKS, S3, RDS 등 인프라 생성
- Helm 차트로 샘플 앱, Ingress, 모니터링 툴 배포
- GitHub Actions로 자동화, ArgoCD로 K8s 배포 자동화
- Drift 감지 실습: 수동 변경 후 Drift 감지 및 복구
- 실습용 시나리오에 정책 위반, 장애 복구, 멀티클러스터 배포 등 추가
- 예시 코드:
```hcl
# infra/prod/main.tf
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  cluster_name = "prod-eks"
  # ...
}
```
```yaml
# helm/myapp/values-prod.yaml
replicaCount: 3
image:
  repository: myrepo/myapp
  tag: "2.0.0"
```

---

## 9. Drift 감지 및 자동 복구 전략

### 9-1. Drift 감지 방법
- Terraform: `terraform plan`, `driftctl`, `infracost` 등으로 상태 불일치 감지
- ArgoCD: 자동 drift 감지, UI/CLI로 diff 확인, Sync 상태 표시
- 실무 예시: 수동으로 리소스 변경 후 ArgoCD UI에서 OutOfSync 상태 확인

### 9-2. 자동 복구 및 정책 적용
- ArgoCD auto-sync, webhook 연동, 정책 위반시 배포 차단
- Policy as Code(Open Policy Agent, Kyverno 등)로 배포 정책 자동화
- Drift 발생 시 자동 알림, 자동 롤백/재적용, 정책 위반시 배포 차단
- 예시: ArgoCD auto-sync 활성화, OPA Gatekeeper로 정책 위반시 배포 차단

---

## 10. 참고 문서 및 실무 적용시 유의사항

### 10-1. 참고 문서
- [GitOps 공식 문서](https://www.gitops.tech/)
- [Terraform 공식 문서](https://www.terraform.io/docs/)
- [ArgoCD 공식 문서](https://argo-cd.readthedocs.io/)
- [Helm 공식 문서](https://helm.sh/docs/)
- [GitHub Actions 공식 문서](https://docs.github.com/en/actions)
- [OPA Gatekeeper](https://open-policy-agent.github.io/gatekeeper/)
- [driftctl](https://github.com/snyk/driftctl)
- [tfsec](https://aquasecurity.github.io/tfsec/)

### 10-2. 실무 적용시 유의사항
- 인프라/앱 코드 분리 및 권한 관리, GitOps 도구(ArgoCD 등) RBAC 및 보안 설정
- Terraform State 파일 보안(S3, GCS, Backend 등), PR 기반 코드 리뷰 및 승인 프로세스
- 자동화 도구 장애 시 수동 복구 플랜 마련, 멀티클러스터, 정책관리, 보안 자동화 등 고도화 방안
- 감사로그, 배포 이력 관리, 장애/이슈 자동 감지 및 대응, 비용 최적화, 리소스 자동 스케일링, 멀티클라우드 지원
- 실무 예시: ArgoCD RBAC 정책, Terraform State 암호화, Slack/Teams 알림, 장애 발생시 자동 롤백