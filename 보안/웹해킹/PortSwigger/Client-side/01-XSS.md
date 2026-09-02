## XXS

서버가 사용자의 입력을 의심 없이 화면에 그대로 내보내는 경우, 텍스트 대신 악성 스크립트를 주입하여 피해자의 브라우저에서 스크립트를 읽었을때 강제로 실행되게 만드는 취약점

대부분의 XSS 취약점을 확인할때 무해한 `alert()`를 활용해서 테스트함. 
참고: 크롬 브라우저에서 2021년 7월 20일부터 버전 92부터, 교차 출처 iframe이 `alert()` 를 호출하는 것을 막혔음. 일부 고급 XSS 공격에서는 `print()`를 사용하는 것을 권장함. 

**종류**
- 반사형 XSS: 악성 스크립트가 담긴 URL을 유포하는 방식
- 저장형 XSS: 악성 스크립트가 타겟 웹사이트의 DB에 저장되는 방식
- DOM 기반 XSS: 오직 사용자의 브라우저(클라이언트 측) 내부에서만 발생하는 방식

### 반사형 XSS

애플리케이션이 HTTP 요청(쿼리 파라미터, HTTP 바디)으로 수신한 데이터를 안전한 처리(인코딩) 없이 즉각적인 응답(HTML 소스 코드)에 그대로 포함할 때 발생하는 취약점. 

#### 영향

- 사용자가 애플리케이션 내에서 수행할 수 있는 모든 작업을 수행 가능
- 사용자가 볼 수 있는 모든 정보 표시 
- 사용자가 수정할 수 있는 모든 정보 변경
- 다른 애플리케이션과의 상호 작용을 통해 또다른 악성 공격을 할 수 있음. 

#### 예시

웹사이트의 검색 기능에서 사용자가 입력한 검색어를 URL 파라미터로 전달하고 
``` url
https://insecure-website.com/search?term=gift
```

그 결과가 HTML 소스 코드에 반영되는 경우 
``` html
<p>You searched for: gift</p>
```

해커의 공격 과정
1. 해커가 악성 스크립트를 작성: 사용자의 쿠키를 가져오거나 등 
``` html
https://insecure-website.com/search?term=<script>악성 스크립트</script>
```

2. 피해자가 저 URL을 들어올 수 있도록 유도: 피싱 메일 등
3. 피해자가 URL에 들어오면 HTML 소스 코드에 해당 악성 스크립트가 포함되어 있고 브라우저가 `<script>` 스크립트를 읽으면 실행시킴. 
``` html
<p>You searched for: <script>악성 스크립트</script></p>
```


#### 테스트 방법

1. 엔드포인트 테스트: 웹 애플리케이션에서 데이터가 들어가는 HTTP 요청을 찾아봄
	- URL 쿼리 파라미터, HTTP 바디, URL 파일 경로 등 
2. 입력한 값이 화면에 반사되어 나오는지 확인: 입력 유효성 검사에 걸리지 않도록 특수문자를 빼고 알파벳과 숫자로만 구성
	- Burp Intruder의 `Grep - Payloads` 기능을 켜두면 내가 쏜 무작위 값이 응답에 반사되어 돌아왔을 때 Burp가 자동으로 체크해줌. 
3. 반사된 위치 파악: 위치에 따라 공격 코드가 달라지므로 응답 HTML의 어떤 위치에 떨어졌는지 파악해야함. 
	- 일반적인 HTML 태그 사이 텍스트: `<div>여기에 반사됨</div>`
	- HTML 태그의 속성값 내부: `<input value="여기에 반사됨">`
	- 자바스크립트 문자열 내부: `let name = '여기에 반사됨';`
4. 테스트 스크립트 주입: `alert()` 같이 기본적인 자바스크립트를 담아서 Repeater로 요청을 보낸 뒤, 응답 화면을 보면서 내 코드가 안 깨지고 잘 들어갔는지 확인
	- 악성 스크립트 앞에 무작위 문자열을 붙여서 보내면 응답 창에서 그 무작위 문자열을 검색해서 빠르게 찾을 수 있음. 
		- 예: `a1b2c3d4<script>alert(1)</script>`
