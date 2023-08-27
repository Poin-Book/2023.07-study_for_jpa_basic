# 엔티티 직접 사용
>
> 작성자: @luke0408

## 목차

- [엔티티 직접 사용](#엔티티-직접-사용)
  - [목차](#목차)
  - [기본 키 값](#기본-키-값)
    - [JPQL 쿼리 작성](#jpql-쿼리-작성)
    - [파라미터 \& 식별자](#파라미터--식별자)
  - [외래 키 값](#외래-키-값)
    - [파라미터 \& 식별자](#파라미터--식별자-1)

## 기본 키 값

- JPQL에서 엔티티를 직접 사용하면 SQL에서 해당 엔티티의 기본 키 값을 사용한다.

### JPQL 쿼리 작성

[JPQL]
  
- 엔티티의 아이디를 사용

  ```java
  String query = "select count(m.id) from Member m";
  Long result = em.createQuery(query, Long.class).getSingleResult();
  ```

- 엔티티를 직접 사용

  ```java
  String query = "select count(m) from Member m";
  Long result = em.createQuery(query, Long.class).getSingleResult();
  ```
  
[SQL] (둘다 동일한 쿼리가 나간다.)

  ```sql
  select count(m.id) as cnt from Member m
  ```

### 파라미터 & 식별자

[파라미터 사용]
  
  ```java
  String query = "select m from Member m where m = :member";
  List<Member> result = em.createQuery(query, Member.class)
      .setParameter("member", member)
      .getResultList();
  ```

[식별자 직접 전달]
  
  ```java
  String query = "select m from Member m where m.id = :memberId";
  List<Member> result = em.createQuery(query, Member.class)
      .setParameter("memberId", memberId)
      .getResultList();
  ```

[SQL]

  ```sql
  select m.* from Member m where m.id = ?
  ```

## 외래 키 값

### 파라미터 & 식별자

[파라미터 사용]
  
  ```java
  String query = "select m from Member m where m.team = :team";
  List<Member> result = em.createQuery(query, Member.class)
      .setParameter("team", team)
      .getResultList();
  ```

[식별자 직접 전달]
  
  ```java
  String query = "select m from Member m where m.team.id = :teamId";
  List<Member> result = em.createQuery(query, Member.class)
      .setParameter("teamId", teamId)
      .getResultList();
  ```

[SQL]

  ```sql
  select m.* from Member m where m.team_id = ?
  ```
