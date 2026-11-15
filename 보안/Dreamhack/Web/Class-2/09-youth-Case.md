---
tier: Silver 3
topics:
  - Case Mapping Collision
  - Path Normalization
solved-with: Research
---
### 문제 구조

- `nginx.conf`: nginx 서버가 리버스 프록시로 존재하는데 `/shop`으로의 경로를 차단함. 
``` nginx.conf
http {
    server {
        listen 80;
        listen [::]:80;
        server_name  _;
        
        location = /shop {
            deny all;
        }

        location = /shop/ {
            deny all;
        }

        location / {
            proxy_pass http://app:3000/;
        }
    }
}
```

- `/shop`: `leg` 파라미터를 소문자로 변환해서 flag와 비교해서 차단하고 다시 대문자로 만들어서 `words`에서 가져옴. `words`에는 FLAG라는 이름으로 플래그가 저장되어 있음. 
``` js
function search(words, leg) {
    return words.find(word => word.name === leg.toUpperCase())
}

app.post("/shop",(req, res)=>{
    const leg = req.body.leg.toLowerCase()

    if (leg == 'flag'){
        return res.status(403).send("Access Denied")
    }

    const obj = search(words,leg)

    if (obj){
        return res.send(JSON.stringify(obj))
    }

    return res.status(404).send("Nothing")
})
```

### 분석 및 풀이

**nginx 우회**: Nginx 서버와 백엔드 서버(여기서는 Express.js)와의 **경로 정규화(Path Normalization)** 차이 이용

`/shop\xA0`는 Nginx에서 특정 버전에서 `/shop`으로 인식하지 못하는데 Express에서는 정규화를 통해서 `/shop`으로 변환됨. 
1. Burp Repeater에서 `/shopX` 입력 
2. `X`를 드래그 하고 Inspector 탭에서 Hex를 `a0`로 변경


**flag 우회**: `toLowerCase()`는 'flag'로 인식이 안되지만 `toUpperCase()`가 되면 'FLAG'로 인식되는 문자가 있음. 
ﬂ (`U+FB02`)는 'fl'이 아니지만 `toUpperCase()`가 되면 'FL'로 인식됨. 

### 시행 착오

nginx에서는 `/shop`으로 처리되지 않지만 백엔드 서버에서는 `/shop`으로 처리되는 것을 찾으려고 검색을 해봤지만 나오지 않았음. 그래서 그냥 'nginx deny all 우회' 라고 검색하고 특정 nginx 버전에서는 hex를 활용한 취약점이 있음을 알게 됨. 

nginx 취약점을 찾다가 키워드 힌트를 받으려고 드림핵에서 키워드를 열었는데 Case Mapping Collision가 있길래 검색해봤음. 그런데 다음 단계인 flag를 찾아내는 것에 관한 취약점이어서 스포를 당함.

### 새롭게 배운 점

#### 경로 정규화

서버가 URL 경로를 일정한 규칙에 따라 표준적인 형태로 변환하는 과정. (라우팅 되기 전에 적용됨.)

서버 별로 경로를 정규화하는 규칙이 다른데 이때 경로 파싱 불일치 취약점이 발생할 수 있음.
예: `/admin\xA0`
- nginx: `\xA0`를 제거할 공백으로 보지 않고 URL 경로에 포함된 바이트로 취급
- Node.js: `\xA0`를 특수 공백으로 보고 제거

| 표기     | 16진수 값 | Unicode  | 의미                             |
| ------ | ------ | -------- | ------------------------------ |
| `\x20` | `0x20` | `U+0020` | Space (일반 공백)                  |
| `\x09` | `0x09` | `U+0009` | Horizontal Tab (수평 탭)          |
| `\x0A` | `0x0A` | `U+000A` | Line Feed (줄바꿈)                |
| `\xA0` | `0xA0` | `U+00A0` | Non-Breaking Space (줄바꿈 없는 공백) |
참고: Unicode는 해당 바이트를 Latin-1(ISO-8859-1)로 해석했을 때의 코드 포인트

URL에 Hex(16진수) 삽입하기 
1. Burp Repeater에서 `/adminX` 입력 
2. `X`를 드래그 하고 Inspector 탭에서 Hex를 `a0`로 변경

#### 대소문자 변환 충돌

서로 다른 유니코드 문자가 대소문자 변환(Case Mapping)을 거치면서 동일한 문자열로 변환되는 현상
어떤 입력값을 소문자/대문자일 때만 비교해서 차단하는 방식이라면 우회가 가능함.
https://gosecure.github.io/unicode-pentester-cheatsheet/

**예시**
`ﬂ` (U+FB02)은 `f`와 `l`이 결합된 하나의 Unicode 문자로, **Ligature(합자)** 라고 함.

```
"ﬂ".toLowerCase() // "ﬂ" 
"ﬂ".toUpperCase() // "FL"
```

여기서 `"ﬂ"`와 `"fl"`은 서로 다른 문자열이지만, 대문자로 변환하면 동일한 `"FL"`이 됨.

