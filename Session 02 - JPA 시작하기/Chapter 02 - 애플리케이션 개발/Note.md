# 애플리케이션 개발
> 작성자:밀리

## 목차
[JPA 구동방식](JPA_구동방식)  

[JPA 엔티티 설계](JPA_엔티티_설계)  

[h2 database 연결하기](h2_database_연결하기)  

[애플리케이션 개발](애플리케이션_개발)  

[JPA에서 객체 저장, 수정 삭제하기](JPA에서_객체_저장,_수정_삭제하기)  

[트랜잭션](트랜잭션)  

[JPA 사용 시 알아야할 점](JPA_사용_시_알아야할_점)  

[JPQL](JPQL)

## JPA 구동방식
1. JPA가 시작되면 Persistence 클래스로 시작한다. 
2. Persistence 클래스에서 META-INF/persistence.xml에 있는 정보를 바탕으로 EntityManagerFactory를 생성한다.
3. EntityManagerFactory라는 클래스를 만들어서 필요할 때마다 EntityManager를 공장처럼 찍어낸다.
   

## JPA 엔티티 설계   
- 반드시 @Entity 어노테이션을 선언해줘야 한다.  
- 엔티티에 @Id 어노테이션을 설정하여 데이터베이스 Pk와 매핑해야 한다.
#### 실제 예시
```
@Entity
@Getter
@Setter
public class Member {
    @Id
    private Long id;
    private String name;

    public Member(Long id, String name){
        this.id = id;
        this.name = name;
    }
}
```
## h2 database 연결하기   
h2 파일로 이동 후 다음 코드 작성
```
   cd bin
   ./h2.sh
```
## 애플리케이션 개발 
```
package hellojpa;
 
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
 
public class JpaMain {
 
    public static void main(String[] args) {
        // 엔티티 매니저 팩토리 생성
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");
        // 엔티티 매니저 생성
        EntityManager em = emf.createEntityManager();
        //트랜잭션 획득
        EntityTransaction tx = em.getTransaction();
        tx.begin();
 
        try {
            tx.begin(); //트랜잭션 시작
            tx.commit(); //트랜잭션 커밋
        } catch (Exception e) {
            tx.rollback(); //트랜잭션 롤백
        } finally {
            em.close(); // 엔티티 매니저 종료
        }
        emf.close(); // 엔티티 매니저 팩토리 종료
        
        
    }
    
```
1) 엔티티 매니저 설정

JPA를 시작하려면 일단 엔티티 매니저 팩토리를 생성해야 한다. 이때 Persistence 클래스를 사용하게 된다.

```EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");```
 

- 이때 persistence.xml의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체에 따라서 데이터베이스 커넥션 풀도 생성하므로 엔티티 매니저 팩토리를 생성하는 비용은 아주 크다.

- 따라서 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.

 

2) 엔티티 매니저 생성

```EntityManager em = emf.createEntityManager();```  

- 엔티티 매니저 팩토리에서 엔티티 매니저를 생성한다. JPA의 기능 대부분은 이 엔티티 매니저가 제공한다.  

- 엔티티 매니저를 사용해서 엔티티를 데이터베이스에 CRUD 할 수 있고, 내부에 데이터베이스 커넥션을 유지하면서 데이터베이스와 통신한다. 즉, 가상의 데이터베이스 역할을 하는 셈이다.  

- 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드 간에 공유하거나 재사용하면 안된다.  

 

3) 종료
```
em.close();
emf.close();
```
- 사용이 끝난 엔티티 매니저와 팩토리는 반드시 종료해야 한다.  
**어떤 행위(조회,생성 등)를 할 때 반드시 엔티티 매니저를 만들어줘야 된다.**  

## JPA에서 객체 저장, 수정 삭제하기
```
// 1. 저장
jpa.persist(member);
 
// 2. 조회
String memberId = "suhwan";
Member member = jpa.find(Member.class, memberId);
 
// 3. 수정
Member member = jpa.find(Member.class, memberId);
member.setName("babo");
 
// 4. 연관된 객체 조회
Member member = jpa.find(Member.class, memberId);
```
**객체 수정할 때는 persist() 할 필요가 없다.**

