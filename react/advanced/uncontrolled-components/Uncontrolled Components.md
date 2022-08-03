# 비제어 컴포넌트

대부분의 경우에 폼을 구현하는데 제어 컴포넌트를 사용하는 것이 좋다. 제어 컴포넌트에서 폼 데이터는 React 컴포넌트에서 다루어진다. 대안인 비제어 컴포넌트는 DOM 자체에서 폼 데이터가 다루어진다.

모든 state 업데이트에 대한 이벤트 핸들러를 작성하는 대신 비제어 컴포넌트를 만들려면 ref를 사용하여 DOM에서 폼값을 가져올 수 있다.
예를 들어 아래 코드는 비제어 컴포넌트에 단일 이름을 허용한다.

```js
class NameForm extends React.Component {
  constructor(props) {
    super(props)
    this.handleSubmit = this.handleSubmit.bind(this)
    this.input = React.createRef()
  }

  handleSubmit(event) {
    alert("A name was submitted: " + this.input.current.value)
    event.preventDefault()
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={this.input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    )
  }
}
```

**비제어 컴포넌트는 DOM에 신뢰 가능한 출처를 유지하므로 비제어 컴포넌트를 사용할 때 React와 non-React코드를 통합하는 것이 쉬울 수 있다.** 빠르고 간편하게 적은 코드를 작성할 수 있지만, 그 외에는 일반적으로 제어된 컴포넌트를 사용해야한다.

React렌더링 생명주기에서 폼 엘리먼트의 value 어트리뷰트는 DOM의 value를 대체한다. 비제어 컴포넌트를 사용하면 React 초기값을 지정하지만, 그 이후의 업데이트는 제어하지 않는 것이 좋다. 이러한 경우에 `value` 어트리뷰트 대신 `defaultValue`를 지정할 수 있다.

컴포넌트가 마운트된이후에 defaultValue 어트리뷰트를 변경해도 DOM의 값이 업데이트되지 않는다.
또한 \<input type="checkbox">와 \<input type="radio">는 defaultChecked를 지원하고 \<select>와 \<textarea>는 defaultValue를 지원합니다.

### 파일 입력 태그

HTML에서 \<input type="file">은 사용자가 장치 저장소에서 하나 이상의 파일을 선택하여 서버에 업로드하거나 파일 API를 사용하여 JavaScript로 조작할 수 있다.

파일 API를 사용하여 파일과 상호작용해야 합니다. 아래 예시에서는 제출 핸들러에서 파일에 접근하기 위해서 DOM 노드의 ref를 만드는 방법을 보여주고 있습니다.

```js
class FileInput extends React.Component {
  constructor(props) {
    super(props)
    this.handleSubmit = this.handleSubmit.bind(this)
    this.fileInput = React.createRef()
  }
  handleSubmit(event) {
    event.preventDefault()
    alert(`Selected file - ${this.fileInput.current.files[0].name}`)
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Upload file:
          <input type="file" ref={this.fileInput} />
        </label>
        <br />
        <button type="submit">Submit</button>
      </form>
    )
  }
}

ReactDOM.render(<FileInput />, document.getElementById("root"))
```