5. 실제 공격
	- Burp Proxy의 인터셉트에서 요청을 가로채서 페이로드를 끼워 넣거나, 브라우저 주소창에 완성된 악성 URL을 직접 복사해 붙여넣음. 


### 저장형 XSS

공격자가 삽입한 악성 스크립트가 서버(DB, 파일 등)에 저장된 후, 해당 데이터를 조회하는 불특정 다수 피해자의 웹 브라우저에서 스크립트가 실행되도록 하는 XSS 공격
예: 블로그 게시물에 대한 댓글, 채탕방의 사용자 닉네임, 고객 주문 연락 정보 등

저장형 XSS는 애플리케이션 내에서 자체적으로 작동하는 공격을 하기 때문에 반사형 XSS와 다르게 피해자가 특정 요청을 보내도록 유도하는 **외부 방법(예: 피싱 메일)을 사용할 필요가 없음.** 

또한 반사형 XSS는 피해자가 로그아웃인 경우 세션 쿠키 같은 것을 탈취하는 것이 불가능하므로 피해자의 로그인 여부에 따라 공격의 성공 여부가 정해지지만, 저장형 XSS는 **로그인 후 이용하는 내부 페이지에 스크립트가 저장되는 경우가 많아서 공격 성공률이 높음.** 

#### 예시

웹사이트에서 게시물에 대한 댓글을 다는 요청 예시
``` http
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net
```

응답 HTML 예시
``` html
<p>This post was extremely helpful.</p>
```

해커의 공격
``` http
comment=<script>악성 스크립트</script>
```
- **URL 인코딩 해야함.**

``` html
<p><script>악성 스크립트</script></p>
```


#### 테스트 방법

저장된 XSS 취약점에 대한 수동 테스트는 입력 지점과 출력 지점을 모두 찾아야하므로 어려움. 
- **입력 지점**: 애플리케이션 처리 과정에서 데이터가 들어가는 지점
- **출력 지점**: 그 데이터가 애플리케이션 응답에서 나타나는 지점
특히 출력 지점은 찾아내기 어려움. 
- 해커가 볼 수 없는 관리자 페이지나 숨겨진 로그에만 나올 수 있음. 
- 최근 검색어처럼 데이터가 영구 보존되지 않고 밀려서 사라질 수 있음. 

1. 입력 지점 찾기
	- URL 쿼리 파라미터 및 POST 바디 데이터 (게시글, 댓글, 프로필 등)
	- HTTP 요청 헤더: `User-Agent`, `Referer` 등 (방문자 로그, 통계 페이지에 저장됨.)
	- 외부 데이터 연동: 웹메일 수신 메일 본문, 외부 API(트위터/SNS 피드, 뉴스 등)를 통해 유입되는 데이터
2. 출력 지점 찾기: 모든 입력창과 모든 페이지를 하나씩 짝지어서 확인하는 것보다 고유한 식별값을 뿌려두고 추적하는 것이 효율적임.
	- 이름 입력창: `test_name_01`
	- 이메일 입력창: `test_email_01`
	- User-Agent 헤더: `test_ua_01`
	- 이후 Burp 검색 기능으로 `test_` 같은 표식이 어떤 응답페이지에서 튀어나오는지 추적 
3. 단순 반사 vs 실제 저장 구별: 응답 내에서 표식이 발견되었을때, 방금 보낸 요청으로 즉각 답이 나온 것(반사형 XSS)인지 아니면 DB에 저장된 후 불러온 것(저장형 XSS)인지 확인해야함.


### DOM 기반 XSS

애플리케이션이 자바스크립트가 공격자가 제어할 수 있는 소스(예: URL)에서 데이터를 가져와 `innerHTML`, `document.write()`, `eval()`, `setTimeout()`처럼 브라우저에서 동적으로 코드를 실행할 때 발생하는 XSS 취약점

#### HTML 싱크

브라우저가 화면을 그리는 과정에서 악성 스크립트가 실행되는 경우
브라우저에서 사용되는 함수: `innerHTML`, `document.write()`, `outerHTML` 등

https://portswigger.net/web-security/cross-site-scripting/dom-based#which-sinks-can-lead-to-dom-xss-vulnerabilities

##### 예시

**주소창 기반으로 HTML에 반영되는 경우**
DOM 기반 XSS 공격의 가장 흔한 경우가 URL을 `window.location`로 접근하는 상황임.

