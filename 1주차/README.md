# 1주차

책참고 : 초보 웹 개발자를 위한 스프링5 프로그래밍 입문 

사이트 : [https://spring.io](https://spring.io) 

예제출처 : https://github.com/madvirus/spring5fs

---

# Spring 기본

## Spring

자바 기반  ‘웹 어플리케이션’  개발 표준 `프레임워크`

### 주요 특징

- 의존 주입 (DI : Dependency Injection) 지원
- AOP(Aspect-Oriented Programming) 지원
- MVC 웹 프레임워크 제공
- JDBC, JPA 연동, 선언적 트랜잭션 처리 등 DB 연동 지원
- 그 외 다양한 기능
    - ex) 스케줄링, 메시지 연동(JMS), 이메일 발송, 테스트 지원

### Spring 관련 프로젝트

- Spring Data
    - 데이터 연동 기능
    - `JPA` , `MongoDB`, `Redis` 등의 저장소 기술 지원
- Spring Security
    - 인증/인가 관련 프레임워크
    - 웹 접근제어, 객체 접근제어 등 다양한 인증방식, `암호화기능` 제공
- Spring Batch
    - 배치 처리에 필요한 기본 기능 제공
    - 로깅/추적, 작업통계, 실패 처리
    

# Spring 기본동작방식

스프링은 객체를 생성하고 초기화 하는 기능을 제공한다. 

### 빈(Bean) 객체

스프링이 생성하는 객체를 `Bean(빈) 객체`라고 부른다. 

빈 객체에 대한 정보를 담고 있는 메서드에는 `@Bean` 어노테이션을 붙여 관리한다.

@Bean 어노테이션을 붙이면, 해당 메서드가 생성한 객체를 스프링이관리하는 빈객체로 등록한다.

### AnnotationConfigApplicationContext

 ApplicationContext라는 인터페이스를 구현한 클래스 중 하나. 

 자바의 main 클래스에서 import되어 사용하는 클래스.

 ‘자바 설정’에서 정보를 읽어와 빈객체를 생성하고 관리한다.

 생성자 파라미터로 전달받은 클래스(AppContext)에서 정의한 @Bean 설정정보를 읽어와 

 객체를 생성하고 초기화 한다. 

EX) Main.java

```java
package chap02;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext ctx = 
				new AnnotationConfigApplicationContext(AppContext.class);
		...
	}
}
```

EX) 클래스 계층도 (Maven 의존 그래프)

![KakaoTalk_20220417_203317345.jpg](1%E1%84%8C%E1%85%AE%E1%84%8E%E1%85%A1%209b6fd/KakaoTalk_20220417_203317345.jpg)

- BeanFactory
    - 가장 상단에 위치한 인터페이스
    - 객체 **생성**과 **검색**에 대한 기능 정의
    - 싱글톤/프로토타입 빈인지 확인하는 기능 정의
    - ex) 생성된 객체를 검색하는 `getBean()` 메서드
- ApplicationContext
    - `컨테이너(Container)`라고도 부름
    - 빈 객체의 생성, 초기화, 보관, 제거 관리
    - 메시지, 프로필/환경변수를 처리할 수 있는 기능을 추가로 정의
- AnnotationConfigApplicationContext
    - 위의 인터페이스를 상속받아 해당 인터페이스에 정의된 기능을 구현

---

# 스프링 DI

## 객체간 의존

A 클래스 내부에서, B클래스의 메서드를 실행할 때, 

“**A 클래스가 B클래스에 의존한다**”고 한다. 

EX) 

```java
package spring;
import java.time.LocalDateTime;

// MemberRegisterService 클래스는 MemberDao 클래스에 의존
public class MemberRegisterService {
	private MemberDao memberDao = new MemberDao(); // 의존 객체를 직접 생성
	...
	
}
```

이처럼 의존 객체를 직접 생성하는 경우, 

변경사항이 있을 시, 해당 객체를 사용하는 곳마다 일일이 찾아 변경해줘야하므로

코드 재활용성이 떨어지고 유지보수 관점에서 문제를 유발한다. 그래서 DI 방식을 사용한다.

## 의존주입 (DI)

 : DI(Dependency Injection) 의존주입

의존하는 객체를 **생성자 파라미터**로 직접 **전달받는** 방식. 

- Unit Test가 용이해짐
- 코드의 재활용성을 높여줌
- 객체간의 의존성을 낮춰줌

EX)

```java
package spring;

import java.time.LocalDateTime;

public class MemberRegisterService {
	private MemberDao memberDao;
	
	// 생성자를 통해 의존객체 MemberDao를 인자로 전달받음
	public MemberRegisterService(MemberDao memberDao) {
				this.memberDao = memberDao; //객체 주입
	}
  ...
}
```

직접 의존객체를 생성하는 대신, 

위처럼 생성자를 통해 의존객체를 전달받아 사용한다. (의존 주입) 

이 때, 의존객체 MemberDao대신 다른 의존객체를 사용하게 되는경우,

MemberRegisterService 내부의 코드를 바꿀 필요없이 다음과 같이 사용가능하다. 

EX)

```java
// 기존의 MemberDao 대신 FixedMemberDao로 변경
MemberDao memberDao = new FixedMemberDao(); 
MemberRegisterService regSvc = new MemberRegisterService(memberDao);
```

---

### 객체조립기(assembler)

**객체를 생성**하고 **의존 객체를 주입**해주는 클래스를 따로 작성해 `조립기(assembler) 클래스` 라고 한다. 스프링이 이러한 객체조립기의 역할을 대신한다. 

## Config 파일 작성

 스프링 설정 클래스 

 스프링의 객체 생성, 의존 주입 방식을 정의한 설정파일 

 보통 config 패키지 하위에 위치

 `@Configuration` 어노테이션을 붙여 구분

EX)

```java
package config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import spring.ChangePasswordService;
import spring.MemberDao;
import spring.MemberRegisterService;

@Configuration
public class AppCtx {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao());
	}
  ...
}
```

 

이 config 설정 클래스를 정의 한 후, 이 클래스를 이용해 스프링 컨테이너를 추가로 생성해야 한다.

Config  클래스를, `AnnotationConfigApplicationContext 클래스`에 생성자 파라미터로 전달해 스프링 컨테이너를 생성한다. 

EX) 사용

```java
// AppCtx config클래스를 생성자 파라미터로 전달 
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppCtx.class);
```
