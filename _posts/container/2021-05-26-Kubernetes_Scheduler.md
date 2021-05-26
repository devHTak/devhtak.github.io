---
layout: post
title: Kubernetes Scheduler
summary: Kubernetes
author: devhtak
date: '2021-05-26 21:41:00 +0900'
category: Container
---

#### 데몬셋

- 레플리케이션컨트롤러와 레플리카셋은 무작위 노드에 포드를 생성
- 데몬셋은 각 하나의 노드에 하나의 포드만을 구성
- kube-proxy가 데몬셋으로 만든 쿠버네티스에서 기본적으로 활동중인 포드

![image](https://user-images.githubusercontent.com/42403023/119589122-26382c80-be0d-11eb-8f66-a24292f9b26f.png)

** 이미지 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의

- 예제
  - kubectl apply -f https://k8s.io/examples/controllers/daemonset.yaml
    ```
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd-elasticsearch
      namespace: kube-system
      labels:
        k8s-app: fluentd-logging
    spec:
      slector:
        matchLabels:
          name: fluentd-elasticsearch
      template:
        metadata:
          labels:
            name: fluentd-elasticsearch
        spec:
          tolerations: # Master에 Pod이 올라가지 않는 것이 default인데, Master에도 올리도록 한다
          - key: node-role.kubernetes.io/master
            effect: NoSchedule
          containers:
          - name: fluentd-elasticsearch
            image: gcr.io/fluentd-elasticsearch/fluentd:v2.5.1
            resources:
              limits:
                memory: 200Mi
              requests:
                cpu: 100m
                memory: 200Mi
            volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
          terminationGracePeriodSeconds: 30
          volumes:  # hostPath를 지정해주어야 한다.
          - name: varlog
            hostPath: 
              path: /var/log
          - name: varlibdockercontainers
            hostPath:
              path: /var/lib/docker/containers
    ```

#### Static Pod

- 스태틱 포드의 필요성
  - 스태틱 포드: kubelet이 직접 실행하는 포드
  - 각각의 노드에서 kubelet에 의해 실행 (apiServer를 타지 않고 kubelet이 관리)
  - 포드들을 삭제할 때 apiserver를 통해서 실행되지 않은 스태틱 포드는 삭제 불가
  - 즉, 노드의 필요에 의해 사용하고자 하는 포드는 스태틱 포드로 세팅
  - 다음 명령어 들을 사용하여 실행하고자 하는 static pod의 위치 설정 가능

- 스태틱 포드의 기본 경로
  - 기본 경로는 /etc/kubernetes/manifests
  - 이미 쿠버네티스 시스템에서는 필요한 기능을 위해 다음과 같은 스태틱 포드를 사용
  - 각각의 컴포넌트의 세부 사항을 설정할 때는 여기 있는 파일들을 수정하면 자동으로 업데이터 되어 포드를 재구성
  - 포드의 작성 요쳥은 기존 포드와 동일
    ```
    $ ls /etc/kubernetes/manifests
    etcd.yaml   kube-apiserver.yaml   kube-controller-manager.yaml    kube-scheduler.yaml
    ```

- 간단한 스태틱 포드의 작성
  - 일반적인 포드와 동일하게 작성
  - 작성된 파일은 반드시 해당 경로에 위치
  - 실행을 위해 별도의 명령은 필요하지 않음
  - 작성 후 바로 get pod을 사용하여 확인
  
  ```
  $ vi /etc/kubernetes/manifests/static-pod.yaml
  
  apiVersion: v1
  kind: Pod
  metadata
    name: static-web
    labels:
      role: myrole
  spec:
    containers:
    - name: web
      image: nginx
      ports:
      - name: web
        containerPort: 80
        protocol: TCP
  ```

#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의
