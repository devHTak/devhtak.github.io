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

** 출처: 데브옵스(DevOps)를 위한 쿠버네티스 마스터 강의