# Redux 어떻게 써야 잘 썼다고 소문이 날까?

리액트 생태계에는 다양한 상태관리 라이브러리들이 있다. 그 중에서, 리덕스(Redux)가 가장 많이 사용되고 있다.

리덕스의 사용방식에 있어서는 딱 정해진 규칙이 존재하지 않기 때문에 개발자들 모두 자신만의 방식으로 서로 다르게 사용을 하고 있다. 자신의 취향에 따라 사용할 수 있다는 것이 장점이기도한데, 단점이기도 하다. 제대로 쓰지 않으면 굉장히 불편할 수 있고, 유지보수에 있어서 오히려 독이 될 수 있다.

## 정말 리덕스가 필요할까?

리덕스가 필요한 프로젝트에서는 리덕스를 아주 유용하게 사용할 수 있지만, 그렇지 않은 프로젝트에선 그저 개발을 귀찮게 만드는 짐이 될 뿐이다. 따라서 리덕스를 사용하기 전에는 꼭 현재 여러분의 프로젝트에 리덕스가 정말 최고인지 고민을 할 필요가 없다.

리액트 개발의 초창기(2015~2018)때는 리액트 프로젝트에서 리덕스를 사용하는 것이 당연시되어 왔다. 하지만 이제 그럴 필요가 없다. 리액트 자체적인 기능만을 사용해서 리덕스 없이 프로젝트를 충분히 개발할 수 있고 다른 라이브러리의 도움을 받아 훨씬 편하게 개발할 수 있는 방법들이 존재한다.

### Context API

단순히 글로벌 상태를 사용하고자 한다면 리액트의 ContextAPI 만으로도 충분히 구현을 할 수 있다.

Context는 리덕스의 비교 대상이 아니다. Context는 수단일 뿐 사실상 상태관리 자체는 리액트 컴포넌트의 useState와 useReducer로 하게되는 것이다.

리덕스와의 주요 차이는 성능면에서 나타나게 된다. 리덕스에서는 컴포넌트에서 글로벌 상태의 특정 값을 의존하게 될 때 해당 값이 바뀔 때에만 리렌더링이 되도록 최적화가 되어있다. 따라서 글로벌 상태 중 의존하지 않는 값이 바뀌게 될 때에는 컴포넌트에서 낭비 렌더링이 발생하지 않는다. 반면 Context에는 이러한 성능 최적화가 이뤄지지 않았다. 컴포넌트에서 만약 Context의 특정 값을 의존하는 경우, 해당 값 말고 다른 값이 변경될 때에도 컴포넌트에서는 리렌더링이 발생하게 된다.

따라서 Context를 사용하게 될 때에는 관심사의 분리가 굉장희 중요하다. 서로 관련이 없는 상태라면 같은 Context에 있으면 안도니다. Context를 따로 만들어주어야한다.

추가적으로 Context에서 상태를 업데이트를 하는 상황에서도 성능적으로 고려해야될 부분이 있다. 우리가 Provider를 만들게 될 때 다음과 같이 상태와, 상태를 업데이트하는 함수를 value에 넣는 상황이 발생하기도 한다.

```js
import React, { createContext, useState, useContext } from "react"

const UserContext = createContext(null)

function UserProvider({ children }) {
  const [user, setUser] = useState(null)
  return (
    <UserContext.Provider value={{ user, setUser }}>
      {children}
    </UserContext.Provider>
  )
}

function useUser() {
  return useContext(UserContext)
}

function UserInfo() {
  const { user } = useUser()
  if (!user) return <div>사용자 정보가 없습니다.</div>
  return <div>{user.username}</div>
}

function Authenticate() {
  const { setUser } = useUser()
  const onClick = () => {
    setUser({ username: "velopert" })
  }
  return <button onClick={onClick}>사용자 인증</button>
}

export default function App() {
  return (
    <UserProvider>
      <UserInfo />
      <Authenticate />
    </UserProvider>
  )
}
```

현재 UserInfo 컴포넌트에서는 Context에 담긴 사용자의 정보를 보여주고 있고, Authenticate 컴포넌트의 경우엔 사용자 상태를 업데이트하는 버튼을 보여주고 있다. 여기서 사용자가 버튼을 누르게 됐을 때, UserInfo 컴포넌트가 리렌더링이 되는 것은 당연하다. 문제는 Authenticate도 함께 리렌더링이 된다는 것이다. setUser함수가 바뀌지 않았는데도 말이다.

