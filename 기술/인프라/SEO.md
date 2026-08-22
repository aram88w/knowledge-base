### robots.txt

`robots.txt` 파일은 검색 엔진 크롤러에게 크롤러가 요청할 수 있는 페이지나 파일, 요청할 수 없는 페이지나 파일을 알려줍니다.

``` txt
// public/robots.txt
 
# Block all crawlers for /accounts
User-agent: *
Disallow: /accounts
 
# Allow all crawlers
User-agent: *
Allow: /
```


### XML 사이트맵

구글이나 네이버 같은 검색 엔진의 크롤러(봇)가 웹사이트의 모든 페이지를 빠짐없이, 그리고 효율적으로 찾을 수 있도록 돕는 역할을 합니다. 특히 다음과 같은 경우에 효과적입니다:

- **사이트 규모가 클 때:** 수백, 수만 개의 페이지가 있는 경우 크롤러가 링크만 타고 모든 페이지를 찾기 어렵습니다.
- **새로운 페이지가 추가되었을 때:** 사이트맵을 통해 검색 엔진에 새 콘텐츠의 존재를 빠르게 알릴 수 있습니다.
- **페이지 간 연결(Internal Linking)이 복잡할 때:** 고립된 페이지도 크롤러가 발견할 수 있게 해줍니다.

``` js
// app/sitemap.ts
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://example.com', // 페이지의 절대 경로
      lastModified: new Date(), // 해당 페이지가 마지막으로 수정된 날짜, 보통 new Date() 사용
      changeFrequency: 'yearly', // 페이지 내용이 얼마나 자주 바뀌는지에 대한 힌트
      priority: 1, // 전체 사이트 내에서 해당 페이지의 상대적 중요도 
    },
    {
      url: 'https://example.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    // 블로그 포스트 등을 동적으로 추가 가능
  ]
}
```
- **`sitemap.ts` (또는 `.js`):** `app` 디렉토리 안에 이 파일을 만들면 Next.js가 자동으로 `/sitemap.xml` 경로를 생성해 줍니다.
- **정적/동적 생성:** 고정된 URL뿐만 아니라, 데이터베이스에서 글 목록을 가져와 실시간으로 사이트맵을 동적으로 생성할 수도 있습니다.
- **자동 관리:** 개발자가 XML 코드를 직접 작성할 필요 없이, 객체 배열만 반환하면 Next.js가 표준 XML 포맷으로 변환해 줍니다.

- **`url`**: 페이지의 절대 경로 URL (예: `https://ytmetronome.com/ko`)
- **`lastModified`**: 해당 페이지가 마지막으로 수정된 날짜 (보통 `new Date()`를 사용)
- **`changeFrequency`**: 페이지 내용이 얼마나 자주 바뀌는지에 대한 힌트 (`daily`, `weekly`, `monthly` 등)
- **`priority`**: 전체 사이트 내에서 해당 페이지의 상대적 중요도 (`0.0` ~ `1.0`)


##### 다국어 지원 사이트맵 작성

- `[locale]` 동적 라우팅 방식을 사용한 경우

``` ts
// sitemap.ts
import { MetadataRoute } from 'next'

// 지원하는 언어 목록 정의
const locales = ['ko', 'en', 'de', 'es', 'fr', 'ja', 'pt', 'zh']
const BASE_URL = 'https://ytmetronome.com'

export default function sitemap(): MetadataRoute.Sitemap {
  // 1. 기본 루트 경로 (https://ytmetronome.com)
  const rootSitemap = {
    url: BASE_URL,
    lastModified: new Date(),
    changeFrequency: 'weekly' as const,
    priority: 1.0,
  }

  // 2. 각 언어별 경로 (https://ytmetronome.com/ko, /en 등)
  const localeSitemaps = locales.map((locale) => ({
    url: `${BASE_URL}/${locale}`,
    lastModified: new Date(),
    changeFrequency: 'weekly' as const,
    // 메인 루트보다는 우선순위를 조금 낮게 설정 (예: 0.8)
    priority: 0.8, 
  }))

  return [rootSitemap, ...localeSitemaps]
}

```


### 캐노니컬 태그

URL은 다르지만 내용(컨텐츠)이 똑같은 경우가 있음. 
1. 검색/필터링 기능
	- `example.com/shoes` (전체 신발 목록)
	- `example.com/shoes?color=red` (빨간 신발만 필터링)
	- 내용은 거의 비슷하거나 같은 '신발 목록'인데, 뒤에 붙은 파라미터(`?color=red`) 때문에 주소는 완전히 다르게 인식됩니다.
2. 여러 카테고리에 속한 제품
	- `example.com/electronics/phone` (전자제품 카테고리 기점)
	- `example.com/mobile/phone` (모바일 카테고리 기점)
3. 주소 표기 방식의 차이
	- `http://www.site.com`
	- `https://site.com` (S 유무, WWW 유무)


