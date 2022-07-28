# React에 SOLID 원칙 적용하기

SOLID 원칙은 다음과 같다.

- 단일 책임 원칙 (Single responsibility principle, SRP)
- 개방-폐쇄 원칙 (Open-closed principle, OCP)
- 리스코프 치환 원칙 (Liskov substitution principle, LSP)
- 인터페이스 분리 원칙 (Interface segregation principle, ISP)
- 의존관계 역전 원칙 (Dependency inversion principle, DIP)

SOLID 원칙을 React에 어떻게 적용할 수 있는지 알아보자.

SOLID 원칙은 객체지향 프로그래밍 언어를 염두에 두고 구상되고 설명되었다. 이러한 원칙과 설명은 클래스와 인터페이스의 개념에 크게 의존하지만, JS에는 실제로 그런 개념이 없다. JS에서 우리가 흔히 "클래스"라고 생각하는 것은 프로토타입 시스템을 사용하여 시뮬레이션된 클래스 유사체일 뿐이며 인터페이스는 존재하지 않는다.

SOLID와 같은 소프트웨어 설계 원칙은 언어에 구애받지 않고 추상화 수준이 높다는 것입니다. 즉, 우리가 좀 더 자세히 살펴보고 입맛에 맞게 해석하면 함수형 React 코드에 더 적용할 수 있습니다.

## 단일 책임 원칙 (SRP)

원래 정의는 `모든 클래스는 단 하나의 책임만 가져야 한다`라고 명시되어있다. 이 원칙은 해석하기 가장 쉽다.

-> `모든 함수/모듈/컴포넌트는 정확히 한가지 작업을 수행해야한다`라는 정의라고 해석할 수 있다.

5가지 원칙중 SRP는 가장 따르기 쉽지만 코드 품질이 크게 향상하기 때문에 가장 영향력 있는 원칙이기도 하다. 컴포넌트가 한 가지 작업을 수행하도록 하기 위해 다음을 수행할 수 있다.

- 너무 많은 작업을 수행하는 큰 컴포넌트를 더 작은 컴포넌트로 나눔
- 주요 컴포넌트 기능과 관련 없는 코드를 별도의 유틸리티 함수로 추출
- 관련 있는 기능들을 커스텀 hook으로 캡슐화

이제 이 원칙을 적용하는 방법으 살펴보자. 먼저 활성 사용자 목록을 표시하는 다음 예제 컴포넌트를 고려해서 시작해보자.

```js
const ActiveUsersList = () => {
  const [users, setUsers] = useState([])

  useEffect(() => {
    const loadUsers = async () => {
      const response = await fetch("/some-api")
      const data = await response.json()
      setUsers(data)
    }

    loadUsers()
  }, [])

  const weekAgo = new Date()
  weekAgo.setDate(weekAgo.getDate() - 7)

  return (
    <ul>
      {users
        .filter((user) => !user.isBanned && user.lastActivityAt >= weekAgo)
        .map((user) => (
          <li key={user.id}>
            <img src={user.avatarUrl} />
            <p>{user.fullName}</p>
            <small>{user.role}</small>
          </li>
        ))}
    </ul>
  )
}
```

이 컴포넌트는 현재 비교적 짧지만 데이터를 가져오고, 필터링하고, 컴포넌트 자체와 목록의 각 항목을 렌더링하는 등 이미 많은 작업을 수행한다.

우선 서로 관련된 useState와 useEffect가 있을 때는 언제든지 커스텀 훅으로 추출할 수 있다.

```js
const useUsers = () => {
  const [users, setUsers] = useState([])

  useEffect(() => {
    const loadUsers = async () => {
      const response = await fetch("/some-api")
      const data = await response.json()
      setUsers(data)
    }

    loadUsers()
  }, [])

  return { users }
}

const ActiveUsersList = () => {
  const { users } = useUsers()

  const weekAgo = new Date()
  weekAgo.setDate(weekAgo.getDate() - 7)

  return (
    <ul>
      {users
        .filter((user) => !user.isBanned && user.lastActivityAt >= weekAgo)
        .map((user) => (
          <li key={user.id}>
            <img src={user.avatarUrl} />
            <p>{user.fullName}</p>
            <small>{user.role}</small>
          </li>
        ))}
    </ul>
  )
}
```

이제 useUsers hook은 오직 API에서 사용자를 가져오는 것 하나에만 관련되어있다. 또한 메인 컴포넌트 코드가 더 짧아졌을 분 아니라 용도를 파악해하는 구조적인 hook을 이름에서 용도를 바로 알 수 있는 도메인 hook으로 대체했기 때문에 보다 읽기 쉽다.

