# **Vue-genesis**

## Micro Frontend란?
기존의 웹 프론트엔드는 같은 프레임워크 안에서 모두 한 번에 협업하는 형식인 모놀리식 방식으로 개발이 이루어졌다. 기술을 가지고 있는 인원들만이 유지보수 면에서 유리하다는 장점을 가진다. 그러나 동일한 문법과 사용법으로 하지 않으면 실행되지 않는다는 한계가 있다. 이는 다른 기술들을 함께 조화를 이루며 사용하기 어려웠다는 뜻이다.
 이와 같은 배경에서 나타난 Micro Frontend는 기존의 개발 방법을 더 작고 간편하게 해줌으로써 이러한 한계를 잘 보완해준다. Micro Frontend에서는 한 페이지를 구성하기 위한 각 구성들을 한 번에 구현해야 했던 기존과 달리, 프론트엔드의 단일 구조들을 완전히 독립적으로 개발, 테스트, 배포, 유지보수 및 관리를 할 수 있다. 그리고 최종적으로는 한 어플리케이션에 각 단일 구조들을 통합하여 응집력 있게 최종 결과물을 구현하게 된다.
## **Vue-genesis의 특징**
 Vue-genesis는 가장 필수적인 요소만 가지고 있는 렌더링 라이브러리이다. 따라서, 자신이 부가적으로 이용하고 싶은 기능이 있다면 그 부분만 추가적으로 설치하여 이용하면 되는 것이다. 즉, 사용하지 않는 기능을 배제하여 용량을 절약할 수 있다. 하지만 Vue-genesis의 장점이 역으로 단점이 될 수 있다. 필수적인 요소만 가지고 있다면, 개발에 필요한 요소들을 일일이 추가로 설치해야 한다. 규모가 커지다 보면 필요한 것들도 늘어나, 그 모든 것을 설치해야한다. 간단하게 만들 경우에 특히 강점이 극대화 되는 것이다. 이점을 살려 micro-frontend를 구현하고자 한다.

## **예제 구현**
원본 깃허브 주소 https://github.com/fmfe/genesis
<br> 

![image](https://user-images.githubusercontent.com/74655724/118503396-8bcd4e80-b765-11eb-8104-882c8d663807.png)

<br>

1.	모든 것은 layout의 app.vue를 바탕으로, home, about의 app.vue를 렌더링하여 하단에 띄우는 방식이다.
2.	렌더링 기능을 이용한 부분은 위의 그림과 같은 빨간 박스의 내용들이다.
3.	① Router link의 각각 ‘home’, ‘About’, ‘404’를 누르면 layout의 하단인 ②에 각각의 app의 화면으로 바뀌며, 주소도 각각 ‘localhost:3000/’, ‘localhost:3000/about’, ‘localhost:3000/404’로 이동한다. (단, ‘404’의 경우는 구현되어 있지 않아, layout앱 밑 화면에는 아무것도 뜨지 않는다)
4.	②에 표시되는 각 app은 서로 이동할 수 있는 하이퍼링크를 가진다. (home은 about으로, about은 home으로)
5.	③ Router Instansce Method는 이를 바탕으로 이전 페이지, 다음 페이지로 이동할 수 있다.

### **vue-genesis 기본 설치 요소**
* genesis-core               npm install @fmfe/genesis-core
* genesis-compiler           npm install @fmfe/genesis-compiler -D
* ts                          npm install ts-node typescript -g
* express(HTTP Service)       npm install express
### **해당 예제를 위한 추가 설치 요소**
* genesis-app                 npm install vue-router @fmfe/genesis-app
* genesis-remote             npm install @fmfe/genesis-remote
* square                     npm install @fmfe/square
* genesis-lint                npm i @fmfe/genesis-lint
* @types/jest                npm i @types/jest 
* next-redux-wrapper      npm install --save @types/node
* redux-devtools-extension npm i redux-devtools-extension


#### ssr-layout/src/app.vue

```html
<div class="border m-2 p-2">
    <div class="text-gray-500 text-sm">Router Instance Methods</div>
    <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded h-10" @click="$router.back()">Back</button>
    <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" @click="$router.forward()">Forward</button>
</div>

```
3에 해당하는 코드로, 2를 이전 페이지, 다음 페이지로 이동시킨다.

```html
<ul class="flex justify-center items-center p-2">
    <li v-for="(navItem, index) in nav" :key="index">
        <router-link :to="navItem.path" class="route-link">{{navItem.label}}</router-link>
    </li>
</ul>

```

```html
nav: [
    {
        path: "/",
        label: "Home"
    },
    {
        path: "/about",
        label: "About"
    },
    {
        path: "/404",
        label: "404"
    }
]
```
1에 해당하는 코드로, ‘localhost:3000/’, ‘localhost:3000/about’, ‘localhost:3000/404’로 연결되어 있다.

```html
<main>
    <remote-view :key="$route.meta.remoteUrl" :fetch="fetch" />
</main>

```
2 자리를 차지하는 코드이다.

```html
1.	        async fetch() {
2.	            const remoteUrl: string = this.$route.meta.remoteUrl;
3.	            const url = decodeURIComponent(this.$route.fullPath);
4.	            const res = await axios.get(
5.	                `http://localhost:3000${remoteUrl}?renderUrl=${url}`
6.	            );
7.	            if (res.status === 200) {
8.	                return res.data;
9.	            }
10.	            return null;
11.	        }


