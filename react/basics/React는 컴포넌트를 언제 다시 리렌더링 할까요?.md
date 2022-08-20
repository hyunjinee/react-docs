# React는 컴포넌트를 언제 다시 리렌더링 할까요?

## 렌더링이란?

React가 어떻게 렌더링하고 리렌더링 하는지 알고싶다면 라이브러리 뒤에서 무슨 일이 일어나는지 이해하는 것이 좋다.  
렌더링은 다양한 추상화 수준ㅇ에서 이해할 수 있는 용어이다. 문맥에 따라 약간 의미가 다르지만, 궁극적으로는 이미지를 생성하는 과정을 설명한다.

시작하려면 DOM이 무엇인지 이해해야한다.

> W3C는 DOM은 프로그램 및 스크립트가 동적으로 접근하여 문서의 스타일과 구조 및 콘텐츠를 업데이트할 수 있는 언어 중립적 인터페이스이다.

쉡게 말해서 DOM은 웹사이트를 열 때, 화면에 표시되는 마크업 언어인 HTML을 통해 표현한다는 것을 의미한다.  
브라우저에서는 자바스크립트 API를 통해 DOM을 수정할 수 있다. 전역적으로 사용 가능한 `document`는 HTML DOM의 해당 상태를 나타내며 수정 기능을 제공한다.

또한, DOM 프로그래밍 인터페이스에 내장되어있는 `document.write`, `Node.appendChild` 또는 `Element.setAttribute`와 같은 함수를 통해 자바스크립트로 DOM을 수정할 수 있다.

## VDOM이란?

다음으로 React의 VirtualDOM(또는 VDOM)과 그 위에 또 다른 추상화 계층이 있다.  
이것은 React 애플리케이션의 요소들로 구성된다.

