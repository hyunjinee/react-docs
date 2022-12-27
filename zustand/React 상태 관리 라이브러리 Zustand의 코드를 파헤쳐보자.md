# React 상태 관리 라이브러리 Zustand의 코드를 파헤쳐보자

https://ui.toast.com/weekly-pick/ko_20210812

## Zustand는 무엇인가?

Zustand는 독일어로 '상태'라는 뜻을 가진 라이브러리이며 Jotai를 만든 카토 다이시가 제작에 참여하고 적극적으로 관리하는 라이브러리이다. 아래의 특징을 가지고 있다.

- 특정 라이브러리에 엮이지 않는다.(그래도 React와 함께 쓸 수 있는 API는 기본적으로 제공한다.)
- 한개의 중앙에 집중된 형식의 스토어 구조를 활용하면서, 상태를 정의하고 사용하는 방법이 단순한다.
- Context API를 사용할 때와 달리 상태 변경 시 불필요한 리렌더링을 일으키지 않도록 제어하기 쉽다.
- React에 직접적으로 의존하지 않기 때문에 자주 바뀌는 상태를 직접 제어할 수 있는 방법도 제공한다.(Transient Update라고 한다.)
- 동작을 이해하기 위해 알아야하는 코드 양이 아주 적다. 핵심 로직의 코드 줄 수가 약 42줄밖에 되지 않는다.(VanillaJS 기준)

### 왜 Context API를 사용하지 않는가?

**Context는 컴포넌트에 의존성을 주입할 수 있는 아주 효과적인 방법 중 하나이다.** 하지만 부모 컴포넌트 쪽에 Context.Provider 컴포넌트를 선언하고 Context로 전달되는 값이 변경될 때 해당 Context를 사용하는 모든 자손 컴포넌트는 리렌더링된다.

다음과 같이 임의의 값과 값을 변경하는 방법을 제공하는 Context와 Provider를 만들었다고 가정해보자.

```tsx
const SomeObjectContext = React.createContext({ input: "", count: 0 })
const SetSomeObjectConteext = React.createContext()

const Provider = ({ children }) => {
  const [someObj, setSomeObj] = React.useState({ input: "", count: 0 })

  return (
    <SetSomeObjectConteext.Provider value={setSomeObj}>
      <SomeObjectContext.Provider value={someObj}>
        {children}
      </SomeObjectContext.Provider>
    </SetSomeObjectConteext.Provider>
  )
}
```

이 Context를 소비하는 컴포넌트가 여러 단계 아래쪽 자손일 수 있다.

```tsx
const InputConsumer = () => {
  const { input } = useContext(SomeObjectContext)
  // ...
}

const CountConsumer = () => {
  const { count } = useContext(SomeObjectContext)
  // ...
}

const App = () => (
  <Provider>
    <DeepChildren>
      <SomeOtherChildren>
        <AnotherChildren>
          {/* input 값만 사용하려 함 */}
          <InputConsumer />
        </AnotherChildren>
        {/* count 값만 사용하려 함 */}
        <CountConsumer />
      </SomeOtherChildren>
    </DeepChildren>
    <IDontCareContextChild />
    {/* SomeObject 값을 비꾸는 역할을 함 */}
    <ContextSetter />
  </Provider>
)
```

이렇게 선언된 컴포넌트 트리의 경우 ContextSetter 컴포넌트에서 Context 값의 일부만 바꾸는 동작을 실행하더라도, InputConsumer, CounterSonsumer 모두 리렌더링이 일어나게된다. 결국 객체 형태로 Context를 관리하면서 Context를 소비하는 컴포넌트가 많아질 경우 불필요한 리렌더링이 많이 일어나 애플리케이션의 성능에 문제가 생길 수 있다.

이 문제를 해결하기 위한 방안은 여러 가지가 있다.

1. 하나의 거대한 값을 가진 Context를 만들지 말고 여럿으로 분리하여 필요한 부분만 사용하기
2. 컴포넌트를 쪼개고 React.memo를 활용하기
3. useMemo 훅을 사용하여 컴포넌트 렌더링 부분을 감싸기

