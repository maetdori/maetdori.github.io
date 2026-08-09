---
title: 스크롤과 페이징
description: 무한 스크롤과 페이징을 UX와 기술 두 관점에서 비교하고, 커서/오프셋 페이징의 성능 차이와 count 쿼리 캐싱까지 정리합니다.
date: '2022.04.27'
pubDate: 2022-04-27
category: Backend · Database
tags: [database, ui·ux]
thumb: /posts/scroll-and-paging/img-01.png
readingTime: 9 min read
---

<figure class="figmock">
  <svg class="mock" viewBox="0 0 720 246" role="img" aria-label="페이징과 무한 스크롤 비교">
    <g transform="translate(8 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/><rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="60" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="72" width="90" height="6" rx="3"/>
      <line class="divider" x1="82" y1="90" x2="258" y2="90"/>
      <rect class="line" x="82" y="102" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="114" width="78" height="6" rx="3"/>
      <line class="divider" x1="82" y1="132" x2="258" y2="132"/>
      <g transform="translate(170 170)"><polyline class="stroke-accent" points="-58,-6 -65,0 -58,6"/><circle class="pg" cx="-38" cy="0" r="4"/><circle class="pg" cx="-19" cy="0" r="4"/><circle class="pg" cx="0" cy="0" r="4"/><circle class="pg" cx="19" cy="0" r="4"/><circle class="pg" cx="38" cy="0" r="4"/><polyline class="stroke-accent" points="58,-6 65,0 58,6"/></g>
    </g>
    <g transform="translate(372 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/><rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="60" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="72" width="90" height="6" rx="3"/>
      <line class="divider" x1="82" y1="90" x2="258" y2="90"/>
      <rect class="line" x="82" y="102" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="114" width="78" height="6" rx="3"/>
      <line class="divider" x1="82" y1="132" x2="258" y2="132"/>
      <circle class="stroke-accent" cx="170" cy="168" r="9" stroke-dasharray="30 16"/>
    </g>
    <text class="label" x="178" y="230" font-size="14" text-anchor="middle">페이징</text>
    <text class="label" x="542" y="230" font-size="14" text-anchor="middle">무한 스크롤</text>
  </svg>
</figure>

<p>목록을 화면에 보여주는 방법은 크게 두 가지다. 페이지 번호로 끊어 보여주는 <strong>페이징</strong>, 그리고 스크롤을 내릴수록 데이터를 계속 이어 붙이는 <strong>무한 스크롤</strong>이다. 페이징은 커뮤니티 게시판·뉴스에서, 무한 스크롤은 SNS 피드·쇼핑몰에서 흔히 볼 수 있다.</p>

<p>둘 다 “많은 데이터를 어떻게 나눠 보여줄 것인가”라는 같은 문제를 풀지만, 사용자에게 주는 경험도, 그것을 떠받치는 쿼리도 꽤 다르다. 이 글에서는 두 방식을 <strong>UX 관점</strong>과 <strong>기술 관점</strong>에서 차례로 비교한다.</p>

<div class="callout">
  <span class="ic">🧭</span>
  <div>
    <p><strong>한 줄 요약</strong></p>
    <p>둘러보는 서비스엔 <mark>무한 스크롤</mark>, 찾아가는 서비스엔 <mark>페이징</mark>. 성능이 중요하면 커서 페이징, 페이지 번호가 꼭 필요하면 오프셋 페이징에 count 캐싱을 얹으면 된다.</p>
  </div>
</div>

## UX의 관점에서

<p>먼저 사용자 입장에서 두 방식이 각각 어떤 경험을 주는지 살펴보자.</p>

### 무한 스크롤

