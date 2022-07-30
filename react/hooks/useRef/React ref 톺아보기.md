# React ref 톺아보기

## Intro

React로 웹 프론트엔드 개발을 하다보면 React 만으로는 DOM을 조작하기 어려울 때가 있다. 가장 흔하게는 어떤 엘리먼트에 포커스 해야하는 경우.

React의 ref는 무엇이고, ref를 사용하는 이유에 대해 알아보자.

## ref가 뭘까?

- ref는 reference의 줄임말. 한국말로는 참조.
- ref는 render 메서드에서 생성된 DOM 노드나 React 엘리먼트에 접근하는 방법을 제공.
- ref는 순수 자바스크립트 객체.

ref를 console.log로 찍으면 current 프로퍼티 하나를 가진 객체가 나타남.

React는 이 객체를 통해 DOM에 대한 직접적인 접근을 가능케 해준다.

```js
class Example extends React.Component {
  constructor(props) {
    super(props)
    this.ref = React.createRef()
  }

  componentDidMount() {
    console.log(this.ref.current.style.backgroundColor)
  }

  render() {
    return (
      <div
        ref={this.ref}
        style={{ backgroundColor: "red", width: "100px", height: "100px" }}
      />
    )
  }
}
```

위와 같이 createRef함수를 통해 ref를 생성하면 ref객체를 React Element의 ref prop에 전달한다. -> 함수가 반환한 객체의 current 프로퍼티로 해당 React Element의 DOM에 접근할 수 있다.

## ref의 활용처: 비제어 컴포넌트

- JS로 DOM 요소에 focus 하기, 텍스트 선택영역, 혹은 미디어의 재생을 관리할 때.
- 애니메이션을 직접적으로 실행시킬 때.
- 서드 파티 DOM 라이브러리를 React와 같이 사용할 때.

위 경우를 `비제어 컴포넌트를 제어할 때`로 일축할 수 있다.

React 시스템 안에서 제어하지 않고, 순수 JS를 이용해 제어하는 컴포넌트를 비제어 컴포넌트라고 한다.

React 시스템을 이용하지 않는다는 이야기는 React가 제공하는 재조정과 같은 feature들을 이용하지 않음을 의미한다.

## 왜 current로 접근해야하나?

querySelector 처럼 그냥 DOM 요소를 반환해주면 좋을 텐데, 왜 createRef/useRef는 왜 객체를 반환하고 current 프로퍼티로 DOM을 전달할까?

이는 React가 가상 돔을 기반으로 작동하는 라이브러리라는 사실을 생각해보면 이유가 명확해진다.

공식 문서는 이렇게 설명한다.

> **컴포넌트가 마운트 될 때 React는 current 프로퍼티에 DOM 엘리먼트를 대입하고, 컴포넌트의 마운트가 해제될 때 current 프로퍼티를 다시 null로 돌려놓는다. ref를 수정하는 작업은 componentDidMount 또는 componentDitUpdate 생명주기 메서드가 호출되기 전에 이루어진다.**

실제 DOM에 React 노드가 렌더링 될 때 ref가 가리키는 DOM 요소의 주소 값은 확정된 것이 아니다.

즉 우리가 ref에 접근할 수 있는 시점은 React 노드가 실제로 DOM에 반영되는 시점부터이다. (라이프 사이클사이클 상에서는 componentDidMount)
이전에는 null이 current 프로퍼티로 담긴다. (useRef는 초기값을 인자로 전달해줄 수 있다.)

가상 DOM이 변경될 때 실제 DOM의 요소도 변경되는 경우가 있기 때문에 DOM이 업데이트되는 경우(componentDidUpdate)도 ref의 current값이 변경되게 된다.

이처럼 유동적이기에 React는 객체를 반환해 current 프로퍼티의 값을 계속해서 수정한다.

## 왜 DOM API를 쓰면 안될까?

```js
class Example extends React.Component {
  constructor(props) {
    super(props)
    this.ref = React.createRef()
  }

  componentDidMount() {
    console.log(
      "querySelector!: ",
      document.querySelector("#hi").style.backgroundColor
    )
  }

  render() {
    return (
      <div
        id="hi"
        style={{ backgroundColor: "red", width: "100px", height: "100px" }}
      />
    )
  }
}
```

ref는 특정 DOM 요소를 가져올 때 더 신뢰할 만하기 떄문이다.
예측하지 못한 상황(대개는 라이프사이클의 흐름을 예측하지 못하는 상황)으로 DOM 요소를 가져오지 못했다면 이는 해당 코드의 결함으로 이어질 것이다.

> 내생각을 추가하자면 예측하지 못한 상황이란 componentDidMount가 된 후에 돔 요소에 접근해야하는데 Mount 전에 dom에 접근하는 상황을 뜻한다.

