---
title: "자바 String 완전 정리 [CS 면접 박살내기]"
date: 2025-04-06 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, String, StringBuilder, StringPool, 면접준비]
---

# 자바 String 완전 정리 🎯

자바를 배우면서 가장 많이 사용하게 되는 것이 바로 String이다. 그런데 String이 왜 특별한지, StringBuilder와는 어떤 차이가 있는지, String Pool은 무엇인지... 이번에는 자바의 String에 대해 차근차근 알아보자.

## String의 특별함 🎭

자바에서 String은 다른 클래스들과는 달리 매우 특별한 위치를 차지한다. 

- **가장 많이 사용되는 클래스**: 거의 모든 자바 프로그램에서 사용
- **불변 객체(Immutable)**: 한 번 생성되면 내용이 변경되지 않음
- **특별한 메모리 관리**: String Pool을 통한 메모리 최적화
- **연산자 오버로딩**: `+` 연산자로 문자열 연결 가능

이런 특별함 때문에 String을 제대로 이해하는 것이 중요하다.

## String vs StringBuilder 비교 ⚖️

### 1. 불변(immutable) vs 가변(mutable)

| 항목 | `String` | `StringBuilder` |
|------|----------|-----------------|
| **변경 가능 여부** | ❌ 불변 객체 (immutable) | ✅ 가변 객체 (mutable) |
| **예시** | `str += "a"` → 새로운 객체 생성 | `sb.append("a")` → 같은 객체에 추가 |
| **메모리 사용** | 매번 새 객체 생성 | 같은 객체 재사용 |

**String의 불변성:**
```java
String str = "Hello";
str += " World";  // 새로운 String 객체가 생성됨
// 원래 "Hello"는 가비지 컬렉션 대상이 됨
```

**StringBuilder의 가변성:**
```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // 같은 객체에 추가
// 메모리 효율적
```

### 2. 성능 차이 🚀

String은 문자열 변경이 많을 경우 **매번 새로운 객체가 생성**되므로 비효율적이다. 반면 StringBuilder는 문자열 추가/삭제/변경이 많은 작업에 훨씬 빠르다.

```java
// ❌ 비효율적인 방식 (String)
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // 매번 새로운 String 생성
}

// ✅ 효율적인 방식 (StringBuilder)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);  // 같은 객체에 추가
}
String result = sb.toString();
```

**성능 비교:**
- **String**: O(n²) - 루프마다 새 객체 생성
- **StringBuilder**: O(n) - 한 번에 처리

### 3. 스레드 안전성 🔒

| 클래스 | 스레드 안전성 | 사용 환경 |
|--------|---------------|-----------|
| `String` | ✅ (불변이므로 안전) | 모든 환경 |
| `StringBuilder` | ❌ (동기화 X) | 단일 스레드용 |
| `StringBuffer` | ✅ (동기화 O) | 멀티스레드용 |

```java
// 멀티스레드 환경에서는 StringBuffer 사용
StringBuffer sb = new StringBuffer();
// 동기화된 메서드들로 안전하게 사용 가능
```

### 4. 언제 무엇을 사용할까? 🤔

**String 사용 시기:**
- 문자열이 고정되어 있고 변경이 적은 경우
- 멀티스레드 환경에서 안전성이 중요한 경우
- 간단한 문자열 조작

**StringBuilder 사용 시기:**
- 루프 안에서 문자열을 반복적으로 수정하는 경우
- 성능이 중요한 경우
- 단일 스레드 환경

## String Pool의 비밀 🏊‍♂️

### String Pool이란?

자바는 같은 문자열 리터럴을 **여러 번 생성하지 않고**, **공유해서 사용하는 방식**을 채택했다. 이 공유 저장소가 바로 **String Constant Pool** (또는 intern pool)이다.

- **저장 위치**: Heap 메모리의 특별한 영역
- **목적**: 메모리 절약과 성능 향상
- **자동 관리**: 문자열 리터럴은 자동으로 Pool에 등록

### String Pool의 작동 방식

```java
String s1 = "hello";
String s2 = "hello";

System.out.println(s1 == s2); // true (같은 주소 참조)
System.out.println(s1.equals(s2)); // true (내용이 같음)
```

**동작 과정:**
1. `s1 = "hello"` → String Pool에 "hello" 저장
2. `s2 = "hello"` → Pool에서 기존 "hello" 참조
3. `s1`과 `s2`는 **같은 메모리 주소**를 가리킴

