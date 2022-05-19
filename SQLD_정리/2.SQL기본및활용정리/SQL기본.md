- **SQL 문장들의 종류**

| 명령어의 종류 | 명령어 | 설명 |
| --- | --- | --- |
| DML | SELECT | 데이터베이스에 들어있는 데이터를 조회하거나 검색하는 명령어 |
|  | INSERT
UPDATE
DELETE | 데이터베이스의 테이블에 들어 있는 데이터에 변형을 가하는 종류의 명령어 |
| DDL | CREATE
ALTER
DROP
RENAME | 테이블과 같은 구조를 정의하는데 사용하는 명령어 |
| DCL | GRANT
REVOKE | 데이터베이스에 접근하고 객체들을 사용하도록 권한을 주고 회수하는 명령어 |
| TCL | COMMIT
ROLLBACK | 논리적인 작업의 단위를 묶어서 DML에 의해 조작된 결과를 작업단위(트랜잭션) 별로 제어하는 명령어 |

**As-Is: 비절차적 데이터조작어(DML)은 사용자가 무슨(what) 데이터를 원하는 지만 명세**

**To-Be: 비절차적 데이터조작어(DML)은 사용자가 무슨(what) 데이터를 원하는지만 명세**

           **절차적 데이터 조작어(PL/SQL, T-SQL 등)는 어떻게(How) 데이터를 접근해야 하는지 명세**

- **연산자**

| 구분 | 연산자 | 연산자의 의미 |
| --- | --- | --- |
| 비교 연산자 | = | 같다 |
|  | > | 보다 같다 |
|  | >= | 보다 크거나 같다 |
|  | < | 보다 작다 |
|  | <= | 보다 작거나 같다 |
| SQL 연산자 | BETWEEN a AND b | a와 b 사이에 있으면 된다(a, b 포함) |
|  | IN(list) | list에 있는 값 중에서 어느 하나라도 일치하면 된다 |
|  | LIKE ‘비교문자열’ | 비교문자열과 형태가 일치하면 된다 |
|  | IS NULL | NULL 값일 경우 |
| 논리 연산자 | AND | 앞의 조건과 뒤의 조건을 동시에 만족 |
|  | OR | 앞뒤 조건 중 하나만 참 |
|  | NOT | 뒤에 오는 조건에 반대되는 결과를 반환 |
| 부정 비교 연산자 | != | 같지 않다 |
|  | ^= | 같지 않다 |
|  | <> | 같지 않다(ISO 표준, 모든 운영체제에서 사용 가능) |
|  | NOT 칼럼명 = | ~와 같지 않다 |
|  | NOT 칼럼명 > | ~보다 크지 않다 |
| 부정 SQL 연산자 | NOT BETWEEN a AND b | a와 b의 값 사이에 있지 않다 |
|  | NOT IN (list) | list 값과 일치하지 않는다 |
|  | IS NOT NULL | NULL 값을 갖지 않는다 |

- **연산자 우선순위**
    1. **괄호로 묶은 연산**
    2. **부정 연산자(NOT)**
    3. **비교 연산자와 SQL 비교 연산자**
    4. **논리 연산자 중 AND, OR 순으로 처리**

- **PK 제약 조건**
    - Oracle
        
        ```sql
        CREATE TABLE EXAMPLE(
        	col1 VARCHAR2(10) NOT NULL,
        	col2 DATE NOT NULL
        	CONSTRAINT EXAMPLE_PK PRIMARY KEY(col1)
        );
        ```
        

- **테이블 칼럼에 대한 정의 변경**
    - SQL Server
    
    ```sql
    ALTER TABLE EXAMPLE ALTER COLUMN COL1 VARCHAR(30) NOT NULL;
    ```
    
    - Oracle
    
    ```sql
    ALTER TABLE EXAMPLE MODIFY COLUMN COL1 VARCHAR(30) NOT NULL;
    ```
    
