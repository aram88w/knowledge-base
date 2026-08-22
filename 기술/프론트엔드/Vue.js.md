### Pinia 

1. 패키지 설치
- `npm install pinia`: Vue 공식 상태 관리 라이브러리
- `npm install pinia-plugin-persistedstate`: 브라우저 새로고침을 해도 데이터가 유지되도록 localStorage 등에 저장해주는 플러그인

2. 설정
``` js
import { createApp } from 'vue'  
import App from './App.vue'
import { createPinia } from 'pinia'  
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const app = createApp(App)  
const pinia = createPinia()

pinia.use(piniaPluginPersistedstate)  // Pinia에 플러그인 적용
app.use(pinia) // Pinia 적용
app.mount('#app')
```

3. 사용
``` js
import { defineStore } from 'pinia'  
  
export const useCounterStore = defineStore('counter', () => {
  const count = ref(0) // state
  const doubleCount = computed(() => count.value * 2) // getters
  function increment() { count.value++ } // actions

  return { count, doubleCount, increment }
}, {
	// 세 번째 인자로 persist 설정을 넣어줍니다
	// 플러그인이 localStorage에 데이터를 JSON 형태로 저장
	persist: true, 
})
```

``` js
// 특정 데이터만 저장
persist: {
  key: 'my-custom-key', // 로컬스토리지에 저장될 이름 변경
  storage: sessionStorage, // 로컬스토리지 대신 세션스토리지 사용
  pick: ['count'], // state 중 'count'만 저장하고 나머지는 제외
},
```


### 다국어

1. 패키지 설치: `npm install vue-i18n`
2. 설정 
- `i18n.js`
``` js
import { createI18n } from 'vue-i18n';
import ko from './locales/ko.json';
import en from './locales/en.json';

const i18n = createI18n({
  legacy: false, // Composition API 사용 시 필수
  globalInjection: true, // 템플릿에서 $t 전역 사용을 위해 추가
  locale: 'en', // 초기 언어 설정
  fallbackLocale: 'en', // 설정된 언어의 번역이 없을 때 보여줄 기본 언어
  messages: { ko, en }
});

export default i18n;
```

- `main.js`
``` js
import { createApp } from 'vue'
import i18n from './i18n'
import App from './App.vue'

const app = createApp(App)
app.use(i18n)
app.mount('#app')
```

- `src/locales/ko.json`
``` json
{
  "home": {
    "features": {
      "interval": { "desc": "간격 설정 설명입니다." }
    }
  }
}
```

3. 사용 
``` vue
<script setup>
import { useI18n } from 'vue-i18n'

// 1. 함수 가져오기
const { t, locale } = useI18n()
// 2-1. 사용
const string = t('home.features.interval.desc')
// 2-2 언어 변경도 가능
const changeLang = () => {
  locale.value = 'en'
}
</script>

<template>
  <!-- 2-3. 사용 -->
  <p>{{ $t('home.features.interval.desc') }}</p>
</template>
```


### 다국어 라우팅

- URL 경로에 언어 정보를 포함하여 구글 검색 시 해당 언어로 검색이 되도록 함. 
- `index.js`
``` js
import { createRouter, createWebHistory, RouterView } from 'vue-router'
import i18n, { supportedLocales, getInitialLocale } from '@/i18n'
import { h } from 'vue'

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      // 2번 규칙: :lang 파라미터가 있는 주요 라우트들
      path: '/:lang',
      component: { render: () => h(RouterView) },
      children: [
        {
          path: '',
          name: 'home',
          component: HomeView
        },
        {
          path: 'extraction',
          name: 'extraction',
          component: ExtractionView,
          meta: { hideFooter: true } // 푸터를 감춤
        },
      ]
    },
    {
      // 3번 규칙: 정의되지 않은 경로 매칭 시 리다이렉트 방어 로직
      path: '/:pathMatch(.*)*',
      redirect: (to) => {
        // 이미 지원하는 언어 코드(첫 번째 세그먼트)가 포함되어 있다면 해당 언어의 홈으로 리다이렉트 (언어 튕김 방지)
        const possibleLang = to.path.split('/')[1];
        if (possibleLang && supportedLocales.includes(possibleLang)) {
          return `/${possibleLang}`;
        }
        // 아예 모르는 경로거나 언어 코드가 없는 경우에는 기본 언어 경로로 리다이렉트
        return `/${getInitialLocale()}`;
      }
    }
  ]
})

// 1번 규칙: 네비게이션 가드를 통한 경로 유실 방지 (Prepend 로직)
router.beforeEach((to, from, next) => {
  // /lang/xxx
  const pathParts = to.path.split('/');
  const lang = pathParts[1];

  // URL의 첫 번째 부분이 지원하는 언어 코드가 아닌 경우 (루트 진입 포함)
  // 예: /extraction 라면 extraction은 지원 언어가 아니게 됨
  if (!supportedLocales.includes(lang)) {
    const correctLang = getInitialLocale(); // 적절한 언어를 가져옴 (예: 브라우저 기본 언어)
    
    // to.fullPath가 '/' 인 경우에만 뒤에 슬래시가 안 붙게 빈 문자열로 넘김
    const cleanPath = to.fullPath === '/' ? '' : to.fullPath;
    // 루트 진입이면 next('/ko'), 서브페이지 진입이면 next('/ko/extraction') 으로 깔끔하게 떨어집니다.
    return next(`/${correctLang}${cleanPath}`);
  }

  // URL 파라미터(언어)가 변경될 때마다 100% 무조건 실행되도록
  i18n.global.locale.value = lang;

  // 정상적인 언어 코드가 들어있는 경로는 그대로 통과
  next();
});

export default router
```


