# 해외 정부 MCP/AI 도구 생태계 전수 조사

> 조사 기준일: 2026년 4월 3일
> 본 조사는 정부 공식 MCP 서버 및 공공 API를 대상으로 한 커뮤니티 MCP 서버를 전수 조사한 결과이다.

---

## 1. 정부 공식 MCP 서버

### 🇺🇸 미국 — GovInfo MCP (GPO)

| 항목 | 내용 |
|------|------|
| **제공기관** | Government Publishing Office (GPO) |
| **URL** | https://www.govinfo.gov/features/mcp-public-preview |
| **GitHub** | https://github.com/usgpo/api |
| **상태** | 퍼블릭 프리뷰 (2026년 1월 22일 런칭) |
| **접근 방식** | 공개 (인증 불필요, 단 GovInfo API 키 권장) |
| **지원 도구 수** | 기본 검색·조회 도구 (초기 릴리스, 점진 확장 예정) |
| **지원 AI 클라이언트** | Claude, ChatGPT, Gemini, Microsoft Copilot |

**커버리지**:
- 연방 관보 (Federal Register)
- 미국 법령 (U.S. Code, Public Laws)
- 의회 기록 (Congressional Record, Bills, Reports)
- 대통령 문서 (Executive Orders, Presidential Proclamations)
- 예산안 (Budget of the U.S. Government)
- 규정집 (Code of Federal Regulations)
- 미국 법원 의견서 (Slip Opinions)

**의의**: 미국 정부 최초의 공식 MCP 서버. GPO가 GovInfo를 "세계에서 유일하게 인증된 신뢰할 수 있는 디지털 레포지토리"로 정의하고, LLM이 공식 연방 문서에 직접 접근할 수 있도록 했다. 의회 데이터 태스크포스(2025년 12월)에서도 주요 AI 연동 사례로 소개되었다.

---

### 🇫🇷 프랑스 — data.gouv.fr MCP (DINUM/Etalab)

| 항목 | 내용 |
|------|------|
| **제공기관** | Direction interministérielle du numérique (DINUM) / Etalab |
| **MCP 엔드포인트** | https://mcp.data.gouv.fr/mcp |
| **GitHub** | https://github.com/datagouv/datagouv-mcp |
| **상태** | 프로덕션 운영 중 |
| **접근 방식** | 완전 공개 (무인증) |
| **지원 도구 수** | 7개 |
| **지원 AI 클라이언트** | Claude, ChatGPT, Gemini 호환 (MCP 표준 준수) |

**7개 도구 목록**:
1. `search_datasets` — 데이터셋 전문 검색
2. `get_dataset_info` — 데이터셋 메타데이터 조회
3. `list_dataset_resources` — 연결 리소스 목록
4. `get_resource_info` — 리소스 메타데이터
5. `query_resource_data` — 데이터 직접 쿼리
6. `download_and_parse_resource` — 리소스 다운로드 및 분석
7. `get_metrics` — 이용 지표 조회

**커버리지**: 4만 개 이상 데이터셋. 지리, 인구통계, 교통, 환경, 예산, 선거 등 전 분야.

**의의**: 세계에서 정부가 공식 운영하는 MCP 서버 두 곳 중 하나(미국 GovInfo와 함께). Etalab의 오픈소스 기조에 따라 코드 전체 공개, 누구나 포크 가능. 2025년 프랑스 정부 기술 블로그(alliance.numerique.gouv.fr)에서 "AI와 국가 살아있는 데이터를 연결하는 표준"으로 공식 소개.

---

### 🇬🇧 영국 — i.AI Parliament MCP (Incubator for AI)

| 항목 | 내용 |
|------|------|
| **제공기관** | Incubator for Artificial Intelligence (i.AI), Cabinet Office |
| **GitHub** | https://github.com/i-dot-ai/parliament-mcp |
| **상태** | 운영 중 (2025년 11월 Cabinet Office·No10에 배포) |
| **접근 방식** | 공개 (오픈소스) |
| **지원 도구 수** | 13개 |
| **지원 AI 클라이언트** | Claude, Humphrey AI 툴킷 내 통합 |
| **활성 사용자** | 700명+ (2025년 11월 기준) |

