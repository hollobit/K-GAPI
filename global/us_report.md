# 🇺🇸 미국 정부 API 생태계 분석

> 작성일: 2026-04-03
> 분석 대상: 미국 연방 정부 주요 API 포털 11개

---

## 1. 개요

### 규모 및 범위

| 항목 | 수치 |
|------|------|
| 분석 대상 주요 포털 수 | 11개 |
| 통합 게이트웨이(api.data.gov) 연계 기관 | 25개 이상 |
| 통합 게이트웨이 API 수 | 450개 이상 |
| Census Bureau 엔드포인트 수 | 1,600개 이상 |
| Data.gov 데이터셋 수 | 300,000개 이상 |
| 관할 기관 유형 | 연방 행정부 각 부처 (GSA, FDA, NASA, NOAA, SEC, Treasury, USGS, GPO 등) |

### 법적·제도적 기반

미국 연방 정부의 오픈 API 생태계는 다음 법령 및 정책에 의해 체계적으로 지원된다.

- **DATA Act (2014)**: 연방 지출 데이터의 표준화 및 공개 의무화 (USAspending.gov 근거)
- **OPEN Government Data Act (2019)**: 연방 데이터를 기본적으로 오픈 데이터로 공개
- **FISMA (Federal Information Security Management Act)**: 연방 IT 시스템 보안 프레임워크 (NIST SP 800-53 기반)
- **FedRAMP**: 클라우드 서비스 표준 보안 승인 프로그램
- **OMB M-13-13 (Open Data Policy)**: "기계 판독 가능, 개방형 형식, 라이선스 없음" 원칙
- **Digital Services Playbook**: 사용자 중심 디지털 서비스 설계 가이드

---

## 2. 주요 포털 상세 분석

### 2-1. api.data.gov — 연방 통합 API 게이트웨이

- **운영 기관**: General Services Administration (GSA) / 18F
- **URL**: https://api.data.gov
- **API 수**: 450개 이상 (25개 이상 기관)
- **기술 스택**: API Umbrella (오픈소스), nginx, Redis
- **인증**: 단일 API 키(api_key 파라미터 또는 X-Api-Key 헤더)
- **특징**:
  - `api.data.gov/AGENCY/ENDPOINT` 패턴의 통일된 URL 구조
  - 이메일 입력만으로 즉시 API 키 발급
  - 표준 rate limit 헤더(X-RateLimit-Limit, X-RateLimit-Remaining) 지원
  - 기관별 사용량 분석 및 정부 전체 통합 분석 제공
  - 오픈소스 공개로 다른 정부도 도입 가능

**사용자 중심성**: 14/14 (A등급) | **AI 적합성**: 14/14 (A등급)

---

### 2-2. Data.gov — 연방 오픈데이터 카탈로그

- **운영 기관**: GSA
- **URL**: https://data.gov / https://catalog.data.gov
- **데이터셋 수**: 300,000개 이상 (메타데이터 카탈로그)
- **기술 스택**: CKAN (오픈소스 데이터 플랫폼)
- **인증**: 불필요 (공개 CKAN API)
- **특징**:
  - 연방 데이터셋 메타데이터 통합 목록 (실제 데이터는 각 기관 API로 연결)
  - CKAN API를 통한 프로그래밍 방식 검색 지원
  - 월별 JSON Lines 벌크 다운로드 제공
  - 실제 데이터에 접근하려면 각 원본 기관 포털 이용 필요

**사용자 중심성**: 11/14 (B등급) | **AI 적합성**: 10/14 (B등급)

---

### 2-3. USAspending.gov API — 연방 재정지출 투명성

- **운영 기관**: Department of the Treasury / OMB
- **URL**: https://api.usaspending.gov
- **기술 스택**: Django (Python), PostgreSQL, 오픈소스
- **인증**: 불필요 (완전 공개)
- **특징**:
  - DATA Act(2014) 기반 법적 공개 의무
  - 연방 계약, 보조금, 대출, 직접지출 등 400개 이상 데이터 요소
  - GitHub 완전 오픈소스 공개
  - 인터랙티브 문서 및 훈련 자료 제공

