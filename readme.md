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

첫번째 라우트의 경우엔 exact 가 붙어있지요? 이게 붙어있으면 주어진 경로와 정확히 맞아 떨어져야만 설정한 컴포넌트를 보여줍니다.