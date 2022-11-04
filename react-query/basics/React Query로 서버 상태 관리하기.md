# React Query로 서버 상태 관리하기

React Query는 서버 상태(server state)를 관리하는 라이브러리이다. 서버 상태란 다음과 같다.

- 원격에 위치한 공간에 저장되며 앱이 소유하거나 제어하지 않는다.
- 데이터를 가져오고 업데이트하기 위해선 비동기 API가 필요하다.
- 다른 사람과 함께 사용하며, 내가 모르는 사이에 업데이트될 수 있다.
- 앱에서 사요하는 데이터가 "유효 기간이 지난" 상태가 될 가능성을 가진다.

예를 들어 쇼핑몰의 상품목록, 게시판의 댓글, 배달앱의 주문 상황 등은 모두 위와 같은 특성을 가지고 있다. 그렇기에 다음과 같은 작업에 대한 필요가 생긴다.

- 캐싱
- 서버 데이터 중복 호출 제거
- 만료된 데이터를 백그라운드에서 제거하기
- 데이터가 언제 만료되는지 알고 있기
- 만료된 데이터는 가능한 빨리 업데이트하기
- 페이지네이션, 레이지 로딩 데이터의 성능 최적화
- 서버 상태의 메모리 관리 및 가비지 콜렉션
- 쿼리 결과의 구조 공유를 통한 메모이제이션

이러한 기능들이 없어도 앱을 구현할 수 있다. 하지만 높은 품질의 앱을 위해서는 필요한 작업이라 할 수 있다. React Query는 위와 같은 기능을 사용할 수 있도록 도와준다.

## 도입할 필요가 있는가?

위에서 제공하는 기능만 본다면 사용하면 좋을 것 같다. 하지만 새로운 라이브러리의 도입은 항상 신중히 결정해야 한다.

### 상태 관리 라이브러리에서 요구하는 boilerplate 코드 제거

React Query를 사용한다면 서버 데이터 처리 방식을 바꿔야 한다. React에서는 보통 redux, mobx같은 상태 관리 라이브러리를 사용하며, 그것으로 서버 상태를 관리하는 방식이 일반적이다. Redux를 사용하고 있다면 redux-thunk, redux-observable, redux-saga 등의 미들웨어를 사용해 서버 데이터 요청 액션이 들어오면 API를 호출하여 redux 상태를 업데이트하는 방식을 사용한다.

