# [FISA 클라우드 엔지니어링 4일차]

### 학습내용

- 최적화된 개발 구조
    - 정형화된 구조
    - 안정적 유지보수 확장이 용이한 선호하는 구조
    - 디자인패턴

디자인 패턴

1. MVC
    1. M:Model/핵심/biz/core
    2. View/presentation
    3. Controller
2. Builder
3. Singleton Pattern
    1. 객체 수를 강제적으로 서버에 하개만 유지할 수 있게하는 기술
4. DAO(Data Access Object)
    1. db연동 전담 클래스
    2. table명DAO.java 권장
5. DTO(Data Transfer Object)
    1. 각 tier별 주고받는 데이터 구조
        1. Customer..
    2. Value Object(VO)
    3. 개발구조
        1. 맴버변수, 기본생성자, 생성자, setter, setter, toString()
    4. 클래스명 권장 규약
        1. CustomerDTO.java, CustomerVO.java,  CustomerBean.java, Customer.java

학습내용

- 배열
- Collection Framework
    - Framework : 스펙에 맞게 개발하는 구조를 의미
    - library : 구조는 자유롭게 단 필요시에 함수(메소드)만 호출해서 사용

MVC 구조 적용

1. Mysql에 people 정보 저장
    1. crud
        1. create, insert
        2. read, select
        3. update, update
        4. delete, delete
2. app으로 설계
    1. rdbms 연동코드 (sql) - ~DAO
    2. client 요청 받는 코드 - 화면, 받은 코드, 받은 코드(controller)
    3. 요청받은 기능 수행 코드 - 핵심 기능, Model, Biz
        1. 검색,삭제,수정
    4. 뷰 기능의 수행 코드 - 화면
    5. 데이터 전송하는 코드 - DTO
3. People table 존재한다는 가정하에 클래스
    1. DTO - Model
    2. DAO - Model
    3. Controller
    4. View
    5. 트랜잭션, 동시성 다양한 데이터를 관리하는 핵심기능 - service: Model
4. 실습 구조
    1. PeopleDTO.java
    2. model.PeopleDAO.java - sql
    3. view.StartView.java - main, 브라우저 첫 인트로 화면이라 가정
    4. view.EndView.java 요청 결과값 출력하는 브라우저 화면이라 가정
    5. Controler.Controller.java 브라우저로부터 받은 요청 구분해서 핵심 로직 지정 및 실행
