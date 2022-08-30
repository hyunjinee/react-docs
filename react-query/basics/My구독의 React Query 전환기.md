# My구독의 React Query 전환기

React Query는 데이터 Fetching, 캐싱, 동기화, 서버 쪽 데이터 업데이트 등을 쉽게 만들어주는 React 라이브러리이다. 기존에 Redux, Mobx, Recoil과 같은 다양하고 훌륭한 상태 관리 라이브러리들이 있긴 하짐나, 클라이언트 쪽의 데이터들을 관리하기에 적합할 순 있어도 서버쪽의 데이터를 관리하기에는 적합하지 않은 점들이 있어서 등장하게 되었다.

예를 들어, 관리자 화면에서 동시에 두 명의 관리자가 같은 페이지를 바라보고 있는 상황에서 한 관리자가 유저의 데이터를 변경하면, 다른 관리자도 그 유저의 변경된 데이터를 바라볼 수 있어야한다. 이러한 유저의 데이터는 클라이언트쪽보다는 서버쪽에 좀더 적합하다고 볼 수 있다.

## React Query로 전환한 이유

My구독은 클라이언트 데이터보다 서버 데이터에 의존적인 성향의 서비스이다.

클라이언트의 복잡도가 높지 않고, 각 페이지에서 공통적으로 사용하는 전역 상태 데이터를 많이 가지고 있지도 않았다.

프로젝트 구성 당시에는 Redux와 Redux Saga를 사용했지만 프로젝트를 진행하면서 점점 API가 추가가되고 하나의 API당 1~4개의 액션 타입과 액션들이 생성되는등 Ducks 구조와 Saga 파일들로 인해 프로젝트의 구성이 좀 더 복잡해짐을 느꼈다.

가지처럼 뻗어 나가는 Redux-Saga의 사이드 이펙트들을 체계적으로 잘 관리, 제어할 수 있다면 Redux-Saga를 유용하게 사용할 수 있겠지만, Saga가 다른 Saga를 호출하고, 또 다른 Saga를 호출하는 등의 과정이 들어가면, 사이드 이펙트들을 관리하고 흐름을 제어하는게 쉽지 않을 수 있다.

또한 서버의 데이터와 클라이언트의 데이터 구분의 모호함도 존재하였고, 서버의 응답을 Store에 저장할 필요가 없는 상황에서도 구조화를 위해 Action Types, Actions, Reducer를 만들고, Action을 호출하는 등의 불필요한 상황, 캐싱 처리 및 관리 문제 등이 존재했습니다.

이러한 상황에서 Redux와 Redux-Saga를 활용한 구조는 My 구독의 서비스 로직과는 잘 맞지 않았다. 이후 프로젝트에 기능이 추가될수록 더 무거워지고 복잡해지면서 관리가 어려울 수 있다는 내부 의견들이 있었고, 좀 더 서비스에 적합한 라이브러리를 찾다가 React Query를 알게 되었다.

React Query를 일부 페이지에 적용하여 테스트를 진행해보니 다음과 같은 이점들 때문에 점진적으로 적용하게 되었다.

1. 프로젝트 구조가 기존보다 단순해졌다.
2. 캐싱 처리가 더 간단해졌다.
3. 직접 만들어서 사용했던 기타 기능들을 옵션들로 지원한다.
4. Redux와 Redux-Saga를 사용할 때는 Success, Failure값을 useEffect의 deps로 전달해서 처리해야하는 과정이 필요했는데, 이 과정을 onSuccess, onError로 간단하게 처리할 수 있다.

React Query외에도, RTK-Query, SWR등이 있었지만, React Query에 비해 오프라인 처리, 쿼리의 select와 같은 기능들이 부족하다고 판단하여 React Query로 전환하였다.

## React Query 전환

### QueryClient의 Default Option

먼저 QueryClient의 Default Option을 보자면, retry 옵션은 API가 실패하면 설정한 값만큼 재시도 하는 옵션이다.

My 구독에서 API 요청 실패가 발생한다는 건, 서버의 문제가 있다고 판단하여 유저에게 빠른 응답을 보여주기 위해 0으로 설정했다.

useErrorBoundary옵션은 리액트 16 이상에서 제공하는 Fallback UI 설정에 대한 옵션이다. 예기치 못한 에러 케이스가 생길 수 있다고 판단하여 queries와 mutations에 true 값으로 설정하였다.

