# ArgoCD 완전 가이드

## 🎯 ArgoCD란?

**ArgoCD**는 Kubernetes용 **GitOps 지속적 배포(CD) 도구**입니다. Git 저장소에 정의된 애플리케이션 상태를 Kubernetes 클러스터에 자동으로 동기화하여 배포를 관리합니다.

### 📋 핵심 특징
- **GitOps 원칙**: Git을 단일 소스 of Truth로 사용
- **선언적**: 원하는 상태를 선언하면 자동으로 동기화
- **자동 동기화**: Git 변경사항을 실시간으로 감지하여 클러스터에 반영
- **웹 UI**: 직관적인 웹 인터페이스로 배포 상태 모니터링
- **멀티 클러스터**: 여러 Kubernetes 클러스터 관리 지원

## 🏗️ ArgoCD 아키텍처

### 📊 전체 구조
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Git 저장소     │    │     ArgoCD      │    │   Kubernetes    │
│                 │    │                 │    │    클러스터      │
│ • K8s 매니페스트 │◄──►│ • GitOps Operator│◄──►│ • Pods          │
│ • Helm Charts   │    │ • 웹 UI         │    │ • Services      │
│ • Kustomize     │    │ • API Server    │    │ • ConfigMaps    │
│ • Config        │    │ • Controller    │    │ • Secrets       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 🔧 핵심 컴포넌트

**1. ArgoCD Server**
- 웹 UI 및 API 서버 제공
- Git 저장소와 Kubernetes 클러스터 간 중재자 역할
- 사용자 인증 및 권한 관리

**2. Application Controller**
- Git 저장소 모니터링
- Kubernetes 리소스 상태 감시
- 자동 동기화 및 상태 조정

**3. Repo Server**
- Git 저장소에서 매니페스트 파일 읽기
- Helm, Kustomize, Jsonnet 등 템플릿 렌더링
- Git 인증 및 SSH 키 관리

## 🚀 ArgoCD 설치 및 설정

### 1. ArgoCD 설치

**Helm을 사용한 설치:**
```bash
# ArgoCD Helm 저장소 추가
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# ArgoCD 설치
helm install argocd argo/argo-cd \
  --namespace argocd \
  --create-namespace \
  --set server.ingress.enabled=true \
  --set server.ingress.hosts[0]=argocd.your-domain.com
```

**kubectl을 사용한 설치:**
```bash
# ArgoCD 네임스페이스 생성
kubectl create namespace argocd

# ArgoCD 설치
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. 초기 설정

**관리자 비밀번호 확인:**
```bash
# 초기 비밀번호 확인
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**ArgoCD CLI 설치:**
```bash
# macOS
brew install argocd

# Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
```

**ArgoCD 서버 로그인:**
```bash
# 포트 포워딩
kubectl port-forward svc/argocd-server -n argocd 8080:443

# 로그인
argocd login localhost:8080 --username admin --password <초기_비밀번호>
```

## 📝 애플리케이션 정의

### 🔧 Application CRD (Custom Resource Definition)

ArgoCD는 `Application` CRD를 사용하여 배포할 애플리케이션을 정의합니다.

**기본 Application 예시:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-repo
    targetRevision: HEAD
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

### 📁 소스 타입별 예시

**1. Plain Kubernetes 매니페스트:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: nginx-app
spec:
  source:
    repoURL: https://github.com/my-org/k8s-manifests
    targetRevision: main
    path: nginx
  destination:
    server: https://kubernetes.default.svc
    namespace: nginx
```

**2. Helm Chart:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: wordpress-app
spec:
  source:
    repoURL: https://charts.bitnami.com/bitnami
    chart: wordpress
    targetRevision: 15.0.0
    helm:
      values: |
        wordpressUsername: admin
        wordpressPassword: secret123
        mariadb.enabled: true
  destination:
    server: https://kubernetes.default.svc
    namespace: wordpress
```

**3. Kustomize:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: kustomize-app
spec:
  source:
    repoURL: https://github.com/my-org/kustomize-demo
    targetRevision: main
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
```

## 🔄 동기화 정책

### 📊 Sync Policy 옵션

**1. 수동 동기화 (Manual Sync):**
```yaml
spec:
  syncPolicy: {}  # 동기화 정책 없음
```

**2. 자동 동기화 (Automated Sync):**
```yaml
spec:
  syncPolicy:
    automated:
      prune: true        # Git에서 삭제된 리소스 자동 제거
      selfHeal: true     # 클러스터에서 수동 변경 시 자동 복구
      allowEmpty: false  # 빈 애플리케이션 허용 안함
