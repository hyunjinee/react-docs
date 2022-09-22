# A (Mostly) Complete Guide to React Rendering Behavior

## "렌더링"이란 무엇인가?

렌더링이란 React가 컴포넌트에게 현재 Props와 State에 기반하여 UI에서 어떻게 보여지고 싶은지 알려달라고 요청하는 과정입니다.

### 전체적인 렌더링 과정

렌더링 과정 동안, React는 컴포넌트 트리의 루트에서부터 시작하여 아래쪽으로 순환하며 업데이트가 필요로하다고 표시된 컴포넌트를 전부 찾습니다. 표시된 각각의 컴포넌트에 대해서 React는 클래스형 컴포넌트일 경우 `classComponentInstance.render()`또는 함수형 컴포넌트인 경우 `FunctionComponent()`를 호출하고 렌더 결과물을 저장합니다.

컴포넌트 렌더 결과물은 보통 JSX 구문으로 작성되며, JS가 컴파일되고 배포 준비가 되는 시점에서 `React.createElement()` 호출로 변환됩니다. `createElement`는 일반적인 JS 객체 형식의 React 엘리먼트를 반환하는데, 이 엘리먼트는 생성하고자하는 UI 구조를 설명합니다.

```tsx
// 다음과 같은 JSX 문법이:
return <SomeComponent a={42} b="testing">Text here</SomeComponent>

// 이런 식의 호출로 변환됩니다:
return React.createElement(SomeComponent, {a: 42, b: "testing"}, "Text Here")

// 그렇게해서 이런 엘리먼트 객체가 됩니다:
{type: SomeComponent, props: {a: 42, b: "testing"}, children: ["Text Here"]}
```

전체 컴포넌트 트리에서 렌더 결과물을 모두 수집하고 나서, React는 새로운 객체 트리("가상 DOM"으로 자주 불리죠)와 비교할 것 입니다. 그러고는 의도한대로 보여지기 위해 실제 DOM에 적용시켜야할 모든 변경 사항 목록을 수집합니다. 이러한 비교 및 계싼 과정을 `재조정(Reconciliation)`이라고 합니다.

그리고 나서 React는 이렇게 계산된 모든 변경사항을 하나의 `동기적 시퀀스(synchronous sequence)`로 실제 DOM에 적용시킵니다.

> Note:최근에 React 팀은 "가상 DOM"이라는 용어가 그리 대단한게 아니라고 밝혔습니다. Dan Abramov는 최근에 다음과 같이 얘기했습니다. 저는 "가상 DOM" 이라는 용어를 폐기했으면 합니다. 이 용어는 2013년에는 말이 됐습니다. 왜냐하면 그 때는 사람들이 React가 매번 렌더할 때마다 DOM노드를 생성한다고 가정했기 때문입니다. 하지만 최근에는 이렇게 가정하는 사람이 거의 없습니다. "가상 DOM"은 마치 무슨 DOM 관련 이슈에 대한 임시방편(Workaround)인 것 처럼 들립니다. 하지만 React는 그런게 아니에요. React는 "value UI" 입니다. React의 핵심 원칙은 UI는 문자열이나 배열처럼 그저 값(value)라는 것 입니다. 여러분은 이 값을 변수에 저장하고, 어디든지 전달할 수 있으며, JavaScript의 제어흐름 (Control Flow)등등을 사용할 수 있습니다. 이러한 표현력(expressiveness)이 핵심입니다. 변경 사항을 DOM에 적용하는 걸 막기 위한 비교 행위같은게 아닙니다. 심지어 React는 항상 DOM을 대표하지도 않습니다. 예를 들어 <Message recipientId={10} /> 같은 것은 DOM이 아닙니다. 개념적으로 React는 Message.bind(null, {recipentId: 10})와 같은 Lazy Function 호출을 대표합니다.

### 렌더와 커밋 단계

React 팀은 이 과정을 개념적으로 크게 2가지 단계로 나눴습니다.

- 렌더 단계: 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 과정이 이루어지는 단계
- 커밋 단계: 변경 사항을 실제 DOM에 적용하는 단계

커밋 단계를 거쳐서 DOM을 업데이트하고 나면, React는 요청된 DOM 노드와 컴포넌트 인스턴스를 가리키도록 모든 참조사항을 업데이트합니다. 그리고나서 `componentDidMount`와 `componentDidUpdate` 클래스 생명주기 메소드 또는 `useLayoutEffect` 훅을 동기적으로 실행합니다.

그후 React는 짧은 타임 아웃을 세팅하고, 타임 아웃이 끝나면 모든 useEffect 훅을 실행합니다. 이 단계는 "수동적 효과(Passive Effect)"라고도 알려져 있습니다.

여기 [훌륭한 React 생명주기 메소드 다이어그램](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)을 통해 클래스 생명 주기를 시각적으로 볼 수 있습니다.

> React에 곧 추가될 "Concurrent Mode"에서는, 브라우저가 이벤트를 처리할 수 있도록 렌더 단계의 작업을 잠시 멈출 수 있게됩니다. React는 나중에 적절한 시점에 해당 작업을 재개하거나, 폐기하거나 또는 재계산할 수 있습니다. 렌더 패스(Render Pass)가 완료되어도 React는 커밋 단계를 동기적으로 한 단계 진행할 것 입니다.

가장 핵심적인 부분은 "렌더링"과 "DOM을 업데이트하는 것"은 같은 것이 아니며 컴포넌트는 어떠한 가시적인 변화가 없이도 렌더링 될 수 있다는 점 입니다.

React가 컴포넌트를 렌더링할 때에는

- 컴포넌트는 바로 직전 결과물과 똑같은 결과물을 반환할 수도 있습니다. 이 경우에는 변경 사항이 없습니다.
- "Concurrent Mode"에서는 React는 컴포넌트 렌더링을 여러번 할 수 있겠지만, 다른 업데이트가 현재 작업을 무효화시키면 매번 렌더링 결과물을 폐기할 것 입니다.

## React는 렌더를 어떻게 다룰까?

### 렌더링 순서 만들기

최초의 렌더가 끝난 이후, React가 리렌더링을 Queue에 넣도록 하는 방법은 여러가지가 있다.

- 클래스형 컴포넌트
  - this.ssetState()
  - this.forceUpdate()
- 함수형 컴포넌트
  - `useState` setters
  - `useReducer` dispatchers
- 그 외
  - `ReactDOM.render(<App/>)`를 다시 호출(루트 컴포넌트에서 `forceUpdate`를 호출하는 것과 동일)

### 표준적인 렌더링 동작

> 기억해야할 가장 중요한 사실은 React는 기본적으로 부모 컴포넌트가 렌더링되면, 그 안에 있는 모든 자식 컴포넌트를 재귀적으로 렌더링한다는 점이다.

예를 들어서 A > B > C > D로 되어있는 컴포넌트 트리가 있다고 해봅시다. 이 컴포넌트들은 이미 페이지에 다 보여지고 있습니다. B 컴포넌트 안에는 버튼이 하나 있는데, 누르면 숫자가 1 증가합니다. 유저가 이 버튼을 한번 클릭하면 다음과 같은 동작이 일어납니다.

- B 컴포넌트에서 setState()가 호출되어 B의 리렌더링이 큐에 들어갑니다.
- React는 트리의 최상단부터 렌더 패스(Render Pass)를 시작합니다.
- React는 A에는 업데이트가 필요하다는 마크가 없는 것을 보고 그냥 지나칩니다.
- React는 B에는 업데이트가 필요하다는 마크가 있는 것을 보고 B를 렌더링합니다. B는 C를 리턴합니다.
- C는 업데이트 필요하다는 마크는 없지만 B가 렌더링되었기 때문에 React는 한 단계 밑으로 내려가서 C까지 렌더링합니다. C는 D를 리턴합니다.
- D 역시도 업데이트가 필요하다는 마크는 없지만, 부모 컴포넌트인 C가 렌더링되었기 때문에 React는 한 단계 밑으로 내려가서 D도 렌더링합니다.

다시 말하자면 일반적으로 컴포넌트가 렌더링되면 그 안에 있는 모든 컴포넌트 역시 렌더링된다는 것입니다.

또 주목해야할 점은 **일반적인 렌더링 과정에서, React는 Props가 변경되었는지 여부는 신경쓰지 않습니다.** 그저 부모 컴포넌트가 렌더링되었기 때문에 자식 컴포넌트도 무조건 렌더링하는 것 입니다.