**사용자 중심성**: 14/14 (A등급) | **AI 적합성**: 14/14 (A등급)

---

### 2-4. Census Bureau API — 인구통계 API

- **운영 기관**: U.S. Census Bureau (Department of Commerce)
- **URL**: https://api.census.gov
- **엔드포인트 수**: 1,600개 이상
- **인증**: 이메일 기반 API 키 즉시 발급 (없이도 일 500회 사용 가능)
- **특징**:
  - ACS(American Community Survey), 10년 인구조사, 경제조사 등 포괄
  - 응답 형식이 `[헤더 배열, 값 배열...]` 구조로 비표준적
  - OpenAPI 스펙 미제공이나 R(censusapi), Python 라이브러리 생태계 풍부
  - 2020-2024 ACS 5년 추정치 최신 데이터 제공

**사용자 중심성**: 10/14 (B등급) | **AI 적합성**: 9/14 (B등급)

---

### 2-5. openFDA — FDA 공개 데이터 API

- **운영 기관**: Food and Drug Administration (FDA)
- **URL**: https://open.fda.gov
- **API 수**: 약 15개 엔드포인트
- **기술 스택**: Elasticsearch 기반
- **인증**: 이메일 API 키 즉시 발급
- **Rate limit**:
  - 키 없음: 분당 240회, 일 1,000회
  - 키 있음: 분당 240회, 일 120,000회
- **특징**:
  - 약물 이상반응, 의료기기 리콜, 식품 안전 경보 등 포함
  - Elasticsearch 쿼리 문법으로 복잡한 검색 지원
  - 인터랙티브 문서, 쿼리 빌더 제공
  - 오픈소스 커뮤니티 활발

**사용자 중심성**: 14/14 (A등급) | **AI 적합성**: 14/14 (A등급)

---

### 2-6. NASA Open APIs — 우주항공 데이터 API

- **운영 기관**: NASA
- **URL**: https://api.nasa.gov
- **API 수**: 약 20개
- **인증**: 이메일 API 키 즉시 발급 / DEMO_KEY 즉시 체험
- **Rate limit**: 등록 사용자 시간당 1,000회
- **특징**:
  - APOD(오늘의 천문사진), NeoWs(근지구천체), EPIC(지구사진), Mars Rover Photos 등
  - DEMO_KEY 제공으로 즉시 테스트 가능 (시간당 30회/일 50회)
  - X-RateLimit-Limit, X-RateLimit-Remaining 헤더 표준 지원
  - 개발자 UX 모범 사례로 자주 인용됨

**사용자 중심성**: 14/14 (A등급) | **AI 적합성**: 14/14 (A등급)

---

### 2-7. api.weather.gov (NWS API) — 국립기상서비스 API

- **운영 기관**: National Weather Service / NOAA
- **URL**: https://api.weather.gov
- **인증**: 불필요 (User-Agent 헤더 설정 권장)
- **특징**:
  - 완전한 REST/JSON 기반 차세대 기상 데이터 서비스
  - OpenAPI v3.0 스펙 공개 (openapi.json / openapi.yaml)
  - GeoJSON, JSON-LD 네이티브 지원
  - DWML XML도 Accept 헤더를 통해 지원
  - GitHub를 통한 개발자 커뮤니티 운영

**사용자 중심성**: 12/14 (A등급) | **AI 적합성**: 12/14 (A등급)

---

### 2-8. SEC EDGAR API — 기업공시 데이터 API

- **운영 기관**: Securities and Exchange Commission (SEC)
- **URL**: https://data.sec.gov
- **인증**: 공개 API 불필요 (자동화 접근 시 User-Agent에 연락처 명시 필요)
- **특징**:
  - 10-K, 10-Q, 8-K 등 모든 기업 공시 데이터 JSON 제공
  - EDGAR 제출 API는 별도 filer/user 토큰 필요 (30일 유효)
  - OpenAPI 스펙 미제공
  - 제3자 sec-api.io가 DX 보완 서비스 제공

