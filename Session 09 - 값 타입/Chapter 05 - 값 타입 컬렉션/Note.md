# 값 타입 컬렉션
> 작성자: @destiny3912

## 목차
- [값 타입 컬렉션](#값-타입-컬렉션)
  - [목차](#목차)
    - [1. 값 타입 컬랙션 정의](#1-값-타입-컬랙션-정의)
    - [2. 값 타입 컬랙션의 사용](#2-값-타입-컬랙션의-사용)
    - [3. 값 타입 컬렉션의 제약사항](#3-값-타입-컬렉션의-제약사항)
    - [4. 값 타입 컬렉션 대안](#4-값-타입-컬렉션-대안)
    - [5. 정리](#5-정리)
  

### 1. 값 타입 컬랙션 정의

![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/7cae9676-7e63-4c8b-a85b-4c2d80371372)

여기서 PK가 복합키인 이유는 인덱스를 부여해 식별하는 것은 엔티티가 되어버리기 때문이다 (테이블에 값만 저장이 되어야한다)

```java
@Entity
public class Member{
	//... 생략

	@Embedded
	private Address homeAddress;
	
	@ElementCollection
	@CollectionTable(name = "FAVORITE_FOOD", joinColumns = 
		@JoinColumn(name = "MEMBER_ID")
	)
	@Column(name ="FOOD_NAME")
	//예외적으로 값이 하나이기 때문에 이름을 지정해줘도 문제 없다
	private Set<String> favoriteFoods = new HashSet<>();
	
	@ElementCollection
	@CollectionTable(name = "ADDRESS", joinColumns = 
		@JoinColumn(name = "MEMBER_ID")
	)
	private List<Address> addressHistory = new ArrayList<>();

	//... 생략
}
```

- 실행
    - Member
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/e7586ca1-3c03-426e-9992-9576fdcfe6f7)
        
    - Favorite_Food
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/f329c783-dceb-4977-ad10-607146909623)
        
    - Address
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/cc29ce12-60b8-4a24-a114-3ad324601608)
        
- 값 타입을 하나 이상 저장할때 사용한다
- @ElementCollection. @CollectionTable사용한다
- 데이터 베이스는 컬렉션을 같은 테이블에 저장할 수 없다
- 컬렉션을 저장하기 위한 별도의 테이블이 필요하다 (일대다로 풀어서)

### 2. 값 타입 컬랙션의 사용

1. 값 타입 저장
    
    ```java
    Member member = new Member();
    
    member.setUsername("name");
    member.setHomeAddress(new Address(...));
    
    member.getFavoriteFoods().add("");
    member.getFavoriteFoods().add("");
    member.getFavoriteFoods().add("");
    
    member.getAddressHistory().add(new Address(...));
    member.getAddressHistory().add(new Address(...));
    
    em.persist(member);
    ```
    
    - 실행
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/1db0d6f9-b8d0-4c83-8774-bc726e02f412)
        
    - member만 persist했음에도 전부 다 같이 들어가는 것을 볼 수 있다
    - Collection이 다른 테이블임에도 불구하고
        - 그 이유는 값 타입이기 때문에
        - Life Cycle을 소속한 엔티티에 의존한다.
2. 값 타입 조회
    - 값 타입 컬렉션도 지연 로딩 전략 사용한다 → 어노테이션 fetch 기본값이 LAZY이다
    
    ```java
    Member findMember = en.find(Member.class, member.getId());
    ```
    
    - 실행
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/3494f7f1-9689-4aca-bfae-419646a9e70c)
        
        - 가져는 왔는데 Member만 가져온다.
        - 여기서 Collection들은 모두 지연 로딩이라는 것을 알 수 있다
    
    ```java
    //지연로딩 조회
    for(Address address : addressHistory) {
    	System.out.println("address = " + address.getCity());
    }
    ```
    
    - 실행
        
        ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/ea63f894-ff78-4d0a-b3e9-8dfbf7af93a2)
        
3. 값 타입 수정
    - 값타입은 setter로 변경 금지
    - 그냥 새로운 인스턴스로 통으로 갈아끼워야 한다.
    
    ```java
    Address a = findMember.getHomeAddress();
    findMember.setHomeAddress(new Address("newCity", a.getStreet(), a.getZipcode()));
    
    //치킨 -> 한식으로 바꾸기
    findMember.getFavoriteFoods().remove("치킨");
    findMember.getFavoriteFoods().add("한식");
    
    //old1 -> new1
    //이렇게하면 지워준다
    //왜? equals 메서드와 hashCode 메서드가 그렇게 구현이 되어있기 때문에 따라서 잘 구현해야한다.
    findMembmer.getAddressHistory().remove(new Address("old1", "street", "100000"));
    findMembmer.getAddressHistory().add(new Address("newCity", "street", "100000"));
    
    ```
    
    - 실행
        - 일반적인 값 타입
            
            ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/1dc64a1a-73c8-47a6-8e57-c81b88f9b5ed)
            
        - 값 타입 컬렉션
            - FavoriteFoods
                
                ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/b6658270-d323-4088-825e-53faeca07716)
                
                ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/204a428b-8165-45b9-9234-292ccf4dd139)
                
            - AddressHistory
                
                ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/740108f9-2959-4cf3-a9b6-2ad457e186f8)
                
                ![image](https://github.com/luke0408/study_for_jpa_basic/assets/74547868/c693fa09-1f8a-49b7-870b-5f7d545e2fff)
                
                - Member에 소속된 데이터를 다 지운다…
                - 그리고 삭제한 데이터 빼고 새로운 데이터와 기존 데이터를 전부 다시 넣어준다
                - 테이블을 완전히 갈아 끼운다고 생각하자
- 참고: 값 타입 컬렉션은 영속성 전이(Cascade) + 고아 객체 제거 기능을 필수로 가진다고 볼 수 있다.

### 3. 값 타입 컬렉션의 제약사항

- 값 타입은 엔티티와 다르게 식별자 개념이 없다
- 값은 변경하면 추적이 어렵다
- 값 타입 컬렉션에 변경 사항이 발생하면 
주인 엔티티와 연관된 모든 데이터를 삭제하고, 
값 타입 컬렉션에 있는 현재 값을 모두 다시 저장한다
    - 소속해있는 엔티티의 PK에 해당하는 값들을 모두 지우고
    - 다시 인서트 한다.
    - 근데 이렇게 복잡하게 쓸거면 다르게 써야한다.
- 값 타입 컬렉션을 매칭하는 테이블은 모든 컬럼을 묶어서 기본키를 구성해야 한다.
    - 그런데 JPA가 자동 생성을 해주면 PK를 그렇게 지정해주지 않는다
    - 따라서 직접 Create 명령으로 만들어주어야 함
    - null 입력 불가
    - 중복 저장 불가

### 4. 값 타입 컬렉션 대안

- 실무에서는 상황에 따라 값 타입 컬렉션 대신에 일대다 관계를 고려한다
- 일대다 관계를 위한 엔티티를 만들어 여기에서 값 타입을 사용한다.
- 영속성 전이(Cascade) + 고아 객체 제거를 사용해서 값 타입 컬렉션 처럼 사용한다.
- 다만 정말 단순한 경우에는 값 타입을 사용해도 무관하다 ( 알아서 잘 판단해서 사용하라)
- 에) AddressEntity → 이걸 이용해서 그냥 엔티티로 매핑을 한다. (값 타입을 엔티티로 승격 시킨다)
    
    ```java
    @Entity
    @Table(name = "ADDRESS")
    public class AddressEntity{
    	
    	@Id @GeneratedValue	
    	private Long id;
    	
    	//값 타입
    	private Address address;
    }
    
    // 이것을 Member에서 다음과 같이 매핑한다
    
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColunm(name = "MEMBER_ID")
    private List<AddressEntity> addressHistory = new ArrayList<>();
    ```
    

### 5. 정리

1. 엔티티 타입의 특징
    - 식별자 존재
    - 생명 주기를 관리한다
    - 공유가 가능하다
2. 값 타입의 특징
    - 식별자가 없음
    - 생명 주기를 엔티티에 의존한다
    - 공유하지 않는 것이 안전하다(복사에서 사용하라)
    - 불변 객체로 만드는 것이 안전하다

> 값 타입은 정말 값 타입이라 판단될 때에만 사용한다

> 엔티티와 값 타입을 혼동해서 엔티티를 값 타입으로 만들면 안된다

> 식별자가 필요하고 지속해서 값을 추적하고 변경해야 한다면 그것은 값 타입이 아닌 엔티티이다.
