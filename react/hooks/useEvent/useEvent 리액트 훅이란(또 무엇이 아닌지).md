# useEvent 리액트 훅이란 (또 무엇이 아닌지)

## 진짜 문제를 해결하려는 노력

리액트의 실행 모델은 대부분 현재값과 이전값을 비교하여 동작한다.  
이는 컴포넌트 내부와 useEffect, useMemo, useCallback과 같은 훅에서 발생한다.

아래와 같은 컴포넌트가 있다고 가정하자.

```js
function MyApp() {
  const [count, setCount] = useState(0)

  return <Counter count={count} />
}
```

Counter 컴포넌트는 count 변수가 변경되면 리렌더링된다.
count가 변경될 때 어떠한 effect가 실행되기를 원한다고 가정해보자.

```js
function MyApp() {
  const [count, setCount] = useState(0)

  useEffect(() => {
    console.log(count)
  }, [count])

  return <Counter count={count} />
}
```

count를 useEffect 훅의 의존성 배열에 추가했기 때문에 effect는 count가 변경될 때마다 재실행될 것이다.

## 그래서 무엇이 문제인가?

이 모델을 사용하면 많은 리액트 개발자들은 컴포넌트가 너무 많이 리렌더되거나 훅이 너무 많이 재실행(때로는 무한히)된다는 동일한 문제에 직면하게 된다.

먼저 컨텍스트에 text라는 상태가 있고 SendMessage를 위한 또 다른 버튼 컴포넌트가 있는 채팅앱을 가정해보자.

```js
function Chat() {
  const [text, setText] = useState("")

  const onClick = () => {
    sendMessage(text)
  }

  return <SendButton onClick={onClick} />
}
```

문제는 text가 변경될 때마다 onClick 함수가 재생성된다는 것이다. sendButton에 전달된 이전의 onClick 함수와 참조가 동일하지 않으므로 모든 키 입력마다 SendButton을 쉽게 리렌더링할 수 있다.

useEffect를 너무 자주 실행하는 예시를 살펴보자. 이 예시는 Dan abramov의 트윗에서 가져온 예시이다. 여기에는 route.url이 바뀔 때마다 페이지 방문을 로깅하는 effect가 있다.

```js
function Page({ route, currentUser }) {
  useEffect(() => {
    logAnalytics("visit_page", route.url, currentUser.name)
  }, [route.url, currentUser.name])
}
```

사용자의 이름이 업데이트 될 때에도 페이지 방문을 기록하는데, 이는 원하지 않은 동작이다. currentUser.name을 의존성 배열에서 제거할 수도 있다. 하지만 이는 리액트에서 권장되지 않는다. 만약 의존성 배열이 effect 함수 내부의 모든 의존을 반영하지 않는다면 오래된 클로저(stale closures)와 추적하기 어려운 버그가 발생한다. 이는 리액트 코어팀이 강력하게 권장하는 react-hooks ESLint 플러그인에 "exhastive deps"룰이 있을 정도로 중요하다.

## 제안된 해결책: useEvent

이 새로운 훅은 의존도에 따라 새로운 함수를 만들지 않고도 함수에 대한 안정적인 참조를 보장하기 위해 작성되었다.

만약 useEvent가 있다면 클릭 핸들러를 감싸 text가 변경되더라도 참조가 변경되지 않는 함수로 만들 수 있다.

```js
function Chat() {
  const [text, setText] = useState("")

  const onClick = useEvent(() => {
    sendMessage(text)
  })

  return <SendButton onClick={onClick} />
}
```

이제 onClick은 렌더링 될 때마다 새로 생성되는 대신 항상 같은 함수를 참조한다. 따라서 SendButton이 계속 리렌러링될 일이 없다.

다음 예시였던 페이지 방문 로거도 살펴보자.

```js
function Page({ route, currentUser }) {
  const logVisit = useEvent((pageUrl) => {
    logAnalytics("visit_page", pageUrl, currentUser.name)
  })

  useEffect(() => {
    logVisit(route.url)
  }, [route.url])
}
```

이제 안정적인 logVisit 함수를 만들 수 있다. currentUser.name을 useEffect 함수 본문에서 제거하고 route.url이 변경될 때에만 effect를 실행시킬 수 있다.

## 이 훅이 해결하지 않은 것

`useEvent`훅은 만병 통치약이 아닙다. `useEvent`는 리액트에 또 다른 개념을 추가한다. 또한 리액트가 진정한 반응성이 아니라는 사실도 변함이없다. 진정한 반응성 프레임워크인 SolidJS 예시로 얼마나 간단히 로거를 만들 수 있는지 빠르게 살펴보자.

```js
function Page(props) {
  createEffect(() => {
    logAnalytics(
      "visit_page",
      props.route.url,
      untrack(() => props.currentUser.name)
    )
  })
}
```
