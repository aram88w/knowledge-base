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


