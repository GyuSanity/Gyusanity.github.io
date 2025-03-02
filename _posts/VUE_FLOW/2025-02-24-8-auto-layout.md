---
title: 8. auto layout
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

## Auto Layout

- vue-flow내 생성된 node/edge 정보를 토대로 수직 또는 수평으로 Layout Align을 맞추는 기능

## 사용 방법

- Version1, 중첩된 노드를 고려하지 않음(Nested Node auto layout 지원 x)
- Node/Edge의 경우 dagre 라이브러리를 사용하여 그래프 객체를 사로 만들고, useVueFlow fitView action을 사용하여 auto layout 수행 ( Node/Edge position을 잡기위해 dagre를 사용하는 것으로 보이나 동작 원리는 추가 분석 필요 )

```javascript
# useLayout.js
import dagre from '@dagrejs/dagre'
import { Position, useVueFlow } from '@vue-flow/core'
import { ref } from 'vue'

/**
 * Composable to run the layout algorithm on the graph.
 * It uses the `dagre` library to calculate the layout of the nodes and edges.
 */
export function useLayout() {
  const { findNode } = useVueFlow()

  const graph = ref(new dagre.graphlib.Graph())

  const previousDirection = ref('LR')

  function layout(nodes, edges, direction) {
    // we create a new graph instance, in case some nodes/edges were removed, otherwise dagre would act as if they were still there
    const dagreGraph = new dagre.graphlib.Graph()

    graph.value = dagreGraph

    dagreGraph.setDefaultEdgeLabel(() => ({}))

    const isHorizontal = direction === 'LR'
    dagreGraph.setGraph({ rankdir: direction })

    previousDirection.value = direction

    for (const node of nodes) {
      // if you need width+height of nodes for your layout, you can use the dimensions property of the internal node (`GraphNode` type)
      const graphNode = findNode(node.id)

      dagreGraph.setNode(node.id, {
        width: graphNode.dimensions.width,
        height: graphNode.dimensions.height,
      })
      console.log(graphNode)
      console.log(dagreGraph)
    }

    for (const edge of edges) {
      dagreGraph.setEdge(edge.source, edge.target)
    }

    dagre.layout(dagreGraph)

    // set nodes with updated positions
    return nodes.map((node) => {
      const nodeWithPosition = dagreGraph.node(node.id)

      return {
        ...node,
        targetPosition: isHorizontal ? Position.Left : Position.Top,
        sourcePosition: isHorizontal ? Position.Right : Position.Bottom,
        position: { x: nodeWithPosition.x, y: nodeWithPosition.y },
      }
    })
  }

  function arrangeNode(nodes, edges) {
    // we create a new graph instance, in case some nodes/edges were removed, otherwise dagre would act as if they were still there
    const dagreGraph = new dagre.graphlib.Graph()

    graph.value = dagreGraph

    dagreGraph.setDefaultEdgeLabel(() => ({}))
    dagreGraph.setGraph({ rankdir: 'TB' })

    for (const node of nodes) {
      // if you need width+height of nodes for your layout, you can use the dimensions property of the internal node (`GraphNode` type)
      const graphNode = findNode(node.id)

      dagreGraph.setNode(node.id, {
        width: graphNode.dimensions.width || 150,
        height: graphNode.dimensions.height || 50,
      })
    }

    for (const edge of edges) {
      dagreGraph.setEdge(edge.source, edge.target)
    }

    // set nodes with updated positions
    return nodes.map((node) => {
      return {
        ...node,
      }
    })
  }

  return { graph, layout, previousDirection, arrangeNode }
}


# main
async function layoutGraph(direction) {
  await stop()

  nodes.value = layout(nodes.value, edges.value, direction)
  nextTick(() => {
    fitView()
  })
}

```

## Version2, 중첩된 노드를 고려한 Layout 정렬

- Nested Node인 경우를 고려한 Auto Layout
- Nested Node의 경우 노드 좌표는 부모 노드의 상대좌표로 환산 되어야함

> x = child.x - parent.x;
> y = child.y - parent.y
>
> > Ref. https://github.com/xyflow/xyflow/discussions/2968

