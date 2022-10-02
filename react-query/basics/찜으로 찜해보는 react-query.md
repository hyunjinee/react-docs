# 찜으로 찜해보는 react-query

## 도입 배경

트렌비에서는 리덕스를 사용하는데 있어 큰 문제는 없었으나 아래와 같은 몇가지 불편함이 있었습니다.

첫째, API를 호출하고 그 결과를 화면에 그리기 위해서 해야할게 너무 많았습니다.

1. 컴포넌트에서 API를 호출하기 위한 액션(action)을 정의해 줘야 합니다.
2. 액션과 연결되는 제너레이터(generator)를 만들어서 API를 호출해야 합니다.
3. API 결과를 처리하기 위해 다시 성공, 실패에 대한 액션을 만들줍니다. 데이터 캐시가 필요하다면 여기서 구현해야 합니다.
4. 스토어(store)에서 이 결과를 저장할 수 있도록 처리해줘야 합니다.
5. 스토어에 저장된 값을 가져오기 위해 셀렉터(selector)를 만들어 줘야 합니다.
6. 마지막으로 컴포넌트는 셀렉터를 통해 API 결과를 화면에 그려준다.

둘째, 트렌비 프론트엔드 구조의 문제로 공통적인 부분을 공통적으로 처리하지 못하고 있었습니다. 컨테이너/컴포넌트 구조를 계속 확장하다 보니 사가(saga)와 스토어가 개별 컨테이너에 속해있었습니다. 웹 페이지 규모가 커지면서 하나의 컨테이너가 다른 컨테이너에 속한 API를 호출해야하는 상황이 빈번히 발생하기 시작했습니다. 예를 들면 트렌비 매거진 목차를 가져오는 API는 매거진 컨테이너에서 호출하지만 홈화면에서도 호출해야 했습니다. 홈에서 매거진 목차를 가져오는 API 하나로 인해 매거진 컨테이너에 대해 종속성이 생기게 됩니다. 따라서 트렌비는 새로운 구조를 만들어서 이 문제를 해결해야했습니다.

셋째, 우리는 정말 상태관리를 하고 있는가라는 근본적인 질문이 생겼습니다. 리덕스를 단순히 사용한다는 것만으로 상태관리를 하고 있다고 할 수 있을까요?

트렌비에서는 비대해지면서 비효율적인 리덕스를 이용한 상태관리를 개선해보기위해 조사를 시작했습니다. 리덕스 사용에 대한 불편함은 리덕스팀에서도 알고 있는지 리덕스 툴킷이라는 패키지도 존재했습니다. 리코일과 몹엑스도 있었으나 리액트 쿼리와 SWR을 최종 후보로 선택했습니다. 가장 큰 이유는 현재 사용하고 있는 리덕스에 대한 수정 없이 리액트쿼리와 SWR을 사용할 수 있다는 것이었습니다.

## 리액트 쿼리 vs SWR

리액트 쿼리의 장점

- mutation하기위한 함수가 따로 존재.
- 리액트 쿼리용 디버깅 도구가 포함되어있어 디버깅 편의성 제공
- 캐시 시간 설정을 통해 캐시삭제 가능. 따라서 캐시 삭제를 위한 구현을 하지 않아도됨.
- SWR보다 많은 레퍼런스

SWR의 장점

- 15.7KB로 리액트 쿼리의 1/3 크기

라이브러리 크기가 3배 크긴 하지만 한번 정한 라이브러리를 바꾸는 것은 어려운 일이며, 라이브러리를 바꿔야하는 상황이 크기로 인한 문제보다는 원하는 기능이 없어서일 확률이 더 클 것이라는 결론을 내리고 리액트 쿼리를 점진적으로 사용해 보기로 했습니다.

트렌비의 다양한 부분에서 도입해서 사용하고 있지만, 이 글에서는 리액트 쿼리로 CRUD를 간단하게 적용해 볼 수 있었던 찜 페이지를 예로 들어 설명해 보겠습니다.

## 찜 페이지를 이용해 찜 해보자

