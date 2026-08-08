---
title: 스크롤과 페이징
description: 무한 스크롤과 페이징을 UX와 기술 두 관점에서 비교하고, 커서/오프셋 페이징의 성능 차이와 count 쿼리 캐싱까지 정리합니다.
date: '2022.04.13'
pubDate: 2022-04-13
category: Backend · Database
tags: [backend, database, ux]
thumb: /posts/scroll-and-paging/img-01.png
readingTime: 8 min read
---

<figure class="figmock">
  <svg class="mock" viewBox="0 0 720 246" role="img" aria-label="페이징과 무한 스크롤 비교">
    <!-- 왼쪽: 페이징 -->
    <g transform="translate(8 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/>
      <rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="60" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="72" width="90" height="6" rx="3"/>
      <line class="divider" x1="82" y1="90" x2="258" y2="90"/>
      <rect class="line" x="82" y="102" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="114" width="78" height="6" rx="3"/>
      <line class="divider" x1="82" y1="132" x2="258" y2="132"/>
      <g transform="translate(170 170)">
        <polyline class="stroke-accent" points="-58,-6 -65,0 -58,6"/>
        <circle class="pg" cx="-38" cy="0" r="4"/><circle class="pg" cx="-19" cy="0" r="4"/><circle class="pg" cx="0" cy="0" r="4"/><circle class="pg" cx="19" cy="0" r="4"/><circle class="pg" cx="38" cy="0" r="4"/>
        <polyline class="stroke-accent" points="58,-6 65,0 58,6"/>
      </g>
    </g>
    <!-- 오른쪽: 무한 스크롤 -->
    <g transform="translate(372 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/>
      <rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="60" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="72" width="90" height="6" rx="3"/>
      <line class="divider" x1="82" y1="90" x2="258" y2="90"/>
      <rect class="line" x="82" y="102" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="114" width="78" height="6" rx="3"/>
      <line class="divider" x1="82" y1="132" x2="258" y2="132"/>
      <circle class="stroke-accent" cx="170" cy="168" r="9" stroke-dasharray="30 16"/>
    </g>
    <!-- 라벨: 창 바깥(카드 위) -->
    <text class="label" x="178" y="230" font-size="14" text-anchor="middle">페이징</text>
    <text class="label" x="542" y="230" font-size="14" text-anchor="middle">무한 스크롤</text>
  </svg>
</figure>

**스크롤**과 **페이징**은 모두 사용자에게 데이터를 보여주기 위한 하나의 방법이다.

**페이징** 방식은 페이지 번호로 구분하여 한번에 일정 개수만 보여주는 방식을 말한다. 가장 흔하게 볼 수 있는 방식으로 커뮤니티 게시판, 뉴스 등에서 사용된다.

**무한 스크롤** 방식은 사용자가 스크롤을 내릴 때마다 새로운 데이터가 자동으로 로딩되어 마치 끝이 없는 것처럼 보인다. SNS 피드 혹은 쇼핑몰에서 자주 사용되는 방식이다.

이 글에서는 두 가지를 UX적 관점, 그리고 기술적 관점 두 가지 측면에서 비교 분석해보려 한다.

