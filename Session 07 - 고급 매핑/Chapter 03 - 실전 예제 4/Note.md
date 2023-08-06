# 실전 예제 4 - 상속관계 매핑
> 작성자: @destiny3912

## 목차
- [실전 예제 4 - 상속관계 매핑](#실전-예제-4---상속관계-매핑)
  - [목차](#목차)
    - [1. 요구사항 추가](#1-요구사항-추가)
    - [2. 도메인 모델](#2-도메인-모델)
    - [3. 도메인 모델 상세](#3-도메인-모델-상세)
    - [4. 테이블 설계](#4-테이블-설계)
    - [5. Code - 상속 관계](#5-code---상속-관계)
    - [6. Result](#6-result)
    - [7. @ManyToMany를 쓰면 안되는 이유 하나 더](#7-manytomany를-쓰면-안되는-이유-하나-더)
    - [8. 실무에서 이렇게 쓰냐?](#8-실무에서-이렇게-쓰냐)

### 1. 요구사항 추가
- 상품의 종류는 음반, 도서, 영화가 있고 이후 더 확장이 가능하다
- 모든 데이터는 등록일과 수정일이 필수다
### 2. 도메인 모델
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/93281f98-1784-428b-8c7d-b91fd9066733)
    
### 3. 도메인 모델 상세
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/c5c2dbfd-bb30-494a-a17a-98c2c6f96bae)
    
### 4. 테이블 설계 
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/af8e0470-0811-4f5e-b989-973155de7eb3)
    
### 5. Code - 상속 관계
- Item
        
     ```java
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE)
    @DiscriminatorColumn
    public abstract class Item extends BaseEntity{
    ```
        
- Album
        
    ```java
    @Entity
    public class Album extends Item{
        
        private String artisit;
        
        private String etc;
        
    //...
    }
    ```
        
- Book
        
    ```java
    @Entity
    public class Book extends Item{
        
        private String author;
        private String isbn;
        
    //...
    }
    ```
        
- Movie
        
    ```java
    @Entity
    public class Movie extends Item{
        
        private String director;
        private String actor;
        
    //...
    }
    ```
        
- BaseEntity
        
    ```java
    @MappedSuperclass
    public class BaseEntity {
        private String createdBy;
        private LocalDateTime createdDate;
        
        private String lastModifiedBy;
        
        private LocalDateTime lastModifiedDate;
        
    //...
    }
    ```
        
- Main
        
    ```java
    Book book = new Book();
    book.setName("JPA");
    book.setAuthor("김연한");
        
    em.persist(book);
        
    tx.commit();
    ```
        
### 6. Result
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/065517b6-8238-40dc-b408-02fd3073a388)
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/78aac7e5-1338-40be-9a79-86f68cb0a173)
    
### 7. @ManyToMany를 쓰면 안되는 이유 하나 더
    
![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/827c3579-1224-43af-a19b-7a5a20d32c99)
    
### 8. 실무에서 이렇게 쓰냐?
- 쓰는 경우도 있고
- 상속으로 쓰다가 너무 복잡해져서
     - 데이터가 억단위가 넘어가기 시작하면 복잡해진다
    - 테이블은 무조건 단순하게
    - 상속을 깨고 JSON으로 말아버리는 경우도 있다.
- 장점과 단점의 Trade Off가 뒤집히는 순간에 바꾼다.