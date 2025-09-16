---
title: "JPA와 영속성 컨텍스트(Chapter 1)"
date: 2024-09-28 10:00:00 +0900
categories: [Spring]
tags: [Spring, JPA, Persistence Context, Entity Manager, Hackathon]
---

# JPA와 영속성 컨텍스트(Chapter 1)

## Techeer Good Night Hackathon

이전에 테커에서 Good Night Hackathon을 진행했다.(꽤 됐지만 블로그는 지금 작성...)
필요한 요구 조건에 맞춰 개발을 진행하고 현직자분께 코드 리뷰를 받을 수 있는 기회가 있어 참여하게 되었다.

![해커톤 개요](/assets/img/posts/2024-09-28-jpa-persistence-context/jpa_1.png)

[Good Night 3rd Hackathon Backend](https://github.com/techeer-sv/Good-Night-3rd-Hackathon-Backend)

지난 소프트 삭제 피드백과 함께 서비스 로직 중 데이터 업데이트는 하는 로직에서 상태 변경이 save() 함수 없이도 업데이트가 일어난다는 피드백을 받았다. 하지만 JPA에 대한 이해도가 떨어지다보니 더티 체킹이 뭔지 왜 save()가 없어도 되는 지를 이해하지 못했고, 공부한 내용을 바탕으로 얘기해보고자 한다.

## JPA와 영속성 컨텍스트

JPA를 공부하면서 반드시 알아가야할 것이 크게 두가지 있다.

1. 객체와 데이터베이스 테이블 매핑과 설계
2. 영속성 컨텍스트에 대한 이해

사실 객체와 데이터베이스 테이블 매핑과 설계를 하는 건 다른 프레임워크에서 ORM을 사용해봤다면 이해하기 어렵지 않은 파트다. 그리고 개념 자체도 크게 어렵지 않기 때문에, 데이터베이스 설계를 할 줄 안다면 어려움 없이 공부할 수 있다.

그러나 영속성 컨텍스트는 말 자체도 익숙하지 않고, 개념 자체도 아예 처음인 사람들이 많을 것이라고 생각한다. 평소에 Spring에서 트랜잭션에 대한 고민을 해보지 않았다면 크게 필요없다고 느낄 수 도 있다. 하지만 알지 못한다면, 나처럼 CRUD를 짜면서도 바보같이 짜게 될 것이다.

## 영속성 컨텍스트를 배우기 전에 기본적으로 알아야 할 개념

- 엔티티 매니저 팩토리 & 엔티티 매니저
- 트랜잭션

### 엔티티 매니저 팩토리와 엔티티 매니저

JPA 시작과 Persistence 클래스 사용에 대해서는 김영한님의 스프링 JPA 강의를 참고

이제 코드를 통해 엔티티 매니저 팩토리 생성과 엔티티 매니저 생성을 알아보자.

```java
/*META-INF/persistence.xml에서 이름이 "hello"인 영속성 유닛을 찾아서 엔티티 매니저 팩토리로 생성*/
EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
EntityManager em = emf.createEntityManager();
```

엔티티 매니저 팩토리를 JPA를 동작시키기 위한 기반 객체라고 생각하면 된다. 다음과 같이 Persistence 클래스의 함수로 만들 수 있는데, 요놈이 (JPA 구현체에 따라) 데이터베이스 커넥션 풀도 생성해서 생성 비용이 큰 녀석이다. 따라서 애플리케이션 전체에서 한 번만 생성하고 공유해서 사용하자.

엔티티 팩토리를 생성했다면 이를 이용해 매니저를 생성할 수 있다. JPA 기능으 대부분을 해주는 녀석으로 등록/수정/삭제/조회와 같은 기능을 해준다. 엔티티 매니저는 내부에 데이터 소스(커넥션)을 유지하면서 데이터베이스와 통신하는데, 좀 쉽게 생각하자면 가상의 데이터베이스 정도가 될 것이다.
이때 반드시 알아야 할 것은, 데이터베이스 커넥션과 밀접한 관계가 있어 쓰레드 간에 공유하거나 재사용해서는 안된다.

### 트랜잭션 관리

```java
EntityTransaction tx = em.getTransaction();
tx.begin();

/* try catch 문이 없을 경우 실행이 제대로 작동되지 않았을 때
이후에 트랜잭션이나 엔티티매니저와 관련된 로직이
정상처리 되지 않을 수 있기 때문에 try catch 문으로 예외처리
*/
try {
    Member member = new Member();
    member.setId(100L);
    member.setName("John");
    
    // 젖장
    em.persist(member);
    
    // 조회
    Member member2 = em.find(Member.class, 100L);
    member2.setName("Jane");

    tx.commit();
} catch (Exception e){
// 예외의 경우 transaction을 롤백한다
    tx.rollback();
}
```

데이터베이스를 공부했다면 트랜잭션에 대해 대충이라도 알고 있을 것이다. JPA에서는 tx.begin()과 commit() or rollback()으로 트랜잭션 구간이 정해지는데, 당연히 요 안에서 데이터 변경을 해야 DB에 내용이 잘 저장된다.

## 영속성 관리와 영속성 컨텍스트

### 엔티티 매니저 팩토리와 엔티티 매니저 복습

우선 영속성 컨텍스트에 대해 배우기 전에 이전에 배운 내용을 조금 복습하면서 이미지로 구체화하여 알아보자.

![영속성 컨텍스트 개념](/assets/img/posts/2024-09-28-jpa-persistence-context/jpa_2.png)

요렇게 엔티티 매니저가 Client의 요청에 엔티티 매니저를 생성하는데, 그림처럼 여러 요청에 따라 여러 엔티티 매니저가 생성된 것을 볼 수 있다. 이때 요청1에 대해서는 커넥션이 이뤄지지 않았지만 요청2에서는 커넥션이 이뤄졌는데, 이는 요청1이 요청2와는 달리 데이터베이스 연결이 꼭 필요한 시점에 도달하지 않아서다. 즉, 데이터 베이스 연결이 꼭 필요한 시점(보통 트랜잭션 시작)일 때 커넥션을 획득하고 이 커넥션 엔티티 매니저 팩토리를 생성할 때 만든 커넥션 풀에서 커넥션을 연결하여 사용한다.

### 영속성 컨텍스트란?

영속하다(Persist) = 영원히 계속하다(continue to exist)
출처 : Naver Korean-English Dictionary

자 그럼 이제 진짜 JPA를 이해하는데 가장 중요한 용어라고 불리는 그 녀석 영속성 컨텍스트를 알아보자. 김영한님은 "엔티티를 영구 저장하는 환경" 이라고 말씀하시는데, 엔티티 매니저가 데이터를 저장하거나 조회할 때 바로 이 영속성 컨텍스트에 보관하고 관리한다.

```java
em.persist(member);
```

이 코드는 회원 엔티티를 저장하는 코드라고 배웠다. 그럼 em.persist를 쓰면 DB에 값이 저장될까?
다음은 tx.commit()을 주석처리 했을 때 콘솔에 sql 쿼리가 날라가는 지 실험한 사진이다.

![영속성 컨텍스트 실습](/assets/img/posts/2024-09-28-jpa-persistence-context/jpa_3.png)

그림처럼 tx.commit() 없이 찍어보면 sql 쿼리가 날라가지 않는다.
왜 그럴까? 엔티티 영구 저장 환경으로 영속성 컨텍스트를 쓴다고 했는데, 이 영속성 컨텍스트가 실제 DB가 아니라 엔티티 매니저가 엔티티를 저장하고 관리하는 가상 데이터베이스 역할을 하기 때문이다.
(내가 이후 단계를 공부하고 느낀 건 영속성 컨텍스트가 가상 DB고 em이 DBMS의 manager 느낌)

## 그렇다면, 영속성 컨텍스트는 어떻게 활용될까?

### em.persist(entity)

entity 저장이라고 했던 그 녀석, 하지만 commit없이는 쿼리가 안 날라가던 그 녀석!
이 녀석의 진짜 의미는 데이터를 영속성 컨텍스트에 저장한다는 것이다.
즉, 엔티티를 영속화한다는 의미 DB에 저장하는 것 아니라는 것이다.

다음 그림처럼 em.persist()를 했을 때, 영속성 컨텍스트의 1차 캐시에 id와 entity를 key value로 저장한다. 또 SQL 저장소에 INSERT 쿼리문도 저장한다. 이러한 방식으로 직접 DB에 값을 저장하는 것이 아니라, 가상의 DB에 저장하고 트랜잭션 발생할 때 DB에 쿼리를 날리고 캐시에 저장된 값을 저장하는 것이다.

![영속성 컨텍스트 내부 구조](/assets/img/posts/2024-09-28-jpa-persistence-context/jpa_4.png)

## 다음으로...

그렇다면, 영속성 컨텍스트를 왜 사용할까? 이건 다음 챕터에서 알아보도록 하겠다.
