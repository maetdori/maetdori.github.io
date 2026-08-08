---
title: 게임 엔진은 모르지만 게임 개발은 하고 싶어
description: 픽셀 아트부터 시작해 게임 엔진 없이 Spring Boot + React로 사이버 가챠샵을 만든 이야기. 확률 테이블 설계를 두 번 갈아엎은 기록.
date: '2024.07'
pubDate: 2024-07-01
category: Side Project
tags: [side-project, game, react, spring]
thumb: /posts/game-dev-without-engine/img-11.png
readingTime: 7 min read
---

## 게임 개발을 해보고 싶어

사내 교육으로 인프런 강의를 신청하였는데 바로 컷 당했다.

<figure>
  <img src="/posts/game-dev-without-engine/img-02.png" alt="시무룩" />
</figure>

사실 그때의 나는 게임 개발에 관심이 있다기보다는 픽셀 아트에 관심이 있었다고 하는 게 맞을 것 같다. (그 무렵 *데이브 더 다이버*에 빠져 있었다.)

근데 뭐.. 픽셀 게임이라면 거창한 것까지는 필요 없을 것 같아서 일단 내가 할 수 있는 범위에서 해보기로 결정. 디자인 문외한이었던지라 픽셀 툴이 따로 있을 거라곤 생각 못 했는데 꽤 여러 가지가 있더라. 그 중에 나는 **Aseprite**를 사용!

<figure>
  <img src="/posts/game-dev-without-engine/img-04.png" alt="Aseprite로 작업 중인 픽셀 아트" />
</figure>

스팀에서 구매했고 사용하는 건 크게 어렵지 않아서 유튜브를 보며 금방 배울 수 있었다.

<figure>
  <img src="/posts/game-dev-without-engine/img-05.png" alt="직접 만든 픽셀 오브젝트들" />
  <figcaption>그렇게 만들어진 나의 작고 소중한 오브젝트들</figcaption>
</figure>

픽셀 찍는 게 재밌긴 했는데 솔직히 공간 디자인까지는 무리였다.. 픽셀 아트라고 해도 질감·양감 다 표현해야 하는데, 미술을 배워본 적도 없는 사람이 그렇게 색을 자유자재로 쓸 수 있을 리가 없잖냐. 사실 배워보고는 싶은데.. 이런 것은 어디서 배우는지??

아무튼 그래서 이 정도 선에서 만들 수 있는 게임이 뭐가 있을까 하다가, **사이버 가챠샵**을 차리자는 생각을 하게 됨.

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 164" role="img" aria-label="사이버 가챠샵 개념">
    <circle class="accentbox" cx="54" cy="82" r="26"/>
    <circle class="stroke-accent" cx="54" cy="82" r="14" stroke-width="2"/>
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
  <figcaption>동전을 넣으면 픽셀 오브젝트가 나오는 사이버 가챠샵</figcaption>
</figure>

사업 아이템을 정했으면 일단 절반은 끝난 것임. (이게 맞다)

## 뚝딱뚝딱 설계로 들어가보자

처음엔 노트에 슥슥 그려가며 게임을 디자인했다. 초기엔 다양한 기능을 구상했지만.. 우선은 가챠 게임이라는 메인 기능만 만들어보기로 😶

<figure>
  <img src="/posts/game-dev-without-engine/img-08.png" alt="컴포넌트 만들기" />
</figure>

가장 기본이 되는 화면에 필요한 컴포넌트들을 만들었다. 기본적인 틀과 조작에 필요한 버튼들, 그리고 GIF 형식으로 움직이는 코인도 만들어보았다.

<figure>
  <img src="/posts/game-dev-without-engine/img-09.png" alt="UI 설계" />
</figure>

다음으로는 이 컴포넌트들을 이용해서 화면 디자인 하기. 버튼 hover 시의 상호작용을 추가하고 싶어서 기존 만들었던 버튼 컴포넌트에 추가로 눌려진 버튼도 만들었다.

