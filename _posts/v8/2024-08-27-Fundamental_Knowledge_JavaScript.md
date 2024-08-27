---
layout: post
title: Fundamental Knowledge of JavaScript ( V8, Pipeline, Garbage collection, Ubercage )
subtitle: Exploit Knowledge
categories: JavaScript
tags: [JavaScript, JS Object, Allocation Folding]
---

# [JavaScript] 배경 지식 - 자바 객체 구조, Allocation folding

## JS object structure

<center> <img src="https://github.com/user-attachments/assets/cb1bd7c2-38f4-47e3-aef5-6e5b736272ef" /> </center>

Javascript Object는 key-value로 이루어진 데이터 구조입니다. V8 엔진은 객체의 구조와 속성에 따라 메타데이터와 실제 데이터를 다르게 처리합니다. 

첫번째 필드에 들어가는 값은 map입니다.  Hidden class라고도 합니다. 히든 클래스는 객체의 프로퍼티 이름과 

타입, 각 프로퍼티가 객체 내에서 차지하는 위치에 대해 메타데이터를 포함합니다. 

히든 클래스에 대해 더 자세히 알아보겠습니다.

<center> <img src="https://github.com/user-attachments/assets/8b9b8fc2-6988-496a-b504-39417d8419a6" /> </center>

객체가 생성될 때 map이 생기게 되는데 새로운 Properties가 생길 때 마다 다른 map이 하나씩 생기게 됩니다.

또한 map은 이전의 map과 다음 map을 동시에 가리키고 있음으로 둘 다 찾아갈 수 있습니다.

<center> <img src="https://github.com/user-attachments/assets/82974c1a-e29d-473a-a78c-83e18962f9c2" /> </center>

이런식으로 구성이 되어 있는 걸 볼 수 있습니다. 

두번째 필드는 Properties 즉 속성입니다.  속성은 key-value로 이루어져 있습니다. 

예를 들면 이렇습니다 .

```jsx
let person = {
    name: "John",
    age: 30,
    isEmployed: true
};

```

이런 식으로 person이 정의가 되어 있다고 하면 

name은 속성 john은  값 

age 는 속성 30은 값 

isEmployed는 속성 true는 값

입니다.

세번째 필드는 Elements 입니다.  Js의 배열 요소를 저장하는 공간을 가리킵니다. 

일반 객체와 달리 배열 객체는 순차적인 숫자 인덱스를 사용하여 데이터를 저장합니다. 

Elements는 이런 배열의 요소들을 저장하거나 관리하는 역할을 합니다. 

네번째 필드는 있는 in-Object Properties는  객체 내부에 직접 저장된 속성들을 말합니다. 

빠른 접근을 위해 사용을 하며 속성이 일정 수 이상으로 많을 경우 사용하지 못하게 됩니다. 

이해를 돕기 위해 나중에 PoC에서 사용할 x 객체와 a 배열의 Debugprint를 가져왔습니다. 

위 내용과 비교하여 보시면 도움이 될 듯 합니다. 

<center> <img src="https://github.com/user-attachments/assets/a4bec213-f773-4842-8e89-f9222bf67d20" /> </center>

(a 배열의 DebugPrint)

<center> <img src="https://github.com/user-attachments/assets/7679ef28-4c9d-44bb-bb64-d9fac04a37bc" /> </center>

(x객체의 DebugPrint)

## Allocation folding

Allocation Folding이란 메모리에 여러 번 할당할 것을 한번에 할당하여 불필요한 할당횟수를 줄이는 메모리 할당 기법이다. 

<center> <img src="https://github.com/user-attachments/assets/bb16d73f-8ffd-4b9d-a259-f4f3d9518615" /> </center>

예를 들어 설명하자면 그림에 나와있는 Class B와 Array a를 할당할땐 각각 1번씩 메모리에 할당하게 되어 총 2번 할당하게 된다. 하지만 여기서 allocation folding 기법을 사용하게 되면 class B와 Array a의 크기를 더한 값 (size(x+y) 만큼의 공간을 할당하여 Array a와 Class B가 나누어 쓰게 된다. 

 

## JS Array

Typearray는 일정한 타입의 배열을 다루기 위해 제공되는 Js Object입니다. 

즉 배열에 들어가는 모든 요소가 동일한 데이터 타입(예: int8 , Uint8,int16 등등)을 가지고 있습니다.

<center> <img src="https://github.com/user-attachments/assets/ce184978-674a-4dce-afb6-a0bc99215d8f" /> </center>

(TypedArray의 종류)

TypedArray를 사용하는 이유는 다 동일한 타입을 가지고 있기 때문에 고정된 크기를 가지고 있어 성능 최적화에 유리합니다. 

또한 Packed Array와 Holey Array의 개념이 있는데 Packed Array부터 설명을 하자면

모든 요소가 연속적으로 정의된 배열입니다. 배열 내에 모든 인덱스가 가득 차 있다는 말입니다.

이 형태의 배열은 메모리에서 연속적으로 저장되므로 접근 속도가 빠릅니다.

다음은 Holey Array 입니다. 배열의 중간에 정의되지 않은 요소 (예: undefined , 특정 인덱스에 값 x) 

이 경우 메모리에서 비연속적으로 저장되게 되어 성능이 저하됩니다. 

코드로 예를 들어보자면

```jsx
let array = [0,1,2]; 
array[3] = 3.1
array[3] = 4
array[4] = "five"
array[6] = 6
```

첫줄은 PACKED_SMI_ELEMENTS이다. 여기서 Smi란 31(x32)비트 또는 63(x64)비트 크기의 작은 정수를 나타내는 v8 엔진 내부의 데이터 형식이다. 모든 배열이 다 차 있기 때문에 PACKED입니다.

두번째줄은 마찬가지로 PACKED이지만 PACKED_DOUBE_ELEMENT이다. 원랜 SMI만 존재했지만 Float값이 들어가 Smi와  Float가 한 배열 안에 존재하기 때문입니다.

세번째줄은  여전히 PACKED_DOUBLE_ELEMENTS입다.  왜냐하면 마지막 부분이 다시 SMI 값이 된다고 PACKED_SMI_ELEMENT가 되는  것이 아니라 한 번 변한 배열은 다시 돌아오지 않기 때문이다. 그렇기 때문애 

PACKED_DOUBLE_ELEMENTS이다.

네번째줄은 PACKED_ELEMENTS이다. 인덱스는 여전히 가득 차 있지만 종류가 3개이상이 되어 DOUBLE이 사라지게 되었다.

다섯번째줄은 HOLEY_ELEMENTS이다. 왜냐하면 갑자기 인덱스 6을 넣었는데 그 전의 값인 array[5]값이 없기 때문에 빈 공간이 생겨서 Holey가 생겼기 때문이다.

나중에 Exploit분석에 나올 내용이니 잘 봐두면 좋다. 

Reference 

Js Object Structure: [https://devkly.com/nodejs/javascript-array/](https://devkly.com/nodejs/javascript-array/)

V8 Hiddenclass : [https://v8.dev/docs/hidden-classes](https://v8.dev/docs/hidden-classes)

v8 TypedArray:[https://v8docs.nodesource.com/node-7.10/d5/dbb/classv8_1_1_typed_array.html](https://v8docs.nodesource.com/node-7.10/d5/dbb/classv8_1_1_typed_array.html)
