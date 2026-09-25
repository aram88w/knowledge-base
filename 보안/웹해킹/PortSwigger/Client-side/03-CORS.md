## CORS

CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)는 브라우저에서 **다른 출처(Origin)의 리소스에 접근할 수 있도록 서버가 허용 여부를 알려주는 표준 방식**

### SOP

SOP(Same-Origin Policy, 동일 출처 정책)는 다른 Origin의 리소스를 JavaScript가 읽거나 조작하는 것을 막는 브라우저의 기본 보안 정책

#### Origin 

**Origin: Scheme + Host + Port**

| URL A                   | URL B                      | same-origin?  |
| ----------------------- | -------------------------- | ------------- |
| `https://example.com/a` | `https://example.com/b`    | ✅ (경로는 무시)    |
| `https://example.com`   | `http://example.com`       | ❌ (scheme 다름) |
| `https://example.com`   | `https://example.com:8080` | ❌ (port 다름)   |
| `https://example.com`   | `https://api.example.com`  | ❌ (host 다름)   |

#### 기본 동작 

SOP는 가져오기와 읽기/접근을 구분함. 대부분의 경우 **가져오기는 허용**하고 **읽기/접근은 막음.** 
예: `fetch`로 다른 Origin의 응답 본문을 읽는 것이 불가능함. 

| 리소스         | 로딩/표시/실행 | JS가 내용 읽기/접근     | CORS 필요 여부         |
| ----------- | -------- | ---------------- | ------------------ |
| `<img>`     | 가능       | 픽셀 읽기 불가         | 픽셀 읽으려면 필요         |
| `<video>`   | 가능       | 프레임 캡처 불가        | 캡처하려면 필요           |
| `fetch`/XHR | 요청은 가능   | 응답 읽기 불가         | 응답 읽으려면 필요         |
| `<script>`  | 로드/실행 가능 | 소스 읽기 불가         | 모듈은 필요             |
| `<iframe>`  | 표시 가능    | DOM 접근 불가        | DOM 접근은 CORS로도 제한적 |
| CSS         | 로드 가능    | `cssRules` 읽기 불가 | 규칙 읽으려면 필요         |
| Storage     | -        | 다른 Origin 접근 불가  | CORS와 무관           |


#### 예외 

1. `location`
	- 읽기: 불가능, 다른 Origin의 `iframe`이나 팝업의 `location.href`에 접근하려고 하면 막힘. 
	- 쓰기(이동): 가능, `location.href = 'https://...'`처럼 다른 URL로 이동시키는 것은 허용

2. `window.length`, `window.closed`
	- 읽기: 가능
	- 쓰기(호출): 불가능

3. `window.close()`, `window.blur()`, `window.focus()`
	- 쓰기(호출): 가능

4. `window.postMessage()`: cross-origin 통신을 안전하게 하기 위한 공식 수단
	- 쓰기(호출): 가능
	- 다른 Origin의 창에 메시지를 보내고 받는 쪽에서 `message` 이벤트로 처리

5. 쿠키: SOP와 별개로 동작
	- 쿠키는 Origin이 아닌 Site 기반으로 동작
	- `HttpOnly`: `document.cookie`로 읽고 쓰는 것이 불가능함. 
	- `Secure`: HTTPS 페이지에서만 Secure 쿠키를 읽고 쓸 수 있음.

6. `document.domain`: SOP를 일부러 완화하는 레거시 기능
	- 같은 상위 도메인을 가진 서브도메인끼리 SOP를 완화할 수 있음. 
		- 예: `marketing.example.com`와 `www.example.com`에서 둘 다 `document.domain = 'example.com'`으로 설정하면 서로 다른 Origin임에도 불구하고 DOM 접근이 허용됨. 
	- 최신 브라우저에서는 보안상의 이유로 deprecated임. 


### CORS

CORS(Cross-Origin Resource Sharing, 교차 출처 리소스 공유)는 HTTP 헤더를 사용해 cross-origin의 접근을 허용하는 방법임. 

**상황** 
`https://normal-website.com`의 JS인 `fetch`로 `https://robust-website.com/data`에 요청하는 경우

- 브라우저가 자동으로 Origin 헤더를 붙임. 
``` http 
GET /data HTTP/1.1
Host: robust-website.com
Origin: https://normal-website.com 
```

- `robust-website.com` 서버의 응답
``` http 
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://normal-website.com
```

