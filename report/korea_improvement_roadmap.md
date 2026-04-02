# 한국 공공 API 개선 로드맵 — 해외 사례 기반

> **작성일**: 2026년 4월 3일  
> **분석 기반**: 미국·영국·싱가포르·프랑스·일본·호주·EU 7개국/지역 API 생태계 실증 조사  

---

## 요약

8개국(한국 포함) 분석 결과, 한국은 OECD 공공데이터 개방 지수 세계 1~2위이나 API 기술 품질 및 AI 적합성은 조사 대상 9개국 중 최하위(AI 적합성 점수 2.0/10, 해외 평균 7.4)다. 문제는 데이터의 양이 아니라 데이터를 쓸 수 있게 만드는 기술 품질에 있다.

이 로드맵은 7개국의 성공 사례에서 한국이 즉시·중기·장기로 도입 가능한 개선안을 구체적 근거와 함께 제시한다. Phase 1의 4개 개선안은 6개월 내 예산 최소 수준으로 실행 가능하며, 이것만으로도 AI 적합성 등급을 D→C로 끌어올릴 수 있다.

### 한국의 현재 위치 — 핵심 수치

| 지표 | 한국 현황 | 해외 기준 |
|------|----------|----------|
| 총 포털 수 | 173개 (파편화) | 1~12개 (통합/연합형) |
| 전체 커버 시 필요 ID | 130개 | 1~3개 |
| data.go.kr 단독 완전 동작 | 18% (31개 포털) | 80~100% |
| AI 적합성 등급 A~B | 1.2% (2개 포털) | 60~100% |
| HTTP 상태코드 표준 준수 | 0% (모두 200 반환) | 100% |
| JSON 기본 응답 | 약 10% | 100% |
| OpenAPI 스펙 제공 | 약 0% | 50~100% |
| CORS 지원 | 소수 | 100% |
| 정부 통합 API 표준 | 없음 | 영국·미국·호주·EU·프랑스 모두 보유 |

---

## Phase 1: 즉시 실행 (6개월, 예산 최소)

Phase 1은 서버 설정 변경 수준에서 실행 가능한 항목만 담았다. 법령 개정, 예산 신청, 기관 협의 없이 data.go.kr 운영팀 단독으로 착수할 수 있다.

---

### 1.1 JSON 기본 응답 전환

**벤치마크**: 미국 GSA/18F — "Support JSON and only JSON" (2014년 표준화)

**현황과 문제**  
한국 공공 API의 약 59%가 XML을 기본 응답으로 반환한다. JSON을 받으려면 `?Type=json` 파라미터를 명시해야 하며, API 게이트웨이 오류 발생 시 JSON 지원 API도 강제로 XML을 반환한다. AI 언어모델에서 XML은 JSON 대비 2.5배 토큰을 소모하여 직접적인 비용 증가로 이어진다. 기상청 실시간 데이터를 LLM에 공급할 경우 JSON 전환만으로 토큰 비용의 60%를 절감할 수 있다.

**미국의 사례**  
18F는 2014년 이미 JSON 전용을 표준으로 확정했다. api.data.gov, USAspending.gov, NASA API, Treasury Fiscal Data API 등 450개 이상의 연방 API 전체가 JSON 기본이다. XML 응답은 `Accept` 헤더를 통한 콘텐츠 협상으로만 제공하거나, 레거시 호환을 위한 극히 예외적 경우에만 허용한다.

**실행 방안**
- data.go.kr 신규 등록 API: JSON 필수, XML 선택(Accept 헤더)으로 정책 전환
- 기존 API: `Type=json`을 URL 기본값으로 변경 (서버 설정 1줄 변경)
- 게이트웨이 오류 응답: XML 대신 JSON으로 반환하도록 nginx/API 게이트웨이 설정 수정
- 문서: JSON 파라미터 없이도 JSON이 기본임을 명시한 공지 발행

**예상 효과**: LLM 토큰 비용 60% 절감, AI 에이전트 연동 가능 API 즉시 5배 증가  
**난이도**: 하 (서버 설정 변경 수준)  
**하위 호환성**: XML 파라미터(`Type=xml`) 병행 유지로 기존 사용자 영향 최소화

---

### 1.2 HTTP 상태코드 정상화 — 최우선 과제

**벤치마크**: 영국 GDS, 미국 GSA, 싱가포르 GovTech, 프랑스 DINUM, 일본 デジタル庁 — 조사 대상 전 국가

**현황과 문제**  
한국 공공 API는 인증 실패, 파라미터 오류, 존재하지 않는 리소스 등 모든 오류 상황에서 HTTP 200 OK를 반환하고 실제 오류 내용을 응답 바디에 담는다. 이는 국제 표준과 정반대 동작이다.

