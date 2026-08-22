### 틱텍토 게임

##### 히스토리 추가 전

``` jsx
import { useState } from 'react';

function Square({value, onSquareClick}) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

export default function Board() {
  const [xIsNext, setXIsNext] = useState(true);
  const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    if (calculateWinner(squares) || squares[i]) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = 'X';
    } else {
      nextSquares[i] = 'O';
    }
    setSquares(nextSquares);
    setXIsNext(!xIsNext);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = 'Winner: ' + winner;
  } else {
    status = 'Next player: ' + (xIsNext ? 'X' : 'O');
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}
```

##### 히스토리 추가

``` jsx
import { useState } from "react";

function Square({ value, onSquareClick }) {
  return (
    <button className="square" onClick={onSquareClick}>
      {value}
    </button>
  );
}

function Board({ xIsNext, squares, onPlay }) {
  // const [xIsNext, setXIsNext] = useState(true);
  // const [squares, setSquares] = useState(Array(9).fill(null));

  function handleClick(i) {
    if (squares[i] || calculateWinner(squares)) {
      return;
    }
    const nextSquares = squares.slice();
    if (xIsNext) {
      nextSquares[i] = "X";
    } else {
      nextSquares[i] = "O";
    }
    onPlay(nextSquares);
  }

  const winner = calculateWinner(squares);
  let status;
  if (winner) {
    status = "Winner: " + winner;
  } else {
    status = "Next player: " + (xIsNext ? "X" : "O");
  }

  return (
    <>
      <div className="status">{status}</div>
      <div className="board-row">
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        <Square value={squares[1]} onSquareClick={() => handleClick(1)} />
        <Square value={squares[2]} onSquareClick={() => handleClick(2)} />
      </div>
      <div className="board-row">
        <Square value={squares[3]} onSquareClick={() => handleClick(3)} />
        <Square value={squares[4]} onSquareClick={() => handleClick(4)} />
        <Square value={squares[5]} onSquareClick={() => handleClick(5)} />
      </div>
      <div className="board-row">
        <Square value={squares[6]} onSquareClick={() => handleClick(6)} />
        <Square value={squares[7]} onSquareClick={() => handleClick(7)} />
        <Square value={squares[8]} onSquareClick={() => handleClick(8)} />
      </div>
    </>
  );
}

export default function Game() {
  const [history, setHistory] = useState([Array(9).fill(null)]);
  const [currentMove, setCurrentMove] = useState(0);
  const xIsNext = currentMove % 2 === 0;
  const currentSquares = history[currentMove];

  function handlePlay(nextSquares) {
    const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
    setHistory(nextHistory);
    setCurrentMove(nextHistory.length - 1);
  }

  function jumpTo(nextMove) {
    setCurrentMove(nextMove);
  }

  const moves = history.map((squares, move) => {
    let description;
    if (move > 0) {
      description = "Go to move #" + move;
    } else {
      description = "Go to game start";
    }
    return (
      <li key={move}>
        <button onClick={() => jumpTo(move)}>{description}</button>
      </li>
    );
  });

  return (
    <div className="game">
      <div className="game-board">
        <Board xIsNext={xIsNext} squares={currentSquares} onPlay={handlePlay} />
      </div>
      <div className="game-info">
        <ol>{moves}</ol>
      </div>
    </div>
  );
}

function calculateWinner(squares) {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
}

```


### 리액트 컴포넌트 

- React 컴포넌트는 _리액트가 브라우저에 마크업을 렌더링할 수 있는 JavaScript_ 함수
- 대문자로 시작해야함. 
``` jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

| Syntax  | Export 구문                           | Import 구문                             |
| ------- | ----------------------------------- | ------------------------------------- |
| Default | export default function Button() {} | import Button from './button.js';     |
| Named   | export function Button() {}         | import { Button } from './button.js'; |
- Default import의 경우에 다른 이름으로 값을 가져올 수 있음.

- React는 `false`를 `null` 또는 `undefined`처럼 JSX 트리의 “구멍”으로 간주하고 그 자리에 아무것도 렌더링하지 않음. (0은 렌더링 돼서 조심해야함. )


### JSX

- JSX: JavaScript를 확장한 문법으로, JavaScript 파일을 HTML과 비슷하게 마크업을 작성할 수 있도록 해줌.
- 컴파일러가 JS 객체로 변환함. 

1. 개발자: JSX 작성 (`<button>클릭</button>`)
2. 컴파일러: JS 객체로 변환 (`{ type: 'button', props: { children: '클릭' } }`)
3. 리액트: 이 객체들을 모아 **가상 DOM** 트리 구성
	- 리액트는 두 개의 가상 DOM을 관리하고 데이터가 바뀔때 원본을 보고 최소한의 업데이트만 하도록 함. 
		- **Current Tree (현재 트리):** 지금 브라우저 화면에 실제로 나타나 있는 UI의 설계도
		- **Work-in-Progress Tree (작업용 트리):** 상태가 변했을 때, "다음엔 이렇게 그려야지" 하고 새롭게 만든 설계도
4. ReactDOM: 실제 브라우저에 **HTML 요소** 생성 및 삽입

- JSX 문법
``` jsx
<article> {/* 1. HTML 세상 */}
  { /* 2. { 중괄호 } 열고 JS 세상(map)으로 입장 */
    poem.lines.map((line) => (
      /* 3. 다시 HTML(JSX) 값을 반환! */
      <p>{line}</p> 
    ))
  }
