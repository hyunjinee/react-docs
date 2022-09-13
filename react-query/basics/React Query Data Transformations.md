# React Query Data Transformations

우리 대부분은 GraphQL을 사용하지 않습니다. 만약 사용하고 있다면 당신은 원하는 형식으로 데이터를 요청할 수 있기 때문에 행복할 것입니다.

하지만 REST로 작업하는 경우 백엔드가 반환하는 내용에 제약을 받습니다. 그렇다면 React Query
로 작업할 때 데이터를 가장 잘 변환하는 방법과 어디에서 변환을 해야할까요? 소프트 웨어 개발에서 가치가 있는 유일한 대답은 여기에도 적용됩니다.

> It depends. - Every developer, always

다음은 각각의 장단점이 있는 데이터를 변환할 수 있는 위치에 대한 `3 + 1 접근 방식`입니다.

## 0. On the backend

제가 가장 좋아하는 방법입니다. 백엔드가 정확히 우리가 원하는 구조로 데이터를 반환하면 우리가 해야할 일은 없습니다. 많은 경우에 이것이 비현실적으로 들릴 수 있지만(예를 들어 public REST API) 엔터프라이즈 애플리케이션에서는 가능합니다. 백엔드를 제어하고 정확한 사용 사례에 대한 데이터를 반환하는 엔드포인트가 있는 경우 예상한 방식으로 데이터를 전달하는 것을 선호합니다.

- 🟢 프론트엔드에서 일 안함.
- 🔴 항상 가능하진 않음.

## 1. In the queryFn

queryFn은 useQuery에 전달하는 함수입니다. 이 함수는 여러분이 작성한 함수가 Promise를 반환할 것으로 예상하고 결과 데이터는 쿼리 캐시에 저장됩니다. 그러나 백엔드가 여기에서 제공하는 구조로 데이터를 절대적으로 반환해야 한다는 의미는 아닙니다. 그렇게 하기전에 변환할 수 있습니다.

```ts
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get("todos")
  const data: Todos = response.data

  return data.map((todo) => todo.name.toUpperCase()) // 데이터 변경
}

export const useTodosQuery = () => useQuery(["todos"], fetchTodos)
```

프론트엔드에서는 이 데이터를 백엔드에서 원래 변형된 구조로 백엔드가 반환한 것처럼 작업할 수 있습니다. 이렇게 하면 여러분은 원래 백엔드가 반환한 데이터 구조에 접근할 수 없습니다. react-query-devtools를 보면 변형된 구조를 볼 수 있고, 네트워크 트레이스를 보면 원래 구조를 볼 수 있습니다.

또한 react-query가 여기서 수행할 수 있는 최적화는 없습니다. 가져오기가 실행될 때마다 변환이 실행됩니다. 만약 그 변환과정이 고비용 계산이라면 다른 대안 중 하나를 고려해야합니다. 일부 회사에서는 데이터 fetching을 추상화하는 공유 API 계층도 있으므로 변환을 수행하기 위해 이 계층에 엑세스하지 못할 수도 있습니다.

정리하면 다음과 같습니다.

- 🟡 변환된 구조는 캐시에 저장되므로 원래 구조에 엑세스할 수 없습니다.
- 🔴 모든 fetch 요청에 대해 변환을 수행합니다.
- 🔴 자유롭게 수정할 수 없는 공유 API 레이어가 있는 경우 불가능

## 2. In the render function

```ts
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get("todos")
  return response.data
}

export const useTodosQuery = () => {
  const queryInfo = useQuery(["todos"], fetchTodos)

  return {
    ...queryInfo,
    data: queryInfo.data?.map((todo) => todo.name.toUpperCase()),
  }
}
```

이것은 fetch가 실행될 때마다 실행될 뿐 아니라 실제로 모든 렌더링(데이터 가져오기를 포함하지 않는 렌더링 포함)에서 실행됩니다. 이것이 전혀 문제가 되지 않을 수 있지만 문제가 있는 경우 `useMemo`를 사용하여 최적화할 수 있습니다. 이때 종속성을 가능한 한 좁게 정의하도록 주의해야합니다. queryInfo 내부의 데이터는 실제로 변경되지 않는 한 참조적으로 안정적이지만 queryInfo 자체는 그렇지 않습니다. queryInfo를 종속성으로 추가하면 변환이 모든 렌더링에서 다시 실행됩니다.

