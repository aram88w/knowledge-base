### 개요

자바스크립트 코드를 실행하기 _전에_ 타입 관련 버그를 미리 발견할 수 있는 도구가 있다면 좋을 것입니다. 
TypeScript와 같은 정적 타입 검사기의 역할이 바로 그것입니다. *정적 타입 시스템*은 우리가 작성한 프로그램에서 사용된 값들의 형태와 동작을 설명합니다.


### 컴파일러

`tsc`명령어를 통한 컴파일로 TypeScript 전용 코드를 제거하거나 변환하여 순수한 JavaScript 파일로 만들어서 기존의 JS와 동일하게 돌아가도록 함. 

타입검사 수행: `.ts` 파일 -> `.js` 파일
- `tsc hello.ts` 
- `tsc --noEmitOnError hello.ts` : 에러 발생시 js 파일 생성 X
- `tsc --target es2015 hello.ts` : ES2015 기준으로 js파일을 생성

`strcit` 플래그를 설정하여 모든 플래그를 활성화 할 수 있음. (대부분 `strict`를 활성화 해서 사용)
1. `tsconfig.json`에 `"strict": true`를 추가하거나
2. CLI에서 `--strict` 플래그를 설정

주요 옵션 
1. `noImplicitAny`: 변수가 `any`로 암묵적으로 추론되는 경우에 오류를 발생시킴. 
2. `strictNullChecks`: 
	- null과 undefined를 독립적인 타입으로 취급함. (string 타입에 null이 들어오거나 할 수 없음.)
	- 어떤 값에 null 또는 undefined를 허용했을 때, 해당 값 사용하기에 앞서 해당 값을 체크해야 하도록 강제함.
		- `!` 표현식:  사용하면 해당 값이 null 또는 undefined가 아니라고 타입 단언을 할 수 있음.


### 타입

#### TS 타입
##### any 타입

- 모든 종류의 값을 허용하는 타입 
- `noImplicitAny` 옵션을 켜야 타입을 모르는 경우에 `any`로 추론되는 경우를 막을 수 있음. 

``` ts
// noImplicitAny 활성화
function add(n) { // <- 여기서 에러 발생! 'n'이 뭔지 모르겠음. (any로 추론되지 않음.)
  return n + 1;
}
// ✅ "n은 확실히 number야!"라고 명시 해줘야함. 
function add(n: number) { 
  return n + 1;
}
// 혹은 n을 any라고 명시 해줘야함.
function add(n: any) { 
  return n + 1;
}
```

##### unknown 타입

- 모든 종류의 값을 허용하는 타입
- 사용하기 전에 타입 확인(`instanceof`)을 무조건 해야함. 
- 타입스크립트에서는 try-catch에서 `catch(err: unknown)` 부분을 `unknwon`으로 강제함. 

``` ts
} catch (err: unknown) {
  // 1. err가 진짜 Error 인스턴스인지 확인 (가장 일반적인 방법)
  if (err instanceof Error) {
    console.error("Fetch Error:", err.message);
    setMessage(`❌ 에러 발생: ${err.message}`);
  } else {
    // 2. Error 객체가 아닌 다른 것이 던져졌을 때 처리
    console.error("Unknown Error:", err);
    setMessage("❌ 알 수 없는 에러가 발생했습니다.");
  }
  
  setStatus("error");
}
```

##### void 타입

- 값을 반환하지 않는 함수의 반환 값
- 다른 값이 반환되어도 무시됨. 

##### 유니온 타입 

- 타입을 조합하여 두 개 이상의 타입 중 하나의 타입을 가질 수 있도록 하는 것
``` ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
```

##### 튜플 

튜플: 몇 번째 자리에 무슨 타입이 오는가 
``` ts
function doSomething(pair: [string, number]) {
  const [name, age] = pair; // 0번은 name으로, 1번은 age로 부르겠다!
  console.log(name.toLowerCase()); // name은 string이니까 문자열 메서드 사용 가능
}
```

