# Practical React Query

2018년에 GraphQL, 특히 `Apollo Client`가 유명해졌을 때, Redux를 완전히 대체하는 것에 대해 많은 소란이 있었고, "Redux는 죽었나요?"라는 질문이 많았습니다.

저는 이것이 무엇에 관한 것인지 이해하지 못한 것을 분명히 기억합니다. `"왜 데이터 fetching 라이브러리가 Redux와 같은 전역 상태 관리자를 대체하는 것일까? 이 둘이 무슨 상관일까?"`

저는 Apollo와 같은 GraphQL 클라이언트는 REST API에서의 axios와 같이 데이터만 가져올 것이라는 인상을 받았으며, 여전히 애플리케이션에서 해당 데이터에 엑세스 할 수 있는 방법이 필요하다고 생각했습니다.

**저는 완전히 틀렸습니다.**

## Client State vs Server State

Apollo가 제공하는 것은 원하는 데이터를 설명할 수 있는 방법(어떤 데이터를 원하는지 선언)과 그 데이터를 가져오는 기능뿐만 아니라, 해당 서버 데이터에 대한 캐시도 함께 제공됩니다. 이 의미는 여러 컴포넌트에서 같은 `useQuery`라는 훅을 썼을 때 데이터를 한 번만 가져온 다음 캐시에서 반환한다는 뜻입니다.

이것은 우리가 주로 Redux를 사용하는 용도와 매우 유사하게 들립니다. 우리가 Redxu를 사용하는 용도는 다음과 같습니다.

**서버로 부터 데이터를 가져와 전역에서 사용가능하게 한다.**

따라서 우리는 항상 이 서버 상태를 클라이언트 상태처럼 취급해왔습니다. 애플리케이션은 서버 상태를 소유하지 않습니다. 애플리케이션은 단지 화면에 최신 버전의 데이터를 표시하기위해 데이터를 빌린 것입니다. 데이터를 소유하는 것은 서버입니다.

저에게 위 사실은 데이터에 대해 생각하는 방식의 패러다임을 바꿔놓았습니다. 캐시를 활용하여 우리가 소유하지 않는 데이터를 표시할 수 있다면 전체 앱에서 사용할 수 있어야 하는 실제 클라이언트 상태가 별로 남지 않습니다.(캐시를 활용해 기존 전역 상태를 제거합니다.) 이 개념은 저에게 Apollo가 Redux를 대체할 수 있다고 생각하게 한 이유를 이해하게 되었습니다.

## React Query

저는 GraphQL을 사용해보지 않았습니다. 우리는 기존 REST API를 잘 사용하고 있으며, over-fetching과 같은 문제를 많이 경험하지 않습니다. 분명히, 우리가 GraphQL로 전환하기에 충분한 pain point가 없습니다. 특히 GraphQL을 사용하면 백엔드도 변경해야 하기 때문에 전환하기 쉽지 않습니다.

저는 로드 및 오류 상태를 포함하여 프론트엔드에서 데이터 가져오기가 단순해지는 것이 부러웠습니다. React에서 REST API를 사용할 때도 비슷한 것이 있었다면...

