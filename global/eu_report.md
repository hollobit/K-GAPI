# 🇪🇺 EU 정부 API 생태계 분석

> 작성일: 2026년 4월 3일  
> 분석 대상: EU 차원 주요 공공 API 포털 9개  
> 비교 기준: 한국 공공데이터포털(data.go.kr) 생태계

---

## 1. 개요 — EU 차원 + 회원국 구조

### 1.1 근본적 구조 차이

EU의 공공 API 생태계는 한국과 근본적으로 다른 **분산형(Decentralized) 아키텍처**를 채택한다. 한국이 단일 허브(data.go.kr)를 통해 중앙집중적으로 API를 게이트웨이화하는 반면, EU는 **메타데이터 연합(Federated Metadata)** 모델을 취한다.

```
EU 구조:
  [회원국 27개 포털] ──DCAT-AP 메타데이터──► [data.europa.eu] (카탈로그 집계)
  [EU 기관 개별 포털] ──직접 API──────────► [개발자]
  [Eurostat, ECB, EMA...] ──도메인별 API──► [개발자]

한국 구조:
  [원본 기관 API] ──게이트웨이──► [data.go.kr] ──serviceKey──► [개발자]
```

### 1.2 주요 특징

| 항목 | EU | 한국 |
|------|-----|------|
| 아키텍처 | 분산형 연합 | 중앙집중 게이트웨이 |
| 인증 방식 | 대부분 키 없음(Keyless) | API 키 필수(serviceKey) |
| 데이터 형식 | JSON + RDF/SPARQL + SDMX | XML 기본, JSON 선택 |
| 법적 근거 | Open Data Directive + HVD 규정 | 공공데이터법 |
| 다국어 지원 | 24개 EU 공식 언어 | 한국어 중심 |
| 링크드데이터 | 성숙한 RDF 생태계 | 미미한 수준 |
| 메타데이터 표준 | DCAT-AP (강제) | 자체 표준(느슨) |

### 1.3 포털 유형별 분류

**EU 기관 직접 운영 포털** (본 연구 분석 대상):
- data.europa.eu — 범EU 데이터 카탈로그 (집계자/허브)
- Eurostat API — EU 통계청 통계 API
- ECB Data Portal — 유럽중앙은행 금융/경제 통계
- INSPIRE Geoportal — 범EU 공간정보 인프라
- Copernicus Data Space — 위성·지구관측 데이터
- EU Publications Office (CELLAR) — EU 법령·출판물 링크드데이터
- EMA ePI API — 의약품 정보 FHIR API
- ECHA CHEM — 화학물질 데이터베이스
- European Parliament Open Data — 의회 데이터 API
- Europeana — 유럽 문화유산 API

**회원국 국가 포털** (본 연구 범위 외, 참고):
- data.gov.uk (영국, EU 탈퇴 후 독자 운영)
- data.gouv.fr (프랑스), govdata.de (독일) 등 각국 포털

---

## 2. 주요 포털 상세 분석

### 2.1 data.europa.eu — 유럽 데이터 포털

**기본 정보**
- URL: https://data.europa.eu
- 운영: 유럽위원회 / EU 출판국
- 역할: 70개 이상 국가·기관 데이터 포털 메타데이터 집계 허브

**API 구조**
data.europa.eu는 두 가지 핵심 프로그래밍 인터페이스를 제공한다:

1. **REST API**: SwaggerHub에 OpenAPI 스펙 공개 (https://app.swaggerhub.com/apis/EU-Open-Data-Portal/eu-open_data_portal/). 기본 URL `https://data.europa.eu/euodp/data/apiodp/`. JSON 형식 입출력. 데이터셋 검색, 조회, 메타데이터 접근 지원.

2. **SPARQL 엔드포인트**: `https://data.europa.eu/data/sparql`. DCAT-AP 기반 RDF 메타데이터를 SPARQL 언어로 쿼리. 복잡한 크로스-포털 데이터 검색에 강력.

**인증**: 읽기/검색 API는 완전 무료·무인증. 쓰기 API는 OpenID Connect 사용(ODP 팀 연락 필요).

**다국어**: 24개 EU 공식 언어 지원. SPARQL 쿼리에서 언어 필터로 특정 언어 메타데이터만 조회 가능.

**개발자 경험 평가**: A등급 (12/14점)
- 강점: 가입 불필요, OpenAPI 스펙, SPARQL 지원, DCAT-AP 표준
- 약점: 에러 메시지 표준화 미흡, SDK 공식 제공 부재

---

### 2.2 Eurostat API — EU 통계청

**기본 정보**
- URL: https://ec.europa.eu/eurostat/web/user-guides/data-browser/api-data-access
- 운영: Eurostat (유럽위원회 통계총국)
- 보유 데이터셋: 약 1,000개 이상 통계 데이터셋

**API 버전 및 기술 스펙**

| API | 버전 | 엔드포인트 패턴 |
|-----|------|----------------|
| SDMX 3.0 REST | 최신 | `/eurostat/api/dissemination/sdmx/3.0/...` |
| SDMX 2.1 REST | 안정 | `/eurostat/api/dissemination/sdmx/2.1/...` |
| API Statistics | 레거시 | 파라미터 순서 자유 |
| Catalogue API | 탐색용 | 데이터셋 목록/구조 조회 |
| Bulk Download | 대용량 | TSV/SDMX 전체 파일 다운로드 |

**지원 형식**: JSON-stat(경량), SDMX-ML, SDMX-CSV, TSV, JSON

**Swagger UI**: `https://ec.europa.eu/eurostat/api/dissemination/swagger-ui` — OpenAPI 3.0 스펙 완전 공개.

**공식 SDK**: R 패키지 `eurostat`, `restatapi` / Python 패키지 `pyrostat` (Eurostat GitHub 공식 제공)

**인증**: 완전 무료·무인증. API 키 없이 즉시 사용.

**개발자 경험 평가**: A등급 (14/14점, 분석 대상 중 최고)
- 강점: 인증 불필요, 다중 API 버전, 공식 SDK, Swagger UI, Bulk Download
- 약점: 레이트 리밋 투명성 부족

---

### 2.3 ECB Data Portal — 유럽중앙은행

**기본 정보**
- URL: https://data.ecb.europa.eu
- 운영: 유럽중앙은행(ECB)
- 제공 데이터: 통화정책, 환율, 금리, 은행통계, 지급결제

**SDMX 2.1 RESTful API**

두 가지 운영 모드:
- **Data Retrieval Mode**: 특정 데이터 직접 조회
- **Data Discovery Mode**: 메타데이터 기반 데이터 탐색

접근 URL: `https://data-api.ecb.europa.eu/service/`

**Third-party 라이브러리**: R 패키지 `ecb`, Python 패키지 `ecbdata`, `pandaSDMX`

**인증**: 완전 무료·무인증.

**개발자 경험 평가**: A등급 (12/14점)
- 강점: 인증 불필요, SDMX 표준, 다양한 언어별 라이브러리
- 약점: OpenAPI 스펙 미공개, 공식 SDK 부재

---

### 2.4 INSPIRE Geoportal — 범EU 공간정보

**기본 정보**
- URL: https://inspire-geoportal.ec.europa.eu
- 법적 근거: INSPIRE Directive 2007/2/EC
- 운영: 유럽위원회 / 공동연구센터(JRC)

**제공 서비스**

| 서비스 유형 | OGC 표준 | 설명 |
|------------|----------|------|
| View Service | WMS 1.3 / WMTS | 지도 타일/이미지 조회 |
| Download Service | WFS 2.0 / ATOM | 벡터 데이터 다운로드 |
| Discovery Service | CSW | 공간데이터 메타데이터 검색 |
| Coverage | WCS | 래스터 데이터 조회 |

**아키텍처**: 34개 Annex 테마별로 각 회원국이 서비스를 운영하고, INSPIRE Geoportal이 중앙 집계. 완전한 분산형.

**한계**: OGC 표준 기반으로 기술적으로 성숙하나 REST/JSON 친화성은 낮음. GeoServer 등 오픈소스 서버로 구현.

**개발자 경험 평가**: B등급 (9/14점)
- 강점: 인증 불필요, 국제 OGC 표준
- 약점: REST/JSON 미성숙, SDK 부재, 문서 분산

---

### 2.5 Copernicus Data Space — 위성·지구관측

**기본 정보**
- URL: https://dataspace.copernicus.eu
- 운영: ESA / 유럽위원회 코페르니쿠스 프로그램
- 2023년 Open Access Hub 대체

**주요 API**

| API | 프로토콜 | 용도 |
|-----|----------|------|
| STAC API | REST/JSON | 위성영상 카탈로그 검색 |
| OData API | REST/JSON | 제품 다운로드 |
| Sentinel Hub API | REST/JSON | 처리 및 시각화 |
| openEO API | REST/JSON | 클라우드 분석 처리 |
| OGC WMS/WMTS | OGC | 지도 시각화 |

**인증**: 무료 계정(이메일) 생성 후 OAuth2. Sentinel Hub는 OAuth client ID/secret. 기본 할당량 내 무료.

**개발자 경험 평가**: A등급 (13/14점)
- 강점: 현대적 API 스택, STAC/openEO, 무료 티어, OpenAPI 스펙
- 약점: 레이트 리밋 정책 복잡

---

### 2.6 EU Publications Office / CELLAR — 법령 링크드데이터

**기본 정보**
- URL: https://op.europa.eu/en/web/cellar
- SPARQL: https://publications.europa.eu/webapi/rdf/sparql
- 운영: EU 출판국

**CELLAR 시스템**: 모든 EU 공식 출판물(유럽연합관보 OJ, 법령, 보고서)을 Virtuoso 트리플 스토어에 RDF로 저장. Common Data Model(CDM) 온톨로지 기반 지식그래프.

**접근 방법**:
1. SPARQL 엔드포인트: CDMQ 언어로 EU법 전체 쿼리
2. HTTP REST: ELI(European Legislation Identifier) URI로 특정 법령 직접 접근
3. R 패키지 `eurlex`: SPARQL/REST 오버헤드 추상화

**인증**: 완전 무료·무인증.

**개발자 경험 평가**: A등급 (11/14점)
- 강점: 전체 EU법 지식그래프, SPARQL, 인증 불필요
- 약점: JSON 네이티브 미흡, 진입장벽(SPARQL 학습 필요)

---

### 2.7 EMA ePI API — 의약품 정보

**기본 정보**
- URL: https://epi.developer.ema.europa.eu
- 운영: 유럽의약품청(EMA)
- FHIR IG: https://epi.ema.europa.eu/fhirig/

**FHIR 기반**: EU 전자제품정보(ePI) 공통 표준은 HL7 FHIR(Fast Healthcare Interoperability Resources) 기반. R4/R5 Implementation Guide 공개.

**ePI Consuming API**: 구독이나 API 키 없이 의약품 허가 정보(SPC, PIL, 라벨링) 조회 가능.

**추가 DB**: EudraGMDP(제조/수입 허가, GMP/GDP 인증서) 별도 개방.

**개발자 경험 평가**: A등급 (12/14점)
- 강점: FHIR 국제 표준, 인증 불필요, IG 공개
- 약점: 파일럿 단계(일부 기능 미완성), SDK 부재

---

### 2.8 ECHA CHEM — 화학물질 데이터베이스

**기본 정보**
- URL: https://echa.europa.eu/echa-chem
- 운영: 유럽화학물질청(ECHA)
- 2024년 초 출시

**주요 데이터**: REACH 등록물질 10만개 이상, C&L 인벤토리, 규제 정보.

**API 유형**:
- **데이터 조회 API**: EC 번호, CAS 번호, 화학명으로 검색. 무료 개방.
- **S2S Submission API**: 기업 IT 시스템과 ECHA 제출 시스템 직접 연동. 기업 등록 필요.

**오픈데이터**: 15,000개 화학물질 데이터 공개(2024년 발표).

**개발자 경험 평가**: A등급 (12/14점)
- 강점: 조회 API 무료, 샌드박스 환경, OpenAPI 스펙
- 약점: 에러 셀프 디스크립션 부족, 레이트 리밋 불투명

---

### 2.9 European Parliament Open Data — 의회 데이터

**기본 정보**
- URL: https://data.europarl.europa.eu
- 운영: 유럽의회

**주요 데이터셋**:
- MEP 정보: 의원 국가, 정치그룹, 국내정당
- 위원회/대표단: 정치그룹, 위원회, 대표단 목록
- 채택 텍스트: 법안, 결의안, 의견서
- 회의 기록: 의제, 속기록, 표결 결과

**기술 특징**: 온톨로지 1개 + 애플리케이션 프로파일 2개 공개. JSON-LD + DCAT-AP 준수. Release notes 정기 공개.

**인증**: 완전 무료·무인증.

**개발자 경험 평가**: A등급 (12/14점)

---

### 2.10 Europeana API — 문화유산

**기본 정보**
- URL: https://api.europeana.eu
- 운영: Europeana Foundation
- 규모: 5,000만 건 이상 문화유산 객체

**API 종류**: Search API, Record API, IIIF Manifest API, Fulltext API

**인증**: 이메일 계정 생성 후 무료 API 키 발급. 개인키(즉시)/프로젝트키(검토 후).

**메타데이터**: Europeana Data Model(EDM) 기반. SPARQL 엔드포인트로 LOD 쿼리 가능.

**개발자 경험 평가**: A등급 (11/14점)
- 약점: 계정 가입 필요, 레이트 리밋 정보 불투명

---

## 3. High Value Dataset 규정 — 법적 의무화 내용

### 3.1 법령 체계

**Open Data Directive (Directive 2019/1024/EU)**
- 2019년 7월 16일 발효, 회원국은 2021년 7월까지 국내법 전환 의무
- 구 PSI 지침(2003, 2013 개정) 대체
- 핵심 원칙: "공공 자금으로 생산된 데이터는 상업적·비상업적 재이용 가능해야 한다"
- 기계 가독 형식, 오픈 표준, 메타데이터 포함 제공 의무

**High Value Datasets Implementing Regulation (Commission Implementing Regulation 2023/138)**
- 2023년 1월 공표, 최대 16개월 이내(2024년 중반) 시행 의무
- Open Data Directive Article 14에 따른 위임입법

### 3.2 6대 고가치 데이터셋 카테고리

| 카테고리 | 주요 데이터 예시 | API 의무 |
|---------|----------------|---------|
| **1. 지리공간(Geospatial)** | 행정구역, 주소, 건물, 도로망 | API + 벌크 다운로드 |
| **2. 지구관측·환경(Earth Observation & Environment)** | 위성영상, 대기질, 토양 | API 필수 |
| **3. 기상(Meteorology)** | 기상 예보, 과거 기상 데이터 | API 필수 |
| **4. 통계(Statistics)** | 인구, GDP, 무역, 고용 | API + 벌크 다운로드 |
| **5. 기업·기업소유권(Company & Company Ownership)** | 법인등록, 수익적 소유자 | API 필수 |
| **6. 이동성(Mobility)** | 대중교통, 도로 교통 | API 필수 |

### 3.3 API 의무 세부 요건

**의무 사항**:
- 기계 가독 형식(machine-readable format)으로 API를 통한 제공
- 원칙적 **무료** 제공 (단, 익명화·상업적 정보 보호 등 한계비용 회수 가능)
- API 이용약관 및 서비스 품질 기준을 **인간 가독 + 기계 가독** 양 형식으로 공개
- 합리적 재사용자 수요에 부합하는 성능·용량·가용성 보장
- 해당 시 벌크 다운로드도 함께 제공

**준수 현황**: 회원국별 이행 수준 편차 존재. EU 기관 포털(Eurostat, ECB 등)은 이미 충족. 일부 회원국은 이행 지연.

---

## 4. DCAT-AP 메타데이터 표준 + SPARQL/RDF

### 4.1 DCAT-AP 개요

**DCAT-AP(Data Catalog Application Profile)**는 W3C DCAT 표준의 EU 적용 프로파일. EU 공공 데이터 포털 간 메타데이터 교환의 공통 어휘.

- 현재 버전: **DCAT-AP v3.0.0** (최신)
- RDF/Turtle, JSON-LD로 표현
- 해결하는 문제: 각국 포털의 이질적 메타데이터를 단일 표준으로 통합 → 크로스-포털 검색 가능
- **HVD AP**: HVD 규정 준수를 위한 DCAT-AP 확장 프로파일
- **HealthDCAT-AP**: 의료 데이터용 확장 프로파일

**핵심 클래스**:
```turtle
dcat:Catalog       → 데이터 카탈로그 (포털)
dcat:Dataset       → 데이터셋
dcat:Distribution  → 배포본 (특정 형식/URL)
dcat:DataService   → API 서비스
dct:Agent          → 발행자/창작자
```

### 4.2 SPARQL 엔드포인트 생태계

EU는 세계에서 가장 성숙한 정부 SPARQL 엔드포인트 생태계를 보유:

| 엔드포인트 | URL | 데이터 범위 |
|-----------|-----|------------|
| data.europa.eu SPARQL | https://data.europa.eu/data/sparql | EU 전체 오픈데이터 메타데이터 |
| CELLAR SPARQL | https://publications.europa.eu/webapi/rdf/sparql | EU 법령·출판물 전체 |
| Eurostat NUTS SPARQL | 별도 엔드포인트 | 통계 지역 분류 |
| Europeana SPARQL | pro.europeana.eu/page/sparql | 유럽 문화유산 |
| EU Vocabularies | op.europa.eu | EU 통제어휘·온톨로지 |

### 4.3 링크드데이터 활용 예시

```sparql
# data.europa.eu SPARQL: 고가치 데이터셋(HVD) 중 지리공간 카테고리 검색
PREFIX dcat: <http://www.w3.org/ns/dcat#>
PREFIX dcatap: <http://data.europa.eu/r5r/>
PREFIX dct: <http://purl.org/dc/terms/>

SELECT ?dataset ?title ?publisher WHERE {
  ?dataset a dcat:Dataset ;
           dct:title ?title ;
           dct:publisher ?publisher ;
           dcatap:applicableLegislation <http://data.europa.eu/eli/reg_impl/2023/138/oj> .
  FILTER(lang(?title) = "en")
}
LIMIT 10
```

### 4.4 HTTP Content Negotiation

EU API들은 HTTP `Accept` 헤더를 통한 콘텐츠 협상을 지원:
```
Accept: application/ld+json      → JSON-LD
Accept: text/turtle              → Turtle RDF
Accept: application/rdf+xml      → RDF/XML
Accept: application/json         → JSON
```

이를 통해 동일 URI에서 다양한 형식으로 데이터 접근 가능(자원 중심 설계).

---

## 5. 인증 체계

### 5.1 EU API 인증 패턴 분류

EU 정부 API의 가장 큰 특징은 **대부분이 인증 불필요(Keyless)**라는 점:

| 포털 | 인증 방식 | 가입 필요 |
|------|---------|---------|
| data.europa.eu (읽기) | 없음(Keyless) | 불필요 |
| Eurostat API | 없음(Keyless) | 불필요 |
| ECB Data Portal | 없음(Keyless) | 불필요 |
| INSPIRE Geoportal | 없음(Keyless) | 불필요 |
| CELLAR SPARQL | 없음(Keyless) | 불필요 |
| EMA ePI API | 없음(Keyless) | 불필요 |
| ECHA 조회 API | 없음(Keyless) | 불필요 |
| European Parliament ODP | 없음(Keyless) | 불필요 |
| Copernicus Data Space | OAuth2 | 이메일(무료) |
| Europeana API | API Key (Header) | 이메일(무료) |
| data.europa.eu (쓰기) | OpenID Connect | ODP 팀 연락 |
| ECHA S2S Submission | API Key | 기업 등록 |

### 5.2 한국과의 인증 비교

한국 공공 API는 사실상 모든 포털에서 serviceKey 발급 필수. EU는 조회 API의 90% 이상이 키 없이 즉시 사용 가능. 이는 개발자 온보딩 속도, AI/LLM 도구 통합 용이성에서 EU가 압도적으로 유리한 요인.

---

## 6. 개발자 경험

### 6.1 강점

**문서화 수준**
- Eurostat: Swagger UI + 상세 사용자 가이드 + 공식 R/Python 패키지
- Copernicus: 현대적 API 문서, 예제 노트북, 처리 단위(Processing Units) 개념 명확 설명
- European Parliament: Release Notes 정기 공개, 온톨로지 문서

**즉시 사용 가능성 (Time-to-First-Call)**
- Eurostat, ECB, CELLAR: URL 알면 즉시 사용. 가입·키 불필요.
- Copernicus: 이메일 가입 후 수 분 내 토큰 발급

**SDK 생태계**
- Eurostat: `eurostat` (R), `restatapi` (R), `pyrostat` (Python)
- ECB: `ecb` (R), `ecbdata` (Python), `pandaSDMX` (Python)
- EMA: FHIR 기반으로 HAPI FHIR 등 범용 라이브러리 활용 가능
- EUR-Lex: `eurlex` (R)

### 6.2 약점

- **SPARQL 학습 곡선**: CELLAR, data.europa.eu의 최대 활용을 위해서는 SPARQL 쿼리 언어 필요
- **분산 구조의 복잡성**: 데이터가 어느 포털에 있는지 파악이 필요. 단일 진입점 없음.
- **레이트 리밋 불투명**: 대부분 포털에서 정확한 레이트 리밋 정책 미공개
- **에러 표준화 부재**: 포털별 에러 메시지 형식 상이, RFC 7807(Problem Details) 미준수 포털 다수
- **공식 SDK 부재 포털**: ECB, ECHA 등은 서드파티 라이브러리에 의존

### 6.3 developer.europa.eu 통합 시도

EU는 개발자 경험 개선을 위해 통합 개발자 허브(https://developer.europa.eu) 운영 시도. 각 기관 API를 한 곳에서 탐색 가능하도록 연결. 그러나 한국 data.go.kr의 통합 게이트웨이 수준에는 미치지 못함.

---

## 7. MCP/AI 도구 통합

### 7.1 현황

2025-2026년 기준 EU 정부 기관이 **공식 MCP(Model Context Protocol) 서버를 출시한 사례는 없다**. 그러나 EU API의 Keyless 특성과 OpenAPI 스펙 공개로 인해 MCP 통합이 기술적으로 매우 용이하다.

### 7.2 AI 통합 관점에서의 EU API 특징

**LLM/AI 도구 통합 유리 요인**:

1. **Keyless API**: API 키 관리 없이 LLM 에이전트가 직접 호출 가능
2. **OpenAPI 스펙**: Eurostat Swagger UI 등 OpenAPI 스펙 존재 → AI 도구 자동 연동
3. **SPARQL**: 지식그래프 쿼리를 통한 복잡한 법령/데이터 연관 검색 → LLM 추론 보완
4. **JSON-LD + RDF**: 의미론적 데이터 → AI 컨텍스트 풍부화
5. **DCAT-AP**: 표준화된 메타데이터 → 데이터 발견 자동화

**Community MCP 서버 (비공식)**:
- Eurostat 데이터용 MCP 서버: 오픈소스 커뮤니티에서 개발 시도
- EUR-Lex 법령 검색용 MCP: 법률 AI 도구에서 활용 가능
- ECB 환율/금리 데이터용 경량 래퍼 API: AI 에이전트용 서드파티 서비스

### 7.3 EU AI Act와 공공 API

EU AI Act(2024년 발효)의 맥락에서 고위험 AI 시스템(의료, 사법 등)에 활용되는 공공 API 데이터의 품질과 추적성 요건이 강화되는 추세. EMA FHIR API, ECHA 화학물질 API 등이 AI 의사결정 파이프라인에 활용 시 데이터 출처 명시 의무 부각.

### 7.4 Common European Data Spaces

EU는 도메인별 **데이터 스페이스(Data Spaces)** 구축 중:
- 모빌리티 데이터 스페이스
- 에너지 데이터 스페이스
- 의료 데이터 스페이스 (EHDS - European Health Data Space)
- 농업 데이터 스페이스

데이터 스페이스는 GAIA-X 기반 연합 인프라로 AI 파이프라인과의 통합 용이성 강화 예정.

---

## 8. 사용자 중심성 평가

### 8.1 포털별 종합 점수 (14점 만점)

| 포털 | 가입 | 승인 | 형식 | 문서 | 레이트 | 에러 | SDK | 합계 | 등급 |
|------|------|------|------|------|--------|------|-----|------|------|
| Eurostat API | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | **A** |
| Copernicus | 2 | 2 | 2 | 2 | 1 | 2 | 2 | 13 | A |
| data.europa.eu | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 12 | A |
| ECB Portal | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 12 | A |
| EMA ePI | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 12 | A |
| ECHA CHEM | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 12 | A |
| EP Open Data | 2 | 2 | 2 | 2 | 2 | 1 | 1 | 12 | A |
| CELLAR | 2 | 2 | 1 | 2 | 2 | 1 | 1 | 11 | A |
| Europeana | 1 | 2 | 2 | 2 | 1 | 1 | 2 | 11 | A |
| INSPIRE | 2 | 2 | 1 | 1 | 2 | 1 | 0 | 9 | B |

### 8.2 분석

**전반적 우수 사항**:
- **가입 편의성(signup_ease)**: Keyless API 덕분에 대부분 2점 만점. 한국 공공 API 대비 압도적 우위.
- **승인 속도(approval_speed)**: 인증 불필요 포털은 즉시 사용 → 2점.
- **데이터 형식(data_format)**: JSON, RDF, SDMX 등 다양한 형식 → 대부분 2점.
- **문서 품질(doc_quality)**: Eurostat, EP, CELLAR 등 우수.

**개선 필요 사항**:
- **에러 핸들링(error_handling)**: 대부분 1점. 표준화된 에러 형식 미흡.
- **SDK/샘플(sdk_samples)**: 기관별 편차 큼. Eurostat 최고, INSPIRE 최저.
- **레이트 리밋 투명성**: 정책 불명확성이 공통 약점.

---

## 9. AI 적합성 평가

### 9.1 포털별 AI 적합성 점수 (14점 만점)

| 포털 | 스키마 | HTTP의미론 | JSON네이티브 | 에러자기기술 | 프로그인증 | 레이트투명 | CORS/HTTPS | 합계 | 등급 |
|------|--------|-----------|------------|-----------|----------|----------|-----------|------|------|
| Copernicus | 2 | 2 | 2 | 2 | 2 | 2 | 2 | **14** | **A** |
| Eurostat | 2 | 2 | 2 | 2 | 2 | 1 | 2 | 13 | A |
| EMA ePI | 2 | 2 | 2 | 2 | 2 | 1 | 2 | 13 | A |
| data.europa.eu | 2 | 2 | 2 | 1 | 2 | 1 | 2 | 12 | A |
| ECB Portal | 2 | 2 | 2 | 1 | 2 | 1 | 2 | 12 | A |
| ECHA CHEM | 2 | 2 | 2 | 1 | 2 | 1 | 2 | 12 | A |
| EP Open Data | 2 | 2 | 2 | 1 | 2 | 1 | 2 | 12 | A |
| Europeana | 2 | 2 | 2 | 1 | 1 | 1 | 2 | 11 | A |
| CELLAR | 2 | 1 | 1 | 1 | 2 | 1 | 2 | 10 | B |
| INSPIRE | 1 | 1 | 1 | 1 | 2 | 1 | 2 | 9 | B |

### 9.2 AI 적합성 특이사항

**AI 관점에서 EU API의 최대 강점**:

1. **Keyless 접근(programmatic_auth 2점 다수)**: LLM 에이전트가 코드 안에 자격증명 없이 즉시 API 호출 가능. Function Calling, Tool Use 구현 시 인증 관리 불필요.

2. **OpenAPI 스펙 제공**: Eurostat Swagger UI, Copernicus OpenAPI → AI 도구가 API 스펙 자동 파악 가능.

3. **HTTPS + CORS**: 모든 주요 API 지원 → 브라우저 기반 AI 도구에서도 직접 호출 가능.

4. **FHIR + RDF + SPARQL**: 구조화된 의미론적 데이터 → LLM 컨텍스트 품질 향상.

**AI 관점에서의 약점**:

1. **에러 자기기술 부족**: LLM이 에러 상황을 자동으로 해석·복구하기 어려움.
2. **레이트 리밋 불투명**: AI 에이전트의 자동 재시도 전략 수립 어려움.
3. **SPARQL 복잡성**: CELLAR/INSPIRE의 최대 활용을 위해 LLM이 SPARQL 생성 능력 필요.

---

## 10. 한국과의 비교 시사점

### 10.1 한국이 EU로부터 배울 점

**① Keyless API 확대**

한국의 모든 공공 API는 serviceKey 발급 필수. EU 방식처럼 **읽기 전용 API는 인증 없이** 개방하는 정책 전환 검토 필요. AI 에이전트, 스크래핑 없이도 접근 가능한 구조.

**② SDMX/JSON-stat 통계 표준 도입**

Eurostat의 SDMX 3.0 + JSON-stat 조합은 통계 데이터 전달의 국제 표준. 한국 통계청(KOSIS) API는 자체 포맷 사용으로 국제 호환성 낮음.

**③ RDF/SPARQL 링크드데이터**

한국 공공데이터는 RDF 생태계가 거의 전무. 기관 간 데이터 연결, AI 지식그래프 구축을 위해 DCAT-AP 수준의 메타데이터 표준화 필요.

**④ OpenAPI 스펙 의무화**

한국 공공 API의 OpenAPI/Swagger 스펙 제공률은 매우 낮음. EU처럼 Swagger UI 표준 제공을 의무화하면 AI 도구 통합 속도 대폭 향상.

**⑤ 법적 의무화(HVD 수준)**

EU HVD 규정처럼 6대 핵심 데이터셋을 API + 무료 + 기계 가독 형식으로 **법적 의무 제공**하는 제도 도입. 한국의 공공데이터법 강화 방향.

### 10.2 한국이 EU보다 우수한 점

**① 통합 게이트웨이의 편의성**

data.go.kr의 단일 serviceKey로 수천 개 API 접근이 가능한 구조는, EU의 분산 구조에서는 구현하기 어려운 통합성. 초보 개발자 진입장벽이 낮음.

**② 승인·메타데이터 표준화 수준**

data.go.kr은 모든 API를 통일된 메타데이터 형식으로 카탈로그화. EU는 회원국별 메타데이터 품질 편차가 큼.

**③ 민간 수요 대응 속도**

한국은 새로운 데이터 수요에 빠르게 API를 추가하는 편. EU는 의사결정 구조상 속도가 느림.

**④ 국문 중심 활용 용이성**

EU의 다국어 지원은 강점이지만, 동시에 모든 문서·메타데이터가 여러 언어로 분산되어 특정 언어 개발자에게는 오히려 복잡.

### 10.3 종합 비교 매트릭스

| 평가 차원 | EU | 한국 | 우위 |
|-----------|-----|------|------|
| 인증 편의성 | ★★★★★ | ★★☆☆☆ | EU |
| 표준 준수 (국제) | ★★★★★ | ★★☆☆☆ | EU |
| 링크드데이터 성숙도 | ★★★★★ | ★☆☆☆☆ | EU |
| AI/LLM 통합 용이성 | ★★★★☆ | ★★☆☆☆ | EU |
| 법적 의무화 수준 | ★★★★★ | ★★★☆☆ | EU |
| 통합 검색 편의성 | ★★★☆☆ | ★★★★☆ | 한국 |
| 초보자 접근성 | ★★★☆☆ | ★★★★☆ | 한국 |
| 에러 핸들링 표준화 | ★★☆☆☆ | ★★☆☆☆ | 동등 |
| SDK/공식 라이브러리 | ★★★☆☆ | ★★☆☆☆ | EU |
| 데이터 형식 다양성 | ★★★★★ | ★★★☆☆ | EU |

### 10.4 정책 제언

1. **단기**: 공공 API에 OpenAPI/Swagger 스펙 제공 의무화. KOSIS 등 주요 통계 API에 SDMX 표준 적용.
2. **중기**: 고가치 데이터셋 범주 법제화(EU HVD 참고). 읽기 전용 API의 인증 의무 완화.
3. **장기**: DCAT-AP 기반 전국 오픈데이터 메타데이터 연합 구축. 기관 간 RDF/SPARQL 연계 인프라 마련. 공공 API MCP 서버 표준 가이드라인 수립.

---

## 참고 자료

- [EU Open Data Directive 2019/1024](https://eur-lex.europa.eu/eli/dir/2019/1024/oj/eng)
- [High Value Datasets Regulation 2023/138](https://eur-lex.europa.eu/eli/reg_impl/2023/138/oj/eng)
- [DCAT-AP v3.0.0 Specification](https://op.europa.eu/en/web/eu-vocabularies/dcat-ap)
- [data.europa.eu API Documentation](https://dataeuropa.gitlab.io/data-provider-manual/api-documentation/)
- [Eurostat SDMX API Guide](https://ec.europa.eu/eurostat/web/user-guides/data-browser/api-data-access/api-introduction)
- [Eurostat Swagger UI](https://ec.europa.eu/eurostat/api/dissemination/swagger-ui)
- [ECB SDMX API Overview](https://data.ecb.europa.eu/help/api/overview)
- [Copernicus Data Space APIs](https://documentation.dataspace.copernicus.eu/APIs.html)
- [CELLAR Linked Data & SPARQL](https://op.europa.eu/en/web/webtools/linked-data-and-sparql)
- [EMA ePI FHIR API](https://epi.developer.ema.europa.eu/api-details)
- [ECHA CHEM Database](https://echa.europa.eu/echa-chem/)
- [European Parliament Open Data API](https://data.europarl.europa.eu/en/developer-corner/opendata-api)
- [Europeana API](https://pro.europeana.eu/page/apis)
- [INSPIRE Geoportal](https://inspire-geoportal.ec.europa.eu/)
- [EU Digital Strategy - Open Data](https://digital-strategy.ec.europa.eu/en/policies/legislation-open-data)
