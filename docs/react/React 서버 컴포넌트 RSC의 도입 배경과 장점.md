# React 서버 컴포넌트 RSC의 도입 배경과 장점

프론트엔드 세계에는, 시간이 갈수록 많은 변화를 겪고, 많은 유용한 프레임워크와 라이브러리들이 생겨나고 있습니다. 그 중에 중요한 요소중 하나가 data를 fetching 및 렌더링하는 부분인데, 그래프 QL이라던가 react-query와 같은 기술들도 모두 서버로부터 데이터를 받아오는데에 더 효율적이고 프로젝트에 적합한 구조를 도입할 수 있도록 해주는 역할을 합니다.

최근에는 NextJS와 같은 프레임워크를 이용해 필요한 곳에서만 부분적으로 SSR을 채용하면서, CSR과 SSR의 각각의 이점을 가져가고 있는 추세입니다. 하지만 이런 구조 또한 완전하다거나 안정적인 방식이라고 하기에는 단점이 명확히 존재하고, 꾸준히 변화하고 있는 부분입니다. 제 개인적인 생각으로는 state management와 마찬가지로, 새로운 큰 혁신이나 변화가 필요한 과도기에 놓여있다고 생각합니다. 이런 상호아에서 리액트 개발팀은 리액트 18을 공개하기 앞서 작년 React Server Component라는 새로운 방식의 도입을 발표합니다. 계속 연구및 개발중이고 많은 사람들의 의견과 피드백을 거쳐서 나올 것이라고 합니다.

## 왜 RSC(React Server Component)일까?

기존 SSR 방식은 다음과 같이 동작합니다.

```js
// ES modules
import ReactDOMServer from "react-dom/server"
// CommonJS
var ReactDOMServer = require("react-dom/server")

ReactDOMServer.renderToString(element)
```