</article>
```

### 리액트로 사고하기

1. UI를 컴포넌트 계층으로 쪼개기
2. React로 정적인 버전 구현하기 
	- 정적인 데이터는 State를 사용하지 않고 Props로 데이터를 넘겨줌.
3. 최소한의 데이터만 이용해서 완벽하게 UI State 표현하기
	- 최소한의 State를 도입하고 나머지는 필요에 따라 실시간으로 계산
	- 시간이 지나거나 하면 데이터가 변하는 것이 State임.
4. State가 어디에 있어야 할 지 정하기
	- State를 사용하는 공통 부모, 공통 부모 상위 컴포넌트
	- State를 소유할 적절한 컴포넌트를 찾지 못했다면, State를 소유하는 컴포넌트를 하나 만들어서 상위 계층에 추가
5. 역 데이터 흐름 추가하기
	- `setXXX`를 하위 컴포넌트로 넘기고 하위 컴포넌트는 `setXXX`를 통해서 State를 업데이트 할 수 있음.

- 선언형 UI: 컴포넌트에서 상태에 따라서 어떻게 무엇을 보여줄지 구분해서 선언해두고 상태를 변경시키는 구조. 


### props

``` jsx
// 1. 구조 분해 할당 (권장)
function Avatar({ person, size }) {
  // ...
}
// 2. 한 번에 받기 ({person:..., size:...})
function Avatar(props) {
  let person = props.person;
  let size = props.size;
  // ...
}
```

- 기본값: 데이터가 없거나 undefined로 전달될 때 사용됨
``` jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

- Props는 컴포넌트의 데이터를 처음에만 반영하는 것이 아니라 모든 시점에 반영합니다.
- 그러나 props 변경을 시도하지 말고 사용자의 입력에 반응해야하는 경우에 State를 사용하는 게 정석
``` jsx
// 시간마다 바뀐 time의 값이 전달되고 color를 바꾸면 색상이 바뀜.
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

- 자식 컴포넌트 children
``` jsx
import Avatar from './Avatar.js';

// <Avatar />가 children으로 들어옴. 
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```


### State

- 특정 시점의 화면 모습을 결정하는 **데이터의 스냅샷**이자, 변화 시 화면을 새로 그리게 만드는 **리렌더링 트리거**
- 컴포넌트 별로 관리됨. 

##### 일반 변수 vs State

- 지역 변수 (State 아님)
	1. **지역 변수는 렌더링 간에 유지되지 않음.** 새로 렌더링 되면 초기화 되는 것임
	2. 지역 변수를 변경해도 **렌더링을 일으키지 않음.** 
- State
	1. 렌더링 간에 데이터를 유지하기 위한 **state 변수**.
	2. 변수를 업데이트하고 React가 컴포넌트를 다시 렌더링하도록 유발하는 **state setter 함수** 
		- 바로 렌더링 되는 것이 아니라 렌더링 대기열에 추가 되는 것

##### 스냅샷

- State는 스냅샷과 유사하게 동작하기 때문에 state 변수를 변경해도 기존의 state 변수는 변경되지 않고 리렌더링을 유발함. 
``` jsx
function Counter() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    // 1. 현재 렌더링 시점의 count는 0입니다.
    console.log('실행 직후:', count); // 출력: 0

    // 2. 리액트에게 "다음번에 1로 그려줘"라고 업데이트 큐에 추가합니다. 
    setCount(count + 1); 

    // 3. 하지만 이 함수 내의 count는 여전히 0입니다 (스냅샷).
    console.log('setCount 호출 직후:', count); // 출력: 0
    
  };
  // 4. 이 이벤트 핸들러가 실행된 후에 리렌더링이 발생해서 새로운 스냅샷을 만드는데 큐를 처리합니다. 
```

##### 객체 

- 객체와 배열을 변경할 경우에 새로운 객체나 기존 객체의 복사본을 만들어서 **통째로 업데이트**해야함. (리액트는 객체 참조값만 비교하기 때문)
	- 객체 전개 구문 `...`을 사용하면 모든 프로퍼티를 각각 복사하지 않고 새로운 객체를 만들 수 있음.
- *주의: `...` 전개 문법은 얕은 복사를 수행하므로 한 레벨 깊이의 내용만 복사함.* 내부의 객체는 기존의 주소값만 복사됨. 그래서 중첩된 경우에는 `...`를 한번 더 사용해줘야함. 

``` jsx
const user = {
  name: 'Gemini',
  address: {
    city: 'Seoul',
    zip: '12345'
  }
};

setUser({
  ...user, // 다른 필드 복사
  address: { // address 교체
    ...user.address, // 기존의 필드 사용
    city: 'Busan'    // 새로운 값 교체
  }
});
```

- 혹은 `useImmer`를 사용할 수도 있음. 
``` jsx
import { useImmer } from 'use-immer';

// ... (컴포넌트 내부)
const [user, updateUser] = useImmer({
  name: 'Gemini',
  address: {
    city: 'Seoul',
    zip: '12345'
  }
});

function handleUpdate() {
  updateUser(draft => {
    // 중첩된 구조라도 그냥 마침표(.)로 접근해서 바로 대입하세요!
    draft.address.city = 'Busan';
  });
}
```

##### 배열

- 객체와 마찬가지로 새 배열을 state 설정 함수에 전달해서 업데이트해야함. 
- `filter()`, `map()`, `slice()`, `[...arr]`를 사용하여 원본 배열로부터 새 배열을 만들 수 있음. (선호)
- `push`, `pop`, `reverse`, `sort`는 기존 배열을 변경함. (비선호)

- *주의: `filter()`, `map()`, `slice()`, `[...arr]` 등은 얕은 복사를 수행하므로 한 레벨 깊이의 내용만 복사함.* 
- 중첩된 내부 객체도 새로 만들어서 교체해야함. 
- `useImmer`를 사용할 수도 있음. 

``` jsx
import { useImmer } from 'use-immer';
...
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];
...
const [myList, updateMyList] = useImmer(initialList);
...
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

