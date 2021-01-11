# Filter, Interceptor, AOP

Security와 Logging을 구축하면서, 몰랐던 부분을 간단히 정리

<br>

## Filter, Interceptor, AOP의 흐름

![image](https://user-images.githubusercontent.com/23515771/104183949-115a8e00-5456-11eb-8950-ff550e8fb335.png)

1. 서버를 실행시키면, `Servlet`이 올라오는 동안에 `init`이 실행된 후, `doFilter`가 실행된다.

2. Controller에 들어가기 전, `PreHandle`이 실행된다.

3. 애플리케이션에서 작업을 마친 후, `@Around`, `PostHandle`, `doFilter` 순으로 진행된다.

<br>

## Filter

`Dispatcher Servlet` 영역에 들어가기 전, 앞 범위에서 수행되며 애플리케이션에서 작업을 마친 후에 응답 처리에 대해서 변경, 조작을 할 수 있다. 추가로 `Spring Context` 이전에 실행되므로, 스프링과 무관하다.

> Filter의 실행 메소드

- `init()` : 필터 인스턴스 초기화

- `doFilter()` : 실제 처리 로직 수행

- `destroy()` : 필터 인스턴스 종료

<br>

## Interceptor

`Dispatcher Servlet`이 Controller를 호출하기 전,후에 끼어들기 때문에 `Spring Context` 내부에서 Controller에 관한 `Request`와 `Response`에 관여한다. 그리고 스프링의 모든 `@Bean`에 접근이 가능하다.

> Interceptor 실행 메소드

- `preHandle()` : Controller 실행 전

- `postHandle()` : Controller 실행 후, View Rendering 실행 전

- `afterCompletion()` : View Rendering 이후

`preHandle()`에서 전처리가 이루어지고, `postHandle()`에서 후처리를 한다.

<br>

## AOP

Controller 처리 후, 비즈니스 로직에서 실행된다. 주로 `로깅`, `트랜잭션`, `에러 처리`등 비즈니스 단의 메소드에서 구체적인 조정이 필요할 때, 사용한다. `Filter`와 `Interceptor`와 달리 메소드 전후 지점에서 자유롭게 설정이 가능하며 주소, 파라미터, 어노테이션등 다양한 방법으로 대상을 지정할 수 있다.

<br>

## @Advice와 HandleInterceptor 차이

- `@Advice`는 `JoinPoint`와 `ProceedingJoinPoint`등을 활용하여 호출이 가능하다.

- `HandleInterceptor`의 경우에는 `HttpServletRequest`, `HttpServletResponse`를 파라미터로 사용한다.

<br>

## 참고

- [https://velog.io/@sa833591/Spring-Filter-Interceptor-AOP-%EC%B0%A8%EC%9D%B4-yvmv4k96](https://velog.io/@sa833591/Spring-Filter-Interceptor-AOP-%EC%B0%A8%EC%9D%B4-yvmv4k96)
