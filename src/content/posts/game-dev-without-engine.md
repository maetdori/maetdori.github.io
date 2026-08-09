---
title: 게임 엔진은 모르지만 게임 개발은 하고 싶어
description: 픽셀 아트부터 시작해 게임 엔진 없이 Spring Boot + React로 사이버 가챠샵을 만든 이야기. 확률 테이블 설계를 두 번 갈아엎은 기록.
date: '2024.07.02'
pubDate: 2024-07-02
category: Side Project
tags: [side-project, game, react, spring]
thumb: /posts/game-dev-without-engine/img-11.png
readingTime: 7 min read
---

## 게임 개발을 해보고 싶어

사내 교육으로 인프런 강의를 신청했는데 바로 컷 당했다.

<figure>
  <img src="/posts/game-dev-without-engine/img-01.png" alt="게임 개발 강의 신청이 반려된 메신저 대화" />
</figure>

<p style="text-align:center; margin-top:10px;">
  <img src="/posts/game-dev-without-engine/img-02-cut.png" alt="시무룩" style="display:inline-block; width:120px; height:auto; margin:0; border:0; box-shadow:none; border-radius:0;" />
</p>

사실 그때의 나는 게임 개발에 관심이 있었다기보다는 픽셀 아트에 관심이 있었다고 하는 게 맞을 것 같다. (그 무렵 ‘데이브 더 다이버’에 빠져 있었다.) 픽셀 게임이라면 거창한 것까지는 필요 없을 것 같아서, 일단 내가 할 수 있는 범위에서 해보기로 했다.

디자인 문외한이었던지라 픽셀 툴이 따로 있을 거라곤 생각도 못 했는데 꽤 여러 가지가 있더라. 그중에 나는 **Aseprite**를 골랐다. 스팀에서 구매했고, 사용법은 크게 어렵지 않아 유튜브를 보며 금방 배울 수 있었다.

<figure>
  <img src="/posts/game-dev-without-engine/img-04.png" alt="Aseprite로 작업 중인 픽셀 아트" />
  <figcaption>Aseprite</figcaption>
</figure>

<figure class="figplain">
  <div class="objflow">
    <div class="track">
    <img src="/posts/game-dev-without-engine/obj-onigiri.png" alt="주먹밥" />
    <img src="/posts/game-dev-without-engine/obj-senbei.png" alt="센베이" />
    <img src="/posts/game-dev-without-engine/obj-salmon_sushi.png" alt="연어초밥" />
    <img src="/posts/game-dev-without-engine/obj-ebi_sushi.png" alt="새우초밥" />
    <img src="/posts/game-dev-without-engine/obj-tamago_sushi.png" alt="계란초밥" />
    <img src="/posts/game-dev-without-engine/obj-sashimi.png" alt="사시미" />
    <img src="/posts/game-dev-without-engine/obj-narutomaki.png" alt="나루토마키" />
    <img src="/posts/game-dev-without-engine/obj-tempura.png" alt="튀김" />
    <img src="/posts/game-dev-without-engine/obj-yakitori.png" alt="야키토리" />
    <img src="/posts/game-dev-without-engine/obj-takoyaki.png" alt="타코야키" />
    <img src="/posts/game-dev-without-engine/obj-dango.png" alt="당고" />
    <img src="/posts/game-dev-without-engine/obj-omlet.png" alt="오므라이스" />
    <img src="/posts/game-dev-without-engine/obj-sunny_side_up.png" alt="계란후라이" />
    <img src="/posts/game-dev-without-engine/obj-shoyu.png" alt="간장" />
    <img src="/posts/game-dev-without-engine/obj-clover.png" alt="클로버" />
    <img src="/posts/game-dev-without-engine/obj-star.png" alt="별" />
    <img src="/posts/game-dev-without-engine/obj-daejang.png" alt="대장 햄토리" />
    <img src="/posts/game-dev-without-engine/obj-rolled_up_paper.png" alt="땅문서" />
    <img src="/posts/game-dev-without-engine/obj-onigiri.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-senbei.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-salmon_sushi.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-ebi_sushi.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-tamago_sushi.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-sashimi.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-narutomaki.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-tempura.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-yakitori.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-takoyaki.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-dango.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-omlet.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-sunny_side_up.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-shoyu.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-clover.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-star.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-daejang.png" aria-hidden="true" />
    <img src="/posts/game-dev-without-engine/obj-rolled_up_paper.png" aria-hidden="true" />
    </div>
  </div>
  <figcaption>그렇게 만들어진 나의 작고 소중한 오브젝트들</figcaption>
