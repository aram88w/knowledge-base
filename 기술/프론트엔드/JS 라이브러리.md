### Promise 

- 비동기 작업에 대한 결과를 주겠다는 약속을 담은 객체

1. 생성
``` js
const myPromise = new Promise((resolve, reject) => {
  // 여기서 비동기 작업을 수행 (예: 이미지 로드, API 호출)
  const success = true;

  if (success) {
    resolve("성공 데이터 💎"); // 성공 시 호출 (성공 메세지를 담음)
  } else {
    reject("실패 이유 ❌");   // 실패 시 호출 (에러 메시지를 담음)
  }
});
```

2. 사용
``` js
/**
 * 방법 1. .then() / .catch() 방식
 */
 myPromise
  // resolve가 호출되었을 때 실행 
  .then((data) => { 
    console.log(data); // "성공 데이터 💎"
  })
  // reject가 호출되었을 때 실행
  .catch((error) => {
    console.error(error);
  })
  // 성공/실패 여부와 상관없이 마지막에 무조건 실행
  .finally(() => {
    console.log("작업 끝!");
  });
  
/**
 * 방법 2. async / await 방식
 */
async function run() {
  try {
    const data = await myPromise; // resolve될 때까지 기다렸다가 값만 쏙!
    console.log(data); 
  } catch (error) {
    console.error(error); // reject되면 여기로 떨어짐
  }
}
```

Promise / async가 비동기 작업으로 만드는 것이 아니고 기존의 비동기 작업(API 요청, I/O)을 감싸는 것임
JS 엔진은 비동기 작업을 만나면 알아서 다른 작업을 처리하도록 되어있음. 
await은 해당 비동기 작업의 결과가 돌아올때까지 해당 async 함수의 실행 흐름을 멈추는 것뿐임. await가 메인스레드를 다른 작업에 할당하도록 하는 것이 아님. 


### async / await

- async: 이 함수가 비동기임을 명시, Promise 객체로 감싸서 내보냄. 
- await 
	1. Promise가 resolve가 될 때까지 **해당 async 함수의 실행만** 그 자리에서 멈춥
	2. resolve가 호출되면 그 결과값을 꺼내서 변수에 담아둠.
	3. async 함수 안에서만 사용 가능

``` js
async function getData() {
  console.log("1. 요청 시작");
  const response = await fetch("https://api.example.com"); // 여기서 기다리네?
  console.log("3. 데이터 받음");
}

getData();
console.log("2. 두번째로 실행됨");
```


### HTTP 통신

##### 응답 구조

- 성공 응답
``` js
{
  status: 200,  // 1. HTTP 상태 코드
  statusText: 'OK', // 2. HTTP 상태 메시지 (서버 설정에 따라 다름)
  headers: {}  // 3. 응답 헤더 (소문자로 정규화됨)
  config: { // 4. 요청 시 사용했던 설정값
    url: '/api/extract',
    method: 'post',
    headers: { Accept: 'application/json, text/plain, */*' },
    // ...
  },
  request: {} // 5. 실제 요청을 수행한 객체 (브라우저라면 XMLHttpRequest 인스턴스)

  // 6. 서버가 실제로 준 데이터 
  data: {
    success: true,
    data: { id: 1, title: "이미지 추출 결과" },
    error: null
  }
}
```

- 실패 응답
``` js
{
  message: "Request failed with status code 400", // Axios 에러 메시지
  name: "AxiosError", // Axios 에러 이름
  code: "ERR_BAD_REQUEST", // Axios 에러 코드
  request: {}, // 실제 요청을 수행한 객체 
  config: {}, // 요청 시 사용했던 설정값

  // 서버가 응답을 준 경우에만 존재
  response: {
    data: {
      success: false,
      data: null,
      error: { code: 'INVALID_PARAM', message: '잘못된 요청입니다.' }
    },
    status: 400,
    statusText: 'Bad Request',
    headers: { ... },
    config: { ... },
    request: { ... }
  }
}
```

##### Axios

1. 패키지 설치: `npm install axios`
2. 설정: `index.js`
``` js
/**
 * axios 객체 생성
 */
const api = axios.create({
  baseURL: 'https://api.example.com/v1', // 1. 기본 주소
  timeout: 5000, // 2. 응답 대기 시간
  // 3. 공통 헤더 설정
  headers: { 
    'Content-Type': 'application/json',
    'X-Custom-Header': 'foobar'
  },
  withCredentials: true, // 4. 쿠키 공유 설정 (로그인 세션 유지 등에 필요)
  // 5. 기본 파라미터 (모든 요청에 ?type=web 이 붙음)
  params: { 
    type: 'web'
  }
}) 

/**
 * axios 요청 인터셉터
 */
api.interceptors.request.use(
  (config) => {
    // 1. 헤더 설정
    config.headers['Accept-Language'] = i18n.global.locale.value || 'en';
    // 2. 로컬스토리지에서 토큰 꺼내서 넣어주기 (사용자가 중간에 언어를 바꿀 수도 있으니 요청 인터셉터에서 처리)
    const token = localStorage.getItem('accessToken');
    config.headers.Authorization = `Bearer ${token}`;

    return config; // 수정된 설정을 반환해야 요청이 출발합니다!
  },
  (error) => {
    // 요청 준비 단계에서 에러가 나면 로딩을 끄고 에러를 던짐
    return Promise.reject(error);
  }
);

/**
 * axios 응답 인터셉터
 */
api.interceptors.response.use(
  /**
   * [분기 1] HTTP 상태 코드가 2xx인 경우 (성공)
   */
  (response) => {
    const payload = response.data
	// 응답 성공 처리...
    return payload
  },

  /**
   * [분기 2] HTTP 상태 코드가 2xx가 아니거나 네트워크 에러인 경우 (실패)
   */
  (error) => {
    const { response, code } = error
    const status = response?.status

    // 1. 네트워크 연결 안 됨 / 타임아웃
    if (!response || code === 'ERR_NETWORK') {
      return Promise.reject(toNormalizedError({ type: 'NETWORK', ... }))
    }

    // 2. 4xx 클라이언트 에러 (잘못된 요청, 인증 실패, 검증 실패 등)
    if (status >= 400 && status < 500) {
      // 밸리데이션 에러 여부에 따라 VALIDATION 또는 CLIENT 타입으로 reject
      return Promise.reject(toNormalizedError({ type: 'CLIENT', ... }))
    }

    // 3. 5xx 서버 에러 (서버 내부 오류)
    return Promise.reject(toNormalizedError({ type: 'SERVER', ... }))
  }
)
```

3. 사용
- `api.post(url, { a:xxx, b:xxx })`, `api.get(url, params: {...})` 등


### Canvas API

- 웹 페이지에 그래픽을 직접 그릴 수 있는 도구
- 전체적인 흐름
	1. 초기화: Canvas 생성 -> Context(2d) 획득
	2. 데이터 준비: 이미지 로드 또는 비디오 프레임 캡처
	3. 드로잉 작업: `drawImage`로 배경/피사체 배치 -> Path로 보조 도구(그리드, 텍스트) 추가
	4. 데이터 활용: `getImageData`로 직접 분석하거나 `toBlob`으로 데이터 전달
	5. 반복/최적화: `requestAnimationFrame`을 통한 애니메이션 구현 등

