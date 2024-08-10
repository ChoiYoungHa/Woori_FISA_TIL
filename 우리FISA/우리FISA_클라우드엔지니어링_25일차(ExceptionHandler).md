# FISA 클라우드 엔지니어링 25일차(ExceptionHandler, JDBC Connection, Session)

### Controller ExceptionHandler

```java
@ExceptionHandler
	public String exceptionProcess(SQLException e, Model message) {
		System.out.println("*************");
		e.printStackTrace();
		message.addAttribute("errorMsg", e.getMessage());
		return "forward:/error.jsp";
	}
```

- ExceptionHandler 는 모든 곳에 try catch를 사용하지 않아도 스프링 컨테이너에서 에러를 핸들링 해줄 수 있다.
- ExceptionHandler를 사용한 덕분에 error 페이지로 이동한다.

### Spring JDBC Connection

```java
package com.ce.fisa;

import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;
public class DataSourceManager {
	
	private static HikariDataSource hs;

	
	static {
		HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:oracle:thin:@localhost:1521:xe");
        config.setUsername("scott");
        config.setPassword("tiger");
        config.setMaximumPoolSize(10);

        hs = new HikariDataSource(config);
	}

	public static Connection getConnection() throws SQLException {
		return hs.getConnection();
	}
	
	
	public static void close(Connection conn, Statement pstmt, ResultSet rset) {
		try {
			if(rset != null) {
				rset.close();
				rset = null;
			}
			if(pstmt != null) {
				pstmt.close();
				pstmt = null;
			}
			if(conn != null) {
				conn.close();
				conn = null;
			}
		}catch(SQLException e) {
			e.printStackTrace();
		}
	}
	
	public static void close(Connection conn, Statement pstmt) {
		try {
			if(pstmt != null) {
				pstmt.close();
				pstmt = null;
			}
			if(conn != null) {
				conn.close();
				conn = null;
			}
		}catch(SQLException e) {
			e.printStackTrace();
		}
	}
	
}

```

- 임의의 DataSourceManager를 만들어서 jdbc connection과 close를 편리하게 관리한다.

### HikariCP vs JDBC DriverManagerDataSource

- HikariCP는 설정은 복잡하지만 안정성과 성능이 중요한 환경에서 사용된다.
- DriverManagerDataSource는 spring에서 제공하는 라이브러리이며 성능에는 한계가 있어 프로젝트가 간단하거나 테스트용도로 사용하기 적합하다.

### Session and Cookie

1. 세션과 쿠키를 Spring API 로 어떻게 활용하는지에 대한 학습
2. 상태유지 기술
    1. 쿠키
        - 상태유지 값을 client 브라우저
        - 문자열만 저장
        - jsp 에서 내장 객체 아님
    2. 세션
        - 서버 메모리
        - 저장 데이터 타입인 객체 타입 모두 허용
        - jsp의 내장 객체
3. 주요 API
    1. @Controller
        - 동기방식 controller 개발시 주로 사용
    2. @GetMapping
        - get방식 http 처리 메소드
    3. @SessionAttributes
        - 세션에서 사용할 key값들 등록시 필수
        - 위치 : class 선언구
    4. @ModelAttribute
        - 세션에 저장된 key로 데이터 획득시 주로 메소드 parameter에서 사용
    5. SessionStatus
    : HttpSession이 아닌 @SessionAttributes와 연관
    : 세션 삭제시 메소드 parameter로 선언
        - setComplete() : 세션 소멸
    6. Model
        - addAttribute() - 데이터 저장
        - getAttribute() - 데이터 반환
        : @ModelAttribute로 Model에 저장했던 데이터 획득(반환) 가능
        - spring api의 카멜레온
        @SessionAttributes에 등록된 key 사용시 HttpSession
        non-@SessionAttributes에 등록시 HttpServletRequest