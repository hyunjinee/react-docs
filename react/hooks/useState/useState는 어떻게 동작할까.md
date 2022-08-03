# useState는 어떻게 동작할까

이 글의 목적은 다음과 같다.

- useState가 어떻게 상태를 변경시키는가.
- 어떻게 컴포넌트 함수가 변경 시킨 값으로 렌더링을 진행하는가.

```js
function App() {
  const [state, setState] = useState(0)

  const onClickHandler = () => {
    console.log("click")

    setState(1)

    if (state === 1) {
      console.log("실행될까?")
    }
  }

  return <div onClick={onClickHandler}>test</div>
}
```

첫번째 클릭에서는 click이 기록되고 두번째 클릭에서는 실행될까가 기록된다.

useState는 배열을 반환한다.

- 배열의 첫번째 요소는 상태 값 저장 면수 (이하 state),
- 두번째 요소는 상태값 갱신함수 이하 (setState)이다.

우리는 이 배열을 비구조화할당을 통해 이렇게 뽑아 사용한다.

```js
const [state, setState] = useState(initialState)
```

> import {useState} from 'react'

먼저 import 구문부터 살펴보자.
우리는 'react'라는 모듈에서 useState를 named import 해서 사용하고 있다.

![image](https://user-images.githubusercontent.com/63354527/182518307-8a645a03-13c7-478b-af6f-7b684985d7de.png)

위 사진은 node_modules/react/cjs/react.development.js 내부에 각종 hooks 함수가 선언된 곳이다. useState는 dispatcher라는 인스턴스를 생성하고 인자로 초기값을 받아 dispatcher.useState에 전달한 후 반환된 값을 리턴한다.

- dispatcher 메서드 useState에 initialState를 전달하면 배열을 반환한다.
- 그 안에 우리가 사용할 state와 setState가 담긴다.

더 거슬러 올라가서 dispatcher를 반환하는 resolveDispatcher 함수를 찾아보자.

![image](https://user-images.githubusercontent.com/63354527/182518455-58a90f7d-aad6-4076-adbc-ebf6e6029a78.png)

이 함수는 어딘가 dispatcher를 가져오고 에러처리를 한다. 새로운 키워드 ReactCurrentDispatcher로 올라가보자.

![image](https://user-images.githubusercontent.com/63354527/182518553-0fd2f762-aac9-407c-ab13-99bd7d7e9c9d.png)

ReactCurrentDispatcher라는 객체는 전역에 선언된 녀석이고, 속성으로 current를 가지고 있다. 이 current가 우리가 찾던 dispatcher가 담길 곳이다. 언제 어떻게 dispatcher가 담기는지는 넘어가겠다.

3줄요약하면 다음과 같다.

- useState를 포함한 hooks는 react모듈에 선언되어있는 함소이다.
- 실행될 때마다 dispatcher를 선언하고 useState 메소드 실행해서 그 값을 반환한다.
- 할당부를 거슬러 올라가니 dispatcher는 전역 변수 ReactCurrentDispatcher로부터 가져온다.

함수가 선언부보다 상위에 있는 값에 접근하는 것 바로 Closure이다.

## setState함수는 상태를 어떻게 변경시키나요?

```js

// react 모듈
let _value

export useState(initialState) {
  if (_value === undefined) {
    _value = initialState
  }

  const setState = (newValue) => {
    _value = newValue
  }

  return [_value, setState]
}
```

App 컴포넌트 함수는 실행되면 먼저 useState를 호출해서 반환값을 비구조화할당으로 추출해 변수에 저장한다.

여기서 중요한것은 App도 함수라는 것이다. jsx를 반환하는 함수. 렌더링이 시작되면 이함수가 호출되어 새로운 jsx를 반환한다.

위의 react 모듈 코드를 보면, useState 밖에 전역으로 선언된 \_value가 있다. 우리가 useState를 통해 관리하는 상태는 바로 이녀석이다. setState는 App 함수에 선언된 state가 아니라, 자신이 선언된 위치에서 접근할수 있는 \_value를 변경한다.

setSTate가 실행되어 리렌더링이 발생했다고 하자. setState가 리렌더링을 트리거하며 App 함수가 두번째로 실행되었을 때

- 다시 인수 0을 useState에 전달하여 호출한다.
- useState는 내부적으로 \_value값을 확인하고, undefined가 아닌 값이 할당되어 있기 때문에 초기값 할당문을 실행하지 않는다.
- 이후 useState가 현재 시점의 \_value와 setState를 반환한다.(이 시점에서 \_value는 1)
- 두번째 실행된 App 함수 내부에서 useState가 반환한 값을 비구조화 할당으로 추출해 변수에 할당한다.

즉, setState 함수는 자신과함께 반환된 변수를 변경시키는게 아니라, 다음 useState가 반환할 react 모듈의 \_value를 변경시키고, 컴포넌트를 리렌더링 시키는 역할을 한다. 변경된 값은 useState가 가져온다.

## 마무리

- 위 예시는 react로직을 아주 단순히 표현했을 뿐, 실제와 많이 다르다. useState를 여러번 사용해도 각기 다른 상태값과 갱신함수를 사용할 수 있는건 단순히 \_value가 아니라 여러 값을 저장하고 있기 때문이다.
- 요점은 setState가 state를 변경시키는게 아니기 때문에, setState 호출 이후 로직에서도 state의 값은 이전과 동일히다는 것이다. 변경된 값은 다음 컴포넌트 함수가 실행될 때 useState가 가져온다.
