---
title: "FSD 적용기 1편 [항해 플러스 회고]"
date: "2025-03-14"
lastmod: "2025-03-14"
tags: ["항해솔직후기", "항해플러스프론트엔드", "항해플러스후기", "회고"]
draft: false
summary: "FSD를 사내에 적용해본 경험을 공유합니다."
---

![](https://github.com/CodyMan0/bucket/blob/main/fsd-refacts.png?raw=true)
<TOCInline toc={props.toc} />

# 서론

> 기획에 따라 변하는 것들을 모아두면 좋은가?

항해 플러스에서 진행했던 클린코드와 함수형 프로그래밍 섹션을 다시 읽어보며 최근 사내 프로젝트에 FSD를 적용하는 과정을 정리해보고자 합니다.
시간을 쪼개 조금씩 리팩터링을 진행했으며 느낀 점을 공유하고자 합니다. FSD 리팩터링 시리즈는 3편으로 나눠 진행할 예정입니다.

> 해당 포스트에서는 환경 세팅과 기본적인 public api rules와 slice and segment rules를 해결하며 고민했던 것들을 공유하고자 합니다.

> 결론적으로 FSD를 적용하면서 느낀 점은

1. 혼자 하는 프로젝트임에도 나부터 FSD의 규칙을 지키지는게 어렵다는 것. 팀원 모두 의지가 있다면 문제없겠지만 지쳐있는 팀원이 있다면 쉽지 않을 수도?

2. FSD를 적용하려다 보니 백엔드에 대한 이해와 DB 구조를 알아야 함을 느꼈다. 그래서 이번 기회에 사내 프로젝트 백엔드 딥다이브를 진행했다. 그에 따라 entities 폴더 하위에 잘 정리할 수 있었다. 도메인 지식을 조금 더 알면 도움이 되는 것 같다.

3. FSD를 잘 모르겠다면 불편해도 강제하는 방법을 도입해서 습관을 들이는 시간이 필요한 것 같다. 우선 기본부터 알고 유연하게 가면 좋겠다고 생각했다.

# 본론

무작정 리팩터링 하기엔,,, 불안했습니다. 리팩터링에 테스트 코드가 필요하듯 폴더 구조와 관련돼서 환경을 설정해야합니다.

> 24년 11월부터 프런트엔드 파트를 혼자 진행한 프로젝트에 적용해 보았습니다. 이 프로젝트를 시작하는 시점에 FSD를 사용해 본 적이 없으며, FSD를 통해 유지보수가 쉬운 프런트엔드를 만들고자 도입하게 됐습니다. 하지만 빠르게 기능과 UI을 만들어야 하는 상황에서 FSD를 엄격하게 지키며 프로젝트를 구현하는 것이 어려웠습니다. 그래서 항해 플러스를 통해 학습한 FSD와 함수형 프로그래밍 지식을 녹여 리팩터링을 진행하였습니다. 큰 개요는 다음과 같습니다.

1. prettier, eslint 적용
2. steiger 라이브러리 적용
3. 우선 slice, segement 모든 파일 적용
4. public api 모든 디렉터리 적용

## prettier, eslint부터 적용하자

가장 기본적으로, 클린 코드에선 일관성이 중요하기에 prettier와 eslint 룰부터 추가하려고 합니다. 또한 프로젝트 파일이 늘어날 것을 대비하여 lint-staged를 활용하여 git add .를 통해 staging 된 파일에 한해서만 prettier과 eslint를 실행하도록 구현했습니다. 또한 husky를 활용하여 git hook을 이용하여 커밋이 발생할 때 lint-staged를 실행하도록 구현했습니다.

package.json에 lint-staged를 추가해주고 자바스크립트와 타입 스크립트 파일 모두 검사하도록 작성하였습니다.

- **husky 도입**

```script
//.husky/pre-commit
npm run lint-staged
```

- **lint-staged 도입**

```json
//package.json
{
  "name" : "vision-system",
  ...
  "scripts": {
    "dev": "vite --host 0.0.0.0",
    "build": "vite build --debug",
    "preview": "vite preview --port 8080",
    "prettier": "prettier --write .",
    "test": "vitest",
    "prepare": "husky",
    "lint-staged": "lint-staged",
  },
  "lint-staged": {
    "*.{js,ts,jsx,tsx}": [
      "eslint --fix",
      "prettier --write"
    ]
  },
}

```

이전 프로젝트를 진행하면서 프로젝트가 커질수록 커밋되는 속도가 늦어지는 것을 보며 알게 된 lint-staged 패키지입니다.

- **FSD에 필요한 prettier 도입**

```json
//.prettierrc
{
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "auto",
  "plugins": [
    "prettier-plugin-tailwindcss",
    "@trivago/prettier-plugin-sort-imports"
  ],
  "importOrder": [
    "<THIRD_PARTY_MODULES>",
    "^@/app/(.*)$",
    "^@/pages/(.*)$",
    "^@/widgets/(.*)$",
    "^@/features/(.*)$",
    "^@/entities/(.*)$",
    "^@/shared/(.*)$",
    "^@/components/(.*)$", //리팩터링 당시 components 디렉터리 존재
    "^@/hooks/(.*)$", //리팩터링 당시 hooks 디렉터리 존재
    "^[./]"
  ],
  "tailwindConfig": "./tailwind.config.mjs",
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true
}
```

두 가지 플러그인을 활용하였습니다.

1. `prettier-plugin-tailwindcss`

2. `@trivago/prettier-plugin-sort-imports`

tailwind를 활용하여 스타일링을 진행하고 있습니다. 이번 리팩터링을 통해 저장을 누르면 일관된 순서대로 작성되도록 하기 위해 1번을,
import문을 자동으로 정리하기 위해 2번을 사용했습니다. 트리바고 플러그인이 가장 잘 정돈되고 fsd에 맞게 커스텀할 수 있다고 판단돼 채택하였습니다.

importOrder 키의 값으로 원하는 순서를 지정하였습니다.

fsd에서는 각 레이어에서 import 할 수 있는 레이어에 제한이 있습니다. 아래의 그림과 같습니다.

그래서 이 규칙에 근거하여 정리할 수 있죠. 모듈이 정리되면 아래와 같습니다.

```tsx
// "<THIRD_PARTY_MODULES>",
import { useSuspenseQuery } from '@tanstack/react-query';
import { ColumnDef } from '@tanstack/react-table';
import dayjs from 'dayjs';
import { SearchIcon } from 'lucide-react';
import { useMemo } from 'react';
import { Link } from 'react-router-dom';

//"^@/shared/(.*)$",
import { ActiveAccountSnapshot, SnapShotQueries } from '@/entities/accountSnapshots';
import { OrderDto } from '@/entities/orders';
import { PositionDto } from '@/entities/positions';

//"^@/entities/(.*)$",
import {
  ASSET_SETTINGS,
  DATE_FORMAT_DD_MM_mm_YYYY_WITH_DOT_IN_KOR,
  SYMBOL_SEETINGS,
  floorToDecimalPlacesWith,
  privatePathKeys,
} from '@/shared/lib';

// "^[./]"
import { parseSymbolForClient, useActivceSnapshotFilter } from '../lib';
import { DataTableDemoWithoutPagination } from './DataTableDemoWithoutPagination';
import EmptyCell from './EmptyCell';
import OrderTooltip from './OrderTooltip';
import PositionTooltip from './PositionTooltip';
import SymbolSelect from './SymbolSelect';

const RiskTable = () => {
  const { data: activeAccountSnapshots } = useSuspenseQuery(
    SnapShotQueries.getActiveAccountsSnapshots(),
  );
  return (
    ...
  )
}
```

```json
  "importOrderSeparation": true,
```

이것을 통해 각 order에 스페이스바를 추가하여 가독성을 높여보았습니다.

- **FSD에 필요한 eslint 도입**

eslint는 9.18.0 버젼을 사용하고 있고 flat config 문법을 사용하여 eslint를 설정해주었다.

```js
//eslint.config.js

import js from "@eslint/js";
import prettierConfig from "eslint-config-prettier";
import prettierPlugin from "eslint-plugin-prettier";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import globals from "globals";
import tseslint from "typescript-eslint";

/** @type {import("eslint").Linter.FlatConfig[]} */
export default [
  js.configs.recommended,
  ...tseslint.configs.recommended, // TypeScript ESLint 설정 추가
  prettierConfig, // Prettier 설정 추가
  {
    files: ["**/*.{ts,tsx}"],
    languageOptions: {
      ecmaVersion: 2020,
      globals: globals.browser,
    },
    plugins: {
      react,
      "react-hooks": reactHooks,
      "react-refresh": reactRefresh,
      prettier: prettierPlugin,
    },
    rules: {
      ...reactHooks.configs.recommended.rules,
      "react-refresh/only-export-components": [
        "warn",
        { allowConstantExport: true },
      ],
      "prettier/prettier": ["error", { endOfLine: "auto" }],
    },
  },
  {
    ignores: ["dist", "node_modules"],
  },
];
```

eslint는 FSD와 관련된 룰을 추가하진 않았고 기본적으로 지켜야하는 룰을 우선 추가하였습니다.

- **FSD를 강제하기 위해 steiger 패키지 도입**

공식문서를 통해 steiger 패키지를 알게 됐습니다.
steiger은 프로젝트의 폴더 아키택쳐를 확인하는 린터입니다. 현재 개발 중인 패키지여서 완전히 신뢰하기는 어려우나 FSD를 강제하고 싶을 때 도움 받을 수 있습니다.

```zsh
// 설치
npm i -D steiger

//1회 실행
npx steiger ./src

//감시모드 실행 (폴더 구조 및 모듈 이름이 변경되는 트리거됨)
npx steiger ./src --watch
```

![](https://github.com/CodyMan0/bucket/blob/main/fsd-error.png?raw=true)
![](https://github.com/CodyMan0/bucket/blob/main/fsd-error-2.png?raw=true)

위와 같이 자세하게 프로젝트 디렉터리 구조에서 규칙을 위반하고 있는 것이 있는지 알려줍니다.

> 이 에러를 메시지를 하나씩 해결해 가며 개발 서버를 띄어놓고 정리하기 시작했습니다.

## 불필요한 폴더 삭제 및 slice, segement 모든 파일 적용

steiger를 통해 확인해 본 결과, 수정하기 전 48개의 위반 에러가 있었습니다. 그래서 하나씩 수정해 본 결과 하나 수정하면 3개의 에러가 추가되는 것을 보며 우선 기본적으로 가장 많이 눈에 보인 두가지가 있습니다.

> **1. 레이어에 따른 Slices와 Segments 규칙**

> **2. Public API 규칙**

두 가지에 집중해서 먼저 수정 후, steiger의 에러를 잡아가는 방향으로 리팩터링을 진행해보았습니다.

우선 FSD의 두 가지 규칙에 대해 정리해보고자 합니다.

### 레이어에 따른 Slices와 Segments

![](https://feature-sliced.design/assets/ideal-img/visual_schema.b6c18f6.1030.jpg)

위의 그림은 공식문서를 통해 확인할 수 있었죠. 즉 각 레이어 (app > pages > widgets > features > entities > shared) 하위에 슬라이스가 반드시 존재해야 한다고 steiger (마이그레이션 도구)가 알려줍니다.

> Slices란?

Slices란 도메인별로 코드를 분할한 디렉터리를 뜻합니다. 해당 프로젝트를 예로 들면 fast api로 구현된 백엔드 코드의 Model 소스 코드를 살펴보니 도메인이 무엇인지 바로 알 수 있었습니다. 이에 대응하는 슬라이스 디렉터리를 만들면 됩니다.

![](https://github.com/CodyMan0/bucket/blob/main/fsd-model.png?raw=true)
(백엔드 모델 일부)

위와 프론트 슬라이스를 비슷하게 가져가면 됩니다.

![](https://github.com/CodyMan0/bucket/blob/main/fsd-front.png?raw=true)

단! App과 Shared 레이어는 슬라이스가 필요 없습니다. 다른 레이어는 복수인데 이 둘만 단수라는 것을 보면 유추할 수 있습니다. 바로 Segments를 필요로 합니다.

> Segments란?

Segment란 목적에 따라 코드를 그룹화한 디렉터리를 의미합니다. 일반적으로 ui, api, model, lib, config와 같은 것들이 있습니다

- ui : 해당 슬라이스 혹은 전역적으로 사용할 UI
- api : 백엔드와 통신 로직
- model : 데이터 모델 : 스키마, 인터페이스, 스토어, 비즈니스 로직
- lib : 슬라이스 안에 있는 다른 모듈이 필요로 하는 라이브러리 코드
- config : 설정 파일과 기능 플래그

프런트엔드 모든 레이어에 Slice와 Segment를 추가하였습니다. page는 도메인과 100퍼센트 동일하지 않지만 도메인과 연관하여 이름을 지어 페이지별로 slices 디렉터리를 만들었습니다. 하지만 여기서 궁금한 것은

> features과 widgets는 slice 이름을 어떻게 짓어야하는지 입니다. 동사형태로 만들어야하는 것 같긴 한데... slice는 도메인별 코드를 분할한 디렉터리라고 하면, entities와 동일하게 만들어야 하는 것인지에 대해 고민이 있었고 우선 동일하게 만들었습니다. 다른 분들은 어떻게 하시는지 궁금합니다.

AS-IS

```json
📦src
┣ 📂app
┣ 📂components //💥HERE component 폴더를 제거하자, 이곳에 있는 것은 아토믹한 컴포넌트는 아니고 조합된 것이라 우선 components를 만들었음.
┣ 📂entities // 💥 HERE entities를 정확하게 작성하지 못함. entities가 뭐지?
┃ ┣ 📂account
┃ ┣ 📂auth
┃ ┣ 📂common
┃ ┣ 📂organization
┃ ┗ 📂user
┣ 📂features // 💥 HERE feature에 모달이 있다?? 이유는 존재.
┃ ┣ 📂accounts
┃ ┣ 📂auth
┃ ┃ ┣ 📂login
┃ ┃ ┣ 📂logout
┃ ┃ ┗ 📂sign-up
┃ ┣ 📂change-password
┃ ┣ 📂download
┃ ┣ 📂modal
┃ ┃ ┣ 📂accounts
┃ ┃ ┣ 📂users
┃ ┃ ┣ 📜hasExistedModal.tsx
┃ ┃ ┣ 📜index.ts
┃ ┃ ┗ 📜modal.ui.tsx
┃ ┣ 📂organization
┃ ┣ 📂records
┃ ┣ 📂settings
┃ ┃ ┗ 📂update-settings
┃ ┗ 📂user
┃ ┃ ┣ 📂delete-user
┃ ┃ ┗ 📂register-user
┣ 📂pages // 💥 HERE 페이지에 재활용되는 Form이 있어서 만들었었음.
┃ ┣ 📂Form
┃ ┣ 📂UiElements
┃ ┣ 📂accounts
┃ ┣ 📂approval-pending
┃ ┣ 📂auth
┃ ┃ ┣ 📂login
┃ ┃ ┣ 📂sign-up
┃ ┃ ┃ ┗ 📂ui
┃ ┃ ┗ 📂signin
┃ ┣ 📂dashboard
┃ ┣ 📂home
┃ ┣ 📂landing
┃ ┣ 📂layout
┃ ┣ 📂organizations
┃ ┣ 📂page-404
┃ ┣ 📂pnl-tracking
┃ ┣ 📂risk-management
┃ ┣ 📂settings
┃ ┣ 📂statistic
┃ ┣ 📂transfer-records
┃ ┣ 📂users
┣ 📂shared
┃ ┣ 📂api // 💥 HERE 백엔드와 통신하는 API는 shared에 있는게 맞는가?
┃ ┃ ┣ 📂account
┃ ┃ ┃ ┣ 📜account-service.ts
┃ ┃ ┃ ┣ 📜account.model.tsx
┃ ┃ ┃ ┣ 📜account.types.ts
┃ ┃ ┃ ┗ 📜index.ts
┃ ┃ ┣ 📂admin
┃ ┃ ┣ 📂auth
┃ ┃ ┣ 📂dashboard
┃ ┃ ┣ 📂management
┃ ┃ ┗ 📂market
┃ ┣ 📂hooks // 💥 HERE 공통 훅도 아닌데 shared에 있을 필요가 있나? 어디로 보내야하지?
┃ ┃ ┣ 📜useCharts.tsx
┃ ┃ ┣ 📜useHistoricalAum.ts
┃ ┃ ┣ 📜useMobile.ts
┃ ┃ ┗ 📜useSnapShotAum.ts
┃ ┣ 📂lib // 💥 HERE 모듈을 index.ts를 활용하여 export하고 있지 않는데?
┃ ┃ ┣ 📂axios
┃ ┃ ┣ 📂zustand
┃ ┃ ┣ 📜business.tsx
┃ ┃ ┣ 📜chart.ts
┃ ┃ ┣ 📜date.ts
┃ ┃ ┣ 📜env.ts
┃ ┃ ┣ 📜globalMessage.ts
┃ ┃ ┣ 📜mswUtils.ts
┃ ┃ ┣ 📜policy.ts
┃ ┃ ┣ 📜stroageKeys.ts
┃ ┃ ┗ 📜utils.ts
┃ ┣ 📂types // 💥 HERE 타입들이 shared에 있을 필요가 있나? 도메인과 관련된것인데?
┃ ┃ ┣ 📜users.ts
┃ ┃ ┣ 📜positions.ts
┃ ┃ ┣ 📜accounts.ts
┃ ┃ ┣ 📜package.ts
┃ ┃ ┣ 📜policy.ts
┃ ┃ ┣ 📜product.ts
┃ ┃ ┗ 📜response.ts
┃ ┗ 📂ui
┃ ┃ ┣ 📜...
┣ 📂widgets
┃ ┃ ┣ 📜...
```

AS-IS의 문제들...

1. 💥 Component 폴더를 제거. 이곳에 있는 것은 아토믹한 컴포넌트는 아니고 조합된 것이라 우선 components를 만들었음.
2. 💥 entities를 명확하게 작성하지 못함.
3. 💥 feature에 모달이 있다. 아직 FSD에 익숙하지 못함.
4. 💥 페이지에 재활용되는 Form이 있어서 Pages에 만들었음.
5. 💥 백엔드와 통신하는 API는 shared에 있는게 맞는가?
6. 💥 공통 훅도 아닌데 shared에 있을 필요가 있나? 어디로 보내야하지?
7. 💥 모듈을 index.ts를 활용하여 export하고 있지 않는데?
8. 💥 타입들이 shared에 있을 필요가 있나? 도메인과 관련된것인데?

리팩터링을 하면서 느낀 점은 내가 만든 프로그램이지만 조금만 방심하면 이해할 수 없을 정도로 쉽게 지저분해지는 것을 경험했습니다. 이젠 eslint와 prettier 그리고 코드에 이유를 적용해 보면서 구현하고자 합니다.

TO-BE

```json
📦src
 ┣ 📂__mock__
 ┣ 📂__test__
 ┣ 📂app
 ┣ 📂entities
 ┃ ┣ 📂accounts
 ┃ ┃ ┣ 📂model
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂auth
 ┃ ┃ ┣ 📂lib
 ┃ ┃ ┣ 📂model
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂organizations
 ┃ ┃ ┣ 📂lib
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┗ 📜organization.lib.ts
 ┃ ┃ ┣ 📂model
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┗ 📜organization.queries.ts
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂...
 ┃ ┃ ┗ 📜index.ts
 ┣ 📂features
 ┃ ┣ 📂...
 ┃ ┃ ┗ 📜index.ts
 ┣ 📂pages
┃ ┣ 📂...
 ┃ ┃ ┗ 📜index.ts
 ┣ 📂shared
 ┃ ┣ 📂lib
 ┃ ┃ ┣ 📂aum
 ┃ ┃ ┣ 📂axios
 ┃ ┃ ┣ 📂constants
 ┃ ┃ ┣ 📂hooks
 ┃ ┃ ┣ 📂math
 ┃ ┃ ┣ 📂react
 ┃ ┃ ┣ 📂react-router
 ┃ ┃ ┣ 📂tailwind-css
 ┃ ┃ ┣ 📂tanstack-query
 ┃ ┃ ┣ 📂zustand
 ┃ ┣ 📂ui
┃ ┃ ┗ 📜...
 ┃ ┗ 📜index.ts
 ┣ 📂widgets
 ┃ ┣ 📂calender
 ┃ ┣ 📂chart
 ┃ ┃ ┣ 📂model
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂common
 ┃ ┃ ┣ 📂lib
 ┃ ┃ ┃ ┗ 📜index.ts
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂setting
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┗ 📜setting-section.ui.tsx
 ┃ ┗ 📂sidebar
```

TO-BE 해결

1. 😊 component 폴더를 제거.

- Header와 같은 재활용되는 컴포넌트는 widgets > common > ui 로 위치하였습니다.
- 나머지 컴포넌트는 shared > ui로 이동시켰습니다.

2. 😊 entities를 정확하게 작성하지 않았던 것.

- 프로그램 전반적으로 어떤 도메인이 존재하고 어떻게 상호작용하는지 백엔드 소스코드를 통해 학습하여 분리하였습니다.

3. 😊 features에 모달 슬라이스가 있다?

features 내부 슬라이스로썬 modal이 이상합니다. 하지만 orgs, accounts, users에 각각 CRUD를 담당하는 modal이 있어 모달로 slice를 만들고 하위에 각 도메인과 관련된 파일을 위치했습니다.

![](https://github.com/CodyMan0/bucket/blob/main/fsd-domain.png?raw=true)

추상화된 modal 컴포넌트는 shared > ui로 이동시키고 각 도메인의 modal 컴포넌트는 slice > ui 세그먼트로 이동시켰습니다.

AS-IS

```json
 ┣ 📂features
 ┃ ┣ 📂modal
 ┃ ┃ ┣ 📂accounts
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┣ 📜modify-accounts-modal.ui.tsx
 ┃ ┃ ┃ ┣ 📜modify-transfer-modal.ui.tsx
 ┃ ┃ ┃ ┣ 📜register-accounts-modal.ui.tsx
 ┃ ┃ ┃ ┗ 📜transfer-modal.ui.tsx
 ┃ ┃ ┣ 📂users
 ┃ ┃ ┃ ┣ 📜delete-user-modal-ui.tsx
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┗ 📜register-user-modal-ui.tsx
 ┃ ┃ ┣ 📜hasExistedModal.tsx
 ┃ ┃ ┣ 📜index.ts
```

steiger에서는 슬라이스 내부에 세그먼트 (ui, api, model, lib, config가 )가 아닐 경우 에러를 발생시킵니다.

TO-BE

```json
 ┣ 📂features
 ┃ ┣ 📂accounts
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┣ 📜modify-accounts-modal.ui.tsx
 ┃ ┃ ┃ ┗ 📜register-accounts-modal.ui.tsx
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂organizations
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┣ 📜modify-organization-modal.ui.tsx
 ┃ ┃ ┃ ┗ 📜register-organization-modal.ui.tsx
 ┃ ┃ ┗ 📜index.ts
 ┃ ┣ 📂records
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┣ 📜modify-transfer-modal.ui.tsx
 ┃ ┃ ┃ ┗ 📜register-transfer-modal.ui.tsx
 ┃ ┃ ┗ 📜index.ts
 ┃ ┗ 📂users
 ┃ ┃ ┣ 📂ui
 ┃ ┃ ┃ ┣ 📜index.ts
 ┃ ┃ ┃ ┣ 📜delete-user-modal-ui.tsx
 ┃ ┃ ┃ ┗ 📜register-user-modal-ui.tsx
 ┃ ┃ ┗ 📜index.ts
```

도메인 별로 분리가 되어져있으니 확실히 가독성이 좋아지는 것 같습니다. 하지만 그렇게 효과적인 방법인지는 조금 더 지켜봐야할 것 같습니다.

4. 😊 API는 모두 share에 위치하는 것이 맞는가?

share > api에 있는 모든 코드들을 각 도메인에 맞게 이동시켰습니다. share -> entities로 이동시켜 가독성을 높였습니다.

5. 😊 공통 훅도 아닌데 shared에 있을 필요가 있나?
   share > hooks 있는 모든 코드들을 각 도메인에 맞게 이동시켰습니다. share -> features로 이동시켜 가독성을 높였습니다.

6. 😊 index.ts로 export하고 있지 않는데?

shared를 제외한 모든 레이어에 index.ts를 통해 공개할 모듈만 포함하여 export하였습니다. steiger의 도움을 많이 받았습니다.

7. 😊 types가 여기 있을 필요가 있나?
   share > types에 있는 모든 코드들을 각 도메인에 맞게 이동시켰습니다. share -> entities로 이동시켜 가독성을 높였습니다.

# 결론

FSD 아키택쳐를 기반으로 아주 기본적인 부분부터 리팩터링을 진행해 보았습니다. 이를 통해 해당 프로젝트에 대한 이해가 깊어졌습니다. 도메인들이 어떤 것들이 있는지, 각 도메인끼리 어떤 연관 관계가 있는지, 중복되고 있는 API가 어떤 것들이 있는지에 알게 됐고

그 결과 매 5분 주기로, 96번 호출된 결과를 통해 그렸던 화면을 1번의 API 호출로 가능할 수 있도록 API 설계를 건의하여 개선하였습니다.

> FSD 리팩터링 2편에서는 조금 더 상세한 부분에 대해 다루려고 합니다.

features는 어떻게 나누었는지, widgets는 구성은 어떻게 하면 좋을지에 대한 고민을 공유하려고 합니다. 정답이 정해져 있지 않아 고민스러운 부분이 많습니다. 이런 고민을 하면 성장하는 것이겠죠??

> 모두 각자의 위치에서 파이팅입니다.

---

항해 플러스 5기 모집 중입니다 :)

https://hanghae99.spartacodingclub.kr/plus/fe

(추천인 코드 0BBFXj 를 입력하시면 수강료를 20만원 할인받을 수 있습니다)

관련 질문은 링크드인을 통해 부탁드립니다!

# 참고 자료

1. [가상 쉬운 웹개발 with Boaz FSD편](https://www.youtube.com/watch?v=xIeUF8dsjpo&t=1065s)
