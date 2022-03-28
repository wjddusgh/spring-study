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
- main 과 분리하기 위한 용도