```ts
export const useTodosQuery = () => {
  const queryInfo = useQuery(["todos"], fetchTodos)

  return {
    ...queryInfo,
    // 🚨 don't do this - the useMemo does nothing at all here!
    data: React.useMemo(
      () => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
      [queryInfo]
    ),

    // ✅ correctly memoizes by queryInfo.data
    data: React.useMemo(
      () => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
      [queryInfo.data]
    ),
  }
}
```

특히 사용자 정의 훅에 데이터 변환과 결합할 추가 로직이 있는 경우 이는 좋은 옵션입니다. 데이터는 잠재적으로 정의되지 않을 수 있으므로 작업할 때 optional chaining을 사용하십시오.

정리하면 다음과 같습니다.

- 🟢 useMemo로 최적화 가능
- 🟡 정확한 구조는 devtools에서 검사할 수 없음
- 🔴 조금 더 복잡한 구문
- 🔴 데이터는 잠재적으로 정의되지 않을 수 있음

## 3. using the select option

v3에서는 데이터 변환에도 사용할 수 있는 내장 선택기를 도입했습니다.

```ts
export const useTodosQuery = () =>
  useQuery(["todos"], fetchTodos, {
    select: (data) => data.map((todo) => todo.name.toUpperCase()),
  })
```

선택자는 데이터가 존재하는 경우에만 호출되므로 여기에서 undefined에 대해서 신경쓸 필요가 없습니다. 위와 같은 선택자는 함수의 identity가 변경되기 때문에 매 렌더링마다 실행됩니다.(인라인 함수와 비슷함.)

변환에 비용이 많이 든다면 `useCallback`을 사용하거나 안정적인 함수 참조로 추출하여 메모화할 수 있습니다.

```ts
// select-memoizations
const transformTodoNames = (data: Todos) =>
  data.map((todo) => todo.name.toUpperCase())

export const useTodosQuery = () =>
  useQuery(["todos"], fetchTodos, {
    // ✅ uses a stable function reference
    select: transformTodoNames,
  })

export const useTodosQuery = () =>
  useQuery(["todos"], fetchTodos, {
    // ✅ memoizes with useCallback
    select: React.useCallback(
      (data: Todos) => data.map((todo) => todo.name.toUpperCase()),
      []
    ),
  })
```

또한 선택 옵션을 사용하여 데이터의 일부만 구독할 수도 있습니다. 이것이 이 접근 방식을 독특하게 만드는 이유입니다. 다음 예를 고려해보겠습니다.

```ts
export const useTodosQuery = (select) =>
  useQuery(["todos"], fetchTodos, { select })

export const useTodosCount = () => useTodosQuery((data) => data.length)
export const useTodo = (id) =>
  useTodosQuery((data) => data.find((todo) => todo.id === id))
```

여기에서 사용자 지정 선택기를 useTodosQuery에 전달하여 [useSelector](https://react-redux.js.org/api/hooks#useselector)와 비슷한 API를 만들었습니다. 사용자 지정 훅은 여전히 이전과 같이 작동합니다. select 함수를 전달하지 않으면 select가 정의되지 않아 전체 상태가 반환되기 때문입니다. 그러나 선택자 함수를 전달하면 이제 선택자 함수의 결과만 구독할 수 있습니다. 이것은 꽤나 큰 의미가 있는데, 우리가 todo의 이름을 업데이트하더라도 우리의 컴포넌트중 todo의 count에 대해서만 구독한 컴포넌트(useTodosCount 사용)는 리렌더링되지 않습니다. 개수는 변경되지 않으므로 react-query는 업데이트에 대해 이 관잘자에게 알리지 않도록 선택할 수 있습니다.(여기서 이것은 약간 단순화되었으며 기술적으로 완전히 사실이 아님을 유의하십시오. 렌더링 최적화에 대해서는 3부에서 자세히 설명합니다.)

- 🟢 best optimizations
- 🟢 부분 구독을 가능하게 해줌
- 🟡 모든 관찰자마다 구조가 다를 수 있음
- 🟡 구조적 공유는 두 번 수행됩니다.(3부에서 이에 대해 더 자세히 설명합니다.)