```tsx
import { filter, map, mergeMap } from "rxjs/operators"
import { ajax } from "rxjs/ajax"

// redux-observable 예제
// SAMPLE_DATA_REQUEST 타입의 액션을 받아서 API 호출 후 리스펀스를 담은 액션으로 맵핑하는 epic
const sampleAsyncEpic = (action$) => {
  return action$.pipe(
    filter((action) => action.type === "SAMPLE_DATA_REQUEST"),
    mergeMap((action) =>
      ajax
        .getJSON(`https://api.server.com/samples`)
        .pipe(map((response) => ({ type: "SAMPLE_DATA_RESPONSE", response })))
    )
  )
}
```

필요에 따라 서버 데이터를 현재 컴포넌트에서 멀리 떨어진 컴포넌트 트리에 전달할 필요가 생길 수도 있다. 하지만 서버 상태는 그 데이터를 가져온 컴포넌트와 1~2단계 아래의 하위 컴포넌트에서 사용하는 경우가 대부분이다. 즉, 서버 데이터 처리와 관련된 redux 액션, 리유서, 미들웨어 코드를 작성할 필요가 없어진다. Redux는 훌륭한 상태관리 기능을 제공하지만, 동시에 많은 양의 boilderplate 코드라는 피로감도 제공하고 있기에 주목할 필요가 있다.

하지만 상태 관리 라이브러리를 사용해서 서버 데이터를 제어하는 쪽을 더 선호한다면 React Query는 도입하지 말거나 앱에서 꼭 필요한 부분에만 사용하는 편이 좋을 것이다.

### 캐싱 & 리프레쉬

채팅 앱처럼 소켓 통신을 사용한다면 서버 상태가 즉시 업데이트되겠지만 그렇지 않다면 주기적으로 업데이트 해주는 기능을 직접 구현해야한다. React Query는 `useQuery`훅의 파라미터를 통해 API 데이터의 만료 시간, 리프레쉬 간격, 데이터를 캐시에서 유지할 기간, 브라우저 포커스시 데이터 리프레쉬 여부, 성공 or 에러 콜백등 다양한 기능을 제어할 수 있다.

예를 들어 앱안에 게시판이 있다고 가정하자. 다음과 같은 시나리오가 가능하다.

- 게시글 필터에서 사용하는 옵션은 서버에서 변경될 가능성이 낮으므로 만료 시간을 무한(Infinity)으로 설정하여 API 추가 호출을 방지할 수 있다.
- 게시글 목록의 만료 시간을 1분으로 설정하여 유저가 페이지 번호를 1에서 2로 반복해서 바꾸는 행동을 취하더라도 API 중복 호출을 방지할 수 있다.
- 게시글 목록의 만료시간이 1분으로 설정되어 있는데 어떤 사용자는 게시글을 그 안에 작성하거나 수정할 수도 있다. 그래서 게시글 작성 후에는 캐시를 강제로 무효화(invalidate)하여 목록을 새로고침한다.
- 사용자가 게시글의 제목을 수정한 후 목록으로 돌아갔을 때 API 호출을 통해 게시글 목록을 서버에서 다시 가져온 후에야 수정 사항이 반영되었음을 확인해줄 수 있다. 하지만 React Query에서는 수정 성공시 캐시되어있는 게시글 제목을 임시로 변경하여 사용자에게 서버에 다시 요청한 게시글 목록의 응답이 즉시 온 것처럼 보이게 만들 수 있다.
  - 게시글 수정 목록 새로고침에 딜레이가 전혀 없는 듯한 사용자 경험을 제공할 수 있다.
  - 실제 API 호출 및 데이터 업데이트는 백그라운드에서 진행되며 에러 발생시 원래 데이터로 복구시킬 수 있다.
  - Optimistic update 참조.
- 브라우저의 다른 탭을 보다가 다시 열었을 때 게시글 목록을 자동으로 불러오게 할 수 있다.

캐싱 관련 기능은 처음 언급한대로 구현되어 있지 않아도 앱 사용하는 데는 문제가 없다. 하지만 클라이언트에서 사용하는 서버 데이터의 종류와 양이 늘어나고 서버에서 관리하고 있는 데이터의 양도 늘어난다면 양쪽 모두의 작업 처리량을 줄일 필요가 자연스럽게 생긴다. 또 더 좋은 사용자 경험을 구현하는데도 도움을 준다.

## React Query 사용 방법

React Query를 통해 관리하는 쿼리 데이터는 라이프사이클에 따라 fetching, fresh, stale, inactive, delete 상태를 가진다.

- fetching: 요청 중인 쿼리
- fresh: 만료되지 않은 쿼리. 컴포넌트가 마운트, 업데이트되어도 데이터를 다시 요청하지 않는다.
- stale: 만료된 쿼리. 컴포넌트가 마운트, 업데이트되면 데이터를 다시 요청한다.
- inactive: 사용하지 않는 쿼리. 일정 시간이 지나면 가비지 컬렉터가 캐시에서 제거한다.
- delete: 가비지 컬렉터에 의해 캐시에서 제거된 쿼리

## important defaults

다음은 React Query에서 제공하는 API 기본이 되는 설정이다.

- `useQuery`로 가져온 데이터는 기본적으로 stale 상태가 된다.
  - `staleTime` 옵션으로 데이터가 stale 상태로 바뀌는데 걸리는 시간을 늘릴 수 있다.
- stale 쿼리는 다음 경우에 백그라운드에서 다시 가져온다.
  - 새로운 쿼리 인스턴스가 마운트되었을 때
  - 브라우저 윈도우가 다시 포커스되었을 때
  - 네트워크가 다시 연결되었을 때
  - refetchInterval 옵션이 있을 때
- 활성화된 `useQuery`, `useInfiniteQuery` 인스턴스가 없는 쿼리 결과는 "inactive" 라벨이 붙으며 다음에 사용될 때까지 남아있는다.
- 백그라운드에서 3회 이상 실패한 쿼리는 에러 처리된다.
  - retry 옵션으로 쿼리 함수에서 오류 발생시 재시도할 횟수, retryDelay 옵션으로 재시도 대기 시간을 설정
- 쿼리 결과는 memoization을 위해 structural sharing을 사용하며 데이터 reference는 변경되지 않는다.
  - immutable.js에서 사용하는 기술([참고할만한 글](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2))
  - 99.9% 케이스에서는이 옵션을 끌 필요가 없음.
  - structural sharing은 JSON 호환 데이터에만 적용되며, 다른 타입의 쿼리 결과는 항상 변경되었다고 판단한다.

## QueryClientProvider 설정

React Query는 캐시를 관리하기 위해 QueryClient 인스턴스를 사용한다. 컴포넌트가 useQuery 훅 안에서 QueryClient 인스턴스에 접근할 수 있도록 QueryClientProvider를 컴포넌트 트리 상위에 추가해줘야 한다.

```tsx
import { QueryClient, QueryClientProvider } from "react-query"

