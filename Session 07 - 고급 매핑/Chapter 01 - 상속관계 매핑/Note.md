# 상속관계 매핑
> 작성자: @hw130

## 목차  
- [상속관계 매핑](#상속관계-매핑)  
    - [조인 전략](#조인-전략)  
    - [단일 테이블 전략](#단일-테이블-전략)  
    - [구현 클래스 전략](#구현-클래스-전략)  
- [결론 및 정리](#결론-및-정리) 

 
 ### 상속관계 매핑  
 ---  
 - 객체는 상속관계가 존재하지만, 관계형 데이터베이스는 상속 관계가 없다.(대부분)  

- 그나마 슈퍼타입 서브타입 관계라는 모델링 기법이 객체 상속과 유사하다.  

- 상속관계 매핑이라는 것은 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑하는 것이다. 즉, 슈퍼타입 서브타입을 논리 모델을 실제 물리 모델로 변환하는 방법이다.  
 
<img width="780" alt="스크린샷 2023-08-06 오전 1 43 13" src="https://github.com/hw130/Algorithm_practice/assets/87763333/2ed855e0-eca0-4de4-9c6f-4f0da327de26">  

#### 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법  
- 객체는 상속을 지원하므로 모델링과 구현이 똑같지만, DB는 상속을 지원하지 않으므로 논리 모델을 물리 모델로 구현할 방법이 필요하다.  
- DB의 슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법은 세가지가 있다. 여기서 중요한건, DB입장에서 세가지로 구현하지만 JPA에서는 어떤 방식을 선택하던 매핑이 가능하다.  
    - 각각 테이블로 변환하는 조인 전략(JOINED)  
    - 단일 테이블 전략이라는 논리 모델을 한 테이블로 합치는 전략(SINGLE_TABLE)  
    - 구현 클래스마다 테이블 전략(TABLE_PER_CLASS)
- JPA가 이 세가지 방식과 매핑하려면 
    - @Inheritance(strategy=InheritanceType.XXX)의 stategy를 설정해주면 된다.  

    - default 전략은 SINGLE_TABLE(단일 테이블 전략)이다.  

        - InheritanceType 종류  

            - JOINED  

            - SINGLE_TABLE  

            - TABLE_PER_CLASS  

    - @DiscriminatorColumn(name="DTYPE")  

        - 부모 클래스에 선언한다. 하위 클래스를 구분하는 용도의 컬럼이다. 관례는 default = DTYPE  

    - @DiscriminatorValue("XXX")  

        - 하위 클래스에 선언한다. 엔티티를 저장할 때 슈퍼타입의 구분 컬럼에 저장할 값을 지정한다.  

        - 어노테이션을 선언하지 않을 경우 기본값으로 클래스 이름이 들어간다.  
  
### 조인 전략  
---  
 - 가장 정규화 된 방법으로 구현하는 방식이다.  
- 엔티티 각각을 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아 기본 키 + 외래 키로 사용하는 전략  
- 자식 테이블 중 어느 테이블을 조회해야하는지 구분하기 위해 DTYPE이란 구분 컬럼을 사용한다.  
- NAME, PRICE가 ITEM 테이블에만 저장되고, ALBUM, MOVIE, BOOK이 각자의 데이터만 저장한다.  
<img width="939" alt="스크린샷 2023-08-06 오전 1 48 17" src="https://github.com/hw130/Algorithm_practice/assets/87763333/51166a9d-1238-4c95-b9ec-10a1845661c7">  
  
#### 예제코드
  
```  
    @Entity
    @Inheritance(strategy = InheritanceType.JOINED)
    @DiscriminatorColumn(name = "DTYPE")
    public class Item {

        @GeneratedValue @Id
        private Long id;  
        ...
```  
    
```  
    @Entity
    @DiscriminatorValue("M")
    public class Movie extends Item {

        private String actor;

        ...
```  
    
- @Inheritance(strategy = InheritanceType.JOINED)  
    - 부모 클래스에 지정. 조인 전략이므로 InheritanceType.JOINED 설정  
- @DiscriminatorColumn(name = "DTYPE")  
    - 구분 컬럼 지정. 부모 클래스에 선언한다. 하위 클래스를 구분하는 용도의 컬럼이다. 관례는 default = DTYPE이고, Default값이 DTYPE이므로 name 속성은 생략 가능  
- @DiscriminatorValue("M")  
    - 구분 컬럼에 입력할 값 지정.  
    - 하위 클래스에 선언한다. 엔티티를 저장할 때 슈퍼타입의 구분 컬럼에 저장할 값을 지정한다.  
    - 어노테이션을 선언하지 않을 경우 기본값으로 클래스 이름이 들어간다.  
        
#### 실제 실행된 DDL  
```  
    Hibernate:
   create table Album (
      artist varchar(255),
      id bigint not null,
      primary key (id)
    )
    Hibernate:
    create table Book (
        author varchar(255),
        isbn varchar(255),
        id bigint not null,
        primary key (id)
    )
    Hibernate:
    create table Item (
        DTYPE varchar(31) not null,
        id bigint generated by default as identity,
        name varchar(255),
        price integer not null,
        primary key (id)
    )
    Hibernate:
    create table Movie (
        actor varchar(255),
        director varchar(255),
        id bigint not null,
        primary key (id)
    )
    
    Hibernate:
    alter table Album
        add constraint FKcve1ph6vw9ihye8rbk26h5jm9
        foreign key (id)
        references Item
    Hibernate:
    alter table Book
        add constraint FKbwwc3a7ch631uyv1b5o9tvysi
        foreign key (id)
        references Item
    Hibernate:
    alter table Movie
        add constraint FK5sq6d5agrc34ithpdfs0umo9g
        foreign key (id)
        references Item 
```  

- 테이블 4개 생성  

- 하위 테이블에 외래키 제약조건 생성. 하위 테이블 입장에서는 ITEM_ID가 PK이면서 FK로 잡아야 한다.  

- 조인 전략에 맞는 테이블들이 생섬됨.  
      
#### 만약 Movie 객체를 저장하면?  

```  
    Movie movie = new Movie();
    movie.setDirector("감독A");
    movie.setActor("배우B");
    movie.setName("분노의질주");
    movie.setPrice(35000);
    ​
    em.persist(movie);
    ​
    tx.commit();  

```  
      
```  
    Hibernate:
    /* insert advancedmapping.Movie
        */ insert
        into
            Item
            (id, name, price, DTYPE)
        values
            (null, ?, ?, 'Movie')
    Hibernate:
    /* insert advancedmapping.Movie
        */ insert
        into
            Movie
            (actor, director, id)
        values
            (?, ?, ?)  

```
    
- Insert 쿼리가 두개 나간다.  

- Item 테이블, Movie 테이블 저장.  

- DTYPE에 클래스 이름이 디폴트로 저장됨.  
      
#### Movie 객체를 조회하면?  

```  
    Movie movie = new Movie();
    movie.setDirector("감독A");
    movie.setActor("배우B");
    movie.setName("분노의질주");
    movie.setPrice(35000);
    ​
    em.persist(movie);
    ​
    em.flush();
    em.clear();  //DB에 insert쿼리 날리고, 1차 캐시 지우므로 find에서 SELECT 쿼리가 나간다.
    ​
    em.find(Movie.class, movie.getId());
    ​
    tx.commit();

```  

```  
    Hibernate:
      select
          movie0_.id as id2_2_0_,
          movie0_1_.name as name3_2_0_,
          movie0_1_.price as price4_2_0_,
          movie0_.actor as actor1_3_0_,
          movie0_.director as director2_3_0_
      from
          Movie movie0_
      inner join
          Item movie0_1_
              on movie0_.id=movie0_1_.id
      where
          movie0_.id=?

```  


- flush(), clear() 해주면, DB에 insert쿼리 날리고, 1차 캐시 지우므로 find에서 SELECT 쿼리가 나간다.  
- Item과 inner join을 통해서 결과를 조회한다.

- 장점  
    - 테이블의 정규화  
    - 외래 키 참조 무결성 제약조건 활용 가능  
        - ITEM의 PK가 ALBUM, MOVIE, BOOK의 PK이자 FK이다. 그래서 다른 테이블에서 아이템 테이블만 바라보도록 설계하는 것이 가능하다.    
    - 저장공간을 효율적으로 사용 가능  
        - 테이블 정규화로 저장공간이 딱 필요한 만큼 소비된다.  
- 단점  
    - 조회시 잦은 조인으로 인해 성능 저하 가능성  
    - 복잡한 조회 쿼리  
    - 데이터 등록 시, 두번 실행되는 INSERT문  
- 정리  
    - 성능 저하라고 되어있지만, 실제로는 영향이 크지 않다.  
    - 오히려 저장공간이 효율화 되기 때문에 장점이 크다.  
    - 기본적으로는 조인 정략이 정석이라고 보면 된다. 객체랑도 잘 맞고, 정규화도 되고, 그래서 설계가 깔끔하게 나온다.    

### 단일 테이블 전략  
---  
- 하나의 테이블을 사용하며 구분 컬럼(DTYPE)을 활용해 데이터를 활용하는 전략
- 단일 테이블 적용은 strategy를 SINGLE_TABLE로 변경하면 끝난다.  
    - JPA의 장점이다. 테이블 구조의 변동이 일어났는데, 코드를 거의 손대지 않고 어노테이션만 수정했다.  
    - 만약 JPA를 쓰지 않았더라면, 테이블 구조의 변경이 일어나면 거의 모든 코드를 손대야 할 것이다.  
    - 단일 테이블 전략에서는 @DiscriminatorColumn을 선언해 주지 않아도, 기본으로 DTYPE 컬럼이 생성된다.  
    - 한 테이블에 모든 컬럼을 저장하기 때문에, DTYPE 없이는 테이블을 판단할 수 없다.  
<img width="814" alt="스크린샷 2023-08-06 오전 1 48 48" src="https://github.com/hw130/Algorithm_practice/assets/87763333/39ba9e53-3eb7-4733-9c1e-728fabf34764">  

#### 예제 코드  

```  
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "DTYPE")
public class Item {

    @GeneratedValue @Id
    private Long id;  
    ...
```
  
```  
@Entity
@DiscriminatorValue("M")
public class Movie extends Item{

    private String actor;

}

```
- @Inheritance(strategy = InheritanceType.SINGLE_TABLE)  
    - 부모 클래스에 지정. 단일 테이블 전략이므로 InheritanceType.SINGLE_TABLE 설정
      
#### 실행된 DDL  

```  
Hibernate:
   create table Item (
      DTYPE varchar(31) not null,
      id bigint generated by default as identity,
      name varchar(255),
      price integer not null,
      artist varchar(255),
      author varchar(255),
      isbn varchar(255),
      actor varchar(255),
      director varchar(255),
      primary key (id)
  )  

```  
- 통합 테이블이 하나 생성됨.  
#### 조인 전략에서 실습했던 Movie 저장, 조회 예제를 그대로 돌려보면? 

```  
Hibernate:
   select
      movie0_.id as id2_0_0_,
      movie0_.name as name3_0_0_,
      movie0_.price as price4_0_0_,
      movie0_.actor as actor8_0_0_,
      movie0_.director as director9_0_0_
   from
      Item movie0_
   where
      movie0_.id=?
       and movie0_.DTYPE='Movie'  
```  
- Item 테이블을 그냥 조회한다. 조인하지 않고, DTYPE을 검색 조건으로 추가해서 Movie를 조회
  
- 장점  
    - INSERT 쿼리도 한 번, SELECT 쿼리도 한 번이다. 조인할 필요가 없고, 조회 성능이 좋다.  
    - 단순한 조회 쿼리  
- 단점  
    - 자식 엔티티가 매핑한 컬럼은 모두 NULL 허용  
    - 높은 테이블이 커질 가능성으로 인해 오히려 조회 성능이 안좋아질 수 있음  

### 구현 클래스 전략  
---  
- 자식 엔티티마다 테이블 생성하는 전략
- 조인 전략과 유사하지만, 슈퍼 타입의 컬럼들을 서브 타입으로 내린다. NAME, PRICE 컬럼들이 중복되도록 허용하는 전략이다.  
- 데이터베이스 설계자와 ORM 전문가 둘 다 추천하지 않는 전략  
<img width="965" alt="스크린샷 2023-08-06 오전 1 49 16" src="https://github.com/hw130/Algorithm_practice/assets/87763333/f7ff3b1f-a354-439b-8615-94e9905b5298">  

#### 구현 예제  

```  
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
@DiscriminatorColumn(name = "DTYPE")
public abstract class Item {

    @GeneratedValue @Id
    private Long id;  
    ...

```

- @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)  
    - 부모 클래스에 지정. 구현 클래스마다 테이블 전략이므로 InheritanceType.TABLE_PER_CLASS 설정  
- 이때, ITEM 엔티티는 실제 생성되는 테이블이 아니므로 abstract 클래스여야 하고, @DiscriminatorColumn도 필요가 없어진다.  
- 하지만, @Id 생성 전략 GenerationType.IDENTITY를 사용하는 경우 문제가 있다.  
    - @Inheritance의 TABLE_PER_CLASS와 @Id의 GenerationType을 IDETITY를 같이 사용할 경우 에러 발생  
        - Cannot use identity column key generation with <union-subclass> mapping for  
    - @Id의 GenerationType을 TABLE 타입으로 변경 적용해서 해결. 그러나, 시퀀스 테이블이 생성되므로 이것에 대한 매핑까지 추가해줘야 완벽히 해결이 된다.  
#### 실행된 DDL  

```  
Hibernate:
   create table Album (
      id bigint not null,
       name varchar(255),
       price integer not null,
       artist varchar(255),
       primary key (id)
  )
Hibernate:
   create table Book (
      id bigint not null,
       name varchar(255),
       price integer not null,
       author varchar(255),
       isbn varchar(255),
       primary key (id)
  )
Hibernate:
   create table Movie (
      id bigint not null,
       name varchar(255),
       price integer not null,
       actor varchar(255),
       director varchar(255),
       primary key (id)
  )  

```  

- 하위 테이블 3개만 생성된다.  
#### 조인 전략에서의 Movie 저장, 조회를 그대로 실행한 결과는?  

```  
Hibernate:
   /* insert advancedmapping.Movie
       */ insert
       into
           Movie
          (name, price, actor, director, id)
       values
          (?, ?, ?, ?, ?)
Hibernate:
   select
       movie0_.id as id1_2_0_,
       movie0_.name as name2_2_0_,
       movie0_.price as price3_2_0_,
       movie0_.actor as actor1_3_0_,
       movie0_.director as director2_3_0_
   from
       Movie movie0_
   where
       movie0_.id=?  

```  
- Movie 테이블에 insert  
- Movie 테이블에서 select  
#### 문제점  

```  
Movie movie = new Movie();
movie.setDirector("감독A");
movie.setActor("배우B");
movie.setName("분노의질주");
movie.setPrice(35000);
​
em.persist(movie);
​
em.flush();
em.clear();
​
em.find(Item.class, movie.getId());
​
tx.commit();  

```  

- 객체지향 프로그래밍에서는 MOVIE, ALBUM, BOOK 객체를 ITEM 타입으로도 조회할 수 있다.  

#### 실행된 SQL  

```  

Hibernate:
   select
      item0_.id as id1_2_0_,
      item0_.name as name2_2_0_,
      item0_.price as price3_2_0_,
      item0_.artist as artist1_0_0_,
      item0_.author as author1_1_0_,
      item0_.isbn as isbn2_1_0_,
      item0_.actor as actor1_3_0_,
      item0_.director as director2_3_0_,
      item0_.clazz_ as clazz_0_
   from
      ( select
          id,
          name,
          price,
          artist,
           null as author,
           null as isbn,
           null as actor,
           null as director,
           1 as clazz_
       from
          Album
       union
      all select
          id,
          name,
          price,
           null as artist,
          author,
          isbn,
           null as actor,
           null as director,
           2 as clazz_
       from
          Book
       union
      all select
          id,
          name,
          price,
           null as artist,
           null as author,
           null as isbn,
          actor,
          director,
           3 as clazz_
       from
          Movie
  ) item0_
where
  item0_.id=?  

```  
- union all로 전체 하위 테이블을 다 찾는다.  
- INSERT 까진 심플했으나, 조회가 시작되면 굉장히 비효율적으로 동작한다.  
- 장점  
    - 서브 타입을 구분해서 처리할 때 효과적  
    - not null 제약조건 사용 가능  
- 단점  
    - 여러 자식 테이블 함께 조회시 성능이 느리다. (UNION SQL)  
    - 자식 테이블을 통합해 쿼리가 어려움  
    - 변경이라는 관점으로 접근할 때 굉장히 좋지 않다.  
        - 예를 들어, ITEM들을 모두 정산하는 코드가 있다고 가정할 때, ITEM 하위 클래스가 추가되면 정산 코드가 변경된다. 추가된 하위 클래스의 정산 결과를 추가하거나 해야 한다.  

### 결론 및 정리  
---  
- 기본적으로는 조인 전략을 가져가자.  

- 조인 전략과 단일 테이블 전략의 trade off를 생각해서 전략을 선택하자.  

- 굉장히 심플하고 확장의 가능성도 적으면 단일 테이블 전략을 가져가자. 그러나 비즈니스 적으로 중요하고, 복잡하고, 확장될 확률이 높으면 조인 전략을 가져가자.
