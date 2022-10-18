# React Query Key 관리

![image](https://user-images.githubusercontent.com/63354527/196449458-1893d16d-6522-4433-8ea3-fa730cf28cdc.png)

queryKey는 React Query에서 아주 중요한 개념이다. 내부적으로 데이터를 캐시하고 쿼리에 대한 종속성이 변경될 때 자동으로 다시 가져올 수 있게 한다. 그리고 필요한 시점에 queryKey를 통해 query cache와 상호작용이 가능하다.

이하 queryKey는 작성 편의상 Key라 칭한다

## Caching Data

내부적으로 query cache는 Key가 직렬화되어 있고, Key는 해쉬되어 관리된다. 공식문서에는 아래와 같이 설명되어있다.

This means that no matter the order of keys in objects, all of the following queries are considered equal:

```ts
useQuery(['todos', { status, page }], ...)
useQuery(['todos', { page, status }], ...)
useQuery(['todos', { page, status, other: undefined }], ...)
```

The following query keys, however, are not equal. Array item order matters!

번역해보면 첫번째 예시는 “오브젝트의 키 순서와 관계없이 다음 쿼리는 모두 같은 쿼리로 취급한다.” 이고, 두 번째는 예시는 “다음 쿼리 키는 같지 않습니다. 배열의 요소 순서가 중요합니다!”

여기서 중요한 사실은 `Key가 쿼리에 대해 유니크 해야 한다는 것이고`, `React Query는 cache에 Key를 이용해 접근한다는 것이다.` 당연히 useQuery 및 useInfiniteQuery에 동일한 Key를 사용할 수 없으며, 결국 하나의 query cache만 유효하게 된다.

```ts
useQuery(["todos"], fetchTodos)

// 🚨 잘못된 사용
useInfiniteQuery(["todos"], fetchInfiniteTodos)

// ✅ 사용 가능
useInfiniteQuery(["infiniteTodos"], fetchInfiniteTodos)
```

## 자동 Refetch

> 쿼리는 선언형이다.

React Query를 처음 사용하는 사람들은 refetch에 대해 명령형으로 실행하고자 하는데 잘못된 방법이다. React Query에서 강조되는 아주 중요한 개념이다.

쿼리가 있고 데이터를 가져오고자 한다. 이제 버튼을 클릭하여 필터링 된 데이터를 다시 가져오고 싶지만 파라미터가 다르다. 일반적으로는 다음과 같이 작성한다.

```ts
function Component() {
  const { data, refetch } = useQuery(['todos'], fetchTodos)

  // ❓ 필터 정보를 넘길 수가 없다 ❓
  return <Filters onApply={() => refetch(???)} />
}
```

위 예시에서 필터된 데이터를 가져올 방법은 무엇일까? 정답은 불가능하다.  
refetching을 위한 것이지 데이터를 변경하기 위한 쿼리가 아니다.

데이터를 변경 하는 state가 있는 경우 Key가 변경 될 때 마다 React Query가 트리거 되어 자동으로 refetching하기 때문에 우리는 Key에 저장하기만 하면 된다. 필터를 적용하려면 state를 변경 시키면 된다.

```ts
function Component() {
  const [filters, setFilters] = React.useState()
  const { data } = useQuery(["todos", filters], fetchTodos)

  return <Filters onApply={setFilters} />
}
```

setFilters에 의해 발생한 리렌더링에 트리거 되어 다른 Key를 React Query에 전달하며 refetching한다.

이제 custom hook을 활용하여 fetch와 filter의 관심사를 분리하는 투두 코드를 살펴보자.

```ts
type State = "all" | "open" | "done"
type Todo = {
  id: number
  state: State
}
type Todos = ReadonlyArray<Todo>

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`)
  return response.data
}

export const useTodosQuery = (state: State) => {
  return useQuery(["todos", state], () => fetchTodos(state))
}

