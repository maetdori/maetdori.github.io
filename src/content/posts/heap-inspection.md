---
title: Heap Inspection
description: 메모리 덤프로 민감정보가 털리지 않으려면 — 왜 String 대신 char[]를 써야 하는지 JVM 메모리 생애주기로 파헤칩니다.
date: '2023.04.19'
pubDate: 2023-04-19
category: Java · Security
tags: [java, security, jvm]
thumb: /posts/heap-inspection/img-01.png
readingTime: 9 min read
---

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 210" role="img" aria-label="메모리 덤프에서 민감정보를 읽어내는 Heap Inspection">
    <rect class="field" x="96" y="24" width="408" height="162" rx="14"/>
    <text class="label" x="120" y="52" font-size="13">MEMORY DUMP</text>
    <g>
      <rect class="fill-surf2" x="120" y="68" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="206" y="68" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="292" y="68" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="378" y="68" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="120" y="102" width="76" height="24" rx="5"/>
      <rect class="accentbox" x="212" y="100" width="100" height="28" rx="5"/>
      <text class="label-accent" x="262" y="119" font-size="13" text-anchor="middle">p4$$w0rd</text>
      <rect class="fill-surf2" x="326" y="102" width="60" height="24" rx="5"/>
      <rect class="fill-surf2" x="396" y="102" width="58" height="24" rx="5"/>
      <rect class="fill-surf2" x="120" y="136" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="206" y="136" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="292" y="136" width="76" height="24" rx="5"/>
      <rect class="fill-surf2" x="378" y="136" width="76" height="24" rx="5"/>
    </g>
    <!-- 돋보기: 노출된 패스워드를 찾아낸다 -->
    <line class="stroke-accent" x1="292" y1="144" x2="322" y2="174" style="stroke-width:11;stroke-linecap:round"/>
    <circle class="stroke-accent" cx="262" cy="114" r="43" stroke-width="6"/>
    <circle class="stroke-accent" cx="262" cy="114" r="34" stroke-width="1.5" opacity="0.4"/>
  </svg>
  <figcaption>메모리 덤프를 뒤지면 그 안에 남아 있던 패스워드가 그대로 읽힌다</figcaption>
</figure>

<p>애플리케이션에 아무리 촘촘하게 보안 모듈을 붙여도, 정작 패스워드가 <strong>메모리에 평문으로 떠 있는 시간</strong>은 좀처럼 신경 쓰지 않는다. 하지만 공격자가 프로세스 메모리를 통째로 떠낼 수 있다면, 암호화 로직을 우회할 필요도 없이 그 순간의 스냅샷에서 값을 그대로 집어 가면 그만이다. 이것을 <strong>Heap Inspection</strong>이라고 부른다.</p>

<p>까다로운 건, 이 취약점은 라이브러리나 프레임워크가 알아서 막아 주지 않는다는 점이다. 오히려 우리가 흔히 쓰는 자료형 선택 하나가 문제의 시작이 된다. 이 글에서는 <strong>왜 패스워드를 <code>String</code>에 담으면 안 되는지</strong>를 JVM이 메모리를 할당하고 해제하는 방식에서부터 따라가 본다.</p>

<div class="callout">
  <span class="ic">🧭</span>
  <div>
    <p><strong>한 줄 요약</strong></p>
    <p>패스워드 같은 민감 정보는 <mark>String 대신 char[]</mark>에 담고, 다 쓰면 즉시 덮어써라. String은 불변이라 지울 방법이 없어 <mark>GC가 수거하기 전까지 메모리에 그대로 남고</mark>, 그 틈이 메모리 덤프에 노출되기 때문이다.</p>
  </div>
</div>

## Heap Inspection이란

<p><strong>Heap Inspection</strong>은 메모리 덤프와 같이 메모리에 남아 있는 데이터를 직접 읽어 정보를 빼내는 공격이다. 메모리에 손을 대는 순간, 그 안에 올라와 있던 패스워드 같은 민감 정보가 그대로 탈취될 수 있다.</p>

