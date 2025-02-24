---
title: 10. behavior tree
author: Yeon Gyu Yang
date: 2025-02-24 00:00:00 +0800
categories: [vue-flow]
tags: [vue-flow vue3 behaviorTree ]
math: true
toc: true
mermaid: true
description: >-
  ROS2에 대해 알아보자
image:
  path: "https://engineering.linecorp.com/wp-content/uploads/2018/10/11/1539226754921.png"
  alt: "behavior tree"

---

## Behavior Tree

- 트리 구조의 알고리즘으로, 에이전트(agent, ex. 로봇)의 행동을 모델링하고 제어하는 데 사용되며,
- Behavior Tree는 에이전트의 상태를 기반으로 다양한 행동(behavior)을 선택하고 실행하는 과정을 자동화할 수 있음

## Behavior Tree 구성 요소

- 기본적으로 자신의 상태를 리턴(success/fail)하는 세가지 종류의 노드로 구성

- `컨트롤 노드(Control Node)`: 트리의 구조를 정의하며, 하위 노드를 제어 ( Root Node는 컨트롤 노드라 생각하면 될듯함)
  - Sequence - 하위노드를 순차적으로 실행하나 성공하는 경우에만 다음 노드를 실행
  - Selector - 하위노드 결과 유무에 상관없이 전부 실행
  - RandomSelector - 하위노드 중 랜덤으로 하나만 실행
  - Parallel - 병렬 실행
- `리프 노드(Leaf Node)`: 실제 행동을 정의하며, 에이전트의 상태를 기반으로 특정 작업을 수행
  - Action - 로봇(agent)의 액션을 정의. (ex. Motion/TTS/Display/LED... )
  - Condition - 액션 혹은 서비스에 대한 성공 유무 확인
- `데코레이터 노드(Decorator Node)`: 하위 노드의 동작을 수정하거나 조건을 추가
  - UntilSuccess/AlwaysSuccess...

## Behavior Tree의 작동 과정

1. 루트 노드에서 시작하여, 하위 노드로 내려가며 행동을 결정

2. 각 노드는 자신의 상태를 평가하여, 성공(success), 실패(fail), 실행 중(running) 중 하나의 상태로 결정

3. 컨트롤 노드는 하위 노드의 상태를 기반으로 자신의 상태를 결정.

   ex. 시퀀스(Sequence) 노드는 하위 노드가 모두 성공할 때까지 실행하며, 셀렉터(Selector) 노드는 하위 노드 중 하나가 성공할 때까지 실행

4. 데코레이터 노드는 하위 노드의 상태를 수정하거나 조건을 추가하여, 에이전트의 행동을 제어함

5. 리프 노드는 실제 행동을 정의하며, 에이전트의 상태를 기반으로 특정 작업을 수행함
