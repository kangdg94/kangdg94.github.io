# [CI/CD] Docker

## Docker 란 ?

> **독립된 환경을 만들어서 하드웨어를 효율적으로 활용하는 기술**
> 

**자세히** : 도커는 컨테이너 기술을 기반으로 한 일종의 가상화 플랫폼.

가상화란 물리적 자원인 하드웨어를 효율적으로 활용하기 위해서 하드웨어 공간 위에 가상의 머신을 만드는 기술이고, 컨테이너란 컨테이너가 실행되고 있는 호스트 os의 기능을 그대로 사용하면서 프로세스를 격리해 독립된 환경을 만드는 기술을 뜻합니다. 

**VM vs 도커 차이점 :**

![docker](/assets/img/docker.png)

> 하이퍼바이저: 단일 하드웨어에서 여러 다른 가상머신을 호스팅 할 수 있게 하는 프로그램
> 

위 그림만 보면 이해가 되듯이, vm은 하이퍼바이저를 통해 os 를 띄우는 개념이라 무겁고 실행시 시간이 오래걸리지만 도커는 host os위에 올리기 때문에 가볍다

## **Docker command (Dockerfile 에 사용되는 명령어만 우선 정리해본다.)**

### **RUN**

RUN 명령문는 쉘(shell)에서 커맨드를 실행하는 것처럼 이미지 빌드 과정에서 필요한 커맨드를 실행하기 위해 사용된다.

보통 이미지 안에 특정 소프트웨어를 설치하기 위해서 많이 사용된다.

### **ENTRYPOINT**

ENTRYPOINT 명령문는 CMD 명령문와 비슷하지만, 컨테이너를 띄울 때 항상 실행되야 하는 커맨드를 지정할 때 사용한다. Docker 이미지를 하나의 실행 파일처럼 사용할 때 매우 유용하다.

이유는 컨테이너가 실행될때 ENTRYPOINT 명령문으로 지정된 커맨드가 실행이 되고, 이 커맨드로 실행 된 프로세스가 죽으면, 컨테이너도 함께 종료 되기 때문이다.

### **CMD**

CMD 명령어는 해당 이미지를 컨테이너로 띄울 때 디폴트로 실행할 커맨드나, ENTRYPOINT 명령문으로 지정된 커맨드에 디폴트로 넘길 파라미터를 지정할 때 사용한다.

예를 들어 아래와 같이 Dockerfile을 작성한다.

## Docker Compose

> 여러 개의 컨테이너를 실행시키는 도커 애플리케이션이 정의를 하기 위한 **Tool** 이다.
> 

Compose를 사용하면 YAML 파일을 사용하여 애플리케이션의 서비스를 구성할 수 있다.

그런 다음 single command를 사용하여 구성에서 모든 서비스를 만들고 시작한다.

Compose는 모든 환경(workflow에서 production, staging, development, testing 및 CI 워크플로우)을 포함한다.

아래와 같이 yaml 파일을 설정 할 수 있다. (실제 사용했던 yaml파일)

```yaml
version: "3"
services:
  nifi:
    container_name: nifi
    image: apache/nifi
    environment:
      NIFI_WEB_HTTPS_HOST: '0.0.0.0'
      NIFI_WEB_HTTPS_PORT: 8443
      TZ: Asia/Seoul
      SINGLE_USER_CREDENTIALS_USERNAME: admin
      SINGLE_USER_CREDENTIALS_PASSWORD: ctsBtRBKHRAx69EqUghvvgEvjnaLjFEB

    ports:
      - 8443:8443
      - 21177:21177
    #restart: always
    networks:
      - nifi-workflow-bridge
    volumes:
      - nifi-conf:/opt/nifi/nifi-current/conf
      - nifi-state:/opt/nifi/nifi-current/state
      - nifi-content:/opt/nifi/nifi-current/content_repository
      - nifi-database:/opt/nifi/nifi-current/database_repository
      - nifi-flowfile:/opt/nifi/nifi-current/flowfile_repository
      - nifi-provenance:/opt/nifi/nifi-current/provenance_repository

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    ports:
      - "5050:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: "donggeun.kang@dtonic.io"
      PGADMIN_DEFAULT_PASSWORD: "123456"
    #restart: always
    networks:
      - nifi-workflow-bridge

  postgis:
    image: kartoza/postgis:11.0-2.5
    container_name: postgis4
    environment:
      POSTGRES_PASS: postgres
      POSTGRES_USER: postgres
      POSTGRES_DBNAME: smart_city
    restart: always
    volumes:
      - ./setup-db.sql:/docker-entrypoint-initdb.d/setup-db.sql
      - ./postgres-data:/var/lib/postgresql/11
    ports:
      - "15432:5432"

networks:
  nifi-workflow-bridge:
    driver: bridge
    name: nifi-workflow-bridge

volumes:
  Nifi-Workflow-Volume:
    name: Nifi-Workflow-Volume
  nifi-conf:
  nifi-state:
  nifi-content:
  nifi-database:
  nifi-flowfile:
  nifi-provenance:
```

docker volume, log 관리, 데이터 관리를 좀 더 정리 해봐야겠다
