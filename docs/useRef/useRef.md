# useRef

```js
const refContainer = useRef(initialValue)
```

`useRef`는 `.current` 프로퍼티로 전달된 인자(`initialValue`)로 초기화된 변경 가능한 ref객체를 반환합니다. 반환된 객체는 컴포넌트의 전 생애주기를 통해 유지됩니다.

일반적인 유스케이스는 자식에게 명령적으로 접근하는 경우입니다.

```js
function TextInputWithFocusButton() {
  const inputEl = useRef(null)
  const onButtonClick = () => {
    // `current` points to the mounted text input element
    inputEl.current.focus()
  }
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  )
}
```

본질적으로 `useRef`는 `.current` 프로퍼티에 변경 가능한 값을 담고 있는 "상자"와 같습니다.
