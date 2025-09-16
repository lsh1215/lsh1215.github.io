---
title: "자바 접근 제어자 완전 정리 [CS 면접 박살내기]"
date: 2025-05-04 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, 접근제어자, public, private, protected, default, 패키지, 캡슐화, 면접준비]
---

# 자바 접근 제어자 완전 정리 🔐

자바에서 클래스, 메서드, 변수에 붙이는 `public`, `private`, `protected` 같은 키워드들을 봤을 것이다. 이들은 단순한 키워드가 아니라 **접근 제어자(Access Modifier)** 라고 불리며, 객체 지향 프로그래밍의 핵심인 **캡슐화(Encapsulation)** 를 구현하는 중요한 도구다. 이번에는 4가지 접근 제어자를 패키지 구조와 함께 차근차근 알아보자.

## 접근 제어자란? 🤔

### 접근 제어자의 목적

접근 제어자는 **클래스, 메서드, 변수에 외부에서 접근할 수 있는 범위를 결정**하는 키워드다.

**주요 목적:**
- **캡슐화**: 객체의 내부 구현을 숨기고 외부 인터페이스만 노출
- **정보 은닉**: 중요한 데이터를 외부에서 직접 접근하지 못하게 보호
- **유지보수성**: 내부 구현이 바뀌어도 외부 코드에 영향 없음
- **안전성**: 잘못된 접근으로 인한 버그 방지

### 자바의 4가지 접근 제어자

| 접근 제어자 | 키워드 | 접근 범위 |
|-------------|--------|-----------|
| **public** | `public` | 어디서든 접근 가능 |
| **protected** | `protected` | 같은 패키지 + 다른 패키지의 자식 클래스 |
| **default** | (키워드 없음) | 같은 패키지 내에서만 |
| **private** | `private` | 같은 클래스 내에서만 |

## public: 모든 곳에서 접근 가능 🌍

### 특징

`public`은 **가장 넓은 접근 범위**를 가지며, 어디서든 접근할 수 있다.

```java
// 패키지: com.example.model
public class User {
    public String name;        // public 필드
    public int age;            // public 필드
    
    public User(String name, int age) {  // public 생성자
        this.name = name;
        this.age = age;
    }
    
    public void introduce() {  // public 메서드
        System.out.println("안녕하세요, " + name + "입니다.");
    }
}
```

### 다른 패키지에서도 접근 가능

```java
// 패키지: com.example.service
import com.example.model.User;

public class UserService {
    public void processUser() {
        User user = new User("김철수", 25);  // ✅ public 생성자 접근 가능
        user.name = "이영희";                // ✅ public 필드 접근 가능
        user.introduce();                   // ✅ public 메서드 접근 가능
    }
}
```

### public 사용 시 주의사항

```java
public class BankAccount {
    public double balance;  // ❌ 위험: 외부에서 직접 수정 가능
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}

// 외부에서 잘못된 사용
BankAccount account = new BankAccount();
account.balance = -1000;  // ❌ 음수 잔액 설정 가능!
```

**개선된 코드:**
```java
public class BankAccount {
    private double balance;  // ✅ private로 보호
    
    public double getBalance() {  // ✅ 안전한 접근 방법
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

## private: 클래스 내부에서만 접근 🔒

### 특징

`private`은 **가장 제한적인 접근 범위**를 가지며, 같은 클래스 내에서만 접근할 수 있다.

```java
public class Student {
    private String studentId;    // private 필드
    private String name;         // private 필드
    private int grade;           // private 필드
    
    // public 생성자로 초기화
    public Student(String studentId, String name, int grade) {
        this.studentId = studentId;
        this.name = name;
        this.grade = grade;
    }
    
    // private 메서드 - 내부 로직 처리
    private boolean isValidGrade(int grade) {
        return grade >= 1 && grade <= 6;
    }
    
    // public 메서드로 안전한 접근 제공
    public String getStudentId() {
        return studentId;
    }
    
    public String getName() {
        return name;
    }
    
    public int getGrade() {
        return grade;
    }
    
