# Spring MVC 초기화와 DispatcherServlet 아키텍처 정리

## 학습 날짜

2026-05-24

## 학습 주제

- Servlet Container와 Spring Container 연결
- WebApplicationInitializer
- ContextLoaderListener
- Spring Boot MVC 자동 설정
- DispatcherServlet
- HandlerMapping
- HandlerAdapter
- ViewResolver
- WebMvcConfigurer

---

## 1. Spring MVC 초기화가 필요한 이유

Spring MVC는 Servlet Container 위에서 동작한다.

따라서 웹 요청이 들어오면 먼저 Servlet Container가 요청을 받고, 그 다음 Spring MVC의 핵심 서블릿인 DispatcherServlet이 요청을 처리한다.

즉, Spring MVC를 이해하려면 두 컨테이너의 관계를 알아야 한다.

- Servlet Container: HTTP 요청 수신, Servlet 실행
- Spring Container: Controller, Service, Repository 같은 Spring Bean 관리

Spring MVC 초기화는 이 둘을 연결하는 과정이다.

---

## 2. 전통적인 Spring MVC 초기화

Servlet 3.0 이후에는 `web.xml` 없이 Java 코드로 Servlet Container를 초기화할 수 있다.

이때 사용되는 표준이 `ServletContainerInitializer`이다.

Spring은 이를 활용해 `SpringServletContainerInitializer`를 제공하고, `WebApplicationInitializer` 구현체를 찾아 실행한다.

개발자는 `WebApplicationInitializer`에서 다음 작업을 할 수 있다.

- Root ApplicationContext 생성
- DispatcherServlet용 Web ApplicationContext 생성
- DispatcherServlet 등록
- Servlet Mapping 설정

---

## 3. ContextLoaderListener와 Root WebApplicationContext

전통적인 Spring MVC 구조에서는 Root WebApplicationContext와 Servlet WebApplicationContext를 구분할 수 있다.

## Root WebApplicationContext

Service, Repository 등 웹 기술과 직접 관련 없는 공통 Bean을 담는다.

## Servlet WebApplicationContext

Controller, HandlerMapping, HandlerAdapter, ViewResolver 등 Spring MVC 관련 Bean을 담는다.

`ContextLoaderListener`는 Root WebApplicationContext를 초기화하는 역할을 한다.

DispatcherServlet은 자신의 WebApplicationContext를 사용해 MVC 관련 Bean을 찾는다.

---

## 4. Spring Boot에서의 초기화

Spring Boot는 내장 WAS를 사용하고, 많은 MVC 설정을 자동으로 구성한다.

개발자가 직접 DispatcherServlet을 등록하지 않아도 Spring Boot가 자동으로 등록한다.

중요한 자동 설정 클래스는 다음과 같다.

- `DispatcherServletAutoConfiguration`
- `WebMvcAutoConfiguration`
- `WebMvcProperties`

`DispatcherServletAutoConfiguration`은 DispatcherServlet을 Bean으로 등록하고 Servlet Container에도 등록한다.

`WebMvcAutoConfiguration`은 Spring MVC에 필요한 HandlerMapping, HandlerAdapter, ViewResolver 등의 기본 설정을 제공한다.

---

## 5. @EnableWebMvc 사용 시 주의

Spring Boot에서는 보통 `@EnableWebMvc`를 직접 사용하지 않는다.

`@EnableWebMvc`를 사용하면 Spring Boot의 WebMvc 자동 설정이 꺼질 수 있다.

그 결과 정적 리소스 처리, 메시지 컨버터, 포맷터 등 Spring Boot가 제공하던 기본 설정을 직접 구성해야 할 수 있다.