```js
new QueryClient({
  defaultOptions: {
    queries: {
      retry: 0,
      useErrorBoundary: true,
    },
    mutations: {
      useErrorBoundary: true,
    },
  },
})
```

### useQuery 사용

useQuery같은 경우는 각 도메인 페이지에서 필요한 옵션들을 설정하여, src/hooks/queries에 선언된 쿼리들을 불러와서 사용하고 있다.

```js
// src/pages/subscribe/index.jsx
import { useKioskData } from '@/hooks/queries;
const { isLoading, data: kioskData } = useKioskData({ storeCode, options: {
      onError:()=>{...}
      onSuccess: () => {...},
});


//src/hooks/queries/useKioskData.js
import { useQuery, useInfiniteQuery } from 'React Query';
import * as queryKeys from '@/constant/queryKeys';

export function useKioskData({ storeCode, options }) {
  return useQuery([queryKeys.KIOSK_DATA, storeCode], () => api.get(`...`), {
    select: ({ data }) => data?.kioskData,
    ...options,
  });
}
```

### Query Key의 관리

React Query에서는 Query Key가 중복되지 않게 관리해야 캐싱을 활용할 수 있기에, 휴먼 에러를 방지하고자, 하나의 파일에서 관리하고, 접두사로는 도메인 네임을 설정하여 구분을 지었다.

## React Query 개선

프로젝트를 진행하면서, 기존 Redux, Redux-Saga 부분을 걷어내고 React Query를 점진적으로 적용하고 있었다.

다만, 세부적인 규칙이 없어 제각각 사용법이 달라 코드 리뷰 과정에서 조금 혼동이 있었고 컴포넌트가 복잡해지는 문제가 생겼다.

프로젝트 이후에, My구독의 React Query 사용 방식을 정리하고, 좀 더 구조적으로 개선하는 작업을 진행했다.

### Query Key 네이밍 수정

파일은 기존 방식과 동일하게 중복되는 네이밍을 방지하기 위해 하나의 파일에서 관리했다.

Query Key가 증가하면서, 기존의 네이밍 컨벤션으로는 조금 혼란스러운 점이 있어서, 리팩토링을 진행하면서 Query Key의 네이밍도 수정을 했다.

접두사는 그대로 도메인 그룹을 지정, 중간에는 query의 함수 이름을 추가하고, 접미사로는 API응답에 영향을 주는 값으로 지정했다.

```
[{도메인 그룹}_{query 함수 이름}, ...(API 응답에 영향을 주는 값)]
```

### useQuery에 중복되는 옵션 제거

defaultOptions에 useErrorBoundary를 true로 설정했지만 해당 값은 onError 옵션 값 여부에 따라 필요하지 않을 수 있다. 이런 부분을 매번 options 값을 확인하고 false로 설정하는 작업은 불필요할 수 있고, 휴먼에러가 생길 수 있기에 useQuery를 감싸서 처리하는 방식으로 변경했다.

```js
import { useQuery as useQueryOrigin } from "React Query"

export default function useQuery(queryKey, queryFn, options = {}) {
  const { onError } = options

  return useQueryOrigin(queryKey, queryFn, {
    ...options,
    useErrorBoundary: !onError,
  })
}
```

### useQuery 분리

먼저, 리팩토링 전, 기존의 폴더 구조를 보자면, pages 폴더 내에 각 페이지들이 선언되있고 쿼리들의 정의는 src/hooks/queries에서 관리하고 있었다.

다른 페이지에서도 사용하는 공통적인 쿼리들이 존재할 수 있어 /src/hooks/queries에 도메인 별로 나눠서 쿼리들을 정의하였다.

