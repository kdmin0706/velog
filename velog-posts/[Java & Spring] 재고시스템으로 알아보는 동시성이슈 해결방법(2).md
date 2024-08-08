<blockquote>
<p>인프런 &quot;재고시스템으로 알아보는 동시성이슈 해결방법&quot; 강의를 듣고 작성한 글입니다.</p>
</blockquote>
<p><a href="https://velog.io/@kdmin0706/Java-Spring-%EC%9E%AC%EA%B3%A0%EC%8B%9C%EC%8A%A4%ED%85%9C%EC%9C%BC%EB%A1%9C-%EC%95%8C%EC%95%84%EB%B3%B4%EB%8A%94-%EB%8F%99%EC%8B%9C%EC%84%B1%EC%9D%B4%EC%8A%88-%ED%95%B4%EA%B2%B0%EB%B0%A9%EB%B2%951">재고 감소가 제대로 처리되지 않은 이유!!</a>
멀티스레드 처리 시에 <strong>레이스 컨디션</strong>이 발생하여 재고 감소가 되지 않았습니다.</p>
<p><code>레이스 컨디션</code> 은 2개 이상의 스레드가 같은 데이터를 <strong>동시에 변경</strong>하기 때문에 발생하는 문제였습니다. 이를 해결하는 방법에 대해 알아보겠습니다.</p>
<h2 id="race-condition-해결방법">Race Condition 해결방법</h2>
<h3 id="1-synchronized-사용">1. synchronized 사용</h3>
<ul>
<li>메서드에 <code>synchronized</code> 를 명시해주면 하나의 스레드만 접근이 가능합니다.</li>
<li>즉, <code>멀티스레드 환경에서 스레드 간의 데이터 동기화</code>를 시켜주기 위해 자바에서 제공한 키워드입니다.</li>
</ul>
<pre><code class="language-java">@Transactional
//재고 감소 메서드
public synchronized void decrease(Long id, Long quantity){
    // Stock 조회
    Stock stock = stockRepository.findById(id).orElseThrow();

    // 재고를 감소
    stock.decrease(quantity);

    // 갱신된 값을 저장
    stockRepository.saveAndFlush(stock);
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/47e3294c-9a0d-40b6-8517-51bc96ebc29b/image.png" /></p>
<p>위의 이미지처럼 테스트가 실패한 것을 확인할 수 있습니다.</p>
<p>그 이유는 Spring의 <code>Transactional</code> 어노테이션의 동작방식 때문입니다.</p>
<p>Transactional 어노테이션은 우리가 만든 클래스를 래핑한 클래스로 새로 만들어 실행하게 됩니다. </p>
<blockquote>
<p>트랜잭션 시작 -&gt; 메서드 호출 -&gt; 메서드 실행 종료 -&gt; 트랜잭션 종료</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/a0de15fb-8136-495b-a578-b082fbf9d207/image.png" /></p>
<p>트랜잭션 종료 시점에 DB 업데이트를 진행하게 되는데, 실제 데이터베이스에 <code>업데이트 되기 전</code>에 다른 스레드가 실행되면서 <code>갱신되기 전의 값</code>을 가져가기 때문에 에러가 발생하게 됩니다.</p>
<blockquote>
<p>즉, <code>synchronized</code>는 <strong>하나의 프로세스</strong> 안에서만 보장이 됩니다.
서버가 1대일 경우에는 데이터 접근을 서버 하나가 해서 괜찮겠지만, 서버가 <code>여러 대</code>일 경우에는 데이터의 접근을 여러 대에서 하기에 <strong>거의 사용하지 않습니다.</strong></p>
</blockquote>
<h2 id="2-database-이용해보기">2. DataBase 이용해보기</h2>
<h3 id="1-pessimistic-lock">1. Pessimistic Lock</h3>
<ul>
<li>실제로 <code>데이터</code>에 <code>Lock</code>을 걸어서 정합성을 맞추는 방법입니다.</li>
<li>exclusive Lock(베타적 잠금)을 걸게되며 다른 트랜잭션에서는 lock 이 해제되기전에 데이터를 가져갈 수 없게됩니다.</li>
<li><code>데드락</code>이 걸릴 수 있기때문에 주의하여 사용하여야 합니다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/bfe23328-1c4c-4719-bfb0-a989f877530f/image.png" /></p>
<blockquote>
<p>자원 요청에 따른 <strong>동시성 문제가 발생할 것이라고 예상하고 Lock</strong>을 
걸어버리는 <code>비관적 락</code> 방식입니다.</p>
</blockquote>
<ul>
<li>Pessimistic Lock 장점<ol>
<li>충돌이 빈번하게 일어나는 경우 Optimistic Lock 보다 성능이 좋을 수 있다.</li>
<li>Lock을 통해 update를 제어하기에 데이터의 정합성이 보장된다.</li>
</ol>
</li>
<li>Pessimistic Lock 단점<ol>
<li>별도의 Lock을 잡기 때문에 성능이 감소할 수 있다.</li>
</ol>
</li>
</ul>
<p><strong>Pessimistic Lock 데이터 적용 과정</strong>
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/a87cdddf-4b0a-40bd-b9ed-f598a4756baa/image.png" /></p>
<p><strong>Pessimistic Lock 적용 예제</strong></p>
<ol>
<li>Repository Interface<ul>
<li>Spring Data JPA에서는 <code>@Lock</code> 어노테이션을 사용하여 구현할 수 있습니다.</li>
</ul>
</li>
</ol>
<pre><code class="language-java">public interface StockRepository extends JpaRepository&lt;Stock, Long&gt; {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query(&quot;select s from Stock s where s.id = :id&quot;)
    Stock findByIdWithPessimistic(Long id);
}</code></pre>
<ol start="2">
<li><p>Service 구현 로직</p>
<pre><code class="language-java">@Transactional
public void decrease(Long id, Long quantity){
 //Pessimistic Lock을 활용해서 데이터를 가져옴
 Stock stock = stockRepository.findByIdWithPessimisticLock(id);

 //재고를 감소시킴
 stock.decrease(quantity);

 //데이터를 저장
 stockRepository.save(stock);
}</code></pre>
</li>
<li><p>테스트 실행</p>
</li>
</ol>
<ul>
<li><p>테스트가 정상적으로 동작하는 것을 확인할 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/5c784a45-de2b-4329-9408-f91708747b23/image.png" /></p>
</li>
<li><p>쿼리를 확인해보면 <code>for update</code> 문구가 있는데, 이 부분이 Lock을 걸고 데이터를 가지고 오는 것이라고 볼 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/efd386c5-ee64-4601-b1f0-53e3c4205c73/image.png" /></p>
</li>
</ul>
<h3 id="2-optimistic-lock">2. Optimistic Lock</h3>
<ul>
<li>실제로 <code>Lock</code>을 사용하지 않고 <code>버전</code>을 이용함으로써 정합성을 맞추는 방법입니다.</li>
<li>먼저 데이터를 읽은 후에 update를 할 때 <strong>내가 읽은 버전이 맞는 확인</strong>하며 업데이트합니다.</li>
<li>내가 읽은 버전에서 수정사항이 생겼을 경우에는 application에서 다시 읽은후에 작업을 수행해야 합니다.</li>
</ul>
<blockquote>
<p>자원에 락을 걸어서 선점하지 않고, 동시성 문제가 발생하면 그 때 처리하는 <code>낙관적 락</code> 방식입니다.</p>
</blockquote>
<ul>
<li>시퀀스
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/0d1dec50-184b-4963-8b49-5d66aa9c4ce0/image.png" /><ol>
<li>server 1이 version1 임을 조건절에 명시하면서 업데이트 쿼리를 날립니다.</li>
<li>version1 쿼리가 업데이트 되면서 mysql의 version = 2 가 됩니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/3dd2cc74-a321-4919-ae21-4a12e3d105e9/image.png" /></li>
<li>server 2가 version1 로 업데이트 쿼리를 날리면 <code>버전이 맞지 않아</code> 실패합니다.</li>
<li>쿼리가 실패하면 server 2에서 다시 조회하여 <code>버전을 맞춘 후</code> 업데이트 쿼리를 날리는 과정을 거칩니다.</li>
</ol>
</li>
</ul>
<p><strong>Optimistic Lock 데이터 적용 과정</strong>
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/45a7687a-e969-420c-a325-cc892f60ca50/image.png" /></p>
<p><strong>Optimistic Lock 적용 예제</strong></p>
<ol>
<li><p>Stock.class에 <code>version</code> 컬럼을 추가합니다.</p>
<ul>
<li>주의점❗ : <code>javax.persistence</code> 패키지에 있는 <code>@Version 어노테이션</code>을 사용해야 합니다!<pre><code class="language-java">@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Stock {
</code></pre>
</li>
</ul>
<p>@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;</p>
<p>private Long productId;</p>
<p>private Long quantity;</p>
<p>@Version
private Long version;</p>
<p>//생략...
}</p>
<pre><code></code></pre></li>
<li><p>Repository Interface</p>
<ul>
<li>Spring Data JPA에서는 <code>@Lock</code> 어노테이션을 사용하여 구현할 수 있습니다.</li>
</ul>
</li>
</ol>
<pre><code class="language-java">public interface StockRepository extends JpaRepository&lt;Stock, Long&gt; {

  @Lock(LockModeType.OPTIMISTIC)
  @Query(&quot; select s from Stock s where s.id = :id &quot;)
  Stock findByIdWithOptimisticLock(Long id);

}</code></pre>
<ol start="3">
<li><p>facade 생성</p>
<ul>
<li><p>Optimistic Lock은 실패했을 경우 재시도를 해야하기 때문에 <code>facade(퍼사드)</code>를 생성해야합니다.</p>
<pre><code class="language-java">@Component
public class OptimisticLockStockFacade {

private final OptimisticLockStockService optimisticLockStockService;

public OptimisticLockStockFacade(OptimisticLockStockService optimisticLockStockService){
   this.optimisticLockStockService = optimisticLockStockService;
}

public void decrease(Long id, Long quantity) throws InterruptedException{
   while (true) {  //업데이트를 실패했을 때 재시도를 해야함
       try {
           optimisticLockStockService.decrease(id, quantity);

           //정상적으로 업데이트 되면 빠져나감
           break;
       } catch (Exception e) {
           //수량 감소 실패 시, 재시도
           Thread.sleep(50);
       }
   }
}
}</code></pre>
</li>
</ul>
</li>
<li><p>테스트 실행 결과
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/7e51fcc7-7cc9-40a3-b41b-75b49677d1bc/image.png" /></p>
</li>
</ol>
<h3 id="3-named-lock">3. Named Lock</h3>
<ul>
<li>이름을 가진 metadata Locking입니다.</li>
<li>이름을 가진 lock을 획득하고 해제할 때까지 다른 세션은 해당 lock을 획득할 수 없습니다.</li>
<li>주의점❗ : <code>트랜잭션이 종료될 때까지</code> lock은 자동으로 해제되지 않습니다.</li>
<li>별도의 명령어로 해제를 수행하거나 선점 시간이 끝나야 해제가 됩니다.</li>
</ul>
<blockquote>
<p>Named Lock은 Pessimistic Lock과 유사하지만 
Pessimistic Lock은 <strong>row나 table 단위로</strong> Lock을 걸지만
NamedLock은 <strong>metadata 단위로</strong> Lock을 건다는 차이점이 있다.</p>
</blockquote>
<ul>
<li>장점<ol>
<li>NamedLock은 주로 <code>분산락</code>을 구현할 때 사용한다.</li>
<li>Pessimistic Lock은 <code>timeOut</code>을 구현하기 힘들지만, NamedLock은 비교적 쉽게 구현 가능하다.</li>
<li>그 외의 <strong>데이터 정합성을 맞추는 경우</strong>에도 사용 가능하다.</li>
</ol>
</li>
</ul>
<p><strong>Named Lock 데이터 적용 과정</strong></p>
<ol>
<li>Named Lock은 DB에 락을 걸지 않고, 별도의 공간에 락을 겁니다.</li>
<li>session-1이 1이라는 이름으로 락을 건다면 session-1이 1을 해지한 후에 락을 얻을 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/83b55318-88db-47db-abcd-292aedb8b0b6/image.png" /></li>
</ol>
<p><strong>Optimistic Lock 적용 예제</strong></p>
<ol>
<li><p>repository 작성</p>
<ul>
<li><p>Mysql에서는 <code>get_lock()</code>을 통해 획득할 수 있고 <code>release_lock()</code> 명령어를 통해 해결할 수 있습니다.</p>
<pre><code class="language-java">public interface LockRepository extends JpaRepository&lt;Stock, Long&gt; {
@Query(value = &quot;select get_lock(:key, 3000)&quot;, nativeQuery = true)
void getLock(String key);

@Query(value = &quot;select release_lock(:key)&quot;, nativeQuery = true)
void releaseLock(String key);
}</code></pre>
</li>
</ul>
</li>
</ol>
<ul>
<li>편의성을 위해서 JPA의 <code>nativeQuery</code> 기능을 활용한 상태이다.<blockquote>
<p>현업에서는 커넥션 풀이 부족해지는 현상으로 다른 서비스에 영향을 끼칠 수 있기에</p>
</blockquote>
</li>
<li><em>데이터 소스를 분리해서 사용하는 것을 권장함*</em></li>
</ul>
<ol start="2">
<li><p>facade 생성</p>
<ul>
<li><p>실제 로직 전후로 Lock 획득 해제를 해주어야 하기 때문에 퍼사드를 생성합니다.</p>
<pre><code class="language-java">@Component
public class NamedLockStockFacade {

private final LockRepository lockRepository;
private final StockService stockService;

public NamedLockStockFacade(LockRepository lockRepository, StockService stockService){
   this.lockRepository = lockRepository;
   this.stockService = stockService;
}

@Transactional
public void decrease(Long id, Long quantity){
   try {
       //lock 획득
       lockRepository.getLock(id.toString());
       //재고 감소
       stockService.decrease(id, quantity);
   }finally {
       //모든 로직이 종료되었을 때, lock 해제
       lockRepository.releaseLock(id.toString());
   }
}
}</code></pre>
</li>
</ul>
</li>
<li><p>service 로직 수정</p>
<ul>
<li><p>부모의 트랜잭션과 <code>별도로 실행</code>되어야하기에 <code>propagation</code>을 변경한다.</p>
<pre><code class="language-java">@Transactional(propagation = Propagation.REQUIRES_NEW)
public void decrease(Long id, Long quantity){
// Stock 조회
Stock stock = stockRepository.findById(id).orElseThrow();

// 재고를 감소
stock.decrease(quantity);

// 갱신된 값을 저장
stockRepository.saveAndFlush(stock);
}</code></pre>
</li>
</ul>
</li>
<li><p>application.yml 커넥션 풀 수정</p>
<ul>
<li>예제에서는 <code>동일한 DataSource</code>를 사용하기에 커넥션 풀의 수를 늘려줘야합니다.<pre><code class="language-yaml">spring:
datasource:
hikari:
 maximum-pool-size: 40</code></pre>
</li>
</ul>
</li>
<li><p>테스트 실행 결과
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/96924326-c549-45a4-8022-0eeae068f509/image.png" /></p>
</li>
</ol>
<h2 id="3-redis-활용하기">3. Redis 활용하기</h2>
<p>동시성 문제를 해결할 수 있는 <code>redis</code>에는 <code>Lettuce</code>와 <code>Reddison</code> 이 존재합니다.</p>
<ol>
<li><p>Lettuce</p>
<ul>
<li><code>setnx</code> 명령어를 활용하여 분산락 구현<ul>
<li>setnx : <code>Set If Not Exist</code>의 줄임말로, 특정 key에 value 값이 존재하지 않을 경우에 값을 설정(set) 하는 명령어</li>
</ul>
</li>
<li>spin Lock 방식<ul>
<li>lock을 획득하려는 스레드가 lock을 사용할 수 있는지 반복적으로 확인하면서 lock을 시도하는 방식</li>
<li><code>retry</code> 로직을 작성해야한다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/d27cb727-a7fe-4a63-92ad-8e26801652ea/image.png" /><blockquote>
<p>Spring Data Redis를 이용하면 <strong>Lettuce</strong>가 기본 라이브러리이기 때문에 
별도 라이브러리를 사용하지 않아도 된다.</p>
</blockquote>
</li>
</ul>
</li>
</ul>
</li>
<li><p>Reddison</p>
<ul>
<li><code>pub-sub</code> 기반으로 Lock 구현 제공<ul>
<li>pub-sub : 채널을 하나 만들고, 락을 점유중인 스레드가 락을 해제했음을 대기중인 스레드에게 알려주면 대기중인 스레드가 락 점유를 시도하는 방식이다.</li>
<li>Lettuce와 다르게 별도의 <code>retry</code> 방식을 <strong>작성하지 않아도 된다.</strong>
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/399eea32-acdb-4466-9643-848b94e11043/image.png" /></li>
</ul>
</li>
</ul>
</li>
</ol>
<h3 id="redis-환경-설정">Redis 환경 설정</h3>
<p><strong>1. redis 실행을 합니다.</strong></p>
<p><code>도커</code>를 사용하는 경우 아래와 같이 실행해줍니다.</p>
<ul>
<li>redis 이미지 다운로드 : docker pull redis</li>
<li>redis 실행 : docker run --name redis -d -p 6379:6379 redis</li>
<li>실행 확인 명령어 : docker ps</li>
</ul>
<p><code>OS</code>에 직접 설치하여 사용하는 경우는 아래와 같이 <code>application.yml</code>에 속성을 추가합니다.</p>
<pre><code class="language-yml">spring:
  data:
    redis:
      host: localhost
      port: 6379</code></pre>
<p><strong>2. build.gradle 의존성을 추가합니다.</strong></p>
<pre><code>dependencies {
    // redis
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}</code></pre><h3 id="1-lettuce-사용해보기">1. Lettuce 사용해보기</h3>
<ol>
<li><code>setnx</code> 명령어 사용해보기</li>
</ol>
<p><code>redis-cli</code>를 활용해서 setnx 명령어를 사용해봅니다.</p>
<p> setnx 명령어는 key와 value 형태로 삽입하는데
 <code>1</code>이라는 key로 맨 처음 삽입할 때는 성공하지만, 재차 입력하면 실패하게 됩니다.</p>
<p> <code>del</code> 명령어를 통해 해당 key의 데이터를 삭제하고 다시 입력하면 성공하는 것을 확인할 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/fe32b66c-c6c6-4a12-86a4-8afa5d83501f/image.png" /></p>
<p><strong>Lettuce를 활용해 재고 로직 작성</strong></p>
<ul>
<li>MySQL의 <code>Named Lock</code>과 비슷하다.</li>
<li>세션 관리에 신경을 쓰지 않아도 된다.</li>
</ul>
<ol>
<li><p>RedisLockRepository</p>
<ul>
<li>key를 이용한 lock과 unlock 정의<pre><code class="language-java">@Component
@RequiredArgsConstructor
public class RedisLockRepository {
</code></pre>
</li>
</ul>
<p>private final RedisTemplate&lt;String, String&gt; redisTemplate;</p>
<p>public Boolean lock(Long key) {
 return redisTemplate.opsForValue().setIfAbsent(generateKey(key), &quot;lock&quot;, Duration.ofMillis(3000));
}</p>
<p>public Boolean unlock(Long key) {
 return redisTemplate.delete(generateKey(key));
}</p>
<p>private String generateKey(Long key) {
 return key.toString();
}
}</p>
<pre><code></code></pre></li>
<li><p>LettuceLockStockFacade 작성</p>
<ol>
<li>SpinLock 방식으로 락을 시도</li>
<li>락을 얻은 후에 로직 실행</li>
<li>이후, 락을 해제<pre><code class="language-java">@Component
@RequiredArgsConstructor
public class LettuceLockStockFacade {
</code></pre>
</li>
</ol>
<p>private final RedisLockRepository redisLockRepository;
private final StockService stockService;</p>
<p>public void decrease(Long id, Long quantity) throws InterruptedException {
 while (!redisLockRepository.lock(id)) {
   Thread.sleep(100);
 }</p>
<p> try {
   stockService.decrease(id, quantity);
 } finally {
   redisLockRepository.unlock(id);
 }</p>
<p>}</p>
</li>
</ol>
<p>}</p>
<pre><code>

3. 테스트 케이스 실행
![](https://velog.velcdn.com/images/kdmin0706/post/09e7b4e2-0a0e-457d-975d-4c828569dc9d/image.png)

&gt; 구현이 간단하는 장점이 있지만, spin Lock 방식이므로 **redis에 부하를 줄 수 있다.**
그렇기 때문에 **Thread.sleep** 을 통한 Lock 획득 재시도 간의 텀을 두어야한다.


### 2. Redisson 사용해보기
- pub-sub 방식을 사용해보기 위해서 2개의 redis cli를 실행시켜보겠습니다.

1. 1번 redis-cli에 `subscribe` 명령어를 사용하여 구독을 진행합니다.
![](https://velog.velcdn.com/images/kdmin0706/post/90de669d-a7f2-4a53-9b8e-d7ad040acfed/image.png)

2. 2번 redis-cli에 `publish` 명령어를 이용하여 메세지를 발행합니다.
![](https://velog.velcdn.com/images/kdmin0706/post/fff59139-f963-40d1-8606-3070d80a40d2/image.png)
- 1번 cli에서는 현재 ch1을 구독하고 있기 때문에 2번 cli에서 보낸 **hello** 메시지를 받아볼 수 있다는 것을 확인할 수 있습니다.

&gt; Redisson은 Lettuce와 달리 **락 획득을 계속 시도하지 않기 때문에**
**redis의 부하를 줄일 수 있습니다.**



**redisson을 사용하기 위해서 의존성을 추가합니다. **</code></pre><p>dependencies {
    // redis(redisson)
    implementation 'org.redisson:redisson-spring-boot-starter:3.23.2'
}</p>
<pre><code>
**redisson을 활용해 재고 로직 작성**
&gt; redisson의 경우에는 **Lock과 관련된 클래스를 제공**해줍니다.


1. ReddisonLockStockFacade 작성
```java
@Component
@RequiredArgsConstructor
public class RedissonLockStockFacade {

  private final RedissonClient redissonClient;
  private final StockService stockService;

  public void decrease(Long id, Long quantity) {
    RLock lock = redissonClient.getLock(id.toString());

    try {
      boolean available = lock.tryLock(10, 1, TimeUnit.SECONDS);

      if (!available) {
        System.out.println(&quot;Lock 획득 실패&quot;);
        return;
      }

      stockService.decrease(id, quantity);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    } finally {
      lock.unlock();
    }
  }
}</code></pre><ol start="2">
<li>테스트 케이스 실행
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/bdbe1bae-4672-4688-8d28-7aea7f179ab3/image.png" /></li>
</ol>
<blockquote>
<p>redis의 부하를 줄여주는 장점은 있지만 구현이 복잡하고 
<strong>별도의 라이브러리</strong>를 이용해야한다는 단점이 존재합니다.</p>
</blockquote>
<p>👉 즉, 재시도가 필요한 경우 <code>Redisson</code>을 활용하고 
재시도가 필요하지 않은 경우 <code>Lettuce</code>를 활용해보는 것으로 진행해볼 수 있습니다.</p>
<h2 id="mysql-vs-redis">MySQL vs Redis</h2>
<ol>
<li>MySQL 특징<ul>
<li>이미 MySQL을 사용하고 있다면 별도의 비용 없이 사용이 가능하다.</li>
<li>성능이 redis 보다 좋지않지만, 어느 정도의 트래픽까지는 문제없이 활용 가능하다.</li>
</ul>
</li>
<li>Redis 특징<ul>
<li>활용 중인 redis가 없다면 별도의 구축비용과 인프라 관리비용이 발생한다.</li>
<li>MySQL보다 성능이 좋기에, 더 많은 요청을 처리 가능하다.</li>
</ul>
</li>
</ol>