##### 구조화 원칙

1. **연관된 state 그룹화하기.** 두 개 이상의 state 변수를 항상 동시에 업데이트한다면, 단일 state 변수로 병합하는 것을 고려하세요.
2. **State의 모순 피하기.** 여러 state 조각이 서로 모순되고 “불일치”할 수 있는 방식으로 state를 구성하는 것은 실수가 발생할 여지를 만듭니다. 이를 피하세요.
	- 예: `setIsSent`와 `setIsSending`을 함께 호출하는 것을 잊어버린 경우, `isSending`과 `isSent`가 동시에 `true`인 상황에 처할 수 있는데 이건 모순임.
3. **불필요한 state 피하기.** 렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면, 컴포넌트의 state에 해당 정보를 넣지 않아야 합니다.
4. **State의 중복 피하기.** 여러 상태 변수 간 또는 중첩된 객체 내에서 동일한 데이터가 중복될 경우 동기화를 유지하기가 어렵습니다. 가능하다면 중복을 줄이세요.
	- 예: `items`와 `selectedItem`를 둘 다 저장할 필요가 없음. `selectedItem` 대신 `selectedId`를 저장해서 `itmes`에서 조회하는 방식이 나음.
5. **깊게 중첩된 state 피하기.** 깊게 계층화된 state는 업데이트하기 쉽지 않습니다. 가능하면 state를 평탄한 방식으로 구성하는 것이 좋습니다.

##### State 끌어올리기

*여러 자식 컴포넌트에서 데이터를 수집하거나 두 자식 컴포넌트가 서로 통신하도록 하려면, 부모 컴포넌트에서 공유 State를 선언하세요. 부모 컴포넌트는 Props를 통해 해당 State를 자식 컴포넌트에 전달할 수 있습니다. 이렇게 하면 자식 컴포넌트가 서로 동기화되고 부모 컴포넌트와도 동기화되도록 유지할 수 있습니다.*

- 각각의 MyButton 컴포넌트가 별도의 State(count)를 가짐. 
``` jsx
import { useState } from 'react';

export default function MyApp() {
  return (
    <div>
      <h1>Counters that update separately</h1>
      <MyButton />
      <MyButton />
    </div>
  );
}

function MyButton() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      Clicked {count} times
    </button>
  );
}
```

- State를 MyApp 컴포넌트로 끌어올리고 MyButton 컴포넌트에 props(프로퍼티)를 내려주는 방식
``` jsx
import { useState } from 'react';

export default function MyApp() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <div>
      <h1>Counters that update together</h1>
      <MyButton count={count} onClick={handleClick} />
      <MyButton count={count} onClick={handleClick} />
    </div>
  );
}

function MyButton({ count, onClick }) {
  return (
    <button onClick={onClick}>
      Clicked {count} times
    </button>
  );
}

```

##### State 보존 및 초기화

리액트는 UI 안에 있는 컴포넌트 구조를 렌더 트리로 만들고 UI 트리에 있는 **위치를 이용해서** 리액트가 가지고 있는 state를 알맞는 컴포넌트와 연결함.

1. 같은 자리의 같은 컴포넌트는 state를 보존
	예: `isFancy ? <Counter isFancy={true} /> : <Counter isFancy={false} />`
2. 같은 위치의 다른 컴포넌트는 state를 초기화
	예: `isFancy ? <div><Counter /></div> : <section><Counter /></section>`
3. 같은 위치에서 state를 초기화 하는 방법 
	- 다른 위치에 컴포넌트를 렌더링하기
		- `{ isPlayerA && <Counter person="Taylor" /> }{ !isPlayerA &&<Counter person="Sarah" /> }`
	-  key를 이용해 state를 초기화하기


### 이벤트 핸들러

`<button>`과 같은 내장 컴포넌트는 `onClick`과 같은 내장 브라우저 이벤트만 지원.
반면 사용자 정의 컴포넌트를 생성하는 경우, 컴포넌트 이벤트 핸들러 props의 역할에 맞는 원하는 이름을 사용할 수 있음.

이벤트를 전달할때 주의점: 이벤트에 파라미터를 넣어서 넘기면 작동하지 않음
``` jsx
<Square value={squares[0]} onSquareClick={handleSquareClick(0)} />
```
이유는 다음과 같습니다. `handleClick(0)` 호출은 보드 컴포넌트 렌더링의 일부가 됩니다. `handleClick(0)`은 `setSquares`를 호출하여 보드 컴포넌트의 State를 변경하기 때문에 보드 컴포넌트 전체가 다시 렌더링 됩니다. 하지만 이 과정에서 `handleClick(0)`은 다시 실행되기 때문에 무한 루프에 빠지게 됩니다.
해결방안: `() => handleClick(0)` 함수를 넘기도록 수정
``` jsx
<Square value={squares[0]} onSquareClick={() => handleSquareClick(0)} />
```

**이벤트 전파: 자식 컴포넌트에서 이벤트가 발생한 경우 (예: onClick) 부모 컴포넌트까지 이벤트가 전파됨.** 
이벤트 전파을 하기 싫다면 자식 컴포넌트에서 `e.stopPropagation()`를 사용하여 전달을 전파를 막을 수 있음.


### 리스트 렌더링 주의점

