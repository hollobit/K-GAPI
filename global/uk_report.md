# 🇬🇧 영국 정부 API 생태계 분석

> 작성일: 2026-04-03 | 데이터 기준: 2024–2025년 | 분석 포털: 12개

---

## 1. 개요

영국 정부의 API 생태계는 **Government Digital Service(GDS)** 가 2016년부터 수립한 API 기술·데이터 표준을 기반으로 운영된다. 미국의 api.data.gov 같은 단일 게이트웨이 모델과 달리, 영국은 각 부처가 자체 API를 독립적으로 운영하되 **GDS 표준을 의무적으로 준수**하는 분산형 거버넌스를 채택하고 있다.

범정부 API 카탈로그(`api.gov.uk`)는 인덱스 역할을 수행하며, 2024년 기준 200개 이상의 정부 API가 등록되어 있다. 각 부처 포털은 독립적 개발자 허브를 운영하며, 그 품질은 HMRC·NHS처럼 세계 최고 수준에서 DWP처럼 내부 기관 전용까지 편차가 존재한다.

### 주요 지표 요약

| 지표 | 현황 |
|------|------|
| 범정부 API 카탈로그 | api.gov.uk (200개+ 등록) |
| 표준 인증 방식 | OAuth 2.0 (GDS 의무화) |
| API Key 정책 | GDS 표준에서 명시적 비권장 ("쉽게 가로챌 수 있음") |
| 데이터 포맷 | JSON 필수 (XML 사실상 폐지) |
| OpenAPI 버전 | OpenAPI 3.0 필수 |
| 샌드박스 | 모든 API에 제공 권장 (사실상 필수) |
| MCP 서버 | 의회 API(공식), ONS·환경청 등 커뮤니티 구현 존재 |

---

## 2. 주요 포털 상세 분석

### 2.1 HMRC 개발자 허브 (세무청) — ★★★★★ 최우수

**URL**: https://developer.service.hmrc.gov.uk/api-documentation

HMRC는 영국에서 가장 성숙한 정부 API 포털이다. Making Tax Digital(MTD) 정책 추진으로 민간 세무소프트웨어와의 연동이 필수화되면서, 개발자 생태계 구축에 막대한 투자가 이루어졌다.

- **API 수**: 약 50개 (VAT MTD, 소득세 MTD, 법인세, CIS, 세관 등)
- **인증**: OAuth 2.0 완전 구현 (client_credentials + authorization_code + PKCE)
- **샌드박스**: 운영 환경과 동일 버전 별도 제공
- **문서**: OpenAPI 3.0 기반, 튜토리얼, 서비스 가이드, 엔드투엔드 가이드 완비
- **승인**: 샌드박스는 즉시, 운영 자격증명은 최대 10 영업일 (앱 심사)
- **사용자 중심성**: A등급 (12/14점)
- **AI 적합성**: A등급 (14/14점)

HMRC가 특히 탁월한 점은 **서버 토큰(client_credentials)** 과 **사용자 제한(authorization_code)** 패턴을 명확하게 문서화했다는 것이다. 어떤 엔드포인트에 어떤 인증이 필요한지 일관되게 안내한다.

---

### 2.2 NHS England API 카탈로그 — ★★★★★ 최우수 (임상 특화)

**URL**: https://digital.nhs.uk/developer/api-catalogue

93개의 API가 FHIR R4 표준을 중심으로 구축되어 있다. 개인정보서비스(PDS), 전자처방서비스(EPS), GP Connect 등 핵심 임상 데이터 API가 포함된다.

- **API 수**: 93개 (FHIR R4 기반 다수, 레거시 HL7 V3 일부)
- **인증 3중 체계**:
  1. **애플리케이션 제한**: 서버 간 API (환자 데이터 없음)
  2. **NHS Login**: 환자가 직접 접근하는 API
  3. **CIS2(Care Identity Service 2)**: 의료진 인증
