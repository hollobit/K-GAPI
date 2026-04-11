# 대한민국 공공 Open API 전환 전략 보고서

**작성일**: 2026-04-12
**대상 독자**: 행정안전부, 공공데이터전략위원회, NIA, 공공기관 정보화 담당자
**작성 방법**: 멀티 에이전트 분석 (MCP 전환 전략 / 글로벌 수준 / 사용자 중심 원칙 / Agentic AI 프로토콜 / API 전환 방식 — 5개 에이전트 동시 분석)

---

## Executive Summary

현재 대한민국 공공 Open API는 **173개 포털**, **수천 개 API**가 파편화되어 운영되고 있으며, AI 적합성 점수는 **2.0/10** (해외 평균 7.4)에 불과하다. JSON 기본 응답 10%, HTTP 상태코드 표준 준수 0%, OpenAPI 스펙 제공 0.6%인 상황에서, 3년 내에 글로벌 수준으로 전환하기 위한 종합 전략을 수립한다.

### 핵심 전략 3가지

```
1. MCP 기반 어댑터 계층 도입
   → 기존 API는 유지, MCP 서버로 추상화

2. 프로토콜 스택: MCP + A2A + LangGraph
   → 에이전트 도구(MCP) + 상호운용(A2A) + 상태관리(LangGraph)

3. 하이브리드 → 준중앙형 거버넌스
   → 단기(자발적 참여) → 중기(인센티브) → 장기(법적 의무)
```

### 3개년 목표

| 지표 | 현재 (2026) | 2027 | 2028 |
|------|-------------|------|------|
| AI 적합성 등급 A~B | 1.2% | 40% | 70% |
| 사용자 중심성 등급 A~B | 0% | 30% | 50% |
| MCP 서버 수 | ~10개 | 30개 | 50개+ |
| API 기술 품질 점수 | 2.0/10 | 6.0/10 | 8.0/10 |

---

## Part 1: 현재 문제 진단

### 1.1 구조적 문제 5가지

조사 결과, 한국 공공 API는 **5가지 핵심 문제**가 IDENTIFIED:

| # | 문제 | 현황 | 영향 |
|---|------|------|------|
| 1 | **통합 API 디자인 표준 부재** | 6개국 유일. 173개 포털 각자 독자 운영 | AI 에이전트가 표준화되지 않은 API 자동 처리 불가 |
| 2 | **HTTP 상태코드 오용** | 인증 실패, 파라미터 오류, 호출 초과 모두 HTTP 200 반환 | RFC 7231 위반. AI 에이전트가 성공/실패 구분 불가 |
| 3 | **XML 우선 응답** | ~59%가 XML 기본. JSON은 `?Type=json` 선택 | LLM 토큰 비용 2.5배 증가, JSON 내장 파싱(fetch/axios) 활용 불가 |
| 4 | **API Key URL 노출** | `?serviceKey=xxxx` URL 파라미터가 기본. 8가지+ 인증 파라미터명 난립 | URL 로그/히스토리/Referrer로 키 노출 위험 |
| 5 | **OpenAPI 스펙 미제공** | HWP/PDF/HTML 문서만 제공, 기계 판독 불가 | MCP 서버 자동 생성, LLM Function Calling 자동 연동 불가 |

**출처**: 173개 포털 전수 조사 (Plans.md), policy_recommendations.md, global_level_strategy.md

### 1.2 API 제공 패턴 3가지

data.go.kr의 실제 운영 형태는 3가지 패턴이 혼재:

| 패턴 | 비율 | 설명 | 전환 난이도 |
|------|------|------|----------|
| **Pattern A** | 18% (31개) | data.go.kr 완전 gateway — serviceKey만으로 완료 | 하 |
| **Pattern B** | 23% | 인덱스만, 원본으로 이동 — 별도 인증 필요 | 중 |
| **Pattern C** | 38% | 양쪽 제공, 범위/버전 상이 | 상 |
| **미등록** | 20% | 독립 API — data.go.kr 연동 없음 | 다양 |

