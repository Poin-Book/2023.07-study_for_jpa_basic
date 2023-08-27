# 다형성 쿼리
>
> 작성자: @luke0408

## 목차

- [다형성 쿼리](#다형성-쿼리)
  - [목차](#목차)
  - [다형성 쿼리 예제](#다형성-쿼리-예제)
  - [TYPE](#type)
  - [TREAT(JPA 2.1)](#treatjpa-21)

## 다형성 쿼리 예제

![image](https://github.com/luke0408/study_for_jpa_basic/assets/98688494/afa1c81c-6c3b-444d-8e7c-7ceb74151f13)

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn
public abstract class Item { // Item

    @Id @GeneratedValue
    private Long id;

    private String name;
    
    private int price;
    ...
}

@Entity
@DiscriminatorValue("A")
public class Album extends Item{ // Album

    private String artist;

    ...
}

@Entity
@DiscriminatorValue("M")
public class Movie extends Item { // Movie

    private String director;
    private String actor;

    ...
}

@Entity
@DiscriminatorValue("B")
public class Book extends Item { // Book

    private String author;
    private String isbn;

    ...
}
```

다음과 같이 조회하면 Item 엔티티의 자식도 함꼐 조회가 된다.

```java
String query = "select i from Item i";
List<Item> items = em.createQuery(query, Item.class).getResultList();
```

- 이때 단일 테이블 전략을 사용하여 실제 SQL 쿼리도 동일하게 나가게 된다.

## TYPE

- 조회 대상을 특정 자식으로 한정
- 예) Item 중에 Book, Movie를 조회해라

```java
String query = "select i from Item i where type(i) IN (Book, Movie)";
List<Item> items = em.createQuery(query, Item.class).getResultList();
```

```sql
select i from Item i where i.DTYPE in ('B', 'M')
```

## TREAT(JPA 2.1)

- JPA 2.1에 추가된 기능으로 자바의 타입 캐스팅과 비슷하다.
- 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용한다.
- JPA 표준은 FROM, WHERE 절에서만 사용할 수 있다.
- 하이버네이트는 SELECT 절에서도 사용할 수 있다.

<br>

- 예) 부모인 Item과 자식인 Book이 있다.

```java
String query = "select i from Item i where treat(i as Book).author = 'kim'";
List<Item> items = em.createQuery(query, Item.class).getResultList();
```

```sql
select i from Item i where i.DTYPE = 'B' and i.author = 'kim'
```

위 처럼 treat(i as Book)을 사용하면 Book으로 다운 캐스팅이 되어서 author를 사용할 수 있다.
