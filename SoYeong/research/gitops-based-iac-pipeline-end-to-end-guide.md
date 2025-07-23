# GitOps 기반 IaC 파이프라인 구축 가이드

GitOps 원칙을 적용하여 Infrastructure as Code(IaC) 배포 파이프라인을 end-to-end로 구축하는 방법을 단계별로 상세히 설명합니다.  
Git 저장소 전략부터 변경 감지, 드리프트 탐지 및 자동 복구까지 핵심 내용을 포함합니다.

---

## 1. Git 저장소 전략

- **단일 리포지토리 vs. 멀티 리포지토리**
  - *단일 리포지토리*: IaC 코드와 매니페스트를 한 곳에서 관리, 소규모 팀에 적합
  - *멀티 리포지토리*: 애플리케이션, 인프라, 환경별 디렉토리 및 저장소 분리, 대규모 조직에 유리

- **브랜치 전략**
  - 환경별 브랜치(예: dev, staging, prod) 운영
  - PR(Merge Request) 기반 코드 변경 리뷰 및 승인 프로세스 적용

- **폴더 및 파일 구조 예시**

├── environments
│   ├── prod
│   │   └── main.tf
│   └── staging
│       └── main.tf
├── modules
│   ├── network
│   └── compute
└── k8s
    ├── manifests
    └── helm-charts


- **변경 이력 및 롤백**
- Git 커밋 및 PR 이력으로 모든 변경사항 추적
- 문제가 발생하면 이전 커밋으로 간편 롤백 가능

---

## 2. 변경 감지 및 자동 배포

### 2-1. PR 기반 변경 및 검증

- PR 생성 시 자동으로 Lint, 문법 검사, 보안 점검 수행
- `terraform plan`, `kustomize build` 등 사전 시뮬레이션을 통해 변경 영향도 검증
- 코드 리뷰 및 승인을 통해 안전한 변경 보장

### 2-2. GitOps 컨트롤러에 의한 배포

- ArgoCD, Flux 같은 GitOps 도구가 Git 리포지토리 변화를 감지
- 선언형 설정과 실제 인프라 상태 차이(Diff)를 비교 후 자동 동기화(Sync) 수행
- 배포 성공/실패 결과를 로그 및 알림 시스템에 통보

---

## 3. 드리프트 감지 및 자동 복구

- **드리프트(Drift) 정의**  
선언된 상태와 실제 인프라 상태 간 불일치 상황

- **탐지 방법**  
GitOps 컨트롤러가 주기적으로 클러스터 상태 점검, OutOfSync 상태 발생 시 경고

- **복구 방법**  
설정된 정책에 따라 자동 동기화로 선언된 상태로 복원(Self-Healing)  
또는 수동으로 Sync 진행 가능

- **알림 및 추적**  
Slack, 이메일 등으로 알림 전송, 운영자 감사 추적

---

## 4. 구축 절차 요약

| 단계             | 상세 내용                                           |
|------------------|--------------------------------------------------|
| 코드 작성 및 PR 생성 | Git 저장소에 IaC 코드 작성, PR로 변경 요청              |
| 자동 검사 및 리뷰    | Lint, 정적분석, Plan 실행 및 리뷰 진행                 |
| Merge 및 컨트롤러 감지 | 코드 승인 후 Merge, GitOps 도구가 자동 배포 시작      |
| 선언형 배포 적용    | 변경된 구성으로 인프라 및 클러스터 리소스 수정 적용           |
| 드리프트 감지 및 복구 | 상태 불일치 감지 시 자동 또는 수동으로 동기화 수행           |
| 모니터링 및 알림    | Grafana, Prometheus 등으로 배포 상태와 이상 모니터링      |

---

## 5. 실무 팁 및 고려사항

- 변경은 반드시 PR을 활용해 관리하고, 충분한 리뷰와 자동 테스트를 거치게 할 것  
- 자동 배포 도구는 실시간 변경 감지 외에도 분산 환경에 맞춰 적절한 동기화 주기 조절 필요  
- 드리프트 복구 정책은 환경 안정성에 따라 자동 또는 수동모드로 구분 운영  
- 인프라 코드 및 선언형 설정 변경 시 버전 관리를 철저히 해 복구와 추적을 용이하게 할 것  
- 배포 후 모니터링, 로그 수집 및 알림으로 문제 조기 발견 및 신속 대응 체계 구축  

---

## 6. 참고 문서 링크

- [GitHub Flow 브랜치 전략](https://docs.github.com/en/get-started/quickstart/github-flow)  
- [HashiCorp Terraform Best Practices](https://developer.hashicorp.com/terraform/enterprise/guides/recommended-practices/vcs)  
- [ArgoCD 공식문서 - Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)  
- [Flux 공식문서](https://fluxcd.io/docs/)  
- [Terraform Plan 및 Apply 가이드](https://developer.hashicorp.com/terraform/tutorials/automation/plan-apply)  
- [GitOps 개념 및 사례](https://www.weave.works/technologies/gitops/)  

---

> 위 가이드를 따라 단계별로 구성과 실습을 진행하면  
> GitOps 기반 IaC 파이프라인을 안정적이고 효율적으로 구축할 수 있습니다.

