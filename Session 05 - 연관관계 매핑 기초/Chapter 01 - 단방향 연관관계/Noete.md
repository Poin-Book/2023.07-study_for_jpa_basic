# 단방향 연관관계
> 작성자: @destiny3912

## 목차
- [단방향 연관관계](#단방향-연관관계)
  - [목차](#목차)
  - [연관관계 매핑 기초](#연관관계-매핑-기초)
    - [Overview](#overview)
      - [1. 연관관계가 필요한 이유](#1-연관관계가-필요한-이유)
      - [2. 단방향 연관관계](#2-단방향-연관관계)
## 연관관계 매핑 기초

### Overview
- 객체와 테이블의 연관 관계의 차이를 이해한다
  - 객체가 참조하는 것과 테이블이 외래키를 매핑하는 것의 차이
- 용어 이해
    - 방항 : 단방향, 양방향
    - 다중성: 다대일, 일대다, 일대일, 다대다
    - 연관관계의 주인: 객체 양방향 연관관계는 관리
#### 1. 연관관계가 필요한 이유
- 객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것
  - 협력이라는 행동을 만들기 위해 연관 관계가 필요하다.
- 예제 시나리오
        
    > 회원과 팀이 있다, 회원은 하나의 팀에만 소속이 가능, 회원과 팀은 대대일 관계이다.
    - 객체를 테이블에 맞추어 모델링
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/ab4a3e4e-c4a0-4de2-838a-a4b07f6002e4)
        *이 경우에는 테이블의 연관관계는 있지만 참조로 이루어져야 하는 객체의 연관관계는 표현이 되어있지 않음 따라서 JPA의 영속성 컨텍스트를 이용한 여러가지 동작이 불가함*
            
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/3ea0d7bc-ec63-407e-85fb-3ad294664e66)
        *참조 대신에 외래키를 그대로 사용*
            
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/ad87d31d-195b-4e74-80b2-539ac0d409e3)   
        *(조회 이슈)외래키 식별자를 그대로 다룬다 -> 이게 애매한거다*
            
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/4b06163d-053f-4e29-b9e3-7ab8b9bb8342)
        *식별자로 다시 조회하는 것 → 객체지향스러운 방식은 아님* -> 계속 물어봐야함… 비효율적
            
        > 객체를 테이블에 맞추어 데이터 중심으로 모델링 하면 협력 관계를 만들 수 없다.
        > 
        - 테이블은 외래 키로 조인을 해서 연관 테이블을 찾는다
        - 객체는 참조를 통해 연관된 객체를 찾는다.
#### 2. 단방향 연관관계
        
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/94a89cc7-a26e-4be0-9efa-e0d3662b6929)
        
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/92d241ff-0a2d-4e45-8e77-296a67513dc3)
        
- 누가 N이고 누가 1인지…?
    - 팀이 1 맴버가 N
        - 맴버 입장에서 ManyToOne
            - 어노테이션을 쓰는 애의 입장이 앞이다.
        - JoinColumn으로 컬럼 지정
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/1de3b0d8-d695-4e5e-8d9f-cd066105e4ec)
*ORM 매핑*
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/106e2d8a-771c-4865-a5be-78c61592fcab)
*저장*
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/aeafb3d4-8e93-4369-a286-1fce4bf2e5c7)
*조회*
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/7784c267-d970-4f61-ac40-1b2603e8d80f)
*수정*