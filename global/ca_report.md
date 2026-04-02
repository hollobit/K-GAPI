# 🇨🇦 캐나다 정부 API 생태계 분석

> 작성일: 2026-04-03 | 조사 기준: 2025년 말 ~ 2026년 초 공개 자료

---

## 1. 개요

캐나다 연방 정부의 API 생태계는 **CKAN 기반 오픈데이터 포털**, **통계청(StatCan) 전문 API**, **캐나다 기상청(MSC) OGC 표준 API**, 그리고 **캐나다 디지털 서비스(CDS)의 현대화 플랫폼**을 축으로 구성되어 있다.

캐나다의 가장 뚜렷한 특징은 **이중언어(영어·프랑스어) 의무 운영**이다. 캐나다 헌법 및 공용어법(Official Languages Act)에 따라 모든 연방 정부 디지털 서비스는 영어와 프랑스어를 동등하게 지원해야 하며, 이 원칙이 API 설계·문서화·응답 형식에 직접 반영된다.

**재무위원회 사무국(Treasury Board Secretariat, TBS)**이 API 표준을 주도하며, 2018년 12월부터 모든 연방 부처에 REST/JSON 우선, OpenAPI 명세 공개, 이중언어 엔드포인트를 의무화하는 지침을 시행 중이다. 그러나 GC API Store가 2023년 9월 종료된 이후 중앙 집중식 API 카탈로그가 부재한 상황이며, 이를 대체할 통합 체계는 아직 정비 중이다.

### 주요 수치 (2025년 기준)

| 지표 | 수치 |
|------|------|
| 오픈데이터 포털 데이터셋 수 | 100,000개 이상 |
| 통계청 WDS API 메서드 수 | 15개 |
| MSC GeoMet 데이터셋 수 | 수천 개 (실시간·아카이브) |
| GC Notify 누적 발송 건수 | 2억 7,700만 건 이상 |
| GC Notify 사용 기관 수 | 895개 프로그램/서비스 |
| GC Forms 발행 폼 수 | 1,870개 (2024 대비 +145%) |

---

## 2. 주요 포털 상세 분석

### 2.1 Open Government Portal (open.canada.ca)

- **운영 기관**: 재무위원회 사무국(TBS)
- **플랫폼**: CKAN (ckanext-canada 확장 적용)
- **API 방식**: CKAN Action API (RPC 스타일, GET 전용)
- **인증**: 불필요 (읽기 전용 공개 접근)
- **특이사항**: 2024년 11월 OpenAPI Specification(OAS3) 최초 공개

캐나다 연방 오픈데이터의 중앙 허브. 연방 부처 데이터와 더불어 주(Province)·지방자치단체 데이터도 일부 연합(federated) 방식으로 포함. ckanext-canada 확장은 GitHub에 오픈소스로 공개되어 있으며, 영불 이중언어 메타데이터를 포털 수준에서 구현한다.

**제약사항**: GET 요청만 지원(쓰기 불가), 레이트 리밋 헤더 미제공, OpenAPI 스펙이 최근에야 제공되기 시작.

### 2.2 Statistics Canada Web Data Service (WDS)

- **운영 기관**: 통계청(StatCan)
- **URL**: `https://www150.statcan.gc.ca/t1/wds/rest/`
- **인증**: 불필요
- **형식**: JSON 및 SDMX-XML 2.1

통계청이 매 평일 오전 8:30(EST) 업데이트하는 집계 통계 데이터를 실시간으로 제공. 15개 REST 메서드를 통해 데이터셋 조회, 시계열 변경 목록, CSV 전체 다운로드 등을 지원. SDMX(Statistical Data and Metadata Exchange) 국제 표준 지원이 특징으로, 국제 통계 시스템과의 상호운용성이 높다.

GC API Store에 등록된 대표 API 중 하나였으며, Python 라이브러리 `stats_can`과 SDMX 클라이언트를 통한 프로그래밍 접근이 가능하다. 별도로 **LODE(Linkable Open Data Environment)**를 통해 오픈소스 도구와 데이터를 연계 제공한다.

**제약사항**: OpenAPI 스펙 미제공, 레이트 리밋 정책 불명확.

