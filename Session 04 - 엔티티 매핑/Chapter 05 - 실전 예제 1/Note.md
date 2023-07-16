# 실전 예제 1 - 요구사항 분석과 기본 매핑
> 작성자: @hw130

## 목차
- [요구사항 분석](#요구사항-분석)  
- [기능 목록](#기능-목록)
- [도메인 모델 분석](#도메인-모델-분석)
- [엔티티, 테이블 차이점](#엔티티,-테이블-차이점)
- [테이블 설계](#테이블-설계)
- [엔티티 설계와 매핑](#엔티티-설계와-매핑)
- [데이터 중심 설계 문제점](#데이터-중심-설계-문제점)

### 요구사항 분석
---
- 회원은 상품을 주문할 수 있다.  
- 주문 시 여러 종류의 상품을 선택할 수 있다.  


### 기능 목록
---
- 회원 기능  
  - 회원 등록  
  - 회원 조회  
- 상품 기능  
  - 상품 등록  
  - 상품 수정  
  - 상품 조회  
- 주문 기능  
  - 상품 주문  
  - 주문내역조회  
  - 주문 취소  
 

### 도메인 모델 분석
---
- 회원과 주문의 관계: 회원은 여러 번 주문할 수 있다.(일대다)  
- 주문과 상품의 관계: 주문할 때 여러 상품을 선택할 수 있고 반대로 같은 상품도 여러 번 주문될 수 있다. 그런데 여기서 주문상품이라는 모델을 만들어서 다대다 관계를 일대다, 다대일 관계로 풀어낸다.  
<img width="871" alt="스크린샷 2023-07-16 오전 2 23 24" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/646bfb97-4abd-4d8a-a4f4-a16d92d5e8ea">

- 여기서 일대다, 다대다 관계란 각 테이블 사이에는 **관계** 라는 개념이 존재하는데, 이때 관계는 일대일(1:1), 일대다(1:N), 다대다(N:M) 관계가 있다고 표현할 수 있다.
<details>
<summary>1:1(일대일) 관계</summary>
<div markdown="1">       
1:1 관계란 어느 엔티티 쪽에서 상대 엔티티와 반드시 단 하나의 관계를 가지는 것을 말한다.  
예를 들어, 우리나라에서 결혼 제도는 일부일처제로, 한 남자는 한 여자와, 한 여자는 한 남자와 밖에 결혼을 할 수 없다.  
남편 또는 부인을 2명 이상 둘 수 없는데, 이러한 관계가 1:1 관계다.  
<img width="563" alt="스크린샷 2023-07-16 오후 12 07 32" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/00fb5c1a-4d87-46fe-8ab5-29088b847fb8">  
</div>
</details>  
<details>
<summary>1:N(일대다) 관계</summary>
<div markdown="1">       
1:N 관계는 한 쪽 엔티티가 관계를 맺은 엔티티 쪽의 여러 객체를 가질 수 있는 것을 의미한다.  
현실세계에는 1:N관계가 많이 있는데, 실제 DB를 설계할 때 자주 쓰이는 방식이다.  
예를 들어, 부모와 자식 관계를 생각해보면, 부모는 자식을 1명, 2명, 3명, 그 이상도 가질 수 있다.  
이를 부모가 자식을 소유한다고(has a 관계) 표현한다.  
반대로 자식 입장에서는 부모(아버지, 어머니의 쌍)를 하나만 가질 수 밖에 없다.  
이러한 관계를 1:N 관계라고 하며, 계층적인 구조로 이해할 수도 있다.  
<img width="490" alt="스크린샷 2023-07-16 오후 12 09 44" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/5b6785b2-b391-455c-876f-117b60b97de2">
</div>
</details>  
<details>
<summary>N:M(다대다) 관계</summary>
<div markdown="1">       
N:M 관계는 관계를 가진 양쪽 엔티티 모두에서 1:N 관계를 가지는 것을 말한다.  
즉, 서로가 서로를 1:N 관계로 보고 있는 것이다.  
예를 들어, 학원과 학생의 관계를 생각해보면, 한 학원에는 여러명의 학생이 수강할 수 있으므로 1:N 관계를 가진다.  
반대로 학생도 여러개의 학원을 수강할 수 있으므로, 이 사이에서도 1:M 관계를 가진다.  
그러므로 학원과 학생은 N:M 관계를 가진다고 할 수 있다.  
<img width="486" alt="스크린샷 2023-07-16 오후 12 11 32" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/e8fc2212-abd5-408c-ab8b-7856954e99ab">  
그런데 실무에서는 다대다 관계를 거의 사용하지 않는데, 이유를 간단히 말하자면 연결 테이블에 수많은 데이터가 들어가게 되어 복잡해질 수 있으므로 잘 사용하지 않는다.
</div>
</details>  


### 엔티티, 테이블 차이점  
---
- 엔티티와 테이블은 데이터베이스에서 사용되는 용어이다.  
> 엔티티 (Entity): 데이터베이스에서 특정한 사물이나 개념을 나타낸다.  
- 예를 들어, 사용자, 제품, 주문과 같은 실제 객체나 개념적인 요소가 될 수 있다.  
- 엔티티는 속성 (attribute)로 구성된다. 예를 들어, 사용자 엔티티는 이름, 나이, 이메일과 같은 속성을 가질 수 있다.  
- 데이터베이스에서 엔티티는 일반적으로 테이블로 표현된다.  
> 테이블 (Table): 데이터베이스에서 정보를 저장하기 위한 구조. 행(row)과 열(column)의 형태로 구성
- 테이블은 특정한 엔티티를 나타내며, 각 열은 해당 엔티티의 속성을 나타낸다. 예를 들어, 사용자 테이블은 이름, 나이, 이메일과 같은 열을 가질 수 있다.  
- 테이블은 데이터를 구조화하고 저장하는 데 사용되며, 관계형 데이터베이스에서 일반적으로 사용된다.

**즉, 엔티티는 특정한 사물이나 개념을 나타내는 개념적인 개체이고, 테이블은 데이터베이스에서 실제로 정보를 저장하는 구조이다.**  


### 테이블 설계
---  

<img width="868" alt="스크린샷 2023-07-16 오후 3 24 28" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/98e86e1b-8764-4b81-b10f-90bed9854523"> 


### 엔티티 설계와 매핑
---  

<img width="872" alt="스크린샷 2023-07-16 오전 11 31 14" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/4e3d1b40-9732-43bb-8d7d-0c6a979ecbc4">  

Member    

```@Entity
@Getter
@Setter
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    private String name;
    private String city;
    private String street;
    private String zipcode;
}
```
Order  

```@Entity
@Table(name = "ORDERS") // order 예약어 때문
@Getter
@Setter
public class Order {
    @Id
    @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;

    @Column(name = "MEMBER_ID")
    private Long memberId;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    //객체 지향 관점에서 설계
    @ManyToOne
    private Member member;

    @OneToMany
    private List<OrderItem> orderItem;
}
```  
Item  

```@Entity
@Getter
@Setter
public class Item {
    @Id
    @GeneratedValue
    @Column(name = "ITEM_ID")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;
}
```

OrderItem  

```@Entity
public class OrderItem {
    @Id
    @GeneratedValue
    @Column(name = "ORDER_ITEM_ID")
    private Long id;

    @Column(name = "ORDER_ID")
    private Long orderId;

    @Column(name = "ITEM_ID")
    private Long itemId;

    private int orderPrice;
    private int count;

    // 객체 지향 관점에서 설계
    @ManyToOne
    private Order order;

    @OneToMany
    private Item item;
}  
```  
JpaMain  
  
```public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpashop");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();

        tx.begin();

        try{


        }catch (Exception e){
            e.printStackTrace();
            tx.rollback();
        }finally {
            em.close();
        }
        emf.close();

    }
}
```  


### 데이터 중심 설계 문제점
---
- 객체 내부에 저장되는 데이터를 기반으로 시스템을 분할하는 방법  
- 객체가 포함해야하는 데이터에 집중  

```Order order = em.find(Order.class, 1L);
Long memberId = order.getMemberId();
Member member = em.find(Member.class, memberId);
```

주문에서 멤버를 찾기 위해서는 위와 같은 방법으로 찾아야 한다.  
그러나 위 방식은 테이블의 외래키를 객체에 그대로 가져와 객체 설계를 테이블 설계에 맞춘 방식이다.  
그래서 객체 그래프 탐색이 불가능하고, 참조가 없으므로 UML도 잘못됐다.  
즉, 객체지향스럽지 못한 설계 방식이다.  

<details>
<summary>UML이란?</summary>
<div markdown="1">       
> :프로그램 설계를 표현하기 위해 사용하는 표기법으로 요구분석, 시스템 설계, 시스템 구현 등 시스템 개발 과정에서 개발자간의 의사소통을 원활하게 하기 위해 표준화한 모델링 언어이다.  
</div>
</details>  

좀 더 객체지향적인 방법으로는 아래와 같이 구현할 수 있다.

```Order order = em.find(Order.class, 1L);
Member findMember = order.getMember();
```

```@Entity
@Table(name = "ORDERS")
@Getter
@Setter
public class Order {
//...
    private Member member;
//...
}
```