![image](https://user-images.githubusercontent.com/63354527/185741916-d4e2f4a9-2b9e-4b9a-825e-9dc610bd65e6.png)

애플리케이션의 상태 변경은 VDOM에 먼저 적용된다. VDOM의 새로운 상태에 대한 UI 변경이 필요한 경우, ReactDOM 라이브러리는 업데이트해야 할 항목만 업데이트하여 효율적으로 작업할을 수행할 수 있다.

예를 들어 속성만 변경되는 경우 React는 `document.setAttribute` 호출을 통해 HTML 요소의 속성만 업데이트한다.

> 빨간색 점은 DOM 트리의 업데이트를 나타낸다. VDOM을 업데이트한다고 해서 반드시 실제 DOM의 업데이트가 트리거 되는 것은 아니다.

VDOM이 업데이트되면, React는 이전의 VDOM 스냅샷과 비교한 뒤 실제 DOM에서 변경된 내용만 업데이트한다. 아무것도 변경되지 않으면 전혀 업데이트되지 않는다. 이전 VDOM과 새 VDOM을 비교하는 이 프로세스를 diffing이라고 한다.

실제 DOM업데이트는 UI를 다시 그리기 때문에 느리다. React는 실제 DOM에서 가능한 가장 적은 범위를 업데이트하여 이를 더 효율적으로 만든다.

따라서 네이티브 DOM 업데이트와 가상 DOM 업데이트의 차이점을 인식할 필요가 있다.

React에서 렌더링에 대해 이야기할 때, 우리는 대부분 render 함수의 실행에 대해서만 이야기한다. 하지만 렌더링이 항상 UI 업데이트를 의미하지는 않는다.

다음 예시를 보자.

```js
const App = () => {
  const [message, setMessage] = React.useState("")
  return (
    <>
      <Tile message={message} />
      <Tile />
    </>
  )
}
```

함수형 컴포넌트에서 함수 전체를 실행하는 것은 클래스형 컴포넌트의 render 함수와 동일하다.  
부모 컴포넌트의 상태가 변경되었을 때, 두번째 컴포넌트는 props를 받지 않아도 2개의 Title 컴포넌트모두 리렌더링된다.

이것은 render 함수가 3번 호출되는 것처럼 해석되지만, 실제로 DOM 수정은 메시지를 표시하는 Title 컴포넌트에서 한번만 발생한다.

![image](https://user-images.githubusercontent.com/63354527/185742234-df47cb01-de57-4ed9-825d-127ee5718fd0.png)

> 빨간색 점은 리렌더링을 나타낸다. React에서 이것은 render함수를 호출하는 것을 의미하며 실제 DOM에서 이것은 UI를 다시 그리는 것을 의미한다.

좋은 소식은 React가 이미 이것을 최적화하기 때문에 UI 다시 그리기의 성능 병목 현상에 대해 크게 걱정할 필요가 없다는 것이다.

나쁜 소식은 왼쪽에 있는 모든 빨간색 점에 해당하는 컴포넌트들의 render 함수들이 실행되었다는 것이다.

이러한 render 함수의 실행에는 두가지 단점이 있다.

1. React는 UI를 업데이트해야 하는지 여부를 확인하기 위해 각 컴포넌트에서 diffing 알고리즘을 수행한다.
2. render 함수 또는 함수형 컴포넌트의 모든 코드가 재실행된다.

React는 그 차이를 매우 효율적으로 계산하기 때문에 첫번째 단점은 그다지 중요하지 않다. 위험한 것은 사용자가 작성한 코드가 모든 React 렌더링에서 반복적으로 실행된다는 점이다.

위의 예시는 작은 컴포넌트의 트리였지만 각 노드에 더 많은 자식들이 있고 이 안에도 하위 컴포넌트가 있을 수 있을 때, 어떤 일이 발생하는지 상상해보자. 이제 우리는 이런 상황을 어떻게 최적화 할 수 있는지 방법을 살펴보자.

React DevTools를 사용하면 Components 탭 -> 환경 설정 아이콘 -> Highlight updates when components render를 통해 가상 렌더링을 강조 표시할 수 있다.

네이티브의 리렌더링을 보려면 Chrome DevTools에서 오른쪽의 점 3개 메뉴 -> More tools -> Rendering -> Paint flashing를 통해 볼 수 있다.

이제 React 리렌더링이 강조 표시된 애플리케이션을 클릭한 다음, 네이티브 렌더링을 클릭하면 React가 네이티브 렌더링을 얼마나 최적화하는지 확인할 수 있습니다.

## React는 언제 리렌더링 될까?

### React는 컴포넌트의 상태가 변경될 때마다 렌더링을 예약한다.

렌더링 예약은 이 작업이 즉시 수행되지 않는다는 것을 의미한다.  
React는 이에 가장 적합한 순간을 찾기 위해 노력할 것이다.

상태를 변경한다는 것은 `setState` 함수를 실행할 때, React 트리거가 업데이트된다는 것을 의미한다. 이는 컴포넌트의 render 함수가 호출된다는 것을 의미할 뿐만 아니라, props 변경 여부와 관계 없이 모든 하위 컴포넌트들이 리렌더링 된다는 것을 의미한다.

애플리케이션이 제대로 구조화되어 있지 않은 경우, 상위 노드를 업데이트하면 모든 자식들의 render 함수를 실행해야하기 때문에 예상보다 훨씬 더 많은 자바스크립트를 실행하게 될 수 있다.

### props가 변경될 때, React 컴포넌트가 업데이트되지 않는 이유는 무엇인가?

props가 변경되었음에도 React가 구성 요소를 업데이트하지 않는 2가지 일반적인 이유가 있다.

1. props가 `setState`를 통해 올바르게 업데이트되지 않았다.
2. props에 대한 참조가 동일하게 유지된다.

props 객체를 직접 변경하는 것은 변경 사항을 트리거하지 않으며 React가 변경사항을 인식하지 못하기 때문에 허용되지 않는다.

```js
this.props.user.name = "Felix"
```

이렇게 하지말고 부모 컴포넌트의 상태를 변경해야한다.

```js
const Parent = () => {
  const [user, setUser] = React.useState({ name: "Felix" })
  const handleInput = (e) => {
    e.preventDefault()
    setUser({
      ...user,
      name: e.target.value,
    })
  }

  return (
    <>
      <input onChange={handleInput} value={user.name} />
      <Child user={user} />
    </>
  )
}

const Child = ({ user }) => <h1>{user.name}</h1>
```

## React 컴포넌트를 강제로 리렌더링하기

전문적으로 React를 이용하여 일해 온 2년동안, 강제 리렌더링이 필요한 순간은 한번도 오지 않았다. 만약 당신에게 그 순간이 왔다면 업데이트되지 않은 React 컴포넌트들을 처리하는 더 좋은 보편적인 방법이 있을 것이다.

강제 업데이트가 필요한 경우 다음과 같이 할 수 있다.

React의 클래스형 컴포넌트

```js
this.forceUpdate()
```

React의 함수형 컴포넌트

```js
const [state, updateState] = React.useState()
const forceUpdate = React.useCallback(() => updateState({}), [])
```

React hooks에서는 forceUpdate기능을 사용할 수 없지만 다음과 같이 컴포넌트 상태를 변경하지 않고 React.useState를 통해 강제 업데이트를 할 수 있다.

## 리렌더링을 최적화하는 방법

비효율적인 리렌더링의 예로는 상위 컴포넌트가 하위 컴포넌트의 상태를 제어하는 경우이다. 컴포넌트의 상태가 변경되면 모든 자식들이 리렌더링 된다.

React.memo는 props가 변경되지 않았을 때 렌더링되는 것을 방지하는 함수이다.

```js
const TileMemo = React.memo(({ children }) => {
  let updates = React.useRef(0)
  return (
    <div className="black-tile">
      Memo
      <Updates updates={updates.current++} />
      {children}
    </div>
  )
})
```

React 클래스에서 동등한 것은 React.PureComponent이다.

`shouldComponentUpdate` 함수는 React의 라이프 사이클 함수중 하나이며 클래스형 컴포넌트의 업데이트 시점을 React에게 알려줌으로써 렌더링 성능을 향상시킬 수 있다.

이것의 인자 값들은 컴포넌트가 렌더링하려는 다음 props 및 다음 상태이다.

```js
shouldComponentUpdate(nextProps, nextState) {
  // return true or false
}
```

이 함수의 사용은 매우 간단하다. true를 반환하면 React가 render 함수를 호출하고 false를 반환하면 이를 방지한다.

React에서는 다음을 수행하는 것이 매우 일반적이다. 무엇이 문제인지 찾아보자.

```js
<div>
  {events.map((event) => (
    <Event event={event} />
  ))}
</div>
```

key 속성이다. 이것이 왜 중요할까?

경우에 따라 React 컴포넌트를 식별하고 성능을 최적화하기 위해 key 속성에 의존한다.
위의 예시에서, 이벤트가 배열의 시작 부분에 추가되면 React는 첫 번째 원소와 이후 모든 원소들이 변경되었다 생각하고 해당 요소를 리렌더링 합니다. 요소에 key 속성을 추가함으로써 이를 방지할 수 있습니다.

```js
<div>
  {events.map((event) => (
    <Event event={event} key={event.id} />
  ))}
</div>
```

배열의 인덱스를 key로 사용하는 것을 피하고 내용을 식별하는 것을 key로 사용하자.  
key는 형제간에만 고유해야 한다.

## 컴포넌트의 구조

리렌더링을 개선하는 더 좋은 방법은 코드를 약간 재구성하는 것이다.

로직을 어디에 두는지 주의해야한다.  
애플리케이션의 루트 컴포넌트에 모든 것을 넣는다면, 내부에 있는 모든 React.memo 함수는 성능 문제를 해결하는데에 도움이 되지 않는다.

만약 데이터가 사용하는 곳에 더 가깝게 배치한다면, React.memo가 필요하지 않을 수도 있다.
