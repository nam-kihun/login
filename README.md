# Nuxt와 vue환경의 마이크로 프론트엔드에서의 SEO 작업

## nuxt에서의 SEO 적용 방법
[관련문서](https://nuxtjs.org/docs/2.x/configuration-glossary/configuration-mode)<br>
nuxt에서 SPA를 사용하는 경우 지원하는 모드 중에 CSR과 SSR을 지원하는 `universe` 모드가 있다. 그리고 CSR만 지원하는 `spa`모드도 있다. SEO를 하기 위해서는 CSR이 필요하기 때문에 `universal` 모드를 적용하는 방향으로 작업을 진행했다. 우선 모드의 동작을 확인하고 사용법을 익히기 위해 기본적인 Nuxt 프로젝트를 만들고 앱 내에서 모드별로 동작을 시켜보았다. 

```vue.js
//nuxt.config.js
export default {
  mode: 'universal',
  ssr: true,
  ...
}
  ```

nuxt로 프로젝트를 구동시켰을 때 spa모드로 구동했을 경우 페이지에 관한 렌더링이 소스 코드에 없었지만 universe모드로 구동했을 때 페이지 렌더링이 소스 코드에 올라온 것을 확인하였다.


## Nuxt를 이용한 마이크로 프론트엔드 환경 구현
이전 작업에서 vue를 이용한 마이크로 프론트엔드 구현은 [single-spa](https://github.com/single-spa/single-spa)를 이용해 구현하였다. Nuxt를 이용한 마이크로 프론트엔드는 single-spa를 기반으로 하는 [qiankun](https://qiankun.umijs.org/guide)을 기반으로 구현된 것을 분석했다. Nuxt에서 SSR 사용 가능 여부를 확인하기 위해 qiankun기반으로 구현된 [데모](https://github.com/FEMessage/nuxt-micro-frontend)를 사용하였다.

## nuxt 기반으로 한 마이크로 프로젝트에서의 SEO 적용
qiankun과 nuxt를 기반으로 한 마이크로 프론트엔드의 코드를 살펴보니 spa모드로 적용되어 있는 것을 확인하였다. 이를 universal 모드로 변경했을 때 동작을 확인하였다. 결과로는 window 오류가 발생하였고 node_module에 있는 파일에서 정보를 받지 못하는 에러가 발생하였다.

![window 오류](https://github.com/jskim16/Nuxt-micro-frontend/blob/main/img/window-is-not-defined.PNG)

결론적으로는 이 오류는 해결할 수 없다는 [답변](https://github.com/umijs/qiankun/issues/772)이 있었다. 이유는 qiankun는 사용할 때 `import-html-entry`모듈이 사용되는데 서버 렌더링 때 window 객체를 가져올 수 없어서 그렇다고 한다. 그렇다면 universal 모드를 사용했을 때 window 객체를 받지 못해 오류로 인해 SSR이 적용되지 않았나 하면 그건 아니다. spa모드로 프로젝트가 동작할 때 페이지 소스와 universal모드로 프로젝트를 동작할 때를 비교하면 알 수 있다.

![spa모드](https://github.com/jskim16/Nuxt-micro-frontend/blob/main/img/spa-mode.PNG)

spa모드일 땐 페이지가 정상 작동을 하지만 소스를 보면 CSR로 구동하기 때문에 소스가 비어있는 것을 확인할 수 있다.

![universal모드](https://github.com/jskim16/Nuxt-micro-frontend/blob/main/img/universal-mode.PNG)

반면 universal모드로 동작했을 때는 페이지는 오류가 생기지만 소스를 보면 SSR로 구현이 되어 소스는 작성되어 있는 것을 확인할 수 있다.

## 정리
현재 Nuxt로 마이크로 프론트엔드 프로젝트를 만들려면 qiankun이 필요하기 때문에 qiankun을 사용하는 이상 Nuxt에서 SEO 동작은 어려울 것으로 보인다. 하지만 qiankun을 사용하지 않는 방식이 있다면 확인이 필요하다. Nuxt에 qiankun 대신 single-spa만으로 마이크로 프론트엔드 동작이 가능하다면 적용했을 때 SSR을 적용할 수 있을 가능성도 생각해 볼 수 있기는 한데 현재 single-spa 문서의 SSR은 React 기반으로 설명이 되어있다. Vue를 지원하는 SSR에 대해 설명이 필요할 것으로 보인다. 또 다른 가능성은 Vue 자체에서 SSR을 구현하는 방식을 찾아 마이크로 프론트엔드를 구현하는 것이다. 하지만 SSR이 가능하게 됐을 때 마이크로 프론트엔드를 지원하는 프레임워크들이 동작을 못할 가능성이 있다. 결론으로 현재로써는 Nuxt로 마이크로 프론트엔드를 Vue로 구현했을 때 SEO 적용이 어렵다는 것이다. SEO를 적용하려면 React 와 같이 구현하는 편이 가능성이 있어보인다.
