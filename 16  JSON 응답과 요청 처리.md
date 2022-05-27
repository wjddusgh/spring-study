# 1. JSON 개요

- `JSON(JavaScript Object Notation)`은 간단한 형식을 갖는 문자열로 데이터교환에 주로 사용한다
- JSON 규칙
  - 중괄호 사용해서 객체 표현
  - 객체는 쌍을 가짐
  - 이름, 값은 콜론(:)으로 구분
  - ex. `{"name": "정연호"}`
  - 값에 가능한 타입: 
    - 문자열, 숫자, 불리언, null
    - 배열
    - 다른 객체

# 2. Jackson 의존 설정

- Jackson: 자바 객체와 JSON형식 문자열 간 변환을 처리하는 라이브러리
- 프로퍼티 타입이 배열이나 List인 경우 JSON 배열로 변환됨

# 3. @RestController로 JSON 형식 응답

- 스프링 MVC 에서 JSON 형식으로 데이터 응답하는법 : @Controller 대신 `@RestController` 사용  // 스프링4이전엔 @ResponseBody를 메서드에 붙임
- Jackson 라이브러리 존재하면 JSON 형식의 문자열로 반환해서 응답
- 크롬 브라우저에 json-formatter 확장프로그램 설치하면 이쁘게 보인다

## 3.1 @JsonIgnore를 이용한 제외 처리
- JSON 응답에 포함시키지 않을 대상에 `@JsonIgnore` 붙이면 제외시킴

## 3.2 날짜 형식 변환 처리: @JsonFormat 사용
- 유닉스 타임 스탬프 : 1970년 1월 1일 이후로 흘러간 시간, 보통 초단위인데 Jackson은 밀리초단위
- 이쁘게 나타내고싶다면 `@JsonFormat` 사용
  - ex. `@JsonFormat(shape = Shape.STRING)` // ISO-8601(날짜와 시간과 관련된 데이터 교환을 다루는 국제 표준) 형식으로 변환
  - 혹은 패턴 사용가능 `@JsonFormat(pattern = "yyyyMMddHHmmss")`
  - 혹은 다양한 라이브러리 활용 (day.js, date-fns 등 // 프론트에서)
 
## 3.3 날짜 형식 변환 처리 : 기본 적용 설정
- 날짜 형식 변환할 모든 대상에 애노테이션 붙이기는 귀찮다
- 응답으로 변환할 Converter에 적용해보자
- Jackson 은 JSON으로 변환을 `MappingJackson2HttpMessageConverter` 사용
- config에서는 `WebMvcConfigurer` 인터페이스의 `extendMessageConverters()` 메서드 구현해서 사용

## 3.4 응답 데이터의 컨텐츠 형식
- Content-Type : application/json 으로 해라

# 4. @RequestBody로 JSON 요청 처리

- POST, PUT 방식을 사용하면 JSON 형식 데이터를 요청 데이터로 전송할 수 있다 (쿼리스트링형식이 아니라)
- 커맨드 객체에 `@ResquestBody` 붙이면 매핑된다
- 요청 컨텐츠 타입은 application/json 이어야함
- 중복된 id(key겠지?) 전송시 409 에러(Conflict) 리턴

## 4.1 JSON 데이터의 날짜 형식 다루기
- 별도 설정이 없다면 `yyyy-MM-ddTHH:mm:ss` 로 변환
- 바꾸고싶다면 `@JsonFormat` 사용
- 전체 적용하고싶다면 3.3 참고

## 4.2 요청 객체 검증하기
- JSON 형식 데이터도 똑같이 `@Valid` 나 별도 Validator로 검증 가능하다 검증실패시 400 에러(Bad Request) 리턴

# 5. ResponseEntity로 객체 리턴하고 응답 코드 지정하기

- `HttpServletResponse`의 setStatus(), sendError() 사용시 에러를 HTML로 응답한다
- 정상은 JSON인데 에러는 HTML이면 클라이언트는 두가지 다 처리해야한다
- 에러도 JSON 전송 해주자

## 5.1 ResponseEntity를 이용한 응답 데이터 처리
- `ResponseEntity` 사용하면 에러도 JSON으로 전송 가능

```java
if (member == null) {
			return ResponseEntity
					.status(HttpStatus.NOT_FOUND)
					.body(new ErrorResponse("멤버없다"));
		}
		return ResponseEntity.ok(member);
```
- body 필용없다면 .body() 대신 .build() 하면 된다
  - 대신 build() 앞에 상태 메서드 붙여줘야함 
    - noContent(): 204
    - badRequest(): 400
    - notFount(): 404 
- 헤더에 값 전달하고싶다면 uri 생성해서 .created(uri) 붙이자. uri 는 URI 타입

## 5.2 @ExceptionHandler 적용 메서드에서 ResponseEntity로 응답하기
- 모든 메서드에 ResponseEntity를 사용한 코드를 붙이면 중복이 매우 커지고 유지보수에 안좋다
- `@ExceptionHandler` 를 적용해서 여기에 ResponseEntity를 사용하자
- `@RestControllerAdvice` 를 사용하면 에러 처리 코드를 별도 클래스로 분리할 수도 있다 // 응답 타입만 다르고 @ControllerAdvice와 동일

## 5.3 @Valid 에러 결과를 JSON으로 응답하기
- @Valid 로 검증 실패시 400 에러 -> HTML로 리턴 아주 곤란
- Errors 타입 파라미터를 넣어서 직접 에러응답에 ResponseEntity 적용하자
