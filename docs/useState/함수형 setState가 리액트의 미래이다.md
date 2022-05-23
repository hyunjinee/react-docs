# 함수형 setState가 리액트의 미래이다

이글은 2017년도에 작성되었습니다.

원문: [Functional setState is the future of React](https://www.freecodecamp.org/news/functional-setstate-is-the-future-of-react-374f30401b6b/#.lt8jkdocr)

리액트는 자바스크립트에서 함수형 프로그래밍(functional programming)을 대중화 했다. 이것이 리액트에서 사용하는 컴포넌트 기반 UI 패펀이 적용된 거대한 프레임워크로 이어졌다. 이제는 함수형 웹 개발 방법이 생태계 전반에 퍼지고 있다.

리액트는 컴포넌트 기반 UI라이브러리입니다. 컴포넌트는 기본적으로 일부 프로퍼티를 허용하고 UI 엘리먼트를 반환하는 함수입니다.

컴포넌트에는 상태(state)가 필요할 수 있다. 클래스형 컴포넌트를 생각해보자.클래스 생성자함수에 state를 선언한다. setState() 메소드의 동작 방식에 주목하자.업데이트할 상태의 부분을 포함하는 객체를 setState인자로 전달한다. 다시 말하면, 전달한 객체에는 컴포넌트의 상태(state)의 키에 해당하는 키가 있고 setState()는 그 객체를 상태(state)에 머지(merge)하여 상태(state)를 업데이트하거나 새로 설정한다. 말 그대로, '상태를 셋한다'(set-state).

## 왜 setState에 함수를 전달할까?

문제는 state 업데이트가 비동기로 실행된다는데 있다.
setState()가 호출될 때 어떤 일이 일어나는지 생각해보자.리액트는 먼저 setState()에 전달한 객체를 현재 상태로 합친다. 그러면 reconciliation을 시작한다. 새로운 리액트 엘리먼트 트리(UI의 객체 표현)을 만들고, 새 트리를 이전 (old)트리와 비교하고 ,setState()에 전달한 객체를 기반으로 변경된 부분을 파악한 다음 DOM을 최종적으로 업데이트한다.

```js
...
state = {score : 0};
// multiple setState() calls
increaseScoreBy3 () {
 this.setState({score : this.state.score + 1});
 this.setState({score : this.state.score + 1});
 this.setState({score : this.state.score + 1});
}
...
```

setState()에 전달하는 것은 일반 객체라는 것을 기억하자. 리액트가 여러 setState호출을 만나면 각 setState호출에 전달된 모든 객체를 추출하여 배치 작업을 수행하고 이를 머지(merge)하여 단일 객체를 형성 한 다음 해당 단일 객체를 사용하여 setState()를 수행한다.

자바스크립트의 객체를 머지(merging)하는 것은 다음과 같을 수 있다.

```js
const singleObject = Object.assign(
  {},
  objectFromSetState1,
  objectFromSetState2,
  objectFromSetState3
)
```

이런 패턴을 오브젝트 컴포지션(object composition)이라고 한다.

자바스크립트에서 만약 3개의 객체가 동일한 키를 갖고 있다면, Object.assign()에 마지막으로 전달된 객체의 키 값이 우선된다.

```js
const me = { name: "Justice" },
  you = { name: "Your name" },
  we = Object.assign({}, me, you)
we.name === "Your name" //true
console.log(we) // {name : "Your name"}
```

you가 we에 머지된(merged) 마지막 객체이기 때문에, you의 객체에 있는 name의 값 - ”Your name“-이 me 객체에 있는 name의 값을 덮어 쓴다. 그래서 ”Your name“이 we의 객체가 된다.

따라서, setState()에서 호출 할 때마다 매번 객체를 전달하고 리액트가 객체를 머지(merge)하게된다. 즉, 우리가 전달한 여러 객체 중에서 새로운 객체를 구성한다. 객체 중 하나에 동일한 키가 포함되어 있으면 동일한 키가 있는 마지막 객체의 키 값이 저장되는 것이다.

즉, 위의 increaseScoreBy3 함수를 사용하면 리액트가 setState() 호출 순서로 상태를 즉시 업데이트 하지 않기 때문에 함수의 최종 결과는 3대신 1이된다. 리액트는 이처럼 먼저 모든 객체를 하나로 결합한 후에 계산한다. {score: this.state.score + 1}

setState()에 객체를 전달하는 것이 문제가 아니다. 진짜 문제는 이전 상태에서 다음 상태를 계산할 때 객체를 setState()로 전달하는 것에 있다. 이것은 안전하지 않다.

> props 및 this.state가 비동기적으로 업데이트 될 수 있으므로 다음 상태 (next state)를 계산할 때 해당 값을 신뢰해서는 안된다.

함수형 setState는 위 문제를 해결한다. 정확히 어떻게 해결한 것일까?

Dan의 트윗을 보자.

```js
It is safe to call setState with a function multiple times. Updates will be queued and later executed in the order thye were called.
```

업데이트는 큐(queue)에 쌓이고 추후에 호출된 순서대로 실행된다.
따라서 리액트가 객체를 머지 (merge)하는 대신 "여러 함수형 setState호출"를 만나면 (물론 머지할 객체가 없다.)리액트는 호출된 순서대로 함수를 큐에 넣는다.

그 후 리액트는 “큐(queue)“의 각 함수를 호출하여 상태를 업데이트하고 이전 상태, 즉 첫 번째 함수형 setState() 호출 이전 상태를 전달한다 (첫 번째 함수형 setState() 인 경우). 만약 첫 번째 함수형 setState() 호출이 아니라면, 큐 내의 이전의 함수형 setState() 호출로부터 최신 갱신된 상태를 전달한다.

코드로 확인해보자. 먼저 컴포넌트 클래스를 생성해보자. 그런 다음 내부에 가짜 setState() 메소드를 만들 것이다. 또한 우리 컴포넌트에는 여러 함수형 setState를 호출하는 increaseScoreBy3() 메서드가 있을 것이다.

```js
class User {
  state = { score: 0 }
  //let's fake setState
  setState(state, callback) {
    this.state = Object.assign({}, this.state, state)
    if (callback) callback()
  }
  // multiple functional setState call
  increaseScoreBy3() {
    this.setState((state) => ({ score: state.score + 1 }))
    this.setState((state) => ({ score: state.score + 1 }))
    this.setState((state) => ({ score: state.score + 1 }))
  }
}
const Justice = new User()
```

setState는 두 번째 파라미터로 콜백 함수도 허용한다. 리액트가 상태를 업데이트 한 후 호출된다.
이제 사용자가 increaseScoreBy3()을 호출하면 리액트는 여러 함수형 setState를 큐에 담는다. 여기서는 페이크 로직을 만들지 않을 것이다. 우리는 함수형 setState를 실제로 안전하게 만드는 것에 중점을 두고 있기 때문이다. 그러나 여러분은 그 “큐잉(queuing)” 프로세스의 결과를 다음과 같은 함수의 배열로 생각할 수 있다.

```js
const updateQueue = [
  (state) => ({ score: state.score + 1 }),
  (state) => ({ score: state.score + 1 }),
  (state) => ({ score: state.score + 1 }),
]
```

```js
// recursively update state in the order
function updateState(component, updateQueue) {
  if (updateQueue.length === 1) {
    return component.setState(updateQueue[0](component.state))
  }
  return component.setState(updateQueue[0](component.state), () =>
    updateState(component, updateQueue.slice(1))
  )
}
updateState(Justice, updateQueue)
```

사실, 이것은 섹시한 코드가 아니다. 여러분이 더 좋은 코드를 만들 수 있다고 생각한다. 그러나 여기서 핵심은 리액트가 함수형 setState에서 함수를 실행 할 때마다 리액트가 업데이트 된 상태의 새로운 복사본을 전달하여 상태를 업데이트한다는 것이다. 이것에 의해, 함수형 setState가 이전의 상태에 근거해서 상태를 셋(set)할 수 있는 것이다.

## 리액트의 일급 비밀

지금까지 리액트에서 여러 함수형 setState를 수행하는것이 왜 안전한지 깊게 살펴봤다. 그러나 실제로 함수형 setState를 완벽하게 정의하지는 못했다.

"상태 변경을 컴포넌트 클래스와 분리해서 작성하라"

수년간 setState에 전달하는 함수 또는 객체는 항상 컴포넌트 클래스 안에 존재했다.

> Best kep React secret: you can declare state changes seperately from the component classes. - Dan Abramov

이것이 함수형 setState의 힘이다. 컴포넌트 클래스 외부에서 상태 업데이트 로직을 선언한다. 그런 다음 컴포넌트 내에서 호출한다.

```js
// outside your component class
function increaseScore (state, props) {
  return {score : state.score + 1}
}
class User{
  ...
// inside your component class
  handleIncreaseScore () {
    this.setState( increaseScore)
  }
  ...
}
```

이것은 선언적 (declarative)이다. 컴포넌트 클래스는 더이상 상태 업데이트 방법을 고려하지 않는다. 단순히 원하는 업데이트 유형을 선언하기만 한다.

이 점에 깊이 감사하기 위해, 많은 상태(state)가 있는 복잡한 컴포넌트에 대해 생각해보자. 각 상태를 다른 액션으로 업데이트 할 것이다. 떄로는 각각의 업데이트 함수에는 많은 코드가 포함되어 있을 수 있다. 이 모든 로직이 컴포넌트 내에 있다는 것이다. 그러나 더 이상은 그럴 필요가 없다.

모듈을 가능한 짧게 유지하기 위해, 너무 길어지면 상태 변경 로직을 다른 모듈로 추출한 다음 컴포넌트에 가져와서 사용할 수 있다.

```js
import {increaseScore} from "../stateChanges";
class User{
  ...
  // inside your component class
  handleIncreaseScore () {
    this.setState( increaseScore)
  }
  ...
}
```

함수형 setState로 다른 무엇을 할 수 있을까?

> Declaring state updates as pure functions makes it a breeze to test complex state transitions. Even no need for shallow rendering! - Dan Abramov

다음 상태를 계산하기 위해 추가 인자를 전달할 수도 있다.

> What if you need to pass an extra argument to calculate the next state? Functions can return functions - Dan Abramov

> It's likely that in future React will eventually support this pattern more directly so i recommend to try using it and see if you like if! - Dan Abramov (January 25, 2017)
