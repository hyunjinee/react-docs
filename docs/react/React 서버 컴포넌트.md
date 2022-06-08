# React 서버 컴포넌트

이번주, React팀은 서버 주도(Server Driven) 멘탈 모델로 모던 UX를 가능하게 하는 것을 목표로하는 zero-bundle-size React 서버 컴포넌트를 시연했다. 서버 컴포넌트는 서버사이드렌더링과는 상당히 다르며 클라이언트 사이드 번들 크기를 매우 줄일 수 있다.

## 서버 사이들 렌더링의 한계

오늘날 서버 사이드 렌더링은 클라이언트 사이드 렌더링의 차선책일 수 있다. 컴포넌트를 구현하기 위한 자바스크립트 코드는 서버에서 HTML 문자열로 렌더링 된다. 이 HTML은 브라우저로 전달되어 빠른 속도로 First Contentful Paint 또는 Largest Contentful Paint로 나타날 수 있다.

그러나, 여전히 상호작용을 위한 hydration 단계가 종종 필요하기 때문에 자바스크립트 코드를 가져와야한다. 일반적으로 서버 사이드 렌더링은 초기 페이지를 로드하는데 사용되므로, hydration 후에는 다시 사용할 수 없다.

> SSR을 활용하여 클라이언트에서 hydrate를 사용하지 않는 서버 전용 React앱을 구축할 수 있는 것은 사실이지만, 모델에서 과도한 상호작용은 React를 벗어난 것을 수반하는 경우가 많다. 서버 컴포넌트를 사용한 하이브리드 모델은 각 컴포넌트별로 이를 결정하게 할 수 있다.

React 서버 컴포넌트를 사용하면, 컴포넌트를 정기적으로 다시 가져올 수 있습니다. 새 데이터가 있을 때 렌더링 되는 컴포넌트가 서버에서 실행되므로 클라이언트에 전송되는 코드의 양을 제한할 수 있습니다.

## Code Splitting

코드 분할을 통해 사용자가 필요로 하는 코드만을 제공하는 것은 모범사례로 여겨진다. 코드 분할을 통해 클라이언트에 보낼 필요가 있는 작은 번들로 나눌 수 있다. 서버 컴포넌트 이전에는 일일히 React.lazy()를 이용해 "분할 지점"을 정의하거나 routes/pages 같은 메타 프레임워크에 의해 설정된 휴리스틱을 사용해 새 청크를 생성했다.

```js
// PhotoRenderer.js (서버 컴포넌트 사용 전)
import React from "react"

// 이 중 하나는 *클라이언트 렌더링 될 때* 로딩을 시작한다.
const OldPhotoRenderer = React.lazy(() => import("./OldPhotoRenderer.js"))
const NewPhotoRenderer = React.lazy(() => import("./NewPhotoRenderer.js"))

function Photo(props) {
  // 로그인, 로그아웃, 컨텐츠 유형 등 기능 플래그 켜기
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />
  } else {
    return <PhotoRenderer {...props} />
  }
}
```

코드 분할에는 다음과 같은 문제가 있다.

- Next.js같은 메타 프레임워크 외부에서는, import 구문을 dynamic import 구문으로 대체해 최적화 작업을 수동으로 처리해야 하는 경우가 많다.
- 유저 경험에 영향을 주는 컴포넌트를 로드하기 시작하면 지연될 수 있다.

서버 컴포넌트는 클라이언트 컴포넌트의 모든 일반 import를 코드 분할 가능 지점으로 처리하는 자동 코드 분할을 도입한다. 또한 개발자는 (서버에서) 훨씬 이전에 사용할 컴포넌트를 선택할 수 있게 되어, 렌더링 프로세스 초기에 클라이언트가 더 일찍 가져올 수 있게 된다.

```js
// PhotoRenderer.server.js - 서버 컴포넌트
import React from "react"

// 이 중 하나가 *렌더링 되고 클라이언트로 스트리밍 되면* 로드를 시작한다.
import OldPhotoRenderer from "./OldPhotoRenderer.client.js"
import NewPhotoRenderer from "./NewPhotoRenderer.client.js"

function Photo(props) {
  // 로그인, 로그아웃, 컨텐츠 유형 등 기능 플래그 켜기
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />
  } else {
    return <PhotoRenderer {...props} />
  }
}
```

## 서버 컴포넌트가 Next.js SSR을 대체할까?

아니다. 둘은 많은 부분 다르다. 서버 컴포넌트의 초기 채택은 연구 및 실험이 계속됨에 따라 Next.js같은 실제 메타 프레임워크에서 이뤄질 것이다.

Dan Abramov의 서버 컴포넌트와 Next.js의 다른점에 대한 설명은 다음과 같다.

- 서버 컴포넌트 코드는 절대 클라이언트에게 전달되지 않는다. 많은 React를 사용한 SSR의 구현은 자바스크립트 번들을 통해 클라이언트로 컴포넌트 코드가 보내지게 된다. 이러 인해 상호작용이 지연될 수 있다.
- 서버 컴포넌트를 사용하면 트리의 어느 곳에서나 백엔드에 접근할 수 있다. Next.js를 사용한다면, 최상위 페이지에서만 가능한 getServerProps()를 통해 백엔드에 접근하는 것에 익숙할 것이다. 하지만, 임의 npm 컴포넌트는 이런 동작이 불가능하다.
- 트리 내부에서 클라이언트 측의 상태(state)을 유지하면서 서버 컴포넌트를 다시 가져올 수 있다. 이는 주요 전송 메커니즘이 HTML보다 훨씬 풍부하기 때문이다. 따라서 내부 상태 (e.g 검색 입력 텍스트, 포커스, 텍스트 선택)을 없애지 않고 서버에서 렌더링 한 부분(e.g 검색 결과 목록)을 다시 가져올 수 있게 한다.

서버 컴포넌트는 웹팩 플러그인을 통해 초기 통합 작업 중 일부를 수행한다.

- 모든 클라이언트 컴포넌트 찾기
- IDs => 청크 URLs간의 매핑 만들기
- Node.js로더를 통해 클라이언트로 import 하는 부분을 이 맵에 대한 참조로 대체한다.

Dan이 언급했듯이, 작업의 목표 중 하나는 메타 프레임워크를 훨씬 더 향상시키는 것이다.
