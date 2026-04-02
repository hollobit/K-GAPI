# OECD OURdata Index vs 실제 API 기술 품질 괴리 분석

> 조사 기준일: 2026년 4월 3일
> 본 연구는 본 프로젝트의 14개 항목 API 평가 데이터(portals.json, us.json, uk.json, fr.json, jp.json, sg.json)와 OECD 공식 지표를 비교·분석한 것이다.

---

## 1. OECD OURdata Index 개요

### 1.1 지표 정의

OECD OURdata Index(Open, Useful and Re-usable data Index)는 각국 정부의 공공데이터 개방 정책을 측정하는 대표적인 국제 지표이다. 2019년부터 격년 주기로 발행되며, 2023년판은 36개 OECD 회원국과 4개 후보국 등 40개국을 대상으로 670개 이상의 데이터 포인트를 수집하였다.

### 1.2 2023년 주요 결과

- **한국 1위** (종합 점수 최고)
- **OECD 평균**: 0.60/1.0 (추정)
- **상위 3개국**: 한국, 프랑스, 폴란드

| 순위 | 국가 | OURdata 점수 (추정) |
|------|------|---------------------|
| 1 | 🇰🇷 한국 | ~0.93 |
| 2 | 🇫🇷 프랑스 | ~0.87 |
| 3 | 🇵🇱 폴란드 | ~0.84 |
| 상위권 | 🇬🇧 영국 | ~0.80 |
| 중상위권 | 🇯🇵 일본 | ~0.62 |
| 비OECD | 🇸🇬 싱가포르 | 비교군 참여 (파트너국) |

### 1.3 측정 구조: 3개 축

**축 1 — Data Availability (데이터 가용성)**
- 국가 공공데이터 전략 존재 여부
- 10개 주제 영역에 걸친 82개 고가치 데이터셋 공개 여부
- 라이선스 개방성 (CC BY, OGL 등)
- 데이터 포맷의 기계 가독성 선언

**축 2 — Data Accessibility (데이터 접근성)**
- 중앙 공개데이터 포털을 통한 접근 가능성
- 포털의 사용 편의성 (검색, 필터, 메타데이터)
- API 형식으로 데이터 제공 여부 (API 존재 유무만 판단)
- 오픈 포맷 비율

**축 3 — Government Support for Data Re-use (데이터 재이용 지원)**
- 민간·스타트업·연구기관과의 파트너십 프로그램
- 데이터 리터러시 교육 지원
- 공공데이터 활용 대회·해커톤 개최
- 데이터 품질 모니터링 정책

---

## 2. 지표의 한계 — 무엇을 측정하고 무엇을 측정하지 않는가

### 2.1 측정하는 것 (정책·선언적 수준)

| 측정 항목 | 한국 수행도 | 비고 |
|-----------|-------------|------|
| 국가 공공데이터 전략 수립 | 매우 높음 | 공공데이터법, 전략 로드맵 |
| 중앙 포털(data.go.kr) 존재 | 매우 높음 | 19,198개 API 등록 |
| 고가치 데이터셋 공개 수 | 매우 높음 | 절대 데이터셋 수 세계 최다 수준 |
| 오픈 라이선스 적용 | 높음 | 공공데이터 이용허락 표준 |
| 민간 파트너십 프로그램 | 높음 | 공공데이터 활용 창업 지원 |
| 데이터 리터러시 교육 | 높음 | NIA 교육 프로그램 |
| 해커톤·공모전 | 매우 높음 | 공공데이터 활용 대회 다수 |

### 2.2 측정하지 않는 것 (기술·개발자 경험 수준)