리액트에서는 `<li>` 태그 같이 여러개의 같은 종류의 엘리먼트가 나열될때 자동으로 Key를 사용하는데 동적 리스트의 경우 배열 인덱스를 Key로 사용함. 하지만 배열 인덱스는 계속 바뀌기 때문에 경고창을 띄워주고 에러가 발생할 수 있음.
그러나 배열의 정렬의 정렬이 바뀌지 않고 새로운 값이 끝에서만 추가된다면 Key로 배열 인덱스를 명시해줌으로서 사용 가능함. 

배열 안에 담긴 JSX 요소인 경우에는 모두 Key가 필요함.

`key={Math.random()}`처럼 즉석에서 key를 생성하면 안됨. 이렇게 하면 렌더링 간에 key가 일치하지 않아 모든 컴포넌트와 DOM이 매번 다시 생성될 수 있음.

`map()` 호출 내부의 JSX 엘리먼트에는 항상 key가 필요
``` jsx
<div>
	<h1>Recipes</h1>
    {recipes.map(recipe =>
	    <Recipe {...recipe} key={recipe.id} />
    )}
</div>
```

### Hook 타입 정의

**주의: 모든 Hook은 컴포넌트 최상단에서 호출되어야 함.**

##### useState

useState를 통해서 데이터를 관리 (컴포넌트별 메모리)
useState는 컴포넌트가 다시 실행되어도 데이터를 잊지 않게 보관하면서, 값이 바뀔때마다 화면을 자동으로 새로 고침해줌. (setXXX를 사용하면)

`const [squares, setSquares] = useState(Array(9).fill(null));`에서 `squareds[0] = "X"` 이런식으로 직접 값을 바꾸면 안되고 `setSquares`를 사용해야함. 직접 값을 바꾸면 리렌더링이 발생하지 않아서 사용자 UI가 업데이트가 안됨. 

**렌더링 중에 State를 업데이트 하는 set 메서드를 사용하면 안됨**. 무한루프에 빠질 수 있음. 
그러나 조건물을 걸어서 딱 한번만 호출되도록 하면 가능함. 

##### useReducer

- State를 업데이트하는 모든 로직을 Reducer 함수로 통합해 관리할 수 있음.
- 여러 상태들이 연관되어 한번에 바뀌어야 할 때
- 다음 상태가 이전 상태에 의존할 때 

``` tsx
// 1. 상태 타입 정의
type RequestState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success', data: any }
  | { status: 'error', error: Error };

// 2. 리듀서 함수 (매뉴얼): 현재 State 값과 Action 객체를 받고 다음 State 값을 반환
function requestReducer(state: RequestState, action: any): RequestState {
  switch (action.type) {
    case 'FETCH_START':
      return { status: 'loading' };
    case 'FETCH_SUCCESS':
      return { status: 'success', data: action.payload };
    case 'FETCH_ERROR':
      return { status: 'error', error: action.error };
    default:
      return state;
  }
}

// 3. 컴포넌트 내부에서 사용
const [state, dispatch] = useReducer<RequestState>(requestReducer, { status: 'idle' });

// 4. 호출할 때
dispatch({ type: 'FETCH_START' });
```

##### useContext

- 하위 컴포넌트에 중간 단계를 건너뛰고 Prop 데이터를 전달해야할 때
- 하위 컴포넌트에서 상위 컴포넌트에 Prop 데이터를 요청할 때

``` tsx
import { createContext, useContext, useState } from 'react';

type Theme = "light" | "dark" | "system";

// 1. 데이터를 담을 컨텍스트 생성, 기본값을 인자로 받음
const ThemeContext = createContext<Theme>("system");

// 2. 데이터를 가져올 메서드 생성
const useGetTheme = () => useContext(ThemeContext);


export default function MyApp() {
  const [theme, setTheme] = useState<Theme>('light');

  return (
	// 3. 데이터를 담은 컨텍스트로 감싸면 그 하위에 있는 컴포넌트는 데이터에 접근 가능
    <ThemeContext value={theme}>
      <MyComponent />
    </ThemeContext>
  )
}

function MyComponent() {
  // 4. 데이터 가져오기
  const theme = useGetTheme();

  return (
    <div>
      <p>Current theme: {theme}</p>
    </div>
  )
}

```

##### Reducer + Context

``` jsx
import { createContext, useContext, useReducer } from 'react';

// State 컨텍스트 생성
const TasksContext = createContext(null);
// dispatch 컨텍스트 생성
const TasksDispatchContext = createContext(null);

/*
 * 프로바이더 생성
 */
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer,initialTasks);

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

// 사용자 Hook 생성 (모든 Context 문법을 이 파일에 두기 위함.)
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

// 리듀서 함수 생성 
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

##### useMemo

- `useMemo()`를 사용하면 리렌더링 시에 지정한 데이터가 변할때만 새로운 계산을 하도록함. 
- 두 번째 인자에서 어떤 데이터가 변할 때 새로 계산을 할 것인지 지정
- *그러나 최근 리액트 컴파일러에서는 useMemo 대신 자동으로 최적화를 시켜주기 때문에 useMemo를 사용할 일이 없음.*

``` jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ todos나 filter가 변경되지 않는 한 getFilteredTodos()를 다시 실행하지 않습니다.
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

##### useCallback

- `useMemo()`와 동일하지만 저장하는 함수를 저장함.
- 함수도 객체이기 때문에 리렌더링 될때마다 다시 생성되는데 이 함수를 받는 자식 컴포넌트가 있으면 불필요하게 다시 그려지게 됨. 
- *그러나 최근 리액트 컴파일러에서는 useCallback 대신 자동으로 최적화를 시켜주기 때문에 useCallback를 사용할 일이 없음.*