<p>무한 스크롤은 끝이 보이지 않는 콘텐츠를 스크롤만으로 계속 탐색하게 하는 패턴이다. 사용자가 목록 하단에 닿으면 다음 데이터가 자동으로 로드된다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 300" role="img" aria-label="무한 스크롤 인터페이스">
    <rect class="win" x="10" y="10" width="580" height="280" rx="12"/><rect class="bar" x="10" y="10" width="580" height="34" rx="12"/>
    <circle class="dot" cx="34" cy="27" r="4"/><circle class="dot" cx="50" cy="27" r="4"/><circle class="dot" cx="66" cy="27" r="4"/>
    <rect class="field-2" x="210" y="18" width="180" height="18" rx="9"/>
    <rect class="field" x="140" y="66" width="320" height="188" rx="6"/>
    <rect class="line" x="160" y="84" width="260" height="7" rx="3.5"/><rect class="line-soft" x="160" y="98" width="150" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="118" x2="440" y2="118"/>
    <rect class="line" x="160" y="132" width="240" height="7" rx="3.5"/><rect class="line-soft" x="160" y="146" width="170" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="166" x2="440" y2="166"/>
    <rect class="line" x="160" y="180" width="270" height="7" rx="3.5"/><rect class="line-soft" x="160" y="194" width="140" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="214" x2="440" y2="214"/>
    <rect class="line-soft" x="160" y="228" width="120" height="7" rx="3.5"/>
    <circle class="stroke-accent" cx="300" cy="274" r="11" stroke-dasharray="36 20"/>
  </svg>
  <figcaption>스크롤 하단에 닿으면 다음 데이터가 자동으로 로딩된다</figcaption>
</figure>

<p><strong>장점</strong></p>
<ul>
  <li><strong>끊김 없는 몰입</strong> — 페이지를 넘기는 클릭 없이 콘텐츠가 계속 이어져, 가볍게 훑어보는 탐색에 몰입감을 준다. 피드형 서비스가 체류 시간을 끌어올리는 방식이기도 하다.</li>
  <li><strong>역동적인 연출</strong> — 스크롤 위치에 반응하는 인터랙션 등, 페이징으로는 만들기 어려운 표현이 가능하다. (예: <a href="https://www.apple.com/kr/macbook-pro-14-and-16/" target="_blank" rel="noopener">애플 제품 소개 페이지</a>)</li>
  <li><strong>모바일 친화적</strong> — 스크롤은 터치 제스처와 궁합이 좋아 모바일에서 특히 직관적이다.</li>
</ul>
<p><strong>단점</strong></p>
<ul>
  <li><strong>현재 위치를 가늠하기 어렵다</strong> — 전체에서 지금 어디쯤인지, 얼마나 남았는지 알 수 없다. 스크롤바마저 실제 데이터 양을 반영하지 못하고, 로딩될 때마다 크기·위치가 계속 바뀐다.</li>
  <li><strong>특정 항목으로 돌아가기 어렵다</strong> — 항목을 URL로 공유하거나 북마크해 다시 그 지점에 닿기 어렵다. 상세 페이지에 들어갔다 뒤로 가면 스크롤 위치와 이미 불러온 목록이 초기화되기 쉽다.</li>
  <li><strong>길수록 무거워진다</strong> — 스크롤이 길어질수록 쌓인 항목 때문에 화면이 버벅이고 배터리 소모도 늘어, 오래 탐색할수록 오히려 경험이 나빠진다.</li>
  <li><strong>닿을 수 없는 Footer</strong> — 목록이 끝없이 늘어나 하단 Footer에 도달하지 못한다. Footer를 상단·사이드바로 옮기거나(예: <a href="https://www.youtube.com/" target="_blank" rel="noopener">YouTube</a>), “더 보기” 버튼으로 로딩을 사용자 요청에 맡겨 해결한다.</li>
</ul>

### 페이징

