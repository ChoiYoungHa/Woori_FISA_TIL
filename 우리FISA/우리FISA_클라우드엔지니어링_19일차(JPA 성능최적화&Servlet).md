# FISA 클라우드엔지니어링 19일차 (JPA 성능최적화 & Servlet)

### EntityManager Find 메서드 자동 형변환

```java
@Test
	public void step02Test() {
		EntityManager em = DBUtil.getEntityManager();
		
		//step01 - team 검색
		// 1 - 정수(int, 32bit) / Team4의 pk 변수는 long(64bit)
		// int -> long으로 자동 형변환(기본 타입의 기질)
		// Integer 와 Long 객체타입 자동형변환 불가 
		// 1 -> new Integer(1) 자동변환
		// 문제는 Integer가 아닌 Long 타입 필요 따라서 Long객체로 변환 가능한 값 표현
		// long 값 표현시 l or L
		Team4 team = em.find(Team4.class, 1L);  
		System.out.println(team.getTeamName());	
		
		em.close();
		em = null;
	}
```

- Member PK에 정의된 기본키의 값이 int면 1L을 넣어도 Long을 int로 변환해주지만, Long으로 정의되어 있는 필드에 1 int 값을 넣으면 형변환이 안되기에 에러가 발생한다.
- private int id; 로 정의 되어 있다면 1L을 넣어도 1로 변환가능
- private Long id;로 정의 되어 있다면 1을 넣으면 1L로 형변환 불가능

### 지연로딩과 즉시로딩

```java
  @NonNull
	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name="team_id")
	private Team4 teamId;
```

- 지연로딩을 사용하면 엔티티 객체를 전체 조인하지 않고 select문을 나누어 데이터를 가져온다.
- 지연로딩 사용권장

```java
 	@NonNull
	// @ManyToOne // Member 하나는 Team 하나에 소속
	@ManyToOne
	@JoinColumn(name="team_id")  // Team4의 pk 변수에 선언된 매핑된 컬럼명, fk 설정
	private Team4 teamId;
```

- 즉시로딩을 사용하면 사용관계와 관련 없이 모든 객체를 조인하여 데이터를 가져온다.
- ManyToOne 어노테이션의 기본값은 ***EAGER***

### 객체탐색 참조보다 fetch join문을 사용해야하는 이유

```java
List<Emp> rs1 = em.createQuery("select E from Emp E where sal > :sal", Emp.class).setParameter("sal", 2000)
					.getResultList();
			rs1.stream().forEach(a -> System.out.println(a.getDeptno().getDname()));

select emp0_.EMPNO as EMPNO1_1_, emp0_.COMM as COMM2_1_,
 emp0_.deptno as deptno8_1_, emp0_.ENAME as ENAME3_1_,
 emp0_.HIREDATE as HIREDATE4_1_, emp0_.JOB as JOB5_1_,
 emp0_.MGR as MGR6_1_, emp0_.SAL as SAL7_1_ from Emp emp0_ where emp0_.SAL>?
 
select dept0_.DEPTNO as DEPTNO1_0_0_, dept0_.DNAME as DNAME2_0_0_,
dept0_.LOC as LOC3_0_0_ from Dept dept0_ where dept0_.DEPTNO=?

select dept0_.DEPTNO as DEPTNO1_0_0_, dept0_.DNAME as DNAME2_0_0_,
dept0_.LOC as LOC3_0_0_ from Dept dept0_ where dept0_.DEPTNO=?

select dept0_.DEPTNO as DEPTNO1_0_0_, dept0_.DNAME as DNAME2_0_0_,
dept0_.LOC as LOC3_0_0_ from Dept dept0_ where dept0_.DEPTNO=?
```

- 위의 jqpl에서는 E객체만 가져오도록 되어 있는데 foreach로 출력하면서 dept 테이블 객체를 참조하고 있다.
- 연봉 2000만원 이상인 부서이름을 출력하기 위해서 dept를 참조하게 된다.

