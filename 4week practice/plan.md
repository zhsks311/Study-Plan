## **🎯 학습 목표**

단순히 기술을 아는 것을 넘어, 각 기술의 **존재 이유(Why)**와 **트레이드오프(Trade-off)**를 이해하고, 대규모 서비스 환경에서 발생할 수 있는 문제점을 예측하고 해결하는 능력을 기릅니다.

## **📚 Spring & Transaction 마스터 8주 커리큘럼**

_10년차 개발자를 위한 이론-실습 통합 심화 과정_

### **🎯 전체 학습 목표**

- Spring Framework 내부 동작 원리 완벽 이해
- 트랜잭션 메커니즘과 분산 환경 처리 능력
- 실무 트러블슈팅과 최적화 역량
- 기술 블로그 8편 + 면접 Q&A 200개 작성

### **📖 학습 구조**

- **일일 학습**: 5시간 (이론 3시간 + 실습 2시간)
- **주간 학습**: 25시간 (5일)
- **전체 기간**: 8주 (200시간)

---

## **Phase 1: Spring Core 완벽 이해 (Week 1-2)**

### **Week 1: IoC Container와 Bean Lifecycle**

#### **Day 1: IoC Container 아키텍처 (5시간)**

**📚 이론 학습 (3시간)**

**1시간: BeanFactory 계층 구조 이해**

```
[핵심 개념]
BeanFactory (최상위 인터페이스)
    ↓
ListableBeanFactory (Bean 목록 조회)
    ↓
HierarchicalBeanFactory (부모-자식 계층)
    ↓
DefaultListableBeanFactory (실제 구현체)

[내부 저장 구조]
- beanDefinitionMap: Bean 정의 메타데이터
- singletonObjects: 완성된 싱글톤 객체
- earlySingletonObjects: 생성 중인 싱글톤
- singletonFactories: 싱글톤 생성 팩토리

[왜 이렇게 복잡한가?]
- 순환 참조 해결
- Lazy 초기화 지원
- 프록시 객체 처리
```

**1시간: ApplicationContext 확장 기능**

```
[BeanFactory vs ApplicationContext]
BeanFactory:
- 지연 로딩 (Lazy Loading)
- 기본적인 DI 기능만 제공

ApplicationContext:
- 즉시 로딩 (Eager Loading)
- 국제화 (MessageSource)
- 이벤트 발행 (ApplicationEventPublisher)
- 리소스 로딩 (ResourceLoader)
- 환경 변수 (Environment)

[실무 선택 기준]
- 99% ApplicationContext 사용
- 메모리가 극도로 제한적인 환경에서만 BeanFactory
```

**1시간: Bean 생성 12단계 프로세스**

```
[상세 단계]
1. BeanDefinition 로딩
2. BeanFactoryPostProcessor 실행
3. Bean 인스턴스 생성
4. 의존성 주입
5. BeanNameAware 콜백
6. BeanFactoryAware 콜백
7. ApplicationContextAware 콜백
8. BeanPostProcessor.postProcessBeforeInitialization
9. @PostConstruct
10. InitializingBean.afterPropertiesSet
11. Custom init-method
12. BeanPostProcessor.postProcessAfterInitialization

[각 단계에서 할 수 있는 일]
- 3단계: 객체 생성 가로채기
- 8단계: 프록시 생성
- 12단계: AOP 적용
```

**🔬 실습: Container 동작 관찰 (2시간)**

**실습 1: BeanFactory 직접 생성과 사용 (30분)**

```java
@Slf4j
public class ContainerTest {
    @Test
    public void BeanFactory_동작_확인() {
        // BeanFactory 직접 생성
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();

        // BeanDefinition 수동 등록
        RootBeanDefinition beanDef = new RootBeanDefinition(MyService.class);
        factory.registerBeanDefinition("myService", beanDef);

        // 디버거 포인트: registerBeanDefinition
        // 관찰: beanDefinitionMap에 저장되는 과정

        MyService service = factory.getBean(MyService.class);

        // 로그로 확인
        log.info("싱글톤 객체 수: {}", factory.getSingletonCount());
        log.info("Bean 정의 수: {}", factory.getBeanDefinitionCount());
    }
}
```

**실습 2: 순환 참조 해결 과정 추적 (45분)**

```java
@Component
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
}

@Component
public class ServiceB {
    @Autowired
    private ServiceA serviceA;
}

// 디버거 브레이크포인트:
// 1. DefaultSingletonBeanRegistry.getSingleton()
// 2. addSingletonFactory()
// 3. earlySingletonObjects 확인

// 로그 설정:
logging.level.org.springframework.beans.factory.support: TRACE

// 관찰 포인트:
// - 3-level 캐시 어떻게 활용되는지
// - 프록시 객체는 언제 생성되는지
```

**실습 3: Bean 생성 생명주기 로깅 (45분)**

```java
@Component
@Slf4j
public class LifecycleBean implements
    BeanNameAware,
    BeanFactoryAware,
    ApplicationContextAware,
    InitializingBean {

    public LifecycleBean() {
        log.info("1. 생성자 호출");
    }

    @Autowired
    public void setDependency(MyDependency dep) {
        log.info("2. 의존성 주입");
    }

    @Override
    public void setBeanName(String name) {
        log.info("3. BeanNameAware: {}", name);
    }

    @Override
    public void setBeanFactory(BeanFactory factory) {
        log.info("4. BeanFactoryAware");
    }

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        log.info("5. ApplicationContextAware");
    }

    @PostConstruct
    public void postConstruct() {
        log.info("6. @PostConstruct");
    }

    @Override
    public void afterPropertiesSet() {
        log.info("7. InitializingBean");
    }
}

// 실행 후 로그 순서 확인
// 각 단계에서 Bean 상태 확인
```

**📝 Day 1 산출물**

- 면접 Q&A 5개: IoC Container 동작 원리
- 실습 로그: Bean 생성 과정 전체 추적

---

#### **Day 2: Bean Scope와 Lazy Loading (5시간)**

**📚 이론 학습 (3시간)**

**1시간: Bean Scope 종류와 구현**

```
[Scope 종류]
1. Singleton (기본값)
   - 애플리케이션당 하나
   - singletonObjects Map에 저장

2. Prototype
   - 요청할 때마다 새로 생성
   - Spring Container가 관리 안 함
   - @PreDestroy 호출 안 됨

3. Request (Web)
   - HTTP 요청당 하나
   - RequestContextHolder에 저장

4. Session (Web)
   - HTTP 세션당 하나
   - SessionScope에서 관리

5. Application (Web)
   - ServletContext당 하나

[실무 함정]
- Singleton → Prototype 주입 문제
- Prototype은 Spring이 파괴 안 함
- Request Scope 테스트 어려움
```

**1시간: Singleton과 Thread Safety**

```
[문제 상황]
@Component
public class UserService {
    private User currentUser;  // 위험!

    public void setUser(User user) {
        this.currentUser = user;
    }
}

[왜 위험한가]
- 모든 스레드가 같은 인스턴스 공유
- Race Condition 발생
- 다른 사용자 정보 노출 위험

[해결 방법]
1. Stateless로 설계
2. ThreadLocal 사용
3. Request Scope 활용
4. 불변 객체 사용
```

**1시간: Lazy Loading 메커니즘**

```
[즉시 로딩 vs 지연 로딩]
즉시 로딩 (기본):
- ApplicationContext 생성 시 모든 싱글톤 생성
- 장점: 시작 시 오류 발견
- 단점: 시작 시간 증가

지연 로딩 (@Lazy):
- 처음 사용할 때 생성
- 장점: 빠른 시작
- 단점: 런타임 오류 가능성

[프록시를 통한 구현]
@Lazy
@Autowired
private ExpensiveService service;
// 실제로는 프록시 주입
// 메서드 호출 시 실제 객체 생성
```

**🔬 실습: Scope 동작 검증 (2시간)**

**실습 1: Singleton vs Prototype 비교 (40분)**

```java
@Component
@Scope("prototype")
public class PrototypeBean {
    private static int instanceCount = 0;
    private final int instanceId;

    public PrototypeBean() {
        this.instanceId = ++instanceCount;
        log.info("PrototypeBean 생성: #{}", instanceId);
    }
}

@Component
public class SingletonBean {
    @Autowired
    private PrototypeBean prototypeBean;  // 문제!

    @Autowired
    private Provider<PrototypeBean> prototypeProvider;  // 해결

    public void usePrototype() {
        // 항상 같은 인스턴스
        log.info("Direct: {}", prototypeBean);

        // 매번 새 인스턴스
        log.info("Provider: {}", prototypeProvider.get());
    }
}

// 테스트
@Test
public void prototype_주입_문제() {
    // SingletonBean을 여러 번 호출
    // prototypeBean은 계속 같은 인스턴스
    // Provider는 매번 새 인스턴스
}
```

**실습 2: Thread Safety 문제 재현 (40분)**

```java
@Component
public class UnsafeService {
    private String data;  // Thread-unsafe

    public void setData(String data) {
        this.data = data;
        try {
            Thread.sleep(100);  // 문제 발생 확률 증가
        } catch (InterruptedException e) {}
    }

    public String getData() {
        return data;
    }
}

@Test
public void 스레드_안전성_테스트() {
    ExecutorService executor = Executors.newFixedThreadPool(10);

    for (int i = 0; i < 10; i++) {
        final String userId = "User" + i;
        executor.submit(() -> {
            unsafeService.setData(userId);
            String result = unsafeService.getData();

            // 다른 스레드의 데이터가 섞임
            if (!userId.equals(result)) {
                log.error("데이터 오염: Expected={}, Actual={}",
                         userId, result);
            }
        });
    }
}
```

**실습 3: Lazy Loading 동작 확인 (40분)**

```java
@Component
@Lazy
@Slf4j
public class ExpensiveService {
    public ExpensiveService() {
        log.info("ExpensiveService 생성 시작");
        // 무거운 초기화 시뮬레이션
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {}
        log.info("ExpensiveService 생성 완료");
    }
}

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        ApplicationContext ctx = SpringApplication.run(Application.class);
        long contextTime = System.currentTimeMillis() - start;

        log.info("Context 로딩 시간: {}ms", contextTime);

        // 이 시점에 ExpensiveService는 아직 생성 안 됨

        long beforeBean = System.currentTimeMillis();
        ctx.getBean(ExpensiveService.class);  // 이제 생성
        long beanTime = System.currentTimeMillis() - beforeBean;

        log.info("Bean 생성 시간: {}ms", beanTime);
    }
}
```

**📝 Day 2 산출물**

- 면접 Q&A 5개: Bean Scope와 Thread Safety
- 실습 보고서: Thread Safety 위반 케이스 3가지

---

#### **Day 3: Dependency Injection 심화 (5시간)**

**📚 이론 학습 (3시간)**

**1시간: DI 방식별 장단점**

```
[Constructor Injection]
장점:
- 불변성 보장 (final 사용 가능)
- 순환 참조 컴파일 시점 발견
- 테스트 용이
- Spring 없이도 객체 생성 가능

단점:
- 보일러플레이트 코드
- 파라미터 많으면 복잡

[Field Injection]
장점:
- 코드 간결
- 편리함

단점:
- 테스트 어려움
- 순환 참조 런타임 발견
- 불변성 보장 안 됨
- Spring 강제 의존

[Setter Injection]
장점:
- 선택적 의존성
- 재설정 가능

단점:
- 불변성 보장 안 됨
- 초기화 시점 불명확

[Spring 팀 권장사항]
- 필수 의존성: Constructor Injection
- 선택적 의존성: Setter Injection
- Field Injection: 테스트에서만
```

**1시간: @Autowired 동작 원리**

```
[AutowiredAnnotationBeanPostProcessor]
1. Bean 생성 후 호출
2. 리플렉션으로 @Autowired 필드/메서드 탐색
3. 의존성 해결 (Type → Name → Qualifier)
4. 주입 실행

[의존성 해결 순서]
1단계: 타입으로 매칭
  - 1개 발견: 주입
  - 0개 발견: NoSuchBeanDefinitionException
  - N개 발견: 2단계로

2단계: @Primary 확인
  - @Primary Bean 있으면 주입

3단계: @Qualifier 확인
  - 일치하는 Qualifier 있으면 주입

4단계: Bean 이름으로 매칭
  - 필드명/파라미터명과 일치하면 주입

5단계: 실패
  - NoUniqueBeanDefinitionException
```

**1시간: Circular Dependency 해결**

```
[3-Level 캐시 메커니즘]
Level 1: singletonObjects
- 완전히 초기화된 Bean

Level 2: earlySingletonObjects
- 생성은 됐지만 초기화 중인 Bean
- 프록시가 필요한 경우 프록시 객체

Level 3: singletonFactories
- Bean을 생성할 수 있는 Factory
- getEarlyBeanReference() 호출

[해결 과정]
A → B → A 순환 참조 시:
1. A 인스턴스 생성 (초기화 전)
2. A를 singletonFactories에 등록
3. A의 의존성 B 처리
4. B 인스턴스 생성
5. B의 의존성 A 필요
6. singletonFactories에서 A 팩토리 발견
7. Early A 참조 획득
8. B 초기화 완료
9. A 초기화 완료

[해결 불가능한 경우]
- Constructor Injection 순환 참조
- Prototype Scope 순환 참조
```

**🔬 실습: DI 메커니즘 검증 (2시간)**

**실습 1: DI 방식별 성능 비교 (40분)**

```java
@Component
public class ConstructorInjection {
    private final ServiceA serviceA;
    private final ServiceB serviceB;
    private final ServiceC serviceC;

    // 생성자 주입
    public ConstructorInjection(ServiceA a, ServiceB b, ServiceC c) {
        this.serviceA = a;
        this.serviceB = b;
        this.serviceC = c;
    }
}

@Component
public class FieldInjection {
    @Autowired
    private ServiceA serviceA;
    @Autowired
    private ServiceB serviceB;
    @Autowired
    private ServiceC serviceC;
}

@Component
public class SetterInjection {
    private ServiceA serviceA;
    private ServiceB serviceB;
    private ServiceC serviceC;

    @Autowired
    public void setServiceA(ServiceA a) {
        this.serviceA = a;
    }
    // ... 나머지 setter
}

// 성능 측정
@Test
public void DI_성능_비교() {
    StopWatch watch = new StopWatch();

    watch.start("Constructor");
    for (int i = 0; i < 1000; i++) {
        ctx.getBean(ConstructorInjection.class);
    }
    watch.stop();

    // Field, Setter도 동일하게 측정

    log.info("결과: {}", watch.prettyPrint());
}
```

**실습 2: @Qualifier 우선순위 테스트 (40분)**

```java
interface PaymentGateway {
    void pay(BigDecimal amount);
}

@Component("paypal")
@Primary
public class PayPalGateway implements PaymentGateway {}

@Component("stripe")
public class StripeGateway implements PaymentGateway {}

@Component
public class PaymentService {
    @Autowired
    private PaymentGateway defaultGateway;  // @Primary 주입

    @Autowired
    @Qualifier("stripe")
    private PaymentGateway stripeGateway;  // @Qualifier 주입

    @Autowired
    private PaymentGateway paypal;  // Bean 이름 매칭

    public void testInjection() {
        log.info("Default: {}", defaultGateway.getClass());
        log.info("Stripe: {}", stripeGateway.getClass());
        log.info("PayPal: {}", paypal.getClass());
    }
}

// 디버거로 DefaultListableBeanFactory.doResolveDependency() 추적
```

**실습 3: Circular Dependency 디버깅 (40분)**

```java
@Configuration
public class CircularConfig {

    @Bean
    public ServiceA serviceA(ServiceB serviceB) {
        return new ServiceA(serviceB);  // Constructor 순환 참조
    }

    @Bean
    public ServiceB serviceB(ServiceA serviceA) {
        return new ServiceB(serviceA);
    }
}

// 실행 시 BeanCurrentlyInCreationException 발생
// 스택 트레이스 분석

// 해결 방법 1: Setter Injection
@Component
public class ServiceA {
    private ServiceB serviceB;

    @Autowired
    public void setServiceB(ServiceB b) {
        this.serviceB = b;
    }
}

// 해결 방법 2: @Lazy
@Component
public class ServiceA {
    private final ServiceB serviceB;

    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;  // 프록시 주입
    }
}

// 디버거로 3-Level 캐시 동작 확인
// singletonFactories → earlySingletonObjects → singletonObjects
```

**📝 Day 3 산출물**

- 면접 Q&A 7개: DI와 순환 참조
- 블로그 포스트: "Spring DI 방식 선택 가이드"

