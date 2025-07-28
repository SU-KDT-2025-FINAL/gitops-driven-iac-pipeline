# 4. 보안 및 거버넌스

## 목차
- [4.1 접근 제어](#41-접근-제어)
- [4.2 시크릿 관리](#42-시크릿-관리)
- [4.3 컴플라이언스](#43-컴플라이언스)

---

## 4.1 접근 제어
- **RBAC**: Kubernetes Role-Based Access Control
- **Service Accounts**: 애플리케이션별 서비스 계정 관리
- **Network Policies**: 네트워크 트래픽 제어
- **Pod Security Standards**: Pod 보안 정책 적용

## 4.2 시크릿 관리
- **External Secrets Operator**: AWS Secrets Manager, HashiCorp Vault 연동
- **Sealed Secrets**: Git에 안전하게 저장 가능한 암호화된 시크릿
- **SOPS**: GitOps 환경에서의 시크릿 암호화
- **Vault Agent**: 동적 시크릿 생성 및 관리

## 4.3 컴플라이언스
- **Policy as Code**: OPA Gatekeeper, Kyverno를 통한 정책 적용
- **감사 로깅**: 모든 API 호출에 대한 감사 로그 수집
- **CIS 벤치마크**: 컨테이너 보안 표준 준수
- **정기 보안 스캔**: 취약점 정기 점검 및 패치

---

**이전 문서**: [3. 인프라 코드 관리](./03-infrastructure-management.md)  
**다음 문서**: [5. 모니터링 및 관찰성](./05-monitoring-observability.md) 