이는 즉, <App> 컴포넌트에서 setState()를 호출하면 컴포넌트 트리 안에 있는 모든 컴포넌트가 렌더링된다는 것을 의미합니다. 결과적으로 React는 [매번 업데이트를 할 때마다 애플리케이션 전체를 다시 그리는 것처럼 동작](https://www.slideshare.net/floydophone/react-preso-v2)합니다.

이제 컴포넌트 트리 안에 있는 대부분의 컴포넌트는 직전과 똑같은 렌더링 결과물을 반환할 가능성이 높습니다. 따라서 React는 DOM에 변화를 줄 필요가 없습니다. 하지만 React는 계속해서 컴포넌트에게 렌더링을 요청하고 그 결과물을 비교해야만 할 것 입니다. 이 두 작업은 모두 시간과 노력이 꽤 걸리죠.

아무튼 기억해야할 점은, 렌더링은 절대로 나쁜게 아니라는 점입니다. **렌더링은 React가 DOM에 변화를 줘야할지 여부를 파악하는 방법일 뿐입니다.**

### React의 렌더링 규칙

React의 렌더링에 있어서 제일 핵심적인 규칙 중 하나는 바로 렌더링은 "순수"해야만 하며 사이드 이펙트를 만들어내서는 안된다는 것 입니다. 좀 혼란스러울 수 있을 겁니다. 왜냐하면 많은 사이드 이펙트들은 명확하지 않고 또 결과물을 망가뜨리거나 하지 않을 수 있기 때문입니다. 예를 들어, 엄격하게 말하자면 console.log()도 사이드 이펙트입니다. 하지만 아무것도 망가뜨리지 않죠. Prop을 변경하는 것 역시 명백한 사이드 이펙트이지만, 아무것도 망가뜨리지 않을 수 있습니다. 렌더링 도중에 AJAX 호출을 하는 행위도 명백한 사이드 이펙트이며, 요청의 타입에 따라 app에 예상하지 못한 영향을 끼칠 수 있습니다.

Sebastian Markbage가 작성한 [The Rules of React](https://gist.github.com/sebmarkbage/75f0838967cd003cd7f9ab938eb1958f)라는 훌륭한 글이다. 이 글에서 그는 `render`를 포함하는 서로 다른 React 생명 주기 메소드들을 설명했습니다. 또한 어떤 작업이 안전하고 "순수"한지 또 어떤 작업이 안전하지 않은지도 얘기했습니다. 글 전체를 읽는 것을 추천드리지만, 그래도 핵심만 요약해보겠습니다.

- 렌더링 로직은 다음과 같은 행위를 절대로 해서는 안됩니다.
  - 현재 존재하는 변수와 객체를 변경하는 행위
  - `Math.random()` 또는 `Date.now()`등의 랜덤 값을 만들어내는 행위
  - 네트워크 요청을 만들어내는 행위
  - 상태 업데이트를 Queue에 넣는 행위
- 다음과 같은 행위는 아마 괜찮을 수 있습니다.
  - 렌더링 도중에 새롭게 만들어진 객체를 변경하는 행위
  - 에러를 발생시키는 행위
  - 캐싱된 값처럼 아직 만들어지지 않은 데이터를 "Lazy 초기화(initialize)"하는 행위

### 컴포넌트 메타데이터와 Fibers

React는 현재 애플리케이션에 존재하는 모든 컴포넌트 인스턴스를 추적하는 내부 자료 구조를 갖고 있습니다. 이 자료 구조의 핵심 부분은 "피버 (Fiber)"라고 불리는 객체입니다. 피버에는 메타데이터 필드가 들어있는데, 이 메타데이터 필드에는 다음의 내용이 있습니다.

- 지금 현재 시점에서 컴포넌트 트리 안에서 렌더링되어야 하는 컴포넌트 유형
- 이 컴포넌트와 관련되어있는 현재 Props와 State
- 부모, 형제 그리고 자식 컴포넌트를 향한 포인터
- React가 렌더링 과정을 추적하기 위해 필요한 다른 내부 메타데이터

여기서 [React 17에서 피버 타입을 어떻게 정의](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiber.new.js#L47-L174)했는지 확인해보실 수 있습니다.

렌더링 패스 과정 동안, React는 이 피버 객체의 트리를 순환할 것이고, 새로운 렌더링 결과물을 계산해서 나온 업데이트된 트리를 생성할 것 입니다.

**이 피버 객체들은 실제 컴포넌트 Props와 State 값을 저장하고 있다는 점**을 주목하세요. 여러분이 컴포넌트에서 Props와 State를 보는 것은, React가 피버 객체에 저장되어있는 값들을 볼 수 있게끔 허가해 주었기 때문에 가능한 것입니다. 사실, 특히 클래스형 컴포넌트의 경우, React는 컴포넌트를 렌더링하기 바로 직전에 `componentInstance.props = newProps`를 똑같이 복제합니다. 따라서 `this.props`가 존재하는 이유는 React가 내부 자료 구조의 참조를 복사했기 때문입니다. 그런 의미에서 컴포넌트는 React 피버 객체에 대한 일종의 외관이라고 볼 수 있습니다.

비슷하게, React 훅이 동작하는 이유는 [React가 컴포넌트에서 쓰는 모든 훅들을 컴포넌트의 피버 객체와 연결된 연결리스트로 만들어서 저장](https://www.swyx.io/hooks/)해놓기 때문입니다. React가 함수형 컴포넌트를 렌더링할 때, React는 피버로부터 훅 관련 연결 리스트를 받아옵니다. [다른 훅을 호출한다면 그 때마다 `React는 훅 객체에 저장되어있는 값 중 적절한 값을 찾아 반환할 것입니다. (state나 useReducer에서 사용하는 dispatch 같은 값들)`](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L795)

부모 컴포넌트가 자식 컴포넌트를 처음으로 렌더링할 때, React 컴포넌트의 "인스턴스"를 추적하기 위한 피버 객체를 생성합니다. 클래스형 컴포넌트에서는 Reactsms [const instance = new YourComponent(props)를 호출](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberClassComponent.new.js#L653)하여 실제 컴포넌트 인스턴스를 피버 객체에 저장합니다. [함수형 컴포넌트에서는, YourComponentType(props)를 함수로 호출](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L405)합니다.

### 컴포넌트 타입과 재조정 (Reconciliation)

[공식 문서]()에서 설명되어있듯이, React는 기존에 존재하는 컴포넌트 트리와 DOM 구조를 최대한 재활용해서, 최대한 효율적으로 리렌더링을 진행하려고 합니다. 만약 React가 같은 유형의 컴포넌트 또는 HTML 노드를 트리의 동일한 위치에 렌더링 해야한다면, React는 새로 만들어내는 대신에 기존의 것을 재사용 하려고 할 것 입니다. 이는 즉 같은 위치에 같은 유형의 컴포넌트를 렌더링하도록 요청이 들어오는 동안 React는 컴포넌트 인스턴스를 계속 유지한다는 것 입니다. 클래스형 컴포넌트인 경우, React는 사실 실제 컴포넌트 인스턴스와 똑같은 인스턴스를 사용합니다. 함수형 컴포넌트의 인스턴스의 경우 클래스형처럼 실제 인스턴스를 갖고 있지는 않지만, <MyFunctionComponent />가 "이 유형의 컴포넌트는 이곳에서 보여질 것이고 계속 유지될 것이다."라는 의미로 인스턴스를 대신한다고 볼 수 있습니다.

그렇다면 결과물이 언제 그리고 어떻게 변경되었는지를 React가 어떻게 알 수 있을까요?

React의 렌더링 로직은 먼저 type 필드에 기반하여 엘리먼트를 비교합니다. 이 때 `===`같은 참조 비교를 사용합니다. 만약 어떤 엘리먼트가 다른 타입으로 변경되었다면 (예를 들어 div -> span 또는 ComponentA -> ComponentB) React는 전체 트리가 변경되었다고 가정하고 비교 절차의 속도를 높일 것 입니다. 결과적으로 React는 모든 DOM노드를 포함하여 현재 존재하는 모든 컴포넌트 트리 영역을 파괴할 것 입니다. 그리고 나서 새로운 컴포넌트 인스턴스로 다시 만들 것입니다.

코드로 설명해보자면, 다음과 같은 행위는 절대 해서는 안됩니다.

```jsx
function ParentComponent() {
  // This creates a new `ChildComponent` reference every time!
  function ChildComponent() {}

  return <ChildComponent />
}
```

대신에, 컴포넌트는 항상 분리해서 선언해야합니다.

```jsx
// This only creates one component type reference
function ChildComponent() {}

function ParentComponent() {
  return <ChildComponent />
}
```

### Keys와 재조정 (Reconciliation)

React 컴포넌트 인스턴스를 식별하는 또 다른 방법은 바로 `key`라는 의사 Prop(pseudo-prop)을 활용하는 방법입니다. `key`는 React에게 있어서 가이드 라인의 느낌이지 실제 컴포넌트로 전달되는 요소는 아닙니다. React는 `key`를 컴포넌트 유형의 특정 인스턴스를 식별하기 위한 고유 식별자로 취급합니다.

`key`가 주로 활용되는 곳은 배열을 렌더링하는 경우입니다. 만약 재정렬하거나, 요소를 추가 또는 삭제등의 방식으로 변경도리 수 있는 배열을 렌더링하는 경우 `key`는 특히 더 중요합니다. 왜냐하면 `Key`는 가능하다면 고유한 값으로 사용해야하기 때문입니다. 배열의 인덱스는 정말 최후의 수단으로 활용하는 것 입니다.

이게 왜 중요한지 예시를 들어보겠습니다. 10개의 TodoListItem 컴포넌트를 가지고 있는 배열을 렌더링하는데 배열 인덱스를 `key`로 사용한다고 해봅시다. React는 `0~9`의 `key`를 갖고 있는 10개의 요소를 보겠죠. 여기서 6번째와 7번째 요소를 지우고 끝에 새로운 요소 3개를 추가합니다. 그럼 이제 0~10의 `key`를 가지고 있는 11개의 요소가 됩니다. 결론적으로 10개에서 11개가 되었기 때문에 React입장에서는 새로운 요소가 하나 더 추가된 것처럼 보일 것 입니다. 따라서 React는 기존에 있는 DOM 노드와 컴포넌트 인스턴스를 재사용하려고 하겠죠. 하지만 이는 즉, \<TodoListItem key={6}>가 배열에서 8번째 요소를 전달받아 렌더링 될 것이라는 의미입니다. 따라서 컴포넌트 인스턴스는 항상 유지되지만 완전히 다른 데이터 객체를 pro으로 받습니다. 이렇게 되면 동작은 하겠으나 예상치 못한 결과로 이어질 수 있습니다. 또한, React는 이제 이 배열 요소들 중 몇 개를 업데이트하여 텍스트와 다른 DOM 요소들을 변경해야합니다. 왜냐하면 지금 존재하는 배열 요소들이 이전과 다른 데이터를 보여줘야하기 때문이죠. 하지만 실제 배열에는 아무런 변화가 없기 때문에 이러한 업데이트는 정말로 불필요한 것 입니다.

만약 대신에 배열의 요소 각각에 key={todo.id}를 사용한다면, React는 정확하게 2개의 요소가 삭제되고 3개의 요소가 추가되었음을 알 수 있습니다. 따라서 정확히 2개의 컴포넌트 인스턴스와 관련된 DOM 요소를 삭제하고 3개의 새로운 컴포넌트 인스턴스와 관련된 DOM 요소를 생성할 것입니다. 바뀌지도 않은 컴포넌트를 불필요하게 업데이트하는 것 보다 훨씬 좋은 방법이죠.

key는 또한 배열에 있는 컴포넌트 인스턴스를 식별하는 것에도 유용합니다. 언제든지 React 컴포넌트에 key를 추가하여 식별자를 만들 수 있고, 이 key를 변경하면 React는 기존의 컴포넌트 인스턴스를 제거하고 새로 생성할 것입니다. 가장 일반적인 사용 예시는 배열 + 상세 정보 폼 (Form) 조합입니다. 이때 상세 정보 폼은 현재 선택된 배열 요소의 데이터를 보여줍니다. 현재 선택된 배열 요소가 변경되면 \<DetailForm key={selectedItem.id}>가 렌더링되어 기존의 요소를 제거하고 새로운 요소를 재생성할 것입니다. 따라서 폼 안에 있는 상태가 너무 오래되어 생길 수 있는 문제들을 방지할 수 있습니다.

### 렌더링 배치 (Batching)와 타이밍

기본적으로, `setState()`가 호출되면 React는 새로운 렌더링 패스를 시작하고, 동기적으로 실행하여 반환합니다. 하지만 React는 또한 렌더링 배치 형식의 최적화를 자동으로 적용합니다. 여기서 이 렌더링 배치는 다수의 `setState()` 호출로 인해 단일 렌더링 패스가 대기열에 저장되고 실행되는 경우를 얘기하며, 일반적으로 약간의 지연이 발생합니다.

React 공식 문서에서는 [상태 업데이트는 아마도 비동기적으로 발생한다](https://reactjs.org/docs/state-and-lifecycle.html#state-updates-may-be-asynchronous)고 언급되어있습니다. 이는 렌더링 배치를 말하는 것입니다. 특히, React는 자동적으로 React 이벤트 핸들러에서 발생하는 상태 업데이트를 일괄처리합니다. 이는 즉, React 이벤트 핸들러는 일반적인 React 앱의 코드 중에서 상당히 큰 부분을 차지하기 때문에, 앱 안에서 일어나는 상태 업데이트는 사실 대부분 일괄 처리 된다는 것을 의미합니다.

React는 렌더링 배치 이벤트 핸들러를 `unstable_batchedUpdates`라고 알려져있는 내장 함수로 감싸는 방법으로 실행합니다. React는 `unstable_batchedUpdates`가 동작하는 동안 대기 중이던 모든 상태 업데이트를 추적하고 단일 렌더링 패스에 모두 적용합니다. 이벤트 핸들러 입장에서 보면 이는 꽤 잘 먹히는 방법입니다. 왜냐하면 React는 이미 주어진 이벤트에 대해 어떤 핸들러를 호출해야하는지를 정확하게 알고 있기 때문입니다.

개념적으로, React가 내부적으로 어떻게 동작하는지를 다음과 같은 의사 코드로 그려볼 수 있습니다.

```ts
// 의사 코드는 실제로 동작하지는 않고 아이디어만 던져 줍니다.
function internalHandleEvent(e) {
  const userProvidedEventHandler = findEventHandler(e)

  let batchedUpdates = []

  unstable_batchedUpdates(() => {
    // 여기서 대기중이던 업데이트는 모두 batchedUpdates로 추가될 것입니다.
    userProvidedEventHandler(e)
  })

  renderWithQueuedStateUpdates(batchedUpdates)
}
```

하지만 이는 `실제 즉시 호출 스택(the actual immediate call stack) 밖에서 대기중이던 상태 업데이트는 함께 처리되지 않는다는 것을 뜻합니다.`

예를 들어보겠습니다.

```ts
const [counter, setCounter] = useState(0)

const onClick = async () => {
  setCounter(0)
  setCounter(1)

  const data = await fetchSomeData()

  setCounter(2)
  setCounter(3)
}
```

이 예시에서는 3개의 렌더링 패스가 실행될 것 입니다. 첫번째 패스는 `setCounter(0)`와 `setCounter(1)`을 일괄 처리할 것 입니다. 왜냐하면 둘 다 원래의 이벤트 핸들러 호출 스택이 진행되는 동안 실행되기 때문입니다. 따라서 둘 다 `unstable_batchedUpdates()` 호출 안에서 실행될 것 입니다.

하지만, `setCounter(2)`의 호출은 await 이후에 발생합니다. 이는 원래의 동기적 호출 스택이 완료되고, 이 함수의 후반부는 완전히 다른 이벤트 루프 호출 스택에서 훨씬 나중에 실행된다는 것을 의미합니다. 이 때문에, React는 `setCounter(2)` 호출의 마지막 단계로써 전체 렌더링 패스를 동기적으로 실행할 것이고, 렌더링 패스를 마친 후 `setCounter(2)`로 부터 반환할 것 입니다.

그리고나서 같은 작업이 `setCounter(3)`에서도 발생할 것 입니다. 왜냐하면 `setCounter(3)` 역시도 원래의 이벤트 핸들러 밖에서 실행되고 일괄 처리되지 않기 때문입니다.

커밋 단계 생명주기 메소드인 `componentDitMount`, `componentDitUpdate`그리고 `useLayoutEffect`안에서도 추가적인 엣지 케이스가 존재합니다. 일반적인 유즈 케이스는 다음과 같습니다.

- 불완전한 일부 데이터로 컴포넌트를 최초로 렌더링할 때
- 커밋 단계의 라이프 사이클에서, 페이지에 있는 DOM 노드의 실제 사이즈를 측정하기 위해 참조를 사용할 때
- 측정 결과에 기반하여 컴포넌트에 상태를 세팅할 때
- 데이터가 업데이트되어 즉각적으로 리렌더링할 때

이 유즈 케이스에서, 일부만 렌더링된 최초의 UI는 사용자에게 보여질 필요가 없습니다. 최종적으로 전부 렌더링된 UI만 보여지면 됩니다. 브라우저는 수정 중인 DOM 구조를 다시 계산하겠지만, JS 스크립트가 여전히 동작중이고 이벤트 루프를 막는 동안에는 화면에 아무것도 페인팅하지 않을 것 입니다. 따라서 div.innerHTML = "a"; div.innerHTML = "b"; 같은 DOM 수정을 여러 번 실행할 수 있으며 "a"는 절대로 나타나지 않을 것입니다.

이러한 이유로 인해, React는 커밋 단계의 라이프 사이클에서 항상 렌더링을 동기적으로 실행할 것입니다. 따라서 "최종"적으로 렌더링 된 컨텐츠만이 화면에 보이게 될 것입니다.

최종적으로, 제가 아는 선에서는, `useEffect` 콜백 함수를 활용한 상태 업데이트는 대기 중으로 저장되고, `useEffect` 콜백 함수가 완료되는 시점에 "수동적 효과(Passive Effect)"의 마지막 부분에서 실행됩니다.

`unstable_batchedUpdates` API가 공개적으로 export되었다는 점에 주목할 필요가 있습니다. 하지만,

- 이름에서 알 수 있듯이, "unstable"이 붙어있고 공식적인 React API로써 지원되지는 않습니다.
- 반대로, React 팀은 "불안정한 API 중에서 가장 안정된 API이며, Facebook을 구성하는 코드 중 절반이 이 함수에 의존하고 있다"고 밝혔습니다.
- react 패키지로 export 되어 있는 다른 코어 React API와는 다르게, unstable_batchedUpdates은 재조정에 특화된 API (a reconciler-specific API)이며 react 패키지에는 포함되어있지 않습니다. 대신에 react-dom과 react-native에 의해 export 되어 있습니다. 즉, react-three-fiber 또는 ink같은 다른 재조정자 (reconcilers)는 unstable_batchedUpdates 함수를 export하지 않을 가능성이 큽니다

React-Redux v7에서는 `unstable_batchedUpdates`를 내부적으로 사용하기 시작했고, ReactDOM과 React-Native와 같이 동작하기 위해 조금 변칙적인 빌드셋업이 필요합니다. 다가오는 React의 Concurrent Mode에서 React는 언제 어디서든지 업데이트를 항상 배치로 진행할 것 입니다.

### 렌더 동닥의 엣지 케이스

> React는 개발 과정에서 \<StrictMode> 태그 안에 있는 컴포넌트를 이중으로 렌더링 할 것입니다. 이는 여러분의 렌더링 로직이 실행되는 횟수와 렌더 패스가 커밋된 횟수가 정확히 일치하지 않는다는 것을 의미하며, console.log()를 사용하여 렌더링 횟수를 파악하는 행위가 항상 정확하지는 않다는 것을 의미합니다. 대신에, React DevTools를 사용하여 전체적으로 커밋 된 렌더링 횟수를 파악하거나, useEffect 훅 또는 componentDidMount/Update 라이플사이클 메소드를 사용하여 기록 (log)을 남기는 것이 더 좋습니다. 이 방법을 사용할 경우 React가 렌더 패스를 완료하고 커밋까지 했을 때에만 기록 (log)이 남을 것입니다.

일반적인 상황에서 여러분이 절대 해서는 안되는 일은 바로 실제 렌더링 로직이 실행되는 동안에 상태 업데이트를 대기열에 넣는 행위입니다. 다시 말해서, `setSomeState()` 콜백 함수를 클릭 이벤트에 설정하는 것은 괜찮지만, 이 `setSomeState()`를 실제 렌더링 동작의 일부로써 호출하면 안된다는 것입니다.

하지만 여기에는 한가지 예외 사항이 있습니다. 함수형 컴포넌트는 조건에 따라서 `setSomeState()`를 렌더링 도중에 바로 호출할 수 있습니다. 하지만 매번 컴포넌트가 렌더링 될 때마다 실행시키는 것은 아닙니다. 이 동작은 클래스형 컴포넌트의 [getDerivedStateFromProps](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops)와 비슷합니다. 만약 함수형 컴포넌트가 렌더링 도중에 상태 업데이트를 대기열에 넣는다면, React는 그 즉시 이 상태 업데이트를 적용할 것이며 동기적으로 곧바로 해당 컴포넌트를 다시 렌더링 할 것입니다. 그 후 다음 단계로 넘어갈 것입니다. 만약 컴포넌트가 끊임 없이 상태 업데이트를 대기열에 넣으면서 React의 리렌더링을 발생시킨다면, React는 계속 이 동작을 반복하다가 특정 횟수가 넘으면 멈추고 에러를 발생시킬 것입니다. (이 횟수는 현재 50회로 지정되어 있습니다.) 이 기법은 Prop의 변화에 기반한 state 값 업데이트를 리렌더링 + `useEffect` 내부에서의 `setSomeState()` 호출과 같은 동작 없이 즉각적으로 발생시키는 일에 사용될 수 있습니다.

## 렌더 성능 향상시키기

비록 렌더링이 React의 기본적인 동작이지만 때때로는 낭비가 될 수 있습니다. 만약 컴포넌트의 렌더링 결과물이 그 전과 아무런 차이가 없고, DOM 상에서도 굳이 업데이트할 필요가 없다면, 렌더링 과정은 사실상 낭비에 불과할 것 입니다.

React 컴포넌트의 렌더링 결과물은 항상 현재의 Props와 State에 기반해야만 합니다. 따라서, 만약 우리가 Props와 State가 변경되지 않았다는 것을 미리 알고 있다면, 우리는 렌더링 결과물이 이전과 똑같을 것이고, 이 컴포넌트에서는 그 어떤 변화도 필요가 없으며, 따라서 우리는 이 렌더링 작업을 안전하게 건너뛸 수 있다는 사실도 인식해야 합니다.

통상적으로 소프트웨어의 성능을 향상시키기 위한 기본적은 접근법은 2개 정도가 있습니다.

1. 똑같은 작업은 빠르게 수행하자.
2. 애초에 작업을 적게 하자.

주로 React에서 렌더링 최적화를 한다면, 컴포넌트 렌더링을 가능하다면 스킵해서 작업을 덜 하도록 할 수 있습니다.

### 컴포넌트 렌더 최적화 기법

React는 컴포넌트 렌더링을 생략할 수 있도록 3개의 API를 제공합니다.

- [React.Component.shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate): Optional 클래스 컴포넌트의 라이프사이클 메서드로써, 렌더링 과정 초반에 호출됩니다. 만약 false를 반환하면 React는 컴포넌트 렌더링을 건너뛸 것 입니다. 여러분은 커스텀 로직을 이 메서드안에 작성해서 true를 반환할지 false를 반환할지를 판단할 수 있습니다. 하지만 가장 일반적인 로직은 컴포넌트의 props와 state가 마지막에 변경되었는지 여부를 검사하고, 변경되지 않았다면 false를 반환하는 것입니다.
- [React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent): shouldComponentUpdate의 가장 일반적인 사용법은 props와 state의 업데이트 여부를 검사하는 것 입니다. PureComponent의 클래스 컴포넌트는 shouldComponentUpdate 메소드를 기본적으로 탑재하고 있습니다. 따라서 Component + shouldComponentUpdate와 동일하다고 볼 수 있습니다.
- [React.memo()](https://reactjs.org/docs/react-api.html#reactmemo): React에 내장된 ["고차 컴포넌트(Higher Order Component)"](https://reactjs.org/docs/higher-order-components.html)입니다. 여러분의 컴포넌트 타입을 인수로 받고, 새로운 Wrapper 컴포넌트를 반환합니다. Wrapper 컴포넌트의 기본 동작은 Props가 변경되었는지 여부를 체크하는 것 입니다. 만약 변경되지 않았다면 리렌더링을 막습니다. 함수형 컴포넌트와 클래스형 컴포넌트 모두 React.memo()로 Wrapping 될 수 있습니다. (여러분이 작성한 임의의 비교 콜백 로직이 전달될 수 있지만, 어쨌든 기존의 Props와 새로운 Props를 비교하는 일만 사실상 가능합니다. 따라서 여러분의 커스텀 비교 콜백 로직은 아마도 특정 Props만을 비교하는 정도로만 사용될 것입니다.)

상위의 방법들은 모두 "얕은 비교 (Shallow Equality)"라고 불리는 비교 방법을 사용합니다. 얕은 비교란 서로 다른 2개 객체를 각각 모두 조사해서 내용물 중에 차이가 있는지 여부를 검사하는 것 입니다. 예를 들면, `obj1.a === obj2.a && obj1.b === obj2.b && ........` 이런 식입니다. 얕은 비교는 대표적으로 빠른 작업입니다. 왜냐하면 === (일치) 비교는 JavaScript 엔진 입장에서 매우 심플한 동작이기 때문입니다. 따라서, 상위의 3가지 방법들은 `const shouldRender = !shallowEqual(newProps, prevProps)`와 같은 방법론을 사용한다고 볼 수 있습니다.

또한 조금 덜 알려진 기법도 있습니다. **만약 React 컴포넌트가 반환한 렌더링 결과물이 그 전에 반환했던 결과물과 정확히 동일한 엘리먼트 참조를 반환한다면, React는 그 부분의 자식 요소 리렌더링을 건너뛸 것입니다.** 이 기법을 구현하고 싶다면 적어도 2가지 방법을 시도할 수 있습니다.

- 만약 여러분의 렌더링 결과물 안에 `props.children`이 들어있다면, 컴포넌트가 state를 업데이트해도 엘리먼트는 똑같을 것 입니다.
- useMemo()로 감싸진 엘리먼트는 의존성 값이 변경되기 전까지는 항상 동일할 것 입니다.

```ts
// The `props.children` content won't re-render if we update state

function SomeProvider({ children }) {
  const [counter, setCounter] = useState(0)

  return (
    <div>
      <button onClick={() => setCounter(counter + 1)}>Count: {counter}</button>
      <OtherChildComponent />
      {children}
    </div>
  )
}

function OptimizedParent() {
  const [counter1, setCounter1] = useState(0)
  const [counter2, setCounter2] = useState(0)

  const memoizedElement = useMemo(() => {
    // This element stays the same reference if counter 2 is updated,
    // so it won't re-render unless counter 1 changes
    return <ExpensiveChildComponent />
  }, [counter1])

  return (
    <div>
      <button onClick={() => setCounter1(counter1 + 1)}>
        Counter 1: {counter1}
      </button>
      <button onClick={() => setCounter1(counter2 + 1)}>
        Counter 2: {counter2}
      </button>
      {memoizedElement}
    </div>
  )
}
```

이 모든 기법들에 대해서, 컴포넌트 렌더링을 건너뛴다는 것은 React가 그 하위 트리 전체를 렌더링하는 것 역시도 건너뛴다는 것을 의미합니다. 왜냐하면 이는 "재귀적으로 자식 요소를 렌더링하는 기본 작업"을 정지시키는 것이기 때문입니다.

### 새 Props 참조가 렌더 최적화에 미치는 영향

우리는 앞에서 기본적으로, React는 Props가 변경되지 않았는데도 모든 중첩 컴포넌트들을 리렌더링한다는 것을 확인했습니다. 이는 또한 자식 컴포넌트에게 새로운 참조를 Props로 전달하는 일이 의미가 없다는 것을 의미합니다. 왜냐하면 전달 여부에 상관없이 렌더링이 발생할 것이기 때문이죠.

다음의 예시를 살펴보겠습니다.

```ts
function ParentComponent() {
  const onClick = () => {
    console.log("Button clicked")
  }

  const data = { a: 1, b: 2 }

  return <NormalChildComponent onClick={onClick} data={data} />
}
```

ParentComponent가 렌더링 될 때마다, 새로운 onClick 함수의 참조와 새로운 data 객체의 참조가 생성되어서 `NormalChildComponent`에게 Props로 전달될 것 입니다. (onClick 이벤트를 정의할 때에는 function 키워드나 화살표 함수나 별 차이가 없다는 점을 기억합시다. - 어쨌든 둘 다 새로운 함수의 참조이니까요.)

이는 또한 "호스트 컴포넌트 (Host Components)"에 대한 렌더링 최적화가 별 의미가 없다는 점도 의미합니다. 예를 들면 \<div>나 \<button> 태그를 React.memo()로 감싸는 것처럼 말이죠. 이러한 기본 컴포넌트는 자식 컴포넌트를 갖고 있지 않습니다. 따라서 렌더링 프로세스가 더이상 진행되지 않겠죠.

하지만, 만약 자식 컴포넌트에서 Props의 변경 여부를 검사하여 렌더링 최적화를 시도한다면, 새로운 참조값을 Props로 전달하는 것은 자식 컴포넌트에서 리렌더링을 발생시킬 것 입니다. 만약 새로운 Props 참조값이 완전히 새로운 데이터라면 문제 없습니다. 하지만 만약 부모 컴포넌트가 콜백 함수를 Props로 내려주는 경우는 어떨까요?

```ts
const MemoizedChildComponent = React.memo(ChildComponent)

function ParentComponent() {
  const onClick = () => {
    console.log("Button clicked")
  }

  const data = { a: 1, b: 2 }

  return <MemoizedChildComponent onClick={onClick} data={data} />
}
```

`ParentComponet`가 렌더링될 때마다 이 새로운 참조값들은 `MemoizedChildComponent`로 하여금 Props 값이 새로운 참조값으로 변경되었는지 여부를 체크하도록 할 것 입니다. 그리고 리렌더링이 발생하겠죠. onClick 함수와 data 객체는 전혀 변경되지 않았는데도 말입니다.

정리해보면 다음과 같습니다.

- 우리는 리렌더링을 최대한 막으려고 했지만 MemoizedChildComponent는 항상 리렌더링이 발생할 것 입니다.
- MemoizedChildComponent가 기존의 Props와 새로운 Props를 비교하는 행위는 굳이 하지 않아도 되는 낭비 행위 입니다.

다른 비슷한 케이스도 있습니다. \<MemoizedChild>\<OtherComponent />\</MemoizedChild> 를 렌더링하는 것 역시 자식 요소에서 리렌더링이 발생합니다. 왜냐하면 `props.children`는 항상 새로운 참조값이기 때문입니다.

### Props 참조 최적화

클래스형 컴포넌트는 새로운 콜백 함수 참조를 실수로 생성하는 일을 걱정할 필요가 없습니다. 왜냐하면 클래스형 컴포넌트는 항상 동일한 참조 값을 갖고 있는 인스턴스 메소드를 가질 수 있기 때문입니다. 하지만 여러 개의 자식들에게 각각 다른 고유한 콜백함수를 생성해서 적용할 필요가 있거나 익명 함수에서 값을 가져와서 자식에게 넘겨줄 필요가 있을 수 있습니다. 이 경우는 새로운 참조가 생성될 것 입니다. 따라서 렌더링 과정에서 새로운 객체가 생성되어 자식에게 Props로 사용될 것 입니다. React는 이러한 케이스를 최적화하기 위한 수단을 갖고 있지 않습니다.

함수형 컴포넌트의 경우, React는 같은 참조를 재사용하기 위한 Hook 두개를 제공합니다.(메모이제이션)

- useMemo: 객체 생성 또는 복잡한 연산과 관련된 일반적인 데이터를 다루는 경우
- useCallback: 새로운 콜백 함수를 생성하는 경우

### 전부 다 메모이제이션 해야할까?

위에서 언급한 것처럼 여러분은 Prop으로 사용하는 모든 함수와 객체를 useMemo()와 useCallback으로 감쌀 필요는 없습니다. 자식에서 변화를 일으키는 친구들에게만 사용하면 됩니다. (즉, useEffect에서 하는 종속성 배열 비교 작업은 일관된 Props 참조가 필요한 자식 요소에서 사용될 수 있습니다. 하지만 로직이 더 복잡해지죠.)

또 항상 드는 고민은 "React는 왜 기본적으로 React.memo로 모든걸 감싸지 않는걸까?"입니다.

Dan Abramov는 [메모이제이션은 Props를 비교하는 비용이 발생한다는 점](https://twitter.com/dan_abramov/status/1095661142477811717)을 계속해서 지적해왔습니다. 그리고 메모이제이션이 리렌더링을 절대로 방지할 수 없는 경우도 많이 있습니다. 컴포넌트가 항상 새로운 Props를 받는 경우 처럼 말이죠. Dan이 작성한 트위터를 예로 보시죠.

모든 컴포넌트를 메모이제이션 하면 에러가 발생할 수 있습니다. 불변성을 지키지 않고 데이터를 수정하는 경우 문제가 생길 수 있죠.

저는 이점에 대해서 Dan과 트위터로 토의를 했었습니다. 저는 개인적으로 `React.memo()`를 광범위하게 사용하는 것이 전반적인 앱 렌더링 성능에 더 이득이라고 생각합니다.

### 불변성과 리렌더링

`React에서 상태 업데이트는 항상 불변성을 지켜야합니다.` 그 이유는 크게 다음과 같습니다.

- 여러분이 어떤 것을 수정하고 또 어디서 수정하느냐에 따라서, 컴포넌트가 여러분의 의도대로 렌더링되지 않을 수 있습니다.
- 데이터가 언제 그리고 왜 업데이트 되었는지 파악하기 어려울 수 있습니다.

더 구체적인 예시를 같이 보시죠.

저희가 봐왔듯이, `React.memo / PureComponent / shouldComponentUpdate` 모두 현재 Props와 이전 Props의 얕은 비교에 기반하고 있습니다. 따라서 `props.someValue !== prevProps.someValue` 이라면 Prop이 새로운 값임을 알 수 있겠죠.

만약 여러분이 데이터를 직접적으로 수정한다면, `someValue`는 같은 참조값입니다. 따라서 컴포넌트는 아무 일도 일어나지 않았다고 판단하겠죠. 저희는 지금까지 불필요한 렌더링을 피해서 성능 최적화를 노리고 있었다는 점에 주목할 필요가 있습니다. "불필요한" 또는 "낭비된" 렌더링은 결국 Props가 변경되지 않았는데 발생하는 렌더링을 의미합니다. 만약 여러분이 데이터를 직접적으로 수정한다면, 컴포넌트는 아무 일도 일어나지 않았다고 잘못 판단할 것이고, 여러분은 컴포넌트가 왜 리렌더링이 안되지? 라며 혼란스러워 하실 것 입니다.

또 다른 이슈는 `useState`와 `useReducer`훅과 관련되어있습니다. 제가 `setCounter()` 또는 `dispatch()`를 호출하는 매 순간, React는 리렌더링을 대기열에 넣을 것 입니다. 하지만 React에서 훅을 통해 상태를 업데이트하려면 새로운 상태로 새로운 참조값을 받거나 반환 해야합니다. 새로운 참조값은 객체, 배열 또는, String, Number등등의 원시 타입도 되겠죠.

React는 렌더링 단계에서 모든 상태 업데이트를 적용합니다. React가 훅을 통한 상태 업데이트를 반영하려고 한다면, 새로운 상태 값이 기존 값과 참조값이 같은지를 확인합니다. React는 업데이트 대기열에 들어가있는 컴포넌트는 항상 렌더링할 것 입니다. 하지만, 상태 값이 기존 값과 똑같은 참조라면, 또 렌더링을 지속할 이유가 없다면 (부모 컴포넌트가 렌더링되었거나 등), React는 컴포넌트 렌더링 결과물을 폐기할 것이고 렌더 패스에서 완전히 빠져나올 것입니다. 따라서 제가 만약 어떤 배열을 다음과 같이 수정한다면:

```js
const [todos, setTodos] = useState(someTodosArray)

const onClick = () => {
  todos[3].completed = true
  setTodos(todos)
}
```

컴포넌트 리렌더링은 실패할 것 입니다.  
기술적으로 보면, 가장 바깥 쪽 참조만 불변성을 지키면서 업데이트 해야합니다. 예시를 다시 수정해보겠습니다.

```js
const onClick = () => {
  const newTodos = todos.slice()
  newTodos[3].completed = true
  setTodos(newTodos)
}
```

이제 저희는 새로운 배열 참조를 생성해서 전달했습니다. 그리고 컴포넌트는 리렌더링 될 것입니다. 여기서 주목할 점은 클래스형 컴포넌트의 `this.setState()`와 함수형 컴포넌트의 `useState` 및 `useReducer` 훅은 수정과 리렌더링 측면에서 동작 방식이 다르다는 것입니다. `this.setState()`는 여러분이 전체를 수정했는지 여부를 신경쓰지 않습니다. 대신에 항상 리렌더링이 발생합니다. 따라서 `this.setState()`는 다음과 같은 경우도 리렌더링이 발생할 것입니다.

```js
const { todos } = this.state
todos[3].completed = true
this.setState({ todos })
```

그리고 사실, `this.setState({})` 처럼 빈 객체를 전달해도 리렌더링 될 것입니다.

모든 실제 렌더링 동작 측면에서 보면, 데이터 수정(mutation)은 React의 표준적인 단방향 데이터 흐름에 혼란을 야기합니다. 데이터 수정은 다른 코드에게 다른 값이 보여지도록 할 수 있습니다. 전혀 원했던 바가 아닐지라도 말이죠. 이러한 현상은 주어진 상태가 언제 그리고 왜 업데이트되어야 하는지, 그리고 어디서 왔는지 등을 파악하기 어렵게 만듭니다.

정리하자면, **React와 React의 에코 시스템은, 불변성을 지키는 업데이트를 간주합니다. 여러분이 불변성을 지키지 않고 데이터를 수정하면 에러를 발생시킬 수 있습니다. 그러니까 하지 마세요.**

### React 컴포넌트 렌더링 성능 측정하기

React는 dev 빌드에서 더 느리게 동작합니다. 애플리케이션을 개발 모드로 프로파일링해서 어떤 컴포넌트가 렌더링되는지와 그 이유를 확인할 수 있습니다. 또한 렌더링하는데 필요한 상대적 시간을 컴포넌트끼리 서로 비교할 수도 있습니다. ("컴포넌트 B는 지금 커밋 단계에서 렌더링하려면 컴포넌트 A보다 3배 더 오래걸리네"처럼 비교할 수 있습니다.) 하지만 React dev 빌드로 절대적인 렌더링 횟수를 측정하려고는 하지 마세요. 절대적인 횟수는 production 빌드에서만 사용해야 합니다.여러분은 Prod-lie 빌드에서 타이밍 데이터를 캡처하기 위해서 프로파일러를 이용하려면, React의 특별한 [프로파일링 빌드](https://kentcdodds.com/blog/profile-a-react-app-for-performance)를 사용하셔야 합니다.

## Context와 렌더링 동작

**React의 Context API는 컴포넌트의 서브 트리에서 단일 사용자 제공 값(a single user-privided value)을 사용 가능하도록 할 수 있는 메커니즘입니다.** \<MyContext.Provider> 안에 들어있는 모든 컴포넌트는 Context 인스턴스로부터 오는 값을 Props Drilling 없이 바로 읽을 수 있습니다.

**Context는 상태 관리 도구가 아닙니다.** Context 안으로 들어오는 값들은 여러분이 스스로 관리하셔야 합니다. 이 작업은 React 컴포넌트 State 안에 데이터를 유지하고, 이 데이터를 기반으로 Context 값을 생성하는 방식으로 보통 이뤄집니다.

### Context 기본

Context provider는 \<MyContext.Provider value={42}> 처럼 단일 값 (single value) prop을 받습니다. 자식 컴포넌트는 Context Consumer 컴포넌트를 렌더링하고 렌더 Prop을 제공하면서 Context를 소비 (Consume) 합니다. 다음과 같이 말이죠.

```ts
<MyContext.Consumer>{(value) => <div>{value}</div>}</MyContext.Consumer>
```

또는 함수형 컴포넌트에서 useContext를 호출할 수도 있습니다.

```ts
const value = useContext(MyContext)
```

### Context 값 업데이트하기

React는 주변 컴포넌트가 Provider를 렌더링했을 때 Context Provider에 새로운 값이 전달되었는지 여부를 체크합니다. 만약 Provider의 값이 새로운 참조값이라면 React는 그 값이 이번에 새로 변경된 값이라는 것과 그 Context를 소비하는 컴포넌트가 업데이트 되어야 한다는 것을 알아차리게 됩니다.

**새로운 객체를 Context Provider에 제공하는 것은 업데이트를 발생시킬 수 있다는 점 을 기억하세요.**

```ts
function GrandchildComponent() {
  const value = useContext(MyContext)
  return <div>{value.a}</div>
}

function ChildComponent() {
  return <GrandchildComponent />
}

function ParentComponent() {
  const [a, setA] = useState(0)
  const [b, setB] = useState("text")

  const contextValue = { a, b }

  return (
    <MyContext.Provider value={contextValue}>
      <ChildComponent />
    </MyContext.Provider>
  )
}
```

이 예시에서는, `ParentComponent`가 렌더링 될 때마다, React는 `MyContext.Provider`가 새로운 값을 받았다는 것을 알아차릴 것이고, 서브 트리를 하나씩 돌면서 `MyContext`를 소비하는 컴포넌트를 찾을 것 입니다.

**Context Provider가 새로운 값을 가질 때면, Context를 사용하는 모든 중첩된 컴포넌트는 리렌더링이 될 것입니다.**

React의 관점에서 보면, 각 Context Provider는 오직 한 개의 값만 갖고 있습니다. 객체, 배열 또는 원시 타입이든 상관 없이 오직 하나의 Context 값만 갖고 있습니다. 현재까지는 Context를 소비하는 컴포넌트 입장에서 새로운 Context 값으로 인한 업데이트를 건너뛸 방법은 없습니다. 새로운 Context 값의 일부만 변경되어도 말이죠.

### 상태 업데이트, Context 그리고 리렌더링

이제 그동안 배웠던 것들을 하나로 합쳐봅시다.

- setState()를 호출하면 컴포넌트 렌더링이 대기열에 들어갑니다.
- React는 기본적으로 중첩 컴포넌트를 재귀적으로 렌더링합니다.
- Context Provider는 컴포넌트가 렌더링하는 값들을 전달 받습니다.
- 이 값들은 보통 부모 컴포넌트의 state로부터 내려옵니다.

이는 즉, **기본적으로 Context Provider를 렌더링하는 부모 컴포넌트 State 업데이트는 그 하위 컴포넌트들을 모두 리렌더링 시킵니다. 그 하위 컴포넌트들이 Context 값을 사용하는지 여부와 관계 없이 말이죠!**

위에 있는 Parent/Child/Grandchild 예시를 다시 살펴보면, GrandchildComponent 컴포넌트는 리렌더링 될 것이지만, Context의 업데이트 때문은 아니라는 것을 알 수 있습니다. 그저 ChildComponent가 렌더링되었기 때문에 리렌더링 된 것일 뿐이죠! 이 예시에서는, "불필요한" 리렌더링을 최적화할 수 있는 방법은 없습니다. 따라서 React는 ParentComponent가 렌더링되면 ChildComponent와 GrandchildComponent를 기본적으로 렌더링합니다. 만약 부모가 MyContext.Provider 안에 새로운 Context 값을 넣으면, GrandchildComponent는 그 값을 사용하기 때문에 리렌더링 될 것입니다. 하지만 이는 Context 업데이트 때문이 아니라 상위 컴포넌트가 리렌더링 되었기 때문입니다. 어쨌든 원인만 다를 뿐 리렌더링이 되긴 합니다.

### Context 업데이트와 렌더 최적화

최적화를 위해 예시를 좀 수정해봅시다. 하지만 `GreatGrandchildComponent`를 맨 밑에 추가해서 변화를 줘보겠습니다.

```ts
function GreatGrandchildComponent() {
  return <div>Hi</div>
}

function GrandchildComponent() {
    const value = useContext(MyContext);
    return (
      <div>
        {value.a}
        <GreatGrandchildComponent />
      </div>
}

function ChildComponent() {
    return <GrandchildComponent />
}

const MemoizedChildComponent = React.memo(ChildComponent);

function ParentComponent() {
    const [a, setA] = useState(0);
    const [b, setB] = useState("text");

    const contextValue = {a, b};

    return (
      <MyContext.Provider value={contextValue}>
        <MemoizedChildComponent />
      </MyContext.Provider>
    )
}
```

이제 여기서 setA(42)를 호출한다면

- ParentComponet가 렌더링 될 것입니다.
- 새로운 contextValue 참조가 생성될 것입니다.
- React는 `MyContext.Provider`가 새로운 context 값을 갖게 되었고, `MyContext`를 사용하는 소비자 컴포넌트들이 렌더링 되어야 한다는 것을 알아차립니다.
- React는 `MemoizedChildComponent`를 렌더링하려고 시도할 것이지만, `React.memo()`로 감싸져있는 것을 발견합니다. Props를 전달받고 있지 않기 때문에 실제로 변경되는 Props는 없습니다.따라서 React는 ChildComponent 렌더링을 건너뜁니다.
- 하지만, `MyContext.Provider`에 업데이트가 있었습니다. 따라서 이 업데이트를 알아차려야 할 컴포넌트가 밑에 있을 수 있습니다.
- React는 밑으로 더 내려가서 `GrandchildComponent`에 도달합니다. 그리고 `GrandchildComponent`가 `MyContext`를 읽고 있는 것을 발견합니다. 따라서 새로운 Context값이 있기 때문에 리렌더링이 필요하다는 것도 알아챕니다. React는 더 진행해서 `GrandchildComponent`를 리렌더링합니다. Context의 변경으로 인한 리렌더링이죠.
- `GrandchildComponent`가 렌더링되었기 때문에, React는 그 안에 들어있는 모든 것을 렌더링할 것 입니다. 따라서 React는 `GreatGrandchildComponent`도 리렌더링할 것 입니다.

다시 말하면, [Sophie Alpert가 말했듯이](https://twitter.com/sophiebits/status/1228942768543686656),

> 여러분의 Context Provider 바로 밑에 있는 React 컴포넌트는 React.memo를 사용해야합니다.

그렇게 하면, 부모 컴포넌트의 State 업데이트는 모든 컴포넌트를 리렌더링하도록 강요 하지 않고, Context를 읽고 있는 부분만 리렌더링 할 것입니다. (ParentComponent가 \<MyContext.Provider>{props.children}\</MyContext.Provider>를 렌더링하는 것과 기본적으로 똑같은 결과물을 얻을 것입니다. 이는 "똑같은 엘리먼트 참조" 기술을 한 단계 업그레이드하여 자식 컴포넌트의 리렌더링을 방지하고, \<ParentComponent>\<ChildComponent />\</ParentComponent>를 한 단계 위로 렌더링합니다.)

하지만 GrandchildComponent가 다음 Context 값을 기반으로 렌더링되면, React는 재귀적으로 모든 것을 리렌더링하는 기본적인 동작으로 복귀할 것임을 알아야합니다. 따라서, GreatGrandchildComponent 가 렌더링 될 것이고, 그 하위에 있는 모든 것들도 렌더링 될 것입니다.

## React-Redux와 렌더링 동작

제가 현재 React 커뮤니티에서 제일 많이 목격한 논쟁은 "Context VS Redux"입니다. 즉, 이 논쟁에서 가장 많이 언급되는 주장 중 하나가 "React-Redux는 렌더링이 필요한 컴포넌트만 골라서 리렌더링하기 때문에 Context 보다 더 성능이 좋다."입니다.

이 말은 어떤 면에서는 맞니만, 진짜 정답은 훨씬 더 복잡 미묘합니다.

### React-Redux 구독

저는 정말 많은 사람들이 React-Redux는 내부에서 Context를 사용한다라는 말을 반복하는 것을 봤습니다. 기술적으로 보면 맞습니다. 하지만 [React-Redux는 Redux 스토어 인스턴스를 전달하기 위해 Context를 사용하는 것이지, 현재의 State 값을 위해 사용하는 것이 아닙니다.](https://blog.isquaredsoftware.com/2020/01/blogged-answers-react-redux-and-context-behavior/) 이는 즉, 우리는 시간이 지남에 따라서 항상 똑같은 Context 값을 \<ReactReduxContext.Provider>로 전달하고 있다는 의미가 됩니다. Redux 스토어는 액션이 디스패치되면 구독자 알림 콜백(Subscriber Notification Callbacks)을 실행한다는 점을 기억하세요. [Redux를 사용할 필요가 있는 UI 레이어는 항상 Redux 스토어를 구독하고, 구독자 콜백(Subscriber Callbacks)으로부터 가장 최신의 State를 읽어온 후, 값을 비교하고, 관련된 데이터가 변경되었으면 리렌더링을 발생시킵니디.](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/) 구독자 콜백 프로세스는 React의 완전 바깥 부분에서 발생합니다. 그리고 React는 React-Redux가 특정 React 컴포넌트에 필요한 데이터가 변경되었음을 알고 있는 경우에만 이 과정에 참여합니다. (`mapState`, `useSelector`의 반환값에 기반해서 판단하죠.)

이는 Context와는 매우 다른 성능 특성으로 이어집니다. 맞습니다. 전체적으로는 더 적은 컴포넌트가 리렌더링 될 수 있습니다. 하지만 React-Redux는 스토어 State가 업데이트 될 때마다 모든 컴포넌트에서 항상 `mapState/useSelector`를 실행해야합니다. 대부분의 케이스에서 보면, 이 selectors를 실행시키는 비용이 React가 다른 렌더 패스를 실행시키는 비용보다 훨씬 더 저렴합니다. 따라서 평균적으로 보면 이득이죠. 하지만 해야되는 작업입니다. 하지만 selectors가 비용이 많이 드는 변형을 하거나 실수로 반환하지 말아야 할 값을 반환한다면, 전체 성능에 좋지 않을 것 입니다.

### connect와 useSelector의 차이

`connect`는 고차 컴포넌트입니다. 여러분이 컴포넌트를 집어넣으면 connect는 스토어를 구독하거나, mapState와 mapDispatch를 실행하거나, 여러가지가 합쳐진 Props를 내려주는 등의 역할을 하는 Wrapper 컴포넌트를 반환합니다.

connect Wrapper 컴포넌트는 항상 PureComponent/React.memo()와 동일하게 동작합니다. 하지만 약간 다른점이 있습니다. connect는 여러분의 컴포넌트가 전달 받는 합쳐진 Props가 변경되면, 이를 리렌더링하는 역할만 수행합니다. 일반적으로, 최종적으로 합쳐진 Props는 {...ownProps, ...stateProps, ...dispatchProps}의 결합체입니다. 따라서 부모로부터 온 새로운 Prop 참조는 결국 여러분의 컴포넌트를 렌더링 시킬 것입니다. PureComponent 또는 React.memo()와 같이 말이죠. 부모로부터 온 Props 말고도 mapState로부터 반환된 새로운 참조값 역시도 여러분의 컴포넌트를 렌더링 할 것입니다. (만약 여러분이 ownProps/stateProps/dispatchProps가 합쳐지는 방식을 수정할 수 있다면, 이 동작을 수정하는 것도 가능할 것입니다.)

`useSelector`는 반대로, 여러분의 함수형 컴포넌트 안에서 호출되는 훅입니다. 이로 인해서 `useSelector`는 부모 컴포넌트의 렌더링으로 인해 여러분의 컴포넌트가 리렌더링되는 것을 막을 수 없습니다!

이 점이 바로 [connect와 useSelector의 핵심적인 성능 차이](https://react-redux.js.org/api/hooks#performance)입니다. `connect`에서는, 모든 연결된 컴포넌트들은 `PureComponent`처럼 동작합니다. 따라서 전체적인 컴포넌트 트리를 타고 내려오는 React의 기본 렌더링 동작을 막는 방화벽처럼 동작합니다. 일반적인 React-Redux 애플리케이션에는 연결된 컴포넌트가 많기 때문에, 대부분의 리렌더링 전파는 컴포넌트 트리 상에서 상당히 작은 구역으로 제한됩니다. React-Redux는 데이터 변경에 기반해서 연결된 컴포넌트를 렌더링 시킵니다. 그 밑에 있는 2~3개의 컴포넌트 역시도 렌더링 될 것입니다.그리고 React는 업데이트할 필요가 없는 연결된 다른 컴포넌트가 실행되면 렌더링 전파를 중지합니다.

추가적으로 더 많이 연결된 컴포넌트를 갖는다는 것은, 각 컴포넌트는 아마도 스토어로부터 작은 데이터 조각을 읽는다는 것을 의미하고, 리덕스 액션으로 인해서 리렌더링이 될 가능성이 더 적어진다는 것을 의미합니다.

만약 여러분이 예외적으로 함수형 컴포넌트와 useSelector를 사용한다면, connect를 사용할 때보다 더 많은 컴포넌트들이 Redux-Store 업데이트로 인해서 리렌더링될 수 있습니다. 왜냐하면 연결된 컴포넌트가 없어서 컴포넌트 트리를 타고 전파되어 내려오는 렌더링을 막을 수 없기 때문이죠.

만약 이로 인해 성능에 악영향을 끼치지 않을지 염려되신다면, 답은 필요에 따라서 `React.memo`로 컴포넌트를 감싸서 부모 컴포넌트로 인해 발생된 불필요한 렌더링을 막아라 입니다.

## 요약

- React는 기본적으로 항상 재귀적으로 컴포넌트를 렌더링합니다. 따라서 부모가 렌더링되면 자식도 렌더링됩니다.
- 렌더링 그 자체는 괜찮습니다. 이는 React가 어떤 DOM 변화가 있는지 체크하는 절차입니다.
- 하지만, 렌더링 시간이 듭니다. 그리고 결과물이 변하지 않는 "낭비된 렌더링"이 발생할 수 있습니다.
- 콜백 함수 또는 객체와 같이 새로운 참조값을 내려주는 것은 항상 괜찮습니다.
- React.memo와 같은 API는 Props가 변경되지 않았다면 불필요한 렌더링을 건너뜁니다.
- 하지만 만약 여러분이 항상 새로운 참조값을 Props로 전달한다면, React.memo()는 절대로 리렌더링을 방지할 수 없기 때문에, 여러분은 이 값을 메모이제이션할 필요가 있을 것입니다.
- Context는 아무리 깊게 중첩되어있는 컴포넌트라도 Context값에 접근할 수 있도록 해줍니다.
- Context Providers는 Context 값이 변경되었는지를 참조를 통해 비교합니다.
- 새로운 Context 값은 모든 중첩된 Consumers를 리렌더링 시킵니다.
- 하지만 자식 컴포넌트는 어쨌든간에 부모 -> 자식의 렌더링 전파로 인해 리렌더링됩니다.
- 따라서 여러분은 아마도 Context Provider 안에 있는 자식 컴포넌트를 React.memo()로 감싸거나 {props.children}을 사용하고 싶을 수 있을것입니다. Context 값이 업데이트 될 때마다 전체 컴포넌트 트리가 리렌더링 되는 것을 방지하기 위해서죠.
- 자식 컴포넌트가 새로운 Context 값에 의해 렌더링되면, React는 그 밑으로 계속 렌더링을 전파시켜서 내려보냅니다.
- React-Redux는 Context를 통해 스토어 State를 전달하는 대신, Redux 스토어를 구독하여 업데이트를 체크하는 방식을 사용합니다.
- 이 구독은 Redux 스토어가 업데이트할 때마다 동작하므로, 최대한 빨라야합니다.
- `connect`는 `React.memo` 처럼 동작합니다. 따라서 많은 연결된 컴포넌트를 갖고 있으면 한번에 렌더링되는 컴포넌트의 갯수를 줄일 수 있습니다.
- useSelector는 훅입니다. 따라서 부모 컴포넌트로부터 오는 렌더링을 막을 수 없습니다. useSelector만 사용하는 애플리케이션은 React.memo를 적절히 사용해서 렌더링 전파를 막아주면 더 좋을 것 입니다.

모든 사람들이 Context는 도대체 언제 사용해야되고 Redux는 도대체 언제 사용해야하는건가요?라고 많이들 물어보시는 것 같아서, 경험에 의한 몇가지 표준 규칙을 요약해보겠습니다.

- Context를 사용해야하는 경우

  - 자주 변경되지 않는 간단한 값을 전달하고 싶은 경우
  - 애플리케이션의 여러 부분에서 사용해야할 State또는 함수가 있는데 Props Drilling으로 넘겨주고 싶지 않은 경우
  - 추가적인 라이브러리를 설치하지 않고 React에 내장된 기능만 사용하고 싶은 경우

- Redux를 사용해야하는 경우

  - 애플리케이션의 많은 부분에서 사용되어야할 대규모의 State를 갖고 있는 경우
  - 애플리케이션의 State가 자주 업데이트 되는 경우
  - State를 업데이트 하는 로직이 복잡한 경우
  - 애플리케이션의 코드 양이 중간 또는 클 경우, 그리고 많은 사람들이 사용하게 될 애플리케이션일 경우

이 규칙들은 어렵지도 않고 배타적인 규칙들도 아닙니다. 그저 이 도구들을 사용할 수 있을만한 상황을 제시해주는 가이드라인일 뿐입니다! 항상 그래왔듯이, 잠시 시간을 갖고 여러분이 현재 처해있는 상황과 제일 걸맞는 도구가 무엇인지 고민해보세요.

결론적으로, 사람들이 이 설명을 보고 다양한 상황에서 React의 렌더링 동작이 정확히 어떻게 이루어지는지에 대한 큰 그림을 얻어갈 수 있기를 바랍니다.
