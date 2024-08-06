# FISA 클라우드 엔지니어링 22일차(JSP, JSTL)

### Redirect 동작방식

- 리다이렉트 시 무조건 Get으로 요청
- 때문에 form 중복제출 방지

### Forward 동작방식

- 서블릿끼리 request, response 객체 공유

### Redirect에서는 상대경로가 먹히고, Forward에서는 왜 상대경로가 먹히지 않을까?

- Redirect는 클라이언트에게 재요청을하기 때문에 서블릿 컨텍스트가 먹히지 않음.
- Forwrad는 서블렛끼리 공유하는 것이기 때문에 url을 전달해줄때 컨텍스트가 사라진다.
- 때문에 forward는 절대경로로 명시하여 컨텍스트 경로를 지정해주어야함.
- 리다이렉트는 사용자가 재접속하기 때문에 서블릿 컨텍스트가 정상 생성되는 것
- **request.getRequestDispatcher("view/welcome").forward(request, response);**
    - forward로 이동하면서 컨텍스트 유실
    - 서블릿의 매핑된 리소스로 이동하게 됨.

### 서블렛이 디렉터리를 확인하는 방법

- 서블렛에 등록되지 않은 리소스를 접속하게 되면 현재 html의 위치로 이동 시킴
- form 액션으로 접속하던 url로 접속하던 매핑된 리소스로 접속하게끔 만들어준다.

### Cookie

- 쿠키는 웹 서버가 사용자의 브라우저에 저장하는 데이터 조각
- 주로 사용자 식별, 세션 유지, 사용자 설정 저장 등의 목적으로 사용

```java
 		Cookie msg1 = new Cookie("msg1", "hi");
		Cookie msg2 = new Cookie("msg2", "fisa");

		msg1.setMaxAge(60 * 60 * 24); // 초단위 설정
		msg2.setMaxAge(60 * 60 * 24 * 365); // 초단위 설정

		response.addCookie(msg1);
		response.addCookie(msg2);
```

- 쿠키 세팅

```java
  // 해당 도메인의 쿠키 정보만 획득
	Cookie[] all = request.getCookies();
	for(Cookie c : all) {
			if (c.getName().equals("msg1")) {
				out.println(c.getValue() + "<br>");
			}
	}
		
		//로그아웃 클릭시에 쿠키 정보 삭제 후 login.html로 이동되는 servlet으로 이동
	out.println("<a href='/step13_web/logout'>");
	out.println("logout</a>");
```

- 쿠키정보 사용

```java
    Cookie[] all = request.getCookies();
		
		for(Cookie c : all) {
			if (c.getName().equals("msg1")) {
				c.setValue("");
				response.addCookie(c);
			}
		}
		
		response.sendRedirect("html/login.html");
```

- 로그아웃 시 쿠키 제거

### Session

- 사용자가 로그인 시 서버 메모리에 사용자의 세션정보를 보관하고 로그인 상태일 때 세션정보를 언제든 사용 가능
- 사용자 로그인 정보를 기억하고, 데이터 저장 용도로도 사용한다.

```java
    PrintWriter out = response.getWriter();
		HttpSession session = request.getSession(); // client만의 고유객체
		System.out.println(session.getId()); // 컨테이너가 구분하기 위해 부여한 id
		// 데이터 저장
		session.setAttribute("key1", "문자 데이터");
		session.setAttribute("key2", 10); // new Integer(10)으로 변환
		
		out.println(session.getAttribute("key1"));
```

- 세션정보 얻기

```java
    HttpSession session = request.getSession();
		session.invalidate();
		session = null;
```

- 세션정보 초기화

### JSP

- Servelt으로 변환되어 실행
    - 최초 클라이언트가 jsp를 호출했을 때
    - servlet 변환 → 컴파일 → 바이트 코드 로딩 → servlet 객체 생성 → 서비스 메소드 실행
- 기능
    - java 데이터를 브라우저 화면에 출력을 담당하는 스펙
- 개발방법
    - html/css/javascript
    - jsp tag
        - 1번과 2번 가급적 최소화 실행
            - jsp 스펙 생성시부터 제시하는 표준 tag
            - 1번,2번은 java 코드가 많이 혼용되어 사용(현추세에 맞지 않음)
        - 3,4 사용권장
            - java 코드 최소화, tag 위주개발 간결하게 권장
        - 스크립트 tag 5가지
        - jsp action tag
        - EL 태그
        - JSTL
            - 별도의 라이브러리 세팅
            - apache 사이트에서 제공
            - 단, 수업시간에 버전 호환 이슈로 tomcat에서 제공하는 library 사용예정
            - 추후 maven으로 설정예정
