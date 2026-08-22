### 기본 설정

- 패키지 설치
	- `pip install fastapi`
	- `pip install uvicorn`: 내장 톰캣 같은 엔진

``` python
app = FastAPI(title="Scene Scan Worker")

# 중략 ...

if __name__ == "__main__":
    import uvicorn
    
    uvicorn.run("worker.main:app", host="0.0.0.0", port=8000, access_log=False)
```
- `app` ("worker.main:app"): 실행할 FastAPI 인스턴스 위치, 문자열로 전달하면 `reload` 옵션을 사용 가능
- `host` ("0.0.0.0"): 서버가 수신할 IP 주소입니다. `0.0.0.0`은 모든 네트워크 인터페이스를 허용
- `port` (8000): 서버가 바인딩할 포트 번호
- `reload` (bool): 코드 변경 시 서버를 자동으로 재시작, 개발 환경(Local)에서만 `True`로 설정
- `workers` (int): 구동할 프로세스 개수. `reload`가 `True`일 때는 사용할 수 없으며, 보통 성능 향상을 위해 CPU 코어 수에 맞춰 설정
- `access_log` (bool): 모든 HTTP 요청마다 찍히는 로그(200 OK 등)의 출력 여부


``` python 
@app.post("/scout/scan", response_model=ScanResponse)
def scan_scene(
    dto: ScanRequest,
    x_request_id: Optional[str] = Header(None, alias="X-Request-ID"),
    x_video_id: Optional[str] = Header(None, alias="X-Video-ID")
):
	# 작업
	return ScanResponse(...)
```
1. 데코레이터 옵션 
	- **`response_model`**: 응답 데이터의 타입을 정의
	- **`status_code`**: 성공 시 기본 응답 코드(예: `201 Created`)를 지정 가능
	- **`tags`**: Swagger UI에서 API들을 그룹화하여 보여줄 때 사용
	- `summary` / `description`: API에 대한 요약과 상세 설명을 추가하여 문서 가독성을 높임.
2. 파라미터 정의
	- `alias` : HTTP 헤더(`X-Request-ID`)처럼 하이픈(-)이 포함된 이름은 파이썬 변수명으로 쓸 수 없음. 이때 `alias`를 사용하면 외부 인터페이스와 내부 변수명을 깔끔하게 매핑 가능
	- `Header(None)`: 파라미터가 필수인지 아닌지 결정. 
		- None: 값이 없어도 에러가 나지 않음. (선택)
		- 옵션 X: 값이 없으면 `422 Unprocessable Entity` 에러를 응답 (필수)
	- `embed=True` (Body 전용): 보통 Body를 하나만 받을 때, FastAPI는 값을 그냥 String으로 받으려 함. 하지만 프론트엔드에서 항상 `{ "item_id": "abc" }` 처럼 **JSON 객체**로 보내고 싶을 때 `embed=True` 사용
	- `Depends`: 여러 API에서 **공통으로 사용하는 로직**(로그인 체크, 데이터베이스 연결, 로깅 등)을 함수로 분리해서 재사용할 때 사용
``` python
from fastapi import Depends

# 1. 공통 로직 정의 (예: 토큰 검증)
def verify_token(x_token: str = Header(...)):
    # 검증 로직
    return x_token

# 2. API에서 사용
@app.get("/secure-data")
def get_data(token: str = Depends(verify_token)):
    # verify_token이 먼저 실행되고, 성공했을 때만 이 함수가 실행됨
    return {"message": "보안 데이터 접근 성공!", "user_token": token}
```


### `__init__.py` 

1. 패키지 인색: 이 파일이 있어야 패키지로 인식이 되어 다른 곳에서 파일들을 불러올 수 있음. 
	- `import worker` 또는 `from worker.main import app`
2. 가독성 및 관례: 현대 파이썬에서는 자동으로 인식이 되지만 패키지임을 알릴때 사용함. 
3.  한번에 내보내기
``` python
# 밖에서 `from worker import ScanRequest` 처럼 짧게 가져다 쓸 수 있습니다.
from .schemas import ScanRequest
```
- (전) `from worker.schemas import ScanRequest`
- (후) `from worker import ScanRequest`


### DTO 

- 보통 `schemas.py`에 DTO를 모아둠.
``` python 
# schemas.py
from pydantic import BaseModel
from typing import Optional

class ScanRequest(BaseModel):
    videoDownloadUrl: str # 필수 값
    startTime: float
    # 선택 값 
    filterOptions: Optional[str] = None  # 1. Optional을 사용하는 방식 
    videoName: str | None = None # 2. | None 사용 (최신 권장) 
```


### 외부 환경변수

##### 기본적인 방식
- settings를 import해서 사용
``` python 
import os

class Settings:
    # Redis Configuration
    REDIS_HOST: str = os.getenv("REDIS_HOST", "localhost") # 기본값 localhost
    REDIS_PORT: int = int(os.getenv("REDIS_PORT", "6379"))
    REDIS_PASSWORD: str = os.getenv("REDIS_PASSWORD", "")

settings = Settings()
```

