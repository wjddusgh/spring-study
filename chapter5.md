# 1. @Component 애노테이션으로 스캔 대상 지정
- @Component 애노테이션이 붙여진 클래스는 스프링이 검색해서 빈으로 등록한다
- @Component의 매개변수는 @Qualifier와 같은 역할을 하는듯하다
- Ex) @Component("naver") 붙이면 빈 이름은 "naver"가 되고, default시 클래스 이름으로 된다(첫글자를 소문자로 변경)

# 2. @ComponentScan 애노테이션으로 스캔 설정
- @Component 가 붙여진 클래스를 찾는 범위는 @ComponentScan 애노테이션의 **basePackages** 속성에 넣는 패키지와 그 하위패키지 모두 다.
- Ex) @ComponentScan(basePackages = {"spring"})

# 4. 스캔 대상에서 제외하거나 포함하기
- @ComponentScan 애노테이션의 속성으로 **excludeFilters** 가 있다
```java
@ComponentScan(
  basePackages = {"spring"}, 
  excludeFilters = @Filter(type = FilterType.REGEX, pattern = "spring\\..*Dao")
)
```
- 특정 애노테이션을 사용한 클래스도 필터로 제외시킬 수 있다
```java
@ComponentScan(
  basePackages = {"spring"}, 
  excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = {NoProduct.class, ManualBean.class})
)
```
- excludeFilters는 배열로 여러 필터를 넣을 수 있다
```java
@ComponentScan(
  basePackages = {"spring"}, 
  excludeFilters = {
    @Filter(type = FilterType.REGEX, pattern = "spring\\..*Dao"),
    @Filter(type = FilterType.ANNOTATION, classes = {NoProduct.class, ManualBean.class})
    }
)
```

## 4.1 기본 스캔 대상
- @Component
- @Controller
- @Service
- @Repository
- @Aspect
- @Configuration
위의 애노테이션은 모두 내부에 @Component 애노테이션을 포함하고 있다

# 5. 컴포넌트 스캔에 따른 충돌 처리

## 5.1 빈 이름 충돌
- 서로 다른 패키지에서 같은 이름의 클래스가 존재할 경우 발생 -> 둘 중 하나 이상 빈 이름 명시적 지정 해주어야한다
## 5.2 수동 등록한 빈과 충돌
- 자동 주입과 다르게 컴포넌트 스캔 보다 설정클래스의 빈 지정이 우선순위가 높다
- 만약 설정클래스에서 이름을 다르게 지었다면? -> 같은 타입의 2개의 빈 생성 : 4장에서 처럼 일치하는 빈이 2개일 수 있으므로 빈 이름 명시적 지정 해주어야 한다
