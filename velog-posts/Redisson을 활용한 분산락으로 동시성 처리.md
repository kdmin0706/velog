<h2 id="시작하며">시작하며</h2>
<p>redis를 통한 <code>분산락</code>을 이용해서 실제 개인 프로젝트에 적용해서 
동시성을 해결하는 방법에 대해 알아보겠습니다. </p>
<h2 id="분산-락distribute-lock-이란">분산 락(Distribute Lock) 이란?</h2>
<ul>
<li>분산 락은 여러 서버가 하나의 자원에 동시에 접근하려는 것을 막고 <code>한 번에 하나의 서버만 작업</code>할 수 있도록 해주는 동기화 매커니즘입니다.</li>
<li>이를 통해 데이터의 동시 변경을 막고 시스템 전체의 데이터 일관성을 보장합니다.</li>
</ul>
<h3 id="redisson을-통한-구현">Redisson을 통한 구현</h3>
<p>분산락을 구현하기 위해서 <code>redisson</code> 라이브러리를 사용하는데 사용한 이유에 대해 알아보겠습니다.</p>
<p>Spring 에서 redis를 사용하는 라이브러리는 대표적으로 <code>Lettuce</code>와 <code>redisson</code>이 있습니다.</p>
<p>해당 라이브러리의 장점과 단점에 대해 간단하게 알아보겠습니다.</p>
<p><strong>1. Lettuce</strong></p>
<blockquote>
<ol>
<li>Lock의 <code>timeout</code>이 <strong>지정되어 있지 않다.</strong><ul>
<li>락 획득 후 모종의 이유로 어플리케이션이 종료되거나 꺼진 경우 락이 <code>해제되지 않는다</code></li>
<li>그렇게 되면 다른 프로세스는 락의 해제만 기다리는 <code>무한 대기 상태</code>가 된다.<ol start="2">
<li><code>spin lock</code>으로 인해 redis에 부하가 많이 쌓인다.</li>
</ol>
</li>
<li>락을 획득하지 못한 경우 redis에 락을 획득하기 위해 무한정 요청을 보내게 된다.</li>
</ul>
</li>
</ol>
</blockquote>
<p><strong>2. redisson</strong></p>
<blockquote>
<ol>
<li>Lock의 <code>timeout</code>을 <strong>지정할 수 있다.</strong><ul>
<li>lock을 획득할 시에 타임아웃을 명시하게 되어있다.(즉, 무한 대기 X)<ol start="2">
<li><code>pub/sub</code> 형식을 사용하기에 spin lock을 사용하지 않는다.   </li>
</ol>
</li>
<li>락이 해제되면 락을 subscribe 하는 클라이언트에게 락이 해제되었다는 신호를 보낸다.</li>
<li>이후, 락을 subscribe 하는 클라이언트들은 더 이상 락에 대한 신호를 보내지 않게 된다.</li>
</ul>
</li>
</ol>
</blockquote>
<h2 id="redisson-적용">redisson 적용</h2>
<ul>
<li>아래 사용한 코드는 <a href="https://github.com/kdmin0706/concurrentStock">깃허브</a>를 참고해주세요</li>
</ul>
<h3 id="1-redisson-의존성-주입">1. redisson 의존성 주입</h3>
<pre><code>dependencies {
    // redisson
    implementation group: 'org.redisson', name: 'redisson', version: '3.27.2'
}</code></pre><h3 id="2-redisson-yml-설정">2. redisson yml 설정</h3>
<pre><code class="language-yml">spring:
  data:
    redis:
      host: localhost
      port: 6379</code></pre>
<h3 id="3-redissonconfigjava">3. RedissonConfig.java</h3>
<pre><code class="language-java">@Configuration
public class RedisConfig {

  @Value(&quot;${spring.data.redis.host}&quot;)
  private String host;

  @Value(&quot;${spring.data.redis.port}&quot;)
  private int port;

