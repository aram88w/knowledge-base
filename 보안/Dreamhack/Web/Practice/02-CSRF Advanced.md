---
tier: Silver 4
topics:
  - CSRF
solved-with: Self
---
### 문제 구조

session_storage: 서버에서 관리하는 로그인 세션 스토리지, session_id:username 저장
token_storage: CSRF 토큰을 관리하는 스토리지, session_id:csrftoken 저장

- `/`: 쿠키에서 sessionid를 가져오고 해당 sessionid로 session_storage에서 username을 조회, admin인 경우 플래그 획득
- `/vuln`: `param` 쿼리 파라미터를 xss_filter(frame, script, on을 `*`로 대체)를 한번 거쳐서 바로 보여줌. XSS 취약점 존재
- `/flag`: `param` 쿼리 파마리터 값으로 `/vuln?param=...` url을 만들고 **크롬 봇에서 admin으로 로그인한 후 해당 url로 이동**
- `/login`: 로그인에 성공하면 
	- session_id를 랜덤하게 생성하고 session_storage에 로그인한 username을 저장
	- username + request.remote_addr(접속한 IP)를 md5 해싱해서 CSRF 토큰으로 token_storage에 저장
	- 응답 쿠키에 session_id를 담음 
- `/change_password`: GET 요청으로 비밀번호를 변경하는 엔드포인트
	- 쿠키의 session_id로 token_storage에서 가져온 csrf_token을 폼에 hidden으로 추가
	- 쿠키의 session_id로 가져온 csrf_token이랑 쿼리 파라미터로 들어온 `csrftoken`이 동일한지 확인 후 동일하면 `pw` 쿼리 파라미터의 비밀번호로 변경

### 분석 및 풀이 과정

목표: `/flag`에서 요청을 보내서 admin의 session_id를 기반으로 GET `/change_password` 요청으로 비밀번호 변경

token_storage에 저장되는 CSRF 데이터는 username+request.remote_addr를 해싱한 것이므로 예측이 가능함. 서버에서 admin으로 접근하는 경우에는 `admin127.0.0.1`를 md5로 해싱한 값인  `7505b9c72ab4aa94b1a4ed7b207b67fb`임. 
따라서 `/flag`에서 `/vuln?param=<img src='/change_password?pw=1111&csrftoken=7505b9c72ab4aa94b1a4ed7b207b67fb'>`를 넣어서 요청을 보내면 됨. 
1. 크롬 봇에서 admin 계정으로 로그인 하는 순간, session_id가 생성되고 session_storage에 username, token_storage에 CSRF 토큰이 저장되고 session_id는 쿠키에 담김. 
2. 크롬 봇이 `/vuln?param=...`으로 접속하면 `/change_password`로 요청을 보내고 csrftoken의 값이 우리가 예측할 수 있는 값이기에 일치하여 비밀번호가 변경됨. 

