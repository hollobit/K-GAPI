# 해외 정부 API 디자인 가이드라인 비교 분석

> 작성일: 2026년 4월 3일  
> 목적: 주요 국가별 정부 공식 API 디자인 표준을 비교하고, 한국의 현황 대비 도입 우선순위를 도출한다.

---

## 1. 개요

각국 정부의 공식 API 디자인 표준 문서를 비교 분석한다. 조사 대상은 영국(GDS), 미국(GSA/18F), 호주(DTA), EU(INSPIRE/EIF), 프랑스(DINUM) 5개국/지역이며, 한국은 현재 통합 API 표준이 부재한 상태다.

### 조사 문서 및 출처

| 국가/지역 | 발행 기관 | 문서명 | 공개 위치 |
|----------|---------|-------|---------|
| 영국 | Government Digital Service (GDS) | API Technical and Data Standards | gov.uk/guidance/gds-api-technical-and-data-standards |
| 미국 | General Services Administration (GSA), 18F | API Standards | github.com/18F/api-standards, github.com/GSA/api-standards |
| 호주 | Digital Transformation Agency (DTA) | National API Design Standards (NAPIDS) | api.gov.au, github.com/apigovau/national-api-design-standards |
| EU | European Commission, INSPIRE MIF | INSPIRE Technical Guidelines, EIF, HVD Regulation | inspire.ec.europa.eu, interoperable-europe.ec.europa.eu |
| 프랑스 | Direction Interministérielle du Numérique (DINUM) | Cadre de recommandations API | numerique.gouv.fr, api.gouv.fr |
| 한국 | 행정안전부, NIA | (통합 표준 없음) | data.go.kr (비공식 포털 가이드 수준) |

---

## 2. 국가별 가이드라인 상세

### 2.1 영국 — GDS API Technical and Data Standards

- **관할**: Government Digital Service (GDS)
- **문서 위치**: https://www.gov.uk/guidance/gds-api-technical-and-data-standards
- **최근 업데이트**: 2024년 7월 19일 (아키텍처 스타일, 보안, 버저닝 항목 추가)
- **법적 구속력**: 사실상 의무 — 중앙정부(Central Government) 대상 GDS 서비스 표준의 일부로 적용되며, 미준수 시 서비스 승인 불가

#### 아키텍처
- RESTful HTTP API 필수
- JSON 응답 표준화 (`application/json`)
- XML 사용: 레거시 SOAP/XML API와 상호운용이 불가피한 경우에만 예외적 허용, 신규 API는 JSON만 사용
- OpenAPI Specification 3.0: 오픈 스탠다즈 보드(Open Standards Board) 권고 표준으로 지정, 실질적 필수

#### 보안 및 인증
- HTTPS 필수 (TLS, HSTS 적용 권장)
- OAuth 2.0 필수: 서비스간 통신은 Client Credentials, 사용자 접근은 Authorization Code + PKCE
- OpenID Connect (OIDC): 사용자 인가에 사용, OAuth 2.0 위에서 구동
- API Key/Basic Auth: 명시적으로 지양 — OAuth 2.0 대비 보안 약점이 큰 단순 방식으로 간주
- JWT: 민감 케이스에서 OAuth 2.0 토큰 형식으로 활용 (Authorization: Bearer 헤더 전달)
- 보안 설계(Secure by Design) 원칙 의무화

#### HTTP 상태코드 및 에러 처리
- 표준 HTTP 상태코드 사용 필수: 200, 201, 204, 400, 401, 403, 404, 409, 429, 500
- 에러 응답: RFC 7807 Problem Details for HTTP APIs 권장
  - `type`, `title`, `status`, `detail`, `instance` 필드 구조화
- 400 Bad Request만으로는 불충분 — 구체적 검증 오류를 응답 바디에 포함

#### 페이지네이션 및 데이터 표현
- 페이지네이션: cursor-based 방식 권장 (안정적 대용량 데이터 처리)
- 날짜/시간: ISO 8601 형식 사용
- 인코딩: UTF-8 필수

#### 버저닝
- URL path 방식 권장: `/v1/`, `/v2/`
- 주요 변경(breaking change) 시 버전 번호 증가
- 이전 버전 유지 기간 명시 및 지원 종료(EOL) 정책 문서화

