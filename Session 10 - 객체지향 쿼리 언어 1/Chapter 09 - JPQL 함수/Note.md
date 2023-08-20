# JPQL 함수
> 작성자: @joowojr

## 목차
### 1. JPQL의 기본 함수
### 2. 사용자 정의 함수

## 1. JPQL의 기본 함수
| JPQL이 제공하는 표준 함수로, DB와 상관 없이 사용할 수 있다.
### JPQL의 기본함수 종류
- CONCAT : 두개의 문자를 합친다.
  - 하이버네이트에서는 CONCAT 대신 || 로도 사용 할 수 있다.
    ```java
    String query = "select 'a' || 'b' from Member m ";
    ```
- SUBSTRING : 주어진 위치에서 원하는 길이 만큼 문자열을 자른다.
- TRIM : 문자열의 양쪽 공백을 제거해준다. LTRIM(왼쪽 공백만 제거), RTRIM(오른쪽 공백만 제거) 가 있다.
- LOWER, UPPER : 모든 문자를 소문자 변환 / 대문자 변환
- LENGTH : 문자열의 길이를 반환한다.
- LOCATE : 해당 문자가 포함된 문자열의 위치를 반환한다.
- ABS, SQRT, MOD : 절댓값/ 제곱근/ 나눗셈 나머지 반환
- SIZE, INDEX(JPA 용도) : SIZE는 컬렉션의 크기를, INDEX는 값 타입 컬렉션의 위치를 구할 때 사용 ( 권장하지 않음 )

## 2. 사용자 정의 함수 호출
- 직접 함수를 정의할 수 있다.
- dialect 를 상속받아 클래스를 생성한다.
  ```java
    public class MyH2Dialect extends H2Dialect {
  
      public MyH2Dialect() {
          registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
      }
  }
  ```
- persistence.xml에 상속받은 dialect 를 등록해줘야한다.
  ```xml
  <persistence version="2.2"
               xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
      <persistence-unit name="hello">
          <properties>
              <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
              <property name="javax.persistence.jdbc.user" value="sa"/>
              <property name="javax.persistence.jdbc.password" value=""/>
              <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test"/>
              <property name="hibernate.dialect" value="dialect.MyH2Dialect"/>
              
              ...
      </persistence-unit>
  </persistence>
```