<p class="fnote"><strong>메모리 덤프</strong> — 실행 중인 프로세스가 쓰던 메모리 영역을 파일로 떠내는 것. 장애 분석용으로도 쓰이지만, 그 안에 평문 민감 정보가 남아 있으면 고스란히 탈취 표적이 된다.</p>

<p>그래서 예방의 핵심은 하나다. <mark>민감한 정보가 메모리에 올라와 있는 시간을 최소화하는 것.</mark> 필요할 때만 잠깐 두고, 다 쓰면 곧바로 메모리에서 지워야 한다. 이 문장이 이 글 전체를 관통하는 목표다.</p>

## 메모리의 생애주기

<p>그렇다면 데이터는 메모리에서 어떻게 사라질까? String과 char[]의 운명이 갈리는 지점이 바로 여기, <strong>메모리에서 해제되는 방식</strong>이다. 그 차이를 보려면 먼저 메모리가 어떻게 태어나고 사라지는지부터 알아야 한다.</p>

<p>프로그램이 메모리를 다루는 흐름은 단순하다. <strong>할당</strong>받아, <strong>사용</strong>하고, 다 쓰면 <strong>해제</strong>한다. 이 세 단계를 기준으로 String과 char[]이 각각 어떻게 움직이는지 따라가 보자.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 140" role="img" aria-label="메모리 생애주기: 한 칸의 상태 변화">
  <rect class="surfbox" x="16" y="22" width="158" height="104" rx="14"/>
  <text class="ink" x="95" y="56" font-size="15" text-anchor="middle" font-weight="700">Allocate</text>
  <text class="label" x="95" y="76" font-size="11" text-anchor="middle">할당 · 빈 칸 확보</text>
  <rect class="surfbox" x="53" y="92" width="84" height="16" rx="4"/>
  <g transform="translate(197 74)"><line class="stroke-accent" x1="-18" y1="0" x2="16" y2="0" style="stroke-width:3"/><polyline class="stroke-accent" points="8,-7 16,0 8,7" style="stroke-width:3"/></g>
  <rect class="surfbox" x="221" y="22" width="158" height="104" rx="14"/>
  <text class="ink" x="300" y="56" font-size="15" text-anchor="middle" font-weight="700">Use</text>
  <text class="label" x="300" y="76" font-size="11" text-anchor="middle">사용 · 값이 참</text>
  <rect class="accentbox" x="258" y="92" width="84" height="16" rx="4"/>
  <rect class="fill-accent" x="266" y="97" width="68" height="6" rx="3"/>
  <g transform="translate(402 74)"><line class="stroke-accent" x1="-18" y1="0" x2="16" y2="0" style="stroke-width:3"/><polyline class="stroke-accent" points="8,-7 16,0 8,7" style="stroke-width:3"/></g>
  <rect class="surfbox" x="426" y="22" width="158" height="104" rx="14"/>
  <text class="ink" x="505" y="56" font-size="15" text-anchor="middle" font-weight="700">Deallocate</text>
  <text class="label" x="505" y="76" font-size="11" text-anchor="middle">해제 · 값이 지워짐</text>
  <rect class="surfbox" x="463" y="92" width="84" height="16" rx="4" style="stroke-dasharray:4 3" opacity="0.55"/>
  <rect class="fill-accent" x="471" y="97" width="68" height="6" rx="3" opacity="0.18"/>
</svg>
  <figcaption>할당 → 사용 → 해제. 문제는 마지막 “해제”가 자료형마다 다르게 일어난다는 점이다</figcaption>
</figure>

## String은 지워지지 않는다

<pre><span class="kw">String</span> password = <span class="st">"helloWorld"</span>;</pre>

