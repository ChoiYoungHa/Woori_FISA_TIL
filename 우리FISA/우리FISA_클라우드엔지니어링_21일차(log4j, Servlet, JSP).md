# FISA 클라우드엔지니어링 21일차 (log4j, Servlet, JSP)

### Log4J 및 Servlet

1. log4j 활용연습 Servlet
    1. log file 자동 생성
        1. 로그 레벨 수정하면서 매핑된 메소드 실행 여부 확인
    2. 메소드로 출력제어
    3. 시스템 시간 변경하면서 백업 로그 파일 생성확인

### Log4j 설정파일 해석

```sql
log4j.rootCategory=debug, file, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%m%n
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.File=C:\\01.lab\\00.log\\weblog.log
log4j.appender.file.DatePattern='.'yyyy-MM-dd
log4j.appender.file.Append=true
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n

#log4j.logger.org.hibernate=OFF
#log4j.loggerjavax.persistence=OFF
```

- %d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
- **2024-08-05 10:49:43 DEBUG FirstServlet:21 - firstServlet info() *****
- %d{yyyy-MM-dd HH:mm:ss} 시간, %-5p 로그레벨을 표시할 글자개수, %c{1} 클래스명, %L - 대시라인, %m%n firstServlet info() *** 메소드명과 개행
1. servlet api 개발 기술
    1. java기반의 웹 원조 스펙
    2. http 통신이 가능한 유일한 자바 파일
    3. jsp 근본
        1. jsp 파일은 html css javascript / java tag 로 구성된 웹 브라우저 출력 스펙
        2. jsp 요청시 servlet으로 자동변환
    4. http 요청 및 처리방식
        1. get 방식 - 데이터 검색
        2. post 방식 - 저장 및 수정
        3. delete 방식
        4. put 방식
    5. web page 이동 api
        1. 모든 웹 개발 스펙 공통방식
        2. html 코드로 처리방식
        3. servlet api 처리방식
            1. forwarding
            2. redirect
    6. client 의 지속성 상태 관리 기술
        1. 웹 기본 스펙 : 무상태 연결 유지
        2. 로그인 ~ 로그아웃 지속성 상태 유지는 코드개발
        3. 모든 웹 개발 스펙 공통 방식
        4. javax.servlet.http.HttpSession
            1. server 메모리에 상태값 저장
        5. javax.servlet.http.Cookie
            1. client 시스템에 상태값 저장
2. web page 이동 방식
3. client 상태 유지를 위한 세션과 쿠키
4. UI 템플릿 반영

### Servlet

1. servlet 객체 수는 client 수와 무관하게 한개로 관리
    1. 생성시점
    2. 예시
        1. 로그인 요청을 받는 servlet으로 가정
        2. client가 입력하는 id/pw 다 다름
        3. 개별 메소드로 처리 및 실행
            1. 맴버 변수로 id/pw 값 받을지? 로컬 변수로 받을지?
    3. 객체는 하나, 메소드는 다수 실행되는 구조로 개발
    4. 객체 생성 및 메소드 호출은 개발자가 코드로 하지 않음
        1. tomcat (web server, web container, servlet engine) 호출

1. Servlet 주요 API

client 접속 수 : Servlet 객체수 : HttpServletRequest, HttpServletResponse

100명 : 1 : 100 : 100

- 객체 생성시점
    - Servlet  : 최초 client가 호출시 단 한번만 생성
    - HttpServletRequest : 클라이언트 접속할 때마다
    - HttpServletResponse : 응답마다
1. HttpServlet
    1. http 통신 관련 로직을 구현해서 제공
    2. 사용자 정의 상속받은 하위 클래스로 필요 방식에 관련된 메소드만 재정의하여 http 통신 가능
2. HttpServletRequest
    1. client 접속 정보 보유하게 되는 객체
    2. 코드로 생성하지 않음(web server 내부의 로직이 처리, web server 필수)
    3. client 요청 시 자동생성
    4. client ip/입력 데이터 등 다 보유
    5. 한글 데이터 깨짐 현상은 고려
    6. 데이터 하나만 착출하는 메소드
        1. String getParameter(key)
    7. 취미 등 처럼 다수의 데이터를 한번에 착출하는 메소드
        1. String[] getParameterValues(key)
