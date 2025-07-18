# Bean 객체

Spring의 3가지 핵심 기술은 의존성 주입, 관점 지향 프로그래밍, 서비스 추상화이다.
이 기술 중 하나인 `의존성 주입`을 이해하기 위해 빈 객체를 이해할 필요가 있다.

스프링 빈과 스프링 빈 컨테이너 사용법에 대해서 알아보자.

---
# `1. Bean 객체란?`
Spring Bean 객체란, Spring의 Bean 컨테이너에서 관리되는 순수 자바 객체를 의미한다.

Bean 컨테이너에 있는 객체들은 다른 객체에서 참조하여 사용할 수 있다. 또한 컨테이너 내부의
객체들은 이름+타입 이 중복되지 않으므로, 싱글톤 객체로 처리할 수 있다.


## `Spring Bean의 특징`

- 스프링 빈 컨테이너가 관리하는 POJO(순수 자바) 객체이다.
- 스프링 빈 컨테이너가 생성, 주입, 종료까지 관리한다. 이 단계를 Bean의 생명주기라고 한다.
- Spring Bean은 다른 Bean을 참조할 수 있다. 이때 참조되는 빈에 객체를 넣어주게 되는데 이를 `주입`이라고 한다.
- Spring 컨테이너에서 Bean의 이름과 타입 정보를 통해 찾으느 빈 객체를 다른 빈 객체의 멤버 변수나 메서드 인수로 넣어 줄 수 있다. 이를 `의존성 주입`이라고 한다.
- `Bean을 구분하는 것`은 class 타입과 이름이므로, 이 둘을 중복해서 빈을 생성하면 안된다.

## `Bean을 읽고 생성하는 과정`
1. 스프링 빈 컨테이너 구현체에 따라 정해진 설정 파일을 로딩한다. (기본 구현체는 ConfigurableApplicationContext)
2. 설정 파일에 정의된 스프링 빈을 로딩한다.
3. 지정된 class 경로에 위치한 class 들을 스캔하고 Bean이 정의되어 있으면 로딩한다.
4. 로딩을 마친 빈 컨테이너가 스프링 Bean을 생성하고 컨테이너에서 관리한다.
5. Bean 사이의 의존성이 있는 객체들을 컨테이너가 조립한다.
6. 스프링 Bean 컨테이너 구현 클래스가 있을 경우 추가 작업을 한다.
7. 작업이 완료되면 실행 준비를 한다.

---

# `2. Bean 객체를 선언하는 방법`

## `@Bean`

@Bean을 통해 Bean 객체를 선언할 수 있다. 우선 Bean 애너테이션 코드를 이해해보자.

```java
@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Bean {
    @AliasFor("name")
    String[] value() default {};

    @AliasFor("value")
    String[] name() default {};

    boolean autowireCandidate() default true;

    String initMethod() default "";

    String destroyMethod() default "(inferred)";
}
```

`@Target({ElementType.METHOD, ElementType.ANNOTATION_TYPE})`
- @Bean 애너테이션은 메서드나 다른 애너테이션에 사용할 수 있다.
- 메서드에 사용할 경우, 메서드에서 반환되는 객체를 Bean으로 생성한다.

`value(), name`
- value와 name이 있는데, 서로를 참조하므로 둘 중 하나만 설정해도 동일하게 작동한다.
- 빈 컨텍스트에서 이름을 통해 찾아낼 때 사용한다.
- 기본값은 메서드 이름이다.

실제 코드를 통해 이해해보자.

```java
@SpringBootApplication
public class SpringBean {
    public static void main(String[] args) {
        ConfigurableApplicationContext ctxt =
                SpringApplication.run(SpringBean.class, args);
    
        Money money = ctxt.getBean("money", Money.class);  // 빈 컨텍스트에서 직접 가져올 수 있다.
    }

    @Bean(name="money") 
    public Money getManyMoney() {
        return new Money(100000);
    }
}
```
ctxt라는 스프링 빈 컨텍스트 내부에 money라는 이름을 갖는 Money 타입의 객체를 사용하는 코드이다.

