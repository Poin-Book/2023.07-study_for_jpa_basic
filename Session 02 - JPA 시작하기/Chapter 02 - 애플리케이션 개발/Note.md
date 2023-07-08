# 애플리케이션 개발
> 작성자:밀리

## 목차

### JPA 구동방식
---
1. JPA가 시작되면 Persistence 클래스로 시작
2. Persistence 클래스에서 persistence.xml 파일을 읽는다.
3. EntityManagerFactory라는 클래스를 만들어서 필요할 때마다 EntityManager를 공장처럼 찍어낸다.