## UX의 관점에서

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 300" role="img" aria-label="무한 스크롤 인터페이스">
    <rect class="win" x="10" y="10" width="580" height="280" rx="12"/>
    <rect class="bar" x="10" y="10" width="580" height="34" rx="12"/>
    <circle class="dot" cx="34" cy="27" r="4"/><circle class="dot" cx="50" cy="27" r="4"/><circle class="dot" cx="66" cy="27" r="4"/>
    <rect class="field-2" x="210" y="18" width="180" height="18" rx="9"/>
    <rect class="field" x="140" y="66" width="320" height="188" rx="6"/>
    <g>
      <rect class="line" x="160" y="84" width="260" height="7" rx="3.5"/><rect class="line-soft" x="160" y="98" width="150" height="7" rx="3.5"/>
      <line class="divider" x1="160" y1="118" x2="440" y2="118"/>
      <rect class="line" x="160" y="132" width="240" height="7" rx="3.5"/><rect class="line-soft" x="160" y="146" width="170" height="7" rx="3.5"/>
      <line class="divider" x1="160" y1="166" x2="440" y2="166"/>
      <rect class="line" x="160" y="180" width="270" height="7" rx="3.5"/><rect class="line-soft" x="160" y="194" width="140" height="7" rx="3.5"/>
      <line class="divider" x1="160" y1="214" x2="440" y2="214"/>
      <rect class="line-soft" x="160" y="228" width="120" height="7" rx="3.5"/>
    </g>
    <circle class="stroke-accent" cx="300" cy="274" r="11" stroke-dasharray="36 20"/>
  </svg>
  <figcaption>스크롤 하단에 닿으면 다음 데이터가 자동으로 로딩된다</figcaption>
</figure>

**무한 스크롤**은 마감선이 보이지 않는 상태에서 방대한 양의 콘텐츠를 스크롤할 수 있는 인터페이스 패턴이다. 이 기술을 구현한 페이지에서는 사용자가 스크롤하여 하단에 닿을 때 새로운 페이지가 로드된다.

**장점**

