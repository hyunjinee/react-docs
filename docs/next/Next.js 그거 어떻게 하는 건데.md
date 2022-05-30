# Next.js 그거 어떻게 하는 건데.

Next.js는 vercel팀에서 만든 정적 페이지와 서버사이드 렌더링을 하이브리드하게 구현할 수 있는 유연한 React Framework이다. 특히 국내에서는 SEO 최적화를 위해서 많이 사용되는 것으로 알려져있다.

Next.js는 공식문서의 Showcase에서 Next.js를 사용하는 기업에 대해 소개하는데 많은 유명 테크 기업에서 사용하기 때문에 충분한 검증이 이루어진 프레임워크라고 할 수 있다.

Next.js와 함께 vercel 플랫폼을 사용한다면 GitHub과 연동하여 마치 매직처럼 배포를 1분 만에 수행할 수 있다. 또한 작업사항을 main 브랜치에 병합하기 위해 PR을 날리면 페이지 내에서 pre-deploying을 수행하여 preview 기능을 제공하는데 이게 또 기가 막힌다. 협업 관점에서 정말 편리한 기능이다.

## Why Next.js

> Next.js gives you the best developer experience with all the features you need for production: hybrid statc & server rendering, TypeScript support, smart bundling, route pre-fetching, and more. No config needed.

**zero config**

Next.js는 React를 보다 더 편리하게 사용할 수 있도록 한번더 추상화하여 개발자에게 고수준 (high level)의 기능들을 제공한다. 즉 사용이 어렵다기 보다는 Next.js에서 추가적으로 제공하는 기능들이 있어서 그렇게 느껴지는거 같다.

Next.js가 자신있게 소개하는 기능들은 다음과 같다.

1. Image Optimization
2. Zero Config
3. Hybrid: SSR and SSG
4. TypeScript Support
5. File-system Routing
6. API ROUTES
7. Code-splitting and Bundling

## File System based Routing

Next.js는 기본적으로 페이지 라우팅을 파일 시스템 기반으로 제공한다.
예를 들어 /pages/orders/thanks.ts 경로에 아래처럼 파일을 생성한다고 해보자.

![image](https://user-images.githubusercontent.com/63354527/170922166-785a4cd2-76cf-4aa4-90c8-c3b62013397b.png)

브라우저에 해당 파일의 위치경로를 주소창 도메인뒤에 suffix로 붙여주면 페이지가 렌더링된다.

Plain React로도 라이브러리를 사용해 Route기능을 활용할 수 있지만 기본적으로 제공하니 따로 설정할 필요도 없고 개인적으로 파일 경로대로 보여주는게 조금더 직관적인 느낌을 받았다. 물론 Next.js에서는 이방식대로 강제하기 때문에 쓰기 싫어도 써야한다는게 단점일 수 있다.

## API Routes

Next.js는 /pages/api 경로에 API핸들러를 생성하면 File system based routing을 통해 Serverless Function을 사용할 수 있다.

간단한 예로 /pages/api/hello.ts 파일을 생성하고 아래와 같이 API 핸들러를 작성해주면 https://domain-name.com/api/hello 로 통신했을 때 해당 json 정보를 받을 수 있다.

![image](https://user-images.githubusercontent.com/63354527/170922449-2818928d-4174-421b-8be9-29780889876d.png)

이 기능 덕분에 client, server 모두 모놀리틱한 구조로 프로그래밍이 가능한 점과 배포 역시 한번에 할 수 있기 때문에 신경써야 할 포인트가 크게 줄어든다. 이로써 좀더 프로그래밍에 집중할 수 있게 되니 생산성에 큰 이점을 얻는다.

모든 API를 완벽한 Production을 목적으로 사용하기에는 Serverless Function이 Stateless한 구조이기 때문에 DB 커넥션을 유지하기 어렵고, 측정해보지는 않았지만 Serverless 아키텍처의 고질적인 Cold Start문제도 있으니 개인적으로 간단한 프로토타입을 만들 때는 추천할 수 있을 것 같다.

또한 Next.js 자체에서 제공하는 미들웨어들은 굉장히 제한적이기 때문에 미들웨어를 만들어 핸들러를 커스텀하기 위해서는 약간의 우회책을 써야하는 점이 조금 아쉽다.

## Pre-rendering

Next.js는 기본적으로 빌드 타임에 HTML을 생성하여 모든 페이지에 대해 pre-render를 지원한다. 이는 퍼포먼스 향상과 SEO최적화에 도움을 준다.

Next.js의 경우 빌드 타임에 pages 폴더 안에 있는 코드를 토대로 HTML 파일을 미리 생성하고, 이를 바로 렌더링 해준다. 바로 정적 파일을 사용자에게 보여주니 유저 Latency가 짧을 수 밖에 없다.

## Static Generation

The HTML is generated at build-time and is reused for each request.

Static Geration은 위에서 언급한대로 간단하다. 빌드 타임에 HTML을 생성하고 해당 HTML을 재사용한다.

정적 생성을 위해서는 pages경로 안에서 getStaticProps 함수를 export하여 사용하면 된다.

![image](https://user-images.githubusercontent.com/63354527/170923243-f844c155-8e9a-4ec8-a16c-b2868e08ea44.png)

getStaticProps를 쓰면 CMS에서 데이터를 생성하거나 수정했을 때 반영이 안되는것 아니냐고 할 수 있는데 이것도 맞는 말이다. 하지만 그럴 경우 revalidate옵션을 주면 Incremental하게 정적 페이지를 재생성할 수 있다. 해당 옵션으로 10초를 준다면 요청 기준 매 10초마다 revalidation을 수행한다.

![image](https://user-images.githubusercontent.com/63354527/170923451-243f4073-9bf3-4619-a967-0a9d69f9dad6.png)

Static Generation은 거의 대부분의 경우에 사용하면 된다. 쇼핑몰로 따지면 소개 페이지나 도움말 그리고 상품 리스트도 revalidate값을 적당히 줘서 사용하면 SSR보다 훨씬 효율적이다.

## Server Side Rendering

The HTML is generated on each request

![image](https://user-images.githubusercontent.com/63354527/170923655-5350e9d3-4b19-4c1c-b563-52a15bb6cd2b.png)

서버사이드 렌더링은 per request로 서버와 통신한다. pre-rendering을 지원하기 때문에 SEO 최적화도 가능하긴 하지만 매 요청마다 서버와 직접 통신을 하다보니 서버에 부하를 줄 수 있고, 아울러 유저 Latency도 Static Generation방식보다는 오래 걸린다. 공식 문서를 보면 vercel team에서도 직접 Static Generation을 추천한다고 언급하고 있다. 하지만 중요한 데이터가 실시간으로 즉각 반영되야한다면 SSR옵션을 사용하는게 좋다.이를 테면 상품 상세페이지에서 수량 정보등 변동되는 중요한 정보들을 디스플레이하고 있다고 가정했을 때 실시간 반영을 위해 SSR을 사용할 수도 있을 것 같다. 하지만 기본적으로 Static Generation에 revalidate를 짧게 줘서 진행하는게 가장 좋긴 하다. SSR 기능을 사용하기 위해서는 getServerSideProps함수를 export 해주면 사용 가능하다.

![image](https://user-images.githubusercontent.com/63354527/170924172-472edbc4-9bb2-4048-9672-02884205f384.png)

일단 Static Generation을 사용하든 SSR을 사용하든 데이터를 pre-fetching 해 온다는 점이 클라이언트 개발을 하면서 편했던 것 같다. 개인적으로 이렇게 data fetching을 수행하면 코드가 깔끔해지는 것 같다.
