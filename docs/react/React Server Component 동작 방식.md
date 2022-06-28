# React Server Component 동작 방식

## React Server Component는 무엇인가?

React Server Component(이하 RSC)를 사용하면, 서버와 클라이언트 (브라우저)가 리액트 애플리케이션을 서로 협력하여 렌더링할 수 있다. 이야기 하기에 앞서, 페이지를 렌더링하는 일반적인 리액트 컴포넌트 트리를 생각해보자. 이 트리에는 리액트 컴포넌트가 있고, 이 컴포넌트는 또 다른 리액트 컴포넌트를 렌더링한다. RSC를 이용하면 이 트리의 일부 컴포넌트는 서버에서 렌더링하거나, 일부 컴포넌트는 브라우저에서 렌더링하는 등 처리를 할 수 있다.

![image](https://user-images.githubusercontent.com/63354527/173498903-ec06b9c9-8a16-43d8-ad27-93cf96fe8dcf.png)

## 서버 사이드 렌더링이 아니다?

RSC는 정확히 말해서 서버 사이드 렌더링이 아니다. 물론 둘다 명칭에 서버가 포함되어있어서 혼란의 여지가 있다. RSC를 사용하면 SSR을 사용할 필요가 없고, 반대의 경우도 마찬가이다. SSR은 응답받은 트리를 raw html로 렌더링하기 위한 환경을 시뮬레이션 한다. 즉 , 서버와 클라이언트 컴포넌트를 별도로 구별하지 않고 동일한 방식으로 렌더링한다.

물론 SSR과 RSC를 함께 사용하여 서버 컴포넌트를 서버 쪽에서 렌더링하고, 브라우저에서는 적절하게 하이드레이션을 거치게 할 수 있다.

## 왜 필요할까?

RSC이전에는 모든 리액트 컴포넌트는 클라이언트 컴포넌트이며, 모두 브라우저에서 실행된다. 브라우저가 리액트 페이지를 방문하면, 필요한 모든 리액트 컴포넌트 코드를 다운로드하고, 리액트 컴포넌트 트리를 만든후 DOM에 렌더링한다. (SSR을 사용하면, DOM에 하이드레이션만 진행한다.) 브라우저는 이벤트 핸들러를 부착하고, 상태를 추적하고, 이벤트에 따른 응답 트리 변경 및 DOM의 효율적인 업데이트 등 리액트 애플리케이션이 인터렉션 할 수 있도록 처리할 수 있는 좋은 곳이다. 그런데 우리가 왜 서버에서 무언가를 렌더링하려고 하는 걸까?

브라우저 대신 서버에서 렌더링을 한다면 다음과 같은 장점을 얻을 수 있다.

- 서버는 데이터베이스, Graphql, 파일시스템등 데이터 원본에 직접 접근할 수 있다. 서버는 공용 api엔드 포인트를 거치지 않고 데이터를 직접 가져올 수 있고, 일반적으로 데이터 소스와 더 가깝게 배치되어 있으므로 브라우저보다 더 빠르게 데이터를 가져올 수 있다.
- 브라우저는 자바스크립트 번들링된 모든 코드를 다운로드 해야하는 것과 달리, 서버는 모든 의존성을 다운로드 할 필요가 없기 때문에(미리 다운로드 해놓고 수시로 재사용이 가능하므로) 무거운 코드 모듈을 저렴하게 사용할 수 있다.

즉 RSC를 활용하면 서버와 브라우저가 각자 잘 수행하는 작업을 처리할 수 있다. 서버 컴포넌트는 데이터를 가져오고 콘텐츠를 렌더링하는데 초점을 맞출수 있으며 페이지 로딩 속도가 빨라지고 자바스크립트 번들 크기가 작아져서 사용자의 환경이 향상될 수 있다.

## 개괄

이제 어떻게 작동하는지 살펴보자.

RSC는 작업 분담, 즉 서버가 할 수 있는 일을 먼저 처리하게 두고 나머지는 브라우저에게 넘겨준다.

일부 컴포넌트는 서버에서 렌더링되고, 일부 컴포넌트는 클라이언트에서 렌더링 되는 상황을 고려해보자. 서버는 일반적인 리액트 컴포넌트를 html의 p, div 태그등으로 렌더링한다. 그러나 브라우저에서 렌더링해야하는 클라이언트 컴포넌트가 나타나면, 클라이언트에서 여기를 렌더링하라는 의미에서 placeholder 같은 것을 둔다. 브라우저가 이 결과물을 받으면, 앞서 빈 곳으로 나왔던 부분을 채워 넣는다. 물론 실제로 이렇게 동작하지는 않지만 대략적인 그림으로 살펴보았다.

## 클라이언트와 서버 컴포넌트로 나누기

먼저 컴포넌트를 서버 컴포넌트인지, 클라이언트 컴포넌트인지 나눌 수 있을까?

리액트 팀에서는 이를 확장자로 구별하는 방식을 택했다. '.server.jsx' 면 서버 컴포넌트이고 client.jsx이면 클라이언트 컴포넌트이다. 만약 둘다 아니라면 이 컴포넌트는 서버나 클라이언트 양쪽 모두에서 사용 가능한 것이다.

이러한 정의는 굉장히 실용적인 방식으로 보인다. 개발자와 번들러 입장에서 구별하기 쉽다. 특히 번들러의 경우 파일이름을 검사하여 클라이언트 컴포넌트를 별도로 처리할 수 있다. 번들러는 RSC를 작동하는데 중요한 역할을 한다.

서버 컴포넌트는 서버에서 클라이언트 컴포넌트는 클라이언트에서 실행되므로 각 컴포넌트에서 수행할 수 있는 작업에는 제한이 있다. 여기에서 특히 기억해야할 것은, 클라이언트 컴포넌트가 서버 컴포넌트를 import할 수 없다는 점이다. 이는 서버 컴포넌트를 브라우저에서 실행할 수 없기 때문이다.

```js
// ClientComponent.client.jsx
// 안됨.
import ServerComponent from "./ServerComponent.server"
export default function ClientComponent() {
  return (
    <div>
      <ServerComponent />
    </div>
  )
}
```

클라이언트 컴포넌트가 서버 컴포넌트를 import할 수 없고, 그래서 서버 컴포넌트를 초기화 할 수 없다면 리액트 컴포넌트 트리를 어떻게 만들 수 있을까?

클라이언트 컴포넌트에서 서버 컴포넌트를 import할 수 없지만, 그래도 여전히 합성은 가능하다. 즉, 클라이언트 컴포넌트는 ReactNode를 props로 받을 수 있고 그리고 이 ReactNode는 서버 컴포넌트에 의해 렌더링 가능하다.

```js
// ClientComponent.client.jsx
export default function ClientComponent({ children }) {
  return (
    <div>
      <h1>Hello from client land</h1>
      {children}
    </div>
  )
}

// ServerComponent.server.jsx
export default function ServerComponent() {
  return <span>Hello from server land</span>
}

// OuterServerComponent.server.jsx
// OuterServerComponent 는 클라와 서버 모두에서 초기화 가능하다
// 따라서 서버 컴포넌트를 클라이언트 컴포넌트의 children으로 보낼 수 있다.
import ClientComponent from './ClientComponent.client'
import ServerComponent from './ServerComponent.server'
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```

이러한 제한은 RSC를 효과적으로 활용하기 위해 컴포넌트를 구성하는 방법에 큰 영향을 미친다.

## RSC렌더링의 라이프 사이클

RSC 서버 컴포넌트를 렌더링하기 위해서 실제로 어떤 일이 일어나는지 알아보기 위해 핵심적인 세부 서항을 살펴보자. 서버 컴포넌트를 사용하기 위해 모든 것을 이해할 필요는 없지만, 작동 방식에 대해 직관적으로 이해할 필요가 있다.

1. 서버가 렌더링 요청을 받는다.

서버가 렌더링 과정일부를 수행해야하므로, 페이지의 라이프 사이클은 항상 서버에서 시작된다. 이 중 root 컴포넌트는 항상 서버 컴포넌트이고, 다른 서버 또는 클라이언트를 렌더링할 수 있다. 서버는 요청에 전달된 정보에 따라 서버 컴포넌트와 어떤 props를 사용할지 결정한다. 이러한 요청은 일반적으로 특정 URL에서 페이지를 요청하는 형태로 나온다.

2. 서버가 루트 컴포넌트 엘리먼트를 JSON으로 직렬화

여기에서 최종 목표는 최초 root서버 컴포넌트를 기본 html 태그와 클라이언트 컴포넌트 placeholder트리로 렌더링하는 것이다. 그리고 이 트리를 직렬화하여 (json으로) 브라우저로 보내면, 브라우저가 이를 다시 역 직렬화하여 클라이언트 placeholder에 실제 클라이언트 컴포넌트를 채우고 최종 결과를 렌더링하는 작업을 수행한다. 그럼 위 예제에서 `OuterServerComponent`를 렌더링하고 싶다면, 단순히 `JSON.stringify(<OuterServerComponent/>)`를 한다면 직렬화된 렌더링 트리를 얻을 수 있을까?

거의 그렇다고 볼 수 있지만, 그것만으로는 충분하지 않다.

```js
// React element for <div>oh my</div>
> React.createElement("div", { title: "oh my" })
// 객체 형태로 표현
{
  $$typeof: Symbol(react.element),
  type: "div",
  props: { title: "oh my" },
  ...
}

// React element for <MyComponent>oh my</MyComponent>
> function MyComponent({children}) {
    return <div>{children}</div>;
  }
> React.createElement(MyComponent, { children: "oh my" });
// 객체 형태로 표현
{
  $$typeof: Symbol(react.element),
  type: MyComponent  // reference to the MyComponent function
  props: { children: "oh my" },
  ...
}
```

기본 html 태그 엘리먼트가 아닌 컴포넌트 엘리먼트의 경우, 컴포넌트를 참조하려고 하기 때문에 직렬화 할 수 없다.

따라서 모든 것을 적절히 JSON-stringify해주기 위해서는 리액트느 replacer function을 JSON.stringify()에 넘겨준다.

- 기본 html 태그의 경우에는 JSON으로 처리가 가능하므로 특별히 처리할 것이 없다.
- 만약 서버 컴포넌트라면 서버 컴포넌트함수를 props와 함께 호출하고 그 결과를 JSON으로 만들어서 내려보낸다. 이렇게 하면 서버 컴포넌트가 효과적으로 렌더링된다. 여기서 목적은 모든 서버 컴포넌트를 html 태그로 바꾸는 것이다.
- 만약 클라이언트 컴포넌트라면, 사실 JSON으로 직렬화가 가능하다. 이미 필드가 컴포넌트 함수가 아닌 모듈 참조 객체 (module reference object)를 가리키고 있다.

## module reference 객체란?

RSC는 module reference라고 불리는 리액트 엘리먼트의 type필드에 새로운 값을 넣을 수 있도록 제공한다. 이 값으로 컴포넌트 함수 대신, 이 `참조`를 직렬화한다.

예를 들어 `ChildComponent`는 아래와 같은 형태를 띈다.

```js
{
  $$typeof: Symbol(react.element),
  // 실제 컴포넌트 함수 대신에, 참조 객체를 갖게됨
  type: {
    $$typeof: Symbol(react.module.reference),
    // ClientComponent is the default export...
    name: "default",
    // from this file!
    filename: "./src/ClientComponent.client.js"
  },
  props: { children: "oh my" },
}
```

그렇다면 클라이언트 컴포넌트 함수에 대한 참조를 직렬화 할 수 있는 module reference 객체로 변환하는 것은 어디에서 이루어지는 것일까?

이러한 작업은 번들러에서 이루어진다. 리액트 팀은 RSC를 웹팩에서 사용할 수 있는 `react-server-dom-webpack`을 webpack loader나 node-register에서 제공하고 있다. 서버 컴포넌트가 \*.client.jsx파일에서 가져올 때, 실제로 import하는 것이 아닌 파일 이름과 그것을 참조하는 모듈 참조 객체만들 가져온다. 즉 클라이언트 컴포넌트 함수는 서버에서 구성되는 리액트 트리의 구성요소가 아니었따.

위 예제를 `JSON` 트리로 직렬화 한다면 아래와 같을 것이다.

```js
{
  // ClientComponent 엘리먼트를 `module reference` 와 함께 placeholder로 배치
  $$typeof: Symbol(react.element),
  type: {
    $$typeof: Symbol(react.module.reference),
    name: "default",
    filename: "./src/ClientComponent.client.js"
  },
  props: {
    // 자식으로 ServerComponent가 넘어간다.
    children: {
      // ServerComponent는 바로 html tag로 렌더링됨
      $$typeof: Symbol(react.element),
      type: "span",
      props: {
        children: "Hello from server land"
      }
    }
  }
}
```

![image](https://user-images.githubusercontent.com/63354527/173991791-a850eb80-e99a-4ea6-84fb-d740feea6527.png)

모든 props는 직렬화되야 한다.

전체 리액트 트리를 JSON 직렬화하고 있기 때문에 클라이언트 컴포넌트가 기본 html 태그에 전달하는 props도 직렬화 할 수 있어야한다. 그러나 (당연하게도) 서버 컴포넌트에서는 이벤트 핸들러를 props 전달할 수 없다.

```js
// 서버 컴포넌트는 함수를 prop으로 넘겨줄 수 없다.
// 함수는 직렬화 할 수 없기 때문이다.
function SomeServerComponent() {
  return <button onClick={() => alert("OHHAI")}>Click me!</button>
}
```

그러나 여기서 유의할 점은 RSC 프로세스 중에 클라이언트 컴포넌트를 마주하게 된다면, 클라이언트 컴포넌트 함수를 호출하거나 클라이언트 컴포넌트를 내림 차순으로 정렬하지 않는다는 것이다. 그러므로, 다른 클라이언트 컴포넌트를 인스턴스화 하는 클라이언트 컴포넌트가 있는 경우,

```js
function SomeServerComponent() {
  return <ClientComponent1>Hello world!</ClientComponent1>;
}

function ClientComponent1({children}) {
  // 클라이언트에서는 가능
  return <ClientComponent2 onChange={...}>{children}</ClientComponent2>;
}
```

ClientComponent2 는 RSC 트리에서 나타나지 않는다. 대신, module reference 가 있는 엘리먼트와 ClientComponent1의 props만 볼 수 있다. 그러므로, ClientComponent1에 이벤트 핸들러가 있는 ClientComponent2를 자식으로 보내는 것은 안전하다.

브라우저는 서버로부터 JSON 결과를 받고, 이제 브라우저에서 렌더링될 리액트 트리를 재구성하기 시작한다. type이 'module reference'인 엘리먼트를 만날 때마다, 실제 클라이언트 컴포넌트 함수에 대한 참조로 대체를 시도할 것이다.

이 작업은 다시 번들러의 도움이 필요하다. 클라이언트 컴포넌트 함수의 기능을 서버 module reference로 대체해 주었던 것도 번들러였고, 이 module reference를 브라우저가 실제 클라이언트 컴포넌트 함수로 대체하는 것을 아는 것도 번들러이다.

![image](https://user-images.githubusercontent.com/63354527/173994446-3611829a-7a5c-41ce-a487-9da3ebcdb992.png)

이제 이 트리를 렌더링하고, DOM에 커밋한다.

Suspense에서도 같은 원리일까?

`Suspense`에 대해서 간략하게 이야기를 하자면, 아직 준비되지 않은 요소가 필요할 때(데이터를 빠르게 가져오거나, 컴포넌트를 느리게 가져오는 등)리액트 컴포넌트에서 promises를 던질 수 있다. 이 promise는 `Suspense boundary`에서 잡을 수 있다. Suspense에서 하위 트리를 렌더링할 때 promise가 던져질 때마다 리액트는 이 promise가 resolve될 때 까지 리액트 하위 트리 렌더링을 일시 중지한 다음 다시 시도한다.

우리가 RSC 결과물을 만들기 위해 서버에서 서버 컴포넌트 함수를 호출할 때, 이 함수들은 각자 필요한 데이터를 가져올 때 promise를 던질 수 있다. 그리고 이 promise를 만나면, 앞서와 마찬가지로 placeholder를 위치 시킨다. 그리고 이 promise가 resolve되면 서버 컴포넌트 함수를 다시 호출하고, 성공하면 이 완료된 청크를 내보낸다. 실제로 RSC 출력 스트림을 생성하고, promise가 나타나면 일시 중지하고, 이것이 resolve되면 추가적인 chunck를 스트리밍한다.

마찬가지로, 브라우저에서 fetch 함수 호출로 RSC JSON 결과물을 스트리밍하고 있다. 이 프로세스 역시 결과물에서 placeholder를 마주하거나 (서버에서 던진 promise를 맞닥뜨린 경우), 스트림에서 placeholder를 아직 보지 못한 경우 (https://github.com/facebook/react/blob/main/packages/react-client/src/ReactFlightClientStream.js) promise를 던지는 것으로 끝날 수도 있다. 또는 클라이언트 컴포넌트 module reference를 마주치지만, 아직 브라우저에 로드된 클라이언트 컴포넌트 함수를 가지고 있지 않은 경우에도 promise를 던질 수 있다.
Suspense를 활용하면 서버 컴포넌트가 데이터를 가져올 때, 서버 스트리밍 RSC출력을 사용할 수 있으며 브라우저가 데이터를 점진적으로 렌더링하고 필요에 따라 클라이언트 컴포넌트 번들을 동적으로 가져올 수 있다.

## RSC Wire format

그런데 정확히 서버가 어떤 형태의 데이터를 보내는 것일까? 정확히 어떤 데이터가 서버에서 브라우저로 스트리밍 되는 것일까?

꽤 간단한 형태로 구성되어 있다. 한줄에 JSON blob데이터가 있고, 여기에 ID로 태그되어 있는 간단한 형식이다.

```js
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
J0:["$","@1",null,{"children":["$","span",null,{"children":"Hello from server land"}]}]

```

`M`으로 시작하는 라인은, 클라이언트 번들에서 컴포넌트 함수를 조회하는데 필요한 정보와 클라이언트 컴포넌트 module reference를 정의한다. `J`로 시작하는 줄은 앞서 `M`라인에서 정의된 클라이언트 컴포넌트를 참조하는 것으로, 실제 리액트 컴포넌트 element 트리를 정의한다.

이 형식의 포맷은 스트리밍으로 전송이 가능하다. 클라이언트가 전체 행을 읽는 즉시 JSON의 일부 구문을 분석하여 작업을 진행할 수 있다. 서버가 렌더링하는 동안 Suspense 바운더리에 도달한 경우, resolve 시각 청크에 해당하는 여러 `J`라인을 볼 수 있다.

아래 예제를 살펴보자.

```js
// Tweets.server.js
import { fetch } from 'react-fetch' // React's Suspense-aware fetch()
import Tweet from './Tweet.client'
export default function Tweets() {
  const tweets = fetch(`/tweets`).json()
  return (
    <ul>
      {tweets.slice(0, 2).map((tweet) => (
        <li>
          <Tweet tweet={tweet} />
        </li>
      ))}
    </ul>
  )
}

// Tweet.client.js
export default function Tweet({ tweet }) {
  return <div onClick={() => alert(`Written by ${tweet.username}`)}>{tweet.body}</div>
}

// OuterServerComponent.server.js
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
      <Suspense fallback={'Loading tweets...'}>
        <Tweets />
      </Suspense>
    </ClientComponent>
  )
}
```

위 예제에서, RSC스트림은 아래와 같이 나타난다.

```js
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}
S2:"react.suspense"
J0:["$","@1",null,{"children":[["$","span",null,{"children":"Hello from server land"}],["$","$2",null,{"fallback":"Loading tweets...","children":"@3"}]]}]
M4:{"id":"./src/Tweet.client.js","chunks":["client8"],"name":""}
J3:["$","ul",null,{"children":[["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}],["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}]]}]
```