##### 인덱스 접근 타입

``` ts
type Person = { age: number; name: string };

// Person 타입에서 "age"라는 칸(속성)의 타입을 가져와라! AgeType은 number
type AgeType = Person["age"];
```
1.  문자열 리터럴 타입 (직접 쓰기): 가장 기본입니다. 따옴표를 붙여서 특정 속성 이름을 직접 적습니다.
	- Person["name"] → string
	
2. 유니온 타입 (여러 개 합치기): 여러 칸의 타입을 한꺼번에 뽑아서 합칠 수도 있습니다.
	- Person["age" | "name"] → number | string
	
3. 다른 타입 변수 (소환하기): 미리 정의된 타입을 넣을 수 있습니다.
	- type MyKey = "age";
	- type Age = Person[MyKey]; → number

``` ts
interface ApiResponse {
  user: {
    id: number;
    profile: {
      nickname: string;
      avatar: string;
    }
  }
}

// 프로필 컴포넌트를 만들 때, 거대한 ApiResponse를 통째로 넘기지 않고
// 딱 profile 부분의 타입만 떼서 쓰고 싶을 때 사용합니다.
type ProfileProps = ApiResponse["user"]["profile"];

function UserProfile({ nickname, avatar }: ProfileProps) {
  return <div>{nickname}</div>;
}
```


##### Mapped 타입

- 새로운 타입으로 변환하는 기술
``` ts
// 객체의 키에 
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```
- **`keyof Type`**: `Type`이 가진 모든 키를 뽑아서 유니온 타입으로 만듭니다.
- **`Property in ...`**: 자바스크립트의 `for...in` 루프처럼, 뽑아낸 키들을 하나씩 순회합니다.
- **`: boolean`**: 원래 어떤 타입이었든 상관없이, 모든 값의 타입을 `boolean`으로 바꿉니다.

``` ts
// 1. 원본 타입 (기능 목록)
type Features = {
  darkMode: () => void;
  newUserProfile: () => void;
};

// 2. 맵드 타입 적용 (설정 플래그로 변환)
type FeatureOptions = OptionsFlags<Features>;

/* 결과적으로 FeatureOptions는 다음과 같이 정의된 것과 같습니다:
type FeatureOptions = {
  darkMode: boolean;       // 함수 타입이 boolean으로 변함
  newUserProfile: boolean; // 함수 타입이 boolean으로 변함
};
*/
```


##### 유틸리티 타입

1. **`Awaited<T>`**: 비동기 작업(`Promise`)의 결과값 타입 추출
``` ts
type A = Awaited<Promise<string>>;
// 결과: string (Promise 상자를 한 번 까서 안의 string을 꺼냄)
```

2. **`Partial<T>`**: 기존타입의 필수인 속성들을 선택사항인 타입으로 생성

3. `Required<T>`: 기존 타입에 `?`가 붙어있어 선택사항이던 타입을 필수인 타입으로 생성

4. `Readonly<T>`: 속성을 수정 불가능하게 함

5. `Record<K, T>`: 이런 이름들(Keys)을 가진 객체를 만들 건데, 내용은 전부 이 모양(Type)으로 채울때 사용
``` ts
type CatName = "miffy" | "boris" | "mordred"; // 이름표 종류들 (Keys)

interface CatInfo { // 내용물 규격 (Type)
  age: number;
  breed: string;
}

// "고양이 이름 3개에 각각 고양이 정보를 담겠다"는 뜻
// CatName에 해당하는 key만 가능
const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
```

6. `Pick<T, K>`: 이미 만들어진 타입에서 내가 필요한 속성만 골라서 새로운 타입을 생성
``` ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}

// Todo에서 'title'과 'completed'만 골라서 'TodoPreview'라는 새 타입을 만듭니다.
type TodoPreview = Pick<Todo, "title" | "completed">;

const todo: TodoPreview = {
  title: "방 청소",
  completed: false,
  // description: "침대 밑까지" <-- 이걸 쓰면 에러가 납니다! (안 골랐으니까요)
};
```