**13개 도구 포함 기능**:
- 의원 검색·상세 조회, 선거 결과, 의석 현황
- 장관직 목록, 정부 직책 조회
- 한사드(의회 토론록) 의미론적 검색 (Azure OpenAI 임베딩 기반)
- 서면 질의 검색

**의의**: 영국 정부 AI 기관(i.AI)이 직접 개발한 의회 데이터 MCP. "Humphrey" AI 툴킷의 일부로, 공무원이 의회 데이터에 AI로 접근하는 21세기 정부 인프라 지향. 정부가 자체 개발하여 내부에 배포한 사례로 희소성이 높다.

---

## 2. 커뮤니티 MCP 서버 (국가별)

### 🇰🇷 한국

한국은 커뮤니티 MCP 생태계가 가장 활발한 나라 중 하나이다. 공공데이터포털(data.go.kr)의 광범위한 API 카탈로그와 활발한 개발자 커뮤니티가 결합된 결과다.

| 프로젝트명 | GitHub URL | 대상 API | 도구 수 | 언어 | 비고 |
|------------|------------|---------|---------|------|------|
| data-go-mcp-servers | https://github.com/Koomook/data-go-mcp-servers | 공공데이터포털 (data.go.kr) | 다수 | TypeScript | 사업자등록, 국민연금, 조달 등 |
| opendata-mcp | https://github.com/ceami/opendata-mcp | 공공데이터포털 | 3+ | Python | search_api, get_std_docs, fetch_data |
| korean-law-mcp (woongaro) | https://github.com/woongaro/korean-law-mcp | 법제처 국가법령정보 API | 다수 | TypeScript | 법령·판례·헌재결정 |
| korean-law-mcp (chrisryugj) | https://github.com/chrisryugj/korean-law-mcp | 법제처 Open API | 87개 | TypeScript | CLI + npm 패키지, 가장 완성도 높음 |
| lexguard-mcp | https://github.com/SeoNaRu/lexguard-mcp | 법제처 API | 159개 | — | 일반인 대상 법률 정보 조회 |
| dartpoint-mcp | https://github.com/dartpointai/dartpoint-mcp | DART (금융감독원 공시) | 다수 | TypeScript | 기업 공시·재무 데이터 |
| data4library-mcp | https://github.com/isnow890/data4library-mcp | 도서관정보나루 API | 25+ | TypeScript | 도서 검색, 인기 트렌드, GPS 기반 도서관 |
| seoul-data-mcp | https://github.com/pinnaclesoft-ko/be-node-seoul-data-mcp | 서울 열린데이터광장 | 다수 | Node.js | 서울시 공공데이터 |

**특징**: 법령 분야에서 특히 활발(3개 이상의 독립 MCP 서버). 공공데이터포털 기반 범용 MCP(opendata-mcp, data-go-mcp-servers)가 공존. 도서관 API처럼 틈새 분야도 커버.

---

### 🇯🇵 일본

일본은 법령 API와 통계 API 분야에서 매우 활발한 MCP 생태계를 보유하고 있다. e-Gov 법령API v2(2025년 3월)가 무인증·OpenAPI 스펙·완전 JSON을 지원하면서 커뮤니티 반응이 폭발적이었다.

| 프로젝트명 | GitHub URL | 대상 API | 도구 수 | 비고 |
|------------|------------|---------|---------|------|
| estat-mcp | https://github.com/ajtgjmdjp/estat-mcp | e-Stat (정부통계 포털) | 다수 | 10만+ 통계 테이블 LLM 접근 |
| estat-mcp-server | https://github.com/cygkichi/estat-mcp-server | e-Stat | 다수 | Docker 지원 대안 구현 |
| e-gov-law-mcp | https://github.com/ryoooo/e-gov-law-mcp | e-Gov 법령API v2 | 다수 | 일본 법령 검색·조회 |
| hourei-mcp-server | https://github.com/groundcobra009/hourei-mcp-server | e-Gov 법령API | 다수 | 법령 검색 대안 구현 |
| tax-law-mcp | https://github.com/kentaroajisaka/tax-law-mcp | e-Gov + 국세청 통달 | 다수 | 세법 + 국세청 해석 통합 |
| labor-law-mcp | https://github.com/kentaroajisaka/labor-law-mcp | e-Gov + 후생노동성 | 다수 | 노동·사회보험법 |
| gbizinfo-mcp | https://glama.ai/mcp/servers/@koizumikento/gbixnfo-mcp | gBizINFO (METI) | 다수 | 법인번호 기반 기업정보 |
| e-gov-mcp | https://lobehub.com/mcp/go-555-e-gov-mcp | e-Gov 전반 | 다수 | 범용 e-Gov 통합 |

