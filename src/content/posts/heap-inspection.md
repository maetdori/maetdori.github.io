---
title: Heap Inspection
description: 메모리 덤프로 민감정보가 털리지 않으려면 — String 대신 char[]를 써야 하는 이유를 JVM 메모리 생애주기로 파헤칩니다.
date: '2023.04.14'
pubDate: 2023-04-14
category: Java · Security
tags: [java, security, jvm]
thumb: /posts/heap-inspection/img-01.png
readingTime: 9 min read
---

Heap Inspection은 보통의 개발자들이 잘 신경 쓰지 않는 보안 취약점이다. 개선하기가 쉽지 않을 뿐 아니라, 대부분의 라이브러리 또는 프레임워크가 이를 처리할 수 있도록 잘 설계되어 있지 않기 때문이다.

## So what is Heap Inspection?

<figure>
  <img src="/posts/heap-inspection/img-01.png" alt="Heap Inspection 개념 일러스트" />
</figure>

*Heap Inspection*이란, 메모리 덤프(Memory dump)와 같은 방법을 통해 메모리에 저장된 데이터를 읽는 정보 탈취 공격을 말한다. 메모리에 직접 접근함으로써 패스워드와 같은 민감한 정보들을 빼갈 수 있게 된다.

이에 대한 예방책은 <mark>민감한 정보가 메모리에 올라와 있는 시간을 최소화</mark>하는 것이다. 필요한 경우 데이터를 암호화하고, 더 이상 필요 없어진 데이터는 메모리에서 해제해야 한다.

## Java secure coding

> *String 객체는 이용자 패스워드와 같이 보안상 민감한 정보를 저장하기에 부적절합니다. 보안상 민감한 정보는 언제나 char 배열에 저장해야 합니다.*
> <cite>— 자바 암호화 설계 가이드 (Java Cryptography Architecture guide)</cite>

이러한 맥락에서, Java 암호화 설계 가이드에서는 비즈니스 로직 상에서 민감 데이터를 다룰 때 String 대신 char array를 사용할 것을 권고하고 있다.

데이터가 저장될 자료형을 선택하는 것이, 민감한 정보가 메모리에 올라와 있는 시간을 최소화하는 것과 무슨 관계가 있길래 이런 가이드라인이 존재하는 걸까?

## Memory Lifecycle

기본적으로 프로그램은 메모리를 할당받고, 사용하고, 해제하는 흐름을 가진다. 이러한 흐름을 바탕으로 Java의 String과 char[]가 각각 어떻게 메모리에 할당되고 해제되는지 알아보자.

<figure>
  <img src="/posts/heap-inspection/img-02.png" alt="메모리 생애주기 — 할당·사용·해제" />
</figure>

## Memory Deallocation of String

<figure>
  <img src="/posts/heap-inspection/img-03.png" alt='String password = "helloWorld";' />
</figure>

쉽게 생각할 수 있는 민감 정보인 **패스워드**를 이와 같이 String 객체에 저장해 사용한다고 생각해보자.

<figure>
  <img src="/posts/heap-inspection/img-04.png" alt="String 객체의 메모리 할당" />
  <figcaption>Memory Allocate</figcaption>
</figure>

그러면 우리가 생성한 String 객체는 위와 같이 메모리에 올라오게 된다. 이때, 다 사용하고 쓸모없어진 `password` 인스턴스를 어떻게 메모리에서 해제해줄 수 있을까?

### 1. 객체를 null로 초기화

JVM의 Garbage Collector는 Stack에서 더 이상 참조하지 않게 된 Heap 영역의 데이터들을 수거하기 때문에, 우리는 우선적으로 **객체를 null로 초기화하는 방법**을 떠올릴 수 있다.

<figure>
  <img src="/posts/heap-inspection/img-05.png" alt="password = null; 이후 메모리 상태" />
  <figcaption>password = null;</figcaption>
</figure>

그러면 위와 같이 `password`는 더 이상 “helloWorld”를 참조하지 않게 된다. 이제 “helloWorld”라는 문자열은 어떤 곳에서도 사용되지 않으므로 가비지 컬렉터의 수집 대상이 되어 가비지 컬렉션이 실행될 때 성공적으로 메모리에서 해제될 수 있다.

password를 null로 초기화함으로써 더 이상 사용되지 않는 문자열 “helloWorld”가 메모리에서 정상적으로 해제되므로 아무런 문제가 없는 것 아니냐고 생각할 수도 있겠지만, 문제는 <mark>가비지 컬렉션이 실행되기 전까지는 “helloWorld”라는 패스워드 정보가 메모리에 여전히 남아있다는 점</mark>이다.

패스워드 정보가 GC 실행 전까지 메모리에 남아있게 되는 것이 문제라면, password 객체를 아예 **다른 값으로 overwrite**하면 어떨까?

