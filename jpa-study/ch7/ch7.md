## 상속 관계 매핑
객체는 상속이라는 개념이 있지만 데이터베이스에는 없다. 대신에 슈퍼타입, 서브타입 관계가 있는데 가장 유사하다. ORM에서 이야기하는 상속 관계 매빙은 객체의 상속과 데이터베이스의 슈퍼타입, 서브 타입 관계를 매핑하는 것이다.

매핑에 대해 다시 한 번 상기하자면 결국 객체와 데이터베이스의 맞지 않는 개념들을 JPA(ORM)이 중간에서 연결해준다는 느낌으로 이해하면 좋을 것 같다. 

JPA가 상속 관계 매핑을 위해 3가지 전략을 사용한다.

### 조인 전략
조인 전략이 객체 상속 개념과 논리적인 개념으로는 가장 유사한 전략인 것 같다. 객체에서 부모 `Item`과 자식들인 `Album`, `Movie`, `Book`이 있다고 할 때 부모와 자식들 테이블을 모두 만든다. 즉 4개의 테이블을 만들게 된다. 그리고 부모인 `Item`테이블의 자식을 구분하는 컬럼을 하나 추가 자식 테이블을 구분하고 자식 테이블은 부모 테이블의 기본키를 받아서 기본키 + 외래 키로 사용한다. 
이렇듯 조회할 때 조인을 사용해야 하기 때문에 조인전략이라고 한다.

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn(name="DTYPE")
public abstract class Item {
    @Id @GeneratedValue
    private Long id;
    
    private String name;

    @Column
    private Team team;
    
    ...
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item{

    private String aritist;
    ...
}

...
```

@Inheritance(strategy = InheritanceType.JOINED): 조인 전략으로 매핑 전략을 지정해준다.  
@DiscriminatorColumn(name="DTYPE"): 자식 테이블을 구분할 컬럼을 지정한다.
@DiscriminatorValue("A"): 엔티티를 저장할 떄 구분 컬럼에 입력할 값을 지정한다.

자식 테이블에서 기본키를 변경할 수 도 있으니 참고하자!

### 단일 테이블 전략
말그대로 테이블을 하나만 사용한다. 대신에 구분 컬럼으로 자식 데이터를 구분한다.  
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)으로 단일 테이블 전략을 사용하면 된다.

조인 전략과 단일 테이블 전략은 장단점이 존재한다. 조인 전략을 사용하면 테이블이 정규화가 되어서 저장 공간의 효율성이 올라간다. 정규화를 간단하게 설명하자면 중복되는 데이터나 빈 데이터가 많을 때 테이블을 쪼개서 공간의 효율성을 높이는 방법이다.  

단일 테이블 전략은 성능이 좋다. 조인 전략은 조인을 통해서 조회를 해야하지만 단일 테이블 전략은 테이블이 하나이기 때문에 조회 성능이 좋을 수 밖에 없다.  
상황에 맞게 두 전략을 고려하는 것이 중요한 것 같다.

### 구현 테이블 전략
구현한 자식 객체마다 테이블을 다 생성하는 것인데 실무에서 추천하지 않는 방법이므로 참고만 하자!

## @MappedSuperclass

부모 클래스틑 테이블과 매핑하지 않고 자식 클래스들만 테이블과 매핑하고 싶은 경우가 있다. 중복적으로 있는 생성 날짜와 같은 경우 여러 자식 클래스들이 갖고 있을 수 있는 필드이다. 이 때 사용하는 것이 @MappedSuperclass 어노테이션이다.

@MappedSuperclass는 추상 클래스와 비슷하다 @Entity 어노테이션과 다르게 테이블과 매핑되지 않고 단순히 매핑 정보를 상속할 목적으로 사용된다. 자식 엔티티는 상황에 따라 매핑 정보를 재정의할 수도 있다.

## 복합 키와 식별 관계 매핑

우선 알아야 하는 것은 데이터베이스 테이블 사이에 관계를 식별, 비식별 관계로 구분한다.

**식별 관계**: 자식 테이블이 부모 테이블의 기본 키를 받아서 자신의 기본키이자 외래키로 사용하는 경우

**비식별 관계**: 부모 테이블의 기본 키를 받아서 외래 키로만 사용하는 경우이고 또 두 가지로 나뉜다.
- 필수적 비식별 관계: 부모와 연관관계를 무조건 맺어야 한다. 즉 외래 키에 NULL을 허용하지 않는다.
- 선택적 비식별 관계: 부모와 연관관계를 맺을지 말지 선택할 수 있고, 외래 키에 NULL을 허용한다.

요즘 추세는 주로 비식별 관계를 사용하고 꼭 필요한 곳에만 식별 관계를 사용한다고 한다.

### 복합 키: 비식별 관계 매핑

단순히 복합 키를 설정하고 싶다고 `@Id`어노테이션을 한 엔티티에 두 개 만들면 매핑 오류가 발생한다. 식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야 한다.  
원래 JPA는 식별자 필드를 통해서 엔티티를 식별했다. 그러나 복합 키는 식별자 클래스로 엔티티를 식별한다.

**@IdClass**  

```java
@Entity
@IdClass(ParentId.class)
public class Parent{