6. `Omit<T, K>`: 특정 속성만 뺀 새로운 타입 생성. `Pick`과 반대
``` ts
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}

// 1. description만 쏙 빼고 싶을 때
type TodoPreview = Omit<Todo, "description">;
// 결과: { title: string; completed: boolean; createdAt: number; }

// 2. 여러 개를 빼고 싶을 때 (| 기호 사용)
type TodoInfo = Omit<Todo, "completed" | "createdAt">;
// 결과: { title: string; description: string; }
```

7. `Exclude<Union, T>`: 유니온타입 목록에서 특정 타입 제외
``` ts
// 1. 단순 문자열 목록에서 빼기
type T0 = Exclude<"a" | "b" | "c", "a">; 
// 결과: "b" | "c" ('a'를 목록에서 탈락시킴)

// 2. 특정 종류의 타입만 빼기
type T2 = Exclude<string | number | (() => void), Function>;
// 결과: string | number (함수 형태인 것들을 다 빼버림)

// 3. 복잡한 객체 유니온에서 빼기
type Shape = 
  | { kind: "circle"; radius: number }
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T3 = Exclude<Shape, { kind: "circle" }>;
// 결과: "square"와 "triangle" 타입만 남음
```

8. `Extract<T, Union>`: 유니온타입 목록에서 내가 지정한 것과 겹치는 타입만 남김.
``` ts
// 1. 겹치는 문자열 찾기
type T0 = Extract<"a" | "b" | "c", "a" | "f">;
// 결과: "a" (양쪽 목록에 공통으로 있는 것만 추출)

// 2. 특정 조건에 맞는 타입만 추출
type T1 = Extract<string | number | (() => void), Function>;
// 결과: () => void (함수 타입인 것만 골라냄)
```

9. `NonNullable<T>`: null, undefined를 타입 목록에서 제외 
``` ts
type T0 = NonNullable<string | number | null | undefined>;
// 결과: string | number (null과 undefined가 모두 사라졌습니다)
```

10. `Parameters<T>`: 함수 타입을 넣으면 인자로 무엇을 받는지 추출해서 튜플(순서가 정해진 배열) 타입으로 반환
``` ts
declare function f1(arg: { a: number; b: string }): void;

// 1. 인자가 없는 함수
type T0 = Parameters<() => string>;
// 결과: [] (재료가 없으니 빈 배열)

// 2. 인자가 하나인 함수
type T1 = Parameters<(s: string) => void>;
// 결과: [s: string] (string이 들어가는 배열 형태)

// 3. 이미 정의된 함수 f1의 인자 가져오기
type T3 = Parameters<typeof f1>;
// 결과: [arg: { a: number; b: string }] 
// f1이 받는 객체 모양을 그대로 가져옵니다.
```

``` ts
function sendMessage(text: string) {
  console.log("메시지 전송:", text);
}

// sendMessage의 인자 타입을 그대로 가져옵니다. -> [text: string]
type SendMessageParams = Parameters<typeof sendMessage>;

function logAndSend(...args: SendMessageParams) {
  console.log("로그 기록 중...");
  // ...args를 사용하면 ["abc"] 형태의 배열이 펼쳐져서
  // sendMessage("abc") 처럼 호출됩니다.
  sendMessage(...args); 
}

logAndSend("안녕하세요!"); // ✅ 정상 작동
```

