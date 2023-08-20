# JPQL 타입 표현과 기타식
> 작성자: @destiny3912

## 목차

### 1. 문자
    
```sql
'HELLO'
'She''s' 
 /* ' 이 필요하면 ' 2개를 넣자 */
```
    
### 2. 숫자
    
```sql
10L(Long)
10D(Double)
10F(Float)
```
    
### 3. Boolean
```sql
TRUE
FALSE
```
    
### 4. ENUM
```sql
jpabook.MemberType.Admin // 패키지명 포함 해야함 아니면 파라미터 바인딩
```
    
### 5. 엔티티 타입
    
```sql
TYPE(m) = Member // 상속에서 사용
    
/*예시*/
select i from Item i where type(i) = Book
```
    
### 6. 기타
- SQL과 문법이 같은 식
    - exists, in
    - and, or, not
    - =, >, > =, <, < =, <>
    - between, like, is null

### 7. 예시
    
```sql
String query = "select m.username, 'HELLO', TRUE From Member m";
List<Object[]> resultList = em.createQuery(query).getResultList();
    
for (Object[] objects : resultList) {
    System.out.println("objects = " + objects[0]);
    System.out.println("objects = " + objects[1]);
    System.out.println("objects = " + objects[2]);
}
```
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/ba9d1735-a4b4-4fdf-93f9-32cbaa0b3cfe)