**통념-vs-현실**: "data.go.kr 1개 ID로 70% 커버"는 Pattern A에만 해당. Pattern B/C 포함 시 **실제 커버리지는 40% 이하**.

### 1.3 레거시 시스템 문제

| 요소 | 현재 상태 | 문제 |
|------|----------|------|
| XML/SOAP 기반 API | 다수 존재 | JSON/REST 시대 부적합 |
| 공동인증서 기반 인증 | 국세청, 일부 금융 | AI 자동화 불가 |
| API Key 2년 만료 | 전 기관 | 자동 갱신 없음, 놓치면 서비스 중단 |
| CORS 미지원 | 대다수 | 브라우저 직접 호출 불가 |

---

## Part 2: MCP 전환 전략

### 2.1 3단계 접근법

```
Phase 1: 개별 MCP 서버 구축 (단기 6개월)
  → 각 기관별 API → MCP Tool로 변환
  → 예: 기상청 MCP, 통계청 MCP, 국토부 MCP

Phase 2: MCP Federation (중기 12개월)
  → Tool 호출 결과를 다른 MCP 서버에 연결
  → 예: "부동산 실거래가" → "해당 지역 후보 정당" → "해당 정당 대표 의안"

Phase 3: 통합 Public API MCP 게이트웨이 (장기 24개월~)
  → 단일 Entry Point로 60+ 포털 통합 검색
  → 예: "최근 통과된 근로법 관련 의안과 해석례" 한 번에 조회
```

### 2.2 전환 우선순위 매트릭스

| 우선순위 | 대상 | 패턴 | 이유 |
|---------|------|------|------|
| **1순위** | 부동산 실거래가 (국토부) | Pattern A | 수요 최상위, serviceKey만으로 완료 |
| **2순위** | 기상청 중기예보/특보 | Pattern C | apihub.kma.go.kr에 전체 API, 생활 밀착 |
| **3순위** | 에어코리아 대기정보 | Pattern A | serviceKey만, 구현 쉬움 |
| **4순위** | DART 전자공시 | Pattern B | 재무제표 분석 수요 높음 |
| **5순위** | ECOS 경제통계 (한국은행) | Pattern B | JSON 지원, 경제지표 수요 높음 |

**우선순위 기준**: Pattern A (기술 쉬움) × B등급 이상 (사용자 편의성) × 高 수요 (실시간 호출량)

### 2.3 Pattern별 전환 전략

**Pattern A (18%) — 즉시 전환**
```
기상청/국토부/환경부 API
  → serviceKey 환경변수 설정
  → MCP Tool 정의 (2~5개/API)
  → npm/npm install 배포
  → Claude Desktop 연동
```

**Pattern B (23%) — Federated Authentication Proxy**
```
원본site별 별도 인증 → MCP 서버가 인증 대행
  → 사용자는 MCP Tool만 호출
  → 개별site 인증은 MCP 서버가 처리 (서버 사이드)
  → 예: DART (crtfc_key), KOSIS (apiKey)
```

**Pattern C (38%) — 원본 우선 + Meta 역할**
```
원본site 우선 사용 (전체 기능)
data.go.kr는 "차이점"만 표시
  → Tool: get_real_estate_deals (원본site 키 사용)
  → Tool: get_real_estate_deals_meta (data.go.kr 키 사용, 차이점만)
```

### 2.4 레거시 시스템 처리 원칙

```
1. XML → JSON 변환: MCP 어댑터가 담당 (기관은 변경 불필요)
2. 공동인증서: 전자서명 필수인 경우만 불가, 나머지는 MCP 키 대행
3. API Key 만료: MCP 서버가 만료 30일 전 알림 + 자동 갱신 로직 추기
```

### 2.5 3개년 마이그레이션 Timeline