객체 배열을 순회하며 매핑하는 경우 배열의 각 항목에 대해 생성하는 JSX의 복잡성에 주의를 기울여야한다.

이벤트 핸들러가 연결되지 않은 한 줄짜리 마크업인 경우 인라인으로 유지하는 것이 좋지만 더 복잡한 마크업의 경우 별도의 컴포넌트로 추출하는 것이 좋다.

```js
const UserItem = ({ user }) => {
  return (
    <li>
      <img src={user.avatarUrl} />
      <p>{user.fullName}</p>
      <small>{user.role}</small>
    </li>
  )
}

const ActiveUsersList = () => {
  const { users } = useUsers()

  const weekAgo = new Date()
  weekAgo.setDate(weekAgo.getDate() - 7)

  return (
    <ul>
      {users
        .filter((user) => !user.isBanned && user.lastActivityAt >= weekAgo)
        .map((user) => (
          <UserItem key={user.id} user={user} />
        ))}
    </ul>
  )
}
```

이전 변경과 마찬가지로 사용자 항목을 렌더링하는 로직을 별도의 컴포넌트로 추출하여 메인 컴포넌트를 더 읽기 쉽게 만들었다.

마지막으로 API로부터 얻은 전체 사용자 목록에서 비활성 사용자를 필터링하는 로직이 있다. 이 로직은 비교적 독립되있고 애플리케이션의 다른 부분에서 재사용될 수 있으므로 유틸리티 함수로 쉽게 추출할 수 있다.

```js
const getOnlyActive = (users) => {
  const weekAgo = new Date()
  weekAgo.setDate(weekAgo.getDate() - 7)

  return users.filter(
    (user) => !user.isBanned && user.lastActivityAt >= weekAgo
  )
}

const ActiveUsersList = () => {
  const { users } = useUsers()

  return (
    <ul>
      {getOnlyActive(users).map((user) => (
        <UserItem key={user.id} user={user} />
      ))}
    </ul>
  )
}
```

이 시점에서 메인 컴포넌트는 더이상 나누지 않아도 될 만큼 짧고 간단하다. 그러나 조금 더 자세히 살펴보면 여전히 해야할 것보다 더 많은 일을 하고 있음을 알 수 있다.

현재 컴포넌트는 데이터를 fetching한다음 필터링을 적용하고 있지만 추가 조작 없이 데이터를 가져와 렌더링하기만 하는 것이 이상적이다.

마지막 개선사항으로 이 로직을 새로운 커스텀 훅으로 캡슐화할 수 있다.

```js
const useActiveUsers = () => {
  const { users } = useUsers()

  const activeUsers = useMemo(() => {
    return getOnlyActive(users)
  }, [users])

  return { activeUsers }
}

const ActiveUsersList = () => {
  const { activeUsers } = useActiveUsers()

  return (
    <ul>
      {activeUsers.map((user) => (
        <UserItem key={user.id} user={user} />
      ))}
    </ul>
  )
}
```

여기서는 fetching및 필터링하는 로직을 처리하기 위해 useActiveUsers hook을 만들어 메인 컴포넌트에는 hook에서 가져온 데이터를 렌더링하는 최소한의 작업만 남겨둡니다.

이제 한가지를 어떻게 해석하는지에 따라 컴포넌트가 여전히 먼저 데이터를 얻은 다음 렌더링하기 때문에 한가지가 아니라고 주장할 수 있습니다.

한 컴포넌트에서 hook을 호출한 다음 그 결과를 다른 컴포넌트에 props로 전달해서 더 나눌 수 있습니다.

여기서는 컴포넌트가 얻은 렌더링 데이터를 한가지로 너그럽게 받아들입니다.

요약하자면 단일 책임 원칙에 따라 우리는 큰 모놀리식 코드 덩어리를 효과적으로 가져와 더 모듈화합니다.

모듈화하면 코드를 파악하기 쉬워지고, 의도치 않은 중복 코드를 작성할 가능성이 줄어듭니다. 또한 작은 모듈은 테스트 및 수정하기 더 쉽기 때문에 결과적으로 코드를 보다 쉽게 유지 관리 할 수 있어 좋습니다.

## 개방-폐쇄 원칙 (OCP)

OCP는 소프트웨어 엔티티는 확장을 위해 열려야 하지만 수정을위해 닫혀야한다.라고 말한다.