리액트 쿼리는 상태 관리 영역중 서버 상태 관리에 초점을 맞추고 있는 라이브러리 입니다. 프론트 개발에서 상태관리는 클라이언트 상태와 서버 상태관리로 나눌 수 있습니다. 그 중 서버 상태관리는 말 그대로 CRUD를 통해 서버와 데이터 싱크를 맞추는 부분이라고 할 수 있습니다.

찜 페이지를 예로 들자면 찜한 개수, 상품 찜하기, 찜한 상품 삭제와 같이 클라이언트에서 발생한 동작이 서버에 영향을 주는 기능들입니다. 반면 클라이언트 상태 관리 영역은 서버에 영향을 주지 않습니다. 어떤 모달창이 열려있는지, 사용자가 스크롤한 위치가 어딘지, 어떤 아코디언 메뉴가 열려있는지와 같은 것 입니다.

찜 페이지에서 사용자가 현재 찜한 개수가 몇개인지 표시되는 부분이 있습니다. 그리고 상품 화면에서 하트 버튼을 클릭하면 상품을 찜하게 되고 이 때 상품 찜 개수도 변경됩니다.

![찜](https://tech.trenbe.com/imgs/posts/202206/1.add-to-wish.gif)

이제 리액트 쿼리에서 제공하는 `useQuery`와 `useMutation`을 사용해 이 과정을 구현해 봅시다.

## 1. useQuery

`useQuery`는 이름 그대로 쿼리(query), 즉 CRUD에서 Read에 해당하는 동작을 위한 함수입니다. 무한 스크롤이나 페이지네이션이 필요한 경우 `useQuery`대신 `useInfiniteQuery`를 사용할 수 있습니다. 두 함수의 차이점은 페이지에 대한 설정이 추가되고 캐시도 페이지별로 가능하다는 점입니다.

```js
const {
  data,
  dataUpdatedAt,
  error,
  errorUpdatedAt,
  failureCount,
  isError,
  isFetched,
  isFetchedAfterMount,
  isFetching,
  isPaused,
  isLoading,
  isLoadingError,
  isPlaceholderData,
  isPreviousData,
  isRefetchError,
  isRefetching,
  isStale,
  isSuccess,
  refetch,
  remove,
  status,
  fetchStatus,
} = useQuery(queryKey, queryFn?, {
  cacheTime,
  enabled,
  networkMode,
  initialData,
  initialDataUpdatedAt,
  isDataEqual,
  keepPreviousData,
  meta,
  notifyOnChangeProps,
  onError,
  onSettled,
  onSuccess,
  placeholderData,
  queryKeyHashFn,
  refetchInterval,
  refetchIntervalInBackground,
  refetchOnMount,
  refetchOnReconnect,
  refetchOnWindowFocus,
  retry,
  retryOnMount,
  retryDelay,
  select,
  staleTime,
  structuralSharing,
  suspense,
  useErrorBoundary,
})
```

이 훅(hook)을 이용해 찜 개수를 가져와 봅시다.

### useQuery를 이용한 찜한 개수 가져오기

트렌비에서 찜한 개수는 찜한 상품 개수와 찜한 브랜드 개수의 합으로 표시됩니다.

![](https://tech.trenbe.com/imgs/posts/202206/2.react-query-count.png)

API 응답 예제입니다.

```js
Request /api/wish/count

Response
{ totalCount: 2, product: 1, brand: 1}
```

`useQuery`를 컴포넌트에서 바로 사용할 수도 있습니다. 그러나 우리는 API가 특정 컴포넌트에 종속되는 실수를 반복하고 싶지 않았습니다.

API와 컴포넌트를 분리하고 어디서든 호출하기 위해 함수를 하나 만들어 `useQuery`를 감싸줬습니다.

```js
export const useWishGetCount = () =>
  useQuery(
    [WISH_GET_COUNT], // <-- 키를 세팅하는 부분
    async (): Promise<WishCount> => {
      // <-- query 함수
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/count",
        })
      )

      return response.data
    }
  )
```

```js
const WISH_GET_COUNT = "wish_get_count" as const
```

리액트 쿼리에서 서버 상태를 관리한다고 했는데, 그 핵심 중 하나는 키(key)사용입니다. 키를 사용해 결과값을 캐시하며, 캐시한 데이터를 수정/삭제할 수 있습니다. 당연히 이 키는 다른 키와 중복되면 안됩니다.

키 설정하는 부분을 보시면 `[WISTH_GET_COUNT]`처럼 배열로 인자를 받고 있습니다. 즉 상황에 따라 얼마든지 키를 더 세분화해서 추가할 수 있습니다. 예를 들어 전체 찜한 개수를 가져오는 API가 있고, 브랜드와 상품찜 수를 가져오는 API가 따로 있다고 가정해보겠습니다. 그러면 아래와 같이 상품찜에 대한 키를 세팅할 수 있습니다.

```js
export const useWishGetCount = (type: "brand" | "product" | "total") =>
  useQuery(
    // type 키를 추가해서 서로 다른 값을 캐싱할 수 있도록 함.
    [WISH_GET_COUNT, type],
    async (): Promise<WishCount> => {
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/count",
          query: { type },
        })
      )

      return response.data
    }
  )
```

배열을 이용해서 키를 설정했기 때문에 캐시에 접근하기 위한 두가지 방법이 생겼습니다. 첫째는 `WISH_GET_COUNT`를 키로 가지는 모든 데이터에 접근하는 방법이고 두번째는 `WISH_GET_COUNT`와 `type`을 이용해서 데이터에 접근하는 방법입니다. `brand`, `product`, `total`세 가지 타입으로 저장된 모든 데이터를 삭제하기 위해서는 `WISH_GET_COUNT` 키만 이용하면 됩니다. 자세한 내용은 `useMutation`에서 좀 더 다루겠습니다.

### useQuery를 이용해 찜한 개수 컴포넌트에 표시하기

`useWishGetCount`을 구현할 때 `useQuery` 훅에서 `return response.data`로 돌려주는 값은 `data` 필드를 통해 접근할 수 있습니다.

```ts
const HeaderMobileIconMenus = () => {
  // data 필드 접근.
  const { data: wishListCount } = useWishGetCount()

  return (
    <li>{wishListCount.totalCount > 99 ? "99+" : wishListCount.totalCount}</li>
  )
}
```

리덕스에서 거의 동일한 동작을 하는 코드는 아래처럼 많은 작업을 해야 합니다. 리액트 쿼리 도입으로 얼마나 많은 개발 시간을 단출할 수 있었는지 알 수 있습니다.

```ts
// constants
const REQUEST_WISH_COUNT = "request_wish_count" as const
const RESPONSE_SUCCESS_WISH_COUNT = "ressponse_success_wish_count" as const
const RESPONSE_ERROR_WISH_COUNT = "response_success_Wish_count" as const

// actions
const getWishCountRequest() {
  return {
    type: REQUEST_WISH_COUNT
  }
}

const getWishCountSuccess(data) {
  return {
    type: RESPONSE_SUCCESS_WISH_COUNT,
    data
  }
}

const getWishCountError(error) {
  return {
    type: RESPONSE_ERROR_WISH_COUNT
    error
  }
}

// saga
function* getWishCount() {
  try {
    const opt = {
      method: 'get',
      headers: {
        'Content-Type': 'application/json',
      },
    }

    const response = yield call(request, 'https://host/api/wish/count', opt)
    yield put(getWishCountSuccess(response.data))
  } catch (error) {
    yield put(getWishCountError(error))
  }
}

// reducer
function wishStore(state = initialState, action) {
  switch (action.type) {
    case RESPONSE_SUCCESS_WISH_COUNT:
      return state
        .set('wishListCount', action.data)

    case RESPONSE_ERROR_WISH_COUNT:
      return state
        .set('error', error);

// selector
export const makeSelectWishCount = () => createSelector(
  selectWish(),
  (wish) => wish.wishCount
);

const HeaderMobileIconMenus = () => {
  const wishListCount = useSelector(makeSelectWishCount())

  return (
    <li>
      { wishListCount.totalCount > 99 ? '99+' : wishListCount.totalCount }
    </li>
  )
}
```

`useQuery`의 아주 기본적인 사용법을 알아보았습니다. 몇 가지 팁을 통해 조금 더 `useQuery`를 잘 사용하는 방법을 알아봅시다.

### TIP1. wishListCount 초기값을 undefined 대신 원하는 초기값으로 수정하기

최초 `useWishGetCount()`를 호출할 때, `data`에서 초기값으로 `undefined`가 대신 0이 전달되도록 해 봅시다. 초기값은 `useQuery`의 마지막 인자인 `Option`에서 설정할 수 있습니다.

```ts
export const useWishGetCount = () =>
  useQuery(
    [WISH_GET_COUNT],
    async (): Promise<WishCount> => {
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/count",
        })
      )

      return response.data
    },
    // Option 부분
    {
      // 초기값.
      initialData: { total: 0, product: 0, brand: 0 },
    }
  )
```

### TIP 2. staleTime vs cacheTime

찜 리스트는 자주 변경되는 데이터가 아니기 때문에 한 번 데이터를 받아오면 새롭게 갱신될 때까지는 데이터를 받아오지 않아도 됩니다. 리액트 쿼리는 캐시를 해준다고 하였으나 `useWishGetCount`를 사용할 때마다 API도 호출하고 있었습니다. 이것은 `useWithGetCount`를 구현할 때 `Option`에 `staleTime`을 설정하지 않았기 때문입니다. 리액트 쿼리는 `cacheTime` 5분 `staleTime` 0이 기본값입니다. 이제 staleTime을 5분으로 설정하면 5분동안 API 호출을 하지 않습니다.

```ts
export const useWishGetCount = () =>
  useQuery(
    [WISH_GET_COUNT],
    async (): Promise<WishCount> => {
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/count",
        })
      )

      return response.data
    },
    {
      // staleTime 추가
      staleTime: 5 * 60 * 1000,
      initialData: { total: 0, product: 0, brand: 0 },
    }
  )
```

그렇다면 staleTime과 cacheTime은 무엇이고 어떻게 다를까요?

`staleTime`은 `useQuery`에서 `return`으로 전달한 값이 얼마나 유효한가에 대한 시간입니다. `staleTime`에서 설정된 시간만큼 유효하기 때문에 `staleTime`동안 API를 다시 호출하지 않고 캐시되어 있는 데이터를 사용합니다. `cacheTime`은 `useQuery`에서 `return`으로 전달한 값을 얼마나 오랫동안 가지고 있을지에 대한 시간입니다. 캐시한 데이터가 `staleTime`을 초과해 유효하지 않다고 하더라도 캐시를 지우지 않고 `cacheTime`동안 보관합니다.

이해를 돕기 위해 `staleTime`과 `cacheTime`을 극단적으로 설정해 봅시다.

CacheTime 5min, StaleTime 0

![](https://tech.trenbe.com/imgs/posts/202206/3.cacheTime5min%20staleTime0min.png)

A Component가 useWishGetCount를 호출하였을 때는 캐시된 데이터가 없기 때문에 초기값을 반환합니다. 그러나 B Component가 `useWishGetCount`를 호출한 시점에는 캐시된 데이터(이미지에서 노란색 부분)가 있기 때문에 캐시된 데이터를 전달합니다. 다만 `staleTime`이 0이기 때문에 다시 API를 호출하고 있습니다. API 응답값이 캐시된 데이터와 다른 경우 캐시를 업데이트하고 변경된 데이터를 B Component로 전달합니다.

CacheTime 5min, StaleTime 5min

![](https://tech.trenbe.com/imgs/posts/202206/4.cacheTime5min%20staleTime5min.png)

A Component가 `useWishGetCount`를 호출하였을 때는 캐시된 데이터가 없기 때문에 초기값을 반환합니다. B Component가 `useWishGetCount`를 호출한 시점에는 캐시된 데이터(이미지에서 노란색 부분)도 있고, `staleTime`(이미지에서 주황색 부분)도 유효하기 때문에 캐시된 데이터를 전달하고 `API`를 호출하지 않습니다. 그러나 `cacheTime`, `staleTime`이 모두 종료된 시점에는 처음 useWishGetCount를 호출할 때와 동일하게 동작합니다.

CacheTime 0, StaleTime 5min

![](https://tech.trenbe.com/imgs/posts/202206/5.cacheTime0min%20staleTime5min.png)

A Component가 `useWishGetCount`를 처음 호출할때는 동일합니다. B Component가 `useWishGetCount`를 호출한 시점에는 캐시된 데이터는 없고, `staleTime`(이미지에서 주황색 부분)만 유효한 상태입니다. 리액트 쿼리는 이 경우 `staleTime`을 무시하고 다시 `API` 호출을 한 후 결과를 다시 전달해 줍니다.

`useQuery`를 통해 찜 개수를 가지고 왔고 초기값도 설정했으며 `staleTime`과 `cacheTime`을 통해 적절히 캐시도 할 수 있게 되었습니다. 이제는 `useMutation`을 통해 서버 상태를 업데이트 해 봅시다.

## 2. useMutation이란?

CRUD에서 READ를 제외한 CUD를 위해 사용할 수 있습니다. [공식문서](https://tanstack.com/query/v4/docs/reference/useMutation)를 통해 자세한 내용을 확인하실 수 있습니다.

### useMutation을 이용해 상품 찜하기

이해를 돕기 위해 API와 응답값은 아래와 같습니다.

```
Request https://host/api/wish/product
Method: POST
Body: { "productId" : number }

Response
{
  success: true,
  data: {
    id: wishId
  }
}
```

`useWishGetCount`와 동일하게 `useMutation`도 함수로 한 번 감싼 후 다른 컴포넌트에서도 언제든 호출할 수 있도록 합니다.

```ts
export const useWishAddProduct = () =>
  useMutation(
    async ({ productId }: AddWishProductReq): Promise<AddWishProductData> => {
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/product",
        }),
        {
          method: "POST",
          body: { productId },
        }
      )

      return response.data
    }
  )
```

캐시를 사용하지 않기 때문에 키, `staleTime`, `cacheTime` 같은 설정없이 상대적으로 단순한 형태를 가지고 있습니다. 이제 찜하기 버튼을 클릭했을 때 이 `useWishAddProduct` 훅을 사용하면 됩니다. `useMutation`은 훅을 사용하는 시점에 API 호출이 발생하지 않고 `mutate`를 사용하는 시점에 API 호출이 발생합니다. 그래서 `mutate`를 사용할 때 인자값으로 `AddWishProductReq` 타입을 넘겨야 합니다.

```ts
function BtnWish({...}: ProductWishBtnProps) {
  // 이름을 재정의해서 가독성을 높입니다.
  const {
    mutate: onAddToWish,
  } = useWishAddProduct()

  const onClickWish = useCallback(
    debounce(() => {
      // mutate 함수를 호출합니다.
      onAddToWish({ productId })
    }, 500),
    [productId],
  )

  return <buttn onClick={onClickWish}>찜하기</button>
}
```

리덕스로 동일한 기능을 구현할 때와 코드양은 비교 불가입니다.

```ts
// constants
const REQUEST_ADD_WISH_PRODUCT = "request_add_wish_count" as const
const RESPONSE_SUCCESS_ADD_WISH_PRODUCT =
  "ressponse_success_add_wish_count" as const
const RESPONSE_ERROR_ADD_WISH_PRODUCT =
  "response_success_add_wish_count" as const

// actions
const postAddWishProductRequest({ productId }) {
  return {
    type: REQUEST_ADD_WISH_PRODUCT,
    data: productId
  }
}

const postAddWishProductSuccess(data) {
  return {
    type: RESPONSE_SUCCESS_ADD_WISH_PRODUCT,
    data
  }
}

const postAddWishProductError(error) {
  return {
    type: RESPONSE_ERROR_ADD_WISH_PRODUCT,
    error
  }
}

// saga
function* postAddWishProduct({ productId }) {
  try {
    const opt = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ productId })
    }

    const response = yield call(request, 'https://host/api/wish/product', opt)
    yield put(postAddWishProductSuccess(response.data))
  } catch (error) {
    yield put(postAddWishProductError(error))
  }
}

// reducer
function wishStore(state = initialState, action) {
  switch (action.type) {
    case RESPONSE_SUCCESS_ADD_WISH_PRODUCT:
      return state
        .set('addWishProduct', action.data)

    case RESPONSE_ERROR_ADD_WISH_PRODUCT:
      return state
        .set('error', error);


      // component
function BtnWish({...}: ProductWishBtnProps) {
  const dispatch = useDispatch()

  const onClickWish = useCallback(
    debounce(() => {
      dispatch(postAddWishProductRequest({ productId }))
    }, 500),
    [productId],
  )

  return <buttn onClick={onClickWish}>찜하기</button>
}
```

서버에 찜한 상품이 하나 추가 되었습니다. 그러자 서버에서 받아와 가지고 있던 기존 데이터들은 서버와 상태가 맞지 않게 되었습니다. 찜한 상품 개수도 맞지 않고, 찜 목록 페이지에 가면 방금 찜한 상품도 보이지 않았습니다. 이제 이 문제들을 풀어 봅시다.

### 문제1. 찜한 상품이 추가되었으나 찜한 개수는 변경되지 않음

앞에서 `useQuery`를 이용해 상품 개수를 가지고 오고 캐시를 하고 있었습니다. 이제 여기서 상품을 추가했으니 자동으로 개수가 1 늘어나면 좋겠지만 그런 일이 발생하지는 않습니다. 특히 상품 찜 개수는 캐시되어 있기 때문에 다른 페이지로 이동하더라도 `staleTime`동안 API가 다시 호출되지 않고 개수가 그대로 유지됩니다. 이 상황을 해결할 수 있는 두 가지 방법이 있습니다.

1. `useQuery`는 `refetch`라는 함수도 응답값에 전달해 줍니다. 이 함수를 이용해서 `staleTime` 시간과 관계없이 API를 호출할 수 있습니다.
2. API 호출 없이 저장된 캐시값만 업데이트 합니다.

여기서는 두 번째 방법을 사용해 보겠습니다.

`useWishGetCount`에서 캐시에 저장될 때 사용하는 키 값으로 `WISH_GET_COUNT`를 설정했었습니다. 따라서 `WISH_GET_COUNT` 키 값을 이용해 캐시 데이터를 불러온 후 값을 업데이트하고 다시 동일한 키 값으로 저장하도록 하겠습니다. 물론 이 모든 동작은 찜하기가 성공했을 때 발생해야 하기 때문에 `useMutation`에서 `onSuccess`콜백에서 작업해야 합니다.

```ts
export const useWishAddProduct = () =>
  useMutation(
    async ({ productId }: AddWishProductReq): Promise<AddWishProductData> => {
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/product",
        }),
        { productId }
      )

      return response.data
    },
    {
      onSuccess: () => {
        // WISH_GET_COUNT로 캐시되어 있던 데이터를 가지고 옵니다.
        const number: WishCountDto = queryClient.getQueryData(
          WISH_GET_COUNT
        ) as WishCountDto

        // query client를 통해 WISH_GET_COUNT 키에 저장된 값을 다시 세팅해 줍니다.
        queryClient.setQueryData(WISH_GET_COUNT, {
          // brandCount는 변경이 없기 때문에 변경하지 않습니다.
          brandCount: number?.brandCount ?? 0,
          // productCount는 1을 추가합니다.
          productCount: number?.productCount + 1 ?? 1,
          // 전체 개수도 1을 추가합니다.
          totalCount: number?.totalCount + 1 ?? 1,
        })
      },
    }
  )
```

이제 찜하기를 누르게 되면 API 요청이 성공했을 때 추가적인 API 호출 없이 찜 개수가 1 증가되는 것을 확인할 수 있습니다.

### 문제2. 찜 목록 페이지로 이동하면 새롭게 찜한 상품이 보이지 않음

`useWishAddProduct`를 호출하고, 찜 페이지에 가보면 사용자가 찜한 상품이 보이지 않습니다. 새로고침하게 되면 잘 보입니다. 즉 서버는 업데이트 되었으나 클라이언트 상태가 변경되지 않은 것입니다. 그러면 찜 개수 변경할 때 처럼 캐시를 업데이트 하거나 `refetch` 함수를 부르는 방법이 있습니다. 그러나 상황이 조금 다릅니다. 찜 개수는 페이지 상단에 있는 헤더에서 항상 숫자가 보이고 있습니다. 찜하기 동작이 완료되었을 때 변경 사항이 바로 적용되어야 합니다. 하지만 찜한 목록은 그렇지 않습니다. 당장 업데이트 하지 않고 기다렸다가 찜 목록 페이지로 진입할 때 새롭게 변경된 데이터가 보이면 됩니다. 이 때 사용할 수 있는 함수가 `invalidateQueries` 입니다. 캐시 데이터를 무효화 시키기 때문에 캐시 데이터를 사용하지 않고 API를 호출하게 됩니다. 인자 값으로 무효화 시킬 키 값을 넘겨주면 됩니다.

```ts
export const useWishAddProduct = () =>
  useMutation(
    async ({ productId }: AddWishProductReq): Promise<AddWishProductData> => {
      const response = await request(
        qs.stringifyUrl({
          url: "https://host/api/wish/product",
        }),
        { productId }
      )

      return response.data
    },
    {
      onSuccess: () => {
        // 찜 목록을 캐시하는 키 값을 전달해서 연결된 캐시를 무효화.
        queryClient.invalidateQueries(WISH_GET_PRODUCT_LIST)
        const number: WishCountDto = queryClient.getQueryData(
          WISH_GET_COUNT
        ) as WishCountDto
        queryClient.setQueryData(WISH_GET_COUNT, {
          brandCount: number?.brandCount ?? 0,
          productCount: number?.productCount + 1 ?? 1,
          totalCount: number?.totalCount + 1 ?? 1,
        })
      },
    }
  )
```

이렇게 두 가지 방법을 통해 `mutation`이 발생했을 때 상태 관리하는 방법을 알아보았습니다.

## 결론

우리가 고민하던 문제가 3개 있었습니다.

첫번째, `간단한 작업을 위해서도 해야할 일이 너무 많다`. 리액트 쿼리는 현재 사용중인 리덕스를 걷어내거나 수정하는 추가적인 노력없이 셀렉터, 액션, 스토어와 같은 보일러플레이트 작성에 들여야 하는 시간을 상당히 줄여 주었습니다. 리덕스에서 5개의 파일을 만들고 작업해야 했던 부분은 두 개의 파일로 줄었습니다. 그 만큼 구현도 단순해지고 디버깅 난이도도 낮아졌습니다.

두번째, `공통적인 부분을 공통으로 처리할 수 없었습니다`. 구조적인 문제를 수정한다는 것은 쉬운일이 아니지만 리액트 쿼리를 사용하면서 우리가 생각하는 이상적인 그림에 조금 더 가깝게 구조를 변경할 수 있었습니다. 여전히 많은 부분에서 리덕스 사가를 사용하고 있으나 여러 페이지에서 자주 사용되는 찜, IT-EM 부분들을 리액트 쿼리로 전환함으로써 약 3,000라인의 중복된 코드를 제거할 수 있었습니다.

세번째, `우리는 정말 리덕스를 이용해 상태를 관리하고 있는가?`리액트 쿼리는 그 목적에 맞게 서버 상태를 관리하는 부분에 집중하면서, 리덕스는 클라이언트 상태를 관리하도록 사용하고 있습니다. 이로 인해 리덕스에 대한 의존성이 낮아지고 있으며, 장기적으로 리덕스 툴킷이나 몹엑스(Mobx)와 같은 다른 상태 관리 도구로 대체할 계획도 세울 수 있게 되었습니다.