```
2026 (Phase 1: Pilot)
───────────────────────────────
Q2: Pattern A 5개 MCP 전환 (부동산, 에어코리아, 기상청, 국토부 일부)
Q3: Pattern B 5개 MCP 전환 (DART, ECOS, 서울, KOSIS, 관광공사)
Q4: Pattern C 3개试探 (기상청 등 양쪽 제공, 차이점 분석)

2027 (Phase 2: Federation)
───────────────────────────────
Q1: Pattern B 나머지 10개 + Pattern C 5개
Q2: MCP Federation 프로토타입 ("의안→정책→법령" 연동)
Q3: 상위 30개 포털 MCP 전환 완료
Q4: 통합 Gateway 베타 (Entry Point 1개)

2028 (Phase 3: Scale)
───────────────────────────────
Q1: 나머지 30개 포털 전환
Q2: Gateway 공식 운영
Q3: OAuth 2.0 전환 (기관 협조 필요)
Q4: 목표 달성 — 60+ 포털 MCP 통합
```

### 2.6 연간 목표 (구체적 숫자)

| 연도 | 전환 포털 수 | MCP 서버 신규 | 커버 API 수 |
|------|------------|-------------|-----------|
| 2026 | **13개** (Pattern A 5 + B 5 + C 3) | 8개 | ~200개 |
| 2027 | **20개** (Pattern B 10 + C 5 + 미등록 5) | 12개 | ~500개 |
| 2028 | **30개** (미등록 + Pattern C 나머지) | 10개 | ~1,000개+ |

**총 3개년: 63개 포털 전환** (현재 60+ 포털 전수 커버 목표)

---

## Part 3: 사용자 중심 원칙 및 평가 체계

### 3.1 5대 필수 원칙

공공 API를 **공급자 중심**이 아닌 **사용자 중심**으로 제공하기 위한 필수 원칙:

| # | 원칙 | 핵심 실천 |
|---|------|----------|
| 1 | **설계자는 사용자를 위해서 설계한다** | 가입→승인→첫 호출 24시간 이내, 이메일/소셜 로그인 |
| 2 | **기계는 사람보다 중요한 클라이언트다** | JSON 기본 + HTTP 상태코드 + OpenAPI 3.0, "예상 소비자=AI 에이전트"로 설계 |
| 3 | **진입장벽은 제로에 가깝어야 한다** | Key-Free 테스트, 샌드박스/테스트콘솔 내장, CORS 완전 지원 |
| 4 | **명확한 계약 — 예측 가능한 동작** | X-RateLimit 헤더, API 변경 6개월 사전 고지, 자동 갱신 OAuth 2.0 |
| 5 | **공급자 편의가 아닌 사용자 경험으로 승부** | A등급 50%+, 다국어 SDK, 1개 ID로 전체 API 접근 |

### 3.2 평가 매트릭스 (28점 만점)

#### 사용자 중심성 7항목 (14점 만점)

| # | 항목 | 0점 (공급자 중심) | 1점 (보통) | 2점 (사용자 중심) |
|---|------|-----------------|-----------|-----------------|
| U1 | 가입 용이성 | 공동인증서/기관심사 | 휴대폰 인증 | 이메일/소셜 (즉시) |
| U2 | 승인 속도 | 3일+ 수동 | 1일 이내 혼재 | 즉시 자동 |
| U3 | 데이터 포맷 | XML만 | XML+JSON 선택 | JSON 기본 |
| U4 | API 문서 품질 | 없음/부실 | 기본 설명 | Swagger/테스트콘솔 |
| U5 | 호출 한도 | 100회/일 이하 | 1,000회/일 | 10,000회+/일 |
| U6 | 에러 처리 | 불명확 | 코드 있으나 설명 부족 | 코드+메시지+해결방안 |
| U7 | SDK/샘플코드 | 없음 | 1개 언어 | 다국어 SDK |

#### AI 적합성 7항목 (14점 만점)

| # | 항목 | 0점 (부적합) | 1점 (보통) | 2점 (AI-Ready) |
|---|------|------------|-----------|---------------|
| A1 | 기계 판독 스키마 | 없음/HWP/PDF | HTML 문서 | OpenAPI 3.0 |
| A2 | HTTP 의미론 | 항상 200 반환 | 일부 상태코드 사용 | 401/404/429/500 표준 |
| A3 | JSON 네이티브 | XML only | JSON 선택 가능 | JSON 기본 |
| A4 | 에러 자기설명성 | 코드만/오타 | 코드+메시지 | RFC 7807 Problem Details |
| A5 | 인증 방식 | 공동인증서/GUI | API Key URL노출 | OAuth 2.0+Header+자동갱신 |
| A6 | Rate Limit 투명성 | 정보 없음 | 문서에만 기재 | X-RateLimit 헤더 표준 |
| A7 | CORS/HTTPS | 미지원 | 일부 지원 | 완전 지원 |