### 2.3 MSC GeoMet OGC API (api.weather.gc.ca)

- **운영 기관**: 기상청(MSC) / 환경기후변화부(ECCC)
- **인증**: 불필요 (완전 익명 무료)
- **지원 표준**: OGC WMS, WFS, WCS, OGC API Features, OGC API Processes

캐나다 정부 API 중 **기술 수준이 가장 높은 포털**. OGC(Open Geospatial Consortium) 국제 표준을 선도적으로 채택한 얼리어답터로, OGC API Features·Processes를 이미 운영 중이며 OGC API Records, Coverages, Maps, Tiles, Environmental Data Retrieval 등도 표준 확정 후 순차 도입 예정.

Swagger UI 기반 인터랙티브 문서 제공, GeoJSON/KML/BUFR 등 다양한 형식 지원, 실시간 기상·수문·기후 수천 개 레이어를 무료로 접근 가능. MSC AniMet(기상 애니메이션 생성 도구) 등 오픈소스 개발 도구도 병행 제공.

**평가**: 개발자 경험(DX) 측면에서 캐나다 정부 API 중 최상위 수준.

### 2.4 Health Canada Drug Product Database (DPD) API

- **URL**: `https://health-products.canada.ca/api/drug/`
- **데이터**: 약 47,000개 의약품 (승인·판매·휴면·취소)
- **인증**: 불필요

인체 의약품, 수의약품, 방사성의약품, 소독제 정보를 JSON 및 XML로 제공. GC API Store에 등록된 Health Canada 대표 API였으나 문서화 수준이 낮고 OpenAPI 스펙이 없어 개발자 친화도가 낮다.

별도로 **Health Infobase API** (`health-infobase.canada.ca/api`)가 공중보건 지표 데이터를 제공하며 4개 라우트(전체 테이블, 단일 컬럼, 커스텀 셀렉트, 메타 조회)를 지원한다.

---

## 3. CKAN 기반 포털 아키텍처

캐나다 오픈데이터 포털은 **CKAN(Comprehensive Knowledge Archive Network)**을 기반으로 구축되었다. CKAN은 Python/PostgreSQL 기반의 오픈소스 데이터 관리 시스템으로, 영국, 미국(Data.gov), 호주 등 주요 국가에서도 채택한 사실상의 정부 오픈데이터 표준 플랫폼이다.

### 캐나다의 CKAN 커스터마이징

캐나다 정부는 `ckanext-canada`라는 자체 CKAN 확장을 개발하여 GitHub에 오픈소스로 공개했다. 주요 커스터마이징 내용:

- **이중언어 메타데이터**: 모든 데이터셋 제목, 설명, 태그를 영어(en)·프랑스어(fr) 병렬 저장
- **연방 부처 통합**: 수십 개 연방 부처의 데이터를 단일 카탈로그로 집약
- **Registry 분리**: 공개 포털(open.canada.ca)과 부처 내부 등록 시스템(Registry, GC 네트워크 내부)을 이원화
- **OpenAPI 명세 발행**: 2024년 11월 OAS3 형식의 포털 API 명세 최초 공개

### CKAN Action API 구조

```
GET https://open.canada.ca/data/api/3/action/{action}
```

주요 액션:
- `package_list` — 전체 데이터셋 목록
- `package_show?id={id}` — 특정 데이터셋 상세
- `resource_search` — 리소스 검색
- `datastore_search` — 구조화 데이터 직접 조회

**제약**: GET 전용. POST·PUT·DELETE 미지원. 레이트 리밋 명시 없음.

---

## 4. 이중언어 (EN/FR) API 운영 모델

캐나다의 이중언어 API 운영은 단순한 문서 번역을 넘어 **API 설계 원칙 자체에 내재화**되어 있다.

### TBS API 표준의 이중언어 요구사항

재무위원회 사무국(TBS)이 2018년 발행하고 이후 갱신한 "Government of Canada Standards on APIs"는 다음을 명시한다:

