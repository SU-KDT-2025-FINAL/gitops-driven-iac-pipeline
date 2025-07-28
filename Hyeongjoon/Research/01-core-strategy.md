# 1. 핵심 전략 및 아키텍처

## 목차
- [1.1 GitOps 원칙 적용](#11-gitops-원칙-적용)
  - [단일 소스 of Truth](#단일-소스-of-truth)
  - [선언적 구성](#선언적-구성)
  - [자동화된 동기화](#자동화된-동기화)
  - [감사 추적](#감사-추적)
- [1.2 멀티 환경 전략](#12-멀티-환경-전략)
  - [단일 소스 of Truth vs 멀티 환경 전략 비교](#단일-소스-of-truth-vs-멀티-환경-전략-비교)
  - [실제 적용 예시](#실제-적용-예시)
  - [서비스별 세분화 레포지토리 예시](#서비스별-세분화-레포지토리-예시)
  - [데이터베이스별 세분화 레포지토리 예시](#데이터베이스별-세분화-레포지토리-예시)
- [1.3 아키텍처 패턴](#13-아키텍처-패턴)

---

## 1.1 GitOps 원칙 적용

### 단일 소스 of Truth
**단일 소스 of Truth**: Git 저장소를 모든 인프라 변경의 단일 소스로 활용
=> 사용자 서비스 포트 : 8080, 주문 서비스 : 포트 8081 과 같이 각 서비스마다 DB 연결정보가 다르게 설정되고 
     로드 밸런서 설정 또한 다른 파일에 유지되어있을 경우 배포 시 설정 누락, DB 연결정보 불일치와 같은 여러문제들이 발생 할 수 있기 때문.

**환경별 폴더 구조 예시:**
```
config/
├── dev/
│   ├── db/
│   │   └── db.config          # 개발용 DB 설정
│   ├── services/
│   │   ├── user-service.yml   # 개발용 사용자 서비스 설정
│   │   ├── order-service.yml  # 개발용 주문 서비스 설정
│   │   └── payment-service.yml # 개발용 결제 서비스 설정
│   └── infrastructure/
│       └── loadbalancer.yml   # 개발용 로드밸런서 설정
├── staging/
│   ├── db/
│   │   └── db.config          # 스테이징용 DB 설정
│   ├── services/
│   │   ├── user-service.yml   # 스테이징용 사용자 서비스 설정
│   │   ├── order-service.yml  # 스테이징용 주문 서비스 설정
│   │   └── payment-service.yml # 스테이징용 결제 서비스 설정
│   └── infrastructure/
│       └── loadbalancer.yml   # 스테이징용 로드밸런서 설정
└── prod/
    ├── db/
    │   └── db.config          # 운영용 DB 설정
    ├── services/
    │   ├── user-service.yml   # 운영용 사용자 서비스 설정
    │   ├── order-service.yml  # 운영용 주문 서비스 설정
    │   └── payment-service.yml # 운영용 결제 서비스 설정
    └── infrastructure/
        └── loadbalancer.yml   # 운영용 로드밸런서 설정
```

### 선언적 구성
**선언적 구성**: Infrastructure as Code를 통한 명확한 상태 정의

**선언적 구성이란?**
- **쉬운 설명**: "어떻게"가 아닌 "무엇을" 원하는지 말하는 방식
- **예시**: "웹서버 3대, 데이터베이스 1대, 로드밸런서 1대가 있어야 해"

**Infrastructure as Code (IaC)란?**
- **쉬운 설명**: 인프라를 코드로 관리하는 것 (설정 파일로 서버를 만드는 것)
- **예시**: Terraform, Kubernetes YAML 파일 등

**명령형 vs 선언형 비교:**

❌ **명령형 (Imperative) - "어떻게"**
```bash
# 1. 서버를 만들어라
# 2. 웹서버 소프트웨어를 설치해라  
# 3. 포트 80을 열어라
# 4. 방화벽을 설정해라
# 5. 로드밸런서를 연결해라
```

✅ **선언형 (Declarative) - "무엇을"**
```yaml
# 원하는 상태만 정의
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  replicas: 3          # 웹서버 3대
  ports:
  - port: 80           # 포트 80
  selector:
    app: web-app
---
apiVersion: v1
kind: Service  
metadata:
  name: db-service
spec:
  replicas: 1          # 데이터베이스 1대
  ports:
  - port: 3306         # 포트 3306
```

**왜 선언적이 좋을까?**
1. **간단함**: 원하는 결과만 말하면 됨
2. **자동화**: 시스템이 알아서 필요한 작업을 수행
3. **일관성**: 항상 같은 결과를 보장
4. **유지보수**: 설정만 바꾸면 자동으로 반영

### 자동화된 동기화
**자동화된 동기화**: Git 변경사항이 자동으로 클러스터에 반영되도록 구성

### 감사 추적
**감사 추적**: 모든 변경사항에 대한 완전한 감사 로그 제공

**개인적인 설명**: 
위의 내용은 IT 인프라 운영을 위한 기술 문서의 예시이고 명령형 예시는 AI 요청 프롬프트가 아님.

---

## 1.2 멀티 환경 전략

### 단일 소스 of Truth vs 멀티 환경 전략 비교

| 구분 | 단일 소스 of Truth | 멀티 환경 전략 |
|------|-------------------|----------------|
| **관리 단위** | 설정 파일별 분리 | 환경별 분리 |
| **구조** | 같은 환경의 설정들을 모아서 관리 | 서로 다른 환경을 별도로 관리 |
| **예시** | `config/dev/` 안에 모든 개발 설정 | `dev-repo/`, `staging-repo/`, `prod-repo/` |
| **목적** | 설정의 일관성과 중앙화 | 환경별 독립성과 격리 |

**공통점**: 환경별로 나누어 관리한다는 기본 원리

**차이점**:
- **단일 소스**: 같은 환경 내에서 설정들을 모아서 관리 (폴더 단위)
- **멀티 환경**: 서로 다른 환경을 별도 저장소로 관리 (저장소 단위)

### 실제 적용 예시

**방법 1: 단일 저장소 + 폴더 분리 (단일 소스 of Truth)**
```
my-infra-repo/
├── dev/
│   ├── db.config
│   ├── services/
│   └── infrastructure/
├── staging/
│   ├── db.config
│   ├── services/
│   └── infrastructure/
└── prod/
    ├── db.config
    ├── services/
    └── infrastructure/
```

**방법 2: 환경별 저장소 분리 (멀티 환경 전략)**

**실제 프로젝트 환경별 레포지토리 예시:**

**개발 환경 레포지토리들:**
```
dev-frontend-repo/      # 프론트엔드 개발 환경
├── react-app/
├── vue-app/
└── angular-app/

dev-backend-repo/       # 백엔드 개발 환경
├── user-service/
├── order-service/
├── payment-service/
└── notification-service/

dev-database-repo/      # 데이터베이스 개발 환경
├── mysql/
├── redis/
├── mongodb/
└── elasticsearch/

dev-infrastructure-repo/ # 인프라 개발 환경
├── kubernetes/
├── docker/
├── terraform/
└── monitoring/
```

**스테이징 환경 레포지토리들:**
```
staging-frontend-repo/   # 프론트엔드 스테이징 환경
├── react-app/
├── vue-app/
└── angular-app/

staging-backend-repo/    # 백엔드 스테이징 환경
├── user-service/
├── order-service/
├── payment-service/
└── notification-service/

staging-database-repo/   # 데이터베이스 스테이징 환경
├── mysql/
├── redis/
├── mongodb/
└── elasticsearch/

staging-infrastructure-repo/ # 인프라 스테이징 환경
├── kubernetes/
├── docker/
├── terraform/
└── monitoring/
```

**운영 환경 레포지토리들:**
```
prod-frontend-repo/      # 프론트엔드 운영 환경
├── react-app/
├── vue-app/
└── angular-app/

prod-backend-repo/       # 백엔드 운영 환경
├── user-service/
├── order-service/
├── payment-service/
└── notification-service/

prod-database-repo/      # 데이터베이스 운영 환경
├── mysql/
├── redis/
├── mongodb/
└── elasticsearch/

prod-infrastructure-repo/ # 인프라 운영 환경
├── kubernetes/
├── docker/
├── terraform/
└── monitoring/
```

### 서비스별 세분화 레포지토리 예시
```
user-service-dev-repo/   # 사용자 서비스 개발 환경
├── api/
├── database/
├── cache/
└── monitoring/

order-service-dev-repo/  # 주문 서비스 개발 환경
├── api/
├── database/
├── cache/
└── monitoring/

payment-service-dev-repo/ # 결제 서비스 개발 환경
├── api/
├── database/
├── cache/
└── monitoring/
```

### 데이터베이스별 세분화 레포지토리 예시

**MySQL 전용 레포지토리:**
```
mysql-dev-repo/          # MySQL 개발 환경
├── schemas/             # 데이터베이스 스키마 정의
│   ├── user_schema.sql
│   ├── order_schema.sql
│   └── payment_schema.sql
├── migrations/          # 데이터베이스 마이그레이션 파일
│   ├── 001_create_users_table.sql
│   ├── 002_create_orders_table.sql
│   └── 003_add_payment_index.sql
├── seeds/               # 초기 데이터 (테스트용)
│   ├── users_seed.sql
│   ├── products_seed.sql
│   └── test_data.sql
├── backups/             # 백업 스크립트 및 정책
│   ├── backup_scripts/
│   ├── restore_scripts/
│   └── backup_schedule.yml
├── configurations/      # 데이터베이스 설정
│   ├── my.cnf
│   ├── replication.conf
│   └── performance.conf
└── monitoring/          # 모니터링 및 알림
    ├── queries/
    ├── dashboards/
    └── alerts/
```

**Redis 전용 레포지토리:**
```
redis-dev-repo/          # Redis 개발 환경
├── configurations/      # Redis 설정 파일
│   ├── redis.conf
│   ├── sentinel.conf
│   └── cluster.conf
├── scripts/             # Redis 스크립트
│   ├── lua_scripts/
│   ├── backup_scripts/
│   └── maintenance_scripts/
├── data_structures/     # 데이터 구조 정의
│   ├── cache_keys.yml
│   ├── session_config.yml
│   └── queue_config.yml
└── monitoring/          # Redis 모니터링
    ├── metrics/
    ├── dashboards/
    └── alerts/
```

**MongoDB 전용 레포지토리:**
```
mongodb-dev-repo/        # MongoDB 개발 환경
├── collections/         # 컬렉션 정의
│   ├── users.json
│   ├── orders.json
│   └── products.json
├── indexes/             # 인덱스 정의
│   ├── user_indexes.js
│   ├── order_indexes.js
│   └── product_indexes.js
├── migrations/          # MongoDB 마이그레이션
│   ├── 001_add_user_fields.js
│   ├── 002_update_order_schema.js
│   └── 003_create_indexes.js
├── backups/             # 백업 설정
│   ├── backup_config.yml
│   ├── restore_scripts/
│   └── retention_policy.yml
├── configurations/      # MongoDB 설정
│   ├── mongod.conf
│   ├── replica_set.conf
│   └── sharding.conf
└── monitoring/          # 모니터링 설정
    ├── oplog_monitoring/
    ├── performance_queries/
    └── health_checks/
```

**PostgreSQL 전용 레포지토리:**
```
postgresql-dev-repo/     # PostgreSQL 개발 환경
├── schemas/             # 스키마 정의
│   ├── public/
│   ├── auth/
│   └── analytics/
├── migrations/          # 마이그레이션 파일
│   ├── 001_initial_schema.sql
│   ├── 002_add_constraints.sql
│   └── 003_create_functions.sql
├── functions/           # 저장 프로시저 및 함수
│   ├── user_functions.sql
│   ├── order_functions.sql
│   └── analytics_functions.sql
├── triggers/            # 트리거 정의
│   ├── audit_triggers.sql
│   ├── notification_triggers.sql
│   └── validation_triggers.sql
├── configurations/      # PostgreSQL 설정
│   ├── postgresql.conf
│   ├── pg_hba.conf
│   └── recovery.conf
└── monitoring/          # 모니터링 설정
    ├── slow_query_logs/
    ├── performance_views/
    └── replication_status/
```

**공존 가능한 레포지토리 조합:**
- ✅ 프론트엔드 + 백엔드 (같은 환경)
- ✅ 백엔드 + 데이터베이스 (같은 환경)
- ✅ 인프라 + 모니터링 (같은 환경)
- ✅ 서비스별 분리 (독립적 운영 가능)

**공존하기 어려운 레포지토리 (제외):**
- ❌ 개발 + 운영 환경 (보안상 분리 필요)
- ❌ 스테이징 + 운영 환경 (데이터 격리 필요)
- ❌ 서로 다른 데이터베이스 타입 (설정 충돌 가능)

- **환경별 브랜치 전략**: `main`, `staging`, `development` 브랜치 활용
- **환경 격리**: 각 환경별 독립적인 네임스페이스 및 리소스 관리
- **프로모션 워크플로우**: 개발 → 스테이징 → 프로덕션 순차적 배포

---

## 1.3 아키텍처 패턴
- **Hub-Spoke 모델**: 중앙 집중식 GitOps 관리
- **Multi-Cluster 관리**: 여러 클러스터에 대한 통합 관리
- **GitOps Operator 패턴**: ArgoCD, Flux 등을 활용한 자동화

---

**다음 문서**: [2. CI/CD 파이프라인 구성](./02-cicd-pipeline.md) 