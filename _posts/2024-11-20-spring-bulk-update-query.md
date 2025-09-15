---
title: "벌크성 수정 쿼리"
date: 2024-11-20 10:00:00 +0900
categories: [Spring]
tags: [Spring, JPA, 벌크수정, Bulk Update, 성능최적화, 영속성컨텍스트]
---

# 벌크성 수정 쿼리

벌크성 수정 쿼리란? 

**if 우리 회사 직원의 연봉을 모두 일괄적으로 10%씩 올리고 싶다**

쿼리를 한번씩 날리면? 개사고

→ 그냥 DB에 update쿼리를 한방에 날리는 게 좋다

예를 들어 사용자 포인트를 일괄적으로 업데이트해야 하는 상황을 생각해보자. 만약 개별적으로 업데이트한다면 성능 이슈가 발생할 수 있다.

## 기본 JPA

![기본 JPA 벌크 수정](/assets/img/posts/2024-11-20-spring-bulk-update-query/bulk_1.png)

```java
// JPA에서 벌크 수정
String jpql = "update Member m set m.age = m.age + 1";
int resultCount = em.createQuery(jpql).executeUpdate();
```

## Spring Data JPA

![Spring Data JPA 벌크 수정](/assets/img/posts/2024-11-20-spring-bulk-update-query/bulk_2.png) 

```java
@Modifying
@Query("update Member m set m.age = m.age + 1")
int bulkAgePlus();
```

## 근데 위와 같은 코드는 위험할 수 있다

왜냐? 원래 Spring data JPA는 영속성 컨텍스트의 개념을 사용해서 엔티티를 조작하는데, 이건 그걸 무시하고 DB에 쿼리연산을 빵 때리는거

![영속성 컨텍스트 무시](/assets/img/posts/2024-11-20-spring-bulk-update-query/bulk_3.png)

그럼 위와 같은 코드를 짰을 때 결과가 어떻게 될까? 우리는 영속성 컨텍스트를 이용하길 바래 member5의 age가 41이 되길 바라지만 한 트랜잭션에서 저걸 조회하면 40이다. 근데 DB는 41로 기록되어 있다. 미친 짓이다

![DB와 영속성 컨텍스트 불일치](/assets/img/posts/2024-11-20-spring-bulk-update-query/bulk_4.png) 

그림과 같이 age가 40이라고 출력된다

![실제 결과](/assets/img/posts/2024-11-20-spring-bulk-update-query/bulk_5.png)

## 그럼 해결법은? em.flush() & em.clear()

![해결법](/assets/img/posts/2024-11-20-spring-bulk-update-query/bulk_6.png)

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1")
int bulkAgePlus();
```

이게 Spring data JPA에서는 `@Modifying(clearAutomatically = true)` 옵션을 준다 

→ 이게 영속성 컨텍스트 초기화

## 실제 사용 예시

### 1. 기본 벌크 수정

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Modifying
    @Query("update Member m set m.age = m.age + 1 where m.age >= :minAge")
    int bulkAgePlus(@Param("minAge") int minAge);
}
```

### 2. 영속성 컨텍스트 초기화

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
    
    @Modifying(clearAutomatically = true)
    @Query("update Member m set m.age = m.age + 1 where m.age >= :minAge")
    int bulkAgePlus(@Param("minAge") int minAge);
}
```

### 3. Service에서 사용

```java
@Service
@Transactional
public class MemberService {
    
    @Autowired
    private MemberRepository memberRepository;
    
    public int updateMemberAges(int minAge) {
        // 벌크 수정 실행
        int updatedCount = memberRepository.bulkAgePlus(minAge);
        
        // 영속성 컨텍스트 초기화 후 조회
        Member member = memberRepository.findById(1L).orElse(null);
        System.out.println("Updated age: " + member.getAge()); // 올바른 값 출력
        
        return updatedCount;
    }
}
```

## 다양한 벌크 수정 예시

### 1. 조건부 벌크 수정

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.status = :status where m.lastLoginDate < :date")
int updateInactiveMembers(@Param("status") String status, @Param("date") LocalDateTime date);
```

