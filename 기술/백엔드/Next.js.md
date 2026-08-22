### 기본 구조

``` txt
/
├── app/
│   ├── layout.tsx                # 전역 레이아웃 (공통 헤더, 푸터)
│   ├── page.tsx                  # 홈페이지 (/)
│   │
│   ├── dashboard/                # /dashboard 경로
│   │   ├── page.tsx              # 대시보드 페이지
│   │   ├── layout.tsx            # 대시보드 전용 레이아웃 (사이드바 등)
│   │   ├── loading.tsx           # 대시보드 전용 로딩 UI
│   │   │
│   │   └── _components/          # 💡 Dashboard 전용 컴포넌트 (비공개 폴더)
│   │       ├── revenue-chart.tsx
│   │       └── latest-invoices.tsx
│   │
│   ├── invoices/                 # /dashboard/invoices 경로
│   │   ├── page.tsx
│   │   ├── table.tsx             # 💡 Invoices 전용 컴포넌트 (현재 폴더에 함께 두기)
│   │   └── status.tsx            # 💡 Invoices 전용 컴포넌트
│   │
│   └── (auth)/                   # 경로에 영향을 주지 않는 라우트 그룹
│       ├── login/
│       │   └── page.tsx
│       └── register/
│           └── page.tsx
│
├── components/                   # 💡 여러 페이지에서 사용되는 '공통' UI 컴포넌트
│   ├── button.tsx
│   ├── input.tsx
│   └── modal.tsx
│
├── lib/                          # 서버 측 유틸리티 및 함수
│   ├── definitions.ts            # TypeScript 타입 정의
│   ├── actions.ts                # Server Actions
│   ├── data.ts                   # Data Fetching 로직 (DB 쿼리)
│   └── utils.ts                  # 일반 유틸리티 함수
│
└── public/                       # 이미지, 폰트 등 정적 자산

```

- `/app` 하위에 **폴더**를 만들면 그 이름이 **URL 주소**가 됨.
- 그 폴더 안에 `page.tsx`이름을 가진 파일만 실제 웹 페이지(URL)가 됨. 
- 그 폴더 안에 `layout.tsx`를 넣으면 **그 폴더와 하위 페이지**들이 공통 디자인을 나눠 가짐. 

- 폴더 이름 앞에 `_`를 붙이면 Next.js 라우팅 시스템이 해당 폴더를 무시함
- 괄호로 감싼 폴더는 URL 경로에 포함되지 않음.

- `global.css`,`fonts.ts`를 어디에 위치시킬지는 정해진 것이 없음. (`/app`, `app/lib`, `app/compoents` 등)
- `global.css`: 공통으로 사용할 CSS 설정 (보통 Tailwind 설정)
- `fonts.ts`: 폰트들을 관리하는 파일
	- Next.js는 `next/font` 모듈을 사용할 때 애플리케이션 내의 폰트를 자동으로 최적화, 빌드 시간에 폰트 파일을 다운로드하고 다른 정적 자산과 함께 호스팅함. 
	- 이는 사용자가 애플리케이션을 방문할때 추가적인 폰트 요청이 없음을 의미함. 


### 서버 / 클라이언트 컴포넌트

**1. 서버 컴포넌트 (Server):** **서버에서 딱 한 번 실행**되어 결과물(HTML)만 브라우저로 보냄. 브라우저는 자바스크립트 코드를 거의 받지 않으므로 로딩이 빠르고 보안에 유리. 
- DB 접근, 백엔드 API 직접 호출 등 자유로움.
- 컴포넌트의 `async/await` 문법을 사용 가능
- Next.js의 app 폴더 하위의 컴포넌트는 **기본적으로 서버 컴포넌트**임. 

**2. 클라이언트 컴포넌트 (Client):** 브라우저에 자바스크립트 코드가 전달되어 **사용자의 브라우저에서 실행**되므로 JS 상호작용이 가능. `'use client'`로 지정.
- `useState`, `useEffect` 등의 훅 사용 가능 (리액트 Hook은 사용자와 상호작용하며 화면을 계속 변화시키는 것이므로 클라이언트 컴포넌트에서만 사용 가능)
- `onClick`, `onChange` 와 같은 이벤트 리스너 사용 가능 
- 브라우저 API(`window`, `document` 등) 접근 가능
- `.tsx` 파일은 `.js`로 변환되기 때문에, 클라이언트 컴포넌트의 코드는 전부 브라우저로 내려간다고 보면 됨.


서버 컴포넌트는 JavaScript Promises를 지원하므로 데이터 가져오기와 같은 비동기 작업을 `async/await` 문법을 사용하여 처리함. (`useEffect` , `useState` 또는 기타 데이터 가져오기 라이브러리가 필요하지 않음)

클라이언트 컴포넌트를 `async`로 만들면 리액트가 제대로 렌더링을 할 수 없음. 그러므로 비동기 처리를 하려면 `useEffect` 안에서 `async/await`를 사용해야 함. 


### 서버 / 클라이언트 캐시

**1. 서버 캐시 (정적 렌더링 캐시)
- 빌드(`npm run build`)할 때 만들어 놓은 **사이트 전체의 정적인 HTML 결과물**을 전부 캐싱해둠. 
- 어떤 사용자든 접속하면 이 캐싱된 결과를 바로 반환해줌. 
**2. 클라이언트 캐시 (브라우저 라우터 캐시)**
- 사용자가 이리저리 이동하면서 **서버로부터 받아왔던 화면 조각들**을 브라우저 메모리에 일정 시간 캐싱해둠. 
- 여기에 더해서, Next.js는 화면에 `<Link>` 태그가 보이면 브라우저가 **서버에서 다음 페이지의 데이터를 미리 땡겨와서(Prefetch) 클라이언트 캐시에 같이 채워 넣음.**


### Next.js 동작 원리 

- 빌드 시점: Next.js가 코드를 분석해서 "이 페이지는 이 자바스크립트 파일만 있으면 돼"라고 딱딱 나눠둠. (**코드 분할**)
- 브라우저 실행 시점: 사용자가 웹 사이트에 접속하면
	1. 서버에서 빌드 때 **미리 만들어둔 HTML 파일**을 즉시 받아서 화면에 그림. 
	2. **Next.js 런타임**과 리액트 코드가 담긴 자바스크립트 파일들을 다운로드 함. 
- Next.js 런타임 역할: 브라우저의 전권을 넘겨받아서 관리함. (가로채기)
	1. **Link 컴포넌트:** 버튼이 화면에 보이면 다음 페이지를 미리 챙겨둠.
	2. 사용자가 클릭하면, 서버에서 전체 페이지가 아니라 **RSC 페이로드(바뀌는 부분에 대한 데이터)** 만 받아와서 화면을 교체함. (브라우저의 페이지 전체를 새로고침하려는 시도를 Next.js 런타임이 가로챔.)
	3. 브라우저 라우터 캐시: 서버로부터 받은 화면 조각들을 캐싱해둠. 


### `layout.tsx`

- 모든 페이지를 감싸는 "껍데기" 역할을 함. `children` 프로퍼티를 받음. 안의 페이지가 바뀌어도 레이아웃은 다시 렌더링되지 않음. 
- 여기에 `global.css`,`fonts.ts` 를 import 해서 전역에 적용함. 

``` tsx
// layout.tsx
import '@/app/ui/global.css';
import { inter } from '@/app/ui/fonts';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={`${inter.className} antialiased`}>{children}</body>
    </html>
  );
}
```
- `fonts.ts`에서 만든 폰트 변수에서 className을 사용해서 적용
- `antialiased`: 글자의 가독성을 높이고 부드럽게 출력되도록 도와주는 Tailwind CSS 설정


### clsx 

- className에서 조건문을 깔끔하게 사용 가능
``` tsx
import { CheckIcon, ClockIcon } from '@heroicons/react/24/outline';
import clsx from 'clsx';

export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-xs',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
	{/* ... */}
    </span>
  );
}

```


### `<Image>` 

`<Image>`컴포넌트는 HTML `<img>`태그의 확장 기능이며, 다음과 같은 자동 이미지 최적화 기능을 제공
- 이미지 로딩 시 레이아웃이 자동으로 변경되는 것을 방지
- 화면 크기가 작은 기기에 큰 이미지를 전송하지 않도록 이미지 크기를 조정
- 기본적으로 이미지는 지연 로딩 (이미지가 뷰포트에 들어올 때 로드됨)
- WebP,AVIF 와 같은 최신 형식으로 이미지를 제공 (브라우저가 지원할 경우)
- 외부 이미지를 가져오는 경우, `next.config.mjs`에 외부 사이트 도메인이 안전하다는 걸 명시적으로 설정해야 함. 

``` tsx
<Image
  src="/hero-desktop.png"
  width={1000}    // 원본 파일이 1000px
  height={760}    // 원본 파일이 760px
  className="w-full h-auto max-w-[500px]" // 실제 화면엔 가로 500px까지만 나오게 조절
  alt="Dashboard"
/>
```
- `width`, `height`에는 실제 이미지의 크기를 넣어줘야하고 화면에 보여지는 것은 Tailwind로 조절 
- `priority`: 화면 맨 위에 바로 보이는 로고나 메인 배너 이미지는 먼저 받을 수 있도록 하는 속성
- `placeholder="blur"`: 이미지가 로딩되는 동안 흐릿한 저화질 이미지를 먼저 보여줌. 
- `fill`: 사진의 가로/세로 크기를 미리 알 수 없을 때, 부모 요소의 크기에 맞춰 이미지를 꽉 채움. 


### `<Link>`

- 애플리케이션 내 페이지 간 링크를 만들 수 있습니다. 이를 통해 JavaScript를 사용하여 클라이언트 측 탐색을 구현할 수 있습니다. 
- 페이지 전체를 새로고침하지 않고 바뀐 부분만 없데이트 됨. (보통 `layout.tsx`는 그대로 유지 되고 해당 URL 주소의 `page.tsx`로 이동하게 됨.)
- 프리페칭: `<Link>` 컴포넌트가 브라우저 화면(뷰포트) 안에 보이면, Next.js는 연결된 페이지의 코드를 배경에서 **미리 다운로드**해 둠. 

``` tsx
import Link from 'next/link';

export default function Nav() {
  return (
    <nav>
      {/* <a> 대신 <Link>를 사용하세요 */}
      <Link href="/dashboard">
        실행 대시보드로 이동
      </Link>
    </nav>
  );
}
```


### DB 접근 

##### DB 연결

- 개발환경: `.env` 파일에 DB를 연결하는 환경변수 추가
- Vercel 등 클라우드 서비스에서 환경변수 추가

##### Postgres 라이브러리

- PostgresSQL 데이터베이스를 연결하는 드라이버

1. 패키지 설치: `pnpm add postgres`
2. 사용: 보통 `app/lib/data.ts` 같은 파일에 DB 접근 로직을 모아둠. 
``` tsx
// data.ts
import postgres from 'postgres';
 
// sql을 통해서 DB에 접근 가능. 
const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });


export async function fetchRevenue() {
  try {
	// 응답 구조: Type[], 예: result = [{}:Type, {}:Type, ...]
    const result = await sql<Type[]>`SELECT * FROM table`;

    return result;
  } catch (error) {
    console.error('Database Error:', error);
    throw new Error('Failed to fetch revenue data.');
  }
}
```
- 백틱을 사용하면 라이브러리가 SQL 인젝션을 방지하기 위해 파라미터화 처리를 자동으로 수행함
- 기본적으로 DB 접근은 비동기 작업임. 그래서 `async/await` 사용
- 제네릭(<>)으로 타입을 지정하면 해당 타입으로 매핑해서 받을 수 있음. 
- 제네릭을 생략하면 `any[]`나 `Row[]`타입을 반환함.
- 무조건 배열의 형태로 받음. 

### Drizzle 

0. 패키지 설치
``` bash
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit @types/pg
```
- `drizzle-orm`: 실제 코드에서 사용할 핵심 라이브러리
- `drizzle-kit`: 마이그레이션(테이블 생성/수정)을 도와주는 개발용 도구

1. `drizzle.config.ts` 설정
``` ts
import { defineConfig } from 'drizzle-kit';
import * as dotenv from 'dotenv';

// .env.local 파일 환경변수 불러오기
dotenv.config({ path: '.env.local' });

export default defineConfig({
  schema: './db/schema.ts', // 스키마 파일 위치
  out: './drizzle',         // 마이그레이션 파일이 저장될 폴더 (옵션)
  dialect: 'postgresql',    // 사용하는 DB 종류
  dbCredentials: {
    url: process.env.DATABASE_URL!, // .env.local의 데이터베이스 주소
  },
});
```