- **온보딩**: 임상 API는 DTAC(디지털 기술 평가 기준) 통과 필수 → 진입 장벽 높음
- **주요 변경**: 2026년 3월 developer.nhs.uk 폐지, digital.nhs.uk로 통합
- **사용자 중심성**: A등급 (11/14점) — 승인 속도 0점(임상 심사 장기)
- **AI 적합성**: A등급 (14/14점)

---

### 2.3 기업등록청 API (Companies House) — ★★★★★ 개발자 친화적 최우수

**URL**: https://developer.company-information.service.gov.uk/

공개 기업 데이터에 대한 즉시 접근과 파일링 API의 OAuth 2.0 통합을 균형 있게 제공한다.

- **API 수**: 약 15개
- **인증 이중 구조**:
  - 공개 데이터 조회: API Key (Basic Auth) — GDS 예외 허용
  - 파일링/변경 API: OAuth 2.0 필수
- **샌드박스**: `api-sandbox.company-information.service.gov.uk` 전용 제공
- **OpenAPI**: 모든 API 스펙 완비
- **Python SDK**: `companies-house-api-client` (PyPI, 2024년 배포)
- **사용자 중심성**: A등급 (13/14점)
- **AI 적합성**: A등급 (13/14점)

GDS 표준이 API Key를 비권장하지만, Companies House는 공개 데이터의 경우 간편 접근을 위해 API Key를 허용하는 실용적 접근을 취한다.

---

### 2.4 국가지도원 데이터 허브 (OS Data Hub) — ★★★★☆

**URL**: https://osdatahub.os.uk/

- **API 수**: 8개 (Maps, Features, Vector Tiles, Linked Identifiers 등)
- **인증**: 프로젝트 기반 API Key — 지리공간 데이터 특성상 GDS 예외
- **공공기관**: Public Sector Plan으로 무료 접근
- **Python SDK**: `osdatahub` (공식 제공, PyPI)
- **데이터**: GeoJSON, Vector Tiles(MVT), WMS/WMTS 지원
- **사용자 중심성**: A등급 (12/14점)
- **AI 적합성**: A등급 (11/14점)

---

### 2.5 국가통계청 개발자 허브 (ONS) — ★★★★☆ 최고 접근성

**URL**: https://developer.ons.gov.uk/

**완전 무인증** 오픈 API로, 등록 없이 `api.beta.ons.gov.uk/v1` 직접 접근 가능하다. 인구·경제·사회 통계 데이터 약 30개 데이터셋 제공. 아직 Beta 상태로 브레이킹 체인지 가능성 있음.

- **사용자 중심성**: A등급 (11/14점)
- **AI 적합성**: A등급 (11/14점)
- **MCP 서버**: `uk-ons-mcp-server` 커뮤니티 구현 존재

---

### 2.6 런던교통공사 통합 API (TfL) — ★★★★☆

**URL**: https://api-portal.tfl.gov.uk/

런던 대중교통 실시간 데이터 제공. 미등록 상태에서도 분당 50회 무료 사용 가능하며, 등록 시 분당 500회로 확장.

- **API 수**: 약 20개 (버스, 지하철, 자전거, 도보 등)
- **인증**: `app_key` 쿼리 파라미터 — API Key 방식
- **Rate Limit**: 익명 50 req/min, 인증 500 req/min (명시적 제공)
- **사용자 중심성**: A등급 (13/14점)
- **AI 적합성**: A등급 (12/14점)

---

### 2.7 차량운전면허청 API (DVLA) — ★★★☆☆

**URL**: https://developer-portal.driver-vehicle-licensing.api.gov.uk/

- 공개 API는 API Key, 보안 API는 2단계 인증(API Key + JWT STS)
- 기관 온보딩 필요로 개인 개발자 접근 어려움
- .NET SDK 커뮤니티 구현 존재
- 2024-2025 전략: 레거시를 실시간 API로 전환 중
- **사용자 중심성**: C등급 (7/14점) — 접근성 저하
- **AI 적합성**: A등급 (11/14점) — 기술 품질은 양호

---

### 2.8 일자리연금부 API (DWP) — ★★☆☆☆ (기관 전용)

