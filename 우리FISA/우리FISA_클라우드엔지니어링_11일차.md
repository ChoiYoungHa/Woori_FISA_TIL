# FISA 클라우드엔지니어링 11일차

### SQL문 function 사용하기

```bash
-- *** [숫자함수] ***
-- 1. 절대값 구하는 함수 : abs()
select 3.5, -3.5, +3.5;
select 3.5, abs(-3.5), +3.5;

-- 2. 반올림 구하는 함수 : round(데이터 [, 반올림자릿수])
-- 반올림자릿수 : 소수점을 기준으로 양수는 소수점 이하 자리수 의미
         -- 음수인 경우 정수자릿수 의미
select 10/3;
select round(10/3); -- 소수 다 버림
select ROUND(10/3, 2); -- 둘째자리까지 표시

-- 3. 지정한 자리수 이하 버리는 함수 : TRUNCATE()
-- 반올림 미적용
-- truncate(데이터, 소수자릿수)
-- 자릿수 : +(소수점 이하), -(정수 의미)
-- 참고 : 존재하는 table의 데이터만 삭제시 방법 : delete[복원]/truncate[복원불가]
select TRUNCATE(10/3, 0); 

show tables;
-- 원본 테이블 복제 : 제약조건 제외한 구조와 데이터 복제
create table emp01 as select * from emp;
desc emp01; -- 제약조건은 없음
select * from emp01;
-- 실습전 auto commit 설정 꼭 풀어놓기
DELETE from emp01;
select * from emp01;
rollback;

-- truncate는 삭제 후 바로 commit rollback 안됨, 대신 삭제속도가 빠름
TRUNCATE emp01;
rollback;

-- 4. 나누고 난 나머지 값 연산 함수 : mod()
-- 모듈러스 연산자, % 표기로 연산, 오라클에선 mod() 함수명 사용
select 10/2;
SELECT mod(10, 3);

-- 5. ? emp table에서 사번(empno)이 홀수인 사원의 이름(ename), 
-- 사번(empno) 검색 
SELECT empno from emp;
select ename, empno from emp where MOD(empno, 2) != 0;

-- *** [문자함수] ***
/* tip : 영어 대소문자 의미하는 단어들
대문자 : upper
소문자 : lower
철자 : case 
*/
-- 1. 대문자로 변화시키는 함수
-- upper() : 대문자[uppercase]
-- lower() : 소문자[lowercase]

-- 2. ? manager로 job 칼럼과 뜻이 일치되는 사원의 사원명 검색하기 
-- mysql은 데이터값의 대소문자 구분없이 검색 가능
-- 해결책 1 : binary()  대소문자 구분을 위한 함수
-- 해결책 2 : alter 명령어로 처리
create table emp02 as select ename, empno, job from emp;
select * from emp02;
drop table emp02;

select ename, job from emp02 where job = 'MANAGER';

-- 3. 문자열 길이 체크함수 : length()
-- 문자열의 길이를 byte 단위로 반환
/**
 * 현 mysql 버전에서는 한글자당 3byte 사용, 원래는 2byte임 정상적인 db는 2byte 사용함.
 */
create table dept01 as select dname from dept;
desc dept01;

select * from dept01;
select dname, LENGTH(dname) from dept01;
insert into dept01 values ('상암');

-- 4. 문자열 일부 추출 함수 : substr()
-- 서브스트링 : 하나의 문자열에서 일부 언어 발췌하는 로직의 표현
SELECT SUBSTR(dname, 1, 2) from dept; 

-- substr(데이터, 시작위치, 추출할 개수)
-- 시작위치 : 1부터 시작

-- 5. ? 년도 구분없이 2월에 입사한 사원(mm = 02)이름, 입사일 검색
-- date 타입에도 substr() 함수 사용 가능
-- 문자열 index 시작 - 1 
select hiredate from emp;

-- 년도만 검색
select ename, hiredate from emp where SUBSTR(hiredate, 1, 4) = 1981;
-- 월만 검색
select ename, hiredate from emp where SUBSTR(hiredate, 6, 2) = 02;
-- 일만 검색
select ename, hiredate from emp where SUBSTR(hiredate, 9, 2) = 03;

-- 7. 문자열 앞뒤의 잉여 여백 제거 함수 : trim()
/* length(trim(' abc ')) 실행 순서
   ' abc ' 문자열에 디비에 생성
   trim() 호출해서 잉여 여백제거
   trim() 결과값으로 length() 실행 */

select ' abc ', CHAR_LENGTH(' abc '), LENGTH(' abc '), LENGTH(TRIM(' abc '));

-- *** [날짜 함수] ***
-- 1. ?어제, 오늘, 내일 날짜 검색 
-- 현재 시스템 날짜에 대한 정보 제공 함수
-- sysdate() & now(): 날짜 시분 초
-- curdate() : 날짜
select SYSDATE(), NOW(), CURDATE();

SELECT * from emp;
-- 2.?emp table에서 근무일수 계산하기, 사번과 근무일수 검색
select empno, CURDATE() - hiredate as '근무일수' from emp; 

-- 3. ? 교육시작 경과일수
-- 순수 문자열을 날짜 형식으로 변환해서 검색
/* 
	yy/mm/dd 포멧으로 연산시에는 반드시 to_date() 라는 날짜 포멧으로
	변경하는 함수 필수 
	단순 숫자 형식으로 문자 데이터 연산시 정상 연산 
*/
select CURDATE() - 20240708;

-- 4. 문자열 날짜로 변경
SELECT str_to_date('2024/07/08', '%Y/%m/%d');
-- 5. 특정 일수 및 개월수 더하는 함수 : ADDATE()
select adddate('2024/07/08', interval 2 day);

select sysdate(), now(), curdate(); 
-- 10일 이후 검색 
select adddate(now(), interval 10 day); 

-- 15분 이후
select adddate(now(), interval 15 minute); 

-- 6. ? emp table에서 입사일 이후 3개월 지난 일자 검색
select hiredate, adddate(hiredate, interval 3 month) as '3개월 지난 일자' from emp; 

-- 7. 두 날짜 사이의 개월수 검색 : months_between()
-- 오늘(sysdate) 기준으로 2021-09-19
select TIMESTAMPDIFF(month,'2021-09-19', CURDATE());
select now(), dayofmonth(now());

-- 특정 기준일로 오늘은 며칠차?(기준일 포함할 경우 +1)
select TIMESTAMPDIFF(day, '2021-09-19', curdate());

-- 8. 오늘을 기준으로 100일은?(오늘이 1일로 계산할 경우 기준일 포함)
select adddate(now(), interval 99 day) as 백일;

-- emp 직원들의 입사일 기준으로 5개월 후의 일자는?
select adddate(hiredate, interval 4 month) from emp; 

-- 9. ?근무 연차(입사하자마자 1년 차로 계산될 경우)
select ename, TIMESTAMPDIFF(year, hiredate, curdate()) + 1 as 연차 from emp;

-- 10. 1년 365일중 오늘은 몇일차?
select TIMESTAMPDIFF(day, '2024-01-01', curdate()) as '이번년도 지난일수';

-- 11. 주어진 날짜를 기준으로 해당 달의 가장 마지막 날짜 : last_day()
select last_day(now());

-- 12.? 2020년 2월의 마지막 날짜는?
select last_day(str_to_date('2020-02-01', '%y-%m-%d')); -- %y 소문자면 함수가 먹히지 않음
SELECT LAST_DAY(STR_TO_DATE('2020-02-01', '%Y-%m-%d')) AS LastDayOfMonth;

-- *** [형변환 함수] ***
-- Data type
-- DATETIME : 'YYYY-MM-DD HH:MM:SS'
-- DATE : 'YYYY-MM-DD'
-- TIME : 'HH:MM:SS'
-- CHAR : String
-- SIGNED : Integer(64bit), 부호 사용 가능
-- UNSIGNED : Integer(64bit), 부호 사용 불가
-- BINARY : binary String

select '1'-1;
select cast('1' as signed); -- mysql은 다른 타입끼리 연산 가능하나 견고한 db에서는 가공 필수

-- 1. cast() - 특정 type으로 형변환
select cast('10' as signed) - cast('10' as signed);
select cast('10100' as signed) - cast('10000' as signed);

-- 2. STR_TO_DATE() : 날짜로 변경 시키는 함수

--  올해 며칠이 지났는지 검색(포멧 yyyy/mm/dd)
-- select sysdate - 20200719; 오류
-- select sysdate - '20200719'; 오류

-- 3. 문자열로 date타입 검색 가능[데이터값 표현이 유연함]
-- 1980년 12월 17일 입사한 직원명 검색
select * from emp where hiredate ='1980-12-17';

-- 4. to_number(문자열, 변환포멧) : 문자열을 숫자로 변환
-- mysql cast() 함수 사용
-- 어떤 숫자 형식으로 변환가능한지에 대한 명확성 필요한 함수 
-- 1. '20,000'의 데이터에서 '10,000' 산술 연산하기 
-- 힌트 - 9 : 실제 데이터의 유효한 자릿수 숫자 의미(자릿수 채우지 않음)
-- ?

-- *** 조건식 함수 ***
-- decode()-if or switch문과 같은 함수 ***
-- decode(조건칼럼, 조건값1,  출력데이터1,
--			   조건값2,  출력데이터2,
--				...,
--			   default값) from table명;

-- 1. deptno 에 따른 출력 데이터
-- 10번 부서는 A로 검색/20번 부서는 B로 검색/그외 무로 검색
select ename, job, sal,
	CASE
		WHEN deptno = 10 THEN 'A부서'
		WHEN deptno = 20 THEN 'B부서'
		ELSE '무'
	END AS level
from emp;

-- 2. emp table의 연봉(sal) 인상계산
-- job이 ANALYST 5%인상(sal*1.05), SALESMAN 은 10%(sal*1.1) 인상, 
-- MANAGER는 15%(sal*1.15), CLERK 20%(sal*1.2) 인상
select ename, job, sal,
	case
		when job = 'ANALYST' then round(sal*1.05)
		when job = 'SALESMAN' then round(sal*1.1)
		when job = 'MANAGER' then round(sal*1.15)
		when job = 'CLERK' then round(sal*1.2)
	end as '연봉협상 후 연봉'
from emp;
	
    

-- 3. 'MANAGER'인 직군은 '갑', 'ANALYST' 직군은 '을', 
-- 나머지는 '병'으로 검색
select ename, sal, job,
	case
		when job = 'MANAGER' then '갑'
		when job = 'ANALYST' then '을'
	end as '계급'
from emp;
```