11. `ReturnType<T>`: 함수 타입을 넣으면, 그 함수가 실행을 마친 뒤 **반환(return)하는 값의 타입**이 무엇인지 추출
``` ts
// 아주 복잡한 객체를 리턴하는 함수
function getComplexUser() {
  return {
    id: 1,
    name: "Gemini",
    preferences: {
      theme: "dark",
      languages: ["ko", "en"]
    }
  };
}

// 이 함수의 결과물 타입을 내가 또 써야 한다면? 일일이 정의하기 힘들죠.
// 이럴 때 ReturnType을 씁니다!
type UserData = ReturnType<typeof getComplexUser>;

// 이제 UserData는 자동으로 { id: number; name: string; ... } 타입을 가집니다.
const myUser: UserData = {
    id: 2,
    name: "TypeScript",
    preferences: { theme: "light", languages: ["jp"] }
};
```


#### 객체 타입
##### type 타입 별칭

- 객체 타입, 유니온 타입에 이름을 부여하여 재사용할 수 있도록 함. 
``` ts
type Point = {
  x: number;
  y: number;
};
```

##### interface

- 타입 별칭과 동일
``` ts
interface Point {
  x: number;
  y: number;
}
```

타입 별칭과 인터페이스는 매우 유사하며, 대부분의 경우 둘 중 하나를 자유롭게 선택하여 사용할 수 있음. `interface`가 가지는 대부분의 기능은 `type`에서도 동일하게 사용 가능함. 
이 둘의 가장 핵심적인 차이는, *타입은 새 프로퍼티를 추가하도록 개방될 수 없는 반면, 인터페이스의 경우 항상 확장될 수 있다는 점입니다.*


#### 규칙
##### 타입 표기

``` ts
// 1. 변수
let myName: string = "Alice"; // string 타입 (그러나 타입 추론이 있기에 타입 표기를 하지 않음.)

// 2. 매개변수 
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}

// 3. 반환 타입
function getFavoriteNumber(): number { // 타입 추론이 있기에 대부분 타입 표기를 하지 않음.
  return 26;
}
```


##### 타입 추론 

``` ts
let a = 1; // TS: "처음 들어온 게 숫자네? 그럼 a는 앞으로 무조건 숫자(number) 전용이야!"

a = 100;   // ✅ 가능 (같은 숫자니까)
a = "str"; // ❌ 에러! (문자열은 숫자 전용 그릇에 담을 수 없음)
```

##### 타입 단언

- 타입 추론으로 결정되는 타입 말고 다른 타입을 지정하고 싶을때 사용
- 타입을 변환시키는 것이 아니라 컴파일 괒어에서 해당 타입으로 인식하게 만드는 것뿐임. 
``` ts
// document.getElementById의 경우 TypeScript는 HTMLElement로 타입을 지정하지만, 당신은 페이지 상에서 사용되는 ID로는 언제나 `HTMLCanvasElement`가 반환된다는 사실을 이미 알고 있을 수도 있습니다.
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```


##### 리터럴 타입

- `string`과 `number`와 같은 일반적인 타입 이외에도, _구체적인_ 문자열과 숫자 값, 불리언 값을 타입 위치에서 지정할 수 있습니다.
- 유니온과 함꼐 사용하여 특정 종류의 값들만 인자로 받도록 하는데 유용함.
``` ts
let changingString = "Hello World"; // 타입: string
const changingString = "Hello World"; // 타입: "Hello World"
```

``` ts
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

##### 리터럴 추론

TS는 객체의 속성 값을 허용해야하기에 객체는 `const`지만 (객체 리터럴 타입) 객체 속성은 일반적인 타입(`number` 타입 등)을 부여함. 
``` ts
const obj = { counter: 0 }; // obj의 타입은 {counter: number}
obj.counter = 1; // obj.counter의 타입은 number이기에 이게 가능함.
```

그러기 때문에 이런 문제가 발생함. req.method는 아까와 같은 이유로 `string` 타입을 부여받음. 그래서 handleRequest()의 파라미터로 타입 불일치가 생김.
```ts
function handleRequest(url: string, method: "GET" | "POST") {
  // ...
}
const req = { url: "https://example.com", method: "GET" }; 
// ❌ 에러 발생!
handleRequest(req.url, req.method); 
// 에러 메시지: "req.method(string 타입)은 'GET' | 'POST' 타입에 할당할 수 없습니다."