3. 기본 설정 
``` html
<!-- Canvas의 크기는 CSS가 아닌 HTML 속성(width, height)으로 설정하는 것이 정확 -->
<canvas id="myCanvas" width="500" height="300"></canvas>
```
``` js
const canvas = document.getElementById('myCanvas'); #
const canvas = document.createElement('canvas');
const canvas = new OffscreenCanvas(width, height);
// 2D 그래픽을 그리기 위한 도구함
const ctx = canvas.getContext('2d');
```
- 도화지(Canvas Elemnet)와 붓(Context)으로 나뉨. 
- 모든 명령은 컨텍스트(ctx)를 통해서 내림.

2. 이미지 그리기
	- `drawImage(image, dx, dy)`: 원본 크기 그대로 그리기
	- `drawImage(image, dx, dy, dw, dh)`: 크기를 조절(Scale)해서 그리기
	- `drawImage(video, sx, sy, sw, sh, dx, dy, dw, dh)`: 이미지의 특정 부분만 잘라내서 그리기

3. 기본 도형 그리기
	1. 사각형 
		- `ctx.fillRect(x, y, width, height)`: 색이 채워진 사각형
		- `ctx.strokeRect(x, y, width, height)`: 테두리만 있는 사각형
		- `ctx.clearRect(x, y, width, height)`: 특정 영역을 지우기
	2. 복잡한 도형: 경로를 시작하고 그려야함.
		``` js
		ctx.beginPath();       // 1. 경로 시작
		ctx.moveTo(50, 50);    // 2. 시작점 이동
		ctx.lineTo(150, 50);   // 3. 직선 경로 추가
		ctx.lineTo(100, 150);  // 4. 다음 직선 추가
		ctx.closePath();       // 5. 시작점과 연결 (선택)
		ctx.stroke();          // 6. 선 그리기 (또는 ctx.fill()로 채우기)
		```
	3. 원과 호 
		``` js
		ctx.beginPath();           // "이제 그리기 시작할게"
		ctx.arc(1, 1, 1, 0, Math.PI * 2); // 중심(1,1), 반지름 1인 원 경로 만들기
		ctx.fillStyle = '#ccc';    // 붓에 묻힐 색상
		ctx.fill();                // 경로 안쪽 채우기 (stroke()는 테두리만 그리기)
		```

4. 데이터 활용 
- Blob 데이터로 저장하는 예시
``` js
// ...
ctx.drawImage(image, 0, 0, canvas.width, canvas.height)
// canvas.toBlob로 Blob 데이터를 생성 (브라우저 메모리에 생성됨)
// (참고) canvas.toBlob 함수는 작업이 끝나면 콜백함수를 호출하도록 설계되어있음. 여기서는 작업이 완료되면 resolve를 호출해서 Promise가 성공적으로 완료되도록 함. 
const blob = await new Promise(resolve => canvas.toBlob(resolve, 'image/jpeg', 0.5))
// 해당 Blob 데이터에 접근할 수 있는 URL 생성
if (blob) thumbnails.push(URL.createObjectURL(blob))
```


### Chart.js

0. 패키지 다운로드 (npm install)
	- `chart.js`
	- `react-chartjs-2`, `vue-chartjs`: 리액트/뷰에서 Chart.js를 컴포넌트 형태로 쓰게 해주는 도구 (래핑)
	- `chartjs-plugin-annotation`: 그래프 위에 '플레이헤드 선'이나 '영역 박스'를 그릴 때 필요
	- `chartjs-plugin-zoom`: 마우스 휠로 확대/축소하고 드래그로 이동할 때 필요

##### 1.  모듈 등록
- Chart.js는 필요한 기능만 골라 쓰는 방식임. 모듈을 등록하는 것부터 시작함

1. 라이브러리에서 차트를 그리는데 필요한 최소 단위 모듈들을 가져오기 
	``` js
	import {
	  Chart as ChartJS,      // 본체
	  CategoryScale,         // X축 (범주형) - 여기서는 사용 안 함 (LinearScale로 대체)
	  LinearScale,           // Y축 (연속적인 숫자 처리)
	  PointElement,          // 그래프의 점
	  LineElement,           // 그래프의 선
	  Title, Tooltip, Legend, // 제목, 툴팁, 범례
	  Filler                 // 선 아래 색상을 채우는 기능 (그라데이션용)
	} from 'chart.js'

	import zoomPlugin from 'chartjs-plugin-zoom'             // 외부 확장: 확대/축소
	import annotationPlugin from 'chartjs-plugin-annotation' // 외부 확장: 플레이헤드 선
	```
	
2. 글로벌 컴포넌트 등록
	``` js
	ChartJS.register(
	  CategoryScale, LinearScale, PointElement, LineElement, 
	  Title, Tooltip, Legend, Filler,
	  annotationPlugin, // 차트 위에 '선'을 그릴 수 있게 됨
	  zoomPlugin        // 차트를 '확대'할 수 있게 됨
	)
	```
	
3. 커스텀 플러그인 확장: Chart.js 기본 옵션으로 안될 때 사용
	``` js
	const gridPatternPlugin = {
		// 실제 데이터를 그리기 직전 라이프사이클 훅
		beforeDatasetsDraw: (chart) => {
		    const { ctx, chartArea, scales } = chart // 차트의 도화지(ctx)와 좌표계(scales)를 가져옴
		    ...
		    ctx.setLineDash([2, 4]) // 점선(빗금) 모양 설정
			ctx.stroke()            // 실제로 선 긋기
			...
		}
	}
	
	// 사용할 플러그인 목록 정의
	const chartPlugins = [gridPatternPlugin]
	
	// <template> 부분에서 차트에 전달
	<Line :plugins="chartPlugins" ... /> 
	```

##### 2. 데이터 구성
- 원본 데이터를 차트가 이해할 수 있는 좌표값으로 바꾸고, 차트에 표시하는(색상 설정) 단계

1. 차트 데이터 구성 
``` js
const chartData = computed(() => {

	// 1. 데이터 소스 가져오기
	const results = videoStore.sceneResults

	// 윈도잉 적용: 현재 가 가시 범위 (+버퍼) 내의 데이터만 필터링
	const filtered = results.filter(item =>
		item.timestamp >= chartMin.value - BUFFER_SEC &&
	    item.timestamp <= chartMax.value + BUFFER_SEC
	)

	// 2. 좌표계 변환
	const dataPoints = filtered.map(item => ({
		x: item.timestamp,
		y: transformY(item.score) // 3. 로그 수식을 적용해 Y축 높이값(0~1)으로 변환
	}))

	// 3. 최종 데이터셋 조립: Chart.js가 요구하는 객체 형태(datasets)로 만듬. 데이터와 스타일이 결합됨
	return {
	    datasets: [
	      {
			label: t('extraction.sceneGraph.chart.datasetLabel'),
			data: dataPoints,
			...
	
```

2. 적용
``` html
<Line ref="chartRef" :data="chartData" :options="chartOptions" ... />
```
- `<Line />`은 `react-chartjs-2`, `vue-chartjs`가 제공하는 차트 컴포넌트 (내부에서 Convas를 사용함)
- chartData(computed)가 바뀔 경우에 `react-chartjs-2`, `vue-chartjs`는 데이터가 바뀌면 자동으로 `chart.update()`를 호출하여 차트를 다시 그림.

##### 3. 차트 옵션 등록
- 차트가 어떻게 움직이고 반응할지 설정하는 단계 (차트의 성능, 반응성, 모든 상호작용의 규칙)