---

#### **Day 4: AOP와 Proxy Pattern (5시간)**

**📚 이론 학습 (3시간)**

**1.5시간: Spring AOP 개념과 구조**

```
[AOP 용어 정리]
Aspect: 관점 (로깅, 트랜잭션 등)
Join Point: 실행 지점 (메서드 호출, 필드 접근)
Advice: 실행할 코드
  - @Before: 메서드 실행 전
  - @After: 메서드 실행 후 (예외 포함)
  - @AfterReturning: 정상 리턴 후
  - @AfterThrowing: 예외 발생 후
  - @Around: 메서드 실행 전후
Pointcut: Join Point 선택 표현식
Target: 원본 객체
Proxy: 타겟을 감싸는 객체
Weaving: Aspect를 타겟에 적용

[Spring AOP vs AspectJ]
Spring AOP:
- 런타임 프록시 기반
- 메서드 실행만 지원
- Spring Bean만 가능
- 성능 오버헤드 적음

AspectJ:
- 컴파일/로드 타임 Weaving
- 모든 Join Point 지원
- 모든 객체 가능
- 강력하지만 복잡
```

**1시간: Proxy 생성 메커니즘**

```
[JDK Dynamic Proxy]
조건: 인터페이스 구현 클래스
생성: java.lang.reflect.Proxy

UserService (interface)
    ↓
UserServiceImpl (target)
    ↓
Proxy$UserService (proxy)
    ↓
InvocationHandler.invoke()
    ↓
Method.invoke(target, args)

[CGLIB Proxy]
조건: 구체 클래스
생성: ByteCode 조작

UserService (class)
    ↓
UserService$$EnhancerBySpringCGLIB (subclass)
    ↓
MethodInterceptor.intercept()
    ↓
MethodProxy.invoke(target, args)

[선택 기준]
proxyTargetClass=false (기본값):
- 인터페이스 있음 → JDK Proxy
- 인터페이스 없음 → CGLIB

proxyTargetClass=true:
- 무조건 CGLIB
```

**30분: Self-Invocation 문제**

```
[문제 상황]
@Service
public class OrderService {
    @Transactional
    public void createOrder() {
        // ...
        this.updateInventory();  // 프록시 거치지 않음!
    }

    @Transactional
    public void updateInventory() {
        // 트랜잭션 적용 안 됨
    }
}

[원인]
this.updateInventory()는 프록시가 아닌
타겟 객체의 메서드 직접 호출

[해결 방법]
1. 자기 주입
@Autowired
private OrderService self;

2. ApplicationContext 사용
@Autowired
private ApplicationContext ctx;
ctx.getBean(OrderService.class).updateInventory();

3. 클래스 분리 (권장)
```

**🔬 실습: AOP 동작 검증 (2시간)**

**실습 1: Proxy 타입 확인 (30분)**

```java
interface UserService {
    void saveUser(String name);
}

@Service
@Slf4j
public class UserServiceImpl implements UserService {
    @Override
    public void saveUser(String name) {
        log.info("Saving user: {}", name);
    }
}

@Service
@Slf4j
public class AdminService {  // 인터페이스 없음
    public void saveAdmin(String name) {
        log.info("Saving admin: {}", name);
    }
}

@Aspect
@Component
public class LoggingAspect {
    @Before("@annotation(Loggable)")
    public void log(JoinPoint jp) {
        log.info("Executing: {}", jp.getSignature());
    }
}

@Test
public void 프록시_타입_확인() {
    // JDK Proxy 확인
    log.info("UserService proxy: {}",
        userService.getClass());  // com.sun.proxy.$Proxy
    log.info("Is JDK Proxy: {}",
        Proxy.isProxyClass(userService.getClass()));

    // CGLIB Proxy 확인
    log.info("AdminService proxy: {}",
        adminService.getClass());  // AdminService$$EnhancerBySpringCGLIB
    log.info("Is CGLIB Proxy: {}",
        adminService.getClass().getName().contains("CGLIB"));
}
```

**실습 2: Self-Invocation 문제 재현 (45분)**

```java
@Service
@Slf4j
public class TransactionService {

    @Transactional
    public void publicMethod() {
        log.info("Public method - TX active: {}",
            TransactionSynchronizationManager.isActualTransactionActive());

        this.internalMethod();  // Self-invocation

        // AopContext 사용 (aspectj-autoproxy expose-proxy="true")
        ((TransactionService) AopContext.currentProxy()).internalMethod();
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void internalMethod() {
        log.info("Internal method - TX active: {}",
            TransactionSynchronizationManager.isActualTransactionActive());
        log.info("TX name: {}",
            TransactionSynchronizationManager.getCurrentTransactionName());
    }
}

// 테스트
@Test
public void self_invocation_테스트() {
    transactionService.publicMethod();
    // 로그 확인:
    // - this.internalMethod(): 같은 트랜잭션
    // - AopContext: 새 트랜잭션
}
```

**실습 3: @Around Advice 실습 (45분)**

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface TimeTrace {
    String value() default "";
}

@Aspect
@Component
@Slf4j
public class PerformanceAspect {

    @Around("@annotation(timeTrace)")
    public Object trace(ProceedingJoinPoint pjp, TimeTrace timeTrace)
            throws Throwable {

        String methodName = pjp.getSignature().toShortString();
        StopWatch stopWatch = new StopWatch(methodName);

        try {
            stopWatch.start();

            // 타겟 메서드 실행
            Object result = pjp.proceed();

            stopWatch.stop();

            if (stopWatch.getTotalTimeMillis() > 1000) {
                log.warn("Slow method: {} took {}ms",
                    methodName, stopWatch.getTotalTimeMillis());
            } else {
                log.info("Method: {} took {}ms",
                    methodName, stopWatch.getTotalTimeMillis());
            }

            return result;

        } catch (Exception e) {
            log.error("Method {} failed: {}", methodName, e.getMessage());
            throw e;
        }
    }
}

@Service
public class SlowService {

    @TimeTrace("Heavy Operation")
    public String heavyOperation() throws InterruptedException {
        Thread.sleep(2000);
        return "Complete";
    }

    @TimeTrace
    public String fastOperation() {
        return "Fast";
    }
}
```

**📝 Day 4 산출물**

- 면접 Q&A 8개: AOP와 Proxy
- 실습 코드: Custom AOP Aspect 3종

---

#### **Day 5: Spring Event System (5시간)**

**📚 이론 학습 (3시간)**

**1시간: Spring Event 아키텍처**

```
[Event 발행 구조]
ApplicationEventPublisher
    ↓
ApplicationEventMulticaster (실제 처리)
    ↓
ApplicationListener 목록 조회
    ↓
각 Listener 실행

[기본 Multicaster]
SimpleApplicationEventMulticaster:
- 동기 실행 (기본)
- 발행 스레드에서 직접 실행
- 트랜잭션 컨텍스트 공유

[비동기 설정]
@Bean
ApplicationEventMulticaster multicaster() {
    SimpleApplicationEventMulticaster m =
        new SimpleApplicationEventMulticaster();
    m.setTaskExecutor(taskExecutor());
    return m;
}

[이벤트 타입]
1. ApplicationEvent 상속 (레거시)
2. POJO 이벤트 (Spring 4.2+)
3. Generic 이벤트 지원
```

**1시간: @EventListener 처리 과정**

```
[EventListenerMethodProcessor]
1. @EventListener 메서드 스캔
2. ApplicationListenerMethodAdapter 생성
3. ApplicationContext에 등록

[@EventListener 옵션]
condition: SpEL 표현식으로 조건부 실행
@EventListener(condition = "#event.success")

classes: 특정 이벤트 타입만
@EventListener(classes = {OrderEvent.class})

@Order: 실행 순서
@Order(1)
@EventListener

[제네릭 이벤트]
public class EntityEvent<T> {
    private T entity;
    private EventType type;
}

@EventListener
public void handleUser(EntityEvent<User> event) {
    // User 타입만 처리
}
```

**1시간: Transaction과 Event**

```
[@TransactionalEventListener]
트랜잭션 단계별 실행:

AFTER_COMMIT (기본값):
- 커밋 성공 후 실행
- 외부 시스템 연동에 안전

AFTER_ROLLBACK:
- 롤백 후 실행
- 보상 트랜잭션

AFTER_COMPLETION:
- 커밋/롤백 관계없이 실행
- 로깅, 모니터링

BEFORE_COMMIT:
- 커밋 직전 실행
- 같은 트랜잭션 내 처리

[동작 원리]
TransactionSynchronization 등록
    ↓
Transaction 진행
    ↓
Commit/Rollback
    ↓
TransactionSynchronization.afterCommit() 등
    ↓
이벤트 발행

[주의사항]
- 트랜잭션 밖에서 발행하면 즉시 실행
- 새 트랜잭션 필요하면 REQUIRES_NEW
```

**🔬 실습: Event System 활용 (2시간)**

**실습 1: Custom Event 발행과 처리 (40분)**

```java
// 이벤트 정의
@Getter
public class OrderCreatedEvent {
    private final Long orderId;
    private final String customerEmail;
    private final LocalDateTime createdAt;

    public OrderCreatedEvent(Long orderId, String email) {
        this.orderId = orderId;
        this.customerEmail = email;
        this.createdAt = LocalDateTime.now();
    }
}

// 이벤트 발행
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService {
    private final ApplicationEventPublisher publisher;

    @Transactional
    public Order createOrder(OrderRequest request) {
        Order order = new Order(request);
        orderRepository.save(order);

        log.info("Publishing event for order: {}", order.getId());
        publisher.publishEvent(
            new OrderCreatedEvent(order.getId(), request.getEmail())
        );

        return order;
    }
}

// 이벤트 리스너
@Component
@Slf4j
public class OrderEventHandler {

    @EventListener
    @Order(1)
    public void sendEmail(OrderCreatedEvent event) {
        log.info("1. Sending email to: {}", event.getCustomerEmail());
    }

    @EventListener
    @Order(2)
    public void updateInventory(OrderCreatedEvent event) {
        log.info("2. Updating inventory for order: {}", event.getOrderId());
    }

    @Async
    @EventListener
    public void generateReport(OrderCreatedEvent event) {
        log.info("3. Async report generation for: {}", event.getOrderId());
    }
}

// 실행 로그 확인
// Order 어노테이션 순서대로 실행
// @Async는 별도 스레드
```

**실습 2: TransactionalEventListener 검증 (40분)**

```java
@Service
@Slf4j
@Transactional
public class PaymentService {

    @Autowired
    private ApplicationEventPublisher publisher;

    public void processPayment(Long orderId, BigDecimal amount) {
        // 결제 처리
        Payment payment = new Payment(orderId, amount);
        paymentRepository.save(payment);

        log.info("TX active: {}",
            TransactionSynchronizationManager.isActualTransactionActive());

        // 이벤트 발행 (트랜잭션 내)
        publisher.publishEvent(new PaymentProcessedEvent(payment));

        if (amount.compareTo(BigDecimal.valueOf(10000)) > 0) {
            throw new RuntimeException("테스트용 예외");
        }
    }
}

@Component
@Slf4j
public class PaymentEventHandler {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void onSuccess(PaymentProcessedEvent event) {
        log.info("AFTER_COMMIT: Payment success - {}", event.getPaymentId());
        // 외부 시스템 알림
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void onRollback(PaymentProcessedEvent event) {
        log.info("AFTER_ROLLBACK: Payment failed - {}", event.getPaymentId());
        // 보상 처리
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMPLETION)
    public void onComplete(PaymentProcessedEvent event) {
        log.info("AFTER_COMPLETION: Payment processed - {}", event.getPaymentId());
        // 로깅, 모니터링
    }
}

// 테스트
@Test
public void 트랜잭션_이벤트_테스트() {
    // 성공 케이스: AFTER_COMMIT 실행
    paymentService.processPayment(1L, BigDecimal.valueOf(100));

    // 실패 케이스: AFTER_ROLLBACK 실행
    assertThrows(RuntimeException.class, () ->
        paymentService.processPayment(2L, BigDecimal.valueOf(20000))
    );
}
```

**실습 3: Event 기반 모듈 분리 (40분)**

```java
// Domain Event
public class DomainEvent {
    private final String aggregateId;
    private final LocalDateTime occurredAt;
    private final String eventType;
}

// Bounded Context A
@Component
public class OrderContext {
    @Autowired
    private ApplicationEventPublisher publisher;

    public void placeOrder(Order order) {
        // 주문 처리

        // 도메인 이벤트 발행
        publisher.publishEvent(new DomainEvent(
            order.getId(),
            LocalDateTime.now(),
            "ORDER_PLACED"
        ));
    }
}

// Bounded Context B
@Component
@Slf4j
public class InventoryContext {

    @EventListener
    public void handle(DomainEvent event) {
        if ("ORDER_PLACED".equals(event.getEventType())) {
            log.info("Inventory: Processing order {}", event.getAggregateId());
            // 재고 차감
        }
    }
}

// Bounded Context C
@Component
@Slf4j
public class ShippingContext {

    @EventListener
    @Async
    public void handle(DomainEvent event) {
        if ("ORDER_PLACED".equals(event.getEventType())) {
            log.info("Shipping: Preparing shipment for {}", event.getAggregateId());
            // 배송 준비
        }
    }
}
```

**📝 Week 1 산출물 종합**

- 면접 Q&A 30개 작성 완료
- 블로그 포스트 1편: "Spring DI 방식 선택 가이드"
- 실습 코드: GitHub 저장소 생성

---

### **Week 2: Spring 고급 기능과 Boot**

#### **Day 6-7: Spring Boot 자동 설정 (10시간)**

**📚 이론 학습 (6시간)**

**2시간: @SpringBootApplication 분해**

```
[@SpringBootApplication 구성]
@SpringBootConfiguration
  = @Configuration
  = 설정 클래스 표시

@EnableAutoConfiguration
  = 자동 설정 활성화
  = AutoConfigurationImportSelector 동작

@ComponentScan
  = 현재 패키지부터 스캔
  = @Component 클래스 찾기

[자동 설정 로딩 과정]
1. AutoConfigurationImportSelector.selectImports()
2. SpringFactoriesLoader.loadFactoryNames()
3. META-INF/spring.factories 읽기 (Boot 2.x)
4. META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports (Boot 3.x)
5. @Conditional 평가
6. 조건 만족 시 Bean 등록
```

**2시간: @Conditional 동작 원리**

```
[@Conditional 종류]
@ConditionalOnClass: 클래스패스에 특정 클래스
@ConditionalOnMissingClass: 클래스 없을 때

@ConditionalOnBean: 특정 Bean 존재
@ConditionalOnMissingBean: Bean 없을 때

@ConditionalOnProperty: 프로퍼티 값 확인
  spring.datasource.enabled=true

@ConditionalOnResource: 리소스 파일 존재

@ConditionalOnWebApplication: 웹 환경
@ConditionalOnNotWebApplication: 비웹 환경

@ConditionalOnExpression: SpEL 표현식

[평가 시점]
ConfigurationPhase.PARSE_CONFIGURATION:
  - 설정 클래스 파싱 시점
  - @Configuration 레벨

ConfigurationPhase.REGISTER_BEAN:
  - Bean 등록 시점
  - @Bean 메서드 레벨

[Custom Condition]
public class OnDatabaseCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context,
                         AnnotatedTypeMetadata metadata) {
        return context.getEnvironment()
            .containsProperty("spring.datasource.url");
    }
}
```

**2시간: 자동 설정 우선순위**

```
[우선순위 제어]
@AutoConfigureBefore: 특정 설정 전 실행
@AutoConfigureAfter: 특정 설정 후 실행
@AutoConfigureOrder: 순서 지정

[설정 오버라이드]
1순위: 사용자 @Configuration
2순위: 사용자 @Bean
3순위: 자동 설정 @Bean

[설정 제외]
@SpringBootApplication(
    exclude = {DataSourceAutoConfiguration.class}
)

또는 properties:
spring.autoconfigure.exclude=\
  org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

**🔬 실습: 자동 설정 분석 (4시간)**

**실습 1: 자동 설정 디버깅 (1시간 20분)**

