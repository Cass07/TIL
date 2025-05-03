#### 비동기/논블로킹 API?
- API 자체는 동기적으로 만들면서, 내부 비즈니스 로직에서만 비동기적으로 처리할수도 있고
  - 이 경우 밖에서 바라볼땐 차이점을 모르겠지만
- API 자체를 비동기적으로 만들어서 콜백까지 서버가 담당할수도 있고
- 그냥 논 블로킹하게 만들어서 호출하면 호출 성공여부와 작업 완료를 확인할 수 있는 엔드포인트를 리턴해줄수도 있다
  - 처리에 시간이 오래 걸리는 경우 (파일 업로드 등등..) 사용하는 경우를 많이 봄


#### 일단 Spring Boot에서의 Async 
- Spring Boot 에서 비동기 프로그래밍을 지원하는 기능
  - 병렬 처리, 스레드 풀 등의 개념을 같이 활용
- JAVA에서의 스레드 풀 생성/작업 과정
  - ExecutorService 인터페이스를 통해 스레드 풀 미리 생성
  - submit 메소드를 통해 작업 제출
  - 작업 실행 
  - 작업 완료
  - shutdown 메소드를 통해 스레드 풀 종료 (더 이상 스레드 풀을 사용하지 않을 때)
- 스레드 풀을 사용하는 이유
  - 스레드 풀을 사용하지 않으면 하나의 작업을 시작하고 종료할 떄마다 매 번 새로운 스레드를 생성하고 삭제해야 해서 오버헤드가 생긴다
  - 스레드 수에 제한을 줄 수 없어 시스템에서 가능한 스레드 수 보다 많은 스레드를 생성하면 오류가 발생할 수 있다


#### Spring boot 주요 어노테이션
- `@EnableAsync`
  - 비동기 처리 작업에 대한 설정을 하는데 사용됨
  - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/annotation/EnableAsync.html
  - ThreadPoolTaskExecutor 객체를 생성하고 옵션을 설정한 뒤 이를 리
- `@Async`
  - 실제 비동기로 처리할 메소드에 붙여줌
  - name 패러미터가 있고, 실행할 비동기 스레드의 이름을 의미 (enableAsync에서 설정가능)
  - proxy를 통해서 외부에서 호출될 때 비동기적으로 수행되도록 처리된다고한다
    - 즉 Transactional 어노테이션과 동일한 주의사항을 가진다
  - 리턴값으로는 아무것도 없거나, future 객체를 반환함
    - 아무것도 없을 경우에는 호출자는 즉시 반환되며 실행 제어권 또한 호출자에게 넘겨준다
    - Future 객체를 반환하는 경우 비동기 처리의 응답값을 호출해 준다

##### Spring Async의 Future 과 그 구현체들
- Future
  - 비동기 결과를 나타내는 클래스
  - cancel() - 해당 비동기 메소드의 취소 요청 (실패할수도있다)
  - get() - 비동기 메소드의 처리가 완료될 때까지 기다리고, 이후 값을 반환 (기다리는 블로킹 메소드임에 유의)
  - isCanceled, isDone - 비동기 메소드의 상태를 확인하는 메소드
- ListenableFuture
  - 이름에서 알 수 있듯이 콜백을 등록해서 작업이 완료됐을 때 특정 동작을 실행할 수 있도록 한 Future 인터페이스의 확장
  - Spring 6부터 Deprecated되었고 이하의 CompletableFuture 클래스 사용을 권장
  - addCallback(ListenableFutureCallback<? super T> callback) - 작업 완료 시 콜백등록
  - addCallback(SuccessCallback<> successCall, FailureCallback failCall) - 성공/실패 시의 콜백 등록
  - completable() - 해당 객체를 CompletableFuture로 반환
- CompletableFuture
  - Future의 확장 클래스고, 작업을 체이닝하거나 작업 완료 후 특정 메소드를 실행하도록 할 수 있다
  - 비동기 작업의 결과를 조합하거나, 체이닝을 통해 연속적으로 비동기 작업을 수행할 때 사용
  - https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/concurrent/CompletableFuture.html

##### Executor
- 실제 비동기 작업을 처리하는 객체
  - 작업을 받아 스레드를 생성하던 스레드 풀에서 가져오건 해서 비동기적 처리를 관리한다
- 구현체들 종류
  - Executor
    - 각 작업마다 새로운 스레드를 생성하여 진행
    - 스레드 풀 사용 X
  - ThreadPoolTaskExecutor
    - 스레드 풀을 사용하여 비동기 작업을 처리
  - ScheduledThreadPoolExecutor
    - 작업을 일정 시간 후, 혹은 주기적으로 처리
  - ForkJoinPool
    - 작업을 작은 단위로 분할해서 병렬 처리 후 다시 합침
    - Work-Stealing 알고리즘을 사용
      - 작업을 쪼개서 여러 스레드에 나눠주고, 모든 작업을 끝낸 스레드는 다른 스레드에 남아 있는 작업을 가져와서 대신 처리함


#### 참고
- https://khdscor.tistory.com/132
- https://adjh54.tistory.com/544
- https://adjh54.tistory.com/547