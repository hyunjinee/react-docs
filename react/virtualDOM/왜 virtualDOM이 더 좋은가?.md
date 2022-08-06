## DOM

DOM은 브라우저 상에서 자바스크립트가 HTML문서를 조작하기 위한 API이다. 자바스크립트가 직접 HTML을 동적으로 바꾸는데에는 한계가 있으니 JS가 조작하기 편하게 DOM이라는 형태로 바꾸어 놓은 것이다.

왜 굳이 JS가 HTML을 조작해야할까요? 더 다양하고 복잡한 인터렉션을 만들고 화면을 구현하기 위해서 필요합니다.

![image](https://user-images.githubusercontent.com/63354527/183247559-23a1f760-3b19-4214-9b08-6202b5b7bf0c.png)

## virtualDOM

JS가 HTML을 조작하기 편하게 만들어진 객체(DOM)를 허상으로 만든 것.

virtualDOM은 JS가 HTML을 조작하기 편하도록 만들어진 객체를 허상으로 만든 것이라고 할 수 있습니다.

1. realDOM으로부터 virtualDOM을 만든다. (virtualDOM은 메모리상에 존재하는 하나의 객체이다.)
2. 변화가 생기면 새로운 버전의 virtualDOM을 만든다.
3. old 버전의 virtualDOM과 new 버전의 virtualDOM을 비교한다. (diffing algorithm)
4. 비교 과정을 통해서 발견한 차이점을 realDOM에 적용한다.

![image](https://user-images.githubusercontent.com/63354527/183251230-565f7f05-b75c-48c5-92a9-21677ed0c362.png)

위 과정을 reconciliation이라고 부른다.

리렌더링이 일어난다는 것은 특정 node에 변화가 생겼다는 것을 의미한다. 그렇다면 이 변화를 어떻게 감지하는 것일까?

1. dirty checking: node tree를 재귀적으로 계속 순회하면서 어떤 노드에 변화가 생겼는지를 인식하는 방법이다. 그리고 그 노드만 리렌더링 시켜준다. 이 방법은 angular.js가 사용하던 방법인데, 이렇게 하면 변화가 없을 때도 재귀적으로 노드를 탐색함으로써 불필요한 비용이 발생하고 성능적인 측면에서도 문제가 있다.
2. observable: observable은 변화가 생긴 노드가 관찰자에게 알림을 보내주는 방식이다. 리액트의 경우에는 state에 변화가 생겼을 때, 리액트에게 렌더링을 해줘야한다는 것을 알려주는 방식으로 이루어진다. 그리고 리액트는 알림을 받으면 렌더링을 시킨다. 이런 방식을 사용할 경우에 변화가 생기기 전까지는 아무일도 하지 않아도 된다. 노드에게 변화가 생겼다는 알림을 받으면 렌더링합니다. 아주 효율적인 방식이라고 할 수 있습니다.

observable의 문제점은 변화에 대한 알림을 받으면 전체를 렌더링 시켜버린다는 것입니다. 전체를 렌더링시키는 것은 말그대로 엄청난 reflow-repaint 비용이 발생합니다. 이 지점에 등장한 것이 바로 virtualDOM입니다.

virtualDOM은 메모리 상에 존재하는 하나의 객체입니다. 리액트는 특정 state에 변화가 생겼다는 알림을 받으면 realDOM 전체를 렌더링 시켜주는 것이 아니라, virtualDOM을 렌더링 시킵니다.

> 브라우저를 렌더링 시키는 비용 vs 객체를 새로 만드는 비용

객체를 새로 만드는 비용이 훨씬 저렴합니다. 리액트는 변화가 감지되면 realDOM을 새롭게 렌더링 시키는 대신 virtualDOM이라는 메모리상의 객체를 새로 만드는 방식을 선택했습니다. 그리고 거기서 변화가 생긴 내용을 비교해 마지막에 가서는 꼭 필요한 부분만 realDOM에 적용시키는 방식으로 효율성을 높인 것 입니다.

그리고 실제로 전체화면을 렌더링하는 것과 일부만 변경시켜주는 것은 비용적인 측면에서 엄청난 차이가 있습니다.

## virtualDOM은 어떻게 객체로 만들어지는가

React의 createElement 소스 코드를 일부분 보자.

```js
export function createElement(type, config, children) {

  ...중략

  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
```

createElement의 내부적인 내용을 보면 ReactElement 함수의 인자에 넘겨줄 값들을 꾸려주고 있다. 결론적으론 ReactElement 함수의 호출값을 반환한다. createElement 함수가 반환하는 값이 무엇인지 확인하려면 ReactElement의 반환값이 무엇인지 알아야한다.

```js
const ReactElement = function (type, key, ref, self, source, owner, props) {
  const element = {
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  }

  return element
}
```

ReactElement 함수 내부를 보니 element라는 객체를 만들고 있었고 그 객체를 반환하고 있다. 그렇다면 React.createElement메서드를 통해서 element라는 어떤 객체가 만들어진다는 것을 알 수 있게 되었다.

## Render 함수 내부에서 일어나는일: React Fiber

React Fiber는 react v16.0에서 소개된 new core algorithm이다. 리액트는 killer feature라고 할 수 있는 virtualDOM과 그것의 핵심 알고리즘인 reconciliation과정을 통해 렌더링 효율을 매우 높였다. 그런데 이 reconciliation에도 한계가 있었다.

old reconciliation의 한계: 두개의 트리를 비교하는데 문제가 있다. 두개의 트리를 비교할 때 이 두 virtual tree 상에서 차이점이 있는 것을 찾아내기 위해서 diff알고리즘이 진행될 텐데, 이때 두 객체를 비교하기 위해선 재귀적으로 진행할 수 밖에 없었다. 재귀 알고리즘은 본질적으로 call stack과 연관이 있습니다. 가장 상단에 있는 함수가 호출되면 해당 함수는 call stack 가장 아래에 쌓입니다. 내부적으로 함수가 호출될 때마다 반복적으로 call stack에 차근차근 쌓여갑니다. 그리고 함수가 반환되면 그때서야 함수는 call stack에서 pop됩니다. 비동기 작업들은 event loop가 call stack이 비어있는 엽를 확인한 후에야 콜백함수들을 call stack에 올려놓고 실행합니다. call stack이 비어있지 않다면, callback queue에 대기중인 함수들은 실행될 수 없을 것 입니다. 여기에 들어가 있는 함수들은 유저의 클릭이벤트가 될 수도 있고, setTimeout, 애니메이션 등등이 있습니다.

어플이 너무 방대해진 나머지 두 virtual tree를 비교하는 재귀 알고리즘이 100ms 동안 진행된다고 해보겠습니다. 그런데 50ms즈음에 유저의 이벤트가 들어왔습니다. 그럼에도 불구하고 아직 재귀 알고리즘은 50ms나 순회를 해야합니다. 그러니까 callstack의 공간을 50ms나 더 차지하고 있어야한다는 말이죠. 이렇게되면 즉각적으로 use event에 대응할 수 없을 뿐더러, 프레임 드롭이라는 문제를 일으킬 수도 있습니다.

> 프레임 드롭: 프레임 드롭이란 무엇일까요? 사용하는 기기에 따라 다르겠지만, 일반적인 윈도우 기기는 60 프레임율을 유지한다고 합니다. 저희가 화면에서 보는 영상은 모두 이미지로 이루어져있습니다. 그 이미지 여러장을 빠른 속도로 보여주니, 그것이 움직이는 동영상 처럼 보이게 되는 것 입니다. 60 프레임율이라는 말은 1초의 시간동안 60장의 이미지를 보여준다는 말입니다. 1초는 1000ms입니다. 그렇다면 60프레임을 1000으로 나누면 1프레임당 소요할 수 있는 시간은 약 16ms가 될 것 입니다. 이 숫자가 중요합니다. 이숫자에 프레임 드롭의 의미가 담겨져있습니다. 한 프레임 안에서 작업을 수행하는데 걸리는 시간이 16ms가 넘어가면, smooth한 화면을 보여줄 수 없게 됩니다. 화면이 뚝뚝 끊기면서 보이게 될 것 입니다. 추가적으로 브라우저가 housekeeping을 위해서 필요한 시간은 6ms정도가 됩니다. 그렇다면 저희에게 남은 시간은 10ms가 될 것입니다. 이 뜻은 저희가 해야할 작업을 10ms안에 끝내주지 않으면 프레임 드롭이 발생할 것이라는 것입니다.

앞서 말했듯이 두 virtual tree를 비교하는 작업은 재귀적으로 이루어집니다. call stack에 쌓여있는 모든 함수들이 return될 때까지 call stack을 비워주지 않습니다. 다른 말로하면, 시작하면 무조건 끝을 봐야한다는 것 입니다. 그런데 이 재귀적으로 tree를 순회하는 시간이 16ms를 넘어서면 프레임 드롭이 발생할 뿐더러, 어플의 크기에 따라 순회하는 시간이 길어지면 유저 이벤트에 즉각적으로 대응하는 것이 어려워집니다.

이 지점에서 react fiber가 등장합니다. react fiber가 해결하고자 하는 것은 이런 순회작업을 멈출수도 있고, 재개할 수도 있고, 필요에 따라서는 그냥 내다버릴 수도 있게 만드는 것입니다. 더 나아가 우선순위에 따라 이것을 처리함으로써, 더욱 똑똑하게 렌더링을 구현합니다.

한마디로 리액트 렌더링 알고리즘에 스케줄링을 구현한 것 입니다.

react fiber의 목적을 정리하면 다음과 같습니다.

- 작업을 멈추고 나중에 다시 시작한다.
- 다양한 종류의 작업에 따라서 우선순위를 부여한다.
- 완성된 작업물을 재사용할 수 있다.
- 더 이상 필요하지 않은 작업물이면 버릴 수 있다.

기존에 사용하던 객체 방식과 그것을 재귀적으로 순회하는 방식은 call stack에 종속됩니다. 위에서 살펴봤든 이런 구조는 특정 문제점을 가집니다.

리액트 팀은 자바스크립트 엔진의 call stack 대신 virtual stack을 구현합니다.

virtualDOM은 실제 DOM이 아니라 메모리 상에 존재하는 가상의 DOM입니다. virtual Stack도 비슷합니다. 실제 Stack이 아니라 메모리 상에 존재하는 가상의 Stack입니다. 더 나아가 스케줄링이 가능한.

리액트 팀은 이것을 단일 연결리스트를 이용해 구현합니다.

render 함수의 인자로 넘어온 element 객체는 그 안에 들어오면서 하나하나 fiber node로 변경됩니다. 그리고 그 node들은 모두 연결됩니다.

```js
class Node {
  constructor(instance) {
    this.instance = instance
    this.child = null
    this.sibling = null
    this.return = null
  }
}
```

각각의 fiber노드들은 3가지 필드를 가집니다.

- child: 자식 노드
- sibling: 형제 노드
- return: 부모 노드 (return을 부모 노드라고 부르는 것이 비관적으로 느껴질 수 있다. return 하고 나면 그 다음에 접근하게 되는 노드가 부모 노드이기 때문에 return 노드라고 부르는 것 같다.)

인자로 받아온 노드들을 모두 단일 연결 리스트로 연결 시켜주는 함수가 있다.

```js
function link(parent, elements) {
  if (elements == null) elements = []

  parent.child = element.reduceRight((prev, cur) => {
    const node = new Node(cur)
    node.return = parent
    node.sibling = prev
    return node
  }, null)
  return parent.child
}
```

그리고 이 link 함수는 parent노드의 가장 첫번째 자식을 반환합니다.

```js
const children = [{ name: "b1" }, { name: "b2" }]
const parent = new Node({ name: "a1" })
const child = link(parent, children)

child.instance.name === "b1" //true
child.sibling.instance === children[1] // true
```

또한 현재 노드와 자식 노드들의 연결을 도와주는 helper 함수가 있습니다.

```js
function doWork(node) {
  console.log(node.instance.name)
  const children = node.instance.render()
  return link(node, children)
}
```

그리고 이제 이렇게 연결된 함수들을 탐색하는 walk 함수가 있습니다. 이 탐색은 기본적으로 깊이 우선 탐색으로 이루어집니다.

```js
function walk(o) {
  let root = o
  let current = o
  while (true) {
    let child = doWork(current)
    //자식이 있으면 현재 active node로 지정한다.
    if (child) {
      current = child
      continue
    }

    //가장 상위 노드까지 올라간 상황이라면 그냥 함수를 끝낸다.
    if (current === root) {
      return
    }

    //형제 노드를 찾을 때까지 while문을 돌린다. 이 함수에서는 자식에서 부모로 올라가면서 형제가 있는지를 찾아주는 역할을 하고 있다.
    while (!current.sibling) {
      //top 노드에 도달했으면 그냥 끝낸다.
      if (!current.return || current.return === root) {
        return
      }

      //부모노드를 현재 노드에 넣어준다.
      current = current.return
    }
    current = current.sibling // while문을 빠져나왔다는 것은 sibling을 찾았다는 것이다. 찾은 sibling을 현재 current node에 넣어준다.
  }
}
```

중요한 점은 이 함수를 사용하면 스택이 계속해서 쌓이지 않는다는 것이다. call stack의 가장 아래에는 walk 함수가 깔려있고, 계속해서 doWork함수가 호출되었다가 사라지는 그림이 그려질 것이다.

이 함수의 핵심은 current node에 대한 참조를 계속해서 유지한다는 점이다. 떄문에 이 함수가 중간에 멈춘다고 할지라도, current node로 돌아와서 작업을 재개할 수 있다. 이런 구조를 통해서 재귀적 순회가 가진 문제를 해결한다. 재귀는 한번시작되면 무대뽀로 끝까지 실행 해야만 하지만, 이제는 중간에 멈추도 이전 작업 기록이 남아있으니 마음 놓고 멈출 수 있다.

지금까지 살펴본 예시 코드를 통해서 어떻게 fiber가 단일 연결리스트로 구현이 되어있는지, 어떻게 중간에 하던 작업을 멈췄다가도 그것을 재실행할 수 있는지를 알아봤습니다. 그렇다면 이제는 이 작업을 언제 실행하고 언제 멈추는지도 알아봐야겠습니다. 이것을 이해하기 위해서는 하나의 프레임 안에서 일어나는 일을 알아야 합니다.

## 하나의 프레임 안에서 일어나는 일

![image](https://user-images.githubusercontent.com/63354527/183253005-537f23a2-6e51-4664-9ddf-578a5bb59538.png)

일반적으로 16ms 안에 하나의 프레임이 실행될 것이라고 말씀드렸습니다. 16ms 동안에 어떤 일들이 일어나는 것일까요? 이 과정을 살펴봄으로써, fiber가 실행되는 순간과, 그것을 멈추는 순간에 대한 이해를 가질 수 있을 것입니다.

1. input event: 가능하면 빠른 유저 피드백을 주기 위해서 input event가 실행된다.
2. Timers: 예약된 시간에 도달했는지 확인하기 위해서 타이머를 확인한다. 그리고 나서 시간이 맞다면 대응하는 콜백함수를 실행해준다.
3. Begin Frame: Begin Frame을 확인한다. (이것은 각각의 프레임의 이벤트이다.) window.resize, scroll, media query change등등도 같이 확인해야한다.
4. requestAnimationFrame: rAF를 실행한다. painting이 시작되기 전에 callback이 실행된다.
5. layout: 레이아웃 작업을 수행한다. 각각 노드의 size와 위치값은 주어졌으니, 각 요소의 내용들이 브라우저에 의해 화면에 채워지는 단계이다.
6. paint: 각각 노드의 size와 위치값은 주어졌으니, 각 요소의 내용들이 브라우저에 의해 화면에 채워지는 단계다.
7. idle period: 브라우저가 유휴 시간에 들어간다. 할 일 없이 빈둥대는 시간이다.

하나의 프레임 동안에 일어나는 일들입니다. 여기서 저희가 주목해야 할 부분은 idle period입니다. 16ms라는 하나의 프레임 안에서 main task가 진행되고 나서 남은 시간이 있다면, 그 시간이 바로 idle period입니다. 브라우저가 할 일이 없어지는 시간.

브라우저가 할 일이 없어지는 순간에 특정 함수가 실행될 수 있도록 하는 api가 있습니다. 바로 requestIdleCallback입니다.

## requestIdleCallback과 fiber

```js
window.requestIdleCallback(callback, { timeout: 1000 })
```

requestIdleCallback함수는 브라우저가 빈둥대는 시간을 노립니다. 미리 콜백을 등록해두면서 브라우저에게 이렇게 말합니다. 브라우저야, 너 main task하고나서 여유있으면 이 callback 함수 실행할 시간좀 나눠줘. 그리고 뒤에는 timeout을 설정해놓았는데, 해당 타임아웃 시간에 도달하면 idle time이 있든 없든 무조건 callback을 실행시킵니다.

이 함수의 콜백함수가 받게 될 파라미터에는 deadline이라는 객체가 있습니다. 그리고 이 객체는 2가지 속성을 가지고 있습니다.

- timeRemining: current frame에서 얼마나 시간이 남아있는지를 return 합니다.
- didTimeout: callback task의 시간이 초과했는지 여부를 return 합니다.

fiber는 requestIdleCallback을 적극 활용합니다. fiber는 주어진 nodes를 잘게 쪼갭니다. 그리고 각각의 fiber node를 하나의 실행단위로 여깁니다. 'fiber = unit of work' 이렇게 잘게 쪼개진 작업들을 idle time에 하나하나씩 실행시키는 것입니다.

순서는 이렇습니다.

main task가 끝나고나면, 남는 시간이 있는지 묻습니다. 브라우저야 남는 시간 없어? -> 없어 -> 알겠어 너할일해

만약 남는 시간이 있다는 대답이 돌아오면, 그 시간을 자신이 활용합니다. cpu를 분배받은 react는 자신이 잘게쪼갠 작업의 단위를 하나씩 실행합니다.

그리고 하나를 실행할 때마다, 다시 브라우저에게 남는 시간이 있는지 물어봅니다.
남는 시간이 없다? 그러면 다시 브라우저에게 cpu를 넘겨줍니다.
근데 남는 시간이 있다? 그러면 자신에게 남아있는 task가 있는지 한번 더 확인해봅니다.
그렇게 확인해서 남은 작업이 없다면 브라우저에게 cpu를 넘겨주고,
남은 작업이 있으면 실행한 후 위의 과정을 반복합니다.

![image](https://user-images.githubusercontent.com/63354527/183253554-f9587638-8177-40d0-8c7a-5b7a2a05959d.png)

이것을 통해서 우리는 fiber가 언제 실행되고, 언제 멈추는지를 알 수 있게 되었습니다. 하지만 아직도 fiber에 대해서 알아야 할 것이 남아있습니다. fiber는 크게 2단계로 나누어집니다. 바로 render phase와 commit phase입니다. 각각에 대해서 알아보겠습니다.

## Fiber의 2가지 단계: render & commit

### render phase

render 단계에서는 작업을 멈췄다가 다시 시작하는 것이 가능합니다. 이 단계에서 하게되는 주된 일은 effect list를 모으는 것입니다. 여기서 말하는 effect란 node에서 일어나는 변경사항을 말합니다. 예를 들어 생성, 삭제, 속성 수정등이 있을 것 입니다. render 단계에서 순회를 하면서 이런 effect들을 list로 모으는 일을 합니다. render 단계의 결과물이 바로 effect list인 것입니다.

render가 순회하는 흐름을 간단하게 알아보겠습니다.

1. current node로부터 순회를 시작합니다. 만약 node에 수정이 필요하면, 그 내용에 알맞는 tag를 붙입니다. (INSERT, DELETE, UPDATE)
2. 자식 노드에 대한 fiber를 생성합니다. 만약에 자식 fiber가 생성되지 않으면 node는 끝난 것 입니다. 그러면 effect list는 부모 노드에 합쳐지고, current node의 sibling node로 순회를 갑니다. 만약 자식 fiber가 생성되면 그 node로 순회를 갑니다.
3. idle time을 한번 확인합니다. 그래서 시간이 남아있다면, 다음 node로 순회를 시작합니다. 시간이 없다면 다시 시간이 주어질 때까지 기다립니다.
4. 만약에 더 이상 노드가 없다면 pendingCommit 상태로 돌아갑니다. 이 상태는 effect list가 다 수집되었다는 것을 의미합니다.

이 effect list는 linear list로 이루어져 있습니다. tree를 순회하는 방식보다 훨씬 빠르기 때문에 commit 단계에서 실제 dom에 적용할 때 효율적일 것 입니다. 이 effect list가 구성되는 그림은 다음과 같을 것입니다.

![image](https://user-images.githubusercontent.com/63354527/183254150-c51e53a8-8920-4d06-a07a-20cb650b032f.png)

빨간색 쳐진 부분이 effect가 있는 노드들입니다. 그 effect들끼리 list가 형성되는 것입니다. 그렇게 나온 결과물은 다음과 같은 모습입니다.

![image](https://user-images.githubusercontent.com/63354527/183254173-adea823e-7699-4870-a1b2-2872a7ad8375.png)

이제 이렇게 effect list가 다 모아졌으면, commit phase로 넘어갑니다.
commit phase에 대한 설명으로 넘어가기 전에, 예시 코드를 살펴보겠습니다.

먼저 자식 vitual dom element 배열을 순회하면서 각각의 element를 fiber노드로 만드는 함수를 만들어보겠습니다.

```js
const reconcileChildren = (currentFiber, newChildren) => {
  let newChildIndex = 0
  let prevSibling // 이전 자식 fiber

  //while문에서 순회하면서 element를 fiber 로 만든다.
  while (newChildIndex < newChildren.length) {
    let newChild = newChildren[newChildIndex]
    let tag

    //여기 fiber type을 정의하기 위해서, if문에서 tag에 적절한 값들을 할당합니다.
    if (newChild.type === ELEMENT_TEXT) {
      tag = TAG_TEXT // type 이 ELEMENT_TEXT라는 것은 text라는 것을 의미합니다.
    } else if (typeof newChild.type === "string") {
      tag = TAG_HOST // string이라는 것은 native DOM이라는 의미힙니다.
    }

    let newFiber = {
      tag,
      type: newChild.type,
      props: newChild.props,
      stateNode: null,
      return: currentFiber,
      effectTag: INSERT,
      nextEffect: null,
    }

    if (newFiber) {
      if (newChildIndex === 0) {
        currentFiber.child = newFiber // 첫번째 child라는 것을 의미한다.
      } else {
        prevSibling.sibling = newFiber // 첫번째 자식의 형제를 두번째 자식을 가리키게 한다.
      }
      prevSibling = newFiber
    }
    newChildIndex++
  }
}
```

그리고 fiber node가 가지고 있는 effect를 모으고, effect list를 만들어내는 함수를 만들어보겠습니다.

이 각각의 fiber는 2가지 속성을 가지고 있습니다.

- firstEffect: effect를 가지고 있는 첫번째 자식을 가리킨다.
- lastEffect: effect를 가지고 있는 마지막 자식을 가리킨다.

그리고 nextEffect는 각각의 자식 fiber를 연결하고, 연결 리스트를 만드는데에 사용됩니다.

```js
const compleUnitOfWork = (currentFiber) => {
  let returnFiber = currentFiber.return
  if (returnFiber) {
    //만약에 부모 fiber의 firstEffect가 값을 가지고 있지 않다면, 이것이 currentFiber의 firstEffect를 가리키게 합니다.
    if (!returnFiber.firstEffect) {
      returnFiber.firstEffect = currentFiber.firstEffect
    }

    //만약 currentFiber의 lastEffect에 값이 있다면
    if (currentFiber.lastEffect) {
      if (returnFiber.lastEffect) {
        returnFiber.lastEffect.nextEffect = currentFiber.firstEffect
      }
      returnFiber.lastEffect = currentFiber.lastEffect
    }

    const effectTag = currentFiber.effectTag

    if (effectTag) {
      if (returnFiber.lastEffect) {
        returnFiber.lastEffect.nextEffect = currentFiber // nextEffect는 두 자식 fiber 사이를 연결짓습니다.
      } else {
        returnFiber.firstEffect = currentFiber
      }
      returnFiber.lastEffect = currentFiber
    }
  }
}
```

마지막으로 모든 fiber node를 순회하고 effect list를 만들어내는 함수를 만들어보겠습니다.

```js
const performUnitOfWork = (currentFiber) => {
  beginWork(currentFiber)

  //자식 node가 있으면 자식 먼저 순회하도록
  if (currentFiber.child) {
    return currentFiber.child
  }

  //자식이 이제 없다면, effect 수집. 자신 -> 형제 -> 부모순으로
  while (currentFiber) {
    completeUnitOfWork(currentFiber)
    if (currentFiber.sibling) {
      return currentFiber.sibling
    }
    currentFiber = currentFiber.return // 만약 형제 node가 없으면 부모 node로 이동.
  }
}
```

### commit phase

commit phase에서는 중간에 작업을 멈출 수 없습니다. 이 단계에서 실제로 dom에 변형이 일어납니다. 이전 단계에서 모아졌던 effect list를 한 번에 dom에 적용하는데, 이때는 멈추는 일 없이 한번에 적용합니다. 이 단계 또한 예시코드로 살펴보면 좋을 것 같습니다.

아래 코드는 commitWork 함수로, currentFiber가 가지고 있는 effectTag에 따라서 실제 dom에 적용해주고 있습니다.

```js
const commitWork = (currentFiber) => {
  if (!currentFiber) return
  let returnFiber = currentFiber.return
  let returnDOM = returnFiber.stateNode // 부모 요소

  if (currentFiber.effectTag === INSERT) {
    returnDOM.appendChild(currentFiber.stateNode)
  } else if (currentFiber.effectTag === DELETE) {
    returnDOM.removeChild(currentFiber.stateNode)
  } else if (currentFiber.effectTag === UPDATE) {
    if (currentFiber.type === ELEMENT_TEXT) {
      if (currentFiber.alternate.props.text !== currentFiber.props.text) {
        currentFiber.stateNode.textContent = currentFiber.props.text
      }
    }
  }
  currentFiber.effectTag = null
}
```

이렇게 commitWork 함수를 정의해줬습니다. 그러면 실제로 effect list를 순회하면서 commitWork를 호출하는 함수가 필요합니다. 그 함수는 commitRoot입니다.

```js
const commitRoot = () => {
  let currentFiber = workInProgressRoot.firstEffect
  while (currentFiber) {
    commitWork(currentFiber)
    currentFiber = currentFiber.nextEffect
  }
  currentRoot = workInProgressRoot // Assign the current root fiber that is successfully rendered to currentRoot
  workInProgressRoot = null
}
```

이제 마지막입니다. requestIdleCallback에 넣어줄 workloop 함수를 정의해보겠습니다. 이 함수 안에서 render phase와 commit phase의 작업이 이루어질 것 입니다.

```js
const workloop = (deadline) => {
  let shouldYield = false // 작업을 할지 말지
  while (nextUnitOfWork && !shouldYield) {
    // 여기가 render phase
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
    shouldYield = deadline.timeRemaining() < 1 // performUnitOfWork작업을 한 후에 1ms도 남지 않았으면, 브라우저에게 다시 통제권을 넘길 것이다.
  }
  if (!nextUnitOfWork && workInProgressRoot) {
    console.log("The end of the render stage ")
    commitRoot() // 여기가 commit 단계. 여기서 effect list에 있는 모든 변경 사항을 view에 적용한다.
  }
  // Request the browser to reschedule another task
  requestIdleCallback(workloop, { timeout: 1000 })
}
```

## 결론

virtualDOM은 DOM전체를 렌더링하지 않고, 메모리 상에서 필요한 부분만 찾아내 실제 DOM에 적용시키기 때문에 좋다. 또한 렌더링 작업을 스케줄링 할 수 있기 때문에 좋다.

리액트에 대해 비유하는 단어 Value UI-> 변수 안에 리액트가 표현할 UI가 담겨 있다.
