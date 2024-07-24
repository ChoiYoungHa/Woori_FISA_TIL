# FISA 클라우드엔지니어링 13일차

### MY SQL DDL 언어연습

```sql
-- 존재 여부 확인 후에 존재할 경우에만 삭제하는 drop 문장
drop table if exists test;

-- 2. table 생성  
-- name(varchar(5), age(int) 칼럼 보유한 people table 생성
-- name은 최대 5개 글자 크기의 문자열 데이터 저장 
drop table if exists people;

create table people(
	name varchar(20),
	age int
);

desc people;

INSERT into people values ('이순신',20);
INSERT into people values ('kim',20);
SELECT * from people;

-- length 점유중인 메모리, char_length 글자길이
SELECT CHAR_LENGTH(name) from people; 
SELECT LENGTH (name) from people; 
```

### Oracle DDL 명령어

```sql
-- 구조와 데이터만 복제됨
CREATE TABLE emp01 AS SELECT empno, ename, deptno FROM emp;
select * from emp01;

-- 필요항목
-- 부모 table의 컬럼(dept, deptno), 자식 table의 컬럼(emp01, deptno)
-- 제약조건에 이름 부여
-- emp01의 empno 먼저 기본키 설정
ALTER TABLE emp01 ADD CONSTRAINT pk_emp01_empno
				  PRIMARY KEY (empno);
-- FK 추가	 
ALTER TABLE emp01 ADD FOREIGN KEY (deptno) 
				  REFERENCES dept (deptno);
				 
--? oracle에 제약조건을 확인 가능한 사전 table명 확인 후 검색해서 제약조건명 확인
SELECT * FROM USER_CONSTRAINTS;
SELECT * FROM USER_CONSTRAINTS WHERE table_name = 'EMP01';
```

```sql
-- 5. deptno=10 조건문 반영해서 empno, ename, deptno로 emp03 table 생성
CREATE TABLE emp03 AS SELECT empno, ename, deptno FROM emp WHERE deptno=10;
select * from emp03;

-- 6. 데이터 insert없이 table 구조로만 새로운 emp04 table생성시 
-- 사용되는 조건식 : where=거짓
CREATE TABLE emp04 AS SELECT * FROM emp WHERE 1=0;
SELECT * FROM emp04;

-- 이미 존재하는 table의 구조를 변경하는 sql문장 
-- 존재하는 컬럼 삭제, 수정
/** 고려사항 - 경우의 수 먼저 도출
 * 1. 데이터가 있다.
 * 1-1 컬럼을 삭제
 * 1-2 컬럼 크기 조정
 * 	1-2-1 데이터들 크기보다 크게 수정
 *  1-2-2 존재하는 데이터들 크기보다 작게 수정
 * 1-3 타입변경
 * 	1-3-1. 데이터가 없는 상태에서 타입변경
 * 	1-3-1. 데이터가 있는 상태에서 타입변경
 */

-- 7. emp01 table에 job이라는 특정 칼럼 추가(job varchar2(10))
-- 이미 데이터를 보유한 table에 새로운 job칼럼 추가 가능 
-- add  : 컬럼 추가 연산자
drop table emp01;

create table emp01 as select empno, ename from emp;
select * from emp01;

-- 최대 10byte 문자열 저장 가능한 job 컬럼 생성 및 추가 
alter table emp01 add job varchar(10);

select * from emp01;

-- 8. 이미 존재하는 칼럼 사이즈 변경 시도해 보기
-- 데이터 미 존재 칼럼의 사이즈 수정(크게/작게 다 수정 가능)
-- modify : 컬럼 변경

drop TABLE emp01;

create table emp01 as select empno, ename, job from emp;
select * from emp01;

-- mysql char_length, oracle length
select LENGTH(job) from emp01;

select job from emp01;

-- job 크기를 10으로 변경
/*
 * oracle 과 mysql 둘 다 varchar가 맞음
 * oracle8 버전 이후 varchar2
 * 		table생성시 varchar 표현할 경우엔 varchar2로 변환
 * mysql - varchar
 */
alter table emp01 modify job varchar2(10);
select * from emp01;  -- 모든 데이터 정상 검색

-- 9. 이미 데이터가 존재할 경우 칼럼 사이즈가 큰 사이즈의 컬럼으로 변경 가능 
alter table emp01 modify job varchar2(3);

-- alter table emp01 modify job varchar(3);  실패 

-- 10. job 칼럼 삭제 
-- 데이터 존재시에도 자동 삭제 
-- drop 
-- add시 필요 정보(컬럼명 타입(사이즈)) / modify 필요 정보(컬럼명 타입(사이즈)) / drop 필요 정보(컬럼명)
/*
 * 데이터 크기보다 작게 수정 불가능
 * 데이터가 존재할 경우 해당 칼럼은 삭제
 * 권장 : table 생성 후 제약조건 추가는 빈번
 * 		 칼럼 삭제는 극히 드문일로 처리
 * 
 */

ALTER TABLE emp01 add job varchar(3);
ALTER TABLE emp01 modify job varchar(10);

alter table emp01 drop column job;
SELECT * FROM emp01;
desc emp01;
```