| 미측정 항목 | 한국 실제 상태 | 영향 |
|-------------|----------------|------|
| **API 응답 포맷** (JSON vs XML) | XML 기본값. 약 70%가 XML 우선 | AI/LLM 연동 시 파싱 레이어 필요 |
| **OpenAPI 스펙 존재 여부** | 사실상 0% (Swagger/OAS 없음) | AI 에이전트 자동 발견 불가 |
| **HTTP 상태코드 준수** | 오류 시에도 200 OK 반환 관행 | 에러 처리 불가능 |
| **CORS 지원 여부** | 대부분 미지원 | 브라우저/클라이언트 직접 호출 불가 |
| **인증 획득 용이성** | 휴대폰 본인인증 필수, 2년 유효기간 | 외국 개발자 접근 차단 |
| **인증 기술 수준** | serviceKey URL 이중 인코딩 버그 존재 | 실제 연동 시 오류 빈발 |
| **Rate Limit 투명성** | 헤더 미제공, 1일 1,000회 경고 방식 | 자동화 어려움 |
| **에러 메시지 품질** | 코드+한글 텍스트, 비표준 | 자동 디버깅 불가 |
| **API 키 발급 속도** | 수동 승인 API 다수 (수일~수주) | 개발 속도 저하 |
| **샌드박스 환경** | 거의 없음 | 프로덕션 데이터로만 테스트 |
| **SDK/라이브러리 품질** | 비공식 서드파티만 존재 | 공식 지원 없음 |
| **MCP/AI 도구 지원** | 공식 0개 | AI 에이전트 연동 불가 |
| **국제화 (영문 문서)** | 거의 없음 | 외국 개발자 접근 차단 |
| **버전 관리·하위 호환성** | 정책 미비 | 갑작스러운 API 변경 위험 |

### 2.3 핵심 왜곡 메커니즘

**"API 존재 여부"와 "API 사용 가능 여부"를 동일시하는 오류**

OURdata Index는 API 형식으로 데이터를 제공하는지를 이진(0/1) 또는 점수로 측정한다. 그러나 다음 두 상황을 동일하게 처리한다:

- 상황 A: JSON 네이티브, OpenAPI 스펙, CORS, 즉시 키 발급, 공식 MCP 서버 → "API 있음"
- 상황 B: XML 기본값, 스펙 없음, CORS 없음, 본인인증 필요, 이중 인코딩 버그 → "API 있음"

한국은 상황 B임에도 불구하고 API 수(19,198개)가 압도적으로 많아 "가용성" 점수에서 최고점을 받는다.

---

## 3. "API 기술 품질 지수" 신규 제안

본 연구팀이 수집한 14개 항목 기반의 독자적 지수를 제안한다. 이 지수는 실제 개발자와 AI 에이전트가 API를 사용할 수 있는지를 측정한다.

### 3.1 14개 평가 항목 (각 0~2점, 총 28점)

**사용자 친화성 (7개 항목, 14점)**
1. **signup_ease** — 회원가입 용이성 (2=없음/이메일, 1=복잡, 0=기관인증)
2. **approval_speed** — API 키 승인 속도 (2=즉시, 1=자동지연, 0=수동심사)
3. **data_format** — 데이터 포맷 (2=JSON기본, 1=JSON+XML, 0=XML전용)
4. **doc_quality** — 문서 품질 (2=우수, 1=보통, 0=미흡)
5. **rate_limit** — 호출 한도 적절성 (2=충분·투명, 1=제한적, 0=불명확)
6. **error_handling** — 오류 처리 품질 (2=표준HTTP+설명, 1=부분, 0=비표준)
7. **sdk_samples** — SDK/샘플 제공 (2=공식SDK, 1=샘플만, 0=없음)

**AI 준비도 (7개 항목, 14점)**
1. **machine_schema** — 기계 판독 스펙 (2=OpenAPI 3.0, 1=부분, 0=없음)
2. **http_semantics** — HTTP 표준 준수 (2=완전준수, 1=부분, 0=비준수)
3. **json_native** — JSON 네이티브 (2=JSON기본, 1=JSON지원, 0=XML전용)
4. **error_self_desc** — 자기설명적 오류 (2=RFC 7807등, 1=부분, 0=미지원)
5. **programmatic_auth** — 프로그래매틱 인증 (2=API키/OAuth2, 1=부분, 0=인증서)
6. **rate_limit_transparency** — 율제한 투명성 (2=헤더제공, 1=문서만, 0=없음)
7. **cors_https** — CORS+HTTPS (2=둘다, 1=HTTPS만, 0=미지원)

