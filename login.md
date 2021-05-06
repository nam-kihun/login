# Vue 앱에 인증 추가

애플리케이션에 로그인 기능을 추가해야 한다. 이 작업을 수행하려면 Auth0의 인증 서비스를 사용해야한다.

먼저 여기서 무료 Auth0 계정을 등록한다. 등록되면 Auth0 관리 대시보드로 이동한다.

왼쪽 사이드바에서 "응용프로그램" 클릭
"응용 프로그램 만들기"라고 표시된 빨간색 큰 단추 클릭
이름을 "Vue Events"(또는 원하는 경우)로 지정한다.
"응용 프로그램 유형"을 보려면 "단일 페이지 웹 응용 프로그램"을 클릭한다.
"만들기" 클릭

🛠️️ Next, "Settings"를 클릭하여 Auth0이 응용 프로그램에 대한 인증을 구성하는 데 필요한 정보를 입력한다.

허용된 콜백 URL - http://localhost:8080

여기서 Auth0은 인증 후 사용자를 리디렉션한다.

허용된 로그아웃 URL - http://localhost:8080

사용자가 응용 프로그램에서 로그아웃한 후 다시 사용할 URL입니다.

허용된 웹 원본 - http://localhost:8080

Auth0가 크로스 오리진 인증에 사용하는 URL이다.

아래로 스크롤하여 "Save Changes" 버튼을 클릭한다.

일단 대시보드에서 필요한 것은 그것뿐이지만, 아직 클릭하지않는다. 대시보드의 이러한 값 중 일부를 곧 애플리케이션으로 가져와야 할 것이다.

# Auth0 SPA 패키지 설치

🛠️헤드 다시 터미널로 가서 Auth0's를 설치한다. auth0-spa-js 응용 프로그램의 루트 디렉터리에 패키지.

```javascript
npm install @auth0/auth0-spa-js
```

인증 래퍼 만들기
다음으로, Auth0 SDK 주위에 재사용 가능한 래퍼 Vue 개체를 생성한다. 이 래퍼를 애플리케이션의 나머지 부분에 노출하는 Vue 플러그인도 생성한다.

🛠️ 이를 위한 새 파일 및 폴더를 생성한다. 아직 안에 있는지 확인. events-app 루트 디렉터리 및 입력:

```javascript
mkdir src/auth touch src/auth/index.js
```

🛠️ 이제 새롭게 창조된 것을 연다. src/auth/index.js 다음 항목에 철하고 붙여넣기:

```javascript
import Vue from "vue"; import createAuth0Client from "@auth0/auth0-spa-js";  /** Define a default action to perform after authentication */ const DEFAULT_REDIRECT_CALLBACK = () =>   window.history.replaceState({}, document.title, window.location.pathname);  let instance;  /** Returns the current instance of the SDK */ export const getInstance = () => instance;  /** Creates an instance of the Auth0 SDK. If one has already been created, it returns that instance */ export const useAuth0 = ({   onRedirectCallback = DEFAULT_REDIRECT_CALLBACK,   redirectUri = window.location.origin,   ...options }) => {   if (instance) return instance;    // The 'instance' is simply a Vue object   instance = new Vue({     data() {       return {         loading: true,         isAuthenticated: false,         user: {},         auth0Client: null,         popupOpen: false,         error: null       };     },     methods: {       /** Authenticates the user using a popup window */       async loginWithPopup(o) {         this.popupOpen = true;          try {           await this.auth0Client.loginWithPopup(o);         } catch (e) {           // eslint-disable-next-line           console.error(e);         } finally {           this.popupOpen = false;         }          this.user = await this.auth0Client.getUser();         this.isAuthenticated = true;       },       /** Handles the callback when logging in using a redirect */       async handleRedirectCallback() {         this.loading = true;         try {           await this.auth0Client.handleRedirectCallback();           this.user = await this.auth0Client.getUser();           this.isAuthenticated = true;         } catch (e) {           this.error = e;         } finally {           this.loading = false;         }       },       /** Authenticates the user using the redirect method */       loginWithRedirect(o) {         return this.auth0Client.loginWithRedirect(o);       },       /** Returns all the claims present in the ID token */       getIdTokenClaims(o) {         return this.auth0Client.getIdTokenClaims(o);       },       /** Returns the access token. If the token is invalid or missing, a new one is retrieved */       getTokenSilently(o) {         return this.auth0Client.getTokenSilently(o);       },       /** Gets the access token using a popup window */        getTokenWithPopup(o) {         return this.auth0Client.getTokenWithPopup(o);       },       /** Logs the user out and removes their session on the authorization server */       logout(o) {         return this.auth0Client.logout(o);       }     },     /** Use this lifecycle method to instantiate the SDK client */     async created() {       // Create a new instance of the SDK client using members of the given options object       this.auth0Client = await createAuth0Client({         domain: options.domain,         client_id: options.clientId,         audience: options.audience,         redirect_uri: redirectUri       });        try {         // If the user is returning to the app after authentication..         if (           window.location.search.includes("code=") &&           window.location.search.includes("state=")         ) {           // handle the redirect and retrieve tokens           const { appState } = await this.auth0Client.handleRedirectCallback();            // Notify subscribers that the redirect callback has happened, passing the appState           // (useful for retrieving any pre-authentication state)           onRedirectCallback(appState);         }       } catch (e) {         this.error = e;       } finally {         // Initialize the internal authentication state         this.isAuthenticated = await this.auth0Client.isAuthenticated();         this.user = await this.auth0Client.getUser();         this.loading = false;       }     }   });    return instance; };  // Create a simple Vue plugin to expose the wrapper object throughout the application export const Auth0Plugin = {   install(Vue, options) {     Vue.prototype.$auth = useAuth0(options);   } };
```

