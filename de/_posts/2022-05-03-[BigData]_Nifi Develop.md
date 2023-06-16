# [BigData] Nifi Develop

---

- 목표

Docker를 통해 NIfi, DB를 띄운 후 Nifi를 사용하여 특정 시간에 SSH를 통해 다른 서버에 접근하여 쉘 스크립트를 실행시킨 후 결과물(데이터)를 유효성을 판단 후 Valid Data, Invalid Data 그리고 log를 각각 DB에 적재한다.

- 개발환경
- DB: image>Postgresql:11.15
- Nifi: image> apache/nifi
- pgAdmin: image> dpage/pgadmin4
- java: jdk 17.0.2

---

- Docker Compose를 작성하여 Nifi와 Postgresql를 도커에 띄워보자.

**docker-compose.yml**

```docker
version: "3"
services:
  PostgreSQL:
    container_name: PostgreSQL1
    image: postgres:11.15
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: "nifi"
      POSTGRES_PASSWORD: "Password"
      TZ : 'Asia/Seoul'
    restart: always
    networks:
     - nifi-workflow-bridge
    volumes:
      - Nifi-Workflow-Volume:/var/lib/postgresql/data
    
  nifi:
    container_name: nifi
    image: apache/nifi
    environment:
      NIFI_WEB_HTTP_PORT: 9095
      NIFI_LOG_DIR: /opt/nifi/logs
      TZ: Asia/Seoul
    ports:
      - 9095:9095
    restart: always
    networks:
      - nifi-workflow-bridge
    volumes:
      - C:/nifi-logs:/opt/nifi/nifi-current/logs
      - C:/nifi-logs/dockslib:/var/lib
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
      PGADMIN_DEFAULT_EMAIL: "user@email.com" 
      PGADMIN_DEFAULT_PASSWORD : "password"
    restart: always
    networks:
     - nifi-workflow-bridge
     
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

[**Dockerhub**](https://hub.docker.com/)에 도커 이미지에 대한 environment의 필수값required과 선택값이 명시되어 있으니 참고하여 작성하면 된다. 필수값을 넣지 않으면 실행되지 않는다. 
필요한 데이터에 대한 Volumes값만 잡아주었다. 이는 도커 재실행시 작업이 날라가는 것을 방지하기 위함이다.~~(본인은 3번 날렸다)~~

Compose경로에서 docker-compose up를 통하여 빌드 후 docker ps를 통하여 컨테이너가 잘 올라왔는지 확인해본다.

![12](/assets/img/12.png)

---

- 포트포워딩을 해준 포트를 통해 Nifi를 접속해보자
1. 좌측 상단의 Process Group를 눌러 프로세스 그룹을 생성한다.

![13](/assets/img/13.png)

1. Porcess Group 생성 후 프로세스를 작성한다.

GetSFTP - RouteOnattribute- ValidateCSV - ConvertRecord - ConverJsontoSQL - ReplaceText - UpdateAttribute - ExcuteSQL 를 사용하여 완성하였다.

- Process 설명

- GetSFTP : SSH를 통해 접근하여 지정한 path에서 디렉토리 전체를 가져온다. 단일 파일을 가져오려면 FetchSFTP를 상용해야하나 Upstream이 필수이므로 적합하지 않다 판단하여 GetSFTP를 사용하여 RouteOnattribute를 통해 필터링 한다.

![14](/assets/img/14.png)

- RouteOnattribute : nifi 정규표현식${filename:contains(${now():toDate():format('yyyyMMdd')})}을 사용하여 현재날짜 파일명을 가진 파일을 가져온다.

![15](/assets/img/15.png)

- ValidateCSV : 가져온CSV파일의 유효성을 검사한다. 각 컬럼의 타입값을 비교하여 유효한 데이터인지 판별 가능하다.

![16](/assets/img/16.png)

- ConvertRecord : 데이터 포맷 형식을 변환해주는 프로세서이다. (CSV → Json로 변환 해주었다.)첫줄을 컬럼명으로 하여 각 행 또는 전체 데이터의 포맷 형식 변환이 가능하다.

![1](/assets/img/1.png)

- ConverJsontoSQL : Json을 SQL 쿼리문으로 변환해주는 프로세서이다. Insert, Update 등 지정해주어 쿼리문 변환이 가능하다.

![2](/assets/img/2.png)

- ReplaceText : content를 변환해주는 프로세서이다.(위 SQL문을 수정하기 위함이다.)

![3](/assets/img/3.png)

- UpdateAttribute : Nifi는 아키텍쳐로 인해 속성(attribute)값을 먼저 조회 후 데이터(content)를 참조하여 실행된다. ConvertRecord 통해 Json형식으로 변환 하면서 속성 값이 추가되었다 이 때문에 Nifi는 ReplaceText에서 수정된 content를 읽어오지 않으므로 UpdateAttribute 프로세서를 통해 추가된 속성 값을 삭제한다.

![4](/assets/img/4.png)

- ExcuteSQL : 데이터(Content)값을 그대로 실행한다.

![5](/assets/img/5.png)

**전체Flow**

![23](/assets/img/23.png)

![24](/assets/img/24.png)

pgAdmin으로 DB에 데이터가 적재가 되었는지 확인해본다.

![25](/assets/img/25.png)

원하는 데이터가 잘 적재 되었는걸 확인해 볼 수 있다.

---

간단한 예제를 통해 직접 Nifi를 사용하여 데이터 모델링을 구현 해보았다. Nifi는 직관적인UI와 프로세서를 가지고 있어 개발하기 편한 소프트웨어임은 분명하다. 

300개가 넘는 프로세서를 모두 외우고 사용하기 힘들겠지만, 가능한 많은 프로세서를 알고 적절하게 사용함으로써 플로우 단계를 줄이는 것 또한 중요할 것이다.