```
#### ssr-layout/src/routes.ts
```html
routes: [
    {
        path: '/',
        meta: {
            remoteUrl: '/api/home'
        },
        component: () => import('./app.vue')
    },
    {
        path: '/about',
        meta: {
        remoteUrl: '/api/about'
        },
        component: () => import('./app.vue')
    }
]	 
```
#### Node_modules/@fmfe/genesis-remote/sidt/esm/index.js
```html
1.	export const RemoteView = {
2.	    name: 'remote-view',
3.	    props: {
4.	        fetch: {
5.	            type: Function
6.	        },
7.	        clientFetch: {
8.	            type: Function
9.	        },
10.	        serverFetch: {
11.	            type: Function
12.	        }
13.	    },
14.	     .
15.	     .
16.	     .
17.	}

```
해당 코드를 바탕으로, ssr-layout/src/app.vue에 인스턴스를 생성하고, 해당 인스턴스는 ssr-layout/src/routes.ts를 바탕으로 2에 띄운다.
### 예제구현에서의 이슈

1)	구현된 화면에서 상, 하단 앱을 통튼 앱을 찾고자 하였으나 찾지 못함
<br>*→layout의 app 하단에 home, about의 app을 띄우는 형태였음*
2)	ssr-layout/src/app.vue에 파란색 하이라이트 부분이 이해가 가지 않음
3)	RemoteView에 대한 코드가 genesis-remote 폴더에 네 가지가 있었으나 어느 것을 기준으로 보아야할 지 알지 못함
4)	타입스크립트로 작성된 코드가 있어, 해석에 어려움을 겪음
<br>*→변환기를 활용하였으나 매끄럽지 못한 부분 다수 존재*
5)	아래 코드가 렌더링에 크게 관여되었다고 파악 중이나, 어떻게 연결된 것인지 아직 알아내지 못함.
#### __tests__/genesis-core/renderer.ts

```typescript
class Home extends SSR {
    public constructor() {
        super({
	        name: 'ssr-home',
            build: {
	                baseDir: path.resolve(__dirname, '../../examples/ssr-home')
	            }
	    });
	}
}
class About extends SSR {
	public constructor() {
	    super({
	        name: 'ssr-about',
	            build: {
	            baseDir: path.resolve(__dirname, '../../examples/ssr-about')
	        }
	    });
	}
}	 
const ssr = {
    home: new Home(),
    about: new About()
};

