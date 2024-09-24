### 엔티티 매핑

JPA가 데이터 베이스에서 테이블과 칼럼들을 잘 사용할 수 있도록 매핑시켜줘야한다. 물론 제대로 매핑이 되어 있지 않으면 기본 값을 사용하거나 런타임 에러가 발생한다.

**회원 엔티티**
```JAVA
@Entitiy
@Table(name="MEMBER")
public class Member{

    @Id
    @Column(name="ID")
    private String id;

    @Column(name="NAME")
    private String username;

    private Integer age;

    @Enumerated(EmumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Lob
    private String description;
    ...
}
```
위와 같이 자바코드가 있다고 생각하고 하나씩 뜯어보자.

**@Entity**  
클래스에 @Entity 어노테이션을 붙여야 JPA가 관리할 수 있고, 이를 엔티티라고 부른다. @Entity를 적용 시 주의사항들이 있다.
- 기본 생성자는 필수다.
- final 클래스, enum, interface, inner 클래스에는 사용할 수 없다.
- 저장할 필드에 final을 사용하면 안된다.

JPA는 엔티티 객체를 생성할 때 기본 생성자를 사용한다. 만약 엔티티에 기본 생성자가 없으면 자동으로 만들지만 다른 생성자가 있을 경우 직접 기본 생성자를 만들어 줘야한다.

**@Table**  
엔티티와 매핑할 테이블을 지정해주는 어노테이션이다. 위 코드에서 name 속성을 사용하였는데 매핑할 테이블 이름을 지정하는 것이다. name속성의 기본값은 엔티티 이름을 사용한다.

@Table 어노테이션 속성에서 유니크 제약조건을 추가하여 해당 테이블에 유니크 제약조건을 추가해줄 수 있다.
```JAVA
@Table(name="MEMBER", uniqueConstraints = {
    @UniqueContraint(
        name="NAME_AGE_UNIQUE",
        columnNames={"NAME", "AGE"}
    )
})
```

**@Id**  
@Id는 기본키를 매핑해주는 어노테이션이다. 기본키를 할당하는 방법으로는 개발자가 직접 기본 키를 애플리케이션에서 할당해주는 방법과 자동 생성으로 할당하는 방법이 있다.  
자동 생성 방식은 여러 가지가 있는데 그 이유는 데이터베이스 벤더마다 지원하는 방식이 다르기 때문이다.
- IDENTITY: 기본 키 생성을 데이터베이스에 위임하는 방식이며, 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용한다.

- SEQUENCE: 데이터베이스 시퀀스를 사용해서 기본 키를 할당하는 방식이며, 주로 오라클, PostgreSQL, DB2, H2에서 사용할 수 있다.

- TABLE: 키 생성 전용 테이블을 하나 만들고 시퀀스를 흉내내는 전략이며, 모든 데이터 베이스에서 사용 가능한 전략이다.

IDENTITY와 SEQUENCE전략은 데이터 베이스와 의존성이 존재하고 TABLE은 모든 데이터 베이스에서 사용이 가능한 방법이다. 각가 전략의 동작 방식은 뒤에 더 자세히 알아보자.

**@Column**  
@Column는 자바의 객체 필드를 테이블의 컬럼에 매핑한다. 여러 가지 속성들이 있지만 가장 많이 사용하는 어노테이션은 name, nullabe이다. name의 값으로는 테이블의 컬럼 이름이 들어가고 nullable은 해당 컬럼에 null값이 가능한지에 대한 속성이다.

**@Enumerated**  
자바의 enum 타입을 매핑할 때 사용한다. 속성 값으로 EnumType.STRING과 EnumType.ORDINAL을 줄 수 있다.
```java
enum RoleType{
    ADMIN, USER
}
```
위와 같이 자바 enum타입이 있을 때
```java
member.setRoloType(RoleType.ADMIN)
```
엔티티에 ADMIN을 저장할 경우 속성 값에 따라 다르게 저장된다.
- EnumType.STRING: 'ADMIN' 문자열로 저장된다.
    - 저장되는 데이터는 ORDINAL보다 크지만 확장성이 좋다. enum 타입을 추가해도 안전하다.

- EnumType.ORDINAL: 정의된 순서대로 0이 저장된다.
    - 저장되는 데이터가 STRING보다 적다. 하지만 추후에 개발 도중에 enum으로 정의된 순서가 달라지면 위험하다.

**@Temporal**  
자바에서 Date 타입에는 년월일 시분초가 있지만 데이터베이스에는 date(날짜), time(시간), timestamp(날짜와 시간)라는 3가지 타입이 별도로 존재한다. 따라서 요구사항에 따라 속성값으로 제대로 매핑해줘야 한다.
```
자바 8부터 도입된 java.time 패키지를 사용할 것을 여러 커뮤니티에서 강력하게 권장하고 있다. 불변객체로 쓰레드에 안전하고 더 직관적이고 일관적으로 쓸 수 있기 때문이다. LocalData, LocalTime, LocalDateTime등 상황에 맞게 클래스를 사용하면 된다.
```

### 기본 키 자동 생성 전략

**IDENTITY 전략**  
기본 키 생성을 데이터베이스에게 위임하는 전략이다. 
```java
...

@Id
@GeneratedValue(strategy=GenerationType.IDENTITY)
private Long id;

...
```

