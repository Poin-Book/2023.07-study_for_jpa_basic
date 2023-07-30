# 다대다 [N:M]
> 작성자: @luke0408

## 목차

## 관계형 데이터베이에서의 다대다 관계
- 다대다 관계는 관계형 데이터베이스에서는 불가능하다.
- `연결 테이블`을 추가해서 일대다, 다대일 관계로 풀어내야 한다.

<center>

![N:M in DBMS](https://github.com/luke0408/study_for_jpa_basic/assets/98688494/1f5abe00-63fc-4b89-94cb-9fc33e3deda0)

</center>

## 객체에서의 다대다 관계
- 객체는 컬렉션을 사용해서 객체 2개로 다대다 관계를 가질 수 있다.
- 즉, 우리는 객체 사이의 매핑을 통해 다대다 관계를 표현해야함.
  - 이때 객체에서도 `연결 테이블`의 형태를 구현해야함.

<center>

![N:M in Object](https://github.com/luke0408/study_for_jpa_basic/assets/98688494/38db68e5-859a-4518-bbe0-c0573383f26c)

</center>

### 관련 어노테이션
- `@ManyToMany`
  - 다대다 관계를 매핑할 때 사용한다.
  - `@JoinTable`을 사용해서 연결 테이블을 지정해야 한다.

- `@JoinTable`
  - 연결 테이블을 지정한다.
  - `name`: 매핑할 연결 테이블의 이름

### 예제
- Member.java
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT")
    private List<Product> products = new ArrayList<>();
    ...
}
```

- Product.java
```java
@Entity
public class Product {
    @Id @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "poducts")
    private List<Member> members = new ArrayList<>();
    ...
}
```

## 다대다 매핑의 한계
- 다대다 매핑은 단순하지만, 실무에서 사용하기에는 한계가 있다.
- 연결 테이블이 단순히 연결만 하고 끝나지 않는다.
  - 주문 시간, 수량 같은 데이터가 추가될 수 있어야 한다.

## 다대다 매핑의 한계 극복
- 연결 테이블용 엔티티 추가
  - 연결 테이블을 엔티티로 승격시킨다.
  - 연결 테이블에 데이터를 추가할 수 있다.
  - 연결 테이블을 더 세밀하게 컨트롤할 수 있다.
- @ManyToMany -> @OneToMany, @ManyToOne

<center>

![N:M Table](https://github.com/luke0408/study_for_jpa_basic/assets/98688494/cb701602-5056-4708-a4d5-cec7ce813cf0)

</center>

### 예제
- Member.java
```java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String username;

    @OneToMany(mappedBy = "member")
    private List<MemberProduct> memberProducts = new ArrayList<>();
    ...
}
```

- Product.java
```java
@Entity
public class Product {
    @Id @GeneratedValue
    @Column(name = "PRODUCT_ID")
    private Long id;

    private String name;

    @OneToMany(mappedBy = "product")
    private List<MemberProduct> memberProducts = new ArrayList<>();
    ...
}
```

- MemberProduct.java
```java
@Entity
public class MemberProduct {
    @Id @GeneratedValue
    @Column(name = "MEMBER_PRODUCT_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;

    private int orderAmount;
    ...
}
```
