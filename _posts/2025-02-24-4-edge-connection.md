---
title: 4. edge connection
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

## Edge Connection

- 생성된 노드간 엣지를 연결하는데 사용되며, Vue Flow에서 사용되는 주요 기능들은 다음과 같음

#### Getter

- getSelectedElements() - returns all currently selected elements

#### Action

- addEdges() - edge parse 후 edge state(node/handle)에 추가하여 노드간 edge 커넥션을 만듦
- fitView() - Fit the viewport around visible nodes
- toObject() - return an object of graph values (elements, viewport transform) for storage and re-loading a graph
- fromObject() - load graph from export obj
- applyNodeChanges() - applies default node change handler
- updateNodeData() - vue flow 내에서 해당하는 노드 id에 data 값을 업데이트
- useNodeConnections() - 노드에 연결된 모든 에지를 검색하는 후크이며, 핸들 유형 및 ID로 필터링도 수행 가능
- useNodesData() - 하나 또는 여러 노드의 데이터를 받아오기 위해 사용

## 코드 구성

#### FlowCanvas.vue

- 드래그앤 드롭은 vue-flow에 emit 이벤트를 받아와 처리(@dragover/@dragLeave)
- 노드간 연결은 vue-flow에 onConnect emit 이벤트를 받아와 addEdge Action Call로 처리
- Json Save/Load/Print는 toObject/fromObject를 활용하여 로컬 스토리지에 저장 및 불러오기 수행
- Delete는 keydown 이벤트와 getSelectedElements로 해당하는 아이디를 받아와 해당하는 노드 업데이트 수행(applyNodeChanges)

```javascript

<script lang="ts" setup>
import { markRaw, ref, nextTick } from 'vue'
import { MiniMap } from '@vue-flow/minimap'
import { VueFlow, useVueFlow } from '@vue-flow/core'
import DropzoneBackground from './DropzoneBackground.vue'
import FlowSidebar from './FlowSidebar.vue'
import useDragAndDrop from './useDnD'

import { CustomTypesDefine, type CustomNode, nodeStroke } from './customnodes/CustomNodeTypes.ts'
import VariableNode from './customnodes/VariableNode.vue'
import OperatorNode from './customnodes/OperatorNode.vue'
import ResultNode from './customnodes/ResultNode.vue'

import { useLayout } from './layout/useLayout'
import { onSave, onRestore, getFlowObject } from './vue-flow-file-control/SaveRestoreControl.ts'

const { addEdges, fitView, toObject, fromObject, getSelectedElements, applyNodeChanges } =
  useVueFlow()
const { onDragOver, onDrop, onDragLeave, isDragOver } = useDragAndDrop()
const { layout } = useLayout()

const nodes = ref([])
const edges = ref([])

const customNodeTypes: CustomNode = {
  [CustomTypesDefine.VARIABLE_NODE]: markRaw(VariableNode),
  [CustomTypesDefine.OPERATOR_NODE]: markRaw(OperatorNode),
  [CustomTypesDefine.RESULT_NODE]: markRaw(ResultNode),
}

async function layoutGraph(direction) {
  await stop()

  nodes.value = layout(nodes.value, edges.value, direction)

  nextTick(() => {
    fitView()
  })
}

const onConnect = (connection) => {
  addEdges(connection)
}

const onSaveClicked = () => {
  const getFlowObject = toObject()
  onSave(getFlowObject)
}

const onLoadClicked = () => {
  const savedObject = onRestore()
  if (savedObject) {
    fromObject(savedObject)
  }
}

const onPrintJsonClicked = () => {
  console.log(getFlowObject())
  console.log(toObject())
  console.log(typeof toObject())
}

const handleKeyDown = (e) => {
  console.log(e)
  console.log(getSelectedElements.value)
  if (e.key === 'Delete') {
    applyNodeChanges([
      {
        id: getSelectedElements.value[0].id,
        type: 'remove',
      },
    ])
  }
}
</script>

<template>
  <div class="d-flex flex-column">
    <div class="d-flex flex-row">
      <div style="width: 96vw">
        <button class="button-style" @click="onSaveClicked">Save</button>
        <button class="button-style" @click="onLoadClicked">LOAD</button>
        <button class="button-style" @click="onPrintJsonClicked">Print Json</button>
        <button style="float: right" class="button-style" @click="layoutGraph('LR')">
          가로 배치
        </button>
        <button style="float: right" class="button-style" @click="layoutGraph('TB')">
          새로 배치
        </button>
      </div>
    </div>

    <div class="dnd-flow" @drop="onDrop">
      <VueFlow
        v-model:nodes="nodes"
        v-model:edges="edges"
        :node-types="customNodeTypes"
        @dragover="onDragOver"
        @dragleave="onDragLeave"
        @connect="onConnect"
        :min-zoom="0.2"
        :default-zoom="1.5"
        :max-zoom="4"
        @keydown="handleKeyDown"
      >
        <DropzoneBackground
          :style="{
            backgroundColor: isDragOver ? 'black' : 'transparent',
            transition: 'background-color 0.2s ease',
          }"
        >
          <p v-if="isDragOver">Drop here</p>
        </DropzoneBackground>

        <MiniMap class="minimap" :node-color="nodeStroke" />
      </VueFlow>
      <FlowSidebar />
    </div>

  </div>
</template>

```

#### VariableNode.vue

- 인풋노드에서 인풋값을 받고 노드 값 업데이트(updateNodeData)

