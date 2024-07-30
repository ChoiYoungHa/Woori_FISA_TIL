# [FISA 클라우드엔지니어링16일차]

### View 사용

emp의 comm 컬럼은 영업직원 이외는 존재 자체를 몰라야 하는 상황

- 개발자도 직원
- comm 컬럼을 제외한 table 정보를 개발자에게 제공
- 방식
- `원본 테이블에서 comm 제외한 복제본`
- `crud로 복제본에 반영시 원본 table 데이터 동기화`
- `뷰 사용이 답`
- 뷰는 원본 테이블을 복제 후에는 원본 테이블의 변경에 영향을 받지 않는다.
- 반대로 뷰를 업데이트하면 원본 테이블도 업데이트 된다.

<aside>
💡 오라클은 기본적으로 일반계정에겐 view 생성 권한을 부여하지 않음
해결점은 admin이 view 생성 및 관리권한을 부여해주어야 한다.

</aside>

```sql
**grant create view to scott;**
```

- 뷰는 원본 테이블을 비추는 거울이기에 원본테이블의 변경이 일어나면 뷰에 반영된다.
- 뷰가 조인과 같은 쿼리로 생성되었다면 뷰가 변경되어도 원본 테이블은 변경되지 않지만 조인없는 단순한 뷰 같은 경우에는 뷰테이블 변경 시 원본 테이블이 변경된다.

### AutoIncrement

- table에 pk 타입 고려시 무궁무진하게 증가 가능한 pk에 적용
- 은행 고객 가입 : pk
    - id or 계좌번호 or email or 연락처..
    - id 하나가 여러명의 계좌 관리

```sql

DROP TABLE IF EXISTS emp01;

-- empno를 auto_increment로 자동 증가치 적용시 pk로 미등록시 에러 발생
create table emp01(
	empno int auto_increment,
	ename varchar(10) not null,
	primary key (empno)
);

desc emp01;

-- insert 
insert into emp01 (ename) values ('연아');
select * from emp01;
select * from emp01 limit 2;

-- 2. 수정 : 자동 증가분 시작값을 100으로 설정해 보기 
alter table emp01 auto_increment = 100;

insert into emp01 (ename) values ('연아');
select * from emp01; 
```

- 게시물 글번호 주로 사용
    - 500 게시글 작성자가 추후 삭제
    - 500번 게시글의 id는 재정렬하지 않고 날리는게 낫다.
    

### Oracle Seqence

```sql
/*
* 1. 개요
	- 새 데이터 저장시 고유 번호가 자동 생성 및 적용하게 하는 기술
	
*/

DROP TABLE dept01;

CREATE TABLE dept01 AS SELECT * FROM DEPT WHERE 1=0;

SELECT * FROM dept01;

-- deptno 1씩 자동증가 시퀀스 사용
DROP SEQUENCE dept01_deptno_sq;
CREATE SEQUENCE dept01_deptno_sq;

-- insert 시에 반드시 sequence명으로 명시
INSERT INTO dept01 VALUES (dept01_deptno_sq.nextval, '교육부', '상암');

SELECT * FROM dept01;

-- 오라클은 시퀀스 객체 생성 이후 insert 시에 해당 객체의 메서드를 사용해서 1씩 증가시킨다.
-- 현 sequence값 확인 명령어
-- 오라클은 from 절 생략불가라서 더미테이블 명으로 검색해야한다.
SELECT dept01_deptno_sq.currval FROM dual;
```

### Index

```sql
/*
문법
	CREATE INDEX index_name
	ON table_name (column1, column2, ...);
*/
   
use fisa;

-- 1. index용 검색 속도 확일을 위한 table 생성 
drop table if exists emp01;

-- 존재하는 table로 부터 복제시에는 제약조건은 미적용
create table emp01 as select * from emp;
desc emp01;

-- empno로 검색시에 빠른 검색이 가능하게 색인 기능 적용
create index idx_emp01_empno on emp01(empno);
desc emp01;

-- drop index
alter table emp01 drop index idx_emp01_empno;

```

- db의 빠른 검색을 위한 색인 기능의 index 학습 primary key에는 기본적으로 자동 index로 설정됨
- DB 자체적으로 빠른 검색 기능 부여
- 어설프게 사용자 정의 index 설정시 오히려 검색 속도 다운 데이터 셋의 15% 이상의 데이터들이 잦은 변경이 이뤄질 경우 index 설정 비추
- 주의사항
    - index가 반영된 컬럼 데이터가 수시로 변경되는 데이터라면 index 적용은 오히려 부적합

### 파티션

