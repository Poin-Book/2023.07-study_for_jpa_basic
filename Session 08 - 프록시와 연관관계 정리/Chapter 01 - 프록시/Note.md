# 프록시
> 작성자: @luke0408

## 목차

- [프록시](#프록시)
  - [목차](#목차)
  - [Member를 조회할 때 Team도 함께 조회해야 하는가?](#member를-조회할-때-team도-함께-조회해야-하는가)
  - [프록시 기초](#프록시-기초)
    - [프록시 조회](#프록시-조회)
    - [프록시의 구조](#프록시의-구조)
    - [프록시 위임](#프록시-위임)
  - [프록시 객체의 초기화](#프록시-객체의-초기화)
    - [초기화 과정](#초기화-과정)
    - [프록시 초기화의 특징](#프록시-초기화의-특징)
  - [프록시 확인](#프록시-확인)
    - [프록시 인스턴스의 초기화 여부 확인](#프록시-인스턴스의-초기화-여부-확인)
    - [프록시 클래스 확인 방법](#프록시-클래스-확인-방법)
    - [프록시 강제 초기화](#프록시-강제-초기화)
  - [Q\&A](#qa)
    - [준영속 상태에서 프록시 초기화가 불가능한 이유](#준영속-상태에서-프록시-초기화가-불가능한-이유)
    - [상속관계에서 프록시를 사용할때 주의점은 없는가?](#상속관계에서-프록시를-사용할때-주의점은-없는가)

## Member를 조회할 때 Team도 함께 조회해야 하는가?

[프록시를 사용하지 않은 경우]
> Member만 조회하고 싶은데 Team도 함께 조회하는 문제가 발생한다.

- Member와 Team을 함께 출력

```java
public void printMemberAndTeam(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("member = " + member.getUsername());
    System.out.println("team = " + team.getName());
}
```

- Member만 출력

```java
public void printMember(String memberId) {
    Member member = em.find(Member.class, memberId);
    Team team = member.getTeam();
    System.out.println("member = " + member.getUsername());
}
```

## 프록시 기초

### 프록시 조회

**em.find()**
- 데이터베이스를 통해서 실제 엔티티 객체 조회
- 즉, `em.find()를 호출할 때` 데이터베이스를 통해서 실제 엔티티 객체를 조회한다.

```java
Member member = em.find(Member.class, memberId); // 이 시점에 쿼리를 날린다.
System.out.println("member.id = " + member.getId());
System.out.println("member.username = " + member.getUsername());
```

**em.getReference()**
- 데이터베이스 조회를 미루는 가짜(프록시) 엔티티 객체 조회
- 즉, `em.getReference()를 호출할 때` 데이터베이스 조회를 하는 것이 아닌, 데이터를 사용할 때 조회한다.

```java
Member member = em.getReference(Member.class, memberId); // 이 시점에는 쿼리를 날리지 않는다.
System.out.println("member.id = " + member.getId()); // 이 시점에 쿼리를 날린다.
System.out.println("member.username = " + member.getUsername()); // 위에서 가져온 데이터를 사용한다.
```

### 프록시의 구조

<center>
<img width="204" alt="Proxy 1-1" src="https://github.com/luke0408/study_for_jpa_basic/assets/98688494/51b5fb0f-219a-4b0b-8fd4-db95cfcc111f">
</center><br>

- 실제 클래스를 상속 받아서 만들어진다.
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다. (이론상)

### 프록시 위임

<center>
<img width="770" alt="Proxy 1-2" src="https://github.com/luke0408/study_for_jpa_basic/assets/98688494/cee6fb69-8849-4f0e-94d2-f911feeed620">
</center>

- 프록시 객체는 실제 객체의 참조(target)를 보관한다.
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출

## 프록시 객체의 초기화
> 프록시 객체 초기화란 프록시 객체가 실제 사용될 때 데이터베이스를 조회해서 `실제 엔티티 객체를 생성`하는 것

### 초기화 과정

```java
Member member = em.getReference(Member.class, memberId); // 프록시 객체를 조회한다.
member.getName(); // 프록시 객체를 초기화한다.
```

<center>
<img width="782" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/98688494/9c620cb1-4b02-4a6c-adae-15e78b52a454">
</center><br>

[과정 설명]

1. 프록시 객체에 member.getName()을 호출해서 실제 데이터를 조회한다.
2. 프록시 객체는 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트 실제 엔티티 생성을 요청한다.
3. 영속성 컨텍스트는 데이터베이스를 조회해서 실제 엔티티 객체를 생성한다.
4. 프록시 객체는 생성된 실제 엔티티 객체의 참조를 Member target 멤버변수에 보관한다.
5. 프록시 객체는 실제 엔티티 객체의 getName()을 호출해서 결과를 반환한다.

### 프록시 초기화의 특징

- 프록시 객체는 처음 사용할 때 한 번만 초기화 된다.

- 프록시 객체를 초기화 할 때, `프록시 객체가 실제 엔티티로 바뀌는 것은 아님`
  - 프록시 객체가 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
    - 1번과 2번 출력이 같은 클래스인 것을 확인할 수 있다.

    ```java
    Member member = em.getReference(Member.class, memberId); // 프록시 객체를 조회한다.
    System.out.println("before member class = " + member.getClass()); // 1번
    System.out.println("member.id = " + member.getId());
    System.out.println("after member class = " + member.getClass()); // 2번
    ```

- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함
  - (== 비교 실패, 대신 instance of 사용)

- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환
  - 왜? 같은 Transaction에서 a와 a는 같은 객체여야하기 때문에

    ```java
    Member member = new Member();
    member.setName("member");
    em.persist(member);

    em.flush();
    em.clear();

    //-----------------
    //영속성 컨텍스트에 찾는 엔티티가 있는 경우 엔티티를 반환한다.
    Member findMember = em.find(Member.class, member.getId());
    System.out.println(findMember.getClass()); //실제 엔티티 반환

    Member reference = em.getReference(Member.class, member.getId());
    System.out.println(reference.getClass()); //실제 엔티티 반환

    System.out.println("a == a : " + (findMember == reference)); //true

    em.clear();
    
    //-----------------
    //프록시가 이미 있는 경우 프록시를 반환한다.
    Member reference2 = em.getReference(Member.class, member.getId());
    System.out.println(reference2.getClass()); //프록시 반환

    Member findMember2 = em.find(Member.class, member.getId());
    System.out.println(findMember2.getClass()); //프록시 반환

    System.out.println("a == a : " + (reference2 == findMember2)); //true
    ```

- 영속성 컨텍스트의 도움을 받을 수 없는 `준영속 상태일 때, 프록시를 초기화`하면 문제 발생
  - 하이버네이트는 org.hibernate.LazyInitializationException 예외를 터트림
  - no Session 에러 발생 (= 영속성 컨텍스트에 없음)

    ```java
    Member member = em.getReference(Member.class, memberId); // 프록시 객체를 조회한다.
    em.detach(member); // 영속성 컨텍스트에서 분리한다.
    member.getName(); // 프록시 초기화 불가능 -> 예외 발생
    ```

## 프록시 확인

### 프록시 인스턴스의 초기화 여부 확인

- emf.getPersistenceUnitUtil().isLoaded(Object entity)
  - JPA 표준은 emf.getPersistenceUnitUtil().isLoaded(Object entity)로 확인
  - Hibernate는 Hibernate.isInitialized(Object entity)로 확인

```java
Member member = em.getReference(Member.class, memberId); // 프록시 객체를 조회한다.
System.out.println("isLoaded = " + emf.getPersistenceUnitUtil().isLoaded(member)); //false
member.getName(); // 프록시 초기화
System.out.println("isLoaded = " + emf.getPersistenceUnitUtil().isLoaded(member)); //true
```

### 프록시 클래스 확인 방법

- entity.getClass().getName()으로 확인

```java
Member member = em.getReference(Member.class, memberId); // 프록시 객체를 조회한다.
System.out.println("member.getClass().getName() = " + member.getClass().getName());
```

### 프록시 강제 초기화

- org.hibernate.Hibernate.initialize(entity)
  - JPA 표준은 강제 초기화 없음
  - Hibernate는 Hibernate.initialize(entity)로 강제 초기화 가능

```java
Member member = em.getReference(Member.class, memberId); // 프록시 객체를 조회한다.
Hibernate.initialize(member); // 강제 초기화
```

## Q&A

### 준영속 상태에서 프록시 초기화가 불가능한 이유

- 영속성 컨텍스트는 캐시를 가지고 있기 때문에 트랜잭션을 통해 DB에서 데이터를 가져오면 영속성 컨텍스트 내부 캐시에 저장된다. 이후 같은 엔티티를 조회할 때는 캐시에서 가져오기 때문에 DB에 다시 접근하지 않는다.
  - [프록시 초기회의 특징](#프록시-초기화의-특징) 중 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference()를 호출해도 실제 엔티티 반환하는 이유 중 하나이기도하다.

- 그러므로 엔티티의 값을 변경할 때는 영속성 컨텍스트 내부에 저장된 값을 변경해야하는데 준영속 상태에서는 영속성 컨텍스트에 접근할 수 없기 때문에 프록시 초기화가 불가능하다.

### 상속관계에서 프록시를 사용할때 주의점은 없는가?

[(3) 프록시 심화 주제](https://www.nowwatersblog.com/jpa/ch15/15-3#%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%951:jpql%EB%A1%9C%EB%8C%80%EC%83%81%EC%A7%81%EC%A0%91%EC%A1%B0%ED%9A%8C)
