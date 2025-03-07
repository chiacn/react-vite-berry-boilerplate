 ### Notice
 
 yarn berry
	pnp.cjs - 의존성의 실제 파일 경로 매핑
	.yarn/cache - 모든 패키지 파일은 여기 압축되어 저장됨.

(yarn 설치 안 되어 있으면 npm install -g yarn으로 설치)

------------------------------------------------------------

### 설치방법


#### 1. 프로젝트 생성
create-vite - react-swc-ts 템플릿 기반 프로젝트 생성

#### 2. yarn berry 세팅 
	a. npm install -g yarn
	b. 프로젝트 루트로 이동해서 yarn berry 버전으로 세팅
		yarn set version berry
		(yarn -v 후 2 이상이면 Berry 버전으로 제대로 설정된 것)


#### 3. corepack 활성화 
> #### Corepack
>
> 모든 개발자가 같은 package manager 버전을 별도 설치과정 없이 사용하게 해줌.
> package manager 버전을 관리하는데 도움을 주는 도구로 node 와 패키지 매니저 사이의 브릿지 역할
> package.json 내부 `packageManager` 필드를 참조

	corepack enable 명령어 입력한다.
	(확인 - corepack --version )
	


#### 4. pnp 관련 설정

> #### pnp 모드
>
> node_modules 대신 .yarn/cache 디렉토리에 .zip 파일로 패키지를 설치하는 모드
> (php.cjs에 의존성의 실제파일 경로 매핑 , 모든 패키지 파일은 .yarn/cache에 압축되어 저장됨 )

	- .yarnrc.yml 파일 생성 및 설정
	(pnp 옵션 및 zero-install (압축된 파일 사용) 사용을 위함)

	```
	enableGlobalCache: false
	nodeLinker: pnp
	```

### enableGlobalCache?
>
> Yarn Berry가 지원하는 zero-install은 디펜던시들이 repository에 커밋될 수 있는 것을 의미한다.
> 이를 위해 enableGlobalCache 옵션을 false로 처리해주어야 한다.
>
> - enableGlobalCache: true - cache를 globalFolder에 저장
> - enableGlobalCache: false - cache를 cacheFolder에 저장
> => git으로 관리되어야하므로 글로벌캐시로 저장되면 안 된다.

<참고>
https://marketsplash.com/how-to-use-yarn-berry-yarn-2/
https://velog.io/@minboykim/Yarn-berry%EB%A1%9C-%EB%AA%A8%EB%85%B8%EB%A0%88%ED%8F%AC-%EA%B5%AC%EC%84%B1%ED%95%98%EA%B8%B0


#### 5. 의존성 설치
여기까지 마쳤으면 `yarn or yarn install`을 통해 의존성 설치

- pnp, zero-install 정상 작동 확인.

> .pnp.cjs / .pnp.loader.mjs 파일 생성 화인 (pnp)
> .yarn/cache 하위 의존성 .zip 설치 확인 (zero-install)


#### 6. vite dev 서버 동작 시 node_modules/.vite 캐시 생성 방지를 위한 추가 설정
- vite를 사용할 경우 기본적으로 node_modules/.vite 폴더에 캐시를 생성하므로 이걸 막아줌.

`
vite.config.ts 파일

cacheDir: './.vite' // 설정 추가

.gitignore 파일 

#vite cache // vite 캐시 파일을 저장하는 폴더가 변경되었으므로 헤딩 디렉토리 git에서 제외.
.vite // ignore 추가
	
`


#### 7. vscode IDE ts 설정 추가
> Yarn Berry에서는 기존의 node_module이 아닌 pnp.cjs로 의존성을 관리하기 때문에, vscode와 같은 IDE에서
> TypeScript, ESLint, Prettier 등의 도구를 올바르게 인식할 수 있도록 추가 설정 필요.


	a. `zipfs` vscode extension 추가
		- yarn에서는 의존성들이 zip로 압축되어 있기 때문에 vscode에서 이를 참조하기 위한 ZipFS가 필요
	b. `yarn dlx @yarnpkg/sdks vscode` 실행
		- Yarn berry에서 제공하는 SDK를 사용하여 IDE와 통합. (.yarn/sdks 디렉토리에 설정파일 생성)
		
	c. .vscode/settings.json 설정
	
```
.vscode/extentions.json
// recommendation - 해당 프로젝트를 열 때 설치를 추천.
{
  "recommendations": [
    "arcanis.vscode-zipfs",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode"
  ]
}



.vscode/settings.json
{
  // search.exclude -  yarn dlx @yarnpkg/sdks vscode에 의해 자동적으로 추가
  "search.exclude": {
    "**/.yarn": true,
    "**/.pnp.*": true
  },
  "eslint.nodePath": ".yarn/sdks",
  "prettier.prettierPath": ".yarn/sdks/prettier/index.js",
  "typescript.tsdk": ".yarn/sdks/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "editor.tabSize": 2,
  "prettier.printWidth": 120
}

```

<br/>

### 주의. prettier v3 이상 및 eslint-config-prettier, eslint-plugin-prettier는 vscode에서 무한로딩이 걸려서 다운그레이드 처리	

### cannnot find module 에러가 뜬다면
Ctrl + Shift + P -> Select TypeScript Version -> Use Workspace Version 선택 (Use VS Code's Version 아님)