- 뛰어난 사용자 인터랙션
  - 튜토리얼과 같이 연속적이고 긴 콘텐츠에서 뛰어난 사용성을 제공한다.
  - 페이징에서 보여줄 수 없는 역동성을 선보일 수 있다. (ex. [애플 스크롤 인터랙션](https://www.apple.com/kr/macbook-pro-14-and-16/))
- 모바일 친화적인 인터페이스
  - 모바일 디바이스의 제스처 제어는 스크롤을 사용하기에 직관적이고 편리하다.

**단점**

- 표시할 수 없는 항목 위치
  - 특정 지점에서의 위치를 표시할 수 없으며, 특정 지점으로 이동하기 어렵다.
- 무의미해진 스크롤바
  - 스크롤바가 실제 데이터 양을 반영하지 못한다.
  - 스크롤바의 크기와 위치가 고정되어있지 않고 하단에 도착할 때마다 변경된다.
- 도달할 수 없는 Footer
  - 방법 1: Footer를 상단 또는 사이드바에 재배치한다. (e.g., [Youtube](https://www.youtube.com/))
  - 방법 2: 추가 로딩 버튼을 사용하여 요청 시 콘텐츠를 로딩하도록 한다.

## 페이징

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 300" role="img" aria-label="페이징 인터페이스">
    <rect class="win" x="10" y="10" width="580" height="280" rx="12"/>
    <rect class="bar" x="10" y="10" width="580" height="34" rx="12"/>
    <circle class="dot" cx="34" cy="27" r="4"/><circle class="dot" cx="50" cy="27" r="4"/><circle class="dot" cx="66" cy="27" r="4"/>
    <rect class="field-2" x="210" y="18" width="180" height="18" rx="9"/>
    <rect class="field" x="140" y="66" width="320" height="160" rx="6"/>
    <g>
      <rect class="line" x="160" y="84" width="260" height="7" rx="3.5"/><rect class="line-soft" x="160" y="98" width="150" height="7" rx="3.5"/>
      <line class="divider" x1="160" y1="118" x2="440" y2="118"/>
      <rect class="line" x="160" y="132" width="240" height="7" rx="3.5"/><rect class="line-soft" x="160" y="146" width="170" height="7" rx="3.5"/>
      <line class="divider" x1="160" y1="166" x2="440" y2="166"/>
      <rect class="line" x="160" y="180" width="270" height="7" rx="3.5"/><rect class="line-soft" x="160" y="194" width="140" height="7" rx="3.5"/>
    </g>
    <g transform="translate(300 258)">
      <polyline class="stroke-accent" points="-92,-7 -101,0 -92,7"/>
      <circle class="pg" cx="-66" cy="0" r="4.5"/><circle class="pg" cx="-44" cy="0" r="4.5"/><circle class="pg" cx="-22" cy="0" r="4.5"/><circle class="pg" cx="0" cy="0" r="4.5"/><circle class="pg" cx="22" cy="0" r="4.5"/><circle class="pg" cx="44" cy="0" r="4.5"/><circle class="pg" cx="66" cy="0" r="4.5"/>
      <polyline class="stroke-accent" points="92,-7 101,0 92,7"/>
    </g>
  </svg>
  <figcaption>페이지 네비게이션으로 원하는 페이지에 바로 접근할 수 있다</figcaption>
</figure>

**페이징**은 콘텐츠를 별도의 페이지로 나누는 사용자 인터페이스 패턴이다. 사용자는 페이지 네비게이션을 통해 다른 페이지로 이동할 수 있다.

**장점**

- 검색에 적합한 인터페이스
  - 사용자의 목적이 단순 탐색이 아닌, 특정 항목에 대한 검색일 때 유용하다.
- 통제가능성
  - 사용자가 필요로 하는 정보의 양 또한 데이터로 제공할 필요가 있다.

    > *“종료점에 도달하는 것은 통제력을 제공한다.”*
    > <cite>— David Kieras, 인간-컴퓨터 상호작용 심리학</cite>

  - 사용자가 총 검색 결과의 수를 볼 때 (총 데이터 양이 무한하지 않은 경우) 실제로 찾고 있는 것을 찾는 데 걸리는 시간을 예상할 수 있다.
- 표시 가능해진 항목 위치
  - 사용자는 항목의 위치를 알 수 있고, 특정 위치로 이동하기 쉽다.
  - 전자 상거래 서비스에 적합하다.

**단점**

- 추가작업
  - 다음 페이지로 이동하기 위해서는 버튼을 클릭하고, 페이지의 로딩을 기다려야 한다.
  - 모바일 환경에서의 페이지 버튼은 사용성을 떨어뜨린다.

## 정리

- 무한 스크롤은 상대적으로 모바일 환경에, 페이징은 상대적으로 PC 환경에 적합하다.
- 무한 스크롤은 방대한 양의 콘텐츠를 <mark>탐색</mark>할 때, 페이징은 특정 정보를 <mark>검색</mark>하고자 할 때 적합하다.
- 무한 스크롤은 Twitter, Facebook, Pinterest, Instagram과 같이 사용자 생성 콘텐츠의 스트리밍 사이트 또는 앱에 가장 적합하다. 반면 페이징은 사용자가 특정 항목을 찾는 목표지향 사이트 및 앱에 적합하다.

<div class="callout">
  <span class="ic">💡</span>
  <div>
    <p><strong>콘텐츠 유형에 따라 검색 방법을 선택하는 방법도 있다.</strong></p>
    <p>구글이 좋은 예시가 될 수 있다. 사용자는 텍스트보다 훨씬 빠르게 이미지를 스캔하고 처리하기 때문에 구글 이미지는 무한 스크롤을 사용한다. 반면 검색 결과를 읽는 데는 훨씬 오래 걸리기 때문에 구글 검색 결과는 여전히 전통적인 페이징 방법을 사용하고 있다.</p>
  </div>
</div>

## 기술의 관점에서

<figure class="figmock">
  <svg class="mock" viewBox="0 0 720 262" role="img" aria-label="오프셋 페이징과 커서 페이징 비교">
    <!-- 오프셋: 페이지 번호로 접근 -->
    <g transform="translate(8 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/>
      <rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="60" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="72" width="90" height="6" rx="3"/>
      <line class="divider" x1="82" y1="90" x2="258" y2="90"/>
      <rect class="line" x="82" y="102" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="114" width="78" height="6" rx="3"/>
      <line class="divider" x1="82" y1="132" x2="258" y2="132"/>
      <g transform="translate(170 170)">
        <polyline class="stroke-accent" points="-58,-6 -65,0 -58,6"/>
        <circle class="pg" cx="-38" cy="0" r="4"/><circle class="pg" cx="-19" cy="0" r="4"/><circle class="pg" cx="0" cy="0" r="4"/><circle class="pg" cx="19" cy="0" r="4"/><circle class="pg" cx="38" cy="0" r="4"/>
        <polyline class="stroke-accent" points="58,-6 65,0 58,6"/>
      </g>
    </g>
    <!-- 커서: 마지막 항목 다음부터 -->
    <g transform="translate(372 8)">
      <rect class="win" x="0" y="0" width="340" height="196" rx="10"/>
      <rect class="bar" x="0" y="0" width="340" height="28" rx="10"/>
      <circle class="dot" cx="18" cy="14" r="3.5"/><circle class="dot" cx="30" cy="14" r="3.5"/><circle class="dot" cx="42" cy="14" r="3.5"/>
      <rect class="field" x="66" y="44" width="208" height="104" rx="6"/>
      <rect class="line" x="82" y="58" width="150" height="6" rx="3"/><rect class="line-soft" x="82" y="70" width="88" height="6" rx="3"/>
      <line class="divider" x1="82" y1="86" x2="258" y2="86"/>
      <rect class="line" x="82" y="96" width="140" height="6" rx="3"/><rect class="line-soft" x="82" y="108" width="76" height="6" rx="3"/>
      <rect class="accentbox" x="74" y="122" width="192" height="22" rx="5"/>
      <rect class="fill-accent" x="84" y="130" width="120" height="6" rx="3"/>
      <g transform="translate(170 172)">
        <rect class="accentbox" x="-38" y="-11" width="76" height="22" rx="11"/>
        <text class="label-accent" x="0" y="4" font-size="11" text-anchor="middle">다음 →</text>
      </g>
    </g>
    <!-- 라벨: 창 바깥(카드 위) -->
    <text class="label" x="178" y="230" font-size="14" text-anchor="middle">오프셋 페이징</text>
    <text class="label" x="542" y="230" font-size="14" text-anchor="middle">커서 페이징</text>
  </svg>
</figure>

앞에서는 스크롤과 페이징으로 설명했지만, 기술적인 관점에서 둘을 **커서 페이징**과 **오프셋 페이징**으로 칭할 수 있다.

**오프셋 페이징**의 경우 80페이지를 조회한다고 하면, 79페이지의 offset을 가지기 때문에 이를 오프셋 페이징이라고 한다.

**커서 페이징**의 경우 커서 = 포인터라고 생각하면 되는데, 특정 항목까지 조회했을 때 이 아이템을 커서로 가리켜놓고 그 다음 항목부터 조회해온다고 해서 커서 페이징이다.

### 오프셋 페이징

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 260" role="img" aria-label="오프셋 페이징 동작">
    <rect class="win" x="10" y="10" width="580" height="240" rx="12"/>
    <rect class="bar" x="10" y="10" width="580" height="34" rx="12"/>
    <circle class="dot" cx="34" cy="27" r="4"/><circle class="dot" cx="50" cy="27" r="4"/><circle class="dot" cx="66" cy="27" r="4"/>
    <rect class="field" x="140" y="66" width="320" height="120" rx="6"/>
    <rect class="line" x="160" y="82" width="260" height="7" rx="3.5"/><rect class="line-soft" x="160" y="96" width="150" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="116" x2="440" y2="116"/>
    <rect class="line" x="160" y="130" width="240" height="7" rx="3.5"/><rect class="line-soft" x="160" y="144" width="170" height="7" rx="3.5"/>
    <line class="divider" x1="160" y1="164" x2="440" y2="164"/>
    <rect class="line" x="160" y="176" width="270" height="7" rx="3.5"/>
    <g transform="translate(300 218)">
      <polyline class="stroke-accent" points="-92,-7 -101,0 -92,7"/>
      <circle class="pg" cx="-66" cy="0" r="4.5"/><circle class="pg" cx="-44" cy="0" r="4.5"/><circle class="pg" cx="-22" cy="0" r="4.5"/><circle class="pg" cx="0" cy="0" r="4.5"/><circle class="pg" cx="22" cy="0" r="4.5"/><circle class="pg" cx="44" cy="0" r="4.5"/><circle class="pg" cx="66" cy="0" r="4.5"/>
      <polyline class="stroke-accent" points="92,-7 101,0 92,7"/>
    </g>
  </svg>
</figure>

오프셋 페이징은 DB의 `offset` 쿼리를 사용하여 페이지 단위로 요청 및 응답한다. 전통적인 방식의 페이징 쿼리는 일반적으로 다음과 같은 형태이다.

<pre><span class="kw">SELECT</span> * <span class="kw">FROM</span> items
<span class="kw">WHERE</span> 조건문
<span class="kw">ORDER BY</span> id <span class="kw">DESC</span>
<span class="kw">OFFSET</span> 건너뛸행 <span class="kw">LIMIT</span> 페이지사이즈</pre>

이와 같은 형태의 페이징 쿼리는 2가지 문제점을 가지고 있다.

1. **데이터 중복 및 누락**
   - 1페이지를 조회하는 중에 누군가가 3개의 데이터를 맨 앞에 추가할 경우, 2페이지로 넘어갔을 때 1페이지에서 보았던 마지막 3개 데이터를 2페이지에서 다시 만나게 된다.
   - 1페이지를 조회하는 중에 누군가가 첫 3개의 데이터를 삭제했을 경우, 2페이지로 넘어갔을 때 2페이지에서 조회했어야 할 첫 3개 데이터를 지나치게 된다.
   - 잦은 수정, 생성, 삭제가 반복되는 페이스북이나 인스타그램과 같은 서비스에는 부적합하다.
2. **offset 쿼리의 성능 이슈**
   - offset 쿼리를 사용하면 뒤로 갈수록 페이지 조회에 드는 비용이 커진다.

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 250" role="img" aria-label="offset 페이징 비용">
    <text class="ink" x="40" y="34" font-size="15" font-weight="700">OFFSET 10000 · LIMIT 20</text>
    <text class="label" x="40" y="54" font-size="13">→ 10,020개 행을 읽어야 한다</text>
    <!-- 스캔한 행 스택 -->
    <rect class="fill-surf2 win" x="50" y="74" width="120" height="126" rx="6"/>
    <rect class="fill-accent" x="50" y="176" width="120" height="24" rx="6"/>
    <line class="divider" x1="50" y1="176" x2="170" y2="176"/>
    <!-- 연결선 -->
    <path class="stroke-muted" d="M170 128 h34" />
    <path class="stroke-accent" d="M170 188 h34" />
    <text class="label" x="212" y="124" font-size="14">앞의 <tspan font-weight="700">10,000행</tspan> — 읽고 그냥 버림 🗑</text>
    <text class="label" x="212" y="152" font-size="12" opacity="0.8">뒤로 갈수록 버리는 행이 늘어 느려진다</text>
    <text class="label-accent" x="212" y="192" font-size="14">마지막 <tspan font-weight="700">20행</tspan> — 실제 사용</text>
  </svg>
  <figcaption>오프셋이 커질수록 읽고 버리는 행이 늘어나 비용이 커진다</figcaption>
</figure>

예를 들어 `offset 10000, limit 20`이라 하면 최종적으로 <mark>10,020개의 행을 읽어야 한다. 그리고 이 중 앞의 10,000개 행을 버리게 된다.</mark> (실제 필요한 건 마지막 20개뿐이니) 뒤로 갈수록 버리지만 읽어야 할 행의 개수가 많아 뒤로 갈수록 느려지고, 낭비도 커진다.

이렇게 오프셋 페이징은 “n개의 row를 읽은 다음 20개 주세요”와 같은 쿼리문을 사용하기 때문에 성능 저하가 발생하는 것인데, 반면에 커서 페이징은 “이 row 다음부터 20개 주세요”와 같은 쿼리문을 사용하기 때문에 offset으로 인한 성능저하가 없다.

즉, offset 페이징은 우리가 원하는 데이터가 “**몇 번째**”에 있는지에 집중하고 있다면, cursor 페이징은 우리가 원하는 데이터가 “**어떤 데이터의 다음에 오는지**”에 집중한다.

### 커서 페이징

커서 페이징은 사용자에게 응답해준 마지막 데이터를 기준으로 다음 n개를 요청 및 응답한다.

<pre><span class="kw">SELECT</span> * <span class="kw">FROM</span> items
<span class="kw">WHERE</span> 조건문
  <span class="kw">AND</span> id &lt; 마지막조회ID  <span class="cm">-- 직전 조회 결과의 마지막 id</span>
<span class="kw">ORDER BY</span> id <span class="kw">DESC</span>
<span class="kw">LIMIT</span> 페이지사이즈</pre>

마지막 조회 결과의 ID를 조건문에 사용하기 때문에 쿼리를 실행하면 조회 시작 부분을 인덱스로 빠르게 찾아 매번 첫 페이지만 읽는다. 즉, 아무리 페이지가 뒤로 가더라도 처음 페이지를 읽은 것과 동일한 성능을 가지게 된다.

성능면에서는 cursor 페이징이 offset 페이징보다 뛰어나지만, 다음과 같은 단점도 가지고 있다.

1. **제한된 정렬 기능**
   - 커서 페이징은 정렬할 컬럼에 중복된 값이 존재하면 안 된다.
   - “이 레코드 다음 레코드를 조회해줘”라고 할 수 있도록, 특정 지점을 커서로 지정할 수 있어야 하기 때문이다.

     <div class="callout">
       <span class="ic">💡</span>
       <div><p>그래서 대부분의 커서 페이징은 <code>timestamp</code> 컬럼을 기준으로 한다. <code>timestamp</code>는 순차적이고 고유하기 때문이다.</p></div>
     </div>

2. **페이지 네비게이션 구현 불가**
   - 회사 혹은 서비스 정책상 무조건 페이지 네비게이션이 있어야 한다면 답이 없다.

위에서처럼, 사업적인 이유로 성능을 차치하고서라도 반드시 offset 페이징을 사용해야 하는 상황이 있다. 따라서 페이징 쿼리 자체를 건드리지 않고 다른 방법으로 성능을 개선할 수 있는 방법이 필요하다.

## Pagination 성능 최적화 방안

대부분의 경우에, 페이지 단위로 조회하는 메서드에서는 매번 Count 조회와 Data 조회가 함께 일어난다. 조회되는 총 건수를 알아야만 페이지 번호를 노출할 수 있기 때문이다.

그러나, 조회 건수에 따라 count 쿼리는 실제 데이터 조회만큼 오래 걸릴 수도 있다. 총 몇 건인지 확인하기 위해 전체를 확인해야 하기 때문이다.

<div class="callout">
  <span class="ic">💡</span>
  <div><p>데이터 조회는 limit 10 등으로 지정된 사이즈만큼 읽고 나서는 더 이상 읽지 않아도 되지만, count는 끝까지 읽어서 몇 건인지 확인해야 한다.</p></div>
</div>

count 쿼리로 인한 성능 이슈를 <mark>첫 페이지 조회 결과를 caching</mark> 함으로써 해결할 수 있다. 처음 검색 시 조회된 count 결과를 응답결과로 내려주어 JS에서 이를 캐싱하고, 매 페이징 버튼마다 count 결과를 함께 내려주는 것이다. 그리고 서버에서는 요청에 넘어온 항목 중, 캐싱된 count 값이 있으면 이를 재사용하고, 없으면 count 쿼리를 수행한다.

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 200" role="img" aria-label="count 결과 캐싱">
    <g transform="translate(300 58)">
      <polyline class="stroke-accent" points="-150,-6 -159,0 -150,6"/>
      <rect class="surfbox" x="-140" y="-16" width="30" height="32" rx="6"/>
      <rect class="surfbox" x="-100" y="-16" width="30" height="32" rx="6"/>
      <rect class="accentbox" x="-60" y="-16" width="30" height="32" rx="6"/>
      <rect class="surfbox" x="-20" y="-16" width="30" height="32" rx="6"/>
      <rect class="surfbox" x="20" y="-16" width="30" height="32" rx="6"/>
      <text class="label" x="-125" y="5" font-size="13" text-anchor="middle">1</text>
      <text class="label" x="-85" y="5" font-size="13" text-anchor="middle">2</text>
      <text class="label-accent" x="-45" y="5" font-size="13" text-anchor="middle">3</text>
      <text class="label" x="-5" y="5" font-size="13" text-anchor="middle">4</text>
      <text class="label" x="35" y="5" font-size="13" text-anchor="middle">5</text>
      <polyline class="stroke-accent" points="70,-6 79,0 70,6"/>
    </g>
    <rect class="accentbox" x="120" y="112" width="360" height="52" rx="10"/>
    <text class="label-accent" x="300" y="136" font-size="14" text-anchor="middle" font-weight="700">count 쿼리 → 최초 1회만 실행</text>
    <text class="label" x="300" y="154" font-size="12" text-anchor="middle">이후 페이지 이동은 캐시된 count 재사용</text>
  </svg>
  <figcaption>첫 조회의 count 결과를 캐싱해 매 페이지마다 반복되는 count 쿼리를 없앤다</figcaption>
</figure>

이 방식은 다음과 같은 상황에서 도움이 된다.

- 조회 요청이 검색 버튼과 페이지 버튼 모두에서 골고루 발생하고
- 실시간으로 데이터가 적재되지 않으며, 마감된 데이터를 사용할 경우

이 방법은 count 쿼리를 최초 1회만 호출하기 때문에 조회 성능을 향상시킬 수 있다. 하지만 JS에서 캐싱하고 있기 때문에 브라우저를 새로고침하게 되면 count가 초기화가 되어 다시 불러와야 한다.

또한 다음과 같은 상황에서는 이 방법이 유효하지 않을 수 있다.

- 첫 페이지 조회가 대부분일 경우 효과가 없다.
  - 추가적인 페이징 조회가 필요하지 않으면 결국 매번 첫 조회라서 cache count를 사용할 수 없다.
- 실시간으로 잦은 데이터 수정이 일어나 페이지 버튼 변경이 반영되어야 하는 경우 사용할 수 없다.
  - 마감된 데이터 혹은 실시간을 유지할 필요가 없는 경우에만 사용할 수 있다.

count를 캐싱하는 최적화는, 관리자 페이지에서 특정 사용자의 내역을 확인하는 등 **실시간성이 중요하지 않은 조회**에서 특히 효과적이다. 이런 화면에서는 count를 캐싱함으로써 매 페이지 이동마다 발생하던 count 쿼리를 최초 1회로 줄여 성능상의 이점을 누릴 수 있다.

<div class="callout">
  <span class="ic">🔗</span>
  <div>
    <p><strong>Reference</strong></p>
    <p>
      <a href="https://uxplanet.org/ux-infinite-scrolling-vs-pagination-1030d29376f1" target="_blank" rel="noopener">UX: Infinite Scrolling vs. Pagination</a><br />
      <a href="https://use-the-index-luke.com/sql/partial-results/fetch-next-page" target="_blank" rel="noopener">Use The Index, Luke — Fetch Next Page</a><br />
      <a href="https://www.eversql.com/faster-pagination-in-mysql-why-order-by-with-limit-and-offset-is-slow/" target="_blank" rel="noopener">Faster Pagination in MySQL</a><br />
      <a href="https://jojoldu.tistory.com/528" target="_blank" rel="noopener">jojoldu — 페이징 성능 개선 (1)</a><br />
      <a href="https://jojoldu.tistory.com/531" target="_blank" rel="noopener">jojoldu — 페이징 성능 개선 (2)</a>
    </p>
  </div>
</div>
