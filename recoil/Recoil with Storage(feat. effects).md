# Recoil with Storage(feat. effects)

## atom을 storage에 자동으로 연동시키는 방법이 있을까?

redux-persist는 설정만 한다면 자동적으로 state를 storage에 저장해서 정말 편하게 개발할 수 있다.  
하지만 또 다른 의존성을 추가(라이브러리를 추가)해야 함으로써 그에 따른 댓가를 치뤄야 한다.

atom을 storage에 자동으로 연동하는 방법은 atom의 effect를 활용하는 것이다.  
atom의 effect란 useEffect와 비슷하다.

atom은 어플리케이션이 실행될 때 App 컨텍스트 외부에서 생성된다.  
atom의 effect는 atom이 생성될 때 실행되는 부수효과 (Side Effect)이다.

```js
export const user = atom<IUsertemp>({
  key: "user",
  default: {} as IUsertemp,
  effects: [localStorageEffect("user"), sessionStorageEffect("user")],
});
```

atom을 생성해주고 인자로 넘겨주는 프로퍼티에 effects라는 프로퍼티를 추가한 후 함수들을 넣어준다.

```js
const localStorageEffect =
  (key: string) =>
  ({ setSelf, onSet }: any) => {
    const savedValue = localStorage.getItem(key)

    if (savedValue !== null) {
      setSelf(JSON.parse(savedValue))
    }

    onSet((newValue: any, _: any, isReset: boolean) => {
      isReset
        ? localStorage.removeItem(key)
        : localStorage.setItem(key, JSON.stringify(newValue))
    })
  }

const sessionStorageEffect =
  (key: string) =>
  ({ setSelf, onSet }: any) => {
    const savedValue = sessionStorage.getItem(key)

    if (savedValue !== null) {
      setSelf(JSON.parse(savedValue))
    }
    onSet((newValue: any, _: any, isReset: any) => {
      const confirm = newValue.length === 0
      confirm
        ? sessionStorage.removeItem(key)
        : sessionStorage.setItem(key, JSON.stringify(newValue))
    })
  }
```

localStorage, SessionStorage에 저장하는 effect함수이다.  
매개변수의 key는 저장소에 저장되는 key값이며, 내부의 콜백함수 객체 속 매개변수로 주어지는 `setSelf` 함수는 연결된 atom의 값을 초기화해주는 함수이며 `onSet`함수는 해당하는 atom의 값이 변경되었을 때 실행하는 함수이다.

## atom의 값을 서버 데이터로 초기화할 순 없을까?

애플리케이션이 실행되었을 때 초기 데이터가 필요하여 서버에서 데이터를 가져와 전역 상태로 초기화 시켜야하는 경우가 있다. 이런 경우 atom에 데이터를 어떻게 초기화할 수 있을까?

먼저 async Effect 함수를 정의하여 effects의 의존성 배열에 넣어준다.

```js
const asyncUserListEffect =
  (key: string) =>
  ({ onSet, setSelf }: any) => {
    setSelf(() => {
      const localData = localStorage.getItem(key);

      //localStorage 에 셋팅된 값이 있다면 해당 값으로 atom 을 초기화, 없다면 API 호출
      if (localData !== null) {
        return JSON.parse(localData);
      } else {
        return getUserList();
      }
    });

    // Trigger 가 발동이 되어야 실행된다. (atom 의 값이 변경이 되었을 경우에 실행된다.)
    onSet((newValue: any, _: any, isReset: boolean) => {
      localStorage.setItem(key, JSON.stringify(newValue));
    });
  };

// Effects 를 활용해 atom 의 초기값 설정
export const userList = atom<IUser[]>({
  key: "userList",
  effects: [asyncUserListEffect("list")],
});
```

위의 로직을 살펴보면, 먼저 로컬 저장소에 저장된 데이터가 있는지 체크하고, 만약 데이터가 없다면 데이터를 패칭하여 atom 값을 초기화시켜준다.

선언된 atom을 살펴보면 default 프로퍼티를 설정하지 않았다. atom이 생성될 때 effects 배열에 저장된 함수들이 실행되면서 atom의 값을 초기화 시키기 때문에 따로 설정하지 않는다.

## 만약 동적인 서버 데이터를 atom에 초기화 할 수 있는 방법은?

웹 애플리케이션을 개발하다보면 반드시 동적인 데이터를 활용해야 하는 경우가 있습니다. 그럴 경우에 어떻게 해야할까요?  
`atom` 대신 `atomFamily`를 활용해 매개변수를 활용해봅니다.

