# 벌크 연산
> 작성자: @asas6978

## 목차
- [벌크 연산](#벌크-연산)
- [벌크 연산 예제](#벌크-연산-예제)
- [벌크 연산 주의](#벌크-연산-주의)


## 벌크 연산
재고가 10개 미만인 모든 상품의 가격을 10% 상승하는 상황이 있다고 가정했을 때, JPA 변경 감지 기능으로 해당 기능을 실행하려면 다음과 같은 과정을 거친다.
1. 재고가 10개 미만인 상품을 리스트로 조회
2. 상품 엔티티의 가격을 10% 증가 
3. 트랜잭션 커밋 시점에 변경감지 동작

만약 변경된 데이터가 100건이라면 100번의 UPDATE SQL을 실행하게 되고, 이러한 연산을 보다 효율적으로 수행하기 위해 벌크 연산을 이용할 수 있다.

## 벌크 연산 예제
- 쿼리 한 번으로 여러 테이블의 row를 변경
- executeUpdate()의 실행 결과로 영향을 받은 엔티티 수를 반환

```java
String qlString = "update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount";
int resultCount = em.createQuery(qlString)
                    .setParameter("stockAmount", 10)
                    .executeUpdate();
```

## 벌크 연산 주의
```java
Member member1 = new Member();
member1.setAge(0);
em.persist(member1);

int resultCount = em.createQuery("update Member m set m.age = 20")
                    .executeUpdate();
System.out.println("member1.getAge() = " + member1.getAge()); //member1.getAge() = 0
```
- 벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리 -> 데이터 정합성에 맞지 않을 수 있음
  - 방법 1) 벌크 연산을 먼저 실행
  - 방법 2) 벌크 연산 수행 후 영속성 컨텍스트 초기화
