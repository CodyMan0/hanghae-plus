---
title: "SPA without a Framework (항해 플러스 프론트엔드)"

date: "2024-12-22"
lastmod: "2024-12-22"
tags: ["항해솔직후기", "항해플러스프론트엔드", "항해플러스후기"]
draft: false
summary: "SPA without a Framework"
---

<TOCInline toc={props.toc} exclude="Introduction" />

> 오랜만에 글쓰기를 복귀합니다. 이제야 제 자리로 돌아온 것 같습니다. 24년 취뽀 이후, 6개월간 회사 내에서 진행한 프로젝트를 진행하고 개인적으로 여러 일들이 있었네요. 현재 24년을 돌아보며 기술적인 성장과 커뮤니케이션 부분에서의 성장을 정리하고 있습니다. 그중 당연 하나의 큼지막한 일인 항해 플러스 1주 차를 돌아보는 시간을 가지려고 합니다.

합류를 결심한 계기서부터 과제를 만나면서 어떤 어려움이 있었고 어떻게 해결했는지 그리고 그 과정에서 어떤 것들을 배웠고 실무에서 어떤 부분에 적용할 수 있을지 그리고 더 나아가 react-router 라이브러리 내부 코드를 보다 조금 더 이해할 수 있던 것들을 공유하고자 합니다.

> 시작하겠습니다.

# 합류를 결심한 계기

2024년, 개발자로 커리어를 시작하게 됐고 6개월간 사내에서 두 개의 프로젝트를 진행하였습니다. 하나는 플랫폼 및 플랫폼을 위한 어드민, 두 번째 운용 고객들을 위한 대시보드와 트레이더분들의 인사이트를 도출을 돕고 현 상황을 볼 수 있는 대시보드를 구현하는 것이었습니다. 그 과정에서 백엔드 개발자분과 수많은 이야기를 진행했으며 결과를 만들어가던 중 연말 피드백을 받게 됐습니다.

**연말 동료 평가를 통해 받은 피드백은 다음과 같습니다.**

1.  피드백에 대해 수용하고 변화하려는 태도가 인상적이라는 것.
2.  유일한 프론트 개발자로서 필요한 기능을 구현하는데 핵심적인 역할을 수행하여 주고 있다는 것.
3.  아직은 주니어 개발자로서, 회사의 업무를 더 정교하고 더 빠르게 수행하기 위해 더 많은 노력이 필요하다는 것.
4.  다소 형식적일 수 있는 부분에 매달리는 듯한 경향이 있다는 것.

4가지 피드백을 통해 스스로 도출한 결론은 다음과 같습니다.

> 현재는 개발 실력을 더 키워야 하고 지식을 위한 지식은 이제 그만!!

회사에서는 현재 react.js를 활용하여 개발을 진행하고 있습니다. 그렇기에 react.j를 보다 잘 알고 있어야 합니다. 실은 제가 만나는 문제의 90퍼센트는 리액트 내부 동작을 알고 있지 않아도 생각보다 쉽게 해결 가능했습니다. 상태의 위치를 변경한다거나 인터페이스의 구조를 변경하거나 혹은 사용하고 있는 라이브러리 공식 문서 등 찬찬히 읽어보면 해결할 수 있는 문제들이 대부분입니다. 하지만 늘 갈망하던 유의미한 문제를 해결하기 위해선 더욱 깊이 라이브러리에 대한 이해와 사용하고 있는 오픈소스의 동작원리를 알아야 가능함을 느끼고 있습니다.

> 그래서 항해 플러스 프론트엔드 4기에 합류하게 됐습니다.

`정리해보면,`

- React를 보다 깊이 사용할 수 있기 위해 (이제 지식을 위한 지식은 경계)
- 깔끔한 코드를 작성하여 새로운 동료가 왔을 때 무맥락일 때조차 이해할 수 있게(현재 사내에서 노력하고 있으나 만족스럽지 않음.),
- 리팩터링이 가능한 환경을 만들기 위해 (충분히 이해해주는 상황임에도 리팩터링으로 인해 기능이 돌아가지 않아 Roll back 하는 경우가 잦음... 결국 시간 낭비가 되는 경우가 많음. 그래서 테스트 코드가 필요함)
- 현재 까보며 이해해보고 있는 react-router와 react-hook-form 그리고 tanstack-query를 보다 더 깊이 이해하고 실무 코드에 녹여내기 위해

# 발제 과제 (프레임워크 없이 SPA 만들기)