const queryClient = new QueryClient() // 인스턴스 생성

function App() {
  return <QueryClientProvider client={queryClient}>...</QueryClientProvider>
}
```

## useQuery로 서버 데이터 가져오기

서버에서 데이터를 가져오고 캐싱을 하는데 사용하는 기본이며 가장 많이 사용하게 되는 훅이다.

```ts
const { data, isLoading } = useQuery(queryKey, queryFunction, options)
```

queryFunction에는 서버에서 데이터를 요청하고 Promise를 리턴하는 함수를 전달한다. 즉 axios.get(...), fetch(...) 등을 리턴하는 함수.

queryKey 에는 문자열과 배열을 넣을 수 있다. 쿼리 키가 가지는 유연함이 곧 캐싱을 처리를 쉽게 만들어준다. 쿼리 키가 다르면 캐싱도 별도로 관리하기 때문이다.

예를 들어 todo 라는 키는 아래와 같이 확장할 수 있다.

```ts
// 다른 키로 취급한다.
useQuery(['todo', 1], ...)
useQuery(['todo', 2], ...)
// 객체 필드의 값이 달라도 다른 키로 취급한다
useQuery(['todo', { preview: true }], ...)
useQuery(['todo', { preview: false }], ...)
// 객체 필드의 순서가 달라도 내용이 같으면 같은 키로 취급한다
useQuery(['todo', { preview: true, status: 'done' }], ...)
useQuery(['todo', { status: 'done', preview: true }], ...)
```

useQuery 훅이 리턴하는 데이터와 옵션의 종류는 매우 다양하다.

- 리턴 데이터
  - data: 쿼리 함수가 리턴한 Promise에서 resolve된 데이터
  - isLoading: 저장된 캐시가 없는 상태에서 데이터를 요청중일 때 true
  - isFetching: 캐시가 있거나 없거나 데이터를 요청중일 때 true
- 옵션
  - cacheTime: unused 또는 inactive 캐시 데이터가 메모리에서 유지될 시간. 기본값은 5분이며 설정한 시간을 초과하면 메모리에서 제거
    - Infinity로 설정하면 쿼리 데이터는 캐시에서 제거되지 않음
  - staleTime: 쿼리 데이터가 fresh에서 stale로 전환되는데 걸리는 시간. 기본값은 0이다.
    - Infinity로 설정하면 쿼리 데이터는 직접 무효화할 때까지 fresh 상태로 유지
    - 캐시는 메모리에서 관리되므로 브라우저 새로고침 후에는 다시 가져온다.
  - enabled: false값이 전달되면 쿼리가 비활성화된다.
    - 데이터 요청에 사용할 파라미터가 유효한 값일 때만 true를 할당하는 식으로 활용할 수 있다.
  - onSuccess: 쿼리함수가 성공적으로 데이터를 가져왔을 때 호출되는 함수
  - onError: 쿼리 함수에서 오류가 발생했을 때
  - onSettled: 쿼리 함수의 성공, 실패 두 경우 모두 실행된다.
  - keepPreviousData: 쿼리 키(ex.페이지 번호)가 변경되어서 새로운 데이터를 요청하는 동안에도 마지막 data값을 유지한다.
    - 페이지네이션을 구현할 때 유용하다. 캐시되지 않은 페이지를 가져올 때 화면에서 목록이 사라지는 깜빡임 현상을 방지할 수 있다.
    - isPreviousData 값으로 현재 쿼리키에 해당하는 값인지 확인할 수 있다.
  - initialData: 캐시된 데이터가 없을 때 표시할 초기값. placeholder로 전달한 데이터와 달리 캐싱이된다. 브라우저 로컬 스토리지에 저장해둔 값으로 데이터를 초기화할 때 사용할 수 있을 것이다.
  - refetchOnWindowFocus: 윈도우가 다시 포커스되었을 때 데이터를 호출할 것인지 여부. 기본값은 true이므로 필요없다고 판단되면 끄면된다.

## useMutation으로 서버 데이터 업데이트

서버 데이터를 가져오는 것은 reactive하게 동작하는 useQuery를 사용하면 되겠지만, 서버 데이터 업데이트는 그런 방식으로 사용하기에는 적절하지 않다. 데이터를 생성/수정/삭제에는 `useMutation` 훅을 사용하면 된다.

```tsx
const mutation = useMutation((newTodo) => axios.post("/todos", newTodo))

