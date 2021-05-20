---
layout: post
title: Kubernetes Volume
summary: Kubernetes
author: devhtak
date: '2021-04-15 21:41:00 +0900'
category: Container
---

#### Storage

- 볼륨
  - 컨테이너가 외부 스토리지에 액세스하고 공유하는 방법
  - 포드의 각 컨테이너에는 고유의 분리된 파일 시스템 존재
  - 볼륨은 포드의 컴포넌트이며 포드의 스펙에 의해 정의
  - 독립적인 쿠버네티스 오브젝트가 아니며 스스로 생성, 삭제 불가
  - 각 컨테이너의 파일 시스템의 볼륨을 마운트하여 생성

  - 볼륨 종류
    - 임시 볼륨: emptyDir    
    - 로컬 볼륨: hostpath, local
    - 네트워크 볼륨: iSCSI, NFS, cephFS, glusterFS, ..etc..
    - (클라우드 종속적)네트워크 볼륨: gcePersistentDisk, awsEBS, azuerFile, ..etc..

  - 주요 사용 가능한 볼륨의 유형
    - emptyDir: 일시적인 데이터 저장, 비어 있는 디렉터리
    - hostPath: 포드에 호스트 노드의 파일 시스템에서 파일이나 디렉토리를 마운트
    - nfs: 기존 NFS (네트워크 파일 시스템) 공유가 포드에 장착
    - gcePersistentDisk: 구글 컴퓨터 엔진(GCE) 영구 디스크 마운트
      - awsElasticBlockStore, azureDisk 또한 클라우드에 사용하는 형태
    - persistentVolumeClaim: 사용자가 특정 클라우드 환경의 세부 사항을 모른 체, GCE PersistenctDisk 또는 iSCSI 볼륨과 같은 내구성 스토리지를 요구(claim)할 수 있는 방법
    - configMap, Secret, downwardAPI: 특수한 유형의 볼륨
    - 참고: https://kubernetes.io/ko/docs/concepts/storage/volumes/

#### emptyDir: 임시 볼륨