React 컴포넌트와 함수는 소프트웨어 엔티티이기 때문에 정의를 바꿀 필요가 없이 원래 형태대로 사용할 수 있다.

개방 폐쇄 원칙은 원본 코드를 변경하지 않고 확장할 수 있는 방식으로 구성요소를 구조화하도록 권고한다.

실제로 작동하는 모습을 보기위해 다음 시나리오를 살펴보자. 다른 페이지에서 공유된 Header 컴포넌트를 사용하는 애플리케이션에서 작업하고 있으며, 현재 있는 페이지에 따라 Header는 조금 다른 UI를 렌더링해야한다.

```js
const Header = () => {
  const { pathname } = useRouter()

  return (
    <header>
      <Logo />
      <Actions>
        {pathname === "/dashboard" && (
          <Link to="/events/new">Create event</Link>
        )}
        {pathname === "/" && <Link to="/dashboard">Go to dashboard</Link>}
      </Actions>
    </header>
  )
}

const HomePage = () => (
  <>
    <Header />
    <OtherHomeStuff />
  </>
)

const DashboardPage = () => (
  <>
    <Header />
    <OtherDashboardStuff />
  </>
)
```

여기에서 현재 페이지에 따라 다른 페이지 컴포넌트를 위한 링크를 렌더링한다. 더 많은 페이지를 추가하기 시작할 때 어떤 일이 일어날지 생각하면 이 구현이 나쁘다는 것을 알 수 있다. 새 페이지가 생성될 때마다 Header 컴포넌트로 돌아가서 렌더링할 작업 링크를 알 수 있도록 구현을 조정해야한다. 이러한 접근 방식을 Header 컴포넌트는 취약해지고 사용되는 컨텍스트와 긴밀하게 결합되어, 개방-폐쇄 원칙에 위배된다.

이 문제를 해결하기 위해 컴포넌트 합성(component composition)을 사용할 수 있다. Header 컴포넌트는 무엇을 렌덜이할지 신경쓸 필요 없으며 대신 children prop을 사용해서 Header를 사용할 컴포넌트에게 이 책임을 위임할 수 있다.

```js
const Header = ({ children }) => (
  <header>
    <Logo />
    <Actions>{children}</Actions>
  </header>
)

const HomePage = () => (
  <>
    <Header>
      <Link to="/dashboard">Go to dashboard</Link>
    </Header>
    <OtherHomeStuff />
  </>
)

const DashboardPage = () => (
  <>
    <Header>
      <Link to="/events/new">Create event</Link>
    </Header>
    <OtherDashboardStuff />
  </>
)
```

이 접근 방식을 사용하면 Header 내부에 있던 변수 로직을 완전히 제거하고 이제 컴포넌트 자체를 수정하지 않고도 문자 그대로 원하는 모든 것을 입력할 수 있도록 합성(composition)을 사용할 수 있다. 그것에 대한 좋은 생각은 우리가 연결할 수 있는 컴포넌트에 플레이스 홀더를 제공하는 것이다.

여러 확장 포인트가 필요한 경우 (또는 children prop이 이미 다른 목적으로 사용된 경우) 대신 원하는 개수 만큼 prop을 사용할 수 있다.

Header에서 이러한 props를 사용하는 컴포넌트로 일부 컨텍스트를 전달해야하는 경우 render prop패턴을 사용할 수 있다.

개방 폐쇄 원칙에 따라 컴포넌트 간의 결합은 줄이고 확장성과 재사용성을 높일 수 있다.

## 리스코프 치환 원칙 (LSP)

LSP는 하위 타입 객체가 상위 타입 객체를 대체할 수 있는 객체간의 관계 유형이다. 이 원칙은 하위 타입과 상위 타입 관계를 정의하기 위해 클래스 상속에 크게 의존하지만, 클래스 상속은 고사하고 클래스를 거의 다루지 않기 때문에 React에서는 거의 적용할 수 없다. 클래스 상속에서 벗어나면 필연적으로 이 원칙이 완전히 다른 것으로 바뀌겠지만 상속을 사용하여 React 코드를 작성하면 의도적으로 나쁜 코드를 만들게 된다.

## 인터페이스 분리 원칙 (ISP)

ISP에 따르면 클라이언트는 사용하지 않는 인터페이스에 의존해서는 안된다. React 애플리케이션을 위해 이것을 컴포넌트는 사용하지 않는 props에 의존해서는 안됩다로 해석할 수 있다.