**등급 산정**:
- **A 등급**: 25~28점 (사용자 중심 + AI 친화)
- **B 등급**: 18~24점
- **C 등급**: 12~17점
- **D 등급**: 6~11점
- **E 등급**: 0~5점

### 3.3 3개년 목표 (2026~2028)

| 지표 | 현재 (2026 기준) | 2026 | 2027 | 2028 |
|------|-----------------|------|------|------|
| AI 적합성 A~B | 1.2% (2개) | **10%** | **40%** | **70%** |
| 사용자 중심성 A~B | 0% | **5%** | **30%** | **50%** |
| JSON 기본 응답 | ~10% | **50%** | **80%** | **100%** |
| OpenAPI 스펙 제공 | ~0.6% | **10%** | **40%** | **80%** |
| API 기술 품질 점수 | 2.0/10 | **4.0** | **6.0** | **8.0** |

### 3.4 품질 관리 체계

**3 Layer 구조**:

| Layer | 내용 | 실행 주체 |
|-------|------|----------|
| **제도적** | API 디자인 표준 고시, OpenAPI 의무화, 연간 품질 평가 공개 | 행정안전부 |
| **기술적** | 자동 검증 시스템, 실시간 모니터링, 변경 이력 관리 | NIA/공공데이터포털 |
| **커뮤니티** | GitHub 공개 가이드, 개발자 피드백 채널, 만족도 조사 | 개발자 커뮤니티 |

**Kaizen 접근**: E/D→C (6개월, 예산最小) → C→B (1~2년) → B→A (2~3년)

**하위 호환성 절대 원칙**: 기존 개발자 코드 동작 안 되는 경우는 만들지 않음.

---

## Part 4: 글로벌 수준 도달 전략

### 4.1 단계별 추진 전략

#### 단기 (6개월 — 서버 설정 변경만, 예산 없음)

| 조치 | 내용 | 실행 기관 |
|------|------|----------|
| JSON 기본 전환 | `Type=json` URL 기본값 변경 | data.go.kr 기술팀 |
| HTTP 상태코드 정상화 | 401/404/429 분기 반환 | 각 기관 |
| CORS 헤더 추가 | `Access-Control-Allow-Origin: *` | 각 기관 |
| X-RateLimit 헤더 | 잔여 호출량 사전 고지 | data.go.kr Gateway |

**예상 효과**: AI 적합성 등급 D → C (2.0 → 4.5/10)

#### 중기 (12~24개월 — 정책 전환, 표준 제정)

| 조치 | 내용 | 실행 주체 |
|------|------|----------|
| 한국 정부 API 디자인 표준 고시 | 영국 GDS 참조 (JSON 필수, OpenAPI 3.0, CORS, Rate Limit) | 행정안전부 |
| OpenAPI 3.0 신규 API 의무 제출 | 신규 API 등록 시 필수 제출 | NIA |
| API Key HTTP 헤더 전환 | URL 파라미터 → `Authorization: Bearer` | 각 기관 |
| 에러 응답 RFC 7807 표준화 | `type`, `title`, `status`, `detail` 구조 | 각 기관 |
| 샌드박스 환경 제공 | Swagger UI, DEMO_KEY 내장 | data.go.kr |

**예상 효과**: AI 적합성 등급 C → B (4.5 → 6.5/10)

#### 장기 (36개월+ — 법령 개정, 문화 변화)

| 조치 | 내용 | 실행 주체 |
|------|------|----------|
| 공공데이터법 개정 | 표준 준수 법적 의무 | 행안부/국회 |
| GovAuth (통합 OAuth 2.0) 인프라 구축 | 60+ 포털 SSO | 행정안전부 |
| data.go.kr Gateway 확대 | 18% → 80%+ 커버 | NIA |
| API 우선(API-First) 행정 원칙 선언 | 전 공공기관 적용 | 행정안전부 |

