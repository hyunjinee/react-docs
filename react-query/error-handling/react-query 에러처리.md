# react-query 에러 처리

> 리액트 쿼리는 비동기 상태 관리 라이브러리이다.

오류 처리는 비동기적 데이터 가져오기 작업의 필수적인 부분이다.
모든 요청이 성공하는 것은 아니며 모든 프로미스가 fulfilled되는 것도 아니다.

에러를 처리하는 방법에 대해 생각하지 않으면 사용자 경험에 부정적인 영향을 미칠 수 있다.

## 사전 지식

react-query로 오류를 올바르게 처리하기 위해 rejected된 Promise가 필요하다.
이는 axios같은 라이브러리로 작업할 때 바로 얻을 수 있다.

기본 fetch api와 같이 4xx또는 5xx와 같은 잘못된 상태 코드에 대해 rejected 프라미스를 제공하지 않는 API 또는 기타 라이브러리로 작업하는 경우 queryFn에서 직접 변환을 수행해야한다.

react-query를 사용하다보면 상태에 따라 컴포넌트에 여러개의 return문이 생기는 상황이 발생한다.

```js
function TodoList() {
  const todos = useQuery(["todos"], fetchTodos)

  if (todos.isLoading) {
    return "Loading..."
  }

  // ✅ standard error handling
  // could also check for: todos.status === 'error'
  if (todos.isError) {
    return "An error occurred"
  }

  return (
    <div>
      {todos.data.map((todo) => (
        <Todo key={todo.id} {...todo} />
      ))}
    </div>
  )
}
```

react-query의 isError boolean 플래그를 확인하여 오류 상황을 처리한다. 이것은 일부 시나리오에서는 확실히 괜찮지만 몇가지 단점도 있다.

1. 백그라운드 오류를 잘 처리하지 못한다. -> [Status Checks in React Query](https://tkdodo.eu/blog/status-checks-in-react-query)
2. 쿼리를 사용하는 컴포넌트마다 보일러 플레이트가 쌓인다.

두번째 문제를 해결하기 위해 React에서 이를 선언적으로 해결할 수 있다. (Suspense, Error Boundary)

컴포넌트입장에서 데이터를 fetching 할 때 매번 렌더링을 해야하는 컴포넌트가 달라지고 이는 결국 한 컴포넌트에서 return 이 상태별로 존재하는 상황으로 귀결된다.

## Error Boundaries

에러 바운더리는 렌더링 중에 발생하는 런타임 오류를 포착해 대체 UI를 표시하기 위한 컴포넌트입니다.
우리가 원하는 크기의 컴포넌트 단위를 래핑할 수 있으므로 나머지 UI가 해당 오류의 영향을 받지 않기 때문에 좋습니다.

react-query에서 Error Boundary가 작동하도록 하기 위해 라이브러리는 내부적으로 오류를 포착하고, 다음 렌더링 주기에서 에러 바운더리가 이를 캐치할 수 있도록 다시 던집니다.

react-query에서 위 작업을 수행하려면 useErrorBoundary 플래그를 쿼리에 전달하거나 기본 설정을 제공하기만 하면 된다.

```js
function TodoList() {
  // ✅ 모든 fetching 오류를 가장 가까운 에러바운더리로 전파합니다.
  const todos = useQuery(["todos"], fetchTodos, { useErrorBoundary: true })

  if (todos.data) {
    return (
      <div>
        {todos.data.map((todo) => (
          <Todo key={todo.id} {...todo} />
        ))}
      </div>
    )
  }

  return "Loading..."
}
```

v3.23.0 부터는 useErrorBoundary에 함수를 제공하여 어떤 오류는 Error Boundary로 내보내고 어떤 오류는 로컬에서 처리할지 선택할 수 있습니다.

```js
useQuery(["todos"], fetchTodos, {
  // 🚀 only server errors will go to the Error Boundary
  useErrorBoundary: (error) => error.response?.status >= 500,
})
```

위 방법은 mutation에도 적용되며 양식 제출을 수행할 때 매우 유용합니다.
4xx 범위의 오류는 로컬에서 처리할 수 있으며 모든 5xx 서버 오류는 에러 바운더리로 전파될 수 있습니다.

## 오류 알림 표시

일부 사용 사례의 경우 화면에 경고 배너를 렌더링하는 대신 오류 토스트 알림을 표시하는 것이 더 나을 수 있습니다.
아래는 일반적으로 react-hot-toast에서 제공하는 것과 같은 명령형 API입니다.

```js
import toast from 'react-hot-toast'

toast.error('Something went wrong')​
```

## The onError Callback

```js
const useTodos = () =>
  useQuery(["todos"], fetchTodos, {
    // ⚠️ looks good, but is maybe _not_ what you want
    onError: (error) => toast.error(`Something went wrong: ${error.message}`),
  })
```

언뜻 onError 콜백은 가져오기가 실패한 경우 사이드 이펙트를 수행하는데 필요한 것과 정확히 같으며 사용자 정의 훅 호출 하나당 한번 호출됩니다.

즉 useQuery의 onError 콜백은 모든 Observer에 대해 호출됩니다.
즉, 애플리케이션에서 useTodos를 두번 호출하면 하나의 네트워크 요청만 실패하더라도 두개의 오류 알림을 받게된다.

```js
const useTodos = () => {
  const todos = useQuery(['todos'], fetchTodos)

  // 🚨 effects are executed for every component
  // that uses this custom hook individually
  React.useEffect(() => {
    if (todos.error) {
      toast.error(`Something went wrong: ${todos.error.message}`)
    }
  }, [todos.error])

  return todos
}​
```