1. **BCP-47 언어 코드 키 사용**: 영불 콘텐츠는 `"en"`·`"fr"`를 키로 중첩(nested) 구조로 반환
2. **Accept-Language 헤더 지원**: 클라이언트가 선호 언어를 지정하면 해당 언어 우선 응답
3. **기본값은 다국어(multilingual)**: `Accept-Language` 미지정 시 영불 병렬 콘텐츠 응답
4. **UTF-8 인코딩 필수**: 프랑스어 특수문자(é, à, ç 등) 완전 지원
5. **ISO 8601 UTC 날짜 형식**: 시간대 표기 표준화

### 실제 응답 예시 패턴

```json
{
  "title": {
    "en": "Population Census 2021",
    "fr": "Recensement de la population 2021"
  },
  "description": {
    "en": "...",
    "fr": "..."
  },
  "language": ["en", "fr"]
}
```

### 이중언어 운영의 현실적 과제

- **레거시 API 비준수**: 구형 API 다수가 영어 전용으로 운영 중
- **번역 지연**: 새 데이터셋 메타데이터의 프랑스어 번역이 영어보다 늦게 제공되는 사례 존재
- **비용 증가**: 이중언어 문서 유지에 따른 관리 부담이 API 공개 속도를 늦추는 요인

---

## 5. GC API Store & Canadian Digital Service

### 5.1 GC API Store (2023년 9월 종료)

**GC API Store** (`api.canada.ca`)는 연방 정부 API를 한 곳에서 탐색·구독할 수 있는 API 카탈로그였다. WSO2 API Manager 기반으로 운영되었으며, 부처별 서브도메인 구조를 사용했다:

- `statcan.api.canada.ca` — 통계청 WDS
- `hc-sc.api.canada.ca` — 보건부 DPD
- `cra-arc.api.canada.ca` — 국세청 GST/HST
- `esdc-edsc.api.canada.ca` — 고용사회개발부 LMI
- `demo.dev.api.canada.ca` — 시험용 데모

**2023년 9월 29일 12:00 EDT에 공식 종료**. 이후 사용자는 각 API 소유 부처에 직접 문의하도록 안내. 현재 대체 중앙 카탈로그는 미확정 상태이며, 이는 캐나다 연방 API 생태계의 가장 큰 공백이다.

### 5.2 Canadian Digital Service (CDS)

캐나다 디지털 서비스는 영국 GDS(Government Digital Service)를 모델로 2017년 설립된 정부 내 디지털 혁신 조직이다. TBS 산하에서 재사용 가능한 디지털 플랫폼을 개발하며, 2025년 기준 주요 플랫폼:

| 플랫폼 | 기능 | 사용 현황 |
|--------|------|----------|
| **GC Notify** | 이메일·SMS 알림 발송 API | 895개 서비스, 2.77억 건 발송 |
| **GC Forms** | 온라인 폼 생성·배포 | 71개 부처, 1,870개 폼 |
| **GC Design System** | 정부 UI 컴포넌트 라이브러리 | WET(Web Experience Toolkit) 후계 |

**GC Notify**는 영국 GOV.UK Notify를 포크하여 캐나다 정부 환경에 맞게 재구성. Python Flask 기반 오픈소스로 GitHub 공개. REST API를 통해 프로그래밍 방식으로 알림 발송 가능. 이메일 연간 2,000만 건, SMS 10만 건 한도(2025년 상향 조정).

**CDS 전략 방향 (2025-2027)**: Protected B 수준 데이터 처리 확장, 인터운영성(interoperability) 개선, API 노출 기능 확대, 아랍어·히브리어 등 RTL 언어 지원.

---

## 6. 인증 체계

캐나다 연방 정부 API의 인증은 접근 대상에 따라 크게 세 계층으로 구분된다.

### 6.1 공개 API (인증 불필요)

| API | 인증 |
|-----|------|
| Open Government Portal CKAN API | 없음 |
| StatCan WDS | 없음 |
| MSC GeoMet | 없음 |
| Health Canada DPD | 없음 |
| NRCan GeoGratis | 없음 |

대다수 데이터 API는 완전 공개로 운영. 레이트 리밋이 명시적으로 선언되지 않는 경우가 많아 대규모 자동화 접근 시 비공식적 제한이 적용될 수 있다.

### 6.2 API 키 기반 (기관 계정)

**GC Notify**: `Authorization: ApiKey-v1 {KEY}` 헤더. 정부 기관 계정 등록 후 서비스별 키 발급. 민간 개발자는 접근 불가.