2. 스키마 정의: DB 테이블 구조를 타입스크립트 코드로 먼저 정의 해둠. 
``` ts
// db/schema.ts
import { pgTable, uuid, text, timestamp } from 'drizzle-orm/pg-core';
import { InferSelectModel } from 'drizzle-orm';

// schema.ts에서 만든 users 테이블
// PostgresSQL에서는 테이블명을 소문자, snake_case 사용
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  createdAt: timestamp('created_at').defaultNow(),
});

// 아래 한 줄로 'User' 타입 자동 생성. 따로 만들 필요가 없음. 
export type User = InferSelectModel<typeof users>;
```

2. DB 연결
``` ts
// db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import * as schema from './schema';

// postgres 라이브러리를 사용하여 데이터베이스 실제 연결 클라이언트 생성
const client = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
// 생성된 DB 클라이언트와 테이블 정보가 담긴 schema를 Drizzle ORM에 전달하여 db 객체 생성
// db 객체를 사용하여 DB 조작이 가능
export const db = drizzle(client, { schema });
```

3. DB 조작
``` ts
// lib/data.ts
import { db } from '@/db/index.ts';
import { users } from '@/db/schema.ts';

export async function fetchUser() {
  try {
	// 응답 구조: User[]
    const data = await db.select().from(users).where(eq(users.email, email));

    return data;
  } catch (error) {
    console.error('Database Error:', error);
    throw new Error('Failed to fetch data.');
  }
}
```
- drizzle이 `db.select().from(users)`에서 `users`의 구조로 타입을 지정해서 결과 데이터를 만들어줌. 

4. 마이그레이션: 타입스크립트로 작성한 **스키마(`schema.ts`)를 바탕으로 실제 DB 테이블을 생성하거나 수정함.** 
	- `drizzle-kit`이라는 도구를 이용해서 코드를 읽어서 SQL 파일을 자동으로 만들어주고 DB에 반영할 수 있음.
	- `npx drizzle-kit push`: 스키마를 바탕으로 실제 DB에 테이블을 만드는 명령어


##### 다대일 관계

Drizzle ORM은 철저하게 **"개발자가 명시적으로 적지 않은 쿼리는 절대 알아서 날리지 않는다"** 는 철학을 가지고 있음. 

- **`with: { video: true }`를 안 적으면?** DB에서 `video` 데이터를 아예 가져오지 않음. 그리고 타입스크립트 레벨에서 `result.video`라는 속성 자체가 존재하지 않게 처리됨. 만약 런타임(JavaScript)에 강제로 `result.video`를 꺼내려고 하면 뒤늦게 쿼리가 날아가는 게 아니라, 그냥 `undefined`가 튀어나옴.
    
- **`with: { video: true }`를 적으면?** 단 한 번의 거대한 쿼리(내부적으로 JSON 매핑 또는 Left Join 사용)를 만들어서 `user_video_requests`와 `videos`를 한 방에 가져와서 예쁘게 `result.video` 객체로 만들어줌.


1. `schema.ts`: `relations` 설정
``` ts
// db/schema.ts 맨 아래에 추가
import { relations } from 'drizzle-orm';

export const userVideoRequestsRelations = relations(userVideoRequests, ({ one }) => ({
  // user_video_requests 입장에서 비디오(video)는 1개(one)와 연결된다는 뜻
  video: one(videos, {
    fields: [userVideoRequests.videoId],
    references: [videos.id],
  }),
}));
```

2. `data.ts`: `with` 또는 join문으로 가져오기

- Relational Query 방식: `findFirst`, `findMany` + `with`
``` ts
export async function getUserVideoRequest(userId: string, videoId: string) {
    try {
        const result = await db.query.userVideoRequests.findFirst({
            where: (req, { and, eq }) => and(eq(req.userId, userId), eq(req.videoId, videoId)),
            with: { video: true } // 👈 이렇게 하면 videos 정보까지 JOIN해서 한 방에 가져옵니다!
        });
        return result || null;
    } catch (error) {
        console.error("UserVideoRequest 조회 중 에러 발생:", error);
        throw error;
    }
}
```
`with` 옵션은 Drizzle의 Relational Queries (즉, `db.query.테이블명.findFirst` 또는 `findMany`)에서만 사용 가능

- `select` + `join`
``` ts
const result = await db.select({
    // 가져올 필드 명시 (중복 이름 충돌 방지를 위해 객체로 묶는 것이 일반적)
    request: userVideoRequests,
    video: videos
})
.from(userVideoRequests)
.innerJoin(videos, eq(userVideoRequests.videoId, videos.id)) // 직접 JOIN
.where(
    and(
        eq(userVideoRequests.userId, userId),
        eq(userVideoRequests.videoId, videoId)
    )
);

result[0].request 
result[0].video
```


##### 트랜잭션

스프링은 Thread-Local을 이용해 보이지 않게(Implicit) 트랜잭션 컨텍스트(Connection)를 관리하지만, Node.js 기반의 Drizzle ORM은 명시적(Explicit)으로 트랜잭션 객체(`tx`)를 함수로 넘겨주어야 함.

1. `actions.ts`에서 트랜잭션을 관리하는 방식
``` ts
// db/data.ts (리포지토리)
export async function createVideo(tx: any, video: NewVideo) { // tx를 주입받아야 함
    await tx.insert(videos).values(video);
}
export async function incrementUserRequestCount(tx: any, userId: string) {
    await tx.update(users)...
}

// lib/actions.ts (또는 별도의 서비스 계층)
export async function myAction() {
    await db.transaction(async (tx) => {
        await createVideo(tx, videoData);
        await incrementUserRequestCount(tx, userId);
    });
}
```

- 장점: `data.ts`의 함수들이 완벽하게 분리되어 있어 재사용성이 높음. 
- 단점: `actions.ts`마다 `db.transaction` 코드가 들어가서 복잡해 보이고, `data.ts` 함수를 호출할 때마다 매번 `tx` 파라미터를 넘겨야해서 코드가 지저분해짐. 

2. `data.ts`에서 전용 트랜잭션 메서드를 만드는 방식
``` ts
// db/data.ts

// 유저 생성과 포인트 지급을 '하나의 세트'로 묶어서 처리하는 함수를 새로 팝니다.
export async function createUserWithPoints(userData: NewUser, amount: number) {
  return await db.transaction(async (tx) => {
    // 1. 유저 생성 (내부니까 외부에서 tx를 받을 필요 없이, 바로 이 스코프의 tx를 사용)
    const result = await tx.insert(users).values(userData).returning();
    
    // 2. 포인트 지급
    await tx.insert(points).values({ userId: newUser.id, amount });
    
    return result[0];
  });
}
```

- 장점: `actions.ts`에서는 트랜잭션을 몰라도 됨. 
- 단점: `data.ts` 함수들의 재사용성이 낮아짐. 

3. `AsyncLocalStorage`로 트랜잭션을 관리하는 방식
	
	스프링에서 `ThreadLocal`을 이용해서 요청마다 트랜잭션 컨텍스트를 유지하듯,  Node.js에는 `AsyncLocalStorage`이 `ThreadLocal`과 비슷한 역할을 함. 
	그것을 이용해서 트랜잭션 래퍼 함수를 만들어서 사용
	
- 트랜잭션 래퍼 함수 만들기 
	``` ts
	// src/lib/tx-manager.ts
	import { AsyncLocalStorage } from 'async_hooks';
	import { db } from '@/db';
	
	// 트랜잭션 전용 스토리지를 만듭니다.
	export const txStorage = new AsyncLocalStorage<any>();
	
	// Spring의 @Transactional 역할을 할 함수
	export async function runInTransaction<T>(callback: () => Promise<T>): Promise<T> {
	  return await db.transaction(async (tx) => {
	    // 현재 실행 흐름(Context)에 tx 객체를 저장합니다.
	    return txStorage.run(tx, callback);
	  });
	}
	```

- 사용
``` ts
// src/lib/data.ts
import { db } from '@/db';
import { txStorage } from './tx-manager';
import { users } from '@/db/schema';

export async function insertUser(userData: any) {
    // 현재 스토리지에 트랜잭션(tx)이 있으면 그걸 쓰고, 없으면 기본 db를 씁니다!
    const client = txStorage.getStore() || db;
    return await client.insert(users).values(userData).returning();
}

// src/app/actions.ts
'use server';

import { runInTransaction } from '@/lib/tx-manager';
import { insertUser, insertPoints } from '@/lib/data';

export async function registerUserAction(formData: any) {
  try {
    // 이 블록 안에서 실행되는 모든 data.ts 메서드는 자동으로 같은 트랜잭션으로 묶입니다.
    // 매번 tx를 인자로 넘길 필요가 없습니다!
    return await runInTransaction(async () => {
      const [newUser] = await insertUser({ email: formData.email });
      await insertPoints({ userId: newUser.id, amount: 1000 });
      return newUser;
    });
  } catch (error) {
    return { success: false };
  }
}
```


##### drizzle-kit

1. **DB의 모든 테이블을 schema에 정의하는 것이 원칙**
	- schema 파일은 "DB 전체의 진실"이어야 합니다. 코드에서 쿼리를 안 하더라도, DB에 존재하는 테이블이면 schema에 넣어두는 게 일반적
	- 이렇게 해야 `push`나 마이그레이션 도구를 안전하게 사용할 수 있음.

2. **실무에서는 `push` 대신 마이그레이션(migration)을 사용**
	- `drizzle-kit push`는 **개발 환경에서 빠르게 프로토타이핑**할 때 쓰는 도구
	- 프로덕션에서는 `drizzle-kit generate`로 **마이그레이션 SQL 파일을 생성**하고, `drizzle-kit migrate`로 적용
	- 마이그레이션은 **변경 사항(diff)만 적용**하므로, "이 테이블 생성", "이 컬럼 추가" 같은 단위로 관리되어 `video_tasks` 삭제 같은 문제가 발생하지 않음.


### 요청 폭포 문제 

``` tsx
const revenue = await fetchRevenue();        // 3초 대기 (시작)
const latestInvoices = await fetchLatestInvoices(); // 앞의 3초가 끝나야 시작
const cardData = await fetchCardData();      // 앞의 모든 작업이 끝나야 시작
```
- 문제점: 데이터 요청이 서로 차단하여 요청 폭포를 만들고 있음. 
- 해결: 각각을 `await` 하지 말고 `Promise.all`로 묶기, 컴포넌트를 분리하기 


### 정적 및 동적 렌더링

- 정적 렌더링 (**Next.js 운영 환경**): 빌드할때 모든 데이터를 가져와서 캐시하고 배포, 더 이상 데이터는 바뀌지 않음.
	- 빠른 웹사이트 - 미리 렌더링된 콘텐츠는 Vercel과 같은 플랫폼에 배포될 때 캐시되고 전역적으로 빠르게 배포될 수 있음.
	- 서버 부하 감소 - 콘텐츠가 캐시되기 때문에 서버는 각 사용자 요청에 대해 콘텐츠를 동적으로 생성할 필요가 없음. (컴퓨팅 비용 절약)
	- 검색 엔진 크롤러가 색인하기 쉬운 프리렌더링 콘텐츠는 페이지가 로드될 때 이미 콘텐츠가 제공되기 때문에 검색 엔진 순위 향상

- 동적 렌더링 (**Next.js 개발 환경**, **수동 설정**)
	- 실시간 데이터 - 동적 렌더링을 통해 애플리케이션이 실시간 또는 자주 업데이트되는 데이터를 표시할 수 있음. 데이터가 자주 변경되는 애플리케이션에 이상적
	- 사용자 특정 콘텐츠 - 개인화된 콘텐츠, 예를 들어 대시보드나 사용자 프로필을 제공하고 **사용자 상호작용에 따라 데이터를 업데이트하는 것이 더 쉬워짐.**
	- 요청 시간 정보 - **동적 렌더링을 통해 요청 시에만 알 수 있는 정보, 예를 들어 쿠키나 URL 검색 매개변수에 접근 가능**


- 동적 렌더링의 경우, 새로운 데이터가 필요하면 RSC 페이로드를 서버에서 새로 계산해서 내려주고, 아닌 경우에는 Next.js가 똑똑하게 캐싱을 하든 해서 효율적으로 운영됨. 
- 동적 렌더링을 유발하는 문법
	1. **`searchParams`**: (Page 컴포넌트의 props)
	2. **`cookies()`**
	3. **`headers()`**
	4. **`fetch(..., { cache: 'no-store' })`** (혹은 `force-cache`가 아닌 다른 동적 설정들)
	5. **`unstable_noStore()`**: 특정 컴포넌트 안에서 "이건 캐싱하지 말고 동적으로 처리해"라고 선언할 때 사용 (`data.ts` 파일 등에서 쓰이곤 함.)
	6. `auth()`: 내부적으로 `cookies()`를 사용함. 

일반적으로 **해당 문법을 사용하면 해당 페이지(혹은 레이아웃 범위) 전체를 동적 렌더링으로 바꿔버림.** 
따라서 새로운 요청이 있으면 **동적 렌더링이 끝날때까지 해당 컴포넌트나 페이지 전체가 그려지지 않음.** 
이것을 피하고 싶다면 (즉, 진짜 필요한 부분만 동적으로 남기고 싶다면), `Suspence`와 `PPR` 기능을 조합해서 사용해 함. 


