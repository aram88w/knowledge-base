---
tier: Silver 4
topics:
  - NoSQL Injection
  - CouchDB
solved-with: Writeup
---
### 문제 구조

``` js
// app.js
...
const nano = require('nano')(`http://${process.env.COUCHDB_USER}:${process.env.COUCHDB_PASSWORD}@couchdb:5984`);
const users = nano.db.use('users');
...

/* POST auth */
app.post('/auth', function(req, res) {
    users.get(req.body.uid, function(err, result) {
        if (err) {
            console.log(err);
            res.send('error');
            return;
        }
        if (result.upw === req.body.upw) {
            res.send(`FLAG: ${process.env.FLAG}`);
        } else {
            res.send('fail');
        }
    });
});
```

CouchDB는 `users.get(uid)`을 하면 GET `http://${COUCHDB_USER}:${COUCHDB_PASSWORD}@couchdb:5984/users/uid` 요청을 CouchDB에 보냄.

HTTP 요청에서 `uid`를 이용해서 CouchDB에 요청을 보내고 응답 데이터의 `upw`랑 HTTP 요청에서의 `upw`랑 비교해서 동일하면 플래그 획득

### 분석 및 풀이

CouchDB에는 특수 구성 요소인 `_all_docs`, `_security` 등에 접근해 데이터베이스의 정보를 획득할 수 있음.

``` json
{
    "uid": "_all_docs"
}
```
`upw`를 아예 없애고 `uid`를 `_all_docs`로 보내면 CouchDB에서 `_all_docs` 데이터를 가져오는데 `result.upw`는 `undefined`가 되고 `req.body.upw`도 `undefined`가 돼서 일치하기 때문에 플래그를 반환함. 

### 시행 착오

답을 볼 생각은 없었는데 CouchDB 취약점 검색 중에 해당 드림핵 문제에 대해서 스포 당함. 

