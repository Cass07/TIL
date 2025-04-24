#### WAS란
- WAS(Web Application Server)는 웹 애플리케이션을 실행하고 관리하는 서버 소프트웨어
- JAVA에서의 WAS
  - JAVA에서의 웹 프로그래밍 기술은 Servlet 으로 이루어져 있기 때문에
  - 이 서블릿 기술 규격을 사용해서, Http Request를 받아서 Servlet 객체를 생성하고, doGet/doPost를 실행해 실제 비즈니스 로직을 실행시키는 것이 바로 서블릿 컨테이너
  - Servlet을 사용한 웹 애플리케이션의 WAS란 바로 서블릿 컨테이너와 같다
- Spring의 WAS
  - Apache Tomcat, Jetty, Undertow 등이 있다
  - Spring Boot의 내장 WAS는 Apache Tomcat임
- 순수 자바에서 톰캣을 구축할 때
  - 톰캣 객체를 생성하고 포트 설정
  - 컨텍스트 추가
  - 서블릿 생성
  - 톰캣에 서블릿 추가
  - 컨텍스트에 서블릿 매핑
  - 톰캣 실행, 호출 대기

- Spring Boot에서 구축
  - 자동으로 진행됨
  - ServletWebServerFactoryAutoConfiguration 클래스에서 서블릿 웹 서버를 자동으로 생성
  - DispatcherServletAutoConfiguration 클래스에서 DispatcherServlet의 생성과 등록을 실행

- 서블릿 컨테이너는 추상화되어 있어서 다른 WAS를 사용할 수 있도록 하기 위해서, 서블릿의 생성과 서블릿 컨테이너의 생성은 별도로 분리되어 있다


#### 서블릿의 동작 플로우
- 클라이언트가 HTTP Request
- Servlet Container가 Request를 받음
  - HttpServletRequest, HttpServletResponse 객체를 생성
- web.xml을 기반으로, 요청이 어느 서블릿 객체를 필요로 하는지 찾음
- 찾은 서블릿 객체의 service 메소드를 호출하고, doGet/doPost를 호출
- 해당 메소드는 동적 페이지를 생성하고, HttpServletResponse 객체에 응답을 보냄
- 응답이 끝나면 두 객체를 소멸시킴

#### 서블릿 컨테이너란
- 웹 서버 통신 지원
- 서블릿 생명주기 관리
- 멀티스레드 지원 및 관리
  - 요청이 올 떄마다 알아서 스레드를 생성하고 없애줌
- 선언적 보안 관리
  - xml등에 서블릿의 보안 설정을 해 두면 이를 통해 인증/인가를 해줌

