# Mapped Superclass - 매핑 정보 상속
> 작성자: @Hyunstone

## 목차
- [@MappedSuperclass](#mappedsuperclass)
- [@Q&A](#qa)

## @MappedSuperclass
![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/23bcd1fa-37d6-42ea-955f-4eba116b5316)
공통 매핑 정보가 필요할 때 DB는 다르나 필드만 객체처럼 상속받아서 사용.<br>
중복같은 것들을 속성만 상속받아서 사용 → MappedSuperclass

![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/496ea17b-e7ee-4322-ae69-1486373cb6cc)

상속할 클래스에 위처럼 `@MappedSuperclass` 어노테이션을 붙이고, 상속받는 엔티티는 저 BaseEntity를 상속받으면 된다.

- 상속관계 매핑X
- 엔티티X, 테이블과 매핑X → BaseEntity는 엔티티가 아님.
- 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
- 조회, 검색 불가(em.find(BaseEntity) 불가)
- 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
- 테이블과 관계 없고, 단순히 엔티티가 공통으로 사용하는 매핑 정보를 모으는 역할
- 주로 등록일, 수정일, 등록자, 수정자 같은 전체 엔티티에서 공통으로 적용하는 정보를 모을 때 사용
- 참고: `@Entity` 클래스는 엔티티나 `@MappedSuperclass`로 지정한 클래스만 상속 가능

## Q&A
**Q: 꼭 모든 엔티티가 공유하는 컬럼만 적용할 수 있는 건가요? 필요에 따라 BaseEntity를 내려받을 엔티티들을 따로 골라서 그 녀석들만 extends를 하고, BaseEntity에는 걔네들끼리만 공통으로 가지는 컬럼을 넣는 게 더 자유로운 사용법이 아닌가 하는 생각이 듭니다.**

`A: 생각하신 내용이 맞습니다. 실제 개발할 때도, 공통 속성이 다 똑같지는 않기 때문에, 예를 들어서 어디는 등록일, 수정일이 모두 필요하고, 어디는 등록일만 필요하고 등등... 필요한 다양한 베이스 클래스들을 만들 수 있습니다.`