- **불필요한 컬럼 삭제**
    
    ```sql
    ALTER TABLE EMP
    DROP COLUMN COMM;
    ```
    

- **테이블 이름 변경**
    
    ```sql
    RENAME STADIUM TO STADIUM_JSC;
    ```
    
- **테이블에 데이터를 입력하는 두 가지 유형**
    
    ```sql
    INSERT INTO TABLE1 (col1, col3, col4) VALUES (1, 3, 4);
    
    INSERT INTO TABLE1 VALUES (col1, col2, col3, col4, col5);
    ```
    

- **입력된 데이터 수정**
    
    ```sql
    UPDATE TABLE1 SET COL3 = 3;
    ```
    
- **NULL**
    - 공백 문자나 숫자 0과는 전혀 다른 값
    - 아직 정의되지 않은 미지의 값
    - 현재 데이터를 입력하지 못하는 경우
    - ORACLE에서는 ‘’ 를 NULL로 처리
    - 결과값을 NULL이 아닌 다른 값을 얻고자 할 때 NVL/ISNULL 함수 사용
    
- **NULL의 연산**
    - NULL 값과의 연산은 NULL을 리턴
    - NULL 값과의 비교연산은 거짓을 리턴
    - 특정 값보다 크다, 적다라고 표현할 수 없음
    
- **NULL 관련 함수**
    - Oracle
        - **NVL(표현식1, 표현식 2)**
            - 표현식1의 결과값이 NULL이면 표현식2의 값 출력
        - **ISNULL(표현식1, 표현식2)**
            - 표현식1의 결과값이 NULL이면 표현식2의 값 출력
    - SQL
        - **NULLIF(표현식1, 표현식2)**
            - 표현식1이 표현식2와 같으면 NULL을, 다르면 표현식1을 리턴
        - **COALESCE(표현식1, 표현식2…..)**
            - 임의의 개수 표현식에서 NULL이 아닌 최초의 표현식
            
- **집계함수의 종류**

| 집계 함수 | 사용 목적 |
| --- | --- |
| COUNT(*) | NULL 값을 포함한 행의 수를 출력 |
| COUNT(표현식) | 표현식의 값이 NULL 값인 것을 제외한 행의 수를 출력 |
| SUM([DISTINCT | ALL] 표현식) | 표현식의 NULL 값을 제외한 합계를 출력 |
| AVG([DISTINCT | ALL] 표현식) | 표현식의 NULL값을 제외한 평균을 출력 |
| MAX([DISTINCT | ALL] 표현식) | 표현식의 최대값을 출력 |
| MIN([DISTINCT | ALL] 표현식) | 표현식의 최소값을 출력 |
| STDDEV([DISTINCT | ALL] 표현식) | 표현식의 표준 편차를 출력 |
| VARIAN([DISTINCT | ALL] 표현식) | 표현식의 분산을 출력 |
| 기타 통계 함수 | 벤더별로 다양한 통계식을 제공 |

- **DELETE(Modift) Action**
    - **Cascade**
        - Master 삭제 시 Child 같이 삭제
    - **Set Null**
        - Master 삭제 시 Child 해당 필드 NULL
    - **Set Default**
        - Master 삭제 시 Child 해당 필드 Default 값으로 설정
    - **Restric**
        - Child 테이블에 PK 값이 없는 경우만 Master 삭제 허용
    - **No Action**
        - 참조무결성을 위반하는 삭제/수정 액션을 취하지 않음
        
- **Insert Action**
    - **Automatic**
        - Master 테이블에 PK가 없는 경우 Master PK를 생성 후 Child 입력
    - **Set Null**
        - Master 테이블에 PK가 없는 경우 Child 외부키를 Null 값으로 처리
    - **Set Default**
        - Master 테이블에 PK가 없는 경우 Child 외부키를 지정된 기본값으로 입력
    - **Dependent**
        - Master 테이블에 PK가 존재할 때만 Child 입력 허용
    - **No Action**
        - 참조무결성을 위반하는 입력 액션을 취하지 않음
        

