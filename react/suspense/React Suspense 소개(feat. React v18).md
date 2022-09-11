# React Suspense 소개 (feat. React v18)

## Suspense란?

Suspense는 2018년에 첫 시연되어 React 커뮤니티에서 큰 반향을 일으킨 후, React v16.6에 실험적인(experimental)기능으로 추가되었다. 많은 리액트 개발자들이 Suspense가 React의 정식 기능이 되기를 목이 빠지게 기다렸던 걸로 아는데 드디어 React v18.0에서 추가되었다.

Suspense라는 React의 신기술을 사용하면 컴포넌트의 렌더링을 어떤 작업이 끝날 때까지 잠시 중단시키고 다른 컴포넌트를 먼저 렌더링할 수 있다. 이 작업이 꼭 어떠한 작업이 되어야 한다는 특별한 제약 사항은 없지만 아무래도 REST API나 GraphQL을 호출하여 네트워크를 통해 비동기로 데이터를 가져오는 작업을 가장 먼저 떠오르게 된다.

비동기로 데이터를 읽어오는 것은 예전에 클래스로 컴포넌트를 작성하던 시절부터 훅(hook)을 사용하는 요즘까지도 항상 필요한 일이지만 React로 직접 구현하기에는 까다로운 면이 있다. 그래서 일반적으로 데이터 로딩(data loading)을 전문으로 하는 라이브러리나 프레임워크에서 제공하는 데이터 로더에 의존하는 경우가 많다.

Suspense는 어떤 컴포넌트가 읽어야 하는 데이터가 아직 준비가 되지 않았다고 리액트에게 알려주는 새로운 메커니즘이다. Suspense를 통해 컴포넌트가 비동기 데이터를 읽어오는 방법을 표준화하고자 리액트 팀의 장기적인 계획도 엿볼 수 있다. Suspense는 얼핏 보기에는 작은 아이디어처럼 보이지만 개인적으로 앞으로 리액트 개발 패러다임을 바꿀 정도로 파급력이 큰 기능이라고 생각한다.

## 기본 문법

기본적으로 리액트는 JSX 코드 안에 들어있는 모든 컴포넌트를 즉시 호출하여 바로 렌더링을 진행한다.

예를 들어, 리액트는 다음과 같이 UserList 컴포넌트가 포함된 JSX 코드를 렌더링할 때, UserList 함수를 바로 호출할 것이다.

```jsx
<UserList />
```

하지만 컴포넌트를 아래와 같이 Suspense로 감싸주면 컴포넌트의 렌더링은 특정 작업 이후로 미루고, 그 작업이 끝날 때 까지는 fallback 속성으로 넘긴 컴포넌트를 대신 보여줄 수 있다.

```jsx
<Suspense fallback={<Spinner />}>
  <UserList />
</Suspense>
```

물론 컴포넌트가 렌더링 되기 전에 구체적으로 어떤 작업이 일어나야 하는지는 UserList 함수안에 명시되어 있을 것이다.

## Suspense 사용 전

먼저 그 동안 우리가 리액트 비동기 데이터를 읽어와야하는 컴포넌트를 어떻게 작성했는지 되돌아보자.

아마도 현재 가장 흔하게 사용되는 방법은 클래스 기반 컴포넌트를 사용할 때는 생명주기 함수인 `componentDidMount()` 구현하는 것이고, 함수형 컴포넌트를 사용할 때는 `useEffect()` 훅 함수를 호출하는 것이다. API를 호출하여 네트워크를 통해 데이터를 가져오는 처리는 컴포넌트에서 발생할 수 있는 대표적인 Side Effect이기 때문이다.

간단한 예로 useEffect() 훅으로 API를 호출하여 가져온 사용자의 글목록을 보여주기 위한 전형적인 함수형 컴포넌트를 작성해보자.
(공개된 REST API 서비스인 JSONPlaceholder를 사용하였고, 로딩 컴포넌트를 유관으로 확인할 수 있도록 3초의 지연시간을 주었다.)

먼저 최상위 컴포넌트인 Main은 단순하게 제목을 표시하고 User 컴포넌트에 userId prop을 "1"로 넘겨주고 있다.

```jsx
import User from "./User"

function Main() {
  return (
    <main>
      <h2>Suspense 미사용</h2>
      <User userId="1" />
    </main>
  )
}

export default Main
```

부모 컴포넌트인 User는 API를 호출하여 가져온 데이터에서 사용자 이름과 이메일을 추출하여 보여주고 있다.

```jsx
import { useState, useEffect } from "react"
import Posts from "./Posts"

function User({ userId }) {
  const [loading, setLoading] = useState(true)
  const [user, setUser] = useState([])

  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then((response) => response.json())
      .then((user) => {
        setTimeout(() => {
          setUser(user)
          setLoading(false)
        }, 3000)
      })
  })

  if (loading) return <p>사용자 정보 로딩중...</p>
  return (
    <div>
      <p>{user.name} 님이 작성한 글</p>
      <Posts userId={userId} />
    </div>
  )
}

export default User
```

자식 컴포넌트인 Posts도 역시 API를 호출하여 가져온 데이터에서 글 아이디와 글 제목을 추출하여 보여주고 있다.

```jsx
import { useState, useEffect } from "react"

function Posts({ userId }) {
  const [loading, setLoading] = useState(true)
  const [posts, setPosts] = useState([])

  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/posts?userId=${userId}`)
      .then((response) => response.json())
      .then((posts) => {
        setTimeout(() => {
          setPosts(posts)
          setLoading(false)
        }, 3000)
      })
  })

  if (loading) return <p>글목록 로딩중...</p>
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          {post.id}. {post.title}
        </li>
      ))}
    </ul>
  )
}