```
## Login/Setting 구현
* 구현목표
</br>

![image](https://user-images.githubusercontent.com/74655724/118511002-76a7ee00-b76c-11eb-87c4-f3beb2e5dde5.png)
</br>

* 홈화면
</br>

![image](https://user-images.githubusercontent.com/74655724/118511089-8c1d1800-b76c-11eb-9004-19c21c19496d.png)
</br>

* login setting화면
</br>

![image](https://user-images.githubusercontent.com/74655724/118511229-a9ea7d00-b76c-11eb-89b9-1035f341aa19.png)
</br>
login화면
</br>

* 실제구현화면
</br>

![image](https://user-images.githubusercontent.com/74655724/118529352-6436b000-b77e-11eb-8b04-acd162682395.png)
</br>

* setting화면
</br>

![image](https://user-images.githubusercontent.com/74655724/118529413-73b5f900-b77e-11eb-9985-cb3f1da78a54.png)
</br>

login화면

#### **Login/Setting 구현에 있어서의 이슈와 해결**

* ssr-about,ssr-home의 index,vue교체 아래와 같은 문제발생
</br>
```typescript
Error from chokidar (c:\):Error: EBUSY: resource busy or locked, lstat 'c:\DumpStack.log.tmp'   
```
![image](https://user-images.githubusercontent.com/74655724/118529544-9811d580-b77e-11eb-9317-8fc25545ccdf.png)
</br>

* npm install로 error from chokidar문제 해결
</br>
```typescript
found 1 low severity vulnerability 
run 'npm audit fix' to fix them, or 'npm audit' for details
```

![image](https://user-images.githubusercontent.com/74655724/118529608-aa8c0f00-b77e-11eb-9c18-e9fd8addff15.png)
</br>

* npm audit fix추가 설치
</br>
```typescript
 fixed 0 of 1 vulnerability in 2184 scanned packages
 1 vulnerability required manual review and could not be updated
```

![image](https://user-images.githubusercontent.com/74655724/118529673-bd064880-b77e-11eb-8fc0-878dd9651939.png)
</br>

* ssr-about\index.vue ssr-home\index.vue genesis-forms에서 genesis로 변경
</br>
```typescript
Module not found: Error: Can't resolve 'vue-genesis-form' in 'C:\workspace\genesis-master\examples\ssr-about\src\views'
resolve 'vue-genesis-forms' in 'C:\workspace\genesis-master\examples]ssr-about\src\views'
```
```typescript
<script>
import {AppForm} from "genesis";
```

![image](https://user-images.githubusercontent.com/74655724/118529767-d4ddcc80-b77e-11eb-847b-61fe0b2b5b66.png)
</br>

![image](https://user-images.githubusercontent.com/74655724/118529832-e6bf6f80-b77e-11eb-8af9-3d75724962e0.png)
</br>
import Vue from 'vue ';으로 변경
</br>
```typescript
export default{
    name: "login",
    components: {
        AppForm
    },
    props: {
        fields: {
            type: Array,
            default: () => []
        }
    }
},
```

![image](https://user-images.githubusercontent.com/74655724/118529908-0191e400-b77f-11eb-811b-5c3252cb3ae9.png)
</br>

에서 AppForm을 Vue로 변경

* genesis dev script failed문제발생
</br>
```typescript
npm ERR! code ELITECYCLE
npm ERR! errno1
npm ERR! genesis@0.0.1 dev: 'ts-node genesis.dev -p=tsconfig.json'
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the genesis@0.0.1dev script.
npm ERR! This is probably not a problem with npm. There si likely additional logging output above.
npm ERR! A complete log of this run can be found in:
npm ERR!  C:\Users\AppData\Roaming\npm-cache\_logs\2021-04-07T18_50_53_549Z-debug.log
```

![image](https://user-images.githubusercontent.com/74655724/118529972-153d4a80-b77f-11eb-8765-181f3de33d88.png)
</br>

위 문제가 발생하여 npm install
</br>
```typescript
found 1 low severity vulnerability
    run'npm audit fix' to fix them, or 'npm audit' for details
```

![image](https://user-images.githubusercontent.com/74655724/118530038-25552a00-b77f-11eb-9932-02bbe12608a2.png)
</br>

npm audit fix
</br>
```typescript
fixed 0 of 1 vulnerability in 2184 scanned packages
    1 vulnerability required manual review and could not be updated
```
![image](https://user-images.githubusercontent.com/74655724/118530080-330aaf80-b77f-11eb-98ed-01f5f59954b1.png)
</br>

npm fund
</br>
```typescript
-message@2.0.4, unist-util-is@4.1.0, unist-util-find-all-after@1.0.5,unist-util-remove-position@1.1.4, vfile-location@2.0.6, mdast-util-compact@1.0.4
+-- https://github.com/sponsors/unifiedjs
| '--micromark@2.11.4
 -- https://github.com/sponsors/RubenVerborgh
  '--follow-redirect@1.13.3