- alter table add, modify, drop 쿼리사용

### MySQL DML 연습

```sql
-- *** update ***
-- 1. 테이블의 모든 행 변경
drop table emp01;
create table emp01 as select * from emp;
select deptno from emp01;

-- deptno값을 50으로 변경
update emp01 set deptno=50 where deptno=10;
SELECT * from emp01

-- 10번 부서 번호를 50으로 변경
-- emp 테이블은 부모테이블 dept의 제약조건 때문에 변경이 불가함.
update emp set deptno=50 where deptno=10;

 
-- 2. ? emp01 table의 모든 사원의 급여를 10%(sal*1.1) 인상하기
-- ? emp table로 부터 empno, sal, hiredate, ename 순으로 table 생성
desc emp01;
drop table if exists emp01;

create table emp01 as select empno, sal, hiredate, ename from emp;
desc emp01;
select * from emp01;

-- 전사원 급여 10% 인상
select sal from emp01;
update emp01 SET sal=sal*1.1;

-- ? 3. emp01의 모든 사원의 입사일을 오늘로 바꿔주세요
select now();
select * from emp01;

update emp01 SET hiredate = now();
select * from emp01;
commit;

-- ? 4. 급여가 3000이상(where sal >= 3000)인 사원의 급여만 10%인상
update emp01 SET sal=sal*1.1 where sal >= 3000;
select sal from emp01;
commit;

-- 5. ?emp01 table 사원의 급여가 1000이상인 사원들의 급여만 500원씩 삭감 
-- insert/update/delete 문장에 한해서만 commit과 rollback 영향을 받음
update emp01 SET sal=sal-500 where sal >= 1000;
select sal from emp01;

-- 6. ? emp01 table에 DALLAS(dept의 loc)에 위치한 부서의 소속 사원들의 급여를 1000인상
-- 서브쿼리 사용
update emp01 SET sal=sal+1000 where deptno = (select deptno from dept where loc = 'DALLAS');

-- 7. ? emp01 table의 SMITH 사원의 부서 번호를 30으로, 직급은 MANAGER 수정
-- 두개 이상의 칼럼값 동시 수정
select deptno, job from emp01 where ename='SMITH';
update emp01 set deptno = 30, job = 'MANAGER' where ename = 'SMITH';

select deptno, job from emp01 where ename='SMITH';
commit;

-- *** delete ***
-- 8. 하나의 table의 모든 데이터 삭제
select * from emp01;

-- 9. 특정 row 삭제(where 조건식 기준)
-- deptno가 30번 부서의 모든 사원들 사제
delete from emp01 where deptno=30;

-- 10. emp01 table에서 comm 존재 자체가 없는(null) 사원 모두 삭제
delete from emp01 where comm IS NULL; 

-- 11. emp01 table에서 comm이 null이 아닌 사원 모두 삭제
DELETE from emp01 WHERE comm IS NOT NULL;

-- 12. emp01 table에서 부서명이 RESEARCH 부서(dept table의 dname)에 소속된 사원 삭제 
-- 서브쿼리 활용
DELETE from emp01 where deptno = (SELECT deptno from dept where dname ='RESEARCH');
```

### 오라클 제약조건

1. constraints
    - 제약 조건등의 정보 확인
    - user constraints
    - oracle의 경우 이런 사전용 tabled에는 대문자명으로 table들 관리
