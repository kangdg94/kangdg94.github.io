# [JAVA] Multi-Thread

## 1. Thread 란?

스레드(thread)란 프로세스(process) 내에서 실제로 작업을 수행하는 주체를 의미함.모든 프로세스에는 한 개 이상의 스레드가 존재하여 작업을 수행함.또한, 두 개 이상의 스레드를 가지는 프로세스를 멀티스레드 프로세스이다.

즉, 실제 server 에서 Clinet 로부터 Request(요청)이 들어오면 Backend는 요청을 받아 한개의 스레드를 생성하여 로직의 순서대로 진행. 하지만 멀티 스레드라면 2개의 스레드가 한개의 request를 처리.

(생각 해야 할 부분)멀티 스레드를 활용하면 작업을 분산시켜 빠르게 처리할 수 있다. 하지만 멀티 스레드를 사용할 때는 스레드 간의 동기화 문제와 공유 자원에 대한 문제에 대해 고려해야 한다. 이를 제대로 처리하지 않으면 예기치 않은 문제가 발생할 수 있으므로 주의가 필요하다.

## 2. 스레드 생성 및 실행 방법

1. **Runnable 인터페이스를 구현하는 방법**

Runnable 인터페이스를 구현하여 스레드를 생성하는 방법은 다음과 같다.

```java
public class MyRunnable implements Runnable {
    public void run() {
        // 스레드에서 실행할 로직 작성
    }
}

// 스레드 생성 및 실행
Thread myThread = new Thread(new MyRunnable());
myThread.start();

```

1. **Thread 클래스의 인스턴스를 생성하는 방법:**

Thread 클래스를 이용하여 스레드를 생성하는 방법은 다음과 같다.

```java
public class MyThread extends Thread {
    public void run() {
        // 스레드에서 실행할 로직 작성
    }
}

// 스레드 생성 및 실행
MyThread myThread = new MyThread();
myThread.start();

```

위 두 가지 방법 모두 스레드를 생성하고 실행하는 방법이다. 하지만 **Runnable 인터페이스를 구현하는 방법을 더 권장**한다. 이유는 자바는 다중 상속을 지원하지 않기 때문에 Thread 클래스를 상속받을 경우 다른 클래스를 상속받을 수 없기 때문이다. 반면, Runnable 인터페이스를 구현하는 방법은 다른 클래스를 상속받을 수 있기 때문에 더 유연한 코드를 작성할 수 있다.

## 3. Thread 생성 중 @Autowired

Spring 프레임워크를 사용하면, 스프링 컨테이너에서 관리되는 빈(Bean)으로부터 `@Autowired`를 사용하여 의존성을 주입할 수 있다. 하지만 Thread생성 중 Autowired를 한다면 Null Exception 에러를 뱉을 가능성이 농후하다. 그렇기 때문에 해당 클래스에서 Bean을 수동 주입해줘야 한다.

- ApplicationContextProvider.java

```java
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class ApplicationContextProvider implements ApplicationContextAware {

	private static ApplicationContext applicationContext;

	@Override
	public void setApplicationContext(ApplicationContext ac) throws BeansException {
		applicationContext = ac;
	}

	public static ApplicationContext getApplicationContext() {
		return applicationContext;
	}
	
	public static <T> T getBean(Class<T> clazz) {
		return applicationContext.getBean(clazz);
	}

}
```

- BatchQueueController .java

```java
public class BatchQueueController extends Thread{
    private aptChainContractExcutor aptChainContractExcutor;

    public void statusCheck() {
        aptChainContractExcutor = ApplicationContextProvider.getBean(aptChainContractExcutor.class);
    }
}
```