  @Bean
  public RedissonClient redissonClient() {
    Config config = new Config();
    config.useSingleServer()
        .setAddress(&quot;redis://&quot; + host + &quot;:&quot; + port);
    return Redisson.create(config);
  }

}</code></pre>
<p><code>RedissonClient</code>를 사용하기 위해 <strong>Config</strong> 설정을 빈으로 등록합니다.</p>
<h3 id="4-redisson-라이브러리">4. redisson 라이브러리</h3>
<p>redisson 에서는 Lock을 사용하기 위해 <code>RLock</code>이라는 인터페이스를 제공합니다.</p>
<p>락을 획득하기 위해서는 <code>tryLock</code>이라는 메서드를 사용하게 됩니다.</p>
<pre><code class="language-java">boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException;</code></pre>
<ul>
<li>waitTime : 락을 획득하기 위한 대기 시간</li>
<li>leaseTime : 락을 임대하는 시간</li>
<li>unit : 시간 단위</li>
</ul>
<h3 id="5-프로덕션-코드">5. 프로덕션 코드</h3>
<pre><code class="language-java">RLock rLock = redissonClient.getLock(key); // (1)

try {
        boolean available = rLock.tryLock(5, 3, TimeUnit.SECONDS); // (2)
        if (!available) {    // (3)
            return false;
        }

        // 락 획득 후 수행 로직...
} catch (InterruptedException e) {
        e.printStackTrace();
        throw new InterruptedException();
} finally {
        rLock.unlock(); // (4)
}</code></pre>
<p>위의 코드는 실제 redisson을 활용하여 개인프로젝트 있는 재고 차감 서비스 코드의 예시입니다.</p>
<p>redisson Lock이 설정된 시퀀스를 확인해보면</p>
<ol>
<li>key 이름에 해당하는 RLock 인스턴스를 가져옵니다.</li>
<li><code>Lock</code> 획득을 시도합니다. (true : 성공 / false : 실패)</li>
<li>획득 실패 시 <code>Lock</code>의 해제를 기다리며 예외 처리를 진행합니다.</li>
<li>finally에서 <code>Lock</code>을 해제합니다.</li>
</ol>
<h2 id="redisson-분산락-어노테이션-구현">redisson 분산락 어노테이션 구현</h2>
<p>위의 코드처럼 redisson Lock 로직 사이에 실행하고자 하는 로직을 넣으면 구현할 수 있습니다.</p>
<p>하지만 이렇게 되면 <strong>Lock의 로직과 비즈니스 로직은 분리가 되지 않게 됩니다.</strong></p>
<p>이를 해결하기 위해 <code>Annotation 기반</code>으로 분산락을 사용하는 방법에 대해 알아보겠습니다.</p>
<h3 id="1-annotation-기반-구현-시-장점">1. Annotation 기반 구현 시 장점</h3>
<ol>
<li>개발 효율성 향상<ul>
<li>annotation을 이용하여 분산락 처리를 보다 쉽게 사용할 수 있다.</li>
</ul>
</li>
<li>비즈니스 로직과 분산락 처리 로직의 분리<ul>
<li>각자의 역할만 담당하여 코드의 가독성을 증대한다.</li>
</ul>
</li>
<li>코드의 재사용성 향상</li>
</ol>
<h3 id="2-distributedlockjava">2. DistributedLock.java</h3>
<pre><code class="language-java">@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DistributedLock {

  /**
   * lock 이름
   */
  String key();

  /**
   * 락의 시간 단위
   */
  TimeUnit timeUnit() default TimeUnit.SECONDS;

  /**
   *  락을 기다리는 시간 (default - 5s)
   *  락 획득을 위해 waitTime 만큼 대기한다.
   */
  long waitTime() default 5L;

  /**
   * 락 임대 시간 (default - 3s)
   * 락을 획득한 이후 leaseTime이 지나면 락을 해제한다.
   */
  long leaseTime() default 3L;
}</code></pre>
<p><code>DistributedLock</code> 어노테이션의 파라미터는 key는 필수값으로 받게 설정합니다.
나머지 값들은 사용자가 직접 설정할 수 있도록 작성했습니다.</p>
<h3 id="3-distributedlockaopjava">3. DistributedLockAop.java</h3>
<pre><code class="language-java">@Aspect
@Component
@Slf4j
@RequiredArgsConstructor
public class DistributeLockAop {
  public static final String REDISSON_LOCK_PREFIX = &quot;Lock:&quot;;

  private final RedissonClient redissonClient;
  private final AopForTransaction aopForTransaction;

