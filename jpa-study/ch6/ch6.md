## 다양한 연관관계 매핑

연관관계에는 `다대일, 일대다, 일대일, 다대다`가 있다. 사실 테이블 관점에서는 다대일, 일대다는 같은 의미이다. 왜냐하면 `다`에 해당하는 테이블이 외래키를 관리하기 때문이다!

또한, 단방향과 양방향이 존재하는데 사실 테이블을 항상 양방향이고 객체는 항상 단방향이다. 객체 관점에서 단방향을 반대로 다시 설정해주어서 양방향처럼 서로 참조할 수 있도록 하는 것이다!

### 일대다

다대일은 지금까지 배운 내용으로 충분히 이해가 간다. JPA에서는 일대다 관계가 있지만 사실 다대일의 반대라고 기본적으로 이해하면된다. 그러나 일대다에서는 `일`에 해당하는 객체가 외래키를 관리한다. 여기서부터 특이한 현상?이 발생한다.

테이블의 외래키는 보통 `다`에 해당하는 곳에서 관리가 되기 때문에 객체상에서 `일`에서 외래키를 관리해도 실제 테이블에는 `다`테이블의 외리키 컬럼과 매핑이 된다. 여기서 단점이 생긴다. 객체의 연관관계랑 테이블의 연관관계랑 살짝 맞지않게 된다. 따라서 책에서는 다대일로 `다`객체가 외래키를 관리하도록 권장한다.

### 일대다 양방향

일대다도 양방향을 설정할 수 있는데 다대일 연관관계랑 다르다. 관계형 데이터베이스에서는 위에서 설명한 것처럼 `다`에서 외래키를 관리한다. 따라서 `@ManyToOne`만 주인이 될 수 있고 `mapperdBy`속성이 없다. 그렇지만 굳이 만들자면 또 만들수는 있다고 한다. 바로 읽기 전용 매핑을 하나 더 만들어서 양방향으로 만드는 방법이다.

```java
@Entity
public class Team {

    @Id @Column(name="team_id")
    private Long id;
    
    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID") // member 테이블의 존재하는 컬럼
    private List<Member> members = new ArrayList<Member>();

    ...
}

@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;
    
    private String name;
    
    @ManyToOne 
    @JoinColumn(name = "TEAM_ID", insetalbe = false, updatable = false)
    private Team team;
    
    ...
}
```

두 엔티티가 같은 키를 관리하면 문제가 되기 때문에 한쪽에는 읽기 전용으로 만들어줘야 한다. 하지만 이것도 역시 굳이..라는 느낌이라 책에서도 추천하지 않는다. 그냥 다대일 양방향 쓰자.

### 일대일

테이블 관계에스는 일대일 관계에서 둘다 외래 키를 가질 수 있다. 따라서 둘 중에 누가 외래 키를 가질지 선택을 해야한다.

**주 테이블에 외래키**  
주 객체가 대상 객체를 찾조하는 것처럼 주 테이블에 외래키를 두는 것이다. 따라서 객체 지향 개발자들이 선호하는 방식이고 JPA도 주 테이블에 외래 키가 있으면 좀 더 편리하게 매핑할 수 있다.

`단방향`

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne 
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
    ...
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;
    
    private String name;
    
    ...
}
```

`@OneToOne`을 사용하여 외래 키에 유니크 제약 조건을 추가했다.  
-> 자동으로 유니크 제약 조건이 추가되나..?찾아보거나 테스트 진행을 해봐야 할듯

<br>

`양방향`

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne 
    @JoinColumn(name = "LOCKER_ID")
    private Locker locker;
    
    ...
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "locker")
    private Member member;
    ...
}
```

양방향으로 설정하였고 `Member.locker`가 연관관계의 주인이다. 즉 주 객체와 테이블이 외래키를 관리한다

**대상 테이블에 외래키**  
전통적인 데이터 베이스 개발자들이 선호하는 방식으로 대상 테이블에 외래 키를 둔다. 대상테이블이 `다`로 변경이될 때 유여한게 대처할 수 있다는 장점이 있다. 그러나 대상 테이블에 외래 키가 있는 단방향은 JPA가 지원하지 않는다. 방향을 수정하거나 양방향을 만들어서 연관관계의 주인으로 설정해야하낟.

`양방향`

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne(mappedBy = "member")
    private Locker locker;
    
    ...
}

@Entity
public class Locker {
    @Id @GeneratedValue
    @Column(name="LOCKER_ID")
    private Long id;
    
    private String name;
    
    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;
    ...
}
```

### 다대다

관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다. 그래서 보통 다대다 관계를 일대다, 다대일 관계로 풀어주기 위해 중간에 연결 테이블을 사용한다.

```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name="MEMBER_ID")
    private Long id;
    
    private String name;
    
    @ManyToMany
    @JoinTable(name="MEMBER_PRODUCT",
                joinColumn = @JoinColumn(name="MEMBER_ID"),
                inverseJoinColumns = @JoinColumn(
                    name="PRODUCT_ID"
                )
    )
    private List<Product> products = new ArrayList<Product>();
    
    ...
}

@Entity
public class Product {
    @Id @GeneratedValue
    @Column(name="PRODUCT_ID")
    private Long id;
    
    private String name;
    
    ...
}
```

`@ManyToMany`와 `@JoinTable`를 사용해서 연결 테이블을 바로 매핑했다.

- `name="MEMBER_PRODUCT"`: MEMBER_PRODUCT 테이블과 매핑했다.
- `joinColumn = @JoinColumn(name="MEMBER_ID")`: 현재 방향인 회원과 매핑할 조인 컬럼 정보를 지정한다.
- `inverseJoinColumns = @JoinColumn(name="PRODUCT_ID")`: 반대 방향인 상품과 매핑할 조인 컬럼 정보를 지정한다 

양방향 방법은 다른 연관관계랑 같기때문에 생략한다.

<br>

**연결 엔티티 사용**

조인 테이블로 자동으로 처리해주면 간단하지만 한계점이 존재한다. 조인 테이블의 추가적인 컬럼(주문 날짜, 주문 양)등이 필요할 때가 있는데 이를 해결해주지 못한다. 따라서 연결 엔티티를 별도로 만들어서 실제 연결 테이블이랑 매핑을 시켜주는 방법으로 해결한다.

이 때 기본키를 선택하는 전략이 2가지 존재한다. 두 테이블의 기본키를 사용해서 복합 기본키를 만드는 방법과 데이터베이스에서 자동으로 생성해주는 대리 키를 사용하는 것이다.

`복합 기본키`  
더 자세한건 7장에 나오지만 복합 기본키를 사용하면 복잡하다. 7장에 더 자세히..

`새로운 키본 키 사용`  
더 간편하고 비즈니스에 의존하지 않는 대리 키를 사용하는 것이 좋다.

```java
@Entity
public class Order{

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private int orderAmount;
}
```

위 코드를 보면 가독성도 좋고 쉽고 간편하게 연결 엔티티를 구성할 수 있다.