**예상 효과**: AI 적합성 등급 B → A (6.5 → 8.0+/10)

### 4.2 권장 거버넌스 모델: 하이브리드 → 준중앙형

**선택 이유**:
- **중앙 통제형(영국 GDS) 즉시 도입**: 법적 근거 부재, 기관 저항, 레거시 부담으로 비현실적
- **하이브리드(미국 GSA/호주 DTA)**: 인센티브(트래픽 분석, DDoS 방어, 단일 키)로 자발적 참여 유도, 중앙 병목 없음

**3대 축**:

| 축 | 내용 | 실행 |
|---|------|------|
| **표준 제시** | 행정안전부 API 디자인 표준 고시 (권고, 미준수 Penalty 없음) | 2026~2027 |
| **공통 인프라** | data.go.kr Gateway 고도화 (API Umbrella 오픈소스 기반) | 2026~2028 |
| **품질 인증** | API 품질 등급제 (A/B/C/D) 공개, A등급 우선 노출 | 2027~2028 |

**단계적 전환**:
```
현재 분산형(B) → 1-2년 후 하이브리드 기반(C-) → 3-4년 후 하이브리드 강화(C+) → 5년+ 준중앙형(A-)
```

---

## Part 5: Agentic AI 프로토콜 스택

### 5.1 프로토콜 현황 분석

| 프로토콜 | 개발자 | 현황 | 공공 서비스 적합성 |
|---------|-------|------|------------------|
| **MCP** | Anthropic → Linux Foundation | 50+ 공식 서버, 97M 월간 SDK 다운로드 | ★★★★★ 필수 — 공공 API를 에이전트 도구로 변환 |
| **A2A** | Google Cloud | v1.0.0 (2026.3), 100+ 파트너, Linux Foundation AAIF | ★★★★★ 최고 — 기관간 에이전트 상호운용 |
| **ANP** | OSS (GaoWei Chang) | Apache 2.0, W3C DID 기반 분산 | ★★★ 중 — 중앙집중적 공공 서비스와 충돌 가능 |
| **ACP** | IBM/BeeAI | UI 자동화 용도 | ★★★ 중 — 공공 서비스에 부적합 |
| **LangGraph** | LangChain | 그래프 기반 상태 관리, 체크포인팅, 인간 인터럽트 | ★★★★★ 최고 — 복잡한 워크플로우 처리 |
| **W3C WebMCP** | Google+Microsoft | Chrome 146 preview (2026.2) | ★★★★ 우수 — 브라우저 연동 |

### 5.2 권장 프로토콜 스택: MCP + A2A + LangGraph

```
핵심 논리:
MCP = 에이전트의 "손" (도구 접근)
A2A = 에이전트의 "말" (상호운용)
LangGraph = 에이전트의 "뇌" (상태 관리)
W3C WebMCP = 브라우저의 "문" (웹 인터랙션)
```

**ANP 미채택 이유**: W3C DID 기반 분산 신원 vs 한국 공공 서비스의 중앙집중적 성격. 장기적으로 재검토 가능.

### 5.3 단계별 프로토콜 도입

#### Phase 1 (2026): MCP + LangGraph
```
[MCP Servers] → [LangGraph Federation Layer] → [Public API Gateway]
 - 개별 기관 MCP 서버 (기상청, 법제처, 통계청 등)
 - LangGraph로 상태 관리 + 워크플로우 오케스트레이션
 - 체크포인팅으로 실패 복구
```

#### Phase 2 (2027): MCP + A2A + LangGraph
```
[MCP Servers] → [A2A Protocol] → [LangGraph Orchestrator] → [Gateway]
 - 기관간 에이전트 위임
 - A2A로 에이전트 발견 및 통신
 - LangGraph로 복잡한 비즈니스 프로세스 관리
```