물론 위와 같은 상황에서는 워낙 소규모의 뷰가 담겨져있는 컴포넌트이고 업데이트도 1회성이기 때문에 실질적으로 성능적으로 큰 문제는 없다. 리액트는 충분히 괜찮은 성능을 갖추고 있기 때문이다.

하지만, 상태도 다양해지고, 뷰도 다양해지게 되면 위와 같은 구조로 Context를 사용하게 되면 낭비되는 렌더링이 너무 많이 발생하게 되어 성능적으로 좋지 못하다.

따라서, 이를 해결하기 위해서는 다음과 같이 두개의 Context를 사용해야한다.

```js
import React, { createContext, useState, useContext } from "react"

const UserContext = createContext(null)
const UserUpdateContext = createContext(null)

function UserProvider({ children }) {
  const [user, setUser] = useState(null)
  return (
    <UserContext.Provider value={user}>
      <UserUpdateContext.Provider value={setUser}>
        {children}
      </UserUpdateContext.Provider>
    </UserContext.Provider>
  )
}

function useUser() {
  return useContext(UserContext)
}

function useUserUpdate() {
  return useContext(UserUpdateContext)
}

function UserInfo() {
  const user = useUser()
  if (!user) return <div>사용자 정보가 없습니다.</div>
  return <div>{user.username}</div>
}

function Authenticate() {
  const setUser = useUserUpdate()
  const onClick = () => {
    setUser({ username: "velopert" })
  }
  return <button onClick={onClick}>사용자 인증</button>
}

export default function App() {
  return (
    <UserProvider>
      <UserInfo />
      <Authenticate />
    </UserProvider>
  )
}
```

위와 같이 UserUpdateContext를 만들어서 상태를 위한 Context와 상태 업데이트를 위한 Context를 따로 사용하게 된다면 성능적인 부분이 많이 해소가 된다. 이제는 버튼을 눌러도 Authenticate 컴포넌트는 리렌더링을 하지 않는다.

Context를 사용한다면 관심사를 분리하는 것도 중요하고, 이렇게 업데이트용과 상태용 Context를 분리하는 것도 중요하다. 전역적으로 관리해야하는 상태가 많아지는 경우에는 성능을 챙기기 위해선 그만큼 다양한 종류의 Context를 위한 코드를 준비해야한다. 따라서 글로벌 상태가 다양해지는 경우 Context의 사용은 적합하지 않을 수 있다.

### Constate

만약 Context를 사용하고 싶은데, 다뤄야하는 상태가 많아진 상황에서 성능도 챙겨가면서 개발도 편하게 하고 싶다면 constate라는 라이브러리가 있다.

이 라이브러리는 Context를 기반으로 작동하느넫, 우리가 아까봤던 두번째 예시처럼 상태를 위한 Context와 상태 업데이트를 위한 Context를 따로 만들었던 작업을 하나의 함수로 간편하게 처리할 수 있게 해준다.

이 라이브러리를 사용하면 아까 작성했던 코드를 다음과 가팅 구현할 수 있다.

```js
import React, { useState } from "react"
import constate from "constate"

// Context를 따로 만들지 않고, 관리하고 싶은 상태를 위한 Hook 작성
function useUser() {
  const [user, setUser] = useState(null)
  return { user, setUser }
}

// useUser Hook을 기반으로 종류별로 Context를 만들고,
// 해당 Context를 사용하는 Provider와 Hook 생성
const [UserProvider, useUserValue, useUserUpdate] = constate(
  useUser,
  (value) => value.user,
  (value) => value.setUser
)

function UserInfo() {
  const user = useUserValue()
  if (!user) return <div>사용자 정보가 없습니다.</div>
  return <div>{user.username}</div>
}

function Authenticate() {
  const setUser = useUserUpdate()
  const onClick = () => {
    setUser({ username: "velopert" })
  }
  return <button onClick={onClick}>사용자 인증</button>
}

export default function App() {
  return (
    <UserProvider>
      <UserInfo />
      <Authenticate />
    </UserProvider>
  )
}
```

constate 함수에 넣는 selector들을 기반으로 Context들이 자동으로 만들어지고, 해당 Context를 사용하는 Hook도 자동으로 만들어진다. 이 라이브러리를 사용하면 Context를 분리하는 작업이 엄청 쉬워진다.

