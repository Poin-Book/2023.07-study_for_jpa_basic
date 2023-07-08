# 애플리케이션 개발
> 작성자:밀리

## 목차

### JPA 구동방식
---
1. JPA가 시작되면 Persistence 클래스로 시작
2. Persistence 클래스에서 persistence.xml 파일을 읽는다.
3. EntityManagerFactory라는 클래스를 만들어서 필요할 때마다 EntityManager를 공장처럼 찍어낸다.


### JPA 엔티티 설계 시 주의할 점!!   
반드시 @Entity 어노테이션을 선언해줘야 한다.  
엔티티에 @Id 어노테이션을 설정하여 데이터베이스 Pk와 매핑해야 한다.   

### h2 database 연결하기   
h2 파일로 이동 후 다음 코드 작성
```cd bin
   ./h2.sh
```
### EntityManagerFactory & EntitiyManager 생성하기   
---
```EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");   
   EntityManager em = emf.createEntityManager();
```
** 어떤 행위(조회,생성 등)를 할 때 반드시 엔티티 매니저를 만들어줘야 된다. **  