```java
ACCOUNTING 10
SALES 30
ACCOUNTING 10
RESEARCH 20
RESEARCH 20
```

- 10번부서 2명, 20번부서 2명, 30번 부서 1명 각각 10,20,30 부서에 해당하는 사원을 불러왔기에 select문이 3번 발생한다.
- 만약에 jqpl을 위처럼 작성했을 때 데이터가 100만건이 있으면 select문이 최소한 90만건 이상은 발생하게 된다.
- 이런 상황에서는 패치 조인문을 사용해서 한번에 객체를 묶어서 데이터를 가져오는 것이 성능에 더 유익하다.

```java
List<Emp> rs1 = em.createQuery(
    "select E from Emp E join fetch E.deptno D where E.sal > :sal", Emp.class)
    .setParameter("sal", 2000)
    .getResultList();

rs1.stream().forEach(a -> System.out.println(a.getDeptno().getDname() + a.getDeptno().getDeptno()));

select emp0_.EMPNO as EMPNO1_1_0_, dept1_.DEPTNO as DEPTNO1_0_1_,
 emp0_.COMM as COMM2_1_0_, emp0_.deptno as deptno8_1_0_,
 emp0_.ENAME as ENAME3_1_0_, emp0_.HIREDATE as HIREDATE4_1_0_,
 emp0_.JOB as JOB5_1_0_, emp0_.MGR as MGR6_1_0_,
 emp0_.SAL as SAL7_1_0_, dept1_.DNAME as DNAME2_0_1_,
 dept1_.LOC as LOC3_0_1_ from Emp emp0_ cross join Dept dept1_ 
 where emp0_.deptno=dept1_.DEPTNO and emp0_.SAL>?
```

- 조인문을 사용하면 쿼리가 한번만 발생한다.

---

### Servlet

**Web Server : html,css,js,img 영상 등 서비스하는 서버**

**Web Container = Servlet engine = tomcat : java web 서비스 서버 (Web Application Server[WAS])**

### Web Server 와 Web Application Server 구분하는 이유

1. 기능을 분리하여 서버 부하 방지
2. WAS는 외부와의 간접적인 연결로 인해 보안을 고려해야 하는 파일 및 리소스들을 안전하게 보호
3. 다수의 Web Server와 여러대의 WAS를 연결하여 수많은 요청을 분산해서 동일한 Application Server를 실행하면서 안정적으로 운영

Servlet 객체

개발환경 

1. tomcat10
2. 브라우저

개발 소스 종류

- *.java
    - 순수자바(http 통신 불가능)
    - http 요청 처리 가능한 servlet
- *.jsp
    - html + css + js + java 코드 다 구현 가능한 스펙
- *.html
    - html / css / js 코드만 구현, 화면 처리용
- *.css
    - 브라우저 출력을 보기 좋게 꾸미는 구성코드
- *.js
    - javascript 로만 구성된 파일
    - js는 html or jsp 파일에 포함되어 개발되기도 함

1. STS와 실제 실행 URL 비교
    1. sts [ =Version: 4.23.1.RELEASE ] web 개발구조
        
        project
        
        - javaResource
        - src
            - main
                - java
                    - *.java (순수자바와 servlet)
                - webapp
                    - *.html
                    - *.jsp
                - WEB-INF
                    - web.xml
2. 주요 API
    1. 구현은 하나 호출 코드는 개발자 권한이 아님
    2. 호출 시점 client가 url 입력 후 요청시 자동실행
    3. 호출은 요청받고 get, post 확인 후 실행
        1. web container 내부 로직
    4. doGet()
        1. 호출시점
        2. 브라우저 주소줄 요청시 자동실행
        3. postman에서 get 방식 지정해서 자동실행
        4. get방식은 별도의 설정 없이도 url 입력해서 요청하면 이동
    5. doPost()
        1. 호출시점
        2. postman에서 Post방식 지정해서 자동실행