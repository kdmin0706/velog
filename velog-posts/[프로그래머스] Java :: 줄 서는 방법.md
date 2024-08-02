<h2 id="문제-설명">문제 설명</h2>
<p>n명의 사람이 일렬로 줄을 서고 있습니다. n명의 사람들에게는 각각 1번부터 n번까지 번호가 매겨져 있습니다. n명이 사람을 줄을 서는 방법은 여러가지 방법이 있습니다. 예를 들어서 3명의 사람이 있다면 다음과 같이 6개의 방법이 있습니다.</p>
<ul>
<li>[1, 2, 3]</li>
<li>[1, 3, 2]</li>
<li>[2, 1, 3]</li>
<li>[2, 3, 1]</li>
<li>[3, 1, 2]</li>
<li>[3, 2, 1]</li>
</ul>
<p>사람의 수 n과, 자연수 k가 주어질 때, 사람을 나열 하는 방법을 사전 순으로 나열 했을 때, k번째 방법을 return하는 solution 함수를 완성해주세요.</p>
<h3 id="제한사항">제한사항</h3>
<ul>
<li>n은 20이하의 자연수 입니다.</li>
<li>k는 n! 이하의 자연수 입니다.</li>
</ul>
<h2 id="입출력-예">입출력 예</h2>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/2f0f3e14-ce87-4312-8bcd-aeda7151bee1/image.png" /></p>
<h2 id="코드">코드</h2>
<pre><code class="language-java">import java.util.*;

class Solution {
    public int[] solution(int n, long k) {        
        int[] answer = new int[n];
        List&lt;Integer&gt; list = new ArrayList&lt;&gt;();

        long f = 1;

        // 리스트 초기화 및 팩토리얼 계산
        for(int i=1;i&lt;=n;i++) {
            list.add(i);
            f *= i;
        }

        k--;

        for (int i = 0; i &lt; n; i++) {
            f /= (n - i);
            int index = (int) (k / f);
            answer[i] = list.remove(index);
            k %= f;
        }


        return answer;
    }
}</code></pre>
<h2 id="문제-풀이">문제 풀이</h2>
<ol>
<li><code>list</code> 리스트에 1부터 n까지의 숫자를 추가합니다.</li>
<li><code>f</code>라는 변수는 <code>n!</code> 값을 계산합니다.</li>
<li><code>0-based</code> 인덱스로 변환<ul>
<li>즉, 문제에서 주어진 k는 1부터 시작하는 <code>1-based 인덱스</code>를 사용합니다. </li>
<li>따라서, 이를 0-based 인덱스로 변환해야 정확한 위치의 순열을 찾을 수 있습니다.<ul>
<li><code>0-based 인덱스</code> : 인덱스가 <code>0</code>부터 시작하는 방식</li>
<li><code>1-based 인덱스</code> : 인덱스가 <code>1</code>부터 시작하는 방식</li>
</ul>
</li>
</ul>
</li>
<li>각 자리수마다 팩토리얼 값을 감소시키면서, 해당 자리에서 선택할 숫자의 인덱스를 계산합니다.<ol>
<li>첫 번째 자리의 경우<ul>
<li>처음에 <code>f</code>는 <code>n!(전체 순열의 수)</code>를 저장하고 있습니다.</li>
<li>첫 번째 올 수 있는 경우의 수는 <code>(n-1)!</code> 이므로, 이를 계산하기 위해서 f을 (n-1)로 나눕니다.</li>
<li>예시) n이 3이고, 처음 f는 <code>6(3!)</code>이고, 이를 2(3-1)로 나누면 <code>3!/2 = 3</code>가 됩니다. 즉, 첫 번째 자리에서 각 숫자가 나올 경우의 수는 <code>2</code>입니다.</li>
</ul>
</li>
<li>두 번째 자리의 경우<ul>
<li>두 번째 자리에서는 남은 <code>n-1</code>명의 사람을 나열하는 수를 계산해야합니다.</li>
<li>이를 위해 <code>f</code>를 <code>(n-2)</code>로 나눕니다.</li>
<li>예시) n이 3이면, 두 번째 올 수 있는 경우의 수는 <code>1! = 1</code>입니다. 즉, 두 번째 자리에서는 각 숫자가 올 수 있는 경우의 수는 <code>1</code>입니다.</li>
</ul>
</li>
</ol>
</li>
<li><code>index</code> 는 현재 자리에서 선택할 숫자의 인덱스입니다.<ul>
<li>선택된 숫자를 <code>list</code> 리스트에서 제거하고, 결과 배열에 추가합니다.</li>
<li><code>k</code>값을 갱신하여 다음 자리 수에서 사용할 값을 설정합니다.</li>
</ul>
</li>
<li>최종적으로 구해진 결과 배열을 출력합니다.</li>
</ol>