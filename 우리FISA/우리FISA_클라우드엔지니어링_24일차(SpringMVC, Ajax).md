# FISA 클라우드 엔지니어링 24일차(Spring MVC, Axios, JSP)

### Controller

- 컨트롤러는 클라이언트의 요청을 받을 수 있는 리소스를 정의하는 곳
- 스프링은 DispacherServlet을 구현하고 HTTP 통신이 가능하게 만들어준다.

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController // 일반 클래스를 http 통신이 가능한 servelt으로 변환
public class FirstSubController {
	
	//get 요청받고, 데이터도 받는 service 메소드(doget)
	@GetMapping("fisadata")
	public String m3(@RequestParam("data") String data) {
		System.out.println("m3() : " + data);
		return data;
	}
	
	
	@PostMapping("fisadata")
	public People m4(@RequestBody People data) {
		System.out.println("m4() : " + data);
		return data;
	}
	

	//post 요청받고, 데이터도 받는 service 메소드(dopost) - People 데이터
	@GetMapping("fisa")
	public String m1() {
		return "spring get"; // 문자열로 응답
	}
	
	
	@PostMapping("fisa")
	public People m2() {
		return new People("연아",10);
	}
}
```

- RestController는 Controller와 ResponseBody를 합친 형태로 응답값을 body에 그려준다.
- RestController는 Json 또는 xml을 반환할 때 주로 사용

### Axios

```java
<body>
	<button id=dataViewPeople>Axios get data 정보</button>
	<button id=dataViewPeople2>Axios Post People 정보</button>
	<div id="dataView"></div>

	<script type="text/javascript">
		
		// GET 요청
		dataViewPeople.addEventListener("click",function(){
			axios.get('/ce/fisadata?data=get 재석')
			  .then(function (response) {
			    // 성공 핸들링
			    console.log(response);
			    dataView.innerText = response.data;
			  })
			  .catch(function (error) {
			    // 에러 핸들링
			    console.log(error);
			  })
			  .finally(function () {
			    // 항상 실행되는 영역
			  });
		});
		
		// Post 요청
		dataViewPeople2.addEventListener("click", function(){
			console.log("click");
			axios.post('/ce/fisadata', {
			    name: 'post 홍철',
			    age: '10'
			  })
			  .then(function (response) {
				  console.log(response);
				  dataView.innerText = response.data.name;
			  })
			  .catch(function (error) {
			    console.log(error);
			  });
		});
		
		
		// ajax 문법 GET 요청
		/* dataViewPeople2.addEventListener("click", function(){
			const xhttp = new XMLHttpRequest(); // 비동기 요청 JS 객체
			xhttp.onreadystatechange = function() { // readyState 
				if (this.readyState == 4 && this.status == 200) { // 4: 응답완료
					let resData = this.responseText; // server 응답 데이터 가로채는 속성
					resData = JSON.parse(resData);
					console.log(resData);
					dataView.innerText = resData;
				}
			};
			xhttp.open("POST", "/ce/fisadata");
			xhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
			xhttp.send("name=연아&age=10");
		}); */
		
		
		// ajax 문법 Post 요청
		/* dataViewPeople.addEventListener("click", function(){
			const xhttp = new XMLHttpRequest(); // 비동기 요청 JS 객체
			xhttp.onreadystatechange = function() { // readyState 
				if (this.readyState == 4 && this.status == 200) { // 4: 응답완료
					let resData = this.responseText; // server 응답 데이터 가로채는 속성
					resData = JSON.parse(resData);
					console.log(resData);
					dataView.innerText = resData.name;
				}
			};
			xhttp.open("POST", "/ce/fisa");
			xhttp.send();
		}); */
		
</script>
	<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
</body>
```

- axios를 통한 비동기 요청 코드 cdn 사용

### SpringBoot에서 JSP를 사용하고 싶다면?

1. JSP 개발 위치
    1. 자동 생성된 폴더(spring.io)에서 제시
    2. resource/static/
    3. 순수 html/css/javascript./이미지/영상…
    4. resource/templates
        1. spring 자체의 tag 타임리프
            1. tomcat에서 타임리프 코드 실행
            2. html 등은 브라우저에게 문자열로 응답
        2. *.html
2. 직접 jsp 저장 위치 생성 및 위치를 설정 파일에 등록
    1. servlet&jsp
        1. WEB-INF 하위엔 절대 개발불가
            1. 서버만의 설정경로
        2. 컴파일된 byte code & web.xml & lib
    2. Spring Framework
        1. 보안강화
        2. WEB-INF 폴더 하위에 jsp 개발허용
            1. 위치는 spring 설정 파일에 등록
        3. js 기반의 프레임워크와 타임리프 부각되는 시점 도래
            1. jsp 사용자제
            2. spring boot는 jsp 미고려
                1. 필요시 해결점 - 사용자들이 환경 구축
    3. Spring Boot
        1. src/main/java
        2. src/main/webapp 폴더 생성
            1. WEB-INF
                - /jsp 개발
                - 반드시 jsp 위치를 설정 파일에 등록

**application.properties 설정**

```java
spring.mvc.view.prefix=/WEB-INF/
spring.mvc.view.suffix=.jsp
```

### 레거시 MVC

```java
@Controller
public class SubController {

	@RequestMapping(path="/fisa", method = RequestMethod.GET)
	public ModelAndView m1() {
		ModelAndView mv = new ModelAndView();
		
		// main.jsp에서 controller에서 저장한 데이터 출력
		mv.addObject("msg","서버에서 저장한 데이터"); // reqeust.setAttribute()
		mv.setViewName("main");
		// request.getRequestDispacher("main.jsp").forward(request, response);
		return mv;
	}
}
```