![image](https://user-images.githubusercontent.com/63354527/187342476-6cbb6ab0-bfef-42b3-8f24-2ac1f3bf61c6.png)

위와 같이 구조화하고 사용하다 보니 약간의 문제가 있었다.

useQuery를 사용할 때 options 쪽의 내용이 많아질수록, 컴포넌트의 복잡도가 증가하는 문제가 있었고, 병렬저긍로 선언된 쿼리가 많아질수록 복잡도가 더 증가했다.

```js

const voucher = useVoucherQuery({
  options: {
    enabled: productDataSuccess,
    useErrorBoundary: false,
    refetchOnMount: false,
    refetchInterval: false,
    refetchOnReconnect: false,
    refetchOnWindowFocus: false,
    refetchIntervalInBackground: false,
    select: () => {
      ...
    },
    onSuccess: () => {
      ...
    },
    onError: () => ...,
  },
});
```

이런 문제들을 해결하고자 쿼리들을 컴포넌트에서 분리하였다.

src/hooks 아래에 있는 API에 매칭되는 모든 쿼리의 선언부들은 기존 방식 그대로 유지하였다. (이하 베이스 쿼리)

컴포넌트 내부에서 사용되던 options 설정이 들어간 커스텀 한 쿼리들은 각 페이지별 hooks 폴더 아래로 위치키시고, 각 페이지에서 필요한 베이스 쿼리들을 불러와서 필요한 옵션들을 설정해서 사용하는 방식으로 변경하였다. (이하 커스텀 쿼리)
![image](https://user-images.githubusercontent.com/63354527/187343200-9c4b1b73-d11c-49e7-8d7e-ca50e1e8c726.png)

위와 같이 커스텀 쿼리는 각 컴포넌트에서 필요한 서비스 로직을 정의하고, 베이스 쿼리를 활용하여 구조화하였다.

베이스 쿼리에서는 위에서 불필요한 useErrorBoundary 옵션을 제거하는 useQuery를 래핑한 함수를 사용하였다.

```js
/* src/hooks/queries/voucher.js (베이스 쿼리) */
import { useQuery } from "@/hooks/queries/useQuery"
export function voucher({ storeCode, options } = {}) {
  return useQuery([queryKeys.VOUCHER_LIST, storeCode], () => api.get("..."), {
    ...options,
  })
}

/* src/pages/Voucher/hooks/queries/useVoucher.js (커스텀 쿼리) */
export function useVoucher({ storeCode }) {
  return voucher({
    storeCode,
    options: {
      // ...
    },
  })
}
```

이처럼 분기가 가능했던 이유는 다른 페이지 내에서 공통적으로 사용하는 API가 많지 않았고 공통적으로 사용하는 API는 각 페이지마다 options에 대한 설정이 달랐기에 위와 같이 변경할 수 있었다.

## React Query 사용 주의사항

### useQueries에는 options 값을 설정할 수 없다.

useQueries는 쿼리들을 묶어서 처리할 수 있게 해주는 훅이다. useQueries 자체적으로는 options 값을 설정할 수는 없지만, useQueries안에 선언되는 쿼리는 options 값을 설정할 수 있다. options에 대한 Object 값을 설정하는게 아닌 options에 대한 값 자체를 선언해야 동작한다.

### useMutation의 mutate 함수는 Promise를 반환하지 않는다.

GET 방식이 아닌 POST,PUT,DELETE 등의 HTTP Method를 사용할 때는 React Query의 useMutation을 활용하게 된다.

useMutation의 mutate 함수를 사용하면, useMutation에 설정한 onSuccess, onError와 같은 옵션을 통해서 서버의 응답 결과를 처리할 수 있다.

mutate 함수는 Promise를 반환하지 않기 때문에 이러한 설정 없이 const result = await mutate()와 같은 방식으로 직접적으로 결과를 할당해서 사용할 수 없다.

반면에, mutateAsync 함수는 Promise를 반환하기 때문에 직접적인 사용이 가능하다. trycatch 문안에 mutateAsync를 호출하여 성공, 실패와 같은 서버 응답 결과를 처리할 수 있다.

Redux-Saga를 사용했을 때는 takeLatest와 같은 이펙트를 사용하여 API의 중복 호출을 제어할 수 있었지만, useMutation은 중복 호출에 대한 제어 옵션이 없기 때문에 이를 제어하기 위해서는 쓰로틀링, 디바운싱을 구현해서 사용하거나 isLoading이나 useIsMutating을 사용해서 제어하는 방법이 존재한다.

### useIsFetching과 queryClient.isFetching은 결과가 다르다.

useIsFetching은 현재 fetching중인 쿼리의 개수를 리턴하는 훅이다. queryClient의 isFetching 또한 동일한 기능을 하는 함수이다.

useIsFetching은 내부적으로 queryClient의 queryCache들을 Observing하면서 fetching개수를 리턴하지만, queryClient.isFetching은 Observing 하지 않고 현재 캐싱된 데이터의 fetching 개수를 리턴한다.

```js
export default function Test() {
  const queryClient = useQueryClient()

  const isFetching = useIsFetching()
  const isFetching2 = queryClient.isFetching()

  const test = useUserData()

  /*
    0 0
    0 1
    1 1
  */
  console.log(isFetching, isFetching2)

  return <></>
}
```

결과적으로는 동일한 기능을 하지만 위와 같이 결과가 다른 이유는  
useIsFetching 내부에서는 Observing의 결과를 useEffect와 useState를 통해 리턴하기 때문에, queryClient.isFetching으로 사용하는 것보다 한 틱 더 늦게 값을 return 한다는 것을 할 수 있다.(여기서 틱이란, 훅이 return 되기까지의 과정을 한 틱이라고 칭한다.)

## React Query 활용

React Query에서 제공하는 기능들을 활용해서 다양한 방식으로 사용이 가능하다.

### Query Key 활용

Query Key는 상황에 따라서 유용하게 쓸 수 있다. 키는 hierarchy 구조이기 때문에 이를 활용해서화면 영역을 분리하는 용도로 키를 설정할 수 있다. 예를 들어,

```
A-Key : [‘page-a’, ‘product’, ‘10’]

B-Key : [‘page-a’, ‘product’, ‘20’]

C-Key : [‘page-a’, ‘user’, ‘30’]
```

쿼리 키를 위와 같이 설정하고 A,B,C 쿼리를 모두 갱신하고 싶은 경우에는 invalidateQueries(['page-a'])로 갱신이 가능하고, A, B 쿼리만 갱신하고 싶은 경우에는 invalidateQueries(['page-a', 'product'])로 갱신이 가능하다.

화면의 영역이 여러개의 영역으로 나누고, 해당 영역에 대한 쿼리를 관리하고 싶으면 Query Key의 네이밍을 전략적으로 구성하면 효과적으로 사용할 수 있다.

만약 쿼리에 대한 캐싱을 활용하고 싶고, 쿼리를 특정 영역별로 분할해서 관리하고 싶으면, 키의 인자 안에 object를 추가하면 좀 더 커스텀 하게 활용도 가능하다.

```js
useQuery(['todos', {section:'section-a'}], ...)
useQuery(['todos', {section:'section-b'}], ...)
```

위 두 개 쿼리의 Query Key는 같은 Key 값으로 인식되어 캐싱이 되고, section-a 쿼리만 갱신 처리하고 싶을 때는 invalidateQueries(['todos', {section:'section-a'}])와 같은 처리가 가능하다.

## Query Client를 활용한 커스텀 훅

Query Client를 활용해서 추가적인 기능의 훅들을 만들 수 있다. 예를 들어, useQuery를 병렬적으로 선언하여 사용하다보면, 선언된 모든 쿼리들의 Fetching 여부를 확인하고 싶을 때가 있다.

선언된 각 쿼리들이 리턴하는 status 값들을 활용할 수 있으나, 선언된 쿼리가 많아지면 status 값들을 활용하여 처리하는 코드도 길어지고, 복잡해질 수 있다.

React Query에서 제공하는 useIsFetching()와 같은 훅을 통해 체크를 할 수 있으나, 예외적인 상황들이 존재할 수 있다.

아래의 코드에서 q2Data와 q3Data의 참조에러를 방지하기 위해,q2Loading, q3Loading에 대한 부분을 분기 처리로 추가했는데 이런 부분들이 많아지면 분기가 복잡해질 수 있다.

이러한 상황에서는 useIsFetching을 활용할 수 있다. useQuery4처럼 enabled 값이 false인 쿼리가 존재하거나, useQuery2,3 처럼 enabled 값이 다른 쿼리의 Fetching 성공 여부나 데이터를 의존하는 경우에는 idle 상태 때문에, useIsFetching으로는 Fetching 여부를 제대로 확인할 수 없다.

```js
export default function Test() {
  const { isSuccess: q1Succeed } = useQuery1()
  const { data: q2Data, isLoading: q2Loading } = useQuery2({
    options: {
      enabled: q1Succeed,
    },
  })
  const { data: q3Data, isLoading: q3Loading } = useQuery3({
    options: {
      enabled: q1Succeed,
    },
  })
  const { refetch: q4Refetch } = useQuery4({
    options: {
      enabled: false,
    },
  })

  if (q2Loading || q3Loading) {
    return <>Loading...</>
  }

  return (
    <>
      {q2Data.tmpData} / {q3Data.tmpData}
    </>
  )
}
```

위 코드 쿼리들의 상태 변화는 아래 표와 같이 변하기 때문에, 3번째 단계에서 useIsFetching은 false를 리턴하지만 아직, useQuery2, useQuery3이 동작하지 않았기에 이 상태로 렌더링을 하게되면 문제가 발생할 수 있다.

<img width="777" alt="image" src="https://user-images.githubusercontent.com/63354527/187348270-167aa548-acaf-495f-83dd-c01d95e2347e.png">

이러한 문제를 해결하기 위해 My구독에서는 모든 쿼리들이 Fetching을 완료했는지 체크하는 커스텀 훅을 만들어서 컴포넌트의 렌더링 조건문을 줄일 수 있었다.

먼저 queryCache를 직접적으로 참조하면 선언된 쿼리들이 다 캐싱되기 전에 훅이 동작을 하기 때문에 useEffect를 사용해 선언된 쿼리들을 참조할 수 있게 만들고, 현재 선언된 쿼리의 개수를 state로 저장한다.

React Query는 enabled 값이 false인 쿼리와 enabled 값이 특정 값을 의존하는 쿼리를 구분할 수 있는 상태 값을 제공하지 않는다.

이 문제를 해결하기 위해 쿼리의 상태 변화 여부에 대한 카운트 값을 현재 선언된 쿼리의 개수만큼 설정한다.

초기값을 선언된 쿼리의 개수만큼 설정하여 enabled 값이 모두 false로 선언되는 쿼리들의 경우에도 상태를 체크할 수 있게 처리했다.

사용자가 임의로 enabled가 false로 설정된 쿼리의 개수를 확인해서 인자로 넘기는 방법으로 구현을 할수도 있지만, 휴먼에러가 발생할 수 있고, 관리해야할 포인트가증가하기 때문에 아래와 같은 방식으로 구현하였다.

```js
export function useQueriesLoading() {
  const client = useQueryClient()
  const queries = client.getQueryCache().findAll()

  const [queryChangedCount, setQueryChangedCount] = useState(0)

  useEffect(() => {
    if (queries) {
      setQueryChangedCount(queries.length)
    }
  }, [])

  useEffect(() => {
    if (
      queries.every(({ state }) => state.status !== "loading") &&
      queryChangedCount >= 0
    ) {
      setQueryChangedCount((prevState) => prevState - 1)
    }
  })

  if (queryChangedCount < 0) {
    return false
  }

  return true
}
```

## 마치며

### 좋았던 점

기존에 Redux-Saga를 통해 관리했을 때는 API 처리의 성공, 실패에 따른 분기들이 파악하기 힘든 경우도 있었고, 사가가 깊어지면 관리하기 까다로운 경우들이 있었는데 React Query로 좀 더 단순화 시킬 수 있어서 좋았다.

추가로 캐싱에 대한 처리라든지, refetching에 대한 설정 기능, placeHolder, failureCount등 다양한 옵션을 제공해서 개발 리소스를 좀 더 줄 일 수 있었던 점들도 좋았다.

### 아쉬웠던 점

위에서 작성한 React Query 활용 부분의 useQueriesLoading과 같은 커스텀 훅의 기능도 React Query에서 제공했으면 더 좋았을 것 같고, 기존 Redux-Saga에서 쓰던 takeLatest와 같은 이펙트처럼 Mutation 중복 호출에 대한 부분을 좀 더 편하게 처리할 수 있게 제공하는 함수들이 있었으면 좋았을 것 같다.

### 어떤 상황에서 React Query를 사용하면 좋을까?

클라이언트 사이드의 데이터보다는 서버 사이드의 데이터 관리가 더 많을 때 유용하게 쓰일 수 있다고 생각한다.

예를 들면 Admin 페이지와 같은 관리형 페이지에서는 클라이언트의 전역 상태 데이터는 많이 필요하지 않을 수 있다. 이러한 페이지에서 Ducks 구조보다는 React Query를 적용하면 구조를 더 단순화 시킬 수 있다.

또한 refetching 옵션들을 통해 서버의 데이터를 동기화할 수 있고, useMutation을 사용할 때도, defaultOptions에 onError를 추가하여 POST,PUT,DELETE등의 API 실패시 안내 문구를 띄우는 기능도 유용하게 쓰일 수 있을 것 같다.
