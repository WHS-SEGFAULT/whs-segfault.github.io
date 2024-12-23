---
layout: post
title: JS Object - SpiderMonkey
subtitle: Exploit Knowledge
excerpt_image: NO_EXCERPT_IMAGE
categories: Adobe
tags: [CVE-2021-39863, ArrayBuffer, DataView]
---

- Exploit은 JS Object로 임의 주소에 접근한 후 Stack Pivoting이라는 기법을 이용해 진행된다
- 이에 사용되는 JavaScript의 Object인 JS Object가 ArrayBuffer Object와 DataView Object이다
- Adobe는 SpiderMonkey라는 JavaScript Engine을 사용해 PDF에 내장된 JS code를 compile한다. 이 블로그는 SpiderMonkey Engine에서 JS Object에 관해 기술하였다.

## 1. ArrayBuffer Object

- Reference 타입의 데이터 형태로 Object가 Heap에 할당된다.
- **고정된 길이**의 연속된 메모리 공간을 할당해 사용한다.

### 1) ArrayBuffer Object의 특징

- 메모리에서 사용할 바이트 수를 명시하며 선언한다
 ```js
 var arrbuf = new ArrayBuffer(bytes)
 ```

- Array와 달리 무작위 위치의 데이터에 접근 할 수 없다.

  - 다시 말해, arrbuf[0], arrbuf[1], ... 등의 형태로 데이터에 접근할 수 없다.
  
  - ```view``` 역할을 하는 객체 (TypedArray)를 사용한다

     - TypedArray : 특정 자료형으로 값을 읽을 수 있다.
       
       - C언어의 Union과 유사한 기능을 수행한다고 볼 수 있다.
       
       - Int8Array : 8 bit의 정수로 값에 접근한다.
       
       - Uint8Array : 8 bit의 음이 아닌 정수로 값에 접근한다.
       
       - ...     
    
![image](https://github.com/user-attachments/assets/e28b4f63-9b9d-45f9-9531-5d5a198f7560)

[그림 1] 16 byte ArrayBuffer Object를 생성하는 모습 (상) 및 생성된 ArrayBuffer의 TypedArray (하)
                    
### 2) ArrayBuffer Header
- ArrayBuffer Object는 0x10 byte 크기의 header를 가지고 있다.

![image](https://github.com/user-attachments/assets/68f556a1-3268-445f-86cb-5442be44db04)

[그림 2] ArrayBuffer Object의 구조

- flags : 해당 ArrayBuffer와 관련된 flag 값이 저장된다.

- initializedLength
    - ArrayBuffer Data의 길이로 ```[ArrayBuffer Object 이름].byteLength``` 값이다.
    ```js
    length = arrbuf.byteLength
    ```
    
- capacity : 해당 ArrayBuffer Object로 DataView Object가 생성되었을 경우 DataView Object의 주소이다.

- length

## 2. DataView Object

- ArrayBuffer를 다루기 위한 API를 제공한다.
    ⇒ DataView Object로 ArrayBuffer의 이진 데이터를 읽거나 쓸 수 있다.
  ```js
  var dv = new DataView(ArrayBuffer_Object)
  ```
    
    - `set`**<U>`????`</U>**`(offset, value, little_endian)`
        
        : ArrayBuffer Data 시작 위치부터 offset 위치에 **`????`** 형식으로 value를 저장한다.
        
        - little_endian == true : little_endian으로 저장한다.
        - little_endian == false : big_endian으로 저장한다.
    
    - `get`**<U>`????`</U>**`(offset, little_endian)`

       : ArrayBuffer Data 시작 위치부터 offset 위치의 값을 **`????`** 형식으로 읽는다.
        
       - little_endian == true : little_endian으로 메모리를 읽는다.
       - little_endian == false : big_endian으로 메모리를 읽는다.
    
 ```js
 dv.setInt16(offset, value, little_endian)
 var result = dv.getInt16(offset, little_endian)
 ```

---

```js
// ArrayBuffer, DataView 예시 코드
// [1] Assign 32 bytes
var arrbuf = new ArrayBuffer(32)
// [2] Print length of arrbuf : 32
console.log(arrbuf.byteLength)

// [3] get DataView object of arrbuf
var dv = new DataView(arrbuf)
// [3] Change 16 byte value at arrbuf with offset 16 byte by little-endian
dv.setInt16(16, 255, true)

// [4] Print the 16 byte value of first half of arrbuf
console.log(dv.getInt16(0, true))
// [5] Print the 16 byte value of second half of arrbuf
console.log(dv.getInt16(16, true))
```
![image](https://github.com/user-attachments/assets/a1e1f12c-60d1-4b69-bd9a-adb8189e4951)

[그림 3] ArrayBuffer, DataView 예시 코드 실행 결과

+) DataView Object가 존재하지 않는 Property를 참조하면 DataView Object는 getProperty 함수를 호출해 부모 Object로부터 해당 Property를 검색한다.

## 3. JS Object

- SpiderMonkey에서는 모든 Object가 ObjectImpl Object를 참조한다.   
    ⇒ Object인 DataView Object 역시 ObjectImpl Object에서 사용하는 메서드를 사용할 수 있다.

![image](https://github.com/user-attachments/assets/b2cdfa84-4e8b-4d49-b8f2-6dcf228595c9)

[그림 4] SpiderMonkey의 Object 계층 구조도 (ex:DataView Object는 ArrayBufferView Object 참조)
        
- JS Object 중 배열을 요소로 갖고 있지 않은 객체(ex : `DataView`)는 `elements_` field에 `emptyElementsHeader`의 base address를 저장한다.
    - 이때, `emptyElementsHeader`는 static instance로 선언되어 EScript.api의 data 영역에 존재한다.
    ![image](https://github.com/user-attachments/assets/c16a8267-8f03-4d53-8aa7-314f9bcf0cd7)
    [그림 5] ArrayBuffer → DataView → emptyElementsHeader까지의 참조도

---

[2. DataView Object](https://whs-segfault.github.io/adobe/2024/08/22/JS-Object-SpiderMonkey.html#h-2-dataview-object)에서도 언급했듯 DataView Object가 존재하지 않는 property를 참조하면 DataView Object는 getProperty 함수를 호출해 해당 Property 검색한다.

![image](https://github.com/user-attachments/assets/24ca4733-a21b-4556-8070-7c96f9707bce)
[그림 6] getProperty 함수의 주소를 찾는 과정을 나타낸 참조도

## REFERENCE
1. [DataView Object](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/DataView/DataView)
2. [SpiderMonkey Internal](https://github.com/ricardoquesada/Spidermonkey/tree/master/js/src/vm)
3. [JSObject](https://www.sidechannel.blog/en/attacking-js-engines/)