엔티티가 영속상태가 되려면 식별자(기본 키)가 반드시 필요하다. 데이터 베이스에게 기본 키 생성을 위임했기 때문에 데이터 베이스에 INSERT 즉 저장을 해야 식별자를 구할 수 있다. 따라서 영속성 컨텍스트에 엔티티를 저장하면 쓰기 지연 저장소에 INSERT SQL이 쌓이지 않고 즉시 데이터 베이스에 전달된다.

**SEQUENCE 전략**  
데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 오브젝트이다. 데이터 베이스가 제공하는 기능중에 하나라고 생각해도 좋을 듯 하다. 시퀀스를 사용하려면 우선 데이터 베이스에 시퀀스를 생성해야 한다.
```SQL
//예시
CREATE SEQUENCE [sequenceName] 
START WITH [initialValue] INCREMENT BY [allocationSize];

CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
```
그리고 나서 해당 시퀀스랑 엔티티를 매핑해줘야 한다.
```java
@Entity
@SequenceGenerator(
    name="BOARD_SEQ_GENERATOR",
    sequenceName="BOARD_SEQ",
    initialValue=1, allocationSize=1
)
public class Board{
    
    @Id
    @GeneratedValue(strategy=GenerationType.SEQUENCE,
                    generator="BOARD_SEQ_GENERATOR")
    private Long id;
    ...
}
```

@SequenceGenerator는 자바 애플리케이션에서 사용할 시퀀스 생서기를 등록하게 해주는 어노테이션이다. name속성은 자바 애플리케이션에서 식별하기 위한 name이고 sequenceName은 실제 데이터 베이스 시퀀스랑 매핑해주는 속성이다.  
내부적으로 동작하는 방식이 IDENTITY와 다르다. 시퀀스 전략은 엔티티를 엔티티 컨텍스느에 저장할 때 실제로 데이터 베이스에 저장하고 기본 키를 받는 것이 아니라 데이터 베이스 시퀀스를 사용해서 식별자를 조회한다. 그 이후에 트랜잭션을 커밋해서 플러시가 일어나면 해당 엔티티가 데이터베이스에 저장되는 방식이다.

initalValue은 기본 키의 초기 값을 설정하는 속성이고 allocationSize는 시퀀스 한 번 호출에 중가하는 수를 설정한다. 데이터 베이스의 시퀀스를 조회해야하기 때문에 어쨌든 데이터 베이스와 통신을 한번 해야한다. 이러한 상황을 줄이고 성능을 올리고자 할 경우에 allocationSize값을 수정하면 된다.

allocationSize는 시퀀스를 조회할 때 한번에 받아오는 식별자(시퀀스)의 수를 설정하는 속성이다. 예를들어 50으로 설정을 했다고 가정하면 JPA는 1~50 시퀀스 값을 미리 할당받아서 메모리에 저장한다. 그리고 엔티티를 영속화시킬 때 데이터 베이스를 조회하지 않고 미리 받은 시퀀스를 사용한다. 51번째 엔티티를 영속화시킬 때 다시 데이터 베이스 시퀀스를 조회해서 시퀀스 값들을 받아온다.

**TABLE 전략**  
테이블 전략은 키 생성 전용 테이블을 하나 만들고 데이터베이스 시퀀스를 흉내내는 전략이다. 먼저 키 생성 용도로 사용할 데이블을 만들어야 한다.
```SQL
create table MY_SEQUENCES(
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)
```
```java
@Entity
@TableGenerator(
    name="BOARD_SEQ_GENERATOR",
    talbe="MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ", allocationSize=1
)
public class Board{
    
    @Id
    @GeneratedValue(strategy=GenerationType.TABLE,
                    generator="BOARD_SEQ_GENERATOR")
    private Long id;
    ...
}
```
@TableGenerator를 사용해서 BOARD_SEQ_GENERATOR이라는 테이블 키 생성자를 등록하고 MY_SEQUENCES테이블의 BOARD_SEQ컬럼과 매핑하였다. 내부적으로 시퀀스 전략과 동일하게 동작하며 하나의 테이블의 여러 시퀀스을 만들 수 있다.
| SEQUENCE_NAME | SEQ_VALUE |
|---------------|-----------|
| BOARD_SEQ     | 1         |
| MEMBER_SEQ     | 1         |

테이블 전략을 시퀀스 전략과 비교해서 쿼리를 한번 더 사용한다. 먼저 테이블을 조회하는 쿼리와 다음 값을 증가키기기 위한 update 쿼리를 사용한다. 이러한 단점들이 있기때문에 성능을 최적화 하는 방법으로 시퀀스 전략과 마찬가지로 allocationSize를 조절해주면 좋아진다.

**AUTO 전략**  
오토 전략은 데이터베이스 방언에 따라 3가지 전략중에 하나를 자동으로 선택해준다. 예를 들어 오라클은 SEQUENCE 전략, mysql은 IDENTITY 전략을 선택한다.

스키마 자동 생성 기능을 사용한다면 하이버네이트가 기본값을 사용해서 적절한 시퀀스나 키생성용 테이블을 만들어 준다.