1. 차트 옵션 구성
``` js
const chartOptions = computed(() => {
  return {
	// 1. 성능 및 렌더링 최적화
    responsive: true,
    animation: false,
    parsing: false, // 성능 최적화: 가공된 데이터 포인트 직접 사용
	normalized: true, // 데이터가 이미 x기준으로 정렬되어 있음을 chartjs에 알림
	clip: false, // 라벨(annotation label)이 차트 밖에서도 보이도록 false 설정
	...
	
	// 2. 상호작용 규칙
	interaction: ... // 마우스를 움직일때 어떤 데이터 포인터를 강조할지 지정
	// 가능한 이벤트들을 열어둠. 
	events: ['mousemove', 'mouseout', 'click', 'touchstart', 'touchmove', 'touchend'],
	// 클릭했을 때 이벤트 설정
	onClick: (e, elements, chart) => { ...
	
	// 3. 좌표계 설정: X축, Y축 세부 설정
    scales: { 
      x: { ... },
      y: { ... }
    },
    
    // 4. 외부 플러그인(Zoom, Annotation 등)의 옵션 설정
    plugins: {
	    annotations: { ... },
	    zoom: { ... }, 
	    ...
    }
  }
})
```

2. 적용
``` html
<Line ref="chartRef" :data="chartData" :options="chartOptions" ... />
```
- `<Line />`은 `react-chartjs-2`, `vue-chartjs`가 제공하는 차트 컴포넌트 (내부에서 Convas를 사용함)
- chartOptions(computed)가 바뀔 경우에 `react-chartjs-2`, `vue-chartjs`는 데이터가 바뀌면 자동으로 `chart.update()`를 호출하여 차트를 다시 그림.

##### 4. 이벤트 설정

``` html
<!-- veu-chartjs에서 제공하는 차트 컴포넌트 -->
<Line v-if="videoStore.sceneResults.length > 0" ref="chartRef" :data="chartData" :options="chartOptions" :plugins="chartPlugins"
@mousedown="handleMouseDown" @touchstart="handleTouchStart" /> <!-- 이벤트 등록: handelMouseDown에 이벤트를 구현하면 됨 -->
```
- chartRef.value.chart가 진짜 차트 객체
- 차트 상태를 읽고 싶을 때 chart.scales, chart.canvas, chart.options로 접근
- 차트 동작을 바꾸고 싶을 때 chart.options...나 annotation 값을 바꾼 뒤 chart.update() 함.

1. `chart.canvas`: 차트가 실제로 그려지는 canvas DOM
	- `getBoundingClientRect()`: HTML 요소의 위치와 크기 정보를 가져오는 함수
	- `chart.canvas.style.cursor`: 마우스 커서 변경
2. `chart.scales`: X축, Y축 객체 모음
	- `chart.scales.x.getValueForPixel(pixel)`: 픽셀 좌표를 X축 값으로 변환
	- `chart.scales.x.getPixelForValue(value)`: X축 값을 픽셀 좌표로 변환
3. `chart.options`: 차트 동작 설정 전체 (interaction, zoom, annotation 등)
	- `chart.options.plugins.annotation.annotations`
	- `chart.options.plugins.annotation.annotations.playhead.value`
	- `chart.options.plugins.annotation.annotations.playheadHandle.xValue`
	- `chart.options.plugins.annotation.annotations.thresholdLine`
	- `chart.options.plugins.zoom.pan.enabled`
	- `chart.options.plugins.zoom.zoom.wheel.enabled`
4. `chart.update('none')`: 바꾼 값들을 다시 그리게 하는 함수 (애니메이션 X)


##### 작동 방식

1. `@mousedown` 등의 이벤트에서 차트 위에 커스텀 도형(선, 박스 등)을 그리는 플러그인 annotation을 업데이트함. 
2. annotation은 chartOptions(computed)에 등록되어있고 chartOptions은 `<Line />`에 등록되어 있음. 
3. chartOptions가 바뀔 경우에 `react-chartjs-2`, `vue-chartjs`는 데이터가 바뀌면 자동으로 `chart.update()`를 호출하여 차트를 다시 그림.


### Konva

0. 패키지 다운로드: `npm install konva`

##### 1. Stage와 Layer 준비
- Stage: 전체 캔버스 영역, HTML 컨테이너(`div`)와 연결 
- Layer: Konva 객체를 그리는 영역, Stage 위에 여러 장 겹칠 수 있음.

``` js
// 1. Stage 생성
const stage = new Konva.Stage({
  container: 'container-id', // div의 id나 ref, 전달 받은 div 태그 안에서 새로운 canvas 요소를 생성
  width: width, // getBoundingClientRect()로 div 요소 전체에 스테이지를 그릴 수 있음
  height: height
});

// 2. Layer 생성 및 추가
const layer = new Konva.Layer();
stage.add(layer);

```

##### 2. 구성 요소(Node) 생성 및 조립
- Konva 객체는 트리 구조를 가짐. 
- Shapes: `Rect`, `Image`, `Text`, `Circle` 등
- Group: 여러 요소를 하나로 묶어 관리 (예: 카드 배경 + 이미지 + 텍스트 = `card-root` 그룹)

``` js
// 루트 그룹
const rootGroup = new Konva.Group({
	x: 0, y: 0,
    width: CARD_W, height: CARD_H,
    name: 'card-root', // 이름 지정
    draggable: true, // 모든 환경에서 드래그 활성화
})
rootGroup.setAttr('type', type) // 명시적으로 속성 설정
rootGroup.setAttr('originalIndex', originalIndex) // 고유 번호 저장

// 박스
const bg = new Konva.Rect({
    width: CARD_W, height: CARD_H,
    fill: '#ffffff',
    cornerRadius: 8,
    shadowColor: 'rgba(0,0,0,0.05)', shadowBlur: 10, shadowOffset: { x: 0, y: 4 },
    stroke: '#e2e8f0', strokeWidth: 1,
    name: 'card-bg',
})

// 이미지 그룹
const contentGroup = new Konva.Group({
    x: PADDING, y: PADDING,
    width: CONTENT_W, height: CONTENT_H,
    name: 'content-group',
})

const konvaImage = new Konva.Image({
	image: imageObj, // HTMLImageElement를 사용하는게 안정적임
    name: 'content-image',
    width: imageObj.width,
    height: imageObj.height,
})
konvaImage.setAttr('originalIndex', originalIndex)

...

rootGroup.add(bg, contentGroup, seqText, ...) // 그룹으로 묶음
contentGroup.add(konvaImage) 
```
- Konva 객체는 name 속성으로 `card.findOne('.content-group')`을 사용해서 그룹이나 객체에 접근할 수 있음. 
- `setAttr(key, value)`: 데이터를 저장할 수 있음.
- `node.destroy()`: Konva 객체 삭제
- Konva 객체(Group, Rect, Image 등)은 기본적으로 JS 객체이기 때문에 `.`을 붙여서 메서드나 프로퍼티를 추가할 수 있음. 
- Konva 객체는 메모리 공간에 객체 정보를 만드는 것뿐이고 화면에 보여지기 위해서는 `layer.draw()` 또는 `layer.batchDraw()`가 호출되어야 함.

***Konva에서 이미지를 다룰 때***
- HTMLImageElement: 사용하는게 호환성과 안정성 면에서 가장 안전한 선택 (수십~수백 개)
- ImageBitmap: 성능이 극도로 중요한 경우에 메모리 효율이나 메인 스레드 점유율 면에서 유리 (수천 개)

