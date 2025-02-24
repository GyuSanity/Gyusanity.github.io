---
title: 1. vue flow intro
author: Yeon Gyu Yang
date: 2025-02-24 00:00:00 +0800
categories: [vue-flow]
tags: [vue-flow vue3 ]
math: true
toc: true
mermaid: true
description: >-
  ROS2에 대해 알아보자
image:
  path: "https://raw.githubusercontent.com/bcakmakoglu/vue-flow/master/vue-flow.gif"
  alt: "vue-flow"

---


## Vue-flow

- React Flow에서 가져온 기존 기능과 코드를 기반으로 구축되었으며, React Flow의 기능을 그대로 가져와 Vue 3로 컨버팅한 라이브러리
- vuex 상태 관리 패턴 라이브러리인 vuex의 구조를 따라 노드/엣지/핸들 생성 관리에 대한 vue flow state(상태), Action(비동기 로직 처리), Getter, Event, (mutation?)들을 제공하며,
- 그외 Utilites/Component들을 제공
  - Ref. React Flow : https://reactflow.dev/examples/nodes/drag-handle
  - Ref. Vue Flow : https://vueflow.dev/guide/ 

## 제공하는 Feature library

#### vue-flow/vue-background

- Vue Flow에는 점과 선의 두 가지 배경 패턴 변형이 제공
- 사용방법 = pass the Background component as a child to the VueFlow component.

```javascript
<script setup>
import { VueFlow } from '@vue-flow/core'
import { Background } from '@vue-flow/background'
</script>

<template>
  <VueFlow>
    <Background />
  </VueFlow>
</template>

```

- Detail : https://vueflow.dev/guide/components/background.html

#### vue-flow/minimap

- 반응성 pannable / zoomable props설정 가능 ( 드래그앤 줌 스크롤을 미니 맵에서 사용가능하게 해줌 )
- 사용방법 = pass the MiniMap component as a child to the VueFlow component

```javascript
<script setup>
import { VueFlow } from '@vue-flow/core'
import { MiniMap } from '@vue-flow/minimap'

// import default minimap styles
import '@vue-flow/minimap/dist/style.css'
</script>

<template>
  <VueFlow>
    <MiniMap />
  </VueFlow>
</template>

```

#### vue-flow/core

