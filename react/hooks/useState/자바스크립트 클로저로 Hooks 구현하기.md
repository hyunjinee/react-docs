# 자바스크립트 클로저로 Hooks구현하기

2019년 싱가폴에서 열린 JSConf에서 Netlify의 개발자인 Shawn이 발표한 Getting Closure on React Hooks라는 발표에서 자바스크립트 Closure 개념을 이용하여 Hooks를 직접 구현해보는 내용입니다.

먼저 React Hooks를 구현해보기 전에, 클로저의 개념을 다시 되짚고 넘어갑시다. 클로저의 개념이 잘 성명되어있는 W3School에서 정의한 개념은 다음과 같습니다.

> Clusure makes it possible for a function to have 'private' variables.

```ts
function getAdd() {
  let foo = 1

  return function () {
    foo = foo + 1
    return foo
  }
}

const add = getAdd()

console.log(add())
console.log(add())
```

이런식으로 함수를 구성하면, add에 할당된 함수는 생성당시 외부 스코프에 있는 변수를 기억해 호출될 때마다 foo의 값에 접근 가능하고, 이 foo의 값은 외부에서는 접근이 불가능하기 때문에 W3schools의 정의와 같이 private한 변수를 가질 수 있게 되는 것입니다.

이와 같은 자바스크립트 클로저 개념을 이용해 구현한 useState는 다음과 같습니다.

```ts
function useState(initVal) {
  let _val = initVal
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

useState함수에서 리턴한 state값을 통해서만 \_val 변수에 접근 가능하게 하고, useState함수에서 리턴한 setState값을 통해서만 \_val 변수를 변경하도록 하는 것입니다.

위의 함수를 이용해 React를 구현한 내용은 다음과 같습니다.

```ts
const React = (function () {
  let _val

  function useState(initVal) {
    const state = _val || initVal
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

function Component() {
  const [count, setCount] = React.useState(1)

  return {
    render: () => console.log(count),
    click: () => setCount(count + 1),
  }
}

var App = React.render(Component)
App.click() // 1
var App = React.render(Component)
App.click() // 2
var App = React.render(Component)
App.click() // 3
```

React 함수의 메소드를 이용해서만 \_val 변수에 접근이 가능하도록, React 함수 내부에 useState와 render 함수를 만들어 넣어놓고, 간단한 click 메소드만 실행하는 Component 함수를 구현한 것입니다.

이 상태에서 state가 하나 이상이되면 어떨까요?

```ts
const React = (function () {
  let _val

  function useState(initVal) {
    const state = _val || initVal
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

function Component() {
  const [count, setCount] = React.useState(1)
  // 추가
  const [text, setText] = React.useState("apple")

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    // 추가
    type: (word) => setText(word),
  }
}

var App = React.render(Component) // {count: 1, text: "apple")
App.click()
var App = React.render(Component) // {count: 2, text: 2}
App.type("pear")
var App = React.render(Component) // {cout: "pear", text: "pear}
```

이렇게 useState만 추가하게 되면 같으 ㄴ변수 \_val에 저장되기 때문에 의도하지 않은 방식으로 결과가 출력됩니다. 따라서 변수는 배열 이미지로 확장될 필요가 있습니다.

```ts
const React = (function () {
  // 변경
  let hooks = []
  let idx = 0

  function useState(initVal) {
    // 변경
    const state = hooks[idx] || initVal
    const setState = (newVal) => {
      // 변경
      hooks[idx] = newVal
    }
    // 변경
    idx++
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

var App = React.render(Component) // {count: 1, text: "apple")
App.click()
var App = React.render(Component) // {count: 2, text: "apple"}
App.type("pear")
var App = React.render(Component) // {count: "pear", text: "apple"}
```

이런식으로 변수 \_val을 배열과 index를 이용해 저장시켜주고, 저장된 다음에는 index를 증가시켜 state가 저장되는 공간이 겹치지 않도록 해주었습니다. 그러나 결과는 처음에는 제대로 count에만 새로운 변수가 할당되도록 두었지만 두번째에는 또 의도하지 않은대로 담깁니다. 이는 App 컴포넌트가 렌더링될 때마다 useState 함수를 호출하게 되고, 그때마다 index를 증가시키기 때문입니다. 따라서 render 할 때마다 index를 다시 초기값으로 돌려주어야합니다.

```ts
const React = (function() {
  // 변경
  let hooks = [];
  let idx = 0;

  function useState(initVal) {
    const state = hooks[idx] || initVal;
    const setState = newVal => {
      hooks[idx] = newVal;
    };
    idx++;
    return [state, setState];
  }

  function render(Component) {
  // 변경
  function render(Component) {
    idx = 0;
    const C = Component();
    C.render();
    return C;
  }

  return { useState, render };
})();

function Component() {
  const [count, setCount] = React.useState(1);
  const [text, setText] = React.useState('apple');

  return {
    render: () => console.log({ count, text }),
    click: () => setCount(count + 1),
    type: (word) => setText(word),
  }
}

var App = React.render(Component); // {count: 1, text: "apple")
App.click();
var App = React.render(Component); // {count: 1, text: "apple"}
App.type('pear');
var App = React.render(Component); // {cout: 1, text: "apple"}
```

그래도 결과값이 이상하게 찍힙니다. 왜냐하면 setState(click, type 메소드)의 인덱스 값이 유동적이기 때문에 render된 후 증가하는 index값에 영향을 받아 원래 의도했던 배열의 첫번째, 두번째값에 저장되는 것이 아니라 증가된 index에 저장되기 때문입니다.

따라서 또 클로저 개념을 이용해서 setState안의 index값이 useState에 의해 변하지 않도록 freeze 시켜줍니다.

```ts
const React = (function () {
  // 변경
  let hooks = []
  let idx = 0

  function useState(initVal) {
    // 변경
    const _idx = idx
    const state = hooks[idx] || initVal
    const setState = (newVal) => {
      // 변경
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

var App = React.render(Component) // {count: 1, text: "apple")
App.click()
var App = React.render(Component) // {count: 2, text: "apple"}
App.type("pear")
var App = React.render(Component) // {count: 2, text: "pear"}
```

이렇게 변경시켜주면 setState가 생성될 당시 index(state가 할당된 당시의 index)를 기억하게 되고, setState가 호출될 때 이 값을 변화시키기 때문에 드디어 정상적으로 작동하게 됩니다!

React Hooks 규칙 중에 hooks를 최상위에서만 호출하고 반복문 또는 조건문안에서 호출하지 말라는 호출하지 말라는 규칙이 있는데 앞에서 구현한 개념을 보면 그 이유를 이해하실 수 있으실 것입니다. render될때마다 특정 index에 state가 순서대로 할당되므로 최상위에서 불러야하는 것입니다.
여기까지가 자바스크립트의 클로저개념으로 구현한 Hooks 내용이었습니다. 원본 영상에는 위에서 구현한 useState 말고도 useEffect 구현 내용도 있으니 궁금하신분들은 youtube에서 참고해보세요!

[youtube](https://www.youtube.com/watch?v=KJP1E-Y-xyo)
