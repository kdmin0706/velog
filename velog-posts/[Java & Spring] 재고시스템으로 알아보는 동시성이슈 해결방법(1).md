<blockquote>
<p>인프런 &quot;재고시스템으로 알아보는 동시성이슈 해결방법&quot; 강의를 듣고 작성한 글입니다.</p>
</blockquote>
<h2 id="동시성-문제란">동시성 문제란!?</h2>
<p>동일한 데이터에 <code>2개</code> 이상의 스레드 혹은, 세션에서 가변 데이터를 동시에 제어할 때 나타나는 문제입니다.</p>
<p>하나의 세션이 데이터를 수정 중이거나, 다른 세션에서 수정 전의 데이터를 조회해 로직을 처리함으로써 데이터의 정합성이 깨지는 문제를 말합니다.</p>
<h2 id="간단한-재고-시스템-로직-작성">간단한 재고 시스템 로직 작성</h2>
<ul>
<li>재고 감소 로직의 코드는 <a href="https://github.com/kdmin0706/concurrentStock">깃허브</a> 참고해주세요!</li>
</ul>
<h3 id="stockclass">stock.class</h3>
<pre><code class="language-java">@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Stock {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private Long productId;

  private Long quantity;

  @Builder
  public Stock(Long productId, Long quantity) {
    this.productId = productId;
    this.quantity = quantity;
  }

  public void decrease(Long quantity) {
    if (this.quantity - quantity &lt; 0) {
      throw new RuntimeException(&quot;재고는 0개 미만이 될 수 없습니다.&quot;);
    }

    this.quantity -= quantity;
  }

}</code></pre>
<h3 id="stockserviceclass">stockService.class</h3>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class StockService {

  private final StockRepository stockRepository;

  @Transactional
  public void decrease(Long id, Long quantity) {
    Stock stock = stockRepository.findById(id).orElseThrow();
    stock.decrease(quantity);

    stockRepository.save(stock);
  }

}</code></pre>
<h2 id="1-동시성-문제">1. 동시성 문제</h2>
<h3 id="테스트-케이스-작성">테스트 케이스 작성</h3>
<p>위의 코드는 간단한 재고 감소 로직이기 때문에 이를 토대로 간단하게 테스트 케이스를 작성해보겠습니다.</p>
<pre><code class="language-java">@Test
@DisplayName(&quot;상품의 재고를 감소시킵니다.&quot;)
void decreaseStock() {
  // given // when
  stockService.decrease(1L, 1L);

  // then
  Stock stock = stockRepository.findById(1L).orElseThrow();
  assertThat(99L).isEqualTo(stock.getQuantity());
}</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/d9e535e9-e918-416e-8b91-242e000f11b5/image.png" /></p>
<p>테스트 케이스 요청이 1개만 들어온 경우에는 테스트가 성공한 것을 확인할 수 있습니다.</p>
<p>만약에 <code>1개</code>가 아닌 <code>100개</code>의 요청이 들어온다면 어떻게 되는지 확인해보겠습니다.</p>
<h3 id="테스트-케이스-작성동시에-100개의-요청">테스트 케이스 작성(동시에 100개의 요청)</h3>
<pre><code class="language-java">@Test
@DisplayName(&quot;동시에 100개의 요청이 들어온다.&quot;)
void sameTime_100_request() throws InterruptedException {
  // given
  int threadCount = 100;
  ExecutorService executorService = Executors.newFixedThreadPool(32);
  CountDownLatch latch = new CountDownLatch(threadCount);

  // when
  for (int i = 0; i &lt; threadCount; i++) {
    executorService.submit(() -&gt; {
      try {
        stockService.decrease(1L, 1L);
      } finally {
        latch.countDown();
      }
    });
  }

  latch.await();

  // then
  Stock stock = stockRepository.findById(1L).orElseThrow();
  assertThat(stock.getQuantity()).isZero();
}</code></pre>
<p>여기서 중점적으로 볼 내용은 동시에 여러 개의 요청을 보내야하기 때문에 <code>멀티스레드</code>를 사용했습니다. </p>
<p>멀티스레드 형식을 사용하기 위해서 사용한 메서드에 대해 알아보겠습니다.</p>
<p><strong>1. ExecutorService</strong></p>
<ul>
<li>병렬 작업 시 여러 개의 작업을 효율적으로 처리하기 위해 제공되는 Java 라이브러리</li>
<li>손쉽게 ThreadPool을 구성하고 Task를 실행하고 관리할 수 있는 역할을 한다.</li>
<li>Executors를 사용하여 ExecutorService 객체를 생성하며, 쓰레드 풀의 개수 및 종류를 지정할 수 있는 메서드를 제공한다.</li>
</ul>
<p><strong>2. CountDownLatch</strong></p>
<ul>
<li><p>어떤 스레드가 다른 쓰레드에서 작업이 완료될 때까지 기다릴 수 있도록 해주는 클래스</p>
</li>
<li><p>멀티스레드가 100번의 작업이 모두 완료되면, 테스트를 하도록 기다리게 해준다.</p>
</li>
<li><p>CountDownLatch 작동 원리</p>
<ol>
<li>new CountDownLatch(count); -&gt; Latch할 갯수를 지정한다.</li>
<li>countDown() 을 호출하면 Latch의 숫자가 <code>1개씩 감소</code>한다.</li>
<li>await()는 Latch 숫자가 <code>0이 될 때</code>까지 기다립니다. </li>
</ol>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/d3c4d176-a34f-434c-8ca1-fe854475b453/image.png" /></p>
</li>
</ul>
<p>CountDownLatch를 활용하여, 멀티스레드 작업이 100번의 재고감소 로직을 호출한 뒤에 재고가 0이 되는지 확인했지만, 실제로는 다른 값이 출력되는 것을 확인할 수 있습니다.</p>
<p>그 이유는 <strong>레이스 컨디션(Race Condition)</strong> 이 발생했기 때문입니다.</p>
<p><strong>레이스 컨디션</strong>이란!?🔥</p>
<ul>
<li>2개 이상의 스레드가 공유 데이터에 엑세스 할 수 있고 동시에 변경하려고 발생하는 문제를 말합니다.</li>
</ul>
<h2 id="2-멀티스레드로-처리할-때-레이스-컨디션이-일어나는-이유">2. 멀티스레드로 처리할 때 레이스 컨디션이 일어나는 이유</h2>
<h3 id="예상-작업-순서">예상 작업 순서</h3>
<p>멀티스레드로 작업할 때, 아래와 같이 순차적으로 데이터에 접근해 처리하고 처리된 데이터를 다음 스레드에 접근하여 처리하는 것으로 볼 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/7a587b5e-fb82-44e2-8f0c-782f7b2fac18/image.png" /></p>
<h3 id="실제-작업-순서">실제 작업 순서</h3>
<p>하지만 실제 작업은 <code>같은 데이터를 동시에 변경</code> 하기 때문에 작업 중 하나가 누락되게 될 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/22132980-a29a-466a-b2c5-c7044b6fbcb6/image.png" /></p>
<blockquote>
<p>Race Condition을 해결하기 위해서
<strong>하나의 스레드가 작업을 완료한 후에, 다른 스레드가 공유된 자원에 접근</strong>하도록 해야합니다.</p>
</blockquote>
<p><strong>다음 포스팅에서 이를 해결하는 방법에 대해 알아보겠습니다</strong></p>