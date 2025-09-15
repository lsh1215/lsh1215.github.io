---
title: "JPQL로 DTO 조회하기"
date: 2024-11-15 10:00:00 +0900
categories: [Spring]
tags: [Spring, JPA, JPQL, DTO, Query, 성능최적화, 데이터조회]
---

# JPQL로 DTO 조회하기

이전까지 나는 대부분의 조회로직에서 다음과 같은 방식을 사용했다:

1. 일단 파라미터로 전달받은 필드값으로 해당 객체를 조회한다
2. Builder나 생성자로 DTO로 반환한다.

근데 이게 여간 귀찮은게 아니다. 매번 Controller에서 DTO 반환 로직을 작성해줘야 하기 때문에 간단해보이지만 코드가 늘어나고 보기가 껄끄러워진다.

그럼 그냥 DTO로 조회하고 반환되게 할 수는 없나? 그럼 간단한 로직에서는 요긴하게 써먹을텐데

**JPQL에서는 이 기능을 제공한다.**

하지만 실제로는 잘 사용하지 않는 방법이기도 하다. 왜냐하면 복잡한 조회 로직에서는 오히려 가독성이 떨어지고, 단순한 조회에서만 편리하기 때문이다. 그래도 알아두면 유용한 경우가 있으니 한번 정리해보자.

## 기존 방식의 문제점

### 1. Entity 조회 후 DTO 변환

```java
@Query("select m from Member m where m.username = :username")
List<Member> findByUsername(String username);

// Controller에서 변환
@GetMapping("/members")
public List<MemberDto> getMembers(@RequestParam String username) {
    List<Member> members = memberRepository.findByUsername(username);
    
    return members.stream()
            .map(member -> MemberDto.builder()
                    .id(member.getId())
                    .username(member.getUsername())
                    .teamName(member.getTeam().getName())
                    .build())
            .collect(Collectors.toList());
}
```

### 2. 문제점들

- **코드 중복**: 매번 동일한 변환 로직 반복
- **성능 이슈**: 불필요한 Entity 조회로 인한 메모리 사용량 증가
- **유지보수성**: DTO 구조 변경 시 여러 곳을 수정해야 함
- **가독성**: 비즈니스 로직과 변환 로직이 섞여 복잡해짐

이런 문제들이 있어서 좀 더 간단하게 할 수 있는 방법이 없나 고민하다가 JPQL DTO 조회 방식을 알게 되었다.

## JPQL DTO 조회 방식

### 1. 기본 사용법

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
       "from Member m join m.team t")
List<MemberDto> findMemberDto();
```

### 2. DTO 클래스 준비

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class MemberDto {
    private Long id;
    private String username;
    private String teamName;
    
    // JPQL에서 사용할 생성자
    public MemberDto(Long id, String username, String teamName) {
        this.id = id;
        this.username = username;
        this.teamName = teamName;
    }
}
```

### 3. 다양한 조회 예시

#### 단일 조건 조회
```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
       "from Member m join m.team t where m.username = :username")
List<MemberDto> findMemberDtoByUsername(@Param("username") String username);
```

#### 페이징 처리
```java
@Query(value = "select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
               "from Member m join m.team t",
       countQuery = "select count(m) from Member m")
Page<MemberDto> findMemberDtoWithPaging(Pageable pageable);
```

#### 조건부 조회
```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, t.name) " +
       "from Member m join m.team t " +
       "where m.age >= :minAge and t.name = :teamName")
List<MemberDto> findMemberDtoByAgeAndTeam(@Param("minAge") int minAge, 
                                         @Param("teamName") String teamName);
```

이런 식으로 사용할 수 있다. 하지만 쿼리가 길어지면 가독성이 떨어지고 관리하기 어려워진다.

## 고급 활용법

### 1. 서브쿼리 활용

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, " +
       "(select count(o) from Order o where o.member = m)) " +
       "from Member m")
List<MemberDto> findMemberWithOrderCount();
```

### 2. CASE WHEN 활용

```java
@Query("select new study.datajpa.dto.MemberDto(m.id, m.username, " +
       "case when m.age >= 18 then '성인' else '미성년자' end) " +
       "from Member m")
List<MemberDto> findMemberWithAgeGroup();
```

### 3. 집계 함수 활용

```java
@Query("select new study.datajpa.dto.TeamStatisticsDto(t.name, count(m), avg(m.age)) " +
       "from Team t left join t.members m group by t.name")
List<TeamStatisticsDto> findTeamStatistics();
```


## 주의사항

### 1. 패키지명 완전 경로 사용

```java
// ❌ 잘못된 예시
@Query("select new MemberDto(m.id, m.username) from Member m")

// ✅ 올바른 예시
@Query("select new com.example.dto.MemberDto(m.id, m.username) from Member m")
```

### 2. 생성자 파라미터 순서

```java
// DTO 생성자
public MemberDto(Long id, String username, String teamName) { ... }

// JPQL에서 순서 맞춰서 사용
@Query("select new com.example.dto.MemberDto(m.id, m.username, t.name) " +
       "from Member m join m.team t")
```

### 3. NULL 처리

```java
@Query("select new com.example.dto.MemberDto(m.id, m.username, " +
       "coalesce(t.name, '팀없음')) from Member m left join m.team t")
List<MemberDto> findMemberDtoWithNullHandling();
```

## 실제 프로젝트 적용 예시

### Controller 개선

```java
@RestController
@RequestMapping("/api/members")
public class MemberController {
    
    @Autowired
    private MemberRepository memberRepository;
    
    // 기존 방식
    @GetMapping("/old")
    public List<MemberDto> getMembersOld(@RequestParam String teamName) {
        List<Member> members = memberRepository.findByTeamName(teamName);
        return members.stream()
                .map(this::convertToDto)
                .collect(Collectors.toList());
    }
    
    // JPQL DTO 조회 방식
    @GetMapping("/new")
    public List<MemberDto> getMembersNew(@RequestParam String teamName) {
        return memberRepository.findMemberDtoByTeamName(teamName);
    }
    
    private MemberDto convertToDto(Member member) {
        return MemberDto.builder()
                .id(member.getId())
                .username(member.getUsername())
                .teamName(member.getTeam().getName())
                .build();
    }
}
```

## 결론

JPQL DTO 조회 방식을 사용하면:

1. **코드 간소화**: 변환 로직 제거로 코드가 깔끔해짐
2. **성능 향상**: 필요한 필드만 조회하여 메모리 사용량 감소
3. **유지보수성**: DTO 구조 변경 시 Repository만 수정하면 됨
4. **가독성**: 비즈니스 로직에 집중할 수 있음

하지만 실제로는 잘 사용하지 않는 방법이다. 왜냐하면:

- 복잡한 조회 로직에서는 오히려 가독성이 떨어짐
- JPQL 쿼리가 길어지면 관리하기 어려워짐
- 단순한 조회에서만 편리함

그래도 간단한 조회 로직에서는 요긴하게 써먹을 수 있으니 알아두면 좋다. 특히 성능이 중요한 부분이나 단순한 데이터 조회가 필요한 경우에는 유용할 것 같다.