**특징**: e-Gov 법령API v2가 "키 없음 + OpenAPI 스펙 + JSON" 3박자를 갖추면서 MCP 서버 제작이 폭발적으로 증가. 세법, 노동법 등 전문 분야 분화가 뚜렷. 한국과 달리 법령 API가 공식적으로 AI 친화적으로 설계되어 커뮤니티 활성화에 유리.

---

### 🇸🇬 싱가포르

| 프로젝트명 | GitHub URL | 대상 API | 도구 수 | 비고 |
|------------|------------|---------|---------|------|
| mcp-sg-lta | https://github.com/arjunkmrm/mcp-sg-lta | LTA DataMall (육상교통청) | 6+ | 버스 도착, MRT 혼잡도, 주차 실시간 |
| Singapore-Data-MCPs | https://github.com/prezgamer/Singapore-Data-MCPs | data.gov.sg + NEA | 다수 | 날씨·환경 + 공공데이터 통합 |
| MCP-Public-Transport | https://github.com/siva-sub/MCP-Public-Transport | LTA DataMall | 다수 | 대중교통 전문 |

**특징**: LTA DataMall이 즉시 API 키 발급 + 인증 간소화 정책 덕분에 MCP 개발이 쉬움. data.gov.sg의 무인증 공개 API가 AI 통합의 진입 장벽을 낮춤.

---

### 🇺🇸 미국 (커뮤니티)

정부 공식 GovInfo MCP 외에도 활발한 커뮤니티 MCP 생태계가 존재한다.

| 프로젝트명 | GitHub URL | 대상 API | 비고 |
|------------|------------|---------|------|
| mcp-congress_gov_server | https://github.com/bsmi021/mcp-congress_gov_server | Congress.gov API v3 | 법안·의회 데이터 |
| us-legal-mcp | https://github.com/JamesANZ/us-legal-mcp | 미국 법령 종합 | 법령 전문 |
| datagov-mcp-server | https://github.com/melaodoidao/datagov-mcp-server | Data.gov | 연방 오픈데이터 |
| us-gov-open-data-mcp | https://lzinga.github.io/us-gov-open-data-mcp/ | 36개 연방 API | 188개 도구, 교차 참조 지원 |

---

### 🇬🇧 영국 (커뮤니티)

| 프로젝트명 | GitHub URL | 대상 API | 비고 |
|------------|------------|---------|------|
| GovUK-MCP | https://github.com/Stealth-Labs-LTD/GovUK-MCP | GOV.UK API 카탈로그, TfL, 환경청 | 범용 영국 정부 MCP |
| uk-ons-mcp-server | https://github.com/dwain-barnes/uk-ons-mcp-server | ONS (국가통계청) | 인구·경제 통계 |

---

### 기타 국가

| 국가 | 프로젝트명 | 대상 | 비고 |
|------|------------|------|------|
| 🇭🇰 홍콩 | Open Data HK MCP | 홍콩 오픈데이터 | https://github.com/mcp-open-data-hk |
| 🌐 다국가 | ckan-mcp-server | CKAN 기반 포털 범용 | data.gouv.fr, data.gov.au 등 지원 |

---

## 3. MCP 서버 기술 비교표