#### 운영 요건
- Rate limiting 헤더 필수: `X-RateLimit-Limit`, `X-RateLimit-Remaining` 등
- Sandbox/테스트 환경 제공 필수 — 통합 전 테스트 가능한 환경
- CORS 지원 필수 — 웹 브라우저 클라이언트를 위한 교차 출처 허용
- API 카탈로그(api.gov.uk) 등록 필수 — 중앙 집중식 API 검색 가능
- API 문서는 공개 인터넷 기본 게시 원칙

---

### 2.2 미국 — 18F/GSA API Standards

- **관할**: General Services Administration (GSA), 18F 디지털 서비스 팀
- **문서**: github.com/18F/api-standards, github.com/GSA/api-standards (18F를 포크)
- **최초 발표**: 2014년 (18F), GSA가 수용·발전
- **법적 구속력**: 권장(advisory) — 연방기관 자율 채택. White House API Standards에서 영향받음

#### 아키텍처
- "Support JSON and only JSON" — JSON 전용 응답 원칙
- 응답은 JSON 객체(object)로 반환 (배열 최상위 응답 금지 — 메타데이터 확장 불가)
- JSONP 금지 — 보안 및 성능 문제
- OpenAPI 스펙: 권장 (필수는 아님)

#### 보안 및 인증
- HTTPS 필수, TLS/SSL 사용, HSTS 적용 권장
- API Key 사용 시 HTTP 헤더 전달 권장 (URL 파라미터 노출 지양)
- OAuth 2.0: 권장이나 강제하지 않음 — 기관 판단
- API Key: 기관에 따라 활용 가능, 단 URL 파라미터 방식 지양

#### HTTP 상태코드 및 에러 처리
- 표준 HTTP 상태코드 사용: 200, 201, 400, 401, 403, 404, 500 등
- 에러 응답 구조화 권장 — 구체적 형식 규정은 없음
- 사람이 읽을 수 있는 에러 메시지 포함 권장

#### 페이지네이션 및 데이터 표현
- 페이지네이션 세 가지 방식 모두 허용:
  - `page` + `per_page` (직관적)
  - `offset` + `limit` (SQL 유사, 안정적 퍼머링크)
  - `since` + `limit` (타임스탬프/ID 기준, 동기화에 유리)
- 응답에 페이지네이션 메타데이터 포함 권장: `count`, `page`, `per_page`

#### 버저닝
- URL path 방식: `/v1/`, `/v2/` 권장
- 버전 번호는 정수(integer)로 단순화

#### 운영 요건
- CORS 지원 권장 — 웹 브라우저 클라이언트 허용
- Rate limiting: 권장 (구체적 헤더 규격은 기관 자율)
- 응답 시간 및 가용성 목표 설정 권장

---

### 2.3 호주 — National API Design Standards (NAPIDS)

- **관할**: Digital Transformation Agency (DTA) + 호주 국세청(ATO) 공동 주도
- **문서**: api.gov.au (베타 서비스), github.com/apigovau/national-api-design-standards
- **특이사항**: 연방정부 + 전 주(state)·준주(territory) 정부 공동 작성·승인한 범국가 표준
- **법적 구속력**: 계층적 적용
  - Experience Level API: **필수(MUST)** 적용
  - Process Level API: 권장(SHOULD) 적용
  - System Level API: 권고(RECOMMENDED) 적용

#### 아키텍처
- REST 아키텍처 기반
- JSON 기본 콘텐츠 타입: `application/json` — 권장 기본값
- OpenAPI v2.0(Swagger) 이상 명세 필수: "All APIs MUST specify a valid OpenAPI document"
  - 최대 지원범위 확보를 위해 v2.0 최소 요구, v3.0 권장

#### 보안 및 인증
- HTTPS 필수: TLS 1.2 이상 (ALL transport MUST occur over HTTPS)
- OAuth 2.0 필수: "All APIs MUST support OAuth 2.0 protocol"
  - Bearer 토큰(JWT): `Authorization: Bearer` 헤더 사용 필수
  - Basic/Digest 인증: 사용 금지(SHOULD NOT)
- OpenID Connect (OIDC): Trusted Digital Identity Framework (TDIF) 기반 SSO 연동
- API Key: 클라이언트 신원 확인 보조 수단으로 허용 (OAuth 병행)
- CORS: 허용하되 와일드카드(`*`) URL 사용 금지 — 공개 리소스 제외
- API Gateway 통한 접근 통제 권장