/*
  req 타입: {url: string, method: string}
  req.url/method 타입: string
*/

```

해결방안 
1. 둘 중에 한 위치에 **타입 단언**을 추가하여 추론 방식을 변경할 수 있습니다.
``` ts
// 방법 1
const req = { url: "https://example.com", method: "GET" as "GET" };
// 방법 2
handleRequest(req.url, req.method as "GET");
```

2. `as const`를 사용하여 객체 전체를 리터럴 타입으로 변환할 수 있습니다.
``` ts
const req = { url: "https://example.com", method: "GET" } as const;
/*
  req 타입: { readonly url: "https://example.com"; readonly method: "GET"; }
  req.url 타입: "https://example.com" (리터럴 타입)
  req.method 타입: "GET" (리터럴 타입)
*/
```


### 타입 좁히기

##### typeof 타입 좁히기

- `typeof null`이 `"object"`가 나오는 자바스크립트의 고질적인 문제가 있음.
- TS에서는 null인지 아닌지를 확인하는 과정을 빼먹으면 에러가 발생함.
``` ts
function printAll(strs: string | string[] | null) {
  // 여기서 "객체(object)"인 것만 걸러내려고 합니다.
  if (typeof strs === "object") {
    // 💥 여기서 에러 발생!
    // 왜? JS에서 'null'도 'object' 판정을 받기 때문이죠.
    // TS는 경고합니다: "야, strs가 배열일 수도 있지만 'null'일 수도 있어!"
    // ...
    
  // 이렇게 null 확인을 해줘야함. 
  if (strs && typeof strs === "object") {
```

##### 참/거짓 기반 타입 좁히기

- `0`
- `NaN`
- `""` (자바스크립트에서 빈 문자열(`""`)은 `false`로 취급됨.)
- `0n` (bigint 타입의 0)
- `null`
- `undefined`

##### 동등 비교를 통한 타입 좁히기

- true/false 체크(`if (strs)`)는 너무 많은 걸(0, "") 한꺼번에 걸러내지만, **정확한 비교**(`!== null`)를 쓰면 우리가 원하는 것만 골라낼 수 있음.
- 자바스크립트에서 `==` (느슨한 비교)는 가끔 위험하지만, **`null`과 `undefined`를 동시에 체크할 때**만큼은 최고의 치트키입니다.
``` ts
// 1. 엄격한 비교 (===)를 쓸 때
if (x === null || x === undefined) {
  console.log("값이 비어있습니다.");
}

// 2. 느슨한 비교 (==) 치트키를 쓸 때: null과 undefined 둘 다 체크됨
if (x == null) {
  console.log("값이 비어있습니다.");
}
```

##### in 연산자로 타입 좁히기 

``` ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) { // swim 속성을 가진 타입만으로 좁혀짐
    return animal.swim();
  }
 
  return animal.fly();
}
```

##### 제어 흐름 분석

- TS는 단순히 문법을 검사하는 수준을 넘어, **코드가 실행되는 "길(Flow)"을 추적**해서 타입을 추적함. 
``` ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input; // 여기서 padding이 number인 경우가 반환되므로
  }
  return padding + input; // 여기서는 padding이 string일 수 밖에 없음. 이 사실을 TS가 알고 있음.
}
```
##### 타입 서술어

- `매개변수명 is 타입`: true, false를 반환하는 자바스크립트 함수의 리턴 타입 자리에 적어서 타입을 서술해줌.
``` ts
function isFish(pet: Fish | Bird): pet is Fish {
  // 실제 로직은 true 아니면 false를 리턴하는 평범한 로직입니다.
  return (pet as Fish).swim !== undefined; 
}

