# JVM Byte code 수정

### **Java Agent와 Byte Code Instrumentation**

**Instrumentation -** 오류를 진단하거나, 추적 정보를 쓰기 위해 성능 정도를 모니터링하거나 측정하는 기능

[java.lang.instrument (Java Platform SE 6)](https://docs.oracle.com/javase/6/docs/api/java/lang/instrument/package-summary.html)

JavaAgent는 2가지의 실행 시점을 가진다.

- premain(String args, Instrumentation inst) : 애플리케이션 실행전 Agent 실행
- agentmain(String args, Instrumentation inst) : 애플리케이션 실행후 Agent 실행

Agent 실행 시점에 따라 둘중 하나만 호출된다.

```java
public static void premain(String args, Instrumentation inst) {
}
public static void agentmain(String args, Instrumentation inst) {
}
```

Instrumentation 객체는 JVM 위에서 동작하는 프로그램의 Instrument를 수행하게 허락해준다.

1. Class가 Load될때 Class의 Bytecode를 Transformation한다. 이것은 ClassFileTransformer 인터페이스의 구현체로 수행할 수 있다.
2. JVM에 이미 로드된 개별적인 Class들을 완벽히 재정의 할 수 있다.

JavaAgent는 Jar파일로 반드시 패키징되어야한다. 또한 MANIFEST.MF 파일에 다음과 같은 Attributes들이 필요하다.

- Premain-class
- Can-Redefine-class
- Can-Retransform-class
- Agent-class

```yaml
jar {
    manifest {
        attributes 'Premain-class' : 'TestAgent',
				'Can-Redefine-Classes' : true,
			  'Can-Retransform-Classes' : true
    }
}
```

**Java Web Application Server인 Tomcat의 Request에 Byte Code를 조작해서, Request 시작과 Request 종료시에 데이터를 수집하는 Collector에게 데이터를 보내자. ⇒ (Tomcat Servlet 기반의 Java 웹 어플리케이션 성능 모니터링)**

---

(Agent를 만들어서 다른걸 더 추가하고 싶다면 추가할 수 있을듯 하다.)

1. **Tomcat이 언제 Request를 보내고 언제 Response를 보내는가? (Tomcat의 소스코드를 확인해볼 필요가 있음)**
2. **데이터를 어떻게 보내는가? (HTTP보다는 Socket를 하나 만들어서 데이터를 보내는게 리소스상 좋아보인다.)**
3. **Tomcat의 소스코드를 어떻게 수정할 것인가?**

Slide Share

[Bytecode Manipulation with a Java Agent and Byte Buddy](https://www.slideshare.net/jyukutyo/bytecode-manipulation-with-a-java-agent-and-byte-buddy)

### Byte Code를 수정하는 방법

1. ByteBuddy

[Byte Buddy - runtime code generation for the Java virtual machine](https://bytebuddy.net/#/)