<p>패스워드를 이렇게 <code>String</code>에 담아 쓴다고 하자. 그러면 참조 변수 <code>password</code>는 Stack에, 실제 문자열 값은 Heap에 자리를 잡는다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 230" role="img" aria-label="String 메모리 할당">
  <rect class="field-2" x="30" y="54" width="232" height="150" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="48" y="40" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="108" y="60" font-size="13" text-anchor="middle">Stack 영역</text>
  <rect class="field-2" x="338" y="54" width="232" height="150" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="356" y="40" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="416" y="60" font-size="13" text-anchor="middle">Heap 영역</text>
  <rect class="surfbox" x="66" y="112" width="158" height="54" rx="11"/>
  <text class="ink" x="145" y="145" font-size="15" text-anchor="middle">password</text>
  <rect class="accentbox" x="360" y="112" width="188" height="54" rx="11"/>
  <text class="label-accent" x="454" y="145" font-size="15" text-anchor="middle">"helloWorld"</text>
  <line class="stroke-accent" x1="226" y1="139" x2="352" y2="139" style="stroke-width:3"/>
  <polyline class="stroke-accent" points="344,133 352,139 344,145" style="stroke-width:3"/>
</svg>
  <figcaption>Stack의 참조 변수가 Heap에 올라온 문자열 값을 가리킨다</figcaption>
</figure>

<p>이제 다 쓴 <code>password</code>를 메모리에서 지우고 싶다. 떠올릴 수 있는 방법은 두 가지인데, 결론부터 말하면 <strong>둘 다 완전하지 않다.</strong></p>

### 방법 1. null로 초기화

<p>JVM의 가비지 컬렉터(GC)는 Stack에서 더 이상 참조하지 않는 Heap 데이터를 수거한다. 그러니 <code>password = null</code>로 참조를 끊으면 문자열이 수거 대상이 될 거라 기대할 수 있다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 230" role="img" aria-label="password를 null로 초기화">
  <rect class="field-2" x="30" y="54" width="232" height="150" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="48" y="40" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="108" y="60" font-size="13" text-anchor="middle">Stack 영역</text>
  <rect class="field-2" x="338" y="54" width="232" height="150" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="356" y="40" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="416" y="60" font-size="13" text-anchor="middle">Heap 영역</text>
  <rect class="surfbox" x="66" y="106" width="158" height="60" rx="11"/>
  <text class="ink" x="145" y="132" font-size="15" text-anchor="middle">password</text>
  <text class="label" x="145" y="152" font-size="12" text-anchor="middle">= null</text>
  <rect class="surfbox" x="360" y="112" width="188" height="54" rx="11" style="stroke-dasharray:5 4" opacity="0.5"/>
  <text class="label" x="454" y="134" font-size="15" text-anchor="middle" opacity="0.75">"helloWorld"</text>
  <text class="label" x="454" y="153" font-size="10" text-anchor="middle" opacity="0.75">참조 없음 · GC 대상</text>
  <line class="stroke-muted" x1="226" y1="139" x2="352" y2="139" style="stroke-dasharray:5 5" opacity="0.5"/>
  <line class="stroke-accent" x1="279" y1="128" x2="299" y2="150" style="stroke-width:3"/>
  <line class="stroke-accent" x1="299" y1="128" x2="279" y2="150" style="stroke-width:3"/>
</svg>
  <figcaption>참조는 끊겼지만, “helloWorld”는 GC가 실행될 때까지 힙에 그대로 있다</figcaption>
</figure>

<p>참조가 끊긴 “helloWorld”는 이제 어디서도 쓰이지 않으니 GC의 수거 대상이 되긴 한다. 하지만 함정이 있다. <mark>가비지 컬렉션이 실제로 실행되기 전까지, “helloWorld”라는 패스워드는 여전히 메모리에 남아 있다.</mark> 그리고 GC가 ‘언제’ 돌지는 우리가 정하지 못한다. 그 틈이 바로 Heap Inspection의 노출 구간이다.</p>

<p class="fnote"><strong>문자열 리터럴은 더 나쁘다</strong> — 코드에 직접 박은 리터럴(<code>"helloWorld"</code>)은 String Pool에 인터닝돼 사실상 프로그램 내내 수거되지 않는다. 반면 실제 패스워드는 입력에서 만들어지는 풀 밖의 인스턴스다. 이 글의 예시도 (편의상 리터럴로 썼지만) 그런 인스턴스로 보면 된다.</p>

