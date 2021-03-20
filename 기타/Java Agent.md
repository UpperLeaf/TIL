# Java Agent

FQCN이 example로 시작하는 클래스들의 start 메서드를 intercept한다.

JavaAgent + ByteBuddy

Advice를 적용하는 방식

```java
public class TestAgent {
    public static void premain(String args, Instrumentation inst){
        new AgentBuilder.Default()
                .type(nameStartsWith("example"))
                    .transform((builder, typeDescription, classLoader, module) ->
                         builder
                                 .method(nameStartsWith("start"))
                                 .intercept(Advice.to(MyInterceptor.class)))

                .installOn(inst);
    }
}

class MyInterceptor {
    @Advice.OnMethodEnter
    public static void enter() {
        System.out.println("Test Method Intercepted OnEnter");
    }

    @Advice.OnMethodExit
    public static void exit() { System.out.println("Test Method Intercepted OnExit"); }
}

```



Gradle 파일 (FatJar를 만들어서 JavaAgent에 모든 의존성을 추가해야한다. (shadow 플러그인 사용))

```groovy
group 'com.upperleaf'
version '1.0-SNAPSHOT'

apply plugin: 'java'
apply plugin: 'com.github.johnrengelman.shadow'

sourceCompatibility = 1.8

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:4.0.3'
    }
}

jar {
    finalizedBy shadowJar
    manifest {
        attributes 'Premain-Class': 'TestAgent',
        'Can-Redefine-Classes' : true,
        'Can-Retransform-Classes' : true
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation group: 'net.bytebuddy', name: 'byte-buddy', version: '1.10.18'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

