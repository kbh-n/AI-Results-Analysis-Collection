# ES 하이브리드 검색 성능 테스트

이 프로젝트는 Elasticsearch와 PostgreSQL을 결합한 하이브리드 검색 시스템의 성능을 측정하기 위한 Locust 기반 성능 테스트 도구입니다.

## 📋 개요

### 테스트 대상
- **API 엔드포인트**: `GET /api/feeds`
- **검색 방식**: ES 하이브리드 검색 (키워드가 있으면 Elasticsearch + DB, 없으면 DB만)
- **주요 기능**: 키워드 검색, 날씨 필터, 정렬, 커서 기반 페이지네이션

### 테스트 시나리오
1. **키워드 단독 검색** (40%): ES 하이브리드 검색 성능 측정
2. **키워드 + 날씨 필터** (30%): 복합 필터 조건 성능 측정
3. **키워드 + 정렬** (20%): 정렬 옵션별 성능 비교
4. **페이지네이션** (10%): 커서 기반 페이지네이션 성능
5. **DB 단독 검색** (5%): 비교 기준용 (키워드 없음)

## 🚀 빠른 시작

### 1. 환경 설정
```bash
# 환경 설정 (가상환경 생성 + 패키지 설치)
./run_test.sh --setup

# 또는 수동 설정
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 2. 서버 상태 확인
```bash
# 대상 서버가 실행 중인지 확인
./run_test.sh --server-check
```

### 3. 테스트 실행

#### 빠른 테스트 (개발/디버깅용)
```bash
# 10명 사용자, 60초간 실행
./run_test.sh --quick
```

#### 부하 테스트 (일반적인 성능 측정)
```bash
# 100명 사용자, 5분간 실행
./run_test.sh --load
```

#### 스트레스 테스트 (한계 성능 측정)
```bash
# 200명 사용자, 10분간 실행
./run_test.sh --stress
```

#### 커스텀 테스트
```bash
# 사용자 정의 설정
./run_test.sh --users 50 --time 300s --host http://localhost:8080
```

## 📁 프로젝트 구조

```
performance_test/
├── config.py                          # 테스트 설정
├── requirements.txt                    # Python 의존성
├── run_test.sh                        # 쉘 실행 스크립트
├── run_hybrid_search_test.py          # Python 실행 스크립트
├── README.md                          # 이 파일
├── scenarios/                         # 테스트 시나리오
│   ├── hybrid_search_performance.py   # 메인 Locust 테스트
│   ├── auth_utils.py                  # JWT 인증 유틸리티
│   └── test_data.py                   # 테스트 데이터 생성기
└── results/                           # 테스트 결과 저장
    ├── *.html                         # HTML 리포트
    ├── *_stats.csv                    # 통계 데이터
    └── *_failures.csv                 # 실패 데이터
```

## 🔧 상세 사용법

### 명령어 옵션

#### 쉘 스크립트 (`./run_test.sh`)
```bash
# 기본 옵션
./run_test.sh [옵션]

# 주요 옵션
-u, --users USERS               # 동시 사용자 수
-t, --time TIME                 # 테스트 시간 (예: 300s, 5m)
--host HOST                     # 대상 호스트
--headless                      # 웹 UI 없이 실행

# 편의 명령어
--quick                         # 빠른 테스트
--load                          # 부하 테스트
--stress                        # 스트레스 테스트

# 유틸리티
--setup                         # 환경 설정
--check-deps                    # 의존성 확인
--server-check                  # 서버 상태 확인
--clean                         # 결과 파일 정리
--list-scenarios                # 시나리오 목록
```

#### Python 스크립트 (`python3 run_hybrid_search_test.py`)
```bash
# 기본 실행
python3 run_hybrid_search_test.py --quick

# 커스텀 설정
python3 run_hybrid_search_test.py \
    --users 100 \
    --spawn-rate 3 \
    --run-time 300s \
    --host http://localhost:8080 \
    --headless