function Component() {
  const [filters, setFilters] = React.useState()
  const { data } = useTodosQuery(filters)

  return <Filters onApply={setFilters} />
}
```

## React Query의 Key 관리

### 배열으로 키 관리

문자열도 Key가 될 수 있겠지만 컨벤션을 맞추기 위해 항상 배열을 사용하는 것이 좋다. **React Query는 내부적으로 키를 배열으로 변환하기 때문에 결국 같은 동작이다.**

```ts
// 🚨 ['todos'] 으로 변환
useQuery("todos")
// ✅
useQuery(["todos"])
```

### 구조

투두리스트를 예로 들어보자. 필터링된 목록과 상세 정보 보기를 허용하는 투두리스트의 Key 구성하는 법은 다음과 같다.

```ts
{
  ['todos', 'list', { filters: 'all' }],
  ['todos', 'list', { filters: 'done' }],
  ['todos', 'detail', 1],
  ['todos', 'detail', 2],
}
```

위와 같은 구조를 사용하면 ['todos']에 대한 모든 정보를 invalidate 시킬 수 있으며, 특정 하나의 목록을 지정할 수 있다. 특히 모든 목록을 대상으로 Mutation Update를 지정 할 수 있어 훨씬 유연해 질 수 있다.

```ts
function useUpdateTitle() {
  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      // ✅ 투두 상세 정보 업데이트
      queryClient.setQueryData(["todos", "detail", newTodo.id], newTodo)

      // ✅ 업데이트 된 투두를 포함하는 모든 목록 업데이트
      queryClient.setQueriesData(["todos", "list"], (previous) =>
        previous.map((todo) => (todo.id === newTodo.id ? newtodo : todo))
      )
    },
  })
}
```

리스트 구조와 상세 정보의 구조가 다른 경우엔 모든 리스트를 invalidate 하면 된다.

```ts
function useUpdateTitle() {
  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      queryClient.setQueryData(["todos", "detail", newTodo.id], newTodo)

      // ✅ 모든 리스트 invalidate
      queryClient.invalidateQueries(["todos", "list"])
    },
  })
}
```

URL에 필터 정보가 담겨 있어 현재 어떤 목록이 있는지 정확히 알고 있다면 위 예제들을 결합하여 네트워크 비용을 아낄 수 있다.

```ts
function useUpdateTitle() {
  // 현재 URL의 필터 정보를 반환하는 훅스
  const { filters } = useFilterParams()

  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      queryClient.setQueryData(["todos", "detail", newTodo.id], newTodo)

      // ✅ 현재 사용중인 목록을 즉시 업데이트
      queryClient.setQueryData(["todos", "list", { filters }], (previous) =>
        previous.map((todo) => (todo.id === newTodo.id ? newtodo : todo))
      )
      // 리스트를 invalidate 시키지만 refetch 하지 않음
      queryClient.invalidateQueries({
        queryKey: ["todos", "list"],
        refetchActive: false,
      })
    },
  })
}
```

### 객체로 Key 관리

위의 예는 Key를 하드 코딩으로 작성했음을 알 수 있다. 휴먼 에러가 발생하기 쉬울 뿐만 아니라, Key를 수정해야 하는 경우 등 유지보수를 하기 어렵게 만든다.

그렇기 때문에 하나의 기능 당 하나의 Key를 객체로 관리하길 권장한다. Key를 생성하는 단순한 오브젝트이며 custom hook에서 사용 가능하다. 위 예제 구조의 경우 다음과 같다.

```ts
const todoKeys = {
  all: ["todos"] as const,
  lists: () => [...todoKeys.all, "list"] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, "detail"] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
}

// 🕺 모든 todos 삭제
queryClient.removeQueries(todoKeys.all)

// 🚀 모든 리스트 invalidate
queryClient.invalidateQueries(todoKeys.lists())

// 🙌 prefetch 하나의 todo
queryClient.prefetchQueries(todoKeys.detail(id), () => fetchTodo(id))
```
