# Status Checks in React Query

React Query의 한가지 장점은 쿼리의 상태 필드에 쉽게 엑세스 할 수 있다는 것 입니다. 쿼리가 로드 중인지 아니면 오류가 있는지 즉시 알 수 있습니다. 이를 위해 라이브러리는 대부분 내부 상태 시스템에서 파생된 많은 boolean 플래그를 노출합니다. 타입을 살펴보면 쿼리가 다음 상태 중 하나일 수 있습니다.

- `success`: 쿼리가 성공했으며 이에 대한 데이터가 있습니다.
- `error`: 쿼리가 작동하지 않았으며 오류가 설정되었습니다.
- `loading`: 쿼리에 데이터가 없으며 현재 처음으로 로드 중 입니다.
- `idle`: 쿼리가 활성화되지 않았기 때문에 실행된 적이 없습니다.

> Updated: React Query v4에서 idle 상태는 사라졌습니다.

`isFetching` 플래그는 내부 상태 머신의 일부가 아닙니다. 요청이 진행 중일 때마다 true가 되는 추가 플래그 입니다. fetching하고 success가 될 수도 있고 fetching하고 error상태가 될 수도 있습니다. 하지만 loading과 success를 동시에 수행할 수는 없습니다. 상태 머신은 이를 확인합니다.

## The standard example

```ts
const todos = useTodos()

if (todos.isLoading) {
  return "Loading..."
}
if (todos.error) {
  return "An error has occurred: " + todos.error.message
}

return <div>{todos.data.map(renderTodo)}</div>
```

여기에서는 먼저 로드 및 오류를 확인한 다음 데이터를 표시합니다. 이것은 일부 사용 사례에서는 괜찮지만 다른 경우에는 그렇지 않을 수 있습니다. 많은 데이터 가져오기 솔루션, 특히 직접 만든 솔루션에는 refetch 메커니즘이 없거나 명시적인 사용자 상호 작용 시에만 refetch 합니다.

하지만 React Query는 가능합니다.

기본적으로 매우 적극적으로 refetch하며 사용자가 refetch 요청을 하지 않고 수행합니다. refetchOnMount, refetchOnWindowFocus, refetchOnReconnect의 개념들은 데이터를 정확하게 유지하는데 유용하지만 이러한 자동 refetch가 실패하면 혼란스러운 ux가 발생할 수 있습니다.

## Background errors

많은 상황에서 백그라운드 다시 가져오기가 실패하면 자동으로 무시될 수 있습니다. 그러나 위의 코드는 그렇게 하지 않습니다. 두 가지 예를 살펴보겠습니다.

- 사용자가 페이지를 열고 초기 쿼리가 성공적으로 로드됩니다. 사용자는 한동안 페이지에서 작업하다가 브라우저 탭을 전환하여 이메일을 확인합니다. 몇분 후 다시 돌아오고 React Query는 background refetch를 수행합니다. 이 때 fetch가 실패합니다.
- 사용자는 목록 보기가 있는 페이지에 있으며 하나의 항목을 클릭하여 세부 정보 보기로 들어갑니다. 이것은 잘 작동하므로 목록 보기로 돌아갑니다. 다시 세부 정보로 이동하면 캐시의 데이터가 표시됩니다. 이는 background refetch가 실패하는 경우를 제외하고는 훌륭합니다.

두 가지 상황 모두에서 쿼리는 다음과 같은 상태가 됩니다.

```json
{
  "status": "error",
  "error": { "message": "Something went wrong" },
  "data": [{ ... }]
}
```

보시다시피 오류와 오래된 데이터를 모두 사용할 수 있습니다. 이것이 React Query를 훌륭하게 만드는 것 입니다. React Query는 stale-whire-revalidate 캐싱 메커니즘을 수용합니다. 즉, 데이터가 존재하는 경우 데이터가 오래된 경우에도 항상 데이터를 제공합니다.

이제 우리가 무엇을 표시할지 결정하는 것은 우리에게 달려 있습니다. 오류를 보여주는게 중요합니까? 오래된 데이터가 있는 경우에만 표시하는 것으로 충분합니까? 아니면 background 오류 표시기를 사용해 둘 다 표시해야합니까?

이 질문에 대한 명확한 답은 없습니다. 단지 사용자의 사용 사례에 따라 다릅니다. 그러나 위의 두가지 예를 고려할 때 데이터가 오류 화면으로 대체된다면 다소 혼란스러운 사용자 경험이 될 것이라고 생각합니다.

이것은 React Query가 쿼리가 실패했을 때 기본적으로 세번 시도하는 것과 관련있습니다. 따라서 stale한 데이터가 error screen으로 대체되는데는 몇 초가 걸릴 수 있습니다. 또한 background에서 fetching이 일어나는 것을 표시해주지 않는다면 당혹스러울 수 있습니다.

이것이 제가 일반적으로 데이터 가용성을 먼저 확인하는 이유입니다.

```ts
const todos = useTodos()

if (todos.data) {
  return <div>{todos.data.map(renderTodo)}</div>
}
if (todos.error) {
  return "An error has occurred: " + todos.error.message
}

return "Loading..."
```

다시 말하지만, 유스케이스에 크게 의존하기 때문에 무엇이 옳은지에 대한 명확한 원칙은 없습니다. 모든 분들이 공격적인 리패칭이 가져오는 결과를 알고 있어야하며 그에 따라 코드를 구성해야합니다.
