# FISA 클라우드엔지니어링 18일차 (JPA)

### JPA 연관관계 매핑하기

- 서로 다른 table간의 연관된 데이터를 기준으로 검색 및 관리
- pk & fk 구조
    - 예시
        - 직원 한명은 하나의 부서에 소속 되어야 함
            - emp 관점에서 dept와 1:1 관계
        - 부서 하나는 여러명의 직원들이 소속되어 있음
            - 부서 하나는 직원이 없거나, 한명, 여러명 직원 소속
                - dept 관점에서 emp와 1:n 관계

### Sequence가 공유되는 문제, 데이터무결성 위배되는 문제

**Member1**

```java
package step01.nonjoin;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor 
@Getter
@Setter
@ToString

@Entity
public class Member1 {

	@Id
	@GeneratedValue // 그냥 Sequence 생성
	@Column(name="member_id")
	private long memberId;
	
	@NonNull
	@Column(length = 20)
	private String name;
	
	@NonNull
	@Column(name="team_id")
	private long teamId;
	
}
/*
create table Member1 (
	member_id number(19,0) not null, 
	name varchar2(20), 
	team_id number(19,0), 
	primary key (member_id)
);
 */

```

**Team1**

```java
package step01.nonjoin;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor

@Getter
@Setter
@ToString

@Entity
public class Team1 {

	@Id // pk
	@GeneratedValue //sequence
	@Column(name = "team_id") //team_id
	private long teamId;
	
	//Team1(String teamName)
	@NonNull
	@Column(name="team_name", length = 20)
	private String teamName;
	
}

/*
create sequence hibernate_sequence start with 1 increment by 1
create table Team1 (
	team_id number(19,0) not null, 
	team_name varchar2(20), 
	primary key (team_id)
);
*/
```

**TestCode**

```java
package step01.nonjoin;

import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;

import org.junit.Test;

import util.DBUtil;

public class Step01RunTest {

	@Test
	public void step01Test() {
		EntityManager em = null;
		EntityTransaction tx = null;
		
		try {
			em = DBUtil.getEntityManager();
			tx = em.getTransaction();
			tx.begin();
			
			Team1 t1 = new Team1("축구1팀");
			em.persist(t1);
			em.persist(new Team1("배구1팀"));
			
			em.persist(new Member1("손흥민", 1));  
			em.persist(new Member1("김연경", 2));  
			em.persist(new Member1("박지성", 3)); // 데이터무결성 위배
			
			tx.commit();
			
		}catch(Exception e) {
			tx.rollback();
			e.printStackTrace();
		}finally {
			if(em != null) {
				em.close();
				em = null;
			}
		}
		
	}
}

```

**테이블 결과**

TEAM_ID|TEAM_NAME|
-------+---------+
1|축구1팀     |
2|배구1팀     |

MEMBER_ID|NAME|TEAM_ID|
---------+----+-------+
3|손흥민 |      1|
4|김연경 |      2|
5|박지성 |      3|

- 시퀀스 번호를 공유하고 있으며 TeamId 3이 없지만 박지성 데이터가 팀 3번을 가지고 있다.
- 데이터 무결성을 위배중

### 시퀀스 문제 해결

**Member**

```java
package step02.onetoonejoin;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.RequiredArgsConstructor;
import lombok.NonNull;
import lombok.Getter;
import lombok.Setter;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor 
@Getter
@Setter
@ToString

@SequenceGenerator(name = "member_seq", sequenceName = "member_seq_id", 
				   allocationSize = 50, initialValue = 1)
@Entity
public class Member2 {

	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq") 
	@Column(name="member_id")
	private long memberId;
	
	@NonNull
	@Column(length = 20) //컬럼 사이즈 20byte
	private String name;
	
	@NonNull
	@Column(name="team_id")
	private long teamId;
	
}

```

- @SequenceGenerator, strategy = GenerationType.SEQUENCE 옵션추가
- 각각의 독립된 시퀀스를 생성하여 시퀀스가 공유되는 문제를 해결

**Team**

```java
package step02.onetoonejoin;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor

@Getter
@Setter
@ToString

@SequenceGenerator(name = "team_seq", sequenceName = "team_seq_id",
				   initialValue = 1, allocationSize = 50)
@Entity
public class Team2 {

	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "team_seq")
	@Column(name = "team_id")
	private long teamId;
	
	@NonNull
	@Column(name="team_name", length = 20)
	private String teamName;
}

```

**테이블 결과**

MEMBER_ID|NAME|TEAM_ID|
---------+----+-------+
1|손흥민 |      1|
2|김연경 |      2|
3|박지성 |      3|

TEAM_ID|TEAM_NAME|
-------+---------+
1|축구1팀     |
2|배구1팀     |

### 데이터 무결성, 시퀀스 문제 모두 해결

**Member**

```java
package step03.onetoonejoin;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.OneToOne;
import javax.persistence.SequenceGenerator;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor 
@Getter
@Setter
@ToString

@SequenceGenerator(name = "member3_seq", sequenceName = "member3_seq_id", 
				   allocationSize = 50, initialValue = 1)
@Entity
public class Member3 {

	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member3_seq") 
	@Column(name="member_id")
	private long memberId;
	
	//  not null은 nullable = fasle
	@NonNull
	@Column(length = 20, nullable = false) //컬럼 사이즈 20byte
	private String name;
	
	/*
	 * 
	 */
	@NonNull
	@OneToOne // Member 하나는 Team 하나에 소속
	@JoinColumn(name="team_id")  //Team3의 pk 변수에 선언된 매핑된 컬럼명
	private Team3 teamId;
	
}

```

- NonNull 롬복 어노테이션으로 필수 생성자로 지정하고, 필드 데이터 타입을 Team 객체로 지정.
- OneToOne으로 teamId와 Member 객체의 teamid FK로 매핑

**Team**

```java
package step03.onetoonejoin;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.SequenceGenerator;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;
import lombok.ToString;

@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor

@Getter
@Setter
@ToString

@SequenceGenerator(name = "team3_seq", sequenceName = "team3_seq_id",
				   initialValue = 1, allocationSize = 50)
@Entity
public class Team3 {

	@Id
	@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "team3_seq")
	@Column(name = "team_id")
	private long teamId;
	
	@NonNull
	@Column(name="team_name", length = 20)
	private String teamName;
}

```

**테스트코드**

```java
package step03.onetoonejoin;

import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;

import org.junit.Test;

import util.DBUtil;

public class Step03RunTest {

	@Test
	public void step01Test() {
		EntityManager em = null;
		EntityTransaction tx = null;
		
		try {
			em = DBUtil.getEntityManager();
			tx = em.getTransaction();
			tx.begin();
			
			Team3 t1 = new Team3("축구1팀");
			em.persist(t1);
			em.persist(new Team3("배구1팀"));
			
			em.persist(new Member3("손흥민", t1));  // 이전 코드와는 다르게 객체주입
			em.persist(new Member3("김연경", t1));
			em.persist(new Member3("박찬호", t1));  
			
			tx.commit();
			
		}catch(Exception e) {
			tx.rollback();
			e.printStackTrace();
		}finally {
			if(em != null) {
				em.close();
				em = null;
			}
		}
		
	}
}

```

- Member 테이블 입장에서 1:1, 하나의 맴버는 하나의 부서에 소속되어야 한다.
- TeamId로 1:1 매핑하여 FK 생성
- 존재하지 않는 3번같은 TeamId를 이젠 가질 수 없음
- 데이터 무결성 보장, 시퀀스 번호 중복방지