만약 @Bean에 `이름을 지정하지 않는다면`, 기본값으로 메서드 이름인 getManyMoney가 할당된다.


# `3. 자바 설정`
Spring에서 사용할 Bean 객체의 범위를 지정할 수 있다.

## `@Configuration`
`자바 설정 클래스`를 정의하는데 사용하는 애너테이션이다. 이게 없으면 `ApplicationContext`이 로딩되지 않는다.

하지만 위의 코드에서는 @Configuration이 없이 잘 실행되었는데 그 이유는 아래의 코드를 보면 알 수 있다.

```java
...
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    ...
}
```
기본적으로 `@SpringBootApplication` 내부에 `@SpringBootConfiguration`이 정의되어있고,
그 내부에 `@Configuration` 이 정의되어 있으므로 실행될 수 있는 것이다.

---

## `@ComponentScan`
스프링 프레임워크를 설정하는 애너테이션 중에 하나다.
`@ComponentScan`은 설정한 패키지 경로에 있는 @Bean 설정, 스테레오 타입 애너테이션이 선언된
클래스, 자바 설정 클래스들을 스캔한다.
선언된 클래스들을 스캔한다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {
    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
    ...
}
```
`basePagckages`
- 스캔할 패키지 경로를 문자열로 입력하는 것이다. 
- `배열`로 여러 패키지 경로를 입력할 수도 있다.
- value와 서로를 참조하므로 하나만 설정해도 된다.

`basePackageClasses`
- 스캔할 클래스를 입력할 수 있는 속성이다. basePackages와 동일하지만, 경로가 아닌 class르르 입력한다.

### 사용 예시
```java
@ComponentScan(
        basePackages = {"com.spring.money"},
        basePackageClasses = {Money.class}
)
public class IWantToSleep {}
```
위와 같이 애너테이션에 값을 포함하는 식으로 사용하면 된다.

패키지 경로가 다른 환경에서, 명시적으로 스캔할 범위를 지정할 때 사용할 수 있다.


## `@Import`
사실상 `@CompoenetScan`과 동일한 기능이다. 
실제로 아무거나 사용해도 큰 차이가 없이 작동한다.
다만 @Import 는 사용할 설정 클래스를 명시적으로 지정해야하므로 직관적이라는 장점이 있지만,
하나하나 작성해야하므로 개발시에는 @ComponentScan 을 사용하는 것이 더 좋다.

---
# `4. 스테레오 타입`

스테레오 타입은 @Bean 방식으로 선언하는게 아니라, `@Component`로 선언하여 Bean으로 관리하는 타입을 말한다.

스테레오 애너테이션의 종류는 4가지이다.
- `@Compoent`
- `@Controller`
- `@Service`
- `@Repository`

아래 3개의 에너테이션은 `@Component`를 내부에 포함한다.

## `@Component`
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

class 타입 위에 사용할 수 있는 애너테이션이다. 
이전 @Bean의 경우, 메서드 위에 사용했지만, 스테레오 타입 애너테이션은 class 타입 자체에 사용한다.

빈 객체는 컨테이너에 저장하기 위해 `3가지 정보`가 필요하다.
1. 이름
2. 타입
3. 객체

위 애너테이션 코드를 보면 value가 있는데, 이름을 저장하는 코드이다.
타입은 애너테이션을 사용한 Class로 자동 설정되며, 객체 또한 해당 Class 인스턴스로 자동 생성되어 지정된다.

이름은 별도로 지정하지 않을 경우, 첫 글자가 소문자인 Class 이름을 따라간다.
예시를 통해 이해해보면
```java
@Service
public class UserService {}
```
UserService에 @Service 를 정의하였다.
별도 이름을 지정하지 않았으므로 빈 컨테이너에 `userService`라는 이름으로 저장된다. 