</figure>

픽셀 찍는 건 재밌었는데, 솔직히 공간 디자인까지는 무리였다. 픽셀 아트라고 해도 질감·양감을 다 표현해야 하는데, 미술을 배워본 적 없는 사람이 색을 그렇게 자유자재로 쓸 수 있을 리가 없잖냐. (이런 건 대체 어디서 배우는 걸까..?)

그래서 이 정도 선에서 만들 수 있는 게임이 뭐가 있을까 하다가, **사이버 가챠샵**을 차리기로 했다. 동전을 넣으면 내가 만든 픽셀 오브젝트가 뽑혀 나오는 가게다.

<figure class="figplain">
  <div class="mockcard">
    <svg class="mock" viewBox="-22 0 600 164" role="img" aria-label="사이버 가챠샵 개념">
      <image href="/posts/game-dev-without-engine/coin.gif" x="26" y="54" width="56" height="56" style="image-rendering:pixelated" />
      <g transform="translate(100 82)"><line class="stroke-accent" x1="0" y1="0" x2="32" y2="0"/><polyline class="stroke-accent" points="24,-6 32,0 24,6"/></g>
      <rect class="surfbox" x="152" y="28" width="150" height="108" rx="14"/>
      <rect class="fill-soft" x="164" y="40" width="126" height="50" rx="10"/>
      <circle class="fill-accent" cx="196" cy="66" r="8"/><circle class="fill-accent" cx="226" cy="60" r="8" opacity="0.5"/><circle class="fill-accent" cx="256" cy="66" r="8" opacity="0.8"/>
      <rect class="field-2" x="206" y="104" width="42" height="22" rx="5"/>
      <text class="label" x="227" y="154" font-size="12" text-anchor="middle">가챠 머신</text>
      <g transform="translate(320 82)"><line class="stroke-accent" x1="0" y1="0" x2="32" y2="0"/><polyline class="stroke-accent" points="24,-6 32,0 24,6"/></g>
      <rect class="accentbox" x="372" y="54" width="156" height="56" rx="12"/>
      <text class="label-accent" x="450" y="88" font-size="14" text-anchor="middle" font-weight="700">픽셀 아이템 🎁</text>
    </svg>
  </div>
  <figcaption>동전을 넣으면 픽셀 오브젝트가 나오는 사이버 가챠샵</figcaption>
</figure>

사업 아이템을 정했으면 일단 절반은 끝난 것이다. (이게 맞다)

## 뚝딱뚝딱 설계로 들어가보자

처음엔 노트에 슥슥 그려가며 게임을 디자인했다. 초기엔 이런저런 기능을 구상했지만, 우선은 가챠 뽑기라는 메인 기능부터 만들어보기로 했다. 😶

가장 기본이 되는 화면에 필요한 컴포넌트부터 찍었다. 기본 틀과 조작용 버튼들, 그리고 GIF로 움직이는 코인까지.

<figure>
  <img src="/posts/game-dev-without-engine/img-08.png" alt="게임 타이틀·버튼·인풋박스·아이템박스 컴포넌트" />
  <figcaption>게임 타이틀, 버튼, 입력·아이템 박스, 스핀 코인까지 하나씩</figcaption>
</figure>

이 컴포넌트들을 조합해 화면을 짰다. 버튼에 hover 상호작용을 넣고 싶어서, 기존 버튼에 더해 ‘눌린 상태’ 버튼도 따로 만들었다.

<figure>
  <img src="/posts/game-dev-without-engine/img-09.png" alt="온보딩·유저네임 입력 화면 설계" />
  <figcaption>온보딩 → 유저네임 입력 → 중복 검사까지의 화면 흐름</figcaption>
</figure>

현실의 가챠는 머신마다 금액이 정해져 있어서, 그 금액을 넣으면 랜덤으로 상품이 나오는 구조다. 그런데 내가 차린 가챠샵은 조금 다르다. 각 아이템에는 **등급**(1성~5성)이 있고, **코인 투자 개수**에 따라 얻을 수 있는 아이템 등급의 확률 분포가 달라진다. 즉 <mark>코인을 많이 투자할수록 높은 등급의 아이템이 나올 확률이 높아지는 것</mark>. 🤑