const handleSubmit = useCallback(
  (newTodo) => {
    mutation.mutate(newTodo)
  },
  [mutation]
)
```

useQuery의 옵션처럼 `onSuccess`, `onError`, `onSettled` 콜백을 전달할 수 있으며 거기에 더해 mutate를 호출했을 때 실행할 `onMutate` 콜백도 사용할 수 있다.

Redux를 사용한다면 리퀘스트 성공 액션을 미들웨어에서 확인하여 추가 액션을 실행할 것입니다. `useMutation`을 사용한다면 `onSuccess` 콜백을 사용해도 되지만, 코드 가독성을 위해 `mutateAsync`를 사용할 수 있다.

```tsx
const mutation = useMutation((newTodo) => axios.post("/todos", newTodo))

const handleSubmit = useCallback(
  async (newTodo) => {
    await mutation.mutateAsync(newTodo)
    setAnotherState()
    dispatch(createAnotherAction())
  },
  [mutation]
)
```

## 쿼리 무효화(Invalidation)

쿼리 데이터가 stale 상태로 바뀌기만을 기다릴 수만은 없는 케이스가 있다. 예를 들어 게시글에 댓글을 작성한 후에는 서버에서 댓글 목록을 다시 가져올 필요가 있다. 이와 같은 경우 지정한 staleTime이 지나기 전에 직접 쿼리를 무효화해서 데이터를 새로 가져오도록 해야한다.

```ts
const queryClient = useQueryClient()

// 캐시에 있는 모든 쿼리를 무효화한다.
queryClient.invalidateQueries()

// todo로 시작하는 모든 쿼리를 무효화한다. ex) ['todos', 1], ['todos', 2], ...
queryClient.invalidateQueries("todos")

// ['todos', 1] 키를 가진 쿼리를 무효화한다.
queryClient.invalidateQueries(["todos", 1])
```

`predicate` 옵션을 사용하면 무효화할 쿼리를 더 자세하게 설정할 수 있다.

```ts
// 쿼리 키 배열의 두번째 객체의 version 필드의 값이 10 이상인 쿼리만 무효화한다.
queryClient.invalidateQueries({
  predicate: (query) =>
    query.queryKey[0] === "todos" && query.queryKey[1]?.version >= 10,
})

// 위의 코드로 무효화된다.
const todoListQuery = useQuery(["todos", { version: 20 }], fetchTodoList)

// 위의 코드로 무효화되지 않는다.
const todoListQuery = useQuery(["todos", { version: 5 }], fetchTodoList)
```