### 스트리밍

- 스트리밍은 서버에서 클라이언트로 데이터를 전송하는 기술, 라우트를 더 작은 "청크"로 나누고 이들이 준비될 때 점진적으로 스트리밍할 수 있게 함. 
- 스트리밍을 통해 느린 데이터 요청이 전체 페이지를 차단하는 것을 방지할 수 있음. 
- 이를 통해 사용자는 모든 데이터가 로드될 때까지 기다릴 필요 없이 페이지의 일부를 볼 수 있고 상호작용할 수 있음.
- 스트리밍은 애초에 **동적 렌더링일 때 페이지가 보이지 않는 현상을 방지**하기 위해서 사용하는 것임. 

##### 전체 페이지 스트리밍

- 폴더에 `loading.tsx`를 만들면 해당 폴더와 그 하위 폴더에 있는 모든 `page.tsx`에서 데이터를 불러오는 동안 `loading.tsx`에 정의된 전체 스켈레톤을 보여주게 됨. 
- `loading.tsx`는 내부적으로 `<Suspense>` 을 자동으로 생성함.
- URL 경로에 포함되지 않는 괄호로 감싼 폴더를 만들어서 특정 `page.tsx`에서만 `loading.tsx`가 지정되도록 할수도 있음.

``` tsx
// app/dashboard/(overview)/loading.tsx
import DashboardSkeleton from '@/app/ui/skeletons';

export default function Loading() {
  // 로딩 스켈레톤
  return <DashboardSkeleton />;
}
```


##### 구성 요소 스트리밍 Suspense

- `<Suspense>`를 활용하여 특정 구성 요소를 세밀하게 스트리밍할 수 있음. 
- 동적 렌더링 컴포넌트를 `<Suspense>`로 감싸면 해당 컴포넌트만 렌더링 되기까지는 스켈레톤을 보여주고 나머지 페이지는 보여줌. 
- 데이터 가져오기를 필요로 하는 컴포넌트로 내려보내고 그 컴포넌트를 `<Suspense>`로 감싸는 것이 일반적으로 좋은 방법

```tsx
export default async function Page() {
  return (
    <main>
      <h1 className={`${lusitana.className} mb-4 text-xl md:text-2xl`}>
        Dashboard
      </h1>
      <div className="grid gap-6 sm:grid-cols-2 lg:grid-cols-4">
        {/* 로딩 중에 보여줄 로딩 스켈레톤을 fallback 속성으로 전달 */}
        <Suspense fallback={<CardsSkeleton />}>
          <CardWrapper />
        </Suspense>
      </div>
      <div className="mt-6 grid grid-cols-1 gap-6 md:grid-cols-4 lg:grid-cols-8">
        <Suspense fallback={<RevenueChartSkeleton />}>
          <RevenueChart />
        </Suspense>
        <Suspense fallback={<LatestInvoicesSkeleton />}>
          <LatestInvoices />
        </Suspense>
      </div>
    </main>
  );
}
```


##### Suspense 작동 방식

``` tsx
// page.tsx
export default async function Page(props: {
  searchParams?: Promise<{
    query?: string;
    page?: string;
  }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || "";
  const currentPage = Number(searchParams?.page) || 1
  const totalPages = await fetchInvoicesPages(query);

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      
      <div className="mt-5 flex w-full justify-center">
        <Pagination totalPages={totalPages} />
      </div>
    </div>
  );
}
```
0. 우선 `searchParams`를 사용하므로 동적 렌더링 대상이 됨. 동적 렌더링 컴포넌트인 `Table`를 `<Suspense>`로 감싸줌. 
1. 서버: `await fetchInvoicesPages`가 끝날 때까지 렌더링 결과물 전송을 **일시 중지(Blocking)** 함.
2. 서버: `await`가 끝나면 **주요 뼈대(Page의 나머지 부분)** 와 **`Table` 자리에 들어갈 스켈레톤**을 한 묶음으로 묶어서 브라우저로 보냄. 
3. 브라우저: 서버에서 온 이 묶음을 받자마자 기존 요소들과 비교하여 화면을 갱신, 이때 **`Table` 영역은 즉시 스켈레톤으로 바뀜**
4. 서버 (백그라운드): 그와 동시에 서버 내부에서는 `Table` 컴포넌트 안의 비동기 로직을 계속 수행하고 있음. (동적 렌더링)
5. 브라우저 (대기): 브라우저 내부의 React는 `Table`이 올때까지 스켈레톤을 보여줌. 
6. 완료: 서버에서 `Table` 렌더링이 정말로 완료되어 추가 데이터를 쏴주면, 브라우저는 대기를 해제하고 진짜 테이블을 화면에 넣어줌.

##### Suspense의 key 속성

-  `key`가 없을 때 (지연 업데이트)	
	Next.js의 탐색(Link 클릭, router.replace 등)은 기본적으로 **Transitions**라는 기술을 사용함. 
	1. URL 변경: 브라우저 주소창이 바뀜. 
	2. 서버 요청: 서버에서 새로운 페이로드를 가져오는 동안, React는 **"기존 화면(구버전 Table)"을 그대로 유지** (화면이 멈춘 것처럼 보일 수 있음)
	3. 데이터 도착: 새로운 테이블 데이터가 도착
	4. 비교 및 교체: 브라우저가 기존 테이블과 새 테이블을 비교해서 쓱 바꿈.
	5. 결과: **스켈레톤은 거의 보이지 않거나**, 아주 찰나의 순간에만 보일 가능성이 높음.

-  `key`가 있을 때 (즉각적인 스켈레톤)
	1. URL 변경: 브라우저 주소창이 바뀝니다.
	2. 서버 응답 시작(Shell): 서버에서 "껍데기와 **새로운 key를 가진 Suspense**" 정보가 도착
	3. 즉시 교체: 브라우저는 "오? `key`가 바뀌었네? 그럼 기존 테이블은 아예 필요 없구나!"라며 **기존 테이블을 즉시 지워버림.**
	4. 스켈레톤 표시: 지워진 자리에 진짜 테이블은 아직 서버에서 오고 있으므로, React는 `key`와 연결된 **스켈레톤(fallback)을 즉시 화면에 띄움.**
	5. 새 데이터 도착 및 교체: 나중에 진짜 테이블 데이터가 도착하면 스켈레톤과 교체합니다.






### 검색 예제 useRouter

1. Search 컴포넌트에서 입력을 받아서 `useRouter`의 `replace`로 `?query=입력값` 경로로 이동시킴
2. Page 컴포넌트에서 `searchParams`로 쿼리 파라미터를 받음. 
3. Page 컴포넌트는 동적 렌더링 대상이므로 Table 컴포넌트가 현재 쿼리 파라미터를 기준으로 다시 렌더링 됨. 
4. Table 컴포넌트의 요청이 완료될 때까지 스켈레톤이 보이다가 완료되면 새로운 데이터가 보임. 

``` tsx
// page.tsx
export default async function Page(props: {
  searchParams?: Promise<{
    query?: string;
    page?: string;
  }>;
}) {
  const searchParams = await props.searchParams;
  const query = searchParams?.query || "";
  const currentPage = Number(searchParams?.page) || 1
  const totalPages = await fetchInvoicesPages(query);

  return (
    <div className="w-full">
      <div className="flex w-full items-center justify-between">
        <h1 className={`${lusitana.className} text-2xl`}>Invoices</h1>
      </div>
      <div className="mt-4 flex items-center justify-between gap-2 md:mt-8">
        <Search placeholder="Search invoices..." />
        <CreateInvoice />
      </div>
      <Suspense key={query + currentPage} fallback={<InvoicesTableSkeleton />}>
        <Table query={query} currentPage={currentPage} />
      </Suspense>
      <div className="mt-5 flex w-full justify-center">
        <Pagination totalPages={totalPages} />
      </div>
    </div>
  );
}

// search.tsx
// 클라이언트 컴포넌트 선언
"use client";

import { MagnifyingGlassIcon } from "@heroicons/react/24/outline";
import { usePathname, useSearchParams, useRouter } from "next/navigation";
import { useDebouncedCallback } from "use-debounce";


export default function Search({ placeholder }: { placeholder: string }) {
  // 현재 URL의 파라미터를 객체 형태로 받음. 
  const searchParams = useSearchParams();
  // 현재 URL의 경로 이름을 받음. 
  const pathname = usePathname();
  // useRouter() replace를 가져옴. replace는 필요한 부분만 변경하기 때문에 브라우저 전체가 다시 로드되지 않음. 
  const { replace } = useRouter();

  // 매 키 입력마다 새로운 DB 요청이 발생하기 때문에 useDebouncedCallback을 사용해서 0.3초의 지연을 줌. 
  const handleSearch = useDebouncedCallback((term) => {
    const params = new URLSearchParams(searchParams);
    params.set('page', '1');
    if (term) {
      params.set('query', term);
    } else {
      params.delete('query');
    }
    replace(`${pathname}?${params.toString()}`);
  }, 300);
  return (
    <div className="relative flex flex-1 flex-shrink-0">
      <label htmlFor="search" className="sr-only">
        Search
      </label>
      <input
        className="peer block w-full rounded-md border border-gray-200 py-[9px] pl-10 text-sm outline-2 placeholder:text-gray-500"
        placeholder={placeholder}
        onChange = {(e) => {handleSearch(e.target.value);}}
        // 처음 생길때 input의 값을 ?query의 값으로 함
        defaultValue={searchParams.get('query')?.toString()} 
      />
      <MagnifyingGlassIcon className="absolute left-3 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-gray-500 peer-focus:text-gray-900" />
    </div>
  );
}
```
- `searchParams`: URL 주소 뒤에 붙는 `?query=abc&page=1` 같은 값들을 객체 형태로 받아옴. Promise를 사용해야 함. 
- `useRouter()`: 페이지를 이동시키거나 주소를 변경함.
	- **클라이언트 컴포넌트에서만 사용 가능**
	- push: 새로운 주소를 브라우저 기록(History)에 **추가** (뒤로 가기 버튼을 누르면 이전 검색어로 돌아감)
    - replace: 현재 주소를 새로운 주소로 **덮어씌웁니다.** 새로고침이 일어나지 않음. 똑똑하게 필요한 부분만 갱신함. (검색어를 한 글자 칠 때마다 기록이 남으면 뒤로 가기를 수십 번 눌러야 하므로, 보통 검색창에서는 기록을 남기지 않는 `replace`를 선호)
- `defaultValue`: 처음 생성될때 값을 지정함. 
- `value`는 계속 값을 추적함. 
- `useDebouncedCallback(콜백함수, 타이머)`: 검색창에 
	1. 이벤트가 시작하면 타이머가 시작됨. (예: onChange)
	2. 새로운 이벤트가 발생하면 타이머가 재설정 됨. 
	3. 타이머가 카운트다운을 끝내면 콜백함수를 실행함.


### 서버 액션

- **클라이언트에서 직접 호출할 수 있는 서버 전용 비동기 함수**
- DB 조작, 외부 API 연동, 파일 시스템 작업
	- 이런 작업들을 브라우저에서 처리하도록 하면 위험함. 
	- DB 주소나 토큰 같은 설정값들은 환경변수로 서버에서 관리하고 이런 작업들을 하는 것이 보안상 안전함. 
- 캐시 및 UI 동기화: `revalidatePath`나 `revalidateTag`를 호출해 서버/클라이언트 캐시를 즉시 갱신하여, **DB 변경 사항이 화면에 바로 반영**되도록 제어

##### 기본 사용

1. actions 정의: form에서 submit 발생 시 실행할 작업 정의 
``` tsx
// lib/actions.ts

// 서버 액션을 만들 때 사용하는 특수 지시어
'use server';

import { z } from "zod";
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import postgres from "postgres";

const sql = postgres(process.env.POSTGRES_URL!, { ssl: "require" });

// 데이터 검증 준비 
const FormSchema = z.object({
  id: z.string(),
  customerId: z.string(),
  amount: z.coerce.number(),
  status: z.enum(["pending", "paid"]),
  date: z.string(),
});

// id, date는 검증에서 제외
const CreateInvoice = FormSchema.omit({ id: true, date: true });

// action은 기본적으로 form의 입력값들을 FormData로 받아옴.
export async function createInvoice(formData: FormData) {
  // 데이터 검증 작업 
  const { customerId, amount, status } = CreateInvoice.parse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });
  const amountInCents = amount * 100;
  const date = new Date().toISOString().split("T")[0];

  await sql`
    INSERT INTO invoices (customer_id, amount, status, date)
    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})
  `;
  
  // revalidatePath로 캐시를 삭제해 서버에서 새로운 요청을 하도록 해야함. 
  revalidatePath('/dashboard/invoices');
  redirect('/dashboard/invoices');
}
```

