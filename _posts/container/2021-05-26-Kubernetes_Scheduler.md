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

#### 수동 스케줄링 - 원하는 포드를 원하는 노드에

- 포드 매뉴얼 스케줄링
  - 특수한 환경의 경우 특정 노드에서 실행되는 것을 선호하도록 포드를 제한
  - 일반적으로 스케줄러는 자동으로 합리적인 배치를 수행하므로 이러한 제한은 필요하지 않는다.
  - 더 많은 제어가 필요할 수 있는 몇가지 케이스
    - SSD가 있는 노드에서 포드가 실행하기 위한 경우
    - 블록체인이나 딥러닝 시스템을 위해 GPU 서비스가 필요한 경우
    - 서비스의 성능을 극대화하기 위해 하나의 노드에 필요한 포드를 모두 배치

- nodeName 필드를 사용한 매뉴얼 스케줄링
  - 포드를 강제로 원하는 node를 스케줄링 할 수 있음
  - spec 아래에 nodeName: work1 과 같이 노드 이름 설정
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: http-go
    spec:
      containers:
      - name: http-go
        image: gasbugs/http-go
      nodeName: work1
    ```

- 노드 셀렉터를 활용한 스케줄링
  - 특정 하드웨어를 가진 노드에서 포드를 실행하고자 하는 경우에 사용
  - GPU, SSD 등의 이슈를 가진 사항을 적용
  - 다음 명령어로 gpu=true 를 적용
    ```
    $ kubectl label node <node_name> gpu=true
    $ kubectl get node
    NAME        STATUS    ROLES    AGE   VERSION
    node_name   READY     gpu      66m   v1.12.8-gke.6
    ```
  - 다음 http-go-gqu.yaml을 작성하고 포드 생성
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: http-go
    spec:
      containers:
      - name: http-go
        image: gasbugs/http-go
      nodeSelector:
        gpu: "true"
    ```

#### 멀티풀 스케줄러

- 필요성
  - 기본 스케줄러가 사용자의 필요에 맞지 않으면 사용자 고유의 스케줄러를 구현 가능
  - 기본 스케줄러와 함께 여러 스케줄러를 동시에 실행 가능
  - 각 포드에 사용할 스케줄러를 지정하는 방식도 가능
  
  - 스케줄러의 사용옵션
    - https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler
  
  - 스케줄러를 실행하는 yaml 파일이 너무 크니 복붙으로 해결
    - https://Kubernetes.io/docs/tasks/administrater-cluster/configure-multiple/schedulers/
    - 이미지의 경우 에러 발생
    - 다음 이미지를 사용
    - k8s.gcr.io/kube-scheduler-amd64v:v1.11.3

- 예제: my-scheduler.yaml
  ```
  apiVersion: v1
  kind: ServiceAccount
  metadata:
    name: my-scheduler
    namespace: kube-system
  ---
  # 권한 부여
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: my-scheduler-as-kube-scheduler
  subjects:
  - kind: ServiceAccount
    name: my-scheduler
    namespace: kube-system
  roleRef:
    kind: ClusterRole
    name: system:kube-scheduler
    apiGroup: rbac.autorization.k8s.io
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
  # ...  
  ```

- 두 포드 스케줄링
  - 멀티 스케줄러를 사용해 두 포드를 스케줄
  - schedulerName에 default-scheduler를 작성하거나 아무 서술이 없는 경우 default-scheduler로 실행
  - 두 번째 포드는 my-scheduler 사용
    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: annotation-default-scheduler
      labels:
        name: multischeduler-example
    spec:
      schedulerName: default-scheduler
      containers:
      - name: pod-with-default-annotation-container
        image: k8s.gcr.io/pause:2.0
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: annotation-second-scheduler
      labels:
        name: multischeduler-example
    spec:
      schedulerName: my-scheduler
      containers:
      - name: pod-with-default-annotation-container
        image: k8s.gcr.io/pause:2.0
    ```
    - 두번째 Pod이 Pending이 된다. -> my-scheduler에 권한이 없기 때문에
    
- 클러스터롤에 권한이 부재
  - 클러스터에 API 권한을 부여하면 my-scheduler도 활성화 가능하다
  - 다음 명령어로 클러스터 롤을 편집
    ```
    $ kubectl edit clusterrole system:kube-scheduler
    ```
  - ResourceNames에 my-scheduler 추가
    ```
    resourceNames:
    - kube-scheduler
    - my-scheduler
    ```
    
#### 오토 스케일링 HPA

- 포드 스케일링의 두가지 방법
  - HPA: 포드 자체를 복제하여 처리할 수 있는 포드의 개수를 늘리는 방법
  - VPA: 리소스를 증가시켜 포드의 사용 가능한 리소스를 늘리는 방법
    - 아직 쿠버네티스에서는 지원하지는 않고 있고, 클라우드에서 제공하는 경우가 있다.
  - CA: 번외로 클러스터 자체를 늘리는 방법(노드 추가)

- HPA(Horizontal Pod Autoscaler)
  - 쿠버네티스에는 기본 오토스케일링 기능을 내장
  - CPU 사용률은 모니터링하여 실행된 포드의 개수를 늘리거나 줄인다

  - 설정 방법
    - 명령어를 사용하여 오토 스케일링 저장
      ```
      $ kubectl autoscale deployment my-ap --max 6 --min 4 --cpu-percent 50
      ```
      - 할당 받은 리소스에서 cpu-percent를 넘어가면 scaling한다.
      
    - HPA yaml을 작성하여 타겟 포드 지정
      ```
      apiVersion: autoscaling/v1
      kind: HorizontalPodAutoscaler
      metadata:
        name: myapp-hpa
        namespace: default
      spec:
        maxReplicas: 10
        minReplicas: 1
        scaleTargetRef:
          apiVersion: extensions/v1beta1
          kind: Deployment
          name: myapp
        targetCPUUtilizationPercentage: 30
      ```
      - 할당 받은 리소스에서 targetCPUUtilizationPercentage를 넘어가면 scaling한다.
      - scaleTargetRef에 설정한 Deployment를 한다.
  
- VPA와 CA는 어떻게하는가
  - 공식 쿠버네티스에서 제공하지는 않으나 클라우드 서비스에서 제공
  
  - CA(클러스터 자동 확장 처리)
    - https://cloud.google.com/kubernetes-engins/docs/concepts/cluster-autoscaler
      ```
      gcloud container clusters create example-cluster \
        --zone us-central1-a \
        --node-locations us-central1-a, us-central1-b, us-central1-f \
        --num-nodes 2 --enable-autoscalling --min-nodes 1 --max-nodes 4
      ```
  - VPA(수직형 POD 자동 확장
    - https://cloud.google.com/kubernetes-engine/docs/how-to/vertical-pod-autoscaling?hl=ko
      ```
      $ gcloud container clusters create [CLUSTER_NAME] --enable-vertical-pod-autoscaling
      $ gcloud container clusters update [CLUSTER_NAME] --enable-vertical-pod-autoscaling
      ```

#### 출처

- 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의
