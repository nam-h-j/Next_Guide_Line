- 프로젝트를 진행하면서 정리한 Next.js 가이드라인
- TS / Next.js / Redux_toolkit / axios / jest / storybook
- (2022.04.05 초안작성)

# contents

- [Directory Structure](#directory-structure-프로젝트-디렉터리-구성)
- [Build, Depoly](#build-deploy)
- [React Component Guideline](#react-component-guideline)
  - [TSX](#tsx)
  - [\_app.tsx](#apptsx)
  - [\_document.tsx](#documenttsx)
  - [DefaultLayout.tsx](#defaultlayouttsx)
- [Style Sheet](#style-sheet)
  - [Module CSS](#module-css)
  - [css-in-js](#css-in-js)
- [Type](#type)
  - [초기화](#초기화)
- [Redux Guideline](#redux-guideline)

# Next.js 가이드라인

## Directory Structure 프로젝트 디렉터리 구성

```bash
Project
│
├── pubilc(프로젝트에서만 사용하는 미디어 파일)
├── src(모든 내용은 여기에 작성)
│   ├─api(API요청 정의)
│   ├─component(컴포넌트, 모듈 scss)
│   │  ├─Atoms -- button, form, nav ...
│   │  ├─Layouts -- layout
│   │  └─Modules -- header, footer, listItem ...
│   ├─pages(_app.tsx, _document.tsx, 404페이지, 점검페이지, 앱의 각 페이지)
│   ├─scss(공통 scss(mixin)등을 정의)
│   ├─store(리덕스 관련 파일)
│   │  └─slice
│   └─types(상태나 api요청에서 가져오는 데이터의 타입을 정의)
└──
```

---

## Build, Deploy

- 로컬환경빌드

  1. npm run dev
  1. localhost:3000에서 확인

- 빌드
  - 배포하기전에 빌드 확인 할 것。
  - npm run build\_[배포환경]
- 배포
  - Jenkins의 파이프라인 스크립트 실행으로 배포

```bash
  //파이프라인 스크립트
  pipeline {
    agent any
    tools {
        nodejs "v14.0.0" //프로젝트의 노드 버전을 입력
        git "Default"
    }
    stages {
        stage('prepare') {
            steps {
                echo 'prepare'
                 git branch: "${BRANCH_NAME}", credentialsId:'${CREDENTIAL_ID}', url: '${GIT_REPOSITORY_URL}'
                 sh  'ls -al'
            }
        }
        stage('build') {
            steps {
                    dir('myapp'){
                        sh 'ls -al'
                        sh "npm install"
                        sh "${BUILD_SCIRIPT}"
                }
            }
        }
        stage('deploy') {
            steps{
                    dir('myapp'){
                        sh 'ls -al'
                        sh "/usr/local/aws-cli/v2/2.2.8/bin/aws s3 sync /var/lib/jenkins/workspace/${JOB_PATH} s3://${S3_BUCKET_NAME} --delete --profile default"
                        echo 'deploy done.'
                }
            }
        }
    }
}
```

- BRANCH_NAME : 배포할 브랜치 명(젠킨스에 커넥션 설정된 깃을 기준으로 함)
- CREDENTIAL_ID : 젠킨스에 설정한 깃 권한 아이디를 입력
- GIT_REPOSITORY_URL : 레포지토리의 url(@git:으로 시작하는)
- BUILD_SCIRIPT : 프로젝트 빌드 스크립트
- JOB_PATH : 배포 파일위치
- S3_BUCKET_NAME : 배포 할 s3 버킷 이름

---

## eslint

## React Component Guideline

### TSX

1. 아토믹 디자인 패턴을 변형하여 작성하였음。(Atoms/Molecules/Organisms/Templates/Pages)

- 아토믹 디자인 패턴 관련 문서
  > [https://bradfrost.com/blog/post/extending-atomic-design/](https://bradfrost.com/blog/post/extending-atomic-design/) > [https://uxdaystokyo.com/articles/glossary/atomic-design/](https://uxdaystokyo.com/articles/glossary/atomic-design/)

1. API요청과 리덕스 상태 변경은 가능한 한, pages、layout 단위에서 관리
1. 비즈니스 로직은 외부로 분리해서 관리하도록 한다.
1. 외부 컴포넌트에서 받아오는 프로퍼티는 컴포넌트 내부에서 분할 대입으로 불러오지 않는다. (store에서 취득하는 상태와 혼동되지 않도록)
1. React.FC 타입을 사용하지 않는다. (타입에서 제공하는 기능에서 프로퍼티가 취득이 안되는 문제가 있기 때문에)
1. 컴포넌트는 함수형으로 작성하고 화살표 함수로 작성한다. 호이스팅 과정에서 TDZ의 효과를 가지도록 하기 위함.
1. 컴포넌트 명은 파스칼케이스(pascal case)로 작성하고 지나친 줄임 표현은 쓰지 않는다.(예 : NotiLiItem(x) NoticeListItem(o))

(Approved)

```javascript
//(Approved)
import React, { ReactNode } from "react";

interface BasicProps {
  id: string;
  password: string;
  age: number;
  action: HTMLElement;
  children: ReactNode;
}

const BasicComponent = ({
  id,
  password,
  age,
  action,
  children,
}: BasicProps): JSX.Element => {
  //API, Redux here
  return (
    <div>
      <a>{id}</a>
      <p>{password}</p>
      <p>{age}</p>
      <button onClick={() => action}></button>
      {children}
    </div>
  );
};

export default BasicComponent;
```

(Not-Approved)

```javascript
//(Not-Approved)
import React, { ReactNode } from "react";

interface BasicProps {
  id: string;
  password: string;
  age: number;
  action: HTMLElement;
  children: ReactNode;
}

function BasicComponent(props: BasicProps) {// Arrow Function 사용
  const { id, password, age, action, children } = props//분할대입 사용하지 않음
  return (
    <div>
      <a>{id}</a>
      <p>{password}</p>
      <p>{age}</p>
      <button onClick={() => action}></button>
      {children}
    </div>
  );
}

export default BasicComponent;

//(Not-Approved2)
import React, { ReactNode } from "react";

interface BasicProps {
  id: string;
  password: string;
  age: number;
  action: HTMLElement;
  children: ReactNode;
}

const BasicComponent:React.FC<Props> = ({props}) => {
  const { id, password, age, action, children } = props
  return (
    <div>
      <a>{id}</a>
      <p>{password}</p>
      <p>{age}</p>
      <button onClick={() => action}></button>
      {children}
    </div>
  );
}
export default BasicComponent;

////(Not-Approved)
import React, { ReactNode } from "react";

interface BasicProps {
  id: string;
  password: string;
  age: number;
  action: HTMLElement;
  children: ReactNode;
}

function BasicComponent({ id, password, age, action, children }: BasicProps) {
  return (
    <div>
      <a>{id}</a>
      <p>{password}</p>
      <p>{age}</p>
      <button onClick={() => action}></button>
      {children}
    </div>
  );
}

export default BasicComponent;
```

---

### \_app.tsx

- 루트가 되는 컴포넌트 입니다. 아래의 내용 이외에는 작성하지 않는다.

```javascript
/* eslint-disable @typescript-eslint/naming-convention */
import { AppProps } from "next/app";
// 공통 스타일의 임포트
import "../scss/_reset.scss";
import "../scss/_base.scss";

// redux 스토어의 임포트
import { Provider } from "react-redux";
import store from "../store/createStore";

// components 공통 컴포넌트의 임포트
import DefaultLayout from "../components/layouts/DefaultLayout";
import CommonStateLoader from "../components/layouts/CommonStateLoader";

const MyApp = ({ Component, pageProps }: AppProps): JSX.Element => {
  return (
    <Provider store={store}>
      <DefaultLayout>
        <Component {...pageProps} />
      </DefaultLayout>
      <CommonStateLoader />
    </Provider>
  );
};
export default MyApp;
```

- \_app.tsx 를 구성하는 컴포넌트

  1. Provider

  - 리덕스 스토어에 액세스 하기 위한 컴포넌트.(수정불필요)

  2.  DefaultLayout

  - 공통으로 사용하는 레이아웃 관련 정보들을 담고 있는 컴포넌트. 아래의 상세 내용 참고.

  3.  Component

  - Next.js 프레임 워크에서 페이지를 표시하기 위해 기본값으로 제공되는 컴포넌트.(수정불필요)

  4.  CommonStateLoader

  - 모든 페이지에서 최초 로드시 변경해야하는 상태 작업들을 작성하는 컴포넌트.

---

### \_document.tsx

- <head>내부에 작성하고 싶은 내용들을 작성.
- <link/>와<script/>요소만 작성.
- 예) 파비콘, 웹폰트, 외부 js소스 등

---

### DefaultLayout.tsx

- 가장 상위 컴포넌트로, 모든 페이지에서 공통적으로 사용하는 상태, 로직, 컴포넌트를 불러온다.
- 공통헤더, 푸터, 모달영역 등을 불러온다.
- 공통으로 사용하는 상태값을 불러온다.
- <head>의 meta, title 정보를 작성한다.

---

## Style Sheet

- SCSS를 기본으로 사용
- Module css, css-in-js 두 가지 형태로 작성한다.
- Component마다 하나의 StyleSheet 파일을 가진다.
- 파일명은 Component명 + .module.scss로 명명한다.(예 : 컴포넌트 = Example.tsx, SCSS = Example.module.scss)
- reset, mixin등의 글로벌 scss는 \_app.tsx에서만 불러온다.

### Module CSS

- Atomic 패턴이 적용 된 컴포넌트 둥 Layout, Module에는 레이아웃 관련 속성 위주로 작성한다.
  - 예) display, position, top, left, flex-direction, justify-content, background...
- Atom에는 색상, 디자인 관련 속성 위주로 작성한다.
  - 예) font-size, font-weigth, color...

### css-in-js

- 상태에 따른 스타일 변화가 공통적으로 적용되어야 하거나, 모듈로 제어가 어려운 경우 부분적으로 사용한다.
- 외부 라이브러리 형태로 별도 파일(.tsx)에서 관리하고 컴포넌트에서는 상황에 맞게 Import해서 사용한다.
- 최대한 Module CSS로 처리해서 되도록 사용하지 않는 방향으로 작성한다.

```js
// css-in-js_example

// Define
export const HtmlPageHeight = (heightProp: string): JSX.Element => (
  <style global jsx>{`
    .indexPage {
      width: 100%;
      height: ${heightProp};
    }
    .indexPage > div {
      height: ${heightProp};
    }
  `}</style>
);
export const HtmlScrollHidden = (overflowProp: string): JSX.Element => (
  <style global jsx>{`
    body {
      overflow-y: ${overflowProp};
    }
  `}</style>
);

// how to use
import { HtmlPageHeight } from "../externalCSS";

const SomeComponentNeedsCssToJS = (): JSX.Element => {
  return{
    <>
    {HtmlPageHeight('prop')}
    </>
  }
};
```

---

## Type

- 데이터 관련한 타입을 별도로 관리한다.
- 요청하는 API데이터 이름뒤에 Type를 붙여서 명명한다.
- 타입 이름은 파스칼케이스, 속성은 카멜케이스를 사용한다.
- 상태에 관한 데이터의 경우 아래의 명명규착을 사용한다
  - Boolean(행위나 상태) : is + 데이터명(명사나 동사)
  - Boolean(데이터의 유무) : has + 데이터명(명사)

### 초기화

- 각 타입에 대한 초기화된 상태도 해당 파일 내에서 작성한다.
- init + 데이터명 으로 명명한다
- 카멜케이스로 작성한다.

```js
//타입 정의
export type UserType = {
  userIndex: string
  userName: string
  gender: string
  age: number
}

//데이터 초기화(리덕스 상태)
const initUser: UserType = {
  userIndex: "USER001"
  userName: "Bruce"
  gender: "Male"
  age: 30
};

export type UserStateType = {
  userInfo : UserType
  isLogin : boolean
  isDarkMode : boolean
}

const initUserState: UserStateType = {
  userInfo: initUser
  isLogin: false
  isDarkMode: false
};

```

## Redux Guideline

- Redux toolkit을 사용한다
- 페이지 별로 Slice를 나눠서 작성한다.
- 각 슬라이스는 리듀서, 액션 순으로 작성한다.
- 리덕스에 다른 비즈니스 로직은 넣지 않는다. 불가피할 경우 별도 함수로 작성하여 호출한다.

### Slice - 리듀서

- Slice명은 데이터명 + Slice로 하고, 파스칼 케이스로 작성
- Reducer 명은 데이터명 + Reducer로 하고, 카멜 케이스로 작성
- Reducer에 가장 처음은 resetStateReducer를 작성한다.
- Reducer는 아래의 코드 이외에 작성하지 않는다.

```js
import { initUserState } from "../types";

export const UserSlice = createSlice({
  name: "User",
  initialState: initUserState,
  reducers: {
    resetUserStateReducer: () => initUserState,

    userInfoReducer: (state, action) => {
      state.userInfo = action.payload;
    },
    isLoginReducer: (state, action) => {
      state.isLogin = action.payload;
    },
    isDarkModeReducer: (state, action) => {
      state.isDarkMode = action.payload;
    },
  },
});
```

## Api
