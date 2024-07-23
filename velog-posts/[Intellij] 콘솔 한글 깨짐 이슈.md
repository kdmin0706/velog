<h1 id="들어가며">들어가며</h1>
<p><code>Intellij</code>를 활용하여 프로덕션 코드의 콘솔을 실행시켰을 때 한글이 깨져서 출력되는 경우가 발생한 적이 있습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/73c5e364-78d0-4eda-98db-e6ab24c181e2/image.png" /></p>
<p>위의 이미지처럼 기존에 출력하려는 문자들이 아닌
알 수 없는 문자들이 출력되는 것을 확인할 수 있습니다.</p>
<p>위의 이슈를 해결할 수 있는 방안에 대해 알아보겠습니다.</p>
<h2 id="file-encoding-변경">File Encoding 변경</h2>
<p>가장 먼저 해볼 방안은 인텔리제이 IDE의 환경 설정 중 <code>File Encodigs</code> 설정 환경을 변경하는 방법입니다.</p>
<p>Intellij 상단 탭 -&gt; Settings -&gt; File Encodings(파일 인코딩)</p>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/2516a980-0cb8-452d-b412-a3d41df995a5/image.png" /></p>
<p>이 화면에서 <code>Global Encodings</code>, <code>Project Encoding</code>, <code>Properties Files</code>를 <code>UTF-8</code>로 설정한다.</p>
<h2 id="vm-options-변경">VM Options 변경</h2>
<p>두 번째 방법은 <code>사용자 지정 VM Options</code>를 변경하는 방법이 있습니다.</p>
<p>Intellij 상단 탭 -&gt; 도움말(Help) -&gt; Help-Edit Custom VM Options(사용자 지정 VM Options 편집)</p>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/5b163987-0e57-46b6-9400-0f83d7038af0/image.png" /></p>
<p>이 파일에서 맨 아래에 <code>-Dfile.encoding=UTF-8</code>, <code>-Dconsole.encoding=UTF-8</code>을 추가한다. 
위의 옵션들을 추가했다면, IntelliJ를 <code>재시작</code>한다.</p>
<h2 id="gradle-rebuild">Gradle rebuild</h2>
<p>위의 2가지를 진행했다면 마지막으로는 <code>Gradle</code>를 <code>rebuild</code>를 진행합니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/8bfbc735-95f6-4d9f-afb2-66c6b8f24017/image.png" /></p>
<p>인텔리제이 IDE 기준 우측 탭에 있는 <code>Gradle</code> 아이콘을 클릭을 합니다.
클릭을 하면 Tasks -&gt; build 탭을 확인합니다.</p>
<p><code>clean</code>을 진행하고 <code>build</code>를 하여
현재 생성되어 있는 코드를 정리하고 빌드를 진행합니다.</p>
<h2 id="마무리">마무리</h2>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/881205b6-20f0-47f8-afce-3e4ff9d38c06/image.png" /></p>
<p>위의 3가지를 순서대로 진행하면
기존에 에러가 발생했던 콘솔 창의 한글 인코딩 출력 문제가 해결되는 것을 확인할 수 있습니다.</p>