코인 개수를 고르고 `GO` 버튼을 누르면! (두구두구~)

<figure>
  <img src="/posts/game-dev-without-engine/img-10.png" alt="뽑기 화면 — 코인 개수 선택 셀렉트박스" />
  <figcaption>코인 개수를 고르고 GO — 셀렉트박스와 버튼 hover 상태까지</figcaption>
</figure>

<figure>
  <img src="/posts/game-dev-without-engine/img-11.png" alt="가챠 결과 화면 — 등급별 아이템 획득" />
  <figcaption>등급에 따라 다른 아이템을 ‘겟또다제 —!!’ ✨</figcaption>
</figure>

게임 엔진을 쓸 이유는 없었기 때문에, **Spring Boot + React** 구조로 가벼운 애플리케이션을 만들었다. 뽑기 알고리즘과 확률 로직은 백엔드가, 가챠 UI와 애니메이션은 프론트가 맡는다.

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 150" role="img" aria-label="개발 스택: Spring Boot + React">
    <rect class="surfbox" x="40" y="26" width="248" height="98" rx="12"/>
    <g transform="translate(62 54) scale(1.5)"><path d="m23.693 10.7058-4.73-8.1844c-.4094-.7106-1.4166-1.2942-2.2402-1.2942H7.2725c-.819 0-1.8308.5836-2.2402 1.2942L.307 10.7058c-.4095.7106-.4095 1.873 0 2.5837l4.7252 8.189c.4094.7107 1.4166 1.2943 2.2402 1.2943h9.455c.819 0 1.826-.5836 2.2402-1.2942l4.7252-8.189c.4095-.7107.4095-1.8732 0-2.5838zM10.9763 5.7547c0-.5365.4377-.9742.9742-.9742s.9742.4377.9742.9742v5.8217c0 .5366-.4377.9742-.9742.9742s-.9742-.4376-.9742-.9742zm.9742 12.4294c-3.6427 0-6.6077-2.965-6.6077-6.6077.0047-2.0896.993-4.0521 2.6685-5.304a.8657.8657 0 0 1 1.2142.1788.8657.8657 0 0 1-.1788 1.2143c-2.1602 1.6048-2.612 4.6592-1.0072 6.8194 1.6049 2.1603 4.6593 2.612 6.8195 1.0072 1.2378-.9177 1.9673-2.372 1.9673-3.9157a4.8972 4.8972 0 0 0-1.9861-3.925c-.386-.2824-.466-.8284-.1836-1.2143.2824-.386.8283-.466 1.2143-.1835 1.6895 1.2471 2.6826 3.2238 2.6873 5.3228 0 3.6474-2.965 6.6077-6.6077 6.6077z" fill="#6DB33F"/></g>
    <text class="ink" x="112" y="68" font-size="16" font-weight="700">Spring Boot</text>
    <text class="label" x="112" y="90" font-size="11">뽑기 알고리즘 · 확률 로직</text>
    <rect class="surfbox" x="312" y="26" width="248" height="98" rx="12"/>
    <g transform="translate(334 54) scale(1.5)"><path d="M14.23 12.004a2.236 2.236 0 0 1-2.235 2.236 2.236 2.236 0 0 1-2.236-2.236 2.236 2.236 0 0 1 2.235-2.236 2.236 2.236 0 0 1 2.236 2.236zm2.648-10.69c-1.346 0-3.107.96-4.888 2.622-1.78-1.653-3.542-2.602-4.887-2.602-.41 0-.783.093-1.106.278-1.375.793-1.683 3.264-.973 6.365C1.98 8.917 0 10.42 0 12.004c0 1.59 1.99 3.097 5.043 4.03-.704 3.113-.39 5.588.988 6.38.32.187.69.275 1.102.275 1.345 0 3.107-.96 4.888-2.624 1.78 1.654 3.542 2.603 4.887 2.603.41 0 .783-.09 1.106-.275 1.374-.792 1.683-3.263.973-6.365C22.02 15.096 24 13.59 24 12.004c0-1.59-1.99-3.097-5.043-4.032.704-3.11.39-5.587-.988-6.38-.318-.184-.688-.277-1.092-.278zm-.005 1.09v.006c.225 0 .406.044.558.127.666.382.955 1.835.73 3.704-.054.46-.142.945-.25 1.44-.96-.236-2.006-.417-3.107-.534-.66-.905-1.345-1.727-2.035-2.447 1.592-1.48 3.087-2.292 4.105-2.295zm-9.77.02c1.012 0 2.514.808 4.11 2.28-.686.72-1.37 1.537-2.02 2.442-1.107.117-2.154.298-3.113.538-.112-.49-.195-.964-.254-1.42-.23-1.868.054-3.32.714-3.707.19-.09.4-.127.563-.132zm4.882 3.05c.455.468.91.992 1.36 1.564-.44-.02-.89-.034-1.345-.034-.46 0-.915.01-1.36.034.44-.572.895-1.096 1.345-1.565zM12 8.1c.74 0 1.477.034 2.202.093.406.582.802 1.203 1.183 1.86.372.64.71 1.29 1.018 1.946-.308.655-.646 1.31-1.013 1.95-.38.66-.773 1.288-1.18 1.87-.728.063-1.466.098-2.21.098-.74 0-1.477-.035-2.202-.093-.406-.582-.802-1.204-1.183-1.86-.372-.64-.71-1.29-1.018-1.946.303-.657.646-1.313 1.013-1.954.38-.66.773-1.286 1.18-1.868.728-.064 1.466-.098 2.21-.098zm-3.635.254c-.24.377-.48.763-.704 1.16-.225.39-.435.782-.635 1.174-.265-.656-.49-1.31-.676-1.947.64-.15 1.315-.283 2.015-.386zm7.26 0c.695.103 1.365.23 2.006.387-.18.632-.405 1.282-.66 1.933-.2-.39-.41-.783-.64-1.174-.225-.392-.465-.774-.705-1.146zm3.063.675c.484.15.944.317 1.375.498 1.732.74 2.852 1.708 2.852 2.476-.005.768-1.125 1.74-2.857 2.475-.42.18-.88.342-1.355.493-.28-.958-.646-1.956-1.1-2.98.45-1.017.81-2.01 1.085-2.964zm-13.395.004c.278.96.645 1.957 1.1 2.98-.45 1.017-.812 2.01-1.086 2.964-.484-.15-.944-.318-1.37-.5-1.732-.737-2.852-1.706-2.852-2.474 0-.768 1.12-1.742 2.852-2.476.42-.18.88-.342 1.356-.494zm11.678 4.28c.265.657.49 1.312.676 1.948-.64.157-1.316.29-2.016.39.24-.375.48-.762.705-1.158.225-.39.435-.788.636-1.18zm-9.945.02c.2.392.41.783.64 1.175.23.39.465.772.705 1.143-.695-.102-1.365-.23-2.006-.386.18-.63.406-1.282.66-1.933zM17.92 16.32c.112.493.2.968.254 1.423.23 1.868-.054 3.32-.714 3.708-.147.09-.338.128-.563.128-1.012 0-2.514-.807-4.11-2.28.686-.72 1.37-1.536 2.02-2.44 1.107-.118 2.154-.3 3.113-.54zm-11.83.01c.96.234 2.006.415 3.107.532.66.905 1.345 1.727 2.035 2.446-1.595 1.483-3.092 2.295-4.11 2.295-.22-.005-.406-.05-.553-.132-.666-.38-.955-1.834-.73-3.703.054-.46.142-.944.25-1.438zm4.56.64c.44.02.89.034 1.345.034.46 0 .915-.01 1.36-.034-.44.572-.895 1.095-1.345 1.565-.455-.47-.91-.993-1.36-1.565z" fill="#61DAFB"/></g>
    <text class="ink" x="384" y="68" font-size="16" font-weight="700">React</text>
    <text class="label" x="384" y="90" font-size="11">가챠 UI · 애니메이션</text>
  </svg>
