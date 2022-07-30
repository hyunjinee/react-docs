# Ref에 대한 고찰

ref는 references의 약자, React에서 특정 컴포넌트를 접근하는데 사용하는 props라고 이해하고 있으면 편합니다. 리액트에서 DOM을 컨트롤할 때 주로 이 ref를 이용하지, ref의 개념이 리액트를 처음 이용하는 사람들은 직관적으로 이해하기 쉽지 않고, 리액트를 어느 정도 활용할줄 아는 사람도 정확히 Ref가 먼지 풀어 설명하기 쉽지 않습니다.

때문에 이 포스팅에서는 ref에 대한 개념을 되새겨보고, 사용예시들을 바탕으로 이런 저런 고민해볼 점들에 대해서 다뤄보고자 합니다.

Ref는 render 메서드에서 생성된 DOM 노드나 React엘리먼트에 접근하는 방법을 제공합니다.

그러나, 일반적인 데이터 플로우에서 벗어나 직접적으로 자식을 수정해야하는 경우도 가끔 있습니다. 수정할 자식은 React 컴포넌트의 인스턴스일 수도 있고, DOM엘리먼트일 수도 있습니다.

React는 두 경우 모두를 위한 해결책을 제공합니다.

리액트 공식문서에서는 위와 같은 문구로 ref에 대한 소개를 하고 있습니다. 정리해보면 ref를 사용하는 케이스는 크게 두가지이죠.

1. 자식 컴포넌트를 직접 접근하여 수정할 때
2. DOM엘리먼트를 접근하고 싶을 때

1번 같은 경우는 리액트 함수 컴포넌트에서는 사용할 수 없는 방법이며,리액트에서 지양하고 있는 방법이므로, 진짜 사용하는 용도는 2번 케이스가 대부분입니다. 특정 엘리먼트 DOM에 접근해서 해당 DOM의 이벤트를 실행시키거나, 특정 attribute들에 접근할 수 있죠. 일반적인 방법으로는 컨트롤하기 힘든 것들이요.

Ref의 바람직한 사용 사례는 다음과 같습니다.

- 포커스, 텍스트 선택영역, 혹은 미디어 재생을 관리할 때
- 애니메이션을 직접적으로 실행시킬 때
- 서드 파티 DOM 라이브러리를 React와 같이 사용할 때

리액트 공식 문서에서는 위와 같은 케이스들을 구체적인 예시로 들고 있습니다.

실제 사용 예시를 보겠습니다.

```js
const useOutsideClick = ({ onClickOutside }) => {
  const ref = useRef(null);

  const handleClick = useCallback(
    (e) => {
      if (inside) return;

      onClickOutside();
    },
    [onClickOutside, ref]
  );

  useEffect(() => {
    document.addEventListener("click", handleClick);

    return () => document.removeEventListener("click", handleClick);
  }, [handleClick]);

  return ref;
};
```

위의 훅은 특정 엘리먼트 바깥이 클릭됐을 때 원하는 함수가 실행되도록 하는 훅입니다. ref를 이용해서, 클릭 이벤트 발생시, 해당 ref를 가지고 있는 컴포넌트가 click이벤틀에 포함된 엘리먼트인지 확인하죠.

활용

```js
const Use = () => {
  const ref = useOutsideClick({ onClickOutside : () => {
    console.log("outside 가 클릭되었음!");
  });

  return (
    <div>
      <h2 ref={ref}>
        inside
      </h2>
    </div>
  );

```

## ref는 어떤 값이 들어가야하는가?

리액트 엘리먼트의 ref 속성에는 과연 무엇이 들어가야 할까요? React.createRef() 로 호출된 값? useRef() 로 호출된 값? 다른 값은 들어갈 수는 없을까요?

우리가 잘 모르고 넘어가는 것 중 하나는 'useRef와 ref의 차이' 입니다. 컴포넌트 ref에 넣어줄 값을 useRef로 선언한다 정도만 알고 있거든요. 그러다보니, useRef를 조금 다르게 활용한다던가, ref 값에 다른 값이 들어간다던가 하면 혼동이 오더라고요. 이번 기회로 조금 더 깊게 들어가보죠.

## useRef란?

useRef는 엘리먼트 ref용 값을 선언할 때 사용하는 훅, 이라고 생각하신다면 이 말은 절반은 맞고 절반은 틀렸습니다. useRef는 아래의 타입을 지니고 있는 하나의 객체를 선언해주는 함수입니다.

```js
interface RefObject<T> {
  readonly current: T | null
}
```

useRef의 결과 값은 항상 위와 같은 타입을 갖게 되죠. 테스트를 해보죠. 리액트 컴포넌트 안에서 useRef를 호출하고, 콘솔로 찍어보죠.

```js
const testRef = useRef(null);
console.log({ testRef });
```

출력 결과

```
Object
testRef: {current: null}
[[Prototype]]: Object
```

