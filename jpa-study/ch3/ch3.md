### 영속성 컨텍스트는 뭔가?

책에서 영속성 컨텍스트(persistence context)를 굳이 한국어로 번역하면 '엔티티를 영구 저장하는 환경'이라고 소개한다. 결국 영속성 엔티티, 영속성 컨텍스트 등등 영속성과 관련된 용어는 데이터를 저장하고 관리하는 방법론을 설명하기 위한 용어인 것 같다.  

영속성 컨텍스트는 논리적인 개념이고 엔티티 매니저가 생성되면 생성된다. 엔티티 매니저를 통해 데이터 베이스 접근할 수 있는데(가상의 데이터 베이스라고 생각하면 편함) 데이터 베이스 연결이 꼭 필요한 시점까지 커넥션을 얻지 않는다. 예를 들어 트랜잭션을 시작할 때 커넥션을 얻는다.

```
사용자 요청 -> 앤티티 매니저 생성 -> 트랜잭션 시작 -> 커넥션 획득 
```

내가 느끼기에 영속성 컨텍스트는 데이터베이스에 저장된 데이터(엔티티) 혹은 저장할 데이터(엔티티)를 어플리케이션 환경에서 먼저 1차적으로 처리하는 논리적인 공간같다.

#### 엔티티의 생명주기

영속성 컨텍스트를 이해하기 위해서는 엔티티의 생명주기(상태)에 대해 이해하면 좋다.
- **비영속**: 영속성 컨텍스트와 전혀 관계가 없는 상태이며 주로 엔티티를 생성했을 때 이다.
- **영속**: 영속성 컨텍스트에 엔티티가 저장된 상태이다.
- **준영속**: 영속성 컨텍스트에 저장되었다고 다시 나온 상태이다.
- **삭제**: 엔티티가 삭제된 상태이다.

#### 영속성 컨텍스트의 특징

영속성 컨테스트는 크게 **1차 캐시**, **스냅샷**, **쓰기 지연 SQL 저장소**로 구성되어 있다.  

**엔티티 저장**  
엔티티를 영속성 컨텍스에 저장을 하면 바로 1차 캐시에 저장되는데 key, value의 형식 **Map**으로 관리된다. 여기서 **Key**는 엔티티의 식별자 값(@Id로 테이블의 기본 키와 매핑한 필드)이고 **value**는 엔티티 인스턴스다.  

JPA는 엔티티를 저장할 때 데이터 베이스에 sql문을 날리지 않는다. 쓰기 지연 sql 저장소에 쿼리를 모아놓고 나중에 commit이나 flush를 실행할 때 모든 쿼리를 데이터 베이스에 반영한다.

```
캐시라는 용어가 헷갈렸다. 컴퓨터 구조에서 캐시라는건 메모리와 프로세서 사이에 있는 게층으로 메모리보다 더 빨리 데이터를 읽을 수 있는 데이터 저장 장치이다. 그러나 여기서 말한는 1차 캐시는 하드웨어적인 개념의 캐시가 아니라 소프트웨어 적으로 어플리케이션 메모리에서 관리된다.
``` 
<br>

**엔티티 조회**  
엔티티를 조회할 때는 먼저 1차 캐시에서 해당 엔티티에 식별자 값으로 조회한다. 만약에 영속성 컨텍스트(1차 캐시)에 없다면 데이터베이스에서 엔티티를 읽어와서 영속성 컨텍스트에 저장하고 반환한다.

**엔티티 수정**  
JPA는 수정하는 메서드가 따로 있지 않다. 영속성 컨텍스트에 엔티티가 저장되면 스냅샷으로 최초로 저장될때에 상태를 저장한다. 따라서 엔티티를 수정하면 스냅샷과 비교하여 자동으로 쓰기 지연 저장소에 sql문을 저장한다. 삭제도 수정과 마찬가지로 쓰기 지연 저장소에 sql문을 자동으로 만들어주고 나중에 commit나 flush가 실행될 때 데이터 베이스에 반영된다. 

### 영속성 컨텍스트를 왜 사용하는가?
JPA는 왜 굳이 영속성 컨텍스트를 사용해서 엔티티를 관리하는 지를 생각해보았다. 결국에 효율성 때문이라고 생각한다. 
데이터 베이스에 **commit**를 해야 데이터가 영구적으로 반영이 되는데 그 말은 **commit**를 하기전에 무슨 짓을 해도 상관없다. (무슨 짓이 뭐냐에 따라 다르긴 함 ;;)

<br>

만약에 영속성 컨텍스트 없이 그때 그때 데이터베이스와 통신한다고 가정해보자.
```
트랜잭션 시작 -> 엔티티 저장 -> 데이터 베이스한테 쿼리 날림 -> 저장된 엔티티 결과 받음 -> 엔티티 수정 -> 데이터 베이스한테 쿼리 날림 -> 수정된 엔티티 결과 받음 -> ... -> 트랙잭션 종료(commit)
```
엔티티를 저장하거나 수정하거나 할때 매번 데이터베이스와 통신을 해야한다. 시간이 오래 걸릴 수 밖에 없다. 집돌이는 나가는 김에 친구도 만나고 물건도 사고 커피도 먹고 싶다.

<br>

영속성 컨텍스트에서 제공하는 **1차 캐시**, **스냅샷**, **쓰기 지연 SQL 저장소**를 통해서 SQL문 모아놓고 나중에 필요할 때 한번에 처리할 수 있다.
```
트랙잭션 시작 -> 엔티티 저장 -> 엔티티 수정 -> ... -> 트랜잭션 종료(commit) -> 데이터 베이스한테 쿼리 날림
```
결론적으로 영속성 컨텍스트로 엔티티를 관리하는 이유는 성능 최적화 문제이다. 한번의 외출로 집돌이는 친구도 만나고 물건도 사고 커피도 먹을 수 있게 된 것이다...!! 효율 최고