- `revalidatePath('/dashboard/invoices')`를 호출하는 순간, 해당 경로에 대해서 2가지 일을 동시에 처리함.
	1. **서버 캐시 파기**: "너가 빌드해둔 옛날 정적 렌더링 HTML 캐시 버리고 새로 그려!"
	2. **클라이언트 캐시 파기 지시**: 클라이언트(브라우저)에게도 신호를 보내서 "너가 메모리에 들고 있던 라우터 캐시 내용 당장 다 지워! 다음부터 무조건 나(서버)한테 새로운 화면 데이터 달라고 꼭 요청해!"
- `revalidateTag()`: `revalidatePath`와 동일한 기능이지만 캐싱된 특정 데이터를 태그로 지정함. 
- 이렇게 함으로써 정적 렌더링 컴포넌트여도 캐시가 없기 때문에 서버에서 새로 렌더링한(서버 액션으로 업데이트 된) 컴포넌트를 브라우저로 내려줌. 

2. action 지정
	- `<form action={formAction}>`에서 action을 지정
	- 일반적인 비동기 함수처럼 바로 호출: 서버 액션(`"use server"`가 붙은 함수)은 사실 내부적으로 보이지 않는 HTTP POST 요청(API 엔드포인트)으로 자동 변환되기 때문에 비동기 함수를 부르듯이 호출이 가능함. 


##### `bind()`

- 서버 액션은 기본적으로 폼 데이터만 `FormData`로 받아오는데 직접 `bind()`로 데이터를 넣어줄 수 있음. 
``` tsx
// edit-form.tsx
const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);
return (
	<form action={updateInvoiceWithId}>
```

``` tsx
// actions.ts
// FormData 외에 직접 바인딩한 id를 받아서 사용 가능
export async function updateInvoice(id: string, formData: FormData) { ... }
```


##### `useActionState`

- 서버 액션의 실행 결과와 상태를 관리하는 UI 매니저
- 서버에서 보내온 데이터(에러 메시지나 성공 결과)를 클라이언트의 변수(State)에 자동으로 담아주는 역할
- 리액트 훅이므로 클라이언트 컴포넌트로 변환해야함. 

``` tsx
// form.tsx
'use client'

const initialState: State = { message: null, errors: {} };

const [state, formAction, isPending] = useActionState(createInvoice, initialState);
```
- `createInvoice` (실행할 함수): 실제로 서버에서 돌아갈 액션 함수
- `initialState` (초기값): 폼을 처음 열었을 때 가질 데이터

- `state` (현재 서버의 대답): 서버 액션이 실행되고 리턴한 결과값이 담김. 처음에는 initialState가 담겨있음. 
- `formAction` (액션 대행자): 기존의 `createInvoice`를 대신해서 `<form action={...}>`에 넣어줄 업그레이드된 함수
- `isPending`: `formAction`이 서버에서 실행 중(요청 중)인 동안 자동으로 `true`가 됨. 

``` ts
// actions.ts
export async function createInvoice(prevState: State, formData: FormData) { ... }
```
- 첫번째 인자는 현재의 상태를 받음. (여기서는 `initialState`)
- 두번째 인자는 그대로 `FormData`를 받음. 


### 데이터 검증 Zod

``` tsx
// actions.ts
import { z } from "zod";
...

const FormSchema = z.object({
  id: z.string(),
  customerId: z.string({invalid_type_error: 'Please select a customer.',}),
  amount: z.coerce.number().gt(0, { message: 'Please enter an amount greater than $0.' }),
  status: z.enum(["pending", "paid"], {invalid_type_error: 'Please select a status.',}),
  date: z.string(),
});

export type State = {
  errors?: {
    customerId?: string[];
    amount?: string[];
    status?: string[];
  };
  message?: string | null;
};

const CreateInvoice = FormSchema.omit({ id: true, date: true });
const UpdateInvoice = FormSchema.omit({ id: true, date: true });


export async function createInvoice(prevState: State, formData: FormData) {
  const validatedFields = CreateInvoice.safeParse({
    customerId: formData.get("customerId"),
    amount: formData.get("amount"),
    status: formData.get("status"),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      message: 'Missing Fields. Failed to create invoice.',
    };
  }
  
  const { customerId, amount, status } = validatedFields.data;
  // 성공 로직 ...
```

1. 스키마 정의: `z.object({ ... })`로 어떤 데이터가 들어와야 하는지 설계도 작성
2. 검증 실행: `safeParse()`로 데이터 검사, 데이터 검증 시 에러가 발생해도 즉시 예외를 던지지 않고 성공 여부를 담은 객체를 반환
	-  성공 시: `{ success: true, data: { ... } }` 반환
	- 실패 시: `{ success: false, error: ZodError }` 반환

3. 결과 처리
	- 성공: `validatedFields.data`에서 깨끗하게 정제된 데이터를 꺼내서 DB 작업 수행
	- 실패: `flatten().fieldErrors`로 필드 에러들을 뽑아서 클라이언트로 반환 (UI에 에러 표시)

- 이때 `ZodError`는 내부 구조가 매우 복잡한데 `flatten().fieldErrors`를 사용하면 **아주 깔끔한 딕셔너리 형태**로 펴줌.
	``` tsx
	// flatten() 결과 예시
	{
	  customerId: ['Please select a customer.'],
	  amount: ['Please enter an amount greater than $0.'],
	  status: ['Please select a status.']
	}
	```






### 동적 경로 세그먼트

1. 폴더 이름을 대괄호로 감싸서 생성
	- `/dashboard/invoices/${id}/edit`의 페이지를 만들고 싶은 경우 `/invoices/[id]/edit/page.tsx` 파일을 만들면 됨.
2. `page.tsx`에서 `params`를 사용하면 동적 세그먼트(경로 파라미터)를 받을 수 있음.
``` tsx
// page.tsx

// /dashboard/invoices/[id]/edit에서 id를 params로 받을 수 있음. (동적 세그먼트)
export default async function Page(props: { params: Promise<{ id: string }> }) {
  const params = await props.params;
  const id = params.id;
  const [invoice, customers] = await Promise.all([
    fetchInvoiceById(id),
    fetchCustomers(),
  ]);

  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: 'Invoices', href: '/dashboard/invoices' },
          {
            label: 'Edit Invoice',
            href: `/dashboard/invoices/${id}/edit`,
            active: true,
          },
        ]}
      />
      <Form invoice={invoice} customers={customers} />
    </main>
  );
}
```

- 동적 세그먼트의 렌더링
	- 기본적으로 `[id]`는 사용자가 어떤 주소로 접속할지 빌드할 때는 알 수 없기 때문에 접속할 때 렌더링하는 **동적 렌더링(Dynamic Rendering)** 으로 동작하는 것이 기본
	- **예외**: `generateStaticParams` 라는 함수를 사용하면, "내 송장 아이디는 무조건 1, 2, 3번뿐이야" 라고 빌드 타임에 미리 Next.js에게 알려줄 수 있음. 이렇게 하면 동적 세그먼트임에도 불구하고 정적 렌더링으로 만들어 버릴 수도 있음.


### 오류 처리 

##### 공통 오류 처리

하위 세그먼트의 에러는 트리 구조를 타고 상위로 전파되며, **가장 가까운(Nearest)** `error.tsx`가 이를 포착함. (에러 버블링)

``` tsx
// error.tsx

'use client';
 
import { useEffect } from 'react';
 
export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    console.error(error);
  }, [error]);
 
  return (
    <main className="flex h-full flex-col items-center justify-center">
      <h2 className="text-center">Something went wrong!</h2>
      <button
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
        onClick={
          // 에러가 발생한 경로를 다시 시도
          () => reset()
        }
      >
        Try again
      </button>
    </main>
  );
}
```
- 클라이언트 컴포넌트여야 함. 
- props로 `error`, `reset`을 고정적으로 받고 타입도 Next.js가 지정한 타입으로 명시
- `error`: 발생한 실제 에러 정보가 담긴 자바스크립트 표준 `Error` 객체, `error.message`를 까보면 에러 원인을 알 수 있음. 
	- 추가로 붙어있는 `digest` 속성은 짧은 문자열(고유 아이디)인데, 서버 내부의 민감한 에러 내역을 방문자에게 다 보여줄 수 없기 때문에 Next.js가 에러마다 고유 번호를 매겨서 던져주고 나중에 개발자가 서버 에러 로그에서 이 번호를 검색해 진짜 원인을 찾을 수 있음.
- `reset`: 에러가 발생했던 경로(Route)를 다시 그려보라고(재시도) 지시하는 함수

##### 404 에러 처리

`notFound()` 함수를 실행하면 가장 가까운 `not-found.tsx`로 매핑이 됨. 

``` tsx
// not-found.tsx

import Link from 'next/link';
import { FaceFrownIcon } from '@heroicons/react/24/outline';
 
export default function NotFound() {
  return (
    <main className="flex h-full flex-col items-center justify-center gap-2">
      <FaceFrownIcon className="w-10 text-gray-400" />
      <h2 className="text-xl font-semibold">404 Not Found</h2>
      <p>Could not find the requested invoice.</p>
      <Link
        href="/dashboard/invoices"
        className="mt-4 rounded-md bg-blue-500 px-4 py-2 text-sm text-white transition-colors hover:bg-blue-400"
      >
        Go Back
      </Link>
    </main>
  );
}
```


### Next.js 에러 처리 방식

| 방식               | 추천 상황              | 특징                   |
| ---------------- | ------------------ | -------------------- |
| 직접 try-catch     | 소규모, 빠른 프로토타이핑     | 단순하지만 반복적임           |
| Wrapper 함수       | 외부 라이브러리 사용을 지양할 때 | 깔끔하고 커스터마이징이 자유로움    |
| next-safe-action | 중대형 프로젝트, 협업       | **현재 가장 권장되는 표준 패턴** |

##### 1. `try-catch`

가장 직관적인 방법입니다. 소규모 프로젝트나 학습 단계에서는 이 방식이 가장 많이 쓰입니다.
- **특징:** 매번 `actions.ts`의 함수마다 `try-catch`를 작성합니다.
- **장점:** 흐름을 한눈에 알 수 있습니다.
- **단점:** 코드가 중복되고, 깜빡하고 `catch`를 안 쓰면 민감한 서버 에러가 클라이언트에 그대로 노출될 위험이 있습니다.
    
``` ts
// actions.ts
export async function updateProfile(data: FormData) {
  try {
    await db.user.update(...);
    revalidatePath('/');
    return { success: true };
  } catch (e) {
    return { success: false, message: "업데이트에 실패했습니다." };
  }
}
```

##### 2. Wrapper 함수 사용

반복되는 `try-catch`를 줄이기 위해 로직을 감싸는 **Wrapper 함수**를 직접 만들어 사용하는 방식입니다. 많은 팀에서 내부적으로 사용하는 '커스텀 국룰' 패턴입니다.
- **특징:** 에러 처리 로직을 한곳에서 관리합니다.
- **장점:** `actions.ts` 코드가 매우 깔끔해집니다. 공통 로깅(Sentry 등)을 붙이기도 쉽습니다.

``` ts
// lib/action-wrapper.ts
export const actionHandler = (fn: Function) => async (...args: any[]) => {
  try {
    return await fn(...args);
  } catch (error) {
    console.error(error); // 서버 로그 기록
    return { error: "서버 내부 오류가 발생했습니다." };
  }
};

// actions.ts
export const updateProfile = actionHandler(async (data: FormData) => {
  // try-catch 없이 로직만 작성
});
```

##### 3. `next-safe-action` 라이브러리

현재 Next.js 생태계에서 가장 추천되는 '국룰' 라이브러리는 `next-safe-action`입니다.
- **특징:** 유효성 검사(Zod)와 에러 처리를 하나의 파이프라인으로 묶어줍니다.
- **장점:**
    - **Type-safe:** 입력값과 결과값의 타입을 완벽하게 보장합니다.
    - **에러 자동 분류:** 입력값 오류(Validation Error)와 서버 내부 오류(Server Error)를 명확히 구분해서 클라이언트에 전달합니다.
    - **낙관적 업데이트(Optimistic UI):** 지원이 강력합니다.

``` ts
// lib/safe-action.ts
export const action = createSafeActionClient({
  // 여기서 서버 에러 발생 시 공통 로깅과 클라이언트로 내려줄 에러 메세지를 1회만 정의합니다.
  handleServerError: (error) => {
    console.error("[글로벌 서버 에러]:", error);
    return "서버에서 문제가 발생했습니다. 잠시 후 다시 시도해주세요."; 
  },
});

// actions.ts
// try-catch가 아예 없습니다. 비즈니스 로직만 남습니다.
export const getVideoById = actionClient
  .schema(z.string()) // 1. 입력값이 문자열인지 검증
  .action(async ({ parsedInput: id }) => {
    // 2. 순수 비즈니스 로직
    const video = await dbGetVideoById(id);
    return video || null; 
  });
```


