# 🚀 Locust SSE 알림 시스템 성능 테스트

Redis Pub/Sub vs Legacy 방식 SSE 알림 시스템의 정확한 성능 비교를 위한 Locust 기반 부하 테스트 도구입니다.

## 📋 테스트 구성

### 1. 핵심 테스트 파일

- **`sse_notification_test.py`**: 기본 SSE 알림 성능 테스트
- **`sse_load_test_scenarios.py`**: 고급 부하 테스트 시나리오
- **`run_sse_comparison.py`**: 자동화된 비교 테스트 실행기
- **`sse_dashboard.py`**: 실시간 성능 모니터링 대시보드

### 2. 의존성 파일

- **`requirements_sse.txt`**: SSE 테스트 전용 추가 라이브러리
- 기존 `requirements.txt`와 함께 설치 필요

## 🛠️ 설치 및 설정

### 1. 의존성 설치

```bash
# 기존 의존성 + SSE 테스트 의존성 설치
cd performance_test
pip install -r requirements.txt -r requirements_sse.txt
```

### 2. 서버 준비

Spring Boot 애플리케이션이 실행 중이어야 합니다:
```bash
# 애플리케이션 실행
./gradlew bootRun

# 또는 IDE에서 실행
```

## 🧪 테스트 실행 방법

### 1. 기본 성능 테스트

#### Redis Pub/Sub 방식 테스트
```bash
locust -f sse_notification_test.py NotificationUser --host=http://localhost:8080
```

#### Legacy 방식 테스트
```bash
locust -f sse_notification_test.py LegacyNotificationUser --host=http://localhost:8080
```

### 2. 고급 부하 테스트

#### Redis 방식 고급 테스트
```bash
locust -f sse_load_test_scenarios.py RedisNotificationUser --host=http://localhost:8080
```

#### 혼합 테스트 (50% Redis + 50% Legacy)
```bash
locust -f sse_load_test_scenarios.py RedisNotificationUser:LegacyNotificationUser --host=http://localhost:8080
```

### 3. 웹 UI 사용

```bash
# 웹 인터페이스로 실행
locust -f sse_load_test_scenarios.py --host=http://localhost:8080

# 브라우저에서 http://localhost:8089 접속하여 테스트 설정
```

### 4. 자동화된 비교 테스트

```bash
# 전체 비교 테스트 (약 10-15분 소요)
python run_sse_comparison.py --host=http://localhost:8080

# 빠른 테스트 (약 2분 소요)
python run_sse_comparison.py --host=http://localhost:8080 --quick
```

### 5. 실시간 모니터링 대시보드

```bash
# 대시보드 실행
python sse_dashboard.py

# 브라우저에서 http://localhost:8050 접속
```

## 📊 테스트 시나리오

### 1. 개인 알림 테스트 (`@task(5)`)
- 사용자별 개인 알림 전송
- 좋아요, 댓글, 팔로우 등 개인 알림
- 응답시간, 처리량 측정

### 2. 브로드캐스트 테스트 (`@task(1)`)
- 전체 사용자 대상 공지사항
- 시스템 알림, 이벤트 공지
- 대량 사용자 동시 전송 성능

### 3. SSE 연결 안정성 테스트
- 연결 유지 상태 확인
- 자동 재연결 테스트
- 메시지 수신 안정성

## 📈 수집되는 성능 지표

### 1. 기본 HTTP 메트릭
- **응답시간**: 평균, 최소, 최대, P95, P99
- **처리량**: 초당 요청 수 (RPS)
- **실패율**: 에러 발생 비율
- **동시 사용자**: 최대 동시 접속 수

### 2. SSE 전용 메트릭
- **연결 시간**: SSE 연결 설정 시간
- **메시지 지연시간**: 발송부터 수신까지 시간
- **메시지 수신율**: 전송 대비 수신 성공률
- **연결 안정성**: 연결 유지 시간, 재연결 횟수

