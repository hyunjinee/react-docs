# Virtual DOM(버추얼 돔,가상 돔)을 직접 만들어보자

virtual DOM의 주요부분은 50줄 미만의 코드로 구현이 가능하다.

두 가지 컨셉이 있다.

- 버추얼 돔은 실제 돔 표현의 모든 종류이다. (실제 돔을 다양한 방식으로 표현한 것이 버추얼 돔이다.)
- 버추얼 돔 트리에서 어떤 것을 변경하면 새로운 버추얼 트리가 나온다. 변경된 새로운 트리와 이전 트리를 비교, 차이를 찾고 실제 DOM에 대해 필요한 최소한 (작은)변경만 수행한다.

우리의 돔 트리를 표현해봅시다.

우리는 어떻게든 DOM트리를 메모리에 저장해야한다. 이부분은 단순하게 JS객체로 구현할 수 있습니다.

```html
<ul class="list">
  <li>item 1</li>
  <li>item 2</li>
</ul>
```

이를 JS 객체로 표현해보겠습니다.

```js
{ type: ‘ul’, props: { ‘class’: ‘list’ }, children: [
  { type: ‘li’, props: {}, children: [‘item 1’] },
  { type: ‘li’, props: {}, children: [‘item 2’] }
] }
```

- DOM 요소를 다음과 같은 객체로 나타낼 수 있습니다.

```js
{ type: ‘…’, props: { … }, children: [ … ] }
```

- 일반 자바스크립트 문자열로 DOM 텍스트 노드를 표현합니다.

하지만 큰 규모의 트리를 이렇게 만드는 것은 꽤나 어렵습니다. 이를 편하게 하기 위해 헬퍼함수를 만들어 보겠습니다.

```js
function h(type, props, …children) {
  return { type, props, children };
}
```

자 이제 우리는 위의 함수를 활용해서 돔 트리를 아래와 같이 작성할 수 있습니다.

```js
h(‘ul’, { ‘class’: ‘list’ },
  h(‘li’, {}, ‘item 1’),
  h(‘li’, {}, ‘item 2’),
);
```

훨씬 깨끗해 보입니다. 한걸음 더 들어가보죠. JSX 들어보셨죠? 그걸 한번 사용해볼께요.

Bable JSX 문서를 보면 Babel이 이 코드를 번역한다는 것을 알 수 있습니다.

```js
<ul className=”list”>
  <li>item 1</li>
  <li>item 2</li>
</ul>
```

```js
React.createElement(‘ul’, { className: ‘list’ },
  React.createElement(‘li’, {}, ‘item 1’),
  React.createElement(‘li’, {}, ‘item 2’),
);
```

비슷한 부분이 있나요? React.createElement 를 우리의 h함수로 대체할 수 있다면, jsx pragma를 통해서 우리도 할 수 있습니다. 소스 파일 상단에 주석과 같은 줄을 포함하면 됩니다

```js
/** @jsx h */
<ul className=”list”>
  <li>item 1</li>
  <li>item 2</li>
</ul>
```

이것은 해당 jsx 에서 React.createElement대신에 h를 사용하라고 Babel에게 전달합니다. ‘h’ 대신 아무거나 넣으셔도 됩니다. (h 대신 트랜스파일 될것입니다)

이제 전에 적었던 사항들을 요약하면 다음과 같이 돔을 작성할 수 있습니다.

```js
/** @jsx h */
const a = (
  <ul className=”list”>
    <li>item 1</li>
    <li>item 2</li>
  </ul>
);
```

이제 이 코드를 Babel이 다음과 같이 트랜스파일 합니다.

```js
const a = (
  h(‘ul’, { className: ‘list’ },
    h(‘li’, {}, ‘item 1’),
    h(‘li’, {}, ‘item 2’),
  );
);
```

h함수를 실행하면, JS 오브젝트(우리의 가상 돔 표현)가 반환됩니다.

```js
const a = (
  { type: ‘ul’, props: { className: ‘list’ }, children: [
    { type: ‘li’, props: {}, children: [‘item 1’] },
    { type: ‘li’, props: {}, children: [‘item 2’] }
  ] }
);
```

우리의 돔 트리는 우리만의 구조를 가진 평범한 JS 객체로 표현됩니다. (굿)이제 이를 활용해서 진짜 돔을 만들어야 합니다. (우리의 돔트리를 실제 돔에 추가할수 없기 때문입니다)

먼저 몇 가지 가정을 하고 용어를 설정해보겠습니다.

- 모든 변수를 “$”로 시작하는 실제 돔(엘리먼트, 텍스트 노드)로 작성할 것입니다.
- 가상 돔 표현은 노드라는 변수에 포함됩니다.
- 리액트에서 처럼, 하나의 루트 노드만 가질 수 있습니다. 모든 다른 노드는 루트 노드 안에 위치합니다.

이제 가상 돔 노드를 가져와서 실제 돔 노드를 반환하는 createElement함수를 작성해보겠습니다. props와 children은 잠시 잊어도 됩니다.

```js
function createElement(node) {
  if (typeof node === ‘string’) {
    return document.createTextNode(node);
  }
  return document.createElement(node.type);
}
```

텍스트노드(JS 문자열), 엘리먼트(아래와 같은 형식의 JS 오브젝트)를 처리 할 수 있습니다.

```js
{ type: ‘…’, props: { … }, children: [ … ] }
```