``` js
/**
 * image 태그에 url을 적용하여 HTMLImageElement를 반환
 */
function createHTMLImageElementFromUrl(url) {
  return new Promise((resolve) => {
    const img = new Image() // 이미지 객체를 메모리에 생성
    img.onload = () => { resolve(img) } // (예약) 로딩 완료 시 HTMLImageElement 그대로 반환
    img.onerror = (err) => { resolve(null) }
    img.src = url // 이미지 객체에 이미지를 다운로드
  })
}
```

##### 3. 렌더링 (그리기)
- 메모리에 만든 객체들을 실제 화면(Canvas)에 출력
- `batchDraw()`: 다음 화면 갱신 주기에 맞춰 효율적으로 그립니다. (가장 권장됨)
- `draw()`: 즉시 그립니다.

``` js
function refreshLayout(draggingNode = null, targetIndex = -1, options = {}) {
	// 중앙 정렬을 위한 동적 시작 X 좌표 계산
	
	// 드래그 중일 경우 시각적 피드백을 위한 임시 순서 계산
	
	// 드래그 중인 본인 노드는 애니메이션하지 않음 (마우스 따라가야 함)
	
	// 등등 화면을 새로 그릴때 필요한 로직들을 같이 넣어둠
	
	layer.batchDraw()
}
```

##### 4. 이벤트 및 애니메이션 처리 

``` js
// 1. Stage 전체 이벤트 설정
stage.on('click tap', (e) => { ... })
stage.on('mouseover', (e) => { ... }) 
stage.on('wheel', (e) => { ... })
...

// 2. 각각의 노드 대한 이벤트 설정
rootGroup.on('mouseenter', () => { ... })
rootGroup.on('mouseleave', () => {
	node.to({ scaleX: 1.1, duration: 0.2 }); // 애니메이션 설정
});
...
```

- ***참고***: 
	Konav의 Stage는 브라우저의 일반적인 HTML 요소처럼 자동으로 스크롤을 지원하지 않음 그냥 그림(Canvas)임. 
	그래서 wheel 이벤트를 추가해서 스크롤이 가능하게 하고  `e.evt.preventDefault()`로 기존 스크롤 이벤트는 막아야함. (안 그러면 브라우저 전체 스크롤도 같이 됨.)

##### 5. 성능 최적화 

1. 캐싱: `node.cache()`를 사용하면 보잡한 노드를 하나의 정적 이미지로 변환하여 렌더링 속도를 획기적으로 높임
``` js
function createCardNode(imageObj, originalIndex = 0) {
	
	...
	
	// 1. 캐싱 처리 로직 추가
	card.executeCustomCaching = () => {
	    if (!isKonvaFontReady.value) return // 이미지가 완전히 배치된 후 캐시하기 위해 렌더링 직후에 실행
	    // 카드 전체 크기만큼 캐시 (그림자 등이 잘리지 않게 여유를 주려면 getClientRect 사용)
		card.cache({ 
			pixelRatio: window.devicePixelRatio || 1 // 사용하는 디바이스의 해상도에 맞춰 캐싱함.
		});
	};
	
	// 2. 카드가 생성된 직후 캐싱 처리
	// 만약 이미지가 아직 안 그려진 경우를 대비해 setTimeout으로 약간의 유예를 줌.
	setTimeout(() => {
		card.executeCustomCaching();
	    card.getLayer()?.batchDraw();
	}, 50);

  return card
}
```
- 카드를 캐시로 굽고 나면 화면에 '이제 라이브 노드가 아니라 구워진 이미지를 보여줘'라고 다시 한번 알려줘야함. 그래서 캐싱 이후에 `batchDraw()`가 호출
- Konva에서 이미지를 포함한 복잡한 그룹을 캐싱할때 노드들이 캔버스에 완전히 배치되고 렌더링될 준비가 끝난 상태여야 깨지지 않고 깨끗하게 구워짐. 그래서 `setTimeout()`으로 `refreshLayout()`이 실행되어서 좌표가 잡힌 후에 캐싱이 되도록 함. 

2. rAF Throttling: `requestAnimationFrame()`를 사용하여 마우스 이동 횟수보다 실행 횟수를 줄여 과부하를 막음.
	``` js
/**
 * 마우스가 움직일때마다 호출되는 dragmove가 호출하는 메서드 
 */
const scheduleDragLayoutUpdate = () => {
    if (!pendingDragCard) return;
    if (dragRafId !== null) return; // 이미 예약된 작업이 있으면 함수를 종료해서 불필요한 연산 차단

	// requestAnimationFrame은 다음 화면 갱신 직전(1/60초 간격)에 실행되도록 예약하는 함수, 작업 고유 번호를 반환
    dragRafId = requestAnimationFrame(() => {
      dragRafId = null; // 작업이 시작되었고 다음 예약을 받을 수 있는 상태로 바꿈.
      if (!pendingDragCard) return;

	  // 드래그 중인 카드의 위치 계산
      const newIndex = getTargetIndexFromCardPosition(pendingDragCard); 
      if (currentTargetIndex !== newIndex) {
        currentTargetIndex = newIndex;
        // 드래그 중에는 애니메이션 반영
        refreshLayout(pendingDragCard, newIndex, { animate: true, recacheChanged: false });
      }
    });
  };
	```
	- 작동 방식 
		1. dragmove 이벤트는 마우스가 드래그 될때마다 호출됨. 
		2. dragmove 이벤트 발생 -> `scheduleDragLayoutUpdate` 호출
		3. `requestAnimationFrame`를 호출하고 콜백함수를 예약, 작업 고유 번호를 `dragRafId`에 저장
		4. 이때 동안 발생하는 dragmove 이벤트는 무시됨. (`dragRafId` !== null)
		5. 콜백함수가 실행되면 `drageRafId`를 null로 바꾸고 드래그 중인 카드의 위치를 계산함. 
			- `requestAnimationFrame`는 화면 갱신에 맞춘 주기에 따라 실행됨 (60프레임, 1/60초)
		6. 2번부터 반복.

##### 6. 기타

1. Konva.Transformer: 사용자가 화면의 요소를 마수으로 잡고 크기를 조절하고 회전할 수 있는 UI 도구
	``` js
	transformer = new Konva.Transformer({
		keepRatio: false, // 비율 유지 X
		rotateEnabled: true, // 회전 기능 O 
		anchorFill: '#3b82f6',
		anchorStroke: '#fff',
		anchorSize: 10,
		borderStroke: '#3b82f6',
		padding: 2,
	});
	layer.add(transformer);
	```

2. 캐싱을 사용할 경우 폰트가 불러와지기 전에 캐싱이 되는 현상
	- 폰트를 가져온 후에 노드들을 생성 및 캐싱하도록 해야함.
	- 폰트를 가져온 후에 노드들을 재캐싱하도록 해야함. 


### 동영상 처리 

- *파일(MP4) -> [디먹서] -> 압축 데이터 -> [디코더] -> 실제 화면(프레임)*
1. 디먹서: MP4, MOV 같은 동영상 파일을 읽어서, 그 안에 담긴 '압축된 영상 데이터'만 골라내는 역할
2. 디코더: 디먹서가 넘겨준 압축된 데이터를 받아서 우리가 실제로 볼 수 있는 '원래의 이미지'로 변환하는 역할