**사용자 중심성**: 9/14 (B등급) | **AI 적합성**: 9/14 (B등급)

---

### 2-9. USGS APIs — 지질·수자원 과학 API

- **운영 기관**: U.S. Geological Survey
- **URL**: https://www.usgs.gov/products/web-tools/apis
- **인증**: 대부분 불필요 (ScienceBase는 OAuth 2.0)
- **특징**:
  - 지진 API: FDSN 국제 표준 준수
  - 수자원 API: 2024년 OGC API Features 표준으로 현대화
  - GeoJSON 네이티브 지원
  - 다수 API가 국제 표준(FDSN, OGC) 준수

**사용자 중심성**: 10/14 (B등급) | **AI 적합성**: 12/14 (A등급)

---

### 2-10. GovInfo API — 연방 공식 문서 API

- **운영 기관**: Government Publishing Office (GPO)
- **URL**: https://api.govinfo.gov
- **인증**: api.data.gov 통합 키 (이메일 즉시 발급)
- **MCP 서버**: 2026년 1월 퍼블릭 프리뷰 런칭 (미국 최초 정부 MCP 서버)
- **특징**:
  - 연방관보(Federal Register), 법률, 의회문서, 연방법전(CFR) 등
  - LLM이 GovInfo 콘텐츠에 직접 접근 가능한 MCP 서버 제공
  - Claude, ChatGPT, Gemini, Copilot 호환

**사용자 중심성**: 13/14 (A등급) | **AI 적합성**: 14/14 (A등급)

---

### 2-11. U.S. Treasury Fiscal Data API — 재무부 재정 API

- **운영 기관**: Bureau of the Fiscal Service / Treasury
- **URL**: https://fiscaldata.treasury.gov
- **인증**: 불필요 (완전 공개)
- **특징**:
  - 연방 부채, 세입, 지출, 금리, 저축채권 데이터
  - RESTful GET, JSON 응답, v1/v2 버전 관리
  - 상업적 이용 제한 없음
  - R(ustfd), Python 라이브러리 존재

**사용자 중심성**: 14/14 (A등급) | **AI 적합성**: 13/14 (A등급)

---

## 3. 통합 API Gateway: api.data.gov 아키텍처

### 개요

api.data.gov는 GSA(일반서비스청)와 18F가 공동 개발한 연방 통합 API 관리 서비스로, 개별 기관이 반복적으로 구축해야 하는 API 관리 기능을 공유 인프라로 제공한다.

### 기술 아키텍처

```
[개발자]
    │
    ▼ api.data.gov/AGENCY/ENDPOINT
[api.data.gov 게이트웨이]
├── API 키 검증
├── Rate Limiting (기관별·키별 설정)
├── 사용량 분석 (익명화)
├── 프록시 (투명한 전달)
└── 응답 헤더 추가 (X-RateLimit-*)
    │
    ▼
[각 기관 원본 API 서버]
(NREL, Census, FDA, NASA 등)
```

### 핵심 기능

| 기능 | 설명 |
|------|------|
| 통합 API 키 | 단일 키로 25개 이상 기관 450개 이상 API 접근 |
| Rate Limiting | 기관별·엔드포인트별 세분화 설정 가능 |
| 사용량 분석 | 기관별 및 정부 전체 API 호출 통계 제공 |
| 투명 프록시 | 기관 기존 API 구조 변경 없이 적용 가능 |
| 오픈소스 | API Umbrella 기반, GitHub 공개 |

### URL 패턴

```
api.data.gov/nrel/alt-fuel-stations/v1.json    # NREL 대체연료 주유소
api.data.gov/census/data/2020/acs/acs5         # Census ACS 데이터
api.data.gov/regulations/v3/                    # 규정 데이터
```