이 파일의 설명은 현재 진행 중인 작업에 대한 세부 정보를 포함하지만 요약하자면 먼저 Auth0 SDK의 인스턴스를 만들거나 반환하는 것이다. 그 예는 단지 Vue 객체일 뿐이다.

인스턴스에는 다음 데이터가 포함되어 있다. loading, isAuthenticated, user, auth0Client, popupOpen, 그리고 error
또한 나중에 호출할 몇 가지 방법을 포함하지만, 지금 주의하십시오. loginWithPopup, handleRedirectCallback, loginWithRedirect, getIdTokenClaims, getTokenSilently, getTokenWithPopup, 그리고 logout
그 동안 created() 수명 주기 후크, SDK의 인스턴스를 생성하는 경우.

사용자가 "로그인"을 클릭하면 Auth0 Universal Login 페이지(이후 자세히 설명)로 리디렉션된다. 그들은 그곳에 자격 증명을 입력한 다음 다시 응용 프로그램으로 리디렉션될 것이다. 여기서 Auth0 대시보드의 "허용된 콜백 URL"이 실행된다. handleRedirectCallback() 인증된 사용자 데이터를 가져오고 토큰을 검색하며 업데이트하는 실행 isAuthenticated

이 인스턴스에는 또한 options 객체. 이 개체는 Auth0에 대한 값을 유지함 clientId, domain, 그리고 audience Auth0 대시보드에서.

```javascript
// Create a new instance of the SDK client using members of the given options object this.auth0Client = await createAuth0Client({   domain: options.domain,   client_id: options.clientId,   audience: options.audience,   redirect_uri: redirectUri });
```

이러한 값은 특별히 민감하지는 않지만, 소스 제어에서 제외하는 것이 여전히 좋은 관행이다(예: GitHub). 그러니 Git이 무시할 수 있도록 추가할 수 있는 파일을 만들어 본다.

🛠️️ Make sure you're still in the events-app 디렉터리 및 실행:

```javascript
touch auth_config.json
```

Git 또는 다른 버전 컨트롤을 사용하는 경우 열기 .gitignore 또는 그에 상응하는 것. 붙여넣기 auth_config.json 어느 선에서든 이제 이 파일은 다음에 당신이 당신의 repo에 밀어넣을 때 무시될 것이다.

🛠️️ 다음, op auth_config.json 다음 항목에 붙여넣기:

```javascript
{   "domain": "your-domain.auth0.com",   "clientId": "yourclientid" }
```

찾기 auth_config 값:

Auth0 대시보드로 이동
"응용프로그램"을 클릭하고 응용 프로그램을 선택한다.
"설정" 클릭
"Domain"의 값을 복사하여 붙여넣기 domain 에서 auth_config.json
"클라이언트 ID" 값을 복사하여 clientId 에서 auth_config.json
다음, 열기 src/main.js 플러그 인을 설치한다. Vue.use. 이 플러그인을 사용하면 애플리케이션 내 어디에서나 전역적으로 인증 상태에 액세스할 수 있다. Vue.use 플러그 인을 호출하는 데 사용되는 글로벌 방법이며, 플러그 인을 호출하기 전에 먼저 배치해야 함 new Vue().

🛠️️ 모든 것을 교체 src/main.js 다음을 포함하여:

```javascript
import Vue from 'vue' import App from './App.vue' import router from './router' import 'bulma/css/bulma.css';  // Import the Auth0 configuration import { domain, clientId } from "../auth_config.json";  // Import the plugin here import { Auth0Plugin } from "./auth";  // Install the authentication plugin here Vue.use(Auth0Plugin, {   domain,   clientId,   onRedirectCallback: appState => {     router.push(       appState && appState.targetUrl         ? appState.targetUrl         : window.location.pathname     );   } });  Vue.config.productionTip = false  new Vue({   router,   render: h => h(App) }).$mount('#app')
```