## 트랜잭션
> 트랜잭션이란 질의(query)를 하나의 묶음 처리해서 만약 중간에 실행이 중단되었을 경우,
처음부터 다시 실행하는 Rollback을 수행하고, 오류없이 실행을 마치면 commit을 하는 실행 단위를 의미
<details>
<summary>트랜잭션 예시</summary>
<div markdown="1">       

- 친구에게 10,000원을 송금하는 상황을 가정.
- 내가 친구에게 송금한다면, 나의 계좌에서 10,000원을 차감하고 친구의 계좌에 10,000원을 증가시켜야 하는데, 알 수 없는 오류로 인해 나의 계좌에서는 10,000원이 줄었지만 친구 계좌에서는 10,000원이 증가 되지 않았다.
- 이렇게 된다면 어떻게 해야할까?
- 이러한 경우가 생기지 않도록 중간에 오류가 발생하면 다시 처음부터 송금을 하도록 하는 것이 rollback.
- 오류 없이 정상적으로 송금이 되었다면 정상적으로 실행이 끝났으므로 commit을 한다.
-> 이렇게 송금 과정을 하나의 트랜잭션이라고 볼 수 있다.
</div>
</details>
- 즉, 한 번 질의가 실행되면 질의가 모두 수행되거나 모두 수행되지 않는 직업수행의 논리적 단위  

- JPA에서 모든 데이터를 변경하는 작업은 반드시 **트랜잭션 안에서** 작업해야된다.
  
- EntityManager는 JavaCollection과 비슷한 개념으로 데이터를 대신 저장하는 역할을 한다.
  
- JPA가 변경됐는지 여부는 트랜잭션 커밋 시점에 다 체크한다. 만약 변경됐으면 update쿼리를 날린다.

**JPA의 모든 데이터 변경은 트랜잭션 안에서 실행**


# JPA 사용 시 알아야할 점
- 엔티티 매니저 팩토리는 말 그대로 공장을 짓는다고 생각하면 되는데, 그만큼 비용이 많이 든다는 의미이다.  
  따라서 애플리케이션 전체에서 하나만 만들어 공유하도록 설계되어 있다. 대신 여러 스레드가 동시에 접근해도 안전해 서로 다른 스레드 간 공유가 가능하다.    
- 엔티티 매니저는 동시에 접근하면 동시성 문제가 발생하므로 쓰레드간에 공유하지 않고 DB 커넥션처럼 사용하고 바로 버려야된다. 


## JPQL
> SQL을 추상화한 객체지향 쿼리 언어이고 데이터베이스 방언에 맞춰서 쿼리를 변경해준다.
- 가장 단순한 조회 방법(`em.find(`) 등)
- SQL은 데이터베이스 테이블을 중심으로 하고 JPQL은 엔티티 객체를 중심으로 한다.
- SQL과 문법이 거의 유사하며 SELECT, FROM, WHERE 뿐만 아니라 GROUP BY, HAVING 등을 사용할 수 있다.
- 문제는 검색 쿼리?
   - 테이블이 아닌 엔티티 객체를 대상으로 검색해야 한다. 그런데 검색 쿼리에 조건을 달아서 검색하려면 데이터베이스의 모든 데이터를
   - 애플리케이션으로 불러와 객체를 변경한 다음 검색해야 하는데, 이는 사실상 불가능하다.
   - 그래서 검색 조건이 포함된 SQL을 사용해야만 하는 상황이 발생하고, JPA는 JPQL이라는 쿼리 언어로 이런 문제를 해결한다.
- SQL 추상화해서 특정 데이터베이스 SQL에 의존하지 않는다.
- 즉, 검색을 할 때도 테이블이 아닌 엔티티 객체를 대상으로 검색한다.