- **제약 조건의 종류**
    - **PRIMARY KEY(기본키)**
        - 테이블에 저장된 행 데이터를 고유하게 식별하기 위한 기본키
        - 하나의 테이블에 하나의 기본키 제약만 정의 가능
        - 기본키 제약을 정의하면 DBMS는 자동으로 UNIQUE 인덱스를 생성하며 기본키를 구성하는 칼럼에는 NULL 입력X
        - 기본키 제약 = 고유키 제약 & NOT NULL 제약
    - **UNIQUE KEY(고유키)**
        - 테이블에 저장된 행 데이터를 고유하게 식별하기 위한 고유키
        - 단 NULL은 고유키 제약의 대상이 아니므로, NULL 값을 가진 행이 여러개가 있더라도 고유키 제약 위반이 되지 않음
    - **NOT NULL**
        - NULL 값의 입력 금지
    - **CHECK**
        - 입력할 수 있는 값의 범위 등을 제한
        - CHECK 제약으로는 TRUE or FALSE로 평가할 수 있는 논리식 지정
    - **FOREIGN KEY(외래키)**
        - 테이블간의 관계를 정의하기 위해 기본키를 다른 테이블의 외래키로 복사하는 경우 외래키 생성
        - 외래키 지정시 참조 무결성 제약 옵션 선택 O
        - 외래키는 NULL 값을 가질 수 있음
        - 한 테이블에 여러개 존재 O
        
    
- **테이블 삭제**

| DROP | TRUNCATE | DELETE |
| --- | --- | --- |
| DDL | DDL
(일부 DML 성격) | DML |
| Rollback 불가능 | Rollback 불가능 | Commit 이전 Rollback 가능 |
| Auto Commit | Auto Commit | 사용자 Commit |
| 테이블이 사용했던 Storage를 모두 Relase | 테이블이 사용했던 Storage 중 최초 테이블 생성시 할당된 Storage만 남기고 Release | 데이터를 모두 Delete해도 사용했던 Storage는 Release되지 않음 |
| 테이블의 정의 자체를 완전히 삭제 | 테이블을 최초 생서된 초기상태로 만듦 | 데이터만 삭제 |

- **트랜잭션의 특성**
    - **원자성**
        - 트랜잭션에서 정의된 연산들은 모두 성공적으로 실행되던지 아니면 전혀 실행되지 않은 상태로 남아 있어야 함
    - **일관성**
        - 트랜잭션이 실행되기 전의 데이터베이스 내용이 잘못 되어 있지 않다면 트랜잭션이 실행된 이후에도 데이터베이스의 내용에 잘못이 있으면 안됨
    - **고립성**
        - 트랜잭션이 실행되는 도중에 다른 트랜잭션의 영향을 받아 잘못된 결과를 만들어서는 안됨
    - **지속성**
        - 트랜잭션이 성공적으로 수행되면 그 트랜잭션이 갱신한 데이터베이스의 내용은 영구적으로 저장
        
- **트랜잭션에 대한 격리성이 낮은 경우 발생할 수 있는 문제점**
    - **Dirty Read**
        - 다른 트랜잭션에 의해 수정되었지만 아직 커밋되지 않은 데이터를 읽는 것
    - **Non-Repeatable Read**
        - 한 트랜잭션 내에서 같은 쿼리를 두 번 수행했는데, 그 사이에 다른 트랜잭션이 값을 수정 또는 삭제하는 바람에 두 쿼리 결과가 다르게 나타나는 현상
    - **Phantom Read**
        - 한 트랜잭션 내에서 같은 쿼리를 두 번 수행했는데, 첫 번째 쿼리에서 없던 유령 레코드가 두 번째 쿼리에서 나타나는 현상
        
