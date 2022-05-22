# 1. 프로젝트 준비
# 2. 로그인 처리를 위한 코드 준비
# 3. 컨트롤러에서 HttpSession 사용하기

- 로그인 상태를 유지하는 방법
  - HttpSession(세션) 이용 (서버)
  - 쿠키 이용 (클라이언트)
  - 외부 DB에 세션 데이터 보관하기 (세션과 비슷)

- HttpSession 사용 방법
  - 요청 매핑 애노테이션 적용 메서드에 HttpSession 파라미터 추가 (Post 요청 파라미터에 추가)
  - 요청 매핑 애노테이션 적용 메서드에 HttpServeletRequest 파라미터 추가하고 이를 이용해 HttpSession 구하기 (HttpServletRequest#getSession() 메서드 활용)
  - 첫번째 방법은 항상 세션을 생성하지만 두번쨰는 필요할 때만 생성 가능
  - 세션 삭제방법 : HttpSession#invalidate() 메서드 이용

# 4. 비밀번호 변경 기능 구현

- 서버 재부팅시 세션 정보는 날아감
- 하지만 톰캣은 디비에 저장하므로 클라이언트가 쿠키에 저장된 세션정보 부르면 세션정보 유지됨
- context.xml 의 <Manager pathname=""/>  사용시 톰캣에서도 세션 정보 날아가게 할수있다

# 5. 인터셉터 사용하기

- 로그인 하지 않은 상태에서 로그인이 되어있어야 하는 url 접속시 (ex. 비밀번호 변경) 접속이 되면 안됨
- `HttpSession` 에 로그인정보 (ex. `authInfo`) 가 있는지 확인후 없다면 로그인 경로로 리다이렉트 시키자
```java
AuthInfo authInfo = (AuthInfo) session.getAttribute("authInfo");
if (authInfo == null) {
  return "redirect://login";
}
```
- 그런데 로그인정보가 필요한 기능이 많다면?? 각 기능마다 같은 코드를 넣어줘야함
- 이럴때 사용하는게 `HandlerInterceptor`

## 5.1 HandlerInterceptor 인터페이스 구현하기
- 세가지 시점에 공통기능을 넣을 수 있는 인터페이스
  - 컨트롤러(핸들러) 실행 전
  - 컨틀롤러(핸들러) 실행 후, 아직 뷰를 실행하기 전
  - 뷰 실행 후

- `HandlerInterceptor` 인터페이스에 정의되어있는 메서드
  - `boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler)`
    - 컨트롤러(핸들러) 실행 전 필요햔 기능을 구현할 때 사용
    - `handler` 파라미터는 웹 요청 처리할 컨트롤러(핸들러) 객체
    - 로그인이 안 된 경우 핸들러 실행 안함
    - 컨트롤러 실행 전에 컨트롤러에서 필요한 정보 생성
    - `false` 리턴시 컨트롤러 실행 안함
  
  -  `void postHandle(HttpServletRequest req, HttpServletResponse res, Object handler, ModelAndView mav)`
      - 핸들러가 정상적으로 실행된 이후에 추가기능 구현할 때 사용
      - 컨트롤러 익셉션 발생시 이 메서드 실행 안함 
  -  `void afterCompletion(HttpServletRequest req, HttpServletResponse res, Object handler, Exception e)`
      - 뷰가 클라이언트에 응답을 전송한 뒤에 실행됨
      - 컨트롤러 익셉셔 발생시 네번째 파라미터에 익셉션 값 전달(익셉션 없으면 null 전달)
      - 컨트롤러 실행 이후 터진 익셉션을 로그로 남기는 등 후처리에 적합

- `HandlerInterceptor` 메서드는 디폴트 메서드 이므로 필요한 메서드만 구현하면 됨
- `preHandle()` 구현 시 `false`만 줄 뿐아니라 리다이렉트 위치도 response 로 전달

## 5.2 HandlerInterceptor 설정하기
- `WebMvcConfigurer` 의 `addInterceptors(InterceptorRegistry r)` 메소드 구현을 통해 설정
- `InterceptorRegistry#addPathPatterns` 메서드로 인터셉터 적용할 경로 패턴 지정
  - Ant 경로 패턴 사용 (* , **, ?)
  - 여러 경로 지정 가능(가변인자처럼)

# 6. 컨트롤러에서 쿠키 사용하기

- 스프링 MVC에서 쿠키를 사용하는 방법
  - `@CookieValue` 애노테이션 사용(컨트롤러 메서드의 파라미터로 쿠키 값 받음, 그러므로 submit시 쿠키생성해서 보내야함)
    - 쿠키 저장시 이메일, 전화번호, 주민번호 같은 내용은 암호화하자!  