- vue-flow를 그리는데 필요한 코어 립 ( vueFlow / Pannel / useVueFlow / useHandleConnections(생성된 핸들 후킹) / useNodeConnections(연결된 노드 후킹) / useNodeData (생성된 노드 Array 정보 받아옴) / - useHandle (커스템 노드 생성시)을 가지고 있으며,
- 기본적인 노드, 엣지, 핸들을 제공
  - 노드 => https://vueflow.dev/guide/node.html
  - CURD
    - 노드 생성(C) = useVueFlow()를 사용해 Panel/VueFlow에서 노드 상태 생성 가능노드 - ID/xy좌표/data로 이루어짐 (프롭스) 
    - 노드 읽기(R) = 노드 프롭스 객체 find 메서드로 접근ex. edges.value.find
    - 노드 삭제(D) = ( prop 이나 를 mode-value)를 사용 하여 그래프에서 노드를 제거 ex. const { removeNodes } = useVueFlow()
    - 노드 업데이트(U) = 반응형 Node 객체 접근해서 업데이트 ex. const instance = useVueFlow(); instance.updateNode(nodeId, { selectable: false, draggable: false })
  - 노드 종류 (node type으로 default, input, output, custom 가능)
  - 노드 이벤트 제어가능(Drag/Stop/Start)
  - 사용자 정의 노드 생성 가능 - 노드 생성 탬플릿 /스크롤 노드 / 스타일 변경 / Drag 금지(noDragClassName) 등등 -엣지 - vueFlow 프롭스로 Edit 정보를 내려 노드간 선 연결 생성 가능(사이드바/도우모음 등에서도 사용 가능), https://vueflow.dev/guide/edge.html
    -CRUD -엣지 생성(C) = const { addEdges } = useVueFlow()를 사용하거나  edges 프롭스를 내려 생성<VueFlow :nodes="nodes" :edges="edges" /> -엣지 삭제(D) = v-model이나 edges 프롭스 접근하여 제거 가능 -엣지 읽기(R) = 프롭스 객체 find 메서드로 접근 ex. edges.value.find -엣지 업데이트(U) = 반응형 객체 edges 프롭스를 접근해서 업데이트 가능 -엣지 종류 - step/smoothstep/straight를 통해 노드 속성을 조절 가능 - 사용자 정의 엣지 생성 가능 - 엣지 이벤트 제어 가능(EdgeClick/EdgeDoubleClick/EdgeContextmenu) - 핸들 - 노드간 연결을 수행하는 작은 원( targetPosition/SourcePosition에 표현 ), https://vueflow.dev/guide/handle.html - 이벤트 - 커스텀 템플릿 에서 이벤트 발생 가능(ex. Emit -updateNodeInternals) - 노드를 커스텀하게 수정하고자 할때 노드 템플릿에서 핸들 속성을 정의하여 사용 가능

```javascript
>핸들 위치/ 노드 내 여러개의 핸들 / 숨겨진 핸들 / 연결 제한 / 동적으로 핸들 위치 추가 및 제거 가능
<script setup>
import { Handle } from '@vue-flow/core'

defineProps(['id', 'sourcePosition', 'targetPosition', 'data'])
</script>

<template>
  <Handle type="source" :position="sourcePosition" />

  <span>{{ data.label }}</span>

  <Handle type="target" :position="targetPosition" />
</template>

```

#### vue-flow/contorls

- 확대, 축소, 맞춤보기 및 잠금/잠금 해제 버튼이 있음
- 사용방법 = 위와 동일

```javascript
<script setup>
import { VueFlow } from '@vue-flow/core'
import { Controls } from '@vue-flow/controls'

// import default controls styles
import '@vue-flow/controls/dist/style.css'
</script>

<template>
  <VueFlow>
    <Controls />
  </VueFlow>
</template>

```

- Detail : https://vueflow.dev/guide/components/controls.html
- Necessary Lib
  - 아래와 같은 lib 설치 필요

```javascript
> npm install @vue-flow/background @vue-flow/controls @vue-flow/minimap @vue-flow/core @vue-flow/node-resizer @vue-flow/node-toolbar --save

  "dependencies": {
    "@vue-flow/background": "^1.3.2",
    "@vue-flow/controls": "^1.1.2",
    "@vue-flow/core": "^1.42.0",
    "@vue-flow/minimap": "^1.5.1",
    "@vue-flow/node-resizer": "^1.4.0",
    "@vue-flow/node-toolbar": "^1.1.0",
    "pinia": "^2.3.0",
    "vue": "^3.5.13",
    "vue-router": "^4.5.0"
  },

```

- 기본 vue flow 스타일  ( custom 필요시 , https://vueflow.dev/guide/theming.html )

```javascript
@import "https://cdn.jsdelivr.net/npm/@vue-flow/core@1.42.0/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/core@1.42.0/dist/theme-default.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/controls@latest/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/minimap@latest/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/node-resizer@latest/dist/style.css";

html,
body,
#app {
margin: 0;
height: 100%;
}

#app {
text-transform: uppercase;
font-family: "JetBrains Mono", monospace;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
text-align: center;
color: #2c3e50;
}

/* .vue-flow__minimap {
transform: scale(75%);
transform-origin: bottom right;
} */

.dnd-flow {
flex-direction: column;
display: flex;
height: 100%;
}

.dnd-flow aside {
color: #fff;
font-weight: 700;
border-right: 1px solid #eee;
padding: 15px 10px;
font-size: 12px;
background: #10b981bf;
-webkit-box-shadow: 0px 5px 10px 0px rgba(0, 0, 0, 0.3);
box-shadow: 0 5px 10px #0000004d;
}

.dnd-flow aside .nodes > * {
margin-bottom: 10px;
cursor: grab;
font-weight: 500;
-webkit-box-shadow: 5px 5px 10px 2px rgba(0, 0, 0, 0.25);
box-shadow: 5px 5px 10px 2px #00000040;
}

.dnd-flow aside .description {
margin-bottom: 10px;
}

.dnd-flow .vue-flow-wrapper {
flex-grow: 1;
height: 100%;
}

@media screen and (min-width: 640px) {
.dnd-flow {
  flex-direction: row;
}

.dnd-flow aside {
  min-width: 25%;
}
}

@media screen and (max-width: 639px) {
.dnd-flow aside .nodes {
  display: flex;
  flex-direction: row;
  gap: 5px;
}
}

.dropzone-background {
position: relative;
height: 100%;
width: 100%;
}

.dropzone-background .overlay {
position: absolute;
top: 0;
left: 0;
height: 100%;
width: 100%;
display: flex;
align-items: center;
justify-content: center;
z-index: 1;
pointer-events: none;
}

```
