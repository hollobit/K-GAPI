# K-GAPI: 대한민국 공공 Open API 전수 조사 프로젝트

**조사 기준일**: 2026-04-03  
**조사 범위**: 199개 포털, 8개국 비교, ~23,760개 API  
**핵심 결론**: 데이터 개방량 OECD 1위, API 기술 품질 조사 대상 9개국 중 최하위

---

## 프로젝트 개요

대한민국 공공 Open API 생태계를 **전수 조사**하고, 해외 8개국(미국/영국/EU/싱가포르/호주/일본/캐나다/프랑스)과 **정량 비교**하여, 사용자 중심성과 AI 시대 적합성을 평가한 프로젝트입니다.

### 핵심 수치

| 지표 | 값 |
|------|-----|
| 조사 포털 수 | **199개** (173개 기존 + 26개 신규 발굴) |
| 추정 API 수 | **~23,760개** |
| 전체 커버 시 필요 ID | **130개** |
| data.go.kr만으로 완전 동작 | **18%** (31개 포털) |
| 사용자 중심성 (한국) | **5.3/14** (D등급, 9개국 중 최하위) |
| AI 적합성 (한국) | **2.7/14** (E등급, 9개국 중 최하위) |
| OECD OURdata Index | **1위** (0.93/1.0) |
| API 기술 품질 지수 | **3.6/10** (해외 평균 7.4) |
| 개발자 시간 낭비 추정 | 연간 **400~800만 시간** (2,000~4,000억원) |

---

## 빠르게 보기