SEO 측면에서의 문제점
1. **점수 분산:** 100점짜리 좋은 페이지인데, 주소가 2개로 갈리면 검색 엔진이 점수를 50점, 50점씩 나눠줄 수 있습니다. 그러면 검색 결과 상단에 오르기 힘들어집니다.
2. **크롤링 낭비:** 검색 로봇이 내 사이트를 훑어볼 수 있는 시간은 정해져 있는데, 똑같은 내용을 주소만 바꿔가며 수십 번 읽게 되면 정작 중요한 새 글은 못 읽고 지나칠 수 있습니다.
3. **검색 결과 혼선:** 내가 의도한 주소는 A인데, 구글은 마음대로 B 주소를 검색 결과에 노출할 수도 있습니다.


캐노니컬 태그 활용
``` tsx
      <Head>
        <title>Canonical Tag Example</title>
        <link
          rel="canonical"
          href="https://example.com/products/phone"
          key="canonical"
        />
      </Head>
```
- 현재 페이지의 주소가 `example.com/mobile/phone`이지만, 진짜 원본을 명시해줌으로써  `https://example.com/products/phone`로 점수를 다 몰아줄 수 있음.


### hreflang 태그

구글 같은 검색 엔진은 다국어 웹사이트를 크롤링할 때 **이 페이지의 다른 언어 버전들이 어떤 주소로 존재하는지**를 한눈에 파악하기를 원함. 
구글 가이드라인에 따르면, 한국어 페이지(`/ko`)에 들어갔을 때도 영어(`/en`), 일본어(`/ja`) 등 지원하는 모든 언어의 주소를 메타데이터로 제공해 주어야 함. 
Next.js에서는 이를 위해 `metadata.alternates.languages` 객체를 제공하며, 이 객체를 반환하면 Next.js가 HTML 헤더에 아래와 같이 지원하는 **모든 언어의 대체 주소 태그들을 일괄 생성**해줌. 

``` html
<!-- ko 페이지에 접속해도 아래 목록이 다 출력되어야 검색 엔진이 언어별 관계를 이해합니다 -->
<!-- 다국어 대체 주소 제공 -->
<link rel="alternate" hreflang="ko" href="https://ytmetronome.com/ko/tempo/60-bpm-metronome" />
<link rel="alternate" hreflang="en" href="https://ytmetronome.com/en/tempo/60-bpm-metronome" />
<link rel="alternate" hreflang="ja" href="https://ytmetronome.com/ja/tempo/60-bpm-metronome" />
<!-- ...기타 언어 생략... -->
<!-- 지원하지 않는 다른 언어권 사용자가 들어오면 영어 페이지로 안내 -->
<link rel="alternate" hreflang="x-default" href="https://ytmetronome.com/en/tempo/60-bpm-metronome" />
```


### 렌더링 전략

1. 정적 사이트 생성(SSG)
	- 빌드 시점에 생성된 HTML이 모든 요청에 사용됨. (이미 모든 페이지가 미리 렌더링 되어있음.)
	- SEO에 가장 적합한 렌더링 전략
2. 서버 측 렌더링(SSR)
	- HTML 요청 시점에 페이지를 렌더링 해서 생성
	- 사전에 렌더링 되므로 SEO에 적합함.
3. 점진적 정적 재생(ISR)
	- 페이지 수가 많은 경우, 사이트 빌드 후에 정적 페이지를 생성하거나 업데이트를 할 수 있음. 
	- 한번 만든 페이지는 특정 시간 동안 캐싱을 해둬서 다음 요청에서는 바로 사용함. 
	- SEO에 적합함. 
4. 클라이언트 측 렌더링 (CSR)
	- 자바스크립트를 불러와 브라우저가 모든 것을 컴파일 함. 
	- 그 전까지는 컨텐츠가 거의 없는 단일 HTML 파일만 제공됨. 
	- SEO에 권장되지 않음. 


### URL 구조

- 의미론적: URL은 의미가 있는 것이 좋으며, 이는 단어를 사용하는 대신 ID나 임의의 숫자를 사용하지 않는다는 의미. 
	- 예: `/learn/basics/create-nextjs-app`가 `/learn/course-1/lesson-1`보다 좋음. 
- 논리적이고 일관된 패턴: URL은 페이지 간에 일관된 어떤 패턴을 따라야 함. 
	- 예를 들어, 각 제품마다 다른 경로를 사용하는 대신 모든 제품 페이지를 그룹화하는 폴더를 만드는 것이 좋음.
- 키워드 중심: 구글은 여전히 웹사이트에 포함된 키워드를 기반으로 상당 부분의 시스템을 운영하고 있음. URL에 키워드를 사용하면 페이지의 목적을 이해하는 데 도움이 됨.
- 매개변수 기반이 아님: 매개변수를 사용하여 URL을 구성하는 것은 일반적으로 좋은 생각이 아님. 이들은 대부분 의미론적이지 않으며, 검색 엔진이 혼란을 느끼고 결과에서 순위를 낮출 수 있음.


