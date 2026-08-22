## API 테스트

API 테스트: API 엔드포인트와 데이터 처리 로직의 취약점 탐색
- 엔드포인트 정찰: 문서화되지 않았거나 UI에 보이지 않는 내부 API 엔드포인트 찾기
- 파라미터 조작: RESTful API나 JSON 구조 내부의 입력값을 변조해 보기 
- 취약점 검증: 공격 시도가 서버 로직을 우회하거나 데이터에 영향을 주는지 테스트하기

### API 재검토

API에 대한 가능한 많은 정보를 수집해야함. 다음과 같은 정보를 확인해야함. 

1. **필수 매개변수와 선택적 매개변수**: 어떤 파라미터(쿼리 파라미터나 바디)를 처리하는지 확인
	- 예: 프론트엔드 UI에서는 `username`, `email`만 입력받도록 만들어둔 `POST /api/user/profile` 엔드포인트가 있다고 가정
    - 백엔드 API를 분석해 보니 hidden 필드로 `role`이라는 선택적 매개변수를 받아 처리한다는 걸 알게 되고 요청 바디에 `{"username": "kim", "email": "a@a.com", "role": "admin"}`을 슬쩍 얹어 보내면, 권한이 상승하는 **대량 할당 취약점**이 터지는지 확인

2. **지원하는 HTTP 메서드**: GET, POST, PUT, DELETE 등이 통하는지 확인
	- 예: 화면에서는 단순히 데이터를 조회하는 `GET /api/posts/123`만 사용하지만, 서버가 `DELETE /api/posts/123` 요청도 막지 않고 그대로 처리하는지 확인

3. **미디어 형식 (Content-Type)**: 다른 형식(JSON, XML 등)을 처리하는지 확인
	- 예: 원래 `Content-Type: application/json`으로 처리하는 API인데, `Content-Type: application/xml`로 헤더와 바디를 바꿔서 날렸을 때 서버가 XML 파싱을 시도하다가 **XXE 취약점**이 터지는지 확인

4. **인증 메커니즘**: 요청할때 JWT 토큰 등을 넣어야하는지 확인
	- 예: `Authorization: Bearer <token>` 헤더를 아예 빼버리거나, 토큰의 서명 부분을 지우고 넘겼을 때도 서버가 데이터를 그냥 내어주는지(인증 우회) 확인

5. **속도 제한**: 짧은 시간 많은 요청을 했을때 차단당하는지 확인
	- 예: 비밀번호 찾기 인증번호 4자리를 입력받는 API `/api/verify-code`에 1초 동안 1,000번의 요청을 연속으로 보냈을 때 서버가 `429 Too Many Requests`로 차단하는지, 아니면 다 받아주어서 **브루트포스**로 인증번호가 뚫리는지 확인합니다.


#### API에 대한 정보를 얻는 방법

1. 개발자의 실수로 운영 서버에 문서가 노출된 경우 (보안 설정 오류)
	- `/api-docs` 같은 경로에 API 문서가 그대로 노출될 수 있음. 
2. 프론트엔드 JS 번들 파일 분석
	- 브라우저가 다운로드한 **프론트엔드 자바스크립트 파일(`main.js`, `app.js` 등)** 내부를 뜯어보면 API 요청 경로, JSON 파라미터 구조, hidden 필드 등이 코드 형태로 다 적혀 있음.


### API 문서

API는 일반적으로 개발자가 사용 및 통합 방법을 알 수 있도록 문서화되어 있음. 
현대에는 백엔드 개발자가 일일이 문서를 작성하지 않아도, 코드에 주석이나 어노테이션 몇 줄만 달면 `Swagger` 같은 도구가 `/swagger-ui.html`이나 `/v3/api-docs` 형태로 **웹 인터페이스 및 JSON 문서**를 자동으로 생성해줌. 

- 사람이 읽을 수 있는 형태: 개발자들끼리 협업을 하기 위한 목적으로 작성됨 
- 컴퓨터가 읽을 수 있는 형태: API 통합 및 검증과 같은 작업을 자동화하기 위해 JSON이나 XML 같이 구조화된 형식으로 작성된 것

1. **Swagger UI** (가장 일반적) : 웹 브라우저에서 `/swagger-ui.html`이나 `/docs` 같은 경로로 접속하면, API 목록이 시각적으로 깔끔하게 나오고 직접 API 테스트를 해볼 수 있음. 혹은 컴퓨터가 읽는 원본 파일(`openapi.json` 또는 `swagger.yaml`)을 자동으로 만들어냄.
2. Postman / Stoplight: API 모음을 만들어 Postman 워크스페이스에 올려 공유
3. 사내 위키 및 기술 문서 플랫폼: 자동화 도구를 사용하지 않고 개발자가 직접 정리할 수도 있음. 


