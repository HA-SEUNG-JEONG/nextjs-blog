---
title: Recoil에서 jotai로
date: '2023-11-21'
lastmod: '2023-11-22'
tags: ['react', 'jotai']
draft: false
summary: 'recoil에서 jotai로 마이그레이션해보았습니다..'
images: '../../public/static/images/avatar.png'
---

### Recoil을 쓰면서 겪었던 이슈

회사 프로젝트를 하면서 내가 실제로 사용하는 컴포넌트는 몇 개 되지 않았는데, props driling을 거치는 단계도 2~3단계 정도였지만 솔직히 일일이 props를 넘겨주는 작업도 생각보다 귀찮았다.

그래서 상태관리 라이브러리를 사용하기로 했고, 이때까지 취준하면서 공부하고 또 가장 많이 사용했던 것이 `Recoil`이라 한번 적용해보기로 했다.

하지만 적용하면서 1가지 이슈가 있었다..

> Duplicate atom key "Atom key". This is a FATAL ERROR in
> production. But it is safe to ignore this warning if it occurred because of hot module replacement.

새로 atom을 생성하거나 사용을 안하면서 필요없어질 때 삭제하면서 발생했었던 이슈다.

공식문서를 통해 .env 파일에 `RecoilEnv.RECOIL_DUPLICATE_ATOM_KEY_CHECKING_ENABLED = false`을 집어넣어서 키 체크를 불가능하게 만드는 방법을 시도해봤으나 여전히 오류는 사라지지 않았다.

.env 파일 설정이 잘못됐나 싶어 조금 멍청한(?) 방법으로 recoil을 사용하는 파일에 일일이 집어넣어봤으나..똑같았다.

### 그럼 뭘 써야하나?

상태관리 라이브러리는 참 많다. Redux, Zustand, Jotai 등등...

![render](/static/images/jotai2.png)

npm trend에서 가장 이름이 날린(?) 라이브러리를 비교해봤다.

나는 선택 기준을 **Star 수, 최근 업데이트**로 잡았는데, 그 중에서 jotai와 zustand를 고민했고 jotai가 zustand보다 Star 수가 적기는 했지만, Recoil과 비슷한 컨셉이고 또한 zustand에 비해 문법(?)이 간단하다고 판단해서 고르게 되었다.

![render](/static/images/jotai.png)

공식 문서에도 recoil과 zustand,jotai를 각각 비교하면서 zustand와 jotai를 써야 하는 상황에 대한 글을 보게 되었는데

> If you need a replacement for useState+useContext, Jotai fits well.

라는 문장이 눈에 띄었다. 왜냐하면 코드에 useState가 너무 많았기 때문에..

그래서 주저없이 jotai를 쓰기로 했다.

참고로 zustand와 jotai의 문법 비교를 해보자면,

#### zustand

```jsx
import { create } from 'zustand'

const useStore = create((set) => ({
  count: 1,
  inc: () => set((state) => ({ count: state.count + 1 })),
}))

function Counter() {
  const { count, inc } = useStore()
  return (
    <div>
      <span>{count}</span>
      <button onClick={inc}>one up</button>
    </div>
  )
}
```

#### jotai

```jsx
mport { atom, useAtom } from 'jotai'

// Create your atoms and derivatives
const textAtom = atom('hello')
const uppercaseAtom = atom(
  (get) => get(textAtom).toUpperCase()
)

// Use them anywhere in your app
const Input = () => {
  const [text, setText] = useAtom(textAtom)
  const handleChange = (e) => setText(e.target.value)
  return (
    <input value={text} onChange={handleChange} />
  )
}
```

### 사용해보기

jotai도 recoil과 비슷하게 Atom 개념을 갖고 있다.

```js
export const stateAtom = atom({
  isLoading: false,
  modalOpen: false,
  //....
})
```

```js
const [isLoading, setIsLoading] = useState(false)
const [modalOpen, setModalOpen] = useState(false)
//.... 그 밖의 더 많은 useState
```

이 많은 state 관리를 다음과 같이 바꿔줄 수 있다.

```js
const [{ isLoading, modalOpen ,...}, setState] = useAtom(stateAtom);

// setter 함수
 const setLoading = (loading) => {
      setState((prev) => ({ ...prev, isLoading }));
    };

const setModalOpen = (open) => {
      setState((prev) => ({ ...prev, modalOpen: open }));
    };
```

하지만 이런식으로 setter 함수를 반복해서 작성해야 하는데.. 솔직히 귀찮다.

그래서 따로 setter 함수를 관리하는 파일을 만들어서 사용하기로 했다.

```js
// setter.js
export const setAtomValue = (setter) => (value) => {
  setter((prev) => ({ ...prev, ...value }))
}

// 다른 파일에 사용할 때는 이런 식으로 작성
const writeAtom = useSetAtom(atom)
const setValue = setAtomValue(writeAtom)
```

다른 파일에서 setter 함수가 필요할 경우, `useSetAtom`이라는 훅과 따로 만든 `setAtomValue`를 이용해서 값을 업데이트 하면 된다.

### 사용하면서 좋았던 점

우선, useReducer와 비슷하게 useState를 반복적으로 작성하지 않아도 되는 점이 좋았다.

코드 길이로만 놓고 보면 더 길어졌지만 상태관리 라이브러리 특성 상 따로 관리하는 저장소만 있으면 그닥 신경쓸 부분이 아니다.

그리고 Recoil을 쓸 때는 key 값을 따로 관리해야 한다.

```jsx
function atom<T>({
  key: string,
  default: T | Promise<T> | RecoilValue<T>,

  effects_UNSTABLE?: $ReadOnlyArray<AtomEffect<T>>,

  dangerouslyAllowMutability?: boolean,
}): RecoilState<T>

```

반면에 jotai를 사용할 때는 atom에 들어가는 객체의 default 값만 넣으면 돼서 개인적으로는 편리했다.

또한, Recoil에서 `useRecoilValue` 같은 훅을 사용할려면 메인에서 `RecoilRoot`를 달아줘야 하지만 jotai에서는 Provider를 굳이 쓸 필요가 없다. 물론 쓸 수는 있지만 Provider를 안 쓸 경우 기본 설정으로 쓸 수 있다고 한다.

### 마무리

세팅 자체는 오래 안 걸렸지만 기존에 Recoil로 떡칠해 놓은 게 있어 조금 뜯어고치느라 시간을 좀 허비했다.

다음 프로젝트에서 상태관리 라이브러리를 사용할 일이 있으면 우선 Recoil은 배제하고 시작할 거 같다.

그리고 앞서 간편하다는 이유로 jotai를 고르긴 했지만 Redux 같은 Flux 패턴도 익혀야 하지 않을까 싶다..

여전히 다른 라이브러리들에 비해 다운로드수나 Star수가 너무나 압도적이라..

### References

https://jotai.org/docs/basics/comparison

https://jotai.org/

https://zustand-demo.pmnd.rs/