#### HTTP 상태코드 및 에러 처리
- 표준 HTTP 상태코드 사용 필수
- 400 Bad Request는 검증 오류에 너무 포괄적 — 응답 바디에 세부 오류 추가 필수
- 에러 객체는 반드시 배열(collection) 형태로 반환
- 구조화된 에러 응답: `id`, `code`, `title`, `detail`, `source` 필드 권장

#### 페이지네이션 및 데이터 표현
- 대용량 데이터셋 페이지네이션 필수
- 페이지네이션 링크는 `links` 객체에 포함 필수 (MUST)
- 총 페이로드 크기 10MB 초과 금지 (MUST NOT exceed 10 MB)
- 날짜/시간: ISO 8601 형식

#### 버저닝
- 모든 API 버저닝 필수 (all APIs MUST be versioned)
- 요청 시 버전 명시 필수
- 주요 버전마다 별도 OpenAPI 명세 제공 (v1, v2, v3 각각)
- 소규모/패치 버전은 문서화로 관리, 끝-of-life 정책 명시

#### 운영 요건
- Rate limiting: 권장 (API Gateway 기능 활용)
- 명명 규칙(naming conventions) 표준화: 17개 섹션으로 세분화된 상세 가이드

---

### 2.4 EU — INSPIRE, EIF, 고가치 데이터셋(HVD) 규정

- **관할**: 유럽 집행위원회(European Commission), INSPIRE MIF, ISA² 프로그램
- **주요 문서**:
  - INSPIRE Implementing Rules for Download Services (지리정보 중심)
  - European Interoperability Framework (EIF) v2.0 (2017)
  - EU 오픈데이터 지침 2019/1024, HVD Implementing Regulation 2023/138
- **법적 구속력**: 규정(Regulation) 수준 — EU 회원국 의무 이행 필요

#### 아키텍처
- OGC API – Features: INSPIRE 다운로드 서비스의 신규 표준으로 채택 (RESTful 코어)
  - 기존 WFS 2.0, ATOM, WCS, SOS 대비 개발자 친화적
  - OpenAPI 컴포넌트 기반, JSON/GeoJSON 응답 지원
- JSON/GeoJSON: 권장 인코딩 — HTML과 함께 필수 지원
- GML(XML): 기존 호환성을 위해 선택적 지원 허용
- DCAT-AP: 데이터셋 메타데이터 표준 — API.gov 카탈로그 공개 필수
- OpenAPI: data.europa.eu 등 EU 데이터 포털 전체 API에 OpenAPI 문서화 적용

#### 고가치 데이터셋(HVD) API 의무화
- EU 규정(2023/138): 2024년 6월부터 27개 회원국 의무 적용
- 고가치 데이터셋은 반드시 API를 통해 제공
- 요건: 무상 제공, 기계판독 가능 형식, 메타데이터 포함, 재사용 친화적 라이선스
- API를 통한 접근 + 벌크 다운로드 병행 제공

#### 보안 및 인증
- 명확한 표준 인증 방식: INSPIRE 규격에는 구체적 방식 미지정 (회원국 재량)
- EIF: 개인정보 보호, 보안, 접근 통제 원칙 권고

#### 운영 요건
- INSPIRE 포털 등록 및 메타데이터 DCAT-AP 형식으로 게시 의무
- European Interoperability Framework (47개 권고사항 포함): 재사용 가능한 IT 솔루션, API, 표준 공유 강조
- 회원국은 국가 상호운용성 프레임워크(NIF)를 EIF에 맞춰 조정 필요

---

### 2.5 프랑스 — DINUM API 가이드라인

- **관할**: Direction Interministérielle du Numérique (DINUM) — 디지털 혁신 부처간 총괄 조직
- **문서**:
  - numerique.gouv.fr — 데이터 공유 API 권고 프레임워크 (Cadre de recommandations)
  - api.gouv.fr — 정부 API 카탈로그 및 가이드 (Doctrine des API)
- **법적 구속력**: 권장(권고) — 부처간 AMDAC(데이터·알고리즘·소스코드 담당관)과 공동 작성

#### 아키텍처
- REST JSON 아키텍처: "가장 널리 알려진 동기 API 표준" — 사실상 기본 표준
- 비동기 API: AsyncAPI 원칙 권장
- "Contract First" 개발 방식 권장: 코드 구현 이전 API 명세 먼저 확정
  - 다수 팀 병렬 개발 지원, 명세 안정화