### 기관 도입 프로세스

각 기관은 DNS 설정만으로 기존 API를 api.data.gov에 연결할 수 있으며, API 키 관리·Rate Limiting·분석 기능을 즉시 활용할 수 있다. 기관은 별도 인프라 없이 반복 작업을 제거할 수 있다.

---

## 4. API 디자인 표준: GSA/18F 가이드라인

### 18F API Standards (github.com/18F/api-standards)

2014년 발표 이후 연방 정부 API 설계의 실질적 표준으로 자리 잡은 이 가이드는 다음 핵심 원칙을 제시한다.

#### JSON 우선 원칙

> "JSON is an excellent, widely supported transport format, suitable for many web APIs. Supporting JSON and only JSON is a practical default for APIs."

- JSON만 지원하는 것을 기본값으로 권장
- 응답은 JSON 배열이 아닌 **JSON 객체**로 반환 (향후 확장성 및 메타데이터 포함 가능)
- JSONP 금지 (보안 및 성능 문제)

#### REST 설계 원칙

- URL은 동사 없이 리소스를 표현 (명사 기반)
- 쿼리 스트링 없이 URL과 HTTP 동사(GET/POST/PUT/DELETE)만으로 의도 파악 가능
- 복수형 명사 사용 (`/articles` 아님 `/article`)

#### HTTP 상태 코드

- 4XX: 클라이언트 오류
- 5XX: 서버 오류
- 오류 시에도 동일한 JSON 형식으로 상세 오류 메시지 반환

#### 버전 관리

- URL 경로에 버전 명시: `/v1/`, `/v2/`
- 구버전 최소 6개월 유지 보장

#### 페이지네이션

- 표준 쿼리 파라미터: `page`, `per_page`
- 응답에 `total`, `count`, `next`, `previous` 메타데이터 포함

### GSA API Standards (tech.gsa.gov/guides/api_standards/)

18F 표준을 확장한 GSA 내부 가이드로, 특히 다음을 강조한다.

- **보안**: HTTPS 필수, API 키 헤더 전송 (URL 파라미터보다 헤더 권장)
- **문서화**: Swagger/OpenAPI 스펙 제공 필수
- **CORS**: 브라우저 기반 접근 지원
- **Rate Limit 투명성**: 헤더를 통한 현재 한도·남은 호출 수 공개

---

## 5. 인증 체계

### 5-1. API 키 (api.data.gov 통합 시스템)

미국 연방 API의 주력 인증 방식. api.data.gov 통합 키를 통해 단일 등록으로 다수 기관 API 접근이 가능하다.

```
등록 방법: api.data.gov/signup/ 이메일 입력
발급 시간: 즉시 (수 분 내 이메일 전송)
전달 방법: api_key 쿼리 파라미터 또는 X-Api-Key 헤더
유효 기간: 무기한
비용: 무료
```

### 5-2. 인증 불필요 공개 API

상당수 연방 API가 인증 없이 공개 접근을 허용한다. 이는 투명성 및 접근성 극대화 원칙에 따른 것이다.

| 포털 | 인증 없는 접근 | 비고 |
|------|---------------|------|
| api.weather.gov | 완전 공개 | User-Agent 권장 |
| data.sec.gov | 완전 공개 | User-Agent 필수 |
| api.usaspending.gov | 완전 공개 | 무제한 |
| fiscaldata.treasury.gov | 완전 공개 | 상업 이용 허용 |
| api.usgs.gov (지진) | 완전 공개 | FDSN 표준 |

### 5-3. OAuth 2.0 / OpenID Connect

민감한 사용자 데이터나 쓰기 작업이 필요한 경우 OAuth 2.0 사용.

- **login.gov**: 연방 정부 통합 ID 플랫폼 (FedRAMP Moderate 승인)
  - OpenID Connect 1.0 + iGov Profile 준수
  - IAL1 (이메일+MFA) / IAL2 (신분증 사진 인증) 두 단계
  - 100개 이상 연방 기관 애플리케이션에서 사용
  - SAML 2.0도 지원

