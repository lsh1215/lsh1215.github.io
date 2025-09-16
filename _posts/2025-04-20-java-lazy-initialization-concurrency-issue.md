---
title: "자바 지연로딩과 동기화 문제 [CS 면접 박살내기]"
date: 2025-04-20 10:00:00 +0900
categories: [CS 면접 박살내기]
tags: [Java, 지연로딩, 동기화, 멀티스레드, Double-Checked Locking, 면접준비]
---

# 자바 지연로딩과 동기화 문제 🎯

자바에서 멀티스레드 환경에서 객체를 초기화할 때 가장 까다로운 부분이 바로 지연로딩(Lazy Initialization)이다. 심지어는 Effective Java에서도 잘못된 예시가 있었다 ㅋㅋㅋ 이번에는 지연로딩에서 발생할 수 있는 동기화 문제와 해결 방법에 대해 차근차근 알아보자.

## Effective Java의 잘못된 예시 📚

### 문제가 된 코드

Effective Java 3판의 Item 83에서 다음과 같은 코드가 제시되었다:

```java
// Double-check idiom for lazy initialization of instance fields
private FieldType getField() {
    FieldType result = field; // (1)
    if (result == null) { // First check (no locking)
        synchronized (this) {
            if (field == null) // Second check (with locking)
                field = result = computeFieldValue(); // (2)
        }
    }
    return result;
}
```

이 코드는 **Double-Checked Locking** 패턴을 사용한 지연로딩 예시로 보이지만, 실제로는 **버그가 있는 잘못된 코드**였다.

### 이슈 제기와 수정

[issue #8](https://github.com/jbloch/effective-java-3e-source-code/issues/8)

2019년 8월, 한 개발자가 이 코드의 문제점을 지적했다

![Issue](/assets/img/posts/2025-04-20-java-lazy-initialization-concurrency-issue/lazy_1.png)


Joshua Bloch는 이 문제를 인정하고 다음과 같이 답변했다

![Comment](/assets/img/posts/2025-04-20-java-lazy-initialization-concurrency-issue/lazy_2.png)


## 문제 분석 🔍

### 왜 이 코드가 잘못되었나?

이 코드의 문제점을 단계별로 분석해보자.

**시나리오:**
1. 스레드 A와 스레드 B가 거의 동시에 `getField()` 메서드를 호출
2. 두 스레드 모두 `field`가 `null`인 상태에서 시작

**실행 과정:**
```java
// 스레드 A와 B가 동시에 실행
FieldType result = field; // (1) - 둘 다 result = null
if (result == null) { // 둘 다 true
    synchronized (this) {
        if (field == null) // 둘 다 true
            field = result = computeFieldValue(); // (2) - A가 먼저 실행
    }
}
return result; // B는 여전히 null을 반환!
```

**문제점:**
- 스레드 A가 `field`를 초기화했지만, 스레드 B의 `result` 변수는 여전히 `null`
- 스레드 B는 `synchronized` 블록에서 `field`가 이미 초기화된 것을 확인하지만, `result`는 업데이트되지 않음
- 결과적으로 스레드 B는 `null`을 반환하게 됨

## 해결 방법 1: 명시적 재할당 🔧

```java
private FieldType getField() {
    FieldType result = field;
    if (result == null) {
        synchronized (this) {
            if (field == null)
                field = result = computeFieldValue();
            else
                result = field; // ✅ 중요: 이미 초기화된 경우 재할당
        }
    }
    return result;
}
```

**개선점:**
- `field`가 이미 초기화된 경우 `result`에 다시 할당
- `null` 반환 문제 해결

## 해결 방법 2: 더 효율적인 방식 🚀

```java
private FieldType getField() {
    FieldType result = field;
    if (result != null) return result; // ✅ 빠른 반환
    
    synchronized (this) {
        result = field; // 다시 확인
        return result != null ? result : (field = computeFieldValue());
    }
}
```

**장점:**
- **성능 최적화**: 대부분의 경우 동기화 없이 빠르게 반환
- **가독성**: 로직이 더 명확함
- **안전성**: 동기화 문제 해결


## 실무에서의 주의사항 ⚠️

### 1. 성능 고려사항

```java
// ❌ 비효율적: 매번 동기화
private synchronized FieldType getField() {
    if (field == null) {
        field = computeFieldValue();
    }
    return field;
}

// ✅ 효율적: double-checked locking
private FieldType getField() {
    FieldType result = field;
    if (result != null) return result;
    
    synchronized (this) {
        result = field;
        return result != null ? result : (field = computeFieldValue());
    }
}
```

### 2. 메모리 가시성

```java
private volatile FieldType field; // volatile 필수!
```

**volatile이 필요한 이유:**
- 멀티스레드 환경에서 `field` 값의 변경사항이 다른 스레드에 즉시 반영되도록 보장
- CPU 캐시와 메인 메모리 간의 동기화

### 3. 초기화 순서

```java
// ❌ 위험: 순환 참조 가능
private FieldType field = computeFieldValue();

// ✅ 안전: 지연 초기화
private FieldType getField() {
    // 필요할 때만 초기화
}
```

## 면접용 핵심 정리 🎯

### 지연로딩이란?
> "지연로딩은 객체나 리소스를 실제로 필요할 때까지 초기화를 미루는 기법으로, 메모리 사용량을 줄이고 애플리케이션 시작 시간을 단축할 수 있다."

### Double-Checked Locking의 문제점
> "Double-Checked Locking 패턴에서 로컬 변수에 값을 복사한 후, 다른 스레드가 해당 필드를 초기화해도 로컬 변수는 업데이트되지 않아 null을 반환할 수 있다. 이를 해결하려면 synchronized 블록 내에서 필드 값을 다시 확인하고 로컬 변수를 재할당해야 한다."

### 올바른 구현 방법
> "지연로딩을 구현할 때는 volatile 키워드로 메모리 가시성을 보장하고, double-checked locking 패턴을 사용할 때는 synchronized 블록 내에서 필드 값을 다시 확인한 후 반환해야 한다. 성능을 고려한다면 초기화된 경우 빠른 반환을 위해 early return을 사용하는 것이 좋다."

### 멀티스레드 환경에서의 주의사항
> "멀티스레드 환경에서 지연로딩을 구현할 때는 동기화, 메모리 가시성, 성능을 모두 고려해야 한다. 

## 결론 🎉

지연로딩은 성능 최적화에 매우 유용한 기법이지만, 멀티스레드 환경에서는 매우 조심스럽게 구현해야 한다.

- **Effective Java도 실수**: 심지어 Joshua Bloch도 실수할 수 있다는 것을 보여줌
- **동기화의 중요성**: 멀티스레드 환경에서는 세심한 동기화가 필요
- **테스트의 중요성**: 복잡한 동기화 코드는 반드시 테스트해야 함
- **성능과 안전성의 균형**: 빠른 성능과 안전한 동기화 사이의 균형이 중요

이런 기본 개념을 정확히 이해하고 있다면, 면접에서도 자신 있게 답변할 수 있을 것이다! 💪
