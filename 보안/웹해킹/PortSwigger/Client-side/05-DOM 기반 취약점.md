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