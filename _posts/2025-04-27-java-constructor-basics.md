---
title: "자바 생성자 완전 정리 [CS 면접 박살내기]"
date: 2025-04-27 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, 생성자, 기본생성자, Spring, JPA, Entity, 면접준비]
---

# 자바 생성자 완전 정리 🏗️

자바에서 객체를 생성할 때 가장 기본이 되는 것이 바로 **생성자(Constructor)**다. 그런데 생성자를 안 만들면 어떻게 될까? 또한, Spring에서 `@Entity`를 사용할 때 왜 `NoArgument` 생성자가 필요한지, 왜 `protected`를 써야 하는지까지 차근차근 알아보자.

## 생성자가 없으면 어떻게 될까? 🤔

### ✅ 기본 생성자(Default Constructor) 자동 생성

자바 클래스에서 **생성자를 하나도 정의하지 않으면**, 컴파일러가 자동으로 기본 생성자를 만들어준다.

```java
class User {
    String name;
    int age;
    boolean isActive;
}

// 컴파일 시 아래와 같은 생성자가 자동 추가됨
// public User() { }
```

**기본 생성자의 특징:**
- **매개변수가 없음**
- **접근 제어자는 클래스와 동일** (public, protected, package-private)
- **필드들을 기본값으로 초기화** (null, 0, false 등)

### 🔍 기본값 초기화 확인

```java
public class ConstructorExample {
    public static void main(String[] args) {
        User user = new User(); // 기본 생성자 호출
        
        System.out.println("Name: " + user.name);        // null
        System.out.println("Age: " + user.age);          // 0
        System.out.println("IsActive: " + user.isActive); // false
    }
}

class User {
    String name;    // null
    int age;        // 0
    boolean isActive; // false
}
```

## 생성자를 하나라도 만들면? ⚠️

### ❗ 기본 생성자 자동 생성 안됨

**생성자를 하나라도 직접 정의하면**, 컴파일러는 기본 생성자를 자동으로 만들지 않는다.

```java
class User {
    String name;
    int age;

    // 매개변수가 있는 생성자 정의
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // ❗ 기본 생성자 없음! → new User() 하면 컴파일 에러
}

public class Test {
    public static void main(String[] args) {
        User user1 = new User("김철수", 25); // ✅ 가능
        User user2 = new User(); // ❌ 컴파일 에러!
        // error: constructor User in class User cannot be applied to given types
    }
}
```

### ✅ 해결 방법: 명시적으로 기본 생성자 정의

```java
class User {
    String name;
    int age;

    // 기본 생성자 명시적 정의
    public User() {
        // 기본값으로 초기화하거나 다른 로직 수행
    }

    // 매개변수가 있는 생성자
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

## 생성자 오버로딩과 this() 🎯

### 생성자 오버로딩

```java
class Person {
    private String name;
    private int age;
    private String email;

    // 기본 생성자
    public Person() {
        this("이름없음", 0, "이메일없음");
    }

    // 이름만 받는 생성자
    public Person(String name) {
        this(name, 0, "이메일없음");
    }

    // 이름과 나이를 받는 생성자
    public Person(String name, int age) {
        this(name, age, "이메일없음");
    }

    // 모든 필드를 받는 생성자
    public Person(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
}
```

### this() 사용의 장점

```java
class Product {
    private String name;
    private int price;
    private String category;

    public Product() {
        this("상품명", 0, "카테고리");
    }

    public Product(String name) {
        this(name, 0, "카테고리");
    }

    public Product(String name, int price) {
        this(name, price, "카테고리");
    }

    // 실제 초기화 로직은 한 곳에서만 관리
    public Product(String name, int price, String category) {
        this.name = name;
        this.price = price;
        this.category = category;
        
        // 공통 초기화 로직
        validateProduct();
        logProductCreation();
    }

    private void validateProduct() {
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("상품명은 필수입니다.");
        }
        if (price < 0) {
            throw new IllegalArgumentException("가격은 0 이상이어야 합니다.");
        }
    }

    private void logProductCreation() {
        System.out.println("상품이 생성되었습니다: " + name);
    }
}
```

## Spring JPA에서의 생성자 🚀

### @Entity와 NoArgument 생성자

Spring JPA에서 `@Entity`를 사용할 때는 **반드시 NoArgument 생성자가 필요**하다.

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false)
    private String name;
    
    @Column(name = "age")
    private Integer age;
    
    // ❌ 이 생성자만 있으면 안됨
    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
    
    // ✅ NoArgument 생성자 필수!
    protected User() {
        // JPA가 사용할 수 있도록 비워둠
    }
}
```

