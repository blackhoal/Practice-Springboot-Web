# 2-1. 테스트 코드 소개

## TDD 개요
- 항상 실패하는 테스트를 먼저 작성(Red)
- 테스트가 통과하는 프로덕션 코드를 작성(Green)
- 테스트 통과 시 프로덕션 코드를 리팩토링(Refactor)

## TDD와 단위 테스트의 차이  
- 단위 테스트는 TDD의 첫 번째 단계인 기능 단위의 테스트 코드를 작성하는 것과 동일

## 단위 테스트의 필요성
- 개발 초기의 문제 발견에 용이
- 개발자가 나중에 코드 리팩토링이나 라이브러리 업그레이드 등에서 기존 기능의 올바른 작동을 확인 가능
- 기능에 대한 불확실성 감소
- 시스템에 대한 문서로 사용 가능

# 2-2. Hello Controller 테스트 코드 작성
1. Application.java
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
- `@SpringBootApplication`
  - 스프링부트의 자동 설정, 스프링 Bean 읽기와 생성을 모두 자동으로 설정
  - 해당 위치부터 설정을 읽어가므로 해당 클래스는 항상 프로젝트의 최상단에 위치
- `SpringApplication.run`
  - 내장 WAS를 실행
  - 내장 WAS를 사용 시 언제 어디서나 같은 환경에서 스프링 부트를 배포할 수 있으므로 권장

2. HelloController.java
```java
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/hello/dto")
    public HelloResponseDto helloDto(@RequestParam("name") String name, @RequestParam("amount") int amount) {
        return new HelloResponseDto(name, amount);
    }
}
```
- `@RestController`
  - 해당 컨트롤러를 JSON을 반환하는 컨트롤러로 변환
- `@GetMapping`
  - HTTP Method인 Get의 요청을 받을 수 있는 API로 변환
- `@RequestParam`
  - 외부에서 API로 넘긴 파라미터를 가져오는 어노테이션
  - 외부에서 넘긴 파라미터를 메소드 파라미터에 저장

3. HelloControllerTest.java
```java
package com.jason.book.springboot;

import com.jason.book.springboot.web.HelloController;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.ResultActions;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }
}
```
- `@RunWith(SpringRunner.class)`
  - 스프링 부트 테스트와 JUnit 사이의 연결자 역할
  - 테스트를 진행 시 JUnit에 내장된 실행자 외에 다른 실행자를 실행
- `@WebMvcTest`
  - Web(Spring MVC)에 집중 가능한 어노테이션
  - @Controller, @ControllerAdvice 사용 가능
  - @Service, @Component, @Repository 사용 불가능
- `@Autowired`
  - 스프링이 관리하는 빈을 주입 
- `private MockMvc mvc`
  - 웹 API를 테스트 시 사용
  - 스프링 MVC 테스트의 시작점
  - 해당 클래스를 통해 GET, POST등에 대한 API 테스트 가능
- `mvc.perform(get("/hello"))`
  - MockMvc를 통해 해당 주소로 GET 요청
  - 체이닝이 지원되어 여러 검증 기능을 이어서 선언 가능
- `.andExpect(status().isOk())`
  - HTTP Header의 Status를 검증
- `.andExpect(content().string(hello))`
  - 응답 본문의 내용을 검증