</figure>

여기까지의 요구사항을 정리하면 규칙은 두 가지다.

<ul>
  <li><strong>등급제</strong> — 모든 아이템은 1성부터 5성까지의 등급을 가진다.</li>
  <li><strong>코인 투자량 = 확률</strong> — 코인은 1·3·5개 단위로 넣을 수 있고, 많이 넣을수록 높은 등급이 나올 확률이 커진다.</li>
</ul>

요구사항은 정해졌는데, 정작 오래 붙잡고 있었던 건 따로 있었다. **‘이 확률을 대체 어떻게 테이블로 관리하지?’** 하는 문제였다.

## 확률 테이블을 두 번 갈아엎다

### IDEA 1 — 아이템별 확률 분배

처음엔 단순하게 생각했다. 아이템마다 등장 확률을 직접 정해두면 되지 않을까?

그런데 여기엔 두 가지 문제가 있었다.

<p><strong>1. 관리해야 할 확률이 너무 많다</strong></p>

코인 투자 금액마다 아이템별 확률을 따로 둬야 한다. 그러면 관리할 확률의 개수는 `아이템 개수 × 재화 단위 개수`가 된다. (재화 단위는 1개·3개·5개, 세 종류다.)

<div class="table-wrap">

| 재화 투자 개수 | 아이템 | 확률 |
| :---: | :--- | :---: |
| 1 | 김과자 | 50% |
| 1 | 새우초밥 | 40% |
| 1 | 땅문서 | 10% |
| 3 | 김과자 | 40% |
| 3 | 새우초밥 | 35% |
| 3 | 땅문서 | 25% |
| 5 | 김과자 | 25% |
| 5 | 새우초밥 | 35% |
| 5 | 땅문서 | 40% |

