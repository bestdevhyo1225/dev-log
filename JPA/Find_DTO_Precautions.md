# JPQL, Querydsl로 DTO 직접 조회시, 주의할 점

- Join을 사용했을때, 신경써야할 부분임 (공부했는데 다시 복습하고자 정리함)

<br>

## Join을 사용하고, DTO를 직접 조회를 구현할 때는 Fetch Join을 사용하면 안된다.

> 다음과 같이 조회용 목적인 `QueryRepository`가 있었다.

```kotlin
@Repository
class MemberQueryRepositoryImpl(private val jpaQueryFactory: JPAQueryFactory) : MemberQueryRepository {

    override fun paginationCoveringIndex(pageNo: Long, pageSize: Long): List<FindMemberDto> {
        val memberIds: List<Long> = jpaQueryFactory
                .select(member.id)
                .from(member)
                .orderBy(member.id.desc())
                .limit(pageSize)
                .offset(pageNo * pageSize)
                .fetch()

        if (CollectionUtils.isEmpty(memberIds)) return ArrayList()

        return jpaQueryFactory
                .select(
                        Projections.constructor(
                                FindMemberDto::class.java,
                                member.id, member.username, member.email, member.team.name
                        )
                )
                .from(member)
                .join(member.team).fetchJoin() // 처음에 여기다 .fetchJoin()을 추가했었음
                .where(member.id.`in`(memberIds))
                .orderBy(member.id.desc())
                .fetch()
    }

}
```

> 위와 같은 코드를 작성하고, 직접 실행을 해보면 아래와 같은 에러가 발생한다.

![스크린샷 2020-10-29 오후 3 29 06](https://user-images.githubusercontent.com/23515771/97533415-8132ae00-19fb-11eb-88b9-f7438d0f64de.png)

> 이를 해결하려면? `fetchJoin()`을 제거하고 사용해야한다.

```kotlin
@Repository
class MemberQueryRepositoryImpl(private val jpaQueryFactory: JPAQueryFactory) : MemberQueryRepository {

    override fun paginationCoveringIndex(pageNo: Long, pageSize: Long): List<FindMemberDto> {
        val memberIds: List<Long> = jpaQueryFactory
                .select(member.id)
                .from(member)
                .orderBy(member.id.desc())
                .limit(pageSize)
                .offset(pageNo * pageSize)
                .fetch()

        if (CollectionUtils.isEmpty(memberIds)) return ArrayList()

        return jpaQueryFactory
                .select(
                        Projections.constructor(
                                FindMemberDto::class.java,
                                member.id, member.username, member.email, member.team.name
                        )
                )
                .from(member)
                .join(member.team) //.fetchJoin() 제거
                .where(member.id.`in`(memberIds))
                .orderBy(member.id.desc())
                .fetch()
    }

}
```

> `fetchJoin()`을 제거하면, 아래와 같이 Select 쿼리가 올바르게 실행되며

![스크린샷 2020-10-29 오후 3 32 34](https://user-images.githubusercontent.com/23515771/97533670-f9996f00-19fb-11eb-8d4f-bee4240c19de.png)

> 결과 값도 올바르게 나온다.

<img width="1080" alt="스크린샷 2020-10-29 오후 3 35 57" src="https://user-images.githubusercontent.com/23515771/97533908-71679980-19fc-11eb-9344-9d45fc5389b1.png">