```
// 한국 공공 API 실제 동작 (문제)
HTTP/1.1 200 OK
{"resultCode":"03","resultMsg":"SERVICE KEY IS NOT REGISTERED"}

// 국제 표준 (목표)
HTTP/1.1 401 Unauthorized
{"type":"about:errors/auth","title":"Unauthorized","status":401,"detail":"API key not found"}
```

이 문제로 인해 AI 에이전트가 성공과 실패를 자동으로 구분할 수 없다. HTTP 상태코드를 보고 분기하는 표준 HTTP 클라이언트 라이브러리(axios, requests, fetch API)가 모두 무용지물이 된다. 개발자는 모든 응답을 파싱하여 바디의 에러 코드를 별도로 확인해야 한다.

**해외 사례**  
영국 GDS는 200, 201, 204, 400, 401, 403, 404, 409, 429, 500 사용을 필수로 지정했다. 미국은 2014년부터 표준 HTTP 상태코드 사용을 정부 API 표준에 명시했다. 조사한 7개국에서 한국처럼 에러에 200을 반환하는 나라는 단 한 곳도 없었다.

**실행 방안 — 최소한 3개 상태코드 즉시 도입**

| 코드 | 상황 | 현재 동작 | 목표 |
|------|------|----------|------|
| 401 | 인증키 없음/만료 | 200 + 에러바디 | 401 반환 |
| 404 | 조회 결과 없음 | 200 + 빈 배열 | 404 반환 (항목 부재) |
| 429 | 호출 한도 초과 | 200 + 에러바디 | 429 반환 |
| 400 | 필수 파라미터 누락 | 200 + 에러바디 | 400 반환 |
| 500 | 서버 오류 | 200 + 에러바디 | 500 반환 |

**하위 호환성**: 기존 응답 바디의 `resultCode`, `resultMsg` 필드는 그대로 유지하되, HTTP 상태코드를 추가로 올바르게 설정한다. 기존 클라이언트는 바디를 계속 파싱하면 되므로 중단 없이 이행 가능하다.

**예상 효과**: AI 에이전트 자동 에러 복구, 표준 HTTP 라이브러리 즉시 활용 가능  
**난이도**: 하~중 (게이트웨이 설정 + 개별 API 응답 로직 수정 필요)

---

### 1.3 CORS 헤더 추가

**벤치마크**: 영국 GDS — CORS 지원 필수 명시, 미국 GSA — 권장, 싱가포르 data.gov.sg — 기본 지원

**현황과 문제**  
대부분의 한국 공공 API는 CORS(Cross-Origin Resource Sharing) 헤더를 제공하지 않는다. 이는 웹 브라우저에서 JavaScript로 직접 공공 API를 호출할 수 없다는 의미다. 개발자는 서버 측 프록시를 별도 구축해야 하며, 이는 인프라 비용과 유지 관리 부담을 유발한다. 웹 기반 AI 챗봇이나 브라우저 확장 프로그램에서 공공 데이터를 활용하는 것이 원천 불가능하다.

**영국 GDS의 기준**  
GDS API 표준은 "CORS must be supported to allow browser-based clients"를 명시적으로 요구한다. 싱가포르 data.gov.sg는 모든 공공 API에 CORS 헤더를 기본 제공하며, 이를 통해 브라우저에서 즉시 API를 테스트하고 웹 앱에서 직접 활용할 수 있다.

**실행 방안**

```nginx
# nginx 설정 1줄 추가 (공개 API의 경우)
add_header 'Access-Control-Allow-Origin' '*';
add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS';
add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
```

data.go.kr의 모든 공개 API(인증 후 조회 가능한 API 포함)에 CORS 헤더를 추가한다. 민감한 개인정보 API는 특정 도메인만 허용하는 방식으로 세분화 적용 가능하다.

**예상 효과**: 웹 프론트엔드 직접 호출, 프록시 서버 불필요, 브라우저 기반 AI 앱 즉시 연동  
**난이도**: 하 (nginx/API 게이트웨이 설정 변경, 코드 수정 불필요)

---

### 1.4 X-RateLimit 헤더 추가

**벤치마크**: 영국 GDS — 필수(`X-RateLimit-Limit`, `X-RateLimit-Remaining`), 미국 NASA·api.data.gov — 표준 지원

**현황과 문제**  
한국 공공 API는 호출 한도(일 1,000회, 일 10,000회 등)를 관리하지만 응답 헤더로 현재 잔여량을 알려주지 않는다. 개발자와 AI 에이전트는 한도 초과 에러를 받기 전까지 사용량을 알 수 없다. 특히 AI 에이전트가 공공 API를 반복 호출할 때, 한도를 초과해도 에러 응답이 200으로 오므로 무한 반복 호출이 발생하는 구조적 문제가 있다.