</div>

아이템이 3개뿐인데도 벌써 9줄이다. 아이템이 늘어날수록 이 표는 걷잡을 수 없이 길어진다.

<p><strong>2. 아이템이 추가되면 확률을 전부 다시 나눠야 한다</strong></p>

더 성가신 문제는 이쪽이다. 전체 확률의 합은 100%로 고정돼 있으니, 아이템이 하나 추가되는 순간 기존 아이템들의 확률까지 전부 다시 계산해야 한다.

<figure class="figplain">
  <svg class="mock" viewBox="0 0 600 316" role="img" aria-label="아이템 추가 시 확률 재분배">
    <defs>
      <clipPath id="rd-top"><rect x="80" y="80" width="440" height="34" rx="8"/></clipPath>
      <clipPath id="rd-bot"><rect x="80" y="256" width="440" height="34" rx="8"/></clipPath>
    </defs>
    <!-- 기존: 아이템 3종, 고정 100% 바를 40/35/25로 분할 -->
    <text class="label" x="80" y="20" font-size="12">기존 · 아이템 3종</text>
    <image href="/posts/game-dev-without-engine/item-senbei.png"  x="153" y="40" width="30" height="30"/>
    <image href="/posts/game-dev-without-engine/item-ebi.png"     x="318" y="40" width="30" height="30"/>
    <image href="/posts/game-dev-without-engine/item-scroll.png"  x="450" y="40" width="30" height="30"/>
    <g clip-path="url(#rd-top)">
      <rect class="fill-soft" x="80" y="80" width="440" height="34"/>
      <rect x="256" y="80" width="2" height="34" fill="var(--border)"/>
      <rect x="410" y="80" width="2" height="34" fill="var(--border)"/>
    </g>
    <rect x="80" y="80" width="440" height="34" rx="8" fill="none" stroke="var(--border)" stroke-width="1.25"/>
    <text class="ink" x="168" y="102" font-size="12" text-anchor="middle" font-weight="700">40%</text>
    <text class="ink" x="333" y="102" font-size="12" text-anchor="middle" font-weight="700">35%</text>
    <text class="ink" x="465" y="102" font-size="12" text-anchor="middle" font-weight="700">25%</text>
    <!-- 전이: 신규 아이템 추가 -->
    <g transform="translate(300 159)">
      <line class="stroke-accent" x1="0" y1="-15" x2="0" y2="15"/>
      <polyline class="stroke-accent" points="-6,7 0,15 6,7"/>
    </g>
    <image href="/posts/game-dev-without-engine/item-hamtori.png" x="318" y="149" width="20" height="20"/>
    <text class="label-accent" x="344" y="164" font-size="12" font-weight="700">신규 아이템 추가</text>
    <!-- 추가 후: 아이템 4종, 같은 100% 바를 35/30/20/15로 재분할 (NEW=orange) -->
    <text class="label" x="80" y="204" font-size="12">추가 후 · 아이템 4종</text>
    <image href="/posts/game-dev-without-engine/item-senbei.png"  x="142" y="224" width="30" height="30"/>
    <image href="/posts/game-dev-without-engine/item-ebi.png"     x="285" y="224" width="30" height="30"/>
    <image href="/posts/game-dev-without-engine/item-scroll.png"  x="395" y="224" width="30" height="30"/>
    <image href="/posts/game-dev-without-engine/item-hamtori.png" x="472" y="224" width="30" height="30"/>
    <g clip-path="url(#rd-bot)">
      <rect class="fill-soft"   x="80"  y="256" width="440" height="34"/>
      <rect class="fill-accent" x="454" y="256" width="66"  height="34"/>
      <rect x="234" y="256" width="2" height="34" fill="var(--border)"/>
      <rect x="366" y="256" width="2" height="34" fill="var(--border)"/>
    </g>
    <rect x="80" y="256" width="440" height="34" rx="8" fill="none" stroke="var(--border)" stroke-width="1.25"/>
    <text class="ink" x="157" y="278" font-size="12" text-anchor="middle" font-weight="700">35%</text>
    <text class="ink" x="300" y="278" font-size="12" text-anchor="middle" font-weight="700">30%</text>
    <text class="ink" x="410" y="278" font-size="12" text-anchor="middle" font-weight="700">20%</text>
    <text x="487" y="278" font-size="12" text-anchor="middle" font-weight="700" fill="#fff">15%</text>
    <text class="label" x="157" y="310" font-size="10.5" text-anchor="middle">40 → <tspan class="label-accent" font-weight="700">35</tspan></text>
    <text class="label" x="300" y="310" font-size="10.5" text-anchor="middle">35 → <tspan class="label-accent" font-weight="700">30</tspan></text>
    <text class="label" x="410" y="310" font-size="10.5" text-anchor="middle">25 → <tspan class="label-accent" font-weight="700">20</tspan></text>
    <text class="label-accent" x="487" y="310" font-size="10.5" text-anchor="middle" font-weight="700">NEW</text>
  </svg>
  <figcaption>아이템 하나가 늘면 기존 확률까지 전부 다시 나눠야 한다 — 관리 테이블 전체 갱신</figcaption>