### 3.2 국가별 API 기술 품질 지수 계산

각국의 대표 API들의 평균 점수를 28점 만점 기준으로 산출한 후 10점 척도로 환산하였다.

#### 🇰🇷 한국 — 공공데이터포털 (data.go.kr) 대표 사례
```
사용자 친화성: signup_ease=1, approval_speed=1, data_format=1, doc_quality=1,
              rate_limit=1, error_handling=1, sdk_samples=1 → 7/14
AI 준비도:     machine_schema=0, http_semantics=0, json_native=1, error_self_desc=0,
              programmatic_auth=1, rate_limit_transparency=0, cors_https=1 → 3/14
총점: 10/28 → 3.6/10
```

#### 🇺🇸 미국 — api.data.gov / GovInfo 대표 사례
```
사용자 친화성: signup_ease=2, approval_speed=2, data_format=2, doc_quality=2,
              rate_limit=2, error_handling=2, sdk_samples=1 → 13/14
AI 준비도:     machine_schema=2, http_semantics=2, json_native=2, error_self_desc=2,
              programmatic_auth=2, rate_limit_transparency=2, cors_https=2 → 14/14
총점: 27/28 → 9.6/10
```

#### 🇬🇧 영국 — HMRC Developer Hub / Companies House 대표 사례
```
사용자 친화성: signup_ease=2, approval_speed=2, data_format=2, doc_quality=2,
              rate_limit=2, error_handling=2, sdk_samples=2 → 14/14
AI 준비도:     machine_schema=2, http_semantics=2, json_native=2, error_self_desc=2,
              programmatic_auth=2, rate_limit_transparency=2, cors_https=2 → 14/14
총점: 28/28 → 10.0/10 (상위 API 기준)
평균(카탈로그 포함): 24/28 → 8.6/10
```

#### 🇫🇷 프랑스 — data.gouv.fr / api.gouv.fr 대표 사례
```
사용자 친화성: signup_ease=2, approval_speed=2, data_format=2, doc_quality=2,
              rate_limit=2, error_handling=2, sdk_samples=2 → 14/14
AI 준비도:     machine_schema=2, http_semantics=2, json_native=2, error_self_desc=2,
              programmatic_auth=2, rate_limit_transparency=2, cors_https=2 → 14/14
총점: 28/28 → 10.0/10 (data.gouv.fr 기준)
평균(전체 포털): 22/28 → 7.9/10
```

#### 🇯🇵 일본 — e-Gov 법령API v2 / e-Stat 대표 사례
```
사용자 친화성: signup_ease=2, approval_speed=2, data_format=2, doc_quality=2,
              rate_limit=2, error_handling=2, sdk_samples=2 → 14/14 (법령API 기준)
AI 준비도:     machine_schema=2, http_semantics=2, json_native=2, error_self_desc=2,
              programmatic_auth=2, rate_limit_transparency=1, cors_https=1 → 12/14
총점 (최우수): 26/28 → 9.3/10
평균(전체 포털): 17/28 → 6.1/10 (RESAS·JPO 등 낙후 API 포함)
```

#### 🇸🇬 싱가포르 — data.gov.sg / LTA DataMall 대표 사례
```
사용자 친화성: signup_ease=2, approval_speed=2, data_format=2, doc_quality=2,
              rate_limit=2, error_handling=2, sdk_samples=1 → 13/14
AI 준비도:     machine_schema=2, http_semantics=2, json_native=2, error_self_desc=2,
              programmatic_auth=2, rate_limit_transparency=2, cors_https=2 → 14/14
총점: 27/28 → 9.6/10 (data.gov.sg 기준)
평균(전체 포털): 21/28 → 7.5/10
```