<figure>
  <img src="/posts/game-dev-without-engine/img-10.png" alt="가챠 뽑기 화면" />
</figure>

현실 세계의 가챠는 머신 별로 금액이 정해져 있어서 해당 금액을 넣으면 가챠를 돌려 랜덤으로 상품을 얻을 수 있는 구조이지만, 내가 차린 사이버 가챠샵은 조금 다르다. 각 아이템에는 **등급**이 있어서(1성~5성), **코인 투자 개수**에 따라 획득할 수 있는 아이템 등급의 확률 분포가 달라진다. 즉 <mark>코인을 많이 투자할수록 높은 등급의 아이템을 얻을 확률이 높아지는 것</mark> 🤑

코인 개수를 선택하고 `GO` 버튼을 누르면 ! (두구두구~)

<figure>
  <img src="/posts/game-dev-without-engine/img-11.png" alt="가챠 결과 화면" />
</figure>

아이템을 *겟또다제 —!! ✨* 하게 된다.

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 150" role="img" aria-label="개발 스택: Spring Boot + React">
    <rect class="surfbox" x="40" y="26" width="248" height="98" rx="12"/>
    <circle cx="80" cy="72" r="18" fill="#6db33f"/>
    <text class="ink" x="112" y="68" font-size="16" font-weight="700">Spring Boot</text>
    <text class="label" x="112" y="90" font-size="11">뽑기 알고리즘 · 확률 로직</text>
    <rect class="surfbox" x="312" y="26" width="248" height="98" rx="12"/>
    <circle cx="352" cy="72" r="17" fill="none" stroke="#61dafb" stroke-width="3"/><circle cx="352" cy="72" r="4" fill="#61dafb"/>
    <text class="ink" x="384" y="68" font-size="16" font-weight="700">React</text>
    <text class="label" x="384" y="90" font-size="11">가챠 UI · 애니메이션</text>
  </svg>
</figure>

게임 엔진을 사용할 필요가 없었기 때문에 **Spring Boot + React** 구조로 가벼운 애플리케이션을 만들어보았다.

<figure>
  <img src="/posts/game-dev-without-engine/img-14.png" alt="요구사항 정리" />
</figure>

앞서 말한 요구사항을 정리하면 위와 같다. 그래서 처음엔 아이템 별로 등장 확률이 정해져 있어야 한다고 생각했다.

### IDEA 1 — 아이템 별 확률 분배

<figure>
  <img src="/posts/game-dev-without-engine/img-15.png" alt="IDEA 1: 아이템 별 확률 분배" />
</figure>

그런데 이렇게 됐을 때 문제점이 있었다. 그건 바로 코인 투자 금액 별 아이템의 획득 확률을 관리해야 한다는 것이다. 이렇게 되면 관리해야 할 확률의 수는 `아이템 개수 × 재화 단위 개수`가 된다. (현재 재화 단위는 1개, 3개, 5개의 3종류)

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

뿐만 아니라 아이템이 하나 추가될 때마다 다시 확률을 분배해줘야 한다는 문제도 있다.

<figure>
  <img src="/posts/game-dev-without-engine/img-16.png" alt="아이템 추가 시 확률 재분배" />
</figure>

그래서 자연스럽게 두 번째 발상으로 옮겨갔다.

### IDEA 2 — 등급 별 확률 분배

<figure class="figmock">
  <svg class="mock" viewBox="0 0 600 96" role="img" aria-label="아이템이 아니라 등급을 뽑는다">
    <rect class="surfbox" x="36" y="26" width="222" height="44" rx="10" opacity="0.6"/>
    <text class="label" x="147" y="53" font-size="13" text-anchor="middle">아이템을 뽑는다</text>
    <line class="stroke-muted" x1="66" y1="48" x2="228" y2="48" stroke-width="2"/>
    <g transform="translate(292 48)"><line class="stroke-accent" x1="-12" y1="0" x2="16" y2="0"/><polyline class="stroke-accent" points="8,-6 16,0 8,6"/></g>
    <rect class="accentbox" x="330" y="26" width="230" height="44" rx="10"/>
    <text class="label-accent" x="445" y="53" font-size="14" text-anchor="middle" font-weight="700">등급을 뽑는다</text>
  </svg>
