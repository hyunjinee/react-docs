# State as a Snapshot

State 변수는 읽고 쓸 수 있는 일반 자바스크립트 변수처럼 보일 수 있습니다. 그러나 state는 스냅샷처럼 작동합니다. state를 설정하여도 이미 가지고 있는 state는 변경되지 않고 대신에 다시 렌더링을 실행합니다.

## state를 설정하여 렌더링을 실행합니다.

사용자 인터페이스는 클릭과 같은 사용자 이벤트에 대한 응답으로 직접 변경되는 것으로 생각할 수 있습니다. React에서는 이 멘탈 모델과 조금 다르게 작동합니다. 인터페이스가 이벤트에 반응하려면 상태를 업데이트해야 합니다.

## 렌더링은 적시에 스냅샷을 생성합니다.

렌더링은 React가 함수 컴포넌트를 호출하는 것을 의미합니다. 함수에서 반환하는 JSX는 시간에 따른 UI의 스냅샷과 같습니다. 컴포넌트의 props, 이벤트 핸들러, 로컬 변수들 모두 state를 사용하여 렌더링 시점에 계산됩니다. 사진이나 동영상 프레임과 달리 반환하는 UI 스냅샷은 상호작용 가능합니다. 여기에는 입력에 대한 응답으로 어떤 일이 발생하는지 지정하는 이벤트 핸들러와 같은 것이 포함됩니다. 그런 다음 React는 이 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결합니다. 결과적으로 버튼을 누르면 JSX에서 클릭 핸들러가 트리거됩니다.

React가 컴포넌트를 다시 렌더링할 때:

1. React가 함수를 다시 호출합니다.
2. 함수가 새 JSX 스냅샷을 반환합니다.
3. 그런 다음 React는 반환된 스냅샷과 일치하도록 화면을 업데이트합니다.

<img width="901" alt="image" src="https://user-images.githubusercontent.com/63354527/192101330-7290316f-f4fd-4acf-9400-4c74c2db6dbe.png">

컴포넌트의 메모리인 state는 함수가 반환된 후 사라지는 일반 변수와 다릅니다. State는 함수 외부에 있는 것처럼 React 자체에 실제로 "살아있습니다." 컴포넌트는 해당 렌더링의 상태값을 사용하여 모두 계산된 JSX의 새로운 props 및 이벤트 핸들러 세트와 함께 UI의 스냅샷을 반환합니다.

<img width="869" alt="image" src="https://user-images.githubusercontent.com/63354527/192101793-b141f2ae-5520-4e65-b20b-6be88e6d8db8.png">

다음 코드에서 alert에 무엇이찍힐지 예상해보자.

```js
import { useState } from "react"

export default function Counter() {
  const [number, setNumber] = useState(0)

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5)
          setTimeout(() => {
            alert(number)
          }, 3000)
        }}
      >
        +5
      </button>
    </>
  )
}
```

React에 저장된 상태는 경고가 실행될 때 변경되었을 수 있지만 사용자가 상호작용할 때 상태의 스냅샷을 사용하여 예약되었습니다. 상태 변수의 값은 이벤트 핸들러의 코드가 비동기적일지라도 렌더 내에서 절대 변경되지 않습니다. 해당 렌더에서 onClick 내부의 number 값은 setNumber(number + 5)가 호출된 후에도 계속 0입니다. React가 컴포넌트를 호출하여 UI의 스냅샷을 찍었을 때 값이 고정되어있습니다.

다음은 이벤트 핸들러가 타이밍 실수를 덜 하게 만드는 방식의 예입니다. 다음은 5초 지연 메시지를 보내는 폼입니다. 다음 시나리오를 상상해 보세요.

1. "Send" 버튼을 누르면 Alice에게 "Hello"가 전송됩니다.
2. 5초의 지연이 끝나기 전에 "To" 필드의 값을 "Bob"으로 변경합니다.

경고창에 표시될 것으로 예상되는 내용은 무엇입니까? "앨리스에게 인사했습니다"가 표시될까요? 아니면 "밥에게 인사했습니다"라고 표시될까요? 알고 있는 내용을 바탕으로 추측한 다음 시도해 보세요.

```js
import { useState } from "react"

export default function Form() {
  const [to, setTo] = useState("Alice")
  const [message, setMessage] = useState("Hello")

  function handleSubmit(e) {
    e.preventDefault()
    setTimeout(() => {
      alert(`You said ${message} to ${to}`)
    }, 5000)
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{" "}
        <select value={to} onChange={(e) => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  )
}
```

**React는 렌더링의 이벤트 핸들러 내에서 state를 고정으로 유지합니다.** 코드가 실행되는 동안 state가 변경되었는지를 걱정할 필요가 없습니다.

그러나 다시 렌더링하기 전에 최신 state를 읽고 싶다면 어떻게 해야 할까요? 다음 페이지에서 다룰 [state 갱신 함수](https://beta.reactjs.org/learn/queueing-a-series-of-state-updates)를 사용하고 싶을 것 입니다.

- state를 설정하면 새 렌더링이 요청됩니다.
- React는 선반에 있는 것처럼 구성요소 외부에 state를 저장합니다.
- `useState`를 호출하면 React는 렌더링의 state에 대한 스냅샷을 제공합니다.
- 변수와 이벤트 핸들러는 "생존"하지 않고 다시 렌더링됩니다. 모든 렌더에는 자체 이벤트 핸들러가 있습니다.
- 모든 렌더(및 그 안의 기능)는 항상 React가 해당 렌더에 부여한 상태의 스냅샷을 "볼" 것입니다.
- 렌더링된 JSX에 대해 생각하는 방식과 유사하게 이벤트 핸들러에서 state를 대체할 수 있습니다.
- 과거에 생성된 이벤트 핸들러는 생성되었을 당시 렌더링의 state 값을 가집니다.