- OpenAPI 표준 명세: 2022년 기준 가장 보편적인 명세 방식으로 권장
- api.gouv.fr 카탈로그에 API 등록 권장 (정부 API 검색 포털)

#### 보안 및 인증
- 공공 인증 체계 활용 권장:
  - 자연인: FranceConnect, AgentConnect, EduConnect (OAuth 2.0/OIDC 기반)
  - 법인: ProConnect
- 제한적 접근 API: 재사용자(행정기관, 개발사, 기업) 신청 후 API 접근 승인 절차
- OAuth 2.0: 공식 인증 체계인 FranceConnect 등이 OAuth 2.0/OIDC 기반이므로 실질적 표준
- Rate limiting, 에러 포맷 규격: 중앙 통합 명세 없음 — 각 API별 자율 구현

#### 버저닝
- 인터페이스 유효 기간 명시 권장
- 단기 유효 버전(1~2개월): 애자일 개발 또는 빈번한 변경이 예상되는 경우
- URL path 버저닝 일반적으로 사용

#### 거버넌스 프레임워크
- DINUM이 식별한 데이터 6대 과제: 발견 가능성, 접근, 활용, 서비스 품질, 큐레이션, 경제 모델
- 수명 주기 관리: 인터페이스 유효성, 버전 명시, 서비스 수준 계약(SLA) 포함 권장

---

### 2.6 한국 — 현황

- **통합 API 표준**: 없음 — 조사 대상 6개국/지역 중 유일하게 정부 공식 API 디자인 표준이 부재
- **data.go.kr 가이드**: 비공식, 포털 활용법 수준의 사용 안내
- **NIA(한국지능정보사회진흥원) 기술 가이드**: 존재하나 구속력 없음, 내용 미흡
- **행정안전부 표준**: 공유서비스(OpenAPI) 개발 가이드라인 존재하나 구체성 및 현대성 부족

#### 현재 공공데이터 API의 실태 (개발자 현장 보고 기반)

**1. HTTP 상태코드 오용**
- 오류 발생 시에도 HTTP 상태코드 200을 반환
- 실제 오류 내용은 응답 바디에 포함 — 클라이언트가 바디를 항상 파싱해야 구분 가능
- 국제 표준(4xx, 5xx 활용)과 정반대 동작

**2. XML 기본값 문제**
- API 기본 응답 형식이 XML — JSON은 선택 파라미터(`Type=json`)로 변경 가능
- 게이트웨이 오류 발생 시 JSON 지원 API도 강제 XML 반환
- 문서 형식: `.hwp` 파일(한글 워드프로세서)로 배포되는 경우 다수

**3. API 키 URL 노출**
- 인증키를 URL 파라미터로 전달: `?serviceKey=xxxx`
- URL 노출에 따른 보안 취약점, 서버 로그·브라우저 히스토리에 키 기록
- 국제 표준(HTTP 헤더 전달)과 반대 방식

**4. OpenAPI 명세 부재**
- 표준화된 API 명세(OpenAPI/Swagger) 제공하지 않음
- 파라미터 설명 부족으로 개발자 혼선
- 샘플코드 오류 및 문서 불일치 빈번

**5. CORS 미지원**
- 공공데이터 API 상당수에서 CORS 헤더 미지원
- 브라우저 기반 웹 애플리케이션에서 직접 호출 불가
- 별도 프록시 서버 구축 필요

**6. Rate Limiting 부재**
- 트래픽 제어 헤더 없음
- 과부하 시 서비스 불안정, 클라이언트에 제한 상태 미통보

**7. 샌드박스 환경 없음**
- 개발 및 테스트용 별도 환경 미제공
- 실서버 직접 연동 테스트만 가능

---

## 3. 항목별 대조표

