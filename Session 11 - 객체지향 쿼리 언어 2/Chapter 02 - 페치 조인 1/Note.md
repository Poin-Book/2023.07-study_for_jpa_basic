# 페치 조인 1 - 기본
> 작성자: @Hyunstone

## 목차
- [JPQL - 페치 조인](#jpql---페치-조인fetch-join)
- [컬렉션 페치 조인](#컬렉션-페치-조인)
- [페치 조인과 DISTINCT](#페치-조인과-distinct)
- [페치 조인과 일반 조인](#페치-조인과-일반-조인의-차이)
- [Q&A](#qa)


## JPQL - 페치 조인(fetch join)

실무에서 정말정말 중요함

**페치 조인(fetch join)**

- SQL 조인 종류X
- JPQL에서 성능 최적화를 위해 제공하는 기능
- 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능 → 한방쿼리로 풀어내는 기능
- join fetch 명령어 사용
- 페치 조인 ::= [ LEFT [OUTER] | INNER ] JOIN FETCH 조인경로

**엔티티 페치 조인**

- 회원을 조회하면서 연관된 팀도 함께 조회(SQL 한 번에)
- SQL을 보면 회원 뿐만 아니라 팀(T.*)도 함께 SELECT*
- **[JPQL]**
select m from Member m join fetch m.team
- **[SQL]**
SELECT M.* , T.* FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_ID=T.ID

→ 즉시 로딩하는 것과 동일. 그런데 차이는 내가 원하는 객체 그래프를 조회하는 것을 명시적으로 동적으로 가져오게 됨

```java
String jpql = "select m from Member m";
    List<Member> members = em.createQuery(jpql, Member.class)
            .getResultList();

    for (Member member : members) {
        System.out.println("username = " + member.getUsername() + ", " + "teamName = " + member.getTeam().name());
        // 회원1, 팀A(SQL)
        // 회원2, 팀A(1차캐시)
        // 회원3, 팀B(SQL)

		// 회원 100명 -> N + 1
    }
```

대부분의 N + 1 문제는 fetch join으로 해결해야 한다. 주로 조회에 많이 사용하게 된다.

페치 조인 사용 코드

String jpql = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(jpql, Member.class)
.getResultList();
for (Member member : members) {
//페치 조인으로 회원과 팀을 함께 조회해서 지연 로딩X
System.out.println("username = " + member.getUsername() + ", " +
"teamName = " + member.getTeam().name());
}
username = 회원1, teamname = 팀A
username = 회원2, teamname = 팀A
username = 회원3, teamname = 팀B

## 컬렉션 페치 조인

- 일대다 관계, 컬렉션 페치 조인
- [JPQL]
select t
from Team t join fetch t.members
where t.name = ‘팀A'
- [SQL]
SELECT T.**,* M*.**
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME = '팀A'

![image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/40d46478-9c5a-4e30-a793-6b13f3e8e75b)

일대다 fetch join의 경우, RDB와 객체의 차이로 인해 데이터가 뻥튀기가 됨

컬렉션 페치 조인 사용 코드

```java 
String jpql = "select t from Team t join fetch t.members where t.name = '팀A'"

List<Team> teams = em.createQuery(jpql, Team.class).getResultList();
for (Team team : teams) {
    System.out.println("teamname = " + team.getName() + ", team = " + team);
}
for (Member member : team.getMembers()) {
    //페치 조인으로 팀과 회원을 함께 조회해서 지연 로딩 발생 안함
    System.out.println("-> username = " + member.getUsername() + ", member = " + member);
}
```
teamname = 팀A, team = Team@0x100
-> username = 회원1, member = Member@0x200
-> username = 회원2, member = Member@0x300

teamname = 팀A, team = Team@0x100
-> username = 회원1, member = Member@0x200
-> username = 회원2, member = Member@0x300

## 페치 조인과 DISTINCT

- SQL의 DISTINCT는 중복된 결과를 제거하는 명령
- JPQL의 DISTINCT 2가지 기능 제공
    - 1. SQL에 DISTINCT를 추가
    - 2. 애플리케이션에서 엔티티 중복 제거
- SQL에 DISTINCT를 추가하지만 데이터가 다르므로 SQL 결과에서 중복제거 실패

→ 데이터가 완전히 일치해야, 모든 칼럼의 값이 일치해야 중복으로 판단해서 distinct가 적용이 됨. 따라서, distinct로 완전히 해결 X

일대다는 데이터가 뻥튀기 되는 위험이 있으나, 반대로 다대일은 그럴 가능성이 없어 자유롭게 fetch join을 사용해도 된다.

## 페치 조인과 일반 조인의 차이

- 일반 조인 실행시 연관된 엔티티를 함께 조회하지 않음
- 페치 조인은 연관된 엔티티를 함께 조회함
- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(사실상 즉시 로딩)
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념 → 한방 쿼리

## Q&A
[https://www.inflearn.com/questions/39516/fetch-조인-엔티티-그래프-질문입니다](https://www.inflearn.com/questions/39516/fetch-%EC%A1%B0%EC%9D%B8-%EC%97%94%ED%8B%B0%ED%8B%B0-%EA%B7%B8%EB%9E%98%ED%94%84-%EC%A7%88%EB%AC%B8%EC%9E%85%EB%8B%88%EB%8B%A4)

JPQL은 연관관계를 즉시로딩으로 설정하는 것과 상관없이 JPQL 자체만으로 SQL로 그대로 번역

**즉시 로딩(EARGR로 설정)**

**1. 멤버 전체를 조회하기 위해 JPQL 실행 select m from member m**

**2. JPQL은 EAGER와 무관하게 SQL로 그대로 번역 -> select m.* from member**

**3. JPQL 결과가 member만 조회하고, team은 조회하지 않음**

**4. member와 team이 즉시 로딩으로 설정되어 있기 때문에 연관된 팀을 각각 쿼리를 날려서 추가 조회 (N+1)**

**지연 로딩(LAZY로 설정)**

**1. 멤버 전체를 조회하기 위해 JPQL 실행 select m from member m**

**2. JPQL은 EAGER와 무관하게 SQL로 그대로 번역 -> select m.* from member**

**3. JPQL 결과가 member만 조회하고, team은 조회하지 않음**

**4. member와 team이 지연 로딩으로 설정되어 있기 때문에 가짜 프록시 객체를 넣어두고, 실제 회원은 팀은 조회하지 않음**

**5. 실제 team을 사용하는 시점에 쿼리를 날려서 각각 조회(N+1)**

**fetch join 또는 엔티티 그래프(EAGER, LAZY 상관 없음)**

**1. 멤버와 팀을 한번에 조회하기 위해 JPQL+fetch join 실행 select m from member m join fetch m.team**

**2. JPQL에서 fetch join을 사용했으므로 SQL은 멤버와 팀을 한 쿼리로 조회 -> select m.*, t.* from member join team ...**

**3. JPQL 결과가 member와 team을 한꺼번에 조회함**

**4. member와 team이 fetch join으로 한번에 조회되었으므로 N+1 문제가 발생하지 않음**

---

left join fetch를 사용하면, Member 테이블과 일치하는 Team 테이블을 조인하면서도 Member 테이블만 참조하는 경우에도 null이 반환됩니다. 이는 lazy loading으로 인한 객체 탐색 문제가 발생하지 않도록 해줍니다.

페치 조인과 일반 조인의 차이에서 일반 조인이 후에 다시 조회하게 되는 이유는 매우 간단합니다. 일반 조인에서는 연관된 엔티티를 함께 즉시 로딩하지 않고, 대신 연관된 엔티티에 대한 추가 select문을 나중에 발생시킵니다. 이 것은 엔티티 관계에서 LAZY 로딩이 기본값이기 때문입니다. LAZY 로딩이란, 필요한 순간에 쿼리를 발생시켜 데이터를 가져온다는 것입니다.

즉, 일반 조인에서는 처음 쿼리에 있는 엔티티만 가져오고, 연관된 엔티티는 필요한 시점에서 쿼리를 발생시켜 다시 가져오게 됩니다.

eager loading을 지양하라고 하는 이유는 말씀하신 것처럼 불필요한 부분도 조회하기 때문. lazy loading + fetch join을 권장드리는 이유는 본질적으로 필요할 때만 같이 불러오기 위해서. 부차적으로는 eager loading을 남발할 때 예상치 못한 SQL이 작성될 수 있습니다.