**URL**: https://www.api.gov.uk/dwp/

Kong API 게이트웨이 기반 정부간 API. 공개 개발자 포털 미제공. 약 2천만 복지 수급자 데이터를 처리하는 중요 인프라이나, 외부 접근이 사실상 불가.

- **사용자 중심성**: D등급 (5/14점)
- **AI 적합성**: A등급 (11/14점) — 내부 기술 품질 양호

---

### 2.9 토지등기소 API (HMLR) — ★★★☆☆

**URL**: https://landregistry.github.io/bg-dev-pack-redesign/

전문 비즈니스 파트너(변호사, 부동산 중개사) 대상 한정. 신규 RESTful API와 레거시 SOAP 병행 운영.

- **사용자 중심성**: C등급 (7/14점)
- **AI 적합성**: A등급 (12/14점)

---

### 2.10 환경청 홍수모니터링 API (EA) — ★★★☆☆ (오픈 데이터)

**URL**: https://environment.data.gov.uk/flood-monitoring/doc/reference

무인증 완전 공개. 15분 간격 실시간 수위/유량 데이터. OpenAPI 스펙 미제공이 아쉬움.

---

### 2.11 영국 의회 API — ★★★★☆ (AI 통합 선도)

**URL**: https://developer.parliament.uk/

무인증 공개 API. 의원/법안/투표/한사드 데이터 제공. **i.AI(정부 AI 기관)의 공식 MCP 서버**가 구현되어 AI 통합 면에서 영국 정부 API 중 가장 선진적이다.

- **사용자 중심성**: A등급 (13/14점)
- **AI 적합성**: A등급 (11/14점)

---

### 2.12 GOV.UK API 카탈로그 (인덱스) — ★★★☆☆

**URL**: https://www.api.gov.uk/

GDS/CDDO가 운영하는 범정부 API 인덱스. GitHub 기반 등록 방식(PR 제출). 200개+ API 수록. 단일 게이트웨이가 아닌 탐색 포털 역할.

---

## 3. GDS API 표준 핵심 내용

GDS API 기술·데이터 표준(2018년 초판, 2024년 7월 최종 업데이트)은 영국 정부 API의 근간이다. 이 표준은 **권고가 아닌 사실상의 의무**로 기능한다.

### 3.1 OAuth 2.0 의무화

GDS 표준은 API 접근 관리에 OAuth 2.0을 사용하도록 규정한다. 구체적으로:

- **client_credentials**: 서비스 간(B2B) API 호출 — 사용자 컨텍스트 없음
- **authorization_code + PKCE**: 사용자가 API에 접근할 때 (웹 앱 등)

JWT(JSON Web Token) 형식의 디지털 서명 토큰을 Authorization 헤더에 전달하는 방식으로, 탈취 후 재사용 공격에 강하다.

### 3.2 API Key 명시적 비권장

GDS 표준은 다음과 같이 명시한다:

> "API 키를 사용하지 말 것. API 키는 API 사용자에게 발급되는 고유 식별자로, URL, 요청 헤더, 쿠키에 포함되어 전송되는데, 공격자가 쉽게 가로채어 재사용할 수 있다."

이는 전 세계 정부 API 표준 중 API Key 사용을 가장 강경하게 금지하는 사례 중 하나다. 단, Companies House(공개 데이터), OS Data Hub(지리공간), TfL(교통 공개데이터) 같이 데이터 특성상 예외가 허용되는 경우도 있다.

### 3.3 샌드박스 환경 필수 권장

GDS 표준은 모든 API에 테스트 서비스(샌드박스)를 제공하도록 권장한다:

> "테스트 서비스(샌드박스) 제공을 시도해야 한다. 사용자들이 테스트 서비스 없이 API를 통합하기 어려울 수 있으므로, 제공하는 것은 프로젝트 시간과 예산의 좋은 사용이다."

HMRC와 Companies House는 운영 환경과 동일한 버전의 샌드박스를 별도 도메인으로 제공한다.

### 3.4 OpenAPI 3.0 필수