```java
// 자동 설정 리포트 활성화
// application.properties
debug=true

// 또는 VM 옵션
-Ddebug

@SpringBootTest
public class AutoConfigTest {

    @Autowired
    private ApplicationContext context;

    @Test
    public void 자동_설정_확인() {
        // 등록된 모든 Bean 출력
        String[] beanNames = context.getBeanDefinitionNames();

        Arrays.stream(beanNames)
            .filter(name -> name.contains("AutoConfiguration"))
            .forEach(name -> {
                log.info("Auto Config: {}", name);

                // Bean 정의 정보
                BeanDefinition bd =
                    ((ConfigurableApplicationContext) context)
                        .getBeanFactory()
                        .getBeanDefinition(name);

                log.info("  Source: {}", bd.getSource());
            });
    }

    @Test
    public void Conditional_평가_결과() {
        // ConditionEvaluationReport 확인
        ConditionEvaluationReport report =
            context.getBean(ConditionEvaluationReport.class);

        // 매칭된 설정
        report.getConditionAndOutcomesBySource()
            .entrySet()
            .stream()
            .filter(e -> e.getValue().isFullMatch())
            .forEach(e -> log.info("Matched: {}", e.getKey()));

        // 제외된 설정과 이유
        report.getExclusions()
            .forEach((key, reason) ->
                log.info("Excluded: {} - {}", key, reason));
    }
}
```

**실습 2: Custom Auto Configuration (1시간 20분)**

```java
// 1. Custom Properties
@ConfigurationProperties(prefix = "custom.cache")
@Data
public class CustomCacheProperties {
    private boolean enabled = true;
    private int maxSize = 1000;
    private long ttlSeconds = 3600;
    private String type = "caffeine";  // caffeine, ehcache
}

// 2. Auto Configuration 클래스
@Configuration
@ConditionalOnClass(Cache.class)
@EnableConfigurationProperties(CustomCacheProperties.class)
@AutoConfigureAfter(RedisAutoConfiguration.class)
public class CustomCacheAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(
        prefix = "custom.cache",
        name = "enabled",
        havingValue = "true",
        matchIfMissing = true
    )
    public CacheManager cacheManager(CustomCacheProperties props) {
        log.info("Creating CacheManager with: {}", props);

        if ("caffeine".equals(props.getType())) {
            return caffeineCacheManager(props);
        } else {
            return ehCacheManager(props);
        }
    }

    private CacheManager caffeineCacheManager(CustomCacheProperties props) {
        CaffeineCacheManager manager = new CaffeineCacheManager();
        manager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(props.getMaxSize())
            .expireAfterWrite(props.getTtlSeconds(), TimeUnit.SECONDS));
        return manager;
    }
}

// 3. spring.factories (Boot 2.x)
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.CustomCacheAutoConfiguration

// 또는 AutoConfiguration.imports (Boot 3.x)
com.example.CustomCacheAutoConfiguration
```

**실습 3: 조건부 Bean 등록 (1시간 20분)**

```java
@Configuration
public class DatabaseConfig {

    @Bean
    @ConditionalOnProperty(name = "db.type", havingValue = "mysql")
    public DataSource mysqlDataSource() {
        log.info("Creating MySQL DataSource");
        return DataSourceBuilder.create()
            .driverClassName("com.mysql.cj.jdbc.Driver")
            .build();
    }

    @Bean
    @ConditionalOnProperty(name = "db.type", havingValue = "postgres")
    public DataSource postgresDataSource() {
        log.info("Creating PostgreSQL DataSource");
        return DataSourceBuilder.create()
            .driverClassName("org.postgresql.Driver")
            .build();
    }

    @Bean
    @ConditionalOnBean(name = "mysqlDataSource")
    public JdbcTemplate mysqlJdbcTemplate(
            @Qualifier("mysqlDataSource") DataSource ds) {
        return new JdbcTemplate(ds);
    }

    @Bean
    @Primary
    @ConditionalOnMissingBean(DataSource.class)
    public DataSource embeddedDataSource() {
        log.info("Creating Embedded H2 DataSource");
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}

// 테스트
@SpringBootTest
@TestPropertySource(properties = "db.type=mysql")
public class ConditionalBeanTest {

    @Autowired
    private ApplicationContext context;

    @Test
    public void MySQL_선택_확인() {
        assertTrue(context.containsBean("mysqlDataSource"));
        assertFalse(context.containsBean("postgresDataSource"));
        assertTrue(context.containsBean("mysqlJdbcTemplate"));
    }
}
```

**📝 Day 6-7 산출물**

- 면접 Q&A 10개: Spring Boot 자동 설정
- Custom Starter 프로젝트 생성

---

#### **Day 8-9: Spring Data Access (10시간)**

**📚 이론 학습 (6시간)**

**3시간: Spring JDBC와 DataSource**

```
[DataSource 추상화]
DataSource: Connection 팩토리
├── DriverManagerDataSource: 매번 새 Connection
├── SingleConnectionDataSource: 하나의 Connection 재사용
└── HikariDataSource: Connection Pool

[Connection Pool 동작]
1. 애플리케이션 시작 시 Pool 초기화
2. minimumIdle 만큼 Connection 생성
3. getConnection() 호출 시:
   - Pool에서 가용 Connection 반환
   - 없으면 대기 또는 생성 (maximumPoolSize까지)
4. Connection.close() 호출 시:
   - 실제로 닫지 않고 Pool에 반환
   - Validation 체크

[HikariCP 핵심 설정]
minimumIdle: 최소 유지 Connection
maximumPoolSize: 최대 Connection 수
connectionTimeout: Connection 획득 대기 시간
idleTimeout: Idle Connection 유지 시간
maxLifetime: Connection 최대 수명
validationTimeout: Connection 유효성 체크 시간

[Pool Size 계산]
connections = ((core_count * 2) + effective_spindle_count)
실무: 대략 10-20개 (DB 성능 고려)
```

**3시간: Transaction 추상화**

```
[PlatformTransactionManager]
인터페이스 - 트랜잭션 추상화
├── DataSourceTransactionManager (JDBC)
├── JpaTransactionManager (JPA)
├── JtaTransactionManager (JTA/XA)
└── MongoTransactionManager (MongoDB)

[TransactionDefinition]
트랜잭션 속성 정의:
- Propagation: 전파 방식
- Isolation: 격리 수준
- Timeout: 타임아웃
- ReadOnly: 읽기 전용
- Rollback Rules: 롤백 규칙

[TransactionStatus]
현재 트랜잭션 상태:
- isNewTransaction(): 새 트랜잭션?
- hasSavepoint(): Savepoint 있음?
- isRollbackOnly(): 롤백 예정?
- isCompleted(): 완료됨?

[동작 과정]
1. TransactionInterceptor 실행
2. TransactionManager.getTransaction()
3. Connection 획득/재사용 결정
4. autoCommit = false
5. 비즈니스 로직 실행
6. commit() 또는 rollback()
7. Connection 정리
```

**🔬 실습: Data Access 최적화 (4시간)**

**실습 1: Connection Pool 모니터링 (1시간 20분)**

```java
@Configuration
public class DataSourceConfig {

    @Bean
    @Primary
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        config.setUsername("root");
        config.setPassword("password");

        // Pool 설정
        config.setMinimumIdle(5);
        config.setMaximumPoolSize(10);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);

        // 모니터링 설정
        config.setMetricRegistry(new MetricRegistry());
        config.setHealthCheckRegistry(new HealthCheckRegistry());

        return new HikariDataSource(config);
    }

    @Bean
    public HikariPoolMXBean poolMXBean(DataSource dataSource) {
        return ((HikariDataSource) dataSource).getHikariPoolMXBean();
    }
}

@RestController
@RequiredArgsConstructor
public class PoolMonitorController {

    private final HikariPoolMXBean poolMXBean;

    @GetMapping("/pool/stats")
    public Map<String, Object> getPoolStats() {
        return Map.of(
            "activeConnections", poolMXBean.getActiveConnections(),
            "idleConnections", poolMXBean.getIdleConnections(),
            "totalConnections", poolMXBean.getTotalConnections(),
            "threadsAwaitingConnection", poolMXBean.getThreadsAwaitingConnection()
        );
    }
}

// 부하 테스트
@Test
public void Connection_Pool_부하_테스트() throws Exception {
    ExecutorService executor = Executors.newFixedThreadPool(20);
    CountDownLatch latch = new CountDownLatch(100);

    for (int i = 0; i < 100; i++) {
        executor.submit(() -> {
            try {
                jdbcTemplate.queryForObject(
                    "SELECT SLEEP(1)", Integer.class
                );
            } finally {
                latch.countDown();
            }
        });
    }

    // 실행 중 /pool/stats 모니터링
    // activeConnections 증가 확인
    // threadsAwaitingConnection 확인

    latch.await();
}
```

**실습 2: JdbcTemplate 활용 (1시간 20분)**

```java
@Repository
@RequiredArgsConstructor
@Slf4j
public class OrderJdbcRepository {

    private final JdbcTemplate jdbcTemplate;
    private final NamedParameterJdbcTemplate namedJdbcTemplate;

    // 단건 조회
    public Order findById(Long id) {
        String sql = "SELECT * FROM orders WHERE id = ?";

        return jdbcTemplate.queryForObject(sql,
            new OrderRowMapper(), id);
    }

    // 목록 조회 with RowMapper
    public List<Order> findAll() {
        String sql = "SELECT * FROM orders";

        return jdbcTemplate.query(sql, (rs, rowNum) ->
            Order.builder()
                .id(rs.getLong("id"))
                .customerName(rs.getString("customer_name"))
                .totalAmount(rs.getBigDecimal("total_amount"))
                .createdAt(rs.getTimestamp("created_at").toLocalDateTime())
                .build()
        );
    }

    // Batch Insert
    public void batchInsert(List<Order> orders) {
        String sql = "INSERT INTO orders (customer_name, total_amount) VALUES (?, ?)";

        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Order order = orders.get(i);
                ps.setString(1, order.getCustomerName());
                ps.setBigDecimal(2, order.getTotalAmount());
            }

            @Override
            public int getBatchSize() {
                return orders.size();
            }
        });

        log.info("Batch inserted {} orders", orders.size());
    }

    // Named Parameters
    public List<Order> findByDateRange(LocalDateTime start, LocalDateTime end) {
        String sql = """
            SELECT * FROM orders
            WHERE created_at BETWEEN :start AND :end
            ORDER BY created_at DESC
            """;

        MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("start", start)
            .addValue("end", end);

        return namedJdbcTemplate.query(sql, params, new OrderRowMapper());
    }

    // SimpleJdbcInsert 활용
    @PostConstruct
    public void init() {
        this.insertOrder = new SimpleJdbcInsert(jdbcTemplate)
            .withTableName("orders")
            .usingGeneratedKeyColumns("id");
    }

    private SimpleJdbcInsert insertOrder;

    public Long insert(Order order) {
        Map<String, Object> params = Map.of(
            "customer_name", order.getCustomerName(),
            "total_amount", order.getTotalAmount(),
            "created_at", Timestamp.valueOf(LocalDateTime.now())
        );

        return insertOrder.executeAndReturnKey(params).longValue();
    }
}

// 성능 테스트
@Test
public void Batch_vs_Loop_성능_비교() {
    List<Order> orders = generateOrders(1000);

    StopWatch watch = new StopWatch();

    // Loop Insert
    watch.start("Loop Insert");
    orders.forEach(repository::insert);
    watch.stop();

    // Batch Insert
    watch.start("Batch Insert");
    repository.batchInsert(orders);
    watch.stop();

    log.info("Performance: \n{}", watch.prettyPrint());
    // Batch가 10배 이상 빠름
}
```

**실습 3: Transaction Template (1시간 20분)**

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class TransferService {

    private final TransactionTemplate transactionTemplate;
    private final AccountRepository accountRepository;

    public TransferService(PlatformTransactionManager txManager) {
        this.transactionTemplate = new TransactionTemplate(txManager);

        // 트랜잭션 설정
        this.transactionTemplate.setIsolationLevel(
            TransactionDefinition.ISOLATION_READ_COMMITTED);
        this.transactionTemplate.setTimeout(10);
    }

    // 프로그래매틱 트랜잭션
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus status) {
                try {
                    Account from = accountRepository.findById(fromId);
                    Account to = accountRepository.findById(toId);

                    log.info("TX Name: {}",
                        TransactionSynchronizationManager.getCurrentTransactionName());
                    log.info("TX Active: {}",
                        TransactionSynchronizationManager.isActualTransactionActive());
                    log.info("TX Isolation: {}",
                        TransactionSynchronizationManager.getCurrentTransactionIsolationLevel());

                    from.withdraw(amount);
                    to.deposit(amount);

                    accountRepository.update(from);
                    accountRepository.update(to);

                    if (amount.compareTo(BigDecimal.valueOf(10000)) > 0) {
                        status.setRollbackOnly();
                        log.warn("Large transfer marked for rollback");
                    }

                } catch (Exception e) {
                    status.setRollbackOnly();
                    throw new TransferException("Transfer failed", e);
                }
            }
        });
    }

    // 반환값이 있는 트랜잭션
    public TransferResult transferWithResult(Long fromId, Long toId, BigDecimal amount) {
        return transactionTemplate.execute(status -> {
            // 트랜잭션 내 처리
            // ...

            return new TransferResult(
                transferId,
                TransferStatus.SUCCESS,
                LocalDateTime.now()
            );
        });
    }

    // 중첩 트랜잭션
    public void nestedTransfer() {
        transactionTemplate.execute(status -> {
            log.info("Outer TX: {}", status.isNewTransaction());

            // REQUIRES_NEW로 새 트랜잭션
            TransactionTemplate newTxTemplate = new TransactionTemplate(txManager);
            newTxTemplate.setPropagationBehavior(
                TransactionDefinition.PROPAGATION_REQUIRES_NEW);

            newTxTemplate.execute(innerStatus -> {
                log.info("Inner TX: {}", innerStatus.isNewTransaction());
                // 독립적인 트랜잭션
                return null;
            });

            return null;
        });
    }
}
```

**📝 Day 8-9 산출물**

- 면접 Q&A 10개: DataSource와 Transaction
- 블로그 포스트: "HikariCP 최적화 가이드"

---

#### **Day 10: Week 2 종합 실습 (5시간)**

**📚 복습 및 통합 (2시간)**

- Spring Boot 자동 설정 플로우 재정리
- DataSource와 Transaction 연결 관계
- 주요 개념 다이어그램 작성

**🔬 종합 프로젝트 (3시간)**

**Multi-DataSource with Transaction**

```java
@Configuration
public class MultiDataSourceConfig {

    @Primary
    @Bean
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    @ConfigurationProperties("spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean
    public PlatformTransactionManager primaryTxManager(
            @Qualifier("primaryDataSource") DataSource ds) {
        return new DataSourceTransactionManager(ds);
    }

    @Bean
    public PlatformTransactionManager secondaryTxManager(
            @Qualifier("secondaryDataSource") DataSource ds) {
        return new DataSourceTransactionManager(ds);
    }
}

@Service
@Slf4j
public class MultiDBService {

    @Transactional("primaryTxManager")
    public void primaryOperation() {
        // Primary DB 작업
    }

    @Transactional("secondaryTxManager")
    public void secondaryOperation() {
        // Secondary DB 작업
    }

