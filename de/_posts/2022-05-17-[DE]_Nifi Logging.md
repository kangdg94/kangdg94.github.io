# [D/E] Nifi Logging

### Page Intrduce

- [ ]  Logging 이란? (What, Why , How )
- [ ]  Logging 설정
- [ ]  Dtonic NIFI logging 설정 확인
- [ ]  프로세서별 Logging하기

# Logging 이란?

> Logging이란 정보를 제공하는 일련의 기록인 Log를 생성하도록 시스템을 작성하는 활동을 말합니다. 시스템 오류 및 비정상 동작의 기록을 통해 감사 추적을 수행하여 문제를 해결할 수 있다.
> 

### LogLevel

- FATAL : 치명적인 에러(시스템적으로 심각한 문제가 발생해서 어플리케이션 작동이 불가능할 경우)
- ERROR : 에러(요청을 처리하는중 문제가 발생한 상태를 나타냄.)
- WARN : 경고(처리 가능한 문제이지만, 향후 시스템 에러의 원인이 될 수 있는 경고성 메시지를 나타냄.)
- INFO : 정보(로그인, 상태변경과 같은 정보성 메시지를 나타냄.)
- DEBUG : 상세 정보(개발시 디버그 용도로 사용한 메시지를 나타냄.)
- TRACE : 모든 정보(디버그 레벨이 너무 광범위한 것을 해결하기 위해서 좀더 상세한 상태를 나타냄.)

### Nifi Log File

- APP_FILE
- USER_FILE
- BOOTSTRAP_FILE

### Directory

**log파일**

> /home/nifi/nifi-1.13.2/logs
> 

**logback.xml 파일(log 설정 파일)**

> /home/nifi/nifi-1.13.2/conf/logback.xml
> 

---

# Logging 설정

### **nifi-app.log**

> 파일 로딩부터 런타임 오류 또는 NiFi 구성 요소에서 발생한 게시판에 이르기까지 Apache NiFi 애플리케이션의 모든 활동을 기록하는 nifi의 기본 로그 파일입니다.
> 

다음은 nifi-app.log 파일에 대한 logback.xml 파일 의 기본 Appender입니다 .

```xml
<appender name="APP_FILE"
class="ch.qos.logback.core.rolling.RollingFileAppender">
   <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app.log</file>
   <rollingPolicy
      class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <fileNamePattern>
         ${org.apache.nifi.bootstrap.config.log.dir}/
	      nifi-app_%d{yyyy-MM-dd_HH}.%i.log
      </fileNamePattern>
      <maxFileSize>100MB</maxFileSize>
      <maxHistory>30</maxHistory>
   </rollingPolicy>
   <immediateFlush>true</immediateFlush>
   <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
   </encoder>
</appender>
```

[nifi-app.log](/assets/img/nifi-app.log)

### RollOver 정책

### **RollingFileAppender**

RollingFileAppender는 파일의 크기 또는 파일 백업 인덱스 등의 지정을 통해서 특정 크기 이상으로 파일 크기가 커지게 되면, 기존파일(target)을 백업파일(history)로 바꾸고, 다시 처음부터 로깅을 시작한다.

RollingFileAppender와 함께 동작하는 두 가지 component가 존재합니다. 첫 번째는 RollingPolicy로 rollover에 필요한 action을 정의합니다. 두 번째는 TriggeringPolicy로 어느 시점에 rollover가 발생할지 정의합니다. 간단히 RollingPolicy는 what, TriggeringPolicy는 when을 담당하고 있다 보시면 될 것 같습니다.

-  fileNamePattern : history 파일명 패턴
-  maxFileSize : 설정한 크기에 도달하면 rollup 진행
-  maxHistory : 최대 백업 파일 유지 개수 ( 초과시 시간순으로 삭제 )
-  immediateFlush : 로그 메시지가 버퍼 되지 않음

*Appender,logger,Layout 설정

- [**Log환경설정 셋팅 방법**](https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte3:fdl:%EC%84%A4%EC%A0%95_%ED%8C%8C%EC%9D%BC%EC%9D%84_%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94_%EB%B0%A9%EB%B2%95)

## **nifi-user.log**

> 이 로그에는 웹 보안, 웹 API 구성, 사용자 권한 부여 등과 같은 사용자 이벤트가 포함되어 있습니다.
> 

아래는 logback.xml 파일의 nifi-user.log에 대한 Appender입니다

```xml
<appender name="USER_FILE"
   class="ch.qos.logback.core.rolling.RollingFileAppender">
   <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user.log</file>
   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>
         ${org.apache.nifi.bootstrap.config.log.dir}/
	      nifi-user_%d.log
      </fileNamePattern>
      <maxHistory>30</maxHistory>
   </rollingPolicy>
   <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
   </encoder>
</appender>
```

[nifi-user.log](/assets/img/nifi-user.log)

## **nifi-bootstrap.log**

> 이 로그에는 **bootstrap**로그, NiFi의 표준 출력(주로 디버깅을 위해 코드에 작성된 모든 출력) 및 표준 오류(코드에 작성된 모든 에러)가 포함됩니다.
> 

아래는 logback.log에 있는 nifi-bootstrap.log에 대한 기본 Appender입니다.

```xml
<appender name="BOOTSTRAP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
   <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap.log</file>
   <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>
         ${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap_%d.log
      </fileNamePattern>
      <maxHistory>5</maxHistory>
   </rollingPolicy>
   <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
      <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
   </encoder>
</appender>
```