---

## 4. OECD OURdata vs 실제 API 기술 품질 — 국가별 비교표

| 국가 | OECD OURdata 2023 | API 기술 품질 지수 (본 연구, /10) | 괴리 | 해석 |
|------|:-----------------:|:--------------------------------:|:----:|------|
| 🇰🇷 한국 | **1위 (~0.93)** | **3.6** | **-6.4** | 정책 1위, 기술 최하위 — 최대 과대평가 |
| 🇫🇷 프랑스 | 2위 (~0.87) | 7.9 | -0.8 | 정책·기술 모두 우수, 소폭 과대평가 |
| 🇬🇧 영국 | 상위권 (~0.80) | 8.6 | +0.6 | 기술이 정책 평가를 소폭 상회 |
| 🇺🇸 미국 | 중위권 (~0.74) | 9.6 | +2.2 | 기술이 정책 평가를 크게 상회 |
| 🇯🇵 일본 | 중상위권 (~0.62) | 6.1 (평균) | -0.2 | 거의 일치. 그러나 편차가 큼(3.0~9.3) |
| 🇸🇬 싱가포르 | 비OECD 파트너 | 7.5 | (기준 외) | 기술 수준은 OECD 상위권에 필적 |

> 주: OECD OURdata 점수는 공개된 순위 정보와 각국 보고서를 기반으로 추정한 것이며, 공식 수치와 소폭 차이가 있을 수 있다. API 기술 품질 지수는 본 프로젝트의 실측 데이터 기반이다.

---

## 5. 괴리가 의미하는 것

### 5.1 한국: 정책의 역설 (Policy Paradox)

한국은 세계 최고 수준의 공공데이터 개방 **정책**을 보유하지만, 실제 개발자와 AI가 그 데이터를 **사용**하기는 가장 어려운 나라 중 하나이다.

**구체적 패러독스**:
- 데이터셋 수: 세계 최다 수준 → 실제 사용 가능한 데이터: 극히 제한
- 공공데이터 활용 지원 예산: 막대 → AI 에이전트 실제 연동: 공식 0건
- 해커톤·대회: 연간 수십 회 → MCP 공식 서버: 0개 (커뮤니티 8개)
- OECD 1위 → 해외 개발자가 실제 사용하는 한국 공공 API: 사실상 없음

**원인 분석**:
1. 지표가 "입력(Input)" 중심: 정책 수립·데이터셋 등록·예산 투입을 측정
2. 지표가 "결과(Outcome)" 무시: 실제 API 호출 성공률·개발자 채택률 미측정
3. 기술 부채 무시: SOAP→REST 미전환, XML 기본값, URL 인코딩 버그 등 기술 문제를 정책 점수로 가릴 수 있음

### 5.2 미국: 기술이 정책을 앞서는 나라

미국은 OECD OURdata 중위권이지만 API 기술 품질은 최고 수준이다. GovInfo MCP, OpenAPI 스펙, CORS, 즉시 키 발급 등 개발자 경험(DX)이 탁월하다. OURdata 지표가 미국의 실제 오픈데이터 생태계 영향력을 과소평가하고 있다.

### 5.3 프랑스: 정책과 기술이 일치하는 모범 사례

프랑스는 OURdata 2위인 동시에 API 기술 품질도 높다. data.gouv.fr의 공식 MCP 서버, OpenAPI 스펙, 무인증 공개 API가 정책 점수를 실제 기술 수준으로 뒷받침한다.

### 5.4 일본: 이분화된 생태계

일본은 e-Gov 법령API v2(세계 최고 수준)와 JMA 기상 API(문서 없는 비공식 API)가 공존하는 이분화 구조다. OURdata 중상위권 점수는 평균을 반영하지만, 최우수 API와 최하위 API의 격차(9.3 vs 3.0)가 가장 크다.

