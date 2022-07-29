# useState vs useRef

![thumbnail](https://velog.velcdn.com/images/hyunjine/post/5c8cfeb8-5f0f-46aa-8cf2-87f00f89f1d1/image.png)

React 프로젝트를 진행하면서 input의 값을 관리할 때 `useState로 관리할 것인가, useRef로 관리할 것인가`에 대해 팀원과 토론했습니다.

이 글은 그 [토론](https://github.com/team-yaza/mozi-client/issues/78)에 기반합니다.

먼저, useState와 useRef를 간단하게 비교하는 것으로 시작해보겠습니다.

## useState

React에서 컴포넌트는 자신의 상태 또는 props가 바뀌면 리렌더링됩니다.
상태를 관리하기 위해 React에서는 `useState`를 활용합니다.

```js
const [state, setState] = useState(initialState)
```

`useState`는 상태 유지 값과 그 값을 갱신하는 함수를 반환합니다. `setState` 함수는 새 state를 받아 컴포넌트 리렌더링 큐에 등록합니다.

컴포넌트는 다음 렌더링 시에 `useState`를 통해 반환받은 첫번째 값은 항상 갱신된 최신 state가 됩니다.

## useRef

Ref는 render 메서드에서 생성된 DOM 노드나 React 엘리먼트에 접근하는 방법을 제공합니다.

```js
function CustomTextInput(props) {
  // textInput은 ref 어트리뷰트를 통해 전달되기 위해서
  // 이곳에서 정의되어야만 합니다.
  const textInput = useRef(null)

  function handleClick() {
    textInput.current.focus()
  }

  return (
    <div>
      <input type="text" ref={textInput} />
      <button onClick={handleClick}>click me</button>
    </div>
  )
}
```

React 공식 문서에 의하면 ref의 바람직한 사용 사례는 다음과 같습니다.

- 포커스, 텍스트 선택영역, 혹은 미디어의 재생을 관리할 때
- 애니메이션을 직접적으로 실행시킬 때
- 서드 파티 DOM 라이브러리를 React와 같이 사용할 때

결국 React에서 ref는 DOM을 조작하기 위해 사용됩니다.
하지만 ref를 다른 용도로 사용할 수도 있습니다. 아래와 예시를 보겠습니다.

```js
const refContainer = useRef(initialValue)
```

`useRef`는 `.current` 프로퍼티로 전달된 인자(`initialValue`)로 초기화된 변경 가능한 ref 객체를 반환합니다.

`useRef`는 순수 자바스크립트 객체를 생성합니다.
또한 `useRef`로 만든 객체를 수정하는 것은 컴포넌트의 렌더링과 무관합니다.  
다시 말하면, `.current` 프로퍼티를 변형하는 것이 리렌더링을 발생시키지 않습니다.

본질적으로 `useRef`는 `.current` 프로퍼티에 변경 가능한 값을 담고 있는 `상자`와 같습니다.
`useRef`는 상자와 같으므로 `useState`처럼 컴포넌트 내의 변수 값을 조회, 수정하는 방법으로도 사용할 수 있습니다.

위 두 사례에 의하면, `useRef`는 일반적으로 특정 DOM을 지정하여 해당 돔의 속성값을 파악하거나 속성값을 변화시키는 용도로 사용할 수도 있고, 순수 자바스크립트 객체를 반환하기 때문에 값을 저장하는 `상자`로 사용할 수도 있습니다.

## useState vs useRef

`useState`와 `useRef`의 사용을 비교해보면 렌더링에서 차이점을 보입니다.

먼저 useState를 사용해서 input을 만들어 테스트를 해보겠습니다.

```js
function Input() {
  const [value, setValue] = useState("")
  return <input value={value} onChange={(e) => setValue(e.target.value)} />
}

export default Input
```

![](https://velog.velcdn.com/images/hyunjine/post/1203cdac-bdb3-4c52-98c5-88113745ab54/image.gif)

당연하게도 상태가 바뀔 때마다 리렌더링되는 모습을 볼 수 있습니다.

이제 `useRef`를 사용해서 input 컴포넌트를 테스트 해보겠습니다.

```js
function Input() {
  const inputRef = useRef(null)
  return <input ref={inputRef} />
}

export default Input
```

![](https://velog.velcdn.com/images/hyunjine/post/50749d8d-8a16-4c84-9840-2873da6b8073/image.gif)

애초에 상태로 관리하지 않으므로 리렌더링이 일어나지 않습니다.

여기서 고민했던 점은 `input 값을 변경할 때 렌더링이 필요한가`에 대한 부분이었습니다.

팀원과 제 생각이 같았던 부분은 `input을 입력하는 과정에서 렌더링이 이렇게 많이 일어나야하나?`라는 생각이었습니다.

반면 팀원과 제 생각이 달랐던 부분은 다음과 같습니다.

- 이현진: 'input의 입력값 또한 어플리케이션의 상태이기 때문에 `useState`로 관리해야한다'
- 팀원: 'input 입력값은 상태가 아니기 때문에 `useRef`로 리렌더링을 막아야한다.'

결국 input 값을 관리할 때 `useState vs useRef`라는 질문은 React에서 `input값이 상태인가`라는 질문으로 귀결됩니다.

## 생각

제 생각은 아래와 같습니다.

`상태란 주어진 시간에 대해 시스템을 나타내는 것으로 언제든지 변경될 수 있는 것`입니다.
사용자가 입력하는 값 또한 어플리케이션의 상태라고 할 수 있고, 시간의 흐름에 따라 변하는 input의 값은 `useState`로 관리해야합니다.

input을 state로 관리할 때 발생하는 리렌더링에 대한 부분에서는 과연 그 input을 리렌더링하는게 고비용 연산인가라는 의문을 제기하고 싶습니다. input을 타이핑할 때 발생하는 렌더링은 고비용 연산이 아니라고 생각합니다.

반면 `useRef`로 만들어진 ref객체는 DOM에 접근할 때나, 매 렌더링시에 만들어줘야하는 고비용 객체나 값을 저장할 때 사용하는 것이 옳다고 생각합니다.

결론은 `input에서 발생하는 사용자의 상호작용 또한 어플리케이션의 상태이므로 상태를 상태답게 관리하기 위해 useState를 사용해야한다.` 입니다.

**여러분은 어떻게 생각하시나요?**