- **USGS ScienceBase**: OAuth 2.0 (쓰기 작업 시)
- **EDGAR 제출 API**: filer API 토큰 (30일 유효, 별도 발급)

### 5-4. 보안 기준

- FISMA: 연방 IT 시스템 보안 법적 의무 (NIST SP 800-53 기반)
- FedRAMP: 클라우드 서비스 제3자 보안 심사 (3PAO 평가)
- HTTPS: 전체 API 필수 (HTTP 리다이렉트 또는 차단)

---

## 6. 개발자 경험 (DX) 분석

### 6-1. 온보딩 프로세스

미국 연방 API의 가장 큰 강점 중 하나는 온보딩의 단순함이다.

```
[일반적인 미국 연방 API 온보딩]
1. api.data.gov/signup/ 접속
2. 이름·이메일·용도 입력 (30초)
3. 이메일 확인 후 API 키 수령 (수 분)
4. 즉시 API 호출 시작

[한국 공공 API 온보딩 (예: 공공데이터포털)]
1. 데이터포털 회원가입 (휴대폰 인증)
2. 원하는 API 신청
3. 승인 대기 (자동: 즉시~익일 / 수동: 3~14일)
4. URL 인코딩 이슈 해결
5. XML→JSON 변환 처리
```

### 6-2. 문서화 수준

| 항목 | 미국 연방 API | 한국 공공 API |
|------|-------------|-------------|
| OpenAPI 스펙 | 대부분 제공 | 거의 없음 |
| 인터랙티브 문서 | 주요 포털 제공 | 드물음 |
| 코드 예제 | 다수 포털 다언어 제공 | 제한적 |
| 쿼리 빌더 | openFDA, Data.gov 등 | 없음 |
| 오류 메시지 | JSON + 상세 설명 | 200 OK + 오류 코드 (비표준) |

### 6-3. Rate Limit 정책

| 포털 | 기본 한도 | 증가 가능 |
|------|---------|---------|
| api.data.gov | 기관별 설정 | 문의 가능 |
| openFDA | 키 포함 일 120,000회 | 문의 가능 |
| NASA API | 시간당 1,000회 | 문의 가능 |
| api.weather.gov | 무제한(합리적 사용) | N/A |
| USAspending | 제한 없음 | N/A |

### 6-4. SDK 및 라이브러리 생태계

공식 SDK보다 커뮤니티 기반 라이브러리가 활성화되어 있다.

- **R**: censusapi, tidycensus, rnoaa, ustfd 등
- **Python**: censusdata, openFDA, usgs 등
- **JavaScript**: 다수의 npm 패키지
- **Postman**: NASA, NOAA 등 공식 컬렉션 제공

---

## 7. MCP/AI 도구 생태계

### 7-1. GovInfo MCP 서버 (미국 정부 최초)

미국 정부 출판국(GPO)이 2026년 1월 공식 런칭한 GovInfo MCP 서버는 **미국 연방 정부 최초의 Model Context Protocol 구현**이다.

```
개발 기관: U.S. Government Publishing Office (GPO)
런칭 시기: 2026년 1월 (퍼블릭 프리뷰)
지원 LLM: Claude, ChatGPT, Gemini, Microsoft Copilot
데이터: 연방관보, 연방법전, 의회문서, 법령, 규정집
```

**MCP 작동 방식**:
1. 사용자가 LLM에 연방 문서 관련 질문
2. LLM이 GovInfo MCP 도구를 호출
3. GovInfo API에서 최신 공식 문서 검색
4. LLM이 실시간 데이터 기반 응답 생성

이 구현은 Anthropic이 2024년 개발하고 Linux Foundation 산하 Agentic AI Foundation에 기증한 MCP 표준을 활용한다.

### 7-2. 기타 AI 연계 동향

