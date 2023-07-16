# 필드와 칼럼 매핑
> 작성자: @Hyunstone

## 목차
- [@Column](#column)
- [@Enumerated](#enumerated)
- [@Temporal](#temporal)
    - [Date? LocalDate?](#date-localdate)
- [@Lob](#lob)
---

```java
@Column(name="") //컬럼 매핑 
@Enumerated(EnumType.STRING) //enum 타입 맵핑
@Temporal(TemporalType.TIMESTAMP)
@Lob //DB에 큰 컨텐츠를 넣고 싶을 때 blob, clob 사용
@Transient // 디비랑 신경쓰지 않고 메모리에서만 계산하기 위해서 사용하는 임시 데이터 해시 데이터
```

## @Column

**name** - 필드랑 매핑할 테이블 컬럼 이름

**insertable, updatable** - 등록, 변경 가능 여부(칼럼 변경 시). default는 true

```java
@Column(name = "test", updatable=false) // 업데이트 안됨
```

**nullable(DDL)** - null 값의 허용 여부 설정, false로 설정하면 DDL 생성 시 Not null 제약 조건. 이것도 default = true

**unique(DDL)** - unique 제약 조건 걸고 싶을 때 사용. 랜덤으로 이름을 생성하기 때문에 이보다는@Table(uniqueConstraints = #) 를 권장

**columnDefinition(DDL)** - 데이터베이스 컬럼 정보 직접 줄 수 있습니다.

```java
columnDefinition = "varchar(100) default ‘EMPTY'"
```

**length(DDL)** - 문자 길이 제약조건 String 타입만 가능합니다.

**precision, scale(DDL)** - BigDecimal 이나 소숫점을 사용할 때 사용합니다.

## @Enumerated()

![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/f38984e4-2c16-4fdc-8c5c-44c3a2cea6a2)

**EnumType.ORDINAL**

enum 순서를 데이터베이스에 저장하며 DB에 숫자로 들어갑니다. enum 변경 시 혼동을 일으킬 위험이 있어 ORDINAL을 사용하지 않는 것을 권고.

**EnumType.STRING**

enum 이름을 데이터베이스에 저장합니다.</br>
varchar 255 들어가고 enum에 있는 스트링 값 그대로 들어갑니다.
</br>순서에 의한 문제가 없다면 string으로 넘기는 것을 권장.

## @Temporal

날짜, 시간, timestamp를 DB Table에 저장하는 데 사용할 수 있는 JPA 어노테이션입니다.

**1. DATE** (java.sql.Date)

**2. TIME** (java.sql.Time)

**3. TIMESTAMP** (java.sql.Timestamp)

```java
@Temporal
private Date joinedDate;
```

LocalDate, LocalDateTime을 사용할 때는 어노테이션 생략 가능(최신 하이버네이트 지원)

```java
private LocalDate testLocalDate;
private LocalDateTime testLocalDateTime;
```

![Image](https://github.com/luke0408/study_for_jpa_basic/assets/110045522/b2b864cc-1a2a-46af-9146-cf334819b451)

## **+) Date? LocalDate?**

- `Date`는 JDK1.0, `Calendar`는 JDK1.1 부터 제공되었다.
- JDK1.8부터 `java.time 패키지`로 Calendar와 Date의 단점을 개선한 클래스들이 추가

### **기존 Java의 날짜 API의 문제.**

Java 8 이전에 사용하던 Date 관련 클래스는 `Date`, `Calendar`, `SimpleDateFormat` 등이 있었으나, 많은 문제가 있어 자바 8 버전 이후부터는 새로운 날짜 관련 API를 제공.
기존 클래스들의 문제는 다음과 같습니다.

**1. 부적절한 클래스와 메서드 이름을 가집니다.**

Date 클래스의 경우, TimeStamp 방식으로 동작하고 시간을 내재하고 있으나, ClassName은 Date입니다.

**2. Thread saftety 하지 않습니다.(not immutable)**

Date 클래스의 경우 mutable 하기 때문에 다른 Thread에서 값을 참조하고 변경할 수 있습니다. 즉, Thread Safe 하지 않습니다.

**3. 버그가 발생할 여지가 많습니다.**

Calendar 클래스의 경우 입력값의 month가 0이 1월로 처리됩니다. 그래서 Calendar.SEPTEMBER 같은 상수를 사용해야 하며, DB 데이터랑 연결하면서 서로 다르게 해석됩니다.

java.sql.Date 클래스는 상위 클래스인 java.util.Date 클래스와 이름이 같다. 이 클래스를 두고 Java 플랫폼 설계자는 클래스 이름을 지으면서 깜빡 존 듯하다는 조롱까지 나왔다.

### Joda Time

위의 여러한 문제들이 있어 Java8 이전에서는 Joda-Time이라는 라이브러리를 사용했으나, 

Java 8부터는 Joda-Time이 자바 표준 라이브러리로 들어왔습니다.

Date & Time API의 목표

- date, time, instant, time-zone을 포함하는 공식 시간 개념 지원
- immutable 구현
- 개발자의 사용성에 중점을 둔 JDK에 적절하고 효과적인 API 제공
- 기존의 JDK API와의 통합
- 제한된 calendar 시스템 세트를 제공하고 다른 것들로 확장 가능
- ISO-8601, CLDR 및 BCP47을 포함한 관련 표준 사용
- UTC에 연결하여 명시적 시간 척도를 기반

다중 서버 환경에서 프로젝트를 개발하고 있는데 인프라팀에서는 분명 서버 간 시간을 동기화하였다고 했는데도 불구하고 LocalDateTime.now()를 로그로 찍어보면 서버 간의 시간이 조금씩 다른 것을 확인할 수 있었습니다.

### **Instant vs LocalDateTime**

Instant 클래스와 LocalDateTime 클래스는 비슷해 보이지만 사실 완전히 다릅니다.
하나는 순간(moment)을 나타내고, 다른 하나는 순간(moment)을 나타내지 않습니다.

### ****LocalDateTime****

LocalDateTime 클래스는 타임존을 가지고 있지 않습니다. 이 클래스는 시간대를 저장하거나 나타내지 않습니다. 대신, 이것은 생일과 같은 **날짜**와 시계에 보이는 **현지 시간**을 결합한 것입니다.</br>
**오프셋과 표준시와 같은 추가 정보가 제공되지 않는 한 타임라인의 특정 지점(Instant)을 나타낼 수 없습니다.**

따라서, Local은 타임존, 그리고 오프셋이 없음을 의미합니다.

### ****Instant****
Instant는 UTC의 타임라인에 있는 한 순간(moment)으로, 1970년 UTC의 첫 번째 모멘트의 발생 이후 nano초 동안의 시간입니다.

대부분의 비즈니스 로직, 데이터 스토리지 및 데이터 교환은 UTC여야 하므로 Instant는 자주 사용할 수 있는 편리한 클래스입니다.

LocalDateTime, LocalDate, LocalTime은 Instant와 성격이 다릅니다. 해당 클래스들은 한 지역이나 시간대에 묶이지 않으며 어느 한 지역 혹은 타임존에 얽매이지 않습니다. 즉, 지역성을 부여하기 전에는 의미를 지니지 않은 클래스들이라고 할 수 있습니다.

클래스에 공통적으로 있는 "Local"이라는 단어는 어떤 지역, 혹은 모든 지역을 의미하지만, **특정한 지역을 의미하지는 않습니다.**

따라서 실제 런칭된 서비스에서는 위에서 언급된 세 가지 클래스가 타임라인의 특정 순간이 아닌 **대략적인 날짜 혹은 시간을 나타내기 때문에 자주 사용되지 않습니다.** 상용에 런칭된 서비스는 송장 도착 시간, 운송을 위해 제품을 발송한 시간, 직원을 고용한 시점, 택시가 차고에서 출발하는 정확한 시간 등을 정확히 저장해야 합니다. 

**따라서 비즈니스 앱 개발자들은 Instant 및 ZonedDateTime 클래스를 가장 많이 사용합니다.**

### 그렇다면 LocalDateTime은 언제 사용해야 될까요?

LocalDateTime을 사용해야 하는 세 가지 상황의 예시는 아래와 같습니다:

- 특정 날짜와 시간을 여러 위치에서 적용하려는 경우
- 예약을 하는 경우
- 타임존이 정해지지 않은 경우

![Image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbuhbpa%2FbtqUGU5kaIq%2F7cADVTBiTdiANwjmfkOb8K%2Fimg.png)

정리를 하자면, 로그를 남기거나 현재 시간을 불러올 때 LocalDateTime 클래스를 사용하는 것은 추천되지 않는 방법인 것 같습니다.

stackoverflow에서 추천을 많이 받은 답변에 의하면 위와 같은 경우 UTC를 사용하는 것을 권장하기 때문에 LocalDateTime이 아닌 Instant 클래스를 사용하는 것이 맞는 방법이라고 합니다. 또한, java.time.* 클래스들이 문자열을 파싱 하거나 생성할 때 [ISO 8601 포맷](https://en.wikipedia.org/wiki/ISO_8601)을 디폴트로 사용하기 때문에 로그를 남기기 위해 텍스트 형태로 직렬화할 때 해당 포맷을 사용하는 것을 권장한다고 합니다.

만약 위와 같은 방법으로도 해결이 되지 않는다면 공통 NTP 서버를 구축하여 모든 서버가 해당 서버의 시간대를 사용하는 방법으로 문제를 해결해야 할 것 같습니다.

Instant 객체와 LocalDateTime 모두 systemUTC() 메서드를 쓰고 있기 때문에 고정된 clock이 아닌 best available system clock 즉, 상황에 따라서 다른 clock을 사용하고 있다고 정의되어있습니다.

**따라서, 현재로서는 ntp 서버를 구축하기 전까지 기존대로 clock 중 하나인 System.currentTimeMillis()를 사용하는 것이 최선의 선택인 것 같습니다.**

그리고 우리나라처럼 전국적으로 타임존이 다르지 않을 경우 LocalDateTime 객체를 사용해도 무방한 듯 합니다.
하지만, 글로벌 서비스를 런칭한다면 Instant 객체를 사용하는 것이 최선인 것 같습니다!

https://jaimemin.tistory.com/1537</br>
https://ryan-han.com/post/java/java-calendar-date/</br>
https://d2.naver.com/helloworld/645609</br>
https://java119.tistory.com/52</br>
https://dev.gmarket.com/49

## @Lob

매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑해줍니다.

- CLOB : String, char[]. java.sql.CLOB
- BLOB : byte[], java.sql.BLOB

LOB란 Large Object.
프로그램을 작성하다 보면 또는 데이터베이스를 정리하다 보면 테이블 칼럼 안에서 대용량의 데이터를 저장할 필요성이 있을 경우가 있다. 이럴 경우 LOB를 사용할 수 있다.

**LOB를 사용하는 이유?**

데이터의 안전한 보호와 데이터 저장의 일관성 때문<br>
LOB 타입은 4GB까지 지원(물론 이만큼 쓰일 일은 X)<br>
LONG 타입의 데이터들은 순차적으로만 접근이 가능하지만 LOB 타입은 랜덤(random)하게 접근할 수 있다.<br>
등등의 이유가 있다.

**But, 그러면 정말 파일 데이터를 DB에 저장해야 할까?**

여기에 대해선 참고 링크로 대체. 요약하자면

이미지를 공개할 수 있는 경우 - 예를 들어 S3와 같이 DB와 다른(아마도 더 저렴한) 스토리지에 이미지를 저장하는 것이 좋습니다.

반대로 이미지가 개인용이어야 하는 경우 DB에 이미지를 저장하는 것이 이점이 있기에 생각해 볼 수 있는 방안이라고 합니다.

https://stackoverflow.com/questions/5285857/when-is-using-mysql-blob-recommended</br>
https://stackoverflow.com/questions/4654004/mysql-binary-storage-using-blob-vs-os-file-system-large-files-large-quantities</br>
https://stackoverflow.com/questions/1717264/should-i-use-mysql-blob-field-type

---
lob 출처</br>
https://www.joinc.co.kr/w/Site/Database/Book/ProcPrograming/11.Large_Objects</br>
[https://sites.google.com/site/smcgbu/home/공부-이야기/향상된-객체-lob](https://sites.google.com/site/smcgbu/home/%EA%B3%B5%EB%B6%80-%EC%9D%B4%EC%95%BC%EA%B8%B0/%ED%96%A5%EC%83%81%EB%90%9C-%EA%B0%9D%EC%B2%B4-lob)<br>
http://www.gurubee.net/lecture/2768