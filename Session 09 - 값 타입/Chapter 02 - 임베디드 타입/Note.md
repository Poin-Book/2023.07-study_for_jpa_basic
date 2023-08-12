# 임베디드 타입

> 작성자: @luke0408

## 목차

- [임베디드 타입](#임베디드-타입)
  - [목차](#목차)
  - [임베디드 타입(Embedded Type)이란?](#임베디드-타입embedded-type이란)
    - [임베디드 타입의 장점](#임베디드-타입의-장점)
  - [임베디드 타입 사용법](#임베디드-타입-사용법)
    - [예시: with @Embeddable, @Embedded](#예시-with-embeddable-embedded)
    - [예시: with @AttributeOverrides, @AttributeOverride](#예시-with-attributeoverrides-attributeoverride)
  - [임베디드 타입의 특징](#임베디드-타입의-특징)
  - [임베디드 타입 사용시 주의점](#임베디드-타입-사용시-주의점)
    - [임베디드 타입 공유 지양](#임베디드-타입-공유-지양)
    - [@Embeddable과 static \[^1\]](#embeddable과-static-1)
  - [Q\&A](#qa)
    - [1. @MappedSuperclass vs @Embedded \[^2\]](#1-mappedsuperclass-vs-embedded-2)
  - [참고 자료](#참고-자료)

## 임베디드 타입(Embedded Type)이란?

- 복합 값 타입을 의미함
- 새로운 값 타입을 직접 정의할 수 있음
- JPA는 임베디드 타입(embedded type)이라 함
- 주로 기본 값 타입을 모아서 만들어서 복합 값 타입이라고도 함

### 임베디드 타입의 장점

- 재사용이 가능함
- 높은 응집도를 가짐
- Period.isWork()처럼 해당 값 타입만 사용하는 의미있는 메소드를 만들 수 있음
  - 현재 근무기간에 속하는지에 대한 메소드를 만들 수 있으며 객체 지향적으로 사용할 수 있음
- 임베디드 타입을 포함한 모든 값 타입은, 값 타입을 소유한 엔티티에 생명주기를 의존함
  - 엔티티가 생성될 때 임베디드 타입도 함께 생성되고, 엔티티가 삭제될 때 임베디드 타입도 함께 삭제됨

## 임베디드 타입 사용법

- @Embeddable: 값 타입을 정의하는 곳에 표시 (임베디드 타입을 사용하기 위해 생성한 Class에 표시)
- @Embedded: 값 타입을 사용하는 곳에 표시 (임베디드 타입을 사용하는 Entity에 표시)
- @AttributeOverrides: 임베디드 타입을 재정의할 때 사용 (임베디드 타입을 재정의할 때 사용)
- @AttributeOverride: 속성을 재정의할 때 사용 (임베디드 타입을 재정의할 때 사용)

### 예시: with @Embeddable, @Embedded

- 임베디드 타입을 사용하기 위해 생성한 Class에 표시
- 입베디드 타입을 선언할 때는 기본 생성자가 필수로 있어야 함

<br>

- Period Class

  ```java
  @Getter
  @Setter
  @NoArgsConstructor // 기본 생성자 필수
  @AllArgsConstructor
  @Embeddable
  public class Period {
    private LocalDateTime startDate;
    private LocalDateTime endDate;

    public boolean isWork() {
      // 근무 중인지 확인하는 로직
    }
  }
  ```

- Address Class

  ```java
  @Getter
  @Setter
  @NoArgsConstructor // 기본 생성자 필수
  @AllArgsConstructor
  @Embeddable
  public class Address {
    private String city;
    private String street;
    private String zipcode;
  }
  ```

- 임베디드 타입을 사용하는 entity에 표시

  ```java
  @Entity
  public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @Embedded
    private Period workPeriod;

    @Embedded
    private Address address;
  }
  ```

- Member 생성
  - member를 persist하면 임베디드 타입이 없는 엔티티 생성 쿼리문과 같은 쿼리문을 확인할 수 있다.

  ```java
  Member member = new Member();
  member.setName("회원1");
  member.setWorkPeriod(new Period());
  member.setAddress(new Address("서울", "강가", "123-123"));
  
  em.persist(member);
  ```

  - 만약 아래와 같이 임베디드 타입에 null를 넣는다면 매핑한 칼럼 값은 모두 null이 된다.

  ```java
  Member member = new Member();
  member.setName("회원1");
  member.setWorkPeriod(null);
  member.setAddress(null);

  em.persist(member);
  ```

### 예시: with @AttributeOverrides, @AttributeOverride

- 만약 하나의 엔티티에 같은 임베디드 타입을 중복해서 사용하고 싶다면 @AttributeOverrides를 사용하면 된다.

- Without @AttributeOverrides, @AttributeOverride
  - homeAddress와 workAddress의 컬럼명이 중복되므로 Repeated column in mapping for entity error가 발생합니다.

  ```java
  @Entity
  public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;

    @Embedded
    private Address workAddress;
  }
  ```

- With @AttributeOverrides, @AttributeOverride

  ```java
  @Entity
  public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @Embedded
    private Period workPeriod;

    @Embedded
    private Address homeAddress;

    @Embedded
    @AttributeOverrides({
      @AttributeOverride(name = "city", column = @Column(name = "WORK_CITY")),
      @AttributeOverride(name = "street", column = @Column(name = "WORK_STREET")),
      @AttributeOverride(name = "zipcode", column = @Column(name = "WORK_ZIPCODE"))
    })
    private Address workAddress;
  }
  ```

## 임베디드 타입의 특징

- 임베디드 타입은 엔티티의 값일 뿐이다.
- 임베디드 타입을 사용하기 전과 후에 매핑하는 테이블에는 변화가 없다.
- 객체와 테이블을 아주 세밀하게(find-grained) 매핑하는 것이 가능하다.
- 잘 설계한 ORM 애플리케이션은 매핑한 테이블의 수보다 클래스의 수가 더 많다.
  - Address, Period 등 class로 빼놓으면 활용할 수 있는 곳이 많다.
  - 이러한 value type들이 많아지기 때문에 테이블의 수보다 클래스의 수가 더 많아진다.
- 도메인의 언어를 공통으로 맞출 수 있다.
  - 예) Address를 사용할 때는 city, street, zipcode 필드를 고정으로 이용한다.
- 임베디드 클래스 안에 Entity를 넣어서 사용 가능하다.

## 임베디드 타입 사용시 주의점

### 임베디드 타입 공유 지양

- 사이드 이펙트 예시

  ```java
  Address address = new Address("서울", "강가", "123-123");
  Member member1 = new Member();
  member1.setName("회원1");
  member1.setAddress(address);

  Member member2 = new Member();
  member2.setName("회원2");
  member2.setAddress(address);

  em.persist(member1);
  em.persist(member2);

  // member2의 address를 변경하면 member1의 address도 변경됨
  member2.getAddress().setCity("newCity");
  ```

[사이드 이펙트 해결 방법]

> Address의 Setter를 구현하지 않거나, private으로 구현

- 객체 타입을 수정할 수 없게 만들어 부작용을 원천 차단
- 값 타입은 불변 객체로 설계해야함
- 생성자로만 값을 설정하고 수정자(Setter)를 만들지 않으면 됨
- 참고: Integer, String은 자바가 제공하는 대표적인 불변 객체

### @Embeddable과 static [^1]

[문제의 코드]

- Pay Entity

  ```java
  @Getter
  @NoArgsConstructor
  @Entity
  public class Pay {
    @Id @GeneratedValue
    private Long id;

    private long amount;

    @Embedded
    private PayDetails payDetails = PayDetails.EMPTY; // (1)

    @Embedded
    private PayEvents payEvents = new PayEvents(); // (2)

    ...
  }
  ```

- PayDetails & PayEvents

  ```java
  @Getter
  @NoArgsConstructor
  @Embeddable
  public class PayDetails {
    public static final PayDetails EMPTY = new PayDetails(); // 전역 변수

    @OneToMany(cascade = CascadeType.ALL, mappedBy = "pay")
    private List<PayDetail> payDetails = new ArrayList<>();
    
    ...
  }
  ```

  ```java
  @Getter
  @NoArgsConstructor
  @Embeddable
  public class PayEvents {
    @OneToMany(cascade = CascadeType.ALL, mappedBy = "pay")
    private List<PayEvent> payEvents = new ArrayList<>();
    
    ...
  }
  ```

[Dto Test 코드]

```java
@Test
void PayDetails_test() throws Exception {
    //given
    Pay pay1 = new Pay();
    Pay pay2 = new Pay();

    //then
    System.out.printf("empty=%s\n pay1=%s\n pay2=%s\n", PayDetails.EMPTY, pay1.getPayDetails(), pay2.getPayDetails());
    assertThat(pay1.getPayDetails()).isEqualTo(pay2.getPayDetails());
}
```

[문제의 코드의 문제점 2가지]

1) `PayDetails.EMPTY`는 Static으로 선언되어 **애플리케이션 실행시 딱 한번 생성**되고, 이후로는 **재사용**된다.
2) `Pay` Entity의 멤버 변수인 `PayDetails`는 `PayDetails.EMPTY`를 참조 (`payDetails = PayDetails.EMPTY`) 받는다.

즉, 위 Test 코드는 아래와 같이 `=`로 **참조를 전달**할 뿐이지 **새로운 객체를 생성**하는 것이 아니다.

```java
PayDetails payDetails1 = PayDetails.EMPTY;
PayDetails payDetails2 = PayDetails.EMPTY;
```

위와 같은 문제가 실제 운영 환경으로 이루어 진다면 아래와 같은 문제로 이어진다.

- A Pay가 payDetail a를 `add` 하고나면, 새롭게 추가되는 B Pay는 **payDetail를 생성하지 않더라도 payDetail a를 가지게 된다.**
- 만약 이런 상태에서 배포등으로 인해 스프링 애플리케이션이 재실행된다면 당연히 `PayDetails.EMPTY`가 다시 생성된다.
- 그러면 이전에 `add` 된 details들은 모두 사라지게 된다.

[문제점 해결 방법]

- 굳이 전역 상수화 시키지 말고, 아래처럼 new 키워드로 매번 생성하는 코드를 작성하면 된다.

```java
@Embedded
private PayDetails payDetails = new PayDetails();
```

## Q&A

### 1. @MappedSuperclass vs @Embedded [^2]

[JPA에서의 차이]

단순하게 이야기 하면 두 어노테이션의 가장 큰 차이는 `상속`과 `위임`이다. 객체지향을 공부하다보면 `상속`과 `조합`의 차이에 대해서 공부하게 되는데 이 상속과 조합의 차이가 두 어노테이션의 차이와 같다고 생각하면 된다.

보다 더 유연한 객체를 다루기 위해 부모 클래스와 의존성이 강하게 엮인 `상속`보다는 `조합`을 이용하라는 말을 많이 들어봤을 것이다. 하지만 단순히 `엔티티의 중복된 필드를 재사용하기 위한 목적`이라면 상속을 고려하는 것이 더욱 편리하다. 또한 `@MappedSuperclass`와 `Auditing` 기능을 함께 사용하면 더 편하게 필드 값을 다룰 수 있게 된다.

[JPQL에서의 차이]

- `@MappedSuperclass` 이용시 JPQL 작성
  
  ```java
  public interface MappedSuperclassCrewRepository extends JpaRepository<MappedSuperclassCrew, Long> {

    @Query("SELECT c " 
            + "FROM MappedSuperclassCrew c " 
            + "WHERE c.createdAt > :dateTime")
    List<MappedSuperclassCrew> findByCreatedAtGreaterThan(final LocalDateTime dateTime);
  }
  ```

- `@Embedded` 이용시 JPQL 작성
  
  ```java
  public interface EmbeddedCrewRepository extends JpaRepository<EmbeddedCrew, Long> {

    @Query("SELECT c "
            + "FROM EmbeddedCrew c "
            + "WHERE c.traceDateTime.createAt > :dateTime")
    List<EmbeddedCrewRepository> findByCreatedAtGreaterThan(final LocalDateTime dateTime);
  }

- 차이점
  - 임베디드 타입을 사용한 경우 `c.traceDateTime.createAt` 처럼 임베디드 타입의 필드를 명시해줘야 한다.
  - 위 예시는 traceDateTime이라는 임베디드 타입을 엔티티에 포함시킨 상태라고 생각하면 된다.

## 참고 자료

[^1]: [JPA 사용시 @Embedded 주의사항](https://jojoldu.tistory.com/559) <br>
[^2]: [@MappedSuperclass vs @Embedded](https://hyeonic.github.io/jpa/basic/mapped-superclass-vs-embedded.html#mappedsuperclass)