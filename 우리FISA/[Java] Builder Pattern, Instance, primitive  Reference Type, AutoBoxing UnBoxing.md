# [Java] Builder Pattern, Instance, primitive / Reference Type, AutoBoxing/ UnBoxing

### Lombok Builder 패턴

```java
/**
 * sts에서는 라이브러리 다운받아서 java -jar 롬복라이브러리.jar 실행 후 sts exe 실행파일 지정
 * sts.imi 파일에 jar의 상세경로가 기재됨.
 * pom.xml 에 의존성 추가 <dependencies> <dependency>레벨에서 사용
 */

package model.domain;

import org.junit.Test;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.NonNull;
import lombok.RequiredArgsConstructor;
import lombok.Setter;

@NoArgsConstructor
@AllArgsConstructor
@Getter
@Setter
// step02 - builder 코드
@Builder
public class Customer {
	public String pw;
	public int age;
	public String grade;
	
	@Test
	public void test() {
		// pw만 명시적으로 초기화 해서 객체 요청
		Customer c = Customer.builder().pw("비밀번호").build();
		Customer c2 = Customer.builder().grade("등급").build();
		// pw, age 초기화 해서 객체 생성 요청
	}
}

```

- builder 어노테이션을 사용하면 원하는 변수를 초기화해서 객체 생성 가능
    - builder 패턴을 쉽게 사용하도록 만들어주는 lombok 어노테이션
- builder와 NoArgsConstructor를 함께 사용하려면 AllArgsConstructor를 같이 사용해야한다.
    - 이유는 builder의 조건이 모든 필드가 초기화가 가능해야한다는 조건이 있는데 기본 생성자를 만들면 아무 필드도 초기화할 수 없기에 사용이 불가하다.
    - 때문에 모든 필드를 초기화해주는 AllArgsConstructor를 함께 사용해야한다.

- Customer.builder()
    - Customer class static 메소드
    - CustomerBuilder를 반환하는 메소드

