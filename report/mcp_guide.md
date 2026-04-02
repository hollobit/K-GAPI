# 한국 공공 API를 위한 MCP 서버 개발 가이드

**작성일**: 2026-04-02
**대상 독자**: MCP 서버를 직접 구축하려는 Python 개발자
**전제 조건**: Python 3.10+, `mcp` SDK, `httpx`, `xmltodict`

---

## 1. 개요

### MCP란?

MCP(Model Context Protocol)는 Anthropic이 공개한 프로토콜로, AI 모델(Claude, Cursor 등)이 외부 도구와 데이터 소스에 **표준화된 방식으로 접근**할 수 있게 해준다. MCP 서버는 하나 이상의 Tool을 정의하고, AI 클라이언트가 이 Tool을 호출하면 서버가 실제 API를 대신 호출하여 결과를 반환한다.

```
[AI 클라이언트] <--MCP 프로토콜--> [MCP 서버] <--HTTP--> [공공 API]
  (Claude, Cursor)                (우리가 만들 것)        (data.go.kr 등)
```

### 왜 한국 공공 API에 MCP가 필요한가?

한국 공공 API는 AI Agent가 직접 사용하기에 심각한 구조적 장벽이 있다:

| 문제 | AI에 미치는 영향 | MCP 서버의 역할 |
|------|----------------|----------------|
| OpenAPI 스펙 미제공 | LLM이 API 구조를 자동 파악 불가 | Tool schema로 구조화된 인터페이스 제공 |
| XML 기본 응답 | JSON 대비 2.5배 토큰 소모 | XML→JSON 변환 후 정제된 데이터 전달 |
| HTTP 200 일관 반환 | 에러 자동 감지 불가 | body 파싱으로 에러를 감지하여 명확한 에러 메시지 반환 |
| serviceKey 이중인코딩 | 인증 실패 빈발 | 서버 내부에서 인코딩 처리 |
| CORS 미지원 | 브라우저 직접 호출 불가 | 서버사이드에서 프록시 역할 |
| 포털마다 비표준 패턴 | 범용 클라이언트 불가 | 포털별 어댑터를 MCP Tool로 추상화 |

MCP 서버는 이 모든 문제를 **한 번만 해결**하면, 이후 모든 AI 클라이언트가 깨끗한 인터페이스로 공공 데이터를 활용할 수 있다.

---

## 2. 한국 공공 API의 특수성 (MCP 개발 시 주의점)

### 2.1 serviceKey 이중인코딩 문제

data.go.kr이 발급하는 인증키에는 Base64 특수문자(`+`, `/`, `=`)가 포함된다. HTTP 클라이언트가 자동으로 URL 인코딩을 수행하면, 이미 인코딩된 키가 **이중 인코딩**되어 인증 실패가 발생한다.

```python
# 잘못된 방법: requests/httpx가 자동으로 serviceKey를 한 번 더 인코딩한다
params = {"serviceKey": "abc+def/ghi=", "pageNo": 1}
response = httpx.get(url, params=params)  # serviceKey가 abc%2Bdef%2Fghi%3D 로 변환됨

# 올바른 방법 1: 디코딩된 키를 직접 URL에 삽입
import urllib.parse

SERVICE_KEY = "abc+def/ghi="
decoded_key = urllib.parse.unquote(SERVICE_KEY)
full_url = f"{base_url}?serviceKey={decoded_key}&pageNo=1"
response = httpx.get(full_url)

# 올바른 방법 2: 인코딩된 키를 그대로 URL 문자열에 포함
ENCODED_KEY = "abc%2Bdef%2Fghi%3D"
full_url = f"{base_url}?serviceKey={ENCODED_KEY}&pageNo=1"
response = httpx.get(full_url)
```

**MCP 서버에서의 해결 전략**:

```python
def build_request_url(base_url: str, service_key: str, params: dict[str, str]) -> str:
    """serviceKey 이중인코딩을 방지하는 URL 빌더."""
    query_parts = [f"serviceKey={service_key}"]
    for key, value in params.items():
        query_parts.append(f"{key}={urllib.parse.quote(str(value))}")
    return f"{base_url}?{'&'.join(query_parts)}"
```

### 2.2 HTTP 200 일관 반환 - body 파싱으로 에러 감지

한국 공공 API의 가장 심각한 문제로, 인증 실패/파라미터 오류/호출 초과 등 **모든 에러가 HTTP 200 OK**로 반환된다. 에러 정보는 응답 body 안의 자체 코드에 들어 있다.

```xml
<!-- HTTP 200 OK 이지만 실제로는 인증 실패 -->
<response>
  <header>
    <resultCode>30</resultCode>
    <resultMsg>SERVICE_KEY_IS_NOT_REGISTERED_ERROR</resultMsg>
  </header>
</response>
```

**MCP 서버에서의 해결 전략**:

