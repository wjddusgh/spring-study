- 스프링 MVC 웹 개발은 결국 **컨트롤러, 뷰 코드 구현** 을 의미

# 2. 요청 매핑 애노테이션을 이용한 경로 매핑

필요한 기능
- 특정 요청 URL을 처리할 코드 (Controller)
- 처리 결과를 HTML과 같은 형식으로 응답하는 코드 (View)

Controller
- 같은 경로의 메소드는 묶는게 좋다
- ex. /register/step1, /register/step2는 같은 register 컨트롤러로 묶자
