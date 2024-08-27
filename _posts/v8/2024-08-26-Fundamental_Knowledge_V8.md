---
layout: post
title: Fundamental Knowledge of V8 engine ( V8, Pipeline, Garbage collection, Ubercage )
subtitle: Exploit Knowledge
categories: V8
tags: [V8, Garbage Collection, Ubercage]
---

# V8이란

우선 v8이 무엇인지에 대해 설명해보겠습니다.

전세계 웹 브라우저 시장 점유율을 가져온 표인데요, 2023년 12월을 기준으로 Chrome 브라우저가 무려 **64.7%**를 차지하는 것을 볼 수 있습니다.

이 Chrome 브라우저에서 사용하는 엔진이 지금 소개 중인 V8 엔진 입니다.

<center> <img src="https://github.com/user-attachments/assets/0a823c86-6e55-41b4-a956-28a7ab7cd5b5" /> </center>
<center> [출처 : [Browser Market Share Worldwide](https://gs.statcounter.com/browser-market-share#monthly-202312-202312-bar)] </center>

## V8 엔진 구성 요소

이 V8 엔진 속에는 다양한 구성 요소로 이루어져 있습니다. 이해하기 편하게 두 개의 사진을 아래에 첨부하였습니다.

V8의 구성 요소는 다음과 같습니다 :

Parser, Ignition, Sparkplug, Maglev, TurboFan

<center> <img src="https://github.com/user-attachments/assets/88795d10-86c5-45f2-9bc8-0ad99580b759" /> </center>
<center> [출처: [V8_pipeline](https://medium.com/dailyjs/understanding-v8s-bytecode-317d46c94775)] </center>

<center> <img src="https://github.com/user-attachments/assets/6aaf0bad-08c3-4520-966f-7f8f8f6ed823" /> </center>
<center> [출처 : [Pipeline with Maglev](https://docs.google.com/document/d/13CwgSL4yawxuYg3iNlM-4ZPCB8RgJya6b8H_E2F-Aek/edit#heading=h.dmhxljs5hbh)] </center>

각각의 구성 요소를 간단하게 설명하겠습니다.

- Parser : JavaScript 소스 코드를 AST 구조로 변환한다.

아래 사이트를 통해 간단하게 볼 수 있다.

[https://astexplorer.net/](https://astexplorer.net/)

<center> <img src="https://github.com/user-attachments/assets/1544592a-d3c2-45ff-ab6d-97d689cfa2a6" /> </center>

<center> <img src="https://github.com/user-attachments/assets/d4214fee-b9f1-4543-a9b5-bec505bf1d1d" /> </center>

- Ignition : V8의 인터프리터로, 파싱된 AST 구조를 bytecode로 변환한다. 코드를 실행할 때 머신 코드로 컴파일하지 않고, 바로 bytecode로 컴파일 및 생성, 그리고 실행하기 때문에, 굉장히 빠르다는 장점이 있다. 그리고 코드를 실행하는 것 뿐만 아닌, 이후 최적화에 사용할 프로파일링 정보를 수집하는 역할을 합니다.

이 사진은 bytecode의 일부분의 모습이다.

<center> <img src="https://github.com/user-attachments/assets/e5b646c6-259a-40cd-bcc5-0e83f93f5841" /> </center>

Ignition에서 수집한 프로파일링 정보를 통해 어느 수준의 최적화를 할지, 이로 성능 속도와 실행 속도를 향상 시키기 위해 JIT 컴파일러를 사용하는데, 여기에는 세가지 종류의 컴파일러가 있습니다.

- Sparkplug : 매우 빠르고 간단한 최적화를 수행하는 JIT 컴파일러이다. 최적화를 하지 않으며, 단순히 bytecode를 native code로 빠르게 변환한다.
- Maglev : 중간 단계 수준의 최적화를 수행하는 JIT 컴파일러이다. 이 최적화를 위해 소스 코드를 정적 분석을 한다. Maglev가 최적화한 모습을 확인하기 위해 Maglev-IR-Graph라는 것을 통해 분석한다.
- TurboFan : V8의 가장 강력한 최적화 JIT 컴파일러로, 동적 분석을 한 후 최적화하여 컴파일하는 특징을 갖고 있다.

이 세가지 컴파일러가 생긴 이유는, 처음에는 Ignition과 TurboFan만 있었는데, 이 둘 사이의 성능 속도와 실행 속도의 간극이 생겨서 이를 보완하기 위해 Sparkplug가 출시되었고, 또 이 Sparkplug와 TurboFan 사이의 속도 차이가 생겨 Maglev가 출시되었습니다.

어느 컴파일러가 최적화를 진행할지는 Ignition이 수집한 프로파일링 정보를 통해 결정이 되는데, 일반적으로 호출 횟수에 따라 정해지기 때문에, 횟수가 증가하면서 **“hot”** 코드로 인식되면 최적화를 진행한다 라고 간단하게 알고 있으면 좋습니다.

# Garbage Collection of V8

제목에 의도적으로 V8의 Garbage Collection이라고 적었는데, 그 이유는 V8의 Garbage Collection과 Java의 Garbage Collection이 다르기 때문입니다.

이것을 이루는 중심 내용은 거의 같으니, 이미 알고 있다면 이해하는데 도움이 될 것 입니다.

이 Garbage Collection은 메모리 중 heap을 관리하는 기능으로, 사용되지 않는 객체를 식별하고 그들이 차지하고 있는 메모리를 회수하는 역할을 합니다.

<center> <img src="https://github.com/user-attachments/assets/0d579d8a-59ff-4a3c-a161-1508caa6db58" /> </center>

기본적으로 V8의 heap은 다음과 같이 영역이 나뉘게 되는데, 우리가 메모리에 할당하는 영역은 Young space와 Old space가 있습니다. 이 두 영역은 서로 다른 방법으로 메모리를 관리하게 됩니다. 

## Young space

이 영역은 새로 생성된 객체들이 위치하는 영역이다. Scavenging이라는 방식을 통해 메모리를 관리하게 되는데, 이 Scavenging은 Minor GC라고도 불린다.

Young space를 반으로 나누어서 이 영역 (semi-space) 을 교대로 사용하게 된다.

- 새로 생성된 객체는 하나의 semi-space에 할당된다.
- 이 semi-space가 가득 차면 scavenger가 실행된다.
- 이때 참조가 유지된 객체 만을 다른 semi-space로 옮기고, 나머지 메모리는 정리한다.
- 두 번의 scavenger가 실행될 동안 살아남은 객체는 old space로 옮겨진다.

참고! Scavenging을 하면서 다른 semi-space로 옮기면서 정리하게 되어 이미 이것을 하면서 compaction과 같은 효과를 갖게 된다. 따라서 별도의 compaction 단계는 없지만, Young space도 객체들을 메모리의 앞으로 이동시키게 된다.

## Old space

Old space는 포인터 영역과 데이터 영역으로 나뉜다. 포인터 영역에는 다른 객체로의 포인터를 가진 객체들을 저장한다. 데이터 영역에는 오직 데이터만 저장한다. ( Strings, boxed numbers and arrays of unboxed doubles )

그리고 이 영역은 Young space에서 살아남아 장기간 유지되는 객체들이 존재하는 영역이다. 이 곳은 Mark-Sweep와 Mark-Compact 방식으로 관리된다. 이것은 Major GC라고도 불린다.

- Mark-Sweep 방식은 사용되는 객체를 마크하고, 마크되지 않은 (즉, 더 이상 사용되지 않는) 객체들을 정리한다. 이때 메모리 단편화가 발생할 수 있다.
- Mark-Compact 방식은 위에서 단편화가 생기는 것을 보완하기 위해 고안 되었으며, 살아남은 객체들을 메모리의 앞으로 이동시킨다.

더 자세하게 공부해보고 싶다면 아래 링크를 참고하면 좋습니다. 굉장히 자세한 설명이 있습니다.

[https://deepu.tech/memory-management-in-v8/](https://deepu.tech/memory-management-in-v8/)

# V8 Sandbox ( a.k.a. Ubercage )



# Preferences

[Data and Object in V8](https://www.dashlane.com/blog/how-is-data-stored-in-v8-js-engine-memory)

[Abstract of Maglev](https://research.google/pubs/maglev-a-fast-and-reliable-software-network-load-balancer/)

[Explanation about Garbage collection in korean](https://medium.com/hcleedev/web-javascript%EC%9D%98-garbage-collection-v8-%EC%97%94%EC%A7%84-9409c5be917c)