```python
from dataclasses import dataclass


# data.go.kr 에러코드 매핑 (불변 딕셔너리)
ERROR_CODE_MAP: dict[str, str] = {
    "00": "NORMAL_SERVICE",
    "01": "APPLICATION_ERROR",
    "02": "DB_ERROR",
    "03": "NODATA_ERROR",
    "04": "HTTP_ERROR",
    "05": "SERVICETIME_OUT",
    "10": "INVALID_REQUEST_PARAMETER_ERROR",
    "11": "NO_MANDATORY_REQUEST_PARAMETERS_ERROR",
    "12": "NO_OPENAPI_SERVICE_ERROR",
    "20": "SERVICE_ACCESS_DENIED_ERROR",
    "21": "TEMPORARILY_DISABLE_THE_SERVICEKEY_ERROR",
    "22": "LIMITED_NUMBER_OF_SERVICE_REQUESTS_EXCEEDS_ERROR",
    "30": "SERVICE_KEY_IS_NOT_REGISTERED_ERROR",
    "31": "DEADLINE_HAS_EXPIRED_ERROR",
    "32": "UNREGISTERED_IP_ERROR",
    "33": "UNSIGNED_CALL_ERROR",
    "99": "UNKNOWN_ERROR",
}


@dataclass(frozen=True)
class ApiResponse:
    """공공 API 응답을 정규화한 불변 객체."""
    success: bool
    data: dict | list | None
    error_code: str | None
    error_message: str | None
    raw_status_code: int


def check_data_go_kr_error(parsed_body: dict) -> ApiResponse | None:
    """data.go.kr 스타일 응답에서 에러를 감지한다.

    에러가 없으면 None을 반환하고, 에러가 있으면 ApiResponse를 반환한다.
    """
    header = parsed_body.get("response", {}).get("header", {})
    result_code = str(header.get("resultCode", ""))

    if result_code == "00" or result_code == "":
        return None

    error_name = ERROR_CODE_MAP.get(result_code, "UNKNOWN_ERROR")
    result_msg = header.get("resultMsg", error_name)

    return ApiResponse(
        success=False,
        data=None,
        error_code=result_code,
        error_message=f"[{error_name}] {result_msg}",
        raw_status_code=200,
    )
```

### 2.3 XML 기본 응답 - JSON 변환 필요

대부분의 공공 API가 XML을 기본 응답으로 사용한다. 일부 API는 `type=json` 파라미터를 지원하지만, 에러 응답은 여전히 XML로 오는 경우가 많다. MCP 서버에서 일관되게 JSON으로 변환해야 한다.

```python
import xmltodict
import json


def parse_api_response(content: str, content_type: str) -> dict:
    """XML 또는 JSON 응답을 파이썬 딕셔너리로 변환한다."""
    if "xml" in content_type or content.strip().startswith("<"):
        parsed = xmltodict.parse(content)
        return json.loads(json.dumps(parsed))  # OrderedDict → dict 변환
    return json.loads(content)
```

### 2.4 CORS 미지원 - 서버사이드 필수

대부분의 공공 API가 CORS 헤더를 설정하지 않아 브라우저 JavaScript에서 직접 호출이 불가능하다. MCP 서버는 본질적으로 서버사이드에서 동작하므로 이 문제가 자연스럽게 해결된다. MCP 아키텍처 자체가 한국 공공 API의 CORS 문제에 대한 구조적 해답이다.

### 2.5 data.go.kr 패턴(A/B/C)별 처리 전략

data.go.kr은 세 가지 패턴으로 API를 제공하며, MCP 서버 구현 시 각 패턴에 맞는 전략이 필요하다.

| 패턴 | 설명 | MCP 서버 구현 전략 |
|------|------|------------------|
| **A: 완전 게이트웨이** | `apis.data.go.kr` 도메인으로 직접 호출 | data.go.kr serviceKey 하나로 Tool 구현 가능 |
| **B: 인덱스만** | 목록만 제공, 원본 사이트로 이동 | 원본 사이트 API를 직접 연동. 별도 인증키 관리 필요 |
| **C: 양쪽 제공 (범위 차이)** | 양쪽에 API가 있지만 버전/범위/키가 다름 | 원본 사이트 우선 사용 권장. 최신 기능과 높은 호출 한도 |

**패턴별 대표 API와 구현 시 주의사항**:

```
Pattern A (data.go.kr 키만으로 OK):
  - 국토교통부 실거래가, 에어코리아 대기정보, 국세청 사업자등록
  - → serviceKey 하나로 바로 구현 가능

Pattern B (원본 사이트 별도 가입 필수):
  - KIPRIS(특허), DART(전자공시), 워크넷(채용), 오피넷(유가)
  - → data.go.kr serviceKey로 호출하면 인증 실패
  - → 각 원본 사이트의 키 파라미터명이 다름 (crtfc_key, apiKey 등)

Pattern C (양쪽 제공, 가장 혼동 유발):
  - 기상청: data.go.kr은 기본 예보만, apihub.kma.go.kr에 전체 API
  - 관광공사: data.go.kr은 구버전, api.visitkorea.or.kr이 최신(4.0)
  - 도로명주소: 키 체계가 다름 (serviceKey vs confmKey)
```

---

## 3. MCP 서버 구축 단계

### Step 1: API 스펙 수동 분석

한국 공공 API는 OpenAPI 스펙을 제공하지 않으므로, HTML/PDF/HWP 문서를 직접 읽고 분석해야 한다.

**분석 체크리스트**:

```markdown
## API 스펙 분석 시트

### 기본 정보
- [ ] Base URL 확인 (도메인이 apis.data.go.kr인지, 원본 사이트인지)
- [ ] 해당 패턴 확인 (A/B/C)
- [ ] 인증키 파라미터명 (serviceKey? apiKey? crtfc_key? confmKey?)
- [ ] 인증키 전달 위치 (Query param? Path? Header?)

### 요청 스펙
- [ ] HTTP 메서드 (대부분 GET)
- [ ] 필수 파라미터 목록 및 타입
- [ ] 선택 파라미터 목록 및 타입
- [ ] 페이징 방식 (pageNo+numOfRows? 시작/끝 인덱스? 기간 기반?)
- [ ] 날짜 형식 (YYYYMMDD? YYYY-MM-DD?)

### 응답 스펙
- [ ] 기본 포맷 (XML? JSON 지원 여부?)
- [ ] 에러코드 체계 (resultCode 패턴? 자체 패턴?)
- [ ] 응답 필드명 및 타입 (한글 필드명 여부)
- [ ] 중첩 구조 확인 (items > item 패턴 등)

### 운영 정보
- [ ] 일일 호출 한도
- [ ] 호출 가능 시간 (24시간? 업무시간만?)
- [ ] 응답 시간 (일반적으로 1~5초)
```

