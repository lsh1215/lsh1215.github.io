---
title: "자바 메모리 구조 완전 정리 [CS 면접 박살내기]"
date: 2025-03-30 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, JVM, 메모리, Stack, Heap, Method Area, 면접준비]
---

# 자바 메모리 구조 완전 정리 🧠

자바를 배우면서 가장 헷갈렸던 부분이 바로 메모리 구조였다. 변수가 어디에 저장되는지, 객체는 어떻게 관리되는지, static 변수는 왜 다른 곳에 저장되는지... 이번에는 JVM의 메모리 구조를 차근차근 파헤쳐보자.

## JVM 메모리 구조 개요 📊

JVM(Java Virtual Machine)은 메모리를 크게 **3개의 주요 영역**으로 나누어 관리한다:

1. **스택 영역(Stack Area)** ⚡ - 메서드 호출과 지역변수
2. **힙 영역(Heap Area)** 🗂️ - 객체와 배열
3. **메서드 영역(Method Area)** 📚 - 클래스 정보와 static 변수

![JVM 메모리 구조 전체 다이어그램](/assets/img/posts/2025-03-30-java-memory-structure/jvm_1.png)

## 스택 영역(Stack Area) ⚡

스택 영역은 **메서드 호출과 관련된 정보**를 저장하는 공간이다. 가장 빠르고 효율적인 메모리 관리가 가능하다.

### 스택의 특징

- **스레드별 독립적**: 각 스레드는 자신만의 스택을 가진다
- **LIFO 구조**: Last-In, First-Out 방식으로 동작
- **빠른 속도**: 메모리 할당/해제가 매우 빠르다
- **자동 관리**: 메서드 종료 시 자동으로 메모리가 해제된다

### 스택 프레임(Stack Frame) 🔄

메서드가 호출될 때마다 **스택 프레임**이라는 독립적인 공간이 생성된다.

<img src="/assets/img/posts/2025-03-30-java-memory-structure/jvm_2.png" alt="스택 프레임 구조" width="50%" style="max-width: 400px; height: auto;">

```java
public class StackExample {
    public static void main(String[] args) {
        int a = 10;           // main 메서드의 스택 프레임
        String name = "Java"; // main 메서드의 스택 프레임
        
        calculate(a);         // calculate 메서드 호출
    }
    
    public static void calculate(int num) {
        int result = num * 2; // calculate 메서드의 스택 프레임
        System.out.println(result);
    } // calculate 메서드 종료 시 스택 프레임 제거
}
```

### 스택 프레임 내부 구조

각 스택 프레임은 다음과 같은 정보를 포함한다:

1. **지역변수 배열(Local Variable Array)**: 메서드의 매개변수와 지역변수
2. **피연산자 스택(Operand Stack)**: 연산을 위한 임시 데이터
3. **실행 환경 정보**: 메서드 실행 상태 정보
4. **상수 풀 참조**: 상수에 대한 참조

![스택 프레임 구조](/assets/img/posts/2025-03-30-java-memory-structure/jvm_3.png)

### 스택에서의 변수 저장

```java
public void example() {
    int primitive = 100;        // 스택에 값 직접 저장
    String reference = "Hello"; // 스택에 힙 주소 저장
    Object obj = new Object();  // 스택에 힙 주소 저장
}
```

- **원시 타입**: 스택에 값이 직접 저장됨
- **참조 타입**: 스택에 힙 메모리의 주소가 저장됨

## 힙 영역(Heap Area) 🗂️

힙 영역은 **객체와 배열**이 저장되는 공간이다. 모든 스레드가 공유하는 메모리 영역이다.

### 힙의 특징

- **모든 스레드 공유**: 모든 스레드가 접근 가능
- **가비지 컬렉션**: 자동으로 메모리 관리
- **동적 할당**: 런타임에 크기가 결정됨
- **상대적으로 느림**: 스택에 비해 접근 속도가 느림

![힙 영역 구조](/assets/img/posts/2025-03-30-java-memory-structure/heap_1.webp)

