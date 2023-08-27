# 페치 조인 2 - 한계
> 작성자: @joowojr

## 목차
### - 페치 조인의 특징과 한계 (1)
### - 페치 조인의 특징과 한계 (2)
### - 페치 조인 정리

## 1. 페치 조인의 특징과 한계
### ❗️ 페치 조인 대상에는 별칭을 줄 수 없다.(별칭 사용하지 않는게 관례)
  - 하이버네이트는 가능하나, 가급적 사용X
    ```java
    String query = "select t From Team t join fetch t.members m";​에서 m을 사용하면 안됨.
    ```
  - 페치 조인 결과는 연관된 모든 엔티티가 있을 것이라 가정하고 사용해야 하는데 이를 어기게 된다. (객체지향과 DB의 불협화음. orm 매핑, 일관성이 깨진다고 보면 된다.)
    - 일대다의 경우
      - 페치 조인의 주인(Team)에게 필터링을 거는 것은 상관이 없다. 
      - 페치 조인의 대상(Member)에게 사용되는 방식은 에러가 발생하지는 않지만 지양되고 있다.
    - 다대일의 경우에는 대상에게 WHERE절 별칭을 걸어도 괜찮다.
      - 조회된 회원은 DB와 동일한 일관성을 유지한 Team 데이터를 갖고 있기 때문에 Member와 Team의 일관성을 해치지 않기 때문이다.
      - 하지만 left join fetch로 되면 일관성이 깨진다. (Team이 null이 아닌 Member에 대해서 null 값이 들어가기 때문이다.)
  - 만일 Members에서 특정한 member만 필터로 조회해 가져오고 싶다면 Team과 Member의 Fetch 조인이 아닌, Member에서 조회하는 방식으로 해결해야 한다.
    ```java
    select t From Team t join fetch t.memberList m; // -> x
    select m From Member m join m.team where m.age > 10; // -> o
    ```
  - 특정 조건의 데이터만 조작하는 경우에도, 해당되지 않는 나머지 데이터들이 변경되거나 제거될 수 있어 데이터 정합성 문제가 발생한다. 
    ex ) CASCADE와 같은 설정이 걸린 경우
  - 별칭이 필요한 유일한 때는 다음과 같이 추가 컬렉션을 재귀적으로 갖고올 때 뿐이다. 그 외는 x 
    ```sql
    from Team t
     inner join fetch t.members
     left join fetch t.members m
     left join fetch m.~
    ```

### ❗️ 둘 이상의 컬렉션은 페치 조인 할 수 없다.
  - 일대다의 경우 데이터 뻥튀기가 발생되는데,  둘 이상의 컬렉션에 패치 조인을 사용하게 되면 데이터 정합성에 문제가 발생할 수 있다.
    ```java
    String query = "select t From t join fetch t.members join fetch t.orders";
     ```
    - 둘 이상의 페치 조인을 이용해서 [1 : N : N] 형태로 쿼리를 조회하면 데이터가 뻥튀기 뿐 아니라 정합성 맞추기가 쉽지 않다.
![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/1aa47d3c-9310-40d3-b9b3-fb561aae1432)
    
### ❗️ 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
  - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
  - 일대다, 다대다 관계에서는 데이터 뻥튀기 현상이 발생 할 수 있어 페이징 처리를 하면 정합성을 보장할 수 없다.
    - 데이터가 적을때는 상관없지만 데이터가 많을 때 문제가 된다.
      ex) Team이 100만개가 있으면, 100만개를 모두 메모리에 적재한 이후에 페이지를 조회하게 된다. 
    ```log
    Hibernate: 
    /* select
        t 
    From
        Team t 
    join
        fetch t.members m */ select
            team0_.id as id1_5_0_,
            members1_.id as id1_3_1_,
            team0_.name as name2_5_0_,
            members1_.age as age2_3_1_,
            members1_.city as city3_3_1_,
            members1_.street as street4_3_1_,
            members1_.zipcode as zipcode5_3_1_,
            members1_.name as name6_3_1_,
            members1_.TEAM_ID as TEAM_ID9_3_1_,
            members1_.endDate as endDate7_3_1_,
            members1_.startDate as startDat8_3_1_,
            members1_.TEAM_ID as TEAM_ID9_3_0__,
            members1_.id as id1_3_0__ 
        from
            Team team0_ 
        inner join
            Member members1_ 
                on team0_.id=members1_.TEAM_ID
    
    teamName=팀A,members=2
    -> member = Member{id=3, name='회원1', age=0, team=Team{id=1, name='팀A'}}
    -> member = Member{id=4, name='회원2', age=0, team=Team{id=1, name='팀A'}}
    teamName=팀B,members=1
    -> member = Member{id=5, name='회원3', age=0, team=Team{id=2, name='팀B'}}
    ```
    - join fetch와 distinct를 사용하면 가공한 결과값에서 페이징을 처리하기 때문에 팀A에 회원 1과 회원2 총 2명이 있다해도, 회원1만 있는것처럼 조회한다.
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/f788089a-2c0f-4107-88b9-9b84e4aa343f)
  - 하이버네이트는 경고 로그를 남기고 메모리에서 페이징(매우 위험)
    ```java
    String query = "select t From Team t join fetch t.members m";
    
    List<Team> teamList = em.createQuery(query, Team.class)
                                .setFirstResult(0)
                                .setMaxResults(1)
                                .getResultList();
    ```
    ```log
    org.hibernate.hql.internal.ast.QueryTranslatorImpl list
    WARN: HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
    ```
  - 해결 방안
  1) 멤버 엔티티로 팀을 조회
      ```java
      String query = "select m From JpqlMember m join fetch m.team t";
      List<JpqlTeam> teamList = em.createQuery(query, JpqlTeam.class)
                                              .setFirstResult(0)
                                              .setMaxResults(1)
                                              .getResultList();
      ```
  2) fetch join을 제거하고, @BetchSize 어노테이션 사용 (N+1문제 x)
      > @ BatchSize
      - 여러 개의 프록시 객체를 조회할 때 WHERE 절이 같은 여러 개의 SELECT 쿼리들을 하나의 IN 쿼리로 만들어준다.
      ![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/1bd4eb44-93fd-457d-8045-1e5a699a03ff)

      - class, field, application 파일에 적용 가능
      - size는 IN 절에 들어갈 요소의 최대 갯수, 이 갯수가 넘어가게 되면 여러개의 IN 쿼리로 나누어 날린다.
      ```java
      @Entity
      public class Team {
          ...
          @BatchSize(size = 100)
          @OneToMany(mappedBy = "team")
          private List<lMember> memberList = new ArrayList<>();
          ...
      }
      ```
      ❗️상황에 따라 배치 사이즈를 다르게 설정해야 한다
        
        https://velog.io/@joonghyun/SpringBoot-JPA-JPA-Batch-Size%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B3%A0%EC%B0%B0
## 2. 페치 조인의 특징과 한계
- fetch join은 연관된 엔티티들을 SQL 한 번으로 조회 - 성능 최적화.
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함.
- @OneToMany(fetch = FetchType.LAZY) // 글로벌 로딩 전략
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩.
- 최적화가 필요한 곳은 페치 조인 적용.
  - 즉, N + 1 문제가 발생하는 곳에만 페치 조인을 적용 한다.

## 3. 페치 조인 정리
- 모든 것을 페치 조인으로 해결할 수 는 없다.
- 페치 조인은 객체 그래프를 유지할 때 사용하면 효과적이다.
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, 페치 조인 보다는 일반 조인을 사용하고 필요한 데이터만 조회해서 DTO로 반환 하는것이 효과적이다.