상태 관리 측면에서 useState, useReducer를 그냥 사용하면 되기 때문에 추가적으로 배울 것도 없다.

### Recoil

Context만을 사용하여 글로벌 상태 관리를 하는 것이 어느 정도 제한이 있다는 것은 페이스북 개발팀에서도 인지를 하고 있다.

- 상태를 업데이트 하기 위해서는 공통 조상 컴포넌트로 상태를 끌어올려야하는데, 이 과정에서 너무 큰 트리가 리렌더링 될 수 있다.
- Context를 사용할 때 Consumer는 다양하게 사용되는데 Context에는 하나의 값만 담을 수 있다.
- 위 두가지 방식 모두 상태가 만들어지는 곳과 상태가 사용되는 곳의 코드 분리가 어렵다.

Recoil은 페이스북 개발팀에서 위 문제들을 리액트스러운 방법으로 개선하기 위하여 만든 상태 관리 라이브러리다.

이 라이브러리의 AtomEffect기능은 아직 개발중이며 나중에 스펙이 바뀔 수 있으니 주의가 필요하지만, 나머지 기능은 많이 안정화가 되어있는 상태이기 때문에 나머지 기능은 프로덕션에서 사용해도 큰 문제가 없다.

Recoil을 사용해서 아까 작성했던 기능을 구현하면 다음과 같이 사용할 수 있다.

```js
import React from "react"
import { RecoilRoot, atom, useSetRecoilState, useRecoilValue } from "recoil"

// 특정 상태를 고유 key 값과 함께 선언합니다
const userState = atom({
  key: "userState",
  default: null,
})

function UserInfo() {
  // useRecoilValue를 사용하면 원하는 상태 값을 조회 할 수 있습니다.
  const user = useRecoilValue(userState)
  if (!user) return <div>사용자 정보가 없습니다.</div>
  return <div>{user.username}</div>
}

function Authenticate() {
  // useSetRecoilState를 사용하면 원하는 상태 업데이터 함수를 가져올 수 있습니다.
  const setUser = useSetRecoilState(userState)
  const onClick = () => {
    setUser({ username: "velopert" })
  }
  return <button onClick={onClick}>사용자 인증</button>
}

export default function App() {
  return (
    <RecoilRoot>
      <UserInfo />
      <Authenticate />
    </RecoilRoot>
  )
}
```

위 코드에서 나타난 기능들 외에도, 상태에서 특정 값만을 조회하는 selector 기능과, 상태와 업데이트를 한꺼번에 가져올 수 있는 useRecoilState라는 기능도 있다.

```js
import { atom, selector, useRecoilValue, useRecoilState } from "recoil;"

const messageState = atom({
  key: "messageState",
  default: "",
})

const messageLengthState = selector({
  key: "messageLengthState",
  get: ({ get }) => get(messageState).length,
})

function MessageInput() {
  // 상태와 업데이터를 한번에 가져오기
  const [message, setMessage] = useRecoilState(messageState)
  return <input value={message} onChange={(e) => setMessage(e.target.value)} />
}

function MessageLength() {
  // selector를 통하여 상태의 특정 부분만을 조회
  const length = useRecoilValue(messageLengthState)
  return <div>Length: {length}</div>
}
```

## Jotai

Jotia에서는 Minimalistic API를 강조한다. Recoil과 비슷하지만 훨씬 간단하다.
Jotai는 다음과 같이 사용할 수 있다.

```js
import React from "react"
import { atom, Provider, useAtom } from "jotai"

// Jotai에서는 모든걸 'atom' 이라고 부릅니다

// null 을 기본 상태로 가지는 atom
const userAtom = atom(null)
// 업데이트 함수만 사용 할 수 있게 해주는 atom
const updateUserAtom = atom(null, (get, set, arg) => set(userAtom, arg))
// 문자열 상태를 지닌 atom
const messageAtom = atom("")
// 문자열의 length 를 조회하는 atom
const messageLengthAtom = atom((get) => get(messageAtom).length)

function UserInfo() {
  // 값을 조회 할 때는 useAtom 을 사용하며, 반환 값은 배열입니다
  const [user] = useAtom(userAtom)
  if (!user) return <div>사용자 정보가 없습니다.</div>
  return <div>{user.username}</div>
}

function Authenticate() {
  // 업데이트 함수만 사용하니까, 첫번째 배열 원소는 생략합니다
  const [, setUser] = useAtom(updateUserAtom)
  const onClick = () => {
    setUser({ username: "velopert" })
  }
  return <button onClick={onClick}>사용자 인증</button>
}

function MessageInput() {
  // 상태와 업데이터를 한번에 가져오기
  const [message, setMessage] = useAtom(messageAtom)
  return <input value={message} onChange={(e) => setMessage(e.target.value)} />
}

function MessageLength() {
  // selector를 통하여 상태의 특정 부분만을 조회
  const [length] = useAtom(messageLengthAtom)
  return <div>Length: {length}</div>
}

export default function App() {
  return (
    <Provider>
      <UserInfo />
      <Authenticate />
      <div>
        <MessageInput />
        <MessageLength />
      </div>
    </Provider>
  )
}
```