- 서버 코드: URL에서 악성 스크립트가 있다면 실행됨. 
``` html
<!-- 서버가 만들어서 보내준 HTML 소스코드 (아주 깨끗함) -->
<html>
  <body>
    결과: <span id="results"></span> 

    <script>
      // 브라우저가 주소창(URL)을 읽고 화면에 그리는 코드 
      let q = new URLSearchParams(window.location.search).get('query');
      document.getElementById('results').innerHTML = q; 
    </script>
  </body>
</html>
```

- 해커의 공격: 피싱 메일 등을 통해서 피해자가 이 경로로 들어오도록 해야함. 
``` html
http://ex.com/search.html?query=<img src=x onerror=alert('해킹성공')>
```

1. `window.location.search`: 쿼리 파라미터 `?query=<img src=x onerror=alert('해킹성공')>`를 가져옴 
2. `ineerHTML`로 화면 렌더링: 브라우저가 `<img src=x onerror=alert('해킹성공')>`를 화면에 그리려고 시도하고 `alert()`가 실행됨. 

**참고**
현재 최신 브라우저들은 보안(HTML5 표준)을 위해 `innerHTML`을 통해 삽입된 `<script>` 태그, `<svg>`태그의 `onload` 이벤트는 실행하지 않고 무시함. 
따라서 `<img>`나 `iframe`의 `onerror`를 사용해야함. 


**404 오류 페이지 예시** 
없는 경로로 페이지를 요청시 "죄송합니다. {해당 경로}를 찾을 수 없습니다."라고 보여주는 웹사이트의 경우에 
`http://example.com/<script>alert(1)</script>`를 넣으면 (`innerHTML`이 아니라고 가정) "죄송합니다 `<script>alert(1)</script>`를 찾을 수 없습니다."가 브라우저에 그려지면서 XSS 공격이 성공함. 

**PHP 기반 웹사이트 예시** 
PHP에서는 `http://example.com/index.php/user/profile` 처럼 파일 이름(`index.php`) 뒤에 슬래시(`/`)를 붙여서 경로를 길게 쓰는 경우가 많았음. 
이때도 프론트엔드 자바스크립트가 저 뒷부분의 경로를 읽어서 화면을 구성한다면, 해커가 그 자리에 악성 코드를 넣어서 공격 가능


###### 속성 벗어나기

검색을 하면 화면에 검색어와 검색결과가 나오는 웹사이트의 경우

TEST1234 검색 -> img 태그 내에 있는거 확인 후 -> img 검색해서 만들어지는 코드 확인
``` js
<script>
    function trackSearch(query) {
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    }
...
```

공격 스크립트: src 속성에서 벗어나야 하기 때문에 앞에 `"`를 붙여줘야함. 
- `"><script>alert(1)</script>`
- `"'><img src onerror=alert(1)>`
- `"><svg onload=alert(1)>`

###### 태그 벗어나기 

``` js
<script>
	var stores = ["London","Paris","Milan"];
	var store = (new URLSearchParams(window.location.search)).get('storeId');
	    document.write('<select name="storeId">');
        if(store) {
            document.write('<option selected>'+store+'</option>');
        }
...
```

공격 스크립트: `</select><img src=1 onerror=alert(1)>`
``` js
<select name="storeId">
<option selected>
// select 태그를 닫아준 뒤 img 사용해야 정상적으로 파싱됨. 
</select><img src=1 onerror=alert(1)>
// 무시됨
</option> 
```
- `select` 태그를 닫아준 뒤 `img` 태그를 사용해야함. 

##### 테스트

1. 테스트용 문자열 입력: 웹 브라우저의 주소창(URL)이나 입력칸에 고유한 문자열을 입력
2. 문자열 검색
	- 페이지 소스 보기: 원본 HTML만 보여주므로 사용 X
	- **개발자 도구**: 자바스크립트가 실행된 후 현재 화면 상태(DOM)를 실시간으로 보여줌. 검색 기능(Cmd + F)을 이용해서 고유한 문자열을 찾아야함. 
	- 문자열을 찾은 후 해당 태그가 만들어지는 JS 코드를 찾아야함. 만약 서버에서 만들어지는 경우라면 JS 코드가 없고 XSS가 불가능한 것임.
