### 문제 구조

`/get_info`에서 받는 `userid`를 검사하지 않고 내부 API 요청 URL에 그대로 삽입함. 

``` python
userid = request.form.get('userid', '')
info = requests.get(f'{API_HOST}/api/user/{userid}').text
```

### 나의 시도 

`../flag`로 내부 주소 `/api/flag`에 접근해서 플래그 값을 받아오려고 했는데 입력값이 `undefined`가 됨. 
개발자도구 - 요소를 확인해보니 프론트엔드에서 `userid`의 입력값을 제한하고 있었고 포스트맨으로 우회해서 성공함. 