모든 REST API는 OpenAPI 3.0 스펙을 제공해야 한다. 이를 통해:
- 자동화된 클라이언트 SDK 생성
- 인터랙티브 문서(Swagger UI 등)
- API 검증 도구 활용

### 3.5 JSON 필수 / XML 사실상 폐지

GDS 표준은 JSON을 기본 데이터 형식으로 규정하며, JSON:API 규격을 권장한다. XML은 레거시 지원에만 허용된다. NHS의 HL7 V3(XML), Land Registry의 SOAP 서비스는 명시적으로 레거시로 분류되어 단계적 폐지 중이다.

### 3.6 기타 기술 표준

| 항목 | GDS 표준 요구사항 |
|------|-----------------|
| HTTPS | 모든 엔드포인트 필수 |
| Rate Limit | 헤더로 명시적 제공 |
| CORS | 클라이언트 측 접근 허용 |
| HTTP 상태 코드 | 적절한 의미론적 사용 필수 |
| 에러 응답 | 문제 세부 정보 표준(RFC 7807) 권장 |
| 버전 관리 | URL 경로 버저닝 또는 Accept 헤더 |

---

## 4. 인증 체계

### 4.1 OAuth 2.0 패턴 분류

영국 정부 API의 OAuth 2.0은 두 가지 핵심 패턴으로 운영된다:

#### 서버 토큰 패턴 (Server Token / Client Credentials)
- **대상**: 서비스 간 통신, 사용자 컨텍스트 불필요
- **사용 사례**: HMRC의 세무 소프트웨어 통합, DWP 부처간 데이터 공유
- **흐름**: `client_id + client_secret → access_token` (사용자 개입 없음)
- **토큰 형식**: JWT (Bearer Token)

#### 사용자 제한 패턴 (User-Restricted)
- **대상**: 사용자를 대신해 접근, 명시적 동의 필요
- **사용 사례**: HMRC 납세자 데이터, NHS 환자 정보, Companies House 파일링
- **흐름**: Authorization Code + PKCE → 사용자 동의 → access_token + refresh_token
- **특징**: 스코프(scope) 기반 세분화된 권한 제어

### 4.2 NHS 특수 인증 체계

NHS는 데이터 민감도에 따라 3단계 인증을 운용한다:

1. **애플리케이션 제한** (Application-Restricted): API Key 또는 서버 토큰 — 비개인 데이터
2. **NHS Login**: 환자 자가 인증 — 환자 포털/앱
3. **CIS2 (Care Identity Service 2)**: 의료진 인증 — 임상 시스템

### 4.3 DVLA 2단계 인증

보안 API는 API Key + JWT STS(보안 토큰 서비스) 2단계 인증을 요구한다. 운전자/차량 정보의 민감성을 반영한 강화된 보안 모델이다.

---

## 5. 개발자 경험 (DX)

### 5.1 developer.service.gov.uk 생태계

GDS는 `developer.service.gov.uk` 도메인 패턴으로 일관된 개발자 포털을 구성한다:
- `developer.service.hmrc.gov.uk` — HMRC
- `developer.company-information.service.gov.uk` — Companies House
- `developer-portal.driver-vehicle-licensing.api.gov.uk` — DVLA

이 패턴은 개발자가 정부 API 포털을 직관적으로 탐색하도록 돕는다.

### 5.2 인터랙티브 샌드박스

HMRC와 Companies House는 운영 환경과 동일한 버전의 샌드박스를 제공한다. 개발자는 실제 데이터 영향 없이 통합을 완성할 수 있다.

- HMRC Sandbox: `test-api.service.hmrc.gov.uk`
- Companies House Sandbox: `api-sandbox.company-information.service.gov.uk`

### 5.3 엔드투엔드 서비스 가이드

HMRC는 단순 API 문서를 넘어, 비즈니스 시나리오별 통합 가이드를 제공한다:
- Making Tax Digital for VAT 엔드투엔드 가이드
- Making Tax Digital for Income Tax 엔드투엔드 가이드
- 단계별 통합 튜토리얼

### 5.4 SDK 및 라이브러리