##### 디먹서 (web-demuxer)

0. 설정
	- 패키지 설치: `npm install web-demuxer`
	- `vite.config.js` 설정
``` js 
// 1. COOP/COEP 헤더 설정
headers: {
	'Cross-Origin-Embedder-Policy': 'require-corp',
	'Cross-Origin-Opener-Policy': 'same-origin',
}

// 2. web-demuxer를 사전 빌드 대상에 명시적으로 포함
optimizeDeps: { include: ['web-demuxer'] }

// 3. 자산 인라이닝 한계를 0으로 설정하여 모든 파일을 별도 파일로 분리
build: {
    assetsInlineLimit: 0, 
  },
```
- `web-demuxer`는 내부적으로 FFmpeg(WASM)을 사용하며, 성능을 위해 멀티스레딩을 사용함. 웹에서 멀티스레딩의 핵심인 `SharedArrayBuffer`라는 메모리 공유 기능을 쓰려면 브라우저 보안 정책상 이 헤더들이 **반드시** 있어야 함.
- Vite는 의존성 패키지들을 미리 빌드하는데 `web-demuxer`처럼 복잡한 ESM 구조를 가진 패키지는 간혹 오류가 날 때가 있어서 명시적으로 해주면 안정적이게 로드가 됨. 
- `assetsInlineLimit: 0`: Vite는 작은 파일(예: 작은 로고 이미지)을 JS 파일 안에 Base64 문자열 형태로 직접 포함시키려 함. 하지만 WASM 파일은 JS 안에 글자로 포함되면 안 되고, 별도의 파일로 존재해야 브라우저가 이를 다운로드하면서 동시에 컴파일할 수 있음.

1. 디먹서(WebDemuxer) 객체 생성
``` js
import { WebDemuxer } from 'web-demuxer'
import wasmUrl from 'web-demuxer/wasm?url'

/**
 * 인스턴스를 하나만 유지하며 비디오 파일을 로드하는 헬퍼 함수입니다.
 */
async function getDemuxer(videoFile) {
  // 1. 디먹서 객체 생성
  if (!demuxer) {
    // WebDemuxer는 내부적으로 워커를 사용하므로, 사파리 호환성을 위해 메인 스레드에서 생성합니다.
    demuxer = new WebDemuxer({
      wasmFilePath: new URL(wasmUrl, import.meta.url).href
    });
  }

  // 2. 비디오 파일을 로드합니다. 이미 로드된 파일이면 내부적으로 최적화되어 처리됩니다.
  // 사용자가 선택한 비디오 파일(videoFile)을 디먹서가 분석하도록 합니다.
  await demuxer.load(videoFile);

  return demuxer;
}
```

2. 사용 
``` js
export async function scanScene({ videoFile, startTime, endTime }) {
  
  // 1. 디먹서 인스턴스 준비 및 비디오 로드
  // (getDemuxer 내부에서 new WebDemuxer() 생성 및 demuxer.load(videoFile) 수행)
  const currentDemuxer = await getDemuxer(videoFile);

  // 2. 비디오 스트림 정보 가져오기
  // 디코더 설정(Codec 정보 등)과 스트림 메타데이터(회전 정보 등)를 확보합니다.
  const decoderConfig = await currentDemuxer.getDecoderConfig('video');
  const videoStream = await currentDemuxer.getMediaStream('video');
  const rotation = videoStream?.rotation || 0;

  try {
    // 3. [핵심] 디먹싱 시작 (스트림 생성)
    // 지정된 구간(startTime ~ endTime)의 영상 데이터를 읽어올 준비를 합니다.
    const stream = currentDemuxer.read('video', startTime, endTime);
    const reader = stream.getReader();

    // 4. [핵심] 반복문을 통해 데이터 조각(Chunk)을 하나씩 읽기
    while (true) {
      // 디먹서로부터 다음 조각을 가져옵니다.
      const { done, value: chunk } = await reader.read();
      
      // 더 이상 읽을 데이터가 없으면 루프 종료
      if (done) break;

		/**
		* 여기서 얻은 'chunk'가 바로 EncodedVideoChunk (압축된 영상 데이터)입니다. 
		* chunk를 디코더에 넣어서 사용
		* EncodedVideoChunk를 워커로 보내야할 경우 JS 객체가 아니므로 변환을 해서 보내야함. 
		*/
    }

  } catch (error) {
    // 예외 처리 
    await reader.cancel(); // reader 해제 (필수)
  }
}
```


##### 디코더 (WebCodecs API)

0. 설정: `vite.config.js`
``` js
build: {
	target: 'esnext',
}
```
- 디코더(`VideoDecoder`)와 같은 최신 Web API를 사용하기 위해서입니다. 또한 WASM을 로드할 때 쓰이는 최신 자바스크립트 문법(Top-level await 등)을 지원하도록 빌드 대상을 최신으로 맞추는 것

1. 디코더(VideoDecoder) 객체 생성 및 실행
```js 
state.decoder = new VideoDecoder({
	 /**
     * 1. output 콜백 설정 디코더가 압축된 데이터를 풀어서 'VideoFrame'을 만들 때마다 호출됨
     */
	output: (frame) => {
		try {
			// VideoFrame을 Canvas를 이용해서 작업
			const bufferCanvas = new OffscreenCanvas(bw, bh);
			const bufferCtx = bufferCanvas.getContext('2d');
			...
		} catch (e) {
			// 예외 처리
		} finally {
			// 사용이 끝난 frame은 반드시 close()를 해줘야 GPU/메모리에서 즉시 수거됨 (필수)
			frame.close(); 
		}
	}
)}
// 2. 디먹서에서 가져온 설정값(코덱, 프로필 등)을 디코더에 주입
state.decoder.configure(decoderConfig);

// 디코딩 시작: 디코딩 -> output 콜백 함수
state.decoder.decode(chunk);
```
- 디코더는 청크를 받자마자 처리하지 않고 어느 정도 모았다가 한꺼번에 처리하는 성질이 있음. (Buffering)
- 마지막에 `flush()`를 호출해줘야 마지막 프레임들이 누락되는 걸 방지할 수 있음.
- 디코더 사용 후에 `decoder.close()`도 해줘야함.


### Web Worker

- 작동 방식 
	1. 워커 지정: `import ... from './worker.js?worker'`로 워커 지정
	2. 호출
		- 메인: `worker.postMessage(데이터)` -----> 워커: `self.onmessage = (e) => { ... }`
	3. 응답
		- 메인: `worker.onmessage = (e) => { ... }` <----- 워커: `self.postMessage(결과)`


 0. 설정: `vite.config.js`
``` js
  worker: {
    format: 'es', // 워커를 현대의 ESM 방식(import/export)을 사용한다고 명시
    plugins: () => [] // 워커 전용 빌드 환경을 깨끗하게 유지함.
  }
```
- Vite는 기본적으로 워커를 빌드할 때 가장 안전한 방식(Classic Worker)으로 빌드하려고 함. 왜냐하면 워커는 메인 앱 안에 들어있는 코드가 아니라 독립된 별도의 파일로 빌드되기 때문.
- 과거의 워커는 import문을 사용하는 것을 지원하지 않았지만 현대의 워커는 import/export를 사용하는 ESM 방식을 지원함. 그래서 ESM 방식을 사용하도록 format을 es로 명시해줘야함.