    // ChainedTransactionManager 대체 (Boot 3.x)
    public void distributedOperation() {
        TransactionTemplate primary = new TransactionTemplate(primaryTxManager);
        TransactionTemplate secondary = new TransactionTemplate(secondaryTxManager);

        try {
            primary.execute(status -> {
                // Primary DB 작업

                secondary.execute(innerStatus -> {
                    // Secondary DB 작업
                    return null;
                });

                return null;
            });
        } catch (Exception e) {
            // 수동 보상 처리
            compensate();
        }
    }
}
```

**📝 Week 2 최종 산출물**

- 면접 Q&A 50개 (누적)
- 블로그 포스트 2편 작성
- 종합 프로젝트 코드 GitHub Push

---

## **Phase 2: Transaction 심화 (Week 3-4)**

### **Week 3: Transaction 메커니즘**

#### **Day 11-12: Transaction 전파와 격리 (10시간)**

**📚 이론 학습 (6시간)**

**3시간: Propagation 완벽 이해**

```
[REQUIRED (기본값)]
동작: 기존 트랜잭션 참여 또는 새로 생성
┌─────────────────────────────┐
│  TX1 시작                    │
│  ├─ Service A (REQUIRED)     │ ← TX1 생성
│  │  └─ Service B (REQUIRED)  │ ← TX1 참여
│  └─ TX1 종료                 │
└─────────────────────────────┘

실무 시나리오:
- 주문 → 재고차감: 같은 트랜잭션
- 하나라도 실패하면 전체 롤백

주의사항:
- B에서 예외 → A도 롤백
- UnexpectedRollbackException 가능

[REQUIRES_NEW]
동작: 항상 새 트랜잭션 생성
┌─────────────────────────────┐
│  TX1 시작                    │
│  ├─ Service A                │
│  │  ┌──────────────────┐    │
│  │  │ TX1 일시 중단     │    │
│  │  │ TX2 시작          │    │
│  │  │ Service B 실행    │    │
│  │  │ TX2 커밋/롤백     │    │
│  │  └──────────────────┘    │
│  │  TX1 재개                 │
│  └─ TX1 커밋/롤백            │
└─────────────────────────────┘

실무 시나리오:
- 주문 → 감사로그: 독립 트랜잭션
- 로그 실패해도 주문은 성공

리소스 사용:
- Connection 2개 필요
- Deadlock 위험 증가

[NESTED]
동작: 중첩 트랜잭션 (Savepoint)
┌─────────────────────────────┐
│  TX1 시작                    │
│  ├─ Service A                │
│  │  ├─ Savepoint S1 생성     │
│  │  ├─ Service B 실행        │
│  │  └─ S1 롤백 가능          │ ← 부분 롤백
│  └─ TX1 커밋                 │
└─────────────────────────────┘

실무 시나리오:
- 배치 처리 중 일부 실패 허용
- 복잡한 비즈니스 로직 격리

DB 지원:
- Oracle, PostgreSQL: 완벽 지원
- MySQL: 제한적 (SAVEPOINT 명령)

[MANDATORY]
- 트랜잭션 필수, 없으면 예외

[NEVER]
- 트랜잭션 있으면 예외

[SUPPORTS]
- 있으면 참여, 없어도 OK

[NOT_SUPPORTED]
- 트랜잭션 중단하고 실행
```

**3시간: Isolation Level 실전**

```
[READ_UNCOMMITTED]
가장 낮은 격리 수준
- Dirty Read 허용
- 커밋 안 된 데이터 읽기 가능

실무:
- 거의 사용 안 함
- 실시간 모니터링 정도

[READ_COMMITTED]
- Dirty Read 방지
- Non-Repeatable Read 가능
- 대부분 DB 기본값

구현 방식:
- Oracle: Undo 세그먼트
- PostgreSQL: MVCC
- MySQL: Undo Log

실무 시나리오:
TX1: SELECT balance FROM account WHERE id=1; -- 1000원
TX2: UPDATE account SET balance=500 WHERE id=1; COMMIT;
TX1: SELECT balance FROM account WHERE id=1; -- 500원 (변경됨)

[REPEATABLE_READ]
- Non-Repeatable Read 방지
- Phantom Read 가능 (MySQL InnoDB는 방지)
- MySQL InnoDB 기본값

구현 방식:
- Snapshot 생성
- 트랜잭션 시작 시점 데이터

실무 문제:
- Lost Update 가능
- Write Skew 발생

[SERIALIZABLE]
- 완벽한 격리
- 성능 최악
- Lock 기반 또는 SSI(Serializable Snapshot Isolation)

실무:
- 정산, 송금 등 중요 트랜잭션
- 성능보다 정합성이 중요한 경우
```

**🔬 실습: 전파와 격리 검증 (4시간)**

**실습 1: Propagation 동작 확인 (1시간 20분)**

```java
@Service
@Slf4j
public class PropagationTestService {

    @Autowired
    private PropagationTestService self;  // 프록시 주입

    @Transactional(propagation = Propagation.REQUIRED)
    public void requiredOuter() {
        log.info("REQUIRED Outer - TX: {}", getCurrentTransactionName());

        orderRepository.save(new Order("REQUIRED"));

        try {
            self.requiredInner();
            self.requiresNewInner();
            self.nestedInner();
        } catch (Exception e) {
            log.error("Inner failed: {}", e.getMessage());
        }
    }

    @Transactional(propagation = Propagation.REQUIRED)
    public void requiredInner() {
        log.info("REQUIRED Inner - TX: {}", getCurrentTransactionName());
        log.info("Is New TX: {}", isNewTransaction());

        paymentRepository.save(new Payment("REQUIRED"));

        if (shouldFail()) {
            throw new RuntimeException("REQUIRED Inner 실패");
        }
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void requiresNewInner() {
        log.info("REQUIRES_NEW Inner - TX: {}", getCurrentTransactionName());
        log.info("Is New TX: {}", isNewTransaction());

        auditRepository.save(new Audit("REQUIRES_NEW"));

        // Connection 모니터링
        logConnectionInfo();
    }

    @Transactional(propagation = Propagation.NESTED)
    public void nestedInner() {
        log.info("NESTED Inner - TX: {}", getCurrentTransactionName());
        log.info("Has Savepoint: {}", hasSavepoint());

        inventoryRepository.save(new Inventory("NESTED"));

        if (shouldFail()) {
            throw new RuntimeException("NESTED Inner 실패");
        }
    }

    private String getCurrentTransactionName() {
        return TransactionSynchronizationManager.getCurrentTransactionName();
    }

    private boolean isNewTransaction() {
        return TransactionSynchronizationManager.isActualTransactionActive() &&
               TransactionSynchronizationManager.isCurrentTransactionReadOnly() == false;
    }

    private void logConnectionInfo() {
        Map<Object, Object> resources =
            TransactionSynchronizationManager.getResourceMap();
        log.info("Active connections: {}", resources.size());
        resources.forEach((key, value) ->
            log.info("  Resource: {} -> {}", key.getClass().getSimpleName(), value)
        );
    }
}

// 테스트
@SpringBootTest
@Transactional
@Rollback(false)
public class PropagationTest {

    @Test
    public void REQUIRED_전파_테스트() {
        // Given
        setFailurePoint("requiredInner");

        // When
        assertThrows(Exception.class, () ->
            service.requiredOuter()
        );

        // Then
        // Order: 롤백 (같은 TX)
        // Payment: 롤백 (같은 TX)
        assertEquals(0, orderRepository.count());
        assertEquals(0, paymentRepository.count());
    }

    @Test
    public void REQUIRES_NEW_격리_테스트() {
        // Given
        setFailurePoint("requiresNewInner");

        // When
        service.requiredOuter();

        // Then
        // Order: 커밋 (TX1)
        // Audit: 롤백 (TX2 - 독립)
        assertEquals(1, orderRepository.count());
        assertEquals(0, auditRepository.count());
    }
}
```

**실습 2: Isolation Level 시연 (1시간 20분)**

```java
@Service
@Slf4j
public class IsolationTestService {

    @Transactional(isolation = Isolation.READ_UNCOMMITTED)
    public void dirtyRead() throws InterruptedException {
        // TX1: 데이터 읽기
        Account account = accountRepository.findById(1L);
        log.info("1차 조회: {}", account.getBalance());

        // TX2가 수정하도록 대기
        Thread.sleep(2000);

        // TX1: 다시 읽기 (커밋 안 된 데이터)
        account = accountRepository.findById(1L);
        log.info("2차 조회 (Dirty): {}", account.getBalance());
    }

    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void nonRepeatableRead() throws InterruptedException {
        // TX1: 데이터 읽기
        Account account = accountRepository.findById(1L);
        log.info("1차 조회: {}", account.getBalance());

        // TX2가 수정하고 커밋하도록 대기
        Thread.sleep(2000);

        // TX1: 다시 읽기 (변경된 데이터)
        account = accountRepository.findById(1L);
        log.info("2차 조회 (Changed): {}", account.getBalance());
    }

    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void phantomRead() throws InterruptedException {
        // TX1: 범위 조회
        List<Account> accounts = accountRepository.findByType("SAVINGS");
        log.info("1차 조회 건수: {}", accounts.size());

        // TX2가 새 데이터 추가하도록 대기
        Thread.sleep(2000);

        // TX1: 다시 범위 조회
        accounts = accountRepository.findByType("SAVINGS");
        log.info("2차 조회 건수: {}", accounts.size());
    }
}

// 동시성 테스트
@Test
public void Dirty_Read_시연() throws Exception {
    CountDownLatch latch = new CountDownLatch(2);

    // TX1: Reader
    CompletableFuture<Void> reader = CompletableFuture.runAsync(() -> {
        try {
            service.dirtyRead();
        } finally {
            latch.countDown();
        }
    });

    // TX2: Writer (커밋 안 함)
    CompletableFuture<Void> writer = CompletableFuture.runAsync(() -> {
        TransactionTemplate template = new TransactionTemplate(txManager);
        template.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITTED);

        template.execute(status -> {
            Account account = accountRepository.findById(1L);
            account.setBalance(BigDecimal.valueOf(999999));
            accountRepository.save(account);

            // 커밋하지 않고 대기
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {}

            status.setRollbackOnly();  // 롤백
            latch.countDown();
            return null;
        });
    });

    latch.await();

    // 로그 확인
    // 1차 조회: 1000
    // 2차 조회 (Dirty): 999999  <- 커밋 안 된 데이터
}
```

**실습 3: Deadlock 재현과 해결 (1시간 20분)**

```java
@Service
@Transactional
public class DeadlockService {

    // Deadlock 유발 시나리오
    public void transferAtoB(Long accountA, Long accountB, BigDecimal amount) {
        // Lock 순서: A → B
        Account a = accountRepository.findByIdForUpdate(accountA);
        Thread.sleep(100);  // Deadlock 확률 증가
        Account b = accountRepository.findByIdForUpdate(accountB);

        a.withdraw(amount);
        b.deposit(amount);

        accountRepository.save(a);
        accountRepository.save(b);
    }

    public void transferBtoA(Long accountB, Long accountA, BigDecimal amount) {
        // Lock 순서: B → A (역순!)
        Account b = accountRepository.findByIdForUpdate(accountB);
        Thread.sleep(100);
        Account a = accountRepository.findByIdForUpdate(accountA);

        b.withdraw(amount);
        a.deposit(amount);

        accountRepository.save(b);
        accountRepository.save(a);
    }
}

// Deadlock 해결 방법
@Service
public class SafeTransferService {

    // 방법 1: Lock 순서 고정
    public void safeTransfer(Long id1, Long id2, BigDecimal amount) {
        // ID 순서로 항상 Lock
        Long firstLock = Math.min(id1, id2);
        Long secondLock = Math.max(id1, id2);

        Account first = accountRepository.findByIdForUpdate(firstLock);
        Account second = accountRepository.findByIdForUpdate(secondLock);

        // 비즈니스 로직
        if (id1.equals(firstLock)) {
            first.withdraw(amount);
            second.deposit(amount);
        } else {
            second.withdraw(amount);
            first.deposit(amount);
        }
    }

    // 방법 2: Lock Timeout 설정
    @Transactional(timeout = 5)
    public void timeoutTransfer(Long id1, Long id2, BigDecimal amount) {
        // 5초 내 Lock 획득 못하면 실패
    }

    // 방법 3: Optimistic Locking
    public void optimisticTransfer(Long id1, Long id2, BigDecimal amount) {
        int retries = 3;

        while (retries > 0) {
            try {
                Account a = accountRepository.findById(id1);
                Account b = accountRepository.findById(id2);

                a.withdraw(amount);
                b.deposit(amount);

                accountRepository.save(a);  // @Version 체크
                accountRepository.save(b);

                break;

            } catch (OptimisticLockingFailureException e) {
                retries--;
                if (retries == 0) throw e;
                Thread.sleep(100 * (4 - retries));  // Exponential backoff
            }
        }
    }
}
```

**📝 Day 11-12 산출물**

- 면접 Q&A 15개: 트랜잭션 전파와 격리
- 실습 보고서: Isolation Level별 문제 상황

---

#### **Day 13-14: 분산 트랜잭션 패턴 (10시간)**

**📚 이론 학습 (6시간)**

**3시간: 2PC와 Saga 패턴**

```
[2PC (Two-Phase Commit)]
Phase 1: Prepare
- Coordinator → Participants: "준비되셨나요?"
- Participants: 로컬 트랜잭션 준비
- Participants → Coordinator: "Yes/No"

Phase 2: Commit/Abort
- 모두 Yes → Commit 명령
- 하나라도 No → Abort 명령

문제점:
- Coordinator 장애 시 Blocking
- 네트워크 파티션 문제
- 성능 저하 (동기식)
- 구현 복잡도

[Saga 패턴]
긴 트랜잭션을 작은 로컬 트랜잭션으로 분할

Choreography Saga:
Service A → Event → Service B → Event → Service C
- 각 서비스가 이벤트 발행
- 분산 제어
- 복잡도 증가

Orchestration Saga:
Orchestrator → Service A
            → Service B
            → Service C
- 중앙 제어
- 흐름 파악 쉬움
- Single Point of Failure

보상 트랜잭션:
T1 → T2 → T3 (실패)
C3 → C2 → C1 (보상)

[Saga 구현 고려사항]
1. Idempotency 보장
2. 보상 가능한 설계
3. 실패 처리 전략
4. 타임아웃 설정
```

**3시간: Outbox 패턴과 Event Sourcing**

```
[Transactional Outbox]
문제: DB 트랜잭션과 메시지 발행 원자성

해결:
1. 비즈니스 데이터 + Outbox 테이블 (같은 TX)
2. 별도 프로세스가 Outbox 읽고 발행
3. 발행 완료 후 Outbox 삭제

Outbox 테이블:
- id: 메시지 ID
- aggregate_id: 엔티티 ID
- event_type: 이벤트 타입
- payload: 이벤트 데이터
- created_at: 생성 시간
- processed: 처리 여부

발행 방식:
- Polling: 주기적으로 조회
- CDC: Change Data Capture (Debezium)

[Event Sourcing]
상태 저장 대신 이벤트 저장

전통 방식:
- 현재 상태만 저장
- 변경 이력 없음

Event Sourcing:
- 모든 변경을 이벤트로 저장
- 이벤트 재생으로 상태 복원
- 완벽한 감사 로그

이벤트 스토어:
- event_id: UUID
- aggregate_id: 엔티티 ID
- event_type: 이벤트 타입
- event_data: JSON
- event_version: 버전
- occurred_at: 발생 시간

Projection:
- 이벤트 → Read Model
- CQRS와 결합
- 최종 일관성
```

**🔬 실습: 분산 트랜잭션 구현 (4시간)**

**실습 1: Saga 패턴 구현 (1시간 20분)**

```java
// Saga Orchestrator
@Service
@Slf4j
public class OrderSagaOrchestrator {

    @Autowired
    private OrderService orderService;
    @Autowired
    private PaymentService paymentService;
    @Autowired
    private InventoryService inventoryService;
    @Autowired
    private ShippingService shippingService;

    public SagaResult processOrder(OrderRequest request) {
        SagaTransaction saga = new SagaTransaction();

        try {
            // Step 1: 주문 생성
            Order order = orderService.createOrder(request);
            saga.addStep("CREATE_ORDER", order.getId(),
                () -> orderService.cancelOrder(order.getId()));

            // Step 2: 결제 처리
            Payment payment = paymentService.processPayment(
                order.getId(), request.getPaymentInfo());
            saga.addStep("PROCESS_PAYMENT", payment.getId(),
                () -> paymentService.refund(payment.getId()));

            // Step 3: 재고 차감
            Reservation reservation = inventoryService.reserveItems(
                order.getItems());
            saga.addStep("RESERVE_INVENTORY", reservation.getId(),
                () -> inventoryService.releaseReservation(reservation.getId()));

            // Step 4: 배송 준비
            Shipment shipment = shippingService.createShipment(order);
            saga.addStep("CREATE_SHIPMENT", shipment.getId(),
                () -> shippingService.cancelShipment(shipment.getId()));

            // 모든 단계 성공
            saga.complete();
            return SagaResult.success(order);

        } catch (Exception e) {
            log.error("Saga failed at step: {}", saga.getCurrentStep(), e);

            // 보상 트랜잭션 실행
            saga.compensate();

            return SagaResult.failure(e.getMessage());
        }
    }
}

// Saga Transaction 관리
public class SagaTransaction {
    private List<SagaStep> steps = new ArrayList<>();
    private String currentStep;

    public void addStep(String name, String id, Runnable compensation) {
        this.currentStep = name;
        steps.add(new SagaStep(name, id, compensation));
    }

    public void compensate() {
        // 역순으로 보상 실행
        Collections.reverse(steps);

        for (SagaStep step : steps) {
            try {
                log.info("Compensating: {}", step.getName());
                step.compensate();
            } catch (Exception e) {
                log.error("Compensation failed for: {}", step.getName(), e);
                // 보상 실패 처리 (수동 개입 필요)
            }
        }
    }

    @Data
    @AllArgsConstructor
    private static class SagaStep {
        private String name;
        private String entityId;
        private Runnable compensation;

