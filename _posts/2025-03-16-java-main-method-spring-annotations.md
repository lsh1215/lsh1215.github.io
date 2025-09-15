---
title: "자바 main 메서드와 Spring 어노테이션 [CS 면접 박살내기]"
date: 2025-03-16 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, Spring, main method, '@SpringBootApplication', 면접준비]
---

# 자바 main 메서드와 Spring 어노테이션

자바를 처음 배울 때 가장 헷갈렸던 부분이 바로 main 메서드의 역할이었다. 그런데 Spring을 배우면서 main 메서드에 붙는 어노테이션들이 뭔지, 왜 붙는지 궁금해졌다. 이번에는 자바의 main 메서드부터 Spring의 어노테이션까지 차근차근 알아보자.

## 자바에서 `main` 메서드란?

main 메서드는 자바 애플리케이션이 시작되는 진입점(entry point)이다.

자바 프로그램을 실행하면 **JVM이 가장 먼저 호출하는 메서드**가 바로 이 `main`이다.

즉, 우리가 작성한 코드가 JVM에 의해 실행되기 위한 시작 지점이다.

## 메서드 시그니처

```java
public static void main(String[] args)
```

각 키워드의 의미는 다음과 같다:

| 키워드 | 의미 |
| --- | --- |
| `public` | JVM이 **어디서든 접근 가능**해야 하므로 public |
| `static` | 객체 생성 없이 클래스만으로 실행 가능해야 하므로 static |
| `void` | 반환값이 없다는 뜻 (시작점이므로 아무것도 반환 안 함) |
| `main` | **JVM이 약속한 이름**. 정확히 이 이름이어야 실행됨 |
| `String[] args` | **명령줄 인자**를 받기 위한 배열 (필요 없으면 안 써도 됨, 하지만 선언은 필수) |

## 기본 예시 코드

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

## Spring에서의 main 메서드

그런데 Spring을 사용할 때는 main 메서드가 조금 다르다. Spring Boot 애플리케이션에서는 main 메서드에 특별한 어노테이션들이 붙는다.

### Spring Boot의 main 메서드

![Spring Boot main 메서드 예시](/assets/img/posts/2025-03-16-java-main-method-spring-annotations/spring_1.png)

여기서 `@SpringBootApplication` 어노테이션이 핵심이다. 이 어노테이션 하나로 Spring Boot의 모든 기능을 활성화할 수 있다.

## @SpringBootApplication의 역할

`@SpringBootApplication`은 사실 여러 어노테이션을 합친 것이다:

![@SpringBootApplication 어노테이션 구조](/assets/img/posts/2025-03-16-java-main-method-spring-annotations/spring_2.png)

### 1. @SpringBootConfiguration

![@SpringBootConfiguration 어노테이션](/assets/img/posts/2025-03-16-java-main-method-spring-annotations/spring_3.png)

- `@Configuration`의 특별한 버전
- Spring의 설정 클래스임을 나타냄
- `@Bean` 어노테이션을 사용해서 빈을 정의할 수 있음

### 2. @EnableAutoConfiguration

![@EnableAutoConfiguration 어노테이션](/assets/img/posts/2025-03-16-java-main-method-spring-annotations/spring_4.png)

- Spring Boot의 **자동 설정** 기능을 활성화
- 클래스패스에 있는 라이브러리들을 보고 자동으로 설정을 구성
- 예: `spring-boot-starter-web`이 있으면 자동으로 웹 서버 설정

### 3. @ComponentScan

![@ComponentScan 어노테이션](/assets/img/posts/2025-03-16-java-main-method-spring-annotations/spring_5.png)

- `@Component`, `@Service`, `@Repository`, `@Controller` 등의 어노테이션이 붙은 클래스들을 자동으로 스캔해서 빈으로 등록
- 기본적으로 main 메서드가 있는 패키지부터 하위 패키지까지 스캔

## 실제 Spring Boot 애플리케이션 예시

```java
@SpringBootApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}

@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello, Spring Boot!";
    }
}

@Service
public class HelloService {
    public String getMessage() {
        return "Hello from Service!";
    }
}
```

## SpringApplication.run()의 역할

```java
SpringApplication.run(Application.class, args);
```

이 메서드가 하는 일들:

1. **Spring 컨테이너 생성**
2. **자동 설정 로드** (`@EnableAutoConfiguration`)
3. **컴포넌트 스캔** (`@ComponentScan`)
4. **웹 서버 시작** (웹 애플리케이션인 경우)
5. **애플리케이션 실행**

## 다른 Spring 어노테이션들

### @Configuration

```java
@Configuration
public class AppConfig {
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

- Spring의 설정 클래스임을 나타냄
- `@Bean` 어노테이션으로 빈을 정의할 수 있음

### @Component

```java
@Component
public class MyComponent {
    // ...
}
```

- Spring이 관리하는 컴포넌트임을 나타냄
- 자동으로 빈으로 등록됨

### @Service, @Repository, @Controller

```java
@Service
public class UserService {
    // ...
}

@Repository
public class UserRepository {
    // ...
}

@Controller
public class UserController {
    // ...
}
```

- 각각의 역할에 맞는 어노테이션
- 모두 `@Component`의 특별한 버전
- 자동으로 빈으로 등록됨

## 주의할 점

- `main` 메서드가 없으면 자바 애플리케이션은 실행되지 않음
- 정확한 시그니처가 아니면 `main method not found` 에러 발생
- `String[] args`는 명령어 실행 시 인자를 받을 수 있는 수단 (예: `java MyApp hello world`)
- Spring Boot에서는 `@SpringBootApplication`이 붙은 클래스의 main 메서드가 진입점
- `@ComponentScan`의 범위를 잘못 설정하면 빈이 등록되지 않을 수 있음

## 면접용 요약

> "main 메서드는 JVM이 프로그램을 시작할 때 가장 먼저 호출하는 진입점이며, Spring Boot에서는 @SpringBootApplication 어노테이션이 붙은 클래스의 main 메서드가 애플리케이션의 시작점이 된다. @SpringBootApplication은 @Configuration, @EnableAutoConfiguration, @ComponentScan을 합친 어노테이션으로, 자동 설정과 컴포넌트 스캔을 통해 Spring 컨테이너를 구성한다."
