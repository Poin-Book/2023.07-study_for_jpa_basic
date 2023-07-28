# 실전 예제 3 - 다양한 연관관계 매핑
> 작성자: @destiny3912

## 목차
- [실전 예제 3 - 다양한 연관관계 매핑](#실전-예제-3---다양한-연관관계-매핑)
  - [목차](#목차)
    - [1. 엔티티](#1-엔티티)
    - [2. 엔티티 상세](#2-엔티티-상세)
    - [3. ERD](#3-erd)
    - [4. Code](#4-code)
    - [5. @ManyToMany를 안쓰는 이유](#5-manytomany를-안쓰는-이유)
    - [6. 연관관계 어노테이션들의 Property](#6-연관관계-어노테이션들의-property)

### 1. 엔티티

- 주문과 배송은 1:1 이며 상품과 카테고리는 N:M 이다

![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/b97934ce-d317-44c2-8be9-ca411be9bf03)

### 2. 엔티티 상세

![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/bff95a1b-00fd-43a6-8e10-54ec289796a9)

### 3. ERD

![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/da0b0d7f-465b-4e8e-abed-2dc80b7bc66c)

### 4. Code

1. Order.java
    
    ```java
    @Table(name = "ORDERS")
    @Entity
    public class Order {
        @Id
        @GeneratedValue
        @Column(name = "ORDER_ID")
        private Long id;
    		
    	//한명의 유저가 주문을 여러번 할 수 있다 (연관관계의 주인)
        @ManyToOne
        @JoinColumn(name = "MEMBER_ID")
        private Member member;
    		
    	//한 주문은 배달을 한번만 한다 (연관관계의 주인)
        @OneToOne
        @JoinColumn(name = "DELIVERY_ID")
        private Delivery delivery;
    		
    	//하나의 주문에는 상품이 여러개 들어간다. (연관관계 주인이 아님)
        @OneToMany(mappedBy = "order")
        private List<OrderItem> orderItems = new ArrayList<>();
    
        private LocalDateTime orderDate;
    
        @Enumerated(EnumType.STRING)
        private OrderStatus status;
    
        /**
         * 연관관계 편의 메서드
         * 주문에 상품을 넣을때 연관관계를 한번에 업데이트 해줌
         * */
        public void addOrderItem(OrderItem orderItem) {
            orderItems.add(orderItem);
            orderItem.setOrder(this);
        }
    }
    ```
    
2. Delivery.java
    
    ```java
    @Entity
    @Table(name = "DELIVERY")
    public class Delivery {
    
        @Id
        @GeneratedValue
        private Long id;
    		
    	//하나의 주문은 한번만 배달된다 (연관관계의 주인이 아님)
        @OneToOne(mappedBy = "delivery")
        private Order order;
    
        private String city;
    
        private String street;
    
        private String zipcode;
    
        private DeliveryStatus status;
    }
    ```
    
3. OrderItem.java
    
    ```java
    @Entity
    public class OrderItem {
    
        @Id
        @GeneratedValue
        @Column(name = "ORDER_ITEM_ID")
        private Long id;
    		
    	//하나의 상품은 여러번 주문될 수 있다(연관관계의 주인)
        @ManyToOne
        @JoinColumn(name = "ITEM_ID")
        private Item item;
    		
    	//하나의 주문은 여러개의 주문된 상품이 있다(연관관계의 주인)
        @ManyToOne
        @JoinColumn(name = "ORDER_ID")
        private Order order;
    
        @Enumerated(EnumType.STRING)
        private OrderStatus status;
    }
    ```
    
4. Item.java
    
    ```java
    @Entity
    public class Item {
    
        @Id
        @GeneratedValue
        @Column(name = "ITEM_ID")
        private Long id;
    
        private String name;
    
        private int price;
    
        private int stockQuantity;
    		
    	//상품과 카테고리는 서로가 서로를 여러개를 가질 수 있다.
    	//중간 테이블이 필요함
        @ManyToMany(mappedBy = "items")
        private List<Category> categories;
    }
    ```
    
5. Category.java
    
    ```java
    @Entity
    public class Category {
        @Id
        @GeneratedValue
        private Long id;
    
        private String name;
    
        @ManyToOne
        @JoinColumn(name = "PARENT_ID")
        private Category parent;
    
        @OneToMany(mappedBy = "parent")
        private List<Category> child = new ArrayList<>();
    
        @ManyToMany
        //중간 조인 테이블 잡기
    	//조인 테이블의 이름이 CATEGORY_ITEM 이다
    	//ManyToMany를 안쓰는 이유가 이거다
        @JoinTable(
                name = "CATEGORY_ITEM",
                joinColumns = @JoinColumn(name = "CATEGORY_ID"),
                inverseJoinColumns = @JoinColumn(name = "ITEM_ID")
        )
        private List<Item> items = new ArrayList<>();
    }
    ```
    
6. SQL
    
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/b6ac45a5-a639-48b2-af12-97589adcca50)

    

### 5. @ManyToMany를 안쓰는 이유

- 실전에서는 중간 테이블이 단순하지 않다
    - 여러가지 테이블이 서로 엮여 있을것이기 때문이다.
    - 또한 단순히 연결만하고 끝나지 않는다
- 단순히 연결만 하고 끝나지 않는 것에서
    - 중간 테이블에 필드 추가가 불가능하다.
        - 왜? 엔티티로 정의를 안했으니까
    - 따라서 실제 필요한 테이블과 일치하지 않는다.
- 이러한 한계를 극복하기 위해서는
    - 중간 테이블을 엔티티로 명확히 정의해주면 된다
    - 단, PK는 따로 생성해준다.

### 6. 연관관계 어노테이션들의 Property

1. @JoinColumn
    
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/e010dc6c-083d-4188-a937-52fecc0f52e7)
    
2. @ManyToOne → 무조건 연관관계의 주인이어야 함
    
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/3ce30902-6ce0-4a21-981c-50296036f1d0)
    
3. @OneToMany
    
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/4c08fbb7-29ec-47c8-bda2-27ccc3d3a3ef)
