---
layout: default
title: Docker
parent: git
---

install for mac
[colima](https://github.com/abiosoft/colima)

https://velog.io/@sblee/install-colima
```shell
brew install colima docker docker-compose

mkdir -p ~/.docker/cli-plugins

ln -sfn $(brew --prefix)/opt/docker-compose/bin/docker-compose ~/.docker/cli-plugins/docker-compose

colima start
```

colima start를 시작 시켜야 도커가 동작한다.

명령어를 통해 컨테이너에 접근할 수 있다.

```shell
docker exec -it {container_id 또는 컨테이너 이름} bash

# ctrl + p + q 를 통해 나올 수 있다.
```

---
## network

```shell
docker network ls
```

---
## remege
```shell
docker rmi {IMAGE_ID}
```

---
## mariadb
https://hub.docker.com/_/mariadb

```shell
docker pull mariadb
```

```shell
docker images
```

```shell
docker run -v ~/data/mariadb:/data/db --name testDB -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=1111 mariadb
```

---
## mongodb

```shell
docker pull mongo
```

```shell
docker run -v ~/data/mongodb:/data/db --name mongodb -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=1111 mongo
```

`-v ~/data:/data/db`는 호스트(컨테이너를 구동하는 로컬 컴퓨터)의 ~/data 디렉터리와 컨테이너의 /data/db 디렉터리를 마운트시킨다. 이처럼 볼륨을 설정하지 않으면 컨테이너를 삭제할 때 컨테이너에 저장되어있는 데이터도 삭제되기 때문에 복구할 수 없다.

---
## redis
https://hub.docker.com/_/redis

```shell
docker pull redis
```

```shell
docker run -d --name testRedis -p 6379:6379 redis --requirepass "1111"
```

---
## kafka
https://hub.docker.com/r/bitnami/kafka

#### create docker-compose.yml

```yaml
version: "3"
services:
  zookeeper:
    image: 'bitnami/zookeeper:latest'
    ports:
      - '2181:2181'
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
  kafka:
    image: 'bitnami/kafka:latest'
    ports:
      - '9092:9092'
    environment:
      - KAFKA_BROKER_ID=1
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://127.0.0.1:9092
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - ALLOW_PLAINTEXT_LISTENER=yes
    depends_on:
      - zookeeper
```

```shell
docker-compose up -d
```

---
## kafka Web UI

https://github.com/obsidiandynamics/kafdrop
https://github.com/provectus/kafka-ui

```bash
// kafkadrop
// 아래 명령어에서 <host:port,host:port> 부분만 수정해서 실행하면 된다. docker run -d --rm -p 9000:9000 \
	-e KAFKA_BROKERCONNECT=<host:port,host:port> \
	-e JVM_OPTS="-Xms32M -Xmx64M" \
	-e SERVER_SERVLET_CONTEXTPATH="/" \
	obsidiandynamics/kafdrop

// 명령어 실행 후 문구
Unable to find image 'obsidiandynamics/kafdrop:latest' locally
latest: Pulling from obsidiandynamics/kafdrop
e0b25ef51634: Pull complete
d1bd2bc15eb1: Pull complete
77d0de1fd7e0: Pull complete
b638505435dc: Pull complete
8bbf85e38402: Pull complete
1ceef1d1d525: Pull complete
643cf0d075ea: Pull complete
Digest: sha256:5337c9e0e2dee204bdde53e90cf97001f44fb9e8c3380340436efa844901a3f4 Status: Downloaded newer image for obsidiandynamics/kafdrop:latest a8d978f7e0d0a715dfef2e0f0a6c69fd6faf4023d5d16611b8bd7a5dcba6103d kms0428@kms0428-laptop:~$

```

```bash
//kafka ui
docker run -p 8080:8080 \
	-e KAFKA_CLUSTERS_0_NAME=local \
	-e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:9092 \
	-d provectuslabs/kafka-ui:latest
```


---

keycloak

```shell
docker run quay.io/keycloak/keycloak start-dev
```