#### Phase 3 (2028+): Full Stack
```
[Human] ↔ [A2A Agent Interface] ↔ [A2A Coordination] ↔ [MCP Tool Access]
                              ↕
                    [LangGraph State + Workflow]
                              ↕
                    [W3C WebMCP Browser Integration]
```

### 5.4 3개년 Agentic AI 목표

| 목표 | 2026 (단기) | 2027 (중기) | 2028 (장기) |
|------|-------------|-------------|-------------|
| **MCP 서버 수** | 10개 (기관별) | 30개 | 50개+ |
| **A2A 도입** | 연구/평가 | 파일럿 (3개 기관) | 프로덕션 |
| **LangGraph Federation** | 개념 검증 | 10개 MCP 연동 | 30개 MCP 연동 |
| **W3C WebMCP** | 모니터링 | 파일럿 (브라우저) | 채택 |

### 5.5 Agentic AI 공공 서비스 시나리오

| 시나리오 | 연동 MCP | 프로토콜 |
|---------|---------|---------|
| **"부동산→정책 연관 검색"** | 부동산원 +assembly-api-mcp + korean-law-mcp | MCP + LangGraph |
| **"기업 설립 종합 민원"** | 법원 + 세무서 + 부동산원 + 근로복지공단 | MCP + A2A + LangGraph |
| **"정책 추천 (의안+법령+예산)"** | assembly-api-mcp + korean-law-mcp + NABO | MCP + LangGraph |

---

## Part 6: 정부 및 공공기관의 구체적 노력

### 6.1 단계별 책임 분담

| 기간 | 행정안전부 | NIA | 각 기관 (금융감독원, 기상청 등) |
|------|-----------|-----|----------------------------|
| **단기 (2026)** | API 디자인 표준 고시 (권고) | Gateway 고도화 기술 지원 | CORS/RateLimit 헤더 추가, JSON 기본 전환 |
| **중기 (2027)** | 품질 등급제 도입 | OpenAPI 스펙 검증 자동화 | 신규 API OpenAPI 3.0 제출, HTTP 상태코드 정상화 |
| **장기 (2028+)** | 공공데이터법 개정 추진 | 통합 Gateway 운영 | OAuth 2.0 전환, API 우선 행정 원칙 적용 |

### 6.2 연간 구체적 과제

#### 2026 — 기반 정비년
- [ ] JSON 기본 전환 (신규 API 100%)
- [ ] HTTP 상태코드 정상화 (50% 적용)
- [ ] CORS + X-RateLimit 헤더 (data.go.kr 100%)
- [ ] 상위 50개 API OpenAPI 스펙 작성
- [ ] MCP-Ready 인증 도입 (조건: JSON + OpenAPI + CORS + 즉시 키 발급)
- [ ] **MCP 서버 10개 신규 구축** (부동산, 기상청, 에어코리아, DART, ECOS 등)

#### 2027 — 표준화·통합년
- [ ] 행정안전부 "한국 정부 API 디자인 표준" 고시
- [ ] OpenAPI 3.0 신규 API 의무 제출
- [ ] API Key HTTP 헤더 전환 방식
- [ ] 에러 응답 RFC 7807 표준화
- [ ] **MCP 서버 20개 추가 구축**
- [ ] **LangGraph Federation 프로토타입** (3개 MCP 연동)
- [ ] **A2A 파일럿** (3개 기관)

#### 2028 — 고도화·확산년
- [ ] 공공데이터법 개정안 제출 (표준 준수 법적 의무)
- [ ] GovAuth (통합 OAuth 2.0) 인프라 구축
- [ ] Gateway 확대 (200개+ API)
- [ ] **MCP 서버 30개 추가 구축** (총 60개+)
- [ ] **A2A + LangGraph 프로덕션** (10개+ 복합 시나리오)
- [ ] **W3C WebMCP Browser Integration 방식**

### 6.3 인센티브 및 지원

| 유형 | 내용 | 대상 |
|------|------|------|
| **재정 지원** | MCP 서버 개발 보조금 500만원/건 | 개별 개발자/팀 |
| **인증 우대** | MCP-Ready 인증 취득 API 우선 노출 | 공공 기관 |
| **표준 채택 인센티브** | OpenAPI 스펙 제공 시 data.go.kr 우선 노출 | 각 기관 |
| **우수 오픈소스 포상** | GitHub Stars 500+ 달성 시 포상 | 개발자/팀 |
| **기술 지원** | NIA 표준 가이드 배포 + 컨설팅 | 공공 기관 |