### 왜 NoArgument 생성자가 필요한가? 🤔

**JPA가 객체를 생성하는 과정:**
1. **리플렉션을 통해 클래스 정보 조회**
2. **NoArgument 생성자로 객체 생성**
3. **setter나 필드 접근을 통해 값 설정**

```java
// JPA 내부 동작 과정 (의사코드)
public <T> T createEntity(Class<T> entityClass) {
    try {
        // 1. NoArgument 생성자로 객체 생성
        T instance = entityClass.getDeclaredConstructor().newInstance();
        
        // 2. 리플렉션으로 필드에 값 설정
        Field[] fields = entityClass.getDeclaredFields();
        for (Field field : fields) {
            field.setAccessible(true);
            // 데이터베이스에서 가져온 값으로 설정
            field.set(instance, valueFromDatabase);
        }
        
        return instance;
    } catch (Exception e) {
        throw new RuntimeException("엔티티 생성 실패", e);
    }
}
```

### 왜 protected를 사용하나? 🛡️

```java
@Entity
public class User {
    // ✅ protected 권장
    protected User() {
        // JPA 전용 생성자
    }
    
    // ✅ public 생성자도 가능하지만 권장하지 않음
    public User() {
        // 외부에서 직접 호출 가능
    }
}
```

**protected를 사용하는 이유:**

1. **의도 명확화**: 이 생성자는 JPA에서만 사용한다는 의미
2. **캡슐화**: 외부에서 직접 호출하는 것을 방지
3. **안전성**: 잘못된 상태의 객체 생성 방지

```java
// ❌ 외부에서 직접 호출하면 안전하지 않음
User user = new User(); // name이 null인 상태로 생성됨

// ✅ Builder 패턴이나 팩토리 메서드 사용 권장
User user = User.builder()
    .name("김철수")
    .age(25)
    .build();
```

## 실무에서의 생성자 패턴 🏭

### 1. Builder 패턴 + Lombok

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED) // JPA용
@AllArgsConstructor
@Builder
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private Integer age;
    private String email;
}
```

### 2. 팩토리 메서드 패턴

```java
@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private int price;
    private String category;
    
    // JPA용 생성자
    protected Product() {}
    
    // 팩토리 메서드들
    public static Product createBook(String name, int price) {
        Product product = new Product();
        product.name = name;
        product.price = price;
        product.category = "BOOK";
        return product;
    }
    
    public static Product createElectronics(String name, int price) {
        Product product = new Product();
        product.name = name;
        product.price = price;
        product.category = "ELECTRONICS";
        return product;
    }
}
```

## 면접용 핵심 정리 🎯

### 기본 생성자 자동 생성
> "자바 클래스에서 생성자를 하나도 정의하지 않으면, 컴파일러가 자동으로 매개변수가 없는 기본 생성자를 만들어준다. 이 생성자는 클래스와 같은 접근 제어자를 가지며, 필드들을 기본값으로 초기화한다."

### 생성자 정의 시 주의사항
> "생성자를 하나라도 직접 정의하면 컴파일러는 기본 생성자를 자동으로 만들지 않는다. 따라서 기본 생성자가 필요한 경우 명시적으로 정의해야 하며, this()를 사용하여 생성자 체이닝을 통해 코드 중복을 줄일 수 있다."

### Spring JPA에서의 생성자
> "Spring JPA의 @Entity 클래스에서는 리플렉션을 통해 객체를 생성하므로 반드시 NoArgument 생성자가 필요하다. 이 생성자는 protected로 선언하여 JPA에서만 사용하도록 제한하는 것이 좋다."

### 실무에서의 생성자 패턴
> "실무에서는 Lombok의 @Builder, 팩토리 메서드 패턴을 활용하여 안전하고 유연한 객체 생성을 구현한다. 특히 JPA 엔티티의 경우 protected NoArgument 생성자와 Lombok Builder를 조합하여 사용한다."

## 결론 🎉

자바의 생성자는 객체 지향 프로그래밍의 핵심 요소로, 올바른 이해와 사용이 중요하다.

- **기본 생성자**: 컴파일러가 자동 생성하지만, 생성자를 정의하면 사라짐
- **생성자 오버로딩**: 다양한 초기화 방법 제공
- **Spring JPA**: NoArgument 생성자 필수, protected 권장
- **실무 패턴**: Lombok Builder, 팩토리 메서드 활용

이런 기본 개념을 정확히 이해하고 있다면, 면접에서도 자신 있게 답변할 수 있을 것이다! 💪
