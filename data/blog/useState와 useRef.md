---
title: useState와 useRef
date: '2023-07-29'
lastmod: ''
tags: ['react', 'hooks']
draft: false
summary: 'useState와 useRef의 차이점에 대해 알아보았습니다.'
---

최근 면접에서 useState와 useRef의 차이점에 대한 질문이 들어왔는데 애매하게 답변을 해서 이번에 이 둘의 차이점에 대해 조사해보고자 한다.

```jsx
// useRef
import { useRef, useState } from 'react'

export default function Counter() {
  const [state, setState] = useState(0)
  const ref = useRef(0)

  function handleStateClick() {
    setState((prev) => prev + 1)
    console.log('You clicked ' + state + ' times!')
  }

  function handleRefClick() {
    ref.current = ref.current + 1
    console.log('You clicked ' + ref.current + ' times!')
  }

  return (
    <>
      <p>State : {state}</p>
      <p>Ref : {ref.current}</p>
      <button onClick={handleStateClick}>Click me!(state)</button>
      <button onClick={handleRefClick}>Click me!(ref)</button>
    </>
  )
}
```

useState의 경우에는 버튼 클릭 때마다 콘솔에 매번 출력이 되고 화면에 렌더링이 된다.

또한 `Click me!(ref)`을 여러번 클릭하고 `Click me!(state)` 버튼을 클릭했을 때 `Click me!(ref)`를 클릭한 횟수만큼 한번에 증가하는 현상이 있는데, 이 부분은 state가 업데이트가 되면서 다시 컴포넌트를 렌더링하기 때문에 발생하는 것이다.

반면, useRef의 경우에는 버튼을 클릭해도 화면에 렌더링이 안된다. 왜 그럴까..?

공식 문서에 나와있는 바로는,

```
If you show {ref.current} in the JSX, the number won’t update on click. This is because setting ref.current does not trigger a re-render.
```

**`ref.current`를 설정해도 리렌더링을 트리거하지 않기 때문에 JSX 코드 내에 변경사항이 반영되지 않는다.** 라고 한다.

참고로 useRef의 내부 구현은 이런식으로 되어있다.

```jsx
const useRef = (value) => {
  const [ref] = useState({ current: value })
  return ref
}
```

공식문서에도 useState와 useRef를 잘 비교해놓은 것 같아서 정리해보려고 한다.

`useState`

- state가 변경될 때마다 컴포넌트가 리렌더링 된다.
- state는 직접 변경이 불가능하며 setState로 변경해야 한다.

`useRef`

- useState와 달리 리렌더링을 트리거하지 않는다.
- current 내에 있는 값을 직접 변경이 가능하다.
- 다만, 렌더링 중에는 current 값을 읽거나 쓰기를 하면 안된다.

두 hook의 차이의 핵심은 결국 리렌더링이 되냐 안되냐의 차이다.

### Reference

https://velog.io/@jinyoung985/React-useRef%EB%9E%80

https://www.daleseo.com/react-hooks-use-ref/

https://react.dev/learn/referencing-values-with-refs#differences-between-refs-and-state