1. 워커 생성 및 준비
``` js
// extractor.js

// ?worker 접미사를 사용해서 Vite에게 worker.js 파일은 일반 JS가 아닌 웹 워커로 사용하는 걸 알림
import ExtractorWorker from './worker.js?worker'

function getWorker() {
  if (!worker) {
    // Vite 환경에서 워커를 생성할 때 미리 가져온 ExtractorWorker 경로를 사용
    worker = new Worker(ExtractorWorker, { type: 'module' });
    
    // 워커로부터 메시지(진행 상황, 성공, 에러)를 받았을 때의 처리 로직입니다.
    worker.onmessage = (e) => {
      const { type, jobId, result, progress, error } = e.data; // 워커에서 보내온 데이터 구조분해 할당
      const job = pendingJobs.get(jobId); // 장부(pendingJobs)에서 해당 jobId에 해당하는 작업을 찾음.
      
      if (type === 'PROGRESS') {
        // 진행률 업데이트 메시지인 경우 저장해둔 onProgress 콜백을 호출
        if (job.onProgress) job.onProgress(progress);
      } else if (type === 'SUCCESS') {
        job.resolve(result); // 작업이 성공적으로 완료되면 Promise를 이행(resolve)
        pendingJobs.delete(jobId); // 완료된 작업은 장부에서 제거
      } else if (type === 'ERROR') {
        ...
	  }
	};
	
	// 워커 자체에서 복구 불가능한 에러(네트워크 단절, WASM 로드 실패 등)가 발생한 경우
    worker.onerror = (err) => { ... }
    
    return worker;
```

2. 명령 내리기 (메인)
``` js
// extractor.js
async scanScene(payload) {
  const worker = this.getWorker(); // 1. 워커 가져오기
  const jobId = generateId();     // 2. 이 작업의 이름표(ID) 만들기
  
  // 3. 나중에 워커에서 SUCCESS 등의 메시지가 오면 처리할 콜백들을 장부에 기록
  const resultPromise = new Promise((resolve, reject) => {
	  ...
	  pendingJobs.set(jobId, {
		  resolve: (val) => {...},
		  reject: (err) => {...}
	  });
	  ...
  }

  // 4. 명령 보내기
  worker.postMessage({
    type: 'START_SCAN', // 어떤 일을 할지 (명령)
    payload: payload,   // 일할 때 필요한 데이터 (비디오 정보 등)
    jobId: jobId        // 나중에 답장 받을 때 구분할 ID
  });
  
  // 나중에 성공 로직에서 pendingJob.resolve(result)를 호출하면 result에 응답데이터를 받을 수 있음.
  const result = await resultPromise;
  return result;
}
```
- `worker.poseMessage()`는 워커에 작업을 추가하고 바로 반환됨. 

3. 명령 받기 (워커)
``` js
// worker.js

// self는 워커 자신을 의미함.
self.onmessage = async (e) => {
  // e.data 안에 extractor.js에서 보낸 데이터가 다 들어있음.
  const { type, payload, jobId } = e.data;

  try {
    switch (type) {
      case 'START_SCAN':
        await startScan(jobId, payload); // 비동기 
        break;
      case 'PUSH_CHUNK':
        pushChunk(jobId, payload); // 동기
        break;
      // ... 다른 명령들
    }
  } catch (error) {
    // 에러가 나면 다시 extractor.js로 메세지를 보내서 에러처리를 하도록 함.
    self.postMessage({ 
      type: 'ERROR', 
      jobId, 
      error: error instanceof Error ? error.message : String(error) 
    });
  }
};
```
- 기본적으로 워커는 비동기 단일 스레드 + FIFO 방식으로 동작
- `await`가 없는 경우 (동기식): 해당 작업이 완료되기 전에는 다음 작업을 하지 않음. 
- `await`가 있는 경우 (비동기식): 해당 작업 중에 CPU가 비게 되면 다음 작업을 할 수 있음.

4. 작업 결과 보고 (워커 -> 메인)
``` js
// worker.js 
// 작업을 완료한 후에 extractor.js로 작업결과와 함께 메세지를 보내서 완료 처리를 하도록 함. 
self.postMessage({
  type: 'SCAN_COMPLETED', // 작업 완료 알림
  jobId: jobId,           // 아까 받은 그 ID
  payload: resultData     // 스캔 결과물
});
```


### 작업 취소 abort

1. 작업 요청을 할때 `new AbortController()` 객체를 만든 후 `.signal`로 시그널 객체를 만든 후 같이 넘김
2. 시그널 객체에서 `'abort'` 이벤트를 등록함. 
``` js
if (signal) {
	// 이미 중단된 상태라면 즉시 중단 처리합니다.
    if (signal.aborted) return abortHandler(); 
    // 중단 이벤트가 발생하면 핸들러를 실행하도록 등록합니다.
    signal.addEventListener('abort', abortHandler, { once: true });
}

const abortHandler = () => { 
	// 작업이 취소되었을때 처리할 로직
}
```
3. 작업 취소를 해야할 때 1번에서 만들어둔 AbortController 객체의 `abort()` 실행



### Tone.js

웹 브라우저에서 상호작용하는 오디오와 음악을 쉽게 생성하고 제어할 수 있는 JS 오디오 라이브러리
브라우저가 기본적으로 제공하는 Web Audio API를 기반으로 구축되어있음. 

1. Transport: 음악의 진행을 관리하는 글로벌 타임라인이자 관리자
	- 시간을 초 단위가 아니라, 음악적 시간 단위인 BPM, Ticks 등의 단위로도 다룰 수 있게 해줌. 
	- `Transport.start()`, `Transport.stop()`, `Transport.pause()`를 통해 음악 전체의 재생 상태를 제어

2. Synth: 소리를 만들어내는 음원 소스
	- `Tone.Synth`: 기본적인 모노포닉(단음) 신디사이저로, 기계음을 만들 수 있음.
	- `Tone.PolySynth`: 여러 화음을 동시에 소리 낼 수 있는 다성 신디사이저
	- `Tone.Sampler`: 오디오 샘플 파일(mp3, wav 등)을 읽어와서 재생하는 악기, 피아노나 드럼 소리 등을 브라우저에서 실제 악기처럼 구현할 때 사용

	- Tone.Synth의 `triggerAttackRelease(note, duration, time, velocity)` 메서드로 소리를 낼 수 있음. 
		- `note`: 연주할 음의 높낮이(주파수) 지정, 예: C4, Gb5, 440(Hz) 등
		- `duration`: 음의 지속 시간, 예: 4n(4분음표), 8n, 0.5(초)
		- `time`: 소리가 시작되어야 할 **오디오 하드웨어 기준 절대 시간**, 보통 Loop, Part 등의 콜백 함수 안에서 인자로 전달받은 time을 그대로 넣어서 사용함.
		- `velocity`: 강도 혹은 볼륨 (0~1)

