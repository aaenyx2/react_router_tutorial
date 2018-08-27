## 1. 프로젝트 구성 및 라이브러리 설치

### 1. create-react-app react-router-tutorial

### 2. 해당 폴더로 가서 리액트 라우터 설치

        $ yarn add react-router-dom
        $ yarn add cross-env --dev

react-router-dom: 브라우저에서 사용되는 리액트 라우터
cross-env: 프로젝트에서 NODE_PATH 를 사용하여 절대경로로 파일을 불러오기 위하여 환경 변수를 설정 할 때 운영체제마다 방식이 다르므로 공통적인 방법으로 설정 할 수 있게 해주는 라이브러리.

### 3. 디렉토리 생성

src/components: 컴포넌트들이 위치하는 디렉토리.
src/pages: 각 라우트들이 위치하는 디렉토리.
src/client: 브라우저 측에서 사용할 최상위 컴포넌트. 우리가 추후 서버사이드 렌더링을 구현 할 것이기 때문에 디렉토리를 따로 구분하였습니다. (서버사이드 렌더링을 할 때에는 서버 전용 라우터를 써야합니다.) 여기서 라우터를 설정합니다.
src/server: 서버측에서 사용 할 리액트 관련 코드를 여기에 넣습니다.
src/shared: 서버와 클라이언트에서 공용으로 사용되는 컴포넌트 App.js 가 여기에 위치합니다.
src/lib: 나중에 웹 연동을 구현 할 때 사용 할 API와 코드스플리팅 할 때 필요한 코드가 여기에 위치합니다.

### 4. NODE_ENV 설정

우리가 코드들을 불러올 때 ‘../components/Something’ 이런식으로 불러와야 하는 코드를 ‘components/Something’ 이렇게 불러 올 수 있도록 프로젝트의 루트경로를 설정하겠습니다. package.json 파일의 script 부분을 다음과 같이 수정하세요.


                "scripts": {
                        "start": "cross-env NODE_PATH=src react-scripts start",
                        "build": "cross-env NODE_PATH=src react-scripts build",
                        "test": "react-scripts test --env=jsdom",
                        "eject": "react-scripts eject"
                }


### 5. 컴포넌트 준비하기

* src/shared/App.js

App 컴포넌트를 만들어주세요. 어떤 주소로왔을때 무엇을 보여줄 지, 나중에 여기서 정의를 하도록 하겠습니다.


                import React, { Component } from 'react';

                class App extends Component {
                        render() {
                        return (
                                <div>
                                Hello React-Router
                                </div>
                        );
                        }
                }

                export default App;


Root 컴포넌트를 만드세요. 이 컴포넌트는 우리의 웹어플리케이션에 BrowserRouter를 적용합니다. 나중에 리덕스를 적용 하게 될 때, 여기서 Provider 를 통하여 프로젝트에 리덕스를 연결시켜줍니다.

* src/client/Root.js

                import React from 'react';
                import { BrowserRouter } from 'react-router-dom';
                import App from 'shared/App';

                const Root = () => (
                <BrowserRouter>
                        <App/>
                </BrowserRouter>
                );

                export default Root;


이제 우리가 만든 파일에 맞춰서 index.js 를 수정하세요.
* src/index.js


                import React from 'react';
                import ReactDOM from 'react-dom';
                import Root from './client/Root';
                import registerServiceWorker from './registerServiceWorker';
                import './index.css';

                ReactDOM.render(<Root />, document.getElementById('root'));
                registerServiceWorker();


## 2. Route와 파라미터

### 1. 기본 라우트 준비

자 그럼 우리의 첫 라우트 Home을 만들어보겠습니다.
이 라우터는 주소에 아무 path 도 주어지지 않았을 때 기본적으로 보여주는 라우트입니다.

* src/pages/Home.js

                import React from 'react';

                const Home = () => {
                return (
                        <div>
                        <h2>
                                홈
                        </h2>
                        </div>
                );
                };

                export default Home;


그리고 같은 형식으로 About 도 만드세요. 이 페이지는 /about 경로로 들어왔을 때 보여줄 페이지입니다.

