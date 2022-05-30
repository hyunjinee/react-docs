# [React 렌더링 이해 및 최적화(With Hook)](https://medium.com/vingle-tech-blog/react-%EB%A0%8C%EB%8D%94%EB%A7%81-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f255d6569849)

React 컴포넌트가 렌더링을 수행하는 시점은 다음과 같습니다.

1. Props가 변경되었을 때
2. State가 변경되었을 때
3. forceUpdate()를 실행하였을 때
4. 부모 컴포넌트가 렌더링되었을 때

1~3번과정을 통해 컴포넌트가 렌더링될 때, 자식 컴포넌트 또한 같은 과정으로 렌더링이됩니다. 하지만 컴포넌트에서 렌더링 결과에 전혀 영향을 미치지 않는 변경사항이 발견된다면, 불필요한 렌더링이 발생하므로 성능 손실이 발생합니다. 이는 렌더링에서 수행하는 로직이 많을 수록, 많은 컴포넌트를 출력할 수록 손실은 배가됩니다.

이럴때 사용할 수 있는 것이 shouldComponentUpdate라는 라이프 사이클 메서드입니다. 이 라이프 사이클 메서드를 정의하면 컴포넌트 렌더링을 제어할 수 있게 되므로, 데이터가 변경되어 렌더링이 필요한 경우에만 렌더링 작업을 수행할 수 있습니다.