상태가 변경되었을 때, Window의 History 스택에 변화가 있을 때 감지하여 페이지의 UI를 바꿔줘야 하는 과제였습니다. 처음 과제를 받고 파일의 시작점이었던 main.js에서 방황했었습니다. 어떤 구조로 구현해야 하는지, 이벤트는 어떻게 바인딩하는 건지...

![웃긴짤 : 써먹기 유용한 것들 모음 : 네이버 블로그](https://mblogthumb-phinf.pstatic.net/MjAxNjEyMDNfMTQ4/MDAxNDgwNzM5NzY4NTYw.F6HSIg7cgvLLJPI2tQUWq-PEpDKQ62SiII5Po694fo8g.eH-LdVOVPpNLgmtRKg6MW5U1dsCEU_D8UewC5jd0QwIg.PNG.zldnld99/%EC%9B%83%EA%B8%B4%EC%A7%A410.png?type=w800)

> 크게 보면 이벤트 처리를 어떻게 할 것인가가 관건이라고 생각합니다.

## 이벤트 처리 구현

### URL 변경시 이벤트 처리

```jsx
//index.html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>항해플러스 SNS</title>
    <script src="https://cdn.tailwindcss.com"></script>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>

//main.js
function updateDOM() {
  // 렌더링
  const path = window.location.pathname;
  const user = localStorage.getItem("user");
  if (!user && path === "/profile") {
	render('/login');
    return;
  }
  render(path);
}

function render(path) {
  window.history.pushState(null, "", path);
  document.body.innerHTML = router(path);
}


window.addEventListener("popstate", updateDOM);
window.addEventListener("load", updateDOM);

```

index.html에서 body 최 하단에 script 태그를 두어 Tag를 참조하는데 문제 되지 않도록 놓고 main.js 내부에 이벤트를 추가하였습니다. 페이지가 로드될 때 트리거되도록

window.addEventListener("load", updateDOM);를 브라우저의 History 객체에 변경이 되었을 때, 즉 window.history.go(), window.history.forward(), window.history.back() 메서드 호출 시에도 이벤트가 트리거 되어 2번째 인자인 updateDOM 함수가 실행되도록 구현하였습니다.

여기서 pushState는 주의해야 할 것이 있습니다. history.pushState을 진행하여 페이지를 이동시켜도 해당 메소드는 이벤트를 발생시키지 않습니다. 그래서 하단에 document.body.innerHTML을 통해 window.location의 pathname에 상응하는 컴포넌트를 가지고 와서 호출하여 DOM을 그려주는 역할을 하도록 구현하였습니다.

### 개별 클릭 및 제출 이벤트 처리

간단한 예시를 살펴보면 좋을 것 같습니다. 로그인 버튼을 클릭하면 로그인이 되도록... 어렵지 않은 기능인데 어떻게 구현할 수 있을지 생각보다 어려웠습니다.

반드시 HTML이 파싱 돼 DOM에 반영된 이후 DOM에 접근하여 JS가 해당 엘리먼트를 제어할 수 있어야 했습니다. 그러니 컴포넌트가 먼저 실행된 후, JS가 실행되어야 했습니다.

```jsx
//main.js
const routes = {
  "/": () => HomePage(),
  "/profile": () => ProfilePage(),
  "/login": () => LoginPage(),
  404: () => Error404Page(),
};

function updateDOM() {
  // 렌더링
  const path = window.location.pathname;
  const user = localStorage.getItem("user");
  if (!user && path === "/profile") {
    render("/login");
    return;
  }
  render(path);
}

function render(path) {
  window.history.pushState(null, "", path);
  document.body.innerHTML = router(path);
  // ADD HERE!!
  router(path).eventFn?.();
}

window.addEventListener("popstate", updateDOM);
window.addEventListener("load", updateDOM);

// 예시 LoginPage.js
export const LoginPage = () => `
  <div class="bg-gray-100 flex items-center justify-center min-h-screen">
    <div class="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
      <h1 class="text-2xl font-bold text-center text-blue-600 mb-8">항해플러스</h1>
      <form id="login-form">
        <input type="text" id="username" placeholder="사용자 이름" class="w-full p-2 mb-4 border rounded" required>
        <input type="password" placeholder="비밀번호" class="w-full p-2 mb-6 border rounded" required>
        <button type="submit" class="w-full bg-blue-600 text-white p-2 rounded">로그인</button>
      </form>
      <div class="mt-4 text-center">
        <a href="#" class="text-blue-600 text-sm">비밀번호를 잊으셨나요?</a>
      </div>
      <hr class="my-6">
      <div class="text-center">
        <button class="bg-green-500 text-white px-4 py-2 rounded">새 계정 만들기</button>
      </div>
    </div>
  </div>
`;
```

함수가 호출되면 템플릿 리터럴 작성된 HTML을 리턴하는 코드입니다. 로그인 페이지가 렌더링 된 이후 로그인 페이지 내부에 있는 엘리먼트에 접근해야 했습니다. 왜냐하면 로그인 버튼을 클릭하면 해당 이벤트가 실행되어야 했기 때문이죠.

```js
//main.js
function render(path) {
  window.history.pushState(null, "", path);
  document.body.innerHTML = router(path);
  // ADD HERE!!
  router(path).eventFn?.();
}
```

그래서 main.js에서 render 함수에 해당 페이지가 innerHTML로 그려진 후, 페이지 파일에 eventFn이 존재하면 해당 함수를 호출하는 방식으로 구현해 보았습니다.

LoginPage.js를 다음과 같이 구현해보았습니다.

```jsx
import { userAPI } from "../../api/userAPI";
import { MyRouter } from "../../shared/router/router";

const LoginPage = () => {
  return `
  <main class="bg-gray-100 flex items-center justify-center min-h-screen">
  <div class="bg-white p-8 rounded-lg shadow-md w-full max-w-md">
    <h1 aria-role="heading" class="text-2xl font-bold text-center text-blue-600 mb-8">항해플러스</h1>
      <form id="login-form">
        <div class="mb-4">
          <input id="username" type="text" placeholder="사용자 이름" class="w-full p-2 border rounded">
        </div>
        <div class="mb-6">
          <input type="password" placeholder="비밀번호" class="w-full p-2 border rounded">
        </div>
        <button type="submit" class="w-full bg-blue-600 text-white p-2 rounded font-bold">로그인</button>
      </form>
      <div class="mt-4 text-center">
        <a href="#" class="text-blue-600 text-sm">비밀번호를 잊으셨나요?</a>
      </div>
      <hr class="my-6">
      <div class="text-center">
        <button class="bg-green-500 text-white px-4 py-2 rounded font-bold">새 계정 만들기</button>
      </div>
    </div>
  </main>
`;
};

LoginPage.eventFn = () => {
  const form = document.getElementById("login-form");
  form.addEventListener("submit", (e) => {
    e.preventDefault();
    userAPI.login({
      username: form.querySelector("input[type=text]").value,
    });
    MyRouter.push("/");
  });
};

export default LoginPage;
```

이렇게 하면 문제없이 로그인 기능을 구현할 수 있고 각 페이지 내부에 필요한 함수를 작성할 수 있게 됐습니다. `동료들의 코드를 참고하여 구현하였고 너무 많은 도움을 받았습니다. 감사해요~.`

## 더 나은 방법 및 고민 과정

위의 코드를 통해 아주 간단히 SPA를 구현한 과정을 보실 수 있었습니다.

- 브라우저의 뒤로 가기와 앞으로 가기 버튼 이벤트를 감지해서 페이지를 변경한다.
- 어플리케이션 내부에서 history.pushState을 통해 url을 변경하면 감지하여 페이지를 변경한다.

또한 이벤트를 최상단 main.js에 페이지가 로드되었을 때와 popState 이벤트를 걸어두고 개별적인 것들은 각 페이지 내부에 페이지가 로드되면 호출되도록 구현해 놓았습니다.

> 더 좋은 방법이 있을까요? 당연하죠!

## 더 나은 이벤트 처리

### URL 변경시 이벤트 처리

전 main.js에 두 가지 코드를 추가하고 pushState는 이후 innerHTML로 DOM을 리렌더링 하였습니다.

```jsx
window.addEventListener("popstate", updateDOM);
window.addEventListener("load", updateDOM);
```

하지만 멘토님의 코드는 달랐습니다. 우선 옵저버 패턴을 활용하여 createRouter 함수를 구현하였습니다.

우선 옵저버 패턴을 쉽게 생각해 보면 말 그대로 객체의 상태를 살펴보고 있다가 변경되면 종속된 객체들에게 변경됨을 알려주는 패턴이라고 할 수 있습니다. 자세한 것은 코드를 통해 살펴보죠.

```jsx
//createRouter.js
import { createObserver } from "./createObserver";

export const createRouter = (routes) => {
  const { subscribe, notify } = createObserver();

  const getPath = () => window.location.pathname;

  const getTarget = () => routes[getPath()];

  const push = (path) => {
    window.history.pushState(null, null, path);
    notify();
  };

  window.addEventListener("popstate", () => notify());

  return {
    get path() {
      return getPath();
    },
    push,
    subscribe,
    getTarget,
  };
};
```

createRouter.js를 살펴보겠습니다. 우선 라우트들을 인자로 받습니다. 그리고 바로 분리되어 있는 createObserver 함수가 보입니다. 해당 함수는 옵저버를 만드는 함수일 것으로 예상됩니다.

```jsx
export const createObserver = () => {
  const listeners = new Set();
  const subscribe = (fn) => listeners.add(fn);
  const notify = () => listeners.forEach((listener) => listener());

  return { subscribe, notify };
};
```

중복을 방지하기 위해 Set 자료 구조를 할당합니다. 그리고 subscribe는 콜백함수를 Set에 등록하는 역할 그리고 notify는 해당 Set을 순회하면서 등록된 listener 함수를 호출하는 역할을 수행하고 반환되고 있습니다.

다시 돌아가서,

```jsx
//createRouter.js
import { createObserver } from "./createObserver";

export const createRouter = (routes) => {
  const { subscribe, notify } = createObserver();

  const getPath = () => window.location.pathname;

  const getTarget = () => routes[getPath()];

  const push = (path) => {
    window.history.pushState(null, null, path);
    notify();
  };

  window.addEventListener("popstate", () => notify());

  return {
    get path() {
      return getPath();
    },
    push,
    subscribe,
    getTarget,
  };
};
```

path와 Target을 createRouter 함수 호출 시 들어오는 인자를 기반으로 각 변수에 담아주고 있습니다. 그렇게 createRouter 함수를 만들어 사용하는 코드에선 다음과 같게 구현하여 SPA를 구현하고 있었습니다.

````jsx
router.set(
  createRouter({
    "/": HomePage,
    "/login": AuthGuard(Boolean, ForbiddenError, LoginPage),
    "/profile": AuthGuard((value) => !value, UnauthorizedError, ProfilePage),
  }),
);```

hashRouter 혹은 historyRouter에 의존하지 않기 위해 router 객체를 추가하여 구현하여 해당 라우터 객체에서 모든 라우터들을 관리하고 있습니다. 이 방식은 react-router 라이브러리와 비슷한 부분이 있는 것 같습니다.

```js
function main() {
  router.get().subscribe(render);
  ...
  render();
  }
main();
````

### 개별 클릭 및 제출 이벤트 처리

위 이벤트 처리 방식의 문제점이 무엇인가요? 의도치 않게 해당 함수 호출이 두어 번 될 경우 예상치 못한 중복 API 요청이 발생할 수 있습니다. 유지보수하기 까다로울 것으로 예상됩니다. 만약 1개의 페이지에 10개의 컴포넌트가 있는데 각 컴포넌트 안에 이벤트가 존재할 때 이벤트 순서를 일일이 생각하면서... 구현한다면 복잡성이 상당한데요. 그래서 이벤트를 위임하여 최상단에서 관리하는 방법을 정리해보려고 합니다.

해당 과제에 대한 멘토님의 코드이며, 1주 차 과제가 끝난 후, 여러 번 읽어보고 이해한 대로 정리해 봅니다.

전 상당히 단순하게 구현했었습니다. 해당 LoginPage가 호출된 후, LoginPage 내부 eventFn 함수가 호출되어 동작하게만 만들었습니다.

멘토님의 코드는 이벤트를 등록하는 로직이 추상화되어 있었습니다. 저의 LoginPage 코드를 활용해서 생각해 보면

```jsx
//BEFORE

//main.js
function render(path) {
  window.history.pushState(null, "", path);
  document.body.innerHTML = router(path);
  // ADD HERE!!
  router(path).eventFn?.();
}

//LoginPage.js
LoginPage.eventFn = () => {
  const form = document.getElementById("login-form");
  form.addEventListener("submit", (e) => {
    e.preventDefault();
    userAPI.login({
      username: form.querySelector("input[type=text]").value,
    });
    MyRouter.push("/");
  });
};

//AFTER

//main.js
function render(path) {
  window.history.pushState(null, "", path);
  document.body.innerHTML = router(path);
  // HERE
  registerGlobalEvents();
}

addEvent("submit", "#login-form", (e) => {
  e.preventDefault();
  userAPI.login({
    username: form.querySelector("input[type=text]").value,
  });
  MyRouter.push("/");
});
```

차이를 확인하셨나요? registerGlobalEvents함수와 addEvent 함수가 호출되고 있습니다. 각 함수의 역할과 내부 로직을 살펴보시죠.

#### registerGlobalEvents

```jsx
//eventUtils.js
const eventHandlers = {};

export const registerGlobalEvents = (() => {
  let init = false;
  return () => {
    if (init) {
      return;
    }

    Object.keys(eventHandlers).forEach((eventType) => {
      document.body.addEventListener(eventType, handleGlobalEvents);
    });

    init = true;
  };
})();

export const addEvent = (eventType, selector, handler) => {
  if (!eventHandlers[eventType]) {
    eventHandlers[eventType] = {};
  }
  eventHandlers[eventType][selector] = handler;
};
```

해당 함수는 즉시 실행 함수로 구현되어있습니다. 주로 화살표 함수를 활용하여 부모 실행 컨텍스트에서 분리하여 새로운 실행 컨텍스트를 만들어 사용했어서 사용해본 적이 없는 형태였습니다. registerGlobalEvents는 함수를 반환하고 있습니다. init은 클로져로 외부에서 접근할 수 없는 값이죠. 즉 init이 false일 경우 모든 핸들러를 순회하면서 그때 body에 이벤트를 걸어주고 있는 것을 확인할 수 있습니다. 그리고 Init을 true로 재할당하여 중복을 방지하고 있습니다.

#### handleGlobalEvents

여기서 handleGlobalEvents 함수를 살펴볼까요?

```jsx
const eventHandlers = {};

const handleGlobalEvents = (e) => {
  const handlers = eventHandlers[e.type];
  if (!handlers) return;

  for (const selector in handlers) {
    if (e.target.matches(selector)) {
      handlers[selector](e);
      break;
    }
  }
};
```

handleGlobalEvents 함수는 addEvent 함수를 통해 이벤트들이 적재된 객체에서 type을 가지고 옵니다. 만약 있다면 여러 개의 이벤트 중, selector와 동일한 타깃의 이벤트만 실행시켜 주고 즉시 for문을 종료합니다. 이벤트 위임 방식과 동일한 구조라고 생각됩니다.

우리가 일반적으로 알고 있는 이벤트 위임은 아래와 같죠!

(사진 추가 예정)

````jsx
document.getElementsByTagName('ul').addEventListener('click', function(e) {
    if(e.target && e.target.nodeName == "LI") {
        console.log("List item ", e.target.id, " was clicked!");
    }
});```

각 li에 모든 이벤트를 추가하는 로직을 한단계 끌어올려 ul 태그에 이벤트를 추가하고 동적으로 li와 관련된 console.log()를 실행시킬 수 있습니다. 이 방식과 상당히 유사하게 전역 이벤트를 관리하고 있습니다.

#### addEvent
```jsx
const eventHandlers = {}

export const addEvent = (eventType, selector, handler) => {

if (!eventHandlers[eventType]) {
	eventHandlers[eventType] = {};
}

eventHandlers[eventType][selector] = handler;
};
````

이벤트 타입, 필요한 이벤트 핸들러를 찾기 위한 식별자, 그리고 핸들러 함수를 인자로 받아 eventHandlers 객체에 저장합니다. 그렇게 해서

```jsx
//main.js
function render(path) {
  window.history.pushState(null, "", path);
  document.body.innerHTML = router(path);
  registerGlobalEvents();
}
```

main.js 파일이 실행될 때 전역 이벤트 함수가 호출되어 각 페이지 내부에 있는 이벤트들이 전역에 저장되고 해당 기능을 사용할 땐 selector를 키로 삼아 해당 핸들러 함수를 호출하는 방식으로 사용할 수 있게 됐습니다.

이로 인해 얻은 이점은 두 가지입니다.

- 중복을 방지할 수 있다는 것.
- 어플리케이션의 복잡성을 줄였다는 것.

# Further more

작성 예정 (2주차 과제로 얼른 넘어가야해서 조만간 돌아오겠습니다. ㅠㅠ)

# 감정 회고

## 조급하지 말자

과제 마감 기한이 있기에 조급해지고 불안감이 엄습하려고 할 때가 있었습니다. 그때 이것을 왜 지원했는지 어떤 것들을 얻길 원하는지를 상고해 보니 불안감이 사라지는 경험을 했습니다. 다양한 동료의 코드를 보고 그동안 지식적으로 알고 있던 개념들을 코드를 통해 더욱 체감하길 원합니다.

더 나아가 실무에서 더 기획 의도에 부합하고 변경에 유연한 코드를 작성할 수 있는 유연함과 힘을 갖길 원했습니다. 비록 이제 1주 차이지만... 남은 9주도 파이팅입니다!

![](https://velog.velcdn.com/images/mython/post/e5deb4de-ccc9-4144-b69e-b5025edd7314/image.png)