제일 권장되는 방법은 1번이지만 여러 Context를 만들어 Provider로 주입할 때 [Provider Hell](https://dev.to/alfredosalzillo/the-react-context-hell-7p4)이라고 불리는 중첩 Provider로 인한 가독성 문제가 생긴다.

### 간략한 Zustand 사용 방법

스토어를 만들 때는 `create` 함수를 이용하여 상태와 그 상태를 변경하는 액션을 정의한다. 그러면 리액트 컴포넌트에서 사용할 수 있는 `useStore` 훅을 리턴한다.

```tsx
import create from "zustand"

// set 함수를 통해서만 상태를 변경할 수 있다
const useStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}))
```

컴포넌트에서 useStore 훅을 사용할 때는 스토어에서 상태를 어떤 형태로 꺼내올지 결정하는 셀렉터 함수를 전달해 주어야 한다. 만약 셀렉터 함수를 전달하지 않는다면 스토어 전체가 리턴된다.

```tsx
// 상태를 꺼낸다
function BearCounter() {
  const bears = useStore((state) => state.bears)
  return <h1>{bears} around here ...</h1>
}

// 상태를 변경하는 액션을 꺼낸다
function Controls() {
  const increasePopulation = useStore((state) => state.increasePopulation)
  return <button onClick={increasePopulation}>one up</button>
}
```

## Zustand 파헤쳐보기 - 코어

원본 코드는 Typescript로 작성되어 있으나, 최대한 간략하게 설명하기 위하여 자바스크립트로 작성된 코드를 보며 동작 원리를 살펴보자

Zustand는 [발행/구독 모델](https://www.rinae.dev/posts/why-every-beginner-front-end-developer-should-know-publish-subscribe-pattern-kr)을 기반으로 이루어져 있다. 스토어의 상태 변경이 일어날 때 실행할 리스너 함수를 모아두었다가(구독) 상태가 변경되었을 때 등록된 리스너들에게 상태가 변경되었다고 알려준다(발행).

또한 스토어를 생성하는 함수를 호출할 때 클로저를 활용한다. 클로저는 간단히 말해서 함수가 선언될 시 그 주변 환경을 기억하는 것으로, 아래의 코드를 보면 스토어의 상태는 스토어를 조회하거나 변경하는 함수 바깥 스코프에 항상 유지되도록 만들어진 것을 볼 수 있다. 그러면 상태의 변경, 조회, 구독 등의 인터페이스를 통해서만 스토어를 다루고 실제 상태는 애플리케이션의 생명 주기 처음부터 끝까지 의도치 않게 변경되는 것을 막을 수 있다.

```tsx
// createState는 예제에서 보았던 생성자 함수이다.
// 함수 최하단부에 언급되지만 set, get 함수와
// `create` 함수를 통해 만들어지는 내부 API를 인자로 전달받을 수 있다.
export function create(createState) {
  // 스토어의 상태는 클로저로 관리된다.
  let state

  // 상태 변경을 구독할 리스너를 Set으로 관리한다.
  // 배열로 관리할 경우 중복된 리스너를 솎아내기 어렵기 때문이다.
  const listeners = new Set()

  const setState = (partial, replace) => {
    const nextState = typeof partial === "function" ? partial(state) : partial
    if (nextState !== state) {
      const previousState = state

      state = replace ? nextState : Object.assign({}, state, nextState)

      listeners.forEach((listener) => listener(state, previousState))
    }
  }

  const getState = () => state

  const subscribeWithSelector = (
    listener,
    selector = getState,
    equalityFn = Object.is
  ) => {
    let currentSlice = selector(state)

    function listenerToAdd() {
      const nextSlice = selector(state)
      if (!equalityFn(currentSlice, nextSlice)) {
        const previousSlice = currentSlice
        listener((currentSlice = nextSlice), previousSlice)
      }
    }

    listeners.add(listenerToAdd)
    return () => listeners.delete(listenerToAdd)
  }

  const subscribe = (listener, selector, equalityFn) => {
    if (selector || equalityFn) {
      return subscribeWithSelector(listener, selector, equalityFn)
    }
    listeners.add(listener)
    return () => listeners.delete(listener)
  }

  // 모든 리스너를 제거한다. 하지만 이미 정의된 상태를 초기화하진 않는다.
  const destroy = () => listeners.clear()

  const api = { setState, getState, subscribe, destroy }

  // 인자로 전달받은 createState 함수를 이용하여 최초 상태를 설정한다.
  state = createState(setState, getState, api)

  return api
}
```

### 상태의 변경

먼저 상태를 변경하는 setState 함수를 살펴보자. 이 함수는 현재 상태를 기반으로 새로운 상태를 리턴하는 함수 혹은, 아예 변경하려는 상태 값을 전달받는다.

```ts
store.setState((state) => ({ counter: state.counter + 1 }))
// 혹은
store.setState({ counter: 10 })
```

이런 동작을 구현하려면 기존 상태를 함수의 인자로 전달하는 방법, 그리고 함수를 갱신하는 방법이 필요하다. 먼저 함수를 전달받을 경우 현재 상태를 인자로 넘겨주는 식으로 '다음 상태' 를 정의한다. 이런 동작을 구현하려면 기존 상태를 함수의 인자로 전달하는 방법, 그리고 함수를 갱신하는 방법이 필요하다. 먼저 함수를 전달바등 ㄹ경우 현재 상태를 인자로 넘겨주는 식으로 '다음 상태'를 정의한다.

```ts
const setState = (partial) => {
  const nextState = typeof partial === "function" ? partial(state) : partial
  // ...
}
```

그리고 nextState가 기존 state와 다른 경우 state를 갱신한다. 상태를 갱신할 때 간단하고 효과적인 Object.assign을 사용한다.

```ts
if (nextState !== state) {
  const previousState = state

  state = Object.assign({}, state, nextState)

  listeners.forEach((listener) => listener(state, previousState))
}
```

Object.assign 은 얕은 복사를 수행하기 때문에 깊이 중첩된 형태의 스토어를 만들었을 때는 setState 를 호출할 때 유의해야 한다. setState 함수는 직접 호출할 일이 거의 없고 생성자 함수로 스토어를 정의할 때 첫 번째 인자로 전달받기 때문에 생성자 함수에서 setState 를 사용하여 상태를 변경하는 액션을 정의하면 된다.

```ts
// 여기 있는 `set` 이 위에서 정의한 `setState` 함수이다.
const create = create(set => ({
  // ...
  // 이렇게 상태를 변경하는 액션을 정의한다.
  someAction: () => set(state => ({ /* ... */ }))
});
```

### 상태의 구독

상태를 구독하는 함수를 등록할 때는 subscribe 함수를 사용한다. 이 함수를 사용하여 모든 상태의 변화를 구독할 수도 있고, 상태의 일부만 구독할 수도 있다.

```ts
const subscribe = (listener, selector, equalityFn) => {
  if (selector || equalityFn) {
    return subscribeWithSelector(listener, selector, equalityFn)
  }
  listeners.add(listener)
  // 구독을 해제하는 함수도 리턴해준다.
  return () => listeners.delete(listener)
}
```

만약 listener만 전달하지 않고 두 번째 인자로 함수의 셀렉터까지 전달한 경우라면 selector로 꺼낸 상태의 일부(슬라이스)를 어딘가에 보관하고, 상태가 바뀔 때마다 이전 슬라이스와 비교하는 과정이 필요하다.

```ts
const subscribeWithSelector = (listener, selector, equalityFn = Object.is) => {
  // 구독 시 이 변수의 클로저가 생성된다.
  let currentSlice = selector(state)

  // 상태가 변경될 때마다 실행해 줄 함수
  function listenerToAdd() {
    const nextSlice = selector(state)
    if (!equalityFn(currentSlice, nextSlice)) {
      const previousSlice = currentSlice
      listener((currentSlice = nextSlice), previousSlice)
    }
  }

  listeners.add(listenerToAdd)
  return () => listeners.delete(listenerToAdd)
}
```

nextSlice 변수를 할당하는 시점에 state 변수는 어딘가에서 호출된 setState 함수 덕분에 새로운 상태로 갱신되었을 것이다. setState 가 호출 될 때 리스너 목록에 등록되어있던 listenerToAdd 함수가 실행되었기 때문이다. 그래서 이전 슬라이스와 새 슬라이스가 다른 값일 경우 지정된 리스너를 호출해주면서 currentSlice 변수를 갱신한다.
