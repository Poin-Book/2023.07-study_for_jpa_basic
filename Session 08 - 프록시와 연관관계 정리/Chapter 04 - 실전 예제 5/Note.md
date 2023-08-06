# 실전 예제 5 - 연관관계 관리
> 작성자: @destiny3912

## 목차
- [실전 예제 5 - 연관관계 관리](#실전-예제-5---연관관계-관리)
  - [목차](#목차)
    - [1. 글로벌 페치 전략 설정](#1-글로벌-페치-전략-설정)
    - [2. 영속성 전이 설정](#2-영속성-전이-설정)


### 1. 글로벌 페치 전략 설정
- 모든 연관 관계를 지연 로딩으로
- @ManyToOne, @OneToOne은 기본이 즉시 로딩이므로 지연 로딩으로 변경
        
     ```java
    @ManyToOne(fetch = FetchType.LAZY)
    @OneToOne(fetch = FetchType.LAZY)
    ```
        
### 2. 영속성 전이 설정
- Order → Delivery를 ALL로 설정
     - Order
            
        ```java
        @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
        @JoinColumn(name = "DELIVERY_ID")
        private Delivery delivery;
        ```
            
- Order → OrderItem을 ALL로 설정
    - Order
            
        ```java
        @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
        private List<OrderItem> orderItems = new ArrayList<>();
        ```
            
- 이렇게 하면 Order persist해줄때 다 같이 들어간다
- 다만 라이프 사이클 관리를 어떻게 하냐에 따라서 이런 구조는 달라져야 한다.
- 요구사항의 상황에 따라 다르게 가져가야한다.