    @Id
    @Column(name="PARENT_ID1")
    private String id1 //ParentId.id1과 매핑

    @Id
    @Column(name="PARENT_ID2")
    private String id2; //ParentId.id2과 매핑
    ...
}

public class ParentId implements Serializable{
    
    private String id1;
    private String id2;

    public ParentId(){

    }

    public ParentId(String id1, String id2){
        this.id1 = id1;
        this.id2 = id2;
    }
}
```

영속성 컨텍스트에 Parent를 저장할 때 두개의 식별자 필드 값에 키 값을 넣어주고 저장하면 된다. ParentId 클래스를 직접 생성할 필요 없다. JPA가 엔티티를 등록하기 전에 내부에서 식별자 클래스를 생성하고 영속성 컨텍스트의 키로 사용한다.

**@EmbeddedId**

좀 더 객체지향적인 방법이다. 실제 엔티티에 식별자 클래스를 필드로 바로 사용하는 것이다.

```java
@Entity
public class Parent{

    @EmbeddedId
    private ParentId id;

    ...
}

@Embeddalbe
public class ParentId implements Serializable{
    
    @Column(name="PARENT_ID1")
    private String id1;

    @Column(name="PARENT_ID2")
    private String id2;

    public ParentId(){

    }

    public ParentId(String id1, String id2){
        this.id1 = id1;
        this.id2 = id2;
    }
}
```

대신에 `@EmbeddedId`를 사용하면 영속성 컨텍스트에 저장할 때 식별자 클래스를 직접 생성하고 엔티티에 저장해줘야 한다.

주의해야하는 점은 두 방식 모두 equals(), hashCode()를 필수로 구현해야한다.
영속성 컨텍스트는 식별자를 비교할때 equals(), hashCode()를 사용하는데 해당 메서드들을 구현하지 않으면 같은 데이터를 갖고 있지만 인스턴스가 달라서 새로운 엔티티로 판단하게 된다. 따라서 잘 구현하자!

@IdClass와 @EmbeddedId 방식 모두 괜찮은 방법이기 때문에 본인의 취향에 맞는 것을 일관성있게만 사용하면 된다.

지금까지는 비식별 관계 매핑이었다. 복합 키와 식별 관계 매핑도 할 수 있지만 책을 참고 하도록 하자.. 기본 키가 계속 자식 테이블에 추가되고 자식의 자식 테이블이 있으면 또 추가 되고 이런 상황들이 있기 때문에 복잡하다.

데이터베이스 관점과 JPA 관점에서 보면 모두 비식별 관계를 선호한다.  
식별 관계인 경우 기본 키 컬럼이 2개 이상인 경우가 많고 비즈니스 요구사항의 변함에 유연하게 대처하지 못하다. 따라서 기본적으로 비식별 관계를 지향하고 필요에 따라 식별 관계를 사용하자! 식별 관계의 장점은 별도의 인덱스를 생성할 필요 없이 기본 키 인덱스를 활용하여 부모와 자식 조회를 할때 더 효율적으로 할 수 있다. 또한, 선택적 비식별 관계보다는 필수적 비식별 관계가 더 좋은데 선택적 비식별은 NULL을 허용하므로 외부 조인을 사용해야 한다.

**식별보다는 비식별을 지향하고 필수적 비식별 관계를 사용하자!!**