# 페이징
> 작성자: @hw130

## 목차  
- [페이징 API](#페이징-API)  
### 페이징 API  
---
페이징을 처리하는 SQL을 반복적으로 작성하는 것은 굉장히 지루하고 귀찮은 일이다.  
persistence의 <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>에서 설정된 DB에 맞게 페이징 SQL을 작성해준다.  
JPA는 페이징 처리를 위한 API를 다음 2가지로 추상화 시켰다. 
- ```setFirstResult(int startPosition)```
- ```setMaxResults(int maxResult)```
- setFirstResult는 조회의 시작 위치를 정해주는 것이고 setMaxResults는 몇 개의 데이터를 페이징 처리할 것인가를 나타낸 것이다.  
- 여기서 주의해야할 점은 **조회의 시작 위치는 0** 이라는 것이다.

참고로 Spring Data JPA 에서는 Page 라는 클래스를 지원하는데 페이징에 대한 기능들을 한 번 더 추상화해 더욱 사용하기 편리하다.  

```
for (int i = 0; i < 100; i++) {
    Member member = new Member();
    member.setUsername("member" + i);
    member.setAge(i);
    em.persist(member);
}

em.flush();
em.clear();

//order by 넣어주는 것이 좋음. sorting이 제대로 되는지 확인용으로
List<Member> result = em.createQuery("select m from Member m order by m.age desc", Member.class)
        .setFirstResult(1) // 조회 시작(offset)
        .setMaxResults(10) // 조회할 데이터 수(limit)
        .getResultList();

System.out.println("result.size() = " + result.size());

for (Member member : result) {
    System.out.println("member = " + member);
}
```
- Member의 나이를 기준으로 내림차순(desc)으로 0부터 10개의 데이터를 조회한다.

```
// 출력
    order by
        member0_.age desc limit ? offset ?
result.size() = 10
member = Member{id=99, username='member98', age=98}
member = Member{id=98, username='member97', age=97}
member = Member{id=97, username='member96', age=96}
member = Member{id=96, username='member95', age=95}
member = Member{id=95, username='member94', age=94}
member = Member{id=94, username='member93', age=93}
member = Member{id=93, username='member92', age=92}
member = Member{id=92, username='member91', age=91}
member = Member{id=91, username='member90', age=90}
member = Member{id=90, username='member89', age=89}

```
- 참고로 offset과 limit은 데이터베이스 방언으로, H2의 표준이라고 보면 된다.