**GC Forms**: Protected B 보안 수준 API 키. 동일하게 기관 계정 전용.

**레거시 GC API Store 등록 API (CRA, ESDC 등)**: GC API Store 종료 이후 키 발급 경로 불명확. 부처별 직접 협의 필요.

### 6.3 GCKey 및 OAuth 2.0 (시민 서비스)

**GCKey**는 캐나다 정부 온라인 서비스 접근을 위한 연방 인증 자격증명 서비스(2012년 출시). My Service Canada Account, CRA My Account, IRCC Online 등 시민 대상 서비스에서 사용. 2023년부터 MFA(다단계 인증) 의무화.

개발자 API 관점에서는:
- **OAuth 2.0**: 신규 RESTful API의 권장 표준 (TBS API 표준 명시)
- **OpenID Connect**: OAuth 2.0 위에서 신원 확인 레이어 추가
- **TLS 1.2 이상**: 모든 API 통신에 필수

TBS API 보안 가이드라인은 OWASP API Security Top 10 기반으로 구성되며, API 키와 OAuth 2.0 병행을 권장한다.

---

## 7. 개발자 경험

### 7.1 강점

**MSC GeoMet**은 캐나다 정부 API 중 개발자 경험이 가장 우수하다:
- Swagger UI 인터랙티브 문서
- 인증 불필요한 완전 공개 접근
- 국제 OGC 표준 준수로 기존 GIS 도구와 바로 연동 가능
- GitHub 오픈소스 도구(MSC AniMet 등) 제공

**GC Notify**는 정부 내부 개발자 대상으로 최고 수준:
- RESTful API + OpenAPI 스펙
- Sandbox 환경 제공
- 명확한 레이트 리밋 정책
- 오픈소스 코드베이스 투명 공개

**StatCan WDS**는 통계 데이터 접근성 측면에서 양호:
- 완전 공개 무인증
- Python 라이브러리(stats_can) 생태계
- SDMX 국제 표준으로 기존 통계 도구 연동

### 7.2 약점

**중앙 API 카탈로그 부재**: GC API Store 종료 이후 전체 연방 API를 일람할 수 있는 허브가 없다. 개발자는 부처별 포털을 개별 탐색해야 한다.

**문서 품질 격차**: 부처별 문서화 수준 편차가 크다. MSC GeoMet·GC Notify는 우수하나, CRA·ESDC·NRCan GeoGratis는 문서가 빈약하다.

**레이트 리밋 불투명**: 대부분 공개 API가 레이트 리밋을 명시하지 않아 자동화 시스템 설계 시 불확실성 초래.

**이중언어 번역 지연**: 신규 데이터셋 메타데이터의 프랑스어 번역이 영어보다 늦는 경우가 상존.

**민간 개발자 접근 제한**: GC Notify, GC Forms 등 CDS 플랫폼은 정부 기관 전용으로 민간 혁신 생태계 활용 불가.

### 7.3 개발자 포털 품질 평가

| API | 품질 등급 | 주요 이유 |
|-----|----------|----------|
| MSC GeoMet | Excellent | OGC 표준, Swagger UI, 무인증 |
| GC Notify | Excellent | OpenAPI, Sandbox, 명확한 문서 |
| Open Government Portal | Good | CKAN 표준, OAS3 최근 추가 |
| StatCan WDS | Good | 문서 양호, 파이썬 라이브러리 |
| Health Canada DPD | Fair | 인증 불필요지만 문서 빈약 |
| NRCan GeoGratis | Fair | 레거시 AtomPub, 문서 노후 |
| CRA API | Poor | GC API Store 종료 이후 접근 불명확 |
| ESDC LMI | Poor | 동일 |

---

## 8. MCP/AI 도구

### 8.1 캐나다 정부 AI 전략

캐나다 연방 정부는 2025-2027 연방 공공서비스 AI 전략을 수립하고 책임있는 AI 활용을 추진 중이다. 주요 정책:

- **캐나다 AI 컴퓨트 접근 기금**: C$3억 규모, 중소기업 AI 인프라 지원
- **캐나다 소버린 AI 컴퓨트 전략**: C$20억 5년 투자 (2024년 12월 발표)
- **연방 공공서비스 AI 사용 지침**: 안전하고 책임있는 AI 활용 원칙 수립

### 8.2 MCP(Model Context Protocol) 통합 현황

2025년 4월 기준, **캐나다 연방 정부가 공식적으로 MCP 서버를 제공하는 사례는 확인되지 않는다**. 이는 미국 GovInfo(GPO)가 2026년 1월 MCP 서버 퍼블릭 프리뷰를 런칭한 것과 대조적이다.

MCP(Anthropic이 2024년 11월 발표한 AI-데이터 연결 표준)는 2025년 기준 97M+ 월간 SDK 다운로드를 기록하며 사실상 AI 에이전트-도구 연결 표준으로 자리잡았으나, 캐나다 정부 API 생태계에는 아직 도입되지 않았다.

### 8.3 AI 활용을 위한 현재 접근 방식

캐나다 정부 데이터에 AI가 접근하기 위해서는 현재 전통적인 REST API 호출 방식에 의존해야 한다:

- **StatCan WDS**: Python `stats_can` 라이브러리를 통한 LLM 파이프라인 연동 가능
- **Open Government Portal CKAN API**: JSON 메타데이터 카탈로그를 RAG(Retrieval Augmented Generation) 파이프라인에 활용 가능
- **MSC GeoMet**: OGC API 표준으로 AI 에이전트의 프로그래밍 방식 접근에 친화적

**향후 전망**: CDS는 2025-2027 전략에서 "인터운영성 향상"과 "API 노출 기능 확대"를 명시했으나 MCP 통합에 대한 구체적 계획은 아직 없다.

---

## 9. 사용자 중심성 평가

### 9.1 종합 점수 (7점 만점)

| API | 가입 | 승인 | 형식 | 문서 | 레이트 리밋 | 오류처리 | SDK | 합계 | 등급 |
|-----|------|------|------|------|------------|---------|-----|------|------|
| MSC GeoMet | 2 | 2 | 2 | 2 | 2 | 1 | 2 | **13** | A |
| GC Notify | 1 | 1 | 2 | 2 | 2 | 2 | 2 | **12** | A |
| Open Gov Portal | 2 | 2 | 2 | 1 | 1 | 1 | 1 | **10** | B |
| StatCan WDS | 2 | 2 | 1 | 2 | 2 | 1 | 1 | **11** | B |
| Health Canada DPD | 2 | 2 | 1 | 1 | 1 | 1 | 0 | **8** | C |
| NRCan GeoGratis | 2 | 2 | 1 | 1 | 1 | 0 | 0 | **7** | C |
| CRA API | 0 | 0 | 1 | 0 | 0 | 0 | 0 | **1** | F |
| ESDC LMI | 0 | 0 | 1 | 0 | 0 | 0 | 0 | **1** | F |

**평가 기준**: 2점=우수, 1점=보통, 0점=미흡

### 9.2 강점 분야

- **접근성**: 주요 데이터 API(기상, 통계, 오픈데이터) 대부분이 인증 불필요로 진입 장벽 낮음
- **국제 표준 준수**: OGC, SDMX, UTF-8, ISO 8601 등 국제 표준 적극 채택
- **오픈소스**: ckanext-canada, GC Notify API, MSC GeoMet 도구 등 주요 플랫폼 오픈소스 공개
- **이중언어**: 이중언어 API 운영 체계 자체는 세계적으로 독보적

### 9.3 개선 필요 분야

- **중앙 카탈로그 부재**: GC API Store 종료 후 공백 지속
- **레이트 리밋 투명성**: 대부분 API가 레이트 리밋 헤더 미제공
- **레거시 부처 API**: CRA, ESDC 등 일부 부처 API의 개발자 접근성 극히 낮음
- **OpenAPI 스펙 보급**: StatCan WDS, Health Canada DPD 등 주요 API에 OpenAPI 스펙 미제공

---

## 10. AI 적합성 평가

### 10.1 종합 점수 (14점 만점)