3. 스케줄링: Web Audio API를 사용하여 오디오 이벤트를 예약함. 
	- `Tone.Loop`: 지정된 고정 시간 간격(인터벌)으로 콜백 함수를 무한히 반복 실행 (정확한 타이밍을 위해서 예약을 해둠.)
		``` ts
	   this.loop = new Tone.Loop((time) => {
        // a. 소리 재생
        const totalTicks = this.getTotalTicksPerBar();
        const currentTick = this.beatCount % totalTicks;
        const beatState = this.currentBeatStates[currentTick] ?? (currentTick === 0 ? 2 : 1);
        
        this.triggerAcousticSound(time, beatState);

        // b. UI 업데이트 예약
        Tone.getDraw().schedule(() => {
          this.onBeat(currentTick);
          this.beatCount++;
        }, time);

      }, `${this.currentNoteValue}n`); // 루프 간격을 동적으로 설정
		```
		
	- `Tone.Part`: 각 이벤트가 실행될 정확한 시간 위치를 지정해두고 재생
		``` ts
		const events = expandedBeatsArray.map((time, index) => {
        const correctedTime = Math.max(0, time - 0.133 + syncOffset);
        return {
	       // 초(Seconds) 단위의 소수점 시간을 Tone.js의 가장 정밀한 시간 단위인 Ticks(틱)으로 변환
          time: Math.round(correctedTime * 960) + "i", 
          beatIndex: index
        };
      });

		// value에는 events 배열 안의 원소 객체 자체가 들어옴.
      this.part = new Tone.Part((time, value) => {
        const totalTicks = this.getTotalTicksPerBar();
        const currentTick = value.beatIndex % totalTicks;
        const beatState = this.currentBeatStates[currentTick] ?? (currentTick === 0 ? 2 : 1);
        
        this.triggerAcousticSound(time, beatState);

        Tone.getDraw().schedule(() => {
          this.onBeat(currentTick);
        }, time);
      }, events); // events에 배열에 담긴 각각의 시간(time)에 도달할때마다 콜백함수 실행
		```


**예시 코드**

``` ts
public async start(beatsArray?: number[] , syncOffset: number = 0, videoStartTime: number = 0) {
    // 1. 오디오 컨텍스트를 시작합니다. (사용자 클릭 이후에 호출해야 에러가 안 납니다), 이미 켜져있으면 스킵
    await this.startAudioEngine();
    
    // 2. 만약 신스(소리를 내는 악기)가 없다면 현재 선택된 사운드 타입으로 생성합니다.
    if (!this.bodySynth) {
      this.setSoundType(this.currentSoundType);
    }

    // 3. 기존 루프 지우기 (기본 메트로놈용)
    if (this.loop) {
      this.loop.dispose();
      this.loop = null;
    }
    // 동기화 모드일 때는 part를 파괴하지 않고 재사용 여부를 판단합니다.
    this.beatCount = 0; // 카운트 초기화

    // A. 기본: 메트로놈만 실행한 경우
    if (!beatsArray || beatsArray.length === 0) {
      console.log("기본 메트로놈 실행");
      // 기존 파트가 있으면 지움 (안 지우면 Part랑 Loop랑 같이 실행됨.)
      if (this.part) {
        this.part.dispose();
        this.part = null;
        this.currentSyncOffset = null;
        this.currentBeatsArrayStr = "";
      }
      // 현재 BPM 적용
      Tone.getTransport().bpm.value = this.currentBpm * this.currentPlaybackRate;

      // 5. 지정된 간격(interval)으로 계속 실행되는 루프를 만듭니다.
      this.loop = new Tone.Loop((time) => {
        // a. 소리 재생
        const totalTicks = this.getTotalTicksPerBar();
        const currentTick = this.beatCount % totalTicks;
        const beatState = this.currentBeatStates[currentTick] ?? (currentTick === 0 ? 2 : 1);
        
        this.triggerAcousticSound(time, beatState);

        // b. UI 업데이트 예약
        Tone.getDraw().schedule(() => {
          this.onBeat(currentTick);
          this.beatCount++;
        }, time);

      }, `${this.currentNoteValue}n`); // 루프 간격을 동적으로 설정

      // 6. 루프를 0초(즉시)부터 시작하도록 등록하고, 전체 오디오 엔진(Transport)을 시작합니다.
      this.loop.start(0);
      Tone.getTransport().start();
    } else { ... 
```

- `Tone.getDraw().schedule()`: 정확히 박자가 울리는 타이밍에 `onBeat()`를 호출해줌. 
- `onBeat()`는 오디오 엔진 실행 시에 화면을 업데이트하는 함수로 지정해둠.


### SSE 서버 전송 이벤트 

SSE (Server-Sent Events) 서버 전송 이벤트는 **서버 푸시 기술**, **실시간 단방향 이벤트 스트리밍**이라고 부름. 

1. 최초 연결: 클라이언트(나)가 서버에 `new EventSource(sseUrl)`를 호출하여 실시간 스트림을 열어달라는 요청을 보냄. 
2. 연결 유지 및 실시간 전송: 서버는 이 요청을 수락하고 HTTP 연결을 끊지 않은 채 열어둠. 서버가 작업 진행 상황을 클라이언트에게 실시간으로 데이터를 밀어넣어 줍니다. 

``` ts
export function waitForDownloadComplete(
    sseUrl: string, 
    onProgress: (progress: number) => void
): Promise<string> {
    return new Promise((resolve, reject) => {
		// 1. 최초 연결
        const eventSource = new EventSource(sseUrl);

		// 2. 연결 유지 및 실시간 전송 
        eventSource.addEventListener('progress', (e) => {
            const data = JSON.parse(e.data);
            onProgress(data.progress ?? 0); // 진행률(숫자)을 콜백으로 전달
        });

        eventSource.addEventListener('completed', (e) => {
            const data = JSON.parse(e.data);
            eventSource.close();
            // 완료되면 download_url을 resolve로 반환
            resolve(data.download_url); 
        });

        eventSource.onerror = () => {
            eventSource.close();
            reject(new Error("다운로드 스트림 오류가 발생했습니다."));
        };
    });
}
```


### 오디오 파일 처리

1. **오디오 파일 다운로드**: 다운받는 오디오 파일(MP3, M4A)는 용량을 줄이기 위해서 고도로 압축되어 있음. 사람이 들을 때는 오디오 플레이어가 알아서 실시간으로 압축을 풀어서 들려주지만, 컴퓨터(Essentia.js 등)가 수학적으로 분석하기에는 **알아 볼 수 없는 암호 덩어리**임.
2. **ArrayBuffer**: 다운로드한 데이터를 일단 자바스크립트가 만질 수 있는 순수한 바이트(Byte) 덩어리 상태인 `ArrayBuffer`로 메모리에 올려놓음. 
``` ts
	// 1. 오디오 파일 다운로드
    const response = await fetch(downloadUrl, {
        method: "GET",
    });

    // 2. 응답을 ArrayBuffer(순수 바이트 데이터)로 변환
    const arrayBuffer = await response.arrayBuffer();
```

3. **디코딩(decodeAudioData)**: 이제 브라우저의 오디오 엔진(`AudioContext`)을 시켜서 이 암호화된 바이트 덩러의 압축을 품. 압축을 풀면 나오는 것이 **PCM 데이터(Float32Array)**임.
4. **결과물 (Float32Array)**: 이 데이터는 1초에 44,100개의 숫자(소리의 파형, 진폭)들이 나열된 거대한 숫자 배열임. Essentia.js 등이 작업을 할 수 있는 상태임. 
``` ts
    const AudioContextClass = window.AudioContext || window.webkitAudioContext;
    const OfflineAudioContextClass =
        window.OfflineAudioContext || window.webkitOfflineAudioContext;

    // 3. 브라우저의 Web Audio API를 사용하여 오디오 데이터 디코딩
    const decodeCtx = new AudioContextClass();
    let decodedBuffer;

    try {
        // 여기서 압축 파일(MP3, M4A 등)의 압축을 풉니다.
        decodedBuffer = await decodeCtx.decodeAudioData(arrayBuffer);
    } catch (e) {
        console.error("오디오 디코딩 실패:", e);
        throw new Error("잘려진 오디오 파일의 구조가 손상되어 디코딩할 수 없습니다.");
    }
```


