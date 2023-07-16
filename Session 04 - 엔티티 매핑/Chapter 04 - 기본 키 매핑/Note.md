# 기본 키 매핑
> 작성자: @luke0408

## 목차
- [기본 키 매핑](#기본-키-매핑)
  - [목차](#목차)
- [기본키 매핑 방법](#기본키-매핑-방법)
  - [직접 할당 (@Id)](#직접-할당-id)
    - [@Id 어노테이션 적용 범위](#id-어노테이션-적용-범위)
  - [자동 생성 (@GeneratedValue)](#자동-생성-generatedvalue)
    - [IDENTITY 전략](#identity-전략)
    - [SEQUENCE 전략](#sequence-전략)
    - [TABLE 전략](#table-전략)
    - [AUTO 전략](#auto-전략)
  - [권장하는 식별자 전략](#권장하는-식별자-전략)


# 기본키 매핑 방법
> 기본키란? 테이블에서 특정 레코드를 구분할 수 있는 식별자

**[기본키 어노테이션]**
- @Id : 직접 할당
- @GeneratedValue : 자동 생성

ex)
```java
@Id @GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

## 직접 할당 (@Id)
> @GeneratedValue 어노테이션 없이 @Id를 사용하여 직접 할당

### @Id 어노테이션 적용 범위

- @Id는 엔티티 클래스의 필드에만 적용 가능
- 사용 가능 타입 (아래 중 하나)
  - primitive 타입
  - primitive 타입의 Wrapper 클래스
  - String
  - java.util.Date, java.sql.Date
  - java.math.BigInteger, java.math.BigDecimal

## 자동 생성 (@GeneratedValue)
> (Optional) @Id 어노테이션과 함께 사용하며 기본키의 생성 전략을 명시

### IDENTITY 전략
> 기본키 생성을 데이터베이스에 위임

**[IDENTITY 전략의 특징]**
- 기본키 생성을 데이터베이스에 위임
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용 <br>
  (ex. MySQL의 AUTO_INCREMENT)
- JPA는 보통 `트랜잭션 commit 시점`에 INSERT SQL 실행
- AUTO_INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
- IDENTITY 전략은 `em.persist() 시점`에 즉시 INSERT SQL을 실행하고 DB에서 식별자를 조회

**[IDENTITY 전략의 작동원리]**
- IDENTITY 전략은 INSERT SQL을 실행한 이후에야 식별자를 구할 수 있음
- 즉, Entity의 식별자를 알려면 `반드시 데이터베이스에 INSERT를 먼저 해야 함`
- 따라서 IDENTITY 전략만 예외적으로 `EntityManager.persist()`를 호출하는 시점에 INSERT SQL을 DB에 전달하고, DB에서 식별자를 조회

ex)
```java
@Entity
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
}
```

### SEQUENCE 전략

**[SEQUENCE 전략의 특징]**
- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트<br>
  (ex. 오라클 시퀀스)
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용 가능

**[@SequenceGenerator 어노테이션]**

<center>

|속성|설명|기본값|
|:--|:--|:--|
|name|식별자 생성기 이름|필수|
|sequenceName|데이터베이스에 등록되엉 있는 시퀀스 이름|hibernate_sequence|
|initialValue|DDL 생성시에만 사용될, 시퀀스 DDL을 생성할 때 처음 1 시작하는 수를 지정|1|
|allocationSize|시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 이용)<br> **데이터베이스 시퀀스 값이 하나씩 증가하도록 설정되어 있으면 이 값을 반드시 1로 설정해야 한다.** | **50** |
|catalog, schema|데이터베이스 catalog, schema의 이름||

</center>

**[SEQUENCE 전략의 작동원리]**
- DB에 SEQUENCE를 만들고 시작값을 맞춰줘야 함
  - initialValue = 1, allocationSize = 1 > 1, 2, 3, 4, 5, ...
- SEQUENCE 전략은 `em.persist()` 시점에 먼저 DB 시퀀스를 사용해서 식별자를 조회
- 그렇게 조회해온 식별자를 엔티티에 할당하고, 엔티티를 영속성 컨텍스트에 저장
- 이후 트랜잭션 커밋 시점에 INSERT SQL 실행

**[SEQUENCE 전략의 의문점]**
- 위 방식대로 진행한다면 DB 시퀀스를 사용해서 식별자를 조회하고, 엔티티에 할당하고, 영속성 컨텍스트에 저장하는 과정에서 `데이터베이스와 2번 통신`하게 됨
- 즉, `성능상 이슈`가 발생할 수 있음

**[SEQUENCE 전략의 해결책]**
- allocationSize를 적절한 크기로 설정 (기본값이 50인 이유)
- DB에 시퀀스 값을 미리 증가시킴
  - ex) allocationSize = 50 > 1, 51, 101, 151, 201, ...
- 이후 `메모리에서만 시퀀스 값을 증가`시킴
  - ex) 1, 2, 3, 4, 5, ... , 50
- allocationSize 단위만큼 메모리에서 시퀀스 값을 증가시키다가, 트랜잭션 커밋 시점에 DB에 증가된 시퀀스 값을 전달

ex)
```java
@Entity
@SequenceGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    sequenceName = "MEMBER_SEQ", // 매핑할 데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1
)
public class Member {
    @Id 
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

### TABLE 전략

**[TABLE 전략의 특징]**
- 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
- 장점
  - 모든 데이터베이스에 적용 가능
- 단점
  - 성능

**[@TableGenerator 어노테이션]**

<center>

|속성|설명|기본값|
|:--|:--|:--|
|name|식별자 생성기 이름|필수|
|table|키 생성 테이블명|hibernate_sequence|
|pkColumnName|시퀀스 컬럼명|sequence_name|
|valueColumnName|시퀀스 값 컬럼명|next_val|
|pkColumnValue|키로 사용할 값 이름|엔티티 이름|
|initialValue|초기 값, 마지막으로 생성된 값이 기준이 됨|0|
|allocationSize|시퀀스 한 번 호출에 증가하는 수 (성능 최적화에 이용) <br> [SQUENCE 전략](#sequence-전략)의 해결책과 같은 방식|50|
|catalog, schema|데이터베이스 catalog, schema의 이름||
|uniqueConstraints(DDL)|유니크 제약 조건 지정||

</center>

ex)
```java
@Entity
@TableGenerator(
    name = "MEMBER_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
public class Member {
    @Id 
    @GeneratedValue(strategy = GenerationType.TABLE,
                    generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
}
```

### AUTO 전략
> 기본값, 설정하지 않으면 자동으로 AUTO 전략을 사용

ex)
```java
@Entity
public class Member {
    @Id 
    @GeneratedValue(strategy = GenerationType.AUTO) // 생략 가능
    private Long id;
}
```

## 권장하는 식별자 전략
> 미래까지 이어질 수 있는 자연키를 찾기는 어렵다. 그러니 대체키를 사용하자.

**[기본 키 제약 조건]**
- null 아님 (not null)
- 유일 (unique)
- `변하면 안됨 (immutable)`

**[권장하는 식별자 전략]**
- Long형 + 대체키 + 키 생성전략 사용
  - Long형 : 10억이 넘어가도 오버플로우 발생하지 않음
  - 대체키 : 비즈니스에 의존하지 않는다. ex) 시퀀스, 키 생성 테이블, UUID
