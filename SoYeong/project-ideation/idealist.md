# 덜 붐비는 지하철 경로 추천 앱 (혼잡도 기반)

---

## 핵심 서비스 아이디어

- **지하철 실시간/예측 혼잡도**를 기반으로 **최적의 경로**를 사용자에게 추천
- **공공 데이터 + 사용자 제보** 혼합 방식으로 혼잡도 정확도 향상
- **시간대/요일/날씨** 패턴 반영한 AI 예측 기능
- 즐겨찾는 역/노선에 대한 **혼잡도 알림**
- **모바일 앱 중심 UX** + **GitHub Actions 기반 CI/CD 파이프라인** 구축 실습 포함

---

## MSA 서비스 구성

| 서비스명 | 기능 요약 |
|----------|-----------|
| `gateway-service` | 요청 라우팅, 인증 처리 |
| `user-service` | 사용자 관리, 즐겨찾기 역 설정 |
| `route-service` | 경로 탐색, 혼잡도 기반 최적 경로 정렬 |
| `congestion-service` | 혼잡도 수집 (공공 API + 사용자 제보 통합) |
| `prediction-service` | 시간대/요일/날씨 기반 혼잡도 예측 |
| `report-service` | 사용자 혼잡도 제보 수집 및 평가 |
| `notification-service` | 즐겨찾기 노선 알림 푸시 (FCM) |
| `frontend-app` | Flutter 기반 앱 UI |
| `admin-web` *(선택)* | 관리자용 웹 대시보드 (혼잡도 관리 등) |
| `infra/ci-cd` | GitHub Actions, Docker, 배포 스크립트 관리 |

---

## MSA 시스템 유형

| 항목 | 설명 |
|------|------|
| **도메인 분리형 MSA** | 서비스 책임별로 기능 나눔 (사용자, 경로, 제보, 예측 등) |
| **API Gateway 기반** | 모든 외부 요청은 gateway-service 통해 라우팅 |
| **서비스 독립 배포** | 각 서비스는 Docker 이미지로 분리 배포 |
| **CI/CD 중심 운영** | GitHub Actions 기반 자동 빌드/배포 |
| **K8s 생략 or 경량 사용** | 로컬 개발 또는 소형 클러스터에서 운영 가능 |
| **Config 중심 관리** | 혼잡 기준, 알림 임계값은 Git 또는 .env로 관리 가능 |

---

## 시스템 구성 (전체 흐름도)

```
subway-app
├── frontend-app/           # Flutter 모바일 앱
├── backend/
│   ├── user-service/
│   ├── route-service/
│   ├── congestion-service/
│   ├── prediction-service/
│   └── notification-service/
├── gateway-service/
├── report-service/         # 사용자 제보 수집 API
├── infra/
│   ├── docker-compose.yaml
│   └── k8s/               # (선택) 쿠버네티스 설정
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
```

---

## 추천 기술 스택

| 분야 | 기술 스택 |
|------|-----------|
| **앱 프론트엔드** | Flutter (Android/iOS) |
| **웹 관리자 (선택)** | React + Vite |
| **백엔드 API** | FastAPI (Python) or Spring Boot (Java) |
| **DB** | PostgreSQL (사용자, 경로), Redis (혼잡 캐시) |
| **AI 예측** | Scikit-learn, LightGBM |
| **지도 시각화** | Kakao Map SDK, Google Maps Flutter |
| **혼잡도 API** | 서울교통공사 혼잡도 오픈 API |
| **알림 시스템** | Firebase Cloud Messaging (FCM) |
| **배포** | Docker + Render / Railway / VPS |
| **CI/CD** | GitHub Actions |

---

## CI/CD 구성 요약

### CI (`ci.yml`)

- 각 PR 또는 push 시 자동으로 실행
- 앱 빌드 여부, 백엔드 테스트, 린트 체크 수행

```yaml
name: CI
on: [push, pull_request]
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install backend deps
        run: pip install -r backend/requirements.txt
      - name: Run tests
        run: pytest
```

---

### CD (`cd.yml`)

- `main` 브랜치로 push 시 자동 배포
- 백엔드 Docker 이미지 빌드 + 서버에 배포

```yaml
name: CD
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Docker Build
        run: docker build -t subway-app backend/
      - name: Docker Push
        run: docker tag subway-app ghcr.io/your-username/subway-app && docker push ghcr.io/your-username/subway-app
      - name: SSH to Server and Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull ghcr.io/your-username/subway-app
            docker stop subway-app || true
            docker rm subway-app || true
            docker run -d -p 80:8000 --name subway-app ghcr.io/your-username/subway-app
```
