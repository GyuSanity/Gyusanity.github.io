---
title: 9. runprocess animation
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

## RunProcess

- Node 별 Process 상태 관리(ERROR/SKIPPED/CANCLED/FINISHED/RUNNING)와 노드를 순회하며 실행 상태를 모니터링 할 수 있는 애니메이션
- FlowEditor에 구성된 노드의 구성 중 시작 노드를 찾은 후 연결된 엣지를 순회함

## 사용방법

- useRunProcess.js 내 run/stop/isRunnning 함수를 버튼 이벤트로 걸어 사용가능

```javascript
> useRunProcess 함수 내에서 RunProcess 상태를 관리
const isRunning = ref(false)
const runningTasks = new Map()
const executedNodes = new Set()
const upcomingTasks = new Set()

>  async run(nodes) 함수를 통해 노드 상태 초기화 후 시작노드들을 찾고 promise로 병렬 실행
// get all starting nodes (nodes with no predecessors)
const startingNodes = nodes.filter((node) => graph.value.predecessors(node.id)?.length === 0)

// run the process on all starting nodes in parallel
await Promise.all(startingNodes.map((node) => runNode(node.id, true)))

> 프로미스로 등록된 runNode 함수드을 실행하며 다음 노드를 찾고, 순차적으로 순회하면서 실행

```

