---
title: 7. import/export
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

## Flow Import/Export란

- vue flow 내 생성된 객체의 값들을 아래와 같은 포맷으로 관리하고 있으며, 이를 통해 vue-flow 내에서 import/export 기능을 구현

```json
{
  "nodes": [
    {
      "id": "group_0",
      "type": "group",
      "initialized": true,
      "position": {
        "x": 141.44791412353516,
        "y": 296.8888854980469
      },
      "data": {
        "label": "group_0"
      },
      "style": {
        "backgroundColor": "rgba(246, 92, 92, 0.49)",
        "height": "202px",
        "width": "570px"
      }
    },
    {
      "id": "group_2",
      "type": "group",
      "initialized": true,
      "position": {
        "x": 111,
        "y": 27
      },
      "data": {
        "label": "group_2"
      },
      "style": {
        "backgroundColor": "rgba(246, 92, 92, 0.49)",
        "height": "153px",
        "width": "434px"
      },
      "parentNode": "group_0",
      "extent": "parent"
    },
    {
      "id": "std_2",
      "type": "start",
      "initialized": true,
      "position": {
        "x": 362.47294143333215,
        "y": 9.425647289998892
      },
      "data": {
        "label": "std_2",
        "status": "",
        "isRunning": false,
        "isStartNode": true
      },
      "sourcePosition": "bottom",
      "parentNode": "group_2",
      "extent": "parent"
    },
    {
      "id": "text_input_4",
      "type": "action",
      "initialized": false,
      "position": {
        "x": 37.498224753758166,
        "y": 53.637558990273874
      },
      "data": {
        "label": "text_input_4",
        "status": "",
        "isRunning": false,
        "isStartNode": false,
        "actionType": "default",
        "initialized": true
      },
      "sourcePosition": "bottom",
      "parentNode": "group_0",
      "extent": "parent"
    },
    {
      "id": "text_input_5",
      "type": "action",
      "initialized": false,
      "position": {
        "x": 45.889757093744606,
        "y": 43.39426528360525
      },
      "data": {
        "label": "text_input_5",
        "status": "",
        "isRunning": false,
        "isStartNode": false,
        "actionType": "default",
        "initialized": true
      },
      "sourcePosition": "bottom",
      "parentNode": "group_2",
      "extent": "parent"
    }
  ],
  "edges": [],
  "position": [-122.77951681025678, -45.614899860049604],
  "zoom": 0.9548415852086296,
  "viewport": {
    "x": -122.77951681025678,
    "y": -45.614899860049604,
    "zoom": 0.9548415852086296
  }
}
```

## 사용방법

- vue-flow/core 내 useVueFlow 스토어 중 toObject/fromObject 메서드를 사용
  - toObject - vueFlow내 생성된 모든 객체 정보 리턴
  - fromObject() - 객체정보를 받아 vueFlow 업데이트
- import(save)의 경우 우선 현재는 브라우저 내 localStorage에 vue-flow--save-restore key값으로 저장

```javascript
# SaveRestorControl.ts
import { useVueFlow, type FlowExportObject } from '@vue-flow/core'

const flowKey = 'vue-flow--save-restore'

const { toObject } = useVueFlow()

export const onSave = (flowObject: object) => {
localStorage.setItem(flowKey, JSON.stringify(flowObject))
}

export const onRestore = (): FlowExportObject | null => {
const flow = JSON.parse(localStorage.getItem(flowKey) ?? '')

if (flow) {
return flow
} else {
console.error('There are no Saveed Files')
return null
}
}

export const getFlowObject = () => {
return JSON.stringify(toObject())
}

# Main 측 사용

import { VueFlow, useVueFlow, isErrorOfType, ErrorCode, VueFlowError } from '@vue-flow/core'
import { onSave, onRestore, getFlowObject } from './vue-flow-file-control/SaveRestoreControl.ts'

const {
toObject,
fromObject,
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

```