### 3. 시스템 리소스 메트릭
- **CPU 사용률**: 서버 CPU 부하
- **메모리 사용량**: JVM 메모리 사용량
- **네트워크 I/O**: Redis 통신 부하

## 🔍 테스트 결과 분석

### 1. 자동 생성 리포트

테스트 완료 후 `results/` 디렉토리에 생성:

- **HTML 리포트**: 상세한 성능 그래프와 통계
- **CSV 데이터**: 원시 성능 데이터
- **JSON 요약**: 비교 분석 결과

### 2. 실시간 대시보드

- **실시간 성능 지표**: 3초마다 업데이트
- **비교 차트**: Redis vs Legacy 실시간 비교
- **시스템 모니터링**: CPU, 메모리 사용량 추적

### 3. 예상 성능 지표

| 시나리오 | Redis Pub/Sub | Legacy Direct | 예상 개선율 |
|---------|---------------|---------------|------------|
| **단일 알림** | ~8ms | ~3ms | -166% ⚠️ |
| **다중 사용자 (100명)** | ~150ms | ~400ms | +167% 🚀 |
| **브로드캐스트 (500명)** | ~50ms | ~800ms | +1500% 🚀 |
| **동시 부하 (200명)** | ~200ms | ~1200ms | +500% 🚀 |

## ⚙️ 테스트 설정 커스터마이징

### 1. 부하 설정 변경

`sse_load_test_scenarios.py`에서 수정:

```python
wait_time = between(0.5, 2.0)  # 작업 간 대기 시간
```

### 2. 테스트 시나리오 가중치

```python
@task(5)  # 높은 빈도
def send_personal_notification(self):

@task(1)  # 낮은 빈도
def send_broadcast_notification(self):
```

### 3. 타겟 서버 변경

```bash
# 다른 서버 테스트
locust -f sse_load_test_scenarios.py --host=http://your-server:8080
```

## 🚨 주의사항

### 1. 서버 준비사항

- Spring Boot 애플리케이션 실행 중
- Redis 서버 실행 중 (Redis 테스트 시)
- 충분한 서버 리소스 (CPU, 메모리)

### 2. 테스트 환경

- 안정적인 네트워크 환경
- 방화벽/보안 설정 확인
- 로컬 테스트 권장 (정확한 측정을 위해)

### 3. 리소스 제한

- 동시 사용자 수는 시스템 성능에 맞게 조절
- 테스트 시간이 길면 시스템 부하 증가
- 테스트 후 결과 파일 정리 필요

## 📝 사용 예제

### 예제 1: 빠른 성능 확인

```bash
# 10명 사용자, 30초 테스트
locust -f sse_load_test_scenarios.py RedisNotificationUser \
       --host=http://localhost:8080 \
       --users=10 \
       --spawn-rate=2 \
       --run-time=30s \
       --headless
```

### 예제 2: 상세 비교 분석

```bash
# 자동화된 전체 비교 테스트
python run_sse_comparison.py --host=http://localhost:8080

# 결과는 results/ 디렉토리에 저장됨
```

### 예제 3: 실시간 모니터링

```bash
# 1. 대시보드 실행
python sse_dashboard.py

# 2. 별도 터미널에서 부하 테스트
locust -f sse_load_test_scenarios.py --host=http://localhost:8080

# 3. 브라우저에서 실시간 성능 모니터링
```

## 🎯 분석 포인트

### 1. Redis 방식이 우수한 경우
- 다중 사용자 알림 (100명 이상)
- 브로드캐스트 성능
- 동시성 처리 능력
- 확장성 (수평 스케일링)

### 2. Legacy 방식이 우수한 경우
- 단일 알림 지연시간
- 메모리 효율성
- 단순한 구조
- 소규모 환경 (50명 미만)

이 테스트를 통해 **정확한 데이터 기반**으로 Redis Pub/Sub 도입 효과를 검증할 수 있습니다.