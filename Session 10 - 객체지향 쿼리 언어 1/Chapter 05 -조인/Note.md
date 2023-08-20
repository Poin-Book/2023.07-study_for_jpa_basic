# 조인
>
> 작성자: @luke0408

## 목차

- [조인](#조인)
  - [목차](#목차)
  - [조인의 종류](#조인의-종류)
    - [예제 도메인 모델](#예제-도메인-모델)
    - [내부(inner) 조인](#내부inner-조인)
    - [외부(outer) 조인](#외부outer-조인)
    - [세타 조인](#세타-조인)
  - [조인 - ON 절](#조인---on-절)
    - [1. 조인 대상 필터링](#1-조인-대상-필터링)
    - [2. 연관관계 없는 엔티티 외부 조인](#2-연관관계-없는-엔티티-외부-조인)

## 조인의 종류

### 예제 도메인 모델

- Member
  - id
  - username
  - age
  - team
  
  ```java
  @Entity
  public class Member {
      @Id @GeneratedValue
      private Long id;
      private String username;
      private int age;

      @ManyToOne(fetch = FetchType.LAZY)
      @JoinColumn(name = "team_id")
      private Team team;
  }
  ```

- Team
  - id
  - name
  
  ```java
  @Entity
  public class Team {
      @Id @GeneratedValue
      private Long id;
      private String name;
  }
  ```

### 내부(inner) 조인

- SQL 예시

  ```sql
  SELECT m
  FROM Member m
  INNER JOIN m.team t
  ```

- JPQL 예시

  ```java
  String query = "select m from Member m inner join m.team t";
  List<Member> resultList = em.createQuery(query, Member.class)
          .getResultList();
  ```

### 외부(outer) 조인

- SQL 예시

  ```sql
  SELECT m
  FROM Member m
  Left Join m.team t
  ```

- JPQL 예시

  ```java
  String query = "select m from Member m left join m.team t";
  List<Member> resultList = em.createQuery(query, Member.class)
          .getResultList();
  ```

### 세타 조인

- SQL 예시

  ```sql
  SELECT count(m) 
  FROM Member m, Team t 
  WHERE m.username = t.name
  ```

- JPQL 예시

  ```java
  String query = "select count(m) from Member m, Team t where m.username = t.name";
  List<Member> resultList = em.createQuery(query, Member.class)
          .getResultList();
  ```

## 조인 - ON 절

- ON 절을 활용한 조인(JPA 2.1부터 지원)
  - ON 절을 활용한 조인은 연관관계가 없는 엔티티를 외부 조인할 때 사용

### 1. 조인 대상 필터링

- 예시
  - 회원과 팀을 조인하면서, 팀 이름이 A인 팀만 조인
  - 회원은 모두 조회

- SQL 예시

  ```sql
  SELECT m, t
  FROM Member m 
  LEFT JOIN m.team t ON t.name = 'A'
  ```

- JPQL 예시

  ```java
  String query = "select m, t from Member m left join m.team t on t.name = 'A'";
  List<Member> resultList = em.createQuery(query, Member.class)
          .getResultList();
  ```

### 2. 연관관계 없는 엔티티 외부 조인

- 예시
  - 회원의 이름과 팀의 이름이 같은 대상 외부 조인
  - 회원은 모두 조회

- SQL 예시

  ```sql
  SELECT m, t
  FROM Member m
  LEFT JOIN Team t 
  ON m.time_id = t.id AND t.name = 'A'
  ```

- JPQL 예시

  ```java
  String query = "select m, t from Member m left join Team t on m.time_id = t.id and t.name = 'A'";
  List<Member> resultList = em.createQuery(query, Member.class)
          .getResultList();
  ```
  