여기에 액세스하기 위해 생성한 Auth0 구성 파일을 가져오는 경우 domain 그리고 clientId 가치 다음, 다음 항목을 가져온다. Auth0Plugin 이전에 만들어진 것.

마지막으로 플러그인을 설치하고 구성한다.

# 로그인 및 로그아웃 버튼 연결

이제 Auth0 인증 플러그인이 구성되었으니, "Sign in" 버튼을 수정하여 실제로 무언가를 할 수 있도록 한다.

🛠️ 오픈 src/components/partials/Nav.vue. 로 시작하는 코드 블록 찾기 <div class="navbar-end"> 다음 항목으로 교체한다.

```javascript
<div class="navbar-end">   <div class="navbar-item">     <div class="buttons">       <!-- Check that the SDK client is not currently loading before accessing is methods -->       <div v-if="!$auth.loading">         <!-- show login when not authenticated -->         <a v-if="!$auth.isAuthenticated" @click="login" class="button is-dark"><strong>Sign in</strong></a>         <!-- show logout when authenticated -->         <a v-if="$auth.isAuthenticated" @click="logout" class="button is-dark"><strong>Log out</strong></a>       </div>     </div>   </div> </div>
```

단추는 이제 로 싸여 있다. !$auth.loading 사용자 상태에 액세스하기 전에 SDK 클라이언트가 로드를 완료했는지 확인한다. 다음으로, 당신은 @click를 호출하여 클릭 이벤트를 처리할 수 있는 login 또는 logout 사용자가 해당 버튼을 클릭하는 방법

🛠️ 지금 그 방법들을 만들어본다. 같은 파일에서 아래쪽으로 스크롤하여 <script> 태그를 지정하고 다음으로 교체한다.

```javascript
<script> export default {   name: 'Nav',   methods: {     // Log the user in     login() {       this.$auth.loginWithRedirect();     },     // Log the user out     logout() {       this.$auth.logout({         returnTo: window.location.origin       });     }   } } </script>
```

이제 애플리케이션으로 돌아가 "로그인"을 클릭하면 Auth0 Universal Login 페이지로 리디렉션된다.

Auth0 Universal Login 페이지를 누른 후 모의 사용자 계정으로 등록한다. 그러면 응용 프로그램이 사용자의 프로필과 전자 메일에 대한 액세스를 요청하고 있음을 알리는 화면이 나타날 것이다.
체크 표시를 클릭하면 애플리케이션 홈페이지인 대시보드에 지정한 "허용된 콜백 URL"로 다시 리디렉션된다. 이제 "로그인" 버튼 대신 "로그아웃" 버튼이 표시되어야 한다.

옵션: Auth0는 대시보드에서 바로 소셜 로그인 옵션을 제공한다. Google은 기본적으로 활성화되어 있으며, Auth0 관리 대시보드에서 개별적으로 더 많은 기능을 설정할 수 있다.

사이드바에서 「연결」 > 「소셜」을 클릭한다. 소셜 로그온을 통합하려면 자체 개발 키를 사용한다. 기본 Auth0 dev 키는 테스트에 적합하지만, 새로 고침 시 로그아웃되는 등 예기치 않은 오류가 발생할 수 있으므로 사용자 고유의 키를 사용하는 것이 좋다.

# 사용자 정보 액세스

Auth0를 사용하면 다음과 같이 템플릿에 로그인한 사용자의 정보에 액세스할 수 있다.

```javascript
{
  {
    $auth.user;
  }
}
```

의 내용 $auth.user 다음과 같은 모양을 하고 있다:

```javascript
{    "nickname": "hollylloyd",   "name": "Holly Lloyd",   "picture": "https://gravatar.com/somefancyimage.png",   "updated_at": "2019-10-09T15:49:28.181Z",   "email": "holly-lloyd@example.com",   "email_verified": false,   "sub": "auth0 xxxxxxxxxxxxxxx" }
```

나중에 프로필 페이지를 추가하려면 이 데이터(및 기타)에 액세스하여 표시할 수 있다.

이제 로그인 버튼을 추가할 수 있게 되었으니, 홈페이지에 있는 두 번째 버튼을 연결한다.

🛠️ 오픈 src/views/Home.vue 사이의 모든 것을 대체하다. <div class="button-block"></div> 다음을 포함하여:

```javascript
<div class="button-block">   <button v-if="!$auth.isAuthenticated" @click="login" class="button is-xl is-dark">Sign Up to Browse Events</button>   <h3 v-if="$auth.isAuthenticated" class="is-size-3 has-background-dark welcome">Welcome, {{ $auth.user.name }}!</h3> </div>
```

