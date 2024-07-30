# [FISA 클라우드엔지니어링 17일차]

### JDBC vs JPA

- jdbc를 추상화 시켜서 코드 10줄을 1줄로 만들어줌

![fisa/jdbc_code.png](fisa/jdbc_code.png)

- 반복적인 커넥션, pstmt객체 생성, resultset 객체 생성
- 각 객체 close 처리 비효율적
- jpa를 사용하면 해당 코드를 줄일 수 있다.

**JPA 코드**

```sql

	@Test
	public void step01() {
		
		EntityManager em = DBUtil.getEntityManager();
		EntityTransaction tx = em.getTransaction();
		tx.begin();
		
		em.persist(new Customer("id1", "연아", 50));
		
		Customer c1 = em.find(Customer.class, 1);
		System.out.println(c1);
		c1.setName("유다연");
		tx.commit();
		em.close();
	}
```

---

### JPA 학습

1. 개발환경 구축
    1. maven
    2. JPA 2.2 & hibernate 5.4
2. 의존관계 Library 버전 확인
    1. JPA Library
        1. hibernate 를 추상화
        2. 버전 확인 후 사용
        3. 개발구조
            1. 설정파일 필요 META-INF/persistence.xml
            해당 버전의 스펙에 맞게 선언구가 명시되어 있어야 한다.
            tag 및 구조도 제시된 스펙에 개발
            2. 해당 API로 개발
        4. 설정 파일 등록
            1. db의 접속정보
            2. db에 table과 매핑되는 자바 클래스명 등록
            3. 설정 파일 재로딩시 table 생성 또는 유지에 대한 설정

---

### JPA 주요 엔티티 어노테이션

- @Entity
- @Id
    - primary key 의미
    - entity 클래스에 필수
- @GeneratedValue(strategy = GenerationType.IDENTITY) // mysql_auto increment
    - 속성 생략이 오라클의 시퀀스로 구동한다.
- @Colunm(unique= true, nullable = false)
    - 생략도 가능
    - 자바 클래스의 맴버 변수와 table의 컬럼명이 일치
    - 해당 컬럼에 대한 옵션 지정가능 (not null, unique 등)
    - name 속성 적용 시 자바 클래스의 변수명과 테이블 컬럼명 다르게 가능

---

### JPA EntityManager

1. **EntityManagerFactory**
    1. EntityManager 객체 생성
    2. xml 설정 정보를 기반으로 해당 db와 entity들 매핑된 정보 read해서 EntityManager 객체생성
2. **EntityManager**
    1. Connection → PrepareStatement → sql 문장 실행 → 결과 활용 → 자원반환의 기능을 처리하는 단일객체
    2. 개별 user가 요청시 해당 객체만 생성해서 제공
    3. 주요 메소드
        1. persist()
        2. find()
        3. entitiy.set()
        4. remove()
        5. close() - ResultSet, Statement, Connection 순으로 자원반환
        6. clear() - persistent context 잔존된 정보를 삭제하는 메소드 clear() 호출 후 find 할 경우엔 무조건 select  재실행
3. **EntityTransaction**
    1. commit과 rollback 필요한 sql 문장
        1. insert, update, delete 로직 처리시에만 EntityTransaction 사용
    2. commit
        1. persist() / entity.set() / remove() 등의 메소드 사용 시 commit 필수
    3. rollback()

---

### EntityManagerFactory, ManagerFactory, Transaction 관계도

1. EntityManagerFactory 1개만 생성
2. ManagerFactory 유저의 트래픽만큼 생성
3. ManagerFactory 1개당 트랜잭션 1개
4. 1:N:1

---

### 영속성 컨텍스트(Persistent Context)

```sql
@Test
	public void step02() {
		EntityManager em = DBUtil.getEntityManager();
		
		em.persist(new Customer("id2", "연아", 50));
		
		Customer c1 = em.find(Customer.class, 1);
		System.out.println("-1-"+c1);
		c1.setName("박지오");
		System.out.println("-2-"+c1);
		
		Customer c2 = em.find(Customer.class, 10);
		
		em.close();
	}
```

- **Customer c1 = em.find(Customer.class, 1);**
    - 영속성 컨텍스트에 Customer에 대한 select 결과가 없기에 select 문 실행
- **c1.setName("박지오");**
    - 영속성 컨텍스트에 반영한다.
- **System.out.println("-2-"+c1);**
    - 정상적으로 박지오라는 이름이 출력이 된다.
- 하지만 Transaction이 없기에 DB에는 반영되지 않는다.
- 위 코드는 영속성컨텍스트를 이해하기 위한 코드다.
- **Customer c2 = em.find(Customer.class, 10);**
    - 위 코드도 영속성 컨텍스트에 10번이 없기 때문에 select 문으로 확인한다.
- 영속성 컨텍스트에 데이터값이 있으면 select를 발생시키지 않고, 데이터값이 없으면 select를 발생시킨다.

```java
 	@Test
	public void step03() {
		EntityManager em = DBUtil.getEntityManager();
		EntityTransaction tx = em.getTransaction();
		
		Customer c = em.find(Customer.class, 1);
		System.out.println(c);
		
		c = em.find(Customer.class, 1);
		
		tx.begin();
		c.setAge(20);
		em.remove(c);
		
		c = em.find(Customer.class, 1);
		System.out.println(c);
		// tx.commit();
	
		em.close();
	}
```

- **Customer c = em.find(Customer.class, 1);**
    - 영속성 컨텍스트에 정보가 없기에 select 발생
- **c = em.find(Customer.class, 1);**
    - 영속성 컨텍스트에서 이미 select를 했기에 select 미발생
- **tx.begin();**
    - 트랜잭션 시작
- **c.setAge(20);
em.remove(c);**
    - update, delete
    - 하지만 commit을 하지 않았기에 영속성 컨텍스트에서만 삭제
- **System.out.println(c);**
    - 마지막에 찍은 print는 null이 나오겠지만 실제 DB에는 데이터가 삭제되지 않음.

---