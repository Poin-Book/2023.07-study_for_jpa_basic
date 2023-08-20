# 프로젝션(SELECT)
> 작성자: @hw130

## 목차  
- [프로젝션](#프로젝션)
- [엔티티 타입 프로젝션](#엔티티-타입-프로젝션)
- [임베디드 타입 프로젝션](#임베디드-타입-프로젝션)
- [스칼라 타입 프로젝션](#스칼라-타입-프로젝션)
- [DISTINCT로 중복 제거](#DISTINCT로-중복-제거)

### 프로젝션  
---
프로젝션이란, SELECT 절에서 조회할 대상을 정하는 것을 의미한다.  
SELECT 절에서 조회할 수 있는 대상은 크게 3가지로 분류할 수 있다.  

### 엔티티 타입 프로젝션  
---
엔티티 프로젝션은 말그대로 원하는 엔티티(객체) 자체를 통째로 조회하는 것이다.  
```  
//Member(Entity)
List<Member> members = em.createQuery("SELECT m FROM Member m", Member.class).getResultList();

//Team
List<Team> team = em.createQuery("SELECT m.team FROM Member m", Team.class).getResultList();

```  
Member 조회 시  
<img width="334" alt="스크린샷 2023-08-20 오후 5 08 27" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/48c7953a-e3f8-4506-b8bc-8ce5c5b7731b">  
Member에 대한 객체 그래프 탐색 Team 조회 시  
<img width="320" alt="스크린샷 2023-08-20 오후 5 08 57" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/5ae734a6-dbd9-47b1-a778-470adfcb1964">  
Member와 Team간에 다대일 관계를 맺었기 때문에 Member는 Team을 참조하고 있고 따라서 Member -> Team으로 객체 그래프 조회를 할 수 있다.  
가장 중요한 사실은 **조회한 엔티티는 영속성 컨텍스트의 관리 대상** 이라는 점이다.  

### 임베디드 타입 프로젝션  
---
임베디드 타입은 엔티티와 비슷하게 사용하긴 한데 한가지 제약이 존재한다.  
> 임베디드 타입을 조회의 시작점으로는 절대 사용하지 못한다. 반드시 엔티티를 거쳐서 사용해야 한다.
  
이 말이 무슨 말인지는 다음 2가지 코드를 비교해보며 알아보도록 하겠다.  
``` List<Address> address = em.createQuery("SELECT a FROM Address a", Address.class);```  

이 코드는 Address를 조회의 시작점으로 생성한 JPQL이다. 그러나  
<img width="670" alt="스크린샷 2023-08-20 오후 5 21 26" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/83cc1017-d87c-4c4c-bf07-ca4f83206b67">  
다음과 같은 오류가 발생했다.  
왜냐하면 JPQL 자체가 "엔티티를 대상으로 쿼리"를 생성하는 것인데 임베디드 타입은 엔티티가 아니라 VO(Value Object) 즉, 값타입이기 때문에 애초에 JPQL에서 조회의 시작점으로 활용할 수 없다.  
따라서 Order의 Address를 조회하려면 다음과 같이 조회해야 한다.  
``` List<Address> address = em.createQuery("SELECT a address From Order o", Address.class).getResultList();```  

<img width="297" alt="스크린샷 2023-08-20 오후 5 23 02" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/b54b9d35-72bf-483e-93ec-ca89015076e9">  
그럼 다음과 같이 조회가 잘 되는 것을 확인할 수 있다.  

### 스칼라 타입 프로젝션  
---
스칼라 타입 프로젝션은 SELECT 절의 조회 대상이 데이터 타입과 상관없이 여러 데이터를 조회하는 경우를 말한다.  
```List resultList = em.createQuery("select m.username, m.age from Member m").getResultList();```  

- username은 String 타입, age는 int 타입이다.
- 이 경우 결과 데이터를 조회하기 위한 여러 방법이 존재한다.

1. Query 타입 조회
```Query query = em.createQuery("select distinct m.username, m.age from Member m");```

- 여러 값(타입)을 동시에 프로젝션 해야하는 상황이 오면 createQuery에서 특정 타입을 지정할 수 없다.  
따라서 createQuery는 TypeQuery가 아니라 Query로 만들어진다.

2. 타입 캐스팅을 이용한 Object[] 조회  
```
List resultList = em.createQuery("select m.username, m.age from Member m").getResultList();

// 타입 캐스팅
Object o = resultList.get(0);
Object[] result = (Object[]) o;

System.out.println("result[0] = " + result[0]);
System.out.println("result[0] = " + result[1]);
```
- 조회된 결과 List에는 Object 타입으로 저장된다.  
- 따라서, 각각의 username과 age를 조회하기 위해서는 배열로 캐스팅해야 한다.

```
List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m").getResultList();

Object[] result = resultList.get(0);
System.out.println("result[0] = " + result[0]);
System.out.println("result[0] = " + result[1]);
```

- 위와 같이 제네릭을 활용하여 타입 캐스팅이 가능하다.

3. new 명령어를 통한 DTO 조회

MemberDTO 생성  
```
@Getter
@Setter
@AllArgsConstructor
public class MemberDTO {

    private String username;
    private int age;

}
```
- 단순 값을 DTO로 바로 조회할 수 있다.

```
List<MemberDTO> result = em.createQuery("select new jpql.MemberDTO(m.username, m.age) from Member m", MemberDTO.class)
                    .getResultList();

MemberDTO memberDTO = result.get(0);
System.out.println("memberDTO = " + memberDTO.getUsername());
System.out.println("memberDTO = " + memberDTO.getAge());
```

- SELECT 절에 패키지 명을 포함하여 전체 클래스 명을 입력한다.
- 순서와 타입이 일치하는 생성자가 필요하다.
- 패키지명을 작성해야 하는 불편함은 QueryDSL을 사용하여 극복할 수 있다.

### DISTINCT로 중복 제거  
---
```em.createQuery("select distinct m.username, m.age from Member m").getResultList();```  

- SELECT 다음에 DISTINCT 키워드를 넣어주면 된다.
