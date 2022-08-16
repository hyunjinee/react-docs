# React-Query 도입을 위한 고민 (feat. Recoil)

Redux를 사용할 때 서버 데이터를 활용하려면 반드시 Redux-saga, Redux-Thunk, RTK같은 또다른 미들웨어를 사용해야한다.  
이 글은 많은 보일러 플레이트 코드를 대체해줄 수 있는 React-Query에 대한 글이다.

## React-Query, Recoil을 도입하기 까지의 고민들

리액트 쿼리의 장점중 하나는 데이터를 캐싱한다는 점이다.

> Declarative & Automatic
> Writing your data fetching logic by hand is over.
> Tell React Query where to get your data and how fresh you need it to be and the rest is automatic. React Query handles caching, background updates and stale data out of the box with zero-configuration.

리액트 쿼리는 기본적으로 데이터를 fetching 해온 후 데이터를 캐싱하게 되며 해당 데이터가 stale 하다고 판단할 때 데이터를 refetching 해오게 된다.

Client Side에서 캐싱은 유용하면서 굉장히 위험한 기술이다. 서버 데이터를 fetching 해 데이터를 캐싱한 후 해당 데이터를 확인할 때 만약 서버 상에서 데이터의 상태가 변경되었다면, 사용자는 잘못된 데이터를 확인하게 되며, 개발자의 입장에서는 이는 데이터를 무결성을 해치는 경우로 인지될 수 있다.

브라우저에서 사용자가 최신 데이터를 봐라봐야 하는 상황은?

- 화면을 보고 있을 때
- 페이지가 전환될 때 (새로운 페이지를 마주했을 때)
- 페이지 전환 없이 뭔가의 데이터를 요청할 때(예를 들면 클릭 이벤트)

React-Query가 데이터를 Refetching 해오는 상황은?

- 브라우저에 포커스가 들어왔을 경우 (refetchOnWindowFocus)
- 새로 마운트가 되었을 경우 (refetchOnMount)
- 네트워크가 끊어졌다가 다시 연결된 경우 (refetchOnReconnect)
- React-Query는 캐싱된 데이터는 항상 stale하다고 판단하며, stale 상태인 데이터를 refetching

결국 서버 데이터를 패칭해 온 데이터를 캐싱했어도, 사용자가 화면을 바라보고 있을 때는 그 시점에 있어서 가장 최신의 데이터를 바라보고 있는 상황이며, 페이지가 전환이 되었을 경우에도 해당 데이터의 상태가 stale 하다고 판단하여 리패칭 하며, 페이지에서 어떤 이벤트가 발생했을 경우엔 개발자가 트리거 를 심어줌으로 써 데이터를 리패칭 할 수 있다.

즉, 위와 같은 React-Query 의 컨셉으로 인해서 사용자는 항상 신선한 (fresh) 데이터를 바라볼 수 있다.

## Client 데이터와 Server 데이터의 분리

Redux, Recoil 은 Client 에서 전역 상태를 관리하고자 사용하는 라이브러리 입니다. React-Query 를 활용한다면 전자의 라이브러리들이 본연의 역할에만 집중할 수 있도록 할 수 있습니다. 즉 서버 데이터와, 클라이언트 데이터를 분리하는 것입니다.

서버와 클라이언트 상태의 결합이 필요하다면 아래와 같이 할 수 있다.

```js
export const cloudReadinessAtom = atom({
  key: "cloudRadiness",
  effects: [],
})

const projectId = getProjectId()
// 서버에서 받아온 List는 useQuery의 데이터만을 사용해야한다. 만약 필요한 경우에 atom을 통해 전역 데이터로 관리
const setCloudReadinessListAtom = useSetRecoilState(coloudReadinessAtom)
const { invalidateQueries } = useQueryClient() // 서버에서 데이터를 리패칭하기 위해 사용할 수 있는 메서드
const { data, isLoading } = useQueryLibrary(
  ["CloudReadinessList"],
  () => {
    return api({
      url: `/api/projects/${projectId}/cloud-readiness/answers`,
      method: "GET",
    })
  },
  {
    onSuccess: (data) => setCloudReadinessListAtom(data),
    onError: (error) => console.log(error),
  }
)
```

서버 데이터를 위와 같이 GlobalState로 사용할 수 있다.

## Success 혹은 Error 상황을 공통적으로 핸들링 할 수 있을까?

Redux와 Redux-saga를 활용할 때 API Call 오류가 났을 시 이를 중앙에서 핸들링하여 통제할 수 있었다. React-Query를 활용한다면 에러 상황을 어떻게 중앙에서 한번에 핸들링할 수 있을까?

아래와 같다.

```js
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: (error) => console.log((error as any).message)
    }
  }
})
```