| 항목 | 영국 GDS | 미국 GSA/18F | 호주 DTA | EU INSPIRE/HVD | 프랑스 DINUM | 한국 현황 |
|------|---------|------------|---------|--------------|------------|---------|
| **통합 API 표준 존재** | O (의무) | O (권고) | O (의무/권고) | O (규정) | O (권고) | **X (없음)** |
| **법적 구속력** | 사실상 필수 | 기관 자율 권장 | 계층적 필수/권장 | EU 규정 의무 | 권고 | 없음 |
| **JSON 기본** | 필수 | 필수 | 필수 | 권장 | 필수 | XML 기본 |
| **XML 허용 여부** | 레거시만 예외 | 금지 | 비권장 | 선택 허용 | 비권장 | **기본값** |
| **HTTPS** | 필수 | 필수 | 필수 (TLS 1.2+) | - | 권장 | 대부분 지원 |
| **인증 방식** | OAuth 2.0 필수 | API Key(헤더) 또는 OAuth | OAuth 2.0 필수 | 회원국 재량 | OAuth/FranceConnect 권장 | **API Key URL 파라미터** |
| **API Key in URL** | 금지 | 금지 | 금지(헤더 사용) | - | 비권장 | **기본 방식** |
| **OpenAPI 명세** | 필수 | 권장 | 필수 (MUST) | 필수 (데이터 포털) | 권장 | **없음** |
| **HTTP 상태코드** | 표준 필수 | 표준 필수 | 표준 필수 | 표준 권장 | 표준 권장 | **항상 200** |
| **CORS** | 필수 | 권장 | 권장 (와일드카드 금지) | - | 권장 | **미지원** |
| **Rate Limit 헤더** | 필수 | 권장 | 권장 | - | 권장 | **없음** |
| **샌드박스 환경** | 필수 | 일부 기관 | 권장 | - | 일부 | **없음** |
| **에러 응답 포맷** | RFC 7807 권장 | 구조화 권장 | 구조화 필수 | - | 구조화 권장 | **비표준(XML/문자열)** |
| **버저닝 방식** | URL path 권장 | URL path 권장 | 버저닝 필수 | - | URL path 일반적 | **없음** |
| **페이지네이션** | cursor-based 권장 | 3가지 방식 허용 | links 객체 필수 | - | - | 규격 없음 |
| **API 카탈로그** | api.gov.uk 등록 필수 | api.data.gov | api.gov.au | data.europa.eu | api.gouv.fr | data.go.kr(비공식) |
| **문서 형식** | 공개 웹 | GitHub 공개 | GitHub 공개 | EU 포털 공개 | 공개 웹 | **HWP 파일** |

---

## 4. 한국 도입 우선순위 제안

### 4.1 즉시 도입 가능 (0~6개월) — 기술 비용 최소, 파급 효과 최대

| 순위 | 항목 | 현재 상태 | 목표 | 비고 |
|-----|------|---------|------|------|
| 1 | HTTP 상태코드 정상 사용 | 오류에도 200 반환 | 4xx/5xx 표준 사용 | 게이트웨이 설정 변경 |
| 2 | JSON 기본 응답 의무화 | XML 기본값 | JSON 기본, XML 선택 | 콘텐츠 협상 지원 |
| 3 | CORS 헤더 추가 | 미지원 | `Access-Control-Allow-Origin` 적용 | 웹 개발자 편의 즉시 향상 |
| 4 | X-RateLimit 헤더 추가 | 없음 | `X-RateLimit-Limit/Remaining` | 클라이언트 트래픽 관리 |

### 4.2 중기 도입 (6개월~2년) — 구조적 개선 필요

| 순위 | 항목 | 현재 상태 | 목표 | 비고 |
|-----|------|---------|------|------|
| 5 | API Key를 HTTP 헤더로 이동 | URL 파라미터 노출 | `Authorization` 헤더 전달 | 보안 핵심 개선 |
| 6 | OpenAPI 3.0 명세 의무화 | 없음 (HWP 문서) | OAS 3.0 명세 필수 제공 | 표준화 기반 마련 |
| 7 | 에러 응답 RFC 7807 표준화 | 비표준 문자열/XML | Problem Details 구조체 | 개발자 경험 개선 |
| 8 | 샌드박스 환경 제공 | 없음 | 개발/테스트 전용 환경 | 연동 테스트 지원 |

### 4.3 장기 도입 (2~3년) — 제도·인프라 변화 필요

| 순위 | 항목 | 현재 상태 | 목표 | 비고 |
|-----|------|---------|------|------|
| 9 | OAuth 2.0 전환 | API Key URL 방식 | OAuth 2.0 + OIDC | 공공 아이디 체계와 연동 |
| 10 | 통합 API Gateway 도입 | 기관별 분산 | 중앙/연합형 게이트웨이 | 일관성 있는 정책 적용 |
| 11 | API 버저닝 표준화 | 없음 | URL path (`/v1/`) 방식 | 하위 호환성 보장 |
| 12 | 정부 API 디자인 표준 고시 | 없음 | 행정안전부 고시 또는 기술규격 | 제도적 뒷받침 |

