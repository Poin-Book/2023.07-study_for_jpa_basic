# 즉시 로딩과 지연 로딩
> 작성자: @joowojr

## 목차


지연 로딩 
LAZY를 사용해서 프록시로 조회
@-to- (fetch = FetchType.LAZY)

예) 
Member member = em.find(Member.class, 멤버 아이디); --> member 정보만 가져옴
member.getTeam().getName() --> 프록시 개체 초기화, 이 때 team 정보 가져옴
member를 사용 할 때, team의 정보도 자주 사용한다면 ? -> 즉시 로딩
member의 정보만 필요할 때가 많다면? -> 지연 로딩

프록시와 즉시로딩 주의
- 가급적 지연로딩만 사용 ( 특히 실무에서 ) 매우 매우 강조 하심
- 즉시 로딩을 사용하면 예상치 못한 SQL 발생
  - 조인해서 정보를 가져오지 않더라도, 참조하는 테이블에 대한 조인 쿼리가 발생함
- 즉시로딩은 JPQL에서 N+1 문제를 일으킨다.
ex ) member 테이블을 조회하고 싶을 때
select m from Member m
1. member 조회 쿼리
2. member의 team 필드 보고 team 조회 쿼리
--> 쿼리 두 방으로 인해 성능 저하
지연 로딩 사용하면
1. member 조회 쿼리
2. 
다른 방법으로는 , 모든 로딩을 지연 로딩으로 설정하고, 즉시 로딩을 원하는 테이블만 아래와 같이 join fetch 사용
select m from Member m join fetch m.team
- @ManyToOne, @OneToOne은 기본이 즉시 로딩 -> LAZY로 설정해줘야함
- @OneToMany, @ManyToMany는 기본이 지연 로딩

## 지연 로딩 활용 - 실무
- 모든 연관관계에 지연 로딩 사용해라!
- 실무에서 즉시 로딩을 사용하지 마라!
- JPQL fetch 조인이나, 엔티티 그래프 기능을 사용해라 
- 즉시 로딩은 사용하지 못한 쿼리가 나간다.


