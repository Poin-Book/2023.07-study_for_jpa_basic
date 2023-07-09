# SQL 중심적인 개발의 문제점
> 작성자: 다나

## 목차
1. 객체 CRUD
2. 객체 vs 관계형 데이터베이스

## 1. 객체 CRUD
- 필드 추가를 하려면 INSERT, DELETE, UPDATE 문에 모두 해당 필드를 추가해야함 -> 복잡하다
- SQL에 의존적인 개발을 피하기 어렵다.

## 2. 객체 vs 관계형 데이터 베이스
- 객체를 저장하는 다양한 형태의 데이터 베이스가 있지만, 현실적인 대안은 관계형 데이터 베이스다.
- 관계형 데이터 베이스는 객체를 SQL 변환하여 저장한다.
- 객체와 관계형 데이터베이스의 차이

1. 상속
    - 객체 상속 관계
        - 데이터 삽입할 때 연관된 모든 테이블에 일일이 삽입해줘야함
        - 각각의 객체 생성해야함
        - 매우 복잡하기 때문에 DB에 저장할 객체에는 상속관계를 사용하지 않는다.
    - Table 슈퍼타입 서브타입 관계
        - 자바 컬렉션에 저장하면 부모 타입으로 조회 후 다형성 활용
2. 연관관계
    - 객체는 참조를 사용 : member.getTeam()
    - 테이블은 외래키를 사용 : JOIN ON M.TEAM_ID = T.TEAM_ID
    - 객체를 테이블에 맞추어 모델링
    - 객체다운 모델링을 하게 되면 과정 복잡
        - SQL문 작성 - SQL 실행 - 객체 생성하여 DB에서 조회한 테이블 정보 입력 - 객체 생성하여 DB에서 조회한 참조 테이블 정보 입력 - 두 테이블 간의 관계 설정
    - 객체 모델링을 자바 컬렉션에 관리하면 훨씬 간단해진다.
      <이미지>
3. 객체 그래프 탐색
   - 객체는 자유롭게 객체 그래프를 탐색할 수 있어야 한다.
   - 처음에 실행하는 SQL에 따라 탐색 범위가 결정되므로 엔티티에 대한 신뢰 문제가 생긴다.
   - 하지만 모든 객체를 미리 로딩할 수는 없다. 너무 많은 쿼리가 나와 성능이 떨어짐
   - 상황에 따라 동일한 회원 조회 메소드를 여러벌 생성하는 방법이 있지만, 이또한 너무 많은 케이스가 나오기 때문에 복잡하다
   - 계층형 아키텍처 -> **진정한 의미의 계층 분할이 어렵다**
4. 비교하기
    ```java
    String memberId = "100";
    Member member1 = memberDAO.getMember(memberId);
    Member member2 = memberDAO.getMember(memberId);
    member1 == member2; //다르다.

    class MemberDAO {
        public Member getMember(String memberId) {
        String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
        ...
        //JDBC API, SQL 실행
        return new Member(...);
            }
    }
    ```
    - SQL 문으로 데이터를 얻을 시에 데이터는 같지만 다른 인스턴스를 가짐

    
    ```java
    String memberId = "100";
    Member member1 = list.get(memberId);
    Member member2 = list.get(memberId);
    member1 == member2; //같다.
    ```
   - 자바 컬렉션에서 조회, 같은 인스턴스이다.


객체 지향적으로 모델링 할수록 매핑 작업만 늘어난다.
### 이러한 문제점을 해결하기 위해 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수 있게 한 것이 JPA

     

