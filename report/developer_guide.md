# 한국 공공 Open API 개발자 온보딩 가이드

> **대상**: 한국 공공 API를 처음 사용하는 개발자
> **최종 업데이트**: 2026-04-02

---

## 1. 시작하기 전에

### 한국 공공 API 생태계 개요

한국의 공공 데이터는 **173개 이상의 포털**에 분산되어 있다. 미국의 `api.data.gov`나 영국의 `gov.uk` 같은 단일 진입점이 없으며, 기관마다 독립적인 API 포털을 운영한다.

| 구분 | 포털 수 | 예시 |
|------|---------|------|
| 중앙정부 | 11개 | data.go.kr, 정부24, 외교부 등 |
| 입법/사법 | 7개 | 국회, 법제처, 대법원 |
| 금융 | 9개 | DART, ECOS, KRX |
| 17개 시/도 지자체 | 17개 | 서울~제주 각각 독립 포털 |
| 전문분야 독립 | 67개 | 기상청, 특허청, ITS, TAGO 등 |
| 공기업/준정부기관 | 62개 | 관광공사, 오피넷, 워크넷 등 |

**핵심 포인트**:
- 모든 포털이 **회원가입 + API Key 발급**을 요구한다. Key 없이 쓸 수 있는 공공 API는 사실상 없다.
- `data.go.kr` 1개 ID로 전체의 약 40~70%를 커버할 수 있다 (패턴에 따라 다름).
- 모든 포털을 완전히 커버하려면 **30~40개 ID**가 필요하다.
- 포털마다 인증키 파라미터명, 페이징 방식, 에러코드 체계가 **전부 다르다**.

### data.go.kr의 3가지 제공 패턴

data.go.kr이 모든 API의 게이트웨이가 아니다. 반드시 어떤 패턴인지 확인해야 한다.

| 패턴 | 설명 | data.go.kr 키로 호출 | 별도 가입 |
|------|------|:---:|:---:|
| **A: 완전 게이트웨이** | `apis.data.go.kr` 도메인으로 직접 호출 | O | X |
| **B: 인덱스(카탈로그)만** | 목록만 제공, 원본 사이트로 이동 필요 | X | O |
| **C: 양쪽 제공 (범위 차이)** | 양쪽에 API가 있지만 버전/범위/키가 다름 | 부분적 | O |

- **패턴 A 예시**: 국토부 실거래가, 에어코리아, 소상공인 상가정보
- **패턴 B 예시**: KIPRIS(특허), 오피넷(유가), DART(전자공시), 서울열린데이터
- **패턴 C 예시**: TourAPI(관광공사), 기상청, 도로명주소, 식품안전나라

---

## 2. 첫 번째 API 호출하기 (5분 퀵스타트)

### Step 1: data.go.kr 가입

