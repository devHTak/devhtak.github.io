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
    NAME              STATUS   AGE
    default           Active   7d1h
    kube-node-lease   Active   7d1h
    kube-public       Active   7d1h
    kube-system       Active   7d1h 
    ```
    - Namespace를 설정하지 않으면 default를 사용하게 된다.
    
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
    $ kubectl create ns "test-ns"
    namespace/test-ns created
    ```
    ```
    $ kubectl create ns test-ns2 --dry-run=client -o yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      creationTimestamp: null
      name: test-ns2
    spec: {}
    status: {}
    ```
    - --dry-run=client: 실행하지 않고, 문법이 맞는지 확인한다.
    - -o yaml: yaml 파일까지 만들어 준다.
      ```
      kubectl create ns test-ns3 -o yaml > test-ns3.yaml
      ```

- 전체 네임스페이스 조회
  - 전체 네임스페이스를 대상으로 kubectl을 실행하는 방법
    ```
    $ kubectl get pod --all-namespaces
    NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
    kube-system   coredns-74ff55c5b-sxvfj            1/1     Running   10         7d1h
    kube-system   etcd-minikube                      1/1     Running   10         7d1h
    kube-system   kube-apiserver-minikube            1/1     Running   10         7d1h
    kube-system   kube-controller-manager-minikube   1/1     Running   12         7d1h
    kube-system   kube-proxy-pdqqx                   1/1     Running   10         7d1h
    kube-system   kube-scheduler-minikube            1/1     Running   10         7d1h
    kube-system   storage-provisioner                1/1     Running   19         7d1h
    ```
  
  - 특정 namespace에 해당하는 deploy, rs, pod 얻기
    ```
    $ kubectl get all -n test-ns
    $ kubectl get pod -n kube-system
    NAME                               READY   STATUS    RESTARTS   AGE
    coredns-74ff55c5b-sxvfj            1/1     Running   10         7d1h
    etcd-minikube                      1/1     Running   10         7d1h
    kube-apiserver-minikube            1/1     Running   10         7d1h
    kube-controller-manager-minikube   1/1     Running   12         7d1h
    kube-proxy-pdqqx                   1/1     Running   10         7d1h
    kube-scheduler-minikube            1/1     Running   10         7d1h
    storage-provisioner                1/1     Running   19         7d1h
    ```

- namespace 할당하기
  - namespace를 주기 위해 create할 때 -n 옵션으로 줄 수 있다.
    ```
    $ kubectl create -f deploy-jenkins.yaml -n test-ns
    ```
  - YAML 파일에 namespace 할당 가능하다.
    ```
    metadata:
      name: test
      namespace: test-ns
    ```
  
- default 네임스페이스가 아닌 특정 네임스페이스로 변경하기
  - ~/.kube/config 를 할당하면 된다.
    ```
    contexts:
    - context:
        cluster: kubernetes
        user: kubernetes-admin
        namespace: test-ns
    ```
    - contexts.context.namespace 추가

- 네임스페이스 삭제하기
  ```
  $ kubectl delete ns test-ns
  ```
  - test-ns에 해당하는 것들이 모두 사라진다.

** 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터
