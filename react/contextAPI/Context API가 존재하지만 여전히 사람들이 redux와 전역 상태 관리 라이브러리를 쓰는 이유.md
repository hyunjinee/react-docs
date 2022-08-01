# Context API가 존재하지만 여전히 사람들이 redux와 전역 상태관리 라이브러리를 쓰는 이유

![image](https://user-images.githubusercontent.com/63354527/182167202-a39895d2-25f1-4fcb-9ab1-731a86c5006e.png)

일반적으로 리액트 데이터는 부모로부터 자식으로 props를 통해 탑-다운으로 전달되는데, 이단계가 너무 많아진다거나 전달을 여러곳에 해줘야하는 경우 전역 스토어에 데이터를 저장하고 이를 데이터가 필요한 컴포넌트에 따로 공유할 수 있다.

어떤 데이터를 전역/로컬에 저장할 것인지는 개발자가 선택해야한다. 몇단계만 전달하면 되거나 굳이 전역으로 관리할 필요 없는 데이터를 전역 스토어에 넣는 것은 코드의 관리 측면에서도 좋지 않다.

## Context API

리액트 16.3부터 도입된 context api를 사용하면 네이티브 리액트만으로 전역 상태관리가 가능하다. 거의 모든 상태관리 라이브러리들은 이 api를 기반으로 개발되어있다.

```js
type State = boolean
type Dispatch = React.Dispatch<React.SetStateAction<State>>
const ModalContext = React.createContext < [State, Dispatch] > [false, () => {}]

const ModalProvider = ({ children }) => {
  const [show, setShow] = useState(false)

  return (
    <ModalContext.Provider value={[show, setShow]}>
      {children}
    </ModalContext.Provider>
  )
}
```

React.createContext 메서드를 통해서 컨텍스트를 만들면 Provider와 Consumer가 제공된다. 위의 코드는 useState가 반환하는 값을 Provider에 제공한 형태다.

```js
const App = () => {
  return (
    <ModalProvider>
      <Modal />
    </ModalProvider>
}
```

위와 같이 위에서 만든 Provider로 감싸주면, <Modal />은 Provider에 저장된 값을 사용할 수 있다.

## 리액트와 하위 컴포넌트 리렌더링

이런 식으로 개발하는 이유는 리액트의 렌더링 특성 때문이다. 기본적으로 리액트는 상위 컴포넌트에서 state가 변하면 하위 컴포넌트를 전부 렌더링한다.

```js
const App = () => {
  const [show, setShow] = useState(false)

  return (
    <div>
      <button onClick={() => setShow(!show)}>{show}</button>
      <Child />
    </div>
  )
}

const Child = () => <div>hello react!</div>
```

위와 같은 코드가 있다고 하자. 버튼을 누르면 show라는 App 내의 state 값이 변경된다. <Child />는 show라는 props를 쓰지 않지만, 하위 컴포넌트이기때문에 렌더링이 된다. 이를 방지하기 위해서 아래와 같은 방법을 사용할 수도 있다.

```js
const Child = React.memo(() => <div>hello react!</div>)
```

**React.memo의 두번째 인자로 아무것도 반환하지 않는다면 자체적으로 props를 비교해서 렌더링 여부를 결정하고, 렌더링 여부 기준을 직접 작성할 수도 있다. 하지만 그렇다고 해서 memo를 무분별하게 남발하는건 좋지 않다. 현재 리렌더링을 memo 하려는 컴포넌트에서 비교 연산과 리렌더링 중 어느 쪽이 퍼포먼스 최적화에 더 적합한지 확인이 필요하기 때문이다.**

```js
const App = () => {
  return (
    <Wrapper>
      <Child />
    </Wrapper>
  )
}
const Child = () => <div>hello react!</div>
const Wrapper: React.FC = ({ children }) => {
  const [show, setShow] = useState(false)
  return (
    <div>
      <button onClick={() => setShow(!show)}>{show}</button>
      {children}
    </div>
  )
}
```

비교적 간단한 해결방법은 위와 같이 children을 활용하는 것이다. 위의 코드에서 children은 리렌더링이 되지 않는다.

```js
const App = () => {
  return (
    <ModalProvider>
      <ModalToggleButton />
      <Modal />
    </ModalProvider>
  )
}
const ModalToggleButton = () => {
  const [, setShow] = useContext(ModalContext)
  return <button onClick={() => setShow((state) => !state)}>모달토글</button>
}
const Modal = () => {
  const [show] = useContext(ModalContext)
  return show ? <div>나 모달</div> : null
}
```

마찬가지로 ModalProvider 내부에서 show가 변경되더라도 children인 <Modal />과 <ModalToggleButton />에는 영향을 주지 않는다. 리액트에서는 useContext라는 훅을 제공해주는데 이 훅에 컨텍스트를 주입하면 아무 컴포넌트에서 컨텍스트 프로바이더에 저장된 값을 사용할 수 있다.

## contextAPI의 문제

그러나 이코드에는 문제가 있다. <Modal/> 만 렌더링되는 것이 아니라 <ModalToggleButton/> 도 같이 렌더링되는 것이다. Context.Provider는 value로 저장된 값이 변경되면 useContext(Context)를 사용하는 컴포넌트도 같이 렌더링을 한다. 그렇기 때문에 show라는 상태를 사용하지 않는 <ModalToggleButton/> 도 같이 렌더링이된다.

이를 해결하려면 아래와 같이 코드를 작성해야한다.

```js
type State = boolean;
type Dispatch = React.Dispatch<React.SetStateAction<State>>;
const ModalStateContext = React.createContext<State>(false);
const ModalDispatchContext = React.createContext<Dispatch>(() => {});
const ModalProvider = ({ children }) => {
  const [show, setShow] = useState(false);
  return (
    <ModalStateContext.Provider value={show}>
      <ModalDispatchContext.Provider value={setShow}>
        {children}
      </ModalDispatchContext.Provider>
    </ModalStateContext>
  )
}
const App = () => {
  return (
    <ModalProvider>
      <ModalToggleButton />
      <Modal />
    </ModalProvider>
  )
}
const ModalToggleButton = () => {
  const setShow = useContext(ModalDispatchContext);
  return <button onClick={() => setShow((state) => !state)}>모달토글</button>;
}
const Modal = () => {
  const show = useContext(ModalStateContext);
  return show ? <div>나 모달</div> : null;
}
```

컨텍스트를 상태값 / 액션으로 나누어서 위에서 언급한 리렌더링 문제는 발생하지 않는다. 그런데 딱 봐도 코드가 좀 지저분하고 보일러플레이트 코드가 너무 많다. 거기다 지금은 간단한 boolean값이었지만, 만약 복잡한 state라면 어떨까? object에서 상태가 부분적으로 변경이 되더라도 컨텍스트를 사용하는 모든 컴포넌트가 리렌더링 될 것이다. 그리고 컨텍스트를 추가할 때마다 프로바이더로 매번 감싸줘야하기 때문에 Provider hell을 야기할 수 있다.

```js
// 이러한 state를 다룰때
const state = {
  foo: {
    bar: "?",
  },
  hello: "world",
}
// context -> state.hello가 변하더라도 리렌더링된다
const context = useContext(FooBarStateContext)
const value = context.foo.bar
// redux -> state.foo.bar가 변경될때만 리렌더링된다
const value = useSelector((state) => state.foo.bar)
```

> **이렇기 때문에 context api는 글로벌 상태관리 라이브러리를 대체할 수 없고, 여전히 많은 리액트 개발자들이 redux, mobx 등을 사용하고 있는 것이다.**
