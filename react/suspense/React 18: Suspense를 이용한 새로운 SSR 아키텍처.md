# React 18: Suspense를 이용한 새로운 SSR 아키텍처

## Introduction

React의 SSR이 어떤 방식으로 동작했고 어떤 한계점이 있었으며, Suspense를 통해 어떻게 극복했는지에 대한 설명이 써있다.

그 동안 Suspense는 '코드 이쁘게 쓴 로딩 스피너'라는 조롱 섞인 질문을 받았는데, 이번에 React Core 팀에서 발표한 내용은 역시 Suspense가 단순히 문법적 설탕 같은 존재가 아님을 다시 한 번 각인시키는 듯 했다.

## Overview

React 18은 server-side-rendering의 성능의 구조적 개선을 포함한다. 이 개선점은 상당한 가치를 지니며, 수년간 작업의 정점이라고 할 수 있다. 개선점의 대부분은 드러나지 않지만, 특히 프레임워크를 사용하지 않는다면 새롭게 도입된 개념들 중 특히 인지하고 있어야 하는 것들도 있다.

새로운 API 중 핵심은 `pipeToNodeWritable`인데, 이것에 대해서는 [Server 사이드 React 18 업그레이드](https://github.com/reactwg/react-18/discussions/22)에서 자세히 읽어볼 수 있다.

## tl;dr

SSR은 서버상의 React Component를 이용하여 HTML을 만들어 유저에게 보낼 수 있도록 해준다. SSR은 유저로 하여금 JavaScript 번들이 로딩되고 실행되기 전에 페이지의 컨텐츠를 볼 수 있게 해준다.

React의 SSR은 아래와 같은 절차를 통해 항상 작동해왔다.

- 서버에서 전체 애플리케이션에서 사용할 데이터를 가져온다.
- 그 후, 서버에서 애플리케이션을 HTML로 렌더링 한 후 응답(response)로 보낸다.
- 그 후, 클라이언트에서 JavaScript로 불러온다.
- 그 후, 클라이언트에서 서버에서 생성된 HTML에 JavaScript를 연결시킨다.

여기서 핵심은 각각의 단계가 다음 단계 시작 전에 전체 애플리케이션에 대한 작업을 완료해야한다는 점이었다. 이 방식은 애플리케이션의 몇몇 일부분만 다른 부분보다 느릴 수 있기에 거의 모든 작지 않은 애플리케이션에 있어 효율적이지 못하다.

React 18은 Suspense를 통해 애플리케이션을 작고 독립적인 단위로 쪼개어 위 단계들을 독립적으로 진행할 수 있게 하고, 전체 애플리케이션의 SSR 프로세스를 막지 않게 해준다. 결과적으로 유저들은 컨텐츠를 더 빨리 볼 수 있고 상호작용도 훨신 빨리 할 수 있게 된다. 애플리케이션의 가장 느린 부분은 가장 빠른 부분의 발목을 잡지 않게 된다. 이 개선점은 자동으로 반영되고 작동시키기 위해 별도의 특수 코드를 작성해줄 필요가 없다.

이는 또한 이제 앞으로 React.lazy가 SSR 환경에서 작동하게 됨을 의미한다.

## SSR이란?

유저가 애플리케이션을 불러왔을 때, 최대한 인터렉션이 가능한 로딩이 완료된 페이지를 보여주고 싶을 것이다.

![image](https://user-images.githubusercontent.com/63354527/189284218-5667def9-34fb-4499-94b3-b75802136055.png)

위 그림은 초록색으로 칠해진 항목들이 페이지상에서 상호작용(interactive)이 가능하다는 것을 보여준다. 다른 말로, 각각의 JavaScript 이벤트 핸들러가 모두 붙어있는 상태이고, 버튼을 누르는 것은 state를 업데이트 해주는 등의 경우다.

하지만, JavaScript 코드가 전부 불러오기 전까지 페이지에 대한 상호작용을 할 수 없다. React뿐만 아니라 애플리케이션 코드 모드를 포함한다. 작은 앱이 아니라면 로딩 시간의 대부분은 애플리케이션 코드를 다운로드 받는데 사용될 것이다.

SSR을 사용하지 않으면 JavaScript가 로딩되는 동안 유저는 아래와 같은 빈 화면만 보게 될 것이다.

![image](https://user-images.githubusercontent.com/63354527/189284635-9b70ee7e-2d30-431e-8caf-9ae6dcbbf047.png)

이런건 좋지 않고, 그렇기에 SSR 사용을 추천한다. SSR은 React Component를 서버상에서 HTML로 렌더링하여 유저에게 보내줄 수 있게 한다. HTML은 링크나 폼 input과 같은 내장 웹 상호작용 요소들을 제외하고는 그다지 인터렉티브하지 않다. 하지만 유저들로 하여금 JavaScript가 불러와지는 동안 '무언가를' 볼 수 있게 해준다.

![image](https://user-images.githubusercontent.com/63354527/189284825-aa340b09-8446-4c24-9e42-e979367d35ca.png)

위 사진에서 회색 배경은 아직 상호작용을 할 수 없는 화면 부분을 나타낸다. 애플리케이션의 JavaScript 코드가 아직 불러와지지 않았기 때문에 버튼을 클릭하더라도 아무일도 일어나지 않는다. 하지만, 컨텐츠가 굉장히 많은 웹사이트의 경우 SSR은 연결 상태가 좋지 않은 유저들도 JavaScript를 불러오는 동안 컨텐츠를 읽거나 볼 수 있도록 하기에 굉장히 유용하다.

React와 애플리케이션 코드가 모두 불러와졌을 때, HTML을 상호작용이 가능하도록 만들고 싶을 것이다. 이제 React에게 말한다: "서버에서 생성한 HTML의 App 컴포넌트가 여기있어. HTML에 이벤트 핸들러 붙여줘!". React는 메모리 상에 컴포넌트 트리를 렌더링하지만, 그 작업을 위해 DOM 노드를 생성하는 대신 모든 로직을 기존에 존재하는 HTML 파일에 붙혀준다.

**컴포넌트를 렌더링하고 이벤트 핸들러를 붙혀주는 일련의 과정을 하이드레이션(hydration)이라 한다.** 굉장히 '드라이'한 HTML에 상호작용과 이벤트 핸들러 '수분'을 공급하는 행동이라한다.

하이드레이션이 끝나면 평소의 리액트라고 할 수 있다. 컴포넌트는 상태 값을 변경할 수 있고, 클릭에 반응할 수도 있다.

![image](https://user-images.githubusercontent.com/63354527/189285510-f7998c8d-98a0-4ec3-9e8c-5c9e380054e4.png)

SSR은 마치 "마술 트릭"처럼 보일 수 있다. 이 과정이 추가된다고 애플리케이션의 상호작용이 더 빨리 준비되지 않는다. 대신, 애플리케이션에서 상호작용이 필요없는 부분을 더 이른 시기에 볼 수 있게 해주어 유저들로 하여금 JS가 로딩되는 동안 정적 컨텐츠를 볼 수 있도록 해준다.하지만 이 마술 트길은 네트워크 상태가 좋지 않은 유저들에게 엄청나게 큰 차이를 가져오고 전반적으로 인지되는 성능(perceived performance) 향상을 가져온다. 또한 인덱싱과 빠른 속도로 인하여 SEO에도 도움을 준다.

## 오늘날의 SSR이 가진 문제점들은?

위 방법은 작동하지만, 많은 부분에 있어 최적이라 볼 수 없다.

### 무언가 보여주기 전에 모든 것을 다 가져와야 한다.

오늘날의 SSR이 가진 문제 중 하나는 컴포넌트로 하여금 "데이터를 기다리도록"하지 않는다. 현재 제공되는 API를 사용하면 HTML에 렌더할 때 서버상에서 컴포넌트에 필요한 데이터를 모두 다 준비해놓아야한다. 이 뜻은 클라이언트에 HTML을 보내기 전에 서버상에서 모든 데이터를 모아놔야 한다는 것이다. 이 방법은 꽤나 비효율적이다.

예를들어, 댓글이 있는 글을 렌더링하고 싶다고 가정해보자. 댓글은 이른 시기부터 보여주는 것이 중요하기 때문에 서버사이드 HTML 출력에 추가하고 싶다. 하지만 DB나 API 레이어의 속도가 느린데 이건 건드릴 수 없는 상황이다. 이럴 경우 힘든 결정을 내려야한다. 서버 출력물에서 제외하면 유저는 JS가 완벽히 불러와지기 전까지 볼 수 없을 것이다. 하지만 서버 출력에 포함시키면 댓글이 불러와지고 전체 트리를 렌더링하기 전까지 나머지 HTML 전송을 하는 것을 지연시켜야한다. (네비게이션바, 사이드바, 그리고 심지어 포스팅 본문까지도 여기에 포함된다.) 이건 좋지 않다.

> 한가지 덧붙이자면, 몇몇 데이터를 가져오는 방법들은 데이터가 완전히 불러와지기 전까지 트리를 HTML에 렌더하고 결과물을 버리는 방식을 반복적으로 수행한다. 이는 React가 더 좋은 옵션을 제공하지 않기 때문이고, 우리는 이런 극단적인 타협책을 요구하지 않는 방법을 제시하고자 한다.

### 상호작용을 하기 전에 모든 항목을 다 하이드레이션 해줘야한다.

하이드레이션 자체에도 비슷한 문제가 있다. 오늘날 React 트리는 트리를 한 번의 작업을 통해 하이드레이션을 진행한다. 이 뜻은, 하이드레이션을 한번 시작하면 (말하자면 컴포넌트 함수를 호출하는 과정), React는 전체 트리에 대해 이 과정을 완료하기 전까지 멈추지 않는다. 결과적으로 컴포넌트 중 어느 하나라도 상호작용 하기 위해서는 모든 컴포넌트가 하이드레이션 되어야한다.

예를 들어 댓글 위젯쪽에 굉장히 시간이 오래 걸리는 렌더링 로직이 들어있다고 가정하자. 본인의 컴퓨터에서는 빠르게 동작할 수 있지만, 저사양 디바이스에서는 모든 로직을 실행하는 것이 빠르지 않고, 심지어 몇 초간 화면을 고정시킬 수도 있다. 물론, 이상적으로 클라이언트 사이드에 이런 로직은 없을 것이다. (그리고 이런 경우를 대비하기 위해 Server Component가 개발되고 있는 것이다.) 하지만 몇몇 로직은 부착된 이벤트 핸들러의 작업을 결정하고 상호작용에 필수적이기 때문에 이런 상황이 불가피하다. 결과적으로 한 번 하이드레이션이 시작되면 전체 트리가 완전히 하이드레이션 되기 전까지는 네이게이션바, 사이드바, 포스팅 본문과 상호작용할 수 없다. 특히나 네비게이션의 경우 유저가 이 페이지 자체에서 떠나고 싶지만 현재 클라이언트에서 열심히 하이드레이션을 진행하고 있기 때문에 더 이상 보고 싶지 않은 페이지에 남아 있어야하는 굉장히 안좋은 케이스이다.

### 어떻게 해결할 수 있을까?

이 문제들 사이에 공통점이 있다. 이른시키부터 무언가를 수행하거나(다른 작업들을 모두 블로킹하기 때문에 UX를 훼손한다.), 나중에 수행하거나(이 경우 시간을 낭비하기 때문에 UX가 훼손된다.)를 선택하도록 강요한다는 것이다.

이런 이유는 "폭포수"가 있기 때문에다. 데이터 가져오기(서버) -> HTML로 렌더링 (서버) -> 코드 불러오기 (클라이언트) -> 하이드레이션(클라이언트). 이 중 그 어떤 단계도 이전 단계가 전체 애플리케이션에 대하여 끝나기 전까지 시작되지 못한다. 그리고 이게 바로 비효율적인 이유이다. 우리가 제시하는 해결책은 작업을 쪼개 전체 애플리케이션이 아닌 각각의 부분들에 대해 이 단계들을 수행할 수 있게 하는 것이다.

이것이 새로운 개념이라고 할 수 없. 예를 들어 Marko는 이런 패턴을 도입한 JavaScript 웹 프레임워크중 하나다. 여기서 과제는 이런 패턴을 React의 프로그래밍 모델에 적용시키는 것이었다. 해결하는데도 다소 시간이 걸렸다. 이런 이유로 2018년에 Suspense 컴포넌트를 소개하였다. 처음에 소개하였을 때, 클라이언트 단에서 단순히 코드 lazy-loading만을 지원하였다. 하지만 목표는 서버 렌더링과 통합하여 이러한 문제들을 해결하는 것이었다.

이제 React 18에서 Suspense를 이용하여 이 문제를 어떻게 해결하는지 알아보자.

## React 18: HTML 스트리밍과 선택적 하이드레이션 (Streaming HTML and Selective Hydration)

React 18에서는 Suspense를 이용하여 두개의 주요 SSR 기능들이 추가된다.

- 서버에서 HTML을 스트리밍 형식으로 전달.renderToString을 새로운 pipeToNodeWritable 메소드로 바꿔줘야한다.
- 클라이언트에서 선택적 하이드레이션. 사용하기 위해 클라이언트 단에서 createRoot로 바꿔주고 애플리케이션의 바뀐 부분을 Suspense로 감싸줘야한다.

이 기능들이 어떤 역할을 하고 어떤 문제들을 해결하는지 보기 위해 아래 예제를 확인해보자.

### 모든 데이터를 불러오기 전에 HTML을 스트리밍(Streaming HTML before all the data is fetched)

오늘날 SSR에서 HTML렌더링과 하이드레이션은 "다 하거나 아무것도 안하거나"만 할 수 있다. 먼저 모든 HTML을 렌더링한다.

```html
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
  </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section>
    <!-- Comments -->
    <p>First comment</p>
    <p>Second comment</p>
  </section>
</main>
```

클라이언트는 HTML을 받게 된다.

![image](https://user-images.githubusercontent.com/63354527/189461625-9a2050c6-154b-406b-aff7-a7a5df0ac310.png)

그 후 코드를 불러온 다음 전체 애플리케이션을 하이드레이션 한다.

![image](https://user-images.githubusercontent.com/63354527/189461638-451bce8b-d8a2-4e5c-9345-aab341661ea2.png)

하지만 React 18은 새로운 가능성을 제공한다. 페이지 부분을 Suspense로 감싸줄 수 있다.

예를 들어 댓글 부분을 감싼 다음 React로 하여금 준비되기 전까지 Spinner 컴포넌트를 보여주도록 한다.

```jsx
<Layout>
  <NavBar />
  <Sidebar />
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

Comments 항목을 Suspense로 감싸줌으로써, React에게 댓글 부분을 기다리지 않고 나머지 페이지에 대해 HTML을 스트리밍 하도록 할 수 있다. 댓글 부분 대신에 React는 placeholder에 해당하는 Spinner 컴포넌트를 보내준다.

Comments 항목을 Suspense로 감싸줌으로써, React에게 댓글 부분을 기다리지 않고 나머지 페이지에 대해 HTML을 스트리밍 하도록 할 수 있다. 댓글 부분 대신에 React는 placeholder에 해당하는 Spinner 컴포넌트를 보내준다.

![image](https://user-images.githubusercontent.com/63354527/189461966-a1613c54-00f9-469b-a687-9d9705796cbd.png)

이제 최초 HTML에서 댓글(Comments)은 찾을 수 없다.

```jsx
<main>
  <nav>
    <!--NavBar -->
    <a href="/">Home</a>
  </nav>
  <aside>
    <!-- Sidebar -->
    <a href="/profile">Profile</a>
  </aside>
  <article>
    <!-- Post -->
    <p>Hello world</p>
  </article>
  <section id="comments-spinner">
    <!-- Spinner -->
    <img width="400" src="spinner.gif" alt="Loading..." />
  </section>
</main>
```

여기서 끝나지 않는다. 서버단에서 댓글에 해당되는 데이터가 준비되면, React는 동일한 스트림에 추가되는 HTML과 해당 HTML을 "올바른 장소"에 위치 시키기 위한 작은 인라인 script 태그를 보내준다.

```js
<div hidden id="comments">
  <!-- Comments -->
  <p>First comment</p>
  <p>Second comment</p>
</div>
<script>
  // This implementation is slightly simplified
  document
    .getElementById("sections-spinner")
    .replaceChildren(document.getElementById("comments"));
</script>
```

결과적으로 클라이언트에서 React 자체가 불러와지기도 전에 늦게 도착한 댓글 부분의 HTML이 들어오게 된다.

![image](https://user-images.githubusercontent.com/63354527/189463052-65b62945-071c-4721-af03-95f74357aea7.png)

이 방법은 우리의 첫번째 문제를 해결한다. 이제 무언가를 보여주기 위해 모든 데이터를 불러와줄 필요가 없다. 화면의 일부가 최초 HTML을 보내는 작업을 지연시키면, 더 이상 모든 HTML을 지연시킬 것인지, 해당 파트를 HTML에서 제외할 것인지 선택할 필요가 없다. 그 부분만 HTML 스트리밍 상 나중에 들어오게 할 수 있다.

전통적인 HTML 스트리밍 방식과 다르게 탑다운 순서로 진행될 필요도 없다. 예를들어, 사이드바가 데이터가 필요하면 Suspense에 감싸주면 React가 그 부분에 placeholder를 넣고 포스팅을 렌더링할 것이다.

그리고 사이드바에 해당하는 HTML이 준비되면 React는 그 HTML을 올바른 곳에 위치시키는 script 태그와 같이 스트리밍 해준다. 이 과정은 이미 트리상에서 더 먼 곳에 위치하는 포스팅 부분이 이미 전송된 다음에도 이뤄질 수 있다. 데이터가 특별한 순서에 맞춰 로딩되어야 하는 필요사항은 없다. 어디에 로딩 스피너가 나타날지 지정해주면, React가 나머지 부분들을 알아서 처리한다.

> 이게 동작하기 위해 데이터를 가져오는 솔루션도 Suspense를 내재해야한다. Server Component는 Suspense가 포함될 예정이지만, 이와 더불어 다른 React data fetching 라이브러리도 Suspense를 내재하기 위한 방법을 제공할 것이다.

### 코드가 모두 불러와지기 전에 페이지 하이드레이팅(Hydrating the page before all the code has loaded)

최초 HTML을 더 이른 시점에 보낼 수 있지만, 아직 문제가 남아있다. 댓글 위젯을 위한 JavaScript 코드가 로딩되기 전에, 클라이언트상에서 애플리케이션을 하이드레이션할 수 없다. 코드 규모가 크면 꽤나 오래 걸릴 수 있는 작업이다.

큰 번들 사이즈를 피하기 위해 주로 코드 스플리팅이 사용된다. 특정 코드의 부분이 동기적으로 로드될 필요없다 명시해주면 번들러가 별도의 script 태그로 분리해준다. React.lazy를 사용하여 댓글 부분의 코드를 스플리팅하여 메인 번들에서 분리시킬 수 있다.

```js
import { lazy } from "react"

const Comments = lazy(() => import("./Comments.js"))

// ...

;<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

이전에, 이 방법은 서버사이드 렌더링 환경에서 동작하지 않았다. 하지만 React 18에서 Suspense는 댓글 위젯이 불러와지기 전에 애플리케이션을 하이드레이션 할 수 있게 해준다.

유저의 관점에서, 최초에 HTML로 스트리밍된 상호작용이 불가능한 컨텐츠를 보게된다.

![image](https://user-images.githubusercontent.com/63354527/189463474-467870e4-98cf-41ac-b13b-ad7cb71236ec.png)

![image](https://user-images.githubusercontent.com/63354527/189463504-4cc15ffd-040e-4425-80b9-32da49169a5b.png)

이제 React에게 하이드레이션을 지시한다. 댓글에 해당하는 코드가 아직 불러와지지 않았지만, 괜찮다.

![image](https://user-images.githubusercontent.com/63354527/189463600-0bb0e901-7928-41c4-94d7-bf00f50d28cb.png)

이게 선택적 하이드레이션(Selective Hydration)의 예제다. Comments를 Suspense로 묶음으로써 React로 하여금 스트리밍과 하이드레이션이 지연되는 요소로 블로킹되는 것을 막아준다.이제 두번째 문제점이 해결되었다. 하이드레이션을 시작하기 위해 모든 코드가 불러와지는 것을 기다릴 필요가 없다. React 코드 부분 부분이 로딩될 때마다 하이드레이션을 진행할 수 있다.

React는 해당 댓글 섹션 코드가 모두 불러와진 뒤에 그 부분만 하이드레이션을 시작하게 된다.

![image](https://user-images.githubusercontent.com/63354527/189463664-dc932ead-598d-455d-8122-45ba1932594b.png)

선택적 하이드레이션 덕분에 무거운 JS 코드 일부가 나머지 페이지의 상호작용을 막지 않게 된다.

### HTML이 모두 스트리밍 되기 전에 하이드레이션 시작하기 (Hydrating the page before all the HTML has been streamed)

React는 이 모든 것을 자동으로 관리하기 때문에 작업이 예상치 못한 순서로 진행되는 것에 대해 걱정할 필요가 없다. 예를 들어, HTML을 스트리밍하는 것 자체도 아래와 같이 시간이 지연될 수 있다.

![image](https://user-images.githubusercontent.com/63354527/189463716-5d07c846-5282-4481-a650-06435f505d68.png)

만약 JavaScript 코드가 전체 HTML 보다도 일찍 불러와진다면, React는 더이상 기다릴 필요가 없다. 나머지 페이지를 하이드레이션하면 되기 때문이다.

![image](https://user-images.githubusercontent.com/63354527/189463746-43f2da64-cb17-4d8c-afb7-423b6a4b7ead.png)

댓글에 해당하는 HTML이 불러와지면, 아직 그 부분은 JS가 불러와지지 않았기 때문에 상호작용이 불가능한 상태로 나타난다.

![image](https://user-images.githubusercontent.com/63354527/189463783-4e31ad40-c0cf-4ba7-9df2-9abd057119f3.png)

마지막으로 댓글 위젯에 대한 JavaScript 코드가 불러와지면, 전체 페이지는 이제 완벽하게 상호작용이 가능해진다.

![image](https://user-images.githubusercontent.com/63354527/189463804-aeb61633-4fba-4a16-b4e3-076108ebd0df.png)

### 모든 컴포넌트가 하이드레이션되기 전 페이지상 상호작용(Interactive with the page before all the components have hydrated)

댓글 부분을 Suspense로 감쌌을 때 드러나지 않은 개선점이 하나 더 있다. 하이드레이션 과정 자체가 더이상 다른 작업을 할 수 없게 브라우저를 점유하지 않는다.

예를 들어, 댓글 부분의 하이드레이션이 진행되는 동안 유저가 사이드바를 클릭했다고 가정하자.

![image](https://user-images.githubusercontent.com/63354527/189463846-65767816-f85b-4667-8fe9-e62045cc0855.png)

React 18에서 Suspense boundary 내부에서 발생하는 하이드레이션 과정에는 브라우저가 이벤트를 핸들링할 수 있도록 작은 구멍들이 포함된다. 이 방법을 통해 클릭은 즉각적으로 처리되고 브라우저는 저사양 디바이스에서 발생하는 긴 하이드레이션 구간에 갇히지 않아도 된다. 예를 들어 유저는 이제 더 이상 관심있지 않은 페이지에서 네비게이션바를 이용하여 이동할 수 있다.

우리의 예제에서 댓글만 Suspens에 감싸졌기에 나머지 페이지에 대한 하이드레이션은 한번의 작업으로 이루어진다. 하지만, Suspense를 아래와 같이 더 많이 배치함으로써 이 문제를 해결할 수 있다. 예를 들어 사이드바에서도 Suspense를 적용해볼 수 있다.

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<Spinner />}>
    <Sidebar />
  </Suspense>
  <RightPane>
    <Post />
    <Suspense fallback={<Spinner />}>
      <Comments />
    </Suspense>
  </RightPane>
</Layout>
```

이제 NavBar와 Post를 가지고 있는 최초의 HTML이 전송된 뒤에도 서버로부터 SideBar와 Comments가 스트리밍 될 수 있다. 하지만 이런 경우 하이드레이션에도 영향을 준다. 예를 들어 두 항목의 HTML이 모두 불러와졌지만, 아직 코드는 불러와지지 않았을 경우 아래와 같이 나타난다.

![image](https://user-images.githubusercontent.com/63354527/189463975-9dcc206a-106e-4af9-ae56-06b6662e9858.png)

이제 사이드바와 댓글 코드를 가지고 있는 번들이 불러와진다. React는 둘 모두를 하이드레이션을 하는데, 트리상에서 더 먼저 발견되는 Suspense boundary 부터 시작한다. 이 경우 사이드바가 해당된다.

![image](https://user-images.githubusercontent.com/63354527/189464017-34d4e6dc-7f16-4385-83fe-dde683ee0917.png)

하지만 예를 들어, 유저는 코드가 로드된 댓글 위젯쪽에 대해 먼저 상호작용을 한다고 가정해보자.

![image](https://user-images.githubusercontent.com/63354527/189464081-21420915-81a9-4692-9a78-570c8a2019ec.png)

React는 해당 클릭을 기록하고, 이것이 더 급하기 때문에 댓글 항목에 대한 하이드레이션에 우선순위를 부여한다.

![image](https://user-images.githubusercontent.com/63354527/189464096-5553fb49-135e-4538-ad61-cd8079390f00.png)

댓글 위젯 코드가 하이드레이션을 마치면, React는 기록된 클릭이벤트를 다시 실행하고 컴포넌트로 하여금 해당 상호작용에 반응하도록 한다. 그 후 이제 React는 급한 작업이 없기에 사이드바를 하이드레이션 할 것이다.

![image](https://user-images.githubusercontent.com/63354527/189464132-276a04cc-51fa-4c5a-9775-cadb3df7e7ea.png)

이 과정은 우리의 세번째 문제를 해결한다. 선택적 하이드레이션 덕분에 우리는 "아무것이나 상호작용하기 위해 모든 것을 다 하이드레이션 해야한다"를 하지 않아도 도니다. React는 최대한 빨리 모든 것을 하이드레이션 할 것이고, 유저의 상호작용을 기반으로 홤녀상에서 가장 급한 부분에 우선순위를 부여할 것이다. 선택적 하이드레이션의 장점은 애플리케이션에 Suspense를 적용하고, 각각의 영역이 더 작아지게 되면 더욱 명확해질 것이다.

![image](https://user-images.githubusercontent.com/63354527/189464205-ee9d118f-5218-4b56-b080-2753170406ea.png)

위 예제에서 유저는 하이드레이션이 시작된 후 첫번째 댓글을 클릭하였다. React는 부모 Suspense 영역에 대한 하이드레이션을 우선시 하지만, 관련없는 형제 컴포넌트에 대한 것은 우선 건너뛸 것이다. 이 방법은 마치 하이드레이션이 즉각적으로 이뤄진다는 착각을 불러오는데 이는 상호작용에 해당하는 컴포넌트가 가장 먼저 하이드레이션 되기 때문이다. React 애플리케이션은 나머지 부분들을 곧 이어 하이드레이션 하게 된다.

실질적으로 Suspense를 애플리케이션의 root에 가장 가까운 곳에 추가해줄 것이다.

```jsx
<Layout>
  <NavBar />
  <Suspense fallback={<BigSpinner />}>
    <Suspense fallback={<SidebarGlimmer />}>
      <Sidebar />
    </Suspense>
    <RightPane>
      <Post />
      <Suspense fallback={<CommentsGlimmer />}>
        <Comments />
      </Suspense>
    </RightPane>
  </Suspense>
</Layout>
```

이 예제를 토대로 최초의 HTML은 NavBar의 컨텐츠를 포함하지만, 나머지는 스트리밍되고 코드가 로드됨과 동시에 부분 부분 하이드레이션되며, 유저가 먼저 상호작용한 부분은 하이드레이션 우선순위를 갖게 된다.

## Demo

새로운 Suspense SSR 아키텍처가 어떻게 동작하는지 보여주기위해 [시도해볼 수 있는 데모]()를 추가했다. 인위적으로 속도 제한이 걸려있고 server/delays.js 파일에서 지연시간을 조절할 수 있다.

- API_DELAY: 서버상에서 댓글 데이터를 가져오는데 더 오래 걸리도록 하여 HTML의 나머지 부분이 이른 시기에 어떻게 보내지는지 보여준다.
- JS_BUNDLE_DELAY: script 태그가 불러와지는 것을 지연시켜 댓글 위젯의 HTML 부분이 React 애플리케이션 번들이 다운로드 되기도 전에 들어올 수 있는지 보여준다.
- ABORT_DELAY: 서버상에서 데이터 가져오기가 지나치게 오랜 시간이 걸릴 경우 서버로 하여금 과정을 포기하고 렌더링을 클라이언트에게 넘기는 것을 보여준다.

## 결론적으로

React 18은 SSR에 있어 두개의 주요한 기능들을 제공한다.

- HTML 스트리밍은 가장 빠른 시점에서부터 HTML을 생성할 수 있도록 해주고, 추가적인 컨텐츠는 해당 장소에 컨텐츠가 갈 수 있도록 해주는 script 태그와 함께 스트리밍 형태로 보내줄 수 있게 해준다.
- 선택적 하이드레이션은 애플리케이션의 나머지 HTML과 JavaScript가 완전히 다운로드되기 전에 하이드레이션을 최대한 빨리 시작할 수 있게 해준다. 또한 유저가 상호작용하는 부분에 대한 하이드레이션에 우선순위를 제공하여 마치 즉각적으로 하이드레이션이 이뤄지는 것 같은 착각을 불러일으킨다.

이 기능들은 React의 SSR이 오랜 기간동안 가지고 있는 세 개의 문제를 해결한다.

- 서버 상에서 HTML을 보내기 전에 더 이상 모든 데이터가 불러와지기를 기다리지 않아도된다. 대신 애플리케이션의 껍데기를 보여줄만큼 준비가 되면 HTML을 보내기 시작하고 나머지 HTML은 준비되었을 때 스트리밍해줄 수 있다.
- 하이드레이션을 시작하기 위해 모든 JavaScript가 불러와지기를 기다리지 않아도 된다. 대신, 서버 렌더링과 코드 스플리팅을 같이 사용할 수 있다. 서버 HTML은 보존되고, React는 관련 코드가 불러와지면 하이드레이션을 한다.
- 페이지상의 상호작용을 위해 모든 컴포넌트가 하이드레이션되기를 기다리지 않아도 된다. 대신 선택적 하이드레이션을 통해 유저가 상호작용하고 있는 컴포넌트에 우선 순위를 부여하고 먼저 하이드레이션 해줄 수 있다.

Suspense 컴포넌트는 이 모든 기능들을 참여시키는 역할을 한다. 개선점들 자체는 React 내부에서 자동으로 이뤄지고 이미 기존에 있는 이미 기존에 있는 대부분의 React 코드와 동작할 것이 기대된다. 이것은 로딩 상태를 선언적으로 표현하는 것의 힘을 보여준다. if (isLoading)을 Suspense로 바꾸는 것은 큰 변화가 아닌 것 같지만, 이 과정은 위 모든 개선점들을 가능하게 해준다.