**미국 NASA의 사례**  
NASA API는 모든 응답에 다음 헤더를 포함한다:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 874
X-RateLimit-Reset: 1711929600
```
이를 통해 API 클라이언트는 잔여 한도에 따라 호출 속도를 자동으로 조절한다. api.data.gov 게이트웨이는 산하 450개 이상 API 전체에 이 헤더를 자동으로 추가한다.

**실행 방안**  
API 게이트웨이(또는 nginx) 레이어에서 헤더를 삽입한다. 개별 API 서버 수정 없이 게이트웨이 단에서 전체 일괄 적용 가능하다.

```
X-RateLimit-Limit: {기관별 일일 한도}
X-RateLimit-Remaining: {잔여 호출 수}
X-RateLimit-Reset: {한도 초기화 Unix 타임스탬프}
```

**예상 효과**: AI 에이전트 자동 속도 조절, 개발자 호출량 선제 관리, 한도 초과 장애 예방  
**난이도**: 하 (게이트웨이 플러그인/설정 추가)

---

### Phase 1 요약 — 6개월 실행 계획

| 순번 | 개선안 | 담당 | 기간 | 예산 |
|------|-------|------|------|------|
| 1 | JSON 기본 전환 | data.go.kr 기술팀 | 1개월 | 없음 |
| 2 | HTTP 상태코드 정상화 (401/404/429) | data.go.kr 기술팀 | 2개월 | 없음 |
| 3 | CORS 헤더 추가 | data.go.kr 기술팀 | 1주 | 없음 |
| 4 | X-RateLimit 헤더 추가 | data.go.kr 기술팀 | 1주 | 없음 |

Phase 1 완료 시 AI 적합성 점수: **2.0 → 4.5/10** (추정, 현재 D등급 → C등급)

---

## Phase 2: 중기 개선 (1~2년)

Phase 2는 정책 결정 및 기관 협의가 필요하지만, 기술적으로는 검증된 방식을 채택하는 개선 항목이다.

---

### 2.1 정부 API 디자인 표준 수립

**벤치마크**: 영국 GDS API Technical and Data Standards (2016년 초판, 2024년 7월 최신 개정), 호주 DTA National API Design Standards (NAPIDS)

**현황과 문제**  
한국은 조사 대상 6개국/지역 중 유일하게 정부 통합 API 디자인 표준이 없다. 행정안전부의 "공유서비스 OpenAPI 개발 가이드라인"은 존재하지만 구체성이 부족하고 현대 표준을 반영하지 않는다. NIA 기술 가이드는 구속력이 없고 내용이 미흡하다.

표준 부재의 직접적 결과: 173개 포털이 각자 다른 인증 방식, 다른 에러 포맷, 다른 파라미터 명명 규칙을 채택하여 개발자가 포털마다 완전히 새로 학습해야 한다.

**영국 GDS 모델**  
영국 GDS는 2016년 API 표준을 발표하고 2024년까지 지속 개정했다. 핵심 내용:
- JSON 응답 필수, XML 레거시만 예외
- OpenAPI 3.0 명세 필수 (Open Standards Board 승인)
- OAuth 2.0 필수 (API Key 명시적 비권장)
- RFC 7807 에러 응답 형식 권장
- Rate limit 헤더 필수
- 샌드박스 환경 필수
- 중앙정부 대상 사실상 의무화 — 미준수 시 서비스 승인 불가

**호주 DTA NAPIDS 모델**  
호주는 연방정부와 전 주·준주 정부가 공동 작성·승인한 범국가 표준을 GitHub(github.com/apigovau/national-api-design-standards)에 공개하고 커뮤니티 기여를 받는다. 계층적 적용: 사용자 대면 API는 필수(MUST), 내부 시스템 API는 권고(RECOMMENDED).

**실행 방안**  
행정안전부 또는 NIA 주관으로 "한국 정부 API 표준"을 제정한다.

*단계별 접근:*
1. **6개월**: 미국 GSA 수준의 '권장 가이드라인' 발표 — JSON, HTTP 상태코드, OpenAPI 스펙, CORS, Rate Limit 헤더 5개 항목 필수화 권고
2. **1년**: 호주 NAPIDS 수준 — 신규 API에 대한 계층적 의무화 도입
3. **2년**: 영국 GDS 수준 — 전면 의무화 및 API 거버넌스 체계 구축

**표준 문서 공개 방식**: GitHub 공개 + 커뮤니티 이슈/PR 수용 (호주 방식). PDF/HWP 내부 문서 방식 지양.

**포함할 최소 항목**:
- JSON 필수, XML 선택
- HTTP 상태코드 표준 준수
- URL 방식 버저닝 (`/v1/`, `/v2/`)
- API Key 헤더 전달 (URL 파라미터 금지)
- OpenAPI 3.0 스펙 동시 제출
- 에러 응답 구조체 정의
- 페이지네이션 형식 (cursor 또는 offset+limit)
- Rate Limit 헤더

---

### 2.2 OpenAPI 3.0 스펙 의무화

**벤치마크**: 영국 GDS — 필수(Open Standards Board 승인), EU INSPIRE/HVD — 필수, 호주 NAPIDS — 필수("All APIs MUST specify a valid OpenAPI document")

**현황과 문제**  
한국 공공 API 문서는 HWP 파일 또는 포털 페이지의 텍스트 설명으로 제공된다. 기계가 읽을 수 없는 형식이다. 이로 인해 AI 에이전트 연동을 위해 개발자가 수동으로 API 스키마를 작성해야 하며, 이것이 한국 공공 API의 MCP 서버 커버리지가 5% 미만에 머무는 핵심 이유다.

**OpenAPI 스펙이 해결하는 문제**  
OpenAPI 3.0 스펙 파일 하나로:
- MCP(Model Context Protocol) 서버를 자동 생성 가능
- LLM function calling tool로 즉시 등록 가능
- Swagger UI / Redoc으로 인터랙티브 문서 자동 제공
- Python/JavaScript/Java 클라이언트 SDK 자동 생성
- API 테스트 자동화 가능

영국 GDS 표준에서 OpenAPI 3.0이 채택된 이후, 영국 정부 API의 AI 활용 사례가 폭발적으로 증가했다. NWS(미국 국립기상서비스)의 경우 OpenAPI 스펙을 제공한 이후 제3자 MCP 서버 및 LLM 통합이 자동으로 구현됐다.

**실행 방안**
- data.go.kr 신규 API 등록 시: OpenAPI 3.0 YAML/JSON 스펙 파일 필수 제출
- 기존 API: 2년 내 전환 계획 수립 및 단계적 전환 (우선순위: 호출 횟수 상위 100개)
- data.go.kr 포털: 각 API 상세 페이지에 Swagger UI 기반 인터랙티브 테스트 콘솔 내장
- NIA: 기관용 "OpenAPI 스펙 작성 가이드" 및 자동 변환 도구 제공

---

### 2.3 API Key 헤더 전환 (URL 노출 금지)

**벤치마크**: 영국 GDS — URL API Key 명시적 금지, 미국 GSA — "HTTP 헤더 전달 권장, URL 파라미터 지양", 일본 RESAS — `X-API-KEY` 헤더 사용

**현황과 문제**  
한국 공공 API의 표준 인증 방식은 `?serviceKey=xxxx` URL 파라미터다. URL에 인증키를 포함시키면:
- 서버 접근 로그에 API 키가 그대로 기록됨
- 브라우저 히스토리, 즐겨찾기, Referrer 헤더를 통해 키가 노출됨
- HTTPS를 사용해도 URL은 로그에 평문 기록 가능
- 복사한 URL을 공유하면 키도 함께 노출됨

미국 18F는 2014년 이미 "API Key를 URL에 넣지 말라"를 표준화했다. 12년이 지난 2026년에 한국만 아직 URL 방식을 기본으로 사용하고 있다.

**실행 방안**
- 신규 API: `Authorization: Bearer {key}` 또는 `X-Service-Key: {key}` 헤더 방식 필수
- 기존 API: URL 파라미터 방식은 Deprecated 마킹, 헤더 방식 병행 지원 후 2년 후 URL 방식 종료
- data.go.kr 개발자 문서: 헤더 방식 예제 코드로 전면 교체

---

### 2.4 통합 인증 체계 — API SSO

**벤치마크**: 프랑스 FranceConnect (OAuth 2.0/OIDC), 싱가포르 SingPass/Corppass, 미국 login.gov

**현황과 문제**  
현재 전체 포털 커버를 위해 필요한 ID가 130개다. data.go.kr만으로 완전히 동작하는 것은 18%(31개 포털)뿐이다. 나머지 82%는 원본 기관 별도 가입이 필요하다. 개발자가 실질적으로 한국 공공 API를 폭넓게 활용하려면 수십 개의 계정을 관리해야 한다.

**프랑스 FranceConnect 사례**  
DINUM이 운영하는 FranceConnect는 OAuth 2.0/OpenID Connect 기반 국가 ID 페더레이션으로, 단일 인증으로 시민 행정 서비스 전체를 접근 가능하게 한다. ProConnect(2024년 신규)는 공무원·전문직 버전으로 부처별 ID 시스템을 단일 계정으로 통합했다. api.gouv.fr의 166개 API가 단일 신청 플로우와 통합되어 있어 개발자가 API별로 별도 계정을 만들 필요가 없다.

**싱가포르 SingPass/Corppass 사례**  
SingPass는 시민용, Corppass는 기업용 단일 ID로 전 정부 서비스를 접근한다. 31개 기관의 APEX 게이트웨이 산하 2,400개 API 기능 전체가 단일 인증 체계 안에서 동작한다.

**실행 방안**  
한국은 이미 정부24 계정(공공 마이데이터 기반)과 공공 아이디 체계를 보유하고 있다. 이를 API 인증 SSO로 확장:
- 1단계: data.go.kr, KOSIS, DART, ECOS 4개 주요 포털 SSO 연동
- 2단계: 17개 지자체 포털 통합
- 3단계: 전 포털 확장 (목표: 130개 ID → 1~3개)

**예상 효과**: 필요 ID 130개 → 1개, 개발자 진입 장벽 획기적 감소

---

### 2.5 에러 응답 표준화 (RFC 7807)

**벤치마크**: 영국 GDS — RFC 7807 Problem Details 권장, 호주 NAPIDS — 구조화 에러 응답 필수

**현황과 문제**  
현재 한국 공공 API의 에러 메시지는 XML 문자열이거나 비표준 JSON 구조다. 에러 형식이 포털마다 다르고, 같은 포털 내에서도 API마다 다른 경우가 많다.

**RFC 7807 형식 (목표)**

```json
{
  "type": "https://data.go.kr/errors/authentication",
  "title": "인증 오류",
  "status": 401,
  "detail": "serviceKey가 만료되었습니다. data.go.kr에서 키를 갱신하세요.",
  "instance": "/api/v1/getAirQualityData",
  "errorCode": "SERVICE_KEY_EXPIRED",
  "renewUrl": "https://www.data.go.kr/mypage/keys"
}
```

이 형식은 AI 에이전트가 에러 원인을 자동으로 파악하고 대응 조치(키 갱신 안내, 재시도 등)를 수행할 수 있게 한다.

**실행 방안**  
표준 에러 응답 라이브러리를 NIA가 공개하고, 각 기관이 채택하도록 가이드 제공. 신규 API는 RFC 7807 형식 의무화.

---

### 2.6 샌드박스/테스트 환경 제공

**벤치마크**: 영국 GDS — 필수, 싱가포르 IRAS — 별도 샌드박스 운영(apisandbox.iras.gov.sg), 미국 openFDA — DEMO_KEY 즉시 체험

**현황과 문제**  
한국 공공 API는 개발 및 테스트용 별도 환경을 제공하지 않는다. 개발자는 실서버에서 직접 테스트해야 하며, 이는 실제 데이터에 대한 무의미한 조회를 발생시키고 호출 한도를 낭비한다.

**미국 NASA의 DEMO_KEY 모델**  
NASA API는 DEMO_KEY라는 예약 키를 통해 회원가입 없이 즉시 API를 체험할 수 있게 한다. 시간당 30회, 일 50회 제한을 두어 남용 방지와 접근성을 동시에 달성했다.

**실행 방안**  
- data.go.kr: 각 API 상세 페이지에 OpenAPI 스펙 기반 Swagger UI 테스트 콘솔 내장
- 주요 API(기상청, 부동산, 교통): 테스트용 DEMO_KEY 제공 (호출 제한 설정)
- 장기: 싱가포르처럼 Key-Free(인증 없는) 소용량 탐색 접근 허용

---

## Phase 3: 장기 비전 (2~3년)

Phase 3는 제도·인프라 전환이 필요한 중장기 항목이다. Phase 1~2 완료 후 자연스럽게 이어지는 다음 단계다.

---

### 3.1 통합 API Gateway

**벤치마크**: 미국 api.data.gov (25개 기관 450개 이상 API), 싱가포르 APEX (31개 기관, 월 1억 건)

**현황과 문제**  
현재 data.go.kr이 게이트웨이 역할을 하는 것은 전체의 18%(31개 포털)뿐이다. 82%의 포털은 독자 서버를 운영하며 data.go.kr과 연계되지 않는다. 각 기관이 Rate Limiting, 인증, 모니터링을 개별 구현하는 중복 투자가 발생한다.

**미국 api.data.gov의 아키텍처**  
GSA의 API Umbrella(오픈소스)를 기반으로, 25개 이상 기관의 API를 투명한 프록시 방식으로 통합했다. 기관은 원본 API 구조를 변경할 필요 없이 게이트웨이에 등록만 하면 된다. 통합 API 키, Rate Limiting, 사용량 분석, 응답 헤더 자동 추가를 게이트웨이가 담당한다.

**싱가포르 APEX의 성과**  
분산된 API 호스팅, 구식 설계 방법론, 일관성 없는 데이터 공유 표준을 안고 있던 싱가포르가 APEX 도입 후 월 1억 건의 API 호출을 단일 플랫폼에서 처리하는 수준에 도달했다.

**실행 방안**  
미국의 API Umbrella(오픈소스 공개, GitHub 제공)를 한국 실정에 맞게 커스터마이징하는 방식으로 초기 개발 비용 최소화. 게이트웨이 전환 목표: data.go.kr 직접 동작 18% → 80%+

---

### 3.2 OAuth 2.0 전환

**벤치마크**: 영국 GDS — OAuth 2.0 필수 (API Key 명시적 금지), 호주 NAPIDS — "All APIs MUST support OAuth 2.0", 프랑스 FranceConnect — OAuth 2.0/OIDC 기반 국가 표준

**현황과 문제**  
현재 방식인 API Key URL 파라미터는 2년마다 수동 갱신이 필요하고, URL 노출로 보안에 취약하며, 자동화 서비스에서 운영 중 갑작스러운 서비스 중단을 유발한다.

**OAuth 2.0 Client Credentials의 장점**  
서버 간 통신(machine-to-machine)에서 토큰을 자동으로 갱신하므로 운영 중단 없이 장기 서비스 운영이 가능하다. 범위(scope) 기반 접근 제어로 최소 권한 원칙을 구현할 수 있다.

**실행 방안**  
단계적 전환:
- 1단계: API Key를 Authorization 헤더로 이동 (Phase 2.3)
- 2단계: JWT 기반 단기 토큰 발급 (30분~1시간 유효)
- 3단계: OAuth 2.0 Client Credentials 완전 전환 + OIDC 연동

---

### 3.3 정부 공식 MCP 서버

**벤치마크**: 미국 GPO GovInfo MCP (2026년 1월 퍼블릭 프리뷰), 프랑스 data.gouv.fr MCP (공개 운영, Claude·ChatGPT·Gemini 호환)

**현황과 필요성**  
한국 공공 API의 MCP 커버리지는 전체의 5% 미만이다. 법령 API(korean-law-mcp)와 일부 통계 API에 대한 커뮤니티 MCP 서버가 존재하지만, 대부분 비공식이고 유지 관리 책임자가 불명확하다.

**프랑스의 선례**  
프랑스 data.gouv.fr 팀이 직접 개발·운영하는 MCP 서버(mcp.data.gouv.fr/mcp)는 세계에서 두 번째 정부 공식 MCP 사례다. 인증 없이 누구나 접근 가능하고, Claude·ChatGPT·Gemini·Mistral 등 주요 AI와 호환된다. 7개 도구를 통해 40,000개 이상 데이터셋 검색, 데이터 조회, 플랫폼 지표 확인이 가능하다.

**한국 data.go.kr MCP 서버 설계안**  

OpenAPI 스펙이 의무화되면 MCP 서버 구축 비용이 극적으로 감소한다. OpenAPI 스펙 → MCP 서버 자동 생성 도구를 활용하면 수백 개 API의 MCP 서버를 일괄 생성할 수 있다.

| 단계 | 내용 | 시점 |
|------|------|------|
| 1 | data.go.kr 데이터셋 검색·발견 MCP 서버 | Phase 2 완료 후 |
| 2 | OpenAPI 스펙 기반 상위 50개 API MCP 도구 자동 생성 | Phase 2 완료 후 |
| 3 | 실시간 데이터(기상청, 교통) MCP 서버 | Phase 3 |
| 4 | 전체 공공 API MCP 통합 서버 | 장기 |

**예상 효과**: Claude, ChatGPT, Gemini에서 자연어로 한국 공공 데이터 직접 조회 가능

---

### 3.4 Key-Free 테스트 접근 도입

**벤치마크**: 싱가포르 data.gov.sg — 모든 공개 API 키 없이 즉시 테스트, 영국 ONS — 완전 무인증 공개 API

**현황과 문제**  
"ID 없이 사용 가능한 공공 API는 사실상 없다." 모든 포털이 최소한 회원가입과 API Key 발급을 요구한다. 이는 개발자가 API를 평가해보기 전에 계정을 만들어야 하는 진입 장벽으로 작용한다.

**싱가포르 data.gov.sg의 전략**  
모든 공개 데이터셋 API는 API 키 없이 호출 가능하다. 개발자는 회원가입 없이 브라우저에서 즉시 API를 테스트할 수 있으며, 이는 진입 장벽을 사실상 제로로 만들었다. 프로덕션 사용에는 이메일 등록(무료) 후 더 높은 Rate Limit이 부여된다.

**실행 방안**  
- 개인정보가 없는 순수 공공 데이터(기상, 부동산 통계, 교통 통계, 관광 정보 등): Key-Free 소용량 접근 허용 (일 100회)
- 등록 사용자: 일 1,000~10,000회 (현행 유지)

---

## 실행 우선순위 매트릭스

| 개선안 | 효과 (1-5) | 난이도 (1-5) | 우선순위 | Phase |
|--------|-----------|------------|---------|-------|
| CORS 헤더 추가 | 4 | 1 | ★★★★★ | 1 |
| X-RateLimit 헤더 | 3 | 1 | ★★★★★ | 1 |
| JSON 기본 전환 | 5 | 2 | ★★★★★ | 1 |
| HTTP 상태코드 정상화 | 5 | 2 | ★★★★★ | 1 |
| API Key 헤더 전환 | 3 | 2 | ★★★★ | 2 |
| OpenAPI 3.0 의무화 | 5 | 3 | ★★★★ | 2 |
| 에러 응답 RFC 7807 | 3 | 2 | ★★★★ | 2 |
| 샌드박스 환경 | 3 | 3 | ★★★★ | 2 |
| API 디자인 표준 제정 | 5 | 3 | ★★★★ | 2 |
| 정부 MCP 서버 | 4 | 3 | ★★★ | 3 |
| 통합 인증 (API SSO) | 5 | 5 | ★★★ | 2~3 |
| OAuth 2.0 전환 | 3 | 4 | ★★ | 3 |
| 통합 API Gateway | 5 | 5 | ★★ | 3 |
| Key-Free 테스트 | 3 | 3 | ★★★ | 3 |

---

## 국가별 벤치마크 요약

한국이 각 국가에서 즉시 참조해야 할 핵심 사례를 정리한다.

### 미국 — 통합 게이트웨이 모델

api.data.gov는 25개 이상 기관 450개 이상 API를 단일 게이트웨이로 통합했다. 개발자는 이메일 입력만으로 즉시 API 키를 발급받고 모든 기관 API에 접근한다. 기반 소프트웨어인 API Umbrella는 오픈소스로 공개되어 한국이 즉시 도입할 수 있다.

**한국 참조 포인트**: 분산된 173개 포털을 게이트웨이로 통합하는 기술 모델, X-RateLimit 헤더 표준화 방식

### 영국 — 가장 엄격한 기술 표준

GDS API Standards는 조사한 국가 중 가장 구체적이고 엄격하다. OAuth 2.0 필수, RFC 7807 에러 포맷, Sandbox 필수, OpenAPI 3.0 필수, CORS 필수. 중앙정부 대상 사실상 의무화로 이를 따르지 않으면 서비스 승인이 나지 않는다.

**한국 참조 포인트**: API 디자인 표준 제정 시 직접 참조할 기술 기준, 샌드박스 의무화 모델

### 싱가포르 — 접근성의 극단

data.gov.sg의 Key-Free 테스트, APEX의 전 정부 단일 게이트웨이(월 1억 건). 도시국가의 이점을 살려 31개 기관 전체를 단일 인프라로 통합했다. SingPass/Corppass 단일 인증으로 기업·개인 모두 단일 ID로 전 정부 서비스 접근.

**한국 참조 포인트**: Key-Free 탐색 접근, APEX 아키텍처를 통한 게이트웨이 설계, 통합 인증 모델

### 프랑스 — G2G API와 MCP 선도

"한 번만 말하기(Dites-le-nous une fois)" 원칙으로 행정기관 간 데이터 재요청을 API로 해결. 세계 2번째 정부 공식 MCP 서버 운영. DINUM이라는 총리실 산하 범부처 디지털 사무국이 전체를 조율.

**한국 참조 포인트**: data.gouv.fr MCP 서버 구현 방식, FranceConnect 단일 인증 모델, DINUM 거버넌스 구조

### 일본 — 같은 문제, 더 빠른 개선

일본도 한국과 유사한 성청별 파편화 구조를 갖고 있었으나, e-Gov 법령API v2(2025년 3월), RESAS(`X-API-KEY` 헤더), gBizINFO(JSON 전용 + OpenAPI 스펙) 등에서 개별 개선을 선행했다. 2021년 출범한 デジタル庁(디지털청)이 표준화를 추진 중.

**한국 참조 포인트**: 파편화 구조에서의 단계적 개선 사례, 디지털청 거버넌스 모델

### 호주 — 커뮤니티 참여형 표준

연방정부와 전 주·준주 정부가 GitHub에서 공동 작성한 NAPIDS. 계층적 의무화(사용자 대면 API는 필수, 내부 API는 권고). OpenAPI 스펙 필수, OAuth 2.0 필수.

**한국 참조 포인트**: 정부 API 표준 제정 방식, GitHub 공개 + 커뮤니티 기여 모델

---

## 거버넌스 제안

### 추진 주체 및 역할

| 기관 | 역할 |
|------|------|
| **행정안전부** | 한국 정부 API 디자인 표준 고시, 부처 의무화 근거 마련 |
| **NIA(한국지능정보사회진흥원)** | 표준 초안 작성, 기관 기술 지원, OpenAPI 스펙 변환 도구 개발 |
| **data.go.kr 운영팀** | Phase 1 즉시 실행, 신규 API 등록 정책 전환 |
| **각 부처·기관** | OpenAPI 스펙 제출 의무 이행, 기존 API 전환 계획 수립 |
| **공공데이터전략위원회** | 거버넌스 감독, 포털별 AI 적합성 평가 공개 도입 |

### 성과 측정 지표

| 지표 | 현재 | Phase 1 목표 | Phase 2 목표 | Phase 3 목표 |
|------|------|------------|------------|------------|
| JSON 기본 응답 비율 | ~10% | 100% (신규) | 80% (전체) | 100% |
| HTTP 상태코드 표준 준수 | ~0% | 50% | 90% | 100% |
| CORS 지원 비율 | 소수 | 100% (data.go.kr) | 100% | 100% |
| OpenAPI 스펙 제공 비율 | ~0% | 10% (신규) | 50% | 90% |
| AI 적합성 등급 A~B | 1.2% | 10% | 40% | 80% |
| 전체 커버 필요 ID 수 | 130개 | 130개 | 30개 | 3개 |

---

## 결론

### 핵심 메시지

"데이터 공개량 세계 1위"에서 "데이터 활용 품질 세계 1위"로의 전환이 필요하다.

공공 데이터의 가치는 얼마나 많이 공개하느냐가 아니라 얼마나 쉽게 활용할 수 있느냐로 결정된다. AI 에이전트 시대에는 사람이 아닌 기계(AI)가 공공 API의 주요 소비자가 된다. 기계가 읽을 수 없는 XML, 기계가 해석할 수 없는 200 에러, 기계가 자동화할 수 없는 URL API Key — 이 세 가지가 한국 공공 API의 AI 시대 적합성을 가로막는 핵심 장벽이다.

### 즉시 행동 촉구

**Phase 1의 4개 개선안은 법령 개정도, 예산 신청도, 기관 협의도 필요 없다.** data.go.kr 운영팀이 내일 당장 시작할 수 있다. 12년 전 미국 18F가 "API Key를 URL에 넣지 말라", "JSON만 써라"를 표준화한 내용이 2026년에도 한국에서 이행되지 않은 현실은, 거버넌스의 문제이지 기술의 문제가 아니다.

**행정안전부 또는 NIA는 최소한 다음 4가지를 즉시 가이드라인으로 발표해야 한다:**

1. HTTP 상태코드 표준 준수 (오류에 4xx/5xx 사용)
2. JSON 기본 응답 방식 전환 (XML 선택 허용 유지)
3. CORS 헤더 필수 적용 (공개 API)
4. OpenAPI 3.0 명세 신규 API 의무 제출

이 4가지만 이행해도 한국 공공 API는 미국 18F 2014년 표준 수준에 도달한다. 그리고 그것이 시작이다.

---

## 참고 문헌 및 출처

### 국가별 분석 보고서
- 미국 정부 API 생태계 분석 (조사 포털 11개): api.data.gov, USAspending.gov, Census Bureau, openFDA, NASA, NWS, SEC EDGAR, USGS, GovInfo, Treasury Fiscal Data
- 영국 정부 API 생태계 분석 (조사 포털 12개): HMRC, NHS England, Companies House, OS Data Hub, ONS, TfL, DVLA, DWP, HMLR
- 싱가포르 정부 API 생태계 분석 (조사 포털 11개): data.gov.sg, APEX, LTA DataMall, OneMap, MAS, IRAS, SingPass, URA, NEA, HDB
- 프랑스 정부 API 생태계 분석: api.gouv.fr, data.gouv.fr, API Particulier, API Entreprise, FranceConnect, SIRENE, IGN, Hub'Eau
- 일본 정부 API 생태계 분석 (조사 포털 9개): e-Stat, e-Gov, RESAS, gBizINFO, EDINET, JPO, MLIT, JMA, Digital Agency
- 해외 정부 API 디자인 가이드라인 비교 분석 (영국·미국·호주·EU·프랑스·한국)
- 대한민국 공공 Open API 현황 종합 보고서 (173개 포털, ~23,760개 API)

### 공식 기술 표준 문서
- [UK GDS API Technical and Data Standards](https://www.gov.uk/guidance/gds-api-technical-and-data-standards) (2024년 7월 최신)
- [18F API Standards](https://github.com/18F/api-standards) (2014년 초판)
- [GSA API Standards](https://github.com/GSA/api-standards)
- [Australia National API Design Standards](https://github.com/apigovau/national-api-design-standards) / [api.gov.au](https://api.gov.au)
- [France DINUM API doctrine](https://www.numerique.gouv.fr/offre-accompagnement/reference-recommandations-partage-donnees-api-administration/)
- [EU High Value Dataset Regulation 2023/138](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32023R0138)
- [RFC 7807 — Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807)
- [OpenAPI Specification 3.0](https://spec.openapis.org/oas/v3.0.3)
