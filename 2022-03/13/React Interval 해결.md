# 2022/03/13

React 내에서 ‘현재 시간’, ‘오늘의 남은 시간’ 을 화면에 rendering 하였다. ‘현재 시간’ 을 render해주는 데에는 문제가 없었지만, ‘오늘의 남은 시간’ 즉, 23:60:60 - ‘현재 시간’ 을 계산하려 할 때 , 문제가 발생했다. ‘오늘의 남은 시간’이 잘 표시 되었지만, 몇 초 지나지 않아 ‘현재 시간’으로 돌아가는 문제가 발생했다. setInterval으로 잘 불러오고 있다고 생각했는데 , 왜 이렇게 동작할까?

## 원인 분석

### 원인 1. setInterval의 시간 보장

보통 subscription API들은 리렌더링이 되면, 이전의 subscription을 해제하고 새로운 subscription을 만드는 데, setInterval은 그렇지 못하다. 이전의 Interval을 해제하기 위해선 직접 clearInterval을 사용해 timer을 해제해야 한다. 

하지만! 이렇게 작성하여도 setInterval은 완벽히 동작하지 않는다. 

![aaa](https://user-images.githubusercontent.com/78465062/158026777-b518b4a4-2ec8-409b-b979-4a3a698fca86.png)


그 이유는, setInterval은 우리가 원하는 delay시간을 100% 보장 못하기 때문이다. setInterval은 함수를 실행하는 시간조차 delay에 포함시키기 때문에, 만약 함수를 실행하는 시간이 (currentTime, 1000 즉 1초) 보다 길다면, 이 실행시키는 1초를 기다리지 않고 다음 함수를 바로 실행해버린다는 것이다.

---

### 원인 2. 예기치 못한 Closure

- Closure 란?  직역하면 갇혀있다 라는 뜻으로, 어떤 내부 함수를 감싸는 외부함수가 실행된 뒤 종료되었다 하더라도, 내부 함수에서 외부함수의 값에 접근할 수 있는 현상을 말한다.
- 즉, setInterval(외부 함수)는 이미 종료되었는데, currentTimer(내부 함수)가 계속 그 값을 기억하고 있는 것이다.

JS는 Single thread기반의 언어이다. 기존 자바스크립트 엔진은 Stack 에 값을 하나 씩 저장하여, 한 번에 하나의 연산 만을 실행 시킬 수 있다.  때문에 실행 시간이 오래 걸리는 함수를 호출한다면 화면이 멈출 수도 있다. 따라서 브라우저에서는 Web API를 제공하여 이를 해결한다. Call Stack에서 비동기 함수가 호출되면 Web API를 통해 Callback Queue에 쌓이게 되고, 이 Queue는 Call Stack이 비면 실행된다. 

setInterval 메소드가 이러한 Web API의 종류 중 하나 이며, 호출되면 바로 실행되지 않고 우리가 등록한 delay 시간을 기다린 뒤 Callback Queue에 쌓여 Call Stack이 비면 그 때 실행되는 구조로 알 수 있다. 다시 말해, setInterval이 주기적으로 실행하라고 남겨놓은 currentTime 함수가 주기적으로 실행되는 것이다!!

## 해결

**Reducer 사용**

- 아래와 같은 Reducer()의 원리를 통해, 이전의 값을 기억해줌으로써 이를 해결할 수 있다. 하지만, 이 방법은 react의 lifecycle과 다소 벗어난 행동이라는 점에서 좋은 해결책이라 볼 수 없다. react에서는 state가 바뀌면, 리렌더링을 하는데 이 방법은 렌더링과 상관없이 setInterval이 계속 살아남아있다는 것이다. 쉽게 말해, 이전의 render된 내용들은 다 잊고, 새로 그려지는 반면 setInterval은 그렇지 않다.

```jsx
import React, { useEffect, useReducer } from "react";
import ReactDOM from "react-dom";

function Counter() {
  const [count, dispatch] = useReducer((state, action) => {
    if (action === 'inc') {
      return state + 1;
    }
  }, 0);

  useEffect(() => {
    let id = setInterval(() => {
      dispatch('inc');
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <h1>{count}</h1>;
}

const rootElement = document.getElementById("root");
ReactDOM.render(<Counter />, rootElement);
```

### setState에 callback() 전달 : useInterval 사용

때문에 위와 같은 문제점을 해결하기 위한 자료를 찾던 중에, 또 이에 맞는 custom Hook이 해외에서 이미 코드화 되어 있다는 점을 알 수 있었다. 해당 코드의 원리를 기반으로 하여, 차근차근 이해해나가며 변형시킴으로써 해당 문제를 해결하였다.

```jsx
import { useState, useEffect, useRef } from 'react';

function useInterval(callback, delay) {
  const savedCallback = useRef(); // Ref를 통해, 가장 최근의 callback 저장.

  useEffect(() => {
    savedCallback.current = callback; // Ref 값을 계속 업데이트.
  }, [callback]);  //callback이 바뀔때마다 새로 실행되도록,

  useEffect(() => {
    function tick() {
      savedCallback.current(); // tick 함수의 실행으로, callback함수를 실행.
    }
    if (delay !== null) { // 만약 delay가 null이 아니라면 
      let id = setInterval(tick, delay); // delay에 맞추어 interval을 새로 실행.
      return () => clearInterval(id); // unmount될 때 clearInterval
    }
  }, [delay]); // delay가 바뀔 때마다 새로 실행되도록,
}
```

## 해결 코드

Main.tsx

```jsx
function Main() {
	const [restTime, setRestTime] = useState("0");
	
	const restTimer = () => {
    const date = new Date();
    const hours = String(23-date.getHours()).padStart(2, "0");
    const minutes = String(60-date.getMinutes()).padStart(2, "0");
    const seconds = String(60-date.getSeconds()).padStart(2, "0");

    setTimer(`${hours}:${minutes}:${seconds}`);
  };

    setInterval(currentTimer, 1000);
    useInterval(() => {
       restTimer();
    }, 1000);
    ...
}	
```



```jsx

function useInterval(callback: Function, delay?: number | null) {
  const savedCallback = useRef<Function>();

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    function tick() {
      if (savedCallback && savedCallback.current) {
        savedCallback.current();
      }
    }
    if (delay !== null) {
      let id = setInterval(tick, delay);
      return () => clearInterval(id);
    }
  }, [delay]);
}
```

참고 사이트: [https://overreacted.io/making-setinterval-declarative-with-react-hooks/](https://overreacted.io/making-setinterval-declarative-with-react-hooks/)

[https://ko.javascript.info/settimeout-setinterval](https://ko.javascript.info/settimeout-setinterval)