| 국가 | 공식 MCP | 커뮤니티 MCP 수 | 최대 도구 수 | 주요 분야 | AI 클라이언트 지원 |
|------|---------|----------------|-------------|-----------|-------------------|
| 🇺🇸 미국 | GovInfo MCP (GPO, 2026.01) | 4+ | 188 (커뮤니티) | 법령·재정·의회 | Claude/ChatGPT/Gemini/Copilot |
| 🇫🇷 프랑스 | data.gouv.fr MCP (DINUM) | 2+ | 7 (공식) | 오픈데이터 전반 | Claude/ChatGPT/Gemini |
| 🇬🇧 영국 | Parliament MCP (i.AI, 비공개 배포) | 2+ | 13 (공식) | 의회·통계·교통 | Claude/Humphrey |
| 🇯🇵 일본 | 없음 | 8+ | 다수 | 법령·통계·기업정보 | Claude 중심 |
| 🇰🇷 한국 | 없음 | 8+ | 159 (커뮤니티) | 법령·공시·도서관 | Claude 중심 |
| 🇸🇬 싱가포르 | 없음 | 3+ | 다수 | 교통·환경 | Claude 중심 |

---

## 4. MCP 커버리지 분석 (전체 정부 API 대비 MCP 커버율)

| 국가 | 추정 전체 공공 API 수 | MCP 커버 API 수(추정) | 커버율 | 주요 공백 |
|------|---------------------|----------------------|--------|-----------|
| 🇰🇷 한국 | ~19,198 (data.go.kr 기준) | ~50개 API 커버 | 0.26% | 복지·보건·교통·기상 등 대부분 |
| 🇺🇸 미국 | ~2,500+ (api.data.gov 기준) | ~100+ (GovInfo + 커뮤니티) | ~4% | 기상·보건 분야 |
| 🇬🇧 영국 | ~200 (api.gov.uk 등록 기준) | ~20 | ~10% | HMRC 세무·NHS 의료 |
| 🇫🇷 프랑스 | ~166 (api.gouv.fr) + 40,000 데이터셋 | 7 도구로 40,000 데이터셋 접근 | 데이터셋 100% | 행정 G2G API |
| 🇯🇵 일본 | ~15개 주요 포털 | ~8개 API 커버 | ~53% (주요 포털 기준) | 특허·지방정부 API |
| 🇸🇬 싱가포르 | ~100 (data.gov.sg + APEX) | ~3개 API 커버 | ~3% | APEX G2G, 세무, 신원 |

**해석**: 
- 프랑스 data.gouv.fr MCP는 단일 서버로 4만 개 데이터셋에 접근할 수 있어 커버리지 효율이 압도적으로 높다. 이는 플랫폼 중심 MCP 설계의 장점이다.
- 한국은 API 수가 가장 많으나 MCP 커버율이 가장 낮다. 개별 API 단위 MCP 서버 개발 방식의 한계.
- 일본은 주요 포털 기준으로는 커버율이 상대적으로 높으나 실제 API 수는 소수에 불과하다.

---

## 5. MCP 개발이 용이한 조건 분석

### MCP 서버가 만들어지는 API의 공통 조건

**조건 1: 무인증 또는 즉시 키 발급**
- e-Gov 법령API v2 (일본): 키 없음 → MCP 서버 8개 이상
- data.gov.sg (싱가포르): 키 없이 테스트 가능 → MCP 서버 3개
- LTA DataMall (싱가포르): 즉시 키 발급 → MCP 서버 3개
- 반례: 한국 data.go.kr — URL 인코딩 이중화 이슈, CORS 없음 → MCP 개발 난이도 증가

**조건 2: JSON 네이티브 + OpenAPI 스펙 제공**
- data.gouv.fr: JSON + OpenAPI → 공식 MCP 서버 제작 가능
- e-Gov 법령API v2: JSON + Swagger UI + Redoc → MCP 생태계 폭발
- 반례: 한국 공공데이터포털 — XML 기본값, OpenAPI 스펙 없음 → MCP 개발 시 변환 레이어 필요

**조건 3: 공식 개발자 포털과 문서 품질**
- 좋은 문서 = MCP 개발자가 API 구조를 빠르게 이해 가능
- LTA DataMall: PDF 문서이지만 명확한 엔드포인트 구조 → MCP 다수 생성
- 반례: 일본 JMA 기상 API — 비공식 엔드포인트, 문서 없음 → MCP 없음