```
![image](https://user-images.githubusercontent.com/74655724/118530137-4584e900-b77f-11eb-8b79-ecd9f49a3ebe.png)
</br>

* pacakge.json 바꾸기
</br>
```typescript
npm ERR! code ELITECYCLE
npm ERR! errno1
npm ERR! genesis@1.0.0 dev: 'ts-node genesis.dev -p=tsconfig.json'
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the genesis@1.0.0dev script.
npm ERR! This is probably not a problem with npm. There si likely additional logging output above.
npm ERR! A complete log of this run can be found in:
npm ERR!  C:\Users\AppData\Roaming\npm-cache\_logs\2021-04-07T18_15_57_447Z-debug.log
```
![image](https://user-images.githubusercontent.com/74655724/118530180-52a1d800-b77f-11eb-99a2-bb538ec81ed1.png)
</br>

* npm install –g npm 실행
</br>
```typescript
added 252 packages from 909 contributors in 25.564s
```

![image](https://user-images.githubusercontent.com/74655724/118530220-5fbec700-b77f-11eb-8a40-ad1f24b36374.png)
</br>

* npm cache verify 로 cache변경
</br>
```typescript
Cache verified and compressed (~\AppData\Roaming\npm-cache\_cacahe):
Content verified: 4556 (207317129 bytes)
Content garbage-collected: 150 (30646944bytes)
Index entries: 7190
Finished in 26.605s
```
![image](https://user-images.githubusercontent.com/74655724/118530275-6cdbb600-b77f-11eb-92a1-f387c6bd290e.png)
</br>

* npm install 다시실행 husky문제발생
</br>
```typescript
Cannot read property 'toString' of null
husky > Failed to uninstall
```
```typescript
found 90 vulnerabilities (73 low, 9 moderate, 8 high)
 run 'npm audit fix' to fix them, or 'npm audit' for details
```

![image](https://user-images.githubusercontent.com/74655724/118530311-78c77800-b77f-11eb-9e48-4529cad93c32.png)
</br>

![image](https://user-images.githubusercontent.com/74655724/118530347-8250e000-b77f-11eb-9cab-3bd06d637261.png)
</br>

* npx husky-init 로 husky문제 해결
</br>
```typescript
husky-init updating package.json
 settins prepare script to command "husky install"
husky - not a Git repository, skipping hooks installation
can't create hook, .husky directory doesn't exist (try running husku install)
```

![image](https://user-images.githubusercontent.com/74655724/118530405-94cb1980-b77f-11eb-9641-3bf36d6388cd.png)
</br>

* npm audit fix실행 최종적으로 webpack문제 발생
</br>
```typescript
npm ERR! code ELITECYCLE
npm ERR! errno1
npm ERR! genesis@1.0.0 dev: 'webpack-dev-server --inline--progress --config build/webpack.dev.conf.js'
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the genesis@1.0.0dev script.
npm ERR! This is probably not a problem with npm. There si likely additional logging output above.
npm ERR! A complete log of this run can be found in:
npm ERR!  C:\Users\AppData\Roaming\npm-cache\_logs\2021-04-08T02_36_59_965Z-debug.log
```
![image](https://user-images.githubusercontent.com/74655724/118530465-a7455300-b77f-11eb-8772-a28f194cd4db.png)
</br>

## 아직 해결되지 않은 문제점
---
```typescript
Error: Cannot find module 'C:\workspace\genesis-master\build\webpack.dev.conf.js'
```

![image](https://user-images.githubusercontent.com/74655724/118530498-b2987e80-b77f-11eb-951e-93f19677a36c.png)
</br>

* 최종적으로 webpack문제 발생
* login과 setting화면에서 개인정보(이메일.패스워드)등을 입력하였을 때 renderer를 이용하여 app응용프로그램 시작불가
* login과 setting화면 router경로구성실패
* login예제 코드와 vue-genesis코드간의 package.json파일의 vue-genesis버전 차이 해결 못함(예제코드 version1.0.0, vue-genesis version0.0.1)
