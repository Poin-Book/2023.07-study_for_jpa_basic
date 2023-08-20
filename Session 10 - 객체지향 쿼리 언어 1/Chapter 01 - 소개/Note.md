# 소개
> 작성자: @Hyunstone

## 목차
 - [JPQL](#jpql-소개)
 - [JPA Criteria](#criteria-소개)
 - [QueryDSL](#querydsl-소개)
 - [Native SQL](#네이티브-sql-소개)
 - [JDBC 직접 사용, SpringJdbcTemplate 등](#jdbc-직접-사용-springjdbctemplate-등)

## JPA는 다양한 쿼리 방법을 지원
- JPQL
- JPA Criteria
- QueryDSL
- 네이티브 SQL
- JDBC API 직접 사용, MyBatis, SpringJdbcTemplate 함께 사용

JPA Criteria, QueryDSL은 자바 코드로 짠 코드를 JPQL를 만들어서 빌드를 해주는 제너레이터. 데이터베이스에 종속적인 코드는 네이티브 SQL을 써야 한다. 대부분은 JPQL로 해결 가능하지만 일부는 네이티브 쿼리나 JDBC, MyBatis 등의 방법을 써서 해결해야 하는 경우가 있을 수 있다.

## JPQL 소개

- 가장 단순한 조회 방법
    - EntityManager.find()
    - 객체 그래프 탐색(a.getB().getC())
- 나이가 18살 이상인 회원을 모두 검색하고 싶다면? → JPQL

### JPQL

- JPA를 사용하면 엔티티 객체를 중심으로 개발
- 문제는 검색 쿼리
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
- 모든 DB 데이터를 객체로 변환해서 검색하는 것은 불가능
- 애플리케이션이 필요한 데이터만 DB에서 불러오려면 결국 검색 조건이 포함된 SQL이 필요
- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- JPQL을 한마디로 정의하면 객체 지향 SQL

특정 조건에 따라서 쿼리가 바껴야 하는, 동적쿼리를 만드는 경우가 어렵다. String으로 다루는 경우 띄어쓰기, 쉼표, 문자 등의 신경을 써야 하는 부분이 많아 버그가 많이 생기는 부분이다.

이런 동적쿼리, 여러 부분 등에서 이점이 있어 Criteria를 사용하게 됨. 자바 표준 스펙에 존재하는 방식.

## Criteria 소개

- 문자가 아닌 자바코드로 JPQL을 작성할 수 있음 → 오타를 컴파일 오류로 잡을 수 있다
- JPQL 빌더 역할
- JPA 공식 기능
- **단점: 너무 복잡하고 실용성이 없다. SQL스럽지 않다**
- Criteria 대신에 **QueryDSL** 사용 권장

영한 선생님은 유지보수에서 어려움을 느껴 실무에서 사용을 하지 않는다고 말씀하셨습니다.

## QueryDSL 소개

- 문자가 아닌 자바코드로 JPQL을 작성할 수 있음
- JPQL 빌더 역할
- 컴파일 시점에 문법 오류를 찾을 수 있음
- 동적쿼리 작성 편리함
- **단순하고 쉬움**
- **실무 사용 권장**

JPQL 문법만 잘 사용하면 공식문서가 잘 되어있어 JPQL을 잘만 사용하면 손쉽게 사용이 가능합니다.

## 네이티브 SQL 소개

- JPA가 제공하는 SQL을 직접 사용하는 기능
- JPQL로 해결할 수 없는 특정 데이터베이스에 의존적인 기능
- 예) 오라클 CONNECT BY, 특정 DB만 사용하는 SQL 힌트

```java
String sql = “SELECT ID, AGE, TEAM_ID, NAME FROM MEMBER WHERE NAME = ‘kim’"; 
List<Member> resultList = em.createNativeQuery(sql, Member.class).getResultList();
```

## JDBC 직접 사용, SpringJdbcTemplate 등

- JPA를 사용하면서 JDBC 커넥션을 직접 사용하거나, 스프링 JdbcTemplate, 마이바티스등을 함께 사용 가능
- 단 영속성 컨텍스트를 적절한 시점에 강제로 플러시 필요
- 예) JPA를 우회해서 SQL을 실행하기 직전에 영속성 컨텍스트
수동 플러시

영한 선생님은 네이티브 쿼리보다는 스프링 JdbcTemplate을 사용하는게 더 편하다고 하셨습니다.

JPQL은 기본적으로 쿼리가 날라갈 때 내부적으로 플러시가 동작을 함. 그런데 위의 경우는 X. 강제로 플러시 해야하기에 주의해야 합니다.