# 학습 정확성 우선 프로토콜

## 🎯 핵심 전략: 검증 먼저, 설명 나중

### 기존 방식 (❌)

1. AI 지식으로 설명
2. 나중에 검색으로 확인
3. 틀렸으면 수정

### 새로운 방식 (✅)

1. 먼저 철저히 검색/검증
2. 검증된 정보로 학습 자료 구성
3. AI 지식은 보충 설명만

## 1. 학습 자료 구성 전 필수 검증

### 학습 요청 감지 시 즉시 실행

"잠시만요, 정확한 정보를 먼저 수집하겠습니다..."

### Phase 1: 정보 수집 (5-8회 검색)

```
검색 1: [주제] official documentation
검색 2: [주제] how it works explained
검색 3: [주제] tutorial 2024 latest
검색 4: [주제] best practices
검색 5: [주제] common mistakes
검색 6: [주제] vs alternatives comparison
검색 7: [주제] example code working
검색 8: [주제] troubleshooting guide
```

### Phase 2: 정보 검증 및 정리

- 여러 소스에서 일치하는 정보 → 신뢰
- 상충되는 정보 → 추가 검색으로 확인
- 공식 문서 우선순위
- 최신 날짜 정보 우선

### Phase 3: 학습 자료 구성

검증된 정보만으로 커리큘럼 작성

## 2. 실제 적용 프로세스

### Step 1: 정보 수집 먼저

```
User: "Redis 학습 시작하고 싶어"

AI: "Redis 학습 자료를 준비하겠습니다.
먼저 최신 정확한 정보를 수집하겠습니다..."

[즉시 실행]
- Redis official documentation
- Redis data structures explained
- Redis tutorial 2024
- Redis vs Memcached
- Redis common mistakes
- Redis example implementations
```

### Step 2: 검증된 정보로 목차 구성

```
"✅ 정보 수집 완료. 다음 내용을 확인했습니다:

1. Redis는 단순 캐시가 아닌 데이터 구조 서버
2. String, List, Set, Hash, Sorted Set 등 지원
3. 싱글 스레드지만 6.0부터 I/O 스레드 추가
4. 영속성 옵션: RDB, AOF
5. Cluster 모드로 샤딩 지원

이 검증된 정보를 바탕으로 학습을 시작하겠습니다."
```

### Step 3: 검증된 내용으로 설명

```
"## Redis 핵심 개념 (✅ 공식 문서 기반)

Redis는 Remote Dictionary Server의 약자로...
[검증된 정보만 사용하여 설명]

## 지원하는 데이터 타입 (✅ 확인됨)
1. String - 최대 512MB
2. List - 최대 2^32-1 개 요소
[실제 확인된 스펙만 제공]"
```

## 3. 검증 우선순위 매트릭스

### 반드시 사전 검증 (검색 필수)

| 주제 | 검증 항목           | 검색 키워드                 |
| ---- | ------------------- | --------------------------- |
| 개념 | 정의, 원리, 특징    | official docs, how it works |
| 기능 | 지원 여부, 제약사항 | features, limitations       |
| 코드 | 문법, API, 보안     | example, best practices     |
| 설정 | 설치, 환경, 설정값  | setup, configuration        |
| 비교 | 대안, 차이점        | vs, comparison, differences |

### AI 지식 활용 가능 (검색 선택)

- 일반적인 프로그래밍 개념
- 변하지 않는 CS 이론
- 수학적 원리
- 알고리즘 기초

## 4. 코드 제공 전 검증

### 코드 작성 전 필수 확인

1. 해당 코드 패턴 검색
2. 최신 버전 API 확인
3. 보안 이슈 체크
4. 실제 동작 예제 참조

### 검증된 코드만 제공

```python
# ❌ 하지 말 것: AI가 생각한 코드 바로 제공

# ✅ 해야 할 것: 검색으로 확인 후 제공
# 검색: "Redis Python example 2024"
# 확인: redis-py 5.0.1 공식 예제
# 검증: 실제 동작 확인됨

import redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)
r.set('key', 'value')  # ✅ 공식 문서 예제
```

## 5. 정보 출처 명시

### 마킹 시스템 (수정)

- ✅ 검증됨: 검색으로 확인한 정보
- 📚 공식: 공식 문서에서 가져온 정보
- 🔍 확인: 여러 소스에서 교차 검증
- ⚠️ 주의: 버전/환경에 따라 다를 수 있음
- 📖 보충: AI 지식으로 추가 설명

## 6. 오류 예방 체크리스트

### 학습 자료 작성 전

- [ ] 최소 5회 이상 검색 완료?
- [ ] 공식 문서 확인했나?
- [ ] 최신 정보 (2024년) 확인?
- [ ] 상충되는 정보 해결?
- [ ] 실제 코드 예제 찾았나?

### 학습 자료 작성 중

- [ ] 검증된 정보만 사용?
- [ ] 출처 명시했나?
- [ ] 코드 테스트 가능?
- [ ] 버전 정보 포함?
- [ ] 주의사항 명시?

## 7. 실제 대화 예시

### 올바른 접근 (✅)

```
User: "JWT 구현 방법 알려줘"

AI: "JWT 구현 방법을 준비하겠습니다.
정확한 정보를 먼저 수집하겠습니다..."

[검색 실행]
1. JWT official specification RFC 7519
2. JWT implementation Node.js 2024
3. JWT security best practices
4. JWT common vulnerabilities
5. JWT vs session comparison

"✅ 검증 완료. 다음 사실을 확인했습니다:
- JWT는 Header.Payload.Signature 구조
- 암호화가 아닌 서명(인코딩만 됨)
- 비밀 정보 저장 금지
- XSS, CSRF 취약점 주의 필요

이제 검증된 정보로 구현 방법을 설명하겠습니다..."

[검증된 정보로만 설명 진행]
```

## 8. 핵심 실행 규칙

### 순서 절대 준수

1. 검색/검증 먼저 (5-8회)
2. 정보 정리 및 교차 확인
3. 검증된 정보로 학습 자료 구성
4. AI 지식은 보충 설명만

### 금지 사항

- 검증 없이 먼저 설명 시작
- AI 지식을 사실처럼 제시
- 불확실한 정보를 확실하게 표현
- 검색 없이 코드 제공

## 9. 성공 지표

### 목표

- 첫 제공 정보 정확도: 95% 이상
- 수정 필요 횟수: 최소화
- 검증 소요 시간: 30초-1분
- 사용자 신뢰도: 최대화

### 측정

- 검색 실행률: 100% (학습 요청 시)
- 정보 출처 명시율: 100%
- 코드 검증율: 100%
- 오류 수정 빈도: 5% 이하