- **대시보드**: [대한민국 공공 Open API 현황 분석](http://hollobit.github.io/K-GAPI/index.html) 을 브라우저에서 열면 전체 분석 결과를 인터랙티브하게 확인 가능
- **종합 보고서**: [`report.md`](report.md) — 전체 분석 결과 요약
- **개선 로드맵**: [`report/korea_improvement_roadmap.md`](report/korea_improvement_roadmap.md) — 해외 사례 기반 단계별 실행 계획

---

## 디렉토리 구조

* 종합 분석 보고서 (report)
   * policy_recommendations.md  ← [정부 대상 정책 제언서](report/policy_recommendations.md)
   * korea_improvement_roadmap.md ← [해외 사례 기반 개선 로드맵](report/korea_improvement_roadmap.md)
   * policy_recommendations.md  ← [정부 대상 정책 제언서](report/policy_recommendations.md)
   * korea_improvement_roadmap.md ← [해외 사례 기반 개선 로드맵](report/korea_improvement_roadmap.md)
   * tco_analysis.md            ← [총 비용(TCO) 분석](report/tco_analysis.md)
   * oecd_gap_analysis.md       ← [OECD 지표 vs 실제 품질 괴리](report/oecd_gap_analysis.md)
   * developer_guide.md         ← [개발자 온보딩 가이드](report/developer_guide.md)
   * mcp_guide.md               ← [MCP 서버 개발 가이드](report/mcp_guide.md)

* 해외 비교 분석 (comparison)
   * api_design_guidelines_comparison.md ← [6개국 API 표준 비교](comparison/api_design_guidelines_comparison.md)
   * domain_api_comparison.md    ← [동일 도메인 1:1 비교 -기상/금융/통계](comparison/domain_api_comparison.md)
   * governance_models.md        ← [거버넌스 모델 비교 -중앙/분산/하이브리드](comparison/governance_models.md)
   * developer_portal_ux_comparison.md ← [개발자 포털 UX 비교](comparison/developer_portal_ux_comparison.md)
   * global_mcp_ecosystem.md    ← [글로벌 MCP/AI 도구 생태계](comparison/global_mcp_ecosystem.md)

* 국가별 상세 보고서 (global)
   * us_report.md               ← [🇺🇸 미국 분석](global/us_report.md)
   * uk_report.md               ← [🇬🇧 영국 분석](global/uk_report.md)
   * eu_report.md               ← [🇪🇺 EU 분석](global/eu_report.md)
   * sg_report.md               ← [🇸🇬 싱가포르 분석](global/sg_report.md)
   * au_report.md               ← [🇦🇺 호주 분석](global/au_report.md)
   * jp_report.md               ← [🇯🇵 일본 분석](global/jp_report.md)
   * ca_report.md               ← [🇨🇦 캐나다 분석](global/ca_report.md)
   * fr_report.md               ← [🇫🇷 프랑스 분석](global/fr_report.md)

* 분야별(sector_reports)
   * finance.md                 ← [금융 분야](sector_reports/finance.md)
   * transport.md               ← [교통 분야](sector_reports/transport.md)
   * health.md                  ← [보건 분야](sector_reports/health.md)

---

## 주요 분석 결과

### 1. 9개국 벤치마크

| 순위 | 국가 | 사용자 중심성 (/14) | AI 적합성 (/14) | 특기 사항 |
|------|------|:------------------:|:--------------:|----------|
| 1 | 🇺🇸 미국 | 12.2 (A) | 12.3 (A) | GovInfo MCP, 45% 무인증 |
| 2 | 🇪🇺 EU | 11.8 (A) | 11.8 (A) | 78% Keyless, HVD 법적 의무 |
| 3 | 🇫🇷 프랑스 | 11.6 (A) | 12.0 (A) | 정부 공식 MCP 서버 운영 |
| 4 | 🇸🇬 싱가포르 | 11.0 (B) | 12.5 (A) | APEX 월 1억회, Key-Free 테스트 |
| 5 | 🇬🇧 영국 | 10.4 (B) | 11.8 (A) | API Key 금지, OAuth 2.0 필수 |
| 6 | 🇯🇵 일본 | 9.2 (B) | 8.0 (C) | 법령API v2 A등급, MCP 6개 |
| 7 | 🇦🇺 호주 | 8.6 (C) | 10.2 (B) | GitHub 공개 API 표준 |
| 8 | 🇨🇦 캐나다 | 7.6 (C) | 8.4 (C) | 이중언어, API Store 종료 교훈 |
| **9** | **🇰🇷 한국** | **5.3 (D)** | **2.7 (E)** | **표준 없음, 200 일관 반환, XML** |

### 2. 한국 공공 API의 구조적 문제

| 문제 | 한국 현황 | 해외 선진국 |
|------|----------|-----------|
| HTTP 상태코드 | **항상 200 반환** (에러도) | 401/404/429 표준 사용 |
| 기본 응답 포맷 | **XML** (59%) | JSON 전용 |
| 인증 방식 | **API Key in URL** | OAuth 2.0 / Header |
| OpenAPI 스펙 | **미제공** (~2%) | 제공 (~75%) |
| CORS | **미지원** (0%) | 지원 필수 |
| Rate Limit 헤더 | **없음** | 표준 제공 |
| API 표준 | **없음** | GDS/GSA/NAPIDS 등 |
| 정부 MCP 서버 | **없음** | 미국/프랑스/영국 운영 |

### 3. 포털 파편화

- **173개 포털**에 **12종의 서로 다른 인증키 파라미터명** 사용
- 같은 데이터가 2~3개 포털에서 **다른 키, 다른 포맷, 다른 버전**으로 제공
- data.go.kr 1개 키로 완전히 동작하는 것은 **전체의 18%뿐**

### 4. 개발자 비용

- 소규모 앱 TCO: 연간 **330만원** (해외 동등 서비스 대비 6.3배)
- serviceKey 인코딩 디버깅: 포털당 **0.5~2일**
- 포털별 어댑터 개발: 포털당 **2~5일**

---

## 개선 로드맵 요약

### 즉시 실행 (6개월, 예산 최소)
1. **CORS 헤더 추가** — nginx 설정 1줄 (난이도: 하)
2. **X-RateLimit 헤더 추가** — 게이트웨이 설정 (난이도: 하)
3. **JSON 기본 응답 전환** — 파라미터 기본값 변경 (난이도: 하)
4. **HTTP 상태코드 정상화** — 401/404/429 도입 (난이도: 중)

### 중기 (1~2년)
5. 정부 API 디자인 표준 제정
6. OpenAPI 3.0 스펙 의무화
7. API Key 헤더 전환 (URL 노출 금지)
8. 통합 인증 체계 (1 ID for all)

### 장기 (2~3년)
9. 통합 API Gateway (data.go.kr 게이트웨이 18%→80%)
10. OAuth 2.0 전환
11. 정부 공식 MCP 서버 구축

> 상세 내용: [`output/korea_improvement_roadmap.md`](output/korea_improvement_roadmap.md)

---

## 평가 기준

### 사용자 중심성 (7개 항목, 각 0~2점, 총 14점)

| 항목 | 0점 (공급자 중심) | 2점 (사용자 중심) |
|------|-----------------|-----------------|
| 가입 용이성 | 공동인증서/기관심사 | 이메일/소셜 로그인 |
| 승인 속도 | 3일+ 수동승인 | 즉시 자동승인 |
| 데이터 포맷 | XML only | JSON 기본 |
| API 문서 품질 | 문서 부실 | Swagger/테스트콘솔 |
| 호출 한도 | 일 100회 이하 | 일 10,000회+ |
| 에러 처리 | 코드 불명확 | 명확한 코드+해결방안 |
| SDK/샘플코드 | 없음 | 다국어 SDK |

> 등급: A(12~14) / B(9~11) / C(6~8) / D(3~5) / E(0~2)

### AI 적합성 (7개 항목, 각 0~2점, 총 14점)

| 항목 | 0점 (부적합) | 2점 (AI-Ready) |
|------|------------|---------------|
| 기계 판독 스키마 | 문서 없음/HWP | OpenAPI 3.0 JSON |
| HTTP 의미론 | 항상 200 반환 | 표준 상태코드 |
| JSON 네이티브 | XML only | JSON 기본 |
| 에러 자기설명성 | 코드만 | RFC 7807 Problem Details |
| 프로그래매틱 인증 | 공동인증서 | OAuth 2.0 + Header |
| Rate Limit 투명성 | 없음 | X-RateLimit 헤더 |
| CORS/HTTPS | 미지원 | 완전 지원 |

---

## 기술 스택

| 항목 | 선택 | 이유 |
|------|------|------|
| 데이터 저장 | JSON 파일 | DB 불필요, 이식성 |
| 대시보드 | 단일 HTML + 인라인 CSS/JS | 서버 불필요, 파일 더블클릭 열기 |
| 차트 | 인라인 SVG/Canvas | CDN 불필요 |
| 보고서 | Markdown | GitHub 렌더링 호환 |
| MCP 서버 | Node.js + @modelcontextprotocol/sdk | 공식 SDK |
| API 스펙 | OpenAPI 3.0 YAML | 업계 표준 |

---

## 라이선스

본 프로젝트의 분석 자료와 코드는 자유롭게 활용 가능합니다. 분석에 사용된 공공 데이터는 각 포털의 공공누리 라이선스를 따릅니다.

---

*본 조사는 2026년 4월 기준이며, 각 포털의 정책은 수시로 변경될 수 있습니다.*
*전종홍 (hollobit@etri.re.kr)*
