# 공공 API를 위한 MCP 전환 전략

**작성일**: 2026-04-12
**대상 독자**: 공공 API 통합을 기획하는 PM, MCP 서버 개발자, 정책 수립자
**전제 조건**: 한국 공공 API 파편화 문제에 대한 기본 이해

---

## 1. 문제 정의: 왜 공공 API 통합이 필요한가

### 1.1 현재 한국의 공공 API 생태계

조사 결과, 한국 공공 API는 심각한 **파편화·비표준·프래그먼트화** 상태에 있다.

| 문제 유형 | 구체적 사례 | 개발자 영향 |
|-----------|------------|------------|
| **60+ 독립 포털** | data.go.kr, gov.kr, 각 시/도별로 각자 가입 | 계정 관리 부담 |
| **5가지 호출 패턴** | Type A (Query 전체), B (Path 전체), C (Path+Query 혼합), D (확장자 방식), E (SOAP/POST) | 범용 클라이언트 불가 |
| **인증 체계 다양** | serviceKey, crtfc_key, apiKey, confmKey, 공동인증서 | 매번 인증 로직 개발 |
| **XML 기본 응답** | 2.5배 토큰 소모, JSON 미지원 API 다수 | 파싱 부담 |
| **HTTP 200 일관 반환** | 실제 에러를 body 내부에서 확인해야 함 | AI 자동 판단 불가 |
| **CORS 미지원** | 브라우저 직접 호출 불가 | 프록시 필수 |
| **Rate Limit 투명성 부재** | X-RateLimit 헤더 없음 | 남은 호출량 파악 불가 |

### 1.2 파편화가 초래하는 비용

```
┌─────────────────────────────────────────────────────────┐
│  현재: 개발자마다 중복 작업                               │
├─────────────────────────────────────────────────────────┤
│  개발자 A: 기상청 API 래핑 (1주)                         │
│  개발자 B: 기상청 API 래핑 (1주)  ← 중복                 │
│  개발자 C: 기상청 API 래핑 (1주)  ← 중복                 │
│  개발자 D: KOSIS API 래핑 (1주)                         │
│  개발자 E: KOSIS API 래핑 (1주)  ← 중복                 │
├─────────────────────────────────────────────────────────┤
│  사회적 비용: N × M × 1주 (N=개발자, M=포털 수)         │
└─────────────────────────────────────────────────────────┘
```

**개선 후**: MCP 서버를 한번 개발하면 모든 AI 클라이언트가 재사용

```
┌─────────────────────────────────────────────────────────┐
│  개선 후: 중앙 집중형 MCP 추상화                          │
├─────────────────────────────────────────────────────────┤
│  기상청 MCP 서버 (1회 개발) ← 모든 AI 에이전트가 활용    │
│  KOSIS MCP 서버 (1회 개발) ← 모든 AI 에이전트가 활용    │
│  통계청 MCP 서버 (1회 개발) ← 모든 AI 에이전트가 활용    │
├─────────────────────────────────────────────────────────┤
│  사회적 비용: M × 1주 (M=포털 수) — 중복 제거            │
└─────────────────────────────────────────────────────────┘
```

### 1.3 공공 API의 "사용자 중심성"과 "AI 적합성" 현황

173개 포털 조사 결과, 현재 상태는 매우 저조하다:

| 평가 항목 | 공공 API 평균 |필요 수준 |
|-----------|-------------|------|
| 사용자 중심성 | C~D 등급 | A 등급 |
| AI 적합성 | D~E 등급 | A 등급 |
| OpenAPI 스펙 제공 | 0.6% | 100% |
| CORS 지원 | 0% | 100% |
| 샌드박스/테스트 콘솔 | 0.6% | 100% |

`★ Insight ─────────────────────────────────────`
**구조적 문제**: 공공 API는 "공급자 중심" 설계로 구축됨. AI 에이전트가 자동 소비하기 위해 필요한 기술적 조건(HTTP 의미론, OpenAPI 스펙, 기계 판독 가능성)이 거의 충족되지 않음.
`─────────────────────────────────────────────────`

---

## 2. 사례 분석: korean-law-mcp와 assembly-api-mcp

### 2.1 korean-law-mcp (법제처 API → MCP)