따라서 대부분의 경우에는 `WebMvcConfigurer`를 구현해 필요한 부분만 추가하는 것이 좋다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        // 인터셉터 추가
    }
}
```

---

## 6. DispatcherServlet 개요

DispatcherServlet은 Spring MVC의 핵심 Front Controller이다.

모든 HTTP 요청을 중앙에서 받아 요청 처리 흐름을 조정한다.

DispatcherServlet이 직접 모든 작업을 처리하는 것은 아니다.

대신 여러 전략 인터페이스와 협력한다.

- HandlerMapping
- HandlerAdapter
- HandlerExceptionResolver
- ViewResolver
- LocaleResolver
- MultipartResolver

이 구조 덕분에 Spring MVC는 다양한 Controller 방식과 View 기술을 유연하게 지원할 수 있다.

---

## 7. Spring MVC 기본 요청 흐름

Spring MVC 요청 흐름은 다음과 같이 정리할 수 있다.

```text
Client
→ DispatcherServlet
→ HandlerMapping
→ HandlerAdapter
→ Controller
→ ModelAndView 또는 Response Body
→ ViewResolver
→ View
→ Response
```

단계별 역할은 다음과 같다.

## 1단계: 요청 수신

Servlet Container가 HTTP 요청을 받고 DispatcherServlet을 호출한다.

## 2단계: Handler 조회

DispatcherServlet이 HandlerMapping을 통해 요청을 처리할 Handler를 찾는다.

## 3단계: Handler 호출

DispatcherServlet은 HandlerAdapter를 통해 Handler를 실행한다.

## 4단계: 결과 처리

Handler 실행 결과로 ModelAndView, View 이름, ResponseBody 등이 반환된다.

## 5단계: View 처리

View 응답인 경우 ViewResolver가 View 이름을 실제 View로 변환하고 렌더링한다.

---

## 8. HandlerMapping

HandlerMapping은 요청 URL, HTTP 메서드, 헤더, 파라미터 등의 조건을 보고 실행할 Handler를 찾는다.

예를 들어 `@RequestMapping`, `@GetMapping`이 붙은 Controller 메서드가 Handler가 될 수 있다.

HandlerMapping은 단순히 Handler만 반환하는 것이 아니라, 인터셉터 목록을 포함한 HandlerExecutionChain을 반환할 수 있다.

즉, 실제 요청 처리 전후에 실행될 공통 로직까지 함께 관리할 수 있다.

---

## 9. HandlerAdapter

HandlerAdapter는 Handler를 실제로 호출하는 역할을 한다.

Spring MVC는 다양한 형태의 Handler를 지원한다.

- 애노테이션 기반 Controller
- HttpRequestHandler
- SimpleControllerHandlerAdapter 기반 Controller

DispatcherServlet이 모든 Handler 호출 방식을 직접 알면 확장성이 떨어진다.

그래서 HandlerAdapter가 중간에서 Handler 호출 방식을 맞춰준다.

즉, HandlerAdapter는 “어떤 방식의 Handler든 DispatcherServlet이 일관되게 호출할 수 있게 해주는 어댑터”이다.

---

## 10. ViewResolver와 View

Controller가 View 이름을 반환하면 DispatcherServlet은 ViewResolver를 통해 실제 View 객체를 찾는다.

예를 들어 다음 설정이 있다고 하자.

```properties
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

Controller가 `hello`를 반환하면 실제 View 경로는 다음처럼 만들어질 수 있다.

```text
/WEB-INF/jsp/hello.jsp
```

View는 Model 데이터를 사용해 최종 HTML 응답을 렌더링한다.

---

## 11. 보충 정리

Spring MVC를 학습할 때는 Controller 애노테이션 사용법만 외우기 쉽다.

하지만 문제가 생겼을 때는 내부 흐름을 알아야 원인을 찾을 수 있다.

예를 들어 다음 질문에 답할 수 있어야 한다.

- 요청을 처리할 Controller는 누가 찾는가?
- Controller 메서드는 누가 호출하는가?
- 파라미터 값은 누가 넣어주는가?
- 반환값은 누가 해석하는가?
- View 이름은 어떻게 실제 View가 되는가?

이 질문의 답이 DispatcherServlet, HandlerMapping, HandlerAdapter, ArgumentResolver, ReturnValueHandler, ViewResolver와 연결된다.

---

## 12. 오늘 정리

Spring MVC의 핵심은 DispatcherServlet을 중심으로 한 Front Controller 구조이다.

중요한 정리는 다음과 같다.

- Spring MVC는 Servlet Container 위에서 동작한다.
- Spring Boot는 DispatcherServlet과 MVC 기본 구성을 자동 설정한다.
- `@EnableWebMvc`를 직접 사용하면 Boot 자동 설정 일부가 비활성화될 수 있다.
- 필요한 MVC 설정은 보통 `WebMvcConfigurer`로 추가한다.
- DispatcherServlet은 요청 처리 흐름을 조정하는 Front Controller이다.
- HandlerMapping은 요청에 맞는 Handler를 찾는다.
- HandlerAdapter는 Handler를 실행한다.
- ViewResolver는 View 이름을 실제 View로 변환한다.
- Spring MVC 내부 구조는 전략 패턴과 어댑터 패턴을 적극적으로 활용한다.
