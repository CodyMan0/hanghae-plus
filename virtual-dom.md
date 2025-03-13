---
title: "Virtual DOM [항해 플러스 리액트 복습]"
date: "2025-03-11"
lastmod: "2025-03-11"
tags: ["항해솔직후기", "항해플러스프론트엔드", "항해플러스후기", "회고"]
draft: false
summary: "머리로 알고 있는 것과 구현하는 것은 다르다..."
---

<TOCInline toc={props.toc} />

![](https://velog.velcdn.com/images/rgfdds98/post/095765c8-848e-4c38-a13b-e9e8253b38d9/image.png)

# 서론

3월 1일 부로 항해 플러스가 끝나게 됐습니다. 마음에 여유가 생겨서 그런지, 다시 각 파트 별로 어려웠던 부분들을 정리하며 복습하고자 합니다.
숨 가쁘게 달리기만 하다 보니 정작 이해하지 못하고 넘어간 게 무지... 많.. 더라고요.
해당 지식이 복잡한 문제를 해결하는 데 사용될 단서로 작용하도록 정확히 이해하고 Next.js로 넘어가려고 합니다.

> 이번 포스트에서는 가상돔 구현 과정을 다시 정리해 보고 실무에서 사용하고 있는 Vite는 어떻게 jsx를 트랜스파일하는지 살펴보며 마무리하고자 합니다.

# 본론

[브라우저 렌더링 과정](https://www.danny-log.xyz/blog/critical-rendering-path)을 잠시 잊었다면, 해당 포스트가 도움이 됩니다. 이를 기반으로 가상돔이 어떤 문제를 해결하고자 했는지 더욱 이해할 수 있습니다.

## Virtual Dom에 대해

브라우저 렌더링을 통해 브라우저 렌더링을 통해 파싱 된 DOM을 보았습니다. DOM의 가벼운 복사본을 리액트에선 가상돔이라 말하며, react 현재 공식문서에서는 가상돔에 대한 언급보다는 `reconciliation`과 `Fiber 아키택쳐`라고 많이 불립니다.

### 타입을 통해 먼저 알아보는 Virtaul DOM

전 JavaScript를 활용하여 과제를 진행했었습니다. 다시 한번 모든 레포를 보며 동료들의 코드를 참고하다 영서님의 코드를 통해 리팩터링을 진행해보았습니다.

```tsx
export interface VNodeProps {
  [key: string]: any;
  children?: VNodeChild[];
}

// 가상돔의 각 노드의 타입
export type VNode = {
  type: string | Function;
  props: VNodeProps | null;
  children: VNodeChild[];
};

export type VNodeChild = string | number | boolean | null | undefined | VNode;

// react 이벤트 타입을 그대로 타입으로 재현
type MouseEvent =
  | "click"
  | "dblclick"
  | "mousedown"
  | "mouseup"
  | "mousemove"
  | "mouseover"
  | "mouseout"
  | "mouseenter"
  | "mouseleave";
type KeyboardEvent = "keydown" | "keyup" | "keypress";
type FormEvent = "submit" | "change" | "focus" | "blur" | "input";
type TouchEvent = "touchstart" | "touchend" | "touchmove" | "touchcancel";
type DragEvent =
  | "dragstart"
  | "drag"
  | "dragenter"
  | "dragleave"
  | "dragover"
  | "drop"
  | "dragend";

export type DOMEventType =
  | MouseEvent
  | KeyboardEvent
  | FormEvent
  | TouchEvent
  | DragEvent;

export const eventTypes: DOMEventType[] = [
  "click",
  "dblclick",
  "mousedown",
  "mouseup",
  "mousemove",
  "mouseover",
  "mouseout",
  "mouseenter",
  "mouseleave",
  "keydown",
  "keyup",
  "keypress",
  "submit",
  "change",
  "focus",
  "blur",
  "input",
  "touchstart",
  "touchend",
  "touchmove",
  "touchcancel",
  "dragstart",
  "drag",
  "dragenter",
  "dragleave",
  "dragover",
  "drop",
  "dragend",
] as const;
```

위의 타입은 실제 리액트에서 사용하고 있는 이벤트들의 타입을 차용한 것으로 보입니다.

## 가상돔 구현을 위한 함수를 살펴봅시다

### 1. createVNode

> Virtual DOM의 노드를 생성하는 기초 함수입니다. type과 props 그리고 자식 컴포넌트를 인자로 받아 가상돔의 노드를 생성합니다.

```tsx
import { VNode, VNodeChild, VNodeProps } from "@types";

/**
 * Virtual DOM의 Node 생성 함수
 * @param type - HTML 태그 이름
 * @param props - 노드의 attributes
 * @param children - 자식 노드들
 * @returns Virtual DOM Node
 */

export function createVNode(
  type: string | Function,
  props: VNodeProps | null,
  ...children: VNodeChild[]
): VNode {
  const processChildren = (items: any[]): any[] => {
    return items
      .flat(Infinity)
      .filter((child) => child === 0 || Boolean(child));
  };

  return {
    type,
    props,
    children: processChildren(children),
  };
}
```

### normalizeVNode

정규화라는 단어가 익숙하실 수 있습니다. 주로 데이터 베이스에서 보았죠. 비슷한 의미입니다. 위에서 만든 VNode는 여러 형태의 값을 반환할 수 있습니다. 예를 들어

```tsx
const renderFn = (): string => "Hello World"; // 문자열
const renderFn = (): null => null; // 빈 값
const renderFn = (): number => 1; // 빈 값
```

각 VNode를 일괄적으로 동일한 구조로 만드는 것이 목적인 함수입니다.

```tsx
import { VNode, VNodeChild } from "@types";

export function normalizeVNode(vNode: VNodeChild): string | VNode {
  if (vNode === null || vNode === undefined || typeof vNode === "boolean") {
    return "";
  }

  if (typeof vNode === "number" || "string") {
    return String(vNode);
  }

  if (typeof vNode.type === "function") {
    const Component = vNode.type;
    const { children: propChildren, ...restProps } = vNode.props || {};

    const componentChildren = [
      ...(propChildren || []),
      ...(vNode.children || []),
    ];

    const result = Component({
      ...restProps,
      children: componentChildren
        .map((child) => normalizeVNode(child))
        .filter((child) => child !== ""),
    });

    return normalizeVNode(result);
  }

  return {
    type: vNode.type,
    props: vNode.props,
    children: (vNode.children || [])
      .map((child) => normalizeVNode(child))
      .filter((child) => child !== ""),
  };
}
```

### createElement

createVNode와 normalizeVNode를 통해 jsx를 Virtual Node로 변경하였습니다. 이젠 createElement 함수를 활용하여

정규화된 가상 DOM 요소를 실제 DOM 요소로 변환하는 과정을 거치게 됩니다. 이 함수에서도 마찬가지로 Node의 타입에 따라 다르게 반환해야 하는 게 핵심입니다.

```jsx
type VNode = {
  type: string | Function;
  props: VNodeProps | null;
  children: VNodeChild[];
};

type VNodeChild = string | number | boolean | null | undefined | VNode;

/**
 * Virtual DOM Node를 실제 DOM Element로 변환하는 함수
 * @param vNode - Virtual DOM Node
 * @returns HTMLElement, Text Node 또는 DocumentFragment
 */

export function createElement(vNode: VNodeChild | any) {
  const isFalsy = (vNode) => vNode === null || vNode === undefined || typeof vNode === "boolean";
  const isStringOrNumber = (vNode) => typeof vNode === "string" || typeof vNode === "number";
  const isArray = (vNode) => vNode && Array.isArray(vNode);

  if (isFalsy(vNode)) return document.createTextNode("");

  if (isStringOrNumber(vNode)) document.createTextNode(String(vNode));

  if (isArray(vNode)) {
    const fragment = document.createDocumentFragment();
    vNode.forEach((child) => {
      fragment.appendChild(createElement(child));
    });
    return fragment;
  }

  const element = document.createElement(vNode.type);

  updateAttributes(element, vNode.props);

  const children = [
    ...(vNode.props?.children || []),
    ...(vNode.children || []),
  ];

  children.forEach((child) => {
    if (child != null) {
      element.appendChild(createElement(child));
    }
  });

  return element;
}


/**
 * DOM Element의 속성을 비교하여 업데이트하는 함수
 * @param target - 업데이트할 DOM Element
 * @param newProps - 새로운 속성들
 * @param oldProps - 이전 속성들
 */
export function updateAttributes(
  element: HTMLElement,
  newProps: VNodeProps | null,
  oldProps: VNodeProps | null = null,
) {
  if (!newProps && !oldProps) return;

  // 이전 속성 제거
  if (oldProps) {
    Object.keys(oldProps).forEach((key) => {
      if (key === "children") return;

      if (key.startsWith("on")) {
        const eventType = key.substring(2).toLowerCase() as DOMEventType;
        removeEvent(element, eventType, oldProps[key]);
      } else if (!newProps || !(key in newProps)) {
        element.removeAttribute(key);
      }
    });
  }

  // 새 속성 설정
  if (newProps) {
    Object.entries(newProps).forEach(([key, value]) => {
      if (key === "children") return;

      if (key === "className") {
        if (value) element.setAttribute("class", value);
        return;
      }

      if (key.startsWith("on")) {
        const eventType = key.substring(2).toLowerCase() as DOMEventType;
        addEvent(element, eventType, value);
        return;
      }

      if (value != null && (!oldProps || oldProps[key] !== value)) {
        element.setAttribute(key, String(value));
      }
    });
  }
}
```

```jsx
// createElement 실행 후 생성되는 실제 DOM:
<div class="container">
  <header class="header">
    <h1>제목</h1>
  </header>
  <main>
    <p>내용 1</p>
    항목 1 항목 2<p>내용 2</p>
  </main>
</div>
```

### updateElement

이 부분이 가장 중요한 로직입니다. 이전 Virtual DOM 트리와 새로운 Virtual DOM 트리를 비교하는 부분입니다. 아주 간단히 구현한 것이지만 현재 재조정 과정과 다를 수 있습니다.
실제로 변경된 부분만 찾아내서 최소한의 실제 DOM 업데이트 수행하는 것을 목적으로 합니다.

```jsx
export function updateElement(parentElement, newNode, oldNode, index = 0) {
  // 노드가 모두 없는 경우
  if (!oldNode && !newNode) {
    return;
  }

  // 이전 노드만 있는 경우 (삭제)
  if (oldNode && !newNode) {
    parentElement.removeChild(parentElement.childNodes[index]);
    return;
  }

  // 새로운 노드만 있는 경우 (추가)
  if (!oldNode && newNode) {
    parentElement.appendChild(createElement(newNode));
    return;
  }

  // 둘 다 텍스트 노드인 경우
  if (typeof newNode === "string" || typeof newNode === "number") {
    if (oldNode !== newNode) {
      const newTextNode = document.createTextNode(String(newNode));
      parentElement.replaceChild(newTextNode, parentElement.childNodes[index]);
    }
    return;
  }

  // 노드 타입이 다른 경우
  if (newNode.type !== oldNode.type) {
    parentElement.replaceChild(
      createElement(newNode),
      parentElement.childNodes[index]
    );
    return;
  }
  // 속성 업데이트
  const element = parentElement.childNodes[index];
  updateAttributes(element, newNode.props, oldNode.props);

  // 자식 노드 비교 및 업데이트
  const newChildren = newNode.children || [];
  const oldChildren = oldNode.children || [];
  const maxLength = Math.max(newChildren.length, oldChildren.length);

  Array.from({ length: maxLength }).forEach((_, i) => {
    updateElement(element, newChildren[i] || null, oldChildren[i] || null, i);
  });
}
```

```tsx
/**
 * Virtual DOM의 변경사항을 실제 DOM에 반영하는 함수
 * @param parentElement - 부모 DOM Element
 * @param newNode - 새로운 Virtual Node
 * @param oldNode - 이전 Virtual Node
 * @param index - 자식 노드의 인덱스
 */
export function updateElement(
  parentElement: HTMLElement,
  newNode: VNodeChild,
  oldNode: VNodeChild,
  index: number = 0
) {
  if (!newNode || typeof newNode === "boolean") {
    // 이전 노드만 있는 경우 (삭제)
    if (oldNode != null) {
      parentElement.removeChild(parentElement.childNodes[index]);
    }
    return;
  }

  if (!oldNode || typeof oldNode === "boolean") {
    const newElement =
      typeof newNode === "object"
        ? createElement(newNode)
        : document.createTextNode(String(newNode));
    parentElement.appendChild(newElement);
    return;
  }

  // 원시 타입(string, number) 노드 처리
  if (typeof newNode !== "object") {
    const newValue = String(newNode);
    // 실질적으로 값이 바뀌지 않은 경우
    if (typeof oldNode !== "object" && String(oldNode) === newValue) {
      return;
    }
    parentElement.replaceChild(
      document.createTextNode(newValue),
      parentElement.childNodes[index]
    );
    return;
  }

  // 둘 중 하나는 VNode, 하나는 문자열인 경우
  if (typeof newNode !== typeof oldNode) {
    if (typeof newNode === "string") {
      parentElement.replaceChild(
        document.createTextNode(newNode),
        parentElement.childNodes[index]
      );
    } else {
      parentElement.replaceChild(
        createElement(newNode),
        parentElement.childNodes[index]
      );
    }
    return;
  }

  // 기존 노드가 원시 타입이면 교체
  if (typeof oldNode !== "object") {
    parentElement.replaceChild(
      createElement(newNode),
      parentElement.childNodes[index]
    );
    return;
  }

  // VNode인 경우 - 타입이 같은 경우
  if (newNode.type !== oldNode.type) {
    parentElement.replaceChild(
      createElement(newNode),
      parentElement.childNodes[index]
    );
    return;
  }

  // 속성과 자식 노드 업데이트
  const element = parentElement.childNodes[index] as HTMLElement;
  updateAttributes(element, newNode.props || {}, oldNode.props || {});

  // 자식 노드 재귀 업데이트
  const newChildren = [
    ...(newNode.props?.children || []),
    ...(newNode.children || []),
  ];
  const oldChildren = [
    ...(oldNode.props?.children || []),
    ...(oldNode.children || []),
  ];

  const maxLength = Math.max(newChildren.length, oldChildren.length);
  Array.from({ length: maxLength }).forEach((_, i) => {
    updateElement(element, newChildren[i], oldChildren[i], i);
  });
}
```

### renderElement

전체 렌더링 프로세스를 조정하는 로직입니다.

```tsx
import { VNode, VNodeChild } from "@types";
import {
  createElement,
  normalizeVNode,
  setupEventListeners,
  updateElement,
} from "@libs";

const vNodeMap = new WeakMap<HTMLElement, VNode | string>();

/**
 * Virtual DOM을 실제 DOM으로 렌더링하는 함수
 * @param vNode - 렌더링할 Virtual Node
 * @param container - 렌더링될 컨테이너 DOM Element
 */
export function renderElement(vNode: VNodeChild, container: HTMLElement) {
  const normalizedNode = normalizeVNode(vNode);
  const oldVNode = vNodeMap.get(container);

  if (!oldVNode) {
    const element = createElement(normalizedNode);
    container.appendChild(element);
  } else {
    updateElement(container, normalizedNode, oldVNode, 0);
  }

  setupEventListeners(container);
  vNodeMap.set(container, normalizedNode);
}
```

### 전반적인 흐름 정리해보면

먼저 createVNode 함수로 Virtual DOM 노드를 생성합니다. 이 함수는 HTML 구조를 JavaScript 객체 형태로 표현하는데, type(태그 이름), props(속성들), children(자식 요소들)을 받아서 하나의 객체로 만들어냅니다.

생성된 Virtual DOM 노드는 normalizeVNode 함수를 통과합니다. 이 함수는 다양한 형태의 입력값(null, 문자열, 숫자, 함수, 배열 등)을 일관된 형태로 정규화합니다. 예를 들어 조건부 렌더링으로 인한 null 값을 제거하고, 동적으로 생성되는 함수 컴포넌트를 실행하여 실제 노드로 변환합니다.

정규화된 Virtual DOM은 createElement 함수를 통해 실제 DOM 요소로 변환됩니다. 이 함수는 Virtual DOM의 type에 따라 실제 DOM 요소를 생성하고, props를 적용하며, children을 재귀적으로 처리합니다.

변경사항이 생기면 updateElement 함수가 이전 Virtual DOM과 새로운 Virtual DOM을 비교하여 실제로 변경된 부분만 찾아냅니다. 이때 노드의 추가, 삭제, 수정을 각각의 경우에 맞게 처리하며, 특히 불필요한 DOM 조작을 최소화하기 위해 같은 타입의 노드는 속성만 업데이트하고, 자식 노드들은 재귀적으로 비교합니다.

마지막으로 이 모든 과정을 renderElement 함수가 관리합니다. 이 함수는 WeakMap을 사용해 각 컨테이너의 현재 Virtual DOM 상태를 저장하고, 최초 렌더링인지 업데이트인지를 판단하여 적절한 함수를 호출합니다. WeakMap을 사용함으로써 컨테이너가 제거될 때 관련된 Virtual DOM 데이터도 자동으로 정리되어 메모리 관리가 효율적으로 이루어집니다.

## 현재 리액트에선 jsx 처리와 재조정이 어떻게 이뤄지고 있을까?

![](https://github.com/CodyMan0/bucket/blob/main/question.jpg?raw=true)

현재 사내 대시보드 프로젝트에선 Vite 기반 React 라이브러리를 사용하고 있습니다.
위에서 가상돔을 구현해보면서 Babel이라는 트랜스파일러를 활용하여 jsx 문법,
기존 Bable로 구현하는 것과 Vite는 어떤 차이가 있고 어떻게 구현되어있는지 살펴보고자 합니다

### Babel을 활용하여 jsx 처리

> Babel은 트랜스파일러로 최신 JavaScript 문법(ES6+, JSX, TypeScript 등)을 구형 브라우저에서도 실행할 수 있는 코드로 변환하는 도구입니다.

Babel은 @babel/preset-react를 사용하여 변환해야 합니다.

```js
//babel 설정 파일
{
  "presets": ["@babel/preset-react"]
}

// Input: JSX 코드
const element = <h1>Hello, world!</h1>

// Babel Output:
;('use strict')
var _react = require('react')
var element = /*#__PURE__*/ _react.createElement('h1', null, 'Hello, world!')
```

리액트 17버전 이상부터는 react/jsx-runtime과 react/jsx-dev-runtime 패키지의 jsx,jsxs,jsxDev를 사용했네요? 이제 알게 됐네요.

#### Before v17

```jsx
function App() {
  return <div>My App</div>;
}

export default App;
```

```jsx
// 위에서 vanilla javascript로 구현한 createElement와 비슷한 인터페이스로 보입니다.
function App() {
  return React.createElement("div", {
    children: "My App",
  });
}
```

#### after v17

```jsx
function App() {
  return <div>My App</div>;
}

export default App;
```

```jsx
// 배포 환경
import { jsx as _jsx } from "react/jsx-runtime";
function App() {
  return /*#__PURE__*/ _jsx("div", {
    children: "My App",
  });
}
export default App;

// 개발 환경
function App() {
  return /*#__PURE__*/ (0,
  react_jsx_dev_runtime__WEBPACK_IMPORTED_MODULE_0__.jsxDEV)(
    "div",
    {
      children: "My App",
    },
    void 0,
    false,
    {
      fileName: _jsxFileName,
      lineNumber: 3,
      columnNumber: 5,
    },
    this
  );
}
```

이로 인해 어떤 이점이 있는 것일까?

1. 개발을 처음 했을 땐 react 내에서 React 모듈을 가져와야했는데 어느 순간 그러지 않아도 됐는데 이 이유였다.
2. 번들 사이드가 작아졌다.

Babel을 활용해서 구현하는데 있어 before17을 사용하고 싶다면

```
//babel 설정 파일
{

  "presets": [
    ["@babel/preset-react", { "runtime": "classic" }] // React.createElement()
    ["@babel/preset-react", { "runtime": "automatic" }] // jsx() || jsxs() || jsxDev()
  ]
}
```

### Vite 에선 어떻게 설정되어 있길래?

```tsx
import react from '@vitejs/plugin-react'
...

const viteConfig = defineConfig({
  plugins: [react(), svgr()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})

const vitestConfig = defineTestConfig({
  test: {
    globals: true, // 전역 expect, describe 등 사용
    environment: 'jsdom', // DOM 환경 설정
    setupFiles: './src/setupTests.ts', // 전역 설정 파일 경로 지정
  },
})

export default mergeConfig(viteConfig, vitestConfig)
```

> `@vitejs/plugin-react` 플러그인의 기본 세팅은 어떻게 되어있을까?

[@vitejs/plugin-react Npm 웹사이트](https://www.npmjs.com/package/@vitejs/plugin-react)를 살펴보니 아래와 같은 설명이 있습니다.

> The default Vite plugin for React projects.

- enable Fast Refresh in development (requires react >= 16.9)
- use the automatic JSX runtime
- use custom Babel plugins/presets
- small installation size

```tsx
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
});
```

실제 소스 코드를 살펴보니

```tsx
const viteBabel: Plugin = {
  name: 'vite:react-babel',
  enforce: 'pre',
  config() {
      if (opts.jsxRuntime === 'classic') {
        return {
          esbuild: {
            jsx: 'transform',
          },
        }
      } else {
        return {
          esbuild: {
            jsx: 'automatic',
            jsxImportSource: opts.jsxImportSource,
        },
        optimizeDeps: { esbuildOptions: { jsx: 'automatic' } },
      }
    }
  },
  configResolved(config) {...},
  async transform(code, id, options) {...},
}
```

이 코드를 통해 위에서 알아보았던 일련의 과정이 추상화된 것이었구나를 깨달았습니다.

# 결론

CRA는 명을 다했고 Vite를 통해 리액트를 이렇게 쉽게 사용하는 것이 얼마나 감사한지 깨달았습니다. 만약 React가 없다면?? 을 기억해 보며 없을 때도 살아남을 수 있도록 더 깊이 있게 학습하려고 합니다.

최근 들어, 사내에서 백엔드 지식을 익혀야 기여할 수 있는 상황이 생겨 백엔드를 하면서 잊고 있던 리액트를 다시 복습하기 위해 정리해 보았습니다.

---

현재 항해 플러스 5기 모집 중입니다 :)
https://hanghae99.spartacodingclub.kr/plus/fe

(추천인 코드 0BBFXj 를 입력하시면 수강료를 20만원 할인받을 수 있습니다)

관련 질문은 링크드인을 통해 부탁드립니다!

# 참고 자료

1. [가상돔 블로그](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Virtual-DOM/#_1-%E1%84%80%E1%85%A1%E1%84%89%E1%85%A1%E1%86%BC%E1%84%83%E1%85%A9%E1%86%B7-virtualdom-%E1%84%86%E1%85%A1%E1%86%AB%E1%84%83%E1%85%B3%E1%86%AF%E1%84%80%E1%85%B5)
2. [리액트 공식문서](https://react.dev/reference/react/createElement)
3. [리액트 17버전 이상 jsx 처리](https://medium.com/@juliaazt/react-js-deep-dive-1-createelement-and-jsx-runtime-63c75882f7b0)
