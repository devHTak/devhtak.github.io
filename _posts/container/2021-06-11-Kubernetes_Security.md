---
layout: post
title: Kubernetes Security
summary: Kubernetes
author: devhtak
date: '2021-06-11 21:41:00 +0900'
category: Container
---

#### 보안을 위한 다양한 리소스

- 모든 통신은 TLS
  - 대부분 엑세스는 kube-apiserver를 통하지 않고서는 불가능하다.
  - 엑세스 가능한 유저
    - 파일 - 유저 이름과 토큰
    - Service-Accounts
    - 인증서(Certificates)
    - External Authentication Providers - LDAP
    
  - 무엇을 할 수 있는가?
    - RBAC Authorization
    - ABAC Authorization
    - Node Authorization
    - WebHook Mode
