# useState vs useRef(2)

useState와 useRef의 기능상의 공통점은 함수형 컴포넌트에서 동적으로 상태관리를 할 수 있게 해준다는 점이다.

간단한 예시로 버튼이 클릭되었을 때 주어진 state를 Humanscape!라는 string으로 변환하는 동일한 프로그램을 useState와 useRef를 통해 아래와 같이 다르게 구현할 수 있다.

```js
import React, { useState } from "react"

const Test = () => {
  const [letter, setLetter] = useState("")
  const onClick = () => {
    setLetter("Humanscape!")
  }

  return (
    <div>
      <button onClick={onClick}>Humanscape?</button>
      <b>{letter}</b>
    </div>
  )
}

export default Test
```

```js
import React, { useRef } from "react"

const Test2 = () => {
  const letter = useRef("")

  const onClick = () => {
    letter.current = "Humanscape!"
    console.log(letter.current)
  }

  return (
    <div>
      <button onClick={onClick}>Humanscape?</button>
    </div>
  )
}

export default Test2
```

useRef를 사용한 구현에서 글씨를 화면에 띄우지 않은 것은 useState와 다르게 useRef는 state를 변화시킨 후에 component를 re-render하지 않기 떄문이다.
다음은 React 공식 문서에 나와있는 내용이다.

> Keep in mind that useRef doesn't nottify you when its content changes. Mutating the .current proerty doesn't cause a re-render. If you want to run some code when React attaches or detaches a ref to a DOM node, you may want to use a callback ref instead.

따라서 useRef를 사용한 구현에서 state를 변화시키더라도 변화후에 re-render가 되지 않아 initial value로 나타날 것이기 때문에 rendering을 할 수는 있지만 변화했다는 의미가 존재하지 않아 따로 화면에 띄우지 않는 것이다.

반면, useState의 경우 선언한 state가 setter function에 의해 update될 경우, re-rendering process가 진행된다.
다음은 React 공식문서의 내용이다.

> The setState function is used to update the state. It accepts a new state value and enqueues a re-render of the component.

결과적으로 각각의 state에서 이용할 hook을 설계할 때, 주로 state의 rendering 여부를 바탕으로 결정하는 것이 좋다. Rendering이 필요한 state의 경우 useState를 이용하는 것이 간편하게 상태를 관리할 수 있으며 rendering이 필요하지 않은 state의 경우 useRef를 쓰는 것이 간단하게 코드를 작성할 수 있다.

## Using useRef for naming

useRef는 각 요소에 이름을 짓는 용도로도 사용할 수 있다. 다음은 useRef를 통해 상위 component에서 하위 component요소에 접근하여 input text box를 focusing하는 코드이다.

- forwardRef

```js
import React, { useRef } from "react"
import Mycomponent from "./Mycomponent"

const Naming = () => {
  const inputEl = useRef(null)

  const onClick = () => {
    inputEl.current.focus()
  }

  return (
    <div>
      <Mycomponent ref={inputEl} />
      <button onClick={onClick}>Focus Text</button>
    </div>
  )
}

export default Naming
```

```js
import React, { forwardRef } from "react"

const Mycomponent = forwardRef((props, ref) => {
  return (
    <div>
      <input ref={ref}></input>
    </div>
  )
})
```

## Different usage of useState vs useRef in code

다음과 같은 프로그램을 작성하고 싶다고 하자.

화면에 1초마다 1씩 증가하는 숫자를 띄우며, 이 숫자의 가시성을 조절할 수 있는 버튼이 존재한다. 숫자가 mount될 때는 0부터 차례로 증가하며, unmount될 때는 현재 화면의 숫자를 alert 해준다.

눈치 빠르신 분들은 이 코드를 왜 예시로 들었는지 벌써 아실 수 있다.