### Step 2: Tool 정의 (function schema 작성)

MCP Tool은 AI가 이해할 수 있는 **구조화된 인터페이스**다. 한글 description을 사용하면 한국어 질문에 대한 Tool 선택 정확도가 높아진다.

```python
from mcp.server import Server
from mcp.types import Tool, TextContent
import mcp.server.stdio

server = Server("korean-public-api")


WEATHER_FORECAST_TOOL = Tool(
    name="get_weather_forecast",
    description=(
        "기상청 단기예보 조회. 특정 지역의 날씨 예보(기온, 강수확률, 하늘상태 등)를 반환한다. "
        "좌표는 기상청 격자 좌표(nx, ny)를 사용한다. "
        "서울 종로구는 nx=60, ny=127이다."
    ),
    inputSchema={
        "type": "object",
        "properties": {
            "base_date": {
                "type": "string",
                "description": "발표일자 (YYYYMMDD 형식, 예: 20260402)",
            },
            "base_time": {
                "type": "string",
                "description": (
                    "발표시각 (HHMM 형식). "
                    "가능한 값: 0200, 0500, 0800, 1100, 1400, 1700, 2000, 2300"
                ),
            },
            "nx": {
                "type": "integer",
                "description": "예보지점 X 격자 좌표 (예: 서울 종로구=60)",
            },
            "ny": {
                "type": "integer",
                "description": "예보지점 Y 격자 좌표 (예: 서울 종로구=127)",
            },
        },
        "required": ["base_date", "base_time", "nx", "ny"],
    },
)
```

### Step 3: 인증 처리 (serviceKey 관리)

API 키는 반드시 환경변수로 관리한다. 코드에 하드코딩하지 않는다.

```python
import os
from dataclasses import dataclass


@dataclass(frozen=True)
class ApiConfig:
    """API 설정을 담는 불변 객체."""
    service_key: str
    base_url: str
    daily_limit: int = 1000


def load_config() -> ApiConfig:
    """환경변수에서 API 설정을 로드한다.

    Raises:
        ValueError: 필수 환경변수가 누락된 경우
    """
    service_key = os.environ.get("DATA_GO_KR_SERVICE_KEY")
    if not service_key:
        raise ValueError(
            "DATA_GO_KR_SERVICE_KEY 환경변수가 설정되지 않았습니다. "
            "data.go.kr에서 발급받은 인증키를 설정해주세요."
        )
    return ApiConfig(
        service_key=service_key,
        base_url="http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0",
    )
```

### Step 4: 응답 정규화 (XML->JSON, 에러코드 매핑)

공공 API 응답을 AI가 소비하기 좋은 형태로 정규화한다.

```python
import httpx
import xmltodict
import json
import urllib.parse
from dataclasses import dataclass


@dataclass(frozen=True)
class NormalizedItem:
    """정규화된 예보 항목."""
    category: str
    category_name: str
    value: str
    forecast_date: str
    forecast_time: str


# 기상청 카테고리 코드 → 한글 매핑 (불변)
CATEGORY_NAMES: dict[str, str] = {
    "POP": "강수확률(%)",
    "PTY": "강수형태",
    "PCP": "1시간 강수량(mm)",
    "REH": "습도(%)",
    "SNO": "1시간 신적설(cm)",
    "SKY": "하늘상태",
    "TMP": "1시간 기온(C)",
    "TMN": "일 최저기온(C)",
    "TMX": "일 최고기온(C)",
    "UUU": "풍속(동서)(m/s)",
    "VVV": "풍속(남북)(m/s)",
    "WAV": "파고(M)",
    "VEC": "풍향(deg)",
    "WSD": "풍속(m/s)",
}

SKY_CODES: dict[str, str] = {
    "1": "맑음",
    "3": "구름많음",
    "4": "흐림",
}

PTY_CODES: dict[str, str] = {
    "0": "없음",
    "1": "비",
    "2": "비/눈",
    "3": "눈",
    "4": "소나기",
}


def normalize_forecast_response(raw_body: dict) -> list[NormalizedItem]:
    """기상청 응답을 정규화된 예보 항목 리스트로 변환한다."""
    items_wrapper = (
        raw_body.get("response", {})
        .get("body", {})
        .get("items", {})
        .get("item", [])
    )

    # 단일 항목인 경우 리스트로 변환
    if isinstance(items_wrapper, dict):
        items_wrapper = [items_wrapper]

    result = []
    for item in items_wrapper:
        category = item.get("category", "")
        value = str(item.get("fcstValue", ""))

        # 하늘상태, 강수형태는 코드를 한글로 변환
        if category == "SKY":
            value = SKY_CODES.get(value, value)
        elif category == "PTY":
            value = PTY_CODES.get(value, value)

        result.append(NormalizedItem(
            category=category,
            category_name=CATEGORY_NAMES.get(category, category),
            value=value,
            forecast_date=item.get("fcstDate", ""),
            forecast_time=item.get("fcstTime", ""),
        ))

    return result
```