### 방법 2. 다른 값으로 덮어쓰기

<pre>password = <span class="st">"helloPassword"</span>;</pre>

<p>그렇다면 참조를 끊는 대신 아예 다른 값으로 덮어쓰면 어떨까? 이 방법도 통하지 않는다. <strong>String은 불변(immutable)</strong>이라 한 번 만들어진 문자열은 절대 바뀌지 않기 때문이다.</p>

<p>재할당은 기존 “helloWorld”를 고치는 게 아니다. JVM은 “helloPassword”라는 새 문자열을 힙에 따로 만들고, <code>password</code>가 가리키는 주소만 그쪽으로 옮긴다. 결과적으로 힙에는 두 문자열이 <strong>동시에</strong> 존재하게 된다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 262" role="img" aria-label="재할당 전후 메모리 상태">
  <rect class="field-2" x="30" y="54" width="232" height="184" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="48" y="40" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="108" y="60" font-size="13" text-anchor="middle">Stack 영역</text>
  <rect class="field-2" x="338" y="54" width="232" height="184" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="356" y="40" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="416" y="60" font-size="13" text-anchor="middle">Heap 영역</text>
  <rect class="surfbox" x="66" y="125" width="158" height="54" rx="11"/>
  <text class="ink" x="145" y="158" font-size="15" text-anchor="middle">password</text>
  <rect class="surfbox" x="360" y="88" width="188" height="52" rx="10" style="stroke-dasharray:5 4" opacity="0.5"/>
  <text class="label" x="454" y="112" font-size="14" text-anchor="middle" opacity="0.75">"helloWorld"</text>
  <text class="label" x="454" y="129" font-size="10" text-anchor="middle" opacity="0.7">여전히 잔존</text>
  <rect class="accentbox" x="360" y="164" width="188" height="52" rx="11"/>
  <text class="label-accent" x="454" y="196" font-size="14" text-anchor="middle">"helloPassword"</text>
  <line class="stroke-accent" x1="226" y1="158" x2="352" y2="188" style="stroke-width:3"/>
  <polyline class="stroke-accent" points="344,183 352,189 342,194" style="stroke-width:3"/>
  <line class="stroke-muted" x1="226" y1="146" x2="352" y2="116" style="stroke-dasharray:5 5" opacity="0.45"/>
</svg>
  <figcaption>참조만 새 문자열로 옮겨갈 뿐, 기존 “helloWorld”는 힙에 그대로 남는다</figcaption>
</figure>

<p>두 방법을 나란히 놓고 보면 문제가 분명해진다.</p>
<ul>
  <li><strong>null로 초기화</strong> — 참조만 끊을 뿐, “helloWorld” 문자열 자체는 GC 전까지 메모리에 남는다.</li>
  <li><strong>다른 값으로 덮어쓰기</strong> — 값을 바꾼 게 아니라 참조만 옮긴 것이라, 기존 “helloWorld”는 그대로 남는다.</li>
</ul>

<p>결국 String을 쓰는 한 어떤 수를 써도 가비지 컬렉션 전까지는 일정 시간 메모리에 잔존한다. 그리고 그 시간이, 메모리 덤프에 노출되는 시간이다.</p>

## char[]는 덮어쓸 수 있다

<p>String의 한계는 분명해졌다. 그렇다면 대안은 뭘까? 열쇠는 <strong>가변(mutable)</strong> 자료형에 있다. char[]은 String과 달리 값 자체를 직접 바꿀 수 있다.</p>

<pre><span class="kw">char</span>[] password = {<span class="st">'s'</span>, <span class="st">'e'</span>, <span class="st">'c'</span>, <span class="st">'r'</span>, <span class="st">'e'</span>, <span class="st">'t'</span>};
<span class="cm">// 사용이 끝나면 그 자리를 스페이스(0x20)로 덮어쓴다</span>
Arrays.<span class="kw">fill</span>(password, (<span class="kw">char</span>) 0x20);</pre>