![image](https://user-images.githubusercontent.com/63354527/169980632-5f6212c3-c745-4512-847f-ce118801683e.png)

shouldComponentUpdate메서드에서는 인자를 통해 변경되는 속성(Props)와 상태(State)를 수신할 수 있으며, 참을 반환하는 경우 렌더링을 수행하고, 거짓이면 갱신을 위한 렌더링을 수행하지 않습니다.

예외로 forceUpdate()는 shouldComponentUpdate를 무시하고 렌더링을 강행합니다.

## React.PureComponent

```js
class Pure extends React.PureComponent {
  render() {
    return <div>{this.props.name}</div>
  }
}
```

만일 변경되는 데이터가 모두원시값이거나 각각의 불변성을 보장할 수 있다면 React.PureComponent를 사용할 수 있습니다. React.Component에 shouldComponentUpdate가 미리 장착되어있는 컴포넌트라고 생각하면 됩니다. 불변성을 강조한 이유는 shallow equal, 얕은 비교를 통해 컴포넌트의 렌더링을 결정하기 때문에 넘겨받는 Props, 사용하는 State를 잘 확인한 후 써주면 좋습니다.

상위의 컴포넌트에서 변화가 일어난다면(예를 들어, Redux로 구현된 컴포넌트의 경우), 하위 컴포넌트에 렌더링을 명령하게 됩니다.따라서 shouldComponentUpdate나 PureComponent등을 사용하지 않고 큰 앱을 개발하게 된다면, 사소한 변경에 모든 컴포넌트가 렌더링을 행하기 때문에 성능저하를 일으킬 수 있습니다.

![image](https://user-images.githubusercontent.com/63354527/169982133-d8e7ea9c-4a93-48d8-9506-9a5a33a3205f.png)

그래서 불변성을 보장할 수 있다면 PureComponent를 사용하여 개발을 할 것을 권장합니다. 그런데 우리는 이 불변성을 지키는 것을 모르고 지나치는 상황이 발생합니다. 흔히 발생하는 시나리오를 가정해보겠습니다.

배열, 객체 리터럴의 경우

우리는 처음 React를 배울때 '불변성'에 대한 예제로 배열과 객체를 다룰 때 새로 생성하여 만드는 것을 배웁니다. 이유는 객체 (배열도 이하 객체로 통일)는 불변인 원시 데이터(문자열,숫자,불리언)과 달리 변경이 가능한(Mutable) 데이터이기 떄문에 그렇습니다.

```js
{ a: 10 } === { a: 10 } // false
```

그래서 객체와 객체의 동일(equal)을 증명하기 위해서 모든 내부에 있는 불변 데이터가 동일하다는 것을 입증해야합니다.

PureComponent를 구현할 때는 불변성을 지켜야합니다. 하지만 직접 객체리터럴을 생성할 경우 불변성을 해치게 됩니다. 하지만 직접 객체 리터럴을 생성할 경우 불변성을 해치게 됩니다. 흔히 다음과 같은 경우를 예로 들 수 있습니다.

```js
return (
  <FileuploadInput deny={["video", "gif"]}>
)
```

<FileUploadInput/> 컴포넌트에 deny prop에 배열을 넘겨주었습니다. 만일 <FileUploadInput /> 컴포넌트가 PureComponent로 이루어져있을 경우, 쓸 때 없이 렌더링 할 때 마다 얕은 평가를 진행하게 됩니다. 어차피 거짓 평가일텐데 낭비가 되는 것이죠.

**함수의 경우**

함수도 객체와 마찬가지로 불변데이터가 아닙니다.더군다나 객체와 달리 동일한 함수임을 증명하려면 상당히 지저분한 방법을 사용하게 됩니다. 따라서 함수는 리터럴로 생성하지 마세요. 다음과 같이 말입니다.

```js
return (
  <FileUploadInput
    deny={["video", "gif"]}
    onUpload{(file) => this.setState({file})}
  />
)
```

Funtion.prototype.toString()을 사용하여 shouldComponentUpdate를 구현하여 함수의 내용이 동일하지 않다는 전재하에 함수가 동일함을 증명할 수 있습니다. fase-deep-equal 패키지는 다음과 같은 함수의 동일함을 증명합니다.

```js
if (a.toString !== Object.prototype.toString)
  return a.toString() === b.toString()
```

**ReactElment의 경우**

가장 놓치기 쉬우면서 가장 비교하기 번거로운 친구입니다. Props로 사용하는 children은 우리가 사용하기 편하라고 마크업 구조로 작성하지만 결국에는 Props이기 때문에 렌더링 변경시의 비교 대상이 됩니다.

```js
return (
  <FileUploadInput
    deny={["video", "gif"]}
    onUpload={(file) => this.setState({ file })}
  >
    <strong>업로드</strong>
  </FileUploadInput>
)
```

이 경우 비교하기 위해서는 동일한 컴포넌트를 사용하는지, 그리고 내부적으로 모든 프로퍼티 (Props, State, Ref)가 동일한지 비교해야합니다. 따라서 사용할 수 있는 Props중에서는 동일함을 비교하는 비용이 높습니다.

우리가 위 데이터들을 사용하면서 렌더링의 영향을 받지 않도록 하려면 어떻게 해야할까요?

## 시나리오 극복하기

### 변하지 않는 상수 데이터의 경우

생태에 전혀 영향을 받지 않는 상수 데이터의 경우 이를 극복하는 방법은 쉽습니다. 렌더링 단계에서 생성하는 것이 아닌, 정적으로 생성하면 됩니다.

```js
const UPLOAD_DENY_TYPES = ["video", "gif"]
const UPLOAD_COMPONENT_CONTENT = <strong>업로드</strong>

class Render extends React.PureComponent {
  state = { file: null }
  render() {
    return (
      <FileUploadInput
        deny={UPLOAD_DENY_TYPES}
        onUpload={(file) => this.setState({ file })}
      >
        {UPLOAD_COMPONENT_CONTENT}
      </FileUploadInput>
    )
  }
}
```

### 즉각적인 반응이 필요하지 않은 데이터의 경우

거의 대부분 이벤트/데이터 핸들러가 이에 해당됩니다. 따로 클래스 속성으로 구현하면 됩니다.

```js
const UPLOAD_DENY_TYPES = ["video", "gif"]
const UPLOAD_COMPONENT_CONTENT = <strong>업로드</strong>

class Render extends React.PureComponent {
  state = { file: null }
  handleUpload = (file) => {
    this.setState({ file })
  }
  render() {
    return (
      <FileUploadInput deny={UPLOAD_DENY_TYPES} onUpload={this.handleUpload}>
        {UPLOAD_COMPONENT_CONTENT}
      </FileUploadInput>
    )
  }
}
```

참고로 위와 같이 화살표 함수로 구현한다면 따로 this를 바인딩할 필요가 없습니다.

### 즉각적인 반응이 필요한 데이터의 경우

여기서 부터는 렌더링 내에서 제어가 필요한 데이터들입니다. 다루기 까다로워지기 시작합니다. 메모이제이션이라는 개념이 등장하기 시작합니다.

> 메모이제이션은 컴퓨터 프로그램이 동일한 계산을 반복해야할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다.

즉, 아무리 복잡한 계산이라도 전달인자만 동일하다면 빠르게 값을 가져올 수 있는 것이죠. loadash의 memoize()를 통해 다음과 같이 구현할 수 있습니다.

```js
import memoize from "loadash/memoize"

const sum = memoize((a, b) => {
  console.log("a = ", a, "b = ", b)
  return a + b
})

sum(10, 15) // 25, console.log("a= 10, b= 15")
sum(10, 15) // 25 ,no console
sum(10, 15) // 25, no console
```

이를 이용하여 변경이 필요한 데이터만 메모이제이션으로 구현하여 불변성을 보장하여 불필요한 렌더링의 영향을 받지 않게 할 수 있습니다.

```js
import memoize from "lodash/memoize"

const UPLOAD_DENY_TYPES = ["video", "gif"]

class Render extends React.PureComponent {
  state = { file: null }
  handleUpload = (file) => {
    this.setState({ file })
  }

  renderUploadChildren = memoize((isUpload) => {
    return <strong>{isUpload ? "업로드되어 있음" : "업로드 하기"}</strong>
  })

  render() {
    return (
      <FileUploadInput deny={UPLOAD_DENY_TYPES} onUpload={this.handleUpload}>
        {this.renderUploadChildren(Boolean(this.state.file))}
      </FileUploadInput>
    )
  }
}
```

위 예제에서 파일 업로드 여부를 불리언 값으로 평가하여 참/거짓에만 결과를 가져오도록 합니다. 이제 FileUploadInput 컴포넌트의 Props는 불변한 데이터임을 보장할 수 있게 되었습니다.

## Hook을 사용해보실래요?

이 아티클의 본 목적입니다. Hook을 사용하여 위와 같은 시나리오들을 더욱 더 간결하고, 쉽게 대응할 수 있습니다. 또한 lodash나 fast-deep-equal 등이 필요없구요 (진짜로요!)

### Hook에서 PureComponent, shouldComponentUpdate 사용하기

```js
const HookPureComponent = React.memo((props) => {
  return <div>{props.message}</div>
})

const HookSCU = React.memo((props) => {
  return <div>{props.message}</div>
}, {currProps, nextProps} => {
  return currProps.message !== nextProps.message
})
```

React.memo() 를 통해 함수형 컴포넌트의 렌더링을 제어할 수 있습니다. 단순히 컴포넌트를 넣으면 PureComponent가 되고, 두 번째 인자로 현재 Props와 미래의 Props를 비교하여 shouldComponentUpdate 처럼 렌더링을 직접 제어할 수 있습니다.

### Hook에서 memoization 사용하기

React Hook에서는 memoization을 자체적으로 지원하고 있습니다. 대표적으로 useMemo()가 있습니다.useMemo()를 사용하여 Props를 넘겨줄 때 간편하게 사용할 수 있습니다.

```js
const FILE_UPLOAD_DENY_TYPES = ["video", "gif"]

function Render() {
  const [file, setFile] = React.useState(null)
  cosnt handleUpload = React.useCallback((uploadFile) => {
    setFile(uploadFile)
  }, [])

  return (
    <FileUploadInput
      deny={FILE_UPLOAD_DENY_TYPES}
      onUpload={handleUpload}
    >
      {React.useMemo(() => {
          return (
            <strong>{file? "업로드되어 있음": "업로드 하기"}</strong>
          )
      }), [Boolean(file)]}
    </FileUploadInput>
  )
}
```

useCallback()은 useMemo()의 핸들러 버전으로 값이 아닌 lodash/memoize 처럼 메모이제이션되는 함수를 가져온다고 생각하시면 됩니다. 차이점은 메모이제이션의 값이 변경되는 시점은 함수 인자가 아닌 의존성(dependencies)를 배열 형태로 받아서 판단을 합니다.

useMemo()나 useCallback()의 두번째 인자로 의존성 값들을 배열로 받아오는데, 여기에 불변한 값들을 넘겨주어 해당 값이 변경되면 감지를 하여 새로운 값(혹은 함수)을 만듭니다.

주의 할 점은 의존성 배열을 빈값으로 보내게되면 아무리 props.message 값이 변경되어도 값은 그대로 유지하게 됩니다. 즉, 절대 변하지 않는 값이 됩니다.

이를 이용하여 불변한 값을 제어하여 렌더링을 쉽게 제어할 수 있습니다.

PureComponent나 React.memo() 등을 사용하여 컴포넌트의 렌더링을 최적화했다고 하지만 만일 불변성이 보장되지 않은 값들이나 children과 같은 ReactElement를 받아와서 사용한다면 최적화가 의미가 없는 결과가 되는 것입니다. 어차피 매 렌더링마다 출력되는 것인데 쓸때없이 비교만 하는 것이 됩니다.

하지만 모르고 쓰는것하고 알고 쓰는것은 다르겠죠? 이제부터라도 고려하면 되는 것입니다. 저희도 잘못 사용한 컴포넌트들을 지속적으로 개선하고 있답니다.
co
git
