# 카카오페이 프론트엔드 개발자들이 React Query를 선택한 이유

프론트엔드 개발에 빠지지 않는 API 통신 어떤 방식으로 처리하고 계신가요?
카카오 페이에서는 API 통신과 비동기 데이터 관리를 위해 React Query를 적극적으로 활용하고 있습니다. 오늘은 저희가 React Query를 전향적으로 사용하게 된 이유에 대해서 이야기 해보려 합니다.

> if(kakao)2021의 [카카오페이 프론트엔드 개발자들이 React Query를 선택한 이유](https://if.kakao.com/2021/session/118)와 함께 보시면 더욱 좋습니다.

## React Query 도입 이전의 우리

저희가 React Query를 전향적으로 사용하게 된 이유를 이야기하기 위해서는 우선 React Query를 도입하기 이전의 상황을 살펴볼 필요가 있습니다.

다른 많은 프로젝트처럼 React Query를 도입하기 전 카카오페이에서는 서버와의 API 통신과 비동기 데이터 관리에 Redux를 주로 사용했습니다. 서비스 특성과 개발자 취향에 따라 redux-thunk, redux-saga 등 다양한 Asynchronous Middleware를 채택하여 사용하고 있었으며, 더 효율적인 업무를 위한 다양한 Custom Middleware를 작성하여 사용하기도 했었습니다.

## Redux, 이건 불편했다.

위와 같은 환경에서 저희가 Redux를 사용하여 API 통신을 수행하고 비동기 데이터를 관리하며 얻은 다양한 장점이 존재했지만, 반대로 불편하다고 느낀 부분도 상당히 있었습니다. Redux는 "Global State Management Library" 입니다. React Application을 개발함에 있어 일종의 De facto standard2(사실상 표준)로써 여겨지고 있고 대부분의 React Application 개발 환경 설정 시 자연스럽게 Redux가 마치 React Stack의 일부인 것처럼 구성되곤 했습니다.

이러한 현실 아래서 웹 프론트엔드에서 빈번하게 수행되는 API 통신에 Redux를 사용하는 것은 일견 자연스러운 선택이였습니다.

비동기 데이터를 React Component의 State에 보관하게 될 경우 다수의 Component의 LifeCycle에 따라 비동기 데이터가 관리되므로 캐싱 등 최적화를 수행하기 어렵습니다. 그리고 다수의 Component에서 동일한 API를 호출하거나, 특정 API 응답이 다른 API에 영향을 미치는 경우등 복잡하지만 빈번하게 요구되는 사용자 시나리오에 대응하기 쉽지 않습니다.

하지만 Global State Managment Library인 Redux를 사용하여 비동기 데이터를 관리할 경우 Component의 LifeCycle과 관계없이 Global State에서 비동기 데이터가 관리되기 때문에 캐싱과 같은 최적화작업을 쉽게 수행할 수 있고 복잡한 사용자 시나리오에 대한 대응도 용이해지기 때문입니다.

저희 카카오페이 프론트엔드 개발자들도 위와 같은 사유로 Redux로 API 통신과 비동기 데이터를 관리하고 있었으나 React Query를 접하고 난 뒤 다음과 같은 부분에서 (비동기 데이터 관리의 측면에서) React Query와 대비되는 Redux의 단점들을 느꼈습니다.

### 너무 장황한 Boilerplate 코드

이미 모두가 알다시피, Redux에는 [Redux 기본 원칙](https://redux.js.org/understanding/thinking-in-redux/three-principles)이 존재합니다. 이 기본 원칙을 충족하기 위해서 Redux를 사용하는데는 장황한 Boilerplate 코드가 요구됩니다. 이러한 이슈를 해결하기 위한 redux-toolkit의 등장 이후 Boilerplate 코드가 많이 줄어들었음에도 불구하고 Redux로 비동기 데이터를 관리하는 일에는 여전히 불필요하게 느껴지는 반복되는 Boilerplate 코드가 필요합니다.

아래는 Redux를 사용한 "비동기 데이터 동기화 기능을 갖춘 TodoList Application" 샘플 프로젝트입니다.

```tsx
/ Todo.tsx
import { useEffect } from 'react';
import { selectTodoList } from 'features/todos/todos.selector';
import {
  requestFetchTodos,
  requestPostTodos,
} from 'features/todos/todos.slice';
import { useForm } from 'react-hook-form';
import { useDispatch, useSelector } from 'react-redux';
function Todo() {
  const dispatch = useDispatch();
  const data = useSelector(selectTodoList);
  const { register, handleSubmit } = useForm<{
    contents: string;
  }>();
  useEffect(() => {
    // 컴포넌트가 마운트 될 때 서버에 저장된 Todo 정보를 불러옵니다.
    dispatch(requestFetchTodos());
  }, [dispatch]);
  const onSubmit = handleSubmit((value) => {
    // 사용자가 Form을 Submit하면 서버에 새로운 Todo 정보를 저장합니다.
    dispatch(requestPostTodos(value.contents));
  });
  return (
    <div>
      <header>
        <form onSubmit={onSubmit}>
          <input
            type="text"
            placeholder="What needs to be done?"
            autoComplete="off"
            {...register('contents')}
          />
        </form>
      </header>
      <div>
        <ul>
          {data?.map(({ id, contents }) => (
            <li key={id}> {contents} </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
export default Todo;
```

```tsx
// features/todos/todos.slice.ts
import { createSlice, PayloadAction } from "@reduxjs/toolkit"
import { TodoItem } from "types/todo"
export interface TodoListState {
  fetchTodos: {
    data?: TodoItem[]
    isLoading: boolean
    error?: Error
  }
  postTodos: {
    isLoading: boolean
    error?: Error
  }
}
const initialState: TodoListState = {
  fetchTodos: {
    data: undefined,
    isLoading: false,
    error: undefined,
  },
  postTodos: {
    isLoading: false,
    error: undefined,
  },
}
export const todoListSlice = createSlice({
  name: "todoList",
  initialState,
  reducers: {
    // 이전 State의 값을 바탕으로 다음 State의 값을 새로 만드는 순수함수 "Reducer"
    // redux-toolkit은 immer를 내부적으로 사용하므로 조금 더 자연스럽게 Reducer를 구성할 수 있게끔 도와줍니다.
    requestFetchTodos: (state) => {
      state.fetchTodos.isLoading = true
    },
    successFetchTodos: (state, action: PayloadAction<TodoItem[]>) => {
      state.fetchTodos.data = action.payload
      state.fetchTodos.isLoading = false
      state.fetchTodos.error = undefined
    },
    errorFetchTodos: (state, action: PayloadAction<string>) => {
      state.fetchTodos.data = undefined
      state.fetchTodos.isLoading = false
      state.fetchTodos.error = action.payload
    },
    requestPostTodos: (state, _: PayloadAction<string>) => {
      state.postTodos.isLoading = true
    },
    successPostTodos: (state) => {
      state.postTodos.isLoading = false
    },
    errorPostTodos: (state, action: PayloadAction<string>) => {
      state.postTodos.isLoading = false
      state.postTodos.error = action.payload
    },
  },
})
export const {
  requestFetchTodos,
  successFetchTodos,
  errorFetchTodos,
  requestPostTodos,
  successPostTodos,
  errorPostTodos,
} = todoListSlice.actions
export default todoListSlice.reducer
```

```tsx
// features/todos/todos.saga.ts
import { PayloadAction } from "@reduxjs/toolkit"
import axios from "axios"
import { call, put, takeEvery } from "redux-saga/effects"
import { TodoItem } from "../../types/todo"
import {
  errorFetchTodos,
  errorPostTodos,
  requestFetchTodos,
  requestPostTodos,
  successFetchTodos,
  successPostTodos,
} from "./todos.slice"
async function getTodoList() {
  const { data } = await axios.get<TodoItem[]>("./todos")
  return data
}
function* requestFetchTodoTask() {
  try {
    const data: TodoItem[] = yield call(getTodoList)
    yield put(successFetchTodos(data))
  } catch (e) {
    yield put(errorFetchTodos(e.message))
  }
}
async function postTodoList(contents: string) {
  await axios.post("/todos", { contents })
}
function* requestPostTodoTask(action: PayloadAction<string>) {
  try {
    yield call(postTodoList, action.payload)
    yield put(successPostTodos())
  } catch (e) {
    yield put(errorPostTodos(e.message))
  }
}
function* successPostTodoTask() {
  // 서버에 새로운 Todo 추가 요청 성공 시
  // 서버에서 Todo 목록을 다시 받아오기 위해 Action Dispatch
  yield put(requestFetchTodos())
}
function* todoListSaga() {
  yield takeEvery(requestFetchTodos.type, requestFetchTodoTask)
  yield takeEvery(requestPostTodos.type, requestPostTodoTask)
  yield takeEvery(successPostTodos.type, successPostTodoTask)
}
export default todoListSaga
```

컴포넌트 구성에 필요한 일부 소스코드만 발췌하였는데도 Application의 기능에 비해 코드의 분량이 상당합니다. redux-toolkit을 사용하여 불필요한 코드를 많이 줄였음에도 불구하고 장황한 분량의 Boilerplate 코드가 남아있으며, 하나의 API 요청을 처리하기 위해 여러개의 Action과 reducer가 필요하여 전체 코드가 눈에 잘 들어오지 않는 것 같습니다. 비동기 Action을 처리하기 위해 채택한 redux-saga쪽 코드도 수행하는 역할에 비해 분량이 너무 장황하게 느껴집니다. 이러한 구조하에서는 처리해야하는 API의 개수가 많아질수록 코드의 분량이 많이 늘어날 뿐더러 비동기 Action을 처리하기 위한 복잡성이 높아지게 될 우려가 있습니다.

### API 요청 수행을 위한 규격화된 방식 부재

너무나 자명하게도, Redux는 API 통신 및 비동기 상태 관리를 위한 라이브러리가 아닙니다.
Redux를 사용하여 비동기 데이터를 관리하기 위해서는 관련된 코드를 하나부터 열까지 개발자가 결정하고 구현해야합니다. API 관련 상태 저장 방법이 가장 대표적인 예시입니다. 개발자의 선택에 따라 API 응답을 전부 State에 보관하고 Selector에서 필요한 값만 계산해서 사용할 수도 있고, 보관할 때 필요한 값만 State에 보관하는 경우도 있습니다. 더 나아가 API의 로딩 여부를 Boolean을 사용해서 관리하는 경우도 있고, IDLE | LOADING | SUCCESS | ERROR등 상태를 세분화해서 관리하는 경우도 있을 겁니다.

```ts
// 로딩 상태를 관리하는 방법도 개발자에 따라 다르게 구현됩니다.
interface ApiState {
  data?: Data
  isLoading: boolean
  error?: Error
}
interface ApiState {
  data?: Data
  status: "IDLE" | "LOADING" | "SUCCESS" | "ERROR"
  error?: Error
}
```

이는 Redux가 비동기 데이터를 관리하기 위한 전문 라이브러리가 아니라, 범용적으로 사용할 수 있는 전역 상태 관리 라이브러리여서 생겨나는 현상입니다. Redux Middleware로 비동기 상태를 불러오고 그 값을 보관할 수는 있지만 내부적인 구현은 모두 개발자가 알아서 하다보니 상황에 따라 데이터를 관리하는 방식과 방법이 달라질 수 밖에 없습니다.

이러한 방식과 방법에 정답은 없지만, 팀의 구성원이 많아지고 협업 관계가 복잡하게 구성될 수록 자연스러운 방향으로 통일된다면 더 효율적인 업무가 가능할 것입니다. 더 나아가 팀 구성원들이 동일한 방법과 방식에 익숙해지고 숙련도가 높아진다면 새로운 Best Practice가 발굴되어 더 나은 제품을 만드는 기반이 될 수 있을 것입니다.

이처럼 `API 상태를 관리하기 위한 규격화된 방식`이 있다면 더 좋은 제품을 보다 효율적으로 만들 수 있을 것입니다. 다만 Redux를 사용하는 경우 구성원들의 환경과 경험이 다르고 프로젝트별 상황이 다르기 때문에 범용적인 방식을 발굴하기에 한계가 존재했습니다.

### 사용자 경험 향상을 위한 더 많은 해야 할 일

저희 카카오페이 프론트엔드 팀은 사용자 경험을 향상시키기 위해 다양한 노력들을 하고 있습니다. 웹뷰 기반 서비스에서의 사용자 행동 양상에 맞춰 Window의 Focusing 상태에 따라 데이터를 최신화하기도 하고, 사용자에게 빠르게 화면을 제공하기 위해 로컬 스토리지에 API 상태를 캐시하여 사용하기도 합니다. 또한 서버에 데이터를 전달해야하는 경우 Optimistic Update를 하기도 하고 API의 상태에 따라 스켈레톤과 에러 페이지를 복잡하게 표현하는 경우도 있습니다.

이러한 사용자 경험 향상을 위한 부가적인 기능들은 우리에게는 또 하나의 '업무'로만 인식되는 경우가 많습니다. 위에서 언급한 바와 같이 Redux는 비동기 데이터 관리 전문 라이브러리가 아니고, 비동기 데이터 관리에 특화된 기능들을 Redux에서 제공하지 않기 때문에 이런 기능의 개발은 오롯이 우리의 몫이기 때문입니다.

대표적인 시나리오를 하나 살펴보겠습니다. 카카오페이 프론트엔드 서비스는 대부분 "카카오톡"과 "카카오페이 앱" 내부의 웹뷰에서 동작합니다. 웹뷰 환경 특성상 사용자들은 애플리케이션을 Background로 내리고 시간이 지난뒤 Foreground로 올리는 사용 행태를 보이게 됩니다. 이런 사용자 행동 아래에서는 앞서 언급한 바와 같이 앱이 Foreground로 올라온 시점에 데이터의 동기화가 다시 수행되어야 사용자 경험이 향상될 수 있는 시나리오가 다수 존재합니다. Redux를 사용하여 비동기 데이터를 관리하는 경우 이러한 시나리오 처리를 위해 다음과 같은 코드를 개발자가 직접 작성해야만 했습니다.

```ts
// Todo.tsx
function Todo() {
  // ...전략
  useEffect(() => {
    const handleVisibilityChange = () => {
      if (document.visibilityState === 'visible') {
        dispatch(requestFetchTodos());
      }
    }
    // window focus 이벤트 발생시 Todo API 요청
    document.addEventListener('visibilitychange', handleVisibilityChange);
    return () => document.removeEventListener('visibilitychange', handleVisibilityChange);
  }, [dispatch]);
  return (
    // ...후략
  );
}
export default Todo;
```

사용자 경험 향상을 위한 시나리오 수행을 위해 위와 같은 코드를 개발자가 직접 구현하면 개발 리소스가 과다하게 소모되고, 프로젝트의 규모가 커지면 코드의 복잡도까지 높아져 유지보수에 대한 부담도 커지게 될 것입니다.

## React Query 소개

위와 같은 Redux를 사용한 API 요청과 비동기 데이터 관리의 불편함을 해소하기 위해 카카오페이 프론트엔드 개발자들은 전향적으로 React Query를 도입하여 사용하고 있습니다.

React Query는 React Application에서 서버 상태를 불러오고, 캐싱하며, 지속적으로 동기화하고 업데이트 하는 작업을 도와주는 라이브러리입니다. React Query는 우리에게 친숙한 Hook을 사용하여 React Component 내부에서 자연스럽게 서버(또는 비동기적인 요청이 필요한 Source)의 데이터를 사용할 수 있는 방법을 제안합니다.

길고 거창한 설명 없이도 아래 샘플 코드를 한번 살펴보시면 React Query를 사용한 API 요청과 상태 관리가 얼마나 쉽고 자연스러운지 알 수 있습니다.

```tsx
import axios from "axios"
import {
  QueryClient,
  QueryClientProvider,
  useMutation,
  useQuery,
  useQueryClient,
} from "react-query"
// React Query는 내부적으로 queryClient를 사용하여
// 각종 상태를 저장하고, 부가 기능을 제공합니다.
const queryClient = new QueryClient()
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Menus />
    </QueryClientProvider>
  )
}
function Menus() {
  const queryClient = useQueryClient()
  // "/menu" API에 Get 요청을 보내 서버의 데이터를 가져옵니다.
  const { data } = useQuery("getMenu", () =>
    axios.get("/menu").then(({ data }) => data)
  )
  // "/menu" API에 Post 요청을 보내 서버에 데이터를 저장합니다.
  const { mutate } = useMutation(
    (suggest) => axios.post("/menu", { suggest }),
    {
      // Post 요청이 성공하면 위 useQuery의 데이터를 초기화합니다.
      // 데이터가 초기화되면 useQuery는 서버의 데이터를 다시 불러옵니다.
      onSuccess: () => queryClient.invalidateQueries("getMenu"),
    }
  )
  return (
    <div>
      <h1> Tomorrow's Lunch Candidates! </h1>
      <ul>
        {data.map((item) => (
          <li key={item.id}> {item.title} </li>
        ))}
      </ul>
      <button
        onClick={() =>
          mutate({
            id: Date.now(),
            title: "Toowoomba Pasta",
          })
        }
      >
        Suggest Tomorrow's Menu
      </button>
    </div>
  )
}
```

React Query는 API 요청을 Query그리고 Mutation이라는 두가지 유형으로 나누어 생각합니다.

### React Query의 Query 요청

```ts
// 가장 기본적인 형태의 React Query useQuery Hook 사용 예시
const { data } = useQuery(
  queryKey, // 이 Query 요청에 대한 응답 데이터를 캐시할 때 사용할 Unique Key (required)
  fetchFn, // 이 Query 요청을 수행하기 위한 Promise를 Return 하는 함수 (required)
  options // useQuery에서 사용되는 Option 객체 (optional)
)
```

useQuery Hook으로 수행되는 Query 요청은 HTTP METHOD GET 요청과 같이 서버에 저장되어 있는 "상태"를 불러와 사용할 때 사용합니다.

> React Query는 다양한 UI에 유연하게 적용할 수 있도록 useQueries, useInfiniteQuery 같은 Hook들도 제공합니다.

React Query의 useQuery Hook은 요청마다 (API마다) 구분되는 Unique Key (aka. Query Key)를 필요로 합니다. React Query는 이 Unique Key로 서버 상태 (aka. API Response)를 로컬에 캐시하고 관리합니다.

```tsx
function Users() {
  const { isLoading, error, data } = useQuery(
    "userInfo", // 'userInfo'를 Key로 사용하여 데이터 캐싱
    // 다른 컴포넌트에서 'userInfo'를 QueryKey로 사용한 useQuery Hook이 있다면 캐시된 데이터를 우선 사용합니다.
    () => axios.get("/users").then(({ data }) => data)
  )
  // FYI, `data === undefined`를 평가하여 로딩 상태를 처리하는것이 더 좋습니다.
  // React Query는 내부적으로 stale-while-revalidate 캐싱 전략을 사용하고 있기 때문입니다.
  if (isLoading) return <div> 로딩중... </div>
  if (error) return <div> 에러: {error.message} </div>
  return (
    <div>
      {" "}
      {data?.map(({ id, name }) => (
        <span key={id}> {name} </span>
      ))}{" "}
    </div>
  )
}
```

```ts
function UserInfo({ userId }) {
  const { isLoading, error, data } = useQuery(
    // 'userInfo', userId를 Key로 사용하여 데이터 캐싱
    ['userInfo', userId],
    () => axios.get(`/users/${userId}`)
  );
  if (isLoading) return <div> 로딩중... </div>;
  if (error) return <div> 에러: {error.message} </div>;
  return <div> {...} </div>;
}
```

### React Query의 Mutation 요청

```ts
// 가장 기본적인 형태의 React Query useMutation Hook 사용 예시
const { mutate } = useMutation(
  mutationFn, // 이 Mutation 요청을 수행하기 위한 Promise를 Return 하는 함수 (required)
  options // useMutation에서 사용되는 Option 객체 (optional)
)
```

useMutation Hook으로 수행되는 Mutation 요청은 HTTP METHOD, POST, PUT, DELETE 요청과 같이 서버에 SideEffect를 발생시켜 서버의 상태를 변경시킬 때 사용합니다.

useMutation Hook의 첫번째 파라미터는 이 Mutation 요청을 수행하기 위한 Promise를 Return 하는 함수이며, useMutation의 return 값 중 mutate(또는 mutateAsync) 함수를 호출하여 서버에 Side Effect를 발생시킬 수 있습니다.

```tsx
function NotificationSwitch({ value }) {
  // mutate 함수를 호출하여 mutationFn 실행
  const { mutate, isLoading } = useMutation(
    (value) => axios.post(URL, { value }) // mutationFn
  )
  return (
    <Switch
      checked={value}
      disabled={isLoading}
      onChange={(checked) => {
        // mutationFn의 파라미터 'value'로 checked 값 전달
        mutate(checked)
      }}
    />
  )
}
```

### React Query를 이용한 비동기 데이터 동기화 기능을 갖춘 Todo List Application

Redux 섹션에서 나왔던 비동기 데이터 동기화 기능을 갖춘 Todo List Application을 React Query를 사용하게끔 변경해 보겠습니다.

```tsx
// Todo.tsx
import useTodosMutation from "quires/useTodosMutation"
import useTodosQuery from "quires/useTodosQuery"
import { useForm } from "react-hook-form"
function Todo() {
  // 서버에서 저장되어 있는 Todo 정보를 사용하기 위한 Custom Hook
  const { data } = useTodosQuery()
  // 서버에 새로운 Todo 정보를 저장하기 위한 Custom Hook
  const { mutate } = useTodosMutation()
  const { register, handleSubmit } = useForm<{
    contents: string
  }>()
  const onSubmit = handleSubmit((value) => {
    // useTodosMutation의 'mutate' 함수를 사용하여 서버로 데이터를 전송합니다.
    mutate(value.contents)
  })
  return (
    <div>
      <header>
        <form onSubmit={onSubmit}>
          <input
            {...register("contents")}
            type="text"
            placeholder="What needs to be done?"
            autoComplete="off"
          />
        </form>
      </header>
      <div>
        <ul>
          {data?.map(({ id, contents }) => (
            <li key={id}> {contents} </li>
          ))}
        </ul>
      </div>
    </div>
  )
}
export default Todo
```

```tsx
// quires/useTodosQuery.ts
import axios from "axios"
import { useQuery } from "react-query"
import { TodoItem } from "types/todo"
// useQuery에서 사용할 UniqueKey를 상수로 선언하고 export로 외부에 노출합니다.
// 상수로 UniqueKey를 관리할 경우 다른 컴포넌트 (or Custom Hook)에서 쉽게 참조가 가능합니다.
export const QUERY_KEY = "/todos"
// useQuery에서 사용할 `서버의 상태를 불러오는데 사용할 Promise를 반환하는 함수`
const fetcher = () => axios.get<TodoItem[]>("/todos").then(({ data }) => data)
const useTodosQuery = () => {
  return useQuery(QUERY_KEY, fetcher)
}
export default useTodosQuery
```

```tsx
// quires/useTodosMutation.ts
import axios from "axios"
import { useMutation, useQueryClient } from "react-query"
import { QUERY_KEY as todosQueryKey } from "./useTodosQuery"
// useMutation에서 사용할 `서버에 Side Effect를 발생시키기 위해 사용할 함수`
// 이 함수의 파라미터로는 useMutation의 `mutate` 함수의 파라미터가 전달됩니다.
const fetcher = (contents: string) => axios.post("/todos", { contents })
const useTodosMutation = () => {
  // mutation 성공 후 `useTodosQuery`로 관리되는 서버 상태를 다시 불러오기 위한
  // Cache 초기화를 위해 사용될 queryClient 객체
  const queryClient = useQueryClient()
  return useMutation(fetcher, {
    // mutate 요청이 성공한 후 queryClient.invalidateQueries 함수를 통해
    // useTodosQuery에서 불러온 API Response의 Cache를 초기화
    onSuccess: () => queryClient.invalidateQueries(todosQueryKey),
  })
}
export default useTodosMutation
```

프로젝트의 규모가 작음에도 불구하고 많은 부분이 달라졌는데요. 위에서 다루었던 "Redux로 API 요청 및 비동기 데이터 관리 시 불편했던 점"에 주목하여 어떤 부분이 바뀌었는지 한번 살펴보겠습니다.

### React Query를 쓰고 이런게 편해졌다.

- Boilerplate 코드의 감소

앞에서 언급한대로, Redux를 사용할 경우 Redux의 기본 원칙 준수를 위한 다양한 Boilerplate 코드들이 필요합니다.
더 나아가 API 상태 관리를 위해 하나의 API 요청을 3가지 Action을 사용해 처리하고 있고, 후에 기능이 추가되어 API 개수가 많아진다면 이런 상용구적인 코드도 함께 늘어나게 됩니다.

> API 상태를 Redux + Saga를 사용하여 관리하는 부분의 코드

```tsx
;/ features/doost / todos.slice.ts
// API 상태를 관리하기 위한 Redux State
import { createSlice, PayloadAction } from "@reduxjs/toolkit"
import { TodoItem } from "types/todo"
export interface TodoListState {
  data?: TodoItem[]
  isLoading: boolean
  error?: Error
}
const initialState: TodoListState = {
  data: undefined,
  isLoading: false,
  error: undefined,
}
export const todoListSlice = createSlice({
  name: "todoList",
  initialState,
  reducers: {
    requestFetchTodos: (state) => {
      state.isLoading = true
    },
    successFetchTodos: (state, action: PayloadAction<TodoItem[]>) => {
      state.data = action.payload
      state.isLoading = false
      state.error = undefined
    },
    errorFetchTodos: (state, action: PayloadAction<string>) => {
      state.data = undefined
      state.isLoading = false
      state.error = action.payload
    },
  },
})
export const { requestFetchTodos, successFetchTodos, errorFetchTodos } =
  todoListSlice.actions
export default todoListSlice.reducer
```

```tsx
// features/todos/todos.saga.ts
import { PayloadAction } from "@reduxjs/toolkit"
import axios from "axios"
import { call, put, takeEvery } from "redux-saga/effects"
import { TodoItem } from "../../types/todo"
import {
  errorFetchTodos,
  errorPostTodos,
  requestFetchTodos,
  requestPostTodos,
  successFetchTodos,
  successPostTodos,
} from "./todos.slice"

async function getTodoList() {
  const { data } = await axios.get<TodoItem[]>("./todos")
  return data
}
function* requestFetchTodoTask() {
  try {
    const data: TodoItem[] = yield call(getTodoList)
    yield put(successFetchTodos(data))
  } catch (e) {
    yield put(errorFetchTodos(e.message))
  }
}
async function postTodoList(contents: string) {
  await axios.post("/todos", { contents })
}
function* requestPostTodoTask(action: PayloadAction<string>) {
  try {
    yield call(postTodoList, action.payload)
    yield put(successPostTodos())
  } catch (e) {
    yield put(errorPostTodos(e.message))
  }
}
function* successPostTodoTask() {
  // 서버에 새로운 Todo 추가 요청 성공 시
  // 서버에서 Todo 목록을 다시 받아오기 위해 Action Dispatch
  yield put(requestFetchTodos())
}
function* todoListSaga() {
  yield takeEvery(requestFetchTodos.type, requestFetchTodoTask)
  yield takeEvery(requestPostTodos.type, requestPostTodoTask)
  yield takeEvery(successPostTodos.type, successPostTodoTask)
}
export default todoListSaga
```

> React Query로 API 상태를 관리하는 부분의 코드

```tsx
// quires/useTodosQuery.ts
// API 상태를 불러오기 위한 React Query Custom Hook
import axios from "axios"
import { useQuery } from "react-query"
import { TodoItem } from "types/todo"
// useQuery에서 사용할 UniqueKey를 상수로 선언하고 export로 외부에 노출합니다.
// 상수로 UniqueKey를 관리할 경우 다른 Component (or Custom Hook)에서 쉽게 참조가 가능합니다.
export const QUERY_KEY = "/todos"
// useQuery에서 사용할 `서버의 상태를 불러오는데 사용할 Promise를 반환하는 함수`
const fetcher = () => axios.get<TodoItem[]>("/todos").then(({ data }) => data)
const useTodosQuery = () => {
  return useQuery(QUERY_KEY, fetcher)
}
export default useTodosQuery
```

단순히 비교해봐도 Redux를 사용한 비도익 데이터 관리 코드와 React Query를 사용한 비동기 데이터 관리 코드의 분량이 크게 차이남을 알 수 있습니다. 코드의 분량이 적어졌다는 것은 개발자에게 불필요한 작업이 필요 없어짐을 뜻하기도 하지만, 소스코드의 복잡도를 낮추어 유지보수의 용이성을 높이고 작업 간에 발생할 수 있는 사이드 이펙트나 휴먼에러를 사전에 더 잘 막을 수 있다는 의미도 갖게 될 것입니다.

- API 요청 수행을 위한 규격화된 방식 제공

앞에서 말씀드린 바와 같이 Redux는 비동기 데이터 관리를 위한 라이브러리가 아닙니다. Redux로 비동기 데이터를 관리하기 위해서 개발자들은 Middleware 부터 State구조까지 다양한 부분을 설계하고 구현해야했습니다. 이러한 상황은 우리에게 하여금 불필요한 고민을 하게 만들고, 커뮤니케이션 비용을 증가시키는 요인으로 작동하기도 했습니다. 대부분의 케이스에 대응할 수 있는 편리하고 규격화된 방식을 제공한다면 이런 비효율적인 요소를 줄여 더 나은 제품을 만드는 방법에 집중할 수 있을 것입니다.

React Query는 React에서 비동기 데이터를 관리하기 위한 라이브러리입니다. React Query는 API 요청 및 상태 관리를 위해(상당히 잘 만들어진)규격화된 방식을 제공합니다.

```ts
interface ApiState {
  data?: Data
  isLoading: boolean
  error?: Error
}
interface ApiState {
  data?: Data
  status: "IDLE" | "LOADING" | "SUCCESS" | "ERROR"
  error?: Error
}
```

> Redux로 API 상태를 관리하는 경우 프로젝트 환경에 따른 설계와 구현이 요구되었습니다.

React Query는 API 상태와 관련된 다양한 데이터를 제공하여 복잡한 구현과 설계 없이도 개발자가 효율적으로 화면을 구성할 수 있게끔 도와줍니다.

```ts
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
  isIdle,
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
} = useQuery(queryKey, queryFn)
```

- 사용자 경험 향상을 위한 기능 제공

저희 카카오페이 프론트엔드 팀은 사용자 경험 향상을 위해 다양한 기법을 사용하고 있습니다. Redux로 비동기 데이터 관리를 할 때는 직접 구현해서 사용하곤 했는데, React Query는 자체적으로 제공하는 다양한 기능이 있어 이를 사용자 경험 향상에 손쉽게 사용할 수 있었습니다.

```tsx
// Todo.tsx
function Todo() {
  const dispatch = useDispatch();
  // ...전략
  useEffect(() => {
    const handleVisibilityChange = () => {
      if (document.visibilityState === 'visible') {
        dispatch(requestFetchTodos());
      }
    }
    // window focus 이벤트 발생시 Todo API 요청
    document.addEventListener('visibilitychange', handleVisibilityChange);
    return () => document.removeEventListener('visibilitychange', handleVisibilityChange);
  }, [dispatch]);
  return (
    // ...후략
  );
}
export default Todo;
```

앞에서 다룬 바와 같이 웹뷰 환경에서 사용자 경험 향상을 위해 Window Focus 이벤트 발생시 서버 상태를 동기화해야하는 시나리오가 있다고 가정했을 때, Redux로 비동기 데이터 관리시에는 React Component단에서 Window Focus이벤트에 Dispatch Action을 직접 바인딩하여 구현해야 했습니다. 만약 소수의 컴포넌트에 이런 작업이 필요하다면 기꺼이 작업을 할 수 있겠지만, 여러 컴포넌트에서 여러 API에 걸쳐 이런 작업을 수행해야 한다면 유지보수 등 다양한 관점에서 부담스럽게 다가올 것입니다.

```ts
// quires/useTodosQuery.ts
// API 상태를 불러오기 위한 React Query Custom Hook
// ...전략
const useTodosQuery = () => {
  return useQuery(QUERY_KEY, fetcher, { refetchOnWindowFocus: true })
}
export default useTodosQuery
```

React Query를 사용할 경우 단순한 옵션 부여만으로 Window Focus 이벤트 발생 시 서버 상태 동기화 시나리오를 달성할 수 있습니다. 다루는 API가 많아지고 컴포넌트 구조가 복잡해질수록 이전의 직접 Event Binding 하는 방식보다 유지보수하기 좋은 코드가 될 것입니다.

React Query와 함께라면 이 아티클에서 다룬 Refetch on window focus외에 API Caching, API Retry, Optimistic Update, Persist Caching등 사용자 경험 향상을 위한 다양한 기법들을 손쉽게 프로젝트에 포함시킬 수 있습니다.

React Query에서 제공하는 이러한 기능들은 우리 개발자들로 하여금 제품과 직접적으로 연관되지 않는 작업에 투입해야 하는 리소스를 경감시켜 더 중요한 비즈니스 로직에 집중할 수 있게끔 도와줍니다. 이러한 환경은 우리가 더 견고한 제품을 만들 수 있는 바탕이 되어주고 있습니다.

## 마치며

지금까지 "불필요한 코드의 감소", "업무와 협업의 효율성을 규격화된 방식 제공", "사용자 경험 향상을 위한 다양한 Built-in 기능" 이라는 세가지 꼭지로 저희가 Redux 대신 React Query를 사용하여 비동기 데이터를 처리하는 이유에 대해 이야기해보았습니다. 하지만 카카오페이에 React Query가 도입되고 1년이 지난 지금에는 위 세가지 이유보다 저희가 더 매력적으로 느끼고 적극적으로 활용하고 있는 React Query 활용법이 존재합니다.

이어지는 [카카오페이 프론트엔드 개발자들이 React Query와 함께 Concurrent UI Pattern을 도입하는 방법](https://tech.kakaopay.com/post/react-query-2/)에서 Concurrent UI Pattern의 개념과 그 활용 사례에 대해 살펴보겠습니다.
