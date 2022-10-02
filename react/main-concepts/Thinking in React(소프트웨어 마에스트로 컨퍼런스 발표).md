# Thinking in React

<img width="774" alt="스크린샷 2022-09-29 오전 12 11 04" src="https://user-images.githubusercontent.com/63354527/193436030-315fe8aa-a153-4bd3-98af-09f6d3216fb6.png">

안녕하세요 이현진입니다.

`Thinking in React(React로 생각하기)`라는 주제로 발표를 진행하겠습니다. React로 생각한다는 표현이 조금 추상적일 수 있는데, 이 발표에는 React를 다룰 때 갖고 있어야하는 생각들을 담았습니다.

![](https://velog.velcdn.com/images/hyunjine/post/c7b40d7b-9225-422c-8b96-92979595cdcb/image.png)

먼저 `DOM`에 대한 이야기부터 시작해보겠습니다. `DOM`은 `Document Object Model`의 약자로, 브라우저가 `HTML`을 파싱하여 객체 형태로 만든 것을 말합니다.

![](https://velog.velcdn.com/images/hyunjine/post/9743851e-3bfe-4af9-82ea-276cb0eadb77/image.png)

위와 같은 `HTML`구조를 갖는 웹사이트가 있다고 가정해보겠습니다.
`HTML`은 문자열입니다.문자열은 다루기 어렵습니다.(파싱, 합치기등의 작업)

![](https://velog.velcdn.com/images/hyunjine/post/cfbcd4c4-4d6f-492a-bdd9-162ea47ffb9c/image.png)

브라우저는 이 다루기 어려운 문자열을 훨씬 다루기 쉬운 객체 형태로 바꿔주고, 이 객체를 `DOM`이라 합니다. 개발자는 JavaScript를 이용해 `DOM`을 조작하여 웹 애플리케이션을 개발합니다.

![](https://velog.velcdn.com/images/hyunjine/post/99b1ec81-37e1-48fd-8c66-c469f45d708d/image.png)

하지만 JavaScript로 직접 `DOM`을 조작하는 것은 여러가지 단점이 따라옵니다.

1. JavaScript로 `DOM`을 조작할 때 HTML의 구조를 파악하기 어렵습니다.(`DOM`을 직접 생성, 수정 삭제할 때 구조를 파악하기 어렵습니다.)
2. 표준을 따르는 브라우저도 많지만, 그 안에서도 다른 동작을 가지고 있을 수 있습니다.
3. [Live Collection과 Static Collection](https://im-developer.tistory.com/110)

![](https://velog.velcdn.com/images/hyunjine/post/4981b176-bf3a-4c99-b5f0-b94e2c0a85fa/image.png)

`Live Collection`은 `DOM API`가 반환한 값이 `Live`한 상태를 의미합니다. 즉, `DOM`의 실시간 변경사항에 따라 언제든 값이 바뀔 수 있습니다. 반면에 `Static Collection`은 `DOM`의 실시간 변경사항에 따라 값이 바뀌지 않습니다.

`DOM API`는 반환하는 값이 `Live` 할 수도 있고, `Static` 할 수도 있기 때문에 `"일관성이 없다"`라고 표현합니다.

`DOM API`가 일관성이 없고 사용하기 불편하다면 어떻게 해야할까요?

![](https://velog.velcdn.com/images/hyunjine/post/0541ef55-68e9-4f91-a3cc-9983f3d017bd/image.png)

가장 간단한 답으로는 `DOM API`을 사용하지 않는 방법이 있습니다. `DOM API`를 직접 사용하지 않고 중간에 매개체를 두어서 `DOM`을 조작할 수 있습니다.

이 매개체가 **React**입니다.

React는 `DOM`조작과 같이 어려운 일은 자신이 하고, 개발자에게는 훨씬 편리한 API를 제공해줍니다.

마치 `HTML`이라는 문자열을 직접 다루기 어렵기 때문에 `DOM`이라는 객체를 만든 것 처럼, `DOM`을 직접 다루기 어렵기 때문에 `React`를 만든 것이라고 할 수 있습니다.

![](https://velog.velcdn.com/images/hyunjine/post/7a256c65-927d-4f3b-ab0f-aa1d077ccf7b/image.png)

React는 웹 애플리케이션의 UI를 재사용 가능한 컴포넌트들을 모아서 구성합니다. 각 컴포넌트에는 `데이터 모델`이 존재합니다. 애플리케이션의 UI와 상호작용하려면 UI에 내재하는 `데이터 모델`을 바꿈으로써 상호작용할 수 있습니다.

이 `데이터 모델`을 React에서는 `State(상태)`라고 합니다. `상태`란 주어진 시간에 대해 시스템을 나타내는 것으로 언제든지 변경될 수 있고 `상태`가 업데이트되면 React 컴포넌트는 `렌더링`됩니다.

![](https://velog.velcdn.com/images/hyunjine/post/3e57afa8-1ec5-4832-b940-a4fe323e8386/image.png)

`렌더링`이란 React가 컴포넌트에게 현재 `Props`와 `State`에 기반하여 UI에 어떻게 보여지고 싶은지 알려달라고 요청하는 과정입니다. `렌더링`은 간단히 말해서 함수 컴포넌트를 호출하는 것이라고 할 수 있으며, 함수에서 반환하는 `JSX`는 시간에 따른 UI의 스냅샷과 같습니다.

![](https://velog.velcdn.com/images/hyunjine/post/3d0db11a-900f-445d-878a-b63fc08153e0/image.png)

컴포넌트가 위와 같은 트리 구조를 갖고 있다고 해보겠습니다. 빨간색 컴포넌트는 상태가 업데이트된 컴포넌트입니다. 상태가 업데이트되면 컴포넌트는 업데이트가 필요하다는 표시를 합니다.(빨간색 React로고)

React는 상태 업데이트를 감지하면 렌더링을 `큐(queue)`에 넣습니다.

![](https://velog.velcdn.com/images/hyunjine/post/fd9bbf85-7061-4a29-8bfc-41dbe1c6173b/image.png)

React는 트리의 최상단(`A`)부터 렌더 패스(`Render Pass`)를 시작합니다. `A`에는 업데이트가 필요하다는 마크가 없는 것을 보고 지나칩니다.

![](https://velog.velcdn.com/images/hyunjine/post/214a6f1d-ba11-4d2b-a367-60edbe14df95/image.png)

다음은 `B`를 방문합니다. React는 `B`에 업데이트가 필요하다는 마크가 있는 것을 보고 렌더링합니다. 여기서 중요한 점은 **React는 기본적으로 부모 컴포넌트가 렌더링되면, 모든 자식 컴포넌트를 재귀적으로 렌더링한다는 점입니다.**

![](https://velog.velcdn.com/images/hyunjine/post/2979d66f-3274-4061-b076-5fbf15451bc3/image.png)

이에 따라 `C`와 `D`는 업데이트가 필요하다는 마크가 없지만 부모 컴포넌트인 `B`가 렌더링되었기 때문에 `C`와 `D`를 렌더링합니다.

![](https://velog.velcdn.com/images/hyunjine/post/3f6f366c-d41e-440d-b4ee-90576b673eba/image.png)

다음으로 남은 `E`를 체크하고 업데이트가 필요하다는 마크가 없으므로 아래로 내려가서 업데이트가 필요하다는 표시가 있는 `F`를 발견하고 `F`를 렌더링합니다.

여기서 컴포넌트 트리 안에 있는 컴포넌트들 중에서는 `C`, `D`와 같이 직전과 똑같은 렌더링 결과물을 반환하는 컴포넌트가 존재합니다. 따라서 같은 결과물을 반환하는 컴포넌트는 `DOM`에 반영할 필요가 없습니다. 하지만 렌더링의 결과물이 같다는 것을 어떻게 알 수 있을까요?

![](https://velog.velcdn.com/images/hyunjine/post/4b8fd2dc-aa6b-402b-9ec1-8233b371b3e7/image.png)

React는 `VirtualDOM`을 활용합니다. 기존 `VirtualDOM`과 상태 업데이트 후의 `VirtualDOM`에서 바뀐 부분만을 계산([`diffing`](https://ko.reactjs.org/docs/reconciliation.html))하여 실제 바뀐 부분만 `DOM`에 적용합니다.

![](https://velog.velcdn.com/images/hyunjine/post/ff68b753-2e7f-4946-88fe-5fbc6eacf504/image.png)

![](https://velog.velcdn.com/images/hyunjine/post/716cf8c9-fa94-4365-8bf3-a4e97569abd5/image.png)

이를 React에서 `Reconciliation(재조정)`이라고 합니다.

![](https://velog.velcdn.com/images/hyunjine/post/d1af7085-410b-4263-829b-d1d54d359fdf/image.png)

이러한 사실을 바탕으로 렌더링을 두 단계로 쪼갤 수 있습니다.

- `Render phase(렌더 단계)`: 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 과정이 이루어지는 단계(`VirtualDOM 조작 단계`)
- `Commit phase(커밋 단계)`: 변경 사항을 실제 DOM에 적용하는 단계

**렌더링과 DOM을 업데이트하는 것은 같은 것이 아니며** 컴포넌트는 가시적인 변화가 없어도 렌더링될 수 있습니다.

![](https://velog.velcdn.com/images/hyunjine/post/dc409c72-6550-41fb-84cb-a27a62efed20/image.png)

렌더링은 기본적으로 `상태` 업데이트에 의해 발생됩니다. 따라서 React 애플리케이션은 `상태 관리`를 어떻게 하느냐에 따라 애플리케이션의 미래가 결정됩니다. 불필요하거나 중복된 상태는 버그의 일반적인 원인이 될 수 있습니다.

즉, 적절한 `상태`를 적절한 `컴포넌트`에 배치시켜야합니다.

![](https://velog.velcdn.com/images/hyunjine/post/04c56eaf-5e05-4622-a401-91473b5ee8ea/image.png)

`Props(properties)`는 컴포넌트간에 값을 전달할 때 사용합니다.(`데이터 전달`)

![](https://velog.velcdn.com/images/hyunjine/post/47988e1a-c5fd-4217-ba98-f9f3a6cfd1dc/image.png)

예를 들어 하위 컴포넌트 두개가 같은 상태(`현진`)를 갖는데 두 상태가 항상 함께 변경되기를 원할 수 있습니다.

![](https://velog.velcdn.com/images/hyunjine/post/8c8360d9-efe5-4f2c-90d1-e489572d00fe/image.png)

같이 변경되어야하는 두 상태는 `중복 상태`이므로 둘 다에서 상태를 제거하고 가장 가까운 공통 부모로 상태를 이동시킨후에 props를 통해 전달합니다. 이를 `상태 끌어올리기(Lifting State Up)`이라고 합니다.

![](https://velog.velcdn.com/images/hyunjine/post/ba942b5e-13ea-4268-ad4f-fdc2c90cda03/image.png)

상태를 끌어올린 후에 하위 컴포넌트에게 `Props`로 전달합니다.

React에서는 데이터의 흐름이 상위 컴포넌트에서 하위 컴포넌트로 한 방향으로만 흐릅니다.(`단방향 데이터 흐름, Unidirectional Data Flow`)

![](https://velog.velcdn.com/images/hyunjine/post/889c0267-f501-4d74-aca6-d53041a07401/image.png)

`현진`이라는 상태를 `이현진`으로 바꾸고 싶다고 해봅시다.
하위 컴포넌트에서 상위 컴포넌트의 상태를 변경하고 싶다면 어떻게 해야할까요?

![](https://velog.velcdn.com/images/hyunjine/post/4db9190f-3d10-4e4d-811f-14b1c84c69fe/image.png)

Props로 **상태를 업데이트하는 함수를 전달**하여 하위 컴포넌트에서 상태를 업데이트하는 함수를 호출하면됩니다.

![](https://velog.velcdn.com/images/hyunjine/post/2ebd2548-5ab0-4d18-977b-d7f9ff68c42c/image.png)

이렇게 상태를 업데이트하면 하위 컴포넌트에서 상위 컴포넌트의 상태를 업데이트할 수 있습니다. 이렇게 `역방향 데이터 흐름(Inverse Data Flow)`을 추가할 수 있습니다.

![](https://velog.velcdn.com/images/hyunjine/post/30670c8a-6bd1-4bdc-ab6a-d11ae9580b77/image.png)

지금까지 상태를 애플리케이션에 분배하고 다뤄봤습니다. 브라우저내에서 모든걸 처리할 수 있다면 클라이언트 상태로도 충분하지만 대다수의 애플리케이션은 서버 상태가 존재합니다.

![](https://velog.velcdn.com/images/hyunjine/post/2fa20785-2bc2-4470-afbf-fa8d9c9b27fe/image.png)

서버상태는 다음과 같은 특성을 지닙니다.

- 서버 상태는 사용자의 제어를 벗어난 위치에서 원격으로 유지된다.
- 비동기 요청을 통해 `fetching`또는 `updating`이 가능하다.
- 소유권을 공유한다. 즉 사용자 모르게 다른 사용자가 변경할 수 있다.
- 시간이 지남에 따라 `stale`또는 `outdated`된다.

![](https://velog.velcdn.com/images/hyunjine/post/d70765bd-bbc5-40e0-a243-0368270fef8b/image.png)

React는 UI 라이브러리이기 때문에 데이터를 `fetching`하는것에는 관심이 없습니다. 단지 `fetching`한 데이터를 UI에 반영시키는 것에만 관심이 많습니다.

![](https://velog.velcdn.com/images/hyunjine/post/a578f5c7-998c-4fad-8293-86f0cfe87681/image.png)

React는 상태에 따라 UI를 어떻게 렌더링할지에 관심이 있기 때문에 서버 상태를 다루려면 여러가지 상태를 정의해야합니다. `Loading`, `Error`, `Success` 상태를 정의하여 각각의 상태별로 매 렌더링마다 UI의 스냅샷을 찍어서 보여줍니다.

React에서는 상태를 업데이트하는 로직이 복잡해지면 `reducer`를 사용하듯이, 컴포넌트 내부에 `Loading`, `Error`, `Success`와 같은 상태를 두지 않고 전역 상태 관리자인 `Redux`를 사용하여 상태를 업데이트하는 로직을 컴포넌트 외부로 빼내서 비동기 요청에 대한 렌더링 로직을 작성했습니다.

![](https://velog.velcdn.com/images/hyunjine/post/194756c0-9119-473c-8914-5085224a6861/image.png)

여기서 우리는 의문을 제기해야할 필요가 있습니다. **전역 상태관리 라이브러리인 `Redux`의 역할이 과연 API 요청에 대한 각각의 상태를 정의해 렌더링 로직을 작성하는 것인가?**

답은 아니라고 생각합니다. 전역 상태관리자의 역할은 전체 애플리케이션에서 정말 전역적으로 관리해야하는 상태(theme, 사이드바 상태등)를 가지고 있어야합니다.

![](https://velog.velcdn.com/images/hyunjine/post/9f898be8-0776-49c0-97b7-415096dd01ed/image.png)

기존에는 API 요청의 상태(Loading, Error, Success)에 따라 적합한 UI를 보여주기 위해 컴포넌트 외부에 수많은 보일러 플레이트 코드를 작성해야했습니다.

React Query는 이를 해결합니다. React Query의 역할은 명확합니다.

`서버 상태를 관리하기위해 필요했던 보일러플레이트 코드를 제거한다. 그리고 단 몇줄의 코드로 대체한다.`

![](https://velog.velcdn.com/images/hyunjine/post/fd41511a-8c9b-44b0-b1a1-86fb7e8253f1/image.png)

React Query를 사용하면 여러 상태를 정의해야하는 문제는 해결됩니다. 하지만 컴포넌트가 `isLoading`과 같은 상태일 때 반환할 UI를 정의해줘야합니다.

**이는 UI의 일관성을 해칩니다.**

![](https://velog.velcdn.com/images/hyunjine/post/507b8693-1f77-4590-b39d-beb1cd980aa9/image.png)

`Suspense`는 이를 해결합니다. `Suspense`의 목표는 서버상태를 읽어오는 것을 React의 props와 state처럼 쉽게 다루는 것입니다.

![](https://velog.velcdn.com/images/hyunjine/post/4bedb6b4-fec8-4ae3-bd72-43e37fb8bcff/image.png)

이렇게 비동기적으로 데이터를 불러오는 컴포넌트를 `Suspense`감싸고 `fallback`으로 보여줄 컴포넌트를 전달합니다. 이렇게하면 기존 UI의 로딩 상태를 `명령형(imperative)` 방식으로 정의해야했던 것을 React의 패러다임에 맞게 `선언적(declarative)`인 방식으로 바꿀 수 있습니다.

> Suspense는 단순히 로딩 스피너가 아닙니다. React 18에서는 Suspense를 이용한 두개의 SSR(Server Side Rendering)기능이 추가됩니다. [HTML Streaming과 Selective Hydration](https://www.youtube.com/watch?v=pj5N-Khihgc)

![](https://velog.velcdn.com/images/hyunjine/post/9123524d-6968-4431-8002-573d5ca2a599/image.png)

지금까지 React에 관한 다양한 내용들을 다루었는데 마지막으로 React v18에 대해서 이야기해 보려고 합니다.

![](https://velog.velcdn.com/images/hyunjine/post/2b6b6393-cf14-4034-9d2b-24c5fe5e9752/image.png)

`2161일.` React팀이 React `v18.0.0`을 릴리즈하는데 걸린 시간입니다.(React팀이 Async rendering이라는 개념을 소개한 이후 2161일 걸림) 왜 이렇게 오래걸렸을까요?

![](https://velog.velcdn.com/images/hyunjine/post/63ea9a6f-83a6-4e17-a0f6-f5cf855a1b58/image.png)

React팀의 목표는 하나였습니다. `성능이 좋은 React를 만들어서 수백만개의 React로 만들어진 웹사이트 성능을 높인다.`

가장 큰 문제는 React가 아닌 React를 만든 언어에 있었습니다.

![](https://velog.velcdn.com/images/hyunjine/post/d511f619-1bb7-4a23-95bc-d7a2c8b84be8/image.png)

React는 JavaScript 위에서 만들어졌기 때문에 JavaScript의 제약을 따를 수 밖에 없습니다. 특히 JavaScript가 브라우저 위에서 동작하는 방식을 따릅니다.

![](https://velog.velcdn.com/images/hyunjine/post/12f2a4c6-5c7b-40df-99f8-13c6eccbf372/image.png)

**브라우저의 메인 스레드는 싱글 스레드로 `한번에 하나`의 작업만 처리할 수 있습니다.** HTML을 파싱하거나 JavaScript를 실행하거나 화면에 보이는 내용을 렌더링하는데 사용됩니다.

![](https://velog.velcdn.com/images/hyunjine/post/7015347b-cf23-44a3-84a5-b23f9487d380/image.png)

React를 비롯한 대다수의 UI 라이브러리 작동 방식도 이 한계(브라우저의 메인 스레드)에 종속될 수 밖에 없습니다. React도 화면에 그리기 위한 내부 연산, 즉 렌더링을 시작해서 화면을 완성할 때까지 실행을 멈출수 없습니다. 이를 React 팀에서는 블로킹 렌더링이라고 부릅니다.

![](https://velog.velcdn.com/images/hyunjine/post/76b8430b-11ae-47e9-87e4-497d6c4561a7/image.png)

정확히는 React 18 이전까지는 그랬습니다.
React 18에서는 동시성 기능이 추가되었습니다.([`Concurrent features`](https://www.youtube.com/watch?v=FZ0cG47msEk&t=1255))

![](https://velog.velcdn.com/images/hyunjine/post/189b5352-c1d1-42bd-a0a6-fe00dab7488d/image.png)

[동시성이란 두개 이상의 독립적인 작업을 잘게 나누어 Context Switching을 하며 동시에 실행되는 것처럼 보이도록 프로그램을 구조화하는 방법입니다.](https://tv.naver.com/v/23652451)

동시성 기능을 활용하면 렌더링을 잘게 쪼개어 상태 업데이트에 우선순위를 두어 좀더 긴급한 상태 업데이트를 먼저 수행할 수 있습니다.

![](https://velog.velcdn.com/images/hyunjine/post/83f45f6e-3d17-42db-a20e-9ed53e3952a4/image.png)

동시성 기능은 마치 고속차선과 일반차선을 두는 것과 같습니다. 고속차선으로는 좀더 긴급한 상태업데이트가 지나갈 수 있도록 하고, 일반차선으로는 좀 덜 긴급한 상태업데이트가 지나갈 수 있도록 개발자가 조절할 수 있습니다.

![](https://velog.velcdn.com/images/hyunjine/post/4f94ab34-cb25-4ab0-b306-568f86c2c088/image.png)

이제 마지막으로 발표내용을 정리해보겠습니다.

- React는 DOM 조작의 문제점을 해결하기위해 만들어졌다.
- React는 Reconciliation(재조정) 과정을 통해 DOM을 업데이트한다.
- React의 핵심은 상태 관리이다.
- React에서 Concurrent Feature를 활용해 렌더링 우선순위를 정할 수 있다.

![](https://velog.velcdn.com/images/hyunjine/post/21f54714-a00d-4729-835f-85a35d386757/image.png)

React는 UI를 변수에 저장할 수 있으며 값으로 전달할 수 있습니다. 즉 React는 `value UI`입니다.

**React의 핵심 원칙은 UI는 값이라는 것입니다.**

![](https://velog.velcdn.com/images/hyunjine/post/b57a1ce3-31ac-47f2-ad8c-70c7f1eca431/image.png)

감사합니다.

## 발표자료

[Thinking in React.pdf](https://drive.google.com/file/d/1cc_6qva6u9h2LOnC6ABlmYANgco7JK7J/view?usp=sharing)
[Thinking in React.key](https://drive.google.com/file/d/1m9r1bv8sCh-pILg4p8T_mTPmT2Bux4Ph/view?usp=sharing)