export default Posts
```

이 두개의 컴포넌트는 공통적으로 크게 두 가지 역할을 담당하고 있는데, 첫 번째는 비동기로 API를 호출하여 원격에 있는 데이터를 가져오는 것이고, 두번째는 데이터 수신 상태에 따라 알맞은 UI를 제공하는 것이다.

React에서 이처럼 비동기 데이터를 읽어오는 컴포넌트를 작성하면 몇가지 고질적인 문제가 발생한다. 우선 최종 사용자(end user) 경험 측면에서 UI가 마치 폭포(waterfall)처럼 순차적으로 나타나는 현상이 일어날 수 있다. 이 waterfall 현상은 특히 한 페이지 상의 여러 컴포넌트에서 동시에 비동기 데이터를 읽어오는 경우 자주 발생하는데 상위 컴포넌트의 데이터 로딩이 끝나야지만 하위 컴포넌트의 데이터 로딩이 시작될 수 있기 때문에 주로 발생하게 된다.

뿐만 아니라 이렇게 초기 렌더링 후에 데이터 로딩 후 다시 렌더링을 수행하는 방법은 경쟁 상태(race conditions)에도 취약한 것으로 알려져있다. 비동기 통신은 반드시 요청한 순서대로 데이터가 응답된다는 보장이 없기 때문에 의도치 않게 싱크가 맞지 않는 데이터를 제공할 수도 있다.

마지막으로 개발 측면에서도 이렇게 if 조건문을 사용하여 어떤 컴포넌트를 보여줄지를 제어하는 것은 명령형(imperative) 코드에 가깝기 때문에 선언적(declarative) 코드를 지향하는 React의 기본 방향성과 맞지 않는다. 기본적으로 데이터 로딩과 UI 렌더링이라는 두 가지 전혀 다른 목표가 하나의 컴포넌트안에 커플링(coupling)되어 코드가 읽기가 어려워지고 테스트를 작성하기도 힘들어진다.

## Suspense 사용 후

동일한 코드를 이번에는 Suspense를 이용해서 재작성해보자.
먼저 API를 호출하여 비동기로 데이터를 가져오는 코드를 별도의 함수로 빼내자.

```js
function fetchUser(userId) {
  let user = null
  const suspender = fetch(
    `https://jsonplaceholder.typicode.com/users/${userId}`
  )
    .then((response) => response.json())
    .then((data) => {
      setTimeout(() => {
        user = data
      }, 3000)
    })
  return {
    read() {
      if (user === null) {
        throw suspender
      } else {
        return user
      }
    },
  }
}

function fetchPosts(userId) {
  let posts = null
  const suspender = fetch(
    `https://jsonplaceholder.typicode.com/posts?userId=${userId}`
  )
    .then((response) => response.json())
    .then((data) => {
      setTimeout(() => {
        posts = data
      }, 3000)
    })
  return {
    read() {
      if (posts === null) {
        throw suspender
      } else {
        return posts
      }
    },
  }
}

function fetchData(userId) {
  return {
    user: fetchUser(userId),
    posts: fetchPosts(userId),
  }
}

export default fetchData
```

이 함수는 컴포넌트에서 필요한 데이터를 제공하는 user와 posts 속성을 담고 있는 객체를 반환하는데 read() 함수는 데이터 수신중에는 suspender 변수에 저장되어 있는 API를 호출하는 코드를 반환하고, 데이터 수신이 완료되면 데이터를 반환한다.

이제 Main 컴포넌트 안에서 User 컴포넌트를 Suspense 컴포넌트로 감싸준다. 기존에 User 컴포넌트 안에 있던 로딩 시 보여줄 컴포넌트가 fallback 속성으로 넘어간다. 그리고 User 컴포넌트에는 prop으로 사용자 아이디 대신에 데이터를 가져오기 위함 함수의 호출이 사용된다.

```js
import { Suspense } from "react"
import User from "./User"
import fetchData from "./fetchData"

function Main() {
  return (
    <main>
      <h2>Suspense 사용</h2>
      <Suspense fallback={<p>사용자 정보 로딩중...</p>}>
        <User resource={fetchData("1")} />
      </Suspense>
    </main>
  )
}

export default Main
```

이제 User 컴포넌트 안에서 prop 으로 넘어온 resource로 부터 사용자 데이터를 읽어 올 수 있다. 그리고 Post 컴포넌트를 사용할 때 마찬가지로 Suspense로 감싸줍니다.

```js
import React, { Suspense } from "react"
import Posts from "./Posts"

function User({ resource }) {
  const user = resource.user.read()

  return (
    <div>
      <p>
        {user.name}({user.email}) 님이 작성한 글
      </p>
      <Suspense fallback={<p>글목록 로딩중...</p>}>
        <Posts resource={resource} />
      </Suspense>
    </div>
  )
}

export default User
```

Posts 컴포넌트 안에서도 마찬가지로 resource로 부터 글목록 데이터를 읽어올 수 있다.

```js
function Posts({ resource }) {
  const posts = resource.posts.read()

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          {post.id}. {post.title}
        </li>
      ))}
    </ul>
  )
}

export default Posts
```

이렇게 코드를 변경해주면 User 컴포넌트와 Post 컴포넌트 간의 waterfall 현상이 사라지고 거의 동시에 화면에 나타나는 것을 확인할 수 있.

코드 측면에서도 데이터 로딩과 UI 렌더링이 완전히 분리되어 코드 가독성과 유지 보수성이 향상된다.
