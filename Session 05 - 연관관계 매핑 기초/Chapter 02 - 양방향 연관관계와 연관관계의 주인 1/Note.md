# 양방향 연관관계와 연관관계의 주인 1 - 기본
> 작성자: @asas6978

## 목차
- [양방향 매핑](#양방향-매핑)
- [연관관계의 주인과 mappedBy](#연관관계의-주인과-mappedby)
 
## 양방향 매핑
```java
@Entity 
public class Member {
   @Id
  @GeneratedValue
  private Long id;

  @Column(name = "USERNAME")
  private String name;

  private int age; 
 
  @ManyToOne 
  @JoinColumn(name = "TEAM_ID") 
  private Team team; 
```
```java
@Entity 
public class Team { 
  @Id
  @GeneratedValue
  private Long id;

  private String name; 

  @OneToMany(mappedBy = "team") 
  List<Member> members = new ArrayList<Member>(); 
```

<img width="500" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/d4613682-a451-4bc6-9dd4-1ee796ee9676">

테이블 연관관계는 테이블에 fk 하나만 넣어주면 양방향 조회가 가능하지만, 문제는 객체이다.
위 코드와 같이 두 객체에 연결하려는 관계를 모두 셋팅해주어야 접근이 가능하다.



## 연관관계의 주인과 mappedBy
mappedBy가 등장하게 된 흐름에는 객체와 테이블 간 연관관계를 맺는 방법의 차이점이 존재하는데, 위의 예제를 통해 두 방법 간 차이를 살펴보려고 한다.

객체는 회원 -> 팀으로 향하는 단방향 연관관계, 팀 -> 회원으로 향하는 연관관계로 총 두 개의 연관관계가 존재하지만 테이블의 경우 fk를 통해 회원 <-> 팀의 연관관계가 1개로 정립될 수 있다.

외래 키를 관리하는 하나의 관계를 연관관계의 주인이라고 하는데, 주인만이 외래 키를 등록하거나 수정하는 관리 권한을 가지게 된다. 주인이 아닌 경우 mappedBy 속성으로 주인을 지정하며, 읽기 권한만을 가지게 된다.

그렇다면 객체의 경우 누가 외래 키를 관리해야하는 것일까?

외래 키가 있는 곳을 주인으로 정하면 된다. 위의 예제에서는 Member.team의 연관관계의 주인이다.
이렇게 하면 반대의 경우보다 관리가 용이하고 성능적으로 유리하다는 장점이 존재하기 때문이다.

<img width="500" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/0902cccd-22e8-4ec4-adf3-81d2884bd6d3">
