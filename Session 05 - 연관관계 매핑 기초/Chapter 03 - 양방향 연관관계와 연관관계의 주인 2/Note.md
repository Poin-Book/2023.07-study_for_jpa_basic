# 양방향 연관관계와 연관관계의 주인 2 - 주의점, 정리
> 작성자: @joowojr

## 목차
1. 양방향 매핑 시 가장 많이 하는 실수
2. 양방향 매핑 정리

## 왜 연관 관계의 주인을 지정해야하는가?
- Member 와 Team이 양방향 연관 관계를 갖는다.
- Member을 원래의 Team과 다른 Team으로 수정하려 할 때, Member 객체에서 수정해야 할지 혹은 Team 객체에서 수정해야 할지 헷갈릴 수 있다.
- 즉, Team에서 Member를 수정할 때 FK(Foreign Key)를 수정할 지, Team에서 Member를 수정할 때 FK(Foreign Key)를 수정할 지를 결정하기 어려운 것이다.
-  이렇게 객체에서 양방향 연관 관계 관리 포인트가 두 개일 때는 테이블과 매핑을 담당하는 JPA입장에서 혼란을 주게 된다.
- 그렇기 때문에 두 객체 사이의 연관 관계의 주인을 정해서 명확하게 Member에서 Team을 수정할 때만 FK를 수정하겠다라고 정하는 것이다.

## 1. 양방향 매핑시 가장 많이 하는 실수
### 연관관계의 주인에 값을 입력하지 않음
- 역방향 ( 주인이 아닌 방향 )만 연관관계 설정
- Team의 members는 @mappedBy를 했기 때문에 읽기 전용으로 바뀐다.
- 역방향(주인이 아닌 방향)만 연관관계를 설정했기 때문에 Member 테이블의 TEAM_ID는 null이 들어간다.

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

//역방향(주인이 아닌 방향)만 연관관계 설정
team.getMembers().add(member);

em.persist(member);
```


- 순수한 객체 관계를 고려하면 항상 양쪽 다 값을 입력해야 한다.
    - flush와 clear을 해주어야 제대로 조회된다.
    - 영속성 컨텍스트를 초기화 하지않으면 1차 캐시에 들어 있는 것이 조회돼서 반환됨
    
```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("Member1");
member.setTeam(team);
em.persist(member);

team.getMembers().add(member);

em.flush();
em.clear();

Team findTeam = em.find(Team.class, team.getId());
List<Member> members = findTeam.getMembers();

tx.commit();                                  
```


- 빠트릴 수 있기 때문에 양방향으로 설정해주는 메소드를 생성하는 것을 추천
```java
public class Member{
  
  private Team team;
  
  public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
  }
  // ...
}
```

[ 연관관계 편의 메서드 작성 시 주의사항 ]
- teamB로 변경할 때 teamA → member1 관계를 제거하지 않았기 때문에 teamA.getMember() 메서드를 실행했을 때 member1이 남아있으므로
  연관관계를 변경할 때는 기존 팀이 있으면 기존 팀과 회원의 연관관계를 삭제하는 코드를 추가해야 한다.

```java
member.setTeam(team1);
member.setTEam(team2); 

Member findMember = teamA.getMember(); // member1이 여전히 조회된다.
```


### 양방향 매핑시에 무한 루프를 조심하자
- toString, lombok, json 생성 라이브러리 사용 시 발생
- toString() 사용 지양
  ```java
        class Member 

        ...
        ...
        ...
        @Override
        public String toStrig(){
        	return "Member{" +
            	"id=":+ id +
                ", username='" + username +'\'' +
                ", team=" +team +
                '}'"
             }
        }
    //Team 에서도 toString 생성
  ```
- 컨트롤러에는 엔티티 반환하지 말 것.
    - 엔티티 변경 가능성 있기 때문에 이 경우 API 스펙이 변경된다.
    - JSON은 컨트롤러에서 엔티티를 반환하지 않고 Dto 객체를 따로 만들어서 사용하면 문제가 생기지 않는다.

### 무조건 양방향 관계를 맺어야 할까 ?
  - 객체 입장에서 양방향 매핑을 했을 때 오히려 복잡해질 수 있다.
    - 예를 들어 사용자(User)엔티티는 굉장히 많은 엔티티와 연관 관계를 갖는다.
    - 이런 경우에 모든 엔티티를 양방향 관계로 설정하게 되면 사용자(User)엔티티는 엄청나게 많은 테이블과 연관 관계를 맺게 되고 클래스가 굉장히 복잡해진다.
    - 그리고 다른 엔티티들도 불필요한 연관관계 매핑으로 인해 복잡성이 증가할 수 있다.
  - 기본적으로 단방향 매핑으로 하고 나중에 역방향으로 객체 탐색이 꼭 필요하다고 느낄 때 추가한다.

## 2.양방향 매핑 정리
- **단방향 매핑만으로도 이미 연관관계 매핑은 완료**
- 양방향 매핑은 반대 방향으로 조회 ( 객체 그래프 탐색 ) 기능이 추가 된것 뿐
- JPQL 에서 역방향으로 탐색할 일이 많음
- 단방향 매핑을 잘 하고, 반대 방향은 필요할 때 추가 하면됨 ( mappedby )
    - 테이블에 영향을 주지 않기 때문에
- 연관관계의 주인을 정하는 기준
    - 비즈니스 로직 기준 x, 외래키의 위치를 기준으로 정한다.
