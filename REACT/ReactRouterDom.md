## React-Router-Dom ##
### 라우팅이란? ###
여러 사용자가 요청한 URL에 맞는 페이지를 보여주는 것으로 리액트에서는 React Router를 사용할 수 있음

<br />

### React Router를 사용하는 이유는? ###
리액트는 SPA 방식으로 동작하기 때문에 새로운 페이지를 로드하는 것이 아니라 하나의 페이지 안에서 필요한 데이터만 가져옴

\<a> 태그를 사용하면 페이지 전체가 새로 로딩되어 화면이 재렌더링 되고, 화면이 깜빡이기 때문에 사용자의 경험이 떨어질 문제가 있음

<br />

### React Router ###
여러 종류의 라우터 컴포넌트를 제공하며 대표적으로 HashRouter, BrowserRouter가 있음

<b>HashRouter</b>
- URL에서의 해시 값(#)을 감지함
- 검색 엔진이 읽지 못하고, History API를 사용하지 않기 때문에 동적 페이지에 불리함

<br />

<b>BrowserRouter</b>
- HTML5의 History API를 사용하여 URL을 관리함
- 사용자가 클릭하는 링크에 따라서 페이지를 전환할 수 있고, 새로고침 없이도 URL이 변경됨
- 기본적으로 서버는 / 경로에 대한 정보만 알고 있으며, 그 외의 경로는 처리할 수 없어 /about과 같은 경로로 새로고침을 시도하면 해당 경로에 대한 리소스를 찾지 못해서 404 에러를 발생시킴
- 이를 해결하기 위해 서버는 모든 요청을 /경로로 리다이렉션 하도록 설정할 수 있음

<br /> 

- 다시 말해, 클라이언트에서 /about 경로로 이동하고 새로고침 하면 서버에 요청이 전송되고, 서버는 해당 경로에 대한 리소스를 찾지 못해서 404에러가 발생함
- 이를 방지하기 위해, 서버에서 / 경로로 리다이렉션 하거나, index.html로 포워딩할 수 있음
- 그런 다음, React Router가 로드된 index.html 파일 내에서 클라이언트 측 라우팅을 처리하여 리액트 애플리케이션이 /about 경로에 맞는 컴포넌트를 렌더링하게 됨

=> 즉, 서버는 요청을 index.html로 포워딩하지만, 그 후의 라우팅은 클라이언트 측에서 React Router가 관리하여 사용자는 새로고침 하더라도 /about에 머무를 수 있고, 404 에러도 발생하지 않음

<br />
<br />

<b>설치</b>
```
npm install react-router-dom
```

<br />

#### BrowserRouter ####
- React Router의 최상위 컴포넌트로 History API를 사용해서 URL을 관리함
  - History API는 내부적으로 stack을 통해 사용자가 방문한 URL을 저장함
- BrowserRouter를 사용하면 SPA에서 URL을 변경하고, 해당 URL에 맞는 컴포넌트를 렌더링할 수 있음
- 라우팅을 진행할 컴포넌트 상위에 BrowserRouter 컴포넌트를 생성해서 감싸줘야 함

#### Routes ####
- Routes는 여러 개의 Route를 포함할 수 있는 컴포넌트로, 현재 URL과 일치하는 유일한 Route를 찾아 해당 컴포넌트로 렌더링 함

#### Route ####
- Route는 특정 경로에 대해 어떤 컴포넌트를 렌더리 할 지를 정의함
- path 속성으로 경로를 지정하고, element 속성으로 렌더링 할 컴포넌트를 지정함

#### Link ####
- Link 컴포넌트는 라우터 내에서 직접 페이지 이동을 하고자 할 때 사용됨
- \<a> 태그 대신 사용 가능하며, History API를 통해 브라우저 주소의 경로만 바꾸는 기능이 내장되어 있음

#### useNavigate ####
- useNavigate 훅을 사용하면 특정 이벤트 (onChange, onClick...)가 발생했을 때 페이지 이동을 트리거할 수 있음
```
import { useNavigate } from 'react-router-dom';
import Button from '@mui/material/Button';

const Nav = () => {
	const navigate = useNaviagte();
	
	const goToSignUp = () => {
		navigate('/sign-up');
	}
	
	const goToSignIn = () => {
		navigate('/sign-in');
	}
	
	return (
		<div>
			<Button 
				variant="Contained"
				onClick={goToSignUp}
			>
				회원가입
			</Button>
			<Button 
				variant="Contained"
				onClick={goToSignIn}
			>
				로그인
			</Button>
		</div>
	);
};

export default Nav;
```

<br />
<br />

<b>React Router 예시 코드</b>

App.js
```
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';
import Home from './Home';
import About from './About';
import Contract from './Contract';

const App = () => {
	return (
		<div className="App">
			<BrowserRouter>
				<nav>
					<ul>
						<li>
							<Link to='/'>홈</Link>
						</li>
						<li>
							<Link to='/about'>소개</Link>
						</li>
						<li>
							<Link to='contract'>연락</Link>
						</li>
					</ul>
				</nav>
				
				<Routes>
					<Route path='/' element={<Home />} />
					<Route path='/about' element={<About />} />
					<Route path='/contract' element={<Contract />} />
				</Routes>
			</BrowserRouter>
		</div>
	);
}

export default App;
```
Home.js
```
const Home = () => {
	return (
		<div>
			<h2>메인 페이지</h2>
		</div>
	);
}

export default Home;
```