따라서 함수에 가상 텍스트 노드와 가상 엘리먼트 노드를 전달 할 수 있어야 합니다. 그래야 동작합니다.

자 이제 children에 대해 생각해봅시다. (이것 또한 텍스트 노드 이거나 엘리먼트 입니다.) 그렇기 때문에 createElement함수로 생성할 수 있습니다. 재귀의 느낌이 납니까? 우리는 엘리먼트의 children에 대해 createElement함수를 호출하고 해당 엘리먼트에 대해 appendChild ()할 수 있습니다.

```js
function createElement(node) {
  if (typeof node === ‘string’) {
    return document.createTextNode(node);
  }
  const $el = document.createElement(node.type);
  node.children
    .map(createElement)
    .forEach($el.appendChild.bind($el));
  return $el;
}
```

굿, 나이스. props는 나중에 보자. props는 버추얼 돔의 기본 개념을 이해하는데 필요하진 않아. 단지 유연성을 위해 차후에 추가할거야.

우리는 버추얼 돔을 실제 돔으로 변경 하였습니다. 이제 우리의 버추얼 트리의 변화를 감지하는 것을 생각해봅시다. 기본 알고리즘을 만들어야 합니다. 알고리즘은 old, new 두 개의 버추얼 트리를 비교하여 실제 DOM에 필요한 변경만 수행합니다.

좋아,. 이제 updateElement라는 함수를 작성할 것입니다. 해당 함수는 3개의 파라미터를 받습니다. parent, newNode, oldNode!(parent는 가상 노드의 실제 DOM 요소-부모-입니다). 이제 위의 설명한 모든 경우를 처리하는 방법을 알아보겠습니다.

이전 노드가 없는 경우(노드가 새로 추가된 경우)

```js
function updateElement($parent, newNode, oldNode) {
  if (!oldNode) {
    $parent.appendChild(createElement(newNode))
  }
}
```

새로운 노드가 없는 경우(노드를 삭제해야 하는 경우)

이제 여기에서 문제가 있습니다. 만약 새로운 버추얼 트리 현재 위치에 노드가 없는 경우, 실제 돔(DOM)에서 제거를 해야합니다. 어떻게 해야할까요? 우리는 부모 엘리먼트를 알고 있기 때문에 $parent.removeChild를 호출하고 해당 함수에 실제 DOM 엘리먼트 참조를 전달해야 합니다. 하지만 우리는 해당 참조를 갖고 있지 않습니다. 만약 부모노드에서 우리 노드의 위치를 알고 있다면, $parent.childNodes[index]로 참조를 얻을 수 있습니다. 여기에서 index는 부모 엘리먼트에 있는 노드의 위치 입니다

```js
function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(createElement(newNode))
  } else if (!newNode) {
    $parent.removeChild($parent.childNodes[index])
  }
}
```

노드가 변경됨

첫째로 우리는 두 노드를 비교하고 노드가 실제로 변경되었는지를 알려주는 함수를 작성해야합니다. 노드는 텍스트, 엘리먼트 노드가 될 수 있다고 생각해야합니다.

```js
function changed(node1, node2) {
  return typeof node1 !== typeof node2 ||
         typeof node1 === ‘string’ && node1 !== node2 ||
         node1.type !== node2.type
}
```

그리고 현재 부모 노드에서 현재 노드의 인덱스를 활용하여 새로 생성된 노드로 쉽게 대체 할 수 있습니다.

```js
function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(createElement(newNode))
  } else if (!newNode) {
    $parent.removeChild($parent.childNodes[index])
  } else if (changed(newNode, oldNode)) {
    $parent.replaceChild(createElement(newNode), $parent.childNodes[index])
  }
}
```

Children 비교

마지막으로, 두 노드의 모든 자식(Children)을 비교해야 합니다. 실제로 각 노드마다 updateElement를 호출해야 합니다. (재귀 입니다)

그러나 코드를 작성하기 전에 고려해야할 사항이 있습니다.

- 텍스트 노드는 자식(children)을 가질 수 없기 때문에, 엘리먼트 노드만 비교를 해야합니다.
- 이제 현재 노드에 대한 참조를 부모로 전달해야합니다.
- 모든 자식(children)을 하나씩 비교해야 합니다. (어떤 시점에 ‘undefined’를 가질 수도 있습니다만 우리 함수는 그것을 처리 할 수 있습니다.)
- 인덱스, 자식(children) 배열의 child 노드의 인덱스입니다.

```js
function updateElement($parent, newNode, oldNode, index = 0) {
  if (!oldNode) {
    $parent.appendChild(createElement(newNode))
  } else if (!newNode) {
    $parent.removeChild($parent.childNodes[index])
  } else if (changed(newNode, oldNode)) {
    $parent.replaceChild(createElement(newNode), $parent.childNodes[index])
  } else if (newNode.type) {
    const newLength = newNode.children.length
    const oldLength = oldNode.children.length
    for (let i = 0; i < newLength || i < oldLength; i++) {
      updateElement(
        $parent.childNodes[index],
        newNode.children[i],
        oldNode.children[i],
        i
      )
    }
  }
}
```

축하해요! 완료되었습니다. 버추얼 돔 구현을 마무리 했습니다. 이글을 읽으면서 버추얼 돔이 어떻게 작동하는지, React가 어떻게 작동하는지에 대한 기본 개념을 이해했으면 합니다.