        public void compensate() {
            compensation.run();
        }
    }
}
```

**실습 2: Outbox 패턴 구현 (1시간 20분)**

```java
// Outbox 엔티티
@Entity
@Table(name = "outbox_events")
@Data
public class OutboxEvent {
    @Id
    private String id = UUID.randomUUID().toString();

    private String aggregateId;
    private String eventType;

    @Column(columnDefinition = "TEXT")
    private String payload;

    private LocalDateTime createdAt = LocalDateTime.now();
    private boolean processed = false;
    private LocalDateTime processedAt;
    private int retryCount = 0;
}

// 비즈니스 서비스
@Service
@Transactional
@Slf4j
public class OrderServiceWithOutbox {

    @Autowired
    private OrderRepository orderRepository;
    @Autowired
    private OutboxRepository outboxRepository;

    public Order createOrder(OrderRequest request) {
        // 1. 비즈니스 로직 실행
        Order order = new Order(request);
        orderRepository.save(order);

        // 2. Outbox에 이벤트 저장 (같은 트랜잭션)
        OutboxEvent event = new OutboxEvent();
        event.setAggregateId(order.getId().toString());
        event.setEventType("OrderCreated");
        event.setPayload(toJson(new OrderCreatedEvent(order)));

        outboxRepository.save(event);

        log.info("Order created with outbox event: {}", order.getId());

        return order;
    }
}

// Outbox Publisher (별도 스레드)
@Component
@Slf4j
public class OutboxPublisher {

    @Autowired
    private OutboxRepository outboxRepository;
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @Scheduled(fixedDelay = 5000)
    @Transactional
    public void publishEvents() {
        List<OutboxEvent> unpublished = outboxRepository
            .findByProcessedFalseAndRetryCountLessThan(3);

        for (OutboxEvent event : unpublished) {
            try {
                // Kafka로 발행
                kafkaTemplate.send(
                    event.getEventType(),
                    event.getAggregateId(),
                    event.getPayload()
                ).get(5, TimeUnit.SECONDS);

                // 발행 성공 표시
                event.setProcessed(true);
                event.setProcessedAt(LocalDateTime.now());
                outboxRepository.save(event);

                log.info("Published event: {} for aggregate: {}",
                    event.getEventType(), event.getAggregateId());

            } catch (Exception e) {
                log.error("Failed to publish event: {}", event.getId(), e);

                event.setRetryCount(event.getRetryCount() + 1);
                outboxRepository.save(event);
            }
        }
    }

    // 오래된 처리 완료 이벤트 정리
    @Scheduled(cron = "0 0 2 * * *")  // 매일 새벽 2시
    public void cleanupProcessedEvents() {
        LocalDateTime cutoff = LocalDateTime.now().minusDays(7);
        int deleted = outboxRepository.deleteByProcessedTrueAndProcessedAtBefore(cutoff);
        log.info("Cleaned up {} old events", deleted);
    }
}
```

**실습 3: Event Sourcing 기초 (1시간 20분)**

```java
// 이벤트 저장소
@Entity
@Table(name = "event_store")
@Data
public class EventStore {
    @Id
    private String eventId = UUID.randomUUID().toString();

    private String aggregateId;
    private String aggregateType;
    private String eventType;

    @Column(columnDefinition = "TEXT")
    private String eventData;

    private Long eventVersion;
    private LocalDateTime occurredAt = LocalDateTime.now();

    @Version
    private Long version;  // Optimistic locking
}

// Aggregate Root
public abstract class AggregateRoot {
    private String id;
    private Long version = 0L;
    private List<DomainEvent> changes = new ArrayList<>();

    protected void applyChange(DomainEvent event) {
        applyChange(event, true);
    }

    private void applyChange(DomainEvent event, boolean isNew) {
        // 이벤트 적용
        apply(event);

        // 새 이벤트면 저장
        if (isNew) {
            changes.add(event);
        }
    }

    protected abstract void apply(DomainEvent event);

    public void markChangesAsCommitted() {
        changes.clear();
    }

    public List<DomainEvent> getUncommittedChanges() {
        return changes;
    }

    public void loadFromHistory(List<DomainEvent> history) {
        history.forEach(e -> applyChange(e, false));
        this.version = (long) history.size();
    }
}

// Account Aggregate
public class Account extends AggregateRoot {
    private String accountNumber;
    private BigDecimal balance;
    private AccountStatus status;

    // Command Handler
    public void openAccount(String accountNumber, BigDecimal initialBalance) {
        applyChange(new AccountOpenedEvent(
            UUID.randomUUID().toString(),
            accountNumber,
            initialBalance
        ));
    }

    public void deposit(BigDecimal amount) {
        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        applyChange(new MoneyDepositedEvent(getId(), amount));
    }

    public void withdraw(BigDecimal amount) {
        if (balance.compareTo(amount) < 0) {
            throw new InsufficientFundsException();
        }

        applyChange(new MoneyWithdrawnEvent(getId(), amount));
    }

    // Event Handlers
    @Override
    protected void apply(DomainEvent event) {
        if (event instanceof AccountOpenedEvent e) {
            this.setId(e.getAccountId());
            this.accountNumber = e.getAccountNumber();
            this.balance = e.getInitialBalance();
            this.status = AccountStatus.ACTIVE;

        } else if (event instanceof MoneyDepositedEvent e) {
            this.balance = this.balance.add(e.getAmount());

        } else if (event instanceof MoneyWithdrawnEvent e) {
            this.balance = this.balance.subtract(e.getAmount());
        }
    }
}

// Event Store Repository
@Repository
public class EventStoreRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Transactional
    public void save(String aggregateId, List<DomainEvent> events, Long expectedVersion) {
        // Version 체크 (Optimistic Locking)
        Long currentVersion = getCurrentVersion(aggregateId);
        if (!expectedVersion.equals(currentVersion)) {
            throw new ConcurrencyException("Version mismatch");
        }

        // 이벤트 저장
        long version = expectedVersion;
        for (DomainEvent event : events) {
            version++;
            saveEvent(aggregateId, event, version);
        }
    }

    public List<DomainEvent> getEvents(String aggregateId) {
        String sql = """
            SELECT * FROM event_store
            WHERE aggregate_id = ?
            ORDER BY event_version
            """;

        return jdbcTemplate.query(sql,
            new Object[]{aggregateId},
            (rs, rowNum) -> deserializeEvent(rs.getString("event_data"))
        );
    }
}
```

**📝 Day 13-14 산출물**

- 면접 Q&A 10개: 분산 트랜잭션
- 블로그 포스트: "Saga 패턴 실전 적용기"

---

#### **Day 15: Week 3 종합 정리 (5시간)**

**📚 복습 (2시간)**

- 트랜잭션 전파 레벨 플로우차트
- 격리 수준별 문제 상황 매트릭스
- 분산 트랜잭션 패턴 비교표

**🔬 종합 프로젝트 (3시간)**

**E-Commerce 시스템 트랜잭션 설계**

```java
// 주문 처리 전체 플로우
@Service
@Slf4j
public class ECommerceOrderService {

    // 메인 주문 처리 (Saga + Outbox)
    @Transactional
    public OrderResult processOrder(OrderRequest request) {
        // 1. 로컬 트랜잭션으로 주문 생성
        Order order = createOrderWithOutbox(request);

        // 2. Saga로 분산 처리
        CompletableFuture<SagaResult> sagaFuture =
            CompletableFuture.supplyAsync(() ->
                sagaOrchestrator.process(order)
            );

        // 3. 타임아웃 설정
        try {
            SagaResult result = sagaFuture.get(30, TimeUnit.SECONDS);

            if (result.isSuccess()) {
                updateOrderStatus(order.getId(), OrderStatus.CONFIRMED);
                return OrderResult.success(order);
            } else {
                updateOrderStatus(order.getId(), OrderStatus.FAILED);
                return OrderResult.failure(result.getError());
            }

        } catch (TimeoutException e) {
            // 타임아웃 시 보상 처리
            sagaOrchestrator.compensate(order.getId());
            updateOrderStatus(order.getId(), OrderStatus.TIMEOUT);
            return OrderResult.timeout();
        }
    }
}
```

**📝 Week 3 최종 산출물**

- 면접 Q&A 75개 (누적)
- 블로그 포스트 3편 (누적)
- GitHub: 분산 트랜잭션 샘플 프로젝트

---

### **Week 4: Database별 트랜잭션과 성능**

#### **Day 16-17: Database별 트랜잭션 특성 (10시간)**

**📚 이론 학습 (6시간)**

**2시간: MySQL InnoDB 트랜잭션**

```
[MVCC (Multi-Version Concurrency Control)]
각 트랜잭션이 자신만의 스냅샷 보유

구현 방식:
- Undo Log: 이전 버전 저장
- Read View: 트랜잭션 시작 시점 스냅샷
- Transaction ID: 각 트랜잭션 고유 ID

버전 가시성:
- 자신의 트랜잭션 ID보다 작은 것만 보임
- 커밋된 트랜잭션만 보임

[InnoDB Locking]
Record Lock: 특정 레코드만
Gap Lock: 레코드 사이 공간
Next-Key Lock: Record + Gap

Lock 발생 상황:
SELECT ... FOR UPDATE: X-Lock
SELECT ... LOCK IN SHARE MODE: S-Lock
UPDATE/DELETE: X-Lock + Gap Lock

[Deadlock Detection]
- Wait-for Graph 구성
- Cycle 감지 시 Deadlock
- 작은 트랜잭션 롤백
```

**2시간: MongoDB 트랜잭션**

```
[MongoDB 4.0+ Multi-Document Transaction]
Replica Set 트랜잭션:
- 단일 Replica Set 내
- ACID 보장
- Snapshot Isolation

Sharded Cluster 트랜잭션:
- 여러 Shard 걸친 트랜잭션
- 2PC 사용
- 성능 오버헤드

[Read/Write Concern]
Read Concern:
- local: 로컬 데이터
- majority: 과반수 복제 확인
- snapshot: 트랜잭션 스냅샷
- linearizable: 선형화 가능

Write Concern:
- w: 복제 개수
- j: Journal 기록
- wtimeout: 타임아웃

[트랜잭션 제약]
- 60초 기본 타임아웃
- 16MB 문서 크기 제한
- 콜렉션/인덱스 생성 불가
```

**2시간: Redis 트랜잭션**

```
[MULTI/EXEC/WATCH]
MULTI: 트랜잭션 시작
EXEC: 트랜잭션 실행
DISCARD: 트랜잭션 취소
WATCH: Optimistic Lock

특징:
- All or Nothing (원자성)
- 격리 수준 없음
- 롤백 없음 (실패해도 계속)

[Lua Script]
진짜 원자성 보장:
- 단일 스레드 실행
- 중간에 다른 명령 끼어들기 불가
- 복잡한 로직 가능

제약:
- 5초 타임아웃
- 디버깅 어려움
```

**🔬 실습: DB별 트랜잭션 검증 (4시간)**

**실습 1: MySQL Lock 동작 확인 (1시간 20분)**

```java
@Repository
public class MySQLLockTestRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // Record Lock 테스트
    public void recordLockTest(Long id) {
        // X-Lock 획득
        jdbcTemplate.queryForObject(
            "SELECT * FROM accounts WHERE id = ? FOR UPDATE",
            new Object[]{id},
            (rs, rowNum) -> rs.getLong("balance")
        );

        // Lock 정보 확인
        List<Map<String, Object>> locks = jdbcTemplate.queryForList(
            "SELECT * FROM information_schema.INNODB_LOCKS"
        );

        locks.forEach(lock -> {
            log.info("Lock Mode: {}, Type: {}, Index: {}",
                lock.get("lock_mode"),
                lock.get("lock_type"),
                lock.get("lock_index"));
        });
    }

    // Gap Lock 테스트
    public void gapLockTest(BigDecimal minBalance, BigDecimal maxBalance) {
        // 범위 조회 + Lock
        jdbcTemplate.query(
            "SELECT * FROM accounts WHERE balance BETWEEN ? AND ? FOR UPDATE",
            new Object[]{minBalance, maxBalance},
            (rs, rowNum) -> rs.getLong("id")
        );

        // 다른 세션에서 INSERT 시도 시 대기
        // INSERT INTO accounts (balance) VALUES (?) -- Gap 내 값
    }

    // Deadlock 유발
    public void causeDeadlock(Long id1, Long id2, boolean reverse) {
        if (!reverse) {
            jdbcTemplate.update("UPDATE accounts SET balance = balance + 100 WHERE id = ?", id1);
            Thread.sleep(100);
            jdbcTemplate.update("UPDATE accounts SET balance = balance - 100 WHERE id = ?", id2);
        } else {
            jdbcTemplate.update("UPDATE accounts SET balance = balance - 100 WHERE id = ?", id2);
            Thread.sleep(100);
            jdbcTemplate.update("UPDATE accounts SET balance = balance + 100 WHERE id = ?", id1);
        }
    }
}

// 테스트
@Test
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void MySQL_Deadlock_테스트() {
    ExecutorService executor = Executors.newFixedThreadPool(2);

    Future<?> tx1 = executor.submit(() -> {
        transactionTemplate.execute(status -> {
            repository.causeDeadlock(1L, 2L, false);
            return null;
        });
    });

    Future<?> tx2 = executor.submit(() -> {
        transactionTemplate.execute(status -> {
            repository.causeDeadlock(1L, 2L, true);
            return null;
        });
    });

    // 한 트랜잭션은 Deadlock 예외 발생
    // ERROR 1213 (40001): Deadlock found
}
```

**실습 2: MongoDB 트랜잭션 구현 (1시간 20분)**

```java
@Configuration
public class MongoTransactionConfig {

    @Bean
    public MongoTransactionManager mongoTransactionManager(MongoDatabaseFactory factory) {
        return new MongoTransactionManager(factory);
    }
}

@Service
@Slf4j
public class MongoTransactionService {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Transactional(transactionManager = "mongoTransactionManager")
    public void transferMoney(String fromId, String toId, BigDecimal amount) {
        // MongoDB 트랜잭션 시작
        log.info("Session: {}", mongoTemplate.getSessionSynchronization());

        // 계좌 조회 및 수정
        Query fromQuery = Query.query(Criteria.where("_id").is(fromId));
        Update withdrawUpdate = new Update().inc("balance", amount.negate());

        UpdateResult fromResult = mongoTemplate.updateFirst(
            fromQuery, withdrawUpdate, Account.class
        );

        if (fromResult.getModifiedCount() == 0) {
            throw new RuntimeException("Source account not found");
        }

        Query toQuery = Query.query(Criteria.where("_id").is(toId));
        Update depositUpdate = new Update().inc("balance", amount);

        UpdateResult toResult = mongoTemplate.updateFirst(
            toQuery, depositUpdate, Account.class
        );

        if (toResult.getModifiedCount() == 0) {
            throw new RuntimeException("Target account not found");
        }

        // 트랜잭션 내 읽기
        Account from = mongoTemplate.findOne(fromQuery, Account.class);
        if (from.getBalance().compareTo(BigDecimal.ZERO) < 0) {
            throw new InsufficientFundsException();
        }
    }

    // Read/Write Concern 설정
    public void configurableConcernOperation() {
        MongoCollection<Document> collection = mongoTemplate
            .getCollection("accounts")
            .withReadConcern(ReadConcern.MAJORITY)
            .withWriteConcern(WriteConcern.MAJORITY);

        // Majority 읽기/쓰기
        Document doc = collection.find(eq("_id", "123")).first();
        collection.insertOne(new Document("test", "data"));
    }
}

// DocumentDB 제약사항 처리
@Service
public class DocumentDBCompatibleService {

    // DocumentDB는 일부 MongoDB 기능 미지원
    @Transactional
    public void documentDBTransaction() {
        try {
            // 트랜잭션 처리
        } catch (MongoCommandException e) {
            if (e.getErrorCode() == 303) {
                // DocumentDB 트랜잭션 미지원
                log.warn("Transaction not supported in DocumentDB");
                // 대체 로직 실행
                performWithoutTransaction();
            }
        }
    }
}
```

**실습 3: Redis 트랜잭션과 Lua (1시간 20분)**

```java
@Service
@Slf4j
public class RedisTransactionService {

    @Autowired
    private StringRedisTemplate redisTemplate;