- **Rollback**
    - 데이터 변경 사항이 취소되어 데이터의 이전 상태로 복구
    - 관련된 행에 대한 잠금이 풀리고 다른 사용자들이 데이터 변경을 할 수 있음
    - BEGIN TRANSACTION으로 트랜잭션을 시작하고 COMMIT 또는 ROLLBACK으로 트랜잭션을 종료
    - ROLLBACK 구문을 만나면 최조의 BEGIN 시점까지 모두 ROLLBACK 수행

- **저장점(SAVEPOINT)**
    - 저장점을 정의하면 ROLLBACK 할 때 트랜잭션에 포함된 전체 작업을 롤백하는 것이 아니라 현 시점에서 저장점까지 트랜잭션의 일부만 롤백 가능

- **함수**
    - 내장함수
        - 단일행 함수
            
            
            | 종류 | 내용 | 함수의 예 |
            | --- | --- | --- |
            | 문자형 함수 | 문자를 입력하면 문자나 숫자 값을 반환 | LOWER, UPPER, SUBSTR/SUBSTRING, LENGTH/LEN, LTRIM, RTRIM, TRIM, ASCII |
            | 숫자형 함수 | 숫자를 입력하면 숫자 값을 반환 | ABS, MOD, ROUND, TRUNC, SIGN, CHR/CHAR, CEIL/CEILING, FLOOR, EXP, LOG, LN, POWER, SIN, COS, TAN |
            | 날짜형 함수 | DATE 타입의 값을 연산 | SYSDTE/GETDATE, EXTRACT/DATEPART, TO_NUMBER(TO_CHAR(d, ‘YYYY’|’MM’|’DD’))/ YEAR|MONTH|DAY |
            | 변환형 함수 | 문자, 숫자, 날짜형 값의 데이터 타입을 변환 | TO_NUMBER, TO_CHAR, TO_DATE/CAST, CONVERT |
            | NULL 관련 함수 | NULL을 처리하기 위한 함수 | NVL/ISNULL, NULLIF, COALESCE |
        - 다중행 함수
            - 집계함수
            - 그룹함수
            - 윈도우 함수
    - 사용자 정의 함수
    
- **GROUP BY 절과 HAVING 절의 특성**
    - GROUP BY 절을 통해 소그룹별 기준을 정한 후, SELECT 절에 집계 함수를 사용
    - 집계 함수의 통계 정보는 NULL 값을 가진 행을 제외하고 수행
    - GROUP BY 절에서는 SELECT 절과는 달리 ALIAS 명을 사용할 수 X
    - 집계 함수는 WHERE 절에는 올 수 없음
    - HAVING절은 GROUP BY절의 기준 항목이나 소그룹의 집계 함수를 이용한 조건을 표시할 수 있음
    - GROUP BY 절에 의한 소그룹별로 만들어진 집계 데이터 중, HAVING 절에서 제한 조건을 두어 조건을 만족하는 내용만 출력
    - HAVING 절은 일반적으로 GROUP BY 절 뒤에 위치
    
- **ORDER BY 절의 특징**
    - 기본적인 정렬 순서는 오름차순
    - ORACLE에서는 NULL 값을 가장 큰 값으로 간주
    - SQL SERVER에서는 NULL 값을 가장 작은 값으로 간주
    
- **문장 실행 순서**
    1. FROM
    2. WHERE
    3. GROUP BY
    4. HAVING
    5. SELECT
    6. ORDER  BY

- **JOIN**
    - 두 개 이상의 테이블들을 연결 또는 결합하여 데이터를 출력하는 것
    - 일반적인 경우 행들은 PRIMARY KEY나 FOREIGN KEY 값의 연관에 의해 JOIN이 성립
    - 하지만, 어떠한 경우에는 이러한 PK, FK의 관계가 없어도 논리적인 값등릐 연관만으로 JOIN 성립
    

---

**reference**

[[SQLD] 과목 2. SQL 기본 및 활용 정리](https://data-make.tistory.com/478)
