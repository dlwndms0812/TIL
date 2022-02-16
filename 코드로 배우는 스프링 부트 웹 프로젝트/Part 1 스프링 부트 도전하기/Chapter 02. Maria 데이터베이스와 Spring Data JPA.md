### 2.3 Spring Data JPA의 소개

JPA(Java Persistence API)

- Java 언어를 통해서 데이터베이스와 같은 영속 계층을 처리하고자 하는 스펙

**2.3.1 ORM과 JPA**

ORM(Object Relational Mapping)

- 객체 지향 패러다임을 관계형 데이터베이스에 보존하는 기술
- 객체 지향 패러다임을 관계형 패러다임으로 매핑
- ‘객체 지향’의 구조와 ‘관계형 데이터베이스’의 유사점에서 시작
- 클래스와 테이블 유사: 새로운 테이블에 칼럼을 정의하고 칼럼에 맞는 데이터 타입을 지정해서 데이터를 보관하는 틀을 만듦
- 인스턴스와 Row(튜플) 유사: 클래스에서 인스턴스라는 공간에 데이터 보관 vs 테이블에서 Row에 데이터를 저장
- 관계와 참조 유사: 관계형 데이터베이스는 테이블 사이의 관게를 통해 구조적인 데이터 표현 vs 객체지향에서 참조를 통해서 어떤 객체가 다른 객체들과 어떤 관계를 맺고 있는지 표현
- ⇒ 객체지향을 자동으로 관계형 데이터베이스에 맞게 처리해 주는 기법

JPA(Java Persistence API)

- ORM을 Java 언어에 맞게 사용하는 스펙
- ex) Hibernate, TopLink, Eclipselink....

**2.3.2 Spring Data JPA와 JPA**

스프링 부트는 Hibernate를 이용

Hibernate

- 오픈소스로 ORM을 지원하는 프레임워크
- 단독으로 프로젝트에 적용이 가능한 독립적인 프레임워크

Spring Data JPA ↔ Hibernate ↔ JDBC ↔ DB

### 2.4 엔티티 클래스와 JpaRepository

Spring Data JPA가 개발에 필요한 것

- JPA를 통해서 관리하게 되는 객체(엔티티 객체)를 위한 엔티티 클래스
- 엔티티 객체들을 처리하는 기능을 가진 Repository

Repository

- Spring Data JPA에서 제공하는 인터페이스로 설계하는데 스프링 내부에서 자동으로 객체를 생성하고 실행하는 구조

```java
package com.example.ex2.entity;

import lombok.*;

import javax.persistence.*;

@Entity
@Table(name="tb1_memo")
@ToString
public class Memo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mno;
}
```

@Entity

- 반드시 추가해야함
- 해당 클래스가 엔티티를 위한 클래스이며, 해당 클래스의 인스턴스들이 JPA로 관리되는 엔티티 객체라는 것을 의미
- 옵션에 따라 자동으로 테이블 생성 가능
- 클래스의 멤버 변수에 따라서 자동으로 칼럼들도 생성

@Table

- @Entity 어노테이션과 같이 사용할 수 있는 어노테이션
- 데이터베이스상에서 엔티티 클래스를 어떠한 테이블로 생성할 것인지에 대한 정보를 담기 위한 어노테이션
- 단순히 테이블의 이름 뿐만 아니라 인덱스 등 생성 가능

@Id

- @Entity가 붙은 클래스는 Primary Key에 해당하는 특정 필드를 @Id로 지정해야함
- 자동으로 생성되는 번호를 사용하려면 @GeneratedValue를 사용

@GeneratedValue

- PK를 자동으로 생성하고자 할 때 사용
- 키 생성 전략
    - AUTO(default): JPA 구현체가 생성 방식을 결정
    - IDENTITY: 사용하는 데이터베이스가 키 생성을 결정
    - SEQUENCE: 데이터베이스의 sequence를 이용해서 키를 생성, @SequenceGenerator와 같이 사용
    - TABLE: 키 생성 전용 테이블을 생성해서 키 생성, @TableGenerator와 함께 사용
    

@Column

- 추가적인 필드가 필요한 경우에 사용
- 다양한 속성을 지정 가능
- @Column으로 기본 값을 지정하기 위해서 columnDefinition을 사용

@Transient

- @Column과 반대로 데이터베이스 테이블에 칼럼으로 생성되지 않는 필드의 경우에 사용

**2.4.2 Spring Data JPA를 위한 스프링 부트 설정**

application.properties에 추가

```java
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3306/bootex
spring.datasource.username=bootuser
spring.datasource.password=bootuser

spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true
```

spring.jpa.hibernate.ddl-auto

- 프로젝트 실행 시에 자동으로 DDL(create, alter, drop 등)을 생성할 것인지를 결정하는 걸정
- create, update, create-drop, validate

spring.jpa.hibernate.format_sql

- 실제 JPA의 구현체인 Hibernate가 동작하면서 발생하는 SQL을 포매팅해서 출력
- 실행되는 SQL의 가독성을 높여줌

spring.jpa.show-sql

- JPA 처리 시에 발생하는 SQL을 보여줄 것인지를 결정

**2.4.3 JpaRepository 인터페이스**

repository ← CrudRepository ← PagingAndSortRepository ← JpaRepository

Spring Data JPA는 인터페이스 선언만으로도 자동으로 스프링의 Bean으로 등록됨

=스프링이 내부적으로 인터페이스 타입에 맞는 객체를 생성으로 빈으로 등록

- insert 작업: save(엔티티 객체)
- select 작업: findById(키 타입), getOne(키 타입)
- update 작업: save(엔티티 객체)
- delete 작업: deleteById(키 타입), delete(엔티티 객체)

save()는 JPA의 구현체가 메모리상에서 객체를 비교하고 없다면 insert, 있다면 update를 동작

```java
@SpringBootTest
public class MemoRepositoryTests {
    @Autowired
    MemoRepository memoRepository;

    //의존성 주입에 문제 없는지 확인
    @Test
    public void testClass(){
        System.out.println(memoRepository.getClass().getName());
    }

    //등록 작업 테스트
    @Test
    public void testInsertDummies() {
        IntStream.rangeClosed(1, 100).forEach(i -> {
            Memo memo = Memo.builder().memoTest("Sample..." + i).build();
            memoRepository.save(memo);
        });
    }

    //조회 작업 테스트
    @Test
    public void testSelect(){
        Long mno=100L;

        Optional<Memo> result=memoRepository.findById(mno);

        System.out.println("========================");

        if(result.isPresent()){
            Memo memo=result.get();
            System.out.println(memo);
        }
    }

    @Transactional
    @Test
    public void testSelect2(){
        Long mno=100L;
        Memo memo=memoRepository.getOne(mno);
        System.out.println(memo);
    }

    //수정 작업 테스트
    @Test
    public void testUpdate(){
        Memo memo=Memo.builder().mno(100L).memoTest("Update Text").build();

        System.out.println(memoRepository.save(memo));
    }

    //삭제 작업 테스트
    @Test
    public void testDelete(){
        Long mno=100L;
        memoRepository.deleteById(mno);
    }
}
```

@Transaction

- 트랜잭션 처리를 위해 사용하는 어노테이션
