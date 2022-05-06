# 1.스프링 MVC 핵심 구성 요소

스프링 mvc : 웹브라우저 -> DispatcherServlet -> (HandlerMapping, ViewResolver, HandlerAdapter, View)
개발자가 구현할 부분 : View를 만들 Jsp, 핸들링할 컨트롤러

![mvc](https://user-images.githubusercontent.com/69251780/167076736-50e35ba5-246e-4e32-bda3-dec972ab613d.png)


- DispatcherServlet : 모든 연결을 담당, 컨트롤러를 직접 검색하지않고 HandlerMapping 객체에 요청
- HandlerMapping은 알맞은 컨트롤러 객체를 돌려줌
- HtttpRequestHandler, @Controller 등의 특수 인터페이스도 사용할 수 있도록 HandlerAdapter 사용
- HandlerAdapter가 알맞은 결과를 DispatcherServlet에 리턴( ModelAndView 객체로 )
- ModelAndView 객체 ViewResolver에 알맞은 view 찾기 위임
- 마지막으로 View에 응답결과 생성 요청 후 웹브라우저에 전달
- DispatcherServlet가 모든 처리를 연결하지만, 처리 자체는 다른 객체에 위임한다

## 1.1 컨트롤러와 핸들러
컨트롤러 객체 찾는게 HandelrMapping인 이유(ControllerMapping 이 아닌 이유)
- 사용자가 직접 컨트롤러가 아닌 클래스를 정의해서 처리할 수도 있다
- 즉 컨트롤러 << 핸들러 로 핸들러가 상위개념
- ModelAndView를 리턴하지 않는 핸들러의 경우 HandlerAdapter가 변환해줌

# 2. DispatcherServlet과 스프링 컨테이너

- DisptacherServlet는 미리 스프링 컨테이너를 생성하고 그 안에서 HandelrMapping 등 필요한 빈 객체를 구한다

# 3. @Controller를 위한 HandlerMapping과 HandlerAdapter

- @EnableWebMvc 는 많은 설정을 추가해주는 훌륭한 애노테이션
- 위에서 나온 HandlerMapping, HandlerAdapter 등의 빈 등록을 해줌
- 그래서 컨트롤러 객체에서 리턴한 값의 이름이 그대로 뷰 이름으로 된다
  - ( return "hello"  -> View 이름이 "hello" )
 
# 4. WebMvcConfigurer 인터페이스와 설정

- @EnableWebMvc 사용하기 위해 Mvc설정을 추가해야한다 ( WebMvcConfigurer 인터페이스 사용)
- 스프링5부터는 자바8부터의 default 메소드를 사용하여 바로 구현가능 (이전버전은 WebConfigurerAdapter 클래스 상속해서 필요한것만 재정의했음)

# 5. JSP를 위한 ViewResolver

configureViewResolvers() 메소드 상속을 통해 jsp 등록
```java
@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
  registry.jsp("WEB-INF/view/", ".jsp");
  }
}
```
- registry.jsp 의미 : 첫번째 : prefix, 두번째 : suffix
- ex. "hello" 라는 view 요청이 들어오면 "WEB-INF/view/hellp.jsp" 를 찾아서 리턴
- ModelAndBView 의 경우 맵 자료형으로 저장
- ex. model.addAttribute("a", "안녕") 의 경우 jsp에서 "a"가 key, "안녕" 이 value

# 6. 디폴트 핸들러와 HandlerMapping의 우선순위

DispatcherServlet은 .jsp를 제외한 모든 요청을 자신이 처리한다
- 근데 컨트롤러에 없는 매핑이라면? -> 404에러
- /index.html 같은경우 자주 디폴트한 방식인데 이것은 어떻게 처리하나
- WebMvcConfigurer 중에서 configureDefaultServletHandling() 메서드 활용
- 어차피 default보다 @EnableWebMvc로 만든 컨트롤러의 우선순위가 높다
- 컨트롤러에 없는 매핑인 경우 default가 처리해준다

# 8. 정리
- DispatcherServlet : 웹 브라우저 요청을 받기 위한 창구, 요청 흐름 제어
- HandlerMapping : 클라이언트의 요청을 처리할 핸들러 객체 찾음
- Handler(찾은 객체) : 클라이언트의 요청 실제로 처리한 뒤 뷰,모델 설정
- HandlerAdapter : DispatcherServlet과 핸들러 객체 사이의 변환을 알맞게 처리해줌
- ViewResolver : 요청 처리 결과를 생성할 View 찾음
- View : 최종적으로 클라이언트에 응답을 생성해서 전달