##### React.memo

- 컴포넌트 자체를 메모이제이션 함. 
- 부모 컴포넌트가 리렌더링 되어도 Prop 데이터가 변하지 않았으면 기존에 메모이제이션된 값을 사용
- Prop 중 하나라도 바뀌면 원래처럼 전체 컴포넌트 리렌더링

``` tsx
import { useMemo, useCallback, memo } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ data, onClick }) {

  const processedData = useMemo(() => {
    return expensiveProcessing(data);
  }, [data]);

  const handleClick = useCallback((item) => {
    onClick(item.id);
  }, [onClick]);

  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} onClick={() => handleClick(item)} />
      ))}
    </div>
  );
});
```


##### useSyncExternalStore

- 외부 저장소를 구독하기 위한 Hook 
``` jsx
/*
* 1. 구독 등록 함수를 통해서 이벤트가 등록됨. 
* 2. 만약 online, offline이 되면 callback 함수 실행
* 3. callback 함수가 실행되면 기존의 스냅샷과 지금의 스냅샷을 비교해서 갱신
*/

// callback은 리액트가 내부적으로 만든 화면 갱신용 함수가 useSyncExternalStore의 구독 등록 함수의 인자로 주어짐.
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {
  // 내장 Hook으로 외부 스토어 구독하기
  return useSyncExternalStore(
    subscribe, // 1. 구독 등록 함수 지정: 이 함수를 실행해서 구독을 등록, 정리 함수 등록
    () => navigator.onLine, // 2. 스냅샷: 클라이언트에서 값을 얻는 방법
    () => true // 3. 서버스냅샷: 서버에서 값을 얻는 방법
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```
1.  첫 번째 렌더링 (화면 그리기): 컴포넌트가 처음 실행될 때입니다.
	1. `ChatIndicator`가 호출됩니다.
	2. `useOnlineStatus()`가 실행됩니다.
	3. `useSyncExternalStore`가 **세 번째 인자(서버 스냅샷, `() => true`)**를 실행해서 "일단은 온라인 상태(`true`)군!" 하고 값을 가져옵니다.
	4. 리액트가 이 값을 바탕으로 화면을 먼저 그립니다. (이때까지는 `subscribe` 함수가 실행되지 않습니다.)

2.  구독 등록 (`subscribe` 실행): 화면이 브라우저에 딱 그려진 **직후**입니다. (마치 `useEffect`처럼요)
	1. 리액트가 **첫 번째 인자인 `subscribe` 함수를 실행**합니다.
	2. 브라우저 `window`에 'online', 'offline' 이벤트 리스너가 등록됩니다.
	3. 동시에, **두 번째 인자인 `() => navigator.onLine` 을 즉시 실행**해서 아까 그린 `true`가 진짜 맞는지 확인합니다. 만약 현재 오프라인(`false`)이라면 여기서 화면을 바로 다시 그립니다.
    
3. 이벤트 발생 (상태 변화): 이제 사용자가 랜선을 뽑거나 와이파이를 켰을 때입니다.
	1. 브라우저가 이벤트를 감지하고 `subscribe` 안에 등록했던 `callback` 함수를 실행합니다.
	2. 이 `callback`은 리액트에게 "야, 데이터 바뀌었으니까 확인해봐!"라고 신호를 보냅니다.
	3. 리액트는 다시 **두 번째 인자(`() => navigator.onLine`)를 실행**해서 최신 상태(예: `false`)를 가져온 뒤 리렌더링합니다.

##### useRef

``` jsx
import { useRef } from 'react';

// 생성 { current: 0 }
const ref = useRef(0);
// 사용
ref.current
```

- State처럼 렌더링 간에 정보를 유지할 수 있음. ㅜㅇㄱ
- State와 달리 값이 변해도 **리렌더링을 발생시키지 않음.** 
- 문자열, 객체, 함수 등 모든 것을 저장할 수 있음.

- *주의: 렌더링 단계에서 `ref.current`를 읽거나 쓰면 안됨.* 컴포넌트를 순수하게 유지해야함. 렌더링 중에 일부 정보가 필요한 경우 State를 사용
	- **이벤트 핸들러**와 **useEffect**에서만 사용해야 안전함. 

- DOM 노드 접근
- DOM 노드의 이벤트 핸들러(`addEventListener`)나 내장 브라우저 API를 사용할 수 있음. 
- DOM 노드를 직접 변경하려고 하면 안됨. (수정, 삭제, 자식 추가 등)
- *주의: 렌더링 단계에서 DOM 노드에 접근하면 안됨. 아직 DOM이 없음.*
``` jsx
const myRef = useRef(null);

// ref에 DOM 노드에 대한 참조 가져오기
<div ref={myRef}>...</div>
// 사용 (브라우저 API)
myRef.current.scrollIntoView();
```

- Ref 콜백
``` jsx
<div>
    <ul>
	    {catList.map((cat) => (
            <li
              key={cat.id}
              ref={(node) => {
                // 1. 노드가 화면에 나타날 때 (Mount) 
                // node 변수에는 실제 <li> DOM 요소가 들어옵니다.
                const map = getMap();
                map.set(cat, node);
                
                // 2. 노드가 화면에서 사라질 때 (Unmount) 실행될 정리 함수를 리턴
                return () => {
                  map.delete(cat);
                };
              }}
            >
              <img src={cat.imageUrl} />
            </li>
        ))}
    </ul>
</div>
```


##### 커스텀 Hook

- 내부적으로 리액트 내장 Hook을 하나라도 사용해야함.
- 함수 이름 앞에 use를 붙여야함. 


