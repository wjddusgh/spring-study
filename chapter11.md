- 스프링 MVC 웹 개발은 결국 **컨트롤러, 뷰 코드 구현** 을 의미

# 2. 요청 매핑 애노테이션을 이용한 경로 매핑

필요한 기능
- 특정 요청 URL을 처리할 코드 (Controller)
- 처리 결과를 HTML과 같은 형식으로 응답하는 코드 (View)

Controller
- 같은 경로의 메소드는 묶는게 좋다
- ex. /register/step1, /register/step2는 같은 register 컨트롤러로 묶자

# 3. GET과 POST 구분: @GetMapping, @PostMapping

- @RequestMapping은 모든 방식(get,post,put,delete) 다 받는다
- 어느 요청만 처리하고싶으면 @GetMapping 등으로 사용
- 방식을 구분하기에 같은 경로로 다른방식 매핑하기 가능
- POST로만 매핑된 경로에 URL로 접근시 405 에러(Method Not Allowed)

# 4. 요청 파라미터 접근

- 클라이언트로부터 POST 요청이 오면 그값을 받는법
1. HttpServletRequest의 getParameter() 사용 (매개변수는 name 태그)
2. @RequestParam 애노테이션 사용
```java
@PostMapping("/register/step2")
	public String handle(
			@RequestParam(value = "agree", defaultValue = "false") Boolean agree,
			Model model) {
	}
```
- @RequestParam 속성

|속성|타입|설명|
|------|---|---|
|value|String|HTTP 요청 파라미터의 이름 지정|
|required|boolean|필수여부 지정, true인데 파라미터에 값이 없으면 익셉션 발생|
|defaultValue|String|요청 파라미터에 값이 없으면 이 값으로 지정, 기본값은 없다|

- 요청 파라미터는 String 타입이지만 @RequestParam을 사용한 변수 타입에 맞게 변경해줌

# 5. 리다이렉트 처리
```java
@GetMapping("/register/step2")
	public String handle() {
		return "redirect:/register/step1";
	}
```
리다이렉트대신 "register/step1" 리턴하면 안되나?
- 쓰는 이유가 있다
1. 기존 페이지 주소가 변경된 경우 새로운 페이지로 옮겨주려고
2. 잘못 적은주소면 되돌려주려고
- 리다이렉트는 기존 상태를 잃기 때문에 포워드를 사용하기도 한다고 한다

# 6. 커맨드 객체를 이용해서 요청 파라미터 사용하기

- @RequestParam, getParameter()은 파라미터 갯수가 증가할때마다 추가돼서 복잡해진다
- 커맨드 객체를 사용하면 간단해진다
- 커맨드 객체는 다른건 아니고 세터메서드를 포함하는 객체

```java
@PostMapping("/register/step3")
	public String handleStep3(RegisterRequest regReq) { // 커맨드객체로 받아옴
		try {
			memberRegisterService.regist(regReq);   
			return "register/step3";
		} catch (DuplicateMemberException ex) {
			return "register/step2";
		}
	}
```

# 7. 뷰 JSP 코드에서 커맨드 객체 사용하기

- jsp코드에서 커맨드객체.필드이름 으로 접근 가능 ex. ${registerRequest.name} (첫글자가 소문자로 됨)

# 8. @ModelAttribute 애노테이션으로 커맨드 객체 속성 이름 변경

- JSP에서 쓰일 커맨드객체 이름 바꾸려면 @ModelAttribute(사용할이름)을 붙이면 됨
```java
@PostMapping("/register/step3")
	public String handleStep3(@ModelAttribute("yeonho") RegisterRequest regReq) {
	}
```
# 9. 커맨드 객체와 스프링 폼 연동

- 잘못 입력해서 다시 입력해야할 때 이전 입력값이 다 날라가면 화난다
- 인풋 태그에 value = ${입력값 ex. yeonho.email} 넣으면 남아있는다
- 스프링 커스텀 태그도 있지만 넘어간다

# 10. 컨트롤러 구현 없는 경로 매핑

컨트롤러에서 내용이 필요없이 매핑된 경로를 View와 연결시키는 경우 config단계에서 해줄수도 있다
- WebMvcConfigurer 사용하자
```java
  @Override
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/main").setViewName("main");    // "/main"경로는 "main" 뷰로 매핑 완료
	}
```

# 11. 주요 에러 발생 상황

## 11.1 요청 매핑 애노테이션과 관련된 주요 익셉션
404(Not Found) 에러 발생시 확인할 사항
- 요청 경로가 올바른지
- 컨트롤러에 설정한 경로가 올바른지
- 컨트롤러 클래스를 빈으로 등록했는지
- 컨트롤러 클래스에 @Controller 애노테이션을 적용했는지

## 11.2 @RequestParam이나 커맨드 객체와 관련된 주요 익셉션
- 필수로 필요한 요청파라미터가 없다면 400(Bad Request)에러 발생
- 파라미터 타입에 맞게 변환할 때 변환할 수 없는 값이 들어오면 400 에러 발생

# 12. 커맨드 객체 : 중첩 . 콜렉션 프로퍼티