2. 제약 조건 종류
emp와 dept의 관계
- dept 의 deptno가 원조 / emp 의 deptno는 참조
- dept : 부모 table / emp : 자식 table(dept를 참조하는 구조)
- dept의 deptno는 중복 불허(not null) - 기본키(pk, primary key)
- emp의 deptno - 참조키(fk, foreign key, 외래키)
    
    2-1. PK[primary key] - 기본키, 중복불가, null불가, 데이터들 구분을 위한 핵심 데이터
    : not null + unique
    2-2. not null - 반드시 데이터 존재
    2-3. unique - 중복 불가, null 허용
    2-4. check - table 생성시 규정한 범위의 데이터만 저장 가능
    2-5. default - insert시에 특별한 데이터 미저장시에도 자동 저장되는 기본 값
    - 자바 관점에는 멤버 변수 선언 후 객체 생성 직후 멤버 변수 기본값으로 초기화
    
    - 2-6. FK[foreign key]
        - 외래키[참조키], 다른 table의 pk를 참조하는 데이터
        - table간의 주종 관계가 형성
        - pk 보유 table이 부모, 참조하는 table이 자식
3. 제약조건 적용 방식
4-1. table 생성시 적용
- create table의 마지막 부분에 설정
방법1 - 제약조건명 없이 설정
방법2 - constraints 제약조건명 명식
    
    ```
     - 참고 : mysql의 pk에 설정한 사용자 정의 제약조건명은 data 사전 table에서 검색 불가
     	oracle db는 명확하게 사용자 정의 제약조건명 검색
    ```
    
    4-2. alter 명령어로 제약조건 추가
    
    - 이미 존재하는 table의 제약조건 수정(추가, 삭제)명령어
    alter table 테이블명 modify 컬럼명 타입 제약조건;
    
    4-3. 제약조건 삭제(drop)
    - table삭제
    alter table 테이블명 alter 컬럼명 drop 제약조건;
    

### JDBC API 활용기술

1. db 연동 표준 API
2. jdk 내장된 API
3. 필수학문
    1. sql + java = db 연동 프로그램
    2. dbeaver java로 구현된 db 연동 프로그램

1. 개발 환경
    1. RDBMS
    2. db연동 driver
        1. oracle driver
        2. mysql driver

### JDBC Connection 단계

1. driver 로딩 (메모리 저장, 인스톨)
2. 특정 db에 접속 가능한 자바 객체 생성
    1. drivermanager api에게 요청
3. sql문장 작성해서 실행 가능 객체 생성
4. sql 문장 실행
    1. DDL/DML
    2. DDL(create/drop/alter)는 가급적 sql 문장으로 개발 권장
    3. DML
        1. DQL : select - table 구조의 결과 집합 (resultset) 반환
        2. insert/update/delete - int 값 반환
5. db 접속해제

### Java → Virsual Box Ubuntu → Docker → MySQL  JDBC 커넥션

1. 버추얼박스 네트워크 포트 포워딩
    1. 3306
2. MySQL mysqld.cnf 파일 
    1. - ./mysqld.cnf:/etc/mysql/conf.d/mysqld.cnf 마운트
3. STS에서 Maven으로 mysql jdbc 드라이버 다운로드
4. jdbc 코드 작성하고 테스트하기

```sql
package step01.basic;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class MysqlJdbcExample {
	
	public static void main(String[] args) {
        String jdbcUrl = "jdbc:mysql://127.0.0.1:3306/fisa";
        String username = "root";
        String password = "1234";

        try {
            // Load MySQL JDBC Driver
            Class.forName("com.mysql.cj.jdbc.Driver");
            
            // Establish a connection
            Connection connection = DriverManager.getConnection(jdbcUrl, username, password);
            
            // Create a statement
            Statement statement = connection.createStatement();
            
            // Execute a query
            ResultSet resultSet = statement.executeQuery("SELECT * FROM emp");
            
            // Process the result set
            while (resultSet.next()) {
                System.out.println("Column1: " + resultSet.getString("ename"));
                System.out.println("Column2: " + resultSet.getInt("empno"));
            }
            
            // Close the resources
            resultSet.close();
            statement.close();
            connection.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}

```