</figure>

아이템 하나 추가하자고 매번 확률 테이블 전체를 손봐야 한다니, 아무래도 방식이 잘못됐다. 그래서 자연스럽게 두 번째 발상으로 넘어갔다.

### IDEA 2 — 등급별 확률 분배

발상을 하나만 뒤집었다. <mark>사용자가 뽑는 것은 ‘아이템’이 아니라 ‘등급’</mark>이고, 뽑힌 등급 안에서 아이템이 랜덤으로 지급된다. 이렇게 하니 IDEA 1의 두 문제가 모두 사라진다.

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 96" role="img" aria-label="아이템이 아니라 등급을 뽑는다">
    <rect x="36" y="26" width="222" height="44" rx="10" fill="var(--surface)" stroke="var(--border)" stroke-width="1.25" stroke-dasharray="5 4"/>
    <text x="147" y="52" font-size="13" text-anchor="middle" fill="var(--faint)" font-family="var(--font)">아이템을 뽑는다</text>
    <line x1="100" y1="47" x2="194" y2="47" stroke="var(--faint)" stroke-width="1.4" stroke-linecap="round"/>
    <g transform="translate(292 48)"><line class="stroke-accent" x1="-12" y1="0" x2="16" y2="0"/><polyline class="stroke-accent" points="8,-6 16,0 8,6"/></g>
    <rect class="accentbox" x="330" y="26" width="230" height="44" rx="10"/>
    <text class="label-accent" x="445" y="53" font-size="14" text-anchor="middle" font-weight="700">등급을 뽑는다</text>
  </svg>
</figure>

<p><strong>1. 관리할 확률이 확 줄어든다</strong></p>

확률이 `아이템 & 투자 금액`이 아니라 `등급 & 투자 금액`으로만 관리되면 된다.

- IDEA 1: **아이템** 개수 × 재화 단위 개수
- IDEA 2: **등급** 개수 × 재화 단위 개수