* src/pages/About.js


                import React from 'react';

                const About = () => {
                return (
                        <div>
                        <h2>About</h2>
                        </div>
                );
                };

                export default About;


이제 이 컴포넌트를 불러와서 한 파일로 내보낼 수 있도록 인덱스를 만들어주세요.

* src/pages/About.js 


                export { default as Home } from './Home';
                export { default as About } from './About';

### 2. 라우트 설정하기

* src/shared/App.js


                        import React, { Component } from 'react';
                        import { Route } from 'react-router-dom';
                        import { Home, About } from 'pages';


                        class App extends Component {
                        render() {
                                return (
                                <div>
                                        <Route exact path="/" component={Home}/>
                                        <Route path="/about" component={About}/>
                                </div>
                                );
                        }
                        }

                        export default App;


라우트를 설정 할 때에는 Route 컴포넌트를 사용하고, 경로는 path 값으로 설정합니다.

첫번째 라우트 / 의 경우에는 Home 컴포넌트를 보여주게 했고, 두번째 라우트 /about 에서는 About 컴포넌트를 보여주게 했습니다.

첫번째 라우트의 경우엔 exact 가 붙어있지요? 이게 붙어있으면 주어진 경로와 정확히 맞아 떨어져야만 설정한 컴포넌트를 보여줌

historyApiFallback 설정을 통하여 어떤 요청으로 들어오든지 index.html 을 보여주도록 설정되어있다.


### 3. 라우트 파라미터 읽기

* history : push, replace 등의 메소드를 이용해 다른 경로로 이동하거나 앞 뒤 페이지로 전환 가능
* location : 현재 경로에 대한 정보와 URL query 정보를 가짐
* match : 이 객체가 어떤 라우트에 매칭되어 있는지, 그리고 params 정보를 가짐

먼저 params 부터 사용해보자.

App 파일에서 params 정보를 가지는 라우트를 설정하고 해당 컴포넌트에 match.params.name을 출력해보자.

* src/pages/About.js

                import React from 'react';

                const About = ({match}) => {
                return (
                        <div>
                        <h2>About {match.params.name}</h2>
                        </div>
                );
                };

                export default About;


query 사용해보자.

                yarn add query-string


그리고 query를 사용할 컴포넌트에서 query 정보를 담은 location을 비동기 할당받고 이를 콘솔 창에 출력해보자.


                import React from 'react';
                import queryString from 'query-string';

                const About = ({location, match}) => {
                const query = queryString.parse(location.search);
                console.log(query);

                return (
                        <div>
                        <h2>About {match.params.name}</h2>
                        </div>
                );
                };

                export default About;


http://localhost:3000/about/name?detail=true 링크로 접속해보면 'detail' query 값이 true로 설정되었다. 이 때 boolean이 아니라 문자열이라는 사실을 주의.

이 query 값을 따라 컴포넌트에 조건부 렌더링을 해보자.

                        import React from 'react';
                        import queryString from 'query-string';

                        const About = ({location, match}) => {
                        const query = queryString.parse(location.search);
                        console.log(query);
                        const detail = query.detail = "true"
                        return (
                                <div>
                                <h2>About {match.params.name} </h2>
                                {detail && 'detail: true'}
                                </div>
                        );
                        };

                        export default About;


라우트 이동을 위한 목차 컴포넌트 Menu.js를 생성.
라우터 간 이동은 <a> 가 아니라 <Link to>를 사용해야 한다. 따라서 react-router-dom에서 Link를 import해오자.

                import React from 'react';
                import { Link } from 'react-router-dom';

                const Menu = () => {
                return (
                        <div>
                        <ul>
                                <li><Link to="/">Home</Link></li>
                                <li><Link to="/about">About</Link></li>
                                <li><Link to="/about/foo">About Foo</Link></li>
                        </ul>
                        <hr/>
                        </div>
                );
                };

                export default Menu;


### 4. 라우트 안의 라우트

Post 라는 페이지 컴포넌트를 만들자. 이 컴포넌트에서는 params.id 를 받아와서 렌더링한다.