그러면 브라우저는 **Origin과 Access-Control-Allow-Origin의 값을 비교해서 같은 경우**에 JS가 `fetch`의 응답 내용을 읽을 수 있게 허용해줌. 


#### 사전 요청

교차 출처(cross-origin) 상황에서 **단순 요청이 아닌 요청(non-simple)** 을 보낼 때, (보통 `fetch`나 `XMLHttpRequest`) **브라우저**가 바로 요청을 보내지 않고 자동으로 OPTIONS 메서드로 사전 요청을 보내서 서버에게 초기 검사를 하는 것이 CORS Preflight(사전 요청)이라고 함. 

| 구분           | 단순 요청 (simple request)                           | non-simple 요청        |
| ------------ | ------------------------------------------------ | -------------------- |
| 메서드          | GET, HEAD, POST                                  | PUT, DELETE, PATCH 등 |
| 헤더           | safelisted header만                               | 커스텀 헤더 포함            |
| Content-Type | form-urlencoded, multipart/form-data, text/plain | application/json 등   |
| HTML만으로 가능?  | 가능 (form, img 등)                                 | 불가능                  |
| Preflight    | 없음                                               | 있음                   |
| 요청 전송        | 바로 보냄                                            | **OPTIONS 먼저**       |
| 응답 읽기        | CORS 헤더 필요                                       | CORS 헤더 필요           |

다음을 검사함. Access-Control-Allow-Headers
- 요청을 보낸 **출처(Origin)**
- 사용하려는 **메서드(Method)**
- 사용하려는 **헤더(Headers)**
- (필요하다면) **인증 정보(Credentials)**

서버가 허용하면(`Access-Control-Allow-*` 헤더로 응답) → 브라우저가 **실제 요청**을 보냄
서버가 허용하지 않으면 → 브라우저가 **실제 요청을 보내지 않고 차단**함 (서버에 부작용이 생기는 걸 막음)


**예시**
PUT 요청, safelisted header가 아닌 Authorization, X-Request-Id(커스텀 헤더)를 사용하는 경우

- 브라우저의 사전 요청 
``` http 
OPTIONS /data HTTP/1.1
Host: <some website>
...
Origin: https://normal-website.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: authorization, x-request-id
```

- 서버의 응답 
``` http 
HTTP/1.1 204 No Content
...
Access-Control-Allow-Origin: https://normal-website.com
Access-Control-Allow-Methods: PUT, POST, OPTIONS
Access-Control-Allow-Headers: authorization, x-request-id, ...
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 240
```
- `Access-Control-Allow-Origin`: 이 출처의 페이지가 응답을 읽어도 됨. 
- `Access-Control-Allow-Methods`: 실제 요청에서 허용하는 메서드 목록
- `Access-Control-Allow-Headers`: 실제 요청에서 허용하는 헤더 목록
- `Access-Control-Allow-Credentials`: 인증 정보(쿠키, Authorization, TLS 클라이언트 인증서)를 포함한 요청을 허용할지

**흐름도** 
``` txt
요청 발생 (fetch)
│
├─ 동일 출처인가?
│   └─ YES → 그냥 보냄 (CORS 무관)
│
└─ 교차 출처인가?
    │
    ├─ no-cors 모드인가?
    │   └─ YES → 그냥 보냄, 응답 읽기 불가, preflight 없음
    │
    └─ cors 모드인가?
        │
        ├─ 단순 요청인가? - GET, HEAD, POST
        │   └─ YES → 그냥 보냄, 응답은 CORS 헤더 있을 때만 읽힘
        │
        └─ non-simple 요청인가? - PUT, DELETE, PATCH emd
            └─ YES → OPTIONS(preflight) 먼저 보냄
                     → 서버가 허용하면 실제 요청 보냄
                     → 허용 안 하면 실제 요청 안 보냄
```

| 구분           | 단순 요청 (simple request)                           | non-simple 요청        |
| ------------ | ------------------------------------------------ | -------------------- |
| 메서드          | GET, HEAD, POST                                  | PUT, DELETE, PATCH 등 |
| 헤더           | safelisted header만                               | 커스텀 헤더 포함            |
| Content-Type | form-urlencoded, multipart/form-data, text/plain | application/json 등   |
| HTML만으로 가능?  | 가능 (form, img 등)                                 | 불가능                  |
| Preflight    | 없음                                               | 있음                   |
| 요청 전송        | 바로 보냄                                            | OPTIONS 먼저           |
| 응답 읽기        | CORS 헤더 필요                                       | CORS 헤더 필요           |