```javascript
<script lang="ts" setup>
import { computed } from 'vue'
import { Handle, Position, useVueFlow } from '@vue-flow/core'

const props = defineProps(['id', 'data'])

const { updateNodeData } = useVueFlow()

const value = computed({
  get: () => props.data.value || 0,
  set: (value) => updateNodeData(props.id, { value }),
})
</script>

<template>
  <div class="vue-flow__node-value">
    <input :id="`${id}-input`" v-model="value" type="number" />

    <Handle type="source" :position="Position.Bottom" :connectable="true" />

  </div>
</template>

```

#### OperatorNode.vue

- 현재 노드에서 target으로된 핸들 노드의 값들을 받아오고,

  - useNodesData = 필터링되어 받아온 노드데이터들의 id로 각 노드의 정보를 배열 형태로 가져옴
  - useNodeConnections = 핸들타입 target으로 연결된 노드 리스트들을 필터링해서 받아옴

- 각 노드 핸들에 인풋 데이터가 아닌 OperatorNode로 연결된 경우 인풋 값이 아닌 계산 결과 값(result)을 받아와 업데이트
  - updateNode = 핸들에 들어온 값들이 VariableNode의 입력값인지, OperatorNode값인지에 따라 계산 결과 값 처리
  - updateNodeData = 계산 결과값을 해당하는 노드에 업데이트 처리

```javascript
<script lang="ts" setup>
import { Handle, Position, useVueFlow, useNodeConnections, useNodesData } from '@vue-flow/core'
import { ref, watch } from 'vue'
import Icon from './OperatorIcon.vue'

const props = defineProps(['id', 'data'])

const operators = ['+', '-', '*', '/']
const mathFunctions = {
  '+': (a, b) => a + b,
  '-': (a, b) => a - b,
  '*': (a, b) => a * b,
  '/': (a, b) => a / b,
}
const selectedOperator = ref('+')
const { updateNodeData } = useVueFlow()

const sourceConnections = useNodeConnections({
  handleType: 'target',
})

const valueData = useNodesData(() => sourceConnections.value.map((connection) => connection.source))

const updateNode = (operator) => {
  if (valueData.value.length < 2) return

  const [a, b] = valueData.value.map(({ data }) => data?.value)
  if (a === undefined && b === undefined) {
    updateNodeData(props.id, {
      result: mathFunctions[operator](
        valueData.value[0].data?.result,
        valueData.value[1].data?.result,
      ),
      operator: operator,
    })
  } else if (a === undefined) {
    updateNodeData(props.id, {
      result: mathFunctions[operator](valueData.value[0].data?.result, b),
      operator: operator,
    })
  } else if (b === undefined) {
    updateNodeData(props.id, {
      result: mathFunctions[operator](a, valueData.value[1].data?.result),
      operator: operator,
    })
  } else {
    updateNodeData(props.id, {
      result: mathFunctions[operator](a, b),
      operator: operator,
    })
  }
}

const onOperatorButtonClicked = (operator) => {
  selectedOperator.value = operator
  updateNode(operator)
}

watch(
  () => [valueData.value],
  async () => {
    updateNode(selectedOperator.value)
  },
)
</script>

<template>
  <div class="vue-flow__node-operator buttons">
    <button
      v-for="operator of operators"
      :key="`${id}-${operator}-operator`"
      :class="{ selected: data.operator === operator }"
      @click="onOperatorButtonClicked(operator)"
    >
      <Icon :name="operator" />
    </button>
  </div>

  <Handle id="input-1" type="target" :position="Position.Top" :connectable="true" />
  <Handle id="input-2" type="target" :position="Position.Top" :connectable="true" />
  <Handle type="source" :position="Position.Bottom" :connectable="true" />
</template>

```

#### ResultNode.vue

- VariableNode/OperatorNode에서 업데이트(updateNodeData)해주는 input Value 값과 연산자 값들을 받아와 식 및 결과값 출력
  - operatorData =결과노드의 연결된 연산자 노드의 값을 받아옴
  - valueData = 결과 노드에 연결된 연산자 노드의 target 핸들에 연결된 노드값들을 받아옴
- valueData에서 target 핸들에 연결된 노드가 variablenode가 아닌 operatornode면 해당 계산 결과값을 가져와 result 출력

```javascript
<script lang="ts" setup>
import { computed } from 'vue'
import { Handle, Position, useNodeConnections, useNodesData } from '@vue-flow/core'

const sourceConnections = useNodeConnections({
  handleType: 'target',
})

const operatorSourceConnections = useNodeConnections({
  handleType: 'target',
  nodeId: () => sourceConnections.value[0]?.source,
})

const operatorData = useNodesData(() =>
  sourceConnections.value.map((connection) => connection.source),
)

const valueData = useNodesData(() =>
  operatorSourceConnections.value.map((connection) => connection.source),
)

const result = computed(() => {
  return operatorData.value[0]?.data?.result
})
</script>

<template>
  <div class>
    <template v-for="(value, i) in valueData" :key="`${value.id}-${value.data}`">
      <span v-if="value.id.startsWith('var_')">
        {{ value.data?.value }}
      </span>
      <span v-else>
        {{ value.data?.result }}
      </span>

      <span v-if="i !== valueData.length - 1">
        {{ operatorData[0].data?.operator }}
      </span>
    </template>

  </div>

<span> = </span>

  <span class="result" :style="{ color: result > 0 ? '#5EC697' : '#f15a16' }">
    {{ result }}
  </span>

<Handle
    type="target"
    :position="Position.Top"
    :connectable="true"
    :style="{ background: result > 0 ? '#5EC697' : '#f15a16' }"
  />
</template>

```