### 6.4 예상 비용

| 항목 | 단일 MCP 서버 개발 비용 |
|------|----------------------|
| API 스펙 분석 | 80~160만원 (8~16시간) |
| MCP 서버 구현 | 160~320만원 (16~32시간) |
| 테스트/디버깅 | 80~160만원 (8~16시간) |
| 문서화 | 40~80만원 (4~8시간) |
| **합계** | **360~720만원/서버** |

**총 예산 (3개년)**:
- MCP 서버 60개 × 평균 500만원 = **3억원**
- Gateway 고도화 = **5억원** (NIA 예산)
- 표준 제정/교육 = **2억원**
- **총 합계: ~10억원** (3개년)

---

## Part 7:Plans.md — 실행 계획

```markdown
# 공공 API 전환 전략 Plans.md

작성일: 2026-04-12

---

## Phase 1: 기반 정비 (2026)

| Task | 내용 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 1.1 | JSON 기본 전환 (신규 API 100%) | data.go.kr 기술팀 서버 설정 변경 완료 | - | cc:TODO |
| 1.2 | HTTP 상태코드 정상화 (50% 적용) | 401/404/429 반환 확인 | 1.1 | cc:TODO |
| 1.3 | CORS + X-RateLimit 헤더 추가 (data.go.kr 100%) | 헤더 존재 확인 (curl 테스트) | - | cc:TODO |
| 1.4 | 한국 정부 API 디자인 표준 고시 (행정안전부) | 관고 발동 | - | cc:TODO |
| 1.5 | OpenAPI 3.0 신규 API 의무 제출 규정 제정 | 고시 완료 | - | cc:TODO |
| 1.6 | 부동산 실거래가 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 1.7 | 기상청 중기예보 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 1.8 | 에어코리아 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 1.9 | DART 전자공시 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 1.10 | ECOS 경제통계 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 1.11 | LangGraph Federation 프로토타입 구축 | assembly-api-mcp + korean-law-mcp 연동 확인 | 1.6, 1.7, 1.8, 1.9, 1.10 | cc:TODO |

---

## Phase 2: 표준화·통합 (2027)

| Task | 내용 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 2.1 | API Key HTTP 헤더 전환试点 (5개 기관) | Authorization 헤더 정상 동작 확인 | Phase 1 | cc:TODO |
| 2.2 | 에러 응답 RFC 7807 표준화 (상위 30개 API) | RFC 7807 구조 에러 반환 확인 | Phase 1 | cc:TODO |
| 2.3 | 샌드박스 환경 제공 (data.go.kr) | Swagger UI 연동 확인 | Phase 1 | cc:TODO |
| 2.4 | 서울 열린데이터 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 2.5 | KOSIS 통계청 MCP 서버 구축 | npm 패키지 공개 + CLI 테스트 통과 | - | cc:TODO |
| 2.6 | 한국 정부 API 디자인 표준 고시 (관식) | 고시 완료 | Phase 1 | cc:TODO |
| 2.7 | MCP-Ready 인증 도입 | 조건 충족 API 등록 완료 | Phase 1 | cc:TODO |
| 2.8 | A2A 파일럿 (3개 기관: 기상청, 법제처, 통계청) | A2A 프로토콜 에이전트 간 통신 확인 | 1.6, 1.7, 1.8, 2.4, 2.5 | cc:TODO |
| 2.9 | 통합 Gateway MVP (Entry Point 1개) | 10개 MCP 서버 연동 확인 | 1.11, 2.4, 2.5 | cc:TODO |

---

## Phase 3: 고도화·확산 (2028)

| Task | 내용 | DoD | Depends | Status |
|------|------|-----|---------|--------|
| 3.1 | 공공데이터법 개정안 제출 (표준 준수 법적 의무) | 국회 제출 완료 | Phase 2 | cc:TODO |
| 3.2 | GovAuth (통합 OAuth 2.0) 인프라 구축 | 통합 SSO 개발 완료 | Phase 2 | cc:TODO |
| 3.3 | Gateway 확대 (200개+ API) | 200개 API 연동 확인 | 2.9 | cc:TODO |
| 3.4 | A2A + LangGraph 프로덕션 (10개+ 복합 시나리오) | 복합 시나리오 10개 동작 확인 | 2.8, 2.9 | cc:TODO |
| 3.5 | W3C WebMCP Browser Integration试点 | Chrome 연동 확인 | Phase 2 | cc:TODO |
| 3.6 | OAuth 2.0 전환 완료 (상위 30개 API) | OAuth 2.0 인증 정상 동작 | 3.2 | cc:TODO |
| 3.7 | 3개년 성과 보고서 작성 | 政府 보고서 발행 | 3.1, 3.2, 3.3, 3.4, 3.5, 3.6 | cc:TODO |

---

## 핵심 지표 (KPI)

| KPI | 2026 | 2027 | 2028 |
|-----|------|------|------|
| MCP 서버 수 | 10개 | 30개 | 60개+ |
| API 기술 품질 점수 | 4.0/10 | 6.0/10 | 8.0/10 |
| AI 적합성 A~B 비율 | 10% | 40% | 70% |
| 사용자 중심성 A~B 비율 | 5% | 30% | 50% |
```