### 5.5 싱가포르: 지표 밖의 우수성

싱가포르는 OECD 회원국이 아니어서 OURdata 공식 순위에 포함되지 않는다. 그러나 GovTech의 API 설계 원칙(무인증 테스트, OpenAPI, CORS, 실시간 데이터)은 OECD 상위권을 능가한다. 지표 자체의 적용 범위 편향이 드러나는 사례이다.

---

## 6. 개선된 지표 제안 (OECD에 권고할 보완 항목)

### 6.1 현행 지표의 문제점 요약

현행 OURdata Index는 **"What is available"(무엇이 존재하는가)**를 측정하지만, **"How usable it is"(얼마나 사용 가능한가)**를 측정하지 않는다. 공공데이터 개방의 궁극적 목표가 활용(re-use)임을 감안하면, 이는 목표와 측정 간의 근본적 불일치다.

### 6.2 신규 보완 지표: "API 기술 품질 하위 지수"

OECD OURdata 4번째 축으로 추가를 권고하는 **"Data Technical Quality"** 하위 지수:

| 보완 지표 | 측정 방법 | 배점 |
|-----------|-----------|------|
| **API 응답 포맷** | JSON 네이티브 API 비율 (%) | 0~1 |
| **기계 판독 스펙** | OpenAPI/JSON Schema 제공 API 비율 | 0~1 |
| **HTTP 표준 준수** | 표준 상태코드 사용 비율 (샘플 테스트) | 0~1 |
| **접근 용이성** | 인증 없이 또는 이메일 즉시 접근 가능한 API 비율 | 0~1 |
| **CORS 지원율** | 브라우저 직접 접근 가능 API 비율 | 0~1 |
| **개발자 채택 지표** | 서드파티 SDK·MCP 서버·npm 패키지 수 (생태계 활성도) | 0~1 |
| **AI 연동 준비도** | 공식 MCP·AI 플러그인·OpenAPI AI 확장 존재 여부 | 0~1 |
| **국제 접근성** | 영문 API 문서 및 외국 개발자 가입 가능 여부 | 0~1 |

### 6.3 측정 방법론 제안

**정량적 측정** (자동화 가능):
- 각 국가 중앙 포털의 상위 20개 API에 대한 실제 테스트 호출
- JSON 응답 여부 자동 확인
- HTTP 상태코드 표준 준수 여부 자동 확인
- CORS 헤더 존재 여부 자동 확인
- OpenAPI 스펙 URL 존재 여부 확인

**정성적 측정** (전문가 패널):
- API 문서 품질 평가 (1~5점)
- 개발자 온보딩 경험 평가 (mystery developer 방식)
- 에러 메시지 자기설명성 평가

**생태계 지표** (GitHub API 활용):
- 해당국 공공 API 관련 GitHub 레포지토리 수
- npm/PyPI/etc. 관련 패키지 수
- MCP 서버 수 및 별점 합산

### 6.4 보완 지표 적용 시 예상 순위 변화

| 국가 | 현행 OURdata (정책) | 보완 기술 지수 | 통합 지수 (70:30 가중) | 순위 변동 |
|------|:-------------------:|:--------------:|:---------------------:|:---------:|
| 🇰🇷 한국 | 1위 | 최하위 | 상위권 탈락 예상 | ↓↓ |
| 🇫🇷 프랑스 | 2위 | 상위권 | 1~2위 유지 | → |
| 🇬🇧 영국 | 상위권 | 최상위 | 순위 상승 | ↑ |
| 🇺🇸 미국 | 중위권 | 최상위 | 순위 대폭 상승 | ↑↑ |
| 🇯🇵 일본 | 중상위권 | 중간 (편차 큼) | 유사 | → |
| 🇸🇬 싱가포르 | 비OECD | 최상위급 | 상위 포함시 상위권 | 신규 포함 필요 |

