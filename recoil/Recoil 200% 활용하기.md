# Recoil 200% 활용하기

## 상태 관리의 중요성

양방향 바인딩을 하는 Angular.js와는 달리, React.js는 단방향 바인딩을 하는 라이브러리이다.  
부모 자식방향으로만 state를 props로 전달할 수 있고, 자식의 props를 부모에게 전달하는 방법은 존재하지 않다.

![image](https://user-images.githubusercontent.com/63354527/184620614-bbf44bb8-70ff-4188-9530-791febb91aef.png)

자식 component에서 부모 component의 state를 바꿀 수 있는 두가지 방법이 존재한다.

1. 자식에게 부모의 state를 modify할 수 있는 setState 함수를 props로 넘겨준다.
2. state management tool(redux, recoil)를 사용한다.

![image](https://user-images.githubusercontent.com/63354527/184621068-5c96e459-0b29-425e-bcbe-27556f15eccd.png)

## 뭔가 탐탁치 않은 Redux, 그리고 출시된 Recoil

원래 React 프로젝트의 대부분은 MVC 아키텍처라는 구조로 설계하고 있었다. Controller가 여러 Model을 제어하고 있는 구조이다. 하지만 프로젝트의 규모가 커지면서 상태도 많아졌기 때문에 Model과 View가 양방향으로 영향을 미치는 이 구조의 관리가 어려웠따.

특히 규모가 큰 facebook에서 이러한 패턴으로는 단순한 상태의 변화도 예측가능한 범위를 벗어나 관리가 힘들다고 판단해 새로운 아키텍처를 고안합니다.

![image](https://user-images.githubusercontent.com/63354527/184621413-aad204e0-adba-40c8-97bc-2b9be2ccefd0.png)

그래서 Facebook이 고안한 Flux아키텍처가 많이 사용되고 있다. 그림과 같이 데이터의 흐름은 무조건 단방향으로 흐른다. dispatcher에서 action을 통해 store로 store에서 view로 또 다시 view에서 action을 통해 dispatcher로 흐르게 되는 구조이다.
![image](https://user-images.githubusercontent.com/63354527/184621766-10743381-e06b-4da4-872d-cf66a25ed4da.png)

이 Flux 아키텍처를 따라간 구현체로 Redux가 가장 인지도가 높고 많이 사용되는 상태관리 툴이다. action, reducer, selector, store를 초기 세팅하는 부분은 간단한 상태 하나를 관리하더라도 많은 코드를 소모하였고 React에 최적화된 라이브러리가 아니었기에 사용에 불편함을 많았습니다.

facebook의 Recoil팀은 Recoil을 다음과 같이 표현하며 출시한다.

> A state management library for React

## Recoil의 기본 Atom

> An atom represents state in Recoil. The atom() function returns a writable RecoilState object.

atom은 기존의 redux에서 쓰이는 store와 유사한 개념으로, 상태의 단위이다.
atom이 업데이트 되면 해당 atom을 구독하고 있던 모든 컴포넌트들의 state가 새로운 값으로 리렌더링된다.

unique한 id인 key로 구분되는 각 atom은 여러 컴포넌트에서 atom을 구독하고 있다면 그 컴포넌트들도 똑같은 상태를 공유한다.

![image](https://user-images.githubusercontent.com/63354527/184625648-724b9a20-879f-45f7-9ae6-e75713ef03ed.png)

```js
export const cookieState = atom({
  key: "cookieState",
  default: [],
})
```

key로 고유한 atom을 구분하고, default에 기본으로 atom에 저장되는 값을 지정할 수 있다.

```js
import React from "react"
import { cookieState } from "../../state"
import { useRecoilState } from "recoil"

const Cookie = () => {
  const [cookies, setCookies] = useRecoilState(cookieState)

  return (
    <div>
      {cookie.map((cookie) => (
        <Card cookies={cookie} key={cookie.id} idx={cookie.id} />
      ))}
    </div>
  )
}
export default Cookie
```

React의 기본 hook인 useState와 굉장히 유사한 형태를 지닌다.

```js
import { useRecoilValue, useSetRecoilState } from "recoil"
import { cookieState } from "../../state"

const cookies = useRecoilValue(cookieState)
const setCookies = useSetRecoilState(cookieState)
```

reset시키는 훅도 있다.

```js
const resetCookies = useResetRecoilState(cookieState)
```

## Selector와 비동기 처리 (Suspense, Loadable)

atom만을 사용해서는 비동기 처리를 할 수 없다.  
selector의 예제를 한번 보자.

```js
import React, { useState, useEffect } from 'react'
import { useRecoilState } from 'recoil';
import { cookieState } from '../../recoil';
import {Loading, Card} from '../../components';

const Cookies = () =>{
  const [cookie, setCookie] = useRecoilState(cookieState);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    (async () => {
      const { data } = await getApi.getCookies();
      setCookie(data);
      // 다음과 같이 컴포넌트에서 직접 비동기 처리를 해주고, api 통신 결과로 받아온 data를 직접 atom에 set 해줍니다.
    })();
    setLoading(false);
  }, [setCookie]);

return (
  <>
  {Loading ?
   <Loading /> // 로딩 중 일때 보여줄 view
   :
   (<div>
        {cookie.map(cookie => (
          <Card
            cookies={cookie}
            key={cookie.id}
            idx={cookie.id}
           />
       ))}
      </div>)
  </>
}

export default Cookies;
```

atom의 상태가 변할때 마다 각 컴포넌트에서 이렇게 따로 비동기 처리를 해준다면, 같은 atom을 구독하고 있던 컴포넌트들은 알아서 re-rendering된다.

하지만, selector에서는 이러한 로직을 한번에 처리해 줄 수 있는 동시에 캐싱 기능이 있어, 이미 받아왔던 정보에 대해서는 빠른 피드백이 가능해 성능적으로 유리하다.

## selector

> A selector represents a piece of derived state

```js
export const cookieState = atom({
  key: "cookieState",
  default: [],
})

export const getCookieSelector = selector({
  key: "cookie/get",
  get: async ({ get }) => {
    try {
      const { data } = await client.get("/cookies")
      return data.data
    } catch (err) {
      throw err
    }
  },
  set: ({ set }, newValue) => {
    set(cookieState, newValue)
  },
})
```

다음과 같이 비동기 로직을 selector에선 한번에 처리할 수 있다. 주의해야할 점은 selector는 순수함수여야 한다는 것이다.

순수함수란, 같은 입력이 들어오면, 해당 입력에 대한 출력은 항상 같은 함수라는 뜻을 가지고 있다.

또한 selector는 read-only한 RecoilValueReadOnly 객체로서 return 값 만을 가질 수 있고 값을 set할 순 없는 특징을 가지고 있다.

```ts
function selector<T>({
  key: string,

  get: ({
    get: GetRecoilValue
  }) => T | Promise<T> | RecoilValue<T>,

  set?: (
    {
      get: GetRecoilValue,
      set: SetRecoilState,
      reset: ResetRecoilState,
    },
    newValue: T | DefaultValue,
  ) => void,

  dangerouslyAllowMutability?: boolean,
})
```

- key: selector을 구분할 수 있는 유일한 id, 즉 key 값을 의미
- get: derived state를 return 하는 곳이다. 예시 코드에서 api call을 통해 받아온 data를 return 한다. (해당 selector가 갖고 있다.)
  ```js
  const cookie = useRecoilValue(selector)
  ```
  다음과 같이 값을 조회할 수 있다.
- set: writable한 state 값을 변경할 수 있는 함수를 return 하는 곳이다. 여기서 주의해야할 점은, 자기 자신 selector를 set하려고 하면, 스스로를 해당 set function에서 set 하는 것이므로 무한루프가 돌게되니 반드시 다른 selector와 atom을 set하는 로직을 구성해야한다. 또한 애초에 selector는 read-only 한 return 값(recoilValue)만 가지기 때문에 set으로는 writable한 atom의 RecoilState만 설정할 수 있다.
  ```js
  set: ({ set }, newValue) => {
    set(getCookieSelector, newValue)
  } // incorrect : cannot allign itself
  set: ({ set }, newValue) => {
    set(cookieState, newValue)
  } // correct : can allign another upstream atom that is writeable RecoilState
  ```
  ```js
  const [cookie, setCookie] = useRecoilState(cookieState)
  ```

selector로 개선

```js
import { getCookieSeletor } from '../../reocil';

const Cookies = () =>{
  const [cookie, setCookie] = useRecoilState(getCookieSelector);
}

return (
  <>
   (<div>
        {cookie.map(cookie => (
          <Card
            cookies={cookie}
            key={cookie.id}
            idx={cookie.id}
           />
       ))}
      </div>)
  </>
});

export default Cookies;
```

selector는 read-only한 return 값만 가진다. 근데 여기서 useRecoilState()는 setCookie()라는 state를 변경할 수 있는 set함수도 반환하고 있다. 위의 set 설명에서 언급했듯, 이는 writable한 state즉 atom의 값만 수정할 수 있다.

selector에서 비동기 처리를 하고 실행하면 에러 페이지가 나올 것이다. fallback ui가 없기 때문이다.

Suspense 대신 Recoil의 Loadable을 사용할 수도 있다.

```js
import { getCookieSeletor } from '../../reocil';
import { useRecoilState, useRecoilValueLoadable } from 'recoil';

const Cookies = () => {
  const cookieLoadable = useRecoilValueLoadable(getCookieSelector);

  switch(cookieLoadable.state){
    case 'hasValue':
      return (
        <>
          (<div>
    	    {cookieLoadable.contents.map(cookie =>(
              <Card
                cookies={cookie}
                key={cookie.id}
                idx={cookie.id}
               />
            ))}
	     </div>)
	  </>
	});
     case 'loading':
  	return <Loading />;
     case 'hasError':
     	throw cookieLoadable.contents;
}


export default Cookies;
```

Loadable atom 이나 selector의 현재 상태를 나타내는 객체 이다.  
Loadable은 state 와 contents 라는 프로퍼티를 가지고 있다.

> Loadable 객체
> state: hasValue, hasError, loading atom이나 selector의 상태를 말하며, 앞의 세가지 상태를 가질 수 있다.
> contents: atom이나 contents의 값을 나타내며, 상태에 따라 다른 값을 가지고 있다.
> hasValue 상태일 땐 value를, hasError 상태일 땐 에러 객체를 그리고 loading 상태일 땐 Promise를 가진다.

store에 대한 상태를 reducer에 action으로 따로 정의해줘야 하는 불편함이 있었지만, Recoil에서는 `useRecoilValueLoadable()` 훅을 사용해 쉽게 접근할 수 있어 훨씬 편하게 사용할 수 있다.

selector는 기본적으로 값을 캐싱한다. 들어왔던 적이 있는 값을 기억하고 있기 때문에 같은 응답을 보내는 api call에 대해선 추가적으로 요청을 하지 않아 성능적으로 매우 유리하다.

동적인 url에 대한 api call을 처리하기위해 selectorFamily를 사용할 수 있다. 외부에서 파라미터로 값을 받아와서 selector에 적용해야 할 경우에 사용한다.

```js
export const githubRepo = selectorFamily({
  key: "github/get",
  get: (githubId) => async () => {
    if (!githubId) return ""

    const { data } = await axios.get(`https://api.github.com/repos/${githubId}`)
    return data
  },
})
```

컴포넌트에서의 사용예제

```js
import { useRecoilValue } from "recoil"
import { selectorFamily } from "../../state"
const Github = () => {
  const githubId = "juno7803"
  const githubRepos = useRecoilValue(githubRepo(githubId))

  return (
    <>
      <div>Repos : {githubRepos}</div>
    </>
  )
}
export default Github
```
