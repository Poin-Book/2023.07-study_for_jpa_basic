# 값 타입의 비교
> 작성자: @joowojr

## 목차
### 값 타입: 인스턴스가 달라도 그 안에 값이 같으면 같은 것으로 봐야함
```java
int a = 10;
int b = 10;
// a == b -> true 값이 들어있기 때문에 참

Address a = new Address("city");
Address b = new Address("city");
// a == b -> false 참조가 들어있기 때문에 거짓
```
### 동일성 비교 : 인스턴스 참조 값을 비교, == 사용

### 동등성 비교 : 인스턴스의 값을 비교, equals()사용

### 값 타입은 a.equals(b)를 사용해서 동등성 비교를 해야 함 

- 값 타입의 equals() 메소드를 적절하게 재정의(주로 모든 필드 사용) 
- 이 때, equals()메소드의 기본은 ==을 사용하기 때문에 Override를 해야 한다.
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Address address = (Address) o;
    return Objects.equals(city, address.city) &&
            Objects.equals(street, address.street) &&
            Objects.equals(zipcode, address.zipcode);
}
@Override
public int hashCode() {
    return Objects.hash(city, street, zipcode);
}
```
  - 하지만 객체에 직접 접근해서 비교하도록 구현한다면 , 프록시 객체의 경우 제대로 비교를 할 수 없다.
    ```java
    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        Team team = (Team) o;
        return Objects.equals(id, team.id);
    }
    ```
    - 이렇게 equals를 재정의하게 된다면, 프록시 객체의 equals를 호출했을 때 문제가 생긴다.
    
    ``` java
    Team team = member.getTeam();
    Team sameTeam = member.getTeam();
    assertThat(team).isEqualTo(sameTeam);
    ``` 
    ![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/857b397d-57fd-49a3-b642-4aabe2baee0c)
    - 주소값이 같은 객체인데, equals 메서드를 통해 동등성을 비교하는 isEqualTo를 통과하지 못한다.
    - sameTeam을 인수로 하여 team의 equals를 호출하는데, getId가 아닌 메서드를 호출했으니 프록시 team이 초기화되고 프록시 team 내부의 **실제 객체**의 equals를 호출한다.
    - equals 안에서 호출되는 getClass의 결과는 Team.class인 반면, 인수로 전달받은 sameTeam의 getClass의 결과로는 프록시 타입이 나오게 되어 equals 결과값이 false가 나오게 된다.
    - 또한 프록시의 필드값은 모두 null이기 때문에 team.id는 사실 null이다. 그래서 Objects.equals(id, team.id)는 id값과 null을 비교하게 되어 false가 반환된다.
      ``` java
      @Override
      public boolean equals(Object o) {
          if (this == o) {
              return true;
          }
          if (!(o instanceof Team)) {
              return false;
          }
          Team team = (Team) o;
          return Objects.equals(id, team.getId());
      }
      ```
    - 따라서 o instanceof Team과 같은 형태로 코드를 작성하는 것이 좋다.
    - 필드값에 접근하지 않고 getId() 메소드를 사용하여 비교해준다.

      → equals() 메서드 재정의 시 "Use getters during code generation" 옵션을 사용하자.
      ![image](https://github.com/luke0408/study_for_jpa_basic/assets/85955988/a3302aac-0435-4071-a2df-af71aa4b1a3f)
  
      → 이 옵션을 사용하는 이유는 필드에 직접 접근하는 것이 아닌 getter()를 사용하여 접근하기 때문이다. 
  
      → Proxy 객체는 필드의 직접 접근이 불가능하다. 반드시 getter()를 호출해야 접근이 가능하다. 
  
      → 즉, Proxy 객체가 언제 사용될 지 확실하지 않으므로 반드시 해당 옵션을 사용한다.
    - hashCode()를 함께 재정의 하는 이유
      - hash를 사용하는 Collection(= HashMap, HashSet, HashTable)의 내부 비교 과정에서 알 수 있디다.
      - hash를 사용하는 Collection은 내부적으로 hashCode()와 equals()를 사용하여 객체의 내용을 비교한다.
      - Object 클래스에 정의된 hashCode()는 객체의 메모리 주소 값을 이용하여 hash 값을 생성한다.
      - 이처럼 객체의 메모리 주소 값을 기준으로 hash 값을 생성하면, 당연히 동일한 내용을 가진 객체라도 다른 hash 값을 가진다.
      - 즉, 동일한 내용의 객체 2개를 HashSet에 저장할 수 있게 되는 것이다. (이는 문제가 된다.)
      - 그러므로 객체의 메모리 주소가 아닌 객체의 내용을 이용하여 hash 값을 생성하도록 hashCode() 메서드를 재정의한다.
      - 해당 클래스의 객체가 hash를 사용하는 Collection과 함께 사용되지 않는다면 재정의할 필요는 없다.
  