등급은 1성~5성 5단계로 정해뒀고, 늘어난다 해도 무한히 늘어나는 아이템과 달리 한계가 있다. 그러니 관리할 행 수는 IDEA 1과 비교가 안 되게 적다.

<p><strong>2. 아이템을 추가해도 재분배가 필요 없다</strong></p>

확률은 등급에만 걸려 있으니, 아이템을 새로 추가해도 해당 등급 풀에 하나 더 넣기만 하면 된다. 확률 테이블은 건드릴 일이 없다.

- IDEA 1: 확률 재분배 필요 (DB Update)
- IDEA 2: 확률 재분배 불필요

## 요구사항이 정리되었으면 남은 것은 구현 뿐

구조가 명확해지니 구현은 간단하다.

1. 사용자로부터 코인 개수를 입력받는다.
2. 1부터 100까지의 정수 중 랜덤한 하나를 뽑는다.
3. 투자한 코인 개수에 해당하는 확률 테이블에서, 뽑은 숫자가 어느 등급 구간에 드는지 확인한다.
4. 그 등급의 아이템들 중 하나를 랜덤하게 골라 사용자에게 보여준다.

말로 풀면 이렇고, 코인 3개를 넣은 경우로 예를 들어보자.

<div class="table-wrap">

| 재화 투자 개수 | 등급 | 확률 | 뽑기 번호 |
| :---: | :---: | :---: | :---: |
| 3 | 1성 | 20% | 1~20 |
| 3 | 2성 | 30% | **21~50** |
| 3 | 3성 | 25% | 51~75 |
| 3 | 4성 | 20% | 76~95 |
| 3 | 5성 | 5% | 96~100 |

</div>

1. 코인 개수 **3**을 입력받는다.
2. 1~100 중 랜덤한 숫자를 뽑는다. **→ 34**
3. 코인 3개 테이블에서 34가 드는 구간을 확인한다. **→ 2성 (21~50)**
4. 2성 아이템들 중 하나를 랜덤하게 골라 보여준다.

이 로직으로 백엔드를 구성하고, 앞서 만든 화면을 React로 붙였다.

<div class="callout">
  <span class="ic">⚛️</span>
  <div>
    <p>프론트 구현 과정은.. <strong>생략토록 하겠습니다.</strong></p>
    <p style="margin-top:14px; margin-bottom:0;">
      <img src="/posts/game-dev-without-engine/omit.png" alt="더 이상의 자세한 설명은 생략한다" style="display:block; width:150px; height:auto; margin:0;" />
    </p>
  </div>
</div>

## 마무리

지금은 가챠 게임의 핵심이 되는 아주 기본적인 뽑기 기능만 구현했다. (디자인) 여력이 되면 확장판으로 이런 기능들도 붙여보고 싶다.

- 🙋 <strong>사용자 정보 저장</strong> — 계정별로 모은 가챠와 코인을 기록한다.
- 🎰 <strong>가챠 머신 세분화</strong> — 주제별 머신을 둔다. 음식 가챠, 호빵맨 가챠, 귀멸의 칼날 가챠…
- 👛 <strong>코인 수집 경로</strong> — 출석 체크·광고 시청으로 코인을 얻고, 과금 유도 구간도 만든다.
- 📜 <strong>가챠 도감</strong> — 수집한 가챠와 아직 못 얻은 가챠 목록을 확인한다.

돌아보면 이 프로젝트에서 가장 게임다웠던 순간은 화려한 렌더링도, 물리 엔진도 아니었다. <mark>확률 테이블을 어떻게 설계하느냐</mark>가 곧 게임의 규칙이자 재미였다. 엔진 없이 웹 스택만으로도 게임을 만들 수 있었던 건, 결국 만들려던 게임의 핵심이 화면이 아니라 그 규칙에 있었기 때문이다.

픽셀 하나 찍는 것부터 확률 로직을 두 번 갈아엎는 것까지, 문외한이 맨손으로 시작해도 게임 비슷한 게 나오긴 하더라. 재밌었으니 그걸로 됐다.

<p style="text-align:center; margin-top:34px;">
  <img src="/posts/game-dev-without-engine/thanks.png" alt="감사합니다" style="display:inline-block; width:320px; max-width:82%; height:auto; margin:0; image-rendering:pixelated; border:0; box-shadow:none; border-radius:0;" />
</p>