### Next.js 에러 처리 구조

현재 생각하신 "data.ts는 던지고, actions.ts와 컴포넌트에서 잡는다"는 전략은 매우 합리적입니다. 하지만 각 단계별 역할 분담을 조금 더 명확히 하면 더 견고한 코드가 됩니다.

1.  `data.ts` (Throwing Phase)
	여기서는 에러를 **잡지 않고 그대로 던지는 것**이 일반적입니다.

	- **이유:** `data.ts`를 어디서 호출할지 모르기 때문입니다. 특정 액션에서 호출할 수도 있고, 서버 컴포넌트(RSC)에서 직접 호출할 수도 있죠.
	- **팁:** 필요하다면 로우 레벨 에러(DB 연결 오류 등)를 '애플리케이션 전용 에러'로 감싸서(Wrap) 던져주면 나중에 디버깅하기 편합니다.
    

2.  `actions.ts` (Boundary Phase)
	서버 액션은 클라이언트(브라우저)와 서버가 만나는 **경계점**입니다. 여기서 에러를 제대로 처리하지 않으면 클라이언트에서 런타임 에러가 발생하거나 민감한 DB 정보가 브라우저 콘솔에 찍힐 수 있습니다.

	- **역할:** `try-catch`를 사용하여 에러를 잡고, 사용자에게 보여줄 **안전한 메시지**로 변환해야 합니다.
	- **반환 형태:** 보통 `{ success: true, data: ... }` 또는 `{ success: false, message: "입력값이 올바르지 않습니다." }` 형태의 객체를 반환하는 것이 표준입니다.
    
3.  컴포넌트 (UI Phase)
	액션이 반환한 결과(성공 여부/메시지)를 바탕으로 사용자에게 피드백을 줍니다.
	
	- **방법:** `useActionState`(또는 이전 버전의 `useFormState`)를 사용하면 액션의 결과와 에러 메시지를 React 상태처럼 편하게 관리할 수 있습니다.

### 접근성 검사 ESLint

1. 패키지 설치: `pnpm add -D eslint eslint-config-next`
2. `eslint.config.mjs` 설정 파일 생성
	``` js
	// eslint.config.mjs
	import { defineConfig, globalIgnores } from 'eslint/config';
	import nextVitals from 'eslint-config-next/core-web-vitals';
 
	const eslintConfig = defineConfig([
	  ...nextVitals,
	  globalIgnores(['.next/**', 'out/**', 'build/**', 'next-env.d.ts']),
	]);
 
	export default eslintConfig;
	```
	- `core-web-vitals`가 하는 일
		- 구글(Google)은 웹사이트의 성능을 측정하는 3가지 핵심 지표를 **Core Web Vitals**라고 함. 
		- 이 설정은 이 지표를 망가뜨릴 수 있는 나쁜 습관을 찾아냄.  
		- **LCP (로딩 속도):** "너무 큰 이미지를 그냥 쓰고 있진 않니? `next/image`를 써서 최적화해!"
		- **FID (응답성):** "브라우저가 일을 너무 많이 하게 해서 클릭이 안 먹히게 만들면 안 돼!"
		- **CLS (시각적 안정성):** "이미지 크기를 안 정해두면 나중에 이미지가 뜨면서 글자가 아래로 툭 떨어지잖아. 이거 고쳐!"

3. `lint` 스크립트를 `pagkage.json` 파일에 추가
4. 실행: `pnpm lint`
	1. 기본 문법 및 잠재적 버그 검사 (Standard ESLint)
		- 사용하지 않는 변수 (`const x = 10;` 해놓고 안 쓸 때)
		- 오타나 잘못된 문법    
		- `useEffect`에서 의존성 배열을 빼먹었을 때
	2. Next.js 권장 사례 검사 (Next.js Specific Rules)
		- `<a>` 태그 대신 `<Link>`를 쓰라고 제안함
		- `<img>` 태그 대신 `<Image>`를 쓰라고 경고함
		- 폰트나 스크립트를 비효율적으로 불러오면 알려줌


### 웹 접근성 

``` tsx
          <div className="relative">
            <select
              id="customer"
              name="customerId"
              className="peer block w-full cursor-pointer rounded-md border border-gray-200 py-2 pl-10 text-sm outline-2 placeholder:text-gray-500"
              defaultValue=""
              // 이 select 태그에서 에러가 발생하면 id=customer-error인 태그를 음성메세지로 읽어줌. 
              aria-describedby="customer-error" 
            >
            
		  {/* ... */}
		  
          <div id="customer-error" aria-live="polite" aria-atomic="true">
            {state.errors?.customerId &&
              state.errors.customerId.map((error: string) => (
                <p className="mt-2 text-sm text-red-500" key={error}>
                  {error}
                </p>
              ))}
          </div>
```
- `aria-describedby="customer-error"` : 이는`select` 요소와 오류 메시지 컨테이너 간의 관계를 설정. 이는 `select` 요소에 포커스를 하면 `id="customer-error"` 의 오류 메세지가 들리는 것을 뜻함. 
- `aria-live="polite"`: 에러가 화면에 나타나는 순간 자동으로 음성메세지를 읽어줌. 
- `aria-atomic="true"`: 에러창 안에 글자가 바뀌면 전체 내용을 다시 읽어줌. 



### 인증

##### 기본 사용

NextAuth.js는 세션 관리, 로그인 및 로그아웃, 그리고 인증의 다른 측면을 관리하는 라이브러리

1. 패키지 다운로드: `pnpm i next-auth@beta` 
2. `.env` 파일에 `AUTH_SECRET`을 추가 (개발 환경), Vercel에서는 따로 환경변수를 추가해줘야함 (운영환경)
	- next-auth는 내부적으로 **로그인에 성공한 경우 해당 사용자 정보(이름, 이메일 등)와 `AUTH_SECRET`를 합쳐서 쿠키를 내려주고 다음 인증때부터는 이 쿠키를 활용하여 사용자를 인증함.** 
3. `auth.config.ts`: NextAuth 설정, next-auth는 NextAuth를 사용하는데 거기에 사용할 설정
``` ts
// auth.config.ts
import type { NextAuthConfig } from 'next-auth';
 
export const authConfig = {
  // 1. 로그인 페이지 설정: NextAuth가 인증 실패한 사용자를 자동으로 이동시킬 로그인 페이지 경로 지정
  pages: {
    signIn: '/login',
  },
  
  // 2. 인증 콜백: NextAuth의 함수들을 설정
  callbacks: {
    // auth 함수 설정: true를 반환하면 접근 허용, false를 반환하면 로그인을 요구함. 
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL('/dashboard', nextUrl));
      }
      return true;
    },
  },
  
  // 3. 로그인(인증) 수단 설정, signIn 함수의 인증 로직을 설정 
  // 각 로그인 방식마다 특정한 규칙이 있음. (구글 로그인, 깃허브, DB의 비번 등)
  providers: [], 
} satisfies NextAuthConfig;
```

4. `proxy.ts`: 인증 프록시 설정, `auth.config.ts`의 기본설정들을 이용한 인증을 프록시에 적용
``` ts
// proxy.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';
 
// proxy.ts에서는 export default된 함수가 해당 페이지로 이동 전에 실행됨. (프록시)
// proxy라는 이름의 함수도 실행됨.
// auth.config.ts에서 정의한 설정을 넣은 NextAuth의 auth를 실행하도록 설정함. (인증)
export default NextAuth(authConfig).auth;
 
// config에서 어떤 경로만 이 프록시를 적용할지 지정할 수 있음. 
export const config = {
  matcher: ['/((?!api|_next/static|_next/image|.*\\.png$).*)'],
};
```

5. `auth.ts`: 기존 `auth.config.ts`에서 Provider를 추가하여 `signIn` 함수를 정의함.
``` ts
// auth.ts
import NextAuth from 'next-auth';
import Credentials from 'next-auth/providers/credentials';
import { authConfig } from './auth.config';
import { z } from 'zod';
import type { User } from '@/app/lib/definitions';
import bcrypt from 'bcrypt';
import postgres from 'postgres';
 
const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });
 
async function getUser(email: string): Promise<User | undefined> {
  try {
    const user = await sql<User[]>`SELECT * FROM users WHERE email=${email}`;
    return user[0];
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw new Error('Failed to fetch user.');
  }
}

export const { auth, signIn, signOut } = NextAuth({
  // 아까 만든 auth.config.ts에서 Provider만 추가함. (signIn의 인증 로직 추가됨.)
  ...authConfig,
  providers: [
    // 로그인으로 내 DB의 비번을 쓰고 싶을때 Credentials 사용
    // signIn('credentials', formData);로 해당 로그인 방식을 사용 가능. 
    Credentials({
      async authorize(credentials) {
        const parsedCredentials = z
          .object({ email: z.string().email(), password: z.string().min(6) })
          .safeParse(credentials);
 
        if (parsedCredentials.success) {
          const { email, password } = parsedCredentials.data;
          const user = await getUser(email);
          if (!user) return null;

          const passwordsMatch = await bcrypt.compare(password, user.password);

          if (passwordsMatch) return user;
        }
 
        return null;
      },
    }),
  ],
});
```

6. 로그인 
- `actions.ts`에 로그인 액션 추가
``` ts
// actions.ts
import { signIn } from "@/auth";
import { AuthError } from "next-auth";

export async function authenticate(
  prevState: string | undefined,
  formData: FormData,
) {
  try {
	// Provider로 추가한 인증 방식을 사용
    await signIn('credentials', formData);
  } catch (error) {
    if (error instanceof AuthError) {
      switch (error.type) {
        case 'CredentialsSignin':
          return 'Invalid credentials.';
        default:
          return 'Something went wrong.';
      }
    }
    throw error;
  }
}
```

- 로그인 폼에서 해당 액션 호출
``` tsx
// login-form.tsx
import { authenticate } from "@/app/lib/actions";
...

export default function LoginForm() {
  // proxy.ts에서 auth.config.ts에서 설정한 authorized가 실행되어서 false가 되면 /login으로 보내는데
  // 이때 원래 가려던 경로를 callbackUrl 쿼리 파라미터로 넣어줌. 
  // 로그인이 성공하면 해당 경로로 리다이렉트를 해주는게 좋음.
  const searchParams = useSearchParams();
  const callbackUrl = searchParams.get("callbackUrl") || "/dashboard";
  const [errorMessage, formAction, isPending] = useActionState(
    authenticate,
    undefined,
  );

  return (
    <form action={formAction} className="space-y-3">
    
    {/* ... */}
    
	<input type="hidden" name="redirectTo" value={callbackUrl} /> 
    <Button className="mt-4 w-full" disabled={isPending} aria-disabled={isPending}>
          Log in <ArrowRightIcon className="ml-auto h-5 w-5 text-gray-50" />
        </Button>
        <div
          className="flex h-8 items-end space-x-1"
          aria-live="polite"
          aria-atomic="true"
        >
          {errorMessage && (
            <>
              <ExclamationCircleIcon className="h-5 w-5 text-red-500" />
              <p className="text-sm text-red-500">{errorMessage}</p>
            </>
          )} 
```
- `<input type="hidden" name="redirectTo" value={callbackUrl} />`: 서버 액션을 호출할때 `FormData`에 `redirectTo`를 담기게 함. 
- `actions.ts`에서 `signIn('credentials', formData)`를 호출
- NextAuth의 `signIn` 함수는 내부적으로 `redirectTo`를 감지하여 인증에 성공하면 해당 주소로 자동 리다이렉트를 시켜줌. 


*로그인 및 인증 흐름*

1. `signIn`을 성공하면 AUTH_SECRET와 사용자의 정보를 합쳐서 암호화한 쿠키를 브라우저에 반환. 
2. 다음 요청이 오면 `proxy.ts`에서 `auth()`가 호출됨.
	- 요청 헤더에서 쿠키를 꺼냄.
	- 환경변수에 저장된 `AUTH_SECRET`을 가져옴.
	- `AUTH_SECRET`을 열쇠삼아 쿠키를 **복호화**
	- 복호화된 내용이 조작되지 않았는지 **대조(검증)**
	- `authorized` 콜백 실행
3. 성공하면 `auth`에 객체가 채워지고 `true`를 반환해서 `auth()`가 통과됨. 
	- 기본적으로 `auth`에는 이름, 이메일, 이미지가 들어있음. 
4. 실패하면 `auth`에 null이 채워지고 `false`를 반환해서 튕겨냄. 



*질문. 왜 `proxy.ts`에서는 `auth.config.ts`만 쓰나요?*

