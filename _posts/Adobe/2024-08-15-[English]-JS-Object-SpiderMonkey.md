---
layout: post
title: "[Eng] JS Object - SpiderMonkey"
subtitle: Exploit Knowledge
excerpt_image: NO_EXCERPT_IMAGE
categories: Adobe
tags: [CVE-2021-39863, ArrayBuffer, DataView, English]
---

- We exploit CVE-2021-39863 by using Stack Pivoting after accessing the arbitrary address by JS Object.
- The ArrayBuffer Object and DataView Object which are Objects of JavaScript are used to access the arbitrary address.
- Adobe compiles JS code embedded in PDF using JavaScript Engine, SpiderMonkey. This blog tells you about JS Object (ArrayBuffer Object, DataView Object) in SpiderMonkey Engine.

## 1. ArrayBuffer Object

- ArrayBuffer Object is the data of reference type, the Object is allocated to the Heap memory.
- This Object allocates and uses the continuous memory space of **fixed length**.

### 1) Characteristics of ArrayBuffer Object

- Declare the number of the bytes to use in the Heap memory.
 ```js
 var arrbuf = new ArrayBuffer(bytes)
 ```
- Unlike Array, it can not access the data by index.
  
  - In other words, we can't access the data in the form of `arrbuf[0]`, `arrbuf[1]`, etc.

  - ArrayBuffer uses the object, TypedArray which plays the role of `view` to access the data stored in the ArrayBuffer object.

    - With TypedArray, we can read data stored in ArrayBuffer in a specific data type.

      - TypedArray performs similar work with Union in C.

      - For example, we can access the data stored in ArrayBuffer with an 8-bit integer with Int8Array. Also, we can access the data by an 8-bit unsigned integer with Uint8Array.
    
![image](https://github.com/user-attachments/assets/e28b4f63-9b9d-45f9-9531-5d5a198f7560)

[Fig 1] Generating 16 bytes ArrayBuffer (Up) and TypedArray of it (Bottom)
                    
### 2) ArrayBuffer Header
- ArrayBuffer Object has a 0x10 byte size header.

![image](https://github.com/user-attachments/assets/68f556a1-3268-445f-86cb-5442be44db04)

[Fig 2] The structure of ArrayBuffer Object with its header (blue rectangles)

- flags : Stored flags related to the current ArrayBuffer Object

- initializedLength
    - The length of ArrayBuffer Data. This value is the same as that of `[ArrayBuffer Object name].byteLength`.
    ```js
    length = arrbuf.byteLength
    ```
    
- capacity : The address of DataView Object if DataView Object of current ArrayBuffer Object is generated.

- length

## 2. DataView Object

- DataView Object provides the API to manage the ArrayBuffer Object.
    ⇒ We can read and write binary data in ArrayBuffer by an index using DataView Object.
  ```js
  var dv = new DataView(ArrayBuffer_Object)
  ```
    
    - `set`**<U>`????`</U>**`(offset, value, little_endian)`
        
        : Write value by `????` type at the offset position from the start of ArrayBuffer Data.
        
        - little_endian == true : write as little_endian
        - little_endian == false : write as big_endian
    
    - `get`**<U>`????`</U>**`(offset, little_endian)`

       : Read value by `????` type at the offset position from the start of ArrayBuffer Data.
        
       - little_endian == true : read as little_endian
       - little_endian == false : read as big_endian
    
 ```js
 dv.setInt16(offset, value, little_endian)
 var result = dv.getInt16(offset, little_endian)
 ```

---

```js
// ArrayBuffer, DataView example code
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

[Fig 3] The result of executing the ArrayBuffer, DataView example code

+) If DataView Object references the non-existent Property then DataView Object calls the `getProperty` function to find that Property to parent Objects.

## 3. JS Object

- All Objects in SpiderMonkey are implemented from ObjectImpl Object.
    ⇒ Therefore, DataView Object which is the Object can use all methods whose ObjectImpl Object uses.

![image](https://github.com/user-attachments/assets/b2cdfa84-4e8b-4d49-b8f2-6dcf228595c9)

[Fig 4] The Hierarchical diagram of Object in SpiderMonkey (ex: DataView Object is implemented from ArrayBuffer Object)

- The Object among the JS Object which doesn't have an array as an element (ex : `DataView Object`) stores the base address of the `emptyElementsHeader` at the `elements_` field.

    - `emptyElementsHeader` declares by static instance, so it is in the .data segment of EScript.api.
    ![image](https://github.com/user-attachments/assets/c16a8267-8f03-4d53-8aa7-314f9bcf0cd7)
    [Fig 5] The reference diagram of ArrayBuffer -> DataView -> emptyElementHeader

---
As I mentioned in [2. DataView Object](https://whs-segfault.github.io/adobe/2024/08/22/JS-Object-SpiderMonkey.html#h-2-dataview-object), if DataView Object references a non-existed property then DataView Object calls getProperty function to find it.

![image](https://github.com/user-attachments/assets/24ca4733-a21b-4556-8070-7c96f9707bce)
[Fig 6] Reference diagram to find the address of the getProperty function

## REFERENCE
1. [DataView Object](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/DataView/DataView)
2. [SpiderMonkey Internal](https://github.com/ricardoquesada/Spidermonkey/tree/master/js/src/vm)
3. [JSObject](https://www.sidechannel.blog/en/attacking-js-engines/)