```javascript
# FlowCanvas.vue
>> script
import { useRunProcess } from "./animation/useRunProcess";
const { run, stop, reset, isRunning } = useRunProcess({ graph, cancelOnError });
const onPlayButtonClicked = () => {
  onResetButtonClicked();
  nodes.value = arrangeNode(nodes.value, edges.value);
  run(nodes.value);
};
>> template
<div v-if="isRunning" class="stop-btn icon-style" @click="stop">⏹️</div>
<div v-else @click="onPlayButtonClicked" class="icon-style">▶️</div>


# useRunProcess.js
import { ref, toRef, toValue } from 'vue'
import { useVueFlow } from '@vue-flow/core'

export const ProcessStatus = {
  ERROR: 'error',
  SKIPPED: 'skipped',
  CANCELLED: 'cancelled',
  FINISHED: 'finished',
  RUNNING: 'running',
}

/**
 * Composable to simulate running a process tree.
 *
 * It loops through each node, pretends to run an async process, and updates the node's data indicating whether the process has finished.
 * When one node finishes, the next one starts.
 *
 * When a node has multiple descendants, it will run them in parallel.
 *
 * @param options
 * @param options.graph The graph object containing the nodes and edges.
 * @param options.cancelOnError Whether to cancel the process if an error occurs.
 */
export function useRunProcess({ graph: dagreGraph, cancelOnError = true }) {
  const { updateNodeData, getConnectedEdges } = useVueFlow()

  const graph = toRef(() => toValue(dagreGraph))

  const isRunning = ref(false)

  /** Map of running tasks with the node ID as the key and the timeout as the value. */
  const runningTasks = new Map()

  /** Set of node ids of nodes that have been executed  */
  const executedNodes = new Set()

  /** Set of node ids yet to be executed */
  const upcomingTasks = new Set()

  /**
   * Run the process on a node.
   * It will mark the node as running, simulate an async process, and then mark the node as finished or errored.
   *
   * @param nodeId The ID of the node to run.
   * @param isStart Whether this is a starting node.
   */
  async function runNode(nodeId, isStart = false) {
    if (executedNodes.has(nodeId)) {
      return
    }

    // save the upcoming task in case it gets cancelled before we even start it
    upcomingTasks.add(nodeId)

    // get all incoming edges to this node
    const incomers = getConnectedEdges(nodeId).filter((connection) => connection.target === nodeId)

    // wait for edge animations to finish before starting the process
    await Promise.all(incomers.map((incomer) => until(() => !incomer.data?.isAnimating)))

    // remove the upcoming task since we are about to start it
    upcomingTasks.clear()

    if (!isRunning.value) {
      // The process was stopped
      return
    }

    // mark the node as executed, so it doesn't run again
    executedNodes.add(nodeId)

    updateNodeStatus(nodeId, ProcessStatus.RUNNING)
    updateNodeIsRunning(nodeId, true)

    // simulate an async process with a random timeout between 1-2 seconds
    // const delay = Math.floor(Math.random() * 2000) + 1000;
    const delay = 1000

    return new Promise((resolve) => {
      const timeout = setTimeout(
        async () => {
          // get all children of this node
          const children = graph.value.successors(nodeId) || []

          // // randomly decide whether the node will throw an error
          // const willThrowError = Math.random() < 0.15;

          // // we avoid throwing an error on the starting node
          // if (!isStart && willThrowError) {
          //   updateNodeStatus(nodeId, ProcessStatus.ERROR);

          //   // if cancelOnError is true, we stop the process and mark all descendants as skipped
          //   if (toValue(cancelOnError)) {
          //     await skipDescendants(nodeId);
          //     runningTasks.delete(nodeId);

          //     resolve(true);
          //     return;
          //   }
          // }

          updateNodeStatus(nodeId, ProcessStatus.FINISHED)
          updateNodeIsRunning(nodeId, false)

          runningTasks.delete(nodeId)

          if (children.length > 0) {
            // run the process on the children in parallel
            await Promise.all(children.map((child) => runNode(child)))
          }

          resolve(true)
        },
        // if this is a starting node, we don't want to wait
        isStart ? 0 : delay,
      )

      // save the timeout so we can cancel it if needed
      runningTasks.set(nodeId, timeout)
    })
  }

  /**
   * Run a sequence of nodes.
   * It will start with the nodes that have no predecessors and then run the process on each node in sequence.
   * If a node has multiple descendants, it will run them in parallel.
   * If an error occurs, it will stop the process and mark all descendants as skipped.
   * If cancelOnError is true, it will stop the process if an error occurs.
   * If the process is stopped, it will mark all running nodes as cancelled.
   *
   * @param nodes The nodes to run.
   */
  async function run(nodes) {
    // if the process is already running, we don't want to start it again
    if (isRunning.value) {
      return
    }

    // reset all nodes to their initial state
    reset(nodes)

    isRunning.value = true

    // get all starting nodes (nodes with no predecessors)
    const startingNodes = nodes.filter((node) => graph.value.predecessors(node.id)?.length === 0)

    // run the process on all starting nodes in parallel
    await Promise.all(startingNodes.map((node) => runNode(node.id, true)))

    clear()
  }

  /**
   * Reset all nodes to their initial state.
   *
   * @param nodes The nodes to reset.
   */
  function reset(nodes) {
    clear()

    for (const node of nodes) {
      updateNodeStatus(node.id, null)
      updateNodeIsRunning(node.id, null)
    }
  }

  /**
   * Skip all descendants of a node.
   *
   * @param nodeId The ID of the node to skip descendants for.
   */
  async function skipDescendants(nodeId) {
    const children = graph.value.successors(nodeId) || []

    for (const child of children) {
      updateNodeStatus(child, ProcessStatus.SKIPPED)
      updateNodeIsRunning(nodeId, false)
      await skipDescendants(child)
    }
  }

  /**
   * Stop the process.
   *
   * It will mark all running nodes as cancelled and skip all upcoming tasks.
   */
  async function stop() {
    isRunning.value = false

    for (const nodeId of upcomingTasks) {
      clearTimeout(runningTasks.get(nodeId))
      runningTasks.delete(nodeId)
      updateNodeStatus(nodeId, ProcessStatus.CANCELLED)
      updateNodeIsRunning(nodeId, false)
      await skipDescendants(nodeId)
    }

    for (const [nodeId, task] of runningTasks) {
      clearTimeout(task)
      runningTasks.delete(nodeId)
      updateNodeStatus(nodeId, ProcessStatus.CANCELLED)
      updateNodeIsRunning(nodeId, false)
      await skipDescendants(nodeId)
    }

    executedNodes.clear()
    upcomingTasks.clear()
  }

  /**
   * Clear all running tasks and executed nodes.
   */
  function clear() {
    isRunning.value = false
    executedNodes.clear()
    runningTasks.clear()
  }

  /**
   * Update the status of a node.
   *
   * @param nodeId The ID of the node to update.
   * @param status The new status of the node.
   */
  function updateNodeStatus(nodeId, status) {
    updateNodeData(nodeId, { status })
  }

  function updateNodeIsRunning(nodeId, isRunning) {
    updateNodeData(nodeId, { isRunning })
  }

  return { run, stop, reset, isRunning }
}

async function until(condition) {
  return new Promise((resolve) => {
    const interval = setInterval(() => {
      if (condition()) {
        clearInterval(interval)
        resolve(true)
      }
    }, 100)
  })
}

```
