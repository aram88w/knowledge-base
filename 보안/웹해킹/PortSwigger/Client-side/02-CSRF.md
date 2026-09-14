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

기본 규칙: 쿠키에 설정된 도메인과 경로에 매칭되는 요청이면 어디서 발생하든 붙음. 
최신 브라우저 규칙: `SameSite=Lax`가 기본값인 경우가 많음. (cross-site POST에서는 쿠키 미전송)

#### Origin

- `same-origin`: 스키마 + 호스트 + 포트, 3가지가 모두 같은 경로 
- `cross-origin`: `same-origin`이 아닌 경로

| URL A                   | URL B                      | same-origin?  |
| ----------------------- | -------------------------- | ------------- |
| `https://example.com/a` | `https://example.com/b`    | ✅ (경로는 무시)    |
| `https://example.com`   | `http://example.com`       | ❌ (scheme 다름) |
| `https://example.com`   | `https://example.com:8080` | ❌ (port 다름)   |
| `https://example.com`   | `https://api.example.com`  | ❌ (host 다름)   |


#### Site

- `same-site`: 돈 주고 산 진짜 대표 주소(소유 도메인)가 같은 경로
- `cross-site`: `same-site`가 아닌 경로

| URL A                 | URL B                      | same-site?    |
| --------------------- | -------------------------- | ------------- |
| `https://example.com` | `https://api.example.com`  | ✅             |
| `https://example.com` | `http://example.com`       | ✅ (scheme 무시) |
| `https://example.com` | `https://example.com:8080` | ✅ (port 무시)   |
| `https://example.com` | `https://evil.com`         | ❌             |


#### SameSite 쿠키 속성

| 값        | 같은 사이트 요청 | cross-site top-level GET | cross-site POST / iframe / AJAX |
| -------- | --------- | ------------------------ | ------------------------------- |
| `Strict` | 전송        | 미전송                      | 미전송                             |
| `Lax`    | 전송        | 전송                       | 미전송                             |
| `None`   | 전송        | 전송                       | 전송 (`Secure` 필수)                |
- top-level: 주소 표시줄의 URL이 바뀌는 네비게이션 
	- 예: 링크 클릭, 주소 입력, 폼 제출, 리다이렉트 
- subresouce: 주소 표시줄의 URL이 바뀌지 않고 페이지 안에 존재
	- 예: `<img>`, `<iframe>`, `fetch()`, `<script>`, `<link>`

1. `SameSite=Strict`: 가장 보안이 강함. 외부 사이트에서 `bank.com` 링크를 클릭해도 쿠키가 안감. (사용자 입장에서 로그인이 안 되어있음.) UX가 불편할 수 있음. 
2. `SameSite=Lax`: **현대 브라우저의 기본값**
	- 같은 사이트 요청: 쿠키 전송
	- 외부 클릭 GET 요청: 쿠키 전송
	- 외부 사이트에서 보내는 POST, iframe, AJAX, img 요청: 쿠키 미전송
3. `SameSite=None`: cross-site 요청에도 쿠키를 보냄. 


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

