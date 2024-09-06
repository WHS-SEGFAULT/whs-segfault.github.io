---
layout: post
title: [English] Windows Heap
subtitle: Exploit Knowledge
categories: Adobe
tags: [CVE-2021-39863, Windows, LFH, UserBlock]
---

- There are two types of Windows heap allocation mechanisms:
    1. **NT Heap** : This is the traditional Windows heap allocation mechanism.
    2. **Segment Heap** : Introduced in Windows 10, this is a newer heap allocation mechanism. However, i[English] Windows Heap ac6965382984472ea71940afa5bc3f8dt has not been widely adopted, because many programs are optimized for the NT heap.

## 1. NT Heap and LFH

- To reduce heap fragmentation and improve performance, heap allocation and management are handled using both Back-End Heap and Front-End Heap mechanisms.

---

- **Back-End Heap :**
    - This process involves the Windows OS directly allocating memory for the heap.
    - Since it accesses memory directly, the processing speed tends to be slower.

---

- **Front-End (LFH, Low Fragmentation Heap) :**
    - LFH is the default Front-End heap manager in Windows OS since the introduction of Windows Vista.
    - It reduces heap fragmentation and improves performance.
    - **How LFH Works:**
        1. When heaps of the same size are allocated multiple times by the Back-End Heap, LFH pre-allocates multiple heaps of that size.
            1. It divides a contiguous heap region into equal-sized sections, preparing multiple heap chunks of the same size.
            2. The `UserBlock` structure manages these equally-sized heap chunks that were divided in the previous step.
        2. When a heap of the same size needs to be allocated again, LFH returns a heap from the chunks managed by the `UserBlock` structure.
            
            ⇒ This improves performance by avoiding the need to access the Back-End heap and reduces heap fragmentation as heaps of the same size are placed consecutively.
            

## 2. UserBlock

- The **UserBlock** structure consists of a UserBlock Header and multiple Heap Chunks.
- **UserBlock Header :**
    - The UserBlock Header contains the following values:
        - **Signature :** A constant that indicates the block is an LFH UserBlock, with the value `0xF0E0D0C0`.
        - **BusyBitmap.Data :** This points to the BitmapData of the UserBlock Header. It is used to access memory addresses within the UserBlock.
        - **BitmapData :** This records the information of the allocated Heap Chunks within the heap memory managed by the UserBlock.
- **Heap Chunks :**
    - Each Heap Chunk includes an 0x8-byte size **Heap Chunk Header**.
    - The Heap Chunk Header contains the **Chunk Number**.
        - The **Chunk Number** is indexed starting from 0 for the first Heap Chunk, which is closest to the UserBlock Header.

    ![image](https://github.com/user-attachments/assets/0817ca65-6609-4abe-84d1-cc495f221c99)
    **[Figure 1] Structure of the UserBlock. The UserBlock Header is partially shown.**

# **REFERENCE**

1. [Windows 10 Nt Heap Exploitation](https://www.slideshare.net/slideshow/windows-10-nt-heap-exploitation-english-version/154467191)
2. [NT heap in CTF](https://null2root.github.io/blog/2020/02/07/LazyFragmentationHeap-WCTF2019-writeup.html)
3. [Windows 7 NT Heap Internal](https://illmatics.com/Understanding_the_LFH.pdf)