- **이유:** 문지기(`proxy.ts`)는 "사용자가 입력한 비번이 맞는지(bcrypt)"까지 검사할 필요가 없음. 문지기의 역할은 오직 **"이 사람이 이미 로그인해서 쿠키(세션)를 들고 있는가? 그리고 그 사람이 가려는 주소가 권한이 있는가?"** 만 확인하면 됨. 
- 만약 `proxy.ts`에서 `auth.ts`를 가져다 쓰려고 하면, `proxy.ts`가 실행될 때 "나 `bcrypt` 같은 무거운 거 몰라!" 하고 에러를 내며 서버가 멈춤.

정리하자면:
- `proxy.ts`는 가벼운 설정(`auth.config.ts`)만 들고 서서 "로그인 토큰 있네? 들어와!" 혹은 "토큰 없네? 나가!"만 판단
- 실제 로그인 페이지에서 로그인 버튼을 누를 때 비로소 무거운 설정(`auth.ts`)이 가동되어 "입력한 비번이 DB랑 맞나?"를 꼼꼼히 검사


##### 세션 데이터 추가

- NextAuth.js는 기본적으로 쿠키에 사용자 정보를 이름, 이메일, 이미지만 넣어놓음. 
- 사용자 id를 활용해야 할 경우에 세션 데이터를 추가해야함. 

``` ts
// auth.config.ts

import type { NextAuthConfig } from "next-auth";

export const authConfig = {
  // 로그인 페이지 설정
  pages: {
    signIn: "/login",
  },
  // 인증 콜백
  callbacks: {
    // 1. JWT 콜백: 로그인이 성공하거나 토큰이 생성될 때 실행됩니다.
    async jwt({ token, user }) {
      if (user) {
        // 처음 로그인 시점에 user 객체에 들어있는 id를 토큰(쿠키의 원재료)에 저장합니다.
        token.id = user.id;
      }
      return token;
    },
    // 2. Session 콜백: auth()를 호출할 때 최종적으로 반환되는 session 객체를 설정합니다.
    async session({ session, token }) {
      if (token.id) {
        // 위에서 저장한 token.id를 실제 세션 바구니(session.user)로 옮겨줍니다.
        session.user.id = token.id as string;
      }
      return session;
    },
    // 3. auth 콜백
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith("/dashboard");
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // Redirect unauthenticated users to login page
      } else if (isLoggedIn) {
        return Response.redirect(new URL("/dashboard", nextUrl));
      }
      return true;
    },
  },
  // 로그인 수단 설정, signIn 함수 (각 로그인 방식마다 특정한 규칙이 있음. 구글 로그인, 깃허브, DB의 비번 등)
  providers: [],
} satisfies NextAuthConfig;
```

``` ts
// next-auth.d.ts
import NextAuth, { DefaultSession } from "next-auth"

// NextAuth 타입에서 user 타입에 id를 명시해줌. 
// session.user.id 사용 가능
declare module "next-auth" {
  interface Session {
    user: {
      id: string
    } & DefaultSession["user"]
  }
}
```


##### 인증 흐름

1. 로그인 단계
	1. `signIn` 호출
	2. `providers` 로직으로 검증
	3. JWT 콜백 실행
		- 기본적으로 주어지는 `token`(이름, 이메일, 이미지)과 `providers`에서 반환한 `user`를 받을 수 있음.
		- `token.id`에 `user.id`를 넣음. 
	4. 해당 토큰과 `AUTH_SECRET`를 활용하여 암호화한 쿠키를 브라우저로 전송

2. 페이지 접근 단계
	1. `proxy.ts`에서 `auth()`가 호출됨. 
	2. jwt 콜백 실행: `user`가 없으니 쿠키 정보만 유지
	3. session 콜백 실행: 복호화된 토큰의 `id`를 `session.user.id`에 담음. 
	4. authorized 콜백 실행: `auth`에 사용자 정보가 있는지 없는지를 판단하여 접근을 허용/차단함.

3. 내부에서 `auth()` 호출: session 객체에서 user.id를 사용하기 위해서 사용
	- `auth()` 함수는 내부에 요청 단위 캐싱이 되어 있음. 
	- 처음에 프록시가 `auth()`를 실행함으로써 만든 세션 객체를 반환해줌. 

``` ts
// actions.ts
const session = await auth();
const userId = session?.user?.id; // 쿠키에서 가져온 내 ID
...
```

``` json
// 기본적인 session 객체 구조
{
  "user": {
    "name": "홍길동",
    "email": "test@example.com",
    "image": "https://..."
  },
  "expires": "2026-05-21T..." // 세션 만료 시간
}

// 타입 확장 후 session 객체 구조
{
  "user": {
    "id": "uuid-1234-5678",  // <-- 우리가 추가한 것!
    "name": "홍길동",
    "email": "test@example.com",
    "image": "https://..."
  },
  "expires": "2026-05-21T..."
}
```


##### NextAuth.js 공통 테이블 

NextAuth는 사용자 관리를 위해 크게 **2개의 테이블**을 사용하도록 설계되어 있음. 
이 2개의 테이블만 있으면 구글이든 카카오든 애플이든 수십 개의 소셜 로그인을 다 붙여도 테이블을 추가할 필요가 없음.

**1. `User` 테이블 (유저 정보 - 공통)** 사람 자체를 나타내는 테이블, 소셜에서 받아온 정보 중 가장 공통적인 것만 담음.
- `id`
- `name` (이름 또는 닉네임)
- `email` (이메일)
- `image` (프로필 사진)

**2. `Account` 테이블 (소셜 연결 정보 - 공통)** 유저가 어떤 소셜 서비스들을 이용해 가입했는지를 기록하는 테이블
- `userId` (User 테이블의 id를 가리킴)
- `provider` (예: `"google"`, `"kakao"`, `"apple"`)
- `providerAccountId` (그 소셜 서비스 내에서 발급한 고유 ID 값)
- `access_token`, `refresh_token` (소셜 서비스와 통신할 때 쓰는 인증 토큰)

**단 하나의 공통된 `users` 테이블과 `accounts` 테이블** 구조만 만들어 두면 나중에 카카오 로그인도 추가할 때, DB 테이블 구조는 수정 하지 않고 NextAuth 설정 코드에 카카오 키 값만 추가하면 됨.


### 로그 및 모니터링 

- **서버:** 개발자가 직접 `Pino`로 필요한 로그(정보, 경고, 에러 등)를 명시적으로 작성함.
- **클라이언트:** 개발자가 직접 로깅 시스템을 구축하지 않고, `console.log`는 디버깅용으로만 쓰며, `Sentry`가 에러를 자동으로 수집함.
- **통합 뷰어:** `Sentry` 대시보드에서 클라이언트의 자동 에러 로그와 서버의 주요 에러/로그를 한눈에 파악함.

##### 1. 서버 세팅 (Pino)

- `pino`를 설치하고 `lib/logger.ts` 같은 유틸리티 파일을 만듭니다.
- 서버 컴포넌트, Route Handler(API), Server Action에서 `console.log` 대신 이 logger를 가져와 사용합니다.
- **주의점:** Pino 로그는 터미널(stdout)로 출력되므로, 서버(예: AWS, Vercel 등) 환경에 따라 이 출력된 로그들을 모아서 저장해 줄 인프라(AWS CloudWatch, Datadog 등)가 나중에는 필요할 수 있습니다. (하지만 Vercel을 쓴다면 Vercel 대시보드의 Logs 탭에서 Pino가 예쁘게 출력한 JSON 로그를 바로 확인할 수 있어서 매우 편리합니다.)

##### 2. 프론트/풀스택 에러 트래킹 세팅 (Sentry)

- `@sentry/nextjs`를 설치하고 `npx @sentry/wizard@latest -i nextjs` 명령어를 실행하면 Sentry가 프로젝트에 필요한 설정 파일들을 자동으로 세팅해 줍니다.
- 이 세팅이 끝나면 **클라이언트 에러와 서버측 에러를 즉시 자동 수집**됩니다.

*옵션: 수동 기록*
- 만약 Server Action 내부에서 `try-catch` 블록을 사용해 에러를 잡은 뒤, 사용자에게 성공/실패 여부(예: `{ success: false, error: '...' }`)를 객체로 반환하도록 코드를 작성했다면, 이 에러는 **예외로 던져지지 않은 것(handled)**으로 취급되어 Sentry에 자동으로 기록되지 않습니다.
- 이 경우에는 `catch` 블록 내에서 명시적으로 Sentry에 보고해야 합니다.

```ts
import * as Sentry from "@sentry/nextjs";

export async function myServerAction() {
  try {
    // 비즈니스 로직
  } catch (error) {
    // 에러를 수동으로 Sentry에 전송
    Sentry.captureException(error);
    return { success: false, error: "서버 오류가 발생했습니다." };
  }
}

```


### 다국어

1. `next-intl` 패키지를 설치합니다.
2. `messages/` 폴더에 각 언어별 번역 파일(`en.json`, `ko.json` 등)을 생성합니다.
3. `routing.ts`에서 지원하는 언어, 기본 언어 설정
4. 프로젝트 루트에 `i18n/request.ts` 파일을 생성하여 번역 파일을 불러오는 설정을 합니다.
5. `proxy.ts` 파일을 생성하여 사용자의 브라우저 언어를 감지하고 URL에 국가 코드(예: `/ko/about`, `/en/about`)를 붙여주도록 설정합니다.
6. `next.config.mjs`에 `next-intl` 플러그인을 적용합니다.
7. 기존 `app/` 폴더 내의 페이지들을 `app/[locale]/` 폴더 하위로 이동시킵니다.
8. 컴포넌트 내에서 `useTranslations` (클라이언트) 또는 `getTranslations` (서버) 훅을 사용하여 텍스트를 출력합니다.

##### 1단계: `routing.ts` 다국어 라우팅 설정

``` ts
import { defineRouting } from "next-intl/routing";
import { createNavigation } from "next-intl/navigation";

// 지원하는 언어, 기본 언어 설정
export const routing = defineRouting({
  // 지원되는 모든 언어 목록
  locales: ["ko", "en", "ja", "es", "pt", "zh", "de", "fr"],

  // 일치하는 언어가 없을 때 사용할 기본 언어
  defaultLocale: "en",
});

// 지원하는 언어 타입을 추출하여 외부에서 사용할 수 있도록 export 합니다.
export type Locale = (typeof routing.locales)[number];

// 라우팅 설정을 반영하는 Next.js 네비게이션 API의 가벼운 래퍼
export const { Link, redirect, usePathname, useRouter, getPathname } =
  createNavigation(routing);
```
- `proxy.ts`나 `request.ts`가 실행될 때 지원하는 언어 목록(locales)과 기본 언어(defaultLocale) 제공
- `createNavigation(routing)`으로 만들어진 다국어 전용 네비게이션 도구를 컴포넌트에 제공 (Link, useRouter 등)

##### 2단계: `proxy.ts`

- **언제 실행되나?** 사용자가 URL을 치고 엔터를 친 직후, 화면(HTML)을 그리기 **전**에 가장 먼저 실행
- 쿠키, 헤더 등을 확인해서 언어에 맞는 경로(`/ko/` 등)로 보내줌. 

``` ts
// proxy.ts
import createMiddleware from "next-intl/middleware";
import { routing } from "./i18n/routing";
import { NextRequest } from "next/server";

/**
 * handleI18nRouting의 4가지 핵심 기능:
 * 1. 언어 감지: 사용자의 브라우저 설정(Accept-Language)을 확인해 지원 언어 판별
 * 2. 리다이렉트: 판별된 언어에 맞는 경로(예: /en)로 자동 이동 (307 Redirect)
 * 3. 경로 연결: URL 변경 없이 서버에 언어 정보(x-next-intl-locale 헤더) 전달
 * 4. 쿠키 관리: 사용자가 선택한 언어를 쿠키에 저장하여 다음 접속 시 우선 적용
 */
const handleI18nRouting = createMiddleware(routing);

export function proxy(request: NextRequest) {
  return handleI18nRouting(request);
}

export const config = {
  // 다국어가 적용된 경로만 매칭
  matcher: ["/", "/(ko|en|ja|es|pt|zh|de|fr)/:path*"],
};
```

##### 3단계: `layout.tsx`와 `getMessages()`

- 이제 Next.js가 `/ko` 경로의 페이지를 바탕으로 화면(HTML)을 그리기 시작합니다.
- `getMessages()`: Next.js **서버 컴포넌트**가 실행될 때, 사용자의 언어에 맞는 번역 JSON 파일(예: `messages/ko.json`)을 서버 내에서 비동기로 미리 로드
- `<NextIntlClientProvider messages={messages}>`: 로드한 번역 JSON 파일을 React Context(상태 공유 시스템)에 주입, 이를 통해 하위 트리 내의 모든 **클라이언트 컴포넌트**들이 추가적인 서버 요청 없이 브라우저에서 `useTranslations()`로 번역된 텍스트를 읽고 렌더링할 수 있음. 