| API | 기계 스키마 | HTTP 의미론 | JSON 네이티브 | 자기기술 오류 | 프로그래밍 인증 | 레이트 리밋 투명성 | CORS/HTTPS | 합계 | 등급 |
|-----|------------|------------|-------------|-------------|--------------|-----------------|----------|------|------|
| GC Notify | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | A |
| MSC GeoMet | 2 | 2 | 2 | 1 | 2 | 1 | 2 | **12** | A |
| Open Gov Portal | 2 | 1 | 2 | 1 | 2 | 0 | 2 | **10** | B |
| StatCan WDS | 1 | 1 | 2 | 1 | 2 | 1 | 2 | **10** | B |
| GC Forms | 1 | 1 | 2 | 1 | 1 | 0 | 1 | **7** | C |
| Health Canada DPD | 0 | 1 | 2 | 1 | 2 | 0 | 2 | **8** | C |
| NRCan GeoGratis | 1 | 1 | 1 | 0 | 2 | 0 | 2 | **7** | C |
| CRA API | 0 | 1 | 1 | 0 | 0 | 0 | 2 | **4** | D |
| ESDC LMI | 0 | 1 | 1 | 0 | 0 | 0 | 2 | **4** | D |

### 10.2 AI 적합성 총평

캐나다 정부 API의 AI 적합성은 **부처별로 극명한 양극화**를 보인다.

**상위 계층 (GC Notify, MSC GeoMet)**: 구조화된 JSON 응답, OpenAPI 스펙(GC Notify), OGC 표준(MSC GeoMet), HTTPS/CORS 완비로 AI 에이전트 연동에 즉시 활용 가능한 수준.

**중위 계층 (Open Government Portal, StatCan WDS)**: 공개 접근 가능하고 JSON 응답을 제공하지만, OpenAPI 스펙 부재(StatCan) 또는 HTTP 의미론 미흡으로 LLM 도구 연동에 추가 작업 필요.

**하위 계층 (CRA, ESDC)**: GC API Store 종료 이후 API 접근 자체가 불명확해져 AI 파이프라인 구성 불가 수준.

**이중언어 응답**은 AI 처리 관점에서 장점이 될 수 있다 — 동일 엔드포인트에서 영불 병렬 데이터를 반환하면 다국어 AI 시스템 훈련에 고품질 평행 코퍼스를 제공하는 부가 가치가 있다.

---

## 11. 한국과의 비교 시사점

### 11.1 구조적 비교

| 항목 | 캐나다 | 한국 |
|------|--------|------|
| 주력 플랫폼 | CKAN (오픈소스) | 공공데이터포털 (자체 개발) |
| 중앙 API 카탈로그 | GC API Store (2023년 종료, 공백) | 공공데이터포털 통합 |
| 인증 표준 | OAuth 2.0, API Key, GCKey | 공공데이터포털 인증키(단순 문자열) |
| 이중언어 | 영어·프랑스어 헌법적 의무 | 한국어 단일 언어 |
| 데이터 형식 | JSON 우선(신규), CSV/XML(레거시) | XML 비중 높음, JSON 전환 중 |
| 국제 표준 | OGC, SDMX, OpenAPI | 일부 OGC, OpenAPI 부분 채택 |
| 정부 혁신 조직 | Canadian Digital Service (CDS) | 한국지능정보사회진흥원(NIA) 등 |
| MCP 통합 | 미도입 | 미도입 |

### 11.2 캐나다에서 배울 점

**1. CKAN 오픈소스 활용**: 캐나다는 CKAN을 기반으로 하되 ckanext-canada를 통해 자국 요구사항에 맞게 확장했다. 플랫폼 벤더 종속 없이 오픈소스 생태계를 활용하는 전략이 장기적으로 유리하다. 한국은 자체 개발 플랫폼으로 운영 중이나 국제 상호운용성이 낮다.

**2. OGC 국제 표준 선도**: MSC GeoMet은 OGC API의 얼리어답터로 국제 GIS 커뮤니티와 자연스럽게 연동된다. 한국 기상청 API도 국제 표준 전환을 가속할 필요가 있다.

**3. 정부 공통 플랫폼(GC Notify/Forms)**: 부처별 알림 시스템·폼을 각자 개발하지 않고 CDS가 공통 API 플랫폼을 제공하는 방식은 중복 투자를 방지한다. 한국의 전자정부 공통컴포넌트와 방향은 유사하나 API-first 설계와 투명한 오픈소스 공개에서 차이가 있다.

