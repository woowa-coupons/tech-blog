# N+1 문제를 어떻게 해결할 수 있을까?

> 이 글은 팀원 [브루니](https://github.com/23Yong) 작성했습니다.

프로젝트를 진행하며 다음과 같은 문제를 해결해야했습니다.

> **`쿠폰그룹` 조회시 `쿠폰그룹` 내의 `쿠폰` 들의 이름과 `쿠폰그룹`들의 \[잔여 개수 / 총개수] 를 계산하고 페이징 처리하여 `쿠폰그룹` 목록을 반환**

이번 포스팅에서는 이 문제를 해결하는 과정을 포스팅해보려 합니다.

### 테이블간의 관계

`쿠폰그룹` 과 `쿠폰` 은 다음과 같은 연관관계를 가지고 있습니다.

![image](https://github.com/woowa-coupons/tech-blog/assets/66981851/c8f64cad-692d-497d-b6ac-ca9c073dfc90)

위의 그림에서 볼 수 있듯이 `쿠폰그룹` 과 `쿠폰`은 **일대다 연관관계**를 가지고 있습니다. 하나의 `쿠폰그룹` 조회시 여러개의 `쿠폰` 을 보여줘야 했는데요. JPA를 사용하고 있는 저희 프로젝트에서 간단하게 떠올릴 수 있는 방법은 다음과 같았습니다.

* OneToMany 양방향 연관관계를 사용해 하나의 쿠폰그룹 조회시 해당 쿠폰그룹의 쿠폰들을 조회

### OneToMany 양방향 연관관계를 이용?

OneToMany 양방향 연관관계를 이용하면 다음과 같이 엔티티를 구성해야 했습니다.

```java
@Entity
public class Coupon {
  // ...

  @JoinColumn(name = "coupon_group_id")
  @ManyToOne(fetch = FetchType.LAZY)
  private CouponGroup couponGroup;

  // ...
}
```

```java
public class CouponGroup {
  // ...
  
  @OneToMany(mappedBy = "couponGroup")
  private List<Coupon> coupons;
}
```

이후 서비스 로직에서 아래와 같이 쿠폰그룹내의 쿠폰들을 조회할 수 있었습니다.

```java
Page<CouponGroup> couponGroups = couponGroupRepository.findAll(pageRequest);

Map<CouponGroup, List<Coupon>> couponGroupCouponMap = couponGroups.getContent().stream()
        .collect(Collectors.toMap(couponGroup -> couponGroup, CouponGroup::getCoupons));
```

하지만 막상 코드를 실행시켜보면 아래와 같은 쿼리가 날라가는 것을 확인할 수 있습니다.

```sql
[Hibernate] 
    select
        coupongrou0_.id as id1_2_,
        coupongrou0_.finished_at as finished2_2_,
        coupongrou0_.promotion_id as promotio5_2_,
        coupongrou0_.started_at as started_3_2_,
        coupongrou0_.title as title4_2_ 
    from
        coupon_group coupongrou0_ limit ? // 쿠폰그룹을 전체 조회하는 쿼리
[Hibernate] 
    select
        // 생략
    from
        coupon coupons0_ 
    where
        coupons0_.coupon_group_id=? // 특정 쿠폰그룹의 쿠폰 목록을 조회하는 쿼리 1
[Hibernate] 
    select
        // 생략
    from
        coupon coupons0_ 
    where
        coupons0_.coupon_group_id=? // 특정 쿠폰그룹의 쿠폰 목록을 조회하는 쿼리 2
[Hibernate] 
    select
        // 생략
    from
        coupon coupons0_ 
    where
        coupons0_.coupon_group_id=? // 특정 쿠폰그룹의 쿠폰 목록을 조회하는 쿼리 3
```

쿼리의 결과를 보면 `쿠폰그룹`을 전체 조회하는 쿼리 1번, 각각의 쿠폰그룹에 대하여 `쿠폰` 목록이 조회되는 쿼리가 N번 날라가는 것을 확인할 수 있습니다. 이는 쿠폰그룹의 개수가 많아질수록 성능이 떨어지기 때문에 개선해야합니다.

### Fetch Join

N + 1 문제를 해결하는 다양한 방법 중 먼저 FETCH JOIN을 사용해 문제를 해결해보고자 했습니다.

* OneToMany 관계에서 조인을 하게 되면 카테시안 곱이 발생해 의도와는 다른 결과를 얻을 수 있기 때문에 `DISTINCT` 키워드를 붙여 중복을 제거합니다.
* 페이징을 수행하기 위해서는 `@Query` 에 `countQuery` attribute도 넣어주어야 하기 때문에 아래와 같이 코드를 작성했습니다.

```java
@Query(value = "SELECT DISTINCT couponGroup FROM CouponGroup couponGroup JOIN FETCH couponGroup.coupons",
            countQuery = "SELECT COUNT(DISTINCT couponGroup) FROM CouponGroup couponGroup INNER JOIN couponGroup.coupons")
Page<CouponGroup> findAll(Pageable pageable);
```

> **왜 countQuery를 넣어야 하는가?🧐** JPA에서 `@Query`를 사용하지 않고 페이징을 수행하게 되면 의도하지 않았던 countQuery가 날라가는 것을 확인할 수 있는데요, 이는 `Page` 인터페이스의 `getTotalElements()`의 메서드의 수행 결과를 반환해야 하기 때문이 아닌가 예상하고 있습니다. 따라서 `@Query`를 사용해 직접 쿼리를 날려 페이징을 수행하기 위해서는 별도의 countQuery가 필요합니다.

결과를 보면 다음과 같은 쿼리가 날라가고 있는 것을 확인할 수 있습니다.

```sql
select
        distinct coupongrou0_.id as id1_2_0_,
        coupons1_.id as id1_1_1_,
        coupongrou0_.admin_nickname as admin_ni2_2_0_,
        coupongrou0_.finished_at as finished3_2_0_,
        coupongrou0_.promotion_id as promotio6_2_0_,
        coupongrou0_.started_at as started_4_2_0_,
        coupongrou0_.title as title5_2_0_,
        coupons1_.created_at as created_2_1_1_,
        coupons1_.coupon_group_id as coupon_g8_1_1_,
        coupons1_.discount as discount3_1_1_,
        coupons1_.initial_quantity as initial_4_1_1_,
        coupons1_.remain_quantity as remain_q5_1_1_,
        coupons1_.title as title6_1_1_,
        coupons1_.type as type7_1_1_,
        coupons1_.coupon_group_id as coupon_g8_1_0__,
        coupons1_.id as id1_1_0__ 
    from
        coupon_group coupongrou0_ 
    inner join
        coupon coupons1_ 
            on coupongrou0_.id=coupons1_.coupon_group_id
```

하나의 쿼리로 조인을 해오는 것을 확인할 수 있었습니다. 그런데 이상한 점은 `LIMIT`이 안보인다는 점인데요. 로그를 보니 아래와 같은 메시지를 보내고 있었습니다.

```sql
WARN firstResult/maxResults specified with collection fetch; applying in memory!
```

JPA를 사용할 때 OneToMany 관계에서 FETCH JOIN과 Pagination을 함께 사용하면 쿼리의 결과를 모두 메모리에 적재한 뒤 메모리에서 페이징을 수행하기 때문입니다. 이와 같은 방법은 OOM(Out Of Memory)문제를 발생시킬 수 있기 때문에 좋지 않은 방법입니다.

### default\_batch\_fetch\_size 를 통한 문제 해결

JPA의 XXXToMany 관계에서 FETCH JOIN과 페이징을 함께 사용할 수는 없습니다. 그렇기 때문에 FETCH JOIN을 제거하고 N+1 문제를 해결하기 위해 `default_batch_fetch_size` 옵션을 통해 문제를 해결했습니다.

`default_batch_fetch_size` 옵션을 활용하면 Lazy Loading으로 인해 발생되는 N개의 쿼리를 설정한 옵션만큼 모아 IN 절로 처리하게 됩니다. 예를들어 100개의 추가 쿼리가 발생한다고 했을 때 `default_batch_fetch_size` 옵션을 10으로설정하게 되면 발생하는 추가 쿼리의 수를 100 / 10으로 줄일 수 있게 됩니다.

실제로 발생하는 쿼리를 확인해볼까요?

```sql
[Hibernate] 
    select
        coupongrou0_.id as id1_2_,
        coupongrou0_.admin_nickname as admin_ni2_2_,
        coupongrou0_.finished_at as finished3_2_,
        coupongrou0_.promotion_id as promotio6_2_,
        coupongrou0_.started_at as started_4_2_,
        coupongrou0_.title as title5_2_ 
    from
        coupon_group coupongrou0_ 
    order by
        coupongrou0_.id desc limit ?
[Hibernate] 
    select
        count(coupongrou0_.id) as col_0_0_ 
    from
        coupon_group coupongrou0_
[Hibernate] 
    select
        coupons0_.coupon_group_id as coupon_g8_1_1_,
        coupons0_.id as id1_1_1_,
        coupons0_.id as id1_1_0_,
        coupons0_.created_at as created_2_1_0_,
        coupons0_.coupon_group_id as coupon_g8_1_0_,
        coupons0_.discount as discount3_1_0_,
        coupons0_.initial_quantity as initial_4_1_0_,
        coupons0_.remain_quantity as remain_q5_1_0_,
        coupons0_.title as title6_1_0_,
        coupons0_.type as type7_1_0_ 
    from
        coupon coupons0_ 
    where
        coupons0_.coupon_group_id in (
            ?, ?, ?, ?, ?, ?, ?, ?, ?, ?
        )
```

마지막 쿼리를 확인해보면 `IN` 절을 통해 데이터를 한 번에 가져오는 것을 확인할 수 있었습니다.

물론 이 방법에도 문제가 있습니다. 쿠폰그룹의 수가 굉장히 많다면 N/(옵션으로 설정한 값)번의 쿼리가 추가적으로 발생하는 것은 어쩔 수 없습니다. 하지만 저희 프로젝트에서는 페이징을 통해 한 페이지당 조회하는 쿠폰그룹의 수가 제한되어 있기 때문에 이 방법을 통해 N+1 문제를 해결할 수 있었습니다.
