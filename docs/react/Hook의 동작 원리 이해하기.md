# Hook은 어떤 방식으로 동작할까?

클로저를 활용해서 useState를 구현해보자.

```js
function useState(initialVal) {
  let _val = initial
  const state = () => _val
  const setState = (newVal) => {
    _val = newVal
  }

  return [state, setState]
}

const [count, setCount] = useState(1)

console.log(count()) // 1
setCount(2)
console.log(count()) // 2
```

컴포넌트에서 Hook 사용하기

리액트에서는 위와 같이 함수로 호출하지 않고 변수로 사용하므로 useState함수를 변경해보자. 먼저 hook을 React 모듈 안으로 넣는다. 이렇게 하면 React는 useState를 반환하므로 사용법이 React.useState로 달라진다.

```js
const React = (function () {
  function useState(initialVal) {
    let _val = initialVal
    const state = () => _val
    const setState = (newVal) => {
      _val = newVal
    }

    return [state, setState]
  }

  return { useState }
})()

const [count, setCount] = React.useState(1)

console.log(count()) //1
setCount(2)
console.log(count()) //2
```

그리고 안에 훅을 넣은 함수인 Component를 만든다.

```js
function Component() {
  const [counet, setCount] = React.useState(1)

  return {
    render: () => console.log(count),
    click: () => setCount(count + 1),
  }
}
```

이제 React에게 어떻게 컴포넌트를 render할 것인지 가르쳐줘야한다. 따라서 Component를 받는 render함수를 추가한다. Component는 함수이므로 호출할 수 있다. 그리고 객체를 리턴하므로 마찬가지로 render도 호출할 수 있다.

```js
const React = (function () {
  function useState(initialVal) {
    let _val = initialVal
    const state = () => _val
    const setState = (newVal) => {
      _val = newVal
    }

    return [state, setState]
  }

  function render(Component) {
    const C = Component()
    C.render()
    return C
  }

  return { useState, render }
})()
```

```js
function Component() {
  const [count, setCount] = React.useState(1)

  return {
    render: () => console.log(count),
    click: () => setCount(count + 1),
  }
}

var App = React.render(Component) // ƒ state() {}
App.click()
var App = React.render(Component) // ƒ state() {}
```

지금은 콘솔에 state함수가 찍히므로 \_val를 위로 끌어올리고 getter 함수를 제거하면 잘 동작한다.

```js
const React = (function () {
  let _val
  function useState(initialVal) {
    const state = _val || initialVal
    const setState = (newVal) => {
      _val = newVal
    }

    return [state, setState]
  }

  //...
})()

var App = React.render(Component) // 1
App.click()
var App = React.render(Component) // 2
```

## Hook을 여러번 사용하기

그런데 위의 예시는 훅을 여러개 가진다면 제대로 동작하지 않는다.

```js
function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  }
}

var App = React.render(Component) // {count: 1, text: "apple"}
App.click()
var App = React.render(Component) // {count: 2, text: 2} 🥲
App.type("orange")
var App = React.render(Component) // {count: "orange", text: "orange"} 🥲
```

문제점 개선 1

지금은 \_val이라는 하나의 변수만 가지고 있으므로 계쏙 값을 덮어쓰기 떄문이다. 따라서 배열과 인덱스를 이용하여 변경하자.

```js
const React = (function () {
  let hooks = []
  let idx = 0

  function useState(initialVal) {
    const state = hooks[idx] || initialVal
    const setState = (newVal) => {
      hooks[idx] = newVal
    }

    idx++ // 다음 훅을 받을 수 있게 인덱스 증가
    return [state, setState]
  }

  function render(Component) {
    const C = Component()
    C.render()
    return C
  }

  return { useState, render }
})()

function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  }
}

var App = React.render(Component) // {count: 1, text: "apple"}
App.click()
var App = React.render(Component) // {count: 2, text: "apple"}
App.type("orange")
var App = React.render(Component) // {count: "orange", text: "apple"} 🥲
```