- emptyDir을 활용한 파일 시스템 공유
  - 공유 스토리지가 없는 동일한 포드의 세 개의 컨테이너
  
    ![image](https://user-images.githubusercontent.com/42403023/114832790-d9266b00-9e09-11eb-936c-488e4613de09.png)
  
  - 두 개의 볼륨을 공유하는 세 개의 컨테이너
    
    ![image](https://user-images.githubusercontent.com/42403023/114832859-f0655880-9e09-11eb-8a4c-5b9c8d734e49.png)
    
  - 이미지 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의
  - 컨테이너가 3개 올라와있는 POD 하나의 상태
  - emptyDir을 사용하여 하나의 POD 내에서 다수의 컨테이너가 볼륨을 공유하도록 해준다.

- emptyDir 볼륨 사용하기
  - 볼륨을 공유하는 애플리케이션 생성
    ```
    $ vi count.sh
    #!/bin/bash
    trap "exit" SIGINT
    mkdir /var/htdocs
    
    SET=${seq 0 99999}
    
    for i in $SET
    do
        echo "Running loop seq " $i > /var/htdocs/index.html
        sleep 10
    done
    $ vi dockerfile
    FROM busybox:latest
    ADD count.sh /bin/count.sh
    ENTRYPOINT /bin/count.sh
    $ docker build -t gasbugs/count .
    $ docker push gasbugs/count
    ```
    
  - POD 생성
    ```
    $ vi count-pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: count
    spec:
      containers: # 2개 이상의 컨테이너를 생성
      - image: gasbugs/count
        name: html-generator
        volumeMounts: # 포드마다 마운트 설정
        - name: html
          mountPath: /var/htdocs # gasbugs/count에 index.html 위치와 동일
      - image: httpd
        name: web-server
        volumeMounts: # 포드마다 마운트 설정
        - name: html
          mountPath: /usr/local/apache2/htdocs # httpd에 설정된 path
          readOnly: true
      ports:
      - containerPort: 80
        protocol: TCP
      volumes: # 볼륨 설정
      - name: html # 이름은 volumeMount할 이름으로 사용
        emptyDir: {} # emptyDir 볼륨 설정
    $ kubectl create -f count-pod.yaml
    $ kubectl get pod -o wide
    NAME                      READY   STATUS    RESTARTS   AGE     IP            NODE       NOMINATED NODE   READINESS GATES
    count                     2/2     Running   0          8m      172.17.0.5    minikube   <none>           <none>
    http-go-ccb794f48-ff7rj   1/1     Running   0          3m33s   172.17.0.7    minikube   <none>           <none>
    $ kubectl exec -it http-go-ccb794f48-ff7rj -- curl 172.17.0.5
    Running loop seq 31
    # 임의의 http-go를 생성하여 실행, 
    ```

#### 로컬볼륨: hostPath

- 컨테이너와 노드간의 공유
- 노드의 파일 시스템에 있는 특정 파일 또는 디렉터리 지정
- 영구 스토리지
- 다른 노드의 포드끼리 데이터 공유는 안된다.

  ![image](https://user-images.githubusercontent.com/42403023/114838982-36bdb600-9e10-11eb-88f5-4f8ca9e9632b.png)

  - 이미지 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의

- hostPath 사용 현황 파악하기 - GCP에서..
  ```
  $ kubectl get pods --namespace kube-system
  NAME                               READY   STATUS    RESTARTS   AGE
  coredns-74ff55c5b-sxvfj            1/1     Running   14         14d
  fluentd-gcp-v3.2.0-fgc7k           1/1     Running   0          14d
  #...
  $ kubectl describe pod fluentd-gcp-v3.2.0-fgc7k --namespace kube-system
  #...
  Volumes:
    varLog:
      Type:   HostPath (bare host directory volume)
      Path:   /var/log
      HostPathType:
    varlidockercontainers:
      Type:   HostPath (bare host directory volume)
      Path:   /var/lib/docker/containers
  #...
  ```
  - fluentd : 외부에서 모니터링할 수 있는 data를 수집한다.
  - HostPath 는 내부에 파일시스템을 공유한다.
    - /var/log, /var/lib/docker/containers 는 노드의 local storage에 저장되어 있다.
    - /var/lib/docker/containers 에는 docker container에 대한 정보를 가지고 있다.

- 예제
  - work1에 디렉터리 생성 및 index.html 생성
    ```
    $ sudo mkdir /var/htdocs
    $ sudo -i
    $ echo "index - work1" > /var/htdocs/index.html
    ```
  - pod 생성 - volumes: hostPath
    ```
    $ vi hostpath-httpd.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: hostpath-http
    spec:
      containers:
      - image: httpd
        name: web-server
        volumeMounts:
        - name: html
          mountPath: /usr/local/apache2/htdocs
        ports:
        - containerPort: 80
          protocol: TCP
      volumes:
      - name: html
        hostPath: 
          path: /var/htdocs
          type: Directory
    $ kubectl create -f hostpath-httpd.yaml
    $ kubectl port-forward hostpath-http 8080:80
    ```
  - 127.0.0.1:8080으로 접속하여 index.html이 정상 접속하는지 확인해보자
  - 정상적으로 뜬다면 해당 POD이 work1 노드에 있다는 것을 확인할 수 있다.
    ```
    $ kubectl get pod -o wide
    ```

#### k8s와 NFS 네트워크 볼륨

- NFS 사용하기 위한 사전 작업
  - NFS 서버 설치
    ```
    $ apt-get update
    $ apt-get install nfs-common nfs-kernel-server portmap
    ```

  - 공유할 디렉터리 생성
    ```
    $ mkdir /home/nfs
    $ chmod 777 /home/nfs
    ```

  - /etc/exports 파일에 다음 내용 추가
    ```
    /home/nfs 10.0.2.15(rw,sync,no_subtree_check) 10.0.2.4(rw,sync,no_subtree_check) 10.0.2.5(rw,sync,no_subtree_check)
    $ service nfs-server restart
    $ showmount -e 127.0.0.1
    ```

  - NFS 클라이언트에서는 mount 명령어로 마운트해서 사용
    ```
    $ mount -t nfs 10.0.2.5:/home/nfs/mnt
    $ echo 'test' > /home/nfs/test.txt
    $ cat /mnt/test.txt
    test
    $ lst /home/nfs/text.txt
    조회되지 않는다.
    ```

- nfs-http.yaml 파일 실행
  - Work2에서 공유할 index.html 생성
    ```
    $ echo "test" > /home/nfs/index.html
    ```
  - Master에서 nfs_http.yaml 생성하여 실행
    ```
    $ vi nfs_http.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nfs-httpd
    spec:
      containers:
      - image: httpd
        name: web
        volumeMounts:
        - mountPath: /usr/local/apache2/htdocs
          name: nfs-volume
          readOnly: true
      volumes:
      - name: nfs-volume
        nfs:
          server: 10.0.2.5 #Worker2 IP
          path: /home/nfs
    $ kubectl create -f 
    ```
  - container가 뜨지 않았다. -> truble shooting
    - 오류 로그 확인
      ```
      $ kubectl describe pod nfs-httpd
      ```
      - /sbin/mount.<type> helper program을 확인하라는 오류 메시지 발견
    
    - 마스터에 NFS와 관련된 발생
      ```
      $ api-get update
      $ ap-get nfs-common nfs-kernel-server portmap
      ```

- 접속 확인
  ```
  $ kubectl port-forward nfs-httpd 8080:80
  $ 127.0.0.1:8080 접속 확인
  ```
  - 포트-포워딩에서 포트 사용으로 인한 오류발생하면 사용하지 않는 포트를 사용하거나 미사용 포트면 죽이자
    ```
    $ ps -eaf | grep port-forward
    $ kill -9 PID
    ```

#### PV와 PVC

- 포드 개발자가 클러스터에서 스토리지를 사용할 때 인프라를 알아야 할까?
- 실제 네트워크 스토리지를 사용하려면 알아야 한다.
- 애플리케이션을 배포하는 개발자가 스토리지 기술의 종류를 몰라도 상관없도록 하는 것이 이상적이다.
- 인프라 관련 처리는 클러스터 관리자의 유일한 도메인
- PV(persistent volume)와 PVC(persistent volume claim)를 사용하여 관리자와 사용자의 영역을 나눈다.

- PersistentVolume(PV) 와 PersistentVolumeClaim(PVC)
  - 인프라 세부 사항을 알지 못해도 클러스터의 스토리지를 사용할 수 있도록 제공해주는 리소스
  - 포드 안에 영구 볼륨을 사용하도록 하는 방법은 다소 복잡
  
  ![image](https://user-images.githubusercontent.com/42403023/118917327-8fcabd80-b96b-11eb-816f-647074053f96.png)

  ** 이미지 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의
  
- PV와 PVC 정의
  - PVC
    - PVC는 네임스페이스에 속하지 않는다.
    - mongo-pvc.yaml
    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mongodb-pvc // 클레임 사용 때 필요
    spec:
      resources:
        requests:
          storage: 1Gi // 요청하는 스토리지 양
      accessModes: // 단일 클라이언트에 읽기 쓰기 지원
      - ReadWriteOnce
      storageClassName: "" // 동적 프로비저닝에서 사용
    ```
    
  - PV
    - mongo-pv.yaml
    ```
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mongodb-pv
    spec:
      capacity:
        storage: 1Gi // PVC와 동일해야 한다.
      accessModes: // PVC와 연결시켜야 한다.
        - ReadWriteOnce // 단일 클라이언트에 읽기쓰기 기능
        - ReadOnlyMany // 여러 번 읽기만 가능
      persistentVolumeReclaimPolicy: Retain
      gcePersistentDisk: // 몽고 DB에 대한 정의
        pdName: mongodb
        fsType: ext4
    ```

    |Reclaiming|설명|
    |---|---|
    |Retain(유지)|PersistentVolumeClaim 삭제하면 PersistentVolume 여전히 존재하고 볼륨은 "해제된" 것으로 간주, 연관된 스토리지 자신의 데이터를 수동으로 정리|
    |Delete(삭제)|외부 인프라의 연관된 스토리지 자신을 모두 제거|
    |Recycle(재사용)|rm -rf /thevolume/* 볼륨에 대한 기본 스크립()을 수행하고 새 클레임에 대해 다시 사용할 수 있도록 함|
    
- PV와 PVC
  - 인프라 세부 사항을 알지 못해도 클러스터의 스토리지를 사용할 수 있도록 제공해주는 리소스
  - 포드 안에 영구 볼륨을 사용하도록 하는 방법은 다소 복잡하다.
  - 개발자는 포드(볼륨)와 PVC를 생성하면 된다.
  - PVC와 일치하는 PV(storage, accessmodel)가 있으면 GCE Storage를 연결할 수 있게 된다
    
- 실습
  - mongo-pod-pv-pvc.yaml
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: mongodb
  spec:
    containers:
    - image: mongo
      name: mongodb
      volumeMounts:
      - mountPath: /data/db
        name: mongodb
    volumes:
    - name: mongodb
      persistentVolumeClaim:
        claimName: mongo-pvc
  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: mongo-pvc
  spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 1Gi
    storageClassName: ""
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: mongo-pv
  spec:
    capacity:
      storage: 1Gi
    volumeMode: Filesystem
    accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
    persistentVolumeReclaimPolicy: Retain
    storageClassName: ""
    gcePersistentDisk:
      pdName: mongodb
      fsType: ext4
  ```
  
  - pv, pvc, pod 확인
    ```
    server1@server1-VirtualBox:~/yaml$ kubectl get pv
    NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
    mongo-pv   1Gi        RWO,ROX        Retain           Bound    default/mongo-pvc                           2m51s
    server1@server1-VirtualBox:~/yaml$ kubectl get pv
    NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM               STORAGECLASS   REASON   AGE
    mongo-pv   1Gi        RWO,ROX        Retain           Bound    default/mongo-pvc                           2m51s
    server1@server1-VirtualBox:~/yaml$ kubectl get pod
    NAME            READY   STATUS              RESTARTS   AGE
    mongodb         0/1     Running             0          4m3s
    ```
    - bound 상태이면 제대로 연결된 것

#### PV 동적 프로비저닝

- PV를 직접 만드는 대신 사용자가 원하는 PV 유형을 선택하도록 오브젝트 정의 가능

![image](https://user-images.githubusercontent.com/42403023/118922294-bc370780-b974-11eb-91c8-157a6bd53176.png)

- StorageClass yaml 파일 제작
  ```
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: storage-class
  provisioner: kubernetes.io/gce-pd // 프로비저닝에 사용할 플러그인 선택
  parameters:
    type: pd-ssd // 제공자에게 전달될 매개 변수
  ```
  - create 및 확인
    ```
    $ kubectl create -f storageclass.yaml
    storageclass.storage.k8s.io/storage-class created
    $ kubectl get sc    
    ```

- pvc 파일 제작
  - 포드와 PVC 모두 삭제 후 재 업로드(apply 명령어 시 권한 에러 발생)
  - mongo-pvc.yaml 변경
    ```
    storageClassName: "" -> storageClassName: storage-class
    ```

- PV 동적 프로비저닝을 사용하면 사용할 디스크와 PV가 자동으로 생성된다.
- PV 동적 프로비저닝 동작 순서 정리
  
  - mongo-pvc-pod.yaml (변경 사항 없음)
    ```
    volumes:
    - name: mongodb-data
      persistentVolumeClaim:
      claimName: mongodb-pvc
    ```
  - mongo-pvc.yaml(storage-class 이름 변경)
    ```
    ...
    metadata:
      name: mongodb-pvc
    ...
    spec:
      ...
      storageClassName: storage-class
    ...
    ```
  - storageclass.yaml (새로 생성)
    ```
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
      name: storage-class
    provisioner: kubernetes.io/gce-pd
    parameters:
      type: pd-ssd
    ```

#### Statefulset

- 애플리케이션의 상태를 저장하고 관리하는 데 사용되는 쿠버네티스 객체
- Statefulset으로 생성되는 포드는 영구 식별자를 가지고 상태를 유지시킬 수 있다.
- 사용 케이스
  - 안정적이고 고유한 네트워크 식별자가 필요한 경우
  - 안정적이고 지속적인 스토리지를 사용해야 하는 경우
  - 질서 정연한 포드의 배치와 확장을 원하는 경우
  - 포드의 자동 롤링업데이트를 사용하기 원하는 경우

- 문제점
  - Statefulset과 관련된 볼륨이 삭제되지 않기 때문에 관리가 필요하다.
  - 포드의 스토리지는 PV나 스토리지클래스로 프로비저닝 수행해야 한다.
  - 롤링업데이트를 수행하는 경우 수동으로 복구해야 할 수 있다.
    - 롤링 업데이트 수행 시 기존의 스토리지와 충돌로 인해 애플리케이션이 오류가 발생할 수 있다는 의미
  - 포드 네트워크 ID를 유지하기 위해 헤드레스(headless) 서비스 필요

- headless 서비스 작성 요령
  - statefulset의 포드 네트워크 ID를 유지하려면 headless 서비스를 작성해야 한다.
  - 헤드레스 서비스는 기존의 서비스를 생성하는 방법으로 만드는데, clusterIP를 None으로 지정하여 생성할 수 있다.
  - 헤드레스 서비스 자체에는 IP가 할당되지 않지만 헤드레스 서비스의 도메인 네임을 사용하여 각 포드에 접근할 수 있다.
  - 기존의 서비스와는 달리 kube-proxy가 밸런싱이나 프록시 형태로 동작하지 않는다.
    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx
    ```

- 작성 요령
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    ports:
    - port: 80
      name: web
    clusterIP: None
    selector:
      app: nginx
  ---

  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: web
  spec:
    selector:
      matchLabels:
        app: nginx
    serviceName: "nginx"
    replicas: 3
    template:
      metadata:
        labels:
          app: nginx
      spec:
        terminationGracePeriodSeconds: 10
        containers:
        - name: nginx
          image: k8s.gcr.io/nginx-slim:0.8
          ports:
          - containerPort: 80
            name: web
          volumeMounts:
          - name: www
            mountPath: /usr/share/nginx/html
    volumeClaimTemplates:
    - metadata:
        name: www
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: "standard"
        resources:
          requests:
            storage: 1Gi
  ```
  
  |속성|설명|
  |---|---|
  |serviceName|serviceName을 통해 연결하고자 하는 헤드레스 서비스를 지정한다. 그리고 당연히 연결하고자 하는 레이블도 일치시킨다.|
  |terminationGracePeriodSeconds|일반 포드에도 들어갈 수 있는 설정. 종료 요청(SIGTERM) 후 30초 기다린다. 그러나 컨테이너가 종료되지 않으면 강제로 SIGKILL을 발생시켜 컨테이너를 강제 종료한다. 0으로 세팅한 것은 프로세스가 정상 종료 루틴을 밟지 않게 하는 것이기 때문에 안전성에 좋지 않다. 프로그램을 설계하는 단계에서 SIGTERM을 받았을 때 적절한 처리 루틴을 밟도록 설계하는 것이 좋다.|
  |volumeClaimTemplates|안정적인 스토리지 제공을 위해 PVC를 작성하는 템플릿. 스테이트풀셋의 spec 아래에 작성한다. 스토리지 클래스를 지정하고 스토리지와 PV를 할당받는다.|
  |volumeMounts|영구 스토리지를 연결하고자 하는 위치|
  
- Statefulset의 다수 포드 식별 요령
  - replicas를 사용하여 다수의 포드를 생성할 수 있다.
  - 상태를 저장해야 하는 포드가 스케일링 기능을 제공하려면 레플리카셋처럼 랜덤한 문자열로 만들어져서는 안된다.
  - 스테이트풀셋은 순차적으로 포드를 하나씩 배포함으로써 상태를 유지할 수 있도록 0번부터 번호를 부여한다.
  
  |구분|순서|예|
  |---|---|---|
  |배포 순서|순차|0, 1, ... , n-1|
  |종료, 업데이트 순서|역순| n-1, ... , 1, 0|
  
- 업데이트 전략
  - 스테이트풀의 업데이트 전략은 두가지 있다.

  |업데이트 전략 옵션|특징|
  |---|---|
  |OnDelete|포드를 자동으로 업데이트하는 기능이 아니다. 수동으로 삭제해주면 스테이트풀 셋의 spec.template을 적용한 새로운 포드가 생성|
  |RollingUpdate|롤링 업데이트를 구현하면 한번에 하나씩 포드를 업데이트, 역순으로 진행한다.|
  
- 

** 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의