```

**3. 선택적 동기화 (Selective Sync):**
```yaml
spec:
  syncPolicy:
    syncOptions:
    - CreateNamespace=true    # 네임스페이스 자동 생성
    - PrunePropagationPolicy=foreground
    - PruneLast=true
```

### 🎯 동기화 옵션

**Sync Options:**
- `CreateNamespace=true`: 네임스페이스 자동 생성
- `PrunePropagationPolicy=foreground`: 삭제 전략
- `PruneLast=true`: 삭제를 마지막에 수행
- `Validate=false`: 검증 건너뛰기
- `ServerSideApply=true`: 서버 사이드 적용

## 🔐 보안 및 인증

### 🔑 Git 인증

**1. SSH 키 인증:**
```bash
# SSH 키 생성
ssh-keygen -t rsa -b 4096 -C "argocd@example.com"

# ArgoCD에 SSH 키 등록
argocd repo add https://github.com/my-org/my-repo \
  --ssh-private-key-path ~/.ssh/id_rsa
```

**2. HTTPS 인증:**
```bash
# 사용자명/비밀번호로 저장소 추가
argocd repo add https://github.com/my-org/my-repo \
  --username myuser \
  --password mypassword
```

**3. TLS 인증서:**
```bash
# 자체 서명된 인증서로 저장소 추가
argocd repo add https://git.example.com/my-repo \
  --insecure-skip-server-verification
```

### 👥 RBAC (Role-Based Access Control)

**Project 정의:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
spec:
  description: My application project
  sourceRepos:
  - 'https://github.com/my-org/*'
  destinations:
  - namespace: production
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  namespaceResourceWhitelist:
  - group: ''
    kind: ConfigMap
  - group: ''
    kind: Secret
  - group: apps
    kind: Deployment
```

