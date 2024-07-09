# [FISA 클라우드 엔지니어링 2일차]

### FISA 클라우드 엔지니어링 2일차 정리

- Virtualbox로 게스트 운영체제 만들기
    - ubuntu 22.0.04 설치(ssh 통신 허용 설정 필수)
    - 만든 VM에 포트 포워딩 네트워크 설정 Local ip(127.0.0.1) 임의의 ip(10.xx.xx.xx) 지정 port 22번으로 통신
    - MobaXterm으로 127.0.0.1 port 22 로 ssh 접속
- 리눅스 명령어 실습
    - 우분투에 java 설치
        - java 명령어 실행 시 jre로 설치하라고 안내 (컴파일을 못하는 인터프리터 실행기)
        - javac 명령어 실행 시 jdk로 설치하라고 안내 (컴파일과 인터프리터 동시에 실행 가능함)
        - apt install 명령어를 통해 안내한 jdk로 자바 설치 (17권장)
    - 기타 소프트웨어 설치
        - tree
        - ifconfig 등등
        - apt install을 통해 설치
- 메서드와 함수의 차이
    - 메서드는 클래스 내에서 개발된 기능
    - 함수는 클래스와 무관하게 개발된 기능
- Java 정적 메서드, 변수 응용
    
    ```java
    import java.util.Stack;
    
    public class MyClass {
        static int staticVar = 0;
        int instanceVar = 0;
    
        public MyClass() {
            staticVar++;
            instanceVar++;
        }
    
        public static void main(String[] args) {
            Stack<Integer> stack = new Stack<>();
    
            MyClass obj1 = new MyClass();
            MyClass obj2 = new MyClass();
            stack.push(MyClass.staticVar);
    
            System.out.println(obj1.instanceVar);  // ? 1
            obj1.instanceVar += 5; // 6
            stack.push(obj1.instanceVar);
    
            System.out.println(obj2.instanceVar);  // ? 1
            stack.push(obj2.instanceVar);
    
            MyClass obj3 = new MyClass();
            obj3.staticVar += 2;
            System.out.println(obj3.staticVar);  // ? 5
            stack.push(obj3.staticVar);
    
            // stack 2, 6, 1, 5
            for (Integer i : stack) {
                System.out.print(i + " ");
            }
            
        }
    }
    ```
    
    - 위 코드는 인스턴스가 생성될 때 static 변수가 +1 되게 만들고, 나머지 인스턴스 내의 변수는 독립적으로 연산이 됨.
    - 정적 변수는 전역적으로 사용되고, 인스턴스로 생성된 변수는 해당 인스턴스에서만 사용가능하다는 것을 이해하기 위한 코드
    - static 변수와 메서드는 클래스가 컴파일되는 동시에 Method Area에 저장
    - 프로그램이 끝날 때까지 전역변수 및 메서드로 활용가능
        - 다만 동시성 이슈에 대해 주의하여 사용해야함
    - 클래스에서 메서드를 설계할 때 NON-STATIC, STATIC 둘 중 어느것으로 만들지 STATIC의 특징을 이해하고, 기준을 정해야한다.
    - String 클래스에서 length, charAt() 예시
        - lenth(), charAt() 처럼 특정 대상이 있어야 하는 메서드는 non-static으로 만든다.
- STS에서 롬복 설치하기
    - Lombok library 구글에서 다운로드
    - java -jar 명령어로 jar파일 실행 후 STS 실행 파일 지정해주기
    - STS 폴더 하위에 imi 파일에 jar의 명확한 위치가 기재됨
    - pom.xml에 dependency 의존성 추가
- Lombok은 왜 사용하는거지?
    - 흔히 자바에서 자주 사용하는 DTO, DAO, Domain 등 중복적인 코드가 자주 발생
        - Constructor, Getter, Setter를 간단하게 어노테이션으로 사용 가능
        - @NoArgsConstuctor, AllArgsConstructor, Getter, Setter