---

## 5. 결론

### 핵심 발견

1. **한국은 조사 대상 6개국/지역 중 유일하게 정부 통합 API 디자인 표준이 없는 국가다.**
   - 영국, 미국, 호주는 10년 이상 지속적으로 발전시킨 표준 체계를 보유
   - EU는 규정 수준의 의무화가 진행 중
   - 프랑스조차 권고 수준이지만 DINUM이 주도하는 명확한 프레임워크 존재

2. **한국 공공 API의 실태는 2024년 기준 국제 표준과 10년 이상 격차가 있다.**
   - 미국 18F가 2014년에 "API Key를 URL에 넣지 말라"고 표준화한 내용이 한국에서 아직도 기본 방식
   - HTTP 상태코드 오용(항상 200), XML 기본값, CORS 미지원은 현대 웹 개발 생태계와 충돌

3. **가장 참고할 모델: 영국 GDS**
   - 가장 구체적이고 엄격한 요건 (OAuth 2.0 필수, RFC 7807, 샌드박스 필수 등)
   - 중앙정부 대상 사실상 의무화 모델
   - OpenAPI 3.0, 커서 기반 페이지네이션 등 현대 표준 선도

4. **한국의 현실적 첫 걸음**
   - 단계 1: 미국 GSA 수준의 '권장 가이드라인' 발표 (6개월 내 달성 가능)
   - 단계 2: 호주 DTA 수준의 계층적 의무화 (Experience API 필수 적용) 도입
   - 단계 3: 영국 GDS 수준의 전면 의무화 및 API 거버넌스 체계 구축

### 즉각 실행 권고

행정안전부 또는 NIA는 최소한 다음 4가지를 즉시 가이드라인으로 발표해야 한다:

1. **HTTP 상태코드 표준 준수** (오류에 4xx/5xx 사용)
2. **JSON 기본 응답 방식 전환** (XML 선택 허용 유지)
3. **API Key를 HTTP 헤더로 전달** (URL 파라미터 방식 중단)
4. **OpenAPI 3.0 명세 제공** (HWP 문서 폐지)

이 4가지만 이행해도 한국 공공 API는 미국 18F 2014년 표준 수준에 도달한다.

---

## 참고 문헌 및 출처

- [UK GDS API Technical and Data Standards](https://www.gov.uk/guidance/gds-api-technical-and-data-standards)
- [UK GDS API Standards v1 2018](https://www.gov.uk/government/publications/api-technical-and-data-standards-v1-2018/gds-api-technical-and-data-standards-v1-2018)
- [18F API Standards (GitHub)](https://github.com/18F/api-standards)
- [GSA API Standards (GitHub)](https://github.com/GSA/api-standards)
- [Australia National API Design Standards (GitHub)](https://github.com/apigovau/national-api-design-standards)
- [Australia api.gov.au](https://api.gov.au)
- [Australia api.gov.au — API Security](https://api.gov.au/sections/api-security.html)
- [INSPIRE OGC API Features Good Practice](https://inspire.ec.europa.eu/good-practice/ogc-api-%E2%80%93-features-inspire-download-service)
- [EU DCAT-AP for High Value Datasets](https://semiceu.github.io/DCAT-AP/releases/2.2.0-hvd/)
- [European Interoperability Framework (EIF)](https://ec.europa.eu/isa2/eif_en/)
- [France DINUM API doctrine — numerique.gouv.fr](https://www.numerique.gouv.fr/offre-accompagnement/reference-recommandations-partage-donnees-api-administration/)
- [France api.gouv.fr — Doctrine des API](https://api.gouv.fr/guides/doctrine-api)
- [Australia's API Design Standard — Salsa Digital 분석](https://salsa.digital/insights/australias-api-design-standard)
- [공공데이터포털 OpenAPI 문제점 — kdev.ing](https://kdev.ing/data-go-openapi/)
- [한국 공공데이터포털](https://www.data.go.kr/)
- [행정안전부 공유서비스 OpenAPI 개발 가이드라인](https://www.mois.go.kr/frt/bbs/type001/commonSelectBoardArticle.do?bbsId=BBSMSTR_000000000045&nttId=34426)
- [UKHSA API Design Guidelines Summary](https://ukhsa-collaboration.github.io/standards-org/api-design-guidelines/api-guidelines-summary/)