<p>페이징은 콘텐츠를 페이지 단위로 끊고, 번호 네비게이션으로 이동하게 하는 패턴이다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 300" role="img" aria-label="페이징 인터페이스">
    <rect class="win" x="10" y="10" width="580" height="280" rx="12"/><rect class="bar" x="10" y="10" width="580" height="34" rx="12"/>
    <circle class="dot" cx="34" cy="27" r="4"/><circle class="dot" cx="50" cy="27" r="4"/><circle class="dot" cx="66" cy="27" r="4"/>
    <rect class="field-2" x="210" y="18" width="180" height="18" rx="9"/>
    <rect class="field" x="140" y="66" width="320" height="160" rx="6"/>
    <rect class="line" x="160" y="84" width="260" height="7" rx="3.5"/><rect class="line-soft" x="160" y="98" width="150" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="118" x2="440" y2="118"/>
    <rect class="line" x="160" y="132" width="240" height="7" rx="3.5"/><rect class="line-soft" x="160" y="146" width="170" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="166" x2="440" y2="166"/>
    <rect class="line" x="160" y="180" width="270" height="7" rx="3.5"/><rect class="line-soft" x="160" y="194" width="140" height="7" rx="3.5"/>
    <g transform="translate(300 258)"><polyline class="stroke-accent" points="-92,-7 -101,0 -92,7"/><circle class="pg" cx="-66" cy="0" r="4.5"/><circle class="pg" cx="-44" cy="0" r="4.5"/><circle class="pg" cx="-22" cy="0" r="4.5"/><circle class="pg" cx="0" cy="0" r="4.5"/><circle class="pg" cx="22" cy="0" r="4.5"/><circle class="pg" cx="44" cy="0" r="4.5"/><circle class="pg" cx="66" cy="0" r="4.5"/><polyline class="stroke-accent" points="92,-7 101,0 92,7"/></g>
  </svg>
  <figcaption>페이지 네비게이션으로 원하는 페이지에 바로 접근할 수 있다</figcaption>
</figure>

<p><strong>장점</strong></p>
<ul>
  <li><strong>검색에 적합</strong> — 단순 탐색이 아니라 특정 항목을 찾는 것이 목적일 때 유용하다.</li>
  <li><strong>통제감</strong> — 전체 결과 수와 페이지가 보이므로, 사용자는 원하는 것을 찾는 데 걸릴 시간을 가늠할 수 있다.
    <blockquote>“종료점에 도달하는 것은 통제력을 제공한다.”<cite>— David Kieras, 인간-컴퓨터 상호작용 심리학</cite></blockquote>
  </li>
  <li><strong>위치 파악과 재접근</strong> — 항목이 몇 페이지에 있는지 알 수 있어 되돌아가기 쉽고, 특정 페이지를 URL로 공유·북마크하기도 좋다. 그래서 전자상거래처럼 항목을 오가며 비교하는 서비스에 잘 맞는다.</li>
  <li><strong>끝이 있다는 완결감</strong> — 무한 스크롤은 끝이 없어 피로를 주지만, 페이징은 “여기까지 봤다”는 종료감을 준다. 다 훑었는지 판단하기 쉬워 이탈 시점도 자연스럽다.</li>
</ul>
<p><strong>단점</strong></p>
<ul>
  <li><strong>추가 조작</strong> — 다음 페이지마다 버튼을 누르고 로딩을 기다려야 하며, 특히 모바일에서 작은 페이지 버튼은 사용성을 떨어뜨린다.</li>
  <li><strong>맥락이 끊긴다</strong> — 페이지가 바뀔 때마다 목록이 새로 로드돼, 쭉 훑어보는 탐색 흐름은 무한 스크롤보다 끊기는 느낌을 준다.</li>
  <li><strong>뒷페이지는 잘 안 본다</strong> — 사용자 대부분이 앞 1~2페이지에 머물러, 뒤쪽 페이지의 콘텐츠는 노출이 급격히 줄어든다.</li>
</ul>

### 정리

<p class="subtitle">탐색이냐 검색이냐</p>

<ul>
  <li>무한 스크롤은 방대한 콘텐츠를 <mark>탐색</mark>할 때, 페이징은 특정 정보를 <mark>검색</mark>할 때 적합하다.</li>
  <li>무한 스크롤은 상대적으로 모바일에, 페이징은 상대적으로 PC 환경에 어울린다.</li>
  <li>그래서 무한 스크롤은 Twitter·Instagram 같은 사용자 생성 콘텐츠 스트리밍 서비스에, 페이징은 사용자가 특정 항목을 찾아가는 목표지향 서비스에 잘 맞는다.</li>
</ul>

<div class="callout">
  <span class="ic">💡</span>
  <div>
    <p><strong>콘텐츠 유형에 따라 방식을 섞을 수도 있다.</strong></p>
    <p>구글이 좋은 예다. 이미지는 텍스트보다 훨씬 빠르게 스캔되므로 구글 이미지는 무한 스크롤을, 한 건씩 읽어야 하는 검색 결과는 여전히 페이징을 쓴다.</p>
  </div>