    // MULTI/EXEC 트랜잭션
    public void multiExecTransaction(String accountId, BigDecimal amount) {
        redisTemplate.execute(new SessionCallback<List<Object>>() {
            @Override
            public List<Object> execute(RedisOperations operations)
                    throws DataAccessException {

                // WATCH - Optimistic Lock
                operations.watch("account:" + accountId);

                // 현재 잔액 확인
                String balanceStr = (String) operations.opsForValue()
                    .get("account:" + accountId);
                BigDecimal balance = new BigDecimal(balanceStr);

                // 트랜잭션 시작
                operations.multi();

                // 잔액 체크
                if (balance.compareTo(amount) >= 0) {
                    BigDecimal newBalance = balance.subtract(amount);
                    operations.opsForValue()
                        .set("account:" + accountId, newBalance.toString());
                    operations.opsForList()
                        .leftPush("transactions:" + accountId,
                            "Withdraw: " + amount);
                }

                // 실행
                return operations.exec();
            }
        });
    }

    // Lua Script 원자성 보장
    public Long rateLimiter(String key, int limit, int window) {
        String luaScript = """
            local key = KEYS[1]
            local limit = tonumber(ARGV[1])
            local window = tonumber(ARGV[2])
            local current = redis.call('GET', key)

            if current == false then
                redis.call('SET', key, 1)
                redis.call('EXPIRE', key, window)
                return 1
            elseif tonumber(current) < limit then
                return redis.call('INCR', key)
            else
                return -1
            end
            """;

        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(luaScript);
        script.setResultType(Long.class);

        return redisTemplate.execute(
            script,
            Collections.singletonList(key),
            String.valueOf(limit),
            String.valueOf(window)
        );
    }

    // 분산 Lock (Redlock 알고리즘)
    public boolean acquireDistributedLock(String resource, String token, int ttl) {
        String luaScript = """
            if redis.call('SET', KEYS[1], ARGV[1], 'NX', 'EX', ARGV[2]) then
                return 1
            else
                return 0
            end
            """;

        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(luaScript);
        script.setResultType(Long.class);

        Long result = redisTemplate.execute(
            script,
            Collections.singletonList("lock:" + resource),
            token,
            String.valueOf(ttl)
        );

        return result != null && result == 1;
    }

    public boolean releaseDistributedLock(String resource, String token) {
        String luaScript = """
            if redis.call('GET', KEYS[1]) == ARGV[1] then
                return redis.call('DEL', KEYS[1])
            else
                return 0
            end
            """;

        DefaultRedisScript<Long> script = new DefaultRedisScript<>();
        script.setScriptText(luaScript);
        script.setResultType(Long.class);

        Long result = redisTemplate.execute(
            script,
            Collections.singletonList("lock:" + resource),
            token
        );

        return result != null && result == 1;
    }
}
```

**📝 Day 16-17 산출물**

- 면접 Q&A 12개: DB별 트랜잭션
- 실습 코드: 각 DB 트랜잭션 테스트

---

#### **Day 18-19: 트랜잭션 성능 최적화 (10시간)**

**📚 이론 학습 (6시간)**

**3시간: Connection Pool 최적화**

```
[HikariCP 핵심 파라미터]
minimumIdle:
- 유지할 최소 Connection
- 트래픽 적을 때도 유지
- 권장: 최소 2-5개

maximumPoolSize:
- 최대 Connection 수
- 계산: connections = ((core_count * 2) + effective_spindle_count)
- 실무: 10-20개

connectionTimeout:
- Connection 획득 대기 시간
- 기본 30초
- 짧으면 빠른 실패, 길면 대기

idleTimeout:
- Idle Connection 유지 시간
- minimumIdle < pool size일 때만 적용
- 기본 10분

maxLifetime:
- Connection 최대 수명
- DB wait_timeout보다 짧게
- 기본 30분

[Connection Leak 감지]
leakDetectionThreshold:
- Connection 사용 시간 임계값
- 초과 시 경고 로그
- 권장: 2-5초

[Pool Sizing 전략]
1. 벤치마킹으로 최적값 찾기
2. Little's Law 적용
   Pool Size = TPS × Response Time
3. 모니터링 후 조정
```

**3시간: Batch 처리와 Bulk Operation**

```
[JDBC Batch]
장점:
- 네트워크 왕복 감소
- Statement 재사용
- 서버 파싱 감소

설정:
rewriteBatchedStatements=true (MySQL)
- 여러 INSERT를 하나로 합침
- INSERT INTO t VALUES (1),(2),(3)

Batch Size:
- 너무 작으면 효과 없음
- 너무 크면 메모리 문제
- 권장: 50-1000

[JPA Batch]
hibernate.jdbc.batch_size: 50
hibernate.order_inserts: true
hibernate.order_updates: true
hibernate.jdbc.batch_versioned_data: true

주의사항:
- IDENTITY 생성 전략 불가
- SEQUENCE + allocation size 사용

[Bulk Operation]
JPA Criteria Bulk Update:
- 영속성 컨텍스트 우회
- 1차 캐시 무효화 필요

Native Query:
- 대량 데이터 직접 처리
- 영속성 컨텍스트 수동 동기화
```

**🔬 실습: 성능 최적화 (4시간)**

**실습 1: Connection Pool 튜닝 (1시간 20분)**

```java
@Configuration
public class OptimizedDataSourceConfig {

    @Bean
    @Primary
    public DataSource optimizedDataSource() {
        HikariConfig config = new HikariConfig();

        // 기본 설정
        config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        config.setUsername("root");
        config.setPassword("password");

        // Pool 크기 최적화
        int cpuCores = Runtime.getRuntime().availableProcessors();
        config.setMinimumIdle(cpuCores);
        config.setMaximumPoolSize(cpuCores * 2);

        // 타임아웃 설정
        config.setConnectionTimeout(3000);  // 3초
        config.setIdleTimeout(600000);      // 10분
        config.setMaxLifetime(1800000);     // 30분
        config.setLeakDetectionThreshold(5000); // 5초

        // 성능 설정
        config.setConnectionTestQuery("SELECT 1");
        config.setValidationTimeout(2000);

        // Connection 초기화
        config.setConnectionInitSql("SET NAMES utf8mb4");

        // 메트릭 수집
        config.setMetricRegistry(metricRegistry());

        return new HikariDataSource(config);
    }

    @Bean
    public MetricRegistry metricRegistry() {
        MetricRegistry registry = new MetricRegistry();

        // 메트릭 리포터 설정
        ConsoleReporter reporter = ConsoleReporter.forRegistry(registry)
            .convertRatesTo(TimeUnit.SECONDS)
            .convertDurationsTo(TimeUnit.MILLISECONDS)
            .build();

        reporter.start(1, TimeUnit.MINUTES);

        return registry;
    }
}

// Pool 모니터링
@Component
@Slf4j
public class PoolMonitor {

    @Autowired
    private DataSource dataSource;

    @Scheduled(fixedDelay = 10000)
    public void monitorPool() {
        if (dataSource instanceof HikariDataSource) {
            HikariDataSource hikari = (HikariDataSource) dataSource;
            HikariPoolMXBean pool = hikari.getHikariPoolMXBean();

            log.info("Pool Stats - Active: {}, Idle: {}, Waiting: {}, Total: {}",
                pool.getActiveConnections(),
                pool.getIdleConnections(),
                pool.getThreadsAwaitingConnection(),
                pool.getTotalConnections());

            // 경고 임계값
            if (pool.getActiveConnections() > hikari.getMaximumPoolSize() * 0.8) {
                log.warn("Pool usage high: {}%",
                    (pool.getActiveConnections() * 100) / hikari.getMaximumPoolSize());
            }

            if (pool.getThreadsAwaitingConnection() > 0) {
                log.warn("Threads waiting for connection: {}",
                    pool.getThreadsAwaitingConnection());
            }
        }
    }
}
```

**실습 2: Batch Insert 최적화 (1시간 20분)**

````java
@Service
@Slf4j
public class BatchInsertService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private EntityManager em;

    // JDBC Batch
    public void jdbcBatchInsert(List<Order> orders) {
        String sql = "INSERT INTO orders (customer_id, amount, status, created_at) VALUES (?, ?, ?, ?)";

        StopWatch watch = new StopWatch();
        watch.start();

        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                Order order = orders.get(i);
                ps.setLong(1, order.getCustomerId());
                ps.setBigDecimal(2, order.getAmount());
                ps.setString(3, order.getStatus().name());
                ps.setTimestamp(4, Timestamp.valueOf(order.getCreatedAt()));
            }

            @Override
            public int getBatchSize() {
                return orders.size();
            }
        });

        watch.stop();
        log.info("JDBC Batch: {} records in {}ms",
            orders.size(), watch.getTotalTimeMillis());
    }

    // JPA Batch
    @Transactional
    public void jpaBatchInsert(List<Order> orders) {
        StopWatch watch = new StopWatch();
        watch.start();

        int batchSize = 50;

        for (int i = 0; i < orders.size(); i++) {
            em.persist(orders.get(i));

            if (i > 0 && i % batchSize == 0) {
                em.flush();
                em.clear();  // 1차 캐시 정리
            }
        }

        em.flush();
        em.clear();

        watch.stop();
        log.info("JPA Batch: {} records in {}ms",
            orders.size(), watch.getTotalTimeMillis());
    }

    // Native SQL Bulk Insert
    @Transactional
    public void nativeBulkInsert(List<Order> orders) {
        StopWatch watch = new StopWatch();
        watch.start();

        StringBuilder sql = new StringBuilder(
            "INSERT INTO orders (customer_id, amount, status, created_at) VALUES ");

        for (int i = 0; i < orders.size(); i++) {
            sql.append("(?, ?, ?, ?)");
            if (i < orders.size() - 1) {
                sql.append(",");
            }
        }

        Query query = em.createNativeQuery(sql.toString());

        int paramIndex = 1;
        for (Order order : orders) {
            query.setParameter(paramIndex++, order.getCustomerId());
            query.setParameter(paramIndex++, order.getAmount());
            query.setParameter(paramIndex++, order.getStatus().name());
            query.setParameter(paramIndex++, order.getCreatedAt());
        }

        query.executeUpdate();

        watch.stop();
        log.info("Native Bulk: {} records in {}ms",
            orders.size(), watch.getTotalTimeMillis());
    }

    // 성능 비교 테스트
    public void compareBatchMethods() {
        List<Order> orders = generateOrders(10000);

        // 1. Loop Insert (최악)
        StopWatch watch = new StopWatch("Batch Comparison");

        watch.start("Loop Insert");
        orders.forEach(order -> orderRepository.save(order));
        watch.stop();

        // 2. JDBC Batch
        watch.start("JDBC Batch");
        jdb
        ```java
        // 2. JDBC Batch
        watch.start("JDBC Batch");
        jdbcBatchInsert(orders);
        watch.stop();

        // 3. JPA Batch
        watch.start("JPA Batch");
        jpaBatchInsert(orders);
        watch.stop();

        // 4. Native Bulk
        watch.start("Native Bulk");
        nativeBulkInsert(orders);
        watch.stop();

        log.info("Performance Results:\n{}", watch.prettyPrint());
        // 결과: Native Bulk > JDBC Batch > JPA Batch >> Loop Insert
    }
}
````

**실습 3: Lock 최소화 전략 (1시간 20분)**

```java
@Service
@Slf4j
public class LockOptimizationService {

    @Autowired
    private AccountRepository accountRepository;

    // Pessimistic Lock (비효율)
    @Transactional
    public void pessimisticTransfer(Long fromId, Long toId, BigDecimal amount) {
        StopWatch watch = new StopWatch();
        watch.start();

        // Lock 획득
        Account from = accountRepository.findByIdWithPessimisticLock(fromId);
        Account to = accountRepository.findByIdWithPessimisticLock(toId);

        // 비즈니스 로직
        Thread.sleep(100);  // 시뮬레이션

        from.withdraw(amount);
        to.deposit(amount);

        accountRepository.save(from);
        accountRepository.save(to);

        watch.stop();
        log.info("Pessimistic Lock: {}ms", watch.getTotalTimeMillis());
    }

    // Optimistic Lock (효율적)
    @Transactional
    public void optimisticTransfer(Long fromId, Long toId, BigDecimal amount) {
        int maxRetries = 3;
        int attempt = 0;

        while (attempt < maxRetries) {
            try {
                StopWatch watch = new StopWatch();
                watch.start();

                // Lock 없이 조회
                Account from = accountRepository.findById(fromId);
                Account to = accountRepository.findById(toId);

                // 비즈니스 로직
                from.withdraw(amount);
                to.deposit(amount);

                // Version 체크하며 저장
                accountRepository.save(from);
                accountRepository.save(to);

                watch.stop();
                log.info("Optimistic Lock Success: {}ms", watch.getTotalTimeMillis());
                break;

            } catch (OptimisticLockingFailureException e) {
                attempt++;
                log.warn("Optimistic lock failed, attempt: {}", attempt);

                if (attempt >= maxRetries) {
                    throw new RetryExhaustedException("Max retries reached");
                }

                // Exponential backoff
                Thread.sleep((long) Math.pow(2, attempt) * 100);
            }
        }
    }

    // Lock-Free 설계 (가장 효율적)
    @Transactional
    public void lockFreeTransfer(Long fromId, Long toId, BigDecimal amount) {
        // Event Sourcing 방식
        TransferEvent event = new TransferEvent(fromId, toId, amount);
        eventStore.append(event);

        // 비동기 처리
        eventPublisher.publish(event);

        // 즉시 반환 (Lock 없음)
    }

    // Row-Level Lock 최소화
    @Transactional
    public void minimizeLockScope() {
        // Bad: 전체 트랜잭션 동안 Lock
        @Transactional
        public void badExample(Long id) {
            Account account = repository.findByIdForUpdate(id);
            // Lock 유지하면서 외부 API 호출 (위험!)
            callExternalApi();
            account.update();
        }

        // Good: Lock 구간 최소화
        public void goodExample(Long id) {
            // 외부 API 먼저 호출
            ApiResult result = callExternalApi();

            // 트랜잭션과 Lock은 최소 구간만
            transactionTemplate.execute(status -> {
                Account account = repository.findByIdForUpdate(id);
                account.update(result);
                repository.save(account);
                return null;
            });
        }
    }
}

// Lock 모니터링
@Component
public class LockMonitor {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void monitorLocks() {
        // MySQL InnoDB Lock 모니터링
        String sql = """
            SELECT
                waiting_trx_id,
                waiting_pid,
                waiting_query,
                blocking_trx_id,
                blocking_pid,
                blocking_query
            FROM sys.innodb_lock_waits
            """;

        List<Map<String, Object>> locks = jdbcTemplate.queryForList(sql);

        locks.forEach(lock -> {
            log.warn("Lock Wait Detected:");
            log.warn("  Waiting: {} - {}",
                lock.get("waiting_trx_id"),
                lock.get("waiting_query"));
            log.warn("  Blocking: {} - {}",
                lock.get("blocking_trx_id"),
                lock.get("blocking_query"));
        });
    }
}
```

**📝 Day 18-19 산출물**

- 면접 Q&A 10개: 트랜잭션 성능
- 블로그 포스트: "트랜잭션 성능 최적화 전략"

---

#### **Day 20: Week 4 종합 프로젝트 (5시간)**

**🔬 종합 프로젝트: Multi-DB 트랜잭션 시스템 (5시간)**

