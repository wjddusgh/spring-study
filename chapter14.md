# 1. 프로젝트 준비

# 2. 날짜를 이용한 회원 검색 기능

- `REGDATE` 를 타임스탬프로 사용
- SQL `BETWEEN ? AND ?` 을 통해 기능 구현

# 3. 커맨드 객체 Date 타입 프로퍼티 변환 처리 : @DateTimeFormat

- `text` 로 입력받은 문자열을 Date 타입으로 변환해야 한다
- 객체 프로퍼티에 `@DateTimeFormat()` 애노테이션 활용하면 가능
```java
public calss ListCommand {
  
  @DateTimeFormat(pattern = "yyyyMMddHH")   // 2022052214 -> 2022년 5월 22일 14시
  private LocalDateTime time; 
  
  ...
    
}
```

## 3.1 변환 에러 처리
- `@DateTimeFormat` 형식에 안맞는 입력은 400 에러 발생
- 컨트롤러 메서드에 `Errors` 타입 파라미터 받아서 `hasErrors()` 를 통해 에러 처리 가능
- 메시지 프로퍼티를 사용해 에러 표시 가능

# 4. 변환 처리에 대한 이해

- `@DateTimeFormat` 의 변환처리는 누가해주는가? `WebDataBinder` 가 해준다
- `WebDataBinder` 는 `RequestMappingHandlerAdapter` 에서 변환처리
- `WebDataBinder` 도 직접 변환이 아님, `@EnableWebMvc` 를 통해 설정된 `ConversionService` 로 사용

# 5. MemberDao 클래스 중복 코드 정리 및 메서드 추가

- 미리 클래스를 필드에 할당하고 재사용 할수 있도록 하여 중복 코드 정리

# 6. @PathVariable을 이용한 경로 변수 처리

- url에 변수를 사용할 수 있게 됨
```java
@GetMapping("/members/{id}")
public string detail(@PathVariable("id") Long memberId, Model model) {
  Member member = memberDao.findById(memberId);
  
  ...
    
  }
```
- `@PathVariable` 애노테이션이 붙은 변수 타입에 알맞게 타입을 변환해줌

# 7. 컨트롤러 익셉션 처리하기

- 경로 변수를 잘 입력했는데 없는 데이터라면 500 에러 (서버 탓)
  - try-catch 구문으로 익셉션 처리도 가능함
- 경로 변수를 타입에 맞게 제대로 입력 안하면 400 에러 (클라이언트 탓)
  - 이건 `@ExceptionHandler` 애노테이션으로 처리하자
 
## 7.1 @ControllerAdvice를 이용한 공통 익셉션 처리
- `@ExceptionHandler` 는 해당 컨트롤러에서만 적용됨
- `@ControllerAdvice` 를 통해 공통적용을 시킬 수 있다
- 익셉션 처리 메서드를 객체로 덮고 `@ControllerAdvice` 애노테이션 붙인 후 빈 등록하면 됨

## 7.2 @ExceptionHandler 적용 메서드의 우선 순위
- `@ControllerAdvice` 에 속한 익셉션핸들러 보다 컨트롤러에 속한 익셉션핸들러의 **우선순위가 더 높다** 
- `@ControllerAdbice` 의 주요 속성(익셉션핸들러 대상을 정하는 기준)
  - `valuebasePackages` : 공통 설정을 적용할 컨트롤러가 속하는 기준 패키지
  - `annotations` : 특정 애노테이션이 적용된 컨트롤러 대상
  - `assignableTypes` : 특정 타입 또는 그 하위 타입인 컨트롤러 대상

## 7.3 @ExceptionHandler 애노테이션 적용 메서드의 파라미터와 리턴 타입
- `@ExceptionHandler` 애노테이션을 붙인 메서드가 가질 수 있는 파라미터 타입
  - `HttpServletRequest, HttpServletResponse, HttpSession`
  -  Model
  -  익셉션
- 리턴 타입
  - ModelAndView
  - String (뷰 이름)
  - (@ResponseBody 애노테이션 붙인 경우) 임의 객체
  - ResponseEntity (?) 