### 2. 객체를 다른 값으로 overwrite

<figure>
  <img src="/posts/heap-inspection/img-06.png" alt='password = "helloPassword";' />
</figure>

결론부터 말하자면 이 방법도 유효하지 않다. String은 불변(immutable) 객체이기 때문이다. String은 한 번 생성이 되면 절대 바뀌지 않는다.

아까 전에 선언한 `password` 객체의 값을 “helloPassword”로 재할당해보자. 이때 기존에 있던 “helloWorld”의 값이 “helloPassword”로 변경된다고 생각할 수도 있겠으나, 그렇지가 않다. JVM은 기존의 “helloWorld”와 별개로 “helloPassword”라는 문자열을 새롭게 생성하고, `password`가 가리키는 힙 주소를 이 “helloPassword” 문자열이 존재하는 위치로 변경한다. 따라서 메모리 상에는 “helloWorld”와 “helloPassword” 두 개의 문자열이 동시에 존재하게 된다.

<div class="cols two">
  <figure>
    <img src="/posts/heap-inspection/img-07.png" alt="재할당 변경 전" />
    <figcaption>변경 전</figcaption>
  </figure>
  <figure>
    <img src="/posts/heap-inspection/img-08.png" alt="재할당 변경 후" />
    <figcaption>변경 후</figcaption>
  </figure>
</div>

이렇게 String을 메모리에서 해제하기 위한 두 가지 방법을 생각해봤는데,

- **객체를 null로 초기화하는 방법**은, password가 더 이상 “helloWorld”를 가리키지 않도록 할 뿐이지, “helloWorld” 문자열 자체는 여전히 메모리 상에 존재하게 된다.
- **객체를 다른 값으로 overwrite하는 방법**은, 값 자체를 바꾼 것이 아니라 참조만 “helloPassword”로 바꾼 것이기 때문에 기존의 “helloWorld” 문자열은 여전히 메모리 상에 존재하게 된다.

결국 String을 사용하면 어떤 방법을 쓰더라도 가비지 컬렉션이 실행되기 전까지는 일정 시간 메모리에 남아있게 되는 문제가 있다. 이렇게 되면 메모리 덤프를 통한 Heap Inspection에 노출될 수가 있게 되는 것이다.

## Memory Deallocation of char[]

그래서 자바 암호화 설계 가이드에서는 패스워드와 같은 민감 데이터를 코드 상에서 다룰 때는 String보다 char[]을 사용하는 것을 권장하고 있다. char[]은 *mutable* 객체로, String과 다르게 값의 변경이 가능하다.

<figure>
  <img src="/posts/heap-inspection/img-09.png" alt="char[] 배열을 overwrite하는 코드" />
</figure>

password를 “secret”이라는 char형 배열로 선언하고, 이를 마음껏 사용한 뒤에 아스키코드 0x20번, 즉 스페이스바로 배열을 덮어씌워주기만 하면, 기존의 “secret”이라는 패스워드 정보는 메모리 공간 어디에서도 찾아볼 수 없게 된다.

위와 같이 char[]는 값을 변경하면 변경된 값이 기존 메모리 공간에 덮어씌워지게 된다. 따라서 char 배열에 할당된 메모리가 더 이상 필요하지 않은 경우, 해당 배열을 `null` 또는 다른 값으로 overwrite하면 Heap Inspection을 방지할 수 있다.

## String vs char[]

여기까지의 내용을 정리하면 다음과 같다.

- String은 불변(immutable) 객체이기 때문에, 가비지 컬렉터가 이를 삭제하기 전까지 일정 시간 메모리 상에 존재한다. 따라서 String에 패스워드와 같은 민감한 정보를 저장할 경우, 해당 정보가 메모리에서 안전하게 삭제되지 않아 Heap Inspection의 대상이 될 수 있다.
- 반면, char[]은 가변(mutable) 배열이기 때문에, 데이터 사용이 완료된 후 개발자가 직접 값을 덮어씌울 수 있다. 따라서 더 이상 사용하지 않는 데이터를 null 또는 다른 값으로 overwrite하여 Heap Inspection을 방지할 수 있다.

**∴ 보안상 민감한 정보를 다룰 때는 char array를 사용하여 overwrite하는 것이 안전하다.**

## Garbage collector

그러면 이제 이렇게 생각할 수 있다.

> char[]로 저장된 패스워드 사용이 끝난 뒤 개발자가 값을 직접 덮어씌우는 방법도 있지만, String으로 패스워드를 저장해 사용하고 사용이 끝난 뒤 개발자가 `System.gc()`로 가비지 컬렉터를 직접 호출해 더 이상 사용하지 않는 String 객체를 삭제하도록 하면 되지 않나?

