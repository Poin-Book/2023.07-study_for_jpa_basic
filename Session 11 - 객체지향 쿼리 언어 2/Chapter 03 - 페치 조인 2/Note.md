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
  - 팀을 조회하는데 맴버만 따로 조회해서 조작한다는 것 자체가 위험한 행위, JPA의 그래프 탐색의 의도는 연관된 모든 데이터를 가지고 오는 것이다.
  - 만일 Members에서 특정한 member만 필터로 조회해 가져오고 싶다면 Team과 Member의 Fetch 조인이 아닌, Member에서 조회하는 방식으로 해결해야 한다.
    ```java
    select t From Team t join fetch t.memberList m; // -> x
    select m From Member m join m.team where m.age > 10; // -> o
    ```
  - 특정 조건의 데이터만 조작하는 경우에도, 해당되지 않는 나머지 데이터들이 변경되거나 제거될 수 있어 데이터 정합성 문제가 발생한다. 
    ex ) CASCADE와 같은 설정이 걸린 경우
  - 또 같은 엔티티의 컬렉션을 한 쪽에서는 전체 조회하고, 한쪽에서는 필터를 통해 10개만 가져왔을때, 영속성 컨텍스트 입장에서는 같은 엔티티의 컬렉션의 데이터가 다르기에 애매한 입장이 된다.
  - 예외로 사용하게 되는 때는 A 와 B의 Fetch 조인 B와 C의 fetch 조인 이런식으로 사용할 때도 있다고 하지만, 정합성 이슈 때문에 거의 사용하지 않는다고 합니다.

### ❗️ 둘 이상의 컬렉션은 페치 조인 할 수 없다.
  - 일대다의 경우 데이터 뻥튀기가 발생되는데,  둘 이상의 컬렉션에 패치 조인을 사용하게 되면 데이터 정합성에 문제가 발생할 수 있다.
![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/1aa47d3c-9310-40d3-b9b3-fb561aae1432)
    
### ❗️ 컬렉션을 페치 조인하면 페이징 API(setFirstResult, setMaxResults)를 사용할 수 없다.
  - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인해도 페이징 가능
  - 일대다, 다대다 관계에서는 데이터 뻥튀기 현상이 발생 할 수 있어 페이징 처리를 하면 정합성을 보장할 수 없다.
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
