<h1 id="github-action-403-error">Github Action 403 Error</h1>
<p>Github Actions 사용 시에 <code>The requestd URL returned error: 403</code> 발생한 오류 해결에 대해서 알아보겠습니다.</p>
<h2 id="에러-발생-원인">에러 발생 원인</h2>
<p>깃허브 액션을 통해 자동으로 README 파일을 작성해줄 수 있도록 설정하는 와중에
아래 에러가 발생하였습니다.</p>
<h2 id="에러-내용">에러 내용</h2>
<blockquote>
<p>Run git config --global user.name 'github-actions[bot]'
remote: Write access to repository not granted.
fatal: unable to access '<a href="https://github.com/kdmin0706/velog_post.git/'">https://github.com/kdmin0706/velog_post.git/'</a>: The requested URL returned error: 403
Error: Process completed with exit code 128.</p>
</blockquote>
<h2 id="에러-해결-내용">에러 해결 내용</h2>
<p>해당 에러의 원인은 권한이나 토큰 문제 등 다양한 상황이 발생할 수 있다고 합니다.
본인의 경우에는 <code>workflow 권한</code> 문제를 파악하여 해결하였습니다.</p>
<h3 id="해결-방법">해결 방법</h3>
<h4 id="1-해당-레포지토리---setting---actions---general">1. 해당 레포지토리 -&gt; setting -&gt; Actions -&gt; General</h4>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/c812f9f8-bb8e-471a-bed2-96b0ba5e1b78/image.png" /></p>
<h4 id="2-workflow-permissions---read-and-write-permissions-권한-설정">2. Workflow Permissions -&gt; Read and write permissions 권한 설정</h4>
<p><img alt="" src="https://velog.velcdn.com/images/kdmin0706/post/d18a71a7-5f2e-4cfb-9ecb-73d3e7dad340/image.png" /></p>
<h2 id="reference">Reference</h2>
<p><a href="https://stackoverflow.com/questions/73687176/permission-denied-to-github-actionsbot-the-requested-url-returned-error-403">https://stackoverflow.com/questions/73687176/permission-denied-to-github-actionsbot-the-requested-url-returned-error-403</a></p>