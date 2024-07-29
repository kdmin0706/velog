<h1 id="들어가며">들어가며</h1>
<p>프로그래밍에서 매직 넘버와 매직 스트링은 코드의 가독성과 유지보수성을 저해하는 주요 원인 중 하나입니다.
이번 포스팅에서는 매직 넘버와 매직 스트링이 무엇인지?, 그리고 이를 개선하는 방안에 대해 알아보겠습니다.</p>
<h2 id="매직-넘버magic-number">매직 넘버(Magic Number)</h2>
<p><code>매직 넘버</code>는 코드 내에 <code>하드코딩된 숫자</code>를 의미합니다.
이러한 숫자는 그 의미를 알기 어렵고 코드를 이해하는 데 혼란을 야기할 수 있습니다.</p>
<p>아래 예시를 보면서 알아보겠습니다.</p>
<pre><code class="language-java">public class MagicNumberExample {
    public static void main(String[] args) {
        int salary = 50000;
        double tax = salary * 0.2;  // 0.2라는 매직 넘버가 사용됨
        System.out.println(&quot;Tax amount: &quot; + tax);
    }
}
</code></pre>
<p>위의 코드에서 <code>0.2</code>는 매직 넘버입니다.
해당 숫자가 무엇을 의미하는 지는 바로 알기 어렵습니다.</p>
<p>아래의 코드는 위의 코드를 리팩토링을 한 코드입니다.</p>
<pre><code class="language-java">public class MagicNumberExample {
    private static final double TAX_RATE = 0.2;  // TAX_RATE라는 상수로 대체

    public static void main(String[] args) {
        int salary = 50000;
        double tax = salary * TAX_RATE;  // 의미 있는 상수를 사용
        System.out.println(&quot;Tax amount: &quot; + tax);
    }
}</code></pre>
<p>위의 코드에서 <code>TAX_RATE</code>라는 의미 있는 이름의 상수를 사용하여 가독성을 높였습니다.</p>
<h2 id="매직-스트링magic-string">매직 스트링(Magic String)</h2>
<p><code>매직 스트링</code>은 코드 내에 <code>하드코딩 된 문자열</code>을 의미합니다.
이 또한 매직 넘버와 동일하게 코드의 가독성을 떨어뜨리고, 유지 보수를 어렵게 만듭니다.</p>
<p>아래 예시를 보면서 알아보겠습니다.</p>
<pre><code class="language-java">public class MagicStringExample {
    public static void main(String[] args) {
        String role = &quot;admin&quot;;

        if (role.equals(&quot;admin&quot;)) {  // &quot;admin&quot;이라는 매직 스트링이 사용됨
            System.out.println(&quot;You have admin access.&quot;);
        }
    }
}</code></pre>
<p>위 코드에서 <code>admin</code> 은 매직 스트링입니다.
매직 스트링을 리팩토링을 진행하면 아래의 코드와 같이 구현할 수 있습니다.</p>
<pre><code class="language-java">public class MagicStringExample {
    private static final String ROLE_ADMIN = &quot;admin&quot;;  // ROLE_ADMIN이라는 상수로 대체

    public static void main(String[] args) {
        String role = ROLE_ADMIN;

        if (role.equals(ROLE_ADMIN)) {  // 의미 있는 상수를 사용
            System.out.println(&quot;You have admin access.&quot;);
        }
    }
}</code></pre>
<p>위 코드에서는 <code>ROLE_ADMIN</code> 이라는 의미 있는 이름의 상수를 사용하여 가독성을 높였습니다.</p>
<h2 id="마무리">마무리</h2>
<p>매직 넘버와 매직 스트링을 의미 있는 상수로 치환하면 코드의 가독성과 유지 보수성을 크게 향상 시킬 수 있습니다.
항상 코드를 작성할 때 이러한 점을 염두에 두고, 매직 넘버와 매직 스트링을 피하도록 노력하면 깨끗한 코드 즉, 클린 코드를 구현하는 데 도움이 될 것이라고 생각됩니다.</p>