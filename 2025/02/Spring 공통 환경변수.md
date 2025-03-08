### 여러 Spring 애플리케이션에서 공통적인 환경 변수를 참조하게 하는 법?
- 여러 스프링 애플리케이션에서, 공통적으로 사용하는 환경 변수값이 존재할 경우, 이를 공통으로 관리할 필요가 있음
- 새로 빌드해서 배포하는 방식이 아닌 방식으로 갱신할 수 있다면 좋음
- 메인 RDBSM을 사용하지 않다면 더 좋음
- 구글링했을 때 나오는 키워드
  - Zookeeper
  - AWS Secret Manager
  - Confluent (컨플루언스 아님)

#### 우선 스프링(부트)에서 환경 변수를 로드하고, 참조하는 그 과정은?
- Spring Boot의 SpringApplication.run() 메소드를 통해 애플리케이션을 실행할 때, 환경 변수를 로드하고 참조하는 과정이 있음
  - public ConfigurableApplicationContext run(String... args) (스프링 부트에 관련된 설정을 로드하고, ApplicationContext를 생성하는 메소드)
```java
public ConfigurableApplicationContext run(String... args) {
    long startTime = System.nanoTime();
    DefaultBootstrapContext bootstrapContext = this.createBootstrapContext();
    ConfigurableApplicationContext context = null;
    this.configureHeadlessProperty();
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting(bootstrapContext, this.mainApplicationClass);

    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
        Banner printedBanner = this.printBanner(environment);
        context = this.createApplicationContext();
        context.setApplicationStartup(this.applicationStartup);
        this.prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), timeTakenToStartup);
        }

        listeners.started(context, timeTakenToStartup);
        this.callRunners(context, applicationArguments);
    } catch (Throwable var12) {
        this.handleRunFailure(context, var12, listeners);
        throw new IllegalStateException(var12);
    }

    try {
        Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
        listeners.ready(context, timeTakenToReady);
        return context;
    } catch (Throwable var11) {
        this.handleRunFailure(context, var11, (SpringApplicationRunListeners)null);
        throw new IllegalStateException(var11);
    }
}
```
- ConfigurableEnvironment environment = this.prepareEnvironment(listeners, bootstrapContext, applicationArguments)

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners, DefaultBootstrapContext bootstrapContext, ApplicationArguments applicationArguments) {
    ConfigurableEnvironment environment = this.getOrCreateEnvironment(); // 1.1
    this.configureEnvironment((ConfigurableEnvironment)environment, applicationArguments.getSourceArgs()); // 1.2
    ConfigurationPropertySources.attach((Environment)environment); // 1.3
    listeners.environmentPrepared(bootstrapContext, (ConfigurableEnvironment)environment); // 1.4
    DefaultPropertiesPropertySource.moveToEnd((ConfigurableEnvironment)environment); // 1.5
    Assert.state(!((ConfigurableEnvironment)environment).containsProperty("spring.main.environment-prefix"), "Environment prefix cannot be set via properties."); // 1.6
    this.bindToSpringApplication((ConfigurableEnvironment)environment); // 1.7
    if (!this.isCustomEnvironment) { // 1.8
        environment = this.convertEnvironment((ConfigurableEnvironment)environment);
    }

    ConfigurationPropertySources.attach((Environment)environment); // 1.9
    return (ConfigurableEnvironment)environment;
}
```

- 조건에 맞는 property source를 찾아서 등록하고, 이를 가지고 스프링부트에서 사용할 environment를 생성하고 초기화하는 과정을 거친다
- 즉, 스프링 부트 애플리케이션을 부트하는 과정에서 환경변수가 등록되기 때문에, 환경 변수를 변경하기 위해서는 스프링 애플리케이션의 리부트가 필요하다
  - 즉, 애플리케이션이 참조할 환경 변수 파일을 수정하면, 이후 pod의 종료 후 재생성 자체는 필요할 것으로 생각됨
  - 간단하게 rollout시키면 될 것으로 생각함
    - `kubectl rollout restart deployment <deployment-name>`


#### 런타임에서 환경 변수를 변경하기?
- EnvironmentPostProcessor 을 사용해서, 애플리케이션 시작 시점에서 환경 설정 값을 동적으로 불러와서 변경하거나 추가할 수 있다고 함
  - 기본 환경변수 설정이 완료되고, 그 뒤에 오버라이드한 postProcessEnvironment 메소드가 실행되니까, 거기서 설정한 환경변수를 덮어쓰는 방법임
- Spring 환경에서는 (Spring Boot를 사용하지 않는다면), ApplicationContextInitializer 를 사용하면 된다고 함
- EnvironmentPostProcessor : environment context가 생성되기 전에 먼저 실행되어 애플리케이션을 원하는 대로 설정할 수 있게 해줌
- 해당 클래스를 구현한 뒤에는 `META-INF/spring.factories` 파일에 등록해주어야 함
  - `org.springframework.boot.env.EnvironmentPostProcessor=구현한.객체의.full.패키지.경로`

```java
// systemEnvironment에 있는 환경변수를 읽어 prefix를 붙여서 새로운 환경변수로 등록하는 예시
public class PriceCalculationEnvironmentPostProcessor implements EnvironmentPostProcessor {