보통은 일반 사용자는 읽을 수 없도록 운영 서버에서는 설정을 꺼둬서 접근이 불가능하게 했을 가능성이 큼. 
하지만 개발자의 실수로 운영 서버에서 문서가 노출 될 경우가 생각보다 자주 발생하므로 가능성이 있는 대표적인 경로들에 공격을 해봐야함. 
- `/api`
- `/swagger/index.html`
- `/swagger-ui.html`
- `/openapi.json`
- `/api-docs`

만약 `/api/swagger/v1/users/123`에서 API 문서가 노출된 경우에 하위 경로들도 다 살펴봐야함. 
- `/api/swagger/v1`
- `/api/swagger`
- `/api`

실무에서는 Intruder 기능이나 **ffuf, DirBuster** 같은 툴을 사용해서 퍼징을 자동화함. 
Github에는 `SecLists`라는 유명한 해킹 사전(Wordlist)이 있고 거기에는 API 문서의 이름으로 자주 사용하는 파일명들이 있음. 


### 엔드포인트 탐색

#### 엔드포인트 탐색 및 발굴

1. JavaScript 파일 및 URL 패턴 분석
	- `/api/`, `/v1/` 같은 기본 URL 구조 분석 및 예측
	- JS 파일 내부의 하드코딩된 API 경로, 미사용 함수 참조 추출 (수동 분석 또는 **JS Link Finder** 등의 도구 활용)
2. Burp Intruder를 활용한 경로 퍼징
	- 이미 알려진 구조(예: `/api/user/update`)의 끝부분에 단어목록을 대입하여 숨겨진 기능 경로(예: `/api/user/delete`, `/api/user/add`) 탐색
3. **Discover Content** 같은 도구 사용: 링크가 걸려있지 않아 화면상에서는 절대 클릭해서 들어갈 수 없는 경로(예: `/backup/`, `/dev/test.php`, `/admin/config.json`)를 **사전(Wordlist)에 등록된 단어들을 하나씩 요청해 보며** 존재 여부를 확인


#### 엔드포인트 확장

1. HTTP 메서드 교체
	- 동일한 URL 경로(예: `/api/tasks`)에 `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `OPTIONS` 등을 변경해 보내면서, 문서에 없거나 권한 검증이 누락된 관리자용 기능이 동작하는지 테스트
2. Content-Type 및 데이터 포맷 변경
	- `application/json`으로 전송하던 요청을 `application/xml`이나 `x-www-form-urlencoded` 등으로 변경 (**Content-Type Converter** 도구 활용)
	- 백엔드 파서(Parser)의 로직 차이, XXE 취약점 유발, 에러 메시지를 통한 내부 정보 유출 여부 확인


#### 숨겨진 파라미터 찾기

API 요청이나 문서에는 명시되어 있지 않은 매개변수들을 넣었을때 작동할 수 있고 이러한 매개변수들을 활용하여 애플리케이션의 동작을 변경할 수 있음. 

1. Burp Intruder를 활용한 퍼징
	- 일반적으로 사용되는 매개변수 이름들이 포함된 단어 목록을 활용하여 탐색
2. **Param Minor** 도구 사용
	- Param Minor는 스코프에서 얻은 정보를 바탕으로 애플리케이션에 필요한 파라미터 이름들을 자동으로 추측해냄. 한번의 요청으로 최대 65536개의 파라미터 이름을 자동으로 추측할 수 있음. 


#### 대량 할당 취약점

서버가 매개변수를 대량 할당(자동 바인딩)으로 처리하는 경우, 개발자가 의도하지 않은 매개변수들을 처리할 수 있음. 

- 개별 할당 (안전한 방식): 개발자가 허용할 필드만 하나씩 직접 지정함. 
``` js
// 클라이언트 요청에서 필요한 값만 골라 할당
user.username = req.body.username;
user.email = req.body.email;
// isAdmin 같은 다른 값은 요청에 들어있어도 무시됨
```

- 대량 할당 (프레임워크의 자동 바인딩 기능: 요청에 들어온 모든 키-값을 객체 필드에 한번에 자동으로 매핑해줌. 
``` js
// 들어온 JSON 데이터 '전체'를 User 객체에 통째로 덮어씌움 (단 한 줄!)
user.update(req.body);
```


이러한 상황을 가정할때, 서버가 대량할당을 사용하고 있다면 `isAdmin` 매개변수를 값을 수정할 수 있음. 
``` json
// PATCH /api/users 요청
{
    "username": "wiener",
    "email": "wiener@example.com",
}

// GET /api/users/123 응답 
{
    "id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "isAdmin": "false"
}
```

``` json
// PATCH /api/users 조작된 요청
{
    "username": "wiener",
    "email": "wiener@example.com",
    "isAdmin": true,
}
```