- **CMS(Centers for Medicare & Medicaid Services)**: 2025년 AI 활용 인벤토리에 MCP 배포 계획 포함
- **Treasury Department**: 2025년 AI 활용 인벤토리에 MCP 배포 계획 포함
- **DiscoverGov**: GPO가 2024년 12월 런칭한 AI 기반 정부 문서 검색 도구
- **openFDA 커뮤니티**: LLM 기반 FDA 데이터 탐색 오픈소스 프로젝트 다수 존재

### 7-3. AI 적합성 평가 요약

미국 연방 API는 구조적으로 AI 에이전트 통합에 매우 유리한 특성을 보인다.

- JSON 네이티브 응답 (한국 XML 중심과 대조)
- OpenAPI 스펙 광범위 제공 (LLM 함수 호출 스키마 자동 생성 가능)
- 표준 HTTP 상태 코드 사용 (오류 판별 자동화 가능)
- CORS 지원 (브라우저 기반 에이전트 작동 가능)
- 프로그래밍 방식 인증 (API 키 헤더)

---

## 8. 사용자 중심성 평가 (7항목)

### 평가 기준 (0~2점 척도)

| 항목 | 평가 기준 |
|------|---------|
| signup_ease | 가입 절차의 간단함 |
| approval_speed | API 접근 승인 속도 |
| data_format | 데이터 형식의 편의성 |
| doc_quality | 문서화 수준 |
| rate_limit | 기본 호출 한도 충분성 |
| error_handling | 오류 메시지 명확성 |
| sdk_samples | SDK/코드 샘플 제공 |

### 포털별 사용자 중심성 점수

| 포털 | 가입 용이 | 승인 속도 | 데이터 형식 | 문서 품질 | Rate Limit | 오류 처리 | SDK/샘플 | 합계 | 등급 |
|------|---------|---------|-----------|---------|-----------|---------|---------|------|-----|
| api.data.gov | 2 | 2 | 2 | 2 | 2 | 2 | 1 | **13** | A |
| USAspending | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| openFDA | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| NASA API | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| api.weather.gov | 2 | 2 | 2 | 2 | 2 | 1 | 1 | **12** | A |
| GovInfo API | 2 | 2 | 2 | 2 | 2 | 2 | 1 | **13** | A |
| Treasury Fiscal | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| Data.gov | 2 | 2 | 2 | 1 | 2 | 1 | 1 | **11** | B |
| Census Bureau | 2 | 2 | 1 | 1 | 1 | 1 | 2 | **10** | B |
| USGS APIs | 2 | 2 | 1 | 1 | 2 | 1 | 1 | **10** | B |
| SEC EDGAR | 2 | 2 | 1 | 1 | 1 | 1 | 1 | **9** | B |

**전체 평균: 12.2 / 14 (A등급)**

### 주목할 사항

- **A등급 포털**: 11개 중 7개 (64%)
- **즉시 승인**: 전체 포털 100% (한국은 수동 승인이 일반적)
- **인증 불필요**: 11개 중 5개 (45%)가 API 키 자체 불필요
- **최저 점수**: SEC EDGAR (9/14) — 문서화 부족과 비표준 응답 형식

---

## 9. AI 적합성 평가 (7항목)

### 평가 기준 (0~2점 척도)

| 항목 | 평가 기준 |
|------|---------|
| machine_schema | OpenAPI/JSON Schema 기계 판독 가능 스펙 제공 |
| http_semantics | 올바른 HTTP 상태 코드 사용 |
| json_native | JSON 기본 응답 형식 |
| error_self_desc | 자기 설명적 오류 메시지 |
| programmatic_auth | 프로그래밍 방식 인증 (헤더 API 키 등) |
| rate_limit_transparency | Rate Limit 정보의 투명한 헤더 제공 |
| cors_https | CORS 지원 및 HTTPS 필수 |

### 포털별 AI 적합성 점수