- 부모-자식 노드 관계를 고려하여 Tree를 만들고 재귀호출 탐색(DFS)으로 layout도 정렬이 가능하지만, dagre-d3의 cluster 기능을 활용하여 x/y/width/height 값을 받아오고,
- 자식 노드일 경우는 부모 노드 좌표로 환산하여 업데이트 수행

```javascritp
# useLayout.js
import dagre from '@dagrejs/dagre'
import * as dagreD3 from 'dagre-d3'
import { Position, useVueFlow } from '@vue-flow/core'
import { ref } from 'vue'

export function useLayout() {
  const { findNode } = useVueFlow()

  const graph = ref(new dagre.graphlib.Graph())

  const previousDirection = ref('LR')

  function layout(nodes, edges, direction) {
    const dagreGraph = new dagreD3.graphlib.Graph({ compound: true })
      .setGraph({})
      .setDefaultEdgeLabel(function () {
        return {}
      })

    graph.value = dagreGraph

    dagreGraph.setDefaultEdgeLabel(() => ({}))

    const isHorizontal = direction === 'LR'
    dagreGraph.setGraph({ rankdir: direction, ranksep: 10 })

    previousDirection.value = direction

    for (const node of nodes) {
      const graphNode = findNode(node.id)

      dagreGraph.setNode(node.id, {
        width: graphNode.dimensions.width,
        height: graphNode.dimensions.height,
      })
    }

    for (const edge of edges) {
      dagreGraph.setEdge(edge.source, edge.target)
    }

    const groupNodes = nodes.filter((node) => node.parentNode)
    for (const groupNode of groupNodes) {
      dagreGraph.setParent(groupNode.id, groupNode.parentNode)
    }

    dagre.layout(dagreGraph)

    return nodes.map((node) => {
      const nodeWithPosition = dagreGraph.node(node.id)

      if (node.parentNode) {
        const parentNode = dagreGraph.node(node.parentNode)
        return {
          ...node,
          targetPosition: isHorizontal ? Position.Left : Position.Top,
          sourcePosition: isHorizontal ? Position.Right : Position.Bottom,
          computedPosition: {
            x: nodeWithPosition.x - nodeWithPosition.width / 2,
            y: nodeWithPosition.y - nodeWithPosition.height / 2,
          },
          position: {
            x:
              nodeWithPosition.x - nodeWithPosition.width / 2 - parentNode.x + parentNode.width / 2,
            y:
              nodeWithPosition.y -
              nodeWithPosition.height / 2 -
              parentNode.y +
              parentNode.height / 2,
          },
          style: {
            width: `${nodeWithPosition.width}px`,
            height: `${nodeWithPosition.height}px`,
          },
        }
      } else {
        return {
          ...node,
          targetPosition: isHorizontal ? Position.Left : Position.Top,
          sourcePosition: isHorizontal ? Position.Right : Position.Bottom,
          computedPosition: {
            x: nodeWithPosition.x - nodeWithPosition.width / 2,
            y: nodeWithPosition.y - nodeWithPosition.height / 2,
          },
          position: {
            x: nodeWithPosition.x - nodeWithPosition.width / 2,
            y: nodeWithPosition.y - nodeWithPosition.height / 2,
          },
          style: {
            width: `${nodeWithPosition.width}px`,
            height: `${nodeWithPosition.height}px`,
          },
        }
      }
    })
  }

  function arrangeNode(nodes, edges) {
    const dagreGraph = new dagre.graphlib.Graph()

    graph.value = dagreGraph

    dagreGraph.setDefaultEdgeLabel(() => ({}))
    dagreGraph.setGraph({ rankdir: 'TB' })

    for (const node of nodes) {
      const graphNode = findNode(node.id)

      dagreGraph.setNode(node.id, {
        width: graphNode.dimensions.width || 150,
        height: graphNode.dimensions.height || 50,
      })
    }

    for (const edge of edges) {
      dagreGraph.setEdge(edge.source, edge.target)
    }

    return nodes.map((node) => {
      return {
        ...node,
      }
    })
  }

  return { graph, layout, previousDirection, arrangeNode }
}

```