### 조건식 함수 사용하기

```sql
*** 조건식 함수 ***
-- decode()-if or switch문과 같은 함수 ***
-- decode(조건칼럼, 조건값1,  출력데이터1,
--			   조건값2,  출력데이터2,
--				...,
--			   default값) from table명;

-- 1. deptno 에 따른 출력 데이터
-- 10번 부서는 A로 검색/20번 부서는 B로 검색/그외 무로 검색
select ename, job, sal,
	CASE
		WHEN deptno = 10 THEN 'A부서'
		WHEN deptno = 20 THEN 'B부서'
		ELSE '무'
	END AS level
from emp;

-- 2. emp table의 연봉(sal) 인상계산
-- job이 ANALYST 5%인상(sal*1.05), SALESMAN 은 10%(sal*1.1) 인상, 
-- MANAGER는 15%(sal*1.15), CLERK 20%(sal*1.2) 인상
select ename, job, sal,
	case
		when job = 'ANALYST' then round(sal*1.05)
		when job = 'SALESMAN' then round(sal*1.1)
		when job = 'MANAGER' then round(sal*1.15)
		when job = 'CLERK' then round(sal*1.2)
	end as '연봉협상 후 연봉'
from emp;
	
    

-- 3. 'MANAGER'인 직군은 '갑', 'ANALYST' 직군은 '을', 
-- 나머지는 '병'으로 검색
select ename, sal, job,
	case
		when job = 'MANAGER' then '갑'
		when job = 'ANALYST' then '을'
	end as '계급'
from emp;

```