``` tsx
// layout.tsx
export default async function RootLayout({
  children,
  params,
}: {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
}) {
  const { locale } = await params;
  setRequestLocale(locale);

  // 들어온 locale이 유효한지 확인
  if (!routing.locales.includes(locale as Locale)) {
    notFound();
  }

  // 클라이언트 사이드에 모든 메시지를 제공하는 것이 가장 간편한 시작 방법입니다
  // request.ts에서 설정한 번역 데이터를 가져옴. 
  const messages = await getMessages();
  ...
  return (
    <html lang={locale} className={`${inter.variable} h-full antialiased`}>
      <body className="min-h-full flex flex-col font-sans">
        <NextIntlClientProvider messages={messages}>
          <Header />
          <main className="flex-1">
            {children}
          </main>
          <Footer />
        </NextIntlClientProvider>
        <Analytics />
      </body>
    </html>
  );
}

```


##### 4단계: `request.ts`

- `getMessages()`가 호출되는 순간, 즉 **화면을 그리는 도중에 번역 데이터가 필요할 때**마다 백그라운드에서 실행
- `requestLocale`: `proxy.ts`가 안내해 준 현재 경로의 언어값, 즉 `"ko"` 또는 `"en"` 이라는 문자열이 들어옴. (내부적으로 Next.js의 캐시나 헤더에서 읽어옴.)
- 역할
    - `requestLocale`이 지원하는 언어인지 검사
    - **실제로 하드디스크에 있는 `../messages/ko.json` 파일을 열어서 읽어옴.** (`import` 구문 실행)
    - 이 JSON 데이터를 덩어리째로 `layout.tsx`에게 넘겨줍니다.

``` ts
// request.ts
// 서버에서 사용자의 요청이 올 때마다 어떤 번역 데이터(ko.json 등)를 가져다 줄지 결정
export default getRequestConfig(async ({ requestLocale }) => {
  // 일반적으로 [locale] 경로 세그먼트에 해당함
  let locale = await requestLocale;

  // 유효한 언어가 사용되도록 보장
  if (!locale || !routing.locales.includes(locale as Locale)) {
    locale = routing.defaultLocale;
  }

  return {
    locale,
    messages: (await import(`../messages/${locale}.json`)).default,
  };
});
```

##### 언어 변경

단순히 `window.location.href`를 바꾸는 게 아니라, 라이브러리가 제공하는 래퍼(wrapper) 함수를 사용함으로써 **현재 주소창의 쿼리 파라미터나 해시 등은 유지하면서 언어만 깔끔하게 전환**하는 것이 핵심

``` tsx
// Header.tsx
const router = useRouter();
const pathname = usePathname();

const onLanguageChange = (nextLocale: string) => {
  // 현재 경로(pathname)는 유지하고, locale 옵션만 바꿔서 replace 합니다.
  router.replace(pathname, { locale: nextLocale });
};
```

1. **경로 유지 (Pathname Preservation):** 사용자가 현재 어느 페이지(예: `/about`, `/settings`)에 있든, 언어를 바꿨을 때 해당 페이지의 내용을 그대로 다른 언어로 보여줘야 함. 그래서 현재의 `pathname` 정보를 가져옴.
2. **클라이언트 사이드 네비게이션:** `next-intl`에서 제공하는 (`routing.ts`에서 만들어둔) `useRouter`를 사용하면 브라우저의 전체 새로고침 없이 URL만 살짝 바꿔주는 '소프트 네비게이션'이 가능.
3. **쿠키 업데이트 (자동 처리):** 이미 `proxy.ts`에 미들웨어에서 사용자가 `/en` 경로로 이동하면 브라우저의 언어 설정 쿠키(`NEXT_LOCALE`)를 자동으로 업데이트. 이렇게 하면 다음에 사이트에 다시 들어올 때도 사용자가 마지막으로 선택한 언어가 유지됨.


### 모바일 아이콘

1. 모바일 브라우저 "북마크(즐겨찾기)" 및 "자주 방문하는 사이트"
사용자가 홈 화면에 추가하지 않고 그냥 사파리나 크롬에서 **즐겨찾기(BookMark)**에 등록하거나, 새 탭을 열었을 때 나오는 **"자주 방문한 사이트" 그리드 목록**에서 이 메트로놈 사이트를 표현해 주는 로고 아이콘으로 바로 이 모바일 아이콘들이 사용됩니다.

- 이게 설정되어 있지 않으면 즐겨찾기 목록이나 새 탭 화면에서 아주 밋밋한 브라우저 기본 지구본 마크나 깨진 로고로 노출되어, 즐겨찾기 폴더 안에서 우리 사이트의 존재감이 낮아집니다.


2. 구글 모바일 검색 결과의 "파비콘(Favicon)" 노출
요즘 구글 모바일 검색창에서 어떤 키워드를 검색하면, 검색 결과 리스트의 사이트 제목 왼쪽에 **둥글고 귀여운 동그라미 로고 아이콘**이 함께 노출되는 것을 보셨을 것입니다. 구글 검색 로봇은 검색 결과의 썸네일 로고로 **웹 매니페스트(`manifest.ts`) 및 `apple-touch-icon`에 정의된 대표 모바일 이미지**를 우선적으로 수집하여 노출시킵니다. 즉, 모바일 구글 검색 유입을 늘리고 사이트의 신뢰도를 높이는 데 필수적입니다.


3. 모바일 멀티태스킹(앱 전환) 화면 및 최근 탭 목록
스마트폰에서 실행 중인 앱들을 옆으로 밀어서 전환하는 **멀티태스킹 화면(Task Manager)**이나 브라우저의 **최근 열어본 탭 리스트**에서도 메트로놈 웹페이지 탭 상단에 이 아이콘이 브랜드 표시등 역할을 하여 사용자가 웹 서핑 중 쉽게 메트로놈 탭을 구별하고 다시 찾아올 수 있도록 돕습니다.


4. 링크 공유(카카오톡, iMessage, 슬랙 등) 시 보완적 활용
카카오톡이나 메신저로 메트로놈 웹사이트 링크를 공유할 때, 간혹 공유 미리보기 전용 대표 이미지(`og:image`)가 로딩에 실패하거나 누락된 경우, 메신저 앱이 차선책으로 `apple-touch-icon`이나 `manifest`에 저장된 모바일 아이콘을 썸네일 카드로 긁어가서 예쁜 로고로 보여줍니다.


### 외부 API 통신 시 타입 단언

**`: Type` (타입 선언)은 "검사해줘"** 이고, **`as Type` (타입 단언)은 "내 말을 믿어"** 
- **일반적인 변수나 객체를 만들 때:** 항상 **`: Type` (타입 선언)** 을 사용하여 TS가 꼼꼼하게 검사하게 만듬.
- **외부 API 통신 (`fetch`, `JSON.parse` 등) 결과:** 어차피 TS가 검사할 수 없는 `any`나 `unknown`이 들어오는 곳이므로, 개발자가 책임을 지고 모양을 씌워준다는 의미를 강조하기 위해 **`as Type` (타입 단언)** 을 관습적으로 많이 사용

외부 API 통신을 할 때는 이렇게 **"내가 받을 데이터의 규격(Interface)을 만들고, `as`로 덮어씌운다"** 라고 생각하시면 Spring에서의 DTO 매핑과 거의 비슷함.
``` ts
// lib/api.ts 상단에 정의
interface YouTubeApiResponse {
    error: boolean;
    message?: string; // 에러 메시지는 에러가 났을 때만 있을 수 있으므로 옵셔널(?) 처리
    linkDownload: string;
    title: string;
    // 그 외 videoId, lengthSeconds 등은 당장 안 쓰니까 생략해도 무방합니다.
}
```


### 컴포넌트 재사용성 

```tsx
// Metronome.tsx
interface Props {
  isSyncMode?: boolean;
}

export default function Metronome({ isSyncMode = false }) {...}
```


- 만약 `?`를 빼고 `isSyncMode: boolean`으로 필수값 처리를 해버리면, 유튜브 기능이 없는 일반 페이지에서도 무조건 `<Metronome isSyncMode={false} />`라고 억지로 적어줘야 하는 불편함이 생김. 
- `?`를 붙여두면 부모가 아무 값을 안 넘겨도 에러가 나지 않으며, 컴포넌트 내부에서 알아서 `false`로 취급할 수 있어서 **컴포넌트의 재사용성**이 훨씬 높아짐.


### 복잡한 프로젝트인 경우

복잡한 프로젝트일수록 이런 식으로 **데이터를 담는 창고(Zustand)** 와 **로직을 처리하는 뇌(Custom Hook)**, 그리고 **화면을 보여주는 얼굴(Component)** 을 분리

##### 커스텀 훅 사용

커스텀 훅으로 로직과 컴포넌트를 분리한 예시 
- 로직만 모아둔 파일
``` ts
// useMetronomeSync.ts
import { useState, useEffect } from 'react';

export function useMetronomeSync() {
    // 여기에 기존 MetronomeStudio에 있던 지저분한 로직들을 다 몰아넣습니다.
    const [bpm, setBpm] = useState(120);
    const [beatsArray, setBeatsArray] = useState([]);
    // ... useEffect 등 각종 함수들 ...

    // 화면(UI)에서 쓸 것들만 바구니에 담아서 반환해줍니다.
    return { bpm, setBpm, beatsArray, setBeatsArray /* 등등 */ };
}
```

- 화면만 남은 파일
``` tsx
// MetronomeStudio.tsx
import { useMetronomeSync } from './useMetronomeSync';

export default function MetronomeStudio() {
    // 훅을 호출해서 로직과 데이터를 쏙 빼옵니다.
    const { bpm, setBpm, beatsArray, setBeatsArray } = useMetronomeSync();

    // 여기는 온전히 화면(UI) 그리는 데만 집중할 수 있습니다!
    return (
        <main>
            <Metronome bpm={bpm} onBpmChange={setBpm} />
            <YoutubeSync beatsArray={beatsArray} />
        </main>
    );
}
```

##### Zustand

- **극도의 간결함:** 복잡한 세팅 없이 `create` 함수 하나로 스토어(Store)를 만들고, 훅(Hook)처럼 바로 컴포넌트에서 꺼내 쓸 수 있음.
- **렌더링 최적화:** 컴포넌트가 자신이 구독한 특정 상태(State)가 바뀔 때만 리렌더링되므로 성능상 매우 유리
- **Context Provider 불필요:** 기본적으로 컴포넌트 트리를 특정 Provider로 감싸지 않아도 되기 때문에 코드가 깔끔해짐.

Next.js는 서버 사이드 렌더링을 수행하므로, 서버와 클라이언트 간의 상태 불일치로 인한 **Hydration(하이드레이션) 오류**를 방지해야함. 가장 안전하고 간단한 방법은 상태를 사용하는 컴포넌트를 **클라이언트 컴포넌트(`"use client"`)** 로 지정하는 것

1. 패키지 설치: `npm install zustand`
2. 스토어 생성: 상태(state)와 상태를 변경하는 함수(action)를 한곳에 정의
``` ts
// src/store/useCounterStore.ts
import { create } from 'zustand';

// 상태의 타입 정의 (TypeScript 사용 시)
interface CounterState {
  count: number;
  increase: () => void;
  decrease: () => void;
}

// 스토어 생성
export const useCounterStore = create<CounterState>((set) => ({
  count: 0, // 초기값
  increase: () => set((state) => ({ count: state.count + 1 })),
  decrease: () => set((state) => ({ count: state.count - 1 })),
}));
```

3. 컴포넌트에서 사용
``` tsx
// src/components/Counter.tsx
'use client';

import { useCounterStore } from '@/store/useCounterStore';

export default function Counter() {
  // 스토어에서 필요한 상태와 함수를 구조 분해 할당으로 가져옵니다.
  const { count, increase, decrease } = useCounterStore();

  return (
    <div style={{ padding: '20px', border: '1px solid #ccc', borderRadius: '8px', width: '200px', textAlign: 'center' }}>
      <h2>카운트: {count}</h2>
      <div style={{ display: 'flex', gap: '10px', justifyContent: 'center' }}>
        <button onClick={decrease} style={{ padding: '5px 10px' }}>-</button>
        <button onClick={increase} style={{ padding: '5px 10px' }}>+</button>
      </div>
    </div>
  );
}
```


개발자들은 **"이 데이터가 진짜 여러 컴포넌트에서 공유되어야 하는 앱의 핵심 상태(Global State)인가?"** 를 고민한 뒤, 정말 필요한 상태(예: 현재 BPM 설정, 로그인한 유저 정보)만 Zustand에 넣고, 나머지는 기본 `useState`와 Props로 해결하는 식으로 **균형(Trade-off)** 을 맞추며 개발함


### SEO 

##### 메타데이터 

메타데이터는 `generateMetadata()`를 사용해서 설정하면 Next.js가 자동으로 적용해줌. 

