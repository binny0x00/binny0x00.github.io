---
layout: post
title: "Vanilla JS로 pre-Fiber React 스타일 Virtual DOM 라이브러리 구현하기"
date: 2026-03-25 00:00:00 +0900
categories: [dev, javascript, react]
tags: [virtual-dom, diff, patch, vanilla-js, benchmark]
description: Vanilla JS로 pre-Fiber React 스타일 Virtual DOM 라이브러리 `cyb-react`를 구현하고, Demo Lab과 Benchmark로 동작을 검증한 작업 기록입니다.
image:
  path: https://github.com/user-attachments/assets/bbcd3bb5-16c8-46b1-998a-47ffdd2d065c
  alt: Virtual DOM 라이브러리 구현
---

이번 주 팀 프로젝트에서는 React의 핵심 개념인 Virtual DOM과 Diff 알고리즘을 직접 구현하는 작업을 진행했다. 목표는 단순히 개념을 정리하는 데서 끝나는 것이 아니라, 실제로 동작하는 라이브러리와 이를 눈으로 검증할 수 있는 시연 앱까지 함께 만드는 것이었다.

이번 작업의 결과물은 크게 두 축으로 나뉜다.

1. `packages/cyb-react`에 위치한 pre-Fiber React 스타일의 Vanilla JS 라이브러리
2. `apps/showcase`에서 이 라이브러리를 검증하는 `Demo Lab`, `Benchmark` 페이지

## 왜 Virtual DOM을 직접 구현했는가

브라우저에서 실제 DOM을 직접 많이 건드리면 생각보다 비용이 크다. 노드 하나만 수정하는 것처럼 보여도 그 뒤에서는 layout 계산과 paint가 다시 일어날 수 있고, 변경이 누적될수록 Reflow / Repaint 부담도 커진다.

React가 Virtual DOM을 두는 이유도 여기서 출발한다. 실제 DOM을 매번 직접 수정하는 대신, 먼저 메모리 상의 트리를 만들고 이전 상태와 새 상태를 비교한 뒤 정말 바뀐 부분만 마지막에 반영한다.

이번 구현에서도 이 흐름을 그대로 따라갔다.

- 실제 DOM을 읽어 Virtual DOM 트리로 변환
- 이전 트리와 새 트리를 비교해 patch 계산
- 계산된 patch만 실제 DOM에 commit

결국 핵심은 "어떻게 최소한의 실제 DOM 변경만 일으킬 것인가"에 있었다.

## `cyb-react`에서 구현한 범위

이번에 만든 `cyb-react`는 Fiber 이전 React 스타일을 의식한 작은 런타임이다. README 기준으로 공개 API는 아래 세 가지다.

- `createElement(type, props, ...children)`
- `render(element, container)`
- `Component`

구현 범위는 다음과 같다.

- synchronous reconciliation
- class component 기반 `setState`
- DOM / text node 업데이트
- key 기반 child reconciliation
- event prop binding

반대로 이번 단계에서 일부러 제외한 범위도 있다.

- Fiber scheduler
- Hooks
- Concurrent rendering
- Suspense

즉, 최신 React 전체를 흉내 내기보다는 pre-Fiber 시절의 핵심 렌더링 모델을 Vanilla JS로 재구성하는 데 집중했다.

## 실제 구현에서 중요했던 포인트

### 1. DOM -> Virtual DOM 변환

`cyb-react` 내부에는 실제 브라우저 DOM을 읽어서 비교 가능한 트리 구조로 바꾸는 로직이 들어 있다. 여기서 단순히 태그 이름과 텍스트만 저장한 것이 아니라, 이후 diff와 patch에 필요한 메타 정보도 함께 보관했다.

- element node / text node 구분
- attribute 수집
- children 재귀 순회
- 공백 text node, comment node 제외
- `path`, `depth`, `key` 정보 기록

특히 `data-key`, `key`, `id` 순으로 비교용 key를 만들도록 처리해서 이후 자식 노드 재정렬까지 추적할 수 있게 했다.

### 2. Diff 알고리즘

두 Virtual DOM을 비교하면서 처리한 핵심 케이스는 다음과 같다.

- `CREATE`: 새 노드 추가
- `REMOVE`: 기존 노드 삭제
- `REPLACE`: 태그나 타입 자체가 바뀐 경우
- `TEXT`: 텍스트만 바뀐 경우
- `ATTR_SET`, `ATTR_REMOVE`: 속성 추가, 수정, 삭제
- `REORDER_CHILDREN`: key 기반 자식 노드 순서 변경

처음에는 추가/삭제/교체 정도만 생각했는데, 실제로 데모를 만들다 보니 reorder를 따로 다뤄야 "같은 노드를 새로 만들지 않고 위치만 바꾼다"는 흐름을 설명하기 쉬웠다. 이 부분이 실제 DOM 변경량을 줄이는 데도 의미가 있었다.

### 3. Patch 적용 순서

patch를 실제 DOM에 반영할 때는 순서도 중요했다. 삭제와 생성, 교체, 재정렬이 섞이면 단순 반복 처리만으로는 path가 어긋날 수 있기 때문이다.

그래서 구현에서는 대략 아래 순서로 적용했다.

1. 깊은 노드부터 `REMOVE`
2. `REPLACE`, `TEXT`, `ATTR` 같은 업데이트
3. 위에서 아래 순서로 `CREATE`
4. 마지막에 `REORDER_CHILDREN`

