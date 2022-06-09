# React의 setState() 제대로 사용하기

이 글은 이화랑님 블로그 글을 읽고 정리한 글 입니다.

리액트 코드를 짜다가 setState()가 제대로 동작하지 않는 두가지 상황을 발견했다.

1. 동일한 state를 변경하는 setState()를 연속적으로 사용하였을 때이다.
2. 두번째 상황은 setState()를 실행한 뒤에 곧 바로 api 호출을 보냈을 때이다.

이 두경우 모두 setState()가 비동기로 동작하는 것이 원인 이었는데 문제 상황을 다시 살펴보며 어떻게 해결했는지 정리해보려고 한다.

setState()를 연속적으로 사용하면 마지막에 setState()만 실행되는 것 처럼 보인다. 왜 그럴까?

아래 세가지를 기억하자.

1. setState는 비동기로 처리된다.
2. setState()를 연속적으로 호출하면 Batch 처리를 한다.
3. state는 객체이다.

setState() 함수가 호출되면 리액트는 바로 전달받은 state로 값을 바꾸는 것이 아니라 이전의 리액트 엘리먼트 트리와 전달받은 state가 적용된 엘리먼트 트리를 비교하는 작업을 거치고, 최종적으로 변경된 부분만 DOM에 적용한다. 이 과정은 비동기로 작동한다. 따라서 setState가 연속적으로 호출되면 -> "번거로운 작업을 한번에 수행할 수 없을까?" 라고 생각하고 전달받은 각각의 state를 합치는 merging 작업을 수행한 뒤에 한번에 setState()를 한다.

mergining은 어떻게 동작할까?

현재 orders에 [“감자 튀김 🍟”, “콜라 🥤”] 가 있다고 했을 때, 배열을 초기화 하고(setOrder([])), “선택하지 않음”을 추가하는 부분(setOrders([...orders, selectedItem]);)을 코드로 살펴보면 다음과 같다.

```js
const newState = Object.assign(
  { orders: ["감자 튀김 🍟", "콜라 🥤"] },
  { orders: [] },
  { orders: [...orders, "선택하지 않음"] }
)

setOrders(newState)
```

Object.assign()으로 여러개의 객체를 합칠 때, 같은 key를 가지고 있다면 이전의 값이 덮어씌워지기 때문에 결국 {orders: [...orders, "선택하지 않음"]} 마지막 명령어가 실행된다. 마지막 명령어가 실행될 때의 orders는 아직 변경되기 전인 상태, 즉 [“감자 튀김 🍟”, “콜라 🥤”] 이기 때문에 배열이 초기화 되지 않고 “선택하지 않음”이 추가되는 것이다.

react코드를 까보면 setState() 함수는 인자로(1) 새로운 state 객체를 받을 수 있고, (2) 이전 state 객체를 인자로 받고 새로운 state 객체를 반환하는 함수를 받을 수 있다. 결론부터 이야기하면 (2)의 방법으로 했을 때 여러번의 setState()를 문제없이 실행할 수 있다.

setState()가 비동기적으로 동작한다는 것은 변함이 없지만, 인자로 넘겨받는 함수들은 Queue에 저장되어 순서대로 실행된다. 따라서 첫번째 함수가 실행된후 리턴하는 업데이트된 state가 두번째 함수의 인자로 들어가는 방식으로 state의 최신 상태가 유지된다.

```js
const onClickHandler = (selectedItem) => {
  if (selectedItem === "선택하지 않음") {
    setOrders((orders) => [])
  }

  if (orders.includes(selectedItem)) {
    setOrders((orders) => orders.filter((order) => order !== selectedItem))
    return
  }

  if (orders.includes("선택하지 않음")) {
    setOrders((orders) => orders.filter((order) => order !== "선택하지 않음"))
  }
  setOrders((orders) => [...orders, selectedItem])
}
```

함수형 setState()로 변경하여 정삭적으로 동작함을 볼 수 있다.

```js
type SetStateAction<S> = S | ((prevState: S) => S)
type Reducer<S, A> = (prevState: S, action: A) => S
```

types/react 모듈의 index.d.ts을 살펴보면 함수형 setState와 Reducer의 로직의 내부 로직이 같은 것을 확인할 수 있다. setState()로 넘기는 함수의 로직을, Reducer의 action type과 매칭되는 함수에 넣어서 실행시킨 뒤 새로운 state 객체를 반환한다.

언제 useState를 쓰고 언제 useReducer을 써야하는가? -> 결론은 상태를 관리하는 로직이 어느정도 수준 이상으로 복잡해지면 useReducer을 쓰자이다.

## setState()를 실행한 뒤에 곧 바로 api 호출을 보냈을 때

POST 요청에 함께 보내야 하는 state값을 변경하고 POST요청을 해야하는 케이스가 있다.
예를들어 '결제하기'나 '제출하기'등의 버튼을 눌렀을 때, POST 요청과 함께 보내야하는 state의 값을 변경하고 POST요청을 해야하는 케이스가 존재한다. 업데이트된 상태값을 넘겨야하니까 머리로 "setState를 실행하고 POST요청을 보내야지!"라고 생각을 하고 코드를 짰는데, setState도 비동기, POST요청도 비동기로 처리되어 심지어 POST요청의 우선순위가 더 높아 업데이트된 상태값이 전달되지 않는 문제가 있었다.

사실 POST 요청을 보내고 다른 페이지로 이동하는 동작이여서 업데이트된 상태를 가지고 있을 필요가 없었고, 따라서 setState()를 굳이 실행하지 않고 일반 객체로 만들어 전달하여 해결했다. 'setState를 사용할 때 정말 여기서 실행해야하나?'라고 한번더 생각하는 습관을 가지면 좋겠다.