## 언제 리덕스가 필요할까?

이제는 글로벌 상태 관리에 있어서 대체로 사용할 수 있는 방법들에 대해서는 그만 알아보겠다. 정말 다양한 선택지가 있는데 그렇다면 언제 리덕스를 사용해야할까?

1. 리덕스를 사용한 개발 스타일이 너무 마음에 들 때

모든 상태 업데이트를 액션으로 정의하고, 액션 정보에 기반하여 리듀서에서 상태를 업데이트하는 이 간단 명료한 방상 덕분에, 상태를 더욱 쉽게 예측 가능하게 하여 유지보수 측면에 긍정적인 효과가 있다.

2. 미들웨어

리덕스와 다른 라이브러리들의 차이점은 미들웨어가 존재한다는 것이다. 특정 액션이 디스패치 됐을 때 상태 업데이트외의 다른 작업들을 처리할 수 있다. 보통 API 요청을 할 때 미들웨어를 사용하곤한다. 이전에는 API 요청을 위하여 리덕스와 미들웨어르 ㄹ사용하는 것이 당연시 되긴 했는데, 이제는 SWR, react-query와 같은 라이브러리가 있기 때문에 단순 API 요청을 위하여 미들웨어를 사용할 필요는 없다. 그렇지만 비동기 작업에 대한 플로우에 대하여 더 많은 컨트롤을 필요로 할 때 미들웨어는 정말 유용하게 사용될 수 있다.

3. 서버 사이드 렌더링

리덕스를 사용하면 API 요청 결과를 사용하여 서버사이드 렌더링을 하는 것이 용이하다. 이 과정에서 미들웨어가 정말 유용하게 사용된다. 리덕스가 없어도 충분히 구현할 수 있기는 하지만 레퍼런스도 부족하고 번거롭다.

4. 더 쉬운 테스팅

리덕스를 사용하는 앱은 테스트를 하기가 비교적 쉽다. 리듀서에서 다양한 상태 업데이트 로직을 테스트하기도 쉽고, 리덕스와 연동된 컴포넌트를 테스트할 때 Mocking할 수도 있고 미들웨어 작동방식도 Mocking할 수 있다.

5. 컴포넌트가 아닌 다른 곳에서 글로벌 상태를 사용하거나 업데이트 해야할 때

WebSocket을 사용한다거나, 리액트 네이티브 브릿지에서 연동을 할 때 getState 또는 dispatch를 바로 호출해서 사용하면 꽤 유용한 상황이 있기도 하다.

6. 그냥 많이 사용돼서

많은 개발자가 리덕스를 사용하는 이유중엔, 이미 유지보수를 하고 있는 프로젝트에서 리덕스를 사용중이기 때문에 확률이 크다고 생각한다. 분명히 과거에는 선택지가 별로 없고 MobX가 그나마 유일했엇기 때문이다. 프로젝트에 리덕스가 필수적이라고 느껴지지 않는다고 해서 아예 걷어내는건 또 큰 공수가 드니 계속 유지하면서 사용하는 케이스도 많을 것이라고 생각한다. 다만, 그러한 경우엔 새로운 기능 또는 리팩토링 하는 기능에 있어선 다른 방식을 시도해보는 것도 좋을 것이라 판단한다.

## Redux Toolkit은 이제 필수템이다.

리덕스를 사용할 필요가 없다면 사용하지 않는 것이 옳고, 만약 사용을 해야한다면 어떻게 사용해야 좋을까?

리덕스의 단점 중에선 보일러 플레이트 코드를 참 많이 준비해야한다는 것이다. 액션 타입, 액션 생성함수, 리듀서 이렇게 3가지 종류로 코드를 준비해얗나다.

