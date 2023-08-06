# 즉시 로딩과 지연 로딩
> 작성자: @joowojr

## 목차
1. 즉시 로딩과 지연 로딩
2. 프록시와 즉시 로딩 주의
3. 지연 로딩 활용

## 1. 즉시로딩과 지연 로딩
### Member를 조회할 때 Team도 함께 조회 해야 할까?
> 비즈니스 로직에서 단순히 멤버 로직만 사용하는데 함께 조회하면, 아무리 연관관계가 걸려있다고 해도 손해이다. 

>JPA는 이 문제를 지연로딩 LAZY를 사용해서 프록시로 조회하는 방법으로 해결 한다.

## 지연 로딩 (LAZY)
![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/2ee30456-3b1b-4bb2-8993-2a58d1f36326)
- 로딩되는 시점에 Lazy 로딩 설정이 되어있는 Team 엔티티는 프록시 객체로 가져온다.
- 후에 실제 객체를 사용하는 시점에 (Team을 사용하는 시점에) 초기화가 된다. DB에 쿼리가 나간다.
- getTeam()으로 Team을 조회하면 프록시 객체가 조회된다.
```java
  Member member = em.find(Member.class, 멤버 아이디); → member 정보만 가져옴
  member.getTeam().getName() → 프록시 개체 초기화, 이 때 team 정보 가져옴
```
## 즉시 로딩 (EAGER)
![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/cbe5ab46-3a93-4397-a171-e696a0e9c5cc)
- fetch 타입을 EAGER로 설정한다.
- 즉시 로딩 실행 시,실제 조회할 때 한방 쿼리로 다 조회해온다.(Member을 조회할 때 다 조회되므로, Team을 사용할 때 쿼리 안나가도 된다.)
- 실행 결과를 보면 Team 객체도 프록시 객체가 아니라 실제 객체이다.

### ✔︎ member를 사용 할 때, team의 정보도 자주 사용한다면 ? → 즉시 로딩
### ✔︎ member의 정보만 필요할 때가 많다면? → 지연 로딩


## 2. 프록시와 즉시로딩 주의
- 가급적 지연로딩만 사용 ( 특히 실무에서 ) 
- 즉시 로딩을 사용하면 예상치 못한 SQL 발생
  - 조인해서 정보를 가져오지 않더라도, 참조하는 테이블에 대한 조인 쿼리가 발생함
- 즉시로딩은 JPQL에서 N+1 문제를 일으킨다.
  -  쿼리를 1개 날렸는데, 그것 때문에 추가 쿼리가 N개 나간다는 의미이다.
  -  member 테이블을 조회하고 싶을 때
        ```sql
        select m from Member m
        ```
        - 즉시로딩 실행
        
          1.member 조회 쿼리
        
          2.member의 team 필드 보고 team 조회 쿼리→ 쿼리 두 방으로 인해 성능 저하
    
    
        - 지연 로딩 실행
          
          1.member 조회 쿼리
            ( team이 필요할 때 team 조회 쿼리 )
    -  Member의 수가 수천명, 수만명으로 가면 성능에 큰 영향을 미치개 된다.

## 즉시 로딩이 필요하다면
> 일단 모든 로딩을 지연 로딩으로 설정하고, 즉시 로딩을 원하는 테이블만 아래와 같이 join fetch 사용
```sql
select m from Member m join fetch m.team
```
- @ManyToOne, @OneToOne은 기본이 즉시 로딩이기 때문에, LAZY로 설정해줘야함
![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/87db6cd1-ebd6-47f6-92e3-fcea9c633cdf)

- @OneToMany, @ManyToMany는 기본이 지연 로딩

## 3. 지연 로딩 활용 - 실무
- ❗️모든 연관관계에 지연 로딩을 사용해라 ❗️
- ❗️ 실무에서 즉시 로딩을 사용하지 마라 ❗️
- JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라 
- 즉시 로딩은 사용하지 못한 쿼리가 나간다.