  @Around(&quot;@annotation(sample.concurrent.api.aop.DistributedLock)&quot;)
  public Object lock(final ProceedingJoinPoint joinPoint) throws Throwable {
    MethodSignature signature = (MethodSignature) joinPoint.getSignature();
    Method method = signature.getMethod();
    DistributedLock distributedLock = method.getAnnotation(DistributedLock.class);  // (1)

    String key = REDISSON_LOCK_PREFIX +
                CustomSpringELParser.getDynamicValue(signature.getParameterNames(),
                joinPoint.getArgs(), distributedLock.key());     // (2)
    RLock lock = redissonClient.getLock(key);  // (3)

    try {
      boolean available = lock.tryLock(distributedLock.waitTime(), distributedLock.leaseTime(),
          distributedLock.timeUnit());    // (4)
      if (!available) {
        return false;
      }

      return aopForTransaction.proceed(joinPoint);    // (5)
    } catch (InterruptedException e) {
      throw new InterruptedException();
    } finally {
      try {
        lock.unlock();  // (6)
      } catch (IllegalMonitorStateException e) {
        log.info(&quot;Redisson Lock Already Unlocked&quot;);
      }
    }
  }
}</code></pre>
<p><code>@DistributeLock</code>을 선언한 메소드를 호출했을때 실행되는 aop 클래스입니다.</p>
<p><code>@DistributeLock</code> 어노테이션의 파라미터 값을 가져와 분산락을 획득 시도를 합니다.
그리고 어노테이션이 선언된 메서드를 실행합니다.</p>
<p>(1) <code>@DistributeLock</code> 어노테이션을 가져옵니다.
(2) <code>@DistributeLock</code> 에 전달한 key를 가져오기 위해 <code>SpringEL</code> 표현식을 파싱한다.
(3) Redisson에 해당 락의 RLock 인터페이스를 가져온다.
(4) tryLock Method를 이용하여 Lock 획득을 시도한다. (획득 실패시 Lock이 해제될 때까지 subscribe)
(5) <code>@DistributeLock</code> 이 선언된 메서드의 로직을 수행한다(별도 트랜잭션으로 분리)
(6) 종료 혹은 예외 발생 시 finally에서 Lock을 해제한다.</p>
<p>여기서 주의해서 볼 부분은 <code>CustomSpringELParser</code> 와 <code>AopForTransaction</code> 클래스입니다.</p>
<h3 id="4-customspringelparserjava">4. CustomSpringELParser.java</h3>
<pre><code class="language-java">public class CustomSpringELParser {

  private CustomSpringELParser() {
  }

  public static Object getDynamicValue(String[] parameterName, Object[] args, String key) {
    ExpressionParser parser = new SpelExpressionParser();
    StandardEvaluationContext context = new StandardEvaluationContext();

    for (int i = 0; i &lt; parameterName.length; i++) {
      context.setVariable(parameterName[i], args[i]);
    }

    return parser.parseExpression(key).getValue(context, Object.class);
  }
}</code></pre>
<p><code>CustomSpringELParser</code> 는 전달받은 Lock의 이름을 <code>Spring Expression Language</code>로 파싱하여 읽어오게됩니다.</p>
<p>즉, <code>Spring Expression Language</code>를 사용하면 Lock의 이름을 자유롭게 사용할 수 있게 됩니다.</p>
<pre><code class="language-java">// (1)
@DistributedLock(key = &quot;#lockName&quot;)
public void shipment(String lockName) {
    ...
}

// (2)
@DistributedLock(key = &quot;#model.getName().concat('-').concat(#model.getShipmentOrderNumber())&quot;)
public void shipment(ShipmentModel model) {
    ...
}</code></pre>
<h3 id="5-aopfortransactionjava">5. AopForTransaction.java</h3>
<pre><code class="language-java">@Component
public class AopForTransaction {

  @Transactional(propagation = Propagation.REQUIRES_NEW)
  public Object proceed(final ProceedingJoinPoint joinPoint) throws Throwable {
    return joinPoint.proceed();
  }

}</code></pre>
<p><code>DistributedLock</code>이 선언된 메서드는 <code>Propagation.REQUIRES_NEW</code> 옵션을 지정하여 부모의 트랙잭션 유무에 관계없이 별도의 트랜잭셔으로 동작되게끔 설정하였습니다.
그리고 <strong>반드시 트랜잭션 커밋 이후 락이 해제되게끔 처리</strong>하였습니다.</p>
<blockquote>
<p>Q : 왜? 트랜잭션 커밋 이후 락이 해제되어야 하나요?
  A : 동시성 환경에서 데이터 정합성을 보장하기 위해서</p>
</blockquote>
<p>만약, 락의 해제 시점이 비즈니스 로직의 트랜잭션 커밋 이전에 발생한다면?</p>
<p>비즈니스 로직 트랜잭션이 커밋되기 전에 다른 스레드에서 락을 얻어 
비즈니스 로직을 수행할 수 있기 때문입니다.</p>
<p><code>재고 차감</code>을 예시로 들어보겠습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/55e448dc-c3b7-4142-935b-999d13220d58/image.png" /></p>
<ol>
<li>Client1, Client2 두 사용자가 재고 차감을 위해 메서드에 동시에 접근</li>
<li>Client1이 락을 선점하고 재고 조회하여 재고가 10개인 것을 확인한다.</li>
<li>Client1은 재고 1개를 차감하고 락을 해제한다(10-1 = 9개), 이 때 <code>트랜잭션은 커밋 되지 않은 상태</code>이다.</li>
<li>Client2는 락이 해제되었다는 신호를 받고 락을 획득하고 재고를 조회한다.</li>
<li>Client1에서 재고를 차감했지만 아직 트랜잭션 커밋이 되지 않은 상태이기에, Client2는 재고 조회 시 <code>10</code>으로 조회한다.</li>
<li>Client2는 동일하게 재고를 하나 차감하고 락을 해제하고 커밋한다.(DB에는 10-1=9개로 반영)</li>
</ol>
<blockquote>
<p>두 사용자가 동시에 접근하여 재고를 차감했지만 실제 DB에 차감된 재고는 <code>1개</code>가 됩니다.
즉, <strong>락의 해제가 트랜잭션 커밋보다 먼저 이뤄지면 데이터의 정합성이 깨질 수 있습니다</strong></p>
</blockquote>
<p><strong>트랜잭션 커밋 이후 락을 해제한다면❗</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/10cdb7c3-9100-4f8c-8288-571d384e9af1/image.png" /></p>
<ol>
<li>Client1,Client2 두 사용자가 재고를 차감하기 위해 동시 접근한다.</li>
<li>Client1이 락을 선점하고 재고를 조회하여 현재 재고가 10인 것을 확인한다.</li>
<li>Client1은 재고 하나를 차감하고 트랜잭션을 커밋한다(재고 = 9)</li>
<li>Client1은 락을 해제하고 Client2는 락이 해제되었다는 신호를 받고 락을 획득한다.</li>
<li>Client2는 락을 획득 후 재고 조회한다.</li>
<li>Client2는 재고를 하나 차감하고(재고 =&gt; 9-1=8) 트랜잭션 커밋 후 락을 해제한다.</li>
</ol>
<blockquote>
<p>두 사용자가 동시에 접근해도 재고가 모두 정상적으로 차감됩니다.<br />  락의 해제가 트랜잭션 커밋보다 뒤에 이뤄진 덕분에 동시성 환경에서도 
  데이터 정합성을 보장할 후 있습니다.</p>
</blockquote>
<h3 id="distributelockstockservicejava">DistributeLockStockService.java</h3>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class DistributeLockStockService {

  private final StockRepository stockRepository;

  @Transactional    //기존 코드
  public void decrease(Long id, Long quantity) {
    Stock stock = stockRepository.findById(id).orElseThrow();

    stock.decrease(quantity);
  }

  @DistributedLock(key = &quot;#lockName&quot;)
  public void decrease(String lockName, Long id, Long quantity) {
    Stock stock = stockRepository.findById(id).orElseThrow();

    stock.decrease(quantity);
  }
}</code></pre>
<h3 id="distributelockstockservicetestjava">DistributeLockStockServiceTest.java</h3>
<pre><code class="language-java">  private Stock stock;

  @BeforeEach
  void setUp() {
    stock = new Stock(1L, 100L);
    stockRepository.save(stock);
  }

  @Test
  @DisplayName(&quot;동시에 100개의 요청이 들어온다.(분산락)&quot;)
  void sameTime_100_request_toDistributeLock() throws InterruptedException {
    // given

    int numberOfThreads = 100;
    ExecutorService executorService = Executors.newFixedThreadPool(numberOfThreads);
    CountDownLatch latch = new CountDownLatch(numberOfThreads);

    // when
    for (int i = 0; i &lt; numberOfThreads; i++) {
      executorService.submit(() -&gt; {
        try {
          // 분산락 적용 메서드 호출
          stockService.decrease(&quot;Lock&quot;, stock.getId(), 1L);
        } finally {
          latch.countDown();
        }
      });
    }

    latch.await();
    executorService.shutdown();

    // then
    Stock persistStock = stockRepository.findById(stock.getId()).orElseThrow();
    assertThat(persistStock.getQuantity()).isEqualTo(0);

    System.out.println(&quot;잔여 재고 개수 : &quot; + persistStock.getQuantity());
  }</code></pre>
<p>테스트 코드를 구현한 예시입니다.</p>
<p>현재 프로젝트에서는 상품과 재고를 생성하여 100개의 주문이 들어왔을 시에 
주문이 제대로 100가 생성되었는지 체크할 수 있는 코드입니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/5e5dd842-7c04-4f13-aa31-213f3bbec161/image.png" /></p>
<p><code>threadCount</code> 의 값처럼 100번의 동작이 구현되어 테스트 성공되는 것을 확인할 수 있습니다.</p>
<h2 id="reference">Reference</h2>
<p><a href="https://helloworld.kurly.com/blog/distributed-redisson-lock">마켓 컬리 기술블로그</a></p>
<p><a href="https://github.com/redisson/redisson">https://github.com/redisson/redisson</a></p>