if (isFish(pet)) {
  // 🔵 함수가 true를 반환한 경우: Fish 타입으로 지정
  // Fish 관련 메서드 사용 가능
  pet.swim();
} else {
  // 🔴 함수가 false를 반환한 경우: Bird 타입으로 지정 (남은게 Bird이므로)
  // Bird 관련 메서드 사용 가능
  pet.fly();
}
```

##### `never` 타입

타입 좁히기(Narrowing)를 계속하다 보면, **모든 가능성이 제거되어 더 이상 남은 타입이 없는 순간**의 타입

``` ts
function check(x: string | number) {
  if (typeof x === "string") {
    // x는 string
  } else if (typeof x === "number") {
    // x는 number
  } else {
    // 🔍 여기서 x의 타입은? 바로 'never'입니다!
    // x가 문자열도 아니고 숫자도 아닐 수는 없으니까요.
  }
}
```


### 함수

***함수도 객체임***

##### 호출 시그니처 

- 함수는 호출이 가능할 뿐만 아니라, 속성도 가질 수 있습니다. (함수도 객체임)
- 함수를 객체로 표현하기 위해서 함수 자신의 표현으로 호출 시그니처를 사용함. 

``` ts
function welcome() { console.log("hi"); }
welcome.admin = true; // 함수는 객체이므로 속성을 붙이는 것이 가능함.
```

- 함수(객체) 타입
``` ts
type DescribableFunction = {
  description: string; // 함수 객체의 속성
  (someArg: number): boolean; // 호출 시그니처, 콜론(:)을 사용하여 정의하고 해당 타입의 함수를 호출하면 실행되는 함수임
};
function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}
```


``` ts
interface GenericIdentityFn {
  <Type>(arg: Type): Type; // (1) 호출 시그니처: 이름이 없음! -> "나 자체가 함수야"
}

interface NormalObject {
  method<Type>(arg: Type): Type; // (2) 메서드: 'method'라는 이름이 있음 -> "내 안에 함수가 있어"
}
```
- (1)번의 경우: `let fn: GenericIdentityFn = ...` 이라고 선언하면, `fn(10)` 처럼 바로 호출할 수 있습니다. (함수 객체)
- (2)번의 경우: `let obj: NormalObject = ...` 이라고 선언하면, `obj.method(10)` 처럼 이름(key)을 거쳐서 호출해야 합니다. (method라는 메서드를 가진 객체)

- 속성을 가지는 함수 객체 생성 방법 (참고)
``` ts
// 방법 1: Object.assign 사용 (권장)
const a: DescribableFunction = Object.assign( 
  (arg: number) => arg > 0,
  { description: "hello" }
);

// 방법 2: 함수 먼저 만들고 나중에 속성 추가 (함수 선언문 활용)
function myFunc(arg: number) { return arg > 0; }
myFunc.description = "hello";
const a: DescribableFunction = myFunc; // 완성된 후에 대입
```

##### 제네릭 함수

``` ts
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```

##### 콜백 함수 주의 사항

콜백에 대한 함수 타입을 작성할 때, _절대로_ 선택적 매개변수(`?`)를 사용 X
콜백 함수 타입을 정의할때 `?`를 사용하지 않은 매개변수도 사용할때는 넘기지 않아도 됨. 

``` ts
// index?를 하면 if (index !== undefined) 같은 쓸데없는 방어 코드를 짜게 됨.
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

``` ts
// ✅ 올바른 정의: index도 콜백함수에 우선 전달함. 
function myForEach(arr: any[], callback: (arg: any, index: number) => void) {
  callback(arr[0], 0);
}

// ⭐️ 사용하는 상황 (할당 상황)
// 받는 쪽이 "난 첫 번째 인자(arg = val)만 쓸래"라고 1개만 적어도 허용됨.
myForEach([1, 2, 3], (val) => {
  console.log(val); 
});

// callback = (val) => { console.log(val); }
// callback(arr[0], 0)를 호출할때 두번째 인자 0은 무시됨. 
```

