# 영속성 컨텍스트 2
> 작성자: 루키/이지호

## 목차

- [영속성 컨텍스트 2](#영속성-컨텍스트-2)
  - [목차](#목차)
  - [영속성 컨텍스트의 이점](#영속성-컨텍스트의-이점)
    - [1. 1차 캐시](#1-1차-캐시)
    - [2. 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)](#2-트랜잭션을-지원하는-쓰기-지연-transactional-write-behind)
    - [3. 변경 감지(Dirty Checking)](#3-변경-감지dirty-checking)
    - [4. 정리 페이지(사진)](#4-정리-페이지사진)

## 영속성 컨텍스트의 이점

### 1. 1차 캐시
        
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a145b9da-6458-416a-84dd-f4802a1b81b9/Untitled.png)
        
    ![select가 안날아간다!](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/319baef0-3246-482f-8e67-e605d9b1698c/Untitled.png)
        
- select가 안날아간다!
        
        ```java
        //객체를 생성한 상태(비영속)
        Member member = new Member();
        member.setId(101L);
        member.setName("회원1");
        
        //객체를 저장한 상태 (영속)
        System.out.println("====== BEFORE ========");
        em.persist(member);
        System.out.println("====== AFTER ========");
        
        Member findMember = em.find(Member.class, 101L);
        ```
        
- PK, member 객체 자체의 쌍으로 들어간다
- 1차 캐시에서 조회
            
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/21d691c7-f8d1-4fed-996d-48f037e1e616/Untitled.png)
            
- DB에서 조회
            
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a434e560-848b-4838-8ec4-0cf17a3ca297/Untitled.png)
            
    - 근데 막 그렇게 이 캐시가 크게 도움이 되진 않는다.
        - 왜? 영속성 컨택스트는 한명의 요청이 끝나면 날아가기 때문에
- 동일성(identity) 보장
        
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/44372341-bb2b-482e-bb05-6276d9a3a32d/Untitled.png)
        
    - 1차 캐시로 반복 가능한 읽기 등급(REPETABLE READ)의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

  ### 2. 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)
    - 엔티티 등록
            
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b3b5c5d-62ea-46c6-9dda-de6c2f980e39/Untitled.png)
            
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fa13ad92-1495-4840-93a7-bbb4b9c38a7f/Untitled.png)
            
        - persist call시 쓰기지연 SQL 저장소(쿼리)와 1차 캐시(엔티티)에 저장한다.
            
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/54352d4d-1596-4947-a702-fd7e535a4655/Untitled.png)
            
        ```java
        //객체를 생성한 상태(비영속)
        Member member1 = new Member(150L, "A");
        Member member2 = new Member(160L, "B");
            
        em.persist(member1);
        em.persist(member2);
            
        System.out.println("======================================");
        tx.commit();
        ```
            
    - 배치 사이즈

        ```xml
        <property name="hibernate.jdbc.batch_size" value="10"/>
        ```
            
                
    - 결국은 그냥 버퍼링 기능이다.
         
   ### 3. 변경 감지(Dirty Checking)
        
    ```java
    Member findMember = em.find(Member.class, 150L);
                    
    findMember.setName("ZZZ");
    ```
        
    - JPA의 목적은 컬랙션을 다루듯이 다룰 수 있는것이 목적이다
    - 따라서 setter로 변경만 해주면 들어간다.
    - 커밋시 변경을 반영
        
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/07d72a30-92e2-4277-959e-ac304d89d94f/Untitled.png)
        
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5850b803-f1be-4ff8-9806-387f91c2d30a/Untitled.png)
        
    - 엔티티 삭제
  
  ### 4. 정리 페이지(사진)
    https://pear-harp-86c.notion.site/f062a881414449dd96a4b42b8859d67f?pvs=4