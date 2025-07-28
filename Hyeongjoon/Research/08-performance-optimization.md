# 8. 성능 최적화

## 목차
- [8.1 클러스터 최적화](#81-클러스터-최적화)
  - [워크로드별 노드 풀 구성](#워크로드별-노드-풀-구성)
  - [네임스페이스별 리소스 제한](#네임스페이스별-리소스-제한)
  - [리소스 사용량 모니터링](#리소스-사용량-모니터링)
  - [실제 최적화 시나리오](#실제-최적화-시나리오)
  - [비용 최적화 효과](#비용-최적화-효과)
  - [자동 스케일링 설정](#자동-스케일링-설정)
  - [성능 모니터링 대시보드](#성능-모니터링-대시보드)
  - [클러스터 최적화 체크리스트](#클러스터-최적화-체크리스트)
- [8.2 애플리케이션 최적화](#82-애플리케이션-최적화)
- [8.3 비용 최적화](#83-비용-최적화)

---

## 8.1 클러스터 최적화

### 🏗️ **클러스터 최적화란?**
- **쉬운 설명**: 서버들을 효율적으로 사용해서 성능을 높이고 비용을 줄이는 것
- **예시**: 마치 아파트를 효율적으로 관리해서 모든 집이 잘 살 수 있게 하는 것

### 🎯 **워크로드별 노드 풀 구성**

**노드 풀이란?**
- **쉬운 설명**: 비슷한 성격의 서버들을 그룹으로 묶어서 관리하는 것
- **예시**: 아파트에서 1층은 상가, 2-5층은 일반 가정, 6층은 고급 가정으로 나누는 것

**왜 노드 풀을 나누나요?**
1. **비용 효율성**: 필요한 만큼만 서버 사용
2. **성능 최적화**: 각 작업에 맞는 서버 사용
3. **보안 강화**: 중요한 작업과 일반 작업 분리

**실제 노드 풀 구성 예시:**

**1. 일반 워크로드 노드 풀 (General Pool)**
```yaml
# 일반적인 웹 애플리케이션용
apiVersion: v1
kind: NodePool
metadata:
  name: general-pool
spec:
  nodeType: general
  resources:
    cpu: "2"        # CPU 2개
    memory: "4Gi"   # 메모리 4GB
  labels:
    workload-type: general
    cost-tier: standard
```

**2. 고성능 워크로드 노드 풀 (High-Performance Pool)**
```yaml
# 데이터베이스, AI/ML 작업용
apiVersion: v1
kind: NodePool
metadata:
  name: high-perf-pool
spec:
  nodeType: high-performance
  resources:
    cpu: "8"        # CPU 8개
    memory: "32Gi"  # 메모리 32GB
    gpu: "1"        # GPU 1개
  labels:
    workload-type: high-performance
    cost-tier: premium
```

**3. 스팟 인스턴스 노드 풀 (Spot Pool)**
```yaml
# 비용 절약용 (중요하지 않은 작업)
apiVersion: v1
kind: NodePool
metadata:
  name: spot-pool
spec:
  nodeType: spot
  resources:
    cpu: "1"        # CPU 1개
    memory: "2Gi"   # 메모리 2GB
  labels:
    workload-type: batch
    cost-tier: cheap
```

**워크로드별 노드 풀 매칭:**
```yaml
# 웹 애플리케이션 → 일반 노드 풀
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  template:
    spec:
      nodeSelector:
        workload-type: general    # 일반 노드 풀에 배치
      containers:
      - name: web-app
        resources:
          requests:
            cpu: "500m"           # CPU 0.5개 요청
            memory: "1Gi"         # 메모리 1GB 요청
          limits:
            cpu: "1000m"          # CPU 1개 제한
            memory: "2Gi"         # 메모리 2GB 제한
```

```yaml
# 데이터베이스 → 고성능 노드 풀
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  template:
    spec:
      nodeSelector:
        workload-type: high-performance    # 고성능 노드 풀에 배치
      containers:
      - name: database
        resources:
          requests:
            cpu: "4"              # CPU 4개 요청
            memory: "16Gi"        # 메모리 16GB 요청
          limits:
            cpu: "8"              # CPU 8개 제한
            memory: "32Gi"        # 메모리 32GB 제한
```

### 🏠 **네임스페이스별 리소스 제한**

**네임스페이스란?**
- **쉬운 설명**: 아파트의 층별 구분처럼, 프로젝트나 팀별로 공간을 나누는 것
- **예시**: 1층은 상가, 2층은 A팀, 3층은 B팀으로 나누는 것

**왜 리소스를 제한하나요?**
1. **비용 통제**: 팀별로 사용량 제한
2. **성능 보장**: 한 팀이 모든 자원을 사용하지 못하게 함
3. **보안 강화**: 팀간 격리

**네임스페이스별 리소스 제한 예시:**

**1. 개발팀 네임스페이스 (제한적)**
```yaml
# 개발팀은 적은 자원만 사용
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    cpu: "4"              # CPU 총 4개만 사용 가능
    memory: "8Gi"         # 메모리 총 8GB만 사용 가능
    pods: "10"            # 파드 10개만 생성 가능
    services: "5"         # 서비스 5개만 생성 가능
```

**2. 운영팀 네임스페이스 (충분한 자원)**
```yaml
# 운영팀은 충분한 자원 사용
apiVersion: v1
kind: ResourceQuota
metadata:
  name: prod-quota
  namespace: production
spec:
  hard:
    cpu: "16"             # CPU 총 16개 사용 가능
    memory: "64Gi"        # 메모리 총 64GB만 사용 가능
    pods: "50"            # 파드 50개 생성 가능
    services: "20"        # 서비스 20개 생성 가능
```

**3. 테스트팀 네임스페이스 (최소 자원)**
```yaml
# 테스트팀은 최소한의 자원만 사용
apiVersion: v1
kind: ResourceQuota
metadata:
  name: test-quota
  namespace: testing
spec:
  hard:
    cpu: "2"              # CPU 총 2개만 사용 가능
    memory: "4Gi"         # 메모리 총 4GB만 사용 가능
    pods: "5"             # 파드 5개만 생성 가능
    services: "3"         # 서비스 3개만 생성 가능
```

### 📊 **리소스 사용량 모니터링**

**1. 네임스페이스별 사용량 확인**
```bash
# 개발팀 자원 사용량 확인
kubectl describe resourcequota dev-quota -n development

# 출력 예시:
# Name:            dev-quota
# Namespace:       development
# Resource         Used    Hard
# --------         ---     ---
# cpu              2       4
# memory           4Gi     8Gi
# pods             5       10
# services         2       5
```

**2. 노드별 사용량 확인**
```bash
# 노드별 자원 사용량 확인
kubectl top nodes

# 출력 예시:
# NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
# node-1     2.5          25%    4Gi             40%
# node-2     1.8          18%    3Gi             30%
# node-3     3.2          32%    6Gi             60%
```

### 🎯 **실제 최적화 시나리오**

**시나리오 1: 웹 애플리케이션 배포**
```yaml
# 웹 애플리케이션은 일반 노드 풀에 배치
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      nodeSelector:
        workload-type: general    # 일반 노드 풀 선택
      containers:
      - name: web-app
        image: web-app:v1.2.0
        resources:
          requests:
            cpu: "500m"           # 최소 CPU 0.5개
            memory: "1Gi"         # 최소 메모리 1GB
          limits:
            cpu: "1000m"          # 최대 CPU 1개
            memory: "2Gi"         # 최대 메모리 2GB
        ports:
        - containerPort: 80
```

**시나리오 2: 데이터베이스 배포**
```yaml
# 데이터베이스는 고성능 노드 풀에 배치
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      nodeSelector:
        workload-type: high-performance    # 고성능 노드 풀 선택
      containers:
      - name: database
        image: mysql:8.0
        resources:
          requests:
            cpu: "4"              # 최소 CPU 4개
            memory: "16Gi"        # 최소 메모리 16GB
          limits:
            cpu: "8"              # 최대 CPU 8개
            memory: "32Gi"        # 최대 메모리 32GB
        ports:
        - containerPort: 3306
```

**시나리오 3: 배치 작업 배포**
```yaml
# 배치 작업은 스팟 노드 풀에 배치 (비용 절약)
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
  namespace: development
spec:
  template:
    spec:
      nodeSelector:
        workload-type: batch      # 스팟 노드 풀 선택
      containers:
      - name: batch-processor
        image: batch-processor:v1.0
        resources:
          requests:
            cpu: "1"              # 최소 CPU 1개
            memory: "2Gi"         # 최소 메모리 2GB
          limits:
            cpu: "2"              # 최대 CPU 2개
            memory: "4Gi"         # 최대 메모리 4GB
      restartPolicy: Never
```

### 💰 **비용 최적화 효과**

**최적화 전 vs 최적화 후:**

| 구분 | 최적화 전 | 최적화 후 | 절약 효과 |
|------|-----------|-----------|-----------|
| **웹 애플리케이션** | 고성능 노드 사용 | 일반 노드 사용 | 60% 비용 절약 |
| **데이터베이스** | 일반 노드 사용 | 고성능 노드 사용 | 성능 3배 향상 |
| **배치 작업** | 일반 노드 사용 | 스팟 노드 사용 | 80% 비용 절약 |
| **전체 비용** | $1000/월 | $400/월 | 60% 비용 절약 |

### 🔧 **자동 스케일링 설정**

**1. 수평 자동 스케일링 (HPA)**
```yaml
# CPU 사용량에 따라 자동으로 파드 수 조절
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2        # 최소 2개
  maxReplicas: 10       # 최대 10개
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70    # CPU 70% 사용 시 스케일링
```

**2. 클러스터 자동 스케일링 (CA)**
```yaml
# 노드 사용량에 따라 자동으로 노드 수 조절
apiVersion: autoscaling.k8s.io/v1
kind: ClusterAutoscaler
metadata:
  name: cluster-autoscaler
spec:
  scaleDown:
    enabled: true
    delayAfterAdd: 10m
    delayAfterDelete: 10s
    delayAfterFailure: 3m
  nodeGroups:
  - name: general-pool
    minSize: 2
    maxSize: 10
  - name: high-perf-pool
    minSize: 1
    maxSize: 5
```

### 📈 **성능 모니터링 대시보드**

**1. 노드 풀별 사용량 대시보드**
```yaml
# Grafana 대시보드 설정
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-dashboard
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "클러스터 최적화 대시보드",
        "panels": [
          {
            "title": "노드 풀별 CPU 사용량",
            "type": "graph",
            "targets": [
              {
                "expr": "sum(cpu_usage{node_pool=\"general\"})",
                "legendFormat": "일반 노드 풀"
              },
              {
                "expr": "sum(cpu_usage{node_pool=\"high-performance\"})",
                "legendFormat": "고성능 노드 풀"
              }
            ]
          },
          {
            "title": "네임스페이스별 메모리 사용량",
            "type": "graph",
            "targets": [
              {
                "expr": "sum(memory_usage{namespace=\"development\"})",
                "legendFormat": "개발팀"
              },
              {
                "expr": "sum(memory_usage{namespace=\"production\"})",
                "legendFormat": "운영팀"
              }
            ]
          }
        ]
      }
    }
```

### 🎯 **클러스터 최적화 체크리스트**

**정기 점검 항목:**
- [ ] 노드 풀이 워크로드에 적합하게 구성되었는가?
- [ ] 네임스페이스별 리소스 제한이 설정되었는가?
- [ ] 자동 스케일링이 올바르게 설정되었는가?
- [ ] 비용 효율적인 노드 타입을 사용하고 있는가?
- [ ] 사용하지 않는 리소스가 정리되었는가?
- [ ] 성능 모니터링이 설정되었는가?

이제 클러스터 최적화에 대해 완전히 이해하셨나요? 🎯✨

---

## 8.2 애플리케이션 최적화
- **Container Optimization**: 멀티스테이지 빌드 및 이미지 최적화
- **Resource Tuning**: CPU/Memory 요청 및 제한 최적화
- **Caching Strategy**: Redis, Memcached 등을 통한 캐싱
- **CDN Integration**: 정적 자원 전송 최적화

## 8.3 비용 최적화
- **Spot Instances**: AWS Spot Instance 활용
- **Reserved Instances**: 예약 인스턴스 구매
- **Auto Scaling**: 부하에 따른 자동 스케일링
- **Resource Monitoring**: 리소스 사용량 모니터링 및 최적화

---

**이전 문서**: [7. 팀 협업 및 워크플로우](./07-team-collaboration.md)  
**다음 문서**: [9. 재해 복구 및 백업](./09-disaster-recovery.md) 