</figure>

즉, <mark>사용자가 뽑는 것은 “아이템”이 아니라 “등급”</mark>이고, 뽑은 등급 내에서 아이템이 랜덤으로 지급된다. 이렇게 될 경우 IDEA 1에서의 문제점은 사라진다.

우선 IDEA 1에서는 뽑기 확률을 관리하는 테이블의 Row 수가 커지는 문제가 있었다. 그러나 사용자가 뽑는 것이 아이템이 아닌 등급이라면, 뽑기 확률은 `등급 & 투자 금액`에 따라서만 관리되면 된다.

- IDEA 1: **아이템** 개수 × 재화 단위 개수
- IDEA 2: **등급** 개수 × 재화 단위 개수

등급을 초기에 1성~5성의 5단계로 정해두었고, 더 늘어난다 하더라도 무한대로 늘어날 수 있는 아이템과 달리 등급 개수가 늘어나는 데는 한계가 있을 것이므로, 관리해야 하는 row 수는 IDEA 1에 비해 현저히 적다.

또한 IDEA 1에서는 확률이 `아이템 & 투자 금액` 별로 관리되기 때문에 아이템이 새로 추가될 때마다 확률을 다시 분배해야 하는 문제가 있었다. 그러나 IDEA 2에서는 그럴 필요가 없다.

- IDEA 1: 확률 재분배 필요 (DB Update)
- IDEA 2: 확률 재분배 불필요

## 요구사항이 정리되었으면 남은 것은 구현 뿐

구조가 명확했기 때문에 구현은 매우 간단하다.

1. 사용자로부터 코인 개수를 입력받는다.
2. 1부터 100까지의 정수 중 랜덤한 하나의 숫자를 추출한다.
3. 투자한 코인 개수에 해당하는 확률 테이블에서 뽑은 숫자의 등급을 확인한다.
4. 해당 등급의 아이템들 중 하나를 랜덤하게 뽑아서 사용자에게 보여준다.

**예시**

| 재화 투자 개수 | 등급 | 확률 | 뽑기 번호 |
| :---: | :---: | :---: | :---: |
| 3 | 1성 | 20% | 1~20 |
| 3 | 2성 | 30% | **21~50** |
| 3 | 3성 | 25% | 51~75 |
| 3 | 4성 | 20% | 76~95 |
| 3 | 5성 | 5% | 96~100 |

1. 사용자로부터 코인 개수 **3**을 입력받는다.
2. 1부터 100까지의 정수 중 랜덤한 하나의 숫자를 추출한다. **→ 34**
3. 투자한 코인 개수에 해당하는 확률 테이블에서 해당 숫자의 등급을 확인한다. **→ 2성**
4. 해당 등급의 아이템들 중 하나를 랜덤하게 뽑아서 사용자에게 보여준다.

위와 같은 로직으로 백엔드 로직을 구성하고, 여기에 아까 만든 화면을 React를 이용해 붙였다.

<div class="callout">
  <span class="ic">⚛️</span>
  <div><p>프론트 구현 과정은.. <strong>생략토록 하겠습니다.</strong></p></div>
</div>

현재로서는 가챠 게임의 핵심이 되는 아주 기본적인 기능에 대해서만 구현했지만, (디자인) 여력이 되면 확장판으로 다양한 기능들도 붙여보고 싶다.

<figure>
  <img src="/posts/game-dev-without-engine/img-21.png" alt="앞으로 붙여보고 싶은 기능들" />
</figure>

<figure>
  <img src="/posts/game-dev-without-engine/img-22.png" alt="감사합니다" />
</figure>
