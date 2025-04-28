#### 웹 소켓이란
- 클라이언트와 서버 간 연결을 유지해서, 양방향 통신이나 데이터 전송을 가능하게 하는 기술
- 실시간으로 통신이 필요한 Realtime Web Application에서 사용

- HTTP와의 차이점
  - HTTP는 클라이언트가 요청을 보내고, 서버가 응답을 보내는 방식으로 통신
  - 웹 소켓은 클라이언트와 서버 간 연결을 유지해서, 양방향 통신 가능
    - 클라이언트에서 매번 HTTP 프로토콜을 통해 서버를 호출할 필요가 없다

##### 특징
- 처음에는 HTTP 프로토콜을 통해 연결됨
  - 이후에는 web socket 프로토콜로 연결을 유지함
  - 연결이 끊어졌을 경우 적절한 대응이 필요할 수 있음
- Statefull 프로토콜임
  - 지속적으로 연결이 유지되기 때문에 변경 사항은 실시간으로 적용됨
- HTTP와 동일한 80port를 사용


#### Spring Boot에서 WebSocket

- Spring Boot에서 WebSocket을 사용하기 위해서는, `spring-boot-starter-websocket` 라이브러리를 사용함
- WebSocketSession 객체를 사용해서 웹소켓 통신을 한다
  - 클라이언트와 서버 간의 연결을 나타내는 객체
- WebSocketHandler를 구현해서, 클라이언트와 서버 간의 메시지를 처리함
  - 보통 예제로 자주 구현하는 텍스트만을 전송하는 websocket 통신의 경우, 구현체 중 하나인 TextWebSocketHandler를 상속받아서 구현함
    - 이외에도 BinaryWebSocketHandler, PingWebSocketHandler 등도 있음
    - https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/socket/WebSocketHandler.html

- WebSocketHandler 인터페이스
  - afterConnectionEstablished : 클라이언트와 연결되어 WebSocket 연결이 사용될 준비가 완료되면 호출
  - handleMessage : 새로운 WebSocket 메시지가 전달되면 호출
  - handleTransportError : WebSocket 메시지 전송 시 오류가 발생하면 호출
  - afterConnectionClosed : 클라이언트/서버 중 한 쪽에서 WebSocket 연결이 종료되거나 전송 오류가 발생되면 호출
  - supportsPartialMessages : 해당 Handler가 부분 메시지를 지원하는지 여부를 반환

- TextWebSocketHandler의 메서드
  - `afterConnectionEstablished(WebSocketSession session)`: 클라이언트와 연결이 성공적으로 이루어졌을 때 호출됨
  - `handleTextMessage(WebSocketSession session, TextMessage message)`: 클라이언트로부터 메시지를 수신했을 때 호출됨
  - `afterConnectionClosed(WebSocketSession session, CloseStatus status)`: 클라이언트와의 연결이 종료되었을 때 호출됨

- WebSocket 통신은 Asynchronous하게 되니까 동시성 처리를 고려해야 한다