**Role 정의:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
spec:
  roles:
  - name: developer
    description: Developer role
    policies:
    - p, proj:my-project:developer, applications, get, my-project/*, allow
    - p, proj:my-project:developer, applications, sync, my-project/*, allow
    groups:
    - my-org:developers
```

## 📊 모니터링 및 알림

### 🔍 상태 모니터링

**애플리케이션 상태:**
- **Synced**: Git과 클러스터 상태 일치
- **OutOfSync**: Git과 클러스터 상태 불일치
- **Unknown**: 상태 확인 불가
- **Error**: 동기화 오류

**헬스 상태:**
- **Healthy**: 모든 리소스 정상
- **Progressing**: 배포 진행 중
- **Degraded**: 일부 리소스 문제
- **Suspended**: 일시 중지됨

### 📧 알림 설정

**Slack 알림 예시:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  service.slack: |
    token: $slack-token
  template.app-sync-succeeded: |
    message: |
      Application {{.app.metadata.name}} sync succeeded.
      Environment: {{.app.spec.destination.namespace}}
      Revision: {{.app.status.sync.revision}}
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [app-sync-succeeded]
```

## 🚀 고급 기능

### 🔄 멀티 클러스터 관리

**여러 클러스터에 배포:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: multi-cluster-app
spec:
  source:
    repoURL: https://github.com/my-org/my-repo
    path: k8s
  destination:
    server: https://cluster2.example.com:6443
    namespace: production
```

### 🎯 ApplicationSet

**여러 애플리케이션 일괄 생성:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: guestbook
spec:
  generators:
  - list:
      elements:
      - cluster: in-cluster
        url: https://kubernetes.default.svc
      - cluster: cluster-2
        url: https://cluster-2.example.com:6443
  template:
    metadata:
      name: '{{cluster}}-guestbook'
    spec:
      project: default
      source:
        repoURL: https://github.com/argoproj/argocd-example-apps
        targetRevision: HEAD
        path: guestbook
      destination:
        server: '{{url}}'
        namespace: guestbook
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### 🔧 Helm Values 관리

**환경별 Helm Values:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helm-app
spec:
  source:
    repoURL: https://github.com/my-org/helm-charts
    chart: my-app
    targetRevision: 1.0.0
    helm:
      valueFiles:
      - values-production.yaml
      values: |
        replicaCount: 3
        image:
          tag: latest
        ingress:
          enabled: true
          hosts:
          - host: my-app.example.com
```

## 🛠️ 문제 해결

### 🔍 일반적인 문제들

**1. 동기화 실패:**
```bash
# 애플리케이션 상태 확인
argocd app get my-app

# 동기화 로그 확인
argocd app logs my-app

# 수동 동기화 시도
argocd app sync my-app
```

**2. Git 인증 문제:**
```bash
# 저장소 연결 상태 확인
argocd repo list

# 저장소 재연결
argocd repo remove https://github.com/my-org/my-repo
argocd repo add https://github.com/my-org/my-repo --ssh-private-key-path ~/.ssh/id_rsa
```

**3. 리소스 충돌:**
```bash
# 충돌 해결을 위한 강제 동기화
argocd app sync my-app --force

# 특정 리소스만 동기화
argocd app sync my-app --resource Deployment:my-app
```

### 📊 디버깅 명령어

```bash
# 애플리케이션 목록
argocd app list

# 애플리케이션 상세 정보
argocd app get my-app

# 동기화 상태 확인
argocd app sync-status my-app

# 리소스 트리 보기
argocd app resources my-app

# 이벤트 로그
argocd app events my-app

# 매니페스트 확인
argocd app manifests my-app
```

## 🎯 ArgoCD vs 다른 도구들

### 📊 비교표

| 기능 | ArgoCD | Flux | Jenkins X | Spinnaker |
|------|--------|------|-----------|-----------|
| **GitOps 지원** | ✅ 완전 | ✅ 완전 | ✅ 부분 | ❌ |
| **웹 UI** | ✅ 우수 | ❌ | ✅ 기본 | ✅ 우수 |
| **멀티 클러스터** | ✅ | ✅ | ✅ | ✅ |
| **Helm 지원** | ✅ | ✅ | ✅ | ✅ |
| **Kustomize 지원** | ✅ | ✅ | ✅ | ❌ |
| **설치 복잡도** | 중간 | 쉬움 | 복잡 | 복잡 |
| **학습 곡선** | 중간 | 쉬움 | 높음 | 높음 |

### 🎯 ArgoCD의 장점

**1. 강력한 웹 UI**
- 직관적인 애플리케이션 관리
- 실시간 상태 모니터링
- 시각적 리소스 트리

**2. GitOps 원칙 준수**
- Git을 단일 소스 of Truth로 사용
- 선언적 배포 관리
- 자동 동기화 및 복구

**3. 다양한 소스 지원**
- Plain Kubernetes 매니페스트
- Helm Charts
- Kustomize
- Jsonnet

**4. 엔터프라이즈 기능**
- RBAC 및 보안
- 멀티 클러스터 관리
- ApplicationSet
- 알림 및 모니터링

## 🚀 실제 사용 시나리오

### 📝 시나리오 1: 웹 애플리케이션 배포

**1. Git 저장소 준비:**
```
my-web-app/
├── k8s/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── kustomization.yaml
│   └── overlays/
│       ├── development/
│       │   └── kustomization.yaml
│       └── production/
│           └── kustomization.yaml
└── README.md
```

**2. ArgoCD Application 생성:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: web-app-prod
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/my-web-app
    targetRevision: main
    path: k8s/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
```

**3. 배포 과정:**
```
개발자 코드 수정 → Git 푸시 → ArgoCD 감지 → 자동 동기화 → 배포 완료
```

### 📝 시나리오 2: 마이크로서비스 배포

**ApplicationSet 사용:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: microservices
spec:
  generators:
  - list:
      elements:
      - name: user-service
        path: services/user-service
      - name: order-service
        path: services/order-service
      - name: payment-service
        path: services/payment-service
  template:
    metadata:
      name: '{{name}}'
    spec:
      source:
        repoURL: https://github.com/my-org/microservices
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{name}}'
      syncPolicy:
        automated:
          prune: true
```

## 🎓 학습 리소스

### 📚 공식 문서
- [ArgoCD 공식 문서](https://argo-cd.readthedocs.io/)
- [GitHub 저장소](https://github.com/argoproj/argo-cd)
- [예제 애플리케이션](https://github.com/argoproj/argocd-example-apps)

### 🎥 튜토리얼
- [ArgoCD 시작하기](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [GitOps 워크플로우](https://argo-cd.readthedocs.io/en/stable/user-guide/auto_sync/)

### 🔧 실습 환경
- [Minikube + ArgoCD](https://argo-cd.readthedocs.io/en/stable/getting_started/#1-install-argocd)
- [Kind + ArgoCD](https://kind.sigs.k8s.io/docs/user/quick-start/)

## 🎯 결론

ArgoCD는 **Kubernetes 환경에서 GitOps를 구현하는 가장 강력한 도구** 중 하나입니다. Git을 단일 소스 of Truth로 사용하여 선언적이고 자동화된 배포를 제공하며, 직관적인 웹 UI와 강력한 기능으로 현대적인 CI/CD 파이프라인을 구축할 수 있습니다.

**주요 장점:**
- ✅ GitOps 원칙 완전 준수
- ✅ 강력한 웹 UI 및 CLI
- ✅ 다양한 소스 타입 지원
- ✅ 멀티 클러스터 관리
- ✅ 엔터프라이즈급 보안 및 RBAC
- ✅ 활발한 커뮤니티 및 지속적 개발 