| 포털 | SDK |
|------|-----|
| Companies House | `companies-house-api-client` (Python, PyPI) |
| OS Data Hub | `osdatahub` (Python, 공식) |
| Parliament | `hansard` (R 패키지) |
| ONS | `onsr` (R 패키지) |
| DVLA | `DVLA.VEHICLE.ENQUIRY.SDK` (.NET, 커뮤니티) |
| TfL | `tfl` (Python, 커뮤니티) |

### 5.5 GitHub 기반 오픈소스 문화

영국 정부는 API 개발을 GitHub에서 공개로 진행하는 경우가 많다:
- `github.com/alphagov` — GDS 공식 저장소
- `github.com/hmrc` — HMRC 소스코드
- `github.com/NHSDigital` — NHS Digital
- `github.com/dwp` — DWP Digital
- `github.com/landregistry` — HM Land Registry

---

## 6. MCP/AI 도구 생태계

### 6.1 공식 정부 MCP 서버

#### 영국 의회 MCP (parliament-mcp) — 정부 최선진 사례
- **운영**: i.AI (정부 AI 기관, Incubator for AI)
- **저장소**: `github.com/i-dot-ai/parliament-mcp`
- **기능**: 13개 AI 도구 — 의원 검색, 선거 결과, 한사드 의미론적 검색
- **기술**: Azure OpenAI 임베딩 기반 의회 토론 벡터 검색
- **연동**: Microsoft Copilot 등 AI 에이전트와 연동 가능

이는 세계 최초 정부 기관이 자국 의회 데이터에 공식 MCP 서버를 구현한 사례 중 하나다.

#### Lex API (법률 데이터베이스)
- **운영**: i.AI
- **URL**: `lex.lab.i.ai.gov.uk`
- **특징**: 1267년 이후 법률 판결 파싱·인덱싱. Humphrey AI 툴킷의 일부

### 6.2 커뮤니티 MCP 서버

| 서버명 | 대상 API | 저장소 |
|--------|----------|--------|
| GovUK-MCP | 다수 GOV.UK 서비스 | github.com/Stealth-Labs-LTD/GovUK-MCP |
| UK ONS MCP Server | ONS 통계 API | github.com/dwain-barnes/uk-ons-mcp-server |
| parliament-mcp (커뮤니티) | 의회 API | github.com/joemclo/hansard-mcp |

**GovUK-MCP**는 단일 MCP 서버로 24개 도구를 제공한다:
- 우편번호, 식품위생, 은행공휴일, 홍수경보, 경찰범죄통계
- 법원, 자선단체, NHS 서비스, 법령, CQC, 의회
- Companies House, EPC, TfL (선택적 API Key 필요)

### 6.3 정부 AI 전략 (Humphrey AI 툴킷)

영국 정부는 'Humphrey' 공무원용 AI 툴킷을 구축 중이다:
- **Parlex**: 의회 API + MCP 기반 의회 감정 분석
- **Lex**: 법령·판례 파싱·인덱싱
- **Consult**: 공공 자문 분석 자동화
- **Redbox**: 장관 보고서 자동화

이는 정부 API를 AI 시대에 맞게 재활용하는 전략적 접근이다.

---

## 7. 사용자 중심성 평가

사용자 중심성은 7개 항목(각 0–2점, 총 14점)으로 평가했다.