### new 키워드의 차이점

```java
String s1 = "hello";           // String Pool 사용
String s2 = new String("hello"); // 새로운 객체 생성

System.out.println(s1 == s2); // false (다른 주소)
System.out.println(s1.equals(s2)); // true (내용은 같음)
```

**new 키워드 사용 시:**
- String Pool에 "hello"가 있어도 **무시**
- Heap 메모리에 **새로운 객체** 생성
- 메모리 사용량 증가

### intern() 메서드의 활용

```java
String s1 = "hello";
String s2 = new String("hello").intern();

System.out.println(s1 == s2); // true (intern으로 Pool에 등록)
```

**intern() 메서드:**
- 문자열을 String Pool에 등록
- 이미 같은 값이 있으면 그걸 참조
- 메모리 절약 효과

## 실제 사용 예시 💻

### 1. 로그 메시지 생성

```java
// ❌ 비효율적
String log = "";
for (String item : items) {
    log += "[" + item + "] ";  // 매번 새 객체
}

// ✅ 효율적
StringBuilder sb = new StringBuilder();
for (String item : items) {
    sb.append("[").append(item).append("] ");
}
String log = sb.toString();
```

### 2. SQL 쿼리 동적 생성

```java
public String buildQuery(String table, List<String> columns) {
    StringBuilder query = new StringBuilder("SELECT ");
    
    for (int i = 0; i < columns.size(); i++) {
        query.append(columns.get(i));
        if (i < columns.size() - 1) {
            query.append(", ");
        }
    }
    
    query.append(" FROM ").append(table);
    return query.toString();
}
```

### 3. 문자열 비교 최적화

```java
// ❌ 비효율적
if (str.equals("CONSTANT")) { ... }

// ✅ 효율적 (String Pool 활용)
if ("CONSTANT".equals(str)) { ... }
// 또는
if (str == "CONSTANT") { ... }  // 리터럴끼리만 가능
```

## 주의사항 ⚠️

### 1. String Pool 메모리 누수

```java
// 위험: 대량의 문자열이 Pool에 쌓임
for (int i = 0; i < 1000000; i++) {
    String str = "prefix" + i;  // 각각 다른 문자열
}
```

### 2. StringBuilder 초기 용량 설정

```java
// ✅ 용량을 미리 설정하면 성능 향상
StringBuilder sb = new StringBuilder(1000);  // 예상 크기 설정
```

### 3. 문자열 비교 시 주의

```java
String s1 = "hello";
String s2 = new String("hello");

// ❌ 잘못된 비교
if (s1 == s2) { ... }  // false

// ✅ 올바른 비교
if (s1.equals(s2)) { ... }  // true
```

## 면접용 핵심 정리 🎯

### String의 특징
> "String은 불변 객체로, 한 번 생성되면 내용이 변경되지 않으며, String Pool을 통해 메모리를 최적화한다. 문자열 변경 시마다 새로운 객체가 생성되므로 성능상 비효율적일 수 있다."

### StringBuilder의 특징
> "StringBuilder는 가변 객체로, 내부 버퍼를 사용해 같은 객체에서 문자열을 수정할 수 있다. 문자열 변경이 많은 작업에서 String보다 훨씬 효율적이지만, 스레드 안전하지 않다."

### String Pool의 역할
> "String Pool은 Heap 메모리의 특별한 영역으로, 같은 문자열 리터럴을 공유하여 메모리를 절약하고 비교 성능을 향상시킨다. new 키워드로 생성한 문자열은 Pool을 사용하지 않는다."

### 사용 가이드라인
> "고정된 문자열이나 멀티스레드 환경에서는 String을, 반복적인 문자열 변경이나 성능이 중요한 경우에는 StringBuilder를 사용한다. String Pool을 활용하면 메모리와 성능을 최적화할 수 있다."

## 결론 🎉

자바의 String은 단순해 보이지만 내부적으로는 매우 정교한 메모리 관리와 최적화가 이루어지고 있다.

- **String**: 불변성과 안전성을 통한 안정성 
- **StringBuilder**: 가변성을 통한 성능 최적화 
- **String Pool**: 메모리 효율성을 위한 공유 메커니즘 🏊‍️

이런 기본 개념을 정확히 이해하고 있다면, 면접에서도 자신 있게 답변할 수 있을 것이다! 💪
