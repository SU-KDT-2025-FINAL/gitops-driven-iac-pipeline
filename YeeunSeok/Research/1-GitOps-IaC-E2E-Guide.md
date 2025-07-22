# GitOps 기반 Infrastructure as Code 파이프라인 종합 가이드

이 문서는 Terraform, AWS EKS, ArgoCD, Helm, Git 저장소 전략, 그리고 인프라 변경 감지 및 드리프트 탐지까지 포함한 GitOps 기반 IaC 파이프라인의 end-to-end 구축 방법을 안내합니다.

---

## 1. 개요
- **GitOps**: Git을 단일 신뢰 소스로 삼아 인프라 및 애플리케이션 배포를 자동화하는 운영 모델
- **IaC**: 인프라를 코드로 관리하여 일관성, 추적성, 자동화를 실현

---

## 2. 아키텍처 개요
- Git 저장소: IaC 코드, Helm 차트, 애플리케이션 매니페스트 관리
- CI/CD: 변경 감지 및 자동화된 배포 트리거
- Terraform: AWS EKS 등 인프라 리소스 프로비저닝
- ArgoCD: 선언적 애플리케이션 배포 및 동기화
- Helm: 쿠버네티스 리소스 템플릿화

---

## 3. Terraform을 이용한 AWS EKS 클러스터 프로비저닝
1. **Terraform 프로젝트 구조 예시**
    ```
    infra/
      ├── main.tf
      ├── variables.tf
      ├── outputs.tf
      └── modules/
    ```
2. **EKS 모듈 활용**
    - [terraform-aws-modules/eks/aws](https://github.com/terraform-aws-modules/terraform-aws-eks)
3. **실행 예시**
    ```bash
    terraform init
    terraform plan
    terraform apply
    ```
4. **상태 관리**
    - 원격 backend(S3, DynamoDB) 사용 권장

---

## 4. ArgoCD를 통한 선언적 배포
1. **ArgoCD 설치**
    - Helm 또는 YAML 매니페스트로 설치
2. **애플리케이션 선언**
    ```yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: my-app
    spec:
      source:
        repoURL: 'https://github.com/org/repo.git'
        path: helm/my-app
        targetRevision: main
      destination:
        server: 'https://kubernetes.default.svc'
        namespace: my-app
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    ```
3. **자동 동기화 및 롤백**
    - Git 변경 → ArgoCD 자동 동기화
    - 드리프트 발생 시 자동 self-heal

---

## 5. Helm 차트 템플릿화
1. **Helm 차트 구조**
    ```
    helm/
      my-app/
        Chart.yaml
        values.yaml
        templates/
    ```
2. **템플릿 예시**
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Values.name }}
    spec:
      replicas: {{ .Values.replicaCount }}
      ...
    ```
3. **Helm 린트 및 배포**
    ```bash
    helm lint helm/my-app
    helm template helm/my-app
    ```

---

## 6. Git 저장소 구조 설계
### 6.1 Mono-repo 전략
- 인프라, 애플리케이션, Helm 차트 등 모든 코드가 하나의 저장소에 존재
- **장점**: 변경 추적 용이, 일관된 PR 관리
- **단점**: 저장소 규모 증가, 권한 분리 어려움

### 6.2 Multi-repo 전략
- 인프라, 애플리케이션, Helm 차트 등을 별도 저장소로 분리
- **장점**: 책임 분리, 팀별 독립성
- **단점**: 변경 추적 복잡, 동기화 관리 필요

### 6.3 예시
```
mono-repo/
  infra/
  helm/
  apps/

multi-repo/
  infra-repo/
  helm-repo/
  app1-repo/
  app2-repo/
```

---

## 7. 인프라 변경사항 자동 감지 및 드리프트 탐지
1. **자동 감지**
    - Git webhook, CI 파이프라인, ArgoCD 자동 sync 활용
2. **드리프트 탐지**
    - Terraform: `terraform plan`을 주기적으로 실행하여 실제 상태와 코드 상태 비교
    - ArgoCD: 리소스 드리프트 자동 감지 및 self-heal
3. **예시: Terraform 드리프트 탐지**
    ```bash
    terraform plan -detailed-exitcode
    # 변경사항 있으면 exit code 2 반환
    ```
4. **예시: ArgoCD 드리프트 탐지**
    - UI 또는 CLI에서 드리프트 상태 확인 및 동기화

---

## 8. End-to-End 구현 플로우
1. Git 저장소에 IaC/Helm/애플리케이션 코드 관리
2. PR/Merge → CI 파이프라인에서 Terraform으로 인프라 변경 적용
3. EKS 클러스터에 ArgoCD 설치 및 애플리케이션 선언
4. ArgoCD가 Git 변경 감지 후 Helm 차트 기반으로 선언적 배포
5. ArgoCD 및 Terraform을 통한 드리프트 탐지 및 자동 복구

---

## 9. 참고 자료
- [Terraform 공식 문서](https://www.terraform.io/docs/)
- [ArgoCD 공식 문서](https://argo-cd.readthedocs.io/)
- [Helm 공식 문서](https://helm.sh/docs/)
- [GitOps CNCF](https://www.cncf.io/projects/gitops/) 