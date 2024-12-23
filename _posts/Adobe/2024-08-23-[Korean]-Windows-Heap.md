---
layout: post
title: Windows Heap
subtitle: Exploit Knowledge
excerpt_image: NO_EXCERPT_IMAGE
categories: Adobe
tags: [CVE-2021-39863, Windows, LFH, UserBlock]
---

- Windows Heap 할당 매커니즘에는 두 종류가 있다.
    1. NT heap : 기존에 존재하던 Windows Heap 할당 매커니즘이다.
    2. Segment Heap
        
        : Windows 10부터 추가되었으나 많은 프로그램이 NT heap에 최적화되어 상용화되지는 않은 Windows Heap 할당 매커니즘이다.
        

## 1. NT Heap, LFH

- Heap Fragmentation을 낮추고, 성능을 향상 시키기 위해 Back-End와 Front-End를 이용해 heap을 할당하고 관리한다.

---

- Back-End Heap
    - Windows OS가 직접 heap을 할당하는 과정으로
    - 메모리에 직접 접근해서 heap을 할당하기에 처리 속도가 느리다.

---

- Front-End (LFH,  Low Fragmentation Heap)
    - LFH : Windows Vista 등장 이후 Windows OS의 기본 Front End 힙 관리자이다.
    - Heap Fragmentation을 낮춰주고, 성능을 향상시켜준다.
    - LFH의 동작 과정
        1. 동일한 크기의 heap이 Back-End Heap으로 여러 번 할당되면
        2. LFH가 해당 크기의 heap 여러 개를 미리 준비한다.
            1. 연속된 Heap 영역을 동일한 크기로 나눠 동일 크기의 Heap Chunk들을 준비한다.
            2. UserBlock 구조체가 a.에서 나눈 동일한 크기의 Heap Chunk를 관리한다.
        3. 이후 동일한 크기의 heap이 또 할당될 때, LFH는 UserBlock 구조체가 관리하는 Heap을 반환하여 할당한다.
            
            ⇒ Back-End heap까지 가지 않아 성능이 향상되고,
            
            동일한 크기의 heap이 연속되어 위치하므로 Heap Fragmentation이 감소한다.
            
## 2. UserBlock

- UserBlock 구조체는 UserBlock Header와 많은 수의 Heap Chunk로 구성된다.
- UserBlock Header
    - UserBlock Header에는 다음 값들이 저장되어 있다.
        - `Signature` : LFH UserBlock 임을 나타내주는 상수, `0xF0E0D0C0`이 저장되어있다.
        - `BusyBitmap.Data`
            
            : UserBlock Header의 `BitmapData`를 가리킨다. 임의 주소에 접근하기 위해 이 값을 사용한다.
            
        - `BitmapData`
            
            : UserBlock이 관리하는 Heap 메모리 (Heap Chunk) 중 할당된 Heap Chunk의 정보가 기록되어있다.
            
            +) 각각의 Heap Chunk에는 0x8 byte의 `Heap Chunk Header`가 있고, Heap Chunk Header에는 `Chunk Number`가 존재한다.
            
            `Chunk Number`는 UserBlock Header와 제일 가까운 Heap Chunk부터 0으로 인덱싱 된다.
            
    ![image](https://github.com/user-attachments/assets/0817ca65-6609-4abe-84d1-cc495f221c99)
    [그림 1] UserBlock 구조체의 구조. UserBlock Header는 일부만 나타냄

## REFERENCE
1. [Windows 10 Nt Heap Exploitation](https://www.slideshare.net/slideshow/windows-10-nt-heap-exploitation-english-version/154467191)
2. [NT heap in CTF](https://null2root.github.io/blog/2020/02/07/LazyFragmentationHeap-WCTF2019-writeup.html)
3. [Windows 7 NT Heap Internal](https://illmatics.com/Understanding_the_LFH.pdf)