##### 구조 분해 할당 주의 사항

- 구조 분해 할당 내부(`{ ... }`)에서 타입을 적으려고 하면 안됨. 그건 변수 이름을 바꾸는 JS 문법일 뿐임.

``` ts
// 에러 발생!
// 여기서 'string'과 'number'는 타입이 아니라 
// 'name'을 'string'이라는 이름의 변수로, 'age'를 'number'라는 변수로 바꾸겠다는 뜻입니다.
function greet({ name: string, age: number }) {
  console.log(string); 
}

/*
* 1. 인라인으로 작성
*/
function greet({ name, age }: { name: string; age: number }) {
  console.log(`안녕하세요 ${name}님, ${age}세군요!`);
}

/*
* 1. 인터페이스 활용
*/
interface User {
  name: string;
  age: number;
}

function greet({ name, age }: User) {
  console.log(name, age);
}
```

### 객체 타입

##### 인터페이스 선택 속성 

``` ts
interface PaintOptions {
  shape: Shape;   // 필수
  xPos?: number;  // 선택 (있다면 숫자여야 함)
  yPos?: number;  // 선택
}
```

- readonly를 붙이면 해당 속성을 수정 불가함. (얕은 수준에서만 작동함.)
``` ts
interface User {
  readonly id: number;      // 속성 이름 'id' 바로 앞에 위치
  name: string;             // 이건 일반 속성 (수정 가능)
}
```

##### 인덱스 시그니처

- 객체의 **속성 이름(key)을 미리 알 수 없을 때**, 어떤 타입의 키와 값을 가질지 정의하는 문법
``` ts
interface StringArray {
  [index: number]: string; // 키는 숫자, 값은 문자열인 객체 타입으로 지정
}

// myArray = { 0: "Hello", 1: "World" } 내부적으론 이런 구조임.
const myArray: StringArray = ["Hello", "World"];
```

##### 배열의 구조 

- 배열도 객체임 (인덱스 시그니처를 사용하고 있음.)
``` ts
// 우리가 보는 배열
const MyArray = [{ name: "Alice" }, { name: "Bob" }];

// 타입스크립트가 내부적으로 이해하는 구조 (인덱스 시그니처)
interface MyArrayType {
  [index: number]: { name: string; age: number }; // "어떤 숫자(index: number)를 넣어도 이 객체가 나온다"
  length: number;
  push: (...) => void;
  // ... 기타 배열 메서드들
}
```

##### extends, 교차타입(&)

- extends, 교차타입(&): 결과는 같은데 extends는 약간 상하 관계, 교차타입은 합친 느낌
``` ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}


// 1. extends: 색깔도 있고, 반지름도 있는 "색깔 있는 원" 타입을 한 번에 만듦
interface ColorfulCircle extends Colorful, Circle {}
// 2. 교차타입(&): 두 타입을 합쳐서 '색깔 있는 원' 타입을 만듭니다. 
type ColorfulCircle = Colorful & Circle;

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```

##### 구별된 유니온

``` ts
interface Circle {
  kind: "circle"; // 오직 "circle"만 가짐 (리터럴 타입)
  radius: number; // 원이라면 반지름은 필수!
}

interface Square {
  kind: "square"; // 오직 "square"만 가짐
  sideLength: number; // 사각형이라면 변의 길이는 필수!
}

type Shape = Circle | Square; // Circle 타입 또는 Square 타입 가능
```

### 제네릭 

``` ts
interface Box<Type> {
  contents: Type;
}
```

- `OrNull<Type>`: "이건 `Type`일 수도 있고 `null`일 수도 있어"
- `OneOrMany<Type>`: "이건 `Type` 하나일 수도 있고, `Type[]` 배열일 수도 있어"

##### 제네릭 제약조건(최소조건)

