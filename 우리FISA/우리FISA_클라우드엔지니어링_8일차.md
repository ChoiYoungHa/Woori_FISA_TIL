# [FISA 클라우드엔지니어링 8일차]

### Ubuntu Mysql 설치

1. ubuntu mysql 설치
2. ubuntu 명령어로 mysql 활용
3. window에서 tool로 access 및 crud
    1. dbber로 ssh 접속
4. kibana
    1. 우리금융 데이터 기반으로 두개 시각화하기
    2. 4인1팀

### MYSQL 설치 후 테이블 생성

```java
username@servername:~$ sudo mysql -u root -p
show databases;

use fisa;

create table ce(
, PK) : int / name : varchar / age : int
    id int primary key,
    name varchar not null,
    age int
);

create table ce (
    id int primary key,
    name varchar(255) not null,
    age int
);
drop table ce;
--
insert into ce values(1, ' ', 10);
```

### 간단한 SQL문 연습

```java
use fisa;
--
show tables;
--
select * from emp;
select * from dept;
-- 4. emp의 사번(empno)만 검색
select empno from emp;
-- 5. emp의 사번(empno), 이름(ename)만 검색
select empno, ename from emp;
-- 6. emp의 사번(empno), 이름(ename)만 검색(별칭 적용)
select empno as 사번, ename as 이름 from emp;
-- 7. 부서 번호(deptno) 검색
select deptno from emp;
-- 8. 중복 제거된 부서 번호 검색
select distinct deptno from emp;
-- 9. 8 + 오름차순 정렬(order by)해서 검색
-- 오름 차순 : asc / 내림차순 : desc
select distinct(deptno) from emp order by deptno; 
select distinct(deptno) from emp order by deptno desc;

-- 10. ? 사번(empno)과 이름(ename) 검색 단 사번은 내림차순(order by
select empno as A#, ename as Ole from emp order by empno desc
-- 11. ? dept table의 deptno 값만 검색 단 오름차순(asc)으로 검색
desc dept;
select deptno from dept order by deptno;

-- 12. ? 입사일(hiredate) 검색,
-- 입사일이 오래된 직원부터 검색되게 해 주세요
-- 고려사항 : date 타입도 정렬(order by) 가능 여부 확인
select hiredate from emp order by hiredate;

-- 13. ?모든 사원의 이름과 월 급여(Sal)와 연봉 검색
-- DB 자체로 연산 확인
-- MySQL : from 절 생략 가능 / oracle : select 1+2 from dual;

select ename, sal, sal*12 from emp;
-- 14. ?모든 사원의 이름과 월 급여(Sal)와 연봉(sal*12) 검색
-- 단 comm 도 고려(+comm) = 연봉(Sal*12) + comm
-- nu11값과 연산시에는 모든 데이터가 null
-- 해결책 : null을 0값으로 대체
-- 모든 db는 지원하는 내장 함수
-- null -> 숫자값으로 대체하는 함수 : IFNULL (nul1보유명, 대체값)
-- 모든 사원의 이름과 월급여(sal)와 연봉(Sal*12)+comm 검색
select ename, sal, (sal*12+IFNULL(comm, 0)) from emp;
. *** 조건식 ***
-- 15. comm이 nul1인 사원들의 이름과 comm만 검색
-- where 절 : 조건식 의미
select ename, comm from emp where comm is null;

-- 16. comm이 null이 아닌 사원들의 이름과 comm만 검색
-- where 절 : 조건식 의미
-- 아니다 라는 부정 연산자 : not
select ename, comm from emp where comm is not null;
-- 모든 직원명, comm으로 내림 차순 정렬
-- null 값 보유 컬럼을 오름차순 정렬 시 null부터 정렬
select ename, comm from emp order by comm;

select ename, comm from emp order by comm desc;
-- 17. ? 사원명이 스미스인 사원의 이름과 사번만 검색
--=: db에선 동등비교 연산자
-- 참고: 자바에선== 동등비교 연산자 / = 대입연산자
select ename from emp;
select ename, empno from emp where ename = 'SMITH' ;
-- 18. 부서번호가 10번 부서의 직원들 이름, 사번, 부서번호 검색
-- 단, 사번은 내림차순 검색
select deptno from emp;
select ename, empno, deptno from emp where deptno = 10 order by desc;
-- 19. ? 기본 syntax를 기반으로
-- emp table 사용하면서 문제 만들기
-- 부서 번호는 오름차순, 단 해당 부서에 속한 직원 번호도 오름차순
select ename, empno, deptno from emp order by deptno asc, empno asc;

```

### 테이블 제약조건 및 트랜잭션 주의사항

**dept table 생성**
한 부서 표현 속성 : 부서번호(중복 불허)/ 부서명 / 지역 주의 사항: varchar(size 필수)
제약 사항 : pk_dept라는 이름으로 기본 키 설정 설정 방법 : 컬럼 선언 또는 선언 후 마지막 라인에 선언, table 생성 후 alter 명령어로 제약 조건 추가

insert/update/ deete 문장에 해당
**실행 후 임시 저장소에 저장 사용을 위해서는 commit이라는 영구 저장 필수**
tool들은 commit 자체적으로 제공 (DBeaver : auto)
insert/update/delete commit : 단일 작업으로 간주
transaction : 작은 단위의 실행 여러 개가 하나의 작업으로 관리되는 메커니즘 ex) 계좌 이체