---
title: 5. nested node
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

## Nested Node ( https://vueflow.dev/examples/nodes/nesting.html )

- 노드간에 부모노드 자식노드 관계를 만들고, 자식노드들을 부모 노드 영역 내에서 관리하도록 해줌

## 생성 방법

- 생성할 노드에 아래 필드를 넣고 생성하면 됨
  - parentNode = 부모노드 ID
  - extent = 자식노드는 부모 노드 내에서만 움직임
  - expandParent = 자식노드를 움직일 경우 부모노드의 영역도 확장시킬수 있음

## 코드 구현

- extent만 사용하여 drop시 노드가 추가되도록 구현

```javascript
function onDrop(event) {
  const position = screenToFlowCoordinate({
    x: Math.floor(event.clientX),
    y: Math.floor(event.clientY),
  });

  const nodeId = getId(draggedType.value);
  let newNode = null;
  switch (draggedType.value) {
    case CustomTypesDefine.START_NODE:
      newNode = {
        id: nodeId,
        type: draggedType.value,
        position,
        sourcePosition: Position.Bottom,
        data: {
          label: nodeId,
          status: "",
          isRunning: false,
          isStartNode: true,
        },
        initialized: true,
      };
      break;
    case CustomTypesDefine.INPUT_NODE:
      newNode = {
        id: nodeId,
        type: draggedType.value,
        position,
        sourcePosition: Position.Bottom,
        data: {
          label: nodeId,
          status: "",
          isRunning: false,
          isStartNode: false,
          actionType: actionType.value,
          initialized: true,
        },
      };
      break;
    case CustomTypesDefine.GROUP_NODE:
      newNode = {
        id: nodeId,
        type: draggedType.value,
        position,
        data: { label: nodeId },
        style: {
          backgroundColor: "rgba(246, 92, 92, 0.49)",
          height: "200px",
          width: "400px",
        },
        initialized: true,
      };
      break;
    default:
      newNode = {
        id: nodeId,
        type: draggedType.value,
        position,
        data: { label: nodeId },
      };
      break;
  }

  console.log("[Drop Position]", position);
  const parentNode = findParentNode(position.x, position.y);
  console.log("parentNode ", parentNode);

  if (parentNode) {
    switch (draggedType.value) {
      case CustomTypesDefine.START_NODE:
      case CustomTypesDefine.INPUT_NODE:
        newNode.parentNode = parentNode.id;
        newNode.extent = `parent`;
        // newNode.expandParent = true
        break;
      case CustomTypesDefine.GROUP_NODE:
        newNode.parentNode = parentNode.id;
        newNode.extent = `parent`;
        break;
      default:
        break;
    }
    const { off } = onNodesInitialized(() => {
      updateNode(nodeId, (node) => ({
        computedPosition: {
          x: position.x,
          y: position.y,
          z: 0,
        },
        position: {
          x:
            position.x -
            parentNode.computedPosition.x -
            node.dimensions.width / 2,
          y:
            position.y -
            parentNode.computedPosition.y -
            node.dimensions.height / 2,
        },
      }));
      off();
    });
  } else {
    /**
     * Align node position after drop, so it's centered to the mouse
     *
     * We can hook into events even in a callback, and we can remove the event listener after it's been called.
     */
    const { off } = onNodesInitialized(() => {
      updateNode(nodeId, (node) => ({
        position: {
          x: node.position.x - node.dimensions.width / 2,
          y: node.position.y - node.dimensions.height / 2,
        },
      }));

      off();
    });
  }

  addNodes(newNode);
}
```
