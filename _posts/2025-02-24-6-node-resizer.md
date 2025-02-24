---
title: 6. node resizer
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

## Node Resizer ( https://vueflow.dev/guide/components/node-resizer.html )

- 생성된 노드의 크기를 동적으로 조절할수 있는 기능을 말하며, @vue-flow/node-resizer library 를 사용하여 조절 가능

## 사용방법

- @vue-flow/node-resizer library 추가 후,
- resize가 가능하도록 조절할 노드 템플릿에 추가하면 됨
- resize의 경우 자식노드가 있을 경우 limit 제한을 위해 updateMinSize 구현
- resize 수행(DOM 요소 크기 변경) 시 너무 많은 ResizeObserver 순환 참조로 인해 debounce를 하여 updateMinSize를 호출
- style scoped로 사용 시 library 내 CSS 스타일이 정상 동작하지 않아 scoped는 사용하지 않음 ( 추후 확인 필요 )

* debounce를 하지 않을 경우 // Uncaught ResizeObserver loop completed with undelivered notifications. 이런 오류가 남
  > Debounce란?
  >
  > > 연속적으로 발생하는 이벤트를 하나로 처리하는 방식으로 성능 저하 문제를 해결 ( 타이머를 사용하여 처음 또는 마지막에 발생한 이벤트에만 함수를 실행)
  > > Throttling 방식[일정주기 간격으로 발생되도록함]도 있으나 Debounce로 사용함

```javascript
# lib 설치
npm install @vue-flow/node-resizer

# 사용하고자할 custom 노드(Group 노드)
<script setup lang="ts">
import { NodeResizer } from '@vue-flow/node-resizer'
import { ref, onMounted, onUpdated, watch } from 'vue'
import { useVueFlow } from '@vue-flow/core'
import { CustomTypesDefine } from '../customnodes/CustomNodeTypes.ts'
import Tooltip from '../tooltip/NodeTooltip.vue'

const props = defineProps(['data'])

const {
  addEdges,
  fitView,
  toObject,
  fromObject,
  getSelectedElements,
  applyNodeChanges,
  updateEdgeData,
  onNodesChange,
  findNode,
} = useVueFlow()

const showDialog = ref(false)
const nodeId = ref('')
const onDialogButtonClicked = (id: string) => {
  showDialog.value = !showDialog.value
  nodeId.value = id
}

const minWidth = ref(100)
const minHeight = ref(30)

const updateMinSize = () => {
  const nodes = toObject().nodes
  const childNodes = nodes.filter((node) => node.parentNode && node.parentNode === props.data.label)

  let maxX = 0
  let maxY = 0
  let maxHeight = 10
  let maxWidth = 10

  if (childNodes.length > 0) {
    for (const childnode of childNodes) {
      const node = findNode(childnode.id)
      if (maxX < node?.position.x) {
        maxX = node.position.x
        maxWidth = node.dimensions.width
      }
      if (maxY < node.position.y) {
        maxY = node.position.y
        maxHeight = node.dimensions.height
      }
    }
  }
  minWidth.value = maxX + maxWidth
  minHeight.value = maxY + maxHeight
}

onMounted(() => {
  updateMinSize()
})

const debounceUpdateMinSize = debounce(updateMinSize, 500)

watch(
  () => toObject().nodes,
  () => {
    debounceUpdateMinSize()
  },
  { deep: true },
)

// DOM 요소 크기 변경시 너무 많은 ResizeObserver 순환 참조 인해 debounce 적용
// ex. Uncaught ResizeObserver loop completed with undelivered notifications.
function debounce(func, wait) {
  let timeout
  return function (...args) {
    clearTimeout(timeout)
    timeout = setTimeout(() => func.apply(this, args), wait)
  }
}
</script>

<template>
  <div @click="onDialogButtonClicked(props.data.label)">
    <NodeResizer :min-width="minWidth" :min-height="minHeight" />
    <Tooltip :show="showDialog" :top="-45" :left="80" :nodeId="nodeId">
      <div class="node_style">
        <slot>
          <div class="group-title">{{ data.label }}</div>
        </slot>
      </div>
    </Tooltip>
  </div>
</template>

<style>
.group-title {
  font-weight: bold;
  margin: 10px;
}

.node_style {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
}

.vue-flow__resize-control.line {
  border-color: whitesmoke;
  border-width: 0;
  border-style: dashed;
  box-shadow: 0 0 20px white;
}

.vue-flow__resize-control.line.top,
.vue-flow__resize-control.line.bottom {
  height: 1px;
  transform: translate(0, -50%);
  left: 0;
  width: 100%;
}

.vue-flow__resize-control.line.left,
.vue-flow__resize-control.line.right {
  width: 1px;
  transform: translate(-50%, 0);
  top: 0;
  height: 100%;
}

.vue-flow__resize-control.top,
.vue-flow__resize-control.bottom {
  cursor: ns-resize;
}

.vue-flow__resize-control.left,
.vue-flow__resize-control.right {
  cursor: ew-resize;
}

.vue-flow__resize-control {
  position: absolute;
}

.vue-flow__resize-control.line.top {
  top: 0;
  border-top-width: 1px;
}

.vue-flow__resize-control.line.right {
  left: 100%;
  border-right-width: 1px;
}

.vue-flow__resize-control.line.bottom {
  border-bottom-width: 1px;
  top: 100%;
}

.vue-flow__resize-control.line.left {
  left: 0;
  border-left-width: 1px;
}

.vue-flow__resize-control.handle.bottom.left,
.vue-flow__resize-control.handle.top.left {
  left: 0;
}

.vue-flow__resize-control.handle.bottom.right,
.vue-flow__resize-control.handle.top.right {
  left: 100%;
}

.vue-flow__resize-control.handle.top {
  top: 0;
}

.vue-flow__resize-control.handle.bottom {
  top: 100%;
}

.vue-flow__resize-control.top.left,
.vue-flow__resize-control.bottom.right {
  cursor: nwse-resize;
}

.vue-flow__resize-control.bottom.left,
.vue-flow__resize-control.top.right {
  cursor: nesw-resize;
}

.vue-flow__resize-control.handle {
  width: 10px;
  height: 10px;
  border: 1px solid #fff;
  background-color: #3367d9;
  transform: translate(-50%, -50%);
}
</style>


```