props와 인터페이스를 모두 객체와 바깥세계(이것이 사용되는 컨텍스트)사이의 계약으로 정의할 수 있기 때문에 둘 사이의 유사점을 그릴 수 있다. 결국 엄격하고 정의에 굴복하지 않고 문제를 해결하기 위해 일반적인 원칙을 적용해보는 것이다.

비디오 목록을 렌더링하는 애플리케이션을 생각해보자.

```js
type Video = {
  title: string,
  duration: number,
  coverUrl: string,
}

type Props = {
  items: Array<Video>,
}

const VideoList = ({ items }) => {
  return (
    <ul>
      {items.map((item) => (
        <Thumbnail key={item.title} video={item} />
      ))}
    </ul>
  )
}
```

각 항목의 Thumbnail 컴포넌트는 다음과 같다.

```js
type Props = {
  video: Video,
}

const Thumbnail = ({ video }: Props) => {
  return <img src={video.coverUrl} />
}
```

Thumbnail 컴포넌트는 매우 짧고 간단하지만 한가지 문제가 있다. 전체 Video 객체가 Props로 전달되어야하며 실질적으로 프로퍼티중 하나만 사용된다.

이것이 왜 문제가 되는 이유를 알아보기 위해 동영상 외에도 LiveStream에 대한 섬네일 이미지도 표시하기로 하고 두 종류의 미디어 리소스가 같은 목록에 혼재되어 있다고 가정해 봅시다.

```js
type LiveStream = {
  name: string,
  previewUrl: string,
}

type Props = {
  items: Array<Video | LiveStream>,
}

const VideoList = ({ items }) => {
  return (
    <ul>
      {items.map((item) => {
        if ("coverUrl" in item) {
          // 여긴 video입니다.
          return <Thumbnail video={item} />
        } else {
          // 여긴 live stream입니다만 우리는 여기서 무엇을 할 수 있을까요?
        }
      })}
    </ul>
  )
}
```

보시다시피 여기에는 문제가 있다. Video와 LiveStream 객체를 쉽게 구별할 수 있지만 Video와 LiveStream이 호환되지 않기 때문에 후자를 Thumbnail 컴포넌트에 전달할 수 없습니다. 첫째, 타입이 다르기 때문에 TypeScript는 즉시 문제를 제기합니다. 둘째, Video 객체는 'coverUrl', LiveStream 객체는 'previewUrl'이라는 서로 다른 프로퍼티에 섬네일 URL을 갖고 있다. 이것이 컴포넌트가 실제로 필요한 것보다 더 많은 props에 의존했을 때 겪게 되는 문제의 핵심이다. 재사용성이 낮아지기 때문에 이를 수정해 보자.

필요한 props에만 의존하도록 Thumbnail 컴포넌트를 리팩터링한다.

```js
type Props = {
  coverUrl: string,
}

const Thumbnail = ({ coverUrl }: Props) => {
  return <img src={coverUrl} />
}
```

이 변경으로 이제 Video와 LiveStream의 섬네일을 렌더링하는 데 사용할 수 있습니다.

```js
type Props = {
  items: Array<Video | LiveStream>,
}

const VideoList = ({ items }) => {
  return (
    <ul>
      {items.map((item) => {
        if ("coverUrl" in item) {
          // 여긴 video입니다.
          return <Thumbnail coverUrl={item.coverUrl} />
        } else {
          // 여긴 live stream입니다.
          return <Thumbnail coverUrl={item.previewUrl} />
        }
      })}
    </ul>
  )
}
```

인터페이스 분리 원칙은 시스템의 컴포넌트들 간 의존성을 최소화해 컴포넌트의 결합도를 낮추고, 재사용성을 높일 수 있도록 합니다.

## 의존 관계 역전 법칙 (DIP)

의존관계 역전 원칙은 구체화가 아닌 추상화에 의존해야한다라고 말합니다. 즉 한 컴포넌트가 다른 컴포넌트에 직접적으로 의존해서는 안되며, 둘다 공통된 추상화에 의존해야한다. 여기서 컴포넌트는 리액트 컴포넌트, 유틸리티 함수, 모듈 또는 서드파티 라이브러리와 같은 애플리케이션의 모든 부분을 나타낸다. 이 원칙은 추상적이므로 이해하기 어려울 수 있다.

```js
import api from "~/common/api"

const LoginForm = () => {
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")

  const handleSubmit = async (evt) => {
    evt.preventDefault()
    await api.login(email, password)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Log in</button>
    </form>
  )
}
```