1. 기능
- 논리적으로 하나의 table 이미잠 물리적으로 여려 개의 table로 분리해서 관리하는 기술
- 데이터와 index를 조각화해서 물리적 메모리를 효율적으로 사용
- 하나의 table이 너무 클 경우 index의 크기가 물리적인 메모리 보다 훨씬 크거나 데이터 특성상 주기적인 삭제 작업이 필요한 경우 필요
- 단일 insert와 빠른 검색
- 파티션 키로 해당 데이터 저장 및 관리
- range() 사용한 partition
- 파티션 키로 연속된 범위로 파티션을 정의하는 방식
- maxvalue라는 키워드를 사용
- 명시되지 않은 범위의 키 값이 담긴 레코드를 저장하는 파티션 정의가 가능

**연도별 파티션**

```sql
CREATE TABLE emp00 (
    empno                int  NOT NULL,
    ename                varchar(20),
    job                  varchar(20),
    mgr                  smallint,
    hiredate             date,
    sal                  numeric(7, 2),
    comm                 numeric(7, 2),
    deptno               int
) partition by range ( year(hiredate) )(  -- 파티션 정의, range() 가 파티션 키 설정
	partition p0 values less than (1980),
	partition p1 values less than (1983),
	partition p2 values less than (1986),
	partition p3 values less than maxvalue 
);
```

**월단위 파티션**

```sql
CREATE TABLE emp00 (
    empno                int  NOT NULL,
    ename                varchar(20),
    job                  varchar(20),
    mgr                  smallint,
    hiredate             date,
    sal                  numeric(7, 2),
    comm                 numeric(7, 2),
    deptno               int
) partition by range ( month(hiredate) )(  -- 파티션 정의, range() 가 파티션 키 설정
	partition m1 values less than (02),
	partition m2 values less than (03),
	partition m3 values less than (04),
	partition m4 values less than (05),
	partition m5 values less than (06),
	partition m6 values less than (07),
	partition m7 values less than (08),
	partition m8 values less than (09),
	partition m9 values less than (10),
	partition m10 values less than (11),
	partition m11 values less than (12),
	partition m12 values less than maxvalue 
);
```

### 파티션을 뷰에도 적용할 수 있을까?

- 원본 table 기반으로 생성되는 view 원본 table의 파티션 기능 적용받음
- 뷰를 생성할 때는 파티션을 직접적으로 지정할 수 없지만, 파티션된 테이블을 기반으로 뷰를 생성할 수 있음.

```sql
sql코드 복사
-- 파티션된 테이블을 기반으로 뷰 생성
CREATE VIEW recent_orders AS
SELECT * FROM orders
WHERE order_date >= '2022-01-01';

```

- 위 예시는 최근 주문을 보여주는 뷰를 생성하는 방법
- 이 뷰는 `orders` 테이블에서 2022년 이후의 주문만을 선택하여 보여줌.
- 파티션된 테이블을 기반으로 하기 때문에, MySQL은 내부적으로 파티션 최적화를 적용할 수 있다.

## 개발설계

**구현로직**

1. 모든 검색
    1. 모든 정보를 하나의 객체에 담아서 view에게 출력 위임
    2. ArrayList<Dept01>
    3. Dept01DTO - 하나의 부서 정보를 보유하게 되는 데이터 표현용 클래스
2. 부서번호로 특정 부서 검색
    1. 경우의 수
        1. 정상 실행
            1. 존재하는 부서번호로 검색
                1. 반환값 : Dept01DTO 객체
            2. 미존재하는 부서 번호로 검색
                1. null 처리 또는 optional 처리
        2. 비정상 실행
            1. 예외 발생 : 처리 가능한 경미한 에러, try catch
            2. 에러 : 심각한 에러
3. 새로운 부서 등록
    1. 경우의 수
        1. pk 중복 고려
            1. 중복 되었다 - 상황 client에게 언급
            2. 중복 아니다 - 정상 저장
        2. 예외발생

1. 클래스 구조
    1. dept01 DTO, DAO

1. 개발환경
    
    win → vb → ubuntu → docker container → ubuntu 14 → oracle
    
    oracle jdbc 통신 포트 1521, ubuntu에서 접속시 포트포워딩 -p 1521:1521
    
    12.0.0.1:1521 ↔ 10.0.2.15:1521
    
2. 소스 개발 구조
    1. maven 기반의 driver 셋팅
    2. gradle 기반의 driver 셋팅

### PrepareStatement API 성능 최적화

PrepareStatemet API 특징
- db에 요청이 되면 문장 받고 -> 문법 검증 -> 실행
 	 매순간 요청마다 검증 작업 수행 - Statement
         최초 받고 매 요청시 재사용 - PrepareStatement(문법검증 생략)