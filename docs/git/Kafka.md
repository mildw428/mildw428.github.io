---
layout: default
title: Kafka
parent: git
---

설정 코드 : https://github.com/mildw/doraemon/blob/main/kafka/src/main/java/com/example/kafka/config/kafka/KafkaConfig.java


**Apache Kafka**
![[producer_consumer.png]]
카프카는 topic을 통해 메시지 피드를 유지 관리한다.
topic producer를 통해 kafka cluster에 메시지를 적재한다.
consumer를 통해 kafka cluster에서 topic을 감지하고 메시지를 가져온다.
그림의 kafka cluster는 1개지만 그 이상의 서버로 구성될 수 있다.

![[log_anatomy.png]]

cluster에 1개의 토픽에 관한 파티션을 살펴보면 위의 그림과 같다.
각 파티션의 메시지는 지속적으로 추가되고, 정렬 또는 변경이 불가능하다.
파티션 별로 메시지에 오프셋이라는 순차 ID가 부여된다.

**메시지 유지 기간**
- cluster는 구성 가능 기간 동안 소비 여부에 관계없이 모든 메시지를 유지한다.
- (보존 기간이 2일로 설정된 경우 메시지가 게시된 후 2일 동안 사용할 수 있고, 그 후에는 공간 확보를 위해 삭제한다.)

**offset**
- 각각의 consumer는 오프셋 번호를 가지고 있다.
- 오프셋은 소비자 마음대로 왔다 갔다 할 수 있으며, 그로 인해 처리했던 메시지를 다시 처리할 수 있다.
- consumer의 오프셋 변화는 cluster 또는 producer에 영향을 미치지 않는다.

**파티션**
- 파티션은 1개의 topic에 대해 많은 양의 파티션으로 구성할 수 있고, 이것은 병렬 처리가 가능하다.
- 주의할 점은 kafka를 사용하면서 **파티션을 늘릴 수는 있지만 줄일 수는 없다.**
- 파티션을 줄이고 싶다면 토픽을 삭제하고 다시 만들어야 한다.

**복사 전략**
- replication-factor 옵션에 따라 여러 cluster 서버에 데이터를 복제하여 안전하게 관리할 수 있다.
- 2로 설정한다면 원본과 복사본의 갯수를 합쳐서 2개의 데이터가 존재한다.
- 전략은 leader - follwers 방식으로 아래 사진을 참고할 수 있다.

![[Adv_KT_Choosing_Rep_Factor_1.jpg]]

leader-follwers 방식
- 클러스터는 브로커라고도 부를 수 있다.
- 각 파티션에서 리더를 선출하여 팔로워들은 리더의 데이터를 복사한다.
- 리더가 죽게 되면 팔로워 중 하나를 리더로 선출한다.

메시지 적제
- producer는 topic 내의 어떤 파티션에 메시지를 할당할지 선택해야 한다.
- 방식은 메시지의 key 기반으로 분할되거나, key 값이 없다면 기본적으로 라운드 로빈 방식으로 분할된다.

메시지 소비
- kafka는 소비자 그룹을 제공한다.
﻿
![[consumer-groups.png]]

consumer에 group-id를 지정한다.
(C1과 C2는 A라는 group-id를 지정하여 하나의 그룹으로 본다.)
동일한 group-id를 가진 consumer 중 하나라도 메시지를 소비하면,
같은 그룹 내의 consumer들은 메시지를 소비하지 않고 넘어간다.
(C1과 C2는 1개의 topic에 대한 파티션처럼 동작한다)

## kafka install
카프카 설치 방법과, cmd에서 토픽 발행, 메시지 프로듀스, 컨슘 명령어를 정리해두었다.

https://kafka.apache.org/quickstart

```shell
curl "https://downloads.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz" -o ~/kafka/kafka_2.13-3.3.1.tgz

tar -xzf kafka_2.13-3.3.1.tgz
cd kafka_2.13-3.3.1

```

### create topic 

```bash
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
```

### check topic
```bash
bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
```

### produce

```shell
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
```

### consum

```bash
bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
```

## ﻿﻿Kafdrop : Kafka Web UI 사용해 보기


[GitHub - obsidiandynamics/kafdrop: Kafka Web UI](https://dthumb-phinf.pstatic.net/?src=%22https://opengraph.githubassets.com/b68199e7ebf0e764e0e2d565a3983212048bbc4450e721c6230daf6ea05203a3/obsidiandynamics/kafdrop%22&type=ff500_300)

[GitHub - provectus/kafka-ui: Open-Source Web UI for Apache Kafka Management](https://dthumb-phinf.pstatic.net/?src=%22https://repository-images.githubusercontent.com/224230574/5657b64d-b026-4e9e-ae34-e8082e3db767%22&type=ff500_300)

위의 두개가 실행방법에는 큰 차이가 없다.

선호에 따라 골라 사용하자.

**Apache Kafka**

Apache Kafka: A Distributed Streaming Platform.

kafka.apache.org

카프카 메시지 조회가 너무 불편하다.

web ui 로 편리하게 토픽을 조회할 수 있는 kafdrop 이란 걸 발견했다.

만약 인텔리제이를 사용한다면,

**Kafka monitoring | IntelliJ IDEA**

www.jetbrains.com

플러그인을 고려해 보자.

(너무 늦게알았다)


```javascript
// 아래 명령어에서 <host:port,host:port> 부분만 수정해서 실행하면 된다.
docker run -d --rm -p 9000:9000 \
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
Digest: sha256:5337c9e0e2dee204bdde53e90cf97001f44fb9e8c3380340436efa844901a3f4
Status: Downloaded newer image for obsidiandynamics/kafdrop:latest
a8d978f7e0d0a715dfef2e0f0a6c69fd6faf4023d5d16611b8bd7a5dcba6103d
kms0428@kms0428-laptop:~$
```

아래는 kafka-ui 의 명령어로 실행하는데 큰 차이가 없다.

```javascript
docker run -p 8080:8080 \
	-e KAFKA_CLUSTERS_0_NAME=local \
	-e KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=kafka:9092 \
	-d provectuslabs/kafka-ui:latest
```

﻿
![[kafdropimage.png]]


localhost:9000으로 접근하게 되면 위와 같은 화면을 볼 수 있다.
﻿
끄고 싶다면 컨테이너를 종료시켜주자.