### Group By 집계함수 사용

```sql
-- 9. 8번문제에 부서번호 내림차순 정렬
select deptno, max(sal) as '최대급여', min(sal)
from emp
group by deptno
having max(sal) < 5000
order by deptno desc;

-- 부서별 가장 연차가 높은 사람 구하기
select deptno, min(hiredate)
from emp
group by deptno;

-- 입사 연도별 직원 수와 반올림하여 평균 급여, 최대급여 구하기
SELECT YEAR(hiredate) AS hire_year, COUNT(*) AS emp_count, AVG(sal) AS avg_salary, max(sal) 
FROM emp
GROUP BY YEAR(hiredate);
```

 

### Join 사용

```sql

-- 동등조인
select ename, empno, loc
from emp, dept
where ename='SMITH' and emp.deptno = dept.deptno;

select loc from dept;
select d.loc, ename, sal from emp e, dept d where e.deptno = d.deptno and loc = 'NEW YORK';

-- *** 2. not-equi 조인 ***
-- salgrade table(급여 등급 관련 table)
select * from salgrade s;

-- 사원의 급여가 몇 등급인지 검색
-- between ~ and : 포함 
select grade, ename, sal from emp, salgrade where sal between losal and hisal;

-- *** 3. self 조인 ***
-- 하나의 테이블을 다수의 테이블로 논리적으로 구분한 조인
-- 11. SMITH 직원의 메니저 이름 검색
-- emp e : 사원 table로 간주 / emp m : 상사 table로 간주
-- emp e 사원, emp m 상사 테이블로 논리적으로 나누기
select m.ename 
from emp e, emp m 
where e.ename = 'SMITH' and e.mgr=m.empno

-- 12. 메니저 이름이 KING(m ename='KING')인 
-- 사원들의 이름(e ename)과 직무(e job) 검색
select e.ename, e.job from emp e, emp m
where m.ename = 'KING' and m.empno = e.mgr;

-- 13. SMITH와 동일한 부서에서 근무하는 사원의 이름 검색
-- 단, SMITH 이름은 제외하면서 검색 : 부정연산자 사용 != 
select friend.ename as 동료이름
from emp my, emp friend
where my.ename = 'SMITH' 
and my.deptno = friend.deptno 
and friend.ename != 'SMITH';
```