```js
export const OPEN = 'msgbox/OPEN';
export const CLOSE = 'msgbox/CLOSE';

export const open = (message) => ({ type: OPEN, message });

const initialState = {
  open: false,
  message: '',
};

export default msgbox(state = initialState, action) {
  switch (action.type) {
	case OPEN:
      return { ...state, open: true, message: action.message };
    case CLOSE:
      return { ...state, open: false };
    default:
      return state;
  }
}
```

이러한 작업은 적응하면 괜찮긴 한데 프로젝트가 커질수록 이 작업이 귀찮아진다. 그리고, 하나의 리듀서에서 관리하는 상태가 커지고, 세부적인 업데이트가 늘어날수록 불변성을 지키기 위하여 ...state를 사용하는 것도 번거롭다.

Redux Toolkit을 사용하면 리듀서, 액션타입, 액션 생성함수, 초기상태를 하나의 함수로 편하게 선언할 수 있다. 이 라이브러리에선 이 4가지를 통틀어서 slice라고 부른다.

만약, 위 코드에서 봤던 메시지 박스를 띄우는 기능에 대한 slice를 생성한다면, 다음과 같이 작성한다.

```js
import { createSlice } from "@reduxjs/toolkit"

const msgboxSlice = createSlice({
  name: "msgbox",
  initialState: {
    open: false,
    message: "",
  },
  reducers: {
    open(state, action) {
      state.open = true
      state.message = action.payload
    },
    close(state) {
      state.open = false
    },
  },
})

export default msgboxSlice

// reducer: msgboxSlice.reducer
// action creators: msgboxSlice.actions.open, msgboxSlice.actions.close
// actionType:
// - msgboxSlice.actions.open.type: 'msgbox/open'
// - msgboxSlice.actions.close.type: 'msgbox/close'
```

이 라이브러리를 사용하면 이렇게 리듀서와 액션 생성 함수를 한방에 만들 수 있다. 그리고 immer가 내장되어있기 때문에, 불변성을 유지하기 위하여 번거로운 코드를 작성하지 않고 원하는 값을 직접 변경하면 알아서 불변성 유지되면서 상태가 업데이트 된다.

TypeScript에서 리덕스를 사용할 때, 리액트 컴포넌트에서 리덕스 상태를 조회하는 경우 다음과 같이 작성한다.

```js
const something = useSelector((state: RootState) => state.some.thing)
```

이 때 RootState 타입을 준비할 때는 다음과 같이하면 편하다.

```js
const rootReducer = combineReducers({ ... })
export type RootState = ReturnType<typeof rootReducer>;
```

그리고 상태를 조회할 때마다 useSelector와 Rootstate를 불러오는데, 이 과정이 귀찮다면 다음과 같이 따로 Hook으로 만들어서 사용한다.

```js
type StateSelector<T> = (state: RootState) => T
type EqualityFn<T> = (left: T, right: T) => boolean

export function useRootState<T>(
  selector: StateSelector<T>,
  equalityFn?: EqualityFn<T>
) {
  return useSelector(selector, equalityFn)
}
```

그럼 나중에 컴포넌트에서 다음과 같이 사용할 수 있다.

```js
function MyComponent() {
  const something = useRootState((state) => state.some.thing)
  // ...
}
```

## Selector 사용하기

리덕스를 사용하여 프로젝트를 개발하시는 분들은 reselect라는 라이브러리를 들어봤을 것이다. 이 라이브러리를 사용하면 우리가 원하는 상태를 조회하는 과정에서 memoization을 할 수 있다. 이 기능은 Redux Toolkit에도 내장되어 createSelector를 통해 사용할 수 있다.

리덕스에서 selector는 필수적인 요소는 아니지만 사용을 한다면 다음 두가지 상황에 유용하게 사용될 수 있다.

- 상태의 위치 (key)가 변경될 때
- 컴포넌트 리렌더링을 최적화할 때

### 상태의 위치가 변경되는 상황

리덕스에서 다음과 같이 상태를 지니고 있다고 가정해보자.

```js
{
  user: {
    id: 1,
    username: 'velopert',
    displayName: 'MinJun'
  },
  settings: { /* ... */ }
}
```

컴포넌트에서 user 값을 사용한다면 다음과 같이 작성할 것이다.

```js
const user = useSelector((state) => state.user)
```

