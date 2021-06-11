---
layout: post
title:  "Redux 패턴에 대한 고찰"
date:   2020-12-04 20:00:00 +0900
categories: script
---

예전에 Vanila.JS 로 위지윅 같이 한 화면에서 변화가 많은 프로그램을 만들어 본 경험이 있다. 이 때 내가 느꼈던 불편했던 점 하나는 데이터가 수정되었을 떄 모델 객체에 이 수정된 내용을 반영하는 것과 수정된 내용이 실제 화면에 렌더링 시키는 동작이 상이하다는 것이 였다. 이를 명확히 하지 못했었고 프로젝트를 진행하면서 이 이유로 혼란이 왔었던 적이 있다.

기존의 MVC 패턴은 고객이 Controller 쪽에 특정 요청을 할 경우 데이터가 변경되야 한다면 Model을 변경하고, 화면이 변경되야 한다면 View 가 변경된다. 사실 이 Controller, Model, View 셋 컴포넌트 간에 데이터가 어느쪽으로 흘러야 한다라는 제약이 강하지 않아서 개발자의 성향에 따라 방식이 달라진다. 

이러한 단점으로 인해서 Flux 패턴이 고안되었고, 이 방식이 React 라이브러리에서도 채택이 되어서 Redux 라는 상태 관리 라이브러리가 생겼는데. 그렇다면 이 Flux, Redux 의 패턴이 기존의 MVC 와 무엇이 다른지 살펴보자.

### 구성요소

MVC 패턴은 Model, View, Contoller 가 핵심 요소 였다면, Redux 패턴은 Action, Dispatcher, Reducer, Store 이 4가지가 중요한 구성 요소이다.

#### 1. Action

고객이 우리 웹 어플리케이션에 접속해서 하는 행위가 Redux가 관리하는 데이터에 변화가 생긴다면 이는 Action 으로 관리해야한다. 이 Action 의 개념은 사전적 정의와 굉장히 유사하다. 단순히 웹 어플리케이션 사용자가 하는 행위들을 말한다.
예를 들어서 카카오톡이 웹 어플리케이션으로 동작하고 있다고 해보자. 하단에 메뉴 4개가 존재하고 각각 친구목록, 대화방 목록, 설정 등을 보여준다. 만약 사용자가 친구 목록을 보고 있는 상태에서 대화방 목록을 보기위해 메뉴 버튼을 눌렀다면. 이 대화방 버튼을 눌렀다는 것 자체가 하나의 Action 이다. Action 은 Redux 패턴에서 다른 구성요소들이 동작하기 위한 시발점이 된다.

```typescript

export const SET_OBJECTIVE_ACTION = 'SET_OBJECTIVE_ACTION' as const
export const SET_PLAN_ACTION = 'SET_PLAN_ACTION' as const

export const setObjectiveAction = (objective: Objective) => ({
    type: SET_OBJECTIVE_ACTION,
    payload: objective
})

export const setPlanAction = (plan: Plan) => ({
    type: SET_PLAN_ACTION,
    payload: plan
})

```

#### 2. Dispatcher

Dispatcher는 용어 그대로 보내다, 분배하다는 느낌이 강하다. Action이 왔다면 그 Action에 맞는 Reducer를 찾아서 보내준다. 예를 들어 대화방 메뉴를 눌렀다면 이 Dispatcher에게 Action이 다가와서 물어본다.

```
Action : "대화방 버튼을 클릭해서 왔어요"
Dispatcher : "대화방 버튼 기능은 이 Reducer에게 가시면 됩니다."
```

위와 같이 모든 Action과 Reducer 사이에서 연결관계를 이어준다.

#### 3. Reducer

Action이 발생했을 때 실제 어떠한 처리를 하는 곳이 Reducer이다 대화방 버튼을 눌렀을 때 대화방 목록이 보이도록 이 Reducer 영역에서 처리를 해야 한다. Reducer 는 입력으로 기존의 상태값과, 변경된 상태값을 가지는데 이 둘을 비교해서 필요한 작업을 처리한다.

```typescript

export default function cell(state: Cell = new Cell("CELL"), action: CellAction) {
    switch(action.type) {
        case SET_OBJECTIVE_ACTION :
            return action.payload;
        case SET_PLAN_ACTION :
            return action.payload;
        default :
            return state;
    }
}

```

#### 4. Store

Reducer는 기존의 상태값과 새롭게 들어온 상태값을 비교하여 비지니스 로직을 구현한다고 위에서 언급했다. 그러면 기존의 상태값은 어디에서 저장되고 있을까? 이 Store 영역에서 상태값들을 모두 가지고 있으며 이들을 관리한다. 그렇기 때문에 Redux 가 상태관리 라이브러리라고 불리는 것이다.

```typescript

const rootReducer = combineReducers({
    objectives,
    plans,
    todos,
    cell,
    page,
    noti,
    sign
})

export default rootReducer;

export type RootState = ReturnType<typeof rootReducer>;

```