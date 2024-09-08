---
layout: post
title: "[English] Encoding [UTF-16BE; ANSI]"
subtitle: Exploit Knowledge
excerpt_image: NO_EXCERPT_IMAGE
categories: Adobe
tags: [CVE-2021-39863, UTF-16BE, ANSI, English]
---

## 1. Unicode

- "Unicode" itself is not an Encoding method, but it is a code table that maps a character and its encoding character one-to-one.

- Unicode represents one character with 2 bytes. Therefore, the string encoded by Unicode ends with `"\x00\x00"`, not `"\x00"`.

- For this reason, the Null Terminator of string encoded by Unicode is 2 bytes : `"\x00\x00"`

### UTF-16BE

- UTF-16 : 16-bit Unicode Transformation Format
- BE : Big Endian

- According to Unicode, UTF-16BE is an encoding method to the form of Big Endian by one character to 16 bit (2 bytes).

- It uses **BOM**, `"%uFEFF"`, at the beginning of the string to represent it as UTF-16BE.

#### BOM, Byte Order Mark

- Originally, BOM plays the role of giving information about the string to the program.
- In UTF-16BE, BOM gives information about Endian.
    
    - BOM : `%uFEFF` : UTF-16<U>BE</U> : Big Endian
    - BOM : `%uFFFE` : UTF-16<U>LE</U> : Little Endian
    
>💡 UTF-16BE Format : `"%uFEFF .... %u0000"`
  
- Ex:) `"A" : "%uFEFF%u0041%u0000"`

## 2. ANSI

- "ANSI" itself is not an Encoding method, but it means the default local/codepage for my system.

- In ANSI, each CodePage has a code table and uses a special CodePage according to each language.

- In ANSI, an English character is represented as 1 byte. Therefore, the Null Terminator of an English string is 1 byte. : `\x00`

- Unlike UTF-16, ANSI doesn't use BOM.
    
>💡 ANSI Format : `"\x.. .. \x00"`
  
- Ex:) `"A" : "\x41\x00"`

## REFERENCE

1. [Unicode, ANSI](https://umbum.dev/328/)
2. [UTF-16](https://ko.wikipedia.org/wiki/UTF-16)
3. [BOM](https://ko.wikipedia.org/wiki/%EB%B0%94%EC%9D%B4%ED%8A%B8_%EC%88%9C%EC%84%9C_%ED%91%9C%EC%8B%9D)
