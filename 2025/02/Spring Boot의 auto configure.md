### 스프링부트란
- 스프링 프레임워크의 복잡한 설정 과정을 간소화하여 개발자가 애플리케이션의 비즈니스 로직 개발에만 집중할 수 있도록 해 주는 프레임워크
- 스프링이란 자바 기반의 애플리케이션 프레임워크
  - POJO, AOP, DI, IoC 등의 개념을 지원해, 개발자가 객체 지향적으로 프로그래밍할 수 있도록 도와준다
  - Bean을 통해 객체의 생성과 소멸을 프레임워크가 직접 관리

- 핵심 기능
  1. Auto Configuration: 애플리케이션의 설정을 자동화해, 개발자가 수동으로 XML이나 Java Config 파일 등을 작성해야 하는 번거로움을 줄여 준다
  2. Embedded Server: Spring 사용 시에는 Tomcat 등의 별도의 웹 애플리케이션 서버 (WAS)를 설치해서 실행해야 하나, 이를 내장하여 별도의 웹 서버 설치 없이 바로 실행할 수 있게 해 준다
  3. Starter Dependency: 특정 기능에 필요한 라이브러리를 묶어서 제공하는 스타터 의존성을 제공해, 프로젝트에 필요한 의존성을 자동으로 추가해 주며, 버전 관리 또한 쉽게 해 준다
  4. Production Ready Features: 애플리케이션의 상태를 모니터링하는 Actuator를 제공하고, 로깅, 보안 설정 등의 기능을 제공한다

#### Auto Configuration
- `@EnableAutoConfiguration` 어노테이션을 통해 활성화된다
- `AutoConfigurationImportSelector` 클래스를 통해 자동 설정 클래스를 찾아서 빈으로 등록한다
  - 등록할 빈이 정의된 클래스의 정보는 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 폴더 내의 파일에 정의되어 있다

- 즉 애플리케이션 실행 시 다음과 같이 빈이 등록된다
  - ImportSelector를 구현한 AutoConfigurationImportSelector가 실행된다
  - `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 파일 내의 AutoConfigure 객체를 로드한다
  - 객체를 읽어 유효하다면 설정된 빈을 등록한다
    - 유효한 빈인지는 어떻게 암??? 컨디셔널 클래스 찾아보기
    - Spring Autoconfiguration Conditional