### 6.5 한국 정부에 대한 정책 권고

**즉시 조치 가능 (기술 수정)**:
1. serviceKey URL 이중 인코딩 버그 수정 (추정 영향: 수천 개 API)
2. HTTP 오류 응답 시 200 OK 반환 금지 (표준 상태코드 의무화)
3. CORS 헤더 허용 (Access-Control-Allow-Origin 추가)
4. JSON 응답을 기본값으로 전환 (XML은 옵션)

**단기 정책 (1~2년)**:
1. 신규 등록 공공 API에 OpenAPI 3.0 스펙 제출 의무화
2. 외국인도 이메일만으로 API 키 발급 가능하도록 인증 간소화
3. 국가법령정보 MCP 공식 서버 운영 (법제처)
4. 공공데이터포털 MCP 게이트웨이 파일럿 운영

**중장기 정책 (3~5년)**:
1. 공공 API 기술 품질 인증 제도 도입 (MCP Ready 배지)
2. 범정부 AI 데이터 게이트웨이 구축 (싱가포르 APEX 벤치마킹)
3. OECD에 "API 기술 품질" 지표 추가 제안 (한국이 선제 주도)

---

## 7. 결론

### 7.1 핵심 발견

> **"한국은 세계에서 가장 많은 공공 API를 보유하고 있으나, 그 중 AI가 실제로 사용할 수 있는 API의 비율은 가장 낮은 수준이다."**

OECD OURdata 1위라는 국제적 공인은 한국의 공공데이터 **정책** 수준이 세계 최고임을 증명한다. 그러나 본 연구가 실측한 API 기술 품질 지수(3.6/10)는 그 정책이 실제 개발자와 AI 에이전트에게 아직 닿지 않았음을 보여준다.

### 7.2 기회

이 괴리는 위협이 아니라 기회이다. 한국은:
- 이미 최대 규모의 공공 API 인프라를 보유
- 세계 최고 수준의 개방 정책과 예산
- 활발한 개발자 커뮤니티(법령 MCP 3개, 공공데이터 MCP 5개 이상)

기술 부채(XML 기본값, URL 인코딩 버그, CORS 없음, OpenAPI 없음)만 해소한다면, 한국은 **정책과 기술 모두에서 세계 1위**를 달성할 수 있는 유일한 나라이다.

### 7.3 OECD에 대한 제언

OURdata Index는 공공데이터 생태계의 절반만 측정하고 있다. "API 기술 품질" 축을 추가하지 않는 한, 이 지표는 정부가 개발자에게 데이터를 "열어준 척"하는 것과 실제로 "사용 가능하게 만드는 것"의 차이를 포착하지 못한다. 본 연구가 제안한 8개 보완 지표가 차기 OURdata Index에 반영되기를 권고한다.

---

**참고 자료**:
- OECD (2023). *2023 OECD Open, Useful and Re-usable data (OURdata) Index*. OECD Publishing. https://www.oecd.org/en/publications/2023-oecd-open-useful-and-re-usable-data-ourdata-index_a37f51c3-en.html
- 과학기술정보통신부/NIA (2023). *공공데이터 제공 및 이용 활성화 시행계획*.
- GovInfo (2026-01-22). *AI Agents Meet Federal Data: Public Preview for the GovInfo MCP Server*. https://www.govinfo.gov/features/mcp-public-preview
- data.gouv.fr (2025). *Le serveur MCP de data.gouv.fr*. https://guides.data.gouv.fr/intelligence-artificielle/le-serveur-mcp-de-data.gouv.fr
- i.AI (2025). *Parliament MCP*. https://github.com/i-dot-ai/parliament-mcp
- 본 프로젝트 실측 데이터: `/data/portals.json`, `/data/global/{us,uk,fr,jp,sg}.json`
