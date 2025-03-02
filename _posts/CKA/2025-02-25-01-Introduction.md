---
title: 01. Certified Kubernetes Administrator(CKA)
author: Yeon Gyu Yang
date: 2025-02-25 00:00:00 +0800
categories: [Certification]
tags: [CKA ]
math: true
toc: true
mermaid: true
description: >-
  CKA에 대해 알아보자(feat. Udemy)
image:
  path: "https://image.inblog.dev/?url=https%3A%2F%2Ffgobbnslcbjgothosvni.supabase.co%2Fstorage%2Fv1%2Fobject%2Fpublic%2Fimages%2Ffeatured_image%2F2024-08-25T17%3A27%3A36.968Z-4e6c06c4-6921-46e5-9e24-888b926522af&w=750&q=75"
  alt: "CKA"
---

## CKA Intro
- 컨테이너를 관리 및 생성하는 가상 머신을 쿠버네티스
- Container Orchestration Tool로 요즘 산업에서 제일 많이 쓰이는 툴
- [Case Study](https://kubernetes.io/case-studies/) 에서 현재 다양한 기업체들에서 사용중인 다양한 사용 예들을 확인 가능
    - IBM 및 요즘 제일 인기 많은 분야 OpenAI 에서 사용되는 예를 확인 가능
        -   `Open AI 사용 예` : 7500개의 노드를 생성하고 컨테이너를 관리하는 방법의 예를 보이며 얼마나 확장 가능한지를 보임
- 위와 같은 업계의 인기로 업계에서 매우 인기 있는 자격증 중 하나는 인증된 Kubernetes Administrator인 `CKA`이다
- 다루는 범위는 아래와 같은 범위들을 전반적으로 다루며,
<img src="/assets/img/post/cka/1.png">
- 기본적인 도커에 대해 개념을 알고, 그 이후 쿠버네티스에 대해 다룬다

- GitHub Repository for CKA
> https://github.com/zealvora/certified-kubernetes-administrator

- Discord Channel Link
> https://kplabs.in/chat

- 초기 학습을 위해서 Managed Provider인 Digital Ocean 을 사용할 예정
  - AWS EKS는 유료이나, Digital Ocean은 Control Plane(Cluster) 구성 무료에 노드는 개당 12달러를 청구하나 현재 200달러 바우처를 제공함으로 Digital Ocean을 사용함
  - Digital Ocean Credit Referral Code
> https://m.do.co/c/74dcb0137794

---

## 실습 환경 셋팅
- kubectl 설치 [link](https://spacelift.io/blog/kubectl-auto-completion)
- kubectl auto completion 
> apt-get install bash-completion 
> kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
> echo 'alias k=kubectl' >>~/.bashrc
> echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
> source ~/.bashrc

##  Docker vs Kubernates
- `docker`
  - 컨테이너를 만들고 실행하는 기술
  - 실행단위 : 개별 컨테이너
  - 사용목적 : 개발 및 태스트 환경 구성

- `kubernetes`
  - 여러개의 노드(서버)에서 컨테이너를 효율적으로 배포하고 관리하는 오케스트레이션 도구를 말함
  - 실행단위 : 컨테이너를 묶은 클러스터
  - 사용목적 : 대규모 서비스 운영 및 배포


## 컨테이너 vs pod

> ✔ 컨테이너: 애플리케이션을 실행하는 독립적인 단위
{: .prompt-tip }

> ✔ Pod: 쿠버네티스에서 컨테이너를 감싸고 관리하는 최소 배포 단위
{: .prompt-tip }

> ✔ Pod이 컨테이너를 포함하고 있으며, 쿠버네티스를 통해 관리됨
{: .prompt-tip }

---