### 메타 태그

``` tsx
import Head from 'next/head';
 
 // Next.js 라우터 방식 예시 코드
function IndexPage() {
  return (
    <div>
      <Head>
        <title>Meta Tag Example</title>
        <meta name="google" content="nositelinkssearchbox" key="sitelinks" />
        <meta name="google" content="notranslate" key="notranslate" />
      </Head>
      <p>Here we show some meta tags off!</p>
    </div>
  );
}
 
export default IndexPage;
```
- `nositelinkssearchbox`: 구글 검색 결과에서 사이트에 특화된 검색 상자를 표시하지 않도록 지시함.
- `notranslate`: 구글에게 이 페이지의 번역을 제공하지 않도록 알려줌. 
- `noindex`: 이 페이지를 검색 결과에 표시하지 않음. 
- `nofollow`: 이 페이지에서 크롤링으로 링크를 따라가지 않음. 


### 메타데이터

1. 제목
	- 사용자가 웹사이트를 클릭할 때 보이는 것
	- 구글이 페이지가 어떤 내용인지 이해하는데 주요한 요소
	- 키워드를 사용하는 것을 권장

``` html
<title>iPhone 12 XS Max For Sale in Colorado - Big Discounts | Apple</title>
```

2. 설명: 순위 결정에는 고려되지 않지만, 검색 결과에서 클릭율에 영향을 미칠 수 있음. 

``` html
<meta
  name="description"
  content="Check out iPhone 12 XR Pro and iPhone 12 Pro Max. Visit your local store and for expert advice."
/>
```

3. 오픈 그래프: 공유될때 풍부한 미리보기(제목, 설명, 이미지)를 제공함. 

``` html
<Head>
	<title>Cool Title</title>
    <meta name="description" content="Checkout our cool page" key="desc" />
    <meta property="og:title" content="Social Title for Cool Page" />
    <meta
        property="og:description"
        content="And a social description for our cool page"
    />
    <meta
        property="og:image"
        content="https://example.com/images/cool-page.jpg"
    />
</Head>
```

##### Next.js 메타데이터

``` tsx
// app/layout.tsx 또는 app/page.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: '내 프로젝트 대시보드',
  description: 'Next.js로 만든 효율적인 관리 시스템',
  openGraph: {
    title: '내 프로젝트 대시보드',
    description: 'Next.js로 만든 효율적인 관리 시스템',
    url: 'https://my-app.com', // 이 페이지의 절대 주소
    siteName: 'Next App',
    images: [
      {
        url: 'https://my-app.com/og-image.png', // 공유 시 보여줄 이미지 주소
        width: 1200,
        height: 630,
      },
    ],
    locale: 'ko_KR',
    type: 'website',
  },
  alternates: { 
    canonical: `https://ytmetronome.com/${locale}/terms`, // 캐노니컬 주소
  },
};
...
```

- 공통 메타데이터
``` tsx
// app/layout.tsx
import { Metadata } from 'next';
 
export const metadata: Metadata = {
  title: {
    template: '%s | Acme Dashboard',
    default: 'Acme Dashboard',
  },
  description: 'The official Next.js Learn Dashboard built with App Router.',
  metadataBase: new URL('https://next-learn-dashboard.vercel.sh'),
};
```

``` tsx
// app/dashboard/invoices/page.tsx
export const metadata: Metadata = {
  title: 'Invoices',
};
```

`app/dashboard/invoices` 페이지 제목이 `Invoices | Acme Dashboard` 로 표시됨.


### JSON-LD

- 구글 검색 결과가 페이지를 이해하는데 도움을 줌. (별점, 가격, 조리 시간 등 부가 정보 포함)
- search.org(검색  엔진들이 웹페이지의 정보를 일관되게 이해할 수 있도록 만든 공용 언어 사전) 문법을 사용
- 현재 웹 SEO(검색 엔진 최적화) 분야에서 **가장 표준적이고 중요한 기술**, 직접적인 클릭률 상승에 큰 도움을 줌.

``` tsx
export default function Page() {
	const blogJsonLd = {
	  "@context": "https://schema.org",
	  "@type": "BlogPosting", // "이 데이터는 블로그 게시글이야"라고 선언
	  "headline": "Next.js에서 SEO를 정복하는 방법",
	  "image": [ // 대표 이미지
	    "https://example.com/photos/1x1/photo.jpg"
	  ],
	  "datePublished": "2024-04-22T12:00:00+09:00", // 발행일
	  "author": { // 작성자 이름
	    "@type": "Person",
	    "name": "김철수"
	  },
	  "description": "Next.js의 Metadata API와 JSON-LD 사용법을 깊이 있게 다룹니다." // 요약
	};

  return (
    <section>
      {/* 검색 엔진을 위한 JSON-LD 주입 */}
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(blogJsonLd) }}
      />
      {/* ... 나머지 UI */}
    </section>
  );
}
```


### 페이지 SEO

1. H1: 각 페이지에 H1 태그를 사용하는 것이 좋음. H1은 `title` 태그와 유사함.
``` tsx
function Page() {
  return <h1>Your Main Page Heading</h1>;
}
```

2. 백링크: 외부의 신뢰도 높은 웹사이트가 특정 사이트를 링크하면 그 웹사이트는 점수가 올라가고 검색엔진에 유리해짐. 
3. 내부 링크: 자신의 웹사이트 내부에서 서로 링크로 연결되어 있으면, 검색 엔진이 사이트의 모든 페이지를 찾을 수 있고 메인 페이지가 받은 점수를 하위 페이지들에게 나눠줄 수 있음. 
4. 외부 링크: 자신의 웹사이트에서 신뢰도 높은 사이트를 링크하면 좋은 정보를 유도하는 허브로써 점수가 오를 수 있음. 


### 웹 바이탈

- 구글이 웹에서 사용자 페이지 경험을 측정하기 위한 가이드라인과 메트릭을 제공
- 코어 웹 바이탈 메트릭에서 낮은 점수를 받으면 검색 엔진 순위에 영향을 미칠 수 있음. 
- https://vercel.com/products/observability 에서 코어 웹 바이탈 기준으로 페이지 성능 테스트를 할 수 있음. 

- **코어 웹 바이탈** 
	1. LCP (대형 컨텐츠 렌더링 시간): **페이지의 로딩 성능 평가**, 페이지의 가장 큰 요소가 뷰포트에 보이는 데 걸리는 시간 측정
	2. FID (첫 입력 지연): **최초의 클릭 반응 속도 평가**, 웹 페이지와 상호작용하는 동안의 사용자 경험
		- 2024년도부터 FID 대신 INF를 사용함. 
		- INF: 페이지에 머무는 동안 발생하는 '모든' 상호작용의 응답 속도를 측정.
	3. CLS (누적 레이아웃 변동): **전반적인 레이아웃 안정성 평가**, DOM에서 처음 렌더링된 후 요소가 예상치 못한 움직임이 발생한 경우에만 반영됨. 
		- 사용자 클릭 직후(500ms 이내)에 발생하는 레이아웃 이동은 반영되지 않음. 
		- 비동기 처리인 경우에 먼저 스켈레톤 UI나 로딩 스피너를 미리 보여주어 공간을 확보하면 됨. 


### 라이트하우스 

- 성능, 접근성, SEO, 권장사항 등 웹사이트의 전체적인 건강 상태를 체크해서 점수를 매기는 툴
- SEO와는 직접적인 영향이 없음. (이 점수가 SEO에 반영되는 것은 아님.)
- 하지만 이 점수를 올리면 SEO 점수가 높은 웹사이트가 됨. 

- 대형 컨텐츠 렌더링 시간 (Largest Contentful Paint): 25%
- 총 차단 시간 (Total Blocking Time): 25%
- 첫 콘텐츠 시간 (First Contentful Paint): 15%
- 속도 지수 (Speed Index): 15%
- 상호작용 시간 (Time to Interactive): 15%
- 누적 레이아웃 변동 (Cumulative Layout Shift): 5%


### 자동 이미지 최적화 

- `next/image`를 사용하면, WebP와 같은 현대 형식의 이미지로 자동 조정하고 최적화하여 보여줌. 
- `width`, `height`는 실제 이미지의 크기를 넣어줘야함.
``` tsx
import Image from 'next/image';

<Image src="/large-image.jpg" alt="Large Image" width={3048} height={2024} />
```

- 용량은 줄이고: WebP/AVIF 변환
- 크기는 딱 맞게: 디바이스별 리사이징
- 필요할 때만 전송: Lazy Loading


### 폰트 최적화

- Next.js에는 웹폰트를 자동으로 최적화하는 기능이 내장되어 있음. 빌드 시점에 웹폰트 관련 CSS가 페이지에 직접 포함됨.
- 그 결과 웹페이지가 로드될 때, 필요한 데이터를 가져오는 과정이 줄어들어, First Contentful Paint(FCP)와 Largest Contentful Paint(LCP)의 성능이 향상됨.

1. 폰트 가져오기
``` tsx
import { Inter, Roboto } from "next/font/google";

// 구글 폰트
const inter = Inter({ variable: "--font-inter", subsets: ["latin"], });
const roboto = Roboto({ weight: '400', subsets: ['latin'] });

// 로컬 폰트
import localFont from 'next/font/local';

