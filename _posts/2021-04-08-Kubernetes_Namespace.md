---
layout: post
title: Kubernetes Namespace
summary: Kubernetes
author: devhtak
date: '2021-04-08 21:41:00 +0900'
category: Container
---

#### Namespaces

- Namespaces란?
  - 하나의 클러스터에서도 Namespace로 나누어 자원, 권한등을 한정적으로 하여 안정적인 서비스를 제공한다.
  - 리소스를 각각의 분리된 영역으로 나누기 좋은 방법  
  - 여러 네임스페이스를 사용하면 쿠버네티스 시스템을 더 작은 그룹으로 분할
  - 멀티 테넌트(Multi-tenant) 환경을 분리하여 리소스를 생산, 개발, QA 환경 등으로 사용
  - 리소스 이름은 네임스페이스 내에서만 고유 명칭 사용
  
  - 현재 클러스터의 기본 네임스페이스 확인하기
    ```
    $ kubectl get ns    
    ```
    
- 각 네임스페이스 상세 내용 확인
  - kubectl get을 옵션없이 사용하면 default 네임스페이스에 질의
  - 다른 사용자와 분리된 환경으로 타인의 접근을 제한
  - 네임스페이스 별로 리소스 접근 허용과 리소스 양도 제어 가능
  - --namespace나 -n을 사용하여 네임스페이스 별로 확인 가능
    ```
    $ kubectl get pod --namespace kube-system
    ```

- YAML 파일로 네임스페이스 만들기
  - test_ns.yaml 파일을 생성하고 create를 사용하여 생성
    ```
    apiVersion: apps/v1
    kind: Namespace
    metadata:
      name: test-ns
    ```
    - 생성 및 확인
      ```
      $ kubectl create -f test_ns.yaml
      $ kubectl get ns
      ```
  - kubectl 명령어로 yaml 없이 바로 네임스페이스 생성 가능
    ```
    $ kubectl create namespace "test-namespace"
    ```

- 전체 네임스페이스 조회
  - 전체 네임스페이스를 대상으로 kubectl을 실행하는 방법
  ```
  $ kubectl get pod --all-namespaces
  ```