### 2. 복합 조건 벌크 수정

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.point = m.point + :point where m.grade = :grade and m.status = 'ACTIVE'")
int addPointByGrade(@Param("point") int point, @Param("grade") String grade);
```

### 3. 삭제 벌크 쿼리

```java
@Modifying(clearAutomatically = true)
@Query("delete from Member m where m.lastLoginDate < :date")
int deleteInactiveMembers(@Param("date") LocalDateTime date);
```

## 주의사항

### 1. @Transactional 필수

```java
@Service
@Transactional  // 필수!
public class MemberService {
    
    public int bulkUpdate() {
        return memberRepository.bulkAgePlus(20);
    }
}
```

### 2. 영속성 컨텍스트 초기화

```java
// ❌ 위험: 영속성 컨텍스트가 초기화되지 않음
@Modifying
@Query("update Member m set m.age = m.age + 1")
int bulkAgePlus();

// ✅ 안전: 영속성 컨텍스트 초기화
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1")
int bulkAgePlus();
```

### 3. 반환값 확인

```java
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.age >= :minAge")
int bulkAgePlus(@Param("minAge") int minAge);

// Service에서
public void updateMembers() {
    int updatedCount = memberRepository.bulkAgePlus(20);
    System.out.println("Updated " + updatedCount + " members");
}
```

## 성능 비교

### 개별 수정 vs 벌크 수정

```java
// ❌ 비효율적: 개별 수정
public void updateMembersIndividually(List<Member> members) {
    for (Member member : members) {
        member.setAge(member.getAge() + 1);
        memberRepository.save(member); // N번의 UPDATE 쿼리
    }
}

// ✅ 효율적: 벌크 수정
@Modifying(clearAutomatically = true)
@Query("update Member m set m.age = m.age + 1 where m.id in :ids")
int bulkUpdateMembers(@Param("ids") List<Long> ids);
```

### 성능 차이

| 방식 | 쿼리 수 | 실행 시간 | 메모리 사용량 |
|------|---------|-----------|---------------|
| 개별 수정 | N개 | 느림 | 높음 |
| 벌크 수정 | 1개 | 빠름 | 낮음 |

## 실제 프로젝트 적용 예시

### 1. 배치 작업

```java
@Service
public class BatchService {
    
    @Autowired
    private MemberRepository memberRepository;
    
    @Scheduled(cron = "0 0 2 * * ?") // 매일 새벽 2시
    public void dailyPointUpdate() {
        // 매일 포인트 10% 증가
        int updatedCount = memberRepository.bulkPointUpdate(0.1);
        log.info("Updated {} members' points", updatedCount);
    }
}
```

### 2. 관리자 기능

```java
@RestController
@RequestMapping("/admin")
public class AdminController {
    
    @Autowired
    private MemberService memberService;
    
    @PostMapping("/members/bulk-update")
    public ResponseEntity<String> bulkUpdateMembers(@RequestBody BulkUpdateRequest request) {
        int updatedCount = memberService.bulkUpdateMembers(request);
        return ResponseEntity.ok("Updated " + updatedCount + " members");
    }
}
```

## 결론

벌크성 수정 쿼리는 성능상 매우 유용하지만 주의해야 할 점들이 있다:

1. **@Transactional 필수**: 트랜잭션 없이는 실행되지 않음
2. **영속성 컨텍스트 초기화**: `clearAutomatically = true` 옵션 사용
3. **반환값 확인**: 실제로 몇 개의 레코드가 수정되었는지 확인
4. **성능 최적화**: 대량 데이터 처리 시 필수적인 방법

하지만 영속성 컨텍스트를 무시하고 DB에 직접 쿼리를 날리는 것이므로, 사용 후에는 반드시 영속성 컨텍스트를 초기화하거나 새로 조회해야 한다.

실무에서는 배치 작업이나 관리자 기능에서 자주 사용되며, 성능이 중요한 부분에서는 벌크 수정을 적극 활용하는 것이 좋다.

---

**관련 포스트:**
- [JPA와 영속성 컨텍스트(Chapter 1)](/posts/jpa-persistence-context/)
- [JPA와 영속성 컨텍스트(Chapter 2)](/posts/jpa-persistence-context-chapter2/)