##### forwardRef

리액트는 보통 부모가 자식에게 props를 내려줘서 자식이 렌더링되게 하는 선언적 방식을 사용함. 
하지만 **부모가 자식의 특정 함수를 사용하도록 해야할 때**가 가끔 있음. 
리액트 19부터는 `forwardRef`를 사용하지 않아도 `ref`를 props로 받을 수 있음.

1. `forwardRef`
	- 역할: 부모 컴포넌트가 던져주는 `ref`(리모컨)를 자식 컴포넌트가 받을 수 있도록 허용해주는 래퍼(Wrapper)
	- 기본적으로 리액트 컴포넌트는 `ref`를 props로 받을 수 없게 막혀 있음. 그걸 허용해줘야함. (리액트 19부터는 허용됨.) 
2. `useImperativeHandle`
	- 역할: `forwardRef`로 받아온 `ref` 객체 안에 부모가 사용할 수 있는 함수들을 지정함. 

``` tsx
// 자식 컴포넌트 
// 1. 부모가 자식의 어떤 기능을 쓸지 '명세서'를 만듭니다.
export interface MetronomeRef {
  startSync: (beatsArray: number[], syncOffset: number, currentTime: number) => Promise<void>;
  stopSync: () => void;
}

interface MetronomeProps {
  bpm: number;
  onBpmChange: (newBpm: number) => void;
}

// 2. forwardRef로 감싸서 부모의 ref를 받아옵니다.
const Metronome = forwardRef<MetronomeRef, MetronomeProps>(({bpm, onBpmChange,}, ref) => {
  
  // 3. 부모가 ref.current.startSync()를 호출하면 실행될 내용을 정의합니다.
  useImperativeHandle(ref, () => ({
    startSync: async (beats, offset, time) => {
      await engineRef.current?.start(beats, offset, time);
      setIsPlaying(true);
    },
    stopSync: () => {
      engineRef.current?.stop();
      setIsPlaying(false);
    }
  }));

  return <div>메트로놈 화면</div>;
});

export default Metronome;
```

``` tsx
// 부모 컴포넌트 
import Metronome, { MetronomeRef } from "./Metronome";

const metronomeRef = useRef<MetronomeRef>(null);

// 부모가 버튼을 눌러 자식의 함수를 직접 실행!
<button onClick={() => metronomeRef.current.startSync(...) }>시작</button>
<Metronome ref={metronomeRef} />
```


##### useImperativeHandle

**부모 컴포넌트가 자식 컴포넌트의 메서드를 실행해야할 때** 사용
리액트 19부터는 `forwardRef`를 사용하지 않아도 `ref`를 props로 받을 수 있음.

1. 자식 컴포넌트 
``` tsx
export interface YoutubePlayerRef {
    getCurrentTime: () => number;
}

interface YoutubePlayerProps {
    videoId: string;
    onVideoStartAction: (videoCurrentTime: number) => void;
    onVideoStopAction: () => void;
    ref?: React.Ref<YoutubePlayerRef>;
}

export default function YoutubePlayer({
    videoId,
    onVideoStartAction,
    onVideoStopAction,
    ref
}: YoutubePlayerProps
) {
	...
    useImperativeHandle(ref, () => ({
        getCurrentTime: () => {
            if (playerRef.current) {
                return playerRef.current.getCurrentTime() || 0;
            }
            return 0;
        }
    }));
    ...
}	
```

2. 부모 컴포넌트 
``` tsx
export default function YoutubeSync({
  isSyncMode,
  onBpmChange,
  ...,
  ref
}: YoutubeSyncProps
) {

  const playerRef = useRef<YoutubePlayerRef>(null);
  
  // playerRef.current.getCurrentTiem로 자식 컴포넌트의 메서드를 사용 가능
  ...
  
  return (
		    <YoutubePlayer ref={playerRef} />
  )
}
```


### 유용한 타입들

##### DOM 이벤트

``` tsx
import { useState } from 'react';

export default function Form() {
  const [value, setValue] = useState("Change me");

  // event의 타입을 Input HTML 태그에서 change 관련 이벤트 타입으로 지정
  function handleChange(event: React.ChangeEvent<HTMLInputElement>) {
    setValue(event.currentTarget.value);
  }

  return (
    <>
      <input value={value} onChange={handleChange} />
      <p>Value: {value}</p>
    </>
  );
}
```

- 리액트에서는 `React.{이벤트}` 형식으로 DOM 이벤트를 지정할 수 있음
- 타입스크립트에서 저렇게 `event` 파라미터의 타입 좁히기를 하지 않으면 `event.currentTarget.value` 처럼 접근하는 것이 불가능함. 

##### Children 속성

- 리액트에서 `children`은 **컴포넌트의 열기 태그와 닫기 태그 사이에 들어가는 내용**을 자동으로 받아오는 특수한 props입니다.

``` tsx
interface ModalRendererProps {
  title: string;
  children: React.ReactNode; // 1. 모든 타입 허용
}

interface ModalRendererProps {
  title: string;
  children: React.ReactElement; // 2. 리액트가 만드는 타입만 하용 (예: JSX 요소(객체))
}
```

``` tsx
function ModalRenderer({ title, children }: ModalRendererProps) { 
  return (
    <div className="modal">
      <h1>{title}</h1> 
      <div className="content">
        {children} {/* 호출 시 태그 사이에 넣었던 <p>와 <button>이 이 자리에 치환됨 */}
      </div>
    </div>
  ); 
}

// 컴포넌트 사용(호출) 예시
<ModalRenderer title="로그인 알림">
  {/* 여기서부터 */}
  <p>아이디와 비밀번호를 확인해주세요.</p>
  <button>확인</button>
  {/* 여기까지가 태그 사이에 넣은 내용이며, 이것이 'children'으로 전달됨 */}
</ModalRenderer>
```