정말 특별한게 없어보이죠? 제가 말한대로만 생각해보면 굳이 왜 이 훅을 사용해야하는가? 라는 생각이 들죠.

가장 중요한 점을 말하지 않았기 떄문이죠. 그것은 바로 useRef로 인해 반환도니 객체는 컴포넌트 전 생애주기를 통해 유지된다는 것입니다.

useRef로 만들어진 객체는 React가 만든 전역 저장소에 저장되기 때문에 함수를 재호출 하더라도 해당 컴포넌트의 생애주기 동안은, 계속 current 값을 유지하고 있을 수 있다는 뜻입니다.

그리고 또 하나 중요한점은 useRef로 부터 생성된 객체는 current 값이 변화해도 리렌더링에 관여하지 않는다는 점 입니. useState와 가장 큰 차이점 입니다.

이것은 useRef() 가 **순수 자바스크립트 객체**를 생성하기 때문입니다. useRef()와 {current: ...} 객체 자체를 생성하는 것의 유일한 차이점이라면 useRef는 매번 렌더링을 할 때 동일한 ref객체를 제공한다는 점 입니다.

리액트 공식문서에서도 위와 같이 설명하고 있습니다. 결과적으로 아래와 같은 케이스들에서 주로 useRef를 이용한 변수들이 사용됩니다.

- setTimeout, setInterval을 통해서 만들어진 id
- 외부 라이브러리를 사용하여 생성된 인스턴스
- scroll 위치

제 경험상으로는 비동기적 함수와 관련이 되어있다던가, 인터랙션 작업들에 관련이 된 변수들이 필요할 때, useRef를 필수적으로 이용하게 되는 것 같습니다. 예를 들면 스크롤에 따라, 특정 엘리먼트의 width에 따라, 이벤트가 발생하게 해야할 때, 인터렉션이 복잡해질수록 코드적으로도 많은 if 처리가 필요합니다. 따라서 이런 변화들이 매번 rerendering이 필요는 없을 것이고 이럴 때 ref.current로 인터랙션의 동작들을 세세하게 구분시켜주곤 합니다.

## ref의 타입

```js
interface RefObject<T> {
   readonly current: T | null;
}
type RefCallback<T> = { bivarianceHack(instance: T | null): void }["bivarianceHack"];
type Ref<T> = RefCallback<T> | RefObject<T> | null;
type LegacyRef<T> = string | Ref<T>;
```

위는 실제 @types/react에 담겨있는, ref props의 타입들입니다. 맨 아랫줄 부터 읽어보죠.

직접적으로 ref속성에 들어가는 타입은 LegacyRef<T>의 타입입니다. LegacyRef<T>는 string값, 혹은 Ref<T>가 들어갈 수 있는군요?

헌데 Ref<T>는 callback 형태인 RefCallback<T>, object 형태인 RefObject<T>, 그리고 null이 가능하네요.

string이 들어가는 것은 Legacy이므로 제외하고, null도 제외한다면 실제로 ref에 들어가는 형태는 크게 RefCallback과 RefObject 두가지 입니다.

useRef의 타입도 한번 살펴볼까요?

일반적으로 useRef훅으로 호출되는 값의 타입은 다음과 같습니다.

```js
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}
```

그렇습니다. 위에서 말했던 것과 같이, useRef는 결국 컴포넌트 생태주기를 함께하는 {current: T} 형태의 객체를 선언하는 것에 불과하다는 것이죠.

React는 노드가 변경될 때마다 변경된 DOM 노드에 그것의 .current 프로퍼티를 설정할 것입니다. 리액트에서 이미 컴포넌트의 ref props로 들어온 객체의 current프로퍼티를 설정하도록 미리 구현이 되어있습니다. 즉 ref에 useRef를 통해 생성된 객체를 집어 넣어주면 해당 컴포넌트가 변경될 때마다 객체의 current 프로퍼티가 컴포넌트의 DOM 객체로 설정이 되고, 우리는 그 DOM객체를 이용할 수 있게 됩니다.

useRef 대신 global 변수로 inputEl을 선언하면 어떨까? 아래는 동작한다. 왜냐? ref는 그냥 자바스크립트 객체이니까.

```js
const inputEl = { current: null };

function App() {
  const onButtonClick = () => {
    inputEl.current.focus();
    inputEl.current.click();
  };
  const onInputClick = () => {
    alert("input clicked");
  };

  console.log({ inputEl });

  return (
    <>
      <input onClick={onInputClick} ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
```

RefCallback<T> 형태는 무엇일까?

useRef 포함해서 지금껏 이야기했던 ref들은 전부 RefObject<T> 형태의 ref였습니다. 그렇다면, RefCallback<T> 형태의 ref는 뭘까요?

바로 callbackRef입니다. ref에는 콜백형태로 값을 집어넣을 수 있습니다. 이 콜백 ref를 이용해서 ref가 설정되고, 해제되는 상황의 동작들을 세세하게 다룰 수 있습니다. 아래 코드를 보죠.

