# Building solid Next JS architecture

Next JS는 서버 사이드 렌더링과 같은 견고한 기능을 제공하거나 간단한 정적 웹 사이트를 생성하는 react framework입니다. Next는 Gatsby와 CRA와 같이 많이 사용되는 react프레임워크입니다. 오늘 우리는 견고한 NEXT JS 프론트엔드 아키텍처를 구축하는 것에 대해서 이야기 할 것 입니다.

가장 기본적인 frontend package들은 다음과 같습니다.

1. styled-components for styling.
2. Redux, MobX for state management.
3. Tailwind or bootstrap for CSS.
4. Material UI or Ant design for web components.
5. Axios for API calls
6. Babel packages

다음과 같은 목록을 나열하는 것을 통해 우리는 react 어플리케이션의 아키텍처 수준을 결정할 수 있습니다.

```sh
yarn init
yarn add next react react-dom
```

이제 pages 폴더를 만들어줍니다. 이 폴더는 server side rendering에 사용됩니다. 이 폴더에 생성되는 파일들은 우리 웹사이트의 route또는 endpoint가 됩니다.

그 후 리덕스 폴더를 아래와 같이 세팅해줍니다.

![image](https://user-images.githubusercontent.com/63354527/170632500-4e4d0089-7ac9-42e8-8198-935d7de34797.png)

![image](https://user-images.githubusercontent.com/63354527/170632843-08e6e8d0-12d9-47ce-b0a1-29ea780ebbc0.png)

이제 styled-components를 추가할 것입니다. styled-components는 간단한 CSS Wrapper입니다.

styled-components를 추가후 내가 따르는 아키텍처는 웹 페이지의 모든 구성요소가 고유한 index.jsx와 styles.jsx 파일을 포함하고 있는 단일 구성요소로 처리도니다는 것 입니다.

이름 덕분에 인덱스 파일은 구성요소 HTML로 구성되고 스타일은 구성요소 CSS로 구성됩니다.

LandingPage라는 폴더 안에 about section, contact section등과 같은 하위 페이지로 구성되어 있습니다.
