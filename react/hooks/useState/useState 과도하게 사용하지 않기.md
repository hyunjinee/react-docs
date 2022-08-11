# useState 과도하게 사용하지 않기

## state란 무엇일까?

- props를 통해 부모로부터 전달되면 state가 아니다.
- 시간이 지나도 변하지 않으면 state가 아니다.
- 컴포넌트의 다른 state 또는 props를 기반으로 계산할 수 있으면 state가 아니다.

상태관리 방식의 대부분은 다음과 같다.

```js
import { fetchData } from "./api"
import { computeCategories } from "./utils"

const App = () => {
  const [data, setData] = React.useState(null)
  const [categories, setCategories] = React.useState([])

  React.useEffect(() => {
    async function fetch() {
      const response = await fetchData()
      setData(response.data)
    }

    fetch()
  }, [])

  React.useEffect(() => {
    if (data) {
      setCategories(computeCategories(data))
    }
  }, [data])

  return <>...</>
}
```

언뜻 보기에 괜찮아 보인다면 다음과 같이 생각했을 것이다. 데이터 페칭 effect와 카테고리를 데이터와 싱크를 맞춰 state로 유지하는 또 다른 effect가 있다. 이것이 현재 useEffect 훅의 목적이다. 그렇다면 이 접근 방식의 단점은 무엇일까?

## 동기화 해제

이것은 실제로 잘 작동하며 가독성이 떨어지거나 추론하기에 어려움은 없다. 문제는 향후 다른 개발자가 'publicly'하게 사용가능한 `setCategories`가 있다는 것이다.

useEffect로 의도한 것이 카테고리를 서버 데이터에 의존하도록 설계한 것이었다면 이는 좋지 않은 소식이다.

```js
import { fetchData } from "./api"
import { computeCategories, getMoreCategories } from "./utils"

const App = () => {
  const [data, setData] = React.useState(null)
  const [categories, setCategories] = React.useState([])

  React.useEffect(() => {
    async function fetch() {
      const response = await fetchData()
      setData(response.data)
    }

    fetch()
  }, [])

  React.useEffect(() => {
    if (data) {
      setCategories(computeCategories(data))
    }
  }, [data])

  return (
    <>
      ...
      <Button onClick={() => setCategories(getMoreCategories())}>
        Get more
      </Button>
    </>
  )
}
```

과연 이제는 어떨까? 우리는 카테고리가 무엇인지 예측할 수 있는 방법이 없다.

- 처음 페이지가 로드 됐을 때 A 카테고리
- 사용자가 버튼을 클릭했을 때 B 카테고리
- 네트워크에 다시 연결할 때 refetch할 수 있는 기능이 있는 (useSWR, react-query) 라이브러리를 사용하고 있어 다시 실행한다면 카테고리는 다시 A

무심코 작성한 우리의 코드는 간헐적으로 발생하는 상황으로 인해 추적하기 어려운 버그가 생겼다.

## 필요없는 state

이것은 결국 useState에 관한 것이 아니라 useEffect에 대한 잘못된 사용법이다. 이러한 방법은 React 외부의 값을 동기화하는데 사용되어야한다. useEffect를 사용하여 두 state를 동기화하는 것은 대부분 옳지 않다.

그래서 다음과 같이 가정하고자 한다.

> state setter 함수가 useEffect에서 동기적으로 사용된다면 state를 제거하자.

짧지만 강력한 조언이다. 더 나아가서 계산 비용이 비싸다는 것을 프로파일링 해서 증명하지 않는 한 메모이제이션하는 것에 대부분 문제 삼지 않을 것이다. (성급하게 최적화하지 말고 프로파일링을 통해 근거를 만들자.)

```js
import { fetchData } from './api'
import { computeCategories } from './utils'

const App = () => {
  const [data, setData] = React.useState(null)
-  const [categories, setCategories] = React.useState([])
+  const categories = data ? computeCategories(data) : []

  React.useEffect(() => {
    async function fetch() {
      const response = await fetchData()
        setData(response.data)
      }

      fetch()
    }, [])

-  React.useEffect(() => {
-    if (data) {
-      setCategories(computeCategories(data))
-    }
-  }, [data])

  return <>...</>
}
```

effect의 양을 줄이면서 복잡석을 줄이고, 이제 카테고리는 데이터에서 파생된 것을 명확하게 확인할 수 있다. 다른 개발자가 카테고리를 계산하려면 함수 내 computeCategories를 실행해야한다. 우리는 항상 카테고리가 무엇이며 어디에서 왔는지에 대한 명확한 이해를 할 수 있을 것이다.

이러한 리팩터링 과정은 우리는 이렇게 부른다.

**단일 진실 공급원(A single source of truth)**