![image](https://github.com/user-attachments/assets/2ccf95e4-2e2d-477c-adf6-8e9106418e4a)

- Customer에서 builder() 생성 가능 builder()는 CustomerBuilder 클래스 반환
- CustomerBuilder에서 하위 변수에 임의의 초기화 가능
    - pw, age 초기화 해서 객체 생성 요청
    - Customer c3 = Customer.*builder*().pw("비밀번호").age(30).build();
- 마지막은 builder()를 사용해서 Customer 반환

![image](https://github.com/user-attachments/assets/6feeab98-58c4-45d2-8de2-2c069b93e715)

- NoArgsConstructor를 사용함으로써 기본 생성자를 만듬
- AllArgsConstructor를 사용함으로써 모든 필드에 대해 생성자를 만듬
- RequiredArgsConstuctor를 사용하면 꼭 초기화 해야하는 필드를 지정해줄 수 있음 지정은 NonNull로 함.
    - 필수적으로 생성되어야 할 필드만 생성자를 생성하는 역할 (필수 필드 지정은 @NonNull, Final로 지정)
- 오른쪽 상단의 Outline을 살펴보면 위의 어노테이션에 따라서 기본생성자, 모든 필드의 생성자, nonNull이 들어간 필드에 한해서 생성자가 만들어짐
    - Person()
    - Person(Stirng, int, String)
    - Persion(String, String)
    - JPA Notnull와 다름
    
    - 자바 기본
        - 힙에는 GC가 존재하여 메모리를 청소한다.
        - 자바는 보유한 맴버가 메모리에 완전히 생성된 후 실행한다.
    
    ```java
    /**
     * 학습 내용
     * 1. 주목적 : 타인 개발 코드 분석의 용이, api 활용 역량 강화
     */
    
    package step02;
    
    import org.junit.Test;
    
    class A{
    	String message = "fisa3기";
    	A(){
    		System.out.println("A");
    	}
    }
    class B{
    	A a = new A(); // A타입의 데이터, 사용자 정의 타입
    	B(){
    		System.out.println("B");
    	}
    	
    	
    }
    class C{
    	B b = new B();
    	C(){
    		System.out.println("C");
    	}
    	
    }
    public class ApplyTest {
    	@Test
    	public void lab() {
    		C c = new C();
    		System.out.println("c:" + c);
    		System.out.println(c.b.a.message);
    		// ? 1. 더이상의 객체 생성 불가 new 생성자 호출 불가
    		// 생성되는 객체 수 몇개? 3개
    		// 생성되는 객체 순서는? + 확인은 어떻게? A -> B -> C
    		// A의 message 변수값 출력 
    	}
    }
    
    ```
    
    - 자바는 모든 값을 초기화 한다음에 실행한다.
    - 객체 생성 순서 : A → B → C
    - 참조구조 : String 객체 → A객체 → B객체 → C객체
    - 서로 주소를 참조하고 있다. String도 객체다.
    - A객체에서 가르키고 있는 String 객체와 연결을 끊고 String 객체를 GC가 처리하게 만드려면 String의 값을 다른 String 값으로 옮기고 message 출력을 안 하면 된다.
        - 또는 String의 값을 null로 초기화 시킨다.
    
    ### String 타입
    
    - String 타입은 불변 객체다.
    - String 객체의 메모리 데이터는 업데이트 불가
        - 조금이라도 수정되는 문자열이 있으면 무조건 힙 메모리에 객체 새로 생성
    - StringBuffer 타입의 객체들 이미 생성된 메모리의 문자열 데이터 업데이트 가능
    - sb 인스턴스는 stack에 저장, StringBuffer 객체 자체는 heap 에 저장 stack에 있는 인스턴스 지역변수는 heap 메모리의 주소를 참조해서 사용
    
    ### StringBuffer 사용
    
    ```java
    /**
     * 학습 내용
     * 1. 주목적 : 타인 개발 코드 분석의 용이, api 활용 역량 강화
     */
    
    package step02;
    
    import org.junit.Test;
    
    class A{
    	String message = "fisa3기";
    	int no = 10;
    	StringBuffer msg2 = new StringBuffer("스트링 타입");
    	A(){
    		System.out.println("A");
    	}
    }
    class B{
    	A a = new A(); // A타입의 데이터, 사용자 정의 타입
    	B(){
    		System.out.println("B");
    	}
    	
    	
    }
    class C{
    	B b = new B();
    	C(){
    		System.out.println("C");
    	}
    	
    }
    public class ApplyTest {
    	@Test
    	public void lab() {
    		C c = new C() ;
    		/* 더이상의 객체 생성 불가 new 생성자호출 불가 
    		 * ? 1. 생성되는 객체 수 몇개?
    		 * ? 2. 생성되는 객체 순서는? + 확인은 어떻게?
    		 * ? 3. A의 message 변수값 출력 
    		 */
    		 String v = c.b.a.message;
    		 System.out.println(v);  //fisa3기
    		 
    		 c.b.a.message = "상암";
    		 System.out.println(c.b.a.message);// 상암
    		 System.out.println(v); //fisa3기
    		 
    		 System.out.println("****2***");
    		 int v2 = c.b.a.no;
    		 System.out.println(v2);//10
    		 c.b.a.no = 3;
    		 System.out.println(c.b.a.no);// 3
    		 System.out.println(v2); //10
    		 
    		 System.out.println("****3***");
    		 System.out.println(c.b.a.msg2);//새로운 문자타입
    		 
    		 // stringbuffer에서 append 발생 시 새로운 string 객체를 생성한 이후 해당 객체를 내부적으로 append 요청한
    		 // string 객체에 추가하고, 힙에 생성한 임시 string 객체의 참조값을 해제한다. 그리고 나중에 gc가 청소한다.
    		 // stringbuffer를 사용하면 string의 원래 생성된 주소값에 string 내용을 변경할 수 있다.
    		 c.b.a.msg2.append(" new data");
    		 System.out.println(c.b.a.msg2);//새로운 문자타입 new data
    	}
    }
    ```
    
    ### 자바의 형변환
    
    - 바이트가 큰 것을 작은 것에 넣으려면 압축을 해야해서 형변환이 필요하고, 작은 것을 큰 것으로 바꾼다면 그냥 넣어주면 된다.
    
![image](https://github.com/user-attachments/assets/20c411fd-d6b0-448d-864a-4939040d736e)
    
### 자바 타입
    
    1. 기본 primitive 타입
        1. 원시값을 나타내는 자료형
        2. 원자성을 나타내는 단일값으로 이뤄짐
        3. 할당되는 메모리 공간에 값을 직접 저장하는 타입
        4. built-in 타입(내장타입)
        5. class 없이 이미 세팅된 타입
    2. Reference 객체 타입
        1. 참조값을 나타내는 자료형
        2. 생성된 객체를 가르키는 참조 값을 나타냄
        3. class 및 interface를 기반으로한 타입
        4. 사용자 정의 타입 참조타입의 default 값 null


    
### DownCasting / Up Casting

| 구분     | 이름    | 문법                     | 자동형변환(down casting)                        | 명시적인 형변환(up casting)                    |
|----------|---------|--------------------------|------------------------------------------------|------------------------------------------------|
| 논리형   | boolean | boolean var = true;       | 형변환 불가                                    | 형변환 불가                                    |
| 문자형   | char    | char var = 'a';           | byte var1 = 3;                                 | char var1 = 'A';                               |
|          |         | char var = '가';          | char var2 = var1;                              | int var2 = var1;                               |
| 정수형   | byte    | byte var = 1;             | byte var2 = 3;                                 | int var1 = 3;                                  |
|          |         |                          | int var2 = var1;                               | byte var2 = (byte)var1;                        |
|          | short   | short var = 1;            | short var1 = 3;                                | int var1 = 3;                                  |
|          |         |                          | int var2 = var1;                               | short var2 = (short)var1;                      |
|          | int     | int var = 1;              | int var1 = 3;                                  | long var1 = 3;                                 |
|          |         |                          | double var2 = var1;                            | double var2 = (int)var1;                       |
|          | long    | long var = 1;             | long var1 = 3;                                 | float var1 = 3.4F;                             |
|          |         |                          | float var2 = var1;                             | long var2 = (long)var1;                        |
| 실수형   | float   | float var = 1.7F;         | 모든 정수는 float과 double 타입에 자동 형변환 | 모든 실수는 모든 정수에 명시적인 형변환         |
|          |         | float var = 1.7f;         |                                                |                                                |
|          | double  | double var = 3.5D         | float var1 = 3.5f;                             | double var1 = 3.5;                             |
|          |         | double var = 35d;         | double var2 = var1;                            | float var2 = (float)var1;                      |




| 구분              | 이름          | 할당 크기 및 표현 범위                                                |
|-------------------|---------------|--------------------------------------------------------------------|
| 논리형            | boolean       | 1bit, true/false, 형변환 불가                                         |
| 문자형            | char(1글자)   | 16bit unicode                                                      |
| 정수형 (메모리 절약) | byte          | 8bit, -128 ~ 127                                                   |
|                   | short         | 16bit, -32768 ~ 32767                                              |
|                   | int           | 32bit, -2147483648 ~ 2147483647                                     |
|                   | long          | 64bit, -9223372036854775808 ~ 9223372036854775807                   |
| 실수형            | float         | 32bit, IEEE 754 single 표준을 따름                                    |
|                   | double        | 64bit, IEEE 754 double 표준을 따름                                    |



### AutoBoxing, UnBoxing 자동변환
```java
// 자바 1.5 부터는 다음 문법이 사용 가능하다.
int i = Integer(5); // unboxing
Integer a = new Ineger(3); // autoboxing
```



- Integer는 레퍼런스 객체타입인데 int i는 원시타입이다. 객체타입의 데이터를 원소타입으로 자동으로 바꿔주는 것이 unboxing
- 반대로 원시타입의 데이터를 객체타입으로 바꿔주는 것을 autoboxing이라고 한다.
    
### 빈번히 사용되는 객체타입을 원시타입으로 알아서 바꿔주는 자바(java.lang 패키지)
    
- String도 객체타입이지만 단순히 “”를 사용해도 “이름”.toCharArray(); 이런거 사용 가능하다. “”만 사용해도 자동으로 String 객체를 생성해준다는 뜻
- 배열도 new 인스턴스 생성 없이 단순히 int [] a = {1,2,3} 이렇게 생성 가능한 것도 객체를 원시타입으로 변경해주기에 가능한 일이다.
    
### v3 변수가 출력이 되지 않는 이유

    
    ```java
    public class WapperAPI {
    	//?
    	static String m1() { // 반환 타입 없음
    		return "fisa";
    	}
    	
    	Integer m2() { // unboxing
    		return 3;
    	}
    	
    	int m3() {
    		Integer i = 5; // autoboxing
    		return i;
    	}
    	
    	
    	public static void main(String[] args) {
    		WapperAPI wa = new WapperAPI();
    		int v1 = wa.m2();
    		System.out.println(v1);
    		int v2 = Integer.valueOf("3"); // Integer에서 제공하는 메소드
    		System.out.println(v2);
    		int v3 = Integer.valueOf("오"); // 안되는 이유
    		// Integer.valueOf 안에 들어있는 Integer.parseInt 메서드가 실행되면서
    		// 10진수, 2진수, 8진수 등등이 맞는지 확인하고 맞으면 들어간 후 Character.digit 메서드로
    		// 문자열 요소 하나하나를 정수로 변환하려고함 13이면 1,3 이렇게
    		// 그런데 digit를 하는 도중에 숫자가 아닌 "오"라서 NumberFormatException 출력
    		System.out.println(v3);
    	}
    ```
- Integer.parseInt 객체를 살펴보면 Character.digit 메서드에서 숫자가 아니라서 예외를 발생시킨다.