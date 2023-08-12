# 값 타입과 불변 객체
> 작성자: @hw130

## 목차  
- [값 타입](#값-타입)  
- [값 타입 공유 참조](#값-타입-공유-참조)  
- [값 타입 복사](#값-타입-복사)  
- [객체 타입의 한계](#객체-타입의-한계)  
- [불변 객체](#불변-객체)  
- [불변 객체의 값 수정](#불변-객체의-값-수정)  
### 값 타입  
---  
> 값 타입은 복잡한 객체 세상을 조금이라도 단순화하려고 만든 개념이다. 따라서 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.  

### 값 타입 공유 참조  
---  
<img width="775" alt="스크린샷 2023-08-12 오후 11 04 45" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/3e030f0c-2a5b-4de6-bedf-488c2c24e92e">  
- 임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험함  
- 예상치 못한 곳에서 문제가 발생하는 **부작용(side effect)** 발생  
같은 주소를 회원 1,2한테 넣어주고 회원 2의 주소를 바꾼다면?  
```  
// 같은 Address 값을 세팅한다.
Address address = new Address("city", "street", "10012");

Member member1 = new Member();
member1.setName("member1");
member1.setHomeAddress(address);
em.persist(member1);

Member member2 = new Member();
member2.setName("member2");
member2.setHomeAddress(address);
em.persist(member2);

// member2의 City를 newCity로
member2.getHomeAddress().setCity("newCity");
```  
<img width="609" alt="스크린샷 2023-08-12 오후 11 18 18" src="https://github.com/hw130/Algorithm_practice/assets/87763333/99e19e84-5b77-4ea6-b789-082de6a84fc3">  
업데이트가 2번 나가고  
<img width="711" alt="스크린샷 2023-08-12 오후 11 18 58" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/59e7da6c-4ea2-45e1-93ad-9d4a507aceca">  
이 경우 member2의 주소를 “NewCity”로 변경하길 원했지만, member1의 주소도 같이 변경된다.  
- 영속성 컨텍스트에서는 회원1과 회원2 둘 다 city 속성이 변경된 것으로 판단하여 각각 UPDATE SQL을 실행  
이처럼 공유 참조로 인해 발생하는 문제는 찾아내기가 어렵다. 이 경우 값을 복사해서 사용해야 한다.  

### 값 타입 복사  
---
값 타입의 실제 인스턴스 값을 공유하는 것은 예상치 못한 곳에서 문제가 발생할 수 있다.  
따라서, 값(인스턴스)을 복사해서 사용해야 한다.  
<img width="739" alt="스크린샷 2023-08-12 오후 11 05 44" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/d58ae955-784d-485c-a977-1a7e05b0b3ab">  
```  
Address address = new Address("city", "street", "10000");

Member member1 = new Member();
member1.setName("member1");
member1.setHomeAddress(address);
em.persist(member1);

Address copyAddress = new Address(address.getCity(), address.getStreet(), address.getZipcode());

Member member2 = new Member();
member2.setName("member2");
member2.setHomeAddress(copyAddress);
em.persist(member2);

// member2의 City를 newCity로
member2.getHomeAddress().setCity("newCity");
```  
<img width="714" alt="스크린샷 2023-08-12 오후 11 19 43" src="https://github.com/luke0408/study_for_jpa_basic/assets/87763333/3598767f-78fb-449f-b23b-d02006c078f2">  

- 공유 참조를 하지 않기 위해 해당 참조 객체를 복사해서 사용한다.  
    - 공유 참조로 인해 발생하는 부작용을 피할 수 있다.  

### 객체 타입의 한계  
---  
- 항상 값을 복사해서 사용하면 공유 참조 때문에 발생하는 부작용을 피할 수 있다.  
- 그런데 임베디드 타입 등 직접 정의한 타입은 자바 기본 타입이 아니라 객체 타입이다.  
  - primitive 같은 값 타입은 =으로 할당해도 복사해서 들어가서 두 변수를 한 번에 수정하는 사이드 이펙트가 발생하지 않는다.  
  - 객체 타입은 참조 값을 직접 가지므로 둘 다 값이 바뀌는 사이드 이펙트를 막을 방법이 없다.  
- 즉, 객체의 공유 참조는 피할 수 없다.  
```
// 기본 타입
int a = 10;
int b = a; // 기본 타입은 값을 복사
b = 4; // a = 10, b = 4

// 객체 타입
Address a = new Address("Old");
Address b = a; // 객체 타입은 참조를 전달
b.setCity("New"); // a와 b 모두 New로 변경  
```
기본 타입은 값을 복사하기 때문에 문제가 없지만, 객체 타입은 참조를 전달하기 때문에 변경하면 둘 다 반영되는 부작용이 발생한다.  
### 불변 객체  
---
불변 객체는 생성 시점 이후 절대 값을 변경할 수 없는 객체를 의미한다.  
값 타입은 부작용 걱정 없이 사용할 수 있어야 한다. 따라서 객체를 불변하게 만들어 애초에 수정할 수 없도록 할 필요가 있다.  
- 값 타입은 불변 객체(immutable object)로 설계해야 한다.  
- 생성자로만 값을 수정하고 수정자(Setter)를 만들지 않으면 된다.  
즉, 값 타입을 생성자로만 값을 설정하고, Setter를 선언하지 않는 것으로 불변 객체로 만들 수 있다.  
- 부작용 문제 해결  
### 불변 객체의 값 수정  
---
불변 객체의 값을 수정해야 할 경우에는 새로운 객체를 생성하여 필요한 부분만 수정해야 한다.  
```
Address address = new Address("city", "street", "10000");

Member member1 = new Member();
member1.setName("member1");
member1.setHomeAddress(address);
em.persist(member1);

// city를 바꾸고 싶다면 setCity() 대신 
// Address 객체를 새로 만들어서 통으로 갈아 끼운다.
Address newAddress = new Address("NewCity", address.getStreet(), address.getZipcode());
member1.setHomeAddress(newAddress);
```  
- 불변이라는 작은 제약으로 부작용이라는 큰 재앙을 막을 수 있다.