**프로젝트 정보**:

| 항목 | 내용 |
|------|------|
| 저장소 | [chrisryugj/korean-law-mcp](https://github.com/chrisryugj/korean-law-mcp) |
| Stars | 1,344 |
| 언어 | TypeScript |
| 라이선스 | MIT |
| 도구 수 | 58개 |
| 생성일 | 2025년 12월, 최종 업데이트 2026년 4월 |

**어떻게 작동하는가**:

```
[AI 에이전트] "근로기준법 제74조 해석례 찾아줘"
                    ↓
           korean-law-mcp (58개 Tool)
                    ↓
           법제처 국가법령정보 Open API
                    ↓
           [AI 에이전트] "해석례 3건 표시: ①~, ②~, ③~"
```

**핵심 성공 요소**:

1. **약칭 자동 변환**: "화관법" → "화학물질관리법" 자동 인식
2. **3단 위임 구조 시각화**: 법률 → 시행령 → 시행규칙 계층 한눈에
3. **HWP → Markdown 추출**: 별표/별지서식 파일도 소비 가능
4. **fly.dev 공개 엔드포인트**: 설치 없이 URL만 등록해서 사용 가능
5. **실무자가 개발**: 광진구청 공무원이라는 점이 실제 수요 반영

**기술적 패턴**:

```python
# 핵심: Tool description에 한국어 맥락을 풍부하게 기술
TOOL = Tool(
    name="search_precedents",
    description=(
        "대법원 판례 검색. 사건명, 사건번호, 법원, 날짜 범위로 검색 가능. "
        "판례 요약, 판결 일자, 관련 조문 정보를 반환한다. "
        "특정 법률 조문의 적용 사례를 찾을 때 유용하다."
    ),
    inputSchema={...}
)
```

### 2.2 assembly-api-mcp (열린국회정보 + 국민참여입법센터 + NABO → MCP)

**프로젝트 정보**:

| 항목 | 내용 |
|------|------|
| 저장소 | [hollobit/assembly-api-mcp](https://github.com/hollobit/assembly-api-mcp) |
| 언어 | TypeScript (Node.js) |
| 라이선스 | Apache-2.0 |
| 버전 | v0.7.0 (2026-04-12) |
| 도구 수 | **6개 Lite / 11개 Full** (도메인 엔티티 통합) |
| API 커버 | **287개** (276개 국회 + 8 국민참여입법센터 + 3개 NABO) |

** arsitektur**:

```
┌─────────────────────────────────────────────────────────┐
│                 assembly-api-mcp                         │
├─────────────────────────────────────────────────────────┤
│  도메인 엔티티 기반 도구 설계                             │
│  ─────────────────────────────────────────────────────  │
│  assembly_bill      — 의안 검색+추적+통계 (하나의 도구)   │
│  assembly_member    —국회의원 검색+분석 (하나의 도구)         │
│  assembly_session  — 일정+회의록+표결 (하나의 도구)       │
│  assembly_org      — 위원회+청원+입법예고 (하나의 도구)   │
│  bill_detail       — 의안 상세 (Full 전용, 4개 API 묶음) │
│  research_data     — 연구자료+예산분석 (Full 전용)       │
├─────────────────────────────────────────────────────────┤
│  Dual Transport 지원                                      │
│  ─────────────────────────────────────────────────────  │
│  stdio (Claude Desktop)  +  HTTP/SSE (원격 서버)          │
│  REST API + OpenAPI 스펙 (/openapi.json) → ChatGPT GPTs  │
├─────────────────────────────────────────────────────────┤
│  성능 최적화                                              │
│  ─────────────────────────────────────────────────────  │
│  SWR 캐시 | DNS 프리패칭 | gzip 압축 | 예측 프리패치      │
│  MCP Progress/Logging 지원                              │
└─────────────────────────────────────────────────────────┘
```

**Lite (6개) / Full (11개) 도구**:

| Lite | Full 추가 |
|------|-----------|
| `assembly_bill` | `bill_detail` (심사경과+이력+제안자) |
| `assembly_member` | `research_data` (연구자료+예산분석) |
| `assembly_session` | `get_nabo` (NABO 보고서/정기간행물/채용) |
| `assembly_org` | |
| `discover_apis` | |
| `query_assembly` | |

**핵심 성공 요소**:

1. **도메인 엔티티 통합 설계**: "의안 하나를 찾을 때" 검색+상세+이력을 각각 호출하지 않고, 하나의 도구 호출로 완결
2. **Lite/Full 이중 프로필**: 토큰 제약이 있는 AI 에이전트용 Lite(6개) vs 심층 분석용 Full(11개) — LLM 도구 선택 정확도 최적 구간(6~8개) 유지
3. **입법 라이프사이클 완전 추적**: 입법계획/예고(입법 전) → 심사/표결(입법 중) → NABO 분석(입법 후) 통합
4. **REST API + OpenAPI 제공**: ChatGPT GPTs Actions 연동 가능 (`/openapi.json` 엔드포인트)
5. **이중 Transport**: 로컬 (Claude Desktop stdio) + 원격 (fly.dev HTTP) — 설치 없이 URL만으로 사용 가능
6. **CLI 지원**: 터미널에서 직접 `./src/cli.ts`로 국회 데이터 조회

**기술적 패턴**:

```typescript
// TypeScript + Node.js — 도메인 엔티티 기반 통합
const server = Server("assembly-api-mcp");

// 하나의 Tool이 여러 API를 자동 호출
@server.call_tool()
async function call_tool(name: string, args: Record<string, unknown>) {
  if (name === "assembly_bill") {
    // 자동 감지: 파라미터 조합으로 "검색" vs "추적" vs "통계" 모드 전환
    // Promise.allSettled로 부분 실패 격리 (API 하나 실패해도 전체 차단 안 됨)
  }
}

// Lite/Full 프로필 전환
const MCP_PROFILE = process.env.MCP_PROFILE || "lite"; // 6개 vs 11개 도구
```

**비교: open-assembly-mcp vs assembly-api-mcp**:

| 항목 | open-assembly-mcp | assembly-api-mcp (실제) |
|------|-------------------|------------------------|
| 언어 | Python | **TypeScript** |
| 도구 수 | 12개 (개별 API 1:1 매핑) | **6개 Lite / 11개 Full (도메인 통합)** |
| API 커버 | 의안/위원/표결/위원회 | **287개 API (국회 + 국민참여입법 + NABO)** |
| Transport | stdio만 | **stdio + HTTP ( dual )** |
| OpenAPI | 없음 | **제공 (/openapi.json)** |
| CLI | 없음 | **지원** |
| 배포 | PyPI/uvx | **npm + 원격 URL (fly.dev)** |
| 관리 | 커뮤니티 (kyusik-yang) | **hollobit (활발 유지보수)** |

### 2.3 두 사례의 공통 패턴

|패턴|korean-law-mcp|assembly-api-mcp|
|---|---|---|
| **출발점** | 법제처 Open API | 열린국회정보 Open API |
| **도구 수** | 58개 | 6개 Lite / 11개 Full |
| **UI/Language** | TypeScript | TypeScript (Node.js) |
| **설치 방식** | npm 글로벌, Claude Desktop | npm + CLI + 원격 URL (fly.dev) |
| **추가 가공** | HWP→Markdown, 약칭 변환 |입법 라이프사이클 통합 추적 |
| **배포** | npm registry, fly.dev | npm registry, fly.dev |
| **인증** | 법제처 API Key (무료, 이메일 가입) | 열린국회정보 API Key (무료, 이메일 가입) |
| **OpenAPI 스펙** | 자체 Tool schema | **자체 Tool schema + REST API (/openapi.json)** |
| **Transport** | stdio | **stdio + HTTP dual** |
| **실측 에러 처리** | resultCode 매핑 | resultCode 매핑 |

`★ Insight ─────────────────────────────────────`
**공통 발견**: 두 MCP 서버 모두 **단일 기관**의 API를 래핑. assembly-api-mcp는 Lite/Full 이중 프로필로 AI 에이전트의 토큰 제약과 심층 분석 필요를 동시에 지원. 통합을 위해서는 이들을 **연결(연계)**하는 상위 계층이 필요함.
`─────────────────────────────────────────────────`

---

## 3. MCP 전환 전략: 파편화 해소를 위한 3단계 접근법

### 3.1 전략 개요

```
┌─────────────────────────────────────────────────────────────┐
│  Phase 1: 개별 MCP 서버 구축 (기관별)                        │
│  ──────────────────────────────────────────────────────────  │
│  각 기관별 API → MCP Tool로 변환                             │
│  예: 기상청 MCP, 통계청 MCP, 국토부 MCP                      │
├─────────────────────────────────────────────────────────────┤
│  Phase 2: MCP 서버 간 Federation (상호 연결)                 │
│  ──────────────────────────────────────────────────────────  │
│ Tool 호출 결과를 다른 MCP 서버에 연결                         │
│  예: "부동산 실거래가" → "해당 지역 후보 정당" 자동 연결      │
├─────────────────────────────────────────────────────────────┤
│  Phase 3: 통합 Public API MCP 게이트웨이                     │
│  ──────────────────────────────────────────────────────────  │
│  단일 Entry Point로 60+ 포털 통합 조회                        │
│  예: "최근 통과된 근로법 관련 의안과 해석례" 한 번에 조회     │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Phase 1: 개별 MCP 서버 구축

**우선순위 매트릭스 (Impact × 개발 난이도)**:

| Impact ↓ / 난이도 → | 저위험 | 고위험 |
|---------------------|---------|---------|
| **高 Impact** | ★ **1순위** (기상, 부동산, 경제지표) | ▲ **2순위** (KOSIS, ECOS) — 스파이크 필요 |
| **低 Impact** | ○ **3순위** (문화, 도서관) | ✕ **후순위** |

**1순위 구현 대상 (저위험 + 고영향)**:

| 순위 | 대상 API | 기관 | 이유 | 현재 상태 |
|------|---------|------|------|----------|
| 1 | **부동산 실거래가** | 국토부 | 수요 최상위, Pattern A로 구현 쉬움 | MCP 없음 |
| 2 | **기상청 중기예보/특보** | 기상청 | 생활 밀착, Pattern C (apihub.kma.go.kr에 전체 API) | basic MCP exists |
| 3 | **에어코리아 대기정보** | 환경부 | Pattern A, 구현 쉬움 | MCP 없음 |
| 4 | **기업 재무정보 (DART)** | 금감원 | Pattern B, 재무제표 분석 수요 높음 | dartpoint-mcp 존재 (3 stars) |
| 5 | **부동산 시세 (R-ONE)** | 한국부동산원 | Pattern B,주택가격 예측 수요 | MCP 없음 |

**구축 표준**:

```yaml
# MCP 서버 구축 표준 (모든 기관 공통)

Naming Convention:
  - 도메인: {기관명}-{카테고리}-mcp
  - 예: kma-weather-mcp, mol-worknet-mcp

Tool Naming:
  - 동사: search_, get_, lookup_, fetch_, list_
  - 명사: weather_forecast, bill_detail, member_info

Tool Description:
  - 한국어 상세 설명 필수 (AI가 올바른 Tool 선택을 위해)
  - 파라미터单位和 설명 포함
  - 예: "YYYYMMDD 형식, 예: 20260412"

에러 처리:
  - HTTP 200 + body 내부 에러코드 매핑 필수
  - 사용자에게 명확한 에러 메시지 반환

응답 정규화:
  - XML → JSON 변환
  - 한글 필드명 → 영어/카멜케이스 변환
  - 날짜/시간 형식 통일 (ISO 8601)

인증:
  - 환경변수 사용 (하드코딩 금지)
  - API Key 유효기간 만료 시 자동 감지 및 알림
```

### 3.3 Phase 2: MCP Federation (상호 연결)

**개념**: 개별 MCP 서버를 **연쇄(Chain)** 호출하여 복잡한 질의 해결

**예시 시나리오**:

```
사용자: "최근 1년 내 반도체 관련 규제 완화 의안을 제안한 의원을
        중심으로, 해당 의원의 소속 정당과 현재 주요 부처 추진 
        정책을 연관지어 알려줘"

 MCP Chain:
 ① assembly-api-mcp → 반도체 관련 의안 검색
 ② → 제안자 정보 (해당议员)
 ③ korean-law-mcp → 해당 의원이 다른 의안 제안했는지 검색
 ④ → 정당 정보 확인
 ⑤ gov.kr-mcp → 해당 부처 추진 정책 조회
 ⑥ → 최종 결과 취합
```

**Federation 구현 패턴**:

```python
# 각 MCP 서버가 독립 실행 (.stdio)
# Federation은 상위 레이어에서 조정

# 예: 통합 검색 서비스
class PublicDataFederation:
    """60+ 공공 API MCP 서버의 페더레이션"""

    def __init__(self):
        self.servers = {
            "assembly": AssemblyMCPServer(),
            "law": LawMCPServer(),
            "weather": WeatherMCPServer(),
            "real_estate": RealEstateMCPServer(),
            # ... 추가
        }

    async def query(self, user_question: str) -> list[ToolResult]:
        # 1. 질문 분석 → 관련 MCP 서버 식별
        relevant_servers = self.route(user_question)

        # 2. 병렬 호출
        results = await asyncio.gather(*[
            server.search(query) for server in relevant_servers
        ])

        # 3. 결과 통합 및 순위화
        return self.merge(results)
```

### 3.4 Phase 3: 통합 Public API MCP 게이트웨이

**목표**: 단일 MCP 서버로 **60+ 포털 통합 query**

**Architecture**:

```
┌─────────────────────────────────────────────────────────┐
│                  Unified Public API Gateway              │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Natural Language Query → Tool Routing Layer       │ │
│  │  "최근 통과된 근로법 수정안과 관련 판례를 검색"       │ │
│  └─────────────────────────────────────────────────────┘ │
│                           ↓                              │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  60+ MCP Server Federation                         │ │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐      │ │
│  │  │Assembly│ │  Law   │ │  KMA   │ │ DART   │ ...  │ │
│  │  │  MCP   │ │  MCP   │ │  MCP   │ │  MCP   │      │ │
│  │  └────────┘ └────────┘ └────────┘ └────────┘      │ │
│  └─────────────────────────────────────────────────────┘ │
│                           ↓                              │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Response Normalization & Ranking                    │ │
│  │  - 중복 데이터 통합                                   │ │
│  │  - 신뢰도 기반 순위화                                 │ │
│  │  - 출처 명시                                         │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

**Tool 설계 예시**:

```python
UNIFIED_PUBLIC_API_TOOL = Tool(
    name="search_public_data",
    description=(
        "한국 공공 데이터 통합 검색. 60+ 공공 포털의 API를 "
        "통일된 인터페이스로 검색한다. "
        "카테고리: 법령, 의안, 통계, 부동산, 날씨, 기업정보, 채용 등. "
        "검색 결과를 출처와 신뢰도 정보와 함께 반환한다."
    ),
    inputSchema={
        "type": "object",
        "properties": {
            "query": {
                "type": "string",
                "description": "자연어 검색어 (예: '반도체 규제 완화 의안 2025')",
            },
            "category": {
                "type": "string",
                "enum": ["law", "assembly", "economy", "real_estate",
                         "weather", "corporate", "job", "all"],
                "description": "검색 범주. 'all'이면 전체 검색.",
            },
            "limit": {
                "type": "integer",
                "description": "반환 결과 수 (기본 10, 최대 50)",
            },
        },
        "required": ["query"],
    },
)
```

---

## 4. 통합 전략: 파편화를 넘어서는 5가지 원칙

### 4.1 원칙 1: "단일 정답"보다 "연속적 스펙트럼"

기존 접근: "공공 API를 이렇게 바꿔야 한다" (강제적)
개선 접근: **점진적 전환** — 각 기관의 현재 상태에서 시작하여 개선

| 현재 상태 | 목표 | 방법 |
|-----------|------|------|
| XML only | XML + JSON 선택 → JSON 기본 | API 버저닝으로 자연 전이 |
| HTTP 200 일관 | 에러 시 4xx 반환 (점진적) | 응답 형식 표준 가이드 배포 |
| Closed API | OpenAPI 스펙 공개 (권장) | incentive（우대/인증）제공 |
| CORS 미지원 | 서버사이드 MCP로 해결 | CORS 지원을 신규 의무화 |

### 4.2 원칙 2: "기능 통합"보다 "인증 통합"

가장 큰 마찰 비용: **계정 관리**

```
현재: 개발자가 60+ 포털에 각각 가입해야 함
이상: Single Sign-On (공공 Portal 통합 인증)

제안: "공공 API 통합 계정" 도입
- 기존: 각 기관별 이메일/본인인증
- 통합: 정부 SSO (GPKI 연동) + 통합 API Key 발급
```

### 4.3 원칙 3: "표준 강제"보다 "어댑터 Layer"

 기관별로 기존 API를 유지하면서, **MCP 어댑터 계층**을 통해 통합

```
┌─────────────────────────────────────────┐
│           AI Agent (Claude 등)          │
└──────────────────┬──────────────────────┘
                   │ MCP 프로토콜
┌──────────────────┴──────────────────────┐
│         MCP Gateway (통합 진입점)         │
│  - Tool Discovery                       │
│  - Query Routing                         │
│  - Response Normalization                │
└──────────────────┬──────────────────────┘
                   │
      ┌────────────┼────────────┐
      ↓            ↓            ↓
┌──────────┐ ┌──────────┐ ┌──────────┐
│ 법제처    │ │ 기상청    │ │ 통계청   │
│ Adapter  │ │ Adapter  │ │ Adapter  │
└────┬─────┘ └────┬─────┘ └────┬─────┘
     ↓            ↓            ↓
  기존 API     기존 API     기존 API
  (변경 없음)   (변경 없음)   (변경 없음)
```

### 4.4 원칙 4: "중앙 관리"보다 "분산 협업"

한국 공공 API의 규모(60+ 포털, 수천 API)를 감안하면 **중앙 팀의 단일 책임**은 비현실적.

**개선 접근: Federated Ownership**

| 기관 | 담당 범위 | 현재 상태 |
|------|----------|----------|
| 법제처 | korean-law-mcp (자율 유지) | ✅ 1,344 stars, 활발 유지 |
| 광진구청 | korean-law-mcp (실무 개발) | ✅ 유지보수 중 |
| hollobit | assembly-api-mcp | ✅ npm + 원격 URL, 활발 유지보수 중 (v0.7.0) |
| Koomook | data-go-mcp-servers | ✅ 233 stars, 유지보수 중 |

**정부의 역할**: 개발자 생태계支援（재정 지원, 인센티브, 표준 가이드 배포）

### 4.5 원칙 5: "기술 해결"보다 "데이터 거버넌스"

MCP 서버를 만드는 것은 기술적 작업이지만, **지속 가능한 생태계**를 위해 필요한 것은 데이터 거버넌스다.

| 요소 | 현재 | 목표 |
|------|------|------|
| API 변경 알림 | 없음/임의 | 사전 공지 + 버전 관리 |
| 폐기 예고 | 없음 | 6개월 전 의무 예고 |
| 데이터 품질 모니터링 | 없음 | 활용량/에러율 대시보드 |
| 오픈소스 기여 가이드 | 없음 | 정부 주도 MCP 컨트리뷰션 가이드 |

`★ Insight ─────────────────────────────────────`
**핵심 통찰**: MCP 서버는 기술적 solution이지만, 지속 가능하려면 **데이터 거버넌스**가 병행되어야 함. 기술만 있고 거버넌스가 없으면 "잃어버린 MCP 제공기"가 됨.
`─────────────────────────────────────────────────`

---

## 5. 실행 로드맵

### 5.1 단기 (6개월): 개별 MCP 서버 집중 구축

**목표**: 가장 수요 높고 개발 난이도 낮은 API부터 MCP 전환

```
월 1-2: 부동산 실거래가 MCP 서버 구축
  - data.go.kr Pattern A (serviceKey만으로 호출)
  - Tool: search_real_estate_deals, get_apt_price
  - 담당: 커뮤니티 기여자 1명

월 3-4: 통계청 KOSIS MCP 서버 구축
  - JSON 지원, 자동승인
  - Tool: search_statistics, get_gdp, get_population
  - 담당: 커뮤니티 기여자 1명

월 5-6: 한국은행 ECOS MCP 서버 구축
  - JSON 지원, 자동승인
  - Tool: search_economic_indicators, get_interest_rate
  - 담당: 커뮤니티 기여자 1명
```

**KPI**:
- MCP 서버 3개 새롭게 증가
- GitHub Stars 합계 100+ 증가
- 활용 개발자 수 50+

### 5.2 중기 (12개월): MCP Federation 실험

**목표**: 개별 MCP 서버 간 연결 실험

```
월 7-9: MCP Federation 프로토타입
  - "의안 →국회의원 → 정당 → 정책" 검색 체인 실험
  - assembly-api-mcp + korean-law-mcp + gov.kr-mcp 연동

월 10-12: 통합 Gateway 베타
  - 단일 Entry Point로 10개 MCP 서버 연동
  - Natural Language Query 테스트
  - 성능/정확도 측정
```

**KPI**:
- Federation 체인 3개 이상 동작
- Gateway Beta 사용자 100+
- 응답 정확도 80%+

### 5.3 장기 (24개월): 공공 API 표준화 제안

**목표**: MCP 전환을 통한 공공 API 전체 품질 향상

```
연도 2: 정부 정책 제언
  - 공공누리 제4유형 활용 가이드 공식 배포
  - 공공 API OpenAPI 스펙 의무화 권고안
  -"MCP-Ready" 공공 API 인증 제도提案

연도 2: 교육 및 확산
  - 공공 기관 담당자 대상 MCP 교육
  - 우수 사례 공유 포럼
  - 정부 소속 개발자 MCP 개발 보조금
```

---

## 6. 기술 표준: MCP 서버 구축 가이드

### 6.1 필수 구현 요소

모든 공공 API용 MCP 서버는 다음 요소를 반드시 구현해야 함:

```yaml
Required Elements:
  Tool Definition:
    - name: 영문 + snake_case (search_bill_details)
    - description: 한국어 상세 설명 (500자 이상)
    - inputSchema: 엄격한 타입 정의
    - required: 필수 파라미터 명확히

  Error Handling:
    - HTTP 레벨 에러 체크 (status_code != 200)
    - Body 내부 에러코드 파싱 (resultCode 등)
    - 사용자 친화적 에러 메시지 반환

  Response Normalization:
    - XML → JSON 변환
    - 한글 필드명 → 영어 변환
    - 날짜/시간 → ISO 8601
    - Null 처리 일관되게

  Authentication:
    - 환경변수 사용 (하드코딩 금지)
    - Key 만료 감지 및 알림
    - Rate Limit 초과 시 재시도 로직

  Documentation:
    - README.md: 설치/설정 방법
    - docs/API.md: Tool 참조
    - docs/ARCHITECTURE.md: 아키텍처 설명
```

### 6.2 권장 구현 요소

```yaml
Recommended Elements:
  Convenience Tools:
    - 복수 API 호출을 묶은 요약 Tool
    - 예: get_bill_summary (search → detail → proposer 한번에)

  Cache Layer:
    - 동일 데이터 반복 호출 방지
    - TTL 설정 (데이터 성격에 따라 5분~24시간)

  Rate Limit Handling:
    - X-RateLimit 헤더 확인 (있는 경우)
    - 초과 시 exponential backoff
    - 사용자에게 호출 한도 정보 제공

  User Context Preservation:
    - Tool description에 사용 맥락 포함
    - 예: "법률 조문의 적용 사례를 찾을 때 유용"
```

### 6.3 배포 표준

```yaml
Deployment Options (우선순위 순):

1. npm (TypeScript)
   - npm install -g 한 줄 설치
   - 예: npm install -g assembly-api-mcp, npm install -g korean-law-mcp

2. Claude Desktop Integration
   - claude_desktop_config.json 설정 파일 제공
   - --setup 마법사 제공 (가능한 경우)

3. Cloud Deployment (선택)
   - fly.dev, Railway 등
   - 설치 없이 URL 등록만으로 사용 가능
```

---

## 7. 예상 비용 및 인센티브

### 7.1 개발 비용 추정 (단일 MCP 서버)

| 항목 | 시간 (시간) | 비용 (만원) |
|------|-----------|-------------|
| API 스펙 분석 | 8~16 | 80~160 |
| MCP 서버 구현 | 16~32 | 160~320 |
| 테스트 및 디버깅 | 8~16 | 80~160 |
| 문서화 | 4~8 | 40~80 |
| **총계** | **36~72** | **360~720** |

### 7.2 인센티브 제안

정부가 MCP 생태계를 촉진하기 위한 인센티브:

| 유형 | 내용 | 대상 |
|------|------|------|
| **재정 지원** | 공공 API용 MCP 서버 개발 시 开发 보조금 500만원 | 개별 개발자/팀 |
| **인증 우대** | MCP-Ready 인증を取得한 API 우대 조건 | 공공 기관 |
| **표준 채택 인센티브** | OpenAPI 스펙 제공 시 우선 노출 | 공공 데이터 포털 |
| **우수 오픈소스 포상** | GitHub Stars 500+ 달성 시 포상 | 개발자/팀 |

---

## 8. 결론 및 추천

### 8.1 핵심 제안 3가지

```
┌─────────────────────────────────────────────────────────┐
│  1. "MCP 어댑터 계층" 도입                                │
│  ────────────────────────────────────────────────────     │
│  기존 API는 그대로 유지. 기관별 어댑터(MCP 서버)를       │
│  구축하여 통합 Gateway에서 연결.                          │
│  → 기존 투자 활용 + 점진적 개선 가능                     │
├─────────────────────────────────────────────────────────┤
│  2. "인증 통합"을 최우선 과제로 설정                      │
│  ────────────────────────────────────────────────────     │
│  60+ 포털 각각 가입의 friction을 줄이는 것이              │
│  개발자 채택률 결정. 통합 SSO + 통합 API Key 도입.        │
├─────────────────────────────────────────────────────────┤
│  3. " federated ownership" 모델 도입                     │
│  ────────────────────────────────────────────────────     │
│  중앙 집중이 아닌, 기관별/커뮤니티별 소유.                 │
│  정부는 표준 가이드 + 인센티브 제공 역할.                 │
└─────────────────────────────────────────────────────────┘
```

### 8.2 기대 효과

| 지표 | 현재 | 2년 후 목표 |
|------|------|------------|
| MCP 서버 수 | ~10개 (추정) | 50+ |
| 커버 포털 수 | ~10개 | 30+ |
| 평균 개발 시간 (API 1개) | 1주 (매번 반복) | 1시간 (재사용) |
| AI 에이전트 활용률 | ~5% | 30%+ |
| 개발자 만족도 (1~5) | 2.1 | 4.0 |

`★ Insight ─────────────────────────────────────`
**비전**: 2년 후, 개발자가 공공 API를 사용할 때 "어떤 MCP 서버를 연결하지?"가 기본 질문이 되는 세계. 정부도 "우리 API가 MCP로 제공되는가?"를 확인하는 시대.
`─────────────────────────────────────────────────`

---

## 참고 자료

### MCP 서버 저장소

- [chrisryugj/korean-law-mcp](https://github.com/chrisryugj/korean-law-mcp) — 1,344 stars, 법령/판례/행정규칙 58개 Tool
- [hollobit/assembly-api-mcp](https://github.com/hollobit/assembly-api-mcp) — TypeScript, 287개 API 커버 (6개 Lite/11개 Full 도구), 원격 URL 제공
- [Koomook/data-go-mcp-servers](https://github.com/Koomook/data-go-mcp-servers) — 233 stars, data.go.kr 6개 API
- [hjsh200219/korea-public-data-mcp](https://github.com/hjsh200219/lawcase-search-mcp) — 법제처 + DART + 공공데이터 통합 34개 Tool
- [SeoNaRu/korean-law-mcp](https://github.com/SeoNaRu/korean-law-mcp) — Python, 24시간 캐싱
- [dikehomme/kr-law-mcp](https://github.com/dikehomme/kr-law-mcp) — 9개 Tool 간소화 버전

### 관련 문서

- [MCP 기반 공공데이터 API와 오픈소스 프로젝트 정리](https://cognitive.tistory.com/496)
- [한국 공공 API 전수 조사 Plans.md](./Plans.md)
- [MCP 서버 개발 가이드](./mcp_guide.md)

---

*이 문서는「대한민국 공공 Open API 전수 조사」프로젝트의 산출물입니다.*