**그게 바로 [React Query](https://tanstack.com/query/v4/?from=reactQueryV3&original=https://react-query-v3.tanstack.com/)입니다.**

React Query는 [Tanner Linsley](https://github.com/tannerlinsley)에 의해 2019년에 만들어졌으며, React Query는 Apollo의 좋은 부분을 REST로 가져옵니다. React Query는 Promise를 반환하고 stale-while-revalidate 캐싱 전략을 수용하는 모든 함수와 함께 작동합니다.

이 라이브러리는 기본 설정 값으로도 데이터를 가능한 한 최신 상태로 유지하는 동시에 가능한 한 빨리 사용자에게 데이터를 표시하여 훌룡한 UX를 제공합니다. 또한 매우 유연하며 기본값이 충분하지 않을 때 다양한 설정을 사용자가 지정할 수 있습니다.

이 글은 React Query에 대한 소개가 아닙니다. 저는 공식문서가 Guides & Concept을 설명하는데 훌륭하다고 생각합니다. 다양한 Talk에서 볼 수 있는 [비디오가](https://tanstack.com/query/v4/docs/videos?from=reactQueryV3&original=https://react-query-v3.tanstack.com/videos) 있으며 Tanner에는 라이브러리에 익숙해지고 싶다면 들을 수 있는 React [Query Essentails](https://learn.tanstack.com/) 코스가 있습니다.

저는 문서 넘어에 있는 더 실용적인 팁들에 집중하고 싶습니다. 이 팁들은 제가 회사에서 라이브러리를 사용했을 때 뿐만 아니라 Discord 및 Github 토론에서 질문에 답변하면서 React Query 커뮤니티에 참여했을 때 지난 몇달 동안 수집한 것들 입니다.

## The Defaults explained

저는 React Query의 기본값들이 매우 잘 선택되었다고 생각하지만, 특히 처음에는 때때로 당신을 당황하게 할 수 있습니다.

우선, React Query는 기본 staleTime이 0인 경우에도 다시 렌더링할 때마다 queryFn을 호출하지 않습니다. 앱은 언제든지 다양한 이유로 다시 렌더링 될 수 있으므로 매번 가져오는 것은 미친 짓 입니다.

> Always code for re-renders, and a lot of them. I like to call it render resiliency.(항상 리렌더링을 위한 코딩을 하세요. 저는 그것을 렌더링 탄력성이라고 부르고 싶습니다.) - Tanner Linsley

예상하지 못한 refetch를 본다면, 당신이 방금 창에 초점을 맞추고 React Query가 `refetchOnWindowFocus`를 수행하고 있기 때문일 수 있습니다.(이 기능은 production에서는 훌륭한 기능입니다.) 사용자가 다른 브라우저 탭으로 이동했다가 앱으로 돌아오면 background refetch가 자동으로 실행되고, 그 동안 서버에서 변경 사항이 있으면 화면에 데이터가 업데이트됩니다.

이 모든 것은 로딩 스피너가 표시되지 않고 발생하며 데이터가 현재 캐시에 있는 것과 동일한 경우 컴포넌트가 리렌더링되지 않습니다.

개발중에는 refetch 현상이 더 빈번하게 발생하는데, DevTools와 애플리케이션을 왔다갔다하는 행위만으로도 refetch가 발생하기 때문입니다. 이 현상에 주의해야합니다.

다음으로, `cacheTime`과 `staleTime` 사이에 약간의 혼동이 있는 것 같으므로 이를 정리해보겠습니다.

- StaleTime: 쿼리가 최신 상태에서 신선하지 않은 상태로 전환될 때까지의 기간입니다. 쿼리가 최신 상태인한, 데이터는 항상 캐시에서만 읽힙니다. 네트워크 요청은 발생하지 않습니다. 쿼리가 오래된 경우(기본 값은 즉시 stale), 캐시에서 데이터를 계속 가져오지만 [특정 조건](https://tanstack.com/query/v4/docs/guides/caching?from=reactQueryV3&original=https://react-query-v3.tanstack.com/guides/caching)에서 backgroud refetch가 발생할 수 있습니다.
- CacheTime: 비활성화된 쿼리가 캐시에서 제거될 때까지의 기간입니다. 기본값은 5분입니다. 등록된 observer(관찰자)가 없는 쿼리는 즉시 비활성 상태로 전환되므로 해당 쿼리를 사용하는 모든 구성 요소가 마운트 해제됩니다.

대부분의 경우 이러한 설정 중 하나를 변경하려면 staleTime을 조정해야합니다. 저는 cacheTime을 조작할 필요가 거의 없었습니다. 공식 문서에도 [cacheTime에 대한 설명](https://tanstack.com/query/v4/docs/guides/caching?from=reactQueryV3&original=https://react-query-v3.tanstack.com/guides/caching#a-detailed-caching-example)이 자세히 나와있습니다.

## Use the React Query DevTools

DevTools는 쿼리의 상태를 이해하는 데 큰 도움이 됩니다. 또한 DevTools는 현재 캐시에 있는 데이터를 알려주므로 디버깅 시간이 단축됩니다. 그 외에도 개발 서버는 일반적으로 빠르기 때문에 background refetch를 더 잘 인식하려면 브라우저 DevTools에서 네트워크 연결을 조절하는 것이 도움이 됩니다.

## Treat the query key like a dependency array

저는 아마 여러분이 친숙할 useEffect 훅의 dependency array와 비슷한 query key와 관련해 말해보겠습니다.

왜 이 둘은 비슷할까요?

React Query는 쿼리 키가 변경될 때마다 다시 refetch를 수행하기 때문입니다. 따라서 queryFn에 변수 매개변수를 전달할 때 항상 해당 값이 변경될 때 fetch를 수행합니다. 수동으로 refetch를 트리거하기 위해 복잡한 효과를 조정하는 대신 쿼리키를 사용할 수 있습니다.

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

export const useTodosQuery = (state: State) =>
  useQuery(["todos", state], () => fetchTodos(state))
```

여기에서 우리의 UI가 필터 옵션과 함께 할 일 목록을 표시한다고 상상해보겠습니다. 해당 필터링을 저장할 로컬 상태가 있고 사용자가 선택을 변경하는 즉시 해당 로컬 상태를 업데이트하고 쿼리 키가 변경되기 때문에 React Query가 자동으로 refetch를 트리거합니다. 따라서 우리는 사용자의 필터 선택을 쿼리 함수와 동기화하도록 유지하고 있습니다. 이는 종속성 배열이 useEffect에 대해 나타내는 것과 매우 유사합니다.

## A new cache entry

쿼리 키가 캐시의 키로 사용되기 때문에 'all'에서 'done'으로 전환하면 새 캐시 항목이 표시되며, 만약 상태를 처음 바꾼 것이라면 로딩 상태로 바뀔 것이다.(아마 로딩 스피너를 보여줌) 이것은 확실히 이상적이지 않으므로 이러한 경우에 `keepPreviousData` 옵션을 사용하거나 가능하면 새로 생성된 캐시 항목을 [initialData](https://tanstack.com/query/v4/docs/guides/initial-query-data?from=reactQueryV3&original=https://react-query-v3.tanstack.com/guides/initial-query-data#initial-data-from-cache)로 미리 채울 수 있습니다. 위 예는 할 일에 대해 클라이언트 측 사전 필터링을 수행할 수 있기 때문에 완벽합니다.

```ts
// pre-filtering
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

export const useTodosQuery = (state: State) =>
  useQuery(["todos", state], () => fetchTodos(state), {
    initialData: () => {
      const allTodos = queryClient.getQueryData<Todos>(["todos", "all"])
      const filteredData =
        allTodos?.filter((todo) => todo.state === state) ?? []

      return filteredData.length > 0 ? filteredData : undefined
    },
  })
```

이제 사용자가 상태를 전환할 때마다 아직 데이터가 없으면 '모든 할일' 캐시의 데이터로 미리 채웁니다. 사용자에게 '완료된' 할 일을 즉시 표시할 수 있으며 백그라운드 fetching이 완료되면 업데이트된 목록이 계속 표시됩니다. React Query v3 이전에는 백그라운드 fetching을 트리거하기 위해 initialStale 속성도 설정해야합니다.

저는 이것이 단지 몇줄의 코드를 작성하는 것으로 훌륭한 UX 개선을 불러올 수 있다고 생각합니다.

## Keep server and client state seperate

useQuery에서 데이터를 가져오는 경우 해당 데이터를 로컬 상태에 두지 마십시오.

이는 Form을 위한 기본 값들을 가져오고 그 기본값들로 Form을 렌더링할 때는 상관없습니다. 백그라운드 업데이트는 새로운 것을 생성할 가능성이 거의 없으며 그렇다 하더라도 당신의 Form은 이미 초기화되어있을 것 입니다. 따라서 의도적으로 그렇게 하는 경우 staleTime을 설정하여 불필요한 백그라운드 다시 가져오기를 실행하지 않도록 하십시오.

```ts
const App = () => {
  const { data } = useQuery('key', queryFn, { staleTime: Infinity })

  return data ? <MyForm initialData={data} /> : null
}

const MyForm = ({ initialData} ) => {
  const [data, setData] = React.useState(initialData)
  ...
}
```

이 컨셉은 사용자가 편집할 수 있도록 하려는 데이터를 표시할 때 수행하기가 조금 더 어렵지만 많은 이점이 있습니다.

## The enabled option is very powerful

useQuery 훅에는 사용자가 정의할 수 있는 많은 옵션들이 있으며, enabled option은 쿼리의 실행 시점을 설정할 수 있게 해줍니다.

- [Dependent Queries(의존 쿼리)](https://tanstack.com/query/v4/docs/guides/dependent-queries?from=reactQueryV3&original=https://react-query-v3.tanstack.com/guides/dependent-queries): 한 쿼리에서 데이터를 가져오고 첫번째 쿼리에서 데이터를 성공적으로 얻은 후에만 두 번째 쿼리를 실행합니다.
- Turn queries on and off: refetchInterval 덕분에 정기적으로 데이터를 폴링하는 쿼리가 하나 있지만 모달이 열려 있으면 화면 뒷면의 업데이트를 피하기 위해 일시적으로 중지할 수 있습니다.
- Wait for user input: 쿼리 키에 일부 필터 기준이 있지만 사용자 필터를 적용하지 않는 한 비활성화하십시오.
- Disable a query after some user input

## Don't use the queryCache as a local state manager

queryCache(queryClient.setQueryData)를 변조하는 경우 optimistic 업데이트 또는 mutation 후 백엔드에서 받은 데이터를 쓰기 위한 것 입니다. 모든 백그라운드 refetch가 해당 데이터를 재정의 할 수 있으므로 로컬 상태에 대해 [다른 것을 사용하십시오](https://zustand-demo.pmnd.rs/).

## Create Custom Hooks

하나의 useQuery 호출을 래핑하기 위한 것일지라도 사용자 지정 훅을 만드는 것은 다음과 같은 이유로 효과가 있습니다.

- 실제 데이터 fetching 로직을 ui 밖으로 뺄 수 있습니다.
- 하나의 쿼리키의 모든 사용을 하나의 파일에 보관할 수 있습니다.(type 정의 포함)
- 일부 설정을 조정하거나 데이터 변환을 추가해야하는 경우 한 곳에서 수행할 수 있습니다.
