---
title: 3. custom node
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

## Custom Node

- 기본적으로 제공하는 노드형태(input/default/output) 외에 커스텀한 노드 형태 정의 가능(노드/핸들/엣지)

## 사용방법 (커스텀 타입/v-slot 생성 방법 2가지가 있음)

#### 커스텀 타입으로 노드 생성

- 커스텀 컴포넌트들을 생성 후 VueFlow 컴포넌트에 node-types 프롭스로 커스텀 노드 생성 가능 (커스텀 타입에 정의된 컴포넌트 렌더링을 수행)

```javascript
## FlowCanvas.vue
## script
import VariableNode from './customnodes/VariableNode.vue'
const customNodeTypes: CustomNode = {
  [CustomTypesDefine.VARIABLE_NODE]: markRaw(VariableNode),
}

## template
  <div class="d-flex flex-column">
    <div class="dnd-flow" @drop="onDrop">
      <VueFlow
        v-model:nodes="nodes"
        v-model:edges="edges"
        :node-types="customNodeTypes"
        @dragover="onDragOver"
        @dragleave="onDragLeave"
        @connect="onConnect"
      >
      ...
      </VueFlow>
      <FlowSidebar />
    </div>
  </div>

# Varible Node
<script lang="ts" setup>
import { Handle, Position } from '@vue-flow/core'
import { ref, watch } from 'vue'

const props = defineProps({
  data: Object,
})

const emit = defineEmits(['update:data'])
const value = ref('')

watch(value, (newValue) => {
  emit('update:data', { ...props.data, value: newValue })
})
</script>

<template>
  <div class="input-node font-style">
    <div class="d-flex flex-row justify-content-center">
      <h1>Number</h1>
      <input class="input-box" type="number" v-model="value" placeholder="Enter value" />
    </div>
    <Handle type="source" :position="Position.Bottom" />
  </div>
</template>

```

#### 템플릿(v-slot)으로 노드 생성

- 커스텀 컴포넌트들을 생성 후 VueFlow 내 template으로 생성
- scoped slot을 통해 자식 컴포넌트에서 올려주는 프롭스와 템플릿 값들로 노드를 생성 (ex. 노드 외에도 줌 패널/ 커넥션 라인 등 scoped slot으로 커스터마이즈 가능)
  > vue slot = > 부모가 원하는 콘텐츠를 삽입할 수 있지만, 자식의 데이터를 활용할 수 없음
  > scoped slot => 자식이 가진 데이터를 부모에게 전달할 수 있어서, 부모가 더 유연하게 렌더링할 수 있음. 동적 리스트, 재사용 가능한 UI 컴포넌트등 유연한 디자인이 필요할때 사용
- ref. https://vuejs.org/guide/components/slots

```javascript
# App.vue

<script setup>
import { ref } from 'vue'
import { VueFlow } from '@vue-flow/core'
import { Background } from '@vue-flow/background'
import { initialEdges, initialNodes } from './initial-elements.js'
import ValueNode from './ValueNode.vue'
import OperatorNode from './OperatorNode.vue'
import ResultNode from './ResultNode.vue'

const nodes = ref(initialNodes)

const edges = ref(initialEdges)
</script>

<template>
  <VueFlow class="math-flow" :nodes="nodes" :edges="edges" fit-view-on-init>
    <template #node-value="props">   ===> 자식 컴포넌트의 템플릿( ValueNode.vue)을 받아와 렌더링을 수행하며, scoped slot 형태로 slot props를 받아와 랜더링 수행
      <ValueNode :id="props.id" :data="props.data" />
    </template>

    <template #node-operator="props">
      <OperatorNode :id="props.id" :data="props.data" />
    </template>

    <template #node-result="props">
      <ResultNode :id="props.id" />
    </template>

    <Background />

  </VueFlow>
</template>

# ValueNode.vue (id는 노드 생성 시 vue-flow에서 id 값을 관리하여 내려줌)

<script setup>
import { computed } from 'vue'
import { Handle, Position, useVueFlow } from '@vue-flow/core'

const props = defineProps(['id', 'data'])

const { updateNodeData } = useVueFlow()

const value = computed({
  get: () => props.data.value,
  set: (value) => updateNodeData(props.id, { value }),
})
</script>

<template>
  <input :id="`${id}-input`" v-model="value" type="number" class="nodrag" />

  <Handle type="source" :position="Position.Right" :connectable="false" />
</template>

```

Ref. https://vuejs.org/guide/components/slots#slot-content-and-outlet