### Step 5: 테스트 및 배포

**로컬 테스트** (MCP Inspector 사용):

```bash
# MCP Inspector로 서버 테스트
npx @modelcontextprotocol/inspector python weather_mcp_server.py
```

**Claude Desktop 연동** (`claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "korean-weather": {
      "command": "python",
      "args": ["/path/to/weather_mcp_server.py"],
      "env": {
        "DATA_GO_KR_SERVICE_KEY": "발급받은_인증키"
      }
    }
  }
}
```

**배포 옵션**:

| 방식 | 장점 | 단점 |
|------|------|------|
| 로컬 stdio | 설정 간단, 지연 없음 | 본인만 사용 가능 |
| Docker + SSE | 팀 공유 가능, 재현 가능 | 서버 운영 필요 |
| Cloudflare Workers | 무료 티어, 글로벌 | 콜드 스타트, 제약 있음 |

---

## 4. 코드 예제: 기상청 단기예보 MCP 서버

아래는 data.go.kr 기상청 단기예보 API를 MCP 서버로 래핑하는 **전체 작동 코드**다.

### 4.1 프로젝트 구조

```
weather-mcp-server/
  pyproject.toml
  src/
    weather_mcp/
      __init__.py
      server.py        # MCP 서버 진입점
      api_client.py    # 기상청 API 호출
      normalizer.py    # 응답 정규화
      config.py        # 설정 관리
      errors.py        # 에러 코드 매핑
```

### 4.2 config.py

```python
"""API 설정 관리 모듈."""

import os
from dataclasses import dataclass


@dataclass(frozen=True)
class WeatherApiConfig:
    """기상청 API 설정 (불변)."""
    service_key: str
    base_url: str = (
        "http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0"
    )
    timeout_seconds: int = 10
    max_retries: int = 2


def load_weather_config() -> WeatherApiConfig:
    """환경변수에서 기상청 API 설정을 로드한다.

    Raises:
        ValueError: DATA_GO_KR_SERVICE_KEY가 설정되지 않은 경우
    """
    service_key = os.environ.get("DATA_GO_KR_SERVICE_KEY")
    if not service_key:
        raise ValueError(
            "DATA_GO_KR_SERVICE_KEY 환경변수를 설정해주세요. "
            "https://www.data.go.kr 에서 '기상청_단기예보 조회서비스'를 "
            "활용신청한 뒤 발급받은 인코딩 키를 사용합니다."
        )
    return WeatherApiConfig(service_key=service_key)
```

### 4.3 errors.py

```python
"""data.go.kr 에러코드 매핑 모듈."""

from dataclasses import dataclass

# data.go.kr 공통 에러코드 (불변)
ERROR_CODE_MAP: dict[str, str] = {
    "00": "정상",
    "01": "어플리케이션 에러",
    "02": "데이터베이스 에러",
    "03": "데이터 없음",
    "04": "HTTP 에러",
    "05": "서비스 타임아웃",
    "10": "잘못된 요청 파라미터",
    "11": "필수 파라미터 누락",
    "12": "해당 OpenAPI 서비스 없음",
    "20": "서비스 접근 거부",
    "21": "일시적으로 사용 중지된 서비스키",
    "22": "일일 호출 한도 초과",
    "30": "등록되지 않은 서비스키",
    "31": "서비스키 유효기간 만료",
    "32": "등록되지 않은 IP",
    "33": "서명되지 않은 호출",
    "99": "알 수 없는 에러",
}


@dataclass(frozen=True)
class ApiError:
    """API 에러 정보 (불변)."""
    code: str
    message: str
    description: str

    def to_user_message(self) -> str:
        """사용자에게 보여줄 에러 메시지를 반환한다."""
        return f"API 오류 [{self.code}]: {self.description} - {self.message}"


def extract_error(parsed_body: dict) -> ApiError | None:
    """data.go.kr 형식의 응답에서 에러를 추출한다.

    정상 응답(코드 00)이면 None을 반환한다.
    """
    header = parsed_body.get("response", {}).get("header", {})
    result_code = str(header.get("resultCode", ""))

    if result_code == "00" or result_code == "":
        return None

    return ApiError(
        code=result_code,
        message=header.get("resultMsg", ""),
        description=ERROR_CODE_MAP.get(result_code, "알 수 없는 에러"),
    )
```

### 4.4 normalizer.py

```python
"""기상청 응답 정규화 모듈."""

import json
import xmltodict
from dataclasses import dataclass, asdict


# 기상청 카테고리 코드 매핑 (불변)
CATEGORY_NAMES: dict[str, str] = {
    "POP": "강수확률(%)",
    "PTY": "강수형태",
    "PCP": "1시간 강수량(mm)",
    "REH": "습도(%)",
    "SNO": "1시간 신적설(cm)",
    "SKY": "하늘상태",
    "TMP": "1시간 기온(C)",
    "TMN": "일 최저기온(C)",
    "TMX": "일 최고기온(C)",
    "UUU": "풍속(동서)(m/s)",
    "VVV": "풍속(남북)(m/s)",
    "WAV": "파고(M)",
    "VEC": "풍향(deg)",
    "WSD": "풍속(m/s)",
}

SKY_CODES: dict[str, str] = {"1": "맑음", "3": "구름많음", "4": "흐림"}
PTY_CODES: dict[str, str] = {
    "0": "없음", "1": "비", "2": "비/눈", "3": "눈", "4": "소나기",
}


@dataclass(frozen=True)
class ForecastItem:
    """정규화된 단기예보 항목."""
    category: str
    category_name: str
    value: str
    forecast_date: str
    forecast_time: str
    nx: int
    ny: int


def parse_response_body(content: str, content_type: str = "") -> dict:
    """XML 또는 JSON 응답을 딕셔너리로 파싱한다."""
    stripped = content.strip()
    if "xml" in content_type or stripped.startswith("<"):
        parsed = xmltodict.parse(stripped)
        # OrderedDict -> dict 변환
        return json.loads(json.dumps(parsed))
    return json.loads(stripped)


def normalize_forecast(parsed_body: dict) -> list[ForecastItem]:
    """기상청 단기예보 응답을 정규화된 리스트로 변환한다."""
    items_raw = (
        parsed_body.get("response", {})
        .get("body", {})
        .get("items", {})
        .get("item", [])
    )

    # 단일 항목일 때 리스트로 통일
    if isinstance(items_raw, dict):
        items_raw = [items_raw]

    result: list[ForecastItem] = []
    for item in items_raw:
        category = item.get("category", "")
        value = str(item.get("fcstValue", ""))

        if category == "SKY":
            value = SKY_CODES.get(value, value)
        elif category == "PTY":
            value = PTY_CODES.get(value, value)

        result.append(ForecastItem(
            category=category,
            category_name=CATEGORY_NAMES.get(category, category),
            value=value,
            forecast_date=item.get("fcstDate", ""),
            forecast_time=item.get("fcstTime", ""),
            nx=int(item.get("nx", 0)),
            ny=int(item.get("ny", 0)),
        ))

    return result


def forecast_items_to_summary(items: list[ForecastItem]) -> str:
    """예보 항목 리스트를 AI가 읽기 좋은 텍스트 요약으로 변환한다."""
    if not items:
        return "해당 조건에 맞는 예보 데이터가 없습니다."

    # 시간대별로 그룹핑
    time_groups: dict[str, list[ForecastItem]] = {}
    for item in items:
        key = f"{item.forecast_date} {item.forecast_time}"
        group = time_groups.get(key, [])
        time_groups[key] = [*group, item]

    lines: list[str] = []
    for time_key in sorted(time_groups.keys()):
        group = time_groups[time_key]
        lines.append(f"\n[{time_key[:8]} {time_key[9:11]}:{time_key[11:]}]")
        for item in group:
            lines.append(f"  {item.category_name}: {item.value}")

    nx = items[0].nx
    ny = items[0].ny
    header = f"기상청 단기예보 (격자좌표: nx={nx}, ny={ny})"

    return f"{header}\n{''.join(lines)}"
```

### 4.5 api_client.py

```python
"""기상청 API 호출 클라이언트 모듈."""

import urllib.parse
import httpx

from .config import WeatherApiConfig
from .errors import ApiError, extract_error
from .normalizer import (
    ForecastItem,
    parse_response_body,
    normalize_forecast,
)


def _build_url(
    config: WeatherApiConfig,
    endpoint: str,
    params: dict[str, str | int],
) -> str:
    """serviceKey 이중인코딩을 방지하는 URL을 생성한다."""
    query_parts = [f"serviceKey={config.service_key}"]
    for key, value in params.items():
        encoded_value = urllib.parse.quote(str(value))
        query_parts.append(f"{key}={encoded_value}")
    return f"{config.base_url}/{endpoint}?{'&'.join(query_parts)}"


async def fetch_short_term_forecast(
    config: WeatherApiConfig,
    base_date: str,
    base_time: str,
    nx: int,
    ny: int,
    num_of_rows: int = 1000,
) -> list[ForecastItem] | ApiError:
    """기상청 단기예보를 조회한다.

    Args:
        config: API 설정
        base_date: 발표일자 (YYYYMMDD)
        base_time: 발표시각 (HHMM)
        nx: 예보지점 X 좌표
        ny: 예보지점 Y 좌표
        num_of_rows: 한 페이지 결과 수

    Returns:
        성공 시 ForecastItem 리스트, 실패 시 ApiError
    """
    url = _build_url(config, "getVilageFcst", {
        "pageNo": 1,
        "numOfRows": num_of_rows,
        "dataType": "XML",
        "base_date": base_date,
        "base_time": base_time,
        "nx": nx,
        "ny": ny,
    })

    async with httpx.AsyncClient(timeout=config.timeout_seconds) as client:
        response = await client.get(url)

    # HTTP 레벨 에러 체크 (드물지만 가능)
    if response.status_code != 200:
        return ApiError(
            code=str(response.status_code),
            message=f"HTTP {response.status_code}",
            description="서버 통신 오류",
        )

    # 응답 파싱
    content_type = response.headers.get("content-type", "")
    parsed = parse_response_body(response.text, content_type)

    # body 내부 에러코드 체크 (HTTP 200이지만 실제 에러인 경우)
    error = extract_error(parsed)
    if error is not None:
        return error

    return normalize_forecast(parsed)
```

### 4.6 server.py (MCP 서버 진입점)

```python
"""기상청 단기예보 MCP 서버."""

import asyncio
import json
import re
from dataclasses import asdict

from mcp.server import Server
from mcp.types import Tool, TextContent
import mcp.server.stdio

from .config import load_weather_config
from .api_client import fetch_short_term_forecast
from .errors import ApiError
from .normalizer import forecast_items_to_summary


server = Server("weather-korea")
config = load_weather_config()


# --- 입력 검증 ---

DATE_PATTERN = re.compile(r"^\d{8}$")
TIME_PATTERN = re.compile(r"^\d{4}$")
VALID_BASE_TIMES = {"0200", "0500", "0800", "1100", "1400", "1700", "2000", "2300"}


def validate_forecast_input(
    base_date: str, base_time: str, nx: int, ny: int,
) -> str | None:
    """입력값을 검증한다. 에러가 있으면 메시지를 반환, 없으면 None."""
    if not DATE_PATTERN.match(base_date):
        return f"base_date는 YYYYMMDD 형식이어야 합니다. 입력값: {base_date}"
    if not TIME_PATTERN.match(base_time):
        return f"base_time은 HHMM 형식이어야 합니다. 입력값: {base_time}"
    if base_time not in VALID_BASE_TIMES:
        return (
            f"base_time은 다음 중 하나여야 합니다: "
            f"{', '.join(sorted(VALID_BASE_TIMES))}. 입력값: {base_time}"
        )
    if not (1 <= nx <= 149):
        return f"nx는 1~149 범위여야 합니다. 입력값: {nx}"
    if not (1 <= ny <= 253):
        return f"ny는 1~253 범위여야 합니다. 입력값: {ny}"
    return None


# --- Tool 정의 ---

FORECAST_TOOL = Tool(
    name="get_weather_forecast",
    description=(
        "기상청 단기예보 조회. 특정 지역의 날씨 예보(기온, 강수확률, 하늘상태 등)를 "
        "반환한다. 좌표는 기상청 격자 좌표(nx, ny)를 사용한다. "
        "주요 지역 좌표: 서울(60,127), 부산(98,76), 대구(89,90), "
        "인천(55,124), 광주(58,74), 대전(67,100), 제주(52,38)"
    ),
    inputSchema={
        "type": "object",
        "properties": {
            "base_date": {
                "type": "string",
                "description": "발표일자 (YYYYMMDD, 예: 20260402)",
            },
            "base_time": {
                "type": "string",
                "description": (
                    "발표시각 (HHMM). "
                    "가능한 값: 0200, 0500, 0800, 1100, 1400, 1700, 2000, 2300"
                ),
            },
            "nx": {
                "type": "integer",
                "description": "예보지점 X 격자좌표 (1~149, 서울=60)",
            },
            "ny": {
                "type": "integer",
                "description": "예보지점 Y 격자좌표 (1~253, 서울=127)",
            },
        },
        "required": ["base_date", "base_time", "nx", "ny"],
    },
)

GRID_LOOKUP_TOOL = Tool(
    name="lookup_grid_coordinates",
    description=(
        "주요 도시의 기상청 격자 좌표(nx, ny)를 조회한다. "
        "get_weather_forecast 호출 전에 좌표를 모를 때 사용한다."
    ),
    inputSchema={
        "type": "object",
        "properties": {
            "city": {
                "type": "string",
                "description": (
                    "도시명 (한글). 예: 서울, 부산, 대구, 인천, 광주, "
                    "대전, 울산, 세종, 수원, 제주"
                ),
            },
        },
        "required": ["city"],
    },
)

# 주요 도시 격자좌표 (불변)
CITY_GRID_MAP: dict[str, tuple[int, int]] = {
    "서울": (60, 127),
    "부산": (98, 76),
    "대구": (89, 90),
    "인천": (55, 124),
    "광주": (58, 74),
    "대전": (67, 100),
    "울산": (102, 84),
    "세종": (66, 103),
    "수원": (60, 121),
    "청주": (69, 107),
    "전주": (63, 89),
    "춘천": (73, 134),
    "강릉": (92, 131),
    "제주": (52, 38),
    "포항": (102, 94),
    "창원": (90, 77),
    "김해": (95, 77),
    "고양": (57, 128),
    "성남": (63, 124),
    "용인": (64, 119),
}


# --- Tool 핸들러 ---

@server.list_tools()
async def list_tools() -> list[Tool]:
    return [FORECAST_TOOL, GRID_LOOKUP_TOOL]


@server.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "get_weather_forecast":
        return await handle_forecast(arguments)
    elif name == "lookup_grid_coordinates":
        return handle_grid_lookup(arguments)
    else:
        return [TextContent(type="text", text=f"알 수 없는 Tool: {name}")]


async def handle_forecast(arguments: dict) -> list[TextContent]:
    """단기예보 조회 Tool 핸들러."""
    base_date = arguments.get("base_date", "")
    base_time = arguments.get("base_time", "")
    nx = arguments.get("nx", 0)
    ny = arguments.get("ny", 0)

    # 입력 검증
    validation_error = validate_forecast_input(base_date, base_time, nx, ny)
    if validation_error is not None:
        return [TextContent(type="text", text=f"입력 오류: {validation_error}")]

    # API 호출
    result = await fetch_short_term_forecast(
        config=config,
        base_date=base_date,
        base_time=base_time,
        nx=nx,
        ny=ny,
    )

    # 에러 처리
    if isinstance(result, ApiError):
        return [TextContent(type="text", text=result.to_user_message())]

    # 성공 응답
    summary = forecast_items_to_summary(result)
    return [TextContent(type="text", text=summary)]


def handle_grid_lookup(arguments: dict) -> list[TextContent]:
    """격자좌표 조회 Tool 핸들러."""
    city = arguments.get("city", "").strip()

    if not city:
        return [TextContent(type="text", text="도시명을 입력해주세요.")]

    coords = CITY_GRID_MAP.get(city)
    if coords is not None:
        nx, ny = coords
        return [TextContent(
            type="text",
            text=f"{city}의 기상청 격자좌표: nx={nx}, ny={ny}",
        )]

    # 부분 일치 검색
    matches = [
        (name, xy)
        for name, xy in CITY_GRID_MAP.items()
        if city in name or name in city
    ]

    if matches:
        lines = [f"'{city}'에 대한 유사 결과:"]
        for name, (nx, ny) in matches:
            lines.append(f"  {name}: nx={nx}, ny={ny}")
        return [TextContent(type="text", text="\n".join(lines))]

    available = ", ".join(sorted(CITY_GRID_MAP.keys()))
    return [TextContent(
        type="text",
        text=f"'{city}'를 찾을 수 없습니다. 지원 도시: {available}",
    )]


# --- 실행 ---

async def main():
    async with mcp.server.stdio.stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            server.create_initialization_options(),
        )


if __name__ == "__main__":
    asyncio.run(main())
```

### 4.7 pyproject.toml

```toml
[project]
name = "weather-mcp-server"
version = "0.1.0"
description = "기상청 단기예보 MCP 서버"
requires-python = ">=3.10"
dependencies = [
    "mcp>=1.0.0",
    "httpx>=0.27.0",
    "xmltodict>=0.13.0",
]

[project.scripts]
weather-mcp = "weather_mcp.server:main"
```

### 4.8 실행 방법

```bash
# 1. 의존성 설치
pip install -e .

# 2. 환경변수 설정
export DATA_GO_KR_SERVICE_KEY="발급받은_인코딩_키"

# 3. MCP Inspector로 테스트
npx @modelcontextprotocol/inspector python -m weather_mcp.server

# 4. Claude Desktop 설정 (claude_desktop_config.json)
# {
#   "mcpServers": {
#     "korean-weather": {
#       "command": "python",
#       "args": ["-m", "weather_mcp.server"],
#       "env": {
#         "DATA_GO_KR_SERVICE_KEY": "발급받은_인코딩_키"
#       }
#     }
#   }
# }
```

---

## 5. 기존 MCP 서버 목록

현재 한국 공공 API를 대상으로 한 MCP 서버 프로젝트 7개가 확인된다.

### 5.1 공공 데이터 포털 (data.go.kr)

| 프로젝트 | GitHub Stars | 대상 API | 특징 |
|---------|-------------|---------|------|
| **data-go-mcp-servers** | 233 | data.go.kr 6개 API (기상, 미세먼지, 관광 등) | 가장 많이 사용되는 프로젝트. 6개 API를 하나의 MCP 서버에 통합 |
| **opendata-mcp** | 9 | data.go.kr 범용 탐색 | 특정 API가 아닌 data.go.kr 자체를 탐색하는 메타 MCP 서버 |

### 5.2 금융/기업 데이터

| 프로젝트 | GitHub Stars | 대상 API | 특징 |
|---------|-------------|---------|------|
| **dartpoint-mcp** | 3 | DART 전자공시 시스템 | 기업 재무정보, 공시 검색. DART API(Pattern B)를 래핑 |

### 5.3 도서관/문화

| 프로젝트 | GitHub Stars | 대상 API | 특징 |
|---------|-------------|---------|------|
| **data4library-mcp** | 5 | 도서관 정보나루 | 도서관 소장 도서, 대출 인기 도서 검색 |

### 5.4 민간 API (참고)

| 프로젝트 | GitHub Stars | 대상 API | 특징 |
|---------|-------------|---------|------|
| **naver-search-mcp** | 59 | 네이버 검색 API | 공공 API는 아니지만, 한국 API MCP 패턴의 참고 사례 |

### 5.5 Python 라이브러리 (MCP 서버 구축 시 활용 가능)

직접적인 MCP 서버는 아니지만, MCP 서버의 백엔드 로직으로 활용할 수 있는 성숙한 라이브러리들이다.

| 라이브러리 | Stars | 대상 | MCP 래핑 적합도 |
|-----------|-------|------|----------------|
| **PublicDataReader** | 557 | 16+ 공공데이터 소스 통합 | 높음 - 이미 정규화된 인터페이스 제공 |
| **FinanceDataReader** | 1,400 | 한국/글로벌 금융 | 높음 - 주가, 환율 등 |
| **pykrx** | 954 | KRX 거래소 | 중간 - 거래소 특화 |
| **dart-fss** | 360 | DART 전자공시 | 높음 - 재무제표 분석 |

---

## 6. MCP 서버가 없는 주요 API (기회 영역)

현재 MCP 커버리지는 전체 공공 API의 **5% 미만**이다. 아래는 수요가 높지만 MCP 서버가 아직 없는 API들이다.

### 6.1 경제/통계 (수요 매우 높음)

| API | 기관 | 주요 데이터 | 난이도 | 비고 |
|-----|------|-----------|--------|------|
| **KOSIS** | 통계청 | GDP, 인구, 물가 등 국가통계 전체 | 중 | JSON 지원. 기간 기반 페이징. apiKey 사용 |
| **ECOS** | 한국은행 | 기준금리, 환율, 통화량, 경제지표 | 중 | JSON 지원. Path에 키 포함. 시작/끝 인덱스 페이징 |
| **KRX 정보데이터** | 한국거래소 | 주가, 시가총액, 거래량, 지수 | 상 | 수동승인 1~7일. 자체 인증 체계 |

### 6.2 부동산 (수요 높음)

| API | 기관 | 주요 데이터 | 난이도 | 비고 |
|-----|------|-----------|--------|------|
| **부동산 실거래가** | 국토교통부 | 아파트/단독/오피스텔 매매/전월세 실거래 | 하 | Pattern A. data.go.kr 키로 바로 호출 가능 |
| **건축물대장** | 국토교통부 | 건물 정보, 층수, 면적 | 하 | Pattern A/C |
| **공시지가** | 국토교통부 | 토지 공시가격 | 중 | Pattern A |

### 6.3 생활/환경

| API | 기관 | 주요 데이터 | 난이도 | 비고 |
|-----|------|-----------|--------|------|
| **기상청 전체** | 기상청 | 중기예보, 레이더, 위성, 특보, 해양 | 중 | Pattern C. apihub.kma.go.kr에 전체 API |
| **에어코리아** | 환경부 | 실시간 대기질, 미세먼지, 오존 | 하 | Pattern A. data.go.kr 키 사용 가능 |
| **서울 열린데이터** | 서울시 | 지하철, 버스, 공원, 문화행사 | 중 | 독립 인증(서울 API Key). Path에 키 포함 |

### 6.4 교통

| API | 기관 | 주요 데이터 | 난이도 | 비고 |
|-----|------|-----------|--------|------|
| **TAGO** | 국토교통부 | 전국 대중교통 노선, 도착정보 | 중 | Pattern A |
| **ITS** | 국토교통부 | 실시간 도로 교통정보, CCTV | 중 | 독립 인증 |

### 6.5 취업/노동

| API | 기관 | 주요 데이터 | 난이도 | 비고 |
|-----|------|-----------|--------|------|
| **워크넷** | 고용노동부 | 채용공고, 직업훈련, 고용동향 | 중 | Pattern B. 원본 사이트 별도 가입 필요 |

### 6.6 뉴스/미디어

| API | 기관 | 주요 데이터 | 난이도 | 비고 |
|-----|------|-----------|--------|------|
| **빅카인즈** | 한국언론진흥재단 | 뉴스 기사 검색, 트렌드 분석 | 중 | 독립 포털. 일부 기능 유료 |

### 6.7 우선 구현 추천 (영향도 x 난이도 기준)

| 순위 | API | 이유 |
|------|-----|------|
| 1 | **부동산 실거래가** | 수요 최상위. Pattern A라 구현 쉬움. data.go.kr 키만으로 OK |
| 2 | **ECOS (한국은행)** | 경제지표 수요 높음. JSON 지원. 자동승인 |
| 3 | **KOSIS (통계청)** | 국가통계 전체 접근. JSON 지원. 자동승인 |
| 4 | **에어코리아** | 생활 밀착 데이터. Pattern A. 구현 쉬움 |
| 5 | **기상청 전체** | 기존 MCP 서버(data-go-mcp-servers)가 기본 예보만 커버. 확장 필요 |
| 6 | **서울 열린데이터** | 서울 거주 개발자 수요 높음. 다양한 생활 데이터 |
| 7 | **워크넷** | AI 채용 에이전트 구축에 핵심 |

---

## 부록: 포털별 인증키 파라미터 치트시트

MCP 서버 구현 시 가장 자주 혼동되는 인증키 파라미터명과 위치를 정리한다.

| 포털 | 파라미터명 | 전달 위치 | 페이징 방식 | 에러코드 체계 |
|------|-----------|----------|-----------|-------------|
| data.go.kr | `serviceKey` | Query param | pageNo + numOfRows | resultCode (00~99) |
| 서울 열린데이터 | (없음, URL path) | Path segment | 시작/끝 인덱스 | CODE (INFO-xxx) |
| ECOS (한국은행) | (없음, URL path) | Path segment | 시작/끝 인덱스 | 자체 |
| KOSIS (통계청) | `apiKey` | Query param | 기간 기반 | 자체 |
| DART (금감원) | `crtfc_key` | Query param | page_no + page_count | status (0xx) |
| 도로명주소 | `confmKey` | Query param | currentPage + countPerPage | 자체 |
| 식품안전나라 | `keyId` | Path segment | 시작/끝 인덱스 | 자체 |
| TourAPI (관광공사) | `serviceKey` | Query param | pageNo + numOfRows | resultCode |

> 이 가이드의 코드 예제는 기상청 단기예보를 기준으로 작성되었지만, 동일한 패턴을 다른 data.go.kr Pattern A API에 그대로 적용할 수 있다. Pattern B/C API는 각 원본 사이트의 인증 방식과 응답 포맷에 맞게 `api_client.py`와 `normalizer.py`를 수정하면 된다.

---

## 부록: MCP 서버 현황 업데이트 (2026-04-02)

### 현재 확인된 MCP 서버 (7개)

| 프로젝트 | 대상 API | 비고 |
|---------|---------|------|
| Koomook/data-go-mcp-servers | data.go.kr 6개 API (기상 포함) | Python MCP, 233 stars |
| ceami/opendata-mcp | data.go.kr 범용 탐색 | TypeScript MCP |
| dartpointai/dartpoint-mcp | DART 전자공시 | MCP |
| isnow890/data4library-mcp | 도서관 정보나루 25+ 도구 | TypeScript MCP |
| isnow890/naver-search-mcp | 네이버 검색+트렌드 | Node.js MCP |
| pinnaclesoft-ko/be-node-seoul-data-mcp | 서울 열린데이터 | TypeScript MCP |
| (data-go-mcp-servers 포함) | 기상청 API허브 | 단기예보 등 |

### MCP가 필요한 우선순위 포털 (미개발)

KOSIS, ECOS, 부동산 실거래가, 워크넷, TourAPI, 빅카인즈, KIPRIS 등 — 본 가이드의 패턴을 그대로 적용하여 개발 가능.