![image](https://user-images.githubusercontent.com/63354527/172386767-569d28d7-499a-4aad-bffb-d5b105884562.png)

renderToString 함수를 통해 초기 렌더링 결과를 HTML String으로 반환하고, 이를 바탕으로 첫 요청의 응답으로 마크업을 포함한 HTML 문서를 빠르게 사용자에게 보여줍니다. 그리고 클라이언트 단에서 ReactDOM.hydrate함수를 통해 바뀐 부분만 수분을 공급해줍니다.

사용 예시는 다음과 같다.

일반적인 컴포넌트

1. 검색창에 뭔가 입력
2. onChange => 검색 Fetch => 검색결과 받아옴
3. 받은 검색 결과 리액트에 넘겨서 컴포넌트 렌더링

RSC

1. 검색창에 뭔가입력
2. onChange => 렌더링 서버에 키워드 Fetch => 서버에서 검색 Fetch요청 보냄 => 검색결과 받아서 비 Json/비 HTML형식으로 마크업 빌드해서 클라이언트에 보냄
3. 클라이언트에서 마크업 받아서 정적 UI로 렌더링 (React 컴포넌트가 아니므로 컴포넌트 처리에 드는 시간 절약)

기존의 방식의 단점은 뭘까요?

기존의 SSR방식에는 몇가지 단점이 존재합니다. SSR방식에서는 페이지가 아닌 컴포넌트를 정적으로 export할 수가 없습니다. 실질적으로 리액트 개발자가 조작하는 코드는 컴포넌트 단위로 구성이되는데, 이는 불편한 점이죠. Next.js를 다뤄보신 분이라면, pages 하위에 있는 컴포넌트가 아닌 이상, SSR관련 함수들 (getServerSideProps) 을 사용할 수 없다는 것을 알고 계실 것입니다. 어쩔 수 없이 pages 컴포넌트에서 서버단에서 fetch해온 데이터를 props등 하위 컴포넌트로 내려주거나, 그렇지 않다면 하위 컴포넌트에서 CSR방식으로 데이터를 가져오는 방법을 택해야합니다.

뿐만 아니라 UI를 렌더링 하는데 필요하지 않은 데이터 처리 과정에서 사용되는 모듈까지 함께 번들링되기 때문에 큰 규모의 프로젝트인 경우 브라우저가 받아와야 하는 파일의 용량이 매우 높아지게 됩니다. 이 불필요한 청크 파일들을 받아오는 것을 막기위해 코드 스플리팅이나 lazy loading과 같은 기술을 이용하지만 이 또한 이또한 시간을 투자해야되는 하나의 불편함으로 작용했습니다.

RSC가 도입되면 클라이언트 컴포넌트, 서버컴포넌트 두 종류로 분리가 됩니다.

```
*.client.js(jsx,ts,tsx) 클라이언트용 컴포넌트
*.server.js(jsx,ts,tsx) 서버용 컴포넌트
```

```js
// Note.server.js - Server Component

import db from "db.server"
// (A1) We import from NoteEditor.client.js - a Client Component.
import NoteEditor from "NoteEditor.client"

function Note(props) {
  const { id, isEditing } = props
  // (B) Can directly access server data sources during render, e.g. databases
  const note = db.posts.get(id)

  return (
    <div>
      <h1>{note.title}</h1>
      <section>{note.body}</section>
      {/* (A2) Dynamically render the editor only if necessary */}
      {isEditing ? <NoteEditor note={note} /> : null}
    </div>
  )
}
```

위는 리액트에서 제공하는 예시중 하나이다. 위에서와 같이, 서버 컴포넌트는 api형태의 방식을 쓰지 않고, 직접 DB에 접근하여 note데이터를 받아오고 있습니다. 이는 RSC의 장점중 하나입니다. 그리고 받아온 데이터를 바탕으로 NoteEditor라는 클라이언트 컴포넌트를 구성합니다. 여기서 NoteEditor는 클라이언트 컴포넌트인데, 서버 컴포넌트가 이 컴포넌트를 import할 때는, 자동적으로 필요로 할 때 dynamic하게 import를 하게 됩니다. 기존 방식처럼 직접 불편한 과정들을 거칠 필요가 없습니다.

## 요약

어떤 상황에서 어떤 컴포넌트를 써야할까?

- 서버 컴포넌트: 데이터를 받아오는 부분, 전처리 과정, 파일 시스템이 필요한 부분
- 클라이언트 컴포넌트: UI위주의 부분, 빠른 interaction이나 사용자의 입력이 필요한 부분

RSC의 장점은

- Zero Bundle-size Components
- 서버 컴포넌트는 번들에 포함되지 않기 때문에 브라우저로 가는 번들 사이즈가 현저하게 줄어든다.
- 서버에서만 사용되는 패키지 모듈들은 서버에서만 유지하면 된다.

Full Access to the Backend

- API 형식으로 불러올 필요 없이, 파일시스템, DB등에 편하게 접근할 수 있다. 물론 클라이언트 단에서 api를 통해 패치하는 방식도 가능하다.

```js
// PhotoRenderer.js
// NOTE: *before* Server Components

import React from "react"

// one of these will start loading *when rendered on the client*:
const OldPhotoRenderer = React.lazy(() => import("./OldPhotoRenderer.js"))
const NewPhotoRenderer = React.lazy(() => import("./NewPhotoRenderer.js"))

function Photo(props) {
  // Switch on feature flags, logged in/out, type of content, etc:
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />
  } else {
    return <PhotoRenderer {...props} />
  }
}
```

기존의 lazy loading 방식. 반드시 React.lazy를 통한 콜백으로 import를 감싸줘야 했다.

```js
import React from "react"

// one of these will start loading *once rendered and streamed to the client*:
import OldPhotoRenderer from "./OldPhotoRenderer.client.js"
import NewPhotoRenderer from "./NewPhotoRenderer.client.js"

function Photo(props) {
  // Switch on feature flags, logged in/out, type of content, etc:
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />
  } else {
    return <PhotoRenderer {...props} />
  }
}
```

client컴포넌트들은 자동적으로 코드 스플리팅이 적용되어 렌더링이 필요한 시점에 lazy하게 import된다.

No waterfalls

아마 여기서 얘기하는 waterfall이라는 것은,렌더링과 로딩이 한번에 진행되는것이 아니라, 렌더링 이후 데이터가 로딩이 되는 방식으로 인해 불필요하게 시간이 소요되는 것을 말하는 것 같습니다.

추가적으로 이 현상은 부모 -> 자식 계층적으로 데이터가 내려가는 컴포넌트 형태가 꾸려져 있을 때 더 악화됩니다.

```js
// Note.js
// NOTE: *before* Server Components

function Note(props) {
  const [note, setNote] = useState(null);
  useEffect(() => {
    // NOTE: loads *after* rendering, triggering waterfalls in children
    fetchNote(props.id).then(noteData => {
      setNote(noteData);
    });
  }, [props.id]);
  if (note == null) {
    return "Loading";
  } else {
    return (/* render note here... */);
  }
}
```

위 예시를 보면, useEffect의 디펜던시에 props가 있기 때문에, 이 Note 컴포넌트는 렌더링이 끝난 후 부모에서 전달해주는 props가 전부 로딩이 되고 난 이후에야 fetchNote를 통해 추가적으로 로딩을 시작하게 됩니다. 그리고 그 동안 UI에서는 과도하게 길게 'Loading'을 보여줘야하는 문제가 생깁니다.

이는 리액트에서 최근에 중점적으로 도입하려고 하는 Concurrent Mode, Suspense와도 연관된 문제입니다.

```js
// Note.server.js - Server Component

function Note(props) {
  // NOTE: loads *during* render, w low-latency data access on the server
  const note = db.notes.get(props.id);
  if (note == null) {
    // handle missing note
  }
  return (/* render note here... */);
}
```

위에서는 렌더링을 함과 동시에 데이터를 불러오기 때문에 렌더링 자체가 데이터를 가져오기를 시작하는 시간에 영향을 주지 않습니다. 자동적으로 성능이 향상이 되겠죠.
