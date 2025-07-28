# 5. 모니터링 및 관찰성

## 목차
- [5.1 로깅 전략](#51-로깅-전략)
- [5.2 메트릭 수집](#52-메트릭-수집)
- [5.3 분산 추적](#53-분산-추적)

---

## 5.1 로깅 전략
- **Centralized Logging**: ELK Stack 또는 Grafana Loki 활용
- **Structured Logging**: JSON 형태의 구조화된 로그 출력
- **Log Aggregation**: Fluentd, Fluent Bit을 통한 로그 수집
- **Log Retention**: 비용 효율적인 로그 보관 정책

## 5.2 메트릭 수집
- **Prometheus**: 메트릭 수집 및 저장
- **Grafana**: 대시보드 및 알림 구성
- **Custom Metrics**: 애플리케이션별 커스텀 메트릭 정의
- **Service Mesh**: Istio, Linkerd를 통한 서비스 메트릭 수집

## 5.3 분산 추적
- **Jaeger**: 분산 추적 시스템
- **OpenTelemetry**: 표준화된 관찰성 데이터 수집
- **APM**: 애플리케이션 성능 모니터링
- **Error Tracking**: Sentry 등을 통한 에러 추적

---

**이전 문서**: [4. 보안 및 거버넌스](./04-security-governance.md)  
**다음 문서**: [6. 운영 및 유지보수](./06-operations-maintenance.md) 