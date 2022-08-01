# React Hooks Testing

먼저 테스트 대상이 될 간단한 React Hook 함수 하나를 작성해보자.

아래 `useToggle`함수는 true또는 false가 될 수 있는 상태 state와 그 상태값을 반전할 수 있는 함수 toggle을 배열에 담아 반환한다. 초기 상태값을 initialState 인자를 통해 받을 수 있으며, 인자를 넘기지 않는 경우 기본값으로 false를 사용.

```js
import { useState, useCallback } from "react"

const useToggle = (initialState = false) => {
  const [state, setState] = useState(initialState)
  const toggle = useCallback(() => setState((state) => !state), [])
  return [state, toggle]
}

export default useToggle
```

## React 컴포넌트를 통한 간접 테스트

일반적으로 React Hook은 해당 프로젝트 내의 다른 React Component에 의해서 쓰여지기 마련이다. 따라서 해당 React Hook을 사용하는 React Component를 테스트함으로써 간접적으로 React Hook을 테스트 할 수 있다.

예를들어, `useToggle`을 사용하는 `<ToggleButton/>`을 만들어보자.

```js
import useToggle from "./useToggle"

function ToggleButton({ initial = false }) {
  const [on, toggle] = useToggle(initial)

  return <button onClick={toggle}>{on ? "ON" : "OFF"}</button>
}

export default ToggleButton
```

그 다음 일반적인 컴포넌트를 테스트하듯이 react-testing-library를 이용해서 테스트 코드를 작성할 수 있다.

```js
import { render, screen } from "@testing-library/react"
import userEvent from "@testing-library/user-event"

import ToggleButton from "./ToggleButton"

test("button text changes from ON to OFF when clicked", () => {
  render(<ToggleButton />)

  const button = screen.getByRole("button")

  expect(button).toHaveTextContent("OFF")

  userEvent.click(button)

  expect(button).toHaveTextContent("ON")
})

test("button text is ON given initial set to true", () => {
  render(<ToggleButton initial={true} />)

  expect(screen.getByRole("button", { name: /on/i })).toBeInTheDocument()
})
```

첫번째 테스트에서는 버튼을 클릭하면 버튼 내부 문구가 OFF에서 ON으로 변경되었는지를 검증하고 두번째 테스트에서는 컴포넌트에 initial prop으로 true를 넘겼을 때 내부 문구가 ON인 버튼이 렌더링되는지 검증한다.

## React Hooks Testing Library 사용

React Hook함수 여러개를 별도의 프로젝트로 관리하는 경우, 위와 같이 다른 React Component을 통한 간접 테스트 전략은 적합하지 않다. 오직 테스팅 목적으로 실제로 사용하지 않는 불필요한 React Component를 작성해야하기 때문이다.

React Hooks Testing Library라는 좀 긴 이름을 가진 React Hook을 테스트 하기 위한 전용 라이브러리가 있다. 이 라이브러리를 이용하면 다른 React Component의 도움 없이도 React Hook을 직접 테스트를 할 수가 있다. (이제는 설치안해도 내장되어 있다.)

React Hooks Testing Library의 `renderHook()`함수에 React Hook 함수를 호출하는 코드를 인자로 넘기면 result 속성을 담고 있는 객체를 반환한다. 이 result 객체는 current 속성을 가지고 있는데, 이 속성을 통해서 해당 React Hook 함수의 반환값에 직접 접근 가능하다.

예를 들어, 위에서 작성한 useToggle() 훅 함수는 state 상테와 toggle 함수로 이루어진 배열을 리턴한다. 따라서 useToggle() 훅 함수를 인자로 renderHook() 함수를 호출하면 다음과 같이 result.current를 통해 state 상테와 toggle 함수에 접근할 수 있다.

```js
const { result } = renderHook(() => useToggle())
console.log(result.current[0]) // state 상태 출력
console.log(result.current[1]) // toggle 함수 출력
```

이를 바탕으로 테스트 코드를 작성해보자.

```js
import { renderHook, act } from "@testing-library/react-hooks"

import useToggle from "./useToggle"

test("update state from false to true when toggle is called", () => {
  const { result } = renderHook(() => useToggle())

  expect(result.current[0]).toBe(false)

  act(() => result.current[1]())

  expect(result.current[0]).toBe(true)
})

test("allows for initial value", () => {
  const { result } = renderHook(() => useToggle(true))

  expect(result.current[0]).toBe(true)
})
```

첫 번째 테스트에서는 toggle 함수가 호출하면 state 상태가 false에서 true로 변경되는지를 검증한다.
두 번째 테스트에서는 훅 함수를 호출할 때 인자로 true를 넘기면 상태가 true로 시작하는지를 검증한다.

여기서 눈 여겨 볼 점은 result.current[1]() 즉, toggle() 함수를 호출부를 act() 함수로 감싸줬다는 것이다. toggle() 함수를 호출하면 state 상태가 바뀌기 때문에 act() 함수를 통해 수동으로 DOM을 업데이트 해주는 것이다. `쉽게 얘기해서 React에게 상태가 업데이트되었으니 다시 랜더링을 하라고 지시해주는 것이 act() 함수의 역할이다.`