3. HttpServletResponse
    1. client의 응답 정보를 보유하게 되는 객체
    2. 코드 생성하지 않음
    3. 접속해서 생성하게 한 client에게만 응답
    4. 한글 데이터 깨짐현상 다 고려
    5. setContentType(”포맷;인코딩”)
        1. 포멧 - mine type
        2. text/html : text 기반의 html 포멧으로 응답
        3. application/json : json 포맷응답
    6. 인코딩
        1. utf-8 : 다국어 지원 설정
        2. euc-kr : 한글,영어, 숫자, 특수기호
        3. ms949 : window 기반의 한글표기

### forward 와 Redirect의 차이

```java
package step02.pagemove;
import java.io.IOException;
import java.io.PrintWriter;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
public class One extends HttpServlet {
	
	//get방식, post방식 모두다 처리 하는 특화된 메소드
	protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		System.out.println("One의 service()");		
		String id = request.getParameter("id");
		System.out.println("id : " + id);
		
		//forward
		//http://localhost/step11_web/pageMove.html
		//http://localhost/step11_web/one?id=fisa
		//pageMove.html -> One -> Two 까지 페이지 이동
		/* 출력결과
		One의 service()
		id : fisa
		Two의 service()
		id : fisa
		 */
//		request.getRequestDispatcher("two").forward(request, response);
		
		//redirect
		//http://localhost/step11_web/pageMove.html
		//http://localhost/step11_web/two
		//pageMove.html -> One -> Two 까지 페이지 이동
		/* 출력결과
		One의 service()
		id : fisa
		Two의 service()
		id : null
		 */
		response.sendRedirect("two");
		
	}
}
```

- forward는 사용자에게 url 변경을 보여주고 싶지 않고 데이터를 다른 페이지로 넘기고 싶을 때 사용
- redirect는 사용자에게 url 변경을 확실히 알려 페이지 변경을 알려주고 url을 변경하여 form의 중복제출을 방지하기 위해서 사용
- redirect는 새로운 서블렛 객체를 만든다.
- forward는 두개의 서블릿이 객체를 공유

### JSP를 수정했는데 변경사항이 바뀌지 않을 때

- jsp는 서블릿이라 여기에 저장된다.
- .metadata 하위에 서블릿이 저장된다.
- 예)C:\01.lab\01.java\.metadata\.plugins\org.eclipse.wst.server.core\tmp0\work\Catalina\localhost\step13_web\org\apache\jsp

### JSP 내장객체

| EL 내장 객체 | 속성 | syntax |
| --- | --- | --- |
| pageScope | page scope에 바인딩된 속성 | ${pageScope...} |
| requestScope | request scope에 바인딩된 속성 | ${requestScope...} |
| sessionScope | session scope에 바인딩된 속성 | ${sessionScope...} |
| applicationScope | application scope에 바인딩된 속성 | ${applicationScope...} |
| param | getParameter() 와 동일 | ${param...} |
| paramValues | getParameterValues()와 동일 | ${paramValues...} |
| cookie | Cookie 객체 | ${cookie...} |
| initParam | getInitParameter() 와동일 | ${initParam...} |

### JSP 예제코드

```sql
<%@page import="org.hibernate.internal.build.AllowSysOut"%>
<%@ page contentType="text/html; charset=utf-8"%>

<%

	String id = request.getParameter("id");
	System.out.println(id);
	
	
	String heart = (String) request.getAttribute("heart");
	System.out.println(heart);
%>

<%-- jsp 주석 --%>
<!-- html 주석 -->

<%-- id값 출력 --%>
<%-- id값 출력 --%>

${param.id}<br>
${requestScope.heart}
${heart}
```

- ${} 표시는 jsp에서 출력 사인
- param 은 request.getParameter 줄임 내장 메서드
- requestScope는 request.getAttribute 줄임 내장 메서드