```

### 환경 변수
```bash
export TARGET_HOST="http://localhost:8080"      # 대상 서버
export MAX_USERS="100"                          # 최대 사용자 수
export RUN_TIME="300s"                          # 테스트 시간
export DETAILED_LOGGING="true"                  # 상세 로깅
export SKIP_SERVER_CHECK="false"                # 서버 확인 건너뛰기
```

## 📊 결과 분석

### 생성되는 파일
- `*_report.html`: 상세 성능 리포트 (그래프 포함)
- `*_stats.csv`: 요청별 통계 데이터
- `*_failures.csv`: 실패한 요청 데이터

### 주요 메트릭
- **응답 시간**: 평균, 최소, 최대, 백분위수 (95%, 99%)
- **처리량**: 초당 요청 수 (RPS)
- **오류율**: 실패한 요청의 비율
- **동시 사용자**: 시간별 활성 사용자 수

### 성능 임계값
- **응답 시간**: 2초 이하 (설정 가능)
- **오류율**: 5% 이하
- **처리량**: 서버 사양에 따라 다름

## 🎯 테스트 시나리오 상세

### 1. 키워드 단독 검색 (40% 가중치)
```
GET /api/feeds?keywordLike=맑은날&limit=20&sortBy=createdAt&sortDirection=DESC
```
- ES에서 텍스트 검색 후 DB에서 메타데이터 조회
- 다양한 키워드 패턴 테스트 (인기/일반/롱테일)

### 2. 키워드 + 날씨 필터 (30% 가중치)
```
GET /api/feeds?keywordLike=봄옷&skyStatusEqual=CLEAR&limit=20
```
- 하이브리드 검색 + 메타데이터 필터링
- 복합 조건에서의 성능 측정

### 3. 키워드 + 정렬 (20% 가중치)
```
GET /api/feeds?keywordLike=티셔츠&sortBy=likeCount&sortDirection=DESC
```
- 좋아요순/최신순 정렬 성능 비교

### 4. 페이지네이션 (10% 가중치)
```
# 첫 페이지
GET /api/feeds?keywordLike=여름&limit=20

# 다음 페이지
GET /api/feeds?keywordLike=여름&cursor=2024-01-01T00:00:00Z&idAfter=uuid
```
- 커서 기반 페이지네이션 성능
- 연속된 페이지 요청 패턴

### 5. DB 단독 검색 (5% 가중치)
```
GET /api/feeds?skyStatusEqual=CLEAR&limit=20
```
- 키워드 없는 검색 (ES 미사용)
- 하이브리드 검색과 성능 비교

## 🛠 개발자 가이드

### 새로운 시나리오 추가
1. `scenarios/` 디렉토리에 새 Python 파일 생성
2. `HttpUser` 클래스를 상속받아 구현
3. `@task` 데코레이터로 테스크 정의
4. 가중치와 대기 시간 설정

```python
from locust import HttpUser, task, between

class CustomSearchUser(HttpUser):
    wait_time = between(1, 3)

    @task(50)  # 가중치 50%
    def custom_search(self):
        # 커스텀 테스트 로직
        pass
```

### 테스트 데이터 확장
`scenarios/test_data.py`에서 키워드, 필터 조건 등을 추가/수정할 수 있습니다.

### 설정 변경
`config.py`에서 다음 설정들을 조정할 수 있습니다:
- 부하 테스트 파라미터
- 성능 임계값
- API 엔드포인트
- 타임아웃 설정

## 🔍 트러블슈팅

### 일반적인 문제들

#### 1. 패키지 설치 오류
```bash
# Python 버전 확인 (3.8+ 권장)
python3 --version

# pip 업그레이드
pip install --upgrade pip

# 가상환경 재생성
rm -rf venv
./run_test.sh --setup
```

#### 2. 서버 연결 실패
```bash
# 서버 상태 확인
./run_test.sh --server-check

# 호스트 주소 확인
curl http://localhost:8080/api/feeds?limit=1

# 방화벽/포트 확인
netstat -tulpn | grep 8080
```

#### 3. 메모리 부족
```bash
# 사용자 수 줄이기
./run_test.sh --users 50 --time 180s

# 시스템 리소스 확인
top
free -h
```

#### 4. 높은 오류율
- 서버 로그 확인
- 데이터베이스 연결 상태 확인
- Elasticsearch 클러스터 상태 확인
- 네트워크 지연 확인

### 로그 및 디버깅

#### 상세 로깅 활성화
```bash
export DETAILED_LOGGING=true
./run_test.sh --load
```

#### Locust 웹 UI 사용
```bash
# 웹 UI로 실행 (헤드리스 모드 제외)
./run_test.sh --users 50 --time 600s
# http://localhost:8089 접속
```

## 📈 성능 최적화 팁

### 서버 측
1. **DB 인덱스 최적화**: 검색 조건에 맞는 인덱스 설정
2. **ES 클러스터 튜닝**: 샤드/레플리카 설정, 메모리 할당
3. **캐시 활용**: Redis 등을 활용한 쿼리 캐싱
4. **커넥션 풀**: DB/ES 커넥션 풀 크기 조정

### 클라이언트 측
1. **연결 재사용**: Keep-Alive 설정
2. **타임아웃 조정**: 네트워크 환경에 맞는 타임아웃
3. **부하 분산**: 여러 클라이언트에서 테스트 실행

## 📞 지원

### 이슈 리포팅
성능 테스트 관련 문제나 개선 사항은 다음 정보와 함께 리포트해주세요:
- 테스트 환경 (OS, Python 버전)
- 실행한 명령어
- 오류 메시지 또는 예상과 다른 결과
- 서버 사양 및 설정

### 추가 리소스
- [Locust 공식 문서](https://docs.locust.io/)
- [Elasticsearch 성능 가이드](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html)
- [PostgreSQL 성능 튜닝](https://wiki.postgresql.org/wiki/Performance_Optimization)
