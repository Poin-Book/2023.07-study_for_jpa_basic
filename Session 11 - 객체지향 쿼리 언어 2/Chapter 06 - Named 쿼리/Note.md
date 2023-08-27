# Named 쿼리
> 작성자: @hw130

## 목차  
- [Named Query](#Named-Query)
- [Named Query 특징](#Named-Query-특징)
- [애플리케이션 로딩 시점에 쿼리를 검증](#애플리케이션-로딩-시점에-쿼리를-검증)  

### Named Query
---
- 쿼리에 이름 부여 가능한 쿼리  
```
@Entity
@NamedQuery(
	name = "Member.findByUsername",
    	query = "select m from Member m where m.username = :username")
public class Member {
	// ... 이하 생략
```
다음과 같이 사용할 수 있다. 샤용하는 방법은
```
List<Member> resultList = 
             em.createNamedQuery("Member.findByUsername", Member.class)
            .setParameter("username", "회원1")
            .getResultList();
```
결과를 보면  
![image](https://github.com/luke0408/study_for_jpa_basic/assets/87763333/f1c52b45-bf09-4a66-9492-fde942c93b45)  
다음과 같이 제대로 나온 것을 확인해볼 수 있다.  
그리고 XML 파일에도 다음과 같이 정의가 가능하다.  
```
<?xml version="1.0" encoding="UTF-8"?>
<entity-mappings xmlns="http://xmlns.jcp.org/xml/ns/persistence/orm" version="2.1">
  <named-query name="Member.findByUsername">
    <query><![CDATA[
      select m
      from Member m
      where m.username = :username
    ]]></query>
  </named-query>
  <named-query name="Member.count">
    <query>select count(m) from Member m</query>
  </named-query>
</entity-mappings>
```
XML 정의가 항상 우선권을 가진다.  
spring-data-jpa를 활용하게 되면 @Query를 사용하여 Repository에 Named 쿼리를 정의해서 사용한다.  
```
@Query("select u from User u where u.username = :username")
List<User> findUser(@Param("username") String username);
```

### Named Query 특징  
---
- 미리 정의해서 이름을 부여해두고 사용하는 JPQL  
- 정적 쿼리(쿼리의 형태가 동일. 동적 쿼리는 파라미터에 따라 달라질 수 있는 (동적으로 바뀌는) 쿼리)  
- 쿼리 재사용 가능  
- 어노테이션, XML에 정의  
- 애플리케이션 로딩 시점에 초기화 후 재사용
  - sql로 파싱해서 캐시한다.  
- **애플리케이션 로딩 시점에 쿼리를 검증**

### 애플리케이션 로딩 시점에 쿼리를 검증  
---
애플리케이션 로딩 시점에 쿼리를 검증한다는 특징은
```
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from MemberQQQQQQQQQQQQQQQQ m where m.username = :username")
```
다음과 같이 jpql을 잘못 작성했다고 하자, 이 때, 어플리케이션을 실행하면  
![image](https://github.com/luke0408/study_for_jpa_basic/assets/87763333/016212b1-ba6e-4dbe-a7cb-f272b85f2b08)  
이렇게 따로 이 쿼리를 사용하지 않아도 미리 검증해준다.  


