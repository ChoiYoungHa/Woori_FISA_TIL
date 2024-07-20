# [FISA 클라우드 엔지니어링 6일차]

## Java new Syntax

### Lambda 문법


### :: 출력 문법

- ::은 가공없이 존재하는 데이터 그대로 출력하겠다는 뜻
- 데이터를 가공하고 싶다면 v → v + 1 람다식 사용하면 된다.

### @FunctionalInterface

- 자바1.8부터 지원

### 익명 이너클래스 생성 시 자바가 클래스 내에서 한번 더 컴파일


### 람다식 적용 생성 및 출력 간소화

```java
Arrays.asList(new Customer("id1", "pw1"), 
					  new Customer("id2", "pw2"),
					  new Customer("id3", "pw3")).forEach(System.out::println);
```

### 익명이너 클래스를 이용

```java
//연산식 지원하는 interface 선언
//기능 없이 메소드명 제시(스펙)
interface Calc{
	//public abstact int calc(int v1, int v2);
	public int calc(int v1, int v2);	
}

//람다식용 설정
@FunctionalInterface
interface Calc2{
	public int calc(int v1, int v2);	
}

class CalcSub implements Calc{
	public int calc(int v1, int v2) {
		return v1+v2;
	}
}
```

### 익명 클래스를 이용한 메서드 재정의

```java
		Calc c = new CalcSub();//다형성 형식으로 객체 생성
		
		//step02 - 익명이너클래스 직접 코드로 개발 기술 학습
		Calc c2 = new Calc() {
			public int calc(int v1, int v2) {
				return v1+v2;
			}
		};
		
		Calc c3 = new Calc() {
			public int calc(int v1, int v2) {
				return v1+v2;
			}
		};
	
```

### 람다식에 적합한 Interface와 다양한 연산식 활용

```java
 		Calc2 cc1 = (v1, v2) -> {return v1+v2;};  {} 리턴 키워드 생략 불가 
		Calc2 cc1 = (v1, v2) -> v1+v2;
		int v = cc1.calc(10, 20);		
		System.out.println(v);
		
		cc1 = (v1, v2) -> v1*v2; // 재정의
		v = cc1.calc(10, 20);		
		System.out.println(v);
		
		//? v1과 v2 더한결과 출력 하고 반환값은 빼기로 반환하기 
		cc1 = (v1, v2) -> {
			System.out.println(v1+v1);
			return v1-v2;
		};
		v = cc1.calc(100, 10);
		System.out.println(v);
```

### Stream 활용

```java
//step01
		List<String> datas1 = Arrays.asList("1", "2", "3", "4", "5", "3");		
		List<String> datas2 = null;
		
		// mapToInt로 parseInt 간소화
		System.out.println(datas1.stream()
							.mapToInt(Integer::parseInt).count());
		
		// {}를 사용하면 return 키워드 필수
		System.out.println(datas1.stream()
				.mapToInt(v -> {return Integer.parseInt(v);})
				.count());
		

		System.out.println(datas1.stream()
				.mapToInt(v -> {
								return Integer.parseInt(v+"10");
						 }).max());
		
		// filter 함수 활용 v는 인자값 변수명은 마음대로 정해도 된다. filter는 조건에 맞는 intStream
		// 반환 후 카운트
		System.out.println(datas1.stream()
				.mapToInt(Integer::parseInt)
				.filter(v -> v==3).count());
		
		// 이번에 String Stream 반환 후 카운트할 수도 있지만 조건 필터링 후 int 스트림으로 변환 카운트
		System.out.println(datas1.stream()
				.filter(v -> v.equals("3"))
				.mapToInt(Integer::parseInt)
				.count());	
```

### Stream 응용

```java
//step02
		//3명의 고객 정보 생성
		//나이가 20 미만 고객들의 나이값 합산하기 + 출력
		ArrayList<Customer> al1 = new ArrayList<>();
		al1.add(new Customer("id1", "pw1", 10, "상암", "vip", "M"));
		al1.add(new Customer("id2", "pw2", 15, "마포", "gold", "F"));
		al1.add(new Customer("id3", "pw3", 50, "상암", "gold", "F"));
		
		System.out.println(al1.stream()
							.filter(c -> (c.getAge() < 20))
							.mapToInt(Customer::getAge)
							.sum());
							
							
```

### Null 처리방식 Optional

```java
//@Test
	public void step05() {
//		String v = null;
		String v = "fisa";
		// Optional.ofNullable 메소드는 Null을 빈객체로 포장, null이 아니면 값을 저장 
		// 값은 get으로 얻을 수 있다. null point를 방지하기에 하위로직 실행을 보장한다.
		Optional<String> opt = Optional.ofNullable(v);
		
		// ifPresent 메소드 : null 이 아닐경우에 람다식을 이용해서 추가적인 작업 가능
		System.out.println("1-----");
		opt.ifPresent(v2 -> System.out.println(v2));
		System.out.println("2-----");
	}
```

### Optional Null이 발생되지 않을 것을 확신할 때

```java
@Test
	public void step06() {
		String v = null;
//		String v = "fisa";
		// Null 발생 시 무조건 exception 발생. 
		Optional<String> opt = Optional.of(v);
		System.out.println("1-----");
//		opt.ifPresent(v2 -> System.out.println(v2));
		System.out.println(opt.get());
		System.out.println("2-----");
	}
```

### 컴파일엔 오류가 없어도 실행 시 Null이 발생되어 에러가 발생하는 경우에도 로직을 계속 진행시키고 싶을 때 Optional 사용

```java
//	@Test
	public void step081() {
	  // 임의로 null 발생
		HashMap<String, Customer> map = null;
		// null이 들어와도 optional로 포장 optional -> hashmap -> customer
		Optional<HashMap<String, Customer>> opt 
		= Optional.ofNullable(map);
		
		// 문법엔 오류가 없어서 정상적으로 컴파일되고 로직이 실행됨.
		System.out.println("1------");
		System.out.println("2-------");
		opt.ifPresent(map2 -> {
			System.out.println(map2.values().stream()
							  .filter(customer -> customer.getAge() < 20)
							  .mapToInt(Customer::getAge)
							  .sum()				
			);
		});
		// optinal 처리한 덕분에 아래 로직까지 실행
		System.out.println("3-------");
		
	}
```