문제점 개선 2

이번에는 click은 잘 동작하지만 text를 orange로 하면 count가 orange로 바뀌어 버린다. App컴포넌트가 render되면 useState함수를 호출하고, 그때마다 계속해서 index가 증가되기 때문이다. 따라서 render될 때마다 hook의 index를 0으로 초기화한다.

```js
const React = (function () {
  let hooks = []
  let idx = 0

  function useState(initialVal) {
    const state = hooks[idx] || initialVal
    const setState = (newVal) => {
      hooks[idx] = newVal
    }

    idx++
    return [state, setState]
  }

  function render(Component) {
    idx = 0
    const C = Component()
    C.render()
    return C
  }

  return { useState, render }
})()

function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  }
}

var App = React.render(Component) // { count: 1, text: 'apple' }
App.click()
var App = React.render(Component) // { count: 1, text: 'apple' } 🥲
App.click()
var App = React.render(Component) // { count: 1, text: 'apple' } 🥲
App.type("orange")
var App = React.render(Component) // { count: 1, text: 'apple' } 🥲
App.type("peach")
var App = React.render(Component) // { count: 1, text: 'apple' } 🥲
```

문제점 개선 3

그러면 상태가 바뀌지 않는데 render된 후에 useState가 호출되므로 증가된 index의 값에 저장이 되기 때문이다.

실제로 hooks 배열을 살펴보면 첫번째, 두번째 인자는 비어있고 세번째 인자에 setState값이 저장되어있다. 그렇기 때문에 계속 상태가 변하지 않은채로 계속 출력된 것이다.

```js
const React = (function () {
  let hooks = []
  let idx = 0

  function useState(initialVal) {
    const state = hooks[idx] || initialVal
    const setState = (newVal) => {
      hooks[idx] = newVal
      console.log(hooks)
    }

    idx++
    return [state, setState]
  }

  //...
})()

// { count: 1, text: 'apple' }
// [ <2 empty items>, 2 ]
// { count: 1, text: 'apple' }
// [ <2 empty items>, 2 ]
// { count: 1, text: 'apple' }
// [ <2 empty items>, 'orange' ]
// { count: 1, text: 'apple' }
// [ <2 empty items>, 'peach' ]
// { count: 1, text: 'apple' }
```

따라서 이걸 고치려면 setState 안의 index가 useState에 의해서 변하지 않게 freeze 한다. 이렇게 하면 useState가 호출된 순간\_idx를 사용하고 정상적으로 동작한다.

```js
const React = (function () {
  let hooks = []
  let idx = 0

  function useState(initialVal) {
    const state = hooks[idx] || initialVal
    const _idx = idx
    const setState = (newVal) => {
      hooks[_idx] = newVal
    }

    idx++
    return [state, setState]
  }

  function render(Component) {
    idx = 0
    const C = Component()
    C.render()
    return C
  }

  return { useState, render }
})()

function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  }
}

var App = React.render(Component) // { count: 1, text: 'apple' }
App.click()
var App = React.render(Component) // { count: 2, text: 'apple' } 😀
App.click()
var App = React.render(Component) // { count: 3, text: 'apple' } 😀
App.type("orange")
var App = React.render(Component) // { count: 3, text: 'orange' } 😀
App.type("peach")
var App = React.render(Component) // { count: 3, text: 'peach' } 😀
```

## 조건문 내에서의 훅

리액트 공식 문서에서는 반복문, 조건문, 중첩된 함수 내에서 Hook을 호출하면 안된다고 적혀있다. 이 규칙을 따라야 컴포넌트가 렌더링 될 때마다 동일한 순서로 Hook이 호출되는 것이 보장되기 때문이다.

만약 아래처럼 조건문 안에 useState를 넣는다면 두번째 Hook의 index는 1이어야하지만, 조건에 따랏 첫번째 Hook이 실행되지 않을 수도 있으므로 index가 0이 될수도 있다는 문제가 있다. 따라서 순서가 보장되지 않으므로 아래와 같이 사용해서는 안된다.