<p>다 사용한 char[] 패스워드를 <code>Arrays.fill</code>로 스페이스로 덮어쓰면, “secret”이라는 값은 <mark>있던 그 자리에서 바로 지워진다.</mark> GC를 기다릴 필요도, 새 객체가 생길 일도 없다. char[]에 담긴 민감 데이터는 다 쓴 즉시 그 자리를 다른 값으로 덮어써(overwrite) Heap Inspection을 막을 수 있다.</p>

<p>사실 이건 새삼스러운 발견이 아니다. 표준 라이브러리도 정확히 같은 이유로 패스워드를 char[]에 담는다. <code>PBEKeySpec</code> 문서는 그 이유를 이렇게 못 박는다.</p>

<blockquote>This class stores passwords as char arrays instead of <code>String</code> objects (which would seem more logical), because the <code>String</code> class is immutable and there is no way to overwrite its internal value when the password stored in it is no longer needed. Hence, this class requests the password as a char array, so it can be overwritten when done.<cite>— Java Platform SE, <code>javax.crypto.spec.PBEKeySpec</code></cite></blockquote>

<div class="callout">
  <span class="ic">✅</span>
  <div>
    <p><strong>정리하면</strong></p>
    <p><code>String</code>은 불변이라 GC 전까지 지울 수 없고, <code>char[]</code>은 가변이라 다 쓴 즉시 덮어쓸 수 있다. 그래서 <strong>민감 정보를 다룰 때는 char[]에 담아 사용 후 overwrite하는 것이 안전하다.</strong></p>
  </div>
</div>

## 그냥 GC를 부르면 안 될까

<p>여기서 이런 반문이 나올 법하다. “char[]로 일일이 덮어쓸 것 없이, String을 쓰되 다 쓴 뒤 <code>System.gc()</code>로 GC를 직접 불러 지우면 되지 않나?”</p>

<p>안타깝지만 그것도 답이 아니다. <mark><code>System.gc()</code>는 가비지 컬렉션을 ‘요청’할 뿐, 즉시 실행을 ‘보장’하지 않기 때문이다.</mark></p>

<p>이유는 GC의 설계에 있다. 가비지 컬렉션이 돌 때 JVM은 애플리케이션 스레드를 잠깐 멈춘다. 이를 <strong>Stop the World</strong>라 하는데, 이 구간 동안에는 애플리케이션이 사실상 정지하므로 성능에 직접적인 영향을 준다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 250" role="img" aria-label="가비지 컬렉션과 Stop the World">
  <rect class="field-2" x="30" y="58" width="404" height="150" rx="14" style="stroke:var(--border);stroke-width:1.2"/>
  <rect class="fill-accent" x="48" y="44" width="120" height="30" rx="8"/><text class="label-accent" style="fill:var(--bg)" x="108" y="64" font-size="13" text-anchor="middle">Heap 영역</text>
  <rect class="accentbox" x="52" y="112" width="112" height="48" rx="10"/>
  <text class="label-accent" x="108" y="141" font-size="13" text-anchor="middle">live</text>
  <rect class="surfbox" x="176" y="112" width="112" height="48" rx="10" style="stroke-dasharray:5 4" opacity="0.5"/>
  <text class="label" x="232" y="141" font-size="13" text-anchor="middle" opacity="0.7">garbage</text>
  <rect class="surfbox" x="300" y="112" width="112" height="48" rx="10" style="stroke-dasharray:5 4" opacity="0.5"/>
  <text class="label" x="356" y="141" font-size="13" text-anchor="middle" opacity="0.7">garbage</text>
  <!-- GC → 쓰레기통 -->
  <g transform="translate(452 136)"><line class="stroke-accent" x1="-18" y1="0" x2="18" y2="0" style="stroke-width:3"/><polyline class="stroke-accent" points="10,-7 18,0 10,7" style="stroke-width:3"/></g>
  <text class="label-accent" x="513" y="104" font-size="13" text-anchor="middle" font-weight="700">GC</text>
  <text x="513" y="152" font-size="42" text-anchor="middle">🗑</text>
  <text class="label" x="232" y="236" font-size="12" text-anchor="middle">Stop the World — GC 동안 애플리케이션 스레드 일시 정지</text>