``` tsx
// layout.tsx
export async function generateMetadata({
  params,
}: {
  params: Promise<{ locale: string }>;
}): Promise<Metadata> {
  const { locale } = await params;
  setRequestLocale(locale);
  const messages = await getMessages({ locale });
  const t = messages.Metadata as {
    defaultTitle: string;
    defaultDescription: string;
  };

  return {
	// 공통 메타데이터
    title: {
      default: t.defaultTitle,
      template: `%s | ${t.defaultTitle}`,
    },
    description: t.defaultDescription,
    icons: {
      icon: "/icon.svg",  // 모든 하위 경로에서도 favicon이 적용되도록 명시적으로 지정
    },
  };
}


// page.tsx
export async function generateMetadata({
  params,
}: Props): Promise<Metadata> {
  const { locale } = await params;
  setRequestLocale(locale);
  const messages = await getMessages({ locale });
  const t = messages.Metadata as {
    title: string;
    description: string;
  };

  return {
    title: {
      absolute: t.title, // layout.tsx의 template(접미사)을 적용하지 않고 메인 타이틀을 그대로 노출
    },
    description: t.description,
    alternates: buildLanguageAlternates("/", locale), // 캐노니컬 태그 설정과 hreflang 태그 설정
  };
}
```

- 캐노니컬 태그와 hreflang 태그 설정을 위해  `metadata.alternates`을 제공해줌. 
``` ts
import { routing } from "@/i18n/routing";

const BASE_URL = "https://ytmetronome.com";

export function buildLanguageAlternates(pathname: string, currentLocale: string) {
  // 1. 전달받은 경로(예: "tempo/60-bpm-metronome")가 "/"로 시작하는지 확인하고 정규화합니다.
  const normalizedPath = pathname.startsWith("/") ? pathname : `/${pathname}`;
  const canonicalLocale = routing.locales.includes(
    currentLocale as (typeof routing.locales)[number]
  )
    ? currentLocale
    : routing.defaultLocale;

  // 2. hreflang 태그 설정
  // 프로젝트가 지원하는 모든 언어(ko, en, de, es, fr, ja, pt, zh)를 돌면서
  // { "ko": "https://ytmetronome.com/ko/...", "en": "https://ytmetronome.com/en/..." } 형태의 객체를 만듭니다.
  const languages = Object.fromEntries(
    routing.locales.map((locale) => [
      locale,
      `${BASE_URL}/${locale}${normalizedPath === "/" ? "" : normalizedPath}`,
    ])
  );

  // 3. Next.js Metadata가 이해하는 구조로 값을 반환합니다.
  return {
    canonical: `${BASE_URL}/${canonicalLocale}${normalizedPath === "/" ? "" : normalizedPath}`,
    languages: {
      ...languages,
      "x-default": `${BASE_URL}/en${normalizedPath === "/" ? "" : normalizedPath}`,
    },
  };
}
```

##### JSON-LD

``` tsx
// layout.tsx

  // 검색엔진에 앱 정보를 전달하기 위한 JSON-LD 구조화된 데이터
  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "WebApplication",
    name: "YT Metronome",
    url: "https://ytmetronome.com",
    description: meta.description,
    applicationCategory: "MusicApplication",
    operatingSystem: "Web",
    offers: {
      "@type": "Offer",
      price: "0",
      priceCurrency: "USD",
    },
    inLanguage: locale,
  };

  return (
    <html lang={locale} className={`${inter.variable} h-full antialiased`}>
      <body className="min-h-full flex flex-col font-sans">
        <script
          type="application/ld+json"
          dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
        />
		...
      </body>
    </html>
  );
}
```

``` tsx
// BeatsSeoContent.tsx

  const jsonLd = {
    "@context": "https://schema.org",
    "@type": "HowTo",
    name: t("common.howToUse.title"),
    description: t(`${slug}.title`),
    url: `https://ytmetronome.com/${locale}/beats/${slug}`,
    inLanguage: locale,
    step: howToSteps.map((step) => ({
      "@type": "HowToStep",
      name: step.name,
      text: step.text,
      url: `https://ytmetronome.com/${locale}/beats/${slug}#beats-howto-${step.id}`,
    })),
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(jsonLd).replace(/</g, "\\u003c"),
        }}
      />
      ...
    </>
  );
}
```

1. 전체 공통 레이아웃 (`layout.tsx`): 웹서비스 전체를 설명하는 공통 정보인 `WebApplication` 스키마 하나만 선언
2. 특정 SEO 전용 컴포넌트 (`BeatsSeoContent` 등): 이 컴포넌트가 있는 페이지는 하단에 **사용 방법(How-To)** 텍스트를 제공하고 있음. 따라서 이에 맞춰 `HowTo` 형식의 스키마를 해당 컴포넌트 내에 추가하는 것이 맞음.


##### generateStaticParams()

- 빌드 시점에 `generateStaticParams()`를 호출해 대상 경로(`slug`) 목록을 사전에 정의하고, 각 경로에 대응하는 HTML 뼈대를 미리 렌더링하여 서버에 저장해둠. (SSG 정적 사이트 생성)
- 장점: 서버 연산이 필요 없어 페이지 응답 속도가 빠름, 구글 크롤러 봇이 자바스크립트를 해석할 필요 없이 정적 HTML 텍스트를 즉시 수집할 수 있음. (SEO 점수 향상)

``` tsx
// app/[locale]/beats/[slug]/page.tsx
import MetronomeStudio from "@/components/MetronomeStudio";
import { notFound } from "next/navigation";
import { BeatsPerBar, NoteValue } from "@/lib/metronome";
import BeatsSeoContent from "@/components/seo/BeatsSeoContent";
import type { Metadata } from "next";
import { getMessages, setRequestLocale } from "next-intl/server";
import { buildLanguageAlternates } from "@/lib/seo";

// 지원하는 박자(Time Signature) 리스트와 각각의 기본 설정값 매핑
const BEATS_MAP: Record<
  string,
  {
    beatsPerBar: BeatsPerBar;
    noteValue: NoteValue;
    beatStates: number[];
  }
> = {
  "2-4-metronome": {
    beatsPerBar: 2,
    noteValue: 4,
    beatStates: [2, 1],
  },
  "3-4-metronome": {
    beatsPerBar: 3,
    noteValue: 4,
    beatStates: [2, 1, 1],
  },
  // ...
};

interface Props {
  params: Promise<{ locale: string; slug: string }>;
}

// 정적 빌드 시 미리 만들 페이지 지정
export function generateStaticParams() {
  return Object.keys(BEATS_MAP).map((slug) => ({
    slug,
  }));
}

// 페이지별 동적 메타데이터 생성
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { locale, slug } = await params;
  setRequestLocale(locale);
  const config = BEATS_MAP[slug];

  if (!config) return {};

  const messages = await getMessages({ locale });
  const meta = messages.Metadata as {
    beatsTitleTemplate: string;
    beatsDescTemplate: string;
  };

  // 템플릿 키를 불러온 후 {beat} 문자열을 동적 값으로 치환
  const titleTemplate = meta.beatsTitleTemplate || "{beat} Metronome";
  const descTemplate =
    meta.beatsDescTemplate ||
    "Free online {beat} metronome. Practice with a precise {beat} beat tempo.";

  const displayBeat = slug
    ? slug.replace("-metronome", "").replace("-", "/")
    : "";
  const title = titleTemplate.replace(/{beat}/g, displayBeat);
  const description = descTemplate.replace(/{beat}/g, displayBeat);

  return {
    title,
    description,
    alternates: buildLanguageAlternates(`/beats/${slug}`, locale),
  };
}

export default async function BeatsPage({ params }: Props) {
  const { locale, slug } = await params;
  setRequestLocale(locale);
  const config = BEATS_MAP[slug];

  // 등록되지 않은 잘못된 박자 슬러그인 경우 404 반환
  if (!config) {
    notFound();
  }

  return (
    <>
      <MetronomeStudio
        key={slug} // 박자가 다른 페이지로 이동 시 컴포넌트를 완전히 다시 마운트하여 설정값을 갱신합니다.
        initialBpm={120} // 기본 속도는 120 BPM으로 세팅
        initialBeatsPerBar={config.beatsPerBar}
        initialNoteValue={config.noteValue}
        initialBeatStates={config.beatStates}
      />
      <BeatsSeoContent slug={slug} />
    </>
  );
}
```
- `generateStaticParams`가 빌드 시점에 실행되면서 `BEATS_MAP` 객체의 키들(`2-4-metronome`, `3-4-metronome` 등)을 뽑아내어 리턴
- Next.js는 이 값들을 받아 배포 파일을 빌드할 때, 각 언어별(`ko`, `en` 등) 폴더 밑에 해당하는 모든 박자 페이지들을 미리 HTML 파일로 다 구워놓음.


##### robots.ts

``` ts
// app/robots.ts
import { MetadataRoute } from "next";

// 모든 검색 엔진 크롤러의 접근을 허용하고 사이트맵 위치를 제공하기 위한 robots.txt 설정
export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: "*",
      allow: "/",
    },
    sitemap: "https://ytmetronome.com/sitemap.xml",
  };
}
```


##### 사이트맵

``` ts
// app/sitemap.ts
import { MetadataRoute } from "next";

// 검색 엔진에 사이트 구조 및 크롤링 힌트를 제공하기 위한 sitemap.xml 설정
export default function sitemap(): MetadataRoute.Sitemap {
  // 1. 기본 루트 경로 설정
  const rootSitemap = {
    url: BASE_URL,
    lastModified: new Date(),
    changeFrequency: "yearly" as const,
    priority: 1.0,
  };

  // 2. 각 언어별 경로 설정 (각 언어별 메인 페이지도 중요도를 1.0으로 동일하게 지정)
  const localeSitemaps = locales.map((locale) => ({
    url: `${BASE_URL}/${locale}`,
    lastModified: new Date(),
    changeFrequency: "yearly" as const,
    priority: 1.0,
  }));

  const legalSitemaps = LEGAL_LOCALES.flatMap((locale) => [
    {
      url: `${BASE_URL}/${locale}/privacy`,
      lastModified: new Date(),
      changeFrequency: "yearly" as const,
      priority: 0.3,
    },
    {
      url: `${BASE_URL}/${locale}/terms`,
      lastModified: new Date(),
      changeFrequency: "yearly" as const,
      priority: 0.3,
    },
  ]);
  // ...
}
```


##### 매니페스트

``` ts
// app/manifest.ts
import { MetadataRoute } from "next";

/**
 * 안드로이드 크롬 기기 및 PWA 환경을 위한 웹 앱 매니페스트 설정을 정의합니다.
 * 홈 화면에 바로가기 앱처럼 등록될 때의 상세 메타정보와 아이콘을 매핑해 줍니다.
 */
export default function manifest(): MetadataRoute.Manifest {
  return {
    name: "YT Metronome", // 설치 시 보이는 앱의 풀 네임
    short_name: "YT Metronome", // 홈 화면 아이콘 아래에 노출될 짧은 이름
    description: "Video-synced Smart Metronome for accurate tempo and beat",
    start_url: "/", // 앱을 열었을 때 최초로 진입할 경로
    display: "standalone", // 주소창을 제거하여 스마트폰 네이티브 앱처럼 전체화면으로 실행
    background_color: "#3c3a3a", // 앱 첫 구동 시 로딩 화면의 배경색
    theme_color: "#648f56", // 안드로이드 시스템 상단바 및 브라우저 색상 테마 적용
    icons: [
      {
        src: "/icon.svg", // app/icon.svg 파일을 사용하여 크기 왜곡 없는 고품질 벡터 아이콘으로 홈 화면 로고 자동 적용
        sizes: "any",
        type: "image/svg+xml",
      },
    ],
  };
}
```

##### iOS 전용 아이콘

``` tsx
// app/apple-icon.tsx
import { ImageResponse } from "next/og";

// 에지 런타임을 사용하여 전세계 어디서든 빠르게 아이콘을 렌더링하고 전송합니다.
export const runtime = "edge";

// iOS Safari 규격에 맞춘 apple-touch-icon 크기 (180x180 px) 및 컨텐츠 타입 지정
export const size = {
  width: 180,
  height: 180,
};
export const contentType = "image/png";

/**
 * iOS 기기 홈 화면 추가 시 앱 아이콘 대용으로 사용되는 동적 PNG 이미지 생성기입니다.
 * 별도의 이미지 편집기 없이 app/icon.svg의 백터 메트로놈 로고를 기반으로
 * 180x180 크기의 고화질 PNG 이미지를 자동으로 렌더링하여 전송해 줍니다.
 */
export default function Icon() {
  return new ImageResponse(
    (
      <div
        style={{
          width: "100%",
          height: "100%",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          background: "#3c3a3a", // globals.css의 --surface/--secondary 회갈색 테마 적용
          borderRadius: "50%",
        }}
      >
		{/* 아이콘 */}
      </div>
    ),
    {
      ...size,
    }
  );
}
```