    public void setGrade(int grade) {
        if (isValidGrade(grade)) {  // private 메서드 호출
            this.grade = grade;
        }
    }
}
```

### 외부에서 접근 시도

```java
public class Main {
    public static void main(String[] args) {
        Student student = new Student("2024001", "김철수", 3);
        
        // ✅ public 메서드로 접근 가능
        System.out.println(student.getName());  // 김철수
        System.out.println(student.getGrade()); // 3
        
        // ❌ private 필드 직접 접근 불가
        // student.name = "이영희";  // 컴파일 에러!
        // student.grade = 5;        // 컴파일 에러!
        
        // ✅ public 메서드로 안전하게 수정
        student.setGrade(5);  // 유효성 검사 후 수정
    }
}
```

## protected: 상속 관계에서 접근 가능 👨‍👩‍👧‍👦

### 특징

`protected`는 **같은 패키지 내**에서 접근 가능하고, **다른 패키지의 자식 클래스**에서도 접근할 수 있다.

```java
// 패키지: com.example.animal
public class Animal {
    protected String name;        // protected 필드
    protected int age;            // protected 필드
    
    protected void makeSound() {  // protected 메서드
        System.out.println("동물이 소리를 냅니다.");
    }
    
    public void introduce() {
        System.out.println("안녕하세요, " + name + "입니다.");
    }
}
```

### 같은 패키지에서 접근

```java
// 패키지: com.example.animal
public class AnimalCare {
    public void careForAnimal(Animal animal) {
        // ✅ 같은 패키지이므로 protected 접근 가능
        animal.name = "멍멍이";
        animal.age = 3;
        animal.makeSound();
    }
}
```

### 다른 패키지의 자식 클래스에서 접근

```java
// 패키지: com.example.pet
import com.example.animal.Animal;

public class Dog extends Animal {
    public Dog(String name, int age) {
        // ✅ 자식 클래스이므로 protected 접근 가능
        this.name = name;
        this.age = age;
    }
    
    @Override
    protected void makeSound() {
        System.out.println("멍멍!");
    }
    
    public void play() {
        // ✅ 부모 클래스의 protected 메서드 호출 가능
        makeSound();
        introduce();
    }
}
```

### 다른 패키지의 일반 클래스에서 접근

```java
// 패키지: com.example.service
import com.example.animal.Animal;

public class AnimalService {
    public void processAnimal(Animal animal) {
        // ❌ 다른 패키지의 일반 클래스이므로 protected 접근 불가
        // animal.name = "고양이";     // 컴파일 에러!
        // animal.makeSound();        // 컴파일 에러!
        
        // ✅ public 메서드만 접근 가능
        animal.introduce();
    }
}
```

## default: 패키지 내에서만 접근 📦

### 특징

`default` 접근 제어자는 **키워드를 명시하지 않을 때** 적용되며, **같은 패키지 내에서만** 접근 가능하다.

```java
// 패키지: com.example.model
class Person {  // default 클래스
    String name;     // default 필드
    int age;         // default 필드
    
    Person(String name, int age) {  // default 생성자
        this.name = name;
        this.age = age;
    }
    
    void introduce() {  // default 메서드
        System.out.println("안녕하세요, " + name + "입니다.");
    }
}
```

### 같은 패키지에서 접근

```java
// 패키지: com.example.model
public class PersonService {
    public void processPerson() {
        Person person = new Person("김철수", 25);  // ✅ 같은 패키지
        person.name = "이영희";                    // ✅ default 필드 접근 가능
        person.introduce();                       // ✅ default 메서드 접근 가능
    }
}
```

### 다른 패키지에서 접근

```java
// 패키지: com.example.service
import com.example.model.Person;  // ❌ 컴파일 에러! default 클래스는 import 불가

public class Service {
    public void doSomething() {
        // Person person = new Person("김철수", 25);  // ❌ 접근 불가
    }
}
```

## 패키지 구조와 접근 제어자 🏗️

### 패키지 계층 구조

```
com.example
├── model
│   ├── User.java          (public class)
│   ├── Person.java        (default class)
│   └── Address.java       (public class)
├── service
│   ├── UserService.java   (public class)
│   └── PersonService.java (public class)
└── util
    ├── StringUtil.java    (public class)
    └── DateUtil.java      (default class)
```

### 접근 제어자별 접근 가능 범위

```java
// com.example.model.User (public class)
public class User {
    public String name;           // 어디서든 접근 가능
    protected String email;       // 같은 패키지 + 자식 클래스
    String phone;                 // 같은 패키지에서만
    private String password;      // 같은 클래스에서만
}

