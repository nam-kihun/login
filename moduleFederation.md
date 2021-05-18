
## 예제를 통한 이해

### **Home**

webpack.config.js

```javascript
plugins: [
  new ModuleFederationPlugin({
    name: "home",
    filename: "remoteEntry.js",
    remotes: {
      home: "home@http://localhost:3002/remoteEntry.js",
    },
    exposes: {
      //내보낼(공유할) 컴포넌트
      "./Content": "./src/components/Content",
      "./Button": "./src/components/Button",
    },
  }),
];
```

<br>

### **Layout**

webpack.config.js

```javascript
plugins: [
  new ModuleFederationPlugin({
    name: "layout",
    filename: "remoteEntry.js",
    remotes: {
      home: "home@http://localhost:3002/remoteEntry.js",
    },
    exposes: {}, //내보내지 않고 받기만 해서 텅빔
  }),
  new HtmlWebpackPlugin({
    template: path.resolve(__dirname, "./index.html"),
    chunks: ["main"],
  }),
];
```

main.js

```javascript
import Layout from "./Layout.vue";

const Content = defineAsyncComponent(() => import("home/Content"));
const Button = defineAsyncComponent(() => import("home/Button"));

const app = createApp(Layout);

app.component("content-element", Content);
app.component("button-element", Button);
```

home의 컴포넌트들을 <content-element>, <button-element>라는 태그로 layout에서 사용할 수 있도록 함.

<br>

## 서브 컴포넌트 앱 추가 해보기

1. home 폴더를 복사해서 sub라는 이름의 폴더로 변경
2. home의 컴포넌트와의 차이를 주기 위해 컴포넌트의 제목과 내용을 변경
3. package.json에서 이름을 /sub로 변경, scripts에서 serve를 3003번 포트로 변경
4. devServer에서 port를 3003번으로 변경
5. sub 폴더 메인 앱에서 컴포넌트 정보 변경(제목을 바꿈에 따른 import 주소 변경, 태그명 변경)
6. webpack.config.js에서 ModuleFederationPlugin 중 exposes에 내보낼 컴포넌트 등록
7. layout main.js에 등록

_=> 여기까지 하면 sub 자체 실행을 하면 잘 실행 되지만, layout에 등록시 에러 발생_

8. layout의 webpack.config.js에 ModuleFederationPlugin에 리모트할 컴포넌트 주소 추가

```javascript
      remotes: {
        home: "home@http://localhost:3002/remoteEntry.js",
        sub: "sub@http://localhost:3003/remoteEntry.js",
      }
```

![Alt text](result.png)

잘됨

## workspaces 활용해보기

package.json

```javascript

{
  "private": true,
  "workspaces": [
    "folder-a",
    "folder-b",
    "folder-c"
  ]
}

```

1. 워크스페이스에 다음과 같이 node-modules를 통합할 **_폴더명_** 을 입력
2. 그 다음에는 해당 폴더들을 통합하는 워크스페이스에 yarn install을 실행
3. 개별적으로 가진 것들을 제외한 모듈들이 워크스페이스로 통합됨