##### Style 속성

``` tsx
import React from 'react';

interface BoxProps {
  label: string;
  // 외부에서 스타일 객체를 통째로 넘겨받을 수 있도록 정의
  containerStyle?: React.CSSProperties; 
}

function Box({ label, containerStyle }: BoxProps) {
  return (
    <div style={containerStyle}> {/* 주입받은 스타일을 인라인으로 적용 */}
      {label}
    </div>
  );
}

// 사용 시
<Box 
  label="확인" 
  containerStyle={{ backgroundColor: 'blue', marginTop: '10px' }} 
/>
```


### 렌더링

-  렌더링과 반영
	1. 렌더링 **유발** (주방에 식사 주문을 전달하기)
		- 컴포넌트의 초기 렌더링인 경우
		- state가 업데이트된 경우: 바로 렌더링 되는 것이 아니라 렌더링 대기열에 추가 되는 것
	2. 컴포넌트 **렌더링** (주방에서 주문을 준비하기): 가상 DOM(새로운 스냅샷)을 만들어서 이전 스냅샷과 비교해 무엇이 바뀌어야 할지 계산하는 작업
	3. DOM에 **반영** (주문을 테이블에 서빙하기)

- 전체 프로세스 
	1. 렌더링 유발 (setState 등)
	2. React가 **컴포넌트 함수를 다시 호출**합니다.
	3. 함수가 새로운 **JSX 스냅샷 (가상 DOM)** 을 생성
	4. 이전 JSX 스냅샷과 새로운 JSX 스냅샷을 비교
	5. 실제 브라우저 DOM에 새로운 JSX 스냅샷과 일치하도록 변경 사항 반영
	6. 브라우저가 화면을 다시 그림


### 리렌더링

1. useState, useReducer, useContext로 관리하는 데이터가 업데이트 되었을 경우 리렌더링 (**이벤트 핸들러의 모든 코드가 실행된 후**)
2. 1번에 해당하는 컴포넌트의 자식 컴포넌트들 전체 리렌더링

useMemo, useCallback, React.memo를 사용하면 자식 컴포넌트에서 어떤 데이터가 업데이트 될때 리렌더링을 할 것인지 수동으로 지정해서 전체 리렌더링을 막을 수 있음. 
*그러나 최신에는 리액트 컴파일러가 그 역할을 자동으로 대신함.*


### 리액트 컴파일러 

- 수동 메모이제이션(useMemo, useCallback, React.memo)을 대신해서 자동으로 메모이제이션을 수행해줌. 
- 리액트 컴파일러는 리액트 **컴포넌트 내부의 데이터들만** 메모이제이션 함. 
- 컴포넌트별 개별 저장소에 저장되므로 다른 컴포넌트끼리 공유가 되는 것은 아님. 
- 리렌더링이 실행될때 리액트 컴포넌트 내부의 함수들의 입력값을 메모이제이션 되어있는 **입력값과 비교**해 같은 경우 기존의 데이터를 사용함. 
- 메모이제이션은 컴파일러에 의존하고 정밀한 제어가 필요한 곳에서 `useMemo`/`useCallback`을 사용하는 것을 권장합니다.

1. 기존 리액트의 리렌더링 
	부모 컴포넌트나 내 상태가 바뀌면 리액트는 **"불안하니까 처음부터 끝까지 다 다시 확인해!"** 라는 전략을 취합니다.
	- 동작: 컴포넌트 함수 내부의 모든 줄(Line)이 위에서 아래로 다시 실행됩니다.
	- 비효율: 데이터가 안 바뀐 함수 호출(`expensiveCall`), 데이터가 똑같은 자식 요소(`<Child />`)들도 전부 **새로운 주소값을 가진 객체로 다시 생성**됩니다.
	- 최종 단계: 리액트가 마지막에 "음, 결과물(Virtual DOM)을 비교해 보니 실제 화면(Real DOM)은 이 부분만 바꾸면 되겠네" 하고 화면을 고칩니다. (즉, 화면 수정은 최소화하지만, **함수 실행과 객체 생성은 전체**가 다 일어납니다.)

2. 리액트 컴파일러 도입 후 리렌더링
	컴파일러는 코드를 읽고 **"어디가 어디에 연결되어 있는지"** 지도를 미리 그려둡니다.
	- 동작: 컴포넌트 함수가 실행되기는 하지만, 각 코드 블록 앞에 **검문소**가 생깁니다.
	- 효율:
	    1. "이 변수는 `a` 데이터로 만든 건데, `a`가 안 바뀌었네? 그럼 예전 값 그대로 써!" (계산 생략)
	    2. "이 JSX 조각은 `b` 데이터로 만든 건데, `b`가 그대로네? 그럼 예전 객체 그대로 내보내!" (객체 생성 생략)
	- 결과: 실제로 CPU가 연산을 수행하고 메모리를 새로 할당하는 작업은 **바뀐 데이터와 직접 연결된 코드**에서만 일어납니다.


정리하면 
리액트는 useState, useReducer, useContext의 데이터가 수정된 경우에만 리렌더링을 실행하는데 자식 컴포넌트 전체를 리렌더링함. 
useMemo, useCallback, React.memo를 사용하면 특정 데이터가 그려질때만 리렌더링을 하도록 지정할 수 있음. 

