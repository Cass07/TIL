
### Spring
- 5.3.x : Spring Boot 2.4.x : JDK 8~21
- 6.0.x : Spring Boot 3.0.x : JDK 17~21
- 6.1.x : Spring Boot 3.1.x~3.3 : JDK 17~23
- 6.2.x : Spring Boot 3.4


#### Spring Boot
- 2.7.x : 2029.6
- 이후 3.0~ 3.4까지는 출시 후 2년간 지원하고
- 3.5 : 2032.6 까지

### JAVA
- LTS별 차이
- 8
  - 날짜, 시간 API 추가
  - Unsigned 정수 지원
  - lambda expression
  - Stream API
- 11
  - G1GC가 기본으로 변경
  - 컬렉션과 스트림에 메소드 추가
- 17
  - record class
    - DTO와 유사하게 사용 가능한 클래스로, getter, setter, equals, hashcode, toString 등을 자동으로 생성
  - Sealed class
    - 상속하거나 구현할 수 있는 클래스를 지정해서 해당 클래스만 상속/구현 가능하도록
  - 난수 생성 api
  - Stream.toList() 추가
    - Stream.collect(Collectors.toList()) 대신 사용 가능
    - toList는 변경 불가능한 리스트 (UnmodifiableList) 를 반환한다는 차이가 있음
    - collect는 변경 불가능함을 보장하지 않음
- 21
  - Sequenced Collection
    - 정해진 순서대로, 혹은 이 역순으로 원소를 조회할 수 있는 컬렉션
  - Virtual Thread
  - Record Pattern
    - 레코드 객체에 타입 패턴을 적용
  - switch 문에 타입 매칭, Null 체크 추가

#### Virtual Thread
- 21부터 지원
- 기존 Java의 Thread Platform Thread로서, OS Thread를 래핑해서 구성한 스레드라, 운영체제의 스레드 수까지만 생성 가능
- Virtual Thread는 OS Thread를 사용하지 않고, JVM 내부에서 스레드를 관리하는 방식
  - OS Thread보다 더 많은 스레드를 생성할 수 있다
  - 스레드의 관리는 Java runtime이 담당하므로, 애플리케이션에서 원하는 대로 관리할 수 있다
  - Virtual 스레드 실행 중 코드가 I/O 블로킹되면, 해당 스레드는 진행을 중지하고 다른 스레드를 실행
  - 그래서 실행시간의 대부분이 I/O 작업인 로직이라면 Virtual 스레드 사용을 통해 동일 시간에 처리할 수 있는 양이 늘어난다
  - 단일 스레드에 대한 처리 속도는 동일하기 때문에, CPU 바운드 작업에 대해서는 Virtual Thread를 사용하는 것이 효과적이지 않다