5. 고객 관점에서 발생 가능한 경우의 수
    1. 검색 하고자 하는 데이터가 없을 때?
    2. 시스템 장애
        1. 
    3. 존재하는 데이터 검색, 수정, 삭제
    4. client의 실수로 입력 자체가 무효한 데이터로 요청
    5. 고가용성, 내구성 설계 (인프라
    6. 가급적 안정적인 고려를 코드에 반영 - 예외 처리
6. Exception 처리
    1. 발생 가능한 불미스러운 경미한 에러를 처리하는 메커니즘
    2. 참고
        1. error : 처리 불가

### 자바배열

```java
/**
 * 배열
 * 	- 하나의 변수로 다수의 데이터 고나리
 *  - 데이터 구분은 고유한 index
 *  - index의 시작은 0부터
 *  - 모든 배열은 객체(heap 메모리에 저장)
 *  
 *  
 *  - 선언 위치
 *  1. class {} - 맴버변수, 객체 생성 시 heap에 객체 내부에 생성
 *  	- 기본값으로 자동초기화(heap 메모리에 생성)
 *  2. 생성자 parameter - 생성자 실행 중에만 사용 가능한 로컬 변수
 *  3. 생성자 {} -  생성자 실행 중에만 사용 가능한 로컬 변수
 *  4. 메소드 parameter - 메소드 실행 중에만 사용 가능한 로컬변수
 *  5. 메소드 {} - 메소드 실행 중에만 사용 가능한 로컬변수
 *  
 *  맴버 변수 : heap에 생성, 기본갑승로 자동 초기화
 * - 정수 0/ 실수 0.0/ boolean flase/ char 'u0000' / 참조 null
 *  로컬 변수 : 자동 초기화 안됨
 *  	     로컬 변수는 선언시 기본값으로라도 초기화 권장
 *  
 *   * 참고 로컬 변수이나 데이터 타입이 배열인 경우는 값을 각 index 별로 초기화 하지 않아도 자동 초기화 진행
 */

package step01_basic;

import org.junit.Test;

import model.domain.PeopleDTO;

public class ArraySyntax {
	// 1차원 배열 정석 문법
	public void m1() {
		//int 1, 2, 3
		int [] i = new int[3]; // 메모리 생성
		i[0] = 10;
		int i2 = 0;
		System.out.println(i[0]);
		System.out.println(i2);
		
		System.out.println("***");
		// String 참조 타입으로 배열 생성 /값 대입 /활용
		// People DTO 참조 타입으로 배열 생성/ 값 대입 / 활용
		String[] s = new String[2];
		System.out.println(s[0]);
		s[0] = "test";
		System.out.println(s[0]);
		
		// ? peopleDTO 참조타입으로 배열 생성/값 대입/ 활용
		// PeopleDTO 0번째 저장 후 이름값 출력
		PeopleDTO[] p = new PeopleDTO[2];
		p[0] = new PeopleDTO("성호",10);
		System.out.println(p[0].getName());
		// PeopleDTO[] 배열
		// p[0]은 People타입
		
	}
	public void m2() {
		int[] i = {1,2,3};
		String [] s = {"a",null};
		PeopleDTO[] p = {null, new PeopleDTO("재석",3)};
	}
	
	public PeopleDTO getPeople() {
		PeopleDTO[] p = {null, new PeopleDTO("재석",3)};
		
		return p[1]; // 1번째값 반환
	}
	
	// 1차원 배열 함축 문법

	// test 메소드
	@Test
	public void running() {
//		ArraySyntax s = new ArraySyntax();
//		s.m1();
		System.out.println(getPeople().getName());
		System.out.println(getPeople().getAge());
	}
}

```

- 핵심은 객체 배열을 생성하고 배열의 요소에 인스턴스를 생성하여 각각 사용하는 방법을 아는 것

### 상속, 다형성

- 최상위 부모 클래스는 Object
- 부모클래스는 모든 자식 클래스를 타입으로 생성할 수 있다.
- 대신에 오버라이딩을 통해 확장한 메서드는 해당 자식 클래스를 통해 사용할 수 있다.

```java
class People{
    String name;
    int age;
}

class Employee extends People{
    int salary;

    public Employee(String name, int age, int salary) {
        this.name = name;
        this.age = age;
        this.salary = salary;
    }
}

class Customer extends People{
    int customer_id;
}

public class EX01 {

    public static void main(String[] args) {
        Object [] p = {new Employee("사원1",22,3600), new Customer(), null};

        System.out.println(((People)p[0]).name);
    }

}
```

### 예외처리

- 복구가 불가능한 문제 상황
    - Error
    - StackOverFlowError (무한재귀 같은 문제상황)
- 복구가 가능한 문제 상황
    - NullPointerException
    - 컴파일 Exception 예외처리 문장 필수
    - 런타임 Exception 예외 처리 문장 불필요(outbound와 같은 문제), 코드 수정으로 해결

### 클래스레벨에서 컴파일 객체레벨 변수

```java
public class PeopleDAO {
	//객체 생성후에만 사용 가능한 일반 변수
	static ArrayList<PeopleDTO> p = new ArrayList();
	//db처럼 멤버 변수를 구성
	
	static {
		p.add(new PeopleDTO("재석", 10));
		p.add(new PeopleDTO("연아", 40));
	}
	
	//select : 모든 사람 정보 반환
	//select * from people;
	//select * from people where id = 'test'
	public static ArrayList<PeopleDTO> getAllPeople(){
		return p;
	}
	
	//이름으로 그 사람의 나이만 수정
	//필요한 데이터 : 이름, 나이
//	public int updatePeople(String name, int newAge) {
		//update people set age=? where name=?
		
//	}
}

```

- static은 클래스를 컴파일할 때 클래스 레벨에서 이미 실행 준비할 수 있도록 준비해둔다. 위 코드를 DB처럼 사용하려면 PeopleDAO를 초기화 했을 때 모든 데이터를 초기화 해두어야한다.
- static을 안쓰고 ArrayList를 생성하면 클래스를 초기화할 때 사용할 수 없기에 static을 붙어서 사용하도록 한다.