### 힙에서의 객체 저장

```java
public class HeapExample {
    public static void main(String[] args) {
        // 스택에 참조 변수, 힙에 실제 객체
        Person person = new Person("김철수", 25);
        int[] numbers = {1, 2, 3, 4, 5};
        String text = "Hello World";
    }
}

class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

### 메모리 참조 관계 🔗

```java
public void memoryExample() {
    // 스택: person 변수
    // 힙: Person 객체 (name, age 포함)
    Person person = new Person("김철수", 25);
    
    // 스택: numbers 변수  
    // 힙: int 배열 [1, 2, 3, 4, 5]
    int[] numbers = {1, 2, 3, 4, 5};
}
```

![스택과 힙의 참조 관계](/assets/img/posts/2025-03-30-java-memory-structure/jvm_4.png)

## 메서드 영역(Method Area) 📚

메서드 영역은 **클래스 정보와 static 변수**를 저장하는 공간이다. 모든 스레드가 공유한다.

### 메서드 영역의 특징

- **모든 스레드 공유**: 클래스 정보는 공통으로 사용
- **클래스 로딩 시 생성**: 클래스가 처음 사용될 때 생성
- **프로그램 종료까지 유지**: static 변수는 프로그램이 끝날 때까지 유지

![메서드 영역 구조](/assets/img/posts/2025-03-30-java-memory-structure/jvm_5.png)

### 메서드 영역에 저장되는 정보

1. **클래스 정보**: 클래스의 메타데이터
2. **메서드 코드**: 컴파일된 바이트코드
3. **static 변수**: 클래스 변수
4. **상수 풀**: 문자열 리터럴, 상수
5. **필드 정보**: 클래스의 필드 메타데이터

### Static 변수의 저장 위치

```java
public class StaticExample {
    // 메서드 영역에 저장됨
    private static int count = 0;
    private static String company = "TechCorp";
    
    // 스택에 저장됨 (지역변수)
    private int instanceVar = 10;
    
    public static void main(String[] args) {
        // static 변수는 메서드 영역에서 직접 접근
        count++;
        System.out.println("Count: " + count);
        
        // 지역변수는 스택에 저장
        int localVar = 20;
    }
}
```

### Java 버전별 변화 🔄

| Java 버전 | 메서드 영역 | 특징 |
|-----------|-------------|------|
| **Java 7 이전** | Permanent Generation (PermGen) | 힙 영역 내부, 고정 크기 |
| **Java 8 이후** | Metaspace | 네이티브 메모리, 동적 크기 |

**PermGen의 문제점:**
- 크기가 고정되어 있어 `OutOfMemoryError: PermGen space` 발생
- 클래스 로딩이 많을 때 메모리 부족

**Metaspace의 장점:**
- 네이티브 메모리 사용으로 크기 제한 없음
- 동적으로 크기 조절 가능
- PermGen 관련 오류 해결

## 실제 메모리 사용 예시 💻

```java
public class MemoryExample {
    // 메서드 영역에 저장
    private static String className = "MemoryExample";
    private static int totalCount = 0;
    
    // 인스턴스 변수 (객체 생성 시 힙에 저장)
    private String name;
    private int age;
    
    public MemoryExample(String name, int age) {
        this.name = name;  // 힙에 저장
        this.age = age;    // 힙에 저장
        totalCount++;      // 메서드 영역의 static 변수 수정
    }
    
    public void processData() {
        // 스택에 저장 (지역변수)
        int localVar = 100;
        String localString = "Processing...";
        
        // 힙에 객체 생성, 스택에 참조 저장
        List<String> list = new ArrayList<>();
        list.add("Item 1");
        list.add("Item 2");
        
        // static 메서드 호출
        printInfo();
    }
    