🛠️️ 이제 methods 섹션만 더하면 된다. login() 기능이다. 아래로 스크롤하여 <script> 태그 및 교체 export default {} 이것과 함께:

```javascript
export default {   name: 'home',   components: {     EventsList   },   methods: {     // Log the user in     login() {       this.$auth.loginWithRedirect();     }   } }
```

이제 사용자는 이 버튼으로도 로그인할 수 있으며, 일단 로그인되면, 그것은 그들의 이름을 사용하는 환영 메시지로 대체될 것이다. $auth.user.name.

# 경로 보호대 설정

이제 인증되지 않은 사용자를 단일 이벤트 페이지에서 다른 곳으로 리디렉션한다. 이는 카드/이벤트 목록이 있는 홈페이지를 볼 수 있어야 하지만 이벤트 세부사항 페이지를 클릭하자마자 로그인 페이지로 이동해야 한다는 것을 의미한다.

⚡️ 중요 ⚡ ️ 이것은 페이지 로드를 막을 뿐, 데이터가 로딩되는 것을 막지는 않는다. 데이터를 보호하기 위해 프런트 엔드에 의존해서는 안 된다.

백엔드 API를 아직 생성하지 않았으므로 프런트엔드의 구성요소에 있는 모든 데이터를 저장하기만 하면 된다. 인증되지 않은 사용자를 단일 이벤트 페이지에서 다른 곳으로 리디렉션하기 위해 라우트 가드를 설정하더라도, 인증되지 않은 사용자는 여전히 JavaScript 파일을 읽고 찾을 수 있다.

page.url에 '/amp/' %%가 포함된 경우 {%} 파트 2에서는 이 데이터가 백엔드에 저장 및 보호되고 사용자가 인증된 경우에만 프런트엔드로 끌어들이도록 API를 구축한다. {% 다른 %} 파트 2에서는 이 데이터가 백엔드에 저장 및 보호되고 사용자가 인증된 경우에만 프런트엔드로 끌어들이도록 API를 구축한다. {% endif %}

데이터가 백엔드에서 올 때 사용자를 로그인 페이지로 차게 하는 UI가 이미 설정되어 있도록 지금 경로 가드를 설정한다.

🛠️️ 라는 파일 생성 authGuard.js 에서 src/auth 전화번호부

{% 프리즘 bash %} touch src/auth/authGuard.js {% endprism %}

🛠️ ️ ️ ️ ️ that that that that that that that that that that.

```javascript
import { getInstance } from "./index";  export const authGuard = (to, from, next) => {   const authService = getInstance();    const fn = () => {     // If the user is authenticated, continue with the route     if (authService.isAuthenticated) {       return next();     }      // Otherwise, log in     authService.loginWithRedirect({ appState: { targetUrl: to.fullPath } });   };    // If loading has already finished, check the auth state using `fn()`   if (!authService.loading) {     return fn();   }    // Watch for the loading property to change before checking isAuthenticated   authService.$watch("loading", loading => {     if (loading === false) {       return fn();     }   }); };
```

getInstance 에서의 방법. src/auth/index.js 사용자가 로그인하지 않은 경우 라우트에 액세스할 수 없도록 하는 기능을 구현하는 파일.

사용자가 인증된 경우 next() 사용자가 클릭한 경로로 계속 이동할 수 있도록 반환됨. 인증되지 않은 사용자는 Auth0 Universal Login 페이지로 리디렉션된다.

다음으로, 당신은 라우터에 이 인증 가드를 추가해서 이것이 어떤 보기가 반환되기 전에 실행되도록 해야 한다.

사용자가 인증되었는지 간단하게 확인한다. 만약 그렇다면, 그들이 통과하게 하고 그렇지 않다면, 그들을 로그인 페이지로 보내라.

🛠️️ 라우터 파일 열기 src/router/index.js 다음 항목으로 교체한다.

```javascript
import Vue from 'vue' import Router from 'vue-router' import Home from '../views/Home.vue' import { authGuard } from "../auth/authGuard";  Vue.use(Router)  export default new Router({   mode: 'history',   base: process.env.BASE_URL,   routes: [     {       path: '/',       name: 'home',       component: Home     },     {       path: '/about',       name: 'about',       component: () => import('../views/About.vue')     },     {       path: '/event/:id',       name: 'eventSingle',       component: () => import('../views/EventSingle.vue'),       beforeEnter: authGuard     }   ] })
```

그 authGuard 맨 위에 수입된다. 이벤트 세부 정보 경로에 인증만 필요하므로 이 경로에 추가됨.

이전에 로그인한 적이 있다면 해당 이벤트 카드 중 하나를 클릭해도 이벤트 페이지 하나를 볼 수 있다.

앞에서 논의한 바와 같이, 이것은 사용자가 해당 페이지의 데이터를 찾는 것을 막지 않는다.