##### Pydantic Settings

- 패키지 설치: `pip install pydantic`

``` python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # 자동으로 환경 변수명을 읽어오고(REDIS_HOST -> redis_host), 타입 검증까지 해줌
    redis_host: str = "localhost"
    redis_port: int = 6379

    class Config:
        env_file = ".env" # .env 파일도 자동으로 읽어옴
```


### 로그 공통화

1. `ContextVar` 설정
	- 현재 작업 줄기(Context)에서만 유효한 전역 변수
	- 스프링의 ThreadLocal과 비슷한 개념이지만 스프링은 요청과 쓰레드가 1대1 방식이지만 FastAPI 같은 비동기 프레임워크는 1개의 쓰레드가 많은 요청을 왔다 갔다 하며(await) 처리함. `ContextVar`는 비동기 환경에서도 각각의 작업 흐름에 맞는 데이터를 정확히 찾아주는 역할까지도 함.

``` python
from contextvars import ContextVar

# 기본값은 빈 문자열 (네 SafeFormatter랑 궁합 맞춤)
request_id_ctx = ContextVar("request_id", default="")
video_id_ctx = ContextVar("video_id", default="")
job_id_ctx = ContextVar("job_id", default="")

def set_log_context(request_id: str="", video_id: str="", job_id: str=""):
    request_id_ctx.set(request_id or "")
    video_id_ctx.set(video_id or "")
    job_id_ctx.set(job_id or "")

def clear_log_context():
    request_id_ctx.set("")
    video_id_ctx.set("")
    job_id_ctx.set("")
```

2. 필터 설정
- `logging.Filter`를 상속 받아서 구현
``` python 
# 1. record 객체에 Context 자동 주입 필터 (핵심)
class ContextFilter(logging.Filter):
    def filter(self, record):
        record.request_id = request_id_ctx.get()
        record.video_id = video_id_ctx.get()
        record.job_id = job_id_ctx.get()
        return True

# 2. stdout으로 정상 로그만 보내기 위한 필터 추가
class StdOutFilter(logging.Filter):
    def filter(self, record):
        # WARNING(30) 보다 낮은 레벨(INFO, DEBUG)만 통과시킴
        return record.levelno < logging.WARNING
```

3. 포매터 설정
- `logging.Formatter`를 상속 받아서 구현
``` python
# 값이 없어도 에러내지 않는 커스텀 포맷터 정의
class SafeFormatter(logging.Formatter):
    def formatMessage(self, record):
        for key in ['request_id', 'video_id', 'job_id']:
            if not hasattr(record, key):
                setattr(record, key, "")
        return super().formatMessage(record)
```

4. 로거 설정
``` python
def setup_logger(name: str = "worker"):
    formatter = SafeFormatter( # 기본인 logging.Formatter를 사용해도 됨
        fmt=(
            "%(asctime)s "
            "%(levelname)-5s "
            "[%(request_id).8s] "
            "[%(video_id).8s] "
            "[%(job_id).8s] "
            "%(name)s"
            ": "
            "%(message)s"
        ),
        datefmt="%Y-%m-%d %H:%M:%S",
        style='%'
    )

    logger = logging.getLogger(name) # 로거 인스턴스 생성
    logger.propagate = False # 전파 방지


    if not logger.handlers: # 핸들러 중복 방지
        logger.setLevel(logging.INFO) # INFO 레벨 이상
        
        # 기존의 handler를 stdout_handler와 stderr_handler 2개로 분리
        # --- stdout 핸들러 (INFO, DEBUG용) ---
        stdout_handler = logging.StreamHandler(sys.stdout)
        # 포매터 지정
        stdout_handler.setFormatter(formatter)
        # 필터 추가
        stdout_handler.addFilter(ContextFilter())
        stdout_handler.addFilter(StdOutFilter()) # 에러는 통과 못하게 막음

        # --- stderr 핸들러 (WARNING, ERROR, CRITICAL용) ---
        stderr_handler = logging.StreamHandler(sys.stderr)
        stderr_handler.setLevel(logging.WARNING) # WARNING 이상만 통과
        stderr_handler.setFormatter(formatter) # 포매터 지정
        stderr_handler.addFilter(ContextFilter()) # 필터 추가

        logger.addHandler(stdout_handler)
        logger.addHandler(stderr_handler)

    return logger
```

- `logging.getLogger(name)`: 주어진 이름을 가진 로거 인스턴스를 가져오거나 생성
	- 보통 `__name__` 값을 넣어서 프로젝트에서 동일한 로거 인스턴스를 사용함.
- `logger.propagate = False`: False로 설정하면 로그가 중복으로 찍히는 것을 방지할 수 있음. 
	- 파이썬의 로거는 트리 구조(예: `worker.main` 로거의 부모는 `worker`, 그 부모는 `Root` 로거)라서 기본적으로 하위 로거에서 발생한 로그는 부모 로거로 전파되어 한 번더 출력이 됨. 

