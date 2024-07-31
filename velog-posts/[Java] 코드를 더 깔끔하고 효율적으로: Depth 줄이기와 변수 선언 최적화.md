<h2 id="들어가며">들어가며</h2>
<p>Java 코드를 작성하다보면, 깊이 있는 중첩 반복문이나 조건문으로 인해 코드의 가독성이 떨어지고 유지보수가 어려운 경우가 많습니다.</p>
<p>이러한 문제를 해결하기 위해 <code>메서드 추출</code>과 <code>변수 선언 최적화</code> 를 통해 코드를 더 깔끔하고 효율적으로 만드는 방법에 대해 알아보겠습니다.</p>
<h2 id="메서드로-추출하여-depth-줄이기">메서드로 추출하여 Depth 줄이기</h2>
<p>중첩 반복문이나 조건문은 코드의 깊이를 증가시켜 가독성을 떨어뜨립니다.</p>
<p>이를 해결하기 위해 메서드를 추출하여 코드를 단순화하고, 추상화를 통해 논리적인 구조를 개선할 수 있습니다.</p>
<p>아래의 코드는 중첩 반복문과 조건문이 있는 코드의 예시입니다.
이중 for문을 활용하여 인덱스 값이 홀수, 짝수인 경우의 값을 달리 콘솔에 출력합니다.</p>
<pre><code class="language-java">public class Main {
    public static void main(String[] args) {
        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        for (int i = 0; i &lt; matrix.length; i++) {
            for (int j = 0; j &lt; matrix[i].length; j++) {
                if (matrix[i][j] % 2 == 0) {
                    System.out.println(matrix[i][j] + &quot; is even.&quot;);
                } else {
                    System.out.println(matrix[i][j] + &quot; is odd.&quot;);
                }
            }
        }
    }
}</code></pre>
<p>위의 코드를 <code>메서드를 추출</code>하여 리팩토링한 예시를 알아보겠습니다.</p>
<pre><code class="language-java">public class Main {
    public static void main(String[] args) {
        int[][] matrix = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };

        printMatrixParity(matrix);
    }

    // 행렬의 요소가 짝수인지 홀수인지 출력하는 메서드
    private static void printMatrixParity(int[][] matrix) {
        for (int[] row : matrix) {
            printRowParity(row);
        }
    }

    // 행의 요소가 짝수인지 홀수인지 출력하는 메서드
    private static void printRowParity(int[] row) {
        for (int value : row) {
            printValueParity(value);
        }
    }

    // 요소가 짝수인지 홀수인지 출력하는 메서드
    private static void printValueParity(int value) {
        if (value % 2 == 0) {
            System.out.println(value + &quot; is even.&quot;);
        } else {
            System.out.println(value + &quot; is odd.&quot;);
        }
    }
}
</code></pre>
<p>위의 리팩토링한 코드를 파악해보겠습니다.</p>
<ol>
<li><p>기능별로 메서드 추출</p>
<blockquote>
<ol>
<li>행렬의 요소를 출력하는 기능을 printMatrixParity 메서드로 추출했습니다.</li>
<li>행의 요소를 출력하는 기능을 printRowParity 메서드로 추출했습니다.</li>
<li>요소가 짝수인지 홀수인지 판단하고 출력하는 기능을 printValueParity 메서드로 추출했습니다.</li>
</ol>
</blockquote>
</li>
<li><p>추상화</p>
<blockquote>
<p>각 메서드는 자신을 역할을 명확히 하며, 상위 메서드는 하위 메서드를 호출하여 처리합니다.</p>
</blockquote>
</li>
<li><p>Depth 감소</p>
<blockquote>
<p>중첩된 반복문과 조건문을 메서드로 분리하여 Depth를 줄여, 코드의 논리적 구조를 개선하여 이해하기 쉽게 만들 수 있습니다.</p>
</blockquote>
</li>
</ol>
<p><a></a></p>
<h3 id="메서드로-추출하여-depth-줄일-때-유의점-❗">메서드로 추출하여 Depth 줄일 때 유의점 ❗</h3>
<blockquote>
<p>💡 무조건 <code>1 Depth로 만들어라</code>는 아니다.</p>
</blockquote>
<p>즉, 보이는 depth를 줄이는 데 급급하는 것이 아닌, <code>추상화</code>를 통한 <code>사고 과정의 depth</code>를 줄이는 것이 중요하다.</p>
<p>만약, 2중 중첩 구조를 표현하는 것이 사고하는 데에 더 도움이 된다고 판단된다면,
메서드 분리보다 그대로 놔두는 것이 더 나은 선택일 수 있다. 때로는 메서드를 분리하는 것이 혼선을 야기할 수 있다는 점은 생각해야한다.</p>
<h2 id="사용하는-변수는-선언과-가까이-하여-사용한다">사용하는 변수는 선언과 가까이 하여 사용한다.</h2>
<p>변수 선언을 사용한 곳과 가깝게 위치시키면 코드의 가독성이 높아지고, 변수의 유효 범위를 쉽게 파악할 수 있습니다.</p>
<pre><code class="language-java">// 리팩토링 전
public class Main {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;

        if (a &gt; b) {
            System.out.println(&quot;a is greater than b&quot;);
        } else {
            System.out.println(&quot;a is not greater than b&quot;);
        }

        int sum = a + b;
        System.out.println(&quot;Sum: &quot; + sum);
    }
}


// 리팩토링 후
public class Main {
    public static void main(String[] args) {
        int a = 10;
        int b = 20;
        int sum = a + b;    // 변수 위치 변경

        if (a &gt; b) {
            System.out.println(&quot;a is greater than b&quot;);
        } else {
            System.out.println(&quot;a is not greater than b&quot;);
        }

        System.out.println(&quot;Sum: &quot; + sum);
    }
}

</code></pre>
<p>위의 코드 예시를 보면 리팩토링 전과 후의 다른 점은 변수 <code>sum</code>을 사용하는 곳의 위치가
변수와 가까이 선언되어 있다는 것입니다.</p>
<p>이를 보면 <code>변수의 유효범위</code>가 줄어들기에 코드의 가독성과 유지보수성이 향상됩니다.</p>
<h2 id="마무리">마무리</h2>
<p>중첩 반복문과 조건문을 메서드로 추출하여 Depth를 줄이는 것은 코드의 가독성과 유지보수성을 크게 향상시키는 중요한 기법입니다. 단순히 보이는 Depth만 줄이는 것이 아니라, 각 기능을 명확히 분리하고 추상화를 통해 코드 구조를 개선함으로써 더 이해하기 쉬운 코드를 작성할 수 있습니다.</p>