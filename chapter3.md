# 1. 의존이란?
DI (Dependency Injection) : **의존성 주입**

```java
public class MemberRegisterService {
    private MemberDao memberDao = new MemberDao();
    
    // 메소드
}
```
- MemberRegisterService 클래스가 DB 처리를 위해 MemberDao 클래스 사용
- MemberRegisterService -> MemberDao **의존**
- MemberDao 클래스를 사용하기 위해 직접 new 연산자를 통해 생성하여 할당
    - 가장 쉬운 방법이지만 유지보수 관점에서 문제 발생
    - MemberDao 클래스 변경 시 직접 MemberRegisterService 클래스의 코드를 수정해야함

# 2. DI를 통한 의존 처리

```java
public class MemberRegisterService {
    private MemberDao memberDao;

    public MemberRegisterService(MemberDao memberDao) {
        this.memberDao = memberDao;
    }
    
    // 메소드
}

MemberDao memberDao = new MemberDao();
MemberRegisterService svc = new MemberRegisterService(memberDao) // 매개변수를 통해 의존객체 주입
```
- 클래스 생성자 매개변수를 통해 의존 객체 전달받음
- 직접 생성하지 않으므로 DI 패턴을 따르고 있다

# 3. DI와 의존 객체 변경의 유연함

```java
public class CachedMemberDao extends MemberDao {
    ...
}
```

- MemberDao를 상속받은 CachedMemberDao 클래스가 있다
- 의존객체를 CachedMemberDao 클래스로 변경하고자 한다
## 의존객체 직접 주입
```java
public class MemberRegisterService {
//  private MemberDao memberDao = new MemberDao();
    private MemberDao memberDao = new CachedMemberDao();
    ...
}
```
- 의존객체 변경을 위해 코드를 직접 수정해야함
- 의존객체를 의존하는 클래스가 많아질수록 번거로워짐

## DI 패턴 사용
```java
public class MemberRegisterService {
    private MemberDao memberDao;
    
    public MemberRegisterService(MemberDao memberdao) {
        this.memberDao = memberDao;
    }
    ...
}

MemberDao memberDao = new CachedMemberDao();    // 이 코드만 수정
MemberRegisterService svc = MemberRegisrerService(memberDao);
```
- memberDao 인스턴스 생성 코드만 변경해주면 의존 클래스는 변경하지 않아도 됨

# 5. 객체 조립기

- 객체를 생성할 클래스
- main 과 분리하기 위한 용도(생성자를 메인으로부터 감출 수 있음)

# 6. 스프링의 DI 설정

- 스프링은 의존, DI 기능을 지원하는 조립기와 유사한 기능을 제공한다
- **@Configuration** 애노테이션을 통해 스프링 설정 클래스 지정
- **@Bean** 애노테이션을 통해 빈(Bean) 객체 지정
- **AnnotationConfigApplicationContext(config 클래스)** 를 통해 application context 받고 getBean() 메소드 활용하여 빈객체 얻음

## DI 방식 : 생성자 vs 세터 메서드
- 생성자 : 빈 개체 생성 시점에 모든 의존객체 주입
    - 파라미터가 많을 경우 유추하기 어려움 
- 세터 메서드 : 메서드 이름을 통해 어떤 의존객체가 주입되는지 알 수 있음
    - 빈객체를 전달하여 NPE(Null Pointer Exception) 위험

# 7. @Configuration 설정 클래스의 @Bean 설정과 싱글톤

- 스프링은 config 클래스를 직접 사용하지 않고 내부적으로 상속클래스를 만들어 한 번 생성한 객체를 보관했다가 이후에 동일한 객체를 리턴한다
- 이로 인해 빈객체는 매번 같은 인스턴스를 사용(싱글톤)

# 8. 두 개 이상의 설정 파일 사용하기

- 설정 클래스 파일을 여러개로 나눌 때 양 쪽 파일 모두에 같은 빈객체가 필요할 경우 생길 수 있다
- 