주의점
- 리액트 컴파일러가 최적화를 하는 과정에서 주소값을 유지할 수도 새로 만들 수도 있음. 
- 따라서 코드를 **객체의 주소값(`===`)에 의존하지 말고 실제 값(`user.id` 등)에 의존**하도록 해야함. 


### 컴포넌트를 순수하게 유지

1. **자신의 일에만 집중합니다.** 함수가 호출되기 전에 존재했던 어떤 객체나 변수도 변경하지 않습니다.
2. **같은 입력, 같은 출력.** 같은 입력이 주어졌다면 순수 함수는 항상 같은 결과를 반환합니다.

- React는 개발 중에 각 컴포넌트의 함수를 두 번 호출하는 “엄격 모드”를 제공. **컴포넌트 함수를 두 번 호출함으로써, 엄격 모드는 이러한 규칙을 위반하는 컴포넌트를 찾는데 도움을 줌**
- **이벤트 핸들러, `useEffect`** 순수할 필요가 없음.
- 데이터를 변경하는 변화들은 전부 이벤트 핸들러, useEffect에서 발생해야함.  

### Effect

##### 기본 동작

- 렌더링 후 특정 코드를 실행하여 리액트 외부의 시스템과 컴포넌트를 동기화할 수 있음. 
- 화면 업데이트 이후에 실행됨. 
- 이벤트 핸들러 vs Effect
	- 이벤트 핸들러: 상호작용 O, 예: 채팅에서 메세지를 전송하는 것 
	- Effect: 상호작용 X, 예: 서버 연결 설정

- `useEffect(() => {...}, [])`에서 배열에 의존성을 지정해서 해당 의존성이 변하는 경우에 Effect가 실행되도록 할 수 있음. 
- `useEffect` 내부에 사용된 prop, state을 포함한 **모든 컴포넌트 내의 변수**들은 필수로 의존성 배열에 넣어야함. 
``` jsx
useEffect(() => {
  // 1. 모든 렌더링 후에 실행 (마운트, 리렌더링)
});

useEffect(() => {
  // 2. 마운트될 때만 실행 (컴포넌트가 나타날 때), 리렌더링 시에는 실행 X
}, []);

useEffect(() => {
 // 3. 마운트될 때 실행, 리렌더링 시 a 또는 b 중 하나라도 변경된 경우에도 실행
}, [a, b]);
```

- Effect가 **다시 실행될때** 이전 Effect의 클린업 함수가 실행된 후, 새로운 Effect가 실행됨. 
- 컴포넌트가 **마운트 해제(제거)될 때**에도 마지막에 클린업 함수를 호출함. 
- 클린업 함수가 잘 설정이 되었는지를 확인하기 위해 React는 기본적으로 개발 모드에서 초기 마운트 후 모든 컴포넌틀 한 번 다시 마운트함. 
``` jsx
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
	  // 클린업 함수
      connection.disconnect();
    };
  }, []);
```


- props나 state의 변경이 다른 state를 조정하는 구조는 권장하지 않음. 
``` jsx
function List({ items }) {
  const [selection, setSelection] = useState(null);

  // 🔴 피하세요: Effect에서 prop 변경 시 state 조정하기
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```
`items`가 변경될 때마다 `List`와 그 자식 컴포넌트들은 처음에는 오래된 `selection` 값으로 렌더링됨. 그런 다음 React는 DOM을 업데이트하고 Effect를 실행. 
Effect 내부에서의 `setSelection(null)` 호출은 `List`와 그 자식 컴포넌트들을 다시 렌더링하여 이 전체 프로세스를 다시 시작하게 됨.

- 각 Effect는 별도의 동기화 프로세스를 나타내야 함. (하나의 역할만 담당)

##### useEffectEvent

- Effect 이벤트 선언하기: `useEffectEvent`를 사용하여 Effect에서 비반응형 로직을 추출할 수 있음. (의존성 제외하기)
- `useEffectEvent`는 Effect 내부에서 사용 가능한 이벤트 핸들러와 비슷함. 
- **내부의 로직은 반응형이 아니며 항상 props, state의 최근 값을 바라봄.**

- Effect 내부에서만 호출
- 절대로 다른 컴포넌트나 Hook에 전달 불가

``` jsx
function ChatRoom({ roomId, theme }) {
  // theme을 반응성에서 제외할 수 있음. 
  const onConnected = useEffectEvent(() => {
    showNotification('연결됨!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 모든 의존성이 선언됨 (theme 의존성을 제외시킬 수 있음)
  // ...
```
- 컴포넌트 외부에 선언된 값은 반응형 값이 아니기 때문에 `theme` 의존성을 제거할 수 있음. 

##### 의존성 제거

- 리액트에서 리렌더링 할때마다 생성되는 **객체와 함수는 내용이 동일하더라도 다른 객체와 함수로 생성됨.** (자바스크립트에서 객체와 함수는 서로 다른 시간에 생성된 경우 서로 다른 것으로 간주됨.)
- 그래서 의도치 않게 의존성이 변해서 Effect가 호출될 수 있음. 그러므로 가능하면 객체와 함수를 Effect의 의존성으로 사용하는 것을 피해야함.

1. 컴포넌트 외부에 선언하는 방법: 외부에 선언하면 반응형이 아님. 
2. Effect 내부에 선언하는 방법
``` jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    // `options`는 Effect 내부에서 선언되었으므로 더 이상 Effect의 의존성이 아닙니다.
    const options = { 
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 모든 의존성 선언됨
  // ...
```

3. 객체를 변수로 받아서 Effect 내부에서 사용하는 방법
``` jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // 의존성에서 객체, 함수를 선언하지 않을 수 있음. 
  // ...
```