결론부터 말하자면, 이렇게 `System.gc()`로 가비지 컬렉터를 직접 호출해서 사용하면 안 된다. 가비지 컬렉터를 명시적으로 호출하더라도 바로 가비지 컬렉션이 실행되지 않을 수 있기 때문이다.

이를 설명하기 위해서는 GC가 설계된 방식을 알아야 한다. 가비지 컬렉션이 실행되면 JVM GC는 더 이상 사용되지 않는 객체들을 메모리 상에서 해제하게 되는데, 이때 JVM은 일시적으로 모든 스레드를 중지한다. 이를 stop the world라고 하는데, 이 때문에 가비지 컬렉션이 실행되는 동안에는 애플리케이션의 성능이 저하될 수 있다.

<figure>
  <img src="/posts/heap-inspection/img-10.png" alt="가비지 컬렉터가 사용되지 않는 객체를 수거하는 다이어그램" />
</figure>

따라서 JVM은 자체적인 알고리즘에 따라 가비지 컬렉션을 수행할 적절한 시점을 찾아 성능을 최적화할 수 있도록 설계되어 있다. 때문에 개발자가 `System.gc()`로 가비지 컬렉터를 호출하더라도, JVM의 판단에 따라 가비지 컬렉터가 즉시 실행되지 않을 수도 있는 것이다. 따라서 `System.gc()` 메서드를 이용해 가비지 컬렉터를 강제로 실행시키는 것은 권장되지 않는다.

## Minimize lifetime of sensitive-data

그러면 이제 민감 데이터를 다룰 때는 String이 아니라 char[]을 써야 한다는 것은 알겠는데, 사용자 측에서 입력한 문자열 데이터를 char[]로 바로 받을 수 있는 방법이 있던가?

대부분의 프레임워크는 입력값을 char[]로 받을 수 있는 방법을 제공하지 않는다. 일반적으로 입력값을 String으로 받고, 이후에 필요에 따라 char[]로 변환하는 방식을 취하게 된다.

<figure>
  <img src="/posts/heap-inspection/img-11.png" alt="사용자 입력이 String으로 들어와 char[]로 변환되는 흐름" />
</figure>

지금까지 보안을 위해 char[]를 써야 하니 어쩌니 했던 것도, 어쨌든 Http Request로 사용자 입력을 받게 되면 서버에 요청이 들어오는 시점에는 필연적으로 String 인스턴스가 존재하게 되고, char[]로 변환한 이후에도 String형 민감 정보는 여전히 메모리에 존재하게 된다는 것이다.

즉, 지금까지 강조한 “민감 데이터를 char[]로 관리하는 것”이 Heap Inspection 취약점을 완전히 제거해주지는 않는다. 서두에 말했듯이, **대부분의 라이브러리 또는 프레임워크가 이를 처리할 수 있도록 잘 설계되어 있지 않기 때문**이다. 하지만 다시 상기해보자. Heap Inspection 예방책으로써의 우리의 목표는 “민감한 정보가 메모리에 올라와 있는 시간을 최소화”하는 것이다.

String 생성 자체를 막을 수 없다면 최대한 빨리 char[]로 변환해서, 가비지 컬렉터가 문자열 데이터를 메모리에서 최대한 빨리 회수할 수 있도록 해야 한다. 이렇게 함으로써 “민감한 정보가 메모리에 올라와 있는 시간을 최소화”할 수 있다.

## Summary

Heap Inspection은 메모리에 저장된 데이터를 읽는 정보 탈취 공격을 말한다. 이를 예방하기 위해서는 패스워드와 같이 보안상 민감한 정보는 언제나 char 배열에 저장하고, 사용이 완료된 후에는 반드시 overwrite 해야 한다.

대부분의 애플리케이션에서는 민감한 정보들을 안전하게 다루기 위해 개발 단계부터 많은 보안요소들을 고려할 것이다. 그럼에도 불구하고, 민감한 데이터가 메모리에 남게 된다면 유능한 해커들에게는 정보 탈취의 좋은 수단이 될 수 있다. 애플리케이션에 여러 보안 모듈들이 적용되어 있어 실제로 Heap Inspection을 통한 공격 가능성이 매우 희박하다고 해도, 잠재적인 리스크들까지도 완전히 제거하기 위해서는 지금까지 설명한 내용을 반드시 고려해야 한다.

<div class="callout">
  <span class="ic">🔗</span>
  <div>
    <p><strong>Reference</strong></p>
    <p>
      <a href="https://docs.oracle.com/javase/6/docs/technotes/guides/security/crypto/CryptoSpec.html#PBEEx" target="_blank" rel="noopener">Java Cryptography Architecture Guide</a><br />
      <a href="https://thesecurityvault.com/heap-inspection/" target="_blank" rel="noopener">The Security Vault — Heap Inspection</a>
    </p>
  </div>
</div>