```java
// 종합 시스템: MySQL + MongoDB + Redis
@Service
@Slf4j
public class MultiDBTransactionService {

    @Autowired
    private PlatformTransactionManager mysqlTxManager;

    @Autowired
    private MongoTransactionManager mongoTxManager;

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    // 주문 처리: MySQL(주문) + MongoDB(로그) + Redis(캐시)
    public OrderResult processOrder(OrderRequest request) {

        // 1. MySQL 트랜잭션으로 주문 생성
        Order order = createOrderInMySQL(request);

        // 2. MongoDB에 이벤트 로그 (Best Effort)
        try {
            logEventToMongoDB(order);
        } catch (Exception e) {
            log.warn("Failed to log to MongoDB", e);
            // 실패해도 계속 진행
        }

        // 3. Redis 캐시 업데이트
        updateRedisCache(order);

        // 4. Saga로 연관 서비스 처리
        CompletableFuture<Void> sagaResult = processSagaAsync(order);

        return OrderResult.builder()
            .orderId(order.getId())
            .status(OrderStatus.PENDING)
            .sagaFuture(sagaResult)
            .build();
    }

    @Transactional("mysqlTxManager")
    private Order createOrderInMySQL(OrderRequest request) {
        // Outbox 패턴 적용
        Order order = new Order(request);
        orderRepository.save(order);

        OutboxEvent event = new OutboxEvent(
            order.getId(),
            "OrderCreated",
            toJson(order)
        );
        outboxRepository.save(event);

        return order;
    }

    private void logEventToMongoDB(Order order) {
        TransactionTemplate mongoTx = new TransactionTemplate(mongoTxManager);

        mongoTx.execute(status -> {
            EventLog log = new EventLog();
            log.setAggregateId(order.getId().toString());
            log.setEventType("OrderCreated");
            log.setEventData(toJson(order));
            log.setOccurredAt(LocalDateTime.now());

            mongoTemplate.save(log);
            return null;
        });
    }

    private void updateRedisCache(Order order) {
        // Lua Script로 원자적 업데이트
        String script = """
            redis.call('HSET', KEYS[1], 'order', ARGV[1])
            redis.call('ZADD', KEYS[2], ARGV[2], ARGV[3])
            redis.call('EXPIRE', KEYS[1], 3600)
            return 1
            """;

        redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Arrays.asList(
                "order:" + order.getId(),
                "recent_orders"
            ),
            toJson(order),
            String.valueOf(System.currentTimeMillis()),
            order.getId().toString()
        );
    }

    private CompletableFuture<Void> processSagaAsync(Order order) {
        return CompletableFuture.runAsync(() -> {
            try {
                // 재고 확인 (MongoDB)
                checkInventoryInMongoDB(order.getItems());

                // 결제 처리 (MySQL)
                processPaymentInMySQL(order);

                // 배송 준비 (MongoDB)
                prepareShipmentInMongoDB(order);

            } catch (Exception e) {
                // 보상 트랜잭션
                compensateOrder(order.getId());
                throw new SagaFailedException(e);
            }
        });
    }
}

// 성능 모니터링
@Component
@Slf4j
public class TransactionMetrics {

    private final MeterRegistry meterRegistry;

    public TransactionMetrics(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }

    @EventListener
    public void handleTransactionEvent(TransactionEvent event) {
        // 트랜잭션 메트릭 수집
        Timer.Sample sample = Timer.start(meterRegistry);

        try {
            // 트랜잭션 처리
            processTransaction(event);

            // 성공 메트릭
            meterRegistry.counter("transaction.success",
                "type", event.getType(),
                "database", event.getDatabase()
            ).increment();

        } catch (Exception e) {
            // 실패 메트릭
            meterRegistry.counter("transaction.failure",
                "type", event.getType(),
                "database", event.getDatabase(),
                "error", e.getClass().getSimpleName()
            ).increment();

        } finally {
            // 실행 시간 메트릭
            sample.stop(Timer.builder("transaction.duration")
                .tag("type", event.getType())
                .tag("database", event.getDatabase())
                .register(meterRegistry));
        }
    }
}
```

**📝 Week 4 최종 산출물**

- 면접 Q&A 100개 (누적)
- 블로그 포스트 4편 (누적)
- GitHub: Multi-DB 트랜잭션 프로젝트

---

## **Phase 3: 실전 적용과 트러블슈팅 (Week 5-6)**

### **Week 5: JPA와 트랜잭션**

#### **Day 21-23: JPA 트랜잭션 심화 (15시간)**

**📚 이론 학습 (9시간)**

**3시간: Persistence Context와 트랜잭션**

```
[영속성 컨텍스트 생명주기]
Transaction-scoped (기본):
- 트랜잭션 시작 → PC 생성
- 트랜잭션 종료 → PC 종료
- Thread-safe

Extended (Stateful):
- 여러 트랜잭션 걸쳐 유지
- Thread-unsafe
- 거의 사용 안 함

[1차 캐시와 동일성 보장]
같은 트랜잭션 내:
- 같은 ID → 같은 객체
- DB 조회 최소화
- 동일성(==) 보장

[Dirty Checking]
- 엔티티 스냅샷 저장
- 트랜잭션 커밋 시 비교
- 변경 감지 → UPDATE 쿼리

[Write Behind]
- 쿼리를 모아서 실행
- 트랜잭션 커밋 시점
- 순서 보장
```

**3시간: Lazy Loading과 트랜잭션**

```
[Proxy 초기화]
- 실제 사용 시점에 초기화
- 영속성 컨텍스트 필요
- LazyInitializationException

[N+1 Problem]
원인: Lazy Loading in Loop
해결:
1. Fetch Join
2. @EntityGraph
3. Batch Fetch
4. DTO Projection

[Open Session in View]
장점:
- View에서도 Lazy Loading
- 편리함

단점:
- DB Connection 오래 유지
- 성능 저하
- 레이어 경계 모호
```

**3시간: JPA Lock과 동시성**

```
[Optimistic Lock]
@Version 사용:
- Version 컬럼 자동 증가
- 커밋 시 Version 체크
- 충돌 시 예외

[Pessimistic Lock]
LockModeType.PESSIMISTIC_READ:
- Shared Lock
- 읽기는 가능, 쓰기 대기

LockModeType.PESSIMISTIC_WRITE:
- Exclusive Lock
- 읽기/쓰기 모두 대기

LockModeType.PESSIMISTIC_FORCE_INCREMENT:
- Version 강제 증가
```

**🔬 실습: JPA 트랜잭션 마스터 (6시간)**

**실습 1: 영속성 컨텍스트 동작 확인 (2시간)**

```java
@Service
@Transactional
@Slf4j
public class PersistenceContextService {

    @PersistenceContext
    private EntityManager em;

    public void demonstratePersistenceContext() {
        // 1. 영속 상태
        User user = new User("John");
        em.persist(user);  // 영속화
        log.info("After persist - Contains: {}", em.contains(user));

        // 2. 1차 캐시 확인
        User cached = em.find(User.class, user.getId());
        log.info("Same instance: {}", user == cached);  // true

        // 3. Dirty Checking
        user.setName("John Doe");
        // 별도 save() 불필요 - 자동 UPDATE

        // 4. Flush 시점 제어
        em.flush();  // 즉시 쿼리 실행
        log.info("After flush");

        // 5. Clear - 1차 캐시 비우기
        em.clear();
        User cleared = em.find(User.class, user.getId());
        log.info("After clear - Same: {}", user == cleared);  // false

        // 6. Detach - 특정 엔티티만 분리
        em.detach(cleared);
        log.info("After detach - Contains: {}", em.contains(cleared));

        // 7. Merge - 분리된 엔티티 재영속화
        cleared.setName("Jane");
        User merged = em.merge(cleared);
        log.info("After merge - Contains: {}", em.contains(merged));
    }

    // FlushMode 제어
    public void demonstrateFlushMode() {
        // AUTO (기본): 쿼리 실행 전, 트랜잭션 커밋 시
        em.setFlushMode(FlushModeType.AUTO);

        User user = new User("Test");
        em.persist(user);

        // JPQL 실행 시 자동 flush
        List<User> users = em.createQuery(
            "SELECT u FROM User u", User.class
        ).getResultList();

        // COMMIT: 트랜잭션 커밋 시에만
        em.setFlushMode(FlushModeType.COMMIT);

        user.setName("Changed");

        // flush 안 됨 - 변경사항 조회 안 됨
        users = em.createQuery(
            "SELECT u FROM User u WHERE u.name = 'Changed'", User.class
        ).getResultList();

        log.info("Found users: {}", users.size());  // 0
    }
}
```

**실습 2: N+1 문제 해결 (2시간)**

```java
@Entity
public class Team {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "team", fetch = FetchType.LAZY)
    private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;
}

@Repository
public interface TeamRepository extends JpaRepository<Team, Long> {

    // N+1 발생
    @Query("SELECT t FROM Team t")
    List<Team> findAllBasic();

    // 해결 1: Fetch Join
    @Query("SELECT DISTINCT t FROM Team t LEFT JOIN FETCH t.members")
    List<Team> findAllWithMembers();

    // 해결 2: EntityGraph
    @EntityGraph(attributePaths = {"members"})
    @Query("SELECT t FROM Team t")
    List<Team> findAllWithEntityGraph();
}

@Service
@Transactional(readOnly = true)
@Slf4j
public class TeamService {

    public void demonstrateN1Problem() {
        // N+1 발생 케이스
        List<Team> teams = teamRepository.findAllBasic();

        // 1번 쿼리: SELECT * FROM team

        for (Team team : teams) {
            // N번 쿼리: SELECT * FROM member WHERE team_id = ?
            log.info("Team: {}, Members: {}",
                team.getName(),
                team.getMembers().size());
        }
    }

    public void solveFetchJoin() {
        // Fetch Join으로 해결
        List<Team> teams = teamRepository.findAllWithMembers();

        // 1번 쿼리로 모든 데이터 로드
        // SELECT t.*, m.* FROM team t LEFT JOIN member m ON t.id = m.team_id

        for (Team team : teams) {
            log.info("Team: {}, Members: {}",
                team.getName(),
                team.getMembers().size());
        }
    }

    public void solveBatchFetch() {
        // application.yml
        // spring.jpa.properties.hibernate.default_batch_fetch_size: 100

        List<Team> teams = teamRepository.findAllBasic();

        // 1번: SELECT * FROM team
        // 1번: SELECT * FROM member WHERE team_id IN (?, ?, ?, ...)

        for (Team team : teams) {
            team.getMembers().size();  // IN 쿼리로 한 번에 로드
        }
    }

    // DTO Projection으로 해결
    @Query("""
        SELECT new com.example.TeamDTO(
            t.id, t.name,
            (SELECT COUNT(m) FROM Member m WHERE m.team = t)
        )
        FROM Team t
        """)
    List<TeamDTO> findTeamStatistics();
}
```

**실습 3: JPA Lock 전략 (2시간)**

```java
@Entity
@Data
public class Product {
    @Id @GeneratedValue
    private Long id;

    private String name;
    private Integer quantity;

    @Version
    private Long version;  // Optimistic Lock
}

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithPessimisticLock(@Param("id") Long id);

    @Lock(LockModeType.OPTIMISTIC_FORCE_INCREMENT)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithOptimisticForceIncrement(@Param("id") Long id);
}

@Service
@Slf4j
public class InventoryService {

    // Optimistic Lock 활용
    @Transactional
    public void decreaseStockOptimistic(Long productId, int quantity) {
        int retries = 3;

        while (retries > 0) {
            try {
                Product product = productRepository.findById(productId)
                    .orElseThrow();

                if (product.getQuantity() < quantity) {
                    throw new InsufficientStockException();
                }

                product.setQuantity(product.getQuantity() - quantity);
                productRepository.save(product);

                log.info("Stock decreased. Version: {}", product.getVersion());
                break;

            } catch (OptimisticLockingFailureException e) {
                retries--;
                log.warn("Optimistic lock failed. Retries left: {}", retries);

                if (retries == 0) {
                    throw new StockUpdateFailedException("Max retries exceeded");
                }

                // Exponential backoff
                try {
                    Thread.sleep((long) Math.pow(2, 3 - retries) * 100);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }

    // Pessimistic Lock 활용
    @Transactional
    public void decreaseStockPessimistic(Long productId, int quantity) {
        // Lock 획득
        Product product = productRepository
            .findByIdWithPessimisticLock(productId)
            .orElseThrow();

        // 다른 트랜잭션은 대기

        if (product.getQuantity() < quantity) {
            throw new InsufficientStockException();
        }

        product.setQuantity(product.getQuantity() - quantity);
        // 트랜잭션 종료 시 Lock 해제
    }

    // Dead Lock 방지 - Lock 순서 보장
    @Transactional
    public void transferStock(Long fromId, Long toId, int quantity) {
        // ID 순서로 Lock 획득
        Long firstId = Math.min(fromId, toId);
        Long secondId = Math.max(fromId, toId);

        Product first = productRepository
            .findByIdWithPessimisticLock(firstId)
            .orElseThrow();

        Product second = productRepository
            .findByIdWithPessimisticLock(secondId)
            .orElseThrow();

        // 비즈니스 로직
        if (fromId.equals(firstId)) {
            first.setQuantity(first.getQuantity() - quantity);
            second.setQuantity(second.getQuantity() + quantity);
        } else {
            second.setQuantity(second.getQuantity() - quantity);
            first.setQuantity(first.getQuantity() + quantity);
        }
    }
}
```

**📝 Day 21-23 산출물**

- 면접 Q&A 15개: JPA 트랜잭션
- 실습 코드: N+1 해결 전략 모음

---

#### **Day 24-25: Kafka 트랜잭션 (10시간)**

**📚 이론 학습 (6시간)**

**3시간: Kafka Transaction API**

```
[Producer Transaction]
트랜잭션 시작:
- initTransactions()
- beginTransaction()
- send() 여러 번
- commitTransaction() or abortTransaction()

트랜잭션 ID:
- Producer 재시작 시에도 유지
- Exactly-once 보장
- Zombie Fencing

[Transaction Coordinator]
- 트랜잭션 상태 관리
- __transaction_state 토픽
- 2PC 프로토콜

[Consumer Transaction]
isolation.level:
- read_uncommitted: 모든 메시지
- read_committed: 커밋된 메시지만
```

**3시간: Exactly-Once Semantics**

```
[EOS 구성 요소]
1. Idempotent Producer
   - enable.idempotence=true
   - 중복 전송 방지

2. Transactional Producer
   - transactional.id 설정
   - 원자적 쓰기

3. Consumer 설정
   - isolation.level=read_committed
   - 트랜잭션 메시지만 읽기

[End-to-End EOS]
Producer → Kafka → Consumer → Kafka
모든 구간 exactly-once 보장
```

**🔬 실습: Kafka 트랜잭션 구현 (4시간)**

**실습 1: Transactional Producer (2시간)**

```java
@Configuration
public class KafkaTransactionConfig {

    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

        // 트랜잭션 설정
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "order-service-tx-");
        props.put(ProducerConfig.ACKS_CONFIG, "all");
        props.put(ProducerConfig.RETRIES_CONFIG, 3);
        props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 1);

        return new DefaultKafkaProducerFactory<>(props);
    }

    @Bean
    public KafkaTransactionManager kafkaTransactionManager(ProducerFactory<String, Object> pf) {
        KafkaTransactionManager tm = new KafkaTransactionManager<>(pf);
        tm.setTransactionIdPrefix("order-tx-");
        return tm;
    }

    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}

@Service
@Slf4j
public class KafkaTransactionService {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    // Spring 트랜잭션과 통합
    @Transactional("kafkaTransactionManager")
    public void sendTransactionalMessages(Order order) {
        try {
            // 여러 토픽에 원자적 전송
            kafkaTemplate.send("orders", order.getId().toString(), order);
            kafkaTemplate.send("inventory", order.getId().toString(),
                new InventoryEvent(order));
            kafkaTemplate.send("shipping", order.getId().toString(),
                new ShippingEvent(order));

            if (order.getAmount().compareTo(BigDecimal.valueOf(10000)) > 0) {
                throw new RuntimeException("Large order validation failed");
            }

            log.info("All messages sent successfully");

        } catch (Exception e) {
            log.error("Transaction will be rolled back", e);
            throw e;  // 트랜잭션 롤백
        }
    }

    // 수동 트랜잭션 관리
    public void manualTransaction() {
        kafkaTemplate.executeInTransaction(operations -> {
            operations.send("topic1", "message1");
            operations.send("topic2", "message2");

            if (shouldRollback()) {
                throw new KafkaException("Rollback transaction");
            }

            return true;
        });
    }

    // DB + Kafka 트랜잭션 통합
    @Transactional
    public void processWithOutbox(Order order) {
        // 1. DB에 저장
        orderRepository.save(order);

        // 2. Outbox 테이블에 이벤트 저장
        OutboxEvent event = new OutboxEvent(order);
        outboxRepository.save(event);

        // 3. 별도 스레드에서 Kafka로 발행
        // (Transactional Outbox Pattern)
    }
}
```

**실습 2: Exactly-Once 구현 (2시간)**

