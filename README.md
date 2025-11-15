# 📊 Performance Test Reports

이 저장소는 **Claude Code 기반 자동화 테스트 환경**을 활용하여 수행된 다양한 시스템 성능 분석 문서를 포함하고 있습니다.  
각 테스트는 **Elasticsearch 기반 하이브리드 검색**, **SSE 실시간 알림 시스템**, **Redis Pub/Sub 기반 확장형 메시징**, **SSE 부하 테스트** 등 실서비스 환경에서 자주 사용되는 컴포넌트들을 대상으로 진행되었습니다.

테스트의 핵심 목적은 다음과 같습니다:

- 실제 트래픽 패턴을 시뮬레이션하여 **시스템의 성능 한계 파악**
- 구성요소별 **병목 구간 식별 및 개선 전략 제시**
- 대규모 트래픽 대비 **확장성 및 안정성 검증**
- 반복 가능한 **자동화된 성능 측정 환경 구축**

---

# 📁 Repository Contents

아래는 저장소 내 주요 분석 문서와 그 내용 요약입니다.

---

## 1. ES하이브리드성능테스트.md  
(Elasticsearch + PostgreSQL 하이브리드 검색 성능 분석)  
<sup>참조: ES하이브리드성능테스트.md</sup>

본 문서는 ES와 DB를 결합한 **Hybrid Search 구조**에서의 성능을 측정한 테스트 리포트입니다.  
테스트 도구는 Locust 기반이며, 서버 상태 체크 / 부하 테스트 / 스트레스 테스트 등 다양한 실행 옵션을 제공합니다.

### 주요 내용
- **테스트 시나리오**  
  - 키워드 단독 검색 (40%)  
  - 키워드 + 날씨 필터 검색 (30%)  
  - 키워드 + 정렬 (20%)  
  - 커서 기반 페이지네이션 (10%)  
  - DB 단독 검색 대비 분석 (5%)
- **프로젝트 구조 및 실행 스크립트 안내**
- **테스트 명령어와 환경 변수 구성**
- **결과 파일(html, csv) 구조 설명**
- **Locust 기반 시나리오 커스터마이징 방법**
- **Troubleshooting & 성능 최적화 팁 제공**

---

## 2. README_SSE_LOCUST.md  
(SSE 기반 실시간 알림 시스템 부하 테스트 도구 설명서)  
<sup>참조: README_SSE_LOCUST.md</sup>

Redis Pub/Sub 기반의 **실시간 알림 SSE 시스템**과 Legacy SSE 시스템을 비교하기 위한 테스트 환경을 구성한 문서입니다.

### 주요 내용
- **SSE 성능 테스트 파일 구성**
  - sse_notification_test.py  
  - sse_load_test_scenarios.py  
  - run_sse_comparison.py  
  - 실시간 대시보드(sse_dashboard.py)
- **테스트 실행 방법**
  - Redis 방식 / Legacy 방식 / 혼합 방식
  - 웹 UI 기반 부하 테스트
  - 자동화된 비교 테스트
- **수집 메트릭**
  - SSE 연결 시간 / 메시지 지연시간 / 수신 성공률
  - HTTP 기본 지표(RPS, P95, P99 등)
- **예상 성능 지표 및 비교 데이터**
- **테스트 커스터마이징(가중치, spawn rate 등)**

---

## 3. SSE_PERFORMANCE_ANALYSIS_REPORT.md  
(SSE 시스템 구조 개선 및 성능 최적화 분석)  
<sup>참조: SSE_PERFORMANCE_ANALYSIS_REPORT.md</sup>

기존 Spring SSE 시스템의 **동기 처리 문제점을 분석**하고, 비동기 + 병렬 처리 구조를 적용했을 때의 성능 향상을 정리한 기술 보고서입니다.

### 주요 내용
- **기존 구조의 병목 분석**
  - 순차적 emitter 처리(forEach)
  - 블로킹 I/O로 인한 지연
  - 메모리 기반 emitter 저장소의 한계
- **개선된 비동기 아키텍처 설계**
  - AsyncConfig 기반 전용 스레드 풀 구성
  - CompletableFuture 기반 병렬 전송
  - 성능 메트릭 실시간 수집 시스템 추가
- **성능 개선 결과**
  - 처리량 최대 **600% 증가**
  - 평균 응답시간 **67% 감소**
  - 동시성 및 안정성 대폭 향상
- **향후 개선 방향**
  - Redis Pub/Sub 또는 Kafka 도입
  - Dead connection 관리 자동화
  - Prometheus/Grafana 기반 심층 모니터링

---

## 4. REDIS_PUBSUB_PERFORMANCE_ANALYSIS.md  
(Redis Pub/Sub 기반 확장형 SSE 아키텍처 성능 분석)  
<sup>참조: REDIS_PUBSUB_PERFORMANCE_ANALYSIS.md</sup>

Legacy 방식과 Redis Pub/Sub 방식의 **정교한 성능 비교**를 수행한 분석 문서입니다.

### 주요 내용
- **아키텍처 차이 비교**
  - Legacy: 단일 서버, 직접 호출, 최소 지연  
  - Redis 기반: 비동기 메시징, 다중 서버 확장
- **정량적 성능 비교**
  - 단일 알림 → Legacy 우위  
  - 다중 사용자, 브로드캐스트 → Redis 압도적 우위
- **주요 성능 결과**
  - 브로드캐스트 성능: **최대 93% 개선**
  - 동시성 처리: **84% 개선**
  - 분산 환경에서 선형적 확장성 확인
- **지연시간 분석(P50, P95, P99)**
- **Redis 도입이 필요한 기준 제시**
- **추천 아키텍처 및 최적화 전략 포함**

---

# 🧪 Test Environment

- **Load Testing Tools:** Claude Code, Locust  
- **Streaming:** Server-Sent Events (SSE)  
- **Messaging:** Redis Pub/Sub  
- **Search Engine:** Elasticsearch  
- **Monitoring:** Grafana, Kibana + System Metrics  

각 테스트는 서로 다른 목적을 가지지만 공통적으로 **실 환경의 트래픽 패턴을 모의하고 시스템 한계점을 분석하는 목적**을 갖습니다.

---

# 🎯 Repository Purpose

이 저장소는 다음 목표를 위해 구성되었습니다:

- 시스템 성능 측정 및 상관관계 파악  
- 성능 병목과 구조적 한계 분석  
- 개선 방향 제안 및 아키텍처 리팩터링 근거 확보  
- 반복 가능한 자동화된 테스트 환경 구축  

---

# 🔧 How to Use This Repository

1. 각 문서를 열어 구성 요소별 성능 분석 결과를 확인합니다.  
2. Locust 기반 테스트 환경이 필요한 경우 제공된 스크립트를 사용해 동일한 시나리오 실행이 가능합니다.  
3. 결과 데이터를 참고해 실제 서비스의 설정값 조정, 아키텍처 개선 등에 활용합니다.  
4. SSE, Redis Pub/Sub, ELK 기반 검색 등 여러 인프라 관점에서 교차 비교가 가능합니다.

---

# 📬 문의 / 개선 제안

문서 내용 보완, 테스트 시나리오 확장, 코드 개선 등 제안이 있다면 편하게 Issue를 등록해주세요.

---