</div>

## 기술의 관점에서

<p>UX에서 “페이징 vs 무한 스크롤”로 나눴던 것을 구현으로 내려오면 <strong>오프셋 페이징(offset)</strong>과 <strong>커서 페이징(cursor)</strong>이라고 부른다.</p>
<ul>
  <li><strong>오프셋 페이징</strong> — “앞의 N개를 건너뛰고 그다음 M개를 달라.” 페이지 번호를 offset으로 환산해 조회한다. 5페이지를 보려면 앞 4페이지 분량을 건너뛰는 식이다.</li>
  <li><strong>커서 페이징</strong> — “이 항목 ‘다음부터’ M개를 달라.” 마지막으로 읽은 항목을 커서(포인터)로 삼아 그 뒤부터 이어 읽는다.</li>
</ul>

<figure class="figmock">
  <svg class="mock" viewBox="0 0 720 262" role="img" aria-label="오프셋 페이징과 커서 페이징 비교">
    <g transform="translate(8 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/><rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="60" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="72" width="90" height="6" rx="3"/>
      <line class="divider" x1="82" y1="90" x2="258" y2="90"/>
      <rect class="line" x="82" y="102" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="114" width="78" height="6" rx="3"/>
      <line class="divider" x1="82" y1="132" x2="258" y2="132"/>
      <g transform="translate(170 170)"><polyline class="stroke-accent" points="-58,-6 -65,0 -58,6"/><circle class="pg" cx="-38" cy="0" r="4"/><circle class="pg" cx="-19" cy="0" r="4"/><circle class="pg" cx="0" cy="0" r="4"/><circle class="pg" cx="19" cy="0" r="4"/><circle class="pg" cx="38" cy="0" r="4"/><polyline class="stroke-accent" points="58,-6 65,0 58,6"/></g>
    </g>
    <g transform="translate(372 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/><rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="58" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="70" width="88" height="6" rx="3"/>
      <line class="divider" x1="82" y1="86" x2="258" y2="86"/>
      <rect class="line" x="82" y="96" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="108" width="76" height="6" rx="3"/>
      <rect class="accentbox" x="74" y="122" width="192" height="22" rx="5"/><rect class="fill-accent" x="84" y="130" width="120" height="6" rx="3"/>
      <g transform="translate(170 172)"><rect class="accentbox" x="-38" y="-11" width="76" height="22" rx="11"/><text class="label-accent" x="0" y="4" font-size="11" text-anchor="middle">다음 →</text></g>
    </g>
    <text class="label" x="178" y="230" font-size="14" text-anchor="middle">오프셋 페이징</text>
    <text class="label" x="542" y="230" font-size="14" text-anchor="middle">커서 페이징</text>
  </svg>
</figure>

### 오프셋 페이징

<p>오프셋 페이징은 DB의 <code>OFFSET</code> 쿼리로 페이지 단위를 잘라 조회한다. 전통적인 페이징 쿼리는 대체로 이런 형태다.</p>

<pre><span class="kw">SELECT</span> * <span class="kw">FROM</span> items
<span class="kw">WHERE</span> 조건문
<span class="kw">ORDER BY</span> id <span class="kw">DESC</span>
<span class="kw">OFFSET</span> 건너뛸행 <span class="kw">LIMIT</span> 페이지사이즈</pre>

<p>편리하지만, 이 방식에는 두 가지 문제가 있다.</p>
<p><strong>1. 데이터 중복·누락</strong></p>
<ul>
  <li>1페이지를 보는 동안 맨 앞에 3건이 ‘추가’되면, 2페이지에서 1페이지의 마지막 3건을 다시 만난다(중복).</li>
  <li>반대로 앞 3건이 ‘삭제’되면, 2페이지에서 봤어야 할 3건을 건너뛴다(누락).</li>
  <li>그래서 생성·삭제가 잦은 SNS 피드 같은 서비스에는 잘 맞지 않는다.</li>