| 포털 | 가입 | 승인 | 포맷 | 문서 | 한도 | 에러 | SDK | 총점 | 등급 |
|------|------|------|------|------|------|------|-----|------|------|
| Companies House | 2 | 2 | 2 | 2 | 1 | 2 | 2 | **13** | **A** |
| Parliament API | 2 | 2 | 2 | 2 | 2 | 1 | 2 | **13** | **A** |
| TfL API | 2 | 2 | 2 | 2 | 2 | 1 | 2 | **13** | **A** |
| OS Data Hub | 2 | 2 | 2 | 2 | 1 | 1 | 2 | **12** | **A** |
| HMRC 개발자 허브 | 1 | 1 | 2 | 2 | 2 | 2 | 2 | **12** | **A** |
| NHS API 카탈로그 | 1 | 0 | 2 | 2 | 2 | 2 | 2 | **11** | **A** |
| ONS 개발자 허브 | 2 | 2 | 2 | 1 | 2 | 1 | 1 | **11** | **A** |
| GOV.UK 카탈로그 | 2 | 2 | 2 | 1 | 1 | 1 | 1 | **10** | **B** |
| 환경청 API | 2 | 2 | 2 | 1 | 2 | 1 | 0 | **10** | **B** |
| DVLA 개발자 포털 | 0 | 0 | 2 | 2 | 1 | 1 | 1 | **7** | **C** |
| HMLR (토지등기소) | 0 | 0 | 2 | 2 | 1 | 1 | 1 | **7** | **C** |
| DWP API 게이트웨이 | 0 | 0 | 2 | 1 | 1 | 1 | 0 | **5** | **D** |

**종합 평균: 10.3/14 (B+)**

공개 API(ONS, 환경청, 의회)의 무인증 접근이 사용자 중심성을 크게 높인다. 반면 기관 전용(DWP, HMLR, DVLA)은 접근성에서 낮은 점수를 받았다.

---

## 8. AI 적합성 평가

AI 적합성은 기계 가독성, HTTP 의미론, JSON 네이티브, 에러 자기설명성, 프로그래매틱 인증, 레이트리밋 투명성, CORS/HTTPS의 7개 항목(각 0–2점, 총 14점)으로 평가했다.

| 포털 | 스키마 | HTTP | JSON | 에러 | 인증 | 한도 | CORS | 총점 | 등급 |
|------|--------|------|------|------|------|------|------|------|------|
| HMRC 개발자 허브 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | **A** |
| NHS API 카탈로그 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | **A** |
| Companies House | 2 | 2 | 2 | 2 | 2 | 1 | 2 | **13** | **A** |
| OS Data Hub | 2 | 2 | 2 | 1 | 1 | 1 | 2 | **11** | **A** |
| ONS 개발자 허브 | 1 | 2 | 2 | 1 | 2 | 1 | 2 | **11** | **A** |
| TfL API | 2 | 2 | 2 | 1 | 1 | 2 | 2 | **12** | **A** |
| Parliament API | 2 | 2 | 2 | 1 | 2 | 0 | 2 | **11** | **A** |
| DVLA 개발자 포털 | 2 | 2 | 2 | 1 | 1 | 1 | 2 | **11** | **A** |
| DWP API 게이트웨이 | 1 | 2 | 2 | 1 | 2 | 1 | 2 | **11** | **A** |
| HMLR (토지등기소) | 2 | 2 | 2 | 1 | 2 | 1 | 2 | **12** | **A** |
| GOV.UK 카탈로그 | 1 | 2 | 2 | 1 | 1 | 1 | 2 | **10** | **B** |
| 환경청 API | 0 | 2 | 2 | 1 | 2 | 0 | 2 | **9** | **B** |

**종합 평균: 11.6/14 (A-)**

GDS 표준의 강제력 덕분에 AI 적합성은 전반적으로 매우 높다. JSON 네이티브(2점)와 HTTPS(2점)는 모든 포털이 달성했다. 환경청만 OpenAPI 스펙 미제공, 레이트리밋 헤더 미제공으로 B등급에 머물렀다.

---

## 9. 한국과의 비교 시사점

### 9.1 영국의 강점 — 한국이 배워야 할 점

#### (1) GDS 표준의 강제성과 구체성

한국의 공공데이터 제공 및 이용 활성화에 관한 법률(공공데이터법)은 API 제공을 의무화하지만, 기술 표준의 구체성이 부족하다. 영국 GDS 표준은:
- **인증 방식 지정** (OAuth 2.0 필수, API Key 비권장)
- **데이터 포맷 지정** (JSON 필수, XML 폐지 경로)
- **샌드박스 제공 권장** (사실상 의무)
- **OpenAPI 3.0 필수**