| 포털 | 기계 스키마 | HTTP 의미론 | JSON 네이티브 | 자기설명 오류 | 프로그래밍 인증 | Rate Limit 투명 | CORS/HTTPS | 합계 | 등급 |
|------|-----------|-----------|-------------|------------|--------------|----------------|-----------|------|-----|
| api.data.gov | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| USAspending | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| openFDA | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| NASA API | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| GovInfo API | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| api.weather.gov | 2 | 2 | 2 | 1 | 2 | 1 | 2 | **12** | A |
| USGS APIs | 2 | 2 | 2 | 1 | 2 | 1 | 2 | **12** | A |
| Treasury Fiscal | 2 | 2 | 2 | 2 | 2 | 1 | 2 | **13** | A |
| Data.gov | 1 | 1 | 2 | 1 | 2 | 1 | 2 | **10** | B |
| Census Bureau | 1 | 1 | 2 | 1 | 2 | 0 | 2 | **9** | B |
| SEC EDGAR | 1 | 1 | 2 | 1 | 2 | 0 | 2 | **9** | B |

**전체 평균: 12.3 / 14 (A등급)**

### AI 적합성 핵심 분석

1. **JSON 네이티브**: 전체 포털 100% JSON 기본 지원 (한국: XML 기본이 다수)
2. **OpenAPI 스펙**: A등급 포털 대부분 제공 — LLM 함수 호출 스키마 자동 생성 가능
3. **Rate Limit 헤더**: api.data.gov, openFDA, NASA, GovInfo 등 주요 포털 표준 지원
4. **취약점**: Census Bureau, SEC EDGAR — 비표준 응답 구조, OpenAPI 스펙 부재

---

## 10. 한국과의 비교 시사점

### 10-1. 종합 비교표

| 비교 항목 | 미국 | 한국 |
|---------|------|------|
| 통합 API 게이트웨이 | api.data.gov (25개 기관, 450+ API) | data.go.kr (부분적 게이트웨이) |
| 기본 데이터 형식 | JSON (100%) | XML 기본, JSON 선택적 |
| API 키 발급 속도 | 즉시 (이메일 입력 후 수 분) | 자동(즉시~익일) / 수동(3~14일) |
| 인증 없는 공개 API | 45% 포털이 인증 불필요 | 거의 없음 (대부분 키 필수) |
| OpenAPI 스펙 제공 | 주요 포털 대부분 제공 | 사실상 없음 |
| HTTP 상태 코드 | 표준 준수 | 200 OK + 내부 오류 코드 혼용 |
| Rate Limit 헤더 | 주요 포털 표준 지원 | 없음 |
| CORS 지원 | 전체 포털 지원 | 대부분 미지원 |
| MCP 서버 | GovInfo (2026년 1월, 공식) | 비공식 커뮤니티 프로젝트 |
| API 디자인 표준 | 18F/GSA 가이드라인 (공식) | 없음 (비표준) |
| 사용자 중심성 평균 | 12.2 / 14 (A등급) | ~7 / 14 (C등급 추정) |
| AI 적합성 평균 | 12.3 / 14 (A등급) | ~4-6 / 14 (D등급 추정) |

### 10-2. 미국이 앞서는 영역

#### 제도적 설계의 우위

미국의 가장 큰 강점은 **기술보다 제도**에 있다. DATA Act, OPEN Government Data Act 등 법령이 API 공개를 의무화하고, OMB 정책이 "기계 판독 가능, 오픈 형식"을 강제한다. 한국은 개별 기관의 자율에 의존하는 경향이 강하다.

#### 통합 인프라의 효율성

api.data.gov 하나로 25개 기관의 API 키 관리, Rate Limiting, 분석이 통합된다. 한국은 data.go.kr이 허브 역할을 하지만 serviceKey 인코딩 이슈, 실제 게이트웨이 기능 미흡 등 기술적 결함이 존재한다.

#### 설계 표준의 일관성

