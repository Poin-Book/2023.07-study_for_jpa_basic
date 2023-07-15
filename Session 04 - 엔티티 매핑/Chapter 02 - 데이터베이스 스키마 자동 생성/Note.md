# 데이터베이스 스키마 자동 생성
> 작성자: @destiny3912

## 목차
- [데이터베이스 스키마 자동 생성](#데이터베이스-스키마-자동-생성)
  - [목차](#목차)
    - [1. 정의](#1-정의)
    - [2. 속성](#2-속성)
    - [3. DDL 생성기능](#3-ddl-생성기능)
### 1. 정의

---

- DDL을 애플리케이션 실행 시점에 자동 생성하는 것
- 테이블 중심의 설계에서 객체 중심의 설계로
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 생성된 DDL은 개발 환경 에서만 사용
- 운영서버에서는 안쓰거나 적절히 다듬어서 사용

### 2. 속성

---

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0765f45-90e9-48ba-8812-c68cf764834f/Untitled.png)

- SpringBoot 기준 application.yml 같은 환경설정 파일에 저 속성이 들어가며
- 강의 기준으로는 persistence.xml에 직접 세팅을 해주어야 한다.
- 주의 사항
    - 데이터베이스 방언 별로 달라지는 것을 확인 해야함
    - 운영장비에는 create, create-drop, update 사용하면 안됨
    - 개발 초기 단계에는 create 또는 update
    - 테스트 서버는 update 또는 validate
    - 스테이징과 운영 서버는 validate 또는 none
    - alter table을 바로 운영 서버에 바로 올리는건 심각한 운영상 문제를 초래할 수 있음
        - 운영중인 서버에 테이블 수정 DDL을 올리면 테이블 수정하는 동안 DB락이 걸리기 때문에 몇분동안 서비스 중단이 되어버린 경우도 있음 (실제 사례라고 함)
        - 그러니까 운영 서버에서는 최대 validation까지만 해라 라는 것

### 3. DDL 생성기능

---

- DDL 자동 생성에만 사용하며 JPA의 실행 로직에는 영향을 주지 않음
- 제약조건 추가 (필수, or 길이) → 간편한 버전
    
    @Colunm(unique = true, length = 10) (필수에 길이는 10인 컬럼이다)
    
- 유니크 제약조건 추가
    
    ```java
    @Table(uniqueConstraints= {@UniqueConstraint(name="NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"})}
    ```
    
    - 복수의 컬럼에 대해서 동시에 지정
- Index 추가
    
    ```java
    @Table(name = "MEMBER", indexes = @Index(name = "IDX_MEMBER_ID", columnList = "MEMBER_ID"))
    ```
    
- 이름 명명 전략 → 기본 lower_snake_case
    - 암시적 → 기본
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d94563d-3d8e-47c6-ba6b-0012d6e0f2df/Untitled.png)
        
    - 명시적 → 클래스, 변수 이름 기준 생성
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f625803b-0652-4ad4-8bb7-c9c253d0aac9/Untitled.png)