##### Float32Array

자바스크립트의 일반 배열(`[]` 또는 `Array`)은 아주 유연함. 숫자, 문자, 객체 등을 마구잡이로 섞어 넣을 수 있고 길이도 마음대로 늘어남. 하지만 이 유연성 때문에 컴퓨터 메모리 상에서는 꽤 무겁고 비효율적으로 동작

`Float32Array`는 **TypedArray(타입화된 배열)**의 한 종류
- 오직 32비트 실수(소수점이 있는 숫자)만 담을 수 있음.
- 메모리에 빈틈없이 꽉꽉 채워져서 아주 빠르게 계산할 수 있도록 설계
- 주로 **오디오 데이터(Web Audio API), 3D 그래픽(WebGL), 대용량 데이터 처리(Wasm 등)** 를 다룰 때 성능(속도)을 위해 강제로 사용하게 됨.

문제는 이 성능 좋은 `Float32Array`가 **데이터를 JSON으로 바꿔서 DB에 저장하는 통신 과정**에서는 표준 포맷으로 인정받지 못하기 때문에 강제로 `{0: x, 1: y}` 같은 객체 모양으로 찌그러져 버리는 부작용이 있음.


### Essentia.js

오디오 분석 및 음악 정보 추출을 하는 라이브러리 
C++ 기반의 음향 분석 라이브러리인 **Essentia를 웹 및 Node.js 환경에서 사용할 수 있도록 자바스크립트와 WASM(웹어셈블리)로 컴파일**한 것

- 리듬 및 템포: BPM 측정, 비트 트래킹(각 박자가 나오는 정확한 시간 구하기) 등 
- 음조 및 화음: 곡의 키(조성) 감지, 코드 인식
- 주파수 분석: 스펙트럼 분석, 피치(음높이) 검출 등

``` ts
export async function extractBpmAndBeat(audioData: Float32Array) {
    // 브라우저 전용 Essentia 빌드를 지연 로드해서 fs 의존성을 번들에서 제외한다.
    const { Essentia, EssentiaWASM } = await loadEssentia();
    const essentia = new Essentia(EssentiaWASM);

    // 1. 자바스크립트 배열을 Essentia 전용 Vector로 변환
    const audioVector = essentia.arrayToVector(audioData);

    // 2. 리듬 분석 알고리즘 실행 (RhythmExtractor2013): 
    // BPM, 박자가 떨어지는 시간(ticks), 신뢰도 등을 반환합니다.
    const result = essentia.RhythmExtractor2013(
        audioVector,
        250,             // maxTempo (최대 BPM 늘림)
        "multifeature",  // method (기본 방식 유지)
        40               // minTempo (기본 최저 BPM 유지)
    ) as RhythmExtractorResult;

    // 3. 결과값 추출 (소수점 둘째 자리까지 반올림)
    const extractedBpm = result.bpm;
    
    // 4. 첫 박자(First Beat) 추출
    // 박자 발생 시간(초 단위)들이 담긴 Vector를 다시 JS 배열로 변환
    const beatsArray = Array.from(essentia.vectorToArray(result.ticks)) as number[]; 

    return { extractedBpm, beatsArray };
}
```


1. **WASM 로드 (`loadEssentia`)**: 실시간으로 브라우저 환경에 맞게 `.wasm` 파일을 비동기로 다운로드하여 실행 준비함.
	- `loadEssentia()` 메서드나 .wasm 파일 등의 초기 세팅이 복잡하고 환경마다 다름.
2. **데이터 변환 (`arrayToVector`)**: Essentia 내부엔진(C++)이 이해할 수 있도록 브라우저의 오디오 데이터(`Float32Array`)를 Essentia 전용 `Vector` 포맷으로 변환
3. **알고리즘 실행 (`RhythmExtractor2013`)**: 변환된 벡터를 사용해 곡의 BPM과 정확한 비트 위치 리스트(`ticks`)를 받음.


### Pino 

- `pino`: JSON(`{"level":30,"time":...}`) 형식으로만 로그를 출력하는 JSON 로거 라이브러리
- `pino-pretty`: `pino`가 출력하는 JSON 로그를 가로채서, 사람이 읽기 좋게 색상을 입히고 시간 형식을 다듬어 주는 포맷터
	- 포맷팅하는 데 추가적인 연산이 발생하므로 **개발 환경에서만 사용**하고 운영 환경에서는 사용하지 않음.

1. 패키지 설치
``` bash
pnpm add pino
pnpm add -D pino-pretty
```

2. 설정
``` ts
// lib/logger.ts
// lib/logger.ts
import pino from 'pino';

// 브라우저 환경 여부 검사
const isBrowser = typeof window !== 'undefined';
// 환경 변수 검사 (Next.js 환경에서는 process.env.NODE_ENV를 사용)
const isProduction = process.env.NODE_ENV === 'production';

export const logger = pino({
  // 프로덕션 환경이면 info 이상의 로그만, 개발 환경이면 debug 이상의 로그까지 모두 출력
  level: isProduction ? 'info' : 'debug',
  
  // 브라우저(클라이언트) 환경에서 호출될 경우 pino가 에러를 내지 않고 일반 console로 동작하게끔 브라우저 설정을 켜줍니다.
  // (원칙적으로는 서버에서만 써야 하지만, 휴먼 에러 방지용)
  browser: {
    asObject: true,
  },
  
  // 개발 환경이면서 서버 환경일 때만 pino-pretty를 사용해 보기 좋게 포맷팅합니다.
  // 프로덕션이거나 브라우저 환경일 때는 이 설정이 무시되고 순수 JSON 또는 일반 콘솔로 출력됩니다.
  ...(isProduction || isBrowser
    ? {}
    : {
        transport: {
          target: 'pino-pretty',
          options: {
            colorize: true, // 색상 적용
            translateTime: 'SYS:yyyy-mm-dd HH:MM:ss.l', // 읽기 쉬운 시간 포맷
            ignore: 'pid,hostname', // 터미널이 지저분해지는 것을 막기 위해 pid, hostname은 숨김
          },
        },
      }),
});
```

3. 사용
``` ts
logger.info({ userId: newUserId, ip }, "새로운 유저 생성됨");
logger.error({ err: error, ip: ip}, "유저 생성(IP) 중 에러");

// JSON 로그 형태
{
  "level": 50,
  "time": 1717316972000,
  "pid": 12345,
  "hostname": "my-server",
  "userId": "user_abcd123",
  "ip": "123.123.123.123",
  "msg": "새로운 유저 생성됨"
}

```
- 로그에 함께 저장하고 싶은 데이터를 객체에 담아 첫번째 인자로 넘기면 JSON 로그 객체 내부로 병합시킴. 
- 첫 번째 인자에 에러(`Error`) 객체를 전달할 때 `err` 또는 `error`라는 키로 감싸서 전달하면 자동으로 에러의 스택 트레이스까지 JSON 내부에 깔끔하게 구조화해서 남겨줌.
- 두번째 인자로 요약 메세지를 넘기면 JSON 로그의 msg 필드로 들어감. 