[nifi-bootstrap.log](NIFI%20logging%20c1dd904e87b04d2db4aa37dadc02c541/nifi-bootstrap.log)

---

## Dtonic 프로젝트별 로그 설정 (아차키, EISS, SARWS)

[아차키logback.txt](NIFI%20logging%20c1dd904e87b04d2db4aa37dadc02c541/%EC%95%84%EC%B0%A8%ED%82%A4logback.txt)

[EISS_logback.txt](NIFI%20logging%20c1dd904e87b04d2db4aa37dadc02c541/EISS_logback.txt)

[sarws_logback.txt](NIFI%20logging%20c1dd904e87b04d2db4aa37dadc02c541/sarws_logback.txt)

### 아차키logback.xml

- APP_FILE

```xml
<appender name="APP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app_%d{yyyy-MM-dd_HH}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <immediateFlush>true</immediateFlush>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

- USER_FILE

```xml
<appender name="USER_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user_%d.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

- BOOTSTRAP_FILE

```xml
<appender name="BOOTSTRAP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap_%d.log</fileNamePatter
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

### SARWS logback.xml

- APP_FILE

```xml
<appender name="APP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app_%d{yyyy-MM-dd_HH}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <immediateFlush>true</immediateFlush>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

- USER_FILE

```xml
<appender name="USER_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user_%d.log</fileNamePatte
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

- BOOTSTRAP_FILE

```xml
<appender name="BOOTSTRAP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap_%d.log</fileNamePattern>
            <maxHistory>5<v/maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

### EISS logback.xml

- APP_FILE

```xml
<appender name="APP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-app_%d{yyyy-MM-dd_HH}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <immediateFlush>true</immediateFlush>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>

```

- USER_FILE

```xml
<appender name="USER_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-user_%d.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

- BOOTSTRAP_FILE

```xml
<appender name="BOOTSTRAP_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-bootstrap_%d.log</fileNamePattern>
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
        </encoder>
    </appender>
```

---

## 프로세서별 Logging 하기

### Log 파일을 생성하여 관리하기

> logback.xml 파일을 통하여 <appender>와 <logger>를 적절하게 설정함으로서 각 프로세스의 log를 원하는 로그 레벨로 logging가능하다.
> 

```xml
<appender name="STATUS_LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>${org.apache.nifi.bootstrap.config.log.dir}/nifi-status.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${org.apache.nifi.bootstrap.config.log.dir}/nifi-status_%d.log</fileNamePattern>
        <maxHistory>30</maxHistory>
    </rollingPolicy>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%date %level [%thread] %logger{40} %msg%n</pattern>
    </encoder>
</appender>

<logger name="org.apache.nifi.processors.standard.LogAttribute" level="TRACE" additivity="false">
    <appender-ref ref="STATUS_LOG_FILE" />
</logger>

<logger name="org.apache.nifi.processors.standard.ConvertJSONToSQL" level="DEBUG" additivity="false">
    <appender-ref ref="STATUS_LOG_FILE" />
</logger>
```

**Appender 설정**

nifi-status.log 로그 파일로 maxHistory 30으로 STATUS_LOG_FILE Appender를 설정 해주었다.

**Logger 설정**

기본적으로 NIFI 프로세서는 org.apache.nifi.processors.standard 클래스를 상속 받으며 로깅하고싶은 프로세서를 설정하면 된다. 위 코드는 ConvertJSONToSQL 과 LogAttribute를  nifi-status.log에 로깅 하도록 설정 하였다.

![logAttr.png](/assets/img/logAttr.png)

1. logAttribute 프로세서를 실행시켜 nifi-status.log 파일에 해당 프로세서에 대한 로그가 남았는지 확인해 보자.

![cat_LogAttr.png](/assets/img/cat_LogAttr.png)

1. nifi-status.log 파일에 해당 로그가 남은걸 확인 할 수 있다.

![Convert.png](/assets/img/Convert.png)

1. ConvertJSONToSQL 실행 시 오류가 발생하였다. 해당 오류가 nifi-status.log 파일에 남아 있는지 확인해보자.

![cat_Convert.png](/assets/img/cat_Convert.png)

1. nifi-status.log 파일에 해당 오류가 남은 걸 확인 할 수 있다.

### Log prefix 를 사용하여 임시적으로 로깅하기

> LogAttribute의 Log prefix 눈에 보기 쉽게 표시를 설정하여 nifi-app.log 볼 시 해당 프로세서에 대한 로그를 쉽게 찾을 수 있도록 한다.
> 

![Log_Properties.png](/assets/img/Log_Properties.png)

1. LogAttribute 프로세서의 Log prefix를 설정해준다.

![nifi-app.log2.png](/assets/img/nifi-app.log2.png)

1. log-app.log 를 통하여 로그를 볼때 미리 입력해둔 표시 때문에 쉽게 찾을 수 있다.

---

## 결론

자주 로그를 봐야 할 프로세서에 대해서 첫번째 설명과 같이 Appender 와 logger를 적절하게 설정하여 log파일을 만들어 따로 관리 하는 것이 편할 것이고, 

자주 사용 하지 않을 프로세서에 대해 로그를 봐야 할 경우 위의 방법이 번거러울 수 있으므로 두번째 방법으로 임시방편이 있겠다. 상황을 고려 하여 취사선택 하면 되겠다.