또 만약 컴포넌트가 하나가 아닌 여러개가 생성되는 경우를 생각해보자. 이때 우리는 id나 class로 특정해서 원하는 DOM 요소를 가져올 수 있을까? 분명 쉽지 않을 것이다. DOM 요소를 특정할 수 있도록 관심 영역을 특정 컴포넌트로 제한하는 역할도 ref가 할 수 있다.

## ref 객체를 만드는 또 다른 방법, callback ref

React는 ref를 만드는 방법 중 createRef외에 또 하나의 방법을 제공한다. 그 이름은 callback ref이다.

별도의 API가 제공되는 것은 아니고, callback, ref prop에 ref 객체를 인스턴스 내 멤버 변수에 할당하는 callback 함수를 전달하면, react가 해당 멤버 변수에 ref객체를 전달해준다.

```js
class Example extends React.Component {
  ref = null

  setDivref(element) {
    this.ref = element
    console.log(this.ref.style.backgroundColor)
  }

  render() {
    return (
      <div
        ref={this.setDivref}
        style={{ backgroundColor: "red", width: "100px", height: "100px" }}
      />
    )
  }
}
```

위 예제는 콜백함수로 this.ref에 ref객체에 할당하고 전달받은 element의 background를 콘솔에 출력하는 예제이다.

callback ref는 함수를 ref에 전달해 ref가 할당될 당시 수행해야 할 동작이 있을 때 사용가능하다.

## 함수 컴포넌트에서 ref 사용하기, useRef와 forwardRef

함수 컴포넌트에서 createRef를 사용할 수는 있다. 하지만 함수컴포넌트는 상태가 바뀔 때마다 새롭게 호출되기에, ref가 가리키는 DOM 요소가 re-render되는 것과 상관없이 새로운 ref객체가 계속 만들어진다.

```js
const Example = () => {
  const ref = createRef(null)
  const [shouldRerender, setShouldRerender] = useState(false)

  useEffect(() => {
    console.log(ref)
  }, [ref])

  const rerender = () => {
    setShouldRerender(!shouldRerender)
  }

  return (
    <div>
      <div
        ref={ref}
        style={{ backgroundColor: "red", width: "100px", height: "100px" }}
      />
      <button onClick={rerender}> rerender</button>
    </div>
  )
}
```

re-render 버튼을 누르면 해당 컴포넌트의 상태가 바뀌기 때문에 함수 컴포넌트가 다시 호출됩니다. createRef를 사용하면 ref를 계속 만듦니다.

하지만 이를 useRef로 바꾼다면 이름에서 알 수 있듯이 hook으로 함수 호출에 관계없이 state를 유지합니다.

함수 컴포넌트를 사용할 때는 useRef를 사용해야 합니다.

또한 함수컴포넌트는 ref를 포워딩할 수 있습니다. (forwardRef)

함수 컴포넌트에서 특정 요소에 ref를 전달하고 싶은 경우가 있습니다. 가장 흔한 케이스는 input 태그를 스타일링하기 위해 컴포넌트로 래핑했는데, 사용자 경험 향상을 위해 프로그램적으로 input에 focus하고 싶을 때 입니다.

ref를 prop으로 받아서 원하는 element에 전달해주면 되지 않을까요?

```js
function App() {
  const ref = useRef(null)

  useEffect(() => {
    console.log(ref)
  }, [ref])

  return (
    <div className="App">
      <StyledInput ref={ref} />
    </div>
  )
}

const StyledInput = ({ ref }) => {
  return <input ref={ref} />
}

export default App
```

위 코드는 에러를 뱉습니다. ref는 자식에게 전달되는 prop이 아닙니다.

```js
export default function App() {
  const ref = useRef(null)

  useEffect(() => {
    console.log(ref)
  }, [ref])

  return (
    <div className="App">
      <StyledInput inputref={ref} />
    </div>
  )
}

const StyledInput = ({ inputref }) => {
  return <input ref={inputref} />
}
```

사실 forwardRef라는 기능 없이도 이렇게 별도의 prop으로 전달해 줄 수 있다.
하지만 ref prop으로 전달해 줄 수 있다면, StyledInput의 코드를 보지 않고도 동작을 예상할 수 있게 되어 보다 더 직관적인 코드가 될 수 있다.

이때 forwardRef라는 HOC를 사용합니다.
이 HOC로 컴포넌트를 감싸면 컴포넌트의 두 번째 인자로 ref를 전달을 할 수 있게 된다.

```js
function App() {
  const ref = useRef(null)

  useEffect(() => {
    console.log(ref)
  }, [ref])

  return (
    <div className="App">
      <StyledInput ref={ref} />
    </div>
  )
}

const StyledInput = forwardRef((props, ref) => {
  return <input ref={ref} />
})
```
