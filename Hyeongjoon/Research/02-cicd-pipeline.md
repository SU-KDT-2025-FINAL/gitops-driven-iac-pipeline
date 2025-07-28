# 2. CI/CD 파이프라인 구성

## 목차
- [2.1 파이프라인 아키텍처](#21-파이프라인-아키텍처)
  - [2.1.1 강력한 CI/CD 도구 조합 추천](#211-강력한-cicd-도구-조합-추천)
- [2.2 자동화 워크플로우](#22-자동화-워크플로우)
- [2.3 품질 보증](#23-품질-보증)

---

## 🎯 CI/CD란?

**CI (Continuous Integration)**: 지속적 통합
- **쉬운 설명**: 여러 사람이 만든 코드를 자동으로 합쳐서 테스트하는 것
- **예시**: 마치 레고를 조립할 때마다 자동으로 "맞나?" 확인하는 것

**CD (Continuous Deployment)**: 지속적 배포
- **쉬운 설명**: 테스트가 통과하면 자동으로 서버에 올리는 것
- **예시**: 숙제를 다 쓰면 자동으로 선생님께 제출되는 것

---

## 2.1 파이프라인 아키텍처

**파이프라인이란?**
- **쉬운 설명**: 자동화된 공장 컨베이어 벨트와 같아요
- **과정**: 코드 작성 → 테스트 → 빌드 → 배포

**주요 도구들:**

**1. GitHub Actions / GitLab CI**
- **역할**: 코드 변경을 감지하고 자동으로 작업 시작
- **쉬운 설명**: 마치 도어벨처럼 "누가 왔어!" 알려주는 것
- **실제 예시**:
  ```yaml
  # GitHub Actions 예시
  name: 자동 빌드 및 테스트
  on: [push]  # 코드를 올리면 자동 실행
  jobs:
    test:
      runs-on: ubuntu-latest
      steps:
      - name: 코드 테스트
        run: npm test
  ```

**2. ArgoCD / Flux**
- **역할**: Git에 있는 설정을 서버에 자동으로 적용
- **쉬운 설명**: 마치 동기화 앱처럼 Git과 서버를 자동으로 맞춰주는 것
- **실제 예시**:
  ```yaml
  # ArgoCD 예시
  apiVersion: argoproj.io/v1alpha1
  kind: Application
  metadata:
    name: my-app
  spec:
    source:
      repoURL: https://github.com/my-repo
      path: k8s/
    destination:
      server: https://kubernetes.default.svc
      namespace: production
  ```

**3. Helm Charts**
- **역할**: 애플리케이션을 패키지로 묶어서 배포
- **쉬운 설명**: 마치 앱스토어에서 앱을 설치하는 것처럼 간단하게 배포
- **실제 예시**:
  ```yaml
  # Helm Chart 예시
  apiVersion: v2
  name: my-web-app
  version: 1.0.0
  dependencies:
  - name: nginx
    version: 1.0.0
  - name: mysql
    version: 1.0.0
  ```

**4. Container Registry**
- **역할**: Docker 이미지를 저장하고 관리
- **쉬운 설명**: 마치 앱스토어처럼 완성된 앱을 보관하는 창고
- **예시**: Docker Hub, AWS ECR, Google Container Registry

---

### 2.1.1 강력한 CI/CD 도구 조합 추천

#### 🏆 엔터프라이즈급 조합

**1. Jenkins + ArgoCD + Helm**
- **CI**: Jenkins (복잡한 파이프라인, 다양한 플러그인)
- **CD**: ArgoCD (Kubernetes GitOps 배포)
- **패키징**: Helm (애플리케이션 패키징)
- **장점**: 
  - Jenkins의 강력한 파이프라인 기능
  - ArgoCD의 GitOps 원칙 준수
  - Helm의 표준화된 배포 패키지
- **사용 시기**: 대규모 엔터프라이즈, 복잡한 배포 요구사항

**2. GitLab CI + ArgoCD + Kustomize**
- **CI**: GitLab CI (GitLab 통합, 강력한 파이프라인)
- **CD**: ArgoCD (Kubernetes GitOps 배포)
- **패키징**: Kustomize (환경별 설정 관리)
- **장점**:
  - GitLab의 통합된 개발 환경
  - ArgoCD의 멀티 클러스터 관리
  - Kustomize의 환경별 오버레이
- **사용 시기**: GitLab 기반 개발팀, 환경별 설정 관리

**3. Azure DevOps + ArgoCD + Terraform**
- **CI**: Azure DevOps (Microsoft 생태계 통합)
- **CD**: ArgoCD (Kubernetes GitOps 배포)
- **IaC**: Terraform (인프라 코드 관리)
- **장점**:
  - Azure 생태계 완벽 통합
  - ArgoCD의 GitOps 배포
  - Terraform의 멀티 클라우드 지원
- **사용 시기**: Azure 기반 환경, 멀티 클라우드 전략

#### 🚀 클라우드 네이티브 조합

**4. GitHub Actions + ArgoCD + Helm**
- **CI**: GitHub Actions (GitHub 통합, 간편한 설정)
- **CD**: ArgoCD (Kubernetes GitOps 배포)
- **패키징**: Helm (표준화된 차트)
- **장점**:
  - GitHub와 완벽 통합
  - 빠른 설정 및 사용
  - ArgoCD의 강력한 웹 UI
- **사용 시기**: GitHub 기반 프로젝트, 빠른 구축 필요

**5. CircleCI + Flux + Kustomize**
- **CI**: CircleCI (클라우드 기반, 빠른 빌드)
- **CD**: Flux (GitOps Kubernetes 배포)
- **패키징**: Kustomize (환경별 설정)
- **장점**:
  - CircleCI의 빠른 빌드 속도
  - Flux의 경량 GitOps
  - Kustomize의 유연한 설정
- **사용 시기**: 클라우드 우선 전략, 빠른 개발 사이클

**6. Tekton + ArgoCD + Helm**
- **CI**: Tekton (Kubernetes 네이티브 CI/CD)
- **CD**: ArgoCD (GitOps Kubernetes 배포)
- **패키징**: Helm (표준 차트)
- **장점**:
  - Kubernetes 네이티브 CI/CD
  - 클라우드 네이티브 아키텍처
  - 확장 가능한 파이프라인
- **사용 시기**: Kubernetes 중심 환경, 클라우드 네이티브 전략

#### 🎯 특화 조합

**7. Jenkins + Spinnaker + Helm**
- **CI**: Jenkins (복잡한 빌드 프로세스)
- **CD**: Spinnaker (고급 배포 전략)
- **패키징**: Helm (애플리케이션 패키징)
- **장점**:
  - Jenkins의 강력한 빌드 기능
  - Spinnaker의 고급 배포 전략 (Blue-Green, Canary)
  - Helm의 표준화
- **사용 시기**: 복잡한 배포 전략, 무중단 배포 요구

**8. GitHub Actions + Flux + Terraform**
- **CI**: GitHub Actions (GitHub 통합)
- **CD**: Flux (경량 GitOps)
- **IaC**: Terraform (인프라 관리)
- **장점**:
  - GitHub 생태계 통합
  - Flux의 경량 GitOps
  - Terraform의 인프라 자동화
- **사용 시기**: 인프라 자동화, 경량 GitOps

**9. GitLab CI + Flux + Kustomize**
- **CI**: GitLab CI (통합 개발 환경)
- **CD**: Flux (경량 GitOps)
- **패키징**: Kustomize (환경별 설정)
- **장점**:
  - GitLab의 통합 환경
  - Flux의 경량 GitOps
  - Kustomize의 유연성
- **사용 시기**: GitLab 기반, 경량 GitOps

#### 🔧 하이브리드 조합

**10. Jenkins + GitHub Actions + ArgoCD**
- **CI**: Jenkins (복잡한 빌드) + GitHub Actions (간단한 자동화)
- **CD**: ArgoCD (Kubernetes GitOps 배포)
- **장점**:
  - Jenkins의 강력한 빌드 기능
  - GitHub Actions의 간편한 자동화
  - ArgoCD의 GitOps 배포
- **사용 시기**: 복잡한 빌드 + 간단한 자동화 혼재

**11. Azure DevOps + GitHub Actions + Flux**
- **CI**: Azure DevOps (Azure 통합) + GitHub Actions (오픈소스)
- **CD**: Flux (경량 GitOps)
- **장점**:
  - Azure 생태계 통합
  - GitHub Actions의 오픈소스 지원
  - Flux의 경량 GitOps
- **사용 시기**: Azure + 오픈소스 혼재 환경

#### 📊 조합별 비교표

| 조합 | CI 도구 | CD 도구 | 패키징 | 복잡도 | 적합 규모 | 주요 장점 |
|------|---------|---------|--------|--------|-----------|-----------|
| **Jenkins + ArgoCD** | Jenkins | ArgoCD | Helm | 높음 | 대규모 | 강력한 파이프라인, GitOps |
| **GitHub Actions + ArgoCD** | GitHub Actions | ArgoCD | Helm | 낮음 | 중소규모 | 빠른 설정, GitHub 통합 |
| **GitLab CI + ArgoCD** | GitLab CI | ArgoCD | Kustomize | 중간 | 중규모 | 통합 환경, 환경별 설정 |
| **Azure DevOps + ArgoCD** | Azure DevOps | ArgoCD | Terraform | 중간 | 대규모 | Azure 통합, IaC |
| **CircleCI + Flux** | CircleCI | Flux | Kustomize | 낮음 | 중소규모 | 빠른 빌드, 경량 GitOps |
| **Tekton + ArgoCD** | Tekton | ArgoCD | Helm | 중간 | 중규모 | K8s 네이티브, 확장성 |

#### 🎯 선택 가이드

**규모별 추천:**
- **소규모 (1-10명)**: GitHub Actions + ArgoCD
- **중규모 (10-50명)**: GitLab CI + ArgoCD 또는 CircleCI + Flux
- **대규모 (50명+)**: Jenkins + ArgoCD 또는 Azure DevOps + ArgoCD

**요구사항별 추천:**
- **빠른 구축**: GitHub Actions + ArgoCD
- **복잡한 파이프라인**: Jenkins + ArgoCD
- **클라우드 네이티브**: Tekton + ArgoCD
- **Azure 환경**: Azure DevOps + ArgoCD
- **경량 GitOps**: CircleCI + Flux
- **고급 배포**: Jenkins + Spinnaker

---

## 2.2 자동화 워크플로우

**전체 과정을 게임으로 비유하면:**

### 1. Pull Request 기반 검증
- **쉬운 설명**: 숙제를 제출하기 전에 친구가 검토해주는 것
- **과정**:
  1. 개발자가 코드 작성
  2. Pull Request 생성 (숙제 제출)
  3. 자동 테스트 실행 (답안 검토)
  4. 코드 리뷰 (친구가 검토)
  5. 승인 후 머지 (합격!)

### 2. Merge 시 자동 배포
- **쉬운 설명**: 합격하면 자동으로 서버에 올라가는 것
- **과정**:
  ```mermaid
  코드 작성 → Pull Request → 테스트 통과 → 승인 → 자동 배포
  ```

### 3. Rollback 메커니즘
- **쉬운 설명**: 문제가 생기면 이전 버전으로 되돌리는 것
- **예시**: 게임에서 세이브 포인트로 돌아가는 것
- **실제 상황**:
  - 새 버전 배포 후 오류 발생
  - 자동으로 이전 버전으로 복구
  - 서비스 중단 없이 문제 해결

### 4. Blue-Green 배포
- **쉬운 설명**: 새 버전을 미리 준비해두고 한 번에 바꾸는 것
- **예시**: 
  - Blue: 현재 운영 중인 버전
  - Green: 새로 준비한 버전
  - 트래픽을 Blue에서 Green으로 한 번에 전환

---

## 2.3 품질 보증

### 1. 정적 코드 분석
- **쉬운 설명**: 코드를 실행하지 않고도 문제를 찾는 것
- **예시**: 문법 검사기처럼 코드의 문법과 스타일을 검사
- **도구**: SonarQube, CodeQL
- **실제 예시**:
  ```javascript
  // 나쁜 예시 (정적 분석에서 잡힘)
  if (user = admin) {  // = 대신 == 사용해야 함
    console.log("관리자입니다");
  }
  
  // 좋은 예시
  if (user == admin) {
    console.log("관리자입니다");
  }
  ```

### 2. 보안 스캔
- **쉬운 설명**: 코드에 보안 위험이 있는지 검사하는 것
- **예시**: 바이러스 검사처럼 악성 코드나 취약점을 찾음
- **도구**: Trivy, Snyk
- **실제 예시**:
  ```javascript
  // 보안 위험 (SQL Injection)
  const query = "SELECT * FROM users WHERE id = " + userId;
  
  // 안전한 방법
  const query = "SELECT * FROM users WHERE id = ?";
  ```

### 3. 인프라 테스트
- **쉬운 설명**: 서버 설정이 제대로 되었는지 테스트하는 것
- **예시**: 집을 지은 후 안전 검사를 하는 것
- **도구**: Terratest, Kitchen-Terraform
- **실제 예시**:
  ```go
  // Terratest 예시
  func TestTerraformBasicExample(t *testing.T) {
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
      TerraformDir: "../examples/terraform-basic-example",
    })
    
    defer terraform.Destroy(t, terraformOptions)
    terraform.InitAndApply(t, terraformOptions)
  }
  ```

### 4. 성능 테스트
- **쉬운 설명**: 많은 사람이 동시에 사용해도 잘 작동하는지 테스트
- **예시**: 다리 위에 많은 사람이 올라가도 무너지지 않는지 테스트
- **도구**: Apache JMeter, K6
- **실제 예시**:
  ```javascript
  // K6 성능 테스트 예시
  import http from 'k6/http';
  import { check } from 'k6';
  
  export default function() {
    const response = http.get('https://my-website.com');
    check(response, {
      'status is 200': (r) => r.status === 200,
      'response time < 500ms': (r) => r.timings.duration < 500,
    });
  }
  ```

---

## 🎮 전체 과정을 게임으로 비유하면:

**1단계: 개발 (코딩)**
- 게임 캐릭터를 만드는 것

**2단계: 테스트 (검증)**
- 캐릭터가 제대로 작동하는지 확인하는 것

**3단계: 빌드 (패키징)**
- 게임을 실행 파일로 만드는 것

**4단계: 배포 (설치)**
- 게임을 서버에 올려서 사람들이 사용할 수 있게 하는 것

**5단계: 모니터링 (관찰)**
- 게임이 잘 작동하는지 지켜보는 것

---

**이전 문서**: [1. 핵심 전략 및 아키텍처](./01-core-strategy.md)  
**다음 문서**: [3. 인프라 코드 관리](./03-infrastructure-management.md) 