```js
function Component() {
  if (Math.random() > 0.5) {
    const [count, setCount] = React.useState(1) // ❌
  }

  const [text, setText] = React.useState("apple")

  return {
    //...
  }
}
```

## useEffect 구현하기

이번에는 useEffect를 만들어보자. useEffect 훅은 콜백과 dependency 배열을 받는다. 먼저 변수 hasChanged를 이용하여 변경되었는지 아닌지를 확인한다. 그리고 dependency가 변경되면 콜백을 실행한다.

그 다음 변화를 감지하려면 old dependencies와 new dependencies의 차이가 필요하므로 dependencies를 저장해야한다. 따라서 호출되고 나면 hooks배열안에 저장한다. 그 이후에는 oldDeps가 존재하면 newArray와 비교하는 작업을 통해 hasChanged를 변경한다.

```js
const React = (function () {
  let hooks = []
  let idx = 0

  function useState(initialValue) {
    //...
  }

  function render(Component) {
    //...
  }

  function useEffect(cb, depArray) {
    const oldDeps = hooks[idx]
    let hasChanged = true // default

    if (oldDeps) {
      hasChanged = depArray.some((dep, i) => !Object.is(dep, oldDeps[i]))
    }

    // 변경을 감지
    if (hasChanged) {
      cb()
    }

    hooks[idx] = depArray
    idx++
  }

  return { useState, render, useEffect }
})()
```

배열에 따른 결과 확인

이제 useEffect를 사용해보자. 두번째 인자에 빈 배열을 넣으면 처음에만 실행된다.

```js
function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  React.useEffect(() => {
    console.log("--- 실행됨! ---")
  }, [])

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  }
}

var App = React.render(Component)
App.click()
var App = React.render(Component)
App.type("orange")
var App = React.render(Component)

// --- 실행됨! ---
// { count: 1, text: 'apple' }
// { count: 2, text: 'apple' }
// { count: 2, text: 'orange' }
```

이제 배열에 count를 넣는다면, count가 업데이트 될 때 실행되는 것을 볼 수 있다. 물론 text를 넣어도 마찬가지로 text가 업데이트 될 때 실행된다.

```js
function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  React.useEffect(() => {
    console.log("--- 실행됨! ---")
  }, [count])

  return {
    //...
  }
}

var App = React.render(Component)
App.click()
var App = React.render(Component)
App.type("orange")
var App = React.render(Component)

// --- 실행됨! ---
// { count: 1, text: 'apple' }
// --- 실행됨! ---
// { count: 2, text: 'apple' }
// { count: 2, text: 'orange' }
```

만약 배열을 제거한다면 매번 실행될 것이다.

```js
function Component() {
  const [count, setCount] = React.useState(1)
  const [text, setText] = React.useState("apple")

  React.useEffect(() => {
    console.log("--- 실행됨! ---")
  })

  return {
    //...
  }
}

var App = React.render(Component)
App.click()
var App = React.render(Component)
App.type("orange")
var App = React.render(Component)

// --- 실행됨! ---
// { count: 1, text: 'apple' }
// --- 실행됨! ---
// { count: 2, text: 'apple' }
// --- 실행됨! ---
// { count: 2, text: 'orange' }
```

### Object.is 와 ===

위의 예시에서 비교를 위해 사용한 Object.is는 첫번째 인자와 두번째 인자가 같은2ㅣ를 판정하는 메서드인데, 비교 연산자 === 와는 달리 NaN와 -0과 0 비교가 가능하다.

```js
NaN === NaN // false
Object.is(NaN, NaN) // true

0 === -0 // true
Object.is(0, -0) // false
```

이렇게 클로저 개념을 이용해 간단히 useState, useEffect를 구현해보면서, 리액트 훅이 어떤 원리로 작동되는지 대략적으로 살펴볼 수 있었따. 더 자세한 설명은 아래 링크를 참조하자.

https://www.youtube.com/watch?v=KJP1E-Y-xyo
