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
- 