# Workspace에 대한 개념과 사용

## workspace의 개념

<br>

[참고문서](https://musma.github.io/2019/04/02/yarn-workspaces.html)<br>
Workspace는 SPA에서 각 앱들이 같은 모듈을 공유하고 서로를 참조할 수 있게 하는 공간이다. 앱 개발시 여러 패키지를 설치하게 되는데 단 한번의 `yarn install`만으로 모든 패키지를 설치하는 것이다. Workspace를 통해 각 앱들은 node modules를 공유하면서 일부 devDependencies에 따라 소수의 모듈만을 앱 내에 받아 사용하게 되며 Workspace의 목록에 등록된 앱들이 모듈처럼 사용될 수 있게 만들어준다.

<br>

## Workspace의 사용 목적

<br>

프로젝트를 진행하면서 각각의 노드 모듈을 설치하게 되면서 서버 동작시 각각의 노드 모듈에서 동일한 모듈을 사용하는 일이 있다. 여러 모듈에서 동일한 모듈을 돌리면 그만큼 받아야 할 내용도 많아지고 통신량도 늘어난다. 그리고 프로젝트 진행을 하면서 노드 모듈 재설치를 할 때마다 각 프로젝트마다 노드 모듈을 통으로 설치하는 일이 많았기 때문에 시간 소요가 많았다. 워크스페이스는 하나의 패키지로 관리하므로 더 많은 프로젝트를 쓸 경우와 현재 프로젝트 관리에 대한 문제 해결이 가능해진다.

<br>

## Workspace의 장점

* 패키지의 dependancies가 서로 같을 수 있다. 이 때 패키지들을 관리하려면 항상 프로젝트마다 사용 가능한 최신 코드를 설치해 쓰게 된다. 프로젝트마다 이런 과정을 거치는 것 대신 워크스페이스 하나에 의존하게 할 수 있다. 또한 구조적으로도 `yarn link`로 전체의 시스템을 관리하는 것보다 하나의 워크스페이스 트리를 관리하는게 더 좋다.
* 프로젝트의 dependancies가 같이 설치될 때 최적화를 더 잘 할 수 있게 yarn에 여유를 줄 수 있다.
* 각각의 프로젝트마다 서로 다른 lock파일을 쓰기 때문에 여러 충돌 가능성이 있고 코드 리뷰가 어려운 문제가 있는데 이를 워크스페이스를 쓰면 하나의 lock파일만 써서 이런 문제를 다소 해결할 수 있다.

<br>

## Workspace의 구현방법

<br>

아래의 코드를 `package.json`에 추가한다. 이 파일이 들어있는 폴더를 워크스페이스 루트라고 부른다. 여기서는 워크스페이스 루트로 SPA프로젝트의 루트 폴더를 쓰며 여기에 `package.json`을 생성하거나 이미 있는 것을 사용한다. 이곳에서 앱들을 Workspace에 등록한다.

```vue.js
//package.json
{
  "private": true
  "workspaces": [
  "workspace-a", 
  "workspace-b", 
  ...
  ]
}
```
<br>

Workspaces 안에 사용할 프로젝트의 폴더를 등록한다. Workspace는 반드시 private 상태일 때 동작하므로 `"private": true`를 설정 후 Workspace를 사용할 수 있다. private를 사용하는 이유는 워크스페이스는 배포되는 폴더가 아니기 때문이다. 그래서 private를 추가함으로써 어떤것도 우연치 않게 노출되지 않게 하는 안전 수단을 추가하는 것이다.

<br>

### workspace-a/package.json:
```vue.js
//package.json
{
  "name": "workspace-a",
  "version": "1.0.0",

  "dependencies": {
    "cross-env": "5.0.5"
  }
}
```
### workspace-b/package.json:
```vue.js
//package.json
{
  "name": "workspace-b",
  "version": "1.0.0",

  "dependencies": {
    "cross-env": "5.0.5",
    "workspace-a": "1.0.0"
  }
}
```
>코드 작성 후 루트 폴더에서 이 코드를 yarn으로 설치를 해준다.

`$ yarn install`

그러면 `package.json`에 따라 루트 폴더에 node_modules와 yarn.lock 파일이 생성된다. 앱들은 node_moudules를 공유하게 되고 yarn은 워크스페이스의 앱들을 로컬 패키지로 인식한다. 제대로 설치가 되었다면 아래의 구조를 따를 것이다.

```
/package.json
/yarn.lock

/node_modules
/node_modules/cross-env
/node_modules/workspace-a -> /workspace-a

/workspace-a/package.json
/workspace-b/package.json
```

`/node_modules/workspace-b`를 찾지 않도록 한다. 다른 패키지가 dependancy로 이것을 사용하지 않는 이상 없을 것이다.

이렇게 함으로써 `workspace-a`를 필요로 하는 `workspace-b`에 있는 파일은 워크스페이스에 있는 파일을 가져오게 되고 두 프로젝트에서 필요한 `cross-env`패키지도 워크스페이스에 들어가 있다. 주목할 점은 `workspace-a`가 심볼릭 링크를 통해서 `/node_modules/workspace-a`라는 가명을 가지는 것이다. 이 기능은 이것이 패키지로 필요할 때 일반적인 패키지로 보이게 하는 것이다. 또한 폴더 이름이 아닌 `/workspace-a/package.json#name`필드가 이용되는 것을 알아야 한다. 이것은 `/workspace-a/package.json`의 `name`필드가 `"pkg-a"`였고, 가명이 `/node_modules/pkg-a -> /workspace-a`을 따르는 것이다. 그렇게 하면 `/workspace-a`에 있는 코드를 `const pkgA = require("pkg-a");`을(또는 `import pkgA from "pkg-a";`) 사용해 import 할 수 있다는 것이다.

<br>

## Workspace 동작 확인

<br>

Worspace의 동작을 확인하기 위해 깃허브에 올라온 [데모](https://github.com/anuroopjoy/npm7-sample)를 이용해 동작을 확인해 보았다. 원본에는 루트의 `package.json`에 libs 하위 폴더만 추가되었기 때문에 apps의 하위 폴더도 등록을 해주고 돌려본 결과와 원본을 비교해보았다. 결론은 둘 다 같은 결과를 얻을 수 있었다. 이유는 Workspace에 등록 후 yarn이 libs의 하위 폴더에 있는 프로젝트들을 로컬 npm 패키지로 인식해 관리하기 때문에 앱들이 yarn start로 서버를 돌릴 때 같은 루트에 있는 앱들이 libs를 사용할 수 있게 된다. 다만 apps를 등록하지 않았을 경우 앱들은 별도로 자신의 node_modules가 필요해진다.

<br>

## Workspace구현

<br>

루트 폴더에 Host(layout), Remote(home), Remote2(sub)가 되는 프로젝트가 있다. Workspace를 사용하지 않는다면 각 프로젝트마다 노드모듈을 설치해 실행시키겠지만 루트 폴더의 `package.json`에서 프로젝트들을 등록하고 모듈들을 한 번에 관리할 수 있다.

#### package.json
```vue.js
{
  "private": true,
  "scripts": {},
  "devDependencies": {},
  "workspaces": [
    "home",
    "layout",
    "sub"
  ]
}
```
>루트에서 `yarn isntall`을 하면 루트에 노드 모듈과 yarn.lock 파일이 생성되고 각 프로젝트 내의 노드 모듈 폴더에는 프로젝트마다 필요한 모듈만 설치되어 있는 것을 확인할 수 있다.

<br>

![이미지](https://github.com/jskim16/workspace-/blob/main/img/root.PNG)

<br>

위 프로젝트는 [demo 브랜치](https://github.com/jskim16/workspace-/tree/demo)에서 확인 가능하다.

</br>

![image](https://user-images.githubusercontent.com/74655724/118648077-01482600-b81d-11eb-8e55-75aead26c96f.png)

</br>

프로젝트 루트에서 yarn install 또는 yarn을 실행하면 루트의 package.json안에 명시되어 있는 dependency, 그리고 각 패키지에 명시되어있는 dependency가 중복을 최대한 줄인 채 루트의 node_modules안에 호이스팅 되어서 설치되고 dependency로 명시 되어있는 모듈은 심링크가 걸려 npm에 배포되어있는 버전이 아니라 로컬에 있는 코드를 바로 볼 수 있게 해준다.

<br>

## Lerna와 비교
* 배포와 버전관리는 Lerna로 - yarn의 워크스페이스는 low-level-primitive라서 Lerna와 같은 도구들이 사용할 수 있다. 워크스페이스는 Lerna가 하는 만큼의 high-level 기능을 지원하지 못한다. 하지만 yarn에서 자체적으로 해결 코드 로직을 구현하고 단계를 연결하는 것으로 새로운 사용과 향상된 기능을 기대할 수 있다.

* 패키지 관리는 yarn으로 - yarn은 yarn workspaces 를 추가적인 라이브러리 설치 없이 쉬운 방법으로 제공하고 있다. Lerna + yarn workspaces의 장점으로는 yarn workspaces가 불필요하게 lerna bootstrap 등의 명령을 실행하지 않으면서 더 안전하고(버그없이) 깔끔하게 패키지를 관리해준다.

<br>

## 팁과 기능
* `workspaces`공간은 워크스페이스가 들어가는 배열이므로 하나하나 열거하기보다 글로벌 패턴으로 한번에 여러개를 추가할 수 있다. 글로벌 패턴의 예를 들어 Bable은 모든 패키지를 `packages/*`로 접근할 수 있는 것 처럼 워크스페이스도 이 형태로 한번에 추가할 수 있다.
* 워크스페이스는 대규모 앱에서도 안정적으로 작동해 일반적으로 설치할 때부터 방식을 변경할 필요가 없지만 워크스페이스가 무언가를 부수고 있다고 생각되면 Yarnrc 파일에 아래를 추가해서 워크스페이스를 비활성화 할 수 있다.
`workspaces-experimental false`
* 하나의 워크스페이스만 변경할 경우 sibling dependacies을 처음부터 만들지 말고 빠르게 레지스트리로부터 설치할 수 있게 [-focus](https://classic.yarnpkg.com/blog/2018/05/18/focused-workspaces/)를 사용할 수 있다.

<br>

## 한계와 경고
* 패키지 레이아웃은 워크스페이스의 것과 사용자가 받을 것이 다를 수 있다. 워크스페이스 dependancies는 파일시스템 위계로 호이스팅 될 수 있다. 호이스팅 과정은 표준화된 것이 아니라서 레이아웃에 대한 가정은 위험하며 이론적으로는 새로운 것이 없다. 이런 문제가 발생하면 [the nohoist option](https://classic.yarnpkg.com/blog/2018/02/15/nohoist/)을 사용해보도록 한다.
* 위의 예제에서 `workspace-b`가 `workspace-a`의 package.json 가 아닌 다른 버전에 종속된다면 dependancy는 연결된 로컬 파일 시스템이 아닌 npm으로부터 설치가 된다. 이는 Bable과 같은 일부 패키지가 새로운 것을 만들기 위해서 이전 버전이 필요하기 때문이다.
* 워크스페이스에서 패키지를 배포할 때는 조심해야 하는데 새로운 배포를 준비중이고 새로운 dependancy를 사용하지만 아직 `package.json`에 등록하지 않았을 경우, 만약 다른 패키지가 워크스페이스 루트로 그 새로운 dependancy를 다운로드 받았을 때, 테스트 작업이 로컬로 처리될 수 있다.
* 워크스페이스들은 폴더의 계층구조 측면에서 워크스페이스 루트의 하위 구조여야 한다. 이 파일 구조 바깥에 있는 다른 워크스페이스는 참조할 수 없고 참조해서도 안된다.
* 현재로써는 중첩 워크스페이스는 지원되지 않는다.

<br>

## 참고문서
https://musma.github.io/2019/04/02/yarn-workspaces.html<br>
https://classic.yarnpkg.com/en/docs/workspaces/