// com.example.service.UserService (public class)
public class UserService {
    public void processUser() {
        User user = new User();
        user.name = "김철수";      // ✅ public
        user.email = "kim@example.com";  // ✅ protected (같은 패키지)
        user.phone = "010-1234-5678";    // ✅ default (같은 패키지)
        // user.password = "1234";       // ❌ private
    }
}

// com.example.other.OtherService (다른 패키지)
public class OtherService {
    public void processUser() {
        User user = new User();
        user.name = "김철수";      // ✅ public
        // user.email = "kim@example.com";  // ❌ protected (다른 패키지)
        // user.phone = "010-1234-5678";    // ❌ default (다른 패키지)
        // user.password = "1234";          // ❌ private
    }
}
```

## 접근 제어자 선택 가이드 📋

### 클래스에 사용할 접근 제어자

| 상황 | 접근 제어자 | 이유 |
|------|-------------|------|
| **라이브러리 API** | `public` | 외부에서 사용해야 함 |
| **내부 구현 클래스** | `default` | 같은 패키지에서만 사용 |
| **상속용 기본 클래스** | `public` | 다른 패키지에서도 상속 가능 |
| **유틸리티 클래스** | `public` + `final` | 상속 방지하면서 사용 가능 |

### 필드에 사용할 접근 제어자

| 상황 | 접근 제어자 | 이유 |
|------|-------------|------|
| **상수** | `public static final` | 어디서든 접근 가능하지만 수정 불가 |
| **인스턴스 변수** | `private` | 캡슐화를 위해 직접 접근 방지 |
| **상속용 필드** | `protected` | 자식 클래스에서 접근 가능 |
| **패키지 내 공유** | `default` | 같은 패키지에서만 접근 |

### 메서드에 사용할 접근 제어자

| 상황 | 접근 제어자 | 이유 |
|------|-------------|------|
| **외부 API** | `public` | 외부에서 호출 가능 |
| **내부 로직** | `private` | 구현 세부사항 숨김 |
| **상속용 메서드** | `protected` | 자식 클래스에서 오버라이드 가능 |
| **패키지 내 공유** | `default` | 같은 패키지에서만 사용 |

## 면접용 핵심 정리 🎯

### 접근 제어자 4가지
> "자바에는 public, protected, default, private 4가지 접근 제어자가 있다. public은 어디서든 접근 가능하고, protected는 같은 패키지와 자식 클래스에서 접근 가능하며, default는 같은 패키지에서만, private은 같은 클래스에서만 접근 가능하다."

### 캡슐화와 접근 제어자
> "접근 제어자는 객체 지향 프로그래밍의 캡슐화를 구현하는 핵심 도구다. private으로 내부 구현을 숨기고, public 메서드를 통해 안전한 접근을 제공함으로써 데이터 무결성과 코드의 유지보수성을 보장한다."

### 실무에서의 활용
> "실무에서는 필드는 대부분 private으로 선언하고 getter/setter를 제공하며, public 메서드는 외부 API로, protected 메서드는 상속용으로, private 메서드는 내부 구현용으로 사용한다. 또한 final 클래스는 상속을 방지하고, private 생성자는 인스턴스 생성을 방지할 때 사용한다."

### 패키지와 접근 제어자
> "패키지 구조는 접근 제어자와 밀접한 관련이 있다. default 접근 제어자는 같은 패키지 내에서만 접근 가능하므로, 관련된 클래스들을 같은 패키지에 배치하여 내부 구현을 숨기고 외부 인터페이스만 노출할 수 있다."

## 결론 🎉

접근 제어자는 자바의 객체 지향 프로그래밍을 위한 핵심 도구다.

- **public**: 최대한 열린 접근, 외부 API용
- **protected**: 상속 관계에서 접근, 확장성 고려
- **default**: 패키지 내 제한, 내부 구현용
- **private**: 최대한 제한, 캡슐화 핵심

적절한 접근 제어자를 선택하여 **안전하고 유지보수 가능한 코드**를 작성하는 것이 중요하다. 이런 기본 개념을 정확히 이해하고 있다면, 면접에서도 자신 있게 답변할 수 있을 것이다! 💪
