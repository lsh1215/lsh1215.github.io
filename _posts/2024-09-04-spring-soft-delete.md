---
title: "Spring 좋은 소프트 삭제 구현하기"
date: 2024-09-04 10:00:00 +0900
categories: [Spring]
tags: [Spring, JPA, Soft Delete, Hackathon]
---

# Spring 좋은 소프트 삭제 구현하기

## Techeer Good Night Hackathon

이번에 테커에서 Good Night Hackathon을 진행했다.
필요한 요구 조건에 맞춰 개발을 진행하고 현직자분께 코드 리뷰를 받을 수 있는 기회가 있어 참여하게 되었다.

![해커톤 개요](/assets/img/posts/2024-09-04-spring-soft-delete/hackathon.png)

[Good Night 3rd Hackathon Backend](https://github.com/techeer-sv/Good-Night-3rd-Hackathon-Backend)

피드백 중 소프트 삭제와 관련된 피드백을 받았고, 이에 대한 내용을 공유하고자 한다.
[소프트 삭제 피드백 PR](https://github.com/techeer-sv/Good-Night-3rd-Hackathon-Backend/pull/10)

## 내가 구현한 소프트 삭제(기존 코드)

나는 다음과 같이 @SQLDelete와 @Where를 사용해서 소프트 삭제를 구현했다.

```java
@NoArgsConstructor
@Entity @Getter
// 소프트 삭제 구현 delete 메소드 사용 시 실제 삭제가 아닌 소프트 삭제로 구동
@SQLDelete(sql = "UPDATE comment SET deleted_at = NOW() WHERE id = ?")
// 매 조회 시 deleted at이 null인 값만
@Where(clause = "deleted_at IS NULL")
public class Comment extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "wish_id", nullable = false)
    private Wish wish;

    private String content;

    public Comment(final String content, final Wish wish) {
        this.content = content;
        this.wish = wish;
    }

}
```

이 방법이 구글링을 했을 때 가장 많이 나오는 방식으로, Spring JPA Soft Delete라는 블로그들에 들어가보면 이 방식을 가장 많이 사용하고 있다. 가장 큰 특징은 Entity에 코드 두 줄을 쓰는 것 만으로도 소프트 삭제와 조회 시 소프트 삭제된 데이터를 제외하고 조회하는 것이 기본 코드(delete나 조회 쿼리)로 쉽게 이뤄진다는 것이다.

## 피드백

![피드백 내용](/assets/img/posts/2024-09-04-spring-soft-delete/feedback.png)

하지만 현직자분으로부터 다음과 같은 피드백을 받았다. 요약하자면, 소프트 삭제를 따로 구현해야 필요 시 하드 삭제와 삭제된 데이터 조회를 수행할 수 있고 코드를 파악하고 데이터를 구분하기에 더 용이하는 피드백이였다.

해당 부분에 대해 실무에서 많이 사용하는 방법에 대해 궁금해져서 좀 더 찾아보다가 인프런 강의 질문에 대한 김영한님의 답변을 볼 수 있었다.

[김영한님의 BaseEntity와 SoftDelete 질문](https://www.inflearn.com/community/questions/304378/baseentity%EC%99%80-softdelete-%EC%A7%88%EB%AC%B8)

## 내가 생각하기에 좋은 방법

BaseEntity에서 다음과 같은 소프트 삭제 함수를 정의하고 사용한다.

```java
public void softDelete() {
    this.deletedAt = LocalDateTime.now();
}
```

이후에 Service 로직에서 소프트 삭제가 되는 부분을 이 함수를 써서 이뤄지도록 수정하고 조회 시에도 쿼리 메서드를 통해서 구현한다.

### Repository Code Example

```java
@Repository
public interface WishRepository extends JpaRepository<Wish, Long> {
    Optional<Wish> findByIdAndDeletedAtIsNull(Long wishId);
}
```

### Service Code Example

```java
public void updateWish(WishUpdateRequest request, Long wishId) {
    Wish wish = wishRepository.findByIdAndDeletedAtIsNull(wishId)
        .orElseThrow(() -> new EntityNotFoundException("Wish not found"));

    wish.changeStatus(request.getStatus()); // 제약 없이 상태 변경
}
```

내가 생각하기에는 이 방법이 가장 구현하고 사용하기 간편하고 좋은 방법인 것 같다.

물론 @sql은 사용하지 않더라도, Entity에서 @Where를 사용하여 조회 시 소프트 삭제된 컬럼들은 제외하고 조회하고, 이후에 소프트 삭제된 데이터까지 조회하고 싶을 때 JPQL을 사용하는 방식도 좋다고 생각한다. 간편하기도 하고, 헤커톤과 같은 토이 프로젝트나 작은 규모의 프로젝트에서는 소프트 삭제된 데이터를 조회하고 관리하는 것이 크게 중요하지 않기 때문이다.

하지만, 토이 프로젝트와는 달리 실제 프로젝트에서는 서비스 규모가 크고 작업 인원도 많다. 또 소프트 삭제된 데이터를 조회하고 관리해야할 일도 있을 것이다. 이럴 때는 간편하게 코드를 짜는 것도 중요하지만, 이 코드를 짜지 않은 사람도 코드를 보고 역할을 명확하게 파악할 수 있어야 한다. 또, 삭제된 데이터라도 하더라고 데이터를 조회하고 관리해야할 일이 있을 것이다.

이러한 이유로, Spring 소프트 삭제를 검색하면 나오는 많은 블로그 글과는 달리 나는 소프트 삭제를 따로 구현하고 쿼리 메서드로 조회하는 방식이 더 좋은 방법이라고 생각한다.