3. 문맥 파악 및 탈출 시도: 문자열이 나타나는 위치를 파악하고 컨텍스트에 맞게 공격 코드를 다르게 작성해야함. 
	- 일반적인 HTML 태그 사이 텍스트: `<div>TEST1234</div>`
	- HTML 태그의 속성값 내부: `<input value="TEST1234">`
	- 자바스크립트 문자열 내부: `let name = 'TEST1234';`
	- 예를 들어 태그의 속성값 내부에서 나타난다면 `"`를 이용해서 탈출 시도를 해봐야함. 


**주의**
최신 브라우저들은 주소창(URL)을 URL 인코딩하는데 이럴 경우 해커가 입력한 특수문자(`<`, `>`, `"`)가 `%3C`, `%3E`, `%22` 같은 안전한 형태(URL 인코딩)로 변환되기 때문에 XSS 공격이 실패할 수 있음. 

최신 브라우저에서 자동 URL 인코딩이 적용되어도 DOM 기반 XSS 공격이 성공하는 경우
- 최신 자바스크립트 API의 자동 디코딩: 요즘 개발자들은 주소창의 파라미터를 읽을 때 `URLSearchParams`라는 최신 자바스크립트 내장 객체를 주로 사용하는데 자동 디코딩을 해버림. 
- 개발자의 디코딩 함수 사용: 사용자가 검색한 단어를 화면에 URL 인코딩해서 보여주면 UX가 떨어지기 때문에 `decodeURIComponent()`나 `decodeURI()` 같은 함수를 직접 사용해둘 수도 있음. 
- URL 인코딩을 거치지 않는 다른 유입 경로: 주소창(`location.search`, `location.hash`)이 아니라 다른 공격 통로 이용
	- `document.referrer`: 사용자가 방금 전 머물렀던 이전 페이지의 URL
	- `window.name`: 브라우저 탭이나 창의 고유 이름
	- `postMessage`, `event.data`: 다른 창이나 프레임 간에 주고받는 메시지
	- `localStorage`, `sessionStorage`: 브라우저 내장 데이터 저장소


#### 자바스크립트 실행 싱크

브라우저가 자바스크립트 명령어 자체를 실행하는 경우 
브라우저에서 사용되는 함수: `eval()`, `setTimeout()`, `setInterval()` 등

##### 예시

**주소창에 따라 다른 기능 실행**
가장 많이 발생하는 케이스입니다. 개발자가 코드를 줄이고 싶어서 머리를 쓴 경우

``` js
// ?action=closePopup 이면 팝업을 닫고, ?action=refresh 면 새로고침을 하려는 의도
let action = new URLSearchParams(window.location.search).get('action');

// 개발자의 의도: setTimeout("closePopup()", 1000) 
setTimeout(action + "()", 1000); 
```
- **문제점**: `setTimeout`은 첫 번째 인자로 '문자열'이 들어오면 그걸 `eval()`처럼 자바스크립트로 실행해 버림.
- 해커의 공격: `?action=alert(1)//` 라고 입력
- 결과: `setTimeout("alert(1)//()", 1000);`, 뒤에 붙은 `//`는 원래 있던 `()`를 주석 처리해버리고 `alert(1)`이 실행됨.


**로그인 후 리다이렉트**
로그인을 하거나 특정 작업이 끝난 뒤, 사용자가 원래 있던 페이지로 돌려보내 주기 위해 주소창에 `returnUrl`을 달고 다니는 경우

``` js
// ?returnUrl=/home/dashboard
let returnUrl = new URLSearchParams(window.location.search).get('returnUrl');

// "돌아가기" 버튼을 누르면 해당 URL로 이동시킴
document.getElementById('back-btn').onclick = function() {
    window.location.href = returnUrl; 
}
```
- **문제점:** 브라우저 주소창(`window.location.href`)에 `javascript:...` 로 시작하는 문자열을 넣으면, 브라우저는 그걸 URL로 이동하는 게 아니라 자바스크립트 코드로 실행해 버림. (이것도 일종의 자바스크립트 실행 싱크임.)
- 해커의 공격: `?returnUrl=javascript:alert('1')`
- 결과: 피해자가 "돌아가기" 버튼을 누르는 순간 `eval()`이란 단어는 코드에 단 한 줄도 없었지만 `alert(1)`이 실행됨.


