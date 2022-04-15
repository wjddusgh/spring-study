가장 어려운 챕터라고 합니다

# 2. 프록시와 AOP

책의 예시 : Calculator 인터페이스의 팩토리얼 메소드를 반복문, 재귀로 각각 구현한 후 실행시간 비교
1. 메소드 코드 내에서 직접 시간 측정 
  - 재귀함수에서는 출력이 반복된다
  - 하드코딩이기 때문에 초 단위를 바꾸는 등 유지보수에 불편하다
2. 프록시 객체를 활용하자

```java
public class X implements Calculator {
  
  private Calculator delegate;  // 다른 객체를 주입받아 사용한다
  
  public X(Calculator delegate) { // Calculator 객체를 주입받는 생성자
    this.delegate = delegate;
  }
  
  @Override
  public long factorial(long num) {
    long start = System.nanoTime();
    long result = delegate.factorial(num);  // 다른 객체의 팩토리얼 메소드 실행
    long end = System.nanoTime();
    System.out.prinln(~~~출력~~~);
    
    return result;
  }
}
```
  - Calculator를 구현하는 X 클래스에서 Calculator 필드를 생성자를 통해 할당받는다
  - X 클래스에서 시간을 측정하고 팩토리얼 계산은 주입받은 다른 Calculator를 이용한다
  
  **장점**
  - 기존 코드의 변경이 없다
  - 코드의 중복을 제거했다
  
  **구현의 특징**
  - 다른 객체에 메소드 실행을 위임함 (예시는 팩토리얼 메소드)
  - 자기 자신은 부가적인 기능을 실행함 (예시는 실행 시간 측정)
  
  결국 **프록시(proxy)** 란 :핵심 기능의 실행은 다른 객체에 위임하고 부가적인 기능을 제공하는 객체 
  **AOP** 란 : 핵심기능 구현과 공통기능 구현의 분리가 핵심 
  
  (근데 또 프록시 보다는 데코레이터랑 비슷하단다. 참고로 Kotlin **by** 가 데코레이터 패턴을 위한 연산자)
  
  ## 2.1 AOP
  Aspect Oriented Programming : 공통 기능을 분리해서 재사용성을 높여줌
  
  **핵심기능에 공통기능 삽입하는 방법**
  1. 컴파일 시점에 코드에 공통 기능 삽입
  2. 클래스 로딩 시점에 바이트 코드에 공통기능 삽입
  - 1,2번은 AspectJ 같은 AOP 전용 툴 사용해야 가능함
  3. 런타임에 프록시 객체를 생성해서 공통기능 삽입
  - 스프링이 제공해주는 방식
  
  **AOP 주요 용어**
  |용어|의미|
|------|---------|
|Advice|공통기능을 핵짐기능에 언제 적용할지 정의|
|Joinpoint|Advice 적용 가능한 지점|
|Pointcut|Joinpoint 의 부분집합|
|Weaving|Advice를 핵심 로직에 적용하는것| 
|Aspect|여러 객체에 공통으로 적용되는 기능, 트랜잭션이나 보안 등이 예시|

## 2.2 Advice의 종류
  |종류|설명|
|------|---------|
|Before Advice|대상 객체 메소드 호출 전에 공통기능 실행|
|After Returning Advice|대상 객체의 메소드가 익셉션 없이 실행된 이후에 공통기능 실행|
|After Throwing Advice|대상객체 메소드가 익셉션 된 경우 공통기능 실행|
|After Advice|대상객체 메소드 실행 후공통기능 실행 (finally 와 유사)| 
|Around Advice|대상객체 메소드 실행 전/후, 익센션발생에 공통기능 실행|

- 널리 사용되는건 Around Advice, 이유는 다양한 시점에 원하는 기능 삽입 가능해서

# 3. 스프링 AOP 구현

- Aspect 로 사용할 클래스에 **@Aspect** 붙임
- **@Pointcut** 으로 공통기능 적용할 Pointcut wjddml
- 공통기능을 구현한 메소드에 **@Around** 붙임
- 애노테이션을 활용하면 스프링이 자동으로 프록시 객체 만들어준다

## 3.1 @Aspect, @Pointcut, @Around를 이용한 구현
```java
/* aspect */

@Aspect
public class ExeAspect {

  @Pointcut("execution(패키지 이름)")  // 이름 명시된 패키지와 그 하위에 위치한 타입의 Public 메소드를 Pointcut 으로 설정
  private void publicTarget() {}
  
  @Around("publicTarget()") // Around Advice 설정, publicTarget() 메소드에 정의한 Pointcut에 공통기능 적용
  public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
    // 메소드 실행 전 공통기능
    try {
      Object result = joinPoint.proceed() // 핵심기능 실행
      return result;
    } finally {
      // 메소드 실행 후 공통기능
    }
  }
}
```
```java
/* config */

@Configuration
@EnableAspectJAutoProxy // 이 애노테이션을 통해 @Aspect 가 적용된 빈 객체를 찾고 @Pointcut, @Around 설정 적용
public class AppCtx {
  @Bean
  public ExeAspect exeAspect() {
    return new ExeAspect();
  }
  ...
}
```
```java
/* main */
//변화 없이 핵심기능으로 실행
```
- 우리는 코드로 핵심기능 클래스를 실행하지만 스프링이 프록시를 실행시켜준다

## 3.2 ProceedingJoinPoint의 메소드
ProceedingJoinPoint 인터페이스의 proceed()외에도 다양한 메소드 있다(파라미터 필요할때 쓰는 용도)
- Signature getSignature() : 호출되는 메소드에 대한 정보
  - Signature 인터페이스 제공 메소드
  - String getName()
  - String toLongString()
  - String toShortString() : 축약표현, 기본구현은 메소드 이름만 
- Object getTarget() : 대상 객체
- Object[] getArgs() : 파라미터 목록

# 4. 프록시 생성 방식
```java
/* main */
ImplementedCalculator cal = ctx.getBean("calculator", ImplementedCalculator.class); //getBean()의 두번쨰파라미터로 인터페이스 대신 인터페이스 구현한 클래스 넣음
```
위 방식은 익셉션 발생
- 스프링이 자동으로 만든 프록시 객체는 인터페이스를 이용함
- 그런데 인터페이스를 구현한 클래스를 통해 받으려하면 익셉션 발생

빈 객체가 인터페이스를 상속할 때 클래스를 이용해서 프록시 생성하고싶은 경우 
**@EnableAspectJAutoProxy(proxyTargetClass = true)**  : 애노테이션의 proxyTargetClass 속성 활용

## 4.1 execution 명시자 표현식
execution(수식어패턴, 리턴타입패턴, 클래스이름패턴, 메서드이름패턴(파라미터패턴))
- 수식어패턴 : public 만 됨, 생략가능
- 리턴타입패턴 : 리턴 타입 명시
- 클래스이름패턴 : 클래스 이름 명시
- 메서드이름패턴 : 메서드 이름 명시, 파라미터있을경우 명시

## 4.2 Advice 적용 순서

순서는 스프링이나 자바의 버전, 코드에 따라 달라질 수 있다
적용 순서가 중요하다면 **@Order** 애노테이션 활용
- 파라미터 값이 작으면 먼저 적용, 크면 나중에 적용
- Ex. @Order(1) 먼저, @Order(2) 나중

## 4.3 @Around의 Pointcut 설정과 @Pointcut 재사용

@Pointcut(execution("")) 이 아닌 @Around("execution("")) 가능!
그래서 하나의 @Pointcut 클래스 만들어놓고 다양한 @Around 붙이는 Aspect에서 execution 붙여 재사용 가능