``` ts
// 1. 기준이 될 인터페이스를 만듭니다
interface Lengthwise {
  length: number;
}
// 2. `extends` 키워드로 제약을 겁니다
// length 속성을 가진 객체만 매개변수로 받기 때문에 함수 내에서 length를 사용 가능
function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // ✅ 이제 안전합니다!
  return arg;
}
```

##### 제네릭 타입 기본값 설정

``` ts
declare function create<
  T extends HTMLElement = HTMLDivElement, // T의 기본값은 div
  U extends HTMLElement[] = T[]           // U의 기본값은 T의 배열
>(
  element?: T,
  children?: U
): Container<T, U>;

// 1. 아무것도 안 넘기면? T는 div, U는 div[]로 자동 결정
// div는 div, div[]를 매개변수로 받는 함수가 된 것임
const div = create(); 
// 2. p 태그를 넘기면? T는 p, U는 p[]로 자동 결정
const p = create(new HTMLParagraphElement());
```



### 연산자
##### `keyof` 연산자

- 타입이 가진 모든 키를 뽑아서 유니온 타입으로 만듬. (예: `"a" | "b"`)
``` ts
type Point = { x: number; y: number };
type P = keyof Point; 
// 결과: type P = "x" | "y"
```

- 객체의 키로 제약조건 설정하기
``` ts
// Type 객체의 Key들의 타입만 key에 허용됨
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
```
- **`Type`**: 함수가 받는 객체의 타입입니다.
- **`keyof Type`**: `Type` 객체가 가지고 있는 모든 키들을 **유니온 타입(예: "a" | "b" | "c")**으로 뽑아냅니다.
- **`Key extends keyof Type`**: `Key`라는 타입 변수가 반드시 `keyof Type` 중 하나에 속해야 한다고 제약을 겁니다.


##### `typeof` 연산자

- 객체의 타입을 추출할 수 있음.
- **이미 만들어진 객체나 함수의 타입을 다시 정의하기 귀찮을 때** 아주 유용하게 쓰이는 도구

``` ts
/*
* 0. 기본 사용
*/
let s = "hello";
let n: typeof s; // n의 타입은 string이 됩니다.

/*
* 1. 함수의 반환 타입 추출 (ReturnType 사용)
*/
function f() {
  return { x: 10, y: 3 };
}

type P = ReturnType<typeof f>; // 결과: type P = { x: number; y: number; }

/*
* 2. 주로 초기값이나 설정 파일에서 타입을 역으로 추출할 때
*/
const initialState = {
  isLoggedIn: false,
  user: { name: "", email: "" },
  theme: "dark" as const
};

// initialState의 구조를 일일이 인터페이스로 만들 필요 없이 바로 추출!
type RootState = typeof initialState;

/* RootState는 자동으로 아래와 같이 정의됩니다:
{
  isLoggedIn: boolean;
  user: { name: string; email: string; };
  theme: "dark";
}
*/
```


##### `...` 연산자

1. **전개 연산자** (Spread Operator): 이미 묶여 있는 배열이나 객체를 개별적인 값으로 **풀어헤칠 때** 씁니다.
``` ts
const numbers = [1, 2, 3];
console.log(...numbers); // console.log(1, 2, 3)과 똑같이 동작합니다.
```

2. **나머지 매개변수** (Rest Parameters): 따로따로 들어오는 여러 개의 값을 하나의 배열로 **합칠 때** 씁니다.
``` ts
// 낱개로 들어오는 값들을 'args'라는 이름의 배열로 모읍니다.
function printAll(...args) {
  console.log(args); // ["A", "B", "C"] (배열이 됨!)
}

printAll("A", "B", "C");
```

3. 구조 분해 할당에서의 나머지
``` ts
const [first, ...others] = [10, 20, 30, 40];

console.log(first);  // 10
console.log(others); // [20, 30, 40] (첫 번째 빼고 '나머지'를 다 모음)
```