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
        
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/1380c436-321f-4e4f-990e-db9fe2a44a54)
        
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/13e0294b-1ef2-4330-b54e-97472ff3f24d)   
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
            
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/1762c78c-fa41-4146-b537-40e48e2a671c)
            
- DB에서 조회
            
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/e5eaf33c-55b0-47fe-a5e2-ad3378c00316)
            
    - 근데 막 그렇게 이 캐시가 크게 도움이 되진 않는다.
        - 왜? 영속성 컨택스트는 한명의 요청이 끝나면 날아가기 때문에
- 동일성(identity) 보장
        
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/36a2c138-9e2b-4e5e-a1fd-6b524b7d3e3b)
        
    - 1차 캐시로 반복 가능한 읽기 등급(REPETABLE READ)의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공

  ### 2. 트랜잭션을 지원하는 쓰기 지연 (transactional write-behind)
    - 엔티티 등록
            
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/8d319050-5cb4-4698-a511-fcddc732cc9d)
            
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/84a0abe6-64ba-4ca9-8910-87f71cea6de9)
            
        - persist call시 쓰기지연 SQL 저장소(쿼리)와 1차 캐시(엔티티)에 저장한다.
            
            ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/29a2e0ee-019c-4524-8644-2227e49ba552)

            
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
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/36655a10-6217-4552-af32-8dbbe4ab1b10)
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/24b74039-7efa-4381-b438-2bb576613b92)
        
    - 엔티티 삭제
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/36720839-c737-4cc9-9fa3-de6b1cfbf041)
  ### 4. 정리 페이지(사진)
    https://pear-harp-86c.notion.site/f062a881414449dd96a4b42b8859d67f?pvs=4