**개발자의 실수**
현실의 웹사이트 코드는 몇 줄이 아니라 수만 줄임. 변수가 꼬리에 꼬리를 물고 이어지기 때문에 개발자 본인도 이 데이터가 어디서 출발했는지 잊어버리는 경우

``` js
// 파일 A.js (입구)
let userData = location.search.get('data');
processData(userData);

// ... (수백 줄 뒤, 혹은 다른 파일 B.js) ...

function processData(data) {
    let formatted = formatString(data);
    renderUI(formatted);
}

// ... (다른 파일 C.js - 출구) ...

function renderUI(uiData) {
    // 개발자는 uiData가 안전한 서버 데이터라고 착각함
    setTimeout(uiData.callback, 0); 
}
```
- **결과:** 소스코드를 짠 사람도 다르고, 파일도 다릅니다. 데이터를 넘겨받은 마지막 개발자는 `uiData`가 주소창에서부터 굴러온 위험한 데이터라는 것을 모르고 `setTimeout()`에 넣어버림.


##### 테스트

입력값이 `eval()`이나 `setTimeout()` 같은 자바스크립트 실행 함수(Sink)로 들어가 실행되기 때문에 HTML 화면에 아예 나타나지 않음. 글자가 화면에 찍히는 게 아니라 백그라운드에서 코드로 바로 실행되어 버리기 때문임. 

1. 자바스크립트 코드에서 입력값 유입로 찾기 
	- 개발자 도구에서 전체 검색 단축키(Cmd + Opt + F) 이용
	- `location.search`, `location.hash` 등 사용자의 입력값을 읽어들이는 키워드를 검색해서 어느 줄에서 이 코드를 사용하는지 찾기
2. 중단점을 통한 변수 추적: 입력값이 어디로 흘러가는지 확인해야함.  
	- 개발자 도구의 소스 탭에서 해당 코드 줄 번호를 클릭해 **중단점 설정** 
	- 그 후 새로고침하면 해당 줄에서 멈춘 것처럼 실행이 일시 정지됨. 
	- 정지된 상태에서 코드를 한 줄 씩 실행시키며 입력값이 어떤 변수를 거쳐 이동하는지 추적
3. 싱크 직전 상태 확인: 추적 끝에 `eval()`, `setTimeout()`같이 자바스크립트 실행 함수를 만났다면 공격이 가능함. 
	- 해당 함수 직전에 멈춰놓고 변수 위에 마우스를 올리면 입력값이 어떻게 다듬어져 있는지 확인할 수 있음. 
	- 해당 형태를 기반으로 공격 스크립트를 작성하여 공격
	- 예: 입력값이 `'TEST1234'`처럼 작은따옴표 안에 갇혀있다면, 주소창의 입력값을 조작해 `TEST'; alert(1); //` 처럼 바꿔서 넣음.

#### DOM Invader

개발자 도구로 일일이 검색하며 추적해야하는 것을 자동화 해주는 Burp Suite 내장 브라우저 기본 확장 프로그램 

- 카나리아(Canary) 자동 추적: 고유한 난수 문자열을 URL 파라미터나 폼 입력칸에 자동으로 넣어주고 이 문자열이 어떤 싱크(Sink)에서 나오는지 감시
- 문맥 자동 분석: 입력값이 HTML 태그 안인지, 자바스크립트 문자열 내부인지 자동으로 분석해 목록으로 보여줍니다.
- 페이로드 자동 생성: 발견된 취약점의 문맥에 딱 맞는 XSS 공격 코드를 자동으로 추천하고 테스트해 줌. 
- 웹 메시지 가로채기: 수동으로는 찾기 힘든 창 간 통신 취약점(`postMessage`)을 찾아서 조작할 수 있도록 함. 


#### jQuery 취약점

##### `attr()`

jQuery의 `attr()` 함수는 HTML 요소의 속성값을 가져오거나 추가 및 변경할 수 있음. 