그런데 어느날 서비스 로직이 좀 많이 변경돼서 이 상태의 위치를 다음과 같이 변경해야된다고 해보자.

```js
{
  auth: {
	  isCertified: false,
    user: {
      id: 1,
      username: 'velopert',
      displayName: 'MinJun',
    }
  },
  settings: { /* ... */ }
}
```

state.user가 더이상 존재하지 않고 state.auth.user를 조회해야하는 상황이 있는데, 이렇게 되면 기존에 state.user에 의존하던 모든 코드들을 찾아서 변경해주어야한다.

하지만 selector를 따로 만들어서 사용했더라면 이야기가 좀 달라진다.

이렇게 기존에 useSelector의 인자에 넣었었던 함수를 따로 선언하고

```js
export const userSelector = (state) => state.user
```

추후 사용할 땐 이렇게 불러와서 사용했더라면

```js
const user = useSelector(userSelector)
```

만약 상태의 위치가 변경되는 상황이 왔을 때 selector만 다음과 같이 변경하면 나머지는 자동으로 반영이 된다.

```js
export const userSelector = (state) => state.auth.user
```

### 리렌더링 최적화를 Memoized Selector 사용하기

상태의 위치가 변경될 때 사용하면 유용하긴 하지만 IDE의 Find & Replace 기능을 사용하면 쉽게 반영 할 수 있기 때문에 상태를 조회할 때 selector를 만드는 것은 불필요한 추가 절차라고 느껴지기도 한다.

하지만, 리렌더링 최적화를 해야하는 상황에서는 정말 유용하게 사용될 수 있다.
다음과 같이 todos 배열을 상태로 지닌 리듀서가 있다고 가정해보자.

```js
;[
  { id: 1, text: "책 읽기", done: true },
  { id: 2, text: "블로그 글 쓰기", done: true },
  { id: 3, text: "운동하기", done: false },
  { id: 4, text: "요리하기", done: false },
]
```

여기서 특정 컴포넌트에서 할 일 목록중 하지 않는 작업만 필더링해서 보여준다고 가정해보자.

일반적으로는 다음과 같이 구현을 하게 된다.

```js
function UndoneTasks() {
  const tasks = useSelector((state) => state.todos.filter((todo) => todo.done))
  // ...
}
```

별 문제가 없어보일 수 있지만 사실 문제가 있다. 리덕스의 todos 말고 리덕스에서 관리되고 있는 관련 없는 상태가 변경될 때에도 위 컴포넌트에선 리렌더링이 발생한다. 그 이유는 배열의 filter 함수는 새로운 배열을 생성하기 때문에 매번 값이 변경된 것으로 간주하여 리렌더링이 되는 것이다.

이 때 Memoized Selector를 사용해 최적화 할 수 있다.

```js
import { createSelector } from "@redusxjs/toolkit"

const todosSelector = (state) => state.todos
const undoneTodos = createSelector(todosSelector, (todos) =>
  todos.filter((todo) => !todo.done)
)

function UndoneTasks() {
  const tasks = useSelector(undoneTodos)
  // ...
}
```

createSelector를 사용해 Memoized Selector를 만들 수 있는데, 이 함수에는 selector들을 연달아사ㅓ 넣을 수 있다. 이 함수에는 selector들을 연달아서 넣을 수 있다. 위 코드에서는 우선 todosSelector가 있다. 이 함수를 createSelector 첫번째 인자로 넣어서 selector에서 반환될 값이 변경될 때만 그 다음 selector를 호출하여 원하는 값을 연산하여 조회한다.

이렇게 하면 todos 배열에 실제로 변화가 있을 때에만 filter 함수를 돌리게 되고, 리렌더링을 하게 되지요.

그렇다고 이러한 상황에서 꼭 Memoized Selector를 사용해야만 최적화를 할 수 있을까? 꼭 그런것은 아니다. useMemo를 활용하면 비슷한 최적화를 할 수 있다. 다음과 같이 말이다.

```js
function UndoneTasks() {
  const tasks = useSelector((state) => state.todos)
  const undoneTasks = useMemo(
    () => tasks.filter((task) => !tasks.done),
    [tasks]
  )
  // ...
}
```

이렇게, 리덕스에서 관리하고 있는 상태를 어떠한 연산을 처리한 후 조회해야한다면 Memoized Selector 또는 useMemo를 꼭 사용하는 것을 권장한다.
