# 실전 예제 6 - 값 타입 매핑
> 작성자: @Hyunstone

## 목차
- [예제 코드](#예제-코드)

## 예제 코드
![image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/a750ada8-0a41-4fb8-beb8-4604a8ae2522)


### Address
```java
@Embeddable
public class Address {

    @Column(length = 10)
    private String city;
    @Column(length = 20)
    private String street;
    @Column(length = 5)
    private String zipcode;

    private String fullAddress() {
        return getCity() + " " + getStreet() + " " + getZipcode();
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getStreet() {
        return street;
    }

    private void setStreet(String street) {
        this.street = street;
    }

    private String getZipcode() {
        return zipcode;
    }

    private void setZipcode(String zipcode) {
        this.zipcode = zipcode;
    }

}
``````

### Member
```java
@Entity
public class Member extends BaseEntity {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;

    @Embedded
    private Address address;

    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Address getAddress() {
        return address;
    }

    public void setAddress(Address address) {
        this.address = address;
    }

    public List<Order> getOrders() {
        return orders;
    }

    public void setOrders(List<Order> orders) {
        this.orders = orders;
    }
}
```

### Delivery
```java
@Entity
public class Delivery extends BaseEntity {

    @Id @GeneratedValue
    private Long id;

    @Embedded
    private Address address;
    private DeliveryStatus status;

    @OneToOne(mappedBy = "delivery", fetch = FetchType.LAZY)
    private Order order;

}
```

equals hashCode 구현할 때

![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/301d7af0-8cfe-4355-b307-c022a9cdae83)

Use getters during code generation 옵션을 사용하는 게 좋다고 합니다. <br>
이 옵션을 사용하게 되면 getter를 호출하게 됩니다. 이것을 사용하지 않으면 필드에 직접 접근하게 됩니다. getter로 호출해야 프록시일때도 진짜 객체한테 값을 가져오게 됩니다.<br>
프록시를 사용하지 않는 경우라면 필드에 직접 접근해도 괜찮으나 안전한 코딩을 위해서는 getter를 통해서 값을 가져오는게 좋습니다.

값 타임의 또다른 장점으로는
```java
private String fullAddress() {
        return getCity() + " " + getStreet() + " " + getZipcode();
    }

public boolean isValid() {
        값 유효성 검사
    }
```
이렇게 의미있는 비즈니스 메소드를 만들 수 있다는 장점이 있습니다.