---
tier: Bronze 1
topics:
  - Apache
solved-with: Self
---
### 문제 구조

목표: admin 계정 세션을 획득 후 `/flag.php`로 접근

- `/login.php`: 로그인을 하면 아이디를 기반으로 `admin.json`, `guest.json`에서 해싱된 비밀번호와 일치하는지 검증 후 일치하면 세션을 부여
- `/edit_profile.php`
	- admin 세션이 아닌 경우 `/`로 리다이렉션 시킴
	- admin 세션인 경우에만 POST 요청으로 사용자의 비밀번호를 수정할 수 있음. 
- `/flag.php`: admin 세션인 경우 플래그를 반환

- `/user/...`: `/admin.json`, `/guest.json`가 저장되어 있음. 
- `.htaccess`: 아파치 서버에서 해당 폴더에 대한 HTTP 접근 설정 파일
	- DELETE 요청 외에는 모두 허용함. 
- `000-default.conf`: 아파치 웹 서버 설정 파일

### 분석 및 풀이

1. Apache 설정 (`000-default.conf`)
```
<Directory /var/www/html/user/>
    DAV On
    Options Indexes
    AllowOverride All
    Require all granted
</Directory>
```
- `DAV On`으로 WebDAV 기능이 활성화되어 파일에 대한 HTTP `PUT` 요청을 처리할 수 있음. 

2. `.htaccess`
```
<Limit DELETE>
    Require all denied
</Limit>
```
- `DELETE` 요청만 거부

3. Dockerfile
```
RUN chmod 777 /var/www/html/user/
RUN chmod 666 /var/www/html/user/*.json
```
- 웹 서버 프로세스가 `admin.json` 파일을 수정할 수 있는 권한까지 부여되어 있음. 

따라서, PUT `/user/admin.json` 요청으로 비밀번호를 마음대로 수정해서 관리자 계정에 접근할 수 있음. 
또는 웹셸 파일을 업로드해서 `cat /flag`를 실행해서 플래그를 획득할 수도 있음. 
``` cmd
$ curl -T webshell.php http://host8.dreamhack.games:15496/user/
```

### 시행 착오

`/edit_profile.php`가 있길래 여기서 관리자 세션 없이 관리자 계정의 비밀번호를 수정할 수 있는 취약점이 존재하는 줄 알고 열심히 찾아봤지만 전혀 사용되지 않는 경로였음. 
워게임에서 모든 경로나 기능이 사용되어야하는 것은 아닌 것을 깨달았음.

### 새롭게 배운 점

#### Apache 설정

아파치  웹 서버 설정 파일: `000-default.conf` (기본 파일명)
``` conf
<VirtualHost *:80>
  ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html // 웹앱의 루트 디렉토리

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

   <Directory /var/www/html/>
     AllowOverride None
     Require all granted
   </Directory>

  <Directory /var/www/html/user/> // (*) 중요! /user 폴더에 webdav를 허용한다.
      DAV On
      Options Indexes
      AllowOverride All
      Require all granted
  </Directory>
</VirtualHost>
```
- `/user` 디렉토리에 대해서 WebDAV On으로 설정함. 

##### WebDAV

WebDAV(웹 기반 분산 저작 및 버전 관리)는 웹 브라우저로 홈페이지를 읽기(보기)만 하는 것을 넘어, **인터넷을 통해 웹 서버에 있는 파일을 직접 수정하고, 복사하고, 붙여넣고, 삭제할 수 있도록 만든 기술**
예: 네이버 MYBOX, 구글 드라이브

아파치 HTTP 서버에서는 dav, dav_fs 모듈을 사용하여 WebDAV를 지원할 수 있고, 사용자는 WebDAV 클라이언트 등을 사용하여 WebDAV 요청을 보내어 웹앱의 특정 폴더에 파일을 추가하거나 수정할 수 있다.


#### .htaccess

아파치(Apache) 웹 서버에서 디렉터리(폴더) 단위로 설정을 제어할 수 있게 해주는 분산 설정 파일

``` .htaccess
<Limit DELETE>
        Require all denied
</Limit>
```
- 여기서는 HTTP 요청 중 DELETE만 거부하는 설정을 함. 

#### Linux 파일 접근 권한

``` Dockerfile
RUN chmod 777 /var/www/html/user/
RUN chmod 666 /var/www/html/user/*.json
```
- `RUN`: Docker 이미지를 빌드하는 과정에서 명령어를 실행하는 Dockerfile 지시어
- `chmod`: Linux에서 파일이나 디렉터리의 접근 권한을 변경하는 명령어

| 기호  | 의미          | 숫자  |
| --- | ----------- | --- |
| `r` | Read: 읽기    | 4   |
| `w` | Write: 쓰기   | 2   |
| `x` | Execute: 실행 | 1   |
	예: 7 = 4 + 2 + 1이므로 읽기, 쓰기, 실행 전부 가능 

