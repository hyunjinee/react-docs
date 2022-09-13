# React Query Render Optimizations

React Query는 이미 매우 우수한 최적화와 기본설정을 제공하며 대부분의 경우 더 이상의 최적화가 필요하지 않습니다. 불필요한 리렌더링은 많은 사람들이 많은 초점을 맞추는 경향이 있는 주제입니다. 그것이 제가 그것을 다루기로 결정한 이유입니다. 그러나 다시 한 번 지적하고 싶습니다. 일반적으로 대부분의 앱에서 렌더링 최적화는 생각만큼 중요하지 않습니다. 앱을 리렌더링하는 것은 좋은 일입니다. 리렌더링은 앱이 최신 상태가 될 수 있도록 해줍니다. 저는 항상 "그곳에 꼭 있어야하는 누락된 렌더링"보다 "불필요한 렌더링" 택합니다. 이 토픽에 대한 내용은 다음 글을 읽어주세요.

- [Fix the slow render before you fix ther re-render](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render)
- [this article by @ryanflorence about premature optimizations](https://reacttraining.com/blog/react-inline-functions-and-performance/)

## isFetching transition

```ts
export const useTodosQuery = (select) =>
  useQuery(["todos"], fetchTodos, { select })
export const useTodosCount = () => useTodosQuery((data) => data.length)

function TodosCount() {
  const todosCount = useTodosCount()

  return <div>{todosCount.data}</div>
}
```

저는 이전 글에서 이 컴포넌트는 todos의 길이가 변경되는 경우에만 다시 렌더링 된다고 말했을 때 완전히 정직하지는 않았습니다. 백그라운드 refetch를 수행할 때마다 이 구성요소는 다음 쿼리 정보로 두 번 다시 렌더링 됩니다.

```ts
{ status: 'success', data: 2, isFetching: true }
{ status: 'success', data: 2, isFetching: false }
```

React Query는 각 쿼리에 대한 많은 메타 정보를 노출하고 isFetching이 그 중 하나이기 때문입니다. 이 플래그는 요청이 진행 중일 때 항상 true입니다. 이것은 백그라운드 로딩 표시기를 표시하려는 경우에 매우 유용합니다. 하지만 그렇게 하지 않는다면 그것도 좀 불필요합니다.

## notifyOnChangeProps

React Query에는 notifyOnChangeProps 옵션이 존재합니다. React Query에 알리기 위해 관찰자별 수준에서 설정할 수 있습니다. 이러한 props 중 하나가 변경되는 경우에만 변경 사항에 대해 관찰자에게 알릴 수 있습니다. 이 옵션을 ['data']로 설정한다면 우리가 찾는 최적화된 버전을 찾을 수 있습니다.

```ts
export const useTodosQuery = (select, notifyOnChangeProps) =>
  useQuery(["todos"], fetchTodos, { select, notifyOnChangeProps })
export const useTodosCount = () =>
  useTodosQuery((data) => data.length, ["data"])
```

문서의 [optimistic-updates-typescript](https://github.com/TanStack/query/blob/9023b0d1f01567161a8c13da5d8d551a324d6c23/examples/optimistic-updates-typescript/pages/index.tsx#L35-L48) 예제에서 작동하는 것을 볼 수 있습니다.

## staying in sync

위 코드는 매우 잘 동작하지만 매우 쉽게 동기화되지 않을 수 있습니다. 오류에도 대응하고 싶다면 어떻게 해야 할까요? 아니면 isLoading 플래그를 사용하기 시작할까요? 우리는 컴포넌트에서 실제로 사용하는 필드와 동기화된 notifyOnChangeProps 목록을 유지해야합니다. 그렇게 하는 것을 잊어버리고 데이터 속성만 관찰했지만 표시되는 오류가 발생하면 구성 요소가 다시 렌더링되지 않으므로 최신 데이터를 갖지 않습니다. 훅은 구성요소가 실제로 무엇을 사용할지 모르기 때문에 사용자 정의 훅에서 이것을 하드 코딩하면 문제가 됩니다.

```ts
export const useTodosCount = () =>
  useTodosQuery((data) => data.length, ["data"])

function TodosCount() {
  // 🚨 we are using error, but we are not getting notified if error changes!
  const { error, data } = useTodosCount()

  return (
    <div>
      {error ? error : null}
      {data ? data : null}
    </div>
  )
}
```

제가 이 글의 첫 부분에서 언급했듯이, 저는 가끔 불필요하게 다시 렌더링하는 것보다 아예 렌더링 되지 않는 것을 훨씬 더 나쁘다고 생각합니다. 물론 옵션을 사용자 정의 훅으로 전달할 수 있지만, 여전히 수동적이고 boilerplate가 많습니다. 이 작업을 자동으로 수행할 수 있습니다.

## Tracked Queries

이 기능이 제 첫 번째 주요 기여였다는 점을 감안할 때 매우 자랑스럽습니다. notifyOnChangeProps를 'tracked'로 설정하면 React Query는 렌더링 중에 사용 중인 필드를 추적하고 이를 사용하여 목록을 계산합니다. 이것은 목록에 대해 생각할 필요가 없다는 점을 제외하고는 목록을 수동으로 지정하는 것과 정확히 같은 방식으로 최적화됩니다.

이 기능을 모든 쿼리에 대해 전역적으로 켤 수도 있습니다.

```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      notifyOnChangeProps: "tracked",
    },
  },
})
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

이를 통해 다시 렌더링에 대해 생각할 필요가 없습니다. 물론 사용량을 추적하는 것도 약간의 오버헤드가 있으므로 현명하게 사용해야 합니다. 추적된 쿼리에는 몇 가지 제한 사항도 있습니다. 이것이 opt-in 기능인 이유입니다.

- 만약 여러분이 객체를 비구조화할당 한다면, 여러분은 효율적으로 모든 필드를 관찰하고 있는 것 입니다. 일반적인 구조할당은 괜찮지만 다음과 같이는 하지마세요.

```ts
// 🚨 will track all fields
const { isLoading, ...queryInfo } = useQuery(...)

// ✅ this is totally fine
const { isLoading, data } = useQuery(...)
```

- 추적된 쿼리는 "렌더링 중"에만 작동합니다. 아래와 같이 useEffect 중에 필드에 접근하면 추적되지 않습니다. Dependency Array에 넣어주어야 접근 가능합니다.

```ts
const queryInfo = useQuery(...)

// 🚨 will not corectly track data
React.useEffect(() => {
    console.log(queryInfo.data)
})

// ✅ fine because the dependency array is accessed during render
React.useEffect(() => {
    console.log(queryInfo.data)
}, [queryInfo.data])
```

- 추적된 쿼리는 각 렌더링에서 리셋되지 않으므로 필드를 한 번 추적하면 관찰자의 수명 동안 추적하게 됩니다.

```ts
const queryInfo = useQuery(...)

if (someCondition()) {
    // 🟡 we will track the data field if someCondition was true in any previous render cycle
    return <div>{queryInfo.data}</div>
}
```

## Structural sharing

React Query가 기본적으로 활성화한 렌더링 최적화는 구조적 공유입니다. 이 기능을 사용하면 모든 수준에서 데이터의 참조 정체성을 유지할 수 있습니다. 예를 들어 다음과 같은 데이터 구조가 있다고 가정해봅시다.

```json
[
  { "id": 1, "name": "Learn React", "status": "active" },
  { "id": 2, "name": "Learn React Query", "status": "todo" }
]
```

이제 우리가 첫번째 Todo를 done 상태로 바꾸고 background refetch를 실행했다고 가정해보겠습니다. 그러면 우리는 백엔드로 부터 다음과 같은 새로운 json 데이터를 받을 것 입니다.

```json
[
-  { "id": 1, "name": "Learn React", "status": "active" },
+  { "id": 1, "name": "Learn React", "status": "done" },
  { "id": 2, "name": "Learn React Query", "status": "todo" }
]
```

이제 React Query는 이전 상태와 새 상태를 비교하고 가능한 한 많은 이전 상태를 유지하려고 시도합니다. 우리의 예제에서는 todo를 업데이트 했기 때문에 todo배열이 새 것입니다.

id 1의 객체도 새 것이지만 id 2의 객체는 이전 상태의 객체와 동일한 참조가 됩니다. React Query는 아무 것도 변경되지 않았기 때문에 새 결과에 복사합니다.

이것은 구독 선택자를 사용할 때 매우 편리합니다.

```ts
// ✅ will only re-render if _something_ within todo with id:2 changes
// thanks to structural sharing
const { data } = useTodo(2)
```

이전에 암시했듯이 선택자의 경우 구조적 공유가 두번 수행됩니다. 한 번은 queryFn에서 반환된 결과에 대해 변경되어 변경된 사항이 있는지 확인한 다음 선택자 함수의 결과에 대해 한 번 더 수행됩니다. 매우 큰 데이터셋이 있는 경우 구조적 공유가 병목 현상이 될 수 있습니다. 또한 json 직렬화가 가능한 데이터에서만 작동합니다.

이 최적화가 필요하지 않으면 모든 쿼리에서 structureSharing: false를 설정하여 끌 수 있습니다.

내부에서 일어나는 일에 대해 자세히 알고 싶다면 [replaceEqaulDeep](https://github.com/TanStack/query/blob/80cecef22c3e088d6cd9f8fbc5cd9e2c0aab962f/src/core/tests/utils.test.tsx#L97-L304) 테스트를 살펴보세요.
