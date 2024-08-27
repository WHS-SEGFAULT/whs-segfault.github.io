---
layout: post
title: Fundamental Knowledge of V8 engine ( V8, Pipeline, Garbage collection, Ubercage )
subtitle: Exploit Knowledge
categories: V8
tags: [V8, Garbage Collection, Ubercage]
---

# [v8] 배경 지식 - V8, Pipeline, Garbage collection, Ubercage

## V8이란

우선 v8이 무엇인지에 대해 설명해보겠습니다.

전세계 웹 브라우저 시장 점유율을 가져온 표인데요, 2023년 12월을 기준으로 Chrome 브라우저가 무려 **64.7%**를 차지하는 것을 볼 수 있습니다.

이 Chrome 브라우저에서 사용하는 엔진이 지금 소개 중인 V8 엔진 입니다.

![image.png](%E1%84%90%E1%85%B5%E1%86%B7%20%E1%84%87%E1%85%B3%E1%86%AF%E1%84%85%E1%85%A9%E1%84%80%E1%85%B3%20683e02acaf074310b3f31fc8e1f9555e/image.png)

인터넷에 V8 엔진으로 검색하면, 8기통 엔진이 나오는데, Chrome이 조금 더 분발해야 검색 창에 바로 Chrome v8엔진이 나오지 않을까 싶네요.

## V8 엔진 구성 요소

이 V8 엔진 속에는 다양한 구성 요소로 이루어져 있습니다. 이해하기 편하게 두 개의 사진을 아래에 첨부하였습니다.

V8의 구성 요소는 다음과 같습니다 :

Parser, Ignition, Sparkplug, Maglev, TurboFan
