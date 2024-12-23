---
layout: post
title: Encoding [UTF-16BE; ANSI]
subtitle: Exploit Knowledge
excerpt_image: NO_EXCERPT_IMAGE
categories: Adobe
tags: [CVE-2021-39863, UTF-16BE, ANSI]
---

## 1. Unicode

Unicode 자체는 encoding 방식이 아니라 문자와 순서를 1:1로 매칭한 하나의 코드 표를 나타낸다.

Unicode는 문자 1개를 2 byte를 표현한다. 그래서 Unicode에 의해 encoding된 문자열은 `"\x00"`가 아니라 `"\x00\x00"`으로 끝난다.

이러한 이유로 Unicode의 Null 종단 문자는 2 byte이다. : `"\x00\x00"`

### UTF-16BE

- UTF-16 : 16-bit Unicode Transformation Format
- BE : Big Endian

- <U>Unicode</U>에 따라, <U>Big Endian</U> 형태로
<U>한 문자를 16 bit (2 byte)로 표현</U>하는 encoding 방식이다.

- UTF-16BE임을 나타내기 위해 문자열 제일 앞에 **BOM**, `"%uFEFF"`을 붙인다.

#### BOM, Byte Order Mark

  - 원래 BOM은 프로그램에게 텍스트에 대한 정보를 주는 역할을 한다.
  - UTF-16에서 BOM은 Endian에 관한 정보를 준다.
    
    - BOM : `%uFEFF` : UTF-16<U>BE</U> : Big Endian
    - BOM : `%uFFFE` : UTF-16<U>LE</U> : Little Endian
    
>💡 UTF-16BE Format : `"%uFEFF .... %u0000"`
  
- Ex:) `"A" : "%uFEFF%u0041%u0000"`

## 2. ANSI

- 특정 인코딩 방식 한 가지를 가리키는 말이 아니라 *the default local/codepage for my system*을 의미한다.

- 각 CodePage 마다 코드표가 따로 존재해 각 언어에 따른 별도의 CodePage를 사용한다.
(ex: EUC-KR, CP949 : 한글에 대응되는 CodePage)

- 한글은 2 byte로 한 문자를 표현하지만 영어는 1 byte로 한 문자를 표현한다.

  - 영어의 경우 Null 종단 문자 역시 1 byte이다. : `\x00`

- UTF-16과 달리 BOM을 사용하지 않는다.
    
>💡 ANSI Format : `"\x.. .. \x00"`
  
- Ex:) `"A" : "\x41\x00"`

## REFERENCE

1. [Unicode, ANSI](https://umbum.dev/328/)
2. [UTF-16](https://ko.wikipedia.org/wiki/UTF-16)
3. [BOM](https://ko.wikipedia.org/wiki/%EB%B0%94%EC%9D%B4%ED%8A%B8_%EC%88%9C%EC%84%9C_%ED%91%9C%EC%8B%9D)