**조건 4: 데이터의 유용성과 수요**
- 법령 (한국·일본): 법률 AI 수요 폭발로 MCP 다수
- 교통 실시간 (싱가포르): 개발자 커뮤니티 수요 높음
- 재정 공시 (미국 USAspending, 한국 DART): fintech 개발자 수요
- 반례: 특수 행정 API — 수요 낮아 MCP 부재

**조건 5: CORS 지원 (브라우저 직접 접근 가능성)**
- CORS 없는 API는 서버사이드 MCP만 가능 (클라우드 비용 발생)
- CORS 있는 API는 로컬 MCP 서버도 쉽게 구현 가능

### MCP 서버가 생기지 않는 이유

| 원인 | 해당 사례 |
|------|-----------|
| XML 전용 응답 | 한국 공공데이터포털 대부분 API |
| 기관 인증 필요 (수주 소요) | 한국 정부24, 일본 특허청 |
| URL 인코딩 버그 등 기술 결함 | 한국 serviceKey 이중 인코딩 |
| CORS 미지원 (서버 구현 어려움) | 한국·일본 대부분 API |
| OpenAPI 스펙 부재 | 한국 공공데이터포털, 일본 기상청 |
| 데이터 수요 낮음 | 행정 내부 G2G API |

---

## 6. 한국 공공 API MCP 로드맵 제안

### 단기 (2026년~2027년): "먼저 열어야 MCP가 따라온다"

**6.1 국가법령정보 MCP 공식 운영**
- 현황: 커뮤니티 구현체 3개 이상 존재 (woongaro, chrisryugj, SeoNaRu)
- 제안: 법제처가 공식 MCP 서버 채택·운영 (일본 e-Gov처럼)
- 기대효과: 법률 AI 서비스의 신뢰성·일관성 확보

**6.2 공공데이터포털 MCP 게이트웨이 개발**
- 현황: data.go.kr은 19,198개 API를 보유하나 MCP 커버율 0.26%
- 제안: 프랑스 data.gouv.fr 모델처럼 단일 MCP 서버로 전체 카탈로그 접근
- 필요 선결조건: serviceKey URL 인코딩 버그 수정, JSON 기본값 전환, CORS 허용

**6.3 API 기술 품질 개선 우선순위 목록**
1. JSON 기본 응답 전환 (현재 XML 기본값)
2. OpenAPI 3.0 스펙 의무 제공
3. CORS 헤더 허용
4. serviceKey 이중 인코딩 버그 수정
5. HTTP 표준 상태코드 준수

### 중기 (2027년~2028년): "MCP 준비된 API 인증제"

**6.4 공공 API MCP 적합성 인증 도입**
- MCP Ready API 인증 배지 부여
- 조건: JSON 네이티브 + OpenAPI 스펙 + CORS + 즉시 키 발급
- 참고: 영국 GDS API 디자인 기준, 프랑스 api.gouv.fr 품질 심사

**6.5 주요 고수요 API MCP 서버 공식 개발**
우선순위:
1. 기상청 동네예보 API (일 1,000만 호출 이상)
2. 부동산원 실거래가 API
3. 건강보험심사평가원 약가 API
4. 국토교통부 대중교통 API

### 장기 (2029년~): "AI 네이티브 공공데이터 생태계"

**6.6 범정부 AI 데이터 게이트웨이**
- 싱가포르 APEX Cloud처럼 단일 게이트웨이에서 MCP 지원
- 모든 공공 API의 MCP 래퍼 자동 생성 도구 제공
- LLM이 공공 API를 직접 발견·인증·호출할 수 있는 AI 네이티브 인프라

**6.7 목표 지표**
| 지표 | 2026 현재 | 2028 목표 | 2030 목표 |
|------|-----------|-----------|-----------|
| MCP 커버율 | 0.26% | 5% | 20% |
| MCP Ready API 수 | ~50 | ~500 | ~2,000 |
| 공식 정부 MCP 서버 | 0 | 3 | 10 |
| MCP 월간 AI 에이전트 호출 | 미집계 | 100만 | 1,000만 |

---

**참고**: 본 조사는 각국 정부 공식 발표, GitHub 공개 레포지토리, MCP 서버 디렉토리(PulseMCP, Glama, LobeHub, mcpservers.org) 등을 종합하여 작성하였다.
