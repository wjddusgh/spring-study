# 1. 컨테이너 초기화와 종료

스프링 컨테이너는 **초기화, 종료** 라는 라이프사이클을 가진다
- 컨테이너 초기화 : 설정클래스에서 정보를 읽어와 알맞은 빈 객체 생성, 의존 주입, 초기화(빈 객체의 지정된 메소드 호출)
- 컨테이너 사용 : 빈 객체 사용
- 컨테이너 종료 : close() 메소드 사용, 빈 객체 소멸(이 때도 빈 객체의 지정된 메소드 호출)

# 2. 스프링 빈 객체의 라이프사이클

스프링 컨테이너는 빈 객체의 라이프사이클을 관리한다
```mermaid
graph LR
a(객체 생성) --> b(의존 설정)
b(의존 설정) --> c(초기화)
c(초기화) --> d(소멸)
```

## 2.1 빈 객체의 초기화와 소멸 : 스프링 인터페이스

빈 객체의 지정한 메소드는 스프링 인터페이스에 정의되어 있다
- 초기화 : **org.springframework.beans.factory.InitializingBean** <- void afterPropertiesSet()
- 소멸 : **org.springframework.beans.factory.DisposableBean**    <- void destory()
- 지정된 메소드가 필요할 경우 위의 인터페이스를 상속받아 메소드를 구현하면 된다
- 필요 예시 : DB 커넥션 풀, 채팅 클라이언트 등 (연결 생성, 연결 종료)

## 2.2 빈 객체의 초기화와 소멸 : 커스텀 메서드

**외부**에서 제공받은 클래스를 스프링 빈 객체로 등록할 경우 인터페이스 상속받아 메소드 실행 불가
- @Bean 의 initMethod, destroyMethod 속성을 활용하라 ( 파라미터는 메소드 이름)

```java
/* client class */
public Client client {
  private String host;
  
  public void setHost() {...}
  public void connect() {...}   //@Bean 의 속성 파라미터로 넣을 메소드이므로 파라미터가 없어야함
  public void sent() {...}
  public void close() {...}     //근데 파라미터 있으면 익셉션 발생
}

/* config class */
@Bean(initMethod = "connect", destroyMethod = "close")
public Client client() {
  Client client = new Client();
  client.setHost("host");
  return client;
}
```
- 클래스의 메소드이므로 @Bean 속성 파라미터 대신 직접 호출도 가능
- 다만 메소드가 두번 호출되지 않도록 주의

# 3. 빈 객체의 생성과 관리 범위

빈 객체는 디폴트로 싱글톤 범위 이다
- config 클래스 에서 빈 객체에 **@Scope 애노테이션** 활용시 범위설정을 바꿀 수 있다
- @Scope("prototype") 의 경우 매번 새로운 객체 생성하여 리턴
- prototype의 경우 컨테이너 종료시 빈객체 소멸이 되지 않으므로 직접 소멸 처리 해줘야 한다



