#### Transaction AOP
* Transactional 붙이면 프록시 등을 통해서 실제로는 다음과 같이 실행된다고 보면된다
```java
@Override
public void someTransaction() {
    Session s = sessionFactory.openSession();
    Transaction tx = s.beginTransaction();
    // 실제 비즈니스 로직
    super.someTransaction();
    ts.commit();
}
```

#### OpenSession? GetCurrentSession?
* OpenSession : sessionFactory에서 새로 세션을 만들어서 가져온다
* GetCurrentSession: 현재 열려 있는 세션을 가져온다
* transactional을 사용하면 알아서 세션을 만들어주고 커밋하기 때문에 내부에서 세션에 접근이 필요하면 getCurrentSession을 쓰라고한다

#### Connection pool library, 장단점
* 기본으로 제공하기도 함
* C3P0 : 느린 편이지만 다중 스레드에서는 유리하다고 함
* DBCP : 옛날에는 데이터베이스 커넥션이 끊어지면 다시 연결 복구를 못 한다는 듯함 (단 최근에 프로젝트가 다시 활발하게 업데이트 중이라고함)
* Proxool
* hikaricp
  * 스프링 부트 2.0부터 기본 
  * 다른 CP들보다 월등하게 빠름
    * 바이트코드를 최적화
    * 프록시와 인터셉터 최적화
    * FastList를 구현해 더 빠르게 배열 탐색 (배열의 끝에서부터 순환하며 검색하기 때문에 제거되는 요소가 끝에 몰려있을 떄 더 유용하다고 함)
    * 커넥션을 관리할 때 ConcurrentBag이라는 자체 구현체를 사용
      * ThreadLocal을 사용해서 각 스레드마다 커넥션을 관리하고, threadlocal에 사용 가능한 항목이 없을 때만 공통 컬렉션을 조회
        * ThreadLocal : 스레드 단위의 로컬 변수를 제공하는 클래스
      * `AbstractQueuedLongSynchronizer`를 사용해서 'lock-less' 하게 작동하도록 함
      * [ConcurrentBag javadoc](https://www.javadoc.io/doc/com.zaxxer/HikariCP/2.6.1/com/zaxxer/hikari/util/ConcurrentBag.html)
    * 코드 사이즈 최소화