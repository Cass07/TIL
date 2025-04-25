#### CORS
- Cross-Origin Resource Sharing
- 기본적으로 브라우저는 다른 도메인의 리소스에 접근하는 것을 막음
  - 왜냐하면 보안적으로 문제가 되니까 (XSS 같이 해커가 다른 리소스로 접근하도록 바꿔치웠을때 못막기)
  - 이를 위한 정책이 바로 동일 출처 정책 (same origin policy)
    - 프로토콜, 도메인, 포트가 같아야지만 허용해줌
- 동일 출처 정책을 우회하면서, 안전하게 다른 도메인의 리소스에 접근하기 위한 것이 바로 CORS

#### CORS의 작동 원리

- 다른 도메인에 리소스를 요청할 때, 자신의 정보를 헤더에 담아서 보낸다
  - `Origin`에다가 도메인과 포트를 보낸다
- 받은 서버는 응답 헤더에 리소스에 접근 가능한 url을 보내준다
  - `Access-Control-Allow-Origin`
- 클라이언트가 `Origin`과 `Access-Control-Allow-Origin`의 값이 같은지 확인한다
  - 같으면 문제없이 가져온다
  - 다르면 CORS 에러가 발생하며 해당 응답을 버린다


#### 세부 작동 시나리오
1. 프리 플라이트 요청
   - 요청 보내기 전에 먼저 OPTIONS 메소드로 예비 요청을 보내서 서버에서 허용하는 Origins와 Method, Header를 받아서 확인한다
   - 해당 요청이 안전하다고 판단되면 실제 요청을 보낸다
2. 단순 요청
   - 예비 요청 없이 바로 호출하고, 헤더에서 Origin이 허용되는 건지 확인해서 사용 여부를 결정하는 방법
3. 인증된 요청
   - 클라이언트에서 호출할 때 인증 정보를 헤더에 담아서 보내는 것
   - 다른 방법과는 다르게 서버에서 `Access-Control-Allow-Credentials` 헤더를 true로 보내준다
     - Credential request일때 이 헤더를 설정하지 않으면 브라우저에서 CORS 에러를 던지기 때문
   - 또한 Origins, Method, Headers에 와일드카드를 설정할 수 없는 점이 다르다고 한다


#### CORS 에러를 막기 위한 방법들
1. 서버에서 Access-Control-Allow-Origin 헤더를 설정해서 보내준다
2. API 호출을 프록시 서버를 거친다
   - 원래 API 서버가 Origin을 허용하지 않으니, 모든 Origin을 허용하는 서버에서 대신 aPI를 호출해서 그 결과를 보내주는 방법
   - 프록시 서버는 모든 Origin을 허용하니까, CORS 에러가 발생하지 않는다