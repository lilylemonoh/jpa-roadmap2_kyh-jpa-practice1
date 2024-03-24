# Introduce
실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발 (김영한)의 예제코드 저장소


**[Inflearn - 강의 링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1/dashboard)**

학습 기간: 2024/3/10~2024/

# Curriculum
### 섹션 0. 강좌 소개
* 강좌 소개

### 섹션 1. 프로젝트 환경설정
* 프로젝트 생성
* 라이브러리 살펴보기
* View 환경 설정
* H2 데이터베이스 설치
* JPA와 DB 설정, 동작확인

### 섹션 2. 도메인 분석 설계
* 요구사항 분석
* 도메인 모델과 테이블 설계
* 엔티티 클래스 개발1
* 엔티티 클래스 개발2
* 엔티티 설계시 주의점

### 섹션 3. 애플리케이션 구현 준비

* 구현 요구사항
* 애플리케이션 아키텍처

### 섹션 4. 회원 도메인 개발
* 회원 리포지토리 개발
* 회원 서비스 개발
* 회원 기능 테스트

### 섹션 5. 상품 도메인 개발
* 상품 엔티티 개발(비즈니스 로직 추가)
* 상품 리포지토리 개발
* 상품 서비스 개발

### 섹션 6. 주문 도메인 개발
* 주문, 주문상품 엔티티 개발
* 주문 리포지토리 개발
* 주문 서비스 개발
* 주문 기능 테스트
* 주문 검색 기능 개발

### 섹션 7. 웹 계층 개발
* 홈 화면과 레이아웃
* 회원 등록
* 회원 목록 조회
* 상품 등록
* 상품 목록
* 상품 수정
* 변경 감지와 병합(merge)
* 상품 주문
* 주문 목록 검색, 취소
* 다음으로

# 참고사항
강의와 다르게 진행한 점:
* 인메모리 모드를 사용하여 H2 데이터베이스를 사용함 (h2 미설치)
* 따라서 application.yml에 다음과 같이 설정함
```
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    username: sa
  h2:
    console:
      enabled: true
```

---
# 필기
## 도메인 모델과 테이블 설계
* 회원, 주문, 상품의 관계
    * 회원은 여러 상품을 주문할 수 있다.
    * 그리고 한 번 주문할 때 여러 상품을 선택할 수 있으므로 주문과 상품은 다대다 관계다.
    * 그러나 이런 다대다 관계는 관계형 데이터베이스는 물론이고 엔티티에서도 거의 사용하지 않는다.
    * 따라서 주문상품이라는 엔티티를 추가해서 다대다 관계를 일대다, 다대일 관계로 풀어냈다.
## 참고 : 값 타입은 변경 불가능하게 설계해야 한다. (ex.Address)
* `@Setter`를 제거하고, 생성자에서 값을 모두 초기화해서 변경 불가능한 클래스를 만들자.
* JPA 스펙상 엔티티나 임베디드 타입(@Embeddable)은 자바 기본 생성자(default constructor)를 `public` 또는 `protected`로 설정해야 한다.
* `public`보다 `protected`가 안전하다.
* JPA가 이런 제약을 두는 이유는 JPA 구현 라이브러리가 객체를 생성할 때 리플렉션 같은 기술을 사용할 수 있도록 지원해야 하기 때문이다.
---
## 변경 감지와 병합(merge)
### 준영속 엔티티?
영속성 컨텍스트가 더는 관리하지 않는 엔티티를 말한다.  
(여기서는 `itemService.saveItem(book)`에서 수정을 시도하는 `Book` 객체다.
`Book` 객체는 이미 DB에 한번 저장되어서 식별자가 존재한다.
이렇게 임의로 만들어낸 엔티티도 기존 식별자를 가지고 있으면 준영속 엔티티로 볼 수 있다.)

### 준영속 엔티티를 수정하는 2가지 방법
* 변경 감지 기능 사용
* 병합(merge) 사용

### 변경 감지 기능 사용
```java
@Transactional
void update(Item itemParam) { // itemParam: 파라미터로 넘어온 준영속 상태의 엔티티
    Item findItem = em.find(Item.class, itemParam.getId()); // 같은 엔티티를 조회한다.
    findItem.setPrice(itemParam.getPrice()); // 데이터를 수정한다. 
    }
```

### 병합 사용
병합은 준영속 상태의 엔티티를 영속 상태로 변경할 때 사용하는 기능이다.
```java
@Transactional
void update(Item itemParam) { // itemParam: 파라미터로 넘어온 준영속 상태의 엔티티
    Item mergeItem = em.merge(itemParam);
}
```

### 병합 동작 방식
1. merge()를 실행한다.
2. 파라미터로 넘어온 준영속 엔티티의 식별자 값으로 1차 캐시에서 엔티티를 조회한다.
3. 만약 1차 캐시에 엔티티가 없으면 데이터베이스에서 엔티티를 조회하고, 1차 캐시에 저장한다.
4. 조회한 영속 엔티티(mergeMember)에 member 엔티티의 값을 채워 넣는다. (member 엔티티의 모든 값을 mergeMember에 밀어 넣는다.)
5. 영속 상태인 mergeMember를 반환한다.

### 병합 시 동작 방식을 간단히 정리
1. 준영속 엔티티의 식별자 값으로 영속 엔티티를 조회한다.
2. 영속 엔티티의 값을 준영속 엔티티의 값으로 모두 교체한다.(병합한다.)
3. 트랜잭션 커밋 시점에 변경 감지 기능이 동작해서 데이터베이스에 UPDATE SQL이 실행


>* 주의 : 변경 감지 기능을 사용하면 원하는 속성만 선택해서 변경할 수 있지만, 병합을 사용하면 모든 속성이 변경된다.
>* 병합 시 값이 없으면 null로 업데이트할 위험성도 있다. (병합은 모든 필드를 교체한다.)

### Best Practice
**엔티티를 변경할 때는 항상 변경 감지를 사용하기**
* 트랜잭션이 있는 서비스 계층에 식별자(id)와 변경할 데이터를 명확하게 전달하기 (parameter or dto)
* 트랜잭션이 있는 서비스 계층에서 영속 상태의 엔티티를 조회하고, 엔티티의 데이터를 직접 변경하기
* 트랜잭션 커밋 시점에 변경 감지 실행