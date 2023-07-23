# 실전 예제 2 - 연관관계 매핑 시작
> 작성자: @Hyunstone

## 목차
 - [예제코드](#예제코드)
 - [Q&A](#Q&A)
 ---

 ## 예제코드
![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/729df6a5-4d10-4616-91c0-1af90f595169)


![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/dbc0b85d-1bd0-45c2-b638-968531a0b4aa)

 ``` java
 @Entity
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "MEMER_ID")
    private Member member;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    // 연관 편의 메서드
    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }

    ```
}
 ```

``` java
@Entity
public class OrderItem {

    @Id @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "ORDER_ID")
    private Order order;

    @ManyToOne
    @JoinColumn(name = "ITEM_ID")
    private Item item;

    private int orderPrice;
    private int count;

    ```
}
```

``` java
@Entity
public class Member {
    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String Name;
    private String city;
    private String street;
    private String zipcode;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    ```
}
```
<br>

```java
// 연관관계 O
Order order = new Order();
em.persist(order);

OrderItem orderItem = new OrderItem();
orderItem.setOrder(order);
em.persist(orderItem);
-------------------------------
// 연관관계 X
Order order = new Order();
order.addOrderItem(new OrderItem());
```

연관관계를 이용하지 않고 첫번째 코드처럼 단순히 이렇게 작성을 해도 된다.

그렇지만 개발상 편의를 위해 양방향 연관관계를 활용을 하게 된다.

JPQL 작성시 복잡한 쿼리를 사용하다보면 양방향 연관관계를 설정해주는 경우가 많다.

가능하면 최대한 단방향으로 하는게 좋다.
하지만 실무에선 조회를 편하게, JPQL 작성을 편하게 하기 위해 양방향 연관관계를 설정하게 된다.

연관관계에서 관심사를 잘 끊어내는게 중요하다.

꼬리에 꼬리를 물고 생각을 하다보면 끝이 없게 된다. 끊어줘야 한다는 것은 연관관계가 너무 복잡해지기 때문입니다.

## Q&A

1) 성능이 단방향, 양방향 매핑에서 차이가 많이 나는지 궁금합니다.

- **> 똑같습니다^^!**

2) 그냥 단방향 매핑으로 모든 거 처리하고, 필요할때만 sql join query 날려서 join해서 불러오면 되는거 아냐? 라는 생각도 드는데요..

- **> 네 그렇게 해도 됩니다^^!**

그런데 실제 개발을 해보면 복잡한 조회 쿼리에서 양방향 매핑을 하고 싶은 경우가 발생합니다.

예를 들어서 team 1:N member 이런 관계가 있을 때 team과 member를 fetch join으로 한번에 조회하고 싶은 경우가 있습니다.

이럴 때 양방향 매핑이 필요합니다.

이후 JPQL을 학습하게 되면 이런 상황에 연관관계가 없어도 Order를 별도로 조회하는 좋은 방법들을 이해하실 수 있습니다.