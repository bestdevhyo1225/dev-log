# 변경 감지(Dirty Checking)와 병합(Merge)

JPA에선 데이터를 변경하는 방법에 있어서 2가지 방법이 있다.

- **변경 감지 (Dirty Checking)**

- **병합 (Merge)**

<br>

## 들어가기에 앞서 영속성 컨텍스트는 Update를 어떻게 처리할까?

가장 먼저 알고 있어야 하는 부분은 `준영속`과 `영속`상태이다. 2가지는 다음과 같은 의미를 지니고 있다.

- `준영속 상태` : 영속성 컨텍스트가 더는 관리하지 않는 상태

- `영속 상태` : 영속성 컨텍스트가 관리하는 상태

우선 영속성 컨텍스트에 엔티티가 존재하면(`영속 상태`), 트랜잭션 커밋 시점에 값이 변경된 부분을 찾아서 `UPDATE` 쿼리를 수행한다. 그러나 영속성 컨텍스트에 엔티티가 존재하지 않으면(`준영속 상태`), 트랜잭션이 있더라도 영속성 컨텍스트가 관리하지 않기 때문에 변경할 근거가 없다.

<br>

## 변경 감지 (Dirty Checking)

변경 감지는 다음과 같은 코드에 의해서 동작한다.

```java
@Entity
public class Member {
  @Id
  @GeneratedValue(strategy = IDENTITY)
  private Long id;
  private String name;
  private String email;
  private String password;
  private String password;
  @Embedded
  private Adress address;

  public void change(String name, String email) {
    this.name = name;
    this.email = email;
  }
}

@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

  private final MemberRepository memberRepository;

  @Transactional // Transaction 커밋 시점에 변경된 부분을 찾고, EntityManger가 flush()를 통해 UPDATE 쿼리를 수행한다.
  public void update(UpdateMemberDto updateMemberDto) {
    // Member를 조회한다. -> 현재 영속성 컨텍스트에서 관리되는 상태
    Member findMember = this.memberRepository.findOne(updateMemberDto.getMemberId());
    // Member 엔티티의 값을 변경한다. -> change() 메소드는 Member 엔티티의 비즈니스 로직
    findMember.change(updateMemberDto.getName(), updateMemberDto.getEmail());
  }

}
```

<br>

## 병합 (Merge)

병합은 다음과 같은 과정을 거쳐 `UPDATE`를 수행한다.

```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class MemberService {

  private final MemberRepository memberRepository;

  @Transactional
  public void update(UpdateMemberDto updateMemberDto) {
    /*
      준영속 상태의 엔티티를 생성 -> 트랜잭션이 있더라도 영속성 컨텍스트에서 관리되지 않는 상태이기 때문에 변경된 상태가 아님
    */
    Member member = new Member();
    member.setId(updateMemberDto.getMemberId());
    member.setName(updateMemberDto.getName());
    member.setEmail(updateMemberDto.getEmail());

    // 준영속 상태의 엔티티를 영속상태로 변경한다.
    this.memberRepository.merge(member);
  }

}
```

#### 주의할 점

```java
Member mergedMember = this.memberRepository.merge(member);
```

`member` 변수가 `준영속상태`에서 `영속상태`로 변경되는 것이 아니고, `mergedMember` 변수가 영속성 컨텍스트에서 관리되는 상태이기 때문에 해당 엔티티를 가지고 무엇을 하려면, `mergedMember` 변수를 사용해야 한다.

<br>

## 병합의 내부 동작은 어떻게 처리될까?

병합의 내부 프로세스는 `변경 감지(Dirty Checking)과 똑같은 프로세스`로 동작하지만 조금 차이가 있다.

```java
// 병합 내부 프로세스
Member member = entityManger.find(Member.class, 1L);
// 모든 값을 변경해버린다...
member.setName(name)
member.setEmail(email)
member.setPassword(password)
member.setAddress(address)
```

병합은 엔티티의 모든 값을 변경하기 때문에 변경 감지와는 조금 차이가 있다.

<br>

## 둘 중에 어떤 방법으로 처리하는 것이 보다 나을까?

사람마다 기준이 다를것 같은데, '자바 ORM 표준 JPA 프로그래밍' 저자이신 김영한님께서는 `변경 감지(Dirty Checking)`방식으로 처리할 것을 권유했다. 이유는 다음과 같다.

- `변경 감지(Dirty Checking)`는 원하는 속성만 바꿀 수 있다.

- `병합(Merge)`은 모든 속성이 변경되기 때문에, 값이 없으면 일부 속성이 null로 업데이트 할 위험이 있다.

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