### SEO 설정 

- `index.html`
``` html
<!DOCTYPE html>
<!-- 이 웹 페이지의 주된 언어 설정 -->
<html lang="en"> 
  <head>
	<!-- 문서의 문자 인코딩 방식 설정 -->
    <meta charset="UTF-8"> 
	<!-- 브라우저 탭에 표시되는 작은 아이콘인 파비콘(Favicon)의 경로를 지정 -->
    <link rel="icon" href="/favicon.ico">
    <!-- 반응형 웹 디자인의 핵심 설정 -->
    <!-- 기기의 가로 너비에 맞춰 페이지 너비를 조절, 초기 확대 비율을 1대1로 고정-->
    <!-- 반응형 웹 디자인의 핵심 설정 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <!-- 브라우저 탭의 제목이자, 구글 등 검색 결과에서 클릭하게 되는 큰 제목 -->
    <title>Video Image Extractor - Extraction and Editing in One</title>
    <!-- 검색 결과 제목 아래에 표시되는 요약 설명 -->
    <meta name="description" content="Easily extract high-quality images from videos and ...">
    
```

``` html
    <!-- SEO: 다국어 검색 최적화 (Hreflang) -->
    <link rel="alternate" hreflang="ko" href="https://extractframe.org/ko" />
    <link rel="alternate" hreflang="en" href="https://extractframe.org/en" />
    <link rel="alternate" hreflang="x-default" href="https://extractframe.org/en" />
    <link rel="canonical" id="canonical-link" href="" />
```

- `router/index.js`
``` js
// SEO: 라우팅이 완료된 후, 현재 페이지 주소를 canonical 링크로 설정합니다.
router.afterEach((to) => {
  const canonicalElement = document.getElementById('canonical-link');
  if (canonicalElement) {
    const domain = 'https://extractframe.org';
    // to.path는 항상 '/'로 시작하므로, 루트('/')일 때와 아닐 때를 구분하지 않아도 결과는 같습니다.
    // 만약 도메인 끝에 /를 안 붙이셨다면 아래 방식이 가장 깔끔합니다.
    const finalUrl = domain + (to.path === '/' ? '' : to.path);
    canonicalElement.setAttribute('href', finalUrl);
  }
});
```


### 폰트 설정

1. 폰트 다운
	- `index.html`에서 `link` 태그 사용 
		- 예: <link href='//spoqa.github.io/spoqa-han-sans/css/SpoqaHanSansNeo.css' rel='stylesheet' type='text/css'>
	- npm 사용
		1. 패키지 다운로드: `npm install spoqa-han-sans`
		2. 프로젝트에 불러오기: `main.js`
``` js
// src/main.js
import { createApp } from 'vue'
import App from './App.vue'

// 폰트 CSS 불러오기 (경로 주의!)
import 'spoqa-han-sans/css/SpoqaHanSansNeo.css'

const app = createApp(App)
app.mount('#app')
```

2. 폰트 적용 (전역)
- `main.css`
``` css
:root {
	...
	--font-main: "Manrope Variable", "Spoqa Han Sans Neo", "Noto Sans JP", -apple-system, sans-serif;
}

/* Tailwind CSS 환경에서 전역 스타일을 정의할 때 @layer base 사용 */
@layer base { 
	body {
    font-family: var(--font-main);
    ...
    }

	/* 브라우저 기본 스타일상 button, input 등은 body의 font-family를 자동으로 상속받지 않은 상황 방지 */
	button,input,select,textarea {
	    font: inherit;
	}
}
```