</svg>
  <figcaption>GC는 공짜가 아니다. 그래서 JVM은 언제 돌릴지를 스스로 판단한다</figcaption>
</figure>

<p>비용이 있으니 JVM은 자체 알고리즘으로 가장 효율적인 시점을 골라 GC를 돌린다. 게다가 <code>System.gc()</code>는 강제 명령이 아니라 명세상 <strong>“요청(hint)”</strong>일 뿐이라, <code>-XX:+DisableExplicitGC</code> 옵션으로 아예 무시되게 막아둘 수도 있다. GC 시점을 코드로 제어하려는 시도가 권장되지 않는 이유다.</p>

<p>더 근본적인 문제도 있다. 설령 GC가 돌아 문자열을 수거하더라도, 그걸로 값이 ‘지워지는’ 것은 아니다. <strong>GC는 “이 자리는 비었다”고 표시해 재사용할 수 있게 만들 뿐, 그 메모리가 다른 값으로 덮이기 전까지 “helloWorld” 바이트는 그대로 남아 있을 수 있다.</strong> 메모리에서 값을 확실히 지우는 길은 결국 하나, 직접 덮어쓰는 것뿐이다.</p>

## 그런데 입력은 결국 String으로 들어온다

<p>“민감 데이터는 char[]로” — 원칙은 알겠는데, 현실적인 벽이 있다. 사용자 입력을 처음부터 char[]로 받을 방법이 있을까?</p>

<p>대부분의 프레임워크는 그런 통로를 제공하지 않는다. HTTP 요청이 서버에 닿는 순간 입력값은 이미 <code>String</code>으로 파싱되어 있고, 우리는 그걸 받아 필요할 때 char[]로 변환할 뿐이다.</p>

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 150" role="img" aria-label="사용자 입력이 String으로 들어와 char 배열로 변환">
  <circle class="fill-surf2" cx="66" cy="55" r="15" style="stroke:var(--border);stroke-width:1.2"/>
  <path class="fill-surf2" d="M45 93 a21 21 0 0 1 42 0 Z" style="stroke:var(--border);stroke-width:1.2"/>
  <text class="label" x="66" y="122" font-size="11" text-anchor="middle">사용자</text>
  <line class="stroke-accent" x1="98" y1="66" x2="344" y2="66" style="stroke-width:3"/>
  <polyline class="stroke-accent" points="336,60 344,66 336,72" style="stroke-width:3"/>
  <rect class="accentbox" x="176" y="48" width="96" height="36" rx="8"/>
  <text class="label-accent" x="224" y="71" font-size="15" text-anchor="middle">String</text>
  <g>
    <rect class="surfbox" x="350" y="24" width="104" height="24" rx="5"/>
    <rect class="line" x="360" y="32" width="7" height="7" rx="1.5"/><rect class="line" x="370" y="32" width="7" height="7" rx="1.5"/><rect class="line" x="380" y="32" width="7" height="7" rx="1.5"/>
    <circle class="fill-accent" cx="420" cy="36" r="3"/><circle class="fill-accent" cx="432" cy="36" r="3"/><circle class="fill-accent" cx="444" cy="36" r="3"/>
    <rect class="surfbox" x="350" y="54" width="104" height="24" rx="5"/>
    <rect class="line" x="360" y="62" width="7" height="7" rx="1.5"/><rect class="line" x="370" y="62" width="7" height="7" rx="1.5"/><rect class="line" x="380" y="62" width="7" height="7" rx="1.5"/>
    <circle class="fill-accent" cx="420" cy="66" r="3"/><circle class="fill-accent" cx="432" cy="66" r="3"/><circle class="fill-accent" cx="444" cy="66" r="3"/>
    <rect class="surfbox" x="350" y="84" width="104" height="24" rx="5"/>
    <rect class="line" x="360" y="92" width="7" height="7" rx="1.5"/><rect class="line" x="370" y="92" width="7" height="7" rx="1.5"/><rect class="line" x="380" y="92" width="7" height="7" rx="1.5"/>
    <circle class="fill-accent" cx="420" cy="96" r="3"/><circle class="fill-accent" cx="432" cy="96" r="3"/><circle class="fill-accent" cx="444" cy="96" r="3"/>
  </g>
  <text class="label" x="402" y="122" font-size="11" text-anchor="middle">서버</text>
  <line class="stroke-accent" x1="460" y1="66" x2="496" y2="66" style="stroke-width:3"/>
  <polyline class="stroke-accent" points="488,60 496,66 488,72" style="stroke-width:3"/>
  <rect class="surfbox" x="500" y="48" width="82" height="36" rx="8"/>
  <text class="ink" x="541" y="71" font-size="15" text-anchor="middle">char[]</text>