* src/pages/Post.js


                        import React from 'react';

                        const Post = ({match}) => {
                        return (
                                <div>
                                포스트 {match.params.id}
                                </div>
                        );
                        };

                        export default Post;


* src/pages/Posts


                        import React from 'react';
                        import { Link, Route } from 'react-router-dom';
                        import { Post } from 'pages'; 

                        const Posts = ({match}) => {
                        return (
                                <div>
                                <h2>Post List</h2> 
                                <ul>
                                        <li><Link to={`${match.url}/1`}>Post #1</Link></li>
                                        <li><Link to={`${match.url}/2`}>Post #2</Link></li>
                                        <li><Link to={`${match.url}/3`}>Post #3</Link></li>
                                        <li><Link to={`${match.url}/4`}>Post #4</Link></li>
                                </ul>
                                <Route exact path={match.url} render={()=>(<h3>Please select any post</h3>)}/>
                                <Route path={`${match.url}/:id`} component={Post}/>
                                </div>
                        );
                        };

                        export default Posts;


page index에서 posts와 post의 export 설정을 마친다. 이후 Menu에 Posts li를 만들고, Posts의 Link to 에 {match.url}이 사용되었는데, 이 url 은 현재의 라우트의 경로를 알려줌.

즉 ”/posts/1″ 랑 여기서는 같다.


* Menu의 Link to에 active style이나 별도의 class style을 적용하려면 <NavLink to="..." activeStyle={...} ..></Navlink>를 사용한다.




# 2. 코드 스플리팅

## 1. 코드 스플리팅의 기본

### 1. webpack 설정을 위해 파일들을 eject 하자

                $ yarn eject

### 2. config/webpack.config..../js 들어가서 entry를 배열[] 형태에서 객체{} 형태로 바꿔주자.

                entry: [
                require.resolve('react-dev-utils/webpackHotDevClient'),
                require.resolve('./polyfills'),
                require.resolve('react-error-overlay'),
                paths.appIndexJs,
                ]


이 부분을 아래와 같이 수정



                entry: {
                dev: 'react-error-overlay',
                vendor: [
                require.resolve('./polyfills'),
                'react',
                'react-dom',
                'react-router-dom'
                ],
                app: ['react-dev-utils/webpackHotDevClient', paths.appIndexJs]
                }


개발에만 필요한 react-error-overlay 는 dev 라는 이름으로 저장하게했고, 전역적으로 사용되는 라이브러리는 vendor 에 넣었으며, (polyfill 은 구형 브라우저에서도 Promise 등의 ES6 전용 코드가 제대로 작동하게하는 라이브러리임) app 부분엔 webpackHotDevClient 와 appIndexJs 를 넣었음

애초에 개발서버를 위한 웹팩 설정에서는 코드 스플리팅을 할 필요가 없지만 이 과정을 배워보면서 어떻게 작동하는지 알아보기 위함.

코드스플리팅을 이해하고 난 다음에는, 프로덕션에서만 코드스플리팅을 하도록 설정할 것임.


### 3. entry 를 수정했면, output 부분의 filename 과 chunkFilename도 다음과 같이 변경


                filename: 'static/js/[name].[hash].js',
                chunkFilename: 'static/js/[name].[chunkhash].chunk.js'


filename 부분에는 [hash] 값을 주었는데요, 이 해쉬 값은 앱이 빌드 될 때마다 새로운 값이 생겨납니다. 그 하단의 chunkFilename 은 [chunkhash]가 들어가있습니다. 웹팩쪽에서 미리 분리시킨 파일말고, 우리가 코드를 통해 직접 분리시킬 파일들은 chunkFile 이라고 칭합니다. 이 부분은 잠시 후 알아볼것이구요, 이렇게 파일이름에 해쉬값을 포함시켜주면 각 파일마다 고유의 이름을 가질 수 있게 되고, 코드가 업데이트 되었을때 기존 캐시를 사용하지 않고 최신 파일을 불러와서 사용하도록 할 수 있습니다.

설정을 다 하셨다면, 웹팩 개발서버를 끄고 다시 실행하세요.