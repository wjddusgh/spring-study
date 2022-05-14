# 1. 프로젝트 준비
# 2. <spring:message> 태그로 메시지 출력하기

- 사용자 화면에 보일 문자열 하드코딩시 변경에 있어 모두 바꿔줘야해서 유지보수에 불편
- 다국어 지원에도 하드코딩 시 jsp를 추가로 만들어야함
- 문자열을 별도 파일에 작성 후 jsp 코드에서 사용하려면
  - 문자열을 담은 메시지 파일을 작성 (.properties 로 resource에 저장)
  - 메시지 파일에서 값을 읽어오는 MessageSource 빈 설정
  - JSP 코드에서 <spring:message> 태그를 사용해서 메시지 출력
- config에서 MessageSource 빈 등록하는 예시
```java
@Bean
public MessageSource ms() {
  ResourceBundleMessageSource rbms = new ResourceBundleMessageSource();
  rbms.setBasenames("message.label", "message.error"); // 매개변수는 가변인자
  rbms.setDefaultEncoding("UTF-8");
  return rbms;
}
```
- 다국어 지원시 locale 문자 2글자를 붙임 (ex. ko, en, jp 등등)

## 2.1 메시지 처리를 위한 MessageSource와 <spring:message> 태그
- 다국어 지원을 위해 MessageSource 인터페이스의 getMessage() 메서드의 파라미터로 locale이 있다
- 해당 메시지가 없다면 익셉션 발생( 500 Internal Server Error )
- ResourceBundle 이 붙은 클래스는 해당 프로퍼티 파일이 클래스 패스에 위치해야 한다

## 2.2 <spring:message> 태그의 메시지 인자 처리
- {index} 를 통해 getMessage()의 argument 접근
```
register.done = {0}님, 회원가입 완료

args[0] = "정연호"
messageSource.getMessage("register.done", args, Locale.KOREA);
```
- <spring:argument> 태그로도 인자 줄수도 있다

# 3. 커맨드 객체의 값 검증과 에러 메시지 처리

- 비정상 값, 중복 값 입력하면 예외처리를 해줘야한다
- 스프링이 제공하는 방법
  - 커맨드 객체를 검증하고 결과를 에러코드로 저장
  - JSP에서 에러 코드로부터 메시지를 출력
 
## 3.1 커맨드 객체 검증과 에러 코드 지정하기
- 값 검사 인터페이스
  - org.springframework.validation.Validator
  - org.springframework.validation.Errors

- Validator의 메서드
  - supports() : Validator가 검증할 수 있는 타입인지 검사
  - validate() : 타겟을 검증하고 결과를 Errors에 담는다
- Errors의 메서드
  - rejectValue() : 프로퍼티 이름과 에러코드를 전달

- ValidationUtils 사용시 간편하게 검증코드 구현 가능
- value가 문제가 아니라 커맨드 객체 자체가 문제일 수도 있다
  - rejectValue() 대신 reject() 사용해 객체 자체를 errors로 전달 (글로벌 에러 라고 부름)
- 웃긴점 Errors 객체를 검증할 객체 앞에 선언하면 익셉션 발생, 이유는 파라미터로 받지않고 왼쪽의 객체를 검증할 객체로 인식하나봄

## 3.2 Errors와 ValidationUtils 클래스의 주요 메서드
- Errors
  - reject() : 파라미터로 에러코드, 디폴트메시지, arguments
  - rejectValue() : 파라미터로 reject()에 검증 객체 이름 추가

- ValidationUtils
  - rejectIfEmpty() : 프로퍼티가 null이거나 빈 문자열("")인 경우 에러
  - rejectIfEmptyOrWhitespace() : 프로퍼티가 null, 빈 문자열, 공백문자(스페이스, 탭 등)인 경우 에러
  - 둘 다 이름으로 기능 유추 가능 

## 3.3 커맨드 객체의 에러 메시지 출력하기
- <form:errors> 태그 사용해서 에러 메시지 출력함
- 에러 코드 찾는 규칙
  - 에러코드 + "." + 커맨드객체이름 + "." + 필드명 이 기본이다
  - ex. errors.rejectValue("email", "required") , 커맨드객체 이름이 "registerRequest" 라면
    - required.registerRequest.email
    - required.email
    - required.String
    - required
    - 순서로 탐색

## 3.4 <form:errors> 태그의 주요 속성
- element : 각 에러메시지 출력할때 사용할 html 태그 (default: span)
- delimiter : 각 에러메시지 구분할때 사용할 html 태그 (default: <br/>)

# 4. 글로벌 범위 Validator와 컨트롤러 범위 Validator
- 스프링 MVC는 모든 컨트롤러에 적용할 수 있는 글로벌 Validator와 단일 컨트롤러에 적용할 Validator 설정방법을 제공한다
- @Valid 애노테이션을 사용한다

## 4.1 글로벌 범위 Validator 설정과 @Valid 애노테이션
- 글로벌 범위 Validator 적용법
  - 설정 클래스에서 WebMvcConfigurer의 getValidator()메서드가 Validator 구현 객체를 리턴하도록 구현
  - 글로벌 범위 Validator가 검증할 커맨드 객체에 @Valid 애노테이션 적용
- @Valid 애노테이션은 Bean Validation API에 포함되어 ㅣㅇㅆ어서 validation-api 모듈을 추가해야 사용가능
- @Valid 애노테이션 사용시 Errors 객체가 없는데 검증 실패 시 400 에러(Bad Request) 발생

## 4.2 @InitBinder 애노테이션을 이용한 컨트롤러 범위 Validator
- @InitBinder 애노테이션 이용시 컨트롤러 범위의 Validator 설정 가능
- 컨트롤러 내부에서 함수 작성
```java
@Controller
public class RegController {
  ... // 커맨드객체에 @Valid 애노테이션 사용해놓음
  
  @InitBinder
  protected void initBinder(WebDataBinder binder) {
    binder.setValidator(new RegisterRequestValidator());
  }
}
```
- @InitBinder 를 붙이면 매핑이되어 요청을 실행할때마다 먼저 실행됨
- WebDataBinder에는 모든 Validator 목록 갖고있고, setValidator()시 다 지우고 인자로 받은 Validator만 남긴다
- addValidator() 사용시 지우지 않고 추가만 하므로 글로벌 Validator 적용 뒤에 컨트롤러 범위 Validator 적용한다

# 5. Bean Validation 을 이용한 값 검증 처리

- Bean Validation 스펙에는 @Valid, @NotNull, @Digits, @Size, @NotBlank, @NotEmpty 등이 정의되어 있다
- 이것들로 Validator 작성 없이 검증을 처리할 수 있다
  - Bean Validation과 관련된 의존을 설정에 추가한다
  - 커맨드 객체에 @NotNull, @Digits 등의 애노테이션을 이용해서 검증 규칙을 설정한다   
  - OptionalValidatorFactoryBean 클래스를 빈으로 등록
