# 일대일 [1:1]
> 작성자: @asas6978

## 목차
- [일대일 관계](#일대일-관계)
- [일대일: 주 테이블에 외래 키](#주-테이블에-외래-키)
- [일대일: 대상 테이블에 외래 키](#대상-테이블에-외래-키)

## 일대일 관계

- 일대일 관계는 그 반대도 일대일; 주 테이블이나 대상 테이블 중 어느 쪽에서든 외래 키를 선택할 수 있음  
  - 주 테이블에 외래 키
  - 대상 테이블에 외래 키
  - 양방향을 만들고 싶다면 양쪽 테이블에 각각 외래키를 추가해주면 됨
    
- DB 입장에서는 외래 키에 데이터베이스 유니크 제약조건을 추가해주어야 일대일 관계가 성립됨
  - 이유
    - 정확성 보장: 유니크 제약조건을 설정함으로써 한 테이블의 레코드와 다른 테이블의 레코드 간의 일대일 관계를 정확하게 보장할 수 있음
    - 중복 데이터 방지: 유니크 제약조건이 없으면 테이블 내 여러 레코드가 동일한 외래키 레코드와 연결될 가능성이 존재하는데, 이런 경우 중복 데이터가 발생할 수 있으며 무결성이 깨질 수 있음 -> 유니크 제약조건을 통해 중복 데이터를 방지
      

## 주 테이블에 외래 키
<img width="400" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/e9a55ace-1af9-4b38-a5af-8767fa8e2cbd">
<img width="400" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/867bb295-f406-4278-86b4-40612ebce8bc">

- 주 객체가 대상 객체의 참조를 가지는 것처럼 주 테이블에 외래 키를 두고 대상 테이블 탐색
- 단방향의 경우 다대일(@ManyToOne) 단방향 매핑과 유사하고, 양방향의 경우 외래 키가 있는 곳이 연관관계의 주인이며 반대편은 mappedBy 적용
<br>

- 장점: 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
- 단점: 값이 없으면 외래 키에 null 허용
  

## 대상 테이블에 외래 키
<img width="400" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/2d611cfd-f332-4816-87fd-8f4fdf4cfe5e">
<img width="400" alt="image" src="https://github.com/luke0408/study_for_jpa_basic/assets/77332981/6c29bd85-10b8-4fac-b4c6-54de29c0759e">

- 대상 테이블에 외래 키가 존재
- 단방향 관계는 JPA에서 지원하지 않으며, 양방향 관계는 지원
  - 객체지향 모델과 데이터베이스 모델 간의 차이점을 양쪽 테이블 모두에 외래 키를 설정하는 방식을 사용하여 양방향 관계로 매핑을 수행하면, 객체지향적인 모델과 데이터베이스 모델 간의 불일치를 최소화할 수 있음
<br>

- 장점: 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
- 단점: 프록시 기능의 한계로 지연 로딩으로 설정해도 항상 즉시 로딩됨