“화면에 1초마다 1씩 증가하는 숫자를 띄우며” 라고 했으니 렌더링에 관여한 state를 선언해야 하니까 useState를 사용해야 하고, mount와 unmount의 순간에 동작이 진행해야 하므로 useEffect를 사용하며 2번째 parameter로 빈 배열 []를 넣어주면 되겠구나!

라고 생각하셨으면 대략적으로 다음과 같이 코드를 작성하셨을 가능성이 높다.

```js
import React, { useState } from "react"
import Counter from "./Counter"

const App = () => {
  const [visibility, setVisibility] = useState(true)

  const onChnageVisibility = () => {
    setVisibility(!visibility)
  }

  return (
    <div>
      <button onClick={onChnageVisibility}>ChnageVisibility</button>
      {visibility && <Counter />}
    </div>
  )
}

export default App
```

```js
import React, { useEffect, useState } from "react"

const Counter = () => {
  const [counter, setCounter] = useState(0)

  useEffect(() => {
    const timer = setInterval(() => {
      setCounter((prev) => prev + 1)
    }, 1000)
    return () => {
      clearInterval(timer)
      alert(counter)
    }
  }, [])

  return (
    <div>
      <p>{counter}</p>
    </div>
  )
}

export default Counter
```

여기서 컴포넌트가 unmount될 때 alert값이 0으로 나타남을 확인할 수 있다.
잠시 생각을 거치신 여러분은 또 이렇게 생각할 수 있다.

아! useEffect가 초기에 함수를 생성할 때 return 부분에 counter를 0을 넣고 이후에 재생성하지 않기 떄문에 실제로 unmount에서도 0이 나타나는구나! 그렇다면 useEffect함수가 counter가 변할 때마다 재 생성 해주면 되겠다!

이렇게 생각하셔서 useEffect의 2번째 parameter로 [counter]를 넣으면 또 counter가 변하는 매 1초마다 cleanup 함수가 반환되어 alert가 1초마다 발생한다.

해결해보자.

```js
import React, { useEffect, useState, useRef } from "react"

const Counter = () => {
  const [counter, setCounter] = useState(0)
  const unmountvisual = useRef(0)

  useEffect(() => {
    const timer = setInterval(() => {
      setCounter((prev) => prev + 1)
      unmountvisual.current += 1
    }, 1000)
    return () => {
      clearInterval(timer)
      alert(unmountvisual.current)
    }
  }, [])

  return (
    <div>
      <p>{counter}</p>
    </div>
  )
}

export default Counter
```

이전 코드와 차이점은 useState대신 useRef를 사용해 alert를 진행한 것이다. 이것이 차이를 일으키는 이유는 useRef를 통해 반환된 객체는 component의 생애주기 내내 변화하는 값을 가리키고 있기 떄문이다.

다음은 React 공식 문서의 Hooks API Reference에 나와있는 내용이다.

> useRef returns a mutable ref object whose .current proerty is initialized to the passed argument (initialValue). The returned object will persist for the full lifetime of the component.

코드에서 useState와 useRef 이용의 차이에 대해 알아보았다. 같은 용도로 두 hook을 사용할 수 있지만 프로그래밍을 진행할 때 자신이 원하는 기능을 더 잘 수행할 수 있는 것을 골라서 사용하는 것이 좋다.

## Conclusion

useState와 useRef모두 상태관리를 위해 사용할 수 있다. 다만 useState의 경우 state 변화 후에 re-rendering을 진행하는 반면 useRef는 진행하지 않는다. 이러한 특성에 맞추어 렌더링이 필요한 state의 경우에는 useState를 사용하며 그렇지 않은 경우 useRef를 사용하는 것이 좋다. 이외에도 useRef는 이름을 지어 접근하는 용도와 생애주기 내내 변화하는 값을 가리키고 있다는 차별점을 가지고 있다. 프로그래밍에 앞서 두 기능을 돌아보면서 어떤 것이 원하는 기능을 구현하는데 적합할지 생각해보는 것이 좋다.