5. 사용 
	- 각 모듈에서 로거 생성 `logger = setup_logger(__name__)` 
	- `logger.info()`, `logger.error`, ... 


### Redis

- 패키지 설치: `pip install redis`

``` python
redis_client = redis.Redis(
    host=settings.REDIS_HOST, # Redis 서버의 주소
    port=settings.REDIS_PORT, # Redis가 사용하는 포트 번호
    password=settings.REDIS_PASSWORD if settings.REDIS_PASSWORD else None, # 비밀번호 인증
    decode_responses=True, # Redis에서 가져온 데이터를 파이썬 문자열(str)로 자동 변환
    # 타임아웃 설정
    socket_timeout=2.0, # 처음 연결을 맺을 때 2초만 기다림
    socket_connect_timeout=2.0 # 연결된 후 2초만 기다림
)
```


### 자식 프로세스 subprocess

- 자식 프로세스 생성 및 실행
##### subprocess.Popen

``` python
# 프로세스 실행 시작
process = subprocess.Popen(
	cmd, # 실행할 명령어와 옵션들의 리스트
	stdout=subprocess.PIPE, # 프로세스가 화면에 출력할 내용을 파이썬이 읽을 수 있게 통로(Pipe)로 연결
	stderr=subprocess.PIPE, 
	text=True # 출력되는 데이터를 bytes가 아니라 문자열(str)로 알아서 변환해서 받음
)

try:
	# 작업이 결과를 대기 및 출력 결과를 stdout, stderr로 가져옴
    stdout, stderr = process.communicate(timeout=SCAN_TIMEOUT_SEC)
except subprocess.TimeoutExpired: # 타임아웃 발생
    process.kill()
    _, stderr = process.communicate() # kill 작업을 끝날때까지 대기
    # 타임아웃 처리 로직
    raise FFmpegError(ErrorCodes.TIMEOUT_ERROR)

if process.returncode != 0: # 프로세스 실행 실패 (0: 성공, 그 외: 실패)
    # 에러처리 로직

# 이후 로직
```
- 실행 직후 바로 다음 줄의 파이썬 코드가 실행됨.`communicate()`(wait를 포함함)를 만나면 그때부터 프로세스 실행 결과를 기다림.
- 타임아웃: 타임아웃 발생 시 예외만 던질 뿐, 프로세스는 여전히 백그라운드에서 돌고 있어서 직접 `kill()`을 해줘야함.
- 반환 값: 프로세스 객체 자체를 반환
##### subprocess.run

``` python 
import subprocess

try:
    # run()은 프로세스가 종료될 때까지 기다립니다.
    result = subprocess.run(
        cmd,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True,
        timeout=SCAN_TIMEOUT_SEC  # 타임아웃 설정을 여기서 바로 합니다.
    )
    
    # 결과 사용
    stdout = result.stdout
    stderr = result.stderr

    if result.returncode != 0:
        # 에러 처리 로직 (예: raise FFmpegError...)

except subprocess.TimeoutExpired as e:
    # run()은 타임아웃 발생 시 '자동으로' 자식 프로세스를 kill하고
    # 지금까지 쌓인 stdout, stderr를 e.stdout, e.stderr에 담아줍니다.
    # 따로 process.kill()을 호출할 필요가 없습니다.
    raise FFmpegError(ErrorCodes.TIMEOUT_ERROR)
```
- 프로세스가 완전히 끝날 때까지 파이썬 코드가 기다림.
- 타임아웃: 타임아웃이 발생 시 내부적으로 해당 프로세스를 `kill()`하고 정리가 끝난 상태로 예외를 던짐
- 반환 값: 실행 결과를 담은 `CompletedProcess` 객체를 반환함. (`result.stdout`, `result.returncode` 등으로 접근)


### 비동기 병렬 처리 ThreadPoolExecutor

``` python
from concurrent.futures import ThreadPoolExecutor, as_completed

	# 최대 10개의 워커(스레드)로 병렬 업로드 진행
    with ThreadPoolExecutor(max_workers=10) as executor:
        future_to_file = []
        ...
        future = executor.submit(실행할 작업 메서드, 파라미터1, 파라미터2, ...) # 작업이 예약된 Future 생성 
        futures.append(future) # 리스트에 추가

		# 먼저 완료된 작업을 처리함. 
        for future in as_completed(future_to_file):
            res_file_name = future.result()
            # 결과 처리 로직
```
- `with ThreadPoolExecutor(max_workers=10) as executor`: 워커(스레드) 10개를 생성, 사용한 자원은 알아서 반환함.
- `executor.submit()`: 작업이 예약된 Future를 생성, 워커는 Future의 작업을 수행.
- `as_completed()`: 작업이 완료되는 순서로 처리함. (메인스레드)