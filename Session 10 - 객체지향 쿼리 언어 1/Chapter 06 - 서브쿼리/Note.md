# 서브쿼리
>
> 작성자: @luke0408

## 목차

- [서브쿼리](#서브쿼리)
  - [목차](#목차)
  - [서브 쿼리 지원 함수](#서브-쿼리-지원-함수)
    - [EXISTS](#exists)
    - [ALL](#all)
    - [ANY, SOME](#any-some)
    - [IN](#in)
  - [JPA 서브 쿼리 한계](#jpa-서브-쿼리-한계)
    - [하이버네이트에서 지원하는 서브 쿼리](#하이버네이트에서-지원하는-서브-쿼리)

## 서브 쿼리 지원 함수

- [Not] EXISTS (subquery): 서브쿼리에 결과가 존재하면 참
  - {ALL | ANY | SOME} (subquery) - 서브쿼리 조건과 비교
  - ALL: 모두 만족하면 참
  - ANY, SOME: 같은 의미, 조건을 하나라도 만족하면 참
- [NOT] IN (subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

### EXISTS

- 팀 A 소속인 회원 조회

  ```java
  select m from Member m
  where exists (select t from m.team t where t.name = '팀A')
  ```

### ALL

- 전체 상품 각각의 재고보다 주문량이 많은 주문들 조회

  ```java
  select o from Order o
  where o.orderAmount > ALL (select p.stockAmount from Product p)
  ```

### ANY, SOME

- 어떤 팀이든 팀에 소속된 회원

  ```java
  select m from Member m
  where m.team = ANY (select t from Team t) // SOME도 사용 가능
  ```

### IN

- 팀 A에 소속된 회원

  ```java
  select m from Member m
  where m.team in (select t from Team t where t.name = '팀A')
  ```

## JPA 서브 쿼리 한계

- JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- SELECT 절도 가능(하이버네이트에서 지원)
- **FROM 절의 서브 쿼리는 현재 JPQL에서 불가능**
  - 조인으로 풀 수 있으면 풀어서 해결
  - 하이버네이트 6 부터는 지원

### 하이버네이트에서 지원하는 서브 쿼리

- FROM 절의 서브 쿼리
  - 하이버네이트는 SELECT 절의 서브 쿼리도 지원
  - 하이버네이트6 부터만 사용 가능한 문법

- 예시

  ```java
  String query = "select mm.age, mm.username from (select m.age, m.username from Member m) as mm";
  List<Member> resultList = em.createQuery(query, Member.class)
          .getResultList();
  ```
