---
title: "React, Beyond the basic (항해 플러스 프론트엔드 3주차 회고)"

date: "2025-01-06"
lastmod: "2024-01-07"
tags: ["항해솔직후기", "항해플러스프론트엔드", "항해플러스후기"]
draft: false
summary: "React, Beyond the basic"
---

<TOCInline toc={props.toc} exclude="Introduction" />

> 12월 14일부로 오늘까지 3주간 SPA, VDOM 그리고 hooks를 직접 구현해 보면서 지식으로만 알고 있던 것들을 눈으로 확인하시는 시간이었습니다. 이번주의 목표는 다음과 같았습니다.

1. `Hooks를 활용하여 상태 관리와 부수효과를 효율적으로 처리할 수 있다.`
2. `메모이제이션을 이용하여 불필요한 연산을 최소화할 수 있다.`
3. `성능 프로파일링을 통해 애플리케이션의 성능을 최적화할 수 있다.`

현업에서 기능을 구현하면서 리액트의 동작 원리를 알아야 해결할 수 있는 문제는 만나보진 못했지만, 동작 방식을 이해하면 보다 효율적인 코드를 작성할 수 있고 재사용성을 높일 수 있다고 생각합니다. 그래서 React 동작 원리를 이해해야 합니다.

## 본론

---

## 발제 과제

이번 주는 React의 기본 hooks를 직접 구현해야 했습니다.