```js
import { atom, atomFamily, selector } from "recoil";
import { IUser } from "../type";
import { getSelectedUser } from "./api";

const asyncUserEffect =
  (key: string, id: number) =>
  ({ onSet, setSelf }: any) => {
    setSelf(() => {
      const localData = localStorage.getItem(key);

      //localStorage 에 셋팅된 값이 있다면 해당 값으로 atom 을 초기화, 없다면 API 호출
      if (localData !== null) {
        return JSON.parse(localData);
      } else {
        return getSelectedUser(id);
      }
    });

    // Trigger 가 발동이 되어야 실행된다. (atom 의 값이 변경이 되었을 경우에 초기화된다.)
    onSet((newValue: any, _: any, isReset: boolean) => {
      localStorage.setItem(key, JSON.stringify(newValue));
    });
  };

// 첫 번째 제네릭은 아톰의 타입, 두 번째 제네릭은 param 의 타입
export const userAtom = atomFamily<IUser, number>({
  key: "userAtom",
  effects: (param) => [asyncUserEffect("userAtom", param)],
});
```

`atomFamily`를 활용해 매개변수를 받는다. effects 프로퍼티의 값으로 함수를 넣어주는데, 매개변수 param을 받아서 해당 param을 effect 함수에 넣어 사용할 수 있다.

`atomFamily`의 제네릭은 두가지를 넣어주어야한다. 첫번째 제네릭은 해당 atom의 값의 타입이며, 두번째 제네릭은 parameter의 타입니다.

```js
import React from "react"
import { useRecoilState } from "recoil"
import { IUser } from "../modules/type"
import { userAtom } from "../modules/User/atom"

// UserList 에서 하나의 User 만 저장하는 상황
// atom 에 파라미터가 필요하다.
// localStorage 와 연동

type FamilyProps = {
  id: number,
}

export default function Family(props: FamilyProps) {
  const { id } = props
  const [oneUser, setOneUser] = useRecoilState < IUser > userAtom(id)
  return (
    <div>
      <h1>{oneUser.id}</h1>
      <h3>{oneUser.name}</h3>
    </div>
  )
}
```

위 예제는 간단한 컴포넌트이다. props로 주어지는 값에 따라서 atom 값을 다르게 초기화한다.

## Cookie를 활용하고 싶다?

마지막으로 Cookie에 값을 저장해보도록 하자.
아쉽게 Recoil에는 Cookie와 연동되는 기능은 없다. 그렇기 때문에 쉽게 Cookie를 활용하기 위해 react-cookie라이브러리를 활용한다.

```js
import React, { useState } from "react"
import { useCookies } from "react-cookie"
import { useSetRecoilState } from "recoil"
import { user } from "../modules/UserList/atom"
import { IUsertemp } from "../modules/type"

function Join() {
  const [id, setId] = useState("")
  const [pwd, setPwd] = useState("")
  const [name, setName] = useState("")
  const [addr, setAddr] = useState("")
  const [coockies, setCoockie, removeCookie] = useCookies(["user"])

  const handleSaveCookie = () => {
    const newbby: IUsertemp = {
      id,
      pwd,
      name,
      addr,
    }
    setCoockie("user", newbby)
  }

  return (
    <>
      <form onSubmit={handleSubmit}>
        <div>
          <span>ID</span>
          <input value={id} onChange={handleId} />
        </div>
        <div>
          <span>PWD</span>
          <input value={pwd} onChange={handlePwd} />
        </div>
        <div>
          <span>NAME</span>
          <input value={name} onChange={handleName} />
        </div>
        <div>
          <span>ADDR</span>
          <input value={addr} onChange={handleAddr} />
        </div>
        <button onClick={handleSaveCookie}>save in Cookie</button>
      </form>
      {/* <div>{coockies.user.id}</div> */}
    </>
  )
}

export default Join
```

버튼 클릭시 user라는 키를 가진 쿠키에 데이터가 저장이도니다. `useCookies`훅을 이용하면 쉽게 리액트에서 쿠키에 값을 저장할 수 있다.

## recoil-persist

recoil-persist라는 라이브러리가 존재한다.

```js
// sessionStorage 와 localStorage 에 동시에 적용할 순 없음
const { persistAtom } = recoilPersist({
  key: "recoil-persist-atom",
  storage: sessionStorage,
  // storage: localStorage,
});

// atom 집합은 React 컨텍스트 외부에서 생성된다
// atom 의 초기값을 서버 데이터로 초기화 하고 싶다면 effects 를 활용!!
export const user = atom<IUsertemp>({
  key: "user",
  default: {} as IUsertemp,
  // effects: [localStorageEffect("user"), sessionStorageEffect("user")],
  effects: [persistAtom],
});
```

persistAtom을 effects 배열에 넣어주기만 하면 된다. 간단한 코드로 자동으로 상태를 유지시킬 수 있다.