**4. TBS 중심 API 표준 거버넌스**: 단일 기관(TBS)이 API 기술 표준, 이중언어 요건, 보안 가이드라인을 통합 관장한다. 한국은 행정안전부·과학기술정보통신부·NIA 등 다기관 거버넌스로 표준이 분산되어 있다.

### 11.3 캐나다의 반면교사

**1. GC API Store 조기 종료**: 중앙 API 카탈로그를 구축했다가 폐쇄한 사례는 지속 가능한 거버넌스와 예산 확보의 중요성을 보여준다. 한국 공공데이터포털이 현재 안정적으로 운영되는 점은 오히려 강점이다.

**2. 부처별 편차 심화**: 캐나다는 TBS 표준이 있음에도 부처별 구현 수준 격차가 크다(MSC GeoMet vs. CRA). 표준 설정만으로는 부족하며 이행 점검 체계가 필요하다.

**3. 민간 개발자 생태계 미흡**: 주요 CDS 플랫폼이 정부 기관 전용으로 제한되어 민간 혁신을 제한한다. 한국의 공공데이터포털은 민간 개방에 더 적극적이다.

**4. MCP/AI 통합 지연**: 양국 모두 AI 에이전트 시대에 맞는 MCP 표준 통합을 아직 시작하지 않았다. 이는 향후 경쟁력 격차로 이어질 수 있는 공통 과제다.

### 11.4 정책 제언 (한국 관점)

1. **CKAN 기반 국제 표준 전환 검토**: 한국 공공데이터포털의 국제 접근성 향상을 위해 CKAN 도입 또는 DCAT/OGC 표준 호환 인터페이스 추가를 고려.

2. **OGC API 전환 가속**: 국토교통부, 기상청 등 공간 데이터 보유 기관의 OGC API Features 전환을 캐나다 MSC GeoMet 사례를 참고하여 추진.

3. **공통 알림 API 표준화**: 각 부처가 개별 운영하는 알림 시스템을 GC Notify 방식의 공통 플랫폼으로 통합—CDS 사례는 구체적 참조 모델이 된다.

4. **MCP 선제 도입 검토**: 미국 GovInfo(GPO)의 MCP 서버 사례를 참고하여 공공데이터포털 또는 공공 LLM 프로젝트에 MCP 서버 파일럿 도입을 선점 추진.

5. **API 표준 단일 거버넌스**: 캐나다 TBS 모델을 참고하여 한국도 단일 기관(예: 행안부 또는 NIA)이 OpenAPI 스펙 의무화, JSON 우선, 레이트 리밋 투명성 등 API 기술 표준을 통합 관장하는 체계로 전환.

---

## 참고 자료

- [Open Government Portal API 문서](https://open.canada.ca/en/access-our-application-programming-interface-api)
- [Government of Canada Standards on APIs (TBS)](https://www.canada.ca/en/government/system/digital-government/digital-government-innovations/government-canada-standards-apis.html)
- [StatCan Web Data Service User Guide](https://www.statcan.gc.ca/en/developers/wds/user-guide)
- [MSC GeoMet OGC API](https://api.weather.gc.ca/)
- [GC Notify 문서](https://documentation.notification.canada.ca/en/)
- [Canadian Digital Service 2025 Highlights](https://digital.canada.ca/2026/02/25/progress-amp-plans-2025-highlights-for-cds-platform-products)
- [CDS Strategic Vision 2025-2027](https://digital.canada.ca/reports/strategy-2024.pdf)
- [ckanext-canada GitHub](https://github.com/open-data/ckanext-canada)
- [GC API Guidance](https://www.canada.ca/en/government/system/digital-government/digital-government-innovations/enabling-interoperability/api-guidance.html)
- [Health Canada DPD API Guide](https://health-products.canada.ca/api/documentation/dpd-documentation-en.html)
- [OGC APIs for Canada Weather Data](https://www.ogc.org/blog-article/ogc-apis-deliver-real-time-and-archived-weather-climate-and-water-datasets-for-canada/)