코드는 여기서 확인하실 수 있어요! 👉🏼👉🏼 [깃헙 레포](https://github.com/CodyMan0/front_4th_chapter1-3/tree/main)

![](/static/images/3.1.png)

위의 함수를 구현하면서 추상적으로 알고 있던 개념들을 정리해볼 수 있었습니다. 하나씩 살펴보시죠.

### 1. 기본 비교, 얕은 비교, 깊은 비교, 참조 비교

React에서는 props를 비교할 때 Object.is() 매소드가 사용됩니다. 아래와 같은 전략으로 사용됩니다.

![](/static/images/3.2.png)
[참고자료](https://github.com/hanghae-plus/front_4th_chapter1-3/pull/59)

과제를 진행하면서 우선적으로 shallowEquals와 deepEquals를 구현해야 했습니다. 각 비교의 차이를 명확히 알 수 있었습니다. 정리해보면 아래와 같습니다.

#### 얕은 비교(Shallow Equal)

- 원시 값은 값 자체를 비교
- 참조 값은 참조(메모리 주소)만 비교
  - 객체의 키와 값이 같아도, 메모리 주소가 다르면 다른 객체로 간주
  - 비교는 1단계(1depth)에서 멈춘다
    - 내부 중첩 구조는 확인하지 않음

#### 깊은 비교(Deep Equal)

- 객체나 배열의 모든 중첩된 구조를 순회하며 값 자체가 같은지 비교하는 방식
- 중첩된 객체와 배열을 재귀적으로 순회하며 값을 비교

#### 참조 값 비교

- 객체나 배열의 모든 중첩된 구조를 순회하며 값 자체가 같은지 비교하는 방식
- 객체 내부가 동일해도 다른 메모리 주소!!!!

```tsx
{name: 'juyoung'} === {name: 'juyoung'} /// false
const human = {name: 'juyoung'}; console.log(human === human)  ///  true
```

#### 얕은 비교와 === (기본 비교)의 차이

- 얕은 비교(shallowEqual)와 `===`(기본 비교)는 비교 방식, 사용 용도가 다름

![](/static/images/3.6.png)
[참고자료](https://github.com/hanghae-plus/front_4th_chapter1-3/pull/59)

### useMemo, useCallback 직접 구현

#### useMemo 구현

useMemo를 활용한 경험이 많지 않습니다. 실무에서 그래프를 렌더링할 때, 총합을 보여주는 상태는 useMemo를 사용하여 데이터 변경에 의존하도록 구현한 적은 있지만 실제 성능 최적화 경험은 없어서 이번 기회에 보다 정확히 알길 원했습니다. useMemo는 말 그대로 값을 메모이제이션하는 것입니다. 계산 비용이 큰 값의 재계산 방지하거나 불필요한 리렌더링을 최적화할 때 사용됩니다.

```tsx
import { DependencyList } from "react";
import { shallowEquals } from "../equalities";
import { useRef } from "./useRef";

// useMemo 훅은 계산 비용이 높은 값을 메모이제이션합니다.
export function useMemo<T>(
  factory: () => T,
  _deps: DependencyList,
  _equals = shallowEquals
): T {
  const memoizedValue = useRef<T | null>(null);
  const prevDeps = useRef(_deps);

  if (memoizedValue.current === null || !_equals(prevDeps.current, _deps)) {
    memoizedValue.current = factory();
    prevDeps.current = _deps;
  }

  return memoizedValue.current;
}
```

#### useCallback 구현

그럼 useCallback은 무엇인가요. 코드를 통해 더욱 이해할 수 있습니다. 결국 useMemo와 동일한데 위에서 구현한 useMemo를 이용해 구현하였습니다. callback 함수와 deps를 useMemo에 인자로 넘깁니다.

```tsx
/* eslint-disable @typescript-eslint/no-unsafe-function-type */
import { DependencyList } from "react";
import { useMemo } from "./useMemo";

export function useCallback<T extends Function>(
  factory: T,
  _deps: DependencyList
) {
  // 직접 작성한 useMemo를 통해서 만들어보세요.
  const memoizedFn = useMemo(() => factory, _deps);
  return memoizedFn;
}
```

## 실무에서 어떻게 적용했는가?

위에서 학습한 기본기를 활용하여 문제를 해결했습니다.

```tsx
type 계좌 = {
  assetName: AssetType
  exchangeName: ExchangeType
  equity: Equity[]
  ...
}

type Equity = {
  updateTimeStamp: number
  equity : number
  ...
}

가공할 자료 구조 타입
const accounts:계좌[] = ...
```

간단히 위의 타입의 데이터 타입을 가공해야 하는 상황이었습니다. 코드 맥락을 간단히 정리해보면 대시 보드 내부에 고객 계좌가 여러 개 있고 그 계좌를 순회돌면서 그 내부의 equity 내부를 한번 더 순회돕니다. 그리고 날짜에 맞는 equity를 각 assetName 프로퍼티에 맞는 위치에 넣어줘야하는 상황이었습니다.

```tsx
// AS-IS
const manifestHistoricalMapper = (
  account: DashboardListGetDto,
  mapper: Map<string, HistoricalAumType>,
  marketInfos: CommonDTO.TransfromMarketDto[],
  currency: AssetFilteredType
) => {
  const { dailies, exchangeName, assetName } = account

  dailies.forEach((daily) => {
    const defaultEquity = {
      btc: 0,
      eth: 0,
      usdc: 0,
      usdt: 0,
    }

    const defaultExchange: HistoricalAumType['equitiesOfExchange'] = {
      binance: defaultEquity,
      okx: defaultEquity,
      bybit: defaultEquity,
    }
    ...
  })
}
```

객체를 변수에 초기화한 후, 해당 변수를 각각의 거래소의 value로 초기화했습니다. 무슨 문제가 발생했을까요?

위에서 학습한대로 변수에 객체를 초기화할 경우, 동일한 주소값을 가지게 됩니다.

![](/static/images/3.7.png)

그래서 위의 이미지와 같이 각 거래소의 값이 덮어씌어지는 문제가 발생했습니다.

```tsx
// TO-BE
const manifestHistoricalMapper = (
  account: DashboardListGetDto,
  mapper: Map<string, HistoricalAumType>,
  marketInfos: CommonDTO.TransfromMarketDto[],
  currency: AssetFilteredType
) => {
  const { dailies, exchangeName, assetName } = account

  dailies.forEach((daily) => {
    const defaultEquity = {
      btc: 0,
      eth: 0,
      usdc: 0,
      usdt: 0,
    }

    const defaultExchange: HistoricalAumType['equitiesOfExchange'] = {
      binance: {...defaultEquity},
      okx: {...defaultEquity},
      bybit: {...defaultEquity},
    }
    ...
  })
}
```

객체를 복사하여 초기화하여 해결해주었습니다!!

## 3주차 멘토링 후기

항해 플러스 프론트엔드 코스에서는 매주 1회 멘토링을 진행할 수 있습니다. 과제에 관련한 것들 이외에도 커리어 및 궁금했던 여러가지에 대해 자문을 구할 수 있는 시간이 있습니다. 그러기 위해 사전 노트를 작성하여 정해진 시간에 젭에서 진행했습니다.

![](/static/images/3.4.png)

이번주 멘토링을 통해 Hooks를 구현하는데 있어 비교하는 로직이외에도 여러 인사이트가 있어 공유합니다.

결국 잘하는 개발자가 되기 위해선

결국 개발을 좋아야해야 한다는 것. 즉, 팀에 기술적인 화두를 던져줄 수 있는 사람이 되어야 한다고 말씀해 주셨습니다. 그렇게 기술적으로 토론하기 위해선 자기만의 논리가 있어야 하지 않을까? 자기만의 논리가 있으려면? 우선 기술적으로 사용해 본 경험이 있어야 할 것 같다. → 결국 어디까지 파보고 어떤 결과를 만들었는지 → 요구 사항에 대한 스펙을 절 정의해야 함. → 이 문제를 해결하기 위해 다양한 방법으로 접근해 보고 → 그럼에도 불구하고 실패해도 제안을 할 수 있어야 함. → 결국에 실패로 끝내면 안 됨.

> 핵심적으론 문제 정의 실력 및 해결 경험을 중요하다.

## 결론

![](/static/images/3.5.png)

한주간 사내에서 작성했던 커밋들을 보면서 주간 회고를 진행했습니다.

![](/static/images/3.8.png)

이 문제를 만났고 이를 해결하기 위해 Try로 아래와 같이 생각을 해보았습니다.

![](/static/images/3.9.png)

> 우연히 이번 4주차부터 6주까지 클린코드에 관한 과제를 진행합니다. 과제를 진행하며 규칙을 발견하고 적용하여 돌아오겠습니다. 긴글 읽어주셔서 감사합니다.
