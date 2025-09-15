---
title: "Page 사용과 count 분리 그리고 Slice"
date: 2024-11-18 10:00:00 +0900
categories: [Spring]
tags: [Spring, JPA, Pagination, Page, Slice, 성능최적화, 데이터조회]
---

# Page 사용과 count 분리 그리고 Slice

백엔드에서 Pagination을 구현하겠다고 해보자. 그러면 `org.springframework.data.domain.Pageable` 기능을 활용하여 Repository 함수의 파라미터로 전달할 것이다.

먼저 PageRequest 객체를 만들고 해당 객체를 전달하게 되고 Repository에서는 함수 파라미터에서 Pageable이라는 type으로 선언했을 것이다.

```java
// Service
PageRequest pageRequest = PageRequest.of(0, 3, Sort.by(Sort.Direction.DESC, "username"));

~~~ = memberRepository.findByAge(10, pageRequest);

// Repository
~~~ findByUsername(String name, Pageable pageable);
```

## 반환 타입과 차이

그런데 이때 해당 함수의 반환타입은 크게 세가지를 선언할 수 있다.

- **Page<T>**
  
![Page 구조](/assets/img/posts/2024-11-18-spring-pagination-page-slice/Page_1.png)
  
- **Slice<T>**

![Slice 구조](/assets/img/posts/2024-11-18-spring-pagination-page-slice/Page_2.png)
  
- **List<T>**
  
  리스트는 Collection이니 알 것이라고 생각하고 생략

이렇게 위와 같이 Page, Slice, List로 선언이 가능한데 이때 Page에서 이 두가지 type과는 다른 점이 두드러지게 나타난다. 바로 `getTotalPages()`와 `getTotalElements()`;

즉, 전체 페이지 수와 전체 데이터 수 정보를 갖고 있다는 것이고 이는 다른말로 count(SQL)를 통해 이 정보들을 셌다는 것이다. 아는 사람들은 알겠지만, 전체 count 쿼리는 매우 무겁기 때문에 성능 저하를 유발할 수 있어 실무에서는 잘 쓰이지 않는다.

## countQuery를 분리하지 않은 경우

### 코드

```java
@Query("select m from Member m")
Page<Member> findMemberAll(Pageable pageable);
```

### 실행 시 SQL 로그

페이징 요청 시, JPA는 데이터를 가져오는 쿼리와 함께 전체 레코드 수를 계산하는 쿼리를 자동으로 생성함

```sql
-- 데이터 가져오는 쿼리
select m.id, m.username, m.email from Member m limit ?, ?

-- 전체 레코드 수를 계산하는 쿼리
select count(m.id) from Member m
```

**특징**:

- JPA가 기본적으로 `select` 쿼리를 분석하여, `count()` 쿼리를 자동으로 생성함
- 자동 생성된 `count()` 쿼리는 전체 컬럼을 포함하는 복잡한 서브쿼리나 조인을 포함할 수 있음
- 불필요한 조인이나 조건이 포함될 가능성이 있어 성능에 영향을 줄 수 있음

## countQuery를 분리한 경우

### 코드

```java
@Query(value = "select m from Member m",
       countQuery = "select count(m.username) from Member m")
Page<Member> findMemberAllCountBy(Pageable pageable);
```

### 실행 시 SQL 로그

페이징 요청 시, `countQuery`에 명시된 쿼리가 실행

```sql
-- 데이터 가져오는 쿼리
select m.id, m.username, m.email from Member m limit ?, ?

-- 전체 레코드 수를 계산하는 쿼리
select count(m.username) from Member m
```

**특징**:

- `countQuery`를 직접 지정했기 때문에, JPA가 쿼리를 자동 생성하지 않고 지정된 쿼리를 사용함
- `countQuery`는 필요한 컬럼만 사용하는 단순한 쿼리를 작성할 수 있어 성능이 최적화됨
- 불필요한 조인이나 조건이 제거되어 쿼리 실행 시간이 단축될 수 있음

## 그럼 해결 방법은?

### 1. 프론트엔드에서 전체 페이지 정보를 원한다? → 그럼 어쩔 수 없다 Page 써야지

대신 효율적으로!

```java
@Query(value = "select m from Member m",
        countQuery = "select count(m.username) from Member m")
Page<Member> findMemberAllCountBy(Pageable pageable); 
```

요렇게 count 쿼리를 분리하면 된다.

### 2. 프론트에서 전체 페이지 정보가 필요 없다고 한다? → 그러면 바로 Slice

Slice는 추가로 limit + 1을 조회한다. 

