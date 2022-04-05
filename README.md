- 프로젝트를 진행하면서 정리한 가이드라인 (2022.04.05)
- TS / Next.js / Redux_toolkit / jest / storybook

# Next.js 가이드라인

## Dir Structure 프로젝트 디렉터리 구성

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
│   └─store(리덕스 관련 파일)
│      └─slice
└──
```

---

## Build, Deploy

- 로컬환경빌드

  1. npm run dev
  1. localhost:3000에서 확인

- 빌드
  - 배포하기전에 빌드 확인 할 것。
  - npm run build_[배포환경]
- 배포
  - Jenkins의 파이프라인 스크립트 실행으로 배포
  ```
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
                    dir('hange-app'){
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

# React Component Guideline

## TSX
  1. 아토믹 디자인 패턴을 변형하여 작성하였음。(Atoms/Molecules/Organisms/Templates/Pages)
  - 아토믹 디자인 패턴 관련 문서
    > [https://bradfrost.com/blog/post/extending-atomic-design/](https://bradfrost.com/blog/post/extending-atomic-design/) 
    > [https://uxdaystokyo.com/articles/glossary/atomic-design/](https://uxdaystokyo.com/articles/glossary/atomic-design/)
  1. API요청과 리덕스 상태 변경은 가능한 한, pages、layout 단위에서 관리
  1. 비즈니스 로직은 외부로 분리해서 관리하도록 한다.
  1. 외부 컴포넌트에서 받아오는 프로퍼티는 컴포넌트 내부에서 분할 대입으로 불러오지 않는다. (store에서 취득하는 상태와 혼동되지 않도록)
  1. React.FC 타입을 사용하지 않는다. (타입에서 제공하는 기능에서 프로퍼티가 취득이 안되는 문제가 있기 때문에)
  1. 컴포넌트는 함수형으로 작성하고 형태는 const fucnName = () => {} 로 작성합니다. 호이스팅 과정에서 TDZ의 효과를 가지도록 하기 위함 입니다.

(Approved)

```javascript
//(Approved)
import React, { ReactNode } from 'react'

interface BasicProps {
  id: string;
  password: string;
  age: number;
  action: HTMLElement;
  children: ReactNode;
}

const BasicComponent = ({ id, password, age, action, children }: BasicProps): JSX.Element => {
  //API, Redux here
  return (
    <div>
      <a>{id}</a>
      <p>{password}</p>
      <p>{age}</p>
      <button onClick={() => action}></button>
      {children}
    </div>
  )
}

export default BasicComponent
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

function BasicComponent(props: BasicProps) {
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

## \_app.tsx

- 루트가 되는 컴포넌트 입니다. 아래의 내용 이외에는 작성하지 마십시오.

```javascript
/* eslint-disable @typescript-eslint/naming-convention */
import { AppProps } from 'next/app'
// 공통 스타일의 임포트
import '../scss/_reset.scss'
import '../scss/_base.scss'

// redux 스토어의 임포트
import { Provider } from 'react-redux'
import store from '../store/createStore'

// components 공통 컴포넌트의 임포트
import DefaultLayout from '../components/layouts/DefaultLayout'
import CommonStateLoader from '../components/layouts/CommonStateLoader'

const HangeApp = ({ Component, pageProps }: AppProps): JSX.Element => {
  return (
    <Provider store={store}>
      <DefaultLayout>
        <Component {...pageProps} />
      </DefaultLayout>
      <CommonStateLoader />
    </Provider>
  )
}
export default HangeApp"
```

- \_app.tsx 를 구성하는 컴포넌트

  1. Provider

  - 리덕스 스토어에 액세스 하기 위한 컴포넌트 입니다.(수정 불필요)

  2.  DefaultLayout

  - 공통으로 사용하는 레이아웃 관련 정보들을 담고 있는 컴포넌트 입니다. 아래에서 자세한 내용을 다루고 있습니다.

  3.  Component

  - Next.js 프레임 워크에서 페이지를 표시하기 위해 기본값으로 제공되는 컴포넌트 입니다.(修正不要)

  4.  CommonStateLoader

  - 모든 페이지에서 최초 로드시 변경해야하는 상태들을 모아두는 컴포넌트 입니다.
  - 
---

## \_document.tsx

- <head>내부에 작성하고 싶은 내용들을 작성합니다. 
- <link/>와<script/>요소만 작성합니다.
- 예) 파비콘, 웹폰트, 외부 js소스 등

---
  
## DefaultLayout.tsx
