# [Monitoring] ELK

## 1. ELK 란?
![elk_6](/assets/img/elk_6.png)

**ELK**는 위 그림과 같이, 분석 및 저장 기능을 담당하는 **ElasticSearch**, 수집 기능을 하는 **Logstash**, 이를 시각화 하는 도구인 **Kibana**의 앞 글자만 딴 단어이다. 먼저 Prometheus를 사용 하였으나, prometheus는 서비스 로그들을 수집 하기 어려우며 push방식이라 각 서비스들이 prometheus에 데이터를 전송 해주는 형태로, 서비스들 코드에 추가되어야 한다. 이는 서비스 운영에 대한 리스크가 존재하며, 포트 충돌도 우려가 되어 polling방식인 ELK 도입을 하게 되었다.

**1) ElasticSearch**

- 크게 Aggregation 과 Query 두 기능으로 나뉘며, Query는 형 분석 Aggregation은 집계, 즉 모니터링 쪽에서 많이 쓰인다. 초기에는 Query로 등장하였으나 최근에는 Aggregation 기능으로 많이 사용되고 있는 것으로 보임
- 로그, 정형, 비정형, 위치정보, 메트릭 등 원하는 방법으로 다양한 유형의 검색을 수행하고 결합할 수 있다.

**2) Logstash**

- 데이터 처리 파이프 라인으로, 다양한 소스에서 동시에 데이터를 수집하고 변환하여 stash 보관소로 보낸다.
- 수집할 로그를 선정해서, 지정된 대상 서버(ElasticSearch)에 인덱싱하여 전송하는 역할을 담당하는 서비스다.

**3) Kibana**

- 데이터를 시각적으로 탐색하고 실시간으로 분석 할 수 있다.
- 시각화를 담당하는 HTML + Javascript 엔진이라고 보면 된다.

## 2. Beats 란?

![elk_1](/assets/img/elk_1.png)

**Beats :** 서버에 에이전트로 설치하여 다양한 유형의 데이터를 ElasticSearch 또는 Logstash에 전송하는 오픈 소스 데이터 발송자다.

- ELK 솔루션에서 Beats 추가되면서 ELK Stack이라고 불린다.
- 크게 서비스 로그를 수집할 filebeat와 서버 메트릭 정보를 수집할 metricbeat를 사용 하였다.

## 3. Pipeline flow

이번 ELK 구축하면서 설계한 데이터 파이프라인 아키텍쳐 이다.
![elk_2](/assets/img/elk_2.png)


- FileBeat만 Logstash로 보낸 이유는 로그 들을 Indexing하기 위함이다. Indexing 되어야 elasticsearch에서 분석을 할 때 검색이나 필터링이 용이하다. Metric정보들은 Indexing할 필요가 없으니 바로 elasticsearch로 전송.
- Kibana는 단지 시각화 용도 이며(브라우저에 kibana포트로 접근) 분석이나 통계 검색 등 모두 뒷단인 elasticsearch에서 실행된다

## 4. File structure

- **File tree structure**

```
📦docker-compose.yml
📦elasticsearch
 ┣ 📂config
 ┃ ┗ 📜elasticsearch.yml
 ┣ 📜.dockerignore
 ┗ 📜Dockerfile
📦logstash
 ┣ 📂config
 ┃ ┗ 📜logstash.yml
 ┣ 📂pipeline
 ┃ ┗ 📜logstash.conf
 ┣ 📜.dockerignore
 ┗ 📜Dockerfile
📦kibana
 ┣ 📂config
 ┃ ┗ 📜kibana.yml
 ┣ 📜.dockerignore
 ┗ 📜Dockerfi
📦metricbeat
 ┗ 📜metricbeat.yml
📦filebeat
 ┣ 📂config
 ┃ ┗ 📜filebeat.yml
 ┣ 📂data
 ┃ ┗ 📜meta.json
 ┗ 📜Dockerfile
```

- **docker-compose.yml**