const myFont = localFont({
  src: './my-custom-font.woff2',
  variable: '--font-custom',
});
```

2. 적용
``` tsx
// layout.tsx
export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html
      lang="en"
      className={`${inter.variable} h-full antialiased`}
    >
      <body className="min-h-full flex flex-col font-sans">
        <Header />
        {children}
      </body>
    </html>
  );
}
```

- 빌드 시점 자동 다운로드: 빌드 시점에 Next.js가 구글 서버에 필요한 폰트 파일을 **자동으로 다운로드**하여 프로젝트 내부에 포함 시킴. 
- 자체 호스팅: 배포된 후 브라우저는 구글 서버가 아닌, 내 서버에서 직접 폰트를 가져옴. 
- 필요한 페이지에서 폰트 파일을 미리 로드할 수 있도록 HTML 헤더에 `<link rel="preload">` 태그를 자동으로 생성
- 폰트가 로드되기 전과 후의 글자 크기 차이로 인해 화면이 덜컥거리는 현상을 막아줌.


### 동적 import

- 무거운 JS라이브러리를 동적으로 (요청 시, 이벤트 발생 시) import 할 수 있음.
- 초기 자바스크립트 번들의 크기가 작아져 **LCP(가장 큰 콘텐츠 렌더링)** 속도가 빨라져 검색 엔진 점수에 유리합니다.

##### 기본 사용

``` tsx
'use client'

import { useState, useMemo } from 'react';
import debounce from 'lodash/debounce'; // lodash-es 사용 권장

export default function CountrySearch({ countries }) {
  const [results, setResults] = useState(countries);

  // 검색 로직을 debounce로 감싸서 효율화: 0.3초마다 이벤트 실행됨.
  const handleSearch = useMemo(() => debounce(async (value) => {
    if (!value) {
      setResults(countries);
      return;
    }

    // 라이브러리 동적 로드
    const Fuse = (await import('fuse.js')).default;
    const fuse = new Fuse(countries, { keys: ['name'], threshold: 0.3 });
    
    const searchResult = fuse.search(value).map(r => r.item);
    setResults(searchResult.length ? searchResult : countries);
  }, 300), [countries]);

  return (
    <input
      type="text"
      placeholder="Country search..."
      onChange={(e) => handleSearch(e.target.value)}
    />
  );
}
```
- `await import`로 동적 import 가능
- 이벤트와 사용할때는 `lodash/debounce`랑 사용 권장: 실행 횟수를 조절 가능 
- `useMemo`를 사용하면 동적 import를 한번만 해도 저장해서 사용 가능
- `onChange`에 동적 import를 하면 첫 검색 시 아주 미세한 지연이 생길 수 있음. 그래서 보통 `onFocus`때 미리 로드하는 기법을 섞어서 사용함. 

- 실행 흐름
	1. 컴포넌트 렌더링: `useMemo`가 실행되지만, 안쪽의 `async` 함수는 '정의'만 될 뿐 실행되지 않음. (이때 `import` 안 일어남)
	2. 사용자 입력: 사용자가 글자를 입력
	3. Debounce 대기: 300ms 동안 추가 입력이 있는지 지켜봄.
	4. 함수 실행: 입력이 멈추면 비로소 `async (value) => { ... }` 블록이 실행
	5. 동적 임포트: **`await import('fuse.js')` 줄에 코드가 닿는 순간** 브라우저가 네트워크 탭을 통해 라이브러리를 가져옴.


##### 컴포넌트 단위

무거운 라이브러리를 사용하는 로직을 별도의 컴포넌트에 두고 컴포넌트를 동적 import 하는 방식
- 컴포넌트 내부: 일반 import
- 메인 페이지: `next/dynamic`으로 동적 import


``` tsx
import dynamic from 'next/dynamic';

// 검색 로직이 포함된 무거운 컴포넌트를 필요할 때만 로드 (dynamic 사용)
const SearchPanel = dynamic(() => import('../components/SearchPanel'), {
  ssr: false, // 클라이언트 사이드에서만 필요하므로
  loading: () => <p>Loading Search...</p>
});

...
return (
  <>
    <button onClick={() => setShow(true)}>검색창 열기</button>
    {show && <SearchPanel />} {/* 이 순간 import 수행! */}
  </>
)
```
- 컴포넌트가 화면에 그려져야할 때 import 됨.
- 조건 없이 바로 렌더링 되는 경우에도 메인 번들 파일에 없고 나중에 별도로 다운로드 됨. 
- `ssr: false`
	- 서버: HTML을 만들 때 `SearchPanel` 자리는 비워두고(혹은 Loading UI만 넣고) 클라이언트로 보냄.
	- 브라우저: HTML을 받고 나서 "아, 이건 내가 직접 가져와야 하는구나"라고 깨닫고 그때서야 파일을 요청


### 외부 스크립트 최적화

- 많은 애플리케이션들은 분석 기능, 광고, 고객 지원 위젯과 같은 다양한 기능을 구현하기 위해 외부 자바스크립트를 사용함. 
- 하지만, 외부 자바스크립트를 사용하면 페이지의 콘텐츠가 제대로 표시되는 데 지연이 생길 수 있으며, 코드가 너무 빨리 로드될 경우 사용자 경험에도 영향을 줄 수 있음.
- Next.js의 Script 컴포넌트는 `strategy` 이라는 속성을 통해서 페이지의 콘텐츠가 모두 로드된 후에 스크립트를 실행하도록 설정할 수 있음.

- 기존 
``` tsx
import Head from 'next/head';
 