```java
@Component
@Slf4j
public class ExactlyOnceProcessor {

    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    // Consumer + Producer 트랜잭션
    @KafkaListener(
        topics = "input-topic",
        containerFactory = "kafkaListenerContainerFactory",
        transactionManager = "kafkaTransactionManager"
    )
    @Transactional("kafkaTransactionManager")
    public void processExactlyOnce(OrderEvent event,
                                  @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
                                  @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
                                  @Header(KafkaHeaders.OFFSET) long offset) {

        log.info("Processing: {} from {}:{} offset {}",
            event, topic, partition, offset);

        try {
            // 처리 로직
            ProcessedEvent processed = processEvent(event);

            // 결과를 다른 토픽으로 전송
            kafkaTemplate.send("output-topic", processed);

            // DB 업데이트 (Kafka 트랜잭션과 별개)
            updateDatabase(processed);

        } catch (Exception e) {
            log.error("Processing failed, transaction will rollback", e);
            throw e;
        }
    }

    // Idempotent Consumer 구현
    @Component
    public class IdempotentConsumer {

        private final Set<String> processedIds = new ConcurrentHashMap<>();

        @KafkaListener(topics = "events")
        public void consume(Event event) {
            // 멱등성 체크
            String eventId = event.getId();

            if (!processedIds.add(eventId)) {
                log.warn("Duplicate event: {}", eventId);
                return;
            }

            try {
                // 비즈니스 로직
                processEvent(event);

            } catch (Exception e) {
                // 실패 시 ID 제거 (재시도 가능)
                processedIds.remove(eventId);
                throw e;
            }
        }
    }
}

// Consumer 설정
@Configuration
public class KafkaConsumerConfig {

    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "exactly-once-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);

        // Exactly-Once 설정
        props.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, Object>>
           kafkaListenerContainerFactory() {

        ConcurrentKafkaListenerContainerFactory<String, Object> factory =
            new ConcurrentKafkaListenerContainerFactory<>();

        factory.setConsumerFactory(consumerFactory());

        // 트랜잭션 설정
        factory.getContainerProperties().setTransactionManager(kafkaTransactionManager());
        factory.getContainerProperties().setAckMode(AckMode.MANUAL);

        // 에러 핸들링
        factory.setErrorHandler(new SeekToCurrentErrorHandler(
            new DeadLetterPublishingRecoverer(kafkaTemplate()),
            new FixedBackOff(1000L, 3)
        ));

        return factory;
    }
}
```

**📝 Day 24-25 산출물**

- 면접 Q&A 8개: Kafka 트랜잭션
- 블로그 포스트: "Kafka Exactly-Once 구현 가이드"

---

### **Week 6: 트러블슈팅과 모니터링**

#### **Day 26-28: 실전 트러블슈팅 (15시간)**

**📚 트러블슈팅 시나리오 (9시간)**

**3시간: 트랜잭션 관련 예외 처리**

```java
@Service
@Slf4j
public class TransactionTroubleshootingService {

    // Case 1: UnexpectedRollbackException
    @Transactional
    public void unexpectedRollbackCase() {
        try {
            innerService.methodThatThrowsException();
        } catch (Exception e) {
            // 예외를 잡아도 이미 rollback-only 마킹됨
            log.error("Caught exception", e);
        }
        // 커밋 시도 시 UnexpectedRollbackException 발생
    }

    // 해결: 새 트랜잭션으로 분리
    public void fixedUnexpectedRollback() {
        try {
            innerService.methodWithRequiresNew();
        } catch (Exception e) {
            log.error("Handled in separate transaction", e);
        }
        // 정상 커밋
    }

    // Case 2: LazyInitializationException
    public OrderDTO lazyInitProblem(Long orderId) {
        Order order = orderRepository.findById(orderId);
        // 트랜잭션 종료

        // View에서 접근 시 예외
        return new OrderDTO(
            order.getId(),
            order.getItems().size()  // LazyInitializationException
        );
    }

    // 해결 1: Fetch Join
    @Transactional(readOnly = true)
    public OrderDTO fixedWithFetchJoin(Long orderId) {
        Order order = orderRepository.findByIdWithItems(orderId);
        return new OrderDTO(order);
    }

    // 해결 2: DTO Projection
    public OrderDTO fixedWithProjection(Long orderId) {
        return orderRepository.findOrderDTOById(orderId);
    }

    // Case 3: TransactionRequiredException
    public void transactionRequiredProblem() {
        // 트랜잭션 없이 persist/merge/remove 호출
        User user = new User();
        entityManager.persist(user);  // TransactionRequiredException
    }

    // 해결: 트랜잭션 추가
    @Transactional
    public void fixedTransactionRequired() {
        User user = new User();
        entityManager.persist(user);  // 정상 동작
    }
}
```

**3시간: 데드락과 타임아웃**

```java
@Service
@Slf4j
public class DeadlockTroubleshootingService {

    // Deadlock 감지와 재시도
    @Retryable(
        value = {DeadlockLoserDataAccessException.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 100, multiplier = 2)
    )
    @Transactional(timeout = 5)
    public void retryableTransaction(Long id1, Long id2) {
        log.info("Attempting transaction for {} and {}", id1, id2);

        Account acc1 = accountRepository.findByIdForUpdate(id1);
        Account acc2 = accountRepository.findByIdForUpdate(id2);

        // 비즈니스 로직
        processAccounts(acc1, acc2);
    }

    // Lock Timeout 처리
    @Transactional
    public void handleLockTimeout(Long accountId) {
        try {
            Account account = accountRepository.findByIdWithLockTimeout(accountId, 3);
            // 3초 내 Lock 획득 못하면 예외

        } catch (PessimisticLockException e) {
            log.error("Lock timeout for account: {}", accountId);
            // 대체 로직 실행
            performAsyncProcessing(accountId);
        }
    }

    // Connection Pool 고갈
    @Component
    public class ConnectionPoolMonitor {

        @EventListener
        public void handlePoolExhaustion(ConnectionPoolExhaustedEvent event) {
            log.error("Connection pool exhausted!");

            // 긴급 조치
            // 1. 장시간 실행 쿼리 종료
            terminateLongRunningQueries();

            // 2. Pool 크기 동적 증가
            increasePoolSize();

            // 3. 알림 발송
            sendAlert("Connection pool exhausted", event);
        }
    }
}
```

**3시간: 성능 문제 진단**

```java
@Component
@Slf4j
public class PerformanceTroubleshootingService {

    // Slow Query 감지
    @EventListener
    public void detectSlowQueries(QueryExecutedEvent event) {
        if (event.getExecutionTime() > 1000) {  // 1초 이상
            log.warn("Slow query detected: {}ms\nSQL: {}",
                event.getExecutionTime(),
                event.getSql());

            // 실행 계획 분석
            analyzeExecutionPlan(event.getSql());
        }
    }

    // N+1 문제 자동 감지
    @Aspect
    @Component
    public class N1Detector {

        private ThreadLocal<Integer> queryCount = ThreadLocal.withInitial(() -> 0);

        @Before("@annotation(org.springframework.transaction.annotation.Transactional)")
        public void resetCounter() {
            queryCount.set(0);
        }

        @Before("execution(* org.springframework.jdbc.core.JdbcTemplate.query*(..))")
        public void countQuery() {
            queryCount.set(queryCount.get() + 1);
        }

        @After("@annotation(org.springframework.transaction.annotation.Transactional)")
        public void checkN1(JoinPoint jp) {
            int count = queryCount.get();
            if (count > 10) {
                log.warn("Potential N+1 detected in {}: {} queries",
                    jp.getSignature().getName(), count);
            }
            queryCount.remove();
        }
    }

    // 트랜잭션 크기 모니터링
    @EventListener
    public void monitorLargeTransactions(TransactionCommittedEvent event) {
        if (event.getDuration() > 5000) {  // 5초 이상
            log.warn("Large transaction: {}ms", event.getDuration());

            // 분할 권장
            suggestTransactionSplit(event);
        }

        if (event.getAffectedRows() > 1000) {
            log.warn("Transaction affected {} rows", event.getAffectedRows());

            // Batch 처리 권장
            suggestBatchProcessing(event);
        }
    }
}
```

**🔬 실습: 트러블슈팅 시뮬레이션 (6시간)**

**실습 1: 트랜잭션 예외 재현 (2시간)**

```java
@SpringBootTest
@Transactional
public class TransactionExceptionTest {

    @Test
    public void testUnexpectedRollback() {
        // Given
        Order order = new Order();

        // When
        assertThrows(UnexpectedRollbackException.class, () -> {
            orderService.createOrderWithNestedFailure(order);
        });

        // Then
        assertEquals(0, orderRepository.count());
    }

    @Test
    public void testLazyInitialization() {
        // Given
        Long orderId = createOrderWithItems();

        // When - 트랜잭션 밖에서 실행
        TestTransaction.end();

        // Then
        assertThrows(LazyInitializationException.class, () -> {
            Order order = orderRepository.findById(orderId);
            order.getItems().size();  // Lazy Loading 시도
        });
    }

    @Test
    @Transactional(propagation = Propagation.NOT_SUPPORTED)
    public void testOptimisticLockFailure() throws Exception {
        // Given
        Long productId = createProduct();

        // When - 동시에 같은 상품 수정
        ExecutorService executor = Executors.newFixedThreadPool(2);
        CountDownLatch latch = new CountDownLatch(2);
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicInteger failCount = new AtomicInteger(0);

        for (int i = 0; i < 2; i++) {
            executor.submit(() -> {
                try {
                    productService.updateStock(productId, 10);
                    successCount.incrementAndGet();
                } catch (OptimisticLockingFailureException e) {
                    failCount.incrementAndGet();
                } finally {
                    latch.countDown();
                }
            });
        }

        latch.await();

        // Then
        assertEquals(1, successCount.get());
        assertEquals(1, failCount.get());
    }
}
```

**실습 2: 성능 모니터링 구현 (2시간)**

```java
@Configuration
@EnableAspectJAutoProxy
public class PerformanceMonitoringConfig {

    @Bean
    public TransactionMonitoringAspect transactionMonitor() {
        return new TransactionMonitoringAspect();
    }
}

@Aspect
@Component
@Slf4j
public class TransactionMonitoringAspect {

    private final MeterRegistry meterRegistry;

    @Around("@annotation(transactional)")
    public Object monitorTransaction(ProceedingJoinPoint pjp, Transactional transactional)
            throws Throwable {

        String methodName = pjp.getSignature().toShortString();
        Timer.Sample sample = Timer.start(meterRegistry);

        // 트랜잭션 시작 전 Connection 수
        int connectionsBefore = getActiveConnections();

        try {
            Object result = pjp.proceed();

            // 성공 메트릭
            meterRegistry.counter("transaction.success",
                "method", methodName,
                "propagation", transactional.propagation().name()
            ).increment();

            return result;

        } catch (Exception e) {
            // 실패 메트릭
            meterRegistry.counter("transaction.failure",
                "method", methodName,
                "exception", e.getClass().getSimpleName()
            ).increment();

            throw e;

        } finally {
            // 실행 시간
            sample.stop(Timer.builder("transaction.duration")
                .tag("method", methodName)
                .publishPercentiles(0.5, 0.95, 0.99)
                .register(meterRegistry));

            // Connection 누수 체크
            int connectionsAfter = getActiveConnections();
            if (connectionsAfter > connectionsBefore) {
                log.warn("Potential connection leak in {}", methodName);
            }
        }
    }
}

// Micrometer 메트릭 엔드포인트
@RestController
@RequestMapping("/metrics")
public class MetricsController {

    @Autowired
    private MeterRegistry meterRegistry;

    @GetMapping("/transactions")
    public Map<String, Object> getTransactionMetrics() {
        return Map.of(
            "successCount", meterRegistry.counter("transaction.success").count(),
            "failureCount", meterRegistry.counter("transaction.failure").count(),
            "avgDuration", meterRegistry.timer("transaction.duration").mean(),
            "maxDuration", meterRegistry.timer("transaction.duration").max()
        );
    }
}
```

**실습 3: 트러블슈팅 도구 제작 (2시간)**

```java
@Component
@Slf4j
public class TransactionDebugger {

    // 현재 트랜잭션 상태 덤프
    public void dumpTransactionState() {
        log.info("=== Transaction State ===");
        log.info("Active: {}",
            TransactionSynchronizationManager.isActualTransactionActive());
        log.info("Name: {}",
            TransactionSynchronizationManager.getCurrentTransactionName());
        log.info("Read-only: {}",
            TransactionSynchronizationManager.isCurrentTransactionReadOnly());
        log.info("Isolation: {}",
            TransactionSynchronizationManager.getCurrentTransactionIsolationLevel());

        Map<Object, Object> resources =
            TransactionSynchronizationManager.getResourceMap();
        log.info("Resources: {}", resources.size());

        resources.forEach((key, value) -> {
            log.info("  {} -> {}", key, value);
        });
    }

    // Lock 정보 조회 (MySQL)
    @Autowired
    private JdbcTemplate jdbcTemplate;

    public void showLocks() {
        // InnoDB Lock 정보
        String sql = """
            SELECT
                trx_id,
                trx_state,
                trx_started,
                trx_wait_started,
                trx_mysql_thread_id,
                trx_query
            FROM information_schema.INNODB_TRX
            """;

        List<Map<String, Object>> transactions = jdbcTemplate.queryForList(sql);

        transactions.forEach(tx -> {
            log.info("Transaction: {}", tx.get("trx_id"));
            log.info("  State: {}", tx.get("trx_state"));
            log.info("  Query: {}", tx.get("trx_query"));
        });

        // Lock Wait 정보
        sql = """
            SELECT * FROM sys.innodb_lock_waits
            """;

        List<Map<String, Object>> lockWaits = jdbcTemplate.queryForList(sql);

        lockWaits.forEach(wait -> {
            log.warn("Lock Wait:");
            log.warn("  Waiting: {}", wait.get("waiting_query"));
            log.warn("  Blocking: {}", wait.get("blocking_query"));
        });
    }
}
```

**📝 Day 26-28 산출물**

- 면접 Q&A 20개: 트러블슈팅 케이스
- 트러블슈팅 가이드 문서 작성

---

#### **Day 29-30: 최종 정리와 프로젝트 (10시간)**

**📚 전체 복습 (4시간)**

- 8주간 학습 내용 총정리
- 핵심 개념 마인드맵 작성
- 면접 예상 질문 200개 검토

**🔬 최종 프로젝트: E-Commerce 트랜잭션 시스템 (6시간)**

```java
// 종합 프로젝트: 실전 E-Commerce 시스템
@SpringBootApplication
@EnableTransactionManagement
@EnableAsync
@EnableScheduling
public class ECommerceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ECommerceApplication.class, args);
    }
}

// 메인 서비스
@Service
@Slf4j
public class CompleteOrderService {

    // 모든 학습 내용 종합
    @Transactional
    public OrderResult processOrder(OrderRequest request) {

        // 1. Spring 트랜잭션
        Order order = createOrder(request);

        // 2. JPA 최적화
        optimizeJPAQueries(order);

        // 3. 분산 트랜잭션 (Saga)
        CompletableFuture<SagaResult> sagaResult =
            processSagaAsync(order);

        // 4. Kafka 트랜잭션
        publishEvents(order);

        // 5. Redis 캐시
        updateCache(order);

        // 6. 모니터링
        recordMetrics(order);

        return OrderResult.of(order, sagaResult);
    }
}
```

**📝 최종 산출물 정리**

- **면접 Q&A**: 총 200개
- **블로그 포스트**: 8편
  1. Spring DI 방식 선택 가이드
  2. Spring 트랜잭션 전파 레벨 완벽 가이드
  3. HikariCP 최적화 가이드
  4. Saga 패턴 실전 적용기
  5. 트랜잭션과 캐시 일관성 보장하기
  6. Kafka Exactly-Once 구현 가이드
  7. JPA N+1 문제 완벽 해결 가이드
  8. 트랜잭션 성능 최적화 전략
- **GitHub 프로젝트**: Spring-Transaction-Master
  - 각 주차별 예제 코드
  - 최종 E-Commerce 프로젝트
  - 트러블슈팅 케이스 모음

---

## **📈 학습 성과 측정**

### **기술 역량 체크리스트**

- [ ] Spring IoC/AOP 내부 동작 설명 가능
- [ ] 트랜잭션 전파 레벨 7가지 완벽 이해
- [ ] 격리 수준별 문제 상황 재현 가능
- [ ] 분산 트랜잭션 3가지 패턴 구현 가능
- [ ] DB별 트랜잭션 특성 파악
- [ ] JPA N+1 문제 4가지 해결법 적용
- [ ] Kafka Exactly-Once 구현 가능
- [ ] 트랜잭션 성능 최적화 수행 가능

### **실무 적용 능력**

- [ ] 트랜잭션 설계 의사결정 가능
- [ ] 트러블슈팅 체계적 접근
- [ ] 성능 병목 지점 파악과 해결
- [ ] 기술 선택 트레이드오프 설명

이 커리큘럼을 완주하시면 Spring과 트랜잭션 영역에서 스태프 엔지니어 수준의 전문성을 갖추게 되실 것입니다. 각 주차별로 꾸준히 실습하시고, 블로그 포스팅과 면접 Q&A 작성을 통해 지식을 체화하시기 바랍니다.
