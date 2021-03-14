---
layout: post
title: Docker Volumn
summary: Docker
author: devhtak
date: '2021-03-11 21:41:00 +0900'
category: Container
---

#### 도커 볼륨

- 도커 이미지로 컨테이너를 생성하면 이미지는 읽기 전용이 되며 컨테이너의 변경 사항만 별도로 저장하여 각 컨테이너의 정보를 보존한다.
- 이미 생성된 이미지는 어떠한 경우로도 변경되지 않으며, 컨테이너 계층에 원래 이미지에서 변경된 파일 시스템 등을 저장한다.
- 이미지에 컨테이너를 실행하기 위한 파일이 들어있고, 컨테이너 계층에는 애플리케션이 실행 중에 생겨나는 데이터가 저장된다.
- 여기에 치명적인 단점이 있는데, 컨테이너가 삭제되면, 해당 데이터 또한 삭제된다.
- 이를 해결하기 위한 벙법 중 하나가 볼륨이다.

- 볼륨 활용 방법
	- 호스트와 볼륨을 공유할 수 있다.
	- 볼륨 컨테이너를 활용할 수 있다.
	- 도커과 관리하는 볼륨을 생성할 수 있다.

- 호스트와 볼륨을 공유함으로써 DB 컨테이너를 삭제해도 데이터는 삭제되지 않도록 설정
	- mysql 데이터베이스 컨테이너 실행
  ```
  $ docker run -d \
  --name wordpress_hostvolume \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=wordpress\
  -v /home/wordpress_db:/var/lib/mysql \
  mysql:5.7
  ```

	- 워드프레스 웹 서버 컨테이너 실행
	```
	$ docker run -d \
	-e WORDPRESS_DB_PASSWORD=password\
	--name wordpress_hostvolume \
	--link wordpressdb_hostvolume:mysql\
	-p 80 \
	wordpress
	```

	- -v /home/wordpress_db:/var/lib/mysql
		- 호스트의 /home/wordpress_db 디렉터리 컨테이너의 /var/lib/mysql 디레터리를 공유한다는 뜻
	- wordpress_hostvolume 컨테이너를 삭제해도, 컨테이너의 /var/lib/mysql 디렉터리는 호스트의 /home/workpress_db 디렉터리에 완전히 같은 디렉터리로 데이터가 저장되어 있다.
	- 호스트의 볼륨 디렉터리는 설정할 때에 존재하지 않으면 생성한다.
	- 호스트의 볼류 디렉터리가 설정할 때에 이미 존재하고 있었다면, 하위 데이터를 지우고 연결한 디렉터리의 파일을 가져온다.

#### 볼륨 컨테이너

	-  볼륨을 사용하는 컨테이너를 다른 컨테이너와 공유
	- --volumes_from 옵션을 설정하면 -v 또는 --volume 옵션을 적용한 커테이너의 볼륨 디렉터리를 공유할 수 있다.
	- 직접 볼륨을 공유하는 것이 아닌 -v 옵션을 적용한 컨테이너를 통해 공유

	```
	$ dockerrun -i -t \
	--name volume_override \
	-v /home/wordpress_db:/home/testdir_2 \ 
	alicek106/volume_test

	$ docker run -i -t \
	--name volumes_from_container \
	--volumes-from volume-override \
	ubuntu:14.04
	```
  
	- 미리 생성한 volume_override 컨테이너에 볼륨을 공유받는 경우
	- 앞에서생성한 volume_override 컨테이너는 /home/testdir_2 디렉터리를 호스트와 공유하고 있으며, 이 컨테이너를 볼류 컨테이너로서 volumes_from_container 컨테이너에 다시 공유한다.
	- 여러개의 컨테이너가 동일한 컨테이너에 --volumes-from 옵션을 사용함으로써 볼륨을 공유하여 사용할 수도 있다.

#### 도커 볼륨

	- 도커 자체에서 제공하는 볼륨 기능을 활용하여 데이터 보존
	- docker volume 명령어로 시작한다.
		```
		$ docker volume create --name myvolume
		$ docker volume ls
		```
    
		- ls 명령에서 driver 가 로컬로 설정되어 있으면 해당 볼륨은 로컬 호스트에 저장되며 도커 엔진에 의해 생성되고 삭제된다.
	
  - -v 옵션을 이용하여 호스트와 볼륨을 공유한다.
		```
    [볼륨의 이름]:[컨테이의 공유 디렉터리]
		```
    
		```
		$ docker run -i -t --name myvolume-1 \
		-v myvolume:/root/ \
		ubuntu:14.0.4
		```
	
  - docker inspect --type volume myvolume 살펴뵉
		- driver: 볼륨이 사용하는 드라이버
		- label: 볼륨을 구분하는 라벨
		- mounpoint: 해당 볼륨이 실제로 호스트의 어디에 저장했는지를 의미
	
  - 도커 볼륨을 생성하지 않아도 -v 옵션을 사용하면 해당 디렉터리에 대한 볼륨을 자동으로 생성한다.
		- -v /root
	
  - 도커 볼륨을 사용하고 있는 컨테이너를 삭제해도 볼륨이 자동으로 삭제되지 않는다.
	
  - 사용하지 않는 볼륨을 삭제 하려면 docker volume prune 명령을 하면 된다.

참고, -v 대신 --mount 옵션을 사용할 수 있다. 
  ```
  --mount type=volume,source=myvolume,target=/root
  --mount type=bind,source=/home/wordpress_db,target:/home/testdir
  ```

#### stateless vs stateful

	- stateless
		- 컨테이너가 아닌 외부에 데이터를 저장하고 컨테이너는 그 데이터로 동작하도록 설계하는 것
		- 컨테이너 자체는 상태가 없고 상태를 결정하는 데이터는 외부로부터 제공받는다.
		- 컨테이너가 삭제돼도 데이터는 보존되므로 스테이트리스한 컨테이너 설계는 도커를 사용할 때 매우 바람직한 설계 방법
	
	- stateful
		- 컨테이너가 데이터를 저장하고 있어 상태가 있는 경우
		- 스테이트풀한 컨테이너 설계는 컨테이너 자체에서 데이터를 보관하므로 지양하는 것이 좋다.