1. [data.go.kr](https://www.data.go.kr) 접속
2. 회원가입 (이메일 또는 소셜 로그인)
3. 이메일 인증 완료

### Step 2: API 활용 신청

1. 원하는 API 검색 (예: "단기예보")
2. "활용신청" 버튼 클릭
3. 활용 목적 간단히 작성 후 제출
4. **자동승인 API는 즉시**, 수동승인 API는 1~7일 소요

### Step 3: 인증키 확인

1. 마이페이지 > 활용신청 현황
2. **일반 인증키(Encoding)** 복사 (Decoding 키가 아님!)
3. 키에 `+`, `/`, `=` 같은 특수문자가 포함되어 있음을 확인

### Step 4: 첫 호출

#### curl 예제 (기상청 단기예보)

```bash
# 인증키를 변수에 저장 (자신의 키로 교체)
SERVICE_KEY="여기에_발급받은_인코딩키_입력"

# 단기예보 조회 (서울 종로구 기준)
curl -s "http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst?\
serviceKey=${SERVICE_KEY}\
&pageNo=1\
&numOfRows=10\
&dataType=JSON\
&base_date=20260402\
&base_time=0500\
&nx=60\
&ny=127"
```

> **중요**: `dataType=JSON`을 반드시 추가하라. 기본값은 XML이다.

#### Python 예제

```python
import requests
from urllib.parse import quote

# 인증키 (Encoding 키 사용)
SERVICE_KEY = "여기에_발급받은_인코딩키_입력"

url = "http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst"
params = {
    "serviceKey": SERVICE_KEY,  # requests가 자동 인코딩하므로 Decoding 키 사용 시 이중인코딩 발생
    "pageNo": 1,
    "numOfRows": 10,
    "dataType": "JSON",
    "base_date": "20260402",
    "base_time": "0500",
    "nx": 60,
    "ny": 127,
}

response = requests.get(url, params=params)
data = response.json()

# 주의: HTTP 200이어도 에러일 수 있다!
header = data.get("response", {}).get("header", {})
if header.get("resultCode") != "00":
    print(f"API 에러: {header.get('resultMsg')}")
else:
    items = data["response"]["body"]["items"]["item"]
    for item in items:
        print(f"{item['category']}: {item['fcstValue']} ({item['fcstDate']} {item['fcstTime']})")
```

### serviceKey 이중인코딩 주의사항

data.go.kr이 발급하는 인증키에는 Base64 특수문자(`+`, `/`, `=`)가 포함되어 있다. HTTP 클라이언트가 자동으로 URL 인코딩을 수행하면 키가 이중으로 인코딩되어 인증 실패가 발생한다.

```python
# 방법 1: Decoding 키 사용 + requests의 params 활용 (권장)
# requests가 자동으로 1회 인코딩하므로 Decoding 키를 넣으면 정확히 1회 인코딩됨
params = {"serviceKey": "디코딩키를여기에"}
response = requests.get(url, params=params)

# 방법 2: Encoding 키 사용 + URL에 직접 삽입
# 이미 인코딩된 키를 URL에 직접 붙이므로 추가 인코딩 없음
encoded_key = "인코딩키를여기에"
response = requests.get(f"{url}?serviceKey={encoded_key}&pageNo=1&...")

# 잘못된 방법: Encoding 키를 params에 넣으면 이중인코딩 발생!
# params = {"serviceKey": "인코딩키"}  <-- 이렇게 하면 안 됨
```

> **팁**: 인증 실패 에러가 나면 가장 먼저 이중인코딩을 의심하라. 관련 블로그 글이 수백 개에 달하는 악명 높은 문제다.

---

## 3. 핵심 주의사항 10가지

### 1. HTTP 200이 에러일 수 있다

한국 공공 API의 가장 심각한 특징이다. 인증 실패, 파라미터 오류, 호출 초과 **모두 HTTP 200 OK를 반환**한다. 반드시 응답 body 안의 `resultCode`를 확인해야 한다.

```xml
HTTP/1.1 200 OK     <!-- 실제로는 인증 실패 -->

<response>
  <header>
    <resultCode>30</resultCode>
    <resultMsg>SERVICE_KEY_IS_NOT_REGISTERED_ERROR</resultMsg>
  </header>
</response>
```

```python
# 올바른 에러 처리 패턴
def call_data_go_kr(url, params):
    response = requests.get(url, params=params)
    response.raise_for_status()  # 네트워크 에러만 잡힘

    data = response.json()
    header = data.get("response", {}).get("header", {})
    result_code = header.get("resultCode", "")

    if result_code != "00":
        raise ValueError(
            f"API 에러 [{result_code}]: {header.get('resultMsg', '알 수 없는 에러')}"
        )

    return data["response"]["body"]
```

### 2. data.go.kr 키와 원본 사이트 키는 다르다

data.go.kr에서 발급받은 `serviceKey`로 원본 사이트 API를 호출하면 **인증 실패**한다. 특히 다음 서비스에서 자주 발생한다:

| 서비스 | data.go.kr 키 파라미터 | 원본 사이트 키 파라미터 |
|--------|----------------------|----------------------|
| 도로명주소 | `serviceKey` | `confmKey` |
| 식품안전나라 | `serviceKey` | 자체 키 |
| KIPRIS(특허) | 사용 불가 | `serviceKey` (별도 발급) |

### 3. API Key 2년 만료 + 수동 갱신

data.go.kr 인증키는 **2년 유효기간**이 있으며 자동 갱신되지 않는다. 만료를 놓치면 서비스가 중단된다.

```python
# 키 만료일 관리 예시
from datetime import datetime, timedelta

KEY_ISSUED_DATE = datetime(2026, 4, 2)
KEY_EXPIRY_DATE = KEY_ISSUED_DATE + timedelta(days=730)  # 2년

def check_key_expiry():
    days_remaining = (KEY_EXPIRY_DATE - datetime.now()).days
    if days_remaining <= 30:
        print(f"경고: API Key가 {days_remaining}일 후 만료됩니다. data.go.kr에서 갱신하세요.")
    return days_remaining
```

### 4. CORS 미지원 - 프론트엔드 직접 호출 불가

대부분의 공공 API가 CORS 헤더를 설정하지 않는다. 브라우저 JavaScript에서 직접 호출하면 차단된다.

```javascript
// 이렇게 하면 CORS 에러 발생
fetch("http://apis.data.go.kr/...")  // 브라우저에서 차단됨
```

**해결 방법**: 반드시 백엔드 프록시를 경유해야 한다.

```python
# Flask 프록시 예시
from flask import Flask, jsonify, request
from flask_cors import CORS
import requests as req

app = Flask(__name__)
CORS(app)

@app.route("/api/proxy/<path:endpoint>")
def proxy(endpoint):
    params = dict(request.args)
    params["serviceKey"] = "서버에_저장된_키"
    response = req.get(f"http://apis.data.go.kr/{endpoint}", params=params)
    return jsonify(response.json())
```

### 5. XML이 기본 - dataType=JSON 파라미터 필수

거의 모든 data.go.kr API의 기본 응답이 XML이다. JSON으로 받으려면 명시적으로 요청해야 한다.

```python
params = {
    "serviceKey": KEY,
    "dataType": "JSON",  # 이 줄을 빠뜨리면 XML이 온다
    # ...
}
```

> 에러 응답은 `dataType=JSON`을 지정해도 **XML로만 반환**되는 경우가 있다. XML 파싱 코드도 준비하라.

### 6. 트래픽 일 1,000회 제한

data.go.kr 경유 API는 기본적으로 **일 1,000회** 호출 제한이 걸린다. 증량 신청은 가능하지만 승인에 3일~2주가 소요된다.

- 원본 사이트에서 직접 발급하면 더 높은 한도를 제공하는 경우가 많다.
- `X-RateLimit-Remaining` 같은 표준 헤더가 **없다**. 남은 호출 횟수를 미리 알 수 없다.
- 초과하면 body에 에러코드가 담긴 HTTP 200 응답이 온다.

### 7. 에러코드 체계가 포털마다 다르다

범용 에러 처리 코드를 작성할 수 없다. 포털마다 별도 파서가 필요하다.

| 포털 | 성공 코드 | 에러 형식 |
|------|----------|----------|
| data.go.kr | `resultCode: "00"` | `resultCode: "01"~"99"` |
| 서울 열린데이터 | `CODE: "INFO-000"` | `CODE: "INFO-xxx"` |
| DART | `status: "000"` | `status: "010"~"800"` |
| ECOS | 자체 코드 | 자체 형식 |

### 8. 새벽 점검/장애가 빈번하다

공공 API는 사전 공지 없이 장애가 발생하거나 새벽 시간대에 점검이 진행되는 경우가 흔하다.

```python
# 재시도 로직 권장
import time

def call_with_retry(url, params, max_retries=3, delay=2):
    for attempt in range(max_retries):
        try:
            response = requests.get(url, params=params, timeout=10)
            data = response.json()
            header = data.get("response", {}).get("header", {})
            if header.get("resultCode") == "00":
                return data
            if header.get("resultCode") in ("99", "04"):  # 서버 오류, 점검
                time.sleep(delay * (attempt + 1))
                continue
            return data  # 다른 에러는 재시도 불필요
        except (requests.exceptions.Timeout, requests.exceptions.ConnectionError):
            time.sleep(delay * (attempt + 1))
    raise RuntimeError(f"{max_retries}회 재시도 후에도 API 호출 실패")
```

### 9. data.go.kr 패턴 A/B/C 확인 필수

API를 사용하기 전에 반드시 해당 API가 어떤 패턴인지 확인하라 (1장 참조).

- **패턴 A**: data.go.kr 키로 바로 호출 가능
- **패턴 B**: data.go.kr에 목록만 있고, 원본 사이트에 별도 가입 필요
- **패턴 C**: 양쪽 다 있지만 버전/기능/키가 다름

확인 방법:
1. API 상세 페이지에서 호출 URL 확인 (`apis.data.go.kr`이면 패턴 A)
2. "활용신청" 시 외부 사이트로 이동하면 패턴 B
3. 블로그 검색으로 다른 개발자 경험 참고

### 10. 공공누리 라이선스 유형 확인

공공 데이터는 무조건 자유롭게 쓸 수 있는 것이 아니다. **공공누리 4가지 유형**을 확인해야 한다.

| 유형 | 출처표시 | 상업적 이용 | 변경 가능 |
|------|---------|-----------|----------|
| 제1유형 | O | O | O |
| 제2유형 | O | O | X |
| 제3유형 | O | X | O |
| 제4유형 | O | X | X |

> API 상세 페이지 하단에서 라이선스 유형을 확인할 수 있다. 상업 서비스를 계획한다면 반드시 제1유형 또는 제2유형인지 확인하라.

---

## 4. 분야별 추천 API (초보자용)

### 기상: 기상청 단기예보 API

전국 날씨를 격자(nx, ny) 좌표로 조회. 가장 많이 사용되는 공공 API 중 하나.

```python
import requests

SERVICE_KEY = "디코딩키"

def get_weather_forecast(base_date, base_time, nx, ny):
    url = "http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst"
    params = {
        "serviceKey": SERVICE_KEY,
        "pageNo": 1,
        "numOfRows": 100,
        "dataType": "JSON",
        "base_date": base_date,  # "20260402"
        "base_time": base_time,  # "0500" (02, 05, 08, 11, 14, 17, 20, 23시 발표)
        "nx": nx,                # 격자 X (서울 종로구: 60)
        "ny": ny,                # 격자 Y (서울 종로구: 127)
    }
    response = requests.get(url, params=params)
    data = response.json()

    header = data["response"]["header"]
    if header["resultCode"] != "00":
        raise ValueError(f"에러: {header['resultMsg']}")

    return data["response"]["body"]["items"]["item"]

# 사용
items = get_weather_forecast("20260402", "0500", 60, 127)
for item in items:
    print(f"{item['category']}: {item['fcstValue']}")
# TMP: 온도, REH: 습도, SKY: 하늘상태, POP: 강수확률
```

> **격자 좌표 변환**: 위경도를 격자 좌표로 변환하는 코드가 필요하다. 기상청 문서에서 변환표를 제공한다.

### 교통: TAGO 버스도착정보

전국 버스 도착 예정 시간 실시간 조회. data.go.kr 게이트웨이(패턴 A)로 접근 가능.

```python
def get_bus_arrival(city_code, node_id):
    """
    city_code: 도시코드 (서울: 11, 부산: 21, 대구: 22 등)
    node_id: 정류소 ID
    """
    url = "http://apis.data.go.kr/1613000/ArvlInfoInqireService/getSttnAcctoArvlPrearngeInfoList"
    params = {
        "serviceKey": SERVICE_KEY,
        "pageNo": 1,
        "numOfRows": 10,
        "dataType": "JSON",
        "cityCode": city_code,
        "nodeId": node_id,
    }
    response = requests.get(url, params=params)
    data = response.json()

    header = data["response"]["header"]
    if header["resultCode"] != "00":
        raise ValueError(f"에러: {header['resultMsg']}")

    return data["response"]["body"]["items"]["item"]
```

### 관광: TourAPI 관광지 검색

전국 관광지, 음식점, 숙소, 축제 등 통합 검색.

```python
def search_tourist_spots(keyword, content_type_id=12):
    """
    content_type_id: 12=관광지, 14=문화시설, 15=축제, 25=여행코스, 28=레포츠, 32=숙박, 38=쇼핑, 39=음식점
    """
    url = "http://apis.data.go.kr/B551011/KorService1/searchKeyword1"
    params = {
        "serviceKey": SERVICE_KEY,
        "pageNo": 1,
        "numOfRows": 10,
        "dataType": "JSON",
        "MobileOS": "ETC",
        "MobileApp": "MyApp",
        "keyword": keyword,
        "contentTypeId": content_type_id,
    }
    response = requests.get(url, params=params)
    data = response.json()

    header = data["response"]["header"]
    if header["resultCode"] != "0000":  # TourAPI는 "0000"이 성공
        raise ValueError(f"에러: {header['resultMsg']}")

    return data["response"]["body"]["items"]["item"]

# 사용
spots = search_tourist_spots("경복궁")
for spot in spots:
    print(f"{spot['title']} - {spot['addr1']}")
```

> **주의**: TourAPI는 성공 코드가 `"0000"`이다 (data.go.kr 일반 API는 `"00"`). 포털마다 에러코드 체계가 다른 대표적 사례.

### 금융: ECOS 환율 조회 (한국은행 경제통계)

한국은행 경제통계 시스템. **data.go.kr과 별도**로 [ecos.bok.or.kr](https://ecos.bok.or.kr)에서 가입/키 발급 필요.

```python
def get_exchange_rate(api_key, start_date, end_date):
    """
    한국은행 ECOS: 주요국 통화의 대원화 환율
    통계표코드: 731Y001, 항목코드: 0000001 (미국 달러)
    """
    url = (
        f"https://ecos.bok.or.kr/api/StatisticSearch"
        f"/{api_key}/json/kr/1/10"
        f"/731Y001/D/{start_date}/{end_date}/0000001"
    )
    response = requests.get(url)
    data = response.json()

    if "StatisticSearch" not in data:
        raise ValueError(f"에러: {data}")

    return data["StatisticSearch"]["row"]

# 사용 (ECOS 자체 키 필요)
# rates = get_exchange_rate("ECOS_API_KEY", "20260401", "20260402")
```

### 부동산: 아파트 실거래가 조회

국토교통부 실거래가 정보. data.go.kr 게이트웨이(패턴 A)로 접근 가능.

```python
def get_apt_trade(lawd_cd, deal_ymd):
    """
    lawd_cd: 법정동코드 앞 5자리 (강남구: 11680)
    deal_ymd: 계약월 (202603)
    """
    url = "http://apis.data.go.kr/1613000/RTMSDataSvcAptTradeDev/getRTMSDataSvcAptTradeDev"
    params = {
        "serviceKey": SERVICE_KEY,
        "pageNo": 1,
        "numOfRows": 100,
        "LAWD_CD": lawd_cd,
        "DEAL_YMD": deal_ymd,
        "dataType": "JSON",
    }
    response = requests.get(url, params=params)
    data = response.json()

    header = data["response"]["header"]
    if header["resultCode"] != "00":
        raise ValueError(f"에러: {header['resultMsg']}")

    return data["response"]["body"]["items"]["item"]

# 사용
# trades = get_apt_trade("11680", "202603")
# for t in trades:
#     print(f"{t['aptNm']} {t['excluUseAr']}m2 - {t['dealAmount']}만원")
```

---

## 5. 유용한 도구

### PublicDataReader (Python)

16개 이상의 공공데이터 소스를 통합하는 Python 라이브러리. 인코딩 문제, 에러 처리 등을 내부적으로 해결해준다.

```bash
pip install PublicDataReader
```

```python
from PublicDataReader import TransactionPrice

# 아파트 실거래가 조회 (serviceKey 이중인코딩 문제를 자동 처리)
service_key = "디코딩키"
apt = TransactionPrice(service_key)

# 강남구 2026년 3월 아파트 매매
df = apt.get_data(
    property_type="아파트",
    trade_type="매매",
    sigungu_code="11680",
    year_month="202603",
)
print(df.head())
```

지원 데이터 소스: 부동산 실거래가, 건축물대장, 상가정보, 국토교통부 통계, KOSIS 등.

- GitHub: [github.com/WooilJeong/PublicDataReader](https://github.com/WooilJeong/PublicDataReader)

### data-go-mcp-servers (Claude/Cursor 연동)

data.go.kr의 주요 API를 Claude Desktop 또는 Cursor에서 바로 사용할 수 있는 MCP 서버.

```json
// Claude Desktop 설정 예시 (claude_desktop_config.json)
{
  "mcpServers": {
    "data-go-kr": {
      "command": "npx",
      "args": ["-y", "data-go-mcp-servers"],
      "env": {
        "DATA_GO_KR_API_KEY": "여기에_디코딩키_입력"
      }
    }
  }
}
```

- 지원 API: 기상청 예보, 대기질, 부동산 실거래가, 버스도착정보 등 6개
- GitHub: [github.com/xnomad-ai/data-go-mcp-servers](https://github.com/xnomad-ai/data-go-mcp-servers)

### Postman 설정 가이드

Postman으로 빠르게 API를 테스트하는 방법:

1. **새 Request 생성**: GET 방식 선택
2. **URL 입력**: `http://apis.data.go.kr/1360000/VilageFcstInfoService_2.0/getVilageFcst`
3. **Params 탭에서 파라미터 추가**:
   - `serviceKey`: 인코딩키 (Postman은 자동 인코딩하지 않으므로 **Encoding 키** 사용)
   - `dataType`: `JSON`
   - 나머지 필수 파라미터
4. **Send 클릭**
5. **응답 확인**: `resultCode`가 `"00"`인지 반드시 확인

> **Postman 팁**: 인코딩 키를 Environment Variable로 저장해두면 여러 API 테스트에 재사용할 수 있다. `{{SERVICE_KEY}}` 형태로 참조.

**Postman에서 serviceKey 이중인코딩 방지**:
- Settings > General > "Encode URL automatically" 옵션을 **끄고** Encoding 키를 직접 사용
- 또는 옵션을 켠 상태에서 **Decoding 키**를 사용

---

## 6. FAQ - 자주 묻는 질문 10개

### Q1. API 키를 발급받았는데 "SERVICE_KEY_IS_NOT_REGISTERED_ERROR"가 나옵니다.

**A**: 가장 흔한 원인 3가지:
1. **이중인코딩**: Encoding 키를 HTTP 라이브러리의 params에 넣으면 이중 인코딩됨. Decoding 키를 params에 넣거나, Encoding 키를 URL에 직접 삽입하라.
2. **승인 대기 중**: 수동승인 API는 발급 즉시 사용 불가. 마이페이지에서 승인 상태를 확인하라.
3. **패턴 B/C API**: data.go.kr 키로 원본 사이트 API를 호출하면 당연히 실패한다. 원본 사이트에서 별도 키를 발급받아야 한다.

### Q2. 응답이 XML로 오는데 JSON으로 받고 싶습니다.

**A**: 대부분의 data.go.kr API는 `dataType=JSON` 파라미터를 지원한다. 단, 일부 오래된 API는 JSON을 지원하지 않는다. 이 경우 `xmltodict` 같은 라이브러리로 파싱하라.

```python
import xmltodict

response = requests.get(url, params=params)
data = xmltodict.parse(response.text)
```

### Q3. 호출 횟수가 일 1,000회로 부족합니다.

**A**: data.go.kr 마이페이지에서 "트래픽 증량 신청"이 가능하다. 사유를 작성하면 검토 후 승인된다 (3일~2주 소요). 또는 원본 사이트에서 직접 키를 발급받으면 더 높은 한도를 제공하는 경우가 많다.

### Q4. 브라우저(React/Vue 등)에서 API를 직접 호출하면 에러가 납니다.

**A**: CORS 미지원 때문이다. 백엔드 프록시 서버를 경유해야 한다. 개발 단계에서는 Vite/CRA의 proxy 설정을 활용할 수 있다.

```javascript
// vite.config.js
export default {
  server: {
    proxy: {
      '/api': {
        target: 'http://apis.data.go.kr',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
};
```

### Q5. API 키가 만료되었는지 어떻게 알 수 있나요?

**A**: data.go.kr 마이페이지 > 활용신청 현황에서 만료일을 확인할 수 있다. 만료 30일 전에 이메일 알림이 오지만, 놓치기 쉽다. 키 발급일 + 730일(2년)을 캘린더에 등록해두는 것을 강력히 권장한다.

### Q6. data.go.kr에서 같은 데이터가 여러 개 검색됩니다. 어떤 것을 써야 하나요?

**A**: data.go.kr에는 **파일 데이터(CSV/Excel)**와 **실시간 API**가 섞여 나온다. "오픈API" 탭에서 필터링하고, "활용건수"가 많은 것을 선택하라. 활용건수가 많을수록 커뮤니티 자료도 풍부하다.

### Q7. 공공 API 데이터로 상업 서비스를 만들어도 되나요?

**A**: 공공누리 라이선스 유형에 따라 다르다. **제1유형**(출처표시)과 **제2유형**(출처표시+변경금지)은 상업적 이용이 가능하다. **제3유형**과 **제4유형**은 비상업적 용도만 허용된다. API 상세 페이지에서 반드시 확인하라.

### Q8. Python 외 다른 언어용 SDK는 없나요?

**A**: 공식 SDK는 거의 없다. 커뮤니티에서 Python 위주로 제공된다. Java, JavaScript, Go 등은 직접 HTTP 클라이언트로 호출해야 한다. 다만, `serviceKey` 인코딩 처리만 주의하면 호출 자체는 단순한 REST GET 요청이므로 어떤 언어든 가능하다.

### Q9. API 응답 속도가 너무 느립니다.

**A**: 공공 API는 일반적으로 1~5초 응답 시간을 예상해야 한다. 특히 새벽 시간대(01:00~06:00)에는 점검으로 더 느려지거나 응답이 없을 수 있다. 캐싱을 적극 활용하고, 타임아웃을 10초 이상으로 설정하라.

```python
response = requests.get(url, params=params, timeout=15)  # 타임아웃 넉넉히
```

### Q10. Claude/ChatGPT에서 공공 API를 쓸 수 있나요?

**A**: MCP 서버를 통해 가능하다. 현재 `data-go-mcp-servers`(data.go.kr 6개 API), `dartpoint-mcp`(DART 전자공시), `data4library-mcp`(도서관) 등이 공개되어 있다. 다만 전체 공공 API의 5% 미만만 MCP로 커버되므로, 대부분은 직접 function calling 스키마를 작성해야 한다.

---

## 부록: 주요 에러코드 (data.go.kr)

| resultCode | resultMsg | 원인 | 해결 |
|-----------|-----------|------|------|
| 00 | NORMAL_SERVICE | 정상 | - |
| 01 | APPLICATION_ERROR | 서버 내부 오류 | 재시도 |
| 02 | DB_ERROR | DB 오류 | 재시도 |
| 03 | NODATA_ERROR | 데이터 없음 | 파라미터 확인 |
| 04 | HTTP_ERROR | HTTP 연결 오류 | 재시도 |
| 10 | INVALID_REQUEST_PARAMETER_ERROR | 파라미터 오류 | 필수 파라미터 확인 |
| 11 | NO_MANDATORY_REQUEST_PARAMETERS_ERROR | 필수 파라미터 누락 | API 문서 확인 |
| 12 | NO_OPENAPI_SERVICE_ERROR | 서비스 없음 | URL 확인 |
| 20 | SERVICE_ACCESS_DENIED_ERROR | 접근 거부 | 활용신청 확인 |
| 22 | LIMITED_NUMBER_OF_SERVICE_REQUESTS_EXCEEDS_ERROR | 호출 한도 초과 | 다음 날 재시도 또는 증량 신청 |
| 30 | SERVICE_KEY_IS_NOT_REGISTERED_ERROR | 키 미등록 | 인코딩 확인 / 승인 확인 |
| 31 | DEADLINE_HAS_EXPIRED_ERROR | 키 만료 | data.go.kr에서 갱신 |
| 32 | UNREGISTERED_IP_ERROR | IP 미등록 | 활용신청에서 IP 추가 |

---

> 이 가이드에서 다루지 못한 포털별 상세 사항은 [report.md](../report.md)를 참조하라.

---

## 부록: 유용한 커뮤니티 라이브러리 (2026-04-02 업데이트)

| 라이브러리 | Stars | 대상 | 설치 |
|-----------|-------|------|------|
| PublicDataReader | 557 | data.go.kr 16+ 소스 | `pip install PublicDataReader` |
| FinanceDataReader | 1,400 | KRX/NYSE/환율 | `pip install finance-datareader` |
| pykrx | 954 | KRX 상세 (투자자별) | `pip install pykrx` |
| dart-fss | 360 | DART 전자공시 | `pip install dart-fss` |
| OpenDartReader | 433 | DART 간편 래퍼 | `pip install opendartreader` |

## 부록: API 폐기 주의사항

2022~2025년 동안 12건의 API 폐기/변경이 확인되었다. **42%가 사전 공지 없이** 이루어졌으므로:

1. **API 응답을 주기적으로 모니터링**하라 — 갑작스러운 필드 변경에 대비
2. **대체 API를 사전에 파악**하라 — 특히 부동산, 기상 분야
3. **API Key 만료일(2년)을 캘린더에 등록**하라 — 자동갱신이 없다
4. 주요 변경 사례: 국토부 실거래가 무공지 스펙변경(2024), 한국마사회 무공지 폐기(2023)