</ul>
<p><strong>2. offset 쿼리의 성능</strong></p>
<ul><li>뒤 페이지로 갈수록 조회 비용이 커진다.</li></ul>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 210" role="img" aria-label="offset 페이징 비용">
    <text class="ink" x="92" y="34" font-size="15" font-weight="700">OFFSET 10000 · LIMIT 20</text>
    <text class="label" x="92" y="55" font-size="13">→ 10,020개 행을 읽는다</text>
    <rect class="surfbox" x="92" y="74" width="120" height="112" rx="7"/>
    <line class="divider" x1="104" y1="90" x2="200" y2="90" opacity="0.5"/>
    <line class="divider" x1="104" y1="102" x2="200" y2="102" opacity="0.5"/>
    <line class="divider" x1="104" y1="114" x2="200" y2="114" opacity="0.5"/>
    <line class="divider" x1="104" y1="126" x2="200" y2="126" opacity="0.5"/>
    <line class="divider" x1="104" y1="138" x2="200" y2="138" opacity="0.5"/>
    <line class="divider" x1="104" y1="150" x2="200" y2="150" opacity="0.5"/>
    <rect class="fill-accent" x="92" y="164" width="120" height="22" rx="7"/>
    <path class="stroke-muted" d="M212 116 h30"/>
    <text class="label" x="250" y="112" font-size="14"><tspan font-weight="700">앞의 10,000행</tspan> — 읽고 버림 🗑</text>
    <text class="label" x="250" y="134" font-size="12" opacity="0.8">뒤로 갈수록 더 많이 버린다</text>
    <path class="stroke-accent" d="M212 175 h30"/>
    <text class="label-accent" x="250" y="180" font-size="14"><tspan font-weight="700">마지막 20행</tspan> — 실제 사용</text>
  </svg>
  <figcaption>오프셋이 커질수록 읽고 버리는 행이 늘어나 비용이 커진다</figcaption>
</figure>

<p>예를 들어 <code>OFFSET 10000 LIMIT 20</code>이면 DB는 <mark>10,020개 행을 읽은 뒤 앞의 10,000개를 버린다.</mark> 실제로 필요한 건 마지막 20개뿐인데도 말이다. 뒤 페이지일수록 버리려고 읽는 행이 많아져 점점 느려지고 낭비도 커진다.</p>

<p>즉 오프셋 페이징이 원하는 데이터가 <strong>“몇 번째”</strong>에 있는지에 집중한다면, 커서 페이징은 <strong>“어떤 데이터 다음에 오는지”</strong>에 집중한다. 후자는 “이 row 다음부터 20개”만 읽으면 되므로 offset 때문에 생기는 낭비가 없다.</p>

### 커서 페이징

<p>커서 페이징은 직전에 응답한 마지막 데이터를 기준으로, 그 다음 n개를 이어서 조회한다.</p>

<pre><span class="kw">SELECT</span> * <span class="kw">FROM</span> items
<span class="kw">WHERE</span> 조건문
  <span class="kw">AND</span> id &lt; 마지막조회ID  <span class="cm">-- 직전 조회 결과의 마지막 id</span>
<span class="kw">ORDER BY</span> id <span class="kw">DESC</span>
<span class="kw">LIMIT</span> 페이지사이즈</pre>

<p>마지막 조회 ID를 조건으로 쓰므로, 인덱스로 시작 지점을 바로 찾아 <mark>매번 첫 페이지를 읽는 것과 같은 속도</mark>를 낸다. 아무리 뒤로 가도 성능이 일정하다.</p>

<p>다만 커서 페이징에도 단점은 있다.</p>
<p><strong>1. 정렬 제약</strong></p>
<ul>
  <li>“이 레코드 다음”을 가리키려면 커서 기준 컬럼이 <strong>순차적이면서 고유</strong>해야 한다. 중복 값이 있으면 그 경계에서 행이 누락·중복되기 때문이다.</li>
</ul>
<div class="callout">
  <span class="ic">💡</span>
  <div>
    <p>그래서 커서는 보통 <strong>auto-increment PK(<code>id</code>)</strong>를 쓴다.</p>
    <p>정렬 기준이 <code>timestamp</code>처럼 중복될 수 있는 값이라면, <code>(timestamp, id)</code>를 함께 묶어 tie-break한다.</p>
    <pre><span class="kw">WHERE</span> (created_at, id) &lt; (:lastTs, :lastId)</pre>
  </div>
