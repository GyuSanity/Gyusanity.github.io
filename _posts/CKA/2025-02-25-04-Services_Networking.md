---
title: 04. [CKA]Services and Networking
author: Yeon Gyu Yang
date: 2025-03-02 00:00:00 +0800
categories: [Certification]
tags: [ CKA ]
math: true
toc: true
mermaid: true
description: >-
  Service에 대해 알아보자
image:
  path: "https://image.inblog.dev/?url=https%3A%2F%2Ffgobbnslcbjgothosvni.supabase.co%2Fstorage%2Fv1%2Fobject%2Fpublic%2Fimages%2Ffeatured_image%2F2024-08-25T17%3A27%3A36.968Z-4e6c06c4-6921-46e5-9e24-888b926522af&w=750&q=75"
  alt: "CKA"
---

- 클러스터로 생성된 노드내 할당된 pod의 경우 각자 Private IP를 할당 받으며, 이 Private IP를 통해 Pod간 서로 통신이 가능함
- pod의 경우 dynamic하게 관리되며 스케쥴링되며, Private IP가 바뀔수가 있음으로 하드코딩해서 관리할순 없음 

<img src="/assets/img/post/cka/10.png">

- 따라서 `Service`가 Gateway 역할을 수행하여 들어오는 요청을 처리하며, 일부 backend pod이 종료되어도 이를 인식하여 롸우팅에서 제외 시켜 동작함.
- `Service` 내 Port/TargetPort를 통해 Incomming/outgoing port를 분리시켜서 서비스 define이 가능함

<img src="/assets/img/post/cka/11.png">

- `Service`를 이용하면 Kubernetes cluster 외부 유저들도 `External IP`를 사용하여 Internal Pods에 접근이 가능함

<img src="/assets/img/post/cka/12.png">
<img src="/assets/img/post/cka/13.png">


- `Service` 정리 및 종류

<img src="/assets/img/post/cka/14.png">
<img src="/assets/img/post/cka/15.png">



- `kubectl get service` : cluster 내 구성된 service(gateway) info 확인
- `kubectl run backend-pod-3 --image-nginx --labels=type=webserver` : 라벨 정보와 함께 노드 생성하는 명령어

### Service 실습

- `kubectl run be --image=nginx` :  be 실행 예제
- `kubectl run fe --image=ubuntu --command -- sleep 36000` : fe 실행 예제 
- `kubectl exec -it fe -- bash` : fe 실행 (curl이 없음으로 설치 필요 apt  update -y && apt install curl -y)
- `kubectl create -f service.yaml` : [service.yaml](https://kubernetes.io/ko/docs/concepts/services-networking/service/) example 파일로 service 생성
- `kubectl create -f endpoint.yaml` : [endpoint.yaml](https://github.com/zealvora/certified-kubernetes-administrator/blob/master/Domain%203%20-%20Services%20and%20Networking/service-endpoints.md) example 파일로 endpoint 생성
- `kubectl get endpoints` : 생성한 endpoint 정보 확인
- `kubectl describe service <service name>` : 생성된 서비스 정보 조회(endpoint kind로 service에 endpoints가 연결된것을 확인 가능)

<img src="/assets/img/post/cka/16.png">

## Selector를 활용한 Service Endpoint 등록 방법

- 실습에서 확인했지만 endpoint를 하드코딩에서 직접 입력하는 방식은 pod 관리에 적절하지 않음
- 따라서 `selector`를 활용하여 endpoint를 설정 가능하며, 관리하고자 하는 pod에 label을 추가해둬야함

## Service Type(ClusterIP)
- `Service`는 위에서 알아봤듯이 incomming Traffic을 적절한 endpoint로 포워딩 해주는 gate 역할을 수행하며,
- `ClusterIP`는 Kubernates에서 제공하는 service type의 `default` 형태이며, Internal IP만을 제공하여 같은 Kubernetes cluster 내에서만 유효한 IP를 가짐
- `ClusterIp`는 VirtualIP이며, 서비스가 유지되는한 IP는 변경되지 않음

### 실습 명령어
- `kubectl create service --help ` : service 명령어 사용 종류 방법 확인 가능
- `kubectl create service clusterip <name> --tcp=<port>:<targetport> --dry-run=client -o yaml` : clusterip 서비스 타입 yaml 파일 출력

## Service Type(NodePort)

- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `

## Service Type(LoadBalancer)

- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `