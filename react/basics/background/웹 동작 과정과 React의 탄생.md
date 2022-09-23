# 웹 동작 과정과 React의 탄생

우리가 웹을 공부할 때 가장 먼저 배우는 것, HTML CSS Javascript 입니다. 기본 틀(마크업)은 `HTML`로 만들고, `CSS`로 옷(스타일)을 입히며 `Javascript`로 웹에 활기(동적으로 제어)를 불어넣어 줍니다.

![image](https://user-images.githubusercontent.com/63354527/191894908-b64da59a-09c4-41b5-89e1-6f7c1f3fa717.png)

위의 그림은 이 셋이 어떻게 뭉쳐서 웹 페이지를 구성하는지 보여줍니다.

> 웹 페이지가 렌더링되는 과정
>
> 1. HTML parser가 HTML을 바탕으로 DOM TREE를 그린다.
> 2. CSS parser가 CSS를 바탕으로 CSSOM을 그린다.
> 3. DOM에 CSSOM을 적용하여 Render Tree를 만든다.
> 4. Render Tree를 바탕으로 Painting 하여 실제 화면에 렌더링한다. (HTML 코드를 읽어 내려가다 스크립트 태그를 만나면 파싱을 잠시 중단하고 JS 파일을 로드한다.)

Render은 화면에 표시하다라는 뜻을 가지고 있습니다. Painting은 말 그대로 화면에 그린다라는 뜻입니다.

## DOM과 CSSOM

DOM먼저 알아봅시다.

> DOM(Document Object Model)은 HTML 요소들의 구조화된 표현으로, 객체에 해당합니다.

객체는 알겠는데, 구조화된 표현은 무엇일까요?

![image](https://user-images.githubusercontent.com/63354527/191895528-279fe668-0c48-4c37-9ceb-6f13f14c291d.png)

다음과 같은 html 코드가 있다고 해봅시다. 우리 눈에는 태그들로 감싸져있고 각각이 무슨뜻인지 알지만, 웹을 그리는 브라우저나 HTML을 동적으로 제어하는 JavaScript에게는 그저 텍스트에 불과합니다.

TV를 생각해볼게요! TV가 일반인인 우리에게 어떤 내부 구조로 이루어진지 알 수 없습니다. 하지만 리모컨을 통해서 쉽게 조작할 수 있죠. 우리가 알아듣기 어려운 내용을 리모컨을 통해서 우린 쉽게 접근할 수 있게 되었습니다.

다시 돌아와서, HTML을 TV에, DOM을 리모컨에 비유할 수 있겠습니다. 그저 텍스트인 HTML을 DOM 이라는 트리구조의 객체로 변환시켜서 이를 읽는 브라우저와 Javascript가 이해할 수 있는 구조로 HTML 파서가 DOM으로 바꿔주는 것 입니다.

![image](https://user-images.githubusercontent.com/63354527/191895653-670f739e-db8e-4fa6-a92e-3bb99d349018.png)

위의 index.html을 DOM으로 추상화한 모습입니다. `html`이 부모 줄기에 해당하고 그 안의 태그인 `head`와 `title`은 자식 나뭇가지에 해당하며 그 안의 "My first web page"와 같은 컨텐츠는 잎에 해당하는 트리 구조로 추상화되는 것 입니다.

> DOM은 웹 브라우저와 JavaScript가 HTML을 이해하기 쉽도록 트리 구조로 파싱하여 만든 객체입니다.

CSSOM(Cascading Style Sheets Object Model)은 무엇일까요? 이름에서 짐작하셨든, DOM에 CSS를 입힌 것을 이야기합니다.

하나의 예시코드만 보고 간단히 넘어가도록 하겠습니다.

```ts
<!doctype html>
<html land="ko">
  <head>
    <title>My first web page</title>
  </head>
  <body>
    <h1>Hello, world!</h1>
    <p style="display: none;">How are you?</p>
  </body>
</html>

html
ㄴhead
  ㄴtitle
    ㄴMy first web page
body
ㄴh1
  ㄴHello, world!
```

DOM은 "How are you?"도 같이 포함되어야 합니다. 하지만 CSS에 포함된 `display: none` 때문에 CSSOM에서는 포함되지 않는 것 입니다.

> CSSOM은 DOM에 CSS가 적용된 객체 모델입니다.

이제 웹을 개발한다는 것은 정적인 HTML만을 작성하는게 아니라, JS를 통해 동적으로 동작할 수 있는 웹을 개발하는 것 입니다.

그럼 DOM을 동적으로 제어(조작)해야 하는데, 이를 JS가 해줄 수 있도록 DOM API라는 것이 기본적으로 제공됩니다.

![image](https://user-images.githubusercontent.com/63354527/191896235-37306ad8-3235-4023-bf58-818e97e4f57e.png)

DOM API를 통해서 DOM을 조작한다는 것은 일반저긍로 다음과 같은 동작을 의미합니다.

```
DOM API
1. DOM에 요소를 추가한다.
2. DOM에 요소를 수정한다.
3. DOM에 요소를 삭제한다.
```

## jQuery의 등장

"Write less do more"이라는 모토를 달고 출시한 jQuery에 대해 잠깐 소개하고자 합니다.

동적으로 DOM을 제어한다라는 측면에 있어 기본 vanilla JS 보다 직관적으로 이해하기 쉬운 코드 때문에 빠르게 배울 수 있고, 모토처럼 js 보다 적은 양의 코드로도 개발을 할 수 있기 때문에 오랜 기간 웹 생태계를 지배하게 됩니다.

우리는 h1 태그로 감싸진 컨텐츠의 색깔을 red로 바꾸고 싶습니다.

```ts
const h1 = document.querySelectorAll("h1")

h1.forEach((ele) => (ele.style.color = "red"))
```

querySelectorAll 로 h1 태그를 가진 요소들을 배열로 반환합니다. 각각의 요소들을 forEach() 메서드를 통해 하나씩 접근하여 요소의 색깔을 바꿉니다.

이 과정을 jQuery로는 아래와 같이 나타낼 수 있습니다.

```js
$("h1").css("color", "red")
```

$ 를 통해 'h1' 태그를 선택하고, 그 프로퍼티 중 css 에 접근하여 색깔을 바꿉니다.

어떤 코드가 이해하기 쉬우셨나요? 대다수의 사람들은 javascript 개발자가 아니더라도 jQuery로 짠 코드는 직관적으로 어떤 결과를 초래할 지 이해할 수 있을 것 같습니다.

## 새로운 강자, Virtual DOM

시간이 지나면서 jQuery는 조금씩 한계를 맞이하게 됩니다. 대표적으로 Facebook이나 instagram과 같이 규모가 큰 웹 어플리케이션이 나오면서 쉽게 개발할 수 있는 것도 중요하지만, 사용자와의 interaction이 많은 웹 앱 특성상 성능이 가장 중요했습니다. 다음과 같은 이유들로 jQuery에 대한 회의적인 시선이 나오게 됩니다.

첫째로, jQuery는 vanilla에 비해 로드해와야 할 패키지가 많아 간단한 코드를 작성하더라도 성능적으로 떨어집니다.

둘째로 jQuery는 조각칼로 DOM을 깎아서 만드는 것에 비유할 수 있어서 대규모 앱에서 유지보수가 쉽지 않습니다. 모듈화나 컴포넌트화 와는 거리가 있기 때문이죠.

셋째로 Interactive Web, 즉 위에서도 언급했듯 웹 어플리케션으로 불리는 요즘의 웹에선 사용자에 의한 동적인 DOM 조작이 잦은데, 이때 jQuery를 이용하면, 배치와 화면표시에 많은 연산을 발생시키기 때문에 브라우저의 성능이 낮아지는 문제가 있습니다.

![image](https://user-images.githubusercontent.com/63354527/191897037-b9f82620-d47f-4eaa-be75-5d390b649bba.png)

DOM을 직접 조작하는것은 생각보다 까다로운 일 입니다. 안정성이 떨어질 뿐만 아니라, DOM API가 너무 low-level이기 때문입니다. 이걸로 DOM을 조작하다 보면 코드의 복잡도가 매우 높아질 수 밖에 없습니다. 규모가 커질수록 말이죠. 따라서 앱의 규모가 커질수록 복잡도가 높아지기 때문에, 기본적으로 돔을 직접적으로 조작하는 jQuery와 같은 라이브러리가 legacy(옛날 코드) 취급을 받는 것 입니다.

아까 TV - 리모컨의 예시를 통해 HTML와 DOM의 관계를 설명드렸습니다. 같은 예시를 DOM과 Virtual DOM에도 적용할 수 있습니다.

TV가 DOM 이라면, 리모컨은 Virutal DOM 입니다. 사용자가 직접 TV의 내부구조를 알고 조작하긴 어렵습니다. 하지만 리모컨을 가지고 조작하는건 비교적 매우 쉬운 일이죠.

사용자가 Virtual DOM을 조작하는건 React와 같은 라이브러리가 비교적 쉽게 해줄 수 있습니다. 그리고 Virtual DOM이 실제 Real DOM을 직접 조작해 주는 것이지요. 그 조작하는 과정([Diffing 알고리즘](https://ko.reactjs.org/docs/reconciliation.html)은 React가 알아서 해주게 됩니다.

그렇게 VirtualDOM을 사용하는 React가 등장하게 됩니다. 오늘 설명할 기술 중 종착역에 왔으니, React가 웹을 렌더하는 과정을 먼저 설명 드린 후 Virtual DOM의 동작 방식을 설명드리는 순서로 기술하겠습니다.

1. 먼저 https://velog.io/@hyunjine 에 접속합니다.
2. 이 주소는 domain 주소에 해당하므로 위의 DNS 서버로 가서 실제 주소에 요청을 보냅니다.(서버에 요청을 보냅니다.)
3. 서버가 클라이언트에게 응답으로 index.html과 App.js 를 보내게 됩니다. (이때 React는 SPA 즉 Single Page Application이므로 index.html은 단 하나만 존재합니다.)
4. 서버로 부터 받아온 파일들로 Render Tree를 구성하고, 이를 바탕으로 실제 화면에 렌더링 합니다.

![image](https://user-images.githubusercontent.com/63354527/191897452-638ea8f0-3b4c-4a49-873f-b4bdaefad83b.png)

먼저 Domain 주소란 기억하기 어려운 IP주소(ex: 240.10.20.1) 대신에 www.naver.com 과 같이 도메인 이름을 붙인 주소를 의미합니다. www.naver.com 으로 접속하게 되면, 이를 관리하고 있는 DNS서버에서 자동으로 그 도메인에 해당하는 IP주소로 요청을 보내게 도와줍니다. 그게 바로 DNS 서버의 역할이죠.

그럼 SPA는 무엇일까요? 위에서도 짧게 말했듯이 Single Page Application의 줄임말 입니다. index.html 하나만 가지고 렌더링 하며 index.html에서는 App.js 라는 스크립트 파일 하나만 로드합니다. 하나만 로드하는데도 어떻게 모든 동작이 동작할 수 있냐는 물음이 나올 수 있는데, 이는 번들러가 해결해 줍니다!

번들러 (`webpack`등)란, 웹 어플리케이션을 동작시키기 위한 자원들을 하나로 묶고 조합해서 하나의 정적인 결과물을 만드는 도구입니다. 왜 하나로 묶어줄까요? 하나로 묶여있지 않으면, 그만큼 서버에 필요한 자원을 여러번 요청해야 하는데, 하나로 묶어서 경량화 시켜주면 그만큼 서버에 가하는 부하도 적어지고 로딩시간도 높일 수 있어 성능적으로 유리합니다.

조금 심화과정일 수 있지만, 마지막으로 딱 한걸음만 더 나아가 보겠습니다. 위와 같은 렌더링 과정은 서버에서 딱 한번 파일을 받아온 후엔 api call 이외의 요청은 모두 클라이언트에서 수행합니다. 이를 CSR(Client-Side-Rendering)이라고 하며, 인터렉션 과정에서 서버가 개입되지 않기 때문에 빠르다는 장점이 있습니다.

하지만, 첫 로딩에 하나의 페이지만 가져오기 때문에 사용자가 첫 화면을 보기까지 오랜시간이 걸릴 수 있다는 점과, index.html이 하나뿐이기 때문에 SEO에 분리하다는 점이 단점으로 꼽을 수 있습니다.

> SEO란 Search Engine Optimization으로 구글이나 네이버와 같은 검색엔진에서 검색을 했을 때 노출되는 것을 뜻 합니다.

주로 검색엔진은 <meta> 태그에서 해당 웨베 사이트의 정보를 크롤링해오는데, 이 방식은 index.html이 하나이기 때문에 페이지가 전환되어도 해당 정보는 유일하여 원하는 페이지에 대한 SEO가 적절하지 않을 수 있기 때문입니다.

## React에서의 Virtual DOM

React는 state와 props라는 개념이 있습니다. 쉽게 생각하면, 화면에서 변하는 값을 의미합니다.

React는 다음과 같은 경우에 리렌더링이 일어납니다.

1. Props가 변경되었을 때
2. State가 변경되었을 때
3. forceUpdate() 를 실행하였을 때.
4. 부모 컴포넌트가 렌더링되었을 때

위의 설명과 같이 React는 상태나 속성 값이 변하게 되면 리렌더링 이라는 과정을 통해서 화면에서의 값을 갱신합니다. 말 그대로 렌더링을 다시한다는 뜻으로 바뀐 상태나 속성 값으로 화면을 그린다는 뜻 입니다.

![image](https://user-images.githubusercontent.com/63354527/191898399-fcd80f81-d16b-48b3-b163-ca66760824c7.png)

만약 상태나 속성값이 변경된 경우, 변경된 값으로 React는 가상 DOM을 그리게 됩니다. 그린 Virtual DOM과 Real DOM을 비교하여 변경된 사항만 반영하여 해당 내용을 실제 DOM에서 수정한 이후 새로운 화면을 렌더링합니다.

```js
function HelloMessage() {
  const [name, setName] = React.useState("foo")

  return <div>Hello {name}</div>
}

ReactDOM.render(<HelloMessage name="bar" />, mountNode)
```

위의 코드는 name을 'foo' 에서 'bar'로 바뀐다면, 화면에서도 'Hello foo' 에서 'Hello bar'로 변경될 것 입니다. 앞서 언급한 대로 name은 state에 해당하는 값이기 때문에, 변경사항을 감지하고 Virtual DOM을 새로 그린 다음 Real DOM과 비교하여 바뀐 내용만 반영해서 새로 화면에 렌더링 합니다!

⭐️ React VDOM의 작동 과정

1. HelloMessage 에서 'Hello foo' 를 return하여 렌더링 중
2. state의 name이 'bar'로 변경
3. state값이 변경되었기 때문에 render 함수를 재호출
4. HelloMessage에서 'Hello bar'를 return
5. Virtual DOM에서 변경된 내용(name)을 감지, 해당 <div>안의 내용을 DOM에서 수정
6. 브라우저에서 변경 값을 감지하고 새로운 화면 렌더링

## 마치며

오늘은 리액트의 동작과정을 넓고 얇게 설명드렸습니다. 어떻게 자세히 동작하는지 보단, 원리와 나오게 된 배경에 주목하여 포스팅 하였으니, 좀 더 깊은 내용을 추후에 블로그에서 다뤄보도록 하겠습니다.

마지막으로 제가 면접질문으로 받았던 질문을 남기고, 본 포스팅의 내용을 바탕으로 해당 질문의 답을 생각해보시면 좋을 것 같습니다!

**자바스크립트나 제이쿼리 개발자에게 React로 개발하라고 설득하고 싶다면, 어떤 이유를 들어서 설득시키겠는가?**
