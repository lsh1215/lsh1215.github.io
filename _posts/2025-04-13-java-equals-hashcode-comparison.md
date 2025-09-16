---
title: "자바 equals와 hashCode 완전 정리 [CS 면접 박살내기]"
date: 2025-04-13 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, equals, hashCode, 객체비교, HashSet, HashMap, 면접준비]
---

# 자바 equals와 hashCode 완전 정리 🎯

자바에서 객체를 비교할 때 가장 헷갈리는 부분이 바로 `==`와 `equals()`의 차이점이다. 그런데 `equals()`만 알면 되는 건가? `hashCode()`는 왜 필요한가? HashSet이 왜 O(1)의 시간복잡도를 가질 수 있는가? 이번에는 자바의 객체 비교에 대해 차근차근 알아보자.

## == vs equals() 비교 ⚖️

### 1. == 연산자

`==` 연산자는 **참조 비교(Reference Comparison)**를 수행한다. 즉, 두 변수가 **같은 메모리 주소**를 가리키는지 확인한다.

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);         // false (다른 객체)
System.out.println(a.equals(b));    // true  (문자열 내용이 같음)
```

### 2. equals() 메서드

`equals()` 메서드는 **논리적 동등성(Logical Equality)**을 비교한다. 즉, 두 객체의 **내용이 같은지** 확인한다.

```java
// String은 equals()가 오버라이딩되어 있음
String str1 = "hello";
String str2 = "hello";
System.out.println(str1.equals(str2)); // true (내용이 같음)
```

### 3. 직접 만든 클래스의 경우

```java
class Person {
    String name;
    
    Person(String name) {
        this.name = name;
    }
}

Person p1 = new Person("Alice");
Person p2 = new Person("Alice");

System.out.println(p1 == p2);         // false (다른 객체)
System.out.println(p1.equals(p2));    // false (equals() 오버라이딩 안 함)
```

**문제점**: `equals()`를 오버라이딩하지 않으면 `Object.equals()`를 사용하는데, 이는 `==`와 동일하게 주소만 비교한다.

## equals() 오버라이딩 📝

### 올바른 equals() 구현

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;                    // 같은 객체인지 확인
    if (obj == null || getClass() != obj.getClass()) return false; // null 체크, 타입 체크
    Person other = (Person) obj;
    return Objects.equals(name, other.name);         // 실제 값 비교
}
```

### equals() 구현 시 주의사항

1. **반사성(Reflexive)**: `x.equals(x)`는 항상 true
2. **대칭성(Symmetric)**: `x.equals(y)`가 true면 `y.equals(x)`도 true
3. **추이성(Transitive)**: `x.equals(y)`, `y.equals(z)`가 true면 `x.equals(z)`도 true
4. **일관성(Consistent)**: 여러 번 호출해도 같은 결과
5. **null 안전성**: `x.equals(null)`은 항상 false

## hashCode()의 중요성 🔢

### hashCode()란?

`hashCode()`는 객체의 **해시값을 반환**하는 메서드로, **Hash 기반 자료구조**에서 사용된다.

```java
@Override
public int hashCode() {
    return Objects.hash(name);
}
```

### 왜 hashCode()가 필요한가?

**HashSet의 동작 원리:**
1. 객체를 저장할 때 `hashCode()`로 **버킷 위치** 결정
2. 같은 버킷에 여러 객체가 있으면 `equals()`로 **실제 비교**
3. `hashCode()`가 다르면 `equals()` 호출 없이 **다른 객체로 판단**

```java
Set<Person> set = new HashSet<>();
set.add(new Person("Alice"));
set.contains(new Person("Alice")); // hashCode() 없으면 false!
```

## equals()와 hashCode()의 관계 🔗

### 핵심 규칙

**`equals()`가 true인 두 객체는 반드시 `hashCode()`도 같아야 한다!**

```java
if (a.equals(b)) {
    // 반드시 a.hashCode() == b.hashCode() 여야 함
}
```

**하지만 반대는 보장되지 않음:**
```java
if (a.hashCode() == b.hashCode()) {
    // equals()가 true일 수도 있고 아닐 수도 있음 (해시 충돌)
}
```

### 왜 이 규칙이 중요한가?

```java
// ❌ 잘못된 예시
class BadPerson {
    String name;
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        BadPerson other = (BadPerson) obj;
        return Objects.equals(name, other.name);
    }
    
    // hashCode() 오버라이딩 안 함!
}

Set<BadPerson> set = new HashSet<>();
set.add(new BadPerson("Alice"));
set.contains(new BadPerson("Alice")); // false! (hashCode 다름)
```

## HashSet의 O(1) 시간복잡도 비밀 🚀

### 일반적인 배열 검색 vs HashSet

```java
// ❌ 배열에서 검색: O(n)
List<Person> list = new ArrayList<>();
// 1000개 중 "Alice" 찾기 → 최대 1000번 비교

// ✅ HashSet에서 검색: O(1)
Set<Person> set = new HashSet<>();
// 1000개 중 "Alice" 찾기 → 1번의 hashCode() + 1번의 equals()
```

### HashSet의 내부 동작

1. **해시 계산**: `hashCode()`로 버킷 위치 결정
2. **버킷 접근**: O(1) 시간에 해당 버킷으로 이동
3. **객체 비교**: 같은 버킷 내에서만 `equals()` 호출

**시간복잡도:**
- **평균**: O(1) - 해시 충돌이 적을 때
- **최악**: O(n) - 모든 객체가 같은 버킷에 있을 때

## 실전 활용: MSA에서의 VO 패턴 🏗️

### Value Object란?

VO(Value Object)는 **값 자체를 나타내는 객체**로, **불변성**과 **값 기반 동등성**이 핵심이다.

```java
@Value
@EqualsAndHashCode
public class Money {
    private final BigDecimal amount;
    private final String currency;
    
    public Money(BigDecimal amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

### MSA에서 VO 사용 시나리오

```java
// 주문 서비스에서 금액 비교
public class OrderService {
    
    public boolean isExpensiveOrder(Order order) {
        Money threshold = new Money(new BigDecimal("1000"), "USD");
        Money orderAmount = new Money(order.getAmount(), "USD");
        
        // equals()로 값 비교
        return orderAmount.compareTo(threshold) > 0;
    }
    
    // Set에 VO 저장
    public Set<Money> getUniqueAmounts(List<Order> orders) {
        Set<Money> uniqueAmounts = new HashSet<>();
        for (Order order : orders) {
            uniqueAmounts.add(new Money(order.getAmount(), "USD"));
        }
        return uniqueAmounts; // 중복 제거됨
    }
}
```

### Lombok을 활용한 자동 생성

```java
@Value
@EqualsAndHashCode
public class UserId {
    private final String value;
}

@Value
@EqualsAndHashCode
public class Email {
    private final String value;
}
```

## 실제 구현 예시 💻

### 완전한 equals()와 hashCode() 구현

```java
public class Person {
    private final String name;
    private final int age;
    private final String email;
    
    public Person(String name, int age, String email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        
        Person person = (Person) obj;
        return age == person.age &&
               Objects.equals(name, person.name) &&
               Objects.equals(email, person.email);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age, email);
    }
}
```


## 주의사항 ⚠️

### 불변 객체로 만들기

```java
// ❌ 위험: 필드가 변경되면 hashCode()도 변경됨
public class MutablePerson {
    private String name; // 변경 가능
    
    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

// ✅ 안전: 불변 객체
public class ImmutablePerson {
    private final String name; // final로 불변 보장
    
    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```


## 면접용 핵심 정리 🎯

### == vs equals()
> "==는 참조 비교로 메모리 주소를 비교하고, equals()는 논리적 동등성으로 객체의 내용을 비교한다. String은 equals()가 오버라이딩되어 있지만, 직접 만든 클래스는 반드시 equals()를 오버라이딩해야 한다."

### hashCode()의 역할
> "hashCode()는 Hash 기반 자료구조에서 객체의 위치를 결정하는 데 사용되며, equals()가 true인 두 객체는 반드시 같은 hashCode()를 가져야 한다. 이를 통해 HashSet이 O(1)의 시간복잡도를 달성할 수 있다."

### equals()와 hashCode() 규칙
> "equals()를 오버라이딩하면 반드시 hashCode()도 함께 오버라이딩해야 하며, 두 메서드 모두 일관된 규칙을 따라야 한다. 이를 위반하면 HashSet, HashMap 등의 자료구조에서 예상과 다른 동작을 할 수 있다."

### MSA에서의 활용
> "MSA에서 VO(Value Object) 패턴을 사용할 때는 값 기반 동등성이 중요하므로, equals()와 hashCode()를 적절히 구현해야 한다. 이를 통해 도메인 객체의 값 비교와 중복 제거를 정확하게 수행할 수 있다."

## 결론 🎉

자바의 객체 비교는 단순해 보이지만 내부적으로는 매우 정교한 메커니즘이 작동하고 있다.

- **==**: 참조 비교로 메모리 주소 확인 
- **equals()**: 논리적 동등성으로 내용 비교 
- **hashCode()**: Hash 자료구조에서의 효율적 검색 
- **VO 패턴**: MSA에서의 값 기반 객체 관리 🏗

이런 기본 개념을 정확히 이해하고 있다면, 면접에서도 자신 있게 답변할 수 있을 것이다! 💪
