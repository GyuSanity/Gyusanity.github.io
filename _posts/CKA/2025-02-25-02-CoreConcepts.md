---
title: 02. [CKA]Core Concepts
author: Yeon Gyu Yang
date: 2025-02-25 00:00:00 +0800
categories: [Certification]
tags: [CKA ]
math: true
toc: true
mermaid: true
description: >-
  core concept 명령어에 대해 알아보자
image:
  path: "https://image.inblog.dev/?url=https%3A%2F%2Ffgobbnslcbjgothosvni.supabase.co%2Fstorage%2Fv1%2Fobject%2Fpublic%2Fimages%2Ffeatured_image%2F2024-08-25T17%3A27%3A36.968Z-4e6c06c4-6921-46e5-9e24-888b926522af&w=750&q=75"
  alt: "CKA"
---

## Core concept 명령어(kubectl 기본 명령어)
- `kubectl get nodes --kubeconfig [config]` : cluster config 파일 설정(DNS/IP or Authentication)

> kube config default path : ~/.kube/config
{: .prompt-tip }

## Domain2 (Core Concepts)

- `kubectl run nginx --images=nginx` : nginx pod 실행
- `kubectl get pods [-o wide]` : 실행중인 pods 확인
- `kubectl get nodes [-o wide]` : 실행중인 노드 정보 확인
- `kubectl logs <pod name>`: pod 로그 조회
- `kubectl describe pod <pod name>` : pod 디테일 정보 확인
- `kubectl exec run --it <pod name> -- bash` : 실행중인 pod access
- `kubectl delete pod <pod name>` : 생성된 container/pod 삭제
- `kubectl delete pods --all` : 생성된 모든 pods들 삭제
- `kubectl apply -f <manifest file>` : manifest 파일을 통해 노드를 생성하는 명령어
- `kubectl delete -f <manifest file>` : manifest 파일을 통해 생성된 노드를 삭제하는 명령어
- `kubectl api-resources` : 쿠버네티스에서 사용가능한 리소스 api들 확인 가능
- `kubectl run nginx --image=nginx --dry-run=clinet` : 검증용으로 사용하는 명령어(실질적인 object는 생성되지 않음)
- `kubectl run nginx --image=nginx -o yaml` : 사용되는 yaml 파일 format 을 확인 가능(dry run 옵션과 함께 사용하여 manifest 파일로 가져다 사용해도 됨)
- `kubectl exec -it <pod name> -c <container-name> -- bash` : multiple container로 실행된 pod 내 특정 continaer 접근 명령어( `옵션 c` 가 필요함 )
- `kubectl run nginx --image=nginx --command -- <command> <args>` : manifest에 정의된 command/args를 무시하고 인자로 전달한 명령어/인자를 컨테이너 실행시 실행
- `kubectl explain <object>` : CLI 로 DOC 문서 확인 (하위 필드는 .으로 검색 ex. kubectl explain Pod.kind)