```js
import React, { useState, useCallback } from "react";

export default function App() {
  const [height, setHeight] = useState(0);
  const callbackRef = (element) => {
    if (element) {
      setHeight(element.getBoundingClientRect().height);
    }
  };

  return (
    <div>
      <input ref={callbackRef} />
    </div>
  );
}
```

해당 엘리먼트가 변경되었을 때, 그 엘리먼트의 값에 dependant하게끔 함수를 실행시켜주고 싶다면? 바로 이 callbackRef를 이용합니ㅏㄷ. 위 예시에서는, input 엘리먼트에 callbackRef를 집어넣어줘서, input엘리먼트에 height 값을 바로 height 이라는 변수에 넣어주고 있습니다. 잘 활용하면 코드량도 줄어들고 생각보다 유용할 때가 많습니다.

지금까지의 내용을 복습할 수 있는 좋은 예시가 있습니다.

```js
const TestLi = ({ active }) => {
  const listRef = useRef < HTMLLIElement > null;

  useEffect(() => {
    if (active && listRef?.current) {
      listRef?.current?.scrollIntoView({ inline: "center" });
    }
  }, [active, listRef?.current]);

  return <li className={cx({ active })} ref={listRef} />;
};
```

위 코드는 아래와 같이 변환할 수 있습니다.

```js
const TestLi = ({ active }) => {
  const [listRef, setRef] = useState < HTMLLIElement > null;

  useEffect(() => {
    if (active && listRef) {
      listRef.scrollIntoView({ inline: "center" });
    }
  }, [active, listRef]);

  return <li className={cx({ active })} ref={setRef} />;
};
```

callbackRef를 이용해서,useState로 선언된 setRef를 넘겨줍니다. 그러면 li 엘리먼트가 변화할 때마다, listRef state 값에 해당 엘리먼트 DOM객체가 들어가게 되겠죠? 이런식으로 수정이 가능합니다.

그러면 useRef로 하면 될 것이지 왜 굳이 이렇게 코드를 짜냐고요? useRef의 의도를 다시한번 생각해보죠.

> if you put [ref.current] in dependencies, you're likely making a mistake. Refs are for values whose changes don't need to trigger a re-render.

즉 setRef에 첫번째 인자로 li element가 들어가고 listRef의 상태가 바뀌므로 컴포넌트를 리렌더링 시킬 수 있다. 이것이 callbackRef이다.

## useState & useRef

리액트에서 state란 리렌더링에 관여하는 변수를 의미합니다. 따라서 useState로 생성한 변수가 아닌 이상, 다른 변수들은 값이 변경되더라도 컴포넌트의 리렌더링을 발생시키지 않습니다.

useRef가 그 일례이죠. useRef가 함수 컴포넌트에서 변수를 사용할 때 종종 쓰이긴 하지만, 컴포넌트의 리렌더링이 필요할 때는 사용하는 것이 의미가 없습니다.

## useRef와 global Variable

global Variable은 컴포넌트 외부의 let 또는 const로 선언하는 variable을 의미합니다. 내부에서 선언되는 variable은 컴포넌트 리렌더링시마다 초기화됩니다. 이는 비교적 당연하게 받아들여지는 케이스기 때문에 자세한 내용은 생략합니다.

useRef를 사용할 때마다 궁금했던점이 렌더링에 관여하지 않는 변수이면 그냥 컴포넌트 외부에 let으로 선언하는 변수와 무엇이 다른것일까라는 의문이 들곤 했습니다. 아니, 애초에 컴포넌트 외에 let 선언한 변수를 컴포넌트 내부에서 사용한 적이 거의 없다보니, global Variable은 시도 해본적이 거의 없었고, 뭔가 금기시되는 것처럼 느껴지기도 했죠.

컴포넌트 안에서도, global Variable을 변화시킬 수 있습니다. 이는 컴포넌트 내부에서 선언한 let const variable과 달리, 리렌더링마다 초기화 되지도 않죠. 이렇게 사용해도 문제가 일어나지는 않습니다.

단 컴포넌트를 1개만 사용할 때에만 말입니다.

리액트 컴포넌트는 재사용이 가능합니다. global하게 let으로 선언한 변수는 여러 컴포넌트를 사용한다면 그 값을 공유하게 됩니다. 컴포넌트가 사라졌다가 등장하게 되더라도, global한 변수의 값은 유지될 것입니다. 물론 이점을 잘 고려한다면 유용하게 사용할 수도 있습니다.

useRef로 선언한 객체는 이와 달리, 컴포넌트와 생애주기를 함께합니다. 컴포넌트가 생성됨과 동시에 값이 초기화되고, 컴포넌트가 unmount 되면 메모리에서 해제됩니다.

요약해서 global Variable과 가장 큰 다른점은, 컴포넌트 재사용시마다 변수도 각 컴포넌트에 맞게 새롭게 초기화 된다는 것을 의미합니다.