    private static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
    private static final List<String> names = Arrays.asList("config.key1", "config.key2"); // 예시 키 리스트

    @Override
    public void postProcessEnvironment(ConfigurableEnvironment environment, SpringApplication application) {
        var system = environment.getPropertySources().get(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME);
        // names의 키에 해당하는 property의 value와, prefix를 붙여서 새로운 환경변수 map을 만듬
        Map<String, Object> prefixed = names.stream()
            .collect(Collectors.toMap(this::rename, system::getProperty));
        // addAfter를 통해서, systemEnvironment 다음에 아까 만든 새로운 환경변수를 추가
        environment.getPropertySources()
            .addAfter(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, new MapPropertySource("prefixer", prefixed));
    }

    private String rename(String originalName) {
        return "prefix." + originalName; // 원하는 방식으로 키 이름 변경
    }
}
```

#### 외부 설정 파일 import
```yaml
spring:
  config:
    import: file:/path/to/your/config/application.yml
```
- jar/war 파일의 외부에 config 파일을 위치시키는 방법

#### Docker 이미지에서 애플리케이션을 실행하기 전 외부 환경 변수 파일을 받기
- 빌드한 후, 도커 이미지가 실행될 때 curl 등을 통해서 환경 변수 파일을 받아오면 안 되나?
  - 서버 리소스를 최대한 덜 잡아먹기 위해서, github action을 통해서 빌드하고 거기서 docker image까지 빌드해서 올리는 상태
- 환경 변수 파일에 대한 별도의 접근 권한 설정 및 관리가 필요한다

```dockerfile
FROM openjdk:21-jdk-slim
# curl 설치
RUN apt-get update \
 && apt-get install -y curl \
# JAR 파일 복사
COPY ./jar-file-name.jar ./app.jar

# curl을 통해 환경 변수 파일을 받은 후, java -jar 실행
ENTRYPOINT ["sh", "-c"]
CMD["curl -o ./path/to/your/config/application.yml https://your-config-server/application.yml & java -jar -Dspring.other.environment=test /app.jar"]
```

- 정리
  - RUN, COPY.. 등등의 명령어는 이미지 빌드 시 실행됨
    - 마지막 명령어가 나오기 전까지 한 줄씩 실행해서 도커 이미지에 커밋을 반복함
  - Entrypoint, CMD 등의 명령어는 이미지가 실행될 때 실행되는 명령어
    - Entrypoint는 이미지 실행 시 무조건 실행
    - CMD는 Entrypoint가 있을 때, Entrypoint의 인자로 실행됨
    - CMD는, 이미지 실행 시 추가 인자값을 받으면 덮어쓰게 됨
  - 모든 명령어는 도커 이미지의 레이어가 된다
- 이 경우에는 로컬에서 테스트할 때 환경변수 따로 갱신해줘야 한다는 단점이 있긴함 (github action 등으로 자동화하면 되긴함)

#### 키 관리 서비스 이용
- Zookeeper, AWS secret manager, Hashicorp Vault 등의 서비스를 이용해서 환경 변수를 관리할 수 있음
  - 서비스가 알아서 환경변수를 안전하기 관리하므로, 유출의 위험성이 상대적으로 적다
  - 단 서비스 운영/사용 비용이 추가로 발생할 수 있다
    - AWS secret manager 기준으로 보안 정보 1개당 0.4\$, 10000번의 호출 당 0.05\$의 비용이 발생
##### AWS Secret manager를 사용해서, Docker 환경의 Spring 애플리케이션의 환경변수 관리
- AWS CLI를 통해서 secret manager의 key-value 갱신/가져오기 가능
  - `aws secretsmanager get-secret-value --secret-id <secret-id>`
- 혹은, JAVA의 AWS Secret Manager SDK를 사용해서 가져오기 가능
  - 조회한 시크릿은 캐싱을 통해 직접 조회 빈도를 줄일 수 있고 속도 또한 개선 가능
```java
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import com.amazonaws.services.lambda.runtime.LambdaLogger;

import com.amazonaws.secretsmanager.caching.SecretCache;

public class SampleClass implements RequestHandler<String, String> {

     private final SecretCache cache  = new SecretCache();

     @Override public String handleRequest(String secretId,  Context context) {
         final String secret  = cache.getSecretString(secretId);

        // Use the secret, return success;

    }
}
```

#### 즉, 가능한 시나리오

- 환경 변수 파일이 있는 EFS 등의 스토리지를 POD에 마운트하고, 거기서 환경 변수 파일을 외부 설정 파일로서 참조하여 애플리케이션 실행
- 환경 변수 파일을 도커 이미지 실행 시 다운로드하고, 이를 외부 설정 파일로서 참조하여 이미지 실행
- 키 관리 서비스를 사용
  - 컨테이너 실행 시 키 관리 서비스를 호출하여 환경 변수를 가져온 뒤 이를 설정 파일로 생성하여 참조, 애플리케이션 실행
  - 애플리케이션 런타임에서 SDK를 사용해 키 관리 서비스에 접근하여 환경 변수를 가져오고, 가능하다면 캐싱하기