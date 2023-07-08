# 애플리케이션 개발
> 작성자:밀리

## 목차

### JPA 구동방식
1. JPA가 시작되면 Persistence 클래스로 시작
2. Persistence 클래스에서 persistence.xml 파일을 읽는다.
3. EntityManagerFactory라는 클래스를 만들어서 필요할 때마다 EntityManager를 공장처럼 찍어낸다.

### JPA 엔티티 설계 시 주의할 점!!   
반드시 @Entity 어노테이션을 선언해줘야 한다.  
엔티티에 @Id 어노테이션을 설정하여 데이터베이스 Pk와 매핑해야 한다.   

### h2 database 연결하기   
h2 파일로 이동 후 다음 코드 작성
```
   cd bin
   ./h2.sh
```
### EntityManagerFactory & EntitiyManager 생성하기   
---
```
    EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");   
    EntityManager em = emf.createEntityManager();
```
** 어떤 행위(조회,생성 등)를 할 때 반드시 엔티티 매니저를 만들어줘야 된다. **  

### 객체 저장하기
``` em.persist(member); ```

### 객체 수정하기
``` member.setName("zzz"); ```
*** 객체 수정할 때는 persist() 할 필요가 없다. ***

### 트랜잭션
JPA에서 모든 데이터를 변경하는 작업은 트랜잭션 안에서 작업해야됨!!
EntityManager == JavaCollection → 데이터 대신 저장
JPA가 변경됐는지 안 됐는지 커밋 시점에 다 체크. 만약 변경됐으면 update쿼리 날림
** JPA의 모든 데이터 변경은 트랜잭션 안에서 실행 **

## JPA 사용 시 알아야할 점
엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에서 공유.
엔티티 매니저는 쓰레드간에 공유x(사용하고 버려야됨) 이유: 데이터베이스 커넥션처럼 바로 버려야됨

### JPQL이란(객체 지향 쿼리)? → 방언에 맞춰서 쿼리 변경해줌
- 가장 단순한 조회 방법(`em.find(`) 등)
- 엔티티 객체를 중심으로 개발
- 문제는 검색 쿼리
- SQL 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색
- 나이가 18살 이상인 회원을 모두 조회하고 싶다면? -> JPQL 사용하기
- SQL은 데이터베이스 테이블을 중심으로 하고 JPQL은 엔티티 객체를 중심으로 한다.
