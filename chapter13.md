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
