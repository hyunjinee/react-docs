# How and Why You Should Use React Query

React 애플리케이션을 만들때 직면하는 문제중 하나는 서버에서 데이터를 가져오기 위한 코드 패턴을 정하는 것입니다.
React에서 데이터 가져오기를 처리하는 가장 일반적인 방법은 전역 상태를 통해 fetch operation의 상태를 정의하는 것입니다.

아래 예제를 보죠.

```js
import React, { useState, useEffect } from "react"
import axios from "axios"
// regular fetch with axios
function App() {
  const [isLoading, setLoading] = useState(false)
  const [isError, setError] = useState(false)
  const [data, setData] = useState({})

  useEffect(() => {
    const fetchData = async () => {
      setError(false)
      setLoading(true)

      try {
        const response = await axios("http://swapi.dev/api/people/1/")

        setData(response.data)
      } catch (error) {
        setError(true)
      }
      setLoading(false)
    }
    fetchData()
  }, [])
  return (
    <div className="App">
      <h1>React Query example with Star Wars API</h1>
      {isError && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <pre>{JSON.stringify(data, null, 2)}</pre>
      )}
    </div>
  )
}
export default App
```

위의 코드는 useState, useEffect와 같은 훅이 모두 필요하고 세가지 다른 상태를 사용하여 데이터를 저장하고 애플리케이션이 데이터를 가져오고 있는지 또는 API에서 이미 오류 응답이 있는지 확인합니다. 이 패턴은 대부분의 애플리케이션 데이터 가져오기 논리에 대해 계속해서 반복됩니다.

이뿐만 아니라 리액트에서 데이터를 불러오는 과정에서 종종 다음 문제들을 겪습니다.

- 데이터가 모든 어플리케이션 인스턴스에서 공유되며 다른 사용자에 의해 임의로 변경될 수 있습니다.
- 데이터의 최신 여부를 보장할 수 없으며 항상 갱신이 필요하다.
- 최적화된 요청 작업을 위해 기존의 캐시를 무효화하는 등 캐시를 조작할 수 있어야합니다.

마지막으로, 테마나 사이드바 설정같은 사용자 설정을 관리하는 로컬 상태와 API로부터 불러온 데이터를 관리하는 원격 상태를 함께 사용할 때에도 문제가 생깁니다.

```js
//what global state commonly look like nowadays
const state = {
  theme: "light",
  sidebar: "off",
  followers: [],
  following: [],
  userProfile: {},
  messages: [],
  todos: [],
}
```

만약 원격 상태와 로컬 상태를 분리할 수 있다면 더 좋지 않을까요? 또 데이터를 불러오기 위해 3개의 상태를 관리하는 대신, 더 간결한 코드를 작성할 수 있다면 어떨까요?

이를 해결하기위한 첫번째 방법은 데이터를 불러오고 관리하기 위한 커스텀 훅을 만드는 것입니다. 이는 매우 유용한 방법으로, Bit(Github)와 같은 컴포넌트 허브(컴포넌트허브는 재사용 가능한 컴포넌트들을 업로드해 공유하는 사이트를 의미)에서 커스텀 훅을 찾거나 공유해 사용할 수도 있습니다.

또다른 방법은 React Query 입니다. 이 라이브러리는 데이터 페칭, 동기화, 업데이트, 캐싱을 도와주면서 두개의 훅과 하나의 유틸리티 함수로 작성할 코드 수를 줄여줍니다.

새로운 항목을 추가할 때느 useMutation 훅이 비동기 함수의 실행 결과를 관측할 것 입니다. 요청에 성공한 후 이전 쿼리를 무효화 (invalidate)할 수도 있으며, React Query는 동일한 키로 쿼리를 다시 불러옵니다.

## useQuery hook

useQuery 훅은 데이터를 불러오는 코드를 React Query 라이브러리에 등록하기 위해 사용합니다. useQuery는 키와 데이터를 불러오는 비동기 함수를 인자로 받아 어플리케이션의 현재 상태를 나타내는 다양한 값을 반환합니다.

위 예제를 react-query를 사용하여 다시 작성하겠습니다.

```js
import React from "react"
import axios from "axios"
import { useQuery } from "react-query"
// react-query fetch with axios
function App() {
  const { isLoading, error, data } = useQuery("fetchLuke", () =>
    axios("http://swapi.dev/api/people/1/")
  )
  return (
    <div className="App">
      <h1>React Query example with star wars API</h1>
      {error && <div>Something went wrong ...</div>}

      {isLoading ? (
        <div>Retrieving Luke Skywalker Information ...</div>
      ) : (
        <pre>{JSON.stringify(data, null, 2)}</pre>
      )}
    </div>
  )
}
export default App
```

늘 사용하던 useState와 useEffect훅을 사용하지 않습니다. 왜냐하면 useQuery에는 애플리케이션에서 활용할 수 있는 다양한 값(로딩상태, 에러응답, 반환된 데이터등)들이 이미 포함되어있기 때문입니다.

## useMutation

useMutation 훅은 원격 데이터의 생성/업데이트/삭제에 주로 사용됩니다.

```js
{
  onSuccess: () => {
    // 쿼리 무효화
    queryCache.invalidateQueries('todos')
    setText('')
  },
}
```

queryCache.invalidateQueries 는 그 특정 키를 갖는 캐시를 무효화시키며, React Query가 데이터를 다시 불러오도록 합니다.

## queryCache 유틸리티

불러온 데이터는 staleTime옵션을 설정하지 않았다면 저절로 최신의 상태를 잃게 됩니다.
queryCache는 쿼리의 추가적인 조작을 위한 다양한 함수를 포함하고 있는 유틸리티 객체로, 예제 코드에서 새로운 요청을 보내 할일 목록을 불러오기 위해 queryCache.incalidateQueries 함수르 ㄹ사용했던 것처럼 queryCache로 할 수 있는 모든 동작을 확인할 수 있습니다.

## 결론

리액트 쿼리는 원격 데이터를 전역 상태로 관리해야만 했던 문제를 해결할 수 있는 완벽한 훅 라이브러리입니다. 사용자가 데이터를 불러올 곳을 라이브러리에 알려주기만 하면, 캐싱, 백그라운드 업데이트, 오래된 데이터 교체까지 어떠한 추가 코드나 설정도 없이 해결합니다.