18F API Standards와 GSA API Standards가 "JSON only", "표준 HTTP 상태 코드", "OpenAPI 스펙 필수" 등을 명시하여 기관 간 일관성을 높인다. 한국은 기관마다 독자적 형식을 사용한다.

#### 개발자 중심 문화

NASA의 DEMO_KEY, openFDA의 인터랙티브 쿼리 빌더, USAspending의 완전 오픈소스 등은 개발자 경험을 최우선으로 설계한 결과다. 한국은 공급자(정부) 중심 설계가 지배적이다.

### 10-3. 한국이 앞서는 영역

#### API 수량

한국 data.go.kr의 약 11,000개 API는 미국 api.data.gov의 450개보다 양적으로 훨씬 많다. 미국은 품질·표준화에 집중하고, 한국은 수량에서 우위를 보인다.

#### 행정 서비스 밀도

한국은 주민등록, 부동산, 건강보험 등 행정 서비스 API의 밀도가 높다. 미국은 이러한 개인 행정 서비스 API가 상대적으로 제한적이다 (privacy 규제 및 연방제 특성).

### 10-4. 한국 공공 API 개선을 위한 시사점

#### 단기 개선 (6개월 이내)

1. **JSON을 기본 응답 형식으로 전환**: 18F 원칙처럼 "JSON only" 정책 채택
2. **표준 HTTP 상태 코드 사용**: 200 OK + 내부 오류 코드 관행 폐지
3. **Rate Limit 헤더 추가**: `X-RateLimit-Limit`, `X-RateLimit-Remaining` 표준화
4. **CORS 허용**: 브라우저 기반 개발자 도구 및 AI 에이전트 지원

#### 중기 개선 (1~2년)

5. **OpenAPI 스펙 의무화**: 모든 신규 공공 API에 OpenAPI 3.0 스펙 제공 의무화
6. **통합 API 키 시스템 구축**: api.data.gov처럼 단일 키로 다수 기관 API 접근
7. **API 디자인 가이드 공식화**: 행정안전부 주도의 공식 API 디자인 표준 수립

#### 장기 개선 (2~3년)

8. **MCP 서버 생태계 구축**: 공공데이터포털 공식 MCP 서버 개발 (GovInfo 모델 참조)
9. **API 품질 평가 제도**: 사용자 중심성·AI 적합성 기반 API 품질 등급제 도입
10. **법적 공개 의무 강화**: 핵심 공공 데이터의 API 공개를 법령으로 의무화 (DATA Act 모델)

### 10-5. 결론

미국 연방 API 생태계는 **"소수의 고품질 표준화 API"** 모델의 성공적 사례다. 한국은 양적으로 앞서지만 표준화·사용자 경험·AI 적합성에서 현저히 뒤처진다. 미국 모델에서 배울 핵심은 기술이 아니라 **제도적 의지와 표준화 거버넌스**다. GSA의 18F 같은 디지털 혁신 조직이 공식 API 표준을 수립하고 이를 강제할 수 있는 법적·예산적 권한을 가질 때 비로소 생태계 수준의 개선이 가능하다.

---

*참고 자료*
- api.data.gov 공식 문서: https://api.data.gov/about/
- 18F API Standards: https://github.com/18F/api-standards
- GSA API Standards: https://tech.gsa.gov/guides/api_standards/
- GovInfo MCP 서버: https://www.govinfo.gov/features/mcp-public-preview
- openFDA 인증 문서: https://open.fda.gov/apis/authentication/
- NASA Open APIs: https://api.nasa.gov/
- USAspending API: https://api.usaspending.gov/
- NOAA NWS API: https://weather-gov.github.io/api/
- Census Bureau API: https://www.census.gov/data/developers.html
- SEC EDGAR API: https://www.sec.gov/about/developer-resources
- U.S. Treasury Fiscal Data: https://fiscaldata.treasury.gov/api-documentation/
- USGS APIs: https://www.usgs.gov/products/web-tools/apis