``` js
$(function() {
	$('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
});
```
- `#backLink`가 `<a id="backLink" href="a">Back</a>` 이런 태그라고 할때, 백링크를 클릭했을때 악성 자바스크립트가 실행되도록 하려면 `javascript:alert(document.domain)` 이걸 넣어주면 됨. 
- `javascript:`는 브라우저가 주소를 해석할 때 사용하는 가상 프로토콜임. 주소를 해석하다가 `href`나 `src` 속성에서 `javascript:`를 만나면 해당 문자열을 자바스크립트로 실행한 후 주소를 이동함. 


##### `$()`

`$()`에 `#section1` 같은 **CSS 선택자**가 들어가면 해당 요소를 화면에서 검색함. (일반적인 사용) 하지만 `$()`에 `<img src=x onerror=alert(1)>` 같은 **HTML 태그 형태의 문자열**이 들어가는 경우, jQuery는 새로운 HTML 요소를 즉시 생성(렌더링)해 버림. 이걸 이용해서 DOM 기반 XSS 공격이 가능함. 

최신 버전의 jQuery는 `$()`에서 첫 글자가 `#`이면 그 이후 HTML 요소들을 문자열로만 취급하도록 패치됨.

``` js
$(window).on('hashchange', function() {
	var element = $(location.hash);
	element[0].scrollIntoView();
});
```

`https://타겟.com/#<img...>` 그냥 이런식의 url을 피해자에게 보내면 초기 로드이므로 `hashchange` 이벤트가 트리거가 되지 않음. 
그래서 해커의 사이트에 `iframe`으로 타겟 사이트를 넣고 피해자가 그 사이트에 들어오도록 함. 
피해자가 들어오면 iframe 태그를 브라우저가 그리게 되고 이때 `onload`로 `hashchange` 이벤트를 트리거 시킴.  
``` html
<iframe src="https://타겟.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```


#### AngularJS 취약점

HTML 요소에 `ng-app` 속성을 사용하면, AngularJS 프레임워크를 통해서 처리됨. AngularJS에서 `{{ }}` 안에 들어있는 값은 자바스크립트 코드로 실행함. 

`alert()`, `document.cookie`, `location` 같은 브라우저 내장 함수들은 전역(Global/Window) 영엑에 존재하기 때문에 사용하기 위해서는 `$on.constructor`를 이용해야함. 
- 예: `{{$on.constructor('alert(1)')()}}`


#### 반사형, 저장형 DOM XSS

해커의 악성 스크립트가 주소창에서 자바스크립트로 직접 들어가는 것이 아니라 **서버를 한번 거쳐서 응답 HTML 내부에 심어진(반사된/저장된) 다음** 그 값을 자바스크립트가 실행시켜서(`innerHTML`, `eval` 등) 발생하는 DOM XSS 취약점

**예시**
내가 보낸 검색어에 대한 결과가 JSON으로 반환되고 그걸 `eval()`하는 상황
``` json
// 응답 
{
	"results":[],
	"searchTerm":"zzz"
}
```

해커의 공격: 현재 서버에서 `"`를 `\"`로 처리하는 방어코드를 실행 중이므로 `\`를 넣어서 무효화시킴
- `/?search=\"};alert();//`: `;`로 표현식 분리
- `/?search=\"-alert()}//`: `-`로 표현식 분리

``` json
// 결과 
{
	"results":[],
	"searchTerm":"\\"
};alert();"}
```
- 입력한 공격스크립트가 응답에서 어떻게 처리되는지 확인해야함. 


##### 참고: 자바스크립트 내장 함수의 함정

자주 나오는 대표적인 자바스크립트 내장 함수들의 문법적 허점

- `replace()`의 1회성 동작: 정규표현식(`/.../g`)이 아닌 단순 문자열을 인자로 넣으면 맨 처음 발견된 하나만 치환함.
- 느슨한 타입 비교 (`==`): 자바스크립트는 `0 == ""` 나 `false == "0"` 같은 기상천외한 비교를 참(True)으로 판정함. 
- 문자열 검색의 한계 (`indexOf`, `includes`): 특정 안전한 주소(`https://example.com`)가 포함되어 있는지 검사하는 로직이 있을 때, 해커는 `javascript:alert(1);https://example.com` 처럼 실행 코드 뒤 주석 영역에 해당 문자열을 배치하여 필터를 속여 넘김.

