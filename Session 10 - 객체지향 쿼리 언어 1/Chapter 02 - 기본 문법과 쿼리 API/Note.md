# 기본 문법과 쿼리 API
> 작성자: @asas6978

## 목차
- **JPQL 소개**
- **JPQL 문법**
- **TypeQuery, Query**
- **결과 조회 API**
- **파라미터 바인딩**
  

## JPQL 소개
- JPQL은 객체지향 쿼리 언어이므로, 테이블을 대상으로 쿼리하는 것이 아닌 엔티티 객체를 대상으로 쿼리
- JPQL은 SQL을 추상화하므로 특정 데이터베이스 SQL에 의존하지 않음
- JPQL은 결국 SQL로 변환
  

## JPQL 문법
<img width="300" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/ed6f8ec4-0ce8-4798-b75b-0bc84dd44023">

- 엔티티와 속성은 대소문자 구분
- JPQL 키워드는 대소문자 구분 X
- 테이블 이름이 아닌 엔티티 이름을 사용
- as 키워드는 생략 가능하나 별칭은 필수로 포함시켜야 함
  

## TypeQuery, Query
- TypeQuery - 반환 타입이 명확할 때 사용
  ```java
  TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
  ```
- Query - 반환 타입이 명확하지 않을 때 사용
  ```java
  Query query = em.createQuery("SELECT m.username, m.age from Member m");
  ```
  

## 결과 조회 API
- `query.getResultList()` - 결과가 하나 이상일 때 사용하며 리스트 반환
  - 결과가 없으면 빈 리스트 반환
- `query.getSingleResult()` - 결과가 정확히 하나일 때 사용하며 단일 객체 반환
  - 결과가 없으면 `javax.persistence.NoResultException`
  - 결과가 둘 이상이면 `javax.persistence.NonUniqueResultException`
    

## 파라미터 바인딩
- 이름 기준
  ```java
  SELECT m FROM Member m where m.username=:username
  query.setParameter("username", usernameParam);
  ```
- 위치 기준 (가능하면 사용하지 말 것, 나중에 코드를 수정하는 과정에서 문제가 될 가능성이 있음)
  ```java
  SELECT m FROM Member m where m.username=?1
  query.setParameter(1, usernameParam);
  ```