function IndexPage() {
  return (
    <div>
      <Head>
        <script src="https://www.googletagmanager.com/gtag/js?id=123" />
      </Head>
    </div>
  );
}
```

- Script 컴포넌트 사용
``` tsx
import Script from 'next/script';
 
function IndexPage() {
  return (
    <div>
      <Script
        strategy="afterInteractive"
        src="https://www.googletagmanager.com/gtag/js?id=123"
      />
    </div>
  );
}
```


### Next.js 분석 도구

##### Vercel Speed Insights

- 코어 웹 바이탈 기반으로 페이지의 성능을 분석한 대시보드
- https://vercel.com/products/observability

##### reportWebVitals

- 코어 웹 바이탈 기반 성능 데이터를 수집하는 센서
- 데이터를 내 서버로 보내거나, 구글 분석기(Google Analytics) 등으로 보낼 수 있음.

1. 설치
``` bash
npx create-next-app@latest nextjs-lighthouse --use-npm --example "https://github.com/vercel/next-learn/tree/main/seo"
```

2. `pages/\_app.js`
``` tsx
export function reportWebVitals(metric) {
  console.log(metric);
  // Google Analytics나 서버로 보내는 로직을 작성해도 됨. 
}
```

##### 기타 도구 

- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/): 구글이 제공하는 페이지 속도 측정 도구
- [Chrome User Experience Report](https://developers.google.com/web/tools/chrome-user-experience-report): # 크롬 사용자 경험 보고서, 실제 크롬 사용자가 웹에서 인기 있는 사이트를 어떻게 경험하는지에 대한 사용자 경험 지표를 제공
- [Search Console](https://support.google.com/webmasters/answer/9205520): 구글 검색, 코어 웹 바이탈 보고서

- [Lighthouse](https://developers.google.com/web/tools/lighthouse?hl=en): 구글이 개발한 오픈소스 자동화 도구로, 웹페이지의 품질을 향상시키는 데 사용
- [Chrome DevTools](https://developer.chrome.com/docs/devtools/): 구글 크롬 브라우저에 직접 내장된 웹 개발을 위한 도구들


### SEO 설정 순서

##### 1단계: 키워드 리서치 (기획 & 조사)

- 영어권, 한국어권, 프랑스어권 등 각 타겟 국가별로 검색량이 높으면서 경쟁해볼 만한 핵심 키워드들을 먼저 리스트업합니다.

##### 2단계: 콘텐츠(설명글 & FAQ) 및 메타데이터 작성

- 조사한 키워드들이 자연스럽게 녹아들도록 **제목(Title), 설명(Description), 본문 텍스트, FAQ 질문과 답변**을 각 언어별로 작성합니다.
- 예: `messages/ko.json` 및 `en.json` 등에 SEO 전용 번역 키를 추가하는 작업이 여기에 해당합니다.

##### 3단계: 기술적 SEO 설정 (Technical SEO - 개발)

- **`robots.txt`**: 검색 로봇의 출입 가이드라인을 제공합니다.
- **`sitemap.xml`**: 검색 로봇이 사이트 맵을 한눈에 볼 수 있도록 전체 페이지 지도를 제공합니다.
- **`layout.tsx` 내 다국어 메타데이터 동적 설정**: 각 언어별로 최적화된 Title, Description, Open Graph(소셜 공유 시 보이는 이미지/설명), `hreflang` 태그가 자동으로 랜더링되도록 구현합니다.

##### 4단계: 구조화된 데이터(Schema Markup) 적용 (권장)

- 구글 검색 결과에서 질문과 답변이 아코디언처럼 바로 노출되도록 하는 **FAQ Schema(JSON-LD)** 코드를 심어 웹페이지의 검색 노출 형태를 풍부하게 만듭니다.


### SEO 키워드 찾기

##### 1. 구글 검색창 활용하기 (가장 쉽고 강력함 🌟)

구글에 직접 검색해 보며 사용자의 의도를 파악하는 직관적인 방법입니다.

- **구글 자동완성 (Google Autocomplete)**: 구글 검색창에 `"메트로놈 "` 또는 `"metronome "`을 입력하고 한 칸 띄어보세요. 뒤이어 나오는 추천 키워드들(예: `메트로놈 추천`, `메트로놈 웹`, `온라인 메트로놈`, `metronome online sync`)이 현재 가장 많이 검색되는 키워드입니다.
- **관련 검색어 및 피플 서치 (People Also Ask)**: 검색 결과 화면의 중간이나 맨 아래로 내려가면 **"관련 검색어"** 영역이 나옵니다. 여기에 나오는 단어들이 검색 엔진이 관련성이 높다고 인지하는 키워드 묶음입니다.

##### 2. 무료 키워드 분석 도구 활용하기

- **구글 키워드 플래너 (Google Keyword Planner)**: 구글 애즈(Google Ads) 계정만 만들면 무료로 쓸 수 있는 구글 공식 도구입니다. 특정 키워드(예: "메트로놈")를 넣으면 관련 키워드 목록과 함께 **월간 검색량**, **경쟁 정도**를 정확한 숫자로 보여줍니다.
- **구글 트렌드 (Google Trends)**: "온라인 메트로놈"과 "인터넷 메트로놈" 중 어떤 단어의 검색량이 더 많은지 비교하고 싶을 때 유용합니다. 기간별, 국가별 대략적인 관심도 추이를 비교할 수 있습니다.
- **AnswerThePublic (answerthepublic.com)**: 구글에 "메트로놈"을 검색하는 사람들이 던지는 **질문형 키워드**를 모아주는 사이트입니다. (예: `메트로놈 사용법은 무엇인가요?`, `왜 메트로놈을 써야 하나요?` 등) 본문의 FAQ 섹션을 작성할 때 완벽한 소스가 됩니다.

##### 3. 경쟁 서비스 분석하기 (벤치마킹)

구글에 "Online Metronome"이나 "온라인 메트로놈"을 검색했을 때 상위에 나오는 1~3위 사이트들을 들어가 봅니다.
그 사이트들이 **제목(`<h1>`, `<h2>`)**과 **본문 텍스트**에 어떤 단어들을 반복적으로 사용하고 있는지 살펴보세요.

💡 **결론적으로**, 이렇게 찾은 키워드들을 자연스럽게 메인 페이지 하단의 제목(`<h2>`, `<h3>`)과 본문 문장에 녹여내면, 검색 엔진이 해당 키워드로 우리 사이트를 상위에 노출해 주게 됩니다.


### 구글 키워드 플래너 

- 경쟁 지표: 돈 내고 광고판이 들어온 광고주들의 밀도, 무료로 검색창에 띄우는 자연 검색(SEO) 영역의 경쟁률을 나타내지는 않음. 
- 월간 평균 검색량 지표: 한 달에 유저들의 검색량, 10~100 이상이면 채택하는 게 좋음

##### 키워드 채택 방법

1. **키워드 플래너:** "한 달에 유저들이 `10~100` 혹은 `1천~1만` 번이나 검색하네? 오케이, 수요 확인!" (1차 통과)
2. **구글 직접 검색:** "1페이지에 나무위키도 없고 제대로 된 메트로놈 웹 앱도 없네? 완전 빈집이구나!" (최종 채택)

##### 글로벌 SEO 

1. **기획:** 먼저 한국어로 사이트 구조, 기능 소개 글, FAQ의 '뼈대 내용'을 완성합니다. (어차피 영어 페이지도 이 순서대로 설명할 거니까요.)
2. **영어 키워드 수집:** 구글 키워드 플래너 국가를 '미국/전 세계'로 바꾸고, 영어권 유저들이 쓸 만한 키워드를 20~30개 수집합니다.
3. **영어 문장 작성:** 1단계에서 만든 한국어 뼈대를 보면서, 2단계에서 수집한 영어 키워드들을 사용해 영어권 감성에 맞는 자연스러운 문장으로 한 줄 한 줄 새로 작성(로컬라이징)합니다.

### 키워드 사용

##### 1단계: 메인 페이지의 '메타데이터' 최적화 (가장 중요)

수집하신 키워드 중 검색량이 가장 높은 메인 키워드(`메트로놈`)와 타겟 키워드(`PC 메트로놈`, `구글 메트로놈`)를 웹사이트의 간판에 걸어야 합니다. Next.js의 `page.tsx` 또는 `layout.tsx`에 다음과 같이 반영하세요.

TypeScript

``` tsx
// app/page.tsx 또는 app/layout.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: '무료 온라인 메트로놈 | YT Metronome (유튜브 동기화)',
  description: '별도 설치 없는 최고의 무료 PC 메트로놈 서비스입니다. 유튜브 동기화 기능으로 원하는 악보나 영상(TAB)을 보며 16비트, 4/4 박자 연습을 간편하게 즐겨보세요.',
  keywords: ['메트로놈', '온라인 메트로놈', 'PC 메트로놈', '구글 메트로놈', '유튜브 메트로놈'],
};
```

- **Title 팁:** 검색량이 높은 `메트로놈`과 서비스 명인 **`YT Metronome`**, 그리고 가장 큰 차별점인 `유튜브 동기화`를 조합해 30자 내외로 잡습니다.
- **Description 팁:** 수집한 **`PC 메트로놈`**, **`16비트`**, **`4/4 박자`** 같은 키워드들을 사용자가 읽기 좋은 자연스러운 문장으로 엮어서 작성합니다. 구글 검색 결과 창에 이 문장이 그대로 노출됩니다.


##### 2단계: 본문 (Body) 콘텐츠에 키워드 자연스럽게 녹이기

구글 봇은 메타데이터뿐만 아니라 화면에 실제로 보이는 텍스트도 꼼꼼히 읽습니다. 현재 메트로놈 UI 아래쪽에 공간이 좀 있으니, 그 영역에 키워드가 포함된 안내 문구를 `<h2>`나 `<p>` 태그로 넣어주면 효과적입니다.

- **구글 봇이 좋아하는 구조 예시:**
    ``` tsx
    <h2>왜 YT 온라인 메트로놈인가요?</h2>
    <p>
      본 서비스는 스마트폰뿐만 아니라 <strong>PC 메트로놈</strong> 환경에서도 완벽하게 최적화되어 있습니다. 
      특히 <strong>유튜브 동기화 기능</strong>을 통해 좋아하는 악보 영상이나 기타 TAB 악보를 화면에 띄워두고 
      <strong>16비트 박자</strong>, <strong>4/4 박자</strong> 등 다양한 리듬 연습을 한 화면에서 동시에 진행할 수 있습니다.
    </p>
    ```
    
- 무작정 키워드만 도배하면 구글이 스팸으로 처리하니, 위처럼 유저에게 기능을 설명하는 자연스러운 문맥 속에 핵심 키워드를 볼드(`<strong>`) 처리해 주는 것이 정석입니다.


##### 3단계: 이미지 `alt` 속성 채우기

현재 화면 하단에 유튜브 썸네일 이미지들이 렌더링되고 있습니다. 구글은 이미지 자체를 읽지 못하므로 `alt` 속성을 보고 판단합니다.

- Next.js의 `<Image>` 태그나 기본 `<img>` 태그를 쓸 때 `alt="유튜브 동기화 메트로놈 연습 영상"` 혹은 `alt="메트로놈 16비트 타이머 아이콘"` 같은 식으로 키워드를 넣어주면 구글 이미지 검색 탭을 통한 유입도 노려볼 수 있습니다.


##### 4단계: 롱테일(세부) 키워드로 콘텐츠 확장하기 (선택)

아까 리스트에서 `16비트 메트로놈`, `4/4 박자 메트로놈` 같은 세부 키워드들을 보셨을 겁니다. 검색량은 적지만 경쟁이 낮아 구글 상위 노출이 아주 쉽습니다.

- 만약 여유가 된다면 Next.js에 하위 라우팅을 파서 가이드 페이지나 블로그 섹션을 만드는 것을 추천합니다.
- 예: `extractframe.org/blog/how-to-practice-16-beat` 같은 주소로 "메트로놈으로 16비트 리듬 쪼개서 연습하는 효율적인 방법"이라는 글을 적어두면, 그 키워드를 검색한 악기 연습 유저들이 글을 보러 왔다가 자연스럽게 내 메트로놈 서비스를 이용하게 됩니다.


### 키워드 배치 전략

##### 1. 키워드 중복 노출(Keyword Stuffing) 피하기

구글 등 검색 엔진은 단순히 비슷한 키워드를 억지로 여러 번 나열하는 행위를 스팸으로 인지하여 검색 순위에 악영향을 줍니다. 따라서 관련 키워드들을 **하나의 자연스러운 문맥(맥락) 안에서 흐름에 맞게 번역 및 배치**해야 합니다.

##### 2. 키워드 등급별 배치 전략

`keyword-en.md`에 정리해두신 키워드들의 검색량(그룹)에 따라 중요도를 다르게 배치합니다.

- **메인 키워드 (검색량이 가장 높은 대표 단어: 예: `online metronome`, `metronome`, `BPM`)**:
    - 소제목(H2, H3)이나 각 본문 문단의 **첫 번째 문장**에 배치하여 검색 로봇에게 이 문단의 핵심 주제를 명확하게 알립니다.
- **서브 키워드 (1천~1만 키워드: 예: `metronome BPM`, `song BPM`, `rhythm practice`)**:
    - 기능을 상세하게 설명하는 문장의 주어/목적어로 녹여냅니다.
- **롱테일 키워드 (100~1천 키워드: 예: `track BPM`, `bpm for music`, `metronome for piano practice`)**:
    - 악기별 연습 사례나 유튜브 기능 같은 세부적인 설명을 다루는 문장에 자연스러운 수식어로 배치합니다.