</div>
<p><strong>2. 페이지 네비게이션 불가</strong></p>
<ul>
  <li>“3페이지로 이동” 같은 번호 이동을 만들 수 없다. 정책상 페이지 번호가 반드시 필요하면 커서 페이징을 쓸 수 없다.</li>
</ul>

### 오프셋을 못 버릴 때

<p class="subtitle">count 캐싱</p>

<p>커서 페이징이 성능은 낫지만, 페이지 번호가 꼭 필요한 서비스에선 오프셋을 버릴 수 없다. 이럴 땐 오프셋을 유지한 채로 비용을 조금씩 덜어내야 한다.</p>

<p>먼저 짚고 갈 게 있다. 페이지 단위 조회에는 <strong>두 가지 비용</strong>이 든다.</p>
<ul>
  <li><strong>data 조회</strong> — 목록을 실제로 가져오는 <code>… OFFSET n LIMIT m</code>. 여기엔 <a href="#오프셋-페이징">앞서 본 offset 스캔 비용</a>(뒤 페이지일수록 느려짐)이 들어 있다.</li>
  <li><strong>count 조회</strong> — 페이지 번호를 그리려면 총 건수를 알아야 해서 매번 함께 실행된다.</li>
</ul>
<p>이 섹션에서 줄이려는 건 이 중 <mark>두 번째, 매 요청마다 반복되는 count 비용</mark>이다. offset 스캔 비용은 성격이 다르므로 뒤에서 따로 이야기한다. 그런데 이 count가 생각보다 만만치 않다.</p>

<div class="callout">
  <span class="ic">⚠️</span>
  <div><p>data 조회는 <code>LIMIT</code>만큼 읽고 멈추면 되지만, count는 몇 건인지 세기 위해 <strong>끝까지 다 읽어야</strong> 한다. 건수가 많으면 count가 data 조회보다 더 오래 걸리기도 한다.</p></div>
</div>

<p>해결의 실마리는 <mark>첫 조회의 count 결과를 캐싱</mark>하는 것이다. 처음 검색할 때 나온 count를 응답에 함께 내려 클라이언트(JS)가 들고 있다가, 이후 페이지 이동 요청마다 그 값을 함께 보낸다. 서버는 요청에 캐싱된 count가 있으면 재사용하고, 없을 때만 count 쿼리를 실행한다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 176" role="img" aria-label="count 캐싱 동작">
    <text class="label" x="36" y="51" font-size="13">첫 조회</text>
    <rect class="accentbox" x="104" y="29" width="34" height="34" rx="7"/>
    <text class="label-accent" x="121" y="52" font-size="14" text-anchor="middle" font-weight="700">1</text>
    <g transform="translate(158 46)"><line class="stroke-accent" x1="0" y1="0" x2="34" y2="0"/><polyline class="stroke-accent" points="26,-6 34,0 26,6"/></g>
    <text class="label-accent" x="206" y="51" font-size="14" font-weight="700">count 쿼리 실행</text>
    <text class="label" x="36" y="133" font-size="13">이후 페이지</text>
    <rect class="surfbox" x="104" y="111" width="34" height="34" rx="7"/><text class="label" x="121" y="134" font-size="14" text-anchor="middle">2</text>
    <rect class="surfbox" x="144" y="111" width="34" height="34" rx="7"/><text class="label" x="161" y="134" font-size="14" text-anchor="middle">3</text>
    <rect class="surfbox" x="184" y="111" width="34" height="34" rx="7"/><text class="label" x="201" y="134" font-size="14" text-anchor="middle">4</text>
    <rect class="surfbox" x="224" y="111" width="34" height="34" rx="7"/><text class="label" x="241" y="134" font-size="14" text-anchor="middle">5</text>
    <g transform="translate(278 128)"><line class="stroke-accent" x1="0" y1="0" x2="34" y2="0"/><polyline class="stroke-accent" points="26,-6 34,0 26,6"/></g>
    <text class="label" x="326" y="133" font-size="14">캐시된 count 재사용 · 쿼리 없음</text>
  </svg>
  <figcaption>첫 조회의 count를 캐싱해 매 페이지마다 반복되던 count 쿼리를 없앤다</figcaption>