### 서브쿼리

```sql
select dname
from dept
where deptno = (select deptno
				from emp
				where ename = 'SMITH'
				);

-- 2. SMITH와 동일한 직급(job)을 가진 사원들의 모든 정보 검색(SMITH 포함)
select *
from emp
where job = (select job from emp where ename = 'SMITH');

-- smith와 동일한 직급을 가진 모든 사람들 스미스는 제외
select *
from emp
where job = (select job from emp where ename = 'SMITH')
and ename != 'SMITH';

-- from select 사용해보기
select empno, ename, sal from emp;
```

### 인라인뷰

```sql
-- empno, ename, sal로만 구성된 table 실제 존재한다고 가정후 작업
-- 인라인뷰 : from 절에 작업 : sql 관점에선 마치 e라는 테이블이 존재한다 생각하고 검색
select empno, t.sal*12
from (select empno, ename, sal from emp) t;
```

### 자바 프로젝트 리팩토링 미션

```sql
// 원본코드
/**
	 * Project의 기부자 수정 - 프로젝트 명으로 검색해서 해당 프로젝트의 기부자 수정
	 * 
	 * @param projectName 프로젝트 이름
	 * @param people      기부자
	 */
	public void donationProjectUpdate(String projectName, Donator people) throws Exception {

	for (TalentDonationProject project : donationProjectList) {
			if (project != null && project.getTalentDonationProjectName().equals(projectName)) {
				if (people != null) {
					project.setProjectDonator(people);
					break;
				} else {
					throw new Exception("프로젝트 이름은 있으나 기부자 정보 누락 재확인 하세요");
				}

			} else {
				throw new Exception("프로젝트 이름과 기부자 정보 재 확인 하세요");
			}
		}
		
	
```

```sql
// 리팩토링
/**
	 * Project의 기부자 수정 - 프로젝트 명으로 검색해서 해당 프로젝트의 기부자 수정
	 * 
	 * @param projectName 프로젝트 이름
	 * @param people      기부자
	 */
	public void donationProjectUpdate(String projectName, Donator people) throws Exception {
		TalentDonationProject project = donationProjectList.stream()
				.filter(p -> p != null && p.getTalentDonationProjectName()
				.equals(projectName))
				.findFirst()
				.orElseThrow(() -> new Exception("프로젝트 이름을 제대로 입력해주세요"));
		
		// 프로젝트 업데이트
		if(people != null) {
			project.setProjectDonator(people);
		}else {
			throw new Exception("프로젝트는 있으나 기부자 정보를 누락하였습니다.");
		}
	}
```