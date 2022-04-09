**자동 주입** : 의존 대상을 설정 코드에서 직접 주입하지 않고 스프링이 자동으로 빈 객체를 주입해주는 기능
  - **@Autowired** 애노테이션 활용
 
# 2. @Autowired 애노테이션을 이용한 의존 자동 주입
```java
@Bean
public MemberDao memberDao() {
  return new MemberDao();
}

@Bean
public PasswordService passwordService() {
  PasswordService pwdSvc = new PasswordService();
  pwdSvc.setMemberDao(memberDao()); // 자동 주입 기능 사용시 생략 가능
  return pwdSvc;
}
```
- 자동 주입 기능을 사용하기 위해서 @Autowired 애노테이션을 의존객체 필드에 붙인다
```java
/* service */
public class PasswordService {
  
  @Autowired
  private MemberDao memberDao;
  
  ...
}
```
- @Autowired 애노테이션 덕에 config에서 ```pwdSvc.setMemberDao(memberDao())``` 생략 가능
- @Autowired 는 필드 뿐 아니라 **세터 메소드** 에도 사용 가능( 둘 다 기능은 같다)
  - 하지만 순환참조 방지를 위해 두 방법 모두 대신 생성자 주입이 좋다(스프링 팀의 추천)//성진이가 적어놓은 블로그 통해 확인
- 자동 주입시 익셉션 상황
  1. 일치하는 빈이 없는 경우 : 만들어라
  2. 일치하는 빈이 2개 이상인 경우 : 하나를 지우거나, **@Qualifier** 애노테이션 활용

# 3. @Qualifier 애노테이션을 이용한 의존 객체 선택
- 말 그대로 **수식어** 설정 가능
- 빈 객체, 클래스 정의에 붙여 수식어를 지정한다

```java
@Bean
@Qualifier("naver")
public Office myOffice1() {
  return new Office();
}

@Bean
@Qualifier("kakao")
public Office myOffice2() {
  return new Office();
}

public class OfficePrinter {
  private Office myOffice;
  
  @Autowired
  @Qualifier("naver")
  public void setMyOffice(Office myOffice) {
    this.myOffice = myOffice;
  }
```

## 3.1 빈 이름과 기본 한정자
- 빈 객체, @Autowired 의존객체의 한정자(qualifier)는 따로 애노테이션으로 명시하지 않으면 객체이름이 한정자가 된다

# 4. 상위.하위 타입 관계와 자동 주입
- 자동 주입은 일치하는 빈을 찾을때 상속받은 객체도 일치한다고 판단한다 -> 자동 주입 익셉션 발생
- 해결책
  1. @Qualifier 애노테이션 활용(config 클래스와, @Autowired된 필드나 세터메소드에 붙인다) 
  2. 상속한 클래스 말고 상속받은 클래스를 의존 객체로 받도록 변경
  - 설계 의도에 따라 어떤 해결책 할지 바뀔 것 같다

# 5. @Autowired 애노테이션의 필수 여부
- 일부 메소드는 Null에 대한 대처가 되어있다. (대부분은 대처를 해놓는다)
- @Autowired는 일치하는 빈을 찾지 못하면 Null이 아닌 익셉션을 발생시킨다
- 익셉션 대신 NPE를 감수하고 실행시키고 싶다면 세가지 방법이 있다
  1. @Autowired 의 required 속성 활용
    - ```@Autowired(required = false)``` default: true, false시 일치하는 빈 못찾으면 자동주입 수행 X
    - 값을 null로 받지는 않고 초기화된 값으로 받는다
  2. required 대신 Optional 래퍼객체 사용(코드에서 Null 체크를 해줘야함)
  3. @Nullable 애노테이션 사용(코틀린의 ?. 연산자와 유사)

# 6. 자동 주입과 명시적 의존 주입 간의 관계
- 직접 의존 주입과 자동 주입을 동시에 한다면?
- 우선순위가 존재한다 -> **자동 주입 >>> 명시적 의존 주입**
- 그러므로 하나라도 자동 주입을 했다면 웬만하면 모든 의존 객체를 자동 주입하도록 하자 