    public static void printInfo() {
        // static 메서드도 스택 프레임 생성
        System.out.println("Class: " + className);
        System.out.println("Total Count: " + totalCount);
    }
}
```

## 메모리 영역별 변수 저장 위치 📍

| 변수 타입 | 저장 위치 | 접근 방식 | 생명주기 |
|-----------|-----------|-----------|----------|
| **지역변수** | 스택 | 직접 접근 | 메서드 종료 시 |
| **인스턴스 변수** | 힙 | 객체 참조 | 객체 GC 시 |
| **static 변수** | 메서드 영역 | 클래스명.변수명 | 프로그램 종료 시 |
| **매개변수** | 스택 | 직접 접근 | 메서드 종료 시 |

## 메모리 관리 주의사항 ⚠️

### 1. 메모리 누수 방지

```java
// 잘못된 예시 - 메모리 누수 가능성
public class BadExample {
    private static List<Object> cache = new ArrayList<>();
    
    public void addToCache(Object obj) {
        cache.add(obj); // 계속 추가만 하고 제거하지 않음
    }
}

// 올바른 예시 - 적절한 크기 제한
public class GoodExample {
    private static final int MAX_SIZE = 100;
    private static List<Object> cache = new ArrayList<>();
    
    public void addToCache(Object obj) {
        if (cache.size() >= MAX_SIZE) {
            cache.remove(0); // 오래된 항목 제거
        }
        cache.add(obj);
    }
}
```

### 2. Static 변수 사용 주의

```java
public class StaticWarning {
    // 위험: 너무 많은 static 변수
    private static Map<String, Object> globalCache = new HashMap<>();
    private static List<String> globalList = new ArrayList<>();
    
    // 권장: 필요한 경우에만 static 사용
    private static final String CONSTANT_VALUE = "Fixed";
    private static int counter = 0;
}
```

### 3. 스택 오버플로우 방지

```java
// 위험: 무한 재귀 호출
public class StackOverflowExample {
    public void recursiveMethod() {
        recursiveMethod(); // 스택 오버플로우 발생 가능
    }
    
    // 안전: 종료 조건이 있는 재귀
    public void safeRecursive(int count) {
        if (count <= 0) return; // 종료 조건
        safeRecursive(count - 1);
    }
}
```

## 면접용 핵심 정리 🎯

### 스택 영역 ⚡
> "스택 영역은 메서드 호출 시 생성되는 스택 프레임을 저장하는 공간으로, 지역변수와 매개변수가 저장되며 LIFO 방식으로 관리된다. 각 스레드마다 독립적이며 메서드 종료 시 자동으로 해제된다."

### 힙 영역 🗂️
> "힙 영역은 new 키워드로 생성된 객체와 배열이 저장되는 공간으로, 모든 스레드가 공유하며 가비지 컬렉터에 의해 자동으로 메모리가 관리된다. 참조 타입 변수는 스택에 주소를, 실제 객체는 힙에 저장된다."

### 메서드 영역 📚
> "메서드 영역은 클래스 정보, static 변수, 상수 풀을 저장하는 공간으로, 모든 스레드가 공유한다. Java 8 이후에는 PermGen이 Metaspace로 변경되어 네이티브 메모리를 사용하며 동적으로 크기가 조절된다."

### 메모리 참조 관계 🔗
> "지역변수는 스택에 저장되지만, 참조 타입의 경우 스택에는 힙의 주소가 저장되고 실제 객체는 힙에 저장된다. static 변수는 메서드 영역에 저장되어 프로그램 종료까지 유지된다."

## 결론 🎉

자바의 메모리 구조를 이해하는 것은 효율적인 프로그래밍과 메모리 관리에 필수적이다.

- **스택**: 빠르고 자동 관리되는 메서드 호출 공간 ⚡
- **힙**: 객체와 배열이 저장되는 공유 공간 🗂️  
- **메서드 영역**: 클래스 정보와 static 변수가 저장되는 공간 📚

이런 기본 구조를 정확히 이해하고 있다면, 메모리 누수나 성능 문제를 예방하고 면접에서도 자신 있게 답변할 수 있을 것이다! 💪
