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