이 코드에서 LoginForm 컴포넌트는 api 모듈을 직접 참조하므로 둘 사이는 긴밀히 결합되어 있다. 한 컴포넌트의 변경이 다른 컴포넌트에 영향을 미치도록 만드는 이러한 의존성은 코드를 변경하기 더 어렵게 하기 때문에 좋지 않다. 의존관계 역전 원칙은 이러한 결합을 깨는 것을 권하므로 이를 어떻게 할 수 있을지 살펴보자.

먼저 LoginForm 내부에서 api 모듈에 대한 직접 참조를 제거하고 대신 props를 통해 필요한 기능을 주입할 수 있도록 합니다.

```js
type Props = {
  onSubmit: (email: string, password: string) => Promise<void>,
}

const LoginForm = ({ onSubmit }: Props) => {
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")

  const handleSubmit = async (evt) => {
    evt.preventDefault()
    await onSubmit(email, password)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <input
        type="password"
        value={password}
        onChange={(e) => setPassword(e.target.value)}
      />
      <button type="submit">Log in</button>
    </form>
  )
}
```

이 변경으로 LoginForm 컴포넌트는 더 이상 api 모듈에 의존하지 않는다. API에 크리덴셜을 제출하는 로직은 onSubmit 콜백을 통해 추상화되었으며 이제 이 로직의 구체적인 구현을 제공하는 것은 상위 컴포넌트의 책임입니다.

이를 위해 폼 제출 로직을 api 모듈에 위임하는 연결된 버전의 LoginForm을 생성한다.

```js
import api from "~/common/api"

const ConnectedLoginForm = () => {
  const handleSubmit = async (email, password) => {
    await api.login(email, password)
  }

  return <LoginForm onSubmit={handleSubmit} />
}
```

ConnectedLoginForm 컴포넌트는 api와 LoginForm 사이의 접착제 역할을 하며 서로 완전히 독립적이다. 복잡한 의존성이 없기 때문에 깨지는 것에 대해 걱정하지 않고 독립적으로 반복하고 테스트할 수 있다. 그리고 LoginForm과 api가 모두 합의된 공통 추상화를 준수하는 한, 전체 애플리케이션은 예상대로 계속 작동할 것입니다.

과거에는 "Dumb" 프리젠테이셔널 컴포넌트를 생성한 다음 여기에 로직를 주입해서 사용했고, 이러한 접근 방식이 많은 타사 라이브러리에서도 사용되었습니다. 가장 잘 알려진 예로는 Redux가 있습니다. Redux에서는 connect HOC(고차 컴포넌트, higher-order component)를 사용해서 컴포넌트의 콜백 props를 dispatch 함수에 바인딩했다. hooks의 도입으로 이 접근 방식은 이제 관련성이 덜 하지만, HOC를 통해 로직을 주입하는 것은 여전히 React 애플리케이션에서 유용합니다.

> 역주 : Dumb 컴포넌트는 Redux의 창시자인 Dan Abramov가 고안한 컴포넌트 구조로 이전에는 "Smart", "Dumb" 컴포넌트로 불렀으나 지금은 "프리젠테이셔널(presentational)", "컨테이너(container)" 컴포넌트로 명칭을 바꿔 부르고 있습니다. ref)

결론적으로 의존관계 역전 원칙은 애플리케이션의 서로 다른 컴포넌트 간의 결합을 최소화하는 것을 목표로 한다. 아시다시피, 최소화는 개별 컴포넌트에 대한 책임 범위를 최소화하는 것 부터 컴포넌트 간의 인식 및 의존성을 최소화하는 것까지 모든 SOLID 원칙 전반에 걸쳐서 반복되는 주제이다.

## 결론

OOP 세상의 문제에서 태어났음에도 불구하고 SOLID 원칙은 그 이상으로 많이 적용된다. 이 글에서 우리는 이러한 원칙을 유연하게 해석해서 React 코드에 적용하고 유지 관리하기 쉽고 강력하게 만드는 방법을 살펴보았다.

그러나 독단적이고 종교적으로 이러한 원칙을 따르는 것은 악영향을 주고 오버-엔지니어링된 코드로 이어질 수 있다는 점을 기억하자. 따라서 컴포넌트를 추가로 분해하거나 분리해도 복잡도 개선에 거의 혹은 전혀 도움되지 않은 경우를 알아보는 방법을 익혀야한다.