이렇게 나누니 동일한 트리 변경이라도 훨씬 안정적으로 반영됐다.

## Demo Lab: 개념 설명이 아니라 직접 만져보는 검증 화면

이번 작업에서 가장 만족스러웠던 부분은 엔진만 만든 게 아니라, 그 엔진을 눈으로 확인할 수 있는 `Demo Lab`을 함께 만든 점이다.

Demo Lab에서는 다음 흐름이 한 화면에서 이어진다.

- 초기 샘플 HTML을 실제 영역에 렌더링
- 그 DOM을 다시 Virtual DOM으로 변환
- 같은 트리로 테스트 영역을 구성
- 사용자가 편집기에서 HTML을 수정
- `Patch` 버튼 클릭 시 diff 계산
- 변경된 patch만 실제 영역에 적용
- 상태 스냅샷을 저장하고 Undo / Redo 지원

여기서 끝나지 않고 `MutationObserver`를 붙여서 실제 영역에서 어떤 DOM 변경이 발생했는지도 로그로 확인할 수 있게 만들었다. 덕분에 "Virtual DOM이 있다"는 설명을 넘어서, "그래서 실제 DOM에는 어떤 변화만 남는가"를 브라우저 레벨에서 바로 볼 수 있었다.

또한 내부적으로는 DFS / BFS 순회 결과, 노드 수, 최대 depth 같은 지표도 함께 보여줘서 단순 렌더링 데모가 아니라 비교 가능한 실험실에 가깝게 구성했다.

## Benchmark: 같은 변화 스트림을 두 런타임에 넣어 비교하기

Benchmark 화면은 단순히 "빠르다 / 느리다"를 숫자로만 말하는 구조로 만들고 싶지 않았다. 같은 모델 변화 스트림을 두 방식에 동시에 주입하고, 그 결과를 시각적으로 비교하는 것이 더 중요하다고 봤다.

한쪽은 `cyb-react`의 `render()`를 통해 diff + commit 흐름을 타고, 다른 한쪽은 imperative DOM 갱신 방식으로 같은 화면을 직접 업데이트한다.

비교 시나리오는 아래처럼 구성했다.

- text 변화
- attribute 변화
- list reorder
- mixed 변화

특히 최종 화면을 세로 목록이 아니라 작은 노드 타일의 매트릭스로 바꾼 것이 컸다. 노드 수가 많아져도 전체 분포가 한 번에 들어오고, reorder가 발생했을 때 위치 변화도 직관적으로 보였다.

결국 이 Benchmark는 단순한 ms 비교기가 아니라, 렌더링 전략의 차이를 눈으로 읽는 도구가 되어야 했다.

## 라이브러리와 쇼케이스를 분리한 이유

이번에 구조를 정리하면서 가장 중요하게 본 것은 "엔진"과 "시연 앱"의 분리였다.

최종 구조는 아래처럼 잡혔다.

```text
week4_project/
├── packages/
│   └── cyb-react/
└── apps/
    └── showcase/
```

이렇게 나누니 좋은 점이 분명했다.

- 결과물이 임시 실험 코드가 아니라 패키지처럼 보인다.
- Demo 앱이 라이브러리의 소비자 역할을 하면서 책임이 분리된다.
- 포트폴리오에서 설명할 때 구현 범위와 검증 범위를 나눠 말하기 쉽다.
- 이후 컴포넌트 모델이나 렌더링 API를 확장할 때 구조를 유지하기 좋다.

특히 `cyb-react`의 public API를 먼저 정리하고, `showcase`는 그 API만 사용하도록 두면서 "라이브러리 설계"와 "데모 UI 제작"을 한 번에 연습할 수 있었다.

## 작업하면서 다시 느낀 점

직접 구현해보니 React의 핵심은 결국 선언형 문법 자체보다도, 상태 변화 이후 어떤 방식으로 트리를 만들고, 무엇이 바뀌었는지 계산하고, 실제 DOM에는 어디까지 반영할지를 분리하는 데 있다는 점이 더 선명하게 보였다.

겉으로 보면 버튼 하나, 텍스트 하나 바뀌는 단순한 UI처럼 보여도 내부에서는 다음 흐름이 계속 반복된다.

- 트리 생성
- 이전 상태와 비교
- 변경점 계산
- commit 시점 분리
- 최소 DOM 갱신

이번 작업은 단순한 학습 정리를 넘어서 구현물 중심으로 남았다는 점에서도 의미가 컸다. 이론만 설명하는 것보다 "어떤 구조로 만들었고, 실제로 무엇을 검증했는가"를 말할 수 있게 되었기 때문이다.

## 마무리

이번 주 작업으로 Virtual DOM 기반 라이브러리 `cyb-react`와 이를 검증하는 `Demo Lab`, `Benchmark` 화면까지 한 흐름으로 묶어볼 수 있었다. 아직 Fiber, Hooks, concurrent rendering 같은 현대 React의 영역까지 간 것은 아니지만, pre-Fiber React가 풀고자 했던 핵심 문제를 구현 관점에서 따라가 본 경험으로는 충분히 의미 있었다.

다음 단계에서는 여기서 더 나아가 컴포넌트 모델을 다듬고, state 업데이트 흐름과 render API를 더 정교하게 정리해서 정말 "작지만 설명 가능한 React 스타일 라이브러리"로 발전시켜 보고 싶다.
