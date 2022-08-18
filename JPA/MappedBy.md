# MappedBy

`mappedBy` 는 주로 `@OneToMany(mappedBy = ...)` 형식으로 사용하며, 양방향 연관관계를 가지면서 연관관계 주인이 아닌곳을 설정할 때 사용된다.

- 연관관계 주인이 아닌곳에 지정
- `외래 키` 를 관리(등록, 수정, 삭제) 할 수 없으며, 오직 `읽기` 만 가능하다.

즉, `mappedBy` 옵션을 사용하면 `연관관계 주인이 아닌 곳에서는 외래키에 대해 읽기만 가능하다.`

## 테스트

> Issue 클래스

```kotlin
@Entity
@Table
class Issue(
    // 생략
) {

    // 생략

    @OneToMany(mappedBy = "issue", fetch = FetchType.LAZY)
    val comments: MutableList<Comment> = mutableListOf()
}
```

> Comment 클래스

```kotlin
@Entity
@Table
class Comment(
    // 생략
) {

    // 생략

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "issue_id")
    var issue: Issue? = null
        protected set
}
```

위의 `Issue` 와 `Comment` 엔티티가 있으며, `1 : N` 의 연관 관계를 맺고 있다. 그리고 `Issue` 엔티티의 `@OneToMany(mappedBy = issue)` 와 같은 설정이 되어 있다.
만약에 아래와 같은 코드로 `Issue` 에서 `Comment` 를 삭제하려고 하면, 정상적으로 동작하지 않는다.

```kotlin
val issue: Issue = issueRepository.findById(issueId)
val comment: Comment = commentRepository.findById(commentId)

// issue 엔티티의 comments 컬렉션을 대상으로 remove(comment) 수행해도 Delete 쿼리는 실행되지 않는다. 
issue.comments.remove(comment)
```

삭제 쿼리가 실행되지 않는 이유는 맨 위에 설명한 것 처럼 `mappedBy 옵션이 설정되면, 외래키를 관리(등록, 수정, 삭제)할 수 없다. 오직 읽기만 가능하다.`
따라서 `issue.comments.remove(comment)` 를 통해 삭제 쿼리가 실행되려면, `mappedBy` 옵션을 제거해야 한다. 나의 경우에는 JPA 책에서 권장하는
방식인 `연관 관계 주인인 곳에서 외래키를 관리` 하도록 하는 방식을 선호하기 때문에 `mappedBy` 를 필수 옵션으로 적용한다. 그리고 `Comment` 삭제에 대한 쿼리를 하나 만들어서 이를 직접 실행하는
방식으로 문제를 풀어나간다.