```yaml
version: '3.7'

services:
  setup:
    profiles:
      - setup
    build:
      context: setup/
      args:
        ELASTIC_VERSION: ${ELASTIC_VERSION}
    init: true
    volumes:
      - ./setup/entrypoint.sh:/entrypoint.sh:ro,Z
      - ./setup/lib.sh:/lib.sh:ro,Z
      - ./setup/roles:/roles:ro,Z
    environment:
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD:-}
      LOGSTASH_INTERNAL_PASSWORD: ${LOGSTASH_INTERNAL_PASSWORD:-}
      KIBANA_SYSTEM_PASSWORD: ${KIBANA_SYSTEM_PASSWORD:-}
      METRICBEAT_INTERNAL_PASSWORD: ${METRICBEAT_INTERNAL_PASSWORD:-}
      FILEBEAT_INTERNAL_PASSWORD: ${FILEBEAT_INTERNAL_PASSWORD:-}
      HEARTBEAT_INTERNAL_PASSWORD: ${HEARTBEAT_INTERNAL_PASSWORD:-}
      MONITORING_INTERNAL_PASSWORD: ${MONITORING_INTERNAL_PASSWORD:-}
      BEATS_SYSTEM_PASSWORD: ${BEATS_SYSTEM_PASSWORD:-}
    networks:
      - elk
    depends_on:
      - elasticsearch

  elasticsearch:
    build:
      context: elasticsearch/
      args:
        ELASTIC_VERSION: ${ELASTIC_VERSION}
    volumes:
      - /home/dongjin/Projects/elk/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml:ro,Z
      - elasticsearch-vol:/usr/share/elasticsearch/data:Z
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      node.name: elasticsearch
      ES_JAVA_OPTS: -Xms512m -Xmx512m
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD:-}
      discovery.type: single-node
    networks:
      - elk

  logstash:
    build:
      context: logstash/
      args:
        ELASTIC_VERSION: ${ELASTIC_VERSION}
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro,Z
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro,Z
    ports:
      - 5044:5044
      - 50000:50000/tcp
      - 50000:50000/udp
      - 9600:9600
    environment:
      LS_JAVA_OPTS: -Xms256m -Xmx256m
      LOGSTASH_INTERNAL_PASSWORD: ${ELASTIC_PASSWORD:-}
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    build:
      context: kibana/
      args:
        ELASTIC_VERSION: ${ELASTIC_VERSION}
    volumes:
      - ./kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml:ro,Z
    ports:
      - 5601:5601
    environment:
      KIBANA_SYSTEM_PASSWORD: ${KIBANA_SYSTEM_PASSWORD:-}
    networks:
      - elk
    depends_on:
      - elasticsearch

  metricbeat:
    image: docker.elastic.co/beats/metricbeat:7.6.2
    #user: root:root
    volumes:
            #- ./metricbeat/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml
      - /var/run/docker.sock:/var/run/docker.sock:ro
    environment:
      - ELASTICSEARCH_HOSTS=elasticsearch:9200
      - ELASTICSEARCH_USERNAME=elastic
      - ELASTICSEARCH_PASSWORD=${ELASTIC_PASSWORD}
      - KIBANA_HOST=kibana:5601
    networks:
      - elk

  filebeat:
    build:
      context: ./filebeat
    user: root:root
    container_name: filebeat
    environment:
      ELASTIC_PASSWORD: ${ELASTIC_PASSWORD:-}
    volumes:
      - ./filebeat/data:/usr/share/filebeat/data
      - ./filebeat/config/filebeat.yml:/usr/share/filebeat/filebeat.yml
      - /var/log:/logs/host
      - /var/log:/usr/share/filebeat/logs
    networks:
      - elk

networks:
  elk:
    driver: bridge

volumes:
  elasticsearch-vol:
```

## 5. 주로 사용하는 기능

- Service Log Streaming
![elk_3](/assets/img/elk_3.png)


Observability탭에서 Stream메뉴를 이용하면 Service Log들을 실시간으로 스트리밍 받아 볼 수 있다. 

- Service Log Filter
![elk_4](/assets/img/elk_4.png)


Analytics탭에서 discover메뉴를 이용하면 index를 골라 필터링을 할 수 있다. 사진은 해당 인덱싱의 로그에서 context내용 중에 mcmot_01에서 error를 검색한 결과이다. context 말고 keyword, timestamp등 많은 내용들로 필터링 할 수 있으며 각 필터들을 or 혹은 and로 덧붙여 사용할 수 있다.

- Metric 정보
![elk_5](/assets/img/elk_5.png)


Observability탭에서 Host메뉴를 이용하면 호스트의 Metric정보들을 볼 수 있다. 아래 Dashboard에 시계열 그래프로도 표현이 되어 한 눈에 파악하기 쉽다.

## TODO

- 현재 Host의 Metric정보만 받아오고 있지만 향 후에 Container별 Metric정보를 수집하여 어떤 서비스에서 많은 리소스를 점유하고 있는지 확인 할 수 있도록 해야 한다.
- 현재 여러 서비스 로그들을 한 Index에 넣어서 수집하고 있지만 향 후에 서비스별 인덱싱을 하여 좀 더 효과적인 필터링 및 분석을 할 수 있도록 해야 한다.
- Error및 커스터마이징한 감지 룰을 통해 alert기능 및 에러 대응 기능 까지 포함 해야 비로써 자동화 모니터링이라고 할 수 있을 것이다 (해당 기능들은 유료 기능임)