- tag 상세 설명
    - 스크립팅 tag 5가지
        - <%@ %> : jsp 선언구 사용필수
            - **<%@ page %> : page 지시자, jsp 설정 정보**
            - **<%@ taglib %> : 외부 tag 라이브러리 가지고 올 때 사용**
        - **<%— —%> : 주석, client에게 전송되지 않는 서버단만의 주석**
        - <% %> : 순수 자바코드 영역, service 메소드 구현부로 자동반영 (가급적 최소로)
        - <%! %> : 선언자, 맴버 변수, 메소드 구현 tag
        - <%= %> : 브라우저 데이터 출력
            - EL tag로 대체 권장
- jsp action
    - - <jsp:..> action tag
    - 주요 tag
        - forward 이동 tag
        - <jsp:forward page=”이동page”>
- EL
    - ${}
    - jsp에 작성하는 tag는 브라우저에 출력
- JSTL
    - 반복 tag, 조건 tag
    - java 코드를 최소화해주는 tag
    - library 셋팅 필수, jsp에서 사용선언

### EL 태그로 자바 객체탐색하여 출력

```sql

	<%	//1
		//dto들을 ArrayList에 저장해서 세션에 또 저장
		//dto들 HashMap에 저장해서 세션에 또 저장
		People p1 = new People("연아", 30);
		People p2 = new People("재석", 50);
		People p3 = new People("은갈치", 40);
		
		ArrayList<People> al = new ArrayList<>();
		al.add(p1);
		al.add(p2);
		al.add(p3);
		
		request.setAttribute("rd3", al);
%>
<br>
4. ${requestScope.rd3[1].getName()}<br> // 재석 출력

<%	//2
		HashMap<String, People> hm = new HashMap<>();
		hm.put("p1", p1);
		hm.put("p2", p2);
		hm.put("p3", p3);
		request.setAttribute("rd4", hm);
%>

5. ${requestScope.rd4.get("p1").getName()} // 연아 출력

<%	//3
		HashMap<String, ArrayList<People>> hm2 = new HashMap<>();
		hm2.put("al", al);
		request.setAttribute("rd5", hm2);		
%>
<br>
6. ${requestScope.rd5.get("al")[2].getName()} // 은갈치 출력

```

### JSTL 태그 사용

- **<%@ taglib prefix=*"c"* uri=*"http://java.sun.com/jsp/jstl/core"* %>**
- c를 alias로 사용

```sql
**<%@ taglib prefix=*"c"* uri=*"http://java.sun.com/jsp/jstl/core"* %>**

<%-- jstl tag 조건식 활용 --%>
	<c:if test="${'a' == 'a'}">
		2. 조건이 true인 경우에만 출력<br>
	</c:if>
	
	<c:if test="${'a' != 'a'}">
		3. 조건이 false인 경우에만 출력<br>
	</c:if>
	
	<%-- 데이터가 null일 때만 ture --%>
	<c:if test="${empty requestScope.key1}">
		4. 비교가 true인 경우에만 출력<br>
	</c:if>
	
	<%-- 5번 조건은 null이 아닐 경우에만 ture --%>
	<c:if test="${not empty requestScope.key1}">
		5. 비교가 true인 경우에만 출력<br>
	</c:if>
```

### JSTL 다중조건식

```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%
	request.setAttribute("rd1", "fisa");
	request.setAttribute("rd2", "클엔");
%>
	<h2>jstl 조건식</h2>
	<br><hr><br>
	1. 단일조건식 <br>
	<c:if test="${requestScope.rd1 == 'fisa'}">
		rd1값이 fisa인 경우에만 출력<br>
	</c:if>
	
	<br><hr><br>
	2. 다중 조건식 : if~else if~else와 흡사한 문장<br>
	<c:choose>
		<c:when test="${2 == 3}">
			2-1 ture? <br>
		</c:when>
		<c:when test="${2 == 2}">
			2-2 true? <br>
		</c:when>
		<c:otherwise>
			2-3. ??? <br>
		</c:otherwise>
	</c:choose>
	<br><hr><br>
</body>
</html>
```

### JSTL forEach

```java
<%@page import="java.util.ArrayList"%>
<%@page import="java.util.HashMap"%>
<%@page import="model.domain.People"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
 <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
 
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<%	//1
		//dto들을 ArrayList에 저장해서 세션에 또 저장
		//dto들 HashMap에 저장해서 세션에 또 저장
		People p1 = new People("연아", 30);
		People p2 = new People("재석", 50);
		People p3 = new People("은갈치", 40);
		
		ArrayList<People> al = new ArrayList<>();
		al.add(p1);
		al.add(p2);
		al.add(p3);
		
		request.setAttribute("rd3", al);
%>
<c:forEach items="${requestScope.rd3}" var="p">
	${p.name}<br>
</c:forEach>
</body>
</html>
```

- **<c:forEach items="${requestScope.rd3}" var="p">**
- for iter문이랑 같은 역할