---

## 참고 자료

### 핵심 분석 문서

- [Plans.md](/Users/jonghongjeon/git/gapi/Plans.md) — 173개 포털 전수 평가
- [mcp-strategy.md](/Users/jonghongjeon/git/gapi/output/mcp-strategy.md) — MCP 전환 전략
- [policy_recommendations.md](/Users/jonghongjeon/git/gapi/output/policy_recommendations.md) — 정책 제언서
- [korea_improvement_roadmap.md](/Users/jonghongjeon/git/gapi/output/korea_improvement_roadmap.md) — 해외 사례 기반 개선 로드맵
- [global_mcp_ecosystem.md](/Users/jonghongjeon/git/gapi/output/global_mcp_ecosystem.md) — 해외 정부 MCP 생태계
- [api_design_guidelines_comparison.md](/Users/jonghongjeon/git/gapi/output/api_design_guidelines_comparison.md) — 5개국 API 가이드라인 비교
- [governance_models.md](/Users/jonghongjeon/git/gapi/output/governance_models.md) — 거버넌스 모델 비교

### 에이전트 분석 산출물

- [task3_user_centric_principles.md](/Users/jonghongjeon/git/gapi/output/task3_user_centric_principles.md) — 사용자 중심 원칙 및 평가 기준
- [global_level_strategy.md](/Users/jonghongjeon/git/gapi/output/global_level_strategy.md) — 글로벌 수준 도달 전략

### MCP 서버 참고 구현

- [hollobit/assembly-api-mcp](https://github.com/hollobit/assembly-api-mcp) — 287개 API, 6개 Lite/11개 Full 도구
- [chrisryugj/korean-law-mcp](https://github.com/chrisryugj/korean-law-mcp) — 1,344 stars, 58개 Tool

### Agentic AI 프로토콜

- [A2A Protocol](https://a2a-protocol.org/) — Google 주축 에이전트 상호운용 표준
- [MCP 2026 Roadmap](https://agentsource.co/articles/mcp-2026-roadmap-what-is-changing) — MCP 에이전트-투-에이전트 위임 예정
- [LangGraph Multi-Agent Tutorial](https://langchain-tutorials.github.io/langgraph-multi-agent-systems-2026/) — 상태 관리 + 워크플로우
- [W3C WebMCP](https://webmcp.link/) — Chrome 146 preview, 브라우저 에이전트 연동

---

*이 문서는 5개 멀티 에이전트 분석 결과를 취합하여 작성됨. 각 에이전트는 다음 영역을 담당: MCP 전환 전략 / 글로벌 수준 / 사용자 중심 원칙 / Agentic AI 프로토콜 / API 전환 방식*