이를 법적 기반과 결합하면, 각 부처가 독립적으로 운영하더라도 **일관된 개발자 경험**을 제공할 수 있다.

#### (2) OAuth 2.0 중심 인증 체계

한국 공공데이터포털(data.go.kr)의 `serviceKey` 쿼리 파라미터 방식은 GDS가 명시적으로 금지한 API Key 패턴이다. 영국 HMRC의 서버 토큰/사용자 제한 패턴을 한국 전자정부 표준프레임워크에 도입하면:
- AI 에이전트가 프로그래매틱하게 접근 가능
- 토큰 갱신 자동화
- 범위(scope) 기반 세분화된 권한 제어

#### (3) 샌드박스 환경의 보편화

한국 공공 API의 대다수는 샌드박스 없이 운영된다. 개발자가 테스트를 위해 실제 API를 호출해야 하는 구조는 일일 호출 한도를 빠르게 소진시킨다. HMRC·Companies House 모델처럼 별도 샌드박스 도메인 제공이 필요하다.

#### (4) MCP/AI 통합의 선제적 대응

영국 i.AI가 의회 API에 공식 MCP 서버를 구현한 반면, 한국은 아직 정부 기관의 공식 MCP 서버가 없다. `data.go.kr` 같은 중앙 포털에 MCP 엔드포인트를 공식 제공하면, AI 에이전트의 공공데이터 접근이 획기적으로 개선될 수 있다.

#### (5) 오픈소스 문화와 GitHub 투명성

영국 정부 부처들은 API 코드를 GitHub에 공개하고, 이슈트래커를 통해 개발자 커뮤니티와 소통한다. 한국 공공 API는 소스코드 비공개, 이슈 신고 채널 부재인 경우가 많다.

---

### 9.2 한국의 강점 — 영국이 배울 수 있는 점

#### (1) 중앙 게이트웨이의 편의성

영국에는 단일 게이트웨이가 없어, 개발자가 각 부처 포털에 별도 등록해야 한다. 한국 `data.go.kr`의 단일 serviceKey로 수천 개 API를 접근하는 방식은 **탐색·통합 편의성** 면에서 우월하다.

#### (2) API 수의 규모

data.go.kr의 1만 개+ API는 영국 카탈로그의 200개를 압도한다. 공공데이터 개방 범위 자체는 한국이 넓다.

---

### 9.3 단기 개선 우선순위 (한국)

1. **공공데이터법에 기술 표준 명시** — GDS 방식의 인증·포맷·샌드박스 표준 법제화
2. **serviceKey 방식에서 OAuth 2.0으로 단계적 전환** — 기존 API Key 병행 운영 후 폐지
3. **data.go.kr 공식 MCP 서버 구축** — AI 에이전트 접근 채널 공식화
4. **부처별 샌드박스 의무화** — 개발자 친화성 제고
5. **OpenAPI 3.0 스펙 100% 제공 의무화** — 현재 미제공 API 다수

---

## 참고 자료

- [GDS API 기술·데이터 표준](https://www.gov.uk/guidance/gds-api-technical-and-data-standards) (2024년 7월 최종 업데이트)
- [GOV.UK API 카탈로그](https://www.api.gov.uk/)
- [HMRC 개발자 허브](https://developer.service.hmrc.gov.uk/api-documentation)
- [NHS England API 카탈로그](https://digital.nhs.uk/developer/api-catalogue)
- [Companies House API](https://developer.company-information.service.gov.uk/)
- [OS Data Hub](https://osdatahub.os.uk/)
- [ONS 개발자 허브](https://developer.ons.gov.uk/)
- [TfL API 포털](https://api-portal.tfl.gov.uk/)
- [DVLA API 개발자 포털](https://developer-portal.driver-vehicle-licensing.api.gov.uk/)
- [영국 의회 MCP 서버](https://github.com/i-dot-ai/parliament-mcp)
- [GovUK-MCP](https://github.com/Stealth-Labs-LTD/GovUK-MCP)
- [UK ONS MCP 서버](https://github.com/dwain-barnes/uk-ons-mcp-server)
