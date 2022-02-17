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

### 2.5 페이징/정렬 처리하기

JPA가 실제 데이터베이스에서 사용하는 SQL의 처리를 자동으로 하기 때문에 개발자들은 SQL이 아닌 API의 객체와 메서드를 사용하는 형태로 페이징 처리를 할 수 있음

Spring Data JPA에서 페이징 처리와 정렬은 findAll()을 사용

findAll()

- JpaRepository 인터페이스의 상위인 PagingAndSortRepository의 메서드
- 파라미터로 전달되는 Pageable이라는 타입의 객체에 의해서 실행되는 쿼리를 결정
- 리턴 타입을 Page<T> 타입으로 지정하는 경우에는 반드시 파라미터를 Pageable 타입을 이용해야함

**2.5.1 Pageable 인터페이스**

Pageable 인터페이스

- 페이지 처리에 필요한 정보를 전달하는 용도의 인터페이스
- 인터페이스이기 때문에 실제 객체를 생성할 때는 구현체인 org.springframework.data.domain.PageRequest라는 클래스를 사용

PageRequest는 선언할 때 new를 이용할 수 없고 of()를 이용함

- of(int page, int size): 0부터 시작하는 페이지 번호와 개수(size), 정렬이 지정되지 않음
- of(int page, int size, Sort.Direction direction, String...props): 0부터 시작하는 페이지 번호와 개수, 정렬의 방향과 정렬 기준 필드들
- of(int page, int size, Sort sort): 페이지 번호와 개수, 정렬 관련 정보

**2.5.2 페이징 처리**

Spring Data JPA를 이용할 때 페이지 처리는 반드시 ‘0’부터 시작함

```java
@Test
    public void testPageDefault(){
        Pageable pageable= PageRequest.of(0,10);

        Page<Memo> result=memoRepository.findAll(pageable);

        System.out.println(result);
    }
```

Page 타입

- 단순히 해당 목록만으로 가져오는데 그치지 않음
- 실제 페이지 처리에 필요한 전체 데이터의 개수를 가져오는 쿼리 역시 같이 처리
- 만일 데이터가 충분하지 않다면 데이터의 개수를 가져오는 쿼리를 실행하지 않음

**2.5.3 정렬 조건 추가하기**

Sort는 한 개 혹은 여러 개의 필드 값을 이용해서 순차적 정렬(asc)이나 역순으로 정렬(desc)을 지정할 수 있음

```java
@Test
    public void testSort(){
        Sort sort1=Sort.by("mno").descending();
        Pageable pageable=PageRequest.of(0,10,sort1);

        Page<Memo> result=memoRepository.findAll(pageable);

        result.get().forEach(memo -> {
            System.out.println(memo);
        });
    }
```

### 2.6 쿼리 메서드(Query Methods) 기능과 @Query

Spring Data JPA에서 다양한 검색 조건을 처리하기 위해서 사용하는 방법

- 쿼리 메서드: 메서드 이름 자체가 쿼리의 구문으로 처리되는 기능
- @Query: SQL과 유사하게 엔티티 클래스의 정보를 이용해서 쿼리를 작성하는 기능
- Querydsl 등의 동적 쿼리 처리 기능

**2.6.1 쿼리 메서드(Query Methods)**

쿼리 메서드

- 메서드의 이름 자체가 질의문(query 문)이 되는 기능
- 주로 findBy나 getBy..로 시작함
- 이후에 필요한 필요한 필드 조건이나 And, Or와 같은 키워드를 이용해서 메서드의 이름 자체로 질의 조건을 만들어 냄
- 사용하는 키워드에 따라서 파라미터 개수가 결정됨
- 리턴 타입의 내용
    - select를 하는 작업이라면 List 타입이나 배열을 이용할 수 있음
    - 파라미터에 Pageable 타입을 넣는 경우에는 무조건 Page<E> 타입
    

```java
public interface MemoRepository extends JpaRepository<Memo, Long> {
    List<Memo> findByMnoBetweenOrderByMnoDesc(Long from, Long to);
}
```

정렬에 관한 부분은 Pageable로 처리해서 좀 더 간단한 형태의 메서드 선언이 가능함

Pageable 파라미터는 모든 쿼리 메서드에 적용할 수 있으므로 일반적인 경우에는 쿼리 메서드에 정렬 조건을 생략하고 만드는 경우가 많음

```java
@Test
public void testQueryMethodWithPageable(){
    Pageable pageable=PageRequest.of(0,10,Sort.by("mno").descending());
    Page<Memo> result=memoRepository.findByMnoBetween(10L,50L,pageable);
    result.get().forEach(memo->System.out.println(memo));
}
```

```java
@Commit
@Transactional
@Test
public void testDeleteQueryMethods(){
    memoRepository.deleteMemoByMnoLessThan(10L);
}

```

@Commit

- 최종 결과를 커밋하기 위해 사용

**2.6.2 @Query 어노테이션**

@Query 

- value는 흔히 객체지향 쿼리라고 불리는 JPQL(Java Persistence Query Language)로 작성
- 실제 SQL에서 사용하는 함수인 avg(), count() 등도 JPQL에서 사용가능
- 메서드의 이름과 상관없이 메서드에 추가한 어노테이션을 통해 원하는 처리가 가능
- 필요한 데이터만 선별적으로 추출하는 기능이 가능
- 데이터베이스에 맞는 순수한 SQL을 사용하는 기능
- insert, update, delete와 같은 select가 아닌 DML 등을 처리하는 기능(@Modifying과 함께 사용)

```java
@Query("select m from Memo m order by m.mno desc")
List<Memo> getListDesc();
```

@Query의 파라미터 바인딩

- 직접 SQL과 유사한 형태로 작성되기 떄문에 where 구문과 그에 맞는 파라미터들을 처리할 때가 많음
- ‘?1, ?2’와 1부터 시작하는 파라미터의 순서를 이용하는 방식
- ‘:xxx’와 같이 ‘:파라미터 이름’을 활용하는 방식
- ‘:#{ }’과 같이 자바 빈 스타일을 이용하는 방식

```java
@Transactional
@Modifying
@Query("update Memo m set m.memoTxt = :memoText where m.mno = :mno")
int updateMemoText(@Param("mno") Long mno, @Param("memoText") String memoText);
```

@Query와 페이징 처리

- Pageable 타입의 파라미터를 적용하면 페이징 처리와 정렬에 대한 부분을 작성하지 않을 수 있음

Obect[] 리턴

- 현재 필요한 데이터만을 Object[]의 형태로 선별적으로 추출할 수 있음
- 적당한 엔티티 타입이 존재하지 않는 경우 Object[] 타입을 리턴 타입으로 지정 가능

Native SQL 처리

- nativeQuery 속성값을 true로 지정하고 데이터베이스 고유의 SQL 구문을 그대로 활용 가능