</svg>
  <figcaption>입력 시점에 String은 이미 만들어진다. char[]로 옮겨도 그 String은 메모리에 남는다</figcaption>
</figure>

<p>즉 char[]로 바꿔 들고 있어도, 요청 초기에 만들어진 String형 민감 정보는 여전히 메모리 어딘가에 남는다. 그러니 <strong>“민감 데이터를 char[]로 관리한다”가 Heap Inspection을 완전히 없애 주지는 못한다.</strong> 서두에 말했듯, 프레임워크 자체가 이걸 막아 주도록 설계돼 있지 않기 때문이다.</p>

<p>그렇다고 무의미한 것도 아니다. 우리의 목표는 애초에 “완전 제거”가 아니라 <mark>메모리에 올라와 있는 시간의 최소화</mark>였다. String 생성 자체를 막을 수 없다면, 최대한 빨리 char[]로 옮기고 원본 String은 곧바로 참조를 끊어 주면 된다. 그러면 그 문자열이 메모리에 남는 시간이 그만큼 짧아진다 — 그게 현실적으로 우리가 할 수 있는 최선이다.</p>

## 마무리

<p>Heap Inspection은 메모리에 남은 데이터를 그대로 읽어 가는 공격이다. 이를 막는 원칙은 처음부터 끝까지 하나였다.</p>
<ul>
  <li><strong>String은 불변</strong> — 지울 방법이 없어 GC 전까지 메모리에 남는다. 패스워드를 담으면 그 잔존 시간이 곧 노출 창이 된다.</li>
  <li><strong>char[]는 가변</strong> — 다 쓴 즉시 overwrite로 그 자리에서 지울 수 있다. 그래서 민감 정보의 그릇으로 적합하다.</li>
  <li><strong>완전 제거는 어렵다</strong> — 입력은 결국 String으로 들어오므로, 목표는 제거가 아니라 <mark>노출 시간의 최소화</mark>다.</li>
</ul>

<p>실무에서 Heap Inspection이 실제로 성공할 가능성은 희박할지 모른다. 하지만 잠재적 리스크까지 걷어내는 건 결국 이런 디테일이다 — 민감 정보가 메모리에 머무는 그 짧은 순간까지 챙기는 습관이 방어를 한 겹 더 두껍게 만든다.</p>

<div class="refs">
  <p class="refs-lbl">참고 자료</p>
  <ul>
    <li><a href="https://docs.oracle.com/en/java/javase/17/docs/api/java.base/javax/crypto/spec/PBEKeySpec.html" target="_blank" rel="noopener">Java Platform SE — PBEKeySpec</a><span class="dom">docs.oracle.com <span class="ext">↗</span></span></li>
    <li><a href="https://thesecurityvault.com/heap-inspection/" target="_blank" rel="noopener">The Security Vault — Heap Inspection</a><span class="dom">thesecurityvault.com <span class="ext">↗</span></span></li>
  </ul>
</div>
