# Equals, HashCode in Jpa Entity

이 글은 팀원 [June](https://github.com/JJONSOO)### 작성했습니다.

테스트 코드를 작성하던 도중, 두 엔티티가 같은지 비교하는 코드가 많아 Java에서 쓰이는 Equals, hahscode가 Jpa에서 어떻게 사용될 수 있는지 알아보았습니다.

```java
Member member = new Member("nickname", "email", "1234");
Member member2 = new Member("nickname", "email", "1234");

assertThat(member).isEqualTo(member2);
```

같은 필드를 지니고 있다고 해서 member == member2 일까요?

equals, hashcode를 override를 하지 않았다면 member, member2 각각의 인스턴스를 가지고 있기 때문에 위 테스트는 통과하지 않습니다.

Jpa환경에서도 알아보고 싶기 때문에 영속성 컨텍스트 안에서 관리될 때 확인해보겠습니다.

```java
@Autowired
MemberRepository memberRepository;

@Test
void EqualsAndHashCode() {
    memberRepository.save(new Member("nickname", "email", "1234"));
    Member member = memberRepository.findById(1L).get();
    Member member2 = memberRepository.findById(1L).get();
    em.clear();
    em.flush();

    assertThat(member).isEqualTo(member2);
}
```

위 코드를 보면 어떤 결과가 나올까요?

member, member2는 영속성 컨텍스트에서 관리되기 때문에 참조값으로 같은 인스턴스인지 확인이 가능합니다. 따라서 테스트를 통과합니다.

영속성 컨텍스트에서 관리되지 않는 member는 어떻게 될까요?

```java
@Test
void EqualsAndHashCode() {
    memberRepository.save(new Member("nickname", "email", "1234"));
    Member member = memberRepository.findById(1L).get();
    Member member2 = memberRepository.findById(1L).get();
		em.detach(member);

    assertThat(member).isEqualTo(member2);
}
```

위 테스트는 실패합니다. em.detach(member)로 member는 준영속 상태이기 때문에 member2와 다른 참조를 가지고 있습니다. `member.getNickname() == member2.getNickname()` 는 True입니다.

### equals(), hashcode()

위 문제를 해결하는 방법은 3가지가 있습니다.

1. PK로만 equals, hashCode 구현
2. PK를 제외하고 equla로 구현
3. 비지니스 키를 사용한 동등성 구현

#### 1. PK로만 equls, hashCode 구현하기

```java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (id == null) return false;
    if (!(o instanceof Member))
        return false;

    final Member user = (Member)o;

    return id.equals(Member.getId());
}

@Override
public int hashCode() {
    int result = id != null ? id.hashCode() : 0;
    return result;
}
```

위 코드는 Object가 Member인지 확인 후, pk인 Id를 비교하여 동등성을 비교하는 코드입니다.

이 방법은 repository.save() 즉 PK가 지정된 후에만 비교할 수 있는 방법입니다.

즉, 객체의 동등성 비교를 영속, 준영속 상태에서만 비교하는게 가능한 방법입니다.

#### 2. PK를 제외하고 equals로 구현

PK를 제외한 모든 Field를 비교하는 방법입니다.

하지만, Entity 내부에 join 과 같은 관계가 있다면, 다른 테이블과 연관관계가 있기 때문에 포함해서는 안됩니다.

```java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (!(o instanceof Member))
        return false;

    Member member = (Member)o;

    if (!Objects.equals(nickname, member.nickname))
        return false;
    if (!Objects.equals(email, member.email))
				return false;
		return Objects.equals(password, member.password);
}

@Override
public int hashCode() {
    int result = nickname != null ? nickname.hashCode() : 0;
    result = 31 * result + (email != null ? email.hashCode() : 0);
    result = 31 * result + (password != null ? password.hashCode() : 0);
    return result;
}
```

위 문제는 영속성 Context 의 instance 를 수정(만약 비밀번호를 변경한다면) 다른 영속성 Context 의 instance 와는 더 이상 일치하지 않습니다.

#### 3. 비지니스 키를 이용한 동등성 구현

PK를 제외하고 Unique를 제약 조건을 가진 email, 또는 변경 불가능한 속성 등을 사용하면 위의 방법들의 단점을 극복할 수 있습니다.

Jpa 환경에서 equals(), hashcode()로 동등성 비교에 대해 알아봤습니다.
