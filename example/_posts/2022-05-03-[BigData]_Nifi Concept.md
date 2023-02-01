# [BigData] Nifi Concept

---

- **Nifi 란?**

![nifi](/assets/img/nifi.png)

> 아파치 나이파이(Apache NiFi)는 소프트웨어 시스템 간 데이터 흐름을 자동화하도록 설계된 아파치 소프트웨어  이며, 데이터를 가져오고 가공 후 적재를 하며 분산환경에 대량의 데이터를 가져오고 처리하는 FBP개념을 구현한 오픈소스 소프트웨어이다.
> 

- **Nifi 주요 용어 정리 및 아키텍쳐**

1. 주요 용어정리

> Nifi에 대해 구글링을 하다보면 아래와 같은 용어들을 많이 접하게 될것이다. 그래서 정리를 기본 주요 용어에 대하여 간단하게 정리를 해보았다.
> 

****Flow Based Programming (fbp)**** 

| NIFI 용어  | 설명 |
| --- | --- |
| FlowFile | Nifi에서 데이터를 표현하는 객체로입니다.
데이터의 구조는 Key/Value 형태로, 속성(Attribute)과 데이터(Content)로 이루어져 있다. 말 그대로 flowfile의 개념이다.  |
| FlowFile Processor | FlowFile은 여러 단계를 걸쳐 속성이 추가되거나 내용이 변경될 수 있는데, 이때 사용되는 것이 FlowFile Processor이다. 즉, 프로세서들을 걸쳐 FlowFile(데이터)들이 가공 되는 것이다. |
| connection | Processor 간의 연결 역할(직관적으로 프로세서 사이를 이어주는 선이다).
Connection은 대기열 역할을하며 다양한 프로세스가 서로 다른 속도로 상호 작용할 수 있도록 한다.
대기열(queueing)뿐만 아니라 라우팅, 처리량 제한, 우선순위 제어, 모니터링 등의 강력한 기능을 제공한다. |
| Flow controller | Processor가 어느 간격 또는 시점에 실행하는지 스케줄링한다. |
| Process Group | 프로세서들을 뭉쳐놓은 일종의 꾸러미 개념이다.
특정 업무, 기능 단위로 여러 Processor를 묶을 수 있으며, Input과 Output 포트를 제공해 Process Group 간의 데이터 이동이 가능하다. |

위 용어를 모두 이해하였다면 구글링을 하거나 간단한 실습을 한다면 크게 문제는 없을것이다.

1. **Nifi 아키텍쳐**

> Nifi를 간단하게 사용하려면 아키텍쳐까지 이해할 필요는 없다. 하지만 더 깊게 이해하고 싶다거나 대량의 데이터를 처리하는 Nifi를 구현 하려면 필히 알아야되는 개념이다.
> 

![a](/assets/img/a.png)

**Web Server**

- 말 그대로 웹서버 이며 웹서비스에 대해 이해도가 있으면 같은 개념이므로 따로 이해 하지 않아도 된다.
- NiFi의 HTTP 기반 명령 및 제어 API를 호스팅 한다.

**Flow Controller**

- Nifi의 cpu역할을 하며 스레드제공, 리소스 수신 등 프로세스의 관리를 한다.

**Extensions**

- JVM내에서 실행 되는 NiFi가 제공하는 기본 프로세스들 외에 개발자가 프로세스를 개발해 확장할 수 있다.

**FlowFile Repostory**

- 간단하게 데이터 FlowFile을 저장 하는 레퍼지토리이다.
- 일반적으로 Raid 10으로 디스크를 구성하여 저장해, 시스템 장애 때 유실되지 않게 한다.

**Content Repository**

- FlowFile의 데이터(Content)가 저장되며, 일반적으로 Raid 10으로 구성해 저장하며, 여러 디렉토리에 분석 저장이 가능하다.

---

Nifi를 사용 해본 결과 매우 직관적인 UI를 가지고 있어 사용하기 정말 편하다. HTTPS 표준 규격을 사용하여 보안에 강하며 FTP, SSH, DB, 하둡 과 같이 작업하기 매우 용이하다.
백견이 불여일타(百見不如一打) 직접 개발해보며 느끼길 바란다.
