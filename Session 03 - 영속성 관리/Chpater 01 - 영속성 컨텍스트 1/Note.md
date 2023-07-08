# 영속성 컨텍스트 1
> 작성자:루키/이지호

## 목차

- [영속성 컨텍스트 1](#영속성-컨텍스트-1)
  - [목차](#목차)
    - [1. 엔티티 매니저 팩토리와 엔티티 매니저](#1-엔티티-매니저-팩토리와-엔티티-매니저)
    - [2. 영속성 컨텍스트](#2-영속성-컨텍스트)
    - [3. 엔티티의 생명주기](#3-엔티티의-생명주기)
    - [4. 정리 페이지(사진)](#4-정리-페이지사진)
### 1. 엔티티 매니저 팩토리와 엔티티 매니저
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/299a3418-e80d-4ca0-98cb-03b2931bc437/Untitled.png)


### 2. 영속성 컨텍스트

  -  JPA를 이해하는데 가장 중요한 용어
  - 엔티티를 영구 저장하는 환경이라는 뜻
  - EntityManager.persist(entity);
      - 영속성 컨텍스트를 통해 entity를 영속화 시킨다.
      - entity를 영속화 컨텍스트 안에 저장한다.
  - 영속성 컨텍스트는 논리적인 개념이다
      - 따라서 눈에 안보임
  - 엔티티 메니저를 통해서 영속성 컨텍스트에 접근한다.
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/16ab1581-2fab-45a3-975f-bc9375024bd0/Untitled.png)
        
### 3. 엔티티의 생명주기
  - 비영속
      - 영속성 컨텍스트와 전혀 관계가 없는 새로운 상태
        

        ```java
            //객체를 생성한 상태(비영속)
            Member member = new Member();
            member.serId("member1");
            member.setUserName("회원1");
        ```
        
  - 영속
      - 영속성 컨텍스트에 관리되는 상태

        ```java
        //객체를 생성한 상태(비영속)
        Member member = new Member();
        member.serId("member1");
        member.setUserName("회원1");
        
        EntityManager em = emf.createEntityManager();
        em.getTranscation().begin();
        
        //객체를 저장한 상태 (영속)
        em.persist(member);
        ```
        
        - 이 상태가 된다고 DB에 쿼리가 날아가지 않는다
        - 커밋이 될때 날아간다.
    c. 준영속
        - 영속성 컨텍스트에 저장되었다가 분리된 상태

        ```java
        em.detach(member);
        ```
        
        - 영속(1차 캐시에 올라간 상태) → 준영속
        - 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)
        - 영속성 컨텍스트가 제공하는 기능을 사용 못함
        - 준영속 상태로 만드는 방법
            - em.detach(entity)
                - 특정 엔티티만 준영속 상태로 전환
                - 해당 엔티티에 관련된 모든게 빠진다.
            - em.clear()
                - 영속성 컨텍스트를 완전히 초기화
            - em.close()
                - 영속성 컨텍스트를 종료
    d. 삭제
        - 삭제된 상태

        ```java
        em.remove(member);
        ```
        
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24287126-9316-4066-8931-b9e1751851e1/Untitled.png)

### 4. 정리 페이지(사진)
https://pear-harp-86c.notion.site/f062a881414449dd96a4b42b8859d67f?pvs=4