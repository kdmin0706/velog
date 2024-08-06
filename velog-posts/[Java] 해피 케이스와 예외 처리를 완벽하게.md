<h2 id="시작하며">시작하며</h2>
<p>프로그래밍을 하게 된다면 우리는 <code>해피 케이스</code>에 몰두하게 됩니다. 해피케이스는 모든 것이 계획대로 잘 동작되는 경우를 말합니다. 그러나 예외 상황은 발생하기 마련입니다. Java에서 예외 처리를 잘 하는 방법과 Null을 효과적으로 다루는 방법에 대해 알아보겠습니다.</p>
<h2 id="1-해피케이스와-예외-처리">1. 해피케이스와 예외 처리</h2>
<p>사람은 본능적으로 해피 케이스에 몰두하는 경향이 있습니다.
하지만 프로덕션 환경에서는 다양한 예외 상황이 발생할 수 있기 때문에 이를 대비하는 것이 중요합니다.</p>
<p>아래의 주문 처리를 하는 코드 예시를 보겠습니다.</p>
<pre><code class="language-java">// 해피 케이스 예시
public void processOrder(Order order) {
    // 주문 처리 로직
    System.out.println(&quot;Order processed successfully&quot;);
}

// 예외 처리 추가 예시
public void processOrder(Order order) {
    try {
        // 주문 처리 로직
        System.out.println(&quot;Order processed successfully&quot;);
    } catch (IllegalArgumentException e) {
        System.err.println(&quot;Invalid order: &quot; + e.getMessage());
    } catch (Exception e) {
        System.err.println(&quot;An unexpected error occurred: &quot; + e.getMessage());
    }
}</code></pre>
<p>해피 케이스의 경우에는 주문이 계획대로 잘 동작하는 경우만 있습니다.</p>
<p>하지만 예외 케이스의 경우에는 예외 상황을 처리할 수 있는 try/catch 문을 활용하여 대비하는 것을 알아볼 수 있습니다.</p>
<h2 id="2-예외-처리를-잘-하는-방법">2. 예외 처리를 잘 하는 방법</h2>
<p>예외 처리를 잘 하기 위해서는 몇 가지의 원칙을 따르는 것이 좋습니다.</p>
<h3 id="2-1-예외가-발생할-가능성을-낮추기">2-1. 예외가 발생할 가능성을 낮추기</h3>
<p>예외는 주로 <code>외부에서 넘어오는 값</code>에서 발생합니다.</p>
<pre><code class="language-java">public void validateInput(String input) throws InvalidInputException {
    if (input == null || input.isEmpty()) {
        throw new InvalidInputException(&quot;Input cannot be null or empty&quot;);
    }
}</code></pre>
<p>사용자의 입력, 객체 생성자, 외부 서버의 요청 등에 발생할 수 있는 예외를 미리 방지하는 것이 중요합니다.</p>
<h3 id="2-2-의도한-예외와-예상하지-못한-예외-구분하기">2-2. 의도한 예외와 예상하지 못한 예외 구분하기</h3>
<p>예외를 처리할 때, 의도한 예외와 예상하지 못한 예외를 구분하는 것이 중요합니다.</p>
<pre><code class="language-java">try {
    // 코드 실행
} catch (KnownException e) {
    // 사용자에게 보여줄 예외 처리
    System.out.println(&quot;Known error: &quot; + e.getMessage());
} catch (Exception e) {
    // 개발자가 처리해야 할 예외 처리
    e.printStackTrace();
}</code></pre>
<p>사용자에게 보여줄 예외와 개발자가 처리해야할 예외를 명확하게 구분하여 처리해야합니다.</p>
<h2 id="3-null을-대하는-자세">3. Null을 대하는 자세</h2>
<h3 id="3-1-항상-nullpointerexception을-방지해야한다">3-1. 항상 NullPointerException을 방지해야한다.</h3>
<p><code>NullPointerException</code>은 Java에서 흔히 발생하는 예외 중 하나입니다. 그렇기에 메서드 설계 시에 Null을 반환하지 않도록 주의해야합니다.</p>
<pre><code class="language-java">// Null을 반환하는 메서드 (지양)
public String getName() {
    return null;
}

// Optional을 사용하는 메서드 (권장)
public Optional&lt;String&gt; getName() {
    return Optional.ofNullable(name);
}</code></pre>
<p>위의 코드 예시처럼 <code>return null</code>을 하는 메서드 사용은 지양하고 만약 어렵다면 <code>Optional</code> 을 사용하는 것도 대체할 수 있는 좋은 방법이 됩니다.</p>
<h3 id="3-2-optional-사용을-고민해본다">3-2. Optional 사용을 고민해본다.</h3>
<p>Optional을 사용한다면 NullPointerException을 방지할 수 있고, 코드의 명확성도 함께 올릴 수 있습니다.</p>
<pre><code class="language-java">public void printName(Optional&lt;String&gt; name) {
    name.ifPresentOrElse(
        System.out::println,
        () -&gt; System.out.println(&quot;Name is not available&quot;)
    );
}</code></pre>
<p>위의 코드는 Name을 콘솔에 출력하는 내용입니다.
<code>ifPresentElse</code>를 활용하여 만약 존재한다면 해당 name을 콘솔에 출력하지만 존재하지 않는다면 &quot;Name is not available&quot;을 출력하게 되면서 NullPointerException을 방지하게 됩니다.</p>
<h2 id="마치며">마치며</h2>
<p>예외 처리는 코드의 안정성과 사용자 경험을 보장하는 중요한 요소입니다. 해피 케이스에만 몰두하지 않고, 다양한 예외 상황을 대비하여 코드의 품질을 높이는 것이 필요하다는 것을 알게되었습니다. Null을 효과적으로 다루고, 예외를 처리하여 안정적인 어플리케이션을 개발해야합니다.</p>