그래서 다음 페이지 여부 확인이 가능 하기 때문에 많은 경우에 List보다 낫다

```java
@Query("select m from Member m")
Slice<Member> findMemberAllSlice(Pageable pageable);
```

## 실제 사용 예시

### Controller에서의 활용

```java
@RestController
@RequestMapping("/api/members")
public class MemberController {
    
    @Autowired
    private MemberRepository memberRepository;
    
    // Page 사용 - 전체 페이지 정보 필요
    @GetMapping("/page")
    public Page<Member> getMembersWithPage(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Pageable pageable = PageRequest.of(page, size);
        return memberRepository.findMemberAllCountBy(pageable);
    }
    
    // Slice 사용 - 다음 페이지 여부만 필요
    @GetMapping("/slice")
    public Slice<Member> getMembersWithSlice(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        Pageable pageable = PageRequest.of(page, size);
        return memberRepository.findMemberAllSlice(pageable);
    }
}
```

### Service에서의 활용

```java
@Service
public class MemberService {
    
    @Autowired
    private MemberRepository memberRepository;
    
    public PageResponse<Member> getMembers(PageRequest pageRequest) {
        Pageable pageable = PageRequest.of(
            pageRequest.getPage(), 
            pageRequest.getSize(),
            Sort.by(Sort.Direction.DESC, "createdAt")
        );
        
        Page<Member> memberPage = memberRepository.findMemberAllCountBy(pageable);
        
        return PageResponse.of(memberPage);
    }
    
    public SliceResponse<Member> getMembersSlice(PageRequest pageRequest) {
        Pageable pageable = PageRequest.of(
            pageRequest.getPage(), 
            pageRequest.getSize(),
            Sort.by(Sort.Direction.DESC, "createdAt")
        );
        
        Slice<Member> memberSlice = memberRepository.findMemberAllSlice(pageable);
        
        return SliceResponse.of(memberSlice);
    }
}
```

## 성능 비교

### Page vs Slice 성능 차이

| 구분 | Page | Slice |
|------|------|-------|
| 쿼리 수 | 2개 (데이터 + count) | 1개 (데이터만) |
| 메모리 사용량 | 높음 | 낮음 |
| 응답 속도 | 느림 | 빠름 |
| 전체 페이지 수 | 제공 | 제공 안함 |
| 다음 페이지 여부 | 제공 | 제공 |

### 언제 어떤 것을 사용할까?

**Page 사용 시기:**
- 전체 페이지 수가 필요한 경우
- 페이지네이션 UI에서 "1, 2, 3, ... 마지막 페이지" 형태로 표시해야 하는 경우
- 데이터가 많지 않고 성능이 크게 중요하지 않은 경우

**Slice 사용 시기:**
- 무한 스크롤이나 "더 보기" 버튼 형태의 UI
- 성능이 중요한 경우
- 전체 페이지 수가 필요 없는 경우
- 대용량 데이터를 다루는 경우

## 주의사항

### 1. countQuery 최적화

```java
// ❌ 비효율적
@Query(value = "select m from Member m join m.team t where t.name = :teamName",
       countQuery = "select count(m) from Member m join m.team t where t.name = :teamName")

// ✅ 효율적
@Query(value = "select m from Member m join m.team t where t.name = :teamName",
       countQuery = "select count(m.id) from Member m join m.team t where t.name = :teamName")
```

### 2. Slice의 limit + 1 동작

```java
// 요청: size = 10
// 실제 조회: limit 11 (10 + 1)
// 반환: 10개 데이터 + hasNext() = true/false
```

### 3. 정렬 최적화

```java
// 인덱스가 있는 컬럼으로 정렬
Pageable pageable = PageRequest.of(0, 10, Sort.by("id")); // ✅ 좋음

// 인덱스가 없는 컬럼으로 정렬
Pageable pageable = PageRequest.of(0, 10, Sort.by("username")); // ❌ 느림
```

## 결론

Page와 Slice는 각각의 용도가 다르다:

- **Page**: 전체 페이지 정보가 필요한 경우, countQuery 분리로 성능 최적화
- **Slice**: 성능이 중요하고 다음 페이지 여부만 필요한 경우

실무에서는 대부분 Slice를 사용하는 경우가 많다. 왜냐하면 성능이 중요하고, 대부분의 UI에서는 전체 페이지 수보다는 "더 보기" 기능이 더 유용하기 때문이다.

하지만 관리자 페이지나 데이터 분석 페이지처럼 전체 페이지 수가 필요한 경우에는 Page를 사용하되, countQuery를 분리해서 성능을 최적화하는 것이 좋다.