</figure>

<p>이 방법은 다음 상황에서 특히 효과적이다.</p>
<ul>
  <li>조회가 검색·페이지 이동 양쪽에서 고루 발생하고,</li>
  <li>실시간 적재가 없는, 이미 마감된 데이터를 다룰 때.</li>
</ul>
<p>반대로 이런 경우엔 효과가 없거나 쓰기 어렵다.</p>
<ul>
  <li>대부분 첫 페이지에서 끝난다면 — 매번 새 조회라 캐시할 count가 없다.</li>
  <li>데이터가 실시간으로 자주 바뀌어 총 건수가 계속 달라진다면 — 캐싱된 count가 곧 틀려진다. (새로고침하면 캐시도 초기화된다.)</li>
</ul>
<p>그래서 count 캐싱은 <strong>실시간성이 중요하지 않은 조회</strong> — 예컨대 관리자 페이지에서 특정 사용자의 지난 내역을 확인하는 화면 — 에서 매 페이지마다 반복되던 count 쿼리를 최초 1회로 줄여 준다.</p>

<div class="callout">
  <span class="ic">💡</span>
  <div><p>총 개수가 꼭 정확할 필요까진 없다면 선택지는 더 있다. <strong>서버측 캐시</strong>(Redis 등)에 총 건수를 두거나, <strong>근사치 count</strong>(예: PostgreSQL <code>reltuples</code> 통계)를 쓰거나, 아예 총 개수를 감추고 “다음” 버튼·“1–10 / 많음” 식으로 UI를 바꾸는 것이다.</p></div>
</div>

<p>물론 이걸로 오프셋이 완전히 해결되는 건 아니다. 남은 건 <strong>offset 스캔 비용</strong> — count 캐싱으로는 사라지지 않는다. 다만 <a href="#페이징">사용자 대부분이 앞 몇 페이지만 본다</a>는 점 덕분에 실무에선 덜 치명적이고, <strong>최대 페이지 수를 제한</strong>하거나 <strong>커버링 인덱스</strong>로 스캔을 가볍게 해 완화한다. 그럼에도 깊은 페이지까지 빠르게 넘나들어야 한다면, 결국 답은 커서 페이징이다.</p>
<p class="fnote"><strong>커버링 인덱스</strong> — 쿼리가 읽는 컬럼을 인덱스가 모두 담고 있어, 테이블 본체를 다시 읽지 않고 인덱스만으로 응답하는 것. 건너뛸 행을 가벼운 인덱스에서만 훑게 해 스캔 부담을 덜어준다.</p>

## 마무리

<p>무한 스크롤과 페이징의 선택은 단순한 UI 취향이 아니라 <strong>데이터를 어떻게 나눠 읽고 보여줄지</strong>에 대한 결정이다. 그리고 그 결정은 UX에서 시작해 쿼리 전략까지 이어진다.</p>
<ul>
  <li>사용자가 <strong>둘러보는</strong> 서비스라면 → 무한 스크롤 + 커서 페이징이 자연스럽다.</li>
  <li>사용자가 <strong>찾아가는</strong> 서비스라면 → 페이징이 맞고, 성능이 걱정되면 커서 페이징으로 바꾸거나 오프셋에 count 캐싱을 얹는다.</li>
</ul>
<p>결국 “어떤 경험을 줄지”가 정해지면 쿼리 전략은 대체로 그 결정을 따라온다. 그 순서만 기억해도 상황에 맞는 선택이 한결 쉬워진다.</p>

<div class="refs">
  <p class="refs-lbl">참고 자료</p>
  <ul>
    <li><a href="https://uxplanet.org/ux-infinite-scrolling-vs-pagination-1030d29376f1" target="_blank" rel="noopener">UX: Infinite Scrolling vs. Pagination</a><span class="dom">uxplanet.org <span class="ext">↗</span></span></li>
    <li><a href="https://use-the-index-luke.com/sql/partial-results/fetch-next-page" target="_blank" rel="noopener">Use The Index, Luke — Fetch Next Page</a><span class="dom">use-